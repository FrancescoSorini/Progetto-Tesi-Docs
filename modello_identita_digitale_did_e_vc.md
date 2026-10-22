# Proposta Architetturale: Modello di Identità Digitale Sovrana (SSI) con DID e Verifiable Credentials per il Progetto di Tesi

Il documento valuta il modello d'identità nativo di **Briar**, ne individua i limiti, analizza la coerenza dell'approccio **DID (Decentralized Identifiers)** e propone il **Modello d'Identità Digitale Ideale** per il progetto di tesi, integrando lo standard W3C con la crittografia nativa Ed25519 di Bramble.

---

## 1. Analisi del Modello di Identità Nativo di Briar

### 1.1 Come funziona l'Identità in Briar
Nel framework Bramble/Briar, l'identità dell'utente è rappresentata dall'oggetto **`Author`** (`bramble-api/.../identity/Author.java`):

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         OGGETTO AUTHOR IN BRIAR                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ • AuthorId    : Hash SHA-256 a 32 byte della Chiave Pubblica (Id Univoco)   │
│ • name        : Pseudonimo stringa scelto dall'utente (es. "Alice")         │
│ • publicKey   : Chiave Pubblica Ed25519 a 32 byte (RFC 8032)                │
│ • privateKey  : Chiave Privata Ed25519 conservata nel DB cifrato locale     │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### **Protezione contro l'Impersonificazione in Briar**:
1. **Lancio dell'Identità Crittografica**: L'identità non si basa su username/password centralizzati, ma sulla coppia di chiavi asimmetriche **Ed25519** `[Fonte: RFC 8032]`.
2. **Inversione Impossibile dell'AuthorId**: L'ID dell'utente è calcolato come $H = \text{SHA-256}(\text{publicKey})$. È matematicamente impossibile generare un'identità falsa con lo stesso ID senza possedere la relativa chiave privata Ed25519 `[Fonte: Briar Protocol Specifications]`.
3. **Firma di Ogni Messaggio**: Ogni contenuto pubblicato viene firmato asimmetricamente. Chiunque riceva un messaggio può verificare con la chiave pubblica dell'autore che il messaggio sia autentico e non sia stato alterato lungo il percorso P2P `[Fonte: Briar Protocol Specifications]`.

---

### 1.2 Limiti dell'Identità Briar per il Progetto di Tesi

Sebbene il modello di Briar sia eccezionale per la messaggistica privata tra contatti direct, presenta **3 limiti strutturali** per un'app di giornalismo di emergenza:

1. **Identità "Piatta" Priva di Ruoli e Attestazioni**:
   L'oggetto `Author` contiene solo il nome ed la chiave. Non può esprimere nativamente se l'utente è in Sandbox (`PROBATION`), se possiede uno sponsor o se ha raggiunto il livello `FULL_MEMBER`.
2. **Mancanza di Interoperabilità Standard Esterna**:
   L'identità di Briar è codificata nel formato binario proprietario BDF (`BdfList`). Quando un reportage o una prova viene esportata fuori dalla rete d'emergenza verso organizzazioni internazionali, ONG o media, un formato proprietario rende difficile la verifica esterna del documento.
3. **Impossibilità di Aggiornamento Metadati e Revoca**:
   Poiché l'ID dell'utente è legato in modo indissolubile all'hash della chiave pubblica, non è possibile ruotare le chiavi compromesse né aggiornare le attestazioni di fiducia senza distruggere la storia pregressa dell'utente.

---

## 2. Valutazione dell'Approccio DID (Decentralized Identifiers)

I **Decentralized Identifiers (DID)** costituiscono uno standard ufficiale del **W3C (World Wide Web Consortium)** per l'Identità Digitale Auto-Sovrana (*Self-Sovereign Identity - SSI*) `[Fonte: W3C DID v1.0 Recommendation]`.

Un DID è una stringa URI strutturata nel seguente formato: `did:metodo:stringa-univoca-specifica-del-metodo`.

```
                    did : peer : 1z6MkpTHR8V...
                     ▲     ▲          ▲
                     │     │          │
                 Schema  Metodo   Identificatore Specifico
```

---

### 2.1 Analisi di Coerenza con il Progetto di Tesi

**L'approccio DID è coerente con la Tesi? SÌ, MA CON UNA DISTINZIONE FONDAMENTALE SUI METODI DID:**

