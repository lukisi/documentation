# Modulo Neighborhood - Analisi Funzionale

1.  [Ruolo del modulo](#ruolo-del-modulo)
1.  [Operazioni di base](#operazioni-di-base)
1.  [Caratteristiche degli archi](#caratteristiche-degli-archi)
1.  [Requisiti](#requisiti)
1.  [Deliverable](#deliverable)
1.  [Classi e interfacce](#classi-e-interfacce)

## Ruolo del modulo

Il ruolo fondamentale del modulo Neighborhood è il rilevamento dei collegamenti (detti *archi*) che si possono
realizzare con altri nodi diretti vicini, la loro realizzazione e la misurazione del costo associato a tali archi.

Un arco associa una specifica interfaccia di rete del nodo corrente con una specifica interfaccia di rete
di un altro nodo diretto vicino. Tecnicamente, con il termine *diretto vicino* si intende che le due interfacce
di rete suddette sono collegate ad un comune [dominio broadcast](https://en.wikipedia.org/wiki/Broadcast_domain).

Come vedremo in seguito, non sono ammessi due archi che partendo da una stessa interfaccia di rete del nodo
corrente colleghino a due distinte interfacce di rete di uno stesso nodo diretto vicino. Per questo ogni arco
deve identificare univocamente non soltanto la specifica interfaccia di rete di un vicino, ma anche il nodo vicino
come entità nel suo insieme.

L'identificativo di un nodo vicino è chiamato NeigborhoodNodeID. Esso è rappresentato in una istanza di una
classe pubblica definita nel modulo Neighborhood: il suo utilizzatore la conosce.  
Il modulo Neighborhood si occupa di realizzare le istanze di NeighborhoodNodeID e fornirle al suo utilizzatore.  
Il modulo costruisce l'istanza di NeighborhoodNodeID che rappresenta il nodo corrente e la rende nota al suo utilizzatore.  
Inoltre, quando segnala un arco al suo utilizzatore gli rende nota l'istanza di NeighborhoodNodeID del peer.

Riassumendo, ogni arco associa una interfaccia di rete del nodo corrente ad una coppia composta
dal NeigborhoodNodeID del vicino e dal MAC address di una sua interfaccia di rete.

## Operazioni di base

Il modulo fa uso delle [tasklet](../Librerie/TaskletSystem.md), un sistema di multithreading
cooperativo. Attraverso di esso esegue il monitoraggio delle schede di rete lasciando libero il chiamante
di svolgere altri task.

Il modulo fa uso del framework [ZCD](../Librerie/ZCD.md), precisamente appoggiandosi alla
libreria di livello intermedio *ntkdrpc* prodotta con questo framework per formalizzare i metodi remoti
usati nel demone *ntkd*.

Quando si inizializza, il modulo produce l'identificativo NeigborhoodNodeID del proprio nodo.

Il modulo riceve subito l'elenco delle interfacce di rete che deve gestire. Ad ognuna associa un indirizzo
IP link-local detto *indirizzo di scheda*.

* * *

Si noti che il fatto di associare un indirizzo IP ad una specifica interfaccia di rete
ha la sua importanza in relazione al protocollo di risoluzione degli indirizzi IP in
indirizzi MAC ([Address Resolution Protocol](https://en.wikipedia.org/wiki/Address_Resolution_Protocol)).
Infatti quando un sistema *a* vuole trasmettere un pacchetto IP ad un suo diretto vicino *b*, esso conosce
l'indirizzo IP di *b* e l'interfaccia di rete di *a* dove trasmettere. Il sistema *a* trasmette un frame Ethernet broadcast
su quel segmento di rete. Consiste nella richiesta: «chi ha l'indirizzo IP XYZ?». Il sistema *b* risponde al sistema *a*
con un frame Ethernet unicast. In questo modo gli indica l'indirizzo MAC della sua interfaccia di rete. Quindi il
sistema *a* può incapsulare il pacchetto IP in un frame Ethernet unicast che riporta gli indirizzi MAC dell'interfaccia
che trasmette e dell'interfaccia che deve ricevere.

* * *

Rileva i nodi vicini raggiungibili attraverso una (o più) interfacce di rete e ne reperisce l'identificativo
NeigborhoodNodeID + MAC. Si veda sotto la trattazione dell'argomento degli archi multipli con un unico nodo
vicino: essi sono ammessi solo se diversi MAC del vicino sono rilevati da diverse interfacce di rete del nodo
corrente. Per verificare questo vincolo è necessario identificare un nodo vicino come singola entità (con il
NeigborhoodNodeID) e non basarsi soltanto sui distinti MAC address.

Con ogni vicino, in modo sincronizzato, crea un arco. Da subito entrambi i nodi
vengono anche a conoscenza dell'*indirizzo di scheda* del vicino.

Quando il modulo crea un nuovo arco, esso imposta le rotte nelle tabelle del kernel per rendere possibile
la comunicazione via TCP con il vicino attraverso gli *indirizzi di scheda*.

Per ogni arco che il modulo nel nodo corrente *a* ha creato con un vicino *b*, la struttura dati che il
modulo mantiene in memoria per l'arco contiene:

*   L'identificativo di *b* (NeighborhoodNodeID).
*   Il MAC address dell'interfaccia di rete di *b*.
*   L'*indirizzo di scheda* associato dal nodo *b* a detta interfaccia.
*   L'interfaccia di rete di *a*.
*   Booleano: se l'arco è esposto.
*   Il costo dell'arco.

Tutti i dati suddetti sono da subito noti al nodo *a*.  
L'arco inizialmente non è esposto dal modulo.  
Il costo dell'arco non è stato misurato. Esso è inizializzato a `NULL`.

Subito dopo i nodi *a* e *b* avviano una comunicazione TCP (reliable) tramite i loro *indirizzi di scheda*.
In essa viene deciso se l'arco vada esposto dal modulo. Uno dei nodi potrebbe rifiutarlo se ha già un numero
di archi elevato: in quel caso anche l'altro nodo deve evitare di esporre l'arco dal modulo.

Se si decide di esporre l'arco, allora entrambi i nodi iniziano a monitorare il costo dell'arco e valorizzare di
conseguenza il dato nella struttura indicata prima. Di fatto il modulo espone l'arco solo dopo che la prima misurazione
del costo ha avuto luogo.

Nel tempo, il modulo gestisce la costituzione di nuovi archi, la rimozione di archi, i cambiamenti del costo
degli archi.

## Caratteristiche degli archi

Quando si crea un arco esso esiste per entrambi i nodi. Quando il modulo espone l'arco, anche nel nodo
collegato il modulo espone l'arco.

* * *

Tra due nodi vicini ci possono essere più collegamenti. Ad esempio i nodi A e B possono avere entrambi una
interfaccia di rete wireless e una ethernet ed essere collegati sia con un cavo (direttamente o per il
tramite di un [hub](https://en.wikipedia.org/wiki/Ethernet_hub)) sia con le interfacce wireless (in
modalità ad-hoc o per il tramite di un access point). Oppure ancora, un nodo C con due interfacce di rete
ethernet può essere collegato tramite due cavi ad un unico [switch](https://en.wikipedia.org/wiki/Network_switch)
al quale è collegato anche il nodo D.

Ci chiediamo: in quali casi può essere utile che i due nodi tengano in considerazione più di un arco? Se
i collegamenti sono su distinti [domini di collisione](https://en.wikipedia.org/wiki/Collision_domain), è
utile considerarli in modo distinto, poiché le trasmissioni fatte su un collegamento non influenzano la
capacità dell'altro collegamento. Ad esempio i nodi A e B possono sfruttare in parallelo i due collegamenti
(uno via cavo e l'altro via etere). Consideriamo il caso del nodo C che ha le due interfacce ethernet
C<sub>0</sub> e C<sub>1</sub> collegate tramite due cavi ad uno switch<sup>1</sup>. Supponiamo che il nodo
D sia collegato allo stesso switch con un solo cavo collegato alla sua interfaccia D<sub>0</sub>. Supponiamo
che sia in corso una trasmissione di dati da D<sub>0</sub> a C<sub>0</sub>. Allora questa trasmissione
influenza la capacità del collegamento tra D<sub>0</sub> e C<sub>1</sub>; quindi non potremmo usare questi
due collegamenti in parallelo in modo efficiente.

Supponiamo, invece, che il nodo D sia collegato allo stesso switch con due cavi collegati alle sue interfacce
D<sub>0</sub> e D<sub>1</sub>. Supponiamo che sia in corso una trasmissione di dati da D<sub>0</sub> a
C<sub>0</sub>. Allora questa trasmissione, per le caratteristiche di uno switch, non influenza la capacità
del collegamento tra D<sub>1</sub> e C<sub>1</sub>; quindi potremmo usare questi due collegamenti in parallelo
in modo efficiente.

Si consideri inoltre che uno switch, pur realizzando distinti domini di collisione, forma un unico
[dominio broadcast](https://en.wikipedia.org/wiki/Broadcast_domain); questo significa che il nodo C tramite
la sua interfaccia C<sub>0</sub> con un messaggio in broadcast rileva la presenza del nodo D anche attraverso
la sua interfaccia D<sub>1</sub>. E la stessa cosa vale per la coppia D<sub>0</sub> e C<sub>1</sub>. Però
non ha senso considerare tutti questi 4 possibili archi: lo sfruttamento ottimale si ha con due archi, ad
esempio D<sub>0</sub>-C<sub>0</sub> e D<sub>1</sub>-C<sub>1</sub>, che possono essere usati per due
trasmissioni in parallelo che non si disturbano a vicenda.

Per avvicinarci allo sfruttamento ottimale descritto sopra, implementiamo questa regola: il modulo consente
la costituzione di un secondo arco tra i nodi A e B solo se le interfacce di rete di entrambi i nodi sono
diverse da quelle su cui è realizzato il primo arco.

*   Nota 1: Si consideri la differenza tra uno switch e un hub. Ovviamente un nodo non è in grado di sapere
se il segmento di rete a cui è collegato tramite una sua interfaccia è supportato da uno switch o da un
semplice hub. Quando un nodo gestisce più di una interfaccia di rete si ipotizza sempre che siano collegate
a diversi domini di collisione o per lo meno ad uno switch. Non avrebbe senso collegare due interfacce
ethernet di un singolo nodo ad un unico hub, perché in nessun caso si potrebbero sfruttare in parallelo.

* * *

Il costo di un arco in una direzione (da A verso B) può essere diverso dal costo nella direzione inversa.
Cioè ogni vertice effettua una misurazione del costo dell'arco indipendente da quella fatta dall'altro
vertice. Questo non è utile quando il costo rappresenta il round-trip time, ma può esserlo se rappresenta
la larghezza di banda in uscita.

* * *

Quando un nodo rimuove un arco tenta di comunicarlo al vertice collegato perché faccia altrettanto.

**TODO: da spostare** in DemoneNTKD/RPC

Facciamo un altro esempio di un modulo *di identità* in cui una precisa identità del nodo *a*, indichiamola
con *a<sub>0</sub>*, vuole chiamare un metodo remoto su un set di precise identità dei nodi diretti vicini,
diciamo *b<sub>0</sub>*, *b<sub>1</sub>*, *c<sub>0</sub>* e *d<sub>0</sub>*. Il nodo *a* chiama un metodo di
Neighborhood per produrre uno stub di tipo *identity_aware_broadcast*. In questo metodo il nodo passa il
NodeID di *a<sub>0</sub>* e questo sarà usato dal modulo Neighborhood per produrre un IdentityAwareSourceID.
Inoltre in questo metodo il nodo passa i vari NodeID di *b<sub>0</sub>*, *b<sub>1</sub>*, *c<sub>0</sub>* e
*d<sub>0</sub>* e questi saranno usati dal modulo Neighborhood per produrre un IdentityAwareBroadcastID. Lo
stub prodotto in questo modo trasmetterà su tutte le interfacce di rete gestite dal nodo *a*. Per il lato
server vedremo sotto come i nodi, ad esempio *b*, *c*, *d* ed anche *e*, avevano istruito ognuno il suo modulo
Neighborhood perché potesse gestire questo tipo di IBroadcastID.

Nelle trasmissioni *broadcast* non sono individuati specifici archi tra *a* e gli altri nodi. Infatti il
messaggio è trasmesso su tutte le interfacce di *a* e contiene come IBroadcastID un oggetto che non individua
gli specifici MAC address dei destinatari. Guardiamo cosa succede alla ricezione da parte di uno dei nodi,
diciamo *b*. Il nodo *b* quando riceve il messaggio trasmesso dalla interfaccia *eth0(a)* del nodo *a*
leggendolo dalla sua interfaccia *eth0(b)*, se esiste l'arco *eth0(a)-eth0(b)*, non può fare altro che
esaminarlo. In particolare, nelle trasmissioni *identity_aware_broadcast* abbiamo aggiunto come informazioni
le identità *a<sub>0</sub>* e *b<sub>0</sub>* e *b<sub>1</sub>*. Il nodo *b* se l'arco *eth0(a)-eth0(b)* è
associato a un *arco-identità* *a<sub>0</sub>-b<sub>0</sub>* deve processarlo su questo *arco-identità*. Allo
stesso modo se l'arco *eth0(a)-eth0(b)* è associato anche su un *arco-identità* *a<sub>0</sub>-b<sub>1</sub>*
deve processarlo anche su questo *arco-identità*. Se i nodi *a* e *b* hanno più di un arco che li collega,
questa cosa può ripetersi per diverse volte a seguito di un solo messaggio broadcast, ad esempio anche
con *wlan0(a)* e *wlan0(b)*.

Nei casi in cui il modulo che vuole comunicare è *di nodo*, il NeigborhoodNodeID costituisce la parte
essenziale delle classi che si usano come ISourceID, IUnicastID e IBroadcastID nella produzione di stub
per chiamare metodi remoti.

Facciamo un esempio di un modulo *di nodo* in cui un nodo *a* vuole chiamare un metodo remoto su un nodo
diretto vicino *b*, passando attraverso l'arco *x*. Il nodo *a* chiama un metodo di Neighborhood per produrre
uno stub di tipo *whole_node_unicast*. In questo metodo il nodo passa un INeighborhoodArc che identifica
l'arco *x* in cui è identificata l'interfaccia di rete di *a* e il MAC address dell'interfaccia di rete di *b*.
Il modulo Neighborhood usa il suo NeigborhoodNodeID per produrre un WholeNodeSourceID. Inoltre dal
INeighborhoodArc *x* si può reperire il NeighborhoodNodeID di *b*; questo sarà usato dal modulo Neighborhood
insieme al MAC address dell'interfaccia di rete di *b* per produrre un WholeNodeUnicastID. Lo stub prodotto
in questo modo trasmetterà su una sola interfaccia di rete, univocamente individuata dal INeighborhoodArc
passato al metodo. Per il lato server vedremo sotto come il nodo *b* aveva istruito il suo modulo Neighborhood
perché potesse gestire questo tipo di IUnicastID.

Come detto prima, nelle trasmissioni *unicast* è sempre individuato un solo arco tra *a* e *b*. Per le
trasmissioni in UDP questo non è scontato. Infatti il messaggio è trasmesso solo su una interfaccia di
*a* ma potrebbe essere rilevato da diverse interfacce di *b*. Per questo dobbiamo usare come IUnicastID un
oggetto che individua uno specifico MAC address di *b*. Quindi il nodo *b* esamina il messaggio solo quando
lo legge da quella particolare interfaccia e non dalle altre. Le trasmissioni unicast fatte dai moduli
*di nodo* possono essere in TCP o in UDP, come vedremo in seguito. In UDP vengono in effetti fatte solo dallo
stesso modulo Neighborhood in due situazioni: quando non ha ancora realizzato l'arco o quando lo ha appena
rimosso.

In tutti i casi, quindi, il nodo *b* esamina il messaggio solo una volta.

Facciamo un altro esempio di un modulo *di nodo* in cui un nodo *a* vuole chiamare un metodo remoto su un
set di nodi diretti vicini, diciamo *b*, *c* e *d*. Il nodo *a* chiama un metodo di Neighborhood per produrre
uno stub di tipo *whole_node_broadcast*. Il modulo Neighborhood usa il suo NeigborhoodNodeID per produrre un
WholeNodeSourceID. Inoltre in questo metodo il nodo passa i vari INeighborhoodArc di *b*, *c* e *d*; i relativi
NeighborhoodNodeID (senza guardare i MAC address) saranno usati dal modulo Neighborhood per produrre un
WholeNodeBroadcastID. Lo stub prodotto in questo modo trasmetterà su tutte le interfacce di rete gestite dal
nodo *a*. Per il lato server vedremo sotto come i nodi, ad esempio *b*, *c*, *d* ed anche *e*, avevano istruito
ognuno il suo modulo Neighborhood perché potesse gestire questo tipo di IBroadcastID.

Come detto prima, nelle trasmissioni *broadcast* non sono individuati specifici archi tra *a* e gli altri
nodi. Quindi ogni nodo che riceve il messaggio potrebbe esaminarlo diverse volte. Diciamo quindi che,
soprattutto per i *moduli di nodo*, le trasmissioni broadcast verso un set di nodi avvengono sempre su tutti
gli archi esistenti.

Ovviamente anche lo stesso modulo Neighborhood, il quale è di per se un modulo *di nodo*, può usare in
completa autonomia le modalità sopra esposte per effettuare chiamate Unicast e Broadcast.

## Requisiti

*   Implementazione del sistema di tasklet.
*   La (o le) interfaccia di rete da monitorare.  
    Durante le operazioni del modulo è possibile aggiungere o rimuovere una interfaccia di rete da monitorare.
*   Numero massimo di archi da realizzare.
*   Factory per creare uno "stub" per invocare metodi remoti nei nodi vicini.
*   Delegato per identificare il chiamante (un nodo vicino) di un metodo remoto invocato su questo nodo.
*   Manager di indirizzi e rotte.
*   Funzione per generare un indirizzo IP link-local.

## Deliverable

*   Emette un segnale per:
    *   `nic_address_set` - Avvenuta assegnazione dell'*indirizzo di scheda* ad una interfaccia di rete gestita.
    *   `arc_added` - Avvenuta costituzione di un arco. Significa anche avvenuto inserimento della rotta nelle tabelle.
    *   `arc_removing` - Il modulo sta per rimuovere un arco. Significa anche che sta per rimuovere la rotta nelle tabelle.  
        Il modulo quando emette questo segnale aspetta che esso sia gestito. Cioè il gestore del segnale
        non dovrebbe avviare una nuova tasklet. Questi infatti deve usare questa opportunità per rimuovere
        tutte le dipendenze di questa rotta diretta.
    *   `arc_removed` - Avvenuta rimozione di un arco. Significa anche avvenuta rimozione della rotta nelle tabelle.
    *   `arc_changed` - Variazione del costo di un arco.
    *   `nic_address_unset` - Avvenuta rimozione dell'*indirizzo di scheda* ad una interfaccia di rete che non si gestisce più.

*   Fornisce metodi per:
    *   `start_monitor` - Aggiunge una interfaccia di rete da monitorare.
    *   `stop_monitor` - Rimuove una interfaccia di rete da monitorare.
    *   `get_my_neighborhood_id` - Restituisce l'identificativo assegnato a questo nodo.
    *   `current_arcs` - Ottenere l'elenco degli archi ora presenti.
    *   `remove_my_arc` - Forzare la rimozione di un arco.

## Classi e interfacce

L'implementazione del sistema di tasklet è passata al modulo dal suo utilizzatore. Si tratta di una istanza dell'interfaccia
ITasklet che è descritta nel relativo [documento](../Librerie/TaskletSystem.md#interfacce).

* * *

Una interfaccia di rete passata al modulo è un oggetto istanza di una classe di cui il modulo conosce l'interfaccia
INeighborhoodNetworkInterface. Tramite questa interfaccia il modulo può:

*   Leggere il nome dell'interfaccia di rete, es: wlan0 (proprietà `dev`).
*   Leggere il MAC address dell'interfaccia di rete, es: CC:AF:78:2E:C8:B6 (proprietà `mac`).
*   Misurare il round-trip time (la latenza) con un vicino (metodo `measure_rtt`).

* * *

La classe usata per l'identificativo di un nodo, cioè NeighborhoodNodeID, è definita nel modulo Neighborhood
e viene resa pubblica. **TODO** Ma solo il modulo ne può creare una nuova istanza (costruttore interno).

Essa è serializzabile secondo la modalità usata in JsonGlib.

* * *

Quando viene creato un arco con un vicino, si crea una istanza di NeighborhoodRealArc.

All'esterno del modulo è nota sola l'interfaccia INeighborhoodArc, implementata da NeighborhoodRealArc.
Questa consente solo la lettura di un sottoinsieme delle informazioni in esso contenute:

*   L'interfaccia di rete dell'arco - proprietà `INeighborhoodNetworkInterface nic`.
*   Il MAC dell'interfaccia di rete del vicino collegata su questo arco - proprietà `string neighbour_mac`.
*   L'indirizzo di scheda dell'interfaccia di rete del vicino collegata su questo arco -
    proprietà `string neighbour_nic_addr`.
*   L'identificativo del nodo come entità nel suo insieme, oltre a quello di una sua interfaccia di rete -
    proprietà `NeighborhoodNodeID neighbour_id`.
*   Il costo dell'arco - proprietà `long cost`. Il costo di un arco può essere espresso con diverse
    metriche (latenza, larghezza di banda, ...). Attualmente l'implementazione del modulo misura la latenza
    e la esprime con un intero in microsecondi.  
    La latenza è il tempo che impiega un messaggio da noi a raggiungere il vertice collegato. In realtà quello
    che si può misurare, quindi quello che il modulo memorizza come costo, è il round-trip time (RTT).

Non tutti gli archi creati vengono comunicati all'esterno del modulo come utilizzabili per l'instradamento
di pacchetti nella rete. Questo avviene con il segnale di avvenuta costituzione di un arco `arc_added`.

Alcuni archi, sebbene non segnalati come detto sopra, sono usati dal modulo che li passa alla
stub factory (che vedremo sotto) per realizzare una comunicazione con il diretto vicino.

* * *

La stub factory è un oggetto di cui il modulo conosce l'interfaccia INeighborhoodStubFactory. Tramite essa il modulo può:

*   Creare uno stub per chiamare un metodo via UDP in broadcast sui nodi vicini (metodo `get_broadcast_for_radar`).  
    Il modulo specifica una interfaccia di rete come istanza di INeighborhoodNetworkInterface, sulla quale
    desidera che lo stub invii il messaggio.  
    Il messaggio è trasmesso senza alcuna affidabilità. Non è previsto alcun modo per verificare la mancata ricezione
    da parte di qualche vicino collegato ad un arco noto che parte da questa interfaccia.
*   Creare uno stub per chiamare un metodo via connessione TCP attraverso uno specifico arco (metodo `get_unicast`).  
    Il modulo specifica l'arco come istanza di INeighborhoodArc.  
    Infine il modulo può specificare se si vuole attendere l'esecuzione del metodo da parte del vicino o no, ma
    comunque se il metodo ritorna senza l'eccezione StubError la ricezione da parte del vicino è garantita.

* * *

Quando su questo nodo viene invocato un metodo remoto, esso è stato chiamato da un diretto vicino. Il delegato
usato per identificare il chiamante è un oggetto di cui il modulo conosce l'interfaccia INeighborhoodQueryCallerInfo.
Infatti essa a partire da un CallerInfo (che viene passato ad ogni invocazione di un metodo remoto) può essere usata
per individuare il chiamante in questi modi:

*   Il metodo remoto `can_you_export` accetta di essere chiamato solo da un diretto vicino con un
    messaggio unicast, e vuole sapere specificamente quale arco è stato usato.  
    In particolare occorre considerare che l'arco in questione non è stato ancora esposto dal
    modulo Neighborhood. Quindi quando viene interrogato a questo scopo il delegato INeighborhoodQueryCallerInfo
    viene fornito, oltre al CallerInfo, anche tutto l'elenco degli archi (istanze di INeighborhoodArc) noti
    al modulo (compresi quelli non ancora esposti).  
    Questa richiesta si fa invocando il metodo `is_from_unicast` di INeighborhoodQueryCallerInfo.
    Se la risposta è positiva il metodo restituisce l'arco tramite il quale la richiesta
    è arrivata.  
    Altrimenti il metodo restituisce *null*.
*   Alcuni metodi remoti (`here_i_am`, `request_arc`, `remove_arc`) accettano di essere chiamati solo da un
    diretto vicino con un messaggio broadcast, e vogliono sapere specificamente su quale delle proprie
    interfacce di rete arco la richiesta è arrivata.  
    Questa richiesta si fa invocando il metodo `is_from_broadcast` di INeighborhoodQueryCallerInfo.
    Se la risposta è positiva il metodo restituisce l'interfaccia di rete (istanza di INeighborhoodNetworkInterface) sulla
    quale la richiesta è arrivata.  
    Altrimenti il metodo restituisce *null*.

* * *

Il manager di rotte e indirizzi è un oggetto di cui il modulo conosce l'interfaccia INeighborhoodIPRouteManager.
Tramite essa il modulo può:

*   Dato il nome di una interfaccia di rete e un indirizzo IP [link-local](https://en.wikipedia.org/wiki/Link-local_address)
    nella dotted form, aggiungere l'indirizzo IP all'interfaccia di rete (metodo `add_address`);
*   Dato il nome di una interfaccia di rete, il suo indirizzo IP link-local associato e un altro indirizzo IP link-local
    nella dotted form, aggiungere la rotta con scope link verso un vicino sull'interfaccia specificando come src
    preferito l'indirizzo di scheda (metodo `add_neighbor`);
*   Dato il nome di una interfaccia di rete, il suo indirizzo IP link-local associato e l'indirizzo IP link-local di
    un vicino, rimuovere la rotta con scope link verso il vicino dall'interfaccia (metodo `remove_neighbor`);
*   Dato il nome di una interfaccia di rete e il suo indirizzo IP link-local associato, rimuovere l'indirizzo IP
    dall'interfaccia (metodo `remove_address`).

Il modulo lo usa per rendere possibile la comunicazione reliable (via TCP) coi diretti vicini tramite un indirizzo IP
che non dipende dalla posizione di essi nella topologia gerarchica della rete. Il modulo
associa ad ogni interfaccia di rete che gestisce un indirizzo detto *indirizzo di scheda*. Per ogni arco che
realizza, il modulo aggiunge la rotta con scope link verso l'indirizzo di scheda dell'interfaccia del vicino
collegata all'arco. Quando rimuove l'arco rimuove anche la rotta. Quando il modulo cessa di gestire un'interfaccia
rimuove il relativo indirizzo.

Quando vuole generare un indirizzo IP link-local il modulo usa una funzione (delegato) come detto nei requisiti.

Il modulo Neighborhood gestisce solo gli indirizzi di scheda dell'*identità principale* del nodo e dei suoi
diretti vicini. (Cioè non si occupa degli indirizzi IP link-local che il nodo vorrà assegnare alle altre *identità*
allo scopo di instradare pacchetti attraverso *identità di connettività*, come delineato nella trattazione del
modulo QSPN)
