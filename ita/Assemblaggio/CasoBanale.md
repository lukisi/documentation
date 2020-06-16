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

### Comunicazioni in rete

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

Le parti dell'implementazione che riguardano le comunicazioni in rete sono trattate
nel documento [ComunicazioniRete](CasoBanaleImplementazioneTestsuite/ComunicazioniRete.md).

### Identità e archi-identità

Le parti dell'implementazione che riguardano la memorizzazione delle informazioni
riguardanti le identità assunte dal nodo e i relativi archi-identità sono trattate
nel documento [ClassiIdentita](CasoBanaleImplementazioneTestsuite/ClassiIdentita.md).

### Routine main

Analizziamo la funzione `main` del file `main.vala`.

#### Argomenti al programma

Per prima cosa si fa il parsing delle opzioni date sulla linea di comando.

Si può passare la topologia della rete (il default è 1,1,1,2).  
Si può passare il primo indirizzo Netsukuku della prima identità.  
Si deve passare un PID per identificare un processo che simula un nodo in una
testsuite.  
Si deve passare una lista di pseudo-interfacce di rete da gestire.  
Si può passare il livello (di default 0) della sottorete a gestione autonoma.  
Si può passare un flag per far sì che il nodo accetti richieste anonime dirette
a lui.  
Si può passare un flag per far sì che il nodo rifiuti di passare richieste
anonime mascherandole.

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

#### Operazioni iniziali

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

Dopo si inizializza il *commander* e il *tn*.  
In particolare il commander è un FakeCommandDispatcher. Va nella variabile globale
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
    ip link set dev ${dev} address ${MAC}
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
di questo nodo con il metodo `get_main_id`; si chiama la funzione
`find_or_create_local_identity` per costruire una istanza di `IdentityData`
e memorizzarla nel hashmap `local_identities` con il prossimo indice
univoco progressivo.  
Si memorizza questa identità anche nella variabile globale `main_identity_data`
poiché è l'identità principale.  
Si memorizza temporaneamente questa istanza anche nella variabile locale
`first_identity_data`.

Dopo si prepara nel membro `my_naddr` di `first_identity_data` una istanza di `N.Naddr` che
rappresenta il primo indirizzo Netsukuku della prima identità; e nel membro `my_fp` una istanza
di `N.Fingerprint` che rappresenta il fingerprint di questo nodo/identità/g-nodo di livello 0.  
Queste info vengono anche prodotte a video.

Dopo si chiamano alcune funzioni di IpCompute e di IpCommands:

*   si chiama `IpCompute.new_main_id` per la prima identità;
*   si chiama `IpCompute.new_id` per la prima identità;
*   si chiama `IpCommands.main_start` per la prima identità.

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

#### Ciclo eventi

Dopo si entra nel main loop. In esso si gestiscono i vari segnali che i moduli
nelle diverse tasklet emettono. Si veda la sezione [gestione segnali]().

#### Operazioni finali

Alla terminazione del programma,
si rimuovono tutte le identità di connettività che sono presenti al momento nel nodo.  
Per ognuna di esse alcune operazioni sono state implementate e sono qui di seguito
indicate; tuttavia nella presente testsuite non si creano identità di connettività.

*   Si chiama il metodo `destroy` della relativa istanza di QspnManager.  
*   Si sconnettono dai segnali di questa istanza di QspnManager i relativi gestori.  
*   Si chiama il metodo `stop_operations` della relativa istanza di QspnManager.  
*   Si rimuove l'identità dalla gestione dell'IdentityManager.  
*   Si rimuove l'istanza di IdentityData dalla hashmap `local_identities` con il
    metodo helper `remove_local_identity`.
*   Si chiama `IpCommands.connectivity_stop`.

A questo punto **deve** essere presente la sola identità principale.  
Essa viene memorizzata temporaneamente nella variabile locale `last_identity_data`.  
Si chiama il metodo `destroy` della relativa istanza di QspnManager.  
Si sconnettono dai segnali di questa istanza di QspnManager i relativi gestori.  
Si chiama il metodo `stop_operations` della relativa istanza di QspnManager.  
Si chiama `IpCommands.main_stop`.  
Si rimuove l'istanza di IdentityData dalla hashmap `local_identities` con il
metodo helper `remove_local_identity`.  
Dopo si può eliminare il riferimento nella variabile globale `last_identity_data`.

Dopo si chiama `stop_rpc` su tutte le pseudo-interfacce di rete gestite.

Dopo si rimuove il NeighborhoodManager (dalla variabile globale).

Dopo si termina lo scheduler delle tasklet.

### Gestione segnali dai moduli

Mettendosi in ascolto sui segnali emessi dai vari moduli l'applicazione coordina le funzionalità
di questi.

#### Segnali da NeighborhoodManager

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

#### Segnali da IdentityManager

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

#### Segnali da QspnManager

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

Alla segnalazione di `bootstrap_complete` o di variazioni alle mappe
vengono eseguiti dei comandi `ip` per mezzo di chiamate a funzioni
nel namespace `UpdateGraph`.

