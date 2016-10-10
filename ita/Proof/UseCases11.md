# Proof of concept - Casi d'uso - Pagina 11

[Pagina precedente](UseCases10.md)

### Ingresso di 𝛼 in G𝛾

Il prossimo evento vede ancora un sistema *𝛼* che costituiva una rete a se, il quale ora si trova
ad entrare in una diversa rete grazie ad un singolo arco con un altro sistema *𝛽* in una diversa rete.

Avviene la formazione dell'arco da parte del modulo Neighborhood. Poi il sistema *𝛼* prende la decisione
di entrare, quindi il modulo Identities crea il network namespace in *𝛼* e le nuove rotte sia in *𝛼* che in *𝛽*.

**sistema 𝛼**
```
ip route add 169.254.96.141 dev eth1 src 169.254.69.30
```

**sistema 𝛽**
```
ip route add 169.254.69.30 dev eth1 src 169.254.96.141
```

**sistema 𝛼**
```
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
ip netns exec entr04 ip route add 169.254.96.141 dev entr04_eth1 src 169.254.202.128
```

**sistema 𝛽**
```
ip route add 169.254.202.128 dev eth1 src 169.254.96.141
```

Nel sistema *𝛼* continuano le operazioni per lo spostamento della vecchia identità nel
nuovo network namespace. Poi la nuova identità assume un indirizzo (temporaneamente virtuale)
nel network namespace default.

**sistema 𝛼**
```
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
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.26
ip address del 10.0.0.26/32 dev eth1
ip address del 10.0.0.90/32 dev eth1
ip address del 10.0.0.58/32 dev eth1
ip address del 10.0.0.50/32 dev eth1
ip address del 10.0.0.40/32 dev eth1
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

Viene costituito un arco nel modulo Qspn delle identità di *𝛼* e *𝛽*.

**sistema 𝛼**
```
(echo; echo "250 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250
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

**sistema 𝛽**
```
(echo; echo "249 ntk_from_00:16:3E:FD:E2:AA # xxx_table_ntk_from_00:16:3E:FD:E2:AA_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249
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

Ora il sistema *𝛼*, che è in fase di bootstrap, riceve un ETP dal sistema *𝛽*. Assumiamo anche
che subito dopo il sistema *𝛼* ottiene il suo definitivo indirizzo reale.

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
ip rule add fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
ip address add 10.0.0.18 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.18
ip address add 10.0.0.82 dev eth1
ip address add 10.0.0.58 dev eth1
ip route del 10.0.0.16/30 table ntk
ip route del 10.0.0.80/30 table ntk
ip route del 10.0.0.56/30 table ntk
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
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

Poi il sistema *𝛼* completa il bootstrap e trasmette un ETP al sistema *𝛽*. Questo viene propagato al
resto della rete.

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
ip rule add fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
```

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.50
ip route change 10.0.0.23/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.40
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

**sistema 𝛿**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.80/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.56/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.22/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.86/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.62/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.50/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
```

**sistema 𝜇**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.80/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.56/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.60
ip route change 10.0.0.22/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.86/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.62/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.60
ip route change 10.0.0.50/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.48
ip route change 10.0.0.21/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.85/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.61/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.60
ip route change 10.0.0.49/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.48
ip route change 10.0.0.41/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.40
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.21/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.85/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
```

Infine, prima o poi, la vecchia identità del sistema *𝛼* viene dismessa.

**sistema 𝛼**
```
ip netns exec entr04 ip route flush table main
ip netns exec entr04 ip link delete entr04_eth1 type macvlan
ip netns del entr04
```

**sistema 𝛽**
```
ip route del 169.254.202.128 dev eth1 src 169.254.96.141
```

[Pagina seguente](UseCases12.md)
