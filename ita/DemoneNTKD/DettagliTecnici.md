# Demone NTKD - Dettagli Tecnici

1.  [Interazioni con la libreria RPC](#RPC_Library)

## <a name="RPC_Library"></a>Interazioni con la libreria RPC

Il demone *ntkd* ha una dipendenza sulla libreria *ntkdrpc*.

La libreria *ntkdrpc* è stata costruita con il tool *rpcdesign* del progetto [ZCD](../Librerie/ZCD.md).  
Essa svolge le sue mansioni appoggiandosi alla libreria di basso livello ZCD. A sua volta poi espone
una interfaccia al suo utilizzatore. In questa interfaccia le operazioni di basso livello sono nascoste:
l'unico requisito che essa impone al suo utilizzatore è che le strutture dati complesse usate come
argomenti e valori di ritorno dei metodi remoti esposti devono essere serializzabili tramite l'uso
della libreria json-glib.  
Questo requisito è di competenza dei singoli moduli in cui il demone *ntkd* è scoposto. Sono essi
infatti a definire i metodi remoti.

Vediamo quali altri requisiti sono richiesti dalla libreria *ntkdrpc* e come sono implementati direttamente
nel programma *ntkd*.
E d'altra parte quali servizi sono forniti dalla libreria *ntkdrpc* e come il
demone *ntkd* li sfrutta.

Per quanto il linguaggio Vala lo consente, si cercherà di mettere tutto il codice relativo a queste
mansioni nel file sorgente `ntkd_rpc.vala`.

### Avvio

Il programma *ntkd* deve per prima cosa fare in modo che tutte le classi che servono come strutture dati
serializzabili vengano inizializzate. Questo lo fa inizializzando i singoli moduli che prevedono
metodi remoti.

Poi il programma deve inizializzare la libreria *ntkdrpc* e passargli l'implementazione del sistema di tasklet.

Poi il programma chiama `tcp_listen` per avviare una tasklet che si mette in ascolto delle
connessioni via TCP. In seguito il programma chiamerà `udp_listen` una volta per ogni interfaccia
di rete da gestire (dopo aver abilitato l'interfaccia di rete stessa) per avviare una tasklet che
si mette in ascolto dei messaggi via UDP sulla suddetta interfaccia.  
Per supportare queste chiamate, il programma dovrà preparare una istanza di `IRpcDelegate` e una
istanza di `IRpcErrorHandler`, come vedremo in dettaglio più sotto.

### Chiusura

In fase di chiusura del demone, il programma *ntkd* dovrà (per quanto concerne le sue dirette
interazioni con la libreria *ntkdrpc*) semplicemente terminare tutte le tasklet che erano in
ascolto di messaggi e connessioni. Per questo il programma deve mantenere gli *handle* di queste tasklet.

### IRpcDelegate

L'interfaccia di reperimento dello skeleton radice. Prevede il metodo `get_addr_set(caller)` che a
partire da un `CallerInfo` restituisce una lista di `IAddressManagerSkeleton`.

Il demone *ntkd* deve fornire una istanza di questa interfaccia. Essa prende le informazioni contenute
nel `CallerInfo` (che a sua volta può essere un `BroadcastCallerInfo`, o un `TcpclientCallerInfo` o un
`UnicastCallerInfo`) e da queste deve decidere quali istanze di `IAddressManagerSkeleton` restituire.

Lo fa usando il modulo Neighborhood, che mette a disposizione i metodi `get_dispatcher` e
`get_dispatcher_set` di `NeighborhoodManager`.

### IRpcErrorHandler

L'interfaccia di gestione degli errori che possono presentarsi nelle fasi gestite da ZCD.
Prevede il metodo `void error_handler (GLib.Error e)`.

Il demone *ntkd* deve fornire una istanza di questa interfaccia. In presenza di errori il
programma si interrompe. **TODO** valutare altre opzioni.

### Interfacce skeleton

Per la classe radice e per ognuna delle classi che prevedono scambi di messaggi RPC (di norma
una classe per modulo) la libreria *ntkdrpc* fornisce una interfaccia skeleton della quale il demone *ntkd*
deve fornire una implementazione.

`INeighborhoodManagerSkeleton` Questa interfaccia è implementata dalla classe `NeighborhoodManager`
fornita dal modulo Neighborhood.

`IIdentityManagerSkeleton` Questa interfaccia è implementata dalla classe `IdentityManager`
fornita dal modulo Identities.

`IQspnManagerSkeleton` Questa interfaccia è implementata dalla classe `QspnManager`
fornita dal modulo QSPN.

`IPeersManagerSkeleton` Questa interfaccia è implementata dalla classe `PeersManager`
fornita dal modulo PeerServices.

`ICoordinatorManagerSkeleton` Questa interfaccia è implementata dalla classe `CoordinatorManager`
fornita dal modulo Coordinator.

`IHookingManagerSkeleton` Questa interfaccia è implementata dalla classe `HookingManager`
fornita dal modulo Hooking.

`IAddressManagerSkeleton` Questa interfaccia è implementata dalle classi `AddressManagerForNode`
e `AddressManagerForIdentity` fornite dal programma *ntkd* stesso. In esse è implementato
il codice che restituisce le istanze skeleton dei suddetti moduli.

### Interfacce stub

Per la classe radice e per ognuna delle classi che prevedono scambi di messaggi RPC
la libreria *ntkdrpc* fornisce una interfaccia stub. Inoltre fornisce tre metodi per
ottenere (secondo le tre modalità di chiamata dei metodi remoti supportate da ZCD) una
istanza della interfaccia della classe radice. Questi metodi sono:

*   `get_addr_tcp_client`
*   `get_addr_unicast`
*   `get_addr_broadcast`

Il programma *ntkd* può chiamare uno di questi metodi e ottenere una istanza della classe
radice `IAddressManagerStub`. Quando è in possesso di questa istanza, può chiamare i suoi
metodi (e proprietà) pubblici per ottenere una istanza delle classi stub dei moduli:

*   `INeighborhoodManagerStub neighborhood_manager`
*   `IIdentityManagerStub identity_manager`
*   `IQspnManagerStub qspn_manager`
*   `IPeersManagerStub peers_manager`
*   `ICoordinatorManagerStub coordinator_manager`
*   `IHookingManagerStub hooking_manager`

Su ogni classe stub così ottenuta il programma può chiamare i metodi pubblici. Facendolo in realtà
produce la trasmissione della chiamata del metodo remoto.

C'è da dire che le classi stub dei moduli sono ottenute come proprietà della classe stub radice.
Cioè l'istanza di (ad esempio) `IIdentityManagerStub` è valida finché resta in vita l'istanza di
`IAddressManagerStub` a cui è legata. Invece nei moduli spesso si è ideata una interfaccia
chiamata `StubFactory` che consente al modulo di ottenere una istanza della classe stub del modulo
stesso.  
Per questo il programma *ntkd* fornisce per ogni modulo una classe stub proxy.


