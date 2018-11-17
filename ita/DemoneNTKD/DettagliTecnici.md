# Demone NTKD - Dettagli Tecnici

1.  [Interazioni con la libreria RPC](#interazioni-con-la-libreria-rpc)
1.  [Indirizzi IP](#indirizzi-ip)

## Interazioni con la libreria RPC

Il demone *ntkd* ha una dipendenza sulla libreria *ntkdrpc*.

La libreria *ntkdrpc* è stata costruita con il tool *rpcdesign* del progetto [ZCD](../Librerie/ZCD.md).  
Essa svolge le sue mansioni appoggiandosi alla libreria di basso livello ZCD. A sua volta poi espone
una interfaccia al suo utilizzatore. In questa interfaccia le operazioni di basso livello sono nascoste:
l'unico requisito che essa impone al suo utilizzatore è che le strutture dati complesse usate come
argomenti e valori di ritorno dei metodi remoti esposti devono essere serializzabili tramite l'uso
della libreria json-glib.  
Questo requisito è di competenza dei singoli moduli in cui il demone *ntkd* è scomposto. Sono essi
infatti a definire i metodi remoti.

Vediamo quali altri requisiti sono richiesti dalla libreria *ntkdrpc* e come sono implementati direttamente
nel programma *ntkd*.
E d'altra parte quali servizi sono forniti dalla libreria *ntkdrpc* e come il
demone *ntkd* li sfrutta.

Per quanto il linguaggio Vala lo consente, si cercherà di mettere tutto il codice relativo a queste
mansioni in alcuni file sorgente dentro la cartella `rpc`.

### Avvio

Il programma *ntkd* deve per prima cosa fare in modo che tutte le classi che servono come strutture dati
serializzabili vengano inizializzate. Questo lo fa inizializzando i singoli moduli che prevedono
metodi remoti.

Poi il programma deve inizializzare la libreria *ntkdrpc* e passargli l'implementazione del sistema di tasklet.

Poi il programma chiama `stream_net_listen` e `datagram_net_listen` ... **TODO: completare**

Per supportare queste chiamate, il programma dovrà preparare una istanza di `IDelegate` e una
istanza di `IErrorHandler`, come vedremo in dettaglio più sotto.

### Chiusura

In fase di chiusura del demone, il programma *ntkd* dovrà (per quanto concerne le sue dirette
interazioni con la libreria *ntkdrpc*) semplicemente terminare tutte le tasklet che erano in
ascolto di messaggi e connessioni. Per questo il programma deve mantenere gli *handle* di queste tasklet.

### IDelegate

L'interfaccia di reperimento dello skeleton radice. Prevede il metodo `get_addr_set(caller)` che a
partire da un `CallerInfo` restituisce una lista di `IAddressManagerSkeleton`.

... **TODO: completare**

### IErrorHandler

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

`IAddressManagerSkeleton` Questa interfaccia è implementata dalle classi `NodeSkeleton`
e `IdentitySkeleton` fornite dal programma *ntkd* stesso. In esse è implementato
il codice che restituisce le istanze skeleton dei suddetti moduli.

### Interfacce stub

Per la classe radice e per ognuna delle classi che prevedono scambi di messaggi RPC
la libreria *ntkdrpc* fornisce una interfaccia stub. Inoltre fornisce alcuni metodi per
ottenere (secondo le tre modalità di chiamata dei metodi remoti supportate da ZCD) una
istanza della interfaccia della classe radice. Questi metodi sono:

... **TODO: completare**

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
Per questo il programma *ntkd* fornisce per ogni modulo una classe stub proxy. Ad esempio
la classe `IdentityManagerStubHolder`.

## Indirizzi IP

Esaminiamo quali indirizzi IP relativi ad un dato indirizzo Netsukuku in una data topologia
di rete debbano essere computati dal programma e usati in qualche modo nel network stack *default* e
negli altri.  
Iniziamo col ricordare che un sistema può avere diverse identità. Sempre ha una identità *principale*
che opera nel network stack *default* e può avere zero o più identità *di connettività* in altrettanti
distinti network stack.  
L'identità principale ha sempre un indirizzo Netsukuku *reale* in tutte le componenti. Una identità
di connettività ha sempre un indirizzo Netsukuku in cui almeno una componente è *virtuale*.

In un sistema, il network stack default deve essere in grado di comunicare con i sistemi che sono
(in un dato momento) nella stessa rete Netsukuku. Cioè, vogliamo che le applicazioni in esecuzione nel
sistema (che di default operano nel network stack default) siano in grado di comunicare nella rete.
Quindi, il sistema deve avere nel network stack default un indirizzo IP associato all'indirizzo
Netsukuku *reale* della sua identità principale e le rotte necessarie a instradare pacchetti IP (generati localmente
o da inoltrare) verso gli indirizzi IP associati a qualsiasi altro indirizzo Netsukuku *reale*.

Negli altri network stack, invece, non opera alcun programma. Nemmeno lo stesso demone *ntkd*.
Quindi in essi nessuna applicazione ha bisogno di inviare o ricevere pacchetti IP. Il sistema
deve avere tali network stack soltanto per instradare correttamente pacchetti IP ricevuti
da un altro sistema e indirizzati ad un altro sistema. Quindi in questi network stack,
il sistema non deve avere un indirizzo IP associato all'indirizzo Netsukuku della relativa identità,
ma soltanto le rotte necessarie a instradare pacchetti IP (da inoltrare) verso indirizzi IP
associati a qualsiasi altro indirizzo Netsukuku.

### Indirizzi IP locali

Consideriamo l'identità principale che opera nel network stack default, poiché abbiamo detto che
è la sola che deve assegnare un indirizzo IP al sistema. Essa ha un indirizzo Netsukuku *reale*
in tutte le componenti.

Con la funzione `ip_global_node` essa computa l'indirizzo IP associato al suo indirizzo Netsukuku.
Questo indirizzo IP è quello con cui globalmente, nella rete Netsukuku a cui appartiene, il sistema
può essere identificato.

Con la funzione `ip_anonymizing_node` essa computa l'indirizzo IP associato al suo indirizzo Netsukuku.
Anche questo indirizzo IP identifica a livello globale questo sistema. Ma questo indirizzo inoltre
codifica l'informazione che il client desidera rimanere anonimo.

Con la funzione `ip_internal_node`, per ogni livello *i* da 1 a `levels-1`, essa computa l'indirizzo IP
associato al suo indirizzo Netsukuku internamente al suo g-nodo di livello *i*.
Questo indirizzo IP è quello con cui localmente, nel g-nodo di livello *i* a cui appartiene, il sistema
può essere identificato; esso rimarrebbe tale anche se il suo intero g-nodo di livello *i* (o uno di
livello superiore) migrasse o entrasse in una diversa rete Netsukuku.

Tutti questi indirizzi IP (il globale, l'anonimizzante e gli interni dal livello 1 al
livello `levels-1`) devono essere associati nel network stack default ad ognuna delle interfacce
di rete gestite dal demone *ntkd*.

### Indirizzi IP nelle tabelle di routing

Consideriamo una identità (può essere la principale o una di connettività) che ha un certo indirizzo
Netsukuku. La mappa gerarchica delle possibili destinazioni che il modulo QSPN mantiene, ognuna con
i relativi percorsi noti, è costituita da tutti i possibili g-nodi che possono essere espressi con
coordinate gerarchiche relative al proprio indirizzo Netsukuku.

#### Identità principale

Esaminiamo dapprima il caso di una identità principale. Essa ha tutte le componenti reali e opera
nel network namespace default.  
Ad esempio, se il nostro indirizzo è 1.2.0.3.2 in una topologia a 5 livelli con gsize=4 per tutti i
livelli (dove 2 è la posizione a livello 0 e 1 è la posizione a livello 4) allora le possibili
destinazioni sono questi g-nodi:

*   a livello 0: (0,0), (0,1), (0,3)
*   a livello 1: (1,0), (1,1), (1,2)
*   a livello 2: (2,1), (2,2), (2,3)
*   a livello 3: (3,0), (3,1), (3,3)
*   a livello 4: (4,0), (4,2), (4,3)

Per ognuna di queste destinazioni espressa in forma gerarchica, l'identità ha modo di computare
l'indirizzo Netsukuku del g-nodo destinazione.  
Prendendo come esempio di destinazione il g-nodo espresso con le coordinate gerarchiche (2,3),
abbiamo che esso è il g-nodo di livello 2 le cui componenti a livello più alto sono 1.2.3. Cioè:
al livello 2 la componente è 3. Ai livelli più alti abbiamo le stesse componenti del nostro indirizzo.

Per ogni indirizzo Netsukuku destinazione così calcolato, l'identità deve computare vari indirizzi
IP (il globale, l'anonimizzante e gli interni dal livello direttamente maggiore al livello `levels-1`). Questa
operazione la fa con le funzioni `ip_global_gnode`, `ip_anonymizing_gnode` e `ip_internal_gnode`.

Per ogni indirizzo IP così calcolato, nel network stack default va aggiunta
una rotta in ogni tabella di routing che il demone *ntkd* gestisce. Vedremo che si tratta della
tabella `ntk` per i pacchetti generati localmente e di varie tabelle `ntk_from_XXX` per i pacchetti
da inoltrare.

#### Identità di connettività

Esaminiamo ora il caso di una identità di connettività. Essa ha almeno una componente virtuale e
opera in un diverso network namespace, dove non risiede nessuna applicazione.  
Consideriamo il livello più alto in cui l'indirizzo Netsukuku di questa identità ha la componente
virtuale, indichiamolo con *i*. Questo significa che tutto il g-nodo di livello *i* a cui questa
identità appartiene è interamente composto di identità di connettività. Quindi nessuna di queste
identità può generare localmente un pacchetto IP.  
Di consequenza, un qualsiasi pacchetto IP che arriva in questo network namespace di questo sistema
è stato generato in un diverso g-nodo di livello *i* rispetto al nostro ed è indirizzato ad un
diverso g-nodo di livello *i* rispetto al nostro. In altre parole, in questo network namespace
(il cui compito è solo quello di inoltrare pacchetti IP) nessuna rotta è necessaria se non
quelle verso g-nodi di livello maggiore o uguale a *i*.

Ad esempio, se il nostro indirizzo è 1.5.0.6.2 nella stessa topologia, allora le possibili
destinazioni sono questi g-nodi:

*   a livello 3: (3,0), (3,1), (3,2), (3,3)
*   a livello 4: (4,0), (4,2), (4,3)

Di nuovo, per ognuna di queste destinazioni espressa in forma gerarchica, l'identità ha modo di computare
l'indirizzo Netsukuku del g-nodo destinazione e quindi i vari indirizzi IP, cioè
il globale, l'anonimizzante e gli interni dal livello `i+1` al livello `levels-1`.  
Per ogni indirizzo IP così calcolato, nel relativo network stack, questa identità aggiunge
una rotta in ogni tabella di routing che il demone *ntkd* gestisce. Vedremo che si tratta solo
delle tabelle `ntk_from_XXX` per i pacchetti da inoltrare.

Ulteriori dettagli sul modo di computare gli indirizzi IP associati ad un indirizzo Netsukuku e
sulle operazioni che ogni identità deve fare sul network namespace che gestisce saranno
illustrati nel documento [Indirizzi IP](IndirizziIP.md).

