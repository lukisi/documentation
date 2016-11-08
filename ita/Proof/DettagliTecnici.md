# Proof of concept - Dettagli Tecnici

1.  **TODO**
1.  **TOC**

Per evitare conflitti di nomi, il programma **qspnclient** definisce le proprie classi e funzioni
nel namespace `ProofOfConcept` mentre usa i diversi namespace `Netsukuku.*` per accedere alle
classi dei vari moduli. Nel namespace `Netsukuku` sono definite le classi fornite dalle
librerie *ntkd-common* e *ntkdrpc*.

### Interazione programma-utente

L'utente avvia il programma **qspnclient** su un sistema eseguendo su una shell il comando `qspnclient init`. In
questo momento fornisce alcuni dati iniziali come argomenti. Il programma
non restituisce il controllo della shell all'utente; la utilizza invece per visualizzare alcune informazioni
utili durante le sue operazioni.

Per le successive comunicazioni con il programma, l'utente dovrà aprire una nuova shell sul sistema e
da questa dare altri comandi (ad esempio `qspnclient prepare_enter_net_phase_1`, ...) e con essi altri dati. Questi comandi
e informazioni saranno comunicate al programma *qspnclient* già in esecuzione, poi il comando restituirà
all'utente la shell, eventualmente dopo aver visualizzato le informazioni pertinenti.

* * *

Il programma **qspnclient** riceve all'avvio dall'utente, con una serie di flag *-i*, l'elenco
delle interfacce di rete da gestire.

Questo elenco di nomi di interfacce viene immediatamente comunicato al modulo Neighborhood tramite una serie
di chiamate al metodo *start_monitor* di NeighborhoodManager, nello stesso ordine in cui è stato proposto dall'utente.

Durante l'esecuzione del metodo *start_monitor* il modulo Neighborhood, dopo aver chiamato
il metodo *add_address* di INeighborhoodIPRouteManager, emette il segnale *nic_address_set*. Quando
riceve questo segnale il programma **qspnclient** crea una istanza di ProofOfConcept.HandledNic che contiene:

*   `string dev`.
*   `string mac`.
*   `string linklocal`.

Poi aggiunge tale istanza alla lista *handlednics*.

In questo stesso momento i dati di questa istanza di ProofOfConcept.HandledNic vengono mostrati a video
con il relativo indice. In seguito l'utente può rivederli con il comando `show_handlednics`.

Poi il programma inizializza il modulo Identities (col costruttore di IdentityManager) passando l'elenco delle
interfacce di rete gestite (che sono in *handlednics*), ognuna col relativo MAC e indirizzo link-local.

Non prevediamo la possibilità che in seguito vengano ad aggiungersi (o rimuoversi) altre interfacce di
rete da gestire. Quindi non dovrebbe rilevarsi in seguito il segnale *nic_address_set* di INeighborhoodIPRouteManager,
che comunque viene ignorato dal programma.

Dopo aver costruito IdentityManager il programma si mette in ascolto dei suoi segnali *identity_arc_added*,
*changed* e *removed*.

* * *

Il programma *qspnclient* vuole tenere traccia delle identità che sono nel nodo. Lo fa tramite il
dizionario *local_identities*, in cui la chiave è l'indice autoincrementante *local_identity_nextindex* e il valore è
una istanza della classe *IdentityData*. Dentro questa classe il programma mantiene alcune informazioni, tra le quali:

*   `NodeID nodeid` - L'identificativo che il modulo Identities ha assegnato all'identità.
*   `my_addr` - L'indirizzo Netsukuku.
*   `my_fp` - Il fingerprint e le anzianità a livello 0.
*   `string network_stack` - Il nome del relativo network namespace.

Per valorizzare la prima istanza di IdentityData, nel dizionario *local_identities* con indice 0 e associata alla
prima identità del nodo, il programma chiama il metodo *get_main_id* di IdentityManager per
recuperare il NodeID che il modulo Identities ha assegnato alla prima identità. Questa prima
identità gestisce il network namespace default, quindi `network_stack = ""`.

* * *

Dopo aver recuperaro il NodeID della sua prima identità, il programma **qspnclient** crea la prima istanza di
QspnManager e la associa a questa identità con il metodo *set_identity_module* di IdentityManager usando il nome di modulo "qspn".

Questa prima istanza di QspnManager viene realizzata col costruttore *create_net*. Per avvalersi di questo
costruttore, al programma serve solo conoscere la topologia della rete e l'indirizzo Netsukuku del nodo. Queste
due informazioni sono passate al programma al momento del suo avvio. L'utente dà il comando `qspnclient init`
sulla riga di comando di una shell del sistema e specifica nei primi due argomenti successivi queste informazioni:

*   args\[2] = topologia. Ad esempio la stringa "4.16.256.256" indica una rete a 4 livelli, dove la gsize al livello 3
    è 4, ecc. ecc.
*   args\[3] = indirizzo. Ad esempio la stringa "3.10.123.45" indica un indirizzo Netsukuku valido nella suddetta topologia.

Per comodità, il programma **qspnclient** assume che la topologia di rete usata sia la stessa per tutti i
nodi che prendono parte alla simulazione. Questo significa che quando l'utente richiederà l'ingresso di un nodo
in una diversa rete (vedremo il comando `prepare_enter_net_phase_1`) la topologia non dovrà essere di nuovo specificata.

In questo momento, quando abbiamo inizializzato il modulo Qspn per la prima identità principale del sistema,
i dati di questa istanza di IdentityData `local_identities[0]` vengono mostrati a video
con il relativo indice. In seguito l'utente può rivederli con il comando `show_local_identities`.

* * *

Quando il programma riceve dal modulo Neighborhood la notifica che un arco fisico è stato realizzato, esso mostra
le informazioni (link-local e MAC dei due estremi) dell'arco a video. Inoltre mette la relativa istanza di
INeighborhoodArc nel dizionario *neighborhood_arcs* con una chiave stringa composta dai due MAC address.

In seguito l'utente può rivederli con il comando `show_neighborhood_arcs`.

Quando l'utente decide di accettare un arco lo comunica al programma **qspnclient** con il comando
`add_real_arc` e indica:

*   MAC della propria interfaccia.
*   MAC del vicino.
*   Costo in latenza in microsecondi.

Se vuole in seguito variare il costo
lo può fare con il comando `change_real_arc`. Se vuole in seguito simularne la completa rimozione
lo può fare con il comando `remove_real_arc`.  
L'utente quando dà uno di questi comandi ad un sistema dovrà in tempi rapidi dare il relativo omologo
comando al sistema all'altro capo.

