# Zero Configuration Dispatchers - Analisi Funzionale

1.  [Obiettivo](#Obiettivo)
1.  [Divisione della logica in 3 livelli](#Divisione_logica_3_livelli)
1.  [Deploy e dipendenze dei 3 livelli](#Deploy_e_dipendenze)
1.  [Tipi di trasmissione](#Trasmissioni)
1.  [Tipi di medium](#Medium)
1.  [Interazioni dei 3 livelli](#Interazioni_dei_livelli)
    1.  [Interfacce tra i livelli](#Interfacce_tra_livelli)
        1.  [Interfaccia tra MOD-RPC e APP come realizzata con "rpcdesign"](#Interfaccia_modrpc_app_con_rpcdesgin)
1.  [Dettagli tecnici](#Dettagli_tecnici)

## <a name="Obiettivo"></a>Obiettivo

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

## <a name="Divisione_logica_3_livelli"></a>Divisione della logica in 3 livelli

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
libreria di livello intermedio MOD-RPC specifica per questo modello di dati. Ad esempio per il demone ntkd
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
JSON. Per questo anche essa può usare internamente la libreria JsonGlib, ma questo non viene imposto da ZCD.

La libreria ZCD non è tenuta a saper interpretare il contenuto delle stringhe JSON che riceve da MOD-RPC,
ma deve solo farle arrivare alla libreria MOD-RPC nel nodo remoto. Per questo la librerià MOD-RPC è libera
di scegliere la sua strategia per comporre l'albero JSON a partire dalla struttura dati che vuole passare
al metodo remoto. Ad esempio, la libreria MOD-RPC può scegliere di usare la serializzazione delle classi
GObject come viene fornita dalla libreria JsonGlib senza per questo dover assumere che anche la libreria
di basso livello ZCD ne faccia uso.

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
    *   Se richiede un ACK:
        *   Avvia una tasklet che trasmetterà un relativo pacchetto "ACK" sulla stessa interfaccia di rete.
    *   Passa ad un *delegato di request* le informazioni estrapolate dal pacchetto "REQUEST". Questi
        restituirà, se il caso, un *dispatcher* da eseguire. Dopo averlo eseguito la tasklet potrà terminare.
*   Se il pachetto è un "ACK":
    *   Passa ad un *delegato di ack* le informazioni estrapolate dal pacchetto "ACK". Questi
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
    ad un pathname su cui questo processo è in ascolto.  
    Lato client questi socket sono associati (in entrambe le modalità di trasmissione)
    ad un pathname di destinazione su cui un altro processo è in ascolto.

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

## <a name="Basso_livello_lato_server"></a>Tasklet in ascolto al basso livello (lato server)

Un programma che usa il framework ZCD per prima cosa inizializza la libreria ZCD indicando dove ascoltare.

Rammentando le modalità di trasmissione e i medium previsti dal framework, abbiamo come conseguenza
che la libreria di basso livello può mettersi in ascolto di messaggi nei seguenti modi:

1.  Attendere connessioni con il protocollo TCP (su una precisa porta TCP) su un proprio indirizzo IP.  
    La libreria espone la funzione `stream_net_listen(ip,tcp_port)` che avvia una tasklet che si mette in ascolto in questo modo.  
    L'utilizzatore specifica l'indirizzo IP (deve essere uno proprio del nodo) e la porta TCP.
1.  Attendere messaggi con il protocollo UDP (su una precisa porta UDP) su una propria interfaccia di
    rete con un socket impostato a "set_broadcast". Con questa modalità di fatto si ascoltano i pacchetti
    broadcast che transitano sul [broadcast domain](https://en.wikipedia.org/wiki/Broadcast_domain)
    al quale è collegata quella interfaccia di rete.  
    La libreria espone la funzione `datagram_net_listen(dev,udp_port)` che avvia una tasklet che si mette in ascolto in questo modo.  
    L'utilizzatore specifica il dev-name della propria interfaccia di rete e la porta UDP.
1.  Attendere connessioni su un socket unix-domain legato ad uno specifico pathname.  
    La libreria espone la funzione `stream_system_listen(pseudoip)` che avvia una tasklet che si mette in ascolto in questo modo.  
    L'utilizzatore specifica un indirizzo IP che il processo (che simula un nodo) finge di aver assegnato a se stesso
    o ad una sua precisa pseudo-interfaccia. Il pathname che viene associato al socket include tale pseudoip.
1.  Attendere messaggi su un socket unix-domain legato ad uno specifico pathname.  
    La libreria espone la funzione `datagram_system_listen(pseudodev)` che avvia una tasklet che si mette in ascolto in questo modo.  
    L'utilizzatore specifica un finto dev-name che il processo (che simula un nodo) finge che sia di
    una sua precisa pseudo-interfaccia. Il pathname che viene associato al socket include tale pseudodev.

L'utilizzatore può inizializzare la libreria ZCD ordinandogli di mettersi in ascolto con delle tasklet in una
o più di una di queste modalità.

Rimandiamo al documento ntkd-RPC nelle sezioni [Tipi di trasmissione](../DemoneNTKD/RPC.md#Trasmissioni)
e [Tipi di medium](../DemoneNTKD/RPC.md#Medium) per approfondire come il demone *ntkd*
usa queste modalità di ascolto.

## <a name="Basso_livello_lato_client"></a>Chiamate a metodi remoti al basso livello (lato client)

Le modalità sono:

*   **Stream**: Unicast. Con o senza `wait_reply`. Può essere per un vicino o per un nodo nel g-nodo comune di livello *i*.
    *   Nel dominio net si usa un indirizzo IP.
    *   Nel dominio sistema si usa un pathname che include:
        *   `pseudoip`. Se è un linklocal si assume sia univoco nell'ambito del contesto del sistema.
        *   Se è routabile, il `netid` del g-nodo comune, all'interno del quale `pseudoip` si assume sia univoco.
*   **Datagram**: Broadcast. Con o senza `ACK`. Si specifica un proprio NIC.
    *   Nel dominio net si usa un NIC reale.
    *   Nel dominio sistema si usa un pathname che include:
        *   `pid`. L'id del processo. Esso identifica univocamente il processo nell'ambito del sistema.
        *   `pseudonic`. Si assume sia univoco nell'ambito del processo.

Il protocollo da parte del client prevede:

*   Nome del metodo
*   Argomenti
*   ISourceID
*   IUnicastID o IBroadcastID
*   Identificazione che permetta al ricevente di individuare un suo arco/arco-identità

Il protocollo da parte del server prevede:

*   ...

Quando l'utilizzatore di ZCD vuole chiamare un metodo remoto lo può fare in 3 diverse modalità, per ognuna
delle quali va invocato un distinto metodo di ZCD. Le modalità di invio del messaggio messe a disposizione
da ZCD sono:

*   **TcpClient**: Con questa modalità il nodo chiamante, conoscendo l'indirizzo IP del nodo destinatario
    del messaggio, si connette a questo con il protocollo TCP che è reliable. Il destinatario è uno solo e
    può trovarsi dovunque all'interno della rete, poiché il messaggio viene instradato.  
    Con questa modalità, se il chiamante non riceve un errore è sicuro che il messaggio è giunto a
    destinazione correttamente.  
    Sebbene il messaggio giunge ad un solo nodo, nel messaggio stesso viene incluso un identificativo che
    indica il destinatario. Questo perché ZCD supporta le *identità* multiple in un nodo.  
    Questo identificativo, chiamato `unicast_id`, non è definito dalla libreria ZCD ma dal suo utilizzatore.
    Per la sua interpretazione la libreria ZCD si avvale dei delegati che gli sono stati passati al momento
    della sua inizializzazione.  
    Per lo stesso motivo, nel messaggio stesso viene incluso anche un identificativo che indica il
    mittente. Questo identificativo, chiamato `source_id`, non è definito dalla libreria ZCD ma dal suo
    utilizzatore. La sua eventuale interpretazione è demandata al *dispatcher*.
*   **Unicast**: Con questa modalità il nodo chiamante trasmette il messaggio su una delle sue interfacce
    di rete in broadcast con il protocollo UDP che è non-reliable.  
    Il chiamante può specificare opzionalmente l'indirizzo IP da usare come *source* nella
    trasmissione. **Nota:** Siccome un indirizzo IP linklocal è associato univocamente ad una interfaccia di
    rete, questa informazione specificata fa sì che il ricevente possa associare il messaggio ad un suo
    specifico arco, anche in presenza di eventuali diversi archi con lo stesso diretto vicino. Ma questa informazione
    (cioè lo specifico arco) che il ricevente desume dall'indirizzo IP (indicato come mittente nel pacchetto
    IP ricevuto) sarebbe meglio includerla come parte obbligatoria nel messaggio stesso: infatti in questo
    modo usando un diverso medium (ad esempio uno unix socket) che non contiene questo dettaglio si potrà
    fare comunque affidamento sullo stesso set di informazioni. **TODO** serve anche ad altro?  
    Il destinatario è uno solo e si suppone che si trovi direttamente collegato al chiamante
    sull'interfaccia di rete indicata, cioè è un diretto vicino. Pur essendo uno solo il destinatario il
    messaggio è trasmesso in broadcast e nel messaggio stesso viene incluso un identificativo che indica
    il nodo destinatario.  
    Questo identificativo, chiamato `unicast_id`, non è definito dalla libreria ZCD ma dal suo utilizzatore.
    Per la sua interpretazione la libreria ZCD si avvale dei delegati che gli sono stati passati al momento
    della sua inizializzazione.  
    Per lo stesso motivo visto prima, nel messaggio stesso viene incluso anche un identificativo che indica
    il mittente. Questo identificativo, chiamato `source_id`, non è definito dalla libreria ZCD ma dal suo
    utilizzatore. La sua eventuale interpretazione è demandata al *dispatcher*.  
    Sebbene il protocollo di trasmissione non garantisca la corretta ricezione del messaggio, tramite un
    meccanismo di risposte KEEPALIVE la libreria ZCD nel nodo chiamante sa determinare se l'operazione è
    in corso o se c'è stato un problema nella ricezione o durante l'elaborazione. Se si verifica un problema,
    l'utilizzatore di ZCD riceve una notifica. Altrimenti l'utilizzatore di ZCD riceve la risposta dal vicino,
    che può essere un valore di ritorno o una eccezione lanciata nel metodo remoto.  
    Questa modalità permette ad un nodo di comunicare con un diretto vicino anche quando questi nodi non
    hanno configurato le loro interfacce di rete con indirizzi e rotte concordate. Da questa feature deriva
    lo stesso nome della libreria, nella parte **zero configuration**.  
    Un esempio di utilizzo di questa possibilità nell'applicazione di Netsukuku è quando un nodo vuole
    comunicare con un diretto vicino ma ancora non ha formato un arco con lui.
*   **Broadcast**: Con questa modalità il nodo chiamante trasmette il messaggio su una delle sue interfacce
    di rete in broadcast con il protocollo UDP che è non-reliable.  
    Il chiamante può specificare opzionalmente l'indirizzo IP da usare come *source* nella
    trasmissione. **Nota:** Vedi sopra sulla descrizione di Unicast.  
    I destinatari sono tra i nodi vicini e possono essere più di uno. Nel messaggio stesso viene incluso un
    identificativo con il quale ogni nodo che lo riceve è in grado di capire se deve considerarsi tra i
    destinatari del messaggio.  
    Questo identificativo, chiamato `broadcast_id`, non è definito dalla libreria ZCD ma dal suo utilizzatore.
    Per la sua interpretazione la libreria ZCD si avvale dei delegati che gli sono stati passati al momento
    della sua inizializzazione.  
    Per lo stesso motivo visto prima, nel messaggio stesso viene incluso anche un identificativo che indica
    il mittente. Questo identificativo, chiamato `source_id`, non è definito dalla libreria ZCD ma dal suo
    utilizzatore. La sua eventuale interpretazione è demandata al *dispatcher*.  
    Un esempio di come siano usati questi identificativi nell'applicazione di Netsukuku può essere un messaggio
    che viene ricevuto da un nodo di una rete distinta da quella a cui appartiene il mittente, mentre il
    messaggio è destinato ai soli nodi della rete. Un altro esempio è un messaggio che ha come destinatari
    tutti i vicini tranne uno in particolare, come succede spesso con i messaggi ETP usati durante
    l'esplorazione della rete.  
    Sebbene il protocollo di trasmissione non garantisca la corretta ricezione del messaggio da parte di
    tutti i vicini, tramite un meccanismo di risposte ACK la libreria ZCD è in grado di comunicare al suo
    utilizzatore quali nodi, entro un certo lasso di tempo, abbiano notificato la corretta ricezione del
    messaggio. L'utilizzatore quindi può avvalersi di questa informazione per decidere il da farsi. Ad esempio
    nell'applicazione di Netsukuku, quando un messaggio di ETP trasmesso in modalità Broadcast non raggiunge
    tutti i nodi vicini di cui il nodo è a conoscenza allora il nodo cerca di inviarlo in modalità TcpClient
    ai nodi che non hanno risposto.
*   **UnixDomainStream**: Con questa modalità il nodo chiamante, conoscendo il path per il socket unix del nodo destinatario
    del messaggio (che termina per `_conn`), si connette a questo in modalità stream che è reliable. Il destinatario è uno solo.  
    Con questa modalità, se il chiamante non riceve un errore è sicuro che il messaggio è giunto a
    destinazione correttamente.  
    Sebbene il messaggio giunge ad un solo nodo, nel messaggio stesso viene incluso un identificativo che
    indica il destinatario. Questo perché ZCD supporta le *identità* multiple in un nodo.  
    Questo identificativo, chiamato `unicast_id`, non è definito dalla libreria ZCD ma dal suo utilizzatore.
    Per la sua interpretazione la libreria ZCD si avvale dei delegati che gli sono stati passati al momento
    della sua inizializzazione.  
    Per lo stesso motivo, nel messaggio stesso viene incluso anche un identificativo che indica il
    mittente. Questo identificativo, chiamato `source_id`, non è definito dalla libreria ZCD ma dal suo
    utilizzatore. La sua eventuale interpretazione è demandata al *dispatcher*.
*   **UnixDomainBroadcast**: Con questa modalità il nodo chiamante, conoscendo i path per i socket unix dei vari nodi destinatari
    del messaggio (che terminano tutti per `_broad`), trasmette il messaggio ad ognuno di questi con la modalità
    non-reliable.  
    I destinatari possono essere più di uno. Nel messaggio stesso viene incluso un
    identificativo con il quale ogni nodo che lo riceve è in grado di capire se deve considerarsi tra i
    destinatari del messaggio e per quali sue identità.  
    Questo identificativo, chiamato `broadcast_id`, non è definito dalla libreria ZCD ma dal suo utilizzatore.
    Per la sua interpretazione la libreria ZCD si avvale dei delegati che gli sono stati passati al momento
    della sua inizializzazione.  
    Per lo stesso motivo visto prima, nel messaggio stesso viene incluso anche un identificativo che indica
    il mittente. Questo identificativo, chiamato `source_id`, non è definito dalla libreria ZCD ma dal suo
    utilizzatore. La sua eventuale interpretazione è demandata al *dispatcher*.  
    Sebbene il protocollo di trasmissione non garantisca la corretta ricezione del messaggio da parte di
    tutti i nodi, tramite un meccanismo di risposte ACK la libreria ZCD è in grado di comunicare al suo
    utilizzatore quali nodi, entro un certo lasso di tempo, abbiano notificato la corretta ricezione del
    messaggio.

Per ognuno dei metodi suddetti l'utilizzatore passa le informazioni che sono necessarie a ZCD per l'invio
di una richiesta. Queste sono:

*   Il nome del metodo da chiamare. Per ZCD questa è semplicemente una stringa.  
    La libreria MOD-RPC può avere qualche ulteriore convenzione, ad esempio il nome della classe dello stub
    seguito da un punto e dal nome del metodo.
*   Un elenco di stringhe in formato JSON che rappresentano gli argomenti del metodo nell'ordine in cui
    sono nella sua signature. Per ZCD sono semplicemente stringhe valide nel formato JSON.  
    La libreria MOD-RPC ha avuto l'incarico di produrle serializzando dei dati e sarà in grado, dall'altro
    lato della comunicazione, di deserializzarle nella giusta forma.
*   Una stringa in formato JSON che rappresenta l'identificativo `source_id`. Anche in questo caso per la
    libreria ZCD è solo una stringa valida nel formato JSON.  
    La libreria MOD-RPC ha avuto l'incarico di produrla serializzando un oggetto e sarà in grado, dall'altro
    lato della comunicazione, di deserializzarla nella giusta forma.
*   Nel caso della modalità TcpClient:
    *   Una stringa in formato JSON che rappresenta l'identificativo `unicast_id`. Di nuovo, per la
        libreria ZCD è solo una stringa valida nel formato JSON.  
        La libreria MOD-RPC l'ha prodotta serializzando un oggetto e la deserializzerà.  
        La libreria ZCD da parte sua deve passare questa stringa al delegato. Questi lo
        deve saper interpretare per restituire un adeguato *dispatcher*.
    *   L'indirizzo del nodo da contattare. Si tratta di una stringa nel formato che può essere usato
        da un socket come indirizzo nell'apertura di una connessione TCP.
    *   La porta TCP.
*   Nel caso della modalità Unicast:
    *   Una stringa in formato JSON che rappresenta l'identificativo `unicast_id`. Valgono le osservazioni
        fatte sopra per l'oggetto `unicast_id`.
    *   La stringa del nome dell'interfaccia di rete da usare.
    *   La porta UDP.
    *   Opzionalmente, l'indirizzo IP da usare come *source*.
*   Nel caso della modalità Broadcast:
    *   Una stringa in formato JSON che rappresenta l'identificativo `broadcast_id`. Valgono le osservazioni
        fatte sopra per l'oggetto `unicast_id`. In particolare in questo caso il *dispatcher* potrebbe essere
        incaricato di notificare il messaggio a più *identità* del nodo<sup>1</sup>.
    *   La stringa del nome dell'interfaccia di rete da usare.
    *   La porta UDP.
    *   Opzionalmente, l'indirizzo IP da usare come *source*.
*   Nel caso della modalità UnixDomainStream:
    *   **TODO**
*   Nel caso della modalità UnixDomainBroadcast:
    *   **TODO**

## <a name="Interazioni_dei_livelli"></a>Interazioni dei 3 livelli

L'utilizzatore di ZCD inizializza come detto prima la libreria di basso livello indicando dove ascoltare.

A tutte le tasklet che stanno in ascolto, fornisce dei delegati che saranno usati per identificare un messaggio
e decidere se vada ignorato o preso in carico.

Il framework ZCD supporta il concetto di *identità* multiple in un nodo. Cioè ogni singolo nodo può assumere
diverse *identità* distinte.

I delegati che vengono passati alla libreria ZCD conoscono il cosidetto *identificativo di identità* che
individua ogni *identità* che il nodo ha assunto.

Così, quando ZCD riceve un messaggio dalla rete essi sono in grado di decidere se darlo in carico ad uno
specifico **dispatcher**<sup>1</sup>.

**TODO**: continuare le modifiche.

* * *

Quando vengono invocati i metodi che iniziano una chiamata a metodo remoto, la libreria ZCD nel nodo
corrente fa pervenire le informazioni alla libreria ZCD nel nodo (o nodi) destinatario. Qui la libreria
ZCD interroga i delegati passati dal suo utilizzatore per sapere se il messaggio è riconosciuto di sua
pertinenza. In questo caso il delegato fornisce alla libreria ZCD la funzione *f<sub>m</sub>* da chiamare
(sotto forma di una istanza di *dispatcher*, che è una interfaccia nota alla libreria ZCD) per processare
il messaggio *m*. La libreria ZCD allora predispone quanto necessario ai suoi compiti (ad esempio per la
trasmissione di un ACK, di periodici KEEPALIVE, eccetera) e poi chiama la funzione *f<sub>m</sub>* passando
il messaggio *m* e anche una struttura dati CallerInfo contenente informazioni sul messaggio ricevuto: il
`source_id`, il tipo di chiamata (TcpClient, Unicast o Broadcast), l'interfaccia di rete dove è stato
ricevuto, la porta, l'indirizzo riportato come mittente.

Al termine delle operazioni la funzione *f<sub>m</sub>* restituisce alla libreria ZCD una stringa in formato
JSON che rappresenta l'esito del messaggio, che può essere un valore di ritorno oppure una eccezione. Il
significato di tale stringa non è di competenza della libreria ZCD. Se la richiesta specificava di voler
attendere l'esito allora la libreria ZCD nel nodo destinatario del messaggio ora trasmette il risultato
alla libreria ZCD nel nodo chiamante e questa lo comunica al suo utilizzatore come valore di ritorno del
metodo che questi aveva chiamato all'inizio del giro.

* * *

<sub>Nota 1: Il dispatcher restituito dal delegato a fronte di un messaggio ricevuto è uno solo.  
Quando il messaggio è di tipo broadcast, il `broadcast_id` può identificare diversi destinatari. Può
succedere che più di uno di questi destinatari sia una *identità* di questo nodo. In questo caso il
dispatcher si prende cura di notificare il messaggio alle diverse *identità* di questo nodo che sono
interessate.  
Inoltre, in questo caso, se il messaggio prevede una comunicazione di ACK da ogni destinatario che
lo riceve, la libreria ZCD trasmette un unico ACK con un identificativo dal quale si possa risalire
all'interfaccia che ha ricevuto il messaggio.  
Chiariamo cosa sia questo identificativo. Sappiamo che i casi di messaggi broadcast sono quelli
ricevuti con il protocollo UDP su una specifica interfaccia di rete, oppure quelli ricevuti su uno unix socket
legato ad un path. Rispettivamente questi messaggi sono stati trasmessi da
altri nodi attraverso le modalità **Broadcast** e **UnixDomainBroadcast**, le quali si aspettano di
ricevere risposte di ACK. Nel caso di una specifica interfaccia di rete l'identificativo è
il MAC address dell'interfaccia che ha ricevuto il messaggio. Nel caso di uno unix socket
legato ad un path l'identificativo è una stringa dalla quale il programma è in grado di
risalire al path stesso.  
Dall'altra parte, il nodo che riceve questo ACK deve essere in grado di associarlo
all'intero set di *identità* di questo nodo che erano interessate.</sub>

### <a name="Interfacce_tra_livelli"></a>Interfacce tra i livelli

L'interfaccia della libreria ZCD nei confronti della libreria di livello intermedio MOD-RPC prevede
il passaggio di alcune strutture dati che la libreria ZCD non conosce. Tali dati vengono passati come
stringhe in formato JSON. Si tratta delle seguenti strutture:

*   Lista di argomenti da passare al metodo remoto. Passati da MOD-RPC a ZCD nel mittente e viceversa
    nel destinatario. Ogni elemento è una stringa in formato JSON.
*   Identificativo `source_id`. Passato da MOD-RPC a ZCD nel mittente e viceversa nel destinatario.
*   Identificativo `unicast_id`, per le chiamate unicast e tcp. Passato da MOD-RPC a ZCD nel mittente
    e viceversa nel destinatario.
*   Identificativo `broadcast_id`, per le chiamate broadcast. Passato da MOD-RPC a ZCD nel mittente e
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
    ad alto livello delle interfacce implementate dai metodi remoti si ottengono rapidamente delle classi
    da usare come stub e skeleton. In esse il lavoro di serializzazione e deserializzazione dei dati e i
    controlli di correttezza sono già stati fatti e testati; quindi si può passare ad occuparsi immediatamente
    degli algoritmi di alto livello (business logic).
*   Si può definire una versione stabile delle interfacce di comunicazione, realizzando ad esempio una libreria
    MOD-RPC versione 1.0. Qualsiasi versione di APP che si appoggia su questa libreria sarà quindi teoricamente
    in grado di comunicare con versioni differenti di APP che si appoggiano però sulla stessa libreria.

#### <a name="Interfaccia_modrpc_app_con_rpcdesgin"></a>Interfaccia tra MOD-RPC e APP come realizzata con "rpcdesign"

Le classi stub prodotte dal tool "rpcdesign" si occupano di serializzare ogni argomento per ogni metodo
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

Le classi skeleton prodotte dal tool "rpcdesign" si occupano di deserializzare ogni argomento per ogni metodo
remoto e poi di invocare il metodo astratto che sarà fornito dalla APP. Il metodo può restituire un valore
di ritorno o una eccezione. Se restituisce un valore la classe skeleton si occupa di serializzarlo. In
questa implementazione il nodo radice è sempre un nodo di tipo *oggetto* con un solo membro di nome
"return-value". Il suo contenuto è analogo a quanto visto per gli argomenti. Se invece va restituita una
eccezione questa viene serializzata con un nodo di tipo *oggetto* con un solo membro di nome "error".
Il suo contenuto è un oggetto con 3 membri di tipo stringa i cui nomi sono "error-domain", "error-code"
e "error-message" e i cui valori sono le rappresentazioni stringa dei relativi valori della struttura di
errore definita in Glib, ad esempio `"SomeDomainError", "MY_ERROR_NAME", "Richiesta non accettata"`.

Infine, le classi stub prodotte dal tool "rpcdesign" si occupano di deserializzare per ogni metodo il valore
di ritorno o l'eccezione. Il valore di ritorno viene restituito al chiamante. Oppure, l'eccezione viene
ricreata e lanciata.

Le classi stub e skeleton prodotte dal tool "rpcdesign" non fanno affidamento sui messaggi che ricevono
dalla rete. Fanno i dovuti controlli e se il contenuto dei messaggi non rispetta le convenzioni stabilite
(tipo degli argomenti, tipo del valore di ritorno, ...) non fanno crashare l'applicazione ma restituiscono
apposite eccezioni.

Le classi stub e skeleton prodotte dal tool "rpcdesign" realizzano una implementazione della libreria di
livello intermedio (cioè MOD-RPC) che verso la APP non lascia trasparire affatto l'uso della serializzazione
dei dati, tantomeno l'uso del formato JSON. Quindi APP può non avere alcuna cognizione della serializzazione
con il formato JSON. Le cose che APP deve conoscere sono le classi stub e skeleton fornite da MOD-RPC con i
loro metodi e le strutture dati che sono anche esse fornite da MOD-RPC.

E' possibile comunque che alcune strutture dati che utilizza APP e che vuole passare nei messaggi ai nodi
remoti non siano note alla libreria MOD-RPC. Consideriamo un metodo remoto la cui signature prevede come
argomento (o valore di ritorno) una istanza di GLib.Object o di una interfaccia che richiede GLib.Object.
Qualsiasi oggetto che deriva da Object può essere passato al metodo (o restituito da esso) purché sia in
grado di essere serializzato e deserializzato con il meccanismo fornito da `Json.gobject_serialize`. Rimandiamo
alla documentazione di JsonGlib per maggiori dettagli.

In questo caso la classe fornita da APP deve fare uso della libreria JsonGlib, quindi anche APP avrà una
dipendenza su questa libreria.

## <a name="Dettagli_tecnici"></a>Dettagli tecnici

I dettagli tecnici sono illustrati in questo [documento](ZcdDettagliTecnici.md).

