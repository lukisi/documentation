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

In una classe memoriziamo le informazioni relative ad un arco di cui il programma viene a conoscenza.  
Questo arco è tra una specifica interfaccia di rete di questo nodo e una specifica interfaccia di rete di un altro nodo
diretto vicino.  
Vedere la classe `N.NodeArc` nel file `system_ntkd.vala`.

Una istanza di `N.NodeArc` viene costruita passando una istanza di `N.N.INeighborhoodArc` e una istanza
di `N.I.IdmgmtArc`. La prima viene prodotta dal modulo `Neighborhood` quando l'arco è rilevato e viene comunicata
tramite un segnale al programma. La seconda viene prodotta dal programma per poi comunicarla al modulo `Identities`.
Entrambe le istanze possono essere in seguito reperite dall'istanza di `N.NodeArc`.

In una lista memoriziamo le istanze di `N.NodeArc` di ogni arco di cui il programma viene a conoscenza.  
Vedere la variabile globale `arc_list` nel file `system_ntkd.vala`.

* * *

Bisogna integrare l'unica istanza di `N.N.NeighborhoodManager` che si crea all'avio del programma e muore alla sua terminazione.

* * *

La fase di inizio ascolto su una interfaccia (in broadcast e in unicast sul linklocal) va svolta in correlazione con
la fase di aggiunta dell'interfaccia alla gestione del `N.N.NeighborhoodManager`.  
Analogamente la fase di fine ascolto va svolta in correlazione con
la fase di rimozione dell'interfaccia dalla gestione del `N.N.NeighborhoodManager`.

Vedere la funzione `void stop_rpc(string dev)` nel file `system_ntkd.vala`.

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
La funzione riceve un `N.N.INeighborhoodArc` che è stato appena aggiunto
dal modulo `Neighborhood`. Con esso crea un
`N.IdmgmtArc` (che implementa `N.I.IIdmgmtArc`). Con entrambi crea un
`N.NodeArc` che mette nella lista `arc_list`.  
Inoltre comunica questo arco al modulo `Identities` usando il metodo `identity_mgr.add_arc`.

Il segnale `arc_changed` è gestito nella funzione `neighborhood_arc_changed`.  
**TODO** in futuri commit; il commento dice:  
// TODO for each identity, for each id-arc, if qspn_arc is present, change cost.

Il segnale `arc_removing` è gestito nella funzione `neighborhood_arc_removing`.  
La funzione riceve un `N.N.INeighborhoodArc` che sta per essere rimosso
dal modulo `Neighborhood`. Cerca la relativa istanza di `N.NodeArc` nella lista `arc_list`,
vi recupera l'istanza di `N.I.IIdmgmtArc` e comunica la rimozione di questo
arco al modulo `Identities` usando il metodo `identity_mgr.remove_arc`.

Il segnale `arc_removed` è gestito nella funzione `neighborhood_arc_removed`.  
La funzione riceve un `N.N.INeighborhoodArc` che è stato appena rimosso
dal modulo `Neighborhood`. Rimuove la relativa istanza di `N.NodeArc` dalla lista `arc_list`.

Il segnale `nic_address_unset` è gestito nella funzione `neighborhood_nic_address_unset`.  
**TODO** in futuri commit.

* * *

Per integrare il modulo `Identities` occorre implementare con una classe l'interfaccia `N.I.IIdmgmtNetnsManager`.  
Vedere la classe `N.IdmgmtNetnsManager` nel file `identities_helpers.vala`.

Quando il modulo `Identities` chiama `create_namespace` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve preparare un nuovo network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire i comandi "`ip netns add ...`",
e alcuni comandi "`ip netns exec ... sysctl ...`".

Quando il modulo `Identities` chiama `create_pseudodev` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve creare una pseudo-interfaccia sopra una interfaccia reale e spostarla su un dato
network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire alcuni comandi:  
Crea la pseudo-interfaccia con "`ip link add dev ... link ...`".  
Poi recupera (o imposta nel caso di una testsuite) il suo indirizzo MAC.  
Poi sposta la pseudo-interfaccia nel namespace con "`ip link set dev ... netns ...`".  
Poi esegue alcuni comandi "`ip netns exec ... sysctl ...`".  
Poi attiva la pseudo-interfaccia con "`ip netns exec ... ip link set dev ... up`".  

