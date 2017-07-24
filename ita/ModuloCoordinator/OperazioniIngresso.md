**NOTA**
Il contenuto di questo file andrà tasferito nella trattazione di un diverso modulo.

## Incontro e fusione di due reti distinte

Si consideri un nodo *n* che appartiene alla rete *G*. Questa è una generalizzazione,
che comprende ad esempio il caso di un singolo nodo che compone una intera rete.

Il modulo X del nodo *n* che appartiene alla rete *G* si avvede del vicino *v* di altra rete *J*.

Questo incontro tra due singoli nodi di reti diverse è l'evento atomico di cui si compone
l'evento macroscopico "la rete *G* incontra la rete *J*".

Assumiamo che le due reti si vogliano fondere. Più precisamente, che la rete *G* voglia entrare
a far parte della rete *J*.

È importante che la decisione venga presa dalla rete *G* come entità atomica, poiché il fatto che il
singolo nodo *n* abbia rilevato il contatto con *J* non esclude che le due reti siano
entrate in contatto più o meno nello stesso momento in diversi punti. Occorre evitare di avviare
distinte operazioni di ingresso che coinvolgono gli stessi g-nodi.  
Inoltre, quando si verifica un contatto fra due distinte reti, è desiderabile attendere un certo
tempo prima di procedere, per cercare di verificare la possibilità di fare ingresso
sfruttando il punto di contatto migliore.

### Prima fase - valutazione del singolo nodo

Il modulo X del nodo *n* chiede e ottiene dal vicino *v* una struttura dati che descrive *J* come è vista da *v*.  
Questa struttura contiene:

*   `int64 netid` = Identificativo della rete *J*.
*   `List<int> gsizes` = Lista che descrive la topologia della rete *J*. Da essa si ricava `levels`.
*   `List<int> neighbor_n_nodes` = Per i valori *i* da 1 a `levels`, l'elemento `neighbor_n_nodes[i-1]` è il
    numero di singoli nodi dentro *g<sub>i</sub>(v)*.
*   `List<int> neighbor_pos` = Per i valori *i* da 1 a `levels`, l'elemento `neighbor_pos[i-1]` è la
    posizione al livello `i-1` di *v* dentro *g<sub>i</sub>(v)*.
*   `List<int> neighbor_n_free_pos` = Per i valori *i* da 1 a `levels`, l'elemento `neighbor_n_free_pos[i-1]` è il
    numero di posizioni libere dentro *g<sub>i</sub>(v)*.

Poi quel modulo confronta la descrizione di *J* data da *v* con la rete *G* come è vista da *n*
e decide, assumiamo per ipotesi, che *G* deve entrare in *J*.

**Importante** Questa prima disamina delle caratteristiche di *J* e di *G* avviene sulla base delle
conoscenze di *n* e di *v*. Non va bene importunare il Coordinator della rete (che può essere anche
grande) ogni qualvolta un singolo nodo della rete incontra un vicino che non appartiene ancora alla
rete. Infatti può essere subito evidente dalle conoscenze dei singoli nodi che si vengono ad
incontrare quale sia la rete più piccola, quella cioè che cercherà un posto all'interno dell'altra.

### Seconda fase - valutazione della rete

Allora il modulo X aggiunge un'altra informazione a quelle della struttura dati di cui sopra:

*   `int minimum_lvl` = Livello minimo a cui il singolo nodo *n* è disposto a fare ingresso. Infatti il nodo *n*
    potrebbe essere un gateway verso una rete privata in cui si vogliono adottare diversi meccanismi di
    assegnazione di indirizzi e routing. In questo caso il gateway potrebbe volere una assegnazione di un
    g-nodo di livello tale (**N.B.** considerando la topologia della rete *J*) da poter disporre di un certo spazio
    (numero di bit) per gli indirizzi interni.

Inoltre sceglie un identificativo univoco random per questa richiesta, `int evaluate_enter_id`.

