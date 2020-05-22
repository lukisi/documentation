# Caso banale

Consideriamo un caso banale. Un nodo in cui il programma si avvia e dopo un po' viene
terminato senza che incontri altri nodi.

Nel presente documento trattiamo la sequenza di operazioni che il programma fa in
questo caso. In particolare le operazioni di creazione e rimozione delle singole
istanze delle classi dei vari moduli.

La parte iniziale sarà comune a tutti i casi. Ogni nodo infatti, appena il programma
si avvia, si considera l'unico membro di una nuova rete.

La parte finale sarà comune a qualsiasi nodo che al termine del programma si trovi
a essere l'ultimo rimasto della sua rete.

## Panoramica delle operazioni

All'avvio del programma si crea una istanza di `NeighborhoodManager`.  
Questa istanza viene subito usata per mettere il nodo in ascolto sulle varie interfacce
di rete che il programma dovrà gestire. Come risultato verranno rilevati su questa
istanza alcuni segnali per mezzo dei quali il programma avrà accesso a informazioni
che sono necessarie alla creazione di una istanza di `IdentityManager`.

Subito dopo si crea una istanza di `IdentityManager`.  
La prima identità del nodo, che è una *identità principale*, nasce nel momento stesso
in cui si avvia il programma. L'istanza di `IdentityManager` quando viene costruita
assegna subito a questa prima identità il suo identificativo. Quindi il programma,
subito dopo aver costruito l'istanza di `IdentityManager`, la interroga per conoscere
la prima identità.

Poi il programma crea una istanza di `QspnManager` che associa alla prima identità
del nodo.  
L'istanza di `QspnManager` è creata in modalità `create_net`. In questa modalità viene
costituita dal modulo `Qspn` una nuova rete composta dal solo nodo.

Ora il programma dovrà creare una istanza di `PeersManager` che assocerà alla prima
identità del nodo. Questa è necessaria perché poi il programma dovrà creare una
istanza di `CoordinatorManager` che assocerà alla prima identità del nodo.
Infine questa è necessaria perché poi il programma dovrà creare una istanza di
`HookingManager` che assocerà alla prima identità del nodo.  
Va notato che le prime istanze di `PeersManager` e`CoordinatorManager` si troveranno a
operare in un contesto molto "semplice", poiché la prima identità del nodo è per
definizione l'unico membro di una nuova rete. I servizi P2P saranno sempre forniti dal
nodo stesso senza nessuna reale comunicazione di rete. Il nodo stesso sarà
il *Coordinator* a tutti i livelli.


## Implementazione testsuite locale

Implementiamo un programma che possiamo usare per verificare le operazioni che farebbe
un nodo in questo semplice scenario.

Useremo il supporto del medium `system` fornito dal framework ZCD.  
Il nome dell'eseguibile sarà `sys_ntkd_alone`. La parte `sys` indica che usa il medium
system, cioè socket di sistema per comunicazioni locali tra processi. La parte `alone`
indica che vogliamo provare questo semplice scenario.

### Appunti dell'implementazione

I file sorgente sono nella dir `sys-ntkd-alone`.

Il nome del pacchetto (nel file configure.ac) è `sys-ntkd-alone`.

Il nome dell'eseguibile sarà `sys_ntkd_alone`.

La routine main è nel file `main.vala`.

Per indicare i namespaces usiamo in seguito le abbreviazioni:

*   `G.` per `Gee`
*   `N.` per `Netsukuku`
*   `N.N.` per `Netsukuku.Neighborhood`
*   `N.I.` per `Netsukuku.Identities`
*   `N.Q.` per `Netsukuku.Qspn`
*   `N.P.` per `Netsukuku.PeerServices`
*   `N.C.` per `Netsukuku.Coordinator`
*   `N.H.` per `Netsukuku.Hooking`
*   `N.A.` per `Netsukuku.Andna`
*   `T.` per `TaskletSystem`

#### Comunicazioni in rete

Simuleremo due interfacce di rete, su due diversi segmenti di rete. In ognuno dei
due il nostro nodo sarà il solo connesso.

Questo comporta alcune conseguenze:

*   Il programma si troverà a trasmettere in broadcast (datagram) alcuni messaggi
    sull'una o l'altra interfaccia.
*   Il programma non si troverà mai a trasmettere in unicast (stream) alcun
    messaggio, poiché il nodo non costituirà nessun arco con altri nodi.
*   Il programma si troverà a rilevare alcuni messaggi in broadcast (datagram)
    sull'una o l'altra interfaccia. Saranno sempre messaggi provenienti dal
    nodo stesso.
*   Il programma non si troverà mai a rilevare alcun messaggio in unicast (stream).

* * *

In una classe memorizziamo le informazioni relative a una interfaccia di rete che
il programma deve gestire.  
Vedere la classe `N.HandledNic` nel file `main.vala`.  
I dati memorizzati includono `dev`, `mac`, `linklocal`, `INeighborhoodNetworkInterface nic`.

In una hashmap memorizziamo le istanze di `N.HandledNic` di ogni interfaccia di rete che il
programma deve gestire, indicizzate per il nome `dev`.  
Vedere la variabile globale `handlednic_map` nel file `main.vala`.

Gli oggetti in `handlednic_map` servono al programma in quanto coordina le operazioni
dei vari moduli.

* * *