Il programma **qspnclient** (quando l'utente accetta un arco fisico) crea una istanza della
classe ProofOfConcept.Arc che ha questi membri:

*   `int cost`.
*   `INeighborhoodArc neighborhood_arc`.
*   `ProofOfConcept.IdmgmtArc idmgmt_arc`.

Mette questa istanza di ProofOfConcept.Arc in un altro dizionario, *real_arcs*, usando sempre come
chiave la stringa composta dai due MAC address.

In questo stesso momento tutti questi dati sono mostrati a video. In seguito
l'utente può rivederli con il comando `show_real_arcs`.

Poi il programma, basandosi su questa istanza di ProofOfConcept.Arc, istanzia una classe
ProofOfConcept.IdmgmtArc che implementa IIdmgmtArc che andrà passata al modulo Identities nel
metodo *add_arc*. Al suo interno mantiene (in un riferimento weak per evitare il loop) l'istanza di ProofOfConcept.Arc.

I dati che servono per implementare i metodi di IIdmgmtArc sono:

*   *get_dev()* – `arc.neighborhood_arc.nic.dev`.
*   *get_peer_linklocal()* – `arc.neighborhood_arc.neighbour_nic_addr`.
*   *get_peer_mac()* – `arc.neighborhood_arc.neighbour_mac`.

Questa istanza di ProofOfConcept.IdmgmtArc viene anche salvata nel membro *idmgmt_arc* della stessa
istanza di ProofOfConcept.Arc. Così il programma in seguito partendo dalla chiave di *real_arcs* potrà recuperarla.

* * *

Quando riceve il segnale *identity_arc_added* di IdentityManager, il programma crea una istanza di
ProofOfConcept.IdentityArc che ha questi membri:

*   `IIdmgmtArc arc`. Si tratta di una istanza di ProofOfConcept.IdmgmtArc che ci permette di risalire
    al ProofOfConcept.Arc.
*   `NodeID id`. Si tratta di uno dei propri NodeID.
*   `IIdmgmtIdentityArc id_arc`.
*   `string peer_mac`. Memorizza l'attuale contenuto di `id_arc.get_peer_mac()`. Sarà utile quando
    in futuro questa istanza di IIdmgmtIdentityArc notificherà una variazione.
*   `string peer_linklocal`. Analogo per il contenuto di `id_arc.get_peer_linklocal()`.

Associa questa istanza di ProofOfConcept.IdentityArc al prossimo valore dell'indice autoincrementante
*identity_arc_nextindex*, nel dizionario *identity_arcs*.

In questo stesso momento tutti questi dati sono mostrati a video con il relativo indice. In seguito
l'utente può rivederli con il comando `show_identity_arcs`.

* * *

Vediamo quali passaggi portano due sistemi, *𝛼* e *𝛽*, a rilevare un arco tra di loro e uno dei due, *𝛽*,
a fare ingresso nella rete dell'altro.

Inizializzazione del sistema *𝛼*.

*   Nel sistema *𝛼* l'utente dà il comando `qspnclient init 4.2.2.2 1.0.0.1 -i eth1`. Nella shell in cui
    è stato dato il comando il controllo non ritorna all'utente. Chiamiamo questa shell *console di qspnclient*.
    Viene usata dal programma per mostrare all'utente alcune segnalazioni.
*   Nella console di qspnclient del sistema *𝛼* viene data questa segnalazione:  
```
handlednic #0: eth1 00:16:3E:EC:A3:E1 169.254.96.141
```
*   Nella console di qspnclient del sistema *𝛼* viene data questa segnalazione:  
```
local_identity #0: indirizzo 1.0.0.1, anzianità 0.0.0.0, namespace default
                   fp0: 56724331, net_fp: 56724331
```

Inizializzazione del sistema *𝛽*.

*   Nel sistema *𝛽* l'utente dà il comando `qspnclient init 4.2.2.2 3.1.0.1 -i eth1`.
*   Nella console di qspnclient del sistema *𝛽* viene data questa segnalazione:  
    `handlednic #0: eth1 00:16:3E:5B:78:D5 169.254.94.223`
*   Nella console di qspnclient del sistema *𝛽* viene data questa segnalazione:  
    `local_identity #0: indirizzo 3.1.0.1, anzianità 0.0.0.0, namespace default`  
    `                   fp0: 75809993, net_fp: 75809993`

Rilevamento arco, sua accettazione, formazione arco-identità.

*   Nella console di qspnclient del sistema *𝛼* viene data questa segnalazione:  
    `neighborhood_arc 00:16:3E:EC:A3:E1-00:16:3E:5B:78:D5 : linklocal 169.254.94.223, cost 934us`
*   Nella console di qspnclient del sistema *𝛽* viene data questa segnalazione:  
    `neighborhood_arc 00:16:3E:5B:78:D5-00:16:3E:EC:A3:E1 : linklocal 169.254.96.141, cost 581us`
*   Nel sistema *𝛼* l'utente dà il comando `qspnclient add_real_arc 00:16:3E:EC:A3:E1 00:16:3E:5B:78:D5 10000`.
*   Rapidamente, nel sistema *𝛽* l'utente dà il comando `qspnclient add_real_arc 00:16:3E:5B:78:D5 00:16:3E:EC:A3:E1 11000`.
*   Nella console di qspnclient del sistema *𝛼* viene data questa segnalazione:  
    `real_arc 00:16:3E:EC:A3:E1-00:16:3E:5B:78:D5 : peer_linklocal 169.254.94.223, cost 10000us`
*   Nella console di qspnclient del sistema *𝛽* viene data questa segnalazione:  
    `real_arc 00:16:3E:5B:78:D5-00:16:3E:EC:A3:E1 : peer_linklocal 169.254.96.141, cost 11000us`
*   Nella console di qspnclient del sistema *𝛼* viene data questa segnalazione:  
    `identity_arc #0: real_arc: 00:16:3E:EC:A3:E1-00:16:3E:5B:78:D5, local_identity: 0`  
    `                 peer_mac: 00:16:3E:5B:78:D5, peer_linklocal: 169.254.94.223`
*   Nella console di qspnclient del sistema *𝛽* viene data questa segnalazione:  
    `identity_arc #0: real_arc: 00:16:3E:5B:78:D5-00:16:3E:EC:A3:E1, local_identity: 0`  
    `                 peer_mac: 00:16:3E:EC:A3:E1, peer_linklocal: 169.254.96.141`

Ingresso.

*   L'utente decide che grazie a questo nuovo arco-identità il nodo 1·0·0·1 entrerà nella rete con cui
    si è incontrato.  
    Quando si fa un ingresso, a fare ingresso è sempre un g-nodo (in questo caso di livello 0)
    il cui indirizzo Netsukuku è completamente *reale*. Quando si tratta di un singolo
    nodo, quindi, è sempre la sua identità principale.  
    Quindi, in questo caso abbiamo che l'identità principale *𝛼<sub>0</sub>* diventa una identità
    di connettività, mentre la duplicata nuova identità *𝛼<sub>1</sub>* (quale unico componente
    del nuovo g-nodo che fa il suo ingresso nell'altra rete) diventa l'identità principale di *𝛼*.  
    Quindi l'utente decide che *𝛼<sub>1</sub>* entra nella rete del sistema *𝛽* (la rete che
    indichiamo con net_fp: 75809993).  
*   Più precisamente l'utente decide che *𝛼<sub>1</sub>*, essendo un g-nodo di livello 0,
    entra nel g-nodo di livello 1 di cui fa parte *𝛽<sub>0</sub>*, che ha un posto libero. Decide
    inoltre che *𝛼<sub>1</sub>*, dentro il g-nodo con indirizzo Netsukuku 3·1·0·, prenderà dapprima
    la posizione virtuale 2 con anzianità 1. Poi prenderà la posizione reale 0 con anzianità 2.  
    L'utente decide inoltre che l'identità di connettività *𝛼<sub>1</sub>*, dentro il g-nodo della
    vecchia rete con indirizzo Netsukuku 1·0·0·, prenderà ora la posizione virtuale 2 con anzianità 1.
*   L'utente valuta che non serve nessuna migration path. Prende a caso un identificativo per
    l'operazione di ingresso, assumiamo sia 13140402.
*   Oltre agli archi-identità interni al g-nodo che fa ingresso (in questo caso nessuno, essendo il
    g-nodo di livello 0) l'utente guarda quali altri archi-identità diverranno archi-qspn per le
    identità che faranno parte del nuovo g-nodo. In questo caso la sola identità che farà parte del
    nuovo g-nodo è *𝛼<sub>1</sub>*, che diventa l'identità principale nel sistema *𝛼*. Il suo
    arco-identità #0 (o meglio il duplicato di esso per *𝛼<sub>1</sub>*) sarà un suo nuovo arco-qspn
    nella rete. Dal punto di vista del sistema *𝛽*, l'arco-identità che sarà un suo nuovo arco-qspn
    è quello che avrà peer_MAC 00:16:3E:EC:A3:E1.
*   Nel sistema *𝛼* l'utente dà il comando `prepare_enter_net_phase_1` con questi dati:
    *   identità interessata = `0`
    *   livello del g-nodo entrante = `0`
    *   livello del g-nodo ospitante = `1`
    *   indirizzo del g-nodo ospitante = `3.1.0`
    *   anzianità/fingerprint del g-nodo ospitante = `??`
    *   posizione virtuale temporanea nel g-nodo ospitante = `2`
    *   anzianità della posizione virtuale temporanea nel g-nodo ospitante = `1`
    *   posizione reale nel g-nodo ospitante = `0`
    *   anzianità della posizione reale nel g-nodo ospitante = `2`
    *   posizione virtuale come g-nodo di connettività = `2`
    *   anzianità della posizione virtuale come g-nodo di connettività = `1`
    *   elenco identificativi degli archi-identità che saranno archi-qspn = `[0]`
    *   identificativo dell'operazione di ingresso = `13140402`
    *   identificativo dell'operazione di migrazione previa = `null`
*   Cioè, nel sistema *𝛼* l'utente dà il comando:  
    `qspnclient prepare_enter_net_phase_1 0 0 1 3.1.0 ?? 2 1 0 2 2 1 [0] 13140402 null`
*   Appena terminato il comando precedente, nel sistema *𝛼* l'utente dà il comando:  
    `qspnclient enter_net_phase_1 0 13140402`  
*   Dopo un attesa di un secondo circa, nel sistema *𝛽* l'utente dà il comando:  
    `qspnclient add_qspn_arc 0 00:16:3E:EC:A3:E1`  

### Vecchio

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
il nuovo network namespace. Ne consegue che l'istanza di NetworkStack, chiamiamola *ns<sub>𝛼</sub>*, che si trovava
memorizzata nell'istanza di IdentityData associata a *i<sub>0</sub>*, nel membro *network_stack*, viene ora memorizzata nella
nuova istanza di IdentityData associata a *i<sub>1</sub>*; nell'istanza di IdentityData associata
a *i<sub>0</sub>* verrà invece memorizzata una nuova istanza di NetworkStack, chiamiamola *ns<sub>𝛽</sub>*, creata per il nuovo
network namespace.

Alcune delle rotte impostate da *ns<sub>𝛼</sub>* nel suo network namespace dovranno presto essere
rimosse, mentre altre dovranno rimanere. Analogamente, alcuni indirizzi IP, che nel tempo sono stati impostati
da *ns<sub>𝛼</sub>* nel suo network namespace assegnandoli alle varie \[pseudo]interfacce gestite,
dovranno presto essere rimossi, mentre altri dovranno rimanere. Tutto questo deriva dal fatto che ora
quel network namespace è gestito da una diversa identità *i<sub>1</sub>*.  
Quando l'utente dà il comando interattivo *add_identity* non sappiamo ancora quale sia il g-nodo
che in blocco entra in una nuova rete oppure migra in un diverso g-nodo. In base al livello
del g-nodo che *si muove* e al livello del g-nodo in cui esso entra, alcuni indirizzi IP interni
e le relative rotte dovranno restare in esistenza nel network namespace gestito da *ns<sub>𝛼</sub>*
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
in una nuova rete. È il momento giusto per dare istruzioni all'istanza di NetworkStack *ns<sub>𝛼</sub>* (che ora è
associata all'identità *i<sub>1</sub>*) di rimuovere le rotte e gli indirizzi IP propri che non sono
più validi, lasciando in vigore le rotte (e relativi indirizzi propri) che sono validi in quanto
interni ad un livello inferiore a quello del g-nodo che migra.

Le rotte verso indirizzi IP globali e le rotte verso indirizzi IP interni ad un livello *k* tale
che *k* > `hooking_gnode_level` vanno rimosse. Per farlo il programma *qspnclient* fa diverse
chiamate al metodo `remove_destination` di *ns<sub>𝛼</sub>* sulla base dei percorsi che sono
noti al precedente QspnManager.

L'indirizzo IP proprio globale e quelli interni ad un livello *k* tale
che *k* > `into_gnode_level-1` vanno rimossi. Per farlo il programma *qspnclient* fa diverse
chiamate al metodo `remove_address` di *ns<sub>𝛼</sub>* sulla base dell'indirzzo Netsukuku che era
detenuto dalla precente identità (memorizzato nella classe IdentityData).

Subito dopo, il comando interattivo *enter_net* costruisce la nuova istanza di QspnManager. Poi,
per mezzo delle istanze di NetworkStack associate alle due identità esegue queste operazioni:

*   L'identità *i<sub>1</sub>* usa la sua istanza di NetworkStack (che ora è *ns<sub>𝛼</sub>*) per assegnarsi i relativi indirizzi IP.  
    Per l'esattezza, l'indirizzo IP globale e quelli interni ai livelli da `into_gnode_level` in su.
*   L'identità *i<sub>0</sub>* usa la sua istanza di NetworkStack (che ora è *ns<sub>𝛽</sub>*) per assegnarsi i relativi indirizzi IP.  
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
*   `string peer_mac`. L'indirizzo MAC della \[pseudo]interfaccia gestita dall'identità nel sistema vicino.  
    Utile per sapere come identificare un generico pacchetto IP da instradare che proviene da questo
    specifico arco-identità.

Tutti i dati sono inclusi nell'istanza di arco-identità per la quale il programma intende
costruire un QspnArc.

I dati che servono per implementare i metodi di IQspnArc sono:

*   *i_qspn_get_cost()* – `arc.cost`.
*   *i_qspn_equals(IQspnArc other)* – Si usa l'uguaglianza dell'istanza.
*   *i_qspn_comes_from(CallerInfo rpc_caller)* – `arc.neighborhood_arc.neighbour_nic_addr` deve essere equivalente
    all'indirizzo IP riportato come `peer_address` dall'istanza di CallerInfo `rpc_caller`.

Quando una istanza di QspnArc viene passata alla QspnStubFactory i suoi dati sono sufficienti
a realizzare uno stub per un modulo di identità con i metodi `get_stub_identity_aware_*` di
NeigborhoodManager.

* * *

Quando riceve il segnale *identity_arc_changed* di IdentityManager, il programma è in grado
di recuperare l'istanza di ProofOfConcept.IdentityArc che aveva creato per questo arco-identità.
Da tale vecchia istanza recupera i vecchi valori di *peer_mac* e *peer_linklocal*, prima di aggiornarli.

Inoltre è in grado di sapere se per tale arco-identità aveva in passato creato una istanza di QspnArc.
In caso affermativo, tutti questi dati servono al programma che deve fare queste operazioni:

*   Creare una nuova tabella (`ntk_from_newmac`) con questo nuovo neighbour come ingresso.
    Ricordiamo che l'aggiunta del neighour nella classe NetworkStack causa anche l'aggiunta
    delle rotte `unreachable` per tutte le destinazioni note.
*   Nella tabella principale (`ntk`) cambiare tutte le rotte che passano per questo gateway.
*   Nella tabella nuova (`ntk_from_newmac`) cambiare tutte le rotte che passano per questo gateway.
*   Rimuovere la tabella vecchia (`ntk_from_oldmac`). Forse riapparirà in seguito con una
    chiamata al metodo add_arc di Qspn.

* * *

Quando l'utente termina l'applicazione con il comando `quit` (o con Ctrl-C) il programma *qspnclient* istruisce
l'istanza di IdentityManager di rimuovere tutte le identità di connettività. In questo modo tutte le
pseudo-interfacce e tutti i network namespace creati dal programma vengono rimossi.

Per pulire il network namespace default, invece, il programma chiama sull'istanza di NetworkStack relativa
dapprima il metodo `stop_management`. Con questo rimuove le tabelle (`ntk` e le varie `ntk_from_XXX`) che
aveva creato e popolato.

Poi, per rimuovere gli indirizzi IP propri, il programma chiama sull'istanza di NetworkStack relativa
il metodo `remove_address` una volta per ogni indirizzo IP e per ognuna delle sue interfacce di rete reali.

## Annotazioni sulla rimozione degli archi

### Comportamento dei moduli

Il modulo Neighborhood chiama *remove_my_arc* (che è anche public) se il monitor di un arco rileva
un fallimento. In questo metodo il modulo notifica il segnale `arc_removing`; poi esegue la rimozione
dell'arco e della rotta nelle tabelle di routing; infine notifica il segnale `arc_removed`. In questo caso la chiamata è fatta con
`do_tell=false`, quindi il metodo non tenta di comunicare con il sistema vicino.

Il modulo Neighborhood chiama *remove_my_arc* anche quando una interfaccia di rete non va più
gestita. In questo caso fa la chiamata su tutti gli archi che partono da quella interfaccia di rete.
In questo caso la chiamata è fatta con `do_tell=true`, quindi il modulo tenta anche
di chiamare il metodo remoto *remove_arc*, che a sua volta nel sistema vicino richiama *remove_my_arc*.

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
In risposta al segnale `arc_removed(ai1, bad_link=true)` il programma *qspnclient* memorizza che
l'arco *ai<sub>1</sub>* è stato rimosso dal QspnManager di *i<sub>1</sub>*; poi rimuove l'arco *a* dal modulo
Neighborhood con il suo metodo *remove_my_arc* con `do_tell=false`. Il modulo Neighborhood, prima di rimuovere la
rotta verso il gateway, emette il segnale `arc_removing(a, is_still_usable=false)`.  
In risposta al segnale `arc_removing(a, is_still_usable=false)` il programma *qspnclient*
rimuove l'arco *a* dal modulo Identities con il suo metodo *remove_arc*. Il modulo
Identities rimuove gli archi-identità *ai<sub>1</sub>* e *ai<sub>2</sub>*, emettendo prima però i segnali
`identity_arc_removing` relativi.  
In risposta del segnale `identity_arc_removing(a, i1, peer_id_j)` il programma *qspnclient* dovrebbe
rimuovere *ai<sub>1</sub>* dal QspnManager di *i<sub>1</sub>*; ma siccome ha memoria che è già stato rimosso
proprio su iniziativa di quel QspnManager, non fa nulla.  
In risposta del segnale `identity_arc_removing(a, i2, peer_id_k)` il programma *qspnclient*
rimuove *ai<sub>2</sub>* dal QspnManager di *i<sub>2</sub>* chiamando dall'esterno il suo
metodo *arc_remove*. Questa chiamata, come abbiamo detto, non comporta una ulteriore notifica
del segnale `arc_removed`.

Il modulo Identities chiama *remove_arc* (che è anche public) quando una comunicazione sull'arco fallisce.  
Il metodo *remove_arc* rimuove tutti gli archi-identità che vi si appoggiavano. Per farlo, prima emette il
segnale `identity_arc_removing`, poi chiama il metodo *remove_identity_arc*
in cui si specifica `do_tell=false`. Il metodo *remove_identity_arc* in questo caso rimuove l'arco-identità
e lo notifica con il segnale `identity_arc_removed`, ma non tenta di comunicare l'avvenimento al sistema vicino.  
Dopo aver chiamato il suo metodo *remove_arc*, il modulo in queste occasioni emette anche il segnale `arc_removed`.

Se viene chiamato dall'esterno il metodo *remove_arc* di Identities, il segnale `arc_removed` non viene emesso.  

Quando una identità di connettività viene rimossa con il metodo *remove_identity* di Identities, il modulo
rimuove tutti gli archi-identità relativi a quella identità, ma senza chiamare il metodo *remove_identity_arc*.
Il metodo *remove_identity* tenta prima di notificare la rimozione di ognuno degli archi-identità ai vicini
con il metodo remoto *notify_identity_arc_removed* e questo nei sistemi vicini fa emettere il segnale `identity_arc_removing`,
poi fa rimuovere l'arco-identità (con una chiamata al metodo *remove_identity_arc* in cui si specifica `do_tell=false`) e fa
emettere il segnale `identity_arc_removed`. Invece nel sistema corrente non viene emesso il segnale `identity_arc_removed`.  
Questo meccanismo non viene attivato quando viene rimossa l'identità principale, cioè con la terminazione del
modulo Identities. Però questo avviene quando viene terminato anche il modulo
Neigborhood. In questa occasione il modulo Neighborhood rimuove tutti i suoi archi da tutte le sue schede
e cerca di notificarlo ai vicini con il metodo remoto *remove_arc*, che a sua volta nel sistema vicino
richiama *remove_my_arc*.

Quando l'utilizzatore del modulo Identities vuole rimuovere uno specifico arco-identità di sua
iniziativa (di norma questo avviene quando il modulo Qspn lo richiede come risultato dell'operazione
*remove_outer_arcs*) chiama il metodo *remove_identity_arc*, che di default ha `do_tell=true`.  
Il metodo *remove_identity_arc* in questo caso, dopo aver rimosso l'arco-identità e prima di emettere
il segnale `identity_arc_removed`, tenta di comunicare l'avvenimento al sistema vicino
con il metodo remoto *notify_identity_arc_removed* e questo nel sistema vicino fa emettere il segnale `identity_arc_removing`,
poi fa rimuovere l'arco-identità (con una chiamata al metodo *remove_identity_arc* in cui si specifica `do_tell=false`) e fa
emettere il segnale `identity_arc_removed`.

### Casi d'uso

Nel caso del programma *qspnclient*, che si presume verrà adoperato su un testbed di macchine
virtuali che non vedranno mai collegamenti malfunzionanti, possiamo fare un elenco delle situazioni
che si vogliono gestire.

*   L'utente dà il comando `remove_nodearc` che deve simulare la rimozione di un arco, ad esempio
    perché i due sistemi non sono più a distanza di rilevamento con le loro schede wireless.  
    Per implementarlo in modo semplicistico si può chiamare il metodo *remove_my_arc* di Neighborhood con
    `do_tell=false`.
*   In un sistema il qspnclient va in crash. Nei sistemi vicini viene prima o poi richiamato
    il metodo *remove_my_arc* di Neighborhood. Può anche verificarsi prima che viene chiamato il
    metodo *arc_remove* di Qspn. Oppure, anche se meno probabile, il metodo *remove_arc* di Identities.
*   L'utente dà il comando `remove_outer_arcs` ad una identità di connettività. Il programma *qspnclient*
    chiama il metodo *remove_outer_arcs* sul QspnManager associato a questa identità.
*   L'utente dà il comando `remove_identity` riguardo una identità di connettività in quanto non serve
    più alla connettività. Il programma *qspnclient* chiama il metodo *remove_identity* sul modulo Identities
    per rimuovere questa identità.
*   Su una identità (qualsiasi) il modulo Identities riceve la chiamata al metodo remoto *notify_identity_arc_removed*.  
    Il modulo Identities emette il segnale `identity_arc_removing`, poi rimuove l'arco-identità, infine
    emette il segnale `identity_arc_removed`.
*   L'utente dà il comando `quit`.

Analiziamo una alla volta questi casi.

* * *

L'utente dà il comando `remove_nodearc`. Il programma *qspnclient* richiama il metodo *remove_my_arc* di Neighborhood con
`do_tell=false`. Il modulo Neighborhood prima di rimuovere l'arco dalle tabelle di routing emette
il segnale `arc_removing` con `is_still_usable=false` (come `do_tell`).

Il programma *qspnclient* in risposta al segnale `arc_removing` guarda se in precedenza questo arco
era stato *accettato* con il comando interattivo `add_nodearc`.

In caso affermativo, il programma reperisce l'istanza di ProofOfConcept.Arc.  
Poi chiama il metodo *remove_arc* sul modulo Identities passandogli l'istanza di IIdmgmtArc
referenziata nell'istanza di ProofOfConcept.Arc.  
Nel suo metodo *remove_arc* (che non prevede un booleano `do_tell` o `is_still_usable`) il modulo Identities
rimuove tutti gli archi-identità che si appoggiavano ad esso: dapprima il modulo emette per ognuno il segnale
`identity_arc_removing`; poi il modulo (chiamando il suo metodo *remove_identity_arc* con `do_tell=false`)
rimuove gli archi-identità anche con delle operazioni sulle tabelle di routing; infine il modulo emette
il segnale `identity_arc_removed`.  
Nella gestione del segnale `identity_arc_removing`, il programma *qspnclient*, prima che il modulo
Identities provveda a rimuovere l'arco verso il gateway dalla tabella di routing, verifica se fra
le istanze di IQspnArc comunicate al modulo Qspn vi è questo arco-identità. In questo caso
chiama il metodo *arc_remove* del modulo Qspn. Questa chiamata fa in modo che siano rimosse
eventuali rotte memorizzate che facevano uso del gateway. Questa chiamata non produce l'emissione del
segnale `arc_removed` dal modulo Qspn.  
Infine il programma *qspnclient* rimuove l'istanza di ProofOfConcept.Arc dal dizionario *nodearcs* e lo
segnala a video (indicando il *nodearc_index*).

In caso negativo tutto questo non viene fatto.

Al termine della gestione del segnale `arc_removing`, il metodo *remove_my_arc* di Neighborhood
rimuove l'arco (anche dalle tabelle di routing) e emette il segnale `arc_removed`.

Il programma *qspnclient* in risposta al segnale `arc_removed` di Neighborhood deve rimuovere
l'arco dal dizionario *neighborhood_arcs* e segnalarlo a video (indicando la chiave stringa
composta dai due MAC address).