Ora il modulo X del nodo *n* prepara una nuova struttura dati con le informazioni di cui sopra
istanziando un `EvaluateEnterData evaluate_enter_data`.  
La classe EvaluateEnterData è nota al modulo X. Si tratta di una classe serializzabile. I membri di questa classe sono:
`int64 netid`, `List<int> gsizes`, `List<int> neighbor_n_nodes`, `List<int> neighbor_pos`, `List<int> neighbor_n_free_pos`,
`int min_lvl`, `int evaluate_enter_id`.  
L'istanza `evaluate_enter_data` andrà passata ad un metodo del modulo X nel nodo Coordinator della rete.

Ora il modulo X del nodo *n* fa in modo che venga richiamato nel modulo Coordinator (dal suo utilizzatore diretto, poiché
non è detto che il modulo X abbia una dipendenza diretta sul modulo Coordinator) il metodo `evaluate_enter`.  
A questo metodo viene passato un `Object evaluate_enter_data`.  
La reale classe che implementa questa struttura dati non è infatti nota al modulo Coordinator. Questi sa solo
che è serializzabile.

Grazie ai meccanismi del modulo Coordinator (di cui è trattato nella relativa documentazione) ora
nel nodo Coordinator della rete *G* viene chiamato dallo stesso modulo Coordinator (a cui era
stato passato come delegato nel suo costruttore) il metodo `evaluate_enter` del modulo X.  
Di fatto il modulo Coordinator chiama il metodo `evaluate_enter` dell'interfaccia IEvaluateEnterHandler
e questi chiamerà il metodo `evaluate_enter` del modulo X.  
Va considerato che, sempre grazie ai meccanismi del modulo Coordinator, oltre alla struttura dati
`evaluate_enter_data` il metodo `evaluate_enter` eseguito sul nodo Coordinator di *G* riceve
come argomento anche l'indirizzo di *n*, `List<int> client_address`.

Vediamo cosa avviene nel metodo `evaluate_enter` del modulo X eseguito sul nodo Coordinator di *G*.

Ora il modulo X nel nodo Coordinator di *G* computa il tempo in millisecondi `global_timeout` entro il quale intende rispondere alle
richieste di ingresso in una diversa rete. Abbiamo accennato prima al fatto che è bene attendere
un tempo per verificare la possibilità di fare ingresso sfruttando il punto di contatto migliore
fra le due reti.  
Questo tempo si calcola esclusivamente sulla base del numero di singoli nodi presenti in *G*. È lecito infatti
presumere che *G* sia la rete più piccola, poiché vuole entrare nell'altra. E lo scopo di questa attesa è
dare il tempo agli altri singoli nodi di *G*, che potrebbero essere venuti in contatto con l'altra rete quasi
simultaneamente, di raggiungere il nodo Coordinator di *G* con la loro proposta.

Oltre alle informazioni ricevute, il nodo Coordinator della rete *G* conosce il livello più basso `int max_lvl`
tale che la rete *G* è composta da un solo g-nodo a quel livello. Questo valore è utile, perché
se si raggiunge la prenotazione di una posizione a questo livello, l'intera rete *G* può entrare in *J*
in blocco. Non è quindi necessario chiedere di più.

