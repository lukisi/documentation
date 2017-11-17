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

