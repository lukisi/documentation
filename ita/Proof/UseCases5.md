# Proof of concept - Casi d'uso - Parte 5

## <a name="step2"></a>Sistemi 𝛽 e 𝛾

Nel sistema *𝛽* la prima identità è *𝛽<sub>0</sub>* con indirizzo Netsukuku 1·0·1·0 in *G<sub>𝛽</sub>*.

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

Nel sistema *𝛾* la prima identità è *𝛾<sub>0</sub>* con indirizzo Netsukuku 2·1·1·0 in *G<sub>𝛾</sub>*.

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

Ora il sistema *𝛽* rileva il sistema *𝛾* e entra in *G<sub>𝛾</sub>*, creando l'identità
*𝛽<sub>1</sub>* partendo da *𝛽<sub>0</sub>*. L'identità *𝛽<sub>0</sub>* prende posto in un
nuovo network namespace "entr02", poi viene dismessa.

**sistema 𝛽**
```
ip route add 169.254.94.223 dev eth1 src 169.254.96.141
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
ip netns exec entr02 ip route flush table main
ip netns exec entr02 ip link delete entr02_eth1 type macvlan
ip netns del entr02
```

Nel sistema *𝛾* nel frattempo:

**sistema 𝛾**
```
# rileva eth1 del 𝛽 (𝛽0)
ip route add 169.254.96.141 dev eth1 src 169.254.94.223
# aggiunge entr02_eth1 del 𝛽 (che passa a 𝛽0)
ip route add 169.254.215.29 dev eth1 src 169.254.94.223
# Prepara tabella per eth1 del 𝛽, che 𝛽1 ora è in nostra rete. Ma prima di usarla va popolata e aggiornata.
(echo; echo "250 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
# Popola tabella con possibili destinazioni.
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
# Aggiorna sulla base del primo ETP che dice che 𝛽1 è virtuale al livello 0
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
# tabella from_xxx è popolata e aggiornata
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
# Aggiorna sulla base del nuovo ETP che dice che 𝛽1 è reale al livello 0
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
# Dismissione 𝛽0
ip route del 169.254.215.29 dev eth1 src 169.254.94.223
```

