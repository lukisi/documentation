# Modulo QSPN - Esplorazione della rete

1.  [Concetti e termini](#Concetti_e_termini)
    1.  [TP - Tracer Packet](#Tracer_packet)
    1.  [ATP - Acyclic Tracer Packet](#Acyclic_tracer_packet)
    1.  [CTP - Continuous Tracer Packet](#Continuous_tracer_packet)
    1.  [Interesting information rule](#Interesting_information_rule)
    1.  [ETP - Extended Tracer Packet](#Extended_tracer_packet)
1.  [Implementazione dell'esplorazione](#Implementazione_esplorazione)
    1.  [Maggiori dettagli sull'ETP](#Dettagli_etp)
    1.  [Rete completamente esplorata](#Rete_esplorata)
1.  [Struttura gerarchica degli indirizzi](#Struttura_gerarchica_indirizzi)
    1.  [Rappresentazione gerarchica degli ID](#Rappresentazione_gerarchica_id)
    1.  [Informazioni aggiuntive sulla destinazione](#Informazioni_aggiuntive_sulla_destinazione)
    1.  [Minimo comune g-nodo e massimo distinto g-nodo](#Minimo_comune_gnodo)
    1.  [Percorso a livelli crescenti](#Percorso_a_livelli_crescenti)
    1.  [Dichiarazione di percorso da ignorare all'esterno di un g-nodo](#Dichiarazione_percorso_da_ignorare)
        1.  [Arco di uscita dal g-nodo](#Arco_di_uscita)
        1.  [Percorsi interni al g-nodo](#Percorsi_interni)
    1.  [Rimozione dei percorsi da ignorare](#Rimozione_percorsi_da_ignorare)
    1.  [Definizione di grouping rule](#Definizione_grouping_rule)
        1.  [Applicazione della grouping rule sulla lista di hops percorsi dall'ETP](#Applicazione_grouping_rule)
    1.  [Definizione di acyclic rule](#Definizione_acyclic_rule)
        1.  [Applicazione della acyclic rule sulla lista di hops percorsi dall'ETP](#Applicazione_acyclic_rule)
    1.  [Contenuto e forma di un messaggio ETP](#Contenuto_forma_messaggio_etp)
        1.  [Informazioni sui percorsi da ignorare](#Informazioni_sui_percorsi_da_ignorare)
        1.  [Informazioni aggiuntive riguardo la destinazione](#Informazioni_aggiuntive_destinazione)
    1.  [Ingresso di un g-nodo in una rete](#Ingresso_gnodo_in_rete)
        1.  [Osservazione sugli indirizzi IP interni](#Osservazione_indirizzi_ip_interni)

## <a name="Concetti_e_termini"></a>Concetti e termini

Questa prima parte non descrive quali messaggi vengano effettivamente passati da un nodo all'altro.

In questa prima parte esaminiamo i concetti generali e introduciamo alcuni termini, per avere una idea di quali
considerazioni abbiano portato alla scelta dell'effettiva implementazione che sarà descritta più sotto.

#### <a name="Tracer_packet"></a>TP - Tracer Packet

Un TP *flood* viene avviato da un nodo *s*. Per farlo *s* genera un TP, ci scrive il suo ID e lo invia a tutti
i suoi vicini. Un nodo *n* che riceve il TP vi aggiunge il suo ID, un identificativo univoco dell'arco *a* attraverso
il quale lo ha ricevuto e il costo dell'arco *c(a)* e poi lo inoltra a tutti i suoi vicini tranne quello da
cui lo ha ricevuto.

Quando un nodo invia un TP (questo vale sia per il nodo *s* che inizia il flood sia per il generico nodo *n* che
lo inoltra) lo invia a più di un vicino, quindi invia in effetti un insieme di TP. Quando questi TP raggiungono
un altro nodo *v* possono quindi aver compiuto percorsi diversi; grazie al loro contenuto il nodo *v* sa valutare
per ogni percorso il suo costo, sommando i costi di ogni singolo arco attraversato.

Un nodo *v* può ricevere più di una volta un TP proveniente da uno stesso flood, ma lo inoltra solo se il percorso
contenuto risulta essere il percorso con costo minore da *v* ad *s*. Quindi se un nodo *s* inizia un TP flood, tutti
i nodi scropriranno per ogni loro vicino il percorso migliore verso *s*.

#### <a name="Acyclic_tracer_packet"></a>ATP - Acyclic Tracer Packet

Un ATP è un TP che, quando un nodo lo riceve lo inoltra a tutti i suoi vicini tranne quello da cui lo riceve,
anche se il flood era già passato da lì. Un ATP non viene inoltrato da un nodo *n* se l'ID di *n* era già presente
in esso. Questo lo rende aciclico. Quindi se un nodo *s* inizia un ATP flood tutti i nodi scopriranno tutti i
possibili percorsi verso *s*, senza cicli.

#### <a name="Continuous_tracer_packet"></a>CTP - Continuous Tracer Packet

Un CTP è un TP che, quando un nodo lo riceve lo inoltra sempre a tutti i suoi vicini tranne quello da cui lo
riceve. Inoltre, se il nodo ricevente ha come unico vicino quello che glielo ha inoltrato allora svuota la lista
di ID e lo inoltra di nuovo al nodo che glielo aveva inoltrato. Quindi un CTP non cessa mai di esplorare tutti
i percorsi della rete.

#### <a name="Interesting_information_rule"></a>Interesting information rule

La regola dell'informazione interessante può essere aggiunta alla decisione sull'inoltro di un generico TP
(TP o ATP o CTP). Questa regola dice che un TP non va inoltrato se non contiene nessuna informazione interessante
per il nodo. Ad esempio nel caso di percorsi se tutti i percorsi in esso presenti erano già noti al nodo.

#### <a name="Extended_tracer_packet"></a>ETP - Extended Tracer Packet

Un ETP è un ATP che contiene una porzione di una mappa, cioè un set *P* di percorsi. Ad esso si associa la
regola dell'informazione interessante, considerando tutti i percorsi contenuti come parte dell'informazione.
Quindi un nodo *n* che riceve l'ETP lo inoltra (a tutti i suoi vicini tranne quello da cui lo riceve) se l'ETP
non ha fatto già un ciclo (non c'è già l'ID di *n*) **e** i percorsi contenuti in *P* hanno apportato qualche
aggiornamento alla mappa di *n*.

## <a name="Implementazione_esplorazione"></a>Implementazione dell'esplorazione

Si descrive ora come viene implementata l'esplorazione di una rete. Affrontiamo subito il problema della
dinamicità della rete, cioè operiamo in un grafo che cambia nel tempo in termini di vertici o di archi o di costo degli archi.

Si noti invece che in questo capitolo non introduciamo subito l'aspetto della struttura gerarchica che viene
imposta sugli indirizzi della rete per ridurre la quantità di informazioni memorizzate in ogni singolo nodo.
Le implicazioni di questa strutturazione verranno descritte nel capitolo successivo.

Tuttavia, in alcuni passaggi verranno fatte delle annotazioni che riguardano concetti relativi alla struttura
gerarchica, come il concetto dei g-nodi a cui un nodo appartiene. Questi concetti dovrebbero comunque essere
stati assimiliati dal lettore nel documento di [analisi](AnalisiFunzionale.md). Si prosegua comunque la lettura
considerando che in seguito tali aspetti verranno dettagliati maggiormente.

### <a name="Dettagli_etp"></a>Maggiori dettagli sull'ETP

Diciamo subito che l'esplorazione della rete avviene attraverso la comunicazione da un nodo ai suoi vicini di messaggi ETP.

Abbiamo detto che un generico TP è costituito da un elenco di hop, ognuno dei quali contiene l'ID del nodo da
cui è passato, l'identificativo dell'arco attraverso il quale è stato comunicato da questo al nodo successivo
e il costo di tale arco. L'insieme di tali informazioni costituisce un percorso *p*.

Correggiamo questi dati dicendo che i costi dei vari archi attraversati vengono sommati, di passaggio in
passaggio, e quindi le informazioni contenute in *p* sono:

*   Una sequenza di ID dei nodi attraversati.
*   Una sequenza di identificativi degli archi percorsi.
*   Il costo totale del percorso.

Inoltre abbiamo detto che un ETP è esso stesso un TP (aciclico) e quindi ha anche esso un elenco di hop. Diciamo
però che questo elenco di hop non contiene tutte le informazioni che contiene un percorso *p*. L'elenco di hop
attraversati da un ETP contiene solo:

*   Una sequenza di ID dei nodi attraversati.

Ricordiamo che un ETP contiene, oltre alla sua sequenza di ID dei nodi attraversati, anche un set *P* di
percorsi *p* che vengono "pubblicizzati" dal nodo che trasmette l'ETP ai suoi vicini.

### <a name="Rete_esplorata"></a>Rete completamente esplorata

Una rete si dice *completamente esplorata*, per brevità diciamo *esplorata*, se tutti i nodi hanno tutte le
conoscenze di loro pertinenza. L'obiettivo che ci siamo prefissati è descritto nel dettaglio nel documento
di analisi funzionale: in sintesi, ogni nodo deve avere per ogni destinazione un numero di percorsi rapidi e
tra loro [disgiunti](PercorsiDisgiunti.md). I dettagli sulle informazioni per ogni percorso verranno man mano
esplicitati in questo documento. In seguito nel documento indicheremo questo insieme di conoscenze di un nodo
con il termine *mappa* di *n*.

Una rete inizia come singolo nodo. Quando inizia è quindi *esplorata* e il nodo considera completata la sua
fase di *bootstrap* (la definizione di tale fase verrà chiarificata sotto).

Sia G una rete *esplorata*. Consideriamo tutti i possibili eventi che cambiano il grafo e quindi rendono la
rete non più *esplorata*. Vediamo quali operazioni sono necessarie a farla ridiventare *esplorata*.

Tali eventi, come si può vedere dall'elenco che segue, sono riconducibili alla nascita/variazione/morte di
archi tra due nodi vicini. Questi eventi sono tutti notificati al modulo QSPN dei nodi diretti interessati,
come si è detto nei [requisiti](AnalisiFunzionale.md#requisiti) dell'analisi funzionale.

**_Evento hooked_**: Sia *n* un nodo appena entrato nella rete G grazie ad un set di archi verso nodi vicini.

Il nodo *n* si considera in fase di *bootstrap*. Durante questa fase il nodo conosce l'esistenza della rete
G di cui vuole entrare a far parte con un dato indirizzo e questo indirizzo viene passato al modulo QSPN.
Il programma che è in esecuzione nel sistema sa riconoscere se su un sistema vicino che ha rilevato
esiste una identità (cioè un nodo) che appartiene a tale rete. Solo se è così il programma costruisce un
arco-identità con quel nodo e comunica questo arco-identità al modulo QSPN di *n*. Il modulo QSPN di *n* non conosce ancora
tutti i percorsi di sua pertinenza, quindi nemmeno tutte le destinazioni che esistono nella rete; il modulo
QSPN del nodo non è in grado di computare il fingerprint dei g-nodi a cui appartiene. In queste condizioni il
modulo non è in grado di produrre un ETP per i suoi vicini.

Mentre è in corso la fase di bootstrap:

*   Il nodo *n* copia la lista dei suoi archi in una lista temporanea, *queued_arcs*.
*   Se *n* riceve un ETP spontaneamente inviato da un vicino (non quelli richiesti da *n* di cui parliamo subito)
    il modulo QSPN lo ignora.
*   Se *n* rileva un nuovo arco, il modulo lo aggiunge alla lista dei suoi archi e in fondo alla lista *queued_arcs*.
    Non esegue altre azioni.
*   Se *n* rileva un cambio di costo di un suo arco, il modulo non esegue alcuna azione.
*   Se *n* rileva la rimozione di un suo arco, il modulo lo rimuove dalla lista dei suoi archi e dalla lista
    *queued_arcs*. Non esegue altre azioni.
*   Se *n* riceve la richiesta di produrre un ETP la rifiuta "perché in fase di bootstrap".

Se la lista *queued_arcs* non è vuota, il nodo *n* chiede un ETP completo al vicino collegato col primo elemento
della lista *queued_arcs*. Se riceve come risposta un rifiuto perché il vicino non ha completato il suo bootstrap,
*n* ignora quell'arco, confidando che sarà lo stesso nodo *v*, una volta completato il suo bootstrap, ad iniziare
un flood di ETP verso ognuno dei suoi vicini. Quindi *n* rimuove l'arco dalla lista temporanea *queued_arcs* e può
provare con il prossimo primo elemento. Se invece riceve un ETP come risposta (un solo ETP da parte di qualsiasi
vicino è già sufficiente in una rete "flat", vedremo in seguito cosa comporta una rete gerarchica) *n* può uscire
dalla fase di bootstrap.

Può succedere come caso estremo che *n* non riceve nessun ETP: in questo caso *n* non è entrato in G. Il nodo *n*
deve in questo caso considerarsi come un singolo nodo isolato che costituisce tutta la rete. Il nodo rimuove tutti
i suoi archi. Il nodo a questo punto non ha percorsi nella mappa, esce lo stesso dalla fase di bootstrap.

Al di fuori del caso estremo appena descritto, abbiamo detto che con il primo ETP ricevuto il nodo *n* aggiorna
la sua mappa ed ha completato la sua fase di bootstrap.

Pur essendo ormai uscito dalla fase di bootstrap, il nodo *n* completa le sue conoscenze chiedendo, uno alla volta,
un ETP completo ad ogni vicino tramite ogni arco della lista *queued_arcs*. Per ogni ETP aggiorna la sua mappa.

Alla fine il nodo *n* prepara un nuovo ETP completo e lo invia in broadcast a tutti i vicini.

**_Evento added-link_**: Sia il nodo *n* ∈ G. *n* rileva la nascita di un nuovo arco con un vicino *v* ∈ G.

Il  nodo *n* chiede a *v* un nuovo ETP completo.

La richiesta di un ETP potrebbe venire rifiutata da *v* perché non ha completato il bootstrap. In questo caso
*n* rinuncia all'aggiornamento, confidando che sarà lo stesso nodo *v*, una volta completato il suo bootstrap, ad
iniziare un flood di ETP verso ognuno dei suoi vicini.

Altrimenti, con  le informazioni in questo ETP il nodo *n* aggiorna la sua mappa. *n* prepara tutti i percorsi
della sua mappa che  hanno subito una variazione e li mette in un set *P*. Se si tratta di un percorso nuovo o
cambiato basta memorizzare in *P* il percorso con i valori correnti. Se si tratta di un percorso rimosso dalla
mappa lo si memorizza nella lista *P* con costo *dead*.

Inoltre il nodo *n* aggiorna tutte le informazioni relative ai suoi g-nodi, cioè fingerprint e numero di nodi
interni, sulla base delle conoscenze aggiornate nella sua mappa. Controlla se tali informazioni sono effettivamente
cambiate rispetto a prima.

Se *P* risulta non vuoto, oppure se sono state apportate variazioni alle informazioni relative ai propri g-nodi,
e solo se il nodo *n* ha altri vicini oltre a *v*, il nodo *n* mette *P* e il suo ID in un nuovo ETP.  Invia l'ETP
in broadcast a tutti i vicini tranne *v*.

Poi *n* prepara, spontaneamente, un  nuovo ETP completo e lo invia a *v*. Questo è un ETP ulteriore rispetto all'ETP
che lo stesso nodo *v* potrebbe aver chiesto nel frattempo a *n* in quanto anche esso si è accorto del nuovo link.

**_Evento changed-link_**: Sia il nodo *n* ∈ G con un arco verso *v*. *n* rileva che l'arco verso *v* cambia il suo costo.

Il nodo *n* chiede a tutti i suoi vicini un nuovo ETP completo. Con la somma delle informazioni da tutti questi
ETP e i costi dei link ai suoi vicini (di cui uno è cambiato), il nodo *n* aggiorna la sua mappa. *n* prepara il
set *P* con tutti i percorsi della sua mappa che hanno subito una variazione.

Inoltre il nodo *n* aggiorna tutte le informazioni relative ai suoi g-nodi e controlla se tali informazioni sono
effettivamente cambiate rispetto a prima.

Se *P* risulta non vuoto, oppure se sono state apportate variazioni alle informazioni relative ai propri g-nodi,
il nodo *n* mette *P* e il suo ID in un nuovo ETP. Invia l'ETP in broadcast a tutti i vicini.

**_Evento removed-link_**: Sia il nodo *n* ∈ G con un arco *a* verso *v*. *n* rileva che l'arco *a* non permette
più di raggiungere il vicino.

Il nodo *n* rimuove l'arco *a* dai suoi archi. Poi rimuove dalla sua mappa tutti i percorsi che iniziavano con
l'arco *a* e li scrive nel set *P* con costo = *dead* . Poi *n* si comporta come per il caso di cambio di costo
dell'arco. L'invio in broadcast a tutti i vicini va fatto solo se *n* ha almeno un vicino.

**_Preparazione di un nuovo ETP completo_**: Sia il nodo *n* ∈ G. *n* intende inviare un ETP completo, cioè informare
i suoi vicini su tutti i percorsi che *n* conosce, anche quelli che aveva già comunicato in precedenza e non
hanno subito variazioni.

Un ETP *completo* deve essere riconosciuto come tale dal vicino che lo riceve. Infatti, se il nodo *q* riceve da
*n* un ETP completo *m* , *q* riceve intrinsecamente anche un'altra informazione di cui deve tenere conto quando
aggiorna la sua mappa: tutti i percorsi che *q* conosceva in precedenza e che hanno *n* come gateway, se non sono
riportati in *m.P* vanno rimossi dalla mappa di *q* , come se fossero riportati con costo = *dead* .

Quando un nodo *n* prepara un nuovo ETP completo, il suo contenuto dipende se lo sta preparando per tutti i suoi
vicini o per un vicino in particolare.

Se il destinatario è un nodo particolare *v*, il nodo *n* prepara in *P* tutti i percorsi della sua mappa che non
contengono *v*, né come gateway né come hop né come destinazione. *n* mette *P* e il suo ID in un ETP e lo invia solo a *v*.

Se l'ETP va inviato a tutti i vicini, il nodo *n* prepara in *P* tutti i percorsi della sua mappa. *n* mette *P*
e il suo ID in un ETP e lo invia in broadcast a tutti i vicini.

In entrambi i casi, il set *P* potrebbe risultare vuoto, ma l'ETP viene comunque prodotto e inviato. Questo perché
l'ETP contiene intrinsecamente l'informazione di cui abbiamo detto sopra e anche il percorso verso lo stesso nodo *n*.

<a name="EtpRicevuto"></a>

**_Evento etp-received_**: Quando un nodo *n* riceve un ETP da un vicino *v*.

Con le informazioni dell'ETP il nodo *n* aggiorna la sua mappa. *n* prepara il set *P* con tutti i percorsi della
sua mappa che hanno subito una variazione.

Inoltre il nodo *n* aggiorna tutte le informazioni relative ai suoi g-nodi e controlla se tali informazioni sono
effettivamente cambiate rispetto a prima.

Se *P* risulta non vuoto, oppure se sono state apportate variazioni alle informazioni relative ai propri g-nodi,
e solo se il nodo *n* ha altri vicini oltre a *v*, *n* produce un ETP con *P*, la lista di hop attraversati
dall'ETP ricevuto da *v* più l'ID di *n*. Il nuovo ETP viene inviato in broadcast a tutti i vicini tranne *v*.

**_Evento periodical-update_**: Sia il nodo *n* ∈ G. Sono passati 10 minuti dal momento in cui *n* ha completato
il suo bootstrap in G oppure dal precedente evento periodical-update.

Questo evento di fittizia variazione del grafo è aggiunto come misura di ridondanza. Se un vicino ha perso qualche
informazione interessante questa è una occasione per rivederla.

Il nodo *n*, solo se ha almeno un vicino, prepara un nuovo ETP completo e lo invia in broadcast a tutti i vicini.

* * *

Quando tutti gli ETP finiscono il loro ciclo di vita la rete è di nuovo *esplorata*.

## <a name="Struttura_gerarchica_indirizzi"></a>Struttura gerarchica degli indirizzi

In questo capitolo prendiamo in esame il fatto che ogni singolo nodo mantiene informazioni sulla rete che sono
limitate ad una visione gerarchica. Quindi anche quando le trasmette esse hanno questo limite. Vedremo cosa questo
comporta in termini di informazioni che devono essere trasmesse in ogni ETP in aggiunta al set di percorsi che abbiamo
prima introdotto. Essendo questo l'ultimo aspetto affrontato, descriveremo di seguito nel dettaglio come è fatto un
messaggio di ETP. Vedremo infine cosa comporta l'ingresso di un intero g-nodo in una rete esistente.

### <a name="Rappresentazione_gerarchica_id"></a>Rappresentazione gerarchica degli ID

Abbiamo già detto che un messaggio di ETP contiene, oltre alla lista di ID dei nodi che ha percorso, anche un set
*P* di percorsi *p*. Ogni percorso contiene una lista degli ID dei nodi che lo costituiscono, e per ognuno di
questi nodi contiene anche l'identificativo dell'arco attraverso il quale passa il percorso; infine contiene il suo costo totale.

Aggiungiamo ora che ogni ID può in effetti rappresentare o un singolo nodo oppure un g-nodo, diciamo genericamente
un g-nodo di livello *i* da 0 a *l* - 1. Anche la destinazione di ognuno dei percorsi nel set *P* è un g-nodo.

Ogni percorso *p* contiene due sequenze di *k* elementi:

*   *p.hops* : sequenza di k g-nodi.
*   *p.arcs* : sequenza di k identificativi di arco, dove `arcs[0]` indica l'arco che congiunge il nodo
    "pubblicizzante" a `hops[0]`, e `arcs[j]` indica l'arco che congiunge `hops[j-1]` a `hops[j]`.

Ogni g-nodo in queste liste è espresso in forma di coordinate gerarchiche che sono valide per il nodo pubblicizzante,
cioè il nodo che ha prodotto l'oggetto ETP. Gli identificativi degli archi in queste liste sono gli effettivi archi
che collegano un border-nodo del g-nodo precedente ad un border-nodo del g-nodo successivo.

### <a name="Informazioni_aggiuntive_sulla_destinazione"></a>Informazioni aggiuntive sulla destinazione

Aggiungiamo inoltre che ogni percorso del set *P* contiene alcune informazioni aggiuntive riguardo il g-nodo
che è la sua destinazione. Contiene il suo fingerprint e il numero approssimato di nodi nel suo interno.

### <a name="Minimo_comune_gnodo"></a>Minimo comune g-nodo e massimo distinto g-nodo

Introduciamo due definizioni che ci saranno utili nel resto del capitolo. Siano due nodi distinti *n* e *v*.

Il nodo *v* ha indirizzo *v<sub>l-1</sub>·...·v<sub>1</sub>·v<sub>0</sub>*. Vale a dire che *v* ha identificativo
*v<sub>0</sub>* all'interno del suo g-nodo di livello 1, il quale ha identificativo *v<sub>1</sub>* all'interno
del suo g-nodo di livello 2, ... fino al suo g-nodo di livello *l* - 1 che ha identificativo *v<sub>l-1</sub>* all'interno
dell'unico g-nodo di livello *l* che costituisce l'intera rete.

Il nodo *n* ha indirizzo *n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>* con analogo significato.

Sia *i*, con *i* < *l*, il più grande intero tale che *v<sub>i</sub>* ≠ *n<sub>i</sub>*, cioè il livello più alto
a cui *v* ed *n* non appartengono allo stesso g-nodo.

Definiamo *minimo comune g-nodo* tra *v* e *n* il g-nodo *v<sub>i+1</sub>* = *n<sub>i+1</sub>*, cioè il più piccolo
g-nodo che contiene sia *n* sia *v*. Potrebbe trattarsi del g-nodo a livello *l* che costituisce l'intera rete.
Siccome si tratta di uno dei g-nodi di *n* e anche uno dei g-nodi di *v*, per entrambi i nodi questo può essere
rappresentato semplicemente con l'intero *i* + 1.

Definiamo *massimo distinto g-nodo di v per n* il g-nodo *v<sub>i</sub>*, cioè il più grande g-nodo che contiene *v*
ma non contiene *n*. Per il nodo *n* questo può essere rappresentato come coordinata gerarchica a livello *i*. Invece
per il nodo *v* può essere rappresentato semplicemente con l'intero *i*.

Simmetricamente abbiamo che il *massimo distinto g-nodo di n per v* è il g-nodo *n<sub>i</sub>*, cioè il più grande
g-nodo che contiene *n* ma non contiene *v*.

### <a name="Percorso_a_livelli_crescenti"></a>Percorso a livelli crescenti

Si consideri che un ETP è un ATP, cioè è aciclico. Quando il nodo *n* riceve dal nodo *v* un ETP *m* che contiene già
il suo ID nella lista del percorso seguito, subito *n* ignora l'ETP *m*. Bisogna considerare che in questa frase con
il termine ID di *n* si intende l'identificativo del massimo distinto g-nodo di *n* per *v*.

Come conseguenza del fatto che non possono essere mantenuti o trasmessi percorsi  ciclici a nessun livello della
gerarchia, le liste di hop percorsi sono sempre in una forma in  cui i livelli salgono man mano che si avanza. Infatti
nel momento in cui un ATP esce da un g-nodo non può più rientrarvi. Ad esempio potrei avere il percorso
(0, 2) - (0, 5) - (1, 3) - (1, 7) - (2, 2) - (4, 2) - (4, 3) - (5, 3) - (5, 6). In questa rappresentazione delle
coordinate il primo numero indica il livello e il secondo l'identificativo.

### <a name="Dichiarazione_percorso_da_ignorare"></a>Dichiarazione di percorso da ignorare all'esterno di un g-nodo

Sia *m* un ETP prodotto dal nodo *v*. Quando questo abbandona il suo g-nodo di livello *i*, cioè quando viene
ricevuto dal nodo vicino *n* il cui minimo comune g-nodo con *v* è *v<sub>i+1</sub>*, esso deve perdere tutte
le informazioni interne al g-nodo *v<sub>i</sub>*. Infatti *n* considera tutti i nodi interni a *v<sub>i</sub>*
come raggruppati in un unico vertice.

Sia *p* ∈ *m.P* un percorso pubblicizzato da *v*. Tale percorso può essere valido per il nodo *n*, oppure no. In
alcuni casi lo stesso nodo *n* è in grado di avvedersene; ma in altri casi, che ora descriveremo, soltanto *v* è
in grado di stabilirlo. Per questo, nel messaggio *m*, per ogni percorso *p* e per ogni livello *i* (da 1 a *l* - 1)
va indicato se *p* è da ignorare all'esterno di *v<sub>i</sub>*.

Vediamo quali casi sono stati individuati.

#### <a name="Arco_di_uscita"></a>Arco di uscita dal g-nodo

Si consideri un ETP che abbandona un g-nodo *g* di livello *i* e giunge al nodo *n*. Ogni percorso noto a *n*, che
tocca il vertice *g*, avrà come successivo hop un altro vertice *h*, di livello *i* o maggiore. Il passaggio da *g*
ad *h* avviene attraverso un arco che è un arco realizzato da uno dei border-nodi di *g*.

Come fa il nodo *n* a distinguere i diversi possibili percorsi che toccano in sequenza i vertici *g* ed *h*? Il nodo
*n* può distinguere tanti diversi percorsi che toccano prima *g* e poi *h* quanti sono gli archi che congiungono
border-nodi di *g* a border-nodi di *h*. Se però un percorso *p1* e un percorso *p2* sono composti da hops diversi
internamente al g-nodo *g* per raggiungere uno stesso arco *a* che collega il g-nodo *g* al g-nodo *h*, allora i 2
percorsi *p1* e *p2* sono indistinguibili per il nodo *n*. In realtà *p1* e *p2* potrebbero aver percorso strade
diverse all'interno di *g* e per questo avere costi completamente diversi.

Di conseguenza, il nodo *n* ∉ *g* è interessato a ricevere informazioni su percorsi che toccano *g* e poi escono
da *g* attraverso l'arco *a* solo se si tratta del miglior percorso (con costo minore) che esce da *g* attraverso l'arco *a*.

Quindi, quando il nodo *v* produce il messaggio *m* e include un percorso *p*, per ogni livello *i* (da 1 a *l* - 1),
se tale percorso non è il migliore tra quelli che escono dal suo g-nodo *v<sub>i</sub>* attraverso un particolare arco
*a*, deve indicare che tale percorso va ignorato all'esterno di *v<sub>i</sub>*.

#### <a name="Percorsi_interni"></a>Percorsi interni al g-nodo

Si consideri un ETP *m* prodotto da *v* che abbandona *v<sub>i</sub>* e giunge al nodo *n*. Ogni percorso *p* la cui
destinazione finale è interna a *v<sub>i</sub>* , cioè con livello inferiore a *i*, va scartato dal nodo *n*. Questo
il nodo *n* sarebbe stato in grado di capirlo da solo. Comunque anche in questo caso, lo facciamo dichiarare
esplicitamente dal nodo *v*.

Quindi, quando il nodo *v* produce il messaggio *m* e include un percorso *p*, per ogni livello *i* (da 1 a *l* - 1),
se tale percorso ha una destinazione interna a *v<sub>i</sub>*, deve indicare che tale percorso va ignorato
all'esterno di *v<sub>i</sub>*.

### <a name="Rimozione_percorsi_da_ignorare"></a>Rimozione dei percorsi da ignorare

Sia *m* un ETP che il nodo *n* riceve dal nodo *v*. Sia *p* ∈ *m.P* un percorso pubblicizzato da *v*.

Sia *i* con *i* < *l* il livello del massimo distinto g-nodo di *v* per *n*.

Il nodo *n* esamina il messaggio *m* per vedere se *p* è un percorso che va ignorato all'esterno di *v<sub>i</sub>*,
come è stato descritto sopra. In questo caso il percorso *p* viene scartato.

<a name="GroupingRule"></a>

### <a name="Definizione_grouping_rule"></a>Definizione di grouping rule

Sia *m* un ETP che il nodo *n* riceve dal nodo *v*. Sia *p* ∈ *m.P* un percorso pubblicizzato da *v*.

Definiamo la *grouping rule* come l'elaborazione che il nodo *n* deve fare su *p* affinché tale lista, dapprima
coerente con i g-nodi a cui appartiene *v* nei vari livelli, diventi coerente con i g-nodi a cui appartiene *n*.

Sia *i* con *i* ≤ *l* il livello del minino comune g-nodo tra *n* e *v*.

Se *i* = 1, cioè se il minimo comune g-nodo tra *n* e *v* è *v<sub>1</sub>*,  cioè *n* e *v* sono nello stesso g-nodo
di livello 1, allora tutti gli  indirizzi che sono nella mappa gerarchica di *v* possono essere nella  mappa gerarchica
di *n*. Quindi *n* considera validi tutti gli hop della lista; inoltre vi aggiunge in testa l'hop composto dal massimo
distinto g-nodo di *v* per *n* - che sarà del tipo (0, *v<sub>0</sub>*) - e dall'arco tramite il quale *n* ha
ricevuto l'ETP.

Se invece *i* > 1  questo significa che il nodo *v* e il nodo *n* sono border-nodi di g-nodi diversi. Questo comporta
che i percorsi che il nodo *v* sta pubblicizzando vanno modificati rimuovendo le informazioni del percorso interno al
g-nodo *v<sub>i</sub>*.

Da ogni percorso *p* ∈ *m.P*, *n* rimuove tutti gli hop iniziali che rappresentano un g-nodo di livello inferiore ad
*i* - 1; alla lista rimarrà sicuramente qualche hop; di seguito *n* vi aggiunge in testa l'hop composto dal massimo
distinto g-nodo di *v* per *n* - che sarà del tipo (*i* - 1, *v<sub>i-1</sub>*) - e dall'arco tramite il quale *n*
ha ricevuto l'ETP.

#### <a name="Applicazione_grouping_rule"></a>Applicazione della grouping rule sulla lista di hops percorsi dall'ETP

La grouping rule come è stata descritta si applica ai percorsi *p* ∈ *m.P*. Ma si applica in modo diverso anche alla
lista di hops percorsi dal messaggio *m*.

Si consideri che un percorso verso una destinazione interna ad un g-nodo *g* non è interessante per il nodo *n* ∉ *g*.
Invece un ETP originato in *g* può essere che porti variazioni a percorsi che escono da *g*, quindi può essere nel
complesso interessante per il nodo *n* ∉ *g*.

Il nodo *n* rimuove da questa lista tutti gli hop iniziali che rappresentano un g-nodo di livello inferiore ad *i* - 1;
di seguito, anche qualora la lista risultasse vuota, vi aggiunge in testa l'hop composto dal massimo distinto g-nodo
di *v* per *n*.

<a name="AcyclicRule"></a>

### <a name="Definizione_acyclic_rule"></a>Definizione di acyclic rule

Sia *m* un ETP che il nodo *n* riceve dal nodo *v*. Sia *p* ∈ *m.P* un percorso pubblicizzato da *v*.

Definiamo la *acyclic rule* come l'elaborazione che permette al nodo *n* di stabilire se in *p* è presente
l'identificativo di uno dei suoi g-nodi, cioè se questo percorso è ciclico a qualsiasi livello della gerarchia.

L'implementazione è banale. Va effettuata su tutti i livelli. Il nodo *n* sa di aver ricevuto questo percorso dal
nodo *v*, quindi, avendo calcolato *i* il livello del minimo comune g-nodo tra *n* e *v*, potrebbe limitarsi a
verificare il livello *i* - 1, poiché in teoria il nodo *v* ha già rimosso i percorsi con cicli nei livelli superiori.
Comunque il nodo *n* non si fida di questo e verifica tutti i livelli da *i* - 1 in su. Quelli inferiori a *i* - 1 sono
stati rimossi dalla grouping rule.

Se la regola non è soddisfatta, cioè se il percorso è ciclico, il percorso *p* viene scartato.

#### <a name="Applicazione_acyclic_rule"></a>Applicazione della acyclic rule sulla lista di hops percorsi dall'ETP

La acyclic rule come è stata descritta si applica ai percorsi *p* ∈ *m.P*. Ma si applica in modo analogo anche alla
lista di hops percorsi dal messaggio *m*.

In questo caso, se la regola non è soddisfatta, cioè se il percorso è ciclico, allora l'intero ETP viene ignorato.

### <a name="Contenuto_forma_messaggio_etp"></a>Contenuto e forma di un messaggio ETP

Un messaggio ETP *m* inviato da un nodo *v* deve contenere:

*   L'indirizzo del nodo *v*, come elenco *v<sub>l-1</sub>·...·v<sub>1</sub>·v<sub>0</sub>*.
*   La lista dei g-nodi percorsi dall'ETP sotto forma di coordinate gerarchiche valide per il nodo *v*.
*   Un set di percorsi *P* che il nodo *v* intende comunicare ai suoi vicini. Per ogni percorso *p* ∈ *P*:
    *   La lista dei g-nodi del percorso *p*; questa comprende il g-nodo destinazione *d*; questa lista è
        espressa come due sequenze:
        *   p.hops: sequenza di k<sub>p</sub> g-nodi sotto forma di coordinate gerarchiche valide per il nodo *v*.
        *   p.arcs: sequenza di k<sub>p</sub> identificativi di arco, dove `arcs[0]` indica l'arco che congiunge il
            nodo v a `hops[0]`, e `arcs[j]` indica l'arco che congiunge `hops[j-1]` a `hops[j]`.
    *   Il fingerprint del g-nodo *d* come riportato dal percorso *p*.
    *   Il numero approssimato di nodi nel g-nodo *d* come riportato dal percorso *p*.
    *   Il costo totale del percorso da *v* a *d*.

#### <a name="Informazioni_sui_percorsi_da_ignorare"></a>Informazioni sui percorsi da ignorare

Quando un ETP *m* prodotto da *v* abbandona *v<sub>i</sub>* esso deve indicare quali percorsi sono da ignorare
all'esterno di *v<sub>i</sub>*.

Quindi il messaggio *m* prodotto da *v* deve contenere anche:

*   Per ogni percorso *p* ∈ *P*:
    *   Per ogni livello *i* da 1 a *l* - 1:
        *   Un booleano che dice se *p* è da ignorare per il nodo *n* ∉ *v<sub>i</sub>*.

La valorizzazione di questo booleano procede così:

*   Per ogni percorso *p* ∈ *P*:
    *   Per ogni livello *i* da 1 a *l* - 1:
        *   Se la destinazione di *p* ha livello maggiore o uguale a *i*, cioè `p.hops.last().lvl` ≥ i:
            *   Sia *j* il più piccolo valore tale che `p.hops[j].lvl` ≥ i.
            *   Il booleano vale True se e solo se *p* NON è il miglior percorso da *v* verso `p.hops[j]` tramite `p.arcs[j]`.
        *   Altrimenti:
            *   Il booleano vale True.

#### <a name="Informazioni_aggiuntive_destinazione"></a>Informazioni aggiuntive riguardo la destinazione

Si consideri il nodo *n* vicino di *v* che riceve questo messaggio *m* tramite un suo arco. Oltre al set di percorsi
*P* contenuto in *m*, il nodo *n* intrinsecamente riceve un percorso verso il massimo distinto g-nodo di *v* per *n*.
Sia *i* il livello di questo g-nodo, indichiamo questo g-nodo con *v<sub>i</sub>*. Il nodo *n* vorrà memorizzare nella
sua mappa questo percorso e dovrà quindi essere informato sulle informazioni aggiuntive di cui abbiamo parlato in
precedenza, cioè il fingerprint di *v<sub>i</sub>* e il numero di nodi all'interno di *v<sub>i</sub>*. Siccome il
nodo *v* quando trasmette l'ETP *m* non conosce il livello *i* (poiché il messaggio *m* potrebbe essere inviato in
broadcast e raggiungere diversi vicini) allora *v* dovrà aggiungere ad *m* queste informazioni per tutti i g-nodi a
cui appartiene ad ogni livello. Quindi il messaggio *m* prodotto da *v* deve contenere anche:

*   Per ogni livello *i* da 0 a *l* - 1:
    *   Il fingerprint del g-nodo *v<sub>i</sub>*.
    *   Il numero approssimato di nodi all'interno del g-nodo *v<sub>i</sub>*.

### <a name="Ingresso_gnodo_in_rete"></a>Ingresso di un g-nodo in una rete

Prima di affrontare l'argomento di rete gerarchica, avevamo detto che un nodo *n* può fare ingresso in una rete *G*
grazie ad un set di archi verso nodi vicini.

Generalizzando, un g-nodo *w* di livello *i*, isomorfo al g-nodo *w’* che si trova in una diversa rete, può fare
ingresso in blocco in una rete *G*, per l'esattezza trovando un posto in un g-nodo *g* ∈ *G* di livello *j* con
*j* > *i*, grazie ad un numero di archi che congiungono alcuni nodi di *w* ad altri nodi di *g*. Il requisito è
che la topologia di rete di *G* sia identica a quella usata da *w’*.

Considerando l'aspetto delle migrazioni, diciamo anche che in modo analogo un g-nodo *w*, isomorfo al g-nodo *w’*
che si trova nel g-nodo *h* ∈ *G* può "fare ingresso", o meglio trovare un ulteriore posto, in blocco, nel g-nodo
*g* ∈ *G* di livello pari o superiore a *h*. Sempre grazie ad un numero di archi che congiungono alcuni nodi di
*w* ad altri nodi di *g*.

Sia nel caso di ingresso in una nuova rete, sia nel caso di migrazione in un diverso g-nodo, consideriamo cosa deve
fare ogni nodo *n’* appartenente al g-nodo *w’* di livello *i* (incluso il caso in cui *w’* sia il g-nodo di livello
0 equivalente al nodo *n’*).

Con modalità che non ci interessa descrivere in questo documento, uno dei nodi di *w’* che ha un arco verso un nodo
di *g* ha ottenuto e propagato a tutti i nodi in *w’* le seguenti informazioni:

*   Il livello *i* del proprio g-nodo *w’*.
*   L'indirizzo Netsukuku di *g* di livello *j* in *G* e la sua anzianità e quella dei g-nodi superiori in *G*.
*   La posizione riservata a *w* in *g* al livello *j* - 1 e la sua anzianità.

Ogni singolo nodo *n’* in *w’* ha ora tutte le informazioni per produrre l'indirizzo Netsukuku e il
fingerprint a livello 0 per una nuova identità *n* in *w*.

Per costruire l'indirizzo Netsukuku di *n*:

*   Per i livelli maggiori o uguali a *j* - 1 usa le posizioni che gli sono state comunicate ora.
*   Per i livelli minori di *j* - 1 usa le posizioni che aveva l'indirizzo dell'identità *n’*.

Per costruire il fingerprint a livello 0 di *n*, come identificativo usa lo stesso identificativo
di *n’*. Per le sue anzianità:

*   Per i livelli maggiori o uguali a *j* - 1 usa le anzianità che gli sono state comunicate ora.
*   Per i livelli maggiori o uguali a *i* e minori di *j* - 1 usa zero (nel senso che è il primo g-nodo).
*   Per i livelli minori di *i* usa le anzianità che erano dei g-nodi a cui apparteneva l'identità *n’*.

Il sistema in cui vive l'identità *n’* crea una nuova istanza del modulo QSPN che gestirà la sua identità *n*. Si
rimanda alla trattazione delle identità per dettagli approfonditi.

Dopo la duplicazione degli archi-identità da *n’* a *n*, nel caso di ingresso in una nuova rete,
bisogna considerare che alcuni archi-identità per i quali il nodo *n’* non aveva costruito un arco-qspn
potrebbero essere tali che il nodo *n* voglia costruire un arco-qspn: questo avviene se ci sono archi fisici
con sistemi che appartenevano già alla nuova rete. Inoltre alcuni archi-identità per i quali il nodo *n’*
aveva costruito un arco-qspn potrebbero essere tali che il nodo *n* non voglia costruire un arco-qspn: questo
avviene se *n’* aveva archi-identità con nodi esterni a *w’* nella vecchia rete.

La nuova istanza del modulo QSPN riceve queste informazioni:

*   L'indirizzo Netsukuku di *n* in *G* e il suo fingerprint a livello 0.
*   Il livello *i* del g-nodo *w* che sta facendo il suo ingresso in *G*.
*   Il livello *j* del g-nodo *g* in cui *w* sta facendo ingresso.
*   Gli archi di *n* esterni a *w*.
*   Un riferimento al modulo di *n’*, gli archi di *n* interni a *w*, e altre informazioni necessarie
    affinché il modulo di *n* possa copiare dal modulo di *n’* i percorsi noti verso i g-nodi di livello
    inferiore a *i* che sono in *w’*.

La nuova istanza del modulo QSPN si considera in fase di *bootstrap* ai livelli da *i* a *j* - 1, nel senso che:

*   Conosce già tutti i percorsi di sua pertinenza verso g-nodi di livello inferiore a *i*.
*   Sa che non esistono altri g-nodi (oltre al suo) di livello tra *i* e *j* - 2 all'interno del g-nodo
    di livello *j* - 1 che gli è stato assegnato dentro *g* nella rete *G*.
*   Sa che esistono (*almeno uno*) altri g-nodi (oltre al suo) di livello *j* - 1 all'interno del g-nodo *g* della
    rete *G*. Ma non conosce ancora alcun percorso verso di essi.
*   Non conosce ancora alcun percorso verso altri eventuali g-nodi di livello *j* o superiore nella rete *G*.

Per questo non può computare il fingerprint del proprio g-nodo di livello *j* (e superiori). Questo implica che non
può costruire un ETP che sia in grado di essere trasmesso all'esterno del g-nodo *w*.

L'identità *n* copia in una lista temporanea *queued_arcs* tutti gli archi che ha verso nodi esterni a *w*.  
Perché il modulo di *n* possa uscire dalla fase di bootstrap, cioè per avere le informazioni necessarie
al nostro nuovo posto, è *necessario e sufficiente* un ETP completo prodotto da un nodo che sia
in *g* e non in *w*.  
La nuova istanza del modulo non ha ancora ricevuto ETP dagli archi e quindi non conosce gli indirizzi Netsukuku dei
suoi vicini esterni a *w*. Non può quindi stabilire (se non ottenendo un primo ETP, magari da scartare) se un vicino
esterno a *w* sia interno a *g*.

Mentre è in corso la fase di bootstrap:

*   Se *n* riceve un ETP *msg* spontaneamente inviato da un vicino *v* (non quelli richiesti da *n* ai nodi in
    *queued_arcs* di cui parliamo sotto):
    *   Se *v* è un vicino esterno a *w*:
        *   Il modulo QSPN ignora il messaggio.
    *   Altrimenti:
        *   Se *msg* non contiene percorsi verso destinazioni di livello uguale a *j* - 1:
            *   Il modulo QSPN ignora il messaggio.
        *   Altrimenti:
            *   L'identità *n* processa *msg* e aggiorna la sua mappa.
            *   Esce dalla fase di bootstrap.
            *   Produce un ETP completo e lo invia a tutti i vicini.
*   Se *n* rileva un nuovo arco, il modulo lo aggiunge alla lista dei suoi archi e in fondo alla lista *queued_arcs*.
    Non esegue altre azioni.
*   Se *n* rileva un cambio di costo di un suo arco, il modulo non esegue alcuna azione.
*   Se *n* rileva la rimozione di un suo arco, il modulo lo rimuove dalla lista dei suoi archi e dalla lista *queued_arcs*.
    Non esegue altre azioni.
*   Se *n* riceve la richiesta di produrre un ETP la rifiuta "perché in fase di bootstrap".

Un nodo *n* in *w*, se non ha alcun arco verso nodi vicini esterni a *w* ma interni a *g* (cioè se
la lista *queued_arcs* è vuota oppure se tentando di ottenere qualche ETP è costretto a scartare tutti
gli archi perché sono esterni a *g*) aspetta di ricevere dagli altri nodi in *w* un ETP
con i percorsi verso l'esterno di *w*.

Se la lista *queued_arcs* non è vuota, *n* chiede un ETP completo al vicino collegato col primo elemento della
lista *queued_arcs*. Se riceve come risposta un rifiuto "perché in fase di bootstrap",
oppure se l'indirizzo Netsukuku del vicino riportato nell'ETP risulta esterno a *g*, ignora quell'arco e può provare
con il prossimo. Altrimenti, con l'ETP che riceve (un solo ETP è già sufficiente) può uscire dalla fase di
bootstrap ai livelli da *i* a *j* - 1, aggiornare la sua mappa e produrre ETP in grado di essere trasmessi
globalmente. Quindi trasmette tali ETP agli altri nodi in *w*. In questo caso l'ingresso di *w* è riuscito e tutti i
nodi ne verranno gradualmente a conoscenza.

Un nodo *m* in *w* che non ha archi verso nodi vicini esterni a *w* ma interni a *g* (o che ha ricevuto tutti rifiuti "perché
in fase di bootstrap") aspetta per un certo tempo *X* di ricevere un ETP dagli altri nodi in *w*, come abbiamo
detto sopra, e quindi uscire dalla fase di bootstrap.

Se invece dopo un tempo massimo nessun ETP ha dato a *m* informazioni con percorsi verso l'esterno di *w*, bisogna
che il nodo *m* riconosca il g-nodo *w* come un g-nodo isolato. Il nodo *m* a questo punto non ha percorsi verso
altri g-nodi di livello *i* o superiori, esce lo stesso dalla fase di bootstrap, e se vuole generare un ETP avrà un
fingerprint a livello *l* che ha l'identificativo del g-nodo *w* come identificativo di rete.

Il tempo *X* da aspettare dipende dalla forma di *w* e dalla posizione di *m* all'interno di *w*. Si ritiene adeguato
come tempo 10 secondi oppure, se maggiore, 1000 volte il tempo del miglior percorso noto verso la peggiore delle
destinazioni note (interne a *w*).

Le operazioni descritte sopra possono essere formalizzate con questo algoritmo:

*   Mentre *queued_arcs* non è vuota **e** *bootstrap in corso*:
    *   Sia *arc* il primo elemento della lista temporanea *queued_arcs*.
    *   Chiede al vicino collegato ad *arc* un nuovo ETP completo.  
        Se il vicino rifiuta perché non ha completato il bootstrap, *n* rimuove *arc* dalla lista *queued_arcs*
        e continua il ciclo.
    *   Se riceve un ETP *msg* come risposta, *n* scopre l'indirizzo Netsukuku del vicino
        collegato a *arc*. Se tale vicino risulta esterno a *g* (livello maggiore o uguale a *j*)
        *n* rimuove *arc* dalla lista *queued_arcs*, ignora l'ETP ricevuto e continua il ciclo.
    *   Altrimenti *n* processa *msg* ed esce dalla fase di
        bootstrap. Poi produce un ETP completo e lo invia a tutti i vicini. Esce dal ciclo.
*   Se ancora *bootstrap in corso*:
    *   Sia *max_wait* = `max(10 sec, 100 * max(bestpath(dst).rtt for dst in known_destinations))`.
    *   Attende *max_wait*.
    *   Se ancora *bootstrap in corso*:
        *   Esce dal *bootstrap* ai livelli da *i* a *j* - 1, poiché il g-nodo *w* di livello *i* è una rete a sé stante.

Dopo che il nodo *n* è uscito dalla fase di bootstrap la lista *queued_arcs*
non serve più e può essere rimossa. Ora che il nodo *n* non è più in bootstrap, per ogni suo arco chiede un nuovo ETP
completo al relativo vicino. Con tale ETP aggiorna la sua mappa.

Alla fine il nodo *n* prepara un nuovo ETP completo e lo invia in broadcast a tutti i vicini.

#### <a name="Osservazione_indirizzi_ip_interni"></a>Osservazione sugli indirizzi IP interni

Durante il tempo in cui il modulo QSPN del nodo *m* si ritiene nella fase di bootstrap ai livelli da *i* a *j* - 1,
esso comunque fornisce al suo utilizzatore i percorsi noti verso destinazioni di livello minore di *i*. Del resto le
mappe nei vari nodi in *w* sono già popolate fino al livello *i* - 1. Questo è importante per il funzionamento degli
indirizzi IP interni, i quali abbiamo detto mitigano i disagi dovuti al cambio di indirizzo di grandi g-nodi (dovuti
a migrazioni o a ingressi in altre reti). Infatti l'identità *m* in *w* può essere la nuova identità *principale* del
suo sistema, ad esempio a motivo di una migrazione di g-nodo. In questo momento un processo nel sistema, usando il
network namespace default, ha a disposizione nella tabella di routing del kernel solo le rotte verso indirizzi IP interni a *w*.

Non è critica la possibilità che, mentre il nodo *m* è nella fase di bootstrap ai livelli da *i* a *j* - 1, ci siano
variazioni locali nel grafo interno a *w*. Quindi, sebbene tali variazioni avrebbero potuto essere comunicate ai nodi
in *w*, ammettiamo il fatto che mentre un nodo è nella fase di bootstrap ai livelli da *i* a *j* - 1 non trasmette ETP
nemmeno all'interno di *w*.

