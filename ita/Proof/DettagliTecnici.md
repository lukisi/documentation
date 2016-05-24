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
*   `LinuxRoute route` - Una classe, che dettaglieremo dopo, con la quale si intende gestire gli
    aspetti delle routing policy di un dato network namespace.

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

Abbiamo già detto che alla prima *identità* il programma *qspnclient* associa la prima istanza della classe
*LinuxRoute*, che è destinata al network namespace default.

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

Successivamente l'utente potrà modificare il costo associato ad un arco che era stato in precedenza
accettato. Lo fa con il comando interattivo `change_nodearc` specificando il nuovo costo e individuando
l'arco tramite il suo *nodearc_index*.

In questo momento il programma *qspnclient* guarda quali delle sue identità hanno una
istanza di QspnArc (di cui parliamo sotto) associata a questo arco. Per ognuna di esse il
programma notifica questa variazione di costo alla relativa istanza di QspnManager.

L'utente potrà anche rimuovere un arco che era stato in precedenza accettato. Lo fa con il comando
interattivo `remove_nodearc` individuando l'arco tramite il suo *nodearc_index*.

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

Come risultato viene creata una nuova identità *i<sub>1</sub>* nel nodo A basata sulla precedente identità *i<sub>0</sub>*,
il cui nuovo NodeID viene restituito al programma dalla chiamata *add_identity*. Il programma
crea una nuova istanza di IdentityData per tenere traccia di *i<sub>1</sub>*; la associa al prossimo valore dell'indice
autoincrementante *nodeid_nextindex*, nel dizionario *nodeids*. In questo esempio sia 1.

Inoltre è stato anche realizzato un nuovo network namespace temporaneo; l'identità *i<sub>1</sub>* inizierà ora a gestire il
vecchio namespace che era gestito da *i<sub>0</sub>*, mentre l'identità *i<sub>0</sub>* avrà in gestione
il nuovo network namespace. Ne consegue che l'istanza di LinuxRoute, chiamiamola *r<sub>𝛼</sub>*, che si trovava
memorizzata nell'istanza di IdentityData associata a *i<sub>0</sub>*, nel membro *route*, viene ora memorizzata nella
nuova istanza di IdentityData associata a *i<sub>1</sub>*; nell'istanza di IdentityData associata
a *i<sub>0</sub>* verrà invece memorizzata una nuova istanza di LinuxRoute, chiamiamola *r<sub>𝛽</sub>*, creata per il nuovo
network namespace.

Alcune delle rotte impostate da *r<sub>𝛼</sub>* nel suo network namespace dovranno presto essere
rimosse, mentre altre dovranno rimanere. Analogamente, alcuni indirizzi IP, che nel tempo sono stati impostati
da *r<sub>𝛼</sub>* nel suo network namespace assegnandoli alle varie \[pseudo]interfacce gestite,
dovranno presto essere rimossi, mentre altri dovranno rimanere. Tutto questo deriva dal fatto che ora
quel network namespace è gestito da una diversa identità *i<sub>1</sub>*.  
Quando l'utente dà il comando interattivo *add_identity* non sappiamo ancora quale sia il g-nodo
che in blocco entra in una nuova rete oppure migra in un diverso g-nodo. In base al livello
del g-nodo che *si muove* e al livello del g-nodo in cui esso entra, alcuni indirizzi IP interni
e le relative rotte dovranno restare in esistenza nel network namespace gestito da *r<sub>𝛼</sub>*
mentre altri dovranno essere rimossi.  
Quindi rinviamo queste considerazioni ad un prossimo momento: in questo caso, al momento in cui
l'utente dà il comando interattivo *enter_net*.

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

A questo punto, come dicevamo prima, il programma *qspnclient* conosce i dettagli di questo ingresso
in una nuova rete. È il momento giusto per dare istruzioni all'istanza di LinuxRoute *r<sub>𝛼</sub>* (che ora è
associata all'identità *i<sub>1</sub>*) di rimuovere le rotte e gli indirizzi IP propri che non sono
più validi, lasciando in vigore le rotte (e relativi indirizzi propri) che sono validi in quanto
interni ad un livello inferiore a quello del g-nodo che migra.

Le rotte verso indirizzi IP globali e le rotte verso indirizzi IP interni ad un livello *k* tale
che *k* > `hooking_gnode_level` vanno rimosse. Per farlo il programma *qspnclient* fa diverse
chiamate al metodo `remove_destination` di *r<sub>𝛼</sub>* sulla base dei percorsi che sono
noti al precedente QspnManager.

