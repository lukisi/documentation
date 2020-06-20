# Classi helper modulo PeerServices

Vedi [analisi](../../ModuloPeers/AnalisiFunzionale.md#classi-e-interfacce).

L'interfaccia `N.P.IPeersMapPaths` è implementata nella classe
`N.PeersMapPaths`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Quando il modulo `PeerServices` chiama `i_peers_get_levels` la classe suddetta deve
restituire il numero di livelli della topologia. Lo trova nella variabile
globale `levels`.

Quando il modulo `PeerServices` chiama `i_peers_get_gsize` la classe suddetta deve
restituire la size di un g-nodo al livello indicato nella chiamata. Lo trova nella
variabile globale `gsizes`.

Quando il modulo `PeerServices` chiama `i_peers_get_my_pos` la classe suddetta deve
restituire l'identificativo del proprio g-nodo al livello indicato nella chiamata.
Lo recupera dall'indirizzo Netsukuku memorizzato nella proprietà `my_naddr`
dell'istanza di `IdentityData`.

Quando il modulo `PeerServices` chiama `i_peers_get_nodes_in_my_group` la classe
suddetta deve restituire il numero di nodi stimato all'interno del proprio g-nodo
del livello indicato nella chiamata. Lo fa chiamando il metodo `get_nodes_inside`
del QspnManager collegato all'istanza di `IdentityData`.

Quando il modulo `PeerServices` chiama `i_peers_exists` la classe suddetta deve
determinare se un certo g-nodo (indicato come coordinate gerarchiche) esiste
nella rete. Lo fa chiamando il metodo `get_known_destinations`
del QspnManager collegato all'istanza di `IdentityData`.

Quando il modulo `PeerServices` chiama `i_peers_gateway` la classe suddetta deve
restituire uno stub per inviare un messaggio al miglior gateway verso un certo
g-nodo, indicato come coordinate gerarchiche.  
A grandi linee, questo sarà implementato facendo uso del metodo `get_paths_to`
del QspnManager collegato all'istanza di `IdentityData`, eventualmente scartando
alcuni gateway sulla base delle richieste indicate nella chiamata; individuato
infine un gateway, cioè un arco-qspn e quindi un arco-identità nel quale
trasmettere dati a quel gateway, si costruisce uno stub.  
In questa testsuite non si realizzano archi, perciò il codice va semplicemente
in errore. Difatti lo stub andava costruito usando il metodo
`get_stub_identity_aware_unicast_from_ia` dello StubFactory, il quale
in questa testsuite non è implementato proprio per quel motivo.  
**TODO** Bisogna andare in errore subito, oppure soltanto dopo aver deliberato
che non sia il caso in cui si lancia l'eccezione `PeersNonexistentDestinationError`?

Quando il modulo `PeerServices` chiama `i_peers_neighbor_at_level` la classe
suddetta deve restituire uno stub per inviare un messaggio a un nodo vicino
(se esiste) che abbia come massimo distinto g-nodo nei miei confronti un g-nodo
di livello *k* indicato nella chiamata.  
In questa testsuite non si realizzano archi, perciò il codice va semplicemente
in errore. Vale lo stesso commento del paragrafo sopra.  
**TODO** Bisogna andare in errore subito, oppure soltanto dopo aver deliberato
che non sia il caso in cui si restituisce *null*?

* * *

L'interfaccia `N.P.IPeersBackStubFactory` è implementata nella classe
`N.PeersBackStubFactory`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Quando il modulo `PeerServices` chiama `i_peers_get_tcp_inside` la classe suddetta
deve restituire uno stub per inviare un messaggio a un dato nodo mediante
connessione TCP interna.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore. Difatti lo stub andava costruito usando il metodo
`get_stub_main_identity_unicast_inside_gnode` dello StubFactory, il quale
in questa testsuite non è implementato ancora.

* * *

L'interfaccia `N.P.IPeersNeighborsFactory` è implementata nella classe
`N.PeersNeighborsFactory`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Quando il modulo `PeerServices` chiama `i_peers_get_broadcast` la classe suddetta
deve restituire uno stub per inviare un messaggio a tutti i vicini appartenenti
alla nostra rete.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore. Difatti lo stub andava costruito usando il metodo
`get_stub_main_identity_unicast_inside_gnode` dello StubFactory, il quale
in questa testsuite non è implementato ancora.

Quando il modulo `PeerServices` chiama `i_peers_get_tcp` la classe suddetta deve
restituire uno stub per inviare un messaggio reliable sull'arco indicato nella
chiamata.  
In questa testsuite non si realizzano archi, perciò il codice va semplicemente
in errore. Difatti lo stub andava costruito usando il metodo
`get_stub_identity_aware_unicast_from_ia` dello StubFactory, il quale
in questa testsuite non è implementato proprio per quel motivo.

* * *