In una classe memorizziamo le informazioni relative a una interfaccia di rete che
il programma deve gestire.  
Vedere la classe `N.PseudoNetworkInterface` nel file `main.vala`. I dati memorizzati
includono `dev`, `mac`, `linklocal`, `INeighborhoodNetworkInterface nic` e anche
informazioni legate al fatto che si tratta di pseudo-interfacce di rete simulate da
un socket di sistema: `send_pathname`, `listen_pathname`, `st_listen_pathname`.

In una hashmap memorizziamo le istanze di `N.PseudoNetworkInterface` di ogni
interfaccia di rete che il programma deve gestire, indicizzate per il nome
`dev`.  
Vedere la variabile globale `pseudonic_map` nel file `main.vala`.

Gli oggetti in `pseudonic_map` servono solo a questa specifica
testsuite in quanto deve fornire i parametri necessari ai metodi di ZCD a supporto
delle comunicazioni nel medium *system*.

* * *

La fase d'inizio ascolto su una interfaccia (in broadcast e in unicast sul
linklocal) va svolta in correlazione con quella di aggiunta della stessa
alla gestione del `N.N.NeighborhoodManager`.  
Analogamente la fase di fine ascolto va svolta in correlazione con
quella di rimozione dell'interfaccia dalla gestione del `N.N.NeighborhoodManager`.

Vedere la funzione `void stop_rpc(string dev)` nel file `main.vala`.

* * *

In una unica classe viene implementato il codice per avviare le tasklet che
si metteranno in ascolto per comunicazioni di rete in entrata. Queste tasklet
in questa particolare testsuite staranno in ascolto solo su socket di sistema;
non ci saranno reali comunicazioni di rete. Per quanto riguarda le modalità,
invece, anche nella presente testsuite saranno presenti entrambe:
stream e datagram.  
Vedere la classe `N.SkeletonFactory` nel file `rpc/skeleton_factory.vala`.

La classe è pensata per avere una sola istanza in una variabile globale
`skeleton_factory` nel file `main.vala`, a uso comune di tutto il codice.

Nella fase di avvio, il programma (nella routine main) costruisce una
istanza di questa classe.  
Più avanti valorizzerà la sua proprietà pubblica `NeighborhoodNodeID whole_node_id`
con l'identificativo di questo nodo fornito dal modulo `Neighborhood`.

Il costruttore di `N.SkeletonFactory` valorizza la proprietà privata
`ServerDelegate dlg`. Servirà alle tasklet che ricevono i
messaggi, come illustreremo a breve.  
Inoltre valorizza la proprietà privata `NodeSkeleton node_skeleton`. Sarà
questa a memorizzare l'identificativo di questo nodo fornito passato
dalla routine main più avanti alla proprietà pubblica `whole_node_id`.

Il metodo `start_stream_system_listen` avvia una tasklet per gestire i messaggi
di tipo stream trasmessi in unicast a uno specifico indirizzo IP. Nella presente
testsuite il programma richiamerà questo metodo sia per gestire un IP linklocal,
sia per gestire un IP pubblico. In realtà non riceveremo mai tali messaggi.  
A questo metodo viene passata la stringa `listen_pathname`, come prescrive la
funzione `stream_system_listen` fornita dalla libreria `ntkdrpc` basata su ZCD.
Inoltre il metodo si costruisce un `N.IErrorHandler` apposito per questo
listener (la classe `ServerErrorHandler` è una privata inner-classe
di `N.SkeletonFactory`) e usa una istanza di `N.IDelegate` in comune con tutti
i listener (la classe `ServerDelegate` è una privata inner-classe
di `N.SkeletonFactory`); entrambe le classi sono prescritte da `ntkdrpc`.  
Infine il metodo `start_stream_system_listen` memorizza in un HashMap l'handle
della tasklet avviata associandolo alla stringa `listen_pathname`, di modo che
la stessa classe `N.SkeletonFactory` con il metodo `stop_stream_system_listen`
possa gestirne la terminazione.

Il metodo `start_datagram_system_listen` avvia una tasklet per gestire i
messaggi di tipo datagram trasmessi in broadcast su un segmento di rete
(broadcast domain) e recepiti da una certa interfaccia di rete
del nodo corrente. In realtà rileveremo tali messaggi solo quando sarà
il nostro stesso nodo a trasmetterli.  
A questo metodo viene passata la stringa `listen_pathname`, la stringa
`send_pathname` e una istanza di `ISrcNic src_nic`, come prescrive la funzione
`datagram_system_listen` fornita dalla libreria `ntkdrpc` basata su ZCD.
Inoltre il metodo si costruisce un `N.IErrorHandler` apposito per questo listener
e usa l'istanza comune di `N.IDelegate`, prescritte da `ntkdrpc`.  
Infine il metodo `start_datagram_system_listen` memorizza in un HashMap l'handle
della tasklet avviata associandolo alla stringa `listen_pathname`, di modo che
la stessa classe `N.SkeletonFactory` con il metodo `stop_datagram_system_listen`
possa gestirne la terminazione.

La classe ServerDelegate (nel metodo `get_addr_set` prescritto da `ntkdrpc`)
per prima cosa discerne se ha ricevuto un messaggio di tipo stream o di tipo
datagram. Il primo caso non è contemplato da questa testsuite, quindi
il codice semplicemente va in errore.  
Nel secondo caso la classe fa il suo lavoro usando il metodo `get_dispatcher_set`
di `N.SkeletonFactory` passando il CallerInfo fornito da ZCD.

