# ntkd - RPC - Chiamate Lato Server

In questo documento vediamo passo passo come il demone *ntkd* gestisce gli aspetti "lato server" delle
chiamate a metodi remoti.

## ntkd si appoggia su ntkdrpc prodotta con "rpcdesign"

Con il tool *rpcdesign* abbiamo prodotto una libreria di livello intermedio (ntkdrpc) con le classi nel
namespace `Netsukuku` che fornisce il root-dispatcher `AddressManager addr`. Come sappiamo ntkdrpc si
appoggia sulla libreria ZCD.

Poi viene costruito il demone *ntkd* che si appoggia su ntkdrpc.

## Le tasklet in ascolto

Il demone *ntkd* chiama alcuni metodi della libreria ntkdrpc (nel namespace `Netsukuku`)
che fanno da proxy per altrettanti metodi della libreria ZCD (nel namespace `zcd`). Questi avviano delle
tasklet *listener* che si mettono in ascolto.

Si trata di un numero di chiamate possibili ai metodi `datagram_net_listen`, `datagram_system_listen`,
`stream_net_listen`, `stream_system_listen`. In ogni chiamata viene passata una istanza
di `Netsukuku.IDelegate` e una di `Netsukuku.IErrorHandler`. La libreria ntkdrpc di
risposta chiama il metodo omonimo della libreria ZCD (nel namespace `zcd`) passando
istanze di `zcd.IStreamDelegate/zcd.IDatagramDelegate` e di `zcd.IErrorHandler`.

### Individuare il (o i) corretto root-dispatcher da invocare

Un messaggio viene recepito da uno dei listener nella libreria ZCD.

ZCD prepara un `zcd.CallerInfo`. Può essere un StreamCallerInfo (che può contenere un
StreamNetListener o un StreamSystemListener) oppure un DatagramCallerInfo (che può contenere un
DatagramNetListener o un DatagramSystemListener). Per passarlo ad un metodo di un delegato
fornito da ntkdrpc.

Questo ci prepara un analogo `Netsukuku.CallerInfo`. Per passarlo al metodo `get_addr_set` di
un delegato `Netsukuku.IDelegate`, la cui implementazione è fornita dal demone *ntkd*.

Il demone *ntkd* in questo metodo recupera dal CallerInfo:

*   Un Listener `listener`.
*   Un ISourceID `source_id`.
*   Un ISrcNic `src_nic`.
*   Caso StreamCallerInfo:
    *   Un IUnicastID `unicast_id`.
*   Caso DatagramCallerInfo:
    *   Un IBroadcastID `broadcast_id`.

Poi il demone *ntkd* a seconda dei casi:

*   Caso StreamCallerInfo. Prepara una lista vuota di IAddressManagerSkeleton.  
    Chiama `SkeletonFactory.get_dispatcher(listener, source_id, src_nic, unicast_id)`. Se questo restituisce
    una istanza la aggiunge alla lista.  
    Restituisce la lista.  
    *Nota:* Il metodo `SkeletonFactory.get_dispatcher` gestisce sia il caso di messaggio di identità da
    un diretto vicino, sia il caso di messaggio di nodo da un diretto vicino, sia il caso di messaggio di
    identità da un nodo di un nostro g-nodo (cioè pervenuto a noi tramite un indirizzo IP routabile).
*   Caso DatagramCallerInfo. Restituisce la lista di IAddressManagerSkeleton ottenuta con la
    chiamata `SkeletonFactory.get_dispatcher_set(listener, source_id, src_nic, broadcast_id)`.  
    *Nota:* Il metodo `SkeletonFactory.get_dispatcher_set` gestisce il caso di messaggio di nodo da un
    diretto vicino eseguito nelle operazioni di "radar scan", quindi da accettare anche se non esiste già
    un arco realizzato dal modulo Neighborhood. Non è previsto al momento alcun altro tipo di messaggio di
    nodo oltre a quelli nelle operazioni di "radar scan". Inoltre gestisce il caso di messaggio di identità
    da un diretto vicino. La modalità broadcast può essere usata solo per i messaggi da diretti vicini.

Infine ntkdrpc riceve un set (forse vuoto) di root-dispatcher su cui invocare un metodo remoto. Sa come
proseguire.

Vediamo come si individuano gli skeleton da restituire.

Nel metodo `IAddressManagerSkeleton? get_dispatcher(Listener listener, ISourceID source_id, ISrcNic src_nic, IUnicastID unicast_id)`:

*   Se `unicast_id` è un IdentityAwareUnicastID:  
    Si tratta del caso di messaggio di identità da un diretto vicino.  
    In questo caso `source_id` deve essere un IdentityAwareSourceID. Altrimenti restituisce subito *null*.  
    In questo caso `src_nic` deve essere una istanza di una classe (???) da cui è possibile ricavare il `peer_address`
    che identifica il nodo diretto vicino. Altrimenti restituisce subito *null*.  
    Si estrapola da `unicast_id` il `NodeID identity_aware_unicast_id`.  
    Si estrapola da `source_id` il `NodeID identity_aware_source_id`.  
    Il metodo, conoscendo le *identità* del nodo e gli *archi-identità* che le collegano ad altre identità in altri nodi,
    restituisce, se esiste, il `identity_skeleton` relativo alla identità `identity_aware_unicast_id`, ma
    solo se questa è collegata alla identità `identity_aware_source_id` tramite un *arco-identità*
    che si appoggia ad un arco formato dal modulo Neighborhood con `peer_address`. Altrimenti *null*.
