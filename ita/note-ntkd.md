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

Per indicare i namespaces usiamo in seguito le abbreviazioni:

`G.` per `Gee`;  
`N.` per `Netsukuku`;  
`N.N.` per `Netsukuku.Neighborhood`;  
`N.I.` per `Netsukuku.Identities`;  
`N.Q.` per `Netsukuku.Qspn`;  
`N.P.` per `Netsukuku.PeerServices`;  
`N.C.` per `Netsukuku.Coordinator`;  
`N.H.` per `Netsukuku.Hooking`;  
`N.A.` per `Netsukuku.Andna`;  
`T.` per `TaskletSystem`;  

* * *

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

La classe `N.PRNGen` nel file `rngen.vala` con i suoi metodi statici `init_rngen` e `int_range` viene usata nel codice
per generare numeri pseudo-casuali. Può essere inizializzato questo generatore con un valore costante per ottenere
risultati deterministici, ad esempio per riprodurre nei test gli stessi risultati. Oppure si possono ottenere
valori più casuali.

* * *

In una classe memoriziamo le informazioni relative ad una interfaccia di rete che il programma deve gestire.  
Vedere la classe `N.HandledNic` nel file `system_ntkd.vala`.  
I dati memorizzati includono `dev`, `mac`, `linklocal`, `INeighborhoodNetworkInterface nic`.

In una hashmap memoriziamo le istanze di `N.HandledNic` di ogni interfaccia di rete che il
programma deve gestire, indicizzate per il nome `dev`.  
Vedere la variabile globale `handlednic_map` nel file `system_ntkd.vala`.

* * *

In una classe memoriziamo le informazioni relative ad una interfaccia di rete che il programma deve gestire.  
Vedere la classe `N.PseudoNetworkInterface` nel file `system_ntkd.vala`.
I dati memorizzati includono `dev`, `mac`, `linklocal`, `INeighborhoodNetworkInterface nic` e anche informazioni
legate al fatto che si tratta di pseudo-interfacce di rete simulate da un socket di sistema:
`send_pathname`, `listen_pathname`, `st_listen_pathname`.

In una hashmap memoriziamo le istanze di `N.PseudoNetworkInterface` di ogni interfaccia di rete che il
programma deve gestire, indicizzate per il nome `dev`.  
Vedere la variabile globale `pseudonic_map` nel file `system_ntkd.vala`.

* * *

In una classe memoriziamo le informazioni relative ad un arco di cui il programma viene a conoscenza.  
Questo arco è tra una specifica interfaccia di rete di questo nodo e una specifica interfaccia di rete di un altro nodo
diretto vicino.  
Vedere la classe `N.NodeArc` nel file `system_ntkd.vala`.

Una istanza di `N.NodeArc` viene costruita passando una istanza di `N.N.INeighborhoodArc` e una istanza
di `N.IdmgmtArc`. La prima viene prodotta dal modulo `Neighborhood` quando l'arco è rilevato e viene comunicata
tramite un segnale al programma. La seconda viene prodotta dal programma per poi comunicarla al modulo `Identities`.
Entrambe le istanze possono essere in seguito reperite dall'istanza di `N.NodeArc`.

In una hashmap memoriziamo le istanze di `N.NodeArc` di ogni arco di cui il programma viene a conoscenza, indicizzate
per l'identificativo numerico `id` dell'istanza di `N.IdmgmtArc`.  
Vedere la variabile globale `arc_map` nel file `system_ntkd.vala`.

* * *

In una classe memoriziamo le informazioni relative ad ogni identità assunta dal nodo corrente: una identità principale
e zero o più identità di connettività.  
Vedere la classe `N.IdentityData` nel file `system_ntkd.vala`.

Anche il modulo `Identities` ha fra i suoi compiti il mantenimento delle info relative ad ogni identità, ma usiamo
anche questa classe per alcuni motivi:
*   praticità nel ciclare fra le identità presenti; e.g. **TODO**
*   memorizzazione di alcuni dati temporanei nelle fasi di aggiornamento delle informazioni; e.g. **TODO**
*   ...

Ogni istanza di `N.IdentityData` quando viene creata riceve un `NodeID` e un identificativo locale numerico
progressivo chiamato `local_identity_index`.

Tutte le istanze che vengono create sono immediatamente messe nell hashmap `local_identities` indicizzate per
il valore intero che costituisce il `NodeID` assegnato all'identità (dal modulo `Identities`).

Il metodo helper `create_local_identity` nel file `system_ntkd.vala` è usato per creare una istanza e
metterla nel hashmap dopo aver verificato che non era già stata creata.

Il metodo helper `find_local_identity` nel file `system_ntkd.vala` è usato per cercare una istanza di cui
si conosce il `NodeID`. Se esiste la restituisce altrimenti restituisce *null*.

Il metodo helper `find_local_identity_by_index` nel file `system_ntkd.vala` è usato per cercare una istanza di cui
si conosce il `local_identity_index`. Se esiste la restituisce altrimenti restituisce *null*.

Il metodo helper `remove_local_identity` nel file `system_ntkd.vala` è usato per rimuovere una istanza dal
hashmap dopo aver verificato che era in effetti presente.

Ogni istanza di `N.IdentityData` memorizza una lista di archi-identità nel membro `identity_arcs`, rappresentati
da istanze di `IdentityArc`.

Ogni istanza di `N.IdentityData` può anche memorizzare:

*   L'indirizzo Netsukuku come istanza `Naddr my_naddr`.
*   Il fingerprint come istanza `Fingerprint my_fp`.
*   I livelli come "identità di connettività" `int connectivity_from_level` e `int connectivity_to_level`.
*   L'identità da cui è derivata `weak IdentityData? copy_of_identity`.
*   L'istanza `QspnManager qspn_mgr`.
*   Un getter `bool main_id` dice se è la principale confrontando se stessa con la
    variabile globale `IdentityData main_identity_data` sempre nel file `system_ntkd.vala`.

Alcuni metodi pubblici della classe `N.IdentityData` sono usati per registrare gli handler dei segnali
che produce il modulo Qspn. Le implementazioni sono comunque funzioni esterne alla classe, per poterle
raggruppare nel file `qspn_signals.vala`.  
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

...



* * *

In una classe memoriziamo le informazioni relative ad ogni arco-identità appartenenti ad una identità
assunta dal nodo corrente.  
Vedere la classe `N.IdentityArc` nel file `system_ntkd.vala`.

Ogni istanza di `N.IdentityArc` memorizza immediatamente:

*   l'identità di questo nodo a cui è associata (come indice `int local_identity_index`)
*   l'arco su cui poggia (come passato al modulo Identites, cioè `IIdmgmtArc arc`)
*   l'arco identità stesso (come ottenuto dal modulo Identites, cioè `IIdmgmtIdentityArc id_arc`)

Siccome l'identità è memorizzata in un `IdentityArc` come indice `int local_identity_index`, questa
classe ha un getter per risalire alla istanza di `IdentityData` che se viene invocato su una identità che
non è più nell hashmap `local_identities` ha l'effetto di terminare la tasklet corrente.

Ogni istanza può anche memorizzare:

*   `string peer_mac`
*   `string peer_linklocal`
*   `QspnArc? qspn_arc`

* * *

Bisogna integrare l'unica istanza di `N.N.NeighborhoodManager` che si crea all'avio del programma e muore alla sua terminazione.

* * *

La fase di inizio ascolto su una interfaccia (in broadcast e in unicast sul linklocal) va svolta in correlazione con
la fase di aggiunta dell'interfaccia alla gestione del `N.N.NeighborhoodManager`.  
Analogamente la fase di fine ascolto va svolta in correlazione con
la fase di rimozione dell'interfaccia dalla gestione del `N.N.NeighborhoodManager`.

Vedere la funzione `void stop_rpc(string dev)` nel file `system_ntkd.vala`.

* * *

Per integrare il modulo `Neighborhood` occorre implementare con una classe l'interfaccia `N.N.INeighborhoodIPRouteManager`.  
Vedere la classe `N.NeighborhoodIPRouteManager` nel file `neighborhood_helpers.vala`.

