**Limiti dell’interfaccia browser HTML**

**1) Perché un’interfaccia browser non basta**

Il browser può mostrare l’interfaccia e gestire alcune funzioni offline,
ma non fornisce tutte le capacità necessarie a una rete DTN tra
smartphone.

- **Non può gestire Wi‑Fi Direct tramite un’API web standard.** Su
  Android questa funzione richiede le API native del sistema operativo.
  [Documentazione Android Wi‑Fi
  Direct](https://developer.android.com/develop/connectivity/wifi/wifip2p)

- **Web Bluetooth non sostituisce un servizio radio nativo.** Ha
  disponibilità limitata tra i browser, richiede il consenso dell’utente
  e si concentra sull’interazione con periferiche BLE. Non è una base
  affidabile per la scoperta continua e la sincronizzazione tra
  telefoni. [Documentazione Web
  Bluetooth](https://developer.mozilla.org/en-US/docs/Web/API/Web_Bluetooth_API)

- **La pagina deve essere raggiungibile per essere caricata.** Un QR che
  contiene un link non può aprire una pagina web se non c’è Internet,
  rete locale o una copia dell’applicazione già disponibile sul
  dispositivo.

- **Il browser non è adatto a fare da nodo persistente della rete.** La
  sincronizzazione in background e la conservazione a lungo termine di
  identità, chiavi e dati non sono abbastanza uniformi da farci
  affidamento per il servizio principale.

Quindi il browser può essere il punto d’ingresso e l’interfaccia; per
scoprire telefoni vicini, trasferire dati via Wi‑Fi Direct e partecipare
alla rete store-carry-and-forward serve un’app con un componente nativo.

**2) Cosa è possibile fare solo tramite web**

Senza installare l’app, un utente può usare i servizi che dipendono da
una pagina web raggiungibile:

- aprire il link o il QR d’ingresso e leggere le informazioni pubbliche
  della comunità;

- consultare regole, topic e contenuti resi disponibili via web;

- compilare e inviare una richiesta d’ingresso, se la connessione
  necessaria è disponibile;

- preparare post o contributi e, se la PWA è già stata caricata,
  conservarli temporaneamente per un successivo invio;

- scegliere gli argomenti d’interesse e consultare lo stato della
  richiesta.

Queste funzioni **non permettono al browser di sincronizzare
direttamente con i telefoni vicini**. Se non c’è un server
raggiungibile, il browser può mostrare solo i dati che ha già caricato o
salvato localmente; i contributi in coda non arrivano agli altri membri
finché non torna una connessione compatibile.

**3) Caso d’uso: nuovo utente senza app che vuole entrare in una
comunità**

**Attore principale:** nuovo utente con smartphone e browser, ma senza
l’app del progetto.\
**Obiettivo:** informarsi sulla comunità e avviare l’ingresso.\
**Risultato atteso:** l’utente può completare l’ingresso e partecipare
alla rete solo dopo aver installato l’app.

**Flusso principale**

1.  L’utente inquadra il QR pubblico della comunità. Il codice apre la
    pagina web d’ingresso, ospitata su un indirizzo raggiungibile.

2.  Il browser mostra descrizione, regole, argomenti e modalità di
    ammissione. L’utente può consultare queste informazioni senza
    installare l’app.

3.  L’utente sceglie tra accesso tramite invito e richiesta autonoma.
    Nel secondo caso compila una richiesta con uno pseudonimo e gli
    argomenti a cui desidera contribuire.

4.  La pagina spiega che il browser può avviare la procedura, ma non può
    collegare direttamente il telefono alla rete radio della comunità.

5.  L’utente installa l’app del progetto. Se la pagina è online, può
    seguire il collegamento di installazione; in alternativa, un membro
    può fornirgli il QR d’invito da importare nell’app.

6.  Nell’app, il dispositivo genera e conserva l’identità crittografica
    dell’utente. L’utente completa o conferma la richiesta e la invia
    alla comunità.

7.  Un membro autorizzato valuta la richiesta. Se approvata, l’utente
    riceve una credenziale con permessi iniziali limitati.

8.  L’utente seleziona i topic di interesse e inizia a consultare e
    contribuire ai contenuti consentiti.

9.  Quando incontra altri membri, l’app usa il componente nativo per
    scoprirli e sincronizzare i dati tramite i trasporti disponibili,
    come Bluetooth e Wi‑Fi Direct.

**Caso senza Internet**

Se il QR rimanda a una pagina non raggiungibile e sul telefono non è già
disponibile l’applicazione web, il browser non può avviare l’onboarding.
Un membro può fornire un invito con un QR, ma il nuovo utente dovrà
comunque installare l’app per partecipare alla sincronizzazione DTN.

**In sintesi:** il web consente di scoprire la comunità e preparare la
richiesta; l’app serve per finalizzare l’identità di rete e usare le
funzioni di comunicazione diretta tra smartphone.
