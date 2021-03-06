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

Il modulo `Neighborhood` segnala al demone `ntkd` quando nasce un arco-nodo.

### Il modulo `Identities`

Il modulo `Identities` ha il compito di gestire il fatto che un singolo nodo può trovarsi
obbligato a impersonare diverse identità.  
In particolare questo modulo fa sì che gli altri moduli si occupino di comunicare attraverso
dei cosiddetti archi-identità.

Il demone `ntkd` indica al modulo `Identities` quando è stato creato un arco-nodo.  
In questo momento, siccome esiste sempre per lo meno l'identità principale nel sistema,
viene sicuramente creato un arco-identità. Il modulo `Identities` segnala al demone `ntkd`
quando nasce un arco-identità.

### Il modulo `Hooking`

Il demone `ntkd` indica al modulo `Hooking` quando è stato creato un arco-identità.  
Il modulo `Hooking` ha adesso il compito di comunicare attraverso questo arco e vedere
se i nodi appartengono alla stessa rete.

In caso positivo emette il segnale `same_network`. A fronte di questo il demone `ntkd`
realizza un arco-qspn e lo fa avere al modulo `Qspn`.

In caso negativo emette il segnale `another_network`. Poi fa in modo che uno dei
due nodi decida di entrare nella rete del vicino (e con lui tutta la rete a cui
esso appartiene). La decisione presa sarà comunicata al demone `ntkd` per mezzo dei
segnali `do_prepare_enter`, ...

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
*   Coadiuvare il modulo `Hooking` nelle sue funzioni (fare da proxy verso
    il nodo Coordinator per alcuni metodi, propagare messaggi all'interno del
    proprio g-nodo, ...).

### Il modulo `ANDNA`