Il metodo `get_dispatcher_set` è usato per gestire i messaggi in datagram. Gli viene
passata una istanza di `DatagramCallerInfo`. Dovrà restituire una lista di
*skeleton* che saranno chiamati a eseguire una procedura remota come effetto
del messaggio rilevato.  
A seconda del tipo di CallerInfo, il metodo capisce se il messaggio è destinato
a una identità o a un nodo. Inoltre capisce se il messaggio è destinato a qualcuno
in questo sistema.

*   Se il caller ha in `source_id` un `WholeNodeSourceID` e in `broadcast_id` un `EveryWholeNodeBroadcastID`:
    *   Siccome i messaggi di questo tipo sono destinati sempre a tutti i nodi, semplicemente
        restituisce in un set il proprio `node_skeleton`.
*   Se il caller ha in `source_id` un `IdentityAwareSourceID` e in `broadcast_id` un `IdentityAwareBroadcastID`
    e in `src_nic` un `NeighbourSrcNic` e in `listener` un `DatagramSystemListener`:
    *   Questo evento non si verifica mai poiché non ci saranno archi. Il codice
        semplicemente va in errore.

* * *

Sempre nella classe `N.SkeletonFactory` nel file `rpc/skeleton_factory.vala`, ci
sono metodi che gli skeleton usano durante l'esecuzione di un metodo remoto, per
avere informazioni aggiuntive sul chiamante del metodo.

Il metodo `from_caller_get_nodearc` è usato per richieste di nodo, per sapere
da quale arco una tale richiesta è arrivata.  
In questa testsuite non si formeranno mai archi, quindi
il codice semplicemente restituisce *null*.

* * *

Sempre nel file `rpc/skeleton_factory.vala`, il `NodeSkeleton node_skeleton`
è uno *skeleton* per i metodi remoti di nodo. Ci sarà
una sola istanza che rappresenta il sistema. Ha il membro pubblico
`NeighborhoodNodeID id` che contiene il suo identificativo di nodo.

Restituisce con appositi getter i manager del modulo Neighborhood e
del modulo Identities, che sono moduli di nodo. Quindi sono implementati
i metodi `neighborhood_manager_getter` e `identity_manager_getter`.

* * *

Sempre nel file `rpc/skeleton_factory.vala`, la classe `IdentitySkeleton`
è uno *skeleton* per i metodi remoti di identità.
Ogni volta che serve, cioè in ricezione di un messaggio, viene costruita
una nuova istanza passando al costruttore l'istanza di `IdentityData`.

Restituisce con appositi getter i manager dei moduli di identità:
`N.IQspnManagerSkeleton`, `N.IPeersManagerSkeleton`, `N.ICoordinatorManagerSkeleton`,
`N.IHookingManagerSkeleton`, `N.IAndnaManagerSkeleton`.  
È implementato il `qspn_manager_getter` che prevede una attesa di qualche istante
se non è stato ancora costruito.  
Non è ancora implementato il `peers_manager_getter`.  
Non è ancora implementato il `coordinator_manager_getter`.  
Non è ancora implementato il `hooking_manager_getter`.  
Va ancora fatto in `ntkdrpc` quanto serve a produrre il metodo astratto `andna_manager_getter`. Dopo
andrà implementato il `andna_manager_getter`.

* * *

In una unica classe viene implementato il codice per ottenere degli stub
radice di vario tipo (broadcast, unicast, di nodo o d'identità).  
Vedere la classe `N.StubFactory` nel file `rpc/stub_factory.vala`.

La classe è pensata per avere una sola istanza in una variabile globale
`stub_factory` nel file `main.vala`, a uso comune di tutto il codice.

Per avere uno stub radice di tipo broadcast verso un modulo di nodo, poiché
l'unico caso d'uso è quello della funzione di radar del modulo `Neighborhood`,
si chiama il metodo `get_stub_whole_node_broadcast_for_radar` passando
l'interfaccia di rete gestita, cioè l'istanza
di `N.N.INeighborhoodNetworkInterface`.  
Il metodo prepara i dati serializzabili che sono previsti dal protocollo
ZCD, cioè:

*   Un `WholeNodeSourceID` che rappresenta il mittente.  
    Il `NeighborhoodNodeID`, che serve per crearlo, viene preso dalla variabile
    globale `skeleton_factory`.
*   Un `EveryWholeNodeBroadcastID` che rappresenta il destinatario, che è chiunque
    in questo caso.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente)
    usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) viene preso dal
    membro `mac` dell'istanza di `N.N.INeighborhoodNetworkInterface` passata
    al metodo.

Poi il metodo usa la libreria `ntkdrpc` basata su ZCD per ottenere uno stub
radice di tipo datagram sul medium system; cioè chiama la funzione
`get_addr_datagram_system`.  
Per comporre la stringa `send_pathname` prescritta da `ntkdrpc`, la classe
`N.StubFactory` usa l'identificativo univoco del processo (che è memorizzato
nella variabile globale `pid` nel file `main.vala`) e il nome della
pseudo interfaccia di rete, che viene preso dal membro `dev` dell'istanza di
`N.N.INeighborhoodNetworkInterface` passata al metodo.

La presente testsuite non necessiterà mai di uno stub per inviare messaggi
di tipo unicast.

* * *

Diverse classi (una o più per ogni modulo) realizzano gli stub dedicati.  
Vedere le classi nel file `rpc/module_stubs.vala`.

La classe `N.NeighborhoodManagerStubHolder` implementa l'interfaccia
`N.INeighborhoodManagerStub` a partire da uno stub radice.

La classe `N.IdentityManagerStubHolder` implementa l'interfaccia
`N.IIdentityManagerStub` a partire da uno stub radice.

