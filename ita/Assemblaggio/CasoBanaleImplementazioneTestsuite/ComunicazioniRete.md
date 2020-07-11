# Comunicazioni in rete

## Panoramica

Simuleremo due interfacce di rete, su due diversi segmenti di rete. In ognuno dei
due il nostro nodo sarà il solo connesso.

Questo comporta alcune conseguenze:

*   Il programma si troverà a trasmettere in broadcast (datagram) alcuni messaggi
    sull'una o l'altra interfaccia.
*   Il programma non si troverà mai a trasmettere in unicast (stream) alcun
    messaggio, poiché il nodo non costituirà nessun arco con altri nodi.
*   Il programma si troverà a rilevare alcuni messaggi in broadcast (datagram)
    sull'una o l'altra interfaccia. Saranno sempre messaggi provenienti dal
    nodo stesso.
*   Il programma non si troverà mai a rilevare alcun messaggio in unicast (stream).

## Strutture dati

In una classe memorizziamo le informazioni relative a una interfaccia di rete che
il programma deve gestire.  
Vedere la classe `N.HandledNic` nel file `main.vala`.  
I dati memorizzati includono `dev`, `mac`, `linklocal`, `INeighborhoodNetworkInterface nic`.

In una hashmap memorizziamo le istanze di `N.HandledNic` di ogni interfaccia di rete che il
programma deve gestire, indicizzate per il nome `dev`.  
Vedere la variabile globale `handlednic_map` nel file `main.vala`.

Gli oggetti in `handlednic_map` servono al programma in quanto coordina le operazioni
dei vari moduli.

* * *

In una classe memorizziamo le informazioni relative a una interfaccia di rete che
il programma deve gestire.  
Vedere la classe `N.PseudoNetworkInterface` nel file `main.vala`. I dati memorizzati
includono `dev`, `mac`, `linklocal`, `INeighborhoodNetworkInterface nic` e anche
informazioni legate al fatto che si tratta di pseudo-interfacce di rete simulate da
un socket di sistema: `send_pathname`, `listen_pathname`, `st_listen_pathname`.

In una hashmap memorizziamo le istanze di `N.PseudoNetworkInterface` di ogni
interfaccia di rete che il programma deve gestire, indicizzate per il nome
`dev`.  
Vedere la variabile globale `pseudonic_map` nel file `main.vala`.

Gli oggetti in `pseudonic_map` servono solo a questa specifica
testsuite in quanto deve fornire i parametri necessari ai metodi di ZCD a supporto
delle comunicazioni nel medium *system*.

## Gestione messaggi in entrata

La fase d'inizio ascolto su una interfaccia (in broadcast e in unicast sul
linklocal) va svolta in correlazione con quella di aggiunta della stessa
alla gestione del `N.N.NeighborhoodManager`.  
Analogamente la fase di fine ascolto va svolta in correlazione con
quella di rimozione dell'interfaccia dalla gestione del `N.N.NeighborhoodManager`.

Vedere la funzione `void stop_rpc(string dev)` nel file `main.vala`.

* * *

In una unica classe viene implementato il codice per avviare le tasklet che
si metteranno in ascolto per comunicazioni di rete in entrata. Queste tasklet
in questa particolare testsuite staranno in ascolto solo su socket di sistema;
non ci saranno reali comunicazioni di rete. Per quanto riguarda le modalità,
invece, anche nella presente testsuite saranno presenti entrambe:
stream e datagram.  
Vedere la classe `N.SkeletonFactory` nel file `rpc/skeleton_factory.vala`.

La classe è realizzata in modo che il programma ne debba
creare una sola istanza e memorizzarla in una variabile globale
`skeleton_factory` nel file `main.vala`, a uso comune di tutto il codice.

Nella fase di avvio, il programma (nella routine main) costruisce una
istanza di questa classe.  
Più avanti valorizzerà la sua proprietà pubblica `NeighborhoodNodeID whole_node_id`
con l'identificativo di questo nodo fornito dal modulo `Neighborhood`.

