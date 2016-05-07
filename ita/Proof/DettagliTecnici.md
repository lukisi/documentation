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
Il programma tiene anche in memoria questo elenco di nomi di interfacce nella lista *real_nics*.

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

Il programma *qspnclient* vuole tenere traccia delle identità che sono nel nodo. Lo fa tramite il
dizionario *nodeids*, in cui la chiave è l'indice autoincrementante *nodeid_nextindex* e il valore è
una istanza della classe *IdentityData*. Dentro questa classe il programma mantiene alcune informazioni, tra le quali:

*   `NodeID nodeid` - L'identificativo che il modulo Identities ha assegnato all'identità.
*   `my_addr` - L'indirizzo Netsukuku.
*   `my_fp` - Il fingerprint e le anzianità a livello 0.
*   `LinuxRoute route` - Una classe che, vedremo dopo, aiuta a gestire il network namespace.

Per valorizzare la prima istanza di IdentityData, nel dizionario con indice 0 e associata alla
prima identità del nodo, il programma chiama il metodo *get_main_id* di IdentityManager per
recuperare il NodeID che il modulo Identities ha assegnato alla prima identità.

In questo stesso momento il NodeID viene mostrato a video con il relativo indice. In seguito l'utente può
rivederli con il comando interattivo *show_nodeids*.

Inoltre in questa prima istanza di IdentityData viene memorizzata la prima istanza di *LinuxRoute* che
gestisce il network namespace default.

* * *

Dopo aver recuperaro il NodeID della sua prima identità, il programma *qspnclient* crea la prima istanza di
QspnManager e la associa a questa identità con il metodo *set_identity_module* di IdentityManager usando il nome di modulo "qspn".

Questa prima istanza di QspnManager viene realizzata col costruttore *create_net*. Per avvalersi di questo
costruttore, al programma serve solo conoscere la topologia della rete e l'indirizzo Netsukuku del nodo. Queste
due informazioni sono passate al programma dalla riga di comando tramite i primi due argomenti.

*   args\[1] = topologia. Ad esempio la stringa "4.16.256.256" indica una rete a 4 livelli, dove la gsize al livello 3
    è 4, ecc. ecc.
*   args\[2] = indirizzo. Ad esempio la stringa "3.10.123.45" indica un indirizzo valido nella suddetta topologia.

Per comodità, il programma *qspnclient* assume che la topologia di rete usata sia la stessa per tutti i
nodi che prendono parte al testbed. Questo significa che quando l'utente richiederà l'ingresso di un nodo
in una diversa rete (vedere sotto il comando interattivo *enter_net*) la topologia non dovrà essere di nuovo specificata.

Qui il programma *qspnclient* crea la prima istanza della classe *LinuxRoute*, che è destinata
al network namespace default. Con la classe LinuxRoute si intende gestire gli aspetti delle routing
policy di un dato network namespace.

In seguito, quando si forma una nuova identità (con il comando interattivo *add_identity*),
necessariamente viene anche creato un nuovo network namespace temporaneo. Allora il programma *qspnclient*
creerà una nuova istanza della classe LinuxRoute destinata a quel network namespace.

Quando viene creata una istanza di QspnManager (qui e anche con il comando interattivo *enter_net*)
come prima operazione con l'istanza di LinuxRoute che gestisce il network namespace associato
a questa identità, il programma *qspnclient* assegna al nodo gli indirizzi IP che servono. Usa per questo
il metodo `add_address` di LinuxRoute.

In questo caso specifico del costruttore *create_net*, siamo di fronte ad una identità principale che detiene un
indirizzo Netsukuku *reale* *n*. Quindi il programma assegna al nodo:

*   L'indirizzo IP globale di *n*.
*   L'indirizzo IP globale anonimizzante di *n*.
*   Per tutti i livelli *j* da 0 a *l* - 2 (indicando con *l* il numero dei livelli della topologia):
    *   L'indirizzo IP interno al livello *j* + 1 di *n*.

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

Come risultato viene creata una nuova identità *i1* nel nodo A basata sulla precedente identità *i0*,
il cui nuovo NodeID viene restituito al programma dalla chiamata *add_identity*. Il programma
crea una nuova istanza di IdentityData per tenere traccia di *i1*; la associa al prossimo valore dell'indice
autoincrementante *nodeid_nextindex*, nel dizionario *nodeids*. In questo esempio sia 1.

Inoltre è stato anche realizzato un nuovo network namespace temporaneo; l'identità *i1* inizierà ora a gestire il
vecchio namespace che era gestito da *i0*, mentre l'identità *i0* avrà in gestione
il nuovo network namespace. Ne consegue che l'istanza di LinuxRoute che si trovava
memorizzata nell'istanza di IdentityData associata a *i0*, nel membro *route*, viene ora memorizzata nella
nuova istanza di IdentityData associata a *i1*; nell'istanza di IdentityData associata
a *i0* verrà invece memorizzata una nuova istanza di LinuxRoute, creata per il nuovo
network namespace.