La classe `N.QspnManagerStubHolder` implementa l'interfaccia
`N.IQspnManagerStub` a partire da uno stub radice.  
In questa testsuite non sarà mai usata, poiché non si realizzano archi.

La classe `N.QspnManagerStubBroadcastHolder` implementa l'interfaccia
`N.IQspnManagerStub` a partire da una lista di stub radice.  
In questa testsuite non sarà mai usata, poiché non si realizzano archi.

La classe `N.QspnManagerStubVoid` implementa l'interfaccia
`N.IQspnManagerStub` semplicemente ignorando i messaggi che gli sono passati.
Infatti viene restituita al modulo `Qspn` quando questi chiama il metodo
`i_qspn_get_broadcast` passando una lista di archi vuota.

**TODO** altri moduli

#### Argomenti al programma

Si può passare la topologia della rete (il default è 1,1,1,2). **TODO**  
Si può passare il primo indirizzo Netsukuku della prima identità. **TODO**  
Si deve passare un PID per identificare un processo che simula un nodo in una
testsuite.  
Si deve passare una lista di pseudo-interfacce di rete da gestire.  
Si può passare una lista di compiti (tasks) da fare in un dato momento. **TODO**  
Si può passare il livello (di default 0) della sottorete a gestione autonoma. **TODO**  
Si può passare un flag per far sì che il nodo accetti richieste anonime dirette
a lui. **TODO**  
Si può passare un flag per far sì che il nodo rifiuti di passare richieste
anonime mascherandole. **TODO**  

#### Identità e archi-identità

Nella classe `N.IdentityData` memorizziamo le informazioni relative a ogni identità
assunta dal nodo corrente: una principale e 0 o più di connettività.  
Fra queste informazioni ci sono i relativi archi-identità, rappresentati da istanze
della classe `N.IdentityArc`.  
Entrambe le classi sono nel file `main.vala`.

Anche il modulo `Identities` ha fra i suoi compiti il mantenimento delle info relative
ad ogni identità, ma usiamo anche questa classe per alcuni motivi:

*   praticità nel ciclare fra le identità presenti; e.g. **TODO**
*   memorizzazione di alcuni dati temporanei nelle fasi di aggiornamento delle
    informazioni; e.g. **TODO**
*   ...

Ogni istanza di `N.IdentityData` quando viene creata riceve un `NodeID` e un
identificativo locale numerico progressivo chiamato `local_identity_index`.

Tutte le istanze che vengono create sono immediatamente messe nell hashmap
`local_identities` indicizzate per il valore intero che costituisce il `NodeID`
assegnato all'identità (dal modulo `Identities`).

Il metodo helper `create_local_identity` nel file `main.vala` è usato per creare
una istanza e metterla nel hashmap dopo aver verificato che non era già stata creata.

Il metodo helper `find_local_identity` nel file `main.vala` è usato per cercare una
istanza di cui si conosce il `NodeID`. Se esiste la restituisce altrimenti
restituisce *null*.

Il metodo helper `find_local_identity_by_index` nel file `main.vala` è usato per
cercare una istanza di cui si conosce il `local_identity_index`. Se esiste la
restituisce altrimenti restituisce *null*.

Il metodo helper `remove_local_identity` nel file `main.vala` è usato per rimuovere
una istanza dal hashmap dopo aver verificato che era in effetti presente.

Ogni istanza di `N.IdentityData` memorizza una lista di archi-identità nel membro
`identity_arcs`, rappresentati da istanze di `IdentityArc`.

Ogni istanza di `N.IdentityData` può anche memorizzare:

*   L'indirizzo Netsukuku come istanza `Naddr my_naddr`.
*   Il fingerprint come istanza `Fingerprint my_fp`.
*   I livelli come "identità di connettività" `int connectivity_from_level` e
    `int connectivity_to_level`.
*   L'identità da cui è derivata `weak IdentityData? copy_of_identity`.
*   L'istanza `QspnManager qspn_mgr`.
*   Un getter `bool main_id` dice se è la principale confrontando se stessa con la
    variabile globale `IdentityData main_identity_data` sempre nel file `main.vala`.

Alcuni metodi pubblici della classe `N.IdentityData` sono usati per registrare gli
handler dei segnali che produce il modulo Qspn. Le implementazioni sono comunque
funzioni esterne alla classe, per poterle raggruppare nel file `qspn_signals.vala`.

Questi sono:

*  `per_identity_qspn_arc_removed`,
*  `per_identity_qspn_changed_fp`,
*  `per_identity_qspn_changed_nodes_inside`,
*  `per_identity_qspn_destination_added`,
*  `per_identity_qspn_destination_removed`,
*  `per_identity_qspn_gnode_splitted`,
*  `per_identity_qspn_path_added`,
*  `per_identity_qspn_path_changed`,
*  `per_identity_qspn_path_removed`,
*  `per_identity_qspn_presence_notified`,
*  `per_identity_qspn_qspn_bootstrap_complete`,
*  `per_identity_qspn_remove_identity`.

* * *

Ogni istanza di `N.IdentityArc` memorizza immediatamente:

*   l'identità a cui è associata (come indice
    `int local_identity_index`)
*   l'arco su cui poggia (come passato al modulo Identites, cioè `IIdmgmtArc arc`)
*   l'arco identità stesso (come ottenuto dal modulo Identites, cioè
    `IIdmgmtIdentityArc id_arc`)