Quando il modulo `Neighborhood` chiama `add_address` di `N.N.INeighborhoodIPRouteManager` la classe
suddetta deve aggiungere l'indirizzo IP passato (si tratterà di un linklocal) alla interfaccia di
rete (reale) passata.  
Lo fa chiedendo al `commander.vala` di eseguire un singolo comando "`ip address add ...`".

Quando il modulo `Neighborhood` chiama `add_neighbor` di `N.N.INeighborhoodIPRouteManager` la classe
suddetta deve aggiungere una route diretta all'indirizzo IP passato (si tratterà di un linklocal) tramite
l'interfaccia di rete (reale) passata.  
Lo fa chiedendo al `commander.vala` di eseguire un singolo comando "`ip route add ...`".

Quando il modulo `Neighborhood` chiama `remove_neighbor` di `N.N.INeighborhoodIPRouteManager` la classe
suddetta deve rimuovere una route diretta all'indirizzo IP passato (si tratterà di un linklocal) tramite
l'interfaccia di rete (reale) passata.  
Lo fa chiedendo al `commander.vala` di eseguire un singolo comando "`ip route del ...`".

Quando il modulo `Neighborhood` chiama `remove_address` di `N.N.INeighborhoodIPRouteManager` la classe
suddetta deve rimuovere l'indirizzo IP passato (si tratterà di un linklocal) alla interfaccia di
rete (reale) passata.  
Lo fa chiedendo al `commander.vala` di eseguire un singolo comando "`ip address del ...`".

* * *

Per integrare il modulo `Neighborhood` occorre implementare con una classe l'interfaccia `N.N.INeighborhoodStubFactory`.  
Vedere la classe `N.NeighborhoodStubFactory` nel file `neighborhood_helpers.vala`.

Quando il modulo `Neighborhood` chiama `get_broadcast_for_radar` di `N.N.INeighborhoodStubFactory` la classe
suddetta deve restituire uno stub per mandare messaggi broadcast allo stesso modulo `Neighborhood` nei
nodi diretti vicini attraverso una specifica interfaccia di rete.  
Lo fa chiamando sulla classe `N.StubFactory` il metodo `get_stub_whole_node_broadcast_for_radar` che gli
ottiene una istanza di stub per messaggi broadcast di nodo. Lo stub ottenuto è radice (si veda `AddressManager addr`
nel file `interfaces.rpcidl` del pacchetto `ntkdrpc`). Per ottenere uno stub dedicato al modulo `Neighborhood`
la classe istanzia un `NeighborhoodManagerStubHolder`.

Quando il modulo `Neighborhood` chiama `get_unicast` di `N.N.INeighborhoodStubFactory` la classe
suddetta deve restituire uno stub per mandare messaggi unicast allo stesso modulo `Neighborhood` in uno
specifico nodo diretto vicino attraverso uno specifico arco `N.N.INeighborhoodArc`.  
Lo fa chiamando sulla classe `N.StubFactory` il metodo `get_stub_whole_node_unicast` che gli
ottiene una istanza di stub per messaggi unicast di nodo. Lo stub ottenuto è radice (si veda `AddressManager addr`
nel file `interfaces.rpcidl` del pacchetto `ntkdrpc`). Per ottenere uno stub dedicato al modulo `Neighborhood`
la classe istanzia un `NeighborhoodManagerStubHolder`.

* * *

Per integrare il modulo `Neighborhood` occorre implementare con una classe l'interfaccia `N.N.INeighborhoodQueryCallerInfo`.  
Vedere la classe `N.NeighborhoodQueryCallerInfo` nel file `neighborhood_helpers.vala`.

Quando il modulo `Neighborhood` chiama `is_from_broadcast` di `N.N.INeighborhoodQueryCallerInfo` la classe
suddetta deve esaminare una istanza di `CallerInfo` (la classe che la libreria `ntkdrpc` basata su ZCD
passa ai metodi remoti quando riceve una chiamata da un altro nodo) e stabilire se il metodo
remoto è stato chiamato da un diretto vicino in modalità broadcast; e in quel caso deve restituire
l'istanza di `N.N.INeighborhoodNetworkInterface` associata all'interfaccia di rete da cui il messaggio è
stato ricevuto.  
Lo fa chiamando sulla classe `N.SkeletonFactory` il metodo `from_caller_get_mydev` che gli restituisce la
stringa con il nome dell'interfaccia da cui è stato ricevuto il messaggio. Tramite questo nome e
l'hashmap `pseudonic_map` risale all'istanza di `N.PseudoNetworkInterface` e in essa trova l'istanza
di `N.N.INeighborhoodNetworkInterface` che deve restituire.

Quando il modulo `Neighborhood` chiama `is_from_unicast` di `N.N.INeighborhoodQueryCallerInfo` la classe
suddetta deve esaminare una istanza di `CallerInfo` e una lista di archi `N.N.INeighborhoodArc`
che gli sono passati e stabilire se il metodo remoto è stato chiamato da un diretto vicino in modalità unicast;
e in quel caso deve restituire l'arco `N.N.INeighborhoodArc` da cui il messaggio è
stato ricevuto.  
Lo fa individuando dal `CallerInfo` l'interfaccia di rete che ha ricevuto e il MAC address dell'interfaccia
di rete del vicino che ha trasmesso (contenuta secondo il protocollo ZCD nella classe `NeighbourSrcNic`
nella classe `CallerInfo`). Poi cicla negli archi e cerca quello che corrisponde.

* * *

Per integrare il modulo `Neighborhood` occorre implementare con una classe l'interfaccia `N.N.INeighborhoodNetworkInterface`.  
Vedere la classe `N.NeighborhoodNetworkInterface` nel file `neighborhood_helpers.vala`.

Le istanze di questa classe sono da passare al modulo `Neighborhood` a rappresentazione di una interfaccia di rete
da gestire. Una istanza è creata passando una istanza di `N.PseudoNetworkInterface`. A sua volta una istanza di
di `N.PseudoNetworkInterface` contiene un riferimento (**TODO** mettere weak) alla relativa istanza di `N.N.INeighborhoodNetworkInterface`.

Quando il modulo `Neighborhood` chiama `measure_rtt` di `N.N.INeighborhoodNetworkInterface` la classe
suddetta deve misurare il Round Trip Time delle comunicazioni tramite questa interfaccia di rete con un
indirizzo IP linklocal che identifica una precisa interfaccia di rete di un nodo diretto vicino.  
Questo avverrà eseguendo il comando `ping` e interpretandone l'output. Per il momento invece, questo
programma di test chiede al `commander.vala` di eseguire un singolo comando "`ping`" e restituisce
una misurazione fake di 1000 usec.

* * *

Per reagire ai segnali emessi dal modulo `Neighborhood` sono implementate delle funzioni
nel file `neighborhood_signals.vala`.

Il segnale `nic_address_set` è gestito nella funzione `neighborhood_nic_address_set`.  
La funzione riceve un `N.N.INeighborhoodNetworkInterface` e l'indirizzo linklocal
che gli è stato appena assegnato.  
La funzione recupera l'istanza di `N.PseudoNetworkInterface` dal set `pseudonic_map`
e valorizza i suoi membri `linklocal` e `st_listen_pathname`.  
Inoltre crea una istanza di `N.HandledNic` valorizzando tutti i dati in essa contenuti
e la mette in `handlednic_map`.  
Inoltre avvia in questo momento la tasklet in ascolto per i messaggi stream unicast
ricevuti dall'indirizzo linklocal di cui sopra (con `skeleton_factory.start_stream_system_listen`).

Il segnale `arc_added` è gestito nella funzione `neighborhood_arc_added`.  
La funzione riceve un `N.N.INeighborhoodArc` che è stato appena aggiunto
dal modulo `Neighborhood`. Con esso crea un
`N.IdmgmtArc` (che implementa `N.I.IIdmgmtArc`). Con entrambi crea un
`N.NodeArc` che mette nella hashmap `arc_map`.  
Inoltre comunica questo arco al modulo `Identities` usando il metodo `identity_mgr.add_arc`.

