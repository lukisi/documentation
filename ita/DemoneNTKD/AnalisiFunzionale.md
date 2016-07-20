# Demone NTKD - Analisi Funzionale

1.  [Definizione di rete Netsukuku](#Definizione_di_rete_netsukuku)
1.  [Funzionamento di una rete Netsukuku](#Funzionamento_di_rete_netsukuku)
    1.  [Esplorazione della rete](#Esplorazione_della_rete)
        1.  [Tabelle di routing di una classica rete TCP/IP](#Tabelle_di_routing_classica)
        1.  [Tabelle di routing di una rete mesh](#Tabelle_di_routing_mesh)
        1.  [Informazioni raccolte da un nodo Netsukuku](#Informazioni_nodo_netsukuku)
    1.  [Stack TCP/IP](#Stack_tcpip)
    1.  [Associazione nomi - indirizzi](#Associazione_nomi_indirizzi)
1.  [Architettura modulare di NTKD](#Architettura_modulare_ntkd)

Il demone *ntkd* deve essere sempre in esecuzione su ogni singolo nodo che compone una rete Netsukuku. Gli obiettivi
che si prefigge sono:

*   Rilevare i vicini. Comunicare con loro per implementare un algoritmo distribuito di esplorazione della rete.
*   Popolare e mantenere le rotte di pertinenza di Netsukuku nelle tabelle di routing dello stack TCP/IP del nodo.
*   Fornire i servizi di base per il funzionamento dei Servizi Distribuiti, opzionali o obbligatori (aka Peer-to-peer Services).
*   Fornire ai client un servizio di interrogazione del database distribuito dei nomi (ANDNA).

## <a name="Definizione_di_rete_netsukuku"></a>Definizione di rete Netsukuku

Una rete Netsukuku si forma automaticamente quando alcuni nodi, su cui è in esecuzione il demone *ntkd*, risultano
fra loro *collegati* attraverso un qualsiasi mezzo, come dei cavi collegati ad uno switch, o delle radio Wi-Fi a
distanza di rilevamento. Questi nodi possono essere dei computer, o degli access point, o altri  dispositivi sui
quali, come già detto, deve essere in esecuzione il  demone *ntkd*.

Una rete esiste, come entità di insieme, fin quando risulta completamente connessa, cioè dati due nodi qualsiasi
esiste un *percorso* che li unisce costituito da nodi tra loro collegati.

Immaginiamo due gruppi di nodi, chiamiamoli *A* e *B*, tali che:

*   per ogni nodo del gruppo *A* esiste almeno un percorso che lo unisce a qualsiasi altro nodo del gruppo *A*;
*   analogamente, per ogni nodo del gruppo *B* esiste almeno un percorso che lo unisce a qualsiasi altro nodo
    del gruppo *B*;
*   per nessun nodo del gruppo *A* esiste alcun percorso che lo unisca ad alcun nodo del gruppo *B*; si può dire
    anche così: nessun nodo del gruppo *A* ha un collegamento diretto con un nodo del gruppo *B*; questo implica
    la stessa cosa da *B* verso *A*.

I gruppi A e B sono "isole" Netsukuku. Fin quando non si incontrano, essi costiuiscono due reti Netsukuku distinte
e separate. Allo stesso modo se la rete A, che si era formata connessa, diventa disconnessa e si formano due gruppi
questi gruppi diventano due isole.

Nel resto del documento, quando parliamo di "la rete Netsukuku" di cui fa parte il nodo *n*, ci riferiamo ad una
rete connessa a cui il nodo *n* "appartiene"; cioè il nodo *n* è direttamente collegato ad almeno un altro nodo
di quella rete.

## <a name="Funzionamento_di_rete_netsukuku"></a>Funzionamento di una rete Netsukuku

Un nodo della rete Netsukuku deve essere configurato inizialmente con queste informazioni:

*   Quali sono le interfacce di rete attraverso le quali il nodo è collegato (o può in futuro collegarsi) ad
    altri nodi della rete Netsukuku.
*   Con quale nome (o nomi) vuole essere identificabile dagli altri nodi all'interno della rete Netsukuku.

Inoltre il nodo deve avere configurato come servizio sempre in esecuzione il demone *ntkd* .

Una volta soddisfatti questi requisiti il nodo è parte della rete Netsukuku. Ciò significa che è in grado di
contattare o di essere contattato da qualsiasi altro nodo nella rete Netsukuku. Il demone *ntkd* esplora la
rete Netsukuku, fornisce le informazioni rilevanti allo stack TCP/IP del S.O. e usa il sistema di nomi ANDNA
per registrare i suoi nomi e risolvere i nomi degli altri nodi.

**Attenzione:** Un nodo che è parte di una rete Netsukuku può fare allo stesso tempo parte anche di un'altra
rete TCP/IP privata o di Internet. Le due cose non sono tra loro incompatibili. Invece, un nodo non può far
parte contemporaneamente di due reti Netsukuku distinte. Infatti, quando due reti distinte si incontrano viene
a formarsi una unica rete.

### <a name="Esplorazione_della_rete"></a>Esplorazione della rete

L'algoritmo distribuito di esplorazione della rete ha obiettivi che vanno oltre le finalità di un classico sistema
di routing nelle reti TCP/IP. Per capirlo meglio, analiziamo dapprima quali informazioni siano necessarie ad un
nodo di una classica rete TCP/IP. Vediamo poi quali differenze si presentano in una rete di tipo mesh. Infine
descriveremo quali informazioni ottiene ogni singolo nodo di una rete Netsukuku dall'esecuzione dell'algoritmo
distribuito di esplorazione.

#### <a name="Tabelle_di_routing_classica"></a>Tabelle di routing di una classica rete TCP/IP

Molto spesso, in un nodo terminale di una classica rete TCP/IP abbiamo una sola interfaccia di rete attiva, che
può essere la scheda ethernet attraverso la quale siamo collegati con un cavo, oppure la scheda radio attraverso
la quale siamo connessi ad un access point. In entrambi i casi, attraverso questa interfaccia abbiamo un nostro
indirizzo IP e nelle tabelle di routing una rotta diretta verso un numero di indirizzi che possono essere assegnati
a nodi che sono connessi allo stesso modo alla nostra cosidetta LAN. Tra questi nodi, uno in particolare viene
definito *default gateway* , perché è attraverso di lui che possiamo raggiungere il resto della rete, ad esempio
Internet o la rete aziendale. Abbiamo, cioè, nelle tabelle di routing una rotta che dice di passare per l'indirizzo
assegnato a quel gateway per raggiungere la classe di indirizzi che rappresenta l'intera rete.

Molto più raramente, un nodo terminale di una rete TCP/IP potrebbe avere due interfacce di rete entrambe attive, di
solito una ethernet e una radio. Per ognuna di esse abbiamo un indirizzo IP (usato di preferenza come indirizzo
sorgente per i messaggi che partono da questo nodo e escono da questa interfaccia) e nelle tabelle una rotta diretta
per una classe di indirizzi. Siamo cioè collegati a due LAN. Potremmo avere su ogni LAN un gateway (raramente più
di uno). Ad esempio, siamo collegati alla rete aziendale attraverso un cavo di  rete, e anche ad un access point
tramite la scheda radio. Attraverso  l'interfaccia cablata vediamo la LAN del nostro ufficio; in particolare  un
nodo di questa LAN ci serve come gateway per raggiungere dei server  che sono in altri uffici della stessa azienda.
Però siccome non possiamo  accedere a Internet attraverso questa rete aziendale, abbiamo un access  point che ci
collega ad un Internet Provider. In questo caso abbiamo 2  indirizzi propri e 4 rotte nelle tabelle. Per esempio,
192.168.100.14  come indirizzo preferito per la scheda ethernet, una rotta diretta  attraverso la scheda ethernet
per la classe 192.168.100.0/24 e una rotta  per la classe 192.168.0.0/16 che passa per il gateway 192.168.100.1
nell'interfaccia ethernet. Poi abbiamo 192.168.2.101 come indirizzo  preferito per la scheda radio, una rotta diretta
attraverso la scheda  radio per la classe 192.168.2.0/24 e una rotta per la classe di  indirizzi dell'intera rete,
la 0.0.0.0/0, che passa per il gateway  192.168.2.1 nell'interfaccia radio.

In questo caso il nostro nodo è collegato a due LAN. Ma gli altri nodi delle due LAN non sanno dell'esistenza della
LAN in cui non si trovano. Il nostro nodo non viene usato come gateway da una LAN verso l'altra.

Questo scenario è un po' più complesso del solito. Affrontare con il solito approccio questa maggiore complessità
pone alcune limitazioni. Nel caso descritto sopra, la rete aziendale vuole pieno controllo degli indirizzi nella
classe 192.168.0.0/16. Probabilmente sono troppi, ma per facilità attua questa politica. L'access point del provider
di Internet vuole il controllo della classe 192.168.2.0/24. Questo comporta che eventuali nodi della rete aziendale
con indirizzi in quest'ultima classe non saranno raggiungibili dal mio nodo terminale collegato alle due LAN.

Iniziamo ora ad esaminare i nodi router di una classica rete TCP/IP. Cioè quei nodi che hanno almeno due interfacce
di rete collegate a due LAN distinte e che sono in grado di smistare i messaggi che devono passare dall'una all'altra.

Un router con *n* interfacce di rete connesse ad altrettante LAN, ha di norma *n* indirizzi propri, ognuno nello
spazio di indirizzi assegnato ad una LAN a cui è connesso. Ha inoltre *n* rotte dirette nel kernel, ognuna delle
quali dice di poter raggiungere direttamente attraverso una certa interfaccia tutti gli indirizzi della classe
assegnata alla LAN a cui l'interfaccia è connessa. Poi ha un insieme *R* di altre rotte. Questo insieme può essere
piccolo o anche molto grande. Ogni rotta *r* ∈ *R* dice che un messaggio destinato ad un indirizzo appartenente
alla classe *d* può essere consegnato ad un certo gateway *gw* attraverso l'interfaccia *nic* .

Le conoscenze di un router di una classica rete TCP/IP gli permettono, date le informazioni contenute in un
pacchetto *p*, di individuare la regola *r* che meglio lo inquadra e così di inoltrarlo attraverso uno dei suoi gateway.

Queste informazioni sono carenti in quanto di norma non consentono di scegliere fra più possibili alternative. Dato
un indirizzo di destinazione è sempre una sola la rotta che il pacchetto dovrà prendere. Se la destinazione potesse
essere effettivamente raggiunta attraverso diversi percorsi dingiunti, questa carenza porta ad uno sfruttamento non
ottimale delle risorse.

Queste informazioni sono anche carenti nella conoscenza del percorso oltre al primo hop, cioè oltre il proprio
gateway. Le informazioni nella tabella di routing, cioè, non dicono quanti e quali altri router andranno percorsi
per raggiungere la destinazione.

Nonostante la scarsità di informazioni mantenute, le dimensioni della tabella di routing di un router del core di
Internet sono ad oggi un [problema](https://en.wikipedia.org/wiki/Border_Gateway_Protocol#Routing_table_growth).
Il protocollo di routing standard usato da questi router per gestire le interconnessioni tra i vari Autonomous
Systems che costituiscono l'intera Internet è il BGP. Nell'ultimo anno (agosto 2014) il numero di rotte che tale
protocollo ha inserito in alcuni router ha superato le 500000.

I protocolli di routing, come il BGP appena citato, cercano di esplorare in modo dinamico la rete, al fine di
rendere possibile lo sfruttamento dei migliori percorsi. Questi protocolli però sono usati solo ai livelli "alti"
della rete Internet, nella sua dorsale. Nelle parti più periferiche, quelle che sono più direttamente percepite
dall'utenza finale, come anche nelle reti TCP/IP di piccole dimensioni come le intranet aziendali, si usa di norma
un approccio statico e centralizzato. Gli indirizzi sono assegnati in modo statico da un ente di controllo centrale
e anche le rotte sono fisse. Per rendere queste ultime facilmente mantenibili si adotta una topologia di rete
cosidetta a stella.

Si tratta di una topologia fortemente gerarchica. Quando un nodo vuole comunicare con un altro, fosse anche molto
vicino a lui, è  costretto a seguire un percorso verso il nodo   centrale della stella, per poi allontanarsene
verso il vertice destinazione.

#### <a name="Tabelle_di_routing_mesh"></a>Tabelle di routing di una rete mesh

L'obiettivo che si propone una rete mesh è che ogni  nodo che vuole raggiungere una destinazione sia in grado di
sfruttare il  miglior percorso fisicamente fruibile nella rete, senza essere  costretto a seguire un percorso
altamente gerarchico.

Un  altro obiettivo desiderabile è che ogni nodo sia in grado di  raggiungere qualsiasi altro indirizzo
appartenente alla rete; cioè non  ci devono essere porzioni di rete che vengono nascoste (NATtate) dietro un
unico indirizzo pubblico e che quindi sarebbero in grado di operare  solo come iniziatori di connessione (client)
verso altri nodi con un  indirizzo pubblico.

Spesso, gli indirizzi dei singoli nodi di una rete mesh vengono  assegnati in modo statico. Anche le varie tabelle
di routing potrebbero  in teoria venire popolate manualmente una volta per tutte, ma questo  raramente accade e si
presta solo a reti molto piccole. Di norma il  popolamento delle tabelle di routing è l'obiettivo di un software che
implementa un routing protocol.

In una rete mesh non esiste il concetto di nodo terminale. Ogni singolo nodo può operare da gateway per un suo vicino
che gli trasmette un pacchetto per una qualsiasi destinazione.

Consideriamo un nodo che ha più interfacce di rete tramite le quali è connesso ad altri nodi. Di norma in una rete
mesh questo nodo non associa a una interfaccia un'intera classe di indirizzi. Invece memorizza nelle tabelle di routing
una singola rotta diretta per l'indirizzo di ogni vicino di cui viene a conoscenza attraverso il protocollo di routing.
Nelle reti mesh il concetto di LAN, inteso come classe di indirizzi associata ad una interfaccia di rete, perde di significato.

Sempre attraverso il protocollo di routing, il nodo viene a conoscenza di altre informazioni sulla rete e questo gli
permette di popolare le tabelle di routing con altre rotte.

Anche in questo caso le rotte sono costituite da poche informazioni: dato un messaggio per una destinazione *d*
esso va consegnato al gateway *gw* attraverso l'interfaccia *nic* . Questo anche se le informazioni raccolte
attraverso il protocollo di routing potrebbero essere più dettagliate.

#### <a name="Informazioni_nodo_netsukuku"></a>Informazioni raccolte da un nodo Netsukuku

Ogni nodo della rete Netsukuku raccoglie (e mantiene aggiornate) informazioni circa ogni destinazione presente
nella rete. Per destinazione qui si intende un indirizzo effettivamente assegnato ad un nodo al momento presente,
oppure una classe di indirizzi al cui interno almeno un indirizzo è assegnato al momento presente.

Per risolvere il problema della dimensione della tabella di routing anche nella rete Netsukuku, come avviene nella
rete Internet, si raggruppano gli indirizzi in classi a diversi livelli. L'automazione di questo procedimento di
agglomerazione lo rende molto più efficiente delle tecniche usate attualmente nella rete Internet. Ad esempio, usando
gli indirizzi IPv4 della classe 10.0.0.0/8, una rete Netsukuku può contenere fino a 2<sup>22</sup> nodi (4 milioni)
e ogni nodo dovrà memorizzare informazioni per un massimo di 36 destinazioni.

Quali informazioni sono raccolte dal singolo nodo per ogni destinazione? Ogni nodo per ogni destinazione viene a
conoscenza di un numero (da 1 fino ad un massimo prefissato nel codice) di possibili percorsi per raggiungerlo.
Ricordiamo che ogni destinazione può essere una classe di indirizzi; il nodo viene a conoscenza anche di una
approssimazione del numero di indirizzi che al suo interno sono effettivamente assegnati a nodi presenti. Anche
altre informazioni sono portate a conoscenza del nodo, che però sono solo funzionali alla corretta esplorazione
della rete e non vengono rese note all'esterno del demone *ntkd* .

Per ogni singolo percorso per raggiungere una destinazione, quali informazioni ha il nodo? Il nodo conosce il
gateway *gw* a cui può consegnare, attraverso l'interfaccia *nic* , un messaggio che ha quella destinazione. Conosce
anche il costo del percorso, il quale può essere espresso in diverse metriche, come latenza o larghezza di banda.

Conosce infine ogni singolo passo del percorso fino alla destinazione. Tenendo di nuovo in considerazione la quantità
di informazioni che sono memorizzate, per singolo passo si intende non necessariamente un indirizzo, ma di solito una
classe di indirizzi.

Per ogni passo di un percorso il nodo conosce, oltre alla classe di indirizzi che costituisce quel passo, anche un
identificativo univoco della specifica interfaccia di rete del nodo che collega quel passo al passo successivo.
Tale informazione viene chiamata *arco* .

In conclusione, ogni percorso verso la destinazione *d* può essere distinto da un altro percorso verso *d* per il
gateway usato (*gw*, *nic*), oppure per la sequenza di passi, oppure anche solo per uno degli archi che collegano
un passo al successivo.

### <a name="Stack_tcpip"></a>Stack TCP/IP

Il software nella suite Netsukuku non intende sostituire lo stack TCP/IP implementato nel S.O. del nodo. Nemmeno
intende interagire con le librerie TCP/IP del S.O. attraverso sistemi di hook.

Quando un'applicazione vuole avviare una connessione TCP con un indirizzo, o inviare un pacchetto di dati UDP ad un
nodo, o mettersi in ascolto, o qualsiasi altra operazione che coinvolge la rete TCP/IP, questi eventi non sono in
alcun modo notificati al software Netsukuku. Sono invece completamente gestiti alla maniera classica dalle librerie
del S.O. del nodo.

Infatti non è in questo momento che il software Netsukuku intende operare, ma preventivamente. Le sue operazioni
mirano a popolare in anticipo le tabelle di routing del nodo con le rotte di sua pertinenza e mantenerle aggiornate.
Si definiscono rotte di pertinenza di Netsukuku tutte le rotte necessarie a che ogni singolo nodo esistente nella
rete Netsukuku possa essere raggiunto e che ogni indirizzo nello spazio di Netsukuku che non è stato assegnato ad
alcun nodo sia riportato come "irraggiungibile".

Il demone *ntkd* mantiene le rotte di pertinenza di Netsukuku nelle tabelle di routing. Decide quali rotte impostare
sulla base delle informazioni che ha raccolto tramite l'esplorazione della rete.

Per ogni destinazione, o classe di indirizzi *d* di cui è venuto a conoscenza, considerando l'insieme *P* dei percorsi
che sa di poter usare per raggiungerla, il demone *ntkd* mantiene nelle tabelle di routing:

*   La rotta ( *gw* + *nic* + *d* ) che riporta i dati relativi al miglior percorso *p* ∈ *P* , cioè quello con costo
    minore.
*   Per ogni vicino *v* del nodo:
    *   La rotta con i dati del miglior percorso *p<sub>v</sub>* ∈ *P* che non ha fra i suoi passi la classe di
        indirizzi a cui appartiene *v* . Questa sarà usata per i pacchetti che il nodo ha ricevuto dal vicino *v* e
        deve inoltrare alla destinazione *d* .

### <a name="Associazione_nomi_indirizzi"></a>Associazione nomi - indirizzi

La suite di software Netsukuku offre un meccanismo di risoluzione dei nomi degli host chiamato ANDNA (A Netsukuku
Domain Name Architecture).

ANDNA si propone come meccanismo alternativo o in aggiunta al classico DNS. Tra i due sistemi si evidenziano queste
differenze:

 * DNS è un sistema centralizzato. ANDNA è distribuito.
    *   Nel sistema DNS esiste un numero di server centrali, i root-nameserver. Solo questi sono autoritativi,
        senza di essi il sistema non può funzionare. Infatti, per risolvere il problema del Single Point Of Failure
        e assicurare la ridondanza, questi server sono molti e vanno tra loro coordinati. Una unica autorità centrale
        statunitense, ICANN, controllata dal Dipartimento del Commercio statunitense, gestisce le informazioni che
        sono su questi server.  
        I root-nameserver decidono quali siano i domini di più alto livello validi (.com, .edu, .it, ...) e per
        ognuno indicano quali altri server siano autoritativi per i nomi appartenenti a quel dominio.
    *   Nel sistema ANDNA non esistono server basilari. Ogni singolo nodo può essere chiamato a memorizzare una parte
        delle informazioni che costituiscono la mappatura nomi-indirizzi.  
        Il funzionamento del sistema è tale per cui viene assicurata ridondanza.  
        Le modifiche alla topologia della rete di fatto cambiano con frequenza il nodo che viene individuato come
        autoritativo per un certo nome, rendendo molto arduo un attacco.
 * DNS è un sistema fortemente gerarchico. ANDNA è flat.
    *   Nel sistema DNS i server centrali demandano la gestione di un dominio top-level (ad esempio .it) ad altri
        server. Questi a loro volta dicono quali sotto-domini esistono all'interno di .it e demandano ad altri server
        la gestione di ogni sotto-dominio. E così via.  
        Per ogni livello esiste quindi una gestione, cioè una autorità che ha il controllo. Di conseguenza una serie
        di adempimenti (almeno burocratici, spesso economici) che chi vuole registrare un nome di host o un dominio
        deve fare.
    *   Nel sistema ANDNA un nome di host è una sequenza di caratteri validi (lettere case-insensitive, numeri, pochi
        altri simboli) con una lunghezza massima prefissata.  
        Se un nome non è stato già assegnato, qualunque nodo può registrarlo per se nel database distribuito.
 * ANDNA reagisce molto più rapidamente di DNS agli aggiornamenti di una associazione nome-indirizzo.
    *   Nel sistema DNS se ogni singola richiesta fosse inviata ai server autoritativi questo comporterebbe un carico
        di lavoro troppo pesante per i root-nameservers e quindi un collo di bottiglia.  
        Per questo il sistema DNS prevede la presenza di numerosi *caching* DNS server. Questi memorizzano le passate
        ricerche e quando interrogati rispondono direttamente sulla base della loro cache. Soltanto periodicamente vanno
        ad aggiornare i record nella loro cache interrogando i server autoritativi.  
        Spesso ogni Internet Service Provider predispone un caching server che fornisce il servizio ai suoi clienti.  
        Questo comporta un ritardo, spesso di alcune ore o giorni, tra il momento in cui viene apportata una variazione
        ad una associazione nome-indirizzo e il momento in cui tutti i nodi di Internet vedono il nuovo indirizzo.
    *   Essendo ANDNA fortemente distribuito, il problema del collo di bottiglia presso i server autoritativi è
        drasticamente ridotto. Sono comunque previsti dei meccanismi di caching, soprattutto per migliorare le
        prestazioni del sistema, ma i tempi di aggiornamento delle cache possono essere ridotti di molto.  
        Questo comporta che al variare di una associazione nome-indirizzo, dopo tempi molto brevi possiamo presumere
        che tutti i nodi della rete vedranno il nuovo indirizzo.  
        Questa caratteristica, d'altra parte, è necessaria in una rete Netsukuku. Infatti in tale rete, poiché non ha
        bisogno di una entità centrale di coordinamento, ogni nodo può essere obbligato a cambiare il suo indirizzo
        in qualsiasi momento senza che questo comporti alcun intervento manuale di un amministratore.

Ogni singolo nodo della rete Netsukuku può cercare di registrare per se stesso un numero illimitato di nomi.
L'algoritmo distribuito consente ad ogni singolo nodo di vedere accettata la registrazione di un totale di 256 nomi
che non siano già stati assegnati ad altri nodi.

Quando un nodo ottiene la registrazione di un nome, questa rimane attiva per 30 giorni. Fino a quando la registrazione
è attiva nessun altro nodo potrà registrare per se lo stesso nome. La registrazione può essere aggiornata dal nodo
che la detiene, il quale si autentica al sistema distribuito ANDNA attraverso un meccanismo di cifratura asimmetrica
(chiave pubblica / chiave privata). Quindi il conteggio dei 30 giorni riparte da capo.

**Attenzione:** Un nodo può essere parte di una rete Netsukuku e fare allo stesso tempo parte anche di un'altra rete
TCP/IP privata o di Internet. In questo caso, oltre a usare il sistema ANDNA nella rete Netsukuku, può continuare ad
usare sull'altra rete il sistema DNS.

Tutto il back-end di ANDNA (la registrazione dei nomi, l'applicazione delle politiche delineate sopra, la ricerca dei
nomi) è realizzato attraverso due servizi peer-to-peer chiamati Andna e Counter. La spiegazione del loro funzionamento
non rientra nell'ambito di questo documento.

Il front-end di ANDNA, cioè quello che succede quando una applicazione chiede una informazione al cosidetto
*resolver* dei nomi del S.O. del nodo su cui è in esecuzione, si compone di due parti: una client e una server.

La parte server è realizzata dallo stesso demone *ntkd*. Esso riceve le richieste dalla parte client e risponde dopo
aver interrogato il back-end.

La parte client può essere interpretata da diversi moduli software, i quali si interfacciano con le applicazioni in
diversi modi a seconda dei meccanismi messi a disposizione dal sistema operativo. I moduli client forniti dalla suite
Netsukuku (`lilbnss_andna`, `dns-to-andna`, `ntk-resolv`) sono descritti in dettaglio in un altro documento.

## <a name="Architettura_modulare_ntkd"></a>Architettura modulare di NTKD

**TODO**
