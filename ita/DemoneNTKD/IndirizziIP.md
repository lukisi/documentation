# Demone NTKD - Indirizzi IP

1.  [Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IP](#Mappatura)
    1.  [Calcolo degli indirizzi IP](#Calcolo)
    1.  [Esempio](#Esempio)
1.  [Indirizzi di interesse per una identità](#Identita)
    1.  [Calcolo indirizzi IP locali e destinazioni](#Computo)
    1.  [Utilizzo indirizzi IP locali](#Utilizzo_locali)
    1.  [Utilizzo indirizzi IP destinazioni](#Utilizzo_destinazioni)

## <a name="Mappatura"></a> Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IP

Lo spazio di indirizzi Netsukuku (quello costituito dagli indirizzi *reali* che sono assegnati alle
identità *principali*) va mappato in un range di indirizzi IP che si
decide di destinare alla rete Netsukuku. Nell'attuale implementazione si presume che questo
range sia la classe IPv4 10.0.0.0/8.

La rete viene suddivisa in un numero arbitrario di livelli.
La [notazione CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) usata per individuare
classi di indirizzi nelle reti IPv4, ci obbliga ad usare come gsize ad ogni livello una potenza di 2.
In teoria, ad esempio, potremmo avere un livello 0 di gsize 5. Ma quando vogliamo indicare nelle
tabelle di routing tutti gli indirizzi di un g-nodo di livello 1 (ad esempio da 10.1.2.0 a 10.1.2.4) non
potremmo farlo in forma compatta. Invece se usiamo un livello 0 di gsize 8, per riferirci agli indirizzi
nel g-nodo da 10.1.2.0 a 10.1.2.7 possiamo usare la notazione 10.1.2.0/29; per gli indirizzi da 10.1.2.8
a 10.1.2.15 useremo 10.1.2.8/29; e così via.

Indichiamo con *l* il numero dei livelli. Indichiamo con *gsize(i)* la dimensione dei g-nodi di livello
*i* + 1. Tali dimensioni devono essere potenze di 2. Indichiamo con *g-exp(i)* l'esponente della potenza
di 2 che dà *gsize(i)*. Il numero di bit necessari a coprire lo spazio
di indirizzi è dato dalla sommatoria per *i* da 0 a *l* - 1 di *g-exp(i)*. 𝛴 *<sub>0 ≤ i < l</sub>* *g-exp(i)*.

Questo numero di bit non può essere maggiore di 22. Gli indirizzi IP nella classe 10.0.0.0/8 hanno 24 bit
a disposizione. Però ai bit sfruttati per la rappresentazione di un indirizzo Netsukuku vanno aggiunti 2 bit,
nella posizione più significativa, che riserviamo per notazioni particolari (routing interno ai g-nodi e
forma anonima).

Inoltre dobbiamo assicurarci che *gsize(l-1)* ≥ *l*. Cioè che nello spazio destinato al livello più alto
sia possibile rappresentare un numero da 0 a *l* - 1. Questo pure ci serve per la notazione usata per il
routing interno ai g-nodi.

Una volta scelti i valori di *l* e di *g-exp(i)* rispettando i vincoli prima esposti, questa mappatura
associa ad un indirizzo Netsukuku *reale* un numero di indirizzi IP:

*   Un indirizzo IP globale.  
    Questo indirizzo IP identifica un preciso *sistema* univocamente all'interno
    di tutta la rete.
*   Un indirizzo IP globale *anonimizzante*.  
    Questo indirizzo IP identifica un preciso *sistema* univocamente all'interno
    di tutta la rete. È una diversa rappresentazione, rispetto all'indirizzo IP globale,
    che identifica lo stesso *sistema*; ma questa convoglia in più l'informazione che il client vuole
    contattare quel *sistema* restando anonimo.
*   Un indirizzo IP interno al livello *i* per ogni valore di *i* da 0 a *l* - 1.  
    Questo indirizzo IP identifica un preciso *sistema* univocamente all'interno
    del suo g-nodo *g* di livello *i*. Questo indirizzo IP si può utilizzare come indirizzo di
    destinazione di un pacchetto IP quando sia il *sistema* mittente che il *sistema*
    identificato (il destinatario) appartengono allo stesso g-nodo *g* di livello *i*. La
    peculiarità di questi indirizzi IP (che si riflette sui pacchetti IP trasmessi a questi
    indirizzi) è che essi non cambiano quando il g-nodo *g* o uno dei suoi g-nodi superiori
    *migra* all'interno della rete o anche in una diversa rete Netsukuku.  
    Anche per il livello 0 si calcola un tale indirizzo IP. Questo indirizzo ha un significato
    simile a `localhost` nel senso che identifica come destinazione lo stesso sistema mittente.

### <a name="Calcolo"></a> Calcolo degli indirizzi IP

Illustriamo come si calcolano gli indirizzi IP locali di un nodo del grafo, cioè quelli che l'identità *principale*
di un sistema vuole assegnarsi nel network namespace default.

Illustriamo anche come si calcolano gli indirizzi IP con notazione CIDR che rappresentano un g-nodo destinazione.
Ogni identità (la principale o una di connettività) è interessata a calcolare questi indirizzi IP per inserirli
nelle tabelle di routing del proprio network namespace, ma solo per i g-nodi che sono visibili nella sua mappa,
cioè quelli che sono rappresentabili anche come coordinate gerarchiche relative al proprio indirizzo di nodo.

Ricordiamo i vincoli sopra esposti nella scelta della topologia della rete. In particolare abbiamo detto
che bisogna lasciare 2 bit liberi nello spazio degli indirizzi IP.

I due bit subito più alti del numero di bit necessari a codificare un indirizzo Netsukuku *reale*
sono riservati. Ad esempio, supponiamo di definire una topologia che sfrutta tutti i 22 bit disponibili
nella classe 10.0.0.0/8 di IPv4. Quindi per codificare un indirizzo Netsukuku *reale* si usano i
bit da 0 a 21. Allora questi due bit subito più alti di cui parliamo sono il 23 e il 22.

Il numero riportato in questi due bit indica il tipo di indirizzo IP:

*   0 - In binario `|0|0|`. Indirizzo IP globale.
*   1 - In binario `|0|1|`. Indirizzo IP interno ad un g-nodo.
*   2 - In binario `|1|0|`. Indirizzo IP globale anonimizzante.
*   3 - In binario `|1|1|`. Riservato ad usi futuri.

Indichiamo con *l* il numero dei livelli. Indichiamo con *gsize(i)* la dimensione dei g-nodi di livello
*i* + 1, che abbiamo detto è una potenza di 2. Indichiamo con *g-exp(i)* l'esponente della potenza
di 2 equivalente a *gsize(i)*.

Una volta scelti i valori di *l* e di *g-exp(i)* rispettando i vincoli prima ricordati, vediamo come si
calcolano i vari tipi di indirizzo IP partendo da un indirizzo Netsukuku (di nodo o di g-nodo).

#### Indirizzo IP globale di un nodo

Sia *n* l'indirizzo Netsukuku di un nodo. Indichiamo con *n<sub>0</sub>* l'identificativo del nodo
all'interno del suo g-nodo di livello 1. E a seguire con *n<sub>i</sub>* l'identificativo del g-nodo di
livello *i* a cui appartiene *n* all'interno del suo g-nodo di livello *i* + 1. L'indirizzo completo sarà
*n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*.

Il valore *n<sub>0</sub>* viene riportato nei bit meno significativi dell'indirizzo IP che stiamo
componendo, cioè partendo dal bit 0 per un numero di *g-exp*(0) bit. Il valore *n<sub>1</sub>* viene
riportato nei successivi bit, cioè partendo da *g-exp*(0) per un numero di *g-exp*(1) bit. E così di
seguito, l'identificativo *n<sub>i</sub>* (con *i* che arriva fino a *l* - 1) viene riportato nei bit
partendo da 𝛴 *<sub>0 ≤ k ≤ i-1</sub>* *g-exp(k)* per un numero di *g-exp(i)* bit.

I due bit più alti (quelli riservati per indicare il tipo di indirizzo IP) li impostiamo a `|0|0|`.

#### Indirizzo IP globale di un g-nodo

Sia *g* l'indirizzo Netsukuku di un g-nodo di livello *i*. L'indirizzo completo sarà
*g<sub>l-1</sub>·...·g<sub>i</sub>*.

Il valore *g<sub>i</sub>* viene riportato nell'indirizzo IP che stiamo componendo partendo dal
bit 𝛴 *<sub>0 ≤ k ≤ i-1</sub>* *g-exp(k)* per un numero di *g-exp*(i) bit. I bit meno significativi
sono messi a 0. Quei bit non saranno comunque presi in considerazione a causa dal suffisso di routing
della notazione CIDR, trattandosi dell'indirizzo IP di un intero g-nodo considerato come una IP subnet.

I valori dei bit più alti si calcolano come visto prima per l'indirizzo di un singolo nodo. Infine si
aggiunge, come accennato, il suffisso di routing, ottenuto come 32 - 𝛴 *<sub>0 ≤ k ≤ i-1</sub>* *g-exp(k)*.

#### Indirizzo IP di un nodo interno ad un suo g-nodo

Sia *n* un nodo con indirizzo *n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*. Sia *g* il suo g-nodo
di livello *i* con 0 ≤ *i* < *l*. Quindi *g* ha indirizzo *n<sub>l-1</sub>·...·n<sub>i</sub>*. Vogliamo
comporre un indirizzo IP di *n* che sia univoco internamente a *g*.

I valori da *n<sub>0</sub>* a *n<sub>i-1</sub>* sono riportati come visto prima nei relativi bit
dell'indirizzo IP che stiamo componendo. Il valore *i* viene riportato nei bit che sarebbero stati
destinati all'identificativo di livello più alto, cioè *n<sub>l-1</sub>*. Gli altri bit, quelli che
avrebbero ospitato gli identificativi da *i* a *l* - 2, sono lasciati a 0.

I due bit più alti li impostiamo a `|0|1|`.

#### Indirizzo IP di un g-nodo interno ad un suo g-nodo superiore

Sia *g* un g-nodo di livello *i* con indirizzo *g<sub>l-1</sub>·...·g<sub>i</sub>* con *i* < *l* - 1.
Sia *h* un suo g-nodo superiore di livello *k*. Quindi *h* ha indirizzo *g<sub>l-1</sub>·...·g<sub>k</sub>*, con
*k* > *i*. Vogliamo comporre un indirizzo IP in notazione CIDR di *g* che sia univoco internamente a *h*.

Per ogni valore *t* da *i* a *k* - 1, il valore di *g<sub>t</sub>* è riportato come visto prima nei relativi
bit dell'indirizzo IP che stiamo componendo. Il valore *k* viene riportato nei bit che sarebbero stati
destinati all'identificativo di livello più alto, cioè *g<sub>l-1</sub>*. Gli altri bit sono lasciati a 0.
Si aggiunge, come prima, il suffisso di routing.

I due bit più alti li impostiamo a `|0|1|`.

#### Indirizzo IP di un nodo o g-nodo contattabile in forma anonima

Sia *n* un nodo con indirizzo *n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*. Vogliamo comporre
un indirizzo IP per tale risorsa tale che un client lo possa usare per contattare il nodo mantenendo
l'anonimato. Chiamiamo un tale indirizzo *anonimizzante*.

I valori da *n<sub>0</sub>* a *n<sub>l-1</sub>* sono riportati come visto prima nei relativi bit
dell'indirizzo IP che stiamo componendo.

I due bit più alti li impostiamo a `|1|0|`.

Nel caso di un g-nodo, per produrre un indirizzo anonimizzante in notazione CIDR si procede in modo
analogo, aggiungendo il suffisso come visto prima.

### <a name="Esempio"></a> Esempio

Prendiamo ad esempio una rete con 4 livelli. In essa abbiamo
2 bit al livello 3, 4 bit al livello 2, 8 bit ai livelli 1 e 0.

Gli indirizzi validi sono 2<sup>22</sup>, cioè circa 4 milioni. Vediamo qual è la dimensione massima
della mappa di un nodo del grafo.

Il numero di indirizzi Netsukuku che ogni nodo dovrà al massimo memorizzare come destinazioni
nella sua mappa di percorsi è di 1×3 + 1×15 + 2×255 = 528. Infatti la *gsize* del livello 3
(cioè il numero massimo di g-nodi di livello 3 dentro lo stesso g-nodo di livello 4)
è 4; ma per tale livello ogni nodo dovrà memorizzare al massimo 3 diversi g-nodi destinazione di livello 3
in quanto il suo stesso g-nodo di livello 3 non sarà mai una destinazione. Allo stesso modo la *gsize* del livello
2 è 16; ma per tale livello ogni nodo dovrà memorizzare al massimo 15 diversi g-nodi destinazione di livello 2
in quanto il suo stesso g-nodo di livello 2 non sarà mai una destinazione. Allo stesso modo la *gsize* del livello
1 è 256; ma per tale livello ogni nodo dovrà memorizzare al massimo 255 diversi g-nodi destinazione di livello 1
in quanto il suo stesso g-nodo di livello 1 non sarà mai una destinazione. Allo stesso modo la *gsize* del livello
0 è 256; ma per tale livello ogni nodo dovrà memorizzare al massimo 255 diversi singoli nodi destinazione
in quanto esso stesso non sarà mai una destinazione.

**Nota:** La topologia portata ad esempio qui sopra è volutamente semplice. Il numero di possibili indirizzi Netsukuku di
destinazione rimane molto alto. La suddivisione ottimale (che riduce al minimo tale numero) verrà descritta alla fine di
questo paragrafo.

Prendiamo 3 nodi. Il nodo *n* è quello di cui esaminiamo le tabelle di  routing. Il nodo *m* è un
nodo con cui *n* ha un qualche g-nodo in comune. Il nodo *o* è un nodo con cui *n* non ha in comune nemmeno
il g-nodo più alto. Vediamo quali informazioni mantiene il nodo *n*.

#### Nodo *n*

Per il nodo *n* sia l'indirizzo Netsukuku 3·10·123·45.

Computiamo il nostro IP globale e i 3 IP validi internamente ad un g-nodo.

```
Globale di *n*:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
   3                   1 1
  10                       1 0 1 0
 123                                 0 1 1 1 1 0 1 1
  45                                                   0 0 1 0 1 1 0 1
 = 10.58.123.45

Interno di *n* nel suo g-nodo di livello 3:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello g-nodo       1 1
  10                       1 0 1 0
 123                                 0 1 1 1 1 0 1 1
  45                                                   0 0 1 0 1 1 0 1
 = 10.122.123.45

Interno di *n* nel suo g-nodo di livello 2:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello g-nodo       1 0
   N/A                     0 0 0 0
 123                                 0 1 1 1 1 0 1 1
  45                                                   0 0 1 0 1 1 0 1
 = 10.96.123.45

Interno di *n* nel suo g-nodo di livello 1:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello g-nodo       0 1
   N/A                     0 0 0 0
   N/A                               0 0 0 0 0 0 0 0
  45                                                   0 0 1 0 1 1 0 1
 = 10.80.0.45
```

*n* si assegna quindi questi IP:

*   il globale: 10.58.123.45
*   l'interno nel livello 3: 10.122.123.45
*   l'interno nel livello 2: 10.96.123.45
*   l'interno nel livello 1: 10.80.0.45

#### Nodo *m*

Per il nodo *m* sia l'indirizzo Netsukuku 3·10·67·89.

Computiamo il suo indirizzo IP globale e quello interno al g-nodo di livello 2 (quello in comune con *n*).

```
Globale di *m*:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
   3                   1 1
  10                       1 0 1 0
  67                                 0 1 0 0 0 0 1 1
  89                                                   0 1 0 1 1 0 0 1
 = 10.58.67.89

Interno di *m* nel suo g-nodo di livello 2:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello g-nodo       1 0
   N/A                     0 0 0 0
  67                                 0 1 0 0 0 0 1 1
  89                                                   0 1 0 1 1 0 0 1
 = 10.96.67.89
```

Quando *n* riceve un ETP che contiene il g-nodo con *m* esso lo vede come (1, 67) cioè come g-nodo *g*
di livello 1 appartenente al suo stesso g-nodo di livello 2 e con identificativo 67 (a livello 1).

Per il g-nodo *g* l'indirizzo Netsukuku è 3·10·67.

Il nodo *n* computa:

```
Globale di *g*:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[0|0|0|0|0|0|0|0]
   3                   1 1
  10                       1 0 1 0
  67                                 0 1 0 0 0 0 1 1
 = 10.58.67.0/24

Interno di *g* nel suo g-nodo di livello 2:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[0|0|0|0|0|0|0|0]
  livello g-nodo       1 0
   N/A                     0 0 0 0
  67                                 0 1 0 0 0 0 1 1
 = 10.96.67.0/24
```

Quindi *n* imposta:

*   la rotta globale: 10.58.67.0/24 via xx dev yy src 10.58.123.45
*   la rotta interna al g-nodo di livello 2: 10.96.67.0/24 via xx dev yy src 10.96.123.45

Ipotiziamo ora di sfruttare il momento della risoluzione *hostname → IP address* per intervenire sulle
operazioni del nodo *n* quando questo vuole iniziare una connessione (o una trasmissione) con il
nodo *m*. Supponiamo che il nodo *m* abbia registrato per se il nome "morfeo".

Quando il nodo *n* richiede la risoluzione del nome "morfeo.ntk" il resolver cerca nel database
ANDNA il nome "morfeo" e trova l'indirizzo Netsukuku 3·10·67·89.

Invece di computare il relativo indirizzo IP globale 10.58.67.89, il resolver vede rispetto al proprio indirizzo
Netsukuku (quello dell'identità principale) qual'è il minimo comune g-nodo (in questo caso 2) e computa
il relativo indirizzo IP interno in quel g-nodo: 10.96.67.89.

Questo nella route table corrisponde a 10.96.67.0/24 quindi *n* manda il pacchetto al suo gateway
indicando come proprio IP 10.96.123.45.

Una volta realizzata una connessione TCP tra questi due indirizzi IP, questa connessione continuerebbe a funzionare anche
se un g-nodo di livello superiore migrasse, anche gradualmente un nodo alla volta, ad un altra posizione di pari livello.

Ad esempio se il g-nodo 3·5 migrasse dal g-nodo 3 al g-nodo 1 assumendo in esso l'identificativo 1·2.
Oppure se il g-nodo 3 in blocco facesse ingresso in una diversa rete assumendo in essa l'identificativo (di livello 3) 1.

Va detto che il nodo *n* sarà comunque in grado di inviare pacchetti e/o realizzare una connessione TCP direttamente
con l'indirizzo IP globale se lo conosce, cioè 10.58.67.89, ma in questo caso la connessione si romperebbe durante una tale migrazione.

La risoluzione inversa non subirebbe alterazioni. Quando il nodo *n* vuole sapere il nome dell'host che
ha indirizzo IP 10.96.67.89 (oppure 10.58.67.89) il resolver lo contatta e riceve la lista di nomi
che il nodo *m* ha registrato.

#### Nodo *o*

Per il nodo *o* sia l'indirizzo Netsukuku 2·10·237·242.

Computiamo solo il suo indirizzo IP globale poiché l'unico g-nodo comune con *n* è il 4 (intera rete).

```
Globale di *o*:
 = 10.42.237.242
```

Quando *n* riceve un ETP che contiene il g-nodo con *o* esso lo vede come (3, 2) cioè come g-nodo *h*
di livello 3 con identificativo 2.

Per il g-nodo *h* l'indirizzo Netsukuku è 2.

Il nodo *n* computa:

```
Globale di *h*:
[0|0|0|0|1|0|1|0].[0|0|?|?|0|0|0|0].[0|0|0|0|0|0|0|0].[0|0|0|0|0|0|0|0]
   2                   1 0
 = 10.32.0.0/12
```

Quindi *n* imposta solo la rotta globale: 10.32.0.0/12 via xx dev yy src 10.58.123.45

#### Richiesta di anonimato

Un nodo, opzionalmente, può dichiararsi disposto ad accettare richieste in forma anonima. Se intende
farlo, il nodo si assegna anche un altro indirizzo globale, identico al primo, ma con il bit *anonimizzante* impostato.

Ogni nodo, invece, per ogni destinazione di cui viene a conoscenza, aggiunge una rotta verso
l'indirizzo IP *anonimizzante* relativo.

Per esempio il nodo *n*:

*   Opzionalmente, si assegna l'indirizzo IP `10.186.123.45`.
*   Aggiunge la rotta `10.186.67.0/24 src 10.58.123.45`.
*   Aggiunge la rotta `10.160.0.0/12 src 10.58.123.45`.

Infine, i nodi che sono disposti a anonimizzare i vicini che ne fanno richiesta, impostano le regole di
source NATting come descritto [sotto](#Source_natting).

#### Suddivisione ottimale

Per ridurre al minimo il numero massimo di rotte da memorizzare in uno spazio a 24 bit come
la classe 10.0.0.0/8 si proceda come illustrato di seguito.

Abbiamo 24 bit a disposizione. Tolti 2 per le codifiche viste nel presente documento, abbiamo 22 bit.
Diamo 4 bit al livello alto; abbiamo così un *gsize* del livello più alto capace di rappresentare fino
a 16 livelli. Per sfruttarli tutti facciamo i livelli da 14 a 3 da 1 bit e i livelli 2, 1 e 0 da 2 bit.

Il numero di indirizzi Netsukuku che ogni nodo dovrà al massimo memorizzare come destinazioni nella
sua mappa di percorsi è di 1×15 + 12×1 + 3×3 = 36. Resta invariato che il numero massimo di nodi nella rete è 2<sup>22</sup>.

## <a name="Identita"></a> Indirizzi di interesse per una identità

Come abbiamo ricordato [qui](DettagliTecnici.md#Indirizzi_IP), l'identità principale del sistema ha un indirizzo
Netsukuku *reale*. Essa assegna degli indirizzi IPv4 locali nel network stack default. Inoltre assegna
delle rotte nelle tabelle sia per pacchetti IP generati localmente che per pacchetti IP da inoltrare. Queste
rotte hanno come possibile destinazione tutti i g-nodi di qualsiasi livello che possono essere espressi in
coordinate gerarchiche relative al suo indirizzo Netsukuku.

Invece, una identità di connettività ha un indirizzo Netsukuku con almeno una componente *virtuale*. Indichiamo
con *i* il livello più alto in cui l'indirizzo Netsukuku di questa identità ha la componente *virtuale*.
Essa non assegna alcun indirizzo IPv4 locale nel suo network stack. Invece assegna
delle rotte nelle tabelle solo per pacchetti IP da inoltrare. Queste
rotte hanno come possibile destinazione g-nodi di livello maggiore o uguale a *i* che possono essere espressi in
coordinate gerarchiche relative al suo indirizzo Netsukuku.

### <a name="Computo"></a> Calcolo indirizzi IP locali e destinazioni

Il computo degli indirizzi IP locali e delle possibili destinazioni associate ad una identità si fa quando
nasce l'identità stessa. Queste informazioni restano valide finché l'identità vive, oppure fino a quando
non si duplica. Quando una identità si duplica le informazioni ad essa associate vanno in parte alterate:
infatti l'indirizzo Netsukuku dell'identità originale cambia prendendo una componente virtuale.

Le strutture dati associate ad una identità che rappresentano queste informazioni sono `local_ip_set`
e `dest_ip_set`.

Vediamo nel dettaglio tutte le possibili situazioni.

#### Calcolo indirizzi IP locali

Questo si fa quando nasce una identità *principale*. Cioè all'avvio quando nasce la prima identità principale, oppure
quando si duplica una identità che era al momento la principale.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *subnetlevel* il livello del g-nodo rappresentato dalla sottorete autonoma.
*   Indichiamo con *n* l'indirizzo Netsukuku dell'identità.
*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Per ogni livello *k* da 0 a *l* - 1:
    *   Calcola `local_ip_set.internal[k] = ip_internal_node(n, inside_level=k)`.
*   Calcola `local_ip_set.global = ip_global_node(n)`.
*   Calcola `local_ip_set.anonymizing = ip_anonymizing_node(n)`.
*   Calcola `local_ip_set.anonymizing_range = ip_anonymizing_range()`.  
    Si tratta del range di indirizzi IP anonimizzanti che comprende tutta la rete.

Se *subnetlevel* > 0 si aggiunge:

*   Calcola `local_ip_set.netmap_range1 = ip_netmap_range1()`.  
    Si tratta del range di indirizzi IP interni al livello *subnetlevel* che contiene tutti i
    nodi della sottorete autonoma.
*   Per ogni livello *k* da *subnetlevel* a *l* - 2:
    *   Sia *g* il g-nodo di livello *k* + 1 di cui fa parte *n* (e tutta la sottorete autonoma).
    *   Calcola `local_ip_set.netmap_range2[k] = ip_netmap_range2(n, inside_level=k)`.  
        Si tratta del range di indirizzi IP interni a *g* che rappresenta la sottorete autonoma.  
        Si basa sulle posizioni di *n* da *subnetlevel* a *k*.
    *   Calcola `local_ip_set.netmap_range3[k] = ip_netmap_range3(inside_level=k)`.  
        Si tratta del range di indirizzi IP interni a *g* che comprende tutti i nodi in *g*.
*   Per il livello *l* - 1:
    *   Calcola `local_ip_set.netmap_range2_upper = ip_netmap_range2_upper(n)`.  
        Si tratta del range di indirizzi IP globali che rappresenta la sottorete autonoma.  
        Si basa sulle posizioni di *n* da *subnetlevel* in su.
    *   Calcola `local_ip_set.netmap_range3_upper = ip_netmap_range3_upper()`.  
        Si tratta del range di indirizzi IP globali che comprende tutta la rete.
*   Calcola `local_ip_set.netmap_range4 = ip_netmap_range4(n)`.  
    Si tratta del range di indirizzi IP anonimizzanti che rappresenta la sottorete autonoma.  
    Si basa sulle posizioni di *n* da *subnetlevel* in su.

#### Calcolo indirizzi IP destinazioni

Questo si fa quando nasce una generica identità. Cioè all'avvio quando nasce la prima identità principale, oppure
quando si duplica una generica identità.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *subnetlevel* il livello del g-nodo rappresentato dalla sottorete autonoma.
*   Indichiamo con *n* l'indirizzo Netsukuku dell'identità.
*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Se si tratta dell'identità principale sia *up_to* = -1. Altrimenti indichiamo con *up_to* il livello
    più alto in cui l'elemento *pos_n(i)* è virtuale.
*   Per *i* che scende da *l* - 1 a `$subnetlevel`,  
    se *i* >= *up_to*,  
    per *j* da 0 a *gsize(i)* - 1,  
    se *pos_n(i)* ≠ *j*:
    *   Indichiamo con *hc* il g-nodo (*i*, *j*).
    *   Calcola `hc_addr` l'indirizzo Netsukuku equivalente *hc* rispetto a *n*.  
        Cioè: `hc_addr = n.slice(i+1); hc_addr.insert_at(0,j)`;
    *   Calcola `dest_ip_set[hc].global = ip_global_gnode(hc_addr)`.
    *   Calcola `dest_ip_set[hc].anonymizing = ip_anonymizing_gnode(hc_addr)`.
    *   Per *k* che scende da *l* - 1 a *i* + 1:
        *   Calcola `dest_ip_set[hc].internal[k] = ip_internal_gnode(hc_addr, inside_level=k)`.

#### Variazioni quando si diventa di connettività

Questo si fa quando una generica identità (che poteva essere principale o di connettività) si duplica
e la originale diventa di connettività per un dato livello.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *n* l'indirizzo Netsukuku dell'identità.
*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Indichiamo con *prev_lvl* il livello in cui la componente da reale è appena divenuta *virtuale*.
    Esso sicuramente è il livello più alto in cui l'elemento *pos_n(i)* è virtuale, poiché soltanto
    un g-nodo *reale* può migrare.
*   Indichiamo con *prev_pos* l'identificativo al livello *prev_lvl* che era dell'identità in
    precedenza.
*   Poni `local_ip_set = null`.
*   Per *i* = `$prev_lvl`,  
    per *j* = `$prev_pos`,  
    *   Indichiamo con *hc* il g-nodo (*i*, *j*).
    *   Calcola `hc_addr` l'indirizzo Netsukuku equivalente *hc* rispetto a *n*.  
        Cioè: `hc_addr = n.slice(i+1); hc_addr.insert_at(0,j)`;
    *   Calcola `dest_ip_set[hc].global = ip_global_gnode(hc_addr)`.
    *   Calcola `dest_ip_set[hc].anonymizing = ip_anonymizing_gnode(hc_addr)`.
    *   Per *k* che scende da *l* - 1 a *i* + 1:
        *   Calcola `dest_ip_set[hc].internal[k] = ip_internal_gnode(hc_addr, inside_level=k)`.
*   Per ogni *hc* in `dest_ip_set.keys`,  
    se `hc.lvl < prev_lvl`:
    *   Rimuovi *hc* dalle possibili destinazioni, cioè esegui `dest_ip_set.unset(hc)`.

### <a name="Utilizzo_locali"></a> Utilizzo indirizzi IP locali

Questo si fa solo con l'identità *principale*.

#### Assegnazione indirizzi locali

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Per ogni livello *k* da 0 a *l* - 1:
    *   Il sistema nel network namespace default si assegna l'indirizzo IP interno al livello *k*.
        Cioè `local_ip_set.internal[k]`.
*   Il sistema nel network namespace default si assegna l'indirizzo IP globale.
    Cioè `local_ip_set.global`.
*   Il sistema può (opzionalmente) assegnarsi il suo indirizzo IP globale anonimizzante.
    Cioè `local_ip_set.anonymizing`.

Quando diciamo che il sistema si assegna un indirizzo IP `$ipaddr` nel network namespace default intendiamo
dire che lo assegna a ognuna delle interfacce di rete che il demone *ntkd* gestisce. Cioè che
esegue per ogni `$dev` il seguente comando:

```
ip address add $ipaddr dev $dev
```

Queste operazioni sono fatte dal programma *ntkd* in determinate circostanze:

*   All'avvio del programma. In questo momento si costituisce la prima identità nel sistema, che è
    in quel momento la principale. Sulla base del suo indirizzo Netsukuku si eseguono tutti i
    comandi `ip address add` come detto prima.
*   Quando l'identità principale si duplica. In questo momento la vecchia identità diventa di connettività
    e il posto di identità principale viene preso dalla nuova identità con un nuovo indirizzo Netsukuku.
    Questo è (potenzialmente) diverso dal precedente a partire da un certo livello: il `host_gnode_level`
    della migrazione (o ingresso) che ha prodotto la duplicazione dell'identità.  
    Sulla base di questa variazione ci sarà una serie di comandi `ip address del` (opportunamente preceduti
    da una serie di comandi `ip route del` nella tabella `ntk` perché in essa sono indicati quegli indirizzi
    locali come `src`) e poi una serie di comandi `ip address add`.
*   Al termine del programma. In questo momento si rimuove la corrente identità principale del sistema.
    Sulla base del suo indirizzo Netsukuku si eseguono tutti i comandi `ip address del` come detto prima.

#### <a name="Source_natting"></a> Regole di Source NATting

Il sistema può (opzionalmente) fare da anonimizzatore. Cioè: quando viene usato per instradare un
pacchetto IP verso un indirizzo IP di tipo anonimizzante, maschera l'indirizzo IP del mittente
con il suo indirizzo IP globale.

Il [source NATting](https://en.wikipedia.org/wiki/Network_address_translation) in un sistema Linux
può essere realizzato istruendo il kernel con il comando `iptables` (utilizzando una regola con
l'estensione `SNAT` nella catena `POSTROUTING` della tabella `nat`). Quando un pacchetto
da inoltrare rientra nei parametri di questa regola (un esempio di parametro che si può usare è
l'indirizzo di destinazione del pacchetto che rientra in un dato range) allora il kernel modifica
l'indirizzo mittente nel pacchetto sostituendolo con uno dei suoi indirizzi pubblici. Inoltre compie
una serie di altre operazioni volte a mantenere inalterate il più possibile le caratteristiche della
comunicazione (ad esempio la connessione stabilita dal protocollo TCP).  
Ad esempio, con il comando «`iptables -t nat -A POSTROUTING -d 10.128.0.0/9 -j SNAT --to 10.1.2.3`»
si ottiene che tutti i pacchetti da inoltrare alle destinazioni 10.128.0.0/9 vanno rimappati al mio indirizzo 10.1.2.3

Il programma *ntkd* all'avvio, opzionalmente, istruisce il kernel per il source natting. Con questa configurazione
il sistema si rende disponibile ad anonimizzare i pacchetti che riceve e che vanno inoltrati verso una
destinazione che accetta richieste anonime.  
L'opzione di rendere anonimi i pacchetti che transitano per il sistema nel percorso verso un'altra destinazione
è distinta e indipendente dall'opzione di accettare richieste anonime.

Il programma ha già calcolato in `local_ip_set.anonymizing_range` lo spazio di indirizzi che indicano una
risorsa da raggiungere in forma anonima. Il programma istruisce il kernel di modificare i pacchetti destinati a quel range
indicando come nuovo indirizzo mittente il suo indirizzo globale. Cioè `local_ip_set.global`. Il comando è il seguente:

```
   iptables -t nat -A POSTROUTING -d $anonymizing_range -j SNAT --to $global
```

Queste operazioni sono fatte dal programma *ntkd* in determinate circostanze:

*   All'avvio del programma. In questo momento si costituisce la prima identità nel sistema, che è
    in quel momento la principale. Sulla base del suo indirizzo Netsukuku si calcola `local_ip_set.global`
    e si esegue il comando `iptables -t nat -A` come detto prima.
*   Quando l'identità principale si duplica. In questo momento la vecchia identità diventa di connettività
    e il posto di identità principale viene preso dalla nuova identità con un nuovo indirizzo Netsukuku.
    Questo è (potenzialmente) diverso dal precedente a partire da un certo livello: il `host_gnode_level`
    della migrazione (o ingresso) che ha prodotto la duplicazione dell'identità.  
    Sulla base di questa variazione ci sarà un comando `iptables -t nat -D` e poi un comando `iptables -t nat -A`.
*   Al termine del programma. In questo momento si rimuove la corrente identità principale del sistema.
    Sulla base del suo indirizzo Netsukuku si esegue il comando `iptables -t nat -D` come detto prima.

#### Mappatura di una sottorete autonoma

Il sistema può fare da gateway verso la rete Netsukuku per una sottorete a gestione autonoma.

Fra le possibilità offerte da `iptables` c'è l'estensione
[NETMAP](https://www.netfilter.org/documentation/HOWTO/netfilter-extensions-HOWTO-4.html#ss4.4)
che ci permette di creare una mappatura 1:1 tra due reti. Cioè di modificare la parte `network` di
un indirizzo IP mantenendo inalterata la parte `host`.  
Abbinata alla catena `PREROUTING` della tabella `nat` questa estensione permette di cambiare l'indirizzo
IP di destinazione di un pacchetto, mentre abbinata alla catena `POSTROUTING` della stessa tabella `nat`
permette di cambiare l'indirizzo del mittente.  
Si può vedere un esempio a questo [link](https://capcorne.wordpress.com/2009/03/24/natting-a-network-range-with-netmapiptables).

Se il sistema vuole fare da gateway per una sottorete a gestione
autonoma, il programma *ntkd* deve assicurarsi che ci siano queste regole:

*   Indichiamo con *subnetlevel* il livello del g-nodo rappresentato dalla sottorete autonoma. Quindi abbiamo *subnetlevel* > 0.
*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *n* l'indirizzo Netsukuku dell'identità principale del sistema.
*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Per *i* che sale da *subnetlevel* a *l* - 2:
    *   Per i pacchetti IP che passano per questo sistema e sono destinati ad
        un indirizzo IP di tipo interno di livello *i* + 1 che identifica un nodo
        interno alla mia sottorete autonoma (di livello *subnetlevel*) e che quindi
        necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
        dell'IP di destinazione affinché risulti nel range degli indirizzi IP di tipo
        interno di livello *subnetlevel*.  
        Cioè: esegue `iptables -t nat -A PREROUTING -d $netmap_range2[i] -j NETMAP --to $netmap_range1`.
    *   Per i pacchetti IP che passano per questo sistema, che hanno per IP di mittente
        un indirizzo IP di tipo interno di livello *subnetlevel* (che cioè
        provengono dall'interno della sottorete autonoma) e che sono destinati ad
        un indirizzo IP di tipo interno di livello *i* + 1, esegui la rimappatura
        dell'IP di mittente affinché risulti nel range degli indirizzi IP
        di tipo interno di livello *i* + 1 e identifichi un nodo
        interno alla mia sottorete autonoma.  
        Cioè: esegue `iptables -t nat -A POSTROUTING -d $netmap_range3[i] -s $netmap_range1 -j NETMAP --to $netmap_range2[i]`.
*   Per i pacchetti IP che passano per questo sistema e sono destinati ad
    un indirizzo IP di tipo globale che identifica un nodo
    interno alla mia sottorete autonoma (di livello *subnetlevel*) e che quindi
    necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
    dell'IP di destinazione affinché risulti nel range degli indirizzi IP
    di tipo interno di livello *subnetlevel*.  
    Cioè: esegue `iptables -t nat -A PREROUTING -d $netmap_range2_upper -j NETMAP --to $netmap_range1`.
*   Per i pacchetti IP che passano per questo sistema, che hanno per IP di mittente
    un indirizzo IP di tipo interno di livello *subnetlevel* (che cioè
    provengono dall'interno della sottorete autonoma) e che sono destinati ad
    un indirizzo IP di tipo globale, esegui la rimappatura
    dell'IP di mittente affinché risulti nel range degli indirizzi IP
    di tipo globale e identifichi un nodo
    interno alla mia sottorete autonoma.  
    Cioè: esegue `iptables -t nat -A POSTROUTING -d $netmap_range3_upper -s $netmap_range1 -j NETMAP --to $netmap_range2_upper`.
*   Se si vuole che ogni sistema nella sottorete autonoma ammetta di essere contattato in forma anonima:
    *   Per i pacchetti IP che passano per questo sistema e sono destinati ad
        un indirizzo IP di tipo anonimizzante che identifica un nodo
        interno alla mia sottorete autonoma (di livello *subnetlevel*) e che quindi
        necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
        dell'IP di destinazione affinché risulti nel range degli indirizzi IP
        di tipo interno di livello *subnetlevel*.  
        Cioè: esegue `iptables -t nat -A PREROUTING -d $netmap_range4 -j NETMAP --to $netmap_range1`.  
        Non è possibile ammettere che qualche nodo sia contattabile in forma anonima
        senza di fatto renderlo possibile a tutti; in quanto l'indirizzo anonimizzante di
        destinazione viene rimappato all'indirizzo interno esattamente come viene
        rimappato l'indirizzo globale.
*   Naturalmente, il gateway permette ai sistemi interni di contattare un sistema esterno in forma anonima. Quindi:
    *   Per i pacchetti IP che passano per questo sistema, che hanno per IP di mittente
        un indirizzo IP di tipo interno di livello *subnetlevel* (che cioè
        provengono dall'interno della sottorete autonoma) e che sono destinati ad
        un indirizzo IP di tipo anonimizzante, esegui la rimappatura
        dell'IP di mittente affinché risulti nel range degli indirizzi IP
        di tipo globale e identifichi un nodo
        interno alla mia sottorete autonoma.  
        Cioè: esegue `iptables -t nat -A POSTROUTING -d $anonymizing_range -s $netmap_range1 -j NETMAP --to $netmap_range2_upper`.

Queste operazioni sono fatte dal programma *ntkd* in determinate circostanze:

*   All'avvio del programma. In questo momento si costituisce la prima identità nel sistema, che è
    in quel momento la principale. Sulla base del suo indirizzo Netsukuku il programma *ntkd* produce i
    comandi `iptables -t nat -A` e `iptables -t nat -D` necessari.
*   Quando l'identità principale si duplica. In questo momento la vecchia identità diventa di connettività
    e il posto di identità principale viene preso dalla nuova identità con un nuovo indirizzo Netsukuku.
    Questo è (potenzialmente) diverso dal precedente a partire da un certo livello: il `host_gnode_level`
    della migrazione (o ingresso) che ha prodotto la duplicazione dell'identità.  
    Sulla base di questa variazione il programma *ntkd* produce i comandi `iptables -t nat -A` e `iptables -t nat -D` necessari.
*   Al termine del programma. In questo momento si rimuove la corrente identità principale del sistema.
    Sulla base del suo indirizzo Netsukuku il programma *ntkd* produce i comandi `iptables -t nat -D` necessari.

### <a name="Utilizzo_destinazioni"></a> Utilizzo indirizzi IP destinazioni

Questo si fa con ogni identità.

#### Rotte nelle tabelle di routing

Il programma *ntkd* deve istruire le policy di routing del sistema (che di norma significa impostare delle
rotte nelle tabelle di routing) in modo da garantire questi comportamenti:

*   Se un processo locale vuole inviare un pacchetto ad un indirizzo IP *x* nello spazio destinato alla rete Netsukuku:
    *   L'indirizzo IP *x* identifica un indirizzo Netsukuku reale *d*. Può trattarsi di un
        indirizzo IP globale, globale anonimizzante o interno.
    *   Se il modulo QSPN non ha alcun percorso verso la destinazione *d*:
        *   Il sistema trova nelle tabelle del kernel una rotta in stato `unreachable` e segnala al processo
            che la destinazione *x* è irraggiungibile.
    *   Altrimenti:
        *   Il sistema trova nelle tabelle del kernel una rotta il cui gateway è costituito dal
            primo hop del miglior percorso (fra tutti quelli che il modulo QSPN ha scoperto) verso
            la destinazione *d*.
*   Se un pacchetto è ricevuto da un vicino *v* ed è destinato ad un indirizzo IP *x* nello spazio
    destinato alla rete Netsukuku che non è un indirizzo IP del sistema:
    *   L'indirizzo IP *x* identifica un indirizzo Netsukuku reale *d*. Può trattarsi di un
        indirizzo IP globale, globale anonimizzante o interno.
    *   Se il modulo QSPN non ha alcun percorso verso la destinazione *d*, tale che non contenga
        fra i suoi hop il *massimo distinto g-nodo* del vicino *v*:
        *   Il sistema trova nelle tabelle del kernel una rotta in stato `unreachable`; quindi scarta il pacchetto
            e invia un pacchetto ICMP «host *x* irraggiungibile» al mittente.
    *   Altrimenti:
        *   Il sistema trova nelle tabelle del kernel una rotta il cui gateway è costituito dal primo
            hop per il miglior percorso verso la destinazione *d*, tale che non contenga il g-nodo del vicino.
        *   Se l'indirizzo IP *x* è globale anonimizzante:
            *   Opzionalmente, il sistema trova una regola che gli impone di mascherare l'indirizzo
                del mittente (cioè di indicare se stesso come mittente e inoltrare le comunicazioni di
                risposta che riceverà al vero mittente).

Inoltre, se il sistema vuole fare da gateway verso una sottorete a gestione autonoma, deve garantire
anche questi comportamenti:

*   Se un pacchetto è ricevuto da un vicino esterno alla sottorete autonoma ed è destinato ad un indirizzo
    IP *x* nello spazio destinato alla sottorete autonoma (può trattarsi di un indirizzo IP globale, globale
    anonimizzante o interno ad un g-nodo di livello superiore al livello del g-nodo che rappresenta la
    dimensione della sottorete autonoma):
    *   Prima di procedere al routing, l'indirizzo di destinazione del pacchetto va mappato all'indirizzo IP
        interno al g-nodo di livello della sottorete autonoma.
    *   In questo modo il pacchetto obbedirà alle regole di routing (per esempio statiche o gestite da
        qualche altro software) che sono di competenza della gestione autonoma della sottorete.
*   Se un pacchetto è ricevuto da un vicino interno alla sottorete autonoma ed è destinato ad un indirizzo
    IP *x* nello spazio destinato alla rete Netsukuku esterna (può trattarsi di un indirizzo IP globale, globale
    anonimizzante o interno ad un g-nodo di livello superiore al livello del g-nodo che rappresenta la
    dimensione della sottorete autonoma):
    *   Dopo aver scelto il gateway per l'inoltro sulla base delle regole di routing, l'indirizzo di mittente
        del pacchetto, che era per l'appunto un indirizzo IP interno al g-nodo di livello della sottorete autonoma,
        va mappato all'indirizzo IP corrispondente a quello di destinazione.
    *   In questo modo il destinatario del pacchetto saprà a chi rispondere.

Operando in questo modo il gateway permette alla sottorete a gestione autonoma di occuparsi solo del routing
per gli indirizzi IP interni al g-nodo di livello della sottorete autonoma stessa. Tutti i nodi della sottorete
autonoma potranno comunque comunicare (sia contattare, sia essere contattati) con tutti gli altri nodi della
rete Netsukuku, purché utilizzino questo sistema come ultimo gateway verso l'esterno.

Abbiamo già visto come si impostano le regole di source natting per il mascheramento dei client
che vogliono restare anonimi (premettendo che il server lo consenta) e quelle di net mapping
per permettere l'accesso di una sottorete a gestione autonoma.

Rimane ora solo l'aspetto del routing.

* * *

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa
il programma *ntkd*.

Abbiamo già avuto modo di evidenziare che un sistema Linux è in grado di replicare il suo
intero network stack in molti distinti network namespace. E che i moduli che stiamo esaminando
assumono come requisito una capacità di questo tipo. Nella trattazione che segue parliamo sempre
di concetti (come le tabelle di routing,
la manipolazione dei pacchetti, ...) che sono da riferirsi ad un particolare network stack.

In un sistema Linux le rotte vengono memorizzate in diverse tabelle. Queste tabelle hanno un
identificativo che è un numero da 0 a 255. Hanno anche un nome descrittivo: l'associazione del
nome al numero (che in realtà è il vero identificativo) è fatta nel file `/etc/iproute2/rt_tables`.

Occorre evidenziare che, in presenza di molteplici network namespace, (di default) il file che
associa il nome mnemonico della tabella al suo numero, `/etc/iproute2/rt_tables`, è comune a tutti
i namespace. Invece le regole di scelta della tabella da esaminare e il contenuto delle tabelle
è distinto in ogni namespace.

Quando un pacchetto va inviato ad una certa destinazione, ci sono delle regole che dicono al sistema su
quali tabelle guardare. Queste regole, visibili con il comando `ip rule list`, di default dicono di
guardare per prima la tabella `local`, per penultima la tabella `main` e per ultima la tabella
`default`. Tra la regola che dice di guardare la `local` e quella che dice di guardare la `main` possono
essere inserite altre regole.

Ogni regola può dire semplicemente di guardare una tabella, oppure di guardarla solo a determinate
condizioni. Una particolare condizione che ci torna utile è questa: «guarda la tabella `XXX` se il
pacchetto da trasmettere è marcato con il numero `YYY`». La marcatura del pacchetto è virtuale, nel
senso che i dati del pacchetto non sono affatto modificati, ma solo il sistema locale lo vede come
marcato; ed è sempre il sistema locale che lo ha precedentemente marcato. Questa marcatura viene
fatta da una parte del kernel che può essere istruita usando l'azione `MARK` del comando `iptables`.

In ogni tabella possono esserci diverse rotte. Ogni rotta ha alcune informazioni importanti:

*   Destinazione. È in formato CIDR. Indica una classe di indirizzi. Solo se il pacchetto è destinato
    a quella classe allora la rotta va presa in considerazione. Se ci sono più rotte nella stessa
    tabella la cui destinazione è una classe che soddisfa il pacchetto, allora si prende quella la
    cui destinazione è la classe più restrittiva.
*   Unreachable. Se presente indica che la destinazione è irraggiungibile.
*   Gateway (gw) e interfaccia (dev). Dice dove trasmettere il pacchetto.
*   Mittente preferito (src). Questa informazione è usata solo per i pacchetti trasmessi dal sistema
    locale, non per i pacchetti da inoltrare. È un indirizzo del sistema locale. Dice quale indirizzo
    locale usare come mittente, se non viene espressamente specificato un indirizzo locale dal processo
    che richiede la trasmissione.

Fatta questa premessa, come si comporta il programma?

Il programma, attraverso i moduli Neighborhood e Identities, ha già automaticamente ottenuto
che nella tabella `main` di ogni network namespace siano memorizzate rotte dirette (cioè senza gateway)
verso ogni suo diretto vicino (più precisamente verso ogni *identità* sua vicina). In queste rotte
sono indicati gli indirizzi di scheda delle proprie interfacce e di quelle dei vicini.

Il programma, solo sul network namespace default, crea una tabella `ntk` con identificativo `YYY`, dove `YYY`
è il primo identificativo libero nel file `/etc/iproute2/rt_tables`. In essa mette tutte le rotte
delle possibili destinazioni IP in base all'indirizzo Netsukuku dell'identità principale.

Il ruolo fondamentale della tabella `ntk` è svolto nel network namespace default. I processi locali
nel sistema che vogliono trasmettere agli altri sistemi nella rete sono serviti da questa tabella.
Quindi in essa il programma mette per ogni destinazione raggiungibile e per ogni indirizzo IP con
cui questa può essere indirizzata (globale, anonimizzante, interni) il gateway per il miglior
percorso scoperto dal modulo Qspn. Per le destinazioni irraggiungibili (sempre per gli indirizzi
IP globale, anonimizzante e interni) mette la rotta come `unreachable`.

La tabella `ntk` serve anche per i pacchetti IP da inoltrare che ci pervengono da altri sistemi
che non fanno *strettamente* parte di Netsukuku. Ci riferiamo in questo senso sia a sistemi che
possono essere NATtati dal sistema corrente, sia a sistemi che appartengono alla sottorete a gestione
autonoma che usa questo sistema come gateway. In entrambi questi casi infatti, i sistemi vicini non hanno
con il sistema corrente un "arco-qspn". Sappiamo anche con certezza che questi sistemi vicini usano
come gateway questo sistema soltanto passando per le sue interfacce di rete che risiedono nel
network namespace default.

Consideriamo ora l'identità principale nel namespace default e ogni altra identità nel relativo
network namespace. Ogni identità ha degli archi-qspn. Per ogni identità nel relativo network
namespace, per ogni suo arco-qspn, il programma crea un'altra tabella chiamata `ntk_from_XXX` con identificativo `YYY`,
dove `XXX` è il MAC address del sistema vicino per questo arco-qspn, `YYY` è il primo identificativo libero nel
file `/etc/iproute2/rt_tables`. Il programma quindi aggiunge una regola nel relativo namespace che
dice di guardare la tabella `ntk_from_XXX` se il pacchetto da trasmettere è marcato con il numero
`YYY`. Inoltre istruisce il kernel, sempre nel relativo namespace, di marcare con il numero `YYY` i pacchetti IP
che hanno `XXX` come MAC di provenienza.

Il ruolo di queste tabelle `ntk_from_XXX` è quello di gestire i pacchetti IP da inoltrare che ci sono
pervenuti attraverso questo arco. Anche in queste tabelle il programma mette tutte le rotte
delle possibili destinazioni IP in base all'indirizzo Netsukuku dell'identità che gestisce quel
namespace. Se il modulo QSPN ha scoperto qualche percorso verso una data destinazione, tale che non contenga
fra i suoi hop il *massimo distinto g-nodo* del vicino collegato a quell'arco, allora il programma
mette su questa tabella il gateway per il miglior percorso tra questi. Altrimenti mette la rotta come `unreachable`.

Sulla base degli eventi segnalati dal modulo QSPN, e se necessario richiamando i suoi metodi pubblici, il
programma *ntkd* popola e mantiene le rotte nelle tabelle `ntk` e `ntk_from_XXX`. I percorsi
segnalati dal modulo QSPN contengono sempre un arco che parte dal sistema corrente come passo iniziale e da tale arco
si può risalire all'indirizzo di scheda del vicino. Le rotte nelle tabelle `ntk` e `ntk_from_XXX` infatti
devono avere come campo gateway (gw) l'indirizzo di scheda del vicino.

Quando il programma ha finito di usare una tabella (ad esempio se un arco che conosceva non è più presente,
oppure se il programma termina) svuota la tabella, poi rimuove la regola, poi rimuove il record
relativo dal file `/etc/iproute2/rt_tables`.

#### Situazione obiettivo di una generica identità

Una generica identità gestisce un network namespace.

La prima identità principale eredita il default network namespace "pulito". Cioè come si trova all'avvio del
programma.

Una nuova identità principale nasce quando la precedente identità principale si duplica. La nuova identità
principale eredita il default network namespace dalla precedente identità principale.

Quando una identità (che era principale o di connettività) diventa di connettività per via di una duplicazione
essa ottiene un nuovo network namespace "vuoto".

Una nuova identità di connettività nasce quando una precedente identità di connettività si duplica. La nuova identità
di connettività eredita un network namespace (non default) dalla precedente identità di connettività.

L'identità principale di un sistema (non le altre) può ricevere un nuovo arco-qspn perché il modulo Neighbohood
ha realizzato un nuovo arco fisico. In questo caso il modulo Identities viene avvisato dal programma *ntkd* e forma
un nuovo arco-identità tra le due identità principali.  
Di seguito alla nascita del nuovo arco-identità segnalata dal modulo Identities, il modulo Hooking viene avvisato e
se le identità appartengono alla stessa rete il modulo Hooking segnala che occorre associare un arco-qspn a
quell'arco-identità. Quindi il programma *ntkd* aggiunge un arco-qspn al modulo QSPN della sua identità interessata;
inoltre deve creare una nuova tabella di routing `ntk_from_xxx` nel network namespace della sua identità interessata
e aggiungervi tutte le rotte possibili come unreachable.

Una identità (principale o di connettività) può ricevere un nuovo arco-qspn perché l'identità di un sistema
vicino con il quale aveva un arco-identità si duplica. In questo caso il modulo Identities aggiunge il
nuovo arco-identità in autonomia.  
Idem come sopra:
Di seguito alla nascita del nuovo arco-identità segnalata dal modulo Identities, il modulo Hooking viene avvisato e
se le identità appartengono alla stessa rete il modulo Hooking segnala che occorre associare un arco-qspn a
quell'arco-identità. Quindi il programma *ntkd* aggiunge un arco-qspn al modulo QSPN della sua identità interessata;
inoltre deve creare una nuova tabella di routing `ntk_from_xxx` nel network namespace della sua identità interessata
e aggiungervi tutte le rotte possibili come unreachable.

Per una identità (principale o di connettività) può succedere che ad un suo arco-identità vengano modificati
il `peer_mac` e il `peer_linklocal` perché l'identità del vicino si duplica. In questo caso il modulo Identities
segnala per prima cosa che il vecchio arco-identità è stato modificato. In seguito, se si verifica, segnala che
un nuovo arco-identità è stato aggiunto (che avrà gli stessi `peer_mac` e `peer_linklocal` che erano prima del
vecchio arco-identità).  
Di seguito alla variazione di `peer_mac` e `peer_linklocal` di un arco-identità segnalata dal modulo Identities,
se l'arco-identità aveva associato a sé un arco-qspn, il programma *ntkd*:

*   Aggiorna tutte le rotte che avevano questo arco-qspn come gateway, perché ora ha un nuovo `peer_linklocal`.
*   Crea una nuova tabella `ntk_from_xxx` nel network namespace di questa identità con il nuovo `peer_mac`. Vi aggiunge
    tutte le rotte come erano prima nella vecchia tabella `ntk_from_yyy` con il vecchio `peer_mac`. Sia quelle
    unreachable che quelle valorizzate.
*   Non sapendo se ci sarà un nuovo arco-identità con il vecchio `peer_mac` e nemmeno se in quel caso ci sarà un
    arco-qspn associato, deve rimuovere la vecchia tabella `ntk_from_yyy` dal network namespace di questa identità.

Per una identità (principale o di connettività) può succedere che un suo arco-identità venga rimosso. ... **TODO**

In seguito, sulla base degli ETP ricevuti il modulo QSPN associato ad una identità aggiorna le conoscenze della
sua mappa ed emette dei segnali. Sulla base di questi segnali l'identità deve aggiornare le rotte nel relativo
network namespace.

Quando una identità muore deve semplicemente rimuovere il relativo network namespace. Fatta eccezione per l'identità
principale che alla sua morte, cioè alla terminazione del programma, deve ripulire il default network namespace.

Mostriamo a quale stato deve essere portato un network namespace da una generica identità.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
    *   Indichiamo con `dest = dest_ip_set[hc]`.
    *   Se si tratta dell'identità principale:
        *   Deve esserci una rotta verso `$dest.global` nella tabella `ntk`.  
            Nella tabella `ntk` nel solo network namespace default ci sono le rotte per i
            pacchetti IP in *partenza*.  
            Se l'identità è a conoscenza di percorsi per una destinazione, allora il programma
            imposta nella tabella il primo gateway del miglior percorso. Altrimenti il programma
            dichiara nella tabella che la destinazione è *non raggiungibile*.  
            Se la destinazione è raggiungibile, in questa tabella va indicato nella rotta l'indirizzo
            preferito come mittente (`src`), che serve se un processo locale vuole inviare un pacchetto
            IP. Si deve usare `local_ip_set.global`.  
            In realtà, la tabella `ntk` nel network namespace default gestito dall'identità principale
            del sistema verrà usata anche per i pacchetti IP in *inoltro* da un MAC address che l'identità
            non conosce. Questo va bene, perché la ricezione di tali pacchetti IP da inoltrare si dovrebbe
            poter verificare solo sul network namespace default e solo se il sistema viene usato consapevolmente
            come gateway: ad esempio se questo sistema è un gateway per una sottorete a gestione autonoma, oppure
            se questo sistema viene usato come NAT per una rete privata.
        *   Analogamente, per l'indirizzo IP anonimizzante,
            deve esserci una rotta verso `$dest.anonymizing` nella tabella `ntk`.  
            Anche in questo caso, se la destinazione è raggiungibile va indicato nella rotta l'indirizzo
            preferito come mittente (`src`). Si deve usare di nuovo `local_ip_set.global`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Deve esserci una rotta verso `$dest.internal[k]` nella tabella `ntk`.  
                Se la destinazione è raggiungibile, in questa tabella va indicato nella rotta l'indirizzo
                preferito come mittente (`src`). Questa volta si deve usare `local_ip_set.internal[k]`.
    *   Per ogni arco-qspn noto al manager di questa identità, indichiamo con *m* il relativo `peer_mac`:
        *   Deve esserci, nel network namespace associato, una rotta verso `$dest.global` nella tabella `ntk_from_$m`.  
            Nella tabella `ntk_from_$m` ci sono le rotte per i pacchetti IP in *inoltro*.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
            La destinazione è *non raggiungibile* per i pacchetti IP
            in *inoltro* provenienti da *m* se non esiste nella rete la destinazione `hc`, oppure se
            l'identità non conosce nessun percorso verso `hc` che non passi per il
            massimo distinto g-nodo di *m* per *n*.  
            In questa tabella non va mai indicato nella rotta l'indirizzo preferito come mittente (`src`).
        *   Analogamente, per l'indirizzo IP anonimizzante,
            deve esserci, nel network namespace associato, una rotta verso `$dest.anonymizing` nella tabella `ntk_from_$m`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Deve esserci, nel network namespace associato, una rotta verso `$dest.internal[k]` nella tabella `ntk_from_$m`.

#### Operazioni della prima identità principale all'avvio

All'avvio del demone *ntkd* viene creata la prima identità principale. Essa forma una nuova rete e
non ha alcun arco-qspn.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *subnetlevel* il livello del g-nodo rappresentato dalla sottorete autonoma.
*   Abbiamo in `devs` l'elenco delle interfacce di rete gestite.
*   Abbiamo in `local_ip_set` gli indirizzi locali computati sulla base del suo primo indirizzo Netsukuku.
*   Abbiamo in `dest_ip_set` gli indirizzi destinazione computati sulla base del suo primo indirizzo Netsukuku.
*   Per ogni `dev` in `devs`:
    *   Per ogni livello *k* da 0 a *l* - 1:
        *   Esegue `ip address add $local_ip_set.internal[k] dev $dev`.
    *   Esegue `ip address add $local_ip_set.global dev $dev`.
    *   Esegue (opzionalmente) `ip address add $local_ip_set.anonymizing dev $dev`.
*   Esegue (opzionalmente) `iptables -t nat -A POSTROUTING -d $local_ip_set.anonymizing_range -j SNAT --to $local_ip_set.global`.
*   Se *subnetlevel* > 0:
    *   Per *i* che sale da *subnetlevel* a *l* - 2:
        *   Esegue `iptables -t nat -A PREROUTING -d $local_ip_set.netmap_range2[i] -j NETMAP --to $local_ip_set.netmap_range1`.
        *   Esegue `iptables -t nat -A POSTROUTING -d $local_ip_set.netmap_range3[i] -s $local_ip_set.netmap_range1 -j NETMAP --to $local_ip_set.netmap_range2[i]`.
    *   Esegue `iptables -t nat -A PREROUTING -d $local_ip_set.netmap_range2_upper -j NETMAP --to $local_ip_set.netmap_range1`.
    *   Esegue `iptables -t nat -A POSTROUTING -d $local_ip_set.netmap_range3_upper -s $local_ip_set.netmap_range1 -j NETMAP --to $local_ip_set.netmap_range2_upper`.
    *   Esegue (opzionalmente) `iptables -t nat -A PREROUTING -d $local_ip_set.netmap_range4 -j NETMAP --to $local_ip_set.netmap_range1`.
    *   Esegue `iptables -t nat -A POSTROUTING -d $local_ip_set.anonymizing_range -s $local_ip_set.netmap_range1 -j NETMAP --to $local_ip_set.netmap_range2_upper`.
*   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
    *   Indichiamo con `dest = dest_ip_set[hc]`.
    *   Esegue `ip route add unreachable $dest.global table ntk`.
    *   Esegue `ip route add unreachable $dest.anonymizing table ntk`.
    *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
        *   Esegue `ip route add unreachable $dest.internal[k] table ntk`.

#### Operazioni di una nuova identità principale alla sua nascita per duplicazione

Una nuova identità principale nasce quando la precedente identità principale si duplica. La nuova identità
principale eredita il default network namespace dalla precedente identità principale.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *subnetlevel* il livello del g-nodo rappresentato dalla sottorete autonoma.
*   Abbiamo in `devs` l'elenco delle interfacce di rete gestite.
*   Indichiamo con `host_gnode_level` il livello del g-nodo esistente in cui la nuova identità
    migra/entra in una diversa rete.
*   Indichiamo con `guest_gnode_level` il livello del g-nodo insieme al quale, in blocco, la nuova identità
    migra/entra in una diversa rete.
*   Abbiamo in `prev_local_ip_set` gli indirizzi locali computati sulla base dell'indirizzo Netsukuku
    della precedente identità.
*   Abbiamo in `prev_dest_ip_set` gli indirizzi destinazione computati sulla base dell'indirizzo Netsukuku
    della precedente identità.
*   Abbiamo in `local_ip_set` gli indirizzi locali computati sulla base dell'indirizzo Netsukuku
    della nuova identità.
*   Abbiamo in `dest_ip_set` gli indirizzi destinazione computati sulla base dell'indirizzo Netsukuku
    della nuova identità.
*   Abbiamo in `prev_peermacs` l'elenco dei MAC address che la vecchia identità aveva come archi-qspn ma
    la nuova identità non li ha più. Può essere perché:  
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità) ma non era nel blocco che è entrato.
*   Abbiamo in `prev_tids` l'elenco dei table-id rispettivi per i MAC address in `prev_peermacs`. Forse non serve.
*   Abbiamo in `new_peermacs` l'elenco dei MAC address che la vecchia identità non aveva come archi-qspn ma
    la nuova identità adesso li ha. Può essere perché:  
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era già nella nuova rete.
        Quindi la mia vecchia identità non aveva un arco-qspn e ora la mia nuova identità ce l'ha.
*   Abbiamo in `new_tids` l'elenco dei table-id rispettivi per i MAC address in `new_peermacs`. Forse non serve.
*   Abbiamo in `both_peermacs` l'elenco dei MAC address che la vecchia identità aveva come archi-qspn e
    la nuova identità anche li ha. Può essere perché:  
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità) ed era nel blocco che è entrato. Quindi
        la nuova identità del vicino (con cui la mia nuova identità avrà un arco) ha lo stesso MAC.
    *   La nuova identità ha fatto una semplice migrazione. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità). Sia che fosse stata nel blocco
        che è migrato, sia che non lo fosse, comunque la mia nuova identità avrà un arco-qspn con lo
        stesso MAC.
*   Per ogni g-nodo `hc` in `prev_dest_ip_set.keys`:
    *   Indichiamo con `prev_dest = prev_dest_ip_set[hc]`.
    *   Esegue `ip route del $prev_dest.global table ntk`.
    *   Esegue `ip route del $prev_dest.anonymizing table ntk`.
    *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1,  
        se `hc.lvl` >= `guest_gnode_level`:
        *   Esegue `ip route del $prev_dest.internal[k] table ntk`.
*   Per ogni `$m` in `both_peermacs` sia `$table` = `ntk_from_$m`:
    *   Per ogni g-nodo `hc` in `prev_dest_ip_set.keys`:
        *   Indichiamo con `prev_dest = prev_dest_ip_set[hc]`.
        *   Esegue `ip route del $prev_dest.global table $table`.
        *   Esegue `ip route del $prev_dest.anonymizing table $table`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1,  
            se `hc.lvl` >= `guest_gnode_level`:
            *   Esegue `ip route del $prev_dest.internal[k] table $table`.
*   Per ogni `$m` in `prev_peermacs` sia `$table` = `ntk_from_$m`, sia `$tid` il relativo table-id:
    *   Esegue `ip route flush table $table`.
    *   Esegue `ip rule del fwmark $tid table $table`.
    *   Esegue `iptables -t mangle -D PREROUTING -m mac --mac-source $m -j MARK --set-mark $tid`.
*   Se *subnetlevel* > 0:
    *   Per ogni livello *k* da `host_gnode_level` a *l* - 2:
        *   Esegue `iptables -t nat -D PREROUTING -d $prev_local_ip_set.netmap_range2[k] -j NETMAP --to $prev_local_ip_set.netmap_range1`.
        *   Esegue `iptables -t nat -D POSTROUTING -d $prev_local_ip_set.netmap_range3[k] -s $prev_local_ip_set.netmap_range1 -j NETMAP --to $prev_local_ip_set.netmap_range2[k]`.
    *   Esegue `iptables -t nat -D PREROUTING -d $prev_local_ip_set.netmap_range2_upper -j NETMAP --to $prev_local_ip_set.netmap_range1`.
    *   Esegue `iptables -t nat -D POSTROUTING -d $prev_local_ip_set.netmap_range3_upper -s $prev_local_ip_set.netmap_range1 -j NETMAP --to $prev_local_ip_set.netmap_range2_upper`.
    *   Esegue (opzionalmente) `iptables -t nat -D PREROUTING -d $prev_local_ip_set.netmap_range4 -j NETMAP --to $prev_local_ip_set.netmap_range1`.
    *   Esegue `iptables -t nat -D POSTROUTING -d $prev_local_ip_set.anonymizing_range -s $prev_local_ip_set.netmap_range1 -j NETMAP --to $prev_local_ip_set.netmap_range2_upper`.
*   Esegue (opzionalmente) `iptables -t nat -D POSTROUTING -d $prev_local_ip_set.anonymizing_range -j SNAT --to $prev_local_ip_set.global`.
*   Per ogni `dev` in `devs`:
    *   Per ogni livello *k* da `host_gnode_level` a *l* - 1:
        *   Esegue `ip address del ${prev_local_ip_set.internal[k]}/32 dev $dev`.
    *   Esegue `ip address del ${prev_local_ip_set.global}/32 dev $dev`.
    *   Esegue (opzionalmente) `ip address del ${prev_local_ip_set.anonymizing}/32 dev $dev`.
*   Per ogni `dev` in `devs`:
    *   Per ogni livello *k* da `host_gnode_level` a *l* - 1:
        *   Esegue `ip address add $local_ip_set.internal[k] dev $dev`.
    *   Esegue `ip address add $local_ip_set.global dev $dev`.
    *   Esegue (opzionalmente) `ip address add $local_ip_set.anonymizing dev $dev`.
*   Esegue (opzionalmente) `iptables -t nat -A POSTROUTING -d $local_ip_set.anonymizing_range -j SNAT --to $local_ip_set.global`.
*   Se *subnetlevel* > 0:
    *   Per ogni livello *k* da `host_gnode_level` a *l* - 2:
        *   Esegue `iptables -t nat -A PREROUTING -d $local_ip_set.netmap_range2[k] -j NETMAP --to $local_ip_set.netmap_range1`.
        *   Esegue `iptables -t nat -A POSTROUTING -d $local_ip_set.netmap_range3[k] -s $local_ip_set.netmap_range1 -j NETMAP --to $local_ip_set.netmap_range2[k]`.
    *   Esegue `iptables -t nat -A PREROUTING -d $local_ip_set.netmap_range2_upper -j NETMAP --to $local_ip_set.netmap_range1`.
    *   Esegue `iptables -t nat -A POSTROUTING -d $local_ip_set.netmap_range3_upper -s $local_ip_set.netmap_range1 -j NETMAP --to $local_ip_set.netmap_range2_upper`.
    *   Esegue (opzionalmente) `iptables -t nat -A PREROUTING -d $local_ip_set.netmap_range4 -j NETMAP --to $local_ip_set.netmap_range1`.
    *   Esegue `iptables -t nat -A POSTROUTING -d $local_ip_set.anonymizing_range -s $local_ip_set.netmap_range1 -j NETMAP --to $local_ip_set.netmap_range2_upper`.
*   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
    *   Indichiamo con `dest = dest_ip_set[hc]`.
    *   Esegue `ip route add unreachable $dest.global table ntk`.
    *   Esegue `ip route add unreachable $dest.anonymizing table ntk`.
    *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1,  
        se `hc.lvl` >= `guest_gnode_level`:
        *   Esegue `ip route add unreachable $dest.internal[k] table ntk`.
*   Per ogni `$m` in `both_peermacs` sia `$table` = `ntk_from_$m`:
    *   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
        *   Indichiamo con `dest = dest_ip_set[hc]`.
        *   Esegue `ip route add unreachable $dest.global table $table`.
        *   Esegue `ip route add unreachable $dest.anonymizing table $table`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1,  
            se `hc.lvl` >= `guest_gnode_level`:
            *   Esegue `ip route add unreachable $dest.internal[k] table $table`.
*   Per ogni `$m` in `new_peermacs` sia `$table` = `ntk_from_$m`, sia `$tid` il relativo table-id:
    *   Esegue `iptables -t mangle -A PREROUTING -m mac --mac-source $m -j MARK --set-mark $tid`.
    *   Esegue `ip rule add fwmark $tid table $table`.
    *   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
        *   Indichiamo con `dest = dest_ip_set[hc]`.
        *   Esegue `ip route add unreachable $dest.global table $table`.
        *   Esegue `ip route add unreachable $dest.anonymizing table $table`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Esegue `ip route add unreachable $dest.internal[k] table $table`.

#### Operazioni di una identità che diventa di connettività e ottiene un nuovo network namespace (vuoto)

Quando una identità (che era principale o di connettività) diventa di connettività per via di una duplicazione
essa ottiene un nuovo network namespace "vuoto".

Ricordiamo che tutte le operazioni eseguite nell'algoritmo qui sotto sono da intendersi eseguite nel
network namespace `$ns` gestito adesso dall'identità. Cioè va premesso `ip netns exec $ns` al comando.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Abbiamo in `dest_ip_set` gli indirizzi destinazione computati sulla base del nuovo indirizzo Netsukuku
    dell'identità, che è virtuale.
*   Abbiamo in `peermacs` l'elenco dei MAC address che l'identità ha adesso come archi-qspn. Può essere che:  
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità) ma non era nel blocco che è entrato.
        In questo caso la vecchia identità che rimane (temporaneamente) nella vecchia rete ha adesso un
        arco-qspn con la stessa identità del vicino che ha lo stesso MAC di prima.
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era già nella nuova rete.
        Quindi la mia vecchia identità non aveva un arco-qspn e ora nemmeno ce l'ha.
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità) ed era nel blocco che è entrato.
        In questo caso la vecchia identità che rimane (temporaneamente) nella vecchia rete ha adesso un
        arco-qspn con la stessa identità del vicino che ha un nuovo MAC.
    *   La nuova identità ha fatto una semplice migrazione. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità) ma non era nel blocco
        che è migrato. In questo caso la vecchia identità che rimane nel vecchio g-nodo ha adesso un
        arco-qspn con la stessa identità del vicino che ha lo stesso MAC di prima.
    *   La nuova identità ha fatto una semplice migrazione. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità) ed era nel blocco
        che è migrato. In questo caso la vecchia identità che rimane nel vecchio g-nodo ha adesso un
        arco-qspn con la nuova identità del vicino che ha un nuovo MAC.
*   Per ogni `$m` in `peermacs` sia `$table` = `ntk_from_$m`, sia `$tid` il relativo table-id:
    *   Esegue `iptables -t mangle -A PREROUTING -m mac --mac-source $m -j MARK --set-mark $tid`.
    *   Esegue `ip rule add fwmark $tid table $table`.
    *   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
        *   Indichiamo con `dest = dest_ip_set[hc]`.
        *   Esegue `ip route add unreachable $dest.global table $table`.
        *   Esegue `ip route add unreachable $dest.anonymizing table $table`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Esegue `ip route add unreachable $dest.internal[k] table $table`.

#### Operazioni di una nuova identità di connettività alla sua nascita per duplicazione

Una nuova identità di connettività nasce quando una precedente identità di connettività si duplica. La nuova identità
di connettività eredita un network namespace (non default) dalla precedente identità di connettività.

Ricordiamo che tutte le operazioni eseguite nell'algoritmo qui sotto sono da intendersi eseguite nel
network namespace `$ns` ereditato dall'identità. Cioè va premesso `ip netns exec $ns` al comando.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con `host_gnode_level` il livello del g-nodo esistente in cui la nuova identità
    migra/entra in una diversa rete.
*   Indichiamo con `guest_gnode_level` il livello del g-nodo insieme al quale, in blocco, la nuova identità
    migra/entra in una diversa rete.
*   Abbiamo in `prev_dest_ip_set` gli indirizzi destinazione computati sulla base dell'indirizzo Netsukuku
    della precedente identità.
*   Abbiamo in `dest_ip_set` gli indirizzi destinazione computati sulla base dell'indirizzo Netsukuku
    della nuova identità.
*   Abbiamo in `prev_peermacs` l'elenco dei MAC address che la vecchia identità aveva come archi-qspn ma
    la nuova identità non li ha più. Può essere perché:  
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità) ma non era nel blocco che è entrato.
*   Abbiamo in `prev_tids` l'elenco dei table-id rispettivi per i MAC address in `prev_peermacs`. Forse non serve.
*   Abbiamo in `new_peermacs` l'elenco dei MAC address che la vecchia identità non aveva come archi-qspn ma
    la nuova identità adesso li ha. Può essere perché:  
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era già nella nuova rete.
        Quindi la mia vecchia identità non aveva un arco-qspn e ora la mia nuova identità ce l'ha.
*   Abbiamo in `new_tids` l'elenco dei table-id rispettivi per i MAC address in `new_peermacs`. Forse non serve.
*   Abbiamo in `both_peermacs` l'elenco dei MAC address che la vecchia identità aveva come archi-qspn e
    la nuova identità anche li ha. Può essere perché:  
    *   La nuova identità ha fatto ingresso in una diversa rete. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità) ed era nel blocco che è entrato. Quindi
        la nuova identità del vicino (con cui la mia nuova identità avrà un arco) ha lo stesso MAC.
    *   La nuova identità ha fatto una semplice migrazione. L'identità del vicino era nella stessa
        rete (quindi aveva un arco-qspn con la mia vecchia identità). Sia che fosse stata nel blocco
        che è migrato, sia che non lo fosse, comunque la mia nuova identità avrà un arco-qspn con lo
        stesso MAC.
*   Per ogni g-nodo `hc` in `prev_dest_ip_set.keys`:
    *   Indichiamo con `prev_dest = prev_dest_ip_set[hc]`.
    *   Esegue `ip route del $prev_dest.global table ntk`.
    *   Esegue `ip route del $prev_dest.anonymizing table ntk`.
    *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1,  
        se `hc.lvl` >= `guest_gnode_level`:
        *   Esegue `ip route del $prev_dest.internal[k] table ntk`.
*   Per ogni `$m` in `both_peermacs` sia `$table` = `ntk_from_$m`:
    *   Per ogni g-nodo `hc` in `prev_dest_ip_set.keys`:
        *   Indichiamo con `prev_dest = prev_dest_ip_set[hc]`.
        *   Esegue `ip route del $prev_dest.global table $table`.
        *   Esegue `ip route del $prev_dest.anonymizing table $table`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1,  
            se `hc.lvl` >= `guest_gnode_level`:
            *   Esegue `ip route del $prev_dest.internal[k] table $table`.
*   Per ogni `$m` in `prev_peermacs` sia `$table` = `ntk_from_$m`, sia `$tid` il relativo table-id:
    *   Esegue `ip route flush table $table`.
    *   Esegue `ip rule del fwmark $tid table $table`.
    *   Esegue `iptables -t mangle -D PREROUTING -m mac --mac-source $m -j MARK --set-mark $tid`.
*   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
    *   Indichiamo con `dest = dest_ip_set[hc]`.
    *   Esegue `ip route add unreachable $dest.global table ntk`.
    *   Esegue `ip route add unreachable $dest.anonymizing table ntk`.
    *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1,  
        se `hc.lvl` >= `guest_gnode_level`:
        *   Esegue `ip route add unreachable $dest.internal[k] table ntk`.
*   Per ogni `$m` in `both_peermacs` sia `$table` = `ntk_from_$m`:
    *   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
        *   Indichiamo con `dest = dest_ip_set[hc]`.
        *   Esegue `ip route add unreachable $dest.global table $table`.
        *   Esegue `ip route add unreachable $dest.anonymizing table $table`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1,  
            se `hc.lvl` >= `guest_gnode_level`:
            *   Esegue `ip route add unreachable $dest.internal[k] table $table`.
*   Per ogni `$m` in `new_peermacs` sia `$table` = `ntk_from_$m`, sia `$tid` il relativo table-id:
    *   Esegue `iptables -t mangle -A PREROUTING -m mac --mac-source $m -j MARK --set-mark $tid`.
    *   Esegue `ip rule add fwmark $tid table $table`.
    *   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
        *   Indichiamo con `dest = dest_ip_set[hc]`.
        *   Esegue `ip route add unreachable $dest.global table $table`.
        *   Esegue `ip route add unreachable $dest.anonymizing table $table`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Esegue `ip route add unreachable $dest.internal[k] table $table`.

#### Operazioni quando il modulo Hooking segnala che occorre associare un arco-qspn a un arco-identità

Il programma *ntkd* può trovarsi a gestire il segnale `add_qspn_arc` dal modulo Hooking associato
all'identità principale o ad una identità di connettività.

Ricordiamo che tutte le operazioni eseguite nell'algoritmo qui sotto sono da intendersi eseguite nel
network namespace `$ns` gestito dall'identità. Cioè, se si tratta di una identità di connettività che quindi
gestisce un namespace diverso dal sefault, va premesso `ip netns exec $ns` al comando.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Abbiamo in `dest_ip_set` gli indirizzi destinazione computati sulla base dell'indirizzo Netsukuku
    dell'identità.
*   Indichiamo con `$m` il `peer_mac` del nuovo arco-qspn.
*   Sia `$table` = `ntk_from_$m`, sia `$tid` il relativo table-id:
*   Esegue `iptables -t mangle -A PREROUTING -m mac --mac-source $m -j MARK --set-mark $tid`.
*   Esegue `ip rule add fwmark $tid table $table`.
*   Per ogni g-nodo `hc` in `dest_ip_set.keys`:
    *   Indichiamo con `dest = dest_ip_set[hc]`.
    *   Esegue `ip route add unreachable $dest.global table $table`.
    *   Esegue `ip route add unreachable $dest.anonymizing table $table`.
    *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
        *   Esegue `ip route add unreachable $dest.internal[k] table $table`.

#### Operazioni quando il modulo QSPN segnala variazioni nella mappa di una identità

Quando il modulo QSPN segnala delle variazioni relative ad un percorso, cioè quando emette uno dei
segnali `path_added`, `path_changed` o `path_removed`, tale segnale prevede una istanza di
`IQspnNodePath p`. Con il codice `HCoord hc = p.i_qspn_get_hops().last().i_qspn_get_hcoord();`
da questa si può ottenere la destinazione interessata sotto forma di coordinate gerarchiche. Al
ricevere uno qualsiasi di questi segnali il programma *ntkd* deve interrogare il modulo QSPN per
vedere quali siano i migliori percorsi per la destinazione `hc` e tenere aggiornate di conseguenza
tutte le tabelle di routing.

Il segnale è emesso da una determinata istanza di QspnManager, cioè è relativo ad una determinata
identità del sistema, che può essere la principale o una di connettività.  
Se si tratta dell'identità principale, allora qualsiasi g-nodo di destinazione sia rappresentato
da `hc` certamente è da indicare nelle tabelle di routing del network namespace default.  
Assumiamo invece che si tratti di una identità di connettività. Indichiamo con `up_to` il livello
più alto in cui l'elemento dell'indirizzo Netsukuku dell'identità è virtuale. (È sicuramente il
livello subito inferiore a `connectivity_from` dell'identità.) Allora gli unici g-nodi di destinazione
da indicare nelle tabelle di routing del network namespace associato all'identità sono quelli
rappresentati da un `hc` tale che `hc.lvl >= up_to`. O, in altre parole, le operazioni nell'algoritmo
che segue sono necessarie solo se `hc` è in `dest_ip_set.keys`.

Ricordiamo che tutte le operazioni eseguite nell'algoritmo qui sotto sono da intendersi eseguite nel
network namespace `$ns` gestito dall'identità. Cioè, se si tratta di una identità di connettività che quindi
gestisce un namespace diverso dal sefault, va premesso `ip netns exec $ns` al comando.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con `qspn_mgr` il modulo QSPN associato a questa identità.
*   Assert `hc` in `dest_ip_set.keys`.
*   Assert `hc.lvl >= subnetlevel`.
*   Indichiamo con `dest = dest_ip_set[hc]`.
*   Se si tratta dell'identità principale:
    *   Computa `IQspnNodePath? best_path = best_path(qspn_mgr, hc)`.  
        La funzione `best_path` trova il miglior percorso per un pacchetto IP generato localmente che vuole
        raggiungere `hc`. Può essere `null` che significa `unreachable`.
    *   Se `best_path == null`:
        *   Esegue `ip route change unreachable $dest.global table ntk`.
        *   Esegue `ip route change unreachable $dest.anonymizing table ntk`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Esegue `ip route change unreachable $dest.internal[k] table ntk`.
    *   Altrimenti:
        *   Computa `IQspnArc gw = best_path.i_qspn_get_arc();`.  
            Si tratta di una istanza di una classe nota al programma *ntkd* dalla quale esso può risalire
            all'indirizzo IP linklocal `gw_ip` e al nome della nostra interfaccia di rete `gw_dev`.
        *   Esegue `ip route change $dest.global via $gw_ip dev $gw_dev table ntk src $local_ip_set.global`.
        *   Esegue `ip route change $dest.anonymizing via $gw_ip dev $gw_dev table ntk src $local_ip_set.global`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Esegue `ip route change $dest.internal[k] via $gw_ip dev $gw_dev table ntk src $local_ip_set.internal[k]`.
*   Per ogni arco-qspn `qspn_arc` noto al manager `qspn_mgr`, indichiamo con *m* il relativo `peer_mac`:
    *   Sia `prev_naddr` l'indirizzo Netsukuku del nodo con cui siamo collegati tramite `qspn_arc`.
    *   Computa `prev_hc` il massimo distinto g-nodo di `prev_naddr` per *n*.
    *   Computa `IQspnNodePath? best_path = best_path_forward(qspn_mgr, hc, prev_hc)`.  
        La funzione `best_path_forward` trova il miglior percorso per un pacchetto IP da inoltrare, il quale
        è stato ricevuto da un vicino e che vuole raggiungere `hc`. Ma di esso sappiamo anche che nel suo
        precedente tragitto è passato per il g-nodo `prev_hc`, per questo vogliamo individuare un percorso
        che non includa di nuovo quel g-nodo. Può essere `null` che significa `unreachable`.
    *   Se `best_path == null`:
        *   Esegue `ip route change unreachable $dest.global table ntk_from_$m`.
        *   Esegue `ip route change unreachable $dest.anonymizing table ntk_from_$m`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Esegue `ip route change unreachable $dest.internal[k] table ntk_from_$m`.
    *   Altrimenti:
        *   Computa `IQspnArc gw = best_path.i_qspn_get_arc();`.  
            Si tratta di una istanza di una classe nota al programma *ntkd* dalla quale esso può risalire
            all'indirizzo IP linklocal `gw_ip` e al nome della nostra interfaccia di rete `gw_dev`.
        *   Esegue `ip route change $dest.global via $gw_ip dev $gw_dev table ntk_from_$m`.
        *   Esegue `ip route change $dest.anonymizing via $gw_ip dev $gw_dev table ntk_from_$m`.
        *   Per *k* che scende da *l* - 1 a *hc.lvl* + 1:
            *   Esegue `ip route change $dest.internal[k] via $gw_ip dev $gw_dev table ntk_from_$m`.

#### Operazioni quando il modulo Identities segnala che un arco-identità esistente è stato modificato.

Il programma *ntkd* può trovarsi a gestire il segnale `identity_arc_changed` dal modulo Identities associato
all'identità principale o ad una identità di connettività.  
Questo segnale individua un particolare arco-identità che collega l'identità interessata ad una
identità in un sistema diretto vicino. Se associato a questo arco-identità il programma aveva creato
un arco-qspn per questa identità, allora questo evento deve essere gestito dal programma *ntkd*
con alcune operazioni.

Ricordiamo che tutte le operazioni eseguite nell'algoritmo qui sotto sono da intendersi eseguite nel
network namespace `$ns` gestito dall'identità. Cioè, se si tratta di una identità di connettività che quindi
gestisce un namespace diverso dal sefault, va premesso `ip netns exec $ns` al comando.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Abbiamo in `dest_ip_set` gli indirizzi destinazione computati sulla base dell'indirizzo Netsukuku
    dell'identità.
*   Indichiamo con `$new_mac` il `peer_mac` nuovo dell'arco-qspn.
*   Indichiamo con `$new_linklocal` il `peer_linklocal` nuovo dell'arco-qspn.
*   Indichiamo con `$prev_mac` il `peer_mac` precedente dell'arco-qspn.
*   Indichiamo con `$prev_linklocal` il `peer_linklocal` precedente dell'arco-qspn.


TODO


Per una identità (principale o di connettività) può succedere che ad un suo arco-identità vengano modificati
il `peer_mac` e il `peer_linklocal` perché l'identità del vicino si duplica. In questo caso il modulo Identities
segnala per prima cosa che il vecchio arco-identità è stato modificato. In seguito, se si verifica, segnala che
un nuovo arco-identità è stato aggiunto (che avrà gli stessi `peer_mac` e `peer_linklocal` che erano prima del
vecchio arco-identità).  
Di seguito alla variazione di `peer_mac` e `peer_linklocal` di un arco-identità segnalata dal modulo Identities,
se l'arco-identità aveva associato a sé un arco-qspn, il programma *ntkd*:

*   Aggiorna tutte le rotte che avevano questo arco-qspn come gateway, perché ora ha un nuovo `peer_linklocal`.
*   Crea una nuova tabella `ntk_from_xxx` nel network namespace di questa identità con il nuovo `peer_mac`. Vi aggiunge
    tutte le rotte come erano prima nella vecchia tabella `ntk_from_yyy` con il vecchio `peer_mac`. Sia quelle
    unreachable che quelle valorizzate.
*   Non sapendo se ci sarà un nuovo arco-identità con il vecchio `peer_mac` e nemmeno se in quel caso ci sarà un
    arco-qspn associato, deve rimuovere la vecchia tabella `ntk_from_yyy` dal network namespace di questa identità.


#### un arco-identità viene rimosso ...

TODO

#### una identità di connettività viene rimossa ...

TODO

#### Operazioni della corrente identità principale all'uscita del programma

TODO

