# Proof of concept - Casi d'uso - Pagina 8

[Pagina precedente](UseCases7.md)

## Sequenza temporale

Oltre a verificare che la sequenza di operazioni in ogni sistema è valida, dobbiamo anche assicurare che
le proprietà della rete sono corrette in ogni momento. Quindi dobbiamo esaminare l'ordine temporale in
cui le operazioni dei vari sistemi possono avvenire. Con questa attenzione rivediamo gli eventi che hanno
portato la rete del nostro esempio allo stato attuale.

### Prime operazioni

Le prime operazioni ogni sistema le fa senza alcuna relazione con l'ambiente circostante
e quindi si possono simulare in qualsiasi ordine.

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

**sistema 𝛽**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip link set dev eth1 address 00:16:3E:EC:A3:E1
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
ip link set dev eth1 up
ip address add 169.254.96.141 dev eth1
ip address add 10.0.0.10 dev eth1
ip address add 10.0.0.74 dev eth1
ip address add 10.0.0.58 dev eth1
ip address add 10.0.0.50 dev eth1
ip address add 10.0.0.40 dev eth1
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
ip route add unreachable 10.0.0.11/32 table ntk
ip route add unreachable 10.0.0.75/32 table ntk
ip route add unreachable 10.0.0.59/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.10
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
ip route change unreachable 10.0.0.11/32 table ntk
ip route change unreachable 10.0.0.75/32 table ntk
ip route change unreachable 10.0.0.59/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
```

**sistema 𝛾**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip link set dev eth1 address 00:16:3E:5B:78:D5
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
ip link set dev eth1 up
ip address add 169.254.94.223 dev eth1
ip address add 10.0.0.22 dev eth1
ip address add 10.0.0.86 dev eth1
ip address add 10.0.0.62 dev eth1
ip address add 10.0.0.50 dev eth1
ip address add 10.0.0.40 dev eth1
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
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
ip route add unreachable 10.0.0.23/32 table ntk
ip route add unreachable 10.0.0.87/32 table ntk
ip route add unreachable 10.0.0.63/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.22
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.23/32 table ntk
ip route change unreachable 10.0.0.87/32 table ntk
ip route change unreachable 10.0.0.63/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
```

**sistema 𝛿**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip link set dev eth1 address 00:16:3E:1A:C4:45
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
ip link set dev eth1 up
ip address add 169.254.253.216 dev eth1
ip address add 10.0.0.29 dev eth1
ip address add 10.0.0.93 dev eth1
ip address add 10.0.0.61 dev eth1
ip address add 10.0.0.49 dev eth1
ip address add 10.0.0.41 dev eth1
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
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.29
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

### Ingresso di 𝛽 in G𝛾

Il primo tipo di evento che ci troviamo ad esaminare è quello di un singolo sistema che
costituiva una rete a se, il quale ora si trova ad entrare in una diversa rete grazie ad un
singolo arco con un altro sistema.

Per prima cosa avviene sicuramente la formazione dell'arco da parte del modulo Neighborhood.
Quindi abbiamo in un qualsiasi ordine queste operazioni:

**sistema 𝛽**
```
ip route add 169.254.94.223 dev eth1 src 169.254.96.141
```

**sistema 𝛾**
```
ip route add 169.254.96.141 dev eth1 src 169.254.94.223
```

Poi nel sistema *𝛽* si crea una nuova identità, con la relativa creazione di un nuovo network
namespace per quella vecchia. Durante questa creazione si ha il dialogo tra i moduli Identities
con il quale si decide a riguardo degli archi-identità, con le relative nuove rotte dirette
nelle tabelle del kernel.  
In questo caso a fare ingresso in un'altra rete (vale anche per le migrazioni? **TODO**) è un singolo nodo che costituiva
l'intera rete (in generale, non aveva altri archi oltre a quello col sistema che ha causato la migrazione).  
Per questo sicuramente non avrà bisogno di coordinarsi con altri sistemi per le fasi di spostamento delle
vecchie rotte nel nuovo network namespace.

