# Modulo Neighborhood - Dettagli Tecnici

1.  [Requisiti](#Requisiti)
1.  [Deliverable](#Deliverable)
1.  [Rilevamento dei vicini, costituzione degli archi, misurazione dei costi](#Rilevamento_dei_vicini_costituzione_degli_archi_misurazione_dei_costi)
1.  [Produzione di uno stub per inviare un messaggio in broadcast](#Produzione_di_uno_stub_per_inviare_un_messaggio_in_broadcast)
1.  [Produzione di uno stub per inviare un messaggio UDP in unicast](#Produzione_di_uno_stub_per_inviare_un_messaggio_UDP_in_unicast)
1.  [Produzione di uno stub per inviare un messaggio reliable ad un vicino tramite un arco](#Produzione_di_uno_stub_per_inviare_un_messaggio_reliable_ad_un_vicino_tramite_un_arco)
1.  [Indirizzo IPv4 di scheda](#Indirizzo_IPv4_di_scheda)

## <a name="Requisiti"></a>Requisiti

L'utilizzatore del modulo Neighborhood per prima cosa inizializza il modulo richiamando il metodo statico
*init* di NeighborhoodManager. In tale metodo viene anche passata l'istanza di ITasklet per fornire
l'implementazione del sistema di tasklet.

Dopo istanzia il suo NeighborhoodManager passando al costruttore:

*   La callback per gestire gli IdentityAwareUnicastID (*get_identity_skeleton*).
*   La callback per gestire gli IdentityAwareBroadcastID (*get_identity_skeleton_set*).
*   Lo skeleton per gestire gli WholeNodeUnicastID e gli WholeNodeBroadcastID (*node_skeleton*).
*   Il numero massimo di archi (int *max_arcs*).
*   La stub factory (istanza di INeighborhoodStubFactory *stub_factory*).
*   Il manager di indirizzi (istanza di INeighborhoodIPRouteManager *ip_mgr*).

Dopo, per ogni interfaccia di rete che intende gestire, richiama sul NeighborhoodManager il metodo
*start_monitor(nic)*, dove *nic* è una INeighborhoodNetworkInterface.

In seguito l'utilizzatore del modulo può aggiungere altre interfacce di rete da gestire, oppure rimuovere
una interfaccia con il metodo *stop_monitor(dev)*, dove *dev* è il nome dell'interfaccia.

Esiste anche un metodo *stop_monitor_all()* per rimuovere tutte le interfacce chiamando su ognuna
lo stesso *stop_monitor*. Questo può essere chiamato dall'utilizzatore prima di dismettere il NeighborhoodManager.  
Il metodo potrebbe anche essere chiamato nel distruttore della classe NeighborhoodManager. Però le
operazioni delle varie chiamate *stop_monitor* causano l'emissione di segnali che possono essere importanti da
gestire, come `nic_address_unset`. Tali segnali non sarebbero ricevuti dall'utilizzatore se fossero emessi
durante l'esecuzione del distruttore della classe. Per questo si preferisce non implementare la chiamata
a *stop_monitor_all* nel distruttore della classe, ma piuttosto demandarla all'utilizzatore.

## <a name="Deliverable"></a>Deliverable

Il modulo segnala l'avvenuta assegnazione dell'indirizzo di scheda ad una interfaccia di rete gestita
attraverso il segnale `nic_address_set` di NeighborhoodManager.

In tale segnale viene riportato:

*   Il nome dell'interfaccia di rete. Una stringa `dev`. Es. "eth0".
*   L'indirizzo assegnato. Una stringa `address`. Es. "169.254.23.45".

* * *

Il modulo segnala la costituzione di un arco attraverso il segnale `arc_added` di NeighborhoodManager.

In tale segnale viene riportato:

*   L'arco costituito. Un INeighborhoodArc `arc`.

Il segnale significa anche che è stata aggiunta nelle tabelle (nel network namespace default) la rotta verso
quell'indirizzo di scheda che è riportato nell'istanza di INeighborhoodArc.

* * *

Il modulo segnala che sta per rimuovere un arco attraverso il segnale `arc_removing` di NeighborhoodManager.

In tale segnale viene riportato:

*   L'arco che sta per essere rimosso. Un INeighborhoodArc `arc`.
*   Un booleano `is_still_usable` che dice se l'arco è ancora utilizzabile per comunicare con il sistema vicino.

Il segnale comunica all'utilizzatore che il modulo sta per rimuovere dalle tabelle (nel network namespace default) la rotta verso
quell'indirizzo di scheda che è riportato nell'istanza di INeighborhoodArc.

Questa notifica avviene in due tipi di situazione. Nel primo caso è perché il modulo si è avveduto da solo
(oppure il suo utilizzatore glielo ha notificato) che l'arco è effettivamente non funzionante. Nel secondo
caso è perché l'utilizzatore ha richiesto al modulo di rimuovere l'arco che ancora si presume essere
funzionante. Il modulo notificando questo segnale `arc_removing` include questa informazione nel booleano
`is_still_usable`.

* * *

Il modulo segnala l'avvenuta rimozione di un arco attraverso il segnale `arc_removed` di NeighborhoodManager.

In tale segnale viene riportato:

*   L'arco rimosso. Un INeighborhoodArc `arc`.

Il segnale significa anche che è stata rimossa dalle tabelle (nel network namespace default) la rotta verso
quell'indirizzo di scheda che è riportato nell'istanza di INeighborhoodArc.

* * *

Il modulo segnala la variazione del costo di un arco attraverso il segnale `arc_changed` di NeighborhoodManager.

In tale segnale viene riportato:

*   L'arco. Un INeighborhoodArc `arc`.

* * *

Il modulo segnala l'avvenuta rimozione dell'indirizzo di scheda ad una interfaccia di rete che non si gestisce
più attraverso il segnale `nic_address_unset` di NeighborhoodManager.

In tale segnale viene riportato:

*   Il nome dell'interfaccia di rete. Una stringa `dev`. Es. "eth0".
*   L'indirizzo assegnato. Una stringa `address`. Es. "169.254.23.45".

* * *

Il modulo permette di ottenere l'elenco degli archi ora presenti con il metodo *current_arcs* di NeighborhoodManager.

* * *

Il modulo fornisce i metodi *get_dispatcher* e *get_dispatcher_set* di NeighborhoodManager per permettere al nodo
di gestire i messaggi unicast e broadcast ricevuti.

* * *

Il modulo fornisce i metodi *get_identity* e *get_node_arc* di NeighborhoodManager per permettere al nodo di
identificare il vicino (*identità* o *nodo*) che ha inviato un messaggio.

* * *

Il modulo permette di ottenere uno stub per inviare un messaggio reliable ad un vicino tramite un arco (o un *arco-identità*)
con il metodo *get_stub_whole_node_unicast* (o *get_stub_identity_aware_unicast*) di NeighborhoodManager.

* * *

Il modulo permette di ottenere uno stub per inviare un messaggio in broadcast con i metodi *get_stub_whole_node_broadcast*
(e *get_stub_identity_aware_broadcast*) di NeighborhoodManager.

* * *

Il modulo permette di forzare la rimozione di un arco con il metodo *remove_my_arc* di NeighborhoodManager.

In questo metodo l'utilizzatore del modulo (o anche il modulo stesso che ne fa uso internamente quando
rileva che l'arco non è più funzionante) può specificare che si possa o meno tentare una ulteriore comunicazione
sull'arco prima di rimuoverlo. Lo si specifica con l'argomento booleano `do_tell`.

## <a name="Rilevamento_dei_vicini_costituzione_degli_archi_misurazione_dei_costi"></a>Rilevamento dei vicini, costituzione degli archi, misurazione dei costi

Quando viene istanziato il NeighborhoodManager il modulo genera il suo NeighborhoodNodeID.

Quando viene chiamato il metodo *start_monitor* sulla istanza di NeighborhoodManager, il modulo inizia a gestire una
interfaccia di rete. Di essa riceve il nome e il MAC address. Per prima cosa verifica che questi valori siano univoci.
Cicla le interfacce che ha già in gestione; se una ha gli stessi valori per nome e MAC allora ignora questa richiesta
che è un duplicato; se una ha lo stesso MAC e nome diverso oppure lo stesso nome e MAC diverso allora il modulo va in errore fatale.

Il modulo, per ogni interfaccia che inizia a gestire, memorizza il suo nome e il suo MAC address e genera e memorizza un
indirizzo locale di scheda. Usa l'oggetto [INeighborhoodIPRouteManager](#INeighborhoodIPRouteManager)
per impostare l'indirizzo generato. Poi emette il segnale *nic_address_set*.

Poi avvia una tasklet nella quale invia un `broadcast_to_dev` (cioè un broadcast solo su quella interfaccia di rete) con
il messaggio "ci sono io" indicando il suo NeighborhoodNodeID, il MAC della interfaccia e il suo indirizzo locale di scheda.
Poi ripete lo stesso messaggio con bassa frequenza, ogni minuto.

I nodi che sono nel dominio broadcast vedono questo messaggio e possono fin da subito accordarsi per un arco. Quelli che hanno
già costituito un arco verso quel MAC address ignorano il nuovo messaggio.

Un nodo che vuole accordarsi per un arco con un vicino di cui ha notato la presenza invia un messaggio "facciamo un arco"
in `broadcast_to_dev` (solo sull'interfaccia dove ha ricevuto il messaggio "ci sono io") indicando sia i dati ricevuti
nel precedente messaggio "ci sono io" (NeighborhoodNodeID, MAC e linklocal del nodo vicino) sia i rispettivi dati
del proprio nodo. Dopo entrambi i nodi avranno un arco che ancora non è esposto dal modulo e il cui costo è ancora non misurato.

Dopo, entrambi i nodi usano l'oggetto [INeighborhoodIPRouteManager](#INeighborhoodIPRouteManager)
per impostare la rotta verso l'indirizzo di scheda del nuovo vicino. Quindi da subito è possibile realizzare connessioni
reliable (con protocollo TCP) tra i due nodi passanti per questo nuovo arco.

Il nodo che ha inviato il messaggio "facciamo un arco" usa ora una connessione TCP per chiamare sul nuovo vicino il metodo
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

La misurazione del costo espresso come RTT avviene attraverso l'uso dell'oggetto
[INeighborhoodNetworkInterface](#INeighborhoodNetworkInterface), come indicato sotto.

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

Questo algoritmo fa in modo che lievi variazioni non scatenino pesanti aggiornamenti e traffico di rete. Inoltre tiene
conto della seguente osservazione: se ci sono misurazioni ravvicinate della latenza che differiscono quella più bassa
è quella più vicina alla latenza puramente dovuta alla distanza dei due nodi, mentre quella più alta è stata probabilmente
maggiormente influenzata dal carico del nodo o dal bufferbloat. Nonostante questa osservazione l'algoritmo pian piano
si adegua se le misurazioni si mantengono su un valore elevato.

<a name="INeighborhoodIPRouteManager"></a>**INeighborhoodIPRouteManager**

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

<a name="INeighborhoodNetworkInterface"></a>**INeighborhoodNetworkInterface**

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

## <a name="Produzione_di_uno_stub_per_inviare_un_messaggio_in_broadcast"></a>Produzione di uno stub per inviare un messaggio in broadcast

Il modulo Neighborhood riceve dal suo utilizzatore (nel costruttore di NeighborhoodManager) un'istanza di un oggetto di
cui conosce l'interfaccia INeighborhoodStubFactory. Questa ha il metodo get_broadcast che restituisce un oggetto stub di
tipo radice (un IAddressManagerStub) che effettua chiamate broadcast.

Quando vuole trasmettere un messaggio in broadcast il modulo chiama sempre subito prima questo metodo per ottenere uno stub
nuovo. In questo metodo può indicare una istanza dell'interfaccia IAckCommunicator per ricevere dopo il timeout la lista
dei MAC address che hanno segnalato con un ACK la ricezione del messaggio.

Il metodo riceve questi parametri:

*   un ISourceID;
*   un IBroadcastID;
*   un elenco di interfacce di rete (Lista-di-string nics);
*   opzionalmente una istanza di IAckCommunicator.

Un IBroadcastID può avere diverse forme per identificare quali nodi diretti vicini (o quali loro *identità*) siano i
destinatari del messaggio. Individuiamo queste possibili classi che implementano IBroadcastID:

*   EveryWholeNodeBroadcastID. Indica che tutti i nodi che ricevono sono i destinatari del messaggio *di nodo*.  
    Questo è usato dal modulo Neighborhood per i messaggi "ci sono io".
*   WholeNodeBroadcastID. Contiene un set di NeighborhoodNodeID. Indica che solo quei nodi sono i destinatari del messaggio *di nodo*.
*   IdentityAwareBroadcastID. Contiene un set di NodeID. Indica che solo quelle *identità* sono i destinatari del messaggio *di identità*.

Quando il modulo neighborhood (NeighborhoodManager) vuole inviare un messaggio in broadcast:

*   *bcid* = una istanza di IBroadcastID a seconda dei destinatari da raggiungere.

*   *nics* = lista di nomi di NIC a seconda dei destinatari da raggiungere.

*   *comm* = null.

*   se desidera specificare un handler per gli archi che non hanno ricevuto il messaggio:
    *   *missing_handler* = istanza di una classe a sua scelta che implementa INeighborhoodMissingArcHandler.missing come
        vuole; potrebbe essere necessario passare al costruttore un riferimento allo stesso NeighborhoodManager per richiamare
        i suoi metodi, ad esempio ` new NeighborhoodRemoveMissing(this) ` ; oppure, se si ha una callback da richiamare la si
        passa al costruttore, ad esempio ` new NeighborhoodActOnMissing(missing_callback) ` .

    *   *lst_expected* = current_arcs_for_broadcast(nics); cioè memorizza in una lista gli archi che dovrebbero ricevere il messaggio.

    *   comm = new NeighborhoodAcknowledgementsCommunicator ( nics, lst_expected, mgr=this, missing_handler ) ; questa classe
        implementa IAckCommunicator.process_macs_list(Gee.List&lt;string&gt; responding_macs) così:

        *   lst_expected = intersezione(lst_expected, mgr.current_arcs_for_broadcast(nics)); cioè usa nuovamente il metodo
            NeighborhoodManager.current_arcs_for_broadcast; solo gli archi che esistevano prima e esistono ora vanno verificati.

        *   per ogni *missed* in lst_expected tale che missed.mac not in responding_macs:

            *   lancia una nuova tasklet in cui:

                *   missing_handler.missing(missed) ; cioè chiama il metodo 'missing' nell'istanza di
                    INeighborhoodMissingArcHandler, passando l'arco mancato.

*   stub = stub_factory.get_broadcast(bcid, nics, comm).

*   chiama il metodo che vuole sullo stub.

Se si è passata *comm*, l'istanza di IAckCommunicator opzionale, allora la chiamata del metodo remoto deve seguire
immediatamente la creazione dello stub, perché la prima chiamata a current_arcs_for_broadcast viene eseguita subito
e la seconda viene eseguita dopo il timeout che parte al momento della chiamata del metodo remoto.

* * *

Il modulo può aver bisogno internamente di comunicare con i suoi vicini e per questo di produrre uno stub. Oppure lo stub
gli può essere richiesto dall'esterno. In entrambi i casi questo algoritmo sopra delineato viene usato. La differenza sta
nella costruzione della istanza di IBroadcastID.

Uno stub è necessario al modulo Neighborhood quando deve trasmettere un messaggio in broadcast per rilevare nuovi archi o
nodi. In questa occasione quello che serve è un EveryWholeNodeBroadcastID. E non serve un INeighborhoodMissingArcHandler.

Quando uno stub è necessario ad un modulo *di nodo* (che non sia il Neighborhood) quel modulo (o comunque l'utilizzatore del
modulo Neighborhood) deve indicare quali nodi vanno raggiunti. Deve cioè fornire un set di INeighborhoodArc da cui il modulo
Neighborhood estrapola un set di NeighborhoodNodeID con cui compone un WholeNodeBroadcastID. Può inoltre fornire un
INeighborhoodMissingArcHandler. Il metodo pubblico di NeighborhoodManager a questo scopo è *get_stub_whole_node_broadcast*.

Quando uno stub è necessario ad un modulo *di identità* quel modulo (o comunque l'utilizzatore del modulo Neighborhood) deve
indicare quali *identità* vanno raggiunte. Deve cioè fornire un set di NodeID con cui il modulo Neighborhood compone un
IdentityAwareBroadcastID. Può inoltre fornire un INeighborhoodMissingArcHandler. Il metodo pubblico di NeighborhoodManager
a questo scopo è *get_stub_identity_aware_broadcast*.

## <a name="Produzione_di_uno_stub_per_inviare_un_messaggio_UDP_in_unicast"></a>Produzione di uno stub per inviare un messaggio UDP in unicast

L'interfaccia INeighborhoodStubFactory permette permette con il metodo get_unnicast al modulo di ottenere uno stub per inviare
un messaggio UDP (non reliable) ad un particolare vicino tramite una sua particolare interfaccia di rete, dati questi parametri:

*   un ISourceID;
*   un IUnicastID;
*   una interfaccia di rete (string);
*   un boolean per indicare se si vuole attendere la processazione.

Un IUnicastID prodotto per un messaggio da trasmettere in UDP ha solo una forma:

*   NoArcWholeNodeUnicastID. Contiene il NeighborhoodNodeID e il MAC address a cui il messaggio è indirizzato. Solo il nodo
    che ha quel NeighborhoodNodeID e solo quando riceve il messaggio attraverso quella interfaccia, si riconosce come
    destinatario del messaggio *di nodo*. Anche se l'arco non esiste ancora.  
    Questo è usato esclusivamente dal modulo Neighborhood, per i messaggi "facciamo un arco" e "rimuovi un arco".

Il modulo Neighborhood può aver bisogno internamente di comunicare con un suo vicino prima di aver realizzato un arco;
in questo caso produce questo tipo di stub con questo tipo di IUnicastID. Come istanza di ISourceID viene usata una
istanza della classe WholeNodeSourceID.

In seguito, quando vi è un arco tra due nodi vicini, il modulo userà solo comunicazioni con protocollo reliable per
inviare messaggi ad uno specifico vicino. Questa modalità non reliable non viene fornita all'esterno dal modulo.

## <a name="Produzione_di_uno_stub_per_inviare_un_messaggio_reliable_ad_un_vicino_tramite_un_arco"></a>Produzione di uno stub per inviare un messaggio reliable ad un vicino tramite un arco

L'interfaccia INeighborhoodStubFactory permette con il metodo get_tcp al modulo anche di ottenere uno stub per inviare
un messaggio reliable ad un particolare vicino tramite un particolare arco, dati questi parametri:

*   un ISourceID;
*   un IUnicastID;
*   l'indirizzo di scheda del vicino su quell'arco (vedi sotto);
*   un boolean per indicare se si vuole attendere la processazione.

Un IUnicastID prodotto per un messaggio da trasmettere in TCP può avere diverse forme per identificare quale nodo
diretto vicino (o quale sua *identità*) sia il destinatario del messaggio. Individuiamo queste possibili classi che
implementano IUnicastID:

*   WholeNodeUnicastID. Non contiene dati, comunque il messaggio è in TCP quindi sarà processato da un solo nodo e una sola volta.
*   IdentityAwareUnicastID. Contiene un NodeID.

Chiamando un metodo su questo stub viene inviato un messaggio con il protocollo TCP, quindi reliable. Il metodo attende
che il messaggio raggiunga il vicino, ma al momento della produzione dello stub si può specificare se si intende
attendere l'esecuzione del metodo nel nodo vicino oppure no.

* * *

Il modulo può aver bisogno internamente di comunicare con un suo vicino passando per un arco; in questo caso produce
questo tipo di stub. Oppure lo stub gli può essere richiesto dall'esterno.

## <a name="Indirizzo_IPv4_di_scheda"></a>Indirizzo IPv4 di scheda

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
l'interfaccia INeighborhoodIPRouteManager, precisamente con il metodo 'add_address'.

Quando si smette di gestire l'interfaccia, il modulo rimuove anche l'indirizzo locale dalla interfaccia, con
il metodo 'remove_address'.

Quando un nodo gestisce una interfaccia di rete, nel messaggio broadcast_to_nic "ci sono io" comunica il suo
indirizzo scelto per quella scheda.

Quando viene realizzato un arco ciascuno dei due nodi:

*   mette l'indirizzo di scheda del vicino come link diretto (associato al suo giusto nic e avendo come source
    preferito il suo indirizzo di scheda) nelle tabelle di routing, con il metodo 'add_neighbor';
*   in una nuova tasklet, periodicamente crea un TCPClient verso l'indirizzo di scheda del vicino e verifica che
    l'arco è ancora funzionante altrimenti lo rimuove.

Alla rimozione di un arco il nodo:

*   rimuove l'indirizzo di scheda del vicino dalle tabelle di routing, con il metodo 'remove_neighbor';
*   rimuove la tasklet che lo monitorava.

Quando si vuole inviare un messaggio ad un vicino attraverso un determinato arco si può usare il TCP verso
l'indirizzo di scheda del vicino memorizzato su questo arco, quindi si ha un collegamento reliable.

