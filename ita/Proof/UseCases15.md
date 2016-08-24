# Proof of concept - Casi d'uso - Pagina 15

[Pagina precedente](UseCases14.md)

## Migrazione di un g-nodo

Ora assumiamo un evento che porta alla decisione di far migrare un g-nodo all'interno della stessa rete.

Un nuovo sistema *𝜆* viene rilevato solo dal sistema *𝜀*. Il sistema *𝜆* vuole gestire in autonomia un
g-nodo di livello 1.

Il g-nodo *g<sub>2</sub>(𝜀)* = 1· è saturo. Anche *g<sub>3</sub>(𝜀)* è saturo.
E abbiamo escluso di costituire un nuovo g-nodo di livello 3. Si deve quindi ricorrere ad una *migration path*.

Ricordiamo che indichiamo con *𝜑<sub>0</sub>* il g-nodo *g<sub>1</sub>(𝜀)* = 1·1·.

Si decide di far entrare il sistema *𝜆* in 1· con indirizzo temporaneamente *virtuale* 1·2·.
Si farà poi migrare il g-nodo *𝜑<sub>0</sub>*. La nuova identità *𝜑<sub>1</sub>* andrà in 0·, con indirizzo dapprima
virtuale 0·2 e dopo con indirizzo reale 0·0. L'identità di connettività *𝜑<sub>0</sub>* avrà indirizzo 1·3·.
In seguito l'indirizzo di *𝜆* diventerà il *reale* 1·1·.

Alla fine il g-nodo 1· resta internamente connesso e tutti
i sistemi hanno l'identità principale con un indirizzo IP globale raggiungibile dagli altri.