Consideriamo ora le implicazioni della topologia della rete. I due nodi *n* e *v* si sono
incontrati; il nodo *n* ha preso visione della topologia della rete *J* e l'ha confrontata con
la topologia di *G*; ha deciso che *G* dovrebbe cercare di fare ingresso in *J*. Nel prendere questa
decisione ha ponderato i vantaggi che avrebbe *G* ad entrare in *J* (probabilmente più grande o
forse con altre caratteristiche positive) ma anche gli svantaggi. Questo ha a che vedere con la
topologia delle due reti.  
Supponiamo che la rete *G* possa entrare in blocco nella rete *J*. Questo vorrebbe dire che
in pratica *G* non subisce alcuna mutazione, cioè i suoi nodi non hanno alcun danno dalla
procedura di ingresso in *J*. Questo implica necessariamente che la topologia di *J* è identica
alla topologia di *G* almeno ai livelli inferiori a `max_lvl`.  
Se invece, anche se le topologie fossero identiche, la rete *G* non potesse entrare in blocco nella
rete *J* formando un nuovo g-nodo di livello `max_lvl` (nemmeno con delle migration-path, cioè se il grafo di *J* al livello `max_lvl` è
*pieno*) allora per fare ingresso la rete *G* cercherà un livello più basso *l* al quale sarà possibile
far entrare i g-nodi di *G* uno alla volta dentro *J*. In questo caso *G* subisce una mutazione
e i suoi nodi un danno: infatti i nodi di *G* di ogni g-nodo di livello *l* perderanno eventuali connessioni
aperte con i nodi di *G* di altri g-nodi di livello *l*.  
Questo tipo di operazioni avviene anche quando la rete *G* vuole entrare nella rete *J* che ha
una diversa topologia. Il livello più grande in cui un singolo g-nodo di *G* può entrare in
blocco in *J* è quello tale che le topologie sono identiche a tutti i livelli inferiori. Quindi
il nodo *n* quando incontra *v* e vede la topologia di *J* deve valutare quanto sia conveniente
per *G* entrare in *J* prima di decidere.

Considerando che la topologia di *G* e quella di *J* possono essere diverse, introduciamo questa regola:
quando il nodo Coordinator della rete *G* esegue il metodo `evaluate_enter` del modulo X esso conosce sia la topologia
di *G* sia la topologia di *J*. Il valore di `max_lvl` precedentemente calcolato deve essere
tale che la topologia di *G* e quella di *J* sono identiche ai livelli inferiori di `max_lvl`;
altrimenti il suo valore va abbassato.  
Alla fine se ancora `max_lvl` ≥ `min_lvl` non ci saranno problemi.  
Se invece `min_lvl` > `max_lvl` allora sarà possibile in questo caso impostare `max_lvl` = `min_lvl` e
proseguire. Infatti in questo caso specifico l'unico g-nodo a fare ingresso sarebbe quello che
costituisce la rete privata a gestione autonoma di indirizzi e routing della quale il nodo *n*
è il gateway. Quindi la topologia usata in *J* ai livelli inferiori a `min_lvl` non è di alcun
interesse per quel g-nodo.

Il nodo Coordinator dell'intera rete *G* non ha particolari informazioni sui g-nodi (di livello inferiore a
`levels`) a cui appartiene *n*. Ma tali informazioni comunque non gli sono necessarie. Esso infatti dovrà
solo individuare, sulla base delle informazioni ricevute circa i g-nodi di *v* in *J*, un livello
a cui cercare di far entrare un g-nodo di *n* dentro un g-nodo di *v*.

Ora il modulo X nel nodo Coordinator di *G* si chiede: se il solo punto di contatto fra *G* e *J*
fosse il nodo *n* con il suo arco verso il nodo *v*, di cui conosciamo la visione della rete *J*,
assumendo che il fatto di cercare di far entrare *G* dentro *J* sia dato per buono,
a quale livello compreso tra `min_lvl` e `max_lvl` andrebbe tentato l'ingresso di (una parte di) *G* in *J*?

**TODO** Inserire qui ogni idea su some rispondere alla domanda.

Diciamo che la risposta alla domanda sia *lvl_0*.

Assumiamo che questa richiesta sia la prima pervenuta che coinvolge il g-nodo *g<sub>lvl_0</sub>(n)*
e la rete *J*. Il modulo X se ne avvede accedendo alla memoria condivisa di tutta la rete (spiegheremo
meglio il significato di questo a breve).  
Allora il modulo X nel nodo Coordinator di *G* associa alla richiesta `evaluate_enter_data`, al nodo *n*
e alla valutazione *lvl_0* una scadenza `timeout = global_timeout da ora`, rappresentata con un oggetto Timer serializzabile.  
Cioè crea una istanza `EvaluateEnterEvaluation resp` con i campi `evaluate_enter_data`, `client_address`,
`lvl`, `timeout` e altri campi che dettaglieremo in seguito. In seguito potremmo riferirci a questa
struttura dati con il termine *valutazione*.

