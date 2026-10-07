# Documentazione Dettagliata: Flusso di Comunicazione dell'Onboarding e Quorum di Prossimità (P2P DTN)

Questo documento descrive nel dettaglio il **flusso temporale**, la **sequenza dei messaggi**, le **strutture BDF** e le **chiamate di codice** necessarie per gestire l'onboarding di un **Nuovo Membro (Applicante in Sandbox)**, la raccolta degli endorsement da parte dei **Membri Esistenti (`FULL_MEMBER`)** e la **valutazione del Quorum di Prossimità Multicriterio**.

---

## 1. Diagramma di Sequenza Temporale (Flusso a Quorum)

```mermaid
sequenceDiagram
    autonumber
    actor Alice as Alice (Nuovo Membro - Sandbox)
    participant Alice_DB as DB Cifrato di Alice
    participant BSS as Bramble Sync Protocol (BTP P2P)
    participant Bob_DB as DB Cifrato di Bob
    actor Bob as Bob (Membro Esistente 1 - FULL_MEMBER)
    actor Charlie as Charlie (Membro Esistente 2 - FULL_MEMBER)

    
    Note over Alice, Alice_DB: FASE 1: Creazione Richiesta e Inizializzazione Sandbox
    Alice->>Alice: 1. Compila OnboardingRequest (Pseudonimo "Alice", Topics)
    Alice->>Alice_DB: 2. AccountManagerImpl.createAccount() -> trustTier = PROBATION, sandboxStartTime = NOW
    Alice->>Alice_DB: 3. ClientHelper.addLocalMessage(OnboardingRequest, shared=true)

    
    Note over Alice, Bob: FASE 2: Incontro P2P #1 con Bob (Primo Endorsement / Sponsor)
    Alice<<->>Bob: Handshake BSS (OFFER -> REQUEST -> MESSAGE -> ACK)
    Bob->>Bob_DB: CommunityMessageHook valida firma Ed25519 di Alice
    Bob->>Bob: Bob clicca "Esprimi Fiducia" in UI
    Bob->>Bob_DB: Genera TrustEndorsementMessage(Alice, Bob) e firma con chiave Bob
    Bob<<->>Alice: BSS trasferisce TrustEndorsementMessage ad Alice
    Alice->>Alice_DB: Incrementa endorsementCount = 1, imposta sponsorAuthorId = Bob

   
    Note over Alice, Charlie: FASE 3: Periodo di Osservazione (72h) & Incontri P2P Successivi

    Note over Alice: Alice pubblica 1 Post valido (validPostsCount = 1, abuseFlagCount = 0)

    Note over Alice, Charlie: Trascorrono 3 giorni (72 ore) dall'onboarding

    Alice<<->>Charlie: Incontro P2P #2: Charlie invia TrustEndorsementMessage(Alice, Charlie)
    Alice->>Alice_DB: Incrementa endorsementCount = 2 (Raccolti K=3 endorsement totali)

    Note over Alice, Alice_DB: FASE 4: Valutazione Quorum e Promozione Locale
    Alice->>Alice_DB: TrustManagerImpl.evaluatePromotionEligibility()
    Note over Alice_DB: Verificati TUTTI i 5 requisiti (72h, K=3 Endorsement, Sponsor, 1 Post, 0 Abusi)
    Alice_DB->>Alice_DB: TrustManagerImpl.executePromotionToFullMember() -> trustTier = FULL_MEMBER
    Alice-->>Alice: Notifica UI: "Sandbox Disattivata! Ora sei Membro Pieno della Comunità."
```

---

## 2. Flusso Temporale Dettagliato e Chiamate di Codice

---

### FASE 1: Creazione della Richiesta e Inizializzazione Sandbox (Alice)

Quando Alice completa il wizard nell'app nativa, la sua app genera la richiesta di onboarding e la memorizza nel proprio database cifrato avviando il contatore temporale di Sandbox.

#### Chiamate di Codice:

