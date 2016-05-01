# Proof of concept - Indirizzi IP

*   **Nota**  
    Questo documento è di pertinenza dell'utilizzatore del modulo QSPN. Quando il modulo
    QSPN comunica di aver scoperto una nuova destinazione, allora l'utilizzatore è tenuto a
    popolare le tabelle di routing tenendo conto delle convenzioni descritte in questo documento,
    ossia bit di rappresentazione di percorsi interni e bit di richiesta di anonimato.  
    Ci sono riferimenti al presente documento in alcuni punti:

    *   Nell'analisi funzionale del modulo PeerServices si parla di "connessione TCP con un percorso interno".
    *   Nei dettagli tecnici del modulo PeerServices si fa riferimento al "documento livelli e bits".
    *   Nell'allegato RoutingIndirizziInterni all'analisi del modulo QSPN si fa riferimento al "documento livelli e bits".
    *   Nell'analisi funzionale del proof-of-concept si fa riferimento al "documento livelli e bits".

Riserviamo un bit (quello più alto dell'indirizzo) che se è 1 significa che l'indirizzo identifica,
univocamente all'interno di tutta la rete, una destinazione che accetta di essere contattata in forma anonima.

Riserviamo un altro bit (il secondo più alto) che se è 1 significa che l'indirizzo identifica univocamente
una destinazione **non** all'interno di tutta la rete, bensì all'interno di un g-nodo di un determinato
livello *i*, con 0 < *i* < *l*.

Non possono essere impostati a 1 entrambi i suddetti bit.

Alla rappresentazione dell'identificativo nel livello più alto della gerarchia dobbiamo destinare
un numero di bit tale da poter rappresentare tutti i livelli presenti. Cioè il *gsize* del livello
più alto deve essere maggiore o uguale al numero di livelli totale. Questo per poter rappresentare,
come vedremo in seguito, ogni possibile destinazione interna ad un gnodo.

Facciamo l'esempio di una rete IPv4, quindi nella classe 10.0.0.0/8.

Abbiamo 24 bit a disposizione.

Tolto 1 per le rappresentazioni interne e 1 per l'anonimato abbiamo 22 bit.

Facciamo una rete di 4 livelli. Diamo 2 bit al livello 3, 4 bit al livello 2, 8 bit ai livelli 1 e 0.

Il *gsize* del livello più alto soddisfa il vincolo di essere maggiore o uguale al numero di livelli totale, cioè 4. Infatti ha 2 bit.

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

Per ogni destinazione di cui viene a conoscenza il protocollo memorizzerà
un massimo di *k* percorsi disgiunti. Solo il migliore di essi verrà riportato nelle tabelle di
routing del kernel, indicando il solo gateway.

**Nota:** Questo esempio è volutamente semplice. Il numero di possibili indirizzi Netsukuku di
destinazione rimane molto alto. La suddivisione ottimale (che riduce al minimo tale numero) verrà descritta dopo.

Ogni indirizzo Netsukuku di destinazione, cioè ogni g-nodo di livello *i* con 0 ≤ *i* < *l*, verrà
riportato nelle tabelle del kernel come indirizzo IP in 3 forme:

*   (A) Come indirizzo senza anonimato, univoco a livello globale.
*   (B) Come indirizzo senza anonimato, univoco all'interno del gnodo di livello *i* + 1. Questa forma
    non si applicherà alle destinazioni di livello *i* = *l* - 1.
*   (C) Come indirizzo con richiesta di anonimato, univoco a livello globale.

Gli indirizzi (C) hanno, come detto, il bit più alto impostato a 1 e il secondo bit più alto
impostato a 0. Gli altri bit compongono l'indirizzo di una destinazione identificata univocamente
all'interno di tutta la rete.

Gli indirizzi (B) hanno il bit più alto impostato a 0 e il secondo bit più alto impostato a 1. Il
numero codificato nei successivi 2 bit è un intero *k*, con 0 < *k* < *l*. Gli altri bit compongono
l'indirizzo di una destinazione identificata univocamente all'interno del gnodo di livello *k*.

## Esempio

Prendiamo 3 nodi. Il nodo *n* è quello di cui esaminiamo le tabelle di  routing. Il nodo *m* è un
nodo con cui *n* ha un qualche gnodo in comune. Il nodo *o* è un nodo con cui *n* non ha in comune nemmeno
il gnodo più alto. Vediamo quali informazioni mantiene il nodo *n*.

### Nodo *n*

Per il nodo *n* sia l'indirizzo Netsukuku 3·5·241·79.

Computiamo il nostro IP globale e i 3 IP validi internamente ad un gnodo.

```
Globale di *n*:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
   3                   1 1
   5                       0 1 0 1
 241                                 1 1 1 1 0 0 0 1
  79                                                   0 1 0 0 1 1 1 1
 = 10.53.241.79

Interno di *n* nel suo gnodo di livello 3:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        1 1
   5                       0 1 0 1
 241                                 1 1 1 1 0 0 0 1
  79                                                   0 1 0 0 1 1 1 1
 = 10.117.241.79

Interno di *n* nel suo gnodo di livello 2:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        1 0
   N/A                     0 0 0 0
 241                                 1 1 1 1 0 0 0 1
  79                                                   0 1 0 0 1 1 1 1
 = 10.96.241.79

Interno di *n* nel suo gnodo di livello 1:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 1
   N/A                     0 0 0 0
   N/A                               0 0 0 0 0 0 0 0
  79                                                   0 1 0 0 1 1 1 1
 = 10.80.0.79
```

*n* si assegna quindi questi IP:

*   il globale: 10.53.241.79
*   l'interno nel livello 3: 10.117.241.79
*   l'interno nel livello 2: 10.96.241.79
*   l'interno nel livello 1: 10.80.0.79

### Nodo *m*

Per il nodo *m* sia l'indirizzo Netsukuku 3·5·14·204.

Computiamo il suo IP globale e quello interno al gnodo di livello 2 (quello in comune con *n*).

```
Globale di *m*:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
   3                   1 1
   5                       0 1 0 1
  14                                 0 0 0 0 1 1 1 0
 204                                                   1 1 0 0 1 1 0 0
 = 10.53.14.204

Interno di *m* nel suo gnodo di livello 2:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        1 0
   N/A                     0 0 0 0
  14                                 0 0 0 0 1 1 1 0
 204                                                   1 1 0 0 1 1 0 0
 = 10.96.14.204
```

Quando *n* riceve un ETP che contiene il gnodo con *m* esso lo vede come (1,14) cioè come gnodo *g*
di livello 1 appartenente al suo stesso gnodo di livello 2 e con identificativo 14 (a livello 1).

Per il g-nodo *g* l'indirizzo Netsukuku è 3·5·14.

Il nodo *n* computa:

```
Globale di *g*:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[0|0|0|0|0|0|0|0]
   3                   1 1
   5                       0 1 0 1
  14                                 0 0 0 0 1 1 1 0
 = 10.53.14.0/24

Interno di *g* nel suo gnodo di livello 2:
[0|0|0|0|1|0|1|0].[0|1|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[0|0|0|0|0|0|0|0]
  livello gnodo        1 0
   N/A                     0 0 0 0
  14                                 0 0 0 0 1 1 1 0
 = 10.96.14.0/24
```

Quindi *n* imposta:

*   la rotta globale: 10.53.14.0/24 via xx dev yy src 10.53.241.79
*   la rotta interna al g-nodo di livello 2: 10.96.14.0/24 via xx dev yy src 10.96.241.79

Ipotiziamo ora di sfruttare il momento della risoluzione *hostname → IP address* per intervenire sulle
operazioni del nodo *n* quando questo vuole iniziare una connessione (o una trasmissione) con il
nodo *m*. Supponiamo che il nodo *m* abbia registrato per se il nome "morfeo".

Quando il nodo *n* richiede la risoluzione del nome "morfeo.ntk" il resolver cerca nel database
ANDNA il nome "morfeo" e trova l'indirizzo Netsukuku 3·5·14·204.

Invece di computare il relativo indirizzo IP globale 10.53.14.204, il resolver vede rispetto al proprio indirizzo
Netsukuku (quello dell'identità principale) qual'è il minimo comune gnodo (in questo caso 2) e computa
il relativo indirizzo IP interno nel suo gnodo di livello 2: 10.96.14.204.

Questo nella route table corrisponde a 10.96.14.0/24 quindi *n* manda il pacchetto al suo gateway
indicando come proprio IP 10.96.241.79.

Una volta realizzata una connessione TCP con questo indirizzo IP, questa connessione continuerebbe a funzionare anche
se un gnodo di livello superiore migrasse, anche gradualmente un nodo alla volta, ad un altra posizione di pari livello.

Ad esempio se il g-nodo 3·5 migrasse dal g-nodo 3 al g-nodo 1 assumendo in esso l'identificativo 1·2.
Oppure se il g-nodo 3 in blocco facesse ingresso in una diversa rete assumendo in essa l'identificativo (di livello 3) 1.

Va detto che il nodo *n* sarà comunque in grado di inviare pacchetti e/o realizzare una connessione TCP direttamente
con l'indirizzo IP globale se lo conosce, cioè 10.53.14.204, ma in questo caso la connessione si romperebbe durante una tale migrazione.

La risoluzione inversa non subirebbe alterazioni. Quando il nodo *n* vuole sapere il nome dell'host che
ha indirizzo IP 10.96.14.204 (oppure 10.53.14.204) il resolver lo contatta e riceve la lista di nomi
che il nodo *m* ha registrato.

### Nodo *o*
Sia *o* con NIP N[5,2,3,0,1,0,2,3,0].

Computiamo solo il suo IP globale poiché l'unico g-nodo comune con *n* è il 9 (intera rete).

```
IP globale di *o*:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
   5                   0 1 0 1
   2                           1 0
   3                                 1 1
   0                                     0 0
   1                                         0 1
   0                                             0 0
   2                                                   1 0
   3                                                       1 1
   0                                                           0 0 0 0
 = 10.22.196.176
```

Quando *n* riceve un ETP che contiene il g-nodo con *o* esso lo vede come (8,5) cioè come g-nodo di livello 8 con id 5.

Il nodo *n* computa solo:

```
G[5]
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|0|0].[0|0|0|0|0|0|0|0].[0|0|0|0|0|0|0|0]
   5                   0 1 0 1
 = 10.20.0.0/14
```

Quindi *n* imposta solo la rotta globale: 10.20.0.0/14 via xx dev yy src 10.44.141.39

### Richiesta di anonimato
Ogni nodo, per ogni suo indirizzo (quello globale e quelli interni nei vari livelli) si assegna anche un altro indirizzo identico ma con il bit di anonimato impostato.

Per esempio *n* aggiunge

*   al suo globale: 10.44.141.39 => 10.108.141.39
*   al suo interno al livello 8: 10.160.141.39 => 10.224.141.39
*   ...

La stessa duplicazione avviene nelle rotte. Ad esempio alle rotte

*   10.44.142.0/24 via xx dev yy src 10.44.141.39
*   10.144.2.0/24 via xx dev yy src 10.144.1.39

si aggiungono le rotte

*   10.108.142.0/24 via xx dev yy src 10.44.141.39
*   10.224.2.0/24 via xx dev yy src 10.144.1.39

Infine, i nodi che sono disposti a anonimizzare i vicini che ne fanno richiesta, impostano le regole di masquerade del firewall. Precisamente, i pacchetti da inoltrare verso 10.64.0.0/10 e verso 10.192.0.0/10 vanno mascherati (NAT). Naturalmente anche il pacchetto inoltrato avrà codificata la richiesta di essere anonimizzato negli hop successivi.

Quando un nodo vuole raggiungere un certo indirizzo in forma anonima invia i suoi pacchetti all'indirizzo con il bit impostato. Durante il tragitto tutti gli hop disposti a farlo lo anonimizzeranno.

Quest'ultima impostazione (sulle regole di masquerade del firewall) è facoltativa. Se un nodo con poche risorse di memoria, ad esempio, non vuole mascherare i pacchetti che inoltra, comunque l'operazione avrà l'effetto desiderato se qualcuno degli hop durante il percorso sarà disposto a farlo.

## Disposizione ottimale

Per ridurre al minimo il numero massimo di rotte da memorizzare in uno spazio a 24 bit come la classe 10.0.0.0/8 si proceda come segue:

Abbiamo 24 bit a disposizione. Tolto 1 per le rappresentazioni interne e 1 per l'anonimato abbiamo 22 bit. Diamo 4 bit al livello alto; abbiamo così un *gsize* del livello più alto capace di rappresentare fino a 16 livelli. Per sfruttarli tutti facciamo i livelli da 14 a 3 da 1 bit e i livelli 2, 1 e 0 da 2 bit.

Il numero di indirizzi Netsukuku che ogni nodo dovrà al massimo memorizzare come destinazioni nella sua mappa di percorsi è di 1×15 + 12×1 + 3×3 = 36. Resta invariato che il numero massimo di nodi nella rete è 2<sup>22</sup>.

