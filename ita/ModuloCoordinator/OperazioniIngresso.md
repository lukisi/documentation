**NOTA**
Il contenuto di questo file andrà tasferito nella trattazione di un diverso modulo.

## <a name="prepare_enter"></a>Prima fase - prepare_enter

Si consideri un nodo *n* che appartiene alla rete *G*. Questa è una generalizzazione,
che comprende ad esempio il caso di un singolo nodo che compone una intera rete.

Il modulo X del nodo *n* che appartiene alla rete *G* si avvede del vicino *v* di altra rete *J*.

Il modulo X del nodo *n* chiede e ottiene dal vicino *v* una struttura dati che descrive *J* come è vista da *v*.  
Questa struttura contiene:

*   `int64 netid` = Identificativo della rete *J*.
*   `List<int> gsizes` = Lista che descrive la topologia della rete *J*. Da essa si ricava `levels`.
*   `List<int> gnode_n_nodes` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_n_nodes[i-1]` è il
    numero di singoli nodi dentro *g<sub>i</sub>(v)*.
*   `List<int> gnode_pos` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_pos[i-1]` è la
    posizione al livello `i-1` di *v* dentro *g<sub>i</sub>(v)*.
*   `List<int> gnode_n_free_pos` = Per i valori *i* da 1 a `levels`, l'elemento `gnode_n_free_pos[i-1]` è il
    numero di posizioni libere dentro *g<sub>i</sub>(v)*.

Poi quel modulo confronta la descrizione di *J* data da *v* con la rete *G* come è vista da *n*
e decide, assumiamo per ipotesi, che *G* deve entrare in *J*.

**Importante** Questa prima disamina delle caratteristiche di *J* e di *G* avviene sulla base delle
conoscenze di *n* e di *v*. Non va bene importunare il Coordinator della rete (che può essere anche
grande) ogni qualvolta un singolo nodo della rete incontra un vicino che non appartiene ancora alla
rete. Infatti può essere subito evidente dalle conoscenze dei singoli nodi che si vengono ad
incontrare quale sia la rete più piccola, quella cioè che cercherà un posto all'interno dell'altra.

Allora il modulo X aggiunge un'altra informazione a quelle della struttura dati di cui sopra:

*   `int minimum_lvl` = Livello minimo a cui il singolo nodo *n* è disposto a fare ingresso. Infatti il nodo *n*
    potrebbe essere un gateway verso una rete privata in cui si vogliono adottare diversi meccanismi di
    assegnazione di indirizzi e routing. In questo caso il gateway potrebbe volere una assegnazione di un
    g-nodo di livello tale (considerando la topologia della rete *J*) da poter disporre di un certo spazio
    (numero di bit) per gli indirizzi interni.

Ora il modulo X del nodo *n* prepara una nuova struttura dati con alcune (quasi tutte) delle informazioni di cui sopra
istanziando un `PrepareEnterData prepare_enter_data`.  
La classe PrepareEnterData è nota al modulo X. Si tratta di una classe serializzabile. I membri di questa classe sono:
`int64 netid`, `List<int> gsizes`, `List<int> n_nodes`, `List<int> n_free_pos`, `int min_lvl`.  
L'istanza `prepare_enter_data` andrà passata ad un metodo del modulo X nel nodo Coordinator della rete.

Ora il modulo X del nodo *n* fa in modo che venga richiamato nel modulo Coordinator (dal suo utilizzatore diretto, poiché
non è detto che il modulo X abbia una dipendenza diretta sul modulo Coordinator) il metodo `prepare_enter`.  
A questo metodo viene passato un `Object prepare_enter_data`.  
La reale classe che implementa questa struttura dati non è infatti nota al modulo Coordinator. Questi sa solo
che è serializzabile.

Grazie ai meccanismi del modulo Coordinator (di cui è trattato nella relativa documentazione) ora
nel nodo Coordinator della rete *G* viene chiamato dallo stesso modulo Coordinator (a cui era
stato passato come delegato nel suo costruttore) il metodo `prepare_enter` del modulo X.  
Va considerato che, sempre grazie ai meccanismi del modulo Coordinator, oltre alla struttura dati
`prepare_enter_data` il metodo `prepare_enter` eseguito sul nodo Coordinator di *G* riceve
come argomento anche l'indirizzo (la lista delle posizioni ai vari livelli) di *n*.

Vediamo cosa avviene nel metodo `prepare_enter` eseguito sul nodo Coordinator di *G*.

Oltre alle informazioni ricevute, il nodo Coordinator della rete *G* conosce il livello più basso `int max_lvl`
tale che la rete *G* è composta da un solo g-nodo a quel livello. Questo valore è utile, perché
se si raggiunge la prenotazione di una posizione a questo livello, l'intera rete *G* può entrare in *J*
in blocco. Non è quindi necessario chiedere di più.

Probabilmente almeno inizialmente potremmo assumere che se i due nodi *n* e *v* delle reti *G* e *J*
decidono di avviare questa fusione significa che le topologie di *G* e di *J* sono identiche.  
Ma anche se così non fosse, dovremmo riuscire a gestire il caso con questa semplice regola: quando
il nodo Coordinator della rete *G* esegue il metodo `prepare_enter` esso conosce sia la topologia
di *G* sia la topologia di *J*. Il valore di `max_lvl` precedentemente calcolato deve essere
tale che la topologia di *G* e quella di *J* sono identiche ai livelli inferiori di `max_lvl`;
altrimenti il suo valore va abbassato.  
Alla fine se ancora `max_lvl` ≥ `min_lvl` non ci saranno problemi.  
Se invece `min_lvl` > `max_lvl` allora sarà possibile in questo caso impostare `max_lvl` = `min_lvl` e
proseguire. Infatti in questo caso specifico l'unico g-nodo a fare ingresso sarebbe quello che
costituisce la rete privata a gestione autonoma di indirizzi e routing della quale il nodo *n*
è il gateway. Quindi la topologia usata in *J* ai livelli inferiori a `min_lvl` non è di alcun
interesse per quel g-nodo, come non lo era la topologia usata in *G*.

Il nodo Coordinator dell'intera rete *G* non ha particolari informazioni sui g-nodi (di livello inferiore a
`levels`) a cui appartiene *n*. Ma tali informazioni comunque non gli sono necessarie. Esso infatti dovrà
solo individuare, sulla base delle informazioni ricevute circa i g-nodi di *v* in *J*, un livello
a cui cercare di far entrare un g-nodo di *n* dentro un g-nodo di *v*.

Ora il modulo X nel nodo Coordinator di *G* si chiede: se il solo punto di contatto fra *G* e *J*
fosse il nodo *n* con il suo arco verso il nodo *v*, di cui conosciamo la visione della rete *J*,
assumendo che il fatto di cercare di far entrare *G* dentro *J* sia dato per buono,
a quale livello compreso tra `min_lvl` e `max_lvl` andrebbe tentato l'ingresso di (una parte di) *G* in *J*?

**TODO** Inserire qui ogni idea su some rispondere alla domanda.

Diciamo che la risposta alla domanda sia *lvl*.


