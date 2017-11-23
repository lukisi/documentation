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

Oltre a questo requisito, vediamo quali altri sono richiesti dalla libreria *ntkdrpc* e come il
demone *ntkd* li implementa. E d'altra parte quali servizi sono forniti dalla libreria *ntkdrpc* e come il
demone *ntkd* li sfrutta.

Per quanto il linguaggio Vala lo consente, si cercherà di mettere tutto il codice relativo a queste
mansioni nel file sorgente `ntkd_rpc.vala`.

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

Per la classe radice e per ognuna delle classi che prevedono scambi di messaggi RPC (di norma
una classe per modulo) la libreria *ntkdrpc* fornisce una interfaccia stub.

**TODO**

