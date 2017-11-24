# Demone NTKD - Analisi Funzionale

1.  [Ruolo del demone ntkd](#Ruolo_ntkd)
    1.  [Definizione di rete Netsukuku](#Definizione_di_rete_netsukuku)
    1.  [Funzionamento di una rete Netsukuku](#Funzionamento_di_rete_netsukuku)
        1.  [Esplorazione della rete](#Esplorazione_della_rete)
        1.  [Assegnazione delle rotte nel kernel](#Assegnazione_rotte)
        1.  [Associazione nomi - indirizzi](#Associazione_nomi_indirizzi)
1.  [Architettura modulare di ntkd](#Architettura_modulare_ntkd)
    1.  [Modulo Neighborhood](#Modulo_Neighborhood)
    1.  [Modulo Identities](#Modulo_Identities)
    1.  [Modulo Qspn](#Modulo_Qspn)
    1.  [Modulo PeerServices](#Modulo_PeerServices)
    1.  [Modulo Coordinator](#Modulo_Coordinator)
    1.  [Modulo Hooking](#Modulo_Hooking)

## <a name="Ruolo_ntkd"></a>Ruolo del demone ntkd

Il demone *ntkd* deve essere sempre in esecuzione su ogni singolo nodo che compone una rete Netsukuku. Gli obiettivi
che si prefigge sono:

*   Rilevare i vicini. Comunicare con loro per implementare un algoritmo distribuito di esplorazione della rete.
*   Popolare e mantenere le rotte di pertinenza di Netsukuku nelle tabelle di routing dello stack TCP/IP del nodo.
*   Fornire i servizi di base per il funzionamento dei Servizi Distribuiti, opzionali o obbligatori (aka Peer-to-peer Services).
*   Fornire ai client un servizio di interrogazione del database distribuito dei nomi (ANDNA).

### <a name="Definizione_di_rete_netsukuku"></a>Definizione di rete Netsukuku

Una rete Netsukuku si forma automaticamente quando alcuni nodi, su cui è in esecuzione il demone *ntkd*, risultano
fra loro *collegati* attraverso un qualsiasi mezzo, come dei cavi collegati ad uno switch, o delle radio Wi-Fi a
distanza di rilevamento. Questi nodi possono essere dei computer, o degli access point, o altri  dispositivi sui
quali, come già detto, deve essere in esecuzione il  demone *ntkd*.

Una rete esiste, come entità di insieme, fin quando risulta completamente connessa, cioè dati due nodi qualsiasi
esiste un *percorso* che li unisce costituito da nodi tra loro collegati.

La rete può inizialmente formarsi in modo graduale. Ad esempio possono formarsi due piccole reti *A* e *B*
connesse al loro interno ma senza collegamenti diretti tra la rete *A* e la rete *B*. Fin quando non si incontrano,
le due reti esistono e sono indipendenti e separate. Ma quando si forma un collegamento fra un nodo di *A*
e un nodo di *B* le due reti si fondono in una unica rete.

Allo stesso modo, se una rete diventa disconnessa a causa della rimozione di alcuni collegamenti e si formano due
o più gruppi connessi al loro interno, questi gruppi diventano altrettante "isole", cioè reti indipendenti e separate.

Nel resto del documento, quando parliamo di "la rete Netsukuku" di cui fa parte il nodo *n*, ci riferiamo ad una
rete connessa a cui il nodo *n* "appartiene"; cioè il nodo *n* è direttamente collegato ad almeno un altro nodo
di quella rete.

### <a name="Funzionamento_di_rete_netsukuku"></a>Funzionamento di una rete Netsukuku

Un nodo della rete Netsukuku deve essere configurato inizialmente con queste informazioni:

*   Quali sono le interfacce di rete attraverso le quali il nodo è collegato (o può in futuro collegarsi) ad
    altri nodi della rete Netsukuku.
*   Con quale nome (o nomi) vuole essere identificabile dagli altri nodi all'interno della rete Netsukuku.

Inoltre il nodo deve avere configurato come servizio sempre in esecuzione il demone *ntkd* .

Una volta soddisfatti questi requisiti il nodo è parte della rete Netsukuku. Ciò significa che è in grado di
contattare o di essere contattato da qualsiasi altro nodo nella rete Netsukuku. Il demone *ntkd* esplora la
rete Netsukuku, fornisce le informazioni rilevanti allo stack TCP/IP del S.O. e usa il sistema di nomi ANDNA
per registrare i suoi nomi e risolvere i nomi degli altri nodi.

#### <a name="Esplorazione_della_rete"></a>Esplorazione della rete

Il demone *ntkd* in esecuzione nei vari nodi della rete implementa un algoritmo distribuito di esplorazione
della rete. Attraverso di esso ogni nodo della rete Netsukuku raccoglie (e mantiene aggiornate) informazioni
riguardo ogni destinazione presente nella rete. Per destinazione qui si intende un indirizzo effettivamente
assegnato ad un nodo al momento presente, oppure una classe di indirizzi al cui interno almeno un indirizzo
è assegnato al momento presente.

Come avviene anche nella rete Internet, la rete Netsukuku raggruppa gli indirizzi in classi a diversi livelli
gerarchici. Lo fa però in modo automatizzato e con maggiore efficacia.  
Infatti nella rete Internet si riscontra ancora oggi e da parecchi anni un problema: le dimensioni della
[tabella di routing](https://en.wikipedia.org/wiki/Border_Gateway_Protocol#Routing_table_growth)
di ogni router del core di Internet crescono sempre più rapidamente.  
Il protocollo di routing standard usato da questi router per gestire le interconnessioni tra i vari Autonomous
Systems che costituiscono l'intera Internet è il BGP. Nell'anno 2014 il numero di rotte che tale
protocollo ha inserito in alcuni router ha superato le 500000, causando problemi di memoria a molti
router. Nell'anno 2017 si sono superate le 700000 rotte.  
Invece una rete Netsukuku, ad esempio assegnandogli la gestione degli indirizzi IPv4 della classe 10.0.0.0/8,
può contenere fino a 2<sup>22</sup> nodi (4 milioni) e ogni nodo dovrà memorizzare informazioni per un
massimo di 36 destinazioni.

Ogni nodo della rete Netsukuku, per ogni destinazione viene a
conoscenza di un numero (da 1 fino ad un massimo prefissato nel codice) di possibili percorsi per raggiungerlo.

Per ogni singolo percorso esso conosce il gateway, i passi che lo compongono e
il costo del percorso, il quale può essere espresso in diverse metriche, come latenza o larghezza di banda.

In questo [documento](EsplorazioneRete.md) ci sono altre note riguardo l'esplorazione della rete.

#### <a name="Assegnazione_rotte"></a>Assegnazione delle rotte nel kernel

Il software nella suite Netsukuku non intende sostituire lo stack TCP/IP implementato nel S.O. del nodo. Nemmeno
intende interagire con le librerie TCP/IP del S.O. attraverso sistemi di hook. Vuole invece semplicemente
farsi carico di gestire le rotte nelle tabelle di routing del kernel.

Si tratta cioè di un protocollo di routing *proactive*.

Le rotte di pertinenza di Netsukuku sono tutte le rotte necessarie a che ogni singolo nodo esistente nella
rete Netsukuku possa essere raggiunto e che ogni indirizzo nello spazio di Netsukuku che non è stato assegnato ad
alcun nodo sia riportato come "irraggiungibile".

In questo [documento](RotteKernel.md) ci sono altre note riguardo le rotte che il demone *ntkd* imposta nelle
tabelle del kernel e ne viene spiegata la logica.

#### <a name="Associazione_nomi_indirizzi"></a>Associazione nomi - indirizzi

La suite di software Netsukuku offre un meccanismo di risoluzione dei nomi degli host chiamato ANDNA (A Netsukuku
Domain Name Architecture).

ANDNA realizza un database distribuito. Non esistono in esso server indispensabili. I compiti sono razionalmente
distribuiti fra i nodi effettivamente presenti nella rete in ogni dato momento.  
Lo spazio dei nomi non viene strutturato in modo gerarchico, con dei domini, ma è flat. Ogni nome è semplicemente
una sequenza di caratteri validi: se il nome è libero qualsiasi nodo può registrarlo per se nel database
distribuito, senza dover sottostare ad adempimenti verso una qualche entità autoritativa.  
Anche la risoluzione dei nomi può avvenire senza passare per entità autoritative per un dato dominio. Questo
comporta che non ci sono molte occasioni di congestione e il ricorso a caching server diventa meno importante
rispetto a quanto avviene in una struttura a domini come il DNS. Per questo ANDNA reagisce molto rapidamente
agli aggiornamenti delle associazioni nome-indirizzo nel database distribuito.

In questo [documento](RisoluzioneNomi.md) sono maggiormente dettagliate le differenze tra ANDNA e DNS e i
meccanismi di registrazione e risoluzione dei nomi.

## <a name="Architettura_modulare_ntkd"></a>Architettura modulare di ntkd

Il codice implementato nel programma *ntkd* si avvale dei moduli indipendenti che ora illustreremo
e li coordina.

### <a name="Modulo_Neighborhood"></a>Modulo Neighborhood

Il modulo Neighborhood rileva i possibili collegamenti tra il sistema corrente e altri sistemi diretti vicini.
Li realizza e li mantiene memorizzati. Nel tempo ne monitora il costo e/o ne rileva la scomparsa.  
Inoltre il modulo Neighborhood ha il compito di consentire le comunicazioni tra nodi.

Il programma *ntkd* crea una istanza di `NeighborhoodManager` all'avvio delle operazioni. Essa resta in vita
per tutta la durata del programma.  
Il programma fornisce nel costruttore i delegati per una serie di operazioni:

*   Creare uno stub per chiamare metodi remoti in broadcast, in unicast o tramite indirizzo IP.
*   Assegnare indirizzi link-local alle proprie interfacce di rete e impostare rotte verso gli indirizzi
    link-local dei propri diretti vicini. Solo per l'identità principale del nodo (nel network namespace
    default) e per le identità principali dei diretti vicini.
*   Ottenere il/i dispatcher associati a una identità del sistema o al sistema.  
    Questo opera in sinergia con l'implementazione di `IRpcDelegate` che il programma *ntkd* passa alla
    libreria *ntkdrpc*, come verrà illustrato nei [dettagli tecnici](DettagliTecnici.md).

Poi il programma passa all'istanza di `NeighborhoodManager` tutte le interfacce di rete che vuole gestire. Per
ognuna di esse il nome, il MAC e un delegato per misurare il RTT con un dato sistema vicino.

Prima di terminare, il programma *ntkd* deve chiamare il metodo `stop_monitor_all` per poter ricevere e
gestire i vari segnali `nic_address_unset`.  
Poi dismetterà l'istanza di `NeighborhoodManager`.

#### Gestione dei messaggi RPC ricevuti

Quando il programma *ntkd* riceve un messaggio di RPC tramite ZCD, esso usa l'istanza di `NeighborhoodManager`
per trovare il/i dispatcher da attivare.

Quando poi un dispatcher avvia un metodo remoto sul sistema corrente, questo fa parte di un modulo. Il
codice in tale metodo, implementato nel modulo interessato, può chiedere all'utilizzatore del modulo,
cioè al programma *ntkd*, di individuare in qualche forma il sistema chiamante; in questo caso il programma *ntkd*
usa l'istanza di `NeighborhoodManager` per identificare il vicino (identità o sistema)
che ha inviato il messaggio di RPC.

L'implementazione di entrambi questi compiti nel modulo Neighborhood rispecchia quanto descritto nel
relativo documento [ChiamateLatoServer](../ModuloNeighborhood/ChiamateLatoServer.md).

Per il primo compito (trovare il/i dispatcher) il programma *ntkd* deve fornire al `NeighborhoodManager`
queste informazioni:

*   uno skeleton per il nodo.
*   un delegato per reperire uno skeleton per una identità.
*   un delegato per reperire una lista di skeleton per più identità.

In particolare, le funzioni delegate per identificare uno o più skeleton di identità saranno implementate
dal programma *ntkd* avvalendosi dei servizi del modulo Identities.

Per il secondo compito (individuare il vicino che ha chiamato il metodo remoto)  il programma *ntkd*
riceve dal `NeighborhoodManager` o un `INeighborhoodArc` (se il modulo interessato è un modulo *di nodo*)
o un `NodeID` (se il modulo interessato è un modulo *di identità*). Il programma *ntkd* deve quindi mantenere
in memoria le dovute associazioni che gli permettano di fornire una risposta adeguata al modulo interessato.

### <a name="Modulo_Identities"></a>Modulo Identities

**TODO**

### <a name="Modulo_Qspn"></a>Modulo Qspn

**TODO**

### <a name="Modulo_PeerServices"></a>Modulo PeerServices

**TODO**

### <a name="Modulo_Coordinator"></a>Modulo Coordinator

**TODO**

### <a name="Modulo_Hooking"></a>Modulo Hooking

**TODO**

