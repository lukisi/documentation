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
operare in un contesto molto "semplice" essendo la prima identità del nodo unico membro
di una nuova rete. I servizi P2P saranno sempre forniti dal nodo stesso senza nessuna
reale comunicazione di rete. Il nodo stesso sarà il *Coordinator* a tutti i livelli.


## Implementazione testsuite locale

Implementiamo un programma che possiamo usare per verificare le operazioni che farebbe
un nodo in questo semplice scenario.

Useremo il supporto del medium `system` fornito dal framework ZCD.  
Il nome dell'eseguibile sarà `sys_ntkd_alone`. La parte `sys` indica che usa il medium
system, cioè socket di sistema per comunicazioni locali tra processi. La parte `alone`
indica che vogliamo provare questo semplice scenario.

### Appunti dell'implementazione

I file sorgente sono nella dir `sys-ntkd-alone`.

Il nome del pacchetto (configure.ac) è `sys-ntkd-alone`.

Il nome dell'eseguibile sarà `sys_ntkd_alone`.

La routine main è nel file `main.vala`.

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
di tipo stream trasmessi in unicast a uno specifico indirizzo IP. In questa
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
di `N.SkeletonFactory` passando il CallerInfo.

**TODO** completare.

* * *

In una unica classe viene implementato il codice per ottenere degli stub
radice di vario tipo (broadcast, unicast, di nodo o d'identità).  
Vedere la classe `N.StubFactory` nel file `rpc/stub_factory.vala`.

La classe è pensata per avere una sola istanza in una variabile globale
`stub_factory` nel file `main.vala`, a uso comune di tutto il codice.

**TODO** completare.


#### Altri aspetti (riorganizzare)

Bisogna poter dare i comandi uno alla volta. Inoltre bisogna avere la possibilità
di dare una serie di comandi senza permettere che altre tasklet possano introdurne
altri in mezzo a questi.  
Vedere il file `commander.vala` e `fake_command_dispatcher.vala`.

* * *

Bisogna integrare l'unica istanza di `N.N.NeighborhoodManager` che si crea all'avio
del programma e muore alla sua terminazione.