Siccome questa era solo una simulazione di un arco che non funzionava più, il programma *qspnclient*
a breve rileverà di nuovo l'arco, ma all'utente basta non accettarlo di nuovo.

* * *

Nell'esempio di prima, la simulazione di un arco che diventa inefficace, il sistema *a* non fa nessuna
notifica al sistema vicino *b*. Quindi *b* non si avvede subito di questa variazione, ma se ne
avvede dopo un po' di tempo in autonomia. Avviene lo stesso di quello che sarebbe accaduto se nel
sistema *a* il programma qspnclient fosse andato in crash. Lo vediamo nel prossimo esempio.

* * *

In un sistema il qspnclient va in crash. Un sistema vicino se ne può accorgere in diverse occasioni.
Supponiamo che se ne avvede il modulo Neighborhood per primo e quindi richiama il suo metodo
*remove_my_arc* con `do_tell=false`. Tutto procede come abbiamo visto prima.

Supponiamo invece che se ne avvede per primo il modulo Qspn. Il modulo richiama il suo metodo
*arc_remove* e poi emette il segnale `arc_removed` specificando `bad_link=true`.

Il programma *qspnclient* in risposta al segnale `arc_removed(bad_link=true)` di Qspn deve
chiamare il metodo *remove_my_arc* di Neighborhood con `do_tell=false`. Poi tutto procede come abbiamo visto prima.  
Bisogna però ricordare che, come abbiamo detto sulla gestione del segnale `identity_arc_removing`
del modulo Identities, occorre considerare che l'arco-identità in oggetto potrebbe essere già
stato rimosso dal modulo Qspn.