Il segnale `arc_changed` è gestito nella funzione `neighborhood_arc_changed`.  
**TODO** in futuri commit; il commento dice:  
// TODO for each identity, for each id-arc, if qspn_arc is present, change cost.

Il segnale `arc_removing` è gestito nella funzione `neighborhood_arc_removing`.  
La funzione riceve un `N.N.INeighborhoodArc` che sta per essere rimosso
dal modulo `Neighborhood`. Cerca la relativa istanza di `N.NodeArc` nella hashmap `arc_map`,
vi recupera l'istanza di `N.I.IIdmgmtArc` e comunica la rimozione di questo
arco al modulo `Identities` usando il metodo `identity_mgr.remove_arc`.

Il segnale `arc_removed` è gestito nella funzione `neighborhood_arc_removed`.  
La funzione riceve un `N.N.INeighborhoodArc` che è stato appena rimosso
dal modulo `Neighborhood`. Rimuove la relativa istanza di `N.NodeArc` dalla hashmap `arc_map`.

Il segnale `nic_address_unset` è gestito nella funzione `neighborhood_nic_address_unset`.  
**TODO** in futuri commit.

* * *

Bisogna integrare l'unica istanza di `N.I.IdentityManager` che si crea all'avio del programma e muore alla sua terminazione.

* * *

Per integrare il modulo `Identities` occorre implementare con una classe l'interfaccia `N.I.IIdmgmtNetnsManager`.  
Vedere la classe `N.IdmgmtNetnsManager` nel file `identities_helpers.vala`.

Quando il modulo `Identities` chiama `create_namespace` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve preparare un nuovo network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire i comandi "`ip netns add ...`",
e alcuni comandi "`ip netns exec ... sysctl ...`".

Quando il modulo `Identities` chiama `create_pseudodev` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve creare una pseudo-interfaccia sopra una interfaccia reale e spostarla su un dato
network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire alcuni comandi:  
Crea la pseudo-interfaccia con "`ip link add dev ... link ...`".  
Poi recupera (o imposta nel caso di una testsuite) il suo indirizzo MAC.  
Poi sposta la pseudo-interfaccia nel namespace con "`ip link set dev ... netns ...`".  
Poi esegue alcuni comandi "`ip netns exec ... sysctl ...`".  
Poi attiva la pseudo-interfaccia con "`ip netns exec ... ip link set dev ... up`".  

Quando il modulo `Identities` chiama `add_address` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve assegnare un indirizzo IP linklocal ad una interfaccia di rete.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip address add ... dev ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.  

Quando il modulo `Identities` chiama `add_gateway` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve assegnare una rotta diretta da una interfaccia di rete del nodo corrente ad una
data interfaccia di un diretto vicino.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip route add ... dev ... src ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.  

Quando il modulo `Identities` chiama `remove_gateway` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve rimuovere una rotta diretta da una interfaccia di rete del nodo corrente ad una
data interfaccia di un diretto vicino.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip route del ... dev ... src ...`".  
Questa operazione può essere richiesta dal modulo `Identities` sia in un dato
network namespace che nel default.  

Le richieste che seguono vengono fatte dal modulo `Identities` di norma nella sequenza
`flush_table`, `delete_pseudodev` e `delete_namespace`.

Quando il modulo `Identities` chiama `flush_table` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve azzerare le tabelle di routing su un dato network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip netns exec ... ip route flush table main`".

Quando il modulo `Identities` chiama `delete_pseudodev` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve eliminare una pseudo-interfaccia di rete che era stata messa su un
dato network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire il
comando "`ip netns exec ... ip link delete ...`".

Quando il modulo `Identities` chiama `delete_namespace` di `N.I.IIdmgmtNetnsManager` la classe
suddetta deve eliminare un network namespace.  
Lo fa chiedendo al `commander.vala` di eseguire il comando "`ip netns del ...`".

* * *

Per integrare il modulo `Identities` occorre implementare con una classe l'interfaccia `N.I.IIdmgmtStubFactory`.  
Vedere la classe `N.IdmgmtStubFactory` nel file `identities_helpers.vala`.

Quando il modulo `Identities` chiama `get_arc` di `N.I.IIdmgmtStubFactory` passando una istanza
di `CallerInfo`, la classe suddetta deve stabilire se il metodo remoto è stato chiamato da un diretto
vicino in modalità unicast;
e in quel caso deve restituire l'arco `N.I.IIdmgmtArc` da cui il messaggio è
stato ricevuto.  
Lo fa passando il `CallerInfo` al metodo `from_caller_get_nodearc` di `skeleton_factory`.

Quando il modulo `Identities` chiama `get_stub` di `N.I.IIdmgmtStubFactory` passando una istanza
di `N.I.IIdmgmtArc`, la classe suddetta deve deve restituire uno stub per mandare messaggi unicast
allo stesso modulo `Identities` attraverso quell'arco.  
Lo fa chiamando sulla classe `N.StubFactory` il metodo `get_stub_whole_node_unicast` che gli
ottiene una istanza di stub per messaggi unicast di nodo. Lo stub ottenuto è radice. Per ottenere uno stub
dedicato al modulo `Identities` la classe istanzia un `IdentityManagerStubHolder`.

* * *

Per integrare il modulo `Identities` occorre implementare con una classe l'interfaccia `N.I.IIdmgmtArc`;
una istanza di questa classe deve rappresentare un arco tra due nodi diretti vicini.  
Vedere la classe `N.IdmgmtArc` nel file `identities_helpers.vala`.

Un'istanza di questa classe viene costruita passando l'istanza di `N.N.INeighborhoodArc`, che il
modulo `Neighborhood` produce quando rileva l'arco. Ricordiamo che esso rappresenta un arco tra una
spcifica interfaccia di rete di questo nodo e una specifica interfaccia di rete di un altro nodo
diretto vicino.

Il modulo `Identities` può interrogare l'interfaccia `N.I.IIdmgmtArc` con i metodi
`get_dev`, `get_peer_linklocal` e `get_peer_mac`, che la classe `N.IdmgmtArc` può facilmente
reperire dall'istanza di `N.N.INeighborhoodArc` ad essa associata.  
Inoltre ad ogni istanza di `N.IdmgmtArc` il costruttore associa un identificativo intero `id`.
Esso viene usato come indice nella variabile globale hashmap `arc_map`.

* * *

Per reagire ai segnali emessi dal modulo `Identities` sono implementate delle funzioni
nel file `identities_signals.vala`.

Il segnale `identity_arc_added` è gestito nella funzione `identities_identity_arc_added`.  
Il segnale indica la creazione di un arco-identità. La funzione riceve l'arco (come istanza
di `IIdmgmtArc` che era stata creata dal programma e passata al modulo Identities), l'identità
in questo nodo (come istanza di `NodeID` che era stata creata dal modulo Identities e resa nota
al programma), l'arco-identità (come istanza di `IIdmgmtIdentityArc`) e eventualmente il
precedente arco-identità (se si tratta di una duplicazione??).  
La funzione scrive a video alcune informazioni e prende nota dell'evento per possibili test.  
Inoltre crea l'istanza di `N.IdentityArc` e la associa all'istanza di `N.IdentityData`
(recuperata con la funzione `find_local_identity`) aggiungendola al suo membro
lista `identity_arcs`.

Il segnale `identity_arc_changed` è gestito nella funzione `identities_identity_arc_changed`.  
Il segnale indica la modifica di un arco-identità. La funzione riceve l'arco `IIdmgmtArc`,
l'identità in questo nodo `NodeID`, l'arco-identità `IIdmgmtIdentityArc`, e un booleano che dice
se `only_neighbour_migrated`.  
La funzione scrive a video alcune informazioni e prende nota dell'evento per possibili test.  
Inoltre recupera l'istanza di `N.IdentityData` con la funzione `find_local_identity`
e con il suo metodo `identity_arcs_find` recupera l'istanza di `N.IdentityArc`. Cosa ci farà
è ancora TODO.

