# Proof of concept - Casi d'uso - Pagina 7

[Pagina precedente](UseCases6.md)

## <a name="step4"></a>Ingresso di 𝛼 in G𝛾

Abbiamo già visto (in [elaborazione ETP](UseCases3.md#Elaborazione_etp)) cosa avviene nel sistema *𝛿* quando il
nodo *𝛼* entra in *G<sub>𝛾</sub>*. Vediamo cosa avviene negli altri sistemi.

### Nel sistema *𝛼*

La prima identità è *𝛼<sub>0</sub>* con indirizzo Netsukuku 3·0·1·0 in *G<sub>𝛼</sub>*.

```
Mio indirizzo 3·0·1·0.
     globale
      10.0.0.26
     anonimizzante
      10.0.0.90
     interno al mio g-nodo di livello 3
      10.0.0.58
     interno al mio g-nodo di livello 2
      10.0.0.50
     interno al mio g-nodo di livello 1
      10.0.0.40

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 1·
     globale
      10.0.0.8/29
     anonimizzante
      10.0.0.72/29
 2·
     globale
      10.0.0.16/29
     anonimizzante
      10.0.0.80/29
 3·1·
     globale
      10.0.0.28/30
     anonimizzante
      10.0.0.92/30
     interno al mio g-nodo di livello 3
      10.0.0.60/30
 3·0·0·
     globale
      10.0.0.24/31
     anonimizzante
      10.0.0.88/31
     interno al mio g-nodo di livello 3
      10.0.0.56/31
     interno al mio g-nodo di livello 2
      10.0.0.48/31
 3·0·1·1
     globale
      10.0.0.27/32
     anonimizzante
      10.0.0.91/32
     interno al mio g-nodo di livello 3
      10.0.0.59/32
     interno al mio g-nodo di livello 2
      10.0.0.51/32
     interno al mio g-nodo di livello 1
      10.0.0.41/32
```

**sistema 𝛼**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip link set dev eth1 address 00:16:3E:FD:E2:AA
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
ip link set dev eth1 up
ip address add 169.254.69.30 dev eth1
ip address add 10.0.0.26 dev eth1
ip address add 10.0.0.90 dev eth1
ip address add 10.0.0.58 dev eth1
ip address add 10.0.0.50 dev eth1
ip address add 10.0.0.40 dev eth1
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.28/30 table ntk
ip route add unreachable 10.0.0.92/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.24/31 table ntk
ip route add unreachable 10.0.0.88/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.27/32 table ntk
ip route add unreachable 10.0.0.91/32 table ntk
ip route add unreachable 10.0.0.59/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.26
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.28/30 table ntk
ip route change unreachable 10.0.0.92/30 table ntk
ip route change unreachable 10.0.0.60/30 table ntk
ip route change unreachable 10.0.0.24/31 table ntk
ip route change unreachable 10.0.0.88/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.27/32 table ntk
ip route change unreachable 10.0.0.91/32 table ntk
ip route change unreachable 10.0.0.59/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
```

Ora si rileva il sistema *𝛽* e si entra in *G<sub>𝛾</sub>*. Viene creata l'identità
*𝛼<sub>1</sub>* partendo da *𝛼<sub>0</sub>*. L'identità *𝛼<sub>0</sub>* prende posto in un
nuovo network namespace "entr04".

**sistema 𝛼**
```
ip route add 169.254.96.141 dev eth1 src 169.254.69.30
ip netns add entr04
ip netns exec entr04 sysctl net.ipv4.ip_forward=1
ip netns exec entr04 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr04_eth1 link eth1 type macvlan
ip link set dev entr04_eth1 netns entr04
ip netns exec entr04 ip link set dev entr04_eth1 address 00:16:3E:78:18:0B
ip netns exec entr04 sysctl net.ipv4.conf.entr04_eth1.rp_filter=0
ip netns exec entr04 sysctl net.ipv4.conf.entr04_eth1.arp_ignore=1
ip netns exec entr04 sysctl net.ipv4.conf.entr04_eth1.arp_announce=2
ip netns exec entr04 ip link set dev entr04_eth1 up
ip netns exec entr04 ip address add 169.254.202.128 dev entr04_eth1
```

Il modulo Identities del sistema *𝛼*, dal dialogo con il vicino, desume che vanno creati/modificati questi
archi-identità:

*   *𝛼<sub>0</sub>-𝛽<sub>1</sub>*. Si ricordi che ora *𝛽<sub>1</sub>* è l'identità principale di *𝛽*.  
    Per questo arco-identità abbiamo che la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝛼<sub>0</sub>*
    la rotta verso il vicino.  
    I nodi *𝛼<sub>0</sub>* e *𝛽<sub>1</sub>* non fanno parte della stessa rete, quindi non è prevista una tabella
    `ntk_from_xxx`.
*   *𝛼<sub>1</sub>-𝛽<sub>1</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝛼<sub>1</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝛼<sub>1</sub>* e *𝛽<sub>1</sub>* faranno parte della stessa rete, ma solo dopo che il nodo *𝛼<sub>1</sub>*
    avrà costruito la nuova istanza di QspnManager; per fare questo il sistema *𝛼* dovrà costruire un QspnArc
    sull'arco-identità *𝛼<sub>1</sub>-𝛽<sub>1</sub>*. Questo nuovo QspnArc inizialmente non comporta operazioni sulle
    tabelle di routing, fino a quando non si riceve il primo ETP attraverso questo arco e così si scopre l'indirizzo
    Netsukuku di questo vicino. Solo a quel punto, ad esempio, nel network namespace gestito da *𝛼<sub>1</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:EC:A3:E1` e la relativa regola.


Il modulo Identities fa queste operazioni:

**sistema 𝛼**
```
ip netns exec entr04 ip route add 169.254.96.141 dev entr04_eth1 src 169.254.202.128
```

Il programma *qspnclient* fa queste operazioni preliminari:

**sistema 𝛼**
```
ip netns exec entr04 ip rule add table ntk
```

Il programma *qspnclient* non deve fare alcuna operazione su rotte verso destinazioni interne al g-nodo
che ha migrato poiché si tratta della migrazione di un singolo nodo.

Il programma *qspnclient* sposta tutte le rotte dal network namespace vecchio al network namespace nuovo.

Avendo però preso per la vecchia identità un indirizzo Netsukuku con posizione *virtuale* nel
livello 0, dove prima aveva posizione *reale*, abbiamo che esiste una ulteriore possibile
destinazione da aggiungere nel nuovo network namespace. La 3·0·1·0.

```
 3·0·1·0
     globale
      10.0.0.26
     anonimizzante
      10.0.0.90
     interno al mio g-nodo di livello 3
      10.0.0.58
     interno al mio g-nodo di livello 2
      10.0.0.50
     interno al mio g-nodo di livello 1
      10.0.0.40
```

**sistema 𝛼**
```
ip netns exec entr04 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.28/30 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.92/30 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.60/30 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.24/31 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.88/31 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.56/31 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.27/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.91/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.59/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.51/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.41/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.26/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.90/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.58/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.50/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.40/32 table ntk

ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.8/29 table ntk
ip route del 10.0.0.72/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.28/30 table ntk
ip route del 10.0.0.92/30 table ntk
ip route del 10.0.0.60/30 table ntk
ip route del 10.0.0.24/31 table ntk
ip route del 10.0.0.88/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.27/32 table ntk
ip route del 10.0.0.91/32 table ntk
ip route del 10.0.0.59/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
```

Il programma *qspnclient* rimuove l'indirizzo globale e tutti gli indirizzi interni ai propri g-nodi
dal network namespace vecchio (che era il default).

**sistema 𝛼**
```
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.26
ip address del 10.0.0.26/32 dev eth1
ip address del 10.0.0.90/32 dev eth1
ip address del 10.0.0.58/32 dev eth1
ip address del 10.0.0.50/32 dev eth1
ip address del 10.0.0.40/32 dev eth1
```

Il programma *qspnclient* aggiorna le rotte nel nuovo network namespace sulla base dei migliori percorsi
noti alla vecchia identità *𝛼<sub>0</sub>*.

**sistema 𝛼**
```
ip netns exec entr04 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.28/30 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.92/30 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.60/30 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.24/31 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.88/31 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.56/31 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.27/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.91/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.59/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.51/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.41/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.26/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.90/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.58/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.50/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.40/32 table ntk
```

Poi il sistema *𝛼* per la nuova identità *𝛼<sub>1</sub>* istanzia un QspnManager con il
costruttore `enter_net` passandogli un QspnArc per l'arco-identità *𝛼<sub>1</sub>-𝛽<sub>1</sub>*.
L'indirizzo Netsukuku del nodo *𝛼<sub>1</sub>* sarà 2·2·1·0.

Siccome la nuova identità *𝛼<sub>1</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku. In questo caso, poiché l'indirizzo è *virtuale* al livello 2, gli indirizzi
interni al livello 0 e 1.

Inoltre il programma *qspnclient* deve assicurarsi che nel network namespace della nuova identità *𝛼<sub>1</sub>*
siano presenti le regole per la consultazione della tabella `ntk` (non ci sono tabelle `ntk_from_xxx` associate agli
archi-identità di *𝛼<sub>1</sub>*). E non altre regole.

Deve poi aggiungere in questa tabella (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝛼<sub>1</sub>*, inizialmente in stato "unreachable".

```
Mio indirizzo 2·2·1·0.
     globale
      N/A
     anonimizzante
      N/A
     interno al mio g-nodo di livello 3
      N/A
     interno al mio g-nodo di livello 2
      10.0.0.50
     interno al mio g-nodo di livello 1
      10.0.0.40

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 1·
     globale
      10.0.0.8/29
     anonimizzante
      10.0.0.72/29
 3·
     globale
      10.0.0.24/29
     anonimizzante
      10.0.0.88/29
 2·0·
     globale
      10.0.0.16/30
     anonimizzante
      10.0.0.80/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 2·1·
     globale
      10.0.0.20/30
     anonimizzante
      10.0.0.84/30
     interno al mio g-nodo di livello 3
      10.0.0.60/30
 2·2·0·
     globale
      N/A
     anonimizzante
      N/A
     interno al mio g-nodo di livello 3
      N/A
     interno al mio g-nodo di livello 2
      10.0.0.48/31
 2·2·1·1
     globale
      N/A
     anonimizzante
      N/A
     interno al mio g-nodo di livello 3
      N/A
     interno al mio g-nodo di livello 2
      10.0.0.51/32
     interno al mio g-nodo di livello 1
      10.0.0.41/32
```

**sistema 𝛼**
```
ip address add 10.0.0.50 dev eth1
ip address add 10.0.0.40 dev eth1
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.16/30 table ntk
ip route add unreachable 10.0.0.80/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.20/30 table ntk
ip route add unreachable 10.0.0.84/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
```

Ora il QspnManager di *𝛼<sub>1</sub>* riceve un ETP dal vicino *𝛽<sub>1</sub>*. A questo punto
conosce l'indirizzo Netsukuku del peer sull'arco *𝛼<sub>1</sub>-𝛽<sub>1</sub>*. Per questo il
programma *qspnclient* può creare la tabella `ntk_from_00:16:3E:EC:A3:E1` e popolarla con tutte le
destinazioni adatte al suo indirizzo. Poi, ma solo dopo che la tabella sarà stata popolata
e aggiornata, aggiungerà la regola di guardare questa tabella per i pacchetti IP da inoltrare
ricevuti su questo arco.

**sistema 𝛼**
```
(echo; echo "250 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

Inoltre il QspnManager di *𝛼<sub>1</sub>* sulla base dell'ETP aggiorna la sua mappa.
Attraverso l'arco *𝛼<sub>1</sub>-𝛽<sub>1</sub>* scopre di poter raggiungere la destinazione
(2, 1), ossia il g-nodo 2·1·.

Poi il QspnManager di *𝛼<sub>1</sub>* notifica il segnale `bootstrap_complete`. Per questo
il programma *qspnclient* aggiorna tutte le rotte sulla base dei migliori percorsi noti.

**sistema 𝛼**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.84/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.60/30 table ntk via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

Alla fine di questo aggiornamento delle tabelle, siccome era rimasta in attesa la regola di guardare
la nuova tabella, il programma *qspnclient* la aggiunge.

**sistema 𝛼**
```
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
```

Ora si verifica l'altro evento: il cambio di indirizzo. Il programma *qspnclient* nel sistema *𝛼* viene notificato
che il suo indirizzo Netsukuku in *G<sub>𝛾</sub>*, che temporaneamente era il *virtuale* 2·2·1·0, diventa il
*reale* (2·0·1·0). Di conseguenza il programma comunica al QspnManager di *𝛼<sub>1</sub>* che ora il suo indirizzo Netsukuku è 2·0·1·0.

Siccome la nuova identità *𝛼<sub>1</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku.  
Deve però tenere presente che alcuni di essi possono essere già stati assegnati dal
precedente detentore del network namespace default o dallo stesso nella fase in cui aveva indirizzo virtuale.
In questo caso gli indirizzi interni al livello 0 e 1.

Deve poi aggiungere in queste tabelle (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝛼<sub>1</sub>*, inizialmente in stato "unreachable".  
Deve però tenere presente che alcune di esse possono essere già state aggiunte dal
precedente detentore del network namespace default o dallo stesso nella fase in cui aveva indirizzo virtuale.
In questo caso sono state già aggiunte quelle di livello maggiore o uguale a 2 e quelle di livello inferiore
ma solo con gli indirizzi interni al g-nodo di livello minore o uguale a 2.  
Deve inoltre considerare che prima la sua attuale posizione *reale* al livello 2 (2·0·) poteva essere
stata aggiunta come destinazione nelle tabelle (se le posizioni superiori erano tutte *reali*). In questo
caso essa va rimossa.

Infine, avendo aggiornato nell'identità principale il suo proprio indirizzo che ora è *reale*, deve fare
un aggiornamento di tutte le sue rotte poiché in alcune di esse il *src* poteva essere mancante.

```
Mio indirizzo 2·0·1·0.
     globale
      10.0.0.18
     anonimizzante
      10.0.0.82
     interno al mio g-nodo di livello 3
      10.0.0.58
     interno al mio g-nodo di livello 2
      10.0.0.50
     interno al mio g-nodo di livello 1
      10.0.0.40

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 1·
     globale
      10.0.0.8/29
     anonimizzante
      10.0.0.72/29
 3·
     globale
      10.0.0.24/29
     anonimizzante
      10.0.0.88/29
 2·0·  era valida prima ma non lo è più.
     globale
      10.0.0.16/30
     anonimizzante
      10.0.0.80/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 2·1·
     globale
      10.0.0.20/30
     anonimizzante
      10.0.0.84/30
     interno al mio g-nodo di livello 3
      10.0.0.60/30
 2·0·0·
     globale
      10.0.0.16/31
     anonimizzante
      10.0.0.80/31
     interno al mio g-nodo di livello 3
      10.0.0.56/31
     interno al mio g-nodo di livello 2
      10.0.0.48/31
 2·0·1·1
     globale
      10.0.0.19/32
     anonimizzante
      10.0.0.83/32
     interno al mio g-nodo di livello 3
      10.0.0.59/32
     interno al mio g-nodo di livello 2
      10.0.0.51/32
     interno al mio g-nodo di livello 1
      10.0.0.41/32
```

**sistema 𝛼**
```
ip address add 10.0.0.18 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.18
ip address add 10.0.0.82 dev eth1
ip address add 10.0.0.58 dev eth1
```

**sistema 𝛼**
```
ip route del 10.0.0.16/30 table ntk
ip route del 10.0.0.80/30 table ntk
ip route del 10.0.0.56/30 table ntk

ip route del 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
```

**sistema 𝛼**
```
ip route add unreachable 10.0.0.16/31 table ntk
ip route add unreachable 10.0.0.80/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.19/32 table ntk
ip route add unreachable 10.0.0.83/32 table ntk
ip route add unreachable 10.0.0.59/32 table ntk

ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EC:A3:E1
```

**sistema 𝛼**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.84/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.60/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.58
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.19/32 table ntk
ip route change unreachable 10.0.0.83/32 table ntk
ip route change unreachable 10.0.0.59/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

Infine, dopo un po' di tempo, l'identità *𝛼<sub>0</sub>* viene dismessa. Il programma *qspnclient*
esegue queste operazioni tramite il modulo Identities:

**sistema 𝛼**
```
ip netns exec entr04 ip route flush table main
ip netns exec entr04 ip link delete entr04_eth1 type macvlan
ip netns del entr04
```

### Nel sistema *𝛽*

Il modulo Neighborhood rileva il vicino sistema *𝛼*.

**sistema 𝛽**
```
ip route add 169.254.69.30 dev eth1 src 169.254.96.141
```

Il modulo Identities fa questa operazione quando si crea una nuova identità in *𝛼* (per fare ingresso
in *G<sub>𝛾</sub>*) mentre si sposta la vecchia identità in un nuovo network namespace.

**sistema 𝛽**
```
ip route add 169.254.202.128 dev eth1 src 169.254.96.141
```

Il programma *qspnclient* prima di informare il modulo QSPN che ha un nuovo arco-identità con un nodo della sua rete,
crea una nuova tabella e la popola con le destinazioni (per ora unreachable) adeguate al suo indirizzo; ma solo
dopo che avrà conosciuto l'indirizzo del peer su quell'arco-identità e di conseguenza avrà aggiornato tutte le
rotte in questa tabella, allora aggiungerà la relativa regola.

**sistema 𝛽**
```
(echo; echo "249 ntk_from_00:16:3E:FD:E2:AA # xxx_table_ntk_from_00:16:3E:FD:E2:AA_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA
```

Il sistema *𝛽* riceve un ETP da *𝛼*. Con esso scopre di avere un percorso verso il g-nodo 2·0·.
Aggiorna quindi le rotte delle sue tabelle.

**sistema 𝛽**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.69.30 dev eth1 src 10.0.0.23
ip route change 10.0.0.80/30 table ntk via 169.254.69.30 dev eth1 src 10.0.0.23
ip route change 10.0.0.56/30 table ntk via 169.254.69.30 dev eth1 src 10.0.0.63
ip route change 10.0.0.20/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.84/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.60/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.63
ip route change 10.0.0.48/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.51
ip route change 10.0.0.22/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.86/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.62/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.63
ip route change 10.0.0.50/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.41
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
```

Tramite l'ETP scopre inoltre l'indirizzo Netsukuku del peer attraverso questo arco. Per questo può aggiornare
anche le rotte nella tabella `ntk_from_xxx` relativa a questo arco. Poi, come detto sopra, aggiunge
la regola per guardare quella tabella

Dopo aver elaborato l'ETP lo propaga al sistema *𝛾*.

**sistema 𝛽**
```
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:FD:E2:AA
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change 10.0.0.22/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.86/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.62/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249
ip rule add fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
```

Il modulo Identities fa questa operazione quando si dismette la vecchia identità in *𝛼*.

**sistema 𝛽**
```
ip route del 169.254.202.128 dev eth1 src 169.254.96.141
```

### Nel sistema *𝛾*

Il sistema *𝛾* riceve un ETP da *𝛽* con il quale scopre di avere un percorso verso il g-nodo 2·0·. Dopo averlo
elaborato lo propaga al sistema *𝛿*.

**sistema 𝛾**
```
ssh gamma sudo ip route change unreachable 10.0.0.0/29 table ntk
ssh gamma sudo ip route change unreachable 10.0.0.64/29 table ntk
ssh gamma sudo ip route change unreachable 10.0.0.8/29 table ntk
ssh gamma sudo ip route change unreachable 10.0.0.72/29 table ntk
ssh gamma sudo ip route change unreachable 10.0.0.24/29 table ntk
ssh gamma sudo ip route change unreachable 10.0.0.88/29 table ntk
ssh gamma sudo ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ssh gamma sudo ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ssh gamma sudo ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ssh gamma sudo ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ssh gamma sudo ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ssh gamma sudo ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.62
ssh gamma sudo ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.50
ssh gamma sudo ip route change 10.0.0.23/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ssh gamma sudo ip route change 10.0.0.87/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ssh gamma sudo ip route change 10.0.0.63/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ssh gamma sudo ip route change 10.0.0.51/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ssh gamma sudo ip route change 10.0.0.41/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.40
ssh gamma sudo ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ssh gamma sudo ip route change 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ssh gamma sudo ip route change 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ssh gamma sudo ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ssh gamma sudo ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ssh gamma sudo ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ssh gamma sudo ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ssh gamma sudo ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
ssh gamma sudo ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ssh gamma sudo ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ssh gamma sudo ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ssh gamma sudo ip route change 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ssh gamma sudo ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ssh gamma sudo ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

### Nel sistema *𝜇*

Il sistema *𝜇* riceve un ETP da *𝛾* con il quale scopre di avere un percorso verso il g-nodo 2·0·.

**sistema 𝜇**
```
TODO
```

[Pagina seguente](UseCases8.md)
