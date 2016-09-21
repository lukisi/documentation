# Proof of concept - Eventi - Pagina 8

[Pagina precedente](Eventi7.md)

### migr01: Spostamento vecchia identità in nuovo network namespace

**sistema 𝛽**
```
ip netns add migr01
ip netns exec migr01 sysctl net.ipv4.ip_forward=1
ip netns exec migr01 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev migr01_eth1 link eth1 type macvlan
ip link set dev migr01_eth1 netns migr01
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.rp_filter=0
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.arp_ignore=1
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.arp_announce=2
ip netns exec migr01 ip link set dev migr01_eth1 up
ip netns exec migr01 ip address add 169.254.27.218 dev migr01_eth1
ip netns exec migr01 ip route add 169.254.69.30 dev migr01_eth1 src 169.254.27.218
ip netns exec migr01 ip route add 169.254.94.223 dev migr01_eth1 src 169.254.27.218
ip netns exec migr01 ip route add 169.254.163.36 dev migr01_eth1 src 169.254.27.218
ip netns exec migr01 ip route add 169.254.133.31 dev migr01_eth1 src 169.254.27.218
```

**sistema 𝛼**
```
ip route add 169.254.27.218 dev eth1 src 169.254.69.30
```

**sistema 𝛾**
```
ip route add 169.254.27.218 dev eth1 src 169.254.94.223
```

**sistema 𝜀**
```
ip route add 169.254.27.218 dev eth1 src 169.254.163.36
ip netns exec entr05 ip route add 169.254.27.218 dev entr05_eth1 src 169.254.133.31
```

Sequenza di operazioni eseguita dal modulo Identities su richiesta del programma **qspnclient** quando
l'utente dà il comando `migrate`.

#### migr01: Copia tabelle e regole, spostamento rotte

**sistema 𝛽**
```
ip netns exec migr01 ip rule add table ntk
ip netns exec migr01 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250
ip netns exec migr01 ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249
ip netns exec migr01 ip rule add fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3C:14:33 -j MARK --set-mark 248

ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.16/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.80/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.20/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.84/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.22/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.86/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.62/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.50/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.23/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.87/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.63/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.51/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.41/32 table ntk

ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5

ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA

ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec migr01 ip route change 10.0.0.16/30 table ntk via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.80/30 table ntk via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.56/30 table ntk via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.20/31 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.84/31 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.60/31 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.48/31 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.22/32 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.86/32 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.62/32 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.50/32 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.40/32 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.23/32 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.87/32 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.63/32 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.51/32 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.41/32 table ntk

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change 10.0.0.22/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.86/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.62/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change blackhole 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA
```

Seconda parte di operazioni eseguita dal programma **qspnclient** quando
l'utente dà il comando `migrate`.

In questa situazione notiamo che la preparazione del vecchio namespace per la nuova identità
viene rimandata a dopo che i vicini di *𝛽<sub>1</sub>* abbiano modificato le
rotte che prevedono di usare lui come gateway.

#### migr01: Aggiornamento dei gateway che si sono spostati in un diverso namespace

**sistema 𝛼**
```
ip rule del fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
ip route flush table ntk_from_00:16:3E:EC:A3:E1
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.18
ip route change 10.0.0.84/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.18
ip route change 10.0.0.60/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.58
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.19/32 table ntk
ip route change unreachable 10.0.0.83/32 table ntk
ip route change unreachable 10.0.0.59/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip rule add fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
```

**sistema 𝛾**
```
ip rule del fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
ip route flush table ntk_from_00:16:3E:EC:A3:E1
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.22
ip route change 10.0.0.80/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.22
ip route change 10.0.0.56/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.62
ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.50
ip route change 10.0.0.23/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.40
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip rule add fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
```

**sistema 𝜀**
```
ip route flush table ntk_from_00:16:3E:EC:A3:E1
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1
```

Operazioni eseguite dal programma **qspnclient** quando **TODO: controllare**
il modulo Identities notifica che un arco-identità ha cambiato il suo peer-linklocal.

#### migr01: Pulizia del vecchio namespace per la nuova identità

**sistema 𝛽**
```
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.8/29 table ntk
ip route del 10.0.0.72/29 table ntk
ip route del 10.0.0.24/29 table ntk
ip route del 10.0.0.88/29 table ntk
ip route del 10.0.0.16/30 table ntk
ip route del 10.0.0.80/30 table ntk
ip route del 10.0.0.56/30 table ntk
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.22/32 table ntk
ip route del 10.0.0.86/32 table ntk
ip route del 10.0.0.62/32 table ntk
ip route del 10.0.0.50/32 table ntk
ip route del 10.0.0.40/32 table ntk

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33

iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.23
ip address del 10.0.0.23/32 dev eth1
ip address del 10.0.0.87/32 dev eth1
ip address del 10.0.0.63/32 dev eth1
ip address del 10.0.0.51/32 dev eth1
ip address del 10.0.0.41/32 dev eth1

ip rule del fwmark 250 table ntk_from_00:16:3E:5B:78:D5

ip rule del fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
```

Seconda parte (bis) di operazioni eseguita dal programma **qspnclient** quando
l'utente dà il comando `migrate`. Dopo che i vicini hanno aggiornato il linklocal di questo gateway.

Notiamo che siccome i vicini *𝛼* e *𝛾* sono esterni al g-nodo che ha migrato, dal vecchio network namespace
della vecchia identità vanno rimosse anche le regole per le tabelle `ntk_from_xxx` relative.
Infatti la nuova identità dovrà attendere un ETP da questi archi prima di poter aggiornare le tabelle.

[Pagina seguente](Eventi9.md)