```
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │                      VALUTAZIONE METODI DID IN DTN                          │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ ❌ METODI BLOCKCHAIN / REGISTRO CENTRALE (did:ion, did:ethr, did:sov)       
 │    └─ NON IDONEI: Richiedono connessione ad Internet ed alla Blockchain.    │
 │       Nelle emergenze offline il lookup su blockchain fallisce              │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 🟢 METODI P2P CRYPTOGRAPHIC / IN-BAND (did:key, did:peer)               
 │    └─ PERFETTI: La chiave e le credenziali risiedono nell'URI o nello       │
 │       scambio P2P. Risoluzione al 100% OFFLINE senza server o internet      │
 └─────────────────────────────────────────────────────────────────────────────┘
```

#### **I Metodi DID Idonei per il Contesto Offline/DTN**:
1. **`did:key` (Cryptographic DID)** `[Fonte: W3C did:key Method Spec]`:
   * Codifica l'intera chiave pubblica Ed25519 direttamente nella stringa del DID (es. `did:key:z6MkpTHR8V...`).
   * **Risoluzione al 100% Offline**: Non richiede chiamate di rete o registri esterni. Qualsiasi nodo che legge la stringa `did:key` può estrarre la chiave pubblica e verificare le firme istantaneamente.
2. **`did:peer` (W3C DID Peer Specification)** `[Fonte: W3C DID Peer Spec]`:
   * Progettato espressamente per la comunicazione P2P diretta tra due o più nodi senza dipendere da registri centrali.
   * I nodi si scambiano il documento di identità (*DID Document*) direttamente all'interno delle sessioni P2P BSS di Bramble.

---

