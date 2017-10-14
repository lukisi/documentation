# Proof of concept - Analisi Funzionale

1.  [Ruolo del qspnclient](#Ruolo_del_qspnclient)
1.  [Operazioni del qspnclient](#Operazioni_del_qspnclient)
    1.  [Interazione programma-utente](#Interazione_programma_utente)
    1.  [Avvio del programma](#Avvio_programma)
    1.  [Archi con i sistemi vicini](#Archi_vicini)
    1.  [Ingresso in una rete - Caso 1](#Ingresso_rete_1)
    1.  [Un nuovo vicino nella stessa rete viene rilevato](#Nuovo_vicino_stessa_rete)
    1.  [Un arco con un vicino nella stessa rete viene rimosso](#Rimosso_vicino_stessa_rete)
    1.  [Ingresso in una rete - Caso 2](#Ingresso_rete_2)
    1.  [Migrazione per ingresso - Caso 1](#Migrazione_ingresso_1)
    1.  [Migrazione per ingresso - Caso 2](#Migrazione_ingresso_2)
1.  [Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IPv4](#Mappatura_indirizzi_ip)
1.  [Identità](#Identita)
    1.  [Assegnazione indirizzi IP](#Indirizzi_ip_propri)
    1.  [Assegnazione rotte](#Assegnazione_rotte)
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
*   La ricerca della più breve *migration path* per liberare un posto in *g* (modulo Hooking).
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

Il programma **qspnclient** all'avvio crea (tramite il modulo Identities) la prima identità *principale*
del sistema. A prescindere dall'oggetto che il modulo Identities usa per riferirsi ad una identità
nel sistema, il programma associa a questa prima identità l'indice autoincrementante *nodeid_nextindex*,
che parte da 0. In seguito il programma quando crea una nuova identità col metodo *add_identity* del
modulo Identities, associa alla nuova identità il prossimo valore di *nodeid_nextindex* e lo mostra
all'utente. Quindi l'utente può usare questo indice per dare al programma **qspnclient** comandi
concernenti una certa identità.

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

L'utente istruisce il sistema *𝛼* di fare ingresso nella rete in un dato g-nodo. Poi l'utente istruisce
il sistema *𝛽* sulla presenza di un nuovo arco nella rete, tramite il quale esso può trasmettere
e ricevere degli ETP per esplorare la rete.

Dopo che l'utente ha istruito il sistema *𝛼* di fare ingresso, il programma **qspnclient** opera:

*   la duplicazione dell'identità,
*   lo spostamento della vecchia identità in un nuovo namespace temporaneo,
*   la preparazione del vecchio namespace per la nuova identità,
*   la costituzione di nuove tabelle per l'inoltro sulla base degli archi nuovi,
*   la dismissione della vecchia identità.

Dopo che l'utente ha istruito il sistema *𝛽* sulla presenza di un nuovo arco nella rete, il programma **qspnclient** opera:

*   la costituzione di una nuova tabella per l'inoltro.

Dopo che sono stati processati nuovi ETP (sia in *𝛼* che in *𝛽*) il programma **qspnclient** opera:

*   l'aggiornamento delle rotte nelle varie tabelle.

### <a name="Nuovo_vicino_stessa_rete"></a> Un nuovo vicino nella stessa rete viene rilevato

[Dettagli](DettagliOperazioni5.md)

Quando il modulo Identities costituisce l'arco-identità principale sopra un arco fisico, le due identità
collegate potrebbero essere già nella stessa rete. In questo caso l'utente deve istruire
entrambi i sistemi sulla presenza di un nuovo arco nella rete.

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
    perché è esterno al g-nodo di sua pertinenza. Oppure l'identità viene dismessa.  
    Può essere anche che l'identità del vicino è l'identità principale e il vicino vuole terminare
    il demone (gracefully).
*   Questo sistema richiede per una sua identità la rimozione di un arco-identità.  
    Le possibili ragioni sono le stesse viste sopra.

In ogni caso, quando il programma **qspnclient** si avvede della rimozione di un arco-identità
il quale era associato ad un arco-qspn, si occupa di rimuovere le tabelle di inoltro e poi di
istruire il modulo Qspn.

### <a name="Ingresso_rete_2"></a> Ingresso in una rete - Caso 2

[Dettagli](DettagliOperazioni6.md)

Esaminiamo un altro caso di incontro di due reti distinte. Sia *𝜑* un g-nodo di livello *i* internamente
connesso costituito da alcuni nodi, ad esempio *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>*. Avvenga che tale
g-nodo si incontra per effetto di qualche arco con una diversa rete. Sia *𝜒* un g-nodo di questa rete
di livello *k* maggiore di *i* che ha un posto *reale* libero al livello *k* - 1 per *𝜑'*.

L'utente istruisce il sistema *𝛿* di fare ingresso, insieme al suo g-nodo di livello 1, nella rete in un dato g-nodo.
L'utente istruisce il sistema *𝜇* di fare ingresso, insieme al suo g-nodo di livello 1, nella rete in un dato g-nodo.
Poi l'utente istruisce i sistemi della nuova rete sulla presenza di nuovi archi nella rete.

### <a name="Migrazione_ingresso_1"></a> Migrazione per ingresso - Caso 1

[Dettagli](DettagliOperazioni7.md)

A volte per permettere l'ingresso di un g-nodo in una rete è necessario operare una o più migrazioni
di altri g-nodi nella rete ospitante. Questa lista di migrazioni è detta *migration path*.

Esaminiamo il caso banale: sia *𝜑* un g-nodo di livello 1 composto da *𝛽<sub>1</sub>* e *𝛾<sub>0</sub>*.
Sia *𝜀<sub>0</sub>* un singolo nodo che vuole entrare in *𝜑*. Per permettere l'ingresso è necessario
che *𝛽<sub>1</sub>* migri in un diverso g-nodo di livello 1, *𝜓* adiacente a *𝜑*.

L'utente istruisce il sistema *𝜀* di fare ingresso come singolo nodo in *𝜑*, indicando che il cambio
della posizione a livello 0 del suo nuovo indirizzo Netsukuku da *virtuale* a *reale* potrà avvenire solo
al termine della migrazione *m<sub>𝛽</sub>*.  
Poi l'utente istruisce i sistemi della nuova rete sulla presenza di nuovi archi nella rete. Potrebbe essere
anche il solo nodo *𝛽<sub>1</sub>* ad avere un arco verso *𝜀*: infatti esso resta come identità di
connettività in *𝜑*.  
Poi l'utente istruisce il sistema *𝛽* di migrare come singolo nodo in *𝜓*, indicando che il cambio
della posizione a livello 0 del suo nuovo indirizzo Netsukuku da *virtuale* a *reale* potrà avvenire
immediatamente.  
Anche per questa migrazione come per l'ingresso, l'utente istruisce gli altri sistemi della rete
sulla presenza di nuovi archi. In questo caso abbiamo ad esempio i nodi *𝛼*, *𝛾* e *𝜀* che avevano
ciascuno un arco verso *𝛽<sub>1</sub>* e ora hanno un nuovo arco verso *𝛽<sub>2</sub>*.  
Infine l'utente istruisce il sistema *𝜀* che la migrazione *m<sub>𝛽</sub>* è terminata.

### <a name="Migrazione_ingresso_2"></a> Migrazione per ingresso - Caso 2

[Dettagli](DettagliOperazioni11.md)

Esaminiamo un caso più complesso: sia *𝜑* un g-nodo di livello 1. Sia *𝜆* un nuovo sistema che ha un solo
arco fisico che lo collega con un nodo che è dentro *𝜑*. Il sistema *𝜆* vuole entrare nella rete come
gateway per una sottorete autonoma, riservando un g-nodo di livello 1.  
Assumiamo che *𝜑* appartiene al g-nodo di livello 2 *𝜓* che è saturo. Assumiamo che *𝜔* sia un g-nodo di
livello 2 non saturo adiacente a *𝜑*. Per permettere l'ingresso di *𝜆* si decide di far migrare *𝜑* da *𝜓*
in *𝜔* per lasciare un posto per *𝜆* dentro *𝜓*.

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

Gli algoritmi di calcolo dei vari tipi di indirizzo IP sono descritti nel documento [IndirizziIP](IndirizziIP.md).

## <a name="Identita"></a> Identità

Ogni identità che vive nel sistema ha un suo indirizzo Netsukuku. Inoltre ha una mappa di percorsi, ognuno
che ha come destinazione (e come passi) un g-nodo *visibile* dal suo indirizzo Netsukuku.

Un sistema ha sempre una identità principale e zero o più identità di connettività.

L'identità principale gestisce il network namespace default. L'identità principale ha un indirizzo
Netsukuku *definitivo* che è *reale*.

L'identità di connettività gestisce un certo network namespace. L'identità di connettività ha un indirizzo
Netsukuku *di connettività* che è *virtuale*.

### <a name="Indirizzi_ip_propri"></a> Assegnazione indirizzi IP

Le identità di connettività non si assegnano mai nessun indirizzo IP nel loro network namespace.

L'identità principale, nel network namespace default, si assegna degli indirizzi IP sulla base del
suo indirizzo Netsukuku.

Sia *n* l'indirizzo Netsukuku dell'identità principale di un sistema. Sia *l* il numero dei livelli della topologia.

*   Per ogni livello *j* da 0 a *l* - 1:
    *   Il sistema si assegna l'indirizzo IP interno al livello *j* di *n*.
*   Il sistema si assegna l'indirizzo IP globale di *n*.
*   Il sistema può (opzionalmente) fare da anonimizzatore. Cioè si aggiunge la regola di SNAT.
*   Il sistema può (opzionalmente) assegnarsi l'indirizzo IP globale anonimizzante di *n*.

### <a name="Assegnazione_rotte"></a> Assegnazione rotte

Ogni identità nel sistema ha un suo indirizzo Netsukuku e gestisce un network namespace. Per ognuna
il programma computa, in base al suo indirizzo Netsukuku, tutti i possibili indirizzi IP di
destinazione, ognuno con suffisso CIDR, secondo l'algoritmo riportato
[qui](DettagliOperazioni2.md#computo_indirizzi_ip_destinazioni).

Per ogni identità nel sistema, per ciascuna nel suo network namespace, il programma assegna le
rotte verso i possibili indirizzi IP di destinazione.

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *n* l'indirizzo Netsukuku del sistema.
*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Per *i* che scende da *l* - 1 a `$subnetlevel`, per *j* da 0 a *gsize(i)* - 1, se *pos_n(i)* ≠ *j*:
    *   Sia *d* il g-nodo di coordinate (*i*, *j*) rispetto a *n*. Indipendentemente dal
        fatto che *d* sia presente o meno nella rete.
    *   Si computa l'indirizzo IP globale di *d* ed anche il suo indirizzo IP anonimizzante.
    *   Il sistema imposta una rotta per i pacchetti IP in *partenza* verso l'indirizzo IP globale di *d*.  
        La tabella usata per i pacchetti IP in *partenza* sarà chiamata `ntk` e sarà presente
        solo nel network namespace default.  
        Nelle tabelle di routing del kernel per ogni indirizzo IP (con suffisso CIDR) si
        può dichiarare che la destinazione è *non raggiungibile* oppure si riporta come informazione
        il gateway da usare per raggiungere la destinazione.  
        Se l'identità è a conoscenza di percorsi per quella destinazione, allora il programma
        imposta nella tabella il primo gateway del miglior percorso (sebbene l'identità sia
        a conoscenza in effetti anche di altre informazioni sul percorso). Altrimenti il programma
        dichiara nella tabella che la destinazione è *non raggiungibile*.
    *   Analogamente il sistema imposta una rotta per i pacchetti IP in *partenza* verso l'indirizzo IP
        anonimizzante di *d*.
    *   In realtà, la tabella `ntk` nel network namespace default gestito dall'identità principale
        del sistema verrà usata anche per i pacchetti IP in *inoltro* da un MAC address che l'identità
        non conosce. Questo va bene, perché la ricezione di tali pacchetti IP da inoltrare si dovrebbe
        poter verificare solo sul network namespace default e solo se il sistema viene usato consapevolmente
        come gateway: ad esempio se questo sistema è un gateway per una sottorete a gestione autonoma, oppure
        se questo sistema viene usato come NAT per una rete privata.
    *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
        *   Il sistema imposta una rotta per i pacchetti IP provenienti da *m* in *inoltro* verso
            l'indirizzo IP globale di *d*.  
            La tabella usata per i pacchetti IP in *inoltro* da *m* sarà chiamata `ntk_from_$m` e sarà
            presente in qualsiasi network namespace.  
            Viene impostata la rotta identificata dal miglior percorso noto per quella
            destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
            La destinazione è *non raggiungibile* per i pacchetti IP
            in *inoltro* provenienti da *m* se non esiste nella rete la destinazione *d*, oppure se
            l'identità non conosce nessun percorso verso *d* che non passi per il
            massimo distinto g-nodo di *m* per *n*.
        *   Analogamente il sistema imposta una rotta in *inoltro* da *m* verso l'indirizzo IP
            anonimizzante di *d*.
    *   Si tenga presente che solo per l'identità principale nel network namespace default
        e solo per la tabella `ntk` va indicato nella rotta l'indirizzo *src* preferito,
        che serve se un processo locale vuole inviare un pacchetto IP. Sia per la rotta che
        punta all'indirizzo IP globale di *d*, sia per la rotta che punta all'indirizzo
        IP anonimizzante, come *src* preferito dovrà essere indicato l'indirizzo IP globale di *n*.
    *   Per *k* che scende da *l* - 1 a *i* + 1:
        *   Si computa l'indirizzo IP interno al livello *k* di *d*.
        *   Il sistema imposta una rotta per i pacchetti IP in *partenza* verso l'indirizzo IP di *d*
            interno al livello *k*.
        *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
            *   Il sistema imposta una rotta per i pacchetti IP provenienti da *m* in *inoltro* verso
                l'indirizzo IP di *d* interno al livello *k* per i pacchetti IP.
        *   Di nuovo solo per l'identità principale nel network namespace default
            e solo per la tabella `ntk`, per la rotta che punta all'indirizzo IP interno
            al livello *k* di *d* come *src* preferito dovrà essere indicato l'indirizzo
            IP di *n* interno al livello *k*.

## <a name="Indirizzi_del_sistema"></a> Indirizzi IP dell'identità principale del sistema

Come abbiamo visto prima, in un sistema possono esistere diverse identità. Ogni identità detiene un
indirizzo Netsukuku. Ma solo l'identità *principale* del sistema, sulla base del suo indirizzo Netsukuku,
si assegna alcuni indirizzi IPv4 che gli permetteranno di comunicare con il resto della rete.

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa il programma *qspnclient*.

In un sistema Linux il sistema si assegna un indirizzo IP associandolo ad una interfaccia di rete.

Fatta questa premessa, come si comporta il programma?

Il programma **qspnclient** sa quando può cambiare l'indirizzo Netsukuku dell'identità *principale* del
sistema:

*   All'avvio del programma. In questo momento si costituisce la prima identità nel sistema, che è
    in quel momento la principale; ad essa viene associato il primo indirizzo Netsukuku che è completamente *reale*.
*   Quando l'identità principale si duplica. In questo momento la vecchia identità diventa di connettività
    e il posto di identità principale viene preso dalla nuova identità; ad essa viene associato un
    nuovo indirizzo Netsukuku che è completamente *reale*.

In queste occasioni il programma **qspnclient** computa gli indirizzi IP che si deve assegnare il
sistema. Li associa tutti a ognuna delle interfacce di rete reali.

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
Si può vedere un esempio a questo [link](https://capcorne.wordpress.com/2009/03/24/natting-a-network-range-with-netmapiptables).

Fatta questa premessa, come si comporta il programma?

Se il sistema vuole fare da gateway per una sottorete a gestione
autonoma, il programma **qspnclient** deve assicurarsi che ci siano queste regole:

*   Indichiamo con *subnetlevel* il livello del g-nodo rappresentato dalla sottorete autonoma. Quindi abbiamo *subnetlevel* > 0.
*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *n* l'indirizzo Netsukuku dell'identità principale del sistema.
*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Per *i* che sale da *subnetlevel* a *l* - 1:
    *   Se *i* < *l* - 1:
        *   Per i pacchetti IP che passano per questo sistema e sono destinati ad
            un indirizzo IP di tipo interno di livello *i* + 1 che identifica un nodo
            interno alla mia sottorete autonoma (di livello *subnetlevel*) e che quindi
            necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
            dell'IP di destinazione affinché risulti nel range degli indirizzi IP di tipo
            interno di livello *subnetlevel*.
        *   Per i pacchetti IP che passano per questo sistema, che hanno per IP di mittente
            un indirizzo IP di tipo interno di livello *subnetlevel* (che cioè
            provengono dall'interno della sottorete autonoma) e che sono destinati ad
            un indirizzo IP di tipo interno di livello *i* + 1, esegui la rimappatura
            dell'IP di mittente affinché risulti nel range degli indirizzi IP
            di tipo interno di livello *i* e identifichi un nodo
            interno alla mia sottorete autonoma.
    *   Altrimenti (cioè per *i* = *l* - 1):
        *   Per i pacchetti IP che passano per questo sistema e sono destinati ad
            un indirizzo IP di tipo globale che identifica un nodo
            interno alla mia sottorete autonoma (di livello *subnetlevel*) e che quindi
            necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
            dell'IP di destinazione affinché risulti nel range degli indirizzi IP
            di tipo interno di livello *subnetlevel*.
        *   Per i pacchetti IP che passano per questo sistema, che hanno per IP di mittente
            un indirizzo IP di tipo interno di livello *subnetlevel* (che cioè
            provengono dall'interno della sottorete autonoma) e che sono destinati ad
            un indirizzo IP di tipo globale, esegui la rimappatura
            dell'IP di mittente affinché risulti nel range degli indirizzi IP
            di tipo globale e identifichi un nodo
            interno alla mia sottorete autonoma.
        *   Se si vuole che ogni sistema nella sottorete autonoma ammetta di essere contattato in forma anonima:
            *   Per i pacchetti IP che passano per questo sistema e sono destinati ad
                un indirizzo IP di tipo anonimizzante che identifica un nodo
                interno alla mia sottorete autonoma (di livello *subnetlevel*) e che quindi
                necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
                dell'IP di destinazione affinché risulti nel range degli indirizzi IP
                di tipo interno di livello *subnetlevel*.  
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

Per assicurare questo, il programma **qspnclient** deve intervenire all'inizio (cioè quando l'identità
principale del sistema assume il suo primo indirizzo Netsukuku) e ogni volta che l'identità principale
del sistema cambia (un'altra identità diventa la principale).

In tutte queste occasioni il programma **qspnclient**, conoscendo l'indirizzo Netsukuku precedente e quello nuovo dell'identità
principale del sistema, produce i comandi `iptables -t nat -A` e `iptables -t nat -D` necessari.

