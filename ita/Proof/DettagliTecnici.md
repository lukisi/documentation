# Proof of concept - Dettagli Tecnici

1.  **TODO**
1.  **TOC**

Per evitare conflitti di nomi, il programma *qspnclient* definisce le proprie classi e funzioni
nel namespace *ProofOfConcept* mentre usa i diversi namespace _Netsukuku.*_ per accedere alle
classi dei vari moduli. Nel namespace *Netsukuku* sono definite le classi fornite dalle
librerie *ntkd-common* e *ntkdrpc*.

* * *

Il programma *qspnclient* riceve all'avvio dall'utente, con una serie di flag *-i*, l'elenco
delle interfacce di rete da gestire.

Questo elenco di nomi di interfacce viene immediatamente passato al modulo Neighborhood (con il
metodo *start_monitor* di NeighborhoodManager) nello stesso ordine in cui è stato proposto dall'utente.
Non c'è bisogno per il programma di tenerli in memoria altrove.

Il modulo Neighborhood, in base all'ordine in cui gli sono passati, produce una serie di chiamate
al metodo *add_address* di INeighborhoodIPRouteManager. Poi emette il segnale *nic_address_set*. Quando
riceve questo segnale il programma crea una istanza di ProofOfConcept.HandledNic che contiene:

*   `string dev`.
*   `string mac`.
*   `string linklocal`.

Poi associa tale istanza al prossimo valore dell'indice autoincrementante *linklocal_nextindex*, nel dizionario *linklocals*.

In questo stesso momento i dati di questa istanza di ProofOfConcept.HandledNic vengono mostrati a video
con il relativo indice. In seguito l'utente può rivederli con il comando interattivo *show_linklocals*.

* * *

Nel momento in cui fa le chiamate al metodo *start_monitor* del NeighborhoodManager, il programma
*qspnclient* rileva il segnale *nic_address_set* quando viene assegnato un link-local a queste interfacce,
quindi in seguito avrà un elenco di ProofOfConcept.HandledNic in *linklocals*.

Poi il programma inizializza il modulo Identities (col costruttore di IdentityManager) passando l'elenco
delle interfacce di rete gestite, ognuna col relativo MAC e indirizzo link-local. Se in seguito rileva
ancora il segnale *nic_address_set* di INeighborhoodIPRouteManager, mentre cioè l'istanza di IdentityManager
è stata già costruita, dovrà chiamare il suo metodo *add_handled_nic*.

Dopo aver costruito IdentityManager il programma si mette in ascolto dei suoi segnali *identity_arc_added*,
*changed* e *removed*.

Poi il programma chiama il metodo *get_main_id* di IdentityManager per recuperare il NodeID che il modulo
Identities ha assegnato alla prima identità. Associa tale NodeID al prossimo valore dell'indice
autoincrementante *nodeid_nextindex*, nel dizionario *nodeids*.

In questo stesso momento il NodeID viene mostrato a video con il relativo indice. In seguito l'utente può
rivederli con il comando interattivo *show_nodeids*.

* * *

Dopo aver recuperaro il NodeID della sua prima identità, il programma *qspnclient* crea la prima istanza di
QspnManager e la associa a questa identità con il metodo *set_identity_module* di IdentityManager usando il nome di modulo "qspn".

Questa prima istanza di QspnManager viene realizzata col costruttore *create_net*. Per avvalersi di questo
costruttore, al programma serve solo conoscere la topologia della rete e l'indirizzo Netsukuku del nodo. Queste
due informazioni sono passate al programma dalla riga di comando tramite i primi due argomenti.

*   args[1] = topologia. Ad esempio la stringa "8.2.2.2" indica una rete a 4 livelli, dove la gsize al livello 3
    è 8 e agli altri livelli è 2.
*   args[2] = indirizzo. Ad esempio la stringa "4.1.0.1" indica un indirizzo valido nella suddetta topologia.

Per comodità, il programma *qspnclient* assume che la topologia di rete usata sia la stessa per tutti i
nodi che prendono parte al testbed. Questo significa che quando l'utente richiederà l'ingresso di un nodo
in una diversa rete (vedere sotto il comando interattivo *enter_net*) la topologia non dovrà essere di nuovo specificata.

* * *

Quando il programma riceve dal modulo Neighborhood la notifica che un arco è stato realizzato, esso mostra
le informazioni (link-local e MAC dei due estremi) dell'arco a video. Inoltre mette la relativa istanza di
INeighborhoodArc nel dizionario *neighborhood_arcs* con una chiave stringa composta dai due MAC address.

