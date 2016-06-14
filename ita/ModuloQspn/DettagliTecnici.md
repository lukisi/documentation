# Modulo QSPN - Dettagli Tecnici

1.  [Terminologia](#Terminologia)
1.  [Inizializzazione del modulo](#Inizializzazione_del_modulo)
1.  [Requisiti e Deliverable](#Requisiti_deliverable)
    1.  [Avvio](#Avvio)
    1.  [Ingresso](#Ingresso)
        1.  [Creazione di una identità nella rete G e dismissione della precedente](#Creazione_in_g_dismissione_precedente)
    1.  [Migrazione](#Migrazione)
        1.  [Creazione di una identità in h](#Creazione_in_h)
        1.  [Modifica della precedente identità in g in una identità di connettività](#Modifica_precedente)
1.  [Ulteriori deliverable](#Ulteriori_deliverable)
1.  [Funzioni di appoggio](#Funzioni_di_appoggio)
1.  [Produzione e trasmissione di ETP](#Produzione_trasmissione_etp)
    1.  [Metodi remoti](#Metodi_remoti)
    1.  [Produzione del primo ETP](#Produzione_primo_etp)
    1.  [Processazione di un set di ETP](#Processazione_set_etp)
    1.  [Rielaborazione dei percorsi in ogni ETP ricevuto](#Rielaborazione_percorsi_etp_ricevuto)
    1.  [Variazioni nella mappa di n](#Variazioni_mappa)
        1.  [Algoritmo](#Variazioni_mappa_Algoritmo)
    1.  [Variazioni nei propri g-nodi](#Variazioni_propri_gnodi)
        1.  [Algoritmo](#Variazioni_propri_gnodi_Algoritmo)
    1.  [Produzione di percorsi da mettere in un ETP da inviare](#Produzione_percorsi_etp_da_inviare)

Il presente documento assume che siano stati letti il documento di [Analisi Funzionale](AnalisiFunzionale.md) e
quelli in esso indicati ([Esplorazione della rete](EsplorazioneRete.md) e [Percorsi disgiunti](PercorsiDisgiunti.md)).

## <a name="Terminologia"></a>Terminologia

Diamo un significato ad alcuni termini che useremo frequentemente nel resto del documento.

Invece di dire "l'utilizzatore del modulo QSPN" in ottica di uso generico, nel resto del documento ci riferiremo
direttamente al "demone *ntkd*" per brevità.

Sappiamo che il demone *ntkd* è in esecuzione su un *sistema*. In tale sistema possono esistere
diverse *identità*, ognuna delle quali costituisce una singola entità della rete Netsukuku, detta anche *nodo del grafo*
o più brevemente *nodo*.

Per una dettagliata analisi del concetto di *identità* e delle sue caratteristiche rimandiamo al documento di
trattazione del modulo Identities.

C'è una relazione di equivalenza tra una *identità* nel sistema e una istanza della classe QspnManager.
A volte ci riferiremo ad una istanza di QspnManager, sottintendendo che questa si riferisce ad una specifica
identità che vive nel sistema.

## <a name="Inizializzazione_del_modulo"></a>Inizializzazione del modulo

Il modulo QSPN fa uso delle [tasklet](../Librerie/TaskletSystem.md), un sistema di multithreading cooperativo.

Il modulo QSPN fa uso del framework [ZCD](../Librerie/ZCD.md), precisamente appoggiandosi ad una libreria intermedia
prodotta con questo framework per formalizzare i metodi remoti usati nel demone *ntkd*.

Il modulo QSPN è un modulo *di identità*. In un singolo sistema il demone *ntkd* può creare diverse istanze della classe
QspnManager, ognuna gestita da una sua identità. Alcune operazioni di inizializzazione del modulo sono fatte una sola
volta, per questo usano metodi statici della classe QspnManager.

Il demone *ntkd* per prima cosa inizializza il modulo QSPN richiamando il metodo statico *init* di QspnManager.

Nel metodo *init* viene passata l'istanza di ITasklet per fornire l'implementazione del sistema di tasklet.

Inoltre vengono passati:

*   `int max_paths` - Il numero massimo di percorsi per destinazione da memorizzare.
*   `float max_common_hops_ratio` - Il massimo rapporto di hops comuni nella verifica di disgiunzione.
*   `int arc_timeout` - Tempo di rilevamento degli archi in millisecondi.
*   `IQspnThresholdCalculator threshold_calculator` - Il calcolatore del tempo di tolleranza.

## <a name="Requisiti_deliverable"></a>Requisiti e Deliverable

Esaminiamo nel dettaglio i requisiti (forniti dal demone *ntkd*) e i deliverable del modulo QSPN in questi possibili scenari:

*   **Avvio** come singolo nodo. Quando un nodo si avvia viene creata la sua *identità principale* ed essa
    forma da sola una "rete".  
    Il demone *ntkd* crea una istanza di QspnManager specificando che si tratta di questo scenario. In questo
    caso al costruttore viene passato un indirizzo *reale* generato in modo del tutto arbitrario.
*   **Ingresso** nella rete di un singolo nodo del grafo o di un g-nodo.  
    Se si tratta di un singolo nodo del grafo allora il demone *ntkd* individua l'istanza di QspnManager che
    ad esso si riferisce.  
    Se si tratta di un g-nodo allora il demone *ntkd* individua una o più istanze di QspnManager che si riferiscono
    a nodi del grafo appartenenti al dato g-nodo.  
    Per ogni istanza di QspnManager individuata, il demone *ntkd* :
    *   Crea una nuova istanza di QspnManager specificando che si tratta di questo scenario e indicando la
        precedente istanza. In questo caso al costruttore viene passato un nuovo indirizzo valido nella nuova
        rete. Questa identità sarà di tipo analogo (*principale* o *di connettività*) a ciò che era la precedente identità.
    *   Dismette la precedente istanza di QspnManager.
*   **Migrazione** (di un singolo nodo del grafo o di un g-nodo) da un g-nodo di livello *i* ad un altro
    g-nodo (sempre di livello *i*) il quale appartiene a g-nodi distinti da quelli del primo fino al livello *j*.  
    Di nuovo, se si tratta di un singolo nodo del grafo allora il demone *ntkd* individua l'istanza di
    QspnManager che ad esso si riferisce.  
    Se si tratta di un g-nodo allora il demone *ntkd* individua una o più istanze di QspnManager che si
    riferiscono a nodi del grafo appartenenti al dato g-nodo.  
    Per ogni istanza di QspnManager individuata, il demone *ntkd* :
    *   Crea una nuova istanza di QspnManager specificando che si tratta di questo scenario e indicando la
        precedente istanza. In questo caso al costruttore viene passato un nuovo indirizzo valido nel nuovo
        g-nodo. Questa identità sarà di tipo analogo (*principale* o *di connettività*) a ciò che era la precedente identità.
    *   Modifica la precedente istanza di QspnManager che ora detiene un indirizzo *virtuale* al livello
        *i* - 1 e diventa una identità *di connettività* ai livelli da *i* a *j*.

### <a name="Avvio"></a>Avvio

Il demone *ntkd* nel sistema *n* produce in  modo completamente arbitrario un indirizzo Netsukuku conforme
ad una certa topologia di rete di *l* livelli. Sceglie un identificativo random per il suo fingerprint a livello
0, il quale avrà tutte le anzianità ai vari livelli a zero (nel senso che è il primo nodo, nel primo g-nodo a
livello 1, ... nel primo g-nodo a livello *l* - 1, nella rete).

Il nodo del grafo non ha archi.

Quindi il demone *ntkd* nel sistema *n* istanzia il suo QspnManager passando al costruttore:

*   `QspnManager.create_net` - Tipo di costruttore: creazione di una rete.
*   `IQspnNaddr my_naddr` - Il proprio indirizzo Netsukuku.
*   `IQspnFingerprint my_fingerprint` - Il suo fingerprint come nodo del grafo.
*   `IQspnStubFactory stub_factory` - La stub factory per le comunicazioni con i vicini.

La lista di istanze di IQspnArc del nodo viene inizializzata vuota dal costruttore.

Una istanza di QspnManager costruita in questo modo esce immediatamente dalla fase di bootstrap. Quando una
istanza di QspnManager esce dalla fase di bootstrap lo notifica con il segnale *bootstrap_complete*.

Abbiamo detto che la rete è composta dal solo nodo del grafo *n*. Se in seguito il sistema *n* decidesse di
entrare in una diversa rete, allora farebbe come vedremo sotto una nuova istanza di QspnManager.

Se invece un altro sistema decide di entrare come nodo nel grafo della mia rete, allora il demone *ntkd* nel
sistema *n* segnala le variazioni sugli archi alla sua istanza di QspnManager con questi metodi:

*   `arc_add (IQspnArc arc)`.
*   `arc_is_changed (IQspnArc changed_arc)`.
*   `arc_remove (IQspnArc removed_arc)`.

Quando il QspnManager memorizza i suoi archi, per ognuno genera un identificativo di arco, cioè un intero random
in uno spazio grande. Memorizza in una lista gli archi, accertandosi che non ci siano archi ripetuti. Memorizza e
gestisce inoltre un dizionario (HashMap) che associa identificativi e archi.

Inoltre il modulo non assume che anche il suo utilizzatore mantenga le istanze degli archi. Questo significa che
il QspnManager per vedere se un arco è nella sua lista si basa sul metodo *i_qspn_equals* fornito dall'interfaccia
IQspnArc. Sul metodo *arc_is_changed* sostituisce l'istanza corrente nella sua lista (e nel suo dizionario degli
identificativi) con quella passata (non assumendo che si tratti della stessa istanza) e dovrà fare lo stesso cambio
su tutte le istanze della classe NodePath.

### <a name="Ingresso"></a>Ingresso

Supponiamo che il g-nodo *w* (a cui appartiene il nodo del grafo *n* che vive nel nostro sistema) fa il suo ingresso
in una rete *G*. In questa trattazione è incluso il caso in cui *w* equivale a *n*.

Per l'esattezza, dire che un g-nodo *w* fa ingresso in una rete *G* significa che prenota un posto in un g-nodo *g* ∈ *G* tale che:

*   Il livello di *g* è maggiore del livello di *w*.
*   Il g-nodo *g* non è saturo.
*   Il g-nodo *w* ha almeno un arco diretto verso *g*.

A fronte di diversi g-nodi che potrebbero soddisfare questi requisiti, la strategia che si adotta per la scelta
del g-nodo non è di competenza del modulo QSPN. Viene descritta nel documento del modulo Migrations. **TODO: inserire.**

Se si tratta di un singolo nodo del grafo *n* che vuole entrare nel g-nodo *g* ∈ *G* — deve trattarsi dell'identità
principale del suo sistema — lo stesso nodo *n* chiede ad un suo diretto vicino in *g*, chiamiamolo *g<sub>0</sub>*,
i dati che servono all'ingresso. Questa richiesta da *n* a *g<sub>0</sub>* viene fatta con i metodi remoti di un
modulo *di identità*. Per rispondere, *g<sub>0</sub>* inizia un dialogo con il Coordinator di *g*. Dopo che *n* ha
ricevuto la risposta, il demone *ntkd* crea una nuova istanza di QspnManager basata sulla precedente *n* che
diventa l'identità *principale* del sistema in *g*.

Se si tratta di un g-nodo *w* che contiene diversi nodi del grafo e che vuole entrare in blocco nel g-nodo *g* ∈ *G*,
in qualche modo si è eletto un suo nodo *n<sub>0</sub>* — che deve essere l'identità principale del suo sistema — a
chiedere ad un suo diretto vicino *g<sub>0</sub>* in *g* i dati che servono all'ingresso. Di nuovo, la richiesta da
*n<sub>0</sub>* a *g<sub>0</sub>* viene fatta con i metodi remoti di un modulo *di identità* e, per rispondere,
*g<sub>0</sub>* inizia un dialogo con il Coordinator di *g*. Poi *n<sub>0</sub>* genera un identificativo di
migrazione *migration_id* e propaga questi dati a tutto il g-nodo *w*, anche qui attraverso chiamate a metodo
remoto fatte con un modulo *di identità*. Infine *n<sub>0</sub>* riceve conferma che tutti i nodi del grafo
appartenenti al g-nodo *w* sono stati portati a conoscenza dei dati e del *migration_id*. A questo
punto *n<sub>0</sub>* propaga analogamente a tutto il g-nodo *w* l'ordine di completare la migrazione.

In questo secondo scenario dobbiamo esaminare cosa avviene in ogni sistema in cui vivono dei nodi del grafo che
appartengono a *w*. In particolare in ogni sistema possono vivere una o più istanze di QspnManager (cioè nodi
del grafo) che appartengono a *w* e quindi il demone *ntkd* farà alcune operazioni per ognuna di esse.

Per ogni istanza *n* di QspnManager che apparteneva a *w* il demone *ntkd* crea una nuova istanza di QspnManager
basata sulla precedente *n* che detiene un indirizzo in *g*. Questa nuova identità è di tipo analogo (*principale*
o *di connettività*) a ciò che era *n* in *w*.

In questo modo si produce di fatto un g-nodo *w’* all'interno del g-nodo *g* ∈ *G* che è isomorfo al g-nodo *w* come esisteva prima.

Notiamo che ci può essere il caso di un sistema in cui una istanza *n* di QspnManager è una identità *di connettività* in *w*,
mentre l'identità *principale* del sistema non è in *w* ma in un diverso g-nodo della stessa rete. In questo
caso se *w* entra in blocco in *G* mentre altri g-nodi della vecchia rete non lo hanno ancora fatto, il sistema
in questione ha la sua identità *principale* in una rete distinta da *G* e una sua identità *di connettività* dentro *G*.

#### <a name="Creazione_in_g_dismissione_precedente"></a>Creazione di una identità nella rete G e dismissione della precedente

I dati che ogni identità *n* riceve in relazione a questo ingresso nel g-nodo *g* di livello *i* in *G* sono:

*   L'identificativo di migrazione *migration_id*. Si tratta di un identificativo univoco (un numero random) reso
    noto a tutti i partecipanti alla migrazione.
*   L'indirizzo Netsukuku di *g* ∈ *G* e la sua anzianità e quella dei g-nodi superiori in *G*.
*   La posizione riservata al nuovo g-nodo *w’* in *g* al livello *i* - 1 e la sua anzianità.

Abbiamo già detto che tali informazioni giungono ad una precisa identità *n* che vive nel sistema. Il demone
*ntkd* per ogni identità *n* crea una nuova identità *n’*.

Questa operazione di creazione di una nuova identità viene realizzata dal demone *ntkd* avvalendosi del modulo
*Identities*. Rimandiamo al relativo [documento](../ModuloIdentities/AnalisiFunzionale.md) se si vuole approfondire.
Qui ricordiamo solo che l'operazione avviene in due fasi, con il metodo *prepare_add_identity* e il metodo
*add_identity*. Questi metodi richiedono due dati: l'identificativo di migrazione *migration_id* e l'identità *n*
da duplicare.

Durante le operazioni (che in questo documento non vengono dettagliate) che portano alla propagazione
in tutto il g-nodo *w* (con avviso di ricevimento) delle suddette informazioni, il demone *ntkd*
riguardo ogni identità *n* in *w* esegue (solo una volta) il primo metodo. In modo tale che quando
il nodo che ha iniziato la propagazione riceve la conferma che tutto il g-nodo *w* è stato informato,
tale nodo sa con certezza che tutti i nodi hanno completato il primo metodo. A questo punto lo stesso
nodo inizierà la propagazione (senza avviso di ricevimento) a tutto *w* dell'ordine di completare la migrazione.

Dopo un certo tempo l'identità *n* riceve l'ordine di completare la migrazione. A questo punto il demone *ntkd*
riguardo l'identità *n* esegue il secondo metodo e realizza così la creazione di *n’*. Così facendo gli *archi-identità*
sono stati correttamente duplicati da *n* a *n’*.

Ora il demone *ntkd* procede con le successive operazioni usando le altre informazioni ricevute.

Il g-nodo *g* di livello *i* è quello che è stato trovato non saturo in *G* e dove è stato riservato un posto per
il g-nodo *w’*, o il singolo nodo *n’*.

Naturalmente *i* è maggiore del livello *k* del g-nodo *w* (*k* vale 0 se l'ingresso è di un singolo nodo *n*).

A queste informazioni il demone *ntkd* aggiunge le altre che già aveva nella sua identità *n* dal livello 0 fino al
livello *k* - 1. Il demone *ntkd* infatti conosce l'indirizzo Netsukuku e il fingerprint a livello 0 dell'identità *n*,
con tutti i valori delle anzianità fino al livello *k* - 1.

Il demone *ntkd* costruisce un indirizzo Netsukuku valido in *g* e il fingerprint a livello 0 per la nuova identità
*n’* in *G*. Per l'identificativo del fingerprint a livello 0, usa lo stesso che aveva prima. Per le posizioni
dell'indirizzo Netsukuku:

*   Per i livelli maggiori o uguali a *i* - 1 usa le posizioni che gli sono state comunicate ora.
*   Per i livelli minori di *i* - 1 usa le posizioni che aveva l'indirizzo dell'identità *n*.

Per le anzianità del fingerprint:

*   Per i livelli maggiori o uguali a *i* - 1 usa le anzianità che gli sono state comunicate ora.
*   Per i livelli maggiori o uguali a *k* e minori di *i* - 1 usa zero (nel senso che è il primo g-nodo).
*   Per i livelli minori di *k* usa le anzianità che erano dei g-nodi a cui apparteneva l'identità *n*.

Poi il demone *ntkd* costruisce le istanze degli archi (IQspnArc) basandosi sugli *archi-identità* duplicati prima.
Riguardo questi archi-identità, il demone deve saper distinguere questi casi:

*   Archi-identità che collegavano *n* ad un nodo *m* in *w*. Quindi il duplicato collega *n’* ad un
    nuovo nodo *m’* in *w’*. Per questi archi-identità il demone *ntkd* costruisce un IQspnArc da passare al nuovo modulo
    QSPN. Inoltre recupera (dal vecchio modulo QSPN) l'indirizzo Netsukuku del vicino *m* che migra
    con noi e da questo costruisce il nuovo indirizzo Netsukuku di *m’*. Infine prepara una
    associazione (in forma di una callback) tra l'arco IQspnArc che era di *n* e quello nuovo costruito
    per *n’*.
*   Archi-identità che collegavano *n* ad un nodo *p* in *G*. Quindi il duplicato collega *n’* a
    *p*. Per questi archi-identità il demone *ntkd* costruisce un IQspnArc da passare al nuovo modulo
    QSPN; mentre non esisteva per essi un IQspnArc di *n*.
*   Archi-identità che collegavano *n* ad un nodo *q* nella vecchia rete ma non in *w*. Quindi il duplicato collega *n’* a
    *q*. Per questi archi-identità non bisogna costruire un IQspnArc da passare al nuovo modulo
    QSPN; mentre esisteva per essi un IQspnArc di *n*.
*   Archi-identità che collegavano *n* ad un nodo *z* in una diversa rete. Quindi il duplicato collega *n’* a
    *z*. Per questi archi-identità *n* non aveva un IQspnArc e nemmeno *n’* lo avrà.

Infine il demone *ntkd* prepara una istanza di IQspnStubFactory.

Il demone *ntkd* costruisce una istanza di QspnManager fornendo:

*   `QspnManager.enter_net` - Tipo di costruttore: ingresso nella rete.
*   `IQspnNaddr my_naddr` - Il proprio indirizzo Netsukuku.
*   `List<IQspnArc> internal_arc_set` - Gli archi tra il nodo *n’* e i suoi vicini in *w’*.
*   `List<IQspnNaddr> internal_arc_peer_naddr_set` - Per gli archi di cui sopra, gli indirizzi Netsukuku
    del vicino in *w’*. Questa lista deve avere la stessa lunghezza della precedente.
*   `List<IQspnArc> external_arc_set` - Gli archi tra il nodo *n’* e i suoi vicini in *G* ma non in *w’*.
*   `PreviousArcToNewArcDelegate old_arc_to_new_arc` - Una callback con firma `IQspnArc? (*) (IQspnArc old)`
    che associa i vecchi archi di *n* in *w* ai nuovi archi di *n’* in *w’*.  
    Questa callback sarà usata solo nel costruttore di QspnManager, poi potrà essere dismessa. Serve
    a copiare correttamente i percorsi noti verso destinazioni in *w’*.
*   `IQspnFingerprint my_fingerprint` - Il suo fingerprint come nodo.
*   `IQspnStubFactory stub_factory` - La stub factory per le comunicazioni con i vicini.
*   `int hooking_gnode_level` - Il livello *k* del g-nodo *w* che sta facendo il suo ingresso in *G*, 0 se era il singolo nodo *n*.
*   `int into_gnode_level` - Il livello *i* del g-nodo *g* in cui *w* sta facendo ingresso.
*   `QspnManager previous_identity` - Il manager precedente. La nuova istanza di QspnManager copia da esso
    nella mappa i percorsi noti verso i g-nodi di livello inferiore a *k* (perché sono in *w*).  
    Grazie a questi percorsi la nuova istanza calcola i fingerprint dal livello 1 al livello *k* (se *k* è maggiore di 0).  
    Inoltre la nuova istanza di QspnManager copia da esso il tipo dell'identità: *principale* o *di connettività* ai livelli
    da *i<sub>0</sub>* a *j<sub>0</sub>*.

Una istanza di QspnManager costruita in questo modo entra in una fase di bootstrap ai livelli da *k* a *i* - 1.
In seguito alcuni nodi di *w’* riceveranno degli ETP completi dai loro diretti vicini esterni a *w’* ma interni
a *g*. Poi propagheranno le nuove conoscenze (cioè percorsi verso g-nodi esterni a *w’*) trasmettendo degli ETP
ai nodi del grafo interni a *w’*. Alla fine ogni nodo del grafo (cioè ogni *identità*) riceve queste nuove
conoscenze e può uscire dalla fase di bootstrap. Quando una istanza di QspnManager esce dalla fase di bootstrap
lo notifica con il segnale *bootstrap_complete*.

Anche ad una istanza di QspnManager costruita in questo modo, il demone *ntkd* segnala le eventuali variazioni sugli archi con i metodi:

*   `arc_add (IQspnArc arc)`.
*   `arc_is_changed (IQspnArc changed_arc)`.
*   `arc_remove (IQspnArc removed_arc)`.

Una volta costruita la nuova istanza, il demone *ntkd* potrà dismettere (rimuovere ogni riferimento che deteneva) la
vecchia istanza di QspnManager che gestisce l'identità *n*, la quale adesso sicuramente non è la *principale*.

Prima di farlo, il demone *ntkd* chiama su di essa il metodo *destroy* per segnalare ai suoi diretti vicini esterni a
*w* che sta uscendo dalla rete. Tale metodo esegue una chiamata broadcast a metodo remoto.

### <a name="Migrazione"></a>Migrazione

Supponiamo che il g-nodo *w* (a cui appartiene il nodo del grafo *n* che vive nel nostro sistema) migra da un g-nodo
*g* di livello *i* ad un diverso g-nodo *h* (sempre di livello *i*) il quale appartiene a g-nodi distinti da quelli
di *g* fino al livello *j*. In questa trattazione è incluso il caso in cui *w* equivale a *n*.

In qualche modo si è eletto un nodo *n<sub>0</sub>* in *w* — che deve essere l'identità principale del suo sistema — che
ha un arco verso *h* a coordinare la migrazione. L'identità *n<sub>0</sub>* chiede ad un suo diretto vicino in *h*,
chiamiamolo *h<sub>0</sub>*, i dati che servono all'ingresso. Questa richiesta da *n<sub>0</sub>* a *h<sub>0</sub>*
viene fatta con i metodi remoti di un modulo *di identità*. Per rispondere, *h<sub>0</sub>* inizia un dialogo con il
Coordinator di *h*. Inoltre l'identità *n<sub>0</sub>* inizia un dialogo con il Coordinator di *g* per ottenere la
prenotazione di un identificativo *virtuale* al livello *i* - 1 in *g* e la sua anzianità. Poi l'identità *n<sub>0</sub>*
genera un identificativo di migrazione *migration_id* e propaga questi dati a tutto il g-nodo *w*, anche qui attraverso
chiamate a metodo remoto fatte con un modulo *di identità*. Infine *n<sub>0</sub>* riceve conferma che tutti i nodi del
grafo appartenenti al g-nodo *w* sono stati portati a conoscenza dei dati e del *migration_id*. A questo punto
*n<sub>0</sub>* propaga analogamente a tutto il g-nodo *w* l'ordine di completare la migrazione.

Dobbiamo esaminare cosa avviene in ogni sistema in cui vivono dei nodi del grafo che appartengono a *w*. In particolare
in ogni sistema possono vivere una o più istanze *n* di QspnManager (cioè nodi del grafo) che appartengono a *w*. Ogni
identità *n* in *w* poteva essere l'identità *principale* o una *di connettività* in g-nodi interni a *w*.<sup>1</sup>
Non sicuramente un'identità *di connettività* in g-nodi di livello superiore a *w*, in quanto il g-nodo *w* che migra
(considerato nel suo insieme come singolo vertice nel grafo *[G]<sub>i-1</sub>*) è sicuramente un g-nodo *reale*. Nel
senso che il suo Netsukuku address, che è composto da identificativi dal livello *i*-1 in su, non ha alcun componente *virtuale*.

Per ogni istanza *n* di QspnManager che apparteneva a *w* il demone *ntkd* crea una nuova istanza di QspnManager *n’*
basata su *n* che è una identità in *h* di tipo analogo (*principale* o *di connettività*) a ciò che era *n* in *w*.

In questo modo si produce di fatto un g-nodo *w’* all'interno del g-nodo *h* che è isomorfo al g-nodo *w* all'interno del g-nodo *g*.

Inoltre viene modificato l'indirizzo detenuto da *n* in uno *di connettività* ai livelli da *i* a *j*.

* * *

**Nota 1:** Quando diciamo che un nodo del grafo (o una identità) *n*, con *n* ∈ *w*, è *di connettività* in g-nodi
interni a *w* intendiamo dire che l'indirizzo che *n* detiene è un indirizzo *di connettività* ai livelli da
*i<sub>0</sub>* a *j<sub>0</sub>*, dove *i<sub>0</sub>* è strettamente minore del livello di *w*. Non si fa alcuna
assunzione invece sul valore *j<sub>0</sub>*.

#### <a name="Creazione_in_h"></a>Creazione di una identità in h

I dati che ogni identità *n* riceve in relazione a questa migrazione da *g* in *h* sono:

*   L'identificativo di migrazione *migration_id*. Si tratta di un identificativo univoco (un numero random) reso
    noto a tutti i partecipanti alla migrazione.
*   L'indirizzo Netsukuku di *h* e la sua anzianità e quella dei suoi g-nodi superiori ai livelli da *i* a *j*.
*   Un identificativo *reale* libero al livello *i* - 1 in *h*, oppure uno *virtuale* e la prenotazione di uno reale
    che verrà presto liberato.
*   L'anzianità del nuovo g-nodo al livello *i* - 1 in *h*.
*   Un identificativo *virtuale* al livello *i* - 1 in *g*, il primo libero.
*   L'anzianità del nuovo identificativo in *g*.

Abbiamo già detto che tali informazioni giungono ad una precisa identità *n* che vive nel sistema. Il demone *ntkd* per
ogni identità *n* crea una nuova identità *n’*.

Questa operazione di creazione di una nuova identità viene realizzata dal demone *ntkd* avvalendosi del modulo
*Identities*, come esposto sopra nella trattazione dell'ingresso in diversa rete.

Dopo il demone *ntkd* procede con le successive operazioni usando le altre informazioni ricevute.

Naturalmente *i* è maggiore del livello *k* del g-nodo *w* (*k* vale 0 se la migrazione è di un singolo nodo *n*).

A queste informazioni il demone *ntkd* aggiunge le altre che già aveva nella sua identità *n* dal livello 0 fino al
livello *k* - 1. Il demone *ntkd* infatti conosce l'indirizzo Netsukuku e il fingerprint a livello 0 dell'identità *n*,
con tutti i valori delle anzianità fino al livello *k* - 1.

Il demone *ntkd* costruisce un indirizzo Netsukuku valido in *h* e il fingerprint a livello 0 per la nuova identità
*n’* in *h*. Per l'identificativo del fingerprint a livello 0, usa lo stesso che aveva prima. Per le posizioni
dell'indirizzo Netsukuku:

*   Per i livelli maggiori di *j* usa le posizioni che aveva l'indirizzo dell'identità *n*.
*   Per i livelli maggiori o uguali a *i* - 1 e minori o uguali a *j* usa le posizioni che gli sono state comunicate ora.
*   Per i livelli minori di *i* - 1 usa le posizioni che aveva l'indirizzo dell'identità *n*.

Per le anzianità del fingerprint:

*   Per i livelli maggiori di *j* usa le anzianità che aveva l'indirizzo dell'identità *n*.
*   Per i livelli maggiori o uguali a *i* - 1 e minori o uguali a *j* usa le anzianità che gli sono state comunicate ora.
*   Per i livelli maggiori o uguali a *k* e minori di *i* - 1 usa zero (nel senso che è il primo g-nodo).
*   Per i livelli minori di *k* usa le anzianità che erano dei g-nodi a cui apparteneva l'identità *n*.

Notiamo che il fatto di usare per la nuova identità di *n* lo stesso identificativo di fingerprint a livello 0 che si usava
nella vecchia identità è necessario e non comporta alcun problema.

*   È necessario perché risultino coerenti le informazioni acquisite con la vecchia identità riguardo i percorsi verso g-nodi
    interni a *w*. Questi verranno copiati nella nuova identità.
*   Non comporta alcun problema riguardo l'assegnazione dell'identificativo di fingerprint ai g-nodi di livello maggiore
    di *k*. Infatti il g-nodo *w* in *g* avrà lo stesso identificativo di fingerprint del g-nodo isomorfo *w’* in *h*. Ma il
    g-nodo *w* in *g* avrà ora una componente *virtuale* al livello *i*-1 e un valore di anzianità maggiore (significa che
    è meno anziano) degli altri g-nodi in *g*. Quindi è impossibile che il g-nodo *g* e il g-nodo *h* ottengano per questo
    un identificativo di fingerprint identico.

Poi il demone *ntkd* costruisce le istanze degli archi (IQspnArc) basandosi sugli *archi-identità* duplicati prima.
Riguardo questi archi-identità, il demone deve saper distinguere questi casi:

*   Archi-identità che collegavano *n* ad un nodo *m* in *w*. Quindi il duplicato collega *n’* ad un
    nuovo nodo *m’* in *w’*. Per questi archi-identità il demone *ntkd* costruisce un IQspnArc da passare al nuovo modulo
    QSPN. Inoltre recupera (dal vecchio modulo QSPN) l'indirizzo Netsukuku del vicino *m* che migra
    con noi e da questo costruisce il nuovo indirizzo Netsukuku di *m’*. Infine prepara una
    associazione (in forma di una callback) tra l'arco IQspnArc che era di *n* e quello nuovo costruito
    per *n’*.
*   Archi-identità che collegavano *n* ad un nodo *q* nella rete ma non in *w*. Quindi il duplicato collega *n’* a
    *q*. Per questi archi-identità il demone *ntkd* costruisce un IQspnArc da passare al nuovo modulo
    QSPN.
*   Archi-identità che collegavano *n* ad un nodo *z* in una diversa rete. Quindi il duplicato collega *n’* a
    *z*. Per questi archi-identità *n* non aveva un IQspnArc e nemmeno *n’* lo avrà.

Infine il demone *ntkd* prepara una istanza di IQspnStubFactory.

Costruisce una istanza di QspnManager fornendo:

*   `QspnManager.migration` - Tipo di costruttore: per migrazione.
*   `IQspnNaddr my_naddr` - Il proprio indirizzo Netsukuku.
*   `List<IQspnArc> internal_arc_set` - Gli archi tra il nodo *n’* e i suoi vicini in *w’*.
*   `List<IQspnNaddr> internal_arc_peer_naddr_set` - Per gli archi di cui sopra, gli indirizzi Netsukuku
    del vicino in *w’*. Questa lista deve avere la stessa lunghezza della precedente.
*   `List<IQspnArc> external_arc_set` - Gli archi tra il nodo *n’* e i suoi vicini nella rete che non sono in *w’*.
*   `PreviousArcToNewArcDelegate old_arc_to_new_arc` - Una callback con firma `IQspnArc? (*) (IQspnArc old)`
    che associa i vecchi archi di *n* in *w* ai nuovi archi di *n’* in *w’*.
*   `IQspnFingerprint my_fingerprint` - Il suo fingerprint come nodo.
*   `IQspnStubFactory stub_factory` - La stub factory per le comunicazioni con i vicini.
*   `int hooking_gnode_level` - Il livello *k* del g-nodo *w* che sta facendo la migrazione, 0 se era il singolo nodo *n*.
*   `int into_gnode_level` - Il livello *i* del g-nodo *g* in cui *w* sta facendo ingresso.
*   `QspnManager previous_identity` - Il manager precedente. La nuova istanza di QspnManager copia da esso
    nella mappa i percorsi noti verso i g-nodi di livello inferiore a *k* (perché sono in *w*).  
    Grazie a questi percorsi la nuova istanza calcola i fingerprint dal livello 1 al livello *k* (se *k* è maggiore di 0).  
    Inoltre la nuova istanza di QspnManager copia da esso il tipo dell'identità: *principale* o *di connettività* ai livelli
    da *i<sub>0</sub>* a *j<sub>0</sub>*.

Una istanza di QspnManager costruita in questo modo entra in una fase di bootstrap ai livelli da *k* a *i* - 1. In seguito
i border-nodi di *w’* riceveranno degli ETP completi dai loro diretti vicini esterni a *w’* ma interni a *g*. Poi
propagheranno le nuove conoscenze (cioè percorsi verso g-nodi esterni a *w’*) trasmettendo degli ETP ai nodi del grafo
interni a *w’*. Alla fine ogni nodo del grafo (cioè ogni *identità*) riceve queste nuove conoscenze e può uscire dalla
fase di bootstrap. Quando una istanza di QspnManager esce dalla fase di bootstrap lo notifica con il segnale *bootstrap_complete*.

Dopo che è uscita dalla fase di bootstrap, una istanza di QspnManager trasmette i suoi primi ETP ai diretti vicini. Dopo
averli trasmessi, aspetta un certo tempo per accertarsi che i vicini abbiano avuto modo di processarli e poi emette il
segnale *presence_notified*. Con questo segnala al suo utilizzatore che i suoi nodi diretti vicini sono a conoscenza dei
percorsi che possono sfruttare attraverso tale nodo del grafo.

Questo fatto è particolarmente importante per una istanza di QspnManager costruita in questo modo. Infatti quando il
demone *ntkd* riceve questo segnale dall'istanza *n’* sa di poter chiedere alla istanza *n* (come vedremo sotto) di
rimuovere gli archi esterni al livello *j*.

Anche ad una istanza di QspnManager costruita in questo modo, il demone *ntkd* segnala le eventuali variazioni sugli archi con i metodi:

*   `arc_add (IQspnArc arc)`.
*   `arc_is_changed (IQspnArc changed_arc)`.
*   `arc_remove (IQspnArc removed_arc)`.

Abbiamo detto che tra i dati di cui è in possesso l'identità *n’* riguardanti questa migrazione c'è un identificativo
*reale* libero al livello *i* - 1 in *h*, oppure uno *virtuale* e la prenotazione di uno reale che verrà presto liberato.
Nel momento in cui l'identità *n’* può trasformare l'identificativo *virtuale* al livello *i* - 1 in uno *reale*, il
demone *ntkd* chiama su questa istanza di QspnManager il metodo *make_real*.

Per la chiamata di tale metodo, solo se *n’* ha già completato la sua fase di bootstrap, verrà avviata immediatamente la
trasmissione di un ETP a tutti i vicini di *n’* esterni a *w’*. In esso la lista dei percorsi è vuota, perché va segnalato
soltanto il nuovo percorso verso il nuovo identificativo *reale* di *w’* al livello *i* - 1.

#### <a name="Modifica_precedente"></a>Modifica della precedente identità in g in una identità di connettività

Con i dati suddetti relativi alla migrazione il demone *ntkd* sull'istanza di QspnManager *n* chiama il metodo
*make_connectivity*. Con questa chiamata segnala al modulo che questa diventa una identità *di connettività*. Gli argomenti passati sono:

*   Il livello *i* della migrazione.
*   Il livello *j* della migrazione.
*   Il nuovo indirizzo Netsukuku. In realtà l'unica cosa che cambia in esso è il nuovo identificativo *virtuale* al
    livello *i* - 1, cioè dentro *g*. Ma questo dato nella classe dell'indirizzo non è accessibile in scrittura al modulo
    QSPN, quindi viene passata una istanza di IQspnMyNaddr che il modulo considera nuova.
*   Il nuovo fingerprint a livello 0. In realtà l'unica cosa che cambia in esso è la nuova anzianità al livello *i* - 1,
    cioè dentro *g*. Ma questo dato nella classe del fingerprint non è accessibile al modulo QSPN, quindi viene passata
    una istanza di IQspnFingerprint che il modulo considera nuova.

Verrà avviata tra pochi istanti la trasmissione di un ETP a tutti i vicini di *n* esterni a *w*. In esso va segnalata
soltanto la rimozione del percorso verso il vecchio identificativo *reale* di *w* al livello *i* - 1.

Il demone *ntkd* sull'istanza di QspnManager *n* deve registrare un ascoltatore sul segnale *remove_identity*. Infatti
solo uno dei membri di *w* verifica in prima persona se il g-nodo *w* può essere dismesso. Gli altri si propagano l'ordine
di dismissione di *w* e ogni singola identità quando lo esegue segnala questo evento al demone *ntkd*.

In seguito il demone *ntkd* sull'istanza di QspnManager *n* può chiamare questi metodi:

*   *remove_outer_arcs*  
    Chiede al modulo di rimuovere gli archi che collegano a g-nodi di livello maggiore di *j*.
*   *check_connectivity*  
    Il demone *ntkd* chiama questo metodo periodicamente sull'istanza di QspnManager *n*. Con esso chiede al modulo
    di controllare se la connettività interna dei g-nodi di livello da *i* a *j* è garantita anche senza il g-nodo *w*.  
    Se il livello del g-nodo *w* è maggiore di 0, cioè se effettivamente era un g-nodo a migrare, vorremmo che fosse
    solo un singolo nodo in *w* a fare periodicamente queste operazioni di verifica. **TODO** investigare su come fare.  
    Vorremmo evitare di fare questa operazione più volte per un g-nodo perché, lo ricordiamo, prima di chiamare
    *check_connectivity* sull'istanza *n* il demone *ntkd* deve ottenere un "lock" sui g-nodi interessati.  
    Se la verifica ha esito positivo:
    *   Se la migrazione era di un g-nodo *w* :
        *   Il demone *ntkd* sull'istanza di QspnManager *n* chiama il metodo *prepare_destroy* : con esso chiede al
            modulo di propagare questa informazione agli altri nodi in *w*. Poi aspetta 10 secondi.
    *   Il demone *ntkd* sull'istanza di QspnManager *n* chiama il metodo *destroy* : con esso chiede al modulo di
        segnalare ai suoi diretti vicini (esterni a *w*) che questa identità sta per essere distrutta.
    *   Il demone *ntkd* dismette (rimuove ogni riferimento che deteneva) l'istanza di QspnManager *n*.
*   *prepare_destroy*  
    Chiede al modulo di istruire i suoi diretti vicini (interni a *w*) di attendere 10 secondi e poi scartare la loro
    identità. La comunicazione ai vicini avviene tramite il metodo remoto *got_prepare_destroy* della classe QspnManager.  
    Chi riceve il messaggio *got_prepare_destroy* si accerta di essere una identità di connettività. Altrimenti ignora il messaggio.  
    **TODO**: Chi riceve il messaggio *got_prepare_destroy* dovrebbe per sicurezza accertarsi che la decisione di rimuovere
    il g-nodo è stata presa da chi di dovere.  
    Chi riceve il messaggio *got_prepare_destroy* propaga il messaggio. Poi aspetta 10 secondi. Poi emette il segnale
    *remove_identity* che istruisce l'utilizzatore del modulo che deve dismettere (rimuovere ogni riferimento che deteneva)
    questa istanza di QspnManager e gli altri moduli relativi alla medesima identità *n*.
*   *destroy*  
    Chiede al modulo di segnalare ai suoi diretti vicini (esterni a *w*) che l'identità chiamante sta per essere
    distrutta. La comunicazione ai vicini avviene tramite il metodo remoto *got_destroy* della classe QspnManager.

## <a name="Ulteriori_deliverable"></a>Ulteriori deliverable

Esaminiamo altri deliverable del modulo QSPN, che sono gli stessi per ogni tipo di identità e di modalità di costruzione del QspnManager.

Il modulo, attraverso il segnale *arc_removed* di QspnManager, notifica che un arco *arc* è stato rimosso di
iniziativa del QspnManager. Questo può avvenire per diverse situazioni: riconoscere il motivo della rimozione
è importante per l'utilizzatore del modulo, perché da questo dipende come l'utilizzatore deve reagire al segnale.

*   Una comunicazione attraverso *arc* fatta con protocollo reliable fallisce. Di norma ci si avvede di questo con
    l'eccezione StubError.  
    In questo caso l'utilizzatore del modulo dovrebbe presumere che il link ha dei problemi. L'utilizzatore dovrà rimuovere
    l'arco-nodo su cui questo arco-identità si poggia; con esso anche tutti gli altri archi-identità che vi si poggiano.
*   Si sono verificati problemi di comprensione di una comunicazione attraverso *arc*. Ad esempio un DeserializeError
    (comprende i casi in cui si riceve un oggetto sconosciuto o che non implementa l'interfaccia voluta), oppure una
    istanza di IQspnEtpMessage che non è un EtpMessage, oppure un EtpMessage con dati errati, oppure una istanza di
    IQspnAddress che non è una istanza di IQspnNaddr.  
    In questo caso il link non dovrebbe avere problemi. Si può ipotizzare che la causa sia un errore nel programma nel
    nodo nostro o nel nodo vicino; oppure un comportamento malevolo nel nodo vicino; oppure una diversa versione del
    programma nel nodo vicino. In ogni caso si può ipotizzare che le altre possibili istanze di QspnManager (quindi le
    altre identità collegate con altri archi-identità) possono continuare a funzionare a dovere.  
    Se questo arco-identità è l'arco *principale* dell'arco-nodo su cui si poggia, allora l'utilizzatore dovrà rimuovere
    l'arco-nodo; con esso anche tutti gli altri archi-identità che vi si poggiano. Altrimenti viene rimosso solo questo
    arco-identità e l'utilizzatore non intraprende altre azioni.
*   Il vicino (una particolare identità del nodo vicino) collegato su *arc* non ha (o non ha più) *arc* fra i suoi archi.
    Di norma ci si avvede di questo con l'eccezione QspnNotAcceptedError.  
    Se questo arco-identità è l'arco *principale* dell'arco-nodo su cui si poggia, allora l'utilizzatore dovrà rimuovere
    l'arco-nodo; con esso anche tutti gli altri archi-identità che vi si poggiano. Altrimenti viene rimosso solo questo
    arco-identità e l'utilizzatore non intraprende altre azioni.
*   Il metodo *remove_outer_arcs* chiamato dall'utilizzatore del modulo ha prodotto la rimozione di *arc*.  
    In questo caso, per definizione, questo arco-identità non è l'arco *principale* dell'arco-nodo su cui si poggia. Quindi
    viene rimosso solo questo arco-identità e l'utilizzatore non intraprende altre azioni.
*   Il metodo remoto *got_destroy* chiamato da un vicino ha prodotto la rimozione di *arc*.  
    Se questo arco-identità è l'arco *principale* dell'arco-nodo su cui si poggia, allora l'utilizzatore dovrà rimuovere
    l'arco-nodo; con esso anche tutti gli altri archi-identità che vi si poggiano. Altrimenti viene rimosso solo questo
    arco-identità e l'utilizzatore non intraprende altre azioni.

Riassumendo, nel segnale *arc_removed* di QspnManager viene incluso un booleano *bad_link* che dice se la rimozione è da
imputare ad un errore nella trasmissione di un pacchetto tentata con protocollo reliable. In questo caso l'utilizzatore
dovrà rimuovere l'arco-nodo. Negli altri casi l'utilizzatore presume che si voglia rimuovere solo quel preciso arco-identità
(a meno che non si tratti dell'arco-identità principale).

* * *

Il modulo notifica l'inserimento di un nuovo g-nodo nella sua mappa attraverso il segnale *destination_added* di
QspnManager e la rimozione di un g-nodo attraverso il segnale *destination_removed* di QspnManager.

* * *

Il modulo notifica l'inserimento di un nuovo percorso verso una destinazione attraverso il segnale *path_added*, la
rimozione di un percorso attraverso il segnale *path_removed* e il cambio di qualche parametro in un percorso attraverso
il segnale *path_changed* di QspnManager.

* * *

Il modulo notifica il cambio nel fingerprint di uno dei suoi g-nodi attraverso il segnale *changed_fp* di QspnManager.

* * *

Il modulo notifica un cambiamento significativo nel numero approssimato di nodi all'interno di uno dei suoi g-nodi
attraverso il segnale *changed_nodes_inside* di QspnManager.

* * *

Il modulo notifica il rilevamento di uno split in un g-nodo di sua conoscenza attraverso il segnale *gnode_splitted* di QspnManager.

* * *

Il modulo permette di vedere se ha completato il suo bootstrap con il metodo *is_bootstrap_complete* di QspnManager.

* * *

Il modulo permette di ottenere la lista dei g-nodi nella sua mappa con il metodo *get_known_destinations* di QspnManager.

* * *

Il modulo permette di ottenere la lista di percorsi noti verso un certo g-nodo con il metodo *get_paths_to* di QspnManager.

* * *

Il modulo permette di leggere il fingerprint di uno dei suoi g-nodi con il metodo *get_fingerprint* di QspnManager.

* * *

Il modulo permette di leggere il numero approssimato di nodi all'interno di uno dei suoi g-nodi con il metodo
*get_nodes_inside* di QspnManager.

## <a name="Funzioni_di_appoggio"></a>Funzioni di appoggio

Alcune funzioni, definite e usate internamente al modulo, sono aggiunte alle classi così come sono state descritte
nell'analisi funzionale. Le illustriamo ora perché nel seguito del documento sono riportati alcuni algoritmi che ne fanno uso.

Aggiunte alla classe NodePath:

*   La proprietà dinamica *cost* : viene calcolata dinamicamente come somma dei costi di *arc* e *path*.
*   La proprietà *exposed* : indica se per questo percorso un segnale di *path_added* era stato emesso e in seguito
    nessun segnale di *path_removed*. Questo flag serve a gestire correttamente i segnali che il modulo deve emettere
    quando diversi percorsi verso un g-nodo di livello maggiore di 0 indicano fingerprint diversi (potenziale split).  
    Quando viene creata una istanza di NodePath la proprietà *exposed* è sempre inizializzata a False. Solo quando l'istanza
    viene messa nella mappa del nodo, allora può venire impostata a True.
*   I metodi *hops_arcs_equal_etppath(EtpPath p)* e *hops_arcs_equal(NodePath np)* :  
    Si ricorda che due percorsi sono distinti se usano un diverso gateway o lo raggiungono attraverso una diversa interfaccia
    di rete del nodo corrente; oppure se la sequenza di passi fino alla destinazione (compresa) non è identica sia come g-nodi
    sia come archi di uscita. Considerato che nei percorsi memorizzati nella mappa la sequenza di archi comprende anche
    l'identificativo dell'arco del nodo corrente che lo collega al gateway; considerato che un arco univocamente identifica
    anche il gateway che vi è collegato; si deduce che il test di uguaglianza di un percorso *NodePath np* nella mappa del
    nodo con un altro percorso si basa esclusivamente sulle liste *np.path.hops* e *np.path.arcs*.  
    Si consideri inoltre che i confronti con altri percorsi ricevuti in un ETP (istanze di *EtpPath* ) vengono fatti dopo che
    la Grouping Rule è stata eseguita, quindi anche essi comprendono come primo elemento delle liste *hops* e *arcs* gli
    identificativi riferiti al passo dal nodo corrente al suo gateway.  
    Il metodo *hops_arcs_equal_etppath(EtpPath p)* : confronta questa istanza con il percorso *p* ricevuto in un ETP tramite
    l'arco *arc* di questa istanza.  
    Il metodo *hops_arcs_equal(NodePath np)* : confronta questa istanza con l'istanza *np*.

## <a name="Produzione_trasmissione_etp"></a>Produzione e trasmissione di ETP

La costruzione e il passaggio di ETP quando necessario ai propri vicini è realizzata dal modulo in autonomia, senza
necessitare di interventi dal suo utilizzatore.

Per sapere a chi deve inviare gli ETP e da chi deve riceverli il modulo si basa sugli archi che gli sono stati passati.

Quando riceve un ETP se non riesce ad associarlo ad uno degli archi che conosce (si intende anche dopo aver atteso il
*tempo di rilevamento*) allora lo ignora. Quando riceve una richiesta di ETP se non riesce ad associarla ad uno degli
archi che conosce allora risponde con l'eccezione QspnNotAcceptedError.

Il modulo assume che i nodi collegati ai suoi archi appartengano alla sua stessa rete.

L'elaborazione degli ETP ricevuti dai miei vicini che appartengono alla mia stessa rete da come risultato l'aggiornamento
della mia mappa; allo stesso tempo mi permette anche di capire quali informazioni saranno di interesse anche per gli altri
miei vicini, quindi mi consente di costruire un messaggio che sia esaustivo e allo stesso tempo più piccolo possibile. Il
dettaglio di come tale elaborazione avviene è descritto nel documento [esplorazione](EsplorazioneRete.md).

### <a name="Metodi_remoti"></a>Metodi remoti

L'implementazione della trasmissione di ETP ai vicini avviene con l'uso di due metodi remoti: *get_full_etp* e *send_etp*.

Il metodo *get_full_etp* viene chiamato dal nodo *n* sul vicino *v* quando vuole ottenere da *v* un ETP *completo*. Il
documento di esplorazione descrive cosa si intende per ETP completo. Quando un nodo chiede espressamente un ETP ad un suo
vicino (o a tutti i vicini uno alla volta) si tratta sempre di un ETP completo.

Il metodo *send_etp* viene usato dal nodo *n* quando vuole spontaneamente inviare un ETP ad un suo vicino o a più vicini
in broadcast. In questo caso può trattarsi di un ETP completo oppure no. Per questo il metodo *send_etp* prevede un
parametro booleano che dice se l'ETP è completo.

### <a name="Produzione_primo_etp"></a>Produzione del primo ETP

Prendiamo in esame un nodo che ha appena costituito una nuova rete. Il modulo QSPN cioè non riceve nessun arco. La sua
mappa è vuota. Vediamo cosa deve fare per costruire un nuovo ETP completo, presumibilmente perché un nuovo nodo ha appena
fatto ingresso nella rete e gli chiede un ETP.

Innanzi tutto la sua mappa è vuota, quindi la lista dei percorsi *P* è vuota.

Ci sono due informazioni che vanno inserite nell'ETP e che sono rappresentate con classi che il modulo non conosce: il
fingerprint dei suoi g-nodi ai vari livelli e il suo indirizzo di nodo.

Al modulo viene passato l'indirizzo del nodo come istanza dell'interfaccia IQspnMyNaddr, che a sua volta richiede
l'interfaccia IQspnNaddr. Nell'ETP va inclusa una istanza di IQspnNaddr, quindi questa istanza va bene. Il modulo assume
che l'istanza sia anche un oggetto serializzabile.

Al modulo viene passato il fingerprint del nodo a livello 0 come istanza dell'interfaccia IQspnFingerprint. Anche questa
si assume che sia un oggetto serializzabile. Inoltre, tramite il metodo i_qspn_construct chiamato sull'istanza del fingerprint
a livello 0 e passando un set vuoto (ci riferiamo sempre ad un nodo con la mappa ancora vuota), gli viene restituita un'altra
istanza di IQspnFingerprint che è il fingerprint del suo g-nodo a livello 1. Anche questa si assume che sia un oggetto
serializzabile. E così via.

La lista del numero approssimativo di nodi all'interno dei propri g-nodi ai vari livelli, per un nodo con la mappa ancora
vuota è banalmente una lista con *l* volte il valore 1.

Nell'ETP vi è anche una lista di HCoord che rappresenta il percorso di questo ETP. Un nodo che produce un ETP nuovo
vi mette una `List<HCoord>` vuota.

In conclusione il modulo è in grado di costruire una istanza della classe EtpMessage con i dati necessari. Si tratta
di un oggetto serializzabile, che quindi può essere usato nelle chiamate (o risposte) a metodi remoti.

### <a name="Processazione_set_etp"></a>Processazione di un set di ETP

Gli algoritmi illustrati nei successivi paragrafi, volti a processare un set di ETP, vengono eseguiti dal nodo *n* in
queste possibili circostanze:

*   Il g-nodo *w* vuole fare ingresso nel g-nodo *g*. Il nodo *n* appartiene a *w* e ha un arco verso un nodo *m* che
    non appartiene a *w* ma appartiene a *g*.  
    Il nodo *n* in fase di bootstrap ha chiesto un ETP al suo vicino *m*.
*   Il g-nodo *w* vuole fare ingresso nel g-nodo *g*. Il nodo *n* appartiene a *w* e ha solo archi verso nodi che appartengono a *w*.  
    Il nodo *n* in fase di bootstrap ha ricevuto un ETP da un vicino che è uscito dal bootstrap.
*   Il nodo *n*, uscito dal bootstrap, ha rilevato un nuovo arco *a* verso un vicino *v* e quindi ha chiesto un ETP a *v*.
*   Il nodo *n*, uscito dal bootstrap, ha rilevato il cambio di costo in un arco *a_changed* e quindi  ha chiesto un
    ETP a tutti i suoi vicini. In questo caso tra gli argomenti di questo algoritmo ho anche l'arco *a_changed* che ha
    avuto una variazione.
*   Il nodo *n*, uscito dal bootstrap, ha rilevato la rimozione di un arco *a_removed* e quindi  ha chiesto un ETP a tutti
    i suoi vicini. In questo caso prima di iniziare questo algoritmo, il nodo *n* ha rimosso dalla sua mappa tutti i
    percorsi che sfruttavano l'arco *a_removed*.
*   Il nodo *n*, uscito dal bootstrap, ha ricevuto un ETP inviato in broadcast da un suo vicino.

### <a name="Rielaborazione_percorsi_etp_ricevuto"></a>Rielaborazione dei percorsi in ogni ETP ricevuto

Sia *m* un ETP che il nodo *n* riceve dal nodo *v* attraverso l'arco *a*.

Quando il modulo riceve una istanza di EtpMessage che gli viene trasmessa in una chiamata a metodo remoto, questa contiene
le istanze degli oggetti che l'utilizzatore del modulo (nell'altro nodo) ha costruito e che il modulo continua a conoscere
solo in quanto implementazioni delle interfacce a lui note. Inoltre riconoscendo l'arco *a* da cui ha ricevuto il messaggio
può ottenere l'istanza di IQspnCost che rappresenta il suo costo. Anche questa si assume che sia un oggetto serializzabile.
Infine, sempre avendo individuato l'arco *a*, ha individuato l'identificativo *id(a)*, che è un intero.

Il nodo *n* calcola il livello *i* del minimo comune g-nodo tra *n* e *v*.

Il nodo *n* esamina la lista di hop percorsi da *m*. Per prima cosa esegue la [Grouping Rule](EsplorazioneRete.md#GroupingRule)
su tale lista per renderla coerente con i g-nodi a cui *n* appartiene. Poi esegue la
[Acyclic Rule](EsplorazioneRete.md#AcyclicRule) sulla lista, e se vede che *m* era già passato per il mio nodo (o g-nodo) lo
ignora del tutto.

Infine *n* esamina il set di percorsi *P* contenuto in *m*, ma deve renderli coerenti con i suoi g-nodi. Ecco come realizza questo:

*   **1)** Il nodo *n* elimina dal set *P* tutti i percorsi che sono stati dichiarati da ignorare all'esterno di *v<sub>i-1</sub>*.
*   **2)** Per ogni percorso *p* rimasto in *P* il nodo *n* esegue la Grouping Rule.
*   **3)** Per ogni percorso *p* in *P* il nodo *n* esegue la Acyclic Rule e se vede che il percorso ha fatto un ciclo lo rimuove da *P*.
*   **4)** Il nodo *n* aggiunge a *P* il percorso verso *v<sub>i-1</sub>* con costo *null* e fingerprint e numero di nodi
    così come riportati in *m*.
*   **5)** Solo se *m* è un ETP completo:
    *   Il nodo *n* cerca in tutta la sua mappa i percorsi che partono dal gateway *v* attraverso l'arco *a*. Li mette in
        un set *M<sub>a</sub>*.
    *   Per ogni percorso *np* in *M<sub>a</sub>* :
        *   Cerca un percorso *p* in *P* tale che *np.hops_arcs_equal_etppath(p)*.
        *   Se non esiste tale *p* :
            *   Crea una copia *np<sub>0</sub>* di *np* con costo *dead* e la mette nel set *Q*. Per farlo, crea una
                nuova istanza di EtpPath *p<sub>0</sub>* copiando i valori di *np’* e mettendo *cost* a *dead*. Non c'è
                bisogno di valorizzare *ignore_outside*. Poi crea una istanza di NodePath *np<sub>0</sub>* associando al
                percorso *p<sub>0</sub>* l'arco *a*.
*   **6)** Per tutti i percorsi *p* del set *P*, i quali sono istanze di EtpPath, il nodo *n* crea una istanza di NodePath
    *q* associando al percorso *p* l'arco *a*. Mette *q* nel set *Q*.

Spieghiamo il punto 4. La ricezione di un ETP comporta sempre l'acquisizione di qualche informazione sulla rete, quindi
contribuisce a popolare la mappa di *n*. Infatti, anche quando la lista *P* nell'ETP *m* fosse vuota, l'ETP contiene
intrinsecamente un percorso verso il massimo distinto g-nodo di *v* per *n*, che indichiamo con *v<sub>i-1</sub>*. Per
rappresentare questo percorso intrinseco, il modulo nel nodo *n* costruisce una istanza della classe EtpPath con questi dati:

*   La sequenza *hops* è composta da un solo g-nodo che è *v<sub>i-1</sub>*.
*   La sequenza *arcs* è composta dal solo *id(a)*.
*   Il costo del percorso è *null*. Esso rappresenta il costo da *v* ad *v*.
*   Il fingerprint del g-nodo è il fingerprint indicato nell'ETP *m* al livello *i* - 1.
*   Il numero di nodi nel g-nodo è il numero di nodi indicato nell'ETP *m* al livello *i* - 1.

Spieghiamo il punto 5. Siccome *m* è un ETP completo, tutti i percorsi che *n* può usare passando per *v* sono presenti
in *m.P*. Quindi i vecchi percorsi che *n* conosceva e che non sono in *m.P* andranno rimossi dalla mappa di *n*.

Spieghiamo il punto 6. In tutte le istanze di EtpPath ricevute nell'ETP (più quella costruita come detto adesso per
rappresentare il percorso intrinseco verso il g-nodo vicino) il costo del percorso rappresenta il costo da *v* alla
destinazione. Invece il costo da *n* ad *v* è il costo dell'arco *a*. Questo dato è già memorizzato nell'oggetto arco
che comunque il modulo deve mantenere associato al percorso, per questo nella mappa interna del nodo *n* ogni percorso
è in realtà memorizzato come istanza della classe NodePath, la quale include l'istanza di EtpPath.

### <a name="Variazioni_mappa"></a>Variazioni nella mappa di n

Quando abbiamo descritto, nel documento [esplorazione](EsplorazioneRete.md), la gestione degli eventi che portano a
variazioni nel grafo della rete, abbiamo più volte usato diciture quali "n aggiorna la sua mappa" e "mette in P i
percorsi che nella sua mappa hanno subito variazioni".

Qui riportiamo l'algoritmo che realizza queste macro-operazioni. Inoltre l'algoritmo riportato evita di occupare memoria
con percorsi che non sono disgiunti da altri già noti, come richiesto nell'analisi funzionale. Infine si occupa di
emettere i segnali di variazioni nella mappa del nodo (nuova destinazione, nuovo percorso, ...) e i segnali di g-nodi splittati.

Il rilevamento dello split di un g-nodo *d* si ha quando per raggiungere *d* vengono rilevati due percorsi che
differiscono per il loro  fingerprint e se questa  situazione si mantiene per un certo lasso di  tempo. Tale tempo
di tolleranza è direttamente proporzionale alla somma delle  latenze associate ai due percorsi che differiscono;
ma il modulo QSPN  non ha questa informazione in quanto il costo associato ad un percorso  non sappiamo se sia espresso
in latenza, in larghezza di banda o in  altra metrica; quindi il calcolo di tale tempo di tolleranza va  demandato
all'utilizzatore del modulo il quale fornisce una callback attraverso l'istanza di IQspnThresholdCalculator.  Per dare
il massimo del supporto a questa callback vengono passate le  istanze di IQspnNodePath che rappresentano i percorsi
discordi. Da  queste la callback potrà estrapolare i costi e sommarli se si tratta in  effetti di latenze. Altrimenti
la callback potrà leggere la destinazione  e confrontarla con il proprio indirizzo per risalire al livello del
minimo g-nodo comune, chiedere allo stesso modulo QSPN la stima dei nodi  all'interno di tale livello e finalmente proporre
un tempo di  tolleranza sulla base di questo dato. Ricevuto questo tempo, il modulo si occupa di emettere alla sua scadenza
il segnale *gnode_splitted* di QspnManager se la situazione si è mantenuta.

#### <a name="Variazioni_mappa_Algoritmo"></a>Algoritmo

Il nodo *n*, dopo aver rielaborato i percorsi nel set di ETP ricevuti, si trova infine con un set di percorsi nuovi che
chiamiamo *Q*. Indichiamo con *M* l'insieme dei percorsi che già prima il nodo *n* conosceva, cioè la sua mappa.

*   Il nodo *n* prepara, per ogni livello *i* da 0 a *l* - 1, un insieme *Z<sub>i</sub>* con i g-nodi di livello *i* diretti
    vicini del suo g-nodo di livello *i*.
*   Il nodo *n* prepara una lista *P*, inizialmente vuota, di percorsi che andranno segnalati ai vicini a cui si inoltrerà l'ETP.
*   Il nodo *n* prepara una lista *B*, inizialmente vuota, di destinazioni per le quali alla fine intende iniziare un nuovo
    flood. Questo per la regola di primo rilevamento di split.
*   Il nodo *n* raggruppa i percorsi contenuti in *Q* per destinazione.
*   Ordina le destinazioni per fare in modo che siano elaborate per prime le destinazioni più interne (con livello più basso);
    inoltre, a parità di livello, siano elaborate per prime le destinazioni più vicine, cioè quelle per le quali il percorso
    più rapido ha un numero minore di hops in quel livello.  
    Questo previo ordinamento fa in modo che almeno uno dei percorsi verso ogni destinazione sarà memorizzato con successo.
    Infatti, se quando il nodo elabora un percorso esso contiene un hop che ancora non è conosciuto dal nodo come possibile
    destinazione, tale percorso va scartato. Purtroppo questo non è sufficiente a rendere subito validi tutti i percorsi, ma
    lo saranno al prossimo ETP.
*   Per ogni destinazione *d* :
    *   Sia *Q<sub>d</sub>* l'insieme dei percorsi nuovi verso *d*.
    *   Sia *M<sub>d</sub>* l'insieme dei percorsi precedentemente noti verso *d*.
    *   Se *d.lvl* > 0:
        *   Il nodo *n* prepara una lista *F* ’ con tutti i fingerprint distinti che conosceva per il g-nodo *d*.
    *   Il nodo *n* prepara una lista *O<sub>d</sub>*, inizialmente vuota, di percorsi verso *d* che sono correnti e andranno ordinati.
    *   Il nodo *n* prepara una lista *V<sub>d</sub>*, inizialmente vuota, di percorsi verso *d* che hanno subito una variazione.
    *   Il nodo *n* prepara una lista *S<sub>d</sub>*, inizialmente vuota, di segnali da emettere.
    *   Per ogni *p’* ∈ *M<sub>d</sub>* :
        *   Il nodo *n* controlla se esiste un  *p’’* ∈ *Q<sub>d</sub>* che abbia la stessa sequenza di passi. Questo
            confronto fra *p’* e *p’’*, che sono entrambi istanze di NodePath, avviene con il metodo *hops_arcs_equal*
            che tiene conto delle liste hops e arcs. Se esiste:
            *   Siamo di fronte a un aggiornamento sulle informazioni relative ad un percorso che era già noto. Può cambiare
                il suo costo, il fingerprint riportato, il numero di nodi  interni riportato. Se si tratta di un cambio di
                fingerprint la modifica  viene apportata. Se il fingerprint non cambia, allora si valuta se  almeno una delle
                altre variazioni supera una data soglia (ad esempio il  30% del costo o il 10% del numero di nodi) per
                decidere se apportare la  modifica nella propria mappa. Infatti se la modifica è non cruciale e non
                rilevante si preferisce ignorarla per ridurre il traffico di rete.
            *   Se l'aggiornamento è giudicato rilevante:
                *   *p’’* viene scartato da *Q<sub>d</sub>*.
                *   *p’’* viene aggiunto a *O<sub>d</sub>*.
                *   *p’’* viene aggiunto a *V<sub>d</sub>*.
            *   Altrimenti:
                *   *p’’* viene scartato da *Q<sub>d</sub>*.
                *   *p’* viene aggiunto a *O<sub>d</sub>*.
                *   Se *p’.arc.i_qspn_equals(a_changed)* :
                    *   *p’* viene aggiunto a *V<sub>d</sub>*.
        *   Altrimenti:
            *   *p’* viene aggiunto a *O<sub>d</sub>*.
            *   Se *p’.arc.i_qspn_equals(a_changed)* :
                *   *p’* viene aggiunto a *V<sub>d</sub>*.
    *   Tutti i percorsi rimasti in *Q<sub>d</sub>* vengono aggiunti a *O<sub>d</sub>*.
    *   Poi il nodo *n* ordina *O<sub>d</sub>* in base al costo crescente. Esegue su questa lista l'algoritmo descritto
        nel documento [percorsi disgiunti](PercorsiDisgiunti.md) per scartare quelli non sufficientemente disgiunti e
        tenere solo i *max_paths* più rapidi. Nel fare questo tiene conto dei requisiti che vincolano i percorsi che il
        nodo *n* può scartare: per questo avrà bisogno dei set *Z<sub>i</sub>* e del set dei diretti vicini. Alla fine in
        *O<sub>d</sub>* non possono rimanere percorsi con costo = *dead*.
    *   Infine il nodo *n* sulla base del contenuto delle liste di appoggio prima descritte (*O<sub>d</sub>*, *M<sub>d</sub>*
        e *V<sub>d</sub>*) crea nuove istanze di EtpPath per popolare la lista *P* che andrà a comporre l'ETP che sarà
        inoltrato ai vicini e allo stesso tempo compone il set *S<sub>d</sub>* dei segnali da emettere per questa
        destinazione. Illustriamo l'algoritmo di questa operazione, tenendo conto che i confronti tra istanze di NodePath
        anche qui vanno fatti con il metodo *hops_arcs_equal* :
        *   *valid_fp<sub>d</sub>* = null.
        *   Se *d.lvl* > 0:
            *   Per ogni *p* ∈ *O<sub>d</sub>* :
                *   *fp<sub>d,p</sub>* = fingerprint di *d* come riportato da *p*.
                *   Se *valid_fp<sub>d</sub>* = null:
                    *   *valid_fp<sub>d</sub>* = *fp<sub>d,p</sub>*.
                *   Altrimenti Se *valid_fp<sub>d</sub>* è meno anziano di *fp<sub>d,p</sub>* :
                    *   *valid_fp<sub>d</sub>* = *fp<sub>d,p</sub>*.
        *   Per ogni *p* ∈ *O<sub>d</sub>* :
            *   Se *p* ∉ *M<sub>d</sub>* :
                *   *fp<sub>d,p</sub>* = fingerprint di *d* come riportato da *p*.
                *   *p2* = copia iniziale di *p*. Vedere sotto l'algoritmo di produzione di un percorso da inviare.
                *   Metti *p2* in *P*.
                *   Se *d.lvl* = 0:
                    *   Prepara un segnale di *path_added* e aggiungilo al set *S<sub>d</sub>*.
                *   Altrimenti:
                    *   Se *fp<sub>d,p</sub>* = *valid_fp<sub>d</sub>* :
                        *   Prepara un segnale di *path_added* e aggiungilo al set *S<sub>d</sub>*.
                        *   *p.exposed* = True.
                    *   Altrimenti:
                        *   Niente.
        *   Per ogni *p* ∈ *M<sub>d</sub>* :
            *   *fp<sub>d,p</sub>* = fingerprint di *d* come riportato da *p*.
            *   Se *p* ∉ *O<sub>d</sub>* :
                *   *p2* = copia iniziale di *p*.
                *   *p2.cost* = *dead*.
                *   Metti *p2* in *P*.
                *   Se *d.lvl* = 0:
                    *   Prepara un segnale di *path_removed* e aggiungilo al set *S<sub>d</sub>*.
                *   Altrimenti:
                    *   Se *p.exposed* :
                        *   Prepara un segnale di *path_removed* e aggiungilo al set *S<sub>d</sub>*.
                    *   Altrimenti:
                        *   Niente.
            *   Altrimenti:
                *   Sia *p1* l'istanza in *O<sub>d</sub>* equivalente di *p*, che la sostituirà in *M<sub>d</sub>*.
                *   Se *p* ∈ *V<sub>d</sub>* :
                    *   *p2* = copia iniziale di *p*.
                    *   Metti *p2* in *P*.
                    *   Se *d.lvl* = 0:
                        *   Prepara un segnale di *path_changed* e aggiungilo al set *S<sub>d</sub>*.
                    *   Altrimenti:
                        *   Se *p.exposed* :
                            *   Se *fp<sub>d,p</sub>* = *valid_fp<sub>d</sub>* :
                                *   Prepara un segnale di *path_changed* e aggiungilo al set *S<sub>d</sub>*.
                                *   *p1.exposed* = True.
                            *   Altrimenti:
                                *   Prepara un segnale di *path_removed* e aggiungilo al set *S<sub>d</sub>*.
                        *   Altrimenti:
                            *   Se *fp<sub>d,p</sub>* = *valid_fp<sub>d</sub>* :
                                *   Prepara un segnale di *path_added* e aggiungilo al set *S<sub>d</sub>*.
                                *   *p1.exposed* = True.
                            *   Altrimenti:
                                *   Niente.
    *   Il nodo *n* sostituisce *O<sub>d</sub>* al vecchio *M<sub>d</sub>*. Cioè:
        *   Se *M<sub>d</sub>* = ∅ e *O<sub>d</sub>* ≠ ∅, allora inserisce in testa al set *S<sub>d</sub>* il
            segnale *destination_added*.
        *   Se *M<sub>d</sub>* ≠ ∅ e *O<sub>d</sub>* = ∅, allora aggiunge in coda al set *S<sub>d</sub>* il
            segnale *destination_removed*.
        *   Rimuove da *M* tutti i percorsi verso *d* e vi aggiunge tutti i percorsi che ora sono in *O<sub>d</sub>*.
        *   Emette tutti i segnali che sono in *S<sub>d</sub>*.
    *   Se *d.lvl* > 0:
        *   Il nodo *n* prepara una lista *F* ’’ con tutti i fingerprint distinti che conosce adesso per il g-nodo *d*.
        *   Valuta dal confronto tra *F* ’ ed *F* ’’ se è necessario iniziare un nuovo flood di ETP con i percorsi
            per *d* per la regola di primo rilevamento di split. Cioè:
            *   Se *F* ’’.size > 1:
                *   Per ogni *fp* ∈ *F* ’’:
                    *   Se *fp* ∉ *F* ’:
                        *   Se *d* ∉ *B* :
                            *   Metti *d* in *B*.
                        *   Esci dal ciclo.
                *   Sia *fp_eldest* il più anziano in *F* ’’.
                *   Sia *bp_eldest* il miglior percorso verso *d* che riporta come fingerprint *fp_eldest*.
                *   Rimuovi *fp_eldest* da *F* ’’.
                *   Per ogni *fp* ∈ *F* ’’:
                    *   Sia *bp* il miglior percorso verso *d* che riporta come fingerprint *fp*.
                    *   Avvia in una nuova tasklet:
                        *   Sia una lista globale del modulo *pending_gnode_split*.
                        *   Se la coppia (*fp_eldest*, *fp* ) è presente in *pending_gnode_split* :
                            *   Esci.
                        *   Metti la coppia (*fp_eldest*, *fp* ) in *pending_gnode_split*.
                        *   Calcola il tempo di tolleranza prima della segnalazione dello split. Si calcola con
                            il *threshold_calculator* passando le istanze *bp_eldest* e *bp*.
                        *   Aspetta il tempo di tolleranza.
                        *   Togli la coppia (*fp_eldest*, *fp* ) da *pending_gnode_split*.
                        *   Se per la destinazione *d* è ancora noto un percorso che riporta il fingerprint *fp_eldest* :
                            *   Per ogni arco *a* nei miei archi:
                                *   Se il vicino collegato tramite *a* appartiene a *d* :
                                    *    Se il percorso per *d* tramite *a* riporta il fingerprint *fp* :
                                         *    Emetti il segnale *gnode_splitted* indicando *a*, *d* ed *fp*.
*   Ora la mappa di *n* è aggiornata.
*   Il nodo *n* cicla la lista *P* per finalizzare le istanze dei percorsi che andranno segnalati ai vicini a cui si
    inoltrerà l'ETP. Vedere sotto l'algoritmo di produzione di un percorso da inviare.

Ora la mappa di *n* è aggiornata. Inoltre abbiamo in *P* l'elenco dei percorsi nella mappa di *n* che hanno subito variazioni.

Infine abbiamo in *B* tutte le destinazioni per cui si ritiene necessario inviare un nuovo flood. Quindi:

*   Se *B* ≠ ∅:
    *   Il nodo *n* prepara un nuovo ETP con tutti i percorsi che conosce verso le destinazioni *d* ∈ *B*. Lo invia
        a tutti i suoi vicini.

### <a name="Variazioni_propri_gnodi"></a>Variazioni nei propri g-nodi

Dopo aver completato un aggiornamento della propria mappa, il nodo *n* aggiorna anche le informazioni relative ai
propri g-nodi, cioè fingerprint e numero di nodi all'interno. Se ci sono state variazioni *n* se ne accorge e questo
è importante per decidere se l'ETP ricevuto va inoltrato ai suoi vicini.

Supponiamo che il nodo *n* riceve un percorso *p* verso la destinazione *d* che prima non conosceva. La destinazione
*d* è un g-nodo di livello *i* all'interno del suo g-nodo di livello *i* + 1. Nel percorso *p* è anche indicato il
fingerprint di *d* e il numero di nodi interni al g-nodo *d*. Il fingerprint di *d* va ora incluso nel set di fingerprint
di g-nodi di livello *i* da passare al metodo *i_qspn_construct* del fingerprint del g-nodo di livello *i* di *n* per
ottenere il fingerprint del g-nodo di livello *i* + 1 di *n*. Analogamente, il numero di nodi interni a *d* va a sommarsi
a quelli interni agli altri g-nodi di livello *i* e al numero di nodi interni al g-nodo di livello *i* di *n* per ottenere
il numero di nodi interni al g-nodo *i* + 1 di *n*.

Quando invece si riceve un percorso verso una destinazione *d* che già si conosceva attraverso altri percorsi, può capitare
che le informazioni riguardo *d* differiscano.

Se per un g-nodo *d* vengono rilevati due percorsi che differiscono per il loro  fingerprint e se questa  situazione si
mantiene per un certo lasso di  tempo, questo è sintomo  dello split del g-nodo *d*. Il modulo  lo segnalerà con un evento,
come descritto prima nell'algoritmo. Nel frattempo il modulo mantiene e trasmette tutti i percorsi verso *d* che conosce con
il loro fingerprint, ma soltanto i percorsi con il fingerprint assegnato dal più anziano *x* all'interno di *d* vengono
restituiti all'esterno del modulo con il metodo *get_paths_to(d)*.

Se per un g-nodo *d* vengono rilevati due percorsi che differiscono per il numero di nodi interni a *d*, invece, questo non
indica che il g-nodo *d* sia splittato,  infatti le variazioni nel numero di nodi possono venire ignorate se il  cambiamento
non è massiccio (per evitare eccessivo traffico). In questi  casi il nodo prende per buono il numero di nodi come riportato
dal percorso più veloce. Userà questa informazione per calcolare il numero  di nodi (stimato) all'interno del suo g-nodo di
livello *i* + 1.

Quando si calcolano il fingerprint e il numero di nodi interni al proprio g-nodo di livello *i*, per ogni g-nodo noto di
livello *i* - 1 si usano i dati riportati dai percorsi verso quel g-nodo che riportano il fingerprint più anziano, in caso
ce ne siano di discordi.

Un approccio particolare va attuato a livello *i* = 1. Considerando il proprio g-nodo di livello 1, i suoi componenti sono
singoli nodi. Per i singoli nodi non ha senso pensare ad uno split, quindi se due percorsi verso una data destinazione di
livello 0 riportano un fingerprint diverso può solo significare che il vecchio nodo è morto e un altro nodo ha preso la sua
posizione; è solo questione di tempo prima che uno dei due fingerprint scompaia. Non avrebbe senso inoltre cercare di stabilire
quale dei due fingerprint discordi sarebbe il più anziano, cercando di confrontare l'anzianità dei suoi g-nodi interni. Infine
c'è da dire che si sa con certezza che il numero di nodi interni ad ogni singolo membro di un g-nodo di livello 1 (cioè il
numero di nodi interni ad un singolo nodo) è 1. Queste osservazioni saranno chiare quando sotto verrà illustrato l'approccio
per i livelli maggiori di 1.

Illustriamo l'approccio da usare a livello 1. È noto il fingerprint *fp<sub>0</sub>* del mio nodo. Poniamo
*nn<sub>0</sub>* = 1. Siano *g<sub>1</sub>*, *g<sub>2</sub>*, ..., *g<sub>y</sub>* i nodi di livello 0 presenti nella
mia mappa. Il calcolo di *fp<sub>1</sub>* dipende da *fp<sub>0</sub>* e dai fingerprint *fp(g<sub>j</sub>)* degli altri
g-nodi. Il calcolo di *nn<sub>1</sub>* è dato da 1 (cioè *nn<sub>0</sub>*) più il numero degli altri nodi. Per ogni
*g<sub>j</sub>* prendiamo solo il percorso più rapido che abbiamo nella mappa e diamo per buono il fingerprint da esso riportato.

Illustriamo l'approccio da usare ai livelli maggiori. Si parte dal livello *i* = 2 e si sale fino a *l*. Diamo per buono
il fingerprint *fp<sub>i-1</sub>* del mio g-nodo di livello *i* - 1 e il numero di nodi al suo interno *nn<sub>i-1</sub>*.
Siano *g<sub>1</sub>*, *g<sub>2</sub>*, ..., *g<sub>y</sub>* i g-nodi di livello *i* - 1 presenti nella mia mappa. Il
calcolo di *fp<sub>i</sub>* dipende da *fp<sub>i-1</sub>* e dai fingerprint *fp(g<sub>j</sub>)* degli altri g-nodi. Allo
stesso modo il calcolo di *nn<sub>i</sub>* dipende da *nn<sub>i-1</sub>* e dal numero di nodi *nn(g<sub>j</sub>)* all'interno
degli altri g-nodi. Sia *g<sub>j</sub>* uno di questi e siano *p<sub>1</sub>(g<sub>j</sub>)*,
*p<sub>2</sub>(g<sub>j</sub>)*, ..., *p<sub>x</sub>(g<sub>j</sub>)* i percorsi noti verso *g<sub>j</sub>*. Raggruppiamo
i percorsi in base al fingerprint che ciascuno riporta, che potrebbero essere diversi. Prendiamo solo il gruppo di
percorsi che hanno il fingerprint che risulta essere stato assegnato dal g-nodo interno più anziano (metodo *i_qspn_elder_seed*).
Consideriamo questo fingerprint come il fingerprint *fp(g<sub>j</sub>)* di *g<sub>j</sub>*. Inoltre da questo gruppo
prendiamo il percorso più rapido e consideriamo il numero di nodi riportato da questo come il numero di nodi
*nn(g<sub>j</sub>)* interni a *g<sub>j</sub>*. Ripetiamo per tutti i g-nodi di livello *i* - 1 presenti nella mia mappa
e usiamo questi valori per il calcolo delle analoghe informazioni relative al mio g-nodo di livello *i*.

Quando queste informazioni portano ad una variazione nel fingerprint di uno dei g-nodi del nodo corrente, il QspnManager
emette il segnale *changed_fp*. Quando portano ad una variazione nel numero di nodi interni ad uno dei g-nodi del nodo
corrente, emette il segnale *changed_nodes_inside*.

#### <a name="Variazioni_propri_gnodi_Algoritmo"></a>Algoritmo

Dopo aver aggiornato la sua mappa sulla base di un set di ETP, il nodo *n* procede così per ricalcolare le informazioni dei propri g-nodi:

*   Sia *my_fingerprints[i]* il valore che *n* conosce attualmente come fingerprint del suo g-nodo di livello *i*.
*   Sia *my_nodes_inside[i]* il valore che *n* conosce attualmente come numero di nodi interni al suo g-nodo di livello *i*.
*   *changes_in_my_gnodes* = False.
*   Per il livello *i* = 1:
    *   *FP* = nuovo set vuoto.
    *   *NN* = 0.
    *   Per ogni destinazione *d* di livello *0* nella mia mappa:
        *   *best_p* = miglior percorso NodePath verso *d*.
        *   *fp<sub>d</sub>* = fingerprint di *d* come riportato da *best_p*.
        *   Aggiungi *fp<sub>d</sub>* al set *FP*.
        *   Somma 1 a *NN*.
    *   *new_fp* = *my_fingerprints[0].i_qspn_construct(FP)*.
    *   Se *new_fp* ≠ *my_fingerprints[1]* :
        *   *my_fingerprints[1]* = *new_fp*.
        *   *changes_in_my_gnodes* = True.
        *   Emetti segnale *changed_fp* a livello 1.
    *   *new_nn* = 1 + *NN*.
    *   Se *new_nn* ≠ *my_nodes_inside[1]* :
        *   *my_nodes_inside[1]* = *new_nn*.
        *   *changes_in_my_gnodes* = True.
        *   Emetti segnale *changed_nodes_inside* a livello 1.
*   Per ogni livello *i* da 2 a *l* :
    *   *FP* = nuovo set vuoto.
    *   *NN* = 0.
    *   Per ogni destinazione *d* di livello *i* - 1 nella mia mappa:
        *   *fp<sub>d</sub>* = null.
        *   *nn<sub>d</sub>* = -1.
        *   *best_p* = null.
        *   Per ogni NodePath *p* in *d* :
            *   *fp<sub>d,p</sub>* = fingerprint di *d* come riportato da *p*.
            *   *nn<sub>d,p</sub>* = numero nodi di *d* come riportato da *p*.
            *   Se *fp<sub>d</sub>* = null:
                *   *fp<sub>d</sub>* = *fp<sub>d,p</sub>*.
                *   *nn<sub>d</sub>* = *nn<sub>d,p</sub>*.
                *   *best_p* = *p*.
            *   Altrimenti:
                *   Se *fp<sub>d</sub>* ≠ *fp<sub>d,p</sub>* :
                    *   Se *fp<sub>d</sub>* è meno anziano di *fp<sub>d,p</sub>* :
                        *   *fp<sub>d</sub>* = *fp<sub>d,p</sub>*.
                        *   *nn<sub>d</sub>* = *nn<sub>d,p</sub>*.
                        *   *best_p* = *p*.
                *   Altrimenti:
                    *   Se *p.cost* < *best_p.cost* :
                        *   *nn<sub>d</sub>* = *nn<sub>d,p</sub>*.
                        *   *best_p* = *p*.
        *   Aggiungi *fp<sub>d</sub>* al set *FP*.
        *   Somma *nn<sub>d</sub>* a *NN*.
    *   *new_fp* = *my_fingerprints[i-1].i_qspn_construct(FP)*.
    *   Se *new_fp* ≠ *my_fingerprints[i]* :
        *   *my_fingerprints[i]* = *new_fp*.
        *   *changes_in_my_gnodes* = True.
        *   Emetti segnale *changed_fp* a livello *i*.
    *   *new_nn* = *my_nodes_inside[i-1]* + *NN*.
    *   Se *new_nn* ≠ *my_nodes_inside[i]* :
        *   *my_nodes_inside[i]* = *new_nn*.
        *   *changes_in_my_gnodes* = True.
        *   Emetti segnale *changed_nodes_inside* a livello *i*.

Ora abbiamo in *changes_in_my_gnodes* se sono state apportate variazioni alle informazioni relative ai g-nodi di *n*.

### <a name="Produzione_percorsi_etp_da_inviare"></a>Produzione di percorsi da mettere in un ETP da inviare

La valorizzazione dei dati in una istanza di EtpPath, cioè un percorso da includere in un ETP da inviare, si fa in due fasi.

La prima fase avviene durante l'algoritmo con cui il nodo *n* aggiorna la sua mappa partendo dai nuovi percorsi
noti. In questa fase si valorizzano tutti i dati che possono essere ricavati dalla istanza di NodePath che è stata
aggiunta/variata/rimossa nella mappa di *n*. 

Sia *q* tale istanza di NodePath. Indichiamo con *q.path* l'istanza di EtpPath inclusa in *q*. Indichiamo con
*q.arc* l'istanza di IQspnArc inclusa in *q*.

Il nodo *n* prepara una nuova istanza *p* di EtpPath. Copia tutte le informazioni da *q.path* a *p*; come costo
indica *q.cost*, cioè la somma dei costi *q.path.cost* e *q.arc.cost*. Poi *p* viene messa in un set *P* con cui comporre l'ETP.

La seconda fase avviene alla fine degli aggiornamenti alla mappa. Questo perché in questa fase si valorizzano i dati
che vanno ricavati non solo dalla istanza di NodePath in esame ma anche da altre informazioni contenute nella mappa di *n*.

Il nodo *n* cicla tutte le istanze *p* in *P*. Per ognuna, ricalcola tutti i valori booleani che indicano se il
percorso *p* va ignorato all'esterno di un suo g-nodo.

*   *p.ignore_outside[0]* = False.
*   Per ogni livello *i* da 1 a *l* - 1:
    *   Se *p.hops.last().lvl* ≥ *i* :
        *   *j* = 0.
        *   Mentre True:
            *   Se *p.hops[j].lvl* ≥ *i* :
                *   Esci dal ciclo.
            *   Incrementa *j* di 1.
        *   Se non esiste nella mia mappa la destinazione *d* che rappresenta *p.hops[j]* :
            *   Se *p.cost.is_dead()* :
                *   *p.ignore_outside[i]* = False.
                *   continue.
            *   Altrimenti:
                *   assert_not_reached.
        *   *best_to_arc* = null.
        *   Per ogni NodePath *q* in *d* :
            *   Se *q.arcs.last()* = *p.arcs[j]* :
                *   Se *best_to_arc* = null:
                    *   *best_to_arc* = *q*.
                *   Altrimenti:
                    *   Se *q.cost* < *best_to_arc.cost* :
                        *   *best_to_arc* = *q*.
        *   Se *best_to_arc* = null:
            *   *p.ignore_outside[i]* = False.
        *   Altrimenti:
            *   *same* = False.
            *   Se *best_to_arc.hops.size* = *j* - 1:
                *   *same* = True.
                *   Per *k* che va da 0 a *j* - 1:
                    *   Se *best_to_arc.hops[k]* ≠ *p.hops[k]* :
                        *   *same* = False.
                        *   Esci dal ciclo.
                    *   Se *best_to_arc.arcs[k]* ≠ *p.arcs[k]* :
                        *   *same* = False.
                        *   Esci dal ciclo.
            *   *p.ignore_outside[i]* = NOT *same*.
    *   Altrimenti:
        *   *p.ignore_outside[i]* = True.

Alla fine tutto il contenuto di *P* è pronto per essere messo nell'EtpMessage da inviare.