**TODO**

### Classi per l'integrazione dei moduli

Il programma, per essere in grado di usare i vari moduli, deve fornire alcune classi
(dette classi helper) che implementino le interfacce definite in essi. Inoltre il
programma deve gestire i segnali emessi dai moduli.

#### Integrazione modulo Neighborhood

Ci sarà una sola istanza di `N.N.NeighborhoodManager` che si crea all'avio
del programma e muore alla sua terminazione.

Le classi helper per il modulo `Neighborhood` sono trattate nel documento
[HelperNeighborhood](CasoBanaleImplementazioneTestsuite/HelperNeighborhood.md).

I segnali emessi dal modulo `Neighborhood` saranno gestiti da funzioni che sono
definite nel file `neighborhood_signals.vala`.

#### Integrazione modulo Identities

Ci sarà una sola istanza di `N.I.IdentityManager` che si crea all'avio
del programma e muore alla sua terminazione.

Le classi helper per il modulo `Identities` sono trattate nel documento
[HelperIdentities](CasoBanaleImplementazioneTestsuite/HelperIdentities.md).

I segnali emessi dal modulo `Identities` saranno gestiti da funzioni che sono
definite nel file `identities_signals.vala`.

#### Integrazione modulo Qspn

Ci saranno diverse istanze di `N.Q.QspnManager` che si creano al momento che nasce
una nuova identità e muoiono alla sua terminazione.

Le classi helper per il modulo `Qspn` sono trattate nel documento
[HelperQspn](CasoBanaleImplementazioneTestsuite/HelperQspn.md).

I segnali emessi dal modulo `Qspn` saranno gestiti da funzioni che sono definite
nel file `qspn_signals.vala`.

### Serializzabili

Nel file `serializables.vala` sono implementate alcune classi serializzabili.  
Molte classi serializzabili usate dai vari moduli sono implementate all'interno dei
moduli stessi. Ma alcune devono essere implementate nel programma.

#### richieste da ntkdrpc

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

#### richieste da Qspn

La classe `N.Naddr` implementa l'interfaccia `N.Q.IQspnMyNaddr`.

La classe `N.Cost` implementa l'interfaccia `N.Q.IQspnCost`.

La classe `N.Fingerprint` implementa l'interfaccia `N.Q.IQspnFingerprint`.

### Altri aspetti (riorganizzare)

Bisogna poter dare i comandi uno alla volta. Inoltre bisogna avere la possibilità
di dare una serie di comandi senza permettere che altre tasklet possano introdurne
altri in mezzo a questi.  
Questo compito è affidato ad un oggetto detto *commander*.  
Vedere il file `commander.vala` e `fake_command_dispatcher.vala`.

I comandi `ip`, in particolare, vengono eseguiti usando il *commander* in
collaborazione con un oggetto `TableNames tn`.  
Vedere il file `table_names.vala`.

***

Esistono i file `enter_network.vala` e `migrate.vala`
che per il momento non servono in questa testsuite.

Sono lasciati per riferimento futuro, ma non sono inclusi nel Makefile.

***

Ci sono delle funzioni nel namespace `Netsukuku.IPCompute`
attraverso le quali per una identità nel nodo corrente vengono computati gli
indirizzi IP pubblici propri e delle possibili destinazioni da mettere nelle
tabelle di routing. La logica dietro queste funzioni si può comprendere leggendo il
documento [IndirizziIP](../DemoneNTKD/IndirizziIP.md).

Ci sono delle funzioni nel namespace `Netsukuku.IpCommands`
attraverso le quali per una identità nel nodo corrente vengono dati i vari
comandi `ip` che servono a inizializzare o aggiornare le tabelle di routing.

Ci sono delle funzioni nel namespace `Netsukuku.UpdateGraph`
attraverso le quali per una identità nel nodo corrente vengono dati i vari
comandi `ip` che servono ad aggiornare le tabelle di routing per variazioni
alle mappe.

Quando si entra in una rete esistente (una nuova identità nasce e una
vecchia identità diventa di connettività) nelle funzioni del file enter_network.vala:

*   Si chiamano alcune funzioni di IpCompute:
    *   si chiama IpCompute.new_main_id per la nuova identità (se main)
    *   si chiama IpCompute.new_id per la nuova identità
    *   si chiama IpCompute.gone_connectivity_id per la vecchia identità
*   Si chiamano alcune funzioni di IpCommands:
    *   si chiama IpCommands.gone_connectivity per la vecchia identità
    *   si chiama IpCommands.main_dup oppure IpCommands.connectivity_dup per la
        nuova identità
    *   dopo un bel po' si chiama IpCommands.connectivity_stop per la vecchia
        identità

Quando il QspnManager (di una identità) segnala bootstrapcomplete o variazioni
alle mappe, nelle funzioni del file qspn_signals.vala:

*   Si chiama UpdateGraph.update_destination:
    *   Questa chiama IpCommands.map_update.

