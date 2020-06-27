# Classi helper modulo Coordinator

L'interfaccia `N.C.IEvaluateEnterHandler` è implementata nella classe
`N.CoordinatorEvaluateEnterHandler`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Il modulo `Coordinator` chiama `evaluate_enter` quando il nostro nodo riceve
una richiesta P2P (attraverso il PeerServices) per cui, in qualità di nodo
Coordinator, deve *valutare l'ingresso* in una diversa rete.  
Perciò, quando il modulo `Coordinator` chiama `evaluate_enter` la classe suddetta
dovrà chiamare il metodo `evaluate_enter` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

* * *

L'interfaccia `N.C.IBeginEnterHandler` è implementata nella classe
`N.CoordinatorBeginEnterHandler`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Il modulo `Coordinator` chiama `begin_enter` quando il nostro nodo riceve
una richiesta P2P per cui, in qualità di nodo
Coordinator, deve *iniziare l'ingresso* in una diversa rete.  
Perciò, quando il modulo `Coordinator` chiama `begin_enter` la classe suddetta
dovrà chiamare il metodo `begin_enter` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

* * *

L'interfaccia `N.C.ICompletedEnterHandler` è implementata nella classe
`N.CoordinatorCompletedEnterHandler`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Il modulo `Coordinator` chiama `completed_enter` quando il nostro nodo riceve
una richiesta P2P per cui, in qualità di nodo
Coordinator, deve *completare l'ingresso* in una diversa rete.  
Perciò, quando il modulo `Coordinator` chiama `completed_enter` la classe suddetta
dovrà chiamare il metodo `completed_enter` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

Quando il modulo `Coordinator` chiama `completed_enter` la classe suddetta deve

* * *

L'interfaccia `N.C.IAbortEnterHandler` è implementata nella classe
`N.CoordinatorAbortEnterHandler`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Il modulo `Coordinator` chiama `abort_enter` quando il nostro nodo riceve
una richiesta P2P per cui, in qualità di nodo
Coordinator, deve *annullare l'ingresso* in una diversa rete.  
Perciò, quando il modulo `Coordinator` chiama `abort_enter` la classe suddetta
dovrà chiamare il metodo `abort_enter` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

* * *

L'interfaccia `N.C.IPropagationHandler` è implementata nella classe
`N.CoordinatorPropagationHandler`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Il modulo `Coordinator` chiama `prepare_migration` (oppure `finish_migration`
o uno degli altri metodi) di questo *"delegato"* durante le operazioni che
realizzano una *propagazione* (con o senza ritorno) in tutti i singoli
nodi di un certo g-nodo. Vedi [analisi](
../../ModuloCoordinator/AnalisiFunzionale.md#esecuzioni-su-tutti-i-nodi-di-un-g-nodo).

Perciò, quando il modulo `Coordinator` chiama `prepare_migration` la classe suddetta
dovrà chiamare il metodo `prepare_migration` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

Similmente, quando il modulo `Coordinator` chiama `finish_migration` la classe
suddetta dovrà chiamare il metodo `finish_migration` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

Similmente, quando il modulo `Coordinator` chiama `prepare_enter` la classe
suddetta dovrà chiamare il metodo `prepare_enter` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

Similmente, quando il modulo `Coordinator` chiama `finish_enter` la classe
suddetta dovrà chiamare il metodo `finish_enter` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

Similmente, quando il modulo `Coordinator` chiama `we_have_splitted` la classe
suddetta dovrà chiamare il metodo `we_have_splitted` del modulo Hooking.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

* * *

L'interfaccia `N.C.IStubFactory` è implementata nella classe
`N.CoordinatorStubFactory`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Si usano i suoi metodi per ottenere oggetti stub per comunicare con i diretti
vicini, tramite un messaggio unicast che attende una risposta o tramite
un messaggio broadcast che non attende risposta.

Quando il modulo `Coordinator` chiama `get_stub_for_all_neighbors` la classe
suddetta deve restituire uno stub adatto a comunicare con i propri vicini
tramite un messaggio broadcast che non attende risposta.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

Quando il modulo `Coordinator` chiama `get_stub_for_each_neighbor` la classe
suddetta deve restituire una lista di stub adatti a comunicare con i propri
vicini, uno alla volta, tramite un messaggio unicast che attende una risposta.  
In questa testsuite non sarà mai usata, perciò il codice va semplicemente
in errore.

* * *

L'interfaccia `N.C.ICoordinatorMap` è implementata nella classe
`N.CoordinatorMap`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Quando il modulo `Coordinator` chiama `get_n_nodes` la classe
suddetta deve restituire il numero di nodi (approssimativo) nell'intera
rete. Lo fa chiedendo al modulo `Qspn`.

Quando il modulo `Coordinator` chiama `can_reserve(lvl)` la classe
suddetta deve indicare se è possibile riservare un posto (reale o virtuale)
al livello *lvl*. Sarà impossibile se siamo in effetti un nodo che fa da
gateway per un g-nodo di livello maggiore di *lvl*. Lo sappiamo guardando
la variabile globale `subnetlevel`.

Quando il modulo `Coordinator` chiama `get_free_pos(lvl)` la classe
suddetta deve restituire l'elenco delle posizioni reali libere
nel livello *lvl* (nel g-nodo di livello *lvl+1*); questo stando alle
conoscenze di questo nodo, senza tenere conto delle possibili prenotazioni
pendenti nel nodo Coordinator. Lo fa chiedendo al modulo `Qspn`.

Quando il modulo `Coordinator` chiama `get_my_pos(lvl)` la classe
suddetta deve restituire la posizione assegnata all'identità in questione
nel proprio g-nodo di livello *lvl*.  
Quando il modulo `Coordinator` chiama `get_fp_id(lvl)` la classe
suddetta deve restituire l'identificativo del proprio g-nodo di
livello *lvl*.  
Entrambi questi metodi servono a identificare il proprio g-nodo di
un dato livello, per facilitare la propagazione di un messaggio (con o
senza ritorno). Entrambe le informazioni sono note per mezzo delle
interazioni con il modulo `Qspn`.
