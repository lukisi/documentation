# Proof of concept - Casi d'uso - Pagina 10

[Pagina precedente](UseCases9.md)

### Ingresso del g-nodo 𝜇+𝛿 in G𝛾

Il prossimo tipo di evento che ci troviamo ad esaminare è quello di un g-nodo *𝜑* che
costituiva una rete a se, il quale ora si trova ad entrare in una diversa rete grazie ad un
qualche arco tra due sistemi delle distinte reti.

Nel nostro esempio assumiamo che si sia formato un solo arco tra *𝛿* e *𝛾*:

**sistema 𝛿**
```
ip route add 169.254.94.223 dev eth1 src 169.254.253.216
```

**sistema 𝛾**
```
ip route add 169.254.253.216 dev eth1 src 169.254.94.223
```

Attraverso un meccanismo di coordinamento nel g-nodo *𝜑* si decide che questo g-nodo farà ingresso
in *G<sub>𝛾</sub>*. Viene incaricato *𝛿* di dialogare con *𝛾* per ottenere un posto per *𝜑* in *G<sub>𝛾</sub>*.
Poi *𝛿* si occupa di propagare in tutto *𝜑* le informazioni riguardanti questo ingresso. Dopo si
avviano le operazioni.

Per prima cosa i sistemi *𝜇* e *𝛿* creano nuove identità e creano un nuovo network namespace per le
vecchie identità. Nel farlo si coordinano fra loro e con eventuali bordernodi (nel nostro caso il
sistema *𝛾*) per la costituzione dei nuovi archi-identità e l'aggiornamento dei vecchi archi-identità
che ora hanno come vertice una nuova pseudo-interfaccia.

**sistema 𝛿**
```
ip netns add entr03
ip netns exec entr03 sysctl net.ipv4.ip_forward=1
ip netns exec entr03 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 ip link set dev entr03_eth1 address 00:16:3E:B9:77:80
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.rp_filter=0
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_ignore=1
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_announce=2
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.83.167 dev entr03_eth1
ip netns exec entr03 ip route add 169.254.242.91 dev entr03_eth1 src 169.254.83.167
ip netns exec entr03 ip route add 169.254.94.223 dev entr03_eth1 src 169.254.83.167
```

**sistema 𝜇**
```
ip netns add entr03
ip netns exec entr03 sysctl net.ipv4.ip_forward=1
ip netns exec entr03 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 ip link set dev entr03_eth1 address 00:16:3E:DF:23:F5
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.rp_filter=0
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_ignore=1
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_announce=2
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.242.91 dev entr03_eth1
ip netns exec entr03 ip route add 169.254.83.167 dev entr03_eth1 src 169.254.242.91
```

**sistema 𝛾**
```
ip route add 169.254.83.167 dev eth1 src 169.254.94.223
```

Inoltre i sistemi *𝜇* e *𝛿* spostano nel nuovo network namespace le configurazioni adeguate alle
vecchie identità.

**sistema 𝛿**
```
ip netns exec entr03 ip rule add table ntk
(echo; echo "249 ntk_from_00:16:3E:DF:23:F5 # xxx_table_ntk_from_00:16:3E:DF:23:F5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:DF:23:F5 -j MARK --set-mark 249
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5
ip route del 10.0.0.28/32 table ntk
ip route del 10.0.0.92/32 table ntk
ip route del 10.0.0.60/32 table ntk
ip route del 10.0.0.48/32 table ntk
ip route del 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:DF:23:F5
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
ip route del 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.29
ip address del 10.0.0.29/32 dev eth1
ip address del 10.0.0.93/32 dev eth1
ip address del 10.0.0.61/32 dev eth1
ip address del 10.0.0.49/32 dev eth1
ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr03 ip route change 10.0.0.40/32 table ntk via 169.254.242.91 dev entr03_eth1
ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5
```