Supponiamo invece che se ne avvede per primo il modulo Identities. Il modulo richiama il suo metodo *remove_arc*, nel quale
rimuove tutti gli archi-identità che si appoggiavano ad esso: dapprima il modulo emette per ognuno il segnale
`identity_arc_removing`; poi il modulo (chiamando il suo metodo *remove_identity_arc* con `do_tell=false`)
rimuove gli archi-identità anche con delle operazioni sulle tabelle di routing; infine il modulo emette
il segnale `identity_arc_removed`.  
Nella gestione del segnale `identity_arc_removing`, come detto sopra, il programma *qspnclient* rimuove, se c'era,
l'istanza di IQspnArc dal QspnManager di quella identità.  
Il modulo Identities, che si è avveduto dell'arco non funzionante, dopo aver chiamato il suo
metodo *remove_arc* emette anche il segnale `arc_removed`.

Poi, in risposta al segnale `arc_removed` di Identities, il programma chiama il metodo *remove_my_arc*
di Neighborhood con `do_tell=false`. Tutto procede come prima.  
Questa volta però, nella gestione del segnale `arc_removing` il programma *qspnclient*, dopo aver
reperito l'istanza di ProofOfConcept.Arc, deve accorgersi/ricordarsi che non serve chiamare di nuovo
il metodo *remove_arc* sul modulo Identities. Dovrà solo rimuovere l'istanza di ProofOfConcept.Arc dal dizionario *nodearcs*.  
Come prima, il modulo Neighborhood rimuove l'arco (anche dalle tabelle di routing) e emette il segnale `arc_removed`.
Il programma *qspnclient* in risposta rimuove l'arco dal dizionario *neighborhood_arcs*.

