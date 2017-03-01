# Modulo PeerServices - Dettagli Tecnici

1.  [Requisiti](#Requisiti)
1.  [Deliverables](#Deliverables)
1.  [Instradamento dei messaggi](#Instradamento_dei_messaggi)
    1.  [Implementazione distribuita](#Implementazione_distribuita)
1.  [Gestione degli errori e della non partecipazione](#Gestione_errori)
    1.  [Modifiche agli algoritmi](#Gestione_errori_Modifiche_algoritmi)
1.  [Mappa dei partecipanti ai servizi opzionali](#Servizi_opzionali_Mappa_partecipanti)
    1.  [Descrizione del meccanismo individuato](#Servizi_opzionali_Descrizione_meccanismo)
    1.  [Modifiche agli algoritmi per la divulgazione della non partecipazione](#Servizi_opzionali_Algoritmi_divulgazione_non_partecipazione)
    1.  [Divulgazione della partecipazione](#Servizi_opzionali_Algoritmi_divulgazione_partecipazione)
1.  [Mantenimento di un database distribuito](#Mantenimento_database_distribuito)
    1.  [Repliche](#Mantenimento_database_distribuito_Repliche)
        1.  [Modifiche agli algoritmi](#Mantenimento_database_distribuito_Algoritmi)
    1.  [Ingresso nella rete di un nodo partecipante](#Mantenimento_database_distribuito_Ingresso_nuovo_nodo)
    1.  [Procedimento di recupero di un record](#Mantenimento_database_distribuito_Recupero_record)
    1.  [Requisiti comuni](#Mantenimento_database_distribuito_Requisiti_comuni)
    1.  [Requisiti specifici](#Mantenimento_database_distribuito_Requisiti_specifici)
        1.  [Database con record che hanno un TTL](#Mantenimento_database_distribuito_TTL)
        1.  [Database con un numero esiguo e fisso di chiavi](#Mantenimento_database_distribuito_FixedKeys)
1.  [Algoritmi](#Algoritmi)

## <a name="Requisiti"></a>Requisiti

L'utilizzatore del modulo PeerServices per prima cosa inizializza il modulo richiamando il metodo statico `init` di PeersManager. In tale metodo viene anche passata l'istanza di INtkdTasklet per fornire l'implementazione del sistema di tasklet.

Quando il nodo ha completato la sua fase di bootstrap come descritto nel [modulo QSPN](../ModuloQspn/AnalisiFunzionale.md), il nodo istanzia il suo PeersManager passando al costruttore:

*   La mappa dei percorsi noti (istanza di IPeersMapPaths `map_paths`).
*   Il livello del g-nodo che questo nodo ha costituito (int `level_new_gnode`).
*   La stub factory per le comunicazioni via percorso interno al nodo che ha avviato una richiesta (istanza di IPeersBackStubFactory `back_stub_factory`).
*   La stub factory per le comunicazioni ai vicini (istanza di IPeersNeighborsFactory `neighbors_factory`).

## <a name="Deliverables"></a>Deliverables

Quando viene costruito il PeersManager gli viene passato il livello del g-nodo che il nodo ha formato. Se non si tratta di un nodo che ha costituito una intera rete ma piuttosto di un nodo entrato in un g-nodo *g* esistente, allora il modulo dialoga con un suo vicino appartenente a *g*, per reperire le mappe dei partecipanti ai servizi opzionali. Lo stub per parlare con il vicino è ottenuto con il metodo `i_peers_fellow` di `map_paths`. Il metodo remoto usato è `get_participant_set`.

* * *

Il modulo segnala quando l'operazione di recupero delle mappe dei partecipanti ai servizi opzionali è completata con successo, attraverso il segnale `participant_maps_ready` di PeersManager.

* * *

Il modulo segnala quando l'operazione di recupero delle mappe dei partecipanti ai servizi opzionali fallisce, attraverso il segnale `participant_maps_failure` di PeersManager.

* * *

Il modulo fornisce su richiesta il livello del g-nodo che il nodo ha formato (informazione che gli è stata passata nel costruttore di PeersManager) e lo stato dell'operazione di recupero delle mappe dei partecipanti ai servizi opzionali, con le proprietà `level_new_gnode`, `participant_maps_retrieved`, `participant_maps_failed` di PeersManager.

* * *

Il modulo fornisce la classe base astratta `PeerService`. Se un nodo vuole fornire un servizio, deve implementare una classe derivata da PeerService. Una istanza di tale classe va registrata con il metodo `register` di PeersManager.

La classe che implementa il servizio può fare in modo che il nodo non venga contattato per elaborare richieste. Questo può essere utile (o anche rendersi necessario) per vari motivi a seconda dello specifico servizio. Questo la classe lo può fare ridefinendo il suo metodo `is_ready()` definito dalla classe base.

* * *

Il modulo aiuta le classi che implementano un servizio nel contattare i nodi da eleggere come replica, con i metodi `begin_replica` e `next_replica` di PeersManager.

* * *

Il modulo fornisce la classe base `PeerClient` per implementare uno stub per fare richieste ad un servizio.

Tale classe, usando il metodo `contact_peer` di PeersManager, permette di avviare il calcolo distribuito di H<sub>t</sub>, ricevere le segnalazioni dei nodi nel tragitto e infine consegnare la richiesta e ricevere la risposta.

## <a name="Instradamento_dei_messaggi"></a>Instradamento dei messaggi

Il calcolo della funzione H<sub>t</sub> deve essere realizzato con una implementazione distribuita perché nella topologia gerarchica di Netsukuku la conoscenza di ogni nodo è parziale. Con questa funzione viene individuato e contattato un nodo esistente nella rete e partecipante al servizio a partire da un indirizzo obiettivo (il risultato di h<sub>p</sub>(*k*)) del quale non sappiamo se è detenuto da un nodo né se il nodo che eventualmente lo detiene partecipa al servizio.

Definiamo la funzione H<sub>t</sub>(x̄) = x, dove *x* è l'indirizzo associato ad un nodo esistente (x ∈ dom(𝛼<sub>t</sub>)) che minimizza la distanza x̄ - x, in modo più rigoroso minarg<sub>x∈dom(𝛼t)</sub>dist(x̄,x). La funzione `dist` rappresenta in modo intuitivo la distanza tra due indirizzi, ma è definita in modo che la funzione H<sub>t</sub>  *cerchi* il primo indirizzo valido *incrementando* l'identificativo fino a *gsize-1* per poi ripartire da *0*. Questo comportamento ci ritornerà utile in  seguito. Precisamente la funzione `dist(x̄,x)` si calcola così:

*   x̄ è formato da x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>l-1</sub>.
*   x è formato da x<sub>0</sub>·x<sub>1</sub>·...·x<sub>l-1</sub>.
*   distanza = 0;
*   Per j da l-1 a 0:
    *   se x̄<sub>j</sub> = x<sub>j</sub>:
        *   distanza += 0;
    *   altrimenti se x<sub>j</sub> > x̄<sub>j</sub>:
        *   distanza += x<sub>j</sub> - x̄<sub>j</sub>;
    *   altrimenti:
        *   distanza += x<sub>j</sub> - x̄<sub>j</sub> + gsize\[j];
    *   se j>0:
        *   distanza *= gsize\[j-1];

### <a name="Implementazione_distribuita"></a>Implementazione distribuita

Sia *n* un nodo che vuole inviare un messaggio *m* all'hash-node della chiave *k* per il servizio *p*. Il nodo n ha indirizzo n<sub>0</sub>·n<sub>1</sub>·...·n<sub>l-1</sub>. Ha inoltre una conoscenza parziale del dominio di 𝛼<sub>t</sub>, conoscenza che indichiamo con dom<sub>n</sub>(𝛼<sub>t</sub>), che comprende:

*   tutti i nodi appartenenti a n<sub>1</sub>,
*   tutti i g-nodi di livello 1 appartenenti a n<sub>2</sub>,
*   ...
*   tutti i g-nodi di livello l-2 appartenenti a n<sub>l-1</sub>,
*   tutti i g-nodi di livello l-1.

Il nodo *n* usa la funzione *h<sub>p</sub>* definita da *p* per calcolare dalla chiave *k* l'indirizzo *x̄* formato da x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>l-1</sub>. A questo punto dovrebbe calcolare *x*=H<sub>t</sub>(x̄). Procede così:

*   Il nodo n calcola H<sub>t</sub>(x̄) secondo le sue conoscenze, cioè trova un livello *j* e un identificativo *x<sub>j</sub>* tali che:
    *   x<sub>j</sub> ∈ n<sub>j+1</sub>, oppure j = l-1.
    *   Il gnodo x<sub>j</sub> è quello, fra le conoscenze di n, che minimizza la funzione dist(x̄, x<sub>j</sub>).
    *   x<sub>j</sub> ≠ n<sub>j</sub>, oppure lo stesso nodo n è il risultato.
    *   x<sub>j</sub> partecipa al servizio (cioè almeno un nodo all'interno di x<sub>j</sub> partecipa).
*   Le  conoscenze del nodo n riguardo la partecipazione al servizio possono  essere non aggiornate rispetto agli altri nodi, ma a questo come vedremo  in seguito l'algoritmo pone rimedio. Si noti invece che il nodo n sa  con certezza se esso stesso partecipa al servizio, quindi il calcolo di   H<sub>t</sub>(x̄) può dare come risultato lo stesso nodo n solo se questo partecipa.
*   Se il risultato è lo stesso nodo n l'algoritmo termina e si passa subito all'esecuzione del messaggio m.
*   Se j = 0 allora n ha trovato con le sue sole conoscenze x<sub>0</sub> = H<sub>t</sub>(x̄).
*   Se j > 0 allora il nodo n non conosce l'interno del gnodo x<sub>j</sub>.    Ma la sua esistenza e partecipazione implica che nella rete esistono uno o più nodi al  suo   interno partecipanti e tra questi senz'altro quello che ha l'indirizzo x che    minimizza la funzione dist(x̄,x).
*   Il nodo n e il nodo x (il destinatario del messaggio m) hanno in comune il g-nodo n<sub>j+1</sub>.  Tutto il percorso che il messaggio deve fare è all'interno di questo  g-nodo; quindi ogni singolo nodo intermedio che riceve il messaggio non  necessita, per inoltrarlo, di identificativi a livelli maggiori di j.
*   Il nodo n prepara un messaggio *m’* da inoltrare al gnodo x<sub>j</sub>. Questo messaggio contiene:
    *   `n`: la tupla n<sub>0</sub>·n<sub>1</sub>·...·n<sub>j</sub>. **Nota**: Il  fatto che la tupla contiene solo gli identificativi inferiori al g-nodo  j+1 fa in modo che questo instradamento funzioni anche all'interno di un g-nodo G mentre G sta gradualmente migrando.
    *   `x̄`: la tupla x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>j-1</sub>.
    *   `lvl, pos`: le coordinate del g-nodo che miriamo a raggiungere, cioè j e  x<sub>j</sub>.
    *   `p_id`: l'identificativo del servizio p.
    *   `msg_id`: un identificativo generato a caso per questo messaggio.
*   Il nodo *n* invia il messaggio m’ al suo miglior gateway verso il gnodo x<sub>j</sub>.  Questo invio viene fatto con protocollo reliable (TCP) senza ricevere  una risposta e senza attendere la sua processazione: d'ora in poi intenderemo tutto questo quando diremo che un nodo instrada un messaggio ad un suo gateway. Se la comunicazione  con il gateway fallisce, *n* cerca un diverso gateway con percorsi verso x<sub>j</sub> e riprova; se giunge a non avere più rotte verso x<sub>j</sub> attende qualche istante che si aggiornino le sue conoscenze e riparte dal calcolo di H<sub>t</sub>(x̄).
*   Il  nodo n  tiene a mente per un certo periodo l'id del messaggio in attesa  di una  risposta. Se passa un certo tempo (basato in qualche modo sulla  dimensione stimata del g-nodo n<sub>j+1</sub>) il nodo n  considera fallito questo tentativo. Più sotto viene descritto come proseguirà n in questo caso.

Il messaggio m’ viene così inoltrato:

*   Il nodo *v* riceve un messaggio m’.
*   Il  nodo v confronta il proprio indirizzo con le coordinate presenti in m’.  Se `v.pos[m’.lvl] = m’.pos` allora il messaggio è giunto al g-nodo che  mirava a raggiungere (vedi sotto il proseguimento dell'algoritmo).
*   Altrimenti  il nodo *v* instrada m’ al suo miglior gateway verso il g-nodo (m’.lvl,  m’.pos). Nella scelta del gateway  si esclude sempre il nodo da cui il messaggio è arrivato. Se la  comunicazione con il gateway fallisce, *v* cerca un diverso gateway (sempre  escluso il nodo di provenienza) con percorsi verso il g-nodo (m’.lvl,  m’.pos) e riprova; se giunge a non avere più rotte verso quel g-nodo  rinuncia.

Il messaggio m’ raggiunge un nodo dentro il gnodo che mirava a raggiungere.

*   Se  m’.lvl = 0 allora il messaggio è giunto alla destinazione finale (vedi  sotto il proseguimento dell'algoritmo). Altrimenti si prosegue.
*   Il nodo v calcola H<sub>t</sub>(m’.x̄) secondo le sue conoscenze relative al suo g-nodo di livello m’.lvl, cioè trova un livello *k* e un identificativo *x<sub>k</sub>*. Sicuramente k﹤m’.lvl.
*   Se il nodo v trova che esso stesso è H<sub>t</sub>(m’.x̄)  allora il messaggio è giunto alla destinazione finale (vedi sotto il  proseguimento dell'algoritmo). Altrimenti si prosegue.
*   Il nodo v duplica il messaggio m’ in m’’ e poi modifica i seguenti membri del messaggio m’’:
    *   lvl diventa k.
    *   pos diventa *x<sub>k</sub>*.
    *   x̄ diventa x̄<sub>0</sub>·x̄<sub>1</sub>·...·x̄<sub>k-1</sub>.
*   Il  nodo *v* instrada  m’’ al suo miglior gateway verso il g-nodo (m’’.lvl,  m’’.pos). Nella scelta del gateway  si esclude sempre il nodo da cui il messaggio è arrivato. L'algoritmo  prosegue come detto prima con il prossimo nodo che riceve m’’. Se la  comunicazione con il gateway fallisce, *v* cerca un diverso gateway   (sempre escluso il nodo di provenienza) con percorsi verso  il g-nodo  (m’’.lvl, m’’.pos) e riprova; se giunge a non avere più rotte verso quel  g-nodo attende qualche istante che si aggiornino le sue conoscenze e  riparte dal calcolo di H<sub>t</sub>(m’.x̄).

Il messaggio m’ raggiunge la destinazione finale:

*   Il  nodo v prepara uno stub TCP per connettersi al nodo originante tramite  percorso interno attraverso la tupla `m’.n`.
*   Una volta realizzata la connessione TCP tra n e v il dialogo consiste in:
    *   v comunica a n `m’.msg_id`;
    *   Se n aveva rinunciato ad attendere la risposta a questo messaggio lo comunica a v e qui si interrompe.
    *   Altrimenti n comunica a v il messaggio m.
    *   v elabora il messaggio e risponde a n. Poi la comunicazione si chiude.
*   Il nodo n completa il metodo p2p che aveva generato il messaggio, eventualmente ritornando un risultato.

## <a name="Gestione_errori"></a>Gestione degli errori e della non partecipazione

Siano *n* ed *m* due nodi che conoscono una certa chiave *k* per un servizio *p*. Entrambi sono in grado di calcolare *x* = H<sub>t</sub>(h<sub>p</sub>(k)) e di contattare x. Sia *jn* il livello del g-nodo comune a n ed x: cioè x ∈ n<sub>jn</sub>, x ∉ n<sub>jn-1</sub>. Sia *jm* il livello del g-nodo comune a m ed x: cioè x ∈ m<sub>jm</sub>, x ∉ m<sub>jm-1</sub>. Sia *j* il minore tra jn e jm. Ne consegue che x ∉ n<sub>j-1</sub>, x ∉ m<sub>j-1.</sub> Cioè l'interno del g-nodo x<sub>j-1</sub>  di livello j-1 a cui appartiene il nodo x è sconosciuto per n e per m.  Supponiamo ora che per qualche motivo i messaggi instradati dal modulo PeerServices si perdano all'interno del g-nodo x<sub>𝜀</sub> (con 𝜀 piccolo a piacere), 𝜀﹤j-1. Oppure se il servizio p è  opzionale può verificarsi che nessun nodo ora partecipa all'interno di x<sub>𝜀</sub> ma che questa informazione non si era ancora divulgata al suo esterno.

Pur  essendo questa anomalia circoscritta ad un g-nodo piccolo a piacere,  questo impedirebbe ai nodi n ed m di scrivere e leggere dati con chiave k  nel servizio p.

Dopo  che n vede fallire il suo tentativo di contattare x per salvare un  record con chiave k, n deve cercare di isolare il g-nodo malfunzionante.  Se lo facesse basandosi solo sulle sue conoscenze potrebbe solo  calcolare *x’* = *H<sub>t</sub>(h<sub>p</sub>(k), exclude_list=[x<sub>jn-1</sub>]  )*. Indichiamo con questa dicitura che *x<sub>jn-1</sub>* viene considerato non valido come risultato. Ora il nodo m cerca di contattare x per leggere il record con chiave  k, vede che il suo tentativo di contattare x fallisce e quindi prova ad  isolare il g-nodo malfunzionante. Basandosi solo sulle sue conoscenze  calcolerebbe  *x’’* = *Ht(hp(k), exclude_list=[x<sub>jm-1</sub>])*. La conseguenza è che se jm ≠ jn allora n ed m contattano nodi diversi.

Questo significa che un meccanismo robusto deve prevedere che il nodo n che cerca di contattare il nodo x ∈ n<sub>jn</sub> e non vi riesce deve rilevare tutti gli identificativi del g-nodo x<sub>𝜀</sub> (dal livello 𝜀 al livello jn-1) in cui il messaggio si è perso.

### <a name="Gestione_errori_Modifiche_algoritmi"></a>Modifiche agli algoritmi

Modifichiamo  l'algoritmo di instradamento considerando che si deve comunicare al  nodo originante del messaggio m’ quali g-nodi sono stati raggiunti  correttamente e quali invece sono stati identificati come non affidabili o non usabili nella presente operazione.

Modifichiamo inoltre l'algoritmo di calcolo distribuito di H<sub>t</sub>  considerando che deve essere possibile escludere un set di determinati  g-nodi.

Gli elementi di questo set sono g-nodi che appartengono ad un unico g-nodo *g* di livello *w*, ma ognuno di questi g-nodi *h* può avere un diverso livello *𝜀*, con  *𝜀* < *w*. Menzioniamo il g-nodo *g* di livello *w* perché in alcuni casi il nodo che inizia la ricerca di un hash-node ha interesse a circoscrivere<sup>1</sup> la sua ricerca all'interno di un suo g-nodo di livello *w*. Lo fa passando alla funzione H<sub>t</sub> una tupla che ha un numero di *w* elementi. Come caso particolare possiamo avere *w* = *l*, cioè l'intera rete è il g-nodo contenente i singoli g-nodi di questo set.

Ogni elemento del suddetto set ha come dati il livello del g-nodo *g* contenitore (*w*) e inoltre la tupla di posizioni del g-nodo *h*, da *𝜀* a *w* - 1. Chiamiamo queste tuple *tuple globali nel g-nodo di ricerca*.

Rendiamo cioè possibile l'esclusione di un g-nodo, di qualsiasi livello, anche se non visibile nella mappa del nodo richiedente, nel mezzo dell'esecuzione del calcolo distribuito di H<sub>t</sub>.

Dal momento che rendiamo possibile l'esclusione *in itinere* anche di un singolo nodo, possiamo anche dare al nodo che riceve il messaggio (quindi dopo che lo ha ricevuto e letto) la possibilità di rifiutarsi di elaborarlo e far proseguire da qui il calcolo distribuito di H<sub>t</sub>. Questo ci consentirà di gestire i casi in cui un nodo voglia demandare al successivo per "mancanza di memoria", come descritto nell'analisi funzionale.

Inoltre introduciamo la possibilità da parte del nodo che riceve il messaggio di rifiutarsi di elaborarlo e dare istruzione al chiamante di riavviare da capo il calcolo distribuito di H<sub>t</sub>. Questo ci servirà nelle situazioni in cui, come vedremo in seguito, un servente sia costretto a fare lunghe operazioni di recupero dati e nel contempo si voglia garantire la coerenza degli stessi in un database distribuito.

Iniziamo.

Il nodo *n* vuole contattare l'hash-node per la chiave *k* per fare una richiesta *r* al servizio *p*. Il nodo deve avere una istanza della classe PeerClient del servizio p. Questa conosce il `p_id`, come calcolare *x̄* = *h<sub>p</sub>(k)*, il tempo massimo di attesa dell'esecuzione `timeout_exec`, come produrre la richiesta *r* di tipo IPeersRequest e come interpretare la risposta di tipo IPeersResponse. Sempre l'istanza della classe PeerClient, avvia il seguente algoritmo, che in seguito chiamiamo `contact_peer`, che ha come argomenti (`p_id, x̄, r, timeout_exec, exclude_myself`) e restituisce come risultato una istanza di IPeersResponse.

*   Metti `refuse_messages` = "".
*   Prepara una lista vuota `exclude_gnode_list` di istanze HCoord. Qui finiranno i g-nodi visibili nella mappa di n che andranno esclusi dal calcolo di H<sub>t</sub>.
*   Se il servizio `p_id` è opzionale:
    *   Mette in `exclude_gnode_list` tutti i g-nodi che stando alle sue conoscenze attuali non partecipano a `p_id`.
*   Se `exclude_myself`, cioè n non intende partecipare (potrebbe volersi escludere anche se p non è opzionale):
    *   Mette se stesso (0, pos\[0]) in `exclude_gnode_list`.
*   Prepara una lista vuota `exclude_tuple_list`  di tuple globali nel g-nodo di ricerca. Qui finiranno i g-nodi non visibili nella mappa di n  che andranno esclusi in eventuali successivi tentativi di raggiungere  l'hash-node.
*   Questa lista è implementata in modo tale che quando vi si inserisce una tupla che rappresenta un g-nodo *g* vengono anche rimosse le tuple che rappresentano un g-nodo *g*’ ∈ *g*.
*   Fino a che non completa (ciclo 1):
    *   Calcola x = H<sub>t</sub>(`x̄, exclude_list=exclude_gnode_list`). Trova x<sub>j</sub>.
    *   In alternativa il calcolo di H<sub>t</sub> può restituire lo stesso nodo n.
        *   In questo caso come detto prima l'algoritmo prova ad eseguire il metodo, ma è prevista la possibilità che il modulo del servizio specifico, analizzando la richiesta, si rifiuti di elaborarla (PeersRefuseExecutionError). Oppure che dia istruzione di riavviare da capo il calcolo di H<sub>t</sub> (PeersRedoFromStartError).
        *   Se si riceve l'eccezione PeersRedoFromStartError:
            *   Rilancia dall'inizio l'algoritmo `contact_peer`.
            *   L'algoritmo termina.
        *   Se si riceve l'eccezione PeersRefuseExecutionError *e*:
            *   Aggiungi a `refuse_messages` il messaggio `e.message`.
            *   Mette se stesso (0, pos\[0]) in `exclude_gnode_list`.
            *   Continua con la prossima iterazione del ciclo 1.
        *   # L'elaborazione è accettata:
        *   Il risultato del metodo viene restituito
        *   L'algoritmo termina.
    *   In alternativa il calcolo di H<sub>t</sub> può restituire l'eccezione "nessun nodo". In questo caso:
        *   Se `refuse_messages` ≠ "":
            *   Viene rilanciata una eccezione PeersDatabaseError.
        *   Altrimenti:
            *   Viene rilanciata una eccezione PeersNoParticipantsInNetworkError.
    *   Prepara il messaggio m’ (generando qui il suo `msg_id`) da instradare verso x<sub>j</sub> come avevamo visto prima. Inoltre nel messaggio m’ aggiunge la lista `m’.exclude_tuple_list`  di tuple interne ad un g-nodo. Il livello del g-nodo per tutte le tuple  in questa lista è j. Infatti in questa lista il nodo n mette tutte le  tuple che trova nella sua lista `exclude_tuple_list` che sono riferite a  g-nodi interni a x<sub>j</sub>.
    *   Come visto in precedenza instrada al miglior gateway verso x<sub>j</sub>  il messaggio m’. Come visto in precedenza questo include il fatto di  rimuovere i gateway che eventualmente falliscono fino alla possibilità  di ripartire dal calcolo di H<sub>t</sub>.
    *   Come visto in precedenza si mette in attesa di risposte per un tempo basato sulla dimensione stimata del g-nodo n<sub>j+1</sub> (ciclo 2). Si possono verificare questi casi:
        1.  Riceve un messaggio che gli segnala come prossima destinazione di m’ il g-nodo x<sub>k</sub> con k < j.
            *   Tiene a mente quale sia la destinazione con il valore di k più piccolo ricevuta (all'inizio era j).
            *   Continua ad aspettare, sempre per un tempo basato sulla dimensione stimata del g-nodo n<sub>j+1</sub>.
        1.  Riceve un messaggio che gli segnala che il g-nodo  x<sub>k</sub>, con k ≤ j, non partecipa al servizio.
            *   Se x<sub>k</sub> è visibile nella mappa di n, cioè se k = j:
                *   Se il servizio p è opzionale:
                    *   Aggiorna nelle sue conoscenze che quel g-nodo non partecipa a p.
                *   Inserisce il g-nodo in `exclude_gnode_list` come istanza di HCoord.
            *   Inserisce il g-nodo in `exclude_tuple_list` come tupla globale nel g-nodo di ricerca.
            *   Non aspetta altri messaggi, esce dal ciclo 2.
        1.  Riceve un messaggio che gli segnala che il g-nodo  x<sub>k</sub>, con k ≤ j, non sa, viste le esclusioni, a chi inoltrare.
            *   Se  x<sub>k</sub> è visibile nella mappa di n, cioè se k = j:
                *   Inserisce il g-nodo in `exclude_gnode_list` come istanza di HCoord.
            *   Inserisce il g-nodo in `exclude_tuple_list` come tupla globale nel g-nodo di ricerca.
            *   Non aspetta altri messaggi, esce dal ciclo 2.
        1.  Viene contattato dal nodo destinatario  x<sub>0</sub>, che gli chiede il messaggio.
            *   Invia il messaggio.
            *   Considera 0 come il valore di k più piccolo ricevuto.
            *   Aspetta altri messaggi, ora per il tempo `timeout_exec`. Accetta da adesso solo il messaggio di risposta, gli altri li ignora e torna ad attendere.
        1.  Viene contattato da x<sub>0</sub>, che gli invia la risposta.
            *   Accetta questo messaggio solo se aveva comunicato la richiesta a x<sub>0</sub> altrimenti lo ignora e torna ad attendere.
            *   Riceve la risposta.
            *   Non aspetta altri messaggi, esce dal ciclo 2 e poi uscirà dal ciclo 1.
        1.  Viene contattato da x<sub>0</sub>, che rifiuta di elaborare la richiesta.
            *   Accetta questo messaggio solo se aveva comunicato la richiesta a x<sub>0</sub> altrimenti lo ignora e torna ad attendere.
            *   Se  x<sub>0</sub> è visibile nella mappa di n, cioè se 0 = j:
                *   Inserisce il g-nodo in `exclude_gnode_list` come istanza di HCoord.
            *   Inserisce il g-nodo in `exclude_tuple_list` come tupla globale nel g-nodo di ricerca.
            *   Aggiungi a `refuse_messages` il messaggio di questo rifiuto.
            *   Non aspetta altri messaggi, esce dal ciclo 2.
        1.  Viene contattato da x<sub>0</sub>, che lo istruisce di ricominciare da capo il calcolo distribuito di H<sub>t</sub>.
            *   Accetta questo messaggio solo se aveva comunicato la richiesta a x<sub>0</sub> altrimenti lo ignora e torna ad attendere.
            *   Rilancia dall'inizio l'algoritmo `contact_peer`.
        1.  Il tempo scade.
            *   Considera il più piccolo k ricevuto.
            *   Se  x<sub>k</sub> è visibile nella mappa di n, cioè se k = j:
                *   Inserisce il g-nodo in `exclude_gnode_list` come istanza di HCoord.
            *   Inserisce il g-nodo in `exclude_tuple_list` come tupla globale nel g-nodo di ricerca.
            *   Non aspetta altri messaggi, esce dal ciclo 2.
    *   Se ha ricevuto la risposta esce dal ciclo 1.
*   Restituisce la risposta al chiamante.

Durante l'istradamento del messaggio m’ il nodo *v* riceve il messaggio.

*   Il nodo v confronta il proprio indirizzo con le coordinate presenti in m’.
*   Se `v.pos[m’.lvl]` ≠ m’.pos:
    *   v  instrada m’ verso (m’.lvl, m’.pos) e questo avviene come detto in  precedenza, con la possibilità di escludere i gateway che falliscono.
*   Altrimenti:
    *   Recupera il servizio p da `m’.p_id`.
    *   Se il servizio p è opzionale e stando alle sue conoscenze il suo g-nodo di livello m’.lvl non partecipa a p:
        *   Il  nodo v si connette via TCP ad  n attraverso la tupla m’.n e gli  comunica (senza necessitare alcuna risposta) che il suo g-nodo di  livello m’.lvl non partecipa a p. Indica in questo messaggio  l'identificativo `m’.msg_id` e la tupla che identifica all'interno di n<sub>j+1</sub> il g-nodo di v, cioè la tupla da m’.lvl a j dove *j* è il livello del g-nodo primo obiettivo. j+1 = `m’.n.tuple.size`.
    *   Altrimenti:
        *   Prepara una lista vuota `exclude_gnode_list` di istanze HCoord.
        *   Se il servizio p è opzionale:
            *   Mette  in `exclude_gnode_list` tutti i g-nodi di livello inferiore a m’.lvl che  stando alle sue conoscenze attuali non partecipano a p.
        *   Se n non intende partecipare (potrebbe volersi escludere anche se p non è opzionale):
            *   Mette se stesso (0, pos\[0]) in `exclude_gnode_list`.
        *   Esamina la lista `m’.exclude_tuple_list` di tuple interne. Se vi sono delle tuple che identificano dei g-nodi visibili nella mappa di v li inserisce in `exclude_gnode_list` come istanza di HCoord.
        *   Calcola x = H<sub>t</sub>(`m’.x̄,  exclude_list=exclude_gnode_list`); come conseguenza del fatto che la  tupla m’.x̄ ha solo m’.lvl elementi la ricerca è ristretta al g-nodo di  livello m’.lvl. Il risultato di questo calcolo può essere di 3 tipi:
            1.  Trova x<sub>k</sub> con k < j.
                *   Il nodo v duplica il messaggio m’ in m’’, come descritto in precedenza. Inoltre nel messaggio m’’ aggiunge la lista `m’’.exclude_tuple_list`  di tuple interne ad un g-nodo. Il livello del g-nodo per tutte le tuple  in questa lista è k. Infatti in questa lista il nodo v mette tutte le  tuple che trova nella lista `m’.exclude_tuple_list` che sono riferite a  g-nodi interni a x<sub>k</sub>.
                *   Il nodo v instrada m’’ verso il nuovo obiettivo come detto in precedenza.
                *   Inoltre il nodo v si connette via TCP ad  n attraverso la tupla m’.n e gli comunica (senza necessitare alcuna risposta) che x<sub>k</sub>  è il nuovo obiettivo del messaggio. Indica in questo messaggio  l'identificativo `m’.msg_id` e la tupla che identifica all'interno di n<sub>j+1</sub> il g-nodo di x<sub>k</sub>, cioè la tupla da k a j dove *j* è il livello del g-nodo primo obiettivo.
            1.  Trova lo stesso nodo v.
                *   Il nodo v si connette via TCP ad  n attraverso la tupla m’.n e questi gli passa la richiesta, come descritto prima. Il nodo v, però, ora ha la possibilità di elaborarla **oppure** di rifiutare l'elaborazione per mancanza di memoria o perché non esaustivo **oppure** di istruire il client di riavviare il calcolo distribuito di H<sub>t</sub>. Il nodo v comunica a n il risultato o il rifiuto.
            1.  Restituisce l'eccezione nessun nodo.
                *   Il  nodo v si connette via TCP ad  n attraverso la tupla m’.n e gli  comunica (senza necessitare alcuna risposta) che il suo g-nodo di  livello m’.lvl non ha trovato una destinazione a motivo delle esclusioni  imposte. Indica in questo messaggio l'identificativo `m’.msg_id` e la  tupla che identifica all'interno di n<sub>j+1</sub> il g-nodo di v cioè la tupla da m’.lvl a j dove *j* è il livello del g-nodo primo obiettivo.

* * *

Note:

**Nota 1**. Quando la tupla passata alla funzione H<sub>t</sub> ha un numero di *w* elementi la ricerca è circoscritta al g-nodo di livello *w*. Soddisfare questo requisito è banale: basta escludere dalla ricerca iniziale fatta dal nodo *n* tutti i g-nodi esterni al g-nodo n<sub>w</sub>.

## <a name="Servizi_opzionali_Mappa_partecipanti"></a>Mappa dei partecipanti ai servizi opzionali

Di default un nodo non partecipa ad un servizio  opzionale. Si ricordi che un nodo non ha bisogno di partecipare ad un  servizio opzionale per poter chiedere i servizi, ma solo se vuole  fornirli.

Quando un nodo *v*  vuole entrare in un servizio opzionale come partecipante deve  comunicarlo ai suoi vicini perché l'informazione si propaghi per tutta  la rete.

Quando  v volesse uscire da un servizio non è necessario che lo comunichi  subito. Grazie al meccanismo di fault tolerance introdotto dal modulo PeerServices, nel momento in cui una richiesta arrivi da un nodo *n* al nodo v, questo comunicherà a n che non partecipa al servizio.

Come  regola generale, si consideri che divulgare l'informazione che un nodo o  un g-nodo partecipano al servizio non è mai dannoso. Supponiamo che si  sparge la voce che il g-nodo *u<sub>j</sub>* partecipa al servizio *p*. Ad un certo momento il nodo n, n ∉ u<sub>j</sub>, vuole salvare un record in p con chiave *k* e il messaggio viene inoltrato fino a u<sub>j</sub> che risulta il più vicino partecipante a *h<sub>p</sub>(k)*.

Sia *w* il border nodo di u<sub>j</sub> che riveve il messaggio, w ∈ u<sub>j</sub>. Supponiamo che w ritiene che u<sub>j</sub> partecipa al servizio in quanto ritiene che u<sub>k</sub> (con j > k ≥ 0) che è nella sua mappa (vale a dire w ∈ u<sub>k+1</sub>, w ∉ u<sub>k</sub>) partecipa. Allora w inoltra il messaggio verso u<sub>k</sub>.

Sia *w̄* il border nodo di u<sub>k</sub> che riceve il messaggio. Supponiamo che w̄ sa che u<sub>k</sub>  non partecipa al servizio. Allora  w̄ contatta il nodo n e gli dice,  con la tupla dal livello k fino al livello comune con n, che u<sub>k</sub>  non partecipa al servizio. A questo punto n può memorizzare questa  conoscenza. Inoltre n ritenterà indicando nel messaggio che u<sub>k</sub> va escluso in quanto non partecipante. Quando il messaggio giunge a w, il border nodo di u<sub>j</sub>, questo scopre che u<sub>k</sub> non partecipa al servizio e da questo conclude che nemmeno u<sub>j</sub>, in quanto l'unico che pensava partecipante era u<sub>k</sub>. Allora w contatta il nodo n e gli dice, con la tupla dal livello j fino al livello comune con n, che u<sub>j</sub>  non partecipa al servizio. A questo punto n può memorizzare questa  conoscenza. Infine n ritenterà con le sue nuove conoscenze, quindi  contatterà un nodo veramente partecipante e memorizzerà in esso il  record.

Supponiamo  ora che un nodo x vuole leggere il record con chiave k. Analogamente a  quanto visto sopra, sebbene x possa inizialmente ritenere erroneamente  che un g-nodo (ad esempio u<sub>j</sub>) partecipi al  servizio, x cercherà di contattarlo e alla fine avrà aggiornate le sue  conoscenze e contatterà il vero partecipante e leggerà il record.  Quindi, nessun danno deriva dal supporre che un g-nodo partecipi.

Diversamente,  supporre che un g-nodo non partecipa, mentre questo partecipa e qualcun  altro lo sa, produce malfunzionamenti nel servizio. Sia u<sub>j</sub>  partecipante al servizio p. Sia n un nodo che lo sa, mentre x lo  ritiene non partecipante. Supponiamo che l'indirizzo più vicino a *h<sub>p</sub>(k)* sia proprio u<sub>j</sub>. Allora se n vuole salvare un record con chiave k in p lo salverà in u<sub>j</sub>, ma se x lo vorrà leggere non lo cercherà in u<sub>j</sub>, bensì altrove.

Il  meccanismo più robusto potrebbe essere quindi quello di considerare  sempre ogni nodo o g-nodo esistente nella rete come partecipante a tutti  i servizi, fino a prova contraria data nel momento in cui si fa una  richiesta. Anche in seguito, a richiesta soddisfatta, si dovrebbe subito  supporre che i g-nodi esclusi potrebbero essere nuovamente entrati nel  servizio.

Questo  meccanismo sarebbe robusto ma introdurrebbe pesantezza e notevoli  ritardi soprattutto in servizi in cui partecipano pochissimi nodi.

### <a name="Servizi_opzionali_Descrizione_meccanismo"></a>Descrizione del meccanismo individuato

Sia *n* un nodo appena entrato in un g-nodo *g* di livello *l*. Esso chiede ad un vicino che è anche membro del g-nodo *g* le mappe dei partecipanti ai servizi opzionali dal livello *l* in su, con il metodo remoto `get_participant_set`.

Il nodo n considera un generico g-nodo *g*, in assenza di comunicazioni a riguardo di g, non partecipante  ad un servizio opzionale. Può venire a conoscenza della partecipazione  solo quando riceve un messaggio del flood che viene illustrato sotto. A  quel punto lo considera partecipante.

Per scoprire che un g-nodo è non partecipante ci saranno due possibilità.

Sia *n* un nodo che avvia per sé una richiesta *r* al servizio *p*. Il nodo *n* ritiene che *H<sub>t</sub>(h<sub>p</sub>(k))* sia il g-nodo *g* visibile nella sua mappa quindi invia il messaggio m’ verso *g*. Dopo qualche istante riceve la segnalazione che *g* non partecipa al servizio, quindi ora considera *g* non partecipante. Questa era la prima possibilità.

Supponiamo invece che un tentativo da parte di *n* di contattare un hash-node fallisce perché viene segnalato che un certo g-nodo *h* ritenuto partecipante in realtà non partecipa. Può essere *h* visibile nella mappa di *n* oppure non visibile. Comunque il nodo *n* aggiunge al messaggio m’ per i prossimi tentativi una informazione che consente di identificare il g-nodo *h* all'interno della rete (oppure all'interno del g-nodo nel quale la ricerca dell'hash-node era eventualmente circoscritta).

Ora il nodo *n* invia il messaggio m’ per un nuovo tentativo di raggiungere l'hash-node. Sia *v* un generico nodo che instrada questo messaggio. Il nodo *v* esaminando il dato di cui sopra nel messaggio, individua il g-nodo *h* che potrebbe essere visibile nella sua mappa. Supponiamo che tale g-nodo era considerato da *v* partecipante al servizio *p*. Ora *v* ha una indicazione diversa. Allora per averne conferma avvierà da lì a breve una finta richiesta verso un nodo a caso in *h*. Se riceve la segnalazione che *h* non partecipa da ora lo considera non partecipante. Questa era la seconda possibilità.

### <a name="Servizi_opzionali_Algoritmi_divulgazione_non_partecipazione"></a>Modifiche agli algoritmi per la divulgazione della non partecipazione

Sia *n* un nodo che tenta di inviare un messaggio ad un certo hash-node in un servizio opzionale *p*. Supponiamo che n riceve l'informazione che un certo g-nodo *g*  non partecipa al servizio. Può essere g visibile nella mappa di n  oppure non visibile. In ogni caso il nodo n sta per inviare un nuovo  messaggio alla ricerca del suo hash-node. Ne approfittiamo per  aggiungere al messaggio l'informazione sulla non partecipazione di g,  che può essere di interesse per i nodi che instradano il messaggio.

Il nodo n riceve questa informazione come tupla interna ad un g-nodo di livello *j*, con *j*﹤*w*. Salva questa tupla in una lista `non_participant_tuple_list`  di tuple globali nel g-nodo di ricerca di livello *w*. Come per la lista `exclude_tuple_list`, anche questa è  implementata in modo tale che quando vi si inserisce una tupla che  rappresenta un g-nodo *g* vengono anche rimosse le tuple che rappresentano un g-nodo *g*’ ∈ *g*. Anche questa lista inoltre viene istanziata all'inizio del tentativo di instradare un messaggio e rimane in vita solo fino a tentativo esaurito.

Supponiamo ora che il nodo n avvii un messaggio verso il g-nodo x<sub>j</sub> all'interno di n<sub>j+1</sub>. Allora aggiunge a m’ la lista `m’.non_participant_tuple_list` di tuple globali mettendovi quelle tuple *t* ∈ `non_participant_tuple_list` che rappresentano un g-nodo *g* tale che sia visibile in alcuni nodi dentro n<sub>j+1</sub>.

Per i g-nodi g con livello l ≥ j questi devono avere il loro g-nodo superiore, al livello l+1, in comune con n. Per i g-nodi g con livello l﹤j questi devono avere il loro g-nodo di livello j+1 in comune con n.

Quando un nodo *v* riceve m’:

*   Sia *p* il servizio individuato da `m’.p_id`.
*   Se v deve solo inoltrare m’:
    *   Inoltra m’ come in precedenza.
*   Altrimenti, cioè deve duplicare m’ in m’’ e inviarlo verso x<sub>k</sub> con k﹤j:
    *   Copia in `m’’.non_participant_tuple_list` solo quelle tuple *t* ∈ `m’.non_participant_tuple_list` che rappresentano un g-nodo *g* tale che sia visibile in alcuni nodi dentro v<sub>k+1</sub>. Cioè ... vedi sopra.
    *   Invia m’’ come in precedenza.
*   Se p è opzionale:
    *   Per ogni tupla *t* in `m’.non_participant_tuple_list`:
        *   Sia *g* il g-nodo individuato da t.
        *   Se g è visibile nella mappa di v:
            *   Se secondo le conoscenze di v, g partecipa al servizio p:
                *   In una tasklet:
                    *   v  avvia una finta richiesta verso un nodo a caso in g. Se riceve la  segnalazione che g non partecipa da ora lo considera non partecipante.

### <a name="Servizi_opzionali_Algoritmi_divulgazione_partecipazione"></a>Divulgazione della partecipazione

Segue  un tentativo di definire un algoritmo distribuito che possa fornire una  notifica di partecipazione a tutta la rete con un buon livello di  affidabilità, che sia leggero in termini di traffico.

Sia *n* un nodo che avvia la sua partecipazione ad un servizio *p*.

*   Per 5 volte:
    *   n segnala ai suoi vicini la sua tupla n<sub>0</sub>·n<sub>1</sub>·...·n<sub>l-1</sub> (con *l* = numero di livelli) e il servizio a cui partecipa.
    *   n aspetta 300 secondi.
*   Per sempre:
    *   n segnala ai suoi vicini la sua tupla n<sub>0</sub>·n<sub>1</sub>·...·n<sub>l-1</sub> (con *l* = numero di livelli) e il servizio a cui partecipa.
    *   n aspetta 1 giorno + random(1..24*60*60) secondi.

Sia *v* un nodo che riceve un messaggio di partecipazione ad un servizio *p*.

*   v riceve il messaggio con tupla n<sub>j</sub>·n<sub>j+1</sub>·...·n<sub>l-1</sub> (con *j* ≥ 0)
*   Se v ∈ n<sub>j</sub>:
    *   v ignora il messaggio.
*   Altrimenti:
    *   Sia *k* (k ≥ j) il livello tale che v ∈ n<sub>k+1</sub>, v ∉ n<sub>k</sub>.
    *   v considera ora la tupla n<sub>k</sub>, cioè n<sub>k</sub>·n<sub>k+1</sub>·...·n<sub>l-1</sub>.
    *   Se la tupla n<sub>k</sub> è nella `lista_recenti`:
        *   v ignora il messaggio.
    *   Altrimenti:
        *   v mette la tupla n<sub>k</sub> nella `lista_recenti`.
        *   v segnala ai vicini che n<sub>k</sub> partecipa a p.
        *   v ora sa che  n<sub>k</sub> partecipa a p.
        *   v attende (in una tasklet apposita) 60 secondi.
        *   v rimuove n<sub>k</sub> dalla `lista_recenti`.

## <a name="Mantenimento_database_distribuito"></a>Mantenimento di un database distribuito

Se un obiettivo essenziale di un certo servizio è quello di mantenere un database distribuito "robusto" occorre che tale servizio tenga in considerazione alcuni aspetti che ora esamineremo. Con il termine "robusto" intendiamo un database che:

*   Offra un certo grado di affidabilità quanto alla *persistenza dei dati*.  
    La struttura fondamentale dell'hashtable distribuito è tale che ogni dato viene memorizzato su un determinato nodo. Va considerato che il nodo che memorizza un certo dato può in qualsiasi momento morire o venire scollegato dal resto della rete. Per migliorare l'affidabilità del database si implementa quindi un meccanismo di repliche.  
    Il grado di affidabilità della persistenza non potrà mai essere assoluto. Ad esempio, se tutti i nodi partecipanti al servizio muoiono, i dati memorizzati andranno persi. Inoltre si consideri che, per i limiti di memoria a disposizione dei singoli nodi oltre che per praticità e questioni di performance della rete, nessun dato potrà essere replicato un numero molto elevato di volte. Quindi basta che tutti i nodi che hanno una replica di quel dato muoiano (o siano scollegati dal resto della rete) improvvisamente, che il dato andrà perso.
*   Garantisca la *coerenza dei dati*.  
    Supponiamo che la persistenza dei dati venga adeguatamente garantita. La flessibilità della rete fa si che nuovi partecipanti al servizio possono entrare nella rete in ogni momento. Le operazioni di tali nodi devono essere concertate, di modo che modifiche alla base dati vengano validate e rese visibili a tutti i nodi. Facciamo un esempio:
    *   Siano *a* e *b* due nodi che usano un servizio *p* che offre un database distribuito.
    *   Sia presente in questo database un record che assegna alla chiave *k* il valore *w*.
    *   Il nodo *a* cerca di scrivere su questo database il valore *v* per la chiave *k*.
    *   Il nodo *a* vede accettata la richiesta al tempo *t*.
    *   Se al tempo *t* + 1 il nodo *b* cerca di leggere il valore associato alla chiave *k*, deve ritrovare *v*, non più *w*.

Come vedremo in seguito, il modulo fornisce dei metodi helper (inclusi nel modulo per evitare duplicazione di codice) che potranno essere usati dai vari servizi registrati che si occupano di mantenere un database distribuito.

Gli algoritmi con cui questi metodi affrontano le problematiche che intendono risolvere, dipendono da quali sono le caratteristiche dello specifico servizio. Vedremo in seguito che al momento sono state individuate due classi di servizi: i *database temporali* e i *database a chiavi fisse*.

Il modulo fornisce la classe DatabaseHandler e l'interfaccia IDatabaseDescriptor.

La classe DatabaseHandler è esposta dal modulo, ma il suo contenuto è del tutto oscuro all'esterno del modulo. I suoi membri, accessibili solo dal modulo, permettono di mantenere per uno specifico servizio le strutture dati necessarie al modulo per gestire le operazioni di mantenimento di questi due tipi di database.

L'interfaccia IDatabaseDescriptor espone dei metodi che il modulo utilizza nello svolgere le operazioni di cui stiamo parlando. Questi sono usati in entrambi questi due tipi di database. Poi l'interfaccia viene estesa da due interfacce, la ITemporalDatabaseDescriptor e la IFixedKeysDatabaseDescriptor, che espongono in aggiunta i metodi che il modulo utilizza nei rispettivi tipi di database.

Tali metodi dovranno essere implementati dalla classe di uno specifico servizio.

Un primo servizio dell'interfaccia IDatabaseDescriptor sarà quello di mantenere come proprietà *dh* l'istanza di DatabaseHandler associata al servizio. Questa proprietà verrà valorizzata dal modulo nel primo metodo del modulo che il servizio richiamerà con l'interfaccia a parametro (di norma il metodo `*_on_startup`).

Alcune strutture dati memorizzate nella classe DatabaseHandler sono:

*   `int p_id`: Identificativo del servizio.
*   `bool ready`: La classe è pronta a rispondere. Di norma questo booleano viene messo inizialmente a *False* solo nei servizi opzionali. Verrà reimpostato a *True* quando le mappe di partecipazione ai servizi opzionali saranno state recuperate con successo dal nodo che ha fatto ingresso in una rete esistente.

### <a name="Mantenimento_database_distribuito_Repliche"></a>Repliche

Le repliche dei dati aumentano l'affidabilità del database quanto alla *persistenza dei dati*.

Quando un nodo *v* riceve la richiesta di memorizzare (o aggiornare, o rinfrescare) un record con chiave *k* e valore *val* nella sua porzione del database distribuito del servizio *p*, il nodo v si occupa di replicare questo dato su un numero *q*  di nodi replica. L'obiettivo è fare sì che se il nodo muore o si  sconnette dalla rete, alla prossima richiesta di lettura del dato venga  comunque data la risposta corretta. Quindi v deve scegliere i nodi che  saranno contattati per la chiave k quando lui non parteciperà più.

Grazie  all'introduzione del meccanismo di fault tolerance descritto sopra,  scegliere e contattare tali nodi diventa un esercizio banale.

#### <a name="Mantenimento_database_distribuito_Algoritmi"></a>Modifiche agli algoritmi

Quando un nodo avvia l'algoritmo per contattare l'hash-node, gli può passare tra gli argomenti una lista di PeerTupleGNode da usare come `exclude_tuple_list`. Se non la passa allora l'algoritmo ne istanzia una nuova, come faceva prima. Se invece la passa si ottengono 2 risultati:

*   Si può specificare inizialmente un set di g-nodi da escludere;
*   Si può vedere, a risposta ottenuta, quali tuple sono state escluse, per riutilizzarle in una futura chiamata.

Inoltre l'algoritmo va modificato affinché, all'inizio, se nella lista di tuple da escludere vi sono g-nodi che sono visibili nella topologia del nodo corrente questi vengano inclusi anche in `exclude_gnode_list` come istanze HCoord.

Infine l'algoritmo va modificato affinché, a risposta ottenuta, la tupla globale nel g-nodo di ricerca che rappresenta il nodo che ha risposto venga restituita come argomento di out.

La lista completa degli argomenti di `contact_peer` diventa ora:

*   `int p_id`,
*   `PeerTupleNode x̄`,
*   `RemoteCall r`,
*   `int timeout_exec`,
*   `bool exclude_myself`,
*   `out PeerTupleNode respondant`,
*   `PeerTupleGNodeContainer? exclude_tuple_list=null`

Una volta apportate queste modifiche, il nodo *v* che vuole contattare *q* nodi che saranno prossimi alla chiave *k* quando lui non sarà più partecipante procederà così:

*   Prepara, tramite la classe PeerClient, questi dati:
    *   `p_id`, l'id di p;
    *   `x̄` = h<sub>p</sub> ( k );
    *   `r` = "replica il dato k,val nella tua memoria del servizio p";
    *   `timeout_exec`, il tempo di attesa massimo per l'esecuzione di r;
*   `lista_repliche = []`  una lista di istanze di tuple globali nel g-nodo di ricerca.
*   `exclude_tuple_list = []`  una lista di istanze di tuple globali nel g-nodo di ricerca.
*   Mentre `lista_repliche.size`﹤q:
    *   `PeerTupleNode respondant`;
    *   Esegue l'algoritmo di avvio contatto:  
        `ret = contact_peer(p_id, x̄, r, timeout_exec, True, out respondant, exclude_tuple_list)`.
    *   Se si riceve l'eccezione PeersNoParticipantsInNetworkError:
        *   break.
    *   Se si riceve l'eccezione PeersDatabaseError:
        *   break.
    *   Lo specifico servizio può implementare comportamenti particolari, ma di norma se non si è ricevuta una eccezione l'esito scontato è che la replica è avvenuta nel nodo 'respondant'.
    *   Aggiungi respondant a `lista_repliche`.
    *   Aggiungi respondant a `exclude_tuple_list`.

### <a name="Mantenimento_database_distribuito_Ingresso_nuovo_nodo"></a>Ingresso nella rete di un nodo partecipante

Un primo evento che introduce criticità riguardo la *coerenza dei dati* è l'ingresso nella rete di un nuovo nodo che partecipa al servizio.

Per ogni servizio *p* quando un nodo *n* entra nella rete (oppure quando inizia a partecipare al servizio) può  venirgli assegnato un indirizzo prossimo a qualche chiave *k* che in precedenza era stata salvata nel database distribuito con il valore *w*. Ma il nodo *n* non ha informazioni sulla chiave *k*, né sul valore *w*.

Se un nodo *q* cercasse ora di accedere in lettura alla chiave *k* contatterebbe il nodo *n*. Questi non ha il record nella sua memoria, ma se rispondesse che il record non esiste questo sarebbe contrario al requisito di coerenza dei dati. Occorre quindi introdurre il concetto di *esaustività* del nodo rispetto a una chiave *k*.

Descriviamo qui gli aspetti fondamentali di questo concetto, ma diciamo subito che alla fine rimanderemo i dettagli ad altri documenti perché essi sono dipendenti dalle caratteristiche proprie di ogni singolo servizio.

Un nodo *n* servente, che cioè partecipa attivamente al servizio *p*, se riceve una richiesta di qualche tipo riferita alla chiave *k*, si deve chiedere se *può* o *non può* asserire di conoscere l'attuale valore del record per la chiave *k*, o di sapere che tale record non esiste nel database.

Ci riferiamo a questo quando diciamo che il nodo *n* è *esaustivo* o *non esaustivo* per la chiave *k*.

Se un nodo viene interrogato su una chiave *k* e per tale chiave si considera *esaustivo*, allora può asserire di conoscere l'attuale valore del record per la chiave *k* e può elaborare la richiesta.

Se un nodo viene interrogato su una chiave *k* e per tale chiave si considera *non esaustivo*, allora **non** può asserire di conoscere l'attuale valore del record per la chiave *k* e **non** può elaborare la richiesta. Dovrà fare in modo, con diverse strategie a seconda del tipo di servizio come vedremo sotto, di sopperire a questa mancanza oppure di indirizzare il client a contattare un altro nodo servente.

### <a name="Mantenimento_database_distribuito_Recupero_record"></a>Procedimento di recupero di un record

Un nodo servente che si considera *non esaustivo* per una chiave *k*, può decidere di avviare un procedimento di recupero del record. Se e quando lo fa, dipende dallo specifico servizio. Ad esempio un certo servizio potrebbe avere un numero esiguo di chiavi e allora si potrebbe stabilire di ricercare subito i record per tutte le chiavi. Un altro servizio potrebbe avere molte possibili chiavi e allora si potrebbe stabilire di ricercare una chiave solo dopo che qualcuno ne ha fatto richiesta.

Se il nodo decide di avviare il procedimento di recupero di un record, questo introduce una seconda criticità riguardo la *coerenza dei dati*.

Quando abbiamo spiegato cosa si intende per coerenza dei dati abbiamo fatto un esempio. Riprendiamolo e vediamo quale problema può sorgere con un database distribuito:

*   Sia *n0* il nodo al momento detentore della chiave *k*.
*   I nodi *a* e *b* sanno dell'esistenza di *n0*.
*   Sia nella memoria di *n0* l'associazione *k* = *w*.
*   Nasce un nuovo nodo *n1* con un indirizzo più prossimo a *h<sub>p</sub>(k)* rispetto a *n0*.
*   Il nodo *a* non è venuto ancora a conoscenza dell'esistenza di *n1*.
*   __Il nodo *n1* per recuperare il valore associato a *k* contatta *n0* e gli chiede il valore di *k*.__
*   __Il nodo *n0* risponde: "k=w".__
*   __Il nodo *n1* memorizza l'associazione *k* = *w*.__
*   __Il nodo *n1* si ritiene in grado di rispondere a richieste di lettura e scrittura per la chiave *k*.__
*   Il nodo *a* per scrivere sul database contatta *n0* e gli chiede di associare a *k* il valore *v*.
*   Il nodo *n0* memorizza l'associazione *k* = *v* e risponde "OK".
*   Solo dopo aver ricevuto la risposta "OK", cioè dopo aver visto la sua richiesta accettata, il nodo *a* viene a conoscenza dell'esistenza di *n1*.
*   Il nodo *b* viene a conoscenza dell'esistenza di *n1*.
*   Il nodo *b* per leggere dal database contatta *n1* e gli chiede il valore di *k*.
*   Il nodo *n1* risponde: "k=w".

Vediamo anche un altro scenario problematico:

*   Sia *n0* il nodo al momento detentore della chiave *k*.
*   I nodi *a* e *b* sanno dell'esistenza di *n0*.
*   Sia nella memoria di *n0* l'associazione *k* = *w*.
*   Nasce un nuovo nodo *n1* con un indirizzo più prossimo a *h<sub>p</sub>(k)* rispetto a *n0*.
*   Il nodo *a* viene a conoscenza dell'esistenza di *n1*.
*   __Il nodo *n1* per recuperare il valore associato a *k* contatta *n0* e gli chiede il valore di *k*.__
*   __Il nodo *n0* risponde: "k=w".__
*   __Il nodo *n1* memorizza l'associazione *k* = *w*.__
*   __Il nodo *n1* si ritiene in grado di rispondere a richieste di lettura e scrittura per la chiave *k*.__
*   Il nodo *a* per scrivere sul database contatta *n1* e gli chiede di associare a *k* il valore *v*.
*   Il nodo *n1* memorizza l'associazione *k* = *v* e risponde "OK".
*   Il nodo *a* vede la sua richiesta accettata.
*   Il nodo *b* non è venuto ancora a conoscenza dell'esistenza di *n1*.
*   Il nodo *b* per leggere dal database contatta *n0* e gli chiede il valore di *k*.
*   Il nodo *n0* risponde: "k=w".
*   Solo dopo aver ricevuto la risposta "k=w", il nodo *b* viene a conoscenza dell'esistenza di *n1*.

Per rimediare a questi possibili scenari si modificano le operazioni che sono state evidenziate nei due precedenti listati. Quando *n1* vuole recuperare il record per la chiave *k* quale suo nuovo detentore, procede così:

*   Il nodo *n1* per recuperare il valore associato a *k* contatta *n0* e gli chiede: attendi un tempo *𝛿*, poi dammi il valore di *k*.
*   Il tempo *𝛿* è sufficiente a che tutti i nodi si avvedano della presenza di *n1* e eventuali messaggi instradati al vecchio detentore *n0* giungano a destinazione. Chiamiamo questo tempo *tempo critico di coerenza*.  
    Tale tempo dipende quindi dalla dimensione del minimo comune g-nodo tra *n1* e *n0*. Siccome *n0* quando riceve la richiesta conosce l'indirizzo di *n1*, il tempo *𝛿* può essere stimato direttamente dal nodo *n0*.
*   Per il nodo *n0* questa è una richiesta di sola lettura un po' particolare. Quando la riceve il nodo *n0* è esaustivo (per definizione, in quanto si accinge a rispondere): cioè esso ha il record oppure può asserire che in record non esiste. In entrambi i casi, il nodo *n0* attende il tempo *𝛿*. Scaduto il tempo *𝛿* deve di nuovo verificare di essere esaustivo.
    *   Se è ancora esaustivo risponde con il record oppure con un `NOT_FOUND`.
    *   Altrimenti rifiuta l'elaborazione e istruisce il client a riavviare da capo il calcolo di H<sub>t</sub>.
*   Durante questo tempo di attesa, se il nodo *n1* riceve richieste di lettura per la chiave *k* le rifiuta come non esaustivo. Il richiedente si vedrà dirottato, attraverso i meccanismi del calcolo distribuito di H<sub>t</sub>, verso il nodo *n0*.
*   Durante questo tempo di attesa, inoltre, se il nodo *n1* riceve richieste di scrittura per la chiave *k* non le rifiuta subito, ma le tiene in sospeso fino a che non riceve il valore corrente o al massimo fino un po' meno del tempo limite di esecuzione stabilito per la richiesta. Poi non le elabora, in quanto dopo questa attesa potrebbe non essere più il nodo con indirizzo più prossimo all'hash-node, ma istruisce il client di riavviare da capo il calcolo distribuito di H<sub>t</sub>.
*   Al termine di questa attesa, il nodo *n0*, o comunque un nodo attualmente esaustivo per *k*, risponde con il record corrente per *k* oppure con un `NOT_FOUND`. Il nodo *n1* memorizza l'associazione.
*   Il nodo *n1*, come detto, se qualche richiesta di scrittura per la chiave *k* era in attesa, istruisce il client di tali richieste di riavviare da capo il calcolo distribuito di H<sub>t</sub>.
*   Il nodo *n1* in seguito si ritiene *esaustivo* per la chiave *k*.

Vediamo nei due esempi precedenti come questo comportamento risolve il problema.

Esempio 1:

*   Sia *n0* il nodo al momento detentore della chiave *k*.
*   I nodi *a* e *b* sanno dell'esistenza di *n0*.
*   Sia nella memoria di *n0* l'associazione *k* = *w*.
*   Nasce un nuovo nodo *n1* con un indirizzo più prossimo a *h<sub>p</sub>(k)* rispetto a *n0*.
*   Il nodo *a* non è venuto ancora a conoscenza dell'esistenza di *n1*.
*   __Il nodo *n1* per recuperare il valore associato a *k* contatta *n0* e gli chiede: attendi un tempo *𝛿*, poi dammi il valore di *k*.__
*   Il nodo *a* per scrivere sul database contatta *n0* e gli chiede di associare a *k* il valore *v*.
*   Il nodo *n0* memorizza l'associazione *k* = *v* e risponde "OK".
*   Solo dopo aver ricevuto la risposta "OK", cioè dopo aver visto la sua richiesta accettata, il nodo *a* viene a conoscenza dell'esistenza di *n1*.
*   Il nodo *b* viene a conoscenza dell'esistenza di *n1*.
*   Il nodo *b* per leggere dal database contatta *n1* e gli chiede il valore di *k*.
*   Il nodo *n1* rifiuta perché non esaustivo. La ricerca di *b* procede e trova *n0*.
*   Il nodo *n0* risponde: "k=v".
*   __Dopo il tempo *𝛿* il nodo *n0* risponde: "k=v".__
*   __Il nodo *n1* memorizza l'associazione *k* = *v*.__
*   __Il nodo *n1* si ritiene in grado di rispondere a richieste di lettura e scrittura per la chiave *k*.__

Esempio 2:

*   Sia *n0* il nodo al momento detentore della chiave *k*.
*   I nodi *a* e *b* sanno dell'esistenza di *n0*.
*   Sia nella memoria di *n0* l'associazione *k* = *w*.
*   Nasce un nuovo nodo *n1* con un indirizzo più prossimo a *h<sub>p</sub>(k)* rispetto a *n0*.
*   Il nodo *a* viene a conoscenza dell'esistenza di *n1*.
*   __Il nodo *n1* per recuperare il valore associato a *k* contatta *n0* e gli chiede: attendi un tempo *𝛿*, poi dammi il valore di *k*.__
*   Il nodo *a* per scrivere sul database contatta *n1* e gli chiede di associare a *k* il valore *v*.
*   Il nodo *n1* non rifiuta la richiesta, ma la mette in attesa.
*   Il nodo *b* non è venuto ancora a conoscenza dell'esistenza di *n1*.
*   Il nodo *b* per leggere dal database contatta *n0* e gli chiede il valore di *k*.
*   Il nodo *n0* risponde: "k=w".
*   __Dopo il tempo *𝛿* il nodo *n0* risponde: "k=w".__
*   __Il nodo *n1* memorizza l'associazione *k* = *w*.__
*   __Il nodo *n1* istruisce il client (nodo *a*) di riavviare (per sicurezza) il calcolo distribuito di H<sub>t</sub>.__
*   __Il nodo *n1* si ritiene in grado di rispondere a future richieste di lettura e scrittura per la chiave *k*.__
*   Supponiamo che ancora, dal nodo *a*, venga individuato il nodo *n1* come hash-node. Il nodo *n1* elabora la richiesta di *a*: memorizza l'associazione *k* = *v* e risponde "OK".
*   Il nodo *a* vede la sua richiesta accettata. Questa volta la lettura "k=w" da parte del nodo *b* avviene prima che la variazione da parte del nodo *a* venga accettata, quindi questa risposta è accettabile.

Affinché un servente *n1* sia in grado di effettuare il recupero di un record per la chiave *k* dal servente *n0* tramite gli algortmi forniti dal modulo, ci sono alcune regole che la classe del servizio deve rispettare:

*   Deve essere possibile rappresentare la chiave di un record con una istanza di una classe derivata da *Object* che sia serializzabile.
*   Deve essere possibile rappresentare il contenuto di un record con una istanza di una classe derivata da *Object* che sia serializzabile. Tale classe non deve necessariamente contenere anche la chiave.
*   La classe del servizio deve fornire queste ulteriori operazioni:
    *   `bool is_valid_key(Object k)`: Dire se una istanza di Object è una valida chiave.
    *   `List<int> evaluate_hash_node(Object k)`: Calcolare la tupla *h<sub>p</sub>(k)* di questo servizio per la chiave *k*, avendo come requisito (cioè il chiamante deve averlo già verificato) che tale chiave è valida.  
        **Nota**: il calcolo di questa tupla viene di norma svolto dal metodo `perfect_tuple` della classe client. Abbiamo detto che tale metodo è virtuale nella classe base PeerClient: la classe di uno specifico servizio la può reimplementare, in particolare se vuole dare ad alcune chiavi un effetto di visibilità locale del dato, cioè circoscrivere la ricerca dell'hash-node.
    *   `bool key_equal_data(Object k1, Object k2)` e `uint key_hash_data(Object k)`: Metodi le cui firme sono adatte per i delegati `Gee.EqualDataFunc<Object>` e `Gee.HashDataFunc<Object>`, per costruire una HashMap o una lista "con funzionalità di ricerca" di chiavi del servizio, avendo come requisito (cioè il chiamante deve averlo già verificato) che *k*, *k1*, *k2* sono valide chiavi.
    *   `bool is_valid_record(Object k, Object rec)`: Dire se una istanza di Object è un valido record.
    *   `bool my_records_contains(Object k)`: Dire se il record per la chiave *k* per questo servizio è attualmente memorizzato dal nodo. Questa funzione deve garantire di essere atomica, cioè di non schedulare altre tasklet.
    *   `Object get_record_for_key(Object k)`: Restituire il record per la chiave *k*, avendo come requisito (cioè il chiamante deve averlo già verificato) che tale chiave è presente nella memoria. Questa funzione deve garantire di essere atomica.
    *   `void set_record_for_key(Object k, Object rec)`: Mettere in memoria il record *rec* per la chiave *k*, avendo come requisito (cioè il chiamante deve averlo già verificato) che la memoria non sia ancora esaurita. Questa funzione deve garantire di essere atomica.

Il modulo usa la classe RequestWaitThenSendRecord. Si tratta di una classe serializzabile, che deriva da Object, e contiene una istanza di Object serializzabile *k*. Implementa l'interfaccia (vuota) IPeersRequest. È la richiesta di aspettare un tempo *𝛿* (valutato direttamente dal nodo che riceve la richiesta) e poi inviare il record relativo alla chiave *k*. Tale classe non viene esposta dal modulo.

Siccome la richiesta implicitamente dice al nodo servente di attendere un tempo che lui stesso dovrà valutare, il nodo client non può fare altro che accettare una attesa che può arrivare al massimo valore che si può ottenere dalla valutazione del tempo critico di coerenza. Questo valore non dipende dal servizio specifico. Viene quindi memorizzato in un membro statico `timeout_exec` della classe *RequestWaitThenSendRecord*.

Il modulo usa la classe RequestWaitThenSendRecordResponse. Si tratta di una classe serializzabile, che deriva da Object, e contiene una istanza di Object serializzabile *record*. Implementa l'interfaccia (vuota) IPeersResponse. È la risposta alla richiesta sopra descritta. Tale classe non viene esposta dal modulo.

Il modulo usa la classe RequestWaitThenSendRecordNotFound. Si tratta di una classe serializzabile, che deriva da Object, e non ha alcun dato al suo interno. Implementa l'interfaccia (vuota) IPeersResponse. È la risposta come eccezione `NOT_FOUND` alla richiesta sopra descritta. Tale classe non viene esposta dal modulo.

L'interfaccia IDatabaseDescriptor espone i metodi sopra descritti: `is_valid_key`, `evaluate_hash_node`, `key_equal_data`, `key_hash_data`, `is_valid_record`, `my_records_contains`, `get_record_for_key` e `set_record_for_key`.

Per gestire il tempo in cui il nodo *n1* attende la risposta da parte del nodo *n0* relativa al recupero di un record, in altre parole per ricordarsi di quali chiavi è stato avviato un procedimento di recupero ancora in corso, il nodo *n1* fa uso di alcune strutture dati memorizzate nella classe DatabaseHandler:

*   Elenco di chiavi `HashMap<Object,INtkdChannel> retrieving_keys` e per ogni chiave dell'elenco un canale di comunicazione tra tasklet.  
    Se il nodo riceve una richiesta relativa alla chiave *k* che è nell'elenco `retrieving_keys` sa che è ancora in corso il procedimento di recupero per tale chiave. Se vuole, può mettersi in attesa di una comunicazione sul canale relativo, ma sempre indicando un tempo massimo di attesa tale da non superare il tempo limite di esecuzione della richiesta.

### <a name="Mantenimento_database_distribuito_Requisiti_comuni"></a>Requisiti comuni

Le classi dei servizi che implementano un database distribuito devono tenere precisi comportamenti quando ricevono una richiesta. Possono farlo usando alcuni algoritmi forniti dal modulo, diversi a seconda del tipo di servizio.

Questi algoritmi pongono alcune regole che la classe del servizio deve rispettare. Esaminiamo qui alcuni requisiti che sono comuni ai diversi tipi di servizio che in seguito analizzeremo separatamente.

*   Non viene imposto nessun obbligo riguardo il formato delle classi che rappresentano le richieste e le risposte del servizio.
*   La classe del servizio deve essere in grado di effettuare alcune operazioni partendo dalla istanza che rappresenta la richiesta:
    *   `Object get_key_from_request(IPeersRequest r)`: Costruire una istanza della chiave interessata da questa richiesta, avendo come requisito (cioè il chiamante deve averlo già verificato) che dalla richiesta *r* si è in grado di reperire una valida chiave; cioè, se non si è in grado di produrre tale chiave questa operazione può abortire il programma.
    *   `int get_timeout_exec(IPeersRequest r)`: Determinare il tempo limite di esecuzione che il client del servizio si aspetta, avendo come requisito (cioè il chiamante deve averlo già verificato) che la richiesta *r* è di tipo *insert* o *update*; cioè, siccome il modulo dovrebbe farne uso solo in questi casi, in tutti gli altri casi questa operazione può abortire il programma.
    *   `bool is_insert_request(IPeersRequest r)`: Stabilire se la richiesta è di inserimento.  
        Significa che la richiesta è quella di inserire un record nuovo nella memoria del nodo. Quindi la richiesta per essere soddisfatta necessita che il nodo abbia spazio nella sua memoria e sia esaustivo riguardo la non esistenza del record.  
        Se questa operazione restituisce True significa che ha già verificato di poter ottenere la chiave *k* relativa e che tale chiave è valida.
    *   `bool is_read_only_request(IPeersRequest r)`: Stabilire se la richiesta è di sola lettura.  
        Significa che la richiesta per essere soddisfatta necessita che il nodo abbia nella sua memoria il record richiesto. Oppure che il nodo sia esaustivo riguardo la non esistenza del record.  
        Se questa operazione restituisce True significa che ha già verificato di poter ottenere la chiave *k* relativa e che tale chiave è valida.
    *   `bool is_update_request(IPeersRequest r)`: Stabilire se la richiesta è di aggiornamento.  
        Può essere di modifica, di rimozione, o di solo refresh del TTL. Anche qui la richiesta per essere soddisfatta necessita che il nodo abbia nella sua memoria il record richiesto. Oppure che il nodo sia esaustivo riguardo la non esistenza del record.  
        Se questa operazione restituisce True significa che ha già verificato di poter ottenere la chiave *k* relativa e che tale chiave è valida.
    *   `bool is_replica_value_request(IPeersRequest r)`: Stabilire se la richiesta è di replica di valorizzazione di un record.  
        Questa viene accettata se il nodo ha nella sua memoria il record richiesto oppure se ha spazio nella sua memoria. Se invece non lo aveva in memoria e non può memorizzare un altro record, comunque fa in modo di non essere esaustivo riguardo la sua non esistenza, e ovviamente demanda ad altri la elaborazione della richiesta.  
        Se questa operazione restituisce True significa che ha già verificato di poter ottenere la chiave *k* relativa e che tale chiave è valida.
    *   `bool is_replica_delete_request(IPeersRequest r)`: Stabilire se la richiesta è di replica di cancellazione di un record.  
        Questa viene accettata sempre: se il nodo ha nella sua memoria il record esso viene rimosso; in ogni caso il nodo diventa esaustivo riguardo la non esistenza del record.  
        Se questa operazione restituisce True significa che ha già verificato di poter ottenere la chiave *k* relativa e che tale chiave è valida.
    *   **Nota**: una richiesta valida per il servizio potrebbe non essere di nessuno di questi 5 tipi (insert, read only, update, replica valore, replica cancellazione).  
        Oltre alle richieste "interne" al modulo (come la RequestWaitThenSendRecord) anche richieste specifiche del servizio possono non essere di questi 5 tipi. Ad esempio: una richiesta si basa su una chiave *k* e la sua risposta non dipende dall'esistenza di un record. Quindi la richiesta può essere soddisfatta da un nodo indipendentemente dallo spazio nella sua memoria e dal fatto che sia esaustivo per la chiave *k*.
    *   `IPeersResponse prepare_response_not_found(IPeersRequest r)`: Costruire una risposta che indica l'eccezione `NOT_FOUND` per la chiave interessata.
    *   `IPeersResponse prepare_response_not_free(IPeersRequest r, Object rec)`: Costruire una risposta che indica l'eccezione `NOT_FREE` per la chiave interessata, opzionalmente con (alcuni) dati del record che adesso vi è associato.
    *   `IPeersResponse execute(IPeersRequest r)`: Elaborare la richiesta e restituire la risposta. Questa operazione viene chiamata se:

        *   la richiesta è di inserimento e il nodo la può elaborare;
        *   la richiesta è di sola lettura e il nodo la può elaborare;
        *   la richiesta è di aggiornamento e il nodo la può elaborare;
        *   la richiesta è di replica di valorizzazione e il nodo la può elaborare;
        *   la richiesta è di replica di cancellazione e il nodo la può elaborare;
        *   la richiesta non è di nessuno dei 5 casi suddetti.  

        Nell'ultimo caso la classe deve fare tutti i controlli necessari: non è detto che una chiave possa essere recuperata, né che la chiave, se c'è, sia valida.  
        L'operazione `execute` può rilanciare le eccezioni PeersRefuseExecutionError e PeersRedoFromStartError, come il metodo `exec` della classe PeerService. Di norma non dovrebbe farlo nei primi 5 casi sopra elencati; anche nell'ultimo caso, di norma se l'operazione non può considerarsi completata con successo va restituita una risposta che rappresenti una eccezione prevista dallo specifico servizio. Tuttavia viene lasciata questa possibilità.

L'interfaccia IDatabaseDescriptor espone anche i metodi sopra descritti: `get_key_from_request`, `get_timeout_exec`, `is_insert_request`, `is_read_only_request`, `is_update_request`, `prepare_response_not_found`, `prepare_response_not_free` e `execute`.

### <a name="Mantenimento_database_distribuito_Requisiti_specifici"></a>Requisiti specifici

Abbiamo identificato alcune tipologie di servizi (al momento due) che usano distinti approcci all'uso del concetto di esaustività; per ogni classe rimanderemo ad un documento di dettaglio.

#### <a name="Mantenimento_database_distribuito_TTL"></a>Database con record che hanno un TTL

Consideriamo un servizio che vuole mantenere un database che abbia queste caratteristiche:

*   Ogni record in esso ha un Time To Live, cioè una scadenza. Se prima della scadenza non viene aggiornato, il record viene rimosso dal database.
*   Le chiavi sono in un set che può essere molto grande. E' impraticabile scorrere tutto il set delle possibili chiavi.
*   Per una chiave, nel database può esserci un record o nessuno.
*   Ogni nodo partecipante al servizio ha un limite di memoria, oltre il quale il nodo rifiuterà di immagazzinare altri record.

Per facilitare l'implementazione di un servizio con queste caratteristiche, il modulo fornisce alcuni algoritmi. Questi algoritmi usano i requisiti descritti sopra e altri descritti nell'interfaccia ITemporalDatabaseDescriptor.

Esaminiamo nel dettaglio questi algoritmi nel documento [Database TTL](DatabaseTTL.md).

#### <a name="Mantenimento_database_distribuito_FixedKeys"></a>Database con un numero esiguo e fisso di chiavi

Consideriamo un servizio che vuole mantenere un database che abbia queste caratteristiche:

*   I record non hanno una scadenza.
*   Le chiavi sono in un dominio ben definito e non molto grande.
*   Ad una chiave è associato sempre un valore, a partire da un valore di default. Non può essere rimosso il record associato ad una chiave, ma solo cambiato.
*   Ogni nodo è in grado di memorizzare i record anche di tutte le chiavi. Non esiste il caso `OUT_OF_MEMORY`.

Per facilitare l'implementazione di un servizio con queste caratteristiche, il modulo fornisce alcuni algoritmi. Questi algoritmi usano i requisiti descritti sopra e altri descritti nell'interfaccia IFixedKeysDatabaseDescriptor.

Esaminiamo nel dettaglio questi algoritmi nel documento [Database a chiavi fisse](DatabaseFixedKeys.md).

## <a name="Algoritmi"></a>Algoritmi

Gli algoritmi in dettaglio sono illutrati nei seguenti documenti:

*   [Strutture dati](AppuntiStrutture.md)
*   [Metodi helper](AppuntiMetodiHelper.md)
*   [Algoritmi di instradamento](AppuntiAlgo1.md)
*   [Algoritmi complementari](AppuntiAlgo2.md)

