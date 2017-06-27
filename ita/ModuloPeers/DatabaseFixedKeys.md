# Modulo PeerServices - Database con un numero esiguo e fisso di chiavi

1.  [Effetto di visibilità locale del dato](#Visibilita_locale)
1.  [Operazione di sola lettura](#Operazione_sola_lettura)
1.  [Operazione di scrittura](#Operazione_scrittura)
1.  [Operazione di replica](#Operazione_replica)
1.  [Fase iniziale](#Fase_iniziale)
1.  [Esaustività del nodo servente](#Esaustivita_nodo_servente)
1.  [Algoritmi](#Algoritmi)

Implementiamo un servizio che mantiene un database distribuito.  Le chiavi sono in un dominio ben definito e non
molto grande.

Ogni nodo partecipante può essere chiamato a mantenere in memoria un record per ognuna delle chiavi nel dominio:
quindi non è previsto il caso di OUT-OF-MEMORY. Inoltre esiste, per ogni chiave, un record prefissato che ogni nodo
partecipante può usare come record di default quando elabora una richiesta di lettura e nessuno aveva in precedenza
salvato un valore: quindi non è previsto il caso di NOT-FOUND.

In questo servizio ipotiziamo che non ci siano vincoli per il client, cioè qualsiasi operazione è permessa (nei
limiti del dominio delle chiavi e dei valori) senza alcun tipo di autorizzazione. Comunque risulterà facile
implementare in un servizio analogo qualsiasi tipo di vincolo da parte del server; basterà codificare nella
classe del valore di ritorno le eccezioni previste dal servizio.

Diamo per assunto che il lettore abbia già letto il documento [Dettagli Tecnici](DettagliTecnici.md).

## <a name="Visibilita_locale"></a>Effetto di visibilità locale del dato

Opzionalmente, il servizio può stabilire di dare ad alcune chiavi un effetto di visibilità locale del dato. Con
questo intendiamo che per una chiave *k* ∈ *K* si può identificare un livello *l*. Se un nodo *n* fa richiesta
di memorizzare il valore *v* per la chiave *k*, tale valore sarà visibile ai soli nodi appartenenti allo stesso
g-nodo di livello *l* a cui appartiene il nodo *n*. In altri g-nodi il valore associato alla chiave *k* sarà distinto.

Per ottenere questo sarà sufficiente che la classe del servizio implementi il metodo `evaluate_hash_node` della
sua istanza di IDatabaseDescriptor in modo da restituire, quando il parametro è la chiave *k*, una tupla di
soli *l* elementi. La stessa implementazione ovviamente dovrà essere usata dalla classe del client del servizio
nel suo metodo `perfect_tuple`.

## <a name="Operazione_sola_lettura"></a>Operazione di sola lettura

Il nodo *q* vuole leggere il valore del record per una chiave *k* nel database distribuito.

Ci sono 2 possibili esiti finali a questa richiesta:

1.  Il nodo *q* viene informato che al momento nella rete non c'è nessun partecipante al servizio. Chiamiamolo
    esito "NO-PARTICIPANTS".
1.  Il nodo *q* viene informato che il record per la chiave *k* ha il valore *v*. Chiamiamolo esito "OK".

## <a name="Operazione_scrittura"></a>Operazione di scrittura

Il nodo *q* vuole modificare il valore del record per una chiave *k* nel database distribuito.

Ci sono 2 possibili esiti finali a questa richiesta:

1.  Il nodo *q* viene informato che al momento nella rete non c'è nessun partecipante al servizio. Chiamiamolo
    esito "NO-PARTICIPANTS".
1.  Il nodo *q* viene informato che il record per la chiave *k* è stato modificato. Chiamiamolo esito "OK".

## <a name="Operazione_replica"></a>Operazione di replica

Il nodo *n0* vuole replicare un record *k=w* che esso ha appena scritto nella sua memoria. La richiesta giunge
ad un nodo che viene dopo *n0* nella ricerca dell'hash-node.

Ci sono 2 possibili esiti finali a questa richiesta:

1.  Il nodo *n0* viene informato che al momento nella rete non c'è nessun partecipante al servizio. Chiamiamolo
    esito "NO-PARTICIPANTS".
1.  Il nodo *n0* viene informato che il record *k=w* è stato replicato. Chiamiamolo esito "OK".

## <a name="Fase_iniziale"></a>Fase iniziale

Sia *n* un nodo partecipante al servizio. Il nodo crea una istanza *p* della classe che rappresenta
il servizio, ereditando dalla classe base PeerService. Possono esserci due casi:

1.  Il nodo *n* crea una nuova rete.  
    In questo caso il costruttore di *p* pone:

    *   `guest_gnode_level` = *-1*. Significa che non c'era una precedente identità da cui reperire uno stato.
    *   `new_gnode_level` = *levels*. Significa che è stato formato un nuovo g-nodo di livello *levels*.

1.  Il nodo *n* entra in una rete. Questa situazione può avvenire in due modi: o il nodo entra in una rete diversa
    da quella in cui si trovava prima, oppure migra in un diverso g-nodo all'interno della stessa rete. In entrambi
    i casi con *n* indichiamo la nuova identità del nodo. Questa fa ingresso in blocco insieme ad un vecchio g-nodo
    *g* di livello *i* (con `0 ≤ i < levels`) all'interno di un g-nodo esistente *w* di livello *j* (con `i < j ≤ levels`)
    assumendo una nuova posizione al livello *j-1* assegnata da *w*.  
    In questo caso il costruttore di *p* pone:

    *   `guest_gnode_level` = *i*. Significa che dalla precedente identità si devono reperire le informazioni
        relative ai dati con visibilità locale ai livelli inferiori a *i*.
    *   `new_gnode_level` = *j-1*. Significa che è stato formato un nuovo g-nodo di livello *j-1*.

    Inoltre in questo caso il costruttore di *p* deve essere messo in grado di recuperare dalla sua precedente
    identità alcune informazioni. Cioè deve avere un riferimento alla precedente istanza della classe del servizio.

Sia *K* l'insieme completo delle chiavi del servizio, preferibilmente ordinato per avere per prime le chiavi
che hanno una visibilità locale più ristretta, se il servizio lo prevede.

Immediatamente *p* avvia un esame su tutte le possibili chiavi *k* ∈ *K*. Sia *l* il livello del g-nodo
in cui la visibilità del dato per la chiave *k* è circoscritta, oppure sia *l* = *levels*.

Se `guest_gnode_level` ≥ *l*, allora *p* mette nella sua memoria il record che era memorizzato
dalla precedente istanza. Inoltre mette a *completato* lo stato dell'operazione iniziale di recupero del record
per la chiave *k*.

Altrimenti, se `new_gnode_level` ≥ *l*, allora *p* mette nella sua memoria il record di
default associato a *k*. Inoltre mette a *completato* lo stato dell'operazione iniziale di recupero del record
per la chiave *k*.

Altrimenti, *p* non ha informazioni per inizializzare il record per la chiave *k*. Di seguito analiziamo cosa fa
il servizio *p* in questo caso.

Il servizio *p* avvia un [procedimento di recupero](DettagliTecnici.md#Procedimento_di_recupero_di_un_record) del
record associato a *k*, con attesa del tempo critico di coerenza *𝛿*.

Il procedimento di recupero consiste nell'inviare la richiesta al nodo che in precedenza era detentore del record
per la chiave *k*, chiamiamolo *current(k)*.

Tali richieste vanno intervallate tra di loro per non appesantire la rete, ma non troppo tempo: infatti non ci
sono altri punti in cui le operazioni di recupero vengono avviate e il nodo corrente potrebbe essere chiamato
a servire un record. Quindi il client dovrebbe attendere a lungo. Il tempo massimo per l'espletamento di queste
operazioni iniziali dovrebbe essere sufficientemente contenuto per il fatto che il numero delle chiavi è esiguo
(come prevede questa tipologia di servizi) e che le chiavi esaminate per prime sono quelle che hanno una visibilità
locale più ristretta. Diciamo dunque che dopo aver avviato (in una nuova tasklet) l'operazione di recupero di un
record (ci riferiamo quindi ai record che non vengono inizializzati da *n*) si attendono 200 millisecondi.

Come detto, tale procedimento prevede che se il servizio *p* nel nodo *n* durante questo tempo riceve richieste per la chiave
*k* che sono di scrittura, allora non le rifiuta subito, ma le mette in attesa. Siccome ora le richieste di
scrittura per la chiave *k* passano prima per il nodo *n* che le fa attendere, di sicuro non giungeranno al
*current(k)*.

E' possibile che queste richieste avvengano in modo circoscritto ad un g-nodo di appartenenza di *n*, se il
servizio come detto prima decide di dare ad alcune chiavi un effetto di visibilità locale del dato.

E' possibile che la richiesta per la chiave *k* restituisca una eccezione PeersNoParticipantsInNetworkError.
Ad esempio se il servizio è opzionale e nessuno dei nodi già esistenti (eventualmente nel g-nodo in cui la
ricerca è stata circoscritta) vi partecipa. In questo caso il nodo corrente, il quale partecipa al servizio,
si può ritenere *esaustivo* per *k*. Quindi, a fronte di questa eccezione, il nodo associa alla chiave *k*
nella sua memoria il record di default per tale chiave.

## <a name="Esaustivita_nodo_servente"></a>Esaustività del nodo servente

Esaminiamo ora cosa avviene per le richieste, di qualsiasi tipo, che il nodo *n* riceve da altri.

Quando il servizio *p* nel nodo *n* riceve una richiesta *r* per una chiave *k*, il suo comportamento dipende da due fattori:

*   Lo stato dell'operazione iniziale di recupero del record per la chiave *k*. Può essere *completato* oppure no.
*   Il tipo della richiesta *r*. Può essere *di sola lettura* oppure no.

Se la richiesta è di sola lettura e il recupero non è ancora completato, allora la rifiuta immediatamente
come *non esaustivo*, di modo che sarà servita dal precedente detentore.

A causa di questo comportamento, un client di questo servizio deve sapere che può ricevere una eccezione
PeersDatabaseError. Non dovrà interpretarla con un "record non trovato" ma dovrà riprovare a fare la
richiesta dopo un po' di tempo: come detto prima gli unici esiti per una richiesta di lettura sono
"NO-PARTICIPANTS" e "OK".

Se la richiesta non è di sola lettura e il recupero non è ancora completato, allora la mette in attesa finché
non ha completato o, al massimo, fino quasi ad esaurire il tempo previsto per l'esecuzione della richiesta
(`timeout_exec`) e poi da istruzioni al client di riavviare da capo il calcolo distribuito di H<sub>t</sub>,
rilanciando una eccezione PeersRedoFromStartError. Questo perché comunque potrebbe non essere più lui il
miglior candidato.

Se il recupero del record è stato completato, qualunque sia il tipo della richiesta, la serve immediatamente.

Per gestire questo aspetto il servizio *p* fa uso di alcune strutture dati memorizzate nella classe DatabaseHandler:

*   Elenco di chiavi `List<Object> not_completed_keys`.  
    Se una chiave *k* è nell'elenco allora il nodo sa che il recupero del relativo record non è ancora
    stato completato.

## <a name="Algoritmi"></a>Algoritmi

La classe del servizio deve fornire, oltre alle operazioni previste in tutti i servizi che implementano
un [database distribuito](DettagliTecnici.md#Mantenimento_di_un_database_distribuito), queste ulteriori operazioni:

*   `List<Object> get_full_key_domain()`: Restituire la lista di tutte le possibili chiavi nel dominio.
    Preferibilmente tale elenco deve essere ordinato per avere per prime le chiavi che eventualmente prevedono
    una visibilità locale.
*   `Object get_default_record_for_key(Object k)`: Restituire una istanza (da mettere in memoria) come valore
    di default per la chiave *k*, avendo come requisito (cioè il chiamante deve garantirlo) che tale chiave è
    del dominio del servizio, come indicato dal metodo precedente. Questa funzione deve garantire di essere
    atomica, cioè di non schedulare altre tasklet. 

Il modulo fornisce l'interfaccia IFixedKeysDatabaseDescriptor. Essa estende l'interfaccia IDatabaseDescriptor
ed espone anche i metodi sopra descritti: `get_full_key_domain` e `get_default_record_for_key`.

La classe che implementa il servizio nel suo costruttore crea una istanza di IFixedKeysDatabaseDescriptor che
userà in tutte le chiamate a questi algoritmi, che sono metodi di PeersManager. Subito richiamerà il metodo
`fixed_keys_db_on_startup`. In seguito per ogni richiesta che riceve richiamerà il metodo `fixed_keys_db_on_request`.

Algoritmo all'avvio:

**void fixed_keys_db_on_startup(IFixedKeysDatabaseDescriptor fkdd, int p_id, int guest_gnode_level, int new_gnode_level, IFixedKeysDatabaseDescriptor? prev_id_fkdd)**

*   assert: `services.has_key(p_id)`.
*   In una nuova tasklet:
    *   `srv` = `services[p_id]`.
    *   `fkdd.dh` = `new DatabaseHandler()`.
    *   `fkdd.dh.p_id` = `p_id`.
    *   `fkdd.dh.not_completed_keys` = `new ArrayList<Object>(fkdd.key_equal_data)`.
    *   `fkdd.dh.retrieving_keys` = `new HashMap<Object,INtkdChannel>(fkdd.key_hash_data, fkdd.key_equal_data)`.
    *   `List<Object> k_set` = `fkdd.get_full_key_domain()`.
    *   Per ogni chiave `Object k` in `k_set`:
        *   Mette `k` in `fkdd.dh.not_completed_keys`.
    *   `wait_before_network_activity` = `False`.
    *   Per ogni chiave `Object k` in `k_set`:
        *   `h_p_k` = `fkdd.evaluate_hash_node(k)`.
        *   `l` = `h_p_k.size`.
        *   Se `guest_gnode_level` ≥ `l`:
            *   Esegui `fkdd.set_record_for_key(k, prev_id_fkdd.get_record_for_key(k))`.
            *   Rimuove `k` da `fkdd.dh.not_completed_keys`.
        *   Altrimenti-Se `new_gnode_level` ≥ `l`:
            *   Esegui `fkdd.set_record_for_key(k, fkdd.get_default_record_for_key(k))`.
            *   Rimuove `k` da `fkdd.dh.not_completed_keys`.
        *   Altrimenti:
            *   Se `wait_before_network_activity`:
                *   Attendi 200 millisecondi.
            *   Esegue `fixed_keys_db_start_retrieve(fkdd, k)`. Cioè avvia in una tasklet il recupero del record per la chiave `k`.
            *   `wait_before_network_activity` = `True`.

Algoritmo alla ricezione della richiesta:

**IPeersResponse fixed_keys_db_on_request(IFixedKeysDatabaseDescriptor fkdd, IPeersRequest r, int common_lvl) throws PeersRefuseExecutionError, PeersRedoFromStartError**

*   Gli argomenti del metodo `fixed_keys_db_on_request` sono:
    *   `fkdd`: istanza di una classe che implementa IFixedKeysDatabaseDescriptor sopra descritta.
    *   `r`: la richiesta.
    *   `common_lvl`: il livello del minimo comune g-nodo con il richiedente, da 0 a `levels` compresi.
*   Se `fkdd.dh` = `null`:
    *   Rilancia `PeersRefuseExecutionError.NOT_EXHAUSTIVE`.
    *   L'algoritmo termina.
*   Se `fkdd.is_read_only_request(r)`:
    *   `Object k` = `fkdd.get_key_from_request(r)`.
    *   Se `k` non è in `fkdd.dh.not_completed_keys`:
        *   assert: `fkdd.my_records_contains(k)`.
        *   Elabora la richiesta di lettura di `k`. Ottiene la risposta da restituire. `res` = `fkdd.execute(r)`.
        *   L'algoritmo termina.
    *   Altrimenti:
        *   Rilancia `PeersRefuseExecutionError.NOT_EXHAUSTIVE`.
        *   L'algoritmo termina.
*   Se `r` is `RequestWaitThenSendRecord`:
    *   `Object k` = `r.k`.
    *   Se `k` è in `fkdd.dh.not_completed_keys`:
        *   Rilancia `PeersRefuseExecutionError.NOT_EXHAUSTIVE`.
        *   L'algoritmo termina.
    *   assert: `fkdd.my_records_contains(k)`.
    *   Calcola `𝛿` il tempo critico di coerenza, basandosi sul numero approssimato di nodi nel minimo comune
        g-nodo tra il nodo corrente e quello del richiedente, cioè il proprio g-nodo di livello `common_lvl`.
    *   Attende `𝛿` o al massimo `RequestWaitThenSendRecord.timeout_exec` - 1000.
    *   assert: `fkdd.my_records_contains(k)`.
    *   `ret` = `new RequestWaitThenSendRecordResponse()`.
    *   `ret.record` = `fkdd.get_record_for_key(k)`.
    *   Return `ret`.
    *   L'algoritmo termina.
*   Se `fkdd.is_update_request(r)`:
    *   `Object k` = `fkdd.get_key_from_request(r)`.
    *   Se `k` non è in `fkdd.dh.not_completed_keys`:
        *   assert: `fkdd.my_records_contains(k)`.
        *   Elabora la richiesta di modifica per `k`. Ottiene la risposta da restituire. `res` = `fkdd.execute(r)`.
        *   L'algoritmo termina.
    *   Altrimenti:
        *   Mentre la chiave `k` non è ancora nell'elenco `fkdd.dh.retrieving_keys`:
            *   Aspetta 200 millisecondi.
        *   Recupera il canale di comunicazione `ch` = `fkdd.dh.retrieving_keys[k]`.
        *   Aspetta sul canale di comunicazione `ch` fino a un massimo di `fkdd.get_timeout_exec(r)` - 1000.
        *   Rilancia `PeersRedoFromStartError`.
        *   L'algoritmo termina.
*   Se `fkdd.is_replica_value_request(r)`:
    *   `Object k` = `fkdd.get_key_from_request(r)`.
    *   Elabora la richiesta di replica di valorizzazione per `k`. Ottiene la risposta da restituire. `res` = `fkdd.execute(r)`.
    *   Restituisce `res`. L'algoritmo termina.
*   Nessuno dei casi precedenti.
*   Elabora la richiesta `r`. Ottiene la risposta da restituire. `res` = `fkdd.execute(r)`.
*   Risponde con `res`.
*   L'algoritmo termina.

Algoritmo di avvio del recupero (in una nuova tasklet) del record per la chiave `k`:

**internal void fixed_keys_db_start_retrieve(IFixedKeysDatabaseDescriptor fkdd, Object k)**

*   Crea un canale di comunicazione `ch`.
*   Mette la chiave `k` nell'elenco `fkdd.dh.retrieving_keys`, associandogli il canale `ch`.
*   In una nuova tasklet:
    *   Se il servizio è opzionale:
        *   Aspetta che siano reperite le mappe di partecipazione per i servizi opzionali almeno
            fino al livello in cui va circoscritta la ricerca dell'hash-node per la chiave `k`.
    *   Avvia il procedimento di recupero del record per la chiave `k`.  
        Si tratta di una richiesta di sola lettura (con attesa del tempo di coerenza) in cui il nodo `n` è
        il client e si auto-esclude come servente.  
        Gli esiti possono essere:
        *   Una istanza `res` di RequestWaitThenSendRecordResponse.  
            Significa che abbiamo recuperato il record.
        *   L'eccezione PeersNoParticipantsInNetworkError o una istanza di una classe inattesa.  
            Significa che va usato come record il valore di default.
    *   `Object? record` = `null`.
    *   `IPeersRequest r` = `new RequestWaitThenSendRecord()`.
    *   `timeout_exec` = `RequestWaitThenSendRecord.timeout_exec`.
    *   Mentre True:
        *   Try:
            *   `PeerTupleNode respondant` = `null`.
            *   `h_p_k` = `fkdd.evaluate_hash_node(k)`.
            *   Esegue `IPeersResponse res = contact_peer(fkdd.dh.p_id, h_p_k, r, timeout_exec, True, out respondant)`.
            *   Se `res` è un `RequestWaitThenSendRecordResponse`:
                *   `record` = `res.record`.
            *   Esce dal ciclo.
        *   Se riceve PeersNoParticipantsInNetworkError:
            *   Esce dal ciclo.
        *   Se riceve PeersDatabaseError:
            *   Aspetta 200 millisecondi. Riproverà alla prossima iterazione del ciclo.
    *   Se `record` ≠ `null` **e** `fkdd.is_valid_record(k, record)`:
        *   Esegui `fkdd.set_record_for_key(k, record)`.
    *   Altrimenti:
        *   Esegui `fkdd.set_record_for_key(k, fkdd.get_default_record_for_key(k))`.
    *   `INtkdChannel temp_ch` = `fkdd.dh.retrieving_keys[k]`.
    *   Esegui `fkdd.dh.retrieving_keys.unset(k)`.
    *   Invia un messaggio asincrono (senza schedulare altre tasklet) a tutti quelli che sono in attesa su `temp_ch`.
    *   Rimuove `k` da `fkdd.dh.not_completed_keys`.