L'indirizzo IP proprio globale e quelli interni ad un livello *k* tale
che *k* > `into_gnode_level-1` vanno rimossi. Per farlo il programma *qspnclient* fa diverse
chiamate al metodo `remove_address` di *r<sub>𝛼</sub>* sulla base dell'indirzzo Netsukuku che era
detenuto dalla precente identità (memorizzato nella classe IdentityData).

Subito dopo, il comando interattivo *enter_net* costruisce la nuova istanza di QspnManager. Poi,
per mezzo delle istanze di LinuxRoute associate alle due identità esegue queste operazioni:

*   L'identità *i<sub>1</sub>* usa la sua istanza di LinuxRoute (che ora è *r<sub>𝛼</sub>*) per assegnarsi i relativi indirizzi IP.  
    Per l'esattezza, l'indirizzo IP globale e quelli interni ai livelli da `into_gnode_level` in su.
*   L'identità *i<sub>0</sub>* usa la sua istanza di LinuxRoute (che ora è *r<sub>𝛽</sub>*) per assegnarsi i relativi indirizzi IP.  
    In realtà, essendo *i<sub>0</sub>* una identità *di connettività*, secondo l'analisi non dovrà assegnarsi nessun indirizzo IP.

Successivamente — ma in tempi molto rapidi perché il modulo QSPN prevede un tempo massimo
di rilevamento dell'arco, che è fissato dal programma *qspnclient* a 10 secondi — sulla console del nodo
B il programma *qspnclient* deve essere istruito dall'utente ad aggiungere un IQspnArc al suo QspnManager
con il metodo *arc_add*. Le informazioni da dare (nel comando interattivo *add_qspnarc*) per questo sono:

*   `int nodeid_index`. L'indice del NodeID di B. Per recuperare l'istanza di QspnManager su cui
    operare. In questo esempio l'indice 0.
*   `int idarc_index`. L'indice dell'arco-identità sul quale va costruito il IQspnArc. In questo esempio 1.
*   `string idarc_address`. L'indirizzo Netsukuku del vicino collegato da questo arco. In questo esempio la stringa "3.10.67.89".

* * *

Quando vuole costruire un IQspnArc da fornire ad una istanza di QspnManager, il programma *qspnclient*
crea una istanza della classe ProofOfConcept.QspnArc. Essa ha questi membri:

*   `ProofOfConcept.Arc arc`. L'istanza di arco su cui si poggia l'arco-identità.
*   `NodeID sourceid`. L'identificativo dell'identità nel sistema vicino.
*   `NodeID destid`. L'identificativo dell'identità nel proprio sistema.
*   `ProofOfConcept.Naddr neighbour_naddr`. L'indirizzo Netsukuku del vicino, rappresentato da una classe
    che implementa IQspnNaddr.

I primi 3 dati sono inclusi nell'istanza di arco-identità per la quale il programma intende
costruire un QspnArc. L'indirizzo Netsukuku del vicino, invece, viene fornito dall'utente che
fa le veci di un diverso modulo che si occupa di reperirlo dialogando con l'identità nel sistema
vicino.

I dati che servono per implementare i metodi di IQspnArc sono:

*   *i_qspn_get_cost()* – `arc.cost`.
*   *i_qspn_get_naddr()* – `neighbour_naddr`.
*   *i_qspn_equals(IQspnArc other)* – Si usa l'uguaglianza dell'istanza.
*   *i_qspn_comes_from(CallerInfo rpc_caller)* – `arc.neighborhood_arc.neighbour_nic_addr` deve essere equivalente
    all'indirizzo IP riportato come `peer_address` dall'istanza di CallerInfo `rpc_caller`.

Quando una istanza di QspnArc viene passata alla QspnStubFactory i suoi dati sono sufficienti
a realizzare uno stub per un modulo di identità con i metodi `get_stub_identity_aware_*` di
NeigborhoodManager.

* * *

Quando l'utente termina l'applicazione con il comando `quit` (o con Ctrl-C) il programma *qspnclient* istruisce
l'istanza di IdentityManager di rimuovere tutte le identità di connettività. In questo modo tutte le
pseudo-interfacce e tutti i network namespace creati dal programma vengono rimossi.

