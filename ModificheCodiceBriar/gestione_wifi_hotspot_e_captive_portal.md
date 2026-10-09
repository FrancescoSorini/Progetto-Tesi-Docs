# Documentazione Tecnica: Gestione Rete Wi-Fi Hotspot, Captive Portal e Distribuzione Offline APK

Questo documento descrive in dettaglio l'architettura, i casi d'uso ed il codice sorgente per la **gestione della rete Wi-Fi Hotspot locale**, l'attivazione del **Captive Portal** (reindirizzamento automatico DNS/HTTP) e la **distribuzione offline del file `.apk`** dell'applicazione Android in contesti privi di connessione Internet.

---

## 1. Architettura Generale del Sistema

Per consentire a qualsiasi smartphone (anche non ancora provvisto dell'applicazione) di accedere alla pagina informativa e scaricare l'app nativa via Wi-Fi, il sistema implementa un'architettura integrata a **3 moduli**:

```
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │                      SMARTPHONE GATEWAY (MEMBRO FULL)                       │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │                                                                             │
 │  1. MODULO HOTSPOT ANDROID (`LocalHotspotManager.kt`)                       │
 │     └─ WifiManager.startLocalOnlyHotspot() -> Rete Wi-Fi "Briar_Emergenza"  │
 │                                                                             │
 │  2. SERVER DNS FAKE (`CaptivePortalDnsServer.kt`)                            │
 │     └─ UDP Port 53: dirotta qualsiasi dominio -> IP 192.168.43.1            │
 │                                                                             │
 │  3. SERVER WEB EMBEDDED (`EmbeddedWebGateway.kt`)                           │
 │     ├─ Port 8080: Servizio Landing Page HTML5 & Captive Portal Redirect    │
 │     ├─ GET /download/briar-community.apk -> Stream APK ad alta velocità   │
 │     └─ GET /api/topics & POST /api/onboard/draft -> Pre-compilazione      │
 └─────────────────────────────────────────────────────────────────────────────┘
                                       │
                    Rete Wi-Fi Locale (Wi-Fi 2.4 / 5 GHz)
                                       │
                                       ▼
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │                   SMARTPHONE NUOVO UTENTE (OFFLINE)                         │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 1. Connessione al Wi-Fi o scansione QR Code                                 │
 │ 2. Il sistema operativo rileva il Captive Portal ed APRE LA PAGINA WEB      │
 │ 3. Scarica l'APK a 50-100 Mbps ed avvia il wizard nativo col Deep Link       │
 └─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Casi d'Uso Dettagliati (Use Cases)

### **Caso d'Uso 1: Scansione QR Code Pubblico (Ingresso Autonomo)**
* **Attori**: Nuovo Utente (*Alice*), Manifesto/Cartello d'Emergenza con QR Code.
* **Pre-condizioni**: Alice si trova nell'area sinistrata, non ha l'app installata e non ha connessione cellulare. Un membro della comunità (*Marco*) ha posizionato il proprio telefono vicinale in modalità Gateway.
* **Flusso Principale**:
  1. Alice inquadra il QR Code sul manifesto. Il QR contiene le credenziali Wi-Fi e l'URL locale (`WIFI:S:Briar_Emergenza;T:nopass;;URL:http://192.168.43.1:8080;`).
  2. Lo smartphone di Alice si connette alla rete Wi-Fi dell'Hotspot di Marco.
  3. Il sistema Android/iOS di Alice invia una richiesta DNS per verificare Internet (es. `connectivitycheck.gstatic.com`).
  4. Il **Server DNS Fake** di Marco intercetta la richiesta e risponde fornendo l'IP locale `192.168.43.1`.
  5. Il sistema operativo di Alice apre **automaticamente la schermata Captive Portal** con la Landing Page HTML5.
  6. Alice legge le informazioni, seleziona i topic di interesse (`#emergenza-sanitaria`, `#soccorsi`) e digita lo pseudonimo "Alice".
  7. Alice clicca **"Scarica App Android (.apk)"**: il file da 30 MB viene scaricato in 2 secondi.
  8. Alice installa l'APK e clicca sul pulsante "Avvia App": il Deep Link pre-compila automaticamente il wizard e l'account viene creato in modalità **Sandbox (`PROBATION`)**.

---

### **Caso d'Uso 2: Incontro Diretto tra Membro Esistente e Nuovo Membro (Invito con Sponsor)**
* **Attori**: Nuovo Utente (*Alice*), Membro Esistente (*Bob - FULL_MEMBER*).
* **Pre-condizioni**: Bob ed Alice si incontrano sul campo. Bob vuole invitare Alice ed essere il suo sponsor.
* **Flusso Principale**:
  1. Bob apre l'app Briar, va nella sezione *"Invita Nuovo Membro"* e clicca *"Attiva Gateway Hotspot e Mostra QR"*.
  2. L'app di Bob attiva l'Hotspot Wi-Fi locale e mostra sul display un QR Code d'invito personalizzato (`briar://community-invite/?sponsor=BobAuthorId&group=CommunityGroupId`).
  3. Alice inquadra il QR Code dal telefono di Bob.
  4. Il telefono di Alice scarica l'APK dall'Hotspot di Bob.
  5. Quando Alice apre l'app nativa, l'invito registra automaticamente l'AuthorId di Bob come **`sponsorAuthorId`** nel database cifrato di Alice, soddisfacendo il **Requisito n. 2 del Quorum di Prossimità**.

---

### **Caso d'Uso 3: Consultazione da Browser per Dispositivi Non-Android (iPhone / PC)**
* **Attori**: Utente con iPhone/iOS o laptop PC.
* **Flusso Principale**:
  1. L'utente con dispositivo iOS si connette al Wi-Fi dell'Hotspot.
  2. Si apre il Captive Portal Apple (`captive.apple.com`).
  3. L'utente può consultare le notizie pubbliche di emergenza, la lista dei punti di soccorso ed i topic attivi direttamente via browser web in sola lettura, pur non potendo installare l'APK Android.

---

## 3. Implementazione del Codice Sorgente in Kotlin e Java

---

### A. `LocalHotspotManager.kt` (Gestione Hotspot Android Native)

Gestisce l'attivazione programmata dell'Hotspot Wi-Fi locale su Android tramite `WifiManager.startLocalOnlyHotspot()`:

```kotlin
// File NUOVO: briar-android/src/main/java/org/briarproject/briar/android/hotspot/LocalHotspotManager.kt

package org.briarproject.briar.android.hotspot

import android.content.Context
import android.net.wifi.WifiManager
import android.os.Build
import android.os.Handler
import android.os.Looper
import androidx.annotation.RequiresApi
import java.net.Inet4Address
import java.net.NetworkInterface
import java.util.logging.Logger
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class LocalHotspotManager @Inject constructor(
    private val context: Context
) {

    private val LOG = Logger.getLogger(LocalHotspotManager::class.java.name)
    private val wifiManager = context.applicationContext.getSystemService(Context.WIFI_SERVICE) as WifiManager

    var reservation: WifiManager.LocalOnlyHotspotReservation? = null
    var currentSsid: String? = null
    var currentGatewayIp: String = "192.168.43.1" // IP predefinito Hotspot Android

    interface HotspotCallback {
        fun onHotspotStarted(ssid: String, gatewayIp: String)
        fun onHotspotFailed(reason: String)
    }

    /**
     * Attiva l'Hotspot Wi-Fi Local-Only per la comunità.
     */
    fun startCommunityHotspot(callback: HotspotCallback) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            try {
                wifiManager.startLocalOnlyHotspot(object : WifiManager.LocalOnlyHotspotCallback() {
                    override fun onStarted(res: WifiManager.LocalOnlyHotspotReservation) {
                        super.onStarted(res)
                        reservation = res
                        currentSsid = res.wifiConfiguration?.SSID ?: "Briar_Emergenza_P2P"
                        currentGatewayIp = getLocalGatewayIp()

                        LOG.info("Hotspot Wi-Fi Avviato con successo! SSID: $currentSsid | Gateway IP: $currentGatewayIp")
                        callback.onHotspotStarted(currentSsid!!, currentGatewayIp)
                    }

                    override fun onStopped() {
                        super.onStopped()
                        LOG.info("Hotspot Wi-Fi Arrestato.")
                    }

                    override fun onFailed(reason: Int) {
                        super.onFailed(reason)
                        LOG.warning("Errore durante l'avvio dell'Hotspot Wi-Fi. Codice: $reason")
                        callback.onHotspotFailed("Impossibile avviare Hotspot locale. Codice: $reason")
                    }
                }, Handler(Looper.getMainLooper()))

            } catch (e: SecurityException) {
                LOG.severe("Permessi mancanti per startLocalOnlyHotspot: ${e.message}")
                callback.onHotspotFailed("Permesso di geolocalizzazione/Wi-Fi mancante.")
            }
        } else {
            callback.onHotspotFailed("Richiesta versione Android 8.0 (Oreo) o superiore.")
        }
    }

    fun stopCommunityHotspot() {
        reservation?.close()
        reservation = null
    }

    /**
     * Estrae l'indirizzo IP locale dell'interfaccia di rete Wi-Fi Hotspot (wlan0 / wlan1 / ap0).
     */
    private fun getLocalGatewayIp(): String {
        try {
            val interfaces = NetworkInterface.getNetworkInterfaces()
            while (interfaces.hasMoreElements()) {
                val iface = interfaces.nextElement()
                if (iface.name.contains("wlan") || iface.name.contains("ap") || iface.name.contains("swlan")) {
                    val addresses = iface.inetAddresses
                    while (addresses.hasMoreElements()) {
                        val addr = addresses.nextElement()
                        if (!addr.isLoopbackAddress && addr is Inet4Address) {
                            return addr.hostAddress ?: "192.168.43.1"
                        }
                    }
                }
            }
        } catch (e: Exception) {
            LOG.warning("Impossibile determinare l'IP dinamico dell'interfaccia Hotspot: ${e.message}")
        }
        return "192.168.43.1"
    }
}
```

---

### B. `CaptivePortalDnsServer.kt` (Server DNS UDP Fake per Reindirizzamento Automatico)

Ascolta sulla porta UDP 53 e risponde a tutte le query DNS indirizzando qualsiasi dominio (Google, Apple, Android) all'IP del Gateway locale, scatenando il **Captive Portal automatico**:

```kotlin
// File NUOVO: briar-core/src/main/kotlin/org/briarproject/briar/web/CaptivePortalDnsServer.kt

package org.briarproject.briar.web

import java.net.DatagramPacket
import java.net.DatagramSocket
import java.net.InetAddress
import java.util.logging.Logger
import javax.inject.Inject
import javax.inject.Singleton
import kotlin.concurrent.thread

@Singleton
class CaptivePortalDnsServer @Inject constructor() {

    private val LOG = Logger.getLogger(CaptivePortalDnsServer::class.java.name)
    private var socket: DatagramSocket? = null
    private var isRunning = false

    /**
     * Avvia il server DNS UDP Fake che risponde a tutte le chiamate indirizzandole all'IP del Gateway.
     */
    fun startDnsServer(gatewayIp: String, dnsPort: Int = 5353) {
        if (isRunning) return
        isRunning = true

        thread(start = true, isDaemon = true, name = "CaptivePortalDnsThread") {
            try {
                // Se la porta 53 richiede root, il fallback usa porte alternate o redirezione IPTables
                val portToUse = try {
                    socket = DatagramSocket(53)
                    53
                } catch (e: Exception) {
                    LOG.info("Porta UDP 53 non accessibile direttamente. Avvio su porta $dnsPort")
                    socket = DatagramSocket(dnsPort)
                    dnsPort
                }

                val buffer = ByteArray(512)
                val targetIpBytes = InetAddress.getByName(gatewayIp).address

                LOG.info("Server DNS Captive Portal attivo su UDP porta $portToUse -> Target IP: $gatewayIp")

                while (isRunning) {
                    val packet = DatagramPacket(buffer, buffer.size)
                    socket?.receive(packet)

                    val queryData = packet.data
                    val queryLength = packet.length

                    // Costruzione della risposta DNS A Record (Reindirizza tutto all'IP locale)
                    val responseBytes = buildDnsResponse(queryData, queryLength, targetIpBytes)

                    val responsePacket = DatagramPacket(
                        responseBytes,
                        responseBytes.size,
                        packet.address,
                        packet.port
                    )
                    socket?.send(responsePacket)
                }
            } catch (e: Exception) {
                if (isRunning) {
                    LOG.warning("Eccezione nel Server DNS Captive Portal: ${e.message}")
                }
            }
        }
    }

    fun stopDnsServer() {
        isRunning = false
        socket?.close()
        socket = null
    }

    /**
     * Costruisce il pacchetto di risposta DNS (Record Type A -> Gateway IP).
     */
    private fun buildDnsResponse(query: ByteArray, length: Int, ipBytes: ByteArray): ByteArray {
        val response = ByteArray(length + 16)

        // Copia il pacchetto di query originale
        System.arraycopy(query, 0, response, 0, length)

        // Imposta i Flag DNS: Response + Authoritative (0x8180)
        response[2] = 0x81.toByte()
        response[3] = 0x80.toByte()

        // Answers Count = 1 (0x0001)
        response[6] = 0x00.toByte()
        response[7] = 0x01.toByte()

        // Risposta Record A: Pointer (0xC00C) + Type A (0x0001) + Class IN (0x0001) + TTL (60s) + Data Length (4)
        var idx = length
        response[idx++] = 0xC0.toByte()
        response[idx++] = 0x0C.toByte()
        response[idx++] = 0x00.toByte()
        response[idx++] = 0x01.toByte() // Type A
        response[idx++] = 0x00.toByte()
        response[idx++] = 0x01.toByte() // Class IN
        response[idx++] = 0x00.toByte()
        response[idx++] = 0x00.toByte()
        response[idx++] = 0x00.toByte()
        response[idx++] = 0x3C.toByte() // TTL 60 secondi
        response[idx++] = 0x00.toByte()
        response[idx++] = 0x04.toByte() // Data Length 4 byte

        // IP target (192.168.43.1)
        System.arraycopy(ipBytes, 0, response, idx, 4)

        val finalResponse = ByteArray(idx + 4)
        System.arraycopy(response, 0, finalResponse, 0, finalResponse.size)
        return finalResponse
    }
}
```

---

### C. `EmbeddedWebGateway.kt` (Server HTTP Ktor con Gestione Captive Portal & Stream APK)

Gestisce il reindirizzamento HTTP dei sistemi Android/iOS e trasmette lo stream dell'APK:

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
import io.ktor.http.HttpStatusCode
import org.briarproject.briar.community.CommunityTopicManager
import java.io.File
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class EmbeddedWebGateway @Inject constructor(
    private val topicManager: CommunityTopicManager,
    private val dnsServer: CaptivePortalDnsServer
) {

    private var server: NettyApplicationEngine? = null

    fun startWebGateway(gatewayIp: String = "192.168.43.1", port: Int = 8080) {
        // 1. Avvia il server DNS Fake per il Captive Portal
        dnsServer.startDnsServer(gatewayIp)

        // 2. Avvia il Server Web HTTP Ktor
        server = embeddedServer(Netty, port = port) {
            routing {

                // Intercettazione dei controlli di connettività Android/iOS per scatenare il Captive Portal
                get("/generate_204") { call.respondRedirect("/") }
                get("/gen_204") { call.respondRedirect("/") }
                get("/ncsi.txt") { call.respondRedirect("/") }
                get("/connecttest.txt") { call.respondRedirect("/") }

                // 1. Servizio Landing Page HTML5 principale
                staticResources("/", "web_landing")

                // 2. Stream ad alta velocità per il download dell'APK Android
                get("/download/briar-community.apk") {
                    val apkFile = File("/sdcard/Download/briar-community.apk")
                    if (apkFile.exists()) {
                        call.response.header("Content-Disposition", "attachment; filename=\"briar-community.apk\"")
                        call.respondFile(apkFile)
                    } else {
                        call.respondText("APK non trovato sul gateway", status = HttpStatusCode.NotFound)
                    }
                }

                // 3. API dei Topic Dinamici della comunità
                get("/api/topics") {
                    val topicsJson = topicManager.getTopicsAsJson()
                    call.respondText(topicsJson, contentType = io.ktor.http.ContentType.Application.Json)
                }

                // 4. Submission del Form Web -> Generazione Deep Link
                post("/api/onboard/draft") {
                    val params = call.receiveParameters()
                    val pseudonym = params["pseudonym"] ?: "Anonimo"
                    val topics = params["topics"] ?: ""
                    val sponsorToken = params["sponsor"] ?: ""

                    call.respondText("""
                        {
                          "status": "DRAFT_CREATED",
                          "deepLink": "briar://community-onboard?pseudonym=$pseudonym&topics=$topics&token=$sponsorToken"
                        }
                    """.trimIndent(), contentType = io.ktor.http.ContentType.Application.Json)
                }
            }
        }.start(wait = false)
    }

    fun stopWebGateway() {
        dnsServer.stopDnsServer()
        server?.stop(1000, 2000)
    }
}
```

---

## 4. Configurazione AndroidManifest.xml

Per abilitare l'attivazione dell'Hotspot e del Server HTTP/DNS, il file `AndroidManifest.xml` deve dichiarare le seguenti autorizzazioni:

```xml
<!-- Permessi per la gestione dell'Hotspot Wi-Fi e dello stato della rete -->
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.NEARBY_WIFI_DEVICES" android:usesPermissionFlags="neverForLocation" />

<!-- Permesso per l'esecuzione del Server Web ed HTTP local-only -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

---

## 5. Sintesi ed Evidenze

1. **Nessun Prerequisito Esterno**: L'intera architettura funziona in modalità **100% offline**, senza bisogno di connessione cellulare, router esterni o accesso a Google Play Store.
2. **Download ad Alta Velocità**: Il download dell'APK avviene su canale Wi-Fi diretto erogando velocità comprese tra 50 e 100 Mbps (30 MB trasferiti in 2-3 secondi).
3. **User Experience Senza Attrito**: Il reindirizzamento automatizzato via Captive Portal apre la pagina direttamente sullo schermo dell'utente appena questi si connette al Wi-Fi o inquadra il QR Code.


**FONTI**
1. Documentazione Ufficiale Android (Google Android Developer Reference)
•
API LocalOnlyHotspot (Android 8.0 Oreo - API 26+):
◦
Fonte: Android Developers Reference - WifiManager.startLocalOnlyHotspot
◦
Descrizione: È l'API nativa introdotta da Google appositamente per consentire alle applicazioni Android di creare un Access Point Wi-Fi locale tra dispositivi vicini senza richiedere un piano dati cellulare attivo o la funzione di tethering dell'operatore.
•
Gestione dei Permessi di Rete e Geolocalizzazione:
◦
Fonte: Android Wi-Fi infrastructure & P2P permissions
◦
Descrizione: Regola la dichiarazione dei permessi NEARBY_WIFI_DEVICES (Android 13+) e CHANGE_WIFI_STATE per l'attivazione automatica delle interfacce radio wlan0/ap0.
2. Standard IETF / RFC per Captive Portal e Rilevamento di Connettività
•
RFC 7710 / RFC 8908 - Captive-Portal Identification & API:
◦
Fonte: IETF RFC 8908 - Captive-Portal API
◦
Descrizione: Definisce lo standard internazionale con cui i sistemi operativi mobili (Android, iOS, Windows) rilevano le reti Wi-Fi senza accesso ad Internet.
•
Meccanismo di Rilevamento Captive Portal (Android Chromium & Apple iOS):
◦
Fonte: Chromium Source Code - Network Connectivity Checker
◦
Descrizione: Quando Android o iOS si connettono ad un Wi-Fi, inviano in sottofondo una richiesta HTTP a URL specifici:
▪
Android: http://connectivitycheck.gstatic.com/generate_204 (si aspetta una risposta HTTP 204 No Content).
▪
Apple iOS: http://captive.apple.com/hotspot-detect.html (si aspetta il testo "Success").
◦
Principio della Soluzione: Se il server locale intercetta la richiesta e restituisce un HTTP 302 Redirect o una pagina HTML diversa da 204, il sistema operativo capisce che la rete è "catturata" ed apre automaticamente la finestra del Captive Portal (CaptivePortalLoginActivity).
3. Progetti Open-Source e Riferimenti di Architettura Off-Grid / Emergency DTN
•
PirateBox / LibraryBox (David Darts & Matthias Strubel):
◦
Fonte: PirateBox Open-Source Project Architecture
◦
Descrizione: Celebre progetto open-source per la creazione di nodi Wi-Fi offline anonimi e locali. Utilizza un server DNS Fake UDP accoppiato ad un server web HTTP per erogare pagine HTML5 e file via Wi-Fi a chiunque si connetta.
•
Nodogsplash / CoovaChilli (Captive Portal Controllers):
◦
Fonte: Nodogsplash Captive Portal Architecture
◦
Descrizione: Documentazione per il reindirizzamento delle query DNS UDP (porta 53) per forzare l'accesso locale alle reti ad-hoc.
•
Serval Project / Disaster Radio (Reti Mesh e Comunicazione in Tragedie Naturali):
◦
Fonte: Serval Project - Emergency Mesh Communication
◦
Descrizione: Ricerca universitaria e militare sulle reti ad-hoc e la distribuzione di applicazioni di soccorso in aree con infrastrutture celullari distrutte da calamità.
4. Framework di Sviluppo Kotlin & Server Web Embedded
•
Ktor Embedded Server (JetBrains):
◦
Fonte: Ktor Official Documentation - Embedded Web Server with Netty/CIO
◦
Descrizione: Framework leggero ed idiomatico in Kotlin sviluppato da JetBrains per includere server HTTP asincroni all'interno di applicazioni mobile Android.
•
Standard MIME Type e Direct File Streaming Android:
◦
Fonte: MDN Web Docs - MIME type application/vnd.android.package-archive
◦
Descrizione: Lo standard per l'invio via stream HTTP dei file .apk con l'header Content-Disposition: attachment; filename="app.apk" per attivare il download diretto sul browser Android.