
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


**TODO** spostare altrove.
L'oggetto CallerInfo contiene queste informazioni:

*   Un ISourceID.
*   Un IUnicastID o un IBroadcastID.
*   Se si tratta di un messaggio ricevuto come pacchetto UDP broadcast:
    *   Questo è necessariamente trasmesso da un nostro diretto vicino.
    *   Un identificativo associabile all'interfaccia di rete sulla quale il diretto vicino ha trasmesso il messaggio.
        *   Può essere il MAC se il medium è la rete.
        *   Se il medium è uno unix-domain, cosa usiamo?
    *   Un identificativo associabile alla propria interfaccia di rete che ha recepito il messaggio.
        *   Può essere il device-name se il medium è la rete.
        *   Se il medium è uno unix-domain, cosa usiamo?
*   Se si tratta di uno stream ricevuto su una connessione TCP tramite indirizzo IP linklocal:
    *   Questa connessione è con un nostro diretto vicino.
    *   Un identificativo associabile all'interfaccia di rete del diretto vicino di cui si compone l'arco
        (o arco-identità) interessato dal messaggio.
        *   Può essere l'indirizzo IP linklocal se il medium è la rete.
        *   Se il medium è uno unix-domain, cosa usiamo?
    *   Un identificativo associabile alla propria interfaccia di rete di cui si compone l'arco
        (o arco-identità) interessato dal messaggio.
        *   Può essere l'indirizzo IP linklocal se il medium è la rete.
        *   Se il medium è uno unix-domain, cosa usiamo?
*   Se si tratta di uno stream ricevuto su una connessione TCP tramite indirizzo IP routabile:
    *   Questa connessione è con un altro nodo di un nostro g-nodo.
    *   Non servono altre informazioni. Si tratta sicuramente di un messaggio destinato ad un
        *modulo di identità* e le identità mittente e destinatario sono le identità principali dei
        due nodi che stanno comunicando. La risposta del server è trasmessa nella stessa connessione TCP.

**FINE-TODO**