Il costruttore di `N.SkeletonFactory` valorizza la proprietà privata
`ServerDelegate dlg`. Servirà alle tasklet che ricevono i
messaggi, come illustreremo a breve.  
Inoltre valorizza la proprietà privata `NodeSkeleton node_skeleton`. Sarà
questa a memorizzare l'identificativo di questo nodo fornito passato
dalla routine main più avanti alla proprietà pubblica `whole_node_id`.

Il metodo `start_stream_system_listen` avvia una tasklet per gestire i messaggi
di tipo stream trasmessi in unicast a uno specifico indirizzo IP. Nella presente
testsuite il programma richiamerà questo metodo sia per gestire un IP linklocal,
sia per gestire un IP pubblico. In realtà non riceveremo mai tali messaggi.  
A questo metodo viene passata la stringa `listen_pathname`, come prescrive la
funzione `stream_system_listen` fornita dalla libreria `ntkdrpc` basata su ZCD.
Inoltre il metodo si costruisce un `N.IErrorHandler` apposito per questo
listener (la classe `ServerErrorHandler` è una privata inner-classe
di `N.SkeletonFactory`) e usa una istanza di `N.IDelegate` in comune con tutti
i listener (la classe `ServerDelegate` è una privata inner-classe
di `N.SkeletonFactory`); entrambe le classi sono prescritte da `ntkdrpc`.  
Infine il metodo `start_stream_system_listen` memorizza in un HashMap l'handle
della tasklet avviata associandolo alla stringa `listen_pathname`, di modo che
la stessa classe `N.SkeletonFactory` con il metodo `stop_stream_system_listen`
possa gestirne la terminazione.

Il metodo `start_datagram_system_listen` avvia una tasklet per gestire i
messaggi di tipo datagram trasmessi in broadcast su un segmento di rete
(broadcast domain) e recepiti da una certa interfaccia di rete
del nodo corrente. In realtà rileveremo tali messaggi solo quando sarà
il nostro stesso nodo a trasmetterli.  
A questo metodo viene passata la stringa `listen_pathname`, la stringa
`send_pathname` e una istanza di `ISrcNic src_nic`, come prescrive la funzione
`datagram_system_listen` fornita dalla libreria `ntkdrpc` basata su ZCD.
Inoltre il metodo si costruisce un `N.IErrorHandler` apposito per questo listener
e usa l'istanza comune di `N.IDelegate`, prescritte da `ntkdrpc`.  
Infine il metodo `start_datagram_system_listen` memorizza in un HashMap l'handle
della tasklet avviata associandolo alla stringa `listen_pathname`, di modo che
la stessa classe `N.SkeletonFactory` con il metodo `stop_datagram_system_listen`
possa gestirne la terminazione.

La classe ServerDelegate (nel metodo `get_addr_set` prescritto da `ntkdrpc`)
per prima cosa discerne se ha ricevuto un messaggio di tipo stream o di tipo
datagram. Il primo caso non è contemplato da questa testsuite, quindi
il codice semplicemente va in errore.  
Nel secondo caso la classe fa il suo lavoro usando il metodo `get_dispatcher_set`
di `N.SkeletonFactory` passando il CallerInfo fornito da ZCD.

Il metodo `get_dispatcher_set` è usato per gestire i messaggi in datagram. Gli viene
passata una istanza di `DatagramCallerInfo`. Dovrà restituire una lista di
*skeleton* che saranno chiamati a eseguire una procedura remota come effetto
del messaggio rilevato.  
A seconda del tipo di CallerInfo, il metodo capisce se il messaggio è destinato
a una identità o a un nodo. Inoltre capisce se il messaggio è destinato a qualcuno
in questo sistema.

*   Se il caller ha in `source_id` un `WholeNodeSourceID` e in `broadcast_id` un
    `EveryWholeNodeBroadcastID`:
    *   Siccome i messaggi di questo tipo sono destinati sempre a tutti i nodi,
        semplicemente restituisce in un set il proprio `node_skeleton`.
