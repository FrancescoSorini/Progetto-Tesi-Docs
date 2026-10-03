**Statement Progetto Tesi**

**SCENARIO**

> Una comunità di utenti si trova in una situazione catastrofica o di
> forte censura dove le connessioni Internet centralizzate non sono più
> affidabili e si vuole riuscire a comunicare in maniera libera e
> protetta. Si formerà quindi una comunità localizzata in una certa
> area, che cerca di raccogliere utenti in grado di scambiare
> informazioni utili al fine di far emergere le problematiche di tale
> area. Queste informazioni devono circolare il più possibile
> all’interno della comunità di nodi locale in modo da essere validate
> dagli altri utenti tramite quorum di prossimità. Un’informazione
> specifica può essere arricchita di dettagli, non solo dall’utente che
> l’ha prodotta, ma eventualmente anche dagli utenti che la ricevono
> all’interno della rete, questi devono essere ritenuti affidabili dagli
> altri nodi a loro volta. Si parla quindi di un meccanismo
> Publish/Subscribe. Le informazioni che circolano possono essere
> consultate dai nodi all’interno della comunità in base agli argomenti
> che trattano (*Topic*) e la locazione di tali informazioni all’interno
> della comunità è protetta e sparsa, non centralizzata. Una volta che
> una specifica informazione riguardante un particolare evento è stata
> consultata e validata da più nodi possibile, viene impacchettata e
> formattata in modo da poter essere condivisa all’esterno della
> comunità come prova non-confutabile di un evento realmente accaduto.
> In generale quindi il sistema non implementa un sistema di
> messaggistica tra utenti, quanto più, per usare termini vicini al
> mondo dei social media, un meccanismo di “post” che possono o meno
> coinvolgere l’interesse di altri nodi

**ATTORI DELLA RETE**

- **Requisiti Tecnologici:** Di base ogni utente deve poter essere in
  possesso di uno smartphone. Questo deve poter avere un minimo di
  potenza di calcolo tale da poter crittografare le informazioni
  scambiate, memoria per salvare alcuni dei dati importanti, scheda di
  rete per comunicare (specificata nei metodi di connessione).

- **Attività nella comunità:** Condividere informazioni che lo
  coinvolgono in prima persona o di cui è venuto a conoscenza. Scegliere
  categorie di interesse sulle quali può portare contributi rilevanti
  con la sua esperienza diretta e, di conseguenza, rimanere aggiornato
  su eventi della stessa categoria

**CONNETTIVITÀ**

> Per assicurare la continuità operativa del flusso informativo in tali
> contesti avversi, l'architettura di sistema deve fondarsi sui
> paradigmi delle reti ad-hoc, opportunistiche e tolleranti alle
> interruzioni. Si parla di **Mobile Ad-Hoc Networks** (**MANET**), reti
> dinamiche prive di infrastruttura fissa, formate da dispositivi mobili
> che comunicano direttamente tramite interfacce radio (es. Wi-Fi IEEE
> 802.11 a 2.4 GHz) in configurazione **peer-to-peer** (P2P). La
> topologia di rete varia in modo stocastico a causa della mobilità dei
> nodi, richiedendo protocolli di instradamento capaci di ricreare
> percorsi multi-hop su canali instabili.
>
> Proprio perché la connettività non è sempre garantita si richiede che
> la rete sia tollerante alle disconnessioni, si parla quindi di
> **Opportunistic Networks** (**OppNet**) e **Disruption-Tolerant
> Network** (**DTN**). Queste sono architetture di rete sovrapposte
> (overlay networks) che operano al di sopra dei protocolli di trasporto
> sottostanti tramite livelli intermedi. Le DTN disaccoppiano il
> mittente dal destinatario, eliminando il requisito di un percorso
> end-to-end contiguo e simultaneamente attivo.
>
> Si ha a che fare principalmente con un paradigma di routing
> **Store-Carry-and-Forward**, ogni nodo intermedio agisce come un
> router autonomo; memorizza i pacchetti di dati nella memoria di massa
> persistente locale (*store*), li trasporta fisicamente durante il
> proprio movimento nello spazio (*carry*) e li trasmette
> opportunisticamente solo quando entra nel raggio di copertura radio di
> un altro nodo idoneo (*forward*).

**MODALITÀ DI ACCESSO**

> Nell’ottica di mantenere una bassa soglia di adozione, il sistema non
> deve imporre l'installazione preventiva di software complessi o
> modifiche radicali alle abitudini degli utenti. L'accesso iniziale
> deve basarsi su protocolli web universali supportati da qualsiasi
> browser HTML5. Un utente che vuole mettersi in contatto con una
> comunità preesistente ha 2 opzioni per farlo.

1)  Scannerizzazione di un QR code presente nell’area geografica dove è
    presente una comunità già formata e attiva

2)  A seguito dell’incontro di un utente già inserito nella comunità,
    quest’ultimo gli fornisce un indirizzo web o un diverso tipo di QR
    code (per un ingresso a più alta affidabilità)

**CASO D’USO: Utente entra per la prima volta all’interno di una
comunità**

> L’ingresso in una comunità può variare in base al tipo di metodo
> utilizzato per accedervi.

1)  Nel caso in cui il nuovo utente voglia entrare autonomamente (senza
    l’invito di un altro utente già appartenente a tale comunità), si
    procede con un onboarding progressivo in quanto si vuole evitare di
    dare il via libera a chiunque venga a contatto con il mezzo
    d’accesso. La richiesta d’ingresso quindi dev’essere intercettata da
    un utente preesistente nella comunità che analizzerà la richiesta,
    nella quale il nuovo utente deve esporre la sua identità (**\***) e
    le proprie risorse tecnologiche che utilizza

2)  Se un nuovo utente entra in contatto con una comunità a seguito di
    uno scambio di informazioni con qualcun altro già al suo interno,
    allora la fase di richiesta viene omessa, ma comunque il nuovo
    utente deve comunicare a chi è in ascolto la sua identità (**\***) e
    le sue risorse

> In seguito, il nuovo nodo comunque non potrà sfruttare a pieno la rete
> e le sue risorse, sempre per non fidarsi pienamente di nuovi utenti.
> Infatti, accederà alla comunità con capacità limitate:

- rate limit (es. pubblicazione di solo un’informazione al giorno);

- comunicare solo con alcuni peer (i più vicini);

- non poter propagare informazioni ritenute critiche;

- non poter votare nelle decisioni della comunità;

- non poter introdurre altri nodi;

- avere accesso solo a una parte delle informazioni pubblicate.

> Se a seguito di un periodo sotto queste limitazioni, l’utente e i suoi
> contributi vengono visti positivamente dagli altri nodi, allora potrà
> sfruttare la rete senza limitazioni. Altrimenti, se l’utente appena
> entrato presenta comportamenti anomali o che minano alla sicurezza
> della comunità, questo può essere isolato ed escluso dalle
> comunicazioni interne.
