# Modulo Coordinator - Dettagli Tecnici

1.  [Requisiti](#Requisiti)
1.  [Deliverables](#Deliverables)
1.  [Comunicazioni tra vicini](#Comunicazioni_tra_vicini)
1.  [Servizio Coordinator](#Coordinator_service)
    1.  [Classe server del servizio Coordinator](#Coordinator_server)
        1.  [Servire una richiesta](#Coordinator_server_generic)
        1.  [Richiesta reserve](#Coordinator_server_reserve)
    1.  [Classe client del servizio Coordinator](#Coordinator_client)
        1.  [Inviare una richiesta](#Coordinator_client_generic)
        1.  [Richiesta reserve](#Coordinator_client_reserve)
    1.  [Classi serializzabili per le comunicazioni](#Serializables)

## <a name="Requisiti"></a>Requisiti

L'utilizzatore del modulo Coordinator inizializza il modulo richiamando il metodo statico `init`
di CoordinatorManager.  In tale metodo viene anche passata l'istanza di ITasklet per fornire
l'implementazione del sistema di tasklet.

Appena il nodo *n* entra in una rete o ne costituisce una nuova, istanzia il suo CoordinatorManager.

1.  Il nodo *n* crea una nuova rete.  
    In questo caso il nodo passa al costruttore di CoordinatorManager:

    *   `guest_gnode_level` = *-1*. Significa che non c'era una precedente identità.
    *   `new_gnode_level` = *levels*. Significa che è stato formato un nuovo g-nodo di livello *levels*.
    *   `prev_id_coord` = *null*. Significa che non c'era una precedente identità.

1.  Il nodo *n* entra in una rete. Questa situazione può avvenire in due modi: o il nodo entra in una rete diversa
    da quella in cui si trovava prima, oppure migra in un diverso g-nodo all'interno della stessa rete. In entrambi
    i casi con *n* indichiamo la nuova identità del nodo. Questa fa ingresso in blocco insieme ad un vecchio g-nodo
    *g* di livello *i* (con `0 ≤ i < levels`) all'interno di un g-nodo esistente *w* di livello *j* (con `i < j ≤ levels`)
    assumendo una nuova posizione al livello *j-1* assegnata da *w*.  
    In questo caso il nodo passa al costruttore di CoordinatorManager:

    *   `guest_gnode_level` = *i*. Significa che dalla precedente identità si devono reperire le informazioni
        relative ai g-nodi ai livelli inferiori a *i*.
    *   `new_gnode_level` = *j-1*. Significa che è stato formato un nuovo g-nodo di livello *j-1*.
    *   `prev_id_coord` = la precedente identità di CoordinatorManager.

    Inoltre in questo caso il costruttore di *p* deve essere messo in grado di recuperare dalla sua precedente
    identità alcune informazioni. Cioè deve avere un riferimento alla precedente istanza della classe del servizio.

Quando il nodo ha completato la fase di bootstrap del modulo QSPN, il nodo informa il suo CoordinatorManager
chiamando il metodo `bootstrap_completed` a cui passa:

*   `peers_manager`. L'istanza di PeersManager.
*   `map`. La mappa delle posizioni libere.

Prima di questo evento il nodo non è in grado di eseguire i metodi remoti chiamati da un suo vicino. Questa
impossibilità viene segnalata con l'eccezione CoordinatorNodeNotReadyError.

## <a name="Deliverables"></a>Deliverables

Quando viene chiamato il suo metodo `bootstrap_completed`, il CoordinatorManager crea una istanza di
CoordinatorService (che descriveremo sotto) e la registra nel PeersManager.

* * *

Il modulo permette di chiedere ad un vicino del nodo informazioni sui posti liberi nei suoi g-nodi con il
metodo `get_neighbor_map` di CoordinatorManager.

Il metodo ha come argomento lo stub per contattare il vicino. Prevede le eccezioni CoordinatorStubNotWorkingError
e CoordinatorNodeNotReadyError.

Restituisce una istanza di ICoordinatorNeighborMap.

* * *

Il modulo permette di chiedere ad un vicino del nodo di prenotare per lui un posto nel suo g-nodo di livello
*l* con il metodo `get_reservation` di CoordinatorManager.

Il metodo ha come argomento lo stub per contattare il vicino e il livello a cui fare richiesta. Prevede le
eccezioni CoordinatorStubNotWorkingError, CoordinatorNodeNotReadyError, CoordinatorInvalidLevelError e
CoordinatorSaturatedGnodeError.

Restituisce una istanza di ICoordinatorReservation.

## <a name="Comunicazioni_tra_vicini"></a>Comunicazioni tra vicini

Quando il modulo vuole chiedere ad un vicino informazioni sui posti liberi nei suoi g-nodi usa il metodo
remoto `retrieve_neighbor_map`. Non ha argomenti. Prevede l'eccezione CoordinatorNodeNotReadyError, oltre
alle solite StubError e DeserializeError. Restituisce una istanza di ICoordinatorNeighborMapMessage.

L'interfaccia ICoordinatorNeighborMapMessage è un segnaposto (vuota) esposto dalla libreria intermedia di ZCD
che espone il metodo remoto. L'interfaccia ICoordinatorNeighborMap è esposta dal modulo Coordinator e ha i metodi
descritti nell'analisi. L'implementazione del metodo remoto crea una istanza della classe NeighborMap. Questa è
una classe serializzabile interna al modulo, che implementa entrambe le interfacce suddette.

* * *

Quando il modulo vuole chiedere ad un vicino di prenotare per lui un posto nel suo g-nodo di livello *l* usa il
metodo remoto `ask_reservation`. Ha come argomento il livello *l*. Prevede le eccezioni CoordinatorNodeNotReadyError,
CoordinatorInvalidLevelError e CoordinatorSaturatedGnodeError, oltre alle solite StubError e DeserializeError.
Restituisce una istanza di ICoordinatorReservationMessage.

L'interfaccia ICoordinatorReservationMessage è un segnaposto (vuota) esposto dalla libreria intermedia di ZCD che
espone il metodo remoto. L'interfaccia ICoordinatorReservation è esposta dal modulo Coordinator e ha i metodi
descritti nell'analisi. L'implementazione del metodo remoto crea una istanza della classe Reservation. Questa è
una classe serializzabile interna al modulo, che implementa entrambe le interfacce suddette.

## <a name="Coordinator_service"></a>Servizio Coordinator

### <a name="Coordinator_server"></a>Classe server del servizio Coordinator

Il modulo deriva da PeerService la classe CoordinatorService. All'interno di tale classe (per poter accedere ai
suoi membri privati) definisce anche la classe DatabaseDescriptor che implementa l'interfaccia IFixedKeysDatabaseDescriptor.

La classe CoordinatorService è interna al modulo. Contiene questi membri:

*   `levels`. Intero. Il numero di livelli nella topologia della rete. Gli viene passato nel costruttore.
*   `peers_manager`. Riferimento al PeersManager. Gli viene passato nel costruttore.
*   `fkdd`. Contiene una istanza di DatabaseDescriptor valorizzata nel costruttore.
*   `booking_lists[i]`, con *i* da 0 a *levels* - 1. Lista di Booking. Memoria delle prenotazioni. La lista di
    prenotazioni attive per il nostro g-nodo di livello *i* + 1. Il costruttore la crea vuota.  
    La classe Booking ha una posizione (int *pos*) e un time-to-live (Timer *ttl*). E' una classe serializzabile
    e interna al modulo.
*   `max_elderships[i]`, con *i* da 0 a *levels* - 1. Intero. Il valore più alto di progressione già assegnato ad
    una prenotazione per un nuovo g-nodo di livello *i* dentro il nostro g-nodo di livello *i* + 1. Il costruttore
    lo inizializza a 0.  
    Quando assegnato ad un g-nodo, un valore di anzianità più alto significa che il g-nodo è arrivato dopo, cioè
    esso è più giovane.

Nel costruttore, dopo aver valorizzato i membri nel modo suddetto, viene fatta la registrazione sul PeersManager.
Dopo, in una tasklet, si avvia il metodo `fixed_keys_db_on_startup` del PeersManager passando l'istanza *fkdd* e
il livello del nuovo g-nodo costituito (che viene passato al costruttore). Questo per avvalersi degli algoritmi di
gestione delle problematiche di mantenimento di un database a chiavi fisse.

Quando viene chiamato il metodo `get_record_for_key` dell'istanza *fkdd*, la classe CoordinatorService fa queste operazioni:

*   Assert: La chiave *k* passata è una istanza valida di CoordinatorKey.
*   Valorizza e restituisce una istanza di CoordinatorRecord con:
    *   `lvl`: lo stesso livello che è nella chiave: `k.lvl`.
    *   `booking_list`: una copia della lista nel membro `booking_lists[lvl - 1]` del CoordinatorService.
    *   `max_eldership`: l'intero che è nel membro `max_elderships[lvl - 1]` del CoordinatorService.

Quando viene chiamato il metodo `set_record_for_key` dell'istanza *fkdd*, la classe CoordinatorService fa queste operazioni:

*   Assert: La chiave *k* passata è una istanza valida di CoordinatorKey.
*   Assert: Il record *rec* passato è una istanza valida di CoordinatorRecord.
*   `booking_lists[rec.lvl - 1]` = una copia della lista in `rec.booking_list`.
*   `max_elderships[rec.lvl - 1]` = `rec.max_eldership`.

#### <a name="Coordinator_server_generic"></a>Servire una richiesta

Ricordiamo che quando una richiesta ad un servizio peer-to-peer viene ricevuta dall'hash_node viene chiamato il
metodo `exec` della classe che deriva la PeerService, nel nostro caso la classe CoordinatorService. Questo metodo
riceve una istanza di IPeersRequest (e la tupla che identifica il chiamante, ma in questo caso non la utiliziamo) e
deve restituire una istanza di IPeersResponse.

Il metodo `exec` della classe CoordinatorService, per avvalersi degli algoritmi di gestione delle problematiche di
mantenimento di un database a chiavi fisse, chiama il metodo `fixed_keys_db_on_request` del PeersManager passando
l'istanza *fkdd*. Questo metodo del modulo PeerServices restituisce il controllo al modulo Coordinator richiamando
sull'istanza *fkdd* il metodo `execute` definito così dall'interfaccia IFixedKeysDatabaseDescriptor:

*   `IPeersResponse execute(IPeersRequest r) throws PeersRefuseExecutionError, PeersRedoFromStartError`

#### <a name="Coordinator_server_reserve"></a>Richiesta reserve

Se il metodo `execute` della classe CoordinatorService.DatabaseDescriptor viene chiamato con una istanza di
CoordinatorReserveRequest (si veda sotto) allora abbiamo ricevuto una richiesta di "prenota un posto nel tuo
g-nodo di livello *lvl*". Le operazioni da fare sono:

*   Se l'argomento *lvl* ha un valore non accettabile viene fatta pervernire una eccezione CoordinatorInvalidLevelError.
    Lo si fa valorizzando appositamente i campi `error_*` di una nuova istanza di CoordinatorReserveResponse che
    viene restituita.
*   Atomic on: queste operazioni devono essere eseguite atomicamente, senza permettere la schedulazione di altre tasklet.
*   Verifica nella mappa *map* del CoordinatorManager quali posizioni sono libere nel livello *lvl*. Se nessuna
    posizione risulta libera, lancia una eccezione CoordinatorSaturatedGnodeError (campi `error_*` di
    CoordinatorReserveResponse).
*   Dalla lista ottenuta di posizioni libere, esclude quelle presenti nelle prenotazioni `booking_lists[lvl - 1]` del
    CoordinatorService. Nello scorrere questa lista, rimuove le istanze di Booking che sono scadute (membro *ttl*). Se
    nessuna posizione risulta libera, lancia una eccezione CoordinatorSaturatedGnodeError (campi `error_*` di
    CoordinatorReserveResponse).
*   Scegli una posizione *pos*.
*   Segnala la prenotazione nella memoria. Cioè crea una istanza di Booking *pos* con *ttl* a 60 secondi
    (una `private const` nel codice) nella lista `booking_lists[lvl - 1]` del CoordinatorService.
*   Incrementa di 1 la `max_elderships[lvl - 1]` nella memoria.
*   Con i dati in memoria prepara la risposta *ret*, una istanza di CoordinatorReserveResponse con:
    *   `pos` = `pos`.
    *   `eldership` = `max_elderships[lvl - 1]`.
*   Atomic off.
*   Avvia la prima replica con il metodo `PeersManager.begin_replica`. Questi sono gli argomenti:

    *   Come *q* mettiamo 15 (una `private const` nel codice).
    *   Come `perfect_tuple` il valore ottenuto con il metodo `perfect_tuple` della classe CoordinatorClient con la
        chiave *lvl*.
    *   Costruisce la richiesta *r* di tipo CoordinatorReplicaRecordRequest il cui contenuto si ottiene con
        il metodo `get_record_for_key` di *fkdd*.

    Attende l'esito, cioè il booleano restituito, la risposta *resp* da interpretare, l'oggetto IPeersContinuation
    *cont* da passare nelle chiamate successive.  
    Il booleano restituito è *False* solo se si è avuta l'eccezione NoParticipantsInNetwork. In questo caso si
    ignora la risposta *resp*.  
    La risposta *resp* dovrebbe essere semplicemente OK, cioè una istanza di CoordinatorReplicaReserveResponse con
    i membri a *null*. Altrimenti va prodotto un warning descrittivo da usare nella fase di debug del modulo; quando
    il modulo è stabile questa situazione va semplicemente ignorata.
*   Se l'esito è *True*:
    *   Avvia la seconda replica con il metodo `PeersManager.next_replica` passando la IPeersContinuation *cont*.  
        Attende l'esito, cioè il booleano restituito e la risposta *resp* da interpretare.  
        Il booleano restituito è *False* solo se si è avuta l'eccezione NoParticipantsInNetwork. In questo caso si
        ignora la risposta *resp*.  
        La risposta *resp* dovrebbe essere semplicemente OK. Altrimenti va prodotto un warning descrittivo.
    *   Se l'esito è *True*:
        *   Avvia una tasklet in cui:
            *   While True:
                *   Avvia una replica con il metodo `PeersManager.next_replica` passando la IPeersContinuation *cont*.  
                    Attende l'esito, cioè il booleano restituito e la risposta *resp* da interpretare.  
                    Il booleano restituito è *False* se abbiamo completato *q* repliche o se si è avuta l'eccezione
                    NoParticipantsInNetwork. In questo caso si ignora la risposta *resp*.  
                    La risposta *resp* dovrebbe essere semplicemente OK. Altrimenti va prodotto un warning descrittivo.
                    *   Se l'esito è *False*:
                        *   Esci dal ciclo. Termina la tasklet.
*   Restituisce *ret*.

### <a name="Coordinator_client"></a>Classe client del servizio Coordinator

Il modulo deriva da PeerClient la classe CoordinatorClient.

Ridefinisce il suo metodo `perfect_tuple`. La chiave che può essere usata per calcolare l'hash-node è un intero *l*
da 1 a *levels*. Si può generare la tupla inizialmente con l'implementazione di base del metodo `perfect_tuple` (quindi
occorre definire il metodo `hash_from_key`), ma poi la tupla restituita deve avere solo i primi *l* elementi.

#### <a name="Coordinator_client_generic"></a>Inviare una richiesta

Ricordiamo che questo servizio è non opzionale. Il nodo stesso che fa la richiesta è partecipante al servizio ed è
sempre incluso nel g-nodo in cui la ricerca viene eventualmente circoscritta. Quindi l'eccezione
PeersNoParticipantsInNetworkError rilanciata dal metodo `call` della classe base PeerClient può sempre essere
trattata come un errore.

Ricordiamo inoltre che la gestione di un database a chiavi fisse è tale che un nodo può rifiutarsi di rispondere
perché *non esaustivo* solo in operazioni di sola lettura. Nelle chiamate al metodo `call` della classe base PeerClient
fatte per richieste che non sono di sola lettura (ad esempio la richiesta 'reserve') l'eccezione PeersDatabaseError
può essere trattata come un errore.

#### <a name="Coordinator_client_reserve"></a>Richiesta reserve

Se il nodo corrente vuole prenotare un posto nel suo g-nodo di livello *lvl* fa al servizio Coordinator la richiesta
CoordinatorReserveRequest con la chiave *lvl*. Per farlo chiama il metodo `reserve` nella classe CoordinatorClient.

Il metodo prende a parametro il livello *lvl*. Restituisce una istanza di Reservation. Può rilanciare le
eccezioni: CoordinatorSaturatedGnodeError.

In caso di comportamento anomalo del nodo che risponde al servizio, il metodo della classe CoordinatorClient
rilancia una eccezione CoordinatorSaturatedGnodeError. Non possiamo in ogni caso sapere se il nodo che risponde
agisce in modo corretto; anche se sappiamo che non agisce correttamente non sappiamo se lo fa per errore o con
malizia; non possiamo sapere se risponderà in modo palesemente errato anche alle richieste provenienti da altri
nodi; in definitiva, cercare di porre rimedio diversamente sarebbe inutile.

Le operazioni da fare sono:

*   Verifica che il livello `lvl` sia nel range valido. Altrimenti il programma abortisce con un messaggio di errore.
*   Prepara la richiesta `r`, il tempo `timeout_exec`, la chiave `k`.
*   `IPeersResponse resp`.
*   `Reservation ret`.
*   Try:
    *   Richiama il metodo `call` della classe base. `resp` = `call(k, r, timeout_exec)`.
*   Se riceve PeersNoParticipantsInNetworkError:
    *   Il programma abortisce con un messaggio di errore.
*   Se riceve PeersDatabaseError:
    *   Il programma abortisce con un messaggio di errore.
*   Se `resp` è una istanza di CoordinatorReserveResponse:
    *   Se `resp.error_*` non sono tutte a *null*:
        *   Se i campi error_* indicano un CoordinatorSaturatedGnodeError:
            *   Rilancia l'eccezione CoordinatorSaturatedGnodeError con il messaggio ricevuto. Termina l'algoritmo.
        *   Scrive un warning riportando l'eccezione indicata e il messaggio ricevuto.
        *   Rilancia l'eccezione CoordinatorSaturatedGnodeError riportando l'eccezione indicata e il messaggio ricevuto. Termina l'algoritmo.
    *   Se `resp.pos` non è coerente con la topologia in `lvl` - 1, oppure è la mia stessa posizione in `lvl` - 1:
        *   Scrive un warning.
        *   Rilancia l'eccezione CoordinatorSaturatedGnodeError. Termina l'algoritmo.
    *   Se `resp.eldership < 1`:
        *   Scrive un warning.
        *   Rilancia l'eccezione CoordinatorSaturatedGnodeError. Termina l'algoritmo.
    *   Se `resp.pos` non è coerente con la topologia in `lvl` - 1:
        *   Scrive un warning.
        *   Rilancia l'eccezione CoordinatorSaturatedGnodeError. Termina l'algoritmo.
    *   Prepara `ret` una istanza di Reservation con:
        *   `levels`: `map.get_levels()`.
        *   `gsizes`: una lista di `levels` elementi: `gsizes[i] = map.get_gsize(i)`.
        *   `lvl`: `lvl - 1`.
        *   `pos`: `resp.pos`.
        *   `eldership`: `resp.eldership`.
        *   `upper_pos`: una lista di `levels - lvl - 1` elementi: `upper_pos[i] = map.get_my_pos(i+lvl+1)`.
        *   `upper_elderships`: una lista di `levels - lvl - 1` elementi: `upper_pos[i] = map.get_eldership(i+lvl+1)`.
*   Altrimenti:
    *   Sappiamo che `resp` è una classe non prevista:
    *   Scrive un warning.
    *   Rilancia l'eccezione CoordinatorSaturatedGnodeError. Termina l'algoritmo.
*   Restituisce `ret`.

### <a name="Serializables"></a>Classi serializzabili per le comunicazioni

Viene definita la classe interna CoordinatorKey. Si tratta di una classe serializzabile che deriva da Object. Serve
per la rappresentare una chiave del servizio, cioè il livello del g-nodo da coordinare. Tale classe contiene:

*   `lvl`. Un intero da 1 a `levels` che indica il livello del g-nodo da coordinare.

* * *

Viene definita la classe interna CoordinatorRecord. Si tratta di una classe serializzabile che deriva da Object.
Serve per la rappresentare un record del servizio, cioè la memoria condivisa del g-nodo da coordinare. Tale
classe contiene:

*   `lvl`. Un intero da 1 a `levels` che indica il livello del g-nodo da coordinare.
*   `booking_list`. Lista di Booking. Memoria delle prenotazioni. La lista di prenotazioni attive per il nostro
    g-nodo di livello `lvl`.
*   `max_eldership`. Intero. Il valore più alto di progressione già assegnato ad una prenotazione per un nuovo
    g-nodo di livello `lvl - 1` dentro il nostro g-nodo di livello `lvl`.

* * *

Viene definita la classe interna CoordinatorReserveRequest. Si tratta di una classe serializzabile che implementa
l'interfaccia segnaposto (vuota) IPeersRequest. Serve per la richiesta 'reserve'. Tale classe contiene:

*   `lvl`. Un intero che indica il livello del g-nodo dentro il quale riservare un posto.

* * *

Viene definita la classe interna CoordinatorReserveResponse. Si tratta di una classe serializzabile che implementa
l'interfaccia segnaposto (vuota) IPeersResponse. Serve come risposta per la richiesta CoordinatorReserveRequest. Tale
classe contiene:

*   `error_domain`. Una stringa nullable.
*   `error_code`. Una stringa nullable.
*   `error_message`. Una stringa nullable. Queste sono tutte a *null* se l'esito è buono, mentre sono valorizzate
    se si vuole indicare una eccezione.
*   `pos`. Un intero. La posizione assegnata al livello `lvl - 1`, dove `lvl` è il valore contenuto nella relativa
    richiesta CoordinatorReserveRequest.
*   `eldership`. Un intero. L'anzianità del g-nodo appena costituito al livello `lvl - 1`.

* * *

Viene definita la classe interna CoordinatorReplicaRecordRequest. Si tratta di una classe serializzabile che
implementa l'interfaccia segnaposto (vuota) IPeersRequest. Serve per la richiesta 'replica'. Tale classe contiene:

*   `record`. Una istanza di CoordinatorRecord.

* * *

Viene definita la classe interna CoordinatorReplicaRecordSuccessResponse. Si tratta di una classe serializzabile
che implementa l'interfaccia segnaposto (vuota) IPeersResponse. Serve come risposta per la richiesta
CoordinatorReplicaRecordRequest. Significa esito positivo. Tale classe non ha membri.

* * *

Viene definita la classe interna CoordinatorUnknownRequestResponse. Si tratta di una classe serializzabile che
implementa l'interfaccia segnaposto (vuota) IPeersResponse. Serve come risposta per le richieste che non sono
state riconosciute dall'implementazione del server. Tale classe non ha membri.