1. **Inizializzazione Profilo Sandbox in `AccountManagerImpl.java`**:
   ```java
   // File: bramble-core/.../account/AccountManagerImpl.java

   private void initializeCommunitySandboxProfile(AuthorId authorId, List<String> topics, @Nullable String token) {
       try {
           Settings settings = new Settings();
           long now = System.currentTimeMillis();
           
           settings.put("trustTier", "PROBATION");
           settings.put("sandboxStartTime", String.valueOf(now)); // Inizio vincolo 72h
           settings.put("endorsementCount", "0");
           settings.put("validPostsCount", "0");
           settings.put("abuseFlagCount", "0");
           settings.put("maxPostsPerDay", "1");
           
           if (token != null) settings.put("onboardingToken", token);
           if (topics != null) settings.put("subscribedTopics", String.join(",", topics));
           
           settingsManager.mergeSettings(settings, "community.trust");
       } catch (DbException e) {
           LOG.log(WARNING, "Errore inizializzazione profilo Sandbox", e);
       }
   }
   ```

2. **Creazione della `OnboardingRequest` in BDF**:
   ```java
   // File: briar-core/.../community/CommunityOnboardingManagerImpl.java

   public Message createOnboardingRequestMessage(Author applicantAuthor, List<String> topics)
           throws FormatException, GeneralSecurityException {

       long timestamp = System.currentTimeMillis();

       BdfList bodyToSign = BdfList.of(
           MSG_TYPE_ONBOARDING_REQUEST,          // Integer: 0
           clientHelper.toList(applicantAuthor),  // BdfList dell'autore applicante (Alice)
           BdfList.of(topics),                   // Topics scelti
           timestamp
       );

       byte[] signature = clientHelper.sign(
           "org.briarproject.briar.community/ONBOARDING_REQUEST",
           bodyToSign,
           identityManager.getLocalAuthor().getPrivateKey()
       );

       BdfList fullBody = BdfList.of(bodyToSign, signature);
       return clientHelper.createMessage(communityTopicGroupId, timestamp, clientHelper.toByteArray(fullBody));
   }
   ```

---

### FASE 2: Trasporto P2P BSS ed Emissione degli Endorsement (`TrustEndorsementMessage`)

Non appena lo smartphone di Alice entra nel raggio di un membro della comunità (Bob), le sessioni BSS (`IncomingSession` e `DuplexOutgoingSession`) scambiano i record.

#### Chiamate di Codice per l'Invio dell'Endorsement (Bob):

1. **Generazione del Messaggio di Attestazione di Fiducia (`TrustEndorsementMessage`)**:
   ```java
   // File: briar-core/.../community/CommunityOnboardingManagerImpl.java

   public void endorseApplicantNode(Author applicantAuthor) 
           throws DbException, FormatException, GeneralSecurityException {

       long timestamp = System.currentTimeMillis();
       Author localFullMember = identityManager.getLocalAuthor(); // Bob (FULL_MEMBER)

       BdfList bodyToSign = BdfList.of(
           MSG_TYPE_TRUST_ENDORSEMENT,                 // Integer: 2
           clientHelper.toList(applicantAuthor),       // Alice (Applicante)
           clientHelper.toList(localFullMember),      // Bob (Firmante FULL_MEMBER)
           timestamp
       );

       byte[] signature = clientHelper.sign(
           "org.briarproject.briar.community/TRUST_ENDORSEMENT",
           bodyToSign,
           localFullMember.getPrivateKey()
       );

       BdfList fullResponseBody = BdfList.of(bodyToSign, signature);
       Message endorsementMessage = clientHelper.createMessage(
           communityTopicGroupId,
           timestamp,
           clientHelper.toByteArray(fullResponseBody)
       );

       db.runInTransaction(false, txn -> {
           clientHelper.addLocalMessage(txn, endorsementMessage, new BdfDictionary(), true);
       });
   }
   ```

---

### FASE 3: Ricezione degli Endorsement e Incremento Quorum (Alice)

Quando Alice riceve un `TrustEndorsementMessage` via BSS, l'hook di messaggio elabora la firma ed aggiorna i metadati locales del quorum.

#### Chiamate di Codice:

