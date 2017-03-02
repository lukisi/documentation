# Modulo QSPN - Percorsi disgiunti

1.  [Definizione](#Definizione)
    1.  [Perché memorizzare diversi percorsi disgiunti](#Perche_memorizzare)
        1.  [Rimozione di uno o più percorsi](#Perche_memorizzare_Rimozione)
        1.  [Congestione di un percorso](#Perche_memorizzare_Congestione)
    1.  [Considerazioni sulla mappa gerarchica](#Mappa_gerarchica)
1.  [Operazioni di scelta dei percorsi](#Operazioni_scelta_percorsi)
    1.  [Esempio](#Esempio)
1.  [Ottimizzazione](#Ottimizzazione)

L'obiettivo del presente documento è definire cosa si intende con il termine *percorsi disgiunti* e delineare l'algoritmo per la scelta di tali percorsi.

## <a name="Definizione"></a>Definizione

Ricordiamo che l'obiettivo del modulo QSPN è permettere ad ogni nodo *n* di reperire e mantenere per ogni destinazione *d* fino a *max_paths* percorsi disgiunti, che siano i più rapidi.

Si definisce *rapporto di hops comuni* tra due percorsi *p* e *q* verso la stessa destinazione, dove *p* è il percorso meno costoso dei due, il rapporto tra il numero di singoli nodi comuni nei due percorsi e il numero di singoli nodi nel percorso *p*. In questo computo non va incluso il singolo nodo destinazione. Se il rapporto è inferiore ad una certa costante *max_common_hops_ratio*, oppure se il percorso *p* è un diretto vicino e quindi il denominatore sarebbe 0, allora si dice che il percorso *q* è disgiunto dal percorso *p*.

Si consideri che i percorsi sono composti da g-nodi. Di ogni g-nodo *g* abbiamo una approssimazione del numero di nodi al suo interno *num(g)*. Se un percorso passa per un g-nodo per arrivare alla destinazione, questo non significa che percorra tutti i nodi al suo interno. Come "stima" del numero di nodi che effettivamente quel percorso prevede prendiamo *floor(1.5 x sqrt(num(g)))*.

Abbiamo detto che bisogna escludere dal computo il singolo nodo destinazione. Ma se la destinazione di un percorso *p* è un g-nodo *g* che contiene più di un singolo nodo, allora dobbiamo considerare che la vera destinazione di un generico pacchetto dati che usa il percorso *p* sarà un singolo nodo al suo interno. Possiamo stimare come numero di nodi che effettivamente quel percorso prevede all'interno di *g* la metà della formula precedente, cioè *floor(0.75 x sqrt(num(g)))*. Da questo valore, se maggiore di 0, togliamo 1, cioè il singolo nodo destinazione.

### <a name="Perche_memorizzare"></a>Perché memorizzare diversi percorsi disgiunti

Quali motivi ci sono per un nodo di memorizzare diversi percorsi verso una destinazione?

#### <a name="Perche_memorizzare_Rimozione"></a>Rimozione di uno o più percorsi

Supponiamo che il nodo *n* ha memorizzato 3 percorsi verso *d*. Assumiamo anche che il nodo *n* per inviare pacchetti verso *d* usi sempre il migliore dei 3. A che serve avere in memoria anche gli altri 2?

Siano questi i percorsi noti a *n* verso *d*:

*   *p1* composto da *n* → *a* → *b* → *c* → *d*. Sia questo il migliore.
*   *p2* composto da *n* → *x* → *y* → *z* → *d*.
*   *p3* composto da *n* → *q* → *r* → *s* → *d*. Sia questo il peggiore.

Supponiamo ora che il collegamento tra i nodi *b* e *c* si interrompa. Il nodo *b* avverte *a* che questo percorso verso *d* non c'è più. Il nodo *a* avverte il nodo *n* che questo percorso verso *d* non c'è più.

Il nodo *n* si vede rimosso il percorso *p1*, il migliore verso *d*, cioè quello che usava sempre. Avendo esso in memoria altri percorsi che non hanno subito variazioni, non ha bisogno di attendere altri eventi né di iniziare altri processi esplorativi, ma subito conosce un altro percorso *p2* da usare per inviare pacchetti verso *d*.

Ovviamente, per ragioni di memoria e di traffico, il nodo *n* non può né memorizzare né ritrasmettere tutti i percorsi possibili verso *d*, ma solo fino ad un certo numero massimo. Vediamo allora perché preferire la memorizzazione di percorsi disgiunti.

Siano questi i percorsi noti a *n* verso *d*:

*   *p1* composto da *n* → *a* → *b* → *c* → *d*. Sia questo il migliore.
*   *p2* composto da *n* → *x* → *b* → *c* → *d*.
*   *p3* composto da *n* → *q* → *r* → *s* → *d*. Sia questo il peggiore.

Supponiamo ora che il collegamento tra i nodi *b* e *c* si interrompa. Il nodo *b* avverte *a* che questo percorso verso *d* non c'è più. Il nodo *a* avverte il nodo *n* che questo percorso verso *d* non c'è più.

Inoltre, il nodo *b* avverte *x* che questo percorso verso *d* non c'è più. Il nodo *x* avverte il nodo *n* che questo percorso verso *d* non c'è più.

In rapida successione, a motivo di un unico collegamento venuto a mancare, il nodo *n* si vede rimossi 2 dei suoi 3 percorsi verso *n*. Questo evento risulta più probabile quando i percorsi tenuti in memoria non sono sufficientemente disgiunti.

#### <a name="Perche_memorizzare_Congestione"></a>Congestione di un percorso

Assumiamo che il lettore abbia una conoscenza di base del meccanismo implementato nel TCP per evitare la congestione di un percorso.

In presenza di un "collo di bottiglia" in un percorso, il meccanismo prevede che il nodo che origina la trasmissione individui in poco tempo quale sia la massima rapidità con cui può trasmettere pacchetti uno dopo l'altro. C'è da dire anche che nei moderni dispositivi di rete usati nelle dorsali di Internet il fenomeno del [bufferbloat](https://en.wikipedia.org/wiki/Bufferbloat) ostacola questo meccanismo, a discapito del buon funzionamento della rete.

Il meccanismo di *congestion avoidance* consiste essenzialmente nel segnalare in qualche modo al nodo originante il fatto che sta trasmettendo troppo in fretta. Questa segnalazione può arrivare fondamentalmente in due modi:

*   alcuni pacchetti sono stati scartati da un router perché li riceve troppo in fretta rispetto a quanto li possa ritrasmettere; quindi il nodo destinazione segnala al mittente che ha ricevuto il pacchetto "n+1" ma non il pacchetto "n".
*   un router segnala direttamente al mittente, prima che sia costretto a scartare i suoi pacchetti, che sta trasmettendo troppo in fretta.

L'esplorazione della rete svolta dal modulo QSPN mira a trovare i percorsi migliori rispetto ad una certa metrica. Di norma questa sarà la latenza; ma anche se fosse la larghezza di banda questa verrebbe misurata per ogni singolo arco in determinati momenti. Questo per dire che il percorso che è stato individuato come il migliore, qualsiasi metrica sia stata usata per la sua individuazione, in un certo momento potrebbe non essere quello che offre il miglior throughput.

Soltanto nel momento in cui il nodo *n* usa un percorso *p1* per inviare una grossa quantità di dati a *d*, allora si può rendere conto dell'efficienza che il percorso ha in quel momento.

Il nodo potrebbe provare anche gli altri percorsi a lui noti, per individuare il migliore in quel momento. Anche in questo caso è evidente che sia meglio per il nodo *n* non solo conoscere diversi percorsi, ma che siano tra loro più disgiunti possibile. Infatti, se il collo di bottiglia di un percorso risulta essere un arco che in quel momento esperimenta un fenomeno di congestione particolarmente elevata, il nodo *n* non avrà alcun vantaggio se usa un altro percorso che passa per lo stesso arco.

Per finire, una osservazione. Supponiamo che in una versione aggiornata del protocollo TCP (o un altro protocollo del livello di trasporto) il router che satura il suo link di trasmissione sia in grado di indicare al nodo mittente non solo che deve rallentare, ma anche il punto in cui il collo di bottiglia si sta verificando.

Con questa informazione, se il nodo ha memoria di altri percorsi per raggiungere la destinazione *d* e di questi percorsi conosce i vari passaggi intermedi, il nodo *n* è in grado di scegliere con maggior accuratezza quale percorso provare per le prossime connessioni.

### <a name="Mappa_gerarchica"></a>Considerazioni sulla mappa gerarchica

Abbiamo mostrato sopra che se due percorsi *p1* e *p2* hanno in comune un passaggio (ad esempio ... → *b* → *c* → ...) questo li rende poco disgiunti.

Questo è vero fintanto che si considera un percorso come sequenza di nodi, ma può non essere vero quando si passa a considerare un percorso come sequenza di gruppi di nodi. Infatti due percorsi *p1* e *p2* che passano per lo stesso gruppo di nodi *g* (specialmente se il gruppo contiene molti nodi) potrebbero essere in realtà in gran parte (o anche del tutto) disgiunti, cioè passare per nodi in gran parte diversi. Quindi, se il percorso *p1* da *n* verso *d* viene rimosso a causa della caduta di un collegamento tra due nodi interni a *g*, il percorso *p2* potrebbe rimanere valido.

Quali informazioni ha il nodo *n* relativamente ad un percorso *p* verso *d* che passa per *g*? Oltre a sapere che passa per *g* sa anche l'identificativo dell'arco attraverso il quale entra in *g* e dell'arco attraverso il quale esce da *g*.

Abbiamo già detto quale formula applichiamo al numero approssimato di nodi presenti in *g* per stimare il numero di nodi interni a *g* che saranno toccati da un percorso *p* che "tocca" *g*.

Consideriamo i due percorsi *p* e *q* verso *d*, con *p* il percorso meno costoso. Supponiamo che entrambi contengano il passaggio in *g*. Sia *numpath(g)* il numero di nodi ottenuto con la suddetta formula come stima dei nodi toccati all'interno di *g*.

Quando si calcola il rapporto di hops comuni tra *p* e *q*, nel computo del numero di singoli nodi in *p* vi sommiamo *numpath(g)*. Invece nel computo del numero di singoli nodi comuni nei due percorsi procediamo in maniera diversa a seconda di questi casi:

*   Se in entrambi i percorsi *p* e *q* si entra in *g* attraverso l'arco *a* e se ne esce attraverso l'arco *b*, allora vi sommiamo *numpath(g)*.
*   Se in entrambi i percorsi *p* e *q* si entra in *g* attraverso l'arco *a*, ma se ne esce attraverso due archi distinti *b1* e *b2*, allora vi sommiamo *ceil (numpath(g) / 2)*.
*   Se nei percorsi *p* e *q* si entra in *g* attraverso due archi distinti *a1* e *a2*, ma se ne esce attraverso lo stesso arco *b*, allora vi sommiamo *ceil (numpath(g) / 2)*.
*   Se nei percorsi *p* e *q* si entra in *g* attraverso due archi distinti *a1* e *a2* e se ne esce attraverso due archi distinti *b1* e *b2*, allora vi sommiamo 0.

## <a name="Operazioni_scelta_percorsi"></a>Operazioni di scelta dei percorsi

Nell'algoritmo occorre tenere in considerazione i requisiti che si prefigge il modulo a riguardo delle conoscenze che il nodo *n* deve mantenere indipendentemente dal valore di *max_paths* e dalle regole di disgiunzione:

*   Per ogni destinazione *d* e per ogni proprio vicino *v*, almeno 1 percorso, se esiste, verso *d* che non contiene *v* tra i suoi passaggi.
*   Per ogni destinazione *d*, per ogni livello *i* da 0 a *d.lvl*, per ogni g-nodo *p* di livello *i* diretto vicino del suo g-nodo di livello *i*, almeno 1 percorso, se esiste, per *d* che passa per *p*.
*   Per ogni destinazione *d* almeno 1 percorso per ogni diverso fingerprint di *d* che gli viene segnalato.

Ogni volta che il nodo *n* finisce di rielaborare i percorsi di uno o più messaggi ETP, si trova con un set di percorsi per alcune destinazioni. A questi, che possono essere nuovi o no, il nodo aggiunge quelli che già conosceva per le stesse destinazioni. Indichiamo con *O* il set di percorsi così ottenuto. Vediamo come il nodo *n* decida quali percorsi mantenere nella propria mappa e quali scartare.

Per prima cosa il nodo *n* fa queste operazioni:

*   Il nodo *n* prepara, per ogni livello *i* da 0 a *l* - 1, un insieme *Z<sub>i</sub>* con i g-nodi di livello *i* diretti vicini del suo g-nodo di livello *i*.

Poi il nodo *n* esamina ogni singola destinazione *d* di questo set *O*. Sia *O<sub>d</sub>* la lista dei percorsi noti al nodo *n* per raggiungere la destinazione *d*. Sia *V<sub>n</sub>* la lista dei vicini di *n*.

Per prima cosa il nodo *n* ordina per costo crescente i percorsi in *O<sub>d</sub>*.

Per ogni percorso *p* in *O<sub>d</sub>*, per ogni g-nodo *h* in *p* (esclusa la destinazione), il nodo *n* verifica di avere *h* come destinazione possibile nella sua mappa e ne reperisce il numero di nodi al suo interno. Se *h* non è presente nella sua mappa allora rimuove *p* da *O<sub>d</sub>*.

Poi *n* prosegue con questo algoritmo:

*   Il nodo *n* prepara un insieme vuoto *F<sub>d</sub>*.
*   Il nodo *n* prepara un insieme vuoto *R<sub>d</sub>*.
*   Il nodo *n* prepara un insieme vuoto *V<sub>nd</sub>*.
*   Per ogni vicino *v* ∈ *V<sub>n</sub>*:
    *   *n* calcola il massimo distinto g-nodo di *v* per *n* e lo mette in *V<sub>nd</sub>*.
*   Il nodo *n* prepara un insieme vuoto *Z1<sub>d</sub>*.
*   Per ogni livello *i* da 0 a *d.lvl*:
    *   Per ogni g-nodo *g* ∈ *Z<sub>i</sub>*:
        *   *n* mette in *Z1<sub>d</sub>* il g-nodo *g*.
*   Inizia un ciclo *c1* per ogni percorso *p1* ∈ *O<sub>d</sub>*:
    *   Se *p1*.cost = *dead*:
        *   Esci dal ciclo *c1*.
    *   *obbligatorio* = False.
    *   Se *p1*.fingerprint ∉ *F<sub>d</sub>*:
        *   *obbligatorio* = True.
        *   *p1*.fingerprint viene inserito in *F<sub>d</sub>*.
    *   Per ogni g-nodo *g* ∈ *V<sub>nd</sub>*:
        *   Se *g* ∉ p1:
            *   Rimuove *g* da *V<sub>nd</sub>*.
            *   *obbligatorio* = True.
    *   Per ogni g-nodo *g* ∈ *Z1<sub>d</sub>*:
        *   Se *g* ∈ p1:
            *   Rimuove *g* da *Z1<sub>d</sub>*.
            *   *obbligatorio* = True.
    *   Se *obbligatorio*:
        *   *p1* viene inserito in *R<sub>d</sub>*.
    *   Altrimenti-Se *R<sub>d</sub>*.size < *max_paths*:
        *   *inserire* = True.
        *   Inizia un ciclo *c2* per ogni percorso *p2* ∈ *R<sub>d</sub>*:
            *   *total_hops* = 0.
            *   *common_hops* = 0.
            *   Per ogni g-nodo *g2* (con arco di ingresso *a_in<sub>g2</sub>* e arco di uscita *a_out<sub>g2</sub>*) ∈ *p2* (esclusa la destinazione):
                *   *num_path_g2* = floor ( 1.5 * sqrt ( num ( *g2* ) ) ).
                *   *total_hops* += *num_path_g2*.
                *   Se g2 ∈ p1:
                    *   Se nel percorso p1 l'arco di ingresso in g2 è *a_in<sub>g2</sub>*:
                        *   Se nel percorso p1 l'arco di uscita da g2 è *a_out<sub>g2</sub>*:
                            *   *common_hops* += *num_path_g2*.
                        *   Altrimenti:
                            *   *common_hops* += ceil (*num_path_g2* / 2).
                    *   Altrimenti:
                        *   Se nel percorso p1 l'arco di uscita da g2 è *a_out<sub>g2</sub>*:
                            *   *common_hops* += ceil (*num_path_g2* / 2).
                        *   Altrimenti:
                            *   *common_hops* += 0.
            *   Se la destinazione *d* è un g-nodo di livello *l* > 0 **e** il nodo ha già *d* nella sua mappa:
                *   *num_path_d* = floor ( 0.75 * sqrt ( num ( *d* ) ) ).
                *   Se *num_path_d* > 0: *num_path_d* = *num_path_d* - 1.
                *   Se *num_path_d* > 0:
                    *   Sia *a_in<sub>d</sub>* l'arco di ingresso in *d* nel percorso *p2*.
                    *   *total_hops* += *num_path_d*.
                    *   Se nel percorso p1 l'arco di ingresso in *d* è *a_in<sub>d</sub>*:
                        *   *common_hops* += *num_path_d*.
                    *   Altrimenti:
                        *   *common_hops* += 0.
            *   Se *total_hops* > 0 AND *common_hops* / *total_hops* > *max_common_hops_ratio*:
                *   *inserire* = False.
                *   Esci dal ciclo *c2*.
        *   Se *inserire*:
            *   *p1* viene inserito in *R<sub>d</sub>*.
*   Il nodo *n* sostituisce l'insieme *O<sub>d</sub>* con *R<sub>d</sub>*.

### <a name="Esempio"></a>Esempio

Portiamo a semplice esempio una rete con un solo livello. Sia il nostro nodo A, l'unico vicino B, e la destinazione G. Impostiamo *max_common_hops_ratio=0.7*. Siano noti i percorsi:

*   BC    150 (AB=50, BC=50, CG=50)
*   BD    160 (AB=50, BD=55, DG=55)
*   BEF   170 (AB=50, BE=50, EF=35, FG=35)
*   BED   180 (AB=50, BE=50, ED=25, DG=55)
*   BDEF  190 (AB=50, BD=55, DE=15, EF=35, FG=35)

Prima ordino i percorsi (come già fatto sopra) in base al costo.

Il primo è preso per diritto.

Il secondo, per decidere se è sufficientemente disgiunto, lo devo confrontare solo con il primo. Il "rapporto di hops comuni" in questo caso è 1/2, quindi il secondo percorso viene preso.

Il terzo percorso va confrontato con tutti i precedenti presi, quindi il primo e il secondo. Il rapporto con il primo è 1/2, con il secondo 1/2.

Quindi il terzo è preso.

Il quarto percorso nel rapporto con il secondo ha 2/2. Quindi non viene preso.

Il quinto percorso va ora confrontato con i tre precedenti presi. Anche qui, il rapporto con il secondo è 2/2, quindi non viene preso.

## <a name="Ottimizzazione"></a>Ottimizzazione

Partendo dal parametro *max_common_hops_ratio* che viene passato al modulo QSPN, per ogni destinazione *g* nota al modulo, ogni volta che si inizia a valutare i percorsi disgiunti per raggiungere *g*, va calcolato un particolare valore *mch_ratio* da usare come massimo rapporto di hop comuni. Questo calcolo si basa su:

*   La dimensione approssimata del g-nodo *g*;
*   Il numero di diretti vicini tramite i quali il nodo ha almeno un percorso verso *g*.

Questo dovrebbe aiutare nell'avere a disposizione un valido ventaglio di percorsi. Supponiamo che un nodo *n* vuole raggiungere una destinazione *d*. I percorsi che *n* memorizza sono sempre verso un g-nodo visibile nella sua mappa. Quindi *n* ha dei percorsi verso il massimo distinto g-nodo di *d*. Sia *g* tale g-nodo. Sia *l* il livello di tale g-nodo.

Il nodo *n* non ha alcuna informazione su dove si trovi il nodo *d* all'interno di *g*. Inoltre sappiamo che tutti i percorsi che *n* memorizza verso il g-nodo *g* sono completamente interni al loro minimo comune g-nodo, chiamiamolo *w*, di livello *l* + 1. Chiamiamo *h* il g-nodo di livello *l* a cui appartiene *n*. Sappiamo che *h* ≠ *g*.

Ci saranno un certo numero di archi che costituiscono l'ultimo possibile passo dei percorsi verso *g*, che cioè partendo da un g-nodo di livello *l* diverso da *g* entrano in *g*. Possiamo dire che più è alto il numero di nodi in *g* più è probabilmente alto il numero di possibili archi che entrano in *g*. C'è da dire che, sebbene il nodo *n* sappia quale sia l'arco entrante in *g* verso il quale ha il percorso più veloce (o meglio meno costoso), il nodo *n* non può sapere quale di questi archi sia quello più prossimo (con il percorso meno costoso) al nodo *d*.

Per semplicità, supponiamo che la *gsize(l)* sia 2. Allora all'interno del g-nodo *w* abbiamo un certo numero di archi che collegano direttamente il g-nodo *h* al g-nodo *g*.

Questa immagine può aiutare a visualizzare lo scenario. Qualsiasi punto dentro l'area *h* può essere il nodo *n*. Qualsiasi punto dentro l'area *g* può essere il nodo *d*.

![grafo1](img/PercorsiDisgiunti/grafo1.png)

Siccome vogliamo che il nodo *n* mantenga un numero fisso *max_paths* di percorsi verso *g*, se il numero di archi da *h* a *g* è piccolo, conviene che il valore di *mch_ratio* sia non troppo basso, altrimenti si rischia di scartarne molti e di avere alla fine poche possibili alternative.

Man mano che il numero di archi da *h* a *g* diventa (probabilmente) grande, conviene che il valore di *mch_ratio* si abbassi tendendo a 0. Infatti diventa più probabile che il nodo *n* individui fino ad almeno *max_paths* percorsi verso *g* con un elevato grado di disgiunzione.

Inoltre tali percorsi molto disgiunti porteranno, con buona probabilità, a fare ingresso in *g* da archi fra loro geograficamente lontani, che di conseguenza avranno probabilmente percorsi verso il singolo nodo *d* anch'essi con un elevato grado di disgiunzione.

Il valore *mch_ratio*, però, non deve raggiungere lo 0 assoluto, anche se il numero di nodi in *g* è molto grande. Infatti, se il numero di nodi in *g* è elevato, e di conseguenza lo è probabilmente anche il numero di archi da *h* a *g*, allora il computo del numero di singoli nodi toccati all'interno di *g* prima di giungere al singolo nodo destinazione (di cui si tiene conto nell'algoritmo sopra delineato) aiuterà a tenere basso il rapporto di hop comuni tra due percorsi *p* e *q*, ma non lo potrà azzerare del tutto se prima di arrivare a *g* entrambi i percorsi hanno qualche singolo nodo in comune. Ad esempio, se il nodo di partenza ha un solo vicino e un solo arco, anche se il g-nodo destinazione fosse molto grande, non avrà mai due percorsi con rapporto di hop comuni uguale a zero.

Quest'ultima riflessione fa pensare che il valore *mch_ratio* vada stabilito anche in base al numero di archi (gateway) tramite i quali *n* ha almeno un percorso verso *g*. Se il numero di gateway è basso (1 o 2) conviene che il *mch_ratio* non sia troppo basso.

Cerchiamo una formula *y = f<sub>v</sub>(x)*, dove *x* è il numero di nodi in *g*, *v* è il numero di gateway tramite i quali *n* ha almeno un percorso verso *g*, il risultato *y* è il valore da usare come *mch_ratio*. Descriviamo la funzione guardando il grafo di questa funzione nel piano *x,y* considerando l'asse delle *x* con una scala logaritmica. Il valore di *y* si deve mantenere prossimo al valore *max_common_hops_ratio* fino a che *x* è minore di 10. Per valori di *x* superiori, il valore di *y* deve scendere gradualmente. Quando il valore di *x* raggiunge 1000 il risultato deve aver quasi raggiunto un valore minimo 𝜀. Per valori di *x* ancora superiori il risultato tende sempre di più a 𝜀.

Il valore minimo, 𝜀, è una frazione di *max_common_hops_ratio* che dipende da *v*. Quando *v* è 1, 𝜀 è circa la metà di *max_common_hops_ratio*, poi decresce man mano che *v* aumenta fino a tendere a 0. quando *v* è uguale a 7, cioè per un numero di vicini piuttosto elevato, 𝜀 è circa un decimo di *max_common_hops_ratio*. Poi l'aumentare di *v* diventa sempre meno incidente sul diminuire di  𝜀.

Usiamo questo algoritmo:

*   Siano dati i valori:
    *   *num* = *num(g)*.
    *   *v* = numero di gateway tramite i quali *n* ha almeno un percorso verso *g*.
*   Se *v* = 1:
    *   𝜆 = 0.45
*   Altrimenti-Se *v* = 2:
    *   𝜆 = 0.35
*   Altrimenti-Se *v* = 3:
    *   𝜆 = 0.27
*   Altrimenti-Se *v* = 4:
    *   𝜆 = 0.20
*   Altrimenti-Se *v* = 5:
    *   𝜆 = 0.15
*   Altrimenti-Se *v* = 6:
    *   𝜆 = 0.12
*   Altrimenti-Se *v* = 7:
    *   𝜆 = 0.10
*   Altrimenti:
    *   𝜆 = 0.08
*   𝜀 = *max_common_hops_ratio* * 𝜆
*   Se *num* < 10:
    *   𝛾 = 1
*   Altrimenti-Se *num* < 25:
    *   𝛾 = 0.9
*   Altrimenti-Se *num* < 75:
    *   𝛾 = 0.8
*   Altrimenti-Se *num* < 250:
    *   𝛾 = 0.6
*   Altrimenti-Se *num* < 750:
    *   𝛾 = 0.3
*   Altrimenti-Se *num* < 3000:
    *   𝛾 = 0.1
*   Altrimenti:
    *   𝛾 = 0.0001
*   *mch_ratio* = (*max_common_hops_ratio* - 𝜀) * 𝛾 + 𝜀

