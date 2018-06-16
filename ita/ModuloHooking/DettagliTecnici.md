# Modulo Hooking - Dettagli Tecnici

1.  [Associazione del modulo ad una identità](#Associazione_identita)
    1.  [Operazioni su un proprio arco-identità](#Operazioni_arco_identita)
    1.  [Operazioni su richieste da altri nodi](#Operazioni_su_propagazione)

## <a name="Associazione_identita"></a>Associazione del modulo ad una identità

Una istanza di `HookingManager` viene costruita quando si crea una identità nel
sistema e a questa viene associata.

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

### <a name="Operazioni_arco_identita"></a>Operazioni su un proprio arco-identità

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

### Interfaccia IHookingMapPaths

...

### Interfaccia ICoordinator

L'interfaccia `ICoordinator`...

Con il suo metodo `evaluate_enter` il modulo Hooking può richiedere al programma di chiamare
il metodo `evaluate_enter` sul nodo Coordinator del suo g-nodo di un dato livello.

```
int evaluate_enter() throws AskAgainError, IgnoreNetworkError;
```

Sul nodo Coordinator del g-nodo interessato il programma dovrà chiamare un corrispettivo metodo pubblico
che il modulo Hooking deve quindi esporre.

```
public int evaluate_enter() throws AskAgainError, IgnoreNetworkError;
```

### Interfaccia IIdentityArc

L'interfaccia `IIdentityArc`...

### Segnali

Il modulo Hooking associato ad una nostra identità può emettere questi segnali:

*   `same_network(IIdentityArc ia)`
*   `another_network(IIdentityArc ia, int64 network_id)`
*   `do_prepare_enter(int enter_id)`
*   `do_finish_enter(int enter_id, int guest_gnode_level, EntryData entry_data, int go_connectivity_position)`
*   `do_prepare_migration(/* TODO */)`
*   `do_finish_migration(/* TODO */)`

### Metodi remoti

Tra diretti vicini i dialoghi sono espletati con questi metodi remoti:

*   `retrieve_network_data()`
*   ``
*   ``