Per pulire il network namespace default, invece, il programma chiama sull'istanza di LinuxRoute relativa
dapprima il metodo `stop_management`. Con questo rimuove le tabelle (`ntk` e le varie `ntk_from_XXX`) che
aveva creato e popolato.

Poi, per rimuovere gli indirizzi IP propri, il programma chiama sull'istanza di LinuxRoute relativa
il metodo `remove_address` una volta per ogni indirizzo IP e per ognuna delle sue interfacce di rete reali.

## Annotazioni sulla rimozione degli archi

### Comportamento dei moduli

Il modulo Neighborhood chiama *remove_my_arc* (che è anche public) se il monitor di un arco rileva
un fallimento. Il metodo notifica il segnale `arc_removed`. In questo caso la chiamata è fatta con
`do_tell=false`, quindi il metodo non tenta di comunicare con il sistema vicino.

Il modulo Neighborhood chiama *remove_my_arc* anche quando una interfaccia di rete non va più
gestita. In questo caso fa la chiamata su tutti gli archi che partono da quella interfaccia di rete.
Il metodo notifica il segnale `arc_removed`. In questo caso la chiamata è fatta con `do_tell=true`, quindi
il metodo tenta di chiamare il metodo remoto *remove_arc*, che a sua volta nel sistema vicino richiama *remove_my_arc*.

Il modulo Qspn chiama *arc_remove* (che è anche public) quando una comunicazione fallisce su un arco-identità.
Oltre a chiamare il metodo, il modulo in queste occasioni notifica il segnale `arc_removed` includendo
in questo segnale anche un booleano `bad_link` che dice se la ragione è un link malfunzionante.

Il modulo Qspn chiama *arc_remove* anche quando è stato chiamato il suo metodo *remove_outer_arcs*. Anche
in questo caso il modulo notifica il segnale `arc_removed`, ovviamente con `bad_link=false`.

Se viene chiamato dall'esterno il metodo *arc_remove* di Qspn, il segnale `arc_removed` non viene emesso.  
Ad esempio, supponiamo che su un arco *a* che parte dal nostro sistema siano stati realizzati diversi
archi-identità (*ai<sub>1</sub>* e *ai<sub>2</sub>*) per via di diverse identità (*i<sub>1</sub>* e *i<sub>2</sub>*) che
vivono nel nostro sistema. Supponiamo che l'arco *a* diventi inutilizzabile, e che l'istanza di
QspnManager della nostra identità *i<sub>1</sub>* se ne avvede per prima. Questa istanza di QspnManager
chiama internamente il metodo *arc_remove* e notifica il segnale `arc_removed(ai1, bad_link=true)`.  
Come conseguenza il programma *qspnclient* rimuove l'arco *a* dal modulo Neighborhood con il suo
metodo *remove_my_arc*. Il modulo Neighborhood emette il segnale `arc_removed`.  
Come conseguenza il programma *qspnclient* rimuove l'arco *a* dal modulo Identities con il suo metodo *remove_arc*. Il modulo
Identities rimuove gli archi-identità *ai<sub>1</sub>* e *ai<sub>2</sub>* emettendo i segnali `identity_arc_removed`
relativi.  
Come conseguenza del segnale `identity_arc_removed(a, i1, ...)` il programma *qspnclient* dovrebbe
rimuovere *ai<sub>1</sub>* dal QspnManager di *i<sub>1</sub>*; ma siccome sa che è già stato rimosso
proprio su sua iniziativa, non fa nulla.  
Come conseguenza del segnale `identity_arc_removed(a, i2, ...)` il programma *qspnclient*
rimuove *ai<sub>2</sub>* dal QspnManager di *i<sub>2</sub>* chiamando dall'esterno il suo
metodo *arc_remove*. Questa chiamata, come abbiamo detto, non comporta una ulteriore notifica
del segnale `arc_removed`.

Il modulo Identities chiama *remove_arc* (che è anche public) quando una comunicazione sull'arco fallisce
durante il metodo *add_identity* o lo stesso metodo *add_arc*. Questo metodo rimuove di conseguenza tutti
gli archi-identità che vi si appoggiavano e notifica per essi il segnale `identity_arc_removed`. Dopo aver
chiamato il suo metodo *remove_arc*, il modulo in queste occasioni emette anche il segnale `arc_removed`.

