## Ciclo di vita delle istanze di ogni modulo

A regime, ci sarà una istanza di NeighborhoodManager. Questa si crea all'avio del programma e muore alla sua terminazione.

A regime, ci sarà una istanza di IdentityManager. Questa si crea all'avvio del programma e muore alla sua terminazione.

A regime, ci saranno *n* istanze di QspnManager. Una per l'identità principale e una per ogni identità di connettività.
La prima si crea all'avvio del programma. Una istanza "di connettività" può morire quando non serve più la relativa identità
di connettività. L'istanza "principale" ultima muore alla terminazione del programma.

A regime, ci saranno *n* istanze di PeerManager.

## Cosa serve alla creazione della sola istanza di NeighborhoodManager

`int max_arcs` - Un limite massimo al numero di archi che il modulo crea con i nodi diretti vicini.

`INeighborhoodStubFactory stub_factory` - Una factory per creare stub RPC.

`INeighborhoodQueryCallerInfo query_caller_info` - Un discriminatore per gli oggetti CallerInfo che si ricevono nelle
chiamate di metodi remoti.

`INeighborhoodIPRouteManager ip_mgr` - Per chiamare i comandi iproute necessari.

`NewLinklocalAddress new_linklocal_address` - Un delegato per scegliere un indirizzo IP linklocal per le proprie schede di rete.

## Cosa serve alla creazione della sola istanza di IdentityManager

`List<string> if_list_dev` - Lista dei nomi delle proprie schede di rete.

`List<string> if_list_mac` - Lista dei MAC delle proprie schede di rete.

`List<string> if_list_linklocal` - Lista degli indirizzi IP linklocal assegnati alle proprie schede di rete.

`IIdmgmtNetnsManager netns_manager` - Per chiamare i comandi iproute necessari alla creazione e gestione di network namespace per
le diverse identità di connettività.

`IIdmgmtStubFactory stub_factory` - Una factory per creare stub RPC e un discriminatore per gli oggetti CallerInfo
che si ricevono nelle chiamate di metodi remoti.

`NewLinklocalAddress new_linklocal_address` - Un delegato per scegliere un indirizzo IP linklocal per le proprie
pseudo-schede di rete nei network namespace per le diverse identità di connettività.

## Cosa serve alla inizializzazione del modulo Qspn

`int max_paths` - Un limite massimo al numero di percorsi memorizzati dal modulo per ogni destinazione.

`double max_common_hops_ratio` - Un coefficiente di disgiunzione dei percorsi.

`int arc_timeout` - Il tempo di attesa perché il modulo Qspn nel nodo diretto vicino riconosca l'arco.

`IQspnThresholdCalculator threshold_calculator` - Un delegato che decida quanto tempo attendere per segnalare lo split
di un g-nodo, dati i due percorsi dai quali arrivano informazioni discordanti sul fingerprint dello stesso.

## Cosa serve alla creazione della prima istanza di QspnManager

`IQspnMyNaddr my_naddr` - Il proprio indirizzo Netsukuku.

`IQspnFingerprint my_fingerprint` - Il proprio fingerprint a livello di nodo.

`IQspnStubFactory stub_factory` - Una factory per creare stub RPC.

## Cosa serve alla creazione di una nuova istanza di QspnManager per migrazione

## Cosa serve alla creazione di una nuova istanza di QspnManager per ingresso



### Annotazioni

Per indicare i namespaces usiamo in seguito le abbreviazioni:

`G.` per `Gee`;  
`N.` per `Netsukuku`;  
`N.N.` per `Netsukuku.Neighborhood`;  
`N.I.` per `Netsukuku.Identities`;  
`N.Q.` per `Netsukuku.Qspn`;  
`N.P.` per `Netsukuku.PeerServices`;  
`N.C.` per `Netsukuku.Coordinator`;  
`N.H.` per `Netsukuku.Hooking`;  
`N.A.` per `Netsukuku.Andna`;  
`T.` per `TaskletSystem`;  

