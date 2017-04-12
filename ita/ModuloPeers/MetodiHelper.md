# Modulo PeerServices - Metodi helper

1.  [my_gnode_participates](#my_gnode_participates)
1.  [get_non_participant_gnodes](#get_non_participant_gnodes)
1.  [get_all_gnodes_up_to_lvl](#get_all_gnodes_up_to_lvl)
1.  [convert_tuple_gnode](#convert_tuple_gnode)
1.  [make_tuple_gnode](#make_tuple_gnode)
1.  [make_tuple_node](#make_tuple_node)
1.  [visible_by_someone_inside_my_gnode](#visible_by_someone_inside_my_gnode)

## <a name="my_gnode_participates"></a>my_gnode_participates

Questo metodo dice se il mio g-nodo di livello *lvl* partecipa attivamente al servizio con identificativo `p_id`.

Firma: `bool my_gnode_participates(int p_id, int lvl)`

*   Se `services.has_key(p_id)`:
    *   Return True.
*   Se **not** `participant_maps.has_key(p_id)`:
    *   Return False.
*   `map` = `participant_maps[p_id]`.
*   Per ogni HCoord `g` in `map.participant_list`:
    *   Se `g.lvl`﹤`lvl`:
        *   Return True.
*   Return False.

## <a name="get_non_participant_gnodes"></a>get_non_participant_gnodes

Questo metodo restituisce una lista con tutte le istanze di HCoord che rappresentano i g-nodi a me visibili
nella topologia della rete che, stando alle mie conoscenze, non partecipano al servizio con identificativo `p_id`.

Firma: `List<HCoord> get_non_participant_gnodes(int p_id, target_levels)`

*   `ret` =  `new ArrayList<HCoord>()`.
*   bool `opzionale` = False.
*   Se `services.has_key(p_id)`:
    *   `opzionale` = `services[p_id].p_is_optional`.
*   Altrimenti:
    *   `opzionale` = True.
*   Se `opzionale`:
    *   `map` = null.
    *   Se `(participant_maps.has_key(p_id))`:
        *   `map` = `participant_maps[p_id]`.
    *   Per ogni HCoord `lp` in `get_all_gnodes_up_to_lvl(target_levels)`:
        *   Se `map` = null:
            *   Aggiungi `lp` a `ret`.
        *   Altrimenti:
            *   Se `lp` ∉ `map.participant_list`:
                *   Aggiungi `lp` a `ret`.
*   Return `ret`.

## <a name="get_all_gnodes_up_to_lvl"></a>get_all_gnodes_up_to_lvl

Questo metodo restituisce una lista con tutte le istanze di HCoord che rappresentano i g-nodi a me visibili
nella topologia della rete che sono dentro il mio g-nodo di livello *lvl*. Compresi i singoli nodi e compreso
me stesso `(0, pos[0])`.

Firma: `List<HCoord> get_all_gnodes_up_to_lvl(int target_levels)`

*   `ret` =  `new ArrayList<HCoord>()`.
*   Per `l` da 0 a `target_levels-1`:
    *   Per `p` da 0 a `gsizes[l]-1`:
        *   Se `pos[l]` ≠ `p`:
            *   Aggiungi `HCoord(l, p)` a `ret`.
*   Aggiungi `HCoord(0, pos[0])` a `ret`.
*   Return `ret`.

## <a name="convert_tuple_gnode"></a>convert_tuple_gnode

Ricordiamo che un'istanza *t* di PeerTupleGNode rappresenta un g-nodo *h* di livello *𝜀* all'interno di un
g-nodo (che chiamiamo g-nodo di riferimento) *g* di livello *l*. Il valore di *l* è memorizzato in *t.top*.
Il valore di *𝜀* si calcola come *t.top* - *t.tuple.size*.

Data un'istanza *t* di PeerTupleGNode il cui g-nodo di riferimento è uno dei g-nodi a cui il nodo corrente
appartiene, questo metodo restituisce le seguenti informazioni:

*   `int case`:
    *   Vale 1 se *t* rappresenta un g-nodo di cui il nodo corrente fa parte.
    *   Vale 2 se *t* rappresenta un g-nodo visibile nella mia topologia come istanza di HCoord.
    *   Vale 3 se *t* rappresenta un g-nodo non visibile nella mia topologia.
*   `HCoord ret`:
    *   Il g-nodo che il nodo corrente ha in comune con *h*.
    *   Nel caso 1 abbiamo `ret.lvl = 𝜀`. Inoltre `pos[ret.lvl] = ret.pos`.
    *   Nel caso 2 abbiamo `ret.lvl = 𝜀`. Inoltre `pos[ret.lvl] ≠ ret.pos`.
    *   Nel caso 3 abbiamo `ret.lvl > 𝜀`.

Firma: `void convert_tuple_gnode(PeerTupleGNode t, out int case, out HCoord ret)`

*   int `lvl` = `t.top`.
*   int `i` = `t.tuple.size`.
*   `assert(i > 0)`.
*   `assert(i ≤ lvl)`.
*   `trovato` = False.
*   While True:
    *   decrementa `lvl` di 1.
    *   decrementa `i` di 1.
    *   Se `pos[lvl] ≠ t.tuple[i]`:
        *   `ret` = `HCoord(lvl, t.tuple[i])`.
        *   `trovato` = True.
        *   Se `i` = 0:
            *   `case` = 2.
        *   Altrimenti:
            *   `case` = 3.
        *   Esci dal ciclo.
    *   Se `i = 0`:
        *   `ret` = `HCoord(lvl, t.tuple[i])`.
        *   `case` = 1.
        *   Esci dal ciclo.
*   Return `case, ret`.

## <a name="make_tuple_gnode"></a>make_tuple_gnode

Questo metodo produce un'istanza di PeerTupleGNode che rappresenta *h* nel nostro g-nodo di livello *top*.

Firma: `PeerTupleGNode make_tuple_gnode(HCoord h, int top)`

*   `assert(top > h.lvl)`.
*   `tuple` = `[]`.
*   `i` = `top`.
*   While True:
    *   Decrementa `i` di 1.
    *   Se `i` = `h.lvl`:
        *   Inserisci `h.pos` in posizione 0 in `tuple`.
        *   Esci dal ciclo.
    *   Altrimenti:
        *   Inserisci `pos[i]` in posizione 0 in `tuple`.
*   Return `new PeerTupleGNode(tuple, top)`.

## <a name="make_tuple_node"></a>make_tuple_node

Questo metodo produce un'istanza di PeerTupleNode che rappresenta *h* nel nostro g-nodo di livello *top*. In
realtà *h* è un g-nodo ma il risultato deve essere un PeerTupleNode perché va usato nel calcolo di *dist*. I
valori delle posizioni inferiori a `h.lvl` non sono significativi purché rientrino nella topologia, quindi
li impostiamo a 0.

Firma: `PeerTupleNode make_tuple_node(HCoord h, int top)`

*   `assert(top > h.lvl)`.
*   `tuple` = `[]`.
*   `i` = `top`.
*   While `i` > 0:
    *   Decrementa `i` di 1.
    *   Se `i` > `h.lvl`:
        *   Inserisci `pos[i]` in posizione 0 in `tuple`.
    *   Se `i` = `h.lvl`:
        *   Inserisci `h.pos` in posizione 0 in `tuple`.
    *   Se `i`﹤`h.lvl`:
        *   Inserisci 0 in posizione 0 in `tuple`.
*   Return `new PeerTupleNode(tuple)`.

## <a name="visible_by_someone_inside_my_gnode"></a>visible_by_someone_inside_my_gnode

Dato un g-nodo rappresentato da *t*, decidere se qualcuno dei nodi dentro il mio g-nodo di livello *lvl* può
rappresentare quel g-nodo in forma di HCoord.

Firma: `bool visible_by_someone_inside_my_gnode(PeerTupleGNode t, int lvl)`

*   `𝜀` = `t.top` - `t.tuple.size`
*   `l` = `𝜀`.
*   Se `l` ≥ `lvl-1`:
    *   `l` = `l` + 1.
*   Altrimenti:
    *   `l` = `lvl`.
*   Se `t.top` ≤ `l`:
    *   Return True.
*   Calcola `h` il g-nodo di livello `l` in cui si trova `g`.
*   PeerTupleGNode `h` = `new PeerTupleGNode(t.tuple.slice(l-𝜀, t.tuple.size), t.top)`.
*   Ora scopriamo se `h` è uno dei g-nodi a cui appartiene `n`.
*   int `case`, HCoord `ret`.
*   `convert_tuple_gnode(h, out case, out ret)`.
*   Se `case` = 1:
    *   Return True.
*   Return False.