**sistema 𝜇**
```
ip netns exec entr03 ip rule add table ntk
(echo; echo "249 ntk_from_00:16:3E:B9:77:80 # xxx_table_ntk_from_00:16:3E:B9:77:80_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:B9:77:80 -j MARK --set-mark 249
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.41/32 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:B9:77:80
ip route del 10.0.0.29/32 table ntk
ip route del 10.0.0.93/32 table ntk
ip route del 10.0.0.61/32 table ntk
ip route del 10.0.0.49/32 table ntk
ip route del 10.0.0.29/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.93/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:B9:77:80
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
ip route del 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.16/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.80/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.24/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.88/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.30/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.94/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address del 10.0.0.28/32 dev eth1
ip address del 10.0.0.92/32 dev eth1
ip address del 10.0.0.60/32 dev eth1
ip address del 10.0.0.48/32 dev eth1
ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr03 ip route change 10.0.0.41/32 table ntk via 169.254.83.167 dev entr03_eth1
ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:B9:77:80
```

Durante queste operazioni, un processo (network namespace default) nel sistema *𝛿* può comunicare con
un processo (network namespace default) nel sistema *𝜇* e questo deve rimanere possibile.
Non ci sono altri sistemi nella rete da cui questo g-nodo migra, quindi non serve assicurare
altro.

Nei sistemi *𝛿* e *𝜇* si instanziano nuovi QspnManager relativi alle nuove identità.
Essi aggiungono le rotte verso le destinazioni adeguate al loro nuovo indirizzo Netsukuku (per ora in
stato unreachable) sul vecchio network namespace (nel nostro caso il default).

**sistema 𝛿**
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
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip address add 10.0.0.21 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.21
ip address add 10.0.0.85 dev eth1
ip address add 10.0.0.61 dev eth1
ip address add 10.0.0.49 dev eth1
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.20/32 table ntk
ip route add unreachable 10.0.0.84/32 table ntk
ip route add unreachable 10.0.0.60/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
```

**sistema 𝜇**
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
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip address add 10.0.0.20 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.20
ip address add 10.0.0.84 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.21/32 table ntk
ip route add unreachable 10.0.0.85/32 table ntk
ip route add unreachable 10.0.0.61/32 table ntk
ip route add unreachable 10.0.0.49/32 table ntk
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.21/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.85/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
```

Il QspnManager del sistema *𝛾* riceve un nuovo arco, ma per ora nessun ETP da esso.

**sistema 𝛾**
```
(echo; echo "249 ntk_from_00:16:3E:1A:C4:45 # xxx_table_ntk_from_00:16:3E:1A:C4:45_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
```

Il QspnManager del sistema *𝛿* viene costruito con un arco verso *𝛾*, ma per ora nessun ETP da esso.
Poi il sistema *𝛿* riceve un ETP da *𝛾*.

**sistema 𝛿**
```
(echo; echo "248 ntk_from_00:16:3E:5B:78:D5 # xxx_table_ntk_from_00:16:3E:5B:78:D5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
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
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 248
ip rule add fwmark 248 table ntk_from_00:16:3E:5B:78:D5
```

Il sistema *𝛿* propaga l'ETP a *𝜇*.

**sistema 𝜇**
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

Il sistema *𝛿* invia un ETP a *𝛾*

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.50
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:1A:C4:45 -j MARK --set-mark 249
ip rule add fwmark 249 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.50
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
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1 src 10.0.0.50
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
ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.50
ip route change 10.0.0.23/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.40
```

Il sistema *𝛽* vede arrivare un ETP da *𝛾* con il quale scopre un percorso verso la destinazione 2·1·0·.

**sistema 𝛽**
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

Infine, uno dei sistemi nel g-nodo di connettività *𝜑* verifica che queste identità di
connettività possono essere dismesse. Propaga le informazioni nel g-nodo *𝜑* e nei suoi
bordernodi esterni. La dismissione provoca delle operazioni, nel nostro esempio nei
sistemi *𝜇*, *𝛿* e *𝛾*.

**sistema 𝜇**
```
ip netns exec entr03 ip route flush table main
ip netns exec entr03 ip link delete entr03_eth1 type macvlan
ip netns del entr03
sed -i '/xxx_table_ntk_from_00:16:3E:B9:77:80_xxx/d' /etc/iproute2/rt_tables
```

**sistema 𝛿**
```
ip netns exec entr03 ip route flush table main
ip netns exec entr03 ip link delete entr03_eth1 type macvlan
ip netns del entr03
sed -i '/xxx_table_ntk_from_00:16:3E:DF:23:F5_xxx/d' /etc/iproute2/rt_tables
```

**sistema 𝛾**
```
ip route del 169.254.83.167 dev eth1 src 169.254.94.223
```

[Pagina seguente](UseCases11.md)