*   Se `unicast_id` è un MainIdentityUnicastID:  
    Si tratta del caso di messaggio di identità da un nodo di un nostro g-nodo.  
    In questo caso `src_nic` non ci interessa. E nemmeno `source_id`.  
    Conosciamo la nostra identità principale. Si restituisce il suo `identity_skeleton`.
*   Se `unicast_id` è un WholeNodeUnicastID:  
    Si tratta del caso di messaggio di nodo da un diretto vicino.  
    In questo caso `source_id` deve essere un WholeNodeSourceID. Altrimenti restituisce subito *null*.  
    In questo caso `src_nic` deve essere una istanza di una classe (???) da cui è possibile ricavare il `peer_address`
    che identifica il nodo diretto vicino. Altrimenti restituisce subito *null*.  
    Si estrapola da `source_id` il `NeighborhoodNodeID whole_node_source_id`.  
    Conosciamo l'istanza `node_skeleton`. Se è stato formato dal modulo Neighborhood un arco con
    `whole_node_source_id` sull'interfaccia del vicino identificata da `peer_address` allora va
    restituita l'istanza `node_skeleton`. Altrimenti *null*.

Nel metodo `Gee.List<IAddressManagerSkeleton> get_dispatcher_set(Listener listener, ISourceID source_id, ISrcNic src_nic, IBroadcastID broadcast_id)`:

*   Se `broadcast_id` è un EveryWholeNodeBroadcastID:  
    Si tratta del caso di messaggio per una operazione di "radar scan" sul modulo Neighborhood.  
    In questo particolare caso, qualsiasi sia l'interfaccia da cui riceve il messaggio e i dati di
    provenienza, il nodo deve ricevere una lista con l'istanza `node_skeleton`.
*   Se `broadcast_id` è un IdentityAwareBroadcastID:  
    Si tratta del caso di messaggio di identità da un diretto vicino.  
    In questo caso `source_id` deve essere un IdentityAwareSourceID. Altrimenti restituisce subito
    lista vuota.  
    In questo caso `src_nic` deve essere una istanza di una classe (???) da cui è possibile ricavare il `peer_address`
    che identifica il nodo diretto vicino. Altrimenti restituisce subito lista vuota.  
    Si estrapola da `broadcast_id` la `Gee.List<NodeID> identity_aware_broadcast_set`.  
    Si estrapola da `source_id` il `NodeID identity_aware_source_id`.  
    Il metodo restituisce una lista con gli `identity_skeleton` relativi alle sue identità che
    sono incluse in `identity_aware_broadcast_set`, ma solo quelle che sono collegate tramite
    un *arco-identità* alla identità `identity_aware_source_id` e tale *arco-identità* si appoggia
    all'arco formato dal modulo Neighborhood con `peer_address` sul proprio NIC (o pseudonic) individuato da `listener`.

### Individuare l'arco da cui arriva la chiamata

L'oggetto CallerInfo preparato da ntkdrpc verrà passato come ulteriore argomento ad ogni metodo remoto
invocato dal framework ZCD.

Supponiamo che viene invocato un metodo remoto di un certo modulo (di *identità* o di *nodo*).  
Supponiamo che questo modulo sa che tale metodo è stato invocato per un messaggio ricevuto da un
diretto vicino.  
In questo caso il modulo potrebbe essere interessato a sapere tramite
quale arco (*arco-nodo* o *arco-identità*) è stato ricevuto il messaggio. Lo chiede al demone ntkd,
ad esempio attraverso un delegato o una interfaccia.

Vediamo come le informazioni presenti nell'oggetto CallerInfo preparato da ntkdrpc siano sufficienti
perché il demone *ntkd* possa individuare l'arco (*arco-nodo* o *arco-identità*) da cui ha ricevuto
il messaggio.

Il demone *ntkd* è in grado di recuperare da qualsiasi tipo di CallerInfo un `ISourceID source_id`.

Dal ISourceID possiamo ottenere un `NeighborhoodNodeID peer_node_id` se il metodo remoto è di un modulo
di *nodo*. Questo identifica il nodo mittente del messaggio, mentre il nodo destinatario è sicuramente
il nodo presente, `NeighborhoodNodeID my_node_id = node_skeleton.id`.

Se invece il metodo remoto è di un modulo di *identità* allora dal ISourceID possiamo ottenere un
`NodeID peer_identity_id` che identifica l'identità mittente del messaggio. In questo caso una nostra
identità è il destinatario e il demone *ntkd* la individua perché il modulo richiedente (che è di *identità*)
specifica la sua istanza e da essa il demone *ntkd* risale a un `NodeID my_identity_id = identity_data.nodeid`.

Il demone *ntkd* è in grado di recuperare da qualsiasi tipo di CallerInfo un `Listener listener`
che identifica la modalità di ascolto con cui è stato recepito il messaggio. Quindi contiene
le informazioni che servono al demone *ntkd* per individuare uno dei propri NIC (o pseudonic).

Il demone *ntkd* è in grado di recuperare da qualsiasi tipo di CallerInfo un `ISrcNic src_nic` che
identifica l'interfaccia di rete del nostro vicino da cui il messaggio è stato trasmesso.

Da tutte queste informazioni con le sue conoscenze il demone *ntkd* è in grado di individuare
univocamente l'arco (*arco-nodo* o *arco-identità*) da cui ha ricevuto il messaggio.

Queste operazioni le fa nei metodi `NodeArc? from_caller_get_nodearc(CallerInfo rpc_caller)` e
`IdentityArc? from_caller_get_identityarc(CallerInfo rpc_caller, IdentityData identity_data)`
di SkeletonFactory.

