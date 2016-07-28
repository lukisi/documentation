# Proof of concept - Casi d'uso - Parte 4

## <a name="Altri_sistemi"></a>Negli altri sistemi

Prendiamo in esame quello che avviene negli altri sistemi.

*   Sistema *𝜇*. Prime operazioni e ingresso in *G<sub>𝛿</sub>*. [Dettagli](#step1).
*   Sistemi *𝛽* e *𝛾*. Parallelamente essi formano un g-nodo di livello 1 in *G<sub>𝛾</sub>*. [Dettagli](UseCases5.md#step2).
*   Ingresso di *𝛿* e *𝜇* (come g-nodo *𝜒'*) in *G<sub>𝛾</sub>*. [Dettagli](UseCases6.md#step3).
*   Ingresso di *𝛼* in *G<sub>𝛾</sub>*. [Dettagli](UseCases7.md#step4).

Oltre a verificare che la sequenza di operazioni in ogni sistema è valida, dobbiamo anche assicurare che
le proprietà della rete sono corrette in ogni momento. Quindi dobbiamo esaminare l'ordine temporale in
cui le operazioni dei vari sistemi possono avvenire. Proseguiamo con questo esame in questa [pagina](UseCases8.md).

## <a name="step1"></a>Sistema 𝜇

La prima identità è *𝜇<sub>0</sub>* con indirizzo Netsukuku 1·0·1·1 in *G<sub>𝜇</sub>*.

```
Mio indirizzo 1·0·1·1.
     globale
      10.0.0.11
     anonimizzante
      10.0.0.75
     interno al mio g-nodo di livello 3
      10.0.0.59
     interno al mio g-nodo di livello 2
      10.0.0.51
     interno al mio g-nodo di livello 1
      10.0.0.41

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 2·
     globale
      10.0.0.16/29
     anonimizzante
      10.0.0.80/29
 3·
     globale
      10.0.0.24/29
     anonimizzante
      10.0.0.88/29
 1·1·
     globale
      10.0.0.12/30
     anonimizzante
      10.0.0.76/30
     interno al mio g-nodo di livello 3
      10.0.0.60/30
 1·0·0·
     globale
      10.0.0.8/31
     anonimizzante
      10.0.0.72/31
     interno al mio g-nodo di livello 3
      10.0.0.56/31
     interno al mio g-nodo di livello 2
      10.0.0.48/31
 1·0·1·0
     globale
      10.0.0.10/32
     anonimizzante
      10.0.0.74/32
     interno al mio g-nodo di livello 3
      10.0.0.58/32
     interno al mio g-nodo di livello 2
      10.0.0.50/32
     interno al mio g-nodo di livello 1
      10.0.0.40/32
```

**sistema 𝜇**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip link set dev eth1 address 00:16:3E:2D:8D:DE
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
ip link set dev eth1 up
ip address add 169.254.119.176 dev eth1
ip address add 10.0.0.11 dev eth1
ip address add 10.0.0.75 dev eth1
ip address add 10.0.0.59 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.41 dev eth1
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.12/30 table ntk
ip route add unreachable 10.0.0.76/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.8/31 table ntk
ip route add unreachable 10.0.0.72/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.10/32 table ntk
ip route add unreachable 10.0.0.74/32 table ntk
ip route add unreachable 10.0.0.58/32 table ntk
ip route add unreachable 10.0.0.50/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.11
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.12/30 table ntk
ip route change unreachable 10.0.0.76/30 table ntk
ip route change unreachable 10.0.0.60/30 table ntk
ip route change unreachable 10.0.0.8/31 table ntk
ip route change unreachable 10.0.0.72/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.10/32 table ntk
ip route change unreachable 10.0.0.74/32 table ntk
ip route change unreachable 10.0.0.58/32 table ntk
ip route change unreachable 10.0.0.50/32 table ntk
ip route change unreachable 10.0.0.40/32 table ntk
```

Ora si rileva il sistema *𝛿* e si entra in *G<sub>𝛿</sub>*. Viene creata l'identità
*𝜇<sub>1</sub>* partendo da *𝜇<sub>0</sub>*. L'identità *𝜇<sub>0</sub>* prende posto in un
nuovo network namespace "entr01".

**sistema 𝜇**
```
ip route add 169.254.253.216 dev eth1 src 169.254.119.176
ip netns add entr01
ip netns exec entr01 sysctl net.ipv4.ip_forward=1
ip netns exec entr01 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr01_eth1 link eth1 type macvlan
ip link set dev entr01_eth1 netns entr01
ip netns exec entr01 ip link set dev entr01_eth1 address 00:16:3E:71:33:12
ip netns exec entr01 sysctl net.ipv4.conf.entr01_eth1.rp_filter=0
ip netns exec entr01 sysctl net.ipv4.conf.entr01_eth1.arp_ignore=1
ip netns exec entr01 sysctl net.ipv4.conf.entr01_eth1.arp_announce=2
ip netns exec entr01 ip link set dev entr01_eth1 up
ip netns exec entr01 ip address add 169.254.101.161 dev entr01_eth1
```

Il modulo Identities del sistema *𝜇*, dal dialogo con il vicino, desume che vanno creati/modificati questi
archi-identità:

*   *𝜇<sub>0</sub>-𝛿<sub>0</sub>*.  
    Per questo arco-identità abbiamo che la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝜇<sub>0</sub>*
    la rotta verso il vicino.  
    I nodi *𝜇<sub>0</sub>* e *𝛿<sub>0</sub>* non fanno parte della stessa rete, quindi non è prevista una tabella
    `ntk_from_xxx`.
*   *𝜇<sub>1</sub>-𝛿<sub>0</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝜇<sub>1</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝜇<sub>1</sub>* e *𝛿<sub>0</sub>* faranno parte della stessa rete, ma solo dopo che il nodo *𝜇<sub>1</sub>*
    avrà costruito la nuova istanza di QspnManager; per fare questo il sistema *𝜇* dovrà costruire un QspnArc
    sull'arco-identità *𝜇<sub>1</sub>-𝛿<sub>0</sub>*. Questo nuovo QspnArc inizialmente non comporta operazioni sulle
    tabelle di routing, fino a quando non si riceve il primo ETP attraverso questo arco e così si scopre l'indirizzo
    Netsukuku di questo vicino. Solo a quel punto, ad esempio, nel network namespace gestito da *𝜇<sub>1</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:1A:C4:45` e la relativa regola.


Il modulo Identities fa queste operazioni:

**sistema 𝜇**
```
ip netns exec entr01 ip route add 169.254.253.216 dev entr01_eth1 src 169.254.101.161
```

Il programma *qspnclient* fa queste operazioni preliminari:

**sistema 𝜇**
```
ip netns exec entr01 ip rule add table ntk
```

Il programma *qspnclient* non deve fare alcuna operazione su rotte verso destinazioni interne al g-nodo
che ha migrato poiché si tratta della migrazione di un singolo nodo.

Il programma *qspnclient* sposta tutte le rotte dal network namespace vecchio al network namespace nuovo.

Avendo però preso per la vecchia identità un indirizzo Netsukuku con posizione *virtuale* nel
livello 0, dove prima aveva posizione *reale*, abbiamo che esiste una ulteriore possibile
destinazione da aggiungere nel nuovo network namespace. La 1·0·1·1.

```
 1·0·1·1
     globale
      10.0.0.11/32
     anonimizzante
      10.0.0.75/32
     interno al mio g-nodo di livello 3
      10.0.0.59/32
     interno al mio g-nodo di livello 2
      10.0.0.51/32
     interno al mio g-nodo di livello 1
      10.0.0.41/32
```

**sistema 𝜇**
```
ip netns exec entr01 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.12/30 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.76/30 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.60/30 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.8/31 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.72/31 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.56/31 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.10/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.74/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.58/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.50/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.11/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.75/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.59/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.51/32 table ntk
ip netns exec entr01 ip route add unreachable 10.0.0.41/32 table ntk

ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/29 table ntk
ip route del 10.0.0.88/29 table ntk
ip route del 10.0.0.12/30 table ntk
ip route del 10.0.0.76/30 table ntk
ip route del 10.0.0.60/30 table ntk
ip route del 10.0.0.8/31 table ntk
ip route del 10.0.0.72/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.10/32 table ntk
ip route del 10.0.0.74/32 table ntk
ip route del 10.0.0.58/32 table ntk
ip route del 10.0.0.50/32 table ntk
ip route del 10.0.0.40/32 table ntk
```

Il programma *qspnclient* rimuove l'indirizzo globale e tutti gli indirizzi interni ai propri g-nodi
dal network namespace vecchio (che era il default).

**sistema 𝜇**
```
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.11
ip address del 10.0.0.11/32 dev eth1
ip address del 10.0.0.75/32 dev eth1
ip address del 10.0.0.59/32 dev eth1
ip address del 10.0.0.51/32 dev eth1
ip address del 10.0.0.41/32 dev eth1
```

Il programma *qspnclient* aggiorna le rotte nel nuovo network namespace sulla base dei migliori percorsi
noti alla vecchia identità *𝜇<sub>0</sub>*.

**sistema 𝜇**
```
ip netns exec entr01 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.12/30 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.76/30 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.60/30 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.8/31 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.72/31 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.56/31 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.10/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.74/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.58/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.50/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.40/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.11/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.75/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.59/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.51/32 table ntk
ip netns exec entr01 ip route change unreachable 10.0.0.41/32 table ntk
```

Poi il sistema *𝜇* per la nuova identità *𝜇<sub>1</sub>* istanzia un QspnManager con il
costruttore `enter_net` passandogli un QspnArc per l'arco-identità *𝜇<sub>1</sub>-𝛿<sub>0</sub>*.
L'indirizzo Netsukuku del nodo *𝜇<sub>1</sub>* sarà 3·1·0·2.

Siccome la nuova identità *𝜇<sub>1</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku. In questo caso nessuno, poiché l'indirizzo è *virtuale* al livello 0.

Inoltre il programma *qspnclient* deve assicurarsi che nel network namespace della nuova identità *𝜇<sub>1</sub>*
siano presenti le regole per la consultazione della tabella `ntk` (non ci sono tabelle `ntk_from_xxx` associate agli
archi-identità di *𝜇<sub>1</sub>*). E non altre regole.

Deve poi aggiungere in questa tabella (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝜇<sub>1</sub>*, inizialmente in stato "unreachable".

```
Mio indirizzo 3·1·0·2.
     globale
      N/A
     anonimizzante
      N/A
     interno al mio g-nodo di livello 3
      N/A
     interno al mio g-nodo di livello 2
      N/A
     interno al mio g-nodo di livello 1
      N/A

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
 3·0·
     globale
      10.0.0.24/30
     anonimizzante
      10.0.0.88/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 3·1·1·
     globale
      10.0.0.30/31
     anonimizzante
      10.0.0.94/31
     interno al mio g-nodo di livello 3
      10.0.0.62/31
     interno al mio g-nodo di livello 2
      10.0.0.50/31
 3·1·0·0
     globale
      10.0.0.28/32
     anonimizzante
      10.0.0.92/32
     interno al mio g-nodo di livello 3
      10.0.0.60/32
     interno al mio g-nodo di livello 2
      10.0.0.48/32
     interno al mio g-nodo di livello 1
      10.0.0.40/32
 3·1·0·1
     globale
      10.0.0.29/32
     anonimizzante
      10.0.0.93/32
     interno al mio g-nodo di livello 3
      10.0.0.61/32
     interno al mio g-nodo di livello 2
      10.0.0.49/32
     interno al mio g-nodo di livello 1
      10.0.0.41/32
```

**sistema 𝜇**
```
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.24/30 table ntk
ip route add unreachable 10.0.0.88/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.30/31 table ntk
ip route add unreachable 10.0.0.94/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
ip route add unreachable 10.0.0.28/32 table ntk
ip route add unreachable 10.0.0.92/32 table ntk
ip route add unreachable 10.0.0.60/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
ip route add unreachable 10.0.0.29/32 table ntk
ip route add unreachable 10.0.0.93/32 table ntk
ip route add unreachable 10.0.0.61/32 table ntk
ip route add unreachable 10.0.0.49/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
```

Dopo un po' di tempo avverranno 2 eventi per *𝜇<sub>1</sub>*, dei quali non possiamo a priori
sapere l'ordine temporale:

*   il QspnManager di *𝜇<sub>1</sub>* emetterà il segnale di `bootstrap_complete` dopo aver ricevuto un ETP dal
    vicino *𝛿<sub>0</sub>*.
*   il programma *qspnclient* nel sistema *𝜇* viene notificato che il suo indirizzo Netsukuku in *G<sub>𝛿</sub>*, che
    temporaneamente era il *virtuale* 3·1·0·2, diventa il *reale* (3·1·0·0). Di conseguenza il programma comunica al
    QspnManager di *𝜇<sub>1</sub>* che ora il suo indirizzo Netsukuku è 3·1·0·0.

Assumiamo che si verifichi prima il segnale di ricezione di un ETP.

Ora il QspnManager di *𝜇<sub>1</sub>* riceve un ETP dal vicino *𝛿<sub>0</sub>*. A questo punto
conosce l'indirizzo Netsukuku del peer sull'arco *𝜇<sub>1</sub>-𝛿<sub>0</sub>*. Per questo il
programma *qspnclient* può creare la tabella `ntk_from_00:16:3E:1A:C4:45` e popolarla con tutte le
destinazioni adatte al suo indirizzo. Poi, ma solo dopo che la tabella sarà stata popolata
e aggiornata, aggiungerà la regola di guardare questa tabella per i pacchetti IP da inoltrare
ricevuti su questo arco.

**sistema 𝜇**
```
(echo; echo "250 ntk_from_00:16:3E:1A:C4:45 # xxx_table_ntk_from_00:16:3E:1A:C4:45_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.28/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.92/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.29/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.93/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
```

Inoltre il QspnManager di *𝜇<sub>1</sub>* sulla base dell'ETP aggiorna la sua mappa.
Attraverso l'arco *𝜇<sub>1</sub>-𝛿<sub>0</sub>* scopre di poter raggiungere la destinazione
(0, 1), ossia il g-nodo 3·1·0·1.

Poi il QspnManager di *𝜇<sub>1</sub>* notifica il segnale `bootstrap_complete`. Per questo
il programma *qspnclient* aggiorna tutte le rotte sulla base dei migliori percorsi noti.

**sistema 𝜇**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/30 table ntk
ip route change unreachable 10.0.0.88/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.30/31 table ntk
ip route change unreachable 10.0.0.94/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change unreachable 10.0.0.28/32 table ntk
ip route change unreachable 10.0.0.92/32 table ntk
ip route change unreachable 10.0.0.60/32 table ntk
ip route change unreachable 10.0.0.48/32 table ntk
ip route change unreachable 10.0.0.40/32 table ntk
ip route change 10.0.0.29/32 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.93/32 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.61/32 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.49/32 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.41/32 table ntk via 169.254.253.216 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.28/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.92/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.29/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.93/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
```

Alla fine di questo aggiornamento delle tabelle, siccome era rimasta in attesa la regola di guardare
la nuova tabella, il programma *qspnclient* la aggiunge.

**sistema 𝜇**
```
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:1A:C4:45 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:1A:C4:45
```

Ora si verifica l'altro evento: il cambio di indirizzo.

Siccome la nuova identità *𝜇<sub>1</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku.  
Deve però tenere presente che alcuni di essi possono essere già stati assegnati dal
precedente detentore del network namespace default o dallo stesso nella fase in cui aveva indirizzo virtuale.
In questo caso nessuno.

Deve poi aggiungere in queste tabelle (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝜇<sub>1</sub>*, inizialmente in stato "unreachable".  
Deve però tenere presente che alcune di esse possono essere già state aggiunte dal
precedente detentore del network namespace default o dallo stesso nella fase in cui aveva indirizzo virtuale.
In questo caso sono state tutte già aggiunte.  
Deve inoltre considerare che prima la sua attuale posizione *reale* al livello 0 (3·1·0·0) poteva essere
stata aggiunta come destinazione nelle tabelle (se le posizioni superiori erano tutte *reali*). In questo
caso essa va rimossa.

Infine, avendo aggiornato nell'identità principale il suo proprio indirizzo che ora è *reale*, deve fare
un aggiornamento di tutte le sue rotte poiché in alcune di esse il *src* poteva essere mancante.

```
Mio indirizzo 3·1·0·0.
     globale
      10.0.0.28
     anonimizzante
      10.0.0.92
     interno al mio g-nodo di livello 3
      10.0.0.60
     interno al mio g-nodo di livello 2
      10.0.0.48
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
 3·0·
     globale
      10.0.0.24/30
     anonimizzante
      10.0.0.88/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 3·1·1·
     globale
      10.0.0.30/31
     anonimizzante
      10.0.0.94/31
     interno al mio g-nodo di livello 3
      10.0.0.62/31
     interno al mio g-nodo di livello 2
      10.0.0.50/31
 3·1·0·0  era valida prima ma non lo è più.
     globale
      10.0.0.28/32
     anonimizzante
      10.0.0.92/32
     interno al mio g-nodo di livello 3
      10.0.0.60/32
     interno al mio g-nodo di livello 2
      10.0.0.48/32
     interno al mio g-nodo di livello 1
      10.0.0.40/32
 3·1·0·1
     globale
      10.0.0.29/32
     anonimizzante
      10.0.0.93/32
     interno al mio g-nodo di livello 3
      10.0.0.61/32
     interno al mio g-nodo di livello 2
      10.0.0.49/32
     interno al mio g-nodo di livello 1
      10.0.0.41/32
```

**sistema 𝜇**
```
ip address add 10.0.0.28 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address add 10.0.0.92 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
ip address add 10.0.0.40 dev eth1
```

**sistema 𝜇**
```
ip route del 10.0.0.28/32 table ntk
ip route del 10.0.0.92/32 table ntk
ip route del 10.0.0.60/32 table ntk
ip route del 10.0.0.48/32 table ntk
ip route del 10.0.0.40/32 table ntk

ip route del 10.0.0.28/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.92/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.60/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.48/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.40/32 table ntk_from_00:16:3E:1A:C4:45
```

**sistema 𝜇**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/30 table ntk
ip route change unreachable 10.0.0.88/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.30/31 table ntk
ip route change unreachable 10.0.0.94/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change 10.0.0.29/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.28
ip route change 10.0.0.93/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.28
ip route change 10.0.0.61/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.60
ip route change 10.0.0.49/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.48
ip route change 10.0.0.41/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.40

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.29/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.93/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
```

Infine, dopo un po' di tempo, l'identità *𝜇<sub>0</sub>* viene dismessa. Il programma *qspnclient*
esegue queste operazioni tramite il modulo Identities:

**sistema 𝜇**
```
ip netns exec entr01 ip route flush table main
ip netns exec entr01 ip link delete entr01_eth1 type macvlan
ip netns del entr01
```

Inoltre, a fronte della rimozione di un network namespace, il programma deve tenere traccia se qualche
particolare tabella `ntk_from_XXX` presente ha cessato di essere referenziata. In quel caso va
rimossa. Nel nostro esempio nessuna.