In seguito l'utente può rivederli con il comando interattivo *show_neighborhood_arcs*.

In seguito l'utente può dare istruzione di accettare un arco con il comando interattivo *add_nodearc* indicando:

*   MAC della propria interfaccia.
*   MAC del vicino.
*   Costo in latenza in microsecondi.

Il programma a questo punto crea una istanza della classe ProofOfConcept.Arc che ha questi membri:

*   `int cost`.
*   `INeighborhoodArc neighborhood_arc`.
*   `ProofOfConcept.IdmgmtArc idmgmt_arc`.

Associa questa istanza di ProofOfConcept.Arc al prossimo valore dell'indice autoincrementante
*nodearc_nextindex*, nel dizionario *nodearcs*.

Poi il programma, basandosi su questa istanza di ProofOfConcept.Arc, istanzia una classe
ProofOfConcept.IdmgmtArc che implementa IIdmgmtArc che andrà passata al modulo Identities nel
metodo *add_arc*. Al suo interno mantiene (in un riferimento weak per evitare il loop) l'istanza di ProofOfConcept.Arc.

I dati che servono per implementare i metodi di IIdmgmtArc sono:

*   *get_dev()* – `arc.neighborhood_arc.nic.dev`.
*   *get_peer_linklocal()* – `arc.neighborhood_arc.neighbour_nic_addr`.
*   *get_peer_mac()* – `arc.neighborhood_arc.neighbour_mac`.

Questa istanza di ProofOfConcept.IdmgmtArc viene anche salvata nel membro *idmgmt_arc* della stessa
istanza di ProofOfConcept.Arc. Così il programma in seguito partendo dall'indice di *nodearcs* potrà recuperarla.

In questo stesso momento tutti questi dati sono mostrati a video con il relativo indice. In seguito
l'utente può rivederli con il comando interattivo *show_nodearcs*.

* * *

Quando riceve il segnale *identity_arc_added* di IdentityManager, il programma crea una istanza di
ProofOfConcept.IdentityArc che ha questi membri:

*   `IIdmgmtArc arc`. Si tratta di una istanza di ProofOfConcept.IdmgmtArc che ci permette di risalire
    al ProofOfConcept.Arc.
*   `NodeID id`. Si tratta di uno dei propri NodeID.
*   `IIdmgmtIdentityArc id_arc`.

Associa questa istanza di ProofOfConcept.IdentityArc al prossimo valore dell'indice autoincrementante
*identityarc_nextindex*, nel dizionario *identityarcs*.

In questo stesso momento tutti questi dati sono mostrati a video con il relativo indice. In seguito
l'utente può rivederli con il comando interattivo *show_identityarcs*.

* * *

Supponiamo ora che due nodi A e B hanno costituito un arco-identità. Il relativo indice di *identityarcs*
è mostrato a video, in questo esempio sia 0 nel nodo A e 0 nel nodo B. Supponiamo che il nodo A vuole entrare
nella rete del nodo B. L'utente alla console del nodo A si inventa un *migration_id* (ad esempio 1234) e
recupera l'indice del *nodeid* che vuole fare ingresso (in questo esempio sia 0) istruisce il programma
*qspnclient* di chiamare i metodi *prepare_add_identity(1234, nodeids[0])* e subito dopo
*add_identity(1234,nodeids[0])* di IdentityManager. I comandi interattivi per questo scopo
sono *prepare_add_identity* e *add_identity*.

Come risultato viene creata una nuova identità nel nodo A. Il nuovo NodeID viene restituito al programma
dalla chiamata *add_identity*. Il programma associa tale NodeID al prossimo valore dell'indice
autoincrementante *nodeid_nextindex*, nel dizionario *nodeids*. In questo esempio sia 1.

Inoltre avverrà anche che nel nodo B si rileva la creazione di un nuovo arco-identità tra l'identità che
già era in B e la nuova identità in A. Verrà mostrato a video nella console del programma in esecuzione
nel nodo B insieme al relativo indice di *identityarcs*. In questo esempio sia 1.

Analogamente nel nodo A si rileva la creazione del medesimo nuovo arco-identità. Verrà mostrato a video
nella console del programma in esecuzione nel nodo A insieme al relativo indice di *identityarcs*. In questo esempio sia 1.

Poi l'utente chiederà dalla console di B di vedere le informazioni di indirizzo e anzianità ai vari livelli
di B. L'utente dovrà prima recuperare l'indice del *nodeid* dove si vuole fare ingresso (supponiamo sia ancora 0).
Alla domanda il programma *qspnclient* visualizzerà ad esempio "indirizzo 4.1.0.1, anzianità 0.0.0.0".
Il comando interattivo per questo scopo è *show_ntkaddress*.

