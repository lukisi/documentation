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


			public QspnManager.create_net (Netsukuku.Qspn.IQspnMyNaddr my_naddr, Netsukuku.Qspn.IQspnFingerprint my_fingerprint, Netsukuku.Qspn.IQspnStubFactory stub_factory);



        IQspnStubFactory:
			public abstract Netsukuku.IQspnManagerStub i_qspn_get_broadcast (Gee.List<Netsukuku.Qspn.IQspnArc> arcs, Netsukuku.Qspn.IQspnMissingArcHandler? missing_handler = null);
			public abstract Netsukuku.IQspnManagerStub i_qspn_get_tcp (Netsukuku.Qspn.IQspnArc arc, bool wait_reply = true);




			public QspnManager.migration (Gee.List<Netsukuku.Qspn.IQspnArc> internal_arc_set, Gee.List<Netsukuku.Qspn.IQspnArc> internal_arc_prev_arc_set, Gee.List<Netsukuku.Qspn.IQspnNaddr> internal_arc_peer_naddr_set, Gee.List<Netsukuku.Qspn.IQspnArc> external_arc_set, Netsukuku.Qspn.IQspnMyNaddr my_naddr, Netsukuku.Qspn.IQspnFingerprint my_fingerprint, Netsukuku.Qspn.ChangeFingerprintDelegate update_internal_fingerprints, Netsukuku.Qspn.IQspnStubFactory stub_factory, int guest_gnode_level, int host_gnode_level, Netsukuku.Qspn.QspnManager previous_identity);


			public QspnManager.enter_net (Gee.List<Netsukuku.Qspn.IQspnArc> internal_arc_set, Gee.List<Netsukuku.Qspn.IQspnArc> internal_arc_prev_arc_set, Gee.List<Netsukuku.Qspn.IQspnNaddr> internal_arc_peer_naddr_set, Gee.List<Netsukuku.Qspn.IQspnArc> external_arc_set, Netsukuku.Qspn.IQspnMyNaddr my_naddr, Netsukuku.Qspn.IQspnFingerprint my_fingerprint, Netsukuku.Qspn.ChangeFingerprintDelegate update_internal_fingerprints, Netsukuku.Qspn.IQspnStubFactory stub_factory, int guest_gnode_level, int host_gnode_level, Netsukuku.Qspn.QspnManager previous_identity);




