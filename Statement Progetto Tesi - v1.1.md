**Progetto di Tesi: Architettura di Rete P2P Opportunistica per Scenari
di Emergenza**

**1. Scenario e Obiettivi**

Una comunità di utenti si trova in una situazione catastrofica o di
forte censura dove le connessioni Internet centralizzate non sono più
affidabili e si vuole riuscire a comunicare in maniera libera e
protetta. Si formerà quindi una comunità localizzata in una certa area,
che cerca di raccogliere utenti in grado di scambiare informazioni utili
al fine di far emergere le problematiche di tale area. Queste
informazioni devono circolare il più possibile all’interno della
comunità di nodi locale in modo da essere validate dagli altri utenti
tramite quorum di prossimità. Un’informazione specifica può essere
arricchita di dettagli, non solo dall’utente che l’ha prodotta, ma
eventualmente anche dagli utenti che la ricevono all’interno della rete,
i quali devono essere ritenuti affidabili dagli altri nodi a loro volta.
Si parla quindi di un meccanismo Publish/Subscribe. Le informazioni che
circolano possono essere consultate dai nodi all’interno della comunità
in base agli argomenti che trattano (**Topic**) e la locazione di tali
informazioni all’interno della comunità è protetta e sparsa, non
centralizzata. Una volta che una specifica informazione riguardante un
particolare evento è stata consultata e validata da più nodi possibile,
viene impacchettata e formattata in modo da poter essere condivisa
all’esterno della comunità come prova non-confutabile di un evento
realmente accaduto. In generale quindi il sistema non implementa un
sistema di messaggistica tra utenti, quanto più, per usare termini
vicini al mondo dei social media, un meccanismo di “post” che possono o
meno coinvolgere l’interesse di altri nodi.

**2. Attori della Rete e Requisiti Tecnologici**

- **Requisiti Hardware:** Di base ogni utente deve poter essere in
  possesso di uno smartphone. Questo deve disporre di una potenza di
  calcolo minima per crittografare le informazioni scambiate, memoria
  per salvare dati importanti e interfacce radio per comunicare.

- **Attività nella comunità:** Condividere informazioni che lo
  coinvolgono in prima persona o di cui è venuto a conoscenza. Scegliere
  categorie di interesse sulle quali può portare contributi rilevanti
  con la sua esperienza diretta e rimanere aggiornato sugli eventi della
  stessa categoria.

**3. Connettività e Infrastruttura di Rete**

Per assicurare la continuità operativa del flusso informativo in
contesti avversi, l'architettura di sistema deve fondarsi sui paradigmi
delle reti ad-hoc, opportunistiche e tolleranti alle interruzioni:

- **MANET (Mobile Ad-Hoc Networks):** Reti dinamiche prive di
  infrastruttura fissa, formate da dispositivi mobili che comunicano
  direttamente tramite interfacce radio (es. Wi-Fi IEEE 802.11 a 2.4
  GHz) in configurazione peer-to-peer (P2P). La topologia varia in modo
  stocastico a causa della mobilità, richiedendo protocolli multi-hop su
  canali instabili.

- **DTN / OppNet (Disruption-Tolerant Networks / Opportunistic
  Networks):** Architetture overlay che operano al di sopra dei
  protocolli di trasporto per gestire le disconnessioni, disaccoppiando
  il mittente dal destinatario.

- **Store-Carry-and-Forward:** Ogni nodo intermedio agisce come un
  router autonomo; memorizza i pacchetti nella memoria locale
  persistente (*store*), li trasporta fisicamente durante il movimento
  (*carry*) e li trasmette opportunisticamente solo quando entra nel
  raggio di copertura radio di un altro nodo idoneo (*forward*).

**4. Superamento dei Limiti HTML e Ruolo dell'Applicazione Nativa**

Sebbene l'obiettivo iniziale sia mantenere una bassa soglia di adozione
sfruttando un punto d'ingresso web universale (tramite browser o QR
code), l'analisi tecnica dei vincoli dei browser evidenzia che
**un'interfaccia puramente basata su HTML/browser non è sufficiente** a
gestire i requisiti di una rete DTN:

- **Assenza di API Web Standard per Wi-Fi Direct:** Su Android e altri
  sistemi operativi mobili, il controllo diretto delle interfacce radio
  necessita delle API native. *Web Bluetooth* non costituisce una base
  affidabile per la scoperta continua e la sincronizzazione in
  background tra telefoni.

- **Persistenza e Nodi di Rete:** Il browser non è adatto a fungere da
  nodo persistente della rete. La conservazione a lungo termine di
  identità crittografiche, chiavi, dati e la sincronizzazione in
  background non sono garantite uniformemente dai browser.

**Divisione Funzionale: Web vs. Componente Nativo**