* * *

Dopo la creazione della sola istanza di NeighborhoodManager, viene chiamato il suo metodo `start_monitor` per
iniziare a gestire una scheda di rete. Di conseguenza quasi immediatamente viene emesso il suo segnale `neighborhood_nic_address_set` e
il programma viene a conoscenza dell'indirizzo IP linklocal associato a quella scheda di rete.

Nella gestione di questo segnale, in `handlednic_map` viene memorizzata una istanza di `HandledNic` che mantiene questa info.

Poco dopo si ha la creazione della sola istanza di IdentityManager. Quindi il programma ha le informazioni che servono a
compilare non solo `if_list_dev` ma anche `if_list_mac` e `if_list_linklocal`.

* * *

Bisogna poter dare i comandi uno alla volta. E avere la possibilità di dare una serie di comandi
senza che si possano introdurre in mezzo altri comandi.  
Vedere il file `commander.vala` e `fake_command_dispatcher.vala`.

* * *

La classe `N.PRNGen` nel file `rngen.vala` con i suoi metodi statici `init_rngen` e `int_range` viene usata nel codice
per generare numeri pseudo-casuali. Può essere inizializzato questo generatore con un valore costante per ottenere
risultati deterministici, ad esempio per riprodurre nei test gli stessi risultati. Oppure si possono ottenere
valori più casuali.

* * *

In una classe memoriziamo le informazioni relative ad una interfaccia di rete che il programma deve gestire.  
Vedere la classe `N.PseudoNetworkInterface` nel file `system_ntkd.vala`.

In una hashmap memoriziamo le istanze di `N.PseudoNetworkInterface` di ogni interfaccia di rete che il
programma deve gestire, indicizzate per il nome `dev`.  
Vedere la variabile globale `pseudonic_map` nel file `system_ntkd.vala`.

* * *

Bisogna integrare l'unica istanza di `N.N.NeighborhoodManager` che si crea all'avio del programma e muore alla sua terminazione.

* * *

La fase di inizio ascolto su una interfaccia (in broadcast e in unicast sul linklocal) va svolta in correlazione con
la fase di aggiunta dell'interfaccia alla gestione del `N.N.NeighborhoodManager`.  
Analogamente la fase di fine ascolto va svolta in correlazione con
la fase di rimozione dell'interfaccia dalla gestione del `N.N.NeighborhoodManager`.

Vedere la funzione `void stop_monitor(string dev)` nel file `system_ntkd.vala`.

* * *

Per integrare il modulo `Neighborhood` occorre implementare con una classe l'interfaccia `N.N.INeighborhoodIPRouteManager`.  
Vedere la classe `N.NeighborhoodIPRouteManager` nel file `neighborhood_helpers.vala`.

Quando il modulo `Neighborhood` chiama `add_address` di `N.N.INeighborhoodIPRouteManager` la classe
suddetta deve aggiungere l'indirizzo IP passato (si tratterà di un linklocal) alla interfaccia di
rete (reale) passata.  
Lo fa chiedendo al `commander.vala` di eseguire un singolo comando "`ip address add ...`".

Quando il modulo `Neighborhood` chiama `add_neighbor` di `N.N.INeighborhoodIPRouteManager` la classe
suddetta deve aggiungere una route diretta all'indirizzo IP passato (si tratterà di un linklocal) tramite
l'interfaccia di rete (reale) passata.  
Lo fa chiedendo al `commander.vala` di eseguire un singolo comando "`ip route add ...`".

Quando il modulo `Neighborhood` chiama `remove_neighbor` di `N.N.INeighborhoodIPRouteManager` la classe
suddetta deve rimuovere una route diretta all'indirizzo IP passato (si tratterà di un linklocal) tramite
l'interfaccia di rete (reale) passata.  
Lo fa chiedendo al `commander.vala` di eseguire un singolo comando "`ip route del ...`".

