# Documentazione Tecnica: Gestione delle Comunicazioni Intermittenti e Disconnessioni in Bramble

Questo documento descrive in dettaglio la suite di meccanismi, architetture e componenti con cui la libreria **Bramble** (`org.briarproject.bramble`) gestisce le **comunicazioni intermittenti**, le disconnessioni improvvise, i timeout e la riconnessione automatica tra nodi mobili in contesti ad-hoc e DTN (*Disruption-Tolerant Networking*).

---

## 1. Architettura della Resilienza alle Interruzioni

Nelle reti mobili ad-hoc (MANET / DTN), le disconnessioni radio tra smartphone (es. perdita del raggio Bluetooth, disattivazione del Wi-Fi Direct, schermature fisiche) non sono eccezioni ma la **norma operativa**.

Bramble gestisce l'intermittenza integrando **4 livelli di resilienza**:

```
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ 1. LIVELLO PERSISTENZA DTN (`DatabaseComponentImpl.java`)                   │
 │    └─ Nessun dato "al volo". Salva tutto nel DB Cifrato prima di inviare    │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 2. LIVELLO TIMEOUT & RILEVAMENTO (`TimeoutMonitorImpl.java`)                │
 │    └─ Monitora le letture bloccate e forzando la chiusura pulita dei socket │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 3. LIVELLO RICONNESSIONE & POLLING (`PollerImpl.java` & `BackoffImpl.java`) │
 │    └─ Polling periodico con Backoff Esponenziale per ripristinare il canale │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 4. LIVELLO SINCRONIZZAZIONE EPIDEMICA (`DuplexOutgoingSession.java`)        │
 │    └─ Ciclo OFFER/REQUEST/MESSAGE/ACK con deduplicazione idempotente        │
 └─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Dettaglio dei 4 Meccanismi di Gestione dell'Intermittenza

---

### A. Persistenza Transazionale (Store-Carry-and-Forward)
Quando l'utente crea un messaggio o quando un messaggio viene ricevuto:
* Il messaggio **non rimane solo nella RAM del socket**, ma viene salvato tramite una transazione atomica nel database cifrato H2/SQLite (`DatabaseComponentImpl`).
* Il messaggio rimane marcato come "da consegnare" (`ack = false`) finché il nodo non riceve una conferma esplicita dal destinatario.
* Se la connessione cade prima della consegna, il messaggio **rimane al sicuro nel DB** in attesa del successivo incontro.

---

### B. Rilevamento delle Interruzioni Silent e Timeout (`TimeoutMonitorImpl.java` & `TimeoutInputStream.java`)
In una rete ad-hoc, quando due nodi si allontanano, il segnale radio sparisce spesso in modo *silent* (senza l'invio del pacchetto TCP FIN o della disconnessione pulita Bluetooth). Per evitare che i thread rimangano bloccati all'infinito in lettura:
* `TimeoutInputStream` registra il timestamp di inizio lettura `readStartedMs`.
* `TimeoutMonitorImpl` esegue un task periodico ogni 10 secondi (`CHECK_INTERVAL_MS`). Se un'operazione di lettura supera `timeoutMs`, forzatamente invoca `stream.close()`.
* La chiusura genera un'eccezione `IOException`, rilascia i socket e scatena l'evento `ConnectionClosedEvent` per ripulire la sessione.

```java
// File: bramble-core/src/main/java/org/briarproject/bramble/io/TimeoutMonitorImpl.java
// Fonte: Repository Ufficiale Briar (GitLab)

class TimeoutMonitorImpl implements TimeoutMonitor {

    private static final long CHECK_INTERVAL_MS = SECONDS.toMillis(10);
    private final TaskScheduler scheduler;

    @IoExecutor
    @Wakeful
    private void checkTimeouts() {
        List<TimeoutInputStream> snapshot;
        synchronized (lock) {
            snapshot = new ArrayList<>(streams);
        }
        for (TimeoutInputStream stream : snapshot) {
            // Rileva se la lettura è bloccata oltre la soglia di timeout
            if (stream.hasTimedOut()) {
                LOG.info("Input stream ha superato il timeout -> Chiusura forzata del socket");
                try {
                    stream.close(); // Forzi la chiusura per sbloccare i thread
                } catch (IOException e) {
                    logException(LOG, INFO, e);
                }
            }
        }
    }
}
```

---

### C. Riconressione Automatica e Backoff Esponenziale (`PollerImpl.java` & `BackoffImpl.java`)
Quando la connessione cade, Bramble non si arrende ma avvia una routine di ricerca e riconnessione dello smartphone del contatto:
* **`PollerImpl.java`**: Ascolta gli eventi di disconnessione (`ConnectionClosedEvent` / `TransportInactiveEvent`).
* **`BackoffImpl.java`**: Per preservare la batteria ed evitare la congestione del canale radio, calcola l'intervallo di tentativo successivo con l'algoritmo del **Backoff Esponenziale**:
  $$\text{pollingInterval} = \min(\text{minInterval} \times \text{base}^{\text{backoff}}, \text{maxInterval})$$
* **Reset al Riconaggiungimento**: Appena le interfacce radio si rilevano nuovamente (`ConnectionOpenedEvent`), il contatore di backoff viene azzerato (`backoff.reset()`) e la sincronizzazione riprende immediatamente.

```java
// File: bramble-core/src/main/java/org/briarproject/bramble/plugin/BackoffImpl.java
// Fonte: Repository Ufficiale Briar (GitLab)

