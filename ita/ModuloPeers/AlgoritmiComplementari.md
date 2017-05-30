# Modulo PeerServices - Algoritmi complementari

1.  [Algoritmo di rilevamento di non partecipazione](#Rilevamento_non_partecipazione)
1.  [Algoritmo di divulgazione della partecipazione](#Divulgazione_partecipazione)
1.  [Algoritmi per il mantenimento di un database distribuito](#Mantenimento_database)
    1.  [Per migliorare la persistenza dei dati](#Persistenza_dati)
    1.  [Per garantire la coerenza dei dati](#Coerenza_dati)

## <a name="Rilevamento_non_partecipazione"></a>Algoritmo di rilevamento di non partecipazione

**check_non_participation**

Firma: `bool check_non_participation(p_id, lvl, _pos)`

*   In questo algoritmo non ci interessa sapere se un g-nodo partecipa, ma solo se è possibile dire con certezza che
    esso non partecipa. In caso di incertezza l'algoritmo restituisce False.
*   Produci `x̄` = la tupla *x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>lvl-1</sub>* dove *x̄<sub>i</sub>* = 0 per ogni *i*.
    La tupla identifica un indirizzo a caso all'interno del g-nodo `g` = `(lvl, _pos)`. Se `lvl` = 0 allora `x̄` è null.
*   Produci `n` = `make_tuple_node(new HCoord(0, pos[0]), lvl+1)` , cioè la tupla *n<sub>0</sub>·n<sub>1</sub>·...·n<sub>lvl</sub>*.
    La tupla che identifica il nodo corrente nel g-nodo di livello `lvl+1` in cui il messaggio si muoverà.
*   `m’` = `new PeerMessageForwarder`.
*   `m’.inside_level` = `levels`.
*   `m’.n` = `n`.
*   `m’.x̄` = `x̄`.
*   `m’.lvl` = `lvl`.
*   `m’.pos` = `_pos`.
*   `m’.p_id` = `p_id`.
*   `m’.msg_id` = un identificativo generato a caso per questo messaggio.
*   Calcola `timeout_instradamento` = *f* (`map_paths.i_peers_get_nodes_in_my_group(lvl + 1)`).
*   Calcola `PeerTupleGNode g` che identifica il g-nodo `(lvl,_pos)` con `top` = `lvl+1`.
*   Prepara `waiting_answer` = `new WaitingAnswer(null, g)`.  
    Il fatto che l'istanza di *IPeersRequest* è a *null* fa in modo che i metodi remoti che ricevono le notifiche
    si comportano in modo adeguato. Sostanzialmente dovrebbe cambiare solo il fatto che quando si riceve la segnalazione
    di `get_request` si risponde sempre con l'eccezione *PeersUnknownMessageError*, anche se si è potuto recuperare
    l'istanza di *WaitingAnswer*. Sull'istanza di *WaitingAnswer* viene poi valorizzato il membro response con qualcosa
    diverso da *null* solo per indicare che il g-nodo partecipa.
*   `waiting_answer_map[m’.msg_id]` = `waiting_answer`.
*   IPeersManagerStub `gwstub`
*   IPeersManagerStub? `failed` = null
*   While True:
    *   Try:
        *   Calcola `gwstub` = `map_paths.i_peers_gateway(lvl, _pos, null, failed)`
    *   Se riceve l'eccezione `PeersNonexistentDestinationError`:
        *   Restituisci True. Rimuovi `waiting_answer_map[m’.msg_id]`. Termina algoritmo.
    *   Try:
        *   Esegue `gwstub.forward_peer_message(m’)`.
    *   Se riceve `StubError` o `DeserializeError`:
        *   `failed` = `gwstub`.
        *   Continua con la prossima iterazione del ciclo.
    *   Esci dal ciclo.
*   Try:
    *   Sta in attesa su `waiting_answer.ch` per al massimo `timeout_instradamento`.
    *   Se `waiting_answer.missing_optional_maps`:
        *   Restituisce False. Rimuovi `waiting_answer_map[m’.msg_id]`. Termina algoritmo.
    *   Altrimenti-Se `waiting_answer.exclude_gnode` ≠ null:
        *   Restituisce False. Rimuovi `waiting_answer_map[m’.msg_id]`. Termina algoritmo.
    *   Altrimenti-Se `waiting_answer.non_participant_gnode` ≠ null:
        *   Significa che abbiamo ricevuto notizia di un gnodo non partecipante.
        *   `waiting_answer.non_participant_gnode` è un PeerTupleGNode che rappresenta un g-nodo `h` dentro il mio g-nodo di livello `top`.
        *   Se è visibile nella mia mappa, cioè se `(lvl,top)` non partecipa:
            *   Restituisci True. Rimuovi `waiting_answer_map[m’.msg_id]`. Termina algoritmo.
        *   Altrimenti:
            *   Restituisce False. Rimuovi `waiting_answer_map[m’.msg_id]`. Termina algoritmo.
    *   Altrimenti-Se `waiting_answer.response` ≠ null:
        *   Significa che abbiamo ricevuto il contatto e che `lvl,_pos` partecipa.
        *   Restituisce False. Rimuovi `waiting_answer_map[m’.msg_id]`. Termina algoritmo.
    *   Altrimenti:
        *   Significa che abbiamo ricevuto un nuovo valore in `waiting_answer.min_target`.
        *   Restituisce False. Rimuovi `waiting_answer_map[m’.msg_id]`. Termina algoritmo.
*   Se riceve l'eccezione `TimeoutError`:
    *   Dobbiamo trattare `waiting_answer.min_target` come da escludere.
    *   Restituisce False. Rimuovi `waiting_answer_map[m’.msg_id]`. Termina algoritmo.

## <a name="Divulgazione_partecipazione"></a>Algoritmo di divulgazione della partecipazione

**publish_my_participation**

Firma: `void publish_my_participation(p_id)`

*   `gn` = `make_tuple_gnode(new HCoord(0, pos[0]), levels)`. La tupla *n<sub>0</sub>·n<sub>1</sub>·...·n<sub>levels-1</sub>*,
    che identifica il nodo corrente nella rete.
*   `tempo_attesa` = 300 secondi.
*   `iterazioni` = 5.
*   While True (per sempre):
    *   Se `iterazioni` > 0:
        *   Decrementa `iterazioni` di 1.
    *   Altrimenti:
        *   `tempo_attesa` = 1 giorno + random(1, 24 \* 60 \* 60) secondi.
    *   Prepara un IPeersMissingArcHandler `missing_handler` che in caso di invocazione esegua:
        *   Calcola `tcp_stub` = `neighbors_factory.i_peers_get_tcp(missing_arc)`.
        *   Try:
            *   `tcp_stub.set_participant(p_id, gn)`.
        *   Se riceve `StubError` o `DeserializeError`:
            *   Ignora.
    *   Calcola `br_stub` = `neighbors_factory.i_peers_get_broadcast(missing_handler)`.
    *   Try:
        *   `br_stub.set_participant(p_id, gn)`.
    *   Se riceve `StubError` o `DeserializeError`:
        *   Ignora.
    *   Aspetta `tempo_attesa`.

**set_participant**

Firma: `void set_participant(int p_id, PeerTupleGNode gn)`

*   E' già stata istanziata  `lista_recenti` un ArrayList di HCoord.
*   Se  `services.has_key(p_id)` **AND NOT** `services[p_id].p_is_optional`:
    *   Ignora il messaggio. Algoritmo termina.
*   int `case`, HCoord `ret`.
*   Calcola `convert_tuple_gnode(gn, out case, out ret)`.
*   Se `case` = 1:
    *   Cioè `gn` rappresenta un mio g-nodo.
    *   Ignora il messaggio. Algoritmo termina.
*   Altrimenti:
    *   Cioè `gn` rappresenta un g-nodo a cui io non appartengo, ed ho già calcolato in `ret` il g-nodo visibile nella mia topologia in cui `gn` si trova.
    *   Se `ret` ∈ `lista_recenti`:
        *   Ignora il messaggio. Algoritmo termina.
    *   Altrimenti:
        *   `lista_recenti.add(ret)`.
        *   Se **NOT** `participant_maps.has_key(p_id)`:
            *   `participant_maps[p_id]` = new ParticipantMap().
        *   `participant_maps[p_id].participant_list.add(ret)`.
        *   `ret_gn` = `make_tuple_gnode(ret, levels)`
        *   Prepara un IPeersMissingArcHandler `missing_handler` che in caso di invocazione esegua:
            *   Calcola `tcp_stub` = `neighbors_factory.i_peers_get_tcp(missing_arc)`.
            *   Try:
                *   `tcp_stub.set_participant(p_id, ret_gn)`.
            *   Se riceve `StubError` o `DeserializeError`:
                *   Ignora.
        *   Calcola `br_stub` = `neighbors_factory.i_peers_get_broadcast(missing_handler)`.
        *   Try:
            *   `br_stub.set_participant(p_id, ret_gn)`.
        *   Se riceve `StubError` o `DeserializeError`:
            *   Ignora.
        *   Svolgi quanto segue in una nuova tasklet portando dietro `ret`:
            *   Aspetta 60 secondi.
            *   `lista_recenti.remove(ret)`.

## <a name="Mantenimento_database"></a>Algoritmi per il mantenimento di un database distribuito

### <a name="Persistenza_dati"></a>Per migliorare la persistenza dei dati

**begin_replica**

Firma: `bool begin_replica(q, p_id, x̄, r, timeout_exec, out IPeersResponse? resp, out IPeersContinuation cont)`

*   Gli argomenti sono:
    *   int `q`: il numero delle repliche richieste,
    *   int `p_id`,
    *   PeerTupleNode `x̄`: la tupla dell'hash-node della chiave del record, cioè *h<sub>p</sub>(k)*,
    *   IPeersRequest `r`: la richiesta di replicare la coppia k,val ,
    *   int `timeout_exec`,
    *   `resp` viene valorizzato con la risposta o null;
    *   `cont` è un oggetto di cui all'esterno si sa solo che implementa l'interfaccia vuota IPeersContinuation.
*   `lista_repliche` = new List di PeerTupleNode.
*   `exclude_tuple_list` = new `PeerTupleGNodeContainer(x̄.tuple.size)`.
*   `cont` = `{q, p_id, x̄, r, timeout_exec, lista_repliche, exclude_tuple_list}`.
*   Return `next_replica(cont, out resp)`.

**next_replica**

Firma: `bool next_replica(IPeersContinuation cont, out IPeersResponse? resp)`

*   `resp` = null.
*   Se `cont.lista_repliche.size` ≥ `cont.q`:
    *   Return False.
*   PeerTupleNode `respondant`;
*   `ret` = `contact_peer(cont.p_id, cont.x̄, cont.r, cont.timeout_exec, True, out respondant, cont.exclude_tuple_list)`.
*   Se si riceve l'eccezione `PeersNoParticipantsInNetworkError` o `PeersDatabaseError`:
    *   Return False. L'eccezione PeersDatabaseError va vista come "OUT-OF-MEMORY" essendo la replica una operazione di sovrascrittura o inserimento.
*   `resp` = `ret`.
*   aggiungi `respondant` a `cont.lista_repliche`.
*   aggiungi `respondant` a `cont.exclude_tuple_list`.
*   Return `cont.lista_repliche.size` < `cont.q`.

### <a name="Coerenza_dati"></a>Per garantire la coerenza dei dati

Per garantire la coerenza dei dati e gestire l'esaustività e le limitazioni di memoria dei singoli nodi, gli algoritmi sviluppati sono illustrati nei seguenti documenti:

*   [Database con un Time To Live](DatabaseTTL.md)
*   [Database a chiavi fisse](DatabaseFixedKeys.md)

