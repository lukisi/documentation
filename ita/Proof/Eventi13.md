# Proof of concept - Eventi - Pagina 13

[Pagina precedente](Eventi12.md)

### entr06: Processazione del ETP

Probabilmente, prima che l'utente dia il comando `enter_net_phase2`, il sistema *𝜆*
riceve e processa un ETP con cui esce dalla fase di bootstrap e abilita le tabelle di
inoltro.

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
ip rule add fwmark 250 table ntk_from_00:16:3E:3B:9F:45

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
ip rule add fwmark 249 table ntk_from_00:16:3E:3C:14:33
```

### entr06: Cambio di indirizzo della nuova identità

**sistema 𝜆**
```
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

# il nuovo indirizzo è valido almeno dentro il g-nodo di livello 2
ip address add 10.0.0.50 dev eth1
iptables -t nat -A PREROUTING -d 10.0.0.50/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.48/30 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.50/31

# il nuovo indirizzo è valido almeno dentro il g-nodo di livello 3
ip address add 10.0.0.62 dev eth1
iptables -t nat -A PREROUTING -d 10.0.0.62/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.56/29 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.62/31

# il nuovo indirizzo è valido in tutta la rete
ip address add 10.0.0.22 dev eth1
iptables -t nat -A PREROUTING -d 10.0.0.22/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.0/27  -s 10.0.0.40/31 -j NETMAP --to 10.0.0.22/31

# il gateway accetta di essere un relay anonimizzante
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.22

# il gateway accetta di essere contattato in forma anonima
ip address add 10.0.0.86 dev eth1

# ogni sistema nella sottorete autonoma accetta di essere contattato in forma anonima
iptables -t nat -A PREROUTING -d 10.0.0.86/31 -j NETMAP --to 10.0.0.40/31

# il gateway permette ai sistemi interni di contattare un sistema esterno in forma anonima
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.22/31

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

Operazioni eseguite dal programma **qspnclient** quando l'utente dà il comando `enter_net_phase2`.


[Pagina seguente](Eventi14.md)
