# ZCD - Dettagli Tecnici

1.  [ZCD](#ZCD)
    1.  [Inizializzazione](#ZCD_Inizializzazione)
    1.  [stream_net_listen](#ZCD_stream_net_listen)
    1.  [send_stream_net](#ZCD_send_stream_net)
    1.  [stream_system_listen](#ZCD_stream_system_listen)
    1.  [send_stream_system](#ZCD_send_stream_system)
    1.  [datagram_net_listen](#ZCD_datagram_net_listen)
    1.  [send_datagram_net](#ZCD_send_datagram_net)
    1.  [datagram_system_listen](#ZCD_datagram_system_listen)
    1.  [send_datagram_system](#ZCD_send_datagram_system)
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

ZCD viene "inizializzata" con la chiamata della funzione `init_tasklet_system`. Questa
fornisce l'implementazione delle interfacce usate per le comunicazioni allo schedulatore delle tasklet.

In seguito, durante la vita dell'applicazione, può essere richiesto alla libreria ZCD di mettersi
in ascolto con varie modalità con queste funzioni:

*   `stream_net_listen`.
*   `stream_system_listen`.
*   `datagram_net_listen`.
*   `datagram_system_listen`.

Ognuna di queste avvia una tasklet e restituisce un handle con il quale è possibile interromperla.  
Si tratta di una istanza dell'interfaccia `IListenerHandle`, che comprende il metodo `void kill()`.

In seguito si può richiedere a ZCD di effettuare una chiamata a metodo remoto invocando queste funzioni:

*   `send_stream_net`.
*   `send_stream_system`.
*   `send_datagram_net`.
*   `send_datagram_system`.

### <a name="ZCD_stream_net_listen"></a>stream_net_listen

La funzione `IListenerHandle stream_net_listen(...)` riceve questi argomenti:

*   `string my_ip, uint16 tcp_port`. Indicano dove ascoltare con il protocollo TCP.
*   `IStreamDelegate stream_dlg`.
*   `IErrorHandler error_handler`.

L'interfaccia `IStreamDelegate` è stata illustrata nel documento di analisi.  
Essa ha un metodo `IStreamDispatcher? get_dispatcher(StreamCallerInfo caller_info)`.
Il suo valore di ritorno, se non nullo, è un oggetto che ha un metodo
`string execute(string m_name, List<string> args, StreamCallerInfo caller_info)`.

Sul delegato `IErrorHandler error_handler`, in caso di errore, la tasklet prima di terminare potrà
chiamare il metodo `void error_handler(Error e)`.

La funzione `stream_net_listen` avvia una tasklet per gestire l'ascolto e poi ritorna al chiamante
l'handler di questo listener.  
Questo handler può essere usato per interrompere senza errori questa tasklet. Ad esempio qualora
l'indirizzo IP su cui ci si è messi in ascolto stia per venire rimosso dagli indirizzi assegnati
a questo nodo.

#### cosa fa la tasklet che gestisce il socket in ascolto

La tasklet apre un socket TCP e si mette in ascolto sulla porta e l'indirizzo specificati. Quando riceve
una connessione avvia una nuova tasklet per gestirla. E si mette di nuovo in attesa di altre connessioni.

In caso di errore nella `listen` o nella `accept` lo passa al metodo `error_handler` e
poi termina.

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

L'albero JSON di un messaggio di richiesta ha come radice un nodo *OBJECT* con 6 membri: "method-name",
"arguments", "source-id", "unicast-id", "src-nic", "wait-reply". Il membro "method-name" è un nodo *VALUE* di tipo
stringa, il membro "arguments" è un nodo *ARRAY* in cui ogni elemento è un valido nodo radice, cioè *OBJECT*
o *ARRAY*. I membri "source-id", "unicast-id", "src-nic" sono nodi *OBJECT*. Il membro "wait-reply" è un nodo *VALUE*
di tipo booleano.

L'albero JSON di un messaggio di risposta ha come radice un nodo *OBJECT* con 1 membro: "response". Il membro
"response" è un valido nodo radice, cioè *OBJECT* o *ARRAY*.

#### cosa fa la tasklet che gestisce una connessione

La tasklet che gestisce una connessione riceve questi parametri iniziali:

*   `IConnectedStreamSocket c` - Il socket connesso.
*   `IStreamDelegate stream_dlg` - Il delegato per le richieste.
*   `Listener listener` - Un oggetto che rappresenta la modalità con cui era in ascolto la tasklet
    che ha ricevuto la connessione.  
    Nel caso in esame (cioè `stream_net_listen`) esso è una `StreamNetListener listener(my_ip, tcp_port)`.
    Ma vedremo in seguito (quando analizzeremo `stream_system_listen`) che la tasklet che gestisce
    una connessione nel medium "system" è del tutto analoga a questa. Quindi questa tasklet riceve
    una generica istanza di `Listener`.

La tasklet legge i 4 bytes che indicano la lunghezza del messaggio e poi legge il numero di bytes indicato.
Se la connessione si chiude prima la tasklet termina.

Alla fine verifica di avere letto una stringa contenente un valido albero JSON. Tale JSON la libreria ZCD è
in grado di interpretarlo, quindi la tasklet ne estrae le seguenti informazioni:

*   Membro **method-name** in `string m_name`. Una stringa con il nome del metodo remoto.
*   Membro **arguments** in `List<string> args`. Un array di nodi JSON che saranno passati, in forma di altrettante singole
    stringhe, al dispatcher come argomenti del metodo. Tali stringhe devono essere ognuna un valido JSON,
    quindi i nodi JSON dell'array devono essere di tipo *OBJECT* o *ARRAY*.
*   Membro **source-id** in `string source_id`. Un nodo JSON di tipo *OBJECT*.
*   Membro **unicast-id** in `string unicast_id`. Un nodo JSON di tipo *OBJECT*.
*   Membro **src-nic** in `string src_nic`. Un nodo JSON di tipo *OBJECT*.
*   Membro **wait-reply** in `bool wait_reply`. Un booleano che dice se è prevista una risposta.

Se qualcuno di questi requisiti non è soddisfatto la tasklet chiude la connessione e termina.

Di seguito la tasklet:
*   Prepara una istanza di `StreamCallerInfo caller_info` con le informazioni:
    *   `source_id`, `src_nic`, `unicast_id`, `m_name`, `wait_reply`, `listener`.
*   Chiama il metodo `stream_dlg.get_dispatcher(caller_info)` e ottiene un `IStreamDispatcher? disp`.  
    Il delegato è in grado di stabilire se il messaggio è da processare. In
    questo caso ritorna una istanza apposita di `IStreamDispatcher`. Altrimenti ritorna `null`.
*   Se `disp == null`:
    *   Significa che si cercava una *identità* nel nodo che non è stata trovata.
    *   Il messaggio va ignorato. La tasklet chiude la connessione e termina.
    *   **TODO** Non sarebbe necessario chiudere la connessione TCP, che di per sé rimane valida. Tuttavia andrebbe
        segnalato uno specifico errore al chiamante.
*   Se il messaggio dichiarava di voler ricevere una risposta:
    *   Esegue `resp = disp.execute(m_name, args, caller_info)`.  
        L'oggetto IStreamDispatcher deserializza gli argomenti. Esegue il metodo e ottiene il risultato (o l'eccezione).
        Serializza il risultato e lo restituisce come stringa JSON.  
        Questa istanza è di una classe fornita da MOD-RPC, quindi conosce le classi degli argomenti da passare e le
        eventuali eccezioni previste dal metodo.
    *   La tasklet ora verifica che la stringa `resp` restituita da `execute` sia un valido albero JSON. Può abortire
        il programma se non lo è.
    *   Prepara un albero JSON la cui radice è un nodo *OBJECT* con il membro **response** valorizzato con il nodo
        radice di `resp`.
    *   Genera una stringa dall'albero JSON.
    *   Trasmette la stringa (dopo i 4 bytes che ne indicano la lunghezza) al socket mittente.
*   Altrimenti:
    *   Viene avviata una nuova tasklet:
        *   Esegue `disp.execute(m_name, args, caller_info)`.
*   Ascolta altre richieste.

### <a name="ZCD_send_stream_net"></a>send_stream_net

La funzione `string send_stream_net(...) throws ZCDError` riceve questi argomenti:

*   `string peer_ip, uint16 tcp_port`. Indicano dove connettersi con il protocollo TCP.
*   `string source_id`.
*   `string src_nic`.
*   `string unicast_id`.
*   `string m_name`. Il nome del metodo.
*   `List<string> arguments`. Serializzazione di una lista di argomenti da passare al metodo.
*   `bool wait_reply`.

La funzione `send_stream_net` apre una connessione con un socket verso la destinazione indicata.  
Per ridurre l'overhead di una connessione TCP, un client può stabilire una connessione TCP con un server ed
utilizzarla per diversi messaggi. Quindi di fatto la funzione internamente gestisce un pool di connessioni
già aperte verso lo stesso indirizzo e pronte per l'uso.

Se viene trovata una connessione aperta e pronta nel pool, la funzione la toglie dal pool, per accertarsi
che la tasklet su cui si trova in esecuzione sia l'unica che la sta usando.  
Di seguito, per prima cosa la funzione usa il socket connesso per scriverci. Se questa operazione va
in errore, il socket connesso viene scartato e la funzione riparte da capo. Cioè guarda di nuovo se
nel pool c'è un socket da usare, altrimenti apre una nuova connessione con un nuovo socket.

Se all'apertura della connessione con un nuovo socket si verifica un errore viene lanciata una
eccezione `ZCDError`.  
Se durante la scrittura sul socket nuovo (appena creato e connesso) si verifica un errore viene lanciata una
eccezione `ZCDError`.  
Se durante la lettura dal socket connesso (appena creato o preso dal pool, in questo caso è lo stesso perché
ormai la trasmissione della richiesta è stata fatta e eravamo in attesa della risposta) si verifica
un errore viene lanciata una eccezione `ZCDError`.  
In tutti i casi in cui viene lanciata una eccezione `ZCDError`, il socket non viene messo nel pool prima di uscire
dalla funzione.

Quando la funzione avrà completato con successo la comunicazione per cui è stata invocata, prima di terminare
metterà nel pool il socket con cui ha lavorato, il quale è pronto per l'uso con una connessione già aperta.

Una volta che una connessione è stata stabilita, la funzione costruisce un albero JSON il cui nodo radice è
un *OBJECT* con i membri:

*   Membro **method-name** da `string m_name`.
*   Membro **arguments** da `List<string> arguments`. Un array di nodi JSON. Ogni elemento della lista `arguments`
    deve essere un valido JSON: viene parsata per costruire l'albero JSON e poi il suo nodo radice viene
    aggiunto a questo array.
*   Membro **source-id** da `string source_id`. Un nodo JSON di tipo *OBJECT* che serializza l'identificativo di identità del mittente.
*   Membro **unicast-id** da `string unicast_id`. Un nodo JSON di tipo *OBJECT* che serializza l'identificativo di identità del destinatario.
*   Membro **src-nic** da `string src_nic`. Un nodo JSON di tipo *OBJECT* che serializza l'identificativo di identità del destinatario.
*   Membro **wait-reply** da `bool wait_reply`. Il booleano per indicare se si attende una risposta.

Dal JSON la funzione produce una stringa *msg*. Se durante la fase di costruzione di *msg* si verifica un
errore la libreria può abortire il programma.

La funzione trasmette la stringa *msg* alla connessione, preceduta da 4 bytes che ne indicano la lunghezza.
Il comportamento in caso di errore qui è stato già descritto.

Se non si vuole attendere una risposta, il metodo restituisce immediatamente una stringa vuota.  
Come abbiamo detto, prima di terminare con successo (restituendo la stringa vuota) il socket che stava usando viene messo nel pool.

Se bisogna attendere una risposta, la funzione la attende dalla stessa connessione, leggendola come sempre:
prima 4 bytes e poi il numero di bytes riportati. Il comportamento in caso di errore qui è stato già descritto.

La stringa ricevuta deve essere un valido albero JSON. Altrimenti viene lanciata una eccezione `ZCDError`.

Il nodo radice deve essere un *OBJECT* con il membro **response** che è un nodo valido come radice. Altrimenti viene
lanciata una eccezione `ZCDError`.

La funzione costruisce un nuovo albero con tale nodo e genera una stringa da esso. Restituisce la stringa.  
Come abbiamo detto, prima di terminare con successo (restituendo la stringa) il socket che stava usando viene messo nel pool.

### <a name="ZCD_stream_system_listen"></a>stream_system_listen

La funzione `IListenerHandle stream_system_listen(...)` riceve questi argomenti:

*   `string listen_pathname`. Indica dove ascoltare con un socket unix-domain per connessioni.
*   `IStreamDelegate stream_dlg`.
*   `IErrorHandler error_handler`.

L'interfaccia `IStreamDelegate` è stata illustrata in precedenza.

La classe che implementa `IStreamDelegate` può essere la stessa che si usa per la funzione
`stream_net_listen`. Anche l'istanza può essere unica. L'importante è che questa abbia conoscenza delle
classi `StreamNetListener` e `StreamSystemListener`.

Sul delegato `IErrorHandler error_handler`, in caso di errore, la tasklet prima di terminare potrà
chiamare il metodo `void error_handler(Error e)`.

La funzione `stream_system_listen` avvia una tasklet per gestire l'ascolto e poi ritorna al chiamante
l'handler di questo listener.  
Questo handler può essere usato per interrompere senza errori questa tasklet. Ad esempio qualora
il pathname su cui ci si è messi in ascolto stia per venire rimosso dai pathname assegnati
a questo processo.  
In questo caso, prima di terminare dovrebbe rimuovere il pathname.

#### cosa fa la tasklet che gestisce il socket in ascolto

La tasklet dovrebbe trovare che il pathname specificato non esiste ancora, quindi crearlo. Altrimenti abortisce il programma.

La tasklet apre un socket unix-domain e si mette in ascolto sul pathname specificato. Quando riceve
una connessione avvia una nuova tasklet per gestirla. E si mette di nuovo in attesa di altre connessioni.

In caso di errore nella `listen` o nella `accept` lo passa al metodo `error_handler` e
poi termina.

Prima di terminare con errore, dovrebbe rimuovere il pathname.

#### cosa fa la tasklet che gestisce una connessione

I messaggi che transitano su una connessione nel medium "system" sono del tutto analoghi a quelli che transitano
su una connessione nel medium "net".

La tasklet che gestisce una connessione riceve questi parametri iniziali:

*   `IConnectedStreamSocket c` - Il socket connesso.
*   `IStreamDelegate stream_dlg` - Il delegato per le richieste.
*   `Listener listener` - Un oggetto che rappresenta la modalità con cui era in ascolto la tasklet
    che ha ricevuto la connessione.  
    Questa volta sarà una `StreamSystemListener listener(listen_pathname)`.

Il codice eseguito dalla tasklet è lo stesso che esegue la tasklet che gestisce le connessioni nel medium "net".

### <a name="ZCD_send_stream_system"></a>send_stream_system

La funzione `string send_stream_system(...) throws ZCDError` riceve questi argomenti:

*   `string send_pathname`. Indica dove connettersi con un socket unix-domain per connessioni.
*   `string source_id`.
*   `string src_nic`.
*   `string unicast_id`.
*   `string m_name`. Il nome del metodo.
*   `List<string> arguments`. Serializzazione di una lista di argomenti da passare al metodo.
*   `bool wait_reply`.

La funzione deve verificare che il pathname specificato esista. Altrimenti lancia una
eccezione `ZCDError`.  
Sarebbe infatti la situazione analoga a quando si prova a connettersi ad un indirizzo IP e si rileva
che non esiste o non è raggiungibile.

La funzione `send_stream_system` apre una connessione con un socket verso la destinazione indicata.  
Non è necessario tenere conto dell'overhead di una connessione e gestire un pool di connessioni aperte.

Il comportamento è del tutto similare a quello visto nella funzione `send_stream_net`. Probabilmente ci
sarà una funzione interna che farà questo lavoro in entrambi i casi.

### <a name="ZCD_datagram_net_listen"></a>datagram_net_listen

La funzione `IListenerHandle datagram_net_listen(...)` riceve questi argomenti:

*   `string my_dev, uint16 udp_port, string ack_mac`. Indicano dove ascoltare con il protocollo UDP
    e l'identificativo (MAC addess) dell'interfaccia di rete propria del nodo.
*   `IDatagramDelegate datagram_dlg`.
*   `IErrorHandler error_handler`.

L'interfaccia `IDatagramDelegate` è stata illustrata nel documento di analisi.  
Essa ha il metodo `bool is_my_own_message(int packet_id)`.  
Inoltre ha il metodo `IDatagramDispatcher? get_dispatcher(DatagramCallerInfo caller_info)`.
Il suo valore di ritorno, se non nullo, è un oggetto che ha un metodo
`void execute(string m_name, List<string> args, DatagramCallerInfo caller_info)`.  
Infine ha il metodo `void got_ack(int packet_id, string ack_mac)`.

Sul delegato `IErrorHandler error_handler`, in caso di errore, la tasklet prima di terminare potrà
chiamare il metodo `void error_handler(Error e)`.

La funzione `datagram_net_listen` avvia una tasklet per gestire l'ascolto e poi ritorna al chiamante
l'handler di questo listener.  
Questo handler può essere usato per interrompere senza errori questa tasklet. Ad esempio qualora
l'interfaccia di rete su cui ci si è messi in ascolto stia per venire rimossa dalle interfacce
gestite.

#### cosa fa la tasklet che gestisce il socket in ascolto

La tasklet istanzia un socket UDP per la ricezione di messaggi broadcast
(un TaskletSystem.IServerDatagramSocket) associato all'interfaccia di rete e alla porta specificate. Nel caso di
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
grosse dimensioni sia impossibile da trasmettere correttamente con la modalità "datagram" del framework.

Esistono diversi tipi di messaggio. Ogni messaggio è un albero JSON che ha come nodo radice un oggetto con
un membro. Dal nome del membro si deduce il tipo e di conseguenza come trattare il suo contenuto, che
comunque è sempre un oggetto. Di seguito elenchiamo le tipologie, indicando il nome del membro sull'oggetto
radice. Sotto ogni tipologia sono poi elencati i vari membri dell'oggetto.

*   Messaggio di *richiesta*.  
    Il membro ha nome "request" e il suo contenuto è un oggetto che la libreria ZCD è
    in grado di interpretare. Quindi la tasklet ne estrae le seguenti informazioni:

    *   Membro **packet-id** in `int packet_id`. Un intero che identifica questo messaggio di *richiesta*.
    *   Membro **method-name** in `string m_name`. Una stringa con il significato descritto in precedenza.
    *   Membro **arguments** in `List<string> args`. Un array di nodi JSON con il significato descritto in precedenza.
    *   Membro **source-id** in `string source_id`. Un nodo JSON di tipo *OBJECT*.
    *   Membro **broadcast-id** in `string broadcast_id`. Un nodo JSON di tipo *OBJECT*.
    *   Membro **src-nic** in `string src_nic`. Un nodo JSON di tipo *OBJECT*.
    *   Membro **send-ack** in `bool send_ack`. Un booleano che dice se è richiesto un messaggio di *ACK*.

*   Messaggio di *ACK* relativo ad una specifica richiesta correttamente ricevuta.  
    Il membro ha nome "ack" e il suo contenuto è un oggetto che la libreria ZCD è
    in grado di interpretare. Quindi la tasklet ne estrae le seguenti informazioni:

    *   Membro **packet-id** in `int packet_id`. Un intero che identifica il messaggio di *richiesta* per il quale
        questo messaggio è il relativo *ACK*.
    *   Membro **ack-mac** in `string ack_mac`. L'identificativo (MAC addess) dell'interfaccia di rete
        che ha ricevuto il messaggio di *richiesta*.

#### cosa fa la tasklet che gestisce un datagram

La tasklet che gestisce un datagram riceve questi parametri iniziali:

*   `string msg` - Il messaggio ricevuto.
*   `IDatagramDelegate datagram_dlg` - Il delegato per le richieste.
*   `Listener listener` - Un oggetto che rappresenta la modalità con cui era in ascolto la tasklet
    che ha ricevuto il messaggio.  
    Nel caso in esame (cioè `datagram_net_listen`) esso è una `DatagramNetListener listener(my_dev, udp_port, ack_mac)`.
    Ma vedremo in seguito (quando analizzeremo `datagram_system_listen`) che la tasklet che gestisce
    un messaggio nel medium "system" è del tutto analoga a questa. Quindi questa tasklet riceve
    una generica istanza di `Listener`.

La tasklet prima di tutto verifica la correttezza del messaggio, che deve essere un valido albero JSON,
altrimenti termina. Analogamente, se il nodo radice non è di tipo oggetto la tasklet termina.

L'oggetto radice deve avere esattamente un membro. Il nome del membro deve essere uno di quelli indicati
precedentemente. Altrimenti la tasklet termina.

Se si tratta di un messaggio di *richiesta*:

*   La tasklet verifica di poter estrapolare dall'albero JSON le informazioni sopra illustrate, pena la sua terminazione.
*   La tasklet chiama il metodo `datagram_dlg.is_my_own_message(packet_id)`.  
    Se la risposta è True significa che il messaggio è partito dal mio nodo, quindi la tasklet lo ignora e termina.
*   Se il messaggio dichiarava di voler ricevere un acknowledge, cioè `send_ack`:
    *   Prepara l'albero JSON contenente il messaggio di *ACK* con il `packet_id` del messaggio ricevuto e
        il `ack_mac` che trova nel `listener` che ha ricevuto come argomento.
    *   In una nuova tasklet, per 3 volte a brevi intervalli casuali tra 10 e 200 msec, trasmette il messaggio
        di *ACK* in un datagram sulla stessa interfaccia su cui ha ricevuto il messaggio. Questo lo fa chiamando
        una sua funzione interna `send_ack_net(my_dev, udp_port, ...)` o `send_ack_system(send_pathname, ...)`
        a seconda della tasklet in ascolto che ha ricevuto il datagram: cioè a seconda del `Listener listener`.
*   Prepara una istanza di `DatagramCallerInfo caller_info` con le informazioni:
    *   `packet_id`, `source_id`, `src_nic`, `broadcast_id`, `m_name`, `send_ack`, `listener`.
*   Chiama il metodo `datagram_dlg.get_dispatcher(caller_info)` e ottiene un `IDatagramDispatcher? disp`.  
    Il delegato è in grado di stabilire se il messaggio è da processare. In
    questo caso ritorna una istanza apposita di `IDatagramDispatcher`. Altrimenti ritorna `null`.
*   Se `disp == null`:
    *   Il messaggio va ignorato. La tasklet termina.
*   Esegue `disp.execute(m_name, args, caller_info)`.  
    Come già discusso, se il messaggio aveva come destinatari delle *identità* piuttosto che dei *nodi*,
    il dispatcher potrebbe dover avviare diverse tasklet e su esse eseguire il metodo sullo *skeleton* di
    ogni *identità* interessata.
*   Poi la tasklet termina.

Se si tratta di un messaggio di *ACK*:

*   La tasklet verifica di poter estrapolare dall'albero JSON le informazioni sopra illustrate, pena la sua terminazione.
*   La tasklet chiama il metodo `datagram_dlg.got_ack(packet_id, ack_mac)` con il `packet_id` e
    il `ack_mac` del messaggio ricevuto.

### <a name="ZCD_send_datagram_net"></a>send_datagram_net

La funzione `void send_datagram_net(...) throws ZCDError` riceve questi argomenti:

*   `string my_dev, uint16 udp_port`. Indicano dove trasmettere con il protocollo UDP.
*   `int packet_id`.
*   `string source_id`.
*   `string src_nic`.
*   `string broadcast_id`.
*   `string m_name`. Il nome del metodo.
*   `List<string> arguments`. Serializzazione di una lista di argomenti da passare al metodo.
*   `bool send_ack`.

L'utilizzatore di ZCD dovrà aver cura di creare un `packet_id` univoco per ogni messaggio datagram che vuole
trasmettere e fare in modo di farlo conoscere all'istanza di `IDatagramDelegate` che aveva passata alla
funzione `datagram_net_listen` relativa a `my_dev`. Tale istanza infatti sarà invocata alla
ricezione dei messaggi di *ACK* relativi.

La funzione costruisce un albero JSON il cui nodo radice è un *OBJECT* con i membri:

*   Membro **packet-id** da `int packet_id`.
*   Membro **method-name** da `string m_name`.
*   Membro **arguments** da `List<string> arguments`. Un array di nodi JSON. Ogni elemento della lista `arguments`
    deve essere un valido JSON: viene parsata per costruire l'albero JSON e poi il suo nodo radice viene
    aggiunto a questo array.
*   Membro **source-id** da `string source_id`. Un nodo JSON di tipo *OBJECT* che serializza l'identificativo di identità del mittente.
*   Membro **broadcast-id** da `string broadcast_id`. Un nodo JSON di tipo *OBJECT* che serializza l'identificativo di identità del destinatario.
*   Membro **src-nic** da `string src_nic`. Un nodo JSON di tipo *OBJECT* che serializza l'identificativo di identità del destinatario.
*   Membro **send-ack** da `bool send_ack`. Il booleano per indicare se si richiede un messaggio di *ACK*.

Dal JSON la funzione produce una stringa *msg*. Se durante la fase di costruzione di *msg* si verifica un
errore la libreria può abortire il programma.

La funzione `send_datagram_net` crea un socket per trasmettere un pacchetto broadcast UDP sull'interfaccia
di rete indicata. Vi trasmette la stringa *msg*. Poi termina.

Se nelle operazioni di creazione socket e trasmissione si verifica un errore viene lanciata una eccezione `ZCDError`.

### <a name="ZCD_datagram_system_listen"></a>datagram_system_listen

La funzione `IListenerHandle datagram_system_listen(...)` riceve questi argomenti:

*   `string listen_pathname, string send_pathname, string ack_mac`. Indicano il pathname dove ascoltare
    i datagram (trasmessi dai nodi vicini) e il pathname dove trasmettere i datagram di *ACK*
    e l'identificativo (MAC addess) dell'interfaccia di rete propria del nodo.
*   `IDatagramDelegate datagram_dlg`.
*   `IErrorHandler error_handler`.

L'interfaccia `IDatagramDelegate` è stata illustrata in precedenza.

La classe che implementa `IDatagramDelegate` può essere la stessa che si usa per la funzione
`datagram_net_listen`. Anche l'istanza può essere unica. L'importante è che questa abbia conoscenza delle
classi `DatagramNetListener` e `DatagramSystemListener`. Questo è vero in particolare perché nella ricezione
di un datagram di *richiesta* che richiede un datagram di *ACK* il delegato deve sapere se vada
chiamata la funzione interna `send_ack_net(my_dev, udp_port, ...)` o `send_ack_system(send_pathname, ...)`.

Sul delegato `IErrorHandler error_handler`, in caso di errore, la tasklet prima di terminare potrà
chiamare il metodo `void error_handler(Error e)`.

La funzione `datagram_system_listen` avvia una tasklet per gestire l'ascolto e poi ritorna al chiamante
l'handler di questo listener.  
Questo handler può essere usato per interrompere senza errori questa tasklet. Ad esempio qualora
il pathname su cui ci si è messi in ascolto stia per venire rimosso dai pathname assegnati
a questo processo.  
In questo caso, prima di terminare dovrebbe rimuovere il pathname.

#### cosa fa la tasklet che gestisce il socket in ascolto

La tasklet dovrebbe trovare che il pathname specificato non esiste ancora, quindi crearlo. Altrimenti abortisce il programma.

La tasklet apre un socket unix-domain e si mette in ascolto sul pathname specificato. Nel caso di
errore nella creazione/configurazione del socket, lo passa al metodo `error_handler` e poi termina.

Prima di terminare con errore, dovrebbe rimuovere il pathname.

La tasklet mette il socket in attesa di un pacchetto con la chiamata `recvfrom`.

La chiamata `recvfrom` può ritornare con successo, fornendo il messaggio ricevuto, oppure segnalare un errore.

Nel caso di errore la tasklet produce un log e torna di seguito ad ascoltare (dopo un delay) con le stesse modalità.

Nel caso di ricezione di un messaggio la tasklet avvia una nuova tasklet per gestire il messaggio e poi torna ad ascoltare.

#### cosa fa la tasklet che gestisce un datagram

I messaggi datagram trasmessi nel medium "system" sono del tutto analoghi a quelli che transitano
su un dominio broadcast nel medium "net".

La tasklet che gestisce un datagram riceve questi parametri iniziali:

*   `string msg` - Il messaggio ricevuto.
*   `IDatagramDelegate datagram_dlg` - Il delegato per le richieste.
*   `Listener listener` - Un oggetto che rappresenta la modalità con cui era in ascolto la tasklet
    che ha ricevuto il messaggio.  
    Questa volta sarà un `DatagramSystemListener listener(listen_pathname, send_pathname, ack_mac)`.

Il codice eseguito dalla tasklet è lo stesso che esegue la tasklet che gestisce i datagram nel medium "net".

### <a name="ZCD_send_datagram_system"></a>send_datagram_system

La funzione `void send_datagram_system(...) throws ZCDError` riceve questi argomenti:

*   `string send_pathname`. Indica dove trasmettere con un socket unix-domain per pacchetti datagram.
*   `int packet_id`.
*   `string source_id`.
*   `string src_nic`.
*   `string broadcast_id`.
*   `string m_name`. Il nome del metodo.
*   `List<string> arguments`. Serializzazione di una lista di argomenti da passare al metodo.
*   `bool send_ack`.

La funzione deve verificare che il pathname specificato esista. Altrimenti ignora il messaggio e
ritorna al chiamante.  
Sarebbe infatti la situazione analoga a quando si scrive su una interfaccia di rete ma non viene
raggiunto nessuno. In pratica, rimandando alle modalità descritte nel documento ntkd-RPC nella
sezione [Tipi di medium](../DemoneNTKD/RPC.md#Medium) per mappare in un pathname le situazioni di
comunicazione con messaggi broadcast, sarebbe dovuto al fatto che non è stato avviato il relativo
processo "domain" o "radio-domain".

La funzione `send_datagram_system` crea un socket unix-domain per trasmettere pacchetti datagram
verso il pathname specificato.

Il comportamento è del tutto similare a quello visto nella funzione `send_datagram_net`. Probabilmente ci
sarà una funzione interna che farà questo lavoro in entrambi i casi.

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
quelli usati come `source_id`, `src_nic`, `unicast_id` e `broadcast_id`.

La libreria fornisce le interfacce `ISourceID`, `ISrcNic`, `IUnicastID`, `IBroadcastID`. Inoltre, per ogni nome di tipo indicato
nella descrizione dei metodi remoti come argomento o come valore di ritorno, oltre ai tipi base, fornisce o una
classe o una interfaccia (se il nome inizia con una 'I' maiuscola seguita da un'altra lettera maiuscola). Se si
tratta di una classe, il tool "rpcdesign" la prepara vuota. Sarà lo sviluppatore a completarla con i dati che
servono e ad accertarsi che possa essere serializzata e deserializzata con `Json.gobject_serialize` e
`Json.gobject_deserialize` di JsonGlib.

Se si tratta di una interfaccia (come per le interfacce `ISourceID`, `ISrcNic`, `IUnicastID`, `IBroadcastID`) il tool "rpcdesign"
la prepara vuota. Il programmatore potrà anche optare per lasciare la libreria inalterata, come preparata dal
tool "rpcdesign", e fornire la classe che la implementa direttamente nell'applicazione che usa la libreria di
livello intermedio. Anche in questo caso, in tale applicazione, dovrà accertarsi che possa essere serializzata
e deserializzata con `Json.gobject_serialize` e `Json.gobject_deserialize` di JsonGlib.

### <a name="MODRPC_server"></a>Lato server

La libreria fornisce una interfaccia skeleton per la classe radice e una per ogni modulo membro.

La libreria fornisce inoltre una interfaccia per il delegato che essa stessa userà per reperire le istanze della
classe radice. Infine fornisce una interfaccia per il gestore di errori; questo sarà invocato nelle tasklet che
stanno in ascolto sui socket in caso di errori che causano la terminazione delle stesse tasklet.

L'applicazione APP dovrà fornire una implementazione per ognuna di
queste interfacce. L'istanza del delegato e quella del gestore di errori andranno passate alla libreria nella
fase di inizializzazione, cioè, come vedremo dopo, quando si avviano le tasklet che stanno in ascolto sui socket.

La libreria, attraverso il delegato, sarà in grado, quando riceve una richiesta, di ottenere le istanze di classi
radice e classi modulo al fine di chiamare un metodo remoto.

Facciamo un esempio di RPC-IDL. Il file interfaces.rpcidl presenta la classe radice `NodeManager node_manager`. Ha un
membro `InfoManager info_manager`. Questo ha il metodo `string get_name() throws AccessDeniedError`. La libreria
prodotta da "rpcdesign" fornirà queste interfacce:

*   `SampleRpc.IInfoManagerSkeleton`.  
    Prevede il metodo astratto:
    *   `string get_name(SampleRpc.CallerInfo? caller=null) throws AccessDeniedError`
*   `SampleRpc.INodeManagerSkeleton`.  
    Prevede il metodo astratto:
    *   `SampleRpc.IInfoManagerSkeleton info_manager_getter()`
*   `SampleRpc.IRpcDelegate`.  
    Prevede il metodo astratto:
    *   `Gee.List<SampleRpc.INodeManagerSkeleton> get_node_manager_set(SampleRpc.CallerInfo caller)`
*   `SampleRpc.IRpcErrorHandler`.  
    Prevede il metodo astratto:
    *   `void error_handler(Error e)`

**Nota 1**: Tutti i metodi remoti nell'interfaccia skeleton ricevono un argomento aggiuntivo `SampleRpc.CallerInfo`
che contiene alcune informazioni sul richiedente. Il contenuto varia a seconda del metodo usato per inviare il
messaggio (stream/datagram, net/system).

**Nota 2**: L'interfaccia `IRpcDelegate` ha un metodo per ogni root-dispatcher. Per ogni richiesta che si riceve
lato server la prima parte del nome del metodo remoto indica il root-dispatcher da usare; la libreria MOD-RPC
prende questo prefisso e sceglie il metodo di `IRpcDelegate` da chiamare.

La libreria MOD-RPC fornisce inoltre alcune chiamate di inizializzazione:

*   `void SampleRpc.init_tasklet_system(ITasklet _tasklet)`
*   `void SampleRpc.stream_net_listen(SampleRpc.IRpcDelegate dlg, SampleRpc.IRpcErrorHandler err,`  
    `string my_ip, uint16 tcp_port)`
*   `void SampleRpc.stream_system_listen(SampleRpc.IRpcDelegate dlg, SampleRpc.IRpcErrorHandler err,`  
    `string listen_pathname)`
*   `void SampleRpc.datagram_net_listen(SampleRpc.IRpcDelegate dlg, SampleRpc.IRpcErrorHandler err,`  
    `string my_dev, uint16 udp_port, string ack_mac)`
*   `void SampleRpc.datagram_system_listen(SampleRpc.IRpcDelegate dlg, SampleRpc.IRpcErrorHandler err,`  
    `string listen_pathname, string send_pathname, string ack_mac)`

La funzione `init_tasklet_system` deve essere chiamata da APP come inizializzazione della libreria. Essa serve
a passare l'implementazione del sistema di tasklet, richiesto dalla libreria ZCD.

Per ogni indirizzo IP proprio del nodo in cui è eseguita, la APP chiama una volta la funzione `stream_net_listen`.  
Per ogni interfaccia di rete propria del nodo in cui è eseguita, la APP chiama una volta la funzione `datagram_net_listen`.  
Con modalità analoghe, nei processi che all'interno di un sistema (in una testsuite) simulano ciascuno un nodo, sono
usate le funzioni `stream_system_listen` e `datagram_system_listen`.  
Ognuna di queste funzioni avvia una nuova tasklet perché stia in ascolto di messaggi.

In realtà ognuna di queste funzioni chiama semplicemente la funzione omonima nella libreria di basso livello.
È la libreria di basso livello che avvia una tasklet in ascolto di messaggi. Quello che fa la libreria MOD-RPC in queste
funzioni è costruire un delegato (`zcd.IStreamDelegate` o `zcd.IDatagramDelegate`) adatto alla funzione della libreria
di basso livello sulla base del delegato (`SampleRpc.IRpcDelegate`) che riceve dalla APP.  
Quando la libreria di basso livello (nella tasklet in ascolto, o meglio in gestione di una precisa connessione
o datagram) riceve un messaggio, produce un `zcd.CallerInfo` e lo passa al delegato `zcd.IStreamDelegate`
o `zcd.IDatagramDelegate` prodotto da MOD-RPC. Questo dai dati del `zcd.CallerInfo` produce un `SampleRpc.CallerInfo`
e lo passa al delegato `SampleRpc.IRpcDelegate` ricevuto dalla APP.

Ricordiamo cosa intendiamo dire con "lo passa al delegato".  
Nel caso di un `zcd.IStreamDelegate` significa che chiama il suo metodo
`IStreamDispatcher? get_dispatcher(StreamCallerInfo caller_info)`
con questo argomento. Se il risultato che ottiene non è nullo ci chiamerà in seguito il metodo
`string execute(string m_name, List<string> args, StreamCallerInfo caller_info)`.  
Nel caso di un `zcd.IDatagramDelegate` significa che chiama il suo metodo
`IDatagramDispatcher? get_dispatcher(DatagramCallerInfo caller_info)`
con questo argomento. Se il risultato che ottiene non è nullo ci chiamerà in seguito il metodo
`void execute(string m_name, List<string> args, DatagramCallerInfo caller_info)`.  
Nel caso di un `SampleRpc.IRpcDelegate` significa che chiama il suo metodo
`Gee.List<SampleRpc.INodeManagerSkeleton> get_node_manager_set(SampleRpc.CallerInfo caller)`
(`node_manager` fa parte del nostro esempio di RPC-IDL) con questo argomento. Se il risultato che ottiene
non è vuoto, su ognuno degli skeleton chiamerà in seguito il metodo specificato in `m_name`.

La classe `SampleRpc.CallerInfo` è analoga alla classe `CallerInfo` usata nella libreria ZCD di basso livello.

La classe CallerInfo è vuota. Viene ereditata dalle classi StreamCallerInfo e DatagramCallerInfo.

Se la tasklet che ha ricevuto il messaggio era in ascolto per connessioni, cioè `stream_net_listen` o
`stream_system_listen`, viene prodotta una istanza di StreamCallerInfo. Questa contiene:

*   `ISourceID source_id`. 
*   `ISrcNic src_nic`
*   `IUnicastID unicast_id`
*   `string m_name`
*   `bool wait_reply`
*   `Listener listener`

Se invece la tasklet che ha ricevuto il messaggio era in ascolto per messaggi broadcast, cioè `datagram_net_listen` 
o `datagram_system_listen`, viene prodotta una istanza di DatagramCallerInfo. Questa contiene:

*   `int packet_id`.
*   `ISourceID source_id`. 
*   `ISrcNic src_nic`
*   `IBroadcastID broadcast_id`
*   `string m_name`
*   `bool send_ack`
*   `Listener listener`

La classe Listener è vuota. Viene ereditata dalle seguenti classi:

*   StreamNetListener. Prodotta da `stream_net_listen`. Contiene i suoi parametri `my_ip, tcp_port`.
*   StreamSystemListener. Prodotta da `stream_system_listen`. Contiene i suoi parametri `listen_pathname`.
*   DatagramNetListener. Prodotta da `datagram_net_listen`. Contiene i suoi parametri `my_dev, udp_port, ack_mac`.
*   DatagramSystemListener. Prodotta da `datagram_system_listen`. Contiene i suoi parametri `listen_pathname, send_pathname, ack_mac`.

Il delegato interrogato dalle tasklet alla ricezione di un messaggio, diversamente da quanto avveniva
nella libreria di basso livello, è uno solo. Una stessa istanza di `SampleRpc.IRpcDelegate dlg` può occuparsi
sia dei messaggi di tipo "stream" sia di quelli di tipo "datagram".

Proseguiamo considerando il nostro esempio di RPC-IDL. Assumiamo il caso di un messaggio ricevuto con la modalità
"stream"; sarà banale comprendere come differiscono solo leggermente le operazioni nel caso di messaggi ricevuti
con la modalità "datagram".  
La tasklet avviata dalla libreria di basso livello per la gestione delle connessioni ha ricevuto una connessione
e ha avviato una nuova tasklet per gestirla. Quest'ultima ha letto dal socket connesso un messaggio e si
appresta a gestirlo.  
Essa chiama `IStreamDispatcher? get_dispatcher(StreamCallerInfo caller_info)` sul `IStreamDelegate stream_dlg`
fornito da MOD-RPC.  
Questo per prima cosa vede che la parte iniziale di `m_name` è `node_manager`.
Allora costruisce un analoga istanza di `SampleRpc.CallerInfo caller` e poi
chiama `Gee.List<INodeManagerSkeleton> get_node_manager_set(caller)` sul `SampleRpc.IRpcDelegate dlg`
fornito da APP.  
Quest'ultima conosce le diverse classi `SampleRpc.CallerInfo` descritte sopra. In base al `caller` ricevuto
(ad esempio guardando `unicast_id` o `broadcast_id` o altro)
decide se restituire zero, una o più istanze dell'interfaccia skeleton della classe radice.  
Ora MOD-RPC (sempre nel codice che implementa `IStreamDelegate.get_dispatcher` nel nostro esempio)
sulla base di questa lista di istanze skeleton, se non è vuota, prepara una istanza di `IStreamDispatcher`
che restituisce alla libreria di basso livello. Altrimenti restituisce `null`.  
La libreria di basso livello, se ottiene un dispatcher non nullo, chiama il suo metodo `execute(m_name, args, caller_info)`
dal quale si attenderà una stringa con la risposta serializzata. Se questa vada trasmessa o meno al mittente
del messaggio se ne occuperà la libreria di basso livello.  
Il metodo `execute`, nel `IStreamDispatcher` fornito da MOD-RPC, esamina il nome del metodo, individua il percorso da
fare (es: `root.info_manager.get_name(...)`).  
Poi deserializza gli argomenti.  
Se durante la deserializzazione degli argomenti si riscontra un problema (ad esempio non sono del tipo previsto
dalla signature del metodo, oppure un oggetto è di una classe sconosciuta al programma, oppure un oggetto viene
deserializzato ma non contiene tutte le proprietà requisito della sua classe) il metodo `execute`
serializza e restituisce una eccezione DeserializeError.  
Poi, per ogni istanza skeleton che era nella lista, chiama il metodo. Siccome nel nostro esempio il messaggio era
in modalità "stream", la lista aveva un solo skeleton. Quindi il metodo `execute` ottenuto l'esito del metodo
chiamato (il valore di ritorno o una eccezione) lo serializza e restituisce nella forma di una stringa.

Le classi fornite da APP per implementare le interfacce skeleton, in particolare quelle dei singoli moduli, avranno
quindi solo il compito di implementare la business logic dei singoli metodi remoti come fossero comuni metodi.
Riceveranno i parametri previsti dalla signature, potranno elaborarli e restituire un valore di ritorno del tipo
previsto dalla signature oppure lanciare una delle eccezioni previste dalla signature.

### <a name="MODRPC_client"></a>Lato client

La libreria fornisce una
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

