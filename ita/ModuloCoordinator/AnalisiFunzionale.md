# Modulo Coordinator - Analisi Funzionale

1.  [Il ruolo del modulo Coordinator](#Ruolo_coordinator)
1.  [Il servizio Coordinator](#Servizio_coordinator)
    1.  [Richieste previste](#Richieste_previste)
        1.  [Incontrata una rete](#Incontrata_rete)
        1.  [Avvio ingresso in altra rete](#Avvio_ingresso)
        1.  [Prenota un posto](#Prenota_un_posto)
        1.  [Confermato ingresso in altra rete](#Confermato_ingresso)
1.  [Richieste al diretto vicino di accesso al servizio Coordinator](#Richieste_al_diretto_vicino)
    1.  [Ingresso in diversa rete](#Per_ingresso)
    1.  [Ingresso come risoluzione di uno split di g-nodo](#Per_split)
1.  [Collaborazioni con gli altri moduli](#Collaborazioni)
1.  [Requisiti](#Requisiti)
1.  [Deliverables](#Deliverables)
1.  [Classi e interfacce](#Classi_e_interfacce)

## <a name="Ruolo_coordinator"></a>Il ruolo del modulo Coordinator

Il modulo cerca di fare sì che un singolo g-nodo di livello *l*, sebbene costituito da un numero di singoli
nodi, abbia un comportamento coerente come singola entità.

Il modulo fa uso delle [tasklet](../Librerie/TaskletSystem.md), un sistema di multithreading cooperativo.

Il modulo fa uso del framework [ZCD](../Librerie/ZCD.md), precisamente appoggiandosi ad una libreria intermedia
prodotta con questo framework per formalizzare i metodi remoti usati nel demone *ntkd* per comunicare con i
diretti vicini.

Il modulo fa uso diretto delle classi e dei servizi forniti dal modulo [PeerServices](../ModuloPeers/AnalisiFunzionale.md).
In particolare, esso realizza un servizio peer-to-peer, chiamato appunto Coordinator, per mezzo del quale svolge
alcuni dei suoi compiti.

Siccome chiamiamo con lo stesso nome Coordinator sia il modulo presente su ogni nodo sia il servizio peer-to-peer
che rappresenta un g-nodo, bisogna che il lettore faccia attenzione al contesto. Di solito se parliamo
di Coordinator di un g-nodo (dal livello 1 fino all'intera rete) ci riferiamo al servizio, cioè
al nodo che al momento viene identificato come servente.

## <a name="Servizio_coordinator"></a>Il servizio Coordinator

Il servizio Coordinator è un servizio non opzionale, cioè tutti i nodi partecipano attivamente. Si tratta di un
servizio che mantiene un database distribuito della tipologia a chiavi fisse.

Lo scopo del servizio Coordinator è quello di realizzare una sorta di memoria condivisa di un g-nodo *g*. Quando
un nodo *n* vuole accedere alla memoria condivisa del suo g-nodo *g* di livello *l*, con 0 < *l* ≤ *levels*, fa
una richiesta al servizio Coordinator usando come chiave il livello *l*.

Il livello *l* è maggiore di 0 poiché non serve realizzare una memoria condivisa per il singolo nodo. Può essere
uguale al numero dei livelli, poiché è necessario avere una memoria condivisa anche per l'unico g-nodo di livello
*l* = *levels* che costituisce l'intera rete.

Lo spazio delle chiavi definito dal servizio Coordinator è appunto l'insieme dei valori che può assumere *l*.

La funzione *h<sub>p</sub>* è definita dal servizio in modo da dare ai dati la visibilità locale circoscritta al
g-nodo in esame. In altre parole, il nodo corrente può contattare solo il Coordinator di uno dei suoi g-nodi; sia
l'hash-node che il nodo che risponde si trovano all'interno del g-nodo stesso.

### <a name="Richieste_previste"></a>Richieste previste

Elenchiamo tutte le richieste che si possono fare al Coordinator.

#### <a name="Incontrata_rete"></a>Incontrata una rete

Questa richiesta *r* fatta al Coordinator dell'intera rete indica che è stata incontrata una diversa rete *J*.
Le informazioni riguardo la rete *J* che sono riportate in questa richiesta sono state ottenute dalle
conoscenze dirette di un singolo nodo (un diretto vicino del client del servizio) che indichiamo con *v*.  
Indichiamo con *G* la rete corrente. Indichiamo con *n* il singolo nodo in *G* che ci ha contattato come client del servizio.

I membri di *r* sono:

*   `int64 netid` = Identificativo della rete *J*.
*   `List<int> gsizes` = Lista che descrive la topologia della rete *J*. Da essa si ricava `levels`.
*   `List<int> gnode_n_nodes` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_n_nodes[i-1]` è il
    numero approssimativo di singoli nodi dentro *g<sub>i</sub>(v)*.
*   `List<int> gnode_pos` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_pos[i-1]` è la
    posizione al livello `i-1` di *v* dentro *g<sub>i</sub>(v)*.
*   `List<int> gnode_n_free_pos` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_n_free_pos[i-1]` è il
    numero di posizioni libere dentro *g<sub>i</sub>(v)*.
*   `int minimum_lvl` = Livello minimo a cui il singolo nodo *n* è disposto a fare ingresso. Infatti il nodo *n*
    potrebbe essere un gateway verso una rete privata in cui si vogliono adottare diversi meccanismi di
    assegnazione di indirizzi e routing. In questo caso il gateway potrebbe volere una assegnazione di un
    g-nodo di livello tale (considerando la topologia della rete *J*) da poter disporre di un certo spazio
    (numero di bits) per gli indirizzi interni.

Per il momento assumiamo che nessun nodo invia una richiesta di fare ingresso in una rete con topologia
diversa da quella di *G*.

Il Coordinator dell'intera rete *G* non ha particolari informazioni sui g-nodi (di livello inferiore a
`levels`) a cui appartiene *n*. Ma tali informazioni comunque non gli sono necessarie. Esso infatti dovrà
solo individuare, sulla base delle informazioni ricevute circa i g-nodi di *v* in *J*, un livello
a cui cercare di far entrare un g-nodo di *n* dentro un g-nodo di *v*.

Questa richiesta viene fatta al servizio Coordinator, in particolare al Coordinator dell'intera rete, per
fare in modo che sia "tutta la rete" come entità atomica a venire interpellata.

È importante che la decisione venga presa dalla rete come entità atomica, poiché il fatto che un
singolo nodo abbia rilevato il contatto con una diversa rete non esclude che le due reti siano
entrate in contatto più o meno nello stesso momento in diversi punti. Occorre evitare di avviare
più operazioni di ingresso che coinvolgono gli stessi g-nodi.  
Sarebbe desiderabile, per quanto possibile, cercare di verificare la possibilità di fare ingresso
sfruttando il punto di contatto migliore.

Sebbene la richiesta venga fatta come detto al Coordinator della rete *G*,
in effetti la strategia di ingresso non è di pertinenza del modulo Coordinator. Per questo viene utilizzato
un delegato passato al modulo dal suo utilizzatore sotto forma di una istanza dell'interfaccia IEnterNetworkHandler.

La risposta ottenuta dal delegato consiste in due valori:

*   `int lvl` - il livello del g-nodo di *n* che dovrebbe tentare l'ingresso in *J*.
*   `uint hash_net` - un identificativo della rete *J* che il modulo Coordinator (pur non conoscendo i dettagli di un network-id)
    può considerare come univoco.

Quindi il Coordinator considera il g-nodo *g<sub>lvl</sub>(n)* come coinvolto in questo ingresso in *J* attraverso il
singolo nodo *n*.

Assumiamo che questa richiesta sia la prima pervenuta che coinvolge il g-nodo *g<sub>lvl_0</sub>(n)*. Allora
il Coordinator della rete *G* registra nella memoria condivisa (del g-nodo di tutta la rete) le informazioni di
questa richiesta e della valutazione ottenuta dal delegato e inoltre
il tempo limite (Timer serializzabile) entro cui intende rispondere. Poi risponde al client *n* con una eccezione AskAgainError
che istruisce il nodo *n* di ripetere la richiesta dopo aver atteso alcuni istanti.  
Siccome c'è una scrittura nella memoria condivisa, prima di rispondere si avvia una nuova tasklet
per provvedere alle repliche con il meccanismo fornito dal modulo PeerServices.

Il tempo limite entro cui il Coordinator intende rispondere, cioè intende prendere una decisione circa
l'ingresso di (una parte di) *G* in *J*, è direttamente proporzionale al numero di singoli nodi presenti
in *G*. È lecito presumere che sia la rete più piccola a voler entrare in quella più grande. Quindi lo
scopo di questa attesa è dare il tempo agli altri singoli nodi di *G*, che potrebbero essere venuti
in contatto con *J* (quasi) simultaneamente, di raggiungere il Coordinator di *G* con la loro proposta.

Supponiamo che nel frattempo giunga al Coordinator della rete *G* una richiesta simile dal nodo *q*
relativa alla rete *J*. Il Coordinator di *G* interroga il delegato e scopre che il g-nodo
coinvolto in questo ingresso, cioè *g<sub>lvl_1</sub>(q)*, interseca (è equivalente, oppure contiene, oppure è contenuto) con
il g-nodo *g<sub>lvl_0</sub>(n)*. Il Coordinator deduce che queste richieste (di *n* e di *q*) vanno considerate insieme
perché sono intersecanti e riguardano la stessa rete *J*. Le due richieste risultano ora collegate
fra di loro nella memoria condivisa.  
Come conseguenza avranno sempre la medesima scadenza. Se `lvl_1` è maggiore di `lvl_0`, ovvero più in generale, se il
livello del g-nodo coinvolto nella richiesta appena pervenuta è maggiore del livello del g-nodo coinvolto in tutte
le richieste ad essa collegate, allora si sceglie un nuovo tempo limite e si aggiorna su tutte le richieste collegate.
Altrimenti il tempo limite che rimane alle richieste precedenti viene mantenuto e usato anche per la richiesta di *q*.  
Dopo aver apportato queste variazioni alla memoria condivisa (e di conseguenza dopo aver avviato una nuova
tasklet per provvedere alle repliche) il Coordinator si accinge a rispondere alla richiesta di *q*.
Se il tempo limite non è ancora scaduto il Coordinator risponde anche a questa richiesta con
una eccezione AskAgainError.

Alla fine arriverà una richiesta di ingresso in *J* tale che il Coordinator di *G* la associerà ad un
gruppo di richieste il cui tempo limite risulta scaduto. A questo punto il Coordinator eleggerà
la migliore fra le soluzioni. Di nuovo, per fare questa scelta si avvarrà del delegato IEnterNetworkHandler.
Poi registrerà la scelta nella memoria condivisa (e provvederà alle repliche).

Dopo aver scelto, se la richiesta proviene dal client *eletto* allora il Coordinator
risponde positivamente, cioè indicando il livello a cui il client deve tentare l'ingresso in *J*.  
In tutti gli altri casi risponde con l'eccezione IgnoreNetworkError che istruisce il nodo client
di non prendere alcuna iniziativa.

In realtà, lo spiegheremo in dettaglio più sotto, i membri di questa richiesta *r* non sono noti al
modulo Coordinator, il quale li riceve come una singola struttura dati e li passa in blocco al
delegato (istanza dell'interfaccia IEnterNetworkHandler).  
Le altre informazioni di cui il modulo Coordinator ha bisogno per gestire questa richiesta sono l'indirizzo
del nodo richiedente *n* e il numero (approssimativo) di singoli nodi presenti in *G*. Entrambe sono
note al Coordinator della rete a prescindere dal contenuto di *r*.

#### <a name="Avvio_ingresso"></a>Avvio ingresso in altra rete

La richiesta *r* di autorizzare l'avvio delle operazioni di ingresso del g-nodo *g* in una diversa rete *J*. Tale
richiesta arriva al nodo Coordinator di *g*.

I membri di *r* sono:

*   `netid` = Identificativo della rete *J*.
*   `lvl` = livello di *g*.

L'arrivo di questa richiesta implica che la topologia della rete *J* dal livello `lvl-1` in giù sia
identica alla topologia di *G*.

Il Coordinator di *g* acquisisce un *lock*.

Poi avvia una tasklet su cui procedere, mentre risponde positivamente al client permettendogli di
proseguire con le operazioni di ingresso di *g* in *J*.

L'acquisizione del lock equivale ad una scrittura nella memoria condivisa, quindi nella nuova tasklet
il Coordinator come prima cosa provvede alle repliche con il meccanismo fornito dal modulo PeerServices.

Poi... **TODO**

#### <a name="Prenota_un_posto"></a>Prenota un posto

La richiesta *r* di prenotare un posto può arrivare ad un nodo *x* come Coordinator di un g-nodo *g* di livello
*l*, con 0 < *l* ≤ *levels*. I membri di *r* sono:

*   `lvl` = livello di *g*.

Il nodo *x* valuta se ci sono posti liberi in *g* considerando la sua mappa dei percorsi. Deve considerare anche
le prenotazioni concesse in precedenza, le quali restano valide per un certo tempo anche se ancora non sono nella
sua mappa perché non sono ancora state confermate da un ETP.

Se un posto è disponibile il nodo *x* lo prenota. Questo equivale ad una scrittura nella memoria condivisa,
quindi è necessario provvedere anche alle repliche con il meccanismo fornito dal modulo PeerServices.

Il nodo che fa la richiesta è un nodo *n* che già appartiene al g-nodo *g*. Questi richiede la prenotazione di
un nuovo posto per conto di un altro nodo suo vicino, *m*, il quale non è ancora in *g* o perfino non è ancora
nella rete.

Nella risposta al nodo *n*, *x* segnala:

*   La posizione assegnata all'interno di *g*.
*   La anzianità della posizione assegnata all'interno di *g*.

Altre informazioni di cui il nodo *m* necessita per fare ingresso in *g* (e eventualmente nella rete) sono
direttamente fornite dal nodo *n* che già le conosce. Queste sono:

*   La topologia della rete.
*   Le posizioni di *g* e dei suoi g-nodi superiori.
*   L'anzianità di *g* e dei suoi g-nodi superiori.

A questo punto il nodo *n* comunica tutte queste informazioni al suo vicino *m*.

Se nessun posto è disponibile il nodo *x* lo segnala con una eccezione, che viene ricevuta da *n*. Anche in
questo caso *n* comunica l'esito al vicino *m*.

#### <a name="Confermato_ingresso"></a>Confermato ingresso in altra rete

La richiesta/segnalazione *r* che sono state completate le operazioni di ingresso del g-nodo *g* in una diversa rete. Tale
richiesta arriva ad un nodo *x* come Coordinator di *g*. **TODO**

## <a name="Richieste_al_diretto_vicino"></a>Richieste al diretto vicino di accesso al servizio Coordinator

In alcuni casi un nodo *n* può voler chiedere ad un suo diretto vicino di accedere al servizio peer-to-peer
del Coordinator.

### <a name="Per_ingresso"></a>Ingresso in diversa rete

Si consideri un nodo *n* che appartiene alla rete *G* e un suo diretto vicino nodo
*v* che appartiene alla rete *J*.  
Supponiamo che - con le modalità viste in "[ingresso in altra rete](#Collaborazioni_ingresso)" - il
nodo *n* riceva dal Coordinator della rete *G* il compito di richiedere l'ingresso del suo g-nodo *g*
di livello *l* dentro *J* tramite il suo arco con *v*.

Ora il nodo *n* comunica direttamente con il servizio Coordinator del livello *l*, cioè di *g*. Il
Coordinator di *g* accetta questa soluzione (vedi la richiesta [avvio-ingresso](#Avvio_ingresso)). Questa operazione consta di alcune parti:

*   Il Coordinator di *g* acquisisce un blocco. Cioè, se erano in coda altre operazioni che collidono
    con questa (ad esempio un'altra operazione di ingresso, *o altro da valutare*) allora attende prima
    di procedere.
*   Acquisito il blocco il Coordinator di *g* avvia una nuova tasklet per fare altre operazioni, ma
    intanto risponde positivamente al nodo *n*.
*   Nella tasklet avviata il Coordinator di *g* aspetta un certo tempo ...

Quando *n* riceve la risposta positiva dal Coordinator di *g*, subito chiede a *v* di trovare un posto per *g* in *J*.

Per rispondere a questa richiesta, il nodo *v* contatta il Coordinator del *suo* g-nodo di
livello *l* + 1, chiamiamolo *h*, chiedendogli di prenotare un posto (vedi la richiesta
[prenota-un-posto](#Prenota_un_posto)). Se ciò non fosse possibile
il Coordinator di *h* risponderebbe con una eccezione a *v* e ci sarebbero altre operazioni da fare
per la ricerca di una migration path; ma in questo esempio assumiamo che il
g-nodo *h* abbia un posto libero per un g-nodo di livello *l*. Allora il Coordinator di *h* prenota
il posto e lo comunica al nodo *v*. Il nodo *v* lo comunica al nodo *n*.

Quello che si voleva evidenziare qui, è che il modulo Coordinator in *n* comunica con il modulo
Coordinator del diretto vicino *v*. Prima di rispondere, il modulo Coordinator in *v* usa il client
del servizio Coordinator per comunicare, non con un diretto vicino, bensì tramite i servizi del
modulo PeerServices, con il Coordinator del suo g-nodo di livello *l* + 1, cioè *h*.

In questo modo il nodo *n*, dialogando con il diretto vicino *v*, è come se avesse dialogato con
l'intero g-nodo *h* come entità unitaria. Inoltre avendo prima il nodo *n* preso accordo con
il Coordinator di *g*, è come se i g-nodi *g* e *h* come entità unitarie avessero dialogato tra loro.

Ora che *n* ha ricevuto la prenotazione, di nuovo comunica con il Coordinator di *g* per informarlo
(vedi la richiesta/segnalazione [confermato-ingresso](#Confermato_ingresso)). Cioè
gli comunica che ogni singolo nodo in *g* dovrà presto attivarsi per formare una nuova identità che
prende posto in *h* (come g-nodo isomorfo di *g*).  
La propagazione delle informazioni necessarie ad ogni singolo nodo avverrà attraverso un diverso
modulo. (*Da valutare* Ogni singolo nodo potrebbe chiedere conferma al Coordinator di *g* dell'esattezza
delle suddette informazioni, ma questo comporterebbe un enorme traffico su *g*).  
Le informazioni suddette che pervengono ad ogni singolo nodo permettono al contempo la trasformazione
del g-nodo *g* in un g-nodo *di connettività* in *G*.

Ricevuta questa segnalazione, il Coordinator di *g*:

*   Il Coordinator di *g* controlla se la richiesta ha l'identificativo del blocco che era
    stato acquisito su *g* ed era in corso.
*   Memorizza le informazioni, rilascia il blocco ed avvia una nuova tasklet per fare altre operazioni, ma
    intanto risponde positivamente al nodo *n*.
*   Nella tasklet avviata il Coordinator di *g* aspetta un certo tempo ...

### <a name="Per_split"></a>Ingresso come risoluzione di uno split di g-nodo

Si consideri un nodo *n* che fa parte di un g-nodo *g* di livello *l*. Il g-nodo *g* fa a sua volta parte di
un g-nodo *h* di livello *l* + 1. Supponiamo che il g-nodo *g* diventi disconnesso, cioè avviene lo split del
g-nodo, mentre *h* è ancora connesso. Supponiamo che l'isola in cui si trova *n* (chiamiamola *g1*) sia una di quelle che non
contengono il nodo più anziano. Supponiamo che *n* abbia un vicino *v* che appartiene a *h* ma non appartiene
a *g*. Per questo il nodo *n* riceve da *v* l'informazione che tutta la sua isola (di livello *l*) deve
cambiare indirizzo.

In questo momento nessun nodo all'interno dell'isola *g1* ha un indirizzo valido in *h*. Quindi
non può comunicare con il server del servizio Coordinator del suo g-nodo di livello *l* + 1, o superiori.
Però si forma automaticamente un nuovo server del servizio Coordinator per *g1*.

Ci troviamo di nuovo in una situazione simile alla precedente. Il nodo *n* comunica direttamente
con il servizio Coordinator di *g1* proponendo un ingresso facilitato dal nodo diretto vicino *v*. Il
Coordinator di *g1* accetta questa soluzione.

A questo punto *n* chiede a *v* di trovare un posto per *g1* in *G*. Per questa richiesta il nodo *v* comunica
con il servizio Coordinator del *suo* g-nodo di livello *l* + 1, che è l'originale g-nodo *h*.

L'obiettivo finale di queste comunicazioni (possono essere necessarie più di una) con il server
del servizio Coordinator sarà ovviamente la prenotazione di un posto di livello *l* in *G*.

## <a name="Collaborazioni"></a>Collaborazioni con gli altri moduli

### <a name="Collaborazioni_ingresso"></a>Ingresso in altra rete

Si consideri un nodo *n* che appartiene alla rete *G*. Questa è una generalizzazione,
che comprende ad esempio il caso di un singolo nodo che compone una intera rete.

Un modulo del nodo *n* che appartiene alla rete *G* si avvede del vicino *v* di altra rete *J*.

Un modulo del nodo *n* chiede e ottiene dal vicino *v* una struttura dati che descrive *J* come è vista da *v*.  
Questa struttura contiene:

*   `int64 netid` = Identificativo della rete *J*.
*   `List<int> gsizes` = Lista che descrive la topologia della rete *J*. Da essa si ricava `levels`.
*   `List<int> gnode_n_nodes` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_n_nodes[i-1]` è il
    numero di singoli nodi dentro *g<sub>i</sub>(v)*.
*   `List<int> gnode_pos` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_pos[i-1]` è la
    posizione al livello `i-1` di *v* dentro *g<sub>i</sub>(v)*.
*   `List<int> gnode_n_free_pos` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_n_free_pos[i-1]` è il
    numero di posizioni libere dentro *g<sub>i</sub>(v)*.

Poi quel modulo confronta la descrizione di *J* data da *v* con la rete *G* come è vista da *n*
e decide che *G* deve entrare in *J*.

**Importante** Questa prima disamina delle caratteristiche di *J* e di *G* avviene sulla base delle
conoscenze di *n* e di *v*. Non va bene importunare il Coordinator della rete (che può essere anche
grande) ogni qualvolta un singolo nodo della rete incontra un vicino che non appartiene ancora alla
rete. Infatti può essere subito evidente dalle conoscenze dei singoli nodi che si vengono ad
incontrare quale sia la rete più piccola, quella cioè che cercherà un posto all'interno dell'altra.

Allora quel modulo aggiunge un'altra informazione alla struttura dati di cui sopra:

*   `int minimum_lvl` = Livello minimo a cui il singolo nodo *n* è disposto a fare ingresso.

Ora nel nodo *n* l'utilizzatore del modulo Coordinator richiama il suo metodo `prepare_enter`.  
Al metodo viene passata la struttura dati di cui sopra. Ma il contenuto di questa struttura non è di
pertinenza del modulo Coordinator, il quale sa solo che si tratta di un oggetto serializzabile.  
L'obiettivo di questo metodo è indicare all'utilizzatore del modulo Coordinator se deve tentare l'ingresso
in *J* e a quale livello.

L'esecuzione di `prepare_enter` del modulo Coordinator consiste in questo:

Viene preparato un client del servizio Coordinator. Su questo viene chiamato il metodo `prepare_enter`
passandogli la stessa struttura dati di cui sopra.

La classe client del servizio sa che questo metodo usa come chiave *k* con `k.lvl = levels`. Cioè va contattato
il Coordinator dell'intera rete.

La classe client nel suo metodo `prepare_enter` prepara una richiesta *r* = "[incontrata-rete](#Incontrata_rete)"
che comprende la struttura dati (ovvero l'istanza di Object serializzabile) di cui sopra.

Poi invia la richiesta *r* e interpreta la risposta. Il metodo `prepare_enter` della classe client può restituire:

*   `int ret`.
*   Eccezione AskAgainError.
*   Eccezione IgnoreNetworkError.

Se riceve AskAgainError, il metodo `prepare_enter` del modulo Coordinator attende alcuni istanti poi riprova.
Altrimenti restituisce il risultato. Quindi questo metodo può restituire:

*   `int ret` = livello a cui fare ingresso.
*   Eccezione IgnoreNetworkError.

Se il risultato è il livello *ret*, questo significa: il nodo *n* richieda l'ingresso del g-nodo *g<sub>ret</sub>(n)*
dentro il g-nodo *g<sub>ret+1</sub>(v)* in *J* tramite il suo arco con *v*.  
Se è l'eccezione IgnoreNetworkError, questo significa: il nodo *n* non prenda alcuna iniziativa.

Come abbiamo visto trattando la richiesta "[incontrata una rete](#Incontrata_rete)", il Coordinator
chiama un delegato (il metodo `choose_target_level` dell'interfaccia IEnterNetworkHandler)
passandogli le informazioni ricevute dal suo client.  
Con questo intendiamo evidenziare che il contenuto di queste informazioni può essere nascosto al modulo
Coordinator, ed essere quindi indipendente. Ad esempio, se il modulo di *n* che recupera da *v* le
informazioni sulla sua rete *J* cambia la sua logica e di conseguenza la cambia il modulo che si occupa
di implementare l'interfaccia IEnterNetworkHandler, ciò non dovrebbe comportare alcun cambiamento al
codice del modulo Coordinator.  
Per fare questo passiamo al metodo `prepare_enter` del modulo Coordinator un argomento di tipo
Object serializzabile. La vera classe di tale oggetto dovrà essere nota ai moduli che forniscono/recuperano
le informazioni sulla rete *J* e al modulo che implementa l'interfaccia IEnterNetworkHandler, ma non
al modulo Coordinator, che si occupa solo di passarla da un modulo del nodo *n* ad un modulo del nodo
Coordinator della rete *G*.

Il nodo Coordinator della rete riceve automaticamente l'indirizzo del nodo client *n*. Conosce da solo la
topologia della rete *G* e il numero approssimativo di singoli nodi in essa. Queste informazioni sono
sfruttate direttamente dal nodo Coordinator per raggruppare le richieste e valutare il tempo di
attesa prima di decidere quale richiesta approvare.  
Inoltre il nodo Coordinator lascia che sia l'utilizzatore del modulo Coordinator nel nodo *n* ad occuparsi di
validare la conformità delle topologie di *G* e di *J*.  
In conclusione il contenuto della richiesta può essere semplicemente:

*   `Object network_data` = un oggetto serializzabile la cui classe è nota al delegato IEnterNetworkHandler.

## <a name="Requisiti"></a>Requisiti

Il nodo quando crea una identità (la prima per avviare il sistema o le successive a seguito di ingressi
o migrazioni) crea una istanza del modulo Coordinator fornendo:

*   Se si tratta di una identità successiva alla prima, per ingresso o migrazione, il livello del g-nodo
    (di cui l'identità fa parte come nodo) che compie in blocco questa operazione
*   Il livello a cui si è costituito un g-nodo nuovo.
*   Se si tratta di una identità successiva alla prima, un riferimento alla istanza precedente del modulo Coordinator.

Durante le sue operazioni, il modulo viene informato quando il nodo ha completato la fase di bootstrap.
In quello stesso momento gli vengono forniti:

*   L'istanza di PeersManager.
*   Mappa delle posizioni libere/occupate ai vari livelli.

## <a name="Deliverables"></a>Deliverables

Fornisce metodi per:

*   Chiedere ad un vicino *v*, dato uno stub per contattarlo, informazioni sulla rete *J* in cui si trova.  
    Questo metodo può rilanciare l'eccezione CoordinatorMemoryNotReadyError se il vicino *v* non ha ancora
    completato la fase di boostrap (vedi modulo [QSPN](../ModuloQspn/EsplorazioneRete.md#Rete_esplorata)).
    Infatti il modulo Coordinator nel nodo *v* non ha ancora ricevuto l'istanza di ICoordinatorMap da cui
    recupera le informazioni richieste.  
    Il vicino *v* potrebbe non appartenere alla stessa rete del nodo corrente *n*. Quindi anche avere una
    diversa topologia. In questo caso le informazioni servono a decidere se e come fare ingresso nell'altra
    rete.  
    Oppure il vicino *v* potrebbe appartenere alla stessa rete del nodo corrente *n*. In questo caso le informazioni
    servono a ... **TODO**.  
    Questa operazione si implementa nel metodo `get_network_info`. Le informazioni sono richieste al
    vicino *v* attraverso il metodo remoto `ask_network_info`. Il vicino *v* compila le informazioni sulla
    base delle sue dirette conoscenze, senza contattare i singoli Coordinator. Le informazioni sono:

    *   `int64 netid` = Identificativo della rete *J*.
    *   `List<int> gsizes` = Lista che descrive la topologia della rete *J*. Da essa si ricava `levels`.
    *   `List<Object> gnode_data` = Lista di informazioni sui g-nodi ai vari livelli secondo la posizione di *v*.  
        Per ogni livello *i* da `levels` a 1 l'elemento `gnode_data[i-1]` contiene:
        *   `int n_nodes` il numero approssimativo di singoli nodi dentro il g-nodo di livello `i` a cui appartiene *v*.
        *   `int pos` la posizione al livello `i-1` di *v* in *J*.
        *   `int n_free_pos` Il numero di posizioni libere (per un g-nodo di livello `i-1`) dentro il g-nodo di livello `i` a cui appartiene *v*.
*   Chiedere ad un vicino *v*, dato uno stub per contattarlo, quanti posti vede liberi (nella sua mappa,
    senza contattare i singoli Coordinator) nei suoi g-nodi. Metodo `get_neighbor_map`.  
    Questo metodo può rilanciare l'eccezione CoordinatorStubNotWorkingError se la comunicazione con il
    vicino non riesce.  
    Questo metodo può rilanciare l'eccezione CoordinatorMemoryNotReadyError se il vicino *v* non ha ancora
    completato la fase di boostrap (vedi modulo [QSPN](../ModuloQspn/EsplorazioneRete.md#Rete_esplorata)).
    Infatti il nodo *v* non è in grado di rispondere alle richieste dell'interfaccia ICoordinatorMap,
    quindi il modulo Coordinator nel nodo *v* non ha ancora ricevuto l'istanza di tale interfaccia.  
    Se invece non sono rilanciate eccezioni, il metodo restituisce una istanza di ICoordinatorNeighborMap.  
    Dalla risposta deve essere possibile leggere queste informazioni:

    *   Il numero di livelli nella topologia.
    *   La gsize di ogni livello.
    *   Il numero di posti liberi in ogni livello.

    Quando un nodo *n* chiede al vicino *v* quanti posti liberi vede nella sua mappa, non assumiamo che il
    nodo *v* appartenga già alla stessa rete di *n*; quindi nemmeno che abbiano la stessa topologia.  
    La topologia della rete in cui si vuole fare ingresso è importante che sia nota. Infatti il nodo richiedente
    potrebbe essere un gateway verso una rete privata in cui si vogliono adottare diversi meccanismi di
    assegnazione di indirizzi e routing. In questo caso il gateway potrebbe volere una assegnazione di un
    g-nodo di livello tale da poter disporre di un certo spazio (numero di bits) per gli indirizzi interni.  
    Il numero di posti liberi in un dato livello potrebbe essere una informazione eccessiva. Probabilmente
    al nodo richiedente è sufficiente sapere se c'è almeno un posto o no. Per ora manteniamo questa informazione.
*   Dato un livello *l*, chiedere ad un vicino *v*, dato uno stub per contattarlo, di richiedere al Coordinator
    del suo g-nodo di livello *l* la prenotazione di un posto, come nuovo g-nodo di livello *l* - 1.
    Metodo `get_reservation`.  
    Questo metodo può rilanciare l'eccezione CoordinatorStubNotWorkingError se la comunicazione con il
    vicino non riesce.  
    Questo metodo può rilanciare l'eccezione CoordinatorMemoryNotReadyError se il vicino *v* non ha ancora
    completato la fase di boostrap (vedi modulo QSPN). Infatti il nodo *v* non ha ancora potuto instanziare
    il suo PeersManager, né il CoordinatorService.  
    Questo metodo può rilanciare l'eccezione CoordinatorInvalidLevelError, segnalata immediatamente dal
    vicino, se il livello richiesto non è coerente con la topologia della rete in cui si trova il vicino.  
    Questo metodo può rilanciare l'eccezione CoordinatorSaturatedGnodeError, segnalata dal vicino dopo
    aver comunicato con il servizio Coordinator, se al livello richiesto non è stato possibile riservare
    un posto.  
    Di solito questa operazione si fa subito dopo aver ottenuto dallo stesso vicino la lista del numero di
    posti che vede liberi. Se da questa lista il livello *l* risultava avere posti liberi, ma la richiesta
    della prenotazione rilancia l'eccezione CoordinatorSaturatedGnodeError, tale esito potrebbe essere dovuto
    alle precedenti prenotazioni non ancora confermate da un ETP. Quindi avrebbe poco senso chiedere di nuovo
    al vicino quanti posti vede liberi. Si assuma che al livello *l* i posti sono 0 e si continui a ritenere
    validi i valori per gli altri livelli, sia superiori che inferiori.  
    Se invece non sono rilanciate eccezioni, il metodo restituisce una istanza di ICoordinatorReservation. Con
    i dati contenuti in questa istanza (si veda sotto il dettaglio) il nodo potrà costituire un nuovo g-nodo.

Implementa il servizio Coordinator derivando la classe CoordinatorService dalla classe base PeerService.

Il modulo Coordinator si occupa di registrare con il PeersManager l'implementazione del servizio Coordinator
e di usare gli algoritmi forniti dal modulo PeerServices per il mantenimento del relativo database a chiavi
fisse. Cioè, esso crea una istanza della classe CoordinatorService.DatabaseDescriptor, che implementa
IFixedKeysDatabaseDescriptor. Con questa istanza come parametro, al bisogno, chiama i metodi
`fixed_keys_db_on_startup` e `fixed_keys_db_on_request` di PeersManager.

Deriva la classe CoordinatorClient dalla classe base PeerClient per avviare il contatto del Coordinator
di un suo g-nodo e richiederne i servizi.

## <a name="Classi_e_interfacce"></a>Classi e interfacce

La mappa delle posizioni libere/occupate ai vari livelli è un oggetto di cui il modulo conosce
l'interfaccia ICoordinatorMap. Tramite essa il modulo può:

*   Leggere un `int64` che rappresenta l'identificativo della rete (metodo `get_netid`).
*   Leggere il numero *l* dei livelli della topologia (metodo `get_levels`).
*   Leggere la gsize di ogni livello *i* da 0 a *l* - 1 (metodo `get_gsize`).  
    Precisiamo il significato di questo indice, restando coerenti con quanto stabilito nella
    trattazione del modulo QSPN sebbene i due moduli siano indipendenti.  
    Per ogni *i* da 0 a *l* - 1, *gsize(i)* è il numero massimo di g-nodi di livello *i* in un
    g-nodo di livello *i* + 1.
*   Leggere l'anzianità del mio g-nodo ad ogni livello *i* da 0 a *l* - 1 (metodo `get_eldership`).  
    L'anzianità di un g-nodo è un numero progressivo che viene assegnato al g-nodo. Nel confronto
    tra due g-nodi di pari livello *i* entrambi appartenenti allo stesso g-nodo di livello *i* + 1,
    un valore più alto significa che il g-nodo è arrivato dopo, cioè esso è più giovane.
*   Leggere il numero approssimativo di singoli nodi contenuti nel mio g-nodo ad ogni livello *i*
    da 0 a *l* - 1 (metodo `get_n_nodes`).  
*   Leggere la posizione del nodo, cioè la posizione del mio g-nodo ad ogni livello *i* da 0
    a *l* - 1 (metodo `get_my_pos`).
*   Leggere l'elenco delle posizioni libere in ogni livello *i* da 0 a *l* - 1 (metodo `get_free_pos`).  
    Cioè le posizioni nel livello *i* che sono libere nel nostro g-nodo di livello *i* + 1. Questa
    informazione è quella che si basa sulla mappa del nodo corrente, senza contattare il Coordinator
    attuale che ha la conoscenza autoritativa delle prenotazioni pendenti.

* * *

Il metodo `get_neighbor_map` restituisce una istanza di ICoordinatorNeighborMap. Tramite questa
interfaccia il modulo può:

*   Leggere il numero di livelli nella topologia (metodo `get_levels`).
*   Leggere la gsize per ogni livello *i*, con *i* da 0 a *levels* - 1 (metodo `get_gsize`).
*   Leggere il numero di posti liberi in ogni livello *i*, con *i* da 0 a *levels* - 1
    (metodo `get_free_pos_count`).

* * *

Una istanza di IEnterNetworkHandler viene passata al modulo Coordinator. Tale istanza viene
interrogata da parte del modulo quando si deve decidere sul fare ingresso in una nuova rete.

I metodi previsti dall'interfaccia IEnterNetworkHandler sono:

*   `int choose_target_level(int min_lvl, int max_lvl, int netid, List<int> gsizes, List<int> n_nodes, List<int> n_free_pos)`  
    Con questo metodo si da una valutazione sul livello in cui sembra opportuno cercare di
    fare ingresso. Questo a partire dalle informazioni sui g-nodi ottenuti da un singolo
    nodo con il quale esiste un arco.  
    Questa prima valutazione viene fatta senza sapere se esistono altri archi che collegano
    le due reti in altri punti della topologia.  
    Per descrivere il significato degli argomenti passati, si consideri che questa valutazione viene
    fatta dalla rete *G* per fare ingresso nella rete *J*; il proponente di questa soluzione è il
    singolo nodo *n* in *G* che ha un arco verso il singolo nodo *v* in *J*; il nodo che fa questa
    valutazione è il Coordinator di *G*.  
    Gli argomenti passati sono:
    *   `min_lvl` è il livello più basso a cui *n* è disposto a fare ingresso in *J*.  
        Come abbiamo detto sopra, per *n* potrebbe essere necessario avere la prenotazione di un livello superiore
        a 0 perché il nodo fa da gateway verso una rete privata in cui si vogliono adottare diversi meccanismi di
        assegnazione di indirizzi e routing. Il gateway necessita di un livello tale da riservare un certo numero
        di bit per gli indirizzi interni. La topologia da considerare per questo calcolo è quella di *J*; sebbene
        è implicitamente necessario che la topologia di *G* e quella di *J* coincidano almeno ai livelli inferiori
        a quello che sarà scelto per l'ingresso.
    *   `max_lvl` è il livello più basso di cui la rete *G* è composta da un solo g-nodo.  
        Se si raggiunge la prenotazione di una posizione a questo livello, l'intera rete *G* può entrare in *J*
        in blocco. Non è quindi necessario chiedere di più.
    *   `netid` è l'identificativo della rete *J*. L'implementatore di IEnterNetworkHandler potrebbe farne uso:
        ad esempio memorizza le reti incontrate (nel nodo stesso oppure nel Coordinator della rete) e sa se
        in precedenza aveva tentato una migration path (al livello *x*) fallita in quella stessa rete.
    *   `gsizes` descrive la topologia della rete *J*. Da questa lista si ricava anche `levels` di *J*.  
        Se la topologia di *G* e quella di *J* dovessero differire (assumendo che questa situazione sia supportata)
        questo limiterebbe il valore del livello che la presente valutazione può restituire; infatti le topologie
        devono essere identiche ai livelli inferiori di *l* se si vuole fare ingresso in *J* con un g-nodo di
        livello *l* che esisteva in *G*.  
        Ma sappiamo che chi chiama il metodo `choose_target_level` di IEnterNetworkHandler appartiene alla rete
        *G*; quindi questi (cioè il modulo Coordinator) si sarà accertato che le due topologie coincidono; se non
        dovessero coincidere ai livelli più alti il Coordinator può variare l'argomento `max_lvl` per fare in modo
        che le topologie coincidano ai livelli da 0 a `max_lvl`. Ovviamente, anche dopo questa variazione deve
        rimanere vero che `max_lvl` ≥ `min_lvl`.
    *   `n_nodes` dice quanti singoli nodi (approssimativamente) esistono in ognuno dei g-nodi di *J* di cui fa
        parte *v*. Per l'esattezza, il valore `n_nodes[i]` dice quanti nodi ci sono nel g-nodo di livello `i+1`.
        Il dato potrebbe essere usato nella valutazione, non so come.
    *   `n_free_pos` dice quanti posti liberi esistono in ognuno dei g-nodi di *J* di cui fa parte *v*. Per
        l'esattezza, il valore `n_free_pos[i]` dice quanti posti liberi ci sono nel g-nodo di livello `i+1`.  
        Essenzialmente, ci serve solo sapere se è maggiore di 0. Se la topologia lo permette, un numero maggiore
        è da preferire, soltanto perché riduce la probabilità dell'evento che un posto apparentemente libero
        sia stato in effetti prenotato. Ricordiamo infatti che il dato è fornito dal singolo nodo *v* senza
        contattare il suo Coordinator in *J* e che, comunque, passa del tempo da quando il dato viene fornito da *v*
        a quando il tentativo di ingresso viene autorizzato dal Coordinator di *G*.
*   `b`  
    Con questo metodo si da una valutazione sulla migliore fra alcune possibili soluzioni
    per l'igresso. **TODO**

**Nota** Supponiamo che si voglia fare ingresso in una rete *J* con topologia a 20 livelli. Supponiamo che il `max_lvl`,
cioè la dimensione del più grande g-nodo in *G*, sia 5.

**_caso 0_** Valutando l'ingresso del g-nodo di livello 5, supponiamo che
non ci sia alcun posto nel g-nodo di livello 6 del singolo nodo *v* con cui siamo entrati in contatto. La stessa cosa per
tutti i livelli fino al 19. Questo significa in altre parole che come minimo serve una migration-path di lunghezza maggiore di 0.  
Si inizia con il livello 5. Cioè si cerca la più breve migration-path che libera un posto nel g-nodo di livello 6 del
singolo nodo *v* facendo spostare *n* border g-nodi di livello 5 (da un g-nodo di livello 6 in un altro) fino al punto
in cui un g-nodo di livello 5 migra dentro un g-nodo di livello 6 o superiore con un posto libero.  
Se una tale migration-path al livello 5 non esiste, questo implica che non esiste nemmeno una migration-path al livello
6 o superiori. In questo caso occorre degradare cercando di fare entrare *G* in *J* con un g-nodo di livello 4 alla volta.  
Se invece esiste la migration-path al livello 5 che muove *n* g-nodi, possiamo accettare questa oppure cercare ai
livelli più alti. Forse esiste una migration-path al livello 6 che muove *k* g-nodi, con *k* minore di *n*.
Forse esiste una migration-path al livello 7 che muove *q* g-nodi, con *q* minore di *k*. Quale soluzione scegliere? **Open question.**

**_caso 1_** Valutando l'ingresso del g-nodo di livello 5, supponiamo che
non ci sia alcun posto nel g-nodo di livello 6 del singolo nodo *v* con cui siamo entrati in contatto. La stessa cosa per
i livelli 6..12. Invece valutando l'ingresso di un g-nodo di livello 13, esiste un posto libero nel g-nodo di livello 14
del singolo nodo *v*. Questo significa in altre parole che sappiamo da subito che la migration-path al livello 13 da
eseguire per far entrare *G* è di lunghezza 0.  
Cerchiamo comunque la più breve migration-path al livello 5? **Open question.**

* * *

Il metodo `get_reservation` restituisce una istanza di ICoordinatorReservation. Tramite questa
interfaccia il modulo può:

*   Leggere il numero di livelli nella topologia (metodo `get_levels`).
*   Leggere la gsize per ogni livello *i*, con *i* da 0 a *levels* - 1 (metodo `get_gsize`).
*   Leggere il livello, la posizione e l'anzianità del g-nodo appena riservato
    (metodi `get_lvl`, `get_pos` e `get_eldership`).
*   Leggere la posizione del g-nodo in cui siamo entrati per ogni livello *i*, con *i* da *lvl* + 1
    fino a *levels* - 1, inclusi (metodo `get_upper_pos`).
*   Leggere il valore di anzianità per ogni livello *i*, con *i* da *lvl* + 1 fino a *levels* - 1,
    inclusi (metodo `get_upper_eldership`).  
    Per i livelli inferiori a *lvl* il nodo, che ha appena costituito un nuovo g-nodo di livello
    *lvl*, userà come anzianità il valore 0 e come posizione un valore random congruente con la topologia.

