# Zero Configuration Dispatchers - Analisi Funzionale

1.  [Obiettivo](#Obiettivo)
1.  [Divisione della logica in 3 livelli](#Divisione_logica_3_livelli)
1.  [Deploy e dipendenze dei 3 livelli](#Deploy_e_dipendenze)
1.  [Tipi di trasmissione](#Trasmissioni)
1.  [Tipi di medium](#Medium)
1.  [Libreria ZCD di basso livello](#Livello_basso)
    1.  [Tasklet in ascolto](#Tasklet_listen)
    1.  [Chiamate a metodi remoti](#Send)
    1.  [Interazioni tra la libreria ZCD e il suo utilizzatore](#Interazioni)

1.  [Interfacce tra i livelli](#Interfacce_tra_livelli)
    1.  [Interfaccia tra MOD-RPC e APP come realizzata con "rpcdesign"](#Interfaccia_modrpc_app_con_rpcdesgin)
1.  [Dettagli tecnici](#Dettagli_tecnici)

## <a id="Obiettivo"></a>Obiettivo

I nodi di una rete Netsukuku per eseguire l'algoritmo distribuito di esplorazione della rete si devono
passare dei messaggi.

Il passaggio di messaggi tra due nodi avviene sempre sotto forma di chiamata a una procedura (o metodo)
remota. Si ha un nodo *n* chiamante, cioè che inizia la comunicazione. Il messaggio che il nodo *n* trasmette
è costituito dal metodo che chiama e dai parametri che tale metodo accetta in ingresso.

In alcuni casi il messaggio che il nodo *n* indirizza al nodo *v* prevede una risposta. Questa è definita
come il valore di ritorno della procedura remota oppure una eccezione che tale procedura dichiara di poter
lanciare. Al termine della procedura il nodo *v* invia un messaggio al nodo *n* indicando il risultato o
l'eccezione. Rientrano in questa tipologia anche i casi in cui la risposta consiste solo nel comunicare che
la procedura è stata completata, cioè le chiamate a metodi che restituiscono *void*.

In altri casi il messaggio, sempre indirizzato dal nodo *n* al nodo *v*, prevede una semplice comunicazione
di avvenuta ricezione. Cioè il nodo *n* viene notificato che la comunicazione è arrivata al nodo *v*, ma
poi non attende il completamento della procedura invocata sul nodo *v*.

In altri casi ancora, il messaggio che il nodo *n* trasmette è comunicato *in broadcast* a tutti i suoi
vicini. Per questa tipologia di messaggi non è previsto alcun messaggio di risposta. In generale non è
nemmeno prevista la notifica di ricezione del messaggio. Tuttavia, almeno in alcune situazioni, il nodo
*n* ha necessità di sapere se, tra i vicini di cui conosce l'esistenza, qualcuno non ha ricevuto
correttamente il messaggio; poiché un messaggio trasmesso in broadcast è di per se non reliable.

Con queste necessità in mente, è stato realizzato un framework, chiamato *ZCD* o *Zero Configuration Dispatchers*,
che permette di realizzare delle Remote Procedure Call.

Sebbene l'obiettivo primario di questo framework sia il supporto dello sviluppo di Netsukuku, in questo
documento tratteremo concetti generali. Per una panoramica su come il demone Netsukuku fa uso del framework
rimandiamo al documento [ntkd-RPC](../DemoneNTKD/RPC.md). Nel presente documento faremo anche dei rimandi puntuali a
sezioni del documento ntkd-RPC.

## <a id="Divisione_logica_3_livelli"></a>Divisione della logica in 3 livelli

Il framework ZCD suddivide la logica delle operazioni in 3 livelli.

Al livello più basso una libreria si occupa delle trasmissioni di messaggi di richiesta e di risposta
attraverso la rete. Si è deciso di usare il formato [JSON](https://en.wikipedia.org/wiki/JSON) per la
serializzazione dei dati da trasmettere nella rete.

Questa libreria è in grado di:

*   Inviare richieste, quando viene invocata dal suo utilizzatore.
*   Ricevere richieste e smistarle, se opportuno, al suo utilizzatore.
*   Inviare risposte.
*   Ricevere risposte e smistarle, se opportuno, al suo utilizzatore.

Mentre la libreria del livello basso è generica e non ha conoscenza dei metodi remoti che
sono stati definiti dall'applicazione che la sta usando, al livello intermedio
abbiamo un'altra libreria specifica che conosce i metodi remoti e le strutture dati che devono essere
passate nei messaggi e sa come serializzarle e deserializzarle. Questa libreria per inviare un messaggio
usa la libreria del livello basso di cui conosce il funzionamento. A sua  volta, verso l'esterno espone
una interfaccia al suo utilizzatore ma in essa può mascherare completamente l'uso della libreria del
livello basso.

L'interfaccia che espone questa libreria intermedia usa classi stub e skeleton. Cioè la libreria intermedia
fornisce al suo utilizzatore una (o più di una) classe stub attraverso la quale esso chiama un metodo
remoto *m* semplicemente invocando un metodo con la stessa signature su un oggetto stub. Inoltre fornisce
una classe skeleton; l'utilizzatore estende questa classe fornendo una implementazione dei metodi remoti
che sarà invocata quando viene ricevuta una chiamata da un client remoto.

Il livello più alto è quindi realizzato dal software che usa la libreria di livello intermedio. Può essere
una applicazione oppure a sua volta una libreria o un modulo. Il ruolo di questa parte è sia chiamare i
metodi remoti negli altri nodi, sia implementare i metodi remoti che sono chiamati dagli altri nodi su questo
nodo. Cioè si occupa della business logic.

## <a name="Deploy_e_dipendenze"></a>Deploy e dipendenze dei 3 livelli

Per brevità chiamiamo ZCD la libreria di livello basso e MOD-RPC la libreria di livello intermedio che
implementa il modello di dati. Chiamiamo APP l'applicazione (che può a sua volta essere una libreria) che
a livello alto usa MOD-RPC.

La libreria ZCD è agnostica, può essere installata su un computer ed essere usata da diversi programmi con
scopi differenti.

Quando si vuole realizzare una applicazione che usa ZCD, bisogna per prima cosa definire le strutture dati
da trasmettere e individuare i metodi remoti con le loro signature. Da questo sarà possibile realizzare la
libreria di livello intermedio MOD-RPC specifica per questo modello di dati. Ad esempio per il demone *ntkd*
si realizzerà la libreria *ntkdrpc*. Tale libreria avrà come diretta dipendenza la libreria ZCD, ma questo
è un dettaglio implementativo che può non essere esposto al suo utilizzatore.

Alla fine il programma / modulo / libreria di alto livello APP avrà come diretta dipendenza la specifica
libreria MOD-RPC, ma non avrà dipendenza da ZCD.

Il framework ZCD mette a disposizione dei tool per facilitare la costruzione della libreria di livello
intermedio automatizzando i passaggi ripetitivi. Attualmente il tool messo a disposizione, chiamato "rpcdesign",
supporta lo sviluppo con il linguaggio Vala. Partendo da una descrizione delle interfacce implementate dai
metodi remoti costruisce il codice Vala che implementa le classi da usare come stub e skeleton. Il codice
prodotto permette di creare una interfaccia tra la libreria MOD-RPC e l'applicazione APP tale che APP non sia
obbligata né ad avere dipendenze verso la libreria ZCD né a usare stringhe in formato JSON.

La libreria ZCD ha bisogno di comporre e interpretare delle stringhe JSON ad uso interno. Ad esempio per
comporre i messaggi delle chiamate o per i messaggi di ACK di cui parleremo in seguito.  
L'implementazione attuale di ZCD usa internamente la libreria JsonGlib per trattare le stringhe in formato
JSON ma non espone questo dettaglio al suo utilizzatore.

La libreria MOD-RPC sa che per interagire con ZCD ha bisogno di produrre e interpretare stringhe in formato
JSON. Ad esempio per serializzare e deserializzare gli argomenti dei messaggi.  
L'implementazione che descriveremo in questo documento (realizzata con "rpcdesign") usa internamente la
libreria JsonGlib, ma questo non viene imposto da ZCD. Infatti la libreria ZCD non è tenuta a saper interpretare
il contenuto delle stringhe JSON che riceve da MOD-RPC, ma deve solo farle arrivare alla libreria MOD-RPC nel
nodo remoto.

## <a name="Trasmissioni"></a>Tipi di trasmissione

Il framework ZCD prevede due tipi di trasmissione:

*   "stream".  
    Con questa modalità ogni messaggio è incapsulato all'interno di una connessione. Quindi
    la ricezione da parte del destinatario è assicurata. Inoltre è possibile inviare in questo
    modo messaggi di qualsiasi dimensione.  
    Con questa modalità la connessione si stabilisce tra il nodo corrente e un altro nodo di
    cui si conosce un indirizzo IP con cui possiamo raggiungerlo.  
    Con questa modalità e solo con questa si trasmettono messaggi destinati ad un unico
    destinatario.
*   "datagram".  
    Con questa modalità ogni messaggio è costituito di un solo pacchetto. Quindi non esiste
    una connessione e la ricezione da parte del destinatario non è assicurata.  
    Con questa modalità ogni pacchetto è di tipo "broadcast" e può essere trasmesso solo su una specifica
    interfaccia di rete. Quindi i possibili destinatari sono quelli collegati allo stesso
    dominio broadcast su cui è collegata questa nostra interfaccia.  
    Con questa modalità e solo con questa si trasmettono messaggi destinati ad un set di
    destinatari.

#### Modalità stream

Lato server:  
Una tasklet attende una connessione su un socket associato ad un proprio indirizzo IP tramite il quale
il mittente lo può identificare. Quando arriva una connessione viene avviata una tasklet che gestisce
quella specifica connessione, mentre la tasklet originale torna ad attendere. La tasklet che gestisce la
connessione può leggere e scrivere sul socket connesso.

Lato client:  
Il mittente del messaggio avvia una connessione con un suo socket (che non ha bisogno di essere identificabile
da altri) verso il socket del destinatario che lui sa identificare. Stabilita
la connessione il mittente può leggere e scrivere sul socket connesso.

Il mittente scrive sul socket connesso la *richiesta*. Il destinatario legge la richiesta. Poi passa ad un *delegato*
le informazioni estrapolate dalla richiesta. Questi restituirà, se il caso, un *dispatcher* da eseguire. La sua
esecuzione produrrà una *risposta*.

Se il messaggio di richiesta prevedeva l'attesa della risposta (`wait_reply=true`) il destinatario
scrive sul socket connesso la risposta. Il mittente legge la risposta.

La connessione termina, per convenzione, quando il mittente la chiude. A quel punto la tasklet che gestisce
quella specifica connessione sul destinatario potrà terminare.

#### Modalità datagram

Lato server:  
Una tasklet ascolta i pacchetti broadcast che transitano
su un dominio broadcast sul quale è "collegata" una sua interfaccia di rete. In questo caso
è il dominio broadcast che in un certo senso può essere identificato dal
mittente, nel senso che anche il mittente è collegato con una sua interfaccia di rete allo stesso
dominio. Quando un pacchetto transita in quel dominio il destinatario lo rileva.

Lato client:  
Il mittente del messaggio tramite una propria interfaccia di rete trasmette
un pacchetto broadcast sul dominio broadcast a cui sa che è collegata una certa interfaccia
di rete del destinatario (o dei destinatari).

In questo caso non c'è una connessione: il mittente non ha la certezza che il pacchetto venga
rilevato dai destinatari. Ma può richiedere che questi notifichino la ricezione con un pacchetto
di "ACK". In questo caso anche il mittente può essere raggiungibile, per il fatto che ha una interfaccia
di rete collegata allo stesso dominio broadcast in cui il destinatario ha rilevato il pacchetto.
Quindi anche il mittente è a sua volta in ascolto con una tasklet.

Quindi la tasklet in ascolto dei pacchetti broadcast ha un duplice compito: i pacchetti che rileva possono
essere "REQUEST" o "ACK".

Quando rileva un pacchetto viene avviata una tasklet che gestisce il pacchetto rilevato, mentre la tasklet originale
torna ad ascoltare.

*   Se il pachetto è un "REQUEST":
    *   Se richiede un ACK (`send_ack=true`):
        *   Avvia una tasklet che trasmetterà un relativo pacchetto "ACK" sulla stessa interfaccia di rete.
    *   Passa ad un *delegato* le informazioni estrapolate dal pacchetto "REQUEST". Questi
        restituirà, se il caso, un *dispatcher* da eseguire. Dopo averlo eseguito la tasklet potrà terminare.
*   Se il pachetto è un "ACK":
    *   Passa al *delegato* le informazioni estrapolate dal pacchetto "ACK". Questi
        non restituirà nulla: la tasklet potrà terminare.

## <a name="Medium"></a>Tipi di medium

Il framework ZCD prevede due tipi di medium per la trasmissione:

*   "net".  
    Due nodi appartenenti ad una rete comunicano tra loro.  
    Si realizzano queste comunicazioni usando i socket classici.  
    Lato server questi socket sono associati (a seconda della modalità di trasmissione):

    *   ad un proprio indirizzo IP e una porta TCP.
    *   ad una propria interfaccia di rete e una porta UDP;

    Lato client questi socket sono associati (a seconda della modalità di trasmissione):

    *   ad un indirizzo IP di destinazione e una porta TCP di destinazione.
    *   ad una propria interfaccia di rete e una porta UDP di destinazione; in questo caso la
        destinazione del messaggio è nel dominio broadcast a cui è collegata l'interfaccia.

*   "system".  
    Due processi in esecuzione su uno stesso sistema comunicano tra loro.  
    Si realizzano queste comunicazioni usando i socket unix-domain.  
    Lato server questi socket sono associati (in entrambe le modalità di trasmissione)
    ad un pathname: questo pathname è creato da questo stesso processo, subito prima di
    mettersi in ascolto su di esso. Se il pathname esiste già il processo rifiuta di mettersi
    in ascolto e va in errore (fatale).  
    Lato client questi socket sono associati (in entrambe le modalità di trasmissione)
    ad un pathname di destinazione: un altro processo dovrebbe aver già creato questo pathname
    ed essersi messo in ascolto su di esso. Se non è così il messaggio non viene in realtà
    trasmesso, ma non si tratta di un errore fatale, il processo lo ignora.

#### Medium net

Il medium "net" è il primario obiettivo del framework.

#### Medium system

Il supporto al medium "system" ha principalmente lo scopo di facilitare la produzione di testsuite
in cui più processi (all'interno di un sistema) simulano più nodi (all'interno di una rete).

D'ora in poi, generalizzando, parleremo di *messaggi ricevuti dalla rete*, anche per indicare i messaggi
ricevuti su un socket unix-domain.

Inoltre parleremo di *nodo* anche nel caso di socket unix-domain, indicando così in questo caso il processo
che usa la libreria ZCD e si mette in ascolto dei messaggi.

Nel documento ntkd-RPC nella sezione [Tipi di medium](../DemoneNTKD/RPC.md#Medium) descriveremo anche un
meccanismo che consente, nel caso di modalità "datagram", di emulare un dominio broadcast
usando questi socket.

## <a name="Livello_basso"></a>Libreria ZCD di basso livello

Analiziamo la libreria ZCD, quella di basso livello.

### <a name="Tasklet_listen"></a>Tasklet in ascolto

Un programma che usa il framework ZCD deve indicare dove intende mettersi in ascolto.

Rammentando le modalità di trasmissione e i medium previsti dal framework, abbiamo come conseguenza
che la libreria di basso livello può mettersi in ascolto di messaggi nei seguenti modi:

1.  Attendere connessioni con il protocollo TCP (su una precisa porta TCP) su un proprio indirizzo IP.  
    La libreria espone la funzione `stream_net_listen(my_ip, tcp_port)` che avvia una tasklet che si mette in
    ascolto in questo modo.  
    L'utilizzatore specifica l'indirizzo IP (deve essere uno proprio del nodo) e la porta TCP.
1.  Attendere connessioni su un socket unix-domain legato ad uno specifico pathname.  
    La libreria espone la funzione `stream_system_listen(listen_pathname)` che avvia una tasklet che si mette
    in ascolto in questo modo.  
    L'utilizzatore specifica un pathname di ascolto che deve essere univoco all'interno del set di processi
    che compongono la testsuite e deve riflettere la modalità di comunicazione con connessione.  
    Indicheremo nel documento ntkd-RPC nella sezione [Tipi di medium](../DemoneNTKD/RPC.md#Medium)
    le scelte fatte nell'applicazione *ntkd* per mappare in un pathname le situazioni di comunicazione con connessione
    che si possono incontrare.
1.  Attendere messaggi broadcast con il protocollo UDP (su una precisa porta UDP) su una propria interfaccia di
    rete. Si fa con un socket impostato a "set_broadcast" e legato all'interfaccia di rete. Con
    questa modalità di fatto si ascoltano i pacchetti broadcast che transitano
    sul [broadcast domain](https://en.wikipedia.org/wiki/Broadcast_domain)
    al quale è collegata quella interfaccia di rete.  
    La libreria espone la funzione `datagram_net_listen(my_dev, udp_port, string ack_mac)` che avvia una tasklet che si mette
    in ascolto in questo modo.  
    L'utilizzatore specifica il dev-name della propria interfaccia di rete e la porta UDP.  
    Le tasklet che ascoltano trasmissioni di tipo "datagram" ricevono un parametro `string ack_mac`. Esso è da utilizzare nella
    successiva trasmissione dei messaggi di *ACK* relativi ai messaggi di *richiesta* ricevuti.
1.  Attendere messaggi su un socket unix-domain legato ad uno specifico pathname.  
    La libreria espone la funzione `datagram_system_listen(listen_pathname, send_pathname, string ack_mac)` che avvia una tasklet
    che si mette in ascolto in questo modo.  
    L'utilizzatore specifica un pathname di ascolto che deve essere univoco all'interno del set di processi
    che compongono la testsuite e deve riflettere la modalità di comunicazione con messaggi broadcast. Inoltre
    specifica un pathname di trasmissione per la trasmissione dei pacchetti ACK.  
    Le tasklet che ascoltano trasmissioni di tipo "datagram" ricevono un parametro `string ack_mac`. Esso è da utilizzare nella
    successiva trasmissione dei messaggi di *ACK* relativi ai messaggi di *richiesta* ricevuti.  
    Indicheremo nel documento ntkd-RPC nella sezione [Tipi di medium](../DemoneNTKD/RPC.md#Medium)
    le scelte fatte nell'applicazione *ntkd* per mappare in un pathname le situazioni di comunicazione con messaggi
    broadcast che si possono incontrare.

L'utilizzatore può ordinare alla libreria ZCD di mettersi in ascolto con delle tasklet in una
o più di una di queste modalità. Per ogni tasklet che viene così avviata l'utilizzatore deve ottenere
un handle col quale possa essere in grado in futuro di terminare la tasklet.

Durante il corso della vita del programma può essere necessario avviare nuove tasklet di ascolto (sia di tipologia
stream che datagram) e terminarne alcune.  
Ad esempio, nel caso del demone *ntkd* alcuni indirizzi IP su cui questo dovrà mettersi in ascolto
(quelli routabili interni ad un g-nodo) possono cambiare nel tempo. Quindi quando stanno per essere rimossi
i vecchi indirizzi IP propri del nodo bisogna prima terminare le tasklet in ascolto su di essi. Anche per
il caso di tasklet in ascolto per datagram su una particolare interfaccia di rete, il demone *ntkd*
dovrebbe prevedere la possibilità di aggiungere o rimuovere dinamicamente le interfacce gestite.

### <a name="Send"></a>Chiamate a metodi remoti

Quando l'utilizzatore di ZCD vuole chiamare un metodo remoto invoca un metodo di ZCD.

Rammentando le modalità di trasmissione e i medium previsti dal framework, abbiamo come conseguenza
che ci sono diverse modalità, per ognuna delle quali ZCD mette a disposizione un distinto metodo. Esse sono:

1.  Stabilire una connessione con il protocollo TCP (su una precisa porta TCP) ad un preciso indirizzo IP.  
    La libreria espone la funzione `send_stream_net(peer_ip, tcp_port, ...)` che apre una connessione e
    porta avanti le comunicazioni legate ad una chiamata a metodo remoto.  
    L'utilizzatore specifica un indirizzo IP (che sa essere del destinatario) e la porta TCP.
1.  Stabilire una connessione su un socket unix-domain legato ad uno specifico pathname.  
    La libreria espone la funzione `send_stream_system(send_pathname, ...)` che apre una connessione e
    porta avanti le comunicazioni legate ad una chiamata a metodo remoto.  
    L'utilizzatore specifica un pathname sul quale sa che il destinatario è in ascolto e che riflette
    la modalità di comunicazione con connessione.
1.  Inviare un messaggio broadcast con il protocollo UDP (su una precisa porta UDP) su una propria interfaccia di
    rete. Si fa con un socket impostato a "set_broadcast" e legato all'interfaccia di rete. Con questa modalità di fatto si
    fa transitare un pacchetto broadcast sul [broadcast domain](https://en.wikipedia.org/wiki/Broadcast_domain)
    al quale è collegata quella interfaccia di rete.  
    La libreria espone la funzione `send_datagram_net(my_dev, udp_port, ...)` che trasmette in questo modo.  
    L'utilizzatore specifica il dev-name della propria interfaccia di rete e la porta UDP.
1.  Inviare un messaggio su un socket unix-domain legato ad uno specifico pathname.  
    La libreria espone la funzione `send_datagram_system(send_pathname, ...)` che trasmette in questo modo.  
    L'utilizzatore specifica un pathname sul quale sa che un programma di proxy è in ascolto. Come è spiegato nel
    documento ntkd-RPC nella sezione [Tipi di medium](../DemoneNTKD/RPC.md#Medium).

#### Modalità stream

Nelle invocazioni di `send_stream_net` e `send_stream_system`, gli altri parametri che l'utilizzatore specifica
sono:

*   `string source_id`. Serializzazione di un oggetto (ignoto al framework ZCD) che rappresenta il mittente
    del messaggio.
*   `string unicast_id`. Serializzazione di un oggetto (ignoto al framework ZCD) che rappresenta il destinatario
    del messaggio.
*   `string src_nic`. Serializzazione di un oggetto (ignoto al framework ZCD) che rappresenta una precisa
    interfaccia di rete del mittente che ha usato per trasmettere il messaggio.
*   `string m_name`. Nome del metodo (ignoto al framework ZCD) da eseguire sul destinatario.
*   `List<string> arguments`. Serializzazione di una lista di oggetti da passare al metodo.
*   `bool wait_reply`. Indica se il mittente resta in attesa del risultato.  
    Se è così, questa funzione termina quando il metodo remoto è stato eseguito dal destinatario.
    Restituisce all'utilizzatore una stringa che è la serializzazione di un oggetto
    (ignoto al framework ZCD) che rappresenta il risultato.  
    Altrimenti, questa funzione termina quando il destinatario ha ricevuto il messaggio.
    Non attende cioè la sua esecuzione.

#### Modalità datagram

Nelle invocazioni di `send_datagram_net` e `send_datagram_system`, gli altri parametri che l'utilizzatore specifica
sono:

*   `int packet_id`. Identificativo univoco del messaggio da trasmettere.
*   `string source_id`. Serializzazione di un oggetto (ignoto al framework ZCD) che rappresenta il mittente
    del messaggio.
*   `string broadcast_id`. Serializzazione di un oggetto (ignoto al framework ZCD) che rappresenta il set
    di destinatari del messaggio.
*   `string src_nic`. Serializzazione di un oggetto (ignoto al framework ZCD) che rappresenta una precisa
    interfaccia di rete del mittente che ha usato per trasmettere il messaggio.
*   `string m_name`. Nome del metodo (ignoto al framework ZCD) da eseguire sul destinatario.
*   `List<string> arguments`. Serializzazione di una lista di oggetti da passare al metodo.
*   `bool send_ack`. Indica se il mittente richiede ai nodi che ricevono il messaggio la trasmissione
    di un pacchetto di ACK.  
    Se è così, ogni nodo che rileva il messaggio da una sua interfaccia di rete trasmette subito sulla stessa
    interfaccia (quindi sullo stesso dominio broadcast su cui il mittente ha trasmesso) un pacchetto ACK in
    cui sono specificati lo stesso `int packet_id` del messaggio di richiesta e un `string ack_mac` che è
    l'identificativo dell'interfaccia di rete (reale se il medium è net, finta se il medium è system)
    che ha ricevuto il messaggio.  
    In ogni caso la funzione termina appena il messaggio è stato trasmesso, senza avere certezza che
    qualcuno lo abbia ricevuto.

#### Elementi comuni

In entrambe le modalità (stream e datagram) abbiamo dei parametri comuni.

*   `m_name`. Il nome del metodo da chiamare. Per ZCD questa è semplicemente una stringa.  
    La libreria MOD-RPC può avere qualche ulteriore convenzione, ad esempio il nome di un attributo della classe
    dello skeleton radice, seguito da un punto e dal nome del metodo.
*   `arguments`. Un elenco di stringhe in formato JSON che rappresentano gli argomenti del metodo nell'ordine in cui
    sono nella sua signature. Per ZCD sono semplicemente stringhe valide nel formato JSON.  
    La libreria MOD-RPC ha avuto l'incarico di produrle serializzando dei dati e sarà in grado, dall'altro
    lato della comunicazione, di deserializzarle nella giusta forma.  
    Laddove questi siano istanze di oggetti (ma questo è un dettaglio implementativo non obbligatorio)
    solitamente la libreria MOD-RPC conosce solo una interfaccia vuota implementata dall'oggetto. Non
    necessita di conoscere la classe dell'oggetto, ma solo che è un `Object` serializzabile.
*   `source_id`. Una stringa in formato JSON che rappresenta il mittente.
*   `unicast_id`. Una stringa in formato JSON che rappresenta il destinatario.
*   `broadcast_id`. Una stringa in formato JSON che rappresenta il set di destinatari.
*   `src_nic`. Una stringa in formato JSON che rappresenta l'interfaccia di rete su cui il mittente trasmette.  
    Anche in questi casi per la libreria ZCD sono solo stringhe valide nel formato JSON.  
    La libreria MOD-RPC ha avuto l'incarico di produrle serializzando un oggetto e sarà in grado, dall'altro
    lato della comunicazione, di deserializzarle nella giusta forma.

### <a name="Interazioni"></a>Interazioni tra la libreria ZCD e il suo utilizzatore

L'utilizzatore di ZCD inizializza come detto prima la libreria di basso livello indicando dove ascoltare.

A tutte le tasklet che stanno in ascolto, fornisce dei delegati che saranno usati per identificare un messaggio
e decidere se vada ignorato o preso in carico.

Il framework ZCD supporta il concetto di *identità* multiple in un nodo. Cioè ogni singolo nodo può assumere
diverse *identità* distinte.  
I delegati che vengono passati alla libreria ZCD conoscono il cosidetto *identificativo di identità* che
individua ogni *identità* che il nodo ha assunto.

Così, quando ZCD riceve un messaggio dalla rete questi delegati sono in grado di decidere se darlo in carico ad uno
specifico **dispatcher**<sup>1</sup>.

Quando vengono invocati i metodi che iniziano una chiamata a metodo remoto, la libreria ZCD nel nodo
corrente fa pervenire le informazioni alla libreria ZCD nel nodo (o nodi) destinatario. Qui la libreria
ZCD interroga i delegati passati dal suo utilizzatore per sapere se il messaggio è riconosciuto di sua
pertinenza.

Per interrogare i delegati, prima di tutto la libreria ZCD costruisce una struttura dati CallerInfo
che contiene informazioni accessorie sul messaggio che è stato ricevuto. Accessorie in quanto ulteriori
rispetto al contenuto del messaggio che è essenzialmente il nome del metodo remoto e i suoi argomenti.

La classe CallerInfo è vuota. Viene ereditata dalle classi StreamCallerInfo e DatagramCallerInfo.

Se la tasklet che ha ricevuto il messaggio era in ascolto per connessioni, cioè `stream_net_listen` o
`stream_system_listen`, viene prodotta una istanza di StreamCallerInfo. Questa contiene:

*   `string source_id`. 
*   `string src_nic`
*   `string unicast_id`
*   `string m_name`
*   `bool wait_reply`
*   `Listener listener`

Se invece la tasklet che ha ricevuto il messaggio era in ascolto per messaggi broadcast, cioè `datagram_net_listen` 
o `datagram_system_listen`, viene prodotta una istanza di DatagramCallerInfo. Questa contiene:

*   `int packet_id`.
*   `string source_id`. 
*   `string src_nic`
*   `string broadcast_id`
*   `string m_name`
*   `bool send_ack`
*   `Listener listener`

La classe Listener è vuota. Viene ereditata dalle seguenti classi esposte dalla libreria ZCD:

*   StreamNetListener. Prodotta da `stream_net_listen`. Contiene i suoi parametri `my_ip, tcp_port`.
*   StreamSystemListener. Prodotta da `stream_system_listen`. Contiene i suoi parametri `listen_pathname`.
*   DatagramNetListener. Prodotta da `datagram_net_listen`. Contiene i suoi parametri `my_dev, udp_port, ack_mac`.
*   DatagramSystemListener. Prodotta da `datagram_system_listen`. Contiene i suoi parametri `listen_pathname, send_pathname, ack_mac`.

Quindi la libreria ZCD passa questo oggetto CallerInfo ai delegati forniti dal suo utilizzatore.

I delegati sono forniti al momento dell'avvio di una tasklet di ascolto. Esistono diversi delegati.
Per le comunicazioni per connessioni abbiamo il delegato `IStreamDelegate stream_dlg`. Per
le comunicazioni per messaggio broadcast abbiamo il delegato `IDatagramDelegate datagram_dlg`.

Come si usa il delegato `stream_dlg`: sulla ricezione di un messaggio su una connessione, dopo aver costruito uno
StreamCallerInfo, viene chiamato il suo metodo `IStreamDispatcher? get_dispatcher(StreamCallerInfo caller_info)`.  
Se il risultato è nullo significa che il messaggio non va processato. La tasklet termina.  
Se il risultato non è nullo, su questo `IStreamDispatcher` viene chiamato il metodo
`string execute(string m_name, List<string> args, StreamCallerInfo caller_info)`. Cioè di fatto viene
processato il messaggio chiamando un metodo di uno skeleton che risiede in questo nodo. Si ottiene una
stringa che è la serializzazione del risultato.  
Ottenuta la stringa risultato, se la richiesta conteneva `wait_reply`, essa viene trasmessa dal destinatario
del messaggio al suo mittente.  
Poi la tasklet termina.

Come si usa il delegato `datagram_dlg`: dobbiamo ricordare che i messaggi broadcast che si ricevono
possono essere *request* o *ack*. Sulla ricezione di una *request* viene per prima cosa chiamato il
suo metodo `bool is_my_own_message(int packet_id)` per evitare che un nodo reagisca al messaggio che
esso stesso ha trasmesso. In questo caso la tasklet termina.  
Il prossimo passaggio sarà di trasmettere, se la richiesta conteneva `send_ack`, il messaggio *ack*.
Il messaggio *ack* viene trasmesso dalla libreria ZCD chiamando una sua funzione interna
(`send_ack_net(my_dev, udp_port, ...)` o `send_ack_system(send_pathname, ...)` a seconda della tasklet
in ascolto che ha ricevuto la *request*) dove si specifica il `int packet_id` e lo `string ack_mac`
dell'interfaccia di rete che ha ricevuto la *request*.  
In seguito, dopo aver costruito un DatagramCallerInfo, viene chiamato il suo metodo
`IDatagramDispatcher? get_dispatcher(DatagramCallerInfo caller_info)`.  
Se il risultato è nullo significa che il messaggio non va processato.  
Se il risultato non è nullo, su questo `IDatagramDispatcher` viene chiamato il metodo
`void execute(string m_name, List<string> args, DatagramCallerInfo caller_info)`. In questo caso non
è previsto comunicare il risultato. La tasklet termina.

Invece sulla ricezione di un *ack* viene chiamato il suo metodo `void got_ack(int packet_id, string ack_mac)`.
Esso non restituisce nulla. La tasklet termina.

In ogni caso il `CallerInfo caller_info` viene passato al metodo remoto eseguito nello skeleton.

* * *

<sub>Nota 1: Il dispatcher restituito dal delegato a fronte di un messaggio ricevuto è uno solo.  
Quando il messaggio è in modalità datagram, il `broadcast_id` può identificare diversi destinatari. Se si tratta
di *identità* piuttosto che di *nodi*, quando un nodo riceve il messaggio può darsi
che più di uno di questi destinatari sia una *identità* di questo nodo. In questo caso il
dispatcher `IDatagramDispatcher` ottenuto dal delegato si prende cura di notificare il messaggio alle
diverse *identità* di questo nodo che sono interessate; di fatto avvia una nuova tasklet per ogni identità
interessata e chiama in essa il metodo sul relativo skeleton.  
Inoltre, in questo caso, se il messaggio prevede una comunicazione di ACK da ogni destinatario che lo riceve,
la libreria ZCD trasmette un unico ACK con un identificativo `string ack_mac` dal quale si possa risalire
all'interfaccia che ha ricevuto il messaggio.  
Dall'altra parte, il nodo che riceve questo ACK deve essere in grado di associarlo
all'intero set di *identità* di questo nodo che erano interessate.</sub>

## <a name="Interfacce_tra_livelli"></a>Interfacce tra i livelli

L'interfaccia della libreria ZCD nei confronti della libreria di livello intermedio MOD-RPC prevede
il passaggio di alcune strutture dati che la libreria ZCD non conosce. Tali dati vengono passati come
stringhe in formato JSON. Si tratta delle seguenti strutture:

*   Lista di argomenti da passare al metodo remoto. Passati da MOD-RPC a ZCD nel mittente e viceversa
    nel destinatario. Ogni elemento è una stringa in formato JSON.
*   Identificativo `source_id`. Passato da MOD-RPC a ZCD nel mittente e viceversa nel destinatario.
*   Identificativo `src_nic`. Passato da MOD-RPC a ZCD nel mittente e viceversa nel destinatario.
*   Identificativo `unicast_id`, per la modalità stream. Passato da MOD-RPC a ZCD nel mittente
    e viceversa nel destinatario.
*   Identificativo `broadcast_id`, per la modalità datagram. Passato da MOD-RPC a ZCD nel mittente e
    viceversa nel destinatario.
*   Esito restituito da un metodo remoto. Passato da MOD-RPC a ZCD nel destinatario e viceversa nel
    mittente. Può essere un valore di ritorno o una eccezione definita da MOD-RPC.

Queste stringhe sono ognuna un JSON valido. Il formato JSON prevede che il nodo radice sia un oggetto o
un array. Oltre a rispettare questo vincolo la libreria MOD-RPC non ha altri vincoli su come costruire
il JSON a partire dalla struttura dati che intende passare, purché sia in grado di interpretare quel
JSON per ricostruire la struttura dati.

L'interfaccia di MOD-RPC verso la APP non ha nessun vincolo imposto dal framework ZCD. Potrebbe anche non
esserci alcuna separazione tra il livello intermedio e quello alto, applicativo, cioè l'applicazione
potrebbe interfacciarsi direttamente con la libreria ZCD.

Comunque, la separazione dei due livelli è consigliabile per alcuni motivi:

*   Se si usa il tool "rpcdesign" (o altri tool che possono essere sviluppati) partendo da una descrizione
    ad alto livello delle interfacce implementate dai metodi remoti (in un formato apposito,
    detto [RPC-IDL](ZcdRpcidl.md)) si ottengono rapidamente delle classi
    da usare come stub e skeleton. In esse il lavoro di serializzazione e deserializzazione dei dati e i
    controlli di correttezza sono già stati fatti e testati; quindi ci si può concentrare
    sugli algoritmi di alto livello (business logic).
*   Si può definire una versione stabile delle interfacce di comunicazione, realizzando ad esempio una libreria
    MOD-RPC versione 1.0. Qualsiasi versione di APP che si appoggia su questa libreria sarà quindi teoricamente
    in grado di comunicare con versioni differenti di APP che si appoggiano però sulla stessa libreria.

### <a name="Interfaccia_modrpc_app_con_rpcdesgin"></a>Interfaccia tra MOD-RPC e APP come realizzata con "rpcdesign"

Abbiamo detto che il tool "rpcdesign" aiuta nella produzione di una libreria MOD-RPC che adotta un
approccio di tipo stub-skeleton come interfaccia verso la APP.

Le classi stub si occupano di serializzare ogni argomento per ogni metodo
remoto. In questa implementazione il nodo radice è sempre un nodo di tipo *oggetto* con un solo membro di
nome "argument". Il suo contenuto è:

*   Per i valori *null* un nodo di tipo *null*.
*   Per i tipi base (interi, booleani, decimali e stringa) un nodo di tipo *value*.
*   Per i dati binari (sequenze di bytes) un nodo di tipo *value* il cui contenuto è una stringa ottenuta
    codificando il flusso binario in [Base64](https://en.wikipedia.org/wiki/Base64).
*   Per qualsiasi oggetto *obj* di una classe derivata da *GLib.Object* un nodo di tipo *oggetto* con due
    membri che definiscono il nome della classe come ottenuto con `obj.get_type().name()` e la serializzazione
    dell'oggetto come fornita da `Json.gobject_serialize(obj)`.
*   Per le liste un nodo di tipo *array*, i cui elementi sono tutti dello stesso tipo in base al tipo della
    lista, come sopra.

Le classi skeleton si occupano di deserializzare ogni argomento per ogni metodo
remoto e poi di invocare il metodo astratto che sarà fornito dalla APP. Il metodo può restituire un valore
di ritorno o una eccezione. Se restituisce un valore la classe skeleton si occupa di serializzarlo. In
questa implementazione il nodo radice è sempre un nodo di tipo *oggetto* con un solo membro di nome
"return-value". Il suo contenuto è analogo a quanto visto per gli argomenti. Se invece va restituita una
eccezione questa viene serializzata con un nodo di tipo *oggetto* con un solo membro di nome "error".
Il suo contenuto è un oggetto con 3 membri di tipo stringa i cui nomi sono "error-domain", "error-code"
e "error-message" e i cui valori sono le rappresentazioni stringa dei relativi valori della struttura di
errore definita in Glib, ad esempio `"SomeDomainError", "MY_ERROR_NAME", "Richiesta non accettata"`.

Infine, le classi stub si occupano di deserializzare per ogni metodo il valore
di ritorno o l'eccezione. Il valore di ritorno viene restituito al chiamante. Oppure, l'eccezione viene
ricreata e lanciata.

Le classi stub e skeleton realizzano una implementazione della libreria di livello intermedio (cioè MOD-RPC) che
verso la APP non lascia trasparire le problematiche della comunicazione. Il codice che implementa APP sa che MOD-RPC
fornisce dei metodi per ottenere istanze di classi stub e che lui stesso (APP) deve fornire istanze di classi skeleton.
La libreria MOD-RPC ovviamente espone le API (cioè le signature dei metodi) delle classi stub e skeleton, così come
definite nel modello di dati espresso nel RPC-IDL.

Per quanto riguarda le strutture dati `source_id`, `src_nic`, `unicast_id`, `broadcast_id`, queste lato client
(cioè lato mittente di un messaggio) sono costruite da APP e poi passate quando chiama i metodi di MOD-RPC che
forniscono le classi stub, per poi chiamarne i metodi remoti. Lato server (cioè lato ricevente di un messaggio)
queste sono esaminate da APP nei delegati che fornisce a MOD-RPC quando lo mette in ascolto.  
Per queste strutture dati, la loro definizione è demandata a APP. La libreria MOD-RPC si limita a definire
una interfaccia vuota per ognuna di esse.  
La loro serializzazione e deserializzazione avviene in MOD-RPC come per un "argument"/"return-value" di
tipo `GLib.Object`.

La libreria MOD-RPC non fa affidamento sui messaggi che riceve
dalla rete. Fa i dovuti controlli e se il contenuto dei messaggi non rispetta le convenzioni stabilite
(tipo degli argomenti, tipo del valore di ritorno, ...) non fa crashare l'applicazione ma restituisce
apposite eccezioni.

Siccome la maggior parte del lavoro di serializzazione e deserializzazione è svolto dalla libreria MOD-RPC
la APP può non avere alcuna cognizione della serializzazione con il formato JSON. Deve tuttavia essere
consapevole che le strutture dati complesse che transitano nei messaggi devono poter essere
serializzate e deserializzate con il meccanismo fornito da `Json.gobject_serialize`. Rimandiamo
alla documentazione di JsonGlib per maggiori dettagli.

## <a name="Dettagli_tecnici"></a>Dettagli tecnici

I dettagli tecnici sono illustrati in questo [documento](ZcdDettagliTecnici.md).

