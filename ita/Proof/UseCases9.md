# Proof of concept - Casi d'uso - Pagina 9

[Pagina precedente](UseCases8.md)

### Ingresso di 𝜇 in G𝛿

Analogo all'evento visto prima.

**sistema 𝜇**
```
ip route add 169.254.253.216 dev eth1 src 169.254.119.176
```

**sistema 𝛿**
```
ip route add 169.254.119.176 dev eth1 src 169.254.253.216
```

**sistema 𝜇**
```
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
ip netns exec entr01 ip route add 169.254.253.216 dev entr01_eth1 src 169.254.101.161
```

**sistema 𝛿**
```
ip route add 169.254.101.161 dev eth1 src 169.254.253.216
```

**sistema 𝜇**
```
ip netns exec entr01 ip rule add table ntk
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

iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.11
ip address del 10.0.0.11/32 dev eth1
ip address del 10.0.0.75/32 dev eth1
ip address del 10.0.0.59/32 dev eth1
ip address del 10.0.0.51/32 dev eth1
ip address del 10.0.0.41/32 dev eth1

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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:1A:C4:45 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:1A:C4:45
```

**sistema 𝛿**
```
(echo; echo "250 ntk_from_00:16:3E:2D:8D:DE # xxx_table_ntk_from_00:16:3E:2D:8D:DE_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:2D:8D:DE -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:2D:8D:DE
```

**sistema 𝜇**
```
ip address add 10.0.0.28 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address add 10.0.0.92 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
ip address add 10.0.0.40 dev eth1
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

**sistema 𝛿**
```
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
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
ip route change 10.0.0.28/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.29
ip route change 10.0.0.92/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.29
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41
```

**sistema 𝜇**
```
ip netns exec entr01 ip route flush table main
ip netns exec entr01 ip link delete entr01_eth1 type macvlan
ip netns del entr01
```

**sistema 𝛿**
```
ip route del 169.254.101.161 dev eth1 src 169.254.253.216
```

[Pagina seguente](UseCases10.md)