Ogni *valutazione* può trovarsi in un particolare stato, anche questo memorizzato nella suddetta struttura
dati nel membro `status`. Lo stato in cui viene inizializzata questa valutazione è "*pending*".

Questa valutazione deve essere memorizzata nella memoria condivisa di tutta la rete *G*.  
Abbiamo già detto che il modulo X può fare in modo che venga richiamato un metodo nel modulo Coordinator, pur non
avendo una dipendenza diretta sul modulo Coordinator. In particolare avremo una coppia di metodi
`get_network_entering_memory` e `set_network_entering_memory` con i quali si recupera e si salva
una istanza di Object (perché il modulo Coordinator non conosce i dati del modulo X) che costituisce
l'intera base dati (cioè la memoria condivisa della rete) relativa agli aspetti gestiti dal modulo X.
In particolare il metodo `set_network_entering_memory` provvede anche ad avviare in una nuova tasklet
le operazioni di replica.  
Il modulo X nel nodo Coordinator di *G* recupera l'intera base dati corrente e la integra con
l'aggiunta di questa nuova *valutazione*. Poi immediatamente salva l'intera base dati. Questa
sequenza di operazioni si può a buon diritto considerare come atomica, in quanto seppure facciamo
uso dei meccanismi dei servizi peer-to-peer (modulo PeerServices) il solo nodo che ha diritto
a fare le richieste relative a questi metodi è lo stesso nodo Coordinator di tutta la rete,
quindi non dovrà essere fatta alcuna operazione bloccante di trasmissione in rete.  
In seguito ci riferiremo a questa sequenza di operazioni semplicemente dicendo che
il modulo X nel nodo Coordinator di *G* accede e/o aggiorna la memoria condivisa di tutta la rete con delle
informazioni di sua pertinenza.

Ora il modulo X nel nodo Coordinator di *G* risponde al client *n* con una eccezione AskAgainError.

L'eccezione AskAgainError ricevuta sulla chiamata del metodo `evaluate_enter` sul modulo Coordinator
istruisce il modulo X di ripetere la stessa richiesta (con le stesse informazioni
tra cui lo stesso `evaluate_enter_id`) dopo aver atteso alcuni istanti.  
Questa attesa deve essere più piccola (almeno 3 o 4 volte) di quella calcolata come `global_timeout`,
che come abbiamo detto può essere calcolata dal modulo X esclusivamente sulla base del numero di singoli nodi presenti in *G*.

* * *

Supponiamo che nel frattempo giunga al Coordinator della rete *G* una richiesta simile dal nodo *q*
relativa alla rete *J*. Eseguendo il metodo `evaluate_enter` del modulo X per la richiesta pervenuta
da *q*, alla domanda "a quale livello andrebbe tentato l'ingresso dal nodo *q*" il modulo X risponde
con il livello *lvl_1*.

Ora il modulo X nel nodo Coordinator di *G* accedendo alla memoria condivisa della rete scopre che esiste una precedente
*valutazione* di ingresso in *J* che ancora è *pending*. Allora confronta i due g-nodi coinvolti
e scopre che il g-nodo *g<sub>lvl_1</sub>(q)* interseca (è equivalente a, oppure contiene, oppure è contenuto in)
il g-nodo *g<sub>lvl_0</sub>(n)*. Il modulo X nel nodo Coordinator di *G* deduce che queste *valutazioni*
(quella sulla richiesta di *n* e quella sulla richiesta di *q*) vanno considerate insieme perché sono intersecanti
e riguardano la stessa rete *J*. Le due *valutazioni* risultano ora collegate fra di loro. Anche questo collegamento farà parte della
memoria condivisa di tutta la rete: ogni *valutazione* mantiene un membro `next_id` che coincide con
il membro `evaluate_enter_data.evaluate_enter_id` della prossima collegata.