Quando il modulo `Identities` chiama `add_address` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve assegnare un indirizzo IP linklocal ad una interfaccia di rete.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip address add ... dev ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.  

Quando il modulo `Identities` chiama `add_gateway` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve assegnare una rotta diretta da una interfaccia di rete del nodo corrente ad una
data interfaccia di un diretto vicino.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip route add ... dev ... src ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.  

Quando il modulo `Identities` chiama `remove_gateway` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve rimuovere una rotta diretta da una interfaccia di rete del nodo corrente ad una
data interfaccia di un diretto vicino.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip route del ... dev ... src ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.  

Le richieste che seguono vengono fatte dal modulo `Identities` di norma nella sequenza
`flush_table`, `delete_pseudodev` e `delete_namespace`.

Quando il modulo `Identities` chiama `flush_table` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve azzerare le tabelle di routing su un dato network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip netns exec ... ip route flush table main`".

Quando il modulo `Identities` chiama `delete_pseudodev` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve eliminare una pseudo-interfaccia di rete che era stata messa su un
dato network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip netns exec ... ip link delete ...`".

Quando il modulo `Identities` chiama `delete_namespace` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve eliminare un network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire il comando "`ip netns del ...`".

* * *

Per integrare il modulo `Identities` occorre implementare con una classe l'interfaccia `N.I.IIdmgmtStubFactory`.  
Vedere la classe `N.IdmgmtStubFactory` nel file `identities_helpers.vala`.

Quando il modulo `Identities` chiama `get_arc` di `N.I.IIdmgmtStubFactory` passando una istanza
di `CallerInfo`, la classe suddetta deve stabilire se il metodo remoto è stato chiamato da un diretto
vicino in modalità unicast;
e in quel caso deve restituire l'arco `N.I.IIdmgmtArc` da cui il messaggio è
stato ricevuto.  
Lo fa passando il `CallerInfo` al metodo `from_caller_get_nodearc` di `skeleton_factory`.

Quando il modulo `Identities` chiama `get_stub` di `N.I.IIdmgmtStubFactory` passando una istanza
di `N.I.IIdmgmtArc`, la classe suddetta deve deve restituire uno stub per mandare messaggi unicast
allo stesso modulo `Identities` attraverso quell'arco.  
Lo fa chiamando sulla classe `N.StubFactory` il metodo `get_stub_whole_node_unicast` che gli
ottiene una istanza di stub per messaggi unicast di nodo. Lo stub ottenuto è radice. Per ottenere uno stub
dedicato al modulo `Identities` la classe istanzia un `IdentityManagerStubHolder`.

* * *

Per integrare il modulo `Identities` occorre implementare con una classe l'interfaccia `N.I.IIdmgmtArc`;
una istanza di questa classe deve rappresentare un arco tra due nodi diretti vicini.  
Vedere la classe `N.IdmgmtArc` nel file `identities_helpers.vala`.

Un'istanza di questa classe viene costruita passando l'istanza di `N.N.INeighborhoodArc`, che il
modulo `Neighborhood` produce quando rileva l'arco. Ricordiamo che esso rappresenta un arco tra una
spcifica interfaccia di rete di questo nodo e una specifica interfaccia di rete di un altro nodo
diretto vicino.

Il modulo `Identities` può interrogare l'interfaccia `N.I.IIdmgmtArc` con i metodi
`get_dev`, `get_peer_linklocal` e `get_peer_mac`, che la classe `N.IdmgmtArc` può facilmente
reperire dall'istanza di `N.N.INeighborhoodArc` ad essa associata.  
Inoltre ad ogni istanza di `N.IdmgmtArc` il costruttore associa un identificativo intero `id`.
**TODO** a che serve.

* * *

Per reagire ai segnali emessi dal modulo `Identities` sono implementate delle funzioni
nel file `identities_signals.vala`.