- **Cosa è possibile fare via Web:** Consultare informazioni pubbliche
  se la pagina è raggiungibile, leggere regole e topic, compilare bozze
  di post (tramite PWA se precaricata) e preparare la richiesta
  d'ingresso.

- **Cosa richiede obbligatoriamente l'App Nativa:** La scoperta dei peer
  vicini, il trasferimento dati tramite Wi-Fi Direct/Bluetooth nativo,
  la gestione della crittografia di identità e la partecipazione attiva
  al routing *store-carry-and-forward*.

**5. Modalità d'Accesso e Onboarding Progressivo**

L'accesso iniziale alla comunità avviene tramite due modalità
principali, integrate con il supporto di installazione dell'applicazione
nativa:

1.  **Scansione autonoma:** Un nuovo utente scannerizza un QR code
    pubblico affisso nell'area geografica. Il codice apre una pagina web
    informativa (se raggiungibile) che spiega il funzionamento della
    comunità e guida l'utente al download/installazione
    dell'applicazione nativa necessaria per la partecipazione effettiva.

2.  **Invito diretto:** A seguito dell'incontro con un membro già
    inserito, questi fornisce un indirizzo web o un QR code d'invito. Se
    non vi è connessione Internet, l'invito viene importato direttamente
    nell'app nativa del nuovo utente.

**Distribuzione dell'App in Assenza di Internet**

Poiché il nuovo utente potrebbe non avere ancora installato l'app, la
comunità deve poter distribuire il pacchetto di installazione senza
dipendere da Internet. Si considerano le seguenti soluzioni:

- **Punto Wi-Fi locale:** un nodo della comunità (ad esempio un router,
  un laptop o un mini-computer) crea una rete locale e ospita una pagina
  di onboarding e il pacchetto di installazione. Un QR code indica
  l'indirizzo locale e può contenere anche i dati d'invito. Il nuovo
  utente si collega alla rete, scarica il pacchetto e prosegue con
  l'installazione. Questa è la modalità raccomandata per il prototipo.

- **Trasferimento diretto o supporto rimovibile:** un membro può
  trasferire il pacchetto tramite Bluetooth, Quick Share, cavo USB,
  memoria USB-C/OTG o scheda di memoria. L'invito può essere trasferito
  insieme al pacchetto oppure importato successivamente tramite QR code.

- **Kit di distribuzione predisposto:** la comunità può conservare una
  copia del pacchetto su un laptop, un mini-server, una chiavetta USB o
  una scheda di memoria, pronta per scenari d'emergenza.

Il pacchetto Android deve essere firmato con la chiave di distribuzione
del progetto. La firma e un'impronta verificabile permettono di
controllarne l'autenticità; Android può richiedere all'utente di
autorizzare l'installazione dalla fonte utilizzata. Il QR code non
contiene l'applicazione: ne indica la posizione nella rete locale o
trasporta i dati d'invito. Una PWA può fornire un'interfaccia offline
solo se è stata caricata o installata in precedenza e non sostituisce
l'app nativa per le funzioni DTN. La distribuzione su iOS richiede un
canale specifico della piattaforma e non è equivalente al trasferimento
di un APK; per il prototipo, la distribuzione offline è quindi
prioritariamente prevista per Android.

**Flusso di Richiesta d'Ingresso e Restrizioni Iniziali**

- **Identità e Risorse:** Una volta installata l'app, il dispositivo
  genera e conserva l'identità crittografica dell'utente e comunica le
  risorse tecnologiche disponibili.

- **Onboarding Controllato:** Se l'utente entra autonomamente, le 
  sue azioni saranno limitate (Sandbox). Se entra tramite invito di un
  membro, la richiesta formale di approvazione può essere snellita ma 
  richiede comunque la comunicazione dell'identità.

- **Capacità Limitate iniziali (Sandbox di Sicurezza):** Per evitare
  minacce alla sicurezza della comunità, il nuovo nodo accede
  inizialmente con forti restrizioni:

  - Rate limit (es. pubblicazione di un solo 1 informazione al giorno);

  - Comunicazione limitata unicamente ai peer più vicini;

  - Divieto di propagare informazioni ritenute critiche;

  - Impossibilità di votare nelle decisioni della comunità o di
    introdurre nuovi nodi;

  - Accesso ristretto a una sola parte delle informazioni pubblicate.

- **Promozione o Isolamento:** Se dopo un periodo di osservazione
  l'utente e i suoi contributi vengono valutati positivamente tramite il
  quorum di prossimità, le restrizioni decadono e il nodo viene
  integrato a pieno titolo. In caso di comportamenti anomali, il nodo
  viene isolato ed escluso dalle <u>comunicazioni</u>.