Le *valutazioni* tra loro collegate devono avere sempre la medesima scadenza. Se `lvl_1` è maggiore di `lvl_0`, ovvero più in generale, se il
livello del g-nodo coinvolto nella *valutazione* della richiesta appena pervenuta è maggiore del livello del g-nodo coinvolto in tutte
le *valutazioni* ad essa collegate, allora il modulo X nel nodo Coordinator di *G* computa una nuova
scadenza `timeout = global_timeout da ora` e la aggiorna su tutte le *valutazioni* collegate. Altrimenti esso
mantiene la precedente scadenza (comune a tutte le *valutazioni* precedenti) e la usa anche per
la *valutazione* sulla richiesta di *q*.

Il modulo X nel nodo Coordinator di *G* aggiorna la memoria condivisa di tutta la rete con tutte queste variazioni.

Ora il modulo X nel nodo Coordinator di *G* si accinge a rispondere alla richiesta di *q*.
Se nella *valutazione* sulla richiesta di *q* il timer `timeout` non è ancora scaduto il Coordinator risponde anche a questa richiesta con
una eccezione AskAgainError.

Se invece `timeout` è scaduto si passa alla terza fase.

### Terza fase - elezione dell'ingresso

Consideriamo che ogni volta che arriva una richiesta `r` di ingresso in *J* il modulo X nel
nodo Coordinator di *G* prima di valutarla accede alla memoria condivisa di tutta la rete per
vedere se a tale richiesta è stata già data una *valutazione*.  
Una *valutazione* `v` recuperata dalla memoria condivisa della rete è sempre identificabile
come quella associata ad una particolare richiesta `r` appena pervenuta verificando che
`r.evaluate_enter_id == v.evaluate_enter_data.evaluate_enter_id`.

Alla fine arriverà una richiesta `req` di ingresso in *J* tale che il modulo X nel nodo Coordinator di *G* ne assocerà
la *valutazione* `resp` ad un gruppo di *valutazioni* nello stato *pending* il cui `timeout` è scaduto. A questo punto il modulo X eleggerà
la migliore fra le soluzioni.

**TODO** Inserire qui ogni idea su some individuare la migliore soluzione. Ancora non abbiamo
avviato alcuna ricerca di migration-path nella rete *J*.

**Nota 1**: una cosa da evitare è quella di formare g-nodi che facilmente potrebbero "splittarsi", diventare
disconnessi.  
Per questo in `EvaluateEnterData evaluate_enter_data` memoriziamo `List<int> neighbor_pos` la posizione
del vicino con cui c'è un arco. Il modulo X nel nodo Coordinator di *G* accedendo nella memoria condivisa
alla lista di *valutazioni* vede che un certo numero di queste ha per nodo vicino un nodo che appartiene
al g-nodo di *J* in cui *G* vorrebbe entrare. Deduce quindi il numero di archi che dovrebbero rompersi
per far sì che il g-nodo divenga disconnesso.

Diciamo che la soluzione eletta sia la *valutazione* `v`.

La *valutazione* eletta `v` passa nello stato "*eletta, da comunicare*" e la sua nuova scadenza viene
valorizzata con `timeout = global_timeout da ora`. Tutte le altre *valutazioni* collegate passano nello
stato "*riconsiderabile*" con scadenza `timeout = global_timeout da ora`.

Il modulo X nel nodo Coordinator di *G* aggiorna la memoria condivisa di tutta la rete con tutte queste variazioni.

Ora il modulo X guarda alla richiesta `req` appena pervenuta. Se la relativa *valutazione* `resp` è proprio `v`
allora il modulo X nel nodo Coordinator di *G* fa queste operazioni:

*   Si prepara a rispondere alla richiesta del client con il livello a cui deve fare ingresso. Cioè memorizza `ret = resp.lvl`.
*   La *valutazione* eletta `v` viene rimossa dall'elenco, operando sui campi `next_id` delle altre *valutazioni* oltre che
    sulle strutture dati della memoria condivisa.
*   Tutte le *valutazioni* collegate passano nello stato "*scartata, da comunicare*" con scadenza `timeout = global_timeout da ora`.
*   Aggiorna la memoria condivisa di tutta la rete.
*   Risponde alla richiesta del client con `ret`.

Altrimenti il modulo X fa queste operazioni:

*   Risponde alla richiesta del client con l'eccezione AskAgainError.

