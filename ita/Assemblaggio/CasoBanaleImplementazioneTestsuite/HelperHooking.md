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
In questa testsuite non verrà mai chiamato, ma l'implementazione è stata
fatta. **TODO** manca una parte all'implementazione.

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


```
        public abstract int get_n_nodes();

        // This is going to be proxied to the coordinator of the whole network: lvl=levels
        public abstract Object evaluate_enter(Object evaluate_enter_data) throws CoordProxyError;

        public abstract Object? get_hooking_memory(int lvl) throws CoordProxyError;
        public abstract void set_hooking_memory(int lvl, Object memory) throws CoordProxyError;

        public abstract Object begin_enter(int lvl, Object begin_enter_data) throws CoordProxyError;
        public abstract Object completed_enter(int lvl, Object completed_enter_data) throws CoordProxyError;
        public abstract Object abort_enter(int lvl, Object abort_enter_data) throws CoordProxyError;

        public abstract void prepare_enter(int lvl, Object prepare_enter_data);
        public abstract void finish_enter(int lvl, Object finish_enter_data);

        public abstract void reserve(int host_lvl, int reserve_request_id, out int new_pos, out int new_eldership) throws CoordReserveError;
        public abstract void delete_reserve(int host_lvl, int reserve_request_id);

        public abstract void prepare_migration(int lvl, Object prepare_migration_data);
        public abstract void finish_migration(int lvl, Object finish_migration_data);

```