Inoltre avverrà anche che nel nodo B si rileva la creazione di un nuovo arco-identità tra l'identità che
già era in B e la nuova identità in A. Verrà mostrato a video nella console del programma in esecuzione
nel nodo B insieme al relativo indice di *identityarcs*. In questo esempio sia 1.

Analogamente nel nodo A si rileva la creazione del medesimo nuovo arco-identità. Verrà mostrato a video
nella console del programma in esecuzione nel nodo A insieme al relativo indice di *identityarcs*. In questo esempio sia 1.

Poi l'utente chiederà dalla console di B di vedere le informazioni di indirizzo e anzianità ai vari livelli
di B. L'utente dovrà prima recuperare l'indice del *nodeid* dove si vuole fare ingresso (supponiamo sia ancora 0).
Alla domanda il programma *qspnclient* visualizzerà ad esempio "indirizzo 3·10·123·45, anzianità 0·0·0·0".
Il comando interattivo per questo scopo è *show_ntkaddress*.

Poi l'utente, simulando un accordo tra il nodo A e il nodo B (e un eventuale dialogo tra il nodo B e il
Coordinator di uno dei suoi g-nodi), determina un posto in un g-nodo della rete di B che viene riservato
per A. Quindi istruirà il programma *qspnclient* nel nodo A con le informazioni che servono affinché esso
crei una nuova istanza di QspnManager con il costruttore *enter_net* da assegnare alla nuova identità.

