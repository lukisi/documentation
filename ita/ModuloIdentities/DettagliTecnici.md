# Modulo Identities - Dettagli Tecnici

1.  [Requisiti](#Requisiti)
1.  [Deliverable](#Deliverable)
1.  [Dettagli](#Dettagli)
    1.  [Associazioni](#Associazioni)
    1.  [Duplicazione di una identità](#Duplicazione_di_una_identita)
    1.  [Aggiunta o rimozione di una interfaccia di rete reale gestita dal sistema](#Aggiunta_interfaccia)
    1.  [Aggiunta o rimozione di un arco](#Aggiunta_arco)
    1.  [Aggiunta o rimozione di un arco-identità su richiesta dell'utilizzatore](#Aggiunta_arco_identita)
    1.  [Rimozione di una identità di connettività](#Rimozione_identita_connettivita)

## <a name="Requisiti"></a>Requisiti

L'utilizzatore del modulo Identities per prima cosa inizializza il modulo richiamando il costruttore di IdentityManager.

Nel costruttore viene passata l'istanza di ITasklet per fornire l'implementazione del sistema di tasklet.

Inoltre vengono passati:

*   Elenco delle interfacce di rete reali gestite dal sistema, coi relativi MAC e indirizzi link-local.  
    Queste informazioni vengono passate con 3 liste di stinghe (dev, mac, linklocal) che devono avere
    uguale lunghezza. Possono anche essere tutte vuote.
*   Manager dei network namespace. Istanza di una classe che implementa IIdmgmtNetnsManager.
*   Factory per creare uno stub dato un arco. Istanza di una classe che implementa IIdmgmtStubFactory.

Durante il tempo l'utilizzatore potrà fornire al modulo altre informazioni con questi metodi:

*   `void add_handled_nic(string dev, string mac, string linklocal)`.  
    Aggiungere una interfaccia di rete reale gestita dal sistema.  
    Si vedano sotto i dettagli al relativo [paragrafo](#add_handled_nic).
*   `void remove_handled_nic(string dev)`.  
    Rimuovere una interfaccia di rete reale che non è più gestita dal sistema.  
    Si vedano sotto i dettagli al relativo [paragrafo](#remove_handled_nic).
*   `void add_arc(IIdmgmtArc arc)`.  
    Aggiungere un arco che il sistema ha realizzato.  
    Si vedano sotto i dettagli al relativo [paragrafo](#add_arc).
*   `void remove_arc(IIdmgmtArc arc)`.  
    Rimuovere un arco che non è più valido.   
    Si vedano sotto i dettagli al relativo [paragrafo](#remove_arc).

## <a name="Deliverable"></a>Deliverable

La prima identità del sistema (che è all'inizio la *principale*) viene creata dal modulo nel
costruttore di IdentityManager. Quando il costruttore ha finito, l'utilizzatore del modulo può
da subito invocare il metodo *get_main_id* di IdentityManager per ottenere il NodeID relativo a tale identità.

L'utilizzatore del modulo chiede alla istanza di IdentityManager di associare ad una sua
identità (individuata con il NodeID) un oggetto con un dato nome. La classe IdentityManager è
provvista dei seguenti metodi:

*   `void set_identity_module(NodeID id, string name, Object obj)`.  
    Associa un oggetto ad un nome in una identità. Abortisce il programma se l'identità indicata non è del sistema corrente.
*   `Object get_identity_module(NodeID id, string name)`.  
    Recupera un oggetto associato in precedenza ad un nome in una identità. Abortisce il programma se
    l'identità indicata non è del sistema corrente. Abortisce il programma se a quel nome non era stato
    in precedenza associato alcun oggetto.

La classe IdentityManager è provvista dei seguenti metodi per fornire all'utilizzatore del modulo
informazioni sul suo stato attuale. Si vedano sotto i dettagli sulle [associazioni](#Associazioni)
mantenute in memoria nel modulo.

*   `NodeID get_main_id()`.  
    Restituisce il NodeID della identità principale del sistema.
*   `Gee.List<NodeID> get_id_list()`.  
    Restituisce la lista dei NodeID di tutte le identità del sistema.
*   `string get_namespace(NodeID id)`.  
    Restituisce il nome del network namespace gestito da una data identità.
*   `string get_pseudodev(NodeID id, string dev)`.  
    Restituisce il nome dell'interfaccia di rete (reale o pseudo) gestita da una data identità al posto di una data interfaccia reale.
*   `Gee.List<IIdmgmtIdentityArc> get_identity_arcs(IIdmgmtArc arc, NodeID id)`.  
    Dato un arco *arc* (che quindi identifica anche un sistema diretto vicino *b*) e una identità
    *id* nel sistema corrente, restituisce i dati di tutti gli *archi-identità* formati su *arc*
    che partono dall'identità *id*.

La classe IdentityManager è provvista dei seguenti metodi per svolgere delle operazioni su richiesta
dell'utilizzatore del modulo:

*   `void prepare_add_identity(int migration_id, NodeID old_id)`.  
    Prepara il modulo ad avviare una concertazione coi diretti vicini per la produzione di una
    nuova identità del sistema basata su una identità precedente.  
    Si vedano sotto i dettagli al relativo [paragrafo](#add_identity).
*   `NodeID add_identity(int migration_id, NodeID old_id)`.  
    Attraverso una concertazione coi diretti vicini, realizza la produzione di una nuova identità
    del sistema basata su una identità precedente.  
    Si vedano sotto i dettagli al relativo [paragrafo](#add_identity).
*   `void add_identity_arc(IIdmgmtArc arc, NodeID id, NodeID peer_nodeid, string peer_mac, string peer_linklocal)`.  
    Forma un nuovo arco-identità su un arco.  
    Si vedano sotto i dettagli al relativo [paragrafo](#add_identity_arc).
*   `void remove_identity_arc(IIdmgmtArc arc, NodeID id, NodeID peer_nodeid)`.  
    Rimuove un arco-identità.  
    Si vedano sotto i dettagli al relativo [paragrafo](#remove_identity_arc).
*   `void remove_identity(NodeID id)`.  
    Rimuove dal sistema una identità di connettività.  
    Una identità non viene mai rimossa dal modulo Identities in autonomia, ma solo su richiesta
    dell'utilizzatore del modulo. Prima di fare al modulo questa richiesta, se l'utilizzatore sa che
    ci sono delle tasklet in esecuzione che presuppongono la persistenza di quella identità (ad esempio
    perché operano sul relativo *network stack*), deve fare in modo che queste siano terminate immediatamente.  
    Si vedano sotto i dettagli al relativo [paragrafo](#remove_identity).

La classe IdentityManager è provvista dei seguenti segnali per notificare degli eventi all'utilizzatore del modulo:

*   `identity_arc_added(IIdmgmtArc arc, NodeID id, IIdmgmtIdentityArc id_arc)`.  
    Un arco-identità è stato aggiunto. Può avvenire su richiesta dell'utilizzatore o in autonomia.
*   `identity_arc_changed(IIdmgmtArc arc, NodeID id, IIdmgmtIdentityArc id_arc)`.  
    Per un arco-identità esistente sono stati modificati i valori *peer_mac* e *peer_linklocal*. Può
    avvenire solo in autonomia.
*   `identity_arc_removed(IIdmgmtArc arc, NodeID id, NodeID peer_nodeid)`.  
    Un arco-identità è stato rimosso. Può avvenire su richiesta dell'utilizzatore o in autonomia.

## <a name="Dettagli"></a>Dettagli

### <a name="Associazioni"></a>Associazioni

L'elenco *dev_list* dei nomi delle interfacce di rete reali attualmente gestite dal sistema è tenuto in
una lista (ArrayList) di stringhe.

L'elenco *arc_list* degli archi attualmente realizzati dal sistema con i suoi vicini è tenuto in una
lista (ArrayList) di oggetti che implementano l'interfaccia IIdmgmtArc.

Ogni volta che un arco gli viene comunicato, il modulo produce una stringa con la quale può univocamente
identificarlo. Ad esempio un contatore. Quindi realiziamo un metodo *arc_to_string* che restituisce per
una istanza di IIdmgmtArc una stringa univoca, sebbene la classe che rappresenta un arco non è nota al modulo.

L'elenco *id_list* delle identità attualmente presenti è tenuto in una lista (ArrayList) di oggetti Identity.

Ogni istanza di Identity può essere espressa come una stringa univoca. Cioè ha un metodo *to_string* che
produce una stringa che può essere usata per identificare l'istanza.

* * *

L'associazione *namespaces* tra una identità *id<sub>0</sub>* e il suo namespace è realizzata con un
dizionario (HashMap) che ha per chiave la stringa prodotta dal *to_string* della identità *id<sub>0</sub>*.

* * *

Per ogni identità, per ogni interfaccia di rete reale gestita dal sistema, il modulo mantiene alcune informazioni (*dev*, *mac*, *linklocal*) in una struttura chiamata HandledNic. Sono i dati della pseudo-interfaccia costruita su quella interfaccia di rete reale (oppure i dati di quella vera) gestita da quella particolare identità del sistema.

L'associazione *handled_nics* tra una coppia *id<sub>0</sub>-dev<sub>i</sub>* (una identità e una interfaccia di rete reale) e la relativa istanza di HandledNic è realizzata con un dizionario (HashMap) che ha per chiave la stringa prodotta dal *to_string* della identità *id<sub>0</sub>* seguita da "-" e dal nome dell'interfaccia di rete reale.

* * *

Per ogni identità, per ogni arco realizzato dal sistema, il modulo mantiene un contenitore di istanze di IdentityArc. Ogni IdentityArc è una struttura dati con alcune informazioni (*peer_nodeid*, *peer_mac*, *peer_linklocal*). Sono i dati della interfaccia di rete (che può essere vera o pseudo) gestita da una particolare identità del sistema vicino raggiunto con quello specifico arco. Se esiste nel contenitore una istanza di IdentityArc significa che esiste un *arco-identità* tra i due sistemi.

L'associazione *identity_arcs* tra una coppia *id<sub>0</sub>-arc<sub>0</sub>* (una identità e un arco) e il relativo contenitore di istanze di IdentityArc è realizzata con un dizionario (HashMap) che ha per chiave la stringa prodotta dal *to_string* della identità *id<sub>0</sub>* seguita da "-" e dalla stringa prodotta dal metodo *arc_to_string* applicato all'arco *arc<sub>0</sub>*.

<a name="add_identity"></a>

### <a name="Duplicazione_di_una_identita"></a>Duplicazione di una identità

Quando sul manager viene chiamato il metodo *prepare_add_identity(migration_id, old_id)*, esso crea una istanza non completamente valorizzata della classe MigrationData. Tale classe ha un flag booleano *ready* che viene impostato a *False* per indicare appunto che non è stata valorizzata. Ma subito sono valorizzati i campi *migration_id* e *old_id*, proprio per riconoscere di quale migrazione si sta tenendo traccia con questo oggetto.

Questa struttura dati viene messa in una variabile che può essere acceduta da altre tasklet, come quelle che risponderanno ai metodi remoti del manager. Dopo di ciò il metodo *prepare_add_identity*, che restituisce `void`, fa ritorno.

L'istanza di MigrationData deve rimanere in memoria per un tempo limitato, sufficiente a garantire che tutti i sistemi diretti vicini hanno completato la migrazione in oggetto. Possiamo dire, ad esempio, che dopo 10 minuti dalla chiamata *prepare_add_identity* può essere rimossa in sicurezza.

Quando sul manager viene chiamato il metodo *add_identity(migration_id, old_id)*, il manager avvia un dialogo nel modulo Identities fra il sistema e i suoi vicini. Illustriamo ora tale dialogo nel dettaglio.

Indichiamo con *id<sub>j</sub>* il vecchio identificativo passato a parametro *old_id*. A fronte di tale richiesta, il manager genera un nuovo identificativo *id<sub>i</sub>* per la nuova identità. Aggiunge la nuova identità a *id_list*. Poi crea un nuovo network namespace *n<sub>temp</sub>*. Assegna alla vecchia identità *id<sub>j</sub>* il nuovo network namespace e alla nuova identità *id<sub>i</sub>* il namespace *n<sub>old</sub>* che prima era assegnato a *id<sub>j</sub>*. Concretamente, questo scambio di assegnazioni avviene modificando i valori dell'associazione *namespaces* mantenuta dal modulo.

Questo significa anche che se *id<sub>j</sub>* era la precedente identità principale, ora non lo è più e lo è diventata l'identità *id<sub>i</sub>*. Quindi può essere necessario modificare anche il valore di *main_id*.

Poi il manager, per ogni interfaccia di rete reale *r* che il sistema gestisce, tramite il *netns-manager* costruisce una nuova pseudo-interfaccia *p(r)* sopra *r* e la sposta nel network namespace *n<sub>temp</sub>*. Inoltre il modulo assegna a *p(r)* un nuovo indirizzo IP link-local scelto a caso. Avviene anche qui uno scambio. Il manager assegna alla vecchia identità *id<sub>j</sub>* le nuove pseudo-interfacce di rete e alla nuova identità *id<sub>i</sub>* le interfacce (reali o pseudo) che prima erano assegnate a *id<sub>j</sub>*. Concretamente, questo scambio di assegnazioni avviene modificando i valori dell'associazione *handled_nics* mantenuta dal modulo.

*   **Nota.** L'indirizzo link-local deve essere univoco a livello dei domini di broadcast dei diretti vicini del sistema. Questo significa che deve essere univoco nel sistema stesso e che nessuno dei suoi diretti vicini deve avere *attualmente* conoscenza di un link-local equivalente. Per una prima implementazione possiamo assumere che uno scelto a caso sia sufficiente a questo scopo, ma una corretta implementazione in futuro dovrebbe essere fatta.

Il manager adesso può popolare del tutto l'istanza di *MigrationData* e raccoglie in essa le seguenti informazioni:
*   *migration_id* - Identificativo numerico univoco per la migrazione.
*   *old_id* - L'identità che migra, in questo esempio *id<sub>j</sub>*.
*   *new_id* - L'identità nuova, in questo esempio *id<sub>i</sub>*.
*   *devices* - Una mappa di oggetti *MigrationDeviceData* che ha per chiave i nomi delle interfacce di rete reali gestite dal sistema.  
    Cioè: `HashMap<string,MigrationDeviceData> devices`.  
    In tale mappa, per ogni interfaccia di rete reale *r*, abbiamo in *devices[r]* una istanza di MigrationDeviceData con queste informazioni:
    *   *old_id_new_dev* - Il nome di *p(r)*.
    *   *old_id_new_mac* - L'indirizzo MAC di *p(r)*.
    *   *old_id_new_linklocal* - L'indirizzo link-local di *p(r)*.
*   *ready* - Adesso è *True*.

Qui termina una prima fase delle operazioni di *add_identity* nel sistema corrente. Indichiamola con il termine *fase di raccolta dati*. Il sistema può vedere che è terminata (ad esempio da un'altra tasklet) proprio esaminando il membro *ready* dell'oggetto MigrationData.

Ora il manager prende in esame ad uno ad uno tutti gli archi-identità che partono dalla identità *id<sub>j</sub>* e compie alcune operazioni. Ci sono due parti: una prima parte in cui le operazioni coinvolgono solo il sistema corrente. Una seconda parte in cui ci sono da fare delle comunicazioni coi diretti vicini, quindi occorre che il sistema sia preparato a gestire comportamenti imprevisti e/o scorretti da parte dei vicini.

Raggruppiamo i vari archi-identità in base all'arco su cui sono formati. Questo perché se riscontriamo un problema nelle comunicazioni attraverso un arco quell'arco va rimosso e con lui tutti gli archi-identità collegati. Quindi non ha senso proseguire le operazioni su altri eventuali archi-identità formati sullo stesso arco.

Per ogni arco *arc* in *arc_list*:
 * Per ogni arco-identità *IdentityArc w<sub>0</sub>* su *arc* che parte dalla identità *id<sub>j</sub>*:
  * Chiamiamo *a* il sistema corrente. Sia *arc<sub>0</sub>* l'arco su cui si appoggia *w<sub>0</sub>*. Sia *b* il sistema collegato all'arco *arc<sub>0</sub>*.
  * Sia *b<sub>j</sub>* l'identità del sistema *b* collegata a *w<sub>0</sub>*.
  * Il manager nel sistema *a* crea un duplicato *w<sub>1</sub>* dell'arco per assegnarlo ad *id<sub>i</sub>*. Cioè:
   * *w<sub>1</sub> = w<sub>0</sub>.copy();*
   * *identity_arcs(id<sub>i</sub>-arc<sub>0</sub>).add(w<sub>1</sub>);*
  * Il manager nel sistema *a* deve comunicare al manager nel sistema *b* attraverso l'arco *arc<sub>0</sub>* il *migration_id* visto prima e le informazioni dell'istanza di MigrationDeviceData relativa all'interfaccia *ir(arc<sub>0</sub>)*. In questa stessa comunicazione il sistema *b*, se la sua identità interessata *b<sub>j</sub>* ha partecipato alla stessa migrazione, risponde con l'identificativo della nuova identità basata su *b<sub>j</sub>* e le informazioni della pseudo-interfaccia gestita ora da *b<sub>j</sub>* relativa all'interfaccia reale su cui ha ricevuto il messaggio.  
    Questa comunicazione avviene concretamente con una chiamata in unicast realiable sull'arco *arc<sub>0</sub>* al metodo remoto *match_duplication* esposto dalla classe IdentityManager. La classe serializzabile usata per il valore di ritorno di questo metodo è DuplicationData.  
    Il manager nel sistema *a*, quando avvia questa chiamata sullo stub, deve concederle solo un certo tempo. Deve essere preparato a gestire l'evento che il sistema *b* termini la connessione, o risponda male, o non risponda entro un certo tempo.  
    La comunicazione se tutto procede bene avviene così:
   * Il sistema *a* comunica:
    * *migration_id* - L'identificativo della migrazione. È un dato primitivo utilizzabile in un metodo remoto.
    * *peer_id* - L'identificativo della identità *b<sub>j</sub>* in *b*. La classe NodeID è serializzabile.
    * *old_id* - L'identificativo della vecchia identità *id<sub>j</sub>* in *a*. La classe NodeID è serializzabile.
    * *new_id* - L'identificativo della nuova identità *id<sub>i</sub>* in *a*. La classe NodeID è serializzabile.
    * *old_id_new_mac* - Il MAC indicato nella istanza di MigrationDeviceData di cui sopra. È un dato primitivo utilizzabile in un metodo remoto.
    * *old_id_new_linklocal* - L'indirizzo link-local indicato nella istanza di MigrationDeviceData di cui sopra. È un dato primitivo utilizzabile in un metodo remoto.
   * Il manager nel sistema *b* ora verifica se riconosce il *migration_id* e se proprio l'identità *b<sub>j</sub>* sta partecipando alla stessa migrazione. Lo fa guardando se esiste una relativa istanza di MigrationData.
   * Se esiste l'istanza *migration_data*:
    * Finché **NOT** *migration_data.ready*:
     * Aspetta un po'. La *fase di raccolta dati* per tale identità non è completata.
    * Il manager nel sistema *b* individua l'interfaccia reale *if_b* su cui ha ricevuto il messaggio e risponde con:
     * *peer_new_id* - L'identificativo della nuova identità frutto della migrazione di *b<sub>j</sub>*, chiamiamola *b<sub>k</sub>*. Cioè *migration_data.new_id*.
     * *peer_old_id_new_mac* - Il MAC della nuova pseudo-interfaccia gestita ora da *b<sub>j</sub>* costruita su *if_b*. Cioè *migration_data.devices[if_b].old_id_new_mac*.
     * *peer_old_id_new_linklocal* - L'indirizzo link-local della nuova pseudo-interfaccia gestita ora da *b<sub>j</sub>* costruita su *if_b*. Cioè *migration_data.devices[if_b].old_id_new_linklocal*.
   * Altrimenti:
    * Il manager nel sistema *b* risponde semplicemente "OK".
  * Se nella comunicazione tra *a* e *b* attraverso l'arco *arc<sub>0</sub>* si riscontra uno dei problemi sopra citati (stub error, o deserialize error, o nessuna risposta entro un tempo limite) bisogna continuare con il prossimo arco *arc*. Questo arco invece verrà rimosso in una nuova tasklet tra qualche istante.
  * Il manager nel sistema *a* ora sa se l'identità *b<sub>j</sub>* ha partecipato anch'essa alla migrazione. Se sì, il sistema *a* conosce i dati *peer_old_id_new_mac* e *peer_old_id_new_linklocal* che sono riferiti a *b<sub>k</sub>*, la quale ora gestisce la vecchia interfaccia del sistema *b* che prima era gestita da *b<sub>j</sub>*.
  * Se *b<sub>j</sub>* ha partecipato alla migrazione:
   * Il manager nel sistema *a* cambia i dati dell'arco assegnato ad *id<sub>j</sub>* relativamente all'interfaccia remota:
    * *w<sub>0</sub>.peer_mac = peer_old_id_new_mac*.
    * *w<sub>0</sub>.peer_linklocal = peer_old_id_new_linklocal*.
   * Il manager nel sistema *a* cambia i dati dell'arco assegnato ad *id<sub>i</sub>* relativamente alla *identità* remota:
    * *w<sub>1</sub>.peer_nodeid* = Il NodeID di *b<sub>k</sub>*.
  * Il manager nel sistema *a* tramite il *netns-manager* aggiunge alle tabelle nel network namespace *namespaces(id<sub>j</sub>)* la rotta verso *w<sub>0</sub>.peer_linklocal* partendo da *old_id_new_linklocal* su *handled_nics(id<sub>j</sub>)(ir(arc<sub>0</sub>)).dev*.
  * Il manager nel sistema *b* a sua volta, ora sa che l'identità *id<sub>j</sub>* in *a* ha migrato e ha dato luogo a *id<sub>i</sub>*. Sa anche che la sua identità *b<sub>j</sub>* aveva un *arco-identità* con *id<sub>j</sub>* in *a* appoggiato sull'arco con *a* che parte da *if_b*. Ovviamente sa anche se l'identità *b<sub>j</sub>* ha partecipato anch'essa alla migrazione formando *b<sub>k</sub>*.
  * Se *b<sub>j</sub>* ha partecipato alla migrazione:
   * Il manager nel sistema *b* non ha bisogno di fare nulla in questo momento. Le sue variazioni le apporterà di sua iniziativa poiché anche in esso è avvenuta la migrazione di *b<sub>j</sub>* in *b<sub>k</sub>*.
  * Altrimenti:
   * Il manager nel sistema *b*, in autonomia come accennato sopra, forma un nuovo *arco-identità* *b<sub>j</sub>*-*id<sub>i</sub>* e lo segnala al suo utilizzatore.
   * Cambia i dati dell' *arco-identità* *b<sub>j</sub>*-*id<sub>j</sub>*, cioè MAC e linklocal, e lo segnala al suo utilizzatore.
   * Aggiunge una rotta nelle tabelle di un suo namespace (quello gestito da *b<sub>j</sub>*) per l'arco *b<sub>j</sub>*-*id<sub>j</sub>*.

Al termine delle operazioni, il metodo *add_identity* restituisce l'identificativo della nuova identità.

### <a name="Aggiunta_interfaccia"></a>Aggiunta o rimozione di una interfaccia di rete reale gestita dal sistema

<a name="add_handled_nic"></a>

Il metodo *add_handled_nic(dev, mac, linklocal)* viene richiamato dall'utilizzatore del modulo per segnalare che d'ora in poi una nuova interfaccia di rete reale sarà gestita dal sistema. Sono passati il nome, il MAC address e l'indirizzo link-local di questa interfaccia reale.

Il manager crea con questi dati una istanza di HandledNic e la aggiunge alla associazione *handled_nics* ma solo per l'identità principale *main_id* e *dev*.

* * *

<a name="remove_handled_nic"></a>

Il metodo *remove_handled_nic(dev)* viene richiamato dall'utilizzatore del modulo per segnalare che d'ora in poi una certa interfaccia di rete reale, che prima era stata segnalata come gestita, non sarà più gestita dal sistema. Viene passato solo il nome di questa interfaccia reale.

Il modulo assume che l'utilizzatore, prima di chiamare questo metodo, aveva rimosso tutti gli archi che erano formati su questa interfaccia, con il metodo *remove_arc*. Tale metodo, come è descritto nel relativo paragrafo, scatena la rimozione di tutti gli archi-identità.

Il manager, per ogni identità *id* in *id_list* tranne la principale *main_id*:
 * Se l'associazione *handled_nics* ha la chiave *id-dev*:
  * Significa che esiste una pseudo-interfaccia costruita su *dev* gestita da *id*.
  * Poniamo *ns = namespaces[id]* il namespace gestito da *id*.
  * Poniamo *pseudodev = handled_nics[id-dev].dev* la pseudo-interfaccia relativa a *dev* gestita da *id*.
  * Il manager usa il *netns-manager* per eliminare la pseudo-interfaccia (metodo *delete_pseudodev(ns, pseudodev)*).
  * Poi la rimuove dall'associazione *handled_nics* di questa identità *di connettività*.
  * Se questo fa restare tale identità senza alcuna pseudo-interfaccia gestita, allora quella identità andrà rimossa. Viene avviata una tasklet che se ne prenderà cura tra pochi istanti.

Poi il manager rimuove l'interfaccia dall'associazione *handled_nics* dell'identità *principale*.

Infine rimuove l'interfaccia dalla lista *devs_list*.

Il manager può ritrovarsi anche senza alcuna interfaccia di rete reale gestita. Non prende iniziative a tal riguardo.

### <a name="Aggiunta_arco"></a>Aggiunta o rimozione di un arco

<a name="add_arc"></a>

Il metodo *add_arc(arc)* viene richiamato dall'utilizzatore del modulo per segnalare la presenza di un arco formato dal sistema con un suo diretto vicino. Viene passata una istanza di IIdmgmtArc *arc*.

Il manager memorizza l'arco in *arc_list*.

Il manager, per ogni sua identità *id*, mette una lista vuota di IdentityArc in *identity_arcs* associato alla coppia *id-arc*.

In seguito il manager può usare l'arco per ottenere uno stub e comunicare con il diretto vicino. Inoltre tramite l'oggetto arco può anche vedere la propria interfaccia di rete reale su cui esso è costruito e l'indirizzo link-local del sistema vicino sull'interfaccia di rete reale in cui è raggiunto tramite questo arco. Attraverso queste informazioni il modulo in autonomia forma sopra questo arco il primo *arco-identità principale*.

Il modulo fa questa operazione in questo modo:
*   Ottiene uno stub tramite l'arco *arc*.
*   Chiama su esso il metodo remoto *get_peer_main_id()* e ottiene il NodeID *peer_id*.
*   Esegue il metodo *add_identity_arc(arc, main_id, peer_id, arc.get_peer_mac(), arc.get_peer_linklocal())*.

* * *

<a name="remove_arc"></a>

Il metodo *remove_arc(arc)* viene richiamato dall'utilizzatore del modulo per segnalare che un arco non è più valido. Viene passata una istanza di IIdmgmtArc *arc*.

Compito di questo metodo è rimuovere tutti gli archi-identità che esistono su questo arco. Vanno rimossi dalle associazioni che il modulo mantiene gli archi-identità formati su questo arco. Per ogni arco-identità tranne l'arco-identità *principale* di *arc*, si deve usare il *netns-manager* per rimuovere il collegamento diretto nel relativo network namespace.

La rimozione del collegamento diretto per l'arco-identità principale nel network namespace default non è compito del modulo Identities.

Il manager fa queste operazioni:
 * Per ogni identità *id* in *id_list* (inclusa la principale):
  * Poniamo *peer_id_list* una lista vuota di NodeID. Ci serve per fare un ciclo, poiché la lista contenuta nell'associazione *identity_arcs(id-arc)* subirà rimozioni durante il ciclo.
  * Per ogni IdentityArc *id_arc* della lista contenuta nell'associazione *identity_arcs(id-arc)*:
   * Metti in *peer_id_list* un elemento: *id_arc.peer_nodeid* cioè l'identificativo dell'altro vertice di *id_arc*.
  * Per ogni NodeID *peer_id* in *peer_id_list*:
   * Il manager chiama *remove_identity_arc(arc, id, peer_id)*.
  * Alla fine la collezione di IdentityArc in *identity_arcs* associata alla coppia *id-arc* dovrebbe risultare vuota.
  * Rimuove dall'associazione *identity_arcs* la collezione vuota.

Infine, il manager rimuove l'arco da *arc_list*.

### <a name="Aggiunta_arco_identita"></a>Aggiunta o rimozione di un arco-identità su richiesta dell'utilizzatore

<a name="add_identity_arc"></a>

Il metodo *add_identity_arc(arc, id, peer_id, peer_mac, peer_linklocal)* aggiunge un arco-identità sopra un arco. La prima chiamata a questo metodo relativamente all'arco *arc* avviene in modo automatico quando il sistema segnala la presenza di *arc* con il metodo *add_arc*. In questa chiamata viene richiesta la realizzazione dell'arco-identità *principale* dell'arco *arc*.

Il modulo in questo metodo deve saper riconoscere che l'arco-identità richiesto è l'arco-identità *principale* di *arc*. Perché in questo caso l'aggiunta del nuovo *arco-identità* non deve comportare la realizzazione con il *netns-manager* di un nuovo collegamento diretto. Infatti il modulo assume che il collegamento diretto tra i relativi indirizzi link-local che concretizza l'arco-identità *principale* di un arco *arc* è stato realizzato (sul *network namespace default*) senza il suo intervento.

Ovviamente il modulo sa verificare se l'identità *id* nel suo sistema è quella *principale*. Per verificare se l'identità *peer_id* nel sistema vicino è quella *principale* il modulo usa il metodo *get_peer_linklocal* dell'oggetto *arc* e lo confronta con l'argomento passato *peer_linklocal* e se coincidono allora assume che *peer_id* nel sistema vicino è la *principale*. Se entrambe le identità risultano essere la *principale* del sistema, allora l'arco-identità richiesto è l'arco-identità *principale* di *arc*.

Di norma l'utilizzatore del modulo non chiama mai esplicitamente questo metodo. Infatti l'arco-identità *principale* viene aggiunto automaticamente. Gli altri archi-identità vengono realizzati in autonomia dal modulo a seguito di una migrazione.

Tuttavia il modulo non vieta al suo utilizzatore di richiedere esplicitamente la realizzazione sopra un arco *arc* di un arco-identità diverso da quello che collega le due identità *principali*.

Riguardo gli archi-identità aggiunti in autonomia dal modulo, nel dettaglio possono esserci due casi:
*   Una identità *b<sub>0</sub>* di un vicino, la quale ha un arco-identità che la collega ad una identità *a<sub>0</sub>* del sistema corrente, ha migrato ed è stata replicata in *b<sub>1</sub>*, ma *a<sub>0</sub>* non ha partecipato alla stessa migrazione. In questo caso il sistema corrente ne viene notificato. Deve aggiungere alle sue associazioni un nuovo arco-identità che collega *a<sub>0</sub>-b<sub>1</sub>*; ma deve anche usare i dati (MAC address e indirizzo link-local) del vecchio arco *a<sub>0</sub>-b<sub>0</sub>* nel nuovo arco *a<sub>0</sub>-b<sub>1</sub>*; inoltre deve modificare i dati del vecchio arco *a<sub>0</sub>-b<sub>0</sub>* come gli sono stati notificati adesso; infine deve aggiungere un collegamento diretto usando il *netns-manager* con i nuovi dati ora assegnati al vecchio arco. In questo caso non si chiama il metodo *add_identity_arc*.
*   Una identità *a<sub>0</sub>* del sistema corrente ha migrato ed è stata replicata in *a<sub>1</sub>*. In questo caso la sequenza di operazioni da fare è un po' diversa ed è stata dettagliata nella descrizione dei metodi *prepare_add_identity* e *add_identity*. In questo caso non si chiama il metodo *add_identity_arc*.

Fatta questa premessa, il manager in questo metodo aggiunge il nuovo arco-identità alle sue associazioni. Se necessario, cioè se non si tratta dell'arco-identità *principale* di *arc*, usa il *netns-manager* per realizzare un nuovo collegamento diretto.

* * *

<a name="remove_identity_arc"></a>

Il metodo *remove_identity_arc(arc, id, peer_id)* rimuove un certo *arco-identità*.

Di norma, se è l'utilizzatore del modulo a fare questa esplicita richiesta, la richiesta non riguarda l'arco-identità *principale* di *arc*.

Ci sono altre situazioni in cui questo metodo può venire chiamato:
*   Il sistema corrente si avvede che un arco cessa di funzionare.  
    In questo caso il sistema chiama il metodo *remove_arc* del manager del modulo Identities. Il metodo *remove_arc* richiama il metodo *remove_identity_arc* per ogni arco-identità formato sull'arco. Anche quello *principale*.
*   Il sistema corrente rimuove una sua interfaccia di rete reale da quelle gestite.  
    In questo caso il sistema chiama prima il metodo *remove_arc* per ogni singolo arco formato tramite quella interfaccia di rete; poi richiamerà il metodo *remove_handled_nic*. Ogni chiamata di *remove_arc*, come visto sopra, produce varie chiamate a *remove_identity_arc*.
*   Un sistema vicino notifica la rimozione di una sua identità *di connettività*.  
    In questo caso il modulo Identities riceve la chiamata remota *notify_identity_removed(peer_id)* su un certo arco *arc*. In risposta a questo evento il modulo richiama il metodo *remove_identity_arc* per ogni arco-identità che collega una sua identità *id* a *peer_id* tramite *arc*.  
    Visto che il metodo *remove_identity_arc* semplicemente ignora la richiesta se non esiste un arco-identità che collega *id* e *peer_id* tramite *arc*, allora quanto appena detto si può implementare semplicemente con un ciclo per tutte le *id* in *id_list*.

Fatta questa premessa, il manager in questo metodo rimuove l'arco-identità dalle sue associazioni. Usa se necessario, cioè se non si tratta dell'arco-identità *principale* di *arc*, il *netns-manager* per eliminare un collegamento diretto.

Il manager fa queste operazioni:
*   Poniamo *dev = arc.get_dev()* l'interfaccia di rete reale su cui è formato l'arco.
*   Poniamo *ns = namespaces[id]* il namespace gestito da *id*.
*   Cerca l'IdentityArc *id_arc* relativo a *peer_id* nella lista contenuta nell'associazione *identity_arcs(id-arc)*. Se non lo trova ignora questa richiesta.
*   Poniamo *peer_linklocal = id_arc.peer_linklocal* l'indirizzo link-local dell'altro vertice di *id_arc*.
*   Poniamo *pseudodev = handled_nics[id-dev].dev* la interfaccia (reale o pseudo) relativa a *dev* gestita da *id*.
*   Il manager, se *id_arc* non è l'arco-identità *principale* di *arc*, usa il *netns-manager* per rimuovere il collegamento diretto (metodo *remove_gateway(ns, linklocal, peer_linklocal, pseudodev)*) rappresentato da *id_arc*.
*   Poi il manager rimuove *id_arc* dalla lista contenuta nell'associazione *identity_arcs(id-arc)*.

### <a name="Rimozione_identita_connettivita"></a>Rimozione di una identità di connettività

<a name="remove_identity"></a>

Il metodo *remove_identity(id)* può essere chiamato dall'utilizzatore del modulo per rimuovere una certa *identità di connettività*.

Questo non può essere fatto per l'identità *principale*. Il modulo lo controlla: se riceve questa richiesta abortisce il programma.

Il manager in questo metodo:
 * Per ogni arco *arc* in *arc_list*:
  * Segnala al vicino (con chiamata unicast reliable tramite *arc*) questa rimozione con il metodo remoto *notify_identity_removed(id)*.  
    Analogamente a quanto visto sopra per la chiamata remota *match_duplication*, il manager avvia questa chiamata sullo stub concedendole solo un certo tempo. Se il sistema vicino termina la connessione, o risponde male, o non risponde entro un certo tempo, il manager prosegue lo stesso.
  * Rimuove la collezione di IdentityArc contenuta nell'associazione *identity_arcs(id-arc)*.
 * Svuota la routing table del relativo network namespace, usando il *netns-manager*.
 * Elimina le relative pseudo-interfacce, usando il *netns-manager* e le rimuove dall'associazione *handled_nics(id-dev)*.
 * Elimina il network namespace relativo, usando il *netns-manager* e la rimuove dall'associazione *namespaces(id)*.
 * Rimuove l'identità dalla lista *id_list*.

