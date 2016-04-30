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

Riserviamo un bit (quello più alto dell'indirizzo) che se è 1 significa che l'indirizzo identifica, univocamente all'interno di tutta la rete, una destinazione che accetta di essere contattata in forma anonima.

Riserviamo un altro bit (il secondo più alto) che se è 1 significa che l'indirizzo identifica univocamente una destinazione **non** all'interno di tutta la rete, bensì all'interno di un g-nodo di un determinato livello *i*, con 0 < *i* < *l*.

Non possono essere impostati a 1 entrambi i suddetti bit.

Alla rappresentazione dell'identificativo nel livello più alto della gerarchia dobbiamo destinare un numero di bit tale da poter rappresentare tutti i livelli presenti. Cioè il *gsize* del livello più alto deve essere maggiore o uguale al numero di livelli totale. Questo per poter rappresentare, come vedremo in seguito, ogni possibile destinazione interna ad un gnodo.

Facciamo l'esempio di una rete IPv4, quindi nella classe 10.0.0.0/8.

Abbiamo 24 bit a disposizione.

Tolto 1 per le rappresentazioni interne e 1 per l'anonimato abbiamo 22 bit.

Facciamo una rete di 4 livelli. Diamo 2 bit al livello 3, 4 bit al livello 2, 8 bit ai livelli 1 e 0.

Il *gsize* del livello più alto soddisfa il vincolo di essere maggiore o uguale al numero di livelli totale, cioè 4. Infatti ha 2 bit.

Gli indirizzi validi sono 2<sup>22</sup>, cioè circa 4 milioni.

Il numero di indirizzi Netsukuku che ogni nodo dovrà al massimo memorizzare come destinazioni nella sua mappa di percorsi è di 1×3 + 1×15 + 2×255 = 528. Per ognuno il protocollo memorizzerà un massimo di *k* percorsi disgiunti. Solo il migliore di essi verrà riportato nelle tabelle di routing del kernel, indicando il solo gateway.

**Nota:** Questo esempio è volutamente semplice. Il numero di possibili indirizzi Netsukuku di destinazione rimane molto alto. La suddivisione ottimale (che riduce al minimo tale numero) verrà descritta dopo.

Ogni indirizzo Netsukuku di destinazione, cioè ogni g-nodo di livello *i* con 0 ≤ *i* < *l*, verrà riportato nelle tabelle del kernel come indirizzo IP in 3 forme:
*   (A) Come indirizzo senza anonimato, univoco a livello globale.
*   (B) Come indirizzo senza anonimato, univoco all'interno del gnodo di livello *i* + 1. Questa forma non si applicherà alle destinazioni di livello *i* = *l* - 1.
*   (C) Come indirizzo con richiesta di anonimato, univoco a livello globale.

Gli indirizzi (C) hanno, come detto, il bit più alto impostato a 1 e il secondo bit più alto impostato a 0. Gli altri bit compongono l'indirizzo di una destinazione identificata univocamente all'interno di tutta la rete.

Gli indirizzi (B) hanno il bit più alto impostato a 0 e il secondo bit più alto impostato a 1. Il numero codificato nei successivi 2 bit è un intero *k*, con 0 < *k* < *l*. Gli altri bit compongono l'indirizzo di una destinazione identificata univocamente all'interno del gnodo di livello *k*.

**Nota:** Gli esempi di calcolo mostrati sotto vanno corretti.

## Esempio

Prendiamo 3 nodi. N1 è il nostro nodo, quello di cui esaminiamo le tabelle di  routing. N2 è un nodo con cui abbiamo un qualche gnodo in comune. N3 è un nodo con cui non abbiamo in comune nemmeno il gnodo più alto. Vediamo quali informazioni mantiene il nodo N1.

### Nodo N1

Sia N1 con NIP N[11,0,2,0,3,1,0,2,7].

Computiamo il nostro IP globale e i 8 IP validi internamente ad un gnodo.

```
Globale di N1:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  11                   1 0 1 1
   0                           0 0
   2                                 1 0
   0                                     0 0
   3                                         1 1
   1                                             0 1
   0                                                   0 0
   2                                                       1 0
   7                                                           0 1 1 1
 = 10.44.141.39

Interno di N1 nel suo gnodo di livello 8:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        1 0 0 0
   0                           0 0
   2                                 1 0
   0                                     0 0
   3                                         1 1
   1                                             0 1
   0                                                   0 0
   2                                                       1 0
   7                                                           0 1 1 1
 = 10.160.141.39

Interno di N1 nel suo gnodo di livello 7:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 1 1 1
   N/A                         0 0
   2                                 1 0
   0                                     0 0
   3                                         1 1
   1                                             0 1
   0                                                   0 0
   2                                                       1 0
   7                                                           0 1 1 1
 = 10.156.141.39

Interno di N1 nel suo gnodo di livello 6:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 1 1 0
   N/A                         0 0
   N/A                               0 0
   0                                     0 0
   3                                         1 1
   1                                             0 1
   0                                                   0 0
   2                                                       1 0
   7                                                           0 1 1 1
 = 10.152.13.39

Interno di N1 nel suo gnodo di livello 5:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 1 0 1
   N/A                         0 0
   N/A                               0 0
   N/A                                   0 0
   3                                         1 1
   1                                             0 1
   0                                                   0 0
   2                                                       1 0
   7                                                           0 1 1 1
 = 10.148.13.39

Interno di N1 nel suo gnodo di livello 4:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 1 0 0
   N/A                         0 0
   N/A                               0 0
   N/A                                   0 0
   N/A                                       0 0
   1                                             0 1
   0                                                   0 0
   2                                                       1 0
   7                                                           0 1 1 1
 = 10.144.1.39

Interno di N1 nel suo gnodo di livello 3:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 0 1 1
   N/A                         0 0
   N/A                               0 0
   N/A                                   0 0
   N/A                                       0 0
   N/A                                           0 0
   0                                                   0 0
   2                                                       1 0
   7                                                           0 1 1 1
 = 10.140.0.39

Interno di N1 nel suo gnodo di livello 2:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 0 1 0
   N/A                         0 0
   N/A                               0 0
   N/A                                   0 0
   N/A                                       0 0
   N/A                                           0 0
   N/A                                                 0 0
   2                                                       1 0
   7                                                           0 1 1 1
 = 10.136.0.39

Interno di N1 nel suo gnodo di livello 1:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 0 0 1
   N/A                         0 0
   N/A                               0 0
   N/A                                   0 0
   N/A                                       0 0
   N/A                                           0 0
   N/A                                                 0 0
   N/A                                                     0 0
   7                                                           0 1 1 1
 = 10.132.0.7
```

N1 si assegna quindi questi IP:

*   il globale: 10.44.141.39
*   l'interno nel livello 8: 10.160.141.39
*   l'interno nel livello 7: 10.156.141.39
*   l'interno nel livello 6: 10.152.13.39
*   l'interno nel livello 5: 10.148.13.39
*   l'interno nel livello 4: 10.144.1.39
*   l'interno nel livello 3: 10.140.0.39
*   l'interno nel livello 2: 10.136.0.39
*   l'interno nel livello 1: 10.132.0.7

### Nodo N2
Sia N2 con NIP N[11,0,2,0,3,2,0,3,10].

Computiamo il suo IP globale e quello interno al gnodo di livello 4 (quello in comune con N1).

```
Globale di N2:
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  11                   1 0 1 1
   0                           0 0
   2                                 1 0
   0                                     0 0
   3                                         1 1
   2                                             1 0
   0                                                   0 0
   3                                                       1 1
  10                                                           1 0 1 0
 = 10.44.142.58

Interno di N2 nel suo gnodo di livello 4:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[?|?|?|?|?|?|?|?]
  livello gnodo        0 1 0 0
   N/A                         0 0
   N/A                               0 0
   N/A                                   0 0
   N/A                                       0 0
   2                                             1 0
   0                                                   0 0
   3                                                       1 1
  10                                                           1 0 1 0
 = 10.144.2.58
```

Quando N1 riceve un ETP che contiene il gnodo con N2 esso lo vede come (3,2) cioè come gnodo di livello 3 appartenente al suo stesso gnodo di livello 4 e con identificativo (a livello 3) = 2.

Il nodo N1 computa:

```
G[11,0,2,0,3,2]
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[0|0|0|0|0|0|0|0]
  11                   1 0 1 1
   0                           0 0
   2                                 1 0
   0                                     0 0
   3                                         1 1
   2                                             1 0
 = 10.44.142.0/24

Interno di G[11,0,2,0,3,2] nel suo gnodo di livello 4:
[0|0|0|0|1|0|1|0].[1|0|?|?|?|?|?|?].[?|?|?|?|?|?|?|?].[0|0|0|0|0|0|0|0]
  livello gnodo        0 1 0 0
   N/A                         0 0
   N/A                               0 0
   N/A                                   0 0
   N/A                                       0 0
   2                                             1 0
 = 10.144.2.0/24
```

Quindi N1 imposta:

*   la rotta globale: 10.44.142.0/24 via xx dev yy src 10.44.141.39
*   la rotta interna al g-nodo di livello 4: 10.144.2.0/24 via xx dev yy src 10.144.1.39

Quando viene richiesto l'indirizzo di n2.ntk il resolver cerca nel db andna il nome n2 e trova il NIP N[11,0,2,0,3,2,0,3,10].

Invece di trasformarlo direttamente in 10.44.142.58, il resolver vede rispetto al proprio NIP (primario) quale è il minimo comune gnodo (in questo caso 4) e computa il relativo interno nel suo gnodo di livello 4: 10.144.2.58.

Questo nella route table corrisponde a 10.144.2.0/24 quindi N1 manda il pacchetto al suo gateway indicando come proprio IP 10.144.1.39.

Una volta realizzata una connessione TCP con questo IP, questa connessione continuerà a funzionare anche se un gnodo di livello superiore migra, anche gradualmente un nodo alla volta, ad un altra posizione di pari livello.

*   **Appunto**: mentre un g-nodo migra un nodo alla volta non si riscontrano problemi nel funzionamento del QSPN. Possono esserci problemi temporanei nei servizi peer-to-peer perché essi dipendono dal completo indirizzo di un nodo all'interno dell'intera rete e non all'interno di un g-nodo. Questi temporanei malfunzionamenti non sono gravi ad esempio nel servizio di risoluzione nomi. Potrebbe essere più insidioso il servizio Coordinator, ad esempio se un nodo nuovo vuole entrare nel g-nodo mentre questo sta migrando.  
Ma possiamo correggere il funzionamento del Coordinator per renderlo invulnerabile: basta che l'oggetto chiave di questo servizio non è più l'indirizzo del g-nodo da coordinare ma solo il suo livello. Questo cambio si può fare per il semplice motivo che il percorso di un messaggio verso il Coordinator di un certo g-nodo è sempre completamente interno al g-nodo stesso.  
Fatto questo cambio otteniamo che quando il messaggio viene inoltrato da un nodo che ha migrato ad un nodo che non ha ancora migrato il significato della chiave non cambia.

Va detto che il nodo N1 sarà comunque in grado di inviare pacchetti e/o realizzare una connessione TCP direttamente con l'IP globale se lo conosce, cioè 10.44.142.58, ma in questo caso la connessione si romperebbe durante una tale migrazione.

Quando viene richiesto il nome di 10.144.2.58 il resolver lo contatta come al solito e riceve la lista di nomi [n2] a cui aggiunge ".NTK".

### Nodo N3
Sia N3 con NIP N[5,2,3,0,1,0,2,3,0].

Computiamo solo il suo IP globale poiché l'unico g-nodo comune con N1 è il 9 (intera rete).

```
IP globale di N3:
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

Quando N1 riceve un ETP che contiene il g-nodo con N3 esso lo vede come (8,5) cioè come g-nodo di livello 8 con id 5.

Il nodo N1 computa solo:

```
G[5]
[0|0|0|0|1|0|1|0].[0|0|?|?|?|?|0|0].[0|0|0|0|0|0|0|0].[0|0|0|0|0|0|0|0]
   5                   0 1 0 1
 = 10.20.0.0/14
```

Quindi N1 imposta solo la rotta globale: 10.20.0.0/14 via xx dev yy src 10.44.141.39

### Richiesta di anonimato
Ogni nodo, per ogni suo indirizzo (quello globale e quelli interni nei vari livelli) si assegna anche un altro indirizzo identico ma con il bit di anonimato impostato.

Per esempio N1 aggiunge

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

Per ridurre al massimo il numero di rotte da memorizzare in uno spazio a 24 bit come la classe 10.0.0.0/8 si proceda come segue:

Abbiamo 24 bit a disposizione. Tolto 1 per le rappresentazioni interne e 1 per l'anonimato abbiamo 22 bit. Diamo 4 bit al livello alto; abbiamo così un *gsize* del livello più alto capace di rappresentare fino a 16 livelli. Per sfruttarli tutti facciamo i livelli da 14 a 3 da 1 bit e i livelli 2, 1 e 0 da 2 bit.

Il numero di indirizzi Netsukuku che ogni nodo dovrà al massimo memorizzare come destinazioni nella sua mappa di percorsi è di 1×15 + 12×1 + 3×3 = 36. Resta invariato che il numero massimo di nodi nella rete è 2<sup>22</sup>.

