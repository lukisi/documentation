# Modulo PeerServices - Algoritmi di instradamento

1.  [Algoritmo di instradamento completo](#Instradamento)
1.  [Messaggi di ritorno](#Ritorno)

## <a name="Instradamento"></a>Algoritmo di instradamento completo

**contact_peer**

Firma: `IPeersResponse contact_peer(p_id, x̄, request, timeout_exec, exclude_myself, respondant, exclude_tuple_list=null) throws PeersNoParticipantsInNetworkError, PeersDatabaseError`

*   Gli argomenti passati sono:
    *   `int p_id`,
    *   `PeerTupleNode x̄`,
    *   `IPeersRequest request`,
    *   `int timeout_exec`,
    *   `bool exclude_myself`,
    *   `out PeerTupleNode respondant`,
    *   `PeerTupleGNodeContainer? exclude_tuple_list`=null.
*   `refuse_messages` = "".
*   `respondant` = null.
*   `target_levels` = `x̄.tuple.size`.
*   `exclude_gnode_list` = new ArrayList di HCoord.
*   bool `opzionale` = False.
*   Se `services.has_key(p_id)`:
    *   `opzionale` = `services[p_id].p_is_optional`.
    *   Se **NOT** `services[m’.p_id].is_ready(target_levels)`:
        *   `exclude_myself` = True.
*   Altrimenti:
    *   `opzionale` = True.
*   Mette in `exclude_gnode_list` tutti i g-nodi risultanti da `get_non_participant_gnodes(p_id)`.
*   Se `exclude_myself`:
    *   Mette `HCoord(0, pos[0])` in `exclude_gnode_list`.
*   Se `exclude_tuple_list` = null:
    *   `exclude_tuple_list` = `new PeerTupleGNodeContainer(target_levels)`.
*   `assert(exclude_tuple_list.top == target_levels)`.
*   Per ogni PeerTupleGNode `gn` in `exclude_tuple_list`:
    *   int `case`, HCoord `ret`.
    *   Calcola `convert_tuple_gnode(gn, out case, out ret)`.
    *   Se `case` = 2:
        *   Cioè `gn` rappresenta un g-nodo visibile nella mia topologia.
        *   Metti `ret` in `exclude_gnode_list`.
*   `non_participant_tuple_list` = `new PeerTupleGNodeContainer(target_levels)`.
*   IPeersResponse? `response` = null.
*   While True (ciclo 1):
    *   `x` = `approximate(x̄, exclude_list=exclude_gnode_list)`. Restituisce un HCoord o null.
    *   Se `x` = null:
        *   Se `refuse_messages` ≠ "":
            *   Lancia eccezione `new PeersDatabaseError(refuse_messages)`.
        *   Lancia eccezione `PeersNoParticipantsInNetworkError`. L'algoritmo termina.
    *   Se `x.lvl` = 0 **AND** `x.pos` = `pos[0]`:
        *   Try:
            *   `response` = `services[p_id].exec(request, new ArrayList<int>())`. In questo caso va riprodotta la
                copiatura degli oggetti IPeersRequest e IPeersResponse, che avverrebbe in modo automatico al momento
                della trasmissione in rete nei metodi remoti `get_request` e `set_response`.
        *   Se riceve `PeersRedoFromStartError`:
            *   Return `contact_peer(...)`. Riavvia l'algoritmo.
        *   Se riceve `PeersRefuseExecutionError e`:
            *   `refuse_messages` += `e.message`.
            *   Se `refuse_messages.length` > 500:
                *   Solo l'ultima parte.
                *   `refuse_messages` = `refuse_messages.substr(refuse_messages.length-500)`.
            *   Mette `HCoord(0, pos[0])` in `exclude_gnode_list`.
            *   Continua con la prossima iterazione del ciclo 1.
        *   Calcola `respondant` = istanza di PeerTupleNode che rappresenta `HCoord(0, pos[0])` con
            `top` = `target_levels`, cioè il nodo stesso.
        *   Restituisce `response`. Termina l'algoritmo.
    *   `m’` = `new PeerMessageForwarder`.
    *   `m’.inside_level` = `target_levels`.
    *   `m’.n` = la tupla *n<sub>0</sub>·n<sub>1</sub>·...·n<sub>j</sub>* dove *j* = `x.lvl`.
    *   `m’.x̄` = la tupla *x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>j-1</sub>*. E' null se *j* = 0.
    *   `m’.lvl` = `x.lvl`.
    *   `m’.pos` = `x.pos`.
    *   `m’.p_id` = `p_id`.
    *   `m’.msg_id` = un identificativo generato a caso per questo messaggio.
    *   Per ogni PeerTupleGNode `t` in `exclude_tuple_list`:
        *   Scopriamo se `t` è interno a `x`:
        *   int `case`, HCoord `ret`.
        *   Calcola `convert_tuple_gnode(t, out case, out ret)`.
        *   Se `case` = 3:
            *   Cioè `t` rappresenta un g-nodo non visibile nella mia topologia interno a `ret`.
            *   Se `ret.equals(x)`:
                *   `𝜀` = `t.top` - `t.tuple.size`
                *   `t’` = `new PeerTupleGNode(t.tuple.slice(0, j-𝜀), j)`.
                *   Aggiungi `t’` a `m’.exclude_tuple_list`.
    *   Per ogni PeerTupleGNode `t` in `non_participant_tuple_list`:
        *   Se `visible_by_someone_inside_my_gnode(t, j+1)`:
            *   Aggiungi `t` a `m’.non_participant_tuple_list`.
    *   Calcola `timeout_instradamento` = *f* ( `map_paths.i_peers_get_nodes_in_my_group(x.lvl + 1)` ).
    *   Prepara `waiting_answer` = `new WaitingAnswer`(`request=request`, `min_target=x` come PeerTupleGNode con
        `top` = `x.lvl+1`).
    *   `waiting_answer_map[m’.msg_id]` = `waiting_answer`.
    *   `IPeersManagerStub gwstub`
    *   `IPeersManagerStub? failed` = null
    *   `bool necessita_ricalcolo` = False
    *   While True:
        *   Try:
            *   Calcola `gwstub` = `map_paths.i_peers_gateway(x.lvl, x.pos, null, failed)`
        *   Se riceve l'eccezione `PeersNonexistentDestinationError`:
            *   `necessita_ricalcolo` = True.
            *   Esci dal ciclo.
        *   Try:
            *   Esegue `gwstub.forward_peer_message(m’)`.
        *   Se riceve l'eccezione `StubError` (non può ricevere un DeserializeError perché lo stub non attende la processazione):
            *   `failed` = `gwstub`.
            *   Continua con la prossima iterazione del ciclo.
        *   Esci dal ciclo.
    *   Se `necessita_ricalcolo`:
        *   Rimuovi `waiting_answer_map[m’.msg_id]`.
        *   Aspetta alcuni istanti.
        *   Continua con la prossima iterazione del ciclo 1.
    *   `tempo_attesa` = `timeout_instradamento`.
    *   While True (ciclo 2):
        *   Try:
            *   Sta in attesa su `waiting_answer.ch` per max `tempo_attesa`.
            *   Se `waiting_answer.exclude_gnode` ≠ null:
                *   Significa che abbiamo ricevuto notizia di un gnodo da escludere.
                *   `waiting_answer.exclude_gnode` è un PeerTupleGNode che rappresenta un g-nodo `h` dentro il mio
                    g-nodo di livello `top`.
                *   Crea `t`, una istanza di PeerTupleGNode con `top=target_levels` che rappresenta il g-nodo `h`.
                *   Se è visibile nella mia mappa:
                    *   Crea `g`, una istanza di HCoord che rappresenta il g-nodo `h`.
                    *   Mette `g` in `exclude_gnode_list`.
                *   Mette `t` in `exclude_tuple_list`.
                *   Rimuovi `waiting_answer_map[m’.msg_id]`.
                *   Esci dal ciclo 2.
            *   Altrimenti-Se `waiting_answer.non_participant_gnode` ≠ null:
                *   Significa che abbiamo ricevuto notizia di un gnodo non partecipante.
                *   `waiting_answer.non_participant_gnode` è un PeerTupleGNode che rappresenta un g-nodo `h` dentro il mio
                    g-nodo di livello `top`.
                *   Crea `t`, una istanza di PeerTupleGNode con `top=target_levels` che rappresenta il g-nodo `h`.
                *   Se è visibile nella mia mappa:
                    *   Crea `g`, una istanza di HCoord che rappresenta il g-nodo `h`.
                    *   Se `opzionale`:
                        *   Se `participant_maps.has_key(p_id)`:
                            *   `map` = `participant_maps[p_id]`.
                            *   Toglie `g` da `map.participant_list`.
                    *   Mette `g` in `exclude_gnode_list`.
                *   Mette `t` in `exclude_tuple_list`.
                *   Mette `t` in `non_participant_tuple_list`.
                *   Rimuovi `waiting_answer_map[m’.msg_id]`.
                *   Esci dal ciclo 2.
            *   Altrimenti-Se `respondant` = null **e** `waiting_answer.respondant_node` ≠ null:
                *   Significa che abbiamo consegnato il messaggio.
                *   `respondant` = `waiting_answer.respondant_node` come tupla fino al livello 0 nel g-nodo di ricerca.
                *   `tempo_attesa` = `timeout_exec`.
            *   Altrimenti-Se `waiting_answer.response` ≠ null:
                *   Significa che abbiamo ricevuto la risposta.
                *   `response` = `waiting_answer.response`.
                *   Rimuovi `waiting_answer_map[m’.msg_id]`.
                *   Esci dal ciclo 2. Poi uscirai dal ciclo 1.
            *   Altrimenti-Se `respondant` ≠ null **e** `waiting_answer.refuse_message` ≠ null:
                *   Significa che l'attuale respondant ha rifiutato di elaborare la richiesta.
                *   `refuse_messages` += `waiting_answer.refuse_message`.
                *   Se `refuse_messages.length` > 500:
                    *   Solo l'ultima parte.
                    *   `refuse_messages` = `refuse_messages.substr(refuse_messages.length-500)`.
                *   Crea `t`, una istanza di PeerTupleGNode con `top=target_levels` che rappresenta il nodo respondant.
                *   Se è visibile nella mia mappa:
                    *   Crea `g`, una istanza di HCoord che rappresenta il nodo respondant.
                    *   Mette `g` in `exclude_gnode_list`.
                *   Mette `t` in `exclude_tuple_list`.
                *   `respondant` = null.
                *   Rimuovi `waiting_answer_map[m’.msg_id]`.
                *   Esci dal ciclo 2.
            *   Altrimenti-Se `respondant` ≠ null **e** `waiting_answer.redo_from_start`:
                *   Significa che l'attuale respondant ha dato istruzione di riavviare il calcolo distribuito di *H<sub>t</sub>*.
                *   Return `contact_peer(...)`. Riavvia l'algoritmo.
            *   Altrimenti:
                *   Significa che abbiamo ricevuto un nuovo valore in `waiting_answer.min_target`.
                *   Nessuna operazione.
        *   Se riceve l'eccezione `TimeoutError`:
            *   Dobbiamo trattare `waiting_answer.min_target` come da escludere.
            *   `waiting_answer.min_target` è un PeerTupleGNode che rappresenta un g-nodo `h` dentro il mio g-nodo di livello `top`.
            *   Crea `t`, una istanza di PeerTupleGNode con `top=target_levels` che rappresenta il g-nodo `h`.
            *   Se è visibile nella mia mappa:
                *   Crea `g`, una istanza di HCoord che rappresenta il g-nodo `h`.
                *   Mette `g` in `exclude_gnode_list`.
            *   Mette `t` in `exclude_tuple_list`.
            *   `respondant` = null.
            *   Rimuovi `waiting_answer_map[m’.msg_id]`.
            *   Esci dal ciclo 2.
    *   Se `response` ≠ null:
        *   Esci dal ciclo 1.
*   Uscito dal ciclo 1.
*   Restituisce `response`.

**forward_peer_message**

Firma: `void forward_peer_message(m’)`

*   Recupera `caller`, cioè le informazioni sul chiamante del metodo remoto.
*   bool `opzionale` = False.
*   bool `exclude_myself` = False.
*   Se `services.has_key(m’.p_id)`:
    *   `opzionale` = `services[m’.p_id].p_is_optional`.
    *   `exclude_myself` = **NOT** `services[m’.p_id].is_ready(m’.inside_level)`.
*   Altrimenti:
    *   `opzionale` = True.
*   Se `pos[m’.lvl] = m’.pos`:
    *   Facciamo riferimento agli algoritmi [helper](MetodiHelper.md).
    *   Se **NOT** `my_gnode_participates(m’.p_id, m’.lvl)`:
        *   `nstub` = `back_stub_factory.i_peers_get_tcp_inside(m’.n.tuple)`.
        *   Calcola `gn` istanza di PeerTupleGNode che rappresenta `HCoord(m’.lvl, m’.pos)` con `top` = `m’.n.tuple.size`.
        *   Try:
            *   Esegue `nstub.set_non_participant(m’.msg_id, gn)`.
        *   Se riceve `StubError` o `DeserializeError`:
            *   Ignora.
    *   Altrimenti:
        *   `exclude_gnode_list` = new ArrayList di HCoord.
        *   Metti in `exclude_gnode_list` tutti i g-nodi risultanti da `get_non_participant_gnodes(p_id)`.
        *   Se `exclude_myself`:
            *   Mette `HCoord(0, pos[0])` in `exclude_gnode_list`.
        *   Per ogni PeerTupleGNode `gn` in `m’.exclude_tuple_list`:
            *   int `case`, HCoord `ret`.
            *   Calcola `convert_tuple_gnode(gn, out case, out ret)`.
            *   Se `case` = 1:
                *   Cioè `gn` rappresenta un mio g-nodo di livello `ret.lvl`:
                *   Metti in `exclude_gnode_list` tutti i g-nodi risultanti da `get_all_gnodes_up_to_level(ret.lvl)`.
            *   Se `case` = 2:
                *   Cioè `gn` rappresenta un g-nodo visibile nella mia topologia:
                *   Metti `ret` in `exclude_gnode_list`.
        *   bool `consegnato` = False.
        *   While **NOT** `consegnato`:
            *   `x` = `approximate(m’.x̄, exclude_list=exclude_gnode_list)`. Restituisce un HCoord o null.
            *   Se `x` = null:
                *   `nstub` = `back_stub_factory.i_peers_get_tcp_inside(m’.n.tuple)`.
                *   Calcola `gn` istanza di PeerTupleGNode che rappresenta `HCoord(m’.lvl, m’.pos)` con
                    `top` = `m’.n.tuple.size`.
                *   Try:
                    *   Esegue `nstub.set_failure(m’.msg_id, gn)`.
                *   Se riceve `StubError` o `DeserializeError`:
                    *   Ignora.
                *   Esci dal ciclo. Poi proseguirà con l'elaborazione delle informazioni di non partecipazione.
            *   Altrimenti-Se `x.lvl` = 0 **AND** `x.pos = pos[0]`:
                *   `nstub` = `back_stub_factory.i_peers_get_tcp_inside(m’.n.tuple)`.
                *   Calcola `tuple_respondant` istanza di PeerTupleNode che rappresenta `HCoord(0, pos[0])` con
                    `top` = `m’.n.tuple.size`.
                *   Try:
                    *   IPeersRequest `request` = `nstub.get_request(m’.msg_id, tuple_respondant)`.
                    *   Try:
                        *   IPeersResponse `resp` = `services[m’.p_id].exec(request, m’.n.tuple)`.
                        *   Esegue `nstub.set_response(m’.msg_id, resp, tuple_respondant)`.
                    *   Se riceve `PeersRedoFromStartError` `e`:
                        *   Try:
                            *   Esegue `nstub.set_redo_from_start(m’.msg_id, tuple_respondant)`.
                        *   Se riceve `StubError` o `DeserializeError`:
                            *   Ignora.
                    *   Se riceve `PeersRefuseExecutionError` `e`:
                        *   Try:
                            *   `err_msg` = `e.message`.
                            *   Esegue `nstub.set_refuse_message(m’.msg_id, err_msg, tuple_respondant)`.
                        *   Se riceve `StubError` o `DeserializeError`:
                            *   Ignora.
                *   Se riceve `PeersUnknownMessageError` o `PeersInvalidRequest`:
                    *   Ignora.
                *   Se riceve `StubError` o `DeserializeError`:
                    *   Ignora.
                *   Esci dal ciclo. Poi proseguirà con l'elaborazione delle informazioni di non partecipazione.
            *   Altrimenti:
                *   `m’’` = copia di `m’`.
                *   `m’’.lvl` = `x.lvl`.
                *   `m’’.pos` = `x.pos`.
                *   `m’’.x̄` = la tupla *x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>k-1</sub>* dove *k* = `x.lvl`. E' null se *k* = 0.
                *   Per ogni PeerTupleGNode `t` in `m’.exclude_tuple_list`:
                    *   Scopriamo se `t` è interno a `x`:
                    *   int `case`, HCoord `ret`.
                    *   Calcola `convert_tuple_gnode(t, out case, out ret)`.
                    *   Se `case` = 3:
                        *   Cioè `t` rappresenta un g-nodo non visibile nella mia topologia interno a `ret`.
                        *   Se `ret.equals(x)`:
                            *   `𝜀` = `t.top` - `t.tuple.size`.
                            *   `t’` = `new PeerTupleGNode(t.tuple.slice(0, k-𝜀), k)`.
                            *   Aggiunge `t’` a `m’’.exclude_tuple_list`.
                *   Per ogni PeerTupleGNode `t` in `m’.non_participant_tuple_list`:
                    *   Se `visible_by_someone_inside_my_gnode(t, k+1)`:
                        *   Aggiunge `t` a `m’’.non_participant_tuple_list`.
                *   IPeersManagerStub `gwstub`
                *   IPeersManagerStub? `failed` = null
                *   While True:
                    *   Try:
                        *   Calcola `gwstub` = `map_paths.i_peers_gateway(m’’.lvl, m’’.pos, caller, failed)`
                    *   Se riceve l'eccezione `PeersNonexistentDestinationError`:
                        *   Aspetta alcuni istanti.
                        *   Esci dal ciclo. Ripeterà il calcolo di `approximate`.
                    *   Try:
                        *   Esegue `gwstub.forward_peer_message(m’’)`.
                    *   Se riceve l'eccezione `StubError` (non può ricevere un `DeserializeError` perché lo stub non
                        attende la processazione):
                        *   `failed` = `gwstub`.
                        *   Continua con la prossima iterazione del ciclo.
                    *   `consegnato` = True.
                    *   `nstub` = `back_stub_factory.i_peers_get_tcp_inside(m’.n.tuple)`.
                    *   Calcola `gn` istanza di PeerTupleGNode che rappresenta `HCoord(x.lvl, x.pos)` con `top` = `m’.n.tuple.size`.
                    *   Try:
                        *   Esegue `nstub.set_next_destination(m’.msg_id, gn)`.
                    *   Se riceve `StubError` o `DeserializeError`:
                        *   Ignora.
                    *   Esci dal ciclo. Poi proseguirà con l'elaborazione delle informazioni di non partecipazione.
*   Altrimenti:
    *   IPeersManagerStub `gwstub`
    *   IPeersManagerStub? `failed` = null
    *   While True:
        *   Try:
            *   Calcola `gwstub` = `map_paths.i_peers_gateway(m’.lvl, m’.pos, caller, failed)`
        *   Se riceve l'eccezione `PeersNonexistentDestinationError`:
            *   Rinuncia all'instradamento. Esci dal ciclo.
        *   Try:
            *   Esegue `gwstub.forward_peer_message(m’)`.
        *   Se riceve l'eccezione `StubError` (non può ricevere un `DeserializeError` perché lo stub non attende la processazione):
            *   `failed` = `gwstub`.
            *   Continua con la prossima iterazione del ciclo.
        *   Esci dal ciclo.
*   Se `opzionale` **AND** `participant_maps.has_key(m’.p_id)`:
    *   Per ogni TupleGNode `t` in `m’.non_participant_tuple_list`:
        *   Se `t` rappresenta un g-nodo visibile nella nostra mappa:
            *   Calcola `g` = istanza di HCoord rappresentante lo stesso g-nodo di `t`.
            *   Se `g` ∈ `participant_maps[m’.p_id].participant_list`:
                *   Svolgi quanto segue in una nuova tasklet portando dietro `p_id=m’.p_id`, `lvl=g.lvl`, `pos=g.pos`:
                    *   Facciamo riferimento agli algoritmi [complementari](AlgoritmiComplementari.md).
                    *   Se `check_non_participation(p_id, lvl, pos)`:
                        *   Se `participant_maps.has_key(p_id)`:
                            *   Rimuove `HCoord(lvl,pos)` da `participant_maps[p_id].participant_list`.

**approximate**

Firma: `HCoord? approximate(x̄, exclude_list)`

*   Gli argomenti sono:
    *   PeerTupleNode? `x̄`,
    *   List di HCoord `exclude_list`.
*   Se `x̄` = null:
    *   La destinazione sono io o nessun altro.
    *   `x` = `new HCoord(0,pos[0])`. Cioè `x` rappresenta me.
    *   Se `x` ∉ `exclude_list`:
        *   Return `x`.
    *   Altrimenti:
        *   Return null.
*   `valid_levels` = `x̄.tuple.size`. Specificando una tupla con un numero di posizioni inferiore al numero totale dei
    livelli della rete si richiede a questa funzione (e di coseguenza al resto dell'algoritmo di instradamento verso
    l'hash-node) di restringere la ricerca all'interno di un certo g-nodo.
*   HCoord? `ret` = null.
*   `min_distance` = -1.
*   Per `l` che va da 0 a `valid_levels-1`: Per `p` che va da 0 a `gsizes[l]-1`: Se `pos[l]` ≠ `p`: Se `map_paths.i_peers_exists(l, p)`:
    *   `x` = `new HCoord(l,p)`. Cioè `x` rappresenta un g-nodo a cui io non appartengo, visibile nella mia topologia,
        interno al g-nodo in cui la ricerca è eventualmente circoscritta, esistente nella rete.
    *   Se `x` ∉ `exclude_list`:
        *   PeerTupleNode `tuple_x` = `make_tuple_node(x, valid_levels)`.
        *   `distance` = `dist(x̄, tuple_x)`.
        *   Se `min_distance` = -1 **OR** `distance` < `min_distance`:
            *   `ret` = `x`.
            *   `min_distance` = `distance`.
*   `x` = `new HCoord(0,pos[0])`. Cioè `x` rappresenta me.
*   Se `x` ∉ `exclude_list`:
    *   PeerTupleNode `tuple_x` = `make_tuple_node(x, valid_levels)`.
    *   `distance` = `dist(x̄, tuple_x)`.
    *   Se `min_distance` = -1 **OR** `distance` < `min_distance`:
        *   `ret` = `x`.
        *   `min_distance` = `distance`.
*   Return `ret`.

**dist**

Firma: `int dist(x̄, x)`

*   Gli argomenti sono:
    *   PeerTupleNode `x̄`,
    *   PeerTupleNode `x`.
*   `assert(x̄.tuple.size == x.tuple.size)`.
*   `distance` = 0.
*   Per `j` da `x.tuple.size-1` a 0:
    *   `distance` *= `gsizes[j]`.
    *   `distance` += `x.tuple[j]` - `x̄.tuple[j]`.
    *   Se `x̄.tuple[j]` > `x.tuple[j]`:
        *   `distance` += `gsizes[j]`.
*   Return `distance`.

## <a name="Ritorno"></a>Messaggi di ritorno

**set_next_destination**

Firma: `void set_next_destination(int msg_id, IPeerTupleGNode _tuple)`

*   Si verifica che il `msg_id` sia presente come chiave in `wainting_answer_map`. Altrimenti si ignora il messaggio.
*   Si recupera dalla mappa il relativo `WaitingAnswer wa`.
*   Si verifica che l'oggetto `_tuple` sia una valida istanza di PeerTupleGNode e si assegna a `tuple`. Altrimenti si ignora il messaggio.
*   Si verifica che `tuple` rientri nel g-nodo di ricerca associato a `wa`, come si può rilevare da `wa.min_target.top`. Altrimenti si ignora il messaggio.
*   Si verifica che `tuple` rappresenti un g-nodo di livello più basso rispetto al precedente `wa.min_target`. Altrimenti si ignora il messaggio.
*   `wa.min_target` = `tuple`.
*   Si segnala l'evento sul canale con `wa.ch.send_async(0)`.

**set_non_participant**

Firma: `void set_non_participant(int msg_id, IPeerTupleGNode _tuple)`

*   Si verifica che il `msg_id` sia presente come chiave in `wainting_answer_map`. Altrimenti si ignora il messaggio.
*   Si recupera dalla mappa il relativo `WaitingAnswer wa`.
*   Si verifica che l'oggetto `_tuple` sia una valida istanza di PeerTupleGNode e si assegna a `tuple`. Altrimenti si ignora il messaggio.
*   Si verifica che `tuple` rientri nel g-nodo di ricerca associato a `wa`, come si può rilevare da `wa.min_target.top`. Altrimenti si ignora il messaggio.
*   Si verifica che `tuple` rappresenti un g-nodo di livello più basso o uguale rispetto al precedente `wa.min_target`. Altrimenti si ignora il messaggio.
*   `wa.non_participant_gnode` = `tuple`.
*   Si segnala l'evento sul canale con `wa.ch.send_async(0)`.

**set_failure**

Firma: `void set_failure(int msg_id, IPeerTupleGNode _tuple)`

*   Si verifica che il `msg_id` sia presente come chiave in `wainting_answer_map`. Altrimenti si ignora il messaggio.
*   Si recupera dalla mappa il relativo `WaitingAnswer wa`.
*   Si verifica che l'oggetto `_tuple` sia una valida istanza di PeerTupleGNode e si assegna a `tuple`. Altrimenti si ignora il messaggio.
*   Si verifica che `tuple` rientri nel g-nodo di ricerca associato a `wa`, come si può rilevare da `wa.min_target.top`. Altrimenti si ignora il messaggio.
*   Si verifica che `tuple` rappresenti un g-nodo di livello più basso o uguale rispetto al precedente `wa.min_target`. Altrimenti si ignora il messaggio.
*   `wa.exclude_gnode` = `tuple`.
*   Si segnala l'evento sul canale con `wa.ch.send_async(0)`.

**get_request**

Firma: `IPeersRequest get_request(int msg_id, IPeerTupleNode _respondant) throws PeersUnknownMessageError, PeersInvalidRequest`

*   Si verifica che il `msg_id` sia presente come chiave in `wainting_answer_map`. Altrimenti si ignora il messaggio, rilanciando l'eccezione PeersUnknownMessageError.
*   Si recupera dalla mappa il relativo `WaitingAnswer wa`.
*   Si verifica che l'oggetto `_respondant` sia una valida istanza di PeerTupleNode e si assegna a `respondant`. Altrimenti si ignora il messaggio, rilanciando l'eccezione PeersInvalidRequest.
*   Si verifica che `respondant` rientri nel g-nodo di ricerca associato a `wa`, come si può rilevare da `wa.min_target.top`. Altrimenti si ignora il messaggio, rilanciando l'eccezione PeersInvalidRequest.
*   `wa.respondant_node` = `respondant`.
*   Si segnala l'evento sul canale con `wa.ch.send_async(0)`.
*   Se `wa.request` = null:
    *   Si tratta di una finta richiesta, vedi `check_non_participation` richiamato dal `forward_peer_message`.
    *   Lancia l'eccezione `PeersUnknownMessageError`.
*   Altrimenti:
    *   Return `wa.request`.

**set_response**

Firma: `void set_response(int msg_id, IPeersResponse response, IPeerTupleNode _respondant)`

*   Si verifica che il `msg_id` sia presente come chiave in `wainting_answer_map`. Altrimenti si ignora il messaggio.
*   Si recupera dalla mappa il relativo `WaitingAnswer wa`.
*   Si verifica che l'oggetto `_respondant` sia una valida istanza di PeerTupleNode e si assegna a `respondant`. Altrimenti si ignora il messaggio.
*   Si verifica che `wa.respondant_node` = `respondant`. Altrimenti si ignora il messaggio.
*   `wa.response` = `response`.
*   Si segnala l'evento sul canale con `wa.ch.send_async(0)`.

**set_refuse_message**

Firma: `void set_refuse_message(int msg_id, string refuse_message, IPeerTupleNode _respondant)`

*   Si verifica che il `msg_id` sia presente come chiave in `wainting_answer_map`. Altrimenti si ignora il messaggio.
*   Si recupera dalla mappa il relativo `WaitingAnswer wa`.
*   Si verifica che l'oggetto `_respondant` sia una valida istanza di PeerTupleNode e si assegna a `respondant`. Altrimenti si ignora il messaggio.
*   Si verifica che `wa.respondant_node` = `respondant`. Altrimenti si ignora il messaggio.
*   `wa.refuse_message` = `refuse_message`.
*   Si segnala l'evento sul canale con `wa.ch.send_async(0)`.

**set_redo_from_start**

Firma: `void set_redo_from_start(int msg_id, IPeerTupleNode _respondant)`

*   Si verifica che il `msg_id` sia presente come chiave in `wainting_answer_map`. Altrimenti si ignora il messaggio.
*   Si recupera dalla mappa il relativo `WaitingAnswer wa`.
*   Si verifica che l'oggetto `_respondant` sia una valida istanza di PeerTupleNode e si assegna a `respondant`. Altrimenti si ignora il messaggio.
*   Si verifica che `wa.respondant_node` = `respondant`. Altrimenti si ignora il messaggio.
*   `wa.redo_from_start` = `True`.
*   Si segnala l'evento sul canale con `wa.ch.send_async(0)`.

