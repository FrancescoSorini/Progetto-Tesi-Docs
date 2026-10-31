**Modulo 1 — Accesso e onboarding**

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case A01</th>
<th>Consultare le informazioni della comunità</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>Il nuovo utente consulta il manifesto, le regole e i topic
disponibili prima di decidere se entrare nella comunità</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Nuovo utente senza app installata</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>L’utente dispone di uno smartphone con browser</p></li>
<li><p>La pagina è raggiungibile tramite Internet o gateway
locale</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>L’utente apre il QR code o l’indirizzo della comunità</p></li>
<li><p>Il sistema mostra manifesto e informazioni disponibili</p></li>
<li><p>L’utente consulta regole, modalità di accesso e topic</p></li>
<li><p>4. Il sistema mostra provenienza e data di aggiornamento, se
disponibili</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Pagina non raggiungibile:</strong> il sistema
informa l’utente; può riprovare quando il gateway è disponibile</p></li>
<li><p><strong>A2 — Informazioni non aggiornate:</strong> il sistema
mostra la data dello snapshot e segnala che i topic potrebbero essere
incompleti</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>L’utente ha consultato le informazioni disponibili</p></li>
<li><p>Non è stato creato un account né un’identità
crittografica</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case A02</th>
<th>Preparare una richiesta d’accesso tramite Web</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>L’utente compila una bozza d’ingresso indicando uno pseudonimo e i
topic d’interesse</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Nuovo utente senza app installata</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>La pagina Web è raggiungibile</p></li>
<li><p>È disponibile un elenco di topic da presentare
all’utente</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>L’utente apre il modulo d’ingresso</p></li>
<li><p>Inserisce uno pseudonimo e seleziona i topic</p></li>
<li><p>Il sistema verifica i campi obbligatori</p></li>
<li><p>Il sistema prepara la bozza per l’eventuale importazione
nell’app</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Campi mancanti:</strong> il sistema li segnala e non
prepara la bozza</p></li>
<li><p><strong>A2 — Connessione interrotta:</strong> la bozza non viene
inviata al gateway; l’utente può riprovare o inserire i dati
nell’app</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>La bozza è stata preparata, se il servizio Web era
disponibile</p></li>
<li><p>La bozza non crea un’identità digitale e non è ancora diffusa
sulla rete P2P</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case A03</th>
<th>Scaricare e installare l’app</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>L’utente scarica il pacchetto Android previsto dal progetto e avvia
l’installazione</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Nuovo utente</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>Lo smartphone è compatibile con la versione Android
prevista</p></li>
<li><p>L’APK firmato è raggiungibile tramite gateway, Internet o
supporto di distribuzione</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>L’utente apre il collegamento al pacchetto</p></li>
<li><p>Il sistema fornisce l’APK</p></li>
<li><p>Android mostra le conferme necessarie all’installazione</p></li>
<li><p>L’utente autorizza l’installazione e avvia l’app</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Pacchetto non raggiungibile:</strong> il sistema
segnala il problema e l’utente può scegliere un altro canal</p></li>
<li><p><strong>A2 — Firma o versione non verificabile:</strong> l’utente
interrompe l’installazione</p></li>
<li><p><strong>A3 — Installazione rifiutata:</strong> l’onboarding
nell’app non può proseguire</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>L’app è installata e può essere avviata, oppure l’installazione è
rimasta incompleta</p></li>
<li><p>Il download da solo non inserisce l’utente nella rete
P2P</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case A04</th>
<th>Predisporre un punto d’accesso locale</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>Un membro rende disponibili localmente la pagina informativa e, se
previsto, il pacchetto dell’app</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Membro della comunità che ospita il punto d’accesso</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>Il membro dispone della pagina e del pacchetto da
distribuire</p></li>
<li><p>Il dispositivo o server locale può ospitare il servizio e ha le
autorizzazioni necessarie</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>Il membro avvia il punto d’accesso locale</p></li>
<li><p>Il sistema avvia il servizio Web e rende disponibili pagina e
pacchetto</p></li>
<li><p>Il sistema mostra un QR code o i dati per collegarsi</p></li>
<li><p>Il nuovo utente si collega e apre la pagina locale</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Hotspot non avviabile:</strong> il sistema segnala
l’errore; il membro può usare un altro gateway</p></li>
<li><p><strong>A2 — Servizio Web non disponibile:</strong> pagina o
pacchetto non possono essere forniti</p></li>
<li><p><strong>A3 — Gateway interrotto:</strong> consultazione e
download non possono proseguire</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>Il punto d’accesso è disponibile finché resta attivo</p></li>
<li><p>La rete locale non implica accesso a Internet né apertura
automatica del captive portal</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case A05</th>
<th>Entrare autonomamente nella comunità</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>L’utente senza invito crea un account nell’app e presenta una
richiesta d’ingresso con privilegi iniziali limitati</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Nuovo utente</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>L’app è installata e avviabile</p></li>
<li><p>L’utente non dispone di un invito personale valido</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>L’utente seleziona l’accesso autonomo</p></li>
<li><p>Inserisce o conferma pseudonimo e topic d’interesse</p></li>
<li><p>L’app genera l’identità crittografica locale</p></li>
<li><p>L’app crea il profilo iniziale [PROBATION]</p></li>
<li><p>L’app memorizza la richiesta e mostra che attende una
sincronizzazione P2P</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Bozza Web disponibile:</strong> l’app importa i dati
e chiede conferma</p></li>
<li><p><strong>A2 — Nessuna bozza:</strong> l’utente inserisce i dati
manualmente</p></li>
<li><p><strong>A3 — Nessun peer raggiungibile:</strong> la richiesta
resta in coda localmente</p></li>
<li><p><strong>A4 — Errore nella creazione o nel salvataggio:</strong>
l’app segnala il problema e non completa il profilo</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>Identità e profilo PROBATION sono stati creati
localmente</p></li>
<li><p>La richiesta è pronta per una sincronizzazione futura; la
ricezione immediata da parte della comunità non è garantita</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case A06</th>
<th>Entrare tramite invito di un membro</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>Il nuovo utente importa un invito personale associato a un membro e
crea un account in stato iniziale [PROBATION]</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Nuovo utente</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>L’app è installata</p></li>
<li><p>L’utente dispone dei dati d’invito, non scaduti e
integri</p></li>
<li><p>È definita una regola per verificare l’identità e il ruolo dello
sponsor</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>Il membro mostra o trasferisce l’invito</p></li>
<li><p>Il nuovo utente lo importa nell’app</p></li>
<li><p>L’app verifica formato, firma e scadenza secondo le regole
previste</p></li>
<li><p>L’utente completa i dati richiesti</p></li>
<li><p>L’app genera l’identità, crea il profilo PROBATION, associa lo
sponsor e memorizza la richiesta</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Invito scaduto o alterato:</strong> l’app lo rifiuta
e invita a richiederne uno nuovo</p></li>
<li><p><strong>A2 — Sponsor non verificabile:</strong> l’app non
considera l’invito autenticato</p></li>
<li><p><strong>A3 — Nessun peer raggiungibile:</strong> la richiesta
resta in coda per un incontro successivo</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>Il profilo PROBATION è stato creato localmente</p></li>
<li><p>Il riferimento allo sponsor è salvato se l’invito è
verificato</p></li>
<li><p>L’invito non assegna automaticamente lo stato
FULL_MEMBER</p></li>
</ul></td>
</tr>
</tbody>
</table>

