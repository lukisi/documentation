# Proof of concept - Eventi - Pagina 12

[Pagina precedente](Eventi11.md)

Quando l'utente dà il comando `prepare_migrate` nei sistemi *𝜀*, *𝛽* e *𝛾* il programma **qspnclient**
non esegue alcuna operazione nel S.O. ma solo memorizza le informazioni passate. Le operazioni che vedremo
in seguito sono avviate quando l'utente inizia a dare i comandi `migrate`.

Prendiamo in esame cosa avviene quando l'utente dà il comando `migrate` nel sistema *𝜀*, tralasciando di ripeterci
con gli altri sistemi del g-nodo che migra.  
Teniamo però in considerazione che il comando `migrate` deve essere dato in tempi rapidi anche agli altri sistemi
del g-nodo, pena la perdita di qualche arco-identità.

### migr02: Spostamento vecchia identità in nuovo network namespace

**sistema 𝜀**
```
ip netns add migr02
ip netns exec migr02 sysctl net.ipv4.ip_forward=1
ip netns exec migr02 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev migr02_eth1 link eth1 type macvlan
ip link set dev migr02_eth1 netns migr02
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
...
ip netns exec migr02 ip route add 169.254.241.153 dev migr02_eth1 src 169.254.42.4
```

**sistema 𝜆**
```
ip route add 169.254.241.153 dev eth1 src 169.254.109.22
ip netns exec entr06 ip route add 169.254.241.153 dev eth1 src 169.254.34.45
```

**sistema 𝛽**
```
ip route add 169.254.241.153 dev eth1 src 169.254.96.141
```

#### migr02: Copia tabelle e regole, spostamento rotte

**sistema 𝜀**
```
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
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

ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:06:3E:90 -j MARK --set-mark 248
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

(echo; echo "247 ntk_from_00:16:3E:BD:34:98 # xxx_table_ntk_from_00:16:3E:BD:34:98_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:BD:34:98 -j MARK --set-mark 247
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
ip netns exec migr02 ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1

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
ip netns exec migr02 ip rule add fwmark 248 table ntk_from_00:16:3E:06:3E:90

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
ip netns exec migr02 ip rule add fwmark 247 table ntk_from_00:16:3E:BD:34:98
```

#### migr02: Aggiornamento dei gateway che si sono spostati in un diverso namespace

Ora il programma *qspnclient* nel sistema *𝜀* lascia il tempo ai sistemi vicini esterni a *𝜑* di modificare le
rotte che prevedono di usare *𝜀<sub>old</sub>* come gateway.  
Dopo prosegue con la pulizia del vecchio namespace per la nuova identità.

#### migr02: Pulizia del vecchio namespace per la nuova identità

**sistema 𝜀**
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

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:EE:AF:D1

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:EC:A3:E1

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:06:3E:90

iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.23
ip address del 10.0.0.23/32 dev eth1
ip address del 10.0.0.87/32 dev eth1
ip address del 10.0.0.63/32 dev eth1
ip address del 10.0.0.51/32 dev eth1
```

Siccome il vicino *𝛽<sub>default</sub>* è esterno al g-nodo *𝜑* che ha migrato, dal vecchio network namespace
della vecchia identità va rimossa anche la regola per la tabella di inoltro relativa.
Infatti la nuova identità dovrà attendere un ETP da questo arco prima di poter aggiornare la tabella.

**sistema 𝜀**
```
ip rule del fwmark 249 table ntk_from_00:16:3E:EC:A3:E1
```

### migr02: Popolamento nuove rotte della nuova identità

**sistema 𝜀**
```
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.20/30 table ntk
ip route add unreachable 10.0.0.84/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.16/31 table ntk
ip route add unreachable 10.0.0.80/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.18/31 table ntk
ip route add unreachable 10.0.0.82/31 table ntk
ip route add unreachable 10.0.0.58/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
```

### migr02: Creazione e popolamento iniziale di tabelle per l'inoltro

**sistema 𝜀**
```
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:06:3E:90

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
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1
```

### migr02: Processazione del ETP

Anche qui ipotiziamo il caso che il sistema *𝜀* con la sua nuova identità riceva e
processi un ETP ed esca dalla fase di bootstrap prima di cambiare il suo indirizzo nel nuovo g-nodo
in cui è migrato.

**sistema 𝜀**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.84/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.60/30 table ntk via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1
ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EE:AF:D1
```

### migr02: Rimozione archi esterni

**sistema 𝜀**
```
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
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98

ip netns exec migr02 ip rule del fwmark 249 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route flush table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249

ip netns exec migr02 ip route del 169.254.96.141 dev migr02_eth1 src 169.254.241.153
```

La rimozione degli archi produce nuovi ETP che vengono propagati e ritornano pure su *𝜀*:

**sistema 𝜀**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.84/30 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.60/30 table ntk via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1

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
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EE:AF:D1
```

### migr02: Cambio di indirizzo della nuova identità

**sistema 𝜀**
```
ip route del 10.0.0.16/31 table ntk
ip route del 10.0.0.80/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90

ip route add unreachable 10.0.0.16/32 table ntk
ip route add unreachable 10.0.0.80/32 table ntk
ip route add unreachable 10.0.0.56/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route change 10.0.0.16/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.80/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.56/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.48/32 table ntk via 169.254.27.218 dev eth1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.16/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.80/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.56/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EE:AF:D1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:06:3E:90

ip address add 10.0.0.17 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.17
ip address add 10.0.0.81 dev eth1
ip address add 10.0.0.57 dev eth1
ip address add 10.0.0.49 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.84/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.60/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.57
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.17
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.17
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.57
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.49
ip route change 10.0.0.16/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.80/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.56/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.57
ip route change 10.0.0.48/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41
```

[Pagina seguente](Eventi13.md)
