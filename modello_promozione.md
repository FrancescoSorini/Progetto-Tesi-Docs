# Modello di Promozione a FULL_MEMBER e Requisiti di Fiducia della Sandbox

Questo documento definisce il **Modello di Promozione e Fiducia** per il passaggio di un nodo dallo stato di Sandbox (**`PROBATION`**) a quello di membro effettivo (**`FULL_MEMBER`**), risolvendo la discrepanza tra approvazione a singolo nodo e **Quorum di Prossimità**.

---

## 1. Principi Guida della Sicurezza in Rete P2P/DTN

In una rete ad-hoc per scenari di emergenza e censura, dare fiducia a un nuovo nodo è un'operazione critica per prevenire:
* **Attacchi Sybil**: Creazione di centinaia di nodi virtuali malevoli per falsificare notizie o manipolare i quorum.
* **Infiltrazione e Spettro Spam**: Nodi che diffondono disinformazione o intasano la rete con rate elevati.
* **Collusione a Singolo Nodo**: Un solo membro compromesso che approva istantaneamente nodi malevoli.

Pertanto, l'uscita dalla Sandbox **non può dipendere da un singolo evento o da un solo membro**, ma deve derivare da un **Modello di Valutazione Ibrido (Multicriterio)** basato su:
1. **Tempo e Permanenza** 
2. **Quorum di Prossimità Multisponsor**
3. **Incontri Fisici Diversificati** 
4. **Attività Utile e Comportamento Pulito**

---

## 2. Modello dei Requisiti Minimi per la Promozione

Per poter passare dallo stato `PROBATION` a `FULL_MEMBER`, il nodo di un nuovo utente deve **soddisfare TUTTI i seguenti 5 requisiti concorrenti** (i valori per la permanenza minima, il quorum di prossimità e la diversità di incontri non sono da intendersi fissi ma modificabili in base al contesto e/o necessità):

```
 ┌─────────────────────────────────────────────────────────────────────────┐
 │                       REQUISITI DI PROMOZIONE                           │
 ├─────────────────────────────────────────────────────────────────────────┤
 │ 1. PERMANENZA MINIMA    : Almeno 3 giorni (72 ore) dall'onboarding      │
 │ 2. SPONSOR INIZIALE     : Link d'invito o prima approvazione da FULL    │
 │ 3. QUORUM DI PROSSIMITÀ : Almeno K=3 Endorsement firmati da nodi diversi│
 │ 4. DIVERSITÀ INCONTRI   : Almeno 3 sessioni P2P distribuite nel tempo   │
 │ 5. BUON COMPORTAMENTO   : 0 segnalazioni/flag e >= 1 post/voto utile    │
 └─────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
                   [ Promozione a FULL_MEMBER Sbloccata ]
```

---

### Dettaglio dei 5 Requisiti:

#### 1. Permanenza Minima Attiva (Time-Lock di Osservazione)
* **Regola**: Il nodo deve essere stato inizializzato nel sistema da **almeno 72 ore (3 giorni solari)**.
* **Motivazione**: Impedisce ad attaccanti di entrare nella rete e diventare immediatamente membri pieni in poche ore per condurre attacchi lampo o alterare le decisioni di un'emergenza in corso.
* **Verifica**: `System.currentTimeMillis() - sandboxStartTime >= 3 * 86_400_000L`.

#### 2. Presenza di uno Sponsor Iniziale (Sponsor / Introducer)
* **Regola**: Il nodo deve possedere una "Catena di Origine":
  * *Ingresso via Invito Direct*: Lo `SponsorAuthorId` è il `FULL_MEMBER` che ha generato il QR code d'invito.
  * *Ingresso via Scansione Autonoma*: Lo `SponsorAuthorId` è il primo `FULL_MEMBER` che incontra via P2P e che accetta di "sponsorizzare" la richiesta d'ingresso.
* **Motivazione**: Ancora l'identità dell'applicante alla rete di fiducia preesistente.

#### 3. Quorum di Prossimità Multisponsor ($K = 3$ Endorsement Distinti)
* **Regola**: Il nodo dell'applicante deve raccogliere nel proprio database cifrato **almeno $K = 3$ messaggi di attestazione di fiducia (`TrustEndorsementMessage`)**, ciascuno firmato con chiave Ed25519 da un `FULL_MEMBER` differente.
* **Motivazione**: Elimina il punto singolo di fallimento. Un singolo membro corrotto non può promuovere autonomamente un utente.
* **Gestione degli Sponsor**: L'eventuale sponsor iniziale conta come **1 dei 3** voti del quorum. Gli altri 2 devono provenire da membri indipendenti incontrati successivamente.

