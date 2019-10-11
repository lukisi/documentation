## Ciclo di vita delle istanze di ogni modulo

A regime, ci sarà una istanza di NeighborhoodManager. Questa si crea all'avio del programma e muore alla sua terminazione.

A regime, ci sarà una istanza di IdentityManager. Questa si crea all'avvio del programma e muore alla sua terminazione.

A regime, ci saranno *n* istanze di QspnManager. Una per l'identità principale e una per ogni identità di connettività.
La prima si crea all'avvio del programma. Una istanza "di connettività" può morire quando non serve più la relativa identità
di connettività. L'istanza "principale" ultima muore alla terminazione del programma.

A regime, ci saranno *n* istanze di PeerManager.

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



### Annotazioni

Dopo la creazione della sola istanza di NeighborhoodManager, viene chiamato il suo metodo `start_monitor` per
iniziare a gestire una scheda di rete. Di conseguenza quasi immediatamente viene emesso il suo segnale `neighborhood_nic_address_set` e
il programma viene a conoscenza dell'indirizzo IP linklocal associato a quella scheda di rete.

Nella gestione di questo segnale, in `handlednic_map` viene memorizzata una istanza di `HandledNic` che mantiene questa info.

Poco dopo si ha la creazione della sola istanza di IdentityManager. Quindi il programma ha le informazioni che servono a
compilare non solo `if_list_dev` ma anche `if_list_mac` e `if_list_linklocal`.

* * *

Bisogna poter dare i comandi uno alla volta. E avere la possibilità di dare una serie di comandi
senza che si possano introdurre in mezzo altri comandi.  
Vedere il file `commander.vala` e `fake_command_dispatcher.vala`.

* * *

Bisogna potersi mettere in ascolto e inviare comunicazioni agli altri nodi:

*   Su una interfaccia di rete in broadcast.  
    Per mettersi in ascolto serve il nome dell'interfaccia di rete. Questo viene passato all'avvio del programma.
*   Su un indirizzo linklocal in unicast.  
    Per mettersi in ascolto serve l'indirizzo IP linklocal assegnato all'interfaccia di rete. Questo
    viene scelto dal `NeighborhoodManager` che lo notifica con un segnale.
*   Su un indirizzo routabile in unicast.  
    Per mettersi in ascolto serve l'indirizzo IP routabile assegnato al nodo. Questo è calcolato sulla base
    dell'indirizzo Netsukuku del nodo, il quale viene gestito dal modulo `qspn`.

...

* * *

In una classe memoriziamo le informazioni relative ad una interfaccia di rete che il programma deve gestire.  
Vedere la classe `PseudoNetworkInterface` nel file `system_ntkd.vala`.

In una hashmap memoriziamo le istanze di `PseudoNetworkInterface` di ogni interfaccia di rete che il
programma deve gestire, indicizzate per il nome `dev`.  
Vedere la variabile globale `pseudonic_map` nel file `system_ntkd.vala`.

* * *

Bisogna integrare l'unica istanza di `NeighborhoodManager` che si crea all'avio del programma e muore alla sua terminazione.

* * *

La fase di inizio ascolto su una interfaccia (in broadcast e in unicast sul linklocal) va svolta in correlazione con
la fase di aggiunta dell'interfaccia alla gestione del `NeighborhoodManager`.  
Analogamente la fase di fine ascolto va svolta in correlazione con
la fase di rimozione dell'interfaccia dalla gestione del `NeighborhoodManager`.

Vedere la funzione `void stop_monitor(string dev)` nel file `system_ntkd.vala`.

