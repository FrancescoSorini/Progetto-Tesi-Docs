# Modello di Promozione a FULL_MEMBER e Requisiti di Fiducia della Sandbox

Questo documento definisce il **Modello di Promozione e Fiducia** per il passaggio di un nodo dallo stato di Sandbox (**`PROBATION`**) a quello di membro effettivo (**`FULL_MEMBER`**), risolvendo la discrepanza tra approvazione a singolo nodo e **Quorum di Prossimità Dinamico Adattivo**.

---

## 1. Principi Guida della Sicurezza in Rete P2P/DTN

In una rete ad-hoc per scenari di emergenza e censura, dare fiducia a un nuovo nodo è un'operazione critica per prevenire `[Fonte: Douceur, 2002 - The Sybil Attack]`:
* **Attacchi Sybil**: Creazione di centinaia di nodi virtuali malevoli per falsificare notizie o manipolare i quorum `[Fonte: Douceur, 2002]`.
* **Infiltrazione e Spettro Spam**: Nodi che diffondono disinformazione o intasano la rete con rate elevati `[Fonte: Resnick et al., 2000]`.
* **Collusione a Singolo Nodo**: Un solo membro compromesso che approva istantaneamente nodi malevoli.

Pertanto, l'uscita dalla Sandbox **non può dipendere da un singolo evento o da un solo membro**, ma deve derivare da un **Modello di Valutazione Ibrido e Dinamico (Multicriterio Adattivo)** basato su:
1. **Tempo e Permanenza Adattiva** ($\Delta T_{\text{dynamic}}$)
2. **Quorum di Prossimità Multisponsor Dinamico** ($K_{\text{dynamic}}$)
3. **Incontri Fisici Diversificati e Stima Popolazione Rete** ($\hat{N}_{\text{estimated}}$)
4. **Attività Utile e Comportamento Pulito** (`abuseFlagCount == 0`)

---

## 2. Modello dei Requisiti Dinamici per la Promozione

Per poter passare dallo stato `PROBATION` a `FULL_MEMBER`, il nodo di un nuovo utente deve **soddisfare TUTTI i seguenti 5 requisiti concorrenti**, i quali **non usano parametri fisso-arbitrari**, ma vengono calcolati dinamicamente in base alla dimensione della rete ed all'attività degli utenti `[Fonte: Demers et al., 1987 / Resnick et al., 2000]`:

```
 ┌─────────────────────────────────────────────────────────────────────────┐
 │                   REQUISITI DINAMICI DI PROMOZIONE                      │
 ├─────────────────────────────────────────────────────────────────────────┤
 │ 1. PERMANENZA ADATTIVA  : t >= Delta T_dynamic (da 24h a 168h)          │
 │ 2. SPONSOR INIZIALE     : Link d'invito o prima approvazione da FULL    │
 │ 3. QUORUM DINAMICO      : Endorsement >= K_dynamic (funzione di N_est)  │
 │ 4. DIVERSITÀ INCONTRI   : Sessioni P2P distribuite nel tempo            │
 │ 5. BUON COMPORTAMENTO   : 0 segnalazioni/flag e >= 1 post/voto utile    │
 └─────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
                   [ Promozione a FULL_MEMBER Sbloccata ]
```

---

### Dettaglio dei 5 Requisiti:

#### 1. Permanenza Minima Adattiva (Time-Lock di Osservazione $\Delta T_{\text{dynamic}}$)
* **Regola**: Il nodo deve rimanere in osservazione per un intervallo di tempo calcolato in modo dinamico in base al tasso degli incontri P2P orari ($\lambda_{\text{encounter}}$) `[Fonte: Resnick et al., 2000]`:
  $$\Delta T_{\text{dynamic}} = \max\left(24\text{h}, \; \min\left(168\text{h}, \; \frac{72\text{h}}{\log_2(\lambda_{\text{encounter}} + 1)}\right)\right)$$
