# Modulo 1: Accesso via Browser ed Onboarding Progressivo (Briar vs Tesi)

Questo documento analizza in dettaglio la riprogettazione del **meccanismo di accesso e onboarding** di Briar, partendo dalle specifiche di **Statement Progetto Tesi - v1.1.md** e **Limiti HTML.md**.

---

## 1. Architettura dell'Accesso: Browser Web vs App Nativa

---

## 2. Mappatura dei File di Briar Coinvolti

| File / Componente | Posizione nel Progetto Briar | Natura Modifica |
| :--- | :--- | :---: |
| **`AccountManagerImpl.java`** | `bramble-core/.../account/AccountManagerImpl.java` | **MODIFICA** |
| **`SetupViewModel.java`** | `briar-android/.../account/SetupViewModel.java` | **MODIFICA** |
| **`AuthorNameFragment.java`** | `briar-android/.../account/AuthorNameFragment.java` | **MODIFICA** |
| **`LinkExchangeFragment.java`** | `briar-android/.../contact/add/remote/LinkExchangeFragment.java` | **MODIFICA** |
| **`EmbeddedWebGateway.kt`** | `briar-core/.../web/EmbeddedWebGateway.kt` | **NUOVO** |
| **`CommunityOnboardingManager.java`** | `briar-core/.../community/CommunityOnboardingManager.java` | **NUOVO** |
| **`OnboardingRequest.java`** | `briar-api/.../community/OnboardingRequest.java` | **NUOVO** |

---

## 3. Documentazione File per File: Briar Originario vs Briar Modificato

---

### A. `AccountManagerImpl.java`

#### 🔴 Come funziona in Briar originariamente:
Il metodo `createAccount(name, password)` genera l'identità crittografica principale dell'utente (`IdentityManager.createIdentity(name)`), crea la chiave del database cifrato (`db.key`) e la salva cifrata con la password specificata. L'account creato è immediatamente "pieno" e non ha restrizioni di ruolo, rate-limiting o sandbox.

#### 🟢 Come funziona dopo le modifiche per la Tesi:
Il metodo `createAccount` accetta ora anche i parametri di onboarding (`List<String> selectedTopics`, `String onboardingInviteToken`). All'atto della creazione dell'account:
1. Imposta lo stato iniziale dell'account su **`TrustTier.PROBATION`**.
2. Salva nei metadati locali la lista dei topic di interesse selezionati dall'utente.
3. Inizializza il contatore di rate-limiting (max 1 post/giorno).

