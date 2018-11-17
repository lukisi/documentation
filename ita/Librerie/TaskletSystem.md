# Tasklet System

1.  [Obiettivi](#obiettivi)
1.  [Interfacce](#interfacce)
1.  [Inizializzazione](#inizializzazione)
1.  [Avvio nuove tasklet](#avvio-nuove-tasklet)
1.  [Comunicazioni sulla rete](#comunicazioni-sulla-rete)
1.  [Tasklet sequenziali](#tasklet-sequenziali)

## Obiettivi

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
    *   Attesa di una comunicazione dalla rete. Include comunicazioni inter-processo nello stesso sistema.
    *   Invio di una comunicazione nella rete. Idem.
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
fornisce loro un meccanismo per comunicare tra loro (a loro scelta in modo sincrono o asincrono) senza mai bloccare
le altre tasklet.

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

## Interfacce

L'interfaccia ITasklet prevede questi metodi:

*   `void schedule()`  
    Passa il controllo ad altre tasklet.
*   `void ms_wait(int msec)`  
    Blocca la tasklet corrente per il tempo indicato, ma consente alle altre di proseguire.
*   `[NoReturn] void exit_tasklet(void * ret)`  
    Termina la tasklet corrente.
*   `ITaskletHandle spawn(ITaskletSpawnable sp, bool joinable=false)`  
    Crea una nuova tasklet, come descritto sotto.
*   `TaskletCommandResult exec_command(List<string> argv)`  
    Avvia un nuovo processo (di cui specifica il comando da eseguire) e attende il suo esito. E' bloccante per la tasklet
    corrente, ma consente alle altre di proseguire.
*   `size_t read(int fd, void* b, size_t maxlen) throws Error`  
    Dato il file-descriptor di un file aperto, bloccando solo la tasklet corrente, legge dal file.  
    Il valore restituito dalla funzione sarà sempre non negativo. In caso di errore la funzione lancia un apposito Error.  
    Il valore di ritorno 0 indica fine-del-file.
*   `size_t write(int fd, void* b, size_t count) throws Error`  
    Dato il file-descriptor di un file aperto, bloccando solo la tasklet corrente, scrive sul file.  
    Il valore restituito dalla funzione sarà sempre non negativo. In caso di errore la funzione lancia un apposito Error.  
    Il valore di ritorno 0 è possibile, indica che niente è stato scritto.
*   `IChannel get_channel()`  
    Crea un canale che permette la comunicazione tra due tasklet.
*   `IServerStreamNetworkSocket get_server_stream_network_socket(string my_addr, uint16 my_tcp_port) throws Error`  
    Crea un socket *stream* e lo mette in ascolto su un proprio indirizzo IP specificando una porta TCP.  
    La chiamata di sistema eseguita in questa funzione non è bloccante, ma le successive operazioni di lettura e scrittura
    nel socket lo saranno; si veda sotto i metodi previsti dall'interfaccia IServerStreamNetworkSocket. Ottenere il socket tramite questa
    funzione consentirà in seguito di effettuare le altre operazioni bloccando solo la tasklet corrente e consentendo
    alle altre di proseguire.
*   `IConnectedStreamNetworkSocket get_client_stream_network_socket(string dest_addr, uint16 dest_tcp_port) throws Error`  
    Crea un socket *stream* client (senza specificare un proprio indirizzo IP né una propria porta) e lo connette ad un server specificando
    l'indirizzo IP e la porta TCP di destinazione. E' bloccante per la tasklet corrente, ma consente alle altre di
    proseguire. Anche le successive operazioni sul socket saranno bloccanti; si veda sotto i metodi previsti
    dall'interfaccia IConnectedStreamNetworkSocket. Il meccanismo fornito consentirà di bloccare solo la tasklet corrente.
*   `IServerDatagramNetworkSocket get_server_datagram_network_socket(uint16 udp_port, string my_dev) throws Error`  
    Crea un socket *datagram*, lo associa ad una propria specifica interfaccia di rete, lo abilita alla ricezione di messaggi UDP broadcast
    sulla porta specificata. La chiamata non è bloccante, ma le successive operazioni lo saranno; si veda sotto i metodi
    previsti dall'interfaccia IServerDatagramNetworkSocket. Ottenere il socket tramite questo meccanismo consentirà in seguito
    di effettuare le altre operazioni bloccando solo la tasklet corrente e consentendo alle altre di proseguire.
*   `IClientDatagramNetworkSocket get_client_datagram_network_socket(uint16 udp_port, string my_dev) throws Error`  
    Crea un socket *datagram* client, lo associa ad una propria specifica interfaccia di rete,
    lo abilita all'invio di messaggi UDP broadcast sulla porta specificata. La chiamata non è bloccante, ma
    le successive operazioni lo saranno; si veda sotto i metodi previsti dall'interfaccia IClientDatagramNetworkSocket. Ottenere
    il socket tramite questo meccanismo consentirà in seguito di effettuare le altre operazioni bloccando solo la tasklet
    corrente e consentendo alle altre di proseguire.
*   `IServerStreamLocalSocket get_server_stream_local_socket(string listen_pathname) throws Error`  
    Crea un socket *stream* locale (unix-domain) e lo mette in ascolto su uno specifico pathname. Il pathname specificato deve essere
    creato in questo momento; se esisteva già la chiamata lancia una eccezione.  
    La funzionalità è analoga alla `get_server_stream_network_socket`, ma usa la modalità locale come meccanismo di IPC.
    Questo meccanismo di IPC è usato per le testsuite in cui diversi processi in un unico sistema simulano diversi nodi
    di una rete di computer. Il pathname rappresenta un indirizzo IP assegnato (virtualmente) al nodo simulato.  
    Si veda sotto i metodi previsti dall'interfaccia IServerStreamLocalSocket.
*   `IConnectedStreamLocalSocket get_client_stream_local_socket(string send_pathname) throws Error`  
    Crea un socket *stream* locale client (senza specificare un proprio pathname) e lo connette ad un server specificando
    il pathname di destinazione. Se il pathname non esiste, significa che nessun processo è in ascolto, quindi la chiamata
    lancia una eccezione.  
    La funzionalità è analoga alla `get_client_stream_network_socket`, ma usa la modalità locale come meccanismo di IPC.  
    Si veda sotto i metodi previsti dall'interfaccia IConnectedStreamLocalSocket.
*   `IServerDatagramLocalSocket get_server_datagram_local_socket(string listen_pathname) throws Error`  
    Crea un socket *datagram* locale, e lo mette in ascolto su uno specifico pathname. Il pathname specificato deve essere
    creato in questo momento; se esisteva già la chiamata lancia una eccezione.  
    La funzionalità è analoga alla `get_server_datagram_network_socket`, ma usa la modalità locale come meccanismo di IPC.
    Questo meccanismo di IPC è usato per le testsuite in cui diversi processi in un unico sistema simulano diversi nodi
    di una rete di computer. Il pathname rappresenta una pseudo-interfaccia di rete del nodo simulato.  
    Si veda sotto i metodi previsti dall'interfaccia IServerDatagramLocalSocket.
*   `IClientDatagramLocalSocket get_client_datagram_local_socket(string send_pathname) throws Error`  
    Crea un socket *datagram* locale client (senza specificare un proprio pathname) e lo prepara a trasmettere messaggi
    ad un server specificando il pathname di destinazione. La chiamata non è bloccante, ma
    le successive operazioni lo saranno. Si veda sotto i metodi previsti dall'interfaccia IClientDatagramLocalSocket.  
    La funzionalità è analoga alla `get_client_datagram_network_socket`, ma usa la modalità locale come meccanismo di IPC.  
    Al momento della trasmissione del messaggio, se il pathname di destinazione non esiste, significa che nessun processo
    è in ascolto, quindi la chiamata lancia una eccezione.

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

### Interfacce per le comunicazioni

L'interfaccia IServerStreamNetworkSocket si usa lato server. Essa prevede questi metodi:

*   `IConnectedStreamNetworkSocket accept() throws Error`  
    Blocca la tasklet corrente in attesa di una connessione, ma consente alle altre tasklet di proseguire.
*   `void close() throws Error`  
    Distrugge il socket che era in ascolto di connessioni.  
    Questa interfaccia è usata con socket in ascolto su un proprio indirizzo IP. Può succedere che non si voglia più gestire un
    certo proprio indirizzo IP (ad esempio nel caso di Netsukuku si può dover rimuovere un proprio indirizzo IP).

L'interfaccia IConnectedStreamNetworkSocket si usa su entrambi gli end point della connessione. Essa prevede questi metodi:

*   `size_t recv(uint8* b, size_t maxlen) throws Error`  
    Blocca la tasklet in attesa di dati dall'altro end point della connessione, ma consente alle altre tasklet di proseguire.  
    Il valore restituito dalla funzione sarà sempre non negativo. In caso di errore la funzione lancia un apposito Error.  
    Il valore di ritorno 0 indica che la connessione è stata chiusa (presumibilmente dall'altro lato).
*   `size_t send_part(uint8* b, size_t len) throws Error`  
    Blocca la tasklet in attesa di trasmettere dati all'altro end point della connessione, ma consente alle
    altre tasklet di proseguire.  
    Questa funzione esegue la trasmissione a basso livello: come è noto restituisce il numero di byte effettivamente
    trasmessi, che può essere minore di quanti richiesti. È fornita anche la funzione `send` sotto illustrata che
    si occupa di iterare questa funzione fin quando tutti i dati sono stati trasmessi.  
    Il valore restituito dalla funzione sarà sempre non negativo. In caso di errore la funzione lancia un apposito Error.  
    Il valore di ritorno 0 indica che niente è stato scritto (non che la connessione è stata chiusa).
*   `void send(uint8* b, size_t len) throws Error`  
    Iterando chiamate alla funzione `send_part` sopra esposta, ritorna quando tutti i byte richiesti sono stati trasmessi
    all'altro end point della connessione.
*   `void close() throws Error`  
    Chiude questa connessione.  
    Di norma la chiamata `close` (per convenzione nel caso d'uso di Netsukuku) viene fatta sul client. Il server se ne avvede
    ricevendo il valore di ritorno 0 nella sua chiamata di `recv`.

L'interfaccia IServerDatagramNetworkSocket prevede questi metodi:

*   `size_t recvfrom(uint8* b, size_t maxlen) throws Error`  
    Blocca la tasklet in attesa di leggere un pacchetto UDP broadcast tramite l'interfaccia di rete, ma consente alle
    altre tasklet di proseguire.
*   `void close() throws Error`  
    Distrugge il socket che era in ascolto sull'interfaccia di rete.  
    L'interfaccia IServerDatagramNetworkSocket è usata con socket in ascolto su una propria interfaccia di rete. Può succedere che
    non si voglia più gestire una propria interfaccia di rete.

L'interfaccia IClientDatagramNetworkSocket prevede questi metodi:

*   `size_t sendto(uint8* b, size_t len) throws Error`  
    Blocca la tasklet in attesa di inviare un pacchetto UDP broadcast sull'interfaccia di rete, ma consente alle altre
    tasklet di proseguire.
*   `void close() throws Error`  
    Distrugge il socket che era stato creato per trasmettere sull'interfaccia di rete. Di norma per ogni messaggio da trasmettere
    viene creato un socket nuovo; quindi trasmesso il messaggio è normale rimuovere il socket.

L'interfaccia IServerStreamLocalSocket si usa lato server. Essa prevede questi metodi:

*   `IConnectedStreamLocalSocket accept() throws Error`  
    Blocca la tasklet corrente in attesa di una connessione, ma consente alle altre tasklet di proseguire.
*   `void close() throws Error`  
    Distrugge il socket che era in ascolto di connessioni.  
    Questa interfaccia è usata nelle simulazioni, con socket locali. Il pathname rappresenta un proprio indirizzo IP.
    Come la chiamata della funzione `get_server_stream_local_socket` aveva creato il pathname, così la chiamata della funzione
    `close` sul relativo oggetto `IServerStreamLocalSocket` lo rimuove.

L'interfaccia IConnectedStreamLocalSocket si usa su entrambi gli end point della connessione. Essa prevede questi metodi:

*   `size_t recv(uint8* b, size_t maxlen) throws Error`  
    Analogo al metodo omonimo nell'interfaccia IConnectedStreamNetworkSocket, ma usa la modalità locale come meccanismo di IPC.
*   `size_t send_part(uint8* b, size_t len) throws Error`  
    Analogo al metodo omonimo nell'interfaccia IConnectedStreamNetworkSocket, ma usa la modalità locale come meccanismo di IPC.
*   `void send(uint8* b, size_t len) throws Error`  
    Analogo al metodo omonimo nell'interfaccia IConnectedStreamNetworkSocket, ma usa la modalità locale come meccanismo di IPC.
*   `void close() throws Error`  
    Analogo al metodo omonimo nell'interfaccia IConnectedStreamNetworkSocket, ma usa la modalità locale come meccanismo di IPC.

L'interfaccia IServerDatagramLocalSocket prevede questi metodi:

*   `size_t recvfrom(uint8* b, size_t maxlen) throws Error`  
    Analogo al metodo omonimo nell'interfaccia IServerDatagramNetworkSocket, ma usa la modalità locale come meccanismo di IPC.
*   `void close() throws Error`  
    Analogo al metodo omonimo nell'interfaccia IServerDatagramNetworkSocket, ma usa la modalità locale come meccanismo di IPC.  
    Questa interfaccia è usata nelle simulazioni, con socket locali. Il pathname rappresenta una propria interfaccia di rete.
    Come la chiamata della funzione `get_server_datagram_local_socket` aveva creato il pathname, così la chiamata della funzione
    `close` sul relativo oggetto `IServerDatagramLocalSocket` lo rimuove.

L'interfaccia IClientDatagramLocalSocket prevede questi metodi:

*   `size_t sendto(uint8* b, size_t len) throws Error`  
    Analogo al metodo omonimo nell'interfaccia IClientDatagramNetworkSocket, ma usa la modalità locale come meccanismo di IPC.
*   `void close() throws Error`  
    Analogo al metodo omonimo nell'interfaccia IClientDatagramNetworkSocket, ma usa la modalità locale come meccanismo di IPC.

Come si è potuto notare, le interfacce usate per le comunicazioni in rete e quelle per le comunicazioni tra processi
nello stesso sistema (quelle che finiscono in "`NetworkSocket`" o "`LocalSocket`" e che sono restituite dai metodi di
`ITasklet` che finiscono in "`_network_socket`" o "`_local_socket`") sono identiche. Si preferisce tenerle distinte per
consentire eventualmente in futuro di aggiungere metodi che abbiano firme diverse a seconda del tipo di comunicazione.  
L'implementazione di un sistema di tasklet, nei metodi "`*_network_socket`" e "`*_local_socket`" dell'oggetto che implementa
l'interfaccia `ITasklet`, restituisce presumibilmente istanze di classi distinte. Ad esempio
una classe `ConnectedStreamNetworkSocket` che implementa l'interfaccia `IConnectedStreamNetworkSocket` e
una classe `ConnectedStreamLocalSocket` che implementa l'interfaccia `IConnectedStreamLocalSocket`.  
Tuttavia, per consentire di scrivere codice che possa funzionare con entrambe le classi, ci saranno le interfacce
`IServerStreamSocket`, `IConnectedStreamSocket`, `IServerDatagramSocket` e `IClientDatagramSocket`
che faranno da basi per le interfacce specializzate viste sopra e che definiranno i metodi comuni. In questo modo il
codice di una applicazione, dopo aver ottenuto ad esempio una istanza di `IConnectedStreamNetworkSocket` chiamando
il metodo `get_client_stream_network_socket` e specificando l'indirizzo IP e la porta TCP di destinazione, potrà
castare l'istanza ricevuta all'interfaccia `IConnectedStreamSocket` e usarla in parti di codice che possono
funzionare sia per le comunicazioni in rete che per le comunicazioni tra processi nello stesso sistema.

## Inizializzazione

Il demone *ntkd* inizializza una libreria di implementazione (ad esempio PthTasklet) e ottiene una istanza di una
classe che implementa l'interfaccia ITasklet.

I moduli del demone *ntkd* che fanno uso delle tasklet vengono inizializzati con il metodo statico `init`, nel quale
il modulo principale fornisce una istanza di ITasklet. In pratica in questo modo dichiara di poter fornire
le implementazioni di tutte le interfacce viste sopra per le comunicazioni allo schedulatore.

## Avvio nuove tasklet

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

## Comunicazioni sulla rete

Abbiamo visto che le operazioni che prevedono una comunicazione tra processi sono bloccanti per la tasklet
corrente. Sia quando si attende un messaggio (da un altro nodo della rete o da un altro processo nel nostro
stesso sistema) sia quando si trasmette un messaggio. Quindi queste operazioni svolte da una tasklet danno
implicitamente l'autorizzazione allo schedulatore di passare il controllo ad altre tasklet in attesa di
eventi che permettano a questa tasklet di procedere.

*   Attesa di una connessione su un socket associato ad un proprio indirizzo IP e una porta TCP. Dalla rete.
*   Richiesta di connessione verso un socket identificato con un indirizzo IP e una porta TCP. Sulla rete.
*   Attesa di una connessione su un socket associato ad un pathname. Nel sistema.
*   Richiesta di connessione verso un socket identificato con un pathname. Nel sistema.
*   Attesa di lettura su un socket connesso. Dalla rete o nel sistema.
*   Richiesta di scrittura verso un socket connesso. Sulla rete o nel sistema.
*   Attesa di lettura di un messaggio (senza previa connessione) su un socket associato ad una propria
    interfaccia di rete e una porta UDP. Per rilevare messagi broadcast nel dominio broadcast a cui
    l'interfaccia è collegata. Dalla rete.
*   Richiesta di scrittura di un messaggio (senza previa connessione) su un socket associato ad una propria
    interfaccia di rete e una porta UDP. Per trasmettere messagi broadcast nel dominio broadcast a cui
    l'interfaccia è collegata. Sulla rete.
*   Attesa di lettura di un messaggio (senza previa connessione) su un socket associato ad un pathname.
    Per emulare il rilevamento di messagi broadcast nel dominio broadcast a cui
    una pseudo-interfaccia è collegata. Nel sistema.
*   Richiesta di scrittura di un messaggio (senza previa connessione) su un socket associato ad un pathname.
    Per emulare la trasmissione di messagi broadcast nel dominio broadcast a cui
    una pseudo-interfaccia è collegata. Nel sistema.

*Nota*: il primario obiettivo nella realizzazione di questa API è il demone che realizza lo scambio di messaggi
nel protocollo Netsukuku. In esso si è convenuto che i messaggi possono essere scambiati solo in due modalità:
per connessione verso uno specifico indirizzo IP oppure per messaggio in broadcast su una propria specifica
interfaccia di rete.  
Cioè: non è previsto un messaggio *datagram* (UDP) verso uno specifico indirizzo IP.

*Nota*: per comprendere come si intenda simulare in una testsuite le comunicazioni in modalità *datagram* con
indirizzamento "broadcast" attraverso un socket locale associato ad un pathname, rimandiamo al documento
ntkd-RPC nella sezione [Tipi di medium](../DemoneNTKD/RPC.md#Medium).




## Tasklet sequenziali

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
