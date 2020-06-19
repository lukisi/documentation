# Classi helper modulo Qspn

L'interfaccia `N.Q.IQspnStubFactory` è implementata nella classe
`N.QspnStubFactory`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Quando il modulo `Qspn` chiama `i_qspn_get_broadcast` gli passa una lista di archi
`List<IQspnArc> arcs`, e opzionalmente una istanza di `IQspnMissingArcHandler` per
gestire gli archi che non comunicano l'avvenuta ricezione del messaggio
broadcast.  
La classe suddetta deve restituire uno stub per comunicare in broadcast con vari
archi. In questa testsuite non si realizzano archi, perciò il codice va semplicemente
in errore se la lista che riceve come argomento contiene qualche arco. Se invece la
lista è vuota restituisce semplicemente una nuova istanza di `QspnManagerStubVoid`.

Quando il modulo `Qspn` chiama `i_qspn_get_tcp` di `N.Q.IQspnStubFactory` gli passa
un arco `IQspnArc arc`, e un booleano `wait_reply`.  
In questa testsuite non si realizzano archi, perciò il codice va semplicemente
in errore.

* * *

L'interfaccia `N.Q.IQspnThresholdCalculator` è implementata nella classe
`N.ThresholdCalculator`.

Quando il modulo `Qspn` chiama `i_qspn_calculate_threshold` gli passa
due istanze di `IQspnNodePath`. Questi percorsi calcolati dal modulo Qspn hanno
ognuno un *costo* che la classe suddetta sa tradurre in microsecondi. La classe
moltiplica la somma dei costi per 50 e la converte in millisecondi. Restituisce
questo valore.

* * *

L'interfaccia `N.Q.IQspnArc` è implementata nella classe `N.QspnArc`.  
Ne viene realizzato lo scheletro, sebbene in questa testsuite non
si realizzano archi, perciò il codice non ne farà mai uso.

Una istanza di questa classe rappresenta un arco-identità. Viene costruita
passando una istanza di `IdentityArc`, il NodeID sorgente (la nostra identità)
e quello destinazione.  
Dalla istanza di `IdentityArc` si accede alla istanza di `N.N.INeighborhoodArc`
prodotta dal modulo Neighborhood che rappresenta l'arco-nodo; attraverso la quale si
può accedere al relativo costo.  
Nel costruttore di `N.QspnArc` si sceglie un intero casuale da 0 a 1000 e lo si
memorizza. Questo viene aggiunto al costo dell'arco-nodo quando viene
richiesto il costo di questo arco-identità. Questa piccola (insignificante)
variazione renda molto improbabile ottenere due path distinti con
costo identico.

Quando il modulo `Qspn` chiama `i_qspn_get_cost` la classe suddetta deve
restituire una istanza di `N.Cost` col valore ottenuto come detto prima.  
In realtà l'implementazione in questa testsuite va semplicemente in errore.

Quando il modulo `Qspn` chiama `i_qspn_equals` la classe suddetta deve
restituire *TRUE* solo se si tratta di due `N.QspnArc` costruite sulla stessa
istanza di `IdentityArc`.  
In realtà l'implementazione in questa testsuite va semplicemente in errore.

Quando il modulo `Qspn` chiama `i_qspn_comes_from` gli passa una istanza di
`CallerInfo`. La classe suddetta usa un metodo di `skeleton_factory` per
vedere se si tratta della stessa istanza di `IdentityArc`.  
In realtà l'implementazione in questa testsuite va semplicemente in errore.
