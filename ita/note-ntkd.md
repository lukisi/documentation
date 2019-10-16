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

Bisogna potersi mettere in ascolto e inviare comunicazioni agli altri nodi:

*   Su una interfaccia di rete in broadcast.  
    Per mettersi in ascolto serve il nome dell'interfaccia di rete. Questo viene passato all'avvio del programma.
*   Su un indirizzo linklocal in unicast.  
    Per mettersi in ascolto serve l'indirizzo IP linklocal assegnato all'interfaccia di rete. Questo
    viene scelto dal `N.N.NeighborhoodManager` che lo notifica con un segnale.
*   Su un indirizzo routabile in unicast.  
    Per mettersi in ascolto serve l'indirizzo IP routabile assegnato al nodo. Questo è calcolato sulla base
    dell'indirizzo Netsukuku del nodo, il quale viene gestito dal modulo `Qspn`.

...

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
una misurazione fake di 1000 msec.

* * *

In una unica classe viene implementato il codice per ottenere degli stub radice di vario tipo (broadcast, unicast,
di nodo o di identità).  
Vedere la classe `N.StubFactory` nel file `rpc/stub_factory.vala`.

**TODO** il metodo `get_stub_whole_node_broadcast_for_radar`

**TODO** il metodo `get_stub_whole_node_unicast`

* * *

Diverse classi (una o più per ogni modulo) realizzano gli stub dedicati.  
Vedere le classi nel file `rpc/module_stubs.vala`.

La classe `N.NeighborhoodManagerStubHolder` implementa l'interfaccia `N.INeighborhoodManagerStub`
a partire da uno stub radice.

* * *

In una unica classe viene implementato il codice per avviare le tasklet in ascolto nelle varie modalità
(stream, datagram, di rete, di sistema, ...).  
Vedere la classe `N.SkeletonFactory` nel file `rpc/skeleton_factory.vala`.

**TODO** ...

* * *

* * *

* * *
