# Comunicazioni in rete

## Panoramica

Diversamente dal caso banale:

*   Il programma si troverà a trasmettere in unicast (stream) alcuni
    messaggi, poiché il nodo costituirà un arco con un altro nodo.
*   Il programma si troverà a rilevare alcuni messaggi in broadcast (datagram)
    sull'una o l'altra interfaccia. Alcuni saranno messaggi provenienti dal
    nodo stesso **e** altri da un altro nodo.
*   Il programma si troverà a rilevare alcuni messaggi in unicast (stream).

## Strutture dati

Come nel caso banale.

## Gestione messaggi in entrata

Come nel caso banale.

Mentre nel caso banale la classe ServerDelegate (nel metodo `get_addr_set`
prescritto da `ntkdrpc`) non si occupava di gestire la ricezione di un
messaggio di tipo stream, in questo scenario questi messaggi vanno gestiti.

Per gestirli il metodo `get_addr_set` chiama il metodo `get_dispatcher`
di `N.SkeletonFactory` passando il CallerInfo fornito da ZCD e poi se
ottiene uno stub lo mette come unico elemento di una lista. Altrimenti
restituisce una lista vuota.  
Il metodo `get_dispatcher` se il CallerInfo contiene un `WholeNodeSourceID`
e un `WholeNodeUnicastID` e quest'ultimo ha un `NeighborhoodNodeID` uguale a
quello memorizzato nel mio `node_skeleton.id` (o `whole_node_id`)
restituisce il proprio `node_skeleton`.  
Se invece il CallerInfo contiene un `IdentityAwareSourceID`... **TODO**

* * *

Per quanto riguarda la ricezione di messaggi di tipo datagram, essi sono
sempre identificati dalla classe ServerDelegate nel metodo `get_addr_set`.
Essi sono gestiti chiamando il metodo `get_dispatcher_set`
di `N.SkeletonFactory` passando il CallerInfo fornito da ZCD.
Mentre nel caso banale se il caller ha in `source_id` un `IdentityAwareSourceID`
il messaggio non viene gestito (poiché non ci saranno archi) in questo scenario
questi messaggi vanno gestiti.

**TODO**

### Reperimento info aggiuntive dal CallerInfo

Come nel caso banale.

Mentre nel caso banale la classe `N.SkeletonFactory` nel
metodo `from_caller_get_nodearc` semplicemente non esiste,
in questo scenario il metodo deve...

**TODO**

### Reperimento degli Skeleton specifici di modulo

Come nel caso banale.

## Gestione messaggi in uscita

Come nel caso banale.

### Trasmissione broadcast a modulo di nodo

Come nel caso banale.

### Trasmissione broadcast a modulo di identità

Mentre nel caso banale questa non era necessaria, nel presente scenario
occorre implementare la
funzione `get_stub_identity_aware_broadcast` e altre a corredo.

**TODO**

### Trasmissione unicast a modulo di nodo

Mentre nel caso banale questa non era necessaria, nel presente scenario
occorre implementare il metodo `get_stub_whole_node_unicast` di `N.StubFactory`.

Il metodo `get_stub_whole_node_unicast` deve restituire uno stub radice.  
Gli viene passato l'arco, cioè l'istanza di `N.N.INeighborhoodArc`.  
Il metodo prepara i dati serializzabili che sono previsti dal protocollo
ZCD, cioè:

*   Un `WholeNodeSourceID` che rappresenta il mittente.  
    Il `NeighborhoodNodeID`, che serve per crearlo, viene preso dalla variabile
    globale `skeleton_factory`.
*   Un `WholeNodeUnicastID` che rappresenta il destinatario.  
    Il `NeighborhoodNodeID`, che serve per crearlo, viene preso dal membro
    `neighbour_id` dell'arco passato.
*   Un `NeighbourSrcNic` che identifica l'interfaccia di rete (del mittente)
    usata per la trasmissione.  
    Il MAC address che serve (della propria scheda di rete) viene preso dal
    membro `mac` del membro `nic` (a sua volta una istanza
    di `N.N.INeighborhoodNetworkInterface`) dell'arco passato.

Poi il metodo usa la libreria `ntkdrpc` basata su ZCD per ottenere uno stub
radice di tipo datagram sul medium system; cioè chiama la funzione
`get_addr_stream_system`.  
Per comporre la stringa `send_pathname` prescritta da `ntkdrpc`, la classe
`N.StubFactory` usa l'IP linklocal che identifica univocamente la scheda di rete
del vicino su cui è formato l'arco. Essa è nel membro `neighbour_nic_addr`
dell'arco passato.

### Trasmissione unicast a modulo di identità

Mentre nel caso banale questa non era necessaria, nel presente scenario
occorre implementare il metodo `get_stub_identity_aware_unicast_from_ia`
di `N.StubFactory`.

Il metodo `get_stub_identity_aware_unicast_from_ia` ... **TODO**  
