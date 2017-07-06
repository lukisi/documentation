# Modulo Coordinator - Analisi Funzionale

1.  [Il servizio Coordinator](#Servizio_coordinator)
    1.  [Servizi previsti](#Servizi_previsti)
        1.  [Prenota un posto](#Prenota_un_posto)
1.  [Richiesta al diretto vicino di accesso al servizio Coordinator](#Richiesta_al_diretto_vicino)
    1.  [Non sono ancora nella rete](#Per_ingresso)
    1.  [Sono in un isola che deve migrare](#Per_migrazione)
1.  [Requisiti](#Requisiti)
1.  [Deliverables](#Deliverables)
1.  [Classi e interfacce](#Classi_e_interfacce)

Il modulo cerca di fare sì che un singolo g-nodo di livello *l*, sebbene costituito da un numero di singoli nodi, abbia un comportamento coerente come singola entità.

Il modulo fa uso delle [tasklet](../Librerie/TaskletSystem.md), un sistema di multithreading cooperativo.

Il modulo fa uso del framework [ZCD](../Librerie/ZCD.md), precisamente appoggiandosi ad una libreria intermedia prodotta con questo framework per formalizzare i metodi remoti usati nel demone *ntkd*.

Il modulo fa uso diretto delle classi e dei servizi forniti dal modulo [PeerServices](../ModuloPeers/AnalisiFunzionale.md). In particolare, esso realizza un servizio peer-to-peer, chiamato appunto Coordinator, per mezzo del quale svolge alcuni dei suoi compiti.

## <a name="Servizio_coordinator"></a>Il servizio Coordinator

Il servizio Coordinator è un servizio non opzionale, cioè tutti i nodi partecipano attivamente. Si tratta di un servizio che mantiene un database distribuito della tipologia a chiavi fisse.

Lo scopo del servizio Coordinator è quello di realizzare una sorta di memoria condivisa di un g-nodo *g*. Quando un nodo *n* vuole accedere alla memoria condivisa del suo g-nodo *g* di livello *l*, con 0 < *l* ≤ *levels*, fa una richiesta al servizio Coordinator usando come chiave il livello *l*.

Il livello *l* è maggiore di 0 poiché non serve realizzare una memoria condivisa per il singolo nodo. Può essere uguale al numero dei livelli, poiché è necessario avere una memoria condivisa anche per l'unico g-nodo di livello *l* = *levels* che costituisce l'intera rete.

Lo spazio delle chiavi definito dal servizio Coordinator è appunto l'insieme dei valori che può assumere *l*.

La funzione *h<sub>p</sub>* è definita dal servizio in modo da dare ai dati la visibilità locale circoscritta al g-nodo in esame. In altre parole, il nodo corrente può contattare solo il Coordinator di uno dei suoi g-nodi; sia l'hash-node che il nodo che risponde si trovano all'interno del g-nodo stesso.

### <a name="Servizi_previsti"></a>Servizi previsti

Elenchiamo tutte le richieste che si possono fare al Coordinator.

#### <a name="Prenota_un_posto"></a>Prenota un posto

La richiesta *r* di prenotare un posto può arrivare ad un nodo *x* come Coordinator di un g-nodo *g* di livello *l*, con 0 < *l* ≤ *levels*. I membri di *r* sono:

*   *lvl* = livello di *g*.

Il nodo *x* valuta se ci sono posti liberi in *g* considerando la sua mappa dei percorsi. Deve considerare anche le prenotazioni concesse in precedenza, le quali restano valide per un certo tempo anche se ancora non sono nella sua mappa perché non sono ancora state confermate da un ETP.

Se un posto è disponibile il nodo *x* lo prenota. Questo equivale ad una scrittura nella memoria condivisa, quindi è necessario provvedere anche alle repliche con il meccanismo fornito dal modulo PeerServices.

Il nodo che fa la richiesta è un nodo *n* che già appartiene al g-nodo *g*. Questi richiede la prenotazione di un nuovo posto per conto di un altro nodo suo vicino, *m*, il quale non è ancora in *g* o perfino non è ancora nella rete.

Nella risposta al nodo *n*, *x* segnala:

*   La posizione assegnata all'interno di *g*.
*   La anzianità della posizione assegnata all'interno di *g*.

Altre informazioni di cui il nodo *m* necessita per fare ingresso in *g* (e eventualmente nella rete) sono direttamente fornite dal nodo *n* che già le conosce. Queste sono:

*   La topologia della rete.
*   Le posizioni di *g* e dei suoi g-nodi superiori.
*   L'anzianità di *g* e dei suoi g-nodi superiori.

A questo punto il nodo *n* comunica tutte queste informazioni al suo vicino *m*.

Se nessun posto è disponibile il nodo *x* lo segnala con una eccezione, che viene ricevuta da *n*. Anche in questo caso *n* comunica l'esito al vicino *m*.

## <a name="Richiesta_al_diretto_vicino"></a>Richiesta al diretto vicino di accesso al servizio Coordinator

In alcuni casi un nodo *n* può voler chiedere ad un suo diretto vicino di accedere al servizio peer-to-peer del Coordinator.

### <a name="Per_ingresso"></a>Non sono ancora nella rete

Si consideri un nodo *n* che non ha fatto ancora ingresso in una rete. Comunicando con un suo vicino *v*, il nodo *n* apprende che *v* appartiene al g-nodo *g* e che in tale g-nodo c'è ancora spazio. Vorrebbe quindi richiedere l'accesso nel g-nodo *g*. In questo caso *n* chiede a *v* di accedere alla memoria condivisa del g-nodo *g* per cercare di riservargli un posto.

### <a name="Per_migrazione"></a>Sono in un isola che deve migrare

Si consideri un nodo *n* che fa parte di un g-nodo *g* di livello *l*. Il g-nodo *g* fa a sua volta parte di un g-nodo *h* di livello *l* + 1. Supponiamo che il g-nodo *g* diventi disconnesso, cioè avviene lo split del g-nodo, mentre *h* è ancora connesso. Supponiamo che l'isola in cui si trova *n* sia una di quelle che non contengono il nodo più anziano. Supponiamo che *n* abbia un vicino *v* che appartiene a *h* ma non appartiene a *g*. Per questo il nodo *n* riceve da *v* l'informazione che tutta la sua isola (di livello *l*) deve cambiare indirizzo.

In questo momento il nodo *n* non ha più un indirizzo valido in *g* e non è sicuro nemmeno che ci sia ancora spazio per un nuovo g-nodo in *h*.

Supponiamo che a seguito di tale evento, per una strategia che non è di pertinenza di questo modulo, il nodo *n* voglia accedere alla memoria condivisa del g-nodo *h*. Se cercasse di farlo autonomamente potrebbe risultare dal calcolo dell'hash-node che esso si trova proprio in *g*. Ma il nodo *n* non è più parte del vero *g* e non può nemmeno avere percorsi nella sua mappa che lo colleghino al vero *g*. Allora *n* deve chiedere a *v* di accedere in vece sua alla memoria condivisa del g-nodo *h*. Anche se l'hash-node si trova proprio in *g* il nodo *v* saprà instradare il messaggio verso il vero *g*.

## <a name="Requisiti"></a>Requisiti

*   Il livello a cui il nodo ha costituito un g-nodo nuovo.

Durante le sue operazioni, il modulo viene informato quando il nodo ha completato la fase di bootstrap. In quello stesso momento gli vengono forniti:

*   L'istanza di PeersManager.
*   Mappa delle posizioni libere/occupate ai vari livelli.

## <a name="Deliverables"></a>Deliverables

Fornisce metodi per:

*   Chiedere ad un vicino *v*, dato uno stub per contattarlo, quanti posti vede liberi (nella sua mappa, senza contattare i singoli Coordinator) nei suoi g-nodi. Metodo `get_neighbor_map`.  
    Questo metodo può rilanciare l'eccezione CoordinatorStubNotWorkingError se la comunicazione con il vicino non riesce.  
    Questo metodo può rilanciare l'eccezione CoordinatorNodeNotReadyError se il vicino *v* non ha ancora completato la fase di boostrap (vedi modulo [QSPN](../ModuloQspn/EsplorazioneRete.md#Rete_esplorata)). Infatti il nodo *v* non è in grado di rispondere alle richieste dell'interfaccia ICoordinatorMap, quindi il modulo Coordinator nel nodo *v* non ha ancora ricevuto l'istanza di tale interfaccia.  
    Se invece non sono rilanciate eccezioni, il metodo restituisce una istanza di ICoordinatorNeighborMap.  
    Dalla risposta deve essere possibile leggere queste informazioni:

    *   Il numero di livelli nella topologia.
    *   La gsize di ogni livello.
    *   Il numero di posti liberi in ogni livello.

    Quando un nodo *n* chiede al vicino *v* quanti posti liberi vede nella sua mappa, non assumiamo che il nodo *v* appartenga già alla stessa rete di *n*; quindi nemmeno che abbiano la stessa topologia.  
    La topologia della rete in cui si vuole fare ingresso è importante che sia nota. Infatti il nodo richiedente potrebbe essere un gateway verso una rete privata in cui si vogliono adottare diversi meccanismi di assegnazione di indirizzi e routing. In questo caso il gateway potrebbe volere una assegnazione di un g-nodo di livello tale da poter disporre di un certo spazio (numero di bits) per gli indirizzi interni.  
    Il numero di posti liberi in un dato livello potrebbe essere una informazione eccessiva. Probabilmente al nodo richiedente è sufficiente sapere se c'è almeno un posto o no. Per ora manteniamo questa informazione.
*   Dato un livello *l*, chiedere ad un vicino *v*, dato uno stub per contattarlo, di richiedere al Coordinator del suo g-nodo di livello *l* la prenotazione di un posto, come nuovo g-nodo di livello *l* - 1. Metodo `get_reservation`.  
    Questo metodo può rilanciare l'eccezione CoordinatorStubNotWorkingError se la comunicazione con il vicino non riesce.  
    Questo metodo può rilanciare l'eccezione CoordinatorNodeNotReadyError se il vicino *v* non ha ancora completato la fase di boostrap (vedi modulo QSPN). Infatti il nodo *v* non ha ancora potuto instanziare il suo PeersManager, né il CoordinatorService.  
    Questo metodo può rilanciare l'eccezione CoordinatorInvalidLevelError, segnalata immediatamente dal vicino, se il livello richiesto non è coerente con la topologia della rete in cui si trova il vicino.  
    Questo metodo può rilanciare l'eccezione CoordinatorSaturatedGnodeError, segnalata dal vicino dopo aver comunicato con il servizio Coordinator, se al livello richiesto non è stato possibile riservare un posto.  
    Di solito questa operazione si fa subito dopo aver ottenuto dallo stesso vicino la lista del numero di posti che vede liberi. Se da questa lista il livello *l* risultava avere posti liberi, ma la richiesta della prenotazione rilancia l'eccezione CoordinatorSaturatedGnodeError, tale esito potrebbe essere dovuto alle precedenti prenotazioni non ancora confermate da un ETP. Quindi avrebbe poco senso chiedere di nuovo al vicino quanti posti vede liberi. Si assuma che al livello *l* i posti sono 0 e si continui a ritenere validi i valori per gli altri livelli, sia superiori che inferiori.  
    Se invece non sono rilanciate eccezioni, il metodo restituisce una istanza di ICoordinatorReservation. Con i dati contenuti in questa istanza (si veda sotto il dettaglio) il nodo potrà costituire un nuovo g-nodo.

Implementa il servizio Coordinator derivando la classe CoordinatorService dalla classe base PeerService.

Il modulo Coordinator si occupa di registrare con il PeersManager l'implementazione del servizio Coordinator e di usare gli algoritmi forniti dal modulo PeerServices per il mantenimento del relativo database a chiavi fisse. Cioè, esso crea una istanza della classe CoordinatorService.DatabaseDescriptor, che implementa IFixedKeysDatabaseDescriptor. Con questa istanza come parametro, al bisogno, chiama i metodi `fixed_keys_db_on_startup` e `fixed_keys_db_on_request` di PeersManager.

Deriva la classe CoordinatorClient dalla classe base PeerClient per avviare il contatto del Coordinator di un suo g-nodo e richiederne i servizi.

## <a name="Classi_e_interfacce"></a>Classi e interfacce

La mappa delle posizioni libere/occupate ai vari livelli è un oggetto di cui il modulo conosce l'interfaccia ICoordinatorMap. Tramite essa il modulo può:

*   Leggere il numero *l* dei livelli della topologia (metodo `get_levels`).
*   Leggere la gsize di ogni livello *i* da 0 a *l* - 1 (metodo `get_gsize`).  
    Precisiamo il significato di questo indice, restando coerenti con quanto stabilito nella trattazione del modulo QSPN sebbene i due moduli siano indipendenti.  
    Per ogni *i* da 0 a *l* - 1, *gsize(i)* è il numero massimo di g-nodi di livello *i* in un g-nodo di livello *i* + 1.
*   Leggere l'anzianità del mio g-nodo ad ogni livello *i* da 0 a *l* - 1 (metodo `get_eldership`).  
    L'anzianità di un g-nodo è un numero progressivo che viene assegnato al g-nodo. Nel confronto tra due g-nodi di pari livello *i* entrambi appartenenti allo stesso g-nodo di livello *i* + 1, un valore più alto significa che il g-nodo è arrivato dopo, cioè esso è più giovane.
*   Leggere la posizione del nodo, cioè la posizione del mio g-nodo ad ogni livello *i* da 0 a *l* - 1 (metodo `get_my_pos`).
*   Leggere l'elenco delle posizioni libere in ogni livello *i* da 0 a *l* - 1 (metodo `get_free_pos`).  
    Cioè le posizioni nel livello *i* che sono libere nel nostro g-nodo di livello *i* + 1. Questa informazione è quella che si basa sulla mappa del nodo corrente, senza contattare il Coordinator attuale che ha la conoscenza autoritativa delle prenotazioni pendenti.

* * *

Il metodo `get_neighbor_map` restituisce una istanza di ICoordinatorNeighborMap. Tramite questa interfaccia il modulo può:

*   Leggere il numero di livelli nella topologia (metodo `get_levels`).
*   Leggere la gsize per ogni livello *i*, con *i* da 0 a *levels* - 1 (metodo `get_gsize`).
*   Leggere il numero di posti liberi in ogni livello *i*, con *i* da 0 a *levels* - 1 (metodo `get_free_pos_count`).

* * *

Il metodo `get_reservation` restituisce una istanza di ICoordinatorReservation. Tramite questa interfaccia il modulo può:

*   Leggere il numero di livelli nella topologia (metodo `get_levels`).
*   Leggere la gsize per ogni livello *i*, con *i* da 0 a *levels* - 1 (metodo `get_gsize`).
*   Leggere il livello, la posizione e l'anzianità del g-nodo appena riservato (metodi `get_lvl`, `get_pos` e `get_eldership`).
*   Leggere la posizione del g-nodo in cui siamo entrati per ogni livello *i*, con *i* da *lvl* + 1 fino a *levels* - 1, inclusi (metodo `get_upper_pos`).
*   Leggere il valore di anzianità per ogni livello *i*, con *i* da *lvl* + 1 fino a *levels* - 1, inclusi (metodo `get_upper_eldership`).  
    Per i livelli inferiori a *lvl* il nodo, che ha appena costituito un nuovo g-nodo di livello *lvl*, userà come anzianità il valore 0 e come posizione un valore random congruente con la topologia.