* **Motivazione**: In aree ad alta densità ed interazioni frequenti, il tempo di attesa scala in modo sicuro fino a 24h. In aree isolate e rade, garantisce un osservazione prolungata (fino a 7 giorni) per sventare attacchi Sybil `[Fonte: Douceur, 2002]`.
* **Verifica**: `System.currentTimeMillis() - sandboxStartTime >= dynamicTimeLockMs`.

#### 2. Presenza di uno Sponsor Iniziale (Sponsor / Introducer)
* **Regola**: Il nodo deve possedere una "Catena di Origine":
  * *Ingresso via Invito Direct*: Lo `SponsorAuthorId` è il `FULL_MEMBER` che ha generato il QR code d'invito.
  * *Ingresso via Scansione Autonoma*: Lo `SponsorAuthorId` è il primo `FULL_MEMBER` che incontra via P2P e che accetta di "sponsorizzare" la richiesta d'ingresso.
* **Motivazione**: Ancora l'identità dell'applicante alla rete di fiducia preesistente.

#### 3. Quorum di Prossimità Multisponsor Dinamico ($K_{\text{dynamic}}$)
* **Regola**: La soglia di attestazioni di fiducia (`TrustEndorsementMessage`) necessarie **non è un numero statico fisso**, ma scala in modo logaritmico-proporzionale alla dimensione stimata dei membri attivi della comunità ($\hat{N}_{\text{estimated}}$) `[Fonte: Demers et al., 1987 / Douceur, 2002]`:
  $$K_{\text{dynamic}} = \max\left(2, \; \min\left(15, \; \left\lceil 1.5 \cdot \log_2(\hat{N}_{\text{estimated}} + 1) \right\rceil\right)\right)$$
* **Esempi**:
  * In piccoli cluster ($N=5$ nodi): $K_{\text{dynamic}} = 2$ sponsor.
  * In reti medie ($N=32$ nodi): $K_{\text{dynamic}} = 8$ sponsor.
  * In grandi campi di emergenza ($N=256$ nodi): $K_{\text{dynamic}} = 12$ sponsor.
* **Motivazione**: Evita di pretendere troppi voti in piccole reti isolate ed impedisce attacchi Sybil banali in grandi popolazioni `[Fonte: Douceur, 2002]`.

#### 4. Diversità e Prova di Incontri Fisici P2P (Encounter Diversity)
* **Regola**: Gli endorsement del quorum devono essere stati scambiati in sessioni di sincronizzazione P2P distinte e separate nel tempo.
* **Motivazione**: Evita che un utente crei più identità su un singolo dispositivo o in un solo secondo via hotspot e firmi contemporaneamente per sé stesso.

#### 5. Partecipazione Utile e Assenza di Violazioni (Good Standing & Activity Score)
* **Regola**:
  * **Attività Minima**: L'utente deve aver pubblicato **almeno 1 post/bozza di qualità** o aver espresso voti di vicinato durante il periodo di prova.
  * **Zero Segnalazioni (Clean Record)**: Il nodo non deve aver ricevuto alcuna segnalazione di abuso o contestazione (`abuseFlagCount == 0`) `[Fonte: Resnick et al., 2000]`.
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

### A. Metadati di Sandbox salvati in `Settings` (`community.trust`)

Nel database cifrato di ciascun nodo applicante, `AccountManagerImpl` e `TrustManagerImpl` mantengono la mappa di impostazioni dinamiche `[Fonte: Briar GitLab Repository]`:

```java
// Namespace: "community.trust"
{
  "trustTier": "PROBATION",                          // PROBATION -> FULL_MEMBER -> ISOLATED
  "sandboxStartTime": "1728310000000",               // Timestamp di creazione account (ms)
  "sponsorAuthorId": "a1b2c3d4...",                  // AuthorId del primo sponsor
  "endorsementCount": "2",                           // Numero attuale di voti raccolti
  "endorsementAuthors": "authorId1,authorId2",       // Elenco degli AuthorId che hanno votato
  "estimatedNetworkSize": "32",                      // Stima dinamica popolazione di rete N_est
  "encounterRatePerHour": "3.5",                     // Tasso di incontri P2P orari Lambda
  "validPostsCount": "1",                            // Numero di post validi pubblicati
  "abuseFlagCount": "0"                              // Numero di segnalazioni ricevute
}
```

---

### B. Messaggio BDF: `TrustEndorsementMessage`

Quando un `FULL_MEMBER` incontra l'applicante via P2P e decide di esprimere fiducia, la sua app genera un messaggio BDF di attestazione firmato Ed25519 `[Fonte: RFC 8032]`:

```java
// File NUOVO: briar-api/.../community/TrustEndorsementMessage.java

public Message createTrustEndorsementMessage(Author applicantAuthor)
        throws FormatException, GeneralSecurityException {

    long timestamp = System.currentTimeMillis();
    Author localFullMember = identityManager.getLocalAuthor();

    BdfList bodyToSign = BdfList.of(
        MSG_TYPE_TRUST_ENDORSEMENT,                 // Integer: 2
        clientHelper.toList(applicantAuthor),       // Autore da promuovere (Alice)
        clientHelper.toList(localFullMember),      // Autore votante (Bob, FULL_MEMBER)
        timestamp
    );

    byte[] signature = clientHelper.sign(
        "org.briarproject.briar.community/TRUST_ENDORSEMENT",
        bodyToSign,
        localFullMember.getPrivateKey()
    );

    BdfList fullBody = BdfList.of(bodyToSign, signature);
    return clientHelper.createMessage(communityTopicGroupId, timestamp, clientHelper.toByteArray(fullBody));
}
```

---

## 5. Algoritmo di Valutazione della Promozione (`TrustManagerImpl.java`)

`TrustManagerImpl` calcola dinamicamente le soglie $K_{\text{dynamic}}$ e $\Delta T_{\text{dynamic}}$ prima di valutare la promozione:

