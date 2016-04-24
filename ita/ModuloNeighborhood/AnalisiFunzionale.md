# Modulo Neighborhood - Analisi Funzionale

1.  [Ruolo del modulo](#Ruolo_del_modulo)
1.  [Operazioni di base](#Operazioni_di_base)
1.  [Caratteristiche degli archi](#Caratteristiche_degli_archi)
1.  [Identità multiple in un nodo](#Identit.2BAOA_multiple_in_un_nodo)
    1.  [Costituzione della prima identità - Associazioni mantenute dal
        nodo](#Costituzione_della_prima_identit.2BAOA_-_Associazioni_mantenute_dal_nodo)
    1.  [Creazione di una nuova identità](#Creazione_di_una_nuova_identit.2BAOA-)
    1.  [Rimozione di un arco-identità](#Rimozione_di_un_arco-identit.2BAOA-)
1.  [Requisiti](#Requisiti)
1.  [Deliverables](#Deliverables)
1.  [Classi e interfacce](#Classi_e_interfacce)

## <a name="Ruolo_del_modulo"></a>Ruolo del modulo

Il ruolo fondamentale del modulo Neighborhood è il rilevamento dei collegamenti (detti *archi*) che si possono
realizzare con altri nodi diretti vicini, la loro realizzazione e la misurazione del costo associato a tali archi.

Un arco associa una specifica interfaccia di rete del nodo corrente con una specifica interfaccia di rete
di un altro nodo diretto vicino. Tecnicamente, con il termine *diretto vicino* si intende che le due interfacce
di rete suddette sono collegate ad un comune [dominio broadcast](https://en.wikipedia.org/wiki/Broadcast_domain).

Come vedremo in seguito, non sono ammessi due archi che partendo da una stessa interfaccia di rete del nodo
corrente colleghino a due distinte interfacce di rete di uno stesso nodo diretto vicino. Per questo ogni arco
deve identificare univocamente non soltanto la specifica interfaccia di rete di un vicino, ma anche il nodo vicino
come entità nel suo insieme.

L'identificativo di un nodo vicino è chiamato NeigborhoodNodeID. Esso è un concetto interno al modulo
Neighborhood. Riassumendo, ogni arco associa una interfaccia di rete del nodo corrente ad una coppia composta
dal NeigborhoodNodeID del vicino e dal MAC address della sua interfaccia.

Come diretta conseguenza del ruolo di realizzazione e mantenimento degli archi, al modulo Neighborhood è demandato
anche il compito di permettere agli altri moduli la comunicazione tra nodi:

*   Lato client. Il modulo Neighborhood fornisce al suo utilizzatore metodi per produrre gli stub che gli altri moduli dell'applicazione usano per
    comunicare con i nodi (diretti vicini o con indirizzo IP).
*   Lato server. Il modulo Neighborhood viene interrogato dal suo utilizzatore quando esso rileva una richiesta tramite una interfaccia di rete.
    Da questa interrogazione l'utilizzatore saprà se deve passare la richiesta a uno
    (o piu d'uno) skeleton nel nodo corrente, il quale potrà richiamare metodi anche di altri moduli.

## <a name="Operazioni_di_base"></a>Operazioni di base

Il modulo fa uso delle [tasklet](../Librerie/TaskletSystem), un sistema di multithreading
cooperativo. Attraverso di esso esegue il monitoraggio delle schede di rete lasciando libero il chiamante
di svolgere altri task.

Il modulo fa uso del framework [ZCD](../Librerie/ZCD), precisamente appoggiandosi alla
libreria di livello intermedio *ntkdrpc* prodotta con questo framework per formalizzare i metodi remoti
usati nel demone *ntkd*.

Quando si inizializza, il modulo produce l'identificativo NeigborhoodNodeID del proprio nodo.

Il modulo riceve subito l'elenco delle interfacce di rete che deve gestire. Ad ognuna associa un indirizzo
locale detto *indirizzo di scheda*.

Rileva i nodi vicini raggiungibili attraverso una (o più) interfacce di rete e ne reperisce l'identificativo
NeigborhoodNodeID + MAC. Si veda sotto la trattazione dell'argomento degli archi multipli con un unico nodo
vicino: essi sono ammessi solo se diversi MAC del vicino sono rilevati da diverse interfacce di rete del nodo
corrente. Per verificare questo vincolo è necessario identificare un nodo vicino come singola entità (con il
NeigborhoodNodeID) e non basarsi soltanto sui distinti MAC address.

Con ogni vicino, poi, si accorda per la creazione (o meno) di un arco. L'accordo potrebbe non essere raggiunto
perché uno dei due vertici ha già un numero di archi elevato. Se l'accordo viene raggiunto entrambi i nodi
vengono anche a conoscenza dell' *indirizzo di scheda* del vicino.

Sia *k* un arco che il modulo nel nodo corrente *a* ha creato con un vicino *b*. La struttura dati che il
modulo mantiene per l'arco *k* contiene:

*   L'identificativo di *b* (NeighborhoodNodeID).

*   Il MAC address dell'interfaccia di rete di *b*.

*   L' *indirizzo di scheda* associato dal nodo *b* a detta interfaccia.

*   L'interfaccia di rete di *a*.

*   Il costo dell'arco.

Quando il modulo crea un nuovo arco, esso imposta le rotte nelle tabelle del kernel per rendere possibile
la comunicazione via TCP con il vicino attraverso gli *indirizzi di scheda*.

Nel tempo, il modulo gestisce la costituzione di nuovi archi, la rimozione di archi, i cambiamenti del costo
degli archi.

## <a name="Caratteristiche_degli_archi"></a>Caratteristiche degli archi

Quando si crea un arco esso esiste per entrambi i nodi.

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

Il costo di un arco in una direzione (da A verso B?) può essere diverso dal costo nella direzione inversa.
Cioè ogni vertice effettua una misurazione del costo dell'arco indipendente da quella fatta dall'altro
vertice. Questo non è utile quando il costo rappresenta il round-trip time, ma può esserlo se rappresenta
la larghezza di banda in uscita.

* * *

Quando un nodo rimuove un arco tenta di comunicarlo al vertice collegato perché faccia altrettanto.

## <a name="Identit.2BAOA_multiple_in_un_nodo"></a>Identità multiple in un nodo

Introduciamo il concetto di *identità*. In un singolo nodo possono in dati momenti sussistere diverse
*identità*. La ragione di essere di queste identità sarà discussa in dettaglio nella trattazione del modulo
QSPN, quando si parla di nodi virtuali.

Ogni *identità* di un nodo ha un suo identificativo. Questo identificativo è distinto dal
NeigborhoodNodeID, il quale è un concetto interno al modulo Neighborhood. L'identificativo assegnato
ad una *identità* di un nodo lo chiamiamo semplicemente NodeID.

Il NodeID assegnato a ogni *identità* è essenzialmente un intero di 32 bit scelto a caso, assumendo
che sia univoco a livello dei domini di collisione in cui il nodo partecipa con le sue interfacce di rete.
Questo dettaglio implementativo non è di pertinenza del modulo Neighborhood. Il modulo sa solo che il
NodeID può essere usato come identificativo univoco che si riferisce ad una precisa *identità* all'interno
di un preciso nodo.

Ogni nodo ha sempre una e una sola *identità principale*. L'identità principale del nodo non è sempre la
stessa: il nodo può in un certo momento creare una nuova identità e farla diventare la sua principale.
L'identità principale (i concetti espressi in questa frase verranno chiariti in seguito nel documento) è
quella che gestisce le interfacce di rete *reali* del nodo nel *network namespace default*, ognuna con il
suo indirizzo di scheda.

Il modulo Neighborhood non ha conoscenza diretta di quali siano le *identità* che vivono nel suo nodo in un
dato momento.

Abbiamo visto che la presenza di identità multiple in un nodo è supportata dal framework ZCD. Con questo
intendiamo evidenziare che è possibile realizzare uno stub che possa essere usato da una precisa identità
del nodo *a*, indichiamola con *a<sub>0</sub>*, per chiamare un metodo remoto su una precisa identità del
nodo *b*, indichiamola con *b<sub>0</sub>*.

Abbiamo detto inoltre che fra i ruoli del modulo Neighborhood c'è quello di permettere agli altri moduli
dell'applicazione la comunicazione con altri nodi. Ora aggiungiamo che un particolare modulo può essere
interessato a questo tipo di identificazione precisa della identità all'interno di un nodo. Un altro modulo,
invece, potrebbe essere agnostico rispetto a queste identità, e voler semplicemente chiamare un metodo
remoto su un particolare nodo. Chiamiamo il primo tipo un *modulo consapevole di identità* o *modulo di
identità* o *identity-aware*. Il secondo tipo è un *modulo di nodo* o *whole-node*. Di norma in un modulo
*di identità* c'è una classe di cui viene creata una specifica istanza per ogni *identità* che il nodo assume.

Il modulo Neighborhood, lato client, deve saper produrre uno stub per ogni esigenza. Inoltre, lato server,
deve saper individuare un elenco (con zero, uno o più elementi) di root-dispatcher a partire dal discriminatore
contenuto in un messaggio ricevuto (cioè dall'oggetto CallerInfo); questi dispatcher, se il modulo da chiamare
è un *modulo di identità*, devono saper indirizzare ognuno una precisa istanza del modulo da chiamare.
Ricordiamo che l'oggetto CallerInfo è fornito dalla libreria di livello intermedio del framework ZCD prodotta
con "rpcdesign" e che contiene queste informazioni:

*   Un ISourceID.
*   Un IUnicastID o un IBroadcastID.
*   L'indirizzo IP che ha trasmesso il pacchetto. Il modulo Neighborhood lato client farà in modo che nei
    messaggi trasmessi ai diretti vicini (quindi sempre per i messaggi trasmessi in UDP), questo indirizzo
    identifichi univocamente una interfaccia di rete del nodo che trasmette. Il nodo che riceve riconosce
    questo indirizzo se ha già realizzato un arco con quella interfaccia.
*   Nel caso di pacchetti TCP, l'indirizzo IP usato come destinazione. Il modulo Neighborhood lato client
    farà in modo che nei messaggi trasmessi ai diretti vicini, questo indirizzo identifichi univocamente una
    interfaccia di rete del nodo destinatario.
*   Nel caso di pacchetti UDP, l'interfaccia di rete del nodo da cui è stato ricevuto il messaggio.

Nei casi in cui il modulo che vuole comunicare è *di identità*, il NodeID che identifica una *identità* di
un nodo è una parte essenziale delle classi che si usano come ISourceID, IUnicastID e IBroadcastID nella
produzione di stub per chiamare metodi remoti.

Facciamo un esempio di un modulo *di identità* (ad esempio QSPN) in cui una precisa identità del nodo *a*,
indichiamola con *a<sub>0</sub>*, vuole chiamare un metodo remoto su una precisa identità del nodo diretto
vicino *b*, indichiamola con *b<sub>0</sub>*, passando attraverso l'arco *x*. Il nodo *a* chiama un metodo
di Neighborhood per produrre uno stub di tipo *identity_aware_unicast*. In questo metodo il nodo passa un
INeighborhoodArc che identifica l'arco *x* (questo è l'oggetto fornito dal modulo Neighborhood al suo
esterno per rappresentare uno specifico arco formato con un diretto vicino). In questo oggetto è identificata
l'interfaccia di rete di *a* e il MAC address dell'interfaccia di rete di *b*. Inoltre in questo metodo il nodo
passa il NodeID di *a<sub>0</sub>* e questo sarà usato dal modulo Neighborhood per produrre un
IdentityAwareSourceID. Infine in questo metodo il nodo passa il NodeID di *b<sub>0</sub>* e questo sarà usato
dal modulo Neighborhood per produrre un IdentityAwareUnicastID. Lo stub prodotto in questo modo trasmetterà
su una sola interfaccia di rete, univocamente individuata dal INeighborhoodArc passato al metodo. Per il lato
server vedremo sotto come il nodo *b* aveva istruito il suo modulo Neighborhood perché potesse gestire questo
tipo di IUnicastID.

Nelle trasmissioni *unicast* è sempre individuato un solo arco tra *a* e *b*. Per le trasmissioni in TCP questo
è scontato, nel senso che il nodo *b* riceve il messaggio una sola volta attraverso l'interfaccia di rete
specifica di quell'arco. E le trasmissioni unicast fatte dai moduli *di identità* sono sempre in TCP, come
vedremo in seguito.

In particolare, nelle trasmissioni *identity_aware_unicast* abbiamo aggiunto come informazioni le identità
*a<sub>0</sub>* e *b<sub>0</sub>*; grazie a queste il nodo ricevente *b* (non il modulo Neighborhood, bensì
il suo utilizzatore) è in grado di identificare la specifica istanza del *modulo di identità* da coinvolgere
e anche di sapere (grazie a delle associazioni che vedremo meglio sotto) se esiste quello specifico
*arco-identità* così individuato.

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

*   **TODO: serve questo?** Alcuni moduli (ad esempio PeerServices) prevedono la possibilità di chiamare un
metodo remoto su un nodo che non è un diretto vicino ma si può raggiungere con un TcpClient ad un certo
indirizzo IP. In questi casi il destinatario del messaggio è sempre uno solo. Anche in questi casi un
modulo potrebbe essere consapevole della presenza di più *identità* in un nodo e voler individuare una
particolare identità sulla quale chiamare il metodo remoto. In questi casi il nodo chiamante deve essere
a conoscenza non solo dell'indirizzo IP a cui raggiungere il nodo, ma anche del NodeID della identità su
cui operare.

Esaminiamo cosa avviene lato server. Una descrizione passo passo è presente in questo
[documento](ita_ChiamateLatoServer).

Abbiamo anticipato che, per gestire le comunicazioni lato server, all'inizio della sua attività il modulo
Neighborhood deve essere istruito su come gestire le chiamate che riceve. L'utilizzatore del modulo dovrà
dire al Neighborhood:

*   se ricevi un IUnicastID di tipo IdentityAwareUnicastID:
    *   usa questa callback

        `IAddressManagerSkeleton? get_identity_skeleton(NodeID source_id, NodeID unicast_id, string peer_address)`

        la quale restituirà <​null> oppure un root-dispatcher che potrà accedere solo i moduli *di identità*, e
        in particolare la istanza indicata nell'IdentityAwareUnicastID. Si noti che in questo caso non abbiamo
        previsto il parametro *dev*; infatti una chiamata unicast ad un modulo *di identità* è sempre fatta in
        protocollo TCP; solo il modulo Neighborhood fa in alcuni casi chiamate unicast in UDP al suo omologo, ma
        il modulo Neighborhood è un modulo *di nodo*.

*   se ricevi un IBroadcastID di tipo IdentityAwareBroadcastID:
    *   usa questa callback

        `Gee.List<IAddressManagerSkeleton> get_identity_skeleton_set(NodeID source_id, Gee.List<NodeID> broadcast_set, string peer_address, string dev)`

        la quale restituirà una lista (che può essere vuota) di root-dispatcher ognuno dei quali potrà accedere
        solo i moduli *di identità*, e in particolare una precisa istanza. Nell'insieme questi dispatcher
        copriranno tutte le *identità* che sono presenti nel IdentityAwareBroadcastID e vivono in questo nodo.

*   se ricevi un IUnicastID di tipo WholeNodeUnicastID e ti riconosci come destinatario:
    *   usa questa istanza

        `IAddressManagerSkeleton node_skeleton`

        la quale è un root-dispatcher che potrà accedere solo i moduli *di nodo*.

*   se ricevi un IBroadcastID di tipo WholeNodeBroadcastID e ti riconosci fra i destinatari:
    *   usa ancora l'istanza *node_skeleton*.

Il modulo viene subito inizializzato con le callback da chiamare per gestire le chiamate *IdentityAware* e lo
skeleton da usare per le chiamate *WholeNode*. Di modo che è pronto a gestire le chiamate che il nodo potrà ricevere.

Grazie al framework ZCD, quando un metodo remoto viene invocato, l'implementazione del metodo riceve un oggetto
chiamato *CallerInfo*. Con tale oggetto esso può identificare attraverso quale *arco-nodo* o *arco-identità* tale
messaggio è pervenuto. Per realizzare questa associazione il modulo Neighborhood deve fornire essenzialmente due metodi:

*   *NodeID get_identity(ISourceID source_id)* - Solo per i messaggi destinati a moduli *di identità*, dato
    l'identificativo del mittente di un messaggio ricevuto restituisce il NodeID dell' *identità* del mittente.
    Tale dato è contenuto nella classe IdentityAwareSourceID, che è interna al modulo Neighborhood.

*   *INeighborhoodArc get_node_arc(ISourceID source_id, string dev)* - Solo per i messaggi destinati a moduli
    *di nodo*, dato l'identificativo del mittente di un messaggio ricevuto e il nome dell'interfaccia di rete
    su cui il messaggio è stato ricevuto, restituisce l'istanza di arco (INeighborhoodArc). Nella classe
    WholeNodeSourceID, che è interna al modulo Neighborhood, è contenuto il NeighborhoodNodeID.

    Se non è possibile associare questi dati ad un arco, il modulo Neighborhood restituisce null. In teoria
    questo caso si può verificare solo se un *arco-nodo* prima valido (altrimenti il metodo remoto non doveva
    essere affatto invocato) è stato rimosso pochi istanti prima. In questo caso l'esecuzione del metodo
    andrebbe "probabilmente" interrotta, ma questo è di pertinenza del codice che implementa il metodo remoto.

### <a name="Costituzione_della_prima_identit.2BAOA_-_Associazioni_mantenute_dal_nodo"></a>Costituzione della prima identità - Associazioni mantenute dal nodo

Con il termine nodo indichiamo l'utilizzatore del modulo Neighborhood.

Quando il nodo *a* inizia l'attività, assume una *identità* che è la sua *identità principale*. Chiamiamola
*a<sub>0</sub>*. Di fatto questo significa che crea un NodeID e crea una prima istanza di ogni modulo *di identità*.

Il nodo mantiene una associazione *ns* tra questa identità e il network namespace default. Lo indichiamo
dicendo *ns(a<sub>0</sub>) = ""*. La stringa vuota rappresenta il network namespace default, altrimenti avremmo
il nome del network namespace.

Il nodo mantiene una associazione *in* tra questa identità e un'altra associazione. Questa associazione interna
*in(a<sub>0</sub>)* (interfacce gestite da *a<sub>0</sub>*) è tra ogni interfaccia di rete reale gestita dal nodo
e una struttura dati che rappresenta l'interfaccia gestita dall'identità.

Ad esempio: costruiamo la struttura dati *n* che identifica l'interfaccia "eth0". Abbiamo questi membri:

*   *n.dev = "eth0"*.

*   *n.mac = "02:AF:78:2E:C8:B6"*.

*   *n.linklocal = "169.254.201.13"*.

e poi diciamo che questa struttura è associata all'interfaccia reale "eth0" come viene gestita da *a<sub>0</sub>*:

*   *in(a<sub>0</sub>)("eth0") = n*.

Per la *identità principale a<sub>p</sub>* abbiamo sempre che *ns(a<sub>p</sub>) = ""* e
*in(a<sub>p</sub>)("xyz").dev = "xyz"*. Cioè la *identità principale* gestisce le interfacce reali
che sono nel network namespace default.

Quando il modulo Neighborhood forma un arco *i*, questo rappresenta un collegamento tra una interfaccia di rete
reale di *a* e una interfaccia di rete reale del nodo collegato, chiamiamolo *b*. Da questo momento il nodo mantiene
una associazione *f* tra la coppia *a<sub>0</sub>-i* e un set di identità nel nodo *b*.

Dopo aver formato l'arco *i* il nodo viene a conoscenza (come lo fa non è di pertinenza del modulo Neighborhood)
che sopra questo arco devono appoggiarsi *n* *archi-identità* tra *a<sub>0</sub>* e le identità
*b<sub>0</sub>* ... *b<sub>n-1</sub>*. Viene a conoscere inoltre per ognuna di queste identità di *b*, sempre
relativamente all'arco *i*, un MAC-address e un indirizzo di scheda. Memorizza tutti questi dati nell'associazione *f*.

Riassumendo, *f(a<sub>0</sub>-i)* è un set di oggetti che chiamiamo *arco-identità*. Un suo elemento,
diciamo *w*, rappresenta l'arco-identità *a<sub>0</sub>-b<sub>j</sub>* che si appoggia sull'arco *i*.
L'elemento *w* contiene:

*   *w.peer_nodeid* - Il NodeID di *b<sub>j</sub>*.

*   *w.peer_mac* - Il MAC address dell'interfaccia gestita da *b<sub>j</sub>*. Se *b<sub>j</sub>* è
la *identità principale* del nodo *b* allora questo MAC address risulta essere lo stesso che si può reperire
dall'oggetto arco *i*.

*   *w.peer_linklocal* - L' *indirizzo di scheda* dell'interfaccia gestita da *b<sub>j</sub>*. Se *b<sub>j</sub>*
è la *identità principale* del nodo *b* allora questo *indirizzo di scheda* risulta essere lo stesso che si può
reperire dall'oggetto arco *i*.

### <a name="Creazione_di_una_nuova_identit.2BAOA-"></a>Creazione di una nuova identità

Abbiamo detto che la creazione di una nuova *identità* di un nodo non è una scelta del modulo, ma del suo
utilizzatore. Vediamo come questo avviene.

*   Per una comprensione delle motivazioni di queste *identità* e del loro rapporto con gli *indirizzi di scheda* si
veda il documento ["Modulo QSPN - Esempio di uso degli indirizzi virtuali"](../../qspn/wiki/ita_Esempio1_Step1),
in particolare la premessa.

Esaminiamo il caso in cui, a fronte di una migrazione, il nodo corrente *a* crea una nuova identità *a<sub>1</sub>*,
cioè un nuovo NodeID, basata su una sua precedente identità *a<sub>0</sub>*.

*   All'inizio della procedura di migrazione, il nodo prepara alcuni dati. Questi dati vanno individuati o scelti
subito, soprattutto se si tratta della migrazione di un intero cluster che ha altri membri oltre all'identità
*a<sub>0</sub>*, perché durante la procedura potranno essere oggetto di comunicazioni con i diretti vicini che così
verificheranno se appartengono al cluster. Appena il nodo decide che una sua identità migra, subito prepara questo set
di dati che chiamiamo *MigrationData migration_data*:

    *   *migration_id* - Identificativo numerico univoco per la migrazione. Questo è stato in precedenza condiviso
        dai nodi membri del cluster che migra.

    *   *old_id* - L'identità che migra, in questo esempio *a<sub>0</sub>*.

    *   *new_id* - L'identità nuova, in questo esempio *a<sub>1</sub>*. Questo identificativo viene immediatamente scelto.

    *   *devices* - Il nodo prepara una mappa vuota di oggetti *MigrationDeviceData* associati ad una interfaccia di rete reale.

    *   Cioè: `HashMap<string,MigrationDeviceData> devices`.

    *   Per ogni interfaccia di rete reale *r*:

        *   Il nodo crea subito una nuova pseudo-interfaccia *p(r)* che usa fisicamente *r*.

        *   Crea una istanza di *MigrationDeviceData* che contiene:

            *   *real_mac* - L'indirizzo MAC di *r*.

            *   *old_id_new_dev* - Il nome di *p(r)*.

            *   *old_id_new_mac* - L'indirizzo MAC di *p(r)*.

            *   *old_id_new_linklocal* - Un nuovo indirizzo link-local scelto ora che verrà in seguito assegnato a *p(r)*.

        *   Mette tale oggetto in *devices[r]*.

*   Di fatto, la creazione di una nuova identità consiste nell'istanziare un nuovo QspnManager. Cioè: *a<sub>1</sub>* è
    un nuovo NodeID associato ad una nuova istanza di ogni modulo *di identità*, in particolare il modulo QSPN.

*   Se *a<sub>0</sub>* era la *identità principale*, allora *a<sub>1</sub>* diventa la *identità principale*.

*   L'utilizzatore del modulo manteneva l'associazione *ns* tra l'identità *a<sub>0</sub>* e un network namespace
    *n<sub>old</sub>*. Se *a<sub>0</sub>* era la *identità principale*, allora *n<sub>old</sub>* era il network namespace default.

*   L'utilizzatore del modulo, in autonomia, crea un network namespace temporaneo *n<sub>temp</sub>*. Poi aggiorna
    le sue associazioni: alla vecchia identità *a<sub>0</sub>* associa *n<sub>temp</sub>*. Alla nuova identità
    *a<sub>1</sub>* associa *n<sub>old</sub>*. D'ora in poi *a<sub>1</sub>* gestirà le interfacce di rete (reali
    o pseudo) che sono in *n<sub>old</sub>*.

*   In pseudo codice:
    *   *n<sub>temp</sub>* = new network_namespace().

    *   *n<sub>old</sub>* = *ns(a<sub>0</sub>)*.

    *   *ns(a<sub>0</sub>)* = *n<sub>temp</sub>*.

    *   *ns(a<sub>1</sub>)* = *n<sub>old</sub>*.

*   L'utilizzatore del modulo manteneva anche l'associazione *in(a<sub>0</sub>)* (interfacce gestite da *a<sub>0</sub>*)
    tra ogni interfaccia di rete reale gestita dal nodo e l'interfaccia (reale o pseudo) gestita dall'identità.

*   Per ogni interfaccia di rete reale *r*:

    *   Avendo creato la pseudo-interfaccia *p(r) = migration_data.devices[r].old_id_new_dev*, sposta *p(r)* sul
        namespace *n<sub>temp</sub>*: sarà infatti gestita da *a<sub>0</sub>*, mentre quella che prima gestiva
        *a<sub>0</sub>* sarà gestita da *a<sub>1</sub>*.

    *   Associa il nuovo indirizzo link-local *addr(p(r)) = migration_data.devices[r].old_id_new_linklocal* a
        *p(r)* in *ns(a<sub>0</sub>)*, cioè *n<sub>temp</sub>*.

    *   Memorizza le nuove associazioni descritte sopra con questo pseudo-codice:
        *   *in(a<sub>1</sub>)(r)* = *in(a<sub>0</sub>)(r)*.

        *   *in(a<sub>0</sub>)(r)* = new *struttura_dati()*.

        *   *in(a<sub>0</sub>)(r).dev* = *p(r)*.

        *   *in(a<sub>0</sub>)(r).mac* = *mac(p(r)) = migration_data.devices[r].old_id_new_linklocal*.

        *   *in(a<sub>0</sub>)(r).linklocal* = *addr(p(r))*.

*   L'utilizzatore del modulo manteneva anche una associazione *f* tra la coppia *a<sub>0</sub>-i* (formata
    dall'identità *a<sub>0</sub>* e un arco *i* che il modulo Neighborhood aveva realizzato tra il nodo *a*
    e un altro nodo) e un set di identità nel nodo collegato all'arco *i*.

*   Entriamo nel dettaglio dell'associazione *f*. Per ogni arco *i*:

    *   L'arco *i* collega una interfaccia di rete reale di *a* con una interfaccia di rete reale del nodo
        collegato. Sia *b* questo nodo. Chiamiamo *ir(i)* l'interfaccia reale che in *a* è l'end-point dell'arco *i*.

    *   L'identità *a<sub>0</sub>* poteva essere la *principale* oppure no. Se non lo era, allora non gestiva
        l'interfaccia reale *ir(i)* bensì una pseudo-interfaccia. Questa usa fisicamente *ir(i)* ma ha un diverso
        MAC address e un diverso *indirizzo di scheda*. In generale abbiamo detto che indichiamo con
        *in(a<sub>0</sub>)(ir(i))* l'interfaccia (reale o pseudo) gestita da *a<sub>0</sub>* che usa fisicamente
        l'interfaccia di rete reale *ir(i)*.

    *   Nel nodo *b*, analogamente, possono esistere più identità. Una di esse, la *principale*, gestisce
        l'interfaccia di rete reale. Le altre gestiscono ognuna una pseudo-interfaccia che è stata creata sulla
        reale ma ha un diverso MAC address e un diverso *indirizzo di scheda* rispetto ai dati riportati dall'arco *i*.

    *   L'associazione *f* nel nodo *a*, collega la coppia *a<sub>0</sub>-i* a un set contenente zero o una o
        più di queste identità di *b*. Supponiamo che siano *n*. Scriviamo *f(a<sub>0</sub>-i).size = n*.

    *   Per ogni elemento *w* di questo set *f(a<sub>0</sub>-i)*:

        *   Sia *b<sub>j</sub>* l'identità di *b* a cui *w* si riferisce.

        *   Questa struttura dati *w* la chiamiamo per brevità *arco-identità* *a<sub>0</sub>*-*b<sub>j</sub>*.

        *   La struttura dati *w* contiene:

            *   *w.peer_nodeid* - Il NodeID di *b<sub>j</sub>*.

            *   *w.peer_mac* - Il MAC address dell'interfaccia di rete (reale o pseudo) gestita da
                *b<sub>j</sub>* che usa fisicamente l'interfaccia reale di *b* che è l'end-point dell'arco *i*.

            *   *w.peer_linklocal* - L' *indirizzo di scheda* dell'interfaccia (reale o pseudo) gestita da
                *b<sub>j</sub>* che usa fisicamente l'interfaccia reale di *b* che è l'end-point dell'arco *i*.

        *   Altre informazioni su questo *arco-identità*, che sono in realtà valide per tutti gli elementi
            del set *f(a<sub>0</sub>-i)*, sono:

            *   *ns = ns(a<sub>0</sub>)* - Il nome del network namespace gestito da *a<sub>0</sub>*.

            *   *dev = in(a<sub>0</sub>)(ir(i)).dev* - Il nome dell'interfaccia (reale o pseudo) gestita
                da *a<sub>0</sub>* che usa fisicamente l'interfaccia reale di *a* che è l'end-point dell'arco *i*.

            *   *mac = in(a<sub>0</sub>)(ir(i)).mac* - Il MAC dell'interfaccia (reale o pseudo) gestita da
                *a<sub>0</sub>* che usa fisicamente l'interfaccia reale di *a* che è l'end-point dell'arco *i*.

            *   *linklocal = in(a<sub>0</sub>)(ir(i)).linklocal* - L' *indirizzo di scheda* dell'interfaccia
                (reale o pseudo) gestita da *a<sub>0</sub>* che usa fisicamente l'interfaccia reale di *a* che è l'end-point dell'arco *i*.

*   Ci sono da fare delle operazioni per ogni *arco-identità* che parte da *a<sub>0</sub>* ora che la nuova identità
    *a<sub>1</sub>* è stata creata basandosi sulla precedente identità *a<sub>0</sub>*.

    Per l'esattezza, quali operazioni vanno fatte dipende anche dal fatto che l'identità nel nodo collegato
    abbia o meno partecipato anch'essa alla migrazione.

*   Per ogni *arco-identità* *w<sub>0</sub>* che parte da *a<sub>0</sub>*:

    *   Sia *i* l'arco su cui si appoggia *w<sub>0</sub>*. Sia *b* il nodo collegato all'arco *i*.

    *   Sia *b<sub>j</sub>* l'identità collegata a *w<sub>0</sub>*.

    *   Il nodo *a* crea un duplicato *w<sub>1</sub>* dell'arco per assegnarlo ad *a<sub>1</sub>*.
        *w<sub>1</sub> = w<sub>0</sub>.copy(); f(a<sub>1</sub>-i).add(w<sub>1</sub>)*.

    *   Oltre a assegnare il nuovo *arco-identità* alla nuova identità nell'associazione *f*, il nodo passa
        un IQspnArc alla nuova istanza di QspnManager. Quindi abbiamo una stretta relazione tra questa struttura
        dati *w<sub>1</sub>* e l'oggetto IQspnArc. Si potrebbe ipotizzare che è questa struttura a implementare
        l'interfaccia IQspnArc e ad essere passata al QspnManager.

    *   Cambia i dati dell'arco assegnato ad *a<sub>0</sub>* relativamente all'interfaccia locale:

        *   *w<sub>0</sub>.ns = ns(a<sub>0</sub>)*.

        *   *w<sub>0</sub>.dev = in(a<sub>0</sub>)(ir(i)).dev*.

        *   *w<sub>0</sub>.mac = in(a<sub>0</sub>)(ir(i)).mac*.

        *   *w<sub>0</sub>.linklocal = in(a<sub>0</sub>)(ir(i)).linklocal*.

    *   In realtà questi dati sono stati cambiati automaticamente nel momento in cui sono stati cambiati i
        valori nelle associazioni *ns(a<sub>0</sub>)* e *in(a<sub>0</sub>)*.

    *   Il nodo *a* deve comunicare al nodo *b* il *migration_id* visto prima e le informazioni che sono relative
        al *device* su cui è realizzato l'arco *i*. In questa stessa comunicazione il nodo *b*, se la sua identità
        interessata *b<sub>j</sub>* ha partecipato alla stessa migrazione, risponde con le informazioni che sono
        relative al *device*. Nel dettaglio:

        *   Il nodo *a* comunica:

            *   *migration_id* - L'identificativo della migrazione.

            *   *peer_id* - L'identificativo della identità *b<sub>j</sub>* in *b*.

            *   *old_id* - L'identificativo della vecchia identità *a<sub>0</sub>* in *a*.

            *   *new_id* - L'identificativo della nuova identità *a<sub>1</sub>* in *a*.

            *   *old_id_new_mac* - Il MAC della nuova pseudo-interfaccia gestita da *a<sub>0</sub>* per questo arco.
                Cioè *in(a<sub>0</sub>)(ir(i)).mac* ossia l'equivalente della scrittura *w<sub>0</sub>.mac*.

            *   *old_id_new_linklocal* - L'indirizzo link-local della nuova pseudo-interfaccia gestita
                da *a<sub>0</sub>* per questo arco. Cioè *in(a<sub>0</sub>)(ir(i)).linklocal* ossia l'equivalente
                della scrittura *w<sub>0</sub>.linklocal*.

        *   Il nodo *b* risponde:

            *   Se *b<sub>j</sub>* ha partecipato alla stessa migrazione:

                *   *peer_new_id* - L'identificativo della nuova identità frutto della migrazione di *b<sub>j</sub>*,
                    chiamiamola *b<sub>k</sub>*.

                *   *peer_old_id_new_mac* - Il MAC della nuova pseudo-interfaccia gestita da *b<sub>j</sub>* per
                    questo arco.

                *   *peer_old_id_new_linklocal* - L'indirizzo link-local della nuova pseudo-interfaccia gestita da
                    *b<sub>j</sub>* per questo arco.

            *   Altrimenti:
                *   "OK".
    *   Il nodo *a* ora sa se l'identità *b<sub>j</sub>* ha partecipato anch'essa alla migrazione. Se sì, il nodo
        *a* conosce i dati *peer_old_id_new_mac* e *peer_old_id_new_linklocal* che sono riferiti a *b<sub>k</sub>*,
        la quale ora gestisce la vecchia interfaccia del nodo *b* che prima era gestita da *b<sub>j</sub>*.

    *   Se *b<sub>j</sub>* ha partecipato alla migrazione:

        *   Il nodo *a* cambia i dati dell'arco assegnato ad *a<sub>0</sub>* relativamente all'interfaccia remota:

            *   *w<sub>0</sub>.peer_mac = peer_old_id_new_mac*.

            *   *w<sub>0</sub>.peer_linklocal = peer_old_id_new_linklocal*.

        *   Il nodo *a* cambia i dati dell'arco assegnato ad *a<sub>1</sub>* relativamente alla *identità* remota:

            *   *w<sub>1</sub>.peer_nodeid* = Il NodeID di *b<sub>k</sub>*.

    *   Il nodo *a* aggiunge alle tabelle nel network namespace *ns(a<sub>0</sub>)* la rotta verso
        *w<sub>0</sub>.peer_linklocal* partendo da *old_id_new_linklocal* su *in(a<sub>0</sub>)(ir(i)).dev*.

    *   Il nodo *b* a sua volta, ora sa che l'identità *a<sub>0</sub>* in *a* ha migrato e ha dato luogo a
        *a<sub>1</sub>*; sa anche che *a<sub>0</sub>* aveva un *arco-identità* con *b<sub>j</sub>* appoggiato sul
        suo arco *i*; ovviamente sa anche se l'identità *b<sub>j</sub>* ha partecipato anch'essa alla migrazione
        formando *b<sub>k</sub>*.

    *   Se *b<sub>j</sub>* ha partecipato alla migrazione:

        *   Il nodo *b* non ha bisogno di fare nulla in questo momento. Le sue variazioni le apporterà di sua iniziativa
            poiché anche in esso è avvenuta la migrazione di *b<sub>j</sub>* in *b<sub>k</sub>*.

    *   Altrimenti:

        *   Il nodo *b*, come vedremo in dettaglio poco più sotto, forma un nuovo *arco-identità* *b<sub>j</sub>*-*a<sub>1</sub>*;
            cambia i dati dell' *arco-identità* *b<sub>j</sub>*-*a<sub>0</sub>*, cioè MAC e linklocal; aggiunge una rotta nelle
            tabelle di un suo namespace (quello gestito da *b<sub>j</sub>*) per l'arco *b<sub>j</sub>*-*a<sub>0</sub>*.

Tutte queste operazioni non coinvolgono direttamente il modulo Neighborhood. Esso resta comunque in grado, ricevendo
dall'utilizzatore i NodeID aggiornati, di produrre uno stub per moduli *di identità* per comunicare dalla sua nuova identità
ad uno o più diretti vicini. Resta anche in grado, dato un messaggio ricevuto che è per moduli *di identità*, di identificare,
per mezzo delle callback ricevute all'inizio, se è per la sua nuova identità e da parte di chi.

Esaminiamo il caso in cui un nodo vicino *b* crea una nuova identità *b<sub>1</sub>* basata su una sua precedente identità
*b<sub>0</sub>*. La precedente identità *b<sub>0</sub>* era collegata attraverso un arco *i* (o più di uno) alla identità del
nodo corrente *a<sub>k</sub>*, la quale non è cambiata. Sia *w* l' *arco-identità* *a<sub>k</sub>*-*b<sub>0</sub>* che si
appoggia su *i*.

*   Il nodo *a* riceve sull'arco *i* da parte del nodo *b* la comunicazione di cui sopra
    (*migration_id*, *peer_id*, *old_id*, *new_id*, *old_id_new_mac*, *old_id_new_linklocal*) dalla quale deduce che
    sono cambiati gli estremi dell' *arco-identità* *a<sub>k</sub>*-*b<sub>0</sub>*.

*   Il nodo *a* riconosce *a<sub>k</sub>* in *peer_id*. Inoltre riconosce *b<sub>0</sub>* in *old_id* e ritrova nelle sue
    associazioni l' *arco-identità* *a<sub>k</sub>*-*b<sub>0</sub>* che si appoggia sull'arco *i*.

*   Il nodo *a* inoltre viene a conoscenza del NodeID di *b<sub>1</sub>* (*new_id*) con il quale dovrà formare un nuovo
    *arco-identità* *a<sub>k</sub>*-*b<sub>1</sub>*. Questo *arco-identità* avrà per MAC e linklocal i valori che aveva
    l' *arco-identità* *a<sub>k</sub>*-*b<sub>0</sub>*.

*   Il nodo *a* inoltre viene a conoscenza dei nuovi valori MAC e linklocal che ora andranno cambiati nell' *arco-identità*
    *a<sub>k</sub>*-*b<sub>0</sub>*.

*   Il nodo *a*, poiché l'identità *a<sub>k</sub>* non ha partecipato alla migrazione *migration_id*, risponde subito
    semplicemente "OK".

*   Cerca nell'elenco che ha nell'associazione *f(a<sub>k</sub>-i)* l' *arco-identità* *w* che lo collega a *b<sub>0</sub>*.

*   Memorizza *old_peer_linklocal = w.peer_linklocal*.

*   Memorizza *old_peer_mac = w.peer_mac*.

*   Aggiunge sulle tabelle di routing di *ns(a<sub>k</sub>)* la rotta che collega *in(a<sub>k</sub>)(ir(i)).linklocal*
    a *old_id_new_linklocal*.

*   Aggiorna tutte le rotte sulle tabelle di routing di *ns(a<sub>k</sub>)* che usano *w* come gateway. Ora dovranno
    avere *old_id_new_linklocal*.

*   Mantiene nelle tabelle di routing di *ns(a<sub>k</sub>)* la rotta che collega *in(a<sub>k</sub>)(ir(i)).linklocal*
    a *old_peer_linklocal*. Questa ora servirà il nuovo *arco-identità* di cui sotto.

*   Aggiorna i dati:
    *   *w.peer_mac = old_id_new_mac*.

    *   *w.peer_linklocal = old_id_new_linklocal*.

*   Il nodo *a* crea un nuovo *arco-identità* sull'arco *i* da *a<sub>k</sub>* a *b<sub>1</sub>* che avrà come valori
    linklocal e MAC *old_peer_linklocal* e *old_peer_mac*. Lo aggiunge all'elenco che ha nell'associazione *f(a<sub>k</sub>-i)*.

Tutte queste operazioni non coinvolgono direttamente il modulo Neighborhood. Esso resta comunque in grado, ricevendo
dall'utilizzatore i NodeID aggiornati, di produrre uno stub per moduli *di identità* per comunicare da una sua identità
alla nuova identità del vicino *b*. Resta anche in grado, dato un messaggio ricevuto che è per moduli *di identità*, di
identificare, per mezzo delle callback ricevute all'inizio, se è per una sua identità da parte della nuova identità del
vicino *b*.

### <a name="Rimozione_di_un_arco-identit.2BAOA-"></a>Rimozione di un arco-identità

In un certo momento, l'utilizzatore del modulo decide che una certa *identità* del nodo corrente *a<sub>k</sub>* non deve
avere più archi verso una certa *identità* di un suo vicino *b<sub>j</sub>*; quindi per ogni arco che il modulo
Neighborhood aveva creato tra *a* e *b*, l'utilizzatore del modulo fa alcune operazioni per rimuovere gli *archi-identità*
*a<sub>k</sub>*-*b<sub>j</sub>*.

In precedenza l'utilizzatore del modulo si era dovuto occupare di rimuovere (o cambiare) tutte le rotte che usavano quell'arco
come gateway.

*   Per ogni arco *i*:

    *   Il nodo *a* comunica sull'arco *i* al nodo *b* che sta rimuovendo il suo *arco-identità* *a<sub>k</sub>*-*b<sub>j</sub>*
        sull'arco *i* affinché anche il nodo *b* apporti le modifiche alle sue associazioni.

    *   Cerca nell'elenco che ha nell'associazione *f(a<sub>k</sub>-i)* l' *arco-identità* *w* che lo collega a *b<sub>j</sub>*.

    *   Elimina dalle tabelle di routing di *ns(a<sub>k</sub>)* la rotta che collegava
        *in(a<sub>k</sub>)(ir(i)).linklocal* a *w.peer_linklocal*.

    *   Elimina *w* dall'elenco *f(a<sub>k</sub>-i)*.

Tutte queste operazioni non coinvolgono direttamente il modulo Neighborhood.

## <a name="Requisiti"></a>Requisiti

*   Implementazione del sistema di tasklet.
*   Delegati per la gestione delle chiamate ricevute:
    *   *get_identity_skeleton* - Una callback per gestire gli IdentityAwareUnicastID.

    *   *get_identity_skeleton_set* - Una callback per gestire gli IdentityAwareBroadcastID.

    *   *node_skeleton* - Uno skeleton del root-dispatcher per gestire gli WholeNodeUnicastID e gli WholeNodeBroadcastID.

*   La (o le) interfaccia di rete da monitorare.
*   Durante le operazioni del modulo è possibile aggiungere o rimuovere una interfaccia di rete da monitorare.
*   Numero massimo di archi da realizzare.
*   Factory per creare uno "stub" per invocare metodi remoti nei nodi vicini.
*   Manager di indirizzi e rotte.

## <a name="Deliverables"></a>Deliverables

*   Emette un segnale per:
    *   Avvenuta assegnazione dell' *indirizzo di scheda* ad una interfaccia di rete gestita.

    *   Costituzione di un arco. Significa anche avvenuto inserimento della rotta nelle tabelle.
    *   Rimozione di un arco. Significa anche avvenuta rimozione della rotta nelle tabelle.
    *   Variazione del costo di un arco.
    *   Avvenuta rimozione dell' *indirizzo di scheda* ad una interfaccia di rete che non si gestisce più.

*   Fornisce metodi per:
    *   *current_arcs* - Ottenere l'elenco degli archi ora presenti.

    *   *get_dispatcher* - Dati alcuni parametri relativi ad un messaggio in unicast ricevuto (IUnicastID,
        ISourceID, peer_address, dev), eventualmente servendosi dei delegati passati per associare i NodeID
        ad uno skeleton, dopo aver verificato di processare il messaggio solo una volta (in caso di protocollo
        UDP in presenza di molteplici interfacce di rete nel nodo corrente collegate su un solo broadcast domain),
        restituisce una istanza di skeleton del root-dispatcher se il messaggio è da processare, oppure null.

    *   *get_dispatcher_set* - Dati alcuni parametri relativi ad un messaggio in broadcast ricevuto (IBroadcastID,
        ISourceID, peer_address, dev), eventualmente servendosi dei delegati passati per associare i NodeID ad uno
        skeleton, restituisce una lista (che può essere vuota) di istanze di skeleton del root-dispatcher: il messaggio
        è da processare su ognuna di queste istanze.

    *   *get_identity* - Solo per i messaggi destinati a moduli *di identità*, dato l'identificativo del mittente di
        un messaggio ricevuto, cioè una istanza di ISourceID, ottenere il NodeID dell' *identità* del mittente. Se
        non è possibile ottenerlo, restituisce null.

    *   *get_node_arc* - Solo per i messaggi destinati a moduli *di nodo*, dato l'identificativo del mittente di un
        messaggio ricevuto, cioè una istanza di ISourceID, e il nome dell'interfaccia di rete su cui il messaggio è
        stato ricevuto, ottenere l'istanza di arco (INeighborhoodArc). Se non è possibile ottenerlo, restituisce null.

    *   *get_stub_identity_aware_unicast* - Dato un arco che collega questo nodo, chiamiamolo *a*, ad un altro nodo
        vicino, chiamiamolo *b*, dato l'identificativo di una *identità* che risiede in *a*, chiamiamola
        *a<sub>k</sub>*, dato l'identificativo di una *identità* che risiede in *b*, chiamiamola *b<sub>j</sub>*,
        ottenere un oggetto stub utilizzabile per chiamare un metodo remoto su un modulo *di identità* da *a<sub>k</sub>*
        a *b<sub>j</sub>*. Questo stub dialoga con il nodo remoto con protocollo reliable.

    *   *get_stub_whole_node_unicast* - Dato un arco che collega questo nodo, chiamiamolo *a*, ad un altro nodo vicino,
        chiamiamolo *b*, ottenere un oggetto stub utilizzabile per chiamare un metodo remoto su un modulo *di nodo* da
        *a* a *b*. Questo stub dialoga con il nodo remoto con protocollo reliable.

    *   *get_stub_identity_aware_broadcast* - Dato l'identificativo di una *identità* che risiede in *a*, chiamiamola
        *a<sub>k</sub>*, dato un set di identificativi di *identità* che risiedono in alcuni nodi vicini, ottenere
        un oggetto stub utilizzabile per inviare un messaggio in broadcast destinato a un modulo *di identità* di queste *identità*.

    *   Il modulo produrrà uno stub che si occuperà di inviare il messaggio in broadcast su tutte le interfacce di rete gestite.
    *   Quando si invia un messaggio tramite questo oggetto l'invio del messaggio è asincrono: procederà in una nuova
        tasklet, mentre il metodo non fornirà alcuna risposta al chiamante. E' possibile fornire un oggetto in cui un
        determinato metodo (callback) verrà richiamato dopo un certo tempo se per qualcuno degli archi noti al modulo
        non si avrà ricevuto un messaggio di ACK dal vicino collegato. Questo controllo viene fatto sugli archi che sono
        esistenti al momento dell'invio **e** sono ancora presenti alla scadenza del timeout. Il metodo callback viene chiamato
        una volta per ogni arco che fallisce e avrà quell'arco come argomento, così che il chiamante possa prendere un
        provvedimento, ad esempio riprovando con diverse chiamate unicast reliable. Si noti che in questo caso ad un arco
        passato alla callback possono corrispondere diversi *archi-identità*.

    *   *get_stub_whole_node_broadcast* - Dato un set di archi che collegano ad alcuni nodi vicini, ottenere un oggetto
        stub utilizzabile per inviare un messaggio in broadcast destinato a un modulo *di nodo* di questi nodi vicini.

    *   Il modulo produrrà una istanza di IBroadcastID che indicherà come destinatari i nodi che sono identificati
        dagli archi che sono stati passati a questo metodo.
    *   Il modulo poi produrrà uno stub che si occuperà di inviare il messaggio in broadcast su tutte le interfacce di rete gestite.
    *   Quando si invia un messaggio tramite questo oggetto l'invio del messaggio è asincrono e non reliable: si veda la
        spiegazione del metodo precedente. Tuttavia in questo caso ad un arco passato alla callback non corrispondono
        diversi *archi-identità*.

    *   *remove_my_arc* - Forzare la rimozione di un arco.

## <a name="Classi_e_interfacce"></a>Classi e interfacce

L'implementazione del sistema di tasklet è passata al modulo dal suo utilizzatore. Si tratta di una istanza dell'interfaccia
ITasklet che è descritta nel relativo [documento](../../tasklet-system/wiki/ita_TaskletSystem#Interfacce).

* * *

Una interfaccia di rete passata al modulo è un oggetto istanza di una classe di cui il modulo conosce l'interfaccia
INeighborhoodNetworkInterface. Tramite questa interfaccia il modulo può:

*   Leggere il nome dell'interfaccia di rete, es: wlan0 (proprietà *dev*).

*   Leggere il MAC address dell'interfaccia di rete, es: CC:AF:78:2E:C8:B6 (proprietà *mac*).

*   Misurare il round-trip time (la latenza) con un vicino (metodo *measure_rtt*).

* * *

La stub factory è un oggetto di cui il modulo conosce l'interfaccia INeighborhoodStubFactory. Tramite essa il modulo può:

*   Creare uno stub per chiamare un metodo via UDP in broadcast sui nodi vicini (metodo 'get_broadcast').
*   Il modulo specifica una o più interfacce di rete, ciascuna con l'indirizzo da usare come *source*, sulle quali
    desidera che lo stub invii il messaggio.

*   Inoltre il modulo specifica l'oggetto ISourceID che lo stub includerà nel messaggio come identificativo della
    *identità* del mittente.

*   Inoltre il modulo specifica l'oggetto IBroadcastID che lo stub includerà nel messaggio. In questo modo viene
    indicato a ogni vicino che riceve il messaggio se debba considerarsi tra i destinatari (con una o più delle
    sue *identità*).

*   Infine il modulo può indicare un'istanza dell'interfaccia IAckCommunicator se vuole ricevere dopo il timeout la
    lista dei MAC address che hanno segnalato con un ACK la ricezione del messaggio. Tale interfaccia è fornita dalla
    libreria di livello intermedio *ntkdrpc* prodotta per usare il framework ZCD.

*   Creare uno stub per chiamare un metodo via UDP su uno specifico vicino (metodo 'get_unicast').
*   Il modulo specifica l'interfaccia di rete, con l'indirizzo da usare come *source*, sulla quale desidera che lo stub invii il messaggio.

*   Inoltre il modulo specifica l'oggetto ISourceID che lo stub includerà nel messaggio come identificativo della *identità* del mittente.

*   Inoltre il modulo specifica l'oggetto IUnicastID che lo stub includerà nel messaggio per indicare a ogni vicino che
    lo riceve se è lui (una sua *identità*) il destinatario.

*   Infine il modulo può specificare se si vuole attendere l'esecuzione del metodo da parte del vicino o no. Se no,
    allora la corretta ricezione del messaggio da parte del vicino **non** è garantita.

*   Il modulo usa questa modalità per comunicare con un vicino quando ancora non è stata negoziata la creazione dell'arco
    e quindi non è ancora possibile realizzare la connessione via TCP.
*   Creare uno stub per chiamare un metodo via TCP su uno specifico indirizzo (metodo 'get_tcp').
*   Il modulo specifica l'indirizzo di scheda associato all'arco.
*   Inoltre il modulo specifica l'oggetto ISourceID che lo stub includerà nel messaggio come identificativo della *identità* del mittente.

*   Inoltre il modulo specifica l'oggetto IUnicastID che lo stub includerà nel messaggio per indicare al nodo
    ricevente quale sia (fra le sue *identità*) il destinatario.

*   Infine il modulo può specificare se si vuole attendere l'esecuzione del metodo da parte del vicino o no, ma
    comunque se il metodo ritorna senza l'eccezione StubError la ricezione da parte del vicino è garantita.

*   Il modulo usa questa modalità per comunicare in modo reliable con un nodo vicino attraverso un suo arco.

* * *

Il manager di rotte e indirizzi è un oggetto di cui il modulo conosce l'interfaccia INeighborhoodIPRouteManager.
Tramite essa il modulo può:

*   Dato il nome di una interfaccia di rete e un indirizzo IP [link-local](http://en.wikipedia.org/wiki/Link-local_address)
    nella dotted form, aggiungere l'indirizzo IP all'interfaccia di rete (metodo 'add_address');

*   Dato il nome di una interfaccia di rete, il suo indirizzo IP link-local associato e un altro indirizzo IP link-local
    nella dotted form, aggiungere la rotta con scope link verso un vicino sull'interfaccia specificando come src
    preferito l'indirizzo di scheda (metodo 'add_neighbor');
*   Dato il nome di una interfaccia di rete, il suo indirizzo IP link-local associato e l'indirizzo IP link-local di
    un vicino, rimuovere la rotta con scope link verso il vicino dall'interfaccia (metodo 'remove_neighbor');
*   Dato il nome di una interfaccia di rete e il suo indirizzo IP link-local associato, rimuovere l'indirizzo IP
    dall'interfaccia (metodo 'remove_address').

Il modulo lo usa per rendere possibile la comunicazione via TCP coi vicini tramite un indirizzo fisso. Il modulo
associa ad ogni interfaccia di rete che gestisce un indirizzo detto *indirizzo di scheda*. Per ogni arco che
realizza, il modulo aggiunge la rotta con scope link verso l'indirizzo di scheda dell'interfaccia del vicino
collegata all'arco. Quando rimuove l'arco rimuove anche la rotta. Quando il modulo cessa di gestire un'interfaccia
rimuove il relativo indirizzo.

Tramite questo meccanismo il modulo gestisce solo gli indirizzi di scheda della *identità principale* del nodo
corrente, nel network namespace default. Allo stesso modo, esso imposta le rotte verso gli indirizzi di scheda della
*identità principale* di ogni nodo vicino, sempre nel network namespace default. Per la gestione delle altre *identità*,
sia come indirizzi propri sia come rotte verso gli indirizzi dei vicini, il nodo le gestisce in autonomia, senza
l'intervento del modulo Neighborhood.

* * *

La classe usata per l'identificativo di una identità, cioè NodeID, è definita nella libreria
[Common](../../ntkd-common/wiki/ita_Common). Il modulo Neighborhood ha una dipendenza su questa libreria,
quindi conosce tale classe.

La conoscenza del modulo Neighborhood relativamente a tale classe si limita al fatto di sapere che essa è
serializzabile secondo la modalità usata in JsonGlib.

* * *

La classe usata per l'identificativo di un nodo, cioè NeighborhoodNodeID, è interna al modulo Neighborhood.
Anche essa è serializzabile secondo la modalità usata in JsonGlib.

* * *

Un arco è un oggetto (NeighborhoodRealArc) noto al modulo. Grazie alle informazioni memorizzate in esso
(my_nic, mac) il modulo è in grado di evitare la creazione di ulteriori archi verso lo stesso vicino se non
usano interfacce di rete distinte da ambo i lati. Sempre con le informazioni memorizzate in questo oggetto
(nic_addr) il modulo è in grado di produrre lo stub che realizza la chiamata di un metodo remoto in TCP (reliable).

L'interfaccia dell'oggetto arco nota all'esterno del modulo, INeighborhoodArc, permette solo un sottoinsieme di operazioni:

*   Leggere il MAC dell'interfaccia di rete del vicino collegata su questo arco (proprietà 'neighbour_mac').
*   Leggere l'indirizzo di scheda dell'interfaccia di rete del vicino collegata su questo arco (proprietà 'neighbour_nic_addr').
*   Leggere il costo dell'arco (proprietà 'cost').
*   Leggere l'interfaccia di rete dell'arco (proprietà 'nic').

* * *

Quando si chiama il metodo che produce uno stub per l'invio di messaggi in broadcast, può essere passato un
oggetto che contiene il codice e i dati necessari a gestire l'evento di 'mancata ricezione di un ACK da un
arco entro il timeout'. Tale oggetto implementa l'interfaccia INeighborhoodMissingArcHandler. L'interfaccia permette di:

*   lanciare il codice che gestisce una arco mancante, passandogli l'arco (metodo 'missing').

* * *

Il costo di un arco può essere espresso con diverse metriche (latenza, larghezza di banda, ...). Attualmente
l'implementazione del modulo misura la latenza e la esprime con un intero in microsecondi.

La latenza è il tempo che impiega un messaggio da noi a raggiungere il vertice collegato. In realtà quello
che si può misurare, quindi quello che il modulo memorizza come costo, è il round-trip time (RTT).