```java
// File: bramble-core/src/main/java/org/briarproject/bramble/account/AccountManagerImpl.java

package org.briarproject.bramble.account;

import org.briarproject.bramble.api.account.AccountManager;
import org.briarproject.bramble.api.crypto.CryptoComponent;
import org.briarproject.bramble.api.crypto.SecretKey;
import org.briarproject.bramble.api.db.DatabaseConfig;
import org.briarproject.bramble.api.db.DbException;
import org.briarproject.bramble.api.identity.AuthorId;
import org.briarproject.bramble.api.identity.Identity;
import org.briarproject.bramble.api.identity.IdentityManager;
import org.briarproject.bramble.api.settings.Settings;
import org.briarproject.bramble.api.settings.SettingsManager;
import org.briarproject.nullsafety.MethodsNotNullByDefault;
import org.briarproject.nullsafety.ParametersNotNullByDefault;

import java.util.List;
import java.util.logging.Logger;

import javax.annotation.Nullable;
import javax.inject.Inject;

import static java.util.logging.Level.WARNING;

@MethodsNotNullByDefault
@ParametersNotNullByDefault
class AccountManagerImpl implements AccountManager {

    private static final Logger LOG = Logger.getLogger(AccountManagerImpl.class.getName());

    // Namespace del database di Bramble per il controllo della fiducia di comunità
    private static final String SETTINGS_NAMESPACE_COMMUNITY = "community.trust";

    private final DatabaseConfig databaseConfig;
    private final CryptoComponent crypto;
    private final IdentityManager identityManager;
    private final SettingsManager settingsManager; // [INIEZIONE DIPENDENZA]

    @Inject
    AccountManagerImpl(DatabaseConfig databaseConfig, 
                       CryptoComponent crypto,
                       IdentityManager identityManager, 
                       SettingsManager settingsManager) {
        this.databaseConfig = databaseConfig;
        this.crypto = crypto;
        this.identityManager = identityManager;
        this.settingsManager = settingsManager;
    }

    @Override
    public boolean createAccount(String name, String password, 
                                 List<String> selectedTopics, 
                                 @Nullable String onboardingToken) {
        synchronized (stateChangeLock) {
            if (hasDatabaseKey())
                throw new AssertionError("Already have a database key");

            // 1. Creazione dell'Identità Crittografica (Author Ed25519)
            Identity identity = identityManager.createIdentity(name);
            identityManager.registerIdentity(identity);

            // 2. Generazione della Chiave Simmetrica del Database Cifrato
            SecretKey key = crypto.generateSecretKey();
            if (!encryptAndStoreDatabaseKey(key, password)) return false;

            databaseKey = key;

            // 3. [INIZIALIZZAZIONE PROFILO SANDBOX PER LA TESI]
            initializeCommunitySandboxProfile(identity.getAuthor().getId(), selectedTopics, onboardingToken);

            return true;
        }
    }

    /**
     * Inizializza le impostazioni della Sandbox (PROBATION) nel DB cifrato di Bramble.
     */
    private void initializeCommunitySandboxProfile(AuthorId authorId, List<String> topics, @Nullable String token) {
        try {
            Settings settings = new Settings();
            
            // A. Imposta lo stato iniziale del nodo su Sandbox (PROBATION)
            settings.put("trustTier", "PROBATION");
            
            // B. Configura la quota iniziale di Rate Limiting (1 post al giorno)
            settings.put("maxPostsPerDay", "1");
            settings.put("postsToday", "0");
            settings.put("lastPostTimestamp", String.valueOf(System.currentTimeMillis()));
            
            // C. Salva i Topic di interesse scelti dall'utente durante l'onboarding
            if (topics != null && !topics.isEmpty()) {
                settings.put("subscribedTopics", String.join(",", topics));
            } else {
                settings.put("subscribedTopics", "");
            }
            
            // D. Registra il token di invito/onboarding (se proveniente da Web Landing o QR)
            if (token != null) {
                settings.put("onboardingToken", token);
            }
            
            // E. Scrittura persistente nel Database Cifrato di Bramble
            settingsManager.mergeSettings(settings, SETTINGS_NAMESPACE_COMMUNITY);
            LOG.info("Inizializzato profilo Sandbox (PROBATION) nel DB per l'autore: " + authorId);

        } catch (DbException e) {
            LOG.log(WARNING, "Errore nell'inizializzazione del profilo Sandbox dell'account", e);
        }
    }
}
```
#### 🟢Come viene letto ed utilizzato questo profilo a runtime?
Successivamente, quando il nodo tenta di pubblicare un post o quando arriva un messaggio dalla rete, il modulo **`TrustManagerImpl`** legge le impostazioni salvate da **`AccountManagerImpl`**:

```java
// File: bramble-core/src/main/java/org/briarproject/bramble/trust/TrustManagerImpl.java

public boolean canPublishPost() throws DbException {
    // Carica le impostazioni dal namespace "community.trust" del DB cifrato
    Settings settings = settingsManager.getSettings("community.trust");
    
    String tier = settings.getOrDefault("trustTier", "PROBATION");
    if ("ISOLATED".equals(tier)) return false;

    if ("PROBATION".equals(tier)) {
        int maxPosts = Integer.parseInt(settings.getOrDefault("maxPostsPerDay", "1"));
        int postsToday = Integer.parseInt(settings.getOrDefault("postsToday", "0"));
        long lastPostTime = Long.parseLong(settings.getOrDefault("lastPostTimestamp", "0"));

        // Se è trascorso un giorno (24h), resetta il contatore
        if (System.currentTimeMillis() - lastPostTime > 86_400_000L) {
            postsToday = 0;
        }

        return postsToday < maxPosts;
    }

    return true; // Per i nodi FULL_MEMBER non vi è restrizione restrittiva
}
```
---

### B. `SetupViewModel.java` & UI Wizard (`AuthorNameFragment.java`)

