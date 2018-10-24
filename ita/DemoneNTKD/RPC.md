# ntkd - RPC

1.  [StubFactory](#StubFactory)
1.  [SkeletonFactory](#SkeletonFactory)
1.  [Tipi di trasmissione](#Trasmissioni)
1.  [Tipi di medium](#tipi-di-medium)
1.  [Tasklet in ascolto](#Tasklet_listen)
1.  [Chiamate a metodi remoti](#Get_stub)
1.  [Identità multiple in un sistema](#Identita_multiple_in_un_sistema)

Il progetto *ntkd* usa il framework ZCD per realizzare le comunicazioni tra nodi.

## <a name="StubFactory"></a>StubFactory

Una classe **StubFactory** è usata per produrre gli stub che i vari moduli dell'applicazione usano per
comunicare con altri nodi (diretti vicini o specifici nodi all'interno di un comune g-nodo).

## <a name="SkeletonFactory"></a>SkeletonFactory

Una classe **SkeletonFactory** è usata quando si rileva una richiesta tramite una interfaccia di rete.
Interrogando questa classe si decide se bisogna passare la richiesta a uno
(o piu d'uno) skeleton nel nodo corrente, il quale potrà richiamare metodi remoti definiti nei vari moduli.

## <a name="Trasmissioni"></a>Tipi di trasmissione

Abbiamo analizzato nel documento [ZCD](../Librerie/ZCD.md#Trasmissioni) che il framework ZCD prevede due tipi
di trasmissione:

*   "stream", per i messaggi unicast.
*   "datagram", per i messaggi broadcast.

Nel demone *ntkd* la modalità unicast-stream è usata per invocare un metodo remoto (cioè inviare
un messaggio) e ottenere una risposta precisa in queste circostanze:

*   TODO

Nel demone *ntkd* la modalità unicast-stream è usata per invocare un metodo remoto (cioè inviare
un messaggio) e solo accertarsi della ricezione (`wait_reply=false`) in queste circostanze:

*   TODO

Nel demone *ntkd* la modalità broadcast-datagram è usata per inviare
un messaggio senza pretendere un puntuale risultato in queste circostanze:

*   TODO

Nel demone *ntkd* la modalità unicast-stream è usata come "rafforzativo" (cioè per ribadire un messaggio
broadcast quando ci si accorge che questo non è stato recepito, con `wait_reply=false`) in queste circostanze:

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

Per emulare il dominio "tau", si lancia `domain -i 1234_eth0 -i 6543_eth0`.  
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
Abbiamo visto pocanzi che il processo `domain` ascolta su `send_1234_eth0`.

Per "beta", si lancia `qspntester -p 6543 -I eth0`. Tale processo computa un pathname `6543_eth0` per
rappresentare il suo pseudonic `eth0`.  
Quando questo processo vuole stare in ascolto per messaggi broadcast su questo pseudonic, in realtà **crea** il
pathname `recv_6543_eth0` e si mette in ascolto con un socket unix-domain su quel pathname.  
Per il momento nessuno scrive su `recv_6543_eth0`.  
Quando questo processo vuole trasmettere un pacchetto tramite questo pseudonic, in realtà lo scrive con
un socket unix-domain sul pathname `send_6543_eth0`, **ma solo** se questo pathname esiste già.  
Abbiamo visto pocanzi che il processo `domain` ascolta su `send_6543_eth0`.

Adesso supponiamo che il nodo "beta" vuole scrivere un pacchetto sul suo pseudonic "eth0". Lo
scrive con un socket unix-domain sul pathname `send_6543_eth0`. Quindi il processo "domain"
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

*   `radio-domain -i 1234_wlan0 -o 6543_wlan0`.  
    Crea `send_1234_wlan0` e si mette in ascolto su esso.  
    Quando riceve:
    *   prova a scrivere su `recv_1234_wlan0` ma soltanto se già esiste.
    *   prova a scrivere su `recv_6543_wlan0` ma soltanto se già esiste.

*   `radio-domain -i 6543_wlan0 -o 1234_wlan0 -o 6789_wlan0`.  
    Crea `send_6543_wlan0` e si mette in ascolto su esso.  
    Quando riceve:
    *   prova a scrivere su `recv_6543_wlan0` ma soltanto se già esiste.
    *   prova a scrivere su `recv_1234_wlan0` ma soltanto se già esiste.
    *   prova a scrivere su `recv_6789_wlan0` ma soltanto se già esiste.

*   `radio-domain -i 6789_wlan0 -o 6543_wlan0`.  
    Crea `send_6789_wlan0` e si mette in ascolto su esso.  
    Quando riceve:
    *   prova a scrivere su `recv_6789_wlan0` ma soltanto se già esiste.
    *   prova a scrivere su `recv_6543_wlan0` ma soltanto se già esiste.

## <a name="Tasklet_listen"></a>Tasklet in ascolto

**TODO**

## <a name="Get_stub"></a>Chiamate a metodi remoti

**TODO**

## <a name="Identita_multiple_in_un_sistema"></a>Identità multiple in un sistema

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
oltre che indicare un `peer_ip` o un `my_dev`.  
Lato server alla recezione di un messaggio costruisce un CallerInfo, indicando in esso la tasklet in
ascolto (Listener) e gli oggetti indicati nel messaggio (ISourceID, ISrcNic, IUnicastID, IBroadcastID).
Sulla base del CallerInfo il suo utilizzatore potrà individuare zero/uno/molti skeleton
a cui far eseguire una procedura.

I delegati passati al framework ZCD sono in grado di riconoscere gli oggetti ISourceID, IUnicastID/IBroadcastID
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
In ogni nodo della rete Netsukuku esiste sempre una e una sola *identità principale*. Essa è
quella che gestisce le interfacce di rete *reali* del nodo nel *network namespace default*.  
L'identità principale del nodo non è sempre la stessa: il nodo può in un certo momento creare una nuova
identità e farla diventare la sua principale.

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
costruendo gli appositi oggetti ISourceID, IUnicastID/IBroadcastID e chiamando i metodi del framework ZCD.

Inoltre lato server, attraverso i delegati passati al framework ZCD, partendo dagli oggetti ISourceID,
IUnicastID/IBroadcastID ricevuti deve saper individuare zero/uno/molti skeleton da attivare. Se il messaggio
era per identità, allora ognuno di questi skeleton è una precisa istanza del modulo interessato.

Ricordiamo che i delegati passati al framework ZCD ricevono una istanza della classe CallerInfo,
fornita dalla libreria di livello intermedio del framework ZCD prodotta con "rpcdesign". Essa contiene fra l'altro
un ISourceID e un IUnicastID/IBroadcastID.

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
Lo stub prodotto in questo modo trasmetterà il messaggio su una sola interfaccia di rete, univocamente individuata
dall'arco *x*.

L'oggetto che identifica l'arco *x* contiene le due interfacce di rete dell'arco. Cioè il device-name della
propria interfaccia di rete del nodo *a* e il MAC address dell'interfaccia di rete di *b*. Inoltre contiene
anche l'indirizzo IP linklocal che il nodo *b* ha associato a quella interfaccia di rete. Sicché una connessione
TCP realizzata dal nodo *a* verso quell'indirizzo IP si appoggia esattamente sull'arco *x*. Infatti al momento
della realizzazione dell'arco il modulo Neighborhood nei due nodi avrà impostato una apposita rotta con scope *link*
nelle tabelle del kernel del *network namespace default*.

Dal NodeID di *a<sub>0</sub>* verrà prodotto un IdentityAwareSourceID.
Dal NodeID di *b<sub>0</sub>* verrà prodotto un IdentityAwareUnicastID.

Di seguito elenchiamo le informazioni che verranno convogliate al nodo *b* attraverso il protocollo del framework ZCD
(oltre naturalmente al messaggio vero e proprio che verrà indicato in seguito allo stub) e che quindi devono
venire specificate dal demone ntkd alla creazione dello stub.

*   `ISourceID source_id`. Identifica il mittente.  
    In questo caso si tratta di un IdentityAwareSourceID, quindi identifica una *identità* nel nodo
    mittente. Ma questo ZCD non è tenuto a saperlo.
*   `IUnicastID unicast_id`. Identifica il destinatario.  
    In questo caso si tratta di un IdentityAwareUnicastID, quindi identifica una *identità* nel nodo
    destinatario. Ma questo ZCD non è tenuto a saperlo.
*   `ISrcNic src_nic`. Identifica il NIC usato dal nodo mittente.  
    In questo caso si tratta di un StreamSrcNic, quindi rappresenta un indirizzo linklocal che il nodo
    mittente ha assegnato a una delle sue interfacce di rete. Ma questo ZCD non è tenuto a saperlo.
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

Poi passa il CallerInfo ad un delegato fornito dal demone ntkd (un `Netsukuku.IDelegate`) il cui compito
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
Il modulo potrà usarlo per chiedere al demone ntkd (ad esempio attraverso un delegato o una
interfaccia) di identificare l'arco-identità tramite il quale il messaggio è stato consegnato.

Prendiamo ad esempio il metodo `i_qspn_comes_from(CallerInfo rpc_caller)` dell'interfaccia `IQspnArc`
come viene usato dallo skeleton del modulo Qspn nel metodo remoto `get_full_etp`.  
Il metodo remoto `get_full_etp` in realtà può essere invocato dai diretti vicini sia con un messaggio
unicast, sia con un messaggio broadcast. Le operazioni il metodo `i_qspn_comes_from` sono comunque
le stesse in entrambi i casi.

Durante l'esecuzione del metodo remoto `get_full_etp` invocato dal framework ZCD, il QspnManager conosce
gli archi-identità di *b<sub>0</sub>* che gli sono stati passati come oggetti che implementano l'interfaccia IQspnArc.
Prende uno di essi e vuole chiedere al demone ntkd se il CallerInfo indica che il messaggio
è stato consegnato proprio attraverso di esso. Quindi ne chiama il metodo `i_qspn_comes_from(rpc_caller)`.

In questo metodo il demone ntkd sa che il mittente del messaggio è una identità di un nodo
diretto vicino. Chiama sullo SkeletonFactory il metodo `from_caller_get_identityarc` e gli passa
il CallerInfo e l'istanza di IdentityData che rappresenta l'identità associata alla
istanza di QspnManager che ha fatto richiesta.  
In questo metodo lo SkeletonFactory presume che il chiamante sia un diretto vicino, ma ammette
che possa aver fatto una trasmissione unicast o broadcast. In ogni caso, sa riconoscere il
tipo di CallerInfo e estrapolarne il ISourceID.  
Lo SkeletonFactory presume che questo sia un IdentityAwareSourceID (se non lo fosse restituirebbe *null*)
e da esso recupera il NodeID dell'identità mittente.  
Lo SkeletonFactory recupera inoltre dal CallerInfo l'identificativo della nostra interfaccia
su cui il messaggio è stato ricevuto e l'identificativo dell'interfaccia del mittente che esso ha usato
per trasmetterlo.  
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
Lo stub prodotto in questo modo trasmetterà il messaggio su una sola interfaccia di rete, individuata
sulle tabelle di routing del kernel del nodo *a* sulla base dell'indirizzo IP routabile prodotto
dall'indirizzo di *b* interno ad un g-nodo di *a*.

Vengono prodotti in questo metodo un MainIdentitySourceID e un MainIdentityUnicastID. Queste due classi
non contengono altre informazioni, individuano l'identità principale dei nodi, qualunque essa sia al momento
della ricezione del messaggio.

Di seguito elenchiamo le informazioni che verranno convogliate al nodo *b* attraverso il protocollo del framework ZCD
(oltre naturalmente al messaggio vero e proprio che verrà indicato in seguito allo stub) e che quindi devono
venire specificate dal demone ntkd alla creazione dello stub.

*   `ISourceID source_id`. Identifica il mittente.  
    In questo caso si tratta di un MainIdentitySourceID. Ma questo ZCD non è tenuto a saperlo.
*   `IUnicastID unicast_id`. Identifica il destinatario.  
    In questo caso si tratta di un MainIdentityUnicastID. Ma questo ZCD non è tenuto a saperlo.
*   `ISrcNic src_nic`. Identifica il NIC usato dal nodo mittente.  
    In questo caso si tratta di un StreamSrcNic, quindi rappresenta un indirizzo routabile che identifica
    il nodo mittente all'interno del g-nodo comune. Ma questo ZCD non è tenuto a saperlo.
*   Se lo stub resta in attesa della risposta. Booleano `wait_reply`.

##### ricezione lato server

Quando nel nodo *b* il framework ZCD riceve il messaggio esso riconosce dalla
modalità di trasmissione (di tipo STREAM, che prevede l'argomento IUnicastID) che si tratta di un
messaggio unicast, quindi prepara una istanza di CallerInfo specifica per i messaggi unicast.  
Nell'oggetto CallerInfo il framework ZCD include le informazioni convogliate nel protocollo del
framework ZCD e anche la conoscenza della modalità `ListenMode listen_mode` con cui era in ascolto
la tasklet che ha recepito il messaggio. In questo caso un ascolto di connessioni su uno specifico
indirizzo IP routabile oppure su uno specifico unix-domain socket.

Poi passa il CallerInfo al delegato `Netsukuku.IDelegate`.
Il delegato riconosce dal CallerInfo che si tratta di un
messaggio unicast, quindi chiama il metodo `get_dispatcher` dello *SkeletonFactory*
passandogli il CallerInfo.  
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



***


**TODO** spostare altrove.

Quando si chiama il metodo che produce uno stub per l'invio di messaggi in broadcast, può essere passato un
oggetto che implementa l'interfaccia `IAckCommunicator` fornita dal `ntkdrpc`. Ogni volta che verrà usato lo
stub per trasmettere un messaggio in broadcast, dopo un certo timeout dal momento della trasmissione,
questo oggetto riceverà (nel metodo `process_macs_list`) un elenco dei MAC (lista di stringhe, o in futuro
istanze di ISrcNic) dai quali abbiamo ricevuto un pacchetto di "ACKnowledgement" di ricezione del nostro messaggio.  
La logica è che questo oggetto ha le informazioni necessarie a gestire il fatto che un certo MAC (in generale
una certa interfaccia di rete di un vicino che conosciamo) non ha trasmesso l'ACK nel tempo previsto.

Attualmente, l'unico caso di questo tipo nel demone *ntkd* è `get_stub_identity_aware_broadcast`. Infatti l'altro
metodo `get_stub_whole_node_broadcast_for_radar` gli passa *null*.  
Quindi più specificamente la logica è che questo oggetto gestisce i singoli archi-identità collegati agli
archi dai quali non abbiamo ricevuto conferma di ricezione.  
L'oggetto passato è un `AcknowledgementsCommunicator`, una classe privata dello StubFactory. Al suo interno
ha un `NodeMissingArcHandlerForIdentityAware` cioè un gestore degli archi-nodo che sono stati "mancati"
dalla trasmissione broadcast che sa che deve operare sugli archi-identità. Questo a sua volta al suo interno ha
una istanza dell'interfaccia `IIdentityAwareMissingArcHandler` cioè un gestore degli archi-identità che sono
stati "mancati" dalla trasmissione broadcast.

L'oggetto `AcknowledgementsCommunicator` può accedere (sia al momento della produzione dello stub sia
al momento dell'esecuzione del metodo `process_macs_list`) alla lista di archi-nodo che il demone *ntkd*
conosce. Ogni arco-nodo ha un identificativo dell'interfaccia di rete del vicino che è confrontabile
con l'identificativo (`string mac` o `ISrcNic src_nic`) che sono state passate nel metodo `process_macs_list`
dal framework ZCD.  
Tutto ciò permette di chiamare una serie di volte (ognuna in una tasklet indipendente) il metodo
`missing(IdentityData identity_data, IdentityArc identity_arc)` della `IIdentityAwareMissingArcHandler`
quando non si riceve nel tempo previsto un ACK da una certa interfaccia.

Le istanze di `IIdentityAwareMissingArcHandler` usate nel codice sono:

*   `MissingArcHandlerForPeers` nel file `peers_helpers.vala`. Usata nel `PeersNeighborsFactory.i_peers_get_broadcast`
    che il modulo usa per chiamare i metodi remoti `set_participant` e `give_participant_maps`.
*   `MissingArcHandlerForQspn` nel file `qspn_helpers.vala`. Usata nel `QspnStubFactory.i_qspn_get_broadcast`
    che il modulo usa per chiamare i metodi remoti `send_etp`, `got_prepare_destroy` e `got_destroy`.

**FINE-TODO**


***


**TODO** spostare altrove.

L'oggetto CallerInfo viene prodotto sul server, in particolare dalla libreria *ntkdrpc*, a seguito di una
richiesta da remoto contenuta in una connessione o un messaggio. La richiesta viene letta da una delle
tasklet che sono state avviate per l'ascolto.

A seconda della tasklet che riceve viene prodotta una istanza di:

*   `StreamCallerInfo` dalla tasklet `stream_net_listen` o dalla tasklet `stream_system_listen`. Era la `TcpclientCallerInfo`.  
    Questa contiene:
    *   `ISourceID sourceid`
    *   `IUnicastID unicastid`
    *   `ISrcNic src_nic`
    *   `bool wait_reply`
    *   `string listening_to_my_ip`
*   `DatagramCallerInfo` dalla tasklet `datagram_net_listen` o dalla tasklet `datagram_system_listen`. Era la `BroadcastCallerInfo`.  
    Questa contiene:
    *   `ISourceID sourceid`
    *   `IBroadcastID broadcastid`
    *   `ISrcNic src_nic`
    *   `bool send_ack`
    *   `string listening_to_my_dev`

**FINE-TODO**