* * *

L'utente dà il comando `remove_outer_arcs` specificando una identità di connettività *i<sub>1</sub>*. Il
programma *qspnclient* sulla relativa istanza di QspnManager richiama il metodo *remove_outer_arcs*.

Supponiamo che in tale metodo il modulo Qspn decida di rimuovere un arco *ai<sub>1</sub>*. Il modulo notifica il
segnale `arc_removed` per l'arco *ai<sub>1</sub>* con `bad_link=false`. In risposta il programma richiama
il metodo *remove_identity_arc* con `do_tell=true` sul modulo Identities. Qui il modulo Identities
rimuove gli archi-identità anche con delle operazioni sulle tabelle di routing; poi lo comunica al
sistema vicino con il metodo remoto *notify_identity_arc_removed*; infine il modulo emette
il segnale `identity_arc_removed`.

* * *

L'utente dà il comando `remove_identity` specificando una identità di connettività *i<sub>1</sub>* in quanto non serve
più alla connettività. Questa simulazione ripercorre i passi che saranno fatti dal demone *ntkd*: nel
caso di migrazione di un g-nodo e relativa formazione di un g-nodo di connettività, dovrà essere solo
un singolo *nodo del grafo* a verificare periodicamente se l'intero g-nodo di connettività può essere rimosso. Quindi
quando l'utente dà il comando `remove_identity` su un sistema l'operazione sarà portata avanti da
tutto il g-nodo di connettività.

