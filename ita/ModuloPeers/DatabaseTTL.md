# Modulo PeerServices - Database con un Time To Live dei record

1.  [Effetto di visibilità locale del dato](#Visibilita_locale)
1.  [Operazione di inserimento](#Operazione_inserimento)
1.  [Operazione di sola lettura](#Operazione_sola_lettura)
1.  [Operazione di modifica](#Operazione_modifica)
1.  [Operazione di replica](#Operazione_replica)
1.  [Esaustività del nodo servente](#Esaustivita_nodo_servente)
1.  [Fase iniziale](#Fase_iniziale)
    1.  [Recupero preventivo delle chiavi](#Recupero_preventivo)
1.  [Algoritmi](#Algoritmi)

Implementiamo un servizio che mantiene un database distribuito. Ogni record in esso ha un Time To Live.
Le chiavi sono in un set che può essere molto grande.

Ogni nodo partecipante può avere un limite di memoria, oltre il quale il nodo rifiuterà di immagazzinare
altri record.

In questo servizio ipotiziamo che non ci siano vincoli per il client, cioè qualsiasi operazione è permessa
(nei limiti del dominio delle chiavi e dei valori) senza alcun tipo di autorizzazione. Comunque risulterà
facile implementare in un servizio analogo qualsiasi tipo di vincolo da parte del server; basterà
codificare nella classe del valore di ritorno le eccezioni previste dal servizio.

Diamo per assunto che il lettore abbia già letto il documento [Dettagli Tecnici](DettagliTecnici.md).

## <a name="Visibilita_locale"></a>Effetto di visibilità locale del dato

Opzionalmente, il servizio può stabilire di dare ad alcune chiavi un effetto di visibilità locale del dato.
Con questo intendiamo che una chiave *k* può avere codificato al suo interno un livello *l*. Se un nodo *n*
fa richiesta di memorizzare il valore *v* per la chiave *k*, tale valore sarà visibile ai soli nodi
appartenenti allo stesso g-nodo di livello *l* a cui appartiene il nodo *n*. In altri g-nodi il valore
associato alla chiave *k* può non esistere o essere diverso.

Per ottenere questo sarà sufficiente che la classe del servizio implementi il metodo `evaluate_hash_node`
della sua istanza di IDatabaseDescriptor in modo da restituire, quando la chiave passata a parametro ha
codificato il livello *l*, una tupla di soli *l* elementi. La stessa implementazione ovviamente dovrà
essere usata dalla classe del client del servizio nel suo metodo `perfect_tuple`.

## <a name="Operazione_inserimento"></a>Operazione di inserimento

Il nodo *q* vuole inserire un record *k=w* nel database distribuito.

Ci sono 4 possibili esiti finali a questa richiesta:

1.  Il nodo *q* viene informato che al momento nella rete non c'è nessun partecipante al servizio.
    Chiamiamolo esito "NO-PARTICIPANTS".
1.  Il nodo *q* viene informato che la chiave *k* era libera e il record *k=w* è stato inserito. Chiamiamolo
    esito "OK".
1.  Il nodo *q* viene informato che la chiave *k* è già valorizzata con il valore *v*. Chiamiamolo
    esito "NOT-FREE".
1.  Il nodo *q* viene informato che la chiave *k* è libera ma la memoria nel database distribuito è esaurita.
    Chiamiamolo esito "OUT-OF-MEMORY".

## <a name="Operazione_sola_lettura"></a>Operazione di sola lettura

Il nodo *q* vuole leggere il valore del record per una chiave *k* nel database distribuito.

Ci sono 3 possibili esiti finali a questa richiesta:

1.  Il nodo *q* viene informato che al momento nella rete non c'è nessun partecipante al servizio. Chiamiamolo
    esito "NO-PARTICIPANTS".
1.  Il nodo *q* viene informato che il record per la chiave *k* è stato trovato e il suo valore è *v*.
    Chiamiamolo esito "OK".
1.  Il nodo *q* viene informato che la chiave *k* non è presente nel database. Chiamiamolo esito "NOT-FOUND".

Il fatto che l'operazione di sola lettura sia distinta dalle operazioni che prevedono un aggiornamento è importante
in questa tipologia di database con scadenza temporale. In particolare questa caratteristica è di aiuto ad un
generico nodo *n’* che, contattato durante il cammino di ricerca dell'​*hash-node* per la chiave *k* e ricevuta
la richiesta, si rifiuta di elaborarla in quanto *non esaustivo*: il nodo *n’*, se la richiesta che riceve può
essere identificata come di "sola lettura", non necessiterà di resettare il tempo per cui si deve considerare
*non esaustivo* per la chiave *k*.

## <a name="Operazione_modifica"></a>Operazione di modifica

Il nodo *q* vuole modificare il valore o rinfrescare il TTL o rimuovere il record per una chiave *k* nel
database distribuito.

Ci sono 3 possibili esiti finali a questa richiesta:

1.  Il nodo *q* viene informato che al momento nella rete non c'è nessun partecipante al servizio. Chiamiamolo
    esito "NO-PARTICIPANTS".
1.  Il nodo *q* viene informato che il record per la chiave *k* è stato modificato/rinfrescato/rimosso.
    Chiamiamolo esito "OK".
1.  Il nodo *q* viene informato che la chiave *k* non è presente nel database. Chiamiamolo esito "NOT-FOUND".

## <a name="Operazione_replica"></a>Operazione di replica

Il nodo *n0* vuole replicare un record *k=w* che esso ha appena scritto nella sua memoria, oppure vuole
replicare la cancellazione di un record con chiave *k*. La richiesta giunge ad un nodo che viene dopo
*n0* nella ricerca dell'hash-node.

Ci sono 2 possibili esiti finali a questa richiesta:

1.  Il nodo *n0* viene informato che al momento nella rete non c'è nessun partecipante al servizio.  
    Oppure il nodo *n0* viene informato che la memoria nel database distribuito è esaurita.  
    Ai fini di questa operazione i due casi suddetti si equivalgono. Chiamiamo questo esito "OUT-OF-MEMORY".
1.  Il nodo *n0* viene informato che il record *k=w* (o la cancellazione del record per *k*) è stato replicato.
    Chiamiamolo esito "OK".

## <a name="Esaustivita_nodo_servente"></a>Esaustività del nodo servente

Se un nodo *n* riceve una richiesta per la chiave *k* e ha nella sua memoria un record per tale chiave,
allora si considera sempre *esaustivo*. Se invece non ha il record per tale chiave nella sua memoria, deve
chiedersi se è esaustivo o no.

Per gestire questo aspetto il nodo *n*  fa uso di alcune strutture dati memorizzate nella classe DatabaseHandler:

*   Elenco di chiavi `HashMap<Object,Timer> not_exhaustive_keys` e per ogni chiave dell'elenco un relativo timer.  
    Se il nodo riceve una richiesta relativa alla chiave *k* che non è nella sua memoria e la chiave *k* è
    nell'elenco `not_exhaustive_keys` e il suo timer non è scaduto, allora il nodo si ritiene *non esaustivo* per
    tale chiave. Se il relativo timer è scaduto tale chiave viene rimossa dall'elenco.
*   Elenco di chiavi `List<Object> not_found_keys`.  
    Se il nodo riceve una richiesta relativa alla chiave *k* che non è nella sua memoria e la chiave *k* è
    nell'elenco `not_found_keys`, allora il nodo si ritiene *esaustivo* per tale chiave.  
    In nessun momento una stessa chiave *k* può appartenere ad entrambi gli elenchi suddetti. Inoltre, se una
    chiave *k* è nella memoria del nodo non può appartenere a nessuno degli elenchi suddetti.
*   Stato di default e relativo timer.  
    Si tratta di un flag che dice al nodo come considerarsi per una chiave *k* che non è nella sua memoria e
    nemmeno nell'elenco `not_exhaustive_keys` e nemmeno nell'elenco `not_found_keys`.  
    Il timer si riferisce al tempo di validità dello stato di default *non esaustivo*. Cioè: quando per qualche
    determinato evento il nodo mette il suo stato di default a *non esaustivo*, allora imposta anche il timer al
    TTL dei record di questo servizio; quando il timer scade, automaticamente lo stato di default del nodo torna
    ad essere *esaustivo*.  
    Si usa una sola variabile: `Timer timer_default_not_exhaustive`.

Il nodo *n* gestisce l'elenco `not_exhaustive_keys` in modo da memorizzare quando e per quanto tempo deve
ritenersi *non esaustivo* per una data chiave *k*. Il nodo *n* è in grado di farlo perché le richieste per
tale chiave giungono al nodo stesso e possono essere esaminate (ad esempio per vedere se sono di sola lettura
o possono produrre un aggiornamento) prima che il nodo risponda rifiutando l'elaborazione. Se nessuna richiesta
giunge per la chiave *k* entro il TTL dei record di questo servizio, allora il record se era presente nel
database adesso non lo è più; per questo il singolo elemento della lista `not_exhaustive_keys` può essere
rimosso dalla lista quando scade il suo timer.

Questo elenco non può crescere senza limite; però non si può permettere che il nodo si consideri *esaustivo*
quando non lo è. Per questo, se l'elenco `not_exhaustive_keys` diventa troppo grande il nodo *n* mette il
suo stato di default a *non esaustivo* per il tempo TTL dei record e può svuotare del tutto l'elenco
`not_exhaustive_keys`. In seguito tornerà ad inserire singole chiavi nell'elenco `not_exhaustive_keys` mentre
il timer dello stato di default andrà scorrendo.

Il nodo *n* gestisce anche l'elenco `not_found_keys` in modo da memorizzare quando deve ritenersi *esaustivo*
per una data chiave *k*. Inserisce una chiave in questo elenco quando elabora una richiesta di rimozione per
la chiave *k* oppure quando la procedura di recupero del record per la chiave *k* si conclude con un "NOT-FOUND".
Questo elenco è una sorta di cache: se una chiave si trova in esso allora il nodo può rispondere senza
costringere il client a ulteriori trasmissioni e attese, ma se non vi si trova e il nodo è *non esaustivo*
per *k* allora rifiuterà l'elaborazione e il client giungerà comunque alla risposta corretta. Non si vuole
far crescere la lista senza limite, ma allo stesso tempo si vuole tenere chiavi utili, quindi si mettono in
testa alla lista gli elementi appena aggiunti e si rimuovono gli elementi in coda quando serve.

La gestione dell'elenco `not_found_keys` è indipendente dallo stato di default del nodo. Cioè, sebbene l'elenco
abbia un effetto concreto solo quando lo stato di default è *non esaustivo*, comunque l'elenco non viene
svuotato quando il nodo passa allo stato di default *esaustivo*.

## <a name="Fase_iniziale"></a>Fase iniziale

Quando un nodo partecipante al servizio costituisce una nuova rete allora esso è da subito *esaustivo* poiché
inizializza un database vuoto. Di seguito analiziamo cosa fa un nodo che, invece, entra in una rete esistente.

Ricordiamo che con questo *ingresso* possiamo intendere anche una *migrazione*. E che in entrambi i casi il
nodo *n* entra in blocco insieme ad un g-nodo *w* di cui conosce il livello. Tale livello è quello che va a
valorizzare il `maps_retrieved_below_level` nel PeersManager.

**TODO** Da approfondire dopo aver completato il [quadro d'insieme](DettagliTecnici.md#Overview).

Inizialmente, quando entra in una rete, il nodo *n* si mette nello stato di default *non esaustivo* per un tempo
pari al TTL dei record che il servizio *p* memorizza.

Se un record collegato alla chiave *k* in *p* deve rimanere in vita, allora un nodo *q* farà una richiesta
(di inserimento, lettura o aggiornamento) per la chiave *k* e questa giungerà al nodo *n*.

Il nodo *n* risponderà rifiutando di elaborare la richiesta in quanto *non esaustivo*; ma al contempo verrà
a conoscenza di una chiave *k*.

Il nodo *q* proseguirà con la sua ricerca e passerà la richiesta ai successivi nodi più prossimi al hash-node
fino a trovare un nodo in grado di rispondere. Questo nodo, chiamiamolo *current(k)*, la elaborerà e risponderà.
Potrà trattarsi di una qualsiasi operazione: inserimento, lettura o aggiornamento.

In parallelo il nodo *n* contatta *current(k)* (per farlo sarà sufficiente avviare `contact_peer` per la
chiave *k* escludendo se stesso) e avvia il [procedimento di recupero](DettagliTecnici.md#Procedimento_di_recupero_di_un_record)
del record associato alla chiave *k* con attesa del tempo critico di coerenza *𝛿*.

Come detto, tale procedimento prevede che se il nodo *n* durante questo tempo riceve richieste per la chiave *k*
che sono di scrittura (aggiornamento o inserimento), allora non le rifiuta subito, ma le mette in attesa. Siccome
ora le richieste di scrittura per la chiave *k* passano prima per il nodo *n* che le fa attendere, di sicuro non
giungeranno al *current(k)*.

Al termine di questa attesa *𝛿*, il nodo *n* riceve il valore corrente del record per la chiave *k* e lo mette
nella sua memoria; oppure riceve "NOT-FOUND" e diventa *esaustivo* per la chiave *k*. Ma, essendo passato del
tempo, non è detto che il suo indirizzo sia ancora il più prossimo all'​*hash-node* per la chiave *k*. Quindi
risponderà a tutte le richieste che aveva eventualmente messo in attesa istruendo il client a ricominciare da
capo il calcolo distribuito di H<sub>t</sub> ("REDO-FROM-START").

In seguito il nodo *n* saprà rispondere in modo autonomo alle richieste di tutti i tipi per la chiave *k*.

### <a name="Recupero_preventivo"></a>Recupero preventivo delle chiavi

Un passaggio ulteriore, sempre riferito ad un nodo che entra in una rete esistente e quindi si considera
inizialmente *non esaustivo*, può essere questo: il nodo *n* richiama `contact_peer` prendendo come obiettivo
`perfect_tuple` il suo stesso indirizzo ed escludendo se stesso; contatta un nodo (chiamiamolo *m*) e gli chiede
tutte le chiavi (non i record) che conosce.

Il modulo usa la classe RequestSendKeys e la classe RequestSendKeysResponse.

La classe RequestSendKeys è una classe serializzabile, che deriva da Object, e contiene il numero massimo di
chiavi da restituire `max_count`. Implementa l'interfaccia (vuota) IPeersRequest. È la richiesta di inviare
tutte le chiavi memorizzate, fino ad un massimo indicato dal client. Tale classe non viene esposta dal modulo.

La classe RequestSendKeysResponse è una classe serializzabile, che deriva da Object, e contiene una lista *lst*
di istanze di Object serializzabili. Implementa l'interfaccia (vuota) IPeersResponse. È la risposta alla richiesta
di inviare tutte le chiavi memorizzate. Tale classe non viene esposta dal modulo.

Se riceve una eccezione PeersNoParticipantsInNetworkError su questa prima richiesta, il nodo *n* deduce che
nessun nodo partecipava (probabilmente il servizio è opzionale) e quindi esso è da subito *esaustivo* poiché
inizializza un database vuoto. Vediamo cosa fa il nodo se, invece, riceve una risposta.

Per ogni chiave *k* che ha ottenuto, il nodo *n* calcola (con il metodo `evaluate_hash_node` dell'istanza di
IDatabaseDescriptor) la tupla dell'hash-node relativo alla chiave *k*. In questo modo il nodo *n* scopre anche
se la ricerca andrà circoscritta ad un certo suo g-nodo di livello *l*; infatti in questo caso otterrà una
tupla di *l* elementi. Se il nodo *m* dal quale siamo venuti a conoscenza della chiave *k* non rientra in tale
g-nodo allora questa chiave va ignorata.

Proseguendo, il nodo *n* valuta se la chiave *k* è tale che avrebbe dovuto essere chiesta a *n*: cioè se
*dist(h<sub>p</sub>(k), n)﹤dist(h<sub>p</sub>(k), m).* In questo caso esegue le operazioni viste prima.
In questo modo previene il momento in cui scopre una chiave *k* perché gli viene richiesta da qualcuno.

Quando il recupero del record per ogni chiave *k* ottenuta da *m* è stato portato a termine, il nodo *n* può
ripetere l'operazione iniziale (quella di chiamare `contact_peer` con il suo stesso indirizzo come obiettivo)
quante volte vuole, escludendo oltre a se stesso i vari nodi *m* che ha precedentemente contattato.

Se riceve una eccezione PeersNoParticipantsInNetworkError su una richiesta successiva alla prima, il nodo
termina queste operazioni e si considera *esaustivo* da subito.

Queste operazioni come detto possono proseguire fin quando si vuole. Però sono operazioni che appesantiscono
la rete e non sono essenziali. Quindi conviene intervallarle (ad esempio una pausa di 2 secondi dopo ogni
recupero di chiavi o di singoli record) e comunque non proseguire oltre un tempo che è una piccola porzione
(ad esempio 1/10) del TTL di ogni record.

Quando il nodo *n* decide di interrompere queste operazioni (pur non avendo ricevuto una eccezione
PeersNoParticipantsInNetworkError) esso deve comunque considerarsi *non esaustivo* per il tempo detto sopra.

## <a name="Algoritmi"></a>Algoritmi

La classe del servizio deve fornire le operazioni previste in tutti i servizi che implementano un
[database distribuito](DettagliTecnici.md#Mantenimento_di_un_database_distribuito), e inoltre altre
operazioni.

Tra le operazioni comuni, in particolare, l'operazione di verifica della presenza di un record (il metodo
`my_records_contains` dell'interfaccia IDatabaseDescriptor) deve tenere conto dei TTL dei record che ha in
memoria. Deve cioè prima rimuovere tutti i record scaduti e solo dopo verificare la presenza del record
richiesto. Invece l'operazione di recupero di un record (il metodo `get_records_for_key` dell'interfaccia
IDatabaseDescriptor) avendo già come requisito che il chiamante abbia verificato la sua presenza, non dovrà
controllare di nuovo il TTL del record che va a reperire.

Le ulteriori operazioni necessarie sono:

*   `int ttl_db_max_records_getter()`: Restituire il numero massimo di record che il nodo è disposto a
    memorizzare nella sua memoria.
*   `int ttl_db_my_records_size()`: Restituire il numero di record attualmente memorizzati nel nodo. I
    record sono memorizzati dalla classe del servizio.
*   `int ttl_db_max_keys_getter()`: Restituire il numero massimo di chiavi (istanze di chiave senza relativa
    istanza di record) che il nodo permette di tenere in memoria. Queste chiavi non sono memorizzate dalla
    classe del servizio, ma direttamente dal modulo PeerServices negli elenchi `not_exhaustive_keys` e `not_found_keys`.
*   `int ttl_db_msec_ttl_getter()`: Restituire il TTL in millisecondi che viene assegnato ai record appena
    memorizzati.
*   `List<Object> ttl_db_get_all_keys()`: Restituire l'elenco delle chiavi dei record attualmente
    memorizzati nel nodo.
*   `int ttl_db_timeout_exec_send_keys_getter()`: Restituire il tempo di attesa massimo che il client
    aspetta per l'esecuzione della richiesta di inviare tutti i record memorizzati dal nodo. Va scelto piuttosto
    alto in via cautelativa, perché il client non sa quale sia la quantità di memoria riservata a questo
    servizio dal server che sarà contattato.

Il modulo fornisce l'interfaccia ITemporalDatabaseDescriptor. Essa estende l'interfaccia IDatabaseDescriptor
ed espone anche i metodi sopra descritti: `ttl_db_max_records_getter`, `ttl_db_my_records_size`,
`ttl_db_max_keys_getter`, `ttl_db_msec_ttl_getter`, `ttl_db_get_all_keys` e `ttl_db_timeout_exec_send_keys_getter`.

La classe che implementa il servizio nel suo costruttore crea una istanza di ITemporalDatabaseDescriptor che
userà in tutte le chiamate a questi algoritmi, che sono metodi di PeersManager. Subito richiamerà il metodo
`ttl_db_on_startup`. In seguito per ogni richiesta che riceve richiamerà il metodo `ttl_db_on_request`.

Algoritmo di valutazione *esaustivo* per *k*:

**internal bool ttl_db_is_exhaustive(ITemporalDatabaseDescriptor tdd, Object k)**

*   Se `k` è in `tdd.dh.not_exhaustive_keys`:
    *   assert: `k` non è in `tdd.dh.not_found_keys`.
    *   Se `tdd.dh.not_exhaustive_keys[k].timer` è scaduto:
        *   Rimuovi `k` da `tdd.dh.not_exhaustive_keys`.
    *   Altrimenti:
        *   Return False.
*   Se `k` è in `tdd.dh.not_found_keys`:
    *   assert: `k` non è in `tdd.dh.not_exhaustive_keys`.
    *   Return True.
*   Se `tdd.dh.timer_default_not_exhaustive` è scaduto:
    *   Return True.
*   Altrimenti:
    *   Return False.

Algoritmo di valutazione *out of memory*:

**internal bool ttl_db_is_out_of_memory(ITemporalDatabaseDescriptor tdd)**

*   Se `tdd.ttl_db_my_records_size()` + `tdd.dh.retrieving_keys.size` ≥ `tdd.ttl_db_max_records`:
    *   Return True.
*   Altrimenti:
    *   Return False.

Algoritmo di aggiunta *k* a `not_exhaustive_keys`:

**internal void ttl_db_add_not_exhaustive(ITemporalDatabaseDescriptor tdd, Object k)**

*   `max_not_exhaustive_keys`= `tdd.ttl_db_max_keys` / 2.
*   assert: `k` non è in `tdd.dh.not_found_keys`.
*   Se `k` è in `tdd.dh.not_exhaustive_keys`:
    *   Reimposta `tdd.dh.not_exhaustive_keys[k]` = `new Timer(tdd.ttl)`.
    *   Return.
*   Se `tdd.dh.not_exhaustive_keys.size` < `max_not_exhaustive_keys`:
    *   Aggiungi `tdd.dh.not_exhaustive_keys[k]` = `new Timer(tdd.ttl)`.
    *   Return.
*   Imposta `tdd.dh.timer_default_not_exhaustive` = `new Timer(tdd.ttl)`.
*   Esegui `tdd.dh.not_exhaustive_keys.clear()`.
*   Return.

Algoritmo di aggiunta *k* a `not_found_keys`:

**internal void ttl_db_add_not_found(ITemporalDatabaseDescriptor tdd, Object k)**

*   `max_not_found_keys`= `tdd.ttl_db_max_keys` / 2.
*   assert: `k` non è in `tdd.dh.not_exhaustive_keys`.
*   Se `k` è in `tdd.dh.not_found_keys`:
    *   Esegui `tdd.dh.not_found_keys.remove(k)`.
*   Se `tdd.dh.not_found_keys.size` ≥ `max_not_found_keys`:
    *   Esegui `tdd.dh.not_found_keys.remove_at(0)`.
*   Esegui `tdd.dh.not_found_keys.add(k)`.

Algoritmo di rimozione di *k* da `not_exhaustive_keys`:

**internal void ttl_db_remove_not_exhaustive(ITemporalDatabaseDescriptor tdd, Object k)**

*   Se `k` è in `tdd.dh.not_exhaustive_keys`:
    *   Esegui `tdd.dh.not_exhaustive_keys.unset(k)`.

Algoritmo di rimozione di *k* da `not_found_keys`:

**internal void ttl_db_remove_not_found(ITemporalDatabaseDescriptor tdd, Object k)**

*   Se `k` è in `tdd.dh.not_found_keys`:
    *   Esegui `tdd.dh.not_found_keys.remove(k)`.

Algoritmo all'avvio:

**void ttl_db_on_startup(ITemporalDatabaseDescriptor tdd, int p_id, bool new_network)**

*   assert: `services.has_key(p_id)`.
*   In una nuova tasklet:
    *   `srv` = `services[p_id]`.
    *   `tdd.dh` = `new DatabaseHandler()`.
    *   `tdd.dh.p_id` = `p_id`.
    *   `tdd.dh.ready` = `False`.
    *   Se `srv.is_optional`:
        *   Mentre **not** `participant_maps_retrieved`:
            *   Se `participant_maps_failed`:
                *   Termina l'algoritmo.
            *   Aspetta 1 secondo.
    *   `tdd.dh.timer_default_non_exhaustive` = un nuovo timer che scade dopo `tdd.ttl_db_msec_ttl` millisecondi.
    *   `tdd.dh.not_found_keys` = `new ArrayList<Object>(tdd.key_equal_data)`.
    *   `tdd.dh.not_exhaustive_keys` = `new HashMap<Object,Timer>(tdd.key_hash_data, tdd.key_equal_data)`.
    *   `tdd.dh.retrieving_keys` = `new HashMap<Object,INtkdChannel>(tdd.key_hash_data, tdd.key_equal_data)`.
    *   `tdd.dh.ready` = `True`.
    *   Se `new_network`:
        *   `tdd.dh.timer_default_non_exhaustive` = un nuovo timer che scade dopo **0** millisecondi.
        *   Return. L'algoritmo termina.
    *   `IPeersRequest r` = `new RequestSendKeys()`.
    *   `PeerTupleNode tuple_n`.
    *   `PeerTupleNode respondant`.
    *   `IPeersResponse ret`.
    *   Try:
        *   `tuple_n` = `make_tuple_node(new HCoord(0, pos[0]), levels)`.
        *   `respondant` = null. Sarà una tupla nel g-nodo di livello *levels*, poiché `tuple_n` ha *levels* elementi.
        *   Esegue `ret = contact_peer(p_id, tuple_n, r, tdd.ttl_db_timeout_exec_send_keys, True, out respondant)`.
    *   Se riceve PeersNoParticipantsInNetworkError:
        *   `tdd.dh.timer_default_non_exhaustive` = un nuovo timer che scade dopo **0** millisecondi.
        *   Return. L'algoritmo termina.
    *   Se riceve PeersDatabaseError:
        *   `tdd.dh.timer_default_non_exhaustive` = un nuovo timer che scade dopo **0** millisecondi.
        *   Return. L'algoritmo termina.
    *   `timer_startup` = un nuovo timer che scade dopo `tdd.ttl_db_msec_ttl / 10` millisecondi.
    *   Try:
        *   Il valore restituito `ret` dovrebbe essere un RequestSendKeysResponse, cioè una lista di Object.
            Altrimenti la risposta viene ignorata.
        *   Se `ret` è una istanza di RequestSendKeysResponse:
            *   Per ogni chiave `k` in `ret`:
                *   Se **not** `ttl_db_is_out_of_memory(tdd)`:
                    *   Se `tdd.is_valid_key(k)`:
                        *   Se **not** `tdd.my_records_contains(k)` **e** **not** `ttl_db_is_exhaustive(tdd, k)`
                            **e** **not** `tdd.dh.retrieving_keys.has_key(k)`:
                            *   Non sa nulla di `k`.
                            *   `h_p_k` = `tdd.evaluate_hash_node(k)`.
                            *   `l` = `h_p_k.size`.
                            *   `int case`.
                            *   `HCoord gn`.
                            *   Calcola `convert_tuple_gnode(tuple_node_to_tuple_gnode(respondant), out case, out gn)`.
                            *   Se `gn.lvl` ≤ `l`:
                                *   `tuple_n_inside_l` = `rebase_tuple_node(tuple_n, l)`.
                                *   `respondant_inside_l` = `rebase_tuple_node(respondant, l)`.
                                *   Se `dist(h_p_k, tuple_n_inside_l)` < `dist(h_p_k, respondant_inside_l)`:
                                    *   Esegue `ttl_db_start_retrieve(tdd, k)`. Cioè recupera il record per la chiave `k`.
                                    *   Attendi 2 secondi.
                                    *   Se `timer_startup.is_expired()`: return.
        *   Prepara `exclude_tuple_list` = \[]  una lista di istanze di tuple globali nel g-nodo di ricerca di livello *levels*.
        *   Metti in `exclude_tuple_list` il nodo `respondant`, espresso come PeerTupleGNode di livello 0 nel g-nodo di livello *levels*.
        *   While **not** `ttl_db_is_out_of_memory(tdd)`:
            *   La memoria destinata da *n* al servizio *p* non è esaurita.
            *   Attendi 2 secondi.
            *   Se `timer_startup.is_expired()`: return.
            *   `respondant` = null. Sarà una tupla nel g-nodo di livello *levels*, poiché `tuple_n` ha *levels*.
            *   Esegue `ret = contact_peer(p_id, tuple_n, r, tdd.ttl_db_timeout_exec_send_keys, True, out respondant, exclude_tuple_list)`.  
                Il valore restituito dovrebbe essere un RequestSendKeysResponse, cioè una lista di Object.
                Altrimenti la risposta viene ignorata.
            *   Se `ret` è una istanza di RequestSendKeysResponse:
                *   Per ogni chiave `k` in `ret`:
                    *   Se `tdd.is_valid_key(k)`:
                        *   Se **not** `tdd.my_records_contains(k)` **e** **not** `ttl_db_is_exhaustive(tdd, k)`
                            **e** **not** `tdd.dh.retrieving_keys.has_key(k)`:
                            *   Non sa nulla di `k`.
                            *   `h_p_k` = `tdd.evaluate_hash_node(k)`.
                            *   `l` = `h_p_k.size`.
                            *   `int case`.
                            *   `HCoord gn`.
                            *   Calcola `convert_tuple_gnode(tuple_node_to_tuple_gnode(respondant), out case, out gn)`.
                            *   Se `gn.lvl` ≤ `l`:
                                *   `tuple_n_inside_l` = `rebase_tuple_node(tuple_n, l)`.
                                *   `respondant_inside_l` = `rebase_tuple_node(respondant, l)`.
                                *   Se `dist(h_p_k, tuple_n_inside_l)` < `dist(h_p_k, respondant_inside_l)`:
                                    *   Esegue `ttl_db_start_retrieve(tdd, k)`. Cioè recupera il record per la chiave `k`.
                                    *   Se `ttl_db_is_out_of_memory(tdd)`: Esci dal ciclo **for**.
                                    *   Attendi 2 secondi.
                                    *   Se `timer_startup.is_expired()`: return.
                *   Se `ttl_db_is_out_of_memory(tdd)`: Esci dal ciclo **while**.
            *   Metti in `exclude_tuple_list` il nodo `respondant`, espresso come PeerTupleGNode di livello 0 nel
                g-nodo di livello *levels*.
    *   Se riceve PeersNoParticipantsInNetworkError:
        *   `tdd.dh.timer_default_non_exhaustive` = un nuovo timer che scade dopo **0** millisecondi.
        *   Return. L'algoritmo termina.
    *   Se riceve PeersDatabaseError:
        *   `tdd.dh.timer_default_non_exhaustive` = un nuovo timer che scade dopo **0** millisecondi.
        *   Return. L'algoritmo termina.

Algoritmo alla ricezione della richiesta:

**IPeersResponse ttl_db_on_request(ITemporalDatabaseDescriptor tdd, IPeersRequest r, int common_lvl) throws PeersRefuseExecutionError, PeersRedoFromStartError**

*   Gli argomenti del metodo `ttl_db_on_request` sono:
    *   `tdd`: istanza di una classe che implementa ITemporalDatabaseDescriptor sopra descritta.
    *   `r`: la richiesta.
    *   `common_lvl`: il livello del minimo comune g-nodo con il richiedente, da 0 a *levels* compresi.
*   Se `tdd.dh` = `null` **o not** `tdd.dh.ready`:
    *   Rilancia `PeersRefuseExecutionError.NOT_EXHAUSTIVE`.
    *   L'algoritmo termina.
*   Se `r` is `RequestSendKeys`:
    *   `ret` = `new RequestSendKeysResponse()`.
    *   Per ogni chiave `Object k` in `tdd.ttl_db_get_all_keys()`:
        *   Aggiungi `k` a `ret.lst`.
        *   Se `ret.lst.size` ≥ `r.max_count`:
            *   Esci dal ciclo.
    *   Return `ret`.
    *   L'algoritmo termina.
*   Se `tdd.is_insert_request(r)`:
    *   `Object k` = `tdd.get_key_from_request(r)`.
    *   Se `tdd.my_records_contains(k)`:
        *   assert: `k` non è in `tdd.dh.not_exhaustive_keys`.
        *   assert: `k` non è in `tdd.dh.not_found_keys`.
        *   `res` = `tdd.prepare_response_not_free(r, tdd.get_record_for_key(tdd.get_key_from_request(r)))`.
        *   Risponde con `res`.
        *   L'algoritmo termina.
    *   Se `ttl_db_is_exhaustive(tdd, k)`:
        *   Se `ttl_db_is_out_of_memory(tdd)`:
            *   Esegui `ttl_db_remove_not_found(tdd, k)`.
            *   Esegui `ttl_db_add_not_exhaustive(tdd, k)`.
            *   Rilancia `PeersRefuseExecutionError.OUT_OF_MEMORY`.
            *   L'algoritmo termina.
        *   Altrimenti:
            *   Esegui `ttl_db_remove_not_found(tdd, k)`.
            *   Esegui `ttl_db_remove_not_exhaustive(tdd, k)`.
            *   Elabora l'inserimento in memoria di `k`. Ottiene la risposta da restituire. `res` = `tdd.execute(r)`.
            *   Se **not** `tdd.my_records_contains(k)`:
                *   Esegui `ttl_db_add_not_found(tdd, k)`.
            *   Risponde con `res`.
            *   L'algoritmo termina.
    *   Altrimenti:
        *   Se la chiave `k` è nell'elenco `tdd.dh.retrieving_keys`, il quale gli associa un canale di comunicazione `ch`:
            *   Aspetta sul canale di comunicazione `ch` fino a un massimo di `tdd.get_timeout_exec(r)` - 1000.
            *   Rilancia `PeersRedoFromStartError`.
            *   L'algoritmo termina.
        *   Altrimenti:
            *   Se **not** `ttl_db_is_out_of_memory(tdd)`:
                *   Esegui `ttl_db_start_retrieve(tdd, k)`.
            *   Esegui `ttl_db_remove_not_found(tdd, k)`.
            *   Esegui `ttl_db_add_not_exhaustive(tdd, k)`.
            *   Rilancia `PeersRefuseExecutionError.NOT_EXHAUSTIVE`.
            *   L'algoritmo termina.
*   Se `tdd.is_read_only_request(r)`:
    *   `Object k` = `tdd.get_key_from_request(r)`.
    *   Se `tdd.my_records_contains(k)`:
        *   assert: `k` non è in `tdd.dh.not_exhaustive_keys`.
        *   assert: `k` non è in `tdd.dh.not_found_keys`.
        *   Elabora la richiesta di lettura di `k`. Ottiene la risposta da restituire. `res` = `tdd.execute(r)`.
        *   L'algoritmo termina.
    *   Se `ttl_db_is_exhaustive(tdd, k)`:
        *   Esegui `ttl_db_add_not_found(tdd, k)`.
        *   `res` = `tdd.prepare_response_not_found(r)`.
        *   Risponde con `res`.
        *   L'algoritmo termina.
    *   Altrimenti:
        *   Rilancia `PeersRefuseExecutionError.NOT_EXHAUSTIVE`.
        *   L'algoritmo termina.
*   Se `r` is `RequestWaitThenSendRecord`:
    *   `Object k` = `r.k`.
    *   Se **not** `tdd.my_records_contains(k)` **e not** `ttl_db_is_exhaustive(tdd, k)`:
        *   Rilancia `PeersRefuseExecutionError.NOT_EXHAUSTIVE`.
        *   L'algoritmo termina.
    *   Calcola *𝛿* il tempo critico di coerenza, basandosi sul numero approssimato di nodi nel minimo comune
        g-nodo tra il nodo corrente e quello del richiedente, cioè il proprio g-nodo di livello `common_lvl`.
    *   Attende *𝛿* o al massimo `RequestWaitThenSendRecord.timeout_exec` - 1000.
    *   Se `tdd.my_records_contains(k)`:
        *   assert: `k` non è in `tdd.dh.not_exhaustive_keys`.
        *   assert: `k` non è in `tdd.dh.not_found_keys`.
        *   `ret` = `new RequestWaitThenSendRecordResponse()`.
        *   `ret.record` = `tdd.get_record_for_key(k)`.
        *   Return `ret`.
        *   L'algoritmo termina.
    *   Se `ttl_db_is_exhaustive(tdd, k)`:
        *   Esegui `ttl_db_add_not_found(tdd, k)`.
        *   Return `new RequestWaitThenSendRecordNotFound()`.
        *   L'algoritmo termina.
    *   Altrimenti:
        *   Rilancia `PeersRedoFromStartError`.
        *   L'algoritmo termina.
*   Se `tdd.is_update_request(r)`:
    *   `Object k` = `tdd.get_key_from_request(r)`.
    *   Se `tdd.my_records_contains(k)`:
        *   assert: `k` non è in `tdd.dh.not_exhaustive_keys`.
        *   assert: `k` non è in `tdd.dh.not_found_keys`.
        *   Elabora la richiesta di modifica per `k`. Ottiene la risposta da restituire. `res` = `tdd.execute(r)`.
        *   L'algoritmo termina.
    *   Se `ttl_db_is_exhaustive(tdd, k)`:
        *   Esegui `ttl_db_add_not_found(tdd, k)`.
        *   `res` = `tdd.prepare_response_not_found(r)`.
        *   Risponde con `res`.
        *   L'algoritmo termina.
    *   Altrimenti:
        *   Se la chiave `k` è nell'elenco `tdd.dh.retrieving_keys`, il quale gli associa un canale di comunicazione `ch`:
            *   Aspetta sul canale di comunicazione `ch` fino a un massimo di `tdd.get_timeout_exec(r)` - 1000.
            *   Rilancia `PeersRedoFromStartError`.
            *   L'algoritmo termina.
        *   Altrimenti:
            *   Se **not** `ttl_db_is_out_of_memory(tdd)`:
                *   Esegui `ttl_db_start_retrieve(tdd, k)`.
            *   Esegui `ttl_db_remove_not_found(tdd, k)`.
            *   Esegui `ttl_db_add_not_exhaustive(tdd, k)`.
            *   Rilancia `PeersRefuseExecutionError.NOT_EXHAUSTIVE`.
            *   L'algoritmo termina.
*   Se `tdd.is_replica_value_request(r)`:
    *   `Object k` = `tdd.get_key_from_request(r)`.
    *   Se `tdd.my_records_contains(k)`:
        *   assert: `k` non è in `tdd.dh.not_exhaustive_keys`.
        *   assert: `k` non è in `tdd.dh.not_found_keys`.
        *   Elabora la richiesta di replica di valorizzazione per `k`. Ottiene la risposta da restituire. `res` = `tdd.execute(r)`.
        *   assert: `tdd.my_records_contains(k)`.
        *   Restituisce `res`. L'algoritmo termina.
    *   Se **not** `ttl_db_is_out_of_memory(tdd)`:
        *   Esegui `ttl_db_remove_not_found(tdd, k)`.
        *   Esegui `ttl_db_remove_not_exhaustive(tdd, k)`.
        *   Elabora la richiesta di replica per `k`. Ottiene la risposta da restituire. `res` = `tdd.execute(r)`.
        *   assert: `tdd.my_records_contains(k)`.
        *   Restituisce `res`. L'algoritmo termina.
    *   Altrimenti:
        *   Esegui `ttl_db_remove_not_found(tdd, k)`.
        *   Esegui `ttl_db_add_not_exhaustive(tdd, k)`.
        *   Rilancia `PeersRefuseExecutionError.OUT_OF_MEMORY`.
        *   L'algoritmo termina.
*   Se `tdd.is_replica_delete_request(r)`:
    *   `Object k` = `tdd.get_key_from_request(r)`.
    *   Elabora la richiesta di replica di cancellazione per `k`. Ottiene la risposta da restituire. `res` = `tdd.execute(r)`.
    *   assert: **not** `tdd.my_records_contains(k)`.
    *   Esegui `ttl_db_add_not_found(tdd, k)`.
    *   Esegui `ttl_db_remove_not_exhaustive(tdd, k)`.
    *   Restituisce `res`. L'algoritmo termina.
*   Nessuno dei casi precedenti.
*   Elabora la richiesta `r`. Ottiene la risposta da restituire. `res` = `tdd.execute(r)`.
*   Risponde con `res`.
*   L'algoritmo termina.

Algoritmo di avvio del recupero (in una nuova tasklet) del record per la chiave *k*:

**internal void ttl_db_start_retrieve(ITemporalDatabaseDescriptor tdd, Object k)**

*   Crea un canale di comunicazione `ch`.
*   Mette la chiave `k` nell'elenco `tdd.dh.retrieving_keys`, associandogli il canale `ch`.
*   In una nuova tasklet:
    *   Avvia il procedimento di recupero del record per la chiave `k`.  
        Si tratta di una richiesta di sola lettura (con attesa del tempo di coerenza) in cui il
        nodo *n* è il client e si auto-esclude come servente.  
        Gli esiti possono essere:
        *   Una istanza `res` di RequestWaitThenSendRecordResponse.  
            Significa che abbiamo recuperato il record.
        *   L'eccezione PeersNoParticipantsInNetworkError o l'eccezione PeersDatabaseError o una
            istanza di RequestWaitThenSendRecordNotFound o una istanza di una classe inattesa.  
            Significa che il record non esiste.  
            L'esito "DATABASE-ERROR" è da interpretare come un "NOT-FOUND", come per tutte le richieste che
            cercano un record esistente.  
            L'esito "NO-PARTICIPANTS" è come un "NOT-FOUND", visto che il nodo *n* è partecipante e la
            chiave `k` non è nella sua memoria.
    *   `Object? record` = `null`.
    *   `IPeersRequest r` = `new RequestWaitThenSendRecord(k)`.
    *   `timeout_exec` = `RequestWaitThenSendRecord.timeout_exec`.
    *   Try:
        *   `PeerTupleNode respondant` = `null`.
        *   `h_p_k` = `tdd.evaluate_hash_node(k)`.
        *   Esegue `IPeersResponse res = contact_peer(tdd.dh.p_id, h_p_k, r, timeout_exec, True, out respondant)`.
        *   Se `res` è un `RequestWaitThenSendRecordResponse`:
            *   `record` = `res.record`.
    *   Se riceve PeersNoParticipantsInNetworkError:
        *   Non fa nulla.
    *   Se riceve PeersDatabaseError:
        *   Non fa nulla.
    *   Se `record` ≠ `null` **e** `tdd.is_valid_record(k, record)`:
        *   Esegui `ttl_db_remove_not_exhaustive(tdd, k)`.
        *   Esegui `ttl_db_remove_not_found(tdd, k)`.
        *   Esegui `tdd.set_record_for_key(k, record)`.
    *   Altrimenti:
        *   Esegui `ttl_db_remove_not_exhaustive(tdd, k)`.
        *   Esegui `ttl_db_add_not_found(tdd, k)`.
    *   `INtkdChannel temp_ch` = `tdd.dh.retrieving_keys[k]`.
    *   Esegui `tdd.dh.retrieving_keys.unset(k)`.
    *   Invia un messaggio asincrono (senza schedulare altre tasklet) a tutti quelli che sono in attesa su `temp_ch`.

