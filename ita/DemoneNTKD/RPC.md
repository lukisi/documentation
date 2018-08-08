# ntkd - RPC

Il progetto *ntkd* usa il framework ZCD per realizzare le comunicazioni tra nodi.

## StubFactory

Una classe **StubFactory** è usata per produrre gli stub che i vari moduli dell'applicazione usano per
comunicare con altri nodi (diretti vicini o specifici nodi all'interno di un comune g-nodo).

## SkeletonFactory

Una classe **SkeletonFactory** è usata quando si rileva una richiesta tramite una interfaccia di rete.
Interrogando questa classe si decide se bisogna passare la richiesta a uno
(o piu d'uno) skeleton nel nodo corrente, il quale potrà richiamare metodi remoti definiti nei vari moduli.

## <a name="Identita_multiple_in_un_sistema"></a>Identità multiple in un sistema

### ZCD

Il framework ZCD supporta la presenza di molteplici *identità* all'interno di un nodo. Con questo
intendiamo evidenziare che è possibile realizzare uno stub che possa essere usato da una precisa identità
del nodo *a*, indichiamola con *a<sub>0</sub>*, per chiamare un metodo remoto su una precisa identità del
nodo *b*, indichiamola con *b<sub>0</sub>*.

Ricordiamo che il framework ZCD identifica il mittente e il destinatario (o i destinatari) di ogni
messaggio attraverso delle classi che sono rappresentate con le interfacce ISourceID, IUnicastID
e IBroadcastID.  
Lato client permette di costruire degli stub che trasmettono messaggi (cioè chiamano procedure remote)
da un ISourceID a un IUnicastID oppure a un IBroadcastID. Lato server alla recezione di un messaggio
sulla base degli oggetti ISourceID, IUnicastID/IBroadcastID ricevuti individua zero/uno/molti skeleton
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
    **TODO**: In `build_json_request` della libreria di basso livello ZCD
    dobbiamo aggiungere l'argomento `string src_nic` in cui è serializzato l'oggetto ISrcNic.
*   Se lo stub resta in attesa della risposta. Booleano `wait_reply`.

##### ricezione lato server

Quando nel nodo *b* il framework ZCD riceve il messaggio esso riconosce dalla
modalità di trasmissione (di tipo STREAM, che prevede l'argomento IUnicastID) che si tratta di un
messaggio unicast, quindi prepara una istanza di CallerInfo specifica per i messaggi unicast.  
Nell'oggetto CallerInfo il framework ZCD include le informazioni convogliate nel protocollo del
framework ZCD e anche la conoscenza della modalità con cui era in ascolto la tasklet che ha recepito
il messaggio. In questo caso un ascolto di connessioni su uno specifico indirizzo IP linklocal
(associato ad una specifica nostra interfaccia di rete) oppure su uno specifico unix-domain socket.
Diciamo che questa informazione è nel campo `ListenMode listen_mode` (non da serializzare)
dell'oggetto CallerInfo.

Poi passa il CallerInfo ad un delegato fornito dal demone ntkd (un `Netsukuku.IRpcDelegate`) il cui compito
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

Poi passa il CallerInfo al delegato `Netsukuku.IRpcDelegate`.
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

*   `StreamNetCallerInfo` dalla tasklet `stream_ip_listen`. Era la `TcpclientCallerInfo`.  
    Questa contiene:
    *   `ISourceID sourceid`
    *   `IUnicastID unicastid`
    *   `ISrcNic src_nic`
    *   `bool wait_reply`
    *   `string listening_to_my_ip`
*   `DatagramNetCallerInfo` dalla tasklet `datagram_nic_listen`. Era la `BroadcastCallerInfo`.  
    Questa contiene:
    *   `ISourceID sourceid`
    *   `IBroadcastID broadcastid`
    *   `ISrcNic src_nic`
    *   `string listening_to_my_dev`
*   `StreamSystemCallerInfo` dalla `stream_pathname_listen`.  
    Questa contiene:
    *   `ISourceID sourceid`
    *   `IUnicastID unicastid`
    *   `ISrcNic src_nic`
    *   `bool wait_reply`
    *   `string listening_to_my_pseudoip`
*   `DatagramSystemCallerInfo` dalla `datagram_pathname_listen`.  
    Questa contiene:
    *   `ISourceID sourceid`
    *   `IBroadcastID broadcastid`
    *   `ISrcNic src_nic`
    *   `string listening_to_my_pseudodev`

**FINE-TODO**