Il programma *qspnclient* fa queste operazioni:

*   Se `connectivity_from_level` (che è un parametro di una identità di connettività che viene impostato
    nel momento in cui l'utente dà il comando `make_connectivity`) è maggiore di 1:
    *   Chiama il metodo *prepare_destroy* del QspnManager di *i<sub>1</sub>*.  
        Questo propaga l'informazione a tutti i membri del g-nodo che si sta rimuovendo.
    *   Aspetta 10 secondi.
*   Chiama il metodo *destroy* del QspnManager di *i<sub>1</sub>*.  
    Questo informa i vicini esterni al g-nodo che si sta rimuovendo.
*   Dismette il QspnManager di *i<sub>1</sub>*.  
    Il demone *ntkd* dovrebbe dismettere tutti i moduli di identità relativi alla stessa identità.  
    Va rimosso ogni riferimento alle istanze memorizzato nel modulo Identities con *unset_identity_module*.
*   Chiama sul modulo Identities il metodo *remove_identity*. 

Come conseguenza della chiamata *prepare_destroy*, su tutti i *nodi del grafo* membri del g-nodo che si sta rimuovendo,
ad eccezione della stessa identità di connettività nel sistema in cui l'utente ha dato il comando `remove_identity`, il
modulo Qspn emetterà tra circa 10 secondi il segnale `remove_identity`. In risposta, il programma *qspnclient* dovrà:

*   Chiamare il metodo *destroy* del QspnManager che ha emesso il segnale.
*   Dismettere il QspnManager che ha emesso il segnale.  
    Il demone *ntkd* dovrebbe dismettere tutti i moduli di identità relativi alla stessa identità.  
    Va rimosso ogni riferimento alle istanze memorizzato nel modulo Identities con *unset_identity_module*.
*   Chiamare sul modulo Identities il metodo *remove_identity*. 

Il modulo Identities nel metodo *remove_identity*, in ogni sistema in cui è stato chiamato, cerca di
comunicare ai sistemi vicini l'avvenimento con una chiamata al metodo remoto *notify_identity_arc_removed*
per ogni arco-identità. Poi provvede a rimuovere completamente le pseudo-interfacce di rete e il network namespace.

* * *

Su una identità (qualsiasi) il modulo Identities riceve la chiamata al metodo remoto *notify_identity_arc_removed*.

Il modulo Identities emette il segnale `identity_arc_removing`. Nella gestione del segnale `identity_arc_removing`,
come detto sopra, il programma *qspnclient* rimuove, se c'era, l'istanza di IQspnArc dal QspnManager di quella identità.

Poi il modulo Identities procede con la rimozione del gateway e infine emette il segnale `identity_arc_removed`.

* * *

Quando l'utente termina l'applicazione con il comando `quit` (o con Ctrl-C) il programma *qspnclient*,
come abbiamo già detto sopra, istruisce l'istanza di IdentityManager di rimuovere tutte le identità di connettività.
Questo non solo fa rimuovere tutte le pseudo-interfacce e tutti i network namespace creati dal programma,
ma inoltre notifica questa rimozione ai sistemi vicini, come visto quando abbiamo illustrato il
comportamento del metodo *remove_identity*.

Poi il programma istruisce l'istanza di NeighborhoodManager di cessare la gestione di tutte le interfacce di rete
nel network namespace default con il metodo *stop_monitor_all*. In questo metodo il modulo Neighborhood, come
visto quando abbiamo illustrato il comportamento dei moduli in questa occasione, chiama il suo
metodo *remove_my_arc* con `do_tell=true` per ogni arco che aveva realizzato. In questo modo
notifica questa rimozione ai sistemi vicini.

## Elenco comandi interattivi

*   **show_handlednics**
*   **show_nodeids**
*   **show_neighborhood_arcs**
*   **add_real_arc**
    *   `string key`  
        La chiave è composta dal MAC della mia interfaccia e il MAC dell'interfaccia del
        vicino, separati da un trattino. Come compare a video con il comando `show_neighborhood_arcs`.
    *   `int cost`
*   **change_real_arc**
    *   `string key`
    *   `int cost`
*   **remove_real_arc**
    *   `string key`
*   **show_real_arcs**
*   **show_identity_arcs**
*   **show_ntkaddress**
    *   `int nodeid_index`
*   **prepare_add_identity**
    *   `int migration_id`
    *   `int nodeid_index`
*   **add_identity**
    *   `int migration_id`
    *   `int nodeid_index`
*   **remove_identity**
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
*   **remove_outer_arcs**
    *   `int nodeid_index`

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

Inoltre ancora, l'identità *i<sub>1</sub>* avrà in eredità un network
namespace gestito con l'istanza di NetworkStack *ns<sub>𝛼</sub>* preesistente
che era stato configurato sulla base delle passate conoscenze di *i<sub>0</sub>*. Alcune di
queste configurazioni vanno subito rimosse da *ns<sub>𝛼</sub>*, altre invece devono rimanere perché
alcune conoscenze di *i<sub>0</sub>* vanno copiate su *i<sub>1</sub>*. In seguito tramite il QSPN
l'identità *i<sub>1</sub>* acquisirà nuove conoscenze e queste produrranno nuove configurazioni
che andranno aggiunte a *ns<sub>𝛼</sub>*.

Invece l'identità *i<sub>0</sub>* avrà assegnato un nuovo network namespace gestito con *ns<sub>𝛽</sub>*. Alcune
delle conoscenze di *i<sub>0</sub>* vengono annullate, altre rimangono valide. Quelle che rimangono valide
vanno subito aggiunte come configurazioni a *ns<sub>𝛽</sub>*. In seguito tramite il QSPN
l'identità *i<sub>0</sub>* acquisirà nuove conoscenze e queste produrranno nuove configurazioni
che andranno aggiunte a *ns<sub>𝛽</sub>*.

Ad esempio possiamo avere in un *sistema* *n* l'identità *n<sub>0</sub>* con indirizzo 3·2·3·1 in una topologia 4·4·4·4.
Questo è un indirizzo *reale* quindi si tratta di una identità principale nel network namespace default (indichiamo
la relativa istanza di NetworkStack con *ns<sub>𝛼</sub>*).  
Ora supponiamo che il nodo 3·2·3·1 vuole migrare in 3·2·2·2, restando nel g-nodo 3·2·3
con l'identificativo *virtuale* 3·2·3·6.  
Quindi dentro *n* si aggiunge l'identità *n<sub>1</sub>* basata su *n<sub>0</sub>* con indirizzo 3·2·2·2  (che nasce come
identità principale con un indirizzo Netsukuku *reale* ed eredita il network namespace default)
mentre l'identità *n<sub>0</sub>* diventa *di connettività* al livello 1 con indirizzo 3·2·3·6
e gli viene associato il nuovo namespace *ns<sub>𝛽</sub>*.

