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

**TODO**

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
