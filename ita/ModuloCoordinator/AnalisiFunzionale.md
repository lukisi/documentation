# Modulo Coordinator - Analisi Funzionale

1.  [Il ruolo del modulo Coordinator](#Ruolo_coordinator)
    1.  [Collaborazione con il modulo Migrations](#Collaborazione_migrations)
1.  [Il servizio Coordinator](#Servizio_coordinator)
    1.  [Contenuto della memoria condivisa di un g-nodo](#Records)
    1.  [Richieste previste](#Richieste_previste)
        1.  [Numero di nodi nella rete](#Numero_nodi_nella_rete)
        1.  [Valuta un ingresso](#Valuta_ingresso)
        1.  [Avvio ingresso in altra rete](#Avvio_ingresso)
        1.  [Confermato ingresso in altra rete](#Confermato_ingresso)
        1.  [Lettura memoria di pertinenza del modulo Migrations](#Get_migrations_memory)
        1.  [Scrittura memoria di pertinenza del modulo Migrations](#Set_migrations_memory)
        1.  [Prenota un posto](#Prenota_un_posto)
        1.  [Cancella prenotazione](#Cancella_prenotazione)
        1.  [Replica memoria condivisa](#Replica)
1.  [Requisiti](#Requisiti)
1.  [Deliverables](#Deliverables)
    1.  [CoordinatorManager](#Deliverables_manager)
    1.  [CoordinatorService e CoordinatorClient](#Deliverables_service)
1.  [Classi e interfacce](#Classi_e_interfacce)
    1.  [Interfacce](#Classi_Interfacce)
    1.  [Delegati](#Classi_Delegati)
    1.  [Strutture dati](#Classi_Strutture)

## <a name="Ruolo_coordinator"></a>Il ruolo del modulo Coordinator

Il modulo cerca di fare sì che un singolo g-nodo di livello *l*, sebbene costituito da un numero di singoli
nodi, abbia un comportamento coerente come singola entità.

Il modulo fa uso delle [tasklet](../Librerie/TaskletSystem.md), un sistema di multithreading cooperativo.

Il modulo fa uso del framework [ZCD](../Librerie/ZCD.md), precisamente appoggiandosi ad una libreria intermedia
prodotta con questo framework per formalizzare i metodi remoti usati nel demone *ntkd* per comunicare con i
diretti vicini.

Il modulo fa uso diretto delle classi e dei servizi forniti dal modulo [PeerServices](../ModuloPeers/AnalisiFunzionale.md).
In particolare, esso realizza un servizio peer-to-peer, chiamato appunto Coordinator, per mezzo del quale svolge
alcuni dei suoi compiti. Lo analizzeremo fra breve.

Siccome chiamiamo con lo stesso nome Coordinator sia il modulo presente su ogni nodo sia il servizio peer-to-peer
che rappresenta un g-nodo, bisogna che il lettore faccia attenzione al contesto. Di solito se parliamo
di Coordinator di un g-nodo (dal livello 1 fino all'intera rete) ci riferiamo al servizio, cioè
al nodo che al momento viene identificato come servente.

### <a name="Collaborazione_migrations"></a>Collaborazione con il modulo Migrations

Le operazioni di ingresso in una rete, che sono messe in atto quando due reti distinte si incontrano
per mezzo di alcuni archi, non sono di pertinenza del modulo Coordinator, bensì del modulo Migrations.

Però gli algoritmi del modulo Migrations per questi compiti hanno dei requisiti che possono essere soddisfatti
attrtaverso una stretta collaborazione con il modulo Coordinator.

#### Esecuzioni sul nodo Coordinator di un g-nodo

In alcuni casi un g-nodo deve comportarsi come una unica entità. In questi casi abbiamo questi requisiti:

1.  Il modulo Migrations in esecuzione in un qualsiasi singolo nodo (ad esempio un border-nodo che ha incontrato
    un vicino di un'altra rete) vuole far eseguire alcuni suoi metodi nel nodo Coordinator di tutta la rete
    o di un g-nodo.
1.  Un metodo del modulo Migrations in esecuzione nel nodo Coordinator di tutta la rete (o di un g-nodo)
    vuole accedere in lettura/scrittura alla memoria condivisa di tutta la rete (o di un g-nodo) per salvare
    e recuperare alcune informazioni di sua pertinenza.

Per il primo requisito esistono metodi proxy del modulo Coordinator, relativi metodi nella classe client del
servizio Coordinator, relative classi di richieste e risposte (IPeersRequest e IPeersResponse) e delegati
che la classe servente del servizio Coordinator (PeerService) può richiamare.  
Di norma per ogni metodo del modulo Migrations (di quelli che vanno eseguiti nel nodo Coordinator su richiesta di
un altro nodo) esiste una specifica istanza dei suddetti elementi.

Per il secondo requisito, esiste una classe serializzabile implementata nel modulo Migrations tale che una sua
istanza contiene tutta la memoria condivisa di tutta la rete (o di un g-nodo) relativamente a quanto è di pertinenza
del modulo Migrations. Tale istanza viene salvata in un apposito membro della classe usata per contenere
i record del database distribuito realizzato dal servizio Coordinator.  
Il modulo Coordinator quindi mette a disposizione dei metodi per leggere o scrivere in questa memoria
garantendo la coerenza del dato nel database distribuito.

Si vedranno nella trattazione i metodi `evaluate_enter`, `begin_enter`, `completed_enter`,
`get_migrations_memory` e `set_migrations_memory`.

#### Esecuzioni su tutti i nodi di un g-nodo

In alcuni casi il modulo Migrations in esecuzione in un qualsiasi singolo
nodo (ad esempio quello che riceve il compito di coordinare una migrazione)
vuole far eseguire alcuni suoi metodi in tutti i singoli nodi di un g-nodo.  
Queste esecuzioni possono essere di due tipi:

*   *Propagazione con ritorno.* Si vuole provocare l'esecuzione di un metodo in ogni nodo del proprio g-nodo
    di livello *k* e ottenere una risposta soltanto dopo che tutti i nodi di quel g-nodo hanno eseguito il metodo.
*   *Propagazione senza ritorno.* Si vuole provocare l'esecuzione di un metodo in ogni nodo del proprio g-nodo
    di livello *k* senza attendere una risposta.

Per queste necessità esistono metodi del modulo Coordinator che iniziano la propagazione, relativi metodi
remoti per la propagazione, e relativi delegati da chiamare (una volta) su ogni singolo nodo.  
Ogni singolo metodo ha il compito di scegliere le informazioni che servono alla propagazione: il metodo che
inizia la propagazione deve raccogliere queste informazioni e metterle nel pacchetto da trasmettere (in
broadcast oppure in unicast). Il metodo remoto che viene chiamato deve verificare sulla base delle
informazioni se il messaggio è rivolto anche a lui, cioè in altre parole se esso fa parte del
g-nodo in cui il messaggio va propagato.  
Ogni singolo metodo di questi ha direttamente codificata in sé la tipologia, cioè se *con* o
*senza* ritorno e nel primo caso come trattare il valore restituito da ogni singolo nodo e combinarlo
col valore restituito dal delegato in questo nodo.

Si vedranno nella trattazione i metodi `prepare_migration`, `finish_migration`.

#### Altre collaborazioni

Una importante funzione del modulo Coordinator è quella di riservare un posto in un g-nodo. Anche
questa si può vedere come una fondamentale collaborazione del modulo Coordinator per le esigenze
del modulo Migrations.

Un altro caso di collaborazione richiesta dal modulo Migrations è un metodo che il modulo Coordinator
mette a disposizione per chiedere al nodo Coordinator della rete il numero di singoli nodi in essa.

Si vedranno nella trattazione i metodi `reserve`, `get_n_nodes`.

## <a name="Servizio_coordinator"></a>Il servizio Coordinator

Il servizio Coordinator è un servizio non opzionale, cioè tutti i nodi partecipano attivamente. Si tratta di un
servizio che mantiene un database distribuito della tipologia a chiavi fisse.

Lo scopo del servizio Coordinator è quello di realizzare una sorta di memoria condivisa di un g-nodo *g*. Quando
un nodo *n* vuole accedere alla memoria condivisa del suo g-nodo *g* di livello *l*, con 0 < *l* ≤ *levels*, fa
una richiesta al servizio Coordinator usando come chiave il livello *l*.

Il livello *l* è maggiore di 0 poiché non serve realizzare una memoria condivisa per il singolo nodo. Può essere
uguale al numero dei livelli, poiché è necessario avere una memoria condivisa anche per l'unico g-nodo di livello
*l* = *levels* che costituisce l'intera rete.

Lo spazio delle chiavi definito dal servizio Coordinator è appunto l'insieme dei valori che può assumere *l*.

La funzione *h<sub>p</sub>* è definita dal servizio in modo da dare ai dati la visibilità locale circoscritta al
g-nodo in esame. In altre parole, il nodo corrente può contattare solo il Coordinator di uno dei suoi g-nodi; sia
l'hash-node che il nodo che risponde si trovano all'interno del g-nodo stesso.

### <a name="Records"></a>Contenuto della memoria condivisa di un g-nodo

Come abbiamo detto, il servizio Coordinator realizza una sorta di memoria condivisa di un g-nodo *g*. Per far questo
esso mantiene un database distribuito della tipologia a chiavi fisse in cui la chiave è il livello del g-nodo
in esame e i dati hanno visibilità locale circoscritta allo stesso g-nodo.

Questo significa che il contenuto di tutta la memoria condivisa di un g-nodo deve essere rappresentabile
interamente con l'istanza di un oggetto serializzabile. La classe definita nel modulo Coordinator usata per
memorizzare e trasmettere il contenuto della memoria condivisa è `CoordGnodeMemory`.

Si tratta di una classe serializzabile. Il significato di alcuni dei suoi membri è noto al modulo Coordinator.
Altri membri invece contengono strutture dati che il modulo Coordinator non è tenuto a conoscere. Questi membri sono di
tipo Object nullable e il modulo Coordinator sa solo che se sono valorizzati sono a loro volta oggetti serializzabili.

#### Contenuto di pertinenza del modulo Coordinator

Alcune informazioni che si devono poter memorizzare e leggere nella memoria condivisa di un g-nodo sono
di pertinenza del modulo Coordinator stesso.

Queste sono memorizzate nei seguenti membri della classe `CoordGnodeMemory`:

*   `List<Booking> reserve_list` - Elenco delle prenotazioni pendenti.  
    Ogni istanza di Booking contiene:
    *   `int reserve_request_id`
    *   `int new_pos`
    *   `int new_eldership`
    *   `Timer timeout`
*   `int max_virtual_pos` - Massimo valore *virtuale* di `pos` assegnato ad un g-nodo al nostro interno.
*   `int max_eldership` - Massimo valore di eldership assegnato ad un g-nodo al nostro interno. Maggiore è questo valore
    e più giovane è il g-nodo.
*   `int n_nodes` e `Timer? n_nodes_timeout` - Solo per il Coordinator di tutta la rete. Numero di nodi nella rete,
    come risposto nella precedente richiesta e timer da aspettare prima di guardare di nuovo alla conoscenza
    acquisita dal modulo Qspn.

#### Contenuto di pertinenza di altri moduli

Alcune informazioni che si devono poter memorizzare e leggere nella memoria condivisa di un g-nodo sono
di pertinenza di altri moduli.

Verranno descritte ognuna con i suoi dettagli di seguito.

#### <a name="Records_modulo_migrations"></a>Modulo Migrations

Le informazioni di pertinenza del modulo Migrations sono memorizzate nel seguente membro della classe `CoordGnodeMemory`:

*   `Object? migrations_memory`.

Vediamo come avviene la scrittura e la rilettura della memoria condivisa di tutta la rete (o di un g-nodo) ad opera
del modulo Migrations. Nella trattazione del modulo Migrations abbiamo detto
(vedi [qui](../ModuloMigrations/AnalisiFunzionale.md#Accesso_memoria_condivisa))
che solo lo stesso nodo Coordinator (di tutta la rete o di un g-nodo)
può essere nella posizione di scrivere/leggere in questa memoria.  
Quando viene chiamato nel modulo Coordinator il metodo `set_migrations_memory` o il metodo
`get_migrations_memory` questi verifica di essere in esecuzione proprio sul nodo Coordinator
del livello richiesto. Può fare questo controllo con il metodo pubblico `am_i_servant_for(k)`
della classe base PeerClient ereditato da CoordinatorClient.
Altrimenti rilancia l'eccezione NotCoordinatorNodeError.  
Quando viene chiamato nel modulo Coordinator il metodo `set_migrations_memory(Object data, int level)`
(dopo che ha verificato di essere in esecuzione sul nodo Coordinator) questo crea una istanza
del CoordinatorClient sulla quale chiama l'omonimo metodo. Questi avvia il
contatto con il servizio Coordinator per la chiave `k.lvl = level`.  
Quando viene contattato con tale richiesta, il servente mette l'argomento ricevuto nel membro `migrations_memory`
dell'istanza di `CoordGnodeMemory` associata alla chiave `k`.  
Poi in una nuova tasklet avvia le operazioni di replica.  
Quando viene chiamato nel modulo Coordinator il metodo `get_migrations_memory(int level)`
(dopo che ha verificato di essere in esecuzione sul nodo Coordinator) questo crea una istanza
del CoordinatorClient sulla quale chiama l'omonimo metodo. Questi avvia il
contatto con il servizio Coordinator per la chiave `k.lvl = level`. Quando viene contattato con tale richiesta,
il servente restituisce l'oggetto che è nel membro `migrations_memory` dell'istanza di `CoordGnodeMemory`
associata alla chiave `k`.

Abbiamo detto che i metodi `set_migrations_memory` e `get_migrations_memory` possono essere eseguiti
solo nello stesso nodo Coordinator. Però l'accesso alla memoria condivisa del g-nodo deve avvenire
comunque attraverso i meccanismi del modulo PeerServices, di modo che venga richiamato
il metodo `fixed_keys_db_on_request` e venga garantita la coerenza dei dati.  
Non è compito del modulo Coordinator, ma bensì del modulo Migrations, garantire l'atomicità delle
sue operazioni. Ad esempio se vuole fare operazioni che prevedono la lettura e la successiva elaborazione
e scrittura di questa memoria, esso può acquisire dei *lock* su tutte le parti del suo codice che accedono
a questa memoria. Questo dovrebbe essere sufficiente: infatti il nodo che fa queste operazioni è solo uno
e il modulo che le esegue è solo il modulo Migrations.

### <a name="Richieste_previste"></a>Richieste previste

Ricordiamo che il servizio Coordinator mantiene un database distribuito della tipologia a chiavi fisse.
Per questo esso definisce una classe CoordinatorService.DatabaseDescriptor che implementa i metodi
dell'interfaccia [IDatabaseDescriptor](../ModuloPeers/DettagliTecnici.md#Mantenimento_database_distribuito)
e quelli dell'interfaccia [IFixedKeysDatabaseDescriptor](../ModuloPeers/DatabaseFixedKeys.md#Algoritmi).

Quando il servente riceve una richiesta, cioè nel metodo `exec` che deve implementare la
classe servente CoordinatorService, questi deve chiamare il metodo `fixed_keys_db_on_request`
del modulo PeerServices.  
Il riconoscimento delle singole richieste che si possono fare al Coordinator dovrà quindi avvenire
nel metodo `execute` della classe CoordinatorService.DatabaseDescriptor.

Elenchiamo tutte le richieste che si possono fare al Coordinator.

#### <a name="Numero_nodi_nella_rete"></a>Numero di nodi nella rete

Una richiesta *r* di tipo NumberOfNodesRequest fatta al Coordinator dell'intera rete *G* indica che
il client del servizio (avendo incontrata una diversa rete di dimensioni simili a questa) vuole
chiedere alla rete (come entità atomica) quanti sono i suoi nodi.  
Questa richiesta non è di tipo *read-only*, bensì di tipo *update*. In seguito si capirà perché.  
Si vedano i tipi descritti sulla sezione
[requisiti comuni](../ModuloPeers/DettagliTecnici.md#Mantenimento_database_distribuito_Requisiti_comuni)
del documento del modulo PeerServices, che descrive un servizio che mantiene un database distribuito.

La classe della richiesta non ha alcun membro.

Il motivo per cui viene fatta questa richiesta al Coordinator di *G* è perché i singoli nodi
della rete hanno una valutazione approssimativa. Supponiamo che ci siano le reti *G* e *J*.
Due distinte coppie di nodi *n<sub>0</sub>-v<sub>0</sub>* e *n<sub>1</sub>-v<sub>1</sub>* si
incontrano con *n<sub>i</sub>* ∈ *G* e *v<sub>i</sub>* ∈ *J*.
Quando si incontrano due nodi di reti diverse questi si accordano affinché una rete cerchi
un posto nell'altra per entrare, di modo che si fondano in una sola. La regola impone che la
rete più piccola cerchi di entrare nella più grande. Ma supponiamo che le dimensioni di *G* e *J* sono
simili, per questo succede che *n<sub>0</sub>-v<sub>0</sub>* stabiliscono che *G* deve entrare
in *J* mentre *n<sub>1</sub>-v<sub>1</sub>* ritengono meglio l'inverso. Per evitare questo
disaccordo entrambe le coppie interrogano il Coordinator.

Però avremo che *n<sub>0</sub>* interroga il Coordinator al tempo *t* e *n<sub>1</sub>* lo
interroga al tempo *t+1* e il valore potrebbe essere cambiato. Questo renderebbe di nuovo
possibile il disaccordo di prima.

Quindi, il nodo Coordinator prima di rispondere sulla base delle conoscenze che ha acquisito con
il Qspn, accede alla memoria condivisa di *G* per vedere se non abbia da poco risposto ad una
precedente richiesta e in questo caso risponde con lo stesso valore di prima.

In ogni caso, il nodo Coordinator prima di rispondere accede in scrittura (con relativa nuova
tasklet che si occupa delle repliche) alla memoria condivisa di *G* per salvare la risposta che
sta per dare, con un relativo timer di scadenza.

Sono i dati memorizzati nei membri `n_nodes` e `n_nodes_timeout` della classe CoordGnodeMemory.

La risposta va restituita al client del servizio attraverso una istanza di NumberOfNodesResponse.  
La classe ha un solo membro:

*   `int n_nodes`

#### <a name="Valuta_ingresso"></a>Valuta un ingresso

Una richiesta *r* di tipo EvaluateEnterRequest fatta al Coordinator dell'intera rete indica che è stata incontrata una diversa rete *J*
sulla quale il singolo nodo *n* (il client del servizio) suggerisce di fare ingresso.  
Questa richiesta non è di nessuno dei tipi classici: *insert*, *read-only*, *update*, *replica-valore*,
*replica-cancellazione*.

I membri di *r* contengono informazioni che non sono di pertinenza del modulo Coordinator.
Il contenuto della richiesta può essere semplicemente:

*   `int lvl` - il livello del g-nodo il cui Coordinator deve essere interpellato.  
    Per questo metodo specifico avremo sempre `lvl` = `levels`, ma questo è un dettaglio non di pertinenza del modulo Coordinator.
*   `Object evaluate_enter_data` = un oggetto serializzabile la cui classe è nota al delegato IEvaluateEnterHandler.

Questa richiesta viene fatta al servizio Coordinator, in particolare al Coordinator dell'intera rete, per
fare in modo che sia "tutta la rete" come entità atomica a venire interpellata.

Sebbene la richiesta venga fatta come detto al Coordinator della rete *G*,
in effetti la strategia di ingresso non è di pertinenza del modulo Coordinator. Per questo viene utilizzato
un delegato passato al modulo dal suo utilizzatore sotto forma di una istanza dell'interfaccia IEvaluateEnterHandler
che ha il metodo `evaluate_enter`.

La risposta ottenuta dal delegato contiene informazioni che non sono di pertinenza del modulo Coordinator.
Si tratta di una istanza di Object che sappiamo essere serializzabile.

Il risultato va restituito così com'è al client del servizio attraverso una istanza di EvaluateEnterResponse.  
Essa ha il solo membro:

*   `Object evaluate_enter_result`.

#### <a name="Avvio_ingresso"></a>Avvio ingresso in altra rete

Una richiesta *r* di tipo BeginEnterRequest fatta al nodo Coordinator di *g* indica che il singolo nodo *n*
(il client del servizio) vuole essere autorizzato ad avviare le operazioni di ingresso del g-nodo *g* in
una diversa rete *J*.  
Questa richiesta non è di nessuno dei tipi classici: *insert*, *read-only*, *update*, *replica-valore*,
*replica-cancellazione*.

I membri di *r* sono:

*   `lvl` = livello di *g*.
*   `Object begin_enter_data` = un oggetto serializzabile la cui classe è nota al delegato IBeginEnterHandler.  
    Contiene informazioni che non sono di pertinenza del modulo Coordinator.

Questa richiesta viene fatta al servizio Coordinator, in particolare al Coordinator di *g*, per
fare in modo che sia *g* come entità atomica a venire interpellata.

Per rispondere, non essendo questa materia di competenza del modulo Coordinator, viene utilizzato
un delegato passato al modulo dal suo utilizzatore sotto forma di una istanza dell'interfaccia IBeginEnterHandler
che ha il metodo `begin_enter`.

La risposta ottenuta dal delegato contiene informazioni che non sono di pertinenza del modulo Coordinator.
Si tratta di una istanza di Object che sappiamo essere serializzabile.

Il risultato va restituito così com'è al client del servizio attraverso una istanza di BeginEnterResponse.  
Essa ha il solo membro:

*   `Object begin_enter_result`.

#### <a name="Confermato_ingresso"></a>Confermato ingresso in altra rete

Una richiesta/segnalazione *r* di tipo CompletedEnterRequest fatta al nodo Coordinator di *g* dal singolo nodo *n*
(il client del servizio) indica che sono state completate le operazioni di ingresso del g-nodo *g* in una diversa rete.  
Questa richiesta non è di nessuno dei tipi classici: *insert*, *read-only*, *update*, *replica-valore*,
*replica-cancellazione*.

I membri di *r* sono:

*   `lvl` = livello di *g*.
*   `Object completed_enter_data` = un oggetto serializzabile la cui classe è nota al delegato ICompletedEnterHandler.  
    Contiene informazioni che non sono di pertinenza del modulo Coordinator.

Questa richiesta viene fatta al servizio Coordinator, in particolare al Coordinator di *g*, perché esso
era stato interpellata all'avvio delle operazioni (con la richiesta).

Per rispondere, non essendo questa materia di competenza del modulo Coordinator, viene utilizzato
un delegato passato al modulo dal suo utilizzatore sotto forma di una istanza dell'interfaccia ICompletedEnterHandler
che ha il metodo `completed_enter`.

La risposta ottenuta dal delegato contiene informazioni che non sono di pertinenza del modulo Coordinator.
Si tratta di una istanza di Object che sappiamo essere serializzabile.

Il risultato va restituito così com'è al client del servizio attraverso una istanza di CompletedEnterResponse.  
Essa ha il solo membro:

*   `Object completed_enter_result`.

#### <a name="Get_migrations_memory"></a>Lettura memoria di pertinenza del modulo Migrations

Le richieste esposte sopra (EvaluateEnter, BeginEnter, CompletedEnter) fanno sì che il modulo Migrations in
esecuzione in un qualsiasi singolo nodo possa far eseguire alcuni suoi metodi nel nodo Coordinator di tutta la rete
o di un g-nodo.  
L'altro requisito è che tali metodi in esecuzione nel nodo Coordinator possano accedere in lettura/scrittura
alla memoria condivisa di tutta la rete, per alcune informazioni di pertinenza del modulo Migrations.  
A questo servono le due richieste che ora esponiamo, GetMigrationsMemory e SetMigrationsMemory.

Queste richieste possono essere avviate solo dallo stesso nodo Coordinator. Infatti le richieste viste sopra
(EvaluateEnter, BeginEnter, CompletedEnter) non coinvolgono mai il reperimento di un record: cioè non sono
di tipo *insert*, *read-only*, *update*, *replica-valore* o *replica-cancellazione*. Quindi non saranno demandate
ad altri nodi, nemmeno nel caso in cui il Coordinator non è ancora esaustivo.

Invece le richieste GetMigrationsMemory e SetMigrationsMemory sono una di tipo *read-only* e l'altra di tipo
*update*. Quindi è possibile (sebbene molto raramente, solo nel caso in cui il nodo Coordinator appena raggiunto
dalla richiesta sia ancora non esaustivo) che esse comportino operazioni di trasmissione in rete.  
In questo caso abbiamo che l'operazione GetMigrationsMemory potrebbe essere servita da un altro nodo. Mentre
per l'operazione SetMigrationsMemory essa sarà servita dal nodo Coordinator ma questi potrebbe potrebbe avere
la necessità di espletare prima le operazioni di recupero del record.

Una richiesta *r* di tipo GetMigrationsMemoryRequest fatta sul g-nodo *g* indica che il client del servizio
(in questo caso lo stesso nodo Coordinator di *g*) chiede la porzione di pertinenza del modulo Migrations
della memoria condivisa di *g*.  
Questa richiesta è di tipo *read-only*.

*   `int lvl` = livello di *g*.

Il nodo servente deve recuperare la risposta dal membro `migrations_memory` dell'istanza di `CoordGnodeMemory`
associata al livello `lvl`.

La risposta viene comunicata al client del servizio attraverso una istanza di GetMigrationsMemoryResponse, che ha
il membro:

*   `Object migrations_memory`

#### <a name="Set_migrations_memory"></a>Scrittura memoria di pertinenza del modulo Migrations

Una richiesta *r* di tipo SetMigrationsMemoryRequest fatta sul g-nodo *g* indica che il client del servizio
(in questo caso lo stesso nodo Coordinator di *g*) vuole scrivere sulla porzione di pertinenza del modulo Migrations
della memoria condivisa di *g*.  
Questa richiesta è di tipo *update*.

*   `int lvl` = livello di *g*.
*   `Object migrations_memory`

Il nodo servente scrive nel membro `migrations_memory` dell'istanza di `CoordGnodeMemory` associata al livello `lvl`.

La richiesta SetMigrationsMemoryRequest dovrebbe essere avviata solo dallo stesso nodo Coordinator. E la risposta,
sebbene sia possibile che prima di processarla siano state fatte le operazioni di reperimento del record, dovrebbe
provenire dallo stesso nodo.  
Perciò, prima di operare, il nodo servente verifica che la richiesta venga da se stesso. Questo può farlo
guardando l'argomento `Gee.List<int> client_tuple` che riceve in quanto richiamato dal metodo astratto
`exec` di PeerService. Esso dovrebbe essere vuoto.

Dopo aver effettuato la scrittura, prima di rispondere al client, come sempre il servente deve avviare
una tasklet che si occupi di replicare la scrittura nei nodi replica.

L'avvenuta scrittura viene comunicata al client del servizio attraverso una istanza di SetMigrationsMemoryResponse, che è vuota.

#### <a name="Prenota_un_posto"></a>Prenota un posto

Una richiesta *r* di tipo ReserveEnterRequest fatta al nodo Coordinator di *g* indica che il singolo nodo *n*
(il client del servizio) chiede la prenotazione di un posto in *g*.  
Questa richiesta è di tipo *update*.

*   `int lvl` = livello di *g*.
*   `int reserve_request_id` = un identificativo di questa prenotazione.

Questa richiesta viene fatta al servizio Coordinator, in particolare al Coordinator di *g*, per
fare in modo che sia *g* come entità atomica a venire interpellata.

Per prima cosa il nodo Coordinator di *g* accede alla memoria condivisa di *g*. Nel membro `reserve_list`
dell'istanza di `CoordGnodeMemory` associata al livello `lvl` vengono memorizzate
in una lista di Booking le prenotazioni pendenti con la scadenza associata. Le prenotazioni scadute vengono adesso
rimosse.

Poi il nodo Coordinator di *g* guarda se esiste già una prenotazione con l'identificativo `reserve_request_id`. In
questo caso gli stessi valori `new_pos` e `new_eldership` saranno restituiti al client. Altrimenti si prosegue.

Il nodo Coordinator di *g* accede alla propria mappa di percorsi, per vedere quali g-nodi
di livello `lvl` - 1 (oltre a quello a cui esso stesso appartiene) dentro al suo g-nodo di livello
`lvl` sono già presenti come destinazioni, quindi esistenti nella rete e non assegnabili alla
nuova richiesta. Cioè usa il metodo `get_free_pos` di ICoordinatorMap.

Questo non basta: il Coordinator di *g* guarda ancora le prenotazioni pendenti nella memoria condivisa
di *g*, quelle cioè assegnate in precedenza anche se ancora nessun ETP ha fatto
sì che venissero salvate come destinazioni nella propria mappa di percorsi. Anche queste non sono assegnabili alla
nuova richiesta.

Se nessuna posizione *reale* è assegnabile viene scelto il prossimo valore *virtuale* come posizione.
Il valore più alto finora assegnato è anch'esso memorizzato nell'istanza di `CoordGnodeMemory` associata
al livello `lvl`, nel membro `max_virtual_pos`. Viene incrementato di 1.

Infine viene assegnata l'anzianità. Anche qui incrementando di 1 il precedente valore massimo, il
quale è memorizzato nell'istanza di `CoordGnodeMemory` associata
al livello `lvl`, nel membro `max_eldership`.

Avendo così scelto nuovi valori `new_pos` e `new_eldership` il nodo Coordinator di *g* prepara una nuova
istanza di Booking con la scadenza default. E la aggiunge alla lista `reserve_list`.

Prima di rispondere al client, in entrambi i casi esaminati (cioè se esisteva già un Booking con
l'identificativo `reserve_request_id` oppure se è stata scelta una nuova posizione e quindi creata una nuova istanza
di Booking) il nodo Coordinator di *g* accede in scrittura alla memoria condivisa di *g*.  
Questo come sappiamo comporta l'avvio di una tasklet che si occupi di replicare la scrittura nei nodi replica.

Il risultato della prenotazione è composto dalla nuova posizione e dalla sua anzianità. Esso viene
restituito al client del servizio attraverso una istanza di ReserveEnterResponse:

*   `int new_pos`
*   `int new_eldership`

Il nodo client del servizio è un nodo *n* che già appartiene al g-nodo *g*. Questi richiede la prenotazione di
un nuovo posto per conto di un altro nodo suo vicino, *m*, il quale non è ancora in *g* o perfino non è ancora
nella rete.  
Ricevuta la risposta dal nodo Coordinator, *n* la comunica al vicino *m*. Ma di questo si occupa il
modulo Migrations.

#### <a name="Cancella_prenotazione"></a>Cancella prenotazione

Una richiesta *r* di tipo DeleteReserveEnterRequest fatta al nodo Coordinator di *g* indica che il singolo nodo *n*
(il client del servizio) chiede la rimozione di una prenotazione pendente in *g*.  
Questa richiesta è di tipo *update*.

*   `int lvl` = livello di *g*.
*   `int reserve_request_id` = l'identificativo della prenotazione pendente da rimuovere.

Il Coordinator di *g* accede alla memoria condivisa di *g*. Nel membro `reserve_list` dell'istanza
di `CoordGnodeMemory` associata al livello `lvl` (come illustrato più sotto) vengono memorizzate
le prenotazioni pendenti, quelle cioè assegnate in precedenza anche se ancora nessun ETP ha fatto
sì che venissero salvate come destinazioni nella propria mappa di percorsi.

Trovata la prenotazione da rimuovere, il Coordinator di *g* accede in scrittura alla memoria condivisa di *g*. Questo come
sappiamo comporta l'avvio di una tasklet che si occupi di replicare la scrittura nei nodi replica.

L'avvenuta rimozione (anche nel caso non si fosse trovata affatto la prenotazione pendente) viene
comunicata al client del servizio attraverso una istanza di DeleteReserveEnterResponse, che è vuota.

#### <a name="Replica"></a>Replica memoria condivisa

Una richiesta *r* di tipo ReplicaRequest con livello *lvl* che giunge a un nodo servente del servizio Coordinator,
il quale appartiene al g-nodo *g* di livello *lvl*, indica che il nodo client della richiesta
chiede la replica di un record nella memoria condivisa di *g*.  
Questa richiesta è di tipo *replica-valore*.

La classe ReplicaRequest contiene:

*   `int lvl`
*   `CoordGnodeMemory memory`

In realtà, come in tutte le repliche, il nodo client in questo caso è il nodo con indirizzo
attualmente più prossimo alla tupla del Coordinator di *g*, mentre il nodo servente è uno dei nodi
che potrebbero trovarsi in sua assenza a rispondere alle future richieste.

Il servente dovrà copiare `memory` nella sua memoria come istanza di `CoordGnodeMemory`
associata al livello `lvl`.

La risposta è una istanza di ReplicaResponse, che non ha membri.

## <a name="Requisiti"></a>Requisiti

Il nodo quando crea una identità (la prima per avviare il sistema o le successive a seguito di ingressi
o migrazioni) crea una istanza del modulo Coordinator fornendo:

*   Se si tratta di una identità successiva alla prima, per ingresso o migrazione, il livello del g-nodo
    (di cui l'identità fa parte come nodo) che compie in blocco questa operazione
*   Il livello a cui si è costituito un g-nodo nuovo.
*   Se si tratta di una identità successiva alla prima, un riferimento alla istanza precedente del modulo Coordinator.

Durante le sue operazioni, il modulo viene informato quando il nodo ha completato la fase di bootstrap.
In quello stesso momento gli vengono forniti:

*   L'istanza di PeersManager.
*   Mappa delle posizioni libere/occupate ai vari livelli.

## <a name="Deliverables"></a>Deliverables

### <a name="Deliverables_manager"></a>Implementazione di CoordinatorManager

Il modulo Coordinator fornisce nella classe CoordinatorManager metodi proxy per la sua collaborazione con il modulo Migrations.  
Il loro utilizzo è questo: l'utilizzatore del modulo
Coordinator nel nodo *n* passa un oggetto al metodo `xyz` (su istruzione
del modulo Migrations); questi fa pervenire questo oggetto al nodo Coordinator (di tutta la
rete o di un g-nodo) il quale lo passa ad un particolare delegato per il metodo `xyz`. Questo delegato
implementato dall'utilizzatore del modulo richiama il metodo `xyz` nel modulo Migrations.

*   `Object evaluate_enter(int lvl, Object evaluate_enter_data)`.  
    L'esecuzione del metodo `evaluate_enter` del modulo Migrations nel nodo Coordinator produce la
    decisione per il nodo *n* di tentare o meno l'ingresso nella nuova rete e se sì a quale livello.
*   `Object begin_enter(int lvl, Object begin_enter_data)`.  
    L'esecuzione del metodo `begin_enter` del modulo Migrations nel nodo Coordinator autorizza
    o nega l'ingresso.
*   `Object completed_enter(int lvl, Object completed_enter_data)`.  
    L'esecuzione del metodo `completed_enter` del modulo Migrations nel nodo Coordinator segnala
    il completamento dell'ingresso.

Il modulo Coordinator fornisce inoltre nella classe CoordinatorManager metodi di accesso alla
memoria condivisa di pertinenza del modulo Migrations.
Questi sono usati solo nel nodo Coordinator (di tutta la rete o di un g-nodo) che sta eseguendo
uno dei metodi visti prima.

*   `get_migrations_memory` e `set_migrations_memory`.  
    Per accedere alla memoria condivisa di pertinenza del modulo Migrations.

Il modulo Coordinator fornisce inoltre nella classe CoordinatorManager metodi di propagazione per
la sua collaborazione con il modulo Migrations.  
Il loro utilizzo è questo: l'utilizzatore del modulo Coordinator nel nodo *n* chiama il metodo `xyz(l)`
su istruzione del modulo Migrations. Questi raccoglie delle informazioni che servono a identificare
i nodi che appartengono al suo stesso g-nodo di livello `l`, come ad esempio la tupla delle
posizioni, l'identificativo del fingerprint, ecc. Poi chiama un omonimo metodo remoto su degli
oggetti stub (diversi di tipo unicast oppure uno di tipo broadcast a seconda che sia prevista una
risposta o meno). I metodi remoti nei nodi riceventi verificano sulla base delle suddette informazioni
se fanno parte del g-nodo di propagazione. In caso positivo questi chiamano
un particolare delegato per il metodo `xyz`. Questo delegato
implementato dall'utilizzatore del modulo richiama il metodo `xyz` nel modulo Migrations.

*   `prepare_migration()`.  
    **TODO**.
*   `finish_migration()`.  
    **TODO**.

Il modulo Coordinator fornisce inoltre nella classe CoordinatorManager i seguenti metodi:

*   `get_n_nodes()`.  
    Per chiedere al Coordinator della rete il numero di singoli nodi.
*   `reserve(int lvl, int reserve_request_id)`.  
    Per riservare un posto nel proprio g-nodo.
*   `delete_reserve(int lvl, int reserve_request_id)`.  
    Per cancellare la prenotazione di un posto nel proprio g-nodo.

#### Metodo evaluate_enter

Quando il modulo Migrations del nodo *n* vuole far eseguire il suo metodo `evaluate_enter` nel nodo Coordinator
della rete *G*, richiama il metodo `evaluate_enter` del modulo Coordinator.

Gli argomenti di questo metodo sono:

*   `int lvl` - il livello del g-nodo il cui Coordinator deve essere interpellato.  
    Per questo metodo specifico avremo sempre `lvl` = `levels`, ma questo è un dettaglio di pertinenza
    del modulo Migrations, quindi lo passiamo al modulo Coordinator come argomento.
*   `Object evaluate_enter_data` - la struttura dati serializzabile che contiene l'input del metodo
    da eseguire nel nodo Coordinator.

L'esecuzione di `evaluate_enter` del modulo Coordinator consiste in questo:

Viene preparato un client del servizio Coordinator. Cioè una istanza di CoordinatorClient.

Su questo CoordinatorClient viene chiamato il metodo `evaluate_enter` passando tutti gli argomenti ricevuti dal
metodo `evaluate_enter` del modulo Coordinator.

Il CoordinatorClient prepara una richiesta *r* = [EvaluateEnterRequest](#Valuta_ingresso)
che comprende i dati di cui sopra: sia i dati di input, sia il livello cioè la chiave da usare nel
database distribuito del servizio Coordinator.

Poi il CoordinatorClient invia la richiesta *r* al servente per la chiave *lvl* e ottiene una risposta che è una istanza di
EvaluateEnterResponse. Essa contiene:

*   `Object evaluate_enter_result` - la struttura dati serializzabile che contiene l'output del metodo
    eseguito nel nodo Coordinator.

Questo Object serializzabile è quello che il metodo `evaluate_enter` del CoordinatorClient restituisce al chiamante.  
Ed è quello che il metodo `evaluate_enter` del modulo Coordinator restituisce al chiamante.

#### Metodo begin_enter

Quando il modulo Migrations del nodo *n* vuole far eseguire il suo metodo `begin_enter` nel nodo Coordinator
del suo g-nodo *g* di livello *lvl*, richiama il metodo `begin_enter` del modulo Coordinator.

Gli argomenti di questo metodo sono:

*   `int lvl` - il livello del g-nodo il cui Coordinator deve essere interpellato.
*   `Object begin_enter_data` - la struttura dati serializzabile che contiene l'input del metodo
    da eseguire nel nodo Coordinator.

L'esecuzione di `begin_enter` del modulo Coordinator consiste in questo:

Viene preparato un client del servizio Coordinator. Cioè una istanza di CoordinatorClient.

Su questo CoordinatorClient viene chiamato il metodo `begin_enter` passando tutti gli argomenti ricevuti dal
metodo `begin_enter` del modulo Coordinator.

Il CoordinatorClient prepara una richiesta *r* = [BeginEnterRequest](#Avvio_ingresso)
che comprende i dati di cui sopra: sia i dati di input, sia il livello cioè la chiave da usare nel
database distribuito del servizio Coordinator.

Poi il CoordinatorClient invia la richiesta *r* al servente per la chiave *lvl* e ottiene una risposta che è una istanza di
BeginEnterResponse. Essa contiene:

*   `Object begin_enter_result` - la struttura dati serializzabile che contiene l'output del metodo
    eseguito nel nodo Coordinator.

Questo Object serializzabile è quello che il metodo `begin_enter` del CoordinatorClient restituisce al chiamante.  
Ed è quello che il metodo `begin_enter` del modulo Coordinator restituisce al chiamante.

#### Metodo completed_enter

Quando il modulo Migrations del nodo *n* vuole far eseguire il suo metodo `completed_enter` nel nodo Coordinator
del suo g-nodo *g* di livello *lvl*, richiama il metodo `completed_enter` del modulo Coordinator.

Gli argomenti di questo metodo sono:

*   `int lvl` - il livello del g-nodo il cui Coordinator deve essere interpellato.
*   `Object completed_enter_data` - la struttura dati serializzabile che contiene l'input del metodo
    da eseguire nel nodo Coordinator.

L'esecuzione di `completed_enter` del modulo Coordinator consiste in questo:

Viene preparato un client del servizio Coordinator. Cioè una istanza di CoordinatorClient.

Su questo CoordinatorClient viene chiamato il metodo `completed_enter` passando tutti gli argomenti ricevuti dal
metodo `completed_enter` del modulo Coordinator.

Il CoordinatorClient prepara una richiesta *r* = [CompletedEnterRequest](#Confermato_ingresso)
che comprende i dati di cui sopra: sia i dati di input, sia il livello cioè la chiave da usare nel
database distribuito del servizio Coordinator.

Poi il CoordinatorClient invia la richiesta *r* al servente per la chiave *lvl* e ottiene una risposta che è una istanza di
CompletedEnterResponse. Essa contiene:

*   `Object completed_enter_result` - la struttura dati serializzabile che contiene l'output del metodo
    eseguito nel nodo Coordinator.

Questo Object serializzabile è quello che il metodo `completed_enter` del CoordinatorClient restituisce al chiamante.  
Ed è quello che il metodo `completed_enter` del modulo Coordinator restituisce al chiamante.

#### Metodi get_migrations_memory e set_migrations_memory

Il metodo `Object get_migrations_memory(int lvl)` e il metodo `void set_migrations_memory(int lvl, Object data)`
del modulo Coordinator vengono chiamati per accedere alla memoria condivisa del proprio g-nodo
di livello *lvl* di pertinenza del modulo Migrations.  
Come detto sopra, possono essere richiamati solo nel nodo Coordinator, quindi prevedono l'eccezione
NotCoordinatorNodeError.

Si vedano i commenti [qui](#Records_modulo_migrations).

#### Metodo prepare_migration

**TODO**

#### Metodo finish_migration

**TODO**

#### Metodo get_n_nodes

Il metodo `get_n_nodes` del modulo Coordinator viene chiamato per chiedere al nodo Coordinator della rete il numero
di singoli nodi in essa.

Il metodo, attraverso la classe client del servizio CoordinatorClient, invia una richiesta
[NumberOfNodesRequest](#Numero_nodi_nella_rete) al nodo Coordinator di *G*. La risposta è un intero.

#### Metodo reserve

Il metodo `reserve` del modulo Coordinator viene chiamato per prenotare un posto (se possibile *reale*, altrimenti *virtuale*)
nel proprio g-nodo di livello *lvl*.

Gli argomenti di questo metodo sono:

*   `int lvl` - il livello del g-nodo in cui prenotare un posto.
*   `int reserve_request_id` - l'identificativo della richiesta di prenotazione.

Il metodo, attraverso la classe client del servizio CoordinatorClient, invia una richiesta
[ReserveEnterRequest](#Prenota_un_posto) al nodo Coordinator del g-nodo *g* di livello *lvl*.
La risposta è una istanza di ReserveEnterResponse che contiene `int new_pos` e `int new_eldership`.

L'oggetto restituito dal metodo è quindi di una classe Reservation che contiene queste informazioni.

#### Metodo delete_reserve

Il metodo `delete_reserve` del modulo Coordinator viene chiamato per cancellare la prenotazione di un posto
nel proprio g-nodo di livello *lvl*. Viene usato solo per prenotazioni che avevano avuto come risultato
un posto *reale*.

Gli argomenti di questo metodo sono:

*   `int lvl` - il livello del g-nodo in cui prenotare un posto.
*   `int reserve_request_id` - l'identificativo della richiesta di prenotazione.

Il metodo, attraverso la classe client del servizio CoordinatorClient, invia una richiesta
[DeleteReserveEnterRequest](#Cancella_prenotazione) al nodo Coordinator del g-nodo *g* di livello *lvl*.
La risposta è una istanza di DeleteReserveEnterResponse che non contiene membri.

Il metodo restituisce `void`.

### <a name="Deliverables_service"></a>Implementazione di CoordinatorService e di CoordinatorClient

Il modulo Coordinator implementa il servizio Coordinator derivando la classe CoordinatorService dalla classe base PeerService.

Il modulo Coordinator si occupa di registrare con il PeersManager l'implementazione del servizio Coordinator
e di usare gli algoritmi forniti dal modulo PeerServices per il mantenimento del relativo database a chiavi
fisse. Cioè, esso crea una istanza della classe CoordinatorService.DatabaseDescriptor, che implementa
IFixedKeysDatabaseDescriptor. Con questa istanza come parametro, al bisogno, chiama i metodi
`fixed_keys_db_on_startup` e `fixed_keys_db_on_request` di PeersManager.

Il modulo Coordinator deriva la classe CoordinatorClient dalla classe base PeerClient per avviare il contatto del Coordinator
di un suo g-nodo e richiederne i servizi.  
I metodi della classe CoordinatorClient sono:

*   `int get_n_nodes()` - chiede al Coordinator della rete il numero di nodi in tutta la rete.  
    Vedi la relativa [richiesta](#Numero_nodi_nella_rete).
*   `Object evaluate_enter(int lvl, Object evaluate_enter_data)` -
    chiede al Coordinator del g-nodo di livello *lvl* di eseguire il delegato del metodo.  
    Vedi la relativa [richiesta](#Valuta_ingresso).
*   `Object begin_enter(int lvl, Object begin_enter_data)` -
    chiede al Coordinator del g-nodo di livello *lvl* di eseguire il delegato del metodo.  
    Vedi la relativa [richiesta](#Avvio_ingresso).
*   `Object completed_enter(int lvl, Object completed_enter_data)` -
    chiede al Coordinator del g-nodo di livello *lvl* di eseguire il delegato del metodo.  
    Vedi la relativa [richiesta](#Confermato_ingresso).
*   `Object get_migrations_memory(int lvl)` - recupera la porzione di dati di pertinenza
    del modulo Migrations nella memoria condivisa del g-nodo di livello *lvl*.  
    Vedi la relativa [richiesta](#Get_migrations_memory).
*   `void set_migrations_memory(Object data, int lvl)` - modifica la porzione di dati di pertinenza
    del modulo Migrations nella memoria condivisa del g-nodo di livello *lvl*.  
    Vedi la relativa [richiesta](#Set_migrations_memory).
*   `void reserve(int lvl, int reserve_request_id, out int new_pos, out int new_eldership)` -
    chiede al Coordinator del g-nodo di livello *lvl* di riservare un posto.  
    Vedi la relativa [richiesta](#Prenota_un_posto).
*   `void delete_reserve(int lvl, int reserve_request_id)` -
    chiede al Coordinator del g-nodo di livello *lvl* di eliminare la prenotazione di un posto.  
    Vedi la relativa [richiesta](#Cancella_prenotazione).
*   `void make_replicas(int lvl)` - dopo aver apportato delle variazioni al contenuto della
    memoria condivisa del g-nodo di livello *lvl*, il nodo attuale Coordinator avvia una
    nuova tasklet e su questa chiama su una sua istanza di classe client questo metodo.  
    L'implementazione di questo metodo non chiama il metodo protetto `call` della classe base
    PeerClient, come avviene per gli altri metodi di CoordinatorClient, bensì chiama
    i metodi `begin_replica` e `next_replica` del modulo PeerServices.
    Vedi la relativa [richiesta](#Replica).

## <a name="Classi_e_interfacce"></a>Classi e interfacce

### <a name="Classi_Interfacce">Interfacce

La mappa delle posizioni *reali* libere ai vari livelli è un oggetto fornito dall'utilizzatore del
modulo Coordinator. Di questo oggetto il modulo conosce l'interfaccia ICoordinatorMap. Tramite essa
il modulo può:

*   Leggere il numero *l* dei livelli della topologia (metodo `get_levels`).
*   Leggere la gsize di ogni livello *i* da 0 a *l* - 1 (metodo `get_gsize`).  
    Precisiamo il significato di questo indice, restando coerenti con quanto stabilito nella
    trattazione del modulo QSPN sebbene i due moduli siano indipendenti.  
    Per ogni *i* da 0 a *l* - 1, *gsize(i)* è il numero massimo di g-nodi di livello *i* in un
    g-nodo di livello *i* + 1.
*   Leggere il numero approssimativo di singoli nodi nella rete secondo le conoscenze acquisite
    dal nodo corrente tramite il Qspn (metodo `get_n_nodes`).  
*   Leggere l'elenco delle posizioni *reali* libere in ogni livello *i* da 0 a *l* - 1 (metodo `get_free_pos`).  
    Cioè le posizioni nel livello *i* che sono libere nel nostro g-nodo di livello *i* + 1. Questa
    informazione è quella che si basa sulla mappa del nodo corrente, senza considerare
    le prenotazioni pendenti.

### <a name="Classi_Delegati">Delegati

L'interfaccia IEvaluateEnterHandler è definita dal modulo Coordinator.  
Una istanza di tale interfaccia viene passata al modulo Coordinator nel costruttore. Di tale istanza viene
usato il metodo `evaluate_enter` quando si riceve una richiesta EvaluateEnterRequest. L'oggetto restituito
dal metodo serve al modulo per produrre l'istanza di EvaluateEnterResponse da restituire.

I metodi previsti dall'interfaccia IEvaluateEnterHandler sono:

*   `Object evaluate_enter(int lvl, Object evaluate_enter_data)`  
    Con questo metodo si richiama il metodo `evaluate_enter` del modulo Migrations. Vedi [qui](../ModuloMigrations/AnalisiFunzionale.md).

* * *

L'interfaccia IBeginEnterHandler è definita dal modulo Coordinator.  
Una istanza di tale interfaccia viene passata al modulo Coordinator nel costruttore. Di tale istanza viene
usato il metodo `begin_enter` quando si riceve una richiesta BeginEnterRequest. L'oggetto restituito
dal metodo serve al modulo per produrre l'istanza di BeginEnterResponse da restituire.

I metodi previsti dall'interfaccia IBeginEnterHandler sono:

*   `Object begin_enter(int lvl, Object begin_enter_data)`  
    Con questo metodo si richiama il metodo `begin_enter` del modulo Migrations. Vedi [qui](../ModuloMigrations/AnalisiFunzionale.md).

* * *

L'interfaccia ICompletedEnterHandler è definita dal modulo Coordinator.  
Una istanza di tale interfaccia viene passata al modulo Coordinator nel costruttore. Di tale istanza viene
usato il metodo `completed_enter` quando si riceve una richiesta CompletedEnterRequest. L'oggetto restituito
dal metodo serve al modulo per produrre l'istanza di CompletedEnterResponse da restituire.

I metodi previsti dall'interfaccia ICompletedEnterHandler sono:

*   `Object completed_enter(int lvl, Object completed_enter_data)`  
    Con questo metodo si richiama il metodo `completed_enter` del modulo Migrations. Vedi [qui](../ModuloMigrations/AnalisiFunzionale.md).

### <a name="Classi_Strutture">Strutture dati

La classe Reservation è definita per contenere le informazioni restituite dal metodo `reserve` fornito nella
classe CoordinatorManager.

Essa contiene:

*   `int new_pos` - la posizione del g-nodo appena riservato nel suo g-nodo superiore.  
    Considerato che il metodo `reserve` ha come argomento `lvl`, il valore di `new_pos`
    è la posizione al livello `lvl`-1.
*   `int new_eldership` - l'anzianità del g-nodo appena riservato nel suo g-nodo superiore.

Non è necessario che questo oggetto sia serializzabile. Infatti il metodo viene richiamato dall'utilizzatore
del modulo Coordinator nel nodo stesso. Questi, attraverso qualche meccanismo, sarà stato provocato
dal modulo Migrations che poi riceverà questo oggetto. Se dovrà, come si suppone, passare ad un altro
nodo le informazioni necessarie per fare un ingresso, si preoccuperà esso stesso di mettere
tutti i dati necessari (questi e altri) in un oggetto serializzabile.