Poi supponiamo che il g-nodo 3·2 migra in 1·0, restando dentro il g-nodo 3 con l'identificativo *virtuale* 3·5. Dentro il
g-nodo 3·2 abbiamo sia il nodo 3·2·2·2 (cioè l'identità *n<sub>1</sub>* dentro *n*) sia il
nodo 3·2·3·6 (cioè l'identità *n<sub>0</sub>* dentro *n*).  
Quindi dentro *n* si aggiunge l'identità *n<sub>2</sub>* basata su *n<sub>0</sub>* con indirizzo 1·0·3·6 (che nasce come
identità *di connettività* al livello 1 ed eredita il network namespace *ns<sub>𝛽</sub>*) mentre l'identità *n<sub>0</sub>* diventa
*di connettività* al livello 3 con indirizzo 3·5·3·6 e gli viene associato il nuovo namespace *ns<sub>𝛾</sub>*.  
Inoltre dentro *n* si aggiunge l'identità *n<sub>3</sub>* basata su *n<sub>1</sub>* con indirizzo 1·0·2·2 (che nasce come
identità principale con un indirizzo Netsukuku *reale* ed eredita il network namespace default) mentre l'identità *n<sub>1</sub>* diventa
*di connettività* al livello 3 con indirizzo 3·5·2·2 e gli viene associato il nuovo namespace *ns<sub>𝛿</sub>*.

Esaminiamo cosa avviene nella prima migrazione, quando si aggiunge l'identità *n<sub>1</sub>* basata su *n<sub>0</sub>*. In questo
caso il livello del g-nodo che migra, `hooking_gnode_level`, è 0. Il programma chiede al precedente QspnManager (quello associato
a *n<sub>0</sub>*) quali destinazioni conosce, a tutti i livelli. Ad esempio diciamo che erano noti
alcuni percorsi verso la destinazione 3·2·3·3, cioè in coordinate gerarchiche (0,3) rispetto a 3·2·3·1.
Ora il programma calcola per questa destinazione l'indirizzo IP globale, quello anonimizzante e quelli
interni al livello *k* con *k* > 0. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_destination` di *ns<sub>𝛼</sub>*. In conclusione, non rimane nessuna rotta in *ns<sub>𝛼</sub>*.

Ora ricordiamo che il livello del g-nodo in cui si entra, `into_gnode_level`, è 1.
Il programma, relativamente all'indirizzo proprio 3·2·3·1, calcola l'indirizzo IP globale,
quello anonimizzante solo se il *sistema* *n* intendeva rispondere a richieste anonime, e quelli
interni al livello *k* con *k* > 0. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_address` di *ns<sub>𝛼</sub>*. In conclusione, non rimane nessun indirizzo proprio in *ns<sub>𝛼</sub>*.

Da adesso *ns<sub>𝛼</sub>* sarà assegnato a *n<sub>1</sub>* mentre *ns<sub>𝛽</sub>* sarà assegnato a *n<sub>0</sub>*.

L'identità *n<sub>0</sub>* ora detiene un nuovo indirizzo Netsukuku per il quale nessun indirizzo
IP proprio si può computare, essendo divenuta una identità *di connettività*. Quindi a questo
proposito nessuna configurazione va aggiunta a *ns<sub>𝛽</sub>*.

Le vecchie conoscenze di *n<sub>0</sub>* rimangono tuttavia valide. Riprendiamo ad esempio i
percorsi che erano noti verso la destinazione 3·2·3·3, cioè in coordinate gerarchiche (0,3) rispetto a 3·2·3·6.
Ora queste configurazioni vanno aggiunte a *ns<sub>𝛽</sub>*. Il programma guarda innanzitutto se
il QspnManager associato a *n<sub>0</sub>* ha completato il bootstrap (altrimenti non aveva
impostato ancora nessuna rotta in *ns<sub>𝛼</sub>* e nessuna rotta va ancora impostata nemmeno
in *ns<sub>𝛽</sub>*). Se il bootstrap è stato completato, il programma chiede quali sono le
destinazioni note e opera quanto è necessario per impostare le rotte relative in *ns<sub>𝛽</sub>*.

