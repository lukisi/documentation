# Proof of concept - Casi d'uso - Pagina 12

[Pagina precedente](UseCases11.md)

## Migrazione di un singolo nodo

Ora assumiamo un evento che porta alla decisione di far migrare un singolo nodo da un g-nodo
ad un altro all'interno della stessa rete.

Un nuovo sistema *𝜀* viene rilevato solo dal sistema *𝛽*. Assumiamo per convenienza di non voler
costituire un nuovo g-nodo di livello 3. Quando indicheremo gli indirizzi Netsukuku quindi tralasceremo
di indicare la parte iniziale che è sempre 2·.

Il sistema *𝜀* può fare ingresso solo in un g-nodo a cui appartiene anche *𝛽*, poiché esso è il suo unico vicino.
Il g-nodo *g<sub>1</sub>(𝛽)* è saturo. Anche *g<sub>2</sub>(𝛽)* è saturo. Anche *g<sub>3</sub>(𝛽)* è saturo.
E abbiamo escluso di costituire un nuovo g-nodo di livello 3. Si deve quindi ricorrere ad una *migration path*.

**Appunto**: quando un g-nodo di livello *i* vuole fare ingresso in una nuova rete, se tramite i suoi archi
non può trovare un g-nodo di livello *j* non saturo, con *i* + 1 ≤ *j* ≤ *i* + *k* dove *k* è un intero positivo
piccolo a piacere, allora si tenta di ricorrere ad una *migration path*. Si vuole cioè evitare di ricorrere ad una
migration path appena un singolo nodo non trova libero un g-nodo di livello 1, ma allo stesso tempo evitare
di occupare un g-nodo di livello alto per l'ingresso di un singolo nodo. Ragionamento simile per
l'ingresso di un g-nodo di livello *i*.

Si decide di far entrare il sistema *𝜀* in *g<sub>1</sub>(𝛽)* = 1·1· con indirizzo temporaneamente *virtuale* 1·1·2.
Si farà poi migrare il sistema *𝛽* in *g<sub>1</sub>(𝛼)* = 0·1·, con indirizzo dapprima virtuale 0·1·2 e dopo
con indirizzo reale 0·1·1. L'identità di connettività con cui *𝛽* resta in 1·1· avrà indirizzo 1·1·3.
In seguito l'indirizzo di *𝜀* diventerà il *reale* 1·1·1. Il g-nodo 1·1· resta internamente connesso e tutti
i sistemi hanno l'identità principale con un indirizzo IP globale raggiungibile dagli altri.

Tutti i dialoghi tra vicini che portano alla scelta delle nuove posizioni occupate e alla comunicazione delle
relative informazioni ai vari sistemi interessati vengono fatte inizialmente (dopo la realizzazione dell'arco
tra *𝜀* e *𝛽*) tramite meccanismi estranei ai moduli di cui stiamo trattando in questa proof-of-concept.

### Operazioni iniziali in *𝜀*

**sistema 𝜀**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip link set dev eth1 address 00:16:3E:3C:14:33
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
ip link set dev eth1 up
ip address add 169.254.163.36 dev eth1
ip address add 10.0.0.28 dev eth1
ip address add 10.0.0.92 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
ip address add 10.0.0.40 dev eth1
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
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
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
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
ip route change unreachable 10.0.0.29/32 table ntk
ip route change unreachable 10.0.0.93/32 table ntk
ip route change unreachable 10.0.0.61/32 table ntk
ip route change unreachable 10.0.0.49/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
```

### Arco tra *𝜀* e *𝛽*

Il modulo Neighborhood produce questi comandi:

**sistema 𝜀**
```
ip route add 169.254.96.141 dev eth1 src 169.254.163.36
```

**sistema 𝛽**
```
ip route add 169.254.163.36 dev eth1 src 169.254.96.141
```

Il modulo Identities produce questi comandi:

**sistema 𝜀**
```
ip netns add entr05
ip netns exec entr05 sysctl net.ipv4.ip_forward=1
ip netns exec entr05 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr05_eth1 link eth1 type macvlan
ip link set dev entr05_eth1 netns entr05
ip netns exec entr05 ip link set dev entr05_eth1 address 00:16:3E:13:28:B2
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.rp_filter=0
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.arp_ignore=1
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.arp_announce=2
ip netns exec entr05 ip link set dev entr05_eth1 up
ip netns exec entr05 ip address add 169.254.133.31 dev entr05_eth1
ip netns exec entr05 ip route add 169.254.96.141 dev entr05_eth1 src 169.254.133.31
```

**sistema 𝛽**
```
ip route add 169.254.133.31 dev eth1 src 169.254.96.141
```

**sistema 𝜀**
```
ip netns exec entr05 ip rule add table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.24/30 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.88/30 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.30/31 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.94/31 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.50/31 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.28/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.92/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.60/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.48/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.29/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.93/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.61/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.49/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.41/32 table ntk
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.8/29 table ntk
ip route del 10.0.0.72/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/30 table ntk
ip route del 10.0.0.88/30 table ntk
ip route del 10.0.0.56/30 table ntk
ip route del 10.0.0.30/31 table ntk
ip route del 10.0.0.94/31 table ntk
ip route del 10.0.0.62/31 table ntk
ip route del 10.0.0.50/31 table ntk
ip route del 10.0.0.29/32 table ntk
ip route del 10.0.0.93/32 table ntk
ip route del 10.0.0.61/32 table ntk
ip route del 10.0.0.49/32 table ntk
ip route del 10.0.0.41/32 table ntk
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address del 10.0.0.28 dev eth1
ip address del 10.0.0.92 dev eth1
ip address del 10.0.0.60 dev eth1
ip address del 10.0.0.48 dev eth1
ip address del 10.0.0.40 dev eth1
ip netns exec entr05 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.24/30 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.88/30 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.56/30 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.30/31 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.94/31 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.50/31 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.28/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.92/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.60/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.48/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.40/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.29/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.93/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.61/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.49/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.41/32 table ntk
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.16/30 table ntk
ip route add unreachable 10.0.0.80/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.20/31 table ntk
ip route add unreachable 10.0.0.84/31 table ntk
ip route add unreachable 10.0.0.60/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.22/32 table ntk
ip route add unreachable 10.0.0.86/32 table ntk
ip route add unreachable 10.0.0.62/32 table ntk
ip route add unreachable 10.0.0.50/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
ip route add unreachable 10.0.0.23/32 table ntk
ip route add unreachable 10.0.0.87/32 table ntk
ip route add unreachable 10.0.0.63/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
```

**completare**

[Pagina seguente](UseCases13.md)
