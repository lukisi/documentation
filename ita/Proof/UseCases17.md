# Proof of concept - Casi d'uso - Pagina 17

[Pagina precedente](UseCases16.md)

#### <a name="Ingresso_lambda_seconda_fase"></a>Ingresso di *𝜆* - Seconda fase

Ora la nuova identità nel sistema *𝜆*, che è in fase di bootstrap, riceve un ETP dal nodo *𝜀<sub>1</sub>* e
dal nodo *𝜀<sub>2</sub>*. Assumiamo anche che subito dopo essa ottiene il suo definitivo indirizzo reale 1·1·
(ricordiamo che è un gateway verso una sottorete autonoma che equivale ad un g-nodo di livello 1).

**sistema 𝜆**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.163.36 dev eth1
ip route change 10.0.0.80/30 table ntk via 169.254.163.36 dev eth1
ip route change 10.0.0.56/30 table ntk via 169.254.163.36 dev eth1
ip route change 10.0.0.20/31 table ntk via 169.254.241.153 dev eth1
ip route change 10.0.0.84/31 table ntk via 169.254.241.153 dev eth1
ip route change 10.0.0.60/31 table ntk via 169.254.241.153 dev eth1
ip route change 10.0.0.48/31 table ntk via 169.254.241.153 dev eth1
ip route change unreachable 10.0.0.22/31 table ntk
ip route change unreachable 10.0.0.86/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3B:9F:45
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.163.36 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.163.36 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.163.36 dev eth1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:3B:9F:45
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:3B:9F:45

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.241.153 dev eth1
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.241.153 dev eth1
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.241.153 dev eth1
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:3C:14:33
ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3B:9F:45 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:3B:9F:45

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3C:14:33 -j MARK --set-mark 249
ip rule add fwmark 249 table ntk_from_00:16:3E:3C:14:33

ip address add 10.0.0.22 dev eth1
ip address add 10.0.0.86 dev eth1
ip address add 10.0.0.62 dev eth1
ip address add 10.0.0.50 dev eth1

iptables -t nat -A PREROUTING -d 10.0.0.22/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.86/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.62/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.50/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.48/30 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.50/31
iptables -t nat -A POSTROUTING -d 10.0.0.56/29 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.62/31
iptables -t nat -A POSTROUTING -d 10.0.0.0/27  -s 10.0.0.40/31 -j NETMAP --to 10.0.0.22/31
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.22/31

iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.22

ip route del 10.0.0.22/31 table ntk
ip route del 10.0.0.86/31 table ntk
ip route del 10.0.0.62/31 table ntk
ip route del 10.0.0.50/31 table ntk
ip route del 10.0.0.22/31 table ntk_from_00:16:3E:3B:9F:45
ip route del 10.0.0.86/31 table ntk_from_00:16:3E:3B:9F:45
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:3B:9F:45
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:3B:9F:45
ip route del 10.0.0.22/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.86/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.22
ip route change 10.0.0.80/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.22
ip route change 10.0.0.56/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.62
ip route change 10.0.0.20/31 table ntk via 169.254.241.153 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk via 169.254.241.153 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk via 169.254.241.153 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk via 169.254.241.153 dev eth1 src 10.0.0.50
```

Uscita dalla fase di bootstrap, la nuova identità nel sistema *𝜆* trasmette un ETP completo ai suoi vicini.

**sistema 𝜀**
```
```

La propagazione di questo ETP ha ripercussioni sulle tabelle di tutti i nodi.

**sistema TODO**
```
```



**TODO**


[Pagina seguente](UseCases18.md)
