# Modulo Neighborhood - Chiamate Lato Server

In questo documento vediamo passo passo come una applicazione debba usare il modulo Neighborhood per gestire gli aspetti "lato server" delle chiamate a metodi remoti.

## Applicazione basata su MOD-RPC prodotta con "rpcdesign"

Supponiamo che con il tool *rpcdesign* abbiamo prodotto una libreria di livello intermedio (MOD-RPC) con le classi nel namespace "Netsukuku" che fornisce il root-dispatcher `AddressManager addr`. Come sappiamo MOD-RPC si appoggia sulla libreria ZCD.

Poi viene costruita una APP che si appoggia su MOD-RPC. La APP contiene al suo interno il modulo Neighborhood.

La APP avvia sia un *listener* TCP sia uno UDP con i metodi della libreria MOD-RPC. Essi si avviano in una tasklet e richiamano i relativi listener della libreria ZCD.

### Individuare il (o i) corretto root-dispatcher da invocare

Un messaggio viene recepito da uno dei listener nella libreria ZCD.

ZCD prepara un zcd.CallerInfo e chiama un metodo di un delegato fornito da MOD-RPC.

MOD-RPC prepara un Netsukuku.CallerInfo, che può essere una istanza di:

*   **caso 1.** TcpclientCallerInfo.
*   **caso 2.** UnicastCallerInfo.
*   **caso 3.** BroadcastCallerInfo.

Poi MOD-RPC chiama il metodo `get_addr_set` di un delegato Netsukuku.IRpcDelegate, la cui implementazione è fornita dalla APP.

La APP in questo metodo scompone il CallerInfo in:

*   **caso 1.** Una stringa `peer_address`, una stringa `my_address`, una istanza di ISourceID `source_id` e una istanza di IUnicastID `unicast_id`.
*   **caso 2.** Una stringa `peer_address`, una stringa `dev`, una istanza di ISourceID `source_id` e una istanza di IUnicastID `unicast_id`.
*   **caso 3.** Una stringa `peer_address`, una stringa `dev`, una istanza di ISourceID `source_id` e una istanza di IBroadcastID `broadcast_id`.

Poi la APP a seconda dei casi:

*   **caso 1.** Prepara una lista vuota di IAddressManagerSkeleton.  
    Chiama `Neighborhood.get_dispatcher(source_id, unicast_id, peer_address, null)`. Se questo restituisce una istanza la aggiunge alla lista.  
    Restituisce la lista.  
    *Nota:* In realtà nel caso di un TcpclientCallerInfo rientra anche un messaggio che arrivi non da un diretto vicino ma da un nodo che ci raggiunge
    tramite un indirizzo IP routabile. Un esempio di questo è un messaggio di risposta di un hash-node contattato dal modulo PeerServices. Questi
    casi però non ci interessano in questo documento perché non sono di pertinenza del modulo Neighborhood. Quindi la APP li gestisce per conto
    suo senza consultare il modulo Neighborhood.
*   **caso 2.** Prepara una lista vuota di IAddressManagerSkeleton.  
    Chiama `Neighborhood.get_dispatcher(source_id, unicast_id, peer_address, dev)`. Se questo restituisce una istanza la aggiunge alla lista.  
    Restituisce la lista.
*   **caso 3.** Restituisce la lista di IAddressManagerSkeleton ottenuta con la chiamata `Neighborhood.get_dispatcher_set(source_id, broadcast_id, peer_address, dev)`.

Infine MOD-RPC riceve un set (forse vuoto) di root-dispatcher su cui invocare un metodo remoto. Sa come proseguire.

Vediamo cosa è avvenuto nel metodo chiamato sul modulo Neighborhood.

Nel caso del metodo `IAddressManagerSkeleton? get_dispatcher(ISourceID source_id, IUnicastID unicast_id, string peer_address, string? dev)`:

