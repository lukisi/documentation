# Proof of concept - Eventi - Pagina 10

[Pagina precedente](Eventi9.md)

### entr05: Processazione del ETP

Probabilmente, prima che l'utente dia il comando `enter_net_phase2`, il sistema *𝜀*
riceve e processa un ETP con cui esce dalla fase di bootstrap e abilita le tabelle di
inoltro.

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
ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1

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
ip rule add fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
```

### entr05: Cambio di indirizzo della nuova identità

**sistema 𝜀**
```
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

ip address add 10.0.0.41 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.63 dev eth1
ip address add 10.0.0.23 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.23
ip address add 10.0.0.87 dev eth1

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

Operazioni eseguite dal programma **qspnclient** quando l'utente dà il comando `enter_net_phase2`.

[Pagina seguente](Eventi11.md)
