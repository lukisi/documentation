# Proof of concept - Eventi - Pagina 5

[Pagina precedente](Eventi4.md)

### Processazione degli ETP

Di nuovo ci troviamo davanti alle operazioni eseguite dal programma **qspnclient** quando un suo
QspnManager ha terminato di processare un ETP. In questo caso abbiamo un ETP trasmesso da *𝛾* a *𝛿*, con
il quale *𝛿* esce dalla fase di bootstrap. Poi uno trasmesso da *𝛿* a *𝜇*. Poi uno da *𝛿* a *𝛾*.
Infine uno da *𝛾* a *𝛽*.

**sistema 𝛿**
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
ip rule add fwmark 248 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
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

**sistema 𝛾**
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
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
ip rule add fwmark 249 table ntk_from_00:16:3E:1A:C4:45
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

Le operazioni sono quelle esaminate in precedenza: relativamente al network namespace
associato all'identità a cui il QspnManager appartiene, vengono aggiornate tutte le rotte
della tabella `ntk` e di quelle tabelle di inoltro dai cui archi abbiamo ricevuto almeno un
ETP; in seguito viene aggiunta la regola per le tabelle di inoltro il cui arco ha ricevuto
proprio adesso il primo ETP.

### Dismissione identità

Infine, come sesta parte di operazioni a seguito del comando di completare le operazioni di migrazione/ingresso,
uno dei sistemi nel g-nodo di connettività *𝜒* verifica che queste identità di
connettività possono essere dismesse. Sull'esito positivo il programma **qspnclient** istruisce il
modulo Identities e propaga l'ordine di dismettere le identità.

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

[Pagina seguente](Eventi6.md)
