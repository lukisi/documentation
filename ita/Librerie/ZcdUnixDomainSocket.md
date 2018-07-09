# ZCD - Aggiunta delle modalità di comunicazione via unix domain socket

Si iniziano con questo commit delle modifiche al modulo ZCD.

Tali modifiche mirano a consentire l'uso di unix socket come medium delle comunicazione tra nodi (che in questo caso
sono distinti processi in un unico sistema). Questo con l'obiettivo di facilitare la produzione di testsuite per
i moduli di *ntkd*.

## Nuove modalità ascolto e trasmissione

La prima modifica a tale scopo è l'aggiunta di nuove modalità nel passaggio di messaggi tra i nodi.
Lato server ci sono nuove modalità con le quali un nodo (processo) ordina alla libreria ZCD di
mettersi in ascolto di messaggi/connessioni. Lato client nuove modalità con cui un altro nodo
chiede alla libreria ZCD di inviare messaggi/connessioni verso una certa destinazione.

Lato server finora si poteva:

1.  stare in ascolto su tutti i propri indirizzi IP con il protocollo TCP. Attraverso questa
    modalità si ricevono e si gestiscono sia le connessioni ai propri indirizzi IP linklocal
    che sono fatte da un nodo diretto vicino; sia le connessioni ai propri indirizzi IP routable
    che partono da un qualsiasi nodo in un proprio g-nodo.
1.  stare in ascolto su una propria interfaccia di rete con il protocollo UDP per pacchetti broadcast.
    Attraverso questa modalità si ricevono e si gestiscono sia i messaggi broadcast che sono trasmessi
    da un nodo diretto vicino; sia i messaggi unicast che sono trasmessi da un nodo diretto vicino
    nelle fasi precedenti alla creazione di un arco.

Adesso si aggiunge:

1.  stare in ascolto su un unix socket con un certo path in modalità connessione reliable.
1.  stare in ascolto su un unix socket con un certo path in modalità singolo messaggio non-reliable.

Analogamente lato client, finora si poteva:

1.  connettersi (**TcpClient**) ad un indirizzo IP. Poteva usarsi un indirizzo IP linklocal per
    comunicare con un nodo diretto vicino. Oppure un indirizzo IP routable (per il modulo PeerServices)
    per comunicare con un qualsiasi nodo in un proprio g-nodo.
1.  inviare un messaggio (**Unicast** e **Broadcast**) con un pacchetto UDP broadcast su una certa
    propria interfaccia di rete, trasmettendo in questo modo su uno specifico dominio broadcast.
    Così da raggiungere gli altri nodi diretti vicini che erano in ascolto su una loro interfaccia di rete.

Adesso si aggiunge:

1.  connettersi ad un unix socket su cui è in ascolto un altro nodo (processo).
1.  inviare un messaggio in modo non-reliable ad un unix socket su cui è in ascolto un altro nodo (processo).

## Informazioni desunte dal tipo di contatto

Un'altra modifica a tale scopo è che la libreria ZCD non dovrebbe passare al suo
utilizzatore alcuna informazione riguardo il mittente del messaggio.

In precedenza alcune informazioni sul mittente di un messaggio/connessione desunte per mezzo dei
dati che il Sistema Operativo associa ai internet socket venivano passate al delegato fornito
alla libreria ZCD e tramite esso al dispatcher che doveva gestire il messaggio.

Usando un unix socket queste informazioni non ci sono. Quindi per rendere trasparente l'uso
di un diverso medium occorre rimuoverle.

C'è da dire che queste informazioni erano comunque incerte.
In realtà quello che un server sa sul mittente di un pacchetto che riceve
è zero. Di sicuro sa solo che è stato "percepito" un pacchetto da una sua
certa interfaccia di rete. Da questo può desumere con certezza soltanto che
il messaggio è stato trasmesso da un qualche sistema che ha accesso al suo
stesso dominio di broadcast.

