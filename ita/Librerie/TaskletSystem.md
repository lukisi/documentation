# Tasklet System

1.  [Tasklet](#Tasklet)
    1.  [Interfacce](#Interfacce)
    1.  [Tasklet sequenziali](#sequenziali)
    1.  [Inizializzazione](#Inizializzazione)

## <a name="Tasklet"></a>Tasklet

Quasi tutti i moduli che compongono il demone *ntkd* fanno uso di *tasklet* per svolgere i loro compiti. Si
tratta di thread cooperativi, cioè che permettono allo schedulatore di passare l'esecuzione ad un altro thread
soltanto su specifica autorizzazione da parte del thread corrente.

**Nota**: Questa tipologia di thread è chiamata anche [green thread](https://en.wikipedia.org/wiki/Green_threads).
Il termine tasklet è preso dalla terminologia usata in Stackless Python.

L'autorizzazione a schedulare altre tasklet viene data da parte della tasklet corrente con una delle seguenti modalità:

*   La tasklet corrente dichiara esplicitamente di voler passare il controllo.
*   La tasklet corrente chiama una delle seguenti funzioni bloccanti:
    *   Attesa di un certo tempo.
    *   Lettura del contenuto di un file.
    *   Scrittura di un file.
    *   Attesa di una comunicazione dalla rete.
    *   Invio di una comunicazione nella rete.
    *   Esecuzione di un comando e attesa dell'esito.

Inoltre la tasklet corrente può creare altre tasklet, attraverso la funzione *spawn*. Questo consiste nel dire
allo schedulatore che una certa chiamata ad una funzione dovrà essere eseguita in una nuova tasklet. La nuova
tasklet non verrà comunque iniziata dallo schedulatore fino a quando la tasklet corrente non lo consentirà attraverso
una delle modalità dette in precedenza.

La funzione *spawn* restituisce un *handle* per la tasklet creata. Tramite questo handle è possibile verificare
se la tasklet è ancora in esecuzione. Si può inoltre richiedere la terminazione della tasklet.

Alla funzione *spawn* si può indicare se la nuova tasklet dovrà essere *joinable*. In questo caso tramite il suo
handle sarà possibile chiamare il metodo `join`: la tasklet corrente si blocca in attesa che la tasklet
individuata dall'handle termini e poi riceve il valore di ritorno dell'esecuzione della tasklet; si tratta di
un `void *` che di norma è il valore di ritorno della funzione con cui la tasklet ha avuto inizio.

La tasklet corrente può richiedere di terminare se stessa, con un eventuale valore di ritorno, indipendentemente
dalla profondità di chiamate nel suo stack che ha raggiunto.

Due tasklet possono comunicare tra loro attraverso un *canale*. Se due tasklet condividono un canale questo
fornisce loro un meccanismo per comunicare tra loro senza bloccarsi.

E' stata realizzata una libreria che contiene alcune interfacce di programmazione verso un generico sistema di
tasklet. Si chiama *tasklet-system*.

Questa libreria sarà una dipendenza nel pacchetto software di ogni singolo modulo. Ma tali pacchetti non saranno
tenuti ad avere una dipendenza sulla specifica libreria che implementa il sistema di tasklet.

La libreria *tasklet-system* sarà una dipendenza anche nel pacchetto software che raccoglie i vari moduli e produce
l'eseguibile del demone *ntkd*. Il compito di tale pacchetto sarà quello di fornire ai vari moduli una
implementazione delle interfacce del sistema di tasklet, quindi avrà una dipendenza anche sulla specifica libreria
che usa a tale scopo.

Si cerca così di rendere i vari moduli del demone *ntkd* indipendenti dalla implementazione delle tasklet. La
prima implementazione che useremo nel demone sarà la libreria PthTasklet basata su GNU/Pth, ma dovrebbe essere
possibile usare altre implementazioni.

### <a name="Interfacce"></a>Interfacce

L'interfaccia ITasklet prevede questi metodi:

*   `void schedule()`  
    Passa il controllo ad altre tasklet.
*   `void ms_wait(int msec)`  
    Blocca la tasklet corrente per il tempo indicato, ma consente alle altre di proseguire.
*   `[NoReturn] void exit_tasklet(void * ret)`  
    Termina la tasklet corrente.
*   `ITaskletHandle spawn(ITaskletSpawnable sp, bool joinable=false)`  
    Crea una nuova tasklet, come descritto sotto.
*   `TaskletCommandResult exec_command(string cmdline)`  
    Avvia un nuovo processo e attende il suo esito. E' bloccante per la tasklet corrente, ma consente alle altre di proseguire.
*   `size_t read(int fd, void* b, size_t maxlen) throws Error`  
    Dato il file-descriptor di un file aperto, bloccando solo la tasklet corrente, legge dal file.  
    Il valore restituito dalla funzione sarà sempre positivo. In caso di errore la funzione lancia un apposito Error.
*   `size_t write(int fd, void* b, size_t count) throws Error`  
    Dato il file-descriptor di un file aperto, bloccando solo la tasklet corrente, scrive sul file.  
    Il valore restituito dalla funzione sarà sempre positivo. In caso di errore la funzione lancia un apposito Error.
*   `IServerStreamSocket get_server_stream_socket(uint16 port) throws Error`  
    Crea un socket e lo mette in ascolto su una porta TCP. La chiamata non è bloccante, ma le successive operazioni
    lo saranno; si veda sotto i metodi previsti dall'interfaccia IServerStreamSocket. Ottenere il socket tramite questo
    meccanismo consentirà in seguito di effettuare le altre operazioni bloccando solo la tasklet corrente e consentendo
    alle altre di proseguire.
*   `IConnectedStreamSocket get_client_stream_socket(string dest_addr, uint16 dest_port, string? my_addr=null) throws Error`  
    Crea un socket e lo connette ad un server TCP. E' bloccante per la tasklet corrente, ma consente alle altre di
    proseguire. Anche le successive operazioni sul socket saranno bloccanti; si veda sotto i metodi previsti
    dall'interfaccia IConnectedStreamSocket. Il meccanismo fornito consentirà di bloccare solo la tasklet corrente.
*   `IServerDatagramSocket get_server_datagram_socket(uint16 port, string dev) throws Error`  
    Crea un socket UDP, lo associa ad una specifica interfaccia di rete, lo abilita alla ricezione di messaggi broadcast
    sulla porta specificata. La chiamata non è bloccante, ma le successive operazioni lo saranno; si veda sotto i metodi
    previsti dall'interfaccia IServerDatagramSocket. Ottenere il socket tramite questo meccanismo consentirà in seguito
    di effettuare le altre operazioni bloccando solo la tasklet corrente e consentendo alle altre di proseguire.
*   `IClientDatagramSocket get_client_datagram_socket(uint16 port, string dev, string? my_addr=null) throws Error`  
    Crea un socket UDP, lo associa ad una specifica interfaccia di rete, lo abilita all'invio di messaggi broadcast sulla
    porta specificata, opzionalmente specificando l'indirizzo IP da usare come *source*. La chiamata non è bloccante, ma
    le successive operazioni lo saranno; si veda sotto i metodi previsti dall'interfaccia IClientDatagramSocket. Ottenere
    il socket tramite questo meccanismo consentirà in seguito di effettuare le altre operazioni bloccando solo la tasklet
    corrente e consentendo alle altre di proseguire.
*   `IChannel get_channel()`  
    Crea un canale.

L'interfaccia ITaskletSpawnable prevede questi metodi:

*   `void * func()`

L'interfaccia ITaskletHandle prevede questi metodi:

*   `bool is_running()`
*   `void kill()`
*   `bool is_joinable()`
*   `void * join()`

La classe TaskletCommandResult ha questi membri:

*   `string stdout`
*   `string stderr`
*   `int exit_status`

L'interfaccia IServerStreamSocket si usa lato server. Essa prevede questi metodi:

*   `IConnectedStreamSocket accept() throws Error`  
    Blocca la tasklet corrente in attesa di una connessione, ma consente alle altre tasklet di proseguire.
*   `void close() throws Error`

L'interfaccia IConnectedStreamSocket si usa su entrambi gli end point della connessione. Essa prevede questi metodi:

*   `unowned string _peer_address_getter()`  
    In realtà si chiama con la property `peer_address`. Si usa sul server dopo aver ricevuto una connessione. Riporta
    l'indirizzo del client.
*   `unowned string _my_address_getter()`  
    In realtà si chiama con la property `my_address`. Si usa sul server dopo aver ricevuto una connessione. Riporta
    l'indirizzo a cui il server è stato contattato.
*   `size_t recv(uint8* b, size_t maxlen) throws Error`  
    Blocca la tasklet in attesa di dati dall'altro end point della connessione, ma consente alle altre tasklet di proseguire.  
    Il valore restituito dalla funzione sarà sempre positivo. In caso di errore la funzione lancia un apposito Error.
    In particolare in caso di 0 bytes (connessione chiusa) viene lanciato l'errore `IOError.Closed`. Questo vale per
    tutte le funzioni di lettura e scrittura sui socket.
*   `void send(uint8* b, size_t len) throws Error`  
    Blocca la tasklet in attesa di completare la trasmissione all'altro end point della connessione, ma consente alle
    altre tasklet di proseguire.
*   `void close() throws Error`

L'interfaccia IServerDatagramSocket prevede questi metodi:

*   `size_t recvfrom(uint8* b, size_t maxlen, out rmt_ip, out rmt_port) throws Error`  
    Blocca la tasklet in attesa di leggere un pacchetto UDP broadcast tramite l'interfaccia di rete, ma consente alle
    altre tasklet di proseguire.
*   `void close() throws Error`

L'interfaccia IClientDatagramSocket prevede questi metodi:

*   `size_t sendto(uint8* b, size_t len) throws Error`  
    Blocca la tasklet in attesa di inviare un pacchetto UDP broadcast sull'interfaccia di rete, ma consente alle altre
    tasklet di proseguire.
*   `void close() throws Error`

L'interfaccia IChannel prevede questi metodi:

*   `void send(Value v)`  
    Accoda un messaggio per la trasmissione su questo canale e blocca la tasklet corrente finché non sia stato letto.
*   `void send_async(Value v)`  
    Accoda un messaggio per la trasmissione su questo canale ma non blocca la tasklet e nemmeno rilascia il controllo
    allo schedulatore.
*   `int get_balance()`  
    Ci dice se ci sono messaggi in attesa di venire letti (valore positivo) o se ci sono altre tasklet in attesa di
    messaggi da leggere (valore negativo).
*   `Value recv()`  
    Legge il prossimo messaggio. Blocca la tasklet in attesa di un messaggio se non ce ne sono. Garantisce che venga
    rispettata la priorità, nel caso in cui altre tasklet cerchino di leggere dallo stesso canale.
*   `Value recv_with_timeout(int timeout_msec) throws ChannelError`  
    Legge il prossimo messaggio. Blocca la tasklet in attesa di un messaggio se non ce ne sono, ma prevede un tempo
    massimo di attesa oltre il quale riprende il controllo.

Quando si desidera avviare una tasklet occorre creare appositamente una istanza di una classe che implementa
l'interfaccia ITaskletSpawnable. Tale classe ha come membri tutti i dati che andrebbero passati alla funzione che
inizia il task. Inoltre ha il metodo `func` che è proprio quello che costituisce il corpo della tasklet che si
vuole avviare. La classe che si istanzia può essere interna ad un'altra classe per fare in modo che abbia accesso
ai suoi membri privati.

Poi si valorizzano i membri della istanza di ITaskletSpawnable, come sarebbero passati gli argomenti alla funzione.
Quindi si chiama il metodo `spawn` di ITasklet, al quale si può indicare se si desidera che la nuova tasklet sia joinable.

L'istanza di ITaskletHandle restituita dal metodo `spawn` potrà essere usata per fare il join della tasklet (se è
joinable) oppure in caso di necessità per abortirla.

Quando il metodo `spawn` restituisce il controllo al chiamante, esso è tenuto a rimuovere immediatamente il suo
riferimento alla istanza di ITaskletSpawnable.

Il metodo `spawn` si occupa di memorizzare un riferimento all'istanza di ITaskletSpawnable per evitare che venga
distrutta e di mantenerlo in vita fino alla fine della tasklet che andiamo a creare. Poi il metodo crea la nuova
tasklet, che eseguirà un'apposita funzione che chiamiamo `real_func`, con l'implementazione che si intende adottare,
sia essa PthTasklet (basata su GNU/Pth) o similari. Di norma la funzione `real_func` ha in C la
signature `void *(*)(void *)`. Come argomento viene passata l'istanza di ITaskletSpawnable dopo averne fatto
il cast a `void *`.

Nella nuova tasklet, la funzione `real_func` riceve il dato come (void *) e ne fa il cast a (ITaskletSpawnable *). Poi
richiama il suo metodo `func` e memorizza il suo valore di ritorno. Infine rimuove il riferimento all'istanza di
ITaskletSpawnable e poi termina restituendo il valore di ritorno di `func`.

### <a name="sequenziali"></a>Tasklet sequenziali

La libreria *tasklet-system* fornisce anche un metodo per assicurare che alcune tasklet vengano eseguite in sequenza.
Usiamo un esempio per illustrare cosa si intende con questa espressione.

Sia un evento *e* che quando si verifica avvia una lunga elaborazione. All'inizio di questa elaborazione viene
scritto a video il messaggio "elaborazione avviata" con un timestamp. Al termine della elaborazione viene scritto a
video il messaggio "elaborazione terminata" con un timestamp. Supponiamo ora che questo evento *e* possa essere segnalato
da due tasklet *t1* e *t2* che sono eseguite parallelamente. Supponiamo che la tasklet *t1* segnala l'evento
*e*. Poi mentre l'elaborazione è in corso, la tasklet *t2* segnala anch'essa l'evento *e*. Noi però, per qualche
motivo, vogliamo che non si accavallino a video i messaggi di inizio e fine elaborazione; vogliamo che il messaggio
di "elaborazione terminata" con il relativo timestamp sia sempre riferibile al messaggio di "elaborazione avviata"
che subito lo precede.

Con il meccanismo che fornisce la libreria *tasklet-system* l'applicazione crea una istanza di DispatchableTasklet
e la memorizza in una variabile *dt*. Inoltre, l'elaborazione da fare come risposta all'evento *e* viene scritta
come una tasklet, cioè nel metodo "func" di una classe che implementa ITaskletSpawnable. Quando si verifica l'evento
*e* viene creata una istanza *ts* di questa classe e valorizzati i suoi membri come per passare gli argomenti alla
funzione "func"; poi viene chiamato su *dt* il metodo `dispatch` passando l'istanza *ts*.

La libreria farà in modo che l'elaborazione avviata dalla tasklet *t1* venga completata prima di iniziare l'elaborazione
avviata dalla tasklet *t2*.

Questo meccanismo viene fornito dalla libreria *tasklet-system*, quindi il suo utilizzatore non lo deve implementare.
Per fornirlo, la libreria si avvale delle implementazioni di tasklet e canali di cui si è parlato prima, le quali
invece sono proprio implementate dall'utilizzatore.

L'interfaccia ITasklet fornisce quindi questo ulteriore metodo, non astratto:

*   `DispatchableTasklet create_dispatchable_tasklet()`  
    Crea un raccoglitore su cui inserire tasklet da eseguire sequenzialmente.

La classe DispatchableTasklet fornisce questi metodi:

*   `void dispatch(ITaskletSpawnable sp, bool wait_end=false, bool wait_start=false)`  
    Accoda una tasklet da eseguire al suo turno. Si può specificare di voler attendere il suo completamento, oppure
    il suo avvio, oppure non attendere affatto.
*   `bool is_emtpy()`  
    Dice se non ci sono tasklet in esecuzione o in attesa in questo raccoglitore.

### <a name="Inizializzazione"></a>Inizializzazione

I moduli del demone *ntkd* che fanno uso delle tasklet vengono inizializzati con il metodo statico `init`, nel quale
il modulo principale fornisce una istanza di ITasklet. In pratica in questo modo dichiara di poter fornire
le implementazioni di tutte le interfacce viste sopra per le comunicazioni allo schedulatore.

