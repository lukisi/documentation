# Interdipendenze funzionali dei moduli

Come sono tra loro connessi i vari moduli dal punto di vista funzionale. Cioè
quali compiti assolvono, a uso di quale altro modulo; quali compiti demandano
a quali altri moduli; ecc.

### Il demone `ntkd`

Il programma vero e proprio. Coordina e gestisce i vari moduli.

Una particolare modalità di uso di questo programma è quella in cui esso gestisce le
reali interfacce di rete di un sistema e attraverso di esse comunica con diretti
vicini. In questo caso il programma è in esecuzione con i permessi di amministratore
del sistema.

Un'altra modalità di uso di questo programma è quella in cui esso gestisce dei socket
di sistema e attraverso di essi comunica con altri processi in una simulazione (o
test suite).

#### Comunicazioni di rete

Il compito di realizzare le effettive comunicazioni (mettersi in ascolto o trasmettere)
non viene affidato ai singoli moduli. Invece se ne occupa direttamente il demone `ntkd`.

Lo fa attraverso le classi `SkeletonFactory` e `StubFactory`. Anche altre parti di codice
sono a questo servizio, principalmente nei file che si trovano sotto la directory `rpc`.

### Il modulo `Neighborhood`

Il modulo `Neighborhood` ha il compito di realizzare e monitorare gli archi di un sistema con
i suoi diretti vicini.

Questi archi sono detti archi-nodo.

### Il modulo `Identities`

Il modulo `Identities` ha il compito di gestire il fatto che un singolo nodo può trovarsi
obbligato a impersonare diverse identità.  
In particolare questo modulo fa sì che gli altri moduli si occupino di comunicare attraverso
dei cosiddetti archi-identità.

### Il modulo `Hooking`

Il modulo `Hooking` viene messo a conoscenza che il nodo ha un arco-identità con un diretto
vicino. Il suo compito adesso è di comunicare attraverso questo arco e vedere se i nodi appartengono
alla stessa rete.

In caso positivo realizza gli archi-qspn e li fa avere al modulo `Qspn`.

In caso negativo fa in modo che uno dei due nodi (e tutta la rete a cui esso appartiene)
decida di entrare nella rete del vicino.

### Il modulo `Qspn`

Il modulo `Qspn` gestisce le comunicazioni di un nodo all'interno dei suoi gruppi (g-nodi) di
appartenenza per far sì che esso possa assegnarsi un indirizzo Netsukuku e
popolare correttamente le sue tabelle di routing per raggiungere gli altri indirizzi Netsukuku.

Il modulo `Qspn` conosce solo archi-identità che collegano il nodo ad altri nodi che
appartengono alla stessa rete.  
Un nodo appena nasce forma una rete a sè. È Lo stesso modulo `Qspn` che costituisce
questa nuova rete e gli dà un identificativo.

### Il modulo `PeerServices`

Il modulo `PeerServices` fa in modo che si possano implementare diversi tipi di servizi P2P
(*peer-to-peer*, *pari a pari*) in cui un preciso nodo possa sempre venire individuato come
responsabile di un certo compito.

Questa operazione d'individuazione del nodo responsabile viene fatta realizzando
un *hash-table distribuito* (DHT). Si basa sulla conoscenza del proprio indirizzo Netsukuku
e di quello di altri nodi esistenti nella rete; cioè si appoggia alle informazioni di
pertinenza del modulo `Qspn`.

### Il modulo `Coordinator`

Il modulo `Coordinator` implementa un servizio P2P per assegnare a un preciso nodo il compito
di prendere decisioni per conto di un gruppo (g-nodo) di qualsiasi livello.

Elenchiamo alcune di queste decisioni.

*   Individuare una posizione libera nel g-nodo.
*   Tenere traccia dei nuovi g-nodi che vogliono entrare in questo g-nodo.

### Il modulo `ANDNA`