Poi l'utente, simulando un accordo tra il nodo A e il nodo B (e un eventuale dialogo tra il nodo B e il
Coordinator di uno dei suoi g-nodi), determina un posto in un g-nodo della rete di B che viene riservato
per A. Quindi istruirà il programma *qspnclient* nel nodo A con le informazioni che servono affinché esso
crei una nuova istanza di QspnManager con il costruttore *enter_net* da assegnare alla nuova identità.

Ad esempio simuliamo che il posto riservato sia "indirizzo 4.1.1, anzianità 0.0.1" di livello 1 (riservato
dal Coordinator del g-nodo esistente "4.1" di livello 2). Supponiamo che il vecchio indirizzo di A
era "indirizzo 3.0.0.1, anzianità 0.0.0.0". Allora il nuovo indirizzo di A (secondo le regole illustrate
nella trattazione del modulo QSPN al punto [Ingresso](../ModuloQSPN/DettagliTecnici.md#Ingresso)) sarà
"indirizzo 4.1.1.1, anzianità 0.0.1.0".

Fatta questa scelta, le informazioni da dare al programma *qspnclient* nel nodo A (nel comando
interattivo *enter_net*) per chiamare il costruttore *enter_net* di QspnManager sono:

*   `int new_nodeid_index`. L'indice del nuovo NodeID di A. In questo esempio l'indice 1.
*   `int previous_nodeid_index`. L'indice del precedente NodeID di A. Serve per copiare i percorsi e
    il tipo di indirizzo (definitivo). In questo esempio l'indice 0.
*   `string address_new_gnode`. L'indirizzo del nuovo g-nodo. In questo esempio la stringa "4.1.1".
*   `string elderships_new_gnode`. Le anzianità del nuovo g-nodo. In questo esempio la stringa "0.0.1".
*   `int hooking_gnode_level`. Il livello del g-nodo che fa ingresso. In questo esempio 0.
*   `int into_gnode_level`. Il livello del g-nodo in cui fa ingresso. In questo esempio 2.
*   `List<int> idarc_index_set`. Una lista degli indici degli archi-identità sui quali va costruita la
    lista di IQspnArc. In questo esempio [1], cioè una lista con un elemento 1.
*   `List<string> idarc_address_set`. Una lista compagna della precedente, con i relativi indirizzi
    Netsukuku dei vicini collegati da questi archi. In questo esempio ["4.1.0.1"].

Allo stesso momento — o per lo meno in tempi molto rapidi perché il modulo QSPN prevede un tempo massimo
di rilevamento dell'arco, che è fissato dal programma *qspnclient* a 10 secondi — sulla console del nodo
B il programma *qspnclient* deve essere istruito dall'utente ad aggiungere un IQspnArc al suo QspnManager
con il metodo *arc_add*. Le informazioni da dare (nel comando interattivo *add_qspnarc*) per questo sono:

*   `int nodeid_index`. L'indice del NodeID di B. Per recuperare l'istanza di QspnManager su cui
    operare. In questo esempio l'indice 0.
*   `int idarc_index`. L'indice dell'arco-identità sul quale va costruito il IQspnArc. In questo esempio 1.
*   `string idarc_address`. L'indirizzo Netsukuku del vicino collegato da questo arco. In questo esempio la stringa "4.1.1.1".

## Elenco comandi interattivi

*   **show_linklocals**
*   **show_nodeids**
*   **show_neighborhood_arcs**
*   **add_nodearc**
    *   `string my_mac`
    *   `string peer_mac`
    *   `int cost`
*   **show_nodearcs**
*   **show_identityarcs**
*   **show_ntkaddress**
    *   `int nodeid_index`
*   **prepare_add_identity**
    *   `int migration_id`
    *   `int nodeid_index`
*   **add_identity**
    *   `int migration_id`
    *   `int nodeid_index`
*   **enter_net**
    *   `int new_nodeid_index`
    *   `int previous_nodeid_index`
    *   `string address_new_gnode`
    *   `string elderships_new_gnode`
    *   `int hooking_gnode_level`
    *   `int into_gnode_level`
    *   ripetuti 1 o più volte:
        *   `int identityarc_index`
        *   `string identityarc_address`
*   **add_qspnarc**
    *   `int nodeid_index`
    *   `int identityarc_index`
    *   `string identityarc_address`