*   Se il caller ha in `source_id` un `IdentityAwareSourceID` e in `broadcast_id`
    un `IdentityAwareBroadcastID` e in `src_nic` un `NeighbourSrcNic` e
    in `listener` un `DatagramSystemListener`:
    *   Questo evento non si verifica mai poiché non ci saranno archi. Il codice
        semplicemente va in errore.

### Reperimento info aggiuntive dal CallerInfo

Sempre nella classe `N.SkeletonFactory` nel file `rpc/skeleton_factory.vala`, ci
sono metodi che gli skeleton usano durante l'esecuzione di un metodo remoto, per
avere informazioni aggiuntive sul chiamante del metodo.

Il metodo `from_caller_get_nodearc` è usato per richieste di nodo, per sapere
da quale arco una tale richiesta è arrivata.  
In questa testsuite non si formeranno mai archi, quindi
il metodo semplicemente non esiste.

### Reperimento degli Skeleton specifici di modulo

Sempre nel file `rpc/skeleton_factory.vala`, il `NodeSkeleton node_skeleton`
è uno *skeleton* per i metodi remoti di nodo. Ci sarà
una sola istanza che rappresenta il sistema. Ha il membro pubblico
`NeighborhoodNodeID id` che contiene il suo identificativo di nodo.

Restituisce con appositi getter i manager del modulo Neighborhood e
del modulo Identities, che sono moduli di nodo. Quindi sono implementati
i metodi `neighborhood_manager_getter` e `identity_manager_getter`.

* * *

Sempre nel file `rpc/skeleton_factory.vala`, la classe `IdentitySkeleton`
è uno *skeleton* per i metodi remoti di identità.
Ad ogni messaggio ricevuto viene costruita
una nuova istanza passando al costruttore l'istanza di `IdentityData`.

Restituisce con appositi getter i manager dei moduli di identità:
`N.IQspnManagerSkeleton`, `N.IPeersManagerSkeleton`, `N.ICoordinatorManagerSkeleton`,
`N.IHookingManagerSkeleton`, `N.IAndnaManagerSkeleton`.  
È implementato il `qspn_manager_getter` che prevede una attesa di qualche istante
se non è stato ancora costruito.  
È implementato il `peers_manager_getter` che prevede una attesa di qualche istante
se non è stato ancora costruito.  
È implementato il `coordinator_manager_getter` che va semplicemente in errore
se non è stato ancora costruito.  
È implementato il `hooking_manager_getter` che va semplicemente in errore
se non è stato ancora costruito.  
Va ancora fatto in `ntkdrpc` quanto serve a produrre il metodo astratto `andna_manager_getter`. Dopo
andrà implementato il `andna_manager_getter`.

## Gestione messaggi in uscita

In una unica classe viene implementato il codice per ottenere degli stub
radice di vario tipo (broadcast, unicast, di nodo o d'identità).  
Vedere la classe `N.StubFactory` nel file `rpc/stub_factory.vala`.

La classe è realizzata in modo che il programma ne debba
creare una sola istanza e memorizzarla in una variabile globale
`stub_factory` nel file `main.vala`, a uso comune di tutto il codice.

### Trasmissione broadcast a modulo di nodo

Per avere uno stub radice di tipo broadcast verso un modulo di nodo, poiché
l'unico caso d'uso è quello della funzione di radar del modulo `Neighborhood`,
si chiama il metodo `get_stub_whole_node_broadcast_for_radar` passando
l'interfaccia di rete gestita, cioè l'istanza
di `N.N.INeighborhoodNetworkInterface`.  
Il metodo prepara i dati serializzabili che sono previsti dal protocollo
ZCD, cioè:

*   Un `WholeNodeSourceID` che rappresenta il mittente.  
    Il `NeighborhoodNodeID`, che serve per crearlo, viene preso dalla variabile
    globale `skeleton_factory`.
*   Un `EveryWholeNodeBroadcastID` che rappresenta il destinatario, che è chiunque
    in questo caso.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente)
    usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) viene preso dal
    membro `mac` dell'istanza di `N.N.INeighborhoodNetworkInterface` passata
    al metodo.