L'identità è memorizzata in un `IdentityArc` come indice `int local_identity_index`.
Questa classe ha un getter per risalire alla istanza
di `IdentityData` che se viene invocato su una identità che non è più nell hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Ogni istanza può anche memorizzare:

*   `string peer_mac`
*   `string peer_linklocal`
*   `QspnArc? qspn_arc`

#### Routine main

Operazioni nella funzione `main` del file `main.vala`.

##### Operazioni iniziali

Per prima cosa si fa il parsing delle opzioni date sulla linea di comando.

La topologia fa valorizzare
la variabile globale `ArrayList<int> gsizes`,
la variabile globale `ArrayList<int> g_exp`,
la variabile globale `int levels`
e la variabile globale `ArrayList<int> hooking_epsilon`,
nel file `main.vala`.  
Se viene passato il primo indirizzo Netsukuku della prima identità, con questo
viene subito valorizzato `ArrayList<int> naddr` che è locale nella funzione `main`;
altrimenti resta per ora vuoto e verrà valorizzato in modo casuale dopo
l'inizializzazione del generatore `PRNGen`.  
Il PID finisce nella variabile globale `int pid` nel file `main.vala`.  
La lista di interfacce finisce in `ArrayList<string> devs` che è locale nella
funzione `main`.  
Il livello della sottorete a gestione autonoma finisce nella variabile globale
`int subnetlevel`.  
Il flag di accettazione richieste anonime finisce nella variabile globale
`bool accept_anonymous_requests`.  
Il flag di rifiuto di mascheramento finisce nella variabile globale
`bool no_anonymize`.

Dopo si inizializza lo scheduler delle tasklet. Va nella variabile globale
`ITasklet tasklet`.

Dopo si inizializzano i singoli moduli (principalmente perché hanno al loro
interno delle classi serializzabili da registrare).  
I moduli si inizializzano con il metodo statico `init` delle classi `NeighborhoodManager`,
`IdentityManager`, `QspnManager`, ...

In particolare l'inizializzazione del modulo `QspnManager` serve anche a impostare
dei parametri comuni a tutte le istanze di `QspnManager` (che saranno più di una nel
singolo nodo). Questi sono stati memorizzati come costanti `max_paths`,
`max_common_hops_ratio`, `arc_timeout`.

Dopo si registrano le classi serializzabili che servono direttamente al programma.
Queste sono:  
`WholeNodeSourceID`, `WholeNodeUnicastID`, `EveryWholeNodeBroadcastID`, `NeighbourSrcNic`,
`IdentityAwareSourceID`, `IdentityAwareUnicastID`, `IdentityAwareBroadcastID`,
`Naddr`, `Fingerprint`, `Cost`, ...

Dopo si inizializza il generatore di numeri pseudo-casuali. Si usa come seed il PID.  
Questa operazione si fa con il metodo statico `init_rngen` delle classi `PRNGen`,
`NeighborhoodManager`,
`IdentityManager`, `QspnManager`, ...

Dopo, se non era valorizzato `naddr`, il primo indirizzo Netsukuku della prima identità,
viene valorizzato in modo casuale nel range imposto dalla topologia.

Dopo si inizializza un FakeCommandDispatcher. Va nella variabile globale
`FakeCommandDispatcher fake_cm`.

Dopo si passa lo scheduler delle tasklet alla libreria `ntkdrpc`, chiamandone la funzione
statica `init_tasklet_system`.

Dopo si istanzia lo `skeleton_factory`.

Dopo si istanzia lo `stub_factory`.

I primi due comandi, dati in blocco, sono qui:

```shell script
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
```

Dopo si istanzia il NeighborhoodManager nella variabile globale `neighborhood_mgr`.
Esso è unico.

Subito dal `neighborhood_mgr` si ottiene il `neighborhood_id` con cui si valorizza il
membro `whole_node_id` di `skeleton_factory`.

Dopo si connettono i segnali del NeighborhoodManager.

Dopo si prepara una hashmap (var globale) `handlednic_map` per le interfacce di rete
da gestire, rappresentate da istanze di `N.HandledNic`.
Delle stesse interfacce di rete si prepara anche una hashmap (var globale)
`pseudonic_map` rappresentate da istanze di `N.PseudoNetworkInterface`.

Delle stesse interfacce di rete si prepara anche una lista (var locale)
`if_list_dev` per i nomi, una lista `if_list_mac` per i MAC e una lista
`if_list_linklocal` per gli indirizzi link-local: queste liste
serviranno nella costruzione del IdentityManager.

Per ogni pseudo-interfaccia di rete da gestire si eseguono questi compiti:

*   Si genera pseudo-random un finto MAC.
*   Si memorizza una nuova istanza di `N.PseudoNetworkInterface` in `pseudonic_map`.
*   Si eseguono dei comandi in blocco per inizializzare la scheda di rete:
    ```shell script
    sysctl net.ipv4.conf.${dev}.rp_filter=0
    sysctl net.ipv4.conf.${dev}.arp_ignore=1
    sysctl net.ipv4.conf.${dev}.arp_announce=2
    ip link set dev ${dev} up
    ```
*   Si avvia una tasklet di ascolto per i messaggi datagram broadcast ricevuti da
    questa interfaccia di rete (con `skeleton_factory.start_datagram_system_listen`).