... **TODO**  

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

La classe `N.IdentityManagerStubHolder` implementa l'interfaccia `N.IIdentityManagerStub`
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
usando i metodi `get_dispatcher` e `get_dispatcher_set` di `N.SkeletonFactory` passando il CallerInfo.

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

Il metodo `from_caller_get_nodearc` è usato (dice il commento) per richieste di nodo, per sapere da quale
arco una tale richiesta è arrivata:

*   Se il caller è un `StreamCallerInfo`, da esso reperisce il `source_id`;
*   Se questo è un `WholeNodeSourceID`, da esso reperisce il `peer_node_id`;
*   Dal caller reperisce il `listener` che deve essere un `StreamSystemListener`: da questo
    reperisce il `listen_pathname`;
*   Dal caller reperisce il `src_nic` che deve essere un `NeighbourSrcNic`: da questo
    reperisce il `neighbour_mac`;
*   Cerca in `arc_list` l'istanza di `N.NodeArc` associata al `neighborhood_arc` che
    soddisfa questi 3 criteri (`listen_pathname`, `neighbour_mac`, `peer_node_id`) e
    se la trova restituisce questo `N.NodeArc`.
*   In tutti gli altri casi restituisce *null*.

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
Si può passare una lista di compiti (tasks) da fare in un dato momento.  

Il PID finisce nella variabile globale `int pid` nel file `system_ntkd.vala`.  
La lista di interfacce finisce in `ArrayList<string> devs` che è locale nella funzione `main`.  
La lista di interfacce finisce in `ArrayList<string> tasks` che è locale nella funzione `main`.  

Dopo si inizializza lo scheduler delle tasklet. Va nella variabile globale `ITasklet tasklet`.

Dopo si inizializza un FakeCommandDispatcher. Va nella variabile globale `FakeCommandDispatcher fake_cm`.

Dopo si inizializzano i singoli moduli (principalmente perché hanno delle classi serializzabili) e si
registrano le classi serializzabili che servono.  
I moduli si inizializzano con il metodo statico `init` delle classi  `NeighborhoodManager`,
`IdentityManager`, ...  
Le classi serializzabili da registrare direttamente in `main` sono `WholeNodeSourceID`, `WholeNodeUnicastID`,
`EveryWholeNodeBroadcastID`, `NeighbourSrcNic`, ...

Dopo si inizializza il generatore di numeri pseudo-casuali. Si usa come seed il PID.  
Questa operazione si fa con il metodo statico `init_rngen` delle classi `PRNGen`, `NeighborhoodManager`,
`IdentityManager`, ...  

Dopo si passa lo scheduler delle tasklet alla libreria `ntkdrpc`, chiamandone la funzione statica `init_tasklet_system`.

Dopo si istanzia lo `skeleton_factory`.

Dopo si istanzia lo `stub_factory`.

Dopo si istanzia il NeighborhoodManager nella variabile globale `neighborhood_mgr`. Esso è unico.

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

Alla terminazione del programma, si chiama `stop_rpc` su tutte le pseudo-interfacce di rete gestite.

Dopo si rimuove il NeighborhoodManager (dalla variabile globale).

Dopo si termina lo scheduler delle tasklet.

* * *

Le funzioni `fake_random_mac` e `fake_random_linklocal` sono usate per... ? **TODO**

* * *

* * *

Dopo il completamento delle parti necessarie all'integrazione del modulo Neighborhood viene realizzata
la script `test_1_neighborhood`.

Sono simulati i nodi 123, 456 e 789. Ognuno con eth0 e wlan0.  
Un solo segmento di rete ethernet collega tutte le tre eth0.  
Invece le radio collegano 123 con 456 e 456 con 789.  
Tutto ciò è simulato con:

```
eth_domain   -i 123_eth0  -i 456_eth0  -i 789_eth0
radio_domain -i 123_wlan0 -o 456_wlan0
radio_domain -o 123_wlan0 -i 456_wlan0 -o 789_wlan0
radio_domain              -o 456_wlan0 -i 789_wlan0
```


