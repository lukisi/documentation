# Modulo Hooking - Dettagli Tecnici

1.  [Associazione del modulo ad una id](#associazione-del-modulo-ad-una-id)
    1.  [Operazioni su un proprio arco-id](#operazioni-su-un-proprio-arco-id)
    1.  [Operazioni su richieste da altri nodi](#Operazioni_su_propagazione)
1.  [Requisiti](#Requisiti)
    1.  [Interfaccia IHookingMapPaths](#Requisiti_IHookingMapPaths)
    1.  [Interfaccia ICoordinator](#Requisiti_ICoordinator)
    1.  [Interfaccia IIdentityArc](#Requisiti_IIdentityArc)
    1.  [Segnali](#Requisiti_Segnali)
    1.  [Metodi remoti](#Requisiti_Metodiremoti)

## Associazione del modulo ad una id

Una istanza di `HookingManager` viene costruita ogni volta che si crea una identità nel
sistema. L'istanza di `HookingManager` viene associata a questa identità.

L'utilizzatore del modulo Hooking comunica poi a questa istanza di `HookingManager` la
nascita e la rimozione di ogni arco-identità associato a quella identità nel sistema. Questo
chiamando i metodi pubblici `add_arc` e `remove_arc` dell'istanza di `HookingManager`.

Il modulo Hooking associato ad una identità, per ogni arco-identità che conosce, avvia
una tasklet che ha il compito di dialogare con il diretto vicino su questo arco-identità.
Da questo dialogo scaturiscono alcune operazioni di pertinenza del modulo Hooking che sono
svolte nella stessa tasklet. E da queste operazioni deriva l'emissione di alcuni segnali
che informano l'utilizzatore del modulo su operazioni da intraprendere.

Altre operazioni di pertinenza del modulo Hooking possono essere richieste al modulo
associato ad una identità attraverso meccanismi di propagazione di messaggi all'interno di
un certo g-nodo. Quindi tali operazioni sono svolte non in questa tasklet ma direttamente
all'arrivo del messaggio. Anche da queste operazioni deriva l'emissione di alcuni segnali
che informano l'utilizzatore del modulo su operazioni da intraprendere.

### Operazioni su un proprio arco-id

Il modulo Hooking quando viene aggiunto un arco-identità, sul metodo `add_arc`, avvia una nuova tasklet.
In essa eseguirà tutte le operazioni relative a quell'arco-identità. Quando viene rimosso un arco-identità,
sul metodo `remove_arc`, il modulo abortisce la tasklet relativa.

#### Esame delle identità

La tasklet per prima cosa esamina il tipo di identità su cui è in esecuzione. Se si tratta di una
identità *di connettività* la tasklet termina. Se si tratta di una identità *principale* (quindi
con indirizzo *reale*) allora prosegue. Questa verifica viene fatta all'inizio
e in seguito ogni volta che la tasklet intende chiamare il metodo remoto `retrieve_network_data`.

La tasklet chiama il metodo remoto `retrieve_network_data` sul suo arco-identità. La risposta può essere
l'eccezione `NotPrincipalError` oppure informazioni sulla rete
di appartenenza del vicino.

Se rileva che l'identità vicina è *di connettività*, allora la tasklet termina.

Se l'identità vicina appartiene alla nostra stessa rete, allora la tasklet emette un segnale `same_network` indicando
questo arco-identità. L'utilizzatore gestirà questo segnale memorizzando che quel dato arco-identità congiunge
due nodi della stessa rete e quindi aggiungendo un IQspnArc al modulo QSPN.  
Poi la tasklet termina. Infatti l'identità vicina potrebbe in seguito diventare di un'altra rete, ad
esempio a causa di uno split di g-nodo. Ma in questo caso l'utilizzatore del modulo procederebbe
a rimuovere l'arco-identità e eventualmente a ricostruirne un altro.

Se l'identità vicina appartiene ad una rete diversa che ha una topologia diversa, allora la tasklet termina.  
Le due reti, infatti, non potranno in alcun modo fondersi.

Se l'identità vicina appartiene ad una rete diversa che ha la stessa topologia, allora la tasklet procede
con la valutazione.

#### Valutazione dell'ingresso

Per prima cosa il modulo Hooking emette il segnale `another_network` indicando il `int64 network_id`.

Se la valutazione suggerisce di non fare ingresso nell'altra rete, allora attende un tempo lungo (10 minuti) prima
di ripetere le operazioni di esame dell'identità.

Se invece la valutazione suggerisce di fare ingresso nell'altra rete, allora la tasklet interroga il nodo
Coordinator della sua rete chiamando ripetutamente il metodo `evaluate_enter` del modulo Coordinator
con le modalità descritte nell'analisi, cioè finché riceve l'eccezione `AskAgainError`.

Se infine riceve l'eccezione `IgnoreNetworkError` allora attende un tempo molto lungo (20 volte il
`global_timeout` calcolato sulla base della dimensione della rete) prima di ripetere di nuovo le operazioni
di esame dell'identità.

Se invece alla fine riceve il valore `int first_ask_lvl`, allora la tasklet ha il compito di tentare
l'ingresso del suo g-nodo di quel livello tramite il suo arco-identità.

La tasklet per prima cosa pone `ask_lvl = first_ask_lvl`. Poi inizia le operazioni di esecuzione
dell'ingresso al livello `ask_lvl`.

#### Esecuzione dell'ingresso al livello ask_lvl

La tasklet interroga il nodo Coordinator del suo g-nodo di livello `ask_lvl` chiamando il metodo
`begin_enter` del modulo Coordinator con le modalità descritte nell'analisi.

Se riceve l'eccezione `AlreadyEnteringError` allora attende un tempo molto lungo (20 volte il
`global_timeout` calcolato sulla base della dimensione della rete) prima di ripetere di nuovo le operazioni
di esame dell'identità.

Altrimenti la tasklet chiama sul suo arco-identità il metodo remoto `search_migration_path(ask_lvl)`.

Se riceve l'eccezione `MigrationPathExecuteFailureError` allora subito riprova con la stessa chiamata
del metodo remoto `search_migration_path` con lo stesso livello.

Se riceve l'eccezione `NoMigrationPathFoundError` allora dovrà fare più operazioni. Per prima cosa
comunica al nodo Coordinator del suo g-nodo di livello `ask_lvl` il fatto che questo tentativo è abortito,
chiamando il metodo `abort_enter` del modulo Coordinator con le modalità descritte nell'analisi.  
Poi decrementa di 1 il valore di `ask_lvl`. Se adesso è minore di 0, allora non è proprio possibile
entrare nell'altra rete. Quindi la tasklet attende un tempo molto lungo (20 volte il
`global_timeout` calcolato sulla base della dimensione della rete) prima di ripetere di nuovo le operazioni
di esame dell'identità.  
Se invece il nuovo valore di `ask_lvl` non è minore di 0, allora ripete dall'inizio le operazioni di
esecuzione dell'ingresso al livello `ask_lvl`, ripartendo dalla chiamata di `begin_enter`.

Se invece il metodo remoto `search_migration_path(ask_lvl)` non rilancia eccezioni, allora la tasklet
riceve una istanza di `EntryData entry_data` con le informazioni necessarie all'ingresso del suo g-nodo di
livello `ask_lvl` nell'altra rete.

Ora la tasklet inventa un identificativo `enter_id`. Poi fa uso della collaborazione con il modulo
Coordinator per avviare una *propagazione con ritorno* a tutti i singoli nodi del suo g-nodo di
livello `ask_lvl`. Essi avviano la prima parte delle operazioni di duplicazione dell'identità usando
l'identificativo `enter_id` comunicato dalla nostra tasklet.  
Quando questa *propagazione con ritorno* è completata, la tasklet fa sì che anche questa nostra
identità avvii la prima parte delle operazioni di duplicazione. Lo fa emettendo il segnale `do_prepare_enter`.

La tasklet di nuovo fa uso della collaborazione con il modulo Coordinator per avviare una *propagazione senza ritorno*
a tutti i singoli nodi del suo g-nodo di livello `ask_lvl`. Essi avviano la seconda parte delle operazioni
di duplicazione dell'identità e la costruzione di un g-nodo isomorfo dentro la nuova rete.  
Ricordiamo che in questa operazione non diamo nessuna importanza al mantenimento della connettività
interna dei g-nodi della vecchia rete: cioè le precedenti identità vengono dismesse immediatamente senza
dover assumere una posizione *di connettività*.

Le informazioni che vanno comunicate ai singoli nodi in questa propagazione sono l'identificativo `enter_id`
e quelle contenute in `entry_data`.

Dopo aver dato il via alla *propagazione senza ritorno* la tasklet fa in modo che anche in questa nostra
identità venga operato questo ingresso. Lo fa emettendo il segnale `do_finish_enter`. Poi,
essendo la sua identità destinata a venire dismessa, termina.

### <a name="Operazioni_su_propagazione"></a>Operazioni su richieste da altri nodi

I comandi che arrivano al modulo Hooking di una nostra identità attraverso i meccanismi di propagazione
all'interno di un g-nodo sono:

*   `prepare_enter`
*   `finish_enter`
*   `prepare_migration`
*   `finish_migration`
*   `we_have_splitted`

## <a name="Requisiti"></a>Requisiti

### <a name="Requisiti_IHookingMapPaths"></a>Interfaccia IHookingMapPaths

Il modulo riceve dal suo utilizzatore una istanza dell'interfaccia `IHookingMapPaths`.  
Tramite questa interfaccia il modulo può:

*   Leggere l'identificativo della rete. Metodo `int64 get_network_id()`.
*   Leggere la dimensione della rete. Metodo `int get_n_nodes()`.  
    Questo metodo restituisce l'informazione secondo le conoscenze del nodo stesso. Cioè non
    interpella il Coordinator della rete.
*   Leggere informazioni sulla topologia della rete:
    *   Metodo `int get_levels()`. Numero di livelli nella rete.
    *   Metodo `int get_gsize(int level)`. Numero di posizioni *reali* per un dato livello.
    *   Metodo `int get_epsilon(int level)`. Se si avvia la ricerca di una migration-path per liberare
        una posizione *reale* al livello `level` questo metodo ci dice di quanti livelli ulteriori
        possiamo salire al massimo per avere una soluzione che ci soddisfa.
*   Leggere informazioni sulla posizione del nodo.
    *   Metodo `int get_my_pos(int level)`. Posizione al livello `level`.  
        Questo dato può cambiare nel tempo. Infatti una identità può diventare di connettività al
        livello `level`.
    *   Metodo `int get_my_eldership(int level)`. Anzianità al livello `level`.  
        L'anzianità del proprio g-nodo di livello `level` (o del nodo stesso se `level=0`) rispetto
        agli altri g-nodi nello stesso g-nodo di livello `level+1` perde di significato nel momento
        in cui una identità diventa di connettività al livello `level` (sebbene il valore che si otterrebbe
        chiamando questo metodo non cambia).
    *   Metodo `int get_subnetlevel()`. Livello della sottorete a gestione autonoma di cui questo nodo
        è il gateway.
*   Leggere informazioni sulle destinazioni di cui il nodo è a conoscenza. Cioè della sua mappa.
    *   Metodo `bool exists(int level, int pos)`. Dice se questo g-nodo esiste nella rete.
    *   Metodo `int get_eldership(int level, int pos)`. Restituisce l'anzianità di questo g-nodo.
    *   Metodo `Gee.List<IPairHCoordInt> adjacent_to_my_gnode(int level_adjacent_gnodes, int level_my_gnode)`.
        Analizzando la mappa dei percorsi noti scopre quali g-nodi di livello `level_adjacent_gnodes`
        siano adiacenti al mio g-nodo di livello `level_my_gnode`.  
        Il metodo può assumere che nella richiesta sia `level_adjacent_gnodes ≥ level_my_gnode`.  
        Per ogni g-nodo adiacente individuato il metodo deve indicare (nei campi appositi dell'interfaccia
        `IPairHCoordInt`) le coordinate gerarchiche del g-nodo relative alla posizione del nostro nodo
        e la posizione del g-nodo di livello `level_my_gnode-1` che risulta essere il border-gnodo
        verso di esso. Quest'ultima deve essere una posizione *reale* all'interno del nostro stesso
        g-nodo di livello `level_my_gnode`.
    *   Metodo `int IHookingManagerStub gateway(int level, int pos)`. Restituisce uno stub per comunicare
        con il gateway del miglior percorso di cui il nodo sia a conoscenza verso quel g-nodo.  
        Questo stub non attende una risposta. Serve a chiamare i metodi remoti `route_xxx`.

### <a name="Requisiti_ICoordinator"></a>Interfaccia ICoordinator

Il modulo riceve dal suo utilizzatore una istanza dell'interfaccia `ICoordinator`.  
Tramite questa interfaccia il modulo può richiedere alcuni tipi di collaborazione con il modulo Coordinator.

In alcuni casi con questa collaborazione si ottiene che si può richiamare un metodo del modulo Hooking stesso
ma da eseguire sul nodo che attualmente risulta il Coordinator del proprio g-nodo di un dato livello.  
Per ognuno di questi casi avremo quindi un metodo nell'interfaccia `ICoordinator` e un metodo nel modulo
Hooking che avranno la stessa (o analoga) firma.  
Con questa modalità abbiamo questi metodi:

*   Metodo `int evaluate_enter(Object evaluate_enter_data) throws AskAgainError, IgnoreNetworkError`.
*   ... `begin_enter`, `completed_enter`, `abort_enter`

* * *

In altri casi, se il nodo in cui ci troviamo è esso stesso il nodo Coordinator del proprio g-nodo di un dato livello,
con questa collaborazione si può accedere in lettura/scrittura alla memoria condivisa del g-nodo. Essa è memorizzata
dal servizio peer-to-peer (vedi modulo PeerServices) implementato nel modulo Coordinator. In tale memoria condivisa
alcune informazioni sono di pertinenza del modulo Hooking.  
Con questa modalità abbiamo questi metodi:

*   ... `get_hooking_memory`, `set_hooking_memory`

* * *

In altri casi con questa collaborazione si ottiene che venga eseguito un metodo del modulo Hooking stesso
su tutti i nodi del proprio g-nodo di un dato livello.  
Per ognuno di questi casi avremo quindi un metodo nell'interfaccia `ICoordinator` e un metodo nel modulo
Hooking che avranno la stessa (o analoga) firma.  
Con questa modalità abbiamo questi metodi:

*   ... `prepare_migration`, `finish_migration`, `prepare_enter`, `finish_enter`, `we_have_splitted`

* * *

In altri casi con questa collaborazione si ottiene che si può richiamare un metodo del modulo Coordinator
nel proprio nodo.  
Con questa modalità abbiamo questi metodi:

*   Metodo `void reserve(int host_lvl, int reserve_request_id, out int new_pos, out int new_eldership) throws CoordReserveError`
*   Metodo `void delete_reserve(int host_lvl, int reserve_request_id)`
*   Metodo `int get_n_nodes()`

### <a name="Requisiti_IIdentityArc"></a>Interfaccia IIdentityArc

Il modulo riceve dal suo utilizzatore una istanza dell'interfaccia `IIdentityArc` per ogni arco-identità
che viene costituito.  
Tramite questa interfaccia il modulo può:

*   Avere uno stub per comunicare col diretto vicino attraverso questo arco-identità.
    Metodo `IHookingManagerStub get_stub()`.  
    Questo stub attende una risposta. Serve a chiamare i metodi remoti `retrieve_network_data` e `search_migration_path`.

### <a name="Requisiti_Segnali"></a>Segnali

Il modulo Hooking associato ad una nostra identità può emettere questi segnali:

*   `same_network(IIdentityArc ia)`
*   `another_network(IIdentityArc ia, int64 network_id)`
*   `do_prepare_enter(int enter_id)`
*   `do_finish_enter(int enter_id, int guest_gnode_level, EntryData entry_data, int go_connectivity_position)`
*   `do_prepare_migration(/* TODO */)`
*   `do_finish_migration(/* TODO */)`

### <a name="Requisiti_Metodiremoti"></a>Metodi remoti

Tra diretti vicini i dialoghi sono espletati con questi metodi remoti:

*   `NetworkData retrieve_network_data(bool ask_coord) throws NotPrincipalError`
*   ``
*   ``

