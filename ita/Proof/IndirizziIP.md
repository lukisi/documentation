# Proof of concept - Indirizzi IP

## Tipi di nodi del grafo

Nel documento di [analisi](AnalisiFunzionale.md) abbiamo precisato che ogni *identità* che vive in
un sistema detiene un indirizzo Netsukuku. Ogni identità è quindi un *nodo del grafo*. Però solo l'identità
*principale* di ogni sistema si assegna degli indirizzi IP legati al suo indirizzo Netsukuku. Infatti
i processi in esecuzione nel sistema che vogliono accedere alla rete (come mittenti o destinatari di pacchetti
IP) sono dentro il solo network namespace default.

L'identità *principale* di un sistema ha sempre un indirizzo Netsukuku completamente *reale*. Quindi essa
sempre si assegna un certo numero di indirizzi IP.

## Calcolo degli indirizzi IP

Questo documento illustra come si calcolano gli indirizzi IP che l'identità *principale* di un sistema vuole assegnarsi.

Spiega anche come si calcolano gli indirizzi IP con notazione CIDR che rappresentano una destinazione per un qualsiasi
*nodo del grafo* che si trova a inviare o inoltrare un pacchetto IP guardando le sue tabelle di routing.

Ricordiamo che nel documento di analisi abbiamo precisato dei vincoli nella
scelta della topologia della rete:

*   La dimensione di ogni g-nodo, detta *gsize*, deve essere una potenza di 2.
*   La somma degli esponenti di tutti i livelli (cioè il numero di bit necessari a codificare
    un indirizzo Netsukuku *reale*) non deve superare *t* - 2, dove *t* è il numero di bit
    a disposizione nella classe di indirizzi IP che si intende destinare alla rete Netsukuku.  
    In altre parole, bisogna lasciare 2 bit liberi nello spazio degli indirizzi IP.  
    Ad esempio, se si destina alla rete Netsukuku la classe 10.0.0.0/8 di IPv4, tale
    somma non deve superare il numero 22.
*   La *gsize* del livello più alto deve essere maggiore o uguale al numero dei livelli.

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

### <a name="Indirizzo_globale_nodo"></a>Indirizzo IP globale di un nodo

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

#### Esempio

Consideriamo una topologia di rete con 4 livelli. Diamo 2 bit al livello 3, 4 bit al livello 2, 8 bit
ai livelli 1 e 0. Sono soddisfatti i due vincoli esposti sopra.

Consideriamo il nodo *n* con indirizzo Netsukuku 3·10·123·45. L'indirizzo IP globale di *n* è 10.58.123.45.

### <a name="Indirizzo_globale_gnodo"></a>Indirizzo IP globale di un g-nodo

Sia *g* l'indirizzo Netsukuku di un g-nodo di livello *i*. L'indirizzo completo sarà
*g<sub>l-1</sub>·...·g<sub>i</sub>*.

Il valore *g<sub>i</sub>* viene riportato nell'indirizzo IP che stiamo componendo partendo dal
bit 𝛴 *<sub>0 ≤ k ≤ i-1</sub>* *g-exp(k)* per un numero di *g-exp*(i) bit. I bit meno significativi
sono messi a 0. Quei bit non saranno comunque presi in considerazione a causa dal prefisso di routing
della notazione CIDR, trattandosi dell'indirizzo IP di un intero g-nodo considerato come una IP subnet.

I valori dei bit più alti si calcolano come visto prima per l'indirizzo di un singolo nodo. Infine si
aggiunge, come accennato, il prefisso di routing, ottenuto come 32 - 𝛴 *<sub>0 ≤ k ≤ i-1</sub>* *g-exp(k)*.

#### Esempio

Consideriamo il nodo *n* di prima nella topologia di rete di prima. Prendiamo in esame un g-nodo che
esso vuole indirizzare, ad esempio *g* = (2, 1) cioè il g-nodo di livello 2 e identificativo 1 che
appartiene al suo stesso g-nodo di livello 3.

L'indirizzo Netsukuku di *g* è 3·1. L'indirizzo IP globale di *g* in notazione CIDR è 10.49.0.0/16.