#### 🔴 Come funziona in Briar originariamente:
Il wizard di primo avvio (`SetupActivity`) prevede 3 passaggi sequenziali:
1. Inserimento Nome Autore (`AuthorNameFragment`).
2. Impostazione Password (`SetPasswordFragment`).
3. Richiesta Esenzione Ottimizzazione Batteria Doze (`DozeFragment`).
Una volta completato, l'utente accede direttamente alla schermata principale di Briar senza selezionare categorie d'interesse o visualizzare vincoli di rete.

#### 🟢 Come funziona dopo le modifiche per la Tesi:
1. **Integrazione Deep Link da Browser**: Se l'utente ha avviato l'app da un link della Landing Page Web (`briar://community-onboard?pseudonym=...`), lo pseudonimo e i topic vengono pre-compilati automaticamente nel wizard.
2. **Selezione Topic di Interesse**: Inserito un nuovo passaggio UI (`SelectTopicsFragment`) in cui l'utente sceglie i topic della comunità a cui desidera contribuire.
3. **Schermata Informativa Sandbox (PROBATION)**: Viene mostrato un avviso chiaro che spiega: *"Stai entrando nella comunità in modalità Sandbox (Restrittiva). Potrai pubblicare max 1 messaggio al giorno e comunicare solo con nodi adiacenti finché la comunità non convaliderà i tuoi contributi."*

```java
// Estratto modificato: briar-android/.../account/SetupViewModel.java

enum State { AUTHOR_NAME, SELECT_TOPICS, SET_PASSWORD, DOZE, SANDBOX_NOTICE, CREATED, FAILED }

public class SetupViewModel extends AndroidViewModel {

    private final List<String> selectedTopics = new ArrayList<>();
    @Nullable private String onboardingToken;

    public void processWebDeepLink(Uri deepLinkUri) {
        // Estrae pseudonimo e topic pre-compilati dalla Landing Page Web HTML5
        String pseudonym = deepLinkUri.getQueryParameter("pseudonym");
        String token = deepLinkUri.getQueryParameter("token");
        if (pseudonym != null) {
            setAuthorName(pseudonym);
            this.onboardingToken = token;
        }
    }

    public void setSelectedTopics(List<String> topics) {
        this.selectedTopics.clear();
        this.selectedTopics.addAll(topics);
        state.setEvent(State.SET_PASSWORD);
    }

    // Invocato al completamento: passa i topic ad AccountManager
    private void createAccount() {
        ioExecutor.execute(() -> {
            if (accountManager.createAccount(authorName, password, selectedTopics, onboardingToken)) {
                state.postEvent(State.CREATED);
            } else {
                state.postEvent(State.FAILED);
            }
        });
    }
}
```

---

### C. `LinkExchangeFragment.java` & Importazione Inviti Comunità

#### 🔴 Come funziona in Briar originariamente:
L'aggiunta di un contatto avviene tramite `LinkExchangeFragment`, dove due utenti devono incollare/scambiare i rispettivi link personali nel formato `briar://<handshake-link-univoco>`. Questo meccanismo è esclusivamente **bidirezionale e simmetrico 1-a-1**.

#### 🟢 Come funziona dopo le modifiche per la Tesi:
Si estende il gestore di link per supportare i **Community Invitation Links / QR Codes** (`briar://community-invite/...`):
- Non è più necessario aggiungere singolarmente ogni membro come contatto reciproco.
- Scansionando il QR d'invito fornito da un membro esistente, l'app nativa importa le chiavi del Topic di Comunità e sottomette la richiesta d'ingresso all'intera rete locale via P2P.

```java
// Estratto modificato: briar-android/.../contact/add/remote/LinkExchangeFragment.java

private void onContinueButtonClicked() {
    String link = getRemoteHandshakeLinkOrNull();
    if (link == null) return;

    if (link.startsWith("briar://community-invite/")) {
        // [NUOVA LOGICA TESI]: Gestione Invito di Comunità
        viewModel.importCommunityInviteLink(link);
    } else {
        // Logica standard Briar (Aggiunta contatto 1-a-1)
        viewModel.setRemoteHandshakeLink(link);
        viewModel.onRemoteLinkEntered();
    }
}
```

---

### D. [NUOVO] `EmbeddedWebGateway.kt` (Server Landing Page HTML5)

#### 🔴 Come funziona in Briar originariamente:
Inesistente in Briar Android. Briar non possiede un server HTTP integrato per la distribuzione via browser.