Il segnale `identity_arc_removing` è gestito nella funzione `identities_identity_arc_removing`.  
Il segnale indica la prossima rimozione di un arco-identità. La funzione riceve l'arco `IIdmgmtArc`,
l'identità in questo nodo `NodeID`, e l'identità nel nodo vicino `NodeID`.  
La funzione scrive a video alcune informazioni e prende nota dell'evento per possibili test.

Il segnale `identity_arc_removed` è gestito nella funzione `identities_identity_arc_removed`.  
Il segnale indica la avvenuta rimozione di un arco-identità. La funzione riceve l'arco `IIdmgmtArc`,
l'identità in questo nodo `NodeID`, e l'identità nel nodo vicino `NodeID`.  
La funzione scrive a video alcune informazioni e prende nota dell'evento per possibili test.

Il segnale `arc_removed` è gestito nella funzione `identities_arc_removed`.  
Il segnale indica la avvenuta rimozione di un arco (per malfunzionamento). La funzione riceve
l'arco `IIdmgmtArc`.  
La funzione scrive a video alcune informazioni e prende nota dell'evento per possibili test.  
Inoltre la relativa istanza di `N.N.INeighborhoodArc` (recuperata attraverso la hashmap `arc_map`)
viene rimossa dal modulo Neighborhood con `neighborhood_mgr.remove_my_arc`.  
Infine l'arco viene rimosso dal hashmap `arc_map`.

* * *

Bisogna integrare le istanze di `N.Q.QspnManager` che si creano al momento che nasce una nuova identità
e muoiono alla sua terminazione.

* * *

Per integrare il modulo `Qspn` occorre implementare con una classe l'interfaccia `N.Q.IQspnStubFactory`.  
Vedere la classe `N.QspnStubFactory` nel file `qspn_helpers.vala`.

Una istanza di questa classe è collegata ad una identità assunta in questo nodo. Nella costruzione
di questa istanza viene passato l'identificativo `int local_identity_index`; questa
classe ha un getter per risalire alla istanza di `IdentityData` che se viene invocato su una identità che
non è più nell hashmap `local_identities` ha l'effetto di terminare la tasklet corrente.

Quando il modulo `Qspn` chiama `i_qspn_get_broadcast` di `N.Q.IQspnStubFactory` gli passa una lista
di archi `IQspnArc`, `arcs`, e opzionalmente una istanza di `IQspnMissingArcHandler` per gestire gli archi che
non comunicano l'avvenuta ricezione del messaggio broadcast.  
La classe `N.Q.IQspnStubFactory`, se la lista di archi è vuota restituisce semplicemente una nuova istanza
di `QspnManagerStubVoid`.  
Altrimenti, prepara una lista di `NodeID`, `broadcast_node_id_set`, che sono gli identificativi delle
identità nei nodi diretti vicini reperite da `arcs`.  
Di seguito prepara una lista di `IAddressManagerStub`, `addr_list`, uno per ogni interfaccia di rete
del nodo corrente su cui va trasmesso il messaggio broadcast. Queste sono ottenute con altrettante
chiamate al metodo `stub_factory.get_stub_identity_aware_broadcast` che indicano tra l'altro il
nome dell'interfaccia di rete e la lista `broadcast_node_id_set` dei destinatari.  
Se era richiesto un `IQspnMissingArcHandler`, anche esso viene passato ad ogni chiamata
di `stub_factory.get_stub_identity_aware_broadcast` come nuova istanza di `MissingArcHandlerForQspn`
costruita sulla istanza di `IQspnMissingArcHandler`. Sotto si illustra il meccanismo.  
Dopo aver ottenuto la lista `addr_list` con essi viene costruita e restituita una nuova istanza
di `QspnManagerStubBroadcastHolder`.

La classe `MissingArcHandlerForQspn` implementa l'interfaccia `N.IIdentityAwareMissingArcHandler`
dichiarata nel file `rpc/stub_factory.vala`.  
Nel suo construttore riceve una istanza di `IQspnMissingArcHandler` che memorizza come `qspn_missing`.  
Nel metodo `missing`, che è una callback che verrà chiamata sugli archi-identità che non sono
stati raggiunti da questo messaggio, chiama il metodo `i_qspn_missing` dell'istanza `IQspnMissingArcHandler`
passata all'inizio al metodo `i_qspn_get_broadcast`.

Quando il modulo `Qspn` chiama `i_qspn_get_tcp` di `N.Q.IQspnStubFactory` gli passa un arco `IQspnArc arc`,
e un booleano `wait_reply`.  
La classe suddetta ottiene una `IAddressManagerStub addrstub`,
chiamando il metodo `stub_factory.get_stub_identity_aware_unicast_from_ia`, dopo aver reperito
da `arc` il relativo `IdentityArc`.  
Con questa viene costruita e restituita una nuova istanza
di `QspnManagerStubHolder`.

* * *

Per integrare il modulo `Qspn` occorre implementare con una classe l'interfaccia `N.Q.IQspnThresholdCalculator`.  
Vedere la classe `N.ThresholdCalculator` nel file `qspn_helpers.vala`.

Quando il modulo `Qspn` chiama `i_qspn_calculate_threshold` di `N.Q.IQspnThresholdCalculator` gli passa
due istanze di `IQspnNodePath`. Questi percorsi calcolati dal modulo Qspn hanno ognuno un *costo* che
la classe suddetta sa tradurre in microsecondi. La classe moltiplica la somma dei costi per 50 e
la converte in millisecondi. Restituisce questo valore.

* * *

Per integrare il modulo `Qspn` occorre implementare con una classe l'interfaccia `N.Q.IQspnArc`.  
Vedere la classe `N.QspnArc` nel file `qspn_helpers.vala`.

Una istanza di questa classe rappresenta un arco-identità. Viene costruita passando una
istanza di `IdentityArc`.  
Dalla istanza di `IdentityArc` si accede alla istanza di `N.N.INeighborhoodArc` prodotta
dal modulo Neighborhood che rappresenta l'arco-nodo; attraverso la quale si può accedere
al relativo costo.  
Nel costruttore di `N.QspnArc` si sceglie un intero randomico da 0 a 1000 e lo si memorizza
per fare in modo che, aggiungendolo al costo dell'arco-nodo quando viene richiesto il
costo di questo arco-identità, questa piccola (insignificante) variazione renda molto
improbabile ottenere due path distinti con costo identico.

Quando il modulo `Qspn` chiama `i_qspn_get_cost` di `N.Q.IQspnArc` la classe suddetta
restituisce una istanza di `N.Cost` col valore ottenuto come detto prima.

Quando il modulo `Qspn` chiama `i_qspn_equals` di `N.Q.IQspnArc` la classe suddetta
restituisce *TRUE* solo se si tratta di due `N.QspnArc` costruite sulla stessa
istanza di `IdentityArc`.

Quando il modulo `Qspn` chiama `i_qspn_comes_from` di `N.Q.IQspnArc` gli passa
una istanza di `CallerInfo`. La classe suddetta usa il metodo `from_caller_get_identityarc`
di `skeleton_factory` per vedere se si tratta della stessa istanza di `IdentityArc`.


* * *

Per integrare il modulo `Qspn` occorre implementare con delle classi serializzabili le
interfacce `N.Q.IQspnMyNaddr`, `N.Q.IQspnCost`, `N.Q.IQspnFingerprint`.  
Vedere le classi `N.Naddr`, `N.Cost`, `N.Fingerprint` nel file `serializables.vala`.

* * *

Per reagire ai segnali emessi dal modulo `Qspn` sono implementate delle funzioni
nel file `qspn_signals.vala`.

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

* * *

Bisogna potersi mettere in ascolto e inviare comunicazioni agli altri nodi:

*   Su una interfaccia di rete in broadcast.  
    Per mettersi in ascolto serve il nome dell'interfaccia di rete. Questo viene passato all'avvio del programma.
*   Su un indirizzo linklocal in unicast.  
    Per mettersi in ascolto serve l'indirizzo IP linklocal assegnato all'interfaccia di rete. Questo
    viene scelto dal `N.N.NeighborhoodManager` che lo notifica con un segnale.