Le successive richieste saranno gestite nella quarta fase.

### Quarta fase - comunicazione della elezione

Quando arriva una richiesta `req` di ingresso in *J* il modulo X nel nodo Coordinator di *G* si avvede che si trova nella quarta fase
perché la sua *valutazione* `v` era stata già fatta ed è nella memoria condivisa di tutta la rete nello
stato *da comunicare* o *riconsiderabile*.

Se `v` è nello stato *eletta, da comunicare*
allora il modulo X nel nodo Coordinator di *G* fa queste operazioni:

*   Si prepara a rispondere alla richiesta del client con il livello a cui deve fare ingresso. Cioè memorizza `ret = v.lvl`.
*   La *valutazione* eletta `v` viene rimossa dall'elenco, operando sui campi `next_id` delle altre *valutazioni* oltre che
    sulle strutture dati della memoria condivisa.
*   Tutte le *valutazioni* collegate passano nello stato "*scartata, da comunicare*" con scadenza `timeout = global_timeout da ora`.
*   Aggiorna la memoria condivisa di tutta la rete.
*   Risponde alla richiesta del client con `ret`.

Altrimenti, se `v` è nello stato *riconsiderabile* il modulo X fa queste operazioni:

*   Se `v.timeout` è scaduto:
    *   Cerca fra le *valutazioni* collegate quella nello stato *eletta, da comunicare* e la rimuove dall'elenco.
    *   Tutte le altre *valutazioni* le mette nello stato *pending* mantenendone immutato il `timeout`.
    *   Il modulo X ricomincia dalla terza fase: cioè si trova a dover eleggere la migliore fra le soluzioni
        collegate a questa.
*   Altrimenti:
    *   Risponde alla richiesta del client con l'eccezione AskAgainError.

Altrimenti, se `v` è nello stato *scartata, da comunicare* il modulo X fa queste operazioni:

*   La *valutazione* `v` viene rimossa dall'elenco.
*   Cicla fra le *valutazioni* collegate e se ne trova qualcuna il cui `timeout` è scaduto la rimuove dall'elenco.  
    Le scadenze dovrebbero giungere tutte insieme, quindi in pratica svuota l'elenco.
*   Aggiorna la memoria condivisa di tutta la rete.
*   Risponde alla richiesta del client con l'eccezione IgnoreNetworkError.

L'eccezione IgnoreNetworkError ricevuta sulla chiamata del metodo `evaluate_enter` sul modulo Coordinator
istruisce il modulo X nel nodo *n* di non prendere alcuna iniziativa e di evitare ulteriori valutazioni di ingresso nella
rete tramite il diretto vicino *v* per un certo tempo.  
Questo tempo potrebbe essere un multiplo (diciamo 20 volte tanto) di quello calcolato come `global_timeout`,
che come abbiamo detto può essere calcolata dal modulo X esclusivamente sulla base del numero di singoli nodi presenti in *G*.

### Quinta fase - comunicazione con il g-nodo entrante

La quinta fase inizia quando un singolo nodo di *G*, assumiamo sia il nodo *n*, riceve l'autorizzazione
dal Coordinator di *G* di tentare l'ingresso in *J* tramite il suo vicino *v* con il suo g-nodo di livello
*lvl*.

**TODO**

## Altre annotazioni

Quando due reti si incontrano e la rete più piccola *G* decide di entrare in *J* il primo tentativo
è quello di entrare in blocco. Cioè con il livello più piccolo tale che *G* è costituita da un solo
g-nodo. Quello che abbiamo chiamato `max_lvl`.

Assumiamo che il nodo *n* di *G* vuole usare il nodo *v* di *J* per far entrare il suo g-nodo *g* di livello *l*
in blocco dentro *J*.