Quando il modulo `Neighborhood` chiama `remove_address` di `N.N.INeighborhoodIPRouteManager` la classe
suddetta deve rimuovere l'indirizzo IP passato (si tratterà di un linklocal) alla interfaccia di
rete (reale) passata.  
Lo fa chiedendo al `commander.vala` di eseguire un singolo comando "`ip address del ...`".

* * *

Per integrare il modulo `Neighborhood` occorre implementare con una classe l'interfaccia `N.N.INeighborhoodStubFactory`.  
Vedere la classe `N.NeighborhoodStubFactory` nel file `neighborhood_helpers.vala`.

Quando il modulo `Neighborhood` chiama `get_broadcast_for_radar` di `N.N.INeighborhoodStubFactory` la classe
suddetta deve restituire uno stub per mandare messaggi broadcast allo stesso modulo `Neighborhood` nei
nodi diretti vicini attraverso una specifica interfaccia di rete.  
Lo fa chiamando sulla classe `N.StubFactory` il metodo `get_stub_whole_node_broadcast_for_radar` che gli
ottiene una istanza di stub per messaggi broadcast di nodo. Lo stub ottenuto è radice (si veda `AddressManager addr`
nel file `interfaces.rpcidl` del pacchetto `ntkdrpc`). Per ottenere uno stub dedicato al modulo `Neighborhood`
la classe istanzia un `NeighborhoodManagerStubHolder`.

Quando il modulo `Neighborhood` chiama `get_unicast` di `N.N.INeighborhoodStubFactory` la classe
suddetta deve restituire uno stub per mandare messaggi unicast allo stesso modulo `Neighborhood` in uno
specifico nodo diretto vicino attraverso uno specifico arco `N.N.INeighborhoodArc`.  
Lo fa chiamando sulla classe `N.StubFactory` il metodo `get_stub_whole_node_unicast` che gli
ottiene una istanza di stub per messaggi unicast di nodo. Lo stub ottenuto è radice (si veda `AddressManager addr`
nel file `interfaces.rpcidl` del pacchetto `ntkdrpc`). Per ottenere uno stub dedicato al modulo `Neighborhood`
la classe istanzia un `NeighborhoodManagerStubHolder`.

* * *

Per integrare il modulo `Neighborhood` occorre implementare con una classe l'interfaccia `N.N.INeighborhoodQueryCallerInfo`.  
Vedere la classe `N.NeighborhoodQueryCallerInfo` nel file `neighborhood_helpers.vala`.

Quando il modulo `Neighborhood` chiama `is_from_broadcast` di `N.N.INeighborhoodQueryCallerInfo` la classe
suddetta deve esaminare una istanza di `CallerInfo` (la classe che la libreria `ntkdrpc` basata su ZCD
passa ai metodi remoti quando riceve una chiamata da un altro nodo) e stabilire se il metodo
remoto è stato chiamato da un diretto vicino in modalità broadcast; e in quel caso deve restituire
l'istanza di `N.N.INeighborhoodNetworkInterface` associata all'interfaccia di rete da cui il messaggio è
stato ricevuto.  
Lo fa chiamando sulla classe `N.SkeletonFactory` il metodo `from_caller_get_mydev` che gli restituisce la
stringa con il nome dell'interfaccia da cui è stato ricevuto il messaggio. Tramite questo nome e
l'hashmap `pseudonic_map` risale all'istanza di `N.PseudoNetworkInterface` e in essa trova l'istanza
di `N.N.INeighborhoodNetworkInterface` che deve restituire.

Quando il modulo `Neighborhood` chiama `is_from_unicast` di `N.N.INeighborhoodQueryCallerInfo` la classe
suddetta deve esaminare una istanza di `CallerInfo` e una lista di archi `N.N.INeighborhoodArc`
che gli sono passati e stabilire se il metodo remoto è stato chiamato da un diretto vicino in modalità unicast;
e in quel caso deve restituire l'arco `N.N.INeighborhoodArc` da cui il messaggio è
stato ricevuto.  
Lo fa individuando dal `CallerInfo` l'interfaccia di rete che ha ricevuto e il MAC address dell'interfaccia
di rete del vicino che ha trasmesso (contenuta secondo il protocollo ZCD nella classe `NeighbourSrcNic`
nella classe `CallerInfo`). Poi cicla negli archi e cerca quello che corrisponde.