*   Si avvia `neighborhood_mgr.start_monitor` con l'istanza
    di `INeighborhoodNetworkInterface nic` memorizzata nella nuova istanza
    di `N.PseudoNetworkInterface`.  
    Sulla chiamata di questo metodo, immediatamente il `neighborhood_mgr` genera
    un IP linklocal e lo assegna a questa *NIC* e emette il
    segnale `nic_address_set`. Di conseguenza (come detto nella sezione sulla
    gestione dei segnali) viene creata la relativa istanza di `N.HandledNic`
    nella `handlednic_map` e viene valorizzato il campo `linklocal`.
*   Si memorizzano i dati della pseudo-interfaccia di rete nelle
    liste `if_list_dev`, `if_list_mac` e `if_list_linklocal`.

Dopo si istanzia il IdentityManager nella variabile globale `identity_mgr`.
Esso è unico.

Dopo si connettono i segnali del IdentityManager.

Dopo si recupera dal `identity_mgr` l'identificativo `NodeID` della prima identità
di questo nodo con il metodo `get_main_id`; si usa la variabile globale
`next_local_identity_index` per assegnargli un indice (progressivo);
si usa l'identificativo `NodeID` e l'indice per costruire una istanza di `IdentityData`
e si memorizza questa nel hashmap `local_identities`.  
Si memorizza temporaneamente questa istanza anche nella variabile locale `first_identity_data`.

Dopo si prepara nel membro `my_naddr` di `first_identity_data` una istanza di `N.Naddr` che
rappresenta il primo indirizzo Netsukuku della prima identità; e nel membro `my_fp` una istanza
di `N.Fingerprint` che rappresenta il fingerprint di questo nodo/identità/g-nodo di livello 0.  
Queste info vengono anche prodotte a video.

Dopo si crea la prima istanza di QspnManager nel membro `qspn_mgr` di `first_identity_data`.
Questa viene creata con il costruttore `create_net`, poiché la prima identità nel nodo viene
a formare una nuova rete. Oltre a passare l'indirizzo e il fingerprint, a questo costruttore
viene passata una nuova istanza di `N.QspnStubFactory` costruita sulla `first_identity_data`.

Dopo si connettono i segnali di questa istanza di QspnManager ai metodi della
istanza `first_identity_data`, di modo che il segnale possa essere associato alla
corretta identità nel nodo.

Dopo si attende che venga emesso e gestito il segnale di `bootstrap_complete` di questa
istanza di QspnManager; questo dovrebbe avvenire immediatamente.

Dopo si può eliminare il riferimento nella variabile globale `first_identity_data`.

Dopo si registra la funzione `safe_exit` come gestore dei segnali Posix di
terminazione (`INT` e `TERM`), per fare in modo di uscire dal loop in cui
adesso il programma entra.

##### Ciclo eventi

Dopo si entra nel main loop. In esso si gestiscono i vari segnali che i moduli
nelle diverse tasklet emettono. Si veda la sezione [gestione segnali]().

##### Operazioni finali

Alla terminazione del programma,
si rimuovono tutte le identità di connettività che sono presenti al momento nel nodo.  
Per ognuna di esse:  
si chiama il metodo `destroy` della relativa istanza di QspnManager;  
si sconnettono dai segnali di questa istanza di QspnManager i relativi gestori;  
si chiama il metodo `stop_operations` della relativa istanza di QspnManager;  
si rimuove l'istanza di IdentityData dalla hashmap `local_identities` con il
metodo helper `remove_local_identity`.

A questo punto **deve** essere presente la sola identità principale.  
Essa viene memorizzata temporaneamente nella variabile locale `last_identity_data`.  
Si chiama il metodo `destroy` della relativa istanza di QspnManager.  
Si sconnettono dai segnali di questa istanza di QspnManager i relativi gestori.  
Si chiama il metodo `stop_operations` della relativa istanza di QspnManager.  
Si rimuove l'istanza di IdentityData dalla hashmap `local_identities` con il
metodo helper `remove_local_identity`.  
Dopo si può eliminare il riferimento nella variabile globale `last_identity_data`.

Dopo si chiama `stop_rpc` su tutte le pseudo-interfacce di rete gestite.

Dopo si rimuove il NeighborhoodManager (dalla variabile globale).

Dopo si termina lo scheduler delle tasklet.

#### Gestione segnali dai moduli

Mettendosi in ascolto sui segnali emessi dai vari moduli l'applicazione coordina le funzionalità
di questi.

##### Segnali da NeighborhoodManager

Per reagire ai segnali emessi dal modulo `Neighborhood` sono implementate delle
funzioni nel file `neighborhood_signals.vala`.

Il segnale `nic_address_set` è gestito nella funzione `neighborhood_nic_address_set`.  
La funzione riceve un `N.N.INeighborhoodNetworkInterface` e l'indirizzo linklocal
che gli è stato appena assegnato.  
La funzione recupera l'istanza di `N.PseudoNetworkInterface` dal set `pseudonic_map`
e valorizza i suoi membri `linklocal` e `st_listen_pathname`.  
Inoltre crea una istanza di `N.HandledNic` valorizzando tutti i dati in essa contenuti,
tra i quali il `linklocal`, e la mette in `handlednic_map`.  
Inoltre avvia in questo momento la tasklet in ascolto per i messaggi stream unicast
ricevuti dall'indirizzo linklocal di cui sopra (con `skeleton_factory.start_stream_system_listen`).

Gli oggetti in `handlednic_map` servono al programma in quanto coordina le operazioni
dei vari moduli. Gli oggetti in `pseudonic_map` servono invece solo a questa specifica
testsuite in quanto deve fornire i parametri necessari ai metodi di ZCD a supporto
delle comunicazioni nel medium *system*.