*   Su un indirizzo routabile in unicast.  
    Per mettersi in ascolto serve l'indirizzo IP routabile assegnato al nodo. Questo è calcolato sulla base
    dell'indirizzo Netsukuku del nodo, il quale viene gestito dal modulo `Qspn`.

* * *

In una unica classe viene implementato il codice per ottenere degli stub radice di vario tipo (broadcast, unicast,
di nodo o di identità).  
Vedere la classe `N.StubFactory` nel file `rpc/stub_factory.vala`.

La classe è pensata per avere una sola istanza in una variabile globale `stub_factory`
nel file `system_ntkd.vala`, ad uso comune di tutto il codice.

Per avere uno stub radice di tipo unicast verso un modulo di nodo si chiama il metodo `get_stub_whole_node_unicast`
passando un arco `N.N.INeighborhoodArc` e indicando se si vuole attendere la risposta con `wait_reply`.  
La classe prepara i dati serializzabili che sono previsti dal protocollo ZCD, cioè:

*   Un `WholeNodeSourceID` che rappresenta il mittente.  
    Il `NeighborhoodNodeID` che serve viene preso dalla variabile globale `skeleton_factory`.
*   Un `WholeNodeUnicastID` che rappresenta il destinatario.  
    Il `NeighborhoodNodeID` che serve viene preso dal membro `neighbour_id` dell'arco passato al metodo.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente) usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) viene preso dal membro `mac` dell'istanza di
    `N.N.INeighborhoodNetworkInterface` che è presa dal membro `nic` dell'arco passato al metodo.

Poi la classe chiama il metodo di `ntkdrpc` che restituisce lo stub radice di tipo stream.  
In particolare in questo programma di test si chiama il metodo che usa il medium system. Per questo serve
la stringa `send_pathname` che la classe costruisce a partire dall'indirizzo IP linklocal dell'interfaccia
di rete del destinatario, il quale è memorizzato in `neighbour_nic_addr`
nell'arco passato al metodo.  
Quindi con tutti questi dati la classe chiama il metodo `get_addr_stream_system`.

Per avere uno stub radice di tipo broadcast verso un modulo di nodo, poiché l'unico caso d'uso è quello
della funzione di radar del modulo `Neighborhood`, si chiama il metodo `get_stub_whole_node_broadcast_for_radar`
passando l'interfaccia di rete gestita, cioè l'istanza di `N.N.INeighborhoodNetworkInterface`.  
La classe prepara i dati serializzabili che sono previsti dal protocollo ZCD, cioè:

*   Un `WholeNodeSourceID` che rappresenta il mittente.  
    Il `NeighborhoodNodeID` che serve viene preso dalla variabile globale `skeleton_factory`.
*   Un `EveryWholeNodeBroadcastID` che rappresenta il destinatario, che è chiunque in questo caso.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente) usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) viene preso dal membro `mac` dell'istanza di
    `N.N.INeighborhoodNetworkInterface` passata al metodo.

Poi la classe chiama il metodo di `ntkdrpc` che restituisce lo stub radice di tipo datagram.  
In particolare in questo programma di test si chiama il metodo che usa il medium system. Per questo serve
la stringa `send_pathname` che la classe costruisce a partire dall'identificativo univoco del processo
(che è memorizzato nella variabile globale `pid` nel file `system_ntkd.vala`) e dal nome della pseudo interfaccia
di rete, che viene preso dal membro `dev` dell'istanza di `N.N.INeighborhoodNetworkInterface` passata al metodo.  
Quindi con tutti questi dati la classe chiama il metodo `get_addr_datagram_system`.

Per avere uno stub radice di tipo unicast verso un modulo di identità si chiama
il metodo `get_stub_identity_aware_unicast` passando l'arco-nodo `INeighborhoodArc arc`,
l'identità mittente `IdentityData identity_data` e l'identificativo dell'identità
destinataria nel nodo destinatario `NodeID unicast_node_id`; oppure si può usare
il metodo scorciatoia `get_stub_identity_aware_unicast_from_ia` passando
l'arco-identità `IdentityArc ia`.  
In entrambi i casi si passa anche opzionalmente un booleano che dice se si intende attendere
la risposta `bool wait_reply=true`.  
La classe prepara i dati serializzabili che sono previsti dal protocollo ZCD, cioè:

*   Un `IdentityAwareSourceID` che rappresenta il mittente.  
    Il `NodeID` che serve viene preso dalla istanza `identity_data` passata al metodo.
*   Un `IdentityAwareUnicastID` che rappresenta il destinatario.  
    Il `NodeID` che serve è il `unicast_node_id` passato al metodo.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente) usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) viene preso dal membro `mac` dell'istanza di
    `N.N.INeighborhoodNetworkInterface nic` memorizzata nell'arco-nodo `arc` passato al metodo.

Poi la classe chiama il metodo di `ntkdrpc` che restituisce lo stub radice di tipo stream.  
In particolare in questo programma di test si chiama il metodo che usa il medium system. Per questo serve
la stringa `send_pathname` che la classe costruisce a partire dall'indirizzo linklocal della
scheda di rete del vicino `neighbour_nic_addr` memorizzato nell'arco-nodo `arc` passato al metodo.  
Quindi con tutti questi dati la classe chiama il metodo `get_addr_stream_system`.

Per avere uno stub radice di tipo broadcast verso un modulo di identità si chiama il metodo
`get_stub_identity_aware_broadcast` passando il nome della scheda di rete `string my_dev` su cui
il messaggio deve essere trasmesso, l'identità mittente `IdentityData identity_data` e
un set con gli identificativo delle identità destinatarie nei nodi destinatari `List<NodeID> broadcast_node_id_set`;
può anche essere opzionalmente passata una callback per gestire gli archi-identità dai quali non
si riceve in tempo il messaggio di avvenuta ricezione (una istanza di `IIdentityAwareMissingArcHandler`).  
La classe prepara i dati serializzabili che sono previsti dal protocollo ZCD, cioè:

*   Un `IdentityAwareSourceID` che rappresenta il mittente.  
    Il `NodeID` che serve viene preso dalla istanza `identity_data` passata al metodo.
*   Un `IdentityAwareBroadcastID` che rappresenta i destinatari.  
    Il set `List<NodeID>` che serve è il `broadcast_node_id_set` passato al metodo.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente) usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) è recuperato, attraverso il `my_dev` passato
    al metodo, dalla relativa istanza di `N.HandledNic` in `handlednic_map`.

Poi la classe chiama il metodo di `ntkdrpc` che restituisce lo stub radice di tipo datagram.  
In particolare in questo programma di test si chiama il metodo che usa il medium system. Per questo serve
la stringa `send_pathname` che la classe costruisce a partire dall'identificativo univoco del processo
(che è memorizzato nella variabile globale `pid` nel file `system_ntkd.vala`) e dal nome della pseudo interfaccia
di rete passata al metodo in `my_dev`.  
Se è stato passato il gestore `IIdentityAwareMissingArcHandler`, si usa come descritto sotto per costruire
una implementazione di `N.IAckCommunicator` da passare al metodo di `ntkdrpc`.  
Quindi con tutti questi dati la classe chiama il metodo `get_addr_datagram_system`.

Se è stato passato il gestore (opzionale) degli archi-identità *mancati* dal messaggio, che è una istanza
di `IIdentityAwareMissingArcHandler`, si costruisce con questa e con l'identità mittente `IdentityData identity_data`
una istanza di `NodeMissingArcHandlerForIdentityAware`; e poi con questa e il nome della scheda di rete `string my_dev`
e l'istanza di `N.StubFactory this`, si costruisce una istanza di `AcknowledgementsCommunicator`.