* * *

Per integrare il modulo `Neighborhood` occorre implementare con una classe l'interfaccia `N.N.INeighborhoodNetworkInterface`.  
Vedere la classe `N.NeighborhoodNetworkInterface` nel file `neighborhood_helpers.vala`.

Le istanze di questa classe sono da passare al modulo `Neighborhood` a rappresentazione di una interfaccia di rete
da gestire. Una istanza è creata passando una istanza di `N.PseudoNetworkInterface`. A sua volta una istanza di
di `N.PseudoNetworkInterface` contiene un riferimento (**TODO** mettere weak) alla relativa istanza di `N.N.INeighborhoodNetworkInterface`.

Quando il modulo `Neighborhood` chiama `measure_rtt` di `N.N.INeighborhoodNetworkInterface` la classe
suddetta deve misurare il Round Trip Time delle comunicazioni tramite questa interfaccia di rete con un
indirizzo IP linklocal che identifica una precisa interfaccia di rete di un nodo diretto vicino.  
Questo avverrà eseguendo il comando `ping` e interpretandone l'output. Per il momento invece, questo
programma di test chiede al `commander.vala` di eseguire un singolo comando "`ping`" e restituisce
una misurazione fake di 1000 usec.

* * *

Per reagire ai segnali emessi dal modulo `Neighborhood` sono implementate delle funzioni
nel file `neighborhood_signals.vala`.

Il segnale `nic_address_set` è gestito nella funzione `neighborhood_nic_address_set`.  
La funzione recupera l'istanza di `N.PseudoNetworkInterface` dal set `pseudonic_map`
e valorizza i suoi membri `linklocal` e `st_listen_pathname`.  
Inoltre avvia in questo momento la tasklet in ascolto per i messaggi stream unicast verso
questa interfaccia di rete (con `skeleton_factory.start_stream_system_listen`).


Il segnale `arc_added` è gestito nella funzione `neighborhood_arc_added`.  
**TODO** in futuri commit.

Il segnale `arc_changed` è gestito nella funzione `neighborhood_arc_changed`.  
**TODO** in futuri commit.

Il segnale `arc_removing` è gestito nella funzione `neighborhood_arc_removing`.  
**TODO** in futuri commit.

Il segnale `arc_removed` è gestito nella funzione `neighborhood_arc_removed`.  
**TODO** in futuri commit.

Il segnale `nic_address_unset` è gestito nella funzione `neighborhood_nic_address_unset`.  
**TODO** in futuri commit.

* * *

Bisogna potersi mettere in ascolto e inviare comunicazioni agli altri nodi:

*   Su una interfaccia di rete in broadcast.  
    Per mettersi in ascolto serve il nome dell'interfaccia di rete. Questo viene passato all'avvio del programma.
*   Su un indirizzo linklocal in unicast.  
    Per mettersi in ascolto serve l'indirizzo IP linklocal assegnato all'interfaccia di rete. Questo
    viene scelto dal `N.N.NeighborhoodManager` che lo notifica con un segnale.
*   Su un indirizzo routabile in unicast.  
    Per mettersi in ascolto serve l'indirizzo IP routabile assegnato al nodo. Questo è calcolato sulla base
    dell'indirizzo Netsukuku del nodo, il quale viene gestito dal modulo `Qspn`.

* * *

In una unica classe viene implementato il codice per ottenere degli stub radice di vario tipo (broadcast, unicast,
di nodo o di identità).  
Vedere la classe `N.StubFactory` nel file `rpc/stub_factory.vala`.

La classe è pensata per avere una sola istanza in una variabile globale `stub_factory`
nel file `system_ntkd.vala`, ad uso comune di tutto il codice.