class BackoffImpl implements Backoff {

    private final int minInterval, maxInterval;
    private final double base;
    private final AtomicInteger backoff;

    @Override
    public int getPollingInterval() {
        // Calcola l'intervallo con incremento esponenziale
        double multiplier = Math.pow(base, backoff.get());
        int interval = (int) (minInterval * multiplier);
        return Math.min(interval, maxInterval);
    }

    @Override
    public void increment() {
        backoff.incrementAndGet(); // Aumenta l'attesa se il contatto è ancora irraggiungibile
    }

    @Override
    public void reset() {
        backoff.set(0); // Reset immediato al ripristino della connessione
    }
}
```

---

### D. Ripresa della Sincronizzazione e Deduplicazione Idempotente (`DuplexOutgoingSession.java`)
Quando la connessione viene ripristinata dopo un'interruzione:
1. Le due sessioni BSS scambiano nuovamente i record `VERSIONS` e poi `OFFER`.
2. Il mittente offre la lista degli ID dei messaggi non ancora confermati con `ACK`.
3. **Gestione dell'Interruzione a Metà**: Se la disconnessione precedente era avvenuta dopo che il ricevitore aveva salvato il messaggio nel DB ma prima che il mittente avesse ricevuto l'ACK:
   * Il mittente ri-offre il messaggio via `OFFER`.
   * Il ricevitore rileva che il `MessageId` è già presente nel DB (`containsMessage()`) e **non lo richiede nuovamente**, o restituisce subito l'ACK mancante.
   * L'operazione è **completamente idempotente**: l'intermittenza della rete non genera mai duplicati per l'utente.

```java
// File: bramble-core/src/main/java/org/briarproject/bramble/sync/DuplexOutgoingSession.java
// Fonte: Repository Ufficiale Briar (GitLab)

public class DuplexOutgoingSession implements SyncSession {

    // Se la rete si ripristina, interroga il DB e ri-offre tutti i messaggi ancora sprovvisti di ACK
    private void generateOffer() {
        dbExecutor.execute(() -> {
            try {
                Offer offer = db.getOffer(contactId);
                if (offer != null) {
                    recordWriter.writeOffer(offer); // Ri-trasmette l'offerta BSS
                }
            } catch (IOException | DbException e) {
                LOG.warning("Errore nella generazione dell'offerta dopo riconnessione: " + e.getMessage());
            }
        });
    }
}
```

---

## 3. Schema del Ciclo di Vita di una Connessione Intermittente

```
   ┌─────────────────────────────────────────────────────────────────────────┐
   │                          CONNESSIONE ATTIVA                             │
   │   - Trasferimento dati BTP / BSS (OFFER -> REQUEST -> MESSAGE -> ACK)   │
   └────────────────────────────────────┬────────────────────────────────────┘
                                        │
                         [ Interruzione Radio / Distanza ]
                                        │
                                        ▼
   ┌─────────────────────────────────────────────────────────────────────────┐
   │                  TIMEOUT & RILEVAMENTO DISCONNESSIONE                   │
   │   - `TimeoutMonitorImpl` rileva lettura bloccata                        │
   │   - `stream.close()` forza il rilascio del socket                       │
   │   - Evento: `ConnectionClosedEvent`                                     │
   └────────────────────────────────────┬────────────────────────────────────┘
                                        │
                                        ▼
   ┌─────────────────────────────────────────────────────────────────────────┐
   │                  RECOVERY & POLLING (BACKOFF ESPONENZIALE)              │
   │   - `PollerImpl` avvia il task di ricerca periodica                     │
   │   - `BackoffImpl` aumenta l'intervallo (minInterval -> maxInterval)     │
   └────────────────────────────────────┬────────────────────────────────────┘
                                        │
                     [ I nodi tornano vicini / Wi-Fi attivo ]
                                        │
                                        ▼
   ┌─────────────────────────────────────────────────────────────────────────┐
   │                   RIPRISTINO & SINCRONIZZAZIONE IDEMPOTENTE             │
   │   - Evento: `ConnectionOpenedEvent`                                     │
   │   - Reset Backoff (`backoff.reset()`)                                   │
   │   - BSS scambia OFFER per messaggi senza ACK -> Deduplicazione SHA-256  │
   └─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Fonti Ufficiali e Bibliografia

1. **Repository Ufficiale del Codice Sorgente di Briar / Bramble**:
   * *Repository*: [Briar Project GitLab Repository](https://code.briarproject.org/briar/briar)
   * *Classi Verificate*:
     * `bramble-core/.../io/TimeoutInputStream.java`
     * `bramble-core/.../io/TimeoutMonitorImpl.java`
     * `bramble-core/.../plugin/BackoffImpl.java`
     * `bramble-core/.../plugin/PollerImpl.java`
     * `bramble-core/.../sync/DuplexOutgoingSession.java`
2. **Standard IETF per Reti Intermittenti (DTN)**:
   * **RFC 4838**: *Delay-Tolerant Networking Architecture* (IETF DTN Research Group) — Definisce il paradigma *Store-Carry-and-Forward*.
   * **RFC 9171**: *Bundle Protocol Version 7* — Standard IETF per la persistenza ed il routing su connessioni ad-hoc intermittenti.
