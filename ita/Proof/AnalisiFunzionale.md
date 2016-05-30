# Proof of concept - Analisi Funzionale

1.  [Ruolo del qspnclient](#Ruolo_del_qspnclient)
    1.  [Interazione programma-utente](#Interazione_programma_utente)
1.  [Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IPv4](#Mappatura_indirizzi_ip)
1.  [Identità](#Identita)
    1.  [Identità principale](#Identita_principale)
    1.  [Identità di connettività](#Identita_di_connettivita)
1.  [Indirizzi IP di ogni identità nel sistema](#Indirizzi_del_sistema)
1.  [Rotte nelle tabelle di routing](#Rotte_nelle_tabelle_di_routing)
    1.  [Source NATting](#Source_natting)
    1.  [Routing](#Routing)

Ci proponiamo di realizzare un programma, **qspnclient**, che si avvale del modulo QSPN e altri
moduli a sostegno (Neighborhood, Identities) per stabilire come impostare le rotte nelle tabelle
di routing del kernel di una macchina. Si tratta di un programma specifico per un sistema Linux.

## <a name="Ruolo_del_qspnclient"></a>Ruolo del qspnclient

Questo programma permette all'utente di fare le veci del demone *ntkd*, simulando le sue operazioni
e quelle di pertinenza di altri moduli:

*   Il dialogo con identità in sistemi vicini appartenenti ad altre reti.
*   Il coordinamento di un g-nodo (moduli PeerServices e Coordinator).
*   La strategia di ingresso in una rete, cioè la scelta del g-nodo *g* in cui chiedere un posto.
*   La ricerca della più breve *migration path* per liberare un posto in *g* (modulo Migrations).
*   La comunicazione delle informazioni a tutte le identità interessate dalla migration path trovata.

Sulla base dei comandi dati sulla console dall'utente, il programma *qspnclient* interagisce coi
moduli suddetti (QSPN, Neighborhood, Identities) e sulla base delle loro elaborazioni interviene
sulle configurazioni di rete del sistema. L'utente sarà quindi in grado di verificare che il
sistema riesca effettivamente a stabilire connessioni con gli altri sistemi della rete, che le rotte
siano quelle che ci si attende, eccetera. Inoltre il programma interattivamente consente di chiedere
al modulo QSPN le informazioni che ha raccolto e mostrarle all'utente.

### <a name="Interazione_programma_utente"></a>Interazione programma-utente

Il programma *qspnclient* prevede che l'utente immetta, come argomenti della riga di comando e in modo
interattivo dalla console durante la sua esecuzione, tutti i requisiti dei moduli di cui si avvale.

Ai parametri che saranno individuati in modo autonomo dai moduli (ad esempio gli identificativi di
nodo, gli indirizzi di scheda, ...) verranno associati degli indici progressivi che saranno visualizzati
all'utente. L'utente si riferirà ad essi tramite questi indici. Questo per rendere più facilmente
riproducibili gli ambienti di test.

* * *

All'avvio del programma nel sistema viene creata l'istanza di NeighborhoodManager. Poi gli sono passati i nomi delle
interfacce di rete che dovrà gestire, tramite il metodo *start_monitor*. Ad ogni interfaccia, il modulo Neighborhood associa un
indirizzo IP link-local scelto a caso. In realtà l'assegnazione dell'indirizzo è fatta proprio
dal programma attraverso una classe che implementa l'interfaccia INeighborhoodIPRouteManager passata
al modulo Neighborhood. Quello che fa il modulo Neighborhood è scegliere l'indirizzo.

Il programma si avvede dell'indirizzo scelto perché il NeighborhoodManager lo notifica con il segnale
*nic_address_set*. Il programma associa questo proprio indirizzo link-local all'indice
autoincrementante *linklocal_nextindex*, che parte da 0.

* * *

L'istanza di NeighborhoodManager deve essere istruita sul numero massimo di archi che può accettare.
Il programma *qspnclient* specifica un numero elevato. Nonostante questo, ogni arco che il modulo
Neighborhood realizza passa al vaglio dell'utente che decide se utilizzarlo o meno. Questo permette
di dirigere il proprio ambiente di test a piacimento anche in particolari scenari, come ad esempio
un gruppo di macchine virtuali che condividono un unico dominio di broadcast ma vogliono simulare un
gruppo di sistemi wireless disposti in un determinato modo.

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

All'avvio del programma nel sistema viene creata l'istanza di IdentityManager e questo crea un NodeID casuale. Il programma
recupera tale NodeID col metodo *get_main_id()* e lo associa all'indice autoincrementante *nodeid_nextindex*,
che parte da 0. In seguito il programma quando crea una nuova identità col metodo *add_identity* associa la
nuova istanza di NodeID al prossimo valore di *nodeid_nextindex*. Quindi l'utente può usare questo indice per
dare dalla console comandi concernenti una certa identità.

* * *

Alla creazione di una nuova identità, il modulo Identities crea un nuovo network namespace. In realtà la creazione
del network namespace è fatta proprio dal programma attraverso una classe che implementa l'interfaccia
IIdmgmtNetnsManager passata al modulo Identities. Quello che fa il modulo Identities è scegliere il nome di
tale namespace.

Il nome di tale namespace è già costruito con un indice progressivo, quindi il programma non gli associa un
indice.

Comunque (per il momento) non c'è nessun comando interattivo in cui l'utente debba specificare un
namespace. Quindi tale nome non è nemmeno mostrato all'utente.

* * *

Sempre alla creazione di una nuova identità, il modulo Identities, di nuovo con chiamate all'interfaccia
IIdmgmtNetnsManager, crea per ogni interfaccia di rete reale una pseudo-interfaccia. Anche qui il nome
della pseudo-interfaccia non è casuale ma generato con lo stesso indice progressivo usato per il
network namespace, quindi il programma non gli associa un indice.

Inoltre (per il momento) non c'è nessun comando interattivo in cui l'utente debba specificare una
pseudo-interfaccia. Quindi tale nome non è nemmeno mostrato all'utente.

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
Ad ogni indice rimane associato sia l'arco, sia la propria identità, sia l'identità nel sistema vicino.
Ricordiamo che dall'associazione "arco + propria identità" si può risalire al link-local dell'identità nel proprio sistema,
che nel tempo può cambiare. Ricordiamo che dall' "arco-identità" si può risalire sia al NodeID del vicino
(che non cambia nel tempo) sia al link-local dell'identità nel sistema vicino, che nel tempo può cambiare.

* * *

All'avvio del programma nel sistema viene creata la prima istanza di QspnManager con il costruttore *create_net* e viene
associata alla prima *identità principale* del sistema. Così si costruisce inizialmente una rete nuova che
comprende solo questa identità. I dati che servono sono forniti dall'utente sulla riga di comando: i dati della topologia e il
primo indirizzo Netsukuku che questa identità si assegna. L'identificativo del fingerprint a livello 0 è scelto
a caso dal programma e le anzianità sono a zero (primo g-nodo) a tutti i livelli.

Siccome il QSPN è un modulo di identità, ogni istanza di QspnManager viene memorizzata come membro di una
identità nel modulo Identities. Quindi non è necessario un ulteriore indice perché l'utente possa
referenziare una istanza di QspnManager: è sufficiente l'indice *nodeid_nextindex*.

* * *

Supponiamo ora che l'utente nel sistema *A* vuole simulare l'ingresso dell'identità *A<sub>0</sub>* in un'altra rete esistente
attraverso un arco tra i sistemi *A* e *B* e in particolare che collega le identità *A<sub>0</sub>* e *B<sub>0</sub>*.

L'utente chiederà al programma in esecuzione su *A* di costruire una nuova *identità* *A<sub>1</sub>*
basata su *A<sub>0</sub>*. Facendo questo si duplica anche l'arco-identità *A<sub>0</sub>*-*B<sub>0</sub>*
in un nuovo arco-identità *A<sub>1</sub>*-*B<sub>0</sub>* e di questo fatto si avvede autonomamente
l'IdentityManager sia nel sistema *A*, sia nel sistema *B*.

Poi l'utente chiederà al programma in esecuzione su *A*, con un comando interattivo che chiamiamo
*enter_net*, di costruire per essa una nuova istanza di QspnManager con il costruttore *enter_net*. I dati che
servono sono forniti dall'utente in modo interattivo: l'indirizzo del nuovo g-nodo prenotato nella
rete appena scoperta, la sua anzianità, gli archi-identità che ci collegano alla nuova rete, e altri.

Esaminiamo gli archi-identità. Per l'arco-identità *A<sub>1</sub>*-*B<sub>0</sub>* che in questo caso l'utente specifica (tramite
il suo indice) quando da il comando *enter_net* sul sistema *A*, il programma crea una istanza di IQspnArc
tale che il modulo QSPN di *A<sub>1</sub>* possa effettuare comunicazioni con il modulo QSPN
di *B<sub>0</sub>*.

Immediatamente l'utente dovrà dare un comando al programma in esecuzione su *B*, che chiamiamo *add_qspnarc*,
con il quale gli chiede di costruire e passare al QspnManager di *B<sub>0</sub>* una istanza di IQspnArc
tale che il modulo QSPN di *B<sub>0</sub>* possa effettuare comunicazioni con il modulo QSPN
di *A<sub>1</sub>*.

* * *

Per ogni *identità* creata, nel relativo network namespace, sulla base dell'indirizzo Netsukuku
ad essa associato, il programma computa un numero di indirizzi IP (in seguito verrà dettagliato
come siano computati) e se li assegna.

Inoltre, sempre per ogni *identità* e nel relativo network namespace, basandosi sui segnali notificati
dalla relativa istanza di QspnManager, il programma popola le tabelle di routing del kernel.

* * *

Al lancio del programma *qspnclient* l'utente indica attraverso appositi flag come vuole che il sistema si
comporti riguardo le forme di contatto anonimo. Questo concetto verrà spiegato più sotto. Il comportamento
di default del programma, se l'utente non indica alcun flag a riguardo, è di:

*   abilitare l'anonimizzazione dei pacchetti che transitano per il sistema;
*   non accettare richieste indirizzate al sistema da un client in forma anonima.

## <a name="Mappatura_indirizzi_ip"></a>Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IPv4

Gli indirizzi Netsukuku dei *nodi del grafo* vanno mappati in un range di indirizzi IP che si
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
di 2 che dà la dimensione dei g-nodi di livello *i* + 1. Il numero di bit necessari a coprire lo spazio
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
    che identifica lo stesso *sistema*; ma questa convoglia in più l'informazione che si vuole
    contattare quel *sistema* restando anonimi.
*   Un indirizzo IP interno al livello *i* per ogni valore di *i* da 1 a *l* - 1.  
    Questo indirizzo IP identifica un preciso *sistema* univocamente all'interno
    di un g-nodo *g* di livello *i*. Questo indirizzo IP si può utilizzare come indirizzo di
    destinazione di un pacchetto IP quando sia il *sistema* mittente che il *sistema*
    identificato (il destinatario) appartengono allo stesso g-nodo *g* di livello *i*. La
    peculiarità di questi indirizzi IP (che si riflette sui pacchetti IP trasmessi a questi
    indirizzi) è che essi non cambiano quando il g-nodo *g* o uno dei suoi g-nodi superiori
    *migra* all'interno della rete o anche in una diversa rete Netsukuku.

Ricordiamo che un indirizzo Netsukuku identifica un *nodo del grafo*, cioè una specifica identità
all'interno di un *sistema*. Però in ogni sistema, in ogni momento, esiste una ed una sola
*identità principale*, che è l'unica che detiene un indirizzo Netsukuku *reale*.

Per l'esattezza, l'identità principale di un sistema può detenere per brevi istanti un
indirizzo Netsukuku *virtuale*, durante le operazioni di una migrazione coinvolta in una
*migration path*. Durante questo periodo quel sistema (e insieme a quello anche tutti
gli altri sistemi che appartengono al g-nodo *g* che sta migrando) non può avere un indirizzo
IP globale. Questo però non inficia sulla possibilità di quel sistema di avere un indirizzo
IP interno al livello *i* per ogni valore di *i* da 1 a *k* - 1, dove *k* è il livello
del g-nodo *g*.

Questo permette, come detto prima, che le connessioni realizzate tra due sistemi appartenenti
al g-nodo *g* non vengano compromesse. Per questo aggiungiamo che la mappatura associa alcuni
indirizzi IP anche ad un indirizzo Netsukuku *virtuale*, purché siano *reali* i suoi
identificativi da 0 a *k* - 1. Questi sono:

*   Un indirizzo IP interno al livello *i* per ogni valore di *i* da 1 a *k*.

Gli algoritmi di calcolo dei vari tipi di indirizzo IP sono descritti nel documento [IndirizziIP](IndirizziIP.md).

## <a name="Identita"></a>Identità

Ogni identità che vive nel sistema ha un suo indirizzo Netsukuku. Inoltre ha una mappa di percorsi, ognuno
che ha come destinazione (e come passi) un g-nodo *visibile* dal suo indirizzo Netsukuku.

Un sistema ha sempre una identità principale e zero o più identità di connettività.

### <a name="Identita_principale"></a>Identità principale

L'identità principale gestisce il network namespace default. L'identità principale ha un indirizzo
Netsukuku *definitivo* che può essere *reale* o *virtuale*.

Se è *reale*, nel network namespace default:

*   Il sistema si assegna l'indirizzo IP globale di *n*.
*   Il sistema può (opzionalmente) assegnarsi l'indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *l* - 1:
    *   Il sistema si assegna l'indirizzo IP interno al livello *j* + 1 di *n*
        (non quando *j* + 1 = *l*; in quel caso abbiamo solo l'indirizzo IP globale).
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.  
                Si tenga presente che se un processo locale vuole inviare un pacchetto a questo
                indirizzo IP, allora come *src* preferito dovrà usare l'indirizzo IP globale di *n*.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.  
                Si tenga presente che se un processo locale vuole inviare un pacchetto a questo
                indirizzo IP, allora come *src* preferito dovrà usare l'indirizzo IP globale di *n*.
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    In questo caso è garantito che tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*, cosa necessaria affinché
                    l'indirizzo IP interno al livello *t* del g-nodo *d* abbia significato.  
                    Si tenga presente che se un processo locale vuole inviare un pacchetto a questo
                    indirizzo IP, allora come *src* preferito dovrà usare l'indirizzo IP
                    di *n* interno al livello *t*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d* e con *n<sub>x</sub>* l'indirizzo IP che
            il sistema *n* intende usare come *src* preferito.
            *   Il sistema imposta una rotta in *partenza* verso *d<sub>x</sub>*. In essa specifica
                l'indirizzo *n<sub>x</sub>* come *src* preferito.  
                Ricordiamo che nelle tabelle di routing del kernel si riporta come informazione solo
                il primo gateway di una rotta, sebbene l'identità sia a conoscenza di altre informazioni.  
                Viene impostata la rotta identificata dal miglior percorso noto all'identità per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile"
                perché abbiamo appena detto che l'identità conosce la destinazione *d*, quindi almeno
                un percorso verso *d*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*: l'identità potrebbe non conoscere nessun percorso
                    verso *d* che non passi per il massimo distinto g-nodo di *m* per *n*.
            *   Il sistema dovrebbe impostare una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                In realtà, per come funziona lo stack TCP/IP di Linux, tale rotta è superflua. Infatti
                essa sarebbe identica a quella che è stata impostata per i pacchetti in *partenza* verso
                *d<sub>x</sub>* e lo stack TCP/IP prende in considerazione questa anche per i pacchetti
                in *inoltro*. Perciò il programma *qspnclient* non imposta una ulteriore rotta.

Se è *virtuale*, significa che l'indirizzo ha una o più componenti virtuali. Sia *i* il livello
più basso in cui la componente è virtuale. Sia *k* il livello più alto in cui la componente è virtuale.

In questo caso, nel network namespace default:

*   NON esiste un indirizzo IP globale di *n*.
*   NON esiste un indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *i*-1:
    *   Il sistema si assegna l'indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    Questo indirizzo IP va calcolato solo se tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*, cosa necessaria affinché
                    l'indirizzo IP interno al livello *t* del g-nodo *d* abbia significato.  
                    Si tenga presente che se un processo locale vuole inviare un pacchetto a questo
                    indirizzo IP, allora come *src* preferito dovrà usare l'indirizzo IP
                    di *n* interno al livello *t*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d* e con *n<sub>x</sub>* l'indirizzo IP che
            il sistema *n* intende usare come *src* preferito.
            *   Il sistema imposta una rotta in *partenza* verso *d<sub>x</sub>*. In essa specifica
                l'indirizzo *n<sub>x</sub>* come *src* preferito.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema dovrebbe impostare una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce. Ma come detto
                sopra lo stack TCP/IP di Linux si avvale della rotta che è stata impostata per i pacchetti
                in *partenza* verso *d<sub>x</sub>*. Perciò il programma *qspnclient* non imposta una ulteriore rotta.
*   Per ogni livello *j* da *i* a *k*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    Questo indirizzo IP va calcolato solo se tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*.  
                    In questo caso è impossibile per un processo locale inviare un pacchetto a questo
                    indirizzo IP, non potendo usare un indirizzo IP
                    di *n* interno al livello *t*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d*.
            *   Il sistema NON ha una rotta in *partenza* verso *d<sub>x</sub>*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".
*   Per ogni livello *j* da *k* a *l*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.  
                In questo caso è impossibile per un processo locale inviare un pacchetto a questo
                indirizzo IP, non potendo usare l'indirizzo IP globale di *n*.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.  
                In questo caso è impossibile per un processo locale inviare un pacchetto a questo
                indirizzo IP, non potendo usare l'indirizzo IP globale di *n*.
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    In questo caso è garantito che tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*.  
                    In questo caso è impossibile per un processo locale inviare un pacchetto a questo
                    indirizzo IP, non potendo usare un indirizzo IP
                    di *n* interno al livello *t*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d*.
            *   Il sistema NON ha una rotta in *partenza* verso *d<sub>x</sub>*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".

### <a name="Identita_di_connettivita"></a>Identità di connettività

Un sistema può avere 0 o più identità di connettività. L'identità di connettività gestisce un certo
network namespace. L'identità di connettività ha un indirizzo Netsukuku *di connettività* che è *virtuale*.

Significa che l'indirizzo ha una o più componenti virtuali. Sia *i* il livello più basso in cui
la componente è virtuale. Sia *k* il livello più alto in cui la componente è virtuale.

Nel network namespace gestito da questa identità:

*   NON esiste un indirizzo IP globale di *n*.
*   NON esiste un indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *k* - 1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    Questo indirizzo IP va calcolato solo se tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*.  
                    In questo caso è impossibile che un processo locale voglia inviare un pacchetto a questo
                    indirizzo IP, poiché questa identità non è nel network namespace default.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d*.
            *   Il sistema NON ha una rotta in *partenza* verso *d<sub>x</sub>*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".
*   Per ogni livello *j* da *k* a *l*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.  
                Di nuovo, è impossibile che un processo locale voglia inviare un pacchetto a questi
                indirizzi IP, poiché questa identità non è nel network namespace default.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.  
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    In questo caso è garantito che tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d*.
            *   Il sistema NON ha una rotta in *partenza* verso *d<sub>x</sub>*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".

## <a name="Indirizzi_del_sistema"></a>Indirizzi IP di ogni identità nel sistema

Come abbiamo visto prima, in un sistema possono esistere diverse identità. Ogni identità detiene un
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
Infatti quando un sistema *a* vuole trasmettere un pacchetto IP ad un suo diretto vicino *b*, esso conosce
l'indirizzo IP di *b* e l'interfaccia di rete di *a* dove trasmettere. Il sistema *a* trasmette un frame Ethernet broadcast
su quel segmento di rete una richiesta: «chi ha l'indirizzo IP XYZ?». Il sistema *b* risponde indicando al
sistema *a* l'indirizzo MAC della sua interfaccia di rete. Quindi il sistema *a* può incapsulare il pacchetto
IP in un frame Ethernet unicast che riporta gli indirizzi MAC dell'interfaccia che trasmette e dell'interfaccia che deve ricevere.

Se il sistema *b* ha diverse interfacce di rete (all'interno di un unico network namespace) tutte
collegate allo stesso segmento di rete, il fatto
di associare diversi indirizzi IP a diverse interfacce può fornire un modo di identificare una precisa
interfaccia di rete a cui un pacchetto va trasmesso. Questo è usato dal modulo Qspn per distinguere
i pacchetti che vanno ricevuti da una pseudo-interfaccia, per questo gli indirizzi link-local sono diversi
su ogni interfaccia e sulle pseudo-interfacce.

Fatta questa premessa, come si comporta il programma?

Il programma *qspnclient*, quando crea una *identità*, a seconda del tipo di identità e del suo indirizzo
Netsukuku, come visto prima, computa gli indirizzi IP che si deve assegnare e li associa tutti
a ognuna delle interfacce di rete che gestisce quell'identità.

Inoltre, prima di rimuovere una identità, li rimuove da tutte le interfacce che gestisce quell'identità.

## <a name="Rotte_nelle_tabelle_di_routing"></a>Rotte nelle tabelle di routing

Il programma deve istruire le policy di routing del sistema (che di norma significa impostare delle
rotte nelle tabelle di routing) in modo da garantire questi comportamenti:

*   Se un processo locale vuole inviare un pacchetto ad un indirizzo IP *x* nello spazio destinato alla rete Netsukuku:
    *   L'indirizzo IP *x* identifica un indirizzo Netsukuku reale *d*. Può trattarsi di un
        indirizzo IP globale, globale anonimizzante o interno.
    *   Se il modulo QSPN non ha alcun percorso verso la destinazione *d*:
        *   Il sistema non trova nelle tabelle del kernel alcuna rotta e segnala al processo
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
        *   Il sistema non trova nelle tabelle del kernel alcuna rotta; quindi scarta il pacchetto
            e invia un pacchetto ICMP «host *x* irraggiungibile» al mittente.
    *   Altrimenti:
        *   Il sistema trova nelle tabelle del kernel una rotta il cui gateway è costituito dal primo
            hop per il miglior percorso verso la destinazione *d*, tale che non contenga il g-nodo del vicino.
        *   Se l'indirizzo IP *x* è globale anonimizzante:
            *   Opzionalmente, il sistema trova una regola che gli impone di mascherare l'indirizzo
                del mittente (cioè di indicare se stesso come mittente e inoltrare le comunicazioni di
                risposta che riceverà al vero mittente).

* * *

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa
il programma *qspnclient*.

Abbiamo già avuto modo di evidenziare che un sistema Linux è in grado di replicare il suo
intero network stack in molti distinti network namespace. E che i moduli che stiamo esaminando
assumono come requisito una capacità di questo tipo. Nella trattazione che segue parliamo sempre
di concetti (come le tabelle di routing, l'assegnazione degli indirizzi alle interfacce di rete,
la manipolazione dei pacchetti, ...) che sono da riferirsi ad un particolare network stack.

Esaminiamo prima l'aspetto del source natting e poi del routing.

### <a name="Source_natting"></a>Source NATting

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
il sistema si rende disponibile ad anonimizzare i pacchetti che riceve e che vanno inoltrati verso una
destinazione che accetta richieste anonime.

L'opzione di rendere anonimi i pacchetti che transitano per il sistema nel percorso verso un'altra destinazione
è distinta e indipendente dall'opzione di accettare richieste anonime, che è stata discussa sopra.

*   **NOTA**: La seguente spiegazione sui motivi per cui l'operazione è opzionale va spostata in un
    documento che affronta ad alto livello le implicazioni sul detenere un sistema nella rete Netsukuku. Il documento
    presente si limita a illustrare i dettagli implementativi del programma *qspnclient* o del demone *ntkd*.  
    Questa azione è opzionale perché il proprietario di un sistema può avere remore a nascondere il vero mittente
    di un messaggio prendendo il suo posto. In realtà questo timore sarebbe infondato, vediamo perché. Per far
    funzionare bene l'operazione di contatto anonimo da parte del client, occorre che il sistema che fa da server
    (fornisce un servizio) si assegni anche gli indirizzi per essere contattato in forma anonima. Se fa questa
    operazione opzionale, significa che è pronto a ricevere alcune richieste dalle quali saprà di non
    poter risalire al mittente. Sarà quindi responsabile di rispondere o meno a tali richieste e non
    potrà far ricadere tale responsabilità sugli altri sistemi.  
    Anche considerando quindi non rischiosa l'azione di implementare nel proprio sistema il source natting,
    l'azione è opzionale perché il sistema che la implementa si carica di un onere che costa un po' in
    termini di memoria. Se il sistema quindi ha scarse risorse (si intende molto scarse, come pochi mega
    di RAM) conviene che non la implementi.  
    Va considerato che se un sistema decide di non implementare questa azione, comunque il meccanismo di
    trasmissione anonima risulta efficace se nel percorso tra il mittente e il destinatario almeno un sistema è
    disposto a implementarla. Invece, se un sistema decide di implementare l'azione e ad un certo punto le sue risorse
    di memoria venissero meno, in questo caso la comunicazione in corso ne verrebbe compromessa.

Se il sistema decide di implementare il source natting, calcola lo spazio di indirizzi che indicano una
risorsa da raggiungere in forma anonima. Una volta calcolato il numero di bit necessari a codificare
un indirizzo Netsukuku *reale* nella topologia della nostra rete, va considerato che nei successivi
2 bit in testa va codificato (per gli indirizzi IP globali *anonimizzanti*) il numero 2, in binario `|1|0|`.

Facciamo un esempio. Supponiamo di destinare alla rete Netsukuku tutta la classe 10.0.0.0/8 di IPv4.
Consideriamo una topologia di rete con 4 livelli. Diamo 2 bit al livello 3, 4 bit al livello 2, 8 bit
ai livelli 1 e 0. Sono soddisfatti i vincoli esposti sopra.

In questo esempio, il range di indirizzi che individuano a livello globale una risorsa da
raggiungere in forma anonima è `10.128.0.0/10`.

Supponiamo che l'indirizzo Netsukuku del nostro sistema in questa topologia sia 3·10·123·45.
L'indirizzo IP globale del sistema è 10.58.123.45. L'indirizzo IP globale *anonimizzante* del sistema è 10.186.123.45.

Allora il programma istruisce il kernel di modificare i pacchetti destinati al range `10.128.0.0/10`
indicando come nuovo indirizzo mittente il suo indirizzo globale (non quello *anonimizzante*). Il comando è il seguente:

```
   iptables -t nat -A POSTROUTING -d 10.128.0.0/10 -j SNAT --to 10.58.123.45
```

Quando il programma termina, se aveva istruito il kernel per fare il source natting, rimuove le regole
che aveva messe nella catena `POSTROUTING` della tabella `nat`.

### <a name="Routing"></a>Routing

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

Occorre evidenziare che, in presenza di molteplici network namespace, (di default) il file che
associa il nome mnemonico della tabella al suo numero, `/etc/iproute2/rt_tables`, è comune a tutti
i namespace. Invece le regole di scelta della tabella da esaminare e il contenuto delle tabelle
è distinto in ogni namespace.

Fatta questa premessa, come si comporta il programma?

Il programma, attraverso i moduli Neighborhood e Identities, ha già automaticamente ottenuto
che nella tabella `main` di ogni network namespace siano memorizzate rotte dirette (cioè senza gateway)
verso ogni suo diretto vicino (più precisamente verso ogni *identità* sua vicina). In queste rotte
sono indicati gli indirizzi di scheda delle proprie interfacce e di quelle dei vicini.

Il programma crea una tabella `ntk` con identificativo `YYY`, dove `YYY` è il primo identificativo
libero nel file `/etc/iproute2/rt_tables`. Tale tabella sarà inizialmente vuota; in essa il programma
andrà a mettere le rotte di pertinenza della rete Netsukuku, cioè quelle con destinazione nello
spazio 10.0.0.0/8. Inoltre aggiunge una regola che dice di guardare la tabella `ntk` prima della `main`.

Il programma, per ogni suo arco, crea un'altra tabella chiamata `ntk_from_XXX` con identificativo `YYY`,
dove `XXX` è il MAC address del sistema vicino, `YYY` è il primo identificativo libero nel
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
segnalati dal modulo QSPN contengono sempre un arco che parte dal sistema corrente come passo iniziale e da tale arco
si può risalire all'indirizzo di scheda del vicino. Le rotte nelle tabelle `ntk` e `ntk_from_XXX` infatti
devono avere come campo gateway (gw) l'indirizzo di scheda del vicino, non il suo indirizzo Netsukuku.

Ogni destinazione nota ad una identità è un g-nodo. Per ogni destinazione il programma *qspnclient*
sceglie (per ogni tabella come descritto sopra) il miglior percorso.

Per ogni percorso scelto dal programma *qspnclient* per entrare in una tabella, in realtà il programma
inserisce nella tabella un numero di rotte pari al numero di indirizzi IP che la mappatura di
cui abbiamo parlato sopra associa ad un indirizzo Netsukuku *reale*. Sia *i* il livello del g-nodo
destinazione del percorso. Queste sono le rotte:

*   Indirizzo IP globale del g-nodo.
*   Indirizzo IP globale anonimizzante del g-nodo.
*   Per ogni valore *t* da *i* + 1 a *l* - 1:
    *   Indirizzo IP interno al livello *t* del g-nodo.

Quando il programma ha finito di usare una tabella (ad esempio se un arco che conosceva non è più presente,
oppure se il programma termina) svuota la tabella, poi rimuove la regola, poi rimuove il record
relativo dal file `/etc/iproute2/rt_tables`.

