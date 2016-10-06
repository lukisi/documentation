# Proof of concept - Analisi Funzionale

1.  [Ruolo del qspnclient](#Ruolo_del_qspnclient)
1.  [Operazioni del qspnclient](#Operazioni_del_qspnclient)
    1.  [Interazione programma-utente](#Interazione_programma_utente)
    1.  [Avvio del programma](#Avvio_programma)
    1.  [Primo segnale `bootstrap_complete`](#Primo_bootstrap_complete)
    1.  [Archi con i sistemi vicini](#Archi_vicini)
    1.  [Ingresso in una rete - Caso 1](#Ingresso_rete_1)
    1.  [Un nuovo vicino nella stessa rete viene rilevato](#Nuovo_vicino_stessa_rete)
    1.  [Un arco con un vicino nella stessa rete viene rimosso](#Rimosso_vicino_stessa_rete)
1.  [Vecchio](#Vecchio)
    1.  [Primi passi](#Primi_passi)
    1.  [Da riordinare](#Da_riordinare)
1.  [Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IPv4](#Mappatura_indirizzi_ip)
1.  [Identità](#Identita)
    1.  [Assegnazione indirizzi IP](#Indirizzi_ip_propri)
    1.  [Assegnazione rotte - Identità principale](#Rotte_Identita_principale)
    1.  [Assegnazione rotte - Identità di connettività](#Rotte_Identita_di_connettivita)
1.  [Indirizzi IP di ogni identità nel sistema](#Indirizzi_del_sistema)
1.  [Rotte nelle tabelle di routing](#Rotte_nelle_tabelle_di_routing)
    1.  [Source NATting](#Source_natting)
    1.  [Routing](#Routing)
    1.  [Mappatura di una sottorete](#Net_mapping)

Ci proponiamo di realizzare un programma, **qspnclient**, che si avvale del modulo QSPN e altri
moduli a sostegno (Neighborhood, Identities) per stabilire come impostare le rotte nelle tabelle
di routing del kernel di una macchina. Si tratta di un programma specifico per un sistema Linux.

Per giungere a delineare le regole generali descritte in questo documento si è proceduto ad un
esame dettagliato delle operazioni da fare su un sistema a fronte dei vari casi d'uso. Questo lavoro
è riportato nel documento [Casi d'uso](UseCases.md).

In seguito si è cercato di associare ogni operazione ad uno specifico momento del ciclo di vita
del programma **qspnclient**: il suo avvio, la rilevazione di qualche segnale da un dato modulo,
le richieste da parte dell'utente. Questo lavoro è riportato nel documento [Eventi](Eventi.md).

## <a name="Ruolo_del_qspnclient"></a>Ruolo del qspnclient

Questo programma permette all'utente di fare le veci del demone *ntkd*, simulando le sue operazioni
e quelle di pertinenza di altri moduli:

*   Il dialogo con identità in sistemi vicini appartenenti ad altre reti.
*   Il coordinamento di un g-nodo (moduli PeerServices e Coordinator).
*   La strategia di ingresso in una rete, cioè la scelta del g-nodo *g* in cui chiedere un posto.
*   La ricerca della più breve *migration path* per liberare un posto in *g* (modulo Migrations).
*   La comunicazione delle informazioni a tutte le identità interessate dalla migration path trovata.

Il programma *qspnclient* interagisce con l'utente (il quale ha a disposizione alcuni meccanismi per dare istruzioni
al programma) e con i moduli suddetti: QSPN, Neighborhood, Identities. Sulla base delle informazioni ottenute tramite
queste interazioni, interviene sulle configurazioni di rete del sistema. L'utente sarà quindi in grado di verificare che il
sistema riesca effettivamente a stabilire connessioni con gli altri sistemi della rete, che le rotte
siano quelle che ci si attende, eccetera. Inoltre il programma consente all'utente di chiedere la visualizzazione
di tutte le informazioni che il modulo QSPN ha raccolto.

## <a name="Operazioni_del_qspnclient"></a>Operazioni del qspnclient

### <a name="Interazione_programma_utente"></a>Interazione programma-utente

L'utente avvia il programma *qspnclient* su un sistema 𝛼 eseguendo su una shell il comando *qspnclient init*. In
questo momento fornisce alcuni dati iniziali come argomenti. Il comando avvia il programma *qspnclient* e
non restituisce il controllo della shell all'utente; la utilizza invece per visualizzare alcune informazioni
utili durante le sue operazioni.

Per le successive comunicazioni con il programma, l'utente dovrà aprire una nuova shell sul sistema 𝛼 e
da questa dare altri comandi (ad esempio *qspnclient enter_net*, ...) e con essi altri dati. Questi comandi
e informazioni saranno comunicate al programma *qspnclient* già in esecuzione, poi il comando restituirà
all'utente la shell, eventualmente dopo aver visualizzato le informazioni pertinenti.

Ai parametri che saranno individuati in modo autonomo dai moduli (ad esempio gli identificativi di
nodo, gli indirizzi di scheda, ...) verranno associati degli indici progressivi che saranno visualizzati
all'utente. L'utente si riferirà ad essi tramite questi indici. Questo per rendere più facilmente
riproducibili gli ambienti di test.

### <a name="Avvio_programma"></a> Avvio del programma

[Dettagli](DettagliOperazioni1.md)

All'avvio del programma **qspnclient** l'utente specifica alcune informazioni persistenti che riguardano
il sistema e la rete a cui intende collegarsi.

Specifica anche l'indirizzo iniziale per il sistema. Tale indirizzo è temporaneo, almeno nella parte
dei livelli alti; come tale potrà essere scelto dal programma in modo casuale nel demone *ntkd*. Ma la
parte dei livelli bassi, se il sistema vuole fare da gateway per una sottorete a gestione autonoma,
deve comunque poter essere specificata dall'utente.

Il programma **qspnclient** all'avvio inizializza il network namespace default del sistema come unico
nodo di una nuova rete. In seguito farà ingresso in un'altra rete o saranno altri nodi ad unirsi alla
sua rete.

### <a name="Primo_bootstrap_complete"></a> Primo segnale `bootstrap_complete`

[Dettagli](DettagliOperazioni2.md)

Immediatamente, poiché il sistema è inizialmente isolato, l'identità principale riceve dal QspnManager
il segnale `bootstrap_complete`. Su questo segnale il programma **qspnclient** aggiorna le rotte
del network namespace in base alle sue conoscenze; per il momento è un nodo isolato.

### <a name="Archi_vicini"></a> Archi con i sistemi vicini

[Dettagli](DettagliOperazioni3.md)

Il modulo Neighborhood rileva i sistemi diretti vicini del sistema corrente, ma è il programma
**qspnclient** che li mostra all'utente il quale decide se utilizzarli o meno. In questo modo
il programma permette all'utente di dirigere il proprio ambiente di test a piacimento per simulare
particolari scenari. Ad esempio, l'utente può far girare il programma su un gruppo di macchine virtuali
che condividono un unico dominio di broadcast. L'utente è messo in grado di simulare in tale
scenario un gruppo di sistemi wireless disposti in un determinato modo, decidendo quali sistemi formano
degli archi e con quale costo (latenza).

L'utente quando accetta un arco dice quale costo gli vuole associare. In seguito può
variarlo a piacimento fino anche a simularne la rimozione.

### <a name="Ingresso_rete_1"></a> Ingresso in una rete - Caso 1

[Dettagli](DettagliOperazioni4.md)

Quando due reti si incontrano, il programma **qspnclient** prevede che sia l'utente a simulare i meccanismi
di dialogo che portano a decidere:

*   Quale g-nodo fa ingresso in blocco nell'altra rete.
*   Quale g-nodo lo ospiterà nella nuova rete e con che posizione.
*   Quale migration path (eventualmente) porterà a liberare quella posizione.

Sarà ancora l'utente a simulare anche i meccanismi che portano all'avvio delle operazioni nei
vari singoli sistemi con le dovute informazioni.

Esaminiamo il caso più banale: sia *𝛼* un singolo nodo che costituiva una rete a sé; esso si incontra con un
diverso singolo nodo *𝛽*; il nodo *𝛽* appartiene ad un g-nodo di livello 1 che ha una posizione
libera per *𝛼*.

Dopo che l'utente ha istruito il sistema *𝛼* di fare ingresso, il programma **qspnclient** opera:

*   la duplicazione dell'identità,
*   lo spostamento della vecchia identità in un nuovo namespace temporaneo,
*   la preparazione del vecchio namespace per la nuova identità e le prime operazioni della nuova
    identità per l'effettivo ingresso nella rete,
*   la dismissione della vecchia identità.

### <a name="Nuovo_vicino_stessa_rete"></a> Un nuovo vicino nella stessa rete viene rilevato

[Dettagli](DettagliOperazioni5.md)

Dall'altra parte, una identità nel sistema *𝛽* rileva un nuovo vicino *𝛼* che inizialmente non fa parte della sua
rete. Poi viene a sapere (tramite il modulo Identities) che una nuova identità di *𝛼* fa adesso parte della
sua rete.

In questa occasione il programma **qspnclient** prepara una tabella di inoltro per i pacchetti che
provengono dal nuovo MAC address rilevato; però questa tabella di inoltro non viene attivata fino a
quando l'identità di *𝛽* non riceve il primo ETP dalla nuova identità di *𝛼*.

### <a name="Rimosso_vicino_stessa_rete"></a> Un arco con un vicino nella stessa rete viene rimosso

[Dettagli](DettagliOperazioni5.md#Rimosso_vicino_stessa_rete)

Ci sono alcune situazioni in cui una identità in un sistema deve rimuovere un arco-qspn.

*   Un arco fisico non è più funzionante. Il modulo Neighborhood lo rileva e lo segnala.  
    Può succedere anche nell'ambiente di test, per esempio se il demone nel sistema vicino va in crash.
*   L'utente nell'ambiente di test vuole simulare che un arco fisico non sia più funzionante. In questo
    caso non è il modulo Neighborhood a segnalarlo, ma l'utente a istruire direttamente il programma.
*   Un sistema vicino rimuove un suo arco-identità con una identità di questo sistema.  
    Può essere che l'identità del vicino è una identità di connettività e vuole rimuovere questo arco
    perché è esterno al g-nodo di sua pertinenza.  
    Può essere anche che l'identità del vicino è l'identità principale e il vicino vuole terminare
    il demone (gracefully).
*   Questo sistema richiede per una sua identità la rimozione di un arco-identità.  
    Le possibili ragioni sono le stesse viste sopra.

Analiziamo il caso in cui un arco fisico non sia più funzionante (realmente o per simulazione).

Il programma **qspnclient** riceve dapprima il segnale `arc_removing` dal modulo Neighborhood (o una
equivalente istruzione dall'utente).

Il programma **qspnclient** grazie al modulo Identities individua tutti gli archi-identità che sono
realizzati su questo arco fisico. Per ognuno di essi si occupa di rimuovere le tabelle di inoltro e poi di
istruire il modulo Qspn.

## <a name="Vecchio"></a>Vecchio

### <a name="Primi_passi"></a>Primi passi

All'avvio del programma nel sistema viene creata l'istanza di NeighborhoodManager. Poi gli sono passati i nomi delle
interfacce di rete che dovrà gestire, tramite il metodo *start_monitor*. Ad ogni interfaccia, il modulo Neighborhood
associa un indirizzo IP link-local scelto a caso.

Il programma si avvede dell'indirizzo scelto perché il NeighborhoodManager lo notifica con il segnale
*nic_address_set*. Il programma associa questo proprio indirizzo link-local all'indice
autoincrementante *linklocal_nextindex*, che parte da 0.

* * *

Per ogni arco che il modulo Neighborhood realizza, le informazioni a disposizione (i due link-local e
i due MAC address) sono visualizzate all'utente. Soltanto agli archi che l'utente decide di accettare
e nell'ordine in cui sono accettati, il programma associa un indice autoincrementante *nodearc_nextindex*,
che parte da 0. In seguito il programma sfrutta questi archi passandoli al modulo Identities.

* * *

All'avvio del programma nel sistema viene creata l'istanza di IdentityManager. Questi nel
costruttore crea la prima identità *principale* del sistema e per essa genera un NodeID casuale. Il programma
recupera tale NodeID col metodo *get_main_id()* e lo associa all'indice autoincrementante *nodeid_nextindex*,
che parte da 0. In seguito il programma quando crea una nuova identità col metodo *add_identity* associa la
nuova istanza di NodeID al prossimo valore di *nodeid_nextindex*. Quindi l'utente può usare questo indice per
dare dalla console comandi concernenti una certa identità.

### <a name="Da_riordinare"></a>Da riordinare

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
*   Un indirizzo IP interno al livello *i* per ogni valore di *i* da 0 a *l* - 1.  
    Questo indirizzo IP identifica un preciso *sistema* univocamente all'interno
    di un g-nodo *g* di livello *i*. Questo indirizzo IP si può utilizzare come indirizzo di
    destinazione di un pacchetto IP quando sia il *sistema* mittente che il *sistema*
    identificato (il destinatario) appartengono allo stesso g-nodo *g* di livello *i*. La
    peculiarità di questi indirizzi IP (che si riflette sui pacchetti IP trasmessi a questi
    indirizzi) è che essi non cambiano quando il g-nodo *g* o uno dei suoi g-nodi superiori
    *migra* all'interno della rete o anche in una diversa rete Netsukuku.  
    Anche per il livello 0 si calcola un tale indirizzo IP. Questo indirizzo ha un significato
    simile a `localhost` nel senso che identifica come destinazione lo stesso sistema mittente.

Ricordiamo che un indirizzo Netsukuku identifica un *nodo del grafo*, cioè una specifica identità
all'interno di un *sistema*. Però in ogni sistema, in ogni momento, esiste una ed una sola
*identità principale*, che è l'unica che detiene un indirizzo Netsukuku *reale*.

Per l'esattezza, l'identità principale di un sistema può detenere per brevi istanti un
indirizzo Netsukuku *virtuale*, durante le operazioni di una migrazione coinvolta in una
*migration path*. Durante questo periodo quel sistema (e insieme a quello anche tutti
gli altri sistemi che appartengono al g-nodo *g* che sta migrando) non può avere un indirizzo
IP globale. Questo però non inficia sulla possibilità di quel sistema di avere un indirizzo
IP interno al livello *i* per ogni valore di *i* da 0 a *k*, dove *k* è il livello
del g-nodo *g*.

Questo permette, come detto prima, che le connessioni realizzate tra due sistemi appartenenti
al g-nodo *g* non vengano compromesse. Per questo aggiungiamo che la mappatura associa alcuni
indirizzi IP anche ad un indirizzo Netsukuku *virtuale* (ma solo per l'identità principale), purché
siano *reali* i suoi identificativi minori di *k*. Questi sono:

*   Un indirizzo IP interno al livello *i* per ogni valore di *i* da 0 a *k*.

Gli algoritmi di calcolo dei vari tipi di indirizzo IP sono descritti nel documento [IndirizziIP](IndirizziIP.md).

## <a name="Identita"></a> Identità

Ogni identità che vive nel sistema ha un suo indirizzo Netsukuku. Inoltre ha una mappa di percorsi, ognuno
che ha come destinazione (e come passi) un g-nodo *visibile* dal suo indirizzo Netsukuku.

Un sistema ha sempre una identità principale e zero o più identità di connettività.

L'identità principale gestisce il network namespace default. L'identità principale ha un indirizzo
Netsukuku *definitivo* che può essere *reale* o *virtuale*.

L'identità di connettività gestisce un certo network namespace. L'identità di connettività ha un indirizzo
Netsukuku *di connettività* che è *virtuale*.

### <a name="Indirizzi_ip_propri"></a> Assegnazione indirizzi IP

Le identità di connettività non si assegnano mai nessun indirizzo IP nel loro network namespace.

L'identità principale, nel network namespace default, si assegna degli indirizzi IP sulla base del
suo indirizzo Netsukuku.

Sia *n* l'indirizzo Netsukuku dell'identità principale di un sistema. Sia *i* il livello più basso in
cui la componente di *n* è virtuale. Diciamo che *i* vale *l* se *n* è del tutto *reale*.

*   Per ogni livello *j* da 0 a *i*:
    *   Se *j* = *l*:
        *   Il sistema si assegna l'indirizzo IP globale di *n*.
        *   Il sistema può (opzionalmente) fare da anonimizzatore. Cioè si aggiunge la regola di SNAT.
        *   Il sistema può (opzionalmente) assegnarsi l'indirizzo IP globale anonimizzante di *n*.
    *   Altrimenti:
        *   Il sistema si assegna l'indirizzo IP interno al livello *j* di *n*.

### <a name="Rotte_Identita_principale"></a> Assegnazione rotte - Identità principale

Sia *n* l'indirizzo Netsukuku dell'identità principale. Se *n* è *reale*, nel network namespace default:

*   Per ogni livello *j* da 0 a *l* - 1:
    *   Per ogni componente *reale* a livello *j*, cioè per *p* da 0 a *gsize(j)* - 1:
        *   Sia *d* il g-nodo di coordinate (*j*, *p*) rispetto a *n*. Indipendentemente dal
            fatto che *d* esista o meno nella rete.
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

Se *n* è *virtuale*, significa che ha una o più componenti virtuali. Sia *i* il livello
più basso in cui la componente è virtuale. Sia *k* il livello più alto in cui la componente è virtuale.

In questo caso, nel network namespace default:

*   Per ogni livello *j* da 0 a *i* - 1:
    *   Per ogni componente *reale* a livello *j*, cioè per *p* da 0 a *gsize(j)* - 1:
        *   Sia *d* il g-nodo di coordinate (*j*, *p*) rispetto a *n*. Indipendentemente dal
            fatto che *d* esista o meno nella rete.
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
*   Per ogni livello *j* da *i* a *k* - 1:
    *   Per ogni componente *reale* a livello *j*, cioè per *p* da 0 a *gsize(j)* - 1:
        *   Sia *d* il g-nodo di coordinate (*j*, *p*) rispetto a *n*. Indipendentemente dal
            fatto che *d* esista o meno nella rete.
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
*   Per ogni livello *j* da *k* a *l* - 1:
    *   Per ogni componente *reale* a livello *j*, cioè per *p* da 0 a *gsize(j)* - 1:
        *   Sia *d* il g-nodo di coordinate (*j*, *p*) rispetto a *n*. Indipendentemente dal
            fatto che *d* esista o meno nella rete.
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

### <a name="Rotte_Identita_di_connettivita"></a> Assegnazione rotte - Identità di connettività

Sia *n* l'indirizzo Netsukuku di un'identità di connettività. 
Sicuramente *n* ha una o più componenti virtuali. Sia *i* il livello più basso in cui
la componente è virtuale. Sia *k* il livello più alto in cui la componente è virtuale.

Nel network namespace gestito da questa identità:

*   Per ogni livello *j* da 0 a *k* - 1:
    *   Per ogni componente *reale* a livello *j*, cioè per *p* da 0 a *gsize(j)* - 1:
        *   Sia *d* il g-nodo di coordinate (*j*, *p*) rispetto a *n*. Indipendentemente dal
            fatto che *d* esista o meno nella rete.
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
*   Per ogni livello *j* da *k* a *l* - 1:
    *   Per ogni componente *reale* a livello *j*, cioè per *p* da 0 a *gsize(j)* - 1:
        *   Sia *d* il g-nodo di coordinate (*j*, *p*) rispetto a *n*. Indipendentemente dal
            fatto che *d* esista o meno nella rete.
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

## <a name="Indirizzi_del_sistema"></a> Indirizzi IP dell'identità principale del sistema

Come abbiamo visto prima, in un sistema possono esistere diverse identità. Ogni identità detiene un
indirizzo Netsukuku. Ma solo l'identità *principale* del sistema, sulla base del suo indirizzo Netsukuku,
si può assegnare zero o più indirizzi IPv4 che gli permetteranno di comunicare con il resto della rete.

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa il programma *qspnclient*.

In un sistema Linux il sistema si assegna un indirizzo IP associandolo ad una interfaccia di rete.

Fatta questa premessa, come si comporta il programma?

Il programma **qspnclient** sa quando può cambiare l'indirizzo Netsukuku dell'identità *principale* del
sistema:

*   All'avvio del programma. In questo momento si costituisce la prima identità nel sistema, che è
    in quel momento la principale; ad essa viene associato il primo indirizzo Netsukuku che è
    completamente *reale*.
*   Quando l'identità principale si duplica. In questo momento la vecchia identità diventa di connettività
    e il posto di identità principale viene preso dalla nuova identità; ad essa viene associato un
    nuovo indirizzo Netsukuku che ha **almeno** una componente *virtuale*: quella al livello direttamente
    inferiore al  g-nodo che ha ospitato l'ultima migrazione/ingresso. Ne potrebbe avere di più
    se in quel momento una sua componente era già *virtuale*.
*   Quando l'identità principale cambia uno dei componenti del suo indirizzo Netsukuku da *virtuale*
    a *reale*.

In queste occasioni il programma **qspnclient** computa gli indirizzi IP che si deve assegnare il
sistema e quelli che si deve rimuovere. Li associa/rimuove tutti a ognuna delle interfacce di rete reali.

## <a name="Rotte_nelle_tabelle_di_routing"></a> Rotte nelle tabelle di routing

Il programma deve istruire le policy di routing del sistema (che di norma significa impostare delle
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

* * *

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa
il programma *qspnclient*.

Abbiamo già avuto modo di evidenziare che un sistema Linux è in grado di replicare il suo
intero network stack in molti distinti network namespace. E che i moduli che stiamo esaminando
assumono come requisito una capacità di questo tipo. Nella trattazione che segue parliamo sempre
di concetti (come le tabelle di routing, l'assegnazione degli indirizzi alle interfacce di rete,
la manipolazione dei pacchetti, ...) che sono da riferirsi ad un particolare network stack.

Esaminiamo prima l'aspetto del source natting, che permette al sistema (se lo vuole fare) di
mascherare il source dei pacchetti che hanno una destinazione *anonimizzante*. Poi esaminiamo l'aspetto
del routing. Infine esaminiamo l'aspetto del net mapping, che permette ad un sistema di fare
efficacemente da gateway verso una sottorete a gestione autonoma.

### <a name="Source_natting"></a> Source NATting

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

### <a name="Routing"></a> Routing

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

Il programma, sempre su ogni network namespace, crea una tabella `ntk` con identificativo `YYY`, dove `YYY`
è il primo identificativo libero nel file `/etc/iproute2/rt_tables`. In essa mette tutte le rotte
delle possibili destinazioni IP in base all'indirizzo Netsukuku dell'identità che gestisce quel
namespace.

Il ruolo fondamentale della tabella `ntk` è svolto nel network namespace default (**TODO** rimuovere
la tabella `ntk` negli altri namespace nel test e verificare che vada bene). I processi locali
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
programma *qspnclient* popola e mantiene le rotte nelle tabelle `ntk` e `ntk_from_XXX`. I percorsi
segnalati dal modulo QSPN contengono sempre un arco che parte dal sistema corrente come passo iniziale e da tale arco
si può risalire all'indirizzo di scheda del vicino. Le rotte nelle tabelle `ntk` e `ntk_from_XXX` infatti
devono avere come campo gateway (gw) l'indirizzo di scheda del vicino.

Quando il programma ha finito di usare una tabella (ad esempio se un arco che conosceva non è più presente,
oppure se il programma termina) svuota la tabella, poi rimuove la regola, poi rimuove il record
relativo dal file `/etc/iproute2/rt_tables`.

### <a name="Net_mapping"></a> Mappatura di una sottorete

Fra le possibilità offerte da `iptables` c'è l'estensione
[NETMAP](https://www.netfilter.org/documentation/HOWTO/netfilter-extensions-HOWTO-4.html#ss4.4)
che ci permette di creare una mappatura 1:1
tra due reti. Cioè di modificare la parte `network` di un indirizzo IP mantenendo inalterata la parte `host`.

Abbinata alla catena `PREROUTING` della tabella `nat` questa estensione permette di cambiare l'indirizzo
IP di destinazione di un pacchetto, mentre abbinata alla catena `POSTROUTING` della stessa tabella `nat`
permette di cambiare l'indirizzo del mittente.  
[Esempio](https://capcorne.wordpress.com/2009/03/24/natting-a-network-range-with-netmapiptables)

Fatta questa premessa, come si comporta il programma?

Il programma **qspnclient**, se il sistema vuole fare da gateway per una sottorete a gestione
autonoma, quando l'identità principale del sistema assume un nuovo indirizzo
Netsukuku, partendo dal livello subito superiore  ad ogni livello

*   Sia *gwl* il livello del g-nodo rappresentato dalla sottorete autonoma.
*   Sia *range1* l'indirizzo IP con suffisso CIDR che rappresenta la sottorete
    autonoma dentro il suo g-nodo di livello *gwl*.  
    Ad esempio `10.0.0.40/31` se *gwl*=1.
*   Sia *n* l'indirizzo Netsukuku dell'identità principale del sistema.
*   Per *i* che sale da *gwl* + 1 a *l* - 1:
    *   Se *n* ha componenti reali da 0 a *i* - 1, cioè è valido dentro il g-nodo di livello *i*:
        *   Sia *range2* l'indirizzo IP con suffisso CIDR che rappresenta la sottorete
            autonoma dentro il suo g-nodo di livello *i*. Si basa sulle posizioni di *n*
            da *gwl* a *i* - 1.  
            Ad esempio `10.0.0.50/31` se *gwl*=1 e *i*=2.  
            Oppure `10.0.0.62/31` se *gwl*=1 e *i*=3.
        *   Sia *g* il g-nodo di livello *i* di cui fa parte *n*.  
            Sia *range3* l'indirizzo IP con suffisso CIDR che comprende l'insieme di tutti
            i nodi in *g* rappresentati con un indirizzo IP interno al g-nodo *g*. Si basa sulla posizione di *n*
            al livello *i* - 1.  
            Ad esempio `10.0.0.48/30` se *i*=2.  
            Oppure `10.0.0.56/29` se *i*=3.
        *   Il programma esegue:  
            `iptables -t nat -A PREROUTING -d $range2 -j NETMAP --to $range1`  
            `iptables -t nat -A POSTROUTING -d $range3 -s $range1 -j NETMAP --to $range2`
*   Se *n* ha componenti reali da 0 a *l* - 1, cioè è del tutto reale:
    *   Sia *range2* l'indirizzo IP con suffisso CIDR che rappresenta la sottorete
        autonoma dentro tutta la rete Netsukuku. Si basa sulle posizioni di *n*
        da *gwl* a *l* - 1.  
        Ad esempio `10.0.0.22/31` se *gwl*=1.
    *   Sia *range3* l'indirizzo IP con suffisso CIDR che comprende l'insieme di tutti
        i nodi nella rete Netsukuku rappresentati con indirizzo IP globale.  
        Ad esempio `10.0.0.0/27`.
    *   Il programma esegue:  
        `iptables -t nat -A PREROUTING -d $range2 -j NETMAP --to $range1`  
        `iptables -t nat -A POSTROUTING -d $range3 -s $range1 -j NETMAP --to $range2`
    *   Se ogni sistema nella sottorete autonoma accetta di essere contattato in forma anonima:
        *   Sia *range4* l'indirizzo IP con suffisso CIDR che rappresenta la sottorete
            autonoma dentro tutta la rete Netsukuku con indirizzo IP anonimizzante. Si basa sulle posizioni di *n*
            da *gwl* a *l* - 1.  
            Ad esempio `10.0.0.86/31` se *gwl*=1.
        *   Il programma esegue:  
            `iptables -t nat -A PREROUTING -d $range4 -j NETMAP --to $range1`  
    *   Sia *range5* l'indirizzo IP con suffisso CIDR che comprende l'insieme di tutti
        i nodi nella rete Netsukuku rappresentati con indirizzo IP anonimizzante.  
        Ad esempio `10.0.0.64/27`.
    *   Il programma esegue:  
        `iptables -t nat -A POSTROUTING -d $range5 -s $range1 -j NETMAP --to $range2`