*   **caso 0.** `unicast_id` è un NoArcWholeNodeUnicastID. In questo particolare caso, l'oggetto contiene il NeighborhoodNodeID e il MAC address del destinatario; bisogna verificare che il NeighborhoodNodeID sia il nostro e che il MAC corrisponda all'interfaccia indicata da `dev`, che in questo caso deve essere valorizzato. Se questi requisiti sono soddisfatti va restituita l'istanza `node_skeleton`. Altrimenti *null*.
*   **caso a.** `unicast_id` è un IdentityAwareUnicastID.  
    In questo caso `source_id` deve essere un IdentityAwareSourceID. Altrimenti restituisce subito *null*.  
    In questo caso `dev` non deve essere valorizzato. Altrimenti restituisce subito *null*.  
    Neighborhood estrapola da `unicast_id` il `NodeID identity_aware_unicast_id`. Se non è possibile, restituisce subito *null*.  
    Neighborhood estrapola da `source_id` il `NodeID identity_aware_source_id`. Se non è possibile, restituisce subito *null*.  
    Chiama il metodo `get_identity_skeleton(identity_aware_source_id, identity_aware_unicast_id, peer_address)` che gli è stato fornito dal suo utilizzatore.  
    L'utilizzatore del modulo, che conosce le *identità* del nodo e gli *archi-identità* che li collegano ad altri nodi, restituisce, se esiste, lo skeleton relativo alla identità `identity_aware_unicast_id`, ma solo se questa è collegata tramite un *arco-identità* alla identità `identity_aware_source_id`. Inoltre deve trattarsi di un *arco-identità* che si appoggia all'arco reale formato con `peer_address`. Altrimenti *null*.
*   **caso b.** `unicast_id` è un WholeNodeUnicastID.  
    In questo caso `source_id` deve essere un WholeNodeSourceID. Altrimenti restituisce subito *null*.  
    In questo caso `dev` non deve essere valorizzato. Altrimenti restituisce subito *null*.  
    Neighborhood estrapola da `source_id` il `NeighborhoodNodeID whole_node_source_id`. Se non è possibile, restituisce subito *null*.  
    Per gestire le chiamate ai *moduli di nodo*, l'utilizzatore ha inizialmente fornito al modulo Neighborhood l'istanza `node_skeleton`. Infatti il modulo Neighborhood è autonomamente in grado di decidere se restituire o meno quella istanza nel suo metodo `get_dispatcher`.  
    Se è stato formato un arco con `whole_node_source_id`, se infine tale arco riporta come indirizzo IP `peer_address` allora va restituita l'istanza `node_skeleton`. Altrimenti *null*.

Nel caso del metodo `Gee.List<IAddressManagerSkeleton> get_dispatcher_set(ISourceID source_id, IBroadcastID broadcast_id, string peer_address, string dev)`:

*   **caso 0.** `broadcast_id` è un EveryWholeNodeBroadcastID. In questo particolare caso, qualsiasi sia l'interfaccia da cui riceve il messaggio e i dati di provenienza, il nodo deve ricevere una lista con l'istanza `node_skeleton`. Tale skeleton sarà certamente usato per un metodo remoto sul modulo Neighborhood.
*   Se non è il caso **0**, subito controlla che esista un arco reale `i` formato su `dev` con `peer_address`. Altrimenti restituisce subito una lista vuota.
*   **caso a.** `broadcast_id` è un IdentityAwareBroadcastID.  
    In questo caso `source_id` deve essere un IdentityAwareSourceID. Altrimenti restituisce subito lista vuota.  
    Neighborhood estrapola da `broadcast_id` la `Gee.List<NodeID> identity_aware_broadcast_set`. Se non è possibile, restituisce subito lista vuota.  
    Neighborhood estrapola da `source_id` il `NodeID identity_aware_source_id`. Se non è possibile, restituisce subito lista vuota.  
    Chiama il metodo `get_identity_skeleton_set(identity_aware_source_id, identity_aware_broadcast_set, peer_address, dev)` che gli è stato fornito dal suo utilizzatore.  
    L'utilizzatore del modulo restituisce una lista con gli skeleton relativi alle sue *identità* che sono incluse in `identity_aware_broadcast_set`, ma solo quelle che sono collegate tramite un *arco-identità* alla identità `identity_aware_source_id` e tale *arco-identità* si appoggia all'arco reale formato su `dev` con `peer_address`.