Se viene chiamato dall'esterno il metodo *remove_arc* di Identities, il segnale `arc_removed` non viene emesso.  

Quando una identità di connettività viene rimossa con il metodo *remove_identity* di Identities, il modulo cerca
di notificarlo ai vicini con il metodo remoto *notify_identity_removed* e questo nei vicini fa rimuovere
l'arco-identità e avvia la notifica `identity_arc_removed`.

Quando l'identità principale viene rimossa (di fatto con la terminazione del modulo Identities) il modulo
Identities non cerca di fare questa notifica. Però questo avviene quando viene terminato anche il modulo
Neigborhood. In questa occasione il modulo Neighborhood rimuove tutti i suoi archi da tutte le sue schede
e cerca di notificarlo ai vicini con il metodo remoto *remove_arc*, che a sua volta nel sistema vicino
richiama *remove_my_arc*.

### Casi d'uso

Nel caso del programma *qspnclient*, che si presume verrà adoperato su un testbed di macchine
virtuali che non vedranno mai collegamenti malfunzionanti, possiamo fare un elenco delle situazioni
che si vogliono gestire.

*   L'utente da il comando `remove_nodearc` che deve simulare la rimozione di un arco, ad esempio
    perché i due sistemi non sono più a distanza di rilevamento con le loro schede wireless.  
    Per implementarlo in modo semplicistico si può chiamare il metodo *remove_my_arc* di Neighborhood.
*   In un sistema il qspnclient va in crash. Nei sistemi vicini viene prima o poi richiamato
    il metodo *remove_my_arc* di Neighborhood. Può anche verificarsi prima che viene chiamato il
    metodo *arc_remove* di Qspn. Oppure, anche se meno probabile, il metodo *remove_arc* di Identities.
*   L'utente da il comando `remove_outer_arcs` ad una identità di connettività. Sul QspnManager
    associato ad essa, il programma chiama il metodo *remove_outer_arcs*.
*   L'utente da il comando `remove_identity` riguardo una identità di connettività in quanto non serve
    più alla connettività. Sul modulo Identities, il programma chiama il metodo *remove_identity*.
*   Su una identità (qualsiasi) il Qspn riceve da remoto l'ordine di rimuovere un arco-identità.  
    **TODO** Sulla base di come si dipanano gli eventi nei due casi sopra (`remove_outer_arcs`
    e `remove_identity`) vedere quali comunicazioni riceve un sistema vicino.
*   L'utente da il comando `quit`.

Analiziamo una alla volta questi casi.

* * *

L'utente da il comando `remove_nodearc`. Il programma *qspnclient* richiama il metodo *remove_my_arc* di Neighborhood.
Il modulo Neighborhood rimuove l'arco da entrambi i sistemi e emette il segnale `arc_removed`.

Il programma *qspnclient* in risposta al segnale `arc_removed` di Neighborhood deve rimuovere
l'arco dal dizionario *neighborhood_arcs* e segnalarlo a video (indicando la chiave stringa
composta dai due MAC address).

Se in precedenza questo arco non era stato *accettato* con il comando interattivo `add_nodearc`,
le operazioni si concludono qui.

Altrimenti, il programma reperisce l'istanza di ProofOfConcept.Arc, la
rimuove dal dizionario *nodearcs* e lo segnala a video (indicando il *nodearc_index*).

Inoltre chiama il metodo *remove_arc* sul modulo Identities passandogli l'istanza di IIdmgmtArc
referenziata nell'istanza di ProofOfConcept.Arc. Questo a sua volta farà sì che il modulo Identities
attui delle operazioni riguardanti tutti gli archi-identità che si appoggiavano ad esso e in seguito
emetta, per ognuno, il segnale `identity_arc_removed`. Nella gestione di questo segnale, come vedremo dopo,
il programma *qspnclient* rimuove, se c'era, l'istanza di IQspnArc dal QspnManager di quella identità.

* * *

In un sistema il qspnclient va in crash. Un sistema vicino se ne può accorgere in diverse occasioni.
Supponiamo che se ne avvede il modulo Neighborhood per primo e quindi richiama il suo metodo
*remove_my_arc*.

In questo caso il modulo Neighborhood rimuove l'arco su questo sistema. Quindi emette il segnale
`arc_removed`. Tutto procede come abbiamo visto prima.