La classe `AcknowledgementsCommunicator` implementa l'interfaccia `N.IAckCommunicator` fornita da `ntkdrpc`, il
cui metodo `process_src_nics_list (Gee.List<Netsukuku.ISrcNic> src_nics_list)` viene chiamato da `ntkdrpc`
per indicare quali istanze (serializzabili) di `ISrcNic` (cioè `NeighbourSrcNic`) hanno segnalato di avere
ricevuto il messaggio. La classe `AcknowledgementsCommunicator` dunque, al momento della sua costruzione
chiama `stub_factory.get_current_arcs_for_broadcast(my_dev)` per sapere quali archi dovrebbero essere raggiunti;
poi al momento della verifica, lo chiama di nuovo perché solo quelli che sono rimasti possono essere
verificati; poi per ogni arco da verificare, se il suo `neighbour_mac` non è fra i `NeighbourSrcNic` che
hanno segnalato, lo prepara in una lista `lst_missed` di archi mancati; alla fine tutti quelli in questa
lista `lst_missed`, ognuno in una tasklet separata, vengono passati al metodo `missing`
di `NodeMissingArcHandlerForIdentityAware`.

Qui entra in gioco l'identità mittente `IdentityData identity_data` che è nota all'istanza
di `NodeMissingArcHandlerForIdentityAware`. Questa cerca fra gli archi-identità di proprietà
di `identity_data` quelli (possono in teoria essere più) che poggiano sull'arco passato al
suo metodo `missing` e, ognuno in una tasklet separata, li passa al metodo `missing` della
istanza di `IIdentityAwareMissingArcHandler` che è stata all'inizio passata al metodo
`get_stub_identity_aware_broadcast` di `N.StubFactory`.

Il metodo `get_current_arcs_for_broadcast(my_dev)` di `N.StubFactory` è privato. Esso
trova gli archi `N.N.INeighborhoodArc` attuali sull'interfaccia di rete `my_dev`
nella hashmap `arc_map`.

Anche le classi `NodeMissingArcHandlerForIdentityAware` e `AcknowledgementsCommunicator`
sono inner-classi private di `N.StubFactory`.

L'interfaccia `N.IIdentityAwareMissingArcHandler` dichiarata nel file `rpc/stub_factory.vala`,
serve come abbiamo visto per definire una callback da chiamare sugli archi-identità che non sono
stati raggiunti da un particolare messaggio. Questa callback è definita nel suo metodo
astratto `missing(IdentityData identity_data, IdentityArc identity_arc)`

* * *

Diverse classi (una o più per ogni modulo) realizzano gli stub dedicati.  
Vedere le classi nel file `rpc/module_stubs.vala`.

La classe `N.NeighborhoodManagerStubHolder` implementa l'interfaccia `N.INeighborhoodManagerStub`
a partire da uno stub radice.

La classe `N.IdentityManagerStubHolder` implementa l'interfaccia `N.IIdentityManagerStub`
a partire da uno stub radice.

Il modulo `Qspn` usa uno stub o per messaggi unicast o per messaggi broadcast. A seconda dei casi
il modulo chiama o `i_qspn_get_tcp` o `i_qspn_get_broadcast`. Il primo restituisce una istanza
di `QspnManagerStubHolder`, mentre il secondo a seconda dei casi restituisce una istanza
di `QspnManagerStubBroadcastHolder` o di `QspnManagerStubVoid`. Tutti implementano
l'interfaccia `N.IQspnManagerStub` a partire da uno stub radice o da una lista di stub radice.
Il `...Void` ignora i messaggi che gli sono passati perché non ci sono archi.

* * *

In una unica classe viene implementato il codice per avviare le tasklet in ascolto nelle varie modalità
(stream, datagram, di rete, di sistema, ...).  
Vedere la classe `N.SkeletonFactory` nel file `rpc/skeleton_factory.vala`.

La classe è pensata per avere una sola istanza in una variabile globale `skeleton_factory`
nel file `system_ntkd.vala`, ad uso comune di tutto il codice.

Nella fase di avvio del nodo, dopo averne costruito l'istanza, viene valorizzata la sua
proprietà `NeighborhoodNodeID whole_node_id` con l'identificativo fornito dal modulo `Neighborhood`
(**TODO** da finire con un successivo commit).

Il costruttore valorizza la proprietà privata `ServerDelegate dlg`. Servirà alle tasklet che ricevono i
messaggi, come illustreremo a breve.  
Inoltre valorizza la proprietà privata `NodeSkeleton node_skeleton`.

Il metodo `start_stream_system_listen` avvia una tasklet per gestire i messaggi di tipo stream trasmessi
in unicast a uno specifico indirizzo IP. Si può usare sia per i messaggi ricevuti da un diretto vicino
(cioè con un IP linklocal) sia per i messaggi ricevuti da un nodo qualsiasi (instradati) all'interno di
un g-nodo di qualsiasi livello in cui si trova anche il nodo corrente (cioè con un IP pubblico).  
A questo metodo viene passata la stringa `listen_pathname`, come prescrive la funzione `stream_system_listen`
fornita dalla libreria `ntkdrpc` basata su ZCD. Inoltre il metodo si costruisce un `N.IErrorHandler`
apposito per questo listener (la classe `ServerErrorHandler` è una privata inner-classe di `N.SkeletonFactory`)
e usa una istanza di `N.IDelegate` in comune con tutti i listener (la classe `ServerDelegate` è una
privata inner-classe di `N.SkeletonFactory`); entrambe le classi sono prescritte da `ntkdrpc`.  
Infine il metodo `start_stream_system_listen` memorizza in un HashMap l'handle della tasklet avviata
associandolo alla stringa `listen_pathname`, di modo che la stessa classe `N.SkeletonFactory` con il
metodo `stop_stream_system_listen` possa gestirne la terminazione.

Il metodo `start_datagram_system_listen` avvia una tasklet per gestire i messaggi di tipo datagram trasmessi
in broadcast su un segmento di rete (broadcast domain) e recepiti da una certa interfaccia di rete
del nodo corrente.  
A questo metodo viene passata la stringa `listen_pathname`, la stringa `send_pathname` e una istanza di
`ISrcNic src_nic`, come prescrive la funzione `datagram_system_listen`
fornita dalla libreria `ntkdrpc` basata su ZCD.
Inoltre il metodo si costruisce un `N.IErrorHandler` apposito per questo listener
e usa l'istanza comune di `N.IDelegate`, prescritte da `ntkdrpc`.  
Infine il metodo `start_datagram_system_listen` memorizza in un HashMap l'handle della tasklet avviata
associandolo alla stringa `listen_pathname`, di modo che la stessa classe `N.SkeletonFactory` con il
metodo `stop_datagram_system_listen` possa gestirne la terminazione.

La classe ServerDelegate (nel metodo `get_addr_set` prescritto da `ntkdrpc`) fa il suo lavoro
usando i metodi `get_dispatcher` e `get_dispatcher_set` di `N.SkeletonFactory` passando il CallerInfo.

Il metodo `get_dispatcher` è usato per gestire i messaggi in stream; in esso il caller è una
istanza di `StreamCallerInfo`:

*   Se il caller ha in `source_id` un `WholeNodeSourceID` e in `unicast_id` un `WholeNodeUnicastID`:
    *   Recupera il `NeighborhoodNodeID` che referenzia il nodo mittente;
    *   Recupera il `NeighborhoodNodeID` che referenzia il nodo destinatario;
    *   Se il nodo destinatario risulta essere il nodo corrente (`node_skeleton.id`)
        allora restituisce il proprio `node_skeleton`.
*   Se il caller ha in `source_id` un `IdentityAwareSourceID` e in `unicast_id` un `IdentityAwareUnicastID`
    e in `src_nic` un `NeighbourSrcNic`:
    *   Recupera il `NodeID` che referenzia l'identità mittente;
    *   Recupera il `NodeID` che referenzia l'identità destinatario;
    *   Recupera il MAC della interfaccia che ha trasmesso;
    *   Chiama `get_identity_skeleton(source_nodeid, unicast_nodeid, peer_mac)`:
        *   Il metodo `get_identity_skeleton` cerca tra le istanze di `IdentityData` che
            rappresentano le identità locali (in `local_identities`) una con il `NodeID`
            idicato come destinatario e si accerta che essa abbia un arco-identità verso il
            `NodeID` indicato come mittente, in particolare con l'interfaccia di rete con il MAC
            indicato. Se la trova restituisce un `IdentitySkeleton` costruito su quella istanza
            di IdentityData, altrimenti restituisce *null*.
