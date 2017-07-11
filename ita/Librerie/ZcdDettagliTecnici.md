# ZCD - Dettagli Tecnici

1.  [ZCD](#ZCD)
    1.  [Inizializzazione](#ZCD_Inizializzazione)
    1.  [TCP - server](#ZCD_tcp_listen)
    1.  [TCP - client](#ZCD_tcp_client)
    1.  [UDP - server](#ZCD_udp_listen)
    1.  [UDP - client](#ZCD_UDP)
1.  [MOD-RPC](#MODRPC)
    1.  [Oggetti serializzabili](#MODRPC_serializzabili)
    1.  [Lato server](#MODRPC_server)
    1.  [Lato client](#MODRPC_client)
    1.  [Serializzazione e deserializzazione di argomenti, valori di ritorno e eccezioni](#MODRPC_argomenti)
1.  [APP](#APP)
    1.  [Lato server](#APP_Server)
    1.  [Lato client](#APP_Client)

## <a name="ZCD"></a>ZCD

La libreria di basso livello.

La libreria ZCD fa uso di *tasklet* appoggiandosi alla libreria [tasklet-system](TaskletSystem.md). La libreria
*tasklet-system* sarà una dipendenza comune sia alla libreria di basso livello (ZCD) sia a quella intermedia
(MOD-RPC) sia all'applicazione (APP) in quanto sarà questa ultima che fornirà l'implementazione delle interfacce
attraverso uno specifico sistema di tasklet.

La libreria ZCD viene "inizializzata" con alcune chiamate, fra cui il metodo `init_tasklet_system`. In esso il
suo utilizzatore fornisce le implementazioni delle interfacce usate per le comunicazioni allo schedulatore.

Se il diretto utilizzatore di ZCD è la libreria intermedia MOD-RPC, allora anche questa avrà un metodo
`init_tasklet_system` che farà da proxy verso quello di ZCD. Così l'applicazione APP, pur non avendo una
dipendenza diretta verso ZCD, sarà in grado di fornire l'implementazione.

La libreria ZCD usa JsonGlib. All'utilizzatore di ZCD non viene imposto di avere una dipendenza diretta da JsonGlib.

### <a name="ZCD_Inizializzazione"></a>Inizializzazione

ZCD viene "inizializzata" con alcune chiamate.

*   Il metodo `init_tasklet_system` fornisce la implementazione delle interfacce usate per le comunicazioni
    allo schedulatore delle tasklet.
*   Il metodo `tcp_listen` avvia una tasklet che con un socket TCP si mette in ascolto su un dato indirizzo
    locale (oppure su tutti) e una data porta.
*   Il metodo `udp_listen` avvia una tasklet che con un socket UDP si mette in ascolto su una data interfaccia
    di rete e una data porta per ricevere pacchetti inviati in broadcast sul broadcast domain collegato alla interfaccia passata.

In seguito si può richiedere a ZCD di effettuare una chiamata a metodo remoto invocando questi metodi:

*   Il metodo `tcp_client` crea e ritorna una istanza di TcpClient.  
    Con i metodi dell'oggetto TcpClient si effettua una chiamata in modalità **TcpClient**.
*   Il metodo `send_unicast_request` trasmette in broadcast una chiamata in modalità **Unicast**.
*   Il metodo `send_broadcast_request` trasmette in broadcast una chiamata in modalità **Broadcast**.

### <a name="ZCD_tcp_listen"></a>TCP - server

Il metodo `tcp_listen` avvia una tasklet che ascolta e gestisce le connessioni TCP provenienti dall'esterno.

#### cosa riceve

I dati che vengono passati al metodo `tcp_listen` sono:

*   La porta TCP e l'indirizzo.
*   Un delegato che ZCD potrà usare alla ricezione di una connessione per ottenere, se opportuno, un
    "dispatcher" a cui passare il messaggio per l'esecuzione.
*   Un delegato che la tasklet in ascolto potrà usare in caso di errore prima di terminare.

Il delegato per `tcp_listen` da usare alla ricezione di una connessione è nella forma di una implementazione
dell'interfaccia IZcdTcpDelegate. Il suo metodo:

*   `IZcdTcpRequestHandler get_new_handler()`

crea una istanza dell'interfaccia IZcdTcpRequestHandler apposita per gestire una connessione. I metodi di
quest'ultima sono:

*   `void set_unicast_id(string unicast_id)`
*   `void set_method_name(string m_name)`
*   `void add_argument(string argument)`
*   `void set_caller_info(TcpCallerInfo caller_info)`
*   `IZcdDispatcher? get_dispatcher()`

La classe TcpCallerInfo contiene i membri:

*   `string my_address`
*   `string peer_address`
*   `string source_id` (serializzazione di un *identificativo di identità*)

L'interfaccia IZcdDispatcher è implementata da un oggetto che verrà istanziato alla chiamata
di `get_dispatcher`. I suoi metodi sono:

*   `string execute()`

Il delegato per `tcp_listen` da usare in caso di errore nella `listen` o nella `accept` è nella
forma di una implementazione dell'interfaccia IZcdTcpAcceptErrorHandler. I suoi metodi sono:

*   `void error_handler(Error e)`

#### cosa fa la tasklet che gestisce il socket in ascolto

La tasklet avviata dal metodo `tcp_listen` apre un socket TCP e si mette in ascolto sulla porta
e l'indirizzo specificati. In caso di errore nella `listen` lo passa al metodo `error_handler` e
poi termina.

Quando riceve una connessione chiama `get_new_handler` e passa l'istanza di IZcdTcpRequestHandler
ad una nuova tasklet per gestire la connessione. In caso di errore nella `accept` lo passa al
metodo `error_handler` e poi termina.

#### quali messaggi passano attraverso una connessione TCP

Le richieste che arrivano da queste connessioni sono chiamate a metodi remoti, ognuna inclusa in un
albero JSON.

Poiché sarebbe complicato (la libreria JsonGlib non lo supporta) leggere uno stream di bytes fino ad
ottenere esattamente un valido albero JSON, si conviene che nelle comunicazioni in TCP tra end-points
della libreria ZCD i primi 4 byte trasmessi indicano la lunghezza totale in bytes del successivo messaggio.
Dopo il messaggio, se la connessione non viene chiusa (dal client) è possibile che venga trasmesso un
successivo blocco di 4 byte e un successivo messaggio.

Se il messaggio ricevuto prevede una risposta, questa sarà trasmessa nella stessa connessione, sempre
con la convenzione dei 4 byte di lunghezza. Si conviene quindi che, quando un client trasmette un messaggio
che prevede una risposta, fino a che la risposta non è stata completamente ricevuta il client non usi
la stessa connessione per trasmettere altri messaggi. Può, invece, aprirne immediatamente una nuova.

L'albero JSON di un messaggio di richiesta ha come radice un nodo *OBJECT* con 5 membri: "method-name",
"arguments", "source-id", "unicast-id", "wait-reply". Il membro "method-name" è un nodo *VALUE* di tipo
stringa, il membro "arguments" è un nodo *ARRAY* in cui ogni elemento è un valido nodo radice, cioè *OBJECT*
o *ARRAY*. I membri "source-id" e "unicast-id" sono nodi *OBJECT*. Il membro "wait-reply" è un nodo *VALUE*
di tipo booleano.

L'albero JSON di un messaggio di risposta ha come radice un nodo *OBJECT* con 1 membro: "response". Il membro
"response" è un valido nodo radice, cioè *OBJECT* o *ARRAY*.

#### cosa fa la tasklet che gestisce una connessione

La tasklet che gestisce una connessione riceve questi parametri iniziali:

*   IConnectedStreamSocket c - Il socket connesso.
*   IZcdTcpRequestHandler req - Il delegato per le richieste.

La tasklet legge i 4 bytes che indicano la lunghezza del messaggio e poi legge il numero di bytes indicato.
Se la connessione si chiude prima la tasklet termina.

Alla fine verifica di avere letto una stringa contenente un valido albero JSON. Tale JSON la libreria ZCD è
in grado di interpretarlo, quindi la tasklet ne estrae le seguenti informazioni:

*   Membro **method-name**. Una stringa con il nome del metodo remoto.
*   Membro **arguments**. Un array di nodi JSON che saranno passati, in forma di altrettante singole
    stringhe, al dispatcher come argomenti del metodo. Tali stringhe devono essere ognuna un valido JSON,
    quindi i nodi JSON dell'array devono essere di tipo *OBJECT* o *ARRAY*.
*   Membro **source-id**. Un nodo JSON di tipo *OBJECT* che sarà passato, in forma di stringa, al dispatcher
    come parte del CallerInfo.
*   Membro **unicast-id**. Un nodo JSON di tipo *OBJECT* che sarà passato, in forma di stringa, al delegato
    IZcdTcpRequestHandler.
*   Membro **wait-reply**. Un booleano che dice se è prevista una risposta.

Se qualcuno di questi requisiti non è soddisfatto la tasklet chiude la connessione e termina.

Di seguito la tasklet:
*   Prepara una istanza di TcpCallerInfo con le informazioni che è in grado di ottenere tramite l'istanza
    del socket connesso, cioè l'indirizzo IP del client remoto e il proprio indirizzo IP dove è stato contattato
    e la stringa **source-id**.
*   Chiama il metodo `set_unicast_id` sulla sua istanza di IZcdTcpRequestHandler. Si assume che l'oggetto
    memorizzi la stringa **unicast-id**.
*   Chiama il metodo `set_method_name` sulla sua istanza di IZcdTcpRequestHandler. Si assume che l'oggetto
    memorizzi il nome del metodo.
*   Chiama (quante volte necessario) il metodo `add_argument` sulla sua istanza di IZcdTcpRequestHandler. Si
    assume che l'oggetto memorizzi gli argomenti da passare al metodo.
*   Chiama il metodo `set_caller_info` sulla sua istanza di IZcdTcpRequestHandler. Si assume che l'oggetto
    memorizzi l'istanza di TcpCallerInfo.
*   Chiama il metodo `get_dispatcher` sulla sua istanza di IZcdTcpRequestHandler e ottiene un IZcdDispatcher *disp*.  
    L'oggetto IZcdTcpRequestHandler a questo punto è in grado di stabilire se il messaggio è da processare. In
    questo caso ritorna una istanza apposita di IZcdDispatcher. Altrimenti ritorna null.  
    In ogni caso l'oggetto IZcdTcpRequestHandler resetta le sue impostazioni. Infatti l'eventuale oggetto
    IZcdDispatcher restituito ha memorizzato le informazioni per l'esecuzione del metodo.
*   **TODO** Se non si è ottenuto un dispatcher, nonostante la comunicazione sia avvenuta in TCP, può essere dovuto
    al fatto che si cercava una *identità* nel nodo che non è stata trovata. Questa situazione non dovrebbe
    necessariamente portare alla chiusura della connessione TCP, che di per sé rimane valida. Tuttavia andrebbe
    segnalato uno specifico errore al chiamante.
*   Se *disp* = *null* il messaggio va ignorato. La tasklet chiude la connessione e termina.
*   Se il messaggio dichiarava di voler ricevere una risposta:
    *   Esegue `resp = disp.execute()`.  
        L'oggetto IZcdDispatcher deserializza gli argomenti. Esegue il metodo e ottiene il risultato (o l'eccezione).
        Serializza il risultato e lo restituisce come stringa JSON.  
        Questa istanza è di una classe fornita da MOD-RPC, quindi conosce le classi degli argomenti da passare e le
        eventuali eccezioni previste dal metodo.
    *   La tasklet ora verifica che la stringa `resp` restituita da `execute` sia un valido albero JSON. Può abortire
        il programma se non lo è.
    *   Prepara un albero JSON la cui radice è un nodo *OBJECT* con il membro **response** valorizzato con il nodo
        radice di *resp*.
    *   Genera una stringa dall'albero JSON.
    *   Trasmette la stringa (dopo i 4 bytes che ne indicano la lunghezza) al socket mittente.
*   Altrimenti:
    *   Viene avviata una nuova tasklet:
        *   Esegue `disp.execute()`.
*   Ascolta altre richieste.

### <a name="ZCD_tcp_client"></a>TCP - client

Per ridurre l'overhead di una connessione TCP, un client può stabilire una connessione TCP con un server ed
utilizzarla per diversi messaggi. Il metodo `tcp_client` restituisce un'istanza di TcpClient che rappresenta
una connessione con il server indicato.

#### cosa riceve

Il metodo `tcp_client` riceve:
*   L'indirizzo IP e la porta TCP del server da contattare.
*   La stringa in formato JSON che serializza l'identificativo di identità del mittente **source-id**.
*   La stringa in formato JSON che serializza l'identificativo di identità del destinatario **unicast-id**.

L'oggetto TcpClient che viene restituito memorizza queste informazioni. Inoltre esso incapsula un socket con
il quale verrà stabilita la connessione. Ma la connessione non viene inizialmente aperta.

### TcpClient

Ogni istanza di TcpClient lavora con il socket che incapsula. Tale oggetto può essere acceduto da diverse tasklet
che lavorano in contemporanea, ma come abbiamo detto non è possibile usare la stessa connessione contemporaneamente
per due operazioni. L'oggetto fornirà al suo utilizzatore questi metodi:

*   `bool is_queue_empty` - Verificare se la connessione è immediatamente disponibile per avviare una operazione
    (non ci sono altre operazioni in corso o accodate).
*   `string enqueue_call` - Accodare una operazione.  
    A questo metodo sono passati:

    *   La stringa con il nome del metodo da chiamare.
    *   Un array di stringhe in formato JSON, una per ogni argomento del metodo.
    *   Un booleano per indicare se si attende una risposta.

    Se la coda è vuota, questo metodo avvia una operazione nella connessione. Altrimenti accoda l'operazione,
    garantendo la sequenzialità delle operazioni accodate. In ogni caso, il metodo non ritorna fino a quando
    non ha eseguito l'operazione, cioè inviato il messaggio e ricevuto, se previsto, la risposta.  
    Se il booleano dice di non voler attendere una risposta, il metodo restituisce una stringa vuota non appena
    ha completato l'invio del messaggio. Altrimenti restituisce la risposta che riceve, che è una stringa in formato JSON.  
    Se questo metodo viene chiamato immediatamente dopo aver verificato che la connessione è disponibile
    (con il metodo precedente) si è certi che l'operazione in realtà non viene accodata, ma immediatamente avviata.

Di norma il client esegue queste operazioni. Per prima cosa crea una istanza di TcpClient con il metodo
`tcp_client` di ZCD e la memorizza per usarla in futuro. Quando vuole effettuare una chiamata non urgente
chiama direttamente il metodo `enqueue_call` sull'istanza che aveva memorizzata. Quando vuole effettuare una
chiamata urgente chiama il metodo `is_queue_empty` e se è vuota chiama immediatamente il metodo `enqueue_call`;
se invece non è vuota crea una nuova istanza di TcpClient con il metodo `tcp_client` di ZCD, la sostituisce
alla precedente e chiama su di essa il metodo `enqueue_call`.

Quando una operazione viene avviata, se la connessione non è aperta viene aperta. L'oggetto non può venire
distrutto (ci saranno riferimenti ad esso) fino a quando ci sono operazioni accodate o in corso. Quando
l'oggetto viene distrutto (rimossi tutti i riferimenti), se la connessione è aperta viene chiusa.

#### come procede una operazione

Se la connessione non era aperta il TcpClient si occupa di aprirla. Se si verifica un errore viene lanciata
una eccezione ZCDError.

Il TcpClient costruisce un albero JSON il cui nodo radice è un *OBJECT* con i membri:

*   Membro **method-name**. Il nome del metodo.
*   Membro **arguments**. Un array di nodi JSON. Ogni stringa JSON dei parametri viene parsata per costruire
    l'albero JSON e poi il nodo radice viene aggiunto a questo array.
*   Membro **source-id**. Un nodo JSON di tipo *OBJECT* che serializza l'identificativo di identità del mittente.
*   Membro **unicast-id**. Un nodo JSON di tipo *OBJECT* che serializza l'identificativo di identità del destinatario.
*   Membro **wait-reply**. Il booleano per indicare se si attende una risposta.

Dal JSON il TcpClient produce una stringa *msg*. Se durante la fase di costruzione di *msg* si verifica un
errore la libreria può abortire il programma.

Il TcpClient trasmette la stringa *msg* alla connessione, preceduta da 4 bytes che ne indicano la lunghezza.
Se si verifica un errore viene lanciata una eccezione ZCDError.

Se non si vuole attendere una risposta, il metodo restituisce immediatamente una stringa vuota.

Se bisogna attendere una risposta, il TcpClient la attende dalla stessa connessione, leggendola come sempre:
prima 4 bytes e poi il numero di bytes riportati. Se si verifica un errore viene lanciata una eccezione ZCDError.

La stringa ricevuta deve essere un valido albero JSON. Altrimenti viene lanciata una eccezione ZCDError.

Il nodo radice deve essere un *OBJECT* con il membro **response** che è un nodo valido come radice. Altrimenti viene
lanciata una eccezione ZCDError.

Il TcpClient costruisce un nuovo albero con tale nodo e genera una stringa da esso. Restituisce la stringa.

Se durante queste operazioni (cioè tra l'inizio della trasmissione del messaggio e la fine della ricezione della
risposta) si verifica un errore che porta a lanciare una eccezione ZCDError, prima il socket viene scartato e
sostituito con un nuovo socket costruito con gli stessi dati del precedente (indirizzo IP e porta TCP) non ancora
connesso. Poi l'operazione in corso lancia l'eccezione ZCDError. Le altre operazioni, che eventualmente erano
state accodate in precedenza, potranno essere avviate e di conseguenza una nuova connessione verrà tentata.

### <a name="ZCD_udp_listen"></a>UDP - server

Il metodo `udp_listen` avvia una tasklet che ascolta e gestisce i messaggi UDP provenienti dall'esterno.

#### cosa riceve

I dati che vengono passati al metodo `udp_listen` sono:

*   La porta UDP e l'interfaccia di rete.
*   Un delegato che ZCD potrà usare alla ricezione di un messaggio di tipo richiesta (si vedano sotto i
    vari tipi di messaggio UDP) per ottenere, se opportuno, un "dispatcher" a cui passare il messaggio per l'esecuzione.
*   Un delegato che ZCD potrà usare alla ricezione di un messaggio di tipo ACK o KEEP-ALIVE o risposta.
*   Un delegato che la tasklet in ascolto potrà usare in caso di errore prima di terminare.

Il delegato per udp_listen da usare alla ricezione di un messaggio di tipo richiesta è nella forma di una
implementazione dell'interfaccia IZcdUdpRequestMessageDelegate. I metodi di quest'ultima sono:

*   `IZcdDispatcher? get_dispatcher_unicast(int id, string unicast_id, string m_name, Gee.List<string> arguments, UdpCallerInfo caller_info)`
*   `IZcdDispatcher? get_dispatcher_broadcast(int id, string broadcast_id, string m_name, Gee.List<string> arguments, UdpCallerInfo caller_info)`

La classe UdpCallerInfo contiene i membri:

*   `string dev`
*   `string peer_address`
*   `string source_id` (serializzazione di un *identificativo di identità*)

L'interfaccia IZcdDispatcher è implementata da un oggetto che verrà istanziato alla chiamata di
`get_dispatcher_unicast` o `get_dispatcher_broadcast`. I suoi metodi sono:

*   `string execute()`

Il delegato per `udp_listen` da usare alla ricezione di altri tipi di messaggio è nella forma di una
implementazione dell'interfaccia IZcdUdpServiceMessageDelegate. I metodi di quest'ultima sono:

*   `bool is_my_own_message(int id)`
*   `void got_keep_alive(int id)`
*   `void got_response(int id, string response)`
*   `void got_ack(int id, string mac)`

Il delegato per `udp_listen` da usare in caso di errore nella preparazione del socket è nella forma di
una implementazione dell'interfaccia IZcdUdpCreateErrorHandler. I suoi metodi sono:

*   `void error_handler(Error e)`

#### cosa fa la tasklet che gestisce il socket in ascolto

La tasklet avviata dal metodo `udp_listen` istanzia un socket UDP per la ricezione di messaggi broadcast
(un IZcdServerDatagramSocket) associato all'interfaccia di rete e alla porta specificate. Nel caso di
errore lo passa al metodo `error_handler` e poi termina.

La tasklet mette il socket in attesa di un pacchetto con la chiamata `recvfrom`.

La chiamata `recvfrom` può ritornare con successo, fornendo il messaggio ricevuto e l'indirizzo e porta
del socket che ha inviato il messaggio, oppure segnalare un errore.

Nel caso di errore la tasklet produce un log e torna di seguito ad ascoltare (dopo un delay) con le stesse modalità.

Nel caso di ricezione di un messaggio la tasklet avvia una nuova tasklet per gestire il messaggio e poi torna ad ascoltare.

#### di quali tipi può essere un messaggio ricevuto in un pacchetto UDP

Il framework ZCD supporta messaggi col protocollo UDP solo trasmessi *in broadcast* sul proprio segmento di
rete, cioè con indirizzo di destinazione IPv4 255.255.255.255.

Ogni messaggio che il framework ZCD invia è costituito da un singolo pacchetto UDP. Visto che esiste un limite
massimo (teorico di quasi 64 KB) alla dimensione di un pacchetto UDP, c'è la possibilità che un messaggio di
grosse dimensioni sia impossibile da trasmettere correttamente con la modalità Unicast o Broadcast del framework.

Esistono diversi tipi di messaggio. Ogni messaggio è un albero JSON che ha come nodo radice un oggetto con
un membro. Dal nome del membro si deduce il tipo e di conseguenza come trattare il suo contenuto, che
comunque è sempre un oggetto. Di seguito elenchiamo le tipologie, indicando il nome del membro sull'oggetto
radice. Sotto ogni tipologia sono poi elencati i vari membri dell'oggetto.

*   Messaggio di richiesta *Unicast*.  
    Il membro ha nome "unicast-request" e il suo contenuto è un oggetto con i seguenti membri:

    *   "ID" - un intero che identifica la richiesta Unicast.
    *   "request" - un oggetto che codifica la chiamata, con i membri:
        *   "unicast-id" - un nodo che è valido come radice, quindi *OBJECT* o *ARRAY*. Tale nodo sarà riprodotto
            come stringa e passato al delegato. Identifica il nodo destinatario.
        *   "method-name" - una stringa con il significato descritto in precedenza nel capitolo tcp_listen.
        *   "arguments" - un array di oggetti con il significato descritto in precedenza.
        *   "source-id" - un nodo che è valido come radice, quindi *OBJECT* o *ARRAY*. Tale nodo sarà riprodotto
            come stringa e messo nel CallerInfo per essere passato al dispatcher. Identifica il nodo mittente.
        *   "wait-reply" - un booleano con il significato descritto in precedenza.

*   KEEP-ALIVE relativo ad una specifica richiesta *Unicast* con richiesta di risposta, che è stata ricevuta
    correttamente ed è ancora in fase di elaborazione.  
    Il membro ha nome "unicast-keepalive" e il suo contenuto è un oggetto con i seguenti membri:

    *   "ID" - un intero che identifica la richiesta Unicast da mantenere in vita.

*   Messaggio di risposta (ad una specifica richiesta *Unicast*).  
    Il membro ha nome "unicast-response" e il suo contenuto è un oggetto con i seguenti membri:

    *   "ID" - un intero che identifica la richiesta Unicast a cui si risponde.
    *   "response" - un nodo valido come radice con il significato descritto in precedenza nel capitolo tcp_listen.

*   Messaggio di richiesta *Broadcast*.  
    Il membro ha nome "broadcast-request" e il suo contenuto è un oggetto con i seguenti membri:

    *   "ID" - un intero che identifica la richiesta Broadcast.
    *   "request" - un oggetto che codifica la chiamata, con i membri:
        *   "broadcast-id" - un nodo che è valido come radice, quindi *OBJECT* o *ARRAY*. Tale nodo sarà riprodotto
            come stringa e passato al delegato. Identifica i nodi destinatari.
        *   "method-name" - una stringa con il significato descritto in precedenza nel capitolo tcp_listen.
        *   "arguments" - un array di oggetti con il significato descritto in precedenza.
        *   "source-id" - un nodo che è valido come radice, quindi *OBJECT* o *ARRAY*. Tale nodo sarà riprodotto
            come stringa e messo nel CallerInfo per essere passato al dispatcher. Identifica il nodo mittente.
        *   "send-ack" - un booleano che indica se si desidera il messaggio ACK.

*   ACK relativo ad una specifica richiesta *Broadcast* correttamente ricevuta.  
    Il membro ha nome "broadcast-ack" e il suo contenuto è un oggetto con i seguenti membri:

    *   "ID" - un intero che identifica la richiesta Broadcast di cui si intende segnalare la corretta ricezione.
    *   "MAC" - una stringa che rappresenta il MAC address della scheda di rete che ha ricevuto il messaggio, in
        esadecimale in maiuscolo e con il separatore ":". Ad esempio "02:AF:78:2E:C8:B6".

#### cosa fa la tasklet che gestisce un messaggio UDP

La tasklet che gestisce un messaggio riceve questi parametri iniziali:

*   `string msg` - Il messaggio ricevuto.
*   `string dev` - L'interfaccia di rete da cui è stato ricevuto.
*   `string peer_address` - L'indirizzo del mittente.
*   `uint16 peer_port` - La porta del mittente.
*   `IZcdUdpRequestMessageDelegate del_req` - Il delegato per le richieste.
*   `IZcdUdpServiceMessageDelegate del_ser` - Il delegato per i messaggi di servizio.

La tasklet prima di tutto verifica la correttezza del messaggio, che deve essere un valido albero JSON,
altrimenti termina. Analogamente, se il nodo radice non è di tipo oggetto la tasklet termina.

L'oggetto radice deve avere esattamente un membro. Il nome del membro deve essere uno di quelli indicati
precedentemente. Altrimenti la tasklet termina.

Se si tratta di una richiesta *Unicast*:

*   La tasklet verifica di poter estrapolare, pena la sua terminazione, dall'albero le seguenti informazioni:
    *   `int id`
    *   `string unicast_id` che è un valido albero JSON
    *   `string m_name`
    *   `Gee.List<string> arguments` che sono validi alberi JSON
    *   `string source_id` che è un valido albero JSON
    *   `bool wait_reply`
*   La tasklet chiama il metodo `is_my_own_message(id)` del delegato per i messaggi di servizio.  
    Se la risposta è True significa che il messaggio è partito dal mio nodo, quindi la tasklet lo ignora e
    termina. Questo controllo è necessario perché i pacchetti UDP con destinazione broadcast che il nodo
    trasmette su una interfaccia di rete vengono ricevuti anche dal nodo stesso tramite il socket in ascolto
    sulla stessa interfaccia, come quelli trasmessi da altri nodi.
*   Prepara una istanza di UdpCallerInfo con le informazioni che ha sul client, cioè l'indirizzo IP del client
    remoto e la scheda di rete dalla quale ha rilevato il messaggio e la stringa **source-id**.
*   Chiama il metodo `get_dispatcher_unicast` sulla sua istanza di IZcdUdpRequestMessageDelegate e ottiene un
    IZcdDispatcher *disp*.  
    Tale metodo a questo punto è in grado di stabilire se il messaggio è da processare e da quale sua
    *identità*; può esservene al massimo una in questo nodo. In questo caso ritorna una istanza apposita di
    IZcdDispatcher che ha memorizzato le informazioni per l'esecuzione del metodo. Altrimenti ritorna null.
*   Se *disp* = *null* il messaggio va ignorato. La tasklet termina.
*   Se il messaggio dichiarava di voler ricevere una risposta:
    *   Avvia una nuova tasklet `t_keepalive` con cui:
        *   Periodicamente, fino a che non viene terminata la tasklet:
            *   Prepara l'albero JSON contenente la keep-alive con quel id.
            *   Trasmette sull'interfaccia *dev* in broadcast UDP la stringa generata dall'albero JSON.
*   Esegue `resp = disp.execute()`.  
    L'oggetto IZcdDispatcher deserializza gli argomenti. Esegue il metodo e ottiene il risultato (o l'eccezione).
    Serializza il risultato e lo restituisce come stringa JSON.  
    Questa istanza è di una classe fornita da MOD-RPC, quindi conosce le classi degli argomenti da passare e
    le eventuali eccezioni previste dal metodo.
*   Se il messaggio dichiarava di voler ricevere una risposta:
    *   Termina la tasklet `t_keepalive`.
    *   La tasklet ora verifica che la stringa `resp` restituita da `execute` sia un valido albero JSON. Può
        abortire il programma se non lo è.
    *   La tasklet prepara l'albero JSON contenente la risposta a Unicast con quel id e questa `resp`.
    *   La tasklet trasmette sull'interfaccia *dev* in broadcast UDP la stringa generata dall'albero JSON.

Se si tratta di una KEEP-ALIVE:

*   La tasklet verifica di poter estrapolare, pena la sua terminazione, dall'albero le seguenti informazioni:
    *   `int id`
*   La tasklet chiama il metodo `got_keep_alive(id)` del delegato per i messaggi di servizio.

Se si tratta di una risposta a *Unicast*:

*   La tasklet verifica di poter estrapolare, pena la sua terminazione, dall'albero le seguenti informazioni:
    *   `int id`
    *   `string response` che è un valido albero JSON
*   La tasklet chiama il metodo `got_response(id, response)` del delegato per i messaggi di servizio.

Se si tratta di una richiesta *Broadcast*:

*   La tasklet verifica di poter estrapolare, pena la sua terminazione, dall'albero le seguenti informazioni:
    *   `int id`
    *   `string broadcast_id` che è un valido albero JSON
    *   `string m_name`
    *   `Gee.List<string> arguments` che sono validi alberi JSON
    *   `string source_id` che è un valido albero JSON
    *   `bool send_ack`
*   La tasklet chiama il metodo `is_my_own_message(id)` del delegato per i messaggi di servizio.  
    Se la risposta è True significa che il messaggio è partito dal mio nodo, quindi la tasklet lo ignora e termina.
*   Se il messaggio dichiarava di voler ricevere un acknowledge:
    *   Avvia una nuova tasklet con cui:
        *   Per 3 volte a brevi intervalli (casuali tra 10 e 200 msec):
            *   Prepara l'albero JSON contenente la ACK con quel id e il MAC address dell'interfaccia *dev*.
            *   Trasmette sull'interfaccia *dev* in broadcast UDP la stringa generata dall'albero JSON.
*   Prepara una istanza di UdpCallerInfo con le informazioni che ha sul client, cioè l'indirizzo IP del client
    remoto e la scheda di rete dalla quale ha rilevato il messaggio e la stringa **source-id**.
*   Chiama il metodo `get_dispatcher_broadcast` sulla sua istanza di IZcdUdpRequestMessageDelegate e ottiene un IZcdDispatcher *disp*.  
    Tale metodo a questo punto è in grado di stabilire se il messaggio è da processare e da quali sue *identità*;
    possono esservene più di una in questo nodo. In questo caso ritorna una istanza apposita di IZcdDispatcher che
    ha memorizzato le informazioni per l'esecuzione del metodo. Altrimenti ritorna null.
*   Se *disp* = *null* il messaggio va ignorato. La tasklet termina.
*   Esegue `disp.execute()`.  
    L'oggetto IZcdDispatcher deserializza gli argomenti. Esegue il metodo su ogni *identità* interessata. Non si
    cura del risultato (o dell'eccezione) di ogni chiamata. Restituisce comunque una stringa vuota.

Se si tratta di una ACK:

*   La tasklet verifica di poter estrapolare, pena la sua terminazione, dall'albero le seguenti informazioni:
    *   `int id`
    *   `string mac`
*   La tasklet chiama il metodo `got_ack(id, mac)` del delegato per i messaggi di servizio.

### <a name="ZCD_UDP"></a>UDP - client

Su iniziativa dell'utilizzatore di ZCD, per trasmettere una richiesta con il protocollo UDP a un nodo (o più
di uno) diretto vicino, possono essere trasmessi i seguenti messaggi:

*   Richiesta *Unicast*.  
    L'utilizzatore di ZCD chiama la funzione `send_unicast_request` passando questi argomenti:
    *   L'identificativo `int id` della richiesta Unicast.
    *   La stringa in formato JSON per l'oggetto `unicast_id`.
    *   La stringa con il nome del metodo da chiamare.
    *   Un array di stringhe in formato JSON, una per ogni argomento del metodo.
    *   La stringa in formato JSON per l'oggetto `source_id`.
    *   Un booleano per indicare se si attende una risposta.
    *   `string dev`.
    *   `uint16 port`.
    *   opzionalmente, `string? src_ip`.
*   Richiesta *Broadcast*.  
    L'utilizzatore di ZCD chiama la funzione `send_broadcast_request` passando questi argomenti:
    *   L'identificativo `int id` della richiesta Broadcast.
    *   La stringa in formato JSON per l'oggetto `broadcast_id`.
    *   La stringa con il nome del metodo da chiamare.
    *   Un array di stringhe in formato JSON, una per ogni argomento del metodo.
    *   La stringa in formato JSON per l'oggetto `source_id`.
    *   Un booleano per indicare se si vuole il messaggio di ACK.
    *   `string dev`.
    *   `uint16 port`.
    *   opzionalmente, `string? src_ip`.

Tutte le funzioni sopra elencate hanno una implementazione molto semplice: costruiscono l'albero JSON dai
dati passati, producono una stringa e la trasmettono in broadcast sull'interfaccia di rete e verso la porta
indicati. Non hanno altri compiti.

L'utilizzatore di ZCD dovrà aver cura di creare un *id* univoco per ogni messaggio inviato e fare in modo
di farlo conoscere all'istanza di IZcdUdpServiceMessageDelegate (il delegato per i messaggi di servizio)
che aveva passata alla funzione `udp_listen` relativa a *dev*. Tale istanza infatti sarà invocata alla
ricezione dei messaggi relativi (keepalive, risposta a unicast, ack).

Tutte le funzioni sopra elencate possono lanciare un ZCDError nel caso di problemi nella trasmissione della
stringa JSON sulla rete. Per la costruzione del JSON invece non sono previsti errori; in particolare se le
stringhe passate che devono contenere un valido JSON (`source_id`, `unicast_id`, `broadcast_id` e gli
argomenti dei metodi) non sono ben formate le funzioni abortiscono il programma.

## <a name="MODRPC"></a>MOD-RPC

La libreria di livello intermedio può essere realizzata con il tool "rpcdesign" partendo da una descrizione
delle interfacce degli oggetti remoti. In questa sezione descriveremo le caratteristiche della libreria che
si ottiene in questo modo.

La descrizione delle interfacce degli oggetti remoti viene preparata dallo sviluppatore in un formato apposito,
detto [RPC-IDL](ZcdRpcidl.md), nel file "interfaces.rpcidl". Poi viene eseguito il comando "rpcdesign", il
quale produce i sorgenti Vala con cui si realizza la libreria. Una descrizione dettagliata del formato è data
nel relativo documento; qui diciamo solo che esso consente di indicare un numero di classi radice, dette
*root-dispatcher*; per ognuna di esse si possono indicare un numero di classi remote, dette *moduli*; per
ognuna di esse si possono indicare un numero di metodi remoti con la loro signature.

I sorgenti prodotti incapsulano tutto nel namespace "SampleRpc", che lo sviluppatore se vuole può facilmente
modificare agendo solo sulle prime righe dei file.

### <a name="MODRPC_serializzabili"></a>Oggetti serializzabili

Nella descrizione dei metodi remoti possono essere indicati i nomi di classi e interfacce le cui istanze sono
oggetti serializzabili da passare come argomenti o da ricevere come risultati. A tali oggetti vanno aggiunti
quelli usati come `source_id`, `unicast_id` e `broadcast_id`.

La libreria fornisce le interfacce ISourceID, IUnicastID, IBroadcastID. Inoltre, per ogni nome di tipo indicato
nella descrizione dei metodi remoti come argomento o come valore di ritorno, oltre ai tipi base, fornisce o una
classe o una interfaccia (se il nome inizia con una 'I' maiuscola seguita da un'altra lettera maiuscola). Se si
tratta di una classe, il tool "rpcdesign" la prepara vuota. Sarà lo sviluppatore a completarla con i dati che
servono e ad accertarsi che possa essere serializzata e deserializzata con `Json.gobject_serialize` e
`Json.gobject_deserialize` di JsonGlib.

Se si tratta di una interfaccia (come per le interfacce ISourceID, IUnicastID, IBroadcastID) il tool "rpcdesign"
la prepara vuota. Il programmatore potrà anche optare per lasciare la libreria inalterata, come preparata dal
tool "rpcdesign", e fornire la classe che la implementa direttamente nell'applicazione che usa la libreria di
livello intermedio. Anche in questo caso, in tale applicazione, dovrà accertarsi che possa essere serializzata
e deserializzata con `Json.gobject_serialize` e `Json.gobject_deserialize` di JsonGlib.

### <a name="MODRPC_server"></a>Lato server

La libreria fornisce una interfaccia skeleton per la classe radice e una per ogni modulo membro.

La libreria fornisce inoltre una interfaccia per il delegato che essa stessa userà per reperire le istanze della
classe radice. Infine fornisce una interfaccia per il gestore di errori; questo sarà invocato nelle tasklet che
stanno in ascolto sui socket in caso di errori che causano la terminazione delle stesse tasklet.

L'applicazione APP che voglia essere in grado di fare da server dovrà fornire una implementazione per ognuna di
queste interfacce. L'istanza del delegato e quella del gestore di errori andranno passate alla libreria nella
fase di inizializzazione, cioè, come vedremo dopo, quando si avviano le tasklet che stanno in ascolto sui socket.

La libreria, attraverso il delegato, sarà in grado, quando riceve una richiesta, di ottenere le istanze di classi
radice e classi modulo al fine di chiamare un metodo remoto.

Facciamo un esempio. Il file interfaces.rpcidl presenta la classe radice `NodeManager node_manager`. Ha un
membro `InfoManager info_manager`. Questo ha il metodo `string get_name() throws AccessDeniedError`. La libreria
prodotta da "rpcdesign" fornirà queste interfacce:

*   SampleRpc.IInfoManagerSkeleton.  
    Prevede il metodo astratto:
    *   `string get_name(SampleRpc.CallerInfo? caller=null) throws AccessDeniedError`
*   SampleRpc.INodeManagerSkeleton.  
    Prevede il metodo astratto:
    *   `SampleRpc.IInfoManagerSkeleton info_manager_getter()`
*   SampleRpc.IRpcDelegate.  
    Prevede il metodo astratto:
    *   `Gee.List<SampleRpc.INodeManagerSkeleton> get_node_manager_set(SampleRpc.CallerInfo caller)`
*   SampleRpc.IRpcErrorHandler.  
    Prevede il metodo astratto:
    *   `void error_handler(Error e)`

**Nota 1**: Tutti i metodi remoti nell'interfaccia skeleton ricevono un argomento aggiuntivo SampleRpc.CallerInfo
che contiene alcune informazioni sul richiedente. Il contenuto varia a seconda del metodo usato per inviare il
messaggio (TcpClient, Unicast, Broadcast).

**Nota 2**: L'interfaccia IRpcDelegate ha un metodo per ogni root-dispatcher. Per ogni richiesta che si riceve
lato server la prima parte del nome del metodo remoto indica il root-dispatcher da usare; la libreria MOD-RPC
prende questo prefisso e sceglie il metodo di IRpcDelegate da chiamare.

La libreria MOD-RPC fornisce inoltre alcune chiamate di inizializzazione:

*   `void SampleRpc.init_tasklet_system(ITasklet _tasklet)`
*   `void SampleRpc.tcp_listen(SampleRpc.IRpcDelegate dlg, SampleRpc.IRpcErrorHandler err, uint16 port, string? my_addr = null)`
*   `void SampleRpc.udp_listen(SampleRpc.IRpcDelegate dlg, SampleRpc.IRpcErrorHandler err, uint16 port, string dev)`

La funzione `init_tasklet_system` deve essere chiamata da APP come inizializzazione della libreria, sia che APP
faccia da client, sia che faccia da server. Essa serve a passare l'implementazione del sistema di tasklet,
richiesto dalla libreria ZCD.

Le funzioni `tcp_listen` e `udp_listen` vanno chiamate da APP solo se APP è interessata a ricevere connessioni TCP
o pacchetti UDP. C'è una differenza tra le due.

La funzione `tcp_listen` va chiamata da APP se vuole essere in grado di ricevere connessioni di iniziativa di un
nodo remoto. Questa chiamata non è necessaria se APP vuole solo iniziare connessioni verso un nodo remoto. In
questo caso, dopo aver iniziato la connessione la APP sarebbe comunque in grado di inviare messaggi e anche di
ricevere risposte, attraverso la stessa connessione.

La funzione `udp_listen`, invece, va chiamata da APP se vuole essere in grado di ricevere pacchetti da un nodo remoto.
Senza questa chiamata APP sarebbe comunque in grado di inviare pacchetti UDP ad un nodo remoto, ma non di ricevere risposte.

La funzione `tcp_listen` e la funzione `udp_listen` avviano entrambe una nuova tasklet perché stia in ascolto di
connessioni o messaggi. Queste due tipologie di tasklet producono due diversi tipi di oggetti CallerInfo: la prima per
ogni connessione produce una istanza di SampleRpc.TcpclientCallerInfo, la seconda per ogni messaggio ricevuto
produce o una istanza di SampleRpc.UnicastCallerInfo oppure di SampleRpc.BroadcastCallerInfo.

Di norma si chiama la funzione `tcp_listen` una sola volta per gestire tutte le connessioni provenienti da qualsiasi
interfaccia di rete e per qualsiasi indirizzo locale del nodo. Invece si chiama la funzione `udp_listen` una volta
per ogni interfaccia di rete, per gestire i pacchetti rilevati tramite quella interfaccia.

Ad esempio si può avviare una tasklet che ascolta i messaggi TCP per qualsiasi indirizzo dell'host + una tasklet che
ascolta i messaggi UDP sulla interfaccia "eth0" + una tasklet che ascolta i messaggi UDP sulla interfaccia "wlan0", e così via.

Per ognuna di queste tasklet in ascolto, l'applicazione APP ha inizialmente fornito un delegato *dlg* che è una istanza
di SampleRpc.IRpcDelegate e un gestore di errori *err* che è una istanza di SampleRpc.IRpcErrorHandler.

Supponiamo ora che una di queste tasklet ha ricevuto un messaggio, composto dal nome del metodo e dai suoi argomenti.
Inoltre ha composto il CallerInfo *caller* il quale contiene ora il `broadcast_id` o il `unicast_id`, oltre che il
`source_id` e altre info sulla comunicazione di rete attraverso cui è giunto il messaggio.

Continuando con l'esempio precedente, la libreria MOD-RPC chiama
`Gee.List<INodeManagerSkeleton> root = dlg.get_node_manager_set(caller);`. Questo metodo implementato dall'istanza
fornita da APP decide in base a questo *caller* se restituire zero, una o più istanze dell'interfaccia skeleton
della classe radice. Infatti, il delegato della libreria ZCD per restituire un *dispatcher* deve basarsi sul
`unicast_id` o sul `broadcast_id`; individua zero, una o più *identità* del suo nodo che sono destinatari del messaggio.

Sulla base di questa lista di istanze skeleton, se non è vuota, viene poi preparata una istanza di IZcdDispatcher.
Su questa istanza verrà ora chiamato il metodo `execute`. In tale metodo essa esamina il nome del metodo, individua
il percorso da fare (es: `root.info_manager.get_name`), deserializza gli argomenti. Poi, per ogni istanza skeleton
che era nella lista, chiama il metodo. Se le istanze sono più d'una il metodo `execute` restituirà stringa vuota
poiché la chiamata è in modalità Broadcast.

Se la chiamata era in modalità Broadcast o se una risposta non era richiesta, allora la chiamata (o le chiamate)
al metodo remoto sono fatte in una nuova tasklet e il metodo subito restituisce una stringa vuota. Altrimenti la
chiamata (in questo caso sicuramente una) al metodo remoto viene fatta, si ottiene l'esito (il valore di ritorno
o una eccezione) lo si serializza e si restituisce nella forma di una stringa.

Se durante la deserializzazione degli argomenti si riscontra un problema (ad esempio non sono del tipo previsto
dalla signature del metodo, oppure un oggetto è di una classe sconosciuta al programma, oppure un oggetto viene
deserializzato ma non contiene tutte le proprietà requisito della sua classe) il metodo `execute` dell'istanza
di IZcdDispatcher non richiama affatto il metodo remoto, invece serializza e restituisce una eccezione DeserializeError.

Le classi fornite da APP per implementare le interfacce skeleton, in particolare quelle dei singoli moduli, avranno
quindi solo il compito di implementare la business logic dei singoli metodi remoti come fossero comuni metodi.
Riceveranno i parametri previsti dalla signature, potranno elaborarli e restituire un valore di ritorno del tipo
previsto dalla signature oppure lanciare una delle eccezioni previste dalla signature.

### <a name="MODRPC_client"></a>Lato client

Per l'applicazione APP che voglia essere in grado di fare da client, invece, la libreria MOD-RPC fornisce una
interfaccia stub per la classe radice e una per ogni modulo membro. La libreria fornisce alcune funzioni all'applicazione
per ottenere una istanza dello stub radice, una funzione per ogni modalità di invio del messaggio.

#### Interfacce

Utilizzando il file interfaces.rpcidl riportato nell'esempio precedente le interfacce prodotte sono:

*   IInfoManagerStub  
    Prevede il metodo astratto:
    *   `string get_name() throws AccessDeniedError, StubError, DeserializeError`
*   INodeManagerStub.  
    Prevede il metodo astratto:
    *   `IInfoManagerStub info_manager_getter()`

I metodi remoti che possono essere chiamati su uno stub prevedono tutti due possibili eccezioni oltre alle
altre eventualmente descritte nel file interfaces.rpcidl. Una è StubError; può essere lanciata dalla libreria
nel nodo locale per segnalare un errore nella fase di trasmissione del messaggio o ricezione della risposta.
L'altra è DeserializeError; può essere trasmessa (in forma serializzata) come risposta dalla libreria nel nodo
remoto se ha riscontrato problemi nel deserializzare gli argomenti che il nodo locale ha passato; oppure può
essere lanciata dalla libreria nel nodo locale per segnalare un problema nel deserializzare il valore di ritorno.

Un caso particolare è il valore di ritorno di un metodo chiamato in modalità Broadcast oppure con `wait_reply`
a *false*. In questi casi lo stub non attende una risposta e quindi il metodo che è stato chiamato non ha un
valore da restituire. Se il metodo doveva restituire *void* non è un problema e questo è il caso più comune per
questi tipi di chiamata a metodo remoto. Tuttavia qualche metodo che restituisce un valore può essere a volte
chiamato in questa modalità. In questo caso il metodo dello stub lancia l'eccezione StubError con il codice
`DID_NOT_WAIT_REPLY`. L'applicazione APP deve essere consapevole di tale possibile situazione che in realtà indica
l'avvenuta corretta trasmissione del messaggio.

Le classi stub fornite dalla libreria MOD-RPC hanno il compito di serializzare gli argomenti, inviare il messaggio
attraverso la rete, ricevere la risposta (valore di ritorno o eccezione), deserializzarla e restituirla.

#### Funzioni

Le funzioni fornite dalla libreria per ottenere una istanza dello stub radice sono:

*   `INodeManagerStub get_node_manager_tcp_client(string peer_address, uint16 peer_port, ISourceID source_id, IUnicastID unicast_id)`
*   `INodeManagerStub get_node_manager_unicast(string dev, uint16 port, string src_ip, ISourceID source_id, IUnicastID unicast_id, bool wait_reply)`
*   `INodeManagerStub get_node_manager_broadcast(Gee.List<string> devs, Gee.List<string> src_ips, uint16 port, ISourceID source_id, IBroadcastID broadcast_id, SampleRpc.IAckCommunicator? notify_ack=null)`

La classe che si ottiene con la funzione `get_node_manager_tcp_client` implementa anche l'interfaccia ITcpClientRootStub.
Tale interfaccia offre questi metodi:

*   `bool get_hurry()`
*   `void set_hurry(bool new_value)`  
    L'impostazione di `hurry` dice se il prossimo metodo chiamato è urgente.
*   `bool get_wait_reply()`
*   `void set_wait_reply(bool new_value)`  
    L'impostazione di `wait_reply` dice se per il prossimo metodo chiamato si desidera attendere una risposta.

La funzione `get_node_manager_broadcast` accetta una istanza di SampleRpc.IAckCommunicator. Se non è *null* allora
lo stub prodotto, quando dovrà chiamare un metodo remoto, richiederà l'invio di un messaggio di ACK da parte dei
nodi che ricevono il messaggio. Poi avvierà una nuova tasklet prima di terminare (con void oppure lanciando
l'eccezione `DID_NOT_WAIT_REPLY`). Nella nuova tasklet attenderà il tempo massimo e infine chiamerà il seguente
metodo dell'interfaccia SampleRpc.IAckCommunicator:

*   `void process_macs_list(Gee.List<string> macs_list)`  
    A parametro riceve l'elenco dei MAC address dei vicini che hanno ricevuto il messaggio.

Inoltre, la funzione `get_node_manager_broadcast` accetta una lista di interfacce di rete e una lista di relativi
indirizzi IP. Lo stub prodotto, quando dovrà chiamare un metodo remoto, deve trasmettere (o cercare di trasmettere)
il messaggio su tutte le interfacce di rete indicate, per ognuna usando il relativo indirizzo IP come *source*.
Se si verifica un errore nella trasmissione su tutte le interfacce di rete allora la chiamata al metodo remoto lancia uno StubError.

Ma se almeno una trasmissione non ha dato problemi il chiamante non vede problemi.

### <a name="MODRPC_argomenti"></a>Serializzazione e deserializzazione di argomenti, valori di ritorno e eccezioni

Come detto nell'analisi funzionale, la libreria MOD-RPC ha il compito di:

*   Lato client serializzare ogni argomento di una chiamata a metodo remoto in una stringa che rappresenta un
    valido albero JSON.
*   Lato client serializzare l'istanza di ISourceID e l'istanza di IUnicastID / IBroadcastID della chiamata in
    una stringa che rappresenta un valido albero JSON.
*   Lato server deserializzare ogni stringa ricevuta (compreso `source_id` e `unicast_id` / `broadcast_id`) nel
    relativo argomento.
*   Lato server serializzare l'esito di una chiamata (può essere il valore di ritorno o una eccezione dichiarata
    nella signature) in una stringa che rappresenta un valido albero JSON.
*   Lato client deserializzare l'esito, distinguendo se si tratta del valore di ritorno o di una eccezione.

La libreria MOD-RPC prodotta con il tool "rpcdesign" si avvale della libreria JsonGlib per generare e leggere stringhe JSON.

#### Serializzazione di un argomento

La classe stub conosce la signature del metodo, quindi per ogni argomento ha il valore e il tipo atteso. Avendo
ricevuto la chiamata come un normale metodo di una classe Vala, ha già fatto un controllo sulla validità degli argomenti.

Per gli argomenti del metodo remoto, ogni albero JSON ha per radice un oggetto con un singolo membro chiamato
"argument". Il suo valore è un nodo JSON. Vediamo in dettaglio per ogni possibile situazione cosa contiene tale nodo.

*   Se il tipo atteso è null-able e il valore è null, qualsiasi sia il tipo, il nodo è un nodo *NULL*.
*   Se il tipo atteso è int64 il nodo è un nodo *VALUE* impostabile con `set_int` di JsonGlib. In questo caso rientra
    anche un tipo che si possa castare a int64 senza perdere informazioni. In questo caso rientra anche un tipo null-able,
    sapendo ormai che il valore effettivo non è null.
*   Se il tipo atteso è double il nodo è un nodo *VALUE* impostabile con `set_double` di JsonGlib. In questo caso rientra
    anche un tipo che si possa castare a double senza perdere informazioni. In questo caso rientra anche un tipo null-able,
    sapendo ormai che il valore effettivo non è null.
*   Se il tipo atteso è boolean il nodo è un nodo *VALUE* impostabile con `set_boolean` di JsonGlib. In questo caso rientra
    anche un tipo null-able, sapendo ormai che il valore effettivo non è null.
*   Se il tipo atteso è string il nodo è un nodo *VALUE* impostabile con `set_string` di JsonGlib. In questo caso rientra
    anche un tipo null-able, sapendo ormai che il valore effettivo non è null.
*   Se il tipo atteso è binario, cioè un `uint8[]`, il nodo è un nodo *VALUE* impostabile con `set_string` di JsonGlib. In
    questo caso rientra anche un `uint8[]?`, sapendo ormai che il valore effettivo non è null. La stringa che rappresenta
    l'array di bytes è ottenuta codificando in Base64.
*   Se il tipo atteso è una istanza di una classe che deriva da GLib.Object il nodo è un nodo *OBJECT*. In questo caso rientra
    anche una interfaccia che richiede Object. In questo caso rientra anche un tipo null-able, sapendo ormai che il valore
    effettivo non è null.  
    In questo nodo *OBJECT* abbiamo 2 membri, uno chiamato "typename" e l'altro "value". Nel membro typename viene messo un
    nodo *VALUE* di tipo stringa con il nome del tipo dell'oggetto *obj* che si ottiene con `obj.get_type().name()`. Nel membro
    value viene messo un nodo che è la serializzazione dell'oggetto *obj* che si ottiene con `Json.gobject_serialize(obj)`.  
    Rimandiamo alla documentazione di JsonGlib per capire cosa contiene il nodo che si ottiene con questa serializzazione.
    Brevemente diciamo solo che in generale è un nodo *OBJECT* con tanti membri quante sono le proprietà con accesso pubblico
    che sono nell'oggetto serializzato; se una proprietà è a sua volta un oggetto questo è serializzato ricorsivamente; se la
    classe dell'oggetto serializzato implementa l'interfaccia Json.Serializable allora ha maggior controllo sul nodo che
    viene prodotto.
*   Se il tipo atteso è una lista Gee.List il nodo è un nodo *ARRAY*. In questo caso rientra anche un tipo null-able,
    sapendo ormai che il valore effettivo non è null.  
    Gli elementi di questo nodo *ARRAY* sono nodi (non alberi) trattati ognuno come sarebbe trattato il nodo di un
    singolo argomento.
*   Altri tipi di argomento non sono supportati direttamente dal codice prodotto da "rpcdesign". Lo sviluppatore potrà
    sviluppare per essi una classe che implementi l'interfaccia Json.Serializable e sia in grado di serializzare e
    deserializzare le informazioni necessarie.

#### Serializzazione di ISourceID, IUnicastID, IBroadcastID

Per serializzare una istanza dell'oggetto che si usa come ISourceID, o di quello che si usa come IUnicastID, o di quello
che si usa come IBroadcastID, si opera come per un argomento che è una istanza (non null) di una classe che deriva da
GLib.Object. Cioè l'albero JSON ha per radice un oggetto con i membri "typename" e "value".

#### Note sulla deserializzazione di un Object

Per deserializzare un oggetto con il metodo `gobject_deserialize` fornito dalla libreria JsonGlib occorre conoscere il
tipo. Per questo ogni Object viene trasmesso sotto forma di un nodo *OBJECT* con i membri "typename" e "value".

Il membro "typename" è un nodo *VALUE* che contiene la stringa *typename* con cui ottenere il Type. La libreria MOD-RPC
esegue `Type type = Type.from_name(typename);`.

L'esecuzione del metodo `Type.from_name` non andrà a buon fine se l'applicazione che vuole deserializzare non ha già
usato almeno una volta la classe *typename*. Questo può essere fatto, ad esempio per la classe License, con il codice
`typeof(License).class_peek();`. Questa "inizializzazione" va fatta dall'applicazione APP. L'implementazione di questa
inizializzazione non può essere demandata alla libreria MOD-RPC, perché alcune classi serializzabili potrebbero essere
fornite da APP e usate in metodi remoti che accettano una interfaccia o una classe base.

Questa inizializzazione da parte di APP va fatta sia per l'applicazione che fa da server, sia per l'applicazione che
fa da client. In particolare il server deve assicurarsi di registrare le classi che può ricevere come argomenti, mentre
il client le classi che può ricevere come valore di ritorno dei metodi remoti.

Il membro "value" è il vero *node* da passare alla deserializzazione. La libreria MOD-RPC esegue
`GLib.Object obj = Json.gobject_deserialize(type, node);`. Questa chiamata, potrebbe apparire strano, non prevede
alcun tipo di eccezione.

Come detto in precedenza, rimandiamo alla documentazione di JsonGlib per dettagli su come questa deserializzazione
avviene. Diciamo solo che la libreria istanzia il tipo indicato e cerca di valorizzare le proprietà (che devono
avere accesso pubblico sia in *get* che in *set*) che riconosce sia nell'albero JSON sia nell'oggetto istanziato.
Ma se ci sono proprietà nell'oggetto che non sono menzionate nel JSON semplicemente le lascia con il loro valore di
default; e se ci sono membri nel JSON che non sono presenti nell'oggetto semplicemente li ignora. Per questo la
libreria MOD-RPC fornisce, come misura di ulteriore verifica, una interfaccia SampleRpc.ISerializable.

Quindi, se il tipo *type* implementa l'interfaccia SampleRpc.ISerializable, allora la libreria richiama sull'oggetto
appena deserializzato il metodo `bool check_serialization()` di tale interfaccia. In tale metodo l'oggetto verifica
che tutte le proprietà necessarie (dette proprietà requisito) siano state correttamente valorizzate. Altrimenti (se
ritorna *false*) la libreria MOD-RPC considera fallita la deserializzazione.

#### Deserializzazione di un argomento

La classe skeleton riceve la stringa JSON di un argomento dalla libreria ZCD. Essa ha già verificato che questa
rappresenti un valido albero JSON. Se questo requisito non è soddisfatto è quindi lecito che la libreria MOD-RPC
abortisca il programma.

Ogni argomento deve essere un nodo di tipo *OBJECT* con un membro chiamato "argument". Altrimenti verrà lanciato
un DeserializeError.

La classe skeleton esamina il contenuto del nodo "argument":

*   Se è un nodo di tipo *NULL*:
    *   Se il tipo atteso è null-able allora sarà passato un null.
    *   Altrimenti verrà lanciato un DeserializeError.
*   Se è un nodo *VALUE*:
    *   Se il tipo atteso è intero, anche se null-able (`int`, `int?`, `int8`, `uint8?`, `uint16`, ...):
        *   Se il nodo si può leggere con `get_int` e il valore rientra nel range del tipo atteso, allora sarà passato il valore.
        *   Altrimenti verrà lanciato un DeserializeError.
    *   Se il tipo atteso è decimale, anche se null-able (`float`, `double?`, ...):
        *   Se il nodo si può leggere con `get_double` e il valore rientra nel range del tipo atteso, allora sarà passato il valore.
        *   Altrimenti verrà lanciato un DeserializeError.
    *   Se il tipo atteso è booleano, anche se null-able (`bool`, `bool?`):
        *   Se il nodo si può leggere con `get_boolean` allora sarà passato il valore.
        *   Altrimenti verrà lanciato un DeserializeError.
    *   Se il tipo atteso è string, anche se null-able (`string`, `string?`):
        *   Se il nodo si può leggere con `get_string` allora sarà passato il valore.
        *   Altrimenti verrà lanciato un DeserializeError.
    *   Se il tipo atteso è binario, anche se null-able (`uint8[]`, `uint8[]?`):
        *   Se il nodo si può leggere con `get_string` e la stringa si può decodificare con Base64 allora sarà passata tale decodifica.
        *   Altrimenti verrà lanciato un DeserializeError.
*   Se è un nodo *ARRAY*:
    *   Se il tipo atteso è Gee.List:
        *   La classe skeleton prepara un ArrayList del tipo previsto nella Gee.List.
        *   Ricorsivamente la classe skeleton esamina ogni nodo elemento dell'ARRAY con il tipo previsto nella Gee.List e
            aggiunge il valore all' ArrayList.
        *   Sarà passato l' ArrayList.
    *   Altrimenti verrà lanciato un DeserializeError.
*   Se è un nodo *OBJECT*:
    *   La classe skeleton verifica che tale oggetto abbia i due membri "typename" e "value". Altrimenti verrà lanciato
        un DeserializeError.
    *   Il membro "typename" è un nodo *VALUE* che contiene la stringa *typename* con cui ottenere il Type. Altrimenti
        verrà lanciato un DeserializeError.
    *   Esegue `Type type = Type.from_name(typename);`. Se la classe non esiste verrà lanciato un DeserializeError.
    *   Se la classe non è del tipo atteso o un suo derivato verrà lanciato un DeserializeError.
    *   Il membro "value" è il vero *node* da passare alla deserializzazione. Esegue
        `GLib.Object obj = Json.gobject_deserialize(type, node);`.
    *   Se *type* implementa l'interfaccia SampleRpc.ISerializable:
        *   Esegue `((SampleRpc.ISerializable)obj).check_serialization()` e se ritorna *false* verrà lanciato un DeserializeError.
    *   Sarà passato *obj*.

#### Deserializzazione di ISourceID, IUnicastID, IBroadcastID

La libreria MOD-RPC riceve la stringa JSON di tali identificativi dalla libreria ZCD. Essa ha già verificato che
questa rappresenti un valido albero JSON. Se questo requisito non è soddisfatto è quindi lecito che la libreria
MOD-RPC abortisca il programma.

Ogni argomento deve essere un nodo di tipo *OBJECT*. Altrimenti verrà lanciato un DeserializeError.

Per il resto, la libreria MOD-RPC opera come per un argomento che è una istanza (non null) di una classe che
deriva da GLib.Object. Cioè l'albero JSON ha per radice un oggetto con i membri "typename" e "value".

#### Serializzazione del valore di ritorno

La classe skeleton conosce il tipo del valore di ritorno del metodo remoto. Se la chiamata del metodo non produce
eccezioni, la classe skeleton si occupa di serializzare il risultato in un albero JSON e generare una stringa da esso.

Per un valore di ritorno del metodo remoto, ogni albero JSON ha per radice un oggetto con un singolo membro
chiamato "return-value". Il suo valore è un nodo JSON realizzato in modo analogo a quanto abbiamo visto sulla
serializzazione di un argomento. Nel caso di un metodo che restituisce void è un nodo *NULL*.

#### Serializzazione delle eccezioni

La classe skeleton conosce i tipi di eccezione previsti dal metodo remoto. Se la chiamata del metodo solleva una
eccezione, la classe skeleton la gestisce e si occupa di serializzarla in un albero JSON e generare una stringa da esso.

Per una eccezione, ogni albero JSON ha per radice un oggetto con 3 membri di tipo stringa i cui nomi sono "error-domain",
"error-code" e "error-message" e i cui valori sono le rappresentazioni stringa dei relativi valori della struttura di errore.

#### Deserializzazione del valore di ritorno

La classe stub conosce il tipo del valore di ritorno e i tipi di eccezione previsti dal metodo remoto.

La classe stub riceve la stringa JSON del valore di ritorno (risultato o eccezione) dalla libreria ZCD. Essa ha già
verificato che questa rappresenti un valido albero JSON. Se questo requisito non è soddisfatto è quindi lecito che la
libreria MOD-RPC abortisca il programma.

La radice deve essere un nodo di tipo *OBJECT* con un membro chiamato "return-value" oppure con i 3 membri "error-domain",
"error-code" e "error-message". Altrimenti verrà lanciato un DeserializeError.

Se è "return-value", il suo valore è un nodo JSON che va interpretato in modo analogo a quanto abbiamo visto sulla
deserializzazione di un argomento. Il valore di ritorno deserializzato verrà restituito dal metodo della classe stub.

Se è "error...", la classe stub:

*   Verifica che i 3 membri "error-domain", "error-code" e "error-message" contengano stringhe. Altrimenti verrà
    lanciato un DeserializeError.
*   Verifica che "error-domain" sia fra le possibili eccezioni del metodo. Altrimenti verrà lanciato un DeserializeError.
*   Verifica che esista il codice "error-code". Altrimenti verrà lanciato un DeserializeError.
*   Crea e solleva l'eccezione indicata da "error-domain", "error-code" e "error-message".

#### Specifiche di trasmissione e ricezione

**Modo TCP**

Il metodo per ottenere uno stub `get_node_manager_tcp_client` prevede questi parametri:

*   Oggetto `source_id` che identifica il mittente.
*   Oggetto `unicast_id` che identifica il destinatario.
*   Indirizzo IP del destinatario.
*   Porta TCP del destinatario.

Lato server, alla ricezione di una connessione, viene valorizzato un oggetto SampleRpc.TcpclientCallerInfo che contiene:

*   Oggetto `source_id` che identifica il mittente.
*   Oggetto `unicast_id` che identifica il destinatario.
*   Indirizzo IP del nodo corrente che è stato usato come destinazione.
*   Indirizzo IP del mittente.

**Modo Unicast**

Il metodo per ottenere uno stub `get_node_manager_unicast` prevede questi parametri:

*   Oggetto `source_id` che identifica il mittente.
*   Oggetto `unicast_id` che identifica il destinatario.
*   Nome dell'interfaccia di rete per la trasmimssione e relativo indirizzo IP *source*.
*   Porta UDP della trasmissione.
*   Booleano per indicare se si vuole attendere una risposta.

Lato server, alla ricezione di un messaggio, viene valorizzato un oggetto UnicastCallerInfo che contiene:

*   Oggetto `source_id` che identifica il mittente.
*   Oggetto `unicast_id` che identifica il destinatario.
*   Nome dell'interfaccia di rete dove il messaggio è stato rilevato.
*   Indirizzo IP del mittente.

**Modo Broadcast**

Il metodo per ottenere uno stub `get_node_manager_broadcast` prevede questi parametri:

*   Oggetto `source_id` che identifica il mittente.
*   Oggetto `broadcast_id` che identifica i destinatari.
*   Nomi delle interfacce di rete per la trasmimssione e relativi indirizzi IP *source*.
*   Porta UDP della trasmissione.
*   Istanza di IAckCommunicator (oppure NULL se non siamo interessati) per rilevare se alcuni nodi diretti vicini
    di cui siamo a conoscenza non hanno risposto con un messaggio di ACK.

Lato server, alla ricezione di un messaggio, viene valorizzato un oggetto BroadcastCallerInfo che contiene:

*   Oggetto `source_id` che identifica il mittente.
*   Oggetto `broadcast_id` che identifica i destinatari.
*   Nome dell'interfaccia di rete dove il messaggio è stato rilevato.
*   Indirizzo IP del mittente.

## <a name="APP"></a>APP

In questa sezione descriveremo a grandi linee come si realizza (di norma) una applicazione che fa uso della libreria di
livello intermedio come viene prodotta dal tool "rpcdesign".

### <a name="APP_Server"></a>Lato server

Bla bla bla.

### <a name="APP_Client"></a>Lato client

Bla bla bla.