Ad esempio simuliamo che il posto riservato sia "indirizzo 3·10·67, anzianità 0·0·1" di livello 1 (riservato
dal Coordinator del g-nodo esistente "3·10" di livello 2). Supponiamo che il vecchio indirizzo di A
era "indirizzo 3·11·230·89, anzianità 0·0·0·0". Allora il nuovo indirizzo di A (secondo le regole illustrate
nella trattazione del modulo QSPN al punto [Ingresso](../ModuloQSPN/DettagliTecnici.md#Ingresso)) sarà
"indirizzo 3·10·67·89, anzianità 0·0·1·0".

Fatta questa scelta, le informazioni da dare al programma *qspnclient* nel nodo A (nel comando
interattivo *enter_net*) per chiamare il costruttore *enter_net* di QspnManager sono:

*   `int new_nodeid_index`. L'indice del nuovo NodeID di A. In questo esempio l'indice 1.
*   `int previous_nodeid_index`. L'indice del precedente NodeID di A. Serve per copiare i percorsi e
    il tipo di indirizzo (definitivo). In questo esempio l'indice 0.
*   `string address_new_gnode`. L'indirizzo del nuovo g-nodo. In questo esempio la stringa "3.10.67".
*   `string elderships_new_gnode`. Le anzianità del nuovo g-nodo. In questo esempio la stringa "0.0.1".
*   `int hooking_gnode_level`. Il livello del g-nodo che fa ingresso. In questo esempio 0.
*   `int into_gnode_level`. Il livello del g-nodo in cui fa ingresso. In questo esempio 2.
*   `List<int> idarc_index_set`. Una lista degli indici degli archi-identità sui quali va costruita la
    lista di IQspnArc. In questo esempio \[1], cioè una lista con un elemento 1.
*   `List<string> idarc_address_set`. Una lista compagna della precedente, con i relativi indirizzi
    Netsukuku dei vicini collegati da questi archi. In questo esempio \["3.10.123.45"].

Subito dopo aver costruito la nuova istanza di QspnManager, il comando interattivo *enter_net*
per mezzo delle istanze di LinuxRoute associate alle due identità esegue queste operazioni:

*   La vecchia identità rimuove dal suo vecchio network namespace i relativi indirizzi IP.
*   La nuova identità si assegna nel network namespace (precedentemente gestito dalla vecchia) i relativi indirizzi IP.
*   La vecchia identità si assegna nel suo nuovo network namespace i relativi indirizzi IP.

Successivamente — ma in tempi molto rapidi perché il modulo QSPN prevede un tempo massimo
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
    *   `string key`  
        La chiave è composta dal MAC della mia interfaccia e il MAC dell'interfaccia del
        vicino, separati da un trattino. Come compare a video con il comando `show_neighborhood_arcs`.
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

## Metodi di LinuxRoute

Elenchiamo le funzionalità che si vogliono implementare nella classe LinuxRoute.

*   Abbiamo una istanza di questa classe per ogni network namespace gestito dal programma.  
    Inizialmente il programma gestisce il network namespace default con la sua prima
    *identità*, che è anche la *principale*. Nell'istanza della classe IdentityData
    che si riferisce a questa identità viene memorizzata, nel membro *route*, l'istanza di
    LinuxRoute che è stata creata per il network namespace default.  
    Quando il programma crea una nuova identità *i1* basata sulla precedente identità *i0*
    viene creato un nuovo network namespace; l'identità *i1* inizierà ora a gestire il
    vecchio namespace che era gestito da *i0*, mentre l'identità *i0* avrà in gestione
    il nuovo network namespace. Ne consegue che l'istanza di LinuxRoute che si trovava
    nell'istanza di IdentityData associata a *i0* viene ora memorizzata nella
    nuova istanza di IdentityData associata a *i1*; nell'istanza di IdentityData associata
    a *i0* verrà invece memorizzata una nuova istanza di LinuxRoute, creata per il nuovo
    network namespace.

*   Una identità, oltre a gestire un particolare network namespace, detiene un particolare
    indirizzo Netsukuku e mantiene una mappa di percorsi verso destinazioni note.  
    Come conseguenza, l'istanza di LinuxRoute che gestisce quel network namespace e che al
    momento è memorizzata nel membro *route* dell'istanza di IdentityData associata
    a questa identità, ha assegnato a quel network namespace un numero di indirizzi IP
    propri e ha impostato nelle sue tabelle un numero di rotte verso altri indirizzi IP.  
    Quando il programma crea una nuova identità *i1* basata sulla precedente identità *i0*
    avviene che l'identità *i0* vede cambiare il proprio indirizzo Netsukuku. Infatti diventa
    una identità *di connettività* per un livello in cui prima non lo era. Inoltre la
    nuova identità *i1* deterrà un nuovo indirizzo Netsukuku ancora diverso.

    Ad esempio possiamo avere nel nodo *n* l'identità *n0* con indirizzo 3·2·3·1 in una topologia 4·4·4·4.
    Questo è un indirizzo *reale* quindi si tratta di una identità principale.  
    Ora supponiamo che il nodo 3·2·3·1 vuole migrare in 3·2·2·2, restando nel g-nodo 3·2·3
    con l'identificativo *virtuale* 3·2·3·6.  
    Quindi dentro *n* si aggiunge l'identità *n1* basata su *n0* con indirizzo 3·2·2·2  (che nasce come
    identità principale con un indirizzo Netsukuku *reale*) mentre l'identità *n0* diventa
    *di connettività* al livello 1 con indirizzo 3·2·3·6.  
    Poi supponiamo che il g-nodo 3·2
    migra in 1·0, restando dentro il g-nodo 3 con l'identificativo *virtuale* 3·5. Dentro il
    g-nodo 3·2 abbiamo sia il nodo 3·2·2·2 (cioè l'identità *n1* dentro *n*) sia il
    nodo 3·2·3·6 (cioè l'identità *n0* dentro *n*).  
    Quindi dentro *n* si aggiunge l'identità *n2* basata su *n0* con indirizzo 1·0·3·6 (che nasce come
    identità *di connettività* al livello 1) mentre l'identità *n0* diventa
    *di connettività* al livello 3 con indirizzo 3·5·3·6.  
    Inoltre dentro *n* si aggiunge l'identità *n3* basata su *n1* con indirizzo 1·0·2·2 (che nasce come
    identità principale con un indirizzo Netsukuku *reale*) mentre l'identità *n1* diventa
    *di connettività* al livello 3 con indirizzo 3·5·2·2.

*   Quando si costruisce una istanza di LinuxRoute, significa che è stato creato un nuovo
    network namespace. Possono esserci questi casi:

    *   Inizio. Il network namespace assegnato alla prima identità *n0* non era gestito in precedenza
        da un'altra identità.  
        L'istanza di LinuxRoute *r_alfa* riceve nel costruttore la stringa `ns` che identifica il
        namespace. In questo caso di norma è il default, cioè `""`.  
        In seguito, sull'istanza di LinuxRoute *r_alfa*, siccome essa gestisce il network namespace default,
        viene chiamato il metodo `add_address(address, dev)` varie
        volte per assegnare un indirizzo IP alle interfacce di rete reali.  
        Nel tempo, sull'istanza di LinuxRoute *r_alfa* come risposta ai segnali notificati dal
        QspnManager della relativa *identità*, vengono chiamati i metodi `add_route` o `change_route`
        o `remove_route` per ogni cambiamento alle policy di routing nel network
        namespace relativo.
    *   Migrazione. L'identità *n1* costruita sull'identità *n0*. Il network namespace
        assegnato alla *n1* da parte dell'IdentityManager è quello che prima era stato gestito
        dalla *n0*. Il programma lo può recuperare usando il metodo `get_namespace` dell'IdentityManager.  
        Ora all'identità *n0* è stato assegnato un diverso namespace *x* da parte dell'IdentityManager.
        Il programma lo può recuperare usando il metodo `get_namespace`.  
        All'identità *n1* viene ora associata l'istanza di LinuxRoute *r_alfa*.  
        Una nuova istanza di LinuxRoute, *r_beta*, viene creata e riceve nel costruttore la stringa `ns` che identifica il
        namespace *x* ora gestito dall'identità *n0*. Poi all'identità *n0* viene associata l'istanza *r_beta*.  
        Sull'istanza *r_alfa*, cioè quella associata a *n1*, viene chiamato il metodo `remove_addresses`
        per rimuovere tutti gli indirizzi IP propri che aveva assegnato al suo network namespace:
        infatti proprio *r_alfa* li aveva assegnati (con `add_address`) alle varie \[pseudo]interfacce
        gestite nel suo network namespace, quando era ancora associata a *n0*.  
        In seguito, sull'istanza di LinuxRoute *r_alfa*, cioè quella associata a *n1*, se essa gestisce il network namespace default,
        viene chiamato il metodo `add_address(address, dev)` varie
        volte per assegnare un indirizzo IP alle interfacce di rete reali.  
        Non è necessario, sull'istanza di LinuxRoute *r_beta*, cioè quella associata a *n0*, chiamare mai il metodo `add_address`
        in quanto nei network namespace diversi dal default il sistema non detiene mai
        un indirizzo IP proprio, nemmeno interno ad un livello.  
        Nel tempo, su entrambe le istanze *r_alfa* e *r_beta* come risposta ai segnali notificati dal
        QspnManager della relativa *identità*, vengono chiamati i metodi `add_route` o `change_route`
        o `remove_route` per ogni cambiamento alle policy di routing nel network
        namespace relativo.

*   Una istanza di LinuxRoute ha impostato (con `add_address`) tutti gli indirizzi IP
    che nel tempo sono stati assegnati alle varie \[pseudo]interfacce gestite in un
    particolare network namespace da una particolare *identità*.  
    In realtà abbiamo già detto che questo avviene solo nel network namespace default, poiché
    non serve assegnare un indirizzo IP proprio (nemmeno interno ad un g-nodo) ad una identità
    *di connettività*; comunque la classe LinuxRoute lo permetterebbe, se volessimo.  
    Quando questa *identità* cessa di gestire questo network namespace (perché cessa
    di esistere oppure perché le viene assegnato un diverso network namespace) questa istanza
    di LinuxRoute è in grado di sapere quali indirizzi IP rimuovere. Quindi in questo
    momento viene chiamato il metodo `remove_addresses` di questa istanza. In esso tutti
    gli indirizzi IP attualmente assegnati (dal programma) alle varie \[pseudo]interfacce
    vengono rimossi.  
    Distinguiamo i seguenti casi:

    *   Quando una *identità* migra (poteva essere l'identità *principale* del nodo oppure
        un'identità *di connettività*) essa abbandona un network namespace che sarà in
        seguito gestito da una diversa *identità*. In questo caso l'operazione `remove_addresses`
        è **strettamente** necessaria, sia che si tratti del network namespace default,
        sia di un altro.
    *   Quando una *identità di connettività* cessa di esistere (ad esempio perché non
        è più necessaria la sua presenza per garantire la connettività dei g-nodi, oppure
        perché nel sistema il programma *qspnclient* termina) essa abbandona un
        network namespace che in seguito non sarà più gestito nemmeno da altre *identità*
        ma verrà distrutto con tutte le relative pseudo-interfacce. In questo caso
        l'operazione `remove_addresses` non è necessaria.
    *   Quando l'*identità principale* cessa di esistere perché nel sistema il programma
        *qspnclient* termina, essa abbandona il network namespace default. In questo caso
        l'operazione `remove_addresses` è necessaria a ripulire il sistema.

### Prontuario sintetico

La parte descrittiva sopra verrà rimossa se/quando tutti gli esempi saranno stati integrati
nella parte alta del documento.

*   **costruttore**  
    Argomenti:

    *   `string ns`  
        Il nome del network namespace gestito attraverso questa nuova istanza.

    Questo costruttore **TODO**...

*   **add_address**  
    Argomenti:

    *   `string address`  
        Indirizzo IP da assegnare.
    *   `string dev`  
        Interfaccia (o pseudo) di rete gestita.

    Questo metodo **TODO**...

*   **remove_addresses**  
    Argomenti: nessuno.  
    Questo metodo **TODO**...