*   ... **TODO** altro?

Il metodo `get_dispatcher_set` è usato per gestire i messaggi in datagram; in esso il caller è una
istanza di `DatagramCallerInfo`:

*   Se il caller ha in `source_id` un `WholeNodeSourceID` e in `broadcast_id` un `EveryWholeNodeBroadcastID`:
    *   Siccome i messaggi di questo tipo sono destinati sempre a tutti i nodi, semplicemente
        restituisce in un set il proprio `node_skeleton`.
*   Se il caller ha in `source_id` un `IdentityAwareSourceID` e in `broadcast_id` un `IdentityAwareBroadcastID`
    e in `src_nic` un `NeighbourSrcNic` e in `listener` un `DatagramSystemListener`:
    *   Recupera il `NodeID` che referenzia l'identità mittente;
    *   Recupera il set di `NodeID` che referenziano le identità destinatarie;
    *   Recupera il MAC della interfaccia che ha trasmesso;
    *   Recupera (cercando in `pseudonic_map`) il nome della interfaccia di questo nodo che ha ricevuto;
    *   Chiama `get_identity_skeleton_set(source_nodeid, broadcast_set, peer_mac, my_dev)`:
        *   Il metodo `get_identity_skeleton_set` prepara una lista di `IAddressManagerSkeleton` che
            verrà restituita. Cerca tra le istanze di `IdentityData` in `local_identities` se ce
            ne sono alcune con il `NodeID` tra quelli idicati come destinatari. Per ognuna che trova,
            si accerta che essa abbia un arco-identità verso il `NodeID` indicato come mittente, in particolare
            che abbia come interfaccia di rete nel nodo corrente la stessa che ha ricevuto il
            messaggio e come interfaccia di rete nel nodo peer una con il MAC indicato. Se è così
            aggiunge alla lista da restituire un `IdentitySkeleton` costruito su quella istanza
            di IdentityData.
*   ... **TODO** altro?

Il metodo `from_caller_get_mydev`, se il caller è un `StreamCallerInfo`, ... **TODO** in futuri commit.

Il metodo `from_caller_get_mydev`, se il caller è un `DatagramCallerInfo`, ciclando le istanze
di `N.PseudoNetworkInterface` nel set `pseudonic_map` trova quella i cui `listen_pathname` e `send_pathname`
corrispondono al `listener` del caller e se lo trova restituisce il nome dell'interfaccia
di rete da cui il messaggio è stato ricevuto.

Il metodo `from_caller_get_nodearc` è usato (dice il commento) per richieste di nodo, per sapere da quale
arco una tale richiesta è arrivata:

*   Riceve come parametro il caller.
*   Se il caller è un `StreamCallerInfo`, da esso reperisce il `source_id`;
*   Se questo è un `WholeNodeSourceID`, da esso reperisce il `peer_node_id`;
*   Dal caller reperisce il `listener` che deve essere un `StreamSystemListener`: da questo
    reperisce il `listen_pathname`;
*   Dal caller reperisce il `src_nic` che deve essere un `NeighbourSrcNic`: da questo
    reperisce il `neighbour_mac`;
*   Cerca in `arc_map` l'istanza di `N.NodeArc` associata al `neighborhood_arc` che
    soddisfa questi 3 criteri (`listen_pathname`, `neighbour_mac`, `peer_node_id`) e
    se la trova restituisce questo `N.NodeArc`.
*   In tutti gli altri casi restituisce *null*.

Il metodo `from_caller_get_identityarc` è usato (dice il commento) per richieste di identità, per sapere da quale
arco-identità una tale richiesta è arrivata:

*   Riceve come parametri il caller e una istanza di `IdentityData`.
*   Se il caller è un `StreamCallerInfo`:
    *   Dal caller reperisce il `source_id` che deve essere un `IdentityAwareSourceID`;
        da esso reperisce il `source_nodeid`.
    *   Dal caller reperisce il `src_nic` che deve essere un `NeighbourSrcNic`;
        da esso reperisce il `peer_mac`.
    *   Serca fra gli archi-identità di quella `IdentityData` quello che soddisfa
        `source_nodeid` e `peer_mac`.
    *   Se non lo trova restituisce *null*.
*   Se il caller è un `DatagramCallerInfo`:
    *   Dal caller reperisce il `source_id` che deve essere un `IdentityAwareSourceID`;
        da esso reperisce il `source_nodeid`.
    *   Dal caller reperisce il `src_nic` che deve essere un `NeighbourSrcNic`;
        da esso reperisce il `peer_mac`.
    *   Dal caller reperisce il `listener` che deve essere un `DatagramSystemListener`;
        da esso reperisce il `my_dev`.
    *   Serca fra gli archi-identità di quella `IdentityData` quello che soddisfa
        `source_nodeid`, `my_dev` e `peer_mac`.
    *   Se non lo trova restituisce *null*.

La classe `NodeSkeleton` (di cui viene costruita una sola istanza) implementa
l'interfaccia `N.IAddressManagerSkeleton` con il compito di restituire i vari manager dei moduli
di nodo: `N.INeighborhoodManagerSkeleton` e `N.IIdentityManagerSkeleton`.