Supponiamo invece che se ne avvede per primo il modulo Qspn. Il modulo richiama il suo metodo
*arc_remove* e poi emette il segnale `arc_removed` specificando `bad_link=true`.

Il programma *qspnclient* in risposta al segnale `arc_removed(bad_link=true)` di Qspn deve
chiamare il metodo *remove_my_arc* di Neighborhood. Poi tutto procede come abbiamo visto prima.  
Bisogna però ricordare che quando parleremo della gestione del segnale `identity_arc_removed`
del modulo Identities occorre considerare che l'arco-identità in oggetto potrebbe essere già
stato rimosso dal modulo Qspn.

Supponiamo invece che se ne avvede per primo il modulo Identities. Il modulo richiama il suo
metodo *remove_arc*, di conseguenza rimuove tutti gli archi-identità che si appoggiavano ad
esso e emette, per ognuno, il segnale `identity_arc_removed`. Dopo aver chiamato il suo
metodo *remove_arc*, il modulo emette anche il segnale `arc_removed`.

Il programma *qspnclient* in risposta al segnale `identity_arc_removed` di Identities
rimuove l'istanza di IQspnArc dal QspnManager di quella identità.  
Poi, in risposta al segnale `arc_removed` di Identities, il programma chiama il metodo *remove_my_arc*
di Neighborhood. Poi tutto procede come abbiamo visto prima.

### Operazioni dell'utilizzatore dei moduli

Sul segnale `arc_removed` di una istanza di QspnManager viene collegato il metodo *arc_removed*
della relativa istanza di IdentityData.

Sul segnale `identity_arc_removed` di IdentityManager viene collegata la funzione *identity_arc_removed*.

Sul segnale `arc_removed` di NeighborhoodManager viene collegata la funzione *arc_removed*.

La funzione *remove_nodearc* viene chiamata dall'utente alla console.

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
*   **change_nodearc**
    *   `int nodearc_index`
    *   `int cost`
*   **remove_nodearc**
    *   `int nodearc_index`
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

## Esempio di migrazione/ingresso

Una identità detiene un particolare indirizzo Netsukuku e mantiene una mappa di percorsi verso
destinazioni note che sono in effetti *coordinate gerarchiche* basate su di esso.

Con queste conoscenze, una identità gestisce un particolare network namespace. Cioè, in tale
network namespace si assegna un numero di indirizzi IP e imposta un numero di rotte verso
destinazioni che sono indirizzi IP in notazione CIDR.

Quando il programma crea una nuova identità *i<sub>1</sub>* basata sulla precedente identità *i<sub>0</sub>*
avviene che l'identità *i<sub>0</sub>* vede cambiare il proprio indirizzo Netsukuku. Infatti diventa
una identità *di connettività* per un livello in cui prima non lo era. Inoltre la
nuova identità *i<sub>1</sub>* deterrà un nuovo indirizzo Netsukuku ancora diverso.

Inoltre ancora, l'identità *i<sub>1</sub>* avrà in eredità un network namespace *r<sub>𝛼</sub>* preesistente
che era stato configurato sulla base delle passate conoscenze di *i<sub>0</sub>*. Alcune di
queste configurazioni vanno subito rimosse da *r<sub>𝛼</sub>*, altre invece devono rimanere perché
alcune conoscenze di *i<sub>0</sub>* vanno copiate su *i<sub>1</sub>*. In seguito tramite il QSPN
l'identità *i<sub>1</sub>* acquisirà nuove conoscenze e queste produrranno nuove configurazioni
che andranno aggiunte a *r<sub>𝛼</sub>*.

Invece l'identità *i<sub>0</sub>* avrà assegnato un nuovo network namespace *r<sub>𝛽</sub>*. Alcune
delle conoscenze di *i<sub>0</sub>* vengono annullate, altre rimangono valide. Quelle che rimangono valide
vanno subito aggiunte come configurazioni a *r<sub>𝛽</sub>*. In seguito tramite il QSPN
l'identità *i<sub>0</sub>* acquisirà nuove conoscenze e queste produrranno nuove configurazioni
che andranno aggiunte a *r<sub>𝛽</sub>*.

