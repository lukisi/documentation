# Modulo Coordinator - Analisi Funzionale

1.  [Il ruolo del modulo Coordinator](#Ruolo_coordinator)
1.  [Il servizio Coordinator](#Servizio_coordinator)
    1.  [Richieste previste](#Richieste_previste)
        1.  [Numero di nodi nella rete](#Numero_nodi_nella_rete)
        1.  [Valuta un ingresso](#Valuta_ingresso)
        1.  [Avvio ingresso in altra rete](#Avvio_ingresso)
        1.  [Prenota un posto](#Prenota_un_posto)
        1.  [Cancella prenotazione](#Cancella_prenotazione)
        1.  [Replica memoria condivisa](#Replica)
        1.  [Confermato ingresso in altra rete](#Confermato_ingresso)
    1.  [Contenuto della memoria condivisa di un g-nodo](#Records)
1.  [Richieste al diretto vicino di accesso al servizio Coordinator](#Richieste_al_diretto_vicino)
    1.  [Ingresso in diversa rete](#Per_ingresso)
    1.  [Ingresso come risoluzione di uno split di g-nodo](#Per_split)
1.  [Collaborazioni con gli altri moduli](#Collaborazioni)
1.  [Requisiti](#Requisiti)
1.  [Deliverables](#Deliverables)
1.  [Classi e interfacce](#Classi_e_interfacce)

## <a name="Ruolo_coordinator"></a>Il ruolo del modulo Coordinator

Il modulo cerca di fare sì che un singolo g-nodo di livello *l*, sebbene costituito da un numero di singoli
nodi, abbia un comportamento coerente come singola entità.

Il modulo fa uso delle [tasklet](../Librerie/TaskletSystem.md), un sistema di multithreading cooperativo.

Il modulo fa uso del framework [ZCD](../Librerie/ZCD.md), precisamente appoggiandosi ad una libreria intermedia
prodotta con questo framework per formalizzare i metodi remoti usati nel demone *ntkd* per comunicare con i
diretti vicini.

Il modulo fa uso diretto delle classi e dei servizi forniti dal modulo [PeerServices](../ModuloPeers/AnalisiFunzionale.md).
In particolare, esso realizza un servizio peer-to-peer, chiamato appunto Coordinator, per mezzo del quale svolge
alcuni dei suoi compiti.

Siccome chiamiamo con lo stesso nome Coordinator sia il modulo presente su ogni nodo sia il servizio peer-to-peer
che rappresenta un g-nodo, bisogna che il lettore faccia attenzione al contesto. Di solito se parliamo
di Coordinator di un g-nodo (dal livello 1 fino all'intera rete) ci riferiamo al servizio, cioè
al nodo che al momento viene identificato come servente.

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

### <a name="Richieste_previste"></a>Richieste previste

Elenchiamo tutte le richieste che si possono fare al Coordinator.

#### <a name="Numero_nodi_nella_rete"></a>Numero di nodi nella rete

Una richiesta *r* di tipo NumberOfNodesRequest fatta al Coordinator dell'intera rete *G* indica che
il client del servizio (avendo incontrata una diversa rete di dimensioni simili a questa) vuole
chiedere alla rete (come entità atomica) quanti sono i suoi nodi.

Il motivo per cui viene fatta questa richiesta al Coordinator di *G* è perché i singoli nodi
della rete hanno una valutazione approssimativa. Supponiamo che ci siano le reti *G* e *J*.
Due distinte coppie di nodi *n<sub>0</sub>-v<sub>0</sub>* e *n<sub>1</sub>-v<sub>1</sub>* si
incontrano con *n<sub>i</sub>* ∈ *G* e *v<sub>i</sub>* ∈ *J*. Le dimensioni di *G* e *J* sono
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
sta per dare, con un relativo timeout di scadenza.

La risposta va restituita al client del servizio attraverso una istanza di NumberOfNodesResponse.

#### <a name="Valuta_ingresso"></a>Valuta un ingresso

Una richiesta *r* di tipo EvaluateEnterRequest fatta al Coordinator dell'intera rete indica che è stata incontrata una diversa rete *J*
sulla quale il singolo nodo *n* (il client del servizio) suggerisce di fare ingresso.

I membri di *r* contengono informazioni che non sono di pertinenza del modulo Coordinator.
Il contenuto della richiesta può essere semplicemente:

*   `Object evaluate_enter_data` = un oggetto serializzabile la cui classe è nota al delegato IEvaluateEnterHandler.

Questa richiesta viene fatta al servizio Coordinator, in particolare al Coordinator dell'intera rete, per
fare in modo che sia "tutta la rete" come entità atomica a venire interpellata.

Sebbene la richiesta venga fatta come detto al Coordinator della rete *G*,
in effetti la strategia di ingresso non è di pertinenza del modulo Coordinator. Per questo viene utilizzato
un delegato passato al modulo dal suo utilizzatore sotto forma di una istanza dell'interfaccia IEvaluateEnterHandler.

La risposta ottenuta dal delegato consiste in un intero oppure una eccezione:

*   `int lvl` - il livello del g-nodo di *n* che dovrebbe tentare l'ingresso in *J*.
*   `AskAgainError` - il client deve attendere alcuni istanti e poi riprovare.
*   `IgnoreNetworkError` - il client deve ignorare l'altra rete.

Il risultato va restituito così com'è al client del servizio attraverso una istanza di EvaluateEnterResponse.

#### <a name="Avvio_ingresso"></a>Avvio ingresso in altra rete

Una richiesta *r* di tipo BeginEnterRequest fatta al nodo Coordinator di *g* indica che il singolo nodo *n*
(il client del servizio) vuole essere autorizzato ad avviare le operazioni di ingresso del g-nodo *g* in
una diversa rete *J*.

I membri di *r* sono:

*   `lvl` = livello di *g*.
*   `Object begin_enter_data` = un oggetto serializzabile la cui classe è nota al delegato IBeginEnterHandler.  
    Contiene informazioni che non sono di pertinenza del modulo Coordinator.

Questa richiesta viene fatta al servizio Coordinator, in particolare al Coordinator di *g*, per
fare in modo che sia *g* come entità atomica a venire interpellata.

Per rispondere, non essendo questa materia di competenza del modulo Coordinator, viene utilizzato
un delegato passato al modulo dal suo utilizzatore sotto forma di una istanza dell'interfaccia IBeginEnterHandler.

Il delegato può completare il metodo correttamente (il metodo ha firma `void`) oppure lanciare una eccezione:

*   `AlreadyEnteringError` - il client deve ignorare l'altra rete.

Il risultato va restituito così com'è al client del servizio attraverso una istanza di BeginEnterResponse.

#### <a name="Prenota_un_posto"></a>Prenota un posto

Una richiesta *r* di tipo ReserveEnterRequest fatta al nodo Coordinator di *g* indica che il singolo nodo *n*
(il client del servizio) chiede la prenotazione di un posto in *g*.

*   `int lvl` = livello di *g*.
*   `int enter_id` = un identificativo di questa prenotazione.

Questa richiesta viene fatta al servizio Coordinator, in particolare al Coordinator di *g*, per
fare in modo che sia *g* come entità atomica a venire interpellata.

Per prima cosa il nodo Coordinator di *g* accede alla memoria condivisa di *g*. Nel membro `reserve_list` dell'istanza
di `CoordGnodeMemory` associata al livello `lvl` (come illustrato più sotto) vengono memorizzate
le prenotazioni pendenti. Se esiste già una prenotazione con l'identificativo `enter_id` allora
la stessa viene restituita al client. Altrimenti si prosegue.

Il nodo Coordinator di *g* accede alla propria mappa di percorsi, per vedere quali g-nodi
di livello `lvl` - 1 (oltre a quello a cui esso stesso appartiene) dentro al suo g-nodo di livello
`lvl` sono già presenti come destinazioni, quindi esistenti nella rete e non assegnabili alla
nuova richiesta.

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

Scelto il posto e l'anzianità, subito il Coordinator di *g* accede in scrittura alla memoria condivisa di *g*.
Questo come sappiamo comporta l'avvio di una tasklet che si occupi di replicare la scrittura nei nodi replica.

Il risultato della prenotazione è composto dalla nuova posizione e dalla sua anzianità. Esso viene
restituito al client del servizio attraverso una istanza di ReserveEnterResponse:

*   `int new_pos`
*   `int new_eldership`

Il nodo client del servizio è un nodo *n* che già appartiene al g-nodo *g*. Questi richiede la prenotazione di
un nuovo posto per conto di un altro nodo suo vicino, *m*, il quale non è ancora in *g* o perfino non è ancora
nella rete.  
Ricevuta la risposta dal nodo Coordinator, *n* la comunica al vicino *m*.

Altre informazioni di cui il nodo *m* necessita per fare ingresso in *g* (e eventualmente nella rete) sono
direttamente fornite dal nodo *n* che già le conosce. Queste sono:

*   La topologia della rete.
*   Le posizioni dei livelli maggiori di `lvl`.
*   Le anzianità dei livelli maggiori di `lvl`.

#### <a name="Cancella_prenotazione"></a>Cancella prenotazione

Una richiesta *r* di tipo DeleteReserveEnterRequest fatta al nodo Coordinator di *g* indica che il singolo nodo *n*
(il client del servizio) chiede la rimozione di una prenotazione pendente in *g*.

*   `int lvl` = livello di *g*.
*   `int enter_id` = l'identificativo della prenotazione pendente da rimuovere.

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

In realtà, come in tutte le repliche, il nodo client in questo caso è il nodo con indirizzo
attualmente più prossimo alla tupla del Coordinator di *g*, mentre il nodo servente è uno dei nodi
che potrebbero trovarsi in sua assenza a rispondere alle future richieste.

La gestione di questa richiesta avviene attraverso il codice del modulo PeerServices. Il modulo
Coordinator ha il compito di implementare i metodi dell'interfaccia IDatabaseDescriptor,
come dettagliato [qui](../ModuloPeers/DettagliTecnici.md#Mantenimento_database_distribuito),
in particolare i metodi `is_replica_value_request` e `execute`.

La risposta è una istanza di ReplicaResponse.

#### <a name="Confermato_ingresso"></a>Confermato ingresso in altra rete

La richiesta/segnalazione *r* che sono state completate le operazioni di ingresso del g-nodo *g* in una diversa rete. Tale
richiesta arriva ad un nodo *x* come Coordinator di *g*. **TODO**

### <a name="Records"></a>Contenuto della memoria condivisa di un g-nodo

Come abbiamo detto, il servizio Coordinator realizza una sorta di memoria condivisa di un g-nodo *g*. Per far questo
esso mantiene un database distribuito della tipologia a chiavi fisse in cui la chiave è il livello del g-nodo
in esame e i dati hanno visibilità locale circoscritta allo stesso g-nodo.

Questo significa che il contenuto di tutta la memoria condivisa di un g-nodo deve essere rappresentabile
interamente con l'istanza di un oggetto serializzabile. La classe definita nel modulo Coordinator usata per
memorizzare e trasmettere il contenuto della memoria condivisa è `CoordGnodeMemory`.

Si tratta di una classe serializzabile. Il significato di alcuni dei suoi membri è noto al modulo Coordinator.
Altri membri invece contengono strutture dati che il modulo Coordinator non deve conoscere. Questi membri sono di
tipo Object nullable e il modulo Coordinator sa solo che se sono valorizzati sono a loro volta oggetti serializzabili.

#### Contenuto di pertinenza del modulo Coordinator

Alcune informazioni che si devono poter memorizzare e leggere nella memoria condivisa di un g-nodo sono
di pertinenza del modulo Coordinator stesso.

Fra queste abbiamo:

*   `reserve_list` - Elenco delle prenotazioni pendenti.
*   `max_virtual_pos` - Massimo valore *virtuale* di `pos` assegnato ad un g-nodo al nostro interno.
*   `max_eldership` - Massimo valore di eldership assegnato ad un g-nodo al nostro interno. Maggiore è questo valore
    e più giovane è il g-nodo.
*   **TODO**

#### Contenuto di pertinenza di altri moduli

Alcune informazioni che si devono poter memorizzare e leggere nella memoria condivisa di un g-nodo sono
di pertinenza di altri moduli.

Verranno descritte ognuna con i suoi dettagli di seguito.

#### Modulo X

Le operazioni di ingresso in una rete, che sono messe in atto quando due reti distinte si incontrano
per mezzo di alcuni archi, non sono di pertinenza del modulo Coordinator, bensì del modulo X.

Però queste operazioni prevedono:

1.  L'esecuzione di alcuni metodi del modulo X nel nodo Coordinator (di tutta la rete o di un g-nodo) su richiesta che
    viene dal modulo X in un singolo nodo qualsiasi (ad esempio un border-nodo che ha incontrato un vicino di un'altra rete).
1.  La memorizzazione e rilettura di alcune informazioni nella memoria condivisa di tutta la rete (o di un g-nodo).

Per il primo motivo esistono metodi del modulo Coordinator, relativi metodi nella classe client del
servizio Coordinator, relativi classi di richieste e risposte (IPeersRequest e IPeersResponse) e delegati
che la classe servente del servizio Coordinator (PeerService) può richiamare.  
Di norma per ogni metodo del modulo X (di quelli che vanno eseguiti nel nodo Coordinator su richiesta di
un altro nodo) esiste una specifica istanza dei suddetti elementi.

Per il secondo motivo, esiste una classe serializzabile implementata nel modulo X tale che una sua
istanza contiene tutta la memoria condivisa di tutta la rete (o di un g-nodo) relativamente a quanto è di pertinenza
del modulo X. Tale istanza viene salvata nel membro `Object? network_entering_memory` della classe `CoordGnodeMemory`.

Vediamo come avviene la scrittura e la rilettura della memoria condivisa di tutta la rete (o di un g-nodo) ad opera
del modulo X. Nella trattazione del modulo X abbiamo detto che solo lo stesso nodo Coordinator (di tutta la rete o di un g-nodo)
può essere nella posizione di scrivere/leggere in questa memoria.  
Quando viene chiamato nel modulo Coordinator il metodo `set_network_entering_memory(Object data, int level)` questi avvia il
contatto con il servizio Coordinator per la chiave `k.lvl = level`. Quando viene contattato con tale richiesta,
il servente Coordinator verifica (pena la terminazione della tasklet che esegue la risposta) che il chiamante
era il nodo stesso.  
Poi il servente mette l'argomento ricevuto nel membro `network_entering_memory` dell'istanza di `CoordGnodeMemory`
associata alla chiave `k`.  
Poi in una nuova tasklet avvia le operazioni di replica. Quando viene contattato un nodo con la richiesta
di replica, esso verifica (pena la terminazione della tasklet che esegue la risposta) che il chiamante era più
prossimo di lui all'indirizzo del servente Coordinator.
**Nota questo è di pertinenza del codice messo a fattor comune nel modulo PeerService. Verificare.**  
Quando viene chiamato nel modulo Coordinator il metodo `get_network_entering_memory(int level)` questi avvia il
contatto con il servizio Coordinator per la chiave `k.lvl = level`. Quando viene contattato con tale richiesta,
il servente Coordinator verifica (pena la terminazione della tasklet che esegue la risposta) che il chiamante
era il nodo stesso.  
Poi il servente restituisce l'oggetto che è nel membro `network_entering_memory` dell'istanza di `CoordGnodeMemory`
associata alla chiave `k`.

## <a name="Richieste_al_diretto_vicino"></a>Richieste al diretto vicino di accesso al servizio Coordinator

In alcuni casi un nodo *n* può voler chiedere ad un suo diretto vicino di accedere al servizio peer-to-peer
del Coordinator.

### <a name="Per_ingresso"></a>Ingresso in diversa rete

Si consideri un nodo *n* che appartiene alla rete *G* e un suo diretto vicino nodo
*v* che appartiene alla rete *J*.  
Supponiamo che - con le modalità viste in "[ingresso in altra rete](#Collaborazioni_ingresso)" - il
nodo *n* riceva dal Coordinator della rete *G* il compito di richiedere l'ingresso del suo g-nodo *g*
di livello *l* dentro *J* tramite il suo arco con *v*.

Ora il nodo *n* comunica direttamente con il servizio Coordinator del livello *l*, cioè di *g*. Il
Coordinator di *g* accetta questa soluzione (vedi la richiesta [avvio-ingresso](#Avvio_ingresso)). Questa operazione consta di alcune parti:

*   Il Coordinator di *g* acquisisce un blocco. Cioè, se erano in coda altre operazioni che collidono
    con questa (ad esempio un'altra operazione di ingresso, *o altro da valutare*) allora attende prima
    di procedere.
*   Acquisito il blocco il Coordinator di *g* avvia una nuova tasklet per fare altre operazioni, ma
    intanto risponde positivamente al nodo *n*.
*   Nella tasklet avviata il Coordinator di *g* aspetta un certo tempo ...

Quando *n* riceve la risposta positiva dal Coordinator di *g*, subito chiede a *v* di trovare un posto per *g* in *J*.

Per rispondere a questa richiesta, il nodo *v* contatta il Coordinator del *suo* g-nodo di
livello *l* + 1, chiamiamolo *h*, chiedendogli di prenotare un posto (vedi la richiesta
[prenota-un-posto](#Prenota_un_posto)). Se ciò non fosse possibile
il Coordinator di *h* risponderebbe con una eccezione a *v* e ci sarebbero altre operazioni da fare
per la ricerca di una migration path; ma in questo esempio assumiamo che il
g-nodo *h* abbia un posto libero per un g-nodo di livello *l*. Allora il Coordinator di *h* prenota
il posto e lo comunica al nodo *v*. Il nodo *v* lo comunica al nodo *n*.

Quello che si voleva evidenziare qui, è che il modulo Coordinator in *n* comunica con il modulo
Coordinator del diretto vicino *v*. Prima di rispondere, il modulo Coordinator in *v* usa il client
del servizio Coordinator per comunicare, non con un diretto vicino, bensì tramite i servizi del
modulo PeerServices, con il Coordinator del suo g-nodo di livello *l* + 1, cioè *h*.

In questo modo il nodo *n*, dialogando con il diretto vicino *v*, è come se avesse dialogato con
l'intero g-nodo *h* come entità unitaria. Inoltre avendo prima il nodo *n* preso accordo con
il Coordinator di *g*, è come se i g-nodi *g* e *h* come entità unitarie avessero dialogato tra loro.

Ora che *n* ha ricevuto la prenotazione, di nuovo comunica con il Coordinator di *g* per informarlo
(vedi la richiesta/segnalazione [confermato-ingresso](#Confermato_ingresso)). Cioè
gli comunica che ogni singolo nodo in *g* dovrà presto attivarsi per formare una nuova identità che
prende posto in *h* (come g-nodo isomorfo di *g*).  
La propagazione delle informazioni necessarie ad ogni singolo nodo avverrà attraverso un diverso
modulo. (*Da valutare* Ogni singolo nodo potrebbe chiedere conferma al Coordinator di *g* dell'esattezza
delle suddette informazioni, ma questo comporterebbe un enorme traffico su *g*).  
Le informazioni suddette che pervengono ad ogni singolo nodo permettono al contempo la trasformazione
del g-nodo *g* in un g-nodo *di connettività* in *G*.

Ricevuta questa segnalazione, il Coordinator di *g*:

*   Il Coordinator di *g* controlla se la richiesta ha l'identificativo del blocco che era
    stato acquisito su *g* ed era in corso.
*   Memorizza le informazioni, rilascia il blocco ed avvia una nuova tasklet per fare altre operazioni, ma
    intanto risponde positivamente al nodo *n*.
*   Nella tasklet avviata il Coordinator di *g* aspetta un certo tempo ...

### <a name="Per_split"></a>Ingresso come risoluzione di uno split di g-nodo

Si consideri un nodo *n* che fa parte di un g-nodo *g* di livello *l*. Il g-nodo *g* fa a sua volta parte di
un g-nodo *h* di livello *l* + 1. Supponiamo che il g-nodo *g* diventi disconnesso, cioè avviene lo split del
g-nodo, mentre *h* è ancora connesso. Supponiamo che l'isola in cui si trova *n* (chiamiamola *g1*) sia una di quelle che non
contengono il nodo più anziano. Supponiamo che *n* abbia un vicino *v* che appartiene a *h* ma non appartiene
a *g*. Per questo il nodo *n* riceve da *v* l'informazione che tutta la sua isola (di livello *l*) deve
cambiare indirizzo.

In questo momento nessun nodo all'interno dell'isola *g1* ha un indirizzo valido in *h*. Quindi
non può comunicare con il server del servizio Coordinator del suo g-nodo di livello *l* + 1, o superiori.
Però si forma automaticamente un nuovo server del servizio Coordinator per *g1*.

Ci troviamo di nuovo in una situazione simile alla precedente. Il nodo *n* comunica direttamente
con il servizio Coordinator di *g1* proponendo un ingresso facilitato dal nodo diretto vicino *v*. Il
Coordinator di *g1* accetta questa soluzione.

A questo punto *n* chiede a *v* di trovare un posto per *g1* in *G*. Per questa richiesta il nodo *v* comunica
con il servizio Coordinator del *suo* g-nodo di livello *l* + 1, che è l'originale g-nodo *h*.

L'obiettivo finale di queste comunicazioni (possono essere necessarie più di una) con il server
del servizio Coordinator sarà ovviamente la prenotazione di un posto di livello *l* in *G*.

## <a name="Collaborazioni"></a>Collaborazioni con gli altri moduli

### <a name="Collaborazioni_ingresso"></a>Ingresso in altra rete

#### Metodo evaluate_enter

Quando il modulo X del nodo *n* vuole far eseguire il suo metodo `evaluate_enter` nel nodo Coordinator
della rete *G*, richiama il metodo `evaluate_enter` del modulo Coordinator.

L'esecuzione di `evaluate_enter` del modulo Coordinator consiste in questo:

Viene preparato un client del servizio Coordinator. Su questo viene chiamato il metodo `evaluate_enter`
passandogli la stessa struttura dati ricevuta dal metodo `evaluate_enter` del modulo Coordinator. Tale
struttura non è nota al modulo Coordinator, che sa solo che è un Object serializzabile.

La classe client del servizio sa che questo metodo usa come chiave *k* con `k.lvl = levels`. Cioè va contattato
il Coordinator dell'intera rete.

La classe client nel suo metodo `evaluate_enter` prepara una richiesta *r* = [EvaluateEnterRequest](#Valuta_ingresso)
che comprende la struttura dati (ovvero l'istanza di Object serializzabile) di cui sopra.

Poi invia la richiesta *r* e ottiene una risposta che è una istanza di EvaluateEnterResponse. Essa contiene
esattamente i possibili risultati previsti dalla signature del metodo `evaluate_enter`.

Quindi il metodo `evaluate_enter` del modulo Coordinator restituisce al chiamante:

*   `int ret`. Oppure:
*   Eccezione `AskAgainError`. Oppure:
*   Eccezione `IgnoreNetworkError`.

#### Metodo begin_enter

Quando il modulo X del nodo *n* vuole far eseguire il suo metodo `begin_enter(begin_enter_data)` nel nodo Coordinator
del suo g-nodo *g* di livello *lvl*, richiama il metodo `begin_enter(lvl, begin_enter_data)` del modulo Coordinator.

L'esecuzione di `begin_enter` del modulo Coordinator consiste in questo:

Viene preparato un client del servizio Coordinator. Su questo viene chiamato il metodo `begin_enter(lvl, begin_enter_data)`.
Ricordiamo che la struttura dati `begin_enter_data` non è nota al modulo Coordinator, che sa solo che è un Object serializzabile.

La classe client del servizio usa come chiave *k* con `k.lvl = lvl`. Prepara una richiesta *r* = [BeginEnterRequest](#Avvio_ingresso)
che comprende il livello *lvl* e la struttura dati (ovvero l'istanza di Object serializzabile) di cui sopra.

Poi invia la richiesta *r* e ottiene una risposta che è una istanza di BeginEnterResponse. Essa contiene
esattamente i possibili risultati previsti dalla signature del metodo `begin_enter`.

Quindi il metodo `begin_enter` del modulo Coordinator restituisce al chiamante:

*   `void`. Oppure:
*   Eccezione `AlreadyEnteringError`.

#### Metodo reserve_enter

Quando il modulo X del nodo *n* vuole far eseguire il suo metodo `reserve_enter(reserve_enter_data)` nel nodo Coordinator
del suo g-nodo *g* di livello *lvl*, richiama il metodo `reserve_enter(lvl, reserve_enter_data)` del modulo Coordinator.

L'esecuzione di `reserve_enter` del modulo Coordinator consiste in questo:

Viene preparato un client del servizio Coordinator. Su questo viene chiamato il metodo `reserve_enter(lvl, reserve_enter_data)`.
Ricordiamo che la struttura dati `reserve_enter_data` non è nota al modulo Coordinator, che sa solo che è un Object serializzabile.

La classe client del servizio usa come chiave *k* con `k.lvl = lvl`. Prepara una richiesta *r* = [ReserveEnterRequest](#Prenota_un_posto)
che comprende il livello *lvl* e la struttura dati (ovvero l'istanza di Object serializzabile) di cui sopra.

Poi invia la richiesta *r* e ottiene una risposta che è una istanza di ReserveEnterResponse. Essa contiene
esattamente i possibili risultati previsti dalla signature del metodo `reserve_enter`.

Quindi il metodo `reserve_enter` del modulo Coordinator restituisce al chiamante:

*   `Object ret`. Oppure:
*   Eccezione `FullNetworkError`.

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

Fornisce metodi per:

*   Valutare un ingresso in una nuova rete. Metodo `evaluate_enter`.  
    La logica per il rilevamento di un vicino appartenente ad una diversa rete e per l'ingresso
    in questa nuova rete è di pertinenza di un diverso modulo X. Vedi [qui](OperazioniIngresso.md).  
    Quello che fa questo metodo in realtà è permettere all'utilizzatore del modulo Coordinator di far eseguire una
    certa operazione sul nodo Coordinator della rete. In particolare, l'utilizzatore del modulo
    Coordinator nel nodo *n* passa un oggetto al metodo `evaluate_enter` (probabilmente su istruzione
    da parte di un altro modulo X); questi fa pervenire questo oggetto al nodo Coordinator della
    rete il quale lo passa al delegato `IEvaluateEnterHandler` (probabilmente implementato dallo
    stesso modulo X) nel suo metodo `evaluate_enter`.  
    L'esecuzione del metodo `IEvaluateEnterHandler.evaluate_enter` produce la decisione per il nodo *n* di tentare
    o meno l'ingresso nella nuova rete e se sì a quale livello.
*   Iniziare un ingresso in una nuova rete. Metodo `begin_enter`.  
    Anche in questo caso la logica di queste operazioni non è di pertinenza del modulo Coordinator.  
    Questo metodo permette all'utilizzatore del modulo Coordinator di far eseguire l'operazione `begin_enter`
    sul nodo Coordinator di un g-nodo *g* di livello *lvl*. Il delegato `IBeginEnterHandler.begin_enter`
    sul nodo Coordinator di *g* effettivamente autorizza/nega l'ingresso.
*   Prenotare un posto (se possibile *reale*, altrimenti *virtuale*) nel proprio g-nodo di livello
    *l*. Metodo `reserve`. **TODO**

Implementa il servizio Coordinator derivando la classe CoordinatorService dalla classe base PeerService.

Il modulo Coordinator si occupa di registrare con il PeersManager l'implementazione del servizio Coordinator
e di usare gli algoritmi forniti dal modulo PeerServices per il mantenimento del relativo database a chiavi
fisse. Cioè, esso crea una istanza della classe CoordinatorService.DatabaseDescriptor, che implementa
IFixedKeysDatabaseDescriptor. Con questa istanza come parametro, al bisogno, chiama i metodi
`fixed_keys_db_on_startup` e `fixed_keys_db_on_request` di PeersManager.

Deriva la classe CoordinatorClient dalla classe base PeerClient per avviare il contatto del Coordinator
di un suo g-nodo e richiederne i servizi.

## <a name="Classi_e_interfacce"></a>Classi e interfacce

La mappa delle posizioni libere/occupate ai vari livelli è un oggetto di cui il modulo conosce
l'interfaccia ICoordinatorMap. Tramite essa il modulo può:

*   Leggere un `int64` che rappresenta l'identificativo della rete (metodo `get_netid`).
*   Leggere il numero *l* dei livelli della topologia (metodo `get_levels`).
*   Leggere la gsize di ogni livello *i* da 0 a *l* - 1 (metodo `get_gsize`).  
    Precisiamo il significato di questo indice, restando coerenti con quanto stabilito nella
    trattazione del modulo QSPN sebbene i due moduli siano indipendenti.  
    Per ogni *i* da 0 a *l* - 1, *gsize(i)* è il numero massimo di g-nodi di livello *i* in un
    g-nodo di livello *i* + 1.
*   Leggere l'anzianità del mio g-nodo ad ogni livello *i* da 0 a *l* - 1 (metodo `get_eldership`).  
    L'anzianità di un g-nodo è un numero progressivo che viene assegnato al g-nodo. Nel confronto
    tra due g-nodi di pari livello *i* entrambi appartenenti allo stesso g-nodo di livello *i* + 1,
    un valore più alto significa che il g-nodo è arrivato dopo, cioè esso è più giovane.
*   Leggere il numero approssimativo di singoli nodi contenuti nel mio g-nodo ad ogni livello *i*
    da 0 a *l* - 1 (metodo `get_n_nodes`).  
*   Leggere la posizione del nodo, cioè la posizione del mio g-nodo ad ogni livello *i* da 0
    a *l* - 1 (metodo `get_my_pos`).
*   Leggere l'elenco delle posizioni libere in ogni livello *i* da 0 a *l* - 1 (metodo `get_free_pos`).  
    Cioè le posizioni nel livello *i* che sono libere nel nostro g-nodo di livello *i* + 1. Questa
    informazione è quella che si basa sulla mappa del nodo corrente, senza contattare il Coordinator
    attuale che ha la conoscenza autoritativa delle prenotazioni pendenti.

* * *

Una istanza di IEvaluateEnterHandler viene passata al modulo Coordinator. Tale istanza viene
interrogata da parte del modulo quando si deve decidere sul fare ingresso in una nuova rete.

I metodi previsti dall'interfaccia IEvaluateEnterHandler sono:

*   `int evaluate_enter(Object evaluate_enter_data)`  
    Con questo metodo si richiama il metodo `evaluate_enter` del modulo X. Vedi [qui](OperazioniIngresso.md).  
    Può rilanciare l'eccezione `AskAgainError`.  
    Può rilanciare l'eccezione `IgnoreNetworkError`.

* * *

Una istanza di IBeginEnterHandler viene passata al modulo Coordinator. Tale istanza viene
interrogata da parte del modulo quando si deve autorizzare l'ingresso in una nuova rete.

I metodi previsti dall'interfaccia IBeginEnterHandler sono:

*   `void begin_enter(int lvl, Object begin_enter_data)`  
    Con questo metodo si richiama il metodo `begin_enter` del modulo X. Vedi [qui](OperazioniIngresso.md).  
    Può rilanciare l'eccezione `AlreadyEnteringError`.

