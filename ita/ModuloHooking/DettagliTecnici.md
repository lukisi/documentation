# Modulo Hooking - Dettagli Tecnici

1.  [Associazione del modulo ad una id](#associazione-del-modulo-ad-una-id)
    1.  [Operazioni su un proprio arco-id](#operazioni-su-un-proprio-arco-id)
    1.  [Operazioni su richieste da altri nodi](#operazioni-su-richieste-da-altri-nodi)
1.  [Requisiti](#Requisiti)
    1.  [Interfaccia IHookingMapPaths](#Requisiti_IHookingMapPaths)
    1.  [Interfaccia ICoordinator](#Requisiti_ICoordinator)
    1.  [Interfaccia IIdentityArc](#Requisiti_IIdentityArc)
    1.  [Segnali](#Requisiti_Segnali)
    1.  [Metodi remoti](#Requisiti_Metodiremoti)

## Associazione del modulo ad una id

Una istanza di `HookingManager` viene costruita ogni volta che si crea una identità nel
sistema. L'istanza di `HookingManager` viene associata a questa identità.

Soltanto dopo che l'identità diventa *bootstrapped* questo evento viene comunicato al
modulo (tramite il metodo `bootstrapped` di `HookingManager`) e esso diventa operativo.  
Nel caso della prima identità questo avviene immediatamente e senza che esistano ancora
archi-identità.  
Nel caso di una identità duplicata questo avviene dopo un po' di tempo; di norma esistono
degli archi-identità che vengono subito comunicati tramite il metodo `bootstrapped` a
questa istanza di `HookingManager`.

L'utilizzatore del modulo Hooking comunica in seguito a questa istanza di `HookingManager` la
nascita e la rimozione di ogni arco-identità associato a quella identità nel sistema. Questo
chiamando i metodi pubblici `add_arc` e `remove_arc` dell'istanza di `HookingManager`.

Il modulo Hooking associato ad una identità, per ogni arco-identità che conosce, avvia
una tasklet che ha il compito di dialogare con il diretto vicino su questo arco-identità.
Da questo dialogo scaturiscono alcune operazioni di pertinenza del modulo Hooking che sono
svolte nella stessa tasklet. E da queste operazioni deriva l'emissione di alcuni segnali
che informano l'utilizzatore del modulo su operazioni da intraprendere.

Alcune operazioni di pertinenza del modulo Hooking possono essere richieste al modulo
associato ad una identità attraverso la ricezione di chiamate di metodi remoti da parte
dei diretti vicini. Queste di solito consistono nella richiesta di informazioni
(come il metodo `retrieve_network_data` e il metodo `search_migration_path`) e il modulo
può servirle solo se la sua identità è *bootstrapped*. Altrimenti rilancia l'eccezione `NotBoostrappedError`.

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
l'eccezione `NotPrincipalError`, l'eccezione `NotBoostrappedError`, oppure informazioni sulla rete
di appartenenza del vicino.  
L'esito di questa chiamata può anche essere un errore `StubError` o simile. In questo caso (vale anche su
altre chiamate) la tasklet segnala che l'arco ha fallito (segnale `failing_arc`) e lo rimuove e termina.

Se rileva che l'identità vicina è *di connettività*, allora la tasklet termina.

Se rileva che l'identità vicina non è ancora *bootstrapped*, allora aspetta un po' e riprova.

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

In teoria a questo punto siamo sicuri che l'identità del vicino è *bootstrapped*. Comunque il metodo
remoto prevede la possibilità di rilanciare l'eccezione `NotBoostrappedError`. Se dovesse ricevere
questa eccezione la tasklet segnala che l'arco-identità va rimosso e poi termina.

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

### Operazioni su richieste da altri nodi

#### Per richiesta di un servizio peer-to-peer

I comandi che arrivano al modulo Hooking della nostra identità principale (la quale si trova ad essere
il Coordinator di un g-nodo) attraverso i meccanismi del modulo PeerServices sono:

*   `evaluate_enter`
*   `begin_enter`
*   `completed_enter`
*   `abort_enter`

#### Per propagazione

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
        una posizione *reale* per un g-nodo guest di livello `level` questo metodo ci restituisce il valore di `𝜀` tale che
        una posizione libera in un g-nodo host di livello `level + 𝜀` ci soddisfa. Sicuramente `𝜀 ≥ 1`.
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

*   Metodo `evaluate_enter`.  
    Quando il modulo Hooking (ad esempio nel nodo *n* che incontra un'altra rete) vuole chiamare questo
    metodo sul Coordinator di un suo g-nodo di livello *l* (nel caso di `evaluate_enter` si interroga sempre il Coordinator
    di tutta la rete) chiama un metodo della classe interna `ProxyCoord` che ha questa signature:  
    `int evaluate_enter(EvaluateEnterData evaluate_enter_data) throws AskAgainError, IgnoreNetworkError, CoordProxyError, UnknownResultError`  
    Questo metodo si avvale di un metodo nell'interfaccia `ICoordinator` che ha questa signature:  
    `Object evaluate_enter(Object evaluate_enter_data) throws CoordProxyError`  
    Il metodo nel modulo `Hooking` ha questa signature:  
    `Object evaluate_enter(Object evaluate_enter_data, Gee.List<int> client_address)`  
    Un metodo con la medesima signature è nella classe interna `ProxyCoord`:  
    `Object execute_proxy_evaluate_enter(Object evaluate_enter_data, Gee.List<int> client_address)`  
    Questa gestisce il lock della funzione (nel caso evaluate serve un lock che garantisca che ci sia una sola
    valutazione alla volta). Inoltre gestisce il lock della memoria condivisa del g-nodo relativa al modulo Hooking.
    Inoltre interpreta le classi serializzabili usate per le comunicazioni e poi chiama un altro metodo
    nella classe interna `ProxyCoord` che invece ha la signature che serve alla logica:  
    `int execute_evaluate_enter(int lock_id, EvaluateEnterData evaluate_enter_data, Gee.List<int> client_address) throws AskAgainError, IgnoreNetworkError`
*   Metodo `begin_enter`.  
    Analogo a sopra. Il metodo per avviare la chiamata è nella classe interna `ProxyCoord` e ha questa signature:  
    `void begin_enter(int lvl, BeginEnterData begin_enter_data) throws AlreadyEnteringError, CoordProxyError, UnknownResultError`  
    Il metodo nell'interfaccia `ICoordinator` ha questa signature:  
    `Object begin_enter(int lvl, Object begin_enter_data) throws CoordProxyError`  
    Il metodo che implementa la logica è ancora nella classe interna `ProxyCoord` e ha questa signature:  
    `void execute_begin_enter(int lock_id, int lvl, BeginEnterData begin_enter_data, Gee.List<int> client_address) throws AlreadyEnteringError`
*   Metodo `completed_enter`.  
    Analogo a sopra. Il metodo per avviare la chiamata è nella classe interna `ProxyCoord` e ha questa signature:  
    `void completed_enter(int lvl, CompletedEnterData completed_enter_data) throws CoordProxyError, UnknownResultError`  
    Il metodo nell'interfaccia `ICoordinator` ha questa signature:  
    `Object completed_enter(int lvl, Object completed_enter_data) throws CoordProxyError`  
    Il metodo che implementa la logica è ancora nella classe interna `ProxyCoord` e ha questa signature:  
    `void execute_completed_enter(int lock_id, int lvl, CompletedEnterData completed_enter_data, Gee.List<int> client_address)`
*   Metodo `abort_enter`.  
    Analogo a sopra. Il metodo per avviare la chiamata è nella classe interna `ProxyCoord` e ha questa signature:  
    `void abort_enter(int lvl, AbortEnterData abort_enter_data) throws CoordProxyError, UnknownResultError`  
    Il metodo nell'interfaccia `ICoordinator` ha questa signature:  
    `Object abort_enter(int lvl, Object abort_enter_data) throws CoordProxyError`  
    Il metodo che implementa la logica è ancora nella classe interna `ProxyCoord` e ha questa signature:  
    `void execute_abort_enter(int lock_id, int lvl, AbortEnterData abort_enter_data, Gee.List<int> client_address)`

* * *

In altri casi, se il nodo in cui ci troviamo è esso stesso il nodo Coordinator del proprio g-nodo di un dato livello,
con questa collaborazione si può accedere in lettura/scrittura alla memoria condivisa del g-nodo. Essa è memorizzata
dal servizio peer-to-peer (vedi modulo PeerServices) implementato nel modulo Coordinator. In tale memoria condivisa
alcune informazioni sono di pertinenza del modulo Hooking.

Si possono garantire operazioni atomiche di lettura e successiva scrittura (aggiornamenti) di questa memoria
condivisa per il fatto che le operazioni sono eseguite esclusivamente nell'attuale nodo Coordinator.  
Infatti, prendiamo ad esempio il metodo `evaluate_enter` visto poco sopra. La sua implementazione
nel metodo interno `ProxyCoord.execute_evaluate_enter` è destinata ad essere chiamata per il tramite
del modulo Coordinator per la ricezione della richiesta p2p EvaluateEnterRequest.  
Il servizio p2p Coordinator (si veda anche la documentazione del modulo PeerServices) gestisce un database
distribuito del tipo a chiavi fisse. In questo tipo non sono previsti rifiuti di tipo `OutOfMemory`.  
In particolare in questo servizio la classe EvaluateEnterRequest non è di tipo `insert`, né `read_only`, né `update`,
né `replica_value`, né `replica_delete`. Per queste caratteristiche non prevede rifiuti di tipo `NotExaustive`.  
Quindi la richiesta è sempre soddisfatta dall'attuale nodo Coordinator, anche nel caso fosse appena entrato nella rete
e quindi fosse ancora nella fase di recupero dei record (`NotExaustive`).  
In conclusione per garantire atomicità nelle operazioni eseguite all'interno dell'implementazione del
metodo `evaluate_enter`, basta garantire atomicità nel singolo nodo.

Quindi le operazioni di aggiornamento degli aspetti di pertinenza del modulo Hooking nella memoria condivisa di un certo
g-nodo vengono fatte esclusivamente nelle implementazioni dei metodi `evaluate_enter`, `begin_enter`,
`completed_enter` e `abort_enter`: prima viene chiamato il metodo `lock_hooking_memory` della classe interna `ProxyCoord`;
poi all'abbisogna i suoi metodi `get_hooking_memory` e `set_hooking_memory`; infine il suo metodo `unlock_hooking_memory`.  
La classe interna `ProxyCoord` viene istanziata nel costruttore di HookingManager. Essa ha il membro `_lock_hooking_memory`
che è un intero nullable che rappresenta un lock.  
Il metodo `lock_hooking_memory` acquisisce un lock, eventualmente attendendo che si liberi il precedente.  
I metodi `get_hooking_memory`, `set_hooking_memory` e `unlock_hooking_memory` devono risultare chiamati dal possessore
attuale del lock, altrimenti vanno in errore.  
Il metodo `unlock_hooking_memory` rilascia il lock.

*   Metodo `get_hooking_memory`.  
    Quando il modulo Hooking nel nodo Coordinator di un g-nodo vuole leggere la memoria condivisa di sua
    pertinenza chiama un metodo della classe interna `ProxyCoord` che ha questa signature:  
    `HookingMemory get_hooking_memory(int lock_id, int lvl)`  
    Questo metodo si avvale di un metodo nell'interfaccia `ICoordinator` che ha questa signature:  
    `Object get_hooking_memory(int lvl) throws CoordProxyError`
*   Metodo `set_hooking_memory`.  
    Quando il modulo Hooking nel nodo Coordinator di un g-nodo vuole scrivere la memoria condivisa di sua
    pertinenza chiama un metodo della classe interna `ProxyCoord` che ha questa signature:  
    `void set_hooking_memory(int lock_id, int lvl, HookingMemory memory)`  
    Questo metodo si avvale di un metodo nell'interfaccia `ICoordinator` che ha questa signature:  
    `void set_hooking_memory(int lvl, Object memory) throws CoordProxyError`

Abbiamo detto che queste operazioni sono sempre svolte nell'attuale nodo Coordinator del g-nodo in questione.
Se però il presente nodo è ancora `NotExaustive` le funzioni fornite dal modulo PeerServices per la
gestione di un database distribuito del tipo a chiavi fisse fanno in modo che in lettura il record
venga comunque recuperato dal precedente detentore.  

* * *

In altri casi con questa collaborazione si ottiene che venga eseguito un metodo del modulo Hooking stesso
su tutti i nodi del proprio g-nodo di un dato livello.

Con questa modalità abbiamo questi metodi:

*   Metodo `prepare_enter`.  
    Quando il modulo Hooking (ad esempio nel nodo *n* che ha deciso di entrare in un'altra rete) vuole far
    eseguire questo metodo con una *propagazione con ritorno* su tutti i nodi del
    suo g-nodo di livello *l* chiama un metodo della classe interna `PropagationCoord` che ha questa signature:  
    `void prepare_enter(int lvl, PrepareEnterData prepare_enter_data)`  
    Questo metodo si avvale di un metodo nell'interfaccia `ICoordinator` che ha questa signature:  
    `void prepare_enter(int lvl, Object prepare_enter_data)`  
    Questo permette la cooperazione con il modulo Coordinator il quale fa in modo che tutti i nodi del g-nodo
    (compreso il nodo presente) eseguano una volta il metodo `prepare_enter` del modulo `Hooking`, che ha questa signature:  
    `void prepare_enter(int lvl, Object prepare_enter_data)`  
    Un metodo con la medesima signature è nella classe interna `PropagationCoord`:  
    `execute_propagate_prepare_enter(int lvl, Object prepare_enter_data)`  
    Questa interpreta le classi serializzabili usate per le comunicazioni e poi chiama un altro metodo
    nella classe interna `PropagationCoord` che invece ha la signature che serve alla logica:  
    `void execute_prepare_enter(int lvl, PrepareEnterData prepare_enter_data)`  
    Questa fa emettere il segnale `do_prepare_enter`.
*   Metodo `finish_enter`.  
    Quando il modulo Hooking (ad esempio nel nodo *n* che ha deciso di entrare in un'altra rete) vuole far
    eseguire questo metodo con una *propagazione senza ritorno* su tutti i nodi del
    suo g-nodo di livello *l* chiama un metodo della classe interna `PropagationCoord` che ha questa signature:  
    `void finish_enter(int lvl, FinishEnterData finish_enter_data)`  
    Il metodo nell'interfaccia `ICoordinator` ha questa signature:  
    `void finish_enter(int lvl, Object finish_enter_data)`  
    Il metodo che implementa la logica è ancora nella classe interna `PropagationCoord` e ha questa signature:  
    `void execute_finish_enter(int lvl, FinishEnterData finish_enter_data)`  
    Questa fa emettere il segnale `do_finish_enter`.
*   Metodo `prepare_migration`.  
    Quando il modulo Hooking (in un nodo che avvia una migrazione parte di una migration_path) vuole far
    eseguire questo metodo con una *propagazione con ritorno* su tutti i nodi del
    suo g-nodo di livello *l* chiama un metodo della classe interna `PropagationCoord` che ha questa signature:  
    `void prepare_migration(int lvl, PrepareMigrationData prepare_migration_data)`  
    Il metodo nell'interfaccia `ICoordinator` ha questa signature:  
    `void prepare_migration(int lvl, Object prepare_migration_data)`  
    Il metodo che implementa la logica è ancora nella classe interna `PropagationCoord` e ha questa signature:  
    `void execute_prepare_migration(int lvl, PrepareMigrationData prepare_migration_data)`  
    Questa fa emettere il segnale `do_prepare_migration`.
*   Metodo `finish_migration`.  
    Quando il modulo Hooking (in un nodo che avvia una migrazione parte di una migration_path) vuole far
    eseguire questo metodo con una *propagazione senza ritorno* su tutti i nodi del
    suo g-nodo di livello *l* chiama un metodo della classe interna `PropagationCoord` che ha questa signature:  
    `void finish_migration(int lvl, FinishMigrationData finish_migration_data)`  
    Il metodo nell'interfaccia `ICoordinator` ha questa signature:  
    `void finish_migration(int lvl, Object finish_migration_data)`  
    Il metodo che implementa la logica è ancora nella classe interna `PropagationCoord` e ha questa signature:  
    `void execute_finish_migration(int lvl, FinishMigrationData finish_migration_data)`  
    Questa fa emettere il segnale `do_finish_migration`.
*   Metodo `we_have_splitted`.  
    ... **TODO**

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

## Annotazioni varie

### strategia di ingresso

Il nodo *n* di *G* vuole chiedere a *v* di *J* il permesso di entrare nella rete *J* in un g-nodo di *v* insieme al
suo g-nodo di livello *l*, con *l* tra 0 e levels-1.  
I nodi *n* e *v* sono identità principali.

Questa richiesta il nodo *n* la fa nel modulo Hooking nella tasklet di gestione di un arco: questa tasklet
viene avviata sull'aggiunta di un arco-identità (il metodo `add_arc` avvia la tasklet `add_arc_tasklet`, vedi
file `arc_handler.vala`).  
La richiesta è fatta con `IEntryData resp2 = stub.search_migration_path(ask_lvl);` e può rilanciare
eccezione NoMigrationPathFoundError (la tasklet prova a degradare) o MigrationPathExecuteFailureError (la
tasklet riprova subito) oppure restituirà i dati per l'ingresso.

Il nodo *v* cerca, se esiste, *P* la *shortest migration-path* di livello *l*.

Le caratteristiche di *P* sono la lunghezza *d* (che può essere 0=impropria o maggiore) e il delta tra *hl* e *l*
che può essere 1 o maggiore.

Una migrazione con *d* maggiore di 0 è possibile solo se *l* è minore di levels-1.

Per ogni livello della topologia della rete è definito un delta *𝜀* accettabile. Quindi una volta trovata
la più breve (*d* minore) soluzione, se il suo delta è superiore a *𝜀*, la ricerca si addentra tra
le possibili alternative, ammettendo che la distanza della successiva non si incrementi più di 5 passi
o di 1.3 volte i passi precedenti, fintanto che non trovi una soluzione con delta accettabile.

Questa ricerca viene avviata nel nodo *v* nel modulo Hooking nel metodo remoto `search_migration_path`
(nel file `hooking.vala`): si guarda il *𝜀* e si calcola il `ok_host_lvl` e si chiama il metodo
`find_shortest_mig` (nel file `hooking.vala`)

La logica del tentativo di trovare una soluzione con delta accettabile è nella funzione `find_shortest_mig`:
infatti questa restituisce una lista di soluzioni: prima di restituirla continua a
cercare fintanto che:

*   ha terminato l'esplorazione:  
    esce dal ciclo `while (! Q.is_empty)`, *oppure*
*   ha trovato una soluzione con delta soddisfacente:  
    `if (final_host_lvl <= ok_host_lvl) ... return solutions;`, *oppure*
*   la successiva soluzione è troppo lontana: 
    `if (prev_sol_distance + 5 ... && prev_sol_distance * 1.3 ...) break;`

La struttura `TupleGNode` è serializzabile si trova nel file `serializable.vala`.

Le strutture `Solution` e `SolutionStep` sono nel file `structs.vala`. La `SolutionStep` ha il metodo `get_distance`.

Le funzioni di utilità `make_tuple_from_level`, `make_tuple_from_hc`, `make_tuple_up_to_level`,
`level`, `positions_equal`, `tuple_contains`, ...
sono nel file `structs.vala`.

Per contattare un nodo in `current.visiting_gnode` il metodo `find_shortest_mig` instrada un pacchetto:
chiama `message_routing.send_search_request` (nel file `message_routing.vala`).
Questo metodo in base ai parametri ricevuti costruisce un pacchetto SearchMigrationPathRequest
che instrada chiamando il metodo remoto `stub.route_search_request` e poi aspetta la risposta su un *channel*.  
Il metodo remoto `route_search_request` (gestito con `message_routing.route_search_request` nel file `message_routing.vala`)
se non capita un problema nel tragitto, finisce che chiama su un nodo in `current.visiting_gnode`
il metodo `execute_search` (nel file `hooking.vala`). Ottiene un risultato e ci costruisce un pacchetto SearchMigrationPathResponse
che instrada chiamando il metodo remoto `stub.route_search_response`. Il metodo
remoto `route_search_response` (gestito con `message_routing.route_search_response` nel file `message_routing.vala`)
finisce con lo scrivere il pacchetto risposta sul *channel* nel nodo originante la richiesta.  
Se invece capita nel tragitto un problema, il metodo remoto `route_search_request` costruisce un pacchetto SearchMigrationPathErrorPkt
e lo instrada chiamando il metodo remoto `stub.route_search_error`. Il metodo
remoto `route_search_error` (gestito con `message_routing.route_search_error` nel file `message_routing.vala`)
finisce con lo scrivere il pacchetto errore sul *channel* nel nodo originante la richiesta.  
Ricevendo così la risposta oppure l'errore, il metodo `message_routing.send_search_request` lo
interpreta e lo restituisce al chiamante, cioè al metodo `find_shortest_mig`.

Nel `message_routing.send_search_request` la `SolutionStep current` viene tradotta con `get_path_hops`
in una lista di `PathHop`.

La struttura `PathHop` è serializzabile si trova nel file `serializable.vala`.

La funzione di utilità `get_path_hops` si trova nel file `structs.vala`.

La logica illustrata nell'analisi che segue il nodo interrogato in `current.visiting_gnode` è
implementata nel metodo `execute_search`. In particolare l'individuazione dei g-nodi adiacenti
che nell'analisi è rappresentata con la funzione `adj_to_me` viene demandata all'utilizzatore
del modulo per mezzo dell'interfaccia `map_paths.adjacent_to_my_gnode`.

Dopo che le possibili soluzioni sono state trovate da `find_shortest_mig`, il metodo remoto `search_migration_path`
per le soluzioni scartate avvia in delle tasklet la richiesta di rimuovere la prenotazione
tramite `message_routing.send_delete_reserve_request`. Inoltre esegue (se necessario) la
migration-path con il metodo `execute_shortest_mig`.  
Di seguito restituisce al chiamante (il nodo *n*) i dati per l'ingresso nella rete.

Il metodo `execute_shortest_mig` instrada i pacchetti che servono all'esecuzione della migration-path
per il tramite di `message_routing.send_mig_request`.

...