Ad esempio possiamo avere in un *sistema* *n* l'identità *n<sub>0</sub>* con indirizzo 3·2·3·1 in una topologia 4·4·4·4.
Questo è un indirizzo *reale* quindi si tratta di una identità principale nel network namespace default (indichiamo
la relativa istanza di LinuxRoute con *r<sub>𝛼</sub>*).  
Ora supponiamo che il nodo 3·2·3·1 vuole migrare in 3·2·2·2, restando nel g-nodo 3·2·3
con l'identificativo *virtuale* 3·2·3·6.  
Quindi dentro *n* si aggiunge l'identità *n<sub>1</sub>* basata su *n<sub>0</sub>* con indirizzo 3·2·2·2  (che nasce come
identità principale con un indirizzo Netsukuku *reale* ed eredita il network namespace default)
mentre l'identità *n<sub>0</sub>* diventa *di connettività* al livello 1 con indirizzo 3·2·3·6
e gli viene associato il nuovo namespace *r<sub>𝛽</sub>*.

Poi supponiamo che il g-nodo 3·2 migra in 1·0, restando dentro il g-nodo 3 con l'identificativo *virtuale* 3·5. Dentro il
g-nodo 3·2 abbiamo sia il nodo 3·2·2·2 (cioè l'identità *n<sub>1</sub>* dentro *n*) sia il
nodo 3·2·3·6 (cioè l'identità *n<sub>0</sub>* dentro *n*).  
Quindi dentro *n* si aggiunge l'identità *n<sub>2</sub>* basata su *n<sub>0</sub>* con indirizzo 1·0·3·6 (che nasce come
identità *di connettività* al livello 1 ed eredita il network namespace *r<sub>𝛽</sub>*) mentre l'identità *n<sub>0</sub>* diventa
*di connettività* al livello 3 con indirizzo 3·5·3·6 e gli viene associato il nuovo namespace *r<sub>𝛾</sub>*.  
Inoltre dentro *n* si aggiunge l'identità *n<sub>3</sub>* basata su *n<sub>1</sub>* con indirizzo 1·0·2·2 (che nasce come
identità principale con un indirizzo Netsukuku *reale* ed eredita il network namespace default) mentre l'identità *n<sub>1</sub>* diventa
*di connettività* al livello 3 con indirizzo 3·5·2·2 e gli viene associato il nuovo namespace *r<sub>𝛿</sub>*.

Esaminiamo cosa avviene nella prima migrazione, quando si aggiunge l'identità *n<sub>1</sub>* basata su *n<sub>0</sub>*. In questo
caso il livello del g-nodo che migra, `hooking_gnode_level`, è 0. Il programma chiede al precedente QspnManager (quello associato
a *n<sub>0</sub>*) quali destinazioni conosce, a tutti i livelli. Ad esempio diciamo che erano noti
alcuni percorsi verso la destinazione 3·2·3·3, cioè in coordinate gerarchiche (0,3) rispetto a 3·2·3·1.
Ora il programma calcola per questa destinazione l'indirizzo IP globale, quello anonimizzante e quelli
interni al livello *k* con *k* > 0. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_destination` di *r<sub>𝛼</sub>*. In conclusione, non rimane nessuna rotta in *r<sub>𝛼</sub>*.

Ora ricordiamo che il livello del g-nodo in cui si entra, `into_gnode_level`, è 1.
Il programma, relativamente all'indirizzo proprio 3·2·3·1, calcola l'indirizzo IP globale,
quello anonimizzante solo se il *sistema* *n* intendeva rispondere a richieste anonime, e quelli
interni al livello *k* con *k* > 0. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_address` di *r<sub>𝛼</sub>*. In conclusione, non rimane nessun indirizzo proprio in *r<sub>𝛼</sub>*.

Da adesso *r<sub>𝛼</sub>* sarà assegnato a *n<sub>1</sub>* mentre *r<sub>𝛽</sub>* sarà assegnato a *n<sub>0</sub>*.

L'identità *n<sub>0</sub>* ora detiene un nuovo indirizzo Netsukuku per il quale nessun indirizzo
IP proprio si può computare, essendo divenuta una identità *di connettività*. Quindi a questo
proposito nessuna configurazione va aggiunta a *r<sub>𝛽</sub>*.

