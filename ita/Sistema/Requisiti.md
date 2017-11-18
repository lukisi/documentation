# Requisiti del Sistema Operativo

1.  [Requisiti](#Requisiti)
    1.  [Return Path](#Requisiti_return_path)
    1.  [Forwarding](#Requisiti_forwarding)
    1.  [Risposte alle richieste ARP](#Requisiti_risposte_arp)
    1.  [Incompatibilità con Zeroconf?](#Requisiti_zeroconf)
1.  [Linux](#Linux)
    1.  [Return Path](#Linux_return_path)
    1.  [Forwarding](#Linux_forwarding)
    1.  [Risposte alle richieste ARP](#Linux_risposte_arp)

## <a name="Requisiti"></a>Requisiti

### <a name="Requisiti_return_path"></a>Return Path

Il processo *ntkd* gestisce un certo numero di interfacce di rete, come l'utente lo configura. Per tali interfacce
il processo deve essere abilitato a vedere tutti i pacchetti che vi transitano, in particolare quelli IPv4 UDP con
destinazione Broadcast (255.255.255.255) qualunque sia l'indirizzo di provenienza.

Molti sistemi abilitano un filtro di sicurezza chiamato Return-Path. La logica dietro questo filtro consiste nel
ritenere che se un pacchetto in uscita con una certa destinazione deve passare per una certa interfaccia di rete, allora
un pacchetto in ingresso con quella provenienza deve essere ricevuto dalla stessa interfaccia.

Ad esempio, sia il nodo *a* con le interfacce *eth0* e *eth1*. Supponiamo che per le destinazioni in 192.168.1.0/24 il
gateway da usare è 192.168.1.1 tramite *eth0*. Invece le destinazioni 192.168.2.0/24 sono direttamente accedibili tramite
l'interfaccia *eth1*. Allora si presume che un pacchetto con provenienza 192.168.2.5 raggiunga il nostro nodo solo
passando per la nostra interfaccia *eth1*. Basandosi su questa idea, per motivi di sicurezza, i sistemi filtrano eventuali
pacchetti con provenienza 192.168.2.5 che sarebbero rilevati attraverso la nostra interfaccia *eth0*. Cioè, tali pacchetti,
anche qualora transitassero nel segmento di rete su cui è collegata *eth0*, non raggiungono i processi del nodo *a* che
stanno in ascolto sulla interfaccia *eth0*.

Questa idea non è valida in una rete Netsukuku, poiché ogni nodo può scegliere diversi percorsi per una destinazione, e
possono esserci percorsi che al ritorno sono diversi da quelli scelti all'andata. Quindi un requisito da soddisfare è che
tale filtro **non** sia attivato sulle interfacce di rete gestite da *ntkd*.

### <a name="Requisiti_forwarding"></a>Forwarding

Il sistema operativo deve essere configurato per abilitare l'inoltro (forwarding) di pacchetti, come minimo per tutti i
pacchetti i cui indirizzi IPv4 di provenienza e destinazione rientrano nel range di indirizzi riservati alla rete Netsukuku
di cui si è parte. Tale range di norma è la classe 10.0.0.0/8.

Questo requisito è vitale per un nodo che ha (o può avere) più di un vicino e usa la versione standard del demone *ntkd*.
Infatti tale demone annuncia ai suoi vicini tutti i percorsi che conosce, quindi se il nodo ha più di un vicino è possibile
che uno dei suoi vicini lo scelga come gateway per alcune destinazioni.

### <a name="Requisiti_risposte_arp"></a>Risposte alle richieste ARP

In alcune situazioni, le regole di routing per i pacchetti da inoltrare si basano sull'interfaccia di rete da cui il
pacchetto viene ricevuto.

Può esserci il caso di un nodo *a* che ha due interfacce di rete *NIC(a,1)* e *NIC(a,2)* entrambe collegate allo stesso
segmento di rete. Ad ognuna il demone *ntkd* assegna un distinto indirizzo IP link-local, ad esempio *IP(a,1)* e
*IP(a,2)*. Sia un nodo *b* collegato con la sua interfaccia di rete sullo stesso segmento di rete. Se il nodo *b* inoltra
un pacchetto attraverso il gateway *a* usando l'indirizzo *IP(a,1)* vogliamo che il routing segua un certo percorso. Se
invece il nodo *b* usa l'indirizzo *IP(a,2)* vogliamo che il routing segua un diverso percorso.

Perché questo funzioni è necessario che quando il nodo *b* trasmette un pacchetto IP al gateway *a* usando l'indirizzo
*IP(a,1)*, lo incapsuli in un frame Ethernet che riporta l'indirizzo MAC di *NIC(a,1)*. Allo stesso modo per l'indirizzo
*IP(a,2)* e l'indirizzo MAC di *NIC(a,2)*.

In conclusione, il sistema operativo deve essere configurato per rispondere alle richieste
[ARP](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) **esclusivamente** con l'indirizzo MAC della sua
interfaccia a cui è stato associato l'indirizzo IP richiesto.

### <a name="Requisiti_zeroconf"></a>Incompatibilità con Zeroconf?

[Zeroconf](https://en.wikipedia.org/wiki/Zero-configuration_networking) è un insieme di tecnologie per la configurazione
automatica dei nodi di una rete locale con il protocollo TCP/IP. Queste tecnologie servono i seguenti scopi:

*   Assegnare ad un nodo (computer) un indirizzo IP univoco a livello della rete locale nel range
    [link-local](https://en.wikipedia.org/wiki/Link-local_address), laddove non sia stato assegnato un qualsiasi
    indirizzo IP manualmente o tramite DHCP.
*   Assegnare una rotta su ogni interfaccia di rete che (con metrica a bassa priorità) indichi qualsiasi indirizzo
    nel range link-local. Cioè una operazione come `ip route add 169.254.0.0/16 dev eth0 metric 1000 scope link` su
    tutte le interfacce di rete.
*   Permettere la risoluzione di nomi di host in indirizzi per i nodi della rete locale, i quali usano nomi con il
    top-level domain [local](https://en.wikipedia.org/wiki/.local).
*   Altro.

Molti sistemi hanno una propria implementazione *zeroconf* abilitata di default. Per i sistemi Linux di norma questi
servizi sono garantiti dai pacchetti *avahi-autoipd* e il modulo *mDNS* di *NSS*.

Per la maggior parte queste operazioni non vanno in conflitto con il lavoro di *ntkd*. Tranne, in un certo senso,
l'assegnazione di una rotta verso 169.254.0.0/16. Il demone *ntkd* infatti assegna un indirizzo link-local ad ogni
interfaccia di rete e imposta specifiche rotte per ogni vicino che rileva. Quando un vicino non è più direttamente
collegato la corrispondente rotta viene rimossa. Quindi la presenza o meno della rotta generica potrebbe portare a
comportamenti differenti se il demone cercasse di comunicare. In teoria queste comunicazioni (con un vicino che è stato
"rimosso") non dovrebbero mai verificarsi.

[Alcuni](https://lists.freedesktop.org/archives/avahi/2006-September/000878.html) sostengono che una tale impostazione
messa per default su ogni sistema (Linux) sia sicura e mai problematica. Per il momento non so dire con certezza se
alcuni scenari potrebbero influire negativamente sulle comunicazioni del demone *ntkd* con i nodi diretti vicini.

## <a name="Linux"></a>Linux

Nei sistemi Linux, molte delle impostazioni che riguardano i servizi di base del sistema si regolano con il tool
*sysctl*. Le impostazioni che vengono modificate con questo tool permangono valide fino al riavvio del sistema. In
fase di boot il sistema regola alcune impostazioni sulla base di alcuni file di configurazione che si trovano in /etc/sysctl.d/.

### <a name="Linux_return_path"></a>Return Path

I sistemi Linux hanno alcune impostazioni che dicono se il sistema deve attivare il filtro Return-Path. Data una
interfaccia di rete **NICX** il filtro risulta attivo se almeno una di queste due impostazioni:

*   `net.ipv4.conf.all.rp_filter`
*   `net.ipv4.conf.NICX.rp_filter`

risulta uguale a 1. Risulta inattivo se entrambe sono uguali a 0.

Vi è una terza impostazione, `net.ipv4.conf.default.rp_filter`, la quale viene presa in esame quando il sistema crea
una interfaccia di rete (di solito in fase di boot) per valorizzare inizialmente l'impostazione specifica di quella interfaccia.

### <a name="Linux_forwarding"></a>Forwarding

I sistemi Linux hanno una impostazione che dice se il forward dei pacchetti va abilitato; il suo nome è
`net.ipv4.ip_forward`. Di default questo è messo a 0 (disabilitato) nei sistemi che si usano come normali computer,
cioè che non sono dei router.

Per il corretto funzionamento di un nodo nella rete Netsukuku questa impostazione va messa a 1.

### <a name="Linux_risposte_arp"></a>Risposte alle richieste ARP

I sistemi Linux hanno alcune impostazioni per ogni interfaccia di rete che influenzano il comportamento del sistema
quando riceve una richiesta ARP attraverso quella interfaccia.

In particolare, le impostazioni `net.ipv4.conf.NICX.arp_announce=0` e `net.ipv4.conf.NICX.arp_ignore=0` nella
configurazione di default fanno si che il sistema risponda su ogni interfaccia anche per indirizzi IP locali (che il
sistema ha assegnati a sé) ma associati ad un'altra interfaccia.

Per il corretto funzionamento di un nodo nella rete Netsukuku, data ogni interfaccia **NICX** gestita dal demone
*ntkd*, occorre impostare:

```
net.ipv4.conf.NICX.arp_ignore=1
net.ipv4.conf.NICX.arp_announce=2
```

Vi sono anche le impostazioni `net.ipv4.conf.default.arp_*` e `net.ipv4.conf.all.arp_*`. Quelle *default* sono prese in
esame quando il sistema crea una interfaccia di rete (di solito in fase di boot) per valorizzare inizialmente le
impostazioni specifiche di quella interfaccia. Quelle *all* sono valutate al momento del bisogno dal sistema insieme
a quelle specifiche di una interfaccia: il sistema usa il valore massimo.

Siccome da queste impostazioni può dipendere il corretto funzionamento di diversi scenari di rete, si consiglia di non
modificarle. Si preferisce modificare i valori sulle singole interfacce di rete che il demone *ntkd* deve gestire.