*   **caso b.** `broadcast_id` è un WholeNodeBroadcastID.  
    In questo caso `source_id` deve essere un WholeNodeSourceID. Altrimenti restituisce subito lista vuota.  
    Neighborhood estrapola da `broadcast_id` la `Gee.List<NeighborhoodNodeID> whole_node_broadcast_set`. Se non è possibile, restituisce subito lista vuota.  
    Neighborhood estrapola da `source_id` il `NeighborhoodNodeID whole_node_source_id`. Se non è possibile, restituisce subito lista vuota.  
    Se `whole_node_broadcast_set` contiene il NeighborhoodNodeID del nodo corrente, se inoltre l'arco reale `i` è stato formato proprio con `whole_node_source_id`, allora va restituita una lista con l'istanza `node_skeleton`. Altrimenti lista vuota.

### Interpretare l'arco da cui arriva la chiamata

Ora vediamo come l'implementatore dello skeleton, fornito dalla APP, nel metodo remoto che è stato invocato è in grado di interpretare l'oggetto CallerInfo, sempre avvalendosi del modulo Neighborhood, per individuare l'arco (*arco-nodo* o *arco-identità*) da cui ha ricevuto il messaggio. Questa operazione ha senso solo se il metodo remoto sa di essere stato invocato da un diretto vicino.

Il modulo che implementa il metodo remoto chiamato, passa il CallerInfo all'utilizzatore del modulo, che è anche l'utilizzatore del modulo Neighborhood. Questi è in grado di scomporre il CallerInfo in un ISourceID `source_id` e una stringa `my_address` o `dev`. Se la chiamata è arrivata con il protocollo TCP abbiamo la stringa `my_address` la quale (almeno per i metodi  remoti chiamati sui diretti vicini) è sempre l'indirizzo di scheda assegnato dal modulo Neighborhood all'interfaccia di rete reale. In base alle sue conoscenze l'utilizzatore del modulo Neighborhood è in grado di associare alla stringa `my_address` una univoca stringa `dev`.

Se il modulo interessato è un modulo *di nodo*, l'utilizzatore chiama il metodo di Neighborhood `get_node_arc(source_id, dev)`. Se il metodo `get_node_arc` restituisce un INeighborhoodArc, l'utilizzatore è poi in grado di associarlo ad un oggetto che il modulo interessato conosce come *arco-nodo*.

Se il metodo `get_node_arc` restituisce *null*, anche l'utilizzatore restituirà *null*. In teoria questo caso si può verificare solo se un *arco-nodo* prima valido (altrimenti il metodo remoto non doveva essere affatto invocato) è stato rimosso pochi istanti prima. In questo caso l'esecuzione del metodo andrebbe "probabilmente" interrotta, ma questo è di pertinenza del codice che implementa il metodo remoto.

Se il modulo interessato è un modulo *di identità*, allora il modulo ha passato all'utilizzatore del modulo Neighborhood anche la sua istanza. In questo modo questi sa individuare quale sia il NodeID dell'identità del nodo corrente interessata, chiamiamola `identity_aware_my_id`. Poi l'utilizzatore chiama il metodo di Neighborhood `get_identity(source_id)` e ottiene il NodeID dell'identità del mittente, chiamiamola `identity_aware_peer_id`. Avendo anche il nome dell'interfaccia di rete reale su cui è arrivato il messaggio, con le sue associazioni l'utilizzatore del modulo Neighborhood è in grado di identificare un oggetto (ad esempio un IQspnArc) che il modulo interessato conosce come *arco-identità*.

Se un *arco-identità* prima valido (altrimenti il metodo remoto non doveva essere affatto invocato) è stato rimosso pochi istanti prima, adesso l'utilizzatore del modulo non lo trova nelle sue associazioni e restituisce *null*. In questo caso l'esecuzione del metodo andrebbe "probabilmente" interrotta, ma questo è di pertinenza del codice che implementa il metodo remoto.

