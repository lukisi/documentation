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
Vedere la classe `N.PseudoNetworkInterface` nel file `main.vala`. I dati memorizzati
includono `dev`, `mac`, `linklocal`, `INeighborhoodNetworkInterface nic` e anche
informazioni legate al fatto che si tratta di pseudo-interfacce di rete simulate da
un socket di sistema: `send_pathname`, `listen_pathname`, `st_listen_pathname`.

In una hashmap memorizziamo le istanze di `N.PseudoNetworkInterface` di ogni
interfaccia di rete che il programma deve gestire, indicizzate per il nome
`dev`.  
Vedere la variabile globale `pseudonic_map` nel file `main.vala`.

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

Il metodo `get_dispatcher_set` dovrà restituire una lista di *skeleton* che saranno
chiamati a eseguire una procedura remota come effetto del messaggio rilevato.  
A seconda del tipo di CallerInfo, il metodo capisce se il messaggio è destinato
a una identità o a un nodo. Inoltre capisce se il messaggio è destinato a qualcuno
in questo sistema.

*Implementazione attuale:* il pacchetto ha sicuramente come `source_id` un WholeNodeSourceID
e come `broadcast_id` un EveryWholeNodeBroadcastID; altrimenti la tasklet che gestisce
il pacchetto termina con `abort_tasklet`. Il metodo restituisce una lista con un
unico *skeleton* che è il `NodeSkeleton node_skeleton`.

* * *

Il `NodeSkeleton node_skeleton` è uno *skeleton* per i metodi remoti di nodo. Ci sarà
una sola istanza che rappresenta il sistema. Ha il membro pubblico
`NeighborhoodNodeID id` che contiene il suo identificativo di nodo.

*Implementazione attuale:* l'unico manager che può restituire è il Neighborhood,
implementa cioè solo `neighborhood_manager_getter`.

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
nella variabile globale `pid` nel file `system_ntkd.vala`) e il nome della
pseudo interfaccia di rete, che viene preso dal membro `dev` dell'istanza di
`N.N.INeighborhoodNetworkInterface` passata al metodo.

La presente testsuite non necessiterà mai di uno stub per inviare messaggi
di tipo unicast.

* * *

Diverse classi (una o più per ogni modulo) realizzano gli stub dedicati.  
Vedere le classi nel file `rpc/module_stubs.vala`.

La classe `N.NeighborhoodManagerStubHolder` implementa l'interfaccia
`N.INeighborhoodManagerStub` a partire da uno stub radice.

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

#### Routine main

Operazioni nella funzione `main` del file `main.vala`.

##### Operazioni iniziali

Per prima cosa si fa il parsing delle opzioni date sulla linea di comando.

Il PID finisce nella variabile globale `int pid` nel file `main.vala`.  
La lista di interfacce finisce in `ArrayList<string> devs` che è locale nella
funzione `main`.  

Dopo si inizializza lo scheduler delle tasklet. Va nella variabile globale
`ITasklet tasklet`.

Dopo si inizializzano i singoli moduli (principalmente perché hanno al loro
interno delle classi serializzabili da registrare) e si registrano le classi
serializzabili che servono direttamente al programma.  
I moduli si inizializzano con il metodo statico `init` delle classi `NeighborhoodManager`,
`IdentityManager` **TODO**, `QspnManager` **TODO**, ...  
Le classi serializzabili da registrare direttamente in `main` sono:
`WholeNodeSourceID`, `WholeNodeUnicastID`, `EveryWholeNodeBroadcastID`, `NeighbourSrcNic`,
`IdentityAwareSourceID` **TODO**, `IdentityAwareUnicastID` **TODO**, `IdentityAwareBroadcastID` **TODO**,
`Naddr` **TODO**, `Fingerprint` **TODO**, `Cost` **TODO**, ...

Dopo si inizializza il generatore di numeri pseudo-casuali. Si usa come seed il PID.  
Questa operazione si fa con il metodo statico `init_rngen` delle classi `PRNGen`,
`NeighborhoodManager`,
`IdentityManager` **TODO**, `QspnManager` **TODO**, ...

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

Dopo si prepara una hashmap (var globale) `pseudonic_map` per le interfacce di rete da gestire
rappresentate da istanze di `N.PseudoNetworkInterface`.

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
    di `N.PseudoNetworkInterface`;  
    sulla chiamata di questo metodo, immediatamente il `neighborhood_mgr` genera
    un IP linklocal e lo assegna a questa *NIC* e emette il
    segnale `nic_address_set`. Di conseguenza... **TODO**
*   **TODO** Si memorizzano i dati della pseudo-interfaccia di rete nelle
    liste `if_list_dev`, `if_list_mac` e `if_list_linklocal`.

Dopo si registra la funzione `safe_exit` come gestore dei segnali Posix di
terminazione (`INT` e `TERM`), per fare in modo di uscire dal loop in cui
adesso il programma entra.

##### Ciclo eventi

Dopo si entra nel main loop. In esso si gestiscono i vari segnali che i moduli
nelle diverse tasklet emettono. Si veda la sezione [gestione segnali]().

##### Operazioni finali

Alla terminazione del programma,
**TODO** si rimuovono tutte le identità di connettività che sono presenti
al momento nel nodo...

**TODO** A questo punto **deve** essere presente la sola identità principale...

Dopo si chiama **TODO** `stop_rpc` su tutte le pseudo-interfacce di rete gestite.

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
**TODO** Inoltre crea una istanza di `N.HandledNic` valorizzando tutti i dati in essa contenuti
e la mette in `handlednic_map`.  
Inoltre avvia in questo momento la tasklet in ascolto per i messaggi stream unicast
ricevuti dall'indirizzo linklocal di cui sopra (con `skeleton_factory.start_stream_system_listen`).

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

#### Altri aspetti (riorganizzare)

Bisogna poter dare i comandi uno alla volta. Inoltre bisogna avere la possibilità
di dare una serie di comandi senza permettere che altre tasklet possano introdurne
altri in mezzo a questi.  
Questo compito è affidato ad un oggetto detto *commander*.  
Vedere il file `commander.vala` e `fake_command_dispatcher.vala`.

* * *

Nel file `serializables.vala` sono implementate le classi serializzabili che servono per comunicare
con il protocollo ZCD e i metodi definiti nel pacchetto `ntkdrpc`.

La classe `N.WholeNodeSourceID` implementa l'interfaccia `N.ISourceID` fornita da `ntkdrpc`. Contiene una
istanza di `NeighborhoodNodeID id` che rappresenta il mittente.

La classe `N.WholeNodeUnicastID` implementa l'interfaccia `N.IUnicastID` fornita da `ntkdrpc`. Contiene una
istanza di `NeighborhoodNodeID neighbour_id` che rappresenta il vicino destinatario.

La classe `N.EveryWholeNodeBroadcastID` implementa l'interfaccia `N.IBroadcastID` fornita da `ntkdrpc`. Essa
non contiene dati; rappresenta infatti chiunque riceva il messaggio.

La classe `N.NeighbourSrcNic` implementa l'interfaccia `N.ISrcNic` fornita da `ntkdrpc`. Contiene una
istanza di `string mac` che identifica la specifica interfaccia di rete del mittente.