### <a name="Indirizzo_interno_nodo"></a>Indirizzo IP di un nodo interno ad un suo g-nodo

Sia *n* un nodo con indirizzo *n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*. Sia *g* il suo g-nodo
di livello *i* con 0 ≤ *i* < *l*. Quindi *g* ha indirizzo *n<sub>l-1</sub>·...·n<sub>i</sub>*. Vogliamo
comporre un indirizzo IP di *n* che sia univoco internamente a *g*.

I valori da *n<sub>0</sub>* a *n<sub>i-1</sub>* sono riportati come visto prima nei relativi bit
dell'indirizzo IP che stiamo componendo. Il valore *i* viene riportato nei bit che sarebbero stati
destinati all'identificativo di livello più alto, cioè *n<sub>l-1</sub>*. Gli altri bit, quelli che
avrebbero ospitato gli identificativi da *i* a *l* - 2, sono lasciati a 0.

I due bit più alti li impostiamo a `|0|1|`.

#### Esempio

Consideriamo il nodo *n* di prima nella topologia di rete di prima. Aggiungiamo anche il nodo *m* con
indirizzo Netsukuku 3·10·67·89.

Fin da subito il nodo *n* si è assegnato, oltre all'indirizzo IP globale 10.58.123.45, anche l'indirizzo
IP interno al g-nodo di livello 2, che è 10.96.123.45. Analogamente, il nodo *m* si è assegnato, oltre
all'indirizzo IP globale 10.58.67.89, anche l'indirizzo IP interno al g-nodo di livello 2, che è 10.96.67.89.

### <a name="Indirizzo_interno_gnodo"></a>Indirizzo IP di un g-nodo interno ad un suo g-nodo superiore

Sia *g* un g-nodo di livello *i* con indirizzo *g<sub>l-1</sub>·...·g<sub>i</sub>* con *i* < *l* - 1.
Sia *h* un suo g-nodo superiore di livello *k*. Quindi *h* ha indirizzo *g<sub>l-1</sub>·...·g<sub>k</sub>*, con
*k* > *i*. Vogliamo comporre un indirizzo IP in notazione CIDR di *g* che sia univoco internamente a *h*.

Per ogni valore *t* da *i* a *k* - 1, il valore di *g<sub>t</sub>* è riportato come visto prima nei relativi
bit dell'indirizzo IP che stiamo componendo. Il valore *k* viene riportato nei bit che sarebbero stati
destinati all'identificativo di livello più alto, cioè *g<sub>l-1</sub>*. Gli altri bit sono lasciati a 0.

I due bit più alti li impostiamo a `|0|1|`.

#### Esempio

Consideriamo i nodi *n* e *m* di prima. Dal punto di vista di *n*, il nodo *m* si trova nel g-nodo
*g* 3·10·67.

Il nodo *n* ha in comune con *g* il g-nodo direttamente superiore *h* 3·10. Il g-nodo *g* all'interno di
*h* viene individuato con l'indirizzo IP 10.96.67.0/24. Quindi *n* imposta nelle tabelle di routing una rotta
per 10.96.67.0/24 a causa del percorso che gli è noto verso la destinazione *g*.

Inoltre il nodo *n* ha in comune con *g* il g-nodo *h'* 3. Il g-nodo *g* all'interno di
*h'* viene individuato con l'indirizzo IP 10.122.67.0/24. Quindi *n* imposta nelle tabelle di routing anche una rotta
per 10.122.67.0/24 a causa del medesimo percorso verso *g*.

### <a name="Indirizzo_anonimizzante"></a>Indirizzo IP di un nodo o g-nodo contattabile in forma anonima

Sia *n* un nodo con indirizzo *n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*. Vogliamo comporre
un indirizzo IP per tale risorsa tale che un client lo possa usare per contattare il nodo mantenendo
l'anonimato. Chiamiamo un tale indirizzo *anonimizzante*.

I valori da *n<sub>0</sub>* a *n<sub>l-1</sub>* sono riportati come visto prima nei relativi bit
dell'indirizzo IP che stiamo componendo.

I due bit più alti li impostiamo a `|1|0|`.

