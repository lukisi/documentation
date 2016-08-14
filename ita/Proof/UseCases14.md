# Proof of concept - Casi d'uso - Pagina 14

[Pagina precedente](UseCases13.md)

### <a name="Ingresso_epsilon_seconda_fase"></a>Ingresso di *𝜀* - Seconda fase

Ora la nuova identità nel sistema *𝜀*, che è in fase di bootstrap, riceve un ETP dal nodo *𝛽<sub>1</sub>* e
dal nodo *𝛽<sub>2</sub>*. Assumiamo anche che subito dopo essa ottiene il suo definitivo indirizzo reale 1·1·1.

**sistema 𝜀**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.20/31 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.84/31 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.60/31 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.48/31 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.22/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.86/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.62/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.50/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.23/32 table ntk
ip route change unreachable 10.0.0.87/32 table ntk
ip route change unreachable 10.0.0.63/32 table ntk
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
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.22/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.86/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.62/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.22/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.86/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.62/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:EE:AF:D1

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
ip route del 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.23
ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.23
ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.63
ip route change 10.0.0.20/31 table ntk via 169.254.27.218 dev eth1 src 10.0.0.23
ip route change 10.0.0.84/31 table ntk via 169.254.27.218 dev eth1 src 10.0.0.23
ip route change 10.0.0.60/31 table ntk via 169.254.27.218 dev eth1 src 10.0.0.63
ip route change 10.0.0.48/31 table ntk via 169.254.27.218 dev eth1 src 10.0.0.51
ip route change 10.0.0.22/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.23
ip route change 10.0.0.86/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.23
ip route change 10.0.0.62/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.63
ip route change 10.0.0.50/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41
```

Uscita dalla fase di bootstrap, la nuova identità nel sistema *𝜀* trasmette un ETP completo ai suoi vicini.

**sistema 𝛽**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.19
ip route change 10.0.0.84/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.19
ip route change 10.0.0.60/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.59
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.82/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.58/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.59
ip route change 10.0.0.50/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.163.36 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.163.36 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.163.36 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.18/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.82/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.58/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3C:14:33 -j MARK --set-mark 248
ip rule add fwmark 248 table ntk_from_00:16:3E:3C:14:33
```

**Completare sistema 𝛽 migr01**

La propagazione di questo ETP... **TODO**

Infine, prima o poi, la vecchia identità del sistema *𝜀* viene dismessa.

**sistema 𝜀**
```
ip netns exec entr05 ip route flush table main
ip netns exec entr05 ip link delete entr05_eth1 type macvlan
ip netns del entr05
```

**sistema 𝛽**
```
ip route del 169.254.133.31 dev eth1 src 169.254.96.141
ip netns exec migr01 ip route del 169.254.133.31 dev migr01_eth1 src 169.254.27.218
```


[Pagina seguente](UseCases15.md)