Per avere uno stub radice di tipo unicast verso un modulo di nodo si chiama il metodo `get_stub_whole_node_unicast`
passando un arco `N.N.INeighborhoodArc` e indicando se si vuole attendere la risposta con `wait_reply`.  
La classe prepara i dati serializzabili che sono previsti dal protocollo ZCD, cioè:

*   Un `WholeNodeSourceID` che rappresenta il mittente.  
    Il `NeighborhoodNodeID` che serve viene preso dalla variabile globale `skeleton_factory`.
*   Un `WholeNodeUnicastID` che rappresenta il destinatario.  
    Il `NeighborhoodNodeID` che serve viene preso dal membro `neighbour_id` dell'arco passato al metodo.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente) usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) viene preso dal membro `mac` dell'istanza di
    `N.N.INeighborhoodNetworkInterface` che è presa dal membro `nic` dell'arco passato al metodo.

Poi la classe chiama il metodo di `ntkdrpc` che restituisce lo stub radice di tipo stream.  
In particolare in questo programma di test si chiama il metodo che usa il medium system. Per questo serve
la stringa `send_pathname` che la classe costruisce a partire dall'indirizzo IP linklocal dell'interfaccia
di rete del destinatario, il quale è memorizzato in `neighbour_nic_addr`
nell'arco passato al metodo.  
Quindi con tutti questi dati la classe chiama il metodo `get_addr_stream_system`.

Per avere uno stub radice di tipo broadcast verso un modulo di nodo, poiché l'unico caso d'uso è quello
della funzione di radar del modulo `Neighborhood`, si chiama il metodo `get_stub_whole_node_broadcast_for_radar`
passando l'interfaccia di rete gestita, cioè l'istanza di `N.N.INeighborhoodNetworkInterface`.  
La classe prepara i dati serializzabili che sono previsti dal protocollo ZCD, cioè:

*   Un `WholeNodeSourceID` che rappresenta il mittente.  
    Il `NeighborhoodNodeID` che serve viene preso dalla variabile globale `skeleton_factory`.
*   Un `EveryWholeNodeBroadcastID` che rappresenta il destinatario, che è chiunque in questo caso.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente) usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) viene preso dal membro `mac` dell'istanza di
    `N.N.INeighborhoodNetworkInterface` passata al metodo.

Poi la classe chiama il metodo di `ntkdrpc` che restituisce lo stub radice di tipo datagram.  
In particolare in questo programma di test si chiama il metodo che usa il medium system. Per questo serve
la stringa `send_pathname` che la classe costruisce a partire dall'identificativo univoco del processo
(che è memorizzato nella variabile globale `pid` nel file `system_ntkd.vala`) e dal nome della pseudo interfaccia
di rete, che viene preso dal membro `dev` dell'istanza di `N.N.INeighborhoodNetworkInterface` passata al metodo.  
Quindi con tutti questi dati la classe chiama il metodo `get_addr_datagram_system`.

* * *

Diverse classi (una o più per ogni modulo) realizzano gli stub dedicati.  
Vedere le classi nel file `rpc/module_stubs.vala`.

La classe `N.NeighborhoodManagerStubHolder` implementa l'interfaccia `N.INeighborhoodManagerStub`
a partire da uno stub radice.

* * *

In una unica classe viene implementato il codice per avviare le tasklet in ascolto nelle varie modalità
(stream, datagram, di rete, di sistema, ...).  
Vedere la classe `N.SkeletonFactory` nel file `rpc/skeleton_factory.vala`.

La classe è pensata per avere una sola istanza in una variabile globale `skeleton_factory`
nel file `system_ntkd.vala`, ad uso comune di tutto il codice.

Nella fase di avvio del nodo, dopo averne costruito l'istanza, viene valorizzata la sua
proprietà `NeighborhoodNodeID whole_node_id` con l'identificativo fornito dal modulo `Neighborhood`
(**TODO** da finire con un successivo commit).