La classe `IdentitySkeleton` (di cui viene costruita di volta in volta una nuova istanza passando
al costruttore l'istanza di `IdentityData`) implementa
l'interfaccia `N.IAddressManagerSkeleton` con il compito di restituire i vari manager dei moduli
di identità: `N.IQspnManagerSkeleton`, `N.IPeersManagerSkeleton`, `N.ICoordinatorManagerSkeleton`,
`N.IHookingManagerSkeleton`, `N.IAndnaManagerSkeleton`.  
È implementato il `qspn_manager_getter` che prevede una attesa di qualche istante
se non è stato ancora costruito.  
Non è ancora implementato il `peers_manager_getter`.  
Non è ancora implementato il `coordinator_manager_getter`.  
Non è ancora implementato il `hooking_manager_getter`.  
Va ancora fatto in `ntkdrpc` quanto serve a produrre il metodo astratto `andna_manager_getter`. Dopo
andrà implementato il `andna_manager_getter`.

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

Vedere le classi `N.Naddr`, `N.Cost`, `N.Fingerprint` nel file `serializables.vala`.

La classe `N.Naddr` implementa l'interfaccia `N.IQspnAddress` fornita da `ntkdrpc` e le interfacce
`N.Q.IQspnNaddr` e `N.Q.IQspnMyNaddr` fornite dal modulo Qspn. Contiene l'indirizzo nella
forma di array di posizioni `ArrayList<int>`; contiene anche la tipologia di rete nella stessa
forma.  
Ha un metodo `bool is_real_from_to(int from, int to)`.

Quando il modulo `Qspn` chiama i metodi `i_qspn_get_pos`, `i_qspn_get_gsize`
o `i_qspn_get_levels` di `N.Q.IQspnNaddr` o il metodo `i_qspn_get_coord_by_address` di `N.Q.IQspnMyNaddr`
la classe suddetta risponde come dovuto.  
Vi è anche un metodo `equals`.

La classe `N.Cost` implementa l'interfaccia `N.Q.IQspnCost` fornita dal modulo Qspn. Contiene
il costo dell'arco in latenza in microsecondi.

Implementa come dovuto i metodi richiesti dal modulo `Qspn`: `i_qspn_is_dead`, `i_qspn_is_null`,
`i_qspn_compare_to`, `i_qspn_add_segment`, `i_qspn_important_variation`.

La classe `N.Fingerprint` implementa l'interfaccia `N.Q.IQspnFingerprint` fornita dal modulo Qspn. Contiene
il fingerprint di un nodo o g-nodo della rete, la sua anzianità nel g-nodo direttamente superiore
e le anzianità di tutti i g-nodi superiori a cui appartiene. Inoltre, se si tratta di un g-nodo, contiene
le anzianità dei g-nodi inferiori da cui ha preso l'identificativo. Si rimanda alla documentazione del modulo
per approfondimenti.

Implementa come dovuto i metodi richiesti dal modulo `Qspn`:
`i_qspn_construct`, `i_qspn_elder_seed`, `i_qspn_equals`, `i_qspn_get_level`.

La classe `N.IdentityAwareSourceID` implementa l'interfaccia `N.ISourceID` fornita da `ntkdrpc`. Contiene una
istanza di `NodeID id` che rappresenta il mittente.

La classe `N.IdentityAwareUnicastID` implementa l'interfaccia `N.IUnicastID` fornita da `ntkdrpc`. Contiene una
istanza di `NodeID id` che rappresenta il vicino destinatario.

La classe `N.IdentityAwareBroadcastID` implementa l'interfaccia `N.IBroadcastID` fornita da `ntkdrpc`. Contiene una
lista `List<NodeID> id_set` che identifica i destinatari.

* * *

All'**avvio del programma** nella funzione `main` del file `system_ntkd.vala`, per prima cosa si parserizzano
le opzioni date sulla linea di comando.  
Si può passare la topologia della rete (il default è 1,1,1,2).  
Si può passare il primo indirizzo Netsukuku della prima identità.  
Si deve passare un PID per identificare un processo che simula un nodo in un testsuite.  
Si deve passare una lista di pseudo-interfacce di rete da gestire.  
Si può passare una lista di compiti (tasks) da fare in un dato momento.  
Si può passare il livello (di default 0) della sottorete a gestione autonoma.  
Si può passare un flag per far sì che il nodo accetti richieste anonime dirette a lui.  
Si può passare un flag per far sì che il nodo rifiuti di passare richieste anonime mascherandole.  

La topologia fa valorizzare
la variabile globale `ArrayList<int> gsizes`,
la variabile globale `ArrayList<int> g_exp`,
la variabile globale `int levels`
e la variabile globale `ArrayList<int> hooking_epsilon`,
nel file `system_ntkd.vala`.  
Se viene passato il primo indirizzo Netsukuku della prima identità, con questo
viene subito valorizzato `ArrayList<int> naddr` che è locale nella funzione `main`; altrimenti
resta per ora vuoto e verrà valorizzato in modo casuale dopo l'inizializzazione del generatore `PRNGen`.  
Il PID finisce nella variabile globale `int pid` nel file `system_ntkd.vala`.  
La lista di interfacce finisce in `ArrayList<string> devs` che è locale nella funzione `main`.  
La lista di interfacce finisce in `ArrayList<string> tasks` che è locale nella funzione `main`.  
Il livello della sottorete a gestione autonoma finisce nella variabile globale `int subnetlevel`.  
Il flag di accettazione richieste anonime finisce nella variabile globale `bool accept_anonymous_requests`.  
Il flag di rifiuto di mascheramento finisce nella variabile globale `bool no_anonymize`.  

Dopo si inizializza lo scheduler delle tasklet. Va nella variabile globale `ITasklet tasklet`.

Dopo si inizializzano i singoli moduli (principalmente perché hanno delle classi serializzabili) e si
registrano le classi serializzabili che servono.  
I moduli si inizializzano con il metodo statico `init` delle classi  `NeighborhoodManager`,
`IdentityManager`, `QspnManager`, ...  
Le classi serializzabili da registrare direttamente in `main` sono `WholeNodeSourceID`, `WholeNodeUnicastID`,
`EveryWholeNodeBroadcastID`, `NeighbourSrcNic`, `IdentityAwareSourceID`, `IdentityAwareUnicastID`,
`IdentityAwareBroadcastID`, `Naddr`, `Fingerprint`, `Cost`, ...

In particolare l'inizializzazione del modulo `QspnManager` serve anche a impostare dei parametri comuni
a tutte le istanze di `QspnManager` (che saranno più di una nel singolo nodo). Questi sono stati
memorizzati come costanti `max_paths`, `max_common_hops_ratio`, `arc_timeout`.

Dopo si inizializza il generatore di numeri pseudo-casuali. Si usa come seed il PID.  
Questa operazione si fa con il metodo statico `init_rngen` delle classi `PRNGen`, `NeighborhoodManager`,
`IdentityManager`, `QspnManager`, ...

Dopo, se non era valorizzato `naddr`, il primo indirizzo Netsukuku della prima identità,
viene valorizzato in modo casuale nel range imposto dalla topologia.

Dopo si inizializza un FakeCommandDispatcher. Va nella variabile globale `FakeCommandDispatcher fake_cm`.

Dopo si passa lo scheduler delle tasklet alla libreria `ntkdrpc`, chiamandone la funzione statica `init_tasklet_system`.

Dopo si istanzia lo `skeleton_factory`.

Dopo si istanzia lo `stub_factory`.

Dopo si istanzia il NeighborhoodManager nella variabile globale `neighborhood_mgr`. Esso è unico.

Dopo si connettono i segnali del NeighborhoodManager.

Dopo si prepara una hashmap (var globale) `pseudonic_map` per le interfacce di rete da gestire
rappresentate da istanze di `N.PseudoNetworkInterface`. Delle stesse interfacce di rete si prepara
anche una hashmap (var globale) `handlednic_map` rappresentate da istanze di `N.HandledNic`.

Delle stesse interfacce di rete
si prepara anche una lista (var locale) `if_list_dev` per i nomi, una lista `if_list_mac`
per i MAC e una lista `if_list_linklocal` per gli indirizzi link-local: queste liste
serviranno nella costruzione del IdentityManager.

Per ogni pseudo-interfaccia di rete da gestire si eseguono questi compiti:

*   Si genera pseudo-random un finto MAC.
*   Si memorizza una nuova istanza di `N.PseudoNetworkInterface` in `pseudonic_map`.
*   Si eseguono dei comandi di inizializzazione della scheda di rete.
*   Si avvia una tasklet di ascolto per i messaggi datagram broadcast ricevuti da
    questa interfaccia di rete (con `skeleton_factory.start_datagram_system_listen`).
*   Si avvia `neighborhood_mgr.start_monitor` con l'istanza di `N.PseudoNetworkInterface`; questo
    imposterà il IP linklocal sulla scheda, e emetterà il segnale `nic_address_set`: questo,
    gestito come già detto sopra, valorizzerà il campo `linklocal` di `N.HandledNic`.
*   Si memorizzano i dati della pseudo-interfaccia di rete nelle liste `if_list_dev`,
    `if_list_mac` e `if_list_linklocal`.

Dopo si istanzia il IdentityManager nella variabile globale `identity_mgr`. Esso è unico.  
Si passa ad esso una funzione callback per la generazione pseudo-random di un indirizzo link-local.

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

Dopo si entra nel main loop degli eventi fino alla terminazione del programma (con il segnale Posix
di terminazione).

Alla terminazione del programma,
si rimuovono tutte le identità di connettività che sono presenti al momento nel nodo.  
Per ognuna di esse
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

* * *

* * *

Dopo il completamento delle parti necessarie all'integrazione del modulo Neighborhood viene realizzata
la script `test_1_neighborhood`.

Sono simulati i nodi 123, 456 e 789. Ognuno con eth0 e wlan0.  
Un solo segmento di rete ethernet collega tutte le tre eth0.  
Invece le radio collegano 123 con 456 e 456 con 789.  
Tutto ciò è simulato con:

```
eth_domain   -i 123_eth0  -i 456_eth0  -i 789_eth0
radio_domain -i 123_wlan0 -o 456_wlan0
radio_domain -o 123_wlan0 -i 456_wlan0 -o 789_wlan0
radio_domain              -o 456_wlan0 -i 789_wlan0
```

* * *

Dopo il completamento delle parti necessarie all'integrazione del modulo Identities viene realizzata
la script `test_1_n_identities`.

Sono simulati i nodi 1223 e 2321. Ognuno con wl0.  
La topologia è simulata con:

```
radio_domain -i 1223_wl0 -o 2321_wl0
radio_domain -i 2321_wl0 -o 1223_wl0
```