1. **Elaborazione dell'Endorsement in `CommunityMessageHook.java`**:
   ```java
   // File: briar-core/.../community/CommunityMessageHook.java

   if (msgType == MSG_TYPE_TRUST_ENDORSEMENT) {
       BdfList bodyToSign = body.getList(0);
       byte[] signature = body.getRaw(1);

       Author approver = clientHelper.parseAndValidateAuthor(bodyToSign.getList(2));

       // 1. Verifica che l'approvatore sia riconosciuto come FULL_MEMBER
       if (trustManager.getTrustTier(approver.getId()) == TrustTier.FULL_MEMBER) {

           Author applicant = clientHelper.parseAndValidateAuthor(bodyToSign.getList(1));

           // 2. Se l'endorsement riguarda l'utente locale (Alice)
           if (applicant.getId().equals(identityManager.getLocalAuthor().getId())) {
               
               // 3. Registra l'endorsement nel DB ed esegui la verifica del quorum
               trustManager.registerIncomingEndorsement(approver.getId());
           }
       }
   }
   ```

2. **Registrazione dell'Endorsement in `TrustManagerImpl.java`**:
   ```java
   // File: bramble-core/.../trust/TrustManagerImpl.java

   public synchronized void registerIncomingEndorsement(AuthorId approverId) throws DbException {
       Settings settings = settingsManager.getSettings("community.trust");

       String authors = settings.getOrDefault("endorsementAuthors", "");
       List<String> authorList = new ArrayList<>(Arrays.asList(authors.split(",")));

       // Previene voti doppi dallo stesso membro
       if (!authorList.contains(approverId.getBytes())) {
           authorList.add(approverId.getBytes());
           int newCount = authorList.size();

           Settings update = new Settings();
           update.put("endorsementCount", String.valueOf(newCount));
           update.put("endorsementAuthors", String.join(",", authorList));

           // Imposta lo sponsor iniziale se non ancora presente
           if (settings.get("sponsorAuthorId") == null) {
               update.put("sponsorAuthorId", approverId.getBytes());
           }

           settingsManager.mergeSettings(update, "community.trust");

           // VALUTAZIONE ISTANTANEA DEL QUORUM E DEI REQUISITI
           evaluatePromotionEligibility();
       }
   }
   ```

---

### FASE 4: Valutazione del Quorum Multicriterio e Promozione

Quando scade il timer delle 72 ore o quando arriva un nuovo endorsement, la funzione `evaluatePromotionEligibility()` verifica che tutti i 5 requisiti siano soddisfatti prima di passare a `FULL_MEMBER`.

```java
// File: bramble-core/.../trust/TrustManagerImpl.java

public synchronized boolean evaluatePromotionEligibility() throws DbException {
    Settings settings = settingsManager.getSettings("community.trust");

    if ("FULL_MEMBER".equals(settings.get("trustTier"))) return true;

    long startTime = Long.parseLong(settings.getOrDefault("sandboxStartTime", "0"));
    int endorsementCount = Integer.parseInt(settings.getOrDefault("endorsementCount", "0"));
    int postsCount = Integer.parseInt(settings.getOrDefault("validPostsCount", "0"));
    int abuseFlags = Integer.parseInt(settings.getOrDefault("abuseFlagCount", "0"));
    String sponsorId = settings.get("sponsorAuthorId");

    // VERIFICA DEI 5 REQUISITI CONCORRENTI
    boolean timeLockPassed = (System.currentTimeMillis() - startTime) >= 3 * 86_400_000L; // 72h
    boolean quorumPassed = endorsementCount >= 3;                                          // K=3
    boolean sponsorPresent = sponsorId != null && !sponsorId.isEmpty();
    boolean activityPassed = postsCount >= 1;
    boolean cleanRecord = abuseFlags == 0;

    if (timeLockPassed && quorumPassed && sponsorPresent && activityPassed && cleanRecord) {
        // PROMOZIONE!
        executePromotionToFullMember();
        return true;
    }

    return false;
}
```

---

## 3. Vantaggi del Flusso Multicriterio

1. **Eliminazione dell'Approvazione ad Arbitrio Singolo**: Un singolo utente compromesso non può promuovere autonomamente un nodo malevole.
2. **Protezione dalle Infiltrazioni Lampo**: Il vincolo temporale di 72 ore unito alla necessità di incontrare 3 membri distinti neutralizza gli attacchi Sybil.
3. **Piena Integrazione Native Bramble**: Sfrutta la messaggistica BDF, l'EventBus locale e le `Settings` cifrate per garantire resilienza senza dipendenze esterne.