**Modulo 2 — Scoperta delle risorse**

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case R01</th>
<th>Selezionare i topic d’interesse</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>Un membro sceglie gli argomenti per cui desidera pubblicare o
consultare annunci di risorse</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Membro della comunità</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>L’app è installata e il membro ha un profilo locale</p></li>
<li><p>È disponibile un elenco di topic, anche memorizzato in
precedenza</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>Il membro apre la gestione dei topic</p></li>
<li><p>Il sistema mostra i topic disponibili</p></li>
<li><p>Il membro seleziona o deseleziona gli argomenti</p></li>
<li><p>Il sistema salva localmente le preferenze</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Elenco non aggiornato:</strong> il sistema mostra
provenienza e data, se disponibili</p></li>
<li><p><strong>A2 — Topic non valido:</strong> il sistema rifiuta il
valore</p></li>
<li><p><strong>A3 — Nessuna connessione:</strong> le preferenze sono
salvate localmente e potranno essere usate in seguito</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>Le preferenze sono salvate sul dispositivo</p></li>
<li><p>La loro applicazione alla sincronizzazione segue le regole di
condivisione definite dal progetto</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case R02</th>
<th>Pubblicare o aggiornare un annuncio di risorsa</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>Un membro pubblica un annuncio su una risorsa o competenza e può in
seguito pubblicarne uno aggiornato</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Fornitore di risorse</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>Il membro dispone di un’identità locale</p></li>
<li><p>Ha selezionato un topic ammesso e può pubblicare secondo le
regole del proprio profilo</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>Il membro sceglie Pubblica risorsa.</p></li>
<li><p>Seleziona il topic e inserisce i dati richiesti.</p></li>
<li><p>Controlla e conferma l’annuncio.</p></li>
<li><p>L’app associa l’annuncio all’identità del membro e lo memorizza
localmente.</p></li>
<li><p>L’app lo rende disponibile per una futura
sincronizzazione</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Dati obbligatori mancanti:</strong> l’app segnala i
campi da completare</p></li>
<li><p><strong>A2 — Pubblicazione non consentita:</strong> l’app rifiuta
l’operazione o applica i limiti del profilo</p></li>
<li><p><strong>A3 — Risorsa aggiornata:</strong> l’app pubblica una
nuova versione secondo la regola di aggiornamento definita</p></li>
<li><p><strong>A4 — Errore locale:</strong> l’annuncio non viene salvato
e l’utente può riprovare</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>L’annuncio è memorizzato localmente e pronto per la
sincronizzazione</p></li>
<li><p>Gli altri nodi non sono necessariamente già in possesso
dell’annuncio</p></li>
<li><p>La firma attesta l’autore del messaggio, non la veridicità della
risorsa dichiarata</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case R03</th>
<th>Sincronizzare annunci con un peer incontrato</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>Due nodi compatibili scambiano gli annunci di risorse consentiti
dalle rispettive regole di condivisione</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Altro nodo Briar della comunità</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>Entrambi i dispositivi hanno l’app attiva e un trasporto
compatibile</p></li>
<li><p>I nodi riescono a stabilire una sessione</p></li>
<li><p>È configurata una regola che consente lo scambio dei gruppi o
topic interessati</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>L’app rileva o accetta la connessione con il peer</p></li>
<li><p>I nodi stabiliscono una sessione sicura</p></li>
<li><p>Ciascun nodo seleziona gli annunci candidati allo
scambio</p></li>
<li><p>I nodi scambiano gli annunci selezionati</p></li>
<li><p>Il ricevente verifica formato, provenienza e scadenza</p></li>
<li><p>Il ricevente salva o aggiorna gli annunci validi nel registro
locale</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Connessione interrotta:</strong> i dati incompleti
non sono considerati acquisiti; gli annunci in coda potranno essere
riproposti</p></li>
<li><p><strong>A2 — Annuncio duplicato:</strong> il ricevente evita di
creare una copia</p></li>
<li><p><strong>A3 — Annuncio non valido o scaduto:</strong> il ricevente
lo scarta o lo marca come non attivo</p></li>
<li><p><strong>A4 — Topic non condiviso o nessun annuncio
candidato:</strong> non viene trasferito alcun annuncio</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>Il ricevente dispone localmente degli annunci accettati</p></li>
<li><p>La ricezione da parte di questo peer non garantisce la
propagazione all’intera comunità</p></li>
<li><p>Gli annunci potranno essere inoltrati in incontri successivi se
le regole lo consentono</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table style="width:79%;">
<colgroup>
<col style="width: 17%" />
<col style="width: 61%" />
</colgroup>
<thead>
<tr>
<th style="text-align: right;">Use Case R04</th>
<th>Consultare il registro locale delle risorse</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: right;">Descrizione</td>
<td>Un membro consulta gli annunci memorizzati sul dispositivo,
filtrandoli per topic e validità</td>
</tr>
<tr>
<td style="text-align: right;">Attore</td>
<td>Membro interessato alle risorse</td>
</tr>
<tr>
<td style="text-align: right;">Pre-condizioni</td>
<td><ul>
<li><p>L’app è installata e il registro locale è disponibile</p></li>
<li><p>Il dispositivo contiene annunci ricevuti o pubblicati; non è
richiesta una connessione</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Base</td>
<td><ol type="1">
<li><p>L’utente apre la consultazione delle risorse</p></li>
<li><p>Seleziona un topic o criterio di ricerca</p></li>
<li><p>Il sistema cerca gli annunci nel registro locale</p></li>
<li><p>Il sistema distingue o esclude gli annunci scaduti</p></li>
<li><p>L’app mostra i risultati con autore e data dell’annuncio</p></li>
</ol></td>
</tr>
<tr>
<td style="text-align: right;">Flusso Alternativo</td>
<td><ul>
<li><p><strong>A1 — Nessun risultato:</strong> il sistema segnala che
sul dispositivo non risultano annunci per il topic</p></li>
<li><p><strong>A2 — Annunci scaduti:</strong> il sistema non li presenta
come risorse attive</p></li>
<li><p><strong>A3 — Dati non recenti:</strong> il sistema mostra la data
dell’annuncio e invita a verificarne la disponibilità</p></li>
</ul></td>
</tr>
<tr>
<td style="text-align: right;">Post-Condizioni</td>
<td><ul>
<li><p>L’utente ha consultato il registro locale</p></li>
<li><p>La consultazione non aggiorna i dati degli altri nodi e non
garantisce che la risorsa sia ancora disponibile</p></li>
</ul></td>
</tr>
</tbody>
</table>