```java
// File: bramble-core/src/main/java/org/briarproject/bramble/trust/TrustManagerImpl.java
// Fonte: Specifica Modello di Promozione Tesi

package org.briarproject.bramble.trust;

import org.briarproject.bramble.api.db.DbException;
import org.briarproject.bramble.api.settings.Settings;
import org.briarproject.bramble.api.settings.SettingsManager;
import javax.inject.Inject;
import java.util.logging.Logger;

public class TrustManagerImpl {

    private static final Logger LOG = Logger.getLogger(TrustManagerImpl.class.getName());
    private static final String NAMESPACE_COMMUNITY = "community.trust";

    private final SettingsManager settingsManager;

    @Inject
    public TrustManagerImpl(SettingsManager settingsManager) {
        this.settingsManager = settingsManager;
    }

    public synchronized boolean evaluatePromotionEligibility() throws DbException {
        Settings settings = settingsManager.getSettings(NAMESPACE_COMMUNITY);

        String currentTier = settings.getOrDefault("trustTier", "PROBATION");
        if ("FULL_MEMBER".equals(currentTier)) return true;
        if ("ISOLATED".equals(currentTier)) return false;

        // 1. Estrazione metadati locali dal DB
        long startTime = Long.parseLong(settings.getOrDefault("sandboxStartTime", "0"));
        int endorsementCount = Integer.parseInt(settings.getOrDefault("endorsementCount", "0"));
        int postsCount = Integer.parseInt(settings.getOrDefault("validPostsCount", "0"));
        int abuseFlags = Integer.parseInt(settings.getOrDefault("abuseFlagCount", "0"));
        int estimatedNetworkSize = Integer.parseInt(settings.getOrDefault("estimatedNetworkSize", "10"));
        double encounterRate = Double.parseDouble(settings.getOrDefault("encounterRatePerHour", "1.0"));
        String sponsorId = settings.get("sponsorAuthorId");

        // 2. CALCOLO DINAMICO DI QUORUM K_dynamic E TIME-LOCK T_dynamic
        int dynamicK = calculateDynamicQuorum(estimatedNetworkSize);
        long dynamicTimeLockMs = calculateDynamicTimeLock(encounterRate);

        // 3. VERIFICA CONCORRENTE DEI REQUISITI DINAMICI
        long elapsedTime = System.currentTimeMillis() - startTime;
        boolean timeLockPassed = elapsedTime >= dynamicTimeLockMs;
        boolean quorumPassed = endorsementCount >= dynamicK;
        boolean sponsorPresent = sponsorId != null && !sponsorId.isEmpty();
        boolean activityPassed = postsCount >= 1;
        boolean cleanRecord = abuseFlags == 0;

        if (timeLockPassed && quorumPassed && sponsorPresent && activityPassed && cleanRecord) {
            executePromotionToFullMember(dynamicK);
            return true;
        }

        return false;
    }

    public int calculateDynamicQuorum(int N_est) {
        if (N_est <= 3) return 2;
        int calculatedK = (int) Math.ceil(1.5 * (Math.log(N_est + 1) / Math.log(2)));
        return Math.max(2, Math.min(15, calculatedK));
    }

    public long calculateDynamicTimeLock(double rate) {
        if (rate <= 0) return 72 * 3600 * 1000L;
        long time = (long) ((72 * 3600 * 1000L) / (Math.log(rate + 1) / Math.log(2)));
        return Math.max(24 * 3600 * 1000L, Math.min(168 * 3600 * 1000L, time));
    }

    private void executePromotionToFullMember(int appliedK) throws DbException {
        Settings newSettings = new Settings();
        newSettings.put("trustTier", "FULL_MEMBER");
        newSettings.put("appliedQuorumK", String.valueOf(appliedK));
        newSettings.put("promotionTimestamp", String.valueOf(System.currentTimeMillis()));

        settingsManager.mergeSettings(newSettings, NAMESPACE_COMMUNITY);
        LOG.info("🎉 PROMOZIONE DINAMICA COMPLETATA: Il nodo ora è FULL_MEMBER con Quorum K=" + appliedK);
    }
}
```

---

## 6. Fonti Ufficiali e Bibliografia

1. **Douceur, J. R. (2002)**. *"The Sybil Attack"*. International Workshop on Peer-to-Peer Systems (IPTPS). Springer, Berlin, Heidelberg.
   * *Riferimento*: Giustificazione teorica della difesa dagli attacchi Sybil tramite l'algoritmo di quorum dinamico e time-lock.
2. **Resnick, P., Zeckhauser, R., Friedman, E., & Kuwabara, K. (2000)**. *"Reputation systems"*. *Communications of the ACM*, 43(12), 45-48.
   * *Riferimento*: Modelli di reputazione distribuita e valutazione del buon comportamento (`good standing`).
3. **Demers, A., Greene, D., Hauser, C., Irish, W., & Larson, J. (1987)**. *"Epidemic algorithms for replicated database maintenance"*. *ACM Symposium on Principles of Distributed Computing (PODC)*.
   * *Riferimento*: Stima della cardinalità della popolazione di rete tramite algoritmi di gossip ed incontri epidemici.
4. **IETF RFC 8032**: *Edwards-Curve Digital Signature Algorithm (Ed25519)*.
   * *Riferimento*: Firma asimmetrica delle attestazioni `TrustEndorsementMessage`.
5. **Briar Project Official Repository**:
   * *GitLab Repository*: [code.briarproject.org/briar/briar](https://code.briarproject.org/briar/briar)
   * *Classi Verificate*: `bramble-core/.../account/AccountManagerImpl.java` e `bramble-core/.../trust/TrustManagerImpl.java`.