Il nodo *n* per prima cosa richiede a *v*: "trova la shortest migration-path che permetta
al mio g-nodo di livello *l* di essere connesso attraverso l'arco *n - v* dentro il tuo *attuale* g-nodo di
livello *l* + 1".  
Il nodo *n* e il nodo *v* devono avere entrambi un indirizzo completamente *reale*.  
Indichiamo con *h* l'attuale g-nodo di livello *l* + 1 di *v*.  
È evidenziato nella richiesta il fatto che *g* vuole entrare dentro *l'attuale* g-nodo di *v*, perché
come risultato della migration-path lo stesso nodo *v* potrebbe migrare lasciando dentro *h* la sua identità
di connettività come link per *g*.

Specifichiamo rigorosamente cosa si intende per migration-path.

### Migration Path

Prendiamo liberamente in prestito alcune notazioni dal documento delle migrazioni.

Sia *h* un g-nodo di livello *l* + 1, con *l* da 0 a *levels* - 1. È possibile che sia *size<sub>l</sub>(h)* = *gsizes(l)*. Cioè *h* può essere saturo.

Definiamo *P* una migration path a livello *l* che parte dal g-nodo *h* (di livello *l* + 1) se *P* è una lista di g-nodi che soddisfa questi requisiti:

*   *P* = (*p<sub>1</sub>*, *p<sub>2</sub>*, ... *p<sub>m</sub>*).
*   *p<sub>1</sub>* = *h*. Se *m* = 1, quindi *p<sub>1</sub>* = *p<sub>m</sub>*, allora *p<sub>1</sub>* può essere *h* o un g-nodo di livello maggiore che contiene *h*.
*   Per ogni *i* da 1 a *m* - 1:
    *   **Nota** può essere *m* = 1, quindi questo ciclo non viene mai valutato.
    *   *lvl(p<sub>i</sub>)* = *lvl(h)* = *l* + 1.
    *   *p<sub>i+1</sub>* ∈ *𝛤<sub>l+1</sub>(p<sub>i</sub>)*, cioè *p<sub>i+1</sub>* è direttamente collegato a *p<sub>i</sub>* nel grafo *[G]<sub>l+1</sub>*.
    *   *size<sub>l</sub>(p<sub>i</sub>)* = *gsizes(l)*, cioè *p<sub>i</sub>* è saturo.
*   *lvl(p<sub>m</sub>)* = *k* ≥ *lvl(h)* = *l* + 1. Indichiamo con *k* il livello di *p<sub>m</sub>*.
*   *size<sub>k-1</sub>(p<sub>m</sub>)* `<` *gsizes(k-1)*, cioè *p<sub>m</sub>* non è saturo.

Usiamo una definizione che include anche una sorta di migration-path impropria, quella di lunghezza 0.

### Uso della migration path

Detto in altri termini, il nodo *v* deve cercare la shortest migration-path a livello *l* che parte da *h*. Cioè:

*   un posto già libero nel suo g-nodo di livello *l* + 1; *oppure*
*   un posto già libero in un suo g-nodo di livello maggiore; *oppure*
*   la shortest migration-path per liberare un posto nel suo g-nodo di livello *l* + 1.

Usare questa migration-path significa far migrare un border g-nodo di livello *l* da ognuno dei g-nodi
di livello *l* + 1 della lista *P* nel successivo in modo tale da liberare un posto nel primo g-nodo
di livello *l* + 1 della lista *P*. Come *costo* abbiamo che viene occupato un ulteriore posto nell'ultimo g-nodo
della lista *P*, il quale non era saturo, ma anche poteva essere di livello maggiore di *l* + 1.

### Caratteristiche della migration path

Una migration-path a livello *l* può essere descritta da 2 principali caratteristiche:

*   *d* - distanza, cioè numero di migrazioni (di g-nodi di livello *l*) necessarie, con *d* ≥ 0.
*   *hl* - host g-node level, cioè il livello dell'ultimo g-nodo della lista, con *levels* ≥ *hl* > *l*.

Da un lato dello spettro abbiamo il caso ottimale: *d* = 0 e *hl* = *l* + 1. Cioè per far entrare il g-nodo
*g* non occorre nessuna migrazione e basta occupare una nuova posizione che era libera in un g-nodo esistente
di livello *l* + 1.

Si intuisce facilmente che allontanandoci da questa situazione, ovvero con il crescere di *d* e/o con
il crescere del delta *hl* - *l*, andiamo a fare operazioni che danneggiano maggiormente la nuova rete *J*.

