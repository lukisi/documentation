# Modulo QSPN - Analisi Funzionale

1.  [Terminologia](#Terminologia)
1.  [Ruolo del modulo](#Ruolo_del_modulo)
1.  [Conoscenze obiettivo del nodo](#Conoscenze_obiettivo)
    1.  [Vicini del nodo](#Vicini_del_nodo)
    1.  [Struttura gerarchica della rete](#Struttura_gerarchica_rete)
        1.  [Partizionamento della rete](#Partizionamento_rete)
    1.  [Mappa gerarchica della rete](#Mappa_gerarchica_rete)
    1.  [Fingerprint](#Fingerprint)
    1.  [Elementi memorizzati nella mappa](#Elementi_memorizzati_mappa)
    1.  [Indirizzo Netsukuku](#Indirizzo_Netsukuku)
    1.  [Migrazioni e indirizzi IP interni](#Migrazioni_e_indirizzi_interni)
    1.  [Nodi virtuali](#Nodi_virtuali)
        1.  [Al livello 0](#Nodi_virtuali_livello_0)
        1.  [Ai livelli superiori](#Nodi_virtuali_altri_livelli)
    1.  [Rimozione dell'indirizzo di connettività di un g-nodo dopo la sua migrazione](#Rimozione_indirizzo_connettivita)
        1.  [Implementazione](#Rimozione_indirizzo_connettivita_implementazione)
1.  [Requisiti](#requisiti)
1.  [Deliverable](#Deliverable)
1.  [Classi e interfacce](#Classi_e_interfacce)

## <a name="Terminologia"></a>Terminologia

Sappiamo che il programma che usa il modulo Qspn è in esecuzione su un *sistema*. In tale sistema possono esistere
diverse *identità*, ognuna delle quali costituisce una singola entità della rete Netsukuku, detta anche *nodo del grafo*
o più brevemente *nodo*.

Il modulo Qspn è un modulo *di identità*, nel senso che quando parliamo delle sue operazioni, dei dati che
mantiene in memoria, e così via, facciamo riferimento ad una particolare *istanza* del modulo (concretamente
una istanza della classe QspnManager) che è associata ad una particolare identità e non al sistema in generale.
Per ogni identità che esiste nel sistema avremo quindi una istanza del modulo Qspn.

Inoltre, quando il modulo si interfaccia con il medesimo modulo nei sistemi vicini, lo fa sempre identificando
una particolare identità nel suo sistema e una particolare identità nel sistema vicino.

Per questo nel presente documento si farà principalmente riferimento ai *nodi* della rete. Quando si inizierà
a trattare l'argomento dei nodi virtuali questa distinzione sarà più chiara.

## <a name="Ruolo_del_modulo"></a>Ruolo del modulo

Il modulo realizza lo scambio di messaggi con i vicini del nodo al fine di esplorare la rete. Tali messaggi sono
chiamati **ETP**, acronimo di Extended Tracer Packet. In questo documento non illustriamo nel dettaglio come sono
fatti questi messaggi. Rimandiamo per questo al documento [esplorazione](EsplorazioneRete.md) ma consigliamo di
leggerlo solo dopo aver letto il presente documento. Per ora basta considerare che ogni nodo usa questi messaggi
per comunicare ai suoi vicini le informazioni riguardanti i percorsi della rete che sono a lui noti.

## <a name="Conoscenze_obiettivo"></a>Conoscenze obiettivo del nodo

L'obiettivo di ogni nodo *n* è di reperire e mantenere per ogni destinazione *dst* fino a *max_paths* percorsi
disgiunti, che siano i percorsi con costo minore.

Cosa si intende per percorsi *disgiunti* si intuisce facilmente. Per una più precisa definizione rimandiamo al
documento di dettaglio [Percorsi Disgiunti](PercorsiDisgiunti.md).

Il concetto di costo di un percorso può essere implementato con una qualsiasi metrica. Si può ad esempio usare
la latenza di un messaggio che attraversa questo percorso. Oppure la larghezza di banda, cioè la quantità di
informazioni per unità di tempo che possono attraversare questo percorso. È sufficiente che siano implementate
queste operazioni:

*   La somma del costo di due segmenti di un percorso. Questo è necessario poiché il costo di un percorso è dato
    dalla somma del costo di tutti gli archi che lo costituiscono.
*   Il confronto del costo di due percorsi verso la stessa destinazione. Questo è necessario per individuare i
    percorsi con costo minore.

Per una prima lettura di questo documento si consiglia di identificare il termine "costo" con il concetto di
latenza di un messaggio che attraversa il percorso.

Inoltre *n* vuole mantenere per ogni destinazione *dst* e per ogni proprio vicino *v*, almeno 1 percorso, se
esiste, indipendentemente dal valore di *max_paths* e dalle regole di disgiunzione, verso *dst* che non contiene
*v* tra i suoi passaggi.

Inoltre *n* vuole mantenere per ogni destinazione *dst* almeno 1 percorso per ogni diverso *fingerprint* di *dst*
che gli viene segnalato. Il concetto di *fingerprint* sarà chiarito in seguito.

Inoltre *n* vuole mantenere per ogni destinazione *dst*, per ogni livello *l* da 0 a *dst.lvl*, per ogni g-nodo
*p* di livello *l* diretto vicino del suo g-nodo di livello *l*, almeno 1 percorso, se esiste, per *dst* che passa
per *p*. I concetti qui enunciati di g-nodi e livelli verranno chiariti in seguito.

### <a name="Vicini_del_nodo"></a>Vicini del nodo

Il modulo QSPN nel nodo *n* considera *vicino di n* un nodo *v* se è a conoscenza di uno o più archi che collegano
*n* a *v*. La conoscenza degli archi a disposizione del nodo *n* viene passata al modulo come requisito. Il modulo
QSPN assume che i nodi collegati ai suoi archi appartengano alla sua stessa rete.

Per ogni arco di cui il modulo QSPN viene portato a conoscenza, esso genera un identificativo che possa essere
considerato univoco. Si adotta un numero random in uno spazio sufficientemente grande. Questo identificativo
dell'arco andrà a far parte, come vedremo in seguito, delle proprietà di un percorso, che lo rendono distinguibile
da un altro. L'identificativo dell'arco è un concetto interno al modulo QSPN, di cui l'utilizzatore non si deve occupare.

La trasmissione di messaggi ETP e la loro ricezione sono subordinate agli archi noti al modulo. Un nodo trasmette
ETP soltanto tramite gli archi che conosce. Analogamente, un nodo accetta un ETP solo se proviene da un vicino
tramite un arco che conosce. Questo fatto comporta il seguente problema.

Siano *n* e *v* due nodi appartenenti alla stessa rete che non hanno alcun arco tra di loro, cioè non sono vicini.
Supponiamo che viene a formarsi un collegamento diretto tra di essi, che quindi diventano vicini. Il nodo *n* si
avvede per primo della presenza di *v* e lo contatta per formare un arco (questo avviene al di fuori del modulo QSPN).
Il nodo *v* conferma. Di seguito entrambi avviano una prima misurazione del costo dell'arco (anche questo avviene al
di fuori del modulo QSPN). Di seguito i sistemi verificano se appartengono alla stessa rete (anche questo avviene al
di fuori del modulo QSPN). Solo in caso affermativo i sistemi informano ciascuno il proprio modulo QSPN
della presenza di un arco. Sia *n* il sistema che per primo dà questa informazione al suo modulo QSPN.
Il modulo QSPN del nodo *n* riceve la notifica di un nuovo arco e cerca di contattare il nodo *v* per richiedere un ETP.
Il nodo *v* non ha ancora dato questa informazione al suo modulo QSPN, quindi il modulo non trova l'arco nella sua lista.

Questo problema rende necessario che quando il modulo QSPN in un nodo *v* riceve una richiesta da un vicino e non trova
l'arco relativo nella sua lista, prima aspetti un certo tempo e verifichi di nuovo la presenza dell'arco. Dopo questa
attesa, che chiamiamo *tempo di rilevamento dell'arco*, se l'arco risulta ancora sconosciuto, il modulo potrà ignorare
la richiesta o rifiutarla con una eccezione. La durata di questa attesa deve essere passata al modulo QSPN dal suo
utilizzatore, in quanto il modulo QSPN non conosce i dettagli dell'iter che porta il suo utilizzatore a comunicargli
la presenza di un arco.

<a name="StrutturaGerarchica"></a>

### <a name="Struttura_gerarchica_rete"></a>Struttura gerarchica della rete

L'assegnazione degli indirizzi ai nodi della rete avviene sulla base di una struttura gerarchica imposta alla rete.
Tale gerarchia è composta da un numero fisso di livelli: *l*.

Al livello 0 ci sono i singoli nodi. Ogni nodo da solo costituisce un vertice.

Al livello 1 i singoli nodi sono raggruppati a costituire gruppi (o cluster) che chiamiamo *g-nodi*  di livello 1.
Un g-nodo di livello 1 può essere costituito anche da un solo nodo, oppure da più nodi fino ad un numero massimo
fissato. Ogni  g-nodo di livello 1 può essere considerato come un singolo vertice.

Al livello 2 i g-nodi di livello 1 sono raggruppati a costituire g-nodi  di livello 2. Un g-nodo di livello 2 può
essere costituito anche da un solo g-nodo di livello 1, oppure da più g-nodi fino ad un numero massimo fissato. Ogni
g-nodo di livello 2 può essere considerato come un singolo vertice. Si noti che i singoli nodi che fanno parte di un
g-nodo di livello 1 che fa parte a sua volta di un g-nodo di livello 2, ognuno di questi singoli nodi fa parte allo
stesso tempo anche del g-nodo di livello 2.

E così via. Nel livello più alto *l* è presente un solo gruppo che costituisce tutta la rete.

Anche il singolo nodo a volte viene chiamato (impropriamente) un g-nodo di livello 0.

Abbiamo detto che ogni g-nodo di livello *i* + 1 contiene un numero massimo fissato di g-nodi di livello *i*. Il numero
massimo di elementi di un g-nodo è detto *gsize*. Ogni livello può avere un valore *gsize* diverso. Chiamiamo
*gsize(i)*, con *i* da 0 a *l* - 1, il numero massimo di g-nodi di livello *i* in un g-nodo di livello *i* + 1.

Ogni g-nodo di livello *i* ha un identificativo che lo individua univocamente all'interno del suo g-nodo di livello
*i* + 1. Tale identificativo è un numero intero che di norma assume valori da 0 a *gsize(i)* - 1. Vedremo in seguito
che in particolari casi può assumere valori maggiori.

L'indirizzo del singolo nodo va ad essere composto mettendo in sequenza gli identificativi di tutti i g-nodi a cui
esso appartiene, a partire da quello di livello *l* - 1 fino a quello di livello 0. Si noti che ogni singolo nodo,
dal momento che conosce il proprio  indirizzo, sa di far parte di un preciso g-nodo di livello 1, di un  preciso g-nodo
di livello 2, e così via, fino al livello più alto.

Questa strutturazione gerarchica è adottata per evitare che la mappa della rete che ogni nodo tiene in memoria
diventi troppo grande.

#### <a name="Partizionamento_rete"></a>Partizionamento della rete

Come detto, ogni g-nodo di qualsiasi livello *i* può essere considerato come se fosse un unico vertice. Si forma cioè
per ogni livello *i* una sorta di partizionamento del grafo che costituisce l'intera rete.

Indichiamo con *G* il grafo che rappresenta una rete. Sia *x* un nodo in *G*. Indichiamo con la scrittura
*g<sub>i</sub>(x)* il g-nodo di livello *i* a cui appartiene *x*.

Se prendiamo tutti i g-nodi di livello *i* del grafo originale *G* e li consideriamo come singoli vertici, otteniamo
un altro grafo che rappresenta tutta la rete come composta da vertici di livello *i*. Questo grafo lo indichiamo
così: *[G]<sub>i</sub>*.

Sia *g* un g-nodo di livello *i*. Indichiamo con *𝛤<sub>i</sub>(g)* l'insieme dei g-nodi di livello *i* che sono
vicini (vertici direttamente collegati) di *g* nel grafo *[G]<sub>i</sub>*. In questo insieme sono inclusi anche
i g-nodi *di connettività*: il significato di questa espressione sarà chiarito in seguito.

### <a name="Mappa_gerarchica_rete"></a>Mappa gerarchica della rete

In ogni singolo nodo, il modulo QSPN ha il compito di costruire e tenere in memoria una mappa a topologia gerarchica
della rete.

Per ogni livello *i* della rete, da 0 a *l* - 1, un nodo *n* deve memorizzare in tale mappa tutti i g-nodi di
livello *i* appartenenti al suo stesso g-nodo di livello *i* + 1 e dei quali
*n* conosce l'esistenza, cioè conosce almeno un percorso per raggiungerli.  Per ognuno vanno memorizzati tutti i
percorsi noti e per ogni percorso alcune informazioni che elencheremo più sotto.

Ad esempio, sia il nodo *n* con indirizzo *n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*. Vale a dire che *n* ha
identificativo *n<sub>0</sub>* all'interno del suo g-nodo di livello 1, il quale ha identificativo *n<sub>1</sub>*
all'interno del suo g-nodo di livello 2, ... fino a *n<sub>l-1</sub>*. Per il livello 0 il nodo *n* dovrà memorizzare
nella mappa tutti i nodi (detti g-nodi di livello 0) che conosce che appartengono al g-nodo di livello 1
*n<sub>1</sub>*. Il nodo *n* non memorizzerà nella sua mappa *n<sub>0</sub>* perché è esso stesso. Per il livello 1
il nodo *n* dovrà memorizzare nella mappa tutti i g-nodi di livello 1 che conosce che appartengono al g-nodo di livello
2 *n<sub>2</sub>* come se fossero singoli vertici. Il nodo *n* non memorizzerà *n<sub>1</sub>* come un singolo vertice
perché di esso ha già memorizzato tutti i vertici di cui è composto. Per il livello 2 il nodo *n* dovrà memorizzare
nella mappa tutti i g-nodi di livello 2 che conosce che appartengono al g-nodo di livello 3 *n<sub>3</sub>* come se
fossero singoli vertici. Il nodo *n* non memorizzerà *n<sub>2</sub>* come un singolo vertice perché di esso ha già
memorizzato tutti i vertici di cui è composto. E così via fino a memorizzare come fossero singoli vertici anche tutti
i g-nodi di livello *l* - 1 che conosce, tranne *n<sub>l-1</sub>*.

Affinché questa mappa gerarchica sia sufficiente al nodo per raggiungere ogni singolo nodo esistente nella rete, ogni
g-nodo deve essere internamente connesso. È compito del modulo  QSPN scoprire e segnalare se un g-nodo di cui si
conosce l'esistenza (e  almeno 2 diversi percorsi) è divenuto disconnesso. Eventuali successive  azioni volte a porre
rimedio non sono di competenza del modulo.

A questo scopo ogni g-nodo ha anche un altro identificativo chiamato *fingerprint*. Vediamo come si genera un
fingerprint e come viene "assegnato" ad un g-nodo.

### <a name="Fingerprint"></a>Fingerprint

A livello 0, il fingerprint di un nodo è composto da un identificativo del nodo, univoco a livello  di rete, e da una
lista di valori che  rappresentano l'anzianità ai vari livelli dal livello 0 al livello *l* - 1. L'anzianità a livello
0 indica quanto è vecchio il nodo  rispetto agli altri nodi del suo stesso g-nodo di livello 1; a livello *i* indica
quanto è vecchio il suo g-nodo di livello *i* rispetto agli altri g-nodi di livello *i* del suo stesso g-nodo di
livello *i* + 1. L'oggetto fingerprint del nodo viene passato al modulo QSPN dal suo utilizzatore; quindi come vengano
generati o recuperati i dati in esso contenuti non è di pertinenza del modulo, e nemmeno in che modo sia implementato
il confronto fra due valori di anzianità.

Il  fingerprint di un g-nodo di livello 1 ha come identificativo l'identificativo del nodo più anziano in esso contenuto
e i suoi stessi valori di anzianità dal livello 1 in su. Questi valori dal livello 1 in su risultano gli stessi per
tutti i nodi che fanno parte di un medesimo g-nodo di livello 1; questa proprietà è assicurata con una modalità che
non è di pertinenza del modulo QSPN, come è stato implicitamente detto prima. Inoltre, il fingerprint di un g-nodo di
livello 1 ricorda anche l'anzianità del nodo in esso contenuto che gli ha dato il suo identificativo. Come vedremo
subito, quando un nodo viene a conoscenza dell'esistenza di un altro nodo di livello 0 nel suo g-nodo di livello 1,
cioè viene a conoscenza di un percorso per raggiungerlo, viene anche portato a conoscenza del fingerprint di quel nodo.
Di conseguenza ogni nodo, potendo confrontare le rispettive anzianità di tali nodi, è in grado di computare il
fingerprint del suo g-nodo di livello 1.

Il fingerprint di un g-nodo di livello *i* ha come identificativo l'identificativo del g-nodo di livello *i* - 1 più
anziano in esso contenuto e i suoi stessi  valori di anzianità dal livello *i* in su (valori che risultano  gli stessi
per tutti i g-nodi di livello *i* - 1 del g-nodo). Inoltre, il fingerprint di un g-nodo di livello *i* ricorda anche
l'anzianità del g-nodo di livello *i* - 1 in esso contenuto che gli ha dato il suo identificativo, l'anzianità del
g-nodo di livello *i* - 2 in esso contenuto che gli ha dato il suo identificativo, e così via fino al livello 0. Anche
a livello *i* abbiamo che  quando un nodo viene a conoscenza dell'esistenza di un altro g-nodo di  livello *i* - 1 nel
suo g-nodo di livello *i*, cioè viene a conoscenza di un  percorso per raggiungerlo, viene anche portato a conoscenza
del  fingerprint di quel g-nodo. Di conseguenza ogni nodo è in grado di  computare il fingerprint del suo g-nodo di
livello *i*.

Il fingerprint a livello *l* non ha valori di anzianità e nemmeno necessita di ricordare l'anzianità del g-nodo di
livello *l* - 1 in esso contenuto che gli ha dato il suo identificativo. Il fingerprint a livello *l* ricorda solo
l'identificativo del g-nodo di livello *l* - 1 più anziano in esso contenuto. Questo, per definizione, è l'identificativo della rete.

Questo  meccanismo di costruzione del fingerprint di un g-nodo a partire da  quelli dei g-nodi in esso contenuti
(sulla base della conoscenza del  nodo corrente) fa in modo che al variare della rete qualche nodo rilevi
immediatamente il verificarsi dello split di un g-nodo (o dell'intera rete). Con questo termine indichiamo che il
g-nodo non è più internamente connesso, ma si sono formate 2 o più isole.

Vediamo con un esempio come avviene questo rilevamento.

Sia *g* un g-nodo di livello 1; sia *f* il nodo più anziano in esso. Siano *v* e *w* due border-nodi appartenenti
al g-nodo *g* di livello 1; il termine border-nodo di *g* indica un nodo appartenente a *g* che ha almeno un vicino
che non appartiene a *g*. Sia *x* il vicino di *v* esterno a *g*; sia *y* il vicino di *w* esterno a *g*; supponiamo
che entrambi siano appartenenti allo stesso g-nodo *a* di livello 2 in cui si trova anche tutto il g-nodo *g*.

I nodi *v* e *w* sono entrambi a conoscenza di alcuni percorsi per raggiungere *f*. Quindi entrambi hanno calcolato
il fingerprint del g-nodo *g* ottenendo un fingerprint che ha lo stesso identificativo di *f* e ricorda l'anzianità di *f*.

Supponiamo che *g* diventi disconnesso, per esempio per via della rimozione di un arco; che si siano formate due
isole; che *v* ed *f* si trovino nella prima isola; che *w* si trovi nella seconda isola. Supponiamo inoltre che
il g-nodo *a* sia ancora internamente connesso. Quindi esiste un percorso che collega *x* ad *y* senza passare per *g*.

Quando *w* scopre di non avere più alcun percorso verso *f* lo considera morto, e ricalcola il fingerprint del
g-nodo *g* ottenendo un diverso fingerprint che ha l'identificativo di un altro nodo *h* e ricorda la sua anzianità.
Per via di questa variazione il nodo *w* trasmette un ETP al nodo *y*. Se siamo fortunati, il nodo *y* aveva memoria
anche di un percorso verso *g* che passava per *x* e questo ha ancora il vecchio fingerprint: significa quindi che
il nodo *y* ha già rilevato un potenziale<sup>1</sup> split del g-nodo *g*. Ma supponiamo che, per via del massimo
numero di percorsi disgiunti verso una destinazione, il nodo *y* non aveva memoria del percorso verso *g* che passava per *x*.

Allora l'ETP ricevuto da *y* si propagherà e raggiungerà *x*.<sup>2</sup> Ora *x* sarà a conoscenza di 2 percorsi verso
la destinazione *g* che hanno informazioni diverse riguardo il fingerprint di *g*.

Il nodo che rileva questa situazione, nel nostro esempio *x*, è un border-nodo dell'isola che contiene il nodo più
anziano. Invece noi vogliamo che sia l'altra isola (o le altre isole) ad essere avvertita. Occorre quindi che si
generi il flood di un nuovo ETP che porti questa informazione indietro al nodo *y*. Aggiungiamo quindi una regola
(all'insieme delle regole che disciplinano l'algoritmo di [esplorazione](EsplorazioneRete.md) della rete) che chiamiamo
*regola di primo rilevamento di split* : quando un nodo *n* vede un percorso *p1* verso un g-nodo *g* con un fingerprint
*fp1*, se prima *n* non conosceva *fp1* e se conosceva *g* attraverso un diverso percorso *p2* con un diverso
fingerprint *fp2*, allora *n* produce un nuovo ETP con tutti i percorsi verso *g* e lo invia a tutti i suoi vicini.

Alla fine anche il nodo *y* verrà a conoscenza di 2 (o più) percorsi verso la destinazione *g* che hanno informazioni
diverse riguardo il fingerprint di *g*.

Dopo aver atteso un certo tempo, il nodo *y* verifica se questa situazione è rimasta invariata. Se i percorsi che
riportavano distinti fingerprint hanno mantenuto gli stessi distinti fingerprint, allora *y* avrà rilevato lo split del g-nodo *g*.

Basandosi sul fatto che un fingerprint di un g-nogo di livello maggiore di 0 ha memoria dell'anzianità del nodo
che gli ha dato il suo identificativo, il nodo *y* può individuare quale dei due fingerprint sia stato assegnato
a *g* dal nodo più anziano in *g*. In altre parole, quale isola formatasi con il potenziale split sia da considerarsi
più anziana. Siccome *y* ha fra i suoi diretti vicini un border-nodo di un'isola di *g* con un fingerprint diverso
da quello dell'isola più anziana, il modulo QSPN segnalerà al suo utilizzatore questo rilevamento. Al ricevere tale
segnalazione, l'utilizzatore del modulo potrà fare in modo che l'isola scollegata venga avvertita.

Si intuisce che questo meccanismo si ripresenta in maniera analoga qualsiasi sia il livello del g-nodo che diventa
disconnesso, basta che il g-nodo di livello superiore sia ancora connesso. Se invece lo split avviene sul livello più
alto, cioè se si divide tutta la rete, quello che si ottiene è che le 2 isole diventano reti distinte con identificativi
di rete distinti. Per entrambe le situazioni, come detto in precedenza, il compito del modulo QSPN è solo quello di
permetterne il rilevamento. In particolare nel caso di split di un g-nodo, è necessario che lo rilevi almeno un nodo
diretto vicino di un border-nodo dell'isola (o delle isole) che non contiene il nodo più anziano.

Infine, una nota sui segnali emessi dal modulo QSPN. Osserviamo cosa succede quando un nodo generico
*n* (non solo *x* e *y*) scopre che due percorsi *p1* e *p2* che conosce verso un g-nodo *g* riportano due diversi
fingerprint. Supponiamo che il nodo *n* individui nel fingerprint riportato dal percorso *p1* quello dell'isola più
anziana. Il nodo *n* nel suo modulo QSPN mantiene entrambi i percorsi in memoria con i loro fingerprint. Quando deve
pubblicizzare i suoi percorsi attraverso un ETP il nodo *n* pubblicizza entrambi. Invece, quando il modulo QSPN del
nodo *n* deve esporre al suo utilizzatore i percorsi noti verso la destinazione *g*, allora solo i percorsi con il fingerprint
dell'isola più anziana vengono esposti. Vale a dire che il percorso *p1* viene esposto e il percorso *p2* no.

Se il percorso *p2* era precedentemente valido, quindi in precedenza il modulo aveva segnalato la sua presenza, adesso
il modulo deve segnalare la rimozione del percorso *p2* (sebbene in memoria rimanga ancora). In futuro, inoltre, se
*p2* tornasse ad essere valido, il modulo dovrà segnalare nuovamente la sua presenza.

* * *

<sub>Nota 1: Potenziale. Il nodo *y* per avere la definitiva certezza che il g-nodo *g* si è disconnesso in due isole,
deve attendere un certo lasso temporale. Durante questo tempo, infatti, *y* potrebbe venire informato anche dal percorso
che passa per *x* che il fingerprint di *g* è cambiato. In questo caso è possibile che il vecchio capostipite *f* sia morto
ma il g-nodo *g* sia ancora del tutto connesso. Se invece dopo l'attesa i due percorsi verso *g* (quello diretto e quello
che passa per *x*) riportano ancora gli stessi diversi fingerprint, allora si ha la certezza che il g-nodo *g* è diventato
disconnesso.</sub>

<sub>Nota 2: Per questo è importante che un nodo sempre mantenga e trasmetta per ogni destinazione *d* almeno un percorso
per ogni diverso fingerprint di *d* che gli viene segnalato attraverso gli ETP ricevuti.</sub>

### <a name="Elementi_memorizzati_mappa"></a>Elementi memorizzati nella mappa

Riassumendo, ogni g-nodo (non *virtuale*)<sup>1</sup> nella topologia gerarchica del nodo corrente *n* è una possibile destinazione
per il nodo *n*. La mappa del nodo *n*, per ogni sua possibile destinazione, mantiene queste informazioni:

*   La destinazione, espressa come livello (*lvl*) e identificativo all'interno di quel livello
    (*pos* : numero da 0 a *gsize(lvl)* - 1). Il modulo
    QSPN lo rappresenta con una istanza della classe [HCoord](#HCoord).  
    Una istanza di HCoord in generale può rappresentare un g-nodo *virtuale*, cioè può avere come *pos* un numero maggiore
    o uguale a *gsize(lvl)*, ma questi non sono mai una destinazione.
*   Tutti i percorsi che il nodo sa di poter usare per raggiungere quella destinazione. Il modulo li rappresenta con istanze
    della classe [NodePath](#Path). Per ogni percorso vanno mantenute queste informazioni:

    *   L'arco verso il gateway.
    *   Tutti i passi del percorso. Ogni passo è un g-nodo; sono inclusi eventuali g-nodi *virtuali* che esistono
        con funzione *di connettività*. Ogni g-nodo è espresso con una istanza di HCoord.
    *   Tutti gli identificativi degli archi che collegano un passo al successivo, compreso l'identificativo dell'arco
        verso il gateway.
    *   Il costo del percorso.
    *   Il fingerprint del g-nodo destinazione come riportato da questo percorso.
    *   Il numero di nodi approssimati all'interno del g-nodo destinazione come riportato da questo percorso.

    Si consideri che due percorsi di cui il nodo viene a conoscenza sono dal nodo considerati distinti se e solo se non
    riportano le stesse sequenze di passi e di archi.

* * *

<sub>Nota 1: I concetti di nodo e g-nodo *virtuale* e *di connettività* saranno chiariti a breve.</sub>

### <a name="Indirizzo_Netsukuku"></a>Indirizzo Netsukuku

L'*indirizzo Netsukuku* è l'indirizzo di una risorsa all'interno della rete, ad esempio un nodo o un g-nodo. Devono
essere noti i parametri della topologia gerarchica della rete:

*   Numero di livelli in cui la rete è suddivisa: *l*.
*   Per ogni livello *i*, numero di posizioni in quel livello: *gsize(i)*.

Dati questi parametri, un indirizzo di nodo è composto da un identificativo per ogni livello da *l* - 1 a 0. Invece,
un indirizzo di g-nodo di livello *i* è composto da un identificativo per ogni livello da *l* - 1 a *i*.

Per convenienza, diciamo che i parametri della topologia fanno parte dell'indirizzo.

Per rappresentare gli indirizzi di nodi verrà definita la classe Naddr, che il modulo QSPN non conosce del tutto, ma
solo la sua interfaccia IQspnNaddr, come verrà dettagliato sotto.

Il modulo QSPN in un nodo *n* conosce, per requisito, il suo indirizzo. Per il tramite di messaggi ETP, esso viene
a conoscenza degli indirizzi di altri g-nodi che può raggiungere tramite i suoi diretti vicini, ma solo di g-nodi che hanno
come g-nodo di livello direttamente superiore il g-nodo di cui anche il nodo *n* è membro. Cioè, solo i g-nodi di
livello *i* che appartengono allo stesso g-nodo di livello *i* + 1 a cui appartiene il nodo *n* saranno individuati
e memorizzati. Per questo per rappresentare gli indirizzi di g-nodi il modulo QSPN usa la classe HCoord (coordinate gerarchiche).

### <a name="Migrazioni_e_indirizzi_interni"></a>Migrazioni e indirizzi IP interni

Il problema dell'assegnazione degli indirizzi ai nodi non è di pertinenza del modulo QSPN. Non lo trattiamo in questo
documento, ma diciamo che di certo è un problema che si risolve con un algoritmo distribuito e dinamico. Cioè è
previsto che un nodo o un intero g-nodo (cioè tutti i singoli nodi di un g-nodo) cambi il suo indirizzo Netsukuku
nel tempo. Chiamiamo questa operazione *migrazione* di un nodo o g-nodo.

Siccome dall'indirizzo Netsukuku di un nodo dipende anche il suo indirizzo IP, il fatto che un g-nodo grande possa
migrare in blocco, comporta un cambio di indirizzo IP ad un notevole numero di nodi. Questo è un effetto collaterale
sgradevole per i nodi interessati.

Però, l'utilizzatore del modulo QSPN può fare sì che ogni nodo in un g-nodo *g* di livello *i*, oltre all'indirizzo IP
*globale* univoco rispetto a tutta la rete *G*, abbia anche un altro indirizzo IP *interno* a *g*, che sia univoco solo
internamente a *g* e che non interferisca con l'indirizzo IP *globale*.

Le informazioni che il modulo Qspn reperisce in un nodo sono sufficienti a comporre le sue regole di routing sia per gli
indirizzi IP *globali* che per gli indirizzi IP *interni* a qualsiasi livello *i*.

Se il nodo 𝛼 vuole stabilire una connessione con il nodo 𝛽 ed entrambi fanno parte del g-nodo *g*, allora il nodo 𝛼 userà
gli indirizzi IP *interni* a *g* di 𝛼 e 𝛽. Qualora il g-nodo *g* o un suo g-nodo superiore per una migrazione cambiasse il
proprio indirizzo Netsukuku, tutti i nodi al suo interno cambierebbero il proprio indirizzo IP *globale*. Ma gli indirizzi
IP *interni* a *g* di 𝛼 e 𝛽 non cambierebbero. Nemmeno le rotte cambierebbero, in quanto il
percorso è sicuramente interno a *g*. Cioè la connessione aperta rimarrebbe valida e funzionante.

Questo accorgimento mitiga molto il disagio dovuto al cambio di indirizzo di un intero g-nodo di grandi dimensioni.

Un esempio dell'uso di indirizzi IP *interni* è illustrato in questo [documento](RoutingIndirizziInterni.md) (insieme ad
alcune considerazioni sul routing che saranno chiare dopo aver esaminato il prossimo paragrafo sui *nodi virtuali*).

### <a name="Nodi_virtuali"></a>Nodi virtuali

Abbiamo detto in precedenza che ogni g-nodo di livello *i* ha un identificativo che lo individua univocamente all'interno
del suo g-nodo di livello *i* + 1 e che tale identificativo è un numero intero da 0 a *gsize(i)* - 1.

In realtà possono esserci casi in cui si vuole temporaneamente includere in un g-nodo di livello *i* + 1 un numero di g-nodi
di livello *i* superiore a *gsize(i)*. In questi casi si vuole dare ad uno specifico g-nodo di livello *i* un
identificativo che è un numero maggiore di *gsize(i)* - 1.

Si noti che il g-nodo, o meglio i nodi al suo interno, comporranno ciascuno il proprio indirizzo Netsukuku mettendo in
sequenza gli identificativi dei vari g-nodi a cui appartengono. Ma se la topologia gerarchica della rete (cioè il numero
di livelli e le *gsize*) è stata costruita sulla base del numero di bit a disposizione per gli indirizzi univoci nella
rete TCP/IP, come di norma succede, questi non potranno avere un corrispettivo indirizzo IP univoco nella rete.

#### <a name="Nodi_virtuali_livello_0"></a>Al livello 0

Spieghiamo i motivi che ci spingono ad ammettere questa situazione e le relative conseguenze assumendo *i* uguale a 0,
per semplicità, ma vedremo poi che i ragionamenti si possono estendere ai vari livelli.

Sia un nodo *n* border-nodo di un g-nodo *g* di livello 1 verso un altro g-nodo *h*. Supponiamo che *n* voglia migrare
da *g* ad *h*. Sia *j* il più alto livello tale che *g* ed *h* non appartengono allo stesso g-nodo di livello *j*,
con 1 ≤ *j* < *l*. Sia *U(g)* il g-nodo di livello *j* a cui appartiene *g*. Sia *U(h)* il g-nodo di livello *j* a cui
appartiene *h*. Il g-nodo *g*, o uno dei suoi g-nodi superiori fino a *U(g)* compreso, può risultare disconnesso (split)
a causa della migrazione di *n*. Per evitare questo effetto collaterale, quando *n* migra dal g-nodo *g* al g-nodo *h*,
oltre ad assumere un identificativo in *h*, mantiene temporaneamente un identificativo *virtuale* in *g*. Un
identificativo *virtuale* all'interno di un g-nodo di livello *i* + 1 è un numero maggiore di *gsize(i)* - 1.

Questa migrazione non è realizzata dal modulo QSPN, ma dal suo utilizzatore attraverso altri meccanismi. Sia la scelta
del g-nodo *h* in cui entrare, sia la scelta dell'identificativo in *h*, sia la scelta dell'identificativo *virtuale*
in *g*, sia di conseguenza la composizione dei due indirizzi Netsukuku uno in *g* e uno in *h*, sono tutte operazioni
di competenza dell'utilizzatore del modulo QSPN. Al modulo QSPN tutte queste informazioni vengono date dall'esterno.

Invece di continuare a parlare di un nodo *n* che ha assunto un identificativo in *h* e inoltre ha mantenuto
un identificativo *virtuale* in *g*, facciamo una astrazione e consideriamo che nello stesso *sistema* in cui
esisteva il nodo *n* ora esiste anche il nodo *n'*. Il nodo *n'* ha l'indirizzo con un identificativo *reale* in *h* mentre
il nodo *n* ha l'indirizzo con un identificativo *virtuale* in *g*.

Avendo assunto tale identificativo *virtuale*, il nodo *n* ha reso libero il suo precedente identificativo *reale* in
*g*: probabilmente era proprio questo l'obiettivo della sua migrazione. Nello stesso tempo, la persistenza del nodo *n*
all'interno di *g* con il nuovo identificativo *virtuale*, permette a *g* e ai
suoi g-nodi superiori fino a *U(g)* di non violare il vincolo di connettività interna. Infatti il nodo *n*
continua a partecipare alle trasmissioni di ETP all'interno di *U(g)* ed è in grado di inoltrare correttamente
pacchetti IP verso destinazioni interne a *U(g)*.

Il nodo *n* ha ora un indirizzo con la componente al livello 0 *virtuale*. Usiamo questo aggettivo
anche per l'indirizzo, diciamo che ha un indirizzo *virtuale* al livello 0.

Si noti che il modulo QSPN al suo interno non ha alcuna difficoltà a gestire tuple *n<sub>0</sub>·...·n<sub>l-1</sub>*
i cui membri non rispettano il limite *gsize(i)*.

Riguardo le mansioni svolte dal modulo QSPN, il nodo *n* adotta un comportamento lievemente differente
rispetto a quello che adotta il nodo *n'*. La differenza sta nel fatto che *n*
non mantiene nessun arco con nodi esterni al g-nodo *U(g)*. Questo comportamento è a tutti gli effetti
di pertinenza del modulo QSPN. Il modulo QSPN conosceva dall'inizio tutti gli archi del nodo *n* in *g* e sarà lui a scegliere
in seguito (come vedremo a breve) quali rimuovere.

Diciamo che il nodo *n* è
*di supporto alla connettività interna* ai livelli da 1 a *j*, nel senso che supporta la connettività interna dei
suoi g-nodi di livello tra 1 e *j*. Per brevità diciamo che è un nodo *di connettività* ai livelli da 1 a *j*.

I nodi *n* e *n'* che convivono nello stesso sistema sono a tutti gli effetti due identità distinte. Per quanto
riguarda il modulo QSPN, essi hanno due distinti indirizzi, due distinti fingerprint
a livello 0, due distinti set di archi, due distinte mappe di percorsi e così via.

Una differenza tra la nuova identità *n'* e quella vecchia *n* è che l'identità *n* in *g* è
destinata a sparire nel momento in cui la connettività interna di *g* e dei suoi g-nodi superiori fino a *U(g)*,
cioè ai livelli da 1 a *j*, risulta garantita da altri collegamenti. Invece l'identità *n'* in *h* è destinata a
rimanere. L'identità vecchia, temporanea, è detta *di connettività*. L'identità
nuova, che rimane, è detta *principale*.

Alcune considerazioni sulle identità *principale* e *di connettività*.

*   In un sistema in ogni momento esiste una e una sola identità *principale*. In un sistema possono esistere,
    in un particolare momento, 0 o 1 o più identità *di connettività* ai livelli da 1 a *x*.
*   Un'identità *di connettività* si crea solo a causa di una migrazione. Quando migra un g-nodo *g* di livello
    *i*, le componenti del suo indirizzo Netsukuku sono sicuramente tutte *reali*. Quindi l'indirizzo di ogni singolo
    nodo che appartiene al g-nodo *g* ha la componente a livello *i* e quelle superiori che sono sicuramente *reali*,
    ma le altre componenti ai livelli inferiori potevano anche essere già *virtuali*. Cioè, un singolo nodo che
    appartiene a *g* poteva essere l'identità *principale* del suo sistema, oppure poteva già essere una
    identità *di connettività* del suo sistema. Sotto verrà dettagliato come si
    trasforma l'indirizzo del singolo nodo che appartiene al g-nodo che migra.
*   Un'identità *di connettività* ai livelli da 1 a *x* ha sempre un indirizzo Netsukuku *virtuale* poiché la sua
    componente a livello 0 è *virtuale*. Più in generale, un'identità *di connettività* ai livelli da *i* a *j* ha
    un indirizzo Netsukuku *virtuale* con una o più componenti *virtuali*. Invece un'identità *principale* ha
    sempre un indirizzo *reale*.

Considerando quanto esposto nella trattazione del
modulo [Identities](../ModuloIdentities/AnalisiFunzionale.md), diciamo che:

*   Indichiamo con *id<sub>0</sub>* l'identità nel nostro sistema che rappresenta il nodo *n* in *g*.
*   Si aggiunge nel nostro sistema l'identità *id<sub>1</sub>* che rappresenta il nodo *n'* in *h*. Essa prende le caratteristiche che erano
    prima di *id<sub>0</sub>*: cioè, se *id<sub>0</sub>* era l'identità *principale* ora *id<sub>1</sub>* è
    l'identità *principale*, se *id<sub>0</sub>* era *di connettività* ai livelli da *x* a *y* ora *id<sub>1</sub>*
    è *di connettività* ai livelli da *x* a *y*.
*   L'identità *id<sub>0</sub>* diventa una identità *di connettività* ai livelli da 1 a *j*.
*   Ora all'identità *id<sub>1</sub>* si associano nuove istanze delle classi dei moduli *di identità*. Nell'esame
    del modulo QSPN diciamo che a *id<sub>1</sub>* si associa una nuova istanza di QspnManager.

Il fingerprint di *id<sub>1</sub>* è una nuova istanza. Nel costruirlo si usa come identificativo a livello 0 lo
stesso che era l'identificativo a livello 0 di *id<sub>0</sub>*. Sono identici anche i valori di anzianità di livello
inferiore al livello del g-nodo che migra (in questo caso, siccome prendiamo in esame un singolo nodo, non esistono
livelli minori di 0) e quelli di livello superiore a *j*. Sono diversi invece i valori di anzianità dal livello 0
(cioè il livello del g-nodo che migra) fino al livello *j*, perché sono quelli relativi al g-nodo *h*.

Il fingerprint di *id<sub>0</sub>* subisce una variazione. Il valore di anzianità del livello del g-nodo che migra
(nel nostro esempio il livello 0) diventa *nullo*: cioè tale che confrontato con quello di qualunque altro g-nodo
risulta più giovane. Il suo identificativo a livello 0 resta lo stesso, come restano identici tutti gli altri
valori di anzianità. Questo significa che un nodo (o g-nodo) *di connettività* non darà mai il suo identificativo
come identificativo ad un suo g-nodo di livello superiore.  
Del resto dobbiamo ricordare che quando un nodo calcola il fingerprint del suo g-nodo di livello *i* lo fa tramite
una operazione sui fingerprint dei g-nodi di livello *i* - 1 in esso contenuti. Questi sono i g-nodi che il
nodo è venuto a conoscere come possibili destinazioni tramite la ricezione di ETP. Quindi per forza di cose questi
sono tutti g-nodi che non sono *di connettività*.  
In un certo senso potremmo dire che il concetto di anzianità perde di significato per un nodo (o g-nodo) *di connettività*.

Un altro concetto che perde di significato per un g-nodo *di connettività* è il numero approssimato di singoli nodi
al suo interno.  
Possiamo dire che nell'identità *id<sub>0</sub>* ora il numero approssimato di singoli nodi all'interno
del g-nodo di livello 0 è 0 anziché 1. Generalizzando, il numero approssimato di singoli nodi all'interno
di un g-nodo *di connettività* di qualsiasi livello è 0. Ma anche in questo caso si tratta di una specificazione superflua,
poiché quando un nodo calcola il numero approssimato di singoli nodi all'interno del suo g-nodo di livello *i* si sommano
i valori riportati dai g-nodi di livello *i* - 1 in esso contenuti, che sono tutti g-nodi che non sono *di connettività*.

Il set di archi di *id<sub>0</sub>* resta invariato, ma ogni arco in esso subisce delle variazioni. Il set di archi
di *id<sub>1</sub>* è un nuovo set che viene popolato.

Le modifiche subite dagli archi di *id<sub>0</sub>* non coinvolgono in alcun modo il modulo QSPN. Il suo utilizzatore,
per il tramite dei servizi del modulo Identities, per ogni *arco-identità* di *id<sub>0</sub>* crea un
*arco-identità* di *id<sub>1</sub>* e apporta delle variazioni al primo. Si tratta di una operazione che comprende:

*   Realizzazione di un distinto network stack dotato di distinte pseudo-interfacce di rete.
*   Concertazione coi diretti vicini, i quali possono essere anch'essi interessati dalla migrazione oppure solo vedersi
    aggiungere un nuovo *arco-identità*.
*   Gli *archi-identità* di *id<sub>0</sub>* diventano ora supportati dalla pseudo-interfaccia appena creata
    sul nuovo network stack.  
    Gli oggetti *arco* che sono stati comunicati al modulo QSPN per la sua identità *id<sub>0</sub>* rifletteranno
    al loro interno questa variazione, ma tale informazione non influenza le operazioni che sono esposte dall'interfaccia
    IQspnArc. Cioè, non interessa affatto il modulo Qspn.  
    Inoltre non cambia il loro identificativo di arco. Avevamo detto che l'identificativo dell'arco è un concetto
    interno al modulo QSPN, di cui l'utilizzatore non si deve occupare.
*   Sugli *archi-identità* nuovi di *id<sub>1</sub>* vengono costruiti ora gli oggetti *arco* da comunicare al
    modulo QSPN per la sua identità *id<sub>1</sub>*. Una volta passati al modulo, essi otterranno un nuovo
    identificativo di arco.

La mappa di *id<sub>0</sub>* resta invariata. La mappa di *id<sub>1</sub>* è una nuova istanza.

La mappa di *id<sub>0</sub>* è quella che era stata popolata con gli ETP ricevuti in passato da *n* in *g*. Sebbene il nodo *n*
abbia cambiato il suo identificativo in *g*, tutti i percorsi che conosceva verso le destinazioni di tutti i livelli
restano validi. Anche i percorsi che ha reso pubblici ai suoi vicini non hanno subito alcuna variazione, quindi non ha
bisogno di inviare ETP correttivi per essi. Ci sono due informazioni soltanto che deve propagare:

*   Che non è più possibile raggiungere tramite lui il suo vecchio identificativo *reale* in *g*.
*   Che è possibile raggiungere tramite lui il suo nuovo identificativo *virtuale* in *g*.

È una informazione che interessa soltanto i nodi che appartengono a *g*. È sufficiente un unico ETP vuoto,
perché da esso il nodo che lo riceve si accorge che il nodo che lo ha inviato ha modificato il suo indirizzo
Netsukuku. Riconoscendo l'arco-identità su cui l'ETP è stato ricevuto, il vicino ha memoria del vecchio indirizzo
Netsukuku e vede quello nuovo.

La mappa di *id<sub>1</sub>* viene inizializzata vuota, in attesa che il modulo QSPN processi nuovi ETP ricevuti da *n'*.
Le attività dell'istanza di QspnManager associata all'identità *id<sub>1</sub>* (cioè al nodo *n'*) e della
relativa mappa procedono come per qualsiasi nodo che entra in una rete esistente. Si vedrà nel
documento di dettaglio [esplorazione](EsplorazioneRete.md) che questo significa iniziare la fase di bootstrap e
richiedere ETP ai propri vicini.

Quando l'istanza di QspnManager dell'identità *id<sub>1</sub>* ha completato la fase di bootstrap e ha
trasmesso i primi ETP, lo segnala al suo utilizzatore. In questo momento esso potrà richiedere all'istanza
di QspnManager dell'identità *id<sub>0</sub>* di rimuovere tutti gli archi con vicini che non appartengono al g-nodo
*U(g)*. Infatti ora quei vicini che si vedranno mancare un arco con *n* (il loro vecchio gateway verso *U(g)*) avranno scoperto
di poter raggiungere *U(g)* tramite *n'*.

Per quanto riguarda la produzione e la processazione di messaggi ETP, l'istanza di QspnManager dell'identità
*principale* e tutte le altre istanze di QspnManager delle identità *di connettività*
si comportano allo stesso modo. Non fa alcuna differenza a questo proposito che il proprio indirizzo sia *virtuale* o *reale*.

Per quanto riguarda i percorsi che ogni istanza di QspnManager acquisisce, essi sono esposti all'utilizzatore
del modulo QSPN. L'utilizzatore del modulo QSPN deve saper trattare queste informazioni per istruire in modo appropriato
le tabelle di routing del sistema. Il modo più semplice di farlo è quello di realizzare un intero network stack
indipendente per ogni identità assunta nel sistema. In un sistema Linux questo si può fare con
i [network namespace](http://man7.org/linux/man-pages/man8/ip-netns.8.html).

Ovviamente l'utilizzatore dovrà far attenzione a non utilizzare le destinazioni che hanno una componente *virtuale*
nel popolamento delle tabelle di routing del kernel.

#### <a name="Nodi_virtuali_altri_livelli"></a>Ai livelli superiori

Come abbiamo detto sopra, questi concetti valgono anche per un livello *i* maggiore di 0.

Sia un g-nodo *n̄* di livello *i* border-nodo di un g-nodo *ḡ* di livello *i* + 1. Supponiamo che *n̄* voglia migrare
da *ḡ* ad un diverso g-nodo *h̄*, di livello *i* + 1 o maggiore. Sia *j* il più alto livello tale che *ḡ* ed *h̄* non
appartengono allo stesso g-nodo di livello *j*, con *i* < *j* < *l*. Sia *U(ḡ)* il g-nodo di livello *j* a cui
appartiene *ḡ*. Sia *U(h̄)* il g-nodo di livello *j* a cui appartiene *h̄*.

Riguardo la migrazione di *n̄*, diciamo che essa può essere considerata come una operazione atomica, sebbene avvenga
per forza di cose in modo graduale. Si considera atomica nel senso che quando un singolo nodo al suo interno
concretizza la sua migrazione si comporta come se sapesse per certo che tutti gli altri singoli nodi lo stanno
facendo nello stesso istante. Possiamo farlo perché all'interno di *n̄* il grafo dei nodi e g-nodi che lo costituiscono
non cambia.

Consideriamo ogni singolo nodo *n* all'interno di *n̄*. Vediamo come da questo verrà generato un nuovo nodo *n'*
che sarà parte di un nuovo g-nodo *n̄'* isomorfo di *n̄*.

Il sistema in cui vive l'identità *id<sub>0</sub>* che rappresenta il nodo *n* (in *n̄*, in *ḡ*), partendo da questa produce una nuova
identità *id<sub>1</sub>* che rappresenta il nodo *n'* (in *n̄'*, in *h̄*). La vecchia identità *id<sub>0</sub>* detiene ora un
indirizzo *virtuale* al livello *i*.

Consideriamo che l'indirizzo precedente di *id<sub>0</sub>* poteva essere anche *virtuale* ai livelli inferiori, ma sicuramente
aveva al livello *i* e ai livelli superiori tutti identificativi *reali*. Da questo si deduce che un indirizzo *virtuale* può
avere uno o più componenti (identificativi) *virtuali*.

L'identità *id<sub>0</sub>*, inoltre, poteva essere precedentemente *principale* o *di connettività* ai livelli da
*k1* a *k2*. Comunque l'identità *id<sub>0</sub>* da adesso sarà *di connettività* ai livelli da *i* + 1 a *j*.

Ad eccezione della componente al livello *i* tutte le altre componenti restano invariate.

La nuova identità *id<sub>1</sub>*, che rappresenta il nodo *n'*, ha ora un indirizzo in *h̄*. Le componenti ai livelli superiori
sono ovviamente quelle di *h̄*. La componente *reale* al livello subito inferiore a quello di *h̄* (che può essere *i*
o maggiore) è quella riservata dal g-nodo *h̄* per l'ingresso di *n̄'*. Le componenti ai livelli inferiori sono le stesse
che componevano l'indirizzo di *id<sub>0</sub>* in *ḡ*.

Inoltre, l'identità *id<sub>0</sub>* poteva essere precedentemente *principale* o *di connettività* ai livelli da
*k1* a *k2*; se era *principale* allora l'identità *id<sub>1</sub>* sarà *principale*, se era *di connettività*
allora l'identità *id<sub>1</sub>* sarà *di connettività* ai livelli da *k1* a *k2*. Ma se *k2* è maggiore di *i*
allora *id<sub>1</sub>* può considerarsi *di connettività* ai livelli da *k1* a *i*.

Per tutti gli aspetti inerenti le istanze di QspnManager associate alle identità *id<sub>0</sub>* e *id<sub>1</sub>*
valgono tutte le osservazioni viste prima.

Riguardo la mappa dei percorsi c'è da dire questo: tutti i percorsi che erano noti al nodo *n* e che hanno per
destinazione un livello inferiore a *i*, cioè un g-nodo interno a *n̄*, saranno validi anche per il nodo *n'* in *n̄'*.
Per questo l'utilizzatore del modulo QSPN quando istanzia il nuovo QspnManager di *id<sub>1</sub>* lo istruisce
di copiare i percorsi con destinazioni di livello inferiore a *i* dalla mappa del QspnManager di *id<sub>0</sub>*.  
In seguito il QspnManager di *id<sub>1</sub>* inizia la fase di bootstrap a livello *i*.  
Solo se *n'* ha un arco con (almeno) un vicino *v* che è esterno a *n̄'* ma interno a *h̄* il modulo richiede un ETP
a questo vicino. Se riceve nuove informazioni esce dalla fase di bootstrap a livello *i* e le propaga all'interno di *n̄'*.  
I nodi interni attendono un certo tempo (dipendente dalle conoscenze che il QspnManager di *n'* ha dei percorsi interni a
*n̄'*) entro il quale dovrebbero ricevere un ETP originatosi all'esterno di *n̄'*, cioè contenente destinazioni di
livello maggiore di *i*, e quindi uscire dalla fase di bootstrap a livello *i*.

Il g-nodo *n̄* in *ḡ* dovrà sparire quando la connettività interna di *ḡ* e dei suoi g-nodi superiori fino a *U(ḡ)*,
cioè ai livelli da *i* + 1 a *j*, risulterà garantita da altri collegamenti. Sarebbe bene che un solo nodo, chiamiamolo
*n<sub>0</sub>* in *n̄*, facesse in modo periodico la richiesta al modulo QSPN di controllare questo evento e in caso
di esito positivo lo comunicasse a tutti i nodi in *n̄*. Possiamo assegnare questo compito al nodo che, stando semplicemente
alle conoscenze della sua mappa ai livelli da *i* - 1 in giù, ha l'indirizzo più prossimo allo 0.

Si osservi che la migrazione di un g-nodo di livello alto coinvolge un numero elevato di nodi, ma allo stesso tempo
permette il mantenimento delle informazioni già reperite per un elevato numero di destinazioni (quelle interne). Queste
informazioni vengono messe a frutto grazie all'utilizzo di indirizzi IP *interni*, come spiegato sopra.

Un esempio dell'uso di indirizzi *virtuali* e *di connettività* è illustrato in questo [allegato](UsoIndirizziVirtuali/Step1.md).

### <a name="Rimozione_indirizzo_connettivita"></a>Rimozione dell'indirizzo di connettività di un g-nodo dopo la sua migrazione

Abbiamo visto come la migrazione di un g-nodo *g* di livello *i* (con 0 ≤ *i* < *l* - 1) porta alla creazione di
una identità *g1* che gestisce un indirizzo *di connettività* ai livelli da *i* + 1 a *j*, con *i* < *j* < *l*.

Sia *n* un singolo nodo appartenente a *g*. Quindi *n* ha una identità *n1* in *g1*. Il nodo *n1* vuole valutare se
sia possibile rimuovere l'indirizzo *di connettività* ai livelli da *i* + 1 a *j* che *g<sub>i</sub>(n1)* detiene
in *g<sub>i + 1</sub>(n1)*. Quindi *n1* vuole stabilire se la rimozione di *g<sub>i</sub>(n1)* **provoca** o **non provoca**
lo split di uno dei g-nodi da *g<sub>i + 1</sub>(n1)* a *g<sub>j</sub>(n1)*. Non servirà mai verificare in
*g<sub>l</sub>(n1)*, vale a dire sapere se provoca uno split di tutta la rete, perché la rimozione è fatta per
migrare ad un altro g-nodo pur rimanendo nella stessa rete.

Questa domanda può essere posta al QspnManager gestito dall'identità *n1*.

Mentre il modulo QSPN valuta la risposta con le modalità che illustreremo sotto, il suo utilizzatore deve assicurarsi
che un altro g-nodo dentro (cioè di livello direttamente inferiore) i g-nodi interessati non faccia le stesse
considerazioni. Cioè, prima di fare la richiesta al modulo QSPN, il suo utilizzatore deve acquisire un *lock* su
tutti i g-nodi a cui ora appartiene di livello da *i* + 1 a *j*. Come questo avviene non è di competenza del modulo QSPN.

Dopo che il modulo QSPN ha completato la valutazione, se questa ammette la rimozione del g-nodo, il nodo *n* deve
richiedere ai vari g-nodi interessati di attendere alcuni istanti e poi rimuovere il lock. Intanto, il nodo *n*
provvede a rimuovere la sua identità *n1* e con essa il relativo QspnManager. L'attesa serve a far sì che, quando
il lock sarà rimosso, le trasmissioni di ETP avranno portato tutti i nodi nei g-nodi interessati a rimuovere i
percorsi che passavano per *g<sub>i</sub>(n1)*.

Se invece la valutazione non ammette la rimozione del g-nodo, allora *n* può richiedere ai vari g-nodi interessati
di rimuovere immediatamente il lock.

Il lock suddetto deve comunque avere un suo tempo di vita massimo, di modo che se il nodo che ha impostato il lock
non ottempera alla sua rimozione, comunque il g-nodo non resta bloccato per sempre.

<a name="ImplementazioneVerificaRimozione"></a>

#### <a name="Rimozione_indirizzo_connettivita_implementazione"></a>Implementazione

Se il g-nodo *g<sub>i</sub>(n1)* non ha alcun vicino in *g<sub>i + 1</sub>(n1)*, cioè *g<sub>i + 1</sub>(n1)* contiene
solo *g<sub>i</sub>(n1)*, allora la domanda da porre è direttamente se la rimozione di *g<sub>i + 1</sub>(n1)*
**provoca** o **non provoca** lo split di uno dei g-nodi da *g<sub>i + 2</sub>(n1)* a *g<sub>j</sub>(n1)*.

Bisogna analizzare solo i casi in cui il g-nodo *g<sub>i</sub>(n1)* ha uno o più vicini in *g<sub>i + 1</sub>(n1)*.

La rimozione di *g<sub>i</sub>(n1)* **non provoca** lo split di uno dei g-nodi da *g<sub>i + 1</sub>(n1)* a *g<sub>j</sub>(n1)* se:

*   Per ogni g-nodo *x* di livello da *i* a *j* - 1 che vedo nella mappa di *n1*,
    cioè *x* ∈ *g<sub>i + 1</sub>(n1)* ∪ ... ∪ *g<sub>j</sub>(n1)*:
    *   Per ogni g-nodo *y* di livello *i* vicino di *g<sub>i</sub>(n1)* e che vedo nella mappa di *n1*,
        cioè *y* ∈ *𝛤<sub>i</sub>(g<sub>i</sub>(n1))*, *y* ∈ *g<sub>i + 1</sub>(n1)*:
        *   Se *x* ≠ *y*:
            *   Esiste un percorso nella mappa di *n1* che ha per destinazione *x* e passa per *y*.

Sicuramente, se *y* ha nella sua mappa dei percorsi per *x* che non passano per *g<sub>i</sub>(n1)*, allora li
ha comunicati a *g<sub>i</sub>(n1)*. Però *n1* potrebbe non averli inclusi nella sua mappa perché aveva raggiunto
il limite *max_paths* o perché non erano sufficientemente disgiunti da altri.

Però abbiamo detto all'inizio che ogni nodo si pone anche l'obiettivo di mantenere per ogni destinazione *d*, per
ogni livello *i* da 0 a *d.lvl*, per ogni g-nodo *p* di livello *i* diretto vicino del mio g-nodo di livello *i*,
almeno 1 percorso, se esiste, per *d* che passa per *p*.

Grazie a questo vincolo, se la condizione detta sopra non è soddisfatta, questo è sufficiente ad affermare che la
rimozione di *g<sub>i</sub>(n1)* **provoca** lo split di uno dei g-nodi da *g<sub>i + 1</sub>(n1)* a *g<sub>j</sub>(n1)*.

## <a name="requisiti"></a>Requisiti

In generale, indipendentemente dalle identità assunte dal nodo, il modulo ha bisogno di questi requisiti:

*   Numero massimo di percorsi per destinazione da memorizzare.
*   Massimo rapporto di passi comuni nella verifica di disgiunzione (vedi documento [percorsi disgiunti](PercorsiDisgiunti.md)).
*   Tempo di rilevamento degli archi.
*   Oggetto per calcolare il lasso temporale di tolleranza prima di segnalare il rilevamento di split di un g-nodo.

In particolare, ogni istanza del modulo creata per gestire una precisa identità del nodo, ha bisogno di questi requisiti:

*   Indirizzo Netsukuku proprio di questa identità. Istanza di IQspnMyNaddr.
*   Archi che esistono tra questa identità del nodo e i suoi vicini.  
    Durante le sue operazioni, il modulo viene informato alla costituzione di un nuovo arco; alla rimozione di un
    arco; al cambio di costo di un arco.
*   Il fingerprint a livello 0 (come singolo nodo) proprio di questa identità (istanza di IQspnFingerprint).
*   Factory per creare uno "stub" per invocare metodi remoti nei vicini che hanno un arco con questa identità.
*   Evento che ha portato alla nascita di questa istanza. Una istanza, cioè una identità di questo sistema, o un
    nodo del grafo, può nascere per questi eventi:
    *   **Avvio** come nuova rete. In questo caso non vanno forniti altri requisiti. Inoltre sicuramente il set
        di archi è vuoto.
    *   **Ingresso** nella rete provenendo da una diversa rete. In questo caso vanno forniti questi ulteriori requisiti:
        *   Livello *k* del g-nodo che sta facendo questo ingresso in blocco. Può essere 0 se si tratta di un singolo
            nodo del grafo.
        *   Istanza dell'identità precedente. Per copiare da essa tutti i percorsi della mappa verso destinazioni di
            livello minore di *k*. Si tratta infatti di percorsi interni all'entità che entra in blocco.
        *   Tipo dell'identità. Cioè *principale* o *di connettività* ai livelli da *i* a *j*.
    *   **Migrazione** in un g-nodo provenendo da un diverso g-nodo della stessa rete. In questo caso vanno forniti
        questi ulteriori requisiti:
        *   Livello *k* del g-nodo che sta facendo questa migrazione in blocco. Può essere 0 se si tratta di un
            singolo nodo del grafo.
        *   Istanza dell'identità precedente. Per copiare da essa tutti i percorsi della mappa verso destinazioni
            di livello minore di *k*. Si tratta infatti di percorsi interni all'entità che migra in blocco.
        *   Tipo dell'identità. Cioè *principale* o *di connettività* ai livelli da *i* a *j*.

## <a name="Deliverable"></a>Deliverable

Ogni istanza del modulo QSPN creata per gestire una precisa identità del nodo:
*   Emette un segnale per:
    *   Il modulo ha completato il *bootstrap* per questa identità, cioè è uscito dalla fase di bootstrap. La
        definizione di tale fase è spiegata nel documento [esplorazione](EsplorazioneRete.md).
    *   I vicini sono stati notificati (tramite ETP) della presenza di questa identità.
    *   Rimosso un arco, perché non funzionava.
    *   Nuovo g-nodo nella mappa, rimosso g-nodo dalla mappa.
    *   Nuovo percorso, percorso cambiato o percorso rimosso per un certo g-nodo.
    *   Cambio nel fingerprint di uno dei miei g-nodi.
    *   Cambio nel numero approssimato di nodi interni ad uno dei miei g-nodi.
    *   Rilevamento di un g-nodo splittato.
*   Fornisce metodi per:
    *   Chiedere se il nodo ha completato il bootstrap nella rete. Restituisce un booleano.
    *   Dato un arco, chiedere l'indirizzo Netsukuku del vicino collegato. La risposta è *null* se
        ancora nessun ETP è stato ricevuto da tale arco e processato: infatti in questo caso il
        modulo non conosce ancora l'indirizzo Netsukuku del vicino collegato. Questo significa che
        lo stesso metodo può essere usato per sapere se almeno un ETP è stato ricevuto da tale arco e processato.
    *   Dato un livello *i*, ottenere l'elenco dei g-nodi *reali* di livello *i* presenti nella mappa
        (quindi nel mio g-nodo di livello *i* + 1). Restituisce una lista di HCoord. Se il nodo è nella fase
        di bootstrap a livello *i* o inferiore, lancia eccezione QspnBootstrapInProgressError.
    *   Relativamente ad un g-nodo a cui il nodo non appartiene, vale a dire dato un HCoord *dst*, ottenere
        tutti i percorsi a disposizione per raggiungerlo, ordinati per costo crescente e per primi quelli
        disgiunti. Metodo `List<IQspnNodePath> get_paths_to`. Restituisce una lista di IQspnNodePath. Se il
        nodo è nella fase di bootstrap a livello *dst.lvl* o inferiore, lancia eccezione QspnBootstrapInProgressError.
    *   Relativamente ad uno dei g-nodi a cui appartiene il nodo, vale a dire dato un livello *i*
        da 0 a *l* compresi, ottenere:
        *   il fingerprint del g-nodo. Restituisce un IQspnFingerprint. Se il nodo è nella fase di bootstrap
            a livello *i* o inferiore, lancia eccezione QspnBootstrapInProgressError.
        *   una approssimazione del numero di singoli nodi al suo interno. In questo computo non si considerano
            le identità *di connettività*. Restituisce un intero. Se il nodo è nella fase di bootstrap a livello
            *i* o inferiore, lancia eccezione QspnBootstrapInProgressError.
*   Se l'istanza gestisce una identità *di connettività* ai livelli da *i* a *j* nel g-nodo *w* di livello
    *i* - 1, fornisce metodi per:
    *   Rimuovere gli archi verso diretti vicini esterni al proprio g-nodo di livello *j*.
    *   Verificare la connettività dei propri g-nodi di livello da *i* a *j* in assenza del g-nodo *w*.
    *   Istruire i propri vicini interni a *w* che *w* deve essere dismesso. I vicini dovranno propagare
        l'informazione in tutto *w*.
    *   Segnalare ai propri vicini esterni a *w* che *w* sta per essere dismesso.
*   Se l'istanza gestisce una identità *di connettività* ai livelli da *i* a *j* nel g-nodo *w* di livello
    *i* - 1, emette un segnale per:
    *   L'identità che detiene questa istanza del modulo QSPN va rimossa, perché *w* viene dismesso.

## <a name="Classi_e_interfacce"></a>Classi e interfacce

L'oggetto che rappresenta gli indirizzi dei nodi non è del tutto noto a questo modulo, che conosce alcune
sue interfacce a seconda dell'uso che può farne.

Per il proprio indirizzo il nodo conosce l'interfaccia IQspnMyNaddr, per gli indirizzi di altri nodi conosce
l'interfaccia IQspnNaddr.

Questi i metodi delle interfacce note al modulo:

*   IQspnNaddr
    *   leggere i parametri della topologia della rete, cioè *l* e *gsize(i)* con *i* da 0 a *l* - 1;
    *   leggere *pos(i)* di questo indirizzo, con *i* da 0 a *l* - 1.
*   IQspnMyNaddr (che richiede IQspnNaddr)
    *   dato un IQspnNaddr (indirizzo di un nodo) ottenere il HCoord riferito al massimo distinto g-nodo
        che lo contiene, distinto rispetto al nostro nodo.

* * *

<a name="HCoord"></a>

La classe HCoord è una classe [comune](../Librerie/Common.md) nota a questo modulo. È una classe serializzabile,
cioè le cui istanze sono adatte al passaggio di dati a metodi remoti (vedi framework [ZCD](../Librerie/ZCD.md)). Una
sua istanza contiene le coordinate gerarchiche di un g-nodo nella mappa del  nodo: livello e identificativo nel livello.

* * *

I passi che costituiscono un percorso noto verso una destinazione sono rappresentati da una doppia sequenza di *k* elementi:

*   *hops* : sequenza di *k* istanze di HCoord.
*   *arcs* : sequenza di *k* identificativi di arco, dove `arcs[0]` indica l'arco che congiunge il nodo corrente
    a `hops[0]`, e `arcs[j]`, con *j* > 0, indica l'arco che congiunge `hops[j-1]` a `hops[j]`.

È inclusa in testa la coordinata che rappresenta il vicino che usiamo come gateway (o meglio il suo massimo distinto
g-nodo rispetto a noi) e in coda la coordinata che rappresenta la destinazione.

La sequenza *hops* è sempre in termini di g-nodi che hanno il g-nodo superiore in  comune con il nodo corrente.

Non viene definita una classe per contenere questa informazione: si usa una lista di HCoord e una lista di **int**.

* * *

I passi che costituiscono il percorso fatto da un ETP sono rappresentati da una singola sequenza di istanze di HCoord.

Per questa informazione si usa una lista di HCoord.

* * *

Il fingerprint di un g-nodo è un oggetto che il modulo non istanzia da solo; gli viene passato il suo fingerprint
di nodo (a livello 0) e il modulo ne conosce l'interfaccia IQspnFingerprint. Il modulo inoltre assume che sia un oggetto serializzabile.

L'interfaccia IQspnFingerprint consente di:

*   Leggere il livello del g-nodo a cui si riferisce (metodo *i_qspn_get_level*).
*   Confrontare due fingerprint che si riferiscono a due distinti g-nodi di livello *i* - 1 appartenenti al mio stesso
    g-nodo di livello *i*, con 1 ≤ *i* ≤ *l*, dove *l* è il numero di livelli. Decidere quale sia più anziano all'interno
    del mio g-nodo.  
    In effetti questo metodo non è esposto dall'interfaccia IQspnFingerprint, ma solo usato internamente alla classe
    (fornita dall'utilizzatore del modulo) per implementare il metodo *i_qspn_construct* descritto sotto.
*   Confrontare due fingerprint che si riferiscono ad un unico g-nodo di livello *i* - 1 appartenente al mio stesso
    g-nodo di livello *i*, con 1 ≤ *i* ≤ *l*, dove *l* è il numero di livelli, i quali fingerprint sono stati portati
    a conoscenza del mio nodo attraverso distinti percorsi. Stabilire se sono identici (metodo *i_qspn_equals*).
*   Confrontare due fingerprint che si riferiscono ad un unico g-nodo di livello *i* - 1 appartenente al mio stesso
    g-nodo di livello *i*, con 2 ≤ *i* ≤ *l*, dove *l* è il numero di livelli, i quali fingerprint sono stati portati
    a conoscenza del mio nodo attraverso distinti percorsi **e** non sono identici. Stabilire a quale dei due sia
    stato assegnato l'identificativo dal g-nodo di livello *i* - 2 più anziano (metodo *i_qspn_elder_seed*).  
    In questo caso il minimo valore di *i* è 2, in quanto non ha senso confrontare due fingerprint dello stesso singolo
    nodo (g-nodo di livello 0) che certamente non può subire uno split.
*   Partendo dal fingerprint del proprio g-nodo *g* di livello *i* - 1, dati i fingerprint di tutti gli altri g-nodi
    conosciuti di livello *i* - 1 dentro il mio g-nodo *h* di livello *i*, con 1 ≤ *i* ≤ *l*, dove *l* è il numero di
    livelli, ottenere l'istanza di fingerprint del g-nodo *h* (metodo *i_qspn_construct*).

* * *

Il costo di un arco e il costo di un percorso sono rappresentati da istanze di una classe non del tutto nota a
questo modulo. La sua interfaccia nota al modulo (IQspnCost) gli consente di:

*   Sommare il costo di un percorso a quello di un arco (metodo *i_qspn_add_segment*).
*   Comparare il costo di due percorsi (metodo *i_qspn_compare_to*).  
    Il funzionamento di questo comparatore è il seguente. Un oggetto IQspnCost indica una misura del costo di un
    percorso. Quindi dato il costo *a* e il costo *b* si dice che *a* > *b* se inviare un messaggio per il tramite
    di *a* è più "oneroso" che inviarlo per il tramite di *b*. Si traduce questa formula con *a.i_qspn_compare_to(b)* > 0.  
    Analogamente per gli altri operatori di confronto ( = , < , ≤ , ≥ , ≠ ).
*   Confrontare due costi (riferiti allo stesso percorso) per valutare se c'è stata una variazione
    significativa (metodo *i_qspn_important_variation*).

Il costo di un percorso, che viene pubblicizzato al modulo QSPN da un vicino, può essere un costo fittizio per
indicare una certa situazione – come *null* per indicare che la destinazione è proprio il vicino, o *dead* per
indicare che il percorso non è più funzionante. Invece il costo di un arco, che viene passato al modulo QSPN dal
suo utilizzatore, è sempre un valore frutto di una reale misurazione. Infatti non ha alcun significato un arco verso
me stesso, e un arco non funzionante viene semplicemente rimosso.

Quindi l'interfaccia IQspnCost consente anche di:

*   Vedere se il valore è *null* (metodo *i_qspn_is_null*).
*   Vedere se il valore è *dead* (metodo *i_qspn_is_dead*).

Un'altra caratteristica importante di questi due costi fittizzi (*null* e *dead*) è che essi rappresentano il
minimo assoluto e il massimo assoluto. Come tali, essi possono essere sommati o confrontati con qualsiasi altro
costo indipendentemente dalla metrica a cui si riferisce (latenza, larghezza di banda, ecc.). Infatti, sia
*c<sub>n</sub>* la costante costo *null*, sia *c<sub>d</sub>* la costante costo *dead* e sia *w* un costo che
rappresenta una effettiva misurazione di una qualsiasi metrica. Abbiamo queste proprietà:

*   c<sub>n</sub> = c<sub>n</sub>
*   c<sub>n</sub> < w
*   c<sub>n</sub> < c<sub>d</sub>
*   w < c<sub>d</sub>
*   c<sub>d</sub> = c<sub>d</sub>
*   c<sub>n</sub> + c<sub>n</sub> = c<sub>n</sub>
*   c<sub>n</sub> + w = w
*   c<sub>n</sub> + c<sub>d</sub> = c<sub>d</sub>
*   c<sub>d</sub> + w = c<sub>d</sub>
*   c<sub>d</sub> + c<sub>d</sub> = c<sub>d</sub>

* * *

Un arco è un oggetto il cui contenuto non è del tutto noto al modulo QSPN. L'interfaccia di questo oggetto
nota al modulo (IQspnArc) gli consente di:

*   Verificare se due archi sono identici (metodo *i_qspn_equals*).
*   Leggere il costo associato all'arco (metodo *i_qspn_get_cost*).
*   Data una istanza di CallerInfo, passata all'inizio dell'esecuzione di un metodo remoto
    (vedi framework [ZCD](../Librerie/ZCD.md)), verificare se la chiamata del metodo è stata ricevuta
    tramite questo arco (metodo *i_qspn_comes_from*).

* * *

Un ETP è una istanza della classe EtpMessage che è interna al modulo. Essa è serializzabile. Un ETP contiene:

*   L'indirizzo del nodo *n* che produce l'ETP. Proprietà `IQspnNaddr node_address`.  
    Il modulo assume che sia anche un oggetto serializzabile. L'utilizzatore del modulo deve provvedere che la
    classe che concretizza un IQspnMyNaddr (che è anche un IQspnNaddr) che viene passata come proprio indirizzo
    nel costruttore sia serializable.
*   La lista di fingerprint per i g-nodi di *n* ai livelli da 0 a *l* - 1. Proprietà `List<IQspnFingerprint> fingerprints`.  
    Il modulo assume che ognuna sia anche un oggetto serializzabile.
*   La lista del numero approssimativo di nodi all'interno dei g-nodi di *n* ai livelli da 0 a *l* - 1. Proprietà
    `List<int> nodes_inside`.
*   La lista dei g-nodi attraversati da questo ETP. Proprietà `List<HCoord> hops`.
*   La lista *P* dei percorsi. Ogni percorso *p* ∈ *P* è una istanza di EtpPath, descritta sotto. Proprietà
    `List<EtpPath> p_list`.

* * *

Un percorso *p* scritto nella lista *P* di un ETP è una istanza della classe EtpPath che è interna al modulo.
Essa è serializzabile. Contiene:

*   La lista dei g-nodi e relativi archi che costituiscono il percorso *p* fino al g-nodo destinazione *d*. Quando
    si riceve un ETP questa lista non comprende il nostro vicino, cioè il nodo *n* che ha prodotto l'ETP; ma
    dopo aver eseguito la Grouping Rule (descritta nel documento [esplorazione](EsplorazioneRete.md)) conterrà
    anche *n* e in  tale stato verrà memorizzato nella mappa dentro un oggetto NodePath. Si compone delle due
    proprietà `List<HCoord> hops` e `List<int> arcs`.
*   Il costo di *p* da *n* a *d*. È una istanza dell'interfaccia IQspnCost. Il modulo assume che sia anche un
    oggetto serializzabile. Proprietà `IQspnCost cost`.
*   Il fingerprint del g-nodo *d* come riportato da questo percorso. È una istanza dell'interfaccia IQspnFingerprint.
    Il modulo assume che sia anche un oggetto serializzabile. Proprietà `IQspnFingerprint fingerprint`.
*   Il numero di nodi nel g-nodo *d* come riportato da questo percorso. Proprietà `int nodes_inside`.
*   Una lista di *l* booleani il cui elemento *i*-esimo (da 0 a *l* - 1) dice se va ignorato questo percorso dai nodi
    che non appartengono al g-nodo di livello *i* del nodo *n* che ha prodotto l'ETP. In realtà per *i* = 0 il
    valore è sempre *false* ma per semplicità teniamo anche questo valore. Proprietà `List<bool> ignore_outside`.

* * *

<a name="Path"></a>

La classe NodePath è interna al modulo. Una sua istanza rappresenta un percorso da questo nodo alla destinazione
comprensivo dell'arco dal nodo al vicino che ha pubblicizzato il percorso. Contiene:

*   L'arco da usare per raggiungere il vicino gateway. Proprietà `IQspnArc arc`.
*   Il percorso come è stato pubblicizzato dal vicino attraverso questo arco. Proprietà `EtpPath path`.

* * *

Il percorso fornito dal metodo pubblico *get_paths_to* del modulo non ha le stesse informazioni usate
internamente al modulo e presenti nella classe NodePath. Si usa un'altra classe, RetPath. Anche questa è una
classe interna al modulo. La sua interfaccia nota all'esterno del modulo (IQspnNodePath) consente di:

*   Leggere l'arco come IQspnArc.
*   Leggere i passi successivi, fino alla destinazione compresa, come IQspnHop (vedi sotto).
*   Leggere il costo come IQspnCost.
*   Leggere il numero di nodi approssimati all'interno del g-nodo destinazione.
*   Verificare, data un'altra istanza, se rappresenta lo stesso percorso.

* * *

L'oggetto che rappresenta un passo dentro un IQspnNodePath è una istanza della della classe RetHop. Tale classe
è interna al modulo. La sua interfaccia nota all'esterno del modulo (IQspnHop) consente di:

*   Leggere l'identificativo dell'arco (un intero) che permette di raggiungere questo passo.
*   Leggere l'indirizzo del passo come HCoord (da correlare al proprio indirizzo Netsukuku).

* * *

Se per un g-nodo *g* vengono rilevati due percorsi (istanze di IQspnNodePath) che differiscono per il loro
fingerprint e se questa situazione si mantiene per un certo lasso di tempo, questo è sintomo dello split del g-nodo *g*.

Per valutare quanto deve attendere prima di segnalare lo split del g-nodo, al modulo viene fornito un oggetto dal
suo utilizzatore, che implementa l'interfaccia IQspnThresholdCalculator. Tramite essa il modulo può:

*   Calcolare, passando un paio di istanze di IQspnNodePath che rappresentano i percorsi discordi, il tempo di
    tolleranza in millisecondi che deve passare da quando si verifica il disallineamento per poter segnalare lo split
    del g-nodo (metodo *i_qspn_calculate_threshold*).

* * *

La stub factory è un oggetto di cui il modulo conosce l'interfaccia IQspnStubFactory. Tramite essa il modulo può:

*   Creare uno stub per chiamare un metodo in broadcast su tutti i propri vicini. Il modulo indica tutti gli
    archi su cui il messaggio deve essere trasmesso.  
    Si consideri che l'implementazione della stub factory utilizzerà il modulo Neighborhood e che il modulo
    QSPN è un modulo *di identità*. Quindi il concetto di arco nel modulo QSPN è uguale al concetto di *arco-identità*
    introdotto nella documentazione del modulo Neighborhood e del modulo Identities.  
    Di fatto una chiamata a metodo remoto fatta con questo stub implica un messaggio che verrà trasmesso una sola
    volta su ogni interfaccia di rete gestita dal nodo. Ma il modulo QSPN può tranquillamente assumere che si tratta
    di una trasmissione non-reliable che viene fatta su ogni singolo *arco* che esso indica.  
    Le chiamate a metodi remoti fatte con questo stub procedono in modo asincrono: l'invio del messaggio procederà in
    una nuova tasklet, mentre il metodo non fornirà alcuna risposta al chiamante. Il modulo può fornire un oggetto
    (istanza di IQspnMissingArcHandler, descritta sotto) in cui un determinato metodo (callback) verrà richiamato dopo
    un certo tempo se per qualcuno degli archi indicati dal modulo non si avrà ricevuto un messaggio di ACK dal
    vicino collegato. Il metodo callback viene chiamato una volta per ogni arco che fallisce e avrà quell'arco come
    argomento, così che il chiamante possa prendere un provvedimento, ad esempio tentando un messaggio con protocollo reliable.
*   Creare uno stub per chiamare un metodo in modo reliable su un vicino tramite un dato arco.  
    Le chiamate a metodi remoti fatte con questo stub procedono in modo sincrono: l'invio del messaggio avviene nella
    stessa tasklet, e il metodo fornirà una risposta al chiamante, che può segnalare la corretta ricezione del messaggio
    o un errore. Il modulo ha la possibilità di dichiarare di voler attendere la processazione del messaggio o soltanto
    la sua ricezione. In ogni caso se non riceve una eccezione StubError il modulo è certo che il messaggio è stato
    ricevuto. In caso contrario il modulo considera l'arco non funzionante; di norma gestisce questa eventualità
    rimuovendo l'arco dalla sua lista e emettendo il relativo segnale.

* * *

Il gestore per gli archi che non segnalano la ricezione di un messaggio in broadcast è una istanza di una classe
interna al modulo, che implementa l'interfaccia IQspnMissingArcHandler. Tramite essa:

*   Al modulo viene segnalato un arco da cui non si è ricevuta la notifica di ricezione del messaggio (metodo *i_qspn_missing*).