#### 4. Diversità e Prova di Incontri Fisici P2P (Encounter Diversity)
* **Regola**: I 3 endorsement del quorum devono essere stati scambiati in **almeno 2 o 3 sessioni di sincronizzazione P2P distinte**, separate nel tempo da un intervallo minimo (es. almeno 6 ore tra un incontro e l'altro).
* **Motivazione**: Evita che un utente crei 3 identità su un singolo telefono virtuale o in un unico secondo via hotspot e firmi contemporaneamente per sé stesso.

#### 5. Partecipazione Utile e Assenza di Violazioni (Good Standing & Activity Score)
* **Regola**:
  * **Attività Minima**: L'utente deve aver pubblicato **almeno 1 o 2 post/bozze di qualità** o aver espresso voti/conferme di vicinato durante il periodo di prova.
  * **Zero Segnalazioni (Clean Record)**: Il nodo non deve aver ricevuto **alcuna segnalazione di abuso o contestazione** (`SpamFlagMessage` / `AbuseReport`) inviata da un `FULL_MEMBER` della vicinanza.
* **Motivazione**: Premia l'utente costruttivo e blocca chi invia spam o contenuti dannosi.

---

## 3. Matrice Comparativa degli Stati: `PROBATION` vs `FULL_MEMBER`

| Capacità / Privilegio | Nodo in Sandbox (`PROBATION`) | Nodo Effettivo (`FULL_MEMBER`) |
| :--- | :--- | :--- |
| **Quota di Pubblicazione** | Max **1 post al giorno** | **Senza limiti** (o rate limit anti-spam standard) |
| **Visibilità Contenuti Ricevuti** | Vede solo post di categoria **`PUBLIC_GENERAL`** | Vede **tutti i post**, inclusi i report **`CRITICAL`** |
| **Sincronizzazione P2P / Relè** | Sincronizza solo coi **peer direttamente adiacenti** | Partecipa attivamente al **routing epidemico completo** |
| **Diritto di Voto su Quorum** | **Disabilitato** (i suoi voti su altri nodi/post sono ignorati) | **Abilitato** (il suo voto conta per quorum e promozioni) |
| **Generazione Inviti / Sponsor** | **Impossibile** (non può invitare altri utenti) | **Abilitato** (può generare QR di invito e fare da sponsor) |

---

## 4. Architettura dei Dati e Messaggi BDF in Briar

Per supportare questo modello senza stravolgere Bramble, definiamo le strutture dati e i metadati da salvare nel DB cifrato.

### A. Metadati di Sandbox salvati in `Settings` (`community.trust`)

Nel database cifrato di ciascun nodo applicante, `AccountManagerImpl` e `TrustManagerImpl` mantengono la seguente mappa di impostazioni:

```java
// Namespace: "community.trust"
{
  "trustTier": "PROBATION",                          // PROBATION -> FULL_MEMBER -> ISOLATED
  "sandboxStartTime": "1728310000000",               // Timestamp di creazione account (ms)
  "sponsorAuthorId": "a1b2c3d4...",                  // AuthorId del primo sponsor
  "endorsementCount": "2",                           // Numero attuale di voti raccolti (soglia = 3)
  "endorsementAuthors": "authorId1,authorId2",       // Elenco degli AuthorId che hanno votato
  "validPostsCount": "1",                            // Numero di post validi pubblicati
  "abuseFlagCount": "0",                             // Numero di segnalazioni ricevute
  "lastEncounterTimestamp": "1728320000000"          // Timestamp dell'ultimo incontro P2P
}
```

---

### B. Messaggio BDF: `TrustEndorsementMessage`

Quando un `FULL_MEMBER` incontra l'applicante via P2P e decide di esprimere fiducia nei suoi confronti, la sua app genera un messaggio BDF di attestazione che viene propagato al nuovo membro:

```java
// File NUOVO: briar-api/.../community/TrustEndorsementMessage.java

public Message createTrustEndorsementMessage(Author applicantAuthor)
        throws FormatException, GeneralSecurityException {

    long timestamp = System.currentTimeMillis();
    Author localFullMember = identityManager.getLocalAuthor();

    // 1. Corpo della dichiarazione di fiducia
    BdfList bodyToSign = BdfList.of(
        MSG_TYPE_TRUST_ENDORSEMENT,                 // Integer: 2
        clientHelper.toList(applicantAuthor),       // Autore da promuovere (Alice)
        clientHelper.toList(localFullMember),      // Autore votante (Bob, FULL_MEMBER)
        timestamp                                   // Timestamp dell'attestazione
    );

    // 2. Firma del FULL_MEMBER
    byte[] signature = clientHelper.sign(
        "org.briarproject.briar.community/TRUST_ENDORSEMENT",
        bodyToSign,
        localFullMember.getPrivateKey()
    );

    BdfList fullBody = BdfList.of(bodyToSign, signature);

    return clientHelper.createMessage(
        communityTopicGroupId,
        timestamp,
        clientHelper.toByteArray(fullBody)
    );
}
```

---

## 5. Algoritmo di Valutazione della Promozione (`TrustManagerImpl.java`)

Ogni volta che arriva un nuovo `TrustEndorsementMessage`, che trascorre del tempo o che si completa una sessione P2P, l'engine invoca la funzione `evaluatePromotionEligibility()`:

```java
// File: bramble-core/src/main/java/org/briarproject/bramble/trust/TrustManagerImpl.java

package org.briarproject.bramble.trust;

import org.briarproject.bramble.api.db.DbException;
import org.briarproject.bramble.api.settings.Settings;
import org.briarproject.bramble.api.settings.SettingsManager;
import javax.inject.Inject;
import java.util.logging.Logger;

public class TrustManagerImpl {

    private static final Logger LOG = Logger.getLogger(TrustManagerImpl.class.getName());
    private static final String NAMESPACE_COMMUNITY = "community.trust";

    // Costanti per i Requisiti Minimi
    private static final long MIN_SANDBOX_DURATION_MS = 3 * 86_400_000L; // 3 Giorni (72 ore)
    private static final int REQUIRED_ENDORSEMENT_QUORUM = 3;             // K = 3 sponsor/voti
    private static final int MIN_VALID_POSTS = 1;                         // Almeno 1 contributo
    private static final int MAX_ALLOWED_ABUSE_FLAGS = 0;                 // 0 tolleranza violazioni

    private final SettingsManager settingsManager;

    @Inject
    public TrustManagerImpl(SettingsManager settingsManager) {
        this.settingsManager = settingsManager;
    }

    /**
     * Valuta se l'utente locale in Sandbox soddisfa TUTTI i requisiti per la promozione a FULL_MEMBER.
     */
    public synchronized boolean evaluatePromotionEligibility() throws DbException {
        Settings settings = settingsManager.getSettings(NAMESPACE_COMMUNITY);

        String currentTier = settings.getOrDefault("trustTier", "PROBATION");
        if ("FULL_MEMBER".equals(currentTier)) return true; // Già promosso
        if ("ISOLATED".equals(currentTier)) return false;   // Nodo isolato/bannato

        // 1. VERIFICA TEMPO TRASCORSO (Almeno 3 Giorni)
        long startTime = Long.parseLong(settings.getOrDefault("sandboxStartTime", "0"));
        long elapsedTime = System.currentTimeMillis() - startTime;
        if (elapsedTime < MIN_SANDBOX_DURATION_MS) {
            LOG.info("Promozione negata: tempo in sandbox insufficiente (" + (elapsedTime / 3600000) + "h / 72h)");
            return false;
        }

        // 2. VERIFICA QUORUM DI PROSSIMITÀ (Almeno K=3 Endorsement distinti)
        int endorsementCount = Integer.parseInt(settings.getOrDefault("endorsementCount", "0"));
        if (endorsementCount < REQUIRED_ENDORSEMENT_QUORUM) {
            LOG.info("Promozione negata: quorum di voti insufficiente (" + endorsementCount + " / " + REQUIRED_ENDORSEMENT_QUORUM + ")");
            return false;
        }

        // 3. VERIFICA PRESENZA SPONSOR INIZIALE
        String sponsorId = settings.get("sponsorAuthorId");
        if (sponsorId == null || sponsorId.isEmpty()) {
            LOG.info("Promozione negata: sponsor iniziale mancante.");
            return false;
        }

        // 4. VERIFICA PARTECIPAZIONE ED ATTIVITÀ MINIMA
        int postsCount = Integer.parseInt(settings.getOrDefault("validPostsCount", "0"));
        if (postsCount < MIN_VALID_POSTS) {
            LOG.info("Promozione negata: nessuna attività utile registrata.");
            return false;
        }

        // 5. VERIFICA ASSENZA VIOLAZIONI / SEGNALAZIONI (Clean Record)
        int abuseFlags = Integer.parseInt(settings.getOrDefault("abuseFlagCount", "0"));
        if (abuseFlags > MAX_ALLOWED_ABUSE_FLAGS) {
            LOG.warning("Promozione negata: il nodo ha ricevuto segnalazioni di abuso (" + abuseFlags + ")");
            return false;
        }

        // TUTTI I REQUISITI SODDISFATTI -> ESEGUI LA PROMOZIONE!
        executePromotionToFullMember();
        return true;
    }

    /**
     * Esegue la promozione effettiva aggiornando il DB cifrato e notificando il sistema.
     */
    private void executePromotionToFullMember() throws DbException {
        Settings newSettings = new Settings();
        newSettings.put("trustTier", "FULL_MEMBER");
        newSettings.put("promotionTimestamp", String.valueOf(System.currentTimeMillis()));

        settingsManager.mergeSettings(newSettings, NAMESPACE_COMMUNITY);
        LOG.info(" PROMOZIONE COMPLETATA CON SUCCESSO! Il nodo ora è FULL_MEMBER.");
    }
}
```
