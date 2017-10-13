# Modulo Migrations - Analisi Funzionale

1.  [Il ruolo del modulo Migrations](#Ruolo_Migrations)
1.  [Incontro e fusione di due reti distinte](#Fusione_reti)
    1.  [Prima fase - valutazione del singolo nodo](#Fusione_reti_fase1)
    1.  [Seconda fase - valutazione della rete](#Fusione_reti_fase2)
    1.  [Terza fase - elezione dell'ingresso](#Fusione_reti_fase3)
    1.  [Quarta fase - comunicazione della elezione](#Fusione_reti_fase4)
    1.  [Quinta fase - comunicazione con il g-nodo entrante](#Fusione_reti_fase5)
    1.  [Sesta fase - richiesta della prenotazione di un posto](#Fusione_reti_fase6)
    1.  [Prenotazione riuscita: Settima fase - ingresso](#Fusione_reti_fase7a)
    1.  [Prenotazione fallita: Settima fase - annullamento al g-nodo entrante](#Fusione_reti_fase7b)
1.  [Strategia di ingresso](#Strategia_ingresso)
    1.  [Definizione della migration-path](#Strategia_ingresso_Definizione_migration_path)
    1.  [Uso della migration-path](#Strategia_ingresso_Uso_migration_path)
    1.  [Caratteristiche della migration-path](#Strategia_ingresso_Caratteristiche_migration_path)
    1.  [Algoritmo di ricerca](#Strategia_ingresso_Algoritmo_ricerca)
    1.  [Cancellazione delle prenotazioni](#Strategia_ingresso_Cancellazione_prenotazioni)
    1.  [Esecuzione della migration-path](#Strategia_ingresso_Esecuzione_migration_path)
    1.  [Algoritmo di esecuzione](#Strategia_ingresso_Algoritmo_esecuzione)
1.  [Risoluzione di uno split di g-nodo](#Split_gnodo)
1.  [Classe MigrationsMemory](#Class_MigrationsMemory)

## <a name="Ruolo_Migrations"></a>Il ruolo del modulo Migrations

Il ruolo del modulo Migrations è quello di trovare una migration-path e di coordinare
la sua esecuzione.

La ricerca di una migration-path ha come obiettivo la liberazione (se necessario) di una posizione
*reale* in un g-nodo. La migration-path più corta può essere di lunghezza zero, se in effetti una
posizione *reale* libera esiste già nel g-nodo.

La motivazione che spinge alla ricerca di una migration-path è sempre l'ingresso di un g-nodo in
una rete a cui non apparteneva. Questo può rendersi necessario a fronte di due situazioni:

*   Una rete *J* incontra una rete distinta *G*. Le due reti si erano formate indipendentemente.
*   Un g-nodo *g* di livello *l*, con *g* ∈ *G*, si è "splittato", cioè non è più internamente connesso.
    Si sono quindi formate varie isole *g'*, *g''*, eccetera. Questo
    mentre il suo g-nodo di livello superiore *h* risulta ancora internamente connesso. Allora
    ogni isola che non contiene il nodo più anziano deve considerarsi come una distinta
    rete *J* (composta da un solo g-nodo di livello *l* o minore) che incontra *G*.

Nelle varie fasi delle sue operazioni, il modulo Migrations si avvale della collaborazione del
modulo Coordinator, pur non avendo una diretta dipendenza sul modulo Coordinator. Questo è reso
possibile dall'utilizzatore di questi due moduli, cioè il demone *ntkd*, attraverso l'implementazione
di precise interfacce o delegati.

Le diverse modalità di questa collaborazione sono dettagliate nel documento di analisi del
modulo Coordinator ([qui](../ModuloCoordinator/AnalisiFunzionale.md#Collaborazione_migrations))
e comunque verranno illustrate nel resto del presente documento dove necessario.

In pratica questa collaborazione consiste in questo: il modulo Migrations
in alcuni dei suoi algoritmi in esecuzione in un singolo nodo *n*  ∈ *g* ha bisogno
di provocare l'esecuzione di altri suoi algoritmi in uno specifico nodo
*n<sub>0</sub>* ∈ *g* che sia sempre quello.  
Si sceglie di assegnare al nodo Coordinator del g-nodo *g* questo ruolo. Quando questa esecuzione
si rende necessaria il modulo Coordinator farà da proxy fra il generico singolo nodo *n*  ∈ *g*
e il nodo Coordinator di *g*.

In altri suoi algoritmi in esecuzione in un singolo nodo *n*  ∈ *g*, il modulo Migrations ha bisogno
di provocare l'esecuzione di altri suoi algoritmi in tutti i singoli nodi di *g*.  
In questo caso il modulo Coordinator coordinerà l'esecuzione su tutti i singoli nodi.

### Terminologia e notazioni

#### La rete con i suoi vertici

Indichiamo con *G = (V, E)* il grafo di una rete con i suoi vertici e archi, cioè i suoi singoli nodi
e i collegamenti fra di essi.

La rete ha una topologia gerarchica. Indichiamo con *levels* il numero di livelli di questa gerarchia
e con *gsizes(i)* il numero di posizioni *reali* al livello *i* dentro un g-nodo di livello *i* + 1, con 0 ≤ *i* ﹤ *levels*.

Sappiamo che un nodo *x* ha indirizzo *x<sub>0</sub>·x<sub>1</sub>·...·x<sub>levels-1</sub>*.

I nodi *x* e *y* appartengono allo stesso g-nodo di livello *l* se hanno un *l*-suffisso uguale, cioè i
valori nelle posizioni dalla *l* fino alla *levels-1*.

Lo denotiamo scrivendo *x* ~<sub>l</sub> *y*, come una relazione di equivalenza.

In particolare, *x* ~<sub>0</sub> *y* significa che *x* = *y*, cioè hanno lo stesso indirizzo.

Mentre *x* ~<sub>levels</sub> *y* significa solo che appartengono alla stessa rete, possono avere diversa
anche l'ultima posizone.

Con il simbolo *[x]<sub>l</sub>* indichiamo la classe dei nodi accomunati al nodo *x* tramite tale equivalenza.
Cioè tutti i nodi che hanno un indirizzo nella forma *\*·x<sub>l</sub>·...·x<sub>levels-1</sub>*.

In particolare, *[x]<sub>0</sub>* = {*x*}.

Mentre, *[x]<sub>levels</sub>* = *V*, cioè tutti i vertici del grafo *G*, tutta la rete.

#### G-nodi

Quando parliamo di un g-nodo, ad esempio il g-nodo di livello *l* a cui appartiene il nodo *x*, facciamo riferimento
ad un particolare grafo. Lo indichiamo con *g<sub>l</sub>(x)*.

Esso si ottiene considerando solo i nodi della classe *[x]<sub>l</sub>* e contraendo tutti i nodi che hanno lo
stesso *(l-1)*-suffisso, considerandoli come singoli vertici.

Ad esempio, *g<sub>levels</sub>(x)* è il grafo di tutta la rete a cui appartiene *x* costituito dai g-nodi (contratti)
di livello *levels-1*.

All'altro estremo, *g<sub>1</sub>(x)* è il grafo dei singoli nodi (livello 0) che appartengono allo stesso g-nodo di
livello 1 di *x*.

Continuiamo a chiamare (impropriamente) *nodi* questi vertici e chiamiamo *singoli nodi* i vertici del grafo originale *G*.

Se prendiamo tutti i g-nodi di livello *l* del grafo originale *G* e li consideriamo come singoli vertici, otteniamo
un altro grafo che rappresenta tutta la rete come composta da vertici di livello *l*. Questo grafo lo indichiamo
così: *[G]<sub>l</sub>*.

**Definizione** *𝛤*:  
Sia *g* un g-nodo di livello *l*. Indichiamo con *𝛤<sub>l</sub>(g)* l'insieme dei diretti vicini
di *g* nel grafo *[G]<sub>l</sub>*.  
In questa definizione sono inclusi anche g-nodi di livello *l* che hanno posizioni *virtuali* a qualche livello tra
*l* e *levels-1*.

Notiamo che un singolo nodo *n* che appartiene a *g* non ha una conoscenza diretta di tutti i g-nodi di *𝛤<sub>l</sub>(g)*.

**Definizione** *size*:  
Sia *g* un g-nodo di livello *l*.
Indichiamo con *size<sub>m</sub>(g)*, dove *m* < *l*, il numero di g-nodi di livello *m* dentro *g*. Questa definizione
va precisata, tenendo in considerazione il fatto che un g-nodo può assumere un indirizzo *virtuale* come descritto nella
trattazione del modulo Qspn. Quindi un g-nodo di livello *m* dentro *g* può avere uno o più identificativi *virtuali* ai
livelli da *m* a *l-1* inclusi. Precisiamo dunque che includiamo nel computo di *size<sub>m</sub>(g)* solo i g-nodi che
non hanno identificativi *virtuali* ai livelli da *m* a *l-1* inclusi.

Ricordando che *g* è un g-nodo di livello *l*, se abbiamo che *size<sub>l-1</sub>(g)* = *gsizes(l-1)* diciamo che il
g-nodo *g* è *saturo*: cioè non esistono dentro *g* g-nodi di livello *l-1* non allocati. Va notato che ogni singolo
nodo *x* appartenente a *g* è in grado autonomamente (tramite le sole conoscenze della sua mappa) di calcolare tale valore.

Se invece abbiamo *size<sub>0</sub>(g) = gsizes(0) \* gsizes(1) \* ... \* gsizes(l-1)* diciamo che il g-nodo *g* è *pieno*:
cioè non ci sono più dentro *g* indirizzi liberi. Va notato che un singolo nodo *x* appartenente a *g* non è in grado
autonomamente di calcolare tale valore, se non nel caso in cui *l* = 1.

## <a name="Fusione_reti"></a>Incontro e fusione di due reti distinte

Si consideri un nodo *n* che appartiene alla rete *G*. Questa è una generalizzazione,
che comprende ad esempio il caso di un singolo nodo che compone una intera rete.

Il modulo Migrations del nodo *n* che appartiene alla rete *G* si avvede del vicino *v* di altra rete *J*.

Questo incontro tra due singoli nodi di reti diverse è l'evento atomico di cui si compone
l'evento macroscopico "la rete *G* incontra la rete *J*".

Esaminiamo in questo documento le fasi che portano il modulo Migrations del nodo *n* a fare la sua parte
al fine di decidere e effettuare l'ingresso della rete *G* dentro la rete *J*.

È importante che la decisione venga presa dalla rete *G* come entità atomica, poiché il fatto che il
singolo nodo *n* abbia rilevato il contatto con *J* non esclude che le due reti siano
entrate in contatto più o meno nello stesso momento in diversi punti. Occorre evitare di avviare
distinte operazioni di ingresso che coinvolgono gli stessi g-nodi.  
Inoltre, quando si verifica un contatto fra due distinte reti, è desiderabile attendere un certo
tempo prima di procedere, per cercare di verificare la possibilità di fare ingresso
sfruttando il punto di contatto migliore.

### <a name="Fusione_reti_fase1"></a>Prima fase - valutazione del singolo nodo

Il modulo Migrations è un modulo *di identità*, cioè viene creata una istanza
per ogni identità nel sistema.

L'utilizzatore del modulo Migrations, cioè il demone *ntkd*, gli comunica la nascita e la rimozione
di ogni arco-identità.

Per ogni arco-identità il modulo Migrations contatta l'identità diretta vicina per chiedere informazioni sulla sua
rete di appartenenza, con il metodo remoto `retrieve_network_data`. La firma completa del metodo è
`NetworkData retrieve_network_data(bool ask_coord=False) throws NotPrincipalError`.

Questa comunicazione si completa solo se entrambe le identità sono *identità principali*, quindi con tutte
le posizioni *reali*. Inoltre, dopo questa comunicazione si procede solo se le topologie delle due
reti *G* e *J* sono identiche.  
Quindi nella nostra trattazione *n* e *v* hanno entrambi un indirizzo completamente *reale* e con
la stessa topologia di rete.

Il dettaglio delle operazioni per far sì che queste comunicazioni avvengano nei tempi e casi desiderati
è illustrato [qui](DettagliTecnici.md#Operazioni_arco_identita).

Il modulo Migrations del nodo *n* chiama il metodo remoto `retrieve_network_data` sul nodo *v* e riceve
una struttura dati che descrive *J* come è vista da *v*.  
La struttura `NetworkData` contiene:

*   `int64 netid` = Identificativo della rete *J*.
*   `List<int> gsizes` = Lista che descrive la topologia della rete *J*. Da essa si ricava `levels`.
*   `int neighbor_n_nodes` = Numero di singoli nodi dentro la rete *J*.
*   `List<int> neighbor_pos` = Per i valori *i* da 1 a `levels`, l'elemento `neighbor_pos[i-1]` è la
    posizione al livello `i-1` di *v* dentro *g<sub>i</sub>(v)*.

Da questa prima operazione il modulo Migrations in autonomia capisce se l'identità diretta vicina
appartiene ad una rete diversa. Ovviamente solo in questo caso procede con le successive operazioni.

Le due reti sicuramente vogliono fondersi in una. Si preferisce che sia la più piccola, come numero di singoli nodi in tutta la rete,
ad entrare nella più grande. Solo in caso di parità assoluta si ricorra all'identificativo della rete (che è un numero
casuale) come discriminatore. Però ricordiamo che il numero di singoli nodi in tutta la rete è un dato che ogni singolo
nodo ha in modo approssimativo.  
Questo significa che quando si incontrano le reti *G* e *J* attraverso molteplici archi è possibile che
in uno di questi archi (ad esempio quello dei nodi *n* e *v*) si decida che *G* deve entrare in *J*, mentre
in un altro arco (formato da altri due singoli nodi) si decida l'inverso. Questo non va bene.

Una soluzione può essere quella che entrambi i nodi chiedano questa informazione al nodo Coordinator della loro
rete. Però, non va bene importunare il Coordinator della rete (che può essere anche molto
grande) ogni qualvolta un singolo nodo della rete incontra un vicino che non appartiene ancora alla
rete. Infatti, soprattutto se si tratta di singoli nodi o di reti molto piccole, questo evento può
accadere molte volte e rapidamente, quindi congestionerebbe la rete soprattutto nella prossimità del
nodo Coordinator.  
Se invece le due reti hanno dimensioni simili, allora possiamo presumere che questo evento accada
raramente, quindi interrogare il nodo Coordinator va bene.

Procediamo dunque così: se il nodo *n* vede che la differenza è grande tra le due reti (diciamo una
20 volte più grande dell'altra) allora prende la decisione da solo. Se *G* deve entrare in *J* il
nodo *n* procede come vedremo nel seguito. Se è vero il contrario, invece, il nodo *n* non fa nulla. Sarà
il nodo *v* di sua iniziativa a fare le operazioni.

Se invece il nodo *n* vede che la differenza non è molta, allora riparte: chiede l'informazione
*numero di singoli nodi in G* al nodo Coordinator di *G*. Qui abbiamo una collaborazione col modulo Coordinator,
quella relativa al suo metodo `get_n_nodes`. Poi chiama di nuovo il metodo remoto `retrieve_network_data` sul nodo
*v* indicandogli però di chiedere l'informazione *numero di singoli nodi in J* al nodo Coordinator di *J*.  
A questo punto, anche se la differenza fosse piccola, il nodo *n* sa se proseguire o meno.

Per analizzare il resto delle operazioni in questo documento, assumiamo per ipotesi che il nodo *n*
decide che *G* deve entrare in *J*.

### <a name="Fusione_reti_fase2"></a>Seconda fase - valutazione della rete

Allora il modulo Migrations aggiunge un'altra informazione a quelle della struttura dati di cui sopra:

*   `int min_lvl` = Livello minimo a cui il singolo nodo *n* è disposto a fare ingresso. Infatti il nodo *n*
    potrebbe essere un gateway verso una rete privata in cui si vogliono adottare diversi meccanismi di
    assegnazione di indirizzi e routing. In questo caso il gateway potrebbe volere una assegnazione di un
    g-nodo di livello tale da poter disporre di un certo spazio
    (numero di bit) per gli indirizzi interni.

Inoltre sceglie un identificativo univoco random per questa richiesta, `int evaluate_enter_id`.

Ora il modulo Migrations del nodo *n* prepara una nuova struttura dati con le informazioni di cui sopra
istanziando un `EvaluateEnterData evaluate_enter_data`.  
La classe EvaluateEnterData è definita nel modulo Migrations. Si tratta di una classe serializzabile. I membri di questa classe sono:
`int64 netid`, `List<int> neighbor_pos`, `int min_lvl`, `int evaluate_enter_id`.  
L'istanza `evaluate_enter_data` andrà passata ad un metodo del modulo Migrations nel nodo Coordinator della rete.

Ora il modulo Migrations del nodo *n* fa in modo che venga richiamato nel modulo Coordinator il
metodo proxy `evaluate_enter` (vedi [qui](../ModuloCoordinator/AnalisiFunzionale.md#Deliverables_manager)).
A questo metodo viene passato il livello *levels* e un `Object evaluate_enter_data`.  
La reale classe che implementa questa struttura dati non è infatti nota al modulo Coordinator. Questi sa solo
che è serializzabile.

Grazie ai meccanismi del modulo Coordinator (di cui è trattato nella relativa documentazione) ora
nel nodo Coordinator della rete *G* viene chiamato dallo stesso modulo Coordinator (tramite un
delegato che ha ricevuto nel suo costruttore) il metodo `evaluate_enter` del modulo Migrations.  
Va considerato che, sempre grazie ai meccanismi del modulo Coordinator, oltre alla struttura dati
`evaluate_enter_data` il metodo `evaluate_enter` eseguito sul nodo Coordinator di *G* riceve
come argomento anche l'indirizzo di *n*, `List<int> client_address`.

Prima di vedere cosa fa il metodo `evaluate_enter` del modulo Migrations specifichiamo quale sarà il
suo output. Il valore restituito da questo metodo è una istanza della classe serializzabile `EvaluateEnterResult`
definita nel modulo Migrations. Anche qui diciamo che la classe non è nota al modulo Coordinator, che
lo riceve come un generico Object, sapendo solo che è serializzabile.  
La classe `EvaluateEnterResult` è in grado di rappresentare i possibili esiti del metodo, cioè
`int first_ask_lvl` oppure `AskAgainError` oppure `IgnoreNetworkError`.

Vediamo cosa avviene nel metodo `evaluate_enter` del modulo Migrations eseguito sul nodo Coordinator di *G*.

Ora il modulo Migrations nel nodo Coordinator di *G* computa il tempo in millisecondi `global_timeout` entro il quale intende rispondere alle
richieste di ingresso in una diversa rete. Abbiamo accennato prima al fatto che è bene attendere
un tempo per verificare la possibilità di fare ingresso sfruttando il punto di contatto migliore
fra le due reti.  
Questo tempo si calcola esclusivamente sulla base del numero di singoli nodi presenti in *G*. È lecito infatti
presumere che *G* sia la rete più piccola, poiché vuole entrare nell'altra. E lo scopo di questa attesa è
dare il tempo agli altri singoli nodi di *G*, che potrebbero venire in contatto a breve con l'altra rete
in altri punti, di raggiungere il nodo Coordinator di *G* con la loro proposta.

Oltre alle informazioni ricevute dal nodo *n*, il nodo Coordinator della rete *G* (sempre riferendoci
al codice in esecuzione nel modulo Migrations) conosce il livello più basso `int max_lvl`
tale che la rete *G* è composta da un solo g-nodo a quel livello. Questo valore è utile, perché
se si raggiunge la prenotazione di una posizione a questo livello, l'intera rete *G* può entrare in *J*
in blocco. Non è quindi necessario chiedere di più.

Il nodo Coordinator dell'intera rete *G* non ha particolari informazioni sui g-nodi (di livello inferiore al
minimo comune g-nodo) a cui appartiene *n*. Ma tali informazioni comunque non gli sono necessarie.

Bisogna individuare un livello a cui cercare di far entrare un g-nodo di *n* dentro un g-nodo di *v*.  
Sicuramente il primo tentativo sarà di entrare in blocco. Cioè con il livello più piccolo tale che la rete *G* è
costituita da un solo g-nodo, cioè `max_lvl`.

Però se abbiamo `max_lvl = levels` allora possiamo tentare al massimo con `levels - 1`. Invece se
abbiamo `max_lvl < min_lvl` allora dobbiamo tentare con `min_lvl`.

Chiamiamo `first_ask_lvl` il livello del g-nodo con cui faremo il primo tentativo. Vedremo in seguito che se questo
fallisce si potrà degradare, cioè tentare ingresso con un g-nodo di livello inferiore.

Il compito del nodo Coordinator di *G* adesso è quello di memorizzare che è stata incontrata la rete *J* attraverso
un arco tra il nodo *n* e il nodo *v*, tramite il quale potremo tentare di far entrare il g-nodo *g<sub>first_ask_lvl</sub>(n)*
dentro un g-nodo di *v*.

Assumiamo che questa richiesta sia la prima pervenuta nella rete *G*. Il modulo Migrations se ne avvede
accedendo alla memoria condivisa di tutta la rete (spiegheremo meglio il significato di questo a breve).  
Allora il modulo Migrations nel nodo Coordinator di *G* crea una istanza `EvaluateEnterEvaluation resp` con
i campi `evaluate_enter_data` e `client_address`. In seguito potremmo riferirci a questa
struttura dati con il termine *valutazione*.  
Inoltre rappresenta con un oggetto serializzabile `Timer timeout = global_timeout da ora` la scadenza di
questa *valutazione*.  
Infine rappresenta con un oggetto serializzabile `EvaluationStatus status = EvaluationStatus.PENDING` lo
stato di questa *valutazione*.  
A questo punto queste informazioni devono essere memorizzate nella memoria condivisa di tutta la rete *G*:
la *valutazione* come elemento di una lista nel membro `evaluation_list`; il livello con cui tentare
inizialmente nel membro `first_ask_lvl`; la scadenza nel membro `timeout`; lo stato nel membro `status`.

La memoria condivisa della rete (o più genericamente di un g-nodo) di pertinenza del modulo Migrations
è una istanza della classe `MigrationsMemory`. L'elenco dei suoi campi è riportato in seguito [qui](#Class_MigrationsMemory).

#### <a name="Accesso_memoria_condivisa"></a>Accesso alla memoria condivisa

Abbiamo già detto che il modulo Migrations può fare in modo che venga richiamato un metodo nel modulo Coordinator, pur non
avendo una dipendenza diretta sul modulo Coordinator. In particolare avremo una coppia di metodi
`get_migrations_memory` e `set_migrations_memory`
(vedi [qui](../ModuloCoordinator/AnalisiFunzionale.md#Deliverables_manager)) con i quali si recupera e si salva
una istanza di Object (perché il modulo Coordinator non conosce i dati del modulo Migrations) che costituisce
l'intera base dati (cioè la memoria condivisa della rete) relativa agli aspetti gestiti dal modulo Migrations.  
Il modulo Migrations nel nodo Coordinator di *G* recupera l'intera base dati corrente e la integra con
l'aggiunta di questa nuova *valutazione*. Poi immediatamente salva l'intera base dati.  
Abbiamo detto nella trattazione del modulo Coordinator che questi garantisce l'affidabilità e
la coerenza del dato. Ma sta al modulo Migrations, che è l'unico che accede a questi dati, garantire
l'atomicità di queste operazioni. Sarà sufficiente acquisire un *lock* **nel nodo corrente**
in tutti i punti del suo codice dove si accede a questa memoria: cioè l'uso del modulo Coordinator
fa sì che non sia necessario un meccanismo di lock su diversi nodi.  
In seguito ci riferiremo a questa sequenza di operazioni semplicemente dicendo che
il modulo Migrations nel nodo Coordinator di *G* accede e/o aggiorna la memoria condivisa di tutta la rete con delle
informazioni di sua pertinenza.

* * *

Ora il modulo Migrations nel nodo Coordinator di *G* risponde al client *n* con una eccezione AskAgainError.

L'eccezione AskAgainError, ricevuta come abbiamo detto sottoforma di una particolare istanza della
classe serializzabile `EvaluateEnterResult` dalla chiamata del metodo `evaluate_enter` sul modulo Coordinator,
istruisce il modulo Migrations nel nodo *n* di ripetere la stessa richiesta (con le stesse informazioni
tra cui lo stesso `evaluate_enter_id`) dopo aver atteso alcuni istanti.  
Questa attesa deve essere più piccola (almeno 3 o 4 volte) di quella calcolata come `global_timeout`,
che come abbiamo detto può essere calcolata dal modulo Migrations esclusivamente sulla base del numero di
singoli nodi presenti in *G*.

Supponiamo che nel frattempo giunga al Coordinator della rete *G* una richiesta simile dal nodo *w*
relativa alla rete *F*. Nel metodo `evaluate_enter` del modulo Migrations eseguito nel nodo Coordinator
di *G* per la richiesta pervenuta da *w*, accedendo alla memoria condivisa della rete si scopre che esiste una
precedente *valutazione* di ingresso in *J*. Si deduce che la seconda richiesta non può essere presa in
considerazione. Il nodo Coordinator deve rispondere con l'eccezione `IgnoreNetworkError`, che sarà spiegata sotto.

Supponiamo, invece, che nel frattempo giunga al Coordinator della rete *G* una richiesta simile dal nodo *q*
relativa alla rete *J*. Accedendo alla memoria condivisa della rete si scopre che esiste una
precedente *valutazione* di ingresso in *J* e che lo `status` ancora è `PENDING`. Si deduce che queste *valutazioni*
(quella sulla richiesta di *n* e quella sulla richiesta di *q*) vanno considerate insieme perché
riguardano la stessa rete *J*. Le due *valutazioni* vanno entrambe memorizzate nella lista `evaluation_list`.

La scadenza delle *valutazioni* resta la stessa memorizzata prima.

Il modulo Migrations nel nodo Coordinator di *G* aggiorna la memoria condivisa di tutta la rete
aggiungendo la *valutazione* relativa alla richiesta di *q*.

Ora il modulo Migrations nel nodo Coordinator di *G* si accinge a rispondere alla richiesta di *q*.
Se il timer `timeout` non è ancora scaduto il Coordinator risponde anche a questa richiesta con
una eccezione AskAgainError.

Se invece `timeout` è scaduto si passa alla terza fase.

### <a name="Fusione_reti_fase3"></a>Terza fase - elezione dell'ingresso

Consideriamo che ogni volta che arriva una richiesta `r` di ingresso in *J* il modulo Migrations nel
nodo Coordinator di *G* prima di valutarla accede alla memoria condivisa di tutta la rete per
vedere se a tale richiesta è stata già data una *valutazione*.  
Una *valutazione* `v` recuperata dalla memoria condivisa della rete è sempre identificabile
come quella associata ad una particolare richiesta `r` appena pervenuta verificando che
`r.evaluate_enter_id == v.evaluate_enter_data.evaluate_enter_id`.

Alla fine arriverà una richiesta `req` di ingresso in *J*. Il modulo Migrations nel nodo Coordinator
di *G* vedrà che esiste un gruppo di *valutazioni* per l'ingresso in *J*, che lo `status` è `PENDING`
e che  il `timeout` è scaduto.  
A questo punto il modulo Migrations dovrà eleggere la migliore fra le soluzioni.

Una cosa da evitare è quella di formare g-nodi che facilmente potrebbero "splittarsi", cioè diventare
internamente disconnessi.  
Per questo in `EvaluateEnterData evaluate_enter_data` memoriziamo `List<int> neighbor_pos` la posizione
del vicino *v* dentro *J*. Il modulo Migrations nel nodo Coordinator di *G* accedendo nella memoria condivisa
alla lista di *valutazioni* vede che un certo numero di queste ha per nodo vicino un nodo che appartiene
al g-nodo di *J* in cui *G* vorrebbe entrare. Deduce quindi il numero di archi che dovrebbero rompersi
per far sì che il g-nodo divenga disconnesso.

Diciamo che la soluzione eletta sia la *valutazione* `v`.

Nella memoria condivisa di tutta la rete il membro `status` passa a `TO_BE_NOTIFIED` e si aggiunge un
nuovo membro `EvaluateEnterEvaluation elected = v`. Il membro `timeout` viene di nuovo impostato
a `timeout = global_timeout da ora`.

Il modulo Migrations nel nodo Coordinator di *G* aggiorna la memoria condivisa di tutta la rete con tutte queste variazioni.

Ora il modulo Migrations guarda alla richiesta `req` appena pervenuta. Se la relativa *valutazione* `resp` è proprio `elected`
allora il modulo Migrations nel nodo Coordinator di *G* fa queste operazioni:

*   La *valutazione* `elected` viene rimossa dalla lista `evaluation_list`.
*   Il membro `status` passa a `NOTIFIED`.
*   Aggiorna la memoria condivisa di tutta la rete.
*   Risponde alla richiesta del client con `first_ask_lvl`.

Altrimenti il modulo Migrations fa queste operazioni:

*   Risponde alla richiesta del client con l'eccezione AskAgainError.

Le successive richieste saranno gestite nella quarta fase.

### <a name="Fusione_reti_fase4"></a>Quarta fase - comunicazione della elezione

Quando arriva una richiesta `req` di ingresso in *J* il modulo Migrations nel nodo Coordinator di *G* si avvede che si trova
nella quarta fase perché nella memoria condivisa di tutta la rete lo `status` è `TO_BE_NOTIFIED` o `NOTIFIED`.

Sia `v` la valutazione data a `req`.  
Può essere che `v` è stata recuperata da `evaluation_list`, oppure non c'era ed è stata calcolata
proprio ora. In quest'ultimo caso il modulo la aggiunge a `evaluation_list` e aggiorna la memoria
condivisa.

Se lo `status` è `TO_BE_NOTIFIED` e la valutazione `v` è proprio `elected`
allora il modulo Migrations nel nodo Coordinator di *G* fa queste operazioni:

*   La *valutazione* `elected` viene rimossa dalla lista `evaluation_list`.
*   Il membro `status` passa a `NOTIFIED`.
*   Aggiorna la memoria condivisa di tutta la rete.
*   Risponde alla richiesta del client con `first_ask_lvl`.

Altrimenti, se lo `status` è `TO_BE_NOTIFIED` il modulo Migrations fa queste operazioni:

*   Se `timeout` è scaduto:
    *   La *valutazione* `elected` viene rimossa dalla lista `evaluation_list`.
    *   Il membro `elected` viene azzerato.
    *   Il membro `status` torna a `PENDING`.
    *   Il membro `timeout` resta immutato.
    *   Il modulo Migrations ricomincia dalla terza fase: cioè si trova a dover eleggere la migliore fra le soluzioni
        presenti ora in `evaluation_list`.
*   Altrimenti:
    *   Risponde alla richiesta del client con l'eccezione AskAgainError.

Altrimenti, se lo `status` è `NOTIFIED` il modulo Migrations fa queste operazioni:

*   La *valutazione* `v` viene rimossa dalla lista `evaluation_list`.
*   Se `timeout` è scaduto svuota la lista `evaluation_list`.
*   Aggiorna la memoria condivisa di tutta la rete.
*   Risponde alla richiesta del client con l'eccezione IgnoreNetworkError.

L'eccezione IgnoreNetworkError, ricevuta come abbiamo detto sottoforma di una particolare istanza della
classe serializzabile `EvaluateEnterResult` dalla chiamata del metodo `evaluate_enter` sul modulo Coordinator,
istruisce il modulo Migrations nel nodo *n* di non prendere alcuna iniziativa e di evitare ulteriori valutazioni di ingresso nella
rete tramite il diretto vicino *v* per un certo tempo.  
Questo tempo potrebbe essere un multiplo (diciamo 20 volte tanto) di quello calcolato come `global_timeout`,
che come abbiamo detto può essere calcolata dal modulo Migrations esclusivamente sulla base del numero di singoli nodi presenti in *G*.

**Nota**: Il nodo Coordinator della rete *G* deve valutare una richiesta `EvaluateEnterData` alla
volta. Ciò significa che l'implementazione del metodo `evaluate_enter` del modulo Migrations deve acquisire
un *lock* (nel nodo corrente) prima di iniziare e rilasciarlo prima di rispondere.

### <a name="Fusione_reti_fase5"></a>Quinta fase - comunicazione con il g-nodo entrante

La quinta fase inizia quando un singolo nodo di *G*, assumiamo sia il nodo *n*, riceve l'autorizzazione
dal Coordinator di *G* di tentare l'ingresso in *J* tramite il suo vicino *v* con il suo g-nodo *g* di livello
*lvl* = `first_ask_lvl`.  
Ricordiamo che il valore `first_ask_lvl` è ricevuto dal nodo *n* come membro di una istanza di `EvaluateEnterResult` dalla chiamata
del metodo `evaluate_enter` sul modulo Coordinator.

Ora il modulo Migrations nel nodo *n* vuole chiamare il metodo `begin_enter` del modulo Migrations nel nodo Coordinator del
g-nodo *g*.

Prima il modulo Migrations del nodo *n* prepara una nuova struttura dati con le informazioni che servono
a questo metodo istanziando un `BeginEnterData begin_enter_data`.  
La classe BeginEnterData è definita nel modulo Migrations. Si tratta di una classe serializzabile. I membri
di questa classe sono:

*   nessuno?

Poi il modulo Migrations del nodo *n* fa in modo che venga richiamato nel modulo Coordinator il
metodo proxy `begin_enter`.  
A questo metodo viene passato il livello *lvl* e un `Object begin_enter_data`.
La reale classe che implementa questa struttura dati non è infatti nota al modulo Coordinator. Questi sa solo
che è serializzabile.

Grazie ai meccanismi del modulo Coordinator ora nel nodo Coordinator del g-nodo *g* viene chiamato
dallo stesso modulo Coordinator il metodo `begin_enter` del modulo Migrations. E questi riceve come argomento,
oltre alla struttura dati `begin_enter_data`, anche l'indirizzo di *n*, `List<int> client_address`.

Il valore che verrà restituito dal metodo `begin_enter` del modulo Migrations è una istanza della classe
serializzabile `BeginEnterResult` definita nel modulo Migrations.  
La classe `BeginEnterResult` è in grado di rappresentare i possibili esiti del metodo, cioè
`void` oppure `AlreadyEnteringError`.

Vediamo cosa avviene nel metodo `begin_enter` del modulo Migrations eseguito sul nodo Coordinator del g-nodo *g*.

Analogamente a quanto detto per il modulo Migrations nel nodo Coordinator di tutta la rete *G*, anche il modulo Migrations nel nodo
Coordinator del g-nodo *g* ha bisogno di poter accedere in lettura e scrittura alla memoria condivisa del
g-nodo *g* relativa agli aspetti gestiti dal modulo Migrations.  
Anche in questo caso, lo fa stimolando la chiamata dei metodi `get_migrations_memory` e `set_migrations_memory` nel modulo
Coordinator. E le operazioni possono essere considerate atomiche se il modulo Migrations si occupa di acquisire
un *lock* nel nodo corrente nei punti del suo codice che accedono a questa memoria condivisa.  
In seguito ci riferiremo a questa sequenza di operazioni semplicemente dicendo che
il modulo Migrations nel nodo Coordinator del g-nodo *g* accede e/o aggiorna la memoria condivisa di *g* con delle
informazioni di sua pertinenza.

Il modulo Migrations nel nodo Coordinator di *g* vuole ora accertarsi che ci sia un solo
nodo che porta avanti l'ingresso di *g* in blocco in una nuova rete.

Il modulo Migrations nel nodo Coordinator di *g* accede alla memoria condivisa di *g* e
verifica che non sia in corso un'altra operazione di ingresso di *g* in un'altra rete; e allo stesso
tempo memorizza che ora è in corso questa operazione e gli assegna un tempo limite per il suo
completamento. **TODO** quanto? deve permettere il completamento della ricerca della migration-path
nella rete *J* in cui stiamo entrando.  
Chiamiamo questa operazione "autorizzazione esclusiva a procedere".

Nella memoria condivisa del g-nodo *g* di pertinenza del modulo Migrations, cioè nell'istanza di
`MigrationsMemory` posta nel membro `migrations_memory` dell'istanza di `CoordGnodeMemory` associata
a *g*, il membro `Timer? begin_enter_timeout` è *null* se non è in corso una operazione di ingresso
di *g*. Se invece è valorizzato significa che è stata avviata una operazione di ingresso riguardante il
g-nodo *g*. Ma se il timer è scaduto l'operazione va considerata abortita.  
Quindi, se questo valore non è *null* e non è scaduto allora l'autorizzazione esclusiva non è riuscita.
Altrimenti il valore viene impostato con la nuova scadenza, viene aggiornata la memoria condivisa del g-nodo
e l'autorizzazione esclusiva è riuscita.

Se l'autorizzazione esclusiva non riesce, allora il metodo `begin_enter` del modulo Migrations
rilancia l'eccezione `AlreadyEnteringError` attraverso una apposita istanza di `BeginEnterResult`.
Il nodo *n* in questo caso desiste dal prendere l'iniziativa: sarà probabilmente un altro singolo nodo
a coordinare un ingresso in altra rete.

Se, invece, l'autorizzazione esclusiva riesce, allora il metodo `begin_enter` del modulo Migrations nel nodo
Coordinator del g-nodo *g* risponde positivamente al nodo *n*
(attraverso una apposita istanza di `BeginEnterResult`) permettendogli di proseguire con le operazioni di
ingresso di *g* in *J*.

### <a name="Fusione_reti_fase6"></a>Sesta fase - richiesta della prenotazione di un posto

La sesta fase inizia quando il nodo *n*, riceve l'autorizzazione dal Coordinator di *g* di chiedere al
suo vicino *v* la prenotazione di un posto per *g* in *J*.

Il modulo Migrations nel nodo *n* chiede al modulo Migrations nel nodo *v* di trovare una migration-path e di riservare
un posto per *g* (cioè per il g-nodo di livello *lvl* a cui appartiente *n*, con 0 ≤ *lvl* ﹤ *levels*) dentro *l'attuale* g-nodo
di *v* di livello *lvl+1* o superiore.  
Questo lo fa con il metodo remoto `search_migration_path` la cui firma è:  
`EntryData search_migration_path(int lvl) throws NoMigrationPathFoundError, MigrationPathExecuteFailureError`  

Il modulo Migrations nel nodo *v* cerca la shortest migration-path come descritto [qui](#Strategia_ingresso).

Se il nodo *v* trova che non esiste una migration-path a livello *lvl* lo comunica a *n*
con l'eccezione `NoMigrationPathFoundError`.  
Il nodo *n* dovrà comunicare al modulo Migrations nel nodo Coordinator di *g* che questo ingresso è abortito.
Questo processo sarà dettagliato nel paragrafo [Prenotazione fallita](#Fusione_reti_fase7b).  
Poi il modulo Migrations nel nodo *n* potrà decidere di degradare, cioè tentare ingresso con un
g-nodo di livello inferiore. Dovrà ripartire dalla comunicazione con il nuovo g-nodo entrante. Cioè
chiamare il metodo `begin_enter` del modulo Migrations nel nodo Coordinator di *g'* (di livello inferiore)
e quindi fare la richiesta di nuovo a *v*.

Altrimenti, cioè se il nodo *v* trova una migration-path, allora la esegue. In questo caso ci sono due
possibili esiti: l'esecuzione riesce oppure fallisce.  
L'esecuzione della migration-path da parte di *v* può fallire nel senso che il nodo *v* avvia la trasmissione
di un pacchetto (si vedrà sotto l'algoritmo di `send_mig_request`) per far eseguire la migrazione
*i*-esima e attende un pacchetto di risposta: se tale risposta non arriva entro un tempo limite deve
considerare fallita l'esecuzione della migration-path. In questo caso il nodo *v* lo comunica a *n*
con l'eccezione `MigrationPathExecuteFailureError`.  
In questo caso il nodo *n* riparte con la richiesta al suo vicino *v* di prenotare un posto per *g* in *J*. Di modo
che viene di nuovo avviata la ricerca di una migration-path allo stesso livello.

Se invece l'esecuzione della migration-path da parte di *v* riesce, questo farà si che
in uno dei g-nodi di *v* di livello maggiore di *lvl* ci sarà un posto riservato per l'ingresso di *g*.

Sia `host_gnode_level` il livello del g-nodo di *v* in cui adesso è stato riservato un posto.  
Questo livello può essere maggiore o uguale a *lvl* + 1. Infatti, come
descritto [qui](#Strategia_ingresso), la migration-path giudicata ottimale potrebbe essere
una di lunghezza zero che comporta la creazione di un nuovo g-nodo di grandezza maggiore.

Ora il nodo *v*, riguardo al suo g-nodo in cui adesso è stato riservato un posto, ha queste informazioni:

*   `host_gnode_level` - il livello.
*   `int new_pos` - la posizione di livello `host_gnode_level` - 1 riservata.
*   `int new_eldership` - l'anzianità della nuova posizione dentro il livello `host_gnode_level`.

Il nodo *v* aggiunge le altre informazioni che servono a *n*.

*   le altre posizioni del nuovo g-nodo, ai livelli superiori, che sono le stesse di *v*.
*   le altre anzianità del nuovo g-nodo, ai livelli superiori.

Va ricordato che *v* ora potrebbe anche avere una posizione *virtuale* al livello `host_gnode_level` - 1,
cioè essere una identità *di connettività* ai livelli da `host_gnode_level` (fino a un altro livello).
Ma questo sicuramente non cambia le sue conoscenze ai livelli superiori.

Il nodo *v* comunica al nodo *n* i dettagli per fare ingresso nel suo g-nodo nel posto che si è appena liberato.  
La classe `EntryData` che il metodo remoto `search_migration_path` restituisce contiene questi campi:

*   `List<int> pos` - Posizioni ai livelli da `host_gnode_level` - 1 a `levels`.
*   `List<int> elderships` - Anzianità ai livelli da `host_gnode_level` - 1 a `levels`.

### <a name="Fusione_reti_fase7a"></a>Prenotazione riuscita: Settima fase - ingresso

Questa fase inizia quando il nodo *n* riceve l'istanza di `EntryData` dal metodo remoto `search_migration_path`
che aveva chiamato sul nodo *v* per (trovare una migration-path e) riservare un posto per il suo g-nodo *g* di
livello *lvl*.

Ora il modulo Migrations nel nodo *n* vuole chiamare il metodo `completed_enter` del modulo Migrations nel nodo Coordinator del
g-nodo *g*.

Prima il modulo Migrations del nodo *n* prepara una nuova struttura dati con le informazioni che servono
a questo metodo istanziando un `CompletedEnterData completed_enter_data`.  
La classe CompletedEnterData è definita nel modulo Migrations. Si tratta di una classe serializzabile. I membri
di questa classe sono:

*   nessuno?

Poi il modulo Migrations del nodo *n* fa in modo che venga richiamato nel modulo Coordinator il
metodo proxy `completed_enter`.  
A questo metodo viene passato il livello *lvl* e un `Object completed_enter_data`.
La reale classe che implementa questa struttura dati non è infatti nota al modulo Coordinator. Questi sa solo
che è serializzabile.

Grazie ai meccanismi del modulo Coordinator ora nel nodo Coordinator del g-nodo *g* viene chiamato
dallo stesso modulo Coordinator il metodo `completed_enter` del modulo Migrations. E questi riceve come argomento,
oltre alla struttura dati `completed_enter_data`, anche l'indirizzo di *n*, `List<int> client_address`.

Il valore che verrà restituito dal metodo `completed_enter` del modulo Migrations è una istanza della classe
serializzabile `CompletedEnterResult` definita nel modulo Migrations.  
La classe `CompletedEnterResult` è in grado di rappresentare i possibili esiti del metodo, cioè
`void`.

Vediamo cosa avviene nel metodo `completed_enter` del modulo Migrations eseguito sul nodo Coordinator del g-nodo *g*.

Il nodo Coordinator del g-nodo *g* accede in scrittura alla memoria condivisa di *g*:
il membro `Timer? begin_enter_timeout` viene posto a *null*, a segnalare che non è più
in corso una operazione di ingresso di *g*.

Poi il metodo `completed_enter` del modulo Migrations nel nodo Coordinator del g-nodo *g* termina.
Il nodo *n* riceve la relativa istanza di `CompletedEnterResult`.

Ora il nodo *n* prosegue con l'ingresso vero e proprio. In modo simile a quanto dettagliato nella
trattazione dell'esecuzione della migration-path.

Per prima cosa il nodo *n* inventa un identificativo `migration_id`.  
Attraverso la *propagazione con ritorno* del metodo `prepare_migration` fa in modo che tutti i singoli nodi di *g* ricevano questa
informazione e avviino la prima parte delle operazioni di duplicazione dell'identità.

In `EntryData` abbiamo:

*   `List<int> pos` - Posizioni ai livelli da `host_gnode_level` - 1 a `levels`.
*   `List<int> elderships` - Anzianità ai livelli da `host_gnode_level` - 1 a `levels`.

Sappiamo che `host_gnode_level` ≥ *lvl* + 1.

Il nodo *n* attraverso la *propagazione senza ritorno* del metodo `finish_migration` fa in modo che queste informazioni giungano a tutti
i singoli nodi di *g*. Questi avviano la seconda parte delle operazioni di duplicazione dell'identità.
Il g-nodo *g* si duplica: viene creato il g-nodo isomorfo *g'* che entra nella nuova rete. Invece il vecchio *g*
viene dismesso.

### <a name="Fusione_reti_fase7b"></a>Prenotazione fallita: Settima fase - annullamento al g-nodo entrante

Questa fase si ha quando il nodo *n* riceve l'eccezione `NoMigrationPathFoundError` dal metodo remoto `search_migration_path`
che aveva chiamato sul nodo *v* per (trovare una migration-path e) riservare un posto per il suo g-nodo *g* di
livello *lvl*.  
Significa che non è possibile far entrare il g-nodo *g* in blocco. E che si potrà tentare l'ingresso in blocco
solo di un g-nodo di livello inferiore.

Ora il modulo Migrations nel nodo *n* vuole chiamare il metodo `abort_enter` del modulo Migrations nel nodo Coordinator del
g-nodo *g*.

Prima il modulo Migrations del nodo *n* prepara una nuova struttura dati con le informazioni che servono
a questo metodo istanziando un `AbortEnterData abort_enter_data`.  
La classe AbortEnterData è definita nel modulo Migrations. Si tratta di una classe serializzabile. I membri
di questa classe sono:

*   nessuno?

Poi il modulo Migrations del nodo *n* fa in modo che venga richiamato nel modulo Coordinator il
metodo proxy `abort_enter`.  
A questo metodo viene passato il livello *lvl* e un `Object abort_enter_data`.
La reale classe che implementa questa struttura dati non è infatti nota al modulo Coordinator. Questi sa solo
che è serializzabile.

Grazie ai meccanismi del modulo Coordinator ora nel nodo Coordinator del g-nodo *g* viene chiamato
dallo stesso modulo Coordinator il metodo `abort_enter` del modulo Migrations. E questi riceve come argomento,
oltre alla struttura dati `abort_enter_data`, anche l'indirizzo di *n*, `List<int> client_address`.

Il valore che verrà restituito dal metodo `abort_enter` del modulo Migrations è una istanza della classe
serializzabile `AbortEnterResult` definita nel modulo Migrations.  
La classe `AbortEnterResult` è in grado di rappresentare i possibili esiti del metodo, cioè
`void`.

Vediamo cosa avviene nel metodo `abort_enter` del modulo Migrations eseguito sul nodo Coordinator del g-nodo *g*.

Il nodo Coordinator del g-nodo *g* accede in scrittura alla memoria condivisa di *g*:
il membro `Timer? begin_enter_timeout` viene posto a *null*, a segnalare che non è più
in corso una operazione di ingresso di *g*.

Poi il metodo `abort_enter` del modulo Migrations nel nodo Coordinator del g-nodo *g* termina.
Il nodo *n* riceve la relativa istanza di `AbortEnterResult`.

Ora il nodo *n* decrementa di 1 il valore di *lvl*. Se adesso è minore di 0, allora non è proprio possibile
entrare nell'altra rete. Il nodo *n* in questo caso desiste dal prendere l'iniziativa.

Se invece il nuovo valore di *lvl* non è minore di 0, allora il nodo *n* potrà ripartire
dalla quinta fase (comunicazione con il g-nodo entrante) con il nuovo valore di *lvl*. Cioè,
considerando se stesso ancora come "eletto" a tentare l'ingresso in *J* per conto della rete *G*,
riparte dalla chiamata di `begin_enter` con il nuovo valore di *lvl*.

## <a name="Strategia_ingresso"></a>Strategia di ingresso

Assumiamo che il nodo *n* di *G* vuole usare il nodo diretto vicino *v* di *J* per far entrare il suo g-nodo *g* di livello *l*
in blocco dentro *J*. In questa richiesta possiamo avere 0 ≤ *l* ﹤ *levels*.

Il nodo *n* e il nodo *v* devono essere entrambi identità *principali* con indirizzi completamente *reali*.

Il nodo *n* potrebbe semplicemente chiedere al nodo *v* se ci sono posti liberi in uno dei suoi g-nodi di livello maggiore di *l*.

Il g-nodo *g*, grazie all'arco tra i nodi *n* e *v*, può entrare immediatamente nella rete e assegnarsi un indirizzo
valido se uno dei g-nodi di livello maggiore di *l* a cui il suo vicino *v* appartiene è *non saturo*. Se un tale
g-nodo non esiste, allora *g* non può entrare immediatamente nella rete.

Descriviamo in termini rigorosi il caso problematico.

Per ogni *i* da *l*+1 a *levels*, supponiamo che *size<sub>i-1</sub>(g<sub>i</sub>(v))* = *gsizes(i-1)*.
Cioè tutti i g-nodi di livello maggiore di *l* a cui appartiene *v* sono *saturi*.

Allora il g-nodo *g* non può entrare, cioè non è possibile assegnargli un indirizzo libero, in nessuno dei g-nodi di *v*.

A meno di trovare un meccanismo (meno invasivo possibile per la rete *J*) che porti alla liberazione di uno dei posti ora
occupati dentro il g-nodo di livello *l* + 1 a cui appartiene *v*. Questa seconda possibilità vale solo se
*l* ﹤ *levels* - 1.

Quindi correggiamo il tiro. La richiesta che il nodo *n* fa al nodo *v* è la seguente: trova, se possibile, un meccanismo
che permetta di liberare un posto nel tuo attuale g-nodo di livello *l* + 1 o superiore.

Abbiamo detto "se possibile". Infatti è possibile che in tutta la rete *J* tutti gli indirizzi possibili per un g-nodo
di livello *l* siano stati occupati.  
In termini rigorosi: *size<sub>l</sub>(g<sub>levels</sub>(v)) = gsizes(l) \* gsizes(l+1) \* ... \* gsizes(levels-1)*.  
Notare che questo, abbiamo detto sopra, non è possibile per il nodo *v* determinarlo autonomamente. Vedremo a breve come
il nodo *v* possa determinare questa situazione.  
In questo caso il nodo *v* comunicherà a *n* questa impossibilità e questi potrà decidere di tentare di far entrare
gradualmente *G* in *J* riprovando con un livello inferiore.

Abbiamo detto "nel tuo *attuale* g-nodo", perché *v* è attualmente una identità principale
in *J*, ma l'esito della richiesta potrebbe essere che lo stesso nodo *v* migra lasciando nel suo *attuale* posto
una identità *di connettività* come link per *g*.

Come risponde il nodo *v* a questa richiesta?

Indichiamo con *h* l'attuale g-nodo di livello *l* + 1 di *v*.  
Il nodo *v* per riservare una posizione per *g* cerca la *shortest migration-path* che libera un posto in *h* o in
un suo g-nodo superiore.

Specifichiamo rigorosamente cosa si intende per *migration-path*.

### <a name="Strategia_ingresso_Definizione_migration_path"></a>Definizione della migration-path

Sia *h* un g-nodo di livello *l* + 1, con *l* da 0 a *levels* - 2. È possibile che sia
*size<sub>l</sub>(h)* = *gsizes(l)*. Cioè *h* può essere saturo.

Definiamo *P* una migration-path a livello *l* che parte dal g-nodo *h* (di livello *l* + 1) se *P* è una
lista di g-nodi che soddisfa questi requisiti:

*   *P* = (*p<sub>1</sub>*, *p<sub>2</sub>*, ... *p<sub>m</sub>*).
*   *p<sub>1</sub>* = *h*. Se *m* = 1, quindi *p<sub>1</sub>* = *p<sub>m</sub>*, allora *p<sub>1</sub>* può essere
    *h* o un g-nodo di livello maggiore che contiene *h*.
*   Per ogni *i* da 1 a *m* - 1:
    *   **Nota** può essere *m* = 1, quindi questo ciclo non viene mai valutato.
    *   *lvl(p<sub>i</sub>)* = *lvl(h)* = *l* + 1.  
        Inoltre è necessario che il g-nodo *p<sub>i</sub>* abbia tutte le posizioni *reali*, ai livelli
        da *l* + 1 a *levels* - 1.
    *   *p<sub>i+1</sub>* ∈ *𝛤<sub>l+1</sub>(p<sub>i</sub>)*, cioè *p<sub>i+1</sub>* è direttamente collegato a
        *p<sub>i</sub>* nel grafo *[G]<sub>l+1</sub>*.  
        Inoltre è necessario che il g-nodo di livello *l* in *p<sub>i</sub>* diretto vicino di
        *p<sub>i+1</sub>* abbia la posizione *reale* al livello *l*.
    *   *size<sub>l</sub>(p<sub>i</sub>)* = *gsizes(l)*, cioè *p<sub>i</sub>* è saturo.
*   *lvl(p<sub>m</sub>)* = *k* ≥ *lvl(h)* = *l* + 1. Indichiamo con *k* il livello di *p<sub>m</sub>*.
*   *size<sub>k-1</sub>(p<sub>m</sub>)* `<` *gsizes(k-1)*, cioè *p<sub>m</sub>* non è saturo.

Usiamo una definizione che include anche una sorta di migration-path impropria, quella di lunghezza 0.

### <a name="Strategia_ingresso_Uso_migration_path"></a>Uso della migration-path

Detto in altri termini, il nodo *v* deve cercare la shortest migration-path a livello *l* che parte da *h*. Cioè:

*   un posto già libero nel suo g-nodo di livello *l* + 1, cioè in *h*; *oppure*
*   un posto già libero in un suo g-nodo di livello maggiore; *oppure*
*   la shortest migration-path per liberare un posto nel suo g-nodo di livello *l* + 1, cioè in *h*.

Usare questa migration-path significa far migrare un border g-nodo di livello *l* (che ha
una posizione *reale* al livello *l*) da ognuno dei g-nodi
di livello *l* + 1 della lista *P* nel successivo in modo tale da liberare un posto nel primo g-nodo
di livello *l* + 1 della lista *P*. Come *costo* abbiamo che viene occupato un ulteriore posto nell'ultimo g-nodo
della lista *P*, il quale non era saturo, ma anche poteva essere di livello maggiore di *l* + 1.

### <a name="Strategia_ingresso_Caratteristiche_migration_path"></a>Caratteristiche della migration-path

Una migration-path a livello *l* ha 2 caratteristiche importanti:

*   *d* = *m* - 1  
    distanza, cioè numero di migrazioni (di g-nodi di livello *l*) necessarie, con *d* ≥ 0.
*   *hl*  
    host g-node level, cioè il livello dell'ultimo g-nodo della lista, con *levels* ≥ *hl* > *l*.

Da un lato dello spettro abbiamo il caso ottimale: *d* = 0 e *hl* = *l* + 1. Cioè per far entrare il g-nodo
*g* non occorre nessuna migrazione e basta occupare una nuova posizione che era libera in un g-nodo esistente
di livello *l* + 1.

Si intuisce facilmente che allontanandoci da questa situazione, ovvero con il crescere di *d* e/o con
il crescere del delta *hl* - *l*, andiamo a fare operazioni che danneggiano maggiormente la nuova rete *J*.

Con il crescere di *d* cresce il numero di g-nodi che cambiano indirizzo, quindi è ovvio il danno.

Più sottile è il danno con il crescere del delta *hl* - *l*, ma non indifferente. Partiamo ad esempio
da un g-nodo di livello 1 saturo. Un nuovo nodo si aggiunge e questo può, senza obbligare a nessuna migrazione,
occupare un nuovo g-nodo di livello 1 dentro il g-nodo di livello 2 che diventa saturo. Un nuovo nodo si
aggiunge e ha un solo link con il g-nodo di livello 1 saturo; che è dentro un g-nodo di livello 2 saturo;
quindi può, senza obbligare a nessuna migrazione, occupare un nuovo g-nodo di livello 2 dentro il g-nodo di
livello 3 che diventa saturo. Un nuovo nodo si aggiunge e ha un solo link con il g-nodo di livello 1 saturo;
che è dentro un g-nodo di livello 2 saturo; che è dentro un g-nodo di livello 3 saturo;
quindi può, senza obbligare a nessuna migrazione, occupare un nuovo g-nodo di livello 3 dentro il g-nodo di
livello 4 che diventa saturo. E così via, con pochi nodi si rende saturo un g-nodo di alto livello
e si forma una rete poco bilanciata.  
Si potrebbe pensare che sarebbe conveniente quando il delta supera un certo *𝜀* investigare per
cercare una migration-path che, sebbene non sia la più breve come *d*, faccia occupare un nuovo posto
all'interno di un g-nodo di livello *hl* non troppo alto.  
Ad esempio un nuovo nodo si aggiunge e ha un solo link con il g-nodo di livello 1 saturo; che è dentro un g-nodo di
livello 2 saturo; che è dentro un g-nodo di livello 3 saturo; sebbene questo sia dentro un g-nodo di livello 4 che
non è saturo si preferisce cercare una diversa migration-path e si trova che facendo migrare un singolo nodo da un g-nodo
di livello 1 dentro un altro g-nodo esistente sempre di livello 1 si libera un posto per il nuovo nodo.

Quindi, quando si sceglie la topologia di una rete Netsukuku si sceglie anche un *𝜀* (ad esempio 5 se la
topologia è composta per lo più da gsize=2) per valutare gli ingressi.

Si procede in questo modo: la prima ricerca della shortest migration-path si fa senza porre alcun limite
superiore a *lh*. Trattandosi di una ricerca *breadth-first* o *in ampiezza* questa si ferma alla prima
shortest migration-path: cioè se dovessimo ripetere la ricerca dall'inizio ponendo dei parametri di ricerca
più restrittivi di sicuro avremmo un risultato la cui distanza *d* non può migliorare.

Quindi dalla prima ricerca otteniamo una migration-path con distanza *d<sub>0</sub>* e con livello del g-nodo
non saturo *hl<sub>0</sub>*. Se *hl<sub>0</sub>* risulta soddisfacente (cioè `hl0 - l < 𝜀`) ci fermiamo. Altrimenti riavviamo una nuova ricerca
ponendo un limite superiore a *hl* di *hl<sub>0</sub>-1*. Se la ricerca non trova
una migration-path ci fermiamo; se invece trova una migration-path avremo distanza *d<sub>1</sub>* e livello ospite *hl<sub>1</sub>*.
Di nuovo, se *hl<sub>1</sub>* risulta soddisfacente ci fermiamo. Altrimenti riavviamo una nuova ricerca
ponendo un limite superiore a *hl* di *hl<sub>1</sub>-1*. Se la ricerca non trova
una migration-path ci fermiamo; se invece trova una migration-path avremo distanza *d<sub>2</sub>* e livello ospite *hl<sub>2</sub>*.
E così via fino a quando non ci fermiamo perché il delta è minore di *𝜀* o perché non esistono
migration-path con il delta minore dell'ultimo *hl*.

Quando ci siamo fermati avremo un insieme di soluzioni tra cui scegliere. Qualsiasi soluzione in
questo insieme, anche se non ottimale, è comunque da preferire all'alternativa di degradare il livello
a cui si tenta di fare ingresso.

Data una soluzione *s<sub>i</sub>* di questo elenco, la successiva *s<sub>i+1</sub>* è da preferire
se *d<sub>i+1</sub>* `<` *d<sub>i</sub>* + 5 oppure
se *d<sub>i+1</sub>* `<` *d<sub>i</sub>* x 1.3.

Vediamo in dettaglio l'algoritmo di questa ricerca.

### <a name="Strategia_ingresso_Algoritmo_ricerca"></a>Algoritmo di ricerca

Premettiamo la definizione di alcune strutture dati che utilizzeremo.

Definiamo una struttura dati con la quale identificare un g-nodo con posizioni *reali* e
mantenere alcune informazioni relative.  
La struttura `TupleGNode` è serializzabile.  
Nel membro `pos` sono indicate tutte le posizioni dal livello del g-nodo fino al livello `levels` - 1.  
Nella struttura non è indicato il valore di `levels`,
cioè il numero di livelli della topologia, poiché si assume che i nodi con cui si entra in
contatto durante la ricerca della migration-path sono tutti della stessa rete.  
Nel membro `eldership` sono indicate tutte le anzianità dal livello del g-nodo fino al livello `levels` - 1.

```
TupleGNode:
  List<int> pos
  List<int> eldership
```

Definiamo le seguenti strutture dati nelle quali il nodo *v* memorizza le informazioni
raccolte durante la ricerca della migration-path. Queste **non** sono serializzabili.

```
SolutionStep:
  TupleGNode visiting_gnode
  int? previous_gnode_border_real_pos
  int? previous_gnode_new_conn_vir_pos
  int? previous_gnode_new_eldership
  SolutionStep? parent

Solution:
  SolutionStep leaf
  int host_lvl
  int real_new_pos
  int real_new_eldership
```

Prima di avviare l'algoritmo che restituirà una lista di soluzioni, il nodo *v* inventa un identificativo
`reserve_request_id`. Questo verrà comunicato ogni volta che, durante questa ricerca, verrà chiesto al Coordinator
di un g-nodo di riservare un posto. Potrà essere usato alla fine per cancellare le prenotazioni superflue.
Sarà spiegato nel dettaglio in seguito.

La signature dell'algoritmo è la seguente:

```
List<Solution> CercaShortestMig(int reserve_request_id, int ask_lvl, int 𝜀)
```

L'algoritmo è il seguente:

```
TupleGNode v = make_tuple_from_level(ask_lvl + 1)
int max_host_lvl = levels
List<Solution> solutions = []
int prev_sol_distance = -1

S = new Set<TupleGNode>
Q = new Queue<SolutionStep>

S.add(v)
SolutionStep root = new SolutionStep(visiting_gnode=v,
                                     previous_gnode_border_real_pos=NIL,
                                     previous_gnode_new_conn_vir_pos=NIL,
                                     previous_gnode_new_eldership=NIL,
                                     parent=NIL)
Q.enqueue(root)

Mentre Q is not empty:
  SolutionStep current = Q.dequeue().
  Se prev_sol_distance ≠ -1 AND
     prev_sol_distance + 5 ≤ n_step.get_distance() AND
     prev_sol_distance * 1.3 ≤ n_step.get_distance():
       Esci dal ciclo.
  // Contatta un singolo nodo in `current.visiting_gnode`.
  // Passa una tupla composta di `ask_lvl`, `max_host_lvl`, `reserve_request_id`, `𝜀 ≥ 1`.
  // La risposta sarà una tupla composta di: `esito`, `host_lvl`, `real_new_pos`, `real_new_eldership`,
  // `max_host_lvl`, `set_adjacent`, `new_conn_vir_pos`, `new_eldership`.
  Esito esito, int host_lvl, int real_new_pos, int real_new_eldership.
  Set<Pair<TupleGNode,int>> set_adjacent, int new_conn_vir_pos, int new_eldership.
  (esito, host_lvl, real_new_pos, real_new_eldership, max_host_lvl,
        set_adjacent, new_conn_vir_pos, new_eldership) =
        ask_enter_net(current, ask_lvl, max_host_lvl, reserve_request_id, 𝜀)
    // Questo algoritmo è eseguito nel singolo nodo contattato in `current.visiting_gnode`.
    Se real_pos_up_to(my_pos) ﹤ levels:
      Instrada eccezione SearchMigrationPathError
    int host_lvl = ask_lvl + 1
    int ok_host_lvl = ask_lvl + 𝜀
    // richiesta al proprio nodo Coordinator di livello host_lvl
    int pos, int eldership = coord_reserve(host_lvl, reserve_request_id)
    Se pos ﹤ gsizes(host_lvl - 1):
      Restituisci esito=GOAL, host_lvl, pos, eldership
    int new_conn_vir_pos = pos
    int new_eldership = eldership
    Mentre host_lvl ﹤ max_host_lvl:
      host_lvl++
      pos, eldership = coord_reserve(host_lvl, reserve_request_id)
      Se pos ﹤ gsizes(host_lvl - 1):
        Se host_lvl ≤ ok_host_lvl:
          Restituisci esito=GOAL, host_lvl, pos, eldership
        max_host_lvl = host_lvl - 1
        // Naturalmente di conseguenza esce dal ciclo.
    Set<Pair<TupleGNode,int>> set_adjacent = new Set<Pair<TupleGNode,int>>
    Per i = ask_lvl + 1 to levels - 1:
      // Vede quali g-nodi di livello i sono adiacenti al mio g-nodo di livello ask_lvl + 1
      // e quale g-nodo di livello ask_lvl dentro il mio g-nodo di livello ask_lvl + 1 sia il
      // relativo border-g-nodo.
      Set<Pair<HCoord,int>> adjacent_hc_set = adj_to_me(i, ask_lvl + 1)
      Per ogni HCoord hc, int border_real_pos in adjacent_hc_set:
        // Produce TupleGNode del g-nodo adiacente
        TupleGNode adj = make_tuple_from_hc(hc)
        // Aggiunge a quello i dati del border-g-nodo
        set_adjacent.add(Pair(adj, border_real_pos))
    Se pos ﹤ gsizes(host_lvl - 1)
      Restituisci esito=SOLUTION, host_lvl, pos, eldership, max_host_lvl,
                        set_adjacent, new_conn_vir_pos, new_eldership
    Altrimenti:
      Restituisci esito=NO_SOLUTION, max_host_lvl,
                        set_adjacent, new_conn_vir_pos, new_eldership
  Su eccezione SearchMigrationPathError:
    S.remove(current.visiting_gnode)
    Continue (prossima iterazione)
  Se esito = GOAL:
    Solution sol = new Solution(current, host_lvl, real_new_pos, real_new_eldership)
    solutions.add(sol)
    Restituisci solutions.
  Se esito = SOLUTION:
    Solution sol = new Solution(current, host_lvl, real_new_pos, real_new_eldership)
    solutions.add(sol)
    prev_sol_distance = sol.get_distance()
  Per ogni TupleGNode n, int border_real_pos in set_adjacent:
    // Notare che n è una tupla di livello maggiore o uguale a `ask_lvl + 1`.
    Se level(n) > ask_lvl + 1:
      // Contatta un singolo nodo in `n` passando per il percorso in `current`.
      // Passa `requested_lvl = ask_lvl + 1`.
      // La risposta sarà il TupleGNode di livello `requested_lvl` a cui esso appartiene.
      n = ask_tuple(current, n, requested_lvl = ask_lvl + 1)
        // Questo algoritmo è eseguito nel singolo nodo contattato in `n`.
        Restituisci make_tuple_from_level(requested_lvl)
      Se ora n ha componenti non reali:
        Continue (prossima iterazione)
    Se n is not in S:
      S.add(n)
      SolutionStep n_step = new SolutionStep(visiting_gnode=n,
                                             previous_gnode_border_real_pos=border_real_pos,
                                             previous_gnode_new_conn_vir_pos=new_conn_vir_pos,
                                             previous_gnode_new_eldership=new_eldership,
                                             parent=current)
      Q.enqueue(n_step)
Restituisci solutions.
```

#### Maggiori dettagli

La funzione `make_tuple_from_level(l)` produce una istanza di `TupleGNode` che identifica il g-nodo
di livello `l` a cui appartiene il nodo corrente. Il nodo corrente conosce per definizione
la posizione e l'anzianità dei suoi g-nodi.

La funzione `make_tuple_from_hc(hc)` produce una istanza di `TupleGNode` che identifica
il g-nodo che il nodo corrente vede nella sua mappa gerarchica con le coordinate `hc`. Il nodo
corrente nel modulo Migrations ha modo di vedere nella sua mappa gerarchica anche l'anzianità
del g-nodo che conosce come `hc`.

La funzione `level(n)` restituisce il livello del g-nodo identificato dalla tupla `n`.

Il nodo *v* ha un valore `𝜀` che ritiene *non soddisfacente* come delta tra il livello del g-nodo che vuole
entrare e il livello nel quale (al termine della migration-path) un g-nodo vedrà diminuito il numero di
posti liberi.

Come detto sopra, per la prima ricerca in ampiezza non viene imposto alcun limite al delta.
Per questo motivo il nodo *v* pone inizialmente `max_host_lvl` = `levels`.

##### Contatta il g-nodo da interrogare

Nell'algoritmo abbiamo detto che il nodo *v* contatta un singolo nodo in `current.visiting_gnode`. Ora dobbiamo vedere
nel dettaglio come si realizza questa comunicazione.  
Va subito tenuto conto del fatto che il nodo *v* ha tutte posizioni *reali*. Ma il primo singolo nodo che
si raggiunge in un dato g-nodo potrebbe avere posizioni *virtuali*, quindi è impossibile per esso stabilire
una connessione TCP con il nodo *v*.

Il contatto tra *v* e un g-nodo viene avviato dal nodo *v*. Esso invia un *pacchetto di richiesta* da instradare con
meccanismi simili al PeerServices. Ma il pacchetto contiene già tutti gli argomenti che servono
alla richiesta, oltre all'indirizzo Netsukuku completo di *v*.  
La comunicazione della risposta (o di un problema nell'instradamento) non avverrà aprendo una
connessione TCP end-to-end (come avviene nel modulo PeerServices) ma con un pacchetto da
instradare fino a *v*.

Bisogna aggiungere che il pacchetto *di richiesta* non viene instradato direttamente al g-nodo `current.visiting_gnode`, bensì attraverso
il percorso indicato in `current.parent.parent....`. Dobbiamo anche verificare durante il tragitto
dal nodo *v* al g-nodo destinazione che tutti i g-nodi indicati nel percorso siano effettivamente
adiacenti l'uno al successivo.  
A questo fine sono necessarie due cose: una è che l'intero percorso venga descritto nel pacchetto
stesso. L'altra è che ogni nodo che inoltra il pacchetto al diretto vicino successivo (cioè al miglior gateway
verso l'attuale destinazione) specifichi il suo indirizzo Netsukuku completo.

Il percorso da *v* verso `current.visiting_gnode` è indicato nell'istanza di SolutionStep `current` in modo ricorsivo.
Ora il nodo *v* prepara questi dati in una lista di strutture dati `List<PathHop> path_hops`, che sono di più
semplice gestione come oggetti serializzabili.

Ogni struttura contiene:

```
PathHop:
  TupleGNode visiting_gnode
  int? previous_gnode_border_real_pos
```

Il significato dei vari membri per l'elemento *i*-esimo della lista è il seguente:

*   `visiting_gnode` indica il g-nodo *p<sub>i</sub>* da raggiungere.  
    Nel primo elemento della lista questo membro è il g-nodo a cui appartiene *v*.
*   `previous_gnode_border_real_pos` è la posizione a livello *l* del g-nodo *𝛽<sub>i-1</sub>* nel g-nodo *p<sub>i-1</sub>*
    che dovrebbe risultare adiacente al g-nodo *p<sub>i</sub>*.  
    Nel primo elemento della lista questo membro è *null*.

Il g-nodo indicato nell'elemento `path_hops[0].visiting_gnode` è quello a cui appartiene *v*, quindi
non va "raggiunto".  
Quando si raggiunge il g-nodo indicato nell'elemento `path_hops[1].visiting_gnode`, occorre verificare che
il precedente singolo nodo sia del g-nodo `path_hops[0].visiting_gnode` e abbia posizione `path_hops[1].previous_gnode_border_real_pos`
al livello `ask_lvl`. Ricordiamo che tutti i g-nodi della lista sono di livello `ask_lvl + 1`.  
E così via.

Per tradurre il contenuto dell'istanza di SolutionStep nella lista di PathHop l'algoritmo è il seguente:

```
List<PathHop> get_path_hops(SolutionStep current):
  List<PathHop> path_hops = [];
  SolutionStep hop = current
  Mentre hop ≠ null:
    PathHop path_hop = new PathHop()
    path_hop.visiting_gnode = hop.visiting_gnode
    path_hop.previous_gnode_border_real_pos = hop.previous_gnode_border_real_pos
    path_hops.insert(0,path_hop)
    hop = hop.parent
  Restituisci path_hops
```

Supponiamo ad esempio che `current` si riferisce al g-nodo D che è adiacente al g-nodo C, adiacente
al g-nodo B, adiacente al g-nodo A, adiacente al g-nodo S a cui appartiene *v*. Tutti questi sono g-nodi
di livello `ask_lvl + 1`. Supponiamo inoltre che il g-nodo di livello `ask_lvl` dentro S diretto vicino
del g-nodo A abbia posizione *reale* al livello `ask_lvl` uguale a *w<sub>A</sub>*. Poi il g-nodo di
livello `ask_lvl` dentro A diretto vicino del g-nodo B abbia posizione *reale* al livello `ask_lvl` uguale
a *w<sub>B</sub>*. Lo stesso per *w<sub>C</sub>* e per *w<sub>D</sub>*. Questi dati sono stati precedentemente
raccolti dentro l'istanza `current`, con modalità che vedremo dopo. Quindi in `current` abbiamo questa situazione:

*   `current.visiting_gnode` = D
*   `current.previous_gnode_border_real_pos` = *w<sub>D</sub>*
*   `current.parent.visiting_gnode` = C
*   `current.parent.previous_gnode_border_real_pos` = *w<sub>C</sub>*
*   `current.parent.parent.visiting_gnode` = B
*   `current.parent.parent.previous_gnode_border_real_pos` = *w<sub>B</sub>*
*   `current.parent.parent.parent.visiting_gnode` = A
*   `current.parent.parent.parent.previous_gnode_border_real_pos` = *w<sub>A</sub>*
*   `current.parent.parent.parent.parent.visiting_gnode` = S
*   `current.parent.parent.parent.parent.previous_gnode_border_real_pos` = *null*
*   `current.parent.parent.parent.parent.parent` = *null*

Quindi avremo:

*   `path_hops[0].previous_gnode_border_real_pos` = *null*
*   `path_hops[0].visiting_gnode` = S
*   `path_hops[1].previous_gnode_border_real_pos` = *w<sub>A</sub>*
*   `path_hops[1].visiting_gnode` = A
*   `path_hops[2].previous_gnode_border_real_pos` = *w<sub>B</sub>*
*   `path_hops[2].visiting_gnode` = B
*   `path_hops[3].previous_gnode_border_real_pos` = *w<sub>C</sub>*
*   `path_hops[3].visiting_gnode` = C
*   `path_hops[4].previous_gnode_border_real_pos` = *w<sub>D</sub>*
*   `path_hops[4].visiting_gnode` = D

Prima si instrada il pacchetto *di richiesta* verso A, poi verso B, poi verso C e infine verso D.

È importante verificare durante il percorso che non ci siano incongruenze con quanto era memorizzato
in `current`. Vediamo come:  
Il pacchetto *di richiesta* inizialmente contiene tutta la lista `path_hops` preparata prima. Ogni nodo che lo riceve verifica
se fa parte del g-nodo corrente destinazione, cioè di `path_hops[1].visiting_gnode`, che all'inizio è A. Se non ne
fa parte allora semplicemente instrada il pacchetto verso `path_hops[1].visiting_gnode`.  
Una volta raggiunto il primo singolo nodo dentro il g-nodo A, questi deve verificare che il passo precedente era
*w<sub>A</sub>* dentro S. Per questo nell'instradamento *della richiesta* ogni nodo, oltre al pacchetto, indica il proprio
indirizzo completo. Questo indirizzo deve risultare in `path_hops[0].visiting_gnode` con posizione al livello `ask_lvl`
uguale a `path_hops[1].previous_gnode_border_real_pos`. Se è così, allora il nodo toglie il primo elemento dalla lista `path_hops` e
poi instrada il pacchetto *di richiesta* verso `path_hops[1].visiting_gnode`, che adesso è B.  
Allo stesso modo, una volta raggiunto il primo singolo nodo dentro il g-nodo B, questi deve verificare che
il passo precedente era *w<sub>B</sub>* dentro A, e così via.  
Se un nodo scopre una incongruenza, allora il fatto viene comunicato al nodo mittente *v* dal nodo
che lo ha scoperto: questi prepara un pacchetto *di eccezione* da instradare verso *v*. Per questo nel
pacchetto *di richiesta* viene indicato l'indirizzo *completo* del nodo mittente *v* e non solo le posizioni
interne al g-nodo comune con la destinazione finale.  
In questo caso, indicato nell'algoritmo con l'eccezione SearchMigrationPathError,
il nodo *v* stralcia completamente la path indicata da `current`, come se
avesse restituito `esito=NO_SOLUTION` con `set_adjacent` vuoto. Ma al contempo ora considera
il g-nodo finale `current.visiting_gnode` come non ancora visitato: cioè potrebbe visitarlo in seguito
attraverso altre path.

Proseguiamo ipotizzando che il percorso invece non contiene incongruenze.  
Indichiamo con *w* il primo singolo nodo che si è incontrato in `current.visiting_gnode`.  
Il nodo *w* nel pacchetto *di richiesta* ha ricevuto `ask_lvl`, `max_host_lvl`, `reserve_request_id`, `𝜀 ≥ 1`.
Ora il nodo *w* prosegue con l'algoritmo, che spiegheremo a breve con maggiori dettagli.
Intanto diciamo che alla fine il nodo *w* prepara un pacchetto *di risposta*
e lo instrada verso *v*. Esso contiene `esito`, `host_lvl`, `real_new_pos`, `real_new_eldership`,
`max_host_lvl`, `set_adjacent`, `new_conn_vir_pos`, `new_eldership`.

Riassumendo, i pacchetti contengono:

```
SearchMigrationPathRequest:
  List<PathHop> path_hops
  TupleGNode v
  int ask_lvl
  int max_host_lvl
  int reserve_request_id
  int 𝜀

SearchMigrationPathError:
  TupleGNode v

SearchMigrationPathResponse:
  TupleGNode v
  Esito esito
  int? host_lvl
  int? real_new_pos
  int? real_new_eldership
  int? max_host_lvl
  Set<Pair<TupleGNode,int>>? set_adjacent
  int? new_conn_vir_pos
  int? new_eldership
```

Per l'instradamento di questi pacchetti saranno previsti i metodi remoti:

```
route_search_request()
route_search_error()
route_search_response()
```

##### Riserva un posto per la migrazione

Il nodo *w* ora deve chiedere al Coordinator del suo g-nodo di livello `ask_lvl + 1` di riservare un posto. Ma per fare questo
è necessario, a causa delle attuali limitazioni del modulo PeerServices, che il nodo *w* abbia tutte le
componenti del suo indirizzo *reali*.  
Durante le operazioni di ricerca della migration-path risulta difficile garantire che questo
requisito sia soddisfatto. Forse in futuro una diversa implementazione del modulo PeerServices potrà
ridurre queste limitazioni.  
Per il momento adottiamo questa soluzione: se il nodo *w* contattato non ha tutte le componenti
*reali*, allora prepara un pacchetto *di eccezione* `SearchMigrationPathError` da instradare verso *v*
come abbiamo visto prima che poteva accadere durante l'instradamento. Questo consente al nodo
originante *v* di ignorare il presente percorso e al contempo non escludere tutto il g-nodo
dalla possibilità di essere visitato di nuovo.

Il nodo *w*, agendo per conto dell'intero g-nodo `current.visiting_gnode` e collaborando con i Coordinator
di quel g-nodo e dei suoi g-nodi superiori, ora vede se c'è un posto disponibile al livello
richiesto o a uno dei livelli superiori accettabili (cioè fino a `max_host_lvl`). Se non
ne trova uno ad un livello *soddisfacente* (cioè fino a `ok_host_lvl`) comunque
restituisce le informazioni (memorizzate in `new_conn_vir_pos` e `new_eldership`) necessarie
alla migration-path, cioè un posto *virtuale* al livello `ask_lvl + 1` e una nuova anzianità.

Per sapere se c'è un posto al livello `host_lvl` viene usata la funzione `coord_reserve`, la quale è un delegato
che chiama nel modulo Coordinator il metodo `reserve(host_lvl, reserve_request_id)`. Questo metodo
invia la richiesta ReserveEnterRequest ([prenota un posto](../ModuloCoordinator/AnalisiFunzionale.md#Prenota_un_posto))
al servizio Coordinator usando come chiave `host_lvl`. Come descritto nel relativo documento,
questo serve a prenotare un posto. Dalla posizione prenotata si deduce se il g-nodo aveva a
disposizione una posizione *reale* oppure no.

Quando il nodo *v* coordina la ricerca di una migration-path, esso ottiene un insieme di
soluzioni e solo una di esse viene adottata. Tutte le altre soluzioni contengono la prenotazione
di una posizione *reale* in un certo g-nodo.
Vedremo in seguito che il nodo *v* per ogni soluzione scartata dovrà inoltrare al g-nodo
interessato la richiesta di avvertire il proprio Coordinator che una certa prenotazione pendente
(identificabile con `reserve_request_id`) va cancellata.

##### Recupera i g-nodi adiacenti per proseguire la ricerca in ampiezza

Vediamo ora come fa il nodo *w* a dire quali g-nodi di livello `ask_lvl + 1` sono adiacenti al
suo g-nodo di livello `ask_lvl + 1`.  
Questo viene fatto solo se non è stato trovato qui un posto *reale* ad un livello con delta
*soddisfacente*.

Il nodo *w* sa quali sono i g-nodi di livello `ask_lvl + 1` interni al suo g-nodo di livello `ask_lvl + 2`,
cioè quali HCoord di livello `ask_lvl + 1` sono presenti nella sua mappa dei percorsi.
Tra questi sa dire quali (se ce ne sono) siano anche adiacenti al suo g-nodo di livello `ask_lvl + 1`: infatti esiste
un percorso verso essi che non contiene passi intermedi di livello `ask_lvl + 1`. È anche necessario
che l'ultimo passo intermedio di livello `ask_lvl` sia un HCoord con posizione *reale* a quel livello. Se non
esiste un passo intermedio di livello `ask_lvl`, allora è necessario che lo stesso nodo *w* abbia
una posizione *reale* a quel livello.

Generalizzando, il nodo *w* sa quali HCoord di livello `i` (con `i` da `ask_lvl + 1` fino a `levels - 1`)
sono adiacenti al suo g-nodo di livello `ask_lvl + 1`: infatti esiste
un percorso verso essi che non contiene passi intermedi di livello tra `ask_lvl + 1` e `i`
e nel quale l'ultimo passo intermedio di livello `ask_lvl` (o lo stesso nodo *w*) abbia
una posizione *reale* a quel livello.

Questo è quanto realizzato dalla chiamata della funzione `adj_to_me`.  
Essa viene reiterata per i valori di `i` da `ask_lvl + 1` fino a `levels - 1`. Per ogni chiamata
produce un set di coppie, di cui il primo elemento è un HCoord di livello `i` adiacente al
g-nodo di livello `ask_lvl + 1` di *w*, e il secondo è la posizione (*reale*) del border-g-nodo
di livello `ask_lvl` del g-nodo di livello `ask_lvl + 1` di *w*.

Dato un HCoord di livello `i`, il nodo *w* sa produrre l'istanza `TupleGNode` del g-nodo di
livello `i` con il metodo `make_tuple_from_hc` visto sopra.  
Per gli HCoord di livello `ask_lvl + 1` questo è quello che serve. Per quelli di livello
superiore, invece, questa informazione non è sufficiente. Ma non
possiamo affidare al nodo *w* il compito di contattare un singolo nodo in essi per scoprire
la tupla completa del g-nodo di livello `ask_lvl + 1` in quanto il nodo *w* potrebbe non avere
tutte le componenti *reali*. In conclusione il nodo *w* ottiene un set di tuple
il cui livello è maggiore o uguale a `ask_lvl + 1`.

##### Risposta al nodo richiedente

Il nodo *w* instrada verso il nodo *v* in un pacchetto *di risposta* la tupla composta di
`esito`, `host_lvl`, `real_new_pos`, `real_new_eldership`, `max_host_lvl`, `set_adjacent`, `new_conn_vir_pos`, `new_eldership`.

##### I g-nodi adiacenti devono essere del livello richiesto

Nel processare gli elementi della lista `set_adjacent` il nodo *v* (che ricordiamo ha un indirizzo
con tutte le componenti *reali*) se si trova la tupla `n` di un g-nodo di livello maggiore di `ask_lvl + 1`
instrada un *pacchetto di esplorazione* con meccanismi simili a quelli descritti per il *pacchetto di richiesta*
per chiedere al primo singolo nodo che incontra in essi la tupla completa del g-nodo di livello `requested_lvl = ask_lvl + 1`.  
Anche qui è necessario seguire tutto il percorso indicato da `current` e poi instradare verso `n`. Non è
invece necessario che durante il tragitto si verifichi anche l'adiacenza come visto prima.  
Il nodo che risponde alla richiesta dovrà instradare il *pacchetto di risposta esplorazione* al
nodo *v*.

Se il nodo che risponde alla richiesta appartiene ad un g-nodo di livello `requested_lvl` che ha componenti
non *reali* (è possibile ai livelli minori del livello della tupla `n` iniziale) allora il
nodo *v* ignora questa tupla. Altrimenti la processa.

Per intuizione crediamo (non è dimostrato al momento) che questo comportamento del nodo *v* gli permette di visitare gradualmente tutti
i g-nodi di livello `ask_lvl + 1` della rete, sebbene ogni singolo g-nodo che esso contatta non sia
in grado da solo di identificare tutti i g-nodi di livello `ask_lvl + 1` adiacenti a sé.

Anche qui si usa la struttura `PathHop` per garantire il corretto instradamento. Si usa sempre la funzione
`get_path_hops(current)` per avere la lista di passi e poi si aggiunge in coda l'elemento con
il membro `visiting_gnode` = `n`. Il membro `previous_gnode_border_real_pos` non serve.

Riassumendo, i pacchetti contengono:

```
ExploreGNodeRequest:
  List<PathHop> path_hops
  TupleGNode v
  int requested_lvl

ExploreGNodeResponse:
  TupleGNode v
  TupleGNode response
```

Per l'instradamento di questi pacchetti saranno previsti i metodi remoti:

```
route_explore_request()
route_explore_response()
```

##### Prosegue l'algoritmo di ricerca in ampiezza della shortest migration-path

Il nodo *v*, ricevuta la risposta (ed eventualmente dopo aver ottenuto i g-nodi adiacenti nel
livello richiesto) prosegue con l'algoritmo di ricerca in ampiezza.

La ricerca si interrompe quando si trova una migration-path con un delta minore di `𝜀`,
oppure quando dopo l'ultima migration-path trovata (sebbene con un delta *non soddisfacente*) sono stati fatti
troppi passi ulteriori senza trovarne una con un delta più piccolo.

Quando l'algoritmo di ricerca si interrompe, se qualche soluzione è stata trovata allora l'ultima
è quella da preferire.  
Inoltre, nel caso in cui ci fossero problemi durante l'esecuzione dell'ultima soluzione, sarebbe comunque
preferibile ripetere la ricerca da capo piuttosto che eseguire un'altra soluzione meno ottimale.  
Quindi il nodo *v* esegue immediatamente al contempo due operazioni: avvia l'esecuzione dell'ultima
soluzione e avvia dei messaggi per la cancellazione delle prenotazioni fatte nelle precedenti
soluzioni.

Se invece nessuna soluzione è stata trovata, il nodo *v* immediatamente comunica al nodo *n*
(che aveva fatto la richiesta di ingresso) questo risultato con l'eccezione `NoMigrationPathFoundError`.
Il nodo *n* potrà tentare di degradare.

### <a name="Strategia_ingresso_Cancellazione_prenotazioni"></a>Cancellazione delle prenotazioni

Per ogni soluzione `Solution s` che decide di non perseguire, il nodo *v* ha i dati necessari a
contattare il g-nodo che aveva fatta una prenotazione con posizione *reale* e a chiederne la
cancellazione.  
La tupla `s.leaf.visiting_gnode` rappresenta il g-nodo contattato, il quale però è di livello `ask_lvl + 1`
mentre il livello in cui è stata ottenuta una prenotazione *reale* è `s.host_lvl`. Quindi il
g-nodo da contattare va aggiustato.  
Il nodo *v* inoltre conosce l'identificativo `reserve_request_id` che è stato usato per etichettare la
richiesta di prenotazione di modo che potesse essere cancellata.

Il nodo *v* prepara una *pacchetto di cancellazione* e ne avvia l'instradamento verso il g-nodo interessato.

Il pacchetto contiene:

```
DeleteReservationRequest:
  TupleGNode dest_gnode
  int reserve_request_id
```

Per l'instradamento di questo pacchetto sarà previsto il metodo remoto:

```
route_delete_reserve_request()
```

Non c'è bisogno di attendere una risposta. Il primo singolo nodo in esso chiama il metodo `delete_reserve` del
modulo Coordinator per cancellare la prenotazione.

### <a name="Strategia_ingresso_Esecuzione_migration_path"></a>Esecuzione della migration-path

Dopo aver scelto la soluzione migliore, il nodo *v* deve coordinare la sua esecuzione.

Una migration-path di lunghezza zero è *impropria* e non ha bisogno di alcuna esecuzione. Semplicemente
il nodo *v* comunica al richiedente la posizione *reale* che è stata riservata e il livello
del g-nodo ospitante, che è uno dei suoi g-nodi di livello maggiore di *l*.  
Notiamo che una migration-path di lunghezza zero è l'unica soluzione possibile se si sta tentando di ottenere un posto
nella rete per un g-nodo di livello *l* = *levels* - 1.

Si consideri invece la migration-path *P* = (*p<sub>1</sub>*, *p<sub>2</sub>*, ... *p<sub>m</sub>*) a
livello *l*. Con 0 ≤ *l* ﹤ *levels* - 1 e *m* > 1. Vediamo come essa viene eseguita.

Ai fini delle comunicazioni che andranno fatte a singoli nodi dentro specifici g-nodi per eseguire la
migration-path, ricordiamo alcune cose:

**1**. Nel generico passo *i* i g-nodi di livello *l* + 1 *p<sub>i</sub>* e *p<sub>i+1</sub>* hanno tutte
posizioni *reali*, da *l* + 1 a *levels* - 1.

**2**. Nel generico passo *i* un border g-nodo *𝛽<sub>i</sub>* di livello *l* deve migrare da *p<sub>i</sub>* in
*p<sub>i+1</sub>*.  
Il g-nodo *𝛽<sub>i</sub>* è adiacente ad un g-nodo *𝛼<sub>i+1</sub>* di livello *l* dentro *p<sub>i+1</sub>*.

**3**. Il g-nodo *𝛽<sub>i</sub>* ha posizione *reale* al livello *l*. Ovviamente anche a quelle superiori
come detto sopra riguardo *p<sub>i</sub>*.  
Il primo singolo nodo che si incontra in *𝛽<sub>i</sub>* potrebbe avere posizioni *virtuali* ai livelli inferiori.  
Lo stesso vale per il singolo nodo in *𝛽<sub>i</sub>* che è in diretto contatto con *𝛼<sub>i+1</sub>*.

**4**. Il g-nodo *𝛼<sub>i+1</sub>* potrebbe avere posizione *virtuale* al livello *l*.

**5**. Dentro *p<sub>1</sub>* abbiamo il nodo *v* che ha fatto la ricerca della migration-path.  
Tale nodo ha posizioni *reali* dal livello 0  a *levels* - 1.

Il nodo *v* è in una posizione vantaggiosa: sappiamo che ha tutte posizioni *reali*. Questo gli
permette di essere contattato, quindi decidiamo che sia lui a coordinare tutte le operazioni.  
Inoltre faremo in modo che il passo che porta alla migrazione di *𝛽<sub>1</sub>* da *p<sub>1</sub>* in
*p<sub>2</sub>* venga eseguito per ultimo, perché vi è la possibilità che *v* appartiene a *𝛽<sub>1</sub>*
e in questo caso *v* assumerebbe in quel momento un indirizzo con posizioni *virtuali*.

Prima di proseguire con la descrizione della modalità di esecuzione della migration-path,
vediamo come viene realizzata una comunicazione tra *v*
e un dato g-nodo *𝛽<sub>i</sub>*. Il nodo *v* contatta un singolo nodo *𝛽0<sub>i</sub>*
in *𝛽<sub>i</sub>*. Questo contatto avviene inviando un pacchetto da instradare con i meccanismi
visti sopra per la ricerca.  
Non è necessario in questo caso seguire un determinato percorso. Basta infatti giungere ad un qualsiasi singolo
nodo dentro *𝛽<sub>i</sub>*.  
Il pachetto contiene tutte le informazioni che *v* vuole passare a *𝛽<sub>i</sub>*, la tupla che
identifica *𝛽<sub>i</sub>* e l'indirizzo Netsukuku completo di *v*.  
Il singolo nodo *𝛽0<sub>i</sub>* dovrà poi comunicare a *v* l'esito delle sue operazioni trasmettendo in modo
analogo un singolo pacchetto verso l'indirizzo Netsukuku completo di *v*.  
Prima di trasmettere l'esito il singolo nodo *𝛽0<sub>i</sub>* può fare uso della collaborazione con
il modulo Coordinator per avviare una *propagazione* a tutto il suo g-nodo, *con* o *senza* ritorno.

Per prima cosa il nodo *v* per ogni passo della migration-path (cioè per *i* da 1 fino a *m* - 1)
inventa un identificativo `migration_id` e lo associa a *𝛽<sub>i</sub>*.

Partendo da *i* = *m* - 1 e scendendo fino a 1, il nodo *v* contatta un singolo nodo *𝛽0<sub>i</sub>*,
che appartiene al g-nodo di livello *l* *𝛽<sub>i</sub>*, che appartiene al g-nodo di livello *l* + 1
*p<sub>i</sub>*. Notiamo che nell'ultimo passo, cioè con *i* = 1, è possibile che lo stesso nodo
*v* appartenga a *𝛽<sub>i</sub>*: in questo caso è lo stesso nodo *v* a fare le veci di *𝛽0<sub>i</sub>*.  
Il nodo *v* passa al nodo *𝛽0<sub>i</sub>* il suo `migration_id`.  
Il nodo *𝛽0<sub>i</sub>* attraverso la *propagazione con ritorno* del metodo `prepare_migration` fa in modo che
tutti i singoli nodi di *𝛽<sub>i</sub>* avviano la prima parte delle operazioni di duplicazione
dell'identità. Quando questa è stata eseguita, *𝛽0<sub>i</sub>* lo comunica al nodo *v* che
prosegue con il prossimo valore di *i*.

Ora il nodo *v* contatta un singolo nodo *𝛽0<sub>m-1</sub>* in *𝛽<sub>m-1</sub>*, in *p<sub>m-1</sub>*.
Se *m* = 2, è possibile che lo stesso *v* sia *𝛽0<sub>m-1</sub>*.  
Conosciamo `mig_gnode_old_pos`, la posizione di *𝛽<sub>m-1</sub>* in *p<sub>m-1</sub>*, che è *reale*. Essa è stata salvata nel membro
`previous_gnode_border_real_pos` di SolutionStep.  
Sappiamo anche che è stato riservato un posto *virtuale* `conn_gnode_pos` in *p<sub>m-1</sub>* che verrà assegnato
all'identità *di connettività* che *𝛽<sub>m-1</sub>* assume in *p<sub>m-1</sub>*. Esso è stato
salvato nel membro `previous_gnode_new_conn_vir_pos` di SolutionStep.  
Sappiamo anche che nel g-nodo *p<sub>m</sub>* di livello *k*, con *k* ≥ *l* + 1, è stato
riservato un posto *reale*. Indichiamo con `final_mig_gnode_host_lvl` il livello *k*, con
`final_mig_gnode_new_pos` la posizione in esso riservata e con `final_mig_gnode_new_eldership` la
sua anzianità. Questi valori sono stati salvati nei membri `host_lvl`, `real_new_pos` e
`real_new_eldership` di Solution.  
Il nodo *v* passa al nodo *𝛽0<sub>m-1</sub>* queste informazioni e il suo `migration_id`.  
Il nodo *𝛽0<sub>m-1</sub>* attraverso la *propagazione senza ritorno* del metodo `finish_migration` fa in modo che
queste informazioni giungano a tutti i singoli nodi di *𝛽<sub>m-1</sub>*. Questi avviano la seconda
parte delle operazioni di duplicazione dell'identità.  
Il g-nodo *𝛽<sub>m-1</sub>* si duplica: viene creato il g-nodo isomorfo *𝛽'<sub>m-1</sub>* che entra in *p<sub>m</sub>*
con la posizione `mig_gnode_new_pos`. Invece il vecchio *𝛽<sub>m-1</sub>* diventa un g-nodo *di connettività*
in *p<sub>m-1</sub>* assumendo la posizione *virtuale* `conn_gnode_pos`.  
Il g-nodo *di connettività* deve garantire di restare in vita
e di non rimuovere nemmeno gli archi esterni ai livelli di connettività per un certo tempo necessario al corretto
svolgimento della migration-path. Si veda a tale riguardo il metodo `remove_identity_arc` dettagliato nella
trattazione del modulo [Identities](../ModuloIdentities/DettagliTecnici.md), il segnale `presence_notified`
dalla nuova identità e il metodo `remove_outer_arcs` sulla vecchia identità nella
trattazione del modulo [QSPN](../ModuloQspn/DettagliTecnici.md#Migrazione). In seguito rimuoverà gli archi
esterni ed inizierà a verificare se c'è ancora bisogno di lui.  
Dopo aver dato il via alla *propagazione senza ritorno* il nodo *𝛽0<sub>m-1</sub>* lo comunica al nodo *v*.

Ora il nodo *v* riparte da *i* = *m* - 2 e scende fino a 1. Il nodo *v* contatta un singolo
nodo *𝛽0<sub>i</sub>* in *𝛽<sub>i</sub>*, in *p<sub>i</sub>*. Se *i* = 1, è possibile che
lo stesso *v* sia *𝛽0<sub>i</sub>*.  
Conosciamo `mig_gnode_old_pos`, la posizione di *𝛽<sub>i</sub>* in *p<sub>i</sub>*, che è *reale*. Essa è stata salvata nel membro
`previous_gnode_border_real_pos` di SolutionStep.  
Sappiamo anche che è stato riservato un posto *virtuale* `conn_gnode_pos` in *p<sub>i</sub>* che verrà assegnato
all'identità *di connettività* che *𝛽<sub>i</sub>* assume in *p<sub>i</sub>*. Esso è stato
salvato nel membro `previous_gnode_new_conn_vir_pos` di SolutionStep.  
Sappiamo anche che intanto un border-g-nodo di livello *l* sta migrando da *p<sub>i+1</sub>*
a *p<sub>i+2</sub>* e che la posizione che aveva in *p<sub>i+1</sub>* è `mig_gnode_new_pos`, ed è *reale*.
Essa è stata salvata nel membro `previous_gnode_border_real_pos` di SolutionStep riferita al passo successivo.  
Il nodo *v* passa al nodo *𝛽0<sub>i</sub>* queste informazioni e il suo `migration_id`.  
Il nodo *𝛽0<sub>i</sub>* attraverso la *propagazione senza ritorno* del metodo `finish_migration` fa in modo che
queste informazioni giungano a tutti i singoli nodi di *𝛽<sub>i</sub>*. Questi avviano la seconda
parte delle operazioni di duplicazione dell'identità.  
Il g-nodo *𝛽<sub>i</sub>* si duplica: viene creato il g-nodo isomorfo *𝛽'<sub>i</sub>* che entra in
*p<sub>i+1</sub>* con la posizione `mig_gnode_new_pos`. Invece il vecchio *𝛽<sub>i</sub>* diventa un g-nodo *di connettività*
in *p<sub>i</sub>* assumendo la posizione *virtuale* `conn_gnode_pos`: questi deve garantire di restare in vita
con tutti i suoi archi per un certo tempo, come già detto prima.  
Dopo aver dato il via alla *propagazione senza ritorno* il nodo *𝛽0<sub>i</sub>* lo comunica al nodo *v* che
prosegue con il prossimo valore di *i*.

### <a name="Strategia_ingresso_Algoritmo_esecuzione"></a>Algoritmo di esecuzione

Il nodo *v* ha tutte le informazioni necessarie ad ogni passo della migration-path codificate in una
istanza di Solution, la quale contiene in modo ricorsivo *m* istanze di SolutionStep. E abbiamo già detto
che non serve eseguire una migration-path di lunghezza zero, quindi *m* > 1.  
Ora il nodo *v* prepara questi dati in una lista di *m* - 1 strutture dati `MigData`, che sono di più
semplice gestione come oggetti serializzabili.

Ogni struttura contiene:

```
MigData:
  TupleGNode from_gnode
  int mig_gnode_old_pos
  int migration_id
  int conn_gnode_pos
  int prev_mig_gnode_new_eldership
  TupleGNode to_gnode
  int? mig_gnode_new_pos
  int? final_mig_gnode_host_lvl
  int? final_mig_gnode_new_pos
  int? final_mig_gnode_new_eldership
```

Il significato dei vari membri per l'elemento *i*-esimo della lista, con 1 ≤ *i* ≤ *m* - 1, è il seguente:

*   `from_gnode` indica il g-nodo *p<sub>i</sub>*, cioè il g-nodo da cui un g-nodo di livello *l*
    dovrà migrare.
*   `mig_gnode_old_pos` è la posizione a livello *l* del g-nodo *𝛽<sub>i</sub>* che
    dovrà migrare. Sappiamo che `mig_gnode_old_pos` è una posizione *reale*.
*   `migration_id` è l'identificativo associato alla migrazione di *𝛽<sub>i</sub>*.
*   `conn_gnode_pos` è la posizione *virtuale* riservata nel g-nodo *p<sub>i</sub>* per l'identità
    *di connettività* con cui il g-nodo *𝛽<sub>i</sub>* resta in *p<sub>i</sub>*. Sappiamo che non
    serve una particolare anzianità per i g-nodi *di connettività*.
*   `prev_mig_gnode_new_eldership` è l'anzianità che può essere assegnata al g-nodo che dal
    precedente passo della migration-path (oppure dal passo di ingresso nella rete che ha provocato
    la ricerca di una migration-path) viene ad occupare la posizione *reale* `mig_gnode_old_pos`
    appena liberata in *p<sub>i</sub>*.
*   `to_gnode` indica il g-nodo *p<sub>i+1</sub>*, cioè il g-nodo in cui il g-nodo *𝛽<sub>i</sub>*
    dovrà migrare.
*   `mig_gnode_new_pos` è *null* nell'ultimo elemento della lista.  
    Negli altri elementi, esso è la posizione *reale* a livello *l* che il g-nodo *𝛽'<sub>i</sub>*
    può assumere in *p<sub>i+1</sub>*.
*   `final_mig_gnode_host_lvl` è *null* in tutti gli elementi della lista tranne l'ultimo.  
    Nell'ultimo elemento della lista, esso è il livello di *p<sub>m</sub>* in cui è stato riservato
    un posto *reale* al livello `final_mig_gnode_host_lvl - 1`.
*   `final_mig_gnode_new_pos` e `final_mig_gnode_new_eldership` sono *null* in tutti gli elementi della lista tranne l'ultimo.  
    Nell'ultimo elemento della lista, essi sono posizione *reale* e sua anzianità riservate nel g-nodo *p<sub>m</sub>*
    per l'ultimo passo della migrazione.

Per tradurre il contenuto dell'istanza di Solution nella lista di MigData l'algoritmo è il seguente:

```
Solution s;
List<MigData> migs = [];

Assert s.leaf.parent ≠ null
bool last = True
int? mig_gnode_new_pos = null
SolutionStep current = s.leaf
Mentre current.parent ≠ null:
  MigData mig = new MigData()
  Se last:
    mig.final_mig_gnode_host_lvl = s.host_lvl
    mig.final_mig_gnode_new_pos = s.real_new_pos
    mig.final_mig_gnode_new_eldership = s.real_new_eldership
  mig.from_gnode = current.parent.visiting_gnode
  mig.mig_gnode_old_pos = current.previous_gnode_border_real_pos
  mig.migration_id = Random_int()
  mig.conn_gnode_pos = current.previous_gnode_new_conn_vir_pos
  mig.prev_mig_gnode_new_eldership = current.previous_gnode_new_eldership
  mig.to_gnode = current.visiting_gnode
  mig.mig_gnode_new_pos = mig_gnode_new_pos
  migs.insert(0,mig)
  last = False
  mig_gnode_new_pos = current.previous_gnode_border_real_pos
  current = current.parent
```

Segue l'algoritmo che *v* usa per instradare un pacchetto di richiesta verso il primo nodo dentro un
preciso g-nodo `dest` e ricevere una risposta.  
L'algoritmo illustrato viene usato sia per la prima fase, in cui si comunica ad un g-nodo di avviare
la prima fase della duplicazione, sia per la seconda, in cui si istruisce il g-nodo di completare la
duplicazione e le altre operazioni della migrazione. Le istanze di `RequestPacket` possono contenere
i dati che servono alla prima o alla seconda fase.  
Il nodo *v* chiama il metodo `send_mig_request` dopo aver preparato il pacchetto del tipo voluto. Il
metodo ritorna solo dopo che un singolo nodo nel g-nodo desiderato ha completato l'esecuzione del
metodo `void execute_mig(RequestPacket p0)`.

```
void send_mig_request(TupleGNode dest, RequestPacket p0) throws MigrationPathExecuteFailureError:
  p0.dest = dest
  Se my_pos in p0.dest:
    execute_mig(RequestPacket p0)
    Return
  // prepare to receive response
  p0.src = my_pos
  p0.pkt_id = Random_int()
  Channel ch = new Channel()
  request_id_map[p0.pkt_id] = ch
  // send request
  Stub st = best_gw_to(p0.dest)
  st.route_mig_request(p0)
  // wait response with timeout
  Try:
    var v = ch.recv(timeout)
  Su eccezione Timeout:
    Lancia eccezione MigrationPathExecuteFailureError
  Return



[Remote] void route_mig_request(RequestPacket p0):
  Se my_pos in p0.dest:
    ResponsePacket p1
    p1.pkt_id = p0.pkt_id
    p1.dest = p0.src
    p1.src = my_pos
    execute_mig(RequestPacket p0)
    // send response
    Stub st = best_gw_to(pq.dest)
    st.route_mig_response(p1)
  Altrimenti:
    // route request
    Stub st = best_gw_to(p0.dest)
    st.route_mig_request(p0)



void execute_mig(RequestPacket p0):
  do_something(p0)



[Remote] void route_mig_response(ResponsePacket p1):
  Se my_pos in p1.dest:
    Se NOT request_id_map.has_key(p1.pkt_id):
      Return
    Channel ch = request_id_map[p1.pkt_id]
    ch.send(p1)
  Altrimenti:
    // route response
    Stub st = best_gw_to(p1.dest)
    st.route_mig_response(p1)

```

Seguono i construttori della struttura dati `RequestPacket` che popolano i suoi membri
sulla base di una istanza di `MigData` per i due momenti.

```
RequestPacket.fase1(MigData mig)
  this.operation = RequestPacketType.PREPARE_MIGRATION
  this.migration_id = mig.migration_id

RequestPacket.fase2(MigData mig, MigData? mig_next)
  this.operation = RequestPacketType.FINISH_MIGRATION
  this.migration_id = mig.migration_id
  this.conn_gnode_pos = mig.conn_gnode_pos
  Se mig_next = null:
    this.to_gnode = mig.to_gnode sliced at level mig.final_mig_gnode_host_lvl
    this.real_new_pos = mig.final_mig_gnode_new_pos
    this.real_new_eldership = mig.final_mig_gnode_new_eldership
  Altrimenti:
    this.to_gnode = mig.to_gnode
    this.real_new_pos = mig.mig_gnode_new_pos
    this.real_new_eldership = mig_next.prev_mig_gnode_new_eldership

```

Segue l'algortimo che *v* usa per eseguire la migration-path descritta in `List<MigData> migs`.

```
Per i = migs.size - 1 scende fino a 0:
  MigData mig = migs[i]
  RequestPacket p0 = new RequestPacket.fase1(mig)
  send_mig_request(mig.from_gnode+mig.mig_gnode_old_pos, p0)
Per i = migs.size - 1:
  MigData mig = migs[i]
  RequestPacket p0 = new RequestPacket.fase2(mig, null)
  send_mig_request(mig.from_gnode+mig.mig_gnode_old_pos, p0)
Per i = migs.size - 2 scende fino a 0:
  MigData mig = migs[i]
  MigData mig_next = migs[i+1]
  RequestPacket p0 = new RequestPacket.fase2(mig, mig_next)
  send_mig_request(mig.from_gnode+mig.mig_gnode_old_pos, p0)

```

Segue l'algoritmo della funzione `execute_mig` che un singolo nodo *w* nel g-nodo destinazione
esegue sulla richiesta di *v*. Esso si compone di due parti, una eseguita nella prima fase e
l'altra nella seconda.

```
void execute_mig(RequestPacket p0):
  Se p0.operation = RequestPacketType.PREPARE_MIGRATION:
    int lvl = level(p0.dest)
    Object prepare_migration_data = new PrepareMigrationData(p0.migration_id)
    Coord.prepare_migration(lvl, prepare_migration_data)
  Altrimenti-Se p0.operation = RequestPacketType.FINISH_MIGRATION:
    int lvl = level(p0.dest)
    Object finish_migration_data = new FinishMigrationData(p0.migration_id,
                                   p0.conn_gnode_pos,
                                   p0.to_gnode,
                                   p0.real_new_pos,
                                   p0.real_new_eldership)
    Coord.finish_migration(lvl, finish_migration_data)
  Altrimenti:
    Ignora pacchetto

```

Nell'algoritmo descritto qui sopra, il modulo Migrations richiama un metodo (`prepare_migration`
oppure `finish_migration`) del modulo Coordinator. Il primo è un metodo a *propagazione con ritorno*,
il secondo è un metodo a *propagazione senza ritorno*, il cui funzionamento è dettagliato
[qui](../ModuloCoordinator/AnalisiFunzionale.md#Collaborazione_migrations) e
[qui](../ModuloCoordinator/AnalisiFunzionale.md#Deliverables_manager)
nella trattazione del modulo Coordinator. Essi provocano in tutti i singoli nodi del g-nodo destinazione
l'invocazione dei metodi (`prepare_migration` e `finish_migration`) del modulo Migrations.  
Si ricorda che il metodo `prepare_migration` deve terminare rapidamente perché il suo completamento
viene atteso. Invece il metodo `finish_migration` sarà eseguito in una nuova tasklet senza attendere
il suo completamento.  
Seguono gli algoritmi dei suddetti metodi.

```
void prepare_migration(lvl, prepare_migration_data):
  int migration_id = prepare_migration_data.migration_id
  var old_id = Identities.get_identity_from_migrations(this)
  Identities.prepare_add_identity(migration_id, old_id)

void finish_migration(lvl, finish_migration_data):
  int migration_id = prepare_migration_data.migration_id
  var old_id = Identities.get_identity_from_migrations(this)
  var new_id = Identities.add_identity(migration_id, old_id)
  var old_qspn_mgr = Identities.get_identity_module(old_id, "qspn")
  var new_qspn_mgr = new Qspn.migration(...)
  Identities.set_identity_module(old_id, "qspn", new_qspn_mgr)
  int host_lvl = level(finish_migration_data.to_gnode)
  int upper_lvl = upper_lvl(finish_migration_data.to_gnode, old_qspn_mgr.my_pos)
  old_qspn_mgr.make_connectivity(host_lvl,upper_lvl,update_addr,update_fp)
  new_qspn_mgr.presence_notified.connect(() => {
    wait(30sec)
    old_qspn_mgr.remove_outer_arcs()
  })
  wait(30sec)
  // inizia a verificare periodicamente se si può rimuovere old_qspn_mgr.
  ...

```

Questi ultimi algoritmi sono stati brevemente delineati, ma in effetti questi non sono di pertinenza
del modulo Migrations, bensì del suo utilizzatore. Quindi compito dei metodi `prepare_migration` e
`finish_migration` del modulo Migrations è quello di emettere i relativi segnali.

## <a name="Split_gnodo"></a>Risoluzione di uno split di g-nodo

Quando nella rete *G* un g-nodo *g* di livello *l* si "splitta" si formano diverse isole. Quelle che non hanno
dentro di sé il nodo che in precedenza era il più anziano, cioè il nodo che ha dato il suo fingerprint
al g-nodo *g*, dovranno considerarsi staccate dalla rete *G*.

Sia *g'* una di queste isole di livello *l*. Ad accorgersi di questa situazione è un border-nodo *n*
che non appartiene a *g'*, che è diretto vicino di un border-nodo *v* che appartiene a *g'*. Precisamente,
se ne accorge il modulo Qspn di *n*, il quale emette un determinato segnale.

Il demone *ntkd* in risposta a questo segnale chiama il metodo `signal_split` del modulo Migrations
indicando l'arco-identità su cui comunicare. Il modulo Migrations in esso chiama il metodo remoto
`you_have_splitted` su quell'arco-identità indicando il livello *l* dello split.

Il nodo *v* in questo metodo si avvale del modulo Coordinator per la *propagazione senza ritorno* del
metodo `we_have_splitted`. In questo metodo ogni singolo nodo del g-nodo *g'* emette il segnale `exit_network` con
il quale si spinge il demone *ntkd* a chiamare un determinato metodo del modulo Qspn.  
In tale metodo il modulo Qspn rimuove tutti i percorsi verso destinazioni esterne a *g'*. Quindi tutto il g-nodo *g'* diventa
una rete a sè, separata dal resto di *G*.  
Inoltre il modulo Qspn, nei nodi più esterni di *g'*, rimuove tutti gli archi che lo collegano direttamente a distinti
g-nodi. Avendo il modulo Qspn segnalato la rimozione degli archi, il demone *ntkd* rimuoverà gli archi-identità nel
modulo Identities. Di conseguenza se tali archi-identità erano principali il modulo Identities li riformerà di nuovo,
e il demone *ntkd*, accorgendosene, comunicherà al modulo Migrations la nascita di un nuovo arco.

Con i meccanismi descritti in precedenza questo provoca le operazioni di ingresso di *g'* nella rete originale con un
nuovo indirizzo come un nuovo distinto g-nodo internamente connesso.

## <a name="Class_MigrationsMemory"></a>Classe MigrationsMemory

La classe `MigrationsMemory` è serializzabile. È usata una sua istanza per rappresentare la
memoria condivisa di un g-nodo di pertinenza del modulo Migrations.

I suoi membri sono:

*   `List<EvaluateEnterEvaluation> evaluate_enter_evaluation_list` - Lista di valutazioni.
    Può essere vuota, se non si sta valutando alcun ingresso.
*   `int? evaluate_enter_first_ask_lvl`
*   `Timer? evaluate_enter_timeout`
*   `EvaluationStatus? evaluate_enter_status`
*   `EvaluateEnterEvaluation? evaluate_enter_elected`
*   `Timer? begin_enter_timeout` - Scadenza per un tentativo in corso di fare ingresso in
    blocco in un'altra rete. È *null* se nessun tentativo di questo tipo è in corso.

