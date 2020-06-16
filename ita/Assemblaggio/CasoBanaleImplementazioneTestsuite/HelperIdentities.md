##### Integrazione modulo Identities

Bisogna integrare l'unica istanza di `N.I.IdentityManager` che si crea all'avio
del programma e muore alla sua terminazione.

Alcune classi dovranno essere prodotte dal programma per implementare le interfacce
definite nel modulo `Identities`. Tali classi sono definite nel file
`identities_helpers.vala`.

I segnali emessi dal modulo `Identities` saranno gestiti da funzioni che sono definite
nel file `identities_signals.vala`.

* * *

L'interfaccia `N.I.IIdmgmtNetnsManager` è implementata nella classe
`N.IdmgmtNetnsManager`.

Quando il modulo `Identities` chiama `create_namespace` la classe suddetta deve
preparare un nuovo network namespace.  
Lo fa chiedendo al *commander* di eseguire i comandi "`ip netns add ...`",
e alcuni comandi "`ip netns exec ... sysctl ...`".

Quando il modulo `Identities` chiama `create_pseudodev` la classe suddetta deve
creare una pseudo-interfaccia sopra una interfaccia reale e spostarla su un dato
network namespace.  
Lo fa chiedendo al *commander* di eseguire alcuni comandi:  
Crea la pseudo-interfaccia con "`ip link add dev ... link ...`".  
Poi recupera (oppure genera e imposta nel caso di una testsuite) il suo
indirizzo MAC.  
Poi sposta la pseudo-interfaccia nel namespace con
"`ip link set dev ... netns ...`".  
Poi esegue alcuni comandi "`ip netns exec ... sysctl ...`".  
Poi attiva la pseudo-interfaccia con
"`ip netns exec ... ip link set dev ... up`".

Quando il modulo `Identities` chiama `add_address` la classe suddetta deve
assegnare un indirizzo IP linklocal a una interfaccia di rete.  
Lo fa chiedendo al *commander* di eseguire il
comando "`ip address add ... dev ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.

Quando il modulo `Identities` chiama `add_gateway` la classe suddetta deve
assegnare una rotta diretta da una interfaccia di rete del nodo corrente verso una
data interfaccia di un diretto vicino.  
Lo fa chiedendo al *commander* di eseguire il
comando "`ip route add ... dev ... src ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.

Quando il modulo `Identities` chiama `remove_gateway` la classe suddetta deve
rimuovere una rotta diretta da una interfaccia di rete del nodo corrente verso una
data interfaccia di un diretto vicino.  
Lo fa chiedendo al *commander* di eseguire il
comando "`ip route del ... dev ... src ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.

Le richieste che seguono vengono fatte dal modulo `Identities` di norma nella sequenza
`flush_table`, `delete_pseudodev` e `delete_namespace`.

Quando il modulo `Identities` chiama `flush_table` la classe suddetta deve
azzerare le tabelle di routing su un dato network namespace.  
Lo fa chiedendo al *commander* di eseguire il
comando "`ip netns exec ... ip route flush table main`".

Quando il modulo `Identities` chiama `delete_pseudodev` la classe suddetta deve
eliminare una pseudo-interfaccia di rete che era stata messa su un
dato network namespace.  
Lo fa chiedendo al *commander* di eseguire il
comando "`ip netns exec ... ip link delete ...`".

Quando il modulo `Identities` chiama `delete_namespace` la classe suddetta deve
eliminare un network namespace.  
Lo fa chiedendo al *commander* di eseguire il comando "`ip netns del ...`".

* * *

L'interfaccia `N.I.IIdmgmtStubFactory` è implementata nella classe
`N.IdmgmtStubFactory`.

Quando il modulo `Identities` chiama `get_arc` passando una istanza di `CallerInfo`,
la classe suddetta deve stabilire se il metodo remoto è stato chiamato da un diretto
vicino in modalità unicast. In quel caso deve restituire l'arco `N.I.IIdmgmtArc` da cui
il messaggio è stato ricevuto.  
Questo non si verifica mai in questa testsuite, quindi il codice semplicemente
restituisce *null*.

Quando il modulo `Identities` chiama `get_stub` passando una istanza
di `N.I.IIdmgmtArc`, la classe suddetta deve deve restituire uno stub per mandare messaggi unicast
allo stesso modulo `Identities` attraverso quell'arco.  
Questo non si verifica mai in questa testsuite, quindi il codice semplicemente
va in errore.

* * *

L'interfaccia `N.I.IIdmgmtArc` è implementata nella classe `N.IdmgmtArc`;
una istanza di questa classe deve rappresentare un arco tra due nodi diretti vicini.

Poiché questa testsuite non prevede la costituzione di archi, questa classe non è
implementata.