Le vecchie conoscenze di *n<sub>0</sub>* rimangono tuttavia valide. Riprendiamo ad esempio i
percorsi che erano noti verso la destinazione 3·2·3·3, cioè in coordinate gerarchiche (0,3) rispetto a 3·2·3·6.
Ora queste configurazioni vanno aggiunte a *r<sub>𝛽</sub>*. Il programma guarda innanzitutto se
il QspnManager associato a *n<sub>0</sub>* ha completato il bootstrap (altrimenti non aveva
impostato ancora nessuna rotta in *r<sub>𝛼</sub>* e nessuna rotta va ancora impostata nemmeno
in *r<sub>𝛽</sub>*). Se il bootstrap è stato completato, il programma chiede quali sono le
destinazioni note e opera quanto è necessario per impostare le rotte relative in *r<sub>𝛽</sub>*.

Ora vediamo cosa avviene nella seconda migrazione. Prima osserviamo quando si aggiunge l'identità
*n<sub>3</sub>* basata su *n<sub>1</sub>*, in quanto questo riguarda il network namespace default
ed è la parte più complessa. In questo caso il livello del g-nodo che migra, `hooking_gnode_level`, è 2.
Il programma chiede al precedente QspnManager (quello associato a *n<sub>1</sub>*) quali
destinazioni conosce, a tutti i livelli. Ad esempio diciamo che erano noti alcuni percorsi
verso la destinazione 3·2·3·3, cioè in coordinate gerarchiche (1,3) rispetto a 3·2·2·2.
Ora il programma calcola per questa destinazione l'indirizzo IP globale, quello anonimizzante e quelli
interni al livello *k* con *k* > 2. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_destination` di *r<sub>𝛼</sub>*. In conclusione, in *r<sub>𝛼</sub>* rimangono quelle
rotte verso (1,3) che usano l'indirizzo IP interno al livello 2. Questo è corretto, poiché nella
migrazione è coinvolto tutto il g-nodo 3·2.

Ora ricordiamo che il livello del g-nodo in cui si entra, `into_gnode_level`, è 3.
Il programma, relativamente all'indirizzo proprio 3·2·2·2, calcola l'indirizzo IP globale,
quello anonimizzante solo se il *sistema* *n* intendeva rispondere a richieste anonime, e quelli
interni al livello *k* con *k* > 2. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_address` di *r<sub>𝛼</sub>*. In conclusione, rimane in *r<sub>𝛼</sub>* l'indirizzo IP
proprio interno al livello 2 e quello interno al livello 1. Questo è corretto e necessario,
poiché abbiamo mantenuto in *r<sub>𝛼</sub>* alcune rotte che hanno questi indirizzi IP propri
come *src* preferito.

Da adesso *r<sub>𝛼</sub>* sarà assegnato a *n<sub>3</sub>* mentre *r<sub>𝛿</sub>* sarà assegnato a *n<sub>1</sub>*.

L'identità *n<sub>1</sub>* ora detiene un nuovo indirizzo Netsukuku per il quale nessun indirizzo
IP proprio si può computare, essendo divenuta una identità *di connettività*. Quindi a questo
proposito nessuna configurazione va aggiunta a *r<sub>𝛿</sub>*.

Le vecchie conoscenze di *n<sub>1</sub>* rimangono tuttavia valide.
Ora queste configurazioni vanno aggiunte a *r<sub>𝛿</sub>*. Il programma guarda innanzitutto se
il QspnManager associato a *n<sub>1</sub>* ha completato il bootstrap (altrimenti non aveva
impostato ancora nessuna rotta in *r<sub>𝛼</sub>* e nessuna rotta va ancora impostata nemmeno
in *r<sub>𝛿</sub>*). Se il bootstrap è stato completato, il programma chiede quali sono le
destinazioni note e opera quanto è necessario per impostare le rotte relative in *r<sub>𝛿</sub>*.