Poi il metodo usa la libreria `ntkdrpc` basata su ZCD per ottenere uno stub
radice di tipo datagram sul medium system; cioè chiama la funzione
`get_addr_datagram_system`.  
Per comporre la stringa `send_pathname` prescritta da `ntkdrpc`, la classe
`N.StubFactory` usa l'identificativo univoco del processo (che è memorizzato
nella variabile globale `pid` nel file `main.vala`) e il nome della
pseudo interfaccia di rete, che viene preso dal membro `dev` dell'istanza di
`N.N.INeighborhoodNetworkInterface` passata al metodo.

### Trasmissione broadcast a modulo di identità

La presente testsuite non necessiterà mai di uno stub per inviare messaggi
di tipo broadcast verso un modulo di identità.  
La funzione `get_stub_identity_aware_broadcast` e altre a corredo sono
state riportate nel file come commento.

### Trasmissione unicast a modulo di nodo

La presente testsuite non necessiterà mai di uno stub per inviare messaggi
di tipo unicast.  
La funzione `get_stub_whole_node_unicast` è stata riportata nel file come commento.  
La funzione `get_stub_identity_aware_unicast_from_ia` è stata riportata nel file come
commento.

### Stub specifici di modulo

Diverse classi (una o più per ogni modulo) realizzano gli stub dedicati.  
Vedere le classi nel file `rpc/module_stubs.vala`.

La classe `N.NeighborhoodManagerStubHolder` implementa l'interfaccia
`N.INeighborhoodManagerStub` a partire da uno stub radice.

La classe `N.IdentityManagerStubHolder` implementa l'interfaccia
`N.IIdentityManagerStub` a partire da uno stub radice.

La classe `N.QspnManagerStubHolder` implementa l'interfaccia
`N.IQspnManagerStub` a partire da uno stub radice.  
In questa testsuite non sarà mai usata, poiché non si realizzano archi.

La classe `N.QspnManagerStubBroadcastHolder` implementa l'interfaccia
`N.IQspnManagerStub` a partire da una lista di stub radice.  
In questa testsuite non sarà mai usata, poiché non si realizzano archi.

La classe `N.QspnManagerStubVoid` implementa l'interfaccia
`N.IQspnManagerStub` semplicemente ignorando i messaggi che gli sono passati.
Infatti viene restituita al modulo `Qspn` quando questi chiama il metodo
`i_qspn_get_broadcast` passando una lista di archi vuota.

La classe `N.PeersManagerStubHolder` implementa l'interfaccia
`N.IPeersManagerStub` a partire da uno stub radice.  
In questa testsuite non sarà mai usata, poiché non si realizzano archi.

La classe `N.PeersManagerStubVoid` implementa l'interfaccia
`N.IPeersManagerStub` semplicemente ignorando i messaggi che gli sono passati.
Infatti viene restituita al modulo `PeerServices` quando questi chiama il metodo
`i_peers_get_broadcast` e l'identità associata non ha alcun arco-identità
di cui è stato fatto un arco-qspn.

La classe `N.CoordinatorManagerStubHolder` implementa l'interfaccia
`N.ICoordinatorManagerStub` a partire da uno stub radice.  
In questa testsuite non sarà mai usata, poiché non si realizzano archi.

La classe `N.CoordinatorManagerStubVoid` implementa l'interfaccia
`N.ICoordinatorManagerStub` semplicemente ignorando i messaggi che gli sono passati.
Infatti viene restituita al modulo `Coordinator` quando questi chiama il metodo
`get_stub_for_all_neighbors` e l'identità associata non ha alcun arco-identità
di cui è stato fatto un arco-qspn.

La classe `N.HookingManagerStubHolder` implementa l'interfaccia
`N.IHookingManagerStub` a partire da uno stub radice.  
In questa testsuite non sarà mai usata, poiché non si realizzano archi.

**TODO** altri moduli
