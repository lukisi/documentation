# Modulo PeerServices - Analisi Funzionale

1.  [Idea generale](#Idea_generale)
    1.  [DHT: Distributed Hash Table](#DHT)
        1.  [Memoria esaurita e memoria non esaustiva](#Memoria_esaurita)
    1.  [HDHT: Hierarchical DHT](#HDHT)
1.  [Ruolo del modulo PeerServices](#Ruolo_del_modulo)
    1.  [Comunicazioni peer-to-peer](#Comunicazioni_peer_to_peer)
1.  [Requisiti](#Requisiti)
1.  [Deliverables](#Deliverables)
1.  [Classi e interfacce](#Classi_e_interfacce)

## <a name="Idea_generale"></a>Idea generale

Il funzionamento di servizi distribuiti in un modello non centralizzato di rete, che possiamo chiamare
servizi peer-to-peer, si basa sul fatto di poter individuare un nodo fra quelli presenti nella rete in un
dato momento al quale rivolgere delle richieste. I nodi presenti nella rete cambiano da un momento all'altro,
così come cambia la loro identificazione. Si potrebbe anche aggiungere che non tutti i nodi sono disposti
a partecipare attivamente al servizio rispondendo alle richieste altrui e anche questo può cambiare nel tempo.

Per ogni servizio occorre definire una funzione che associ ad ogni chiave *k* (nel dominio di chiavi
definito dal servizio) un nodo esistente nella rete. A tale nodo andranno indirizzate le richieste
concernenti la chiave *k*. Siccome tale nodo dovrà rispondere alle richieste, se il servizio prevede la
possibilità che un nodo decida di non partecipare attivamente (chiamiamo questo tipo di servizio un
*servizio opzionale*) va aggiunto il requisito che la funzione associ ad ogni chiave *k* un nodo
*partecipante* al servizio.

Sia *S* lo spazio di indirizzi validi per i nodi della rete. Per indirizzo valido intendiamo un indirizzo
Netsukuku *reale* (senza componenti *virtuali* come definiti nel documento di analisi del modulo
[Qspn](../ModuloQspn/AnalisiFunzionale.md#Nodi_virtuali)).

Sia *V<sub>t</sub>* il set di nodi nella rete al tempo *t*.

Sia *𝛼<sub>t</sub>* : *S* → *V<sub>t</sub>* la funzione che al tempo *t*  associa ad un indirizzo
il nodo che lo detiene. È una funzione non
completamente  definita in *S* poiché un indirizzo potrebbe non essere stato assegnato ad  alcun nodo.

Definiamo una funzione *H<sub>t</sub>* che al tempo *t* assegni ad ogni indirizzo in *S* un indirizzo nel dominio di
*𝛼<sub>t</sub>*. Cioè dato un indirizzo valido che potrebbe non essere stato assegnato tale funzione ritorna
un indirizzo assegnato.

**Definizione:** *H<sub>t</sub>* : *S* → *dom(𝛼<sub>t</sub>)*

### <a name="DHT"></a>DHT: Distributed Hash Table

Sia *p* un servizio distribuito che vuole implementare un database, sia *K* lo spazio delle chiavi definito
da *p* per questo database. Il servizio *p* definisce una funzione di hash *h<sub>p</sub>* che mappa lo spazio delle chiavi sullo
spazio degli indirizzi.

**Definizione:** *h<sub>p</sub>* : *K* → *S*

Quando un nodo, al tempo *t*, vuole scrivere la coppia chiave-valore *(k, v)* nel database distribuito
il nodo calcola:

**Definizione:** *hash_node(k)* = *𝛼<sub>t</sub> ( H<sub>t</sub> ( h<sub>p</sub> ( k ) ) )*

Contatta quindi il nodo *hash_node(k)* e chiede di memorizzare la coppia *(k, v)*.

Analogamente  il nodo che vuole reperire il dato associato alla chiave *k*, calcola  *hash_node(k)*, contatta
il nodo e chiede di leggere il dato associato a *k*.

Questo  procedimento realizza un database distribuito, perché ogni nodo  mantiene solo una porzione delle
associazioni chiave-valore.

Fondamentale è la funzione *H<sub>t</sub>*. Questa funzione ha un comportamento unico, indipendente
dalle caratteristiche dello specifico servizio *p*, può quindi essere definita e implementata una volta sola.
Nelle sue operazioni, comunque, deve conoscere l'identificativo univoco del servizio *p* in esame
in quanto, se il servizio è opzionale, il risultato dipende dalla conoscenza di quali indirizzi sono detenuti
da nodi che partecipano al servizio. Si tratta in questi casi di un sottinsieme del dominio di *𝛼<sub>t</sub>*.

La conoscenza degli indirizzi detenuti dai nodi presenti nella rete (cioè del dominio di *𝛼<sub>t</sub>*) è realizzata attraverso il protocollo di
routing Qspn. Occorre invece definire un ulteriore meccanismo per giungere alla conoscenza di quali indirizzi
sono detenuti da nodi che partecipano ad ognuno dei servizi opzionali.

#### <a name="Memoria_esaurita"></a>Memoria esaurita e memoria non esaustiva

Inoltre, nell'implementazione della funzione *H<sub>t</sub>*, occorre dare la possibilità al singolo nodo, una volta
che è stato individuato e contattato e gli è stata fatta una richiesta, di rifiutarsi di elaborarla e così di
obbligare la stessa funzione  *H<sub>t</sub>* a trovare un altro nodo. Ad esempio, questo si rende necessario quando
il singolo nodo contattato, chiamiamolo *n*, non ha più memoria libera.

Se la richiesta era di inserimento il nodo *n*, se non ha più memoria libera, rifiuta di memorizzare e quindi la
funzione *H<sub>t</sub>* trova il prossimo nodo. Allo stesso tempo il nodo *n* si ricorderà (fin quando permarrà in
questo stato) di non avere tutte le informazioni necessarie per rispondere "NOT_FOUND" ad una richiesta di lettura
o aggiornamento, cioè di non essere *esaustivo*.

Se la richiesta era di lettura o aggiornamento il nodo *n*, se non trova il record per la chiave *k* nella sua
memoria **e** ricorda di non essere esaustivo, rifiuta di rispondere e quindi la funzione *H<sub>t</sub>* trova
il prossimo nodo.

Questo concetto di essere non *esaustivo* viene usato da un nodo *n* anche quando è da poco entrato nella rete ed
è ancora in attesa di reperire i record di sua pertinenza.

### <a name="HDHT"></a>HDHT: Hierarchical DHT

In una struttura gerarchica come Netsukuku ([dettagli](../ModuloQspn/AnalisiFunzionale.md#StrutturaGerarchica)) un nodo
non ha la conoscenza  di tutti i nodi esistenti nella rete, quindi non può da solo computare la funzione
*H<sub>t</sub>* in quanto non conosce per intero *dom(𝛼<sub>t</sub>)*.

Infatti ogni nodo *n* con indirizzo n<sub>0</sub>·n<sub>1</sub>·...·n<sub>l-1</sub> ha solo conoscenza di:

*   tutti i nodi appartenenti a n<sub>1</sub>,
*   tutti i g-nodi di livello 1 appartenenti a n<sub>2</sub>,
*   ...
*   tutti i g-nodi di livello l-2 appartenenti a n<sub>l-1</sub>,
*   tutti i g-nodi di livello l-1.

Questa conoscenza la possiamo chiamare *dom<sub>n</sub>(𝛼<sub>t</sub>)*, cioè il dominio della funzione
*𝛼<sub>t</sub>* secondo le conoscenze di *n*.

L'implementazione della funzione *H<sub>t</sub>* deve dunque avvenire in modo distribuito. Vedremo come nel
documento dei [dettagli](DettagliTecnici.md).

## <a name="Ruolo_del_modulo"></a>Ruolo del modulo PeerServices

Nel modulo PeerServices il nodo registra i servizi peer-to-peer ai quali intende partecipare, specificando quali
di questi sono opzionali.

Ogni servizio peer-to-peer ha un intero positivo come suo identificativo, *p_id*.

Il modulo PeerServices si occupa di divulgare in tutta la rete (tramite trasmissione di vicino in vicino) la
conoscenza della partecipazione del nodo ad ogni servizio opzionale. Questa operazione non è necessaria per i
servizi non opzionali, proprio perché ogni nodo esistente nella rete partecipa attivamente ad essi.

Allo stesso tempo il modulo PeerServices, nei singoli nodi in cui riceve questa informazione e la propaga, si
occupa di mantenere la conoscenza, sempre in forma gerarchica, di tutti i partecipanti a tutti i servizi opzionali
esistenti nella rete; questo pur senza necessitare di conoscere a priori quali servizi opzionali esistano. Chiamiamo
l'insieme di tali conoscenze *mappe dei servizi opzionali*.

**Nota**: Dato un servizio opzionale *p_id*, la mappa gerarchica relativa a questo servizio ha significato ai
livelli da -1 a l-1.  
Al livello -1 esiste un solo elemento che è il nodo stesso. Esso viene identificato con
`HCoord (0, pos[0])`.  
Al livello 0 esistono tutti i nodi di livello 0 che appartengono al mio stesso g-nodo di
livello 1. Essi sono identificati con `HCoord (0, x)` dove `x ≠ pos[0]`.  
Al livello `i`, con `0 ≤ i < l`, esistono tutti i nodi di livello `i` che appartengono al mio stesso g-nodo di
livello `i + 1`. Essi sono identificati con `HCoord (i, x)` dove `x ≠ pos[i]`.  
In realtà la mappa di un servizio può essere implementata come un insieme di HCoord, e in tali oggetti
il livello va da 0 a l-1. Ma quando facciamo ragionamenti legati al livello del g-nodo che fa ingresso e/o
al livello del g-nodo ospitante occorre tenere presente questa differenza: il contenuto della mappa
che si riferisce a `HCoord (0, pos[0])` è per definizione di livello -1.

Quando un nodo *n* entra in una rete, oppure quando migra da un g-nodo ad un diverso g-nodo, le sue mappe gerarchiche dei servizi
opzionali dovranno essere aggiornate. Se un g-nodo *w* in blocco entra in una nuova rete o migra, tutti i nodi al
suo interno dovranno reperire e aggiornare le loro mappe.

Generalizzando, quando un nodo *n* come membro di un g-nodo *w* di livello *i* entra in un g-nodo *g* di livello
*k* (con `k > i`), se fra i suoi diretti vicini c'è un nodo (non appartenente a *w*) che era in *g* prima di lui
lo contatta e richiede le mappe dei servizi opzionali ai livelli da `k - 1` in su. Altrimenti si aspetta di ricevere tali informazioni su
iniziativa di un altro nodo che era in *w*.

In ogni caso, il nodo *n* mantiene valide le informazioni che aveva fino al livello *i* - 1. Ricordiamo che questo tipo
di eventi (ingresso o migrazione) si realizzano sempre sotto forma di creazione di una nuova identità nel sistema.
L'istanza del modulo PeerServices associata alla nuova identità recupera alcune informazioni dall'istanza del
modulo che era già associata alla vecchia identità.

* * *

In ogni momento un nodo può fare al suo modulo PeerServices una richiesta relativa ad un servizio con un dato *p_id*.

In realtà in ogni sistema, solo la *identità principale* è abilitata a fare richieste al suo modulo PeerServices. E se
l'identità ha qualche componente *virtuale* nel suo indirizzo, può fare richieste solo circoscritte al massimo
g-nodo dentro il quale tutte le sue componenti sono *reali*.

Il modulo PeerServices saprà a chi indirizzare la richiesta. Infatti, se il servizio è non-opzionale per definizione
esso è fra quelli registrati nel modulo, quindi il modulo lo conosce, sa che è non-opzionale e non ha bisogno di mappe di
partecipazione per ricercare un suo hash-node. Se il servizio pur essendo opzionale è stato registrato nel modulo, anche in questo
caso il modulo lo conosce e sa che deve consultare le mappe di partecipazione (in questo caso almeno il nodo stesso
è nelle mappe). Se il servizio opzionale non è stato registrato nel modulo dal nodo, cioè il nodo non vi partecipa attivamente possono
esservi due casi:

1.  Il modulo è venuto a conoscenza di alcuni nodi nella rete che partecipano. Allora ha le mappe per avviare la
    ricerca dell'hash-node.
1.  Il modulo non ha mai ricevuto informazioni di partecipazioni al servizio con questo identificativo. Allora deduce
    che nessun nodo nella rete è partecipante al servizio.

* * *

Un nodo che intende partecipare attivamente ad un servizio, quando fa ingresso (da solo o in blocco insieme ad un g-nodo)
in una rete esistente oppure quando migra da un g-nodo ad un altro, può trovarsi
in bisogno di reperire i record che sono di sua pertinenza nel mantenimento del database distribuito, come destinatario
principale o come replica. Nell'espletamento di questa mansione il nodo si può avvalere di alcune *funzioni helper*
che il modulo PeerServices fornisce.

* * *

Nella fattispecie di un nodo *n* che partecipa ad un servizio *p* e che viene chiamato ad operare quale
server di una certa richiesta *r*, il modulo PeerServices fornisce al nodo *n* gli strumenti per richiedere
la replica di *r* (ad esempio la memorizzazione di un dato) su un numero *q* di nodi partecipanti al servizio che
sarebbero stati i più prossimi destinatari della richiesta *r* in caso di assenza del nodo *n*.

### <a name="Comunicazioni_peer_to_peer"></a>Comunicazioni peer-to-peer

Le comunicazioni peer-to-peer rese possibili da questo modulo, ad alto livello si possono intendere in questo modo:
un *nodo* della rete vuole un servizio, quindi si comporta come *client* del servizio *p*. Usando una determinata
chiave *k* identifica implicitamente un altro *nodo* della rete ad agire come *server*. Il *client* trasmette un
messaggio al *server* e questi a sua volta gli trasmette una risposta.

Scendendo ad un livello più basso senza comunque entrare troppo nei dettagli delle operazioni, indichiamo le fasi
che comporta il primo messaggio (dal *client* al *server*) e poi il secondo (dal *server* al *client*).

Il nodo client incapsula il messaggio *client-server* in un *pacchetto-p2p*. Tale pacchetto contiene anche altre
informazioni. Tra queste c'è, in una certa forma, l'indirizzo del nodo server. Ma non è un indirizzo completo.

In pratica, nel pacchetto c'è la parte bassa dell'indirizzo del nodo *teorico* (*h<sub>p</sub>(k)*), e l'indirizzo in forma
gerarchica (*HCoord*) rispetto al nodo corrente *n* (inizialmente rispetto al nodo client) del g-nodo visibile
nella sua mappa che contiene il nodo *concreto* (cioè *H<sub>t</sub>(h<sub>p</sub>(k))* secondo la conoscenza
*dom<sub>n</sub>(𝛼<sub>t</sub>)* del nodo corrente *n*).

Il nodo client (con una logica ovviamente mirata ad avvicinare il pacchetto alla sua destinazione finale) sceglie
un suo vicino a cui comunica il *pacchetto-p2p*. Questo vicino diventa quindi un nodo
intermedio, chiamiamolo *forwarder*.

Ogni nodo forwarder è chiamato a instradare il *pacchetto-p2p* trasmettendolo ad un altro suo vicino verso la
destinazione finale, cioè il server. Un nodo forwarder che riceve un *pacchetto-p2p* si trova in una di 3 possibili
situazioni:

1.  Il nodo forwarder non fa parte del g-nodo indicato come destinazione nel pacchetto. Quindi deve solo trasmettere
    il *pacchetto-p2p* al suo diretto vicino più adeguato.
1.  Il nodo forwarder fa parte del g-nodo indicato come destinazione nel pacchetto. Quindi deve intervenire nel
    calcolo distribuito di *H<sub>t</sub>(h<sub>p</sub>(k))*: individua un nuovo g-nodo di livello più basso
    in cui si trova il nodo server e modifica di conseguenza le informazioni contenute
    nel pacchetto. Poi deve di nuovo trasmettere il *pacchetto-p2p* al suo diretto vicino più adeguato.
1.  Il nodo forwarder fa parte del g-nodo indicato come destinazione nel pacchetto, ed è in effetti esso stesso
    il nodo server. Ovviamente nel nodo server il servizio *p* deve essere stato registrato nel modulo PeerServices,
    cioè il nodo server deve essere partecipante al servizio *p*.

Per quanto riguarda il messaggio *server-client*, invece, il server ha conoscenza di tutto l'indirizzo del nodo
client e quindi questa comunicazione può avvenire direttamente con una connessione TCP dal server al client.

Fatta questa premessa, scendiamo ancora di livello e vediamo quali operazioni ogni nodo coinvolto deve fare in
termini di calcolo di *H<sub>t</sub>* e di *h<sub>p</sub>*.

Il nodo client richiede al suo modulo PeerServices di avviare una comunicazione verso
un hash-node per la chiave *k* di un servizio *p*; il modulo PeerServices del nodo
client avvia il calcolo distribuito di *H<sub>t</sub>*. Per fare questo il modulo non ha
bisogno di conoscere l'implementazione della funzione *h<sub>p</sub>* ma soltato il risultato di *h<sub>p</sub>(k)*;
d'altra parte questa richiesta arriva dal nodo stesso quindi il nodo conosce l'implementazione della funzione
*h<sub>p</sub>*. Inoltre per fare questa operazione non è necessario che il nodo partecipi al servizio *p*.

Lo stesso modulo, nei nodi forwarder, si occupa di instradare il messaggio e di proseguire il
calcolo distribuito di  *H<sub>t</sub>*. Per fare questo il modulo non ha bisogno di conoscere la logica interna del
servizio *p*, ma deve solo sapere l'identificativo del servizio *p_id* e il valore di *h<sub>p</sub>(k)*; questi dati
sono contenuti nel *pacchetto-p2p*. Quindi per fare questa operazione il nodo non ha bisogno né di partecipare
al servizio *p* e nemmeno di conoscere nulla sull'implementazione del servizio *p*.  
Un caso particolare, però, si ha quando il nodo forwarder deve dettagliare a livelli più bassi l'indirizzo del
server nel *pacchetto-p2p* e allo stesso tempo il servizio *p* è opzionale.  
Infatti dobbiamo tenere presente che un nodo che fa ingresso in una nuova rete (o che migra in un diverso g-nodo)
deve aspettare di ricevere le *mappe dei servizi opzionali* da chi le conosceva prima di lui. Fino ad allora tale
nodo non può ricalcolare la funzione *H<sub>t</sub>* per inoltrare richieste di un servizio opzionale, a meno
che non si tratti di richieste espressamente circoscritte fin dall'inizio ad un suo g-nodo di livello inferiore
o uguale al g-nodo che ha fatto questo ingresso (o migrazione) in blocco con lui.  
L'informazione del livello di g-nodo in cui la richiesta è stata inizialmente circoscritta è
contenuta nel *pacchetto-p2p*.

Lo stesso modulo, nel nodo server, si occupa di ricevere la richiesta del nodo client, di processarla e di
trasmettere al client il relativo messaggio di risposta. Tutte queste comunicazioni il nodo server le può
fare per mezzo di una connessione TCP diretta con il nodo client. Per questo è necessario (e accettabile come
requisito) che i nodi client e server abbiano nel loro indirizzo componenti *reali* a tutti i livelli
più bassi del livello di g-nodo in cui la richiesta è circoscritta.

## <a name="Requisiti"></a>Requisiti

*   Mappa corrente dei percorsi noti.
*   Factory per aprire una connessione TCP con un percorso interno ad un proprio  g-nodo verso un nodo di cui si
    conosce l'identificativo interno.
*   Factory per ottenere uno stub per inviare un messaggio in broadcast (con callback per gli archi in cui il
    messaggio fallisce) o uno stub per inviare un messaggio su un arco in modo reliable.

Inoltre, se non si tratta di un sistema appena avviato (cioè della prima identità del sistema, la quale nasce
come unico nodo in una nuova rete) ma di una identità che viene creata per sostituire una precedente
identità, cioè o in caso di ingresso in un'altra rete o in caso di migrazione in un altro g-nodo, il modulo
deve conoscere:

*   L'istanza del modulo che era associata alla precedente identità.
*   Il livello del g-nodo ospite.
*   Il livello del g-nodo ospitante.

## <a name="Deliverables"></a>Deliverables

*   Viene definita una classe base astratta (PeerService) che deve essere derivata per implementare ogni specifico
    servizio peer-to-peer, sia quelli opzionali che quelli non opzionali.
*   Viene definita una classe base astratta (PeerClient) che deve essere derivata per implementare una classe da
    usare come client per un servizio peer-to-peer.
*   Fornisce un metodo (`register`) per registrare le istanze di classi che rappresentano un servizio a cui il
    nodo partecipa.
*   Fornisce dei metodi helper (inclusi nel modulo per evitare duplicazione di codice) che potranno essere usati
    dai vari servizi registrati per svolgere alcune mansioni comuni:

    *   Recupero record di pertinenza garantendo la [coerenza del dato](DettagliTecnici.md#Mantenimento_database_distribuito).
    *   Gestione memoria esaurita e non esaustiva.

    Tali metodi dipendono da quali sono le caratteristiche dello specifico servizio e come possono essere sfruttate
    per il mantenimento del database distribuito e della coerenza dei dati. Al momento sono state individuate queste
    classi di servizi e per esse sono stati delineati degli algoritmi:
    *   Alcuni servizi danno una scadenza (Time To Live) ai record che mantengono. Inoltre hanno un numero alto e
        indefinibile di possibili chiavi. Per questi servizi, gli algoritmi delineati sono `ttl_db_on_startup` e
        `ttl_db_on_request`.
    *   Alcuni servizi non danno alcuna scadenza ai record che mantengono. Però hanno un numero esiguo di possibili
        chiavi. Inoltre non prevedono l'eccezione `NOT_FOUND` per una chiave. Ad esempio il servizio Coordinator.
        Per questi servizi, gli algoritmi delineati sono `fixed_keys_db_on_startup` e `fixed_keys_db_on_request`.

*   Fornisce dei metodi (`begin_replica` e `next_replica`) per i vari servizi registrati perché possano replicare
    i record salvati su un numero di nodi replica.  
    Questi vanno chiamati dalle classi che implementano un servizio, appunto quando hanno salvato (o modificato) un record.
*   Fornisce un metodo (`contact_peer`) per effettuare una richiesta ad un servizio peer-to-peer. In realtà il
    metodo non è esposto dal modulo: esso è chiamato internamente dalla classe base PeerClient. Le informazioni per
    questo metodo che provengono dall'esterno del modulo sono:

    *   L'identificativo del servizio. Informazione implicita nella classe base PeerClient. Gli viene passata dalla
        classe derivata nel costruttore.
    *   La chiave *k*.
    *   L'istanza di IPeersRequest che rappresenta la richiesta da fare.
    *   Il tempo limite per l'esecuzione della richiesta una volta consegnata all'hash-node.

    L'utilizzatore di un servizio chiama (con i dovuti argomenti) uno specifico metodo di una specifica classe
    derivata da PeerClient. Tale classe prepara le suddette informazioni e le passa al modulo chiamando il metodo
    protetto di PeerClient `call`. Con queste informazioni il modulo è in grado di effettuare la richiesta senza
    dover conoscere altro sul servizio: infatti per fare una richiesta ad un servizio il nodo non ha bisogno di
    partecipare attivamente al servizio, quindi il nodo potrebbe non aver passato il servizio in questione al
    metodo register; basta che il nodo sappia come interfacciarsi con il servizio.  
    Il metodo restituisce una istanza di IPeersResponse che può rappresentare il risultato della richiesta o una
    eccezione prevista dal servizio stesso.  
    Oppure rilancia una eccezione PeersNoParticipantsInNetworkError se al momento nessun nodo nella rete partecipa
    attivamente al servizio.  
    Oppure rilancia una eccezione PeersDatabaseError se almeno un nodo ha risposto rifiutandosi di elaborare per
    memoria esaurita o non esaustiva e in seguito nessun altro nodo ha risposto elaborando la richiesta. Questa
    eccezione va interpretata dallo specifico metodo della classe derivata da PeerClient in uno di questi modi:

    *   Se la richiesta era di inserire un record nuovo, allora è un errore di tipo "memoria esaurita". Anche se
        qualche nodo aveva rifiutato per memoria non esaustiva, questa eccezione può sempre essere gestita dal
        client come memoria esaurita. Si consideri che la situazione di memoria esaurita nel database distribuito
        è sempre "temporanea": il client che riceve questa risposta non può far nulla per porre rimedio, se non
        riprovare in seguito. Diversi eventi, infatti, possono aver modificato la situazione.
    *   Se la richiesta era di leggere o modificare o rimuovere un record esistente, allora è un errore di tipo
        "record non trovato".

## <a name="Classi_e_interfacce"></a>Classi e interfacce

La classe HCoord è una classe [comune](../Librerie/Common.md) nota a questo modulo. È una classe serializzabile,
cioè le cui istanze sono adatte al passaggio di dati a metodi remoti (vedi framework [ZCD](../Librerie/ZCD.md)).
Una sua istanza contiene le coordinate gerarchiche di un g-nodo nella mappa del  nodo: livello e identificativo
nel livello.

* * *

La mappa delle rotte è un oggetto di cui il modulo conosce l'interfaccia IPeersMapPaths. Tramite essa il modulo può:

*   Leggere il numero di livelli della topologia (metodo `i_peers_get_levels`);
*   Leggere la gsize di ogni livello (metodo `i_peers_get_gsize`);
*   Leggere l'identificativo del proprio nodo a ogni livello (metodo `i_peers_get_my_pos`);
*   Leggere il numero di nodi stimato all'interno del proprio g-nodo a ogni livello
    (metodo `i_peers_get_nodes_in_my_group`);
*   Determinare se un certo g-nodo esiste nella rete – cioè se appartiene a *dom<sub>n</sub>(𝛼<sub>t</sub>)*
    (metodo `i_peers_exists`);
*   Ottenere uno stub per inviare un messaggio al miglior gateway verso un certo  g-nodo, segnalando
    opzionalmente se si vuole escludere un certo gateway  perché era il passo precedente nell'instradamento
    del messaggio e / o se  si vuole escludere e rimuovere un certo gateway che nel tentativo  precedente ha
    fallito la comunicazione (metodo `i_peers_gateway`). Questo stub è tale che il client invia il messaggio
    con protocollo reliable (TCP) senza ricevere una risposta e senza attendere la sua processazione, ma solo
    la conferma della ricezione.
*   Dato un livello *k* ottenere uno stub per inviare un messaggio ad un mio vicino (se esiste) che abbia come
    massimo distinto g-nodo nei miei confronti un g-nodo di livello *k* (metodo `i_peers_neighbor_at_level`).  
    Quando viene costruita una istanza della classe PeersManager (la classe principale del modulo PeerServices)
    essa è legata ad una identità del sistema. Se questa identità nasce per fare ingresso insieme al suo
    g-nodo di livello *i* dentro un g-nodo di livello `k + 1`, allora nel costruttore di PeersManager si
    userà questo metodo per individuare un vicino che faceva già parte di quello stesso g-nodo in cui è entrato.
    Userà questo stub per richiedere a tale vicino la mappa corrente dei g-nodi che partecipano a servizi opzionali.
    Questo stub, quindi, è tale che il client invia il messaggio con protocollo reliable (TCP) e attende una risposta.

* * *

In  alcuni casi sarà necessario realizzare una comunicazione TCP verso un  nodo originante di un messaggio.
In tali casi si vuole raggiungere il  nodo attraverso un percorso interno al proprio g-nodo di un dato  livello.

In  tutti i casi in cui questo è necessario il dialogo tra i due nodi è  molto semplice e prevede sempre uno
scambio di richiesta e risposta ad  iniziativa del nodo che inizia la connessione. Sarà quindi sufficiente
che si ottenga uno stub che realizza la chiamata di un metodo remoto tramite  questa connessione TCP.

L'oggetto fornito al modulo a questo scopo implementa l'interfaccia IPeersBackStubFactory, la quale permette di:

*   ottenere uno stub per inviare un messaggio ad un dato nodo mediante connessione TCP interna
    (metodo `i_peers_get_tcp_inside`).

* * *

In  alcuni casi sarà necessario inviare una comunicazione ai vicini che appartengono alla mia stessa rete.
L'oggetto fornito al modulo a questo scopo implementa l'interfaccia IPeersNeighborsFactory, la quale permette di:

*   ottenere uno stub per inviare un messaggio a tutti i vicini appartenenti alla nostra rete
    (metodo `i_peers_get_broadcast`) specificando una istanza di IPeersMissingArcHandler per gestire gli archi
    su cui non si è ricevuto un acknowledgement.
*   data una istanza di IPeersArc ottenere uno stub per inviare un messaggio reliable su quell'arco
    (metodo `i_peers_get_tcp`).

Quando il modulo usa il metodo `i_peers_get_broadcast` sull'oggetto IPeersNeighborsFactory `nf` fornito dal suo
utilizzatore, gli passa un oggetto IPeersMissingArcHandler `ah`. Lo stub che l'utilizzatore fornisce ora al
modulo tiene conto di questo `ah`. Quando lo stub trasmette un messaggio si aspetta di ricevere, entro un tempo
limite, un acknowledgement tramite ognuno degli archi noti al nodo (non sono noti al modulo, ma al suo utilizzatore):
se per qualcuno degli archi questo non avviene, allora lo stub si occuperà di richiamare su `ah` il metodo
`i_peers_missing` che riceve un'istanza di IPeersArc. Questa istanza è di un oggetto del tutto oscuro al modulo,
che lo può solo passare al metodo `i_peers_get_tcp` dell'oggetto `nf`.

* * *

L'interfaccia IPeersRequest non specifica nulla. Va implementata da un oggetto serializzabile. Tale oggetto deve
contenere le informazioni che possano identificare una chiamata ad un metodo remoto in un certo servizio.

Il "client" che prepara questo oggetto è specifico di un certo servizio *p*. Esso può assumere che il "server"
che lo riceverà è il server specifico dello stesso servizio *p*.

Segue un esempio, per nulla vincolante, di cosa questo oggetto potrebbe contenere.

*   Il nome del metodo. Si tratta di una stringa.
*   Gli argomenti del metodo. Si tratta di un array di stringhe che rappresentano in formato JSON gli argomenti
    del metodo.

* * *

L'interfaccia IPeersResponse non specifica nulla. Va implementata da un oggetto serializzabile. Tale oggetto
deve contenere le informazioni che possano rappresentare una risposta fornita da un metodo remoto in un
certo servizio.

Il "server" che prepara questo oggetto è specifico di un certo servizio *p* ed è stato invocato su un
particolare metodo *m* esposto da questo servizio. Esso può assumere che il "client" che lo riceverà è il client
specifico dello stesso servizio *p* e sa che si tratta di una risposta a quel particolare metodo *m*.

Segue un esempio, per nulla vincolante, di cosa questo oggetto potrebbe contenere.

*   L'eccezione lanciata. Si tratta di 3 stringhe nullable: `error_domain`, `error_code`, `error_message`.
*   Il valore di ritorno. Si tratta di una stringa. Se le stringhe `error_*` non sono valorizzate, allora questa
    stringa rappresenta in formato JSON il risultato della chiamata del metodo.

* * *

Tutte le classi che implementano un servizio derivano da PeerService. Questa è una classe astratta definita
dal modulo.

La classe base ha un membro `p_id` che identifica un servizio in tutta la rete. Esso è valorizzato nel costruttore
ed è in seguito di sola lettura. L'istanza del servizio è passata come istanza di PeerService al modulo
(PeersManager) nella chiamata iniziale al metodo di  registrazione e questo la memorizza associandola al suo
identificativo.

La classe base ha un membro booleano `p_is_optional` che dice se il servizio è da considerarsi opzionale. Esso è
valorizzato nel costruttore ed è in seguito di sola lettura.

La classe base ha un metodo astratto `exec` che viene richiamato sull'hash-node che è chiamato a servire una
richiesta. Esso riceve una istanza di IPeersRequest, ma è ancora in grado di rifiutarsi di
elaborare la richiesta, ad esempio per memoria esaurita o non esaustiva, rilanciando una
eccezione PeersRefuseExecutionError.

Inoltre, è in grado di dare istruzione al client di riavviare da capo il calcolo distribuito di *H<sub>t</sub>*,
rilanciando una eccezione PeersRedoFromStartError. Lo fa se ritiene plausibile di non essere più il miglior
candidato rispetto a quando la richiesta è stata avviata, ad esempio se ha fatto attendere il client per una
richiesta di scrittura mentre reperiva il record dal precedente detentore.

Altrimenti, elabora la richiesta e restituisce una istanza di IPeersResponse.

Nell'elaborazione della richiesta la classe del servizio ha a disposizione (come ulteriore argomento del
metodo `exec`) la tupla q<sub>0</sub>·q<sub>1</sub>·...·q<sub>j-1</sub> parte dell'indirizzo del nodo richiedente
*q* che ha in comune con il nodo corrente il g-nodo di livello *j*. Si tratta di una lista di interi (`List<int>`),
che può anche essere vuota: significa che il richiedente e il servente coincidono.

* * *

La classe PeerClient deve essere derivata per implementare il client di un servizio, sia esso opzionale o non opzionale.

La classe che deriva PeerClient deve fornire alla classe base (nel suo costruttore)
la conoscenza della topologia della rete, cioè il numero  di livelli e per ognuno la gsize. Inoltre
l'identificativo del servizio e un riferimento all'istanza di PeersManager.  
Tramite queste conoscenze la classe base fornisce l'implementazione di due metodi importanti:

*   `call` - chiamando il metodo *internal* `contact_peer`, instrada una richiesta e riceve la risposta.
*   `am_i_servant_for(k)` - dice se lo stesso nodo corrente è il servente per una data chiave. Questo
    sulla base del solo indirizzo, a prescindere dall'eventualità di nodi che non intendono rispondere
    ad esempio perché non esaustivi. Può risultare utile.

La classe derivata, per ogni tipo di richiesta che è prevista dal servizio, ha il compito di produrre l'oggetto
IPeersRequest che rappresenta la richiesta, di specificare quale sia il tempo massimo di attesa per l'esecuzione,
di chiamare il metodo `call` con i relativi argomenti per instradare la richiesta e ricevere la risposta,
di interpretare l'istanza di IPeersResponse ricevuta come risposta.

Nella classe derivata va definito il calcolo di *h<sub>p</sub>*. La funzione *h<sub>p</sub>* deve associare ad una
chiave *k* un indirizzo in *S*, cioè una tupla *x̄* = *x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>l-1</sub>* le cui componenti
siano compatibili con la topologia della rete. La classe base non sa come ottenere da una chiave *k* la tupla *x̄*,
questo procedimento spetta alla classe derivata. Tuttavia molte  operazioni saranno uguali nella maggior parte dei
servizi. Quindi la  classe base cerca di fornire i servizi comuni senza tuttavia essere di  impedimento alla classe
derivata se vuole usare altre modalità di  calcolo. Per fare questo la classe base fornisce:

*   un metodo virtuale `perfect_tuple` che riceve a parametro la chiave `Object k` e restituisce la tupla *x̄*.
*   un metodo astratto `hash_from_key` che riceve a parametro la chiave `Object k` e un intero `top` e restituisce
    un intero tra 0 e `top`.

Quando le operazioni del modulo richiedono il calcolo di *h<sub>p</sub>(k)* su un certo servizio *p*, il metodo
`perfect_tuple` viene  richiamato sull'istanza di PeerClient, quindi tale metodo deve essere pubblico.

Se tale metodo non viene ridefinito dalla classe derivata, il suo comportamento è il seguente. L'istanza conosce
le dimensioni dei g-nodi  ad ogni livello (gsizes) quindi calcola la dimensione dello spazio degli indirizzi validi.
Poi richiama il metodo `hash_from_key` passando oltre  alla chiave *k* il valore massimo dell'hash (la dimensione dello
spazio di indirizzi meno uno). In questo metodo la classe derivata deve occuparsi di associare alla chiave un valore
di hash (di norma uniformemente distribuito) compreso tra 0 e il valore massimo (inclusi). Questo metodo è demandato
alla classe derivata e quindi è definito astratto. Inoltre deve essere usato solo con la modalità sopra descritta,
cioè può essere chiamato solo dalla stessa istanza della classe, quindi può essere definito protetto.

Poi, nel metodo `perfect_tuple`, l'istanza usa il valore di hash per produrre una tupla *x̄* sulla base della sua
conoscenza di gsizes.

Se  invece la classe derivata ridefinisce il metodo `perfect_tuple` è  libera di calcolare direttamente la tupla *x̄* a
partire dalla chiave e  dalle sue conoscenze. In questo caso, inoltre, può decidere di restituire una tupla con un
numero di elementi inferiore al numero di livelli della rete. In questo caso la tupla
*x̄ *= *x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>j</sub>* quando viene passata alla funzione *H<sub>t</sub>* circoscrive la
sua ricerca dell'hash-node al g-nodo *n<sub>j+1</sub>* del nodo *n* che fa la richiesta.

