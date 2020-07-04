# Classi helper modulo Hooking

## Interfaccia IHookingMapPaths

L'interfaccia `N.H.IHookingMapPaths` è implementata nella classe
`N.HookingMapPaths`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

### Network ID

Quando il modulo `Hooking` chiama `get_network_id` la classe suddetta deve
restituire il `network_id` della rete di cui fa parte il nodo. Questa
informazione il programma la recupera per mezzo del QspnManager associato
all'identità.  
In questa testsuite ogni nodo sarà da solo e avrà generato una rete a
sé; il modulo `Qspn` del nodo avrà prodotto il suo `network_id`.

### Network Topology

Quando il modulo `Hooking` chiama `get_levels` la classe suddetta deve
restituire il numero di livelli della rete di cui fa parte il nodo.  
Questa informazione il programma la prende dalla variabile globale `levels`.

Quando il modulo `Hooking` chiama `get_gsize` la classe suddetta deve
restituire il numero di posizioni *reali* dentro il livello passato
alla chiamata, nella rete di cui fa parte il nodo.  
Questa informazione il programma la prende dalla variabile globale `gsizes`.

Quando il modulo `Hooking` chiama `get_epsilon` la classe suddetta deve
restituire il numero di livelli che si possono accettare come differenza
nel livello cercato in una *migration path*. Va calcolato sulla base
delle dimensioni dei vari livelli, a partire dal livello passato
alla chiamata, nella rete di cui fa parte il nodo.  
Questa informazione il programma la prende dalla variabile
globale `hooking_epsilon`, la quale viene inizializzata dal codice sulla
base delle dimensioni della topologia, in modo tale che si raggiunga
una differenza di 5 bit.

### Network Size

Quando il modulo `Hooking` chiama `get_n_nodes` la classe suddetta deve
restituire una stima della dimensione in nodi di tutta la rete.  
Questa informazione il programma la recupera per mezzo del QspnManager associato
all'identità.  
In questa testsuite il modulo `Qspn` dovrebbe sapere che è 1.

### My Position

Quando il modulo `Hooking` chiama `get_my_pos` la classe suddetta deve
restituire la posizione del nodo al livello passato
alla chiamata, nella rete di cui fa parte il nodo.  
Questa informazione il programma la prende dalla variabile globale `my_naddr`.  
In questa testsuite il nodo appena nasce forma una rete a sé, quindi
la sua posizione è scelta da esso stesso in modo arbitrario e passata
al costruttore `create_net` del QspnManager. In future implementazioni, quando il
nodo entrerà in una rete esistente conoscerà la posizione che dovrà andare a
ricoprire e la comunicherà al costruttore `enter` del QspnManager. Oltre a comunicare
questa informazione al QspnManager, la salva in `my_naddr`.

Quando il modulo `Hooking` chiama `get_my_eldership` la classe suddetta deve
restituire la anzianità del nodo (o g-nodo) al livello passato
alla chiamata, nel g-nodo direttamente superiore.  
Questa informazione il programma la prende dalla variabile globale `my_fp`.  
In questa testsuite il nodo appena nasce forma una rete a sé, quindi
la sua anzianità è 0; questa informazione viene passata al QspnManager e
viene salvata in `my_fp`.

Quando il modulo `Hooking` chiama `get_subnetlevel` la classe suddetta deve
restituire il numero di livelli per cui questo sistema gestisce una
sottorete autonoma.  
Questa informazione il programma la prende dalla variabile globale `subnetlevel`.

### My Map

Quando il modulo `Hooking` chiama `exists` la classe suddetta deve
dire se esiste un certo g-nodo nella mappa mantenuta dal QspnManager.

Quando il modulo `Hooking` chiama `get_eldership` la classe suddetta deve
restituire l'anzianità di un certo g-nodo nella mappa mantenuta dal QspnManager.  
Si assume che è stato prima verificato che il g-nodo esiste nella mappa.

Il modulo `Hooking` chiama `adjacent_to_my_gnode` durante la ricerca distribuita
di una *migration path*. Il nodo *w* è chiamato a individuare i g-nodi di livello
`l` o superiore adiacenti al proprio g-nodo di livello `l`. Scompone questo problema
in diverse chiamate `adjacent_to_my_gnode(level_adjacent_gnodes, level_my_gnode)`
in ognuna delle quali questo metodo (implementato dalla classe suddetta) deve
restituire una lista di oggetti dai quali si possono leggere 2 informazioni: la
prima è un g-nodo (in coordinate gerarchiche rispetto a *w*) di livello
`level_adjacent_gnodes` adiacente al g-nodo di livello `level_my_gnode` di *w*, e
la seconda è la posizione (*reale*) del border-g-nodo
di livello `level_my_gnode - 1` del g-nodo di livello `level_my_gnode` di *w*.

### My Gateways

