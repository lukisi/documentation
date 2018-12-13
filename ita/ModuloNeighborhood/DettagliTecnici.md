# Modulo Neighborhood - Dettagli Tecnici

1.  [Requisiti](#requisiti)
1.  [Deliverable](#deliverable)
1.  [Rilevamento dei vicini, costituzione degli archi, misurazione dei costi](#rilevamento-dei-vicini-costituzione-degli-archi-misurazione-dei-costi)
1.  [Indirizzo IPv4 di scheda](#indirizzo-ipv4-di-scheda)

## Requisiti

L'utilizzatore del modulo Neighborhood per prima cosa inizializza il modulo richiamando il metodo statico
`init` di NeighborhoodManager. In tale metodo viene anche passata l'istanza di ITasklet per fornire
l'implementazione del sistema di tasklet.

Dopo istanzia il suo NeighborhoodManager passando al costruttore:

*   `int max_arcs` - Il numero massimo di archi.
*   `INeighborhoodStubFactory stub_factory` - La stub factory.
*   `INeighborhoodQueryCallerInfo query_caller_info` - Il delegato per identificare il chiamante.
*   `INeighborhoodIPRouteManager ip_mgr` - Il manager di indirizzi.
*   `NewLinklocalAddress new_linklocal_address` - Funzione per generare un indirizzo IP link-local.

Dopo, per ogni interfaccia di rete che intende gestire, richiama sul NeighborhoodManager il metodo
`start_monitor(nic)`, dove *nic* è una INeighborhoodNetworkInterface.

In seguito l'utilizzatore del modulo può aggiungere altre interfacce di rete da gestire, oppure rimuovere
una interfaccia con il metodo `stop_monitor(dev)`, dove *dev* è il nome dell'interfaccia.

## Deliverable

Il modulo segnala l'avvenuta assegnazione dell'indirizzo di scheda ad una interfaccia di rete gestita
attraverso il segnale `nic_address_set` di NeighborhoodManager.

In tale segnale viene riportato:

*   Il nome dell'interfaccia di rete. Una stringa `dev`. Es. "eth0".
*   L'indirizzo assegnato. Una stringa `address`. Es. "169.254.23.45".

* * *

Il modulo segnala la costituzione di un arco attraverso il segnale `arc_added` di NeighborhoodManager.

In tale segnale viene riportato:

*   L'arco costituito. Un INeighborhoodArc `arc`.

In realtà, prima l'arco viene costituito e viene aggiunta nelle tabelle (nel network namespace default) la
rotta verso l'indirizzo di scheda del vicino. Poi, soltanto se entrambi i nodi sono disponibili, viene deciso
di esporre l'arco all'utilizzatore del modulo. Solo a questo punto il segnale viene emesso.

* * *

Il modulo segnala che sta per rimuovere un arco (che era stato precedentemente esposto) attraverso il
segnale `arc_removing` di NeighborhoodManager.

In tale segnale viene riportato:

*   L'arco che sta per essere rimosso. Un INeighborhoodArc `arc`.
*   Un booleano `is_still_usable` che dice se l'arco è ancora utilizzabile per comunicare con il sistema vicino.

Il segnale comunica all'utilizzatore che il modulo sta per rimuovere dalle tabelle (nel network namespace default) la rotta verso
quell'indirizzo di scheda che è riportato nell'istanza di INeighborhoodArc.

La rimozione di un arco avviene con il metodo `remove_my_arc` che (come illustreremo sotto) è pubblico. Quando
si chiama questo metodo si specifica anche (argomento booleano `is_still_usable`) se si ritiene che l'arco sia
ancora funzionante. Se si ritiene (a torto o a ragione) che sia funzionante, verrà tentata una ultima comunicazione
(comunque in broadcast senza alcuna verifica di ricezione) affinché anche l'altro capo rimuova l'arco dalla sua lista.

La chiamata al metodo `remove_my_arc` avviene in due tipi di situazione. Nel primo caso è perché, il modulo 
Neigborhood stesso oppure un altro, si è avveduto che l'arco è effettivamente non funzionante. Nel secondo
caso è perché l'utilizzatore, per altri motivi, ha richiesto al modulo di rimuovere l'arco che ancora si presume essere
funzionante.

Nella esecuzione del metodo `remove_my_arc`, solo se l'arco era stato esposto dal modulo Neighborhood,
prima della rimozione viene emesso il segnale `arc_removing`.  
Poi viene rimosso l'arco; viene tentata, se richiesto, l'ultima comunicazione affinché anche l'altro
capo rimuova l'arco dalla sua lista.  
Infine, solo se l'arco era stato esposto, verrà emesso il segnale `arc_removed` (che vedremo sotto).

* * *

Il modulo segnala l'avvenuta rimozione di un arco (che era stato precedentemente esposto) attraverso il
segnale `arc_removed` di NeighborhoodManager.

In tale segnale viene riportato:

*   L'arco rimosso. Un INeighborhoodArc `arc`.

Il segnale significa anche che è stata rimossa dalle tabelle (nel network namespace default) la rotta verso
quell'indirizzo di scheda che è riportato nell'istanza di INeighborhoodArc.

* * *

Il modulo segnala la variazione del costo di un arco (che era stato precedentemente esposto) attraverso il
segnale `arc_changed` di NeighborhoodManager.

In tale segnale viene riportato:

*   L'arco. Un INeighborhoodArc `arc`.

* * *

Il modulo segnala l'avvenuta rimozione dell'indirizzo di scheda ad una interfaccia di rete che non si gestisce
più attraverso il segnale `nic_address_unset` di NeighborhoodManager.

In tale segnale viene riportato:

*   Il nome dell'interfaccia di rete. Una stringa `dev`. Es. "eth0".
*   L'indirizzo assegnato. Una stringa `address`. Es. "169.254.23.45".

* * *

Il modulo permette di ottenere l'identificativo assegnato a questo nodo con il
metodo `get_my_neighborhood_id` di NeighborhoodManager.

* * *

Il modulo permette di ottenere l'elenco degli archi ora presenti con il metodo `current_arcs` di NeighborhoodManager.

* * *

Il modulo permette di forzare la rimozione di un arco con il metodo `remove_my_arc` di NeighborhoodManager.

Questo metodo è pubblico perché può essere l'utilizzatore del modulo che vuole per qualche motivo forzare la rimozione
di un arco. Ma anche il modulo stesso può usarlo internamente in alcune occasioni.

In entrambi i casi, il chiamante del metodo può specificare che si possa o meno tentare una ulteriore comunicazione
all'altro capo dell'arco. Lo si specifica con l'argomento booleano `is_still_usable`. In questa ultima comunicazione,
se richiesta, si trasmette in broadcast sull'interfaccia di rete dell'arco il messaggio `remove_arc` affinché anche
l'altro capo rimuove l'arco dalla sua lista.

## Rilevamento dei vicini, costituzione degli archi, misurazione dei costi

Quando viene istanziato il NeighborhoodManager il modulo genera il suo NeighborhoodNodeID.

Quando viene chiamato il metodo `start_monitor` sulla istanza di NeighborhoodManager, il modulo inizia a gestire una
interfaccia di rete. Di essa riceve il nome e il MAC address. Per prima cosa verifica che questi valori siano univoci.
Cicla le interfacce che ha già in gestione; se una ha gli stessi valori per nome e MAC allora ignora questa richiesta
che è un duplicato; se una ha lo stesso MAC e nome diverso oppure lo stesso nome e MAC diverso allora il modulo va in errore fatale.

Il modulo, per ogni interfaccia che inizia a gestire, memorizza il suo nome e il suo MAC address e genera e memorizza un
indirizzo locale di scheda. Usa l'oggetto INeighborhoodIPRouteManager (descritto sotto)
per impostare l'indirizzo generato. Poi emette il segnale `nic_address_set`.

Poi avvia una tasklet nella quale (usando `get_broadcast_for_radar` della stub factory) trasmette in broadcast solo su quella
interfaccia di rete il messaggio "ci sono io" (`here_i_am`) indicando il suo NeighborhoodNodeID, il MAC della interfaccia e il suo indirizzo locale di scheda.
Poi ripete lo stesso messaggio con bassa frequenza, ogni minuto.

I nodi che sono nel dominio broadcast vedono questo messaggio e possono fin da subito accordarsi per un arco. Quelli che hanno
già costituito un arco verso quel MAC address ignorano il nuovo messaggio.

Un nodo che vuole accordarsi per un arco con un vicino di cui ha notato la presenza,
prima usa l'oggetto INeighborhoodIPRouteManager per impostare la rotta verso l'indirizzo di scheda del nuovo vicino,
poi invia un messaggio "facciamo un arco" (`request_arc`)
sempre in broadcast solo sull'interfaccia dove ha ricevuto il messaggio "ci sono io", indicando sia i dati ricevuti
nel precedente messaggio "ci sono io" (NeighborhoodNodeID, MAC e linklocal del nodo vicino) sia i rispettivi dati
del proprio nodo.

Alla ricezione del messaggio "facciamo un arco" anche il vicino appena rilevato da parte sua
usa l'oggetto INeighborhoodIPRouteManager per impostare la rotta verso l'indirizzo di scheda del nuovo vicino.
Inoltre, questi sa che il vicino (quello che ha trasmesso "facciamo un arco") aveva già
impostato la rotta verso il suo indirizzo di scheda. Quindi da subito può realizzare una connessione
reliable (con protocollo TCP) con esso.

A questo punto entrambi i nodi avranno un arco che ancora non è esposto dal modulo e il cui costo è ancora non misurato.

Il nodo che ha ricevuto il messaggio "facciamo un arco" realizza ora una connessione TCP
(usando `get_unicast` della stub factory) per chiamare sul nuovo vicino il metodo
remoto "esponi l'arco". In questo metodo, con la firma `bool can_you_export(bool i_can_export)`, entrambi i nodi dichiarano
la loro disponibilità ad esporre l'arco dal modulo. Come detto in precedenza, se uno rifiuta anche l'altro deve
evitare di esporre l'arco.

Se entrambi i nodi sono disposti ad esporre l'arco, il costo dell'arco non è stato però ancora misurato.
Un arco il cui costo non è ancora stato misurato non va a far parte della lista ufficiale che il modulo Neighborhood
espone all'applicazione.

Se entrambi i nodi sono disposti ad esporre l'arco, entrambi i nodi
avviano una tasklet che si occuperà della monitorazione dell'arco e della
misurazione del costo ad esso associato. In questa tasklet viene realizzata subito una prima misurazione e in seguito
ogni 30 secondi si ripete. Nell'effettuare la misurazione si verifica anche il funzionamento stesso dell'arco e se non
funziona viene rimosso.

La misurazione del costo espresso come RTT avviene attraverso l'uso dell'oggetto INeighborhoodNetworkInterface, come indicato sotto.

Dopo che è stata fatta la prima misurazione, il modulo emette il segnale *arc_added* e l'arco va a far parte della
lista ufficiale.

Per quanto riguarda la memorizzazione del costo, dopo aver rilevato la prima misurazione l'algoritmo è il seguente:

```
 . costo_nuovo = <costo appena rilevato>
 . costo_memorizzato = costo_nuovo
 . costo_ufficiale = costo_memorizzato
```

Dopo ogni successiva misurazione l'algoritmo è il seguente:

```
 . costo_nuovo = <costo appena rilevato>
 . costo_delta = costo_nuovo - costo_memorizzato
 . se (costo_delta > 0) costo_delta = costo_delta / 10
 . se (costo_delta < 0) costo_delta = costo_delta / 3
 . costo_memorizzato = costo_memorizzato + costo_delta
 . se (costo_memorizzato < costo_ufficiale*0.5 OR costo_memorizzato > costo_ufficiale*2)
     . costo_ufficiale = costo_memorizzato
```

Questo algoritmo fa in modo che lievi variazioni non scatenino soventi aggiornamenti e traffico di rete. Inoltre tiene
conto della seguente osservazione: se ci sono misurazioni ravvicinate della latenza che differiscono, quella più bassa
è quella più vicina alla latenza puramente dovuta alla distanza dei due nodi, mentre quella più alta è stata probabilmente
maggiormente influenzata dal carico del nodo o dal bufferbloat. Nonostante questa osservazione l'algoritmo pian piano
si adegua se le misurazioni si mantengono su un valore elevato.

**INeighborhoodIPRouteManager**

Tramite l'oggetto che implementa questa interfaccia dovrà essere possibile apportare delle modifiche al sistema. In un
sistema Linux ad esempio, dotato del software `iproute`, si associano ai metodi dell'interfaccia questi comandi:

*   metodo `add_address`:  
    `ip address add 169.254.1.1 dev wlan0`
*   metodo `add_neighbor`:  
    `ip route add 169.254.22.33 dev wlan0 src 169.254.1.1`
*   metodo `remove_neighbor`:  
    `ip route del 169.254.22.33 dev wlan0 src 169.254.1.1`
*   metodo `remove_address`:  
    `ip address del 169.254.1.1/32 dev wlan0`

**INeighborhoodNetworkInterface**

Una istanza di tale interfaccia viene passata al modulo quando l'utilizzatore gli chiede di monitorare
una interfaccia di rete (metodo `start_monitor`). L'istanza di questo oggetto passato al modulo è legata
alla specifica interfaccia di rete.

Il modulo Neighborhood deve poter misurare il RTT tra l'interfaccia di rete `dev_a` del nodo *a* e
l'interfaccia di rete `dev_b` del nodo *b*. Il modulo non fa tale misurazione in autonomia. Se lo facesse
risulterebbe molto inaccurato, in quanto il modulo fa uso di tasklet, cioè thread cooperativi, che non
possono essere accurati nei tempi di risposta. Quindi il modulo delega questa misurazione all'oggetto che
implementa questa interfaccia.

L'utilizzatore del modulo Neighborhood misura il RTT di un certo arco nell'implementazione del metodo
`measure_rtt` dell'interfaccia INeighborhoodNetworkInterface.

Il modulo Neighborhood lo richiama per stabilire il costo (in latenza) di un arco, e passa come parametro
l'indirizzo di scheda del vicino. Il modulo Neighborhood in seguito verifica anche che presso il vicino
il modulo stesso sia ancora in vita eseguendo, con protocollo reliable, un metodo remoto che non fa alcuna
operazione: `nop`. Se riceve un errore rimuove l'arco.

Quando il nodo *a* ha un vicino *b* questi possono avere più di un arco che li unisce. Ma ogni arco ha una
interfaccia esclusiva su entrambi i nodi. Per esempio il nodo *a* può avere solo un arco che parte dalla sua
interfaccia *eth0* e arriva al nodo *b*. Un altro arco potrebbe ad esempio partire dall'interfaccia *eth1*
di *a* e arrivare a *b*. Allo stesso modo il nodo *b* deve avere più interfacce di rete per avere due
archi verso *a*.

Ogni interfaccia di rete di ogni nodo, inoltre, ha un indirizzo di scheda univoco. Di conseguenza, quando
il nodo *a* indica un indirizzo di scheda del suo vicino *b*, il nodo *a* indica precisamente un arco che
lo congiunge a *b* attraverso due specifiche interfacce di rete `dev_a` e `dev_b`.

Sapendo questo, ad esempio in Linux, il comando `ping -q -n -c 1 169.254.21.36` potrà essere usato per misurare la
latenza esattamente dell'arco che ci interessa, anche nel caso in cui per lo stesso vicino esistano due o più archi.

Il modulo non esclude altri meccanismi di misurazione del costo. Per dare il massimo supporto al suo utilizzatore,
nel metodo `measure_rtt` sono passate queste informazioni relative all'arco:

*   `string peer_addr` - l'indirizzo di scheda della interfaccia del vicino,
*   `string peer_mac` - il MAC della interfaccia del vicino,
*   `string my_dev` - il nome della interfaccia del nodo.
*   `string my_addr` - l'indirizzo di scheda della interfaccia del nodo.

## Indirizzo IPv4 di scheda

La comunicazione tra due nodi collegati da un arco deve poter avvenire in modo reliable. Infatti se la comunicazione
risulta impossibile si deve procedere alla rimozione dell'arco stesso. Per implementare la comunicazione reliable si
può far uso del protocollo TCP, ma per questo occorre poter associare ad ogni vertice dell'arco un indirizzo IP fisso.

Si assegna subito un indirizzo "locale" distinto ad ogni scheda di rete del nodo e lo si mantiene sempre, anche quando
il nodo fa l'ingresso in una nuova rete, o migra in un diverso g-nodo, ecc.

L'unico requisito da soddisfare è che tale indirizzo IPv4 sia univoco tra tutti i vicini. Lo scegliamo in modo
random nella classe di indirizzi [169.254.0.0/16](https://en.wikipedia.org/wiki/Link-local_address).

Il caso in cui due nodi vicini si scelgano lo stesso indirizzo è ignorabile.

Quando si aggiunge una interfaccia di rete da monitorare, il modulo genera un nuovo indirizzo IPv4 locale e lo
aggiunge ai suoi indirizzi su questa interfaccia di rete. Per farlo usa l'oggetto passatogli che implementa
l'interfaccia INeighborhoodIPRouteManager, precisamente con il metodo `add_address`.

Quando si smette di gestire l'interfaccia, il modulo rimuove anche l'indirizzo locale dalla interfaccia, con
il metodo `remove_address`.

Quando un nodo gestisce una interfaccia di rete, nel messaggio broadcast "ci sono io" comunica il suo
indirizzo scelto per quella scheda.

Quando viene realizzato un arco ciascuno dei due nodi:

*   Subito, senza attendere che l'arco venga esposto, mette l'indirizzo di scheda del vicino come link
    diretto (associato al suo giusto nic e avendo come source preferito il suo indirizzo di scheda) nelle
    tabelle di routing, con il metodo `add_neighbor`.
*   Dopo che l'arco è stato esposto, in una nuova tasklet, periodicamente crea una connessione reliable
    verso l'indirizzo di scheda del vicino e verifica che l'arco sia ancora funzionante altrimenti lo rimuove.

Alla rimozione di un arco il nodo:

*   Rimuove l'indirizzo di scheda del vicino dalle tabelle di routing, con il metodo `remove_neighbor`.
*   Se l'arco era esposto, abortisce la tasklet che lo monitorava.