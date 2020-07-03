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
...

### Network Topology

Quando il modulo `Hooking` chiama `get_levels` la classe suddetta deve
...

Quando il modulo `Hooking` chiama `get_gsize` la classe suddetta deve
...

Quando il modulo `Hooking` chiama `get_epsilon` la classe suddetta deve
...

### Network Size

Quando il modulo `Hooking` chiama `get_n_nodes` la classe suddetta deve
...

### My Position

Quando il modulo `Hooking` chiama `get_my_pos` la classe suddetta deve
...

Quando il modulo `Hooking` chiama `get_my_eldership` la classe suddetta deve
...

Quando il modulo `Hooking` chiama `get_subnetlevel` la classe suddetta deve
...

### My Map

Quando il modulo `Hooking` chiama `exists` la classe suddetta deve
...

Quando il modulo `Hooking` chiama `get_eldership` la classe suddetta deve
...

Quando il modulo `Hooking` chiama `adjacent_to_my_gnode` la classe suddetta deve
...

### My Gateways

Quando il modulo `Hooking` chiama `gateway` la classe suddetta deve
...

## Interfaccia IHookingMapPaths

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

* * *

ICoordinator