Quando il modulo `Hooking` chiama `gateway` la classe suddetta deve
restituire uno stub per comunicare (si intende una comunicazione RPC tra
moduli `Hooking`) con il favorito gateway verso un certo g-nodo.  
Si assume che è stato prima verificato che il g-nodo esiste nella mappa.  
In questa testsuite il metodo non verrà mai chiamato. L'implementazione è stata
riportata come commento.

## Interfaccia ICoordinator

L'interfaccia `N.H.ICoordinator` è implementata nella classe
`N.HookingCoordinator`.

Una istanza di questa classe è collegata a una identità assunta in questo nodo.
Nella costruzione di questa istanza viene passato l'identificativo
`int local_identity_index`; questa classe ha un getter per risalire alla istanza di
`IdentityData` che se viene invocato su una identità che non è più nella hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Questa classe sarà usata per realizzare le chiamate...

Riguarda la collaborazione del modulo `Coordinator` ai fini della
realizzazione delle procedure che servono al modulo `Hooking`.

### Collaborazioni A

Il modulo Hooking in esecuzione in un nodo generico *x*
vuole far eseguire alcuni suoi metodi nel nodo *y* che è il
Coordinator di tutta la rete o di un g-nodo.

Il modulo `Hooking` nel nodo *x* chiama `evaluate_enter` della classe che implementa
l'interfaccia `N.H.ICoordinator`.  
Questa esegue `coord_mgr.evaluate_enter`. Il modulo `Coordinator` fa in modo che il messaggio
giunga al nodo *y*. Nel nodo *y* il modulo `Coordinator` chiama `evaluate_enter` della classe
che implementa l'interfaccia `N.C.IEvaluateEnterHandler`.  
Questa esegue `hook_mgr.evaluate_enter`. Il modulo `Hooking` nel nodo *y* fa
il suo lavoro e restituisce il risultato al metodo `evaluate_enter` della classe
che implementa l'interfaccia `N.C.IEvaluateEnterHandler`.  
Nel nodo *y* il modulo `Coordinator`, ottenuta questa risposta la fa arrivare come
messaggio al nodo *x*.  
Nel nodo *x* la precedente chiamata a `coord_mgr.evaluate_enter` restituisce una
risposta al metodo `evaluate_enter` della classe che implementa
l'interfaccia `N.H.ICoordinator`. Il quale la restituisce al modulo `Hooking`.

In conclusione, quando il modulo `Hooking` chiama `evaluate_enter` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.evaluate_enter`.  
Un eventuale errore nelle operazioni di comunicazione è segnalato con il lancio dell'eccezione
`N.H.CoordProxyError`.  
In realtà in questa testsuite non sarà mai eseguito questo metodo.

Similmente, quando il modulo `Hooking` chiama `begin_enter` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.begin_enter`.

Similmente, quando il modulo `Hooking` chiama `completed_enter` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.completed_enter`.

Similmente, quando il modulo `Hooking` chiama `abort_enter` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.abort_enter`.

### Collaborazioni B

Il modulo Hooking in esecuzione nel nodo *y* che è il
Coordinator di tutta la rete o di un g-nodo vuole accedere in lettura/scrittura alla memoria
condivisa (di tutta la rete o di un g-nodo) per salvare e recuperare alcune informazioni
di sua pertinenza.

Il modulo `Hooking` nel nodo *y* chiama `get_hooking_memory` della classe che implementa
l'interfaccia `N.H.ICoordinator`.  
Questa esegue `coord_mgr.get_hooking_memory`. Il modulo `Coordinator` fa in modo di recuperare
la sua memoria condivisa, cioè il valore nel database distribuito (DHT) associato alla chiave
che rappresente il livello del suo g-nodo di pertinenza; in esso c'è un dato che costituisce
la memoria condivisa di pertinenza del modulo `Hooking`. In altre parole, il modulo
`Coordinator` si occupa di reperire questa informazione e la restituisce al metodo
`get_hooking_memory` della classe che implementa l'interfaccia `N.H.ICoordinator`. Il quale la
restituisce al modulo `Hooking`.

In conclusione, quando il modulo `Hooking` chiama `get_hooking_memory` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.get_hooking_memory`.  
Un eventuale errore nelle operazioni di comunicazione è segnalato con il lancio dell'eccezione
`N.H.CoordProxyError`.  
In realtà in questa testsuite non sarà mai eseguito questo metodo.

Similmente, quando il modulo `Hooking` chiama `set_hooking_memory` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.set_hooking_memory`.

### Collaborazioni C

Il modulo Hooking in esecuzione in un nodo generico *x*
vuole far eseguire alcuni suoi metodi in tutti i singoli nodi del proprio
g-nodo di un determinato livello.  
Vuole inoltre attendere il completamento di tutte le operazioni nei singoli nodi
che hanno ricevuto il messaggio. Con eventuale ricezione di un esito
cumulativo.