Infine osserviamo cosa avviene quando si aggiunge l'identità *n<sub>2</sub>* basata su *n<sub>0</sub>*. In questo
caso il livello del g-nodo che migra, `hooking_gnode_level`, è 2. Il programma chiede al precedente QspnManager (quello associato
a *n<sub>0</sub>*) quali destinazioni conosce, a tutti i livelli. Ad esempio diciamo che erano noti
alcuni percorsi verso la destinazione 3·2·3·3, cioè in coordinate gerarchiche (0,3) rispetto a 3·2·3·6.
Ora il programma calcola per questa destinazione l'indirizzo IP globale, quello anonimizzante e quelli
interni al livello *k* con *k* > 2. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_destination` di *r<sub>𝛽</sub>*. In conclusione, in *r<sub>𝛽</sub>* rimangono quelle
rotte verso (0,3) che usano l'indirizzo IP interno al livello 1 e l'indirizzo IP interno al
livello 2. Questo è corretto, poiché nella migrazione è coinvolto tutto il g-nodo 3·2.

Ora ricordiamo che il livello del g-nodo in cui si entra, `into_gnode_level`, è 3.
Il programma, relativamente all'indirizzo proprio 3·2·3·6, calcola l'indirizzo IP globale,
quello anonimizzante solo se il *sistema* *n* intendeva rispondere a richieste anonime, e quelli
interni al livello *k* con *k* > 2. In questo caso nessuno di questi indirizzi IP si può computare
in quanto la precedente identità *n<sub>0</sub>* era già una identità *di connettività*. Quindi
non andrà mai chiamato il metodo `remove_address` di *r<sub>𝛽</sub>*.

Da adesso *r<sub>𝛽</sub>* sarà assegnato a *n<sub>2</sub>* mentre *r<sub>𝛾</sub>* sarà assegnato a *n<sub>0</sub>*.

L'identità *n<sub>0</sub>* ora detiene un nuovo indirizzo Netsukuku per il quale nessun indirizzo
IP proprio si può computare, essendo divenuta una identità *di connettività*. Quindi a questo
proposito nessuna configurazione va aggiunta a *r<sub>𝛾</sub>*.

Le vecchie conoscenze di *n<sub>0</sub>* rimangono tuttavia valide.
Ora queste configurazioni vanno aggiunte a *r<sub>𝛾</sub>*. Il programma guarda innanzitutto se
il QspnManager associato a *n<sub>0</sub>* ha completato il bootstrap (altrimenti non aveva
impostato ancora nessuna rotta in *r<sub>𝛽</sub>* e nessuna rotta va ancora impostata nemmeno
in *r<sub>𝛾</sub>*). Se il bootstrap è stato completato, il programma chiede quali sono le
destinazioni note e opera quanto è necessario per impostare le rotte relative in *r<sub>𝛾</sub>*.

## Metodi di LinuxRoute

Una istanza di LinuxRoute viene creata per ogni network namespace. Cioè, la prima per gestire
il network namespace default, e in seguito una per ogni nuovo network namespace che viene
creato.

*   **costruttore**  
    Argomenti:

    *   `string ns`  
        Il nome del network namespace gestito attraverso questa nuova istanza.
    *   `string whole_network`  
        Il range di indirizzi IP (in notazione CIDR) riservato per la rete Netsukuku
        in base alla topologia scelta.

*   **add_address**  
    Argomenti:

    *   `string address`  
        Indirizzo IP da assegnare.
    *   `string dev`  
        Interfaccia (o pseudo) di rete gestita.

    Questo metodo **TODO**...

*   **remove_address**  
    Argomenti:

    *   `string address`  
        Indirizzo IP da assegnare.
    *   `string dev`  
        Interfaccia (o pseudo) di rete gestita.

    Questo metodo **TODO**...

*   **add_destination**  
    Argomenti:

    *   `string dest`  
        Indirizzo IP (in notazione CIDR) destinazione.

    Questo metodo **TODO**...

*   **remove_destination**  
    Argomenti:

    *   `string dest`  
        Indirizzo IP (in notazione CIDR) destinazione.

    Questo metodo **TODO**...

*   **change_best_path**  
    Argomenti:

    *   `string dest`  
        Indirizzo IP (in notazione CIDR) destinazione.
    *   `string? dev`  
        Interfaccia (o pseudo) di rete tramite cui raggiungere il gateway.  
        Vale `null` se questa destinazione è irraggiungibile.
    *   `string? gw`  
        Interfaccia (o pseudo) di rete tramite cui raggiungere il gateway.  
        Vale `null` se questa destinazione è irraggiungibile.
    *   `string? src`  
        Indirizzo IP proprio da usare come *src* preferito.  
        Viene ignorato se questa destinazione è irraggiungibile.  
        Vale `null` se con questa operazione si vuole solo impostare una rotta
        da usare per i pacchetti in *inoltro*.
    *   `string? neighbour_mac`  
        MAC address di provenienza.  
        Vale `null` se con questa operazione si vuole impostare la rotta principale
        da usare per i pacchetti in *partenza*.

    Questo metodo **TODO**...

*   **stop_management**  
    Argomenti: nessuno.

    Questo metodo **TODO**...

