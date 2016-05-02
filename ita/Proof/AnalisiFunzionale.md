# Proof of concept - Analisi Funzionale

1.  [Ruolo del qspnclient](#Ruolo_del_qspnclient)
    1.  [Interazione programma-utente](#Interazione_programma_utente)
1.  [Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IPv4](#Mappatura_indirizzi_ip)
    1.  [Indirizzo IP globale di un nodo](#Indirizzo_globale_nodo)
    1.  [Indirizzo IP globale di un g-nodo](#Indirizzo_globale_gnodo)
    1.  [Indirizzo IP di un nodo interno ad un suo g-nodo](#Indirizzo_interno_nodo)
    1.  [Indirizzo IP di un g-nodo interno al suo g-nodo direttamente superiore](#Indirizzo_interno_gnodo)
    1.  [Indirizzo IP di un nodo o g-nodo contattabile in forma anonima](#Indirizzo_anonimizzante)
1.  [Identità](#Identita)
    1.  [Identità principale](#Identita_principale)
    1.  [Identità di connettività](#Identita_di_connettivita)
1.  [Indirizzi del nodo](#Indirizzi_del_nodo)
1.  [Rotte nelle tabelle di routing](#Rotte_nelle_tabelle_di_routing)
    1.  [Source NATting](#Source_natting)
    1.  [Routing](#Routing)

Ci proponiamo di realizzare un programma, **qspnclient**, che si avvale del modulo QSPN e altri
moduli a sostegno (Neighborhood, Identities) per stabilire come impostare le rotte nelle tabelle
di routing del kernel di una macchina. Si tratta di un programma specifico per un sistema Linux.

## <a name="Ruolo_del_qspnclient"/>Ruolo del qspnclient

Questo programma permette all'utente di fare le veci del demone *ntkd*, simulando le sue operazioni
e quelle di pertinenza di altri moduli:

*   Il dialogo con vicini appartenenti ad altre reti.
*   Il coordinamento di un g-nodo (moduli PeerServices e Coordinator).
*   La strategia di ingresso in una rete, cioè la scelta del g-nodo *g* in cui chiedere un posto.
*   La ricerca della più breve *migration path* per liberare un posto in *g* (modulo Migrations).
*   La comunicazione delle informazioni a tutti i nodi interessati dalla migration path trovata.

Sulla base dei comandi dati sulla console dall'utente, il programma *qspnclient* interagisce coi
moduli suddetti (QSPN, Neighborhood, Identities) e sulla base delle loro elaborazioni interviene
sulle configurazioni di rete del sistema. L'utente sarà quindi in grado di verificare che il
sistema riesca effettivamente a stabilire connessioni con gli altri nodi della rete, che le rotte
siano quelle che ci si attende, eccetera. Inoltre il programma interattivamente consente di chiedere
al modulo QSPN le informazioni che ha raccolto e mostrarle all'utente.

### <a name="Interazione_programma_utente"/>Interazione programma-utente

Il programma *qspnclient* prevede che l'utente immetta, come argomenti della riga di comando e in modo
interattivo dalla console durante la sua esecuzione, tutti i requisiti del modulo.

Ai parametri che saranno individuati in modo autonomo dai moduli (ad esempio gli identificativi del
nodo, gli indirizzi di scheda, ...) verranno associati degli indici progressivi che saranno visualizzati
all'utente. L'utente si riferirà ad essi tramite questi indici. Questo per rendere più facilmente
riproducibili gli ambienti di test.

* * *

All'avvio del nodo viene creata l'istanza di NeighborhoodManager e gli sono passati i nomi delle
interfacce di rete che dovrà gestire. Ad ogni interfaccia, il modulo Neighborhood associa un
indirizzo IP link-local scelto a caso. In realtà l'assegnazione dell'indirizzo è fatta proprio
dal programma attraverso una classe che implementa l'interfaccia INeighborhoodIPRouteManager passata
al modulo Neighborhood. Quello che fa il modulo Neighborhood è scegliere l'indirizzo. Quindi il
programma si avvede dell'indirizzo scelto proprio perché viene chiamato il metodo *add_address* di
INeighborhoodIPRouteManager. Il programma associa questo proprio indirizzo link-local all'indice
autoincrementante *linklocal_nextindex*, che parte da 0.

* * *

L'istanza di NeighborhoodManager deve essere istruita sul numero massimo di archi che può accettare.
Il programma *qspnclient* specifica un numero elevato. Nonostante questo, ogni arco che il modulo
Neighborhood realizza passa al vaglio dell'utente che decide se utilizzarlo o meno. Questo permette
di dirigere il proprio ambiente di test a piacimento anche in particolari scenari, come ad esempio
un gruppo di macchine virtuali che condividono un unico dominio di broadcast ma vogliono simulare un
gruppo di nodi wireless posizionati secondo una determinata disposizione.

Per ogni arco che il modulo Neighborhood realizza, le informazioni a disposizione (i due link-local e
i due MAC address) sono visualizzate all'utente. Soltanto agli archi che l'utente decide di accettare
e nell'ordine in cui sono accettati, il programma associa un indice autoincrementante *nodearc_nextindex*,
che parte da 0. In seguito il programma sfrutta questi archi passandoli al modulo Identities.

Sempre per dare all'utente il maggior controllo possibile sulle dinamiche del test, anche il costo
di un arco che viene rilevato dal modulo Neighborhood non è lo stesso che viene usato dal programma
*qspnclient*. L'utente quando accetta un arco dice quale costo gli vuole associare. In seguito può
variarlo a piacimento fino anche a simularne la rimozione.

Quindi gli effettivi segnali di *arc_changed* del modulo Neighborhood sono in realtà ignorati dal programma.

* * *

All'avvio del nodo viene creata l'istanza di IdentityManager e questo crea un NodeID casuale. Il programma
recupera tale NodeID col metodo *get_main_id()* e lo associa all'indice autoincrementante *nodeid_nextindex*,
che parte da 0. In seguito il programma quando crea una nuova identità col metodo *add_identity* associa la
nuova istanza di NodeID al prossimo valore di *nodeid_nextindex*. Quindi l'utente può usare questo indice per
dare dalla console comandi concernenti una certa identità.

* * *

Alla creazione di una nuova identità, il modulo Identities crea un nuovo network namespace. In realtà la creazione
del network namespace è fatta proprio dal programma attraverso una classe che implementa l'interfaccia
IIdmgmtNetnsManager passata al modulo Identities. Quello che fa il modulo Identities è scegliere il nome di
tale namespace. Quindi il programma si avvede del nome scelto proprio perché viene chiamato il metodo
*create_namespace* di IIdmgmtNetnsManager.

Il nome di tale namespace è già costruito con un indice progressivo, quindi il programma non gli associa un
indice ma semplicemente lo mostra all'utente. Questi potrà usare direttamente il nome del namespace nei
comandi che dà sulla console.

* * *

Sempre alla creazione di una nuova identità, il modulo Identities, di nuovo con chiamate all'interfaccia
IIdmgmtNetnsManager, crea per ogni interfaccia di rete reale un pseudo-interfaccia. Anche qui il nome
della pseudo-interfaccia non è casuale ma generato con lo stesso indice progressivo usato per il
network namespace, quindi il programma lo mostra semplicemente all'utente.

Invece il modulo Identities genera casualmente un indirizzo IP link-local e lo associa alla pseudo-interfaccia
col metodo *add_address* di IIdmgmtNetnsManager. Il programma vede tale indirizzo e lo associa all'indice
autoincrementante *linklocal_nextindex*, che avevamo introdotto prima.

* * *

Quando al modulo Identities viene comunicato un arco, esso vi costruisce sopra automaticamente un
*arco-identità*. Al modulo Identities può venire espressamente richiesto dal suo utilizzatore (cioè dal
programma *qspnclient*) di aggiungere un arco-identità, ma di norma questo non si fa. Inoltre quando
viene creata una nuova identità coi metodi *prepare_add_identity* e *add_identity* (a seguito di una
migrazione o un ingresso in una rete) tutti gli archi-identità della precedente identità vengono duplicati.
Infine quando una identità nuova si crea in un vicino (a seguito di una migrazione che coinvolge l'identità
del vicino ma non coinvolge la nostra identità) il modulo Identities riceve direttamente dalla rete
istruzioni per creare un nuovo arco-identità.

In ogni occasione in cui viene aggiunto un arco-identità, il modulo Identities emette un segnale
*identity_arc_added* con i dati "arco" (istanza di IIdmgmtArc), "propria identità" (istanza di NodeID)
e "arco-identità" (istanza di IIdmgmtIdentityArc). In questo momento il programma *qspnclient* può
identificare il nuovo arco-identità con l'indice autoincrementante *identityarc_nextindex*, che parte da 0.
Ad ogni indice rimane associato sia l'arco, sia la propria identità, sia l'identità nel nodo vicino.
Ricordiamo che dall'associazione "arco + propria identità" si può risalire al link-local del proprio nodo,
che nel tempo può cambiare. Ricordiamo che dall' "arco-identità" si può risalire sia al NodeID del vicino
(che non cambia nel tempo) sia al link-local del vicino, che nel tempo può cambiare.

* * *

All'avvio del nodo viene creata la prima istanza di QspnManager con il costruttore *create_net*. Ogni nodo
costruisce inizialmente una rete nuova che comprende solo la sua prima identità principale. Per questo
devono essere passati fin dall'inizio dall'utente al programma *qspnclient* i dati della topologia e il
primo indirizzo Netsukuku che il nodo si assegna. L'identificativo del fingerprint a livello 0 è scelto
a caso dal programma e le anzianità sono a zero (primo g-nodo) a tutti i livelli.

Siccome il QSPN è un modulo di identità, ogni istanza di QspnManager viene memorizzata come membro di una
identità nel modulo Identities. Quindi non è necessario un ulteriore indice perché l'utente possa
referenziare una istanza di QspnManager: è sufficiente l'indice *nodeid_nextindex*.

* * *

Al lancio del programma *qspnclient* l'utente indica attraverso appositi flag come vuole che il nodo si
comporti riguardo le forme di contatto anonimo. Questo concetto verrà spiegato più sotto. Il comportamento
di default del programma, se l'utente non indica alcun flag a riguardo, è di:

*   abilitare l'anonimizzazione dei pacchetti che transitano per il nodo;
*   non accettare richieste indirizzate al nodo da un client in forma anonima.

## <a name="Mappatura_indirizzi_ip"/>Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IPv4

Gli indirizzi dei nodi nella rete Netsukuku, nella sua attuale implementazione, vanno mappati nella
classe IPv4 10.0.0.0/8.

La rete viene suddivisa in un numero arbitrario di livelli.
La [notazione CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) usata per individuare
classi di indirizzi nelle reti IPv4, ci obbliga ad usare come gsize ad ogni livello una potenza di 2.
In teoria, ad esempio, potremmo avere un livello 0 di gsize 5. Ma quando vogliamo indicare nelle
tabelle di routing tutti gli indirizzi di un g-nodo di livello 1 (ad esempio da 10.1.2.0 a 10.1.2.4) non
potremmo farlo in forma compatta. Invece se usiamo un livello 0 di gsize 8, per riferirci agli indirizzi
nel g-nodo da 10.1.2.0 a 10.1.2.7 possiamo usare la notazione 10.1.2.0/29; per gli indirizzi da 10.1.2.8
a 10.1.2.15 useremo 10.1.2.8/29; e così via.

Indichiamo con *l* il numero dei livelli. Indichiamo con *gsize(i)* la dimensione dei g-nodi di livello
*i* + 1. Tali dimensioni devono essere potenze di 2. Indichiamo con *g-exp(i)* l'esponente della potenza
di 2 che dà la dimensione dei g-nodi di livello *i* + 1. Il numero di bit necessari a coprire lo spazio
di indirizzi è dato dalla sommatoria per *i* da 0 a *l* - 1 di *g-exp(i)*. 𝛴 *<sub>0 ≤ i < l</sub>* *g-exp(i)*.

Questo numero di bit non può essere maggiore di 22. Gli indirizzi IP nella classe 10.0.0.0/8 hanno 24 bit
a disposizione. Però ai bit sfruttati per la rappresentazione di un indirizzo Netsukuku vanno aggiunti 2 bit,
nella posizione più significativa, che riserviamo per notazioni particolari (routing interno ai g-nodi e
forma anonima).

Inoltre dobbiamo assicurarci che *gsize(l-1)* ≥ *l*. Cioè che nello spazio destinato al livello più alto
sia possibile rappresentare un numero da 0 a *l* - 1. Questo pure ci serve per la notazione usata per il
routing interno ai g-nodi.

Una volta scelti i valori di *l* e di *g-exp(i)* rispettando i vincoli prima esposti, vediamo come si mappa
un indirizzo Netsukuku (di nodo o di g-nodo) nei vari tipi di indirizzi IP. Si veda anche il
documento [livelli e bits](../ModuloQSPN/LivelliBits.md).

### <a name="Indirizzo_globale_nodo"/>Indirizzo IP globale di un nodo

Sia *n* l'indirizzo Netsukuku di un nodo. Indichiamo con *n<sub>0</sub>* l'identificativo del nodo
all'interno del suo g-nodo di livello 1. E a seguire con *n<sub>i</sub>* l'identificativo del g-nodo di
livello *i* a cui appartiene *n* all'interno del suo g-nodo di livello *i* + 1. L'indirizzo completo sarà
*n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*.

Il valore *n<sub>0</sub>* viene riportato nei bit meno significativi dell'indirizzo IP che stiamo
componendo, cioè partendo dal bit 0 per un numero di *g-exp*(0) bit. Il valore *n<sub>1</sub>* viene
riportato nei successivi bit, cioè partendo da *g-exp*(0) per un numero di *g-exp*(1) bit. E così di
seguito, l'identificativo *n<sub>i</sub>* (con *i* che arriva fino a *l* - 1) viene riportato nei bit
partendo da 𝛴 *<sub>0 ≤ k < i-1</sub>* *g-exp(k)* per un numero di *g-exp(i)* bit.

I successivi 2 bit (quelli riservati per routing interno ai g-nodi e forma anonima) li impostiamo a 0.

#### Esempio

Consideriamo una topologia di rete con 4 livelli. Diamo 2 bit al livello 3, 4 bit al livello 2, 8 bit
ai livelli 1 e 0. Sono soddisfatti i due vincoli esposti sopra.

Consideriamo il nodo *n* con indirizzo Netsukuku 3·10·123·45. L'indirizzo IP globale di *n* è 10.58.123.45.

### <a name="Indirizzo_globale_gnodo"/>Indirizzo IP globale di un g-nodo

Sia *g* l'indirizzo Netsukuku di un g-nodo di livello *i*. L'indirizzo completo sarà
*g<sub>l-1</sub>·...·g<sub>i</sub>*.

Il valore *g<sub>i</sub>* viene riportato nell'indirizzo IP che stiamo componendo partendo dal
bit 𝛴 *<sub>0 ≤ k < i-1</sub>* *g-exp(k)* per un numero di *g-exp*(i) bit. I bit meno significativi
sono messi a 0. Quei bit non saranno comunque presi in considerazione a causa dal prefisso di routing
della notazione CIDR, trattandosi dell'indirizzo IP di un intero g-nodo considerato come una IP subnet.

I valori dei bit più alti si calcolano come visto prima per l'indirizzo di un singolo nodo. Infine si
aggiunge, come accennato, il prefisso di routing, ottenuto come 32 - 𝛴 *<sub>0 ≤ k < i-1</sub>* *g-exp(k)*.

#### Esempio

Consideriamo il nodo *n* di prima nella topologia di rete di prima. Prendiamo in esame un g-nodo che
esso vuole indirizzare, ad esempio *g* = (2, 1) cioè il g-nodo di livello 2 e identificativo 1 che
appartiene al suo stesso g-nodo di livello 3.

L'indirizzo Netsukuku di *g* è 3·1. L'indirizzo IP globale di *g* in notazione CIDR è 10.49.0.0/16.

### <a name="Indirizzo_interno_nodo"/>Indirizzo IP di un nodo interno ad un suo g-nodo

Sia *n* un nodo con indirizzo *n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*. Sia *g* il suo g-nodo
di livello *i* con 0 < *i* < *l*. Quindi *g* ha indirizzo *n<sub>l-1</sub>·...·n<sub>i</sub>*. Vogliamo
comporre un indirizzo IP di *n* che sia univoco internamente a *g*.

I valori da *n<sub>0</sub>* a *n<sub>i-1</sub>* sono riportati come visto prima nei relativi bit
dell'indirizzo IP che stiamo componendo. Il valore *i* viene riportato nei bit che sarebbero stati
destinati all'identificativo di livello più alto, cioè *n<sub>l-1</sub>*. Gli altri bit sono lasciati a 0.

Il secondo bit più alto (quello riservato per routing interno ai g-nodi) lo impostiamo a 1. Il primo
bit più alto (quello riservato per forma anonima) lo impostiamo a 0.

#### Esempio

Consideriamo il nodo *n* di prima nella topologia di rete di prima. Aggiungiamo anche il nodo *m* con
indirizzo Netsukuku 3·10·67·89.

Fin da subito il nodo *n* si è assegnato, oltre all'indirizzo IP globale 10.58.123.45, anche l'indirizzo
IP interno al g-nodo di livello 2, che è 10.96.123.45. Analogamente, il nodo *m* si è assegnato, oltre
all'indirizzo IP globale 10.58.67.89, anche l'indirizzo IP interno al g-nodo di livello 2, che è 10.96.67.89.

### <a name="Indirizzo_interno_gnodo"/>Indirizzo IP di un g-nodo interno al suo g-nodo direttamente superiore

Sia *g* un g-nodo di livello *i* con indirizzo *g<sub>l-1</sub>·...·g<sub>i</sub>* con *i* < *l* - 1.
Sia *h* il suo g-nodo di livello *i* + 1. Quindi *h* ha indirizzo *g<sub>l-1</sub>·...·g<sub>i+1</sub>*.
Vogliamo comporre un indirizzo IP in notazione CIDR di *g* che sia univoco internamente a *h*.

Il valore di *g<sub>i</sub>* è riportato come visto prima nei relativi bit dell'indirizzo IP che stiamo
componendo. Il valore *i* + 1 viene riportato nei bit che sarebbero stati destinati all'identificativo di
livello più alto, cioè *g<sub>l-1</sub>*. Gli altri bit sono lasciati a 0.

Il secondo bit più alto lo impostiamo a 1. Il primo bit più alto lo impostiamo a 0.

#### Esempio

Consideriamo i nodi *n* e *m* di prima. Dal punto di vista di *n*, il nodo *m* si trova nel g-nodo
*g* 3·10·67 che ha in comune con lui il g-nodo direttamente superiore *h* 3·10.

Il g-nodo *g* all'interno di *h* viene individuato con l'indirizzo IP 10.96.67.0/24. Questo significa che
il nodo *n* ha nelle tabelle di routing una rotta per 10.96.67.0/24 che scaturisce dal percorso che gli è
noto verso la destinazione *g*. Quindi, se il nodo *n* trasmette un pacchetto IP all'indirizzo 10.96.67.89
(che *m* si era assegnato nell'esempio sopra) tale pacchetto prende quella rotta.

### <a name="Indirizzo_anonimizzante"/>Indirizzo IP di un nodo o g-nodo contattabile in forma anonima

Sia *n* un nodo con indirizzo *n<sub>l-1</sub>·...·n<sub>1</sub>·n<sub>0</sub>*. Vogliamo comporre
un indirizzo IP per tale risorsa tale che un client lo possa usare per contattare il nodo mantenendo
l'anonimato. Chiamiamo un tale indirizzo *anonimizzante*.

I valori da *n<sub>0</sub>* a *n<sub>l-1</sub>* sono riportati come visto prima nei relativi bit
dell'indirizzo IP che stiamo componendo.

Il secondo bit più alto (quello riservato per routing interno ai g-nodi) lo impostiamo a 0. Il primo
bit più alto (quello riservato per forma anonima) lo impostiamo a 1.

Nel caso di un g-nodo, per produrre un indirizzo anonimizzante in notazione CIDR si procede in modo
analogo, aggiungendo il prefisso come visto prima.

#### Esempio

Consideriamo il nodo *n* di prima. L'indirizzo IP globale anonimizzante di *n* è 10.186.123.45.

Se il nodo *n* ammette la possibilità di venire contattato in forma anonima (questa è una sua scelta) si
assegna anche questo indirizzo.

## <a name="Identita"/>Identità

Ogni identità che vive nel nodo ha un suo indirizzo Netsukuku. Inoltre ha una mappa di percorsi, ognuno
che ha come destinazione (e come passi) un g-nodo *visibile* dal suo indirizzo Netsukuku.

Un nodo ha sempre una identità principale e zero o più identità di connettività.

### <a name="Identita_principale"/>Identità principale

L'identità principale gestisce il network namespace default. L'identità principale ha un indirizzo
Netsukuku *definitivo* che può essere *reale* o *virtuale*.

Se è *reale*, riguardo questo indirizzo Netsukuku *n*:

*   Il nodo si assegna, su ogni interfaccia gestita, l'indirizzo IP globale di *n*.
*   Il nodo può (opzionalmente) assegnarsi, su ogni interfaccia gestita, l'indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *l* - 1:
    *   Il nodo si assegna, su ogni interfaccia gestita, l'indirizzo IP interno al livello *j* + 1 di *n*
        (non quando *j* + 1 = *l*; in quel caso abbiamo solo l'indirizzo IP globale).
    *   Per ogni g-nodo *d* di livello *j* che il nodo conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il nodo computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.
            *   *d<sub>i</sub>* - Indirizzo IP interno al livello *j* + 1 del g-nodo *d*. Questo
                solo quando *j* + 1 ≠ *l*.
        *   Il nodo imposta una rotta in *partenza* verso *d<sub>g</sub>*. In essa l'indirizzo
            usato come *src* è l'indirizzo IP globale di *n*.  
            Ricordiamo che nelle tabelle di routing del kernel si riporta come informazione solo
            il primo gateway di una rotta, sebbene il nodo sia a conoscenza di altre informazioni.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>g</sub>* non può essere "non raggiungibile"
            perché abbiamo appena detto che il nodo conosce la destinazione *d*, quindi almeno
            un percorso verso *d*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>g</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>g</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*: il nodo potrebbe non conoscere nessun percorso
                verso *d* che non passi per il massimo distinto g-nodo di *m* per *n*.
        *   Il nodo dovrebbe impostare una rotta in *inoltro* verso *d<sub>g</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            In realtà, per come funziona lo stack TCP/IP di Linux, tale rotta è superflua. Infatti
            essa sarebbe identica a quella che è stata impostata per i pacchetti in *partenza* verso
            *d<sub>g</sub>* e lo stack TCP/IP prende in considerazione questa anche per i pacchetti
            in *inoltro*. Perciò il programma *qspnclient* non imposta una ulteriore rotta.
        *   Il nodo imposta una rotta in *partenza* verso *d<sub>a</sub>*. In essa l'indirizzo
            usato come *src* è l'indirizzo IP globale di *n*.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>g</sub>* non può essere "non raggiungibile".
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>a</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>g</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo dovrebbe impostare una rotta in *inoltro* verso *d<sub>a</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce. Ma come detto
            sopra lo stack TCP/IP di Linux si avvale della rotta che è stata impostata per i pacchetti
            in *partenza* verso *d<sub>a</sub>*. Perciò il programma *qspnclient* non imposta una ulteriore rotta.
        *   Il nodo imposta una rotta in *partenza* verso *d<sub>i</sub>*. In essa l'indirizzo
            usato come *src* è l'indirizzo IP interno al livello *j* + 1 di *n*.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>g</sub>* non può essere "non raggiungibile".
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>i</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo dovrebbe impostare una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce. Ma come detto
            sopra lo stack TCP/IP di Linux si avvale della rotta che è stata impostata per i pacchetti
            in *partenza* verso *d<sub>i</sub>*. Perciò il programma *qspnclient* non imposta una ulteriore rotta.

Se è *virtuale*, significa che l'indirizzo ha una o più componenti virtuali. Sia *i* il livello
più basso in cui la componente è virtuale. Sia *k* il livello più alto in cui la componente è virtuale.

In questo caso, riguardo questo indirizzo Netsukuku *n*:

*   NON esiste un indirizzo IP globale di *n*.
*   NON esiste un indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *i*-1:
    *   Il nodo si assegna, su ogni interfaccia gestita, l'indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che il nodo conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il nodo computa questi indirizzi IP:
            *   *d<sub>i</sub>* - Indirizzo IP interno al livello *j* + 1 del g-nodo *d*.
        *   Il nodo imposta una rotta in *partenza* verso *d<sub>i</sub>*. In essa l'indirizzo
            usato come *src* è l'indirizzo IP interno al livello *j* + 1 di *n*.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>g</sub>* non può essere "non raggiungibile".
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>i</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo dovrebbe impostare una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce. Ma come detto
            sopra lo stack TCP/IP di Linux si avvale della rotta che è stata impostata per i pacchetti
            in *partenza* verso *d<sub>i</sub>*. Perciò il programma *qspnclient* non imposta una ulteriore rotta.
*   Per ogni livello *j* da *i* a *k*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che il nodo conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il nodo computa questi indirizzi IP:
            *   *d<sub>i</sub>* - Indirizzo IP interno al livello *j* + 1 del g-nodo *d*.
        *   Il nodo NON ha una rotta in *partenza* verso *d<sub>i</sub>*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>i</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>i</sub>* non può essere "non raggiungibile".
*   Per ogni livello *j* da *k* a *l*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che il nodo conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il nodo computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.
            *   *d<sub>i</sub>* - Indirizzo IP interno al livello *j* + 1 del g-nodo *d*. Questo
                solo quando *j* + 1 ≠ *l*.
        *   Il nodo NON ha una rotta in *partenza* verso *d<sub>g</sub>*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>g</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>g</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo imposta una rotta in *inoltro* verso *d<sub>g</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>g</sub>* non può essere "non raggiungibile".
        *   Il nodo NON ha una rotta in *partenza* verso *d<sub>a</sub>*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>a</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>a</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo imposta una rotta in *inoltro* verso *d<sub>a</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>a</sub>* non può essere "non raggiungibile".
        *   Il nodo NON ha una rotta in *partenza* verso *d<sub>i</sub>*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>i</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>i</sub>* non può essere "non raggiungibile".

### <a name="Identita_di_connettivita"/>Identità di connettività

Un nodo può avere 0 o più identità di connettività. L'identità di connettività gestisce un certo
network namespace. L'identità di connettività ha un indirizzo Netsukuku *di connettività* che è *virtuale*.

Significa che l'indirizzo ha una o più componenti virtuali. Sia *i* il livello più basso in cui
la componente è virtuale. Sia *k* il livello più alto in cui la componente è virtuale.

In questo caso, riguardo questo indirizzo Netsukuku *n*:

*   NON esiste un indirizzo IP globale di *n*.
*   NON esiste un indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *k*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che il nodo conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il nodo computa questi indirizzi IP:
            *   *d<sub>i</sub>* - Indirizzo IP interno al livello *j* + 1 del g-nodo *d*.
        *   Il nodo NON ha una rotta in *partenza* verso *d<sub>i</sub>*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>i</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>i</sub>* non può essere "non raggiungibile".
*   Per ogni livello *j* da *k* a *l*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che il nodo conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il nodo computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.
            *   *d<sub>i</sub>* - Indirizzo IP interno al livello *j* + 1 del g-nodo *d*. Questo
                solo quando *j* + 1 ≠ *l*.
        *   Il nodo NON ha una rotta in *partenza* verso *d<sub>g</sub>*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>g</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>g</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo imposta una rotta in *inoltro* verso *d<sub>g</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>g</sub>* non può essere "non raggiungibile".
        *   Il nodo NON ha una rotta in *partenza* verso *d<sub>a</sub>*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>a</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>a</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo imposta una rotta in *inoltro* verso *d<sub>a</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>a</sub>* non può essere "non raggiungibile".
        *   Il nodo NON ha una rotta in *partenza* verso *d<sub>i</sub>*.
        *   Per ogni MAC address *m* di diretto vicino che il nodo conosce:
            *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
                provenienti da *m*.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                La destinazione *d<sub>i</sub>* può essere "non raggiungibile" per i pacchetti
                in *inoltro* provenienti da *m*.
        *   Il nodo imposta una rotta in *inoltro* verso *d<sub>i</sub>* per i pacchetti
            provenienti da un MAC address che non è fra quelli che il nodo conosce.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione. La destinazione *d<sub>i</sub>* non può essere "non raggiungibile".

## <a name="Indirizzi_del_nodo"/>Indirizzi IP di ogni identità nel nodo

Come abbiamo visto prima, in un nodo possono esistere diverse identità. Ogni identità detiene un
indirizzo Netsukuku. A seconda del tipo, sulla base del suo indirizzo Netsukuku ogni identità
può volersi assegnare zero o più indirizzi IPv4.

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa il programma *qspnclient*.

In un sistema Linux il sistema si assegna un indirizzo associandolo ad una interfaccia di rete. In realtà
ogni interfaccia può avere più indirizzi e anche lo stesso indirizzo può essere associato a più interfacce;
inoltre anche se un indirizzo *x* viene associato alla interfaccia *nicA* e non all'interfaccia *nicB*, un
pacchetto destinato a *x* ricevuto tramite l'interfaccia *nicB* giunge ugualmente al processo che sta in
ascolto sull'indirizzo *x*.

Si noti che il fatto di associare un indirizzo IP ad una specifica interfaccia di rete
ha la sua importanza in relazione al protocollo di risoluzione degli indirizzi IP in
indirizzi MAC ([Address Resolution Protocol](https://en.wikipedia.org/wiki/Address_Resolution_Protocol)).
Infatti quando un nodo *a* vuole trasmettere un pacchetto IP ad un suo diretto vicino *b*, esso conosce
l'indirizzo IP di *b* e l'interfaccia di rete di *a* dove trasmettere. Il nodo *a* trasmette un frame Ethernet broadcast
su quel segmento di rete una richiesta: «chi ha l'indirizzo IP XYZ?». Il nodo *b* risponde indicando al
nodo *a* l'indirizzo MAC della sua interfaccia di rete. Quindi il nodo *a* può incapsulare il pacchetto
IP in un frame Ethernet unicast che riporta gli indirizzi MAC dell'interfaccia che trasmette e dell'interfaccia che deve ricevere.

Se il nodo *b* ha diverse interfacce di rete tutte collegate allo stesso segmento di rete, il fatto
di associare diversi indirizzi IP a diverse interfacce può fornire un modo di identificare una precisa
interfaccia di rete a cui un pacchetto va trasmesso. Questo è usato dal modulo Qspn per distinguere
i pacchetti che vanno ricevuti da una pseudo-interfaccia, per questo gli indirizzi link-local sono diversi
su ogni interfaccia e sulle pseudo-interfacce.

Fatta questa premessa, come si comporta il programma?

Il programma *qspnclient*, quando crea una *identità*, a seconda del tipo di identità e del suo indirizzo
Netsukuku, come visto prima, computa gli indirizzi IP che si deve assegnare e li associa tutti
a ognuna delle interfacce di rete che gestisce quell'identità.

Inoltre, prima di rimuovere una identità, li rimuove da tutte le interfacce che gestisce quell'identità.

## <a name="Rotte_nelle_tabelle_di_routing"/>Rotte nelle tabelle di routing

Il programma deve istruire le policy di routing del sistema (che di norma significa impostare delle
rotte nelle tabelle di routing) in modo da garantire questi comportamenti:

*   Se un processo locale vuole inviare un pacchetto ad un indirizzo *d* nello spazio di Netsukuku:
    *   Se il modulo QSPN non ha alcun percorso verso la destinazione *d*:
        *   Il sistema non trova nelle tabelle del kernel alcuna rotta e segnala al processo
            che la destinazione *d* è irraggiungibile.
    *   Altrimenti:
        *   Il sistema trova nelle tabelle del kernel una rotta il cui gateway è costituito dal
            primo hop del miglior percorso (fra tutti quelli che il modulo QSPN ha scoperto) verso
            la destinazione *d*.
*   Se un pacchetto è ricevuto da un vicino *v* ed è destinato ad un altro indirizzo *d* nello spazio di Netsukuku:
    *   Se il modulo QSPN non ha alcun percorso verso la destinazione *d*, tale che non contenga
        fra i suoi hop il *massimo distinto g-nodo* del vicino *v*:
        *   Il sistema non trova nelle tabelle del kernel alcuna rotta; quindi scarta il pacchetto
            e invia un pacchetto ICMP «host *d* irraggiungibile» al mittente.
    *   Altrimenti:
        *   Il sistema trova nelle tabelle del kernel una rotta il cui gateway è costituito dal primo
            hop per il miglior percorso verso la destinazione *d*, tale che non contenga il g-nodo del vicino.
        *   Se l'indirizzo *d* è nello spazio di indirizzi che codificano la richiesta di anonimato:
            *   Opzionalmente, il sistema trova una regola che gli impone di mascherare l'indirizzo
                del mittente (cioè di indicare se stesso come mittente e inoltrare le comunicazioni di
                risposta che riceverà al vero mittente).

* * *

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa
il programma *qspnclient*. Esaminiamo prima l'aspetto del source natting e poi del routing.

### <a name="Source_natting"/>Source NATting

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

Fatta questa premessa, come si comporta il programma?

Il programma, all'avvio, opzionalmente, istruisce il kernel per il source natting. Con questa configurazione
il nodo si rende disponibile ad anonimizzare i pacchetti che riceve e che vanno inoltrati verso una
destinazione che accetta richieste anonime.

L'opzione di rendere anonimi i pacchetti che transitano per il nodo nel percorso verso un'altra destinazione
è distinta e indipendente dall'opzione di accettare richieste anonime, che è stata discussa sopra.

*   **NOTA**: La seguente spiegazione sui motivi per cui l'operazione è opzionale va spostata in un
    documento che affronta ad alto livello le implicazioni sul detenere un nodo nella rete Netsukuku. Il documento
    presente si limita a illustrare i dettagli implementativi del programma *qspnclient* o del demone *ntkd*.  
    Questa azione è opzionale perché il proprietario di un nodo può avere remore a nascondere il vero mittente
    di un messaggio prendendo il suo posto. In realtà questo timore sarebbe infondato, vediamo perché. Per far
    funzionare bene l'operazione di contatto anonimo da parte del client, occorre che il nodo che fa da server
    (fornisce un servizio) si assegni anche gli indirizzi per essere contattato in forma anonima. Se fa questa
    operazione opzionale, significa che è pronto a ricevere alcune richieste dalle quali saprà di non
    poter risalire al mittente. Sarà quindi responsabile di rispondere o meno a tali richieste e non
    potrà far ricadere tale onere sugli altri nodi.  
    Anche considerando quindi non rischiosa l'azione di implementare nel proprio nodo il source natting,
    l'azione è opzionale perché il nodo che la implementa si carica di un onere che costa un po' in
    termini di memoria. Se il nodo quindi ha scarse risorse (si intende molto scarse, come pochi mega
    di RAM) conviene che non la implementi.  
    Va considerato che se un nodo decide di non implementare questa azione, comunque il meccanismo di
    trasmissione anonima risulta efficace se nel percorso tra il mittente e il destinatario almeno un nodo è
    disposto a implementarla. Invece, se un nodo decide di implementare l'azione e ad un certo punto le sue risorse
    di memoria venissero meno, in questo caso la comunicazione in corso ne verrebbe compromessa.

Se il nodo decide di implementare il source natting, calcola lo spazio di indirizzi che indicano una
risorsa da raggiungere in forma anonima. Una volta calcolato il numero di bit necessari a ricoprire i vari
livelli della nostra rete -si rilegga il paragrafo sopra sullo spazio di indirizzi Netsukuku- vanno considerati
altri 2 bit in testa; il primo di questi va impostato e tutti gli altri sono liberi. Se ad esempio si usano
tutti i 22 bit a disposizione per gli indirizzi, allora il range di indirizzi che indicano una risorsa da
raggiungere in forma anonima è 10.128.0.0/9.

Poi il range viene ulteriormente scomposto in un numero di sottoclassi pari al numero dei livelli nella
rete. Il primo blocco costituisce gli indirizzi globali delle risorse da raggiungere in forma anonima; si
caratterizza per avere NON impostato il secondo bit. Nel nostro esempio, il range di questo blocco è 10.128.0.0/10.

Prendiamo ora ogni livello *i* della nostra rete, da 1 a *l* - 1 inclusi, dove *l* è il numero di
livelli. Il blocco del livello *i* costituisce gli indirizzi interni al nostro g-nodo di livello *i* delle
risorse (interne al suddetto g-nodo) da raggiungere in forma anonima; si caratterizza per avere impostato
il secondo bit e per avere codificato il numero *i* nei bit destinati all'identificativo del livello più alto.

Riprendiamo il nostro esempio e supponiamo che ci siano nella rete 10 livelli e il più alto abbia
dimensione 16, vale a dire gsize(9) = 16, g-exp(9) = 4. Allora il blocco del livello *i* ha impostato
il primo e il secondo bit ed ha codificato il numero *i* nei 4 bit dalla posizione 3 alla 6 incluse. Questi sono quindi i blocchi:

```
 . 10.196.0.0/14
 . 10.200.0.0/14
 . 10.204.0.0/14
 . 10.208.0.0/14
 . 10.212.0.0/14
 . 10.216.0.0/14
 . 10.220.0.0/14
 . 10.224.0.0/14
 . 10.228.0.0/14
```

Supponiamo ora che il nostro nodo sia nel g-nodo di livello 9 con posizione 5 e, per semplicità,
supponiamo posizione 0 a tutti gli altri livelli. Quindi -si rilegga il paragrafo sopra sugli
indirizzi del nodo- il nostro nodo ha assegnati a se questi indirizzi (in forma non anonima):

```
 . 10.20.0.0 (come indirizzo globale)
 . 10.68.0.0 (come interno al suo g-nodo di livello 1)
 . 10.72.0.0
 . 10.76.0.0
 . 10.80.0.0
 . 10.84.0.0
 . 10.88.0.0
 . 10.92.0.0
 . 10.96.0.0
 . 10.100.0.0 (come interno al suo g-nodo di livello 9)
```

Allora il programma istruisce il kernel di modificare i pacchetti destinati ai seguenti range
indicando come nuovo indirizzo mittente uno dei propri indirizzi, come segue:

```
   iptables -t nat -A POSTROUTING -d 10.128.0.0/10 -j SNAT --to 10.20.0.0
   iptables -t nat -A POSTROUTING -d 10.196.0.0/14 -j SNAT --to 10.68.0.0
   iptables -t nat -A POSTROUTING -d 10.200.0.0/14 -j SNAT --to 10.72.0.0
   iptables -t nat -A POSTROUTING -d 10.204.0.0/14 -j SNAT --to 10.76.0.0
   iptables -t nat -A POSTROUTING -d 10.208.0.0/14 -j SNAT --to 10.80.0.0
   iptables -t nat -A POSTROUTING -d 10.212.0.0/14 -j SNAT --to 10.84.0.0
   iptables -t nat -A POSTROUTING -d 10.216.0.0/14 -j SNAT --to 10.88.0.0
   iptables -t nat -A POSTROUTING -d 10.220.0.0/14 -j SNAT --to 10.92.0.0
   iptables -t nat -A POSTROUTING -d 10.224.0.0/14 -j SNAT --to 10.96.0.0
   iptables -t nat -A POSTROUTING -d 10.228.0.0/14 -j SNAT --to 10.100.0.0
```

Quando il programma termina, se aveva istruito il kernel per fare il source natting, rimuove le regole
che aveva messe nella catena `POSTROUTING` della tabella `nat`.

### <a name="Routing"/>Routing

In un sistema Linux le rotte vengono memorizzate in diverse tabelle. Queste tabelle hanno un
identificativo che è un numero da 0 a 255. Hanno anche un nome descrittivo: l'associazione del
nome al numero (che in realtà è il vero identificativo) è fatta nel file `/etc/iproute2/rt_tables`.

In ogni tabella possono esserci diverse rotte. Ogni rotta ha alcune informazioni importanti:

*   Destinazione. È in formato CIDR. Indica una classe di indirizzi. Solo se il pacchetto è destinato
    a quella classe allora la rotta va presa in considerazione. Se ci sono più classi che soddisfano il
    pacchetto, allora si prende quella più restrittiva.
*   Unreachable. Se presente indica che la destinazione è irraggiungibile.
*   Gateway (gw) e interfaccia (dev). Dice dove trasmettere il pacchetto.
*   Mittente preferito (src). Questa informazione è usata solo per i pacchetti trasmessi dal sistema
    locale, non per i pacchetti da inoltrare. È un indirizzo del sistema locale. Dice quale indirizzo
    locale usare come mittente, se non viene espressamente specificato un indirizzo locale dal processo
    che richiede la trasmissione.

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

Fatta questa premessa, come si comporta il programma?

Per ogni arco verso un vicino, il programma memorizza una rotta diretta (cioè senza gateway) nella
tabella `main`, indicando come destinazione l'indirizzo di scheda del vicino e come mittente preferito
il proprio indirizzo di scheda. Questo lo abbiamo visto poco sopra; infatti il comando `ip route add`
senza specificare un nome di tabella si riferisce alla tabella `main`.

Il programma crea una tabella `ntk` con identificativo `YYY`, dove `YYY` è il primo identificativo
libero nel file `/etc/iproute2/rt_tables`. Tale tabella sarà inizialmente vuota; in essa il programma
andrà a mettere le rotte di pertinenza della rete Netsukuku, cioè quelle con destinazione nello
spazio 10.0.0.0/8. Inoltre aggiunge una regola che dice di guardare la tabella `ntk` prima della `main`.

Il programma, per ogni suo arco, crea un'altra tabella chiamata `ntk_from_XXX` con identificativo `YYY`,
dove `XXX` è il MAC address del nodo vicino, `YYY` è il primo identificativo libero nel
file `/etc/iproute2/rt_tables`. Questa tabella conterrà rotte da esaminare solo per i pacchetti da
inoltrare che ci sono pervenuti attraverso questo arco. Il programma quindi aggiunge una regola che
dice di guardare la tabella `ntk_from_XXX` se il pacchetto da trasmettere è marcato con il numero
`YYY`. Inoltre istruisce il kernel di marcare con il numero `YYY` i pacchetti che hanno `XXX` come MAC di provenienza.

Anche le varie tabelle `ntk_from_XXX` conterranno solo rotte di pertinenza della rete Netsukuku, cioè
quelle con destinazione nello spazio 10.0.0.0/8.

Inoltre, sia la tabella `ntk` sia le varie `ntk_from_XXX` conterranno la rotta "unreachable 10.0.0.0/8".
Questa rotta verrà presa in esame solo se un pacchetto ha una destinazione all'interno dello spazio
di Netsukuku, ma per tale destinazione non esistono altre rotte valide con una classe più restrittiva.
In altre parole, una destinazione per la quale non si conosce nessun percorso. Questa particolare rotta dice
che il pacchetto non potrà giungere a destinazione e il suo mittente ne va informato.

Sulla base degli eventi segnalati dal modulo QSPN, e se necessario richiamando i suoi metodi pubblici, il
programma *qspnclient* popola e mantiene le rotte nelle tabelle `ntk` e `ntk_from_XXX`. I percorsi
segnalati dal modulo QSPN contengono sempre un arco del nodo corrente come passo iniziale e da tale arco
si può risalire all'indirizzo di scheda del vicino. Le rotte nelle tabelle `ntk` e `ntk_from_XXX` infatti
devono avere come campo gateway (gw) l'indirizzo di scheda del vicino, non il suo indirizzo Netsukuku.

Per ogni percorso scelto dal programma *qspnclient* per entrare in una tabella, in realtà il programma
inserisce nella tabella fino a 4 rotte. Sia *i* il livello del g-nodo destinazione del percorso. Queste
sono le rotte:

*   Globale. La destinazione di questa rotta è l'indirizzo del g-nodo nella sua forma globale, il
    suo "src" è il mio indirizzo nella sua forma globale.
*   Anonimo globale. La destinazione di questa rotta è l'indirizzo del g-nodo nella sua forma globale
    e con anonimato, il suo "src" è il mio indirizzo nella sua forma globale.
*   Se *i* < *levels* - 1:
    *   Interno al nostro g-nodo di livello *i* + 1. La destinazione di questa rotta è l'indirizzo
        del g-nodo nella sua forma interna al g-nodo di livello *i* + 1, il suo "src" è il mio indirizzo
        nella sua forma interna al g-nodo di livello *i* + 1.
    *   Anonimo interno al nostro g-nodo di livello *i* + 1. La destinazione di questa rotta è l'indirizzo
        del g-nodo nella sua forma interna al g-nodo di livello *i* + 1 e con anonimato, il suo "src" è il
        mio indirizzo nella sua forma interna al g-nodo di livello *i* + 1.

Quando il programma ha finito di usare una tabella (ad esempio se un arco che conosceva non è più presente,
oppure se il programma termina) svuota la tabella, poi rimuove la regola, poi rimuove il record
relativo dal file `/etc/iproute2/rt_tables`.