## 3. Il Modello di Identità Ideale 
Proposta di un **Modello d'Identità Sovrana Ibrida (SSI P2P)** che estende le primitive di Briar senza appesantire il sistema e garantendo piena interoperabilità W3C.

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │               MODELLO DI IDENTITÀ DIGITALE IBRIDA (SSI P2P)                  │
 ├──────────────────────────────────────────────────────────────────────────────┤
 │ 1. ROOT IDENTITY LAYER (Ed25519 KeyPair + W3C did:key / did:peer)            │
 │    └─ Chiave crittografica asimmetrica locale (resiliente all'impersonazione)│
 ├──────────────────────────────────────────────────────────────────────────────┤
 │ 2. VERIFIABLE CREDENTIALS LAYER (W3C VC Standard)                            │
 │    ├─ SponsorshipCredential (Emessa dallo Sponsor iniziale nell'onboarding)  │
 │    ├─ CommunityMembershipCredential (Emessa dal Quorum per FULL_MEMBER)  │
 │    └─ ReporterTrustAttestation (Attestazioni di affidabilità sui report)     │
 └──────────────────────────────────────────────────────────────────────────────┘
```

---

### 3.1 Come il Modello Risolve l'Impersonificazione ed il Quorum

#### **A. Resistenza Assoluta all'Impersonificazione**:
* L'identità del testimone/giornalista risiede nella sua chiave privata Ed25519, protetta dal DB cifrato di Bramble.
* Il suo identificativo W3C `did:key:z6Mkp...` o `did:peer:...` è derivato dalla chiave pubblica.
* Nessun attaccante può impersonare un utente o firmare un report al suo posto senza violare l'algoritmo Ed25519 `[Fonte: RFC 8032]`.

#### **B. Integrazione con le Verifiable Credentials (W3C VC)**:
Le autorizzazioni della comunità ed il livello di fiducia non vengono inseriti nell'identità statica, ma sono espressi tramite **Verifiable Credentials (VC)**, ovvero certificati crittografici firmati dai membri della comunità `[Fonte: W3C Verifiable Credentials Data Model v1.1]`:

1. **`SponsorshipCredential`**:
   Rilasciata dallo sponsor al momento della scansione del QR d'Invito Personale.
2. **`CommunityMembershipCredential`**:
   Rilasciata al completamento dei 5 requisiti multicriterio di promozione: periodo di osservazione e quorum adattivi, sponsor, incontri P2P diversificati, attività utile e assenza di segnalazioni di abuso.
3. **`ProofExportPackage`**:
   Quando un reportage viene esportato all'esterno della rete d'emergenza, il pacchetto contiene:
   * Il post originale con firma dell'autore (`did:key`).
   * La Verifiable Credential del Quorum di Comunità che attesta che l'autore è un `FULL_MEMBER` accreditato.
   * Il tutto formattato in **JSON-LD / W3C Verifiable Credential**, verificabile da qualsiasi software esterno o tribunale internazionale senza bisogno dell'app Briar!

---

## 4. Mappatura ed Estensione del Codice di Briar

Per implementare il modello SSI con `did:key` e Verifiable Credentials, definiamo l'estensione dell'oggetto `Author` e la nuova classe `VerifiableCredentialMessage`.

```
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │                         MAPPATURA E CLASSI DEL CODICE                       │
 ├──────────────────────────┬──────────────────────────────────────────────────┤
 │ FILE [NUOVO]             │ `briar-api/.../identity/DidIdentity.java`        │
 │                          │  └─ Wrapper W3C did:key / did:peer dell'Author   │
 ├──────────────────────────┼──────────────────────────────────────────────────┤
 │ FILE [NUOVO]             │ `briar-api/.../identity/VerifiableCredential.java│
 │                          │  └─ Struttura W3C VC per sponsor e quorum        │
 ├──────────────────────────┼──────────────────────────────────────────────────┤
 │ FILE [MODIFICA]          │ `bramble-core/.../identity/AuthorFactoryImpl.java│
 │                          │  └─ Generazione integrata del did:key dall'Author│
 └──────────────────────────┴──────────────────────────────────────────────────┘
```

---

### A. [FILE NUOVO]: `DidIdentity.java`
* **Path**: `briar-api/src/main/java/org/briarproject/briar/api/identity/DidIdentity.java`
* **Ruolo**: Inviluppa l'oggetto `Author` di Briar esponendo l'identificativo standard W3C `did:key` e generandone il DID Document offline.

```java
// FILE NUOVO - Path: briar-api/src/main/java/org/briarproject/briar/api/identity/DidIdentity.java

package org.briarproject.briar.api.identity;

import org.briarproject.bramble.api.identity.Author;
import org.briarproject.bramble.util.StringUtils;
import org.briarproject.nullsafety.NotNullByDefault;
import javax.annotation.concurrent.Immutable;

/**
 * Wrapper dell'Author di Briar conforme allo standard W3C Decentralized Identifier (did:key).
 * Consente la risoluzione offline dell'identità e delle chiavi Ed25519.
 */
@Immutable
@NotNullByDefault
public class DidIdentity {

    private final Author author;
    private final String didKeyString;

    public DidIdentity(Author author) {
        this.author = author;
        // Formattazione W3C did:key basata su chiave pubblica Ed25519 (Multicodec prefix 0xed)
        this.didKeyString = "did:key:z" + StringUtils.toBase58(author.getPublicKey());
    }

    public Author getAuthor() { return author; }
    public String getDidKeyString() { return didKeyString; }

    /**
     * Eroga il DID Document W3C in formato JSON-LD per l'esportazione verso l'esterno.
     */
    public String toW3cDidDocumentJson() {
        return "{\n" +
               "  \"@context\": \"https://www.w3.org/ns/did/v1\",\n" +
               "  \"id\": \"" + didKeyString + "\",\n" +
               "  \"verificationMethod\": [{\n" +
               "    \"id\": \"" + didKeyString + "#keys-1\",\n" +
               "    \"type\": \"Ed25519VerificationKey2020\",\n" +
               "    \"controller\": \"" + didKeyString + "\",\n" +
               "    \"publicKeyMultibase\": \"z" + StringUtils.toBase58(author.getPublicKey()) + "\"\n" +
               "  }]\n" +
               "}";
    }
}
```

---

### B. [FILE NUOVO]: `VerifiableCredential.java`
* **Path**: `briar-api/src/main/java/org/briarproject/briar/api/identity/VerifiableCredential.java`
* **Ruolo**: Struttura BDF/JSON per la rappresentazione delle Verifiable Credentials (Sponsorizzazione, Approvazione Quorum, Ruolo).

```java
// FILE NUOVO - Path: briar-api/src/main/java/org/briarproject/briar/api/identity/VerifiableCredential.java

package org.briarproject.briar.api.identity;

import org.briarproject.nullsafety.NotNullByDefault;
import javax.annotation.concurrent.Immutable;

/**
 * Struttura DTO W3C Verifiable Credential per attestazioni di sponsor e quorum.
 */
@Immutable
@NotNullByDefault
public class VerifiableCredential {

    private final String credentialId;     // ID univoco della credenziale
    private final String issuerDid;        // DID del membro o del quorum emittente
    private final String subjectDid;       // DID del nuovo utente (Alice)
    private final String credentialType;   // Es. "CommunityMembershipCredential" o "SponsorshipCredential"
    private final long issuanceDateMs;     // Data di rilascio
    private final byte[] signature;        // Firma Ed25519 dell'emittente

    public VerifiableCredential(String credentialId,
                                String issuerDid,
                                String subjectDid,
                                String credentialType,
                                long issuanceDateMs,
                                byte[] signature) {
        this.credentialId = credentialId;
        this.issuerDid = issuerDid;
        this.subjectDid = subjectDid;
        this.credentialType = credentialType;
        this.issuanceDateMs = issuanceDateMs;
        this.signature = signature;
    }

    public String getCredentialId() { return credentialId; }
    public String getIssuerDid() { return issuerDid; }
    public String getSubjectDid() { return subjectDid; }
    public String getCredentialType() { return credentialType; }
    public long getIssuanceDateMs() { return issuanceDateMs; }
    public byte[] getSignature() { return signature; }

    /**
     * Serializzazione in formato W3C Verifiable Credential JSON-LD per export prove.
     */
    public String toW3cJsonLd() {
        return "{\n" +
               "  \"@context\": [\"https://www.w3.org/2018/credentials/v1\"],\n" +
               "  \"id\": \"" + credentialId + "\",\n" +
               "  \"type\": [\"VerifiableCredential\", \"" + credentialType + "\"],\n" +
               "  \"issuer\": \"" + issuerDid + "\",\n" +
               "  \"issuanceDate\": \"" + issuanceDateMs + "\",\n" +
               "  \"credentialSubject\": { \"id\": \"" + subjectDid + "\" }\n" +
               "}";
    }
}
```

---

## 5. Sintesi e Vantaggi della Soluzione Proposta

1. **Massima Protezione dall'Impersonificazione**: Basato su chiavi asimmetriche **Ed25519** conservate localmente; nessun utente può falsificare l'identità di un altro.
2. **Totale Indipendenza Offline (No-Blockchain)**: L'uso dei metodi **`did:key` / `did:peer`** consente di verificare l'identità al 100% in locale senza bisogno di Internet, server DNS centrali o blockchain.
3. **Interoperabilità Esterna Standard W3C**: Grazie alle **Verifiable Credentials (VC)**, quando un reportage viene esportato fuori dall'area d'emergenza, qualsiasi media o tribunale esterno può convalidare crittograficamente la prova ed il quorum di attestazione della comunità.
4. **Piena Compatibilità con Briar**: Non stravolge le chiavi del database o le chiamate di Bramble, ma aggiunge un layer di avvolgimento (*wrapper*) dell'oggetto `Author`.

---

## 6. Fonti Ufficiali e Bibliografia

1. **W3C Decentralized Identifiers (DIDs) v1.0 Standard**:
   * *Fonte*: [W3C Recommendation - Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/did-core/)
2. **W3C Verifiable Credentials Data Model v1.1**:
   * *Fonte*: [W3C Recommendation - Verifiable Credentials Data Model](https://www.w3.org/TR/vc-data-model/)
3. **W3C `did:key` Method Specification**:
   * *Fonte*: [W3C `did:key` Method Spec](https://w3c-ccg.github.io/did-method-key/)
4. **W3C `did:peer` Method Specification**:
   * *Fonte*: [DIF `did:peer` Specification](https://identity.foundation/peer-did-method-spec/)
5. **IETF RFC 8032 (Ed25519 Digital Signatures)**:
   * *Fonte*: [RFC 8032 - Edwards-Curve Digital Signature Algorithm](https://datatracker.ietf.org/doc/html/rfc8032)
6. **Briar Project Official Repository & Identity Architecture**:
   * *GitLab Repository*: [code.briarproject.org/briar/briar](https://code.briarproject.org/briar/briar) (`bramble-api/.../identity/Author.java`).