Nel caso di un g-nodo, per produrre un indirizzo anonimizzante in notazione CIDR si procede in modo
analogo, aggiungendo il prefisso come visto prima.

#### Esempio

Consideriamo il nodo *n* di prima. L'indirizzo IP globale anonimizzante di *n* è 10.186.123.45.

Se il nodo *n* ammette la possibilità di venire contattato in forma anonima (questa è una sua scelta) si
assegna anche questo indirizzo.

Inoltre, siccome il nodo *n* conosce un percorso per il g-nodo *g* 3·10·67, imposta nelle tabelle di
routing anche una rotta per 10.186.67.0/24. Questo lo fa indipendentemente dal fatto che si sia
assegnato o meno il suo indirizzo anonimizzante. In ogni caso, quando il nodo *n* usa la rotta verso
10.186.67.0/24, usa come indirizzo IP *src* il suo indirizzo globale 10.58.123.45.

## Dimensione massima della mappa di un nodo del grafo

Prendiamo in esame l'esempio di topologia usato sopra. Cioè una rete con 4 livelli. In essa abbiamo
2 bit al livello 3, 4 bit al livello 2, 8 bit ai livelli 1 e 0.

Gli indirizzi validi sono 2<sup>22</sup>, cioè circa 4 milioni.

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

**Nota:** La topologia portata ad esempio qui sopra era volutamente semplice. Il numero di possibili indirizzi Netsukuku di
destinazione rimane molto alto. La suddivisione ottimale (che riduce al minimo tale numero) verrà descritta dopo.

## Esempio

Abbiamo già descritto nel documento di analisi quali sono i tipi di indirizzo IP che un
*nodo del grafo* assegna a se stesso sulla base del suo indirizzo Netsukuku. Abbiamo anche visto quali
indirizzi IP vengono aggiunti come destinazioni nelle tabelle di routing del kernel sulla
base delle conoscenze nella sua mappa.

Riportiamo un esempio di questi calcoli.

Prendiamo 3 nodi. Il nodo *n* è quello di cui esaminiamo le tabelle di  routing. Il nodo *m* è un
nodo con cui *n* ha un qualche g-nodo in comune. Il nodo *o* è un nodo con cui *n* non ha in comune nemmeno
il g-nodo più alto. Vediamo quali informazioni mantiene il nodo *n*.

### Nodo *n*

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

### Nodo *m*

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

### Nodo *o*

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

### Richiesta di anonimato

Un nodo, opzionalmente, può dichiararsi disposto ad accettare richieste in forma anonima. Se intende
farlo, il nodo si assegna anche un altro indirizzo globale, identico al primo, ma con il bit *anonimizzante* impostato.

Ogni nodo, invece, per ogni destinazione di cui viene a conoscenza, aggiunge una rotta verso
l'indirizzo IP *anonimizzante* relativo.

Per esempio il nodo *n*:

*   Opzionalmente, si assegna l'indirizzo IP `10.186.123.45`.
*   Aggiunge la rotta `10.186.67.0/24 src 10.58.123.45`.
*   Aggiunge la rotta `10.160.0.0/12 src 10.58.123.45`.

Infine, i nodi che sono disposti a anonimizzare i vicini che ne fanno richiesta, impostano le regole di
masquerade del firewall come descritto nel documento di analisi.

## Disposizione ottimale

Per ridurre al minimo il numero massimo di rotte da memorizzare in uno spazio a 24 bit come
la classe 10.0.0.0/8 si proceda come illustrato di seguito.

Abbiamo 24 bit a disposizione. Tolti 2 per le codifiche viste nel presente documento, abbiamo 22 bit.
Diamo 4 bit al livello alto; abbiamo così un *gsize* del livello più alto capace di rappresentare fino
a 16 livelli. Per sfruttarli tutti facciamo i livelli da 14 a 3 da 1 bit e i livelli 2, 1 e 0 da 2 bit.

Il numero di indirizzi Netsukuku che ogni nodo dovrà al massimo memorizzare come destinazioni nella
sua mappa di percorsi è di 1×15 + 12×1 + 3×3 = 36. Resta invariato che il numero massimo di nodi nella rete è 2<sup>22</sup>.