Tutte le altre informazioni sono in realtà desunte del contenuto stesso del
messaggio (indirizzo IP del mittente, MAC address del mittente, ordine del
pacchetto all'interno di uno stream, ...)

Tanto vale imporre che le informazioni sul mittente che servono al programma
(TODO verificare quali informazioni sono veramente indispensabili a *ntkd*)
vengano fornite come parte integrante del messaggio.
In questo modo si ottiene che passando da un medium ad un altro (ad esempio
passando dai internet socket ai unix socket per realizzare una testsuite)
la logica del programma resta la stessa.

## Altre modifiche

Intendiamo rimuovere del tutto la modalità **Unicast**. Con questa si realizzava il dialogo
con i nodi diretti vicini prima di aver costruito un arco. Si era già a conoscenza del
*unicastid* del nodo diretto vicino interessato, ma non del suo indirizzo IP linklocal.  
L'unico punto in cui questa modalità era usata è nel modulo Neighborhood nelle operazioni
di costruzione di un arco.

Abbiamo una modalità di dialogo con i nodi di cui conosciamo l'indirizzo IP, la **TcpClient**.  
Questa modalità viene usata per dialogare con uno specifico nodo all'interno di un g-nodo
comune al nostro nodo. In questo caso il nostro nodo è in qualche modo venuto a conoscere
l'indirizzo IP routable univoco nel g-nodo del nodo destinatario.  
La medesima modalità viene usata anche per dialogare con un nodo diretto vicino, dopo che
è stato costruito un arco. Infatti in questo momento siamo a conoscenza dell'indirizzo
IP linklocal del nodo diretto vicino.

La stessa è simulata (nel caso dell'uso di unix socket) con un unix socket in modalità connessione
reliable, indicata sotto con **UnixDomainStream**.

Si ha inoltre una modalità per trasmettere messaggi broadcast
a tutti i propri diretti vicini su un dominio broadcast, cioè **Broadcast**.

La stessa è simulata (nel caso dell'uso di unix socket) con un unix socket che trasmette
messaggi in modo non-reliable, indicata sotto con **UnixDomainBroadcast**.

Dovrebbe risultare facile l'implementazione delle fasi necessarie alla costruzione di un arco
tramite l'uso di soli messaggi broadcast.

## Dettagli tecnici

1.  **UnixDomainStream** È l'equivalente del **TcpClient**.  
    Si usa in 2 modi:  
    1) Lato server il nodo (processo) viene istruito ad ascoltare su un numero
    di path con nome `xxx_n_conn` per connessioni. Ognuno di questi path rappresenta un indirizzo linklocal
    che questo nodo ha assegnato ad una sua interfaccia di rete. Lato client
    un nodo vicino (un altro processo) che ha un arco (conosce il path `xxx_n_conn`) si connette al socket
    in ascolto e comunica in modo reliable.  
    2) Lato server il nodo (processo) viene istruito ad ascoltare su un numero
    di path con nome `xxx_r_conn` per connessioni. Ognuno di questi path rappresenta un indirizzo proprio di
    questo nodo che è routable all'interno di un g-nodo. Lato client un altro nodo (un altro processo) nello
    stesso g-nodo che conosce l'indirizzo (conosce il path `xxx_r_conn`) si connette al socket
    in ascolto e comunica in modo reliable.
1.  **UnixDomainBroadcast** È l'equivalente del **Broadcast**.  
    Lato server il nodo ascolta su un numero di path con nome `xxx_n_broad` per
    messaggi. Ognuno di questi path rappresenta una interfaccia di rete del nodo. Lato client un nodo vicino
    (un altro processo) che ha una interfaccia di rete collegata allo stesso dominio broadcast (conosce cioè
    tutti i path `xxx_n_broad` degli altri processi che simulano una propria interfaccia di rete collegata
    a quell'unico dominio broadcast) invia a tutti i socket in ascolto un messaggio in modo non-reliable.

Con l'uso di **UnixDomainBroadcast** a simulare le interfacce di rete collegate ad un certo dominio
broadcast si dovrebbe riuscire a simulare le possibili caratteristiche di una radio: in particolare il fatto
che se un nodo *B* con una interfaccia di rete è diretto vicino di *A* e di *C* questo non implica
necessariamente che i nodi *A* e *C* siano fra loro diretti vicini.  
Questa situazione non era facilmente simulabile con una serie di macchine virtuali collegate a segmenti
di rete virtuali.