Poiché questa testsuite non prevede la costituzione di archi,
non è necessario gestire i segnali `neighborhood_arc_*`.

Il segnale `arc_added` è gestito nella funzione `neighborhood_arc_added`.

Il segnale `arc_changed` è gestito nella funzione `neighborhood_arc_changed`.

Il segnale `arc_removing` è gestito nella funzione `neighborhood_arc_removing`.

Il segnale `arc_removed` è gestito nella funzione `neighborhood_arc_removed`.

Il segnale `nic_address_unset` è gestito nella funzione
`neighborhood_nic_address_unset`. Al momento non fa nulla eccetto scrivere
delle informazioni.

##### Segnali da IdentityManager

Per reagire ai segnali emessi dal modulo `Identities` sono implementate delle
funzioni nel file `identities_signals.vala`.

Poiché questa testsuite non prevede la costituzione di archi,
non è necessario gestire i segnali `identity_arc_*`, né il segnale
`arc_removed`.

Il segnale `identity_arc_added` è gestito nella funzione
`identities_identity_arc_added`.

Il segnale `identity_arc_changed` è gestito nella funzione
`identities_identity_arc_changed`.

Il segnale `identity_arc_removing` è gestito nella funzione
`identities_identity_arc_removing`.

Il segnale `identity_arc_removed` è gestito nella funzione
`identities_identity_arc_removed`.

Il segnale `arc_removed` è gestito nella funzione `identities_arc_removed`.

##### Segnali da QspnManager

Per reagire ai segnali emessi dal modulo `Qspn` sono implementate delle
funzioni nel file `qspn_signals.vala`.

Essi sono:
`per_identity_qspn_arc_removed`,
`per_identity_qspn_changed_fp`,
`per_identity_qspn_changed_nodes_inside`,
`per_identity_qspn_destination_added`,
`per_identity_qspn_destination_removed`,
`per_identity_qspn_gnode_splitted`,
`per_identity_qspn_path_added`,
`per_identity_qspn_path_changed`,
`per_identity_qspn_path_removed`,
`per_identity_qspn_presence_notified`,
`per_identity_qspn_qspn_bootstrap_complete`,
`per_identity_qspn_remove_identity`.

**TODO**

#### Integrazione modulo Neighborhood

Bisogna integrare l'unica istanza di `N.N.NeighborhoodManager` che si crea all'avio
del programma e muore alla sua terminazione.

Alcune classi dovranno essere prodotte dal programma per implementare le interfacce
definite nel modulo `Neighborhood`. Tali classi sono definite nel file
`neighborhood_helpers.vala`.

I segnali emessi dal modulo `Neighborhood` saranno gestiti da funzioni che sono definite
nel file `neighborhood_signals.vala`.

* * *

L'interfaccia `N.N.INeighborhoodIPRouteManager` è implementata nella classe
`N.NeighborhoodIPRouteManager`.

Quando il modulo `Neighborhood` chiama `add_address` la classe suddetta deve
aggiungere l'indirizzo IP passato (si tratterà di un linklocal) alla interfaccia di
rete (reale) passata.  
Lo fa chiedendo al *commander* di eseguire un singolo comando "`ip address add ...`".

Quando il modulo `Neighborhood` chiama `add_neighbor` la classe suddetta deve
aggiungere una route diretta all'indirizzo IP passato (si tratterà di un linklocal)
tramite l'interfaccia di rete (reale) passata.  
Lo fa chiedendo al *commander* di eseguire un singolo comando "`ip route add ...`".

Quando il modulo `Neighborhood` chiama `remove_neighbor` la classe suddetta deve
rimuovere una route diretta all'indirizzo IP passato (si tratterà di un linklocal)
tramite l'interfaccia di rete (reale) passata.  
Lo fa chiedendo al *commander* di eseguire un singolo comando "`ip route del ...`".

Quando il modulo `Neighborhood` chiama `remove_address` la classe suddetta deve
rimuovere l'indirizzo IP passato (si tratterà di un linklocal) alla interfaccia di
rete (reale) passata.  
Lo fa chiedendo al *commander* di eseguire un singolo comando "`ip address del ...`".

* * *

L'interfaccia `N.N.INeighborhoodStubFactory` è implementata nella classe
`N.NeighborhoodStubFactory`.

Quando il modulo `Neighborhood` chiama `get_broadcast_for_radar` la classe suddetta deve
restituire uno stub per mandare messaggi broadcast allo stesso modulo `Neighborhood` nei
nodi diretti vicini attraverso una specifica interfaccia di rete.  
Lo fa chiamando sulla classe `N.StubFactory` il metodo
`get_stub_whole_node_broadcast_for_radar` che gli ottiene una istanza di stub per
messaggi broadcast di nodo. Lo stub ottenuto è radice (si veda `AddressManager addr`
nel file `interfaces.rpcidl` del pacchetto `ntkdrpc`). Per ottenere uno stub dedicato
al modulo `Neighborhood` la classe crea una istanza di `NeighborhoodManagerStubHolder`.

Quando il modulo `Neighborhood` chiama `get_unicast` la classe suddetta deve
restituire uno stub per mandare messaggi unicast allo stesso modulo `Neighborhood`
in uno specifico nodo diretto vicino attraverso uno specifico
arco `N.N.INeighborhoodArc`.  
Questo non si verifica mai in questa testsuite, quindi il codice semplicemente
va in errore.

* * *

L'interfaccia `N.N.INeighborhoodQueryCallerInfo` è implementata nella classe
`N.NeighborhoodQueryCallerInfo`.