Ora vediamo cosa avviene nella seconda migrazione. Prima osserviamo quando si aggiunge l'identità
*n<sub>3</sub>* basata su *n<sub>1</sub>*, in quanto questo riguarda il network namespace default
ed è la parte più complessa. In questo caso il livello del g-nodo che migra, `hooking_gnode_level`, è 2.
Il programma chiede al precedente QspnManager (quello associato a *n<sub>1</sub>*) quali
destinazioni conosce, a tutti i livelli. Ad esempio diciamo che erano noti alcuni percorsi
verso la destinazione 3·2·3·3, cioè in coordinate gerarchiche (1,3) rispetto a 3·2·2·2.
Ora il programma calcola per questa destinazione l'indirizzo IP globale, quello anonimizzante e quelli
interni al livello *k* con *k* > 2. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_destination` di *ns<sub>𝛼</sub>*. In conclusione, in *ns<sub>𝛼</sub>* rimangono quelle
rotte verso (1,3) che usano l'indirizzo IP interno al livello 2. Questo è corretto, poiché nella
migrazione è coinvolto tutto il g-nodo 3·2.

Ora ricordiamo che il livello del g-nodo in cui si entra, `into_gnode_level`, è 3.
Il programma, relativamente all'indirizzo proprio 3·2·2·2, calcola l'indirizzo IP globale,
quello anonimizzante solo se il *sistema* *n* intendeva rispondere a richieste anonime, e quelli
interni al livello *k* con *k* > 2. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_address` di *ns<sub>𝛼</sub>*. In conclusione, rimane in *ns<sub>𝛼</sub>* l'indirizzo IP
proprio interno al livello 2 e quello interno al livello 1. Questo è corretto e necessario,
poiché abbiamo mantenuto in *ns<sub>𝛼</sub>* alcune rotte che hanno questi indirizzi IP propri
come *src* preferito.

Da adesso *ns<sub>𝛼</sub>* sarà assegnato a *n<sub>3</sub>* mentre *ns<sub>𝛿</sub>* sarà assegnato a *n<sub>1</sub>*.

L'identità *n<sub>1</sub>* ora detiene un nuovo indirizzo Netsukuku per il quale nessun indirizzo
IP proprio si può computare, essendo divenuta una identità *di connettività*. Quindi a questo
proposito nessuna configurazione va aggiunta a *ns<sub>𝛿</sub>*.

Le vecchie conoscenze di *n<sub>1</sub>* rimangono tuttavia valide.
Ora queste configurazioni vanno aggiunte a *ns<sub>𝛿</sub>*. Il programma guarda innanzitutto se
il QspnManager associato a *n<sub>1</sub>* ha completato il bootstrap (altrimenti non aveva
impostato ancora nessuna rotta in *ns<sub>𝛼</sub>* e nessuna rotta va ancora impostata nemmeno
in *ns<sub>𝛿</sub>*). Se il bootstrap è stato completato, il programma chiede quali sono le
destinazioni note e opera quanto è necessario per impostare le rotte relative in *ns<sub>𝛿</sub>*.

Infine osserviamo cosa avviene quando si aggiunge l'identità *n<sub>2</sub>* basata su *n<sub>0</sub>*. In questo
caso il livello del g-nodo che migra, `hooking_gnode_level`, è 2. Il programma chiede al precedente QspnManager (quello associato
a *n<sub>0</sub>*) quali destinazioni conosce, a tutti i livelli. Ad esempio diciamo che erano noti
alcuni percorsi verso la destinazione 3·2·3·3, cioè in coordinate gerarchiche (0,3) rispetto a 3·2·3·6.
Ora il programma calcola per questa destinazione l'indirizzo IP globale, quello anonimizzante e quelli
interni al livello *k* con *k* > 2. Per ognuno di questi indirizzi IP il programma chiama il metodo
`remove_destination` di *ns<sub>𝛽</sub>*. In conclusione, in *ns<sub>𝛽</sub>* rimangono quelle
rotte verso (0,3) che usano l'indirizzo IP interno al livello 1 e l'indirizzo IP interno al
livello 2. Questo è corretto, poiché nella migrazione è coinvolto tutto il g-nodo 3·2.

Ora ricordiamo che il livello del g-nodo in cui si entra, `into_gnode_level`, è 3.
Il programma, relativamente all'indirizzo proprio 3·2·3·6, calcola l'indirizzo IP globale,
quello anonimizzante solo se il *sistema* *n* intendeva rispondere a richieste anonime, e quelli
interni al livello *k* con *k* > 2. In questo caso nessuno di questi indirizzi IP si può computare
in quanto la precedente identità *n<sub>0</sub>* era già una identità *di connettività*. Quindi
non andrà mai chiamato il metodo `remove_address` di *ns<sub>𝛽</sub>*.

Da adesso *ns<sub>𝛽</sub>* sarà assegnato a *n<sub>2</sub>* mentre *ns<sub>𝛾</sub>* sarà assegnato a *n<sub>0</sub>*.

L'identità *n<sub>0</sub>* ora detiene un nuovo indirizzo Netsukuku per il quale nessun indirizzo
IP proprio si può computare, essendo divenuta una identità *di connettività*. Quindi a questo
proposito nessuna configurazione va aggiunta a *ns<sub>𝛾</sub>*.

Le vecchie conoscenze di *n<sub>0</sub>* rimangono tuttavia valide.
Ora queste configurazioni vanno aggiunte a *ns<sub>𝛾</sub>*. Il programma guarda innanzitutto se
il QspnManager associato a *n<sub>0</sub>* ha completato il bootstrap (altrimenti non aveva
impostato ancora nessuna rotta in *ns<sub>𝛽</sub>* e nessuna rotta va ancora impostata nemmeno
in *ns<sub>𝛾</sub>*). Se il bootstrap è stato completato, il programma chiede quali sono le
destinazioni note e opera quanto è necessario per impostare le rotte relative in *ns<sub>𝛾</sub>*.

## Classe NetworkStack

Una istanza di NetworkStack viene creata per ogni network namespace. Cioè, la prima per gestire
il network namespace default, e in seguito una per ogni nuovo network namespace che viene
creato.

I metodi di questa classe fanno delle operazioni che implicano comandi al sistema operativo
riguardanti un network stack. Questi metodi possono essere chiamati da diverse tasklet (thread)
e la loro esecuzione può comportare operazioni *bloccanti*, che cioè danno occasione al sistema
di schedulare altre tasklet concorrenti. Tuttavia queste operazioni devono essere completate
in sequenza. Per questo ogni metodo di NetworkStack fa in modo che le dovute operazioni di sistema
vengano eseguite da una unica tasklet (per ogni istanza) e aspetta il loro completamento prima
di ritornare al chiamante.

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

