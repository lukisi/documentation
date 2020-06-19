# Classi helper modulo Neighborhood

L'interfaccia `N.N.INeighborhoodIPRouteManager` è implementata nella classe
`N.NeighborhoodIPRouteManager`.

Quando il modulo `Neighborhood` chiama `add_address` la classe suddetta deve
aggiungere l'indirizzo IP passato (si tratterà di un linklocal) alla interfaccia di
rete (reale) passata.  
Lo fa chiedendo al *commander* di eseguire un singolo comando "`ip address add ...`".

Quando il modulo `Neighborhood` chiama `add_neighbor` la classe suddetta deve
aggiungere una route diretta all'indirizzo IP passato (si tratterà di un linklocal)
tramite l'interfaccia di rete (reale) passata.  
Lo fa chiedendo al *commander* di eseguire un singolo comando "`ip route add ...`".

Quando il modulo `Neighborhood` chiama `remove_neighbor` la classe suddetta deve
rimuovere una route diretta all'indirizzo IP passato (si tratterà di un linklocal)
tramite l'interfaccia di rete (reale) passata.  
Lo fa chiedendo al *commander* di eseguire un singolo comando "`ip route del ...`".

Quando il modulo `Neighborhood` chiama `remove_address` la classe suddetta deve
rimuovere l'indirizzo IP passato (si tratterà di un linklocal) alla interfaccia di
rete (reale) passata.  
Lo fa chiedendo al *commander* di eseguire un singolo comando "`ip address del ...`".

* * *

L'interfaccia `N.N.INeighborhoodStubFactory` è implementata nella classe
`N.NeighborhoodStubFactory`.

Quando il modulo `Neighborhood` chiama `get_broadcast_for_radar` la classe suddetta deve
restituire uno stub per mandare messaggi broadcast allo stesso modulo `Neighborhood` nei
nodi diretti vicini attraverso una specifica interfaccia di rete.  
Lo fa chiamando sulla classe `N.StubFactory` il metodo
`get_stub_whole_node_broadcast_for_radar` che gli ottiene una istanza di stub per
messaggi broadcast di nodo. Lo stub ottenuto è radice (si veda `AddressManager addr`
nel file `interfaces.rpcidl` del pacchetto `ntkdrpc`). Per ottenere uno stub dedicato
al modulo `Neighborhood` la classe crea una istanza di `NeighborhoodManagerStubHolder`.

Quando il modulo `Neighborhood` chiama `get_unicast` la classe suddetta deve
restituire uno stub per mandare messaggi unicast allo stesso modulo `Neighborhood`
in uno specifico nodo diretto vicino attraverso uno specifico
arco `N.N.INeighborhoodArc`.  
Questo non si verifica mai in questa testsuite, quindi il codice semplicemente
va in errore.

* * *

L'interfaccia `N.N.INeighborhoodQueryCallerInfo` è implementata nella classe
`N.NeighborhoodQueryCallerInfo`.

Quando il modulo `Neighborhood` chiama `is_from_broadcast` la classe suddetta deve
esaminare una istanza di `CallerInfo` (passata da ZCD ai metodi remoti quando riceve
una chiamata da un altro nodo) e stabilire se il metodo remoto è stato chiamato da un
diretto vicino in modalità broadcast; e in quel caso deve restituire l'istanza di
`N.N.INeighborhoodNetworkInterface` associata all'interfaccia di rete da cui il
messaggio è stato ricevuto.  
Lo fa chiamando sulla classe `N.SkeletonFactory` il metodo `from_caller_get_mydev`
che gli restituisce la stringa con il nome dell'interfaccia da cui è stato ricevuto
il messaggio. Tramite questo nome e l'hashmap `pseudonic_map` risale all'istanza
di `N.PseudoNetworkInterface` e in essa trova l'istanza
di `N.N.INeighborhoodNetworkInterface` che deve restituire.

Quando il modulo `Neighborhood` chiama `is_from_unicast` la classe suddetta deve
esaminare una istanza di `CallerInfo` e una lista di archi `N.N.INeighborhoodArc`
che gli sono passati e stabilire se il metodo remoto è stato chiamato da un diretto
vicino in modalità unicast; e in quel caso deve restituire l'arco
`N.N.INeighborhoodArc` da cui il messaggio è stato ricevuto.  
Il nodo non crea nessuna arco in questa testsuite, quindi il codice semplicemente
restituisce *null*.

* * *

L'interfaccia `N.N.INeighborhoodNetworkInterface` è implementata nella classe
`N.NeighborhoodNetworkInterface`.

Le istanze di questa classe sono da passare al modulo `Neighborhood` a rappresentare
una interfaccia di rete da gestire. Una istanza è creata passando una istanza di
`N.PseudoNetworkInterface`. A sua volta una istanza di `N.PseudoNetworkInterface`
contiene un riferimento (**TODO** mettere weak) alla relativa istanza
di `N.N.INeighborhoodNetworkInterface`.

Quando il modulo `Neighborhood` chiama `measure_rtt` la classe suddetta deve misurare
il Round Trip Time delle comunicazioni tramite questa interfaccia di rete con un
indirizzo IP linklocal che identifica una precisa interfaccia di rete di un nodo
diretto vicino.  
Questo avverrà eseguendo il comando `ping` e interpretandone l'output. Per il momento
invece, questo programma di test chiede al *commander* di eseguire un singolo
comando "`ping`" e restituisce una misurazione fake di 1000 usec.