#### 🟢 Come funziona dopo le modifiche per la Tesi:
Modulo server web integrato (utilizzando Ktor o NanoHTTPD embedded) attivo quando un dispositivo della comunità funge da Hotspot/Gateway locale:
1. Eroga la **Landing Page HTML5** accessibile da qualsiasi browser senza installazione.
2. Mostra regole, informazioni pubbliche della comunità e form per la bozza di richiesta d'ingresso.
3. Spiega i **limiti del browser** (impossibilità di usare Wi-Fi Direct in background) e fornisce il pulsante per scaricare direttamente l'APK dell'App Nativa.

```kotlin
// File NUOVO: briar-core/src/main/kotlin/org/briarproject/briar/web/EmbeddedWebGateway.kt
package org.briarproject.briar.web

import io.ktor.server.application.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import io.ktor.server.http.content.*
import io.ktor.server.request.*
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class EmbeddedWebGateway @Inject constructor() {

    fun startWebGateway(port: Int = 8080) {
        embeddedServer(Netty, port = port) {
            routing {
                // 1. Pagina Web HTML5 principale (Landing Page)
                staticResources("/", "web_landing")

                // 2. Download diretto dell'APK dell'App Nativa per l'utente senza Internet
                get("/download/briar-community.apk") {
                    val apkFile = java.io.File("/sdcard/Download/briar-community.apk")
                    if (apkFile.exists()) {
                        call.respondFile(apkFile)
                    } else {
                        call.respondText("APK non disponibile sul gateway", status = io.ktor.http.HttpStatusCode.NotFound)
                    }
                }

                // 3. Endpoint per la sottomissione della bozza di richiesta da browser
                post("/api/onboard/draft") {
                    val params = call.receiveParameters()
                    val pseudonym = params["pseudonym"] ?: "Anonimo"
                    val topics = params["topics"] ?: ""

                    // Ritorna le istruzioni ed il Deep Link per l'app nativa
                    call.respondText("""
                        {
                          "status": "DRAFT_CREATED",
                          "notice": "Il browser non può connettersi via Wi-Fi Direct. Installa l'app nativa per completare l'onboarding.",
                          "deepLink": "briar://community-onboard?pseudonym=$pseudonym&topics=$topics"
                        }
                    """.trimIndent())
                }
            }
        }.start(wait = false)
    }
}
```

---

### E. [NUOVO] `CommunityOnboardingManager.java` & `OnboardingRequest.java`

#### 🔴 Come funziona in Briar originariamente:
Inesistente. Non esiste un concetto di "richiesta di ammissione a una comunità" né una gestione automatica dei permessi dei membri.

#### 🟢 Come funziona dopo le modifiche per la Tesi:
Gestisce il ciclo di vita delle richieste di onboarding inviate e ricevute sulla rete ad-hoc:

```java
// File NUOVO: briar-api/src/main/java/org/briarproject/briar/api/community/OnboardingRequest.java
package org.briarproject.briar.api.community;

import org.briarproject.bramble.api.identity.Author;
import java.util.List;

public class OnboardingRequest {
    private final Author applicantAuthor;
    private final List<String> requestedTopics;
    private final long timestamp;
    private boolean approved;

    public OnboardingRequest(Author applicantAuthor, List<String> requestedTopics, long timestamp) {
        this.applicantAuthor = applicantAuthor;
        this.requestedTopics = requestedTopics;
        this.timestamp = timestamp;
        this.approved = false;
    }

    public Author getApplicantAuthor() { return applicantAuthor; }
    public List<String> getRequestedTopics() { return requestedTopics; }
    public boolean isApproved() { return approved; }
    public void setApproved(boolean approved) { this.approved = approved; }
}
```

---

## 4. Sintesi dei Risultati del Modulo 1

1. **Integrazione Perfetta con i Limiti HTML**: Il browser agisce da punto d'ingresso informativo e guida l'utente al download/avvio dell'app nativa.
2. **Onboarding in Sandbox**: L'utente entra subito nella rete nativa ma con lo stato **`TrustTier.PROBATION`** gestito dentro `AccountManagerImpl`.
3. **Piena Compatibilità con Briar**: Nessun stravolgimento della cifratura o del DB di Bramble, ma un'estensione elegante del flusso di registrazione e gestione delle identità.
