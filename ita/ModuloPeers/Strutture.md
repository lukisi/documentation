# Modulo PeerServices - Strutture dati

1.  [PeerTupleNode](#PeerTupleNode)
1.  [PeerTupleGNode](#PeerTupleGNode)
1.  [PeerTupleGNodeContainer](#PeerTupleGNodeContainer)
1.  [PeerMessageForwarder](#PeerMessageForwarder)
1.  [WaitingAnswer](#WaitingAnswer)
1.  [PeerParticipantMap](#PeerParticipantMap)
1.  [PeerParticipantSet](#PeerParticipantSet)

## <a name="PeerTupleNode"></a>PeerTupleNode

La classe serializzabile che identifica un nodo *n* (o un indirizzo da approssimare) all'interno di un
g-nodo *g*. Deve contenere:

*   `tuple`:
    *   la lista di posizioni n<sub>0</sub>·...·n<sub>j-1</sub>, dove *j* è il livello del g-nodo *g*.

Il livello del g-nodo *g* è intrinsecamente indicato nella dimensione della tupla.

Il g-nodo *g* è sempre individuabile come vedremo sotto nei casi d'uso.

Si può testare la validità di un oggetto PeerTupleNode ricevuto dalla rete verificando che:

*   0﹤`tuple.size` ≤ `levels`
*   Per `i` da 0 a `tuple.size-1`:
    *   0 ≤ `tuple[i]`﹤`gsizes[i]`

Questa classe è usata per restituire il risultato di *h<sub>p</sub>(k)*. Di norma in questo caso rappresenta
una tupla globale, cioè un indirizzo all'interno del g-nodo che costituisce l'intera rete; quindi in questo
caso il g-nodo *g* è l'intera rete. Ma qualche specifico servizio può implementare la funzione *h<sub>p</sub>*
in modo diverso e restituire una tupla con un numero minore di elementi per indicare che la ricerca va
circoscritta ad un proprio g-nodo; in questo caso il g-nodo *g* è uno dei g-nodi a cui il nodo corrente
(che ha calcolato *h<sub>p</sub>*) appartiene.

Questa classe è usata anche per indicare, all'interno del messaggio *m’*, un indirizzo all'interno del g-nodo
che è la corrente destinazione del messaggio; in questo caso il g-nodo *g* è individuato dalle coordinate
`(m’.lvl, m’.pos)`.

Questa classe è usata anche per individuare il nodo *n* originante di un messaggio e viene scritta nel
messaggio, nel suo membro *m’.n*. Il nodo *n*, quando genera il messaggio *m’*, individua anche un g-nodo a cui
esso appartiene e all'interno del quale avverrà tutto l'instradamento di *m’*. Esso è il g-nodo *g* in questo caso.

## <a name="PeerTupleGNode"></a>PeerTupleGNode

La classe serializzabile che identifica un g-nodo *h* (possibile che sia di livello 0 cioè un nodo) all'interno
di un g-nodo *g*. Deve contenere:

*   `top`:
    *   int. Il livello del g-nodo *g*.
*   `tuple`:
    *   la lista di posizioni h<sub>𝜀</sub>·...·h<sub>top-1</sub>.

Il valore di *𝜀*, che è il livello del g-nodo *h*, si calcola come `top - tuple.size`.

Il g-nodo *g* è sempre individuabile come vedremo sotto nei casi d'uso.

Si può testare la validità di un oggetto PeerTupleGNode ricevuto dalla rete verificando che:

*   0﹤`tuple.size` ≤ `top` ≤ `levels`
*   `𝜀` = `top` - `tuple.size`
*   Per `i` da 0 a `tuple.size-1`:
    *   0 ≤ `tuple[i]`﹤`gsizes[𝜀+i]`

Questa classe è usata per identificare, all'interno del messaggio *m’*, un g-nodo *h* da escludere dal calcolo
distribuito di *H<sub>t</sub>*; in questo caso il g-nodo *g* è individuato dalle coordinate `(m’.lvl, m’.pos)`.

Questa classe è usata anche per identificare un g-nodo *h* di cui si sta  divulgando la partecipazione ad un
servizio  opzionale; questa divulgazione avviene sempre a livello globale, quindi in questo caso il g-nodo *g*
è l'intera rete.

Questa classe è usata anche per identificare un g-nodo *h* di cui si sta divulgando la non-partecipazione ad
un servizio opzionale; questa divulgazione avviene contestualmente all'instradamento di un messaggio *m’* per
il calcolo distribuito di *H<sub>t</sub>* e siccome questo calcolo può avvenire anche in modo circoscritto ad
un g-nodo, in questo caso il g-nodo *g* di riferimento può non essere l'intera rete, ma sicuramente è sempre
uno dei g-nodi a cui il nodo corrente appartiene.

## <a name="PeerTupleGNodeContainer"></a>PeerTupleGNodeContainer

Classe contenitore non serializzabile, che contiene un set di PeerTupleGNode e fa in modo che quando vi si
inserisce un g-nodo *h* vengano contestualmente rimossi i g-nodi *h’* ∈ *h*.

Al momento della creazione di questo set va individuato il livello del g-nodo *g* all'interno del quale i
singoli g-nodi di questo set appartengono.

L'interfaccia esposta dalla classe consente di:

*   creare un contenitore vuoto, indicando il livello del g-nodo *g* contenitore.
*   aggiungere una tupla *h*.
*   ciclare le tuple presenti.

## <a name="PeerMessageForwarder"></a>PeerMessageForwarder

La classe serializzabile per inviare i messaggi *m’*. Deve contenere:

*   `inside_level`:
    *   int. La ricerca dell'hash-node per la chiave è stata fin dall'inizio circoscritta al g-nodo di questo livello.
*   `n`:
    *   PeerTupleNode di posizioni da 0 a *j* che identifica il nodo originante dentro il g-nodo di livello *j+1*
        in cui il messaggio si muove sin dall'inizio.
*   `x_macron`:
    *   PeerTupleNode di posizioni da 0 a `lvl-1` che identifica un indirizzo di nodo dentro il g-nodo attuale
        destinazione del messaggio. Può essere null se `lvl` è 0.
*   `lvl`:
    *   int.
*   `pos`:
    *   int. `(lvl, pos)` identificano il g-nodo attuale destinazione del messaggio, che ora si muove internamente
        al livello `lvl+1`.
*   `p_id`:
    *   int.
*   `msg_id`:
    *   int.
*   `exclude_tuple_list`:
    *   lista di PeerTupleGNode che rappresentano g-nodi da escludere dentro il g-nodo attuale destinazione
        del messaggio.
*   `non_participant_tuple_list`:
    *   lista di PeerTupleGNode che rappresentano g-nodi non partecipanti;
    *   Tali g-nodi sono dentro un g-nodo *g* a cui il nodo corrente appartiene. Se la ricerca distribuita
        di *H<sub>t</sub>* è portata avanti in modo circoscritto ad un g-nodo allora *g* è quel g-nodo; altrimenti
        *g* è l'intera rete.

Si può testare la validità di un oggetto PeerMessageForwarder ricevuto dalla rete verificando che:

*   `n` è valido
*   0 ≤ `lvl`﹤`levels`
*   0 ≤ `pos`﹤`gsizes[lvl]`
*   `n.tuple.size` > `lvl`
*   se `x_macron` non è null:
    *   `x_macron` è valido
    *   `x_macron.tuple.size` = `lvl`
*   se `exclude_tuple_list` non è vuoto:
    *   ogni elemento è valido
    *   ogni elemento ha `top` = `lvl`
*   se `non_participant_tuple_list` non è vuoto:
    *   ogni elemento è valido
    *   il primo elemento ha `top` > `lvl`
    *   ogni successivo elemento ha lo stesso valore del primo per `top`

## <a name="WaitingAnswer"></a>WaitingAnswer

Questa classe non serializzabile è usata dal nodo che inizia una chiamata. Il nodo ne crea un'istanza per mettervi le
informazioni da memorizzare durante l'attesa delle risposte, per ricevere le segnalazioni di eventi durante questa
attesa e per leggervi i risultati. Mette questa istanza in una mappa `waiting_answer_map` associandola al `msg_id`
come indice.

Contiene:

*   `ch`:
    *   Subito valorizzato. Channel usato per comunicare gli eventi alla tasklet che sta in attesa.
*   `request`:
    *   Subito valorizzato. Istanza di IPeersRequest da comunicare all'hash-node quando ci contatta. Può essere
        valorizzata a null se questa istanza di WaitingAnswer è in realtà usata solo per verificare la partecipazione
        di un dato g-nodo ad un servizio opzionale.
*   `min_target`:
    *   Inizialmente valorizzato dal nodo stesso con la  prima destinazione. Istanza di PeerTupleGNode, dice che quel
        g-nodo è  l'ultimo che è stato segnalato come prossima destinazione.
*   `exclude_gnode`:
    *   Inizialmente a null. Istanza di PeerTupleGNode, dice che quel g-nodo non ha altre destinazioni valide
        escludendo i g-nodi al suo interno che sono stati vietati dal  richiedente e bisogna ripartire.
*   `non_participant_gnode`:
    *   Inizialmente a null. Istanza di PeerTupleGNode, dice che quel g-nodo non partecipa e bisogna ripartire.
*   `respondant_node`:
    *   Inizialmente a null. Istanza di PeerTupleNode, dice che la richiesta è stata comunicata a quel nodo e quindi
        ora siamo in  attesa della risposta.
*   `response`:
    *   Inizialmente a null. Istanza di IPeersResponse ricevuta dall'hash-node come risposta, sia essa un risultato o
        una eccezione prevista dal servizio.
*   `refuse_message`:
    *   Stringa inizialmente a null. Viene valorizzata per segnalare che il `respondant_node` corrente, al quale è stata
        comunicata la richiesta, ha rifiutato di elaborarla. Quel nodo va quindi considerato da escludere e bisogna
        ripartire.
*   `redo_from_start`:
    *   Booleano inizialmente a False. Dice che è stata ricevuta l'istruzione di riavviare da capo il calcolo
        distribuito di *H<sub>t</sub>*.

## <a name="PeerParticipantMap"></a>PeerParticipantMap

Classe serializzabile in cui il nodo mantiene la mappa dei g-nodi visibili nella sua topologia che partecipano ad un
servizio opzionale. E' serializzabile perché viene comunicata ai nodi che la richiedono.

L'istanza contiene:

*   `participant_list`:
    *   lista di HCoord dei partecipanti: le coordinate sono riferite all'indirizzo del nodo che mantiene l'istanza.

Si può testare la validità di un oggetto PeerParticipantMap ricevuto dalla rete verificando che:

*   Per ogni HCoord `h` in `participant_list`:
    *   0 ≤ `h.lvl`﹤`levels`
    *   0 ≤ `h.pos`﹤`gsizes[h.lvl]`

La classe PeersManager ha un set di istanze di PeerParticipantMap, `participant_maps`, indicizzato con l'identificativo
del servizio.

Se il nodo partecipa al servizio opzionale *p* allora l'istanza `participant_maps[p.p_id]` esiste e almeno la sua
posizione è memorizzata.

Se il nodo non partecipa e non ha mai sentito parlare dell'identificativo `p_id` allora l'istanza `participant_maps[p_id]`
non esiste.

Quando il nodo (nel modulo PeersManager) viene informato della partecipazione o della non partecipazione al servizio
`p_id` di un g-nodo che è visibile nella sua topologia, allora agisce di conseguenza su `participant_maps[p_id]`.

Se ad un certo punto il nodo viene a conoscenza della partecipazione di un g-nodo al servizio `p_id` e
`participant_maps[p_id]` non esisteva, allora viene istanziata e viene memorizzata la posizione di quel g-nodo.

Se ad un certo punto il nodo viene a conoscenza della non partecipazione di un g-nodo al servizio `p_id` e
`participant_maps[p_id]` non esisteva, allora questo non ha conseguenze su `participant_maps`.

Se ad un certo punto il nodo viene a conoscenza della non partecipazione di un g-nodo al servizio `p_id` e questo fa
sì che `participant_maps[p_id].participant_list` risulta ora vuota, allora viene rimosso l'elemento `p_id` dal
set `participant_maps`.

## <a name="PeerParticipantSet"></a>PeerParticipantSet

Classe serializzabile usata per passare tutto insieme il set `participant_maps` di PeersManager. Contiene:

*   `participant_set`:
    *   HashMap con chiave int (l'identificativo del servizio opzionale) e valore PeerParticipantMap.

Si può testare la validità di un oggetto PeerParticipantSet ricevuto dalla rete verificando che:

*   Per ogni PeerParticipantMap `m` in `participant_set`:
    *   `m` è valido