Quindi diciamo che la seconda parte di operazioni nel sistema *𝛽* (nell'elenco seguente) avviene senza alcun
bisogno di coordinamento con altri sistemi.

**sistema 𝛽**
```
ip netns add entr02
ip netns exec entr02 sysctl net.ipv4.ip_forward=1
ip netns exec entr02 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr02_eth1 link eth1 type macvlan
ip link set dev entr02_eth1 netns entr02
ip netns exec entr02 ip link set dev entr02_eth1 address 00:16:3E:8E:91:B9
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.rp_filter=0
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.arp_ignore=1
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.arp_announce=2
ip netns exec entr02 ip link set dev entr02_eth1 up
ip netns exec entr02 ip address add 169.254.215.29 dev entr02_eth1
ip netns exec entr02 ip route add 169.254.94.223 dev entr02_eth1 src 169.254.215.29
```

**sistema 𝛾**
```
ip route add 169.254.215.29 dev eth1 src 169.254.94.223
```

**sistema 𝛽**
```
ip netns exec entr02 ip rule add table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.12/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.76/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.60/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.8/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.72/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.56/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.11/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.75/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.59/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.51/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.41/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.10/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.74/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.58/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.50/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.40/32 table ntk
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
ip route del 10.0.0.11/32 table ntk
ip route del 10.0.0.75/32 table ntk
ip route del 10.0.0.59/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.10
ip address del 10.0.0.10/32 dev eth1
ip address del 10.0.0.74/32 dev eth1
ip address del 10.0.0.58/32 dev eth1
ip address del 10.0.0.50/32 dev eth1
ip address del 10.0.0.40/32 dev eth1
ip netns exec entr02 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.12/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.76/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.60/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.8/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.72/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.56/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.11/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.75/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.59/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.51/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.41/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.10/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.74/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.58/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.50/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.40/32 table ntk
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

Ancora il sistema *𝛽* (la sua nuova identità) non ha completato il bootstrap nella nuova rete, quindi per
primo sarà esso a ricevere un ETP prodotto dal sistema *𝛾*.

**sistema 𝛽**
```
(echo; echo "250 ntk_from_00:16:3E:5B:78:D5 # xxx_table_ntk_from_00:16:3E:5B:78:D5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.22/32 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.86/32 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.62/32 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.50/32 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.94.223 dev eth1
ip route change unreachable 10.0.0.23/32 table ntk
ip route change unreachable 10.0.0.87/32 table ntk
ip route change unreachable 10.0.0.63/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5
```

Poi il sistema *𝛽* trasmette un ETP al sistema *𝛾*.

**sistema 𝛾**
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
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.23/32 table ntk
ip route change unreachable 10.0.0.87/32 table ntk
ip route change unreachable 10.0.0.63/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
```

Poi il sistema *𝛽* si assegna un indirizzo *reale* e lo comunica con un nuovo ETP al sistema *𝛾*.  
Ricordiamo che quando si assegna un indirizzo *reale* nel network namespace default, il programma
aggiorna le rotte per fare in modo di mettere in esse un *src* preferito.

**sistema 𝛽**
```
ip address add 10.0.0.23 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.23
ip address add 10.0.0.87 dev eth1
ip address add 10.0.0.63 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.41 dev eth1
ip route del 10.0.0.23/32 table ntk
ip route del 10.0.0.87/32 table ntk
ip route del 10.0.0.63/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
ip route del 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
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
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
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

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.23/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.40
```

Infine il sistema *𝛽* dismette la sua vecchia identità. Il modulo Identities lo comunica al sistema *𝛾*.

**sistema 𝛽**
```
ip netns exec entr02 ip route flush table main
ip netns exec entr02 ip link delete entr02_eth1 type macvlan
ip netns del entr02
```

**sistema 𝛾**
```
ip route del 169.254.215.29 dev eth1 src 169.254.94.223
```

Da questo momento il sistema *𝛽* è in grado di comunicare con il sistema *𝛾*. Può farlo usando l'indirizzo
IP globale, oppure l'indirizzo IP anonimizzante, oppure quello interno al g-nodo di livello 1, oppure quello
interno al g-nodo di livello 2, oppure 3. Vale a dire gli indirizzi IP:

*   `10.0.0.22` - globale
*   `10.0.0.86` - anonimizzante
*   `10.0.0.62` - interno livello 3
*   `10.0.0.50` - interno livello 2
*   `10.0.0.40` - interno livello 1

In realtà un sistema nella rete Netsukuku per contattare un altro sistema usa sempre il suo
indirizzo IP interno al massimo comune g-nodo dei due sistemi, oppure quello anonimizzante.
Infatti un processo quando vuole contattare un altro sistema usa il nome del sistema; il meccanismo
di risoluzione dei nomi nel database ANDNA confronta l'indirizzo Netsukuku trovato con quello
che è attualmente assegnato all'identità principale del proprio sistema e restituisce il suo
indirizzo IP interno al massimo comune g-nodo dei due sistemi, oppure quello anonimizzante.

Nel nostro caso gli indirizzi IP
che il sistema *𝛽* usa per comunicare con il sistema *𝛾* sono:

*   `10.0.0.86` - anonimizzante
*   `10.0.0.40` - interno livello 1

Noi ci concentreremo a verificare che si mantenga la connettività tra i due sistemi quando viene usato
per la connessione l'indirizzo IP interno al massimo comune g-nodo.

Durante le prime fasi dell'esempio trattato in questo documento non dovrebbe mai perdersi
questa connettività tra *𝛽* e *𝛾*. Fino al momento in cui simuleremo la migrazione del sistema *𝛽*
dal g-nodo 2·1·1· nel g-nodo 2·0·1·.

Dovremmo poter verificare periodicamente (o nelle fasi critiche di variazione della rete) la
connettività tra due sistemi, come *𝛽* e *𝛾*. Un banale test può essere un `ping` che eseguito dal
sistema *𝛽* con l'indirizzo IP di *𝛾* (interno al massimo comune g-nodo dei due sistemi) dia
esito positivo. Ma sarebbe più corretto verificare anche che a rispondere sia effettivamente il
sistema *𝛾* e non un altro. **TODO**

[Pagina seguente](UseCases9.md)
