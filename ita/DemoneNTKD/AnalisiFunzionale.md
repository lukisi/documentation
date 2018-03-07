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
    libreria *ntkdrpc*, come verrà illustrato nei [dettagli tecnici](DettagliTecnici.md#RPC_Library).

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
dal programma *ntkd* avvalendosi dei dati che ha memorizzato sulla base delle segnalazioni fatte
dal modulo Identities.

Per il secondo compito (individuare il vicino che ha chiamato il metodo remoto)  il programma *ntkd*
riceve dal `NeighborhoodManager` o un `INeighborhoodArc` (se il modulo interessato è un modulo *di nodo*)
o un `NodeID` (se il modulo interessato è un modulo *di identità*). Il programma *ntkd* deve quindi mantenere
in memoria le dovute associazioni che gli permettano di fornire una risposta adeguata al modulo interessato.

### <a name="Modulo_Identities"></a>Modulo Identities

Il sistema su cui è in esecuzione il demone *ntkd* è un elemento di una rete.

Possiamo fare in modo che nel sistema esistano diverse identità. Ognuna di esse gestisce
un *network stack* indipendente. Quindi ognuna di esse è un distinto elemento della rete.  
Questo stratagemma ci consente di trovare una soluzione al problema di assegnazione degli
indirizzi in una rete a topologia gerarchica.

Il modulo Identities aiuta il programma nella realizzazione e gestione di queste distinte identità
nel sistema.

Il programma *ntkd* crea una istanza di `IdentityManager` all'avvio delle operazioni. Essa resta in vita
per tutta la durata del programma.  
Il programma fornisce nel costruttore l'elenco delle interfacce di rete da gestire (per ognuna di esse
è fornito il nome, il MAC e l'indirizzo IP link-local) e i delegati per una serie di operazioni:

*   Creare, gestire e rimuovere i *network stack* indipendenti. Il primo, chiamato *network namespace default*,
    esiste da subito e non viene mai rimosso. In esso vivono tutte le applicazioni in esecuzione
    nel sistema, se non viene espressamente richiesta la loro esecuzione su un altro stack.
*   Ottenere uno stub per comunicazioni reliable ad un vicino dato un arco.  
    Per questo il programma si avvale del modulo Neighborhood, metodo `get_stub_whole_node_unicast`.
*   Ottenere l'arco da cui è stata ricevuta una chiamata a metodo remoto, passando il `caller` ricevuto
    nei parametri del metodo remoto.  
    Per questo il programma si avvale del modulo Neighborhood, metodo `get_node_arc`.

La prima identità del sistema (che è inizialmente la *principale*) viene subito creata, quindi il
programma immediatamente ne recupera un riferimento chiamando il metodo `get_main_id`.
Lo utilizza per costruire un oggetto `IdentityData` in cui mantiene alcune associazioni interne
al programma stesso, che non sono di pertinenza del modulo Identities.  
Ad esempio il modulo Identities sa se una certa identità è la principale o una di connettività, ma nel secondo
caso non sa dire quali siano i livelli dei g-nodi di cui essa supporta la connettività interna: tale dettaglio
il programma lo mantiene nei membri `connectivity_from_level` e `connectivity_to_level` della
classe `IdentityData`.  
Altri dettagli memorizzati in tale classe sono:

*   `int connectivity_from_level`. Se è 0 significa che è l'identità principale. Se è una
    identità di connettività questo valore può andare da 1 a `levels-1`.
*   `int connectivity_to_level`. Se è 0 significa che è l'identità principale. Se è una
    identità di connettività questo valore può andare da `int connectivity_from_level` a `levels-1`.
*   `IdentitySkeleton identity_skeleton`. Lo skeleton radice da usare per i moduli di identità.
*   `weak QspnManager qspn_mgr`. Un riferimento all'istanza del modulo Qspn relativa a questa identità.
*   `weak PeersManager peers_mgr`. Un riferimento all'istanza del modulo PeerServices relativa a questa identità.
*   `List<IdentityArc> identity_arcs`. Lista degli archi-identità.
*   `...`

Similmente, il programma quando si avvede della creazione di archi di identità segnalati dal modulo Identities
costruisce un oggetto `IdentityArc` (memorizzato nella lista `identity_arcs` del relativo `IdentityData`)
in cui mantiene alcune associazioni che non sono di pertinenza del modulo Identities.  
I dettagli memorizzati in tale classe sono:

*   `weak IdentityData identity_data`. Riferimento alla propria identità.
*   `IIdmgmtArc arc`. Riferimento all'arco fisico. Come passato al modulo Identities.
*   `IIdmgmtIdentityArc id_arc`. Riferimento all'arco-identità. Come ottenuto dal modulo Identities.
*   `string peer_mac`. MAC del vicino. Come riportato l'ultima volta dal modulo Identities.
*   `string peer_linklocal`. Indirizzo IP link-local del vicino. Come riportato l'ultima volta dal
    modulo Identities.
*   `string? prev_peer_mac`. MAC precedente del vicino. Valorizzato quando il modulo Identities
    segnala che l'identità del vicino collegato a noi su un esistente arco-identità ha cambiato i
    suoi parametri.
*   `string? prev_peer_linklocal`. Indirizzo IP link-local precedente del vicino. Valorizzato quando
    il modulo Identities segnala che l'identità del vicino collegato a noi su un esistente
    arco-identità ha cambiato i suoi parametri.
*   `IQspnArc? qspn_arc`. Una istanza di una classe che viene passata al modulo QSPN. Se è valorizzato
    significa che il *peer* appartiene alla stessa rete di questa identità.
*   `int64? network_id`. Identifica la rete a cui appartiene il *peer*. Se il programma ha questa informazione
    essa gli è stata comunicata dal modulo Hooking. Questo può avvenire solo se questo è un arco-identità principale.
*   `...`

Quando il modulo Neighborhood segnala `neighborhood_arc_added`, cioè che un nuovo
arco fisico (un collegamento con un sistema diretto vicino) è stato aggiunto, il programma
lo comunica al modulo Identities con il metodo `add_arc`; similmente alla segnalazione
dal modulo Neighborhood `neighborhood_arc_removing`, cioè che un arco fisico sta per essere
rimosso, il programma chiama sul modulo Identities il metodo `remove_arc`.  
In particolare, quando viene aggiunto un arco fisico (rappresentato dal modulo Neighborhood
con una istanza di `INeighborhoodArc`), il programma crea e memorizza (in `arc_list`)
una istanza di una classe `IdmgmtArc` che implementa l'interfaccia `IIdmgmtArc` che si aspetta
il modulo Identities nel metodo `add_arc`. Quando un arco fisico sta per essere rimosso il programma
cerca nella lista `arc_list` la relativa istanza di `IdmgmtArc` per passarla al modulo Identities
nel metodo `remove_arc`.

#### Archi identità

Il programma riceve dal modulo Identities degli eventi che sono riferiti alla sua gestione
degli archi-identità.  
Alla creazione di un arco-identità il modulo aggiunge una rotta diretta verso il relativo indirizzo IP link-local;
analogamente, alla rimozione il modulo rimuove la relativa rotta. Queste operazioni sulle rotte (cioè sulle
tabelle di routing del kernel del S.O.) non avvengono per l'arco-identità principale, perché in quel caso se
ne occupa il modulo Neighborhood.  
Tenendo traccia delle segnalazioni ricevute dal modulo, è compito del programma *ntkd* gestire opportunamente
le rotte verso qualsiasi destinazione che usano come gateway un certo arco-identità.

I segnali che il modulo Identities emette sono:

*   `identity_arc_added`. Quando è stato aggiunto un arco-identità.
*   `identity_arc_changed`. Quando un arco-identità cambia il suo `peer-mac` e `peer_linklocal`.
*   `identity_arc_removing`. Quando un arco-identità sta per essere rimosso.
*   `identity_arc_removed`. Quando un arco-identità è stato rimosso.

##### Aggiunta arco fisico

Consideriamo un singolo arco fisico. Quando viene aggiunto, è il programma stesso che lo
comunica al modulo Identities. Il modulo Identities crea in quel momento il primo arco-identità ad esso
associato, che è quello che collega l'identità principale nel sistema corrente con l'identità
principale nel sistema diretto vicino; è chiamato arco-identità *principale*. Il modulo Identities
non aggiunge una rotta in questo caso perché essa è stata aggiunta nel *network namespace default* dal
modulo Neighborhood.  
Il programma riceve il segnale `identity_arc_added` relativo all'arco-identità principale. A questo
segnale crea una istanza di `IdentityArc` e la associa alla relativa istanza di `IdentityData`.
Inoltre passa questo arco-identità al modulo Hooking, il cui compito è gestire l'incontro di reti
distinte oppure costruire un arco Qspn.

##### Duplicazione identità

Ora supponiamo che nel sistema vicino l'identità principale (a noi collegata tramite un arco-identità)
si duplica (a causa di una migrazione). Il modulo Identities nel sistema vicino contatta il modulo
Identities nel nostro sistema e comunica questo evento. In risposta, il modulo Identities nel nostro
sistema aggiunge un arco-identità e modifica le proprietà (`peer_linklocal` e `peer_mac`) del vecchio
arco-identità.  
Prima aggiunge un arco-identità nella sua memoria, ma non aggiunge la rotta all'indirizzo IP link-local
perché già esiste. Poi modifica le proprietà del vecchio arco-identità nella sua memoria e aggiunge la
rotta al nuovo indirizzo IP link-local. Poi il modulo segnala prima `identity_arc_changed` e in
seguito `identity_arc_added`.  
Il programma riceve il segnale `identity_arc_changed` relativo al vechio arco-identità. Il programma trova
nella sua memoria l'istanza di `IdentityArc` e guarda le sue vecchie proprietà e le memorizza. Poi
le aggiorna. Inoltre qui, se questo arco-identità era stato comunicato al modulo Qspn in quanto l'identità
vicina appartiene alla stessa rete della nostra identità, sarà necessario aggiornare nelle tabelle del
kernel tutte le rotte che usano questo arco-identità come gateway.  
Poi il programma riceve il segnale `identity_arc_added` relativo al nuovo arco-identità, che nel nostro caso
è di nuovo il *principale*, ma questo non ci interessa. Come prima, il programma crea una istanza
di `IdentityArc` e la associa alla relativa istanza di `IdentityData`. E la passa al modulo Hooking.

Ora vediamo invece cosa avviene quando una identità nel nostro sistema, la quale ha degli archi-identità
associati, si duplica.  
Il modulo Identities ha creato il *network namespace* indipendente che sarà gestito dalla vecchia identità,
mentre ha associato alla nuova identità il precedente *network namespace*.  
Il modulo Identities nel nostro sistema contatta il modulo Identities nel sistema vicino e comunica questo evento.
Al tempo stesso apprende che l'identità nel sistema vicino non ha preso parte alla stessa migrazione.  
Non serve aggiungere una rotta nel vecchio namespace (perché già esiste) che realizza il nuovo arco-identità.  
Il modulo Identities nel nostro sistema aggiunge un arco-identità alla nuova identità e lo comunica con il
segnale `identity_arc_added`.  
Nel nuovo namespace, invece, gestito dalla vecchia identità, viene aggiunta la rotta che realizza il vecchio
arco-identità.  
Il modulo Identities non riporta alcun segnale relativamente al vecchio arco-identità.  
Quindi il programma riceve solo il segnale `identity_arc_added` relativo al nuovo arco-identità. Come prima,
il programma crea una istanza di `IdentityArc` e la associa alla relativa istanza di `IdentityData`. E la
passa al modulo Hooking.

Infine vediamo cosa avviene quando una identità nel nostro sistema si duplica per via di una migrazione
che coinvolge anche una identità di un sistema vicino che è ad essa collegata.  
Quanto segue avviene in entrambi i sistemi (per le identità che hanno partecipato alla stessa migrazione)
indipendentemente dall'ordine in cui i due sistemi avviano le operazioni.  
Il modulo Identities ha creato il *network namespace* indipendente che sarà gestito dalla vecchia identità,
mentre ha associato alla nuova identità il precedente *network namespace*.  
Il modulo Identities nel nostro sistema contatta il modulo Identities nel sistema vicino e comunica questo evento.
Al tempo stesso apprende che l'identità nel sistema vicino ha preso parte alla stessa migrazione.  
Non serve aggiungere una rotta nel vecchio namespace (perché già esiste) che realizza il nuovo arco-identità.  
Il modulo Identities nel nostro sistema aggiunge un arco-identità alla nuova identità e lo comunica con il
segnale `identity_arc_added`.  
Nel nuovo namespace, invece, gestito dalla vecchia identità, viene aggiunta la rotta che realizza il vecchio
arco-identità.  
Questo vecchio arco-identità ha però le proprietà modificate. Quindi il modulo Identities riporta
per esso il segnale `identity_arc_changed`.  
Il programma riceve per primo il segnale `identity_arc_added` relativo al nuovo arco-identità. Come prima, il programma
crea una istanza di `IdentityArc` e la associa alla relativa istanza di `IdentityData`. E la passa al modulo Hooking.  
Il programma riceve poi il segnale `identity_arc_changed` relativo al vechio arco-identità. Ma in questa situazione
c'è da dire che i due segnali sono relativi a due identità distinte nel nostro sistema, quindi operano su
due distinti *network namespace*. Il programma trova nella sua memoria l'istanza di `IdentityArc` e guarda le sue vecchie
proprietà e le memorizza. Poi le aggiorna. Inoltre, se questo arco-identità era stato comunicato al modulo Qspn, sarà
necessario aggiornare nelle tabelle del kernel tutte le rotte che usano questo arco-identità come gateway.

##### Rimozione identità

**TODO**

#### Terminazione del programma

Prima di terminare, il programma *ntkd* deve rimuovere tutte le identità di connettività che
eventualmente esistono nel sistema con il metodo `remove_identity` del modulo Identities.

**TODO**

### <a name="Modulo_Qspn"></a>Modulo Qspn

Il modulo QSPN viene portato a conoscenza dell' *indirizzo Netsukuku* del nodo del grafo nel
quale sta operando. Poi il modulo si occupa di raccogliere e diffondere le conoscenze necessarie
al popolamento delle tabelle di routing. In questo modo, dato che una istanza del modulo QSPN
opera in ogni nodo del grafo, la rete Netsukuku è in grado di instradare i pacchetti IP da un
qualsiasi nodo a qualsiasi altro nodo. Ricordiamo che un nodo del grafo è una specifica identità
che vive in un dato sistema.

Il programma *ntkd* associa ad ogni identità nel sistema una istanza di QspnManager. Tale istanza resta in vita
finché resta in vita l'identità.

La prima identità viene creata all'avvio del programma *ntkd* nel sistema. Lo abbiamo detto anche prima trattando
le interazioni del programma con il modulo Identities. Tale identità è un nodo che costituisce una rete a se
stante. Il nodo non ha alcun arco.  
Ricordiamo che un arco passato al modulo Qspn (che in seguito chiameremo *arco-qspn*) è costruito su
un arco-identità che collega due identità che appartengono alla stessa rete.

Quindi il programma *ntkd* ha il semplice compito di creare in autonomia la prima istanza di QspnManager e
associarla alla prima identità del sistema. Lo fa con il costruttore `create_net`, al quale passa
l'indirizzo Netsukuku e il fingerprint.  
L'indirizzo Netsukuku è composto dalle posizioni del nodo in ogni g-nodo a cui esso appartiene; siccome
si tratta del primo nodo di una rete, per definizione nel momento in cui nasce il nodo nascono anche tutti
i g-nodi (di livello 1, 2, ..., `levels`) a cui esso appartiene e nessuno di essi aveva posizioni occupate;
quindi tutte le posizioni dell'indirizzo possono essere scelte in modo arbitrario nel range da 0 a `gsizes[i]-1`.  
Il fingerprint è composto da un identificativo univoco del nodo e dalle anzianità: l'anzianità del nodo nel suo
g-nodo di livello 1, l'anzianità del g-nodo di livello 1 nel suo g-nodo di livello 2, e così via; l'identificativo
univoco del nodo è scelto in modo casuale; trattandosi del primo nodo di una rete tutte le anzianità sono impostate
a 0, che significa appunto il membro più anziano in assoluto del suo g-nodo di livello superiore.

Successivamente il programma *ntkd* potrà richiamare altre operazioni sul modulo QSPN. Potrà aggiungere (o rimuovere)
un arco-qspn ad una istanza di QspnManager. Potrà duplicare una sua identità e di conseguenza creare una nuova istanza
di QspnManager con il costruttore `enter_net` o `migrate` (non più con `create_net`).  
Queste decisioni saranno prese a fronte di segnalazioni ricevute dal modulo Hooking. Ne parleremo quindi
in seguito.

Inoltre il programma *ntkd* si mette in ascolto dei segnali emessi dal modulo QSPN per essere aggiornato
sulle conoscenze del *nodo del grafo* rappresentato da una identità. Sulla base di queste conoscenze il
programma dovrà tenere aggiornate le policy di routing del sistema.

Tutte le operazioni che il programma deve fare a tal proposito vengono illustrate
nel [documento di dettaglio](DettagliTecnici.md#Indirizzi_IP).

**TODO**

### <a name="Modulo_PeerServices"></a>Modulo PeerServices

Il modulo PeerServices ha il ruolo di registrare i servizi peer-to-peer ai quali un nodo partecipa attivamente
e di propagare le informazioni necessarie all'instradamento dei pacchetti delle comunicazioni peer-to-peer.  
Il programma *ntkd* da parte sua deve soltanto registrare nella classe PeersManager (associata ad ogni identità)
i servizi peer-to-peer ai quali il nodo (la suddetta identità) partecipa attivamente.  

Il programma *ntkd* associa ad ogni identità nel sistema una istanza di PeersManager. Tale istanza resta in vita
finché resta in vita l'identità.

All'avvio, il programma *ntkd* ha il semplice compito di creare in autonomia la prima istanza di PeersManager e
associarla alla prima identità del sistema. Per questo deve fornire:

*   Un'interfaccia per reperire le informazioni sulla mappa dei percorsi e sui gateway.  
    Serve anche per trasmettere pacchetti al gateway per l'instradamento dei pacchetti delle comunicazioni
    peer-to-peer dal client verso il server.
*   Un delegato per trasmettere pacchetti ai diretti vicini per la propagazione delle mappe di partecipazione.
*   Un delegato per aprire una connessione dal server sul client.

Successivamente il programma *ntkd* potrà registrare le istanze dei servizi a cui il nodo partecipa attivamente.

Il programma *ntkd* potrà duplicare una sua identità e di conseguenza creare una nuova istanza di PeersManager
fornendo in questo caso anche:

*   L'istanza del modulo che era associata alla precedente identità.
*   Il livello del g-nodo ospite.
*   Il livello del g-nodo ospitante.

Queste decisioni saranno prese a fronte di segnalazioni ricevute dal modulo Hooking. Ne parleremo quindi
in seguito.

**TODO**

### <a name="Modulo_Coordinator"></a>Modulo Coordinator

**TODO**

### <a name="Modulo_Hooking"></a>Modulo Hooking

**TODO**

#### Ingresso di una identità in una diversa rete

Consideriamo le conoscenze del programma *ntkd* rispetto agli archi-identità di una identità `id` nel sistema.
Esso ha memorizzato alcune informazioni su istanze di `IdentityArc` in `id.identity_arcs`.

Se, riguardo un certo arco-identità, il programma ha già associato un arco-qspn, allora sa che l'identità
collegata nel diretto vicino (il *peer* dell'arco-identità) appartiene alla nostra stessa rete.
L'istanza di `IQspnArc` è stata memorizzata dal programma nel membro `qspn_arc` di `IdentityArc`.

Se invece il programma non ha associato un `qspn_arc`, quindi il *peer* non appartiene alla nostra stessa rete,
soltanto nel caso che si tratti di un arco-identità principale (cioè se `id` è l'identità principale del nostro sistema
e il *peer* di questo arco-identità è l'identità principale del sistema vicino) il programma sa dire quale sia
il network-id della rete a cui appartiene il *peer*: questa informazione gli è stata comunicata dal modulo
Hooking e il programma l'ha memorizzata nel membro `network_id` di `IdentityArc`.

Consideriamo ora il momento in cui il programma *ntkd* gestisce una sua identità `old_id` che si è duplicata
in `new_id` per fare ingresso in un'altra rete di cui conosce il network-id. Cioè subito dopo il
completamento della chiamata `identity_mgr.add_identity`.

Relativamente ai suoi archi-identità che avevano un `qspn_arc`, il programma sa discernere se l'identità del peer
ha preso parte all'ingresso in altra rete insieme con lui: lo capisce dai membri `prev_peer_*` di IdentityArc.

In sostanza, il programma in questo momento sa discernere per ogni arco-identità di `old_id` se c'era un arco-qspn
nella vecchia rete e se ci sarà un arco-qspn nella nuova rete per la relativa copia di arco-identità di `new_id`.

Quindi prepara queste liste di coppie di IdentityArc, cioè di `IdentityArcPair`:

*   Consideriamo una classe `IdentityArcPair` che contiene il membro `IdentityArc old_id_arc` e il
    membro `IdentityArc new_id_arc`. Il primo viene valorizzato con una istanza di arco-identità
    che appartiene a `old_id` e il secondo con la relativa copia che appartiene a `new_id`.
*   `prev_arcs`: coppie in cui l'arco-identità `old_id_arc` aveva un arco-qspn nella vecchia rete e
    l'arco-identità `new_id_arc` non avrà un arco-qspn nella nuova rete.  
    L'identità del peer di `old_id` era nella stessa vecchia rete ma non era nel blocco che ha fatto
    ingresso nella nuova rete.
*   `new_arcs`: coppie in cui l'arco-identità `old_id_arc` non aveva un arco-qspn nella vecchia rete ma
    l'arco-identità `new_id_arc` avrà un arco-qspn nella nuova rete.  
    L'identità del peer di `old_id` era già nella nuova rete.
*   `both_arcs`: coppie in cui l'arco-identità `old_id_arc` aveva un arco-qspn nella vecchia rete e
    l'arco-identità `new_id_arc` avrà un arco-qspn nella nuova rete.  
    L'identità del peer di `old_id` era nella stessa vecchia rete ed era nel blocco che ha fatto
    ingresso nella nuova rete.

Queste informazioni sono necessarie per creare l'istanza di QspnManager che va associata alla nuova identità
e per espletare una serie di operazioni sia nel network namespace (nuovo) della vecchia identità, sia nel
network namespace (ereditato) della nuova identità.