Il modulo `Hooking` nel nodo *x* chiama `prepare_enter` della classe che implementa
l'interfaccia `N.H.ICoordinator`.  
Questa esegue `coord_mgr.prepare_enter`. Il modulo `Coordinator` fa in modo che un messaggio
di `void execute_prepare_enter` giunga a tutti i nodi del suo g-nodo e
venga eseguito una e una sola volta su ogni nodo. In questo caso si tratta di un messaggio
che ha esito `void`, quindi la sola risposta che riceve è il completamento dell'esecuzione.
Ogni singolo nodo, quando riceve il messaggio e se intende eseguirlo (cioè se si riconosce
destinatario e non lo ha già eseguito), chiama il metodo `prepare_enter` della classe
che implementa l'interfaccia `N.C.IPropagationHandler`.  
Questa esegue `hook_mgr.prepare_enter`. Il modulo `Hooking` nel singolo nodo destinatario
fa il suo lavoro e restituisce il risultato al metodo `prepare_enter` della classe
che implementa l'interfaccia `N.C.IPropagationHandler`.  
Questa la restituisce al modulo `Coordinator` il quale la fa giungere come risposta
in conclusione al modulo `Coordinator` del nodo *x*.  
Infine il modulo `Coordinator` del nodo *x* risponde al metodo `prepare_enter` della
classe che implementa l'interfaccia `N.H.ICoordinator` e questa risponde al modulo
`Hooking`.

Similmente, quando il modulo `Hooking` chiama `prepare_migration` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.prepare_migration`.

### Collaborazioni D

Il modulo Hooking in esecuzione in un nodo generico *x*
vuole far eseguire alcuni suoi metodi in tutti i singoli nodi del proprio
g-nodo di un determinato livello.  
Non ha interesse stavolta di attendere l'esecuzione delle operazioni nei singoli
nodi.

Il modulo `Hooking` nel nodo *x* chiama `finish_enter` della classe che implementa
l'interfaccia `N.H.ICoordinator`.  
Questa esegue `coord_mgr.finish_enter`. Il modulo `Coordinator` fa in modo che un messaggio
di `void execute_finish_enter` parta per giungere a tutti i nodi del suo g-nodo e
venire eseguito una e una sola volta su ogni nodo. Ma non attende il risultato di
questo messaggio.  
Ogni singolo nodo, quando riceve il messaggio e se intende eseguirlo (cioè se si riconosce
destinatario e non lo ha già eseguito), chiama il metodo `finish_enter` della classe
che implementa l'interfaccia `N.C.IPropagationHandler`.  
Questa esegue `hook_mgr.finish_enter`. Il modulo `Hooking` nel singolo nodo destinatario
fa il suo lavoro. Ma indipendentemente da questo, il modulo `Coordinator` del nodo *x*
risponde immediatamente al metodo `finish_enter` della
classe che implementa l'interfaccia `N.H.ICoordinator` e questa risponde al modulo
`Hooking`.

Similmente, quando il modulo `Hooking` chiama `finish_migration` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.finish_migration`.

Similmente, quando il modulo `Hooking` chiama `we_have_splitted` la classe suddetta deve
restituire quanto ritornato dalla chiamata `coord_mgr.we_have_splitted`.

### Collaborazioni E

Una importante funzione del modulo Coordinator è quella di riservare un posto in un g-nodo. Anche
questa si può vedere come una fondamentale collaborazione del modulo Coordinator per le esigenze
del modulo Hooking.

Un altro caso di collaborazione richiesta dal modulo Hooking è un metodo che il modulo Coordinator
mette a disposizione per chiedere al nodo Coordinator della rete il numero di singoli nodi in essa.

Si vedranno nella trattazione i metodi `reserve`, `get_n_nodes`.




```
  .      public abstract int get_n_nodes();

  V      public abstract Object evaluate_enter(Object evaluate_enter_data) throws CoordProxyError;

  V      public abstract Object? get_hooking_memory(int lvl) throws CoordProxyError;
  V      public abstract void set_hooking_memory(int lvl, Object memory) throws CoordProxyError;

  V      public abstract Object begin_enter(int lvl, Object begin_enter_data) throws CoordProxyError;
  V      public abstract Object completed_enter(int lvl, Object completed_enter_data) throws CoordProxyError;
  V      public abstract Object abort_enter(int lvl, Object abort_enter_data) throws CoordProxyError;

  v      public abstract void prepare_enter(int lvl, Object prepare_enter_data);
  v      public abstract void finish_enter(int lvl, Object finish_enter_data);

  .      public abstract void reserve(int host_lvl, int reserve_request_id, out int new_pos, out int new_eldership) throws CoordReserveError;
  .      public abstract void delete_reserve(int host_lvl, int reserve_request_id);

  v      public abstract void prepare_migration(int lvl, Object prepare_migration_data);
  v      public abstract void finish_migration(int lvl, Object finish_migration_data);

```