Il costruttore valorizza la proprietà privata `ServerDelegate dlg`. Servirà alle tasklet che ricevono i
messaggi, come illustreremo a breve.  
Inoltre valorizza la proprietà privata `NodeSkeleton node_skeleton`. Questa classe privata implementa
l'interfaccia `N.IAddressManagerSkeleton` con il compito di restituire i vari manager dei moduli
di nodo: `N.INeighborhoodManagerSkeleton` e `N.IIdentityManagerSkeleton`.

Il metodo `start_stream_system_listen` avvia una tasklet per gestire i messaggi di tipo stream trasmessi
in unicast a uno specifico indirizzo IP. Si può usare sia per i messaggi ricevuti da un diretto vicino
(cioè con un IP linklocal) sia per i messaggi ricevuti da un nodo qualsiasi (instradati) all'interno di
un g-nodo di qualsiasi livello in cui si trova anche il nodo corrente (cioè con un IP pubblico).  
A questo metodo viene passata la stringa `listen_pathname`, come prescrive la funzione `stream_system_listen`
fornita dalla libreria `ntkdrpc` basata su ZCD. Inoltre il metodo si costruisce un `N.IErrorHandler`
apposito per questo listener (la classe `ServerErrorHandler` è una privata inner-classe di `N.SkeletonFactory`)
e usa una istanza di `N.IDelegate` in comune con tutti i listener (la classe `ServerDelegate` è una
privata inner-classe di `N.SkeletonFactory`); entrambe le classi sono prescritte da `ntkdrpc`.  
Infine il metodo `start_stream_system_listen` memorizza in un HashMap l'handle della tasklet avviata
associandolo alla stringa `listen_pathname`, di modo che la stessa classe `N.SkeletonFactory` con il
metodo `stop_stream_system_listen` possa gestirne la terminazione.

Il metodo `start_datagram_system_listen` avvia una tasklet per gestire i messaggi di tipo datagram trasmessi
in broadcast su un segmento di rete (broadcast domain) e recepiti da una certa interfaccia di rete
del nodo corrente.  
A questo metodo viene passata la stringa `listen_pathname`, la stringa `send_pathname` e una istanza di
`ISrcNic src_nic`, come prescrive la funzione `datagram_system_listen`
fornita dalla libreria `ntkdrpc` basata su ZCD.
Inoltre il metodo si costruisce un `N.IErrorHandler` apposito per questo listener
e usa l'istanza comune di `N.IDelegate`, prescritte da `ntkdrpc`.  
Infine il metodo `start_datagram_system_listen` memorizza in un HashMap l'handle della tasklet avviata
associandolo alla stringa `listen_pathname`, di modo che la stessa classe `N.SkeletonFactory` con il
metodo `stop_datagram_system_listen` possa gestirne la terminazione.

La classe ServerDelegate (nel metodo `get_addr_set` prescritto da `ntkdrpc`) fa il suo lavoro
usando i metodi `get_dispatcher` e `get_dispatcher_set` passando il CallerInfo.

Il metodo `get_dispatcher` è usato per gestire i messaggi in stream:

*   Se il caller ha un `WholeNodeSourceID` e un `WholeNodeUnicastID` che referenzia il nodo corrente
    allora restituisce il proprio `node_skeleton`.
*   ... **TODO** in futuri commit.

Il metodo `get_dispatcher_set` è usato per gestire i messaggi in datagram:

*   Se il caller ha un `WholeNodeSourceID` e un `EveryWholeNodeBroadcastID`
    allora restituisce in un set il proprio `node_skeleton`.
*   ... **TODO** in futuri commit.

Il metodo `from_caller_get_mydev`, se il caller è un `StreamCallerInfo`, ... **TODO** in futuri commit.

Il metodo `from_caller_get_mydev`, se il caller è un `DatagramCallerInfo`, ciclando le istanze
di `N.PseudoNetworkInterface` nel set `pseudonic_map` trova quella i cui `listen_pathname` e `send_pathname`
corrispondono al `listener` del caller e se lo trova restituisce il nome dell'interfaccia
di rete da cui il messaggio è stato ricevuto.

