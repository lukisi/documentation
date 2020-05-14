# Ciclo di vita dei moduli

## Sequenza di creazione iniziale

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

Poi il programma associa alla prima identità del nodo una istanza di `QspnManager`
che è creata in modalità `create_net`. In questa modalità viene creata dal modulo `Qspn`
una nuova rete composta dal solo nodo.  
*Nota:* le altre modalità sono `enter_net` e `migration`.

Ora il programma dovrà associare alla prima identità del nodo una istanza di `PeersManager`.
Questa è necessaria per poi associare alla prima identità del nodo una istanza
di `CoordinatorManager`. Infine questa è necessaria per associare alla prima identità del
nodo una istanza di `HookingManager`.  
Va notato che le prime istanze di `PeersManager` e`CoordinatorManager` si troveranno a
operare in un contesto molto "semplice" essendo la prima identità del nodo unico membro
di una nuova rete. La realizzazione di una testsuite per queste due prime istanze dovrebbe
risultare immediata.



# appunti vecchi

## Istanze di classi a regime

A regime, ci sarà una istanza di `NeighborhoodManager`. Questa si crea all'avio del programma
e muore alla sua terminazione.

A regime, ci sarà una istanza di `IdentityManager`. Questa si crea all'avvio del programma
e muore alla sua terminazione.

A regime, ci saranno *n* istanze di `QspnManager`. Una per l'identità principale e una per ogni
identità di connettività.  
La prima si crea all'avvio del programma. Una istanza "di connettività" può morire quando non serve
più la relativa identità di connettività. L'istanza "principale" ultima muore alla terminazione del programma.

A regime, ci saranno *n* istanze di `PeerManager`.

## Cosa serve alla creazione della sola istanza di NeighborhoodManager

`int max_arcs` - Un limite massimo al numero di archi che il modulo crea con i nodi diretti vicini.

`INeighborhoodStubFactory stub_factory` - Una factory per creare stub RPC.

`INeighborhoodQueryCallerInfo query_caller_info` - Un discriminatore per gli oggetti CallerInfo che si ricevono nelle
chiamate di metodi remoti.

`INeighborhoodIPRouteManager ip_mgr` - Per chiamare i comandi iproute necessari.

`NewLinklocalAddress new_linklocal_address` - Un delegato per scegliere un indirizzo IP linklocal per le proprie schede di rete.

## Cosa serve alla creazione della sola istanza di IdentityManager

`List<string> if_list_dev` - Lista dei nomi delle proprie schede di rete.

`List<string> if_list_mac` - Lista dei MAC delle proprie schede di rete.

`List<string> if_list_linklocal` - Lista degli indirizzi IP linklocal assegnati alle proprie schede di rete.

`IIdmgmtNetnsManager netns_manager` - Per chiamare i comandi iproute necessari alla creazione e gestione di network namespace per
le diverse identità di connettività.

`IIdmgmtStubFactory stub_factory` - Una factory per creare stub RPC e un discriminatore per gli oggetti CallerInfo
che si ricevono nelle chiamate di metodi remoti.

`NewLinklocalAddress new_linklocal_address` - Un delegato per scegliere un indirizzo IP linklocal per le proprie
pseudo-schede di rete nei network namespace per le diverse identità di connettività.

## Cosa serve alla inizializzazione del modulo Qspn

`int max_paths` - Un limite massimo al numero di percorsi memorizzati dal modulo per ogni destinazione.

`double max_common_hops_ratio` - Un coefficiente di disgiunzione dei percorsi.

`int arc_timeout` - Il tempo di attesa perché il modulo Qspn nel nodo diretto vicino riconosca l'arco.

`IQspnThresholdCalculator threshold_calculator` - Un delegato che decida quanto tempo attendere per segnalare lo split
di un g-nodo, dati i due percorsi dai quali arrivano informazioni discordanti sul fingerprint dello stesso.

## Cosa serve alla creazione della prima istanza di QspnManager

`IQspnMyNaddr my_naddr` - Il proprio indirizzo Netsukuku.

`IQspnFingerprint my_fingerprint` - Il proprio fingerprint a livello di nodo.

`IQspnStubFactory stub_factory` - Una factory per creare stub RPC.

## Cosa serve alla creazione di una nuova istanza di QspnManager per migrazione

## Cosa serve alla creazione di una nuova istanza di QspnManager per ingresso

