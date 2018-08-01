# ntkd - RPC - Chiamate Lato Server

In questo documento vediamo passo passo come il demone *ntkd* gestisce gli aspetti "lato server" delle
chiamate a metodi remoti.

## ntkd si appoggia su ntkdrpc prodotta con "rpcdesign"

Con il tool *rpcdesign* abbiamo prodotto una libreria di livello intermedio (ntkdrpc) con le classi nel
namespace "Netsukuku" che fornisce il root-dispatcher `AddressManager addr`. Come sappiamo ntkdrpc si
appoggia sulla libreria ZCD.

Poi viene costruito il demone *ntkd* che si appoggia su ntkdrpc.

Il demone *ntkd* avvia sia un *listener* TCP sia uno UDP con i metodi della libreria ntkdrpc. Essi si
avviano in una tasklet e richiamano i relativi listener della libreria ZCD. **TODO** si aggiungeranno
listener a connessioni e a messaggi nel medium unix-domain socket.

### Individuare il (o i) corretto root-dispatcher da invocare

Un messaggio viene recepito da uno dei listener nella libreria ZCD.

ZCD prepara un zcd.CallerInfo e chiama un metodo di un delegato fornito da ntkdrpc.

ntkdrpc prepara un Netsukuku.CallerInfo, che può essere una istanza di:

*   TcpclientCallerInfo.
*   UnicastCallerInfo. **TODO** si rimuoverà.
*   BroadcastCallerInfo.
*   StreamUnixCallerInfo. **TODO** si aggiungerà.
*   DatagramUnixCallerInfo. **TODO** si aggiungerà.

Poi ntkdrpc chiama il metodo `get_addr_set` di un delegato Netsukuku.IRpcDelegate, la cui implementazione
è fornita dal demone *ntkd*.

Il demone *ntkd* in questo metodo recupera dal CallerInfo:

*   Caso TcpclientCallerInfo. Una stringa `peer_address`, una istanza di ISourceID `source_id` e una
    istanza di IUnicastID `unicast_id`.  
    **TODO** La stringa `peer_address` che al momento il framework ZCD ottiene come informazione dal
    socket che gli è passato dal sistema operativo, dovrà essere sostituita da una informazione che il
    protocollo ZCD deve prevedere sottoforma di istanza di ISrcNic che rappresenta un identificativo
    univoco dell'interfaccia di rete che trasmette il messaggio.
*   Caso UnicastCallerInfo. è stato rimosso. **TODO** togliere.
*   Caso BroadcastCallerInfo. Una stringa `peer_address`, una stringa `dev`, una istanza di ISourceID
    `source_id` e una istanza di IBroadcastID `broadcast_id`.  
    **TODO** La stringa `peer_address` come detto sopra va sostituita con una ISrcNic trasmessa nel
    protocollo ZCD. Inoltre la stringa `dev` che al momento il framework ZCD ottiene come informazione
    dal socket che gli è passato dal sistema operativo, dovrà essere sostituita da una informazione che
    rappresenta la modalità con cui la presente tasklet era in ascolto: su una specifica interfaccia di
    rete o su un unix-domain.
*   Caso StreamUnixCallerInfo. **TODO** si aggiungerà.
*   Caso DatagramUnixCallerInfo. **TODO** si aggiungerà.

Poi il demone *ntkd* a seconda dei casi:

*   Caso TcpclientCallerInfo. Prepara una lista vuota di IAddressManagerSkeleton.  
    Chiama `SkeletonFactory.get_dispatcher(source_id, unicast_id, peer_address)`. Se questo restituisce
    una istanza la aggiunge alla lista.  
    Restituisce la lista.  
    *Nota:* Il metodo `SkeletonFactory.get_dispatcher` gestisce sia il caso di messaggio di identità da
    un diretto vicino, sia il caso di messaggio di nodo da un diretto vicino, sia il caso di messaggio di
    identità da un nodo di un nostro g-nodo (cioè pervenuto a noi tramite un indirizzo IP routabile).
*   Caso UnicastCallerInfo. è stato rimosso. **TODO** togliere.
*   Caso BroadcastCallerInfo. Restituisce la lista di IAddressManagerSkeleton ottenuta con la
    chiamata `SkeletonFactory.get_dispatcher_set(source_id, broadcast_id, peer_address, dev)`.  
    *Nota:* Il metodo `SkeletonFactory.get_dispatcher_set` gestisce il caso di messaggio di nodo da un
    diretto vicino eseguito nelle operazioni di "radar scan", quindi da accettare anche se non esiste già
    un arco realizzato dal modulo Neighborhood. Non è previsto al momento alcun altro tipo di messaggio di
    nodo oltre a quelli nelle operazioni di "radar scan". Inoltre gestisce il caso di messaggio di identità
    da un diretto vicino. La modalità broadcast può essere usata solo per i messaggi da diretti vicini.

Infine ntkdrpc riceve un set (forse vuoto) di root-dispatcher su cui invocare un metodo remoto. Sa come
proseguire.

Vediamo come si individuano gli skeleton da restituire.

Nel metodo `IAddressManagerSkeleton? get_dispatcher(ISourceID source_id, IUnicastID unicast_id, string peer_address)`:

*   Se `unicast_id` è un IdentityAwareUnicastID:  
    In questo caso `source_id` deve essere un IdentityAwareSourceID. Altrimenti restituisce subito *null*.  
    Si estrapola da `unicast_id` il `NodeID identity_aware_unicast_id`.  
    Si estrapola da `source_id` il `NodeID identity_aware_source_id`.  
    Chiama il metodo `get_identity_skeleton(identity_aware_source_id, identity_aware_unicast_id, peer_address)`.  
    Questo metodo, conoscendo le *identità* del nodo e gli *archi-identità* che li collegano ad altri nodi,
    restituisce, se esiste, il `identity_skeleton` relativo alla identità `identity_aware_unicast_id`, ma
    solo se questa è collegata tramite un *arco-identità* alla identità `identity_aware_source_id`. Inoltre
    deve trattarsi di un *arco-identità* che si appoggia all'arco formato dal modulo Neighborhood
    con `peer_address`. Altrimenti *null*.
*   Se `unicast_id` è un WholeNodeUnicastID:  
    In questo caso `source_id` deve essere un WholeNodeSourceID. Altrimenti restituisce subito *null*.  
    Si estrapola da `source_id` il `NeighborhoodNodeID whole_node_source_id`.  
    Conosciamo l'istanza `node_skeleton`. Se è stato formato un arco dal modulo Neighborhood con
    `whole_node_source_id`, se inoltre tale arco riporta come indirizzo IP `peer_address` allora va
    restituita l'istanza `node_skeleton`. Altrimenti *null*.
*   Se `unicast_id` è un PeersUnicastID:  
    Conosciamo la nostra identità principale. Si restituisce il suo `identity_skeleton`.

Nel metodo `Gee.List<IAddressManagerSkeleton> get_dispatcher_set(ISourceID source_id, IBroadcastID broadcast_id, string peer_address, string dev)`:

*   Se `broadcast_id` è un EveryWholeNodeBroadcastID:  
    In questo particolare caso, qualsiasi sia l'interfaccia da cui riceve il messaggio e i dati di
    provenienza, il nodo deve ricevere una lista con l'istanza `node_skeleton`. Tale skeleton sarà
    certamente usato per una operazione di "radar scan" sul modulo Neighborhood.
*   Se non è un EveryWholeNodeBroadcastID, subito controlla che esista un arco formato dal modulo
    Neighborhood su `dev` con `peer_address`. Altrimenti restituisce subito una lista vuota.
*   Se `broadcast_id` è un IdentityAwareBroadcastID:  
    In questo caso `source_id` deve essere un IdentityAwareSourceID. Altrimenti restituisce subito
    lista vuota.  
    Si estrapola da `broadcast_id` la `Gee.List<NodeID> identity_aware_broadcast_set`.  
    Si estrapola da `source_id` il `NodeID identity_aware_source_id`.  
    Chiama il metodo `get_identity_skeleton_set(identity_aware_source_id, identity_aware_broadcast_set, peer_address, dev)`.  
    Questo metodo restituisce una lista con gli `identity_skeleton` relativi alle sue identità che
    sono incluse in `identity_aware_broadcast_set`, ma solo quelle che sono collegate tramite
    un *arco-identità* alla identità `identity_aware_source_id` e tale *arco-identità* si appoggia
    all'arco formato dal modulo Neighborhood su `dev` con `peer_address`.

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

**NIC proprio, modalità nuova**
Il demone *ntkd* è in grado di recuperare da qualsiasi tipo di CallerInfo un `ListenMode listen_mode`
che identifica la modalità di ascolto con cui è stato recepito il messaggio. Quindi contiene
le informazioni che servono al demone *ntkd* per individuare uno dei propri NIC.

**NIC proprio, modalità vecchia**
Se la chiamata è arrivata con il protocollo TCP abbiamo la stringa `my_address` la quale (almeno
per i metodi remoti chiamati sui diretti vicini) è sempre l'indirizzo di scheda assegnato dal
modulo Neighborhood all'interfaccia di rete reale. In base alle sue conoscenze il demone *ntkd*
è in grado di associare alla stringa `my_address` una univoca stringa `dev`.  
Se la chiamata è arrivata con il protocollo UDP abbiamo la stringa `dev`.

**NIC del peer, modalità nuova**
Il demone *ntkd* è in grado di recuperare da qualsiasi tipo di CallerInfo un `ISrcNic src_nic` che
identifica l'interfaccia di rete del nostro vicino da cui il messaggio è stato trasmesso.

**NIC del peer, modalità vecchia**
Il demone *ntkd* è in grado di recuperare da qualsiasi tipo di CallerInfo un `string peer_address` che
identifica l'interfaccia di rete del nostro vicino da cui il messaggio è stato trasmesso.

Da tutte queste informazioni con le sue conoscenze il demone *ntkd* è in grado di individuare
univocamente l'arco (*arco-nodo* o *arco-identità*) da cui ha ricevuto il messaggio.

Queste operazioni le fa nei metodi `NodeArc? from_caller_get_nodearc(CallerInfo rpc_caller)` e
`IdentityArc? from_caller_get_identityarc(CallerInfo rpc_caller, IdentityData identity_data)`
di SkeletonFactory.