Il metodo `from_caller_get_nodearc`, ... **TODO** in futuri commit.

* * *

Nel file `serializables.vala` sono implementate le classi serializzabili che servono per comunicare
con il protocollo ZCD e i metodi definiti nel pacchetto `ntkdrpc`.

La classe `N.WholeNodeSourceID` implementa l'interfaccia `N.ISourceID` fornita da `ntkdrpc`. Contiene una
istanza di `NeighborhoodNodeID id` che rappresenta il mittente.

La classe `N.WholeNodeUnicastID` implementa l'interfaccia `N.IUnicastID` fornita da `ntkdrpc`. Contiene una
istanza di `NeighborhoodNodeID neighbour_id` che rappresenta il vicino destinatario.

La classe `N.EveryWholeNodeBroadcastID` implementa l'interfaccia `N.IBroadcastID` fornita da `ntkdrpc`. Essa
non contiene dati; rappresenta infatti chiunque riceva il messaggio.

La classe `N.NeighbourSrcNic` implementa l'interfaccia `N.ISrcNic` fornita da `ntkdrpc`. Contiene una
istanza di `string mac` che identifica la specifica interfaccia di rete del mittente.

* * *

All'**avvio del programma** nella funzione `main` del file `system_ntkd.vala`, per prima cosa si parserizzano
le opzioni date sulla linea di comando.  
Si deve passare un PID per identificare un processo che simula un nodo in un testsuite.  
Si deve passare una lista di pseudo-interfacce di rete da gestire.  

Il PID finisce nella variabile globale `int pid` nel file `system_ntkd.vala`.  
La lista di interfacce finisce in `ArrayList<string> devs` che è locale nella funzione `main`.  

Dopo si inizializza lo scheduler delle tasklet. Va nella variabile globale `ITasklet tasklet`.

Dopo si inizializza un FakeCommandDispatcher. Va nella variabile globale `FakeCommandDispatcher fake_cm`.

Dopo si inizializzano i singoli moduli (principalmente perché hanno delle classi serializzabili) e si
registrano le classi serializzabili che servono.  
Sono `WholeNodeSourceID`, `WholeNodeUnicastID`, `EveryWholeNodeBroadcastID`, `NeighbourSrcNic`, ...

Dopo si inizializza il generatore di numeri pseudo-casuali. Si usa come seed il PID.

Dopo si passa lo scheduler delle tasklet alla libreria `ntkdrpc`, chiamandone la funzione statica `init_tasklet_system`.

Dopo si istanzia lo `skeleton_factory`.

Dopo si istanzia lo `stub_factory`.

Dopo si istanzia il NeighborhoodManager.

Dopo si connettono i segnali del NeighborhoodManager.

Per ogni pseudo-interfaccia di rete da gestire si eseguono questi compiti:

*   Si genera uno pseudo MAC.
*   Si memorizza una nuova istanza di `N.PseudoNetworkInterface` in `pseudonic_map`.
*   Si eseguono dei comandi di inizializzazione della scheda di rete.
*   Si avvia una tasklet di ascolto per i messaggi datagram broadcast ricevuti da
    questa interfaccia di rete (con `skeleton_factory.start_datagram_system_listen`).
*   Si avvia `neighborhood_mgr.start_monitor` con l'istanza di `N.PseudoNetworkInterface`; questo
    imposterà il IP linklocal sulla scheda, e emetterà il segnale `nic_address_set` gestito come già
    detto sopra.

Dopo si entra nel main loop degli eventi fino alla terminazione del programma (con il segnale Posix
di terminazione).

Alla terminazione del programma, si chiama `stop_monitor` su tutte le pseudo-interfacce di rete gestite.

Dopo si rimuove il NeighborhoodManager (dalla variabile globale).

Dopo si termina lo scheduler delle tasklet.

* * *
