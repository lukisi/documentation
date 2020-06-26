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

Quando il modulo `Coordinator` chiama `xxx` la classe suddetta deve

* * *

L'interfaccia `N.C.IStubFactory` è implementata nella classe
`N.CoordinatorStubFactory`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Quando il modulo `Coordinator` chiama `xxx` la classe suddetta deve

* * *

L'interfaccia `N.C.ICoordinatorMap` è implementata nella classe
`N.CoordinatorMap`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Quando il modulo `Coordinator` chiama `xxx` la classe suddetta deve