1.  [Operazioni iniziali in *𝜆*](#Operazioni_iniziali_lambda)
1.  [Arco tra *𝜆* e *𝜀*](#Arco_lambda_epsilon)
1.  [Ingresso di *𝜆* - Prima fase](#Ingresso_lambda_prima_fase)
1.  [Migrazione di *𝜑*](#Migrazione_phi)

### <a name="Operazioni_iniziali_lambda"></a>Operazioni iniziali in *𝜆*

**sistema 𝜆**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip address add 10.0.0.32 dev lo
ip link set dev eth1 address 00:16:3E:06:3E:90
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
ip link set dev eth1 up
ip address add 169.254.109.22 dev eth1
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
```

Va ricordato che questo sistema è un gateway di una sottorete gestita autonomamente di 2 indirizzi.
Questa sottorete autonoma usa il sistema *𝜆* come gateway verso la rete globale. (potrebbe usarne un numero a
piacere se fosse più grande)  
Indipendentemente dall'indirizzo Netsukuku del g-nodo di livello 1 che è stato assegnato al sistema
*𝜆*, in tutti i sistemi all'interno della sottorete autonoma devono esserci delle rotte solo per gli indirizzi
IP interni al g-nodo di livello 1.  
Come caso esemplificativo diciamo che il sistema *𝜆* ha indirizzo 0 e abbiamo un sistema *𝜌* che ha indirizzo 1
ed è direttamente collegato a *𝜆*.

**sistema 𝜆**
```
ip address add 10.0.0.40 dev eth1
ip route add 169.254.110.188 dev eth1 src 169.254.109.22
ip route add 10.0.0.41/32 table ntk via 169.254.110.188 dev eth1 src 10.0.0.40
```

**sistema 𝜌**
```
ip link set dev eth1 up
ip address add 169.254.110.188 dev eth1
ip route add 169.254.109.22 dev eth1 src 169.254.110.188
ip address add 10.0.0.41 dev eth1
ip route add 10.0.0.40/32 via 169.254.109.22 dev eth1 src 10.0.0.41
ip route add 10.0.0.0/25 via 169.254.109.22 dev eth1 src 10.0.0.41
```

Proseguono le operazioni iniziali in *𝜆*.

**sistema 𝜆**
```
ip address add 10.0.0.12 dev eth1
ip address add 10.0.0.76 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.8/30 table ntk
ip route add unreachable 10.0.0.72/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.14/31 table ntk
ip route add unreachable 10.0.0.78/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk

# PREROUTING => map destination
iptables -t nat -A PREROUTING -d 10.0.0.12/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.76/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.60/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.48/31 -j NETMAP --to 10.0.0.40/31
# POSTROUTING => map source
iptables -t nat -A POSTROUTING -d 10.0.0.48/30 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.48/31
iptables -t nat -A POSTROUTING -d 10.0.0.56/29 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.60/31
iptables -t nat -A POSTROUTING -d 10.0.0.0/27  -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31

iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.12
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.8/30 table ntk
ip route change unreachable 10.0.0.72/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.14/31 table ntk
ip route change unreachable 10.0.0.78/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
```

### <a name="Arco_lambda_epsilon"></a>Arco tra *𝜆* e *𝜀*

**sistema 𝜆**
```
ip route add 169.254.163.36 dev eth1 src 169.254.109.22
```

**sistema 𝜀**
```
ip route add 169.254.109.22 dev eth1 src 169.254.163.36
```

### <a name="Ingresso_lambda_prima_fase"></a>Ingresso di *𝜆* - Prima fase

**sistema 𝜆**
```
ip netns add entr06
ip netns exec entr06 sysctl net.ipv4.ip_forward=1
ip netns exec entr06 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr06_eth1 link eth1 type macvlan
ip link set dev entr06_eth1 netns entr06
ip netns exec entr06 ip link set dev entr06_eth1 address 00:16:3E:FC:CA:2C
ip netns exec entr06 sysctl net.ipv4.conf.entr06_eth1.rp_filter=0
ip netns exec entr06 sysctl net.ipv4.conf.entr06_eth1.arp_ignore=1
ip netns exec entr06 sysctl net.ipv4.conf.entr06_eth1.arp_announce=2
ip netns exec entr06 ip link set dev entr06_eth1 up
ip netns exec entr06 ip address add 169.254.34.45 dev entr06_eth1
ip netns exec entr06 ip route add 169.254.163.36 dev entr06_eth1 src 169.254.34.45
```

**sistema 𝜀**
```
ip route add 169.254.34.45 dev eth1 src 169.254.163.36
```

**sistema 𝜆**
```
ip netns exec entr06 ip rule add table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.8/30 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.72/30 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.14/31 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.78/31 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.50/31 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.12/31 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.76/31 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec entr06 ip route add unreachable 10.0.0.48/31 table ntk
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/29 table ntk
ip route del 10.0.0.88/29 table ntk
ip route del 10.0.0.8/30 table ntk
ip route del 10.0.0.72/30 table ntk
ip route del 10.0.0.56/30 table ntk
ip route del 10.0.0.14/31 table ntk
ip route del 10.0.0.78/31 table ntk
ip route del 10.0.0.62/31 table ntk
ip route del 10.0.0.50/31 table ntk

iptables -t nat -D PREROUTING -d 10.0.0.12/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -D PREROUTING -d 10.0.0.76/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -D PREROUTING -d 10.0.0.60/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -D PREROUTING -d 10.0.0.48/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -D POSTROUTING -d 10.0.0.48/30 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.48/31
iptables -t nat -D POSTROUTING -d 10.0.0.56/29 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.60/31
iptables -t nat -D POSTROUTING -d 10.0.0.0/27  -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31

iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.12
ip address del 10.0.0.12/32 dev eth1
ip address del 10.0.0.76/32 dev eth1
ip address del 10.0.0.60/32 dev eth1
ip address del 10.0.0.48/32 dev eth1

ip netns exec entr06 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.8/30 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.72/30 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.56/30 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.14/31 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.78/31 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.50/31 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.12/31 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.76/31 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.60/31 table ntk
ip netns exec entr06 ip route change unreachable 10.0.0.48/31 table ntk
```

**sistema 𝜆**
```
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
ip route add unreachable 10.0.0.22/31 table ntk
ip route add unreachable 10.0.0.86/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
```

**sistema 𝜆**
```
(echo; echo "250 ntk_from_00:16:3E:3C:14:33 # xxx_table_ntk_from_00:16:3E:3C:14:33_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33
```

**sistema 𝜀**
```
(echo; echo "248 ntk_from_00:16:3E:06:3E:90 # xxx_table_ntk_from_00:16:3E:06:3E:90_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:06:3E:90
```

### <a name="Migrazione_phi"></a>Migrazione di *𝜑*

#### <a name="Cambio_namespace_phi0"></a>Cambio namespace per i singoli nodi in *𝜑<sub>0</sub>*

I singoli nodi in *𝜑<sub>0</sub>* - vale a dire le identità *𝜀<sub>1</sub>* *𝛽<sub>1</sub>* e
*𝛾<sub>0</sub>* - si spostano nel network namespace **migr02** e diventano di connettività.

Notiamo che nel nuovo network namespace il g-nodo *𝜑<sub>0</sub>* avrà indirizzo virtuale al
livello 1. Per questo avrà un'ulteriore possibile destinazione a livello 1 (con prefix=31).
Ogni singolo nodo al suo interno non avrà tra le possibili destinazioni quelle a livello 0.

Verifichiamo che il *passaggio* delle vecchie identità nei singoli nodi in *𝜑<sub>0</sub>* ad un nuovo network
namespace nel loro sistema è fatto in modo tale che in nessun momento si interrompe il servizio di
forwarding offerto dal g-nodo *𝜑*. Ad esempio, una connessione tra il sistema *𝛼* e il sistema *𝛿*
non si interrompe e non perde pacchetti.

Il modulo Identities produce questi comandi per preparare il nuovo network
namespace per la vecchia identità:

**sistema 𝜀**
```
ip netns add migr02
ip netns exec migr02 sysctl net.ipv4.ip_forward=1
ip netns exec migr02 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev migr02_eth1 link eth1 type macvlan
ip link set dev migr02_eth1 netns migr02
ip netns exec migr02 ip link set dev migr02_eth1 address 00:16:3E:3B:9F:45
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.rp_filter=0
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.arp_ignore=1
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.arp_announce=2
ip netns exec migr02 ip link set dev migr02_eth1 up
ip netns exec migr02 ip address add 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route add 169.254.96.141 dev migr02_eth1 src 169.254.241.153
ip netns exec migr02 ip route add 169.254.42.4 dev migr02_eth1 src 169.254.241.153
ip netns exec migr02 ip route add 169.254.109.22 dev migr02_eth1 src 169.254.241.153
ip netns exec migr02 ip route add 169.254.34.45 dev migr02_eth1 src 169.254.241.153
```

**sistema 𝛽**
```
ip netns add migr02
ip netns exec migr02 sysctl net.ipv4.ip_forward=1
ip netns exec migr02 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev migr02_eth1 link eth1 type macvlan
ip link set dev migr02_eth1 netns migr02
ip netns exec migr02 ip link set dev migr02_eth1 address 00:16:3E:BD:34:98
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.rp_filter=0
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.arp_ignore=1
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.arp_announce=2
ip netns exec migr02 ip link set dev migr02_eth1 up
ip netns exec migr02 ip address add 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route add 169.254.241.153 dev migr02_eth1 src 169.254.42.4
ip netns exec migr02 ip route add 169.254.24.198 dev migr02_eth1 src 169.254.42.4
```

**sistema 𝛾**
```
ip netns add migr02
ip netns exec migr02 sysctl net.ipv4.ip_forward=1
ip netns exec migr02 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev migr02_eth1 link eth1 type macvlan
ip link set dev migr02_eth1 netns migr02
ip netns exec migr02 ip link set dev migr02_eth1 address 00:16:3E:AF:4C:2A
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.rp_filter=0
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.arp_ignore=1
ip netns exec migr02 sysctl net.ipv4.conf.migr02_eth1.arp_announce=2
ip netns exec migr02 ip link set dev migr02_eth1 up
ip netns exec migr02 ip address add 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route add 169.254.96.141 dev migr02_eth1 src 169.254.24.198
ip netns exec migr02 ip route add 169.254.42.4 dev migr02_eth1 src 169.254.24.198
ip netns exec migr02 ip route add 169.254.253.216 dev migr02_eth1 src 169.254.24.198
```

**sistema 𝜆**
```
ip route add 169.254.241.153 dev eth1 src 169.254.109.22
ip netns exec entr06 ip route add 169.254.241.153 dev eth1 src 169.254.34.45
```

**sistema 𝛿**
```
ip route add 169.254.24.198 dev eth1 src 169.254.253.216
```

**sistema 𝛽**
```
ip route add 169.254.24.198 dev eth1 src 169.254.96.141
ip route add 169.254.241.153 dev eth1 src 169.254.96.141
```

Il programma *qspnclient*, per trasferire le impostazioni relative alla sua vecchia identità
nel nuovo network namespace, produce questi comandi:

**sistema 𝜀**
```
ip netns exec migr02 ip rule add table ntk
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
ip netns exec migr02 ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:06:3E:90 -j MARK --set-mark 248
ip netns exec migr02 ip rule add fwmark 248 table ntk_from_00:16:3E:06:3E:90
(echo; echo "247 ntk_from_00:16:3E:BD:34:98 # xxx_table_ntk_from_00:16:3E:BD:34:98_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:BD:34:98 -j MARK --set-mark 247
ip netns exec migr02 ip rule add fwmark 247 table ntk_from_00:16:3E:BD:34:98

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:06:3E:90

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:06:3E:90

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98
```

**sistema 𝛽**
```
ip netns exec migr02 ip rule add table ntk
(echo; echo "247 ntk_from_00:16:3E:AF:4C:2A # xxx_table_ntk_from_00:16:3E:AF:4C:2A_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:AF:4C:2A -j MARK --set-mark 247
ip netns exec migr02 ip rule add fwmark 247 table ntk_from_00:16:3E:AF:4C:2A
(echo; echo "246 ntk_from_00:16:3E:3B:9F:45 # xxx_table_ntk_from_00:16:3E:3B:9F:45_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3B:9F:45 -j MARK --set-mark 246
ip netns exec migr02 ip rule add fwmark 246 table ntk_from_00:16:3E:3B:9F:45

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:AF:4C:2A

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:3B:9F:45

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:AF:4C:2A

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:3B:9F:45
```

**sistema 𝛾**
```
ip netns exec migr02 ip rule add table ntk
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:1A:C4:45 -j MARK --set-mark 249
ip netns exec migr02 ip rule add fwmark 249 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 248
ip netns exec migr02 ip rule add fwmark 248 table ntk_from_00:16:3E:EC:A3:E1
(echo; echo "247 ntk_from_00:16:3E:BD:34:98 # xxx_table_ntk_from_00:16:3E:BD:34:98_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:BD:34:98 -j MARK --set-mark 247
ip netns exec migr02 ip rule add fwmark 247 table ntk_from_00:16:3E:BD:34:98

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1

ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98
```

**completare**

[Pagina seguente](UseCases16.md)