Quando il modulo `Neighborhood` chiama `is_from_broadcast` la classe suddetta deve
esaminare una istanza di `CallerInfo` (passata da ZCD ai metodi remoti quando riceve
una chiamata da un altro nodo) e stabilire se il metodo remoto è stato chiamato da un
diretto vicino in modalità broadcast; e in quel caso deve restituire l'istanza di
`N.N.INeighborhoodNetworkInterface` associata all'interfaccia di rete da cui il
messaggio è stato ricevuto.  
Lo fa chiamando sulla classe `N.SkeletonFactory` il metodo `from_caller_get_mydev`
che gli restituisce la stringa con il nome dell'interfaccia da cui è stato ricevuto
il messaggio. Tramite questo nome e l'hashmap `pseudonic_map` risale all'istanza
di `N.PseudoNetworkInterface` e in essa trova l'istanza
di `N.N.INeighborhoodNetworkInterface` che deve restituire.

Quando il modulo `Neighborhood` chiama `is_from_unicast` la classe suddetta deve
esaminare una istanza di `CallerInfo` e una lista di archi `N.N.INeighborhoodArc`
che gli sono passati e stabilire se il metodo remoto è stato chiamato da un diretto
vicino in modalità unicast; e in quel caso deve restituire l'arco
`N.N.INeighborhoodArc` da cui il messaggio è stato ricevuto.  
Il nodo non crea nessuna arco in questa testsuite, quindi il codice semplicemente
restituisce *null*.

* * *

L'interfaccia `N.N.INeighborhoodNetworkInterface` è implementata nella classe
`N.NeighborhoodNetworkInterface`.

Le istanze di questa classe sono da passare al modulo `Neighborhood` a rappresentare
una interfaccia di rete da gestire. Una istanza è creata passando una istanza di
`N.PseudoNetworkInterface`. A sua volta una istanza di `N.PseudoNetworkInterface`
contiene un riferimento (**TODO** mettere weak) alla relativa istanza
di `N.N.INeighborhoodNetworkInterface`.

Quando il modulo `Neighborhood` chiama `measure_rtt` la classe suddetta deve misurare
il Round Trip Time delle comunicazioni tramite questa interfaccia di rete con un
indirizzo IP linklocal che identifica una precisa interfaccia di rete di un nodo
diretto vicino.  
Questo avverrà eseguendo il comando `ping` e interpretandone l'output. Per il momento
invece, questo programma di test chiede al *commander* di eseguire un singolo
comando "`ping`" e restituisce una misurazione fake di 1000 usec.

#### Integrazione modulo Identities

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

#### Integrazione modulo Qspn

Bisogna integrare le istanze di `N.Q.QspnManager` che si creano al momento che nasce
una nuova identità e muoiono alla sua terminazione.

Alcune classi dovranno essere prodotte dal programma per implementare le interfacce
definite nel modulo `Qspn`. Tali classi sono definite nel file
`qspn_helpers.vala`.

I segnali emessi dal modulo `Qspn` saranno gestiti da funzioni che sono definite
nel file `qspn_signals.vala`.

* * *

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
passando una istanza di `IdentityArc`.  
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

#### Serializzabili

Nel file `serializables.vala` sono implementate alcune classi serializzabili.  
Molte classi serializzabili usate dai vari moduli sono implementate all'interno dei
moduli stessi. Ma alcune devono essere implementate nel programma.

##### richieste da ntkdrpc

La classe `N.WholeNodeSourceID` implementa l'interfaccia `N.ISourceID`.
Contiene una istanza di `NeighborhoodNodeID id` che rappresenta il
mittente.

La classe `N.WholeNodeUnicastID` implementa l'interfaccia `N.IUnicastID`.
Contiene una istanza di `NeighborhoodNodeID neighbour_id` che rappresenta
il vicino destinatario.

La classe `N.EveryWholeNodeBroadcastID` implementa l'interfaccia `N.IBroadcastID`.
Essa non contiene dati; rappresenta infatti chiunque riceva
il messaggio.

La classe `N.NeighbourSrcNic` implementa l'interfaccia `N.ISrcNic`.
Contiene una istanza di `string mac` che identifica la specifica
interfaccia di rete del mittente.

La classe `N.IdentityAwareSourceID` implementa l'interfaccia `N.ISourceID`.
Contiene una istanza di `NodeID id` che rappresenta il mittente.

La classe `N.IdentityAwareUnicastID` implementa l'interfaccia `N.IUnicastID`.
Contiene una istanza di `NodeID id` che rappresenta il vicino destinatario.

La classe `N.IdentityAwareBroadcastID` implementa l'interfaccia `N.IBroadcastID`.
Contiene una lista `List<NodeID> id_set` che identifica i destinatari.

##### richieste da Qspn

La classe `N.Naddr` implementa l'interfaccia `N.Q.IQspnMyNaddr`.

La classe `N.Cost` implementa l'interfaccia `N.Q.IQspnCost`.

La classe `N.Fingerprint` implementa l'interfaccia `N.Q.IQspnFingerprint`.

#### Altri aspetti (riorganizzare)

Bisogna poter dare i comandi uno alla volta. Inoltre bisogna avere la possibilità
di dare una serie di comandi senza permettere che altre tasklet possano introdurne
altri in mezzo a questi.  
Questo compito è affidato ad un oggetto detto *commander*.  
Vedere il file `commander.vala` e `fake_command_dispatcher.vala`.