Con il crescere di *d* cresce il numero di g-nodi che cambiano indirizzo, quindi è ovvio il danno.

Più sottile è il danno con il crescere del delta *hl* - *l*, ma non indifferente. Partiamo ad esempio
da un g-nodo di livello 2 saturo. Un nuovo nodo si aggiunge e questo può, senza obbligare a nessuna migrazione,
occupare un nuovo g-nodo di livello 2 dentro il g-nodo di livello 3 che diventa saturo. Un nuovo nodo si
aggiunge e ha un solo link con il g-nodo di livello 2 saturo; che è dentro un g-nodo di livello 3 saturo;
quindi può, senza obbligare a nessuna migrazione, occupare un nuovo g-nodo di livello 3 dentro il g-nodo di
livello 4 che diventa saturo. Un nuovo nodo si aggiunge e ha un solo link con il g-nodo di livello 2 saturo;
che è dentro un g-nodo di livello 3 saturo; che è dentro un g-nodo di livello 4 saturo;
quindi può, senza obbligare a nessuna migrazione, occupare un nuovo g-nodo di livello 4 dentro il g-nodo di
livello 5 che diventa saturo. E così via, con pochi nodi si rende saturo un g-nodo di alto livello
e si forma una rete poco bilanciata.  
Si potrebbe pensare che sarebbe conveniente quando il delta supera un certo *𝜀* investigare per
cercare una migration-path che, sebbene non sia la più breve come *d* faccia occupare un nuovo posto
all'interno di un g-nodo di livello *hl* non troppo alto.  
Ad esempio un nuovo nodo si aggiunge e ha un solo link con il g-nodo di livello 2 saturo; che è dentro un
g-nodo di livello 3 saturo; sebbene questo sia dentro un g-nodo di livello 4 che non è saturo
si preferisce cercare una diversa migration path e si trova che facendo migrare un singolo nodo da un g-nodo
di livello 1 dentro un altro g-nodo esistente sempre di livello 1 si libera un posto per il nuovo nodo.

Si procede in questo modo: la prima ricerca della shortest migration-path si fa senza porre alcun limite
superiore a *lh*. Trattandosi di una ricerca *breadth-first* o *in ampiezza* questa si ferma alla prima
shortest migration-path: cioè se dovessimo ripetere la ricerca dall'inizio ponendo dei parametri di ricerca
più restrittivi di sicuro avremmo un risultato la cui distanza *d* non può migliorare.

Quindi dalla prima ricerca otteniamo una migration-path con distanza *d0* e con livello del g-nodo
non saturo *hl0*. Se *hl0* risulta soddisfacente ci fermiamo. Altrimenti riavviamo una nuova ricerca
ponendo un limite superiore a *hl* minore di *hl0*, decrementando di 1. Se la ricerca non trova
una migration-path ci fermiamo; se invece trova una migration path avremo distanza *d1* e livello ospite *hl1*.
Di nuovo, se *hl1* risulta soddisfacente ci fermiamo. Altrimenti riavviamo una nuova ricerca
ponendo un limite superiore a *hl* minore di *hl1*, decrementando di 1. Se la ricerca non trova
una migration-path ci fermiamo; se invece trova una migration path avremo distanza *d2* e livello ospite *hl2*.
E così via fino a quando non ci fermiamo perché il delta è minore di *𝜀* o perché non esistono
migration-path con il delta minore dell'ultimo *hl*.  
Quando ci siamo fermati avremo un insieme di soluzioni (che potrebbe essere vuoto) tra cui scegliere.

Queste prime ricerche *esplorative* non mettono un blocco sulla rete, che quindi potrebbe anche evolversi nel
frattempo. Ma accettiamo questa possibilità. Una volta scelta una migration path sulla base della
ricerca esplorativa appena fatta, si procede con una nuova ricerca in ampiezza con gli stessi
parametri (cioè con o senza limite superiore) che però sarà *esecutiva*, confidando che l'esito
non cambierà di molto.

