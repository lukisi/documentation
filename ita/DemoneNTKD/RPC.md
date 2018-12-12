# ntkd - RPC

1.  [Tipi di trasmissione](#tipi-di-trasmissione)
1.  [Tipi di medium](#tipi-di-medium)
1.  [Lato server](#lato-server)
1.  [Lato client](#lato-client)
1.  [Diversi casi di messaggi](#diversi-casi-di-messaggi)

Il progetto *ntkd* usa il framework ZCD per realizzare le comunicazioni tra nodi.

## Tipi di trasmissione

Abbiamo analizzato nel documento [ZCD](../Librerie/ZCD.md#Trasmissioni) che il framework ZCD prevede due tipi
di trasmissione:

*   "stream", per i messaggi unicast.
*   "datagram", per i messaggi broadcast.

Vediamo tutti i possibili casi usati nel demone *ntkd*.

La modalità unicast-stream è usata per invocare un metodo remoto (cioè inviare
un messaggio) su un diretto vicino e ottenere una risposta precisa in queste circostanze:

*   Neighborhood: `can_you_export` - parte della fase radar scan.
*   Neighborhood: `nop` - periodico controllo su un arco.
*   TODO

La modalità unicast-stream è usata per invocare un metodo remoto (cioè inviare
un messaggio) su un diretto vicino e solo accertarsi della ricezione (`wait_reply=false`) in queste circostanze:

*   TODO

La modalità broadcast-datagram è usata per inviare un messaggio sui diretti vicini
senza richiedere un ACK in queste circostanze:

*   Neighborhood: `here_i_am` - parte della fase radar scan.
*   Neighborhood: `request_arc` - parte della fase radar scan.
*   Neighborhood: `remove_arc` - rimozione di un arco (richiesta dall'altro capo).
*   TODO

La modalità broadcast-datagram è usata per inviare un messaggio sui diretti vicini
richiedendo da loro un ACK in queste circostanze:

*   TODO

Quando si inviano messaggi trasmessi in broadcast che richiedono un ACK, lo scopo è reagire al caso
in cui da un arco noto al presente nodo (o più di uno) non si riceve il messaggio di ACK relativo.  
La reazione in teoria potrebbe essere di vario tipo. Nella maggior parte dei casi (**TODO** in tutti?)
il demone *ntkd* reagisce ritrasmettendo lo stesso messaggio all'arco in difetto con la
modalità unicast-stream, con `wait_reply=false`. Questa modalità (su un diretto vicino) è
usata in queste circostanze:

*   TODO

## Tipi di medium

Abbiamo analizzato nel documento [ZCD](../Librerie/ZCD.md#Medium) che il framework ZCD prevede due tipi
di medium per la trasmissione:

*   "net", per comunicazioni tra nodi di una rete.
*   "system", per comunicazioni tra processi di un sistema.

Per quanto concerne il demone *ntkd*, il supporto al medium "system" (in aggiunta al classico medium "net")
ha lo scopo di facilitare la produzione di testsuite (per i vari moduli o per l'intero demone *ntkd*)
in cui più processi (all'interno di un sistema) simulano più nodi (all'interno di una rete).

Per trasmettere un messaggio da un processo ad un preciso altro processo si usano i socket unix-domain
associati ad un pathname. I pathname per identificare questi socket devono essere univoci all'interno della testsuite.

Vediamo dunque come opera il framework ZCD nella modalità "stream" e nella modalità "datagram" e come
si ottiene nel caso di medium "system" un risultato analogo a quello con il medium "net".

#### Emulazione modalità stream

Stare in ascolto per connessioni su un indirizzo IP significa che la tasklet attende una connessione su
un socket con porta "well-known" a quell'indirizzo. Quando arriva una connessione viene avviata una tasklet che gestisce
quella specifica connessione, mentre la tasklet originale torna ad attendere. La tasklet che gestisce la
connessione può leggere e scrivere sul socket connesso.

Quindi stare in ascolto per connessioni su un pathname è del tutto simile. La tasklet attende una connessione su
un socket con quel pathname. Quando arriva una connessione viene avviata una tasklet che gestisce
quella specifica connessione, mentre la tasklet originale torna ad attendere. La tasklet che gestisce la
connessione può leggere e scrivere sul socket connesso.

Trasmettere per connessioni su un indirizzo IP significa che viene tentata una connessione da parte
di un socket con porta "effimera" verso un socket con porta "well-known" a quell'indirizzo. Stabilita
la connessione il client può leggere e scrivere sul socket connesso.

Quindi trasmettere per connessioni su un pathname è del tutto simile. Viene tentata una connessione da parte
di un socket unnamed verso un socket con quel pathname. Stabilita
la connessione il client può leggere e scrivere sul socket connesso.

Per simulare un socket in ascolto per connessioni su un suo indirizzo IP:

*   Se l'indirizzo IP `<ip>` è un linklocal, ipotiziamo che sia univoco nell'intera testsuite.  
    Allora il pathname sarà `conn_<ip>`.
*   Se l'indirizzo IP `<ip>` è routabile, potremmo avere una testsuite in cui si prevedono diverse
    reti che vengono in contatto fra di loro. Oppure all'interno della stessa rete potremmo
    avere indirizzi IP interni ad un g-nodo, che sono univoci solo dentro il g-nodo. In entrambi i
    casi l'indirizzo potrebbe essere non univoco nell'intera testsuite.

    *   Se l'indirizzo IP è globale (o anonimizzante) sia `<netid>` l'identificativo della rete.
    *   Se è interno ad un g-nodo di livello *i* sia `<netid>` l'identificativo del g-nodo di livello *i*.

    In entrambi i casi assumiamo come **requisito che tali identificativi non cambiano**
    **nel tempo di vita della testsuite**.  
    Allora il pathname sarà `conn_<netid>_<ip>`.
    
#### Emulazione modalità datagram

Stare in ascolto per messaggi broadcast su un nic significa che la tasklet ascolta i pacchetti che transitano
su un dominio broadcast sul quale è "collegata" una sua interfaccia di rete. Questo si fa con un socket
impostato a "set_broadcast" e associato ad un certo nic.

Trasmettere un pacchetto tramite una propria interfaccia di rete si fa con
un socket impostato a "set_broadcast" e associato ad un certo nic. Il risultato è che quel pacchetto 
transita sul dominio broadcast sul quale è "collegato" quel nic. Quindi viene rilevato da qualsiasi altro
nic collegato allo stesso dominio broadcast.

Per simulare un socket che opera (in ascolto o in trasmissione) con messaggi broadcast su un suo pseudo-NIC:

*   Sia `<dev>` l'identificativo univoco di quel NIC (in realtà pseudo-NIC) all'interno del nodo (in realtà del processo).  
    Sia `<pid>` l'identificativo del processo. Però, invece di usare l'identificativo del processo assegnato dal
    Sistema Operativo, lasciamo che sia l'utente a specificarlo quando lancia l'applicazione. In questo modo l'utente
    conoscendolo in anticipo può indicarlo a un altro programma, come illustreremo a breve.  
    Allora **una parte** del pathname sarà `<pid>_<dev>`.

Per simulare queste operazioni in una testsuite in cui più processi (all'interno di un sistema) simulano più
nodi (all'interno di una rete) occorre attivare anche altri processi "domain" che emulano un dominio di broadcast.

Ad esempio si voglia simulare un nodo "alfa" (a cui assegneremo il pid 1234) e un nodo "beta"
(a cui assegneremo il pid 6543) i quali entrambi hanno una interfaccia "eth0" sullo
stesso dominio, che chiameremo dominio "tau". Vogliamo lanciare il comando "qspntester" su entrambi i nodi (simulati)
indicando l'interfaccia "eth0".

Per emulare il dominio "tau", si lancia `eth_domain -i 1234_eth0 -i 6543_eth0`.  
Questo processo avvia una tasklet associata a `1234_eth0`. Questa **crea** il pathname `send_1234_eth0`. Poi si mette in
ascolto con un socket unix-domain su quel pathname. Quando questa tasklet riceve un pacchetto fa da proxy:
avviando una nuova tasklet per ogni pathname (anche lo stesso NIC che trasmette un pacchetto
può al contempo riceverlo) che gli è stato passato come argomento, trasmette lo
stesso pacchetto scrivendolo con un socket unix-domain sul pathname `recv_<pathname>`, **ma solo** se questo
pathname esiste già.  
Inoltre, in modo analogo, il processo avvia una tasklet associata a `6543_eth0` che si mette in ascolto
sul pathname `send_6543_eth0`. Anche questa quando riceve un pacchetto fa da proxy.

Per "alfa", si lancia `qspntester -p 1234 -I eth0`. Tale processo computa un pathname `1234_eth0` per
rappresentare il suo pseudonic `eth0`.  
Quando questo processo vuole stare in ascolto per messaggi broadcast su questo pseudonic, in realtà **crea** il
pathname `recv_1234_eth0` e si mette in ascolto con un socket unix-domain su quel pathname.  
Per il momento nessuno scrive su `recv_1234_eth0`.  
Quando questo processo vuole trasmettere un pacchetto tramite questo pseudonic, in realtà lo scrive con
un socket unix-domain sul pathname `send_1234_eth0`, **ma solo** se questo pathname esiste già.  
Abbiamo visto pocanzi che il processo `eth_domain` ascolta su `send_1234_eth0`.

Per "beta", si lancia `qspntester -p 6543 -I eth0`. Tale processo computa un pathname `6543_eth0` per
rappresentare il suo pseudonic `eth0`.  
Quando questo processo vuole stare in ascolto per messaggi broadcast su questo pseudonic, in realtà **crea** il
pathname `recv_6543_eth0` e si mette in ascolto con un socket unix-domain su quel pathname.  
Per il momento nessuno scrive su `recv_6543_eth0`.  
Quando questo processo vuole trasmettere un pacchetto tramite questo pseudonic, in realtà lo scrive con
un socket unix-domain sul pathname `send_6543_eth0`, **ma solo** se questo pathname esiste già.  
Abbiamo visto pocanzi che il processo `eth_domain` ascolta su `send_6543_eth0`.

Adesso supponiamo che il nodo "beta" vuole scrivere un pacchetto sul suo pseudonic "eth0". Lo
scrive con un socket unix-domain sul pathname `send_6543_eth0`. Quindi il processo `eth_domain`
lo riceve tramite la sua tasklet associata a `6543_eth0` e lo copia sul pathname `recv_1234_eth0`.
Quindi il nodo "alfa" lo riceve.  
Anche il nodo "beta" stesso lo riceve: deve saper riconoscere che è stato trasmesso da lui stesso
e quindi (probabilmente) ignorarlo.

Si intuisce facilmente, sebbene abbiamo fatto un esempio con due soli nodi, che questo meccanismo
consente di emulare efficacemente un dominio in cui un nodo trasmette in broadcast un pacchetto
e questo viene rilevato da *n* altri nodi.

#### Emulazione radio

Supponiamo di voler emulare un dominio radio. Questi hanno una particolare caratteristica: supponiamo che il
nodo beta ha un nic radio collegato allo stesso dominio broadcast (stesso canale, stesso SSID) a
cui è collegato il nodo alfa e il nodo gamma; allora se beta è in grado di comunicare sia con alfa sia con
gamma questo non implica che il nodo alfa sia in grado di comunicare direttamente con il nodo gamma.

Si può fare con un processo (non adatto a simulare la mobilità dei nodi) che sappia fin dall'inizio per una
particolare scheda di rete di un particolare processo, quali altre interfacce di rete di altri processi
possono ricevere i pacchetti trasmessi dalla prima.  
Ad esempio, assumiamo che i nodi alfa beta e gamma di cui sopra abbiano i pid rispettivamente 1234,
6543 e 6789. Lanciamo:

*   `radio_domain -i 1234_wlan0 -o 6543_wlan0`.  
    Crea `send_1234_wlan0` e si mette in ascolto su esso.  
    Quando riceve:
    *   prova a scrivere su `recv_1234_wlan0` ma soltanto se già esiste.
    *   prova a scrivere su `recv_6543_wlan0` ma soltanto se già esiste.

*   `radio_domain -i 6543_wlan0 -o 1234_wlan0 -o 6789_wlan0`.  
    Crea `send_6543_wlan0` e si mette in ascolto su esso.  
    Quando riceve:
    *   prova a scrivere su `recv_6543_wlan0` ma soltanto se già esiste.
    *   prova a scrivere su `recv_1234_wlan0` ma soltanto se già esiste.
    *   prova a scrivere su `recv_6789_wlan0` ma soltanto se già esiste.

*   `radio_domain -i 6789_wlan0 -o 6543_wlan0`.  
    Crea `send_6789_wlan0` e si mette in ascolto su esso.  
    Quando riceve:
    *   prova a scrivere su `recv_6789_wlan0` ma soltanto se già esiste.
    *   prova a scrivere su `recv_6543_wlan0` ma soltanto se già esiste.

Il framework ZCD fornisce di default anche questi due tool (`radio_domain` e `eth_domain`).
Essi sono usati in alcune testsuite di ZCD, ma potranno essere utili allo sviluppatore anche nella produzione
di testsuite per altri moduli di Netsukuku.

## Lato server

Una classe **SkeletonFactory** è usata per avviare le tasklet in ascolto.

Quando viene creata l'istanza di `SkeletonFactory` questa crea una istanza di `ServerDelegate` che
implementa `IDelegate`. Quando poi l'istanza di `SkeletonFactory` viene usata per chiamare i metodi
`??_??_listen`, a questi viene passata l'istanza di `ServerDelegate`.

Quando si rileva una richiesta tramite una interfaccia di rete,
la classe `ServerDelegate` in `get_addr_set` usa a sua volta la stessa `SkeletonFactory` per
individuare (`get_dispatcher` se in modalità stream, `get_dispatcher_set` se in datagram)
gli skeleton da invocare.

## Lato client

### StubFactory

Una classe **StubFactory** è usata per produrre gli stub che i vari moduli dell'applicazione usano per
comunicare con altri nodi (diretti vicini o specifici nodi all'interno di un comune g-nodo).

### Chiamate a metodi remoti

**TODO**

#### IAckCommunicator

Quando si chiama il metodo che produce uno stub per l'invio di messaggi in broadcast (`get_addr_broadcast`), può essere passato un
oggetto che implementa l'interfaccia `IAckCommunicator` fornita dal `ntkdrpc`. Ogni volta che verrà usato lo
stub per trasmettere un messaggio in broadcast, dopo un certo timeout dal momento della trasmissione,
questo oggetto riceverà (nel metodo `process_src_nics_list`) un elenco dei NIC (interfacce di rete rappresentate
da istanze di ISrcNic) che ci hanno notificato (con un pacchetto di "ACKnowledgement") la ricezione del nostro messaggio.  
La logica è che questo oggetto ha le informazioni necessarie a gestire il fatto che una certa interfaccia di rete
(di un vicino che conosciamo) non ha trasmesso l'ACK nel tempo previsto.

Attualmente, gli unici casi di questo tipo nel demone *ntkd* sono relativi a messaggi broadcast tra moduli di identità,
come vedremo sotto nell'analisi di dettaglio dei diversi casi.  
Quindi più specificamente la logica è che questo oggetto gestisce i singoli archi-identità collegati agli
archi dai quali non abbiamo ricevuto conferma di ricezione.  
L'oggetto passato è un `AcknowledgementsCommunicator`, una classe privata dello StubFactory. Al suo interno
ha un `NodeMissingArcHandlerForIdentityAware` cioè un gestore degli archi-nodo che sono stati "mancati"
dalla trasmissione broadcast che sa che deve operare sugli archi-identità. Questo a sua volta al suo interno ha
una istanza dell'interfaccia `IIdentityAwareMissingArcHandler` cioè un gestore degli archi-identità che sono
stati "mancati" dalla trasmissione broadcast.

L'oggetto `AcknowledgementsCommunicator` può accedere (sia al momento della produzione dello stub sia
al momento dell'esecuzione del metodo `process_src_nics_list`) alla lista di archi-nodo che il demone *ntkd*
conosce. Ogni arco-nodo ha un identificativo `ISrcNic` dell'interfaccia di rete del vicino che è confrontabile
con gli identificativi che sono passati al metodo `process_src_nics_list` dal framework ZCD.  
Tutto ciò permette di chiamare una serie di volte (ognuna in una tasklet indipendente) il metodo
`missing(IdentityData identity_data, IdentityArc identity_arc)` della `IIdentityAwareMissingArcHandler`
quando non si riceve nel tempo previsto un ACK da una certa interfaccia.

Le istanze di `IIdentityAwareMissingArcHandler` usate nel codice sono:

*   `MissingArcHandlerForPeers` nel file `peers_helpers.vala`. Usata nel `PeersNeighborsFactory.i_peers_get_broadcast`
    che il modulo usa per chiamare i metodi remoti `set_participant` e `give_participant_maps`.
*   `MissingArcHandlerForQspn` nel file `qspn_helpers.vala`. Usata nel `QspnStubFactory.i_qspn_get_broadcast`
    che il modulo usa per chiamare i metodi remoti `send_etp`, `got_prepare_destroy` e `got_destroy`.

## Diversi casi di messaggi

Ci sono diversi casi di messaggi usati nel demone *ntkd*. In alcuni casi si ricorre al concetto
di *identità* all'interno di un nodo, in altri casi no. In alcuni casi i messaggi sono diretti
ad un particolare vicino, in altri a tutti i vicini, in altri ad un nodo non diretto vicino ma dentro
lo stesso g-nodo. Nei vari casi cambiano le classi usate come ISourceID, ISrcNic, IUnicastID e IBroadcastID.

Qui esaminiamo, nei diversi possibili casi, il percorso del messaggio nella sua interezza: chiamata, ricezione,
esecuzione, risposta.

Per prima cosa parliamo delle *identità* all'interno di un nodo.

### ZCD

Il framework ZCD supporta la presenza di molteplici *identità* all'interno di un nodo. Con questo
intendiamo evidenziare che è possibile realizzare uno stub che possa essere usato da una precisa identità
del nodo *a*, indichiamola con *a<sub>0</sub>*, per chiamare un metodo remoto su una precisa identità del
nodo *b*, indichiamola con *b<sub>0</sub>*.

Ricordiamo che il framework ZCD identifica il mittente e il destinatario (o i destinatari) di ogni
messaggio attraverso delle classi che sono rappresentate con le interfacce ISourceID, IUnicastID
e IBroadcastID.  
Inoltre, nel caso di messaggi scambiati tra diretti vicini, identifica l'interfaccia di rete da cui
il messaggio è trasmesso e l'interfaccia di rete a cui è diretto. Questo lo fa attraverso una classe
rappresentata con l'interfaccia ISrcNic per il mittente e attraverso la specifica tasklet che rileva
il messaggio nel destinatario (identificata nell'oggetto Listener del CallerInfo).  
Lato client permette di costruire degli stub che trasmettono messaggi (cioè chiamano procedure remote)
permettendo al suo utilizzatore di specificare un ISourceID, un ISrcNic, un IUnicastID, un IBroadcastID,
oltre che indicare una connessione (`peer_ip` o `send_pathname`) o una propria interfaccia di
rete (`my_dev` o `send_pathname`).  
Lato server alla recezione di un messaggio costruisce un CallerInfo, indicando in esso la tasklet in
ascolto (Listener) e gli oggetti indicati nel messaggio (ISourceID, ISrcNic, IUnicastID, IBroadcastID).
Sulla base del CallerInfo il suo utilizzatore potrà individuare zero/uno/molti skeleton
a cui far eseguire una procedura.

I delegati passati al framework ZCD sono in grado di riconoscere gli oggetti ISourceID, ISrcNic, IUnicastID/IBroadcastID
ricevuti e quindi sanno se il messaggio è pertinente ad un nodo nella sua interezza o ad una identità
all'interno di un nodo.  
Se il messaggio è per nodi, allora ogni nodo che lo recepisce può individuare zero o uno skeleton da
attivare. Se invece è per identità, allora ogni nodo che lo recepisce può individuare zero/uno/molti skeleton da
attivare.

### ntkd

Il progetto *ntkd* utilizza le *identità*. In un singolo nodo della rete possono in dati momenti sussistere diverse
*identità*. La ragion d'essere di queste identità è discussa in dettaglio nella trattazione del modulo
*QSPN*, quando si parla di nodi virtuali. La gestione di queste identità è demandata al modulo *Identities*.

Nella rete Netsukuku ogni nodo ha un suo identificativo univoco, chiamato NeigborhoodNodeID. Inoltre
ogni *identità* di un nodo ha un suo identificativo univoco, chiamato NodeID.  
**N.B.** Fare attenzione alla nomenclatura che, per motivi storici, può indurre in errore: la
classe NodeID identifica una identità, non un nodo.  
Relativamente alla trattazione di questo documento ci interessa solo evidenziare che
in ogni nodo della rete Netsukuku esiste sempre una e una sola *identità principale*.

L'applicazione *ntkd* è composta da moduli tra loro indipendenti. Ogni modulo gestisce un particolare singolo
aspetto della problematica.  
Nello svolgere le sue funzioni, un modulo può rappresentare una particolare identità nel nodo e comunicare
con altre specifiche identità nei nodi vicini. Un altro modulo invece può rappresentare il
nodo nella sua interezza e comunicare con i nodi vicini.  
Chiamiamo il primo tipo un *modulo consapevole di identità* o *modulo di identità*
o *identity-aware*. Il secondo tipo è un *modulo di nodo* o *whole-node*. Di norma in un modulo
*di identità* c'è una classe di cui viene creata una specifica istanza per ogni *identità* che il nodo assume.

Per la cronaca, nell'applicazione *ntkd* abbiamo:  
moduli di nodo: *Neighborhood*, *Identities*.  
moduli di identità: *QSPN*, *PeerServices*, *Coordinator*, *Hooking*, *Andna*.

Il progetto *ntkd*, lato client, deve saper produrre su richiesta dei moduli uno stub per ogni esigenza,
costruendo gli appositi oggetti ISourceID, ISrcNic, IUnicastID/IBroadcastID e chiamando i metodi del framework ZCD.

Inoltre lato server, attraverso i delegati passati al framework ZCD, partendo dagli oggetti ISourceID, ISrcNic,
IUnicastID/IBroadcastID ricevuti deve saper individuare zero/uno/molti skeleton da attivare. Se il messaggio
era per identità, allora ognuno di questi skeleton è una precisa istanza del modulo interessato.

Ricordiamo che i delegati passati al framework ZCD ricevono una istanza della classe CallerInfo,
fornita dalla libreria di livello intermedio del framework ZCD prodotta con "rpcdesign". Essa contiene fra l'altro
un ISourceID, un ISrcNic e un IUnicastID/IBroadcastID.

Nei casi in cui il modulo che vuole comunicare è *di nodo*, il NeighborhoodNodeID (che identifica univocamente
un nodo nella rete Netsukuku) è una parte essenziale delle classi che si usano come ISourceID, IUnicastID e IBroadcastID.  
Nei casi in cui il modulo che vuole comunicare è *di identità*, il NodeID (che identifica univocamente una *identità* di
un nodo della rete) è una parte essenziale delle suddette classi.

### Moduli di identità

#### Unicast - diretto vicino

Facciamo un esempio di un modulo *di identità* in cui una precisa identità del nodo *a*,
indichiamola con *a<sub>0</sub>*, vuole chiamare un metodo remoto su una precisa identità del nodo diretto
vicino *b*, indichiamola con *b<sub>0</sub>*, passando attraverso l'arco *x*.

##### chiamata lato client

Il nodo *a* chiama il metodo `get_stub_identity_aware_unicast` dello *StubFactory*. Gli passa un
INeighborhoodArc che identifica l'arco *x*, il quale collega il nodo *a* con il nodo *b* attraverso
due specifiche interfacce di rete. Inoltre gli passa il NodeID di *a<sub>0</sub>* e il NodeID di *b<sub>0</sub>*.
Infine specifica se si vuole attendere una risposta al messaggio.  
Lo stub prodotto in questo modo trasmetterà il messaggio in modalità reliable, cioè con un socket di tipo stream.
Sia i dati di trasmissione che i dati di ack (del protocollo TCP) transiteranno per le due interfacce di rete
identificate dall'arco *x*, cioè quella nel nodo locale e quella nel nodo diretto vicino. Questo perché come
indirizzo di destinazione si userà l'indirizzo IP linklocal associato a quell'interfaccia di rete del vicino.  
La trasmissione avrà caratteristiche analoghe anche qualora si trattasse di una trasmissione sul medium "system"
con un socket unix-domain.

L'oggetto che identifica l'arco *x* contiene:

*   il NeighborhoodNodeID `neighbour_id` del vicino.
*   il MAC `neighbour_mac` e l'indirizzo IP linklocal `neighbour_nic_addr` del vicino.
*   il mio INeighborhoodNetworkInterface `my_nic`.

Il demone *ntkd* conosce la classe che implementa `INeighborhoodNetworkInterface my_nic` e
da esso sa identificare un dev o pseudodev del nodo *a*.

Quindi il metodo `get_stub_identity_aware_unicast` può individuare i parametri che servono a
realizzare una connessione con `get_addr_stream_net` o `get_addr_stream_system`. In particolare nel caso `_net`
avremo una connessione TCP che si appoggia esattamente sull'arco *x*. Infatti al momento
della realizzazione dell'arco il modulo Neighborhood nei due nodi avrà impostato una apposita rotta con scope *link*
nelle tabelle del kernel del *network namespace default*.

Di seguito elenchiamo le informazioni che verranno convogliate al nodo *b* attraverso il protocollo del framework ZCD
(oltre naturalmente al messaggio vero e proprio che verrà indicato in seguito allo stub) e che quindi devono
venire specificate dal demone *ntkd* alla creazione dello stub.

*   `ISourceID source_id`. Identifica il mittente.  
    Dal NodeID di *a<sub>0</sub>* verrà prodotto un IdentityAwareSourceID.
*   `IUnicastID unicast_id`. Identifica il destinatario.  
    Dal NodeID di *b<sub>0</sub>* verrà prodotto un IdentityAwareUnicastID.
*   `ISrcNic src_nic`. Identifica il NIC usato dal nodo mittente.  
    Dal INeighborhoodNetworkInterface `x.my_nic` verrà prodotto un NeighbourSrcNic.
*   Se lo stub resta in attesa della risposta. Booleano `wait_reply`.

##### ricezione lato server

Nel nodo *b*, a ricevere il messaggio è una tasklet avviata dal framework ZCD con la funzione `stream_net_listen`
o `stream_system_listen`. Questa prepara una istanza di CallerInfo specifica per i messaggi unicast.  
Nell'oggetto CallerInfo il framework ZCD include le informazioni convogliate nel protocollo del
framework ZCD e anche la conoscenza della modalità con cui era in ascolto la tasklet che ha recepito
il messaggio. In questo caso un ascolto di connessioni su uno specifico indirizzo IP linklocal
(associato ad una specifica nostra interfaccia di rete) oppure su uno specifico pathname.
Questa informazione è nel campo `Listener listener` dell'oggetto CallerInfo: può essere un
StreamNetListener (con i campi `my_ip` e `tcp_port`) o un StreamSystemListener (con il campo `listen_pathname`).

Poi passa il CallerInfo ad un delegato fornito dal demone *ntkd* (un `Netsukuku.IDelegate`) il cui compito
è di reperire un set di istanze skeleton su cui il messaggio va processato.  
Le operazioni complete che si svolgono lato server quando viene recepito un messaggio
sono illustrate nel documento [RPCLatoServer](../DemoneNTKD/RPCLatoServer.md).
Nel presente documento invece ci concentriamo sull'esame di un caso alla volta nella sua interezza.

Il delegato riconosce dal CallerInfo che si tratta di un
messaggio unicast, quindi chiama il metodo `get_dispatcher` dello *SkeletonFactory*. Gli passa
le informazioni presenti nel CallerInfo, cioè quelle convogliate nel protocollo del framework ZCD.  
In questo metodo lo SkeletonFactory, che conosce le classi IdentityAwareSourceID e IdentityAwareUnicastID
e sa recuperarne il NodeID di *a<sub>0</sub>* e quello di *b<sub>0</sub>*, è in grado di identificare
la specifica identità di *b* da coinvolgere.  
Inoltre conosce gli archi-identità che sono associati a *b<sub>0</sub>*; per ognuno di essi sa vedere
su quale arco formato dal modulo Neighborhood questo si appoggia; per un arco formato dal
modulo Neighborhood sa vedere l'identificativo del NIC del vicino.  
Se riesce a individuare un arco-identità tra *b<sub>0</sub>* e *a<sub>0</sub>* che poggia
su un arco in cui il NIC del vicino è lo stesso del ISrcNic specificato in questo messaggio,
restituirà il `identity_skeleton` specifico per l'identità *b<sub>0</sub>*.

Il delegato restituisce lo skeleton (sarà di sicuro uno solo in questo caso, ma generalmente un set)
al framework ZCD, il quale potrà chiamarne i metodi che referenziano la giusta
istanza del modulo di identità interessato dal messaggio.

##### esecuzione lato server

Quando viene chiamato il metodo specificato dal messaggio sull'istanza dello skeleton del modulo
interessato, agli argomenti specifici del messaggio viene aggiunto il CallerInfo.  
Il modulo potrà usarlo per chiedere al demone *ntkd* (ad esempio attraverso un delegato o una
interfaccia) di identificare l'arco-identità tramite il quale il messaggio è stato consegnato.

Prendiamo ad esempio il metodo `i_qspn_comes_from(CallerInfo rpc_caller)` dell'interfaccia `IQspnArc`
come viene usato dallo skeleton del modulo Qspn nel metodo remoto `get_full_etp`.  
Il metodo remoto `get_full_etp` in realtà può essere invocato dai diretti vicini sia con un messaggio
unicast, sia con un messaggio broadcast. Le operazioni del metodo `i_qspn_comes_from` sono comunque
le stesse in entrambi i casi.

Durante l'esecuzione del metodo remoto `get_full_etp` invocato dal framework ZCD, il QspnManager conosce
gli archi-identità di *b<sub>0</sub>* che gli sono stati passati come oggetti che implementano l'interfaccia IQspnArc.
Prende uno di essi e vuole chiedere al demone *ntkd* se il CallerInfo indica che il messaggio
è stato consegnato proprio attraverso di esso. Quindi ne chiama il metodo `i_qspn_comes_from(rpc_caller)`.

In questo metodo il demone *ntkd* sa che il mittente del messaggio è una identità di un nodo
diretto vicino. Chiama sullo SkeletonFactory il metodo `from_caller_get_identityarc` e gli passa
il CallerInfo e l'istanza di IdentityData che rappresenta l'identità associata alla
istanza di QspnManager che ha fatto richiesta.  
In questo metodo lo SkeletonFactory presume che il chiamante sia un diretto vicino, ma ammette
che possa aver fatto una trasmissione unicast o broadcast. In ogni caso, sa riconoscere il
tipo di CallerInfo e estrapolarne il ISourceID.  
Lo SkeletonFactory presume che questo sia un IdentityAwareSourceID (se non lo fosse restituirebbe *null*)
e da esso recupera il NodeID dell'identità mittente.  
Lo SkeletonFactory recupera inoltre dal CallerInfo l'identificativo della nostra interfaccia
su cui il messaggio è stato ricevuto (informazione nel campo `listener` del CallerInfo) e
l'identificativo dell'interfaccia del mittente che esso ha usato
per trasmetterlo (informazione nel NeighbourSrcNic del CallerInfo).  
Con queste informazioni il metodo `from_caller_get_identityarc` individua e restituisce l'arco-identità
su cui il messaggio è stato ricevuto da *b<sub>0</sub>*.

Al ricevere questo arco-identità, il metodo `i_qspn_comes_from` sa vedere se l'arco IQspnArc in
esame è proprio quello che rappresenta questo arco-identità.

##### risposta dal server

Al termine dell'esecuzione del metodo remoto, se lo stub che aveva trasmesso il messaggio aveva
indicato nel protocollo ZCD di restare in attesa del risultato, il framework ZCD trasmette
il risultato nella stessa connessione in cui aveva ricevuto il messaggio.

#### Unicast - nodo dentro un g-nodo

Facciamo un esempio di un modulo *di identità* in cui una precisa identità del nodo *a*,
indichiamola con *a<sub>0</sub>*, vuole chiamare un metodo remoto su una precisa identità del nodo *b*,
il quale non è (necessariamente) un nostro diretto vicino ma un nodo di cui conosciamo l'indirizzo
che lo identifica univocamente all'interno di un nostro comune g-nodo.

In questo caso il demone *ntkd* del nodo *a* non conosce gli identificativi delle identità nel nodo
*b*. Essi infatti sono resi noti attraverso il modulo di nodo Identities solo ai nodi diretti vicini.
Nel demone *ntkd* si è prevista solo la possibilità di comunicare attraverso questa modalità tra
le due identità principali di due nodi. Quindi in questo caso abbiamo che *a<sub>0</sub>* è
l'identità principale di *a* e vuole comunicare con l'identità principale di *b*, chiamiamola
*b<sub>0</sub>*, della quale non conosce l'identificativo.

##### chiamata lato client

Il nodo *a* chiama il metodo `get_stub_main_identity_unicast_inside_gnode` dello *StubFactory*. Gli passa
l'indirizzo di *b* interno ad un suo g-nodo, sottoforma di lista di posizioni ai livelli da 0 al livello
subito inferiore del livello del g-nodo comune.  
Infine specifica se si vuole attendere una risposta al messaggio.  
Lo stub prodotto in questo modo trasmetterà il messaggio in modalità reliable, cioè con un socket di tipo stream.
Il percorso dei dati di trasmissione e dei dati di ack (del protocollo TCP) sarà individuato sulla base
delle rotte nelle tabelle di routing del kernel dei vari nodi nel g-nodo, associate agli indirizzi IP routabili
interni al g-nodo comune tra *a* e *b*.  
La trasmissione avrà caratteristiche analoghe anche qualora si trattasse di una trasmissione sul medium "system"
con un socket unix-domain.

Quindi il metodo `get_stub_main_identity_unicast_inside_gnode` può computare, sulla base della lista di
posizioni passata, l'indirizzo routabile interno al g-nodo comune: cioè i parametri che servono a
realizzare una connessione con `get_addr_stream_net` o `get_addr_stream_system`. In particolare nel caso `_system`
il metodo è a conoscenza del `<netid>` che identifica il g-nodo comune.

Vengono prodotti in questo metodo un MainIdentitySourceID e un MainIdentityUnicastID. Queste due classi
non contengono altre informazioni, individuano l'identità principale dei nodi, qualunque essa sia al momento
della ricezione del messaggio.

Di seguito elenchiamo le informazioni che verranno convogliate al nodo *b* attraverso il protocollo del framework ZCD
(oltre naturalmente al messaggio vero e proprio che verrà indicato in seguito allo stub) e che quindi devono
venire specificate dal demone *ntkd* alla creazione dello stub.

*   `ISourceID source_id`. Identifica il mittente.  
    Verrà prodotto un MainIdentitySourceID.
*   `IUnicastID unicast_id`. Identifica il destinatario.  
    Verrà prodotto un MainIdentityUnicastID.
*   `ISrcNic src_nic`. Identifica il NIC usato dal nodo mittente.  
    Verrà prodotto un RoutableSrcNic che rappresenta un indirizzo routabile che identifica
    il nodo mittente all'interno del g-nodo comune.
*   Se lo stub resta in attesa della risposta. Booleano `wait_reply`.

##### ricezione lato server

Nel nodo *b*, a ricevere il messaggio è una tasklet avviata dal framework ZCD con la funzione `stream_net_listen`
o `stream_system_listen`. Questa prepara una istanza di CallerInfo specifica per i messaggi unicast.  
Nell'oggetto CallerInfo il framework ZCD include le informazioni convogliate nel protocollo del
framework ZCD e anche la conoscenza della modalità con cui era in ascolto la tasklet che ha recepito
il messaggio. In questo caso un ascolto di connessioni su uno specifico indirizzo IP routabile
oppure su uno specifico pathname.
Questa informazione è nel campo `Listener listener` dell'oggetto CallerInfo: può essere un
StreamNetListener (con i campi `my_ip` e `tcp_port`) o un StreamSystemListener (con il campo `listen_pathname`).

Poi passa il CallerInfo al delegato `Netsukuku.IDelegate`.
Il delegato riconosce dal CallerInfo che si tratta di un
messaggio unicast, quindi chiama il metodo `get_dispatcher` dello *SkeletonFactory*
passandogli le informazioni presenti nel CallerInfo.  
In questo metodo lo SkeletonFactory, che conosce le classi MainIdentitySourceID e MainIdentityUnicastID
sa che deve coinvolgere l'identità principale di *b*. Quindi
restituirà il `identity_skeleton` specifico per tale identità.

Il delegato restituisce lo skeleton (sarà di sicuro uno solo in questo caso, ma generalmente un set)
al framework ZCD, il quale potrà chiamarne i metodi che referenziano la giusta
istanza del modulo di identità interessato dal messaggio.

##### esecuzione lato server

Quando viene chiamato il metodo specificato dal messaggio sull'istanza dello skeleton del modulo
interessato, agli argomenti specifici del messaggio viene aggiunto il CallerInfo.  
Il modulo interessato sa che un dato metodo remoto viene chiamato attraverso la modalità
"nodo dentro un g-nodo", quindi che non può chiedere di identificare l'arco-identità tramite il
quale il messaggio è stato consegnato.

D'altra parte la modalità prevede una connessione reliable, sulla quale eventualmente sarà
possibile trasmettere una risposta al chiamante.

Se per altri motivi fosse necessario al ricevente conoscere una modalità per contattare in
altro momento il chiamante (ad esempio l'indirizzo IP univoco del chiamante all'interno del
g-nodo comune) questa informazione dovrà far parte degli argomenti del metodo remoto chiamato.

Per completezza ricordiamo che il metodo `from_caller_get_identityarc` dello SkeletonFactory
se gli viene passato un CallerInfo che contiene un ISourceID che non è un IdentityAwareSourceID
(come nel nostro caso che sarebbe un MainIdentitySourceID) restituisce *null*.

##### risposta dal server

Al termine dell'esecuzione del metodo remoto, se lo stub che aveva trasmesso il messaggio aveva
indicato nel protocollo ZCD di restare in attesa del risultato, il framework ZCD trasmette
il risultato nella stessa connessione in cui aveva ricevuto il messaggio.

#### Broadcast - diretti vicini

**TODO**

### Moduli di nodo

#### Unicast - diretto vicino

Facciamo un esempio di un modulo *di nodo* in cui il nodo *a* vuole chiamare un metodo remoto
sul nodo diretto vicino *b*, passando attraverso l'arco *x*.

##### chiamata lato client

Il nodo *a* chiama il metodo `get_stub_whole_node_unicast` dello *StubFactory*. Gli passa un
INeighborhoodArc che identifica l'arco *x*, il quale collega il nodo *a* con il nodo *b* attraverso
due specifiche interfacce di rete. Inoltre specifica se si vuole attendere una risposta al messaggio.  
Come detto prima (nel caso moduli di identità - unicast - diretto vicino) lo stub prodotto in questo modo
trasmetterà il messaggio in modalità reliable attraverso l'arco *x* che congiunge le due interfacce di rete
dei vicini (o pseudo-interfacce se usiamo il medium "system").

Ricordiamo che l'oggetto che identifica l'arco *x* contiene:

*   il NeighborhoodNodeID `neighbour_id` del vicino.
*   il MAC `neighbour_mac` e l'indirizzo IP linklocal `neighbour_nic_addr` del vicino.
*   il mio INeighborhoodNetworkInterface `my_nic`.

Quindi il metodo `get_stub_whole_node_unicast` può individuare i parametri che servono a
realizzare una connessione con `get_addr_stream_net` o `get_addr_stream_system`.

Di seguito elenchiamo le informazioni che verranno convogliate al nodo *b* attraverso il protocollo del framework ZCD
(oltre naturalmente al messaggio vero e proprio che verrà indicato in seguito allo stub) e che quindi devono
venire specificate dal demone *ntkd* alla creazione dello stub.

*   `ISourceID source_id`. Identifica il mittente.  
    Dal NeighborhoodNodeID del nodo locale (che il metodo conosce) verrà prodotto un WholeNodeSourceID.
*   `IUnicastID unicast_id`. Identifica il destinatario.  
    Dal NeighborhoodNodeID `x.neighbour_id` verrà prodotto un WholeNodeUnicastID.
*   `ISrcNic src_nic`. Identifica il NIC usato dal nodo mittente.  
    Dal INeighborhoodNetworkInterface `x.my_nic` verrà prodotto un NeighbourSrcNic.
*   Se lo stub resta in attesa della risposta. Booleano `wait_reply`.

##### ricezione lato server

Nel nodo *b* una tasklet *listener* (`stream_net_listen` o `stream_system_listen`) riceve il messaggio,
prepara un CallerInfo e lo passa al delegato `Netsukuku.IDelegate`.  
Il delegato riconosce dal CallerInfo che si tratta di un
messaggio unicast, quindi chiama il metodo `get_dispatcher` dello *SkeletonFactory*
passandogli le informazioni presenti nel CallerInfo.  
In questo metodo lo SkeletonFactory, che conosce le classi WholeNodeSourceID e WholeNodeUnicastID,
sa recuperarne il NeigborhoodNodeID di *a* e quello di *b*.  
Verifica che il suo NeigborhoodNodeID corrisponda a quello di *b*. Altrimenti termina la tasklet.  
Ci si aspetta che questo messaggio provenga da una connessione reliable stabilita tra due nodi
diretti vicini per i quali il modulo Neighborhood ha realizzato l'arco che li collega.
Ci sono due possibilità: l'arco è stato esposto dal modulo al suo utilizzatore (quindi il metodo
dello SkeletonFactory lo conosce) oppure non è stato ancora esposto: ad esempio quando il modulo
Neighborhood stesso intende usare l'arco per decidere di comune accordo col vicino se esporlo
(quindi il metodo dello SkeletonFactory non conosce l'arco). In questa fase non fa differenza:
il metodo `get_dispatcher` dello *SkeletonFactory* non cerca di individuare l'arco.  
Quindi il metodo `get_dispatcher` restituirà il `node_skeleton` del nodo locale.

Il delegato restituisce lo skeleton (sarà di sicuro uno solo in questo caso, ma generalmente un set)
al framework ZCD, il quale potrà chiamarne i metodi che referenziano l'istanza
del modulo di nodo interessato dal messaggio.

##### esecuzione lato server

Quando viene chiamato il metodo specificato dal messaggio sull'istanza dello skeleton del modulo
interessato, agli argomenti specifici del messaggio viene aggiunto il CallerInfo.  
Il modulo potrà usarlo per chiedere al demone *ntkd* (ad esempio attraverso un delegato o una
interfaccia) di identificare l'arco tramite il quale il messaggio è stato consegnato.

Questo sarà possibile solo per i messaggi che vengono trasmessi su un arco *x* che è stato
esposto dal modulo Neighborhood al suo utilizzatore.

Il codice nel demone *ntkd* incaricato di cercare l'arco partendo da un CallerInfo, sa che il
mittente del messaggio è un nodo diretto vicino. Chiama sullo SkeletonFactory il
metodo `from_caller_get_nodearc` e gli passa il CallerInfo.  
In questo metodo lo SkeletonFactory presume che il chiamante sia un diretto vicino, ma ammette
che possa aver fatto una trasmissione unicast o broadcast. In ogni caso, sa riconoscere il
tipo di CallerInfo e estrapolarne il ISourceID.  
Lo SkeletonFactory presume che questo sia un WholeNodeSourceID (se non lo fosse restituirebbe *null*)
e da esso recupera il NodeID dell'identità mittente.  
Lo SkeletonFactory recupera inoltre dal CallerInfo l'identificativo della nostra interfaccia
su cui il messaggio è stato ricevuto (informazione nel campo `listener` del CallerInfo) e
l'identificativo dell'interfaccia del mittente che esso ha usato
per trasmetterlo (informazione nel NeighbourSrcNic del CallerInfo).  
Con queste informazioni il metodo `from_caller_get_nodearc` individua e restituisce l'arco
su cui il messaggio è stato ricevuto da *b*.

##### risposta dal server

Al termine dell'esecuzione del metodo remoto, se lo stub che aveva trasmesso il messaggio aveva
indicato nel protocollo ZCD di restare in attesa del risultato, il framework ZCD trasmette
il risultato nella stessa connessione in cui aveva ricevuto il messaggio.

#### Broadcast - radar scan

Facciamo un esempio di un modulo *di nodo* in cui il nodo *a* vuole chiamare in broadcast un metodo remoto
sui nodi diretti vicini che raggiunge attraverso una sua interfaccia *nic*.  
L'unico caso nel demone *ntkd* è l'operazione di radar-scan. I metodi remoti di questa operazione
non fanno richiesta di un ACK.

##### chiamata lato client

Il nodo *a* chiama il metodo `get_stub_whole_node_broadcast_for_radar` dello *StubFactory*. Gli passa un
INeighborhoodNetworkInterface che identifica l'interfaccia di rete *nic*.  
Lo stub prodotto in questo modo trasmetterà il messaggio in modalità non-reliable attraverso l'interfaccia
di rete *nic*.  
La trasmissione avrà caratteristiche analoghe anche qualora si trattasse di una trasmissione sul medium "system"
con un socket unix-domain. In questo caso il fatto che il messaggio giunga ai nodi destinatari dipende dal
"corretto" funzionamento del proxy (e.g. `eth_domain` o `radio_domain` forniti da ZCD).

La classe che implementa INeighborhoodNetworkInterface contiene le informazioni necessarie affinché
il metodo `get_stub_whole_node_broadcast_for_radar` possa individuare i parametri che servono a
trasmettere un messaggio con `get_addr_datagram_net` o `get_addr_datagram_system`.

Di seguito elenchiamo le informazioni che verranno convogliate ai nodi
che rilevano il messaggio attraverso il protocollo del framework ZCD
(oltre naturalmente al messaggio vero e proprio che verrà indicato in seguito allo stub) e che quindi devono
venire specificate dal demone *ntkd* alla creazione dello stub.

*   `ISourceID source_id`. Identifica il mittente.  
    Dal NeighborhoodNodeID del nodo locale (che il metodo conosce) verrà prodotto un WholeNodeSourceID.
*   `IBroadcastID broadcast_id`. Identifica i destinatari.  
    Verrà prodotto un EveryWholeNodeBroadcastID, che non contiene altra informazione: tutti devono ricevere.
*   `ISrcNic src_nic`. Identifica il NIC usato dal nodo mittente.  
    Dal INeighborhoodNetworkInterface `nic` verrà prodotto un NeighbourSrcNic.
*   Come IAckCommunicator `notify_ack` viene passato *null*: non è richiesto alcun messaggio di ACK dai riceventi.

##### ricezione lato server

Nel nodo *b*, a ricevere il messaggio è una tasklet avviata dal framework ZCD con la funzione `datagram_net_listen`
o `datagram_system_listen`. Questa prepara una istanza di CallerInfo specifica per i messaggi broadcast.  
Nell'oggetto CallerInfo il framework ZCD include le informazioni convogliate nel protocollo del
framework ZCD e anche la conoscenza della modalità con cui era in ascolto la tasklet che ha recepito
il messaggio. In questo caso un ascolto di pacchetti su una specifica nostra interfaccia di rete
oppure su uno specifico pathname.
Questa informazione è nel campo `Listener listener` dell'oggetto CallerInfo: può essere un
DatagramNetListener (con i campi `my_dev, udp_port, src_nic`) o un DatagramSystemListener
(con i campi `listen_pathname, send_pathname, src_nic`).

Poi passa il CallerInfo ad un delegato fornito dal demone *ntkd* (un `Netsukuku.IDelegate`) il cui compito
è di reperire un set di istanze skeleton su cui il messaggio va processato.

Il delegato riconosce dal CallerInfo che si tratta di un
messaggio broadcast, quindi chiama il metodo `get_dispatcher_set` dello *SkeletonFactory*. Gli passa
le informazioni presenti nel CallerInfo, cioè quelle convogliate nel protocollo del framework ZCD.  
In questo metodo lo SkeletonFactory, che conosce le classi WholeNodeSourceID e EveryWholeNodeBroadcastID
sa recuperare il NeigborhoodNodeID di *a* e sa che deve coinvolgere il `node_skeleton` del nodo locale.

Il delegato restituisce lo skeleton (sarà di sicuro uno solo in questo caso, ma generalmente un set)
al framework ZCD, il quale potrà chiamarne i metodi che referenziano l'istanza
del modulo di nodo interessato dal messaggio.

##### esecuzione lato server

Quando viene chiamato il metodo specificato dal messaggio sull'istanza dello skeleton del modulo
interessato, agli argomenti specifici del messaggio viene aggiunto il CallerInfo.  
Il modulo potrà usarlo per chiedere al demone *ntkd* di identificare l'interfaccia di rete tramite cui il
messaggio è stato rilevato.

Ad esempio il modulo di nodo Neighborhood prevede a questo scopo il metodo `is_from_broadcast`
dell'interfaccia INeighborhoodQueryCallerInfo.  
In questo metodo il demone *ntkd* sa che il mittente del messaggio è un nodo
diretto vicino. Chiama sullo SkeletonFactory il metodo `from_caller_get_mydev` e gli passa
il CallerInfo.  
In questo metodo lo SkeletonFactory sa riconoscere il tipo di CallerInfo e estrapolarne
il nome della propria interfaccia di rete (o pseudo se il medium è "system").  
Da questo nome il metodo `is_from_broadcast` recupera l'istanza di INeighborhoodNetworkInterface
che la rappresenta nel modulo di nodo Neighborhood.