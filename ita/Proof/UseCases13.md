# Proof of concept - Casi d'uso - Pagina 13

[Pagina precedente](UseCases12.md)

#### <a name="Costituzione_beta2"></a>Costituzione *𝛽<sub>2</sub>*

Ora la nuova identità *𝛽<sub>2</sub>* prende possesso del precedente network namespace assumendo un
indirizzo virtuale nel g-nodo destinazione della migrazione, nel nostro caso 0·1·2. Seguiamo i suoi
passi fino alla produzione del primo ETP.

Nel sistema *𝛽* il modulo QSPN, relativamente alla nuova identità *𝛽<sub>2</sub>*, ha archi verso
i sistemi *𝛼* e *𝛾* e *𝜀*. Per tutti questi aspettiamo di avere un ETP da loro prima di usare le relative tabelle
`ntk_from_xxx`.

**sistema 𝛽**
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
ip route add unreachable 10.0.0.18/32 table ntk
ip route add unreachable 10.0.0.82/32 table ntk
ip route add unreachable 10.0.0.58/32 table ntk
ip route add unreachable 10.0.0.50/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
ip route add unreachable 10.0.0.19/32 table ntk
ip route add unreachable 10.0.0.83/32 table ntk
ip route add unreachable 10.0.0.59/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.18/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.82/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.58/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.18/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.82/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.58/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.18/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.82/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.58/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33
```

**sistema 𝛾**
```
(echo; echo "248 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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
```

**sistema 𝛼**
```
(echo; echo "249 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

**sistema 𝜀**
```
(echo; echo "249 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

Ora l'identità *𝛽<sub>2</sub>* riceve un ETP da *𝛼*, che fa parte del suo g-nodo in cui ha
migrato. Questo è sufficiente a fargli popolare la mappa e completare la fase
di bootstrap; potrebbe quindi già aggiornare le rotte che poi subirebbero un nuovo aggiornamento
dopo alla ricezione dell'ETP da *𝛾*. Per rapidità, assumiamo che riceve subito anche un ETP
da *𝛾* e popola le rotte nelle tabelle.

**sistema 𝛽**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.84/30 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.60/30 table ntk via 169.254.94.223 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/32 table ntk via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk via 169.254.69.30 dev eth1
ip route change 10.0.0.50/32 table ntk via 169.254.69.30 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.69.30 dev eth1
ip route change unreachable 10.0.0.19/32 table ntk
ip route change unreachable 10.0.0.83/32 table ntk
ip route change unreachable 10.0.0.59/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk

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
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:5B:78:D5
ip route change blackhole 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.18/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.82/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.58/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249
ip rule add fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
```

Poi l'identità *𝛽<sub>2</sub>* prepara e trasmette un ETP completo a *𝛼* e *𝛾* (lo trasmette anche
a *𝜀*, ma questi lo ignora perché non ha ancora completato il bootstrap). Ora essi sanno come
gestire i pacchetti provenienti da questo arco e quindi abilitano le relative tabelle.

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.50
ip route change unreachable 10.0.0.23/32 table ntk
ip route change unreachable 10.0.0.87/32 table ntk
ip route change unreachable 10.0.0.63/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

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
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 248
ip rule add fwmark 248 table ntk_from_00:16:3E:EC:A3:E1
```

**sistema 𝛼**
```
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
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change blackhole 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1
```

La propagazione dell'ETP trasmesso da *𝛽<sub>2</sub>* apporta ai vari nodi queste nuove conoscenze:

*   Il nodo *𝛼* sa di poter raggiungere il g-nodo 1· passando per il gateway *𝛽<sub>2</sub>*.  
    Non cambia il miglior percorso verso quella destinazione.
*   Il nodo *𝛼* sa di poter raggiungere il nodo 0·1·2 passando per il gateway *𝛽<sub>2</sub>*.  
    La destinazione è virtuale, quindi non compare nelle tabelle.
*   Il nodo *𝛾* sa di poter raggiungere il g-nodo 0· passando per il gateway *𝛽<sub>2</sub>*.  
    Cambia il miglior percorso verso quella destinazione. Abbiamo riportato questo cambiamento
    qualche riga sopra.
*   Il nodo *𝛽<sub>1</sub>* sa di poter raggiungere il g-nodo 0· passando per il gateway *𝛾*.  
    Non cambia il miglior percorso verso quella destinazione.

Per via di queste variazioni sui vari sistemi si eseguiranno i soliti comandi di cambio delle rotte
più attuali. Non li riportiamo perché non ci sono reali variazioni rispetto a prima.

#### <a name="Rimozione_arco_beta1_alfa"></a>Rimozione arco *𝛽<sub>1</sub>* - *𝛼*

Dopo qualche istante dalla trasmissione del primo ETP da parte di *𝛽<sub>2</sub>*, il nodo di
connettività *𝛽<sub>1</sub>* rimuove il suo arco verso *𝛼* poiché è esterno ai g-nodi di cui
supporta la connettività.

In ognuno dei due sistemi (il sistema *𝛼* per la sua identità principale e il sistema *𝛽*
per l'identitità *𝛽<sub>1</sub>*) dopo aver rimosso l'arco nel modulo QSPN e aver ricalcolato
i migliori percorsi verso le possibili destinazioni le operazioni che il programma *qspnclient*
fa sono:

*   cambio rotte nelle tabelle
*   rimozione tabella dell'arco (`ntk_from_xxx`) e relativa regola e marcamento
*   rimozione arco (realizzata dal modulo Identities)

**sistema 𝛼**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.84/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.60/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.58
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.19/32 table ntk
ip route change unreachable 10.0.0.83/32 table ntk
ip route change unreachable 10.0.0.59/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1

ip rule del fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
ip route flush table ntk_from_00:16:3E:EE:AF:D1
sed -i '/xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx/d' /etc/iproute2/rt_tables
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250

ip route del 169.254.27.218 dev eth1 src 169.254.69.30
```

**sistema 𝛽**
```
ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec migr01 ip route change 10.0.0.16/30 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.80/30 table ntk via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.56/30 table ntk via 169.254.94.223 dev migr01_eth1
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
ip netns exec migr01 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
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

ip netns exec migr01 ip rule del fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route flush table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249

ip netns exec migr01 ip route del 169.254.69.30 dev migr01_eth1 src 169.254.27.218
```

Ora il nodo *𝛽<sub>1</sub>*, avendo rimosso un suo arco, comunica le variazioni apportate alla sua mappa tramite
un ETP agli altri vicini. Quindi il nodo 𝛾, al ricevere tale ETP, sa che non può più raggiungere il g-nodo 0· passando
per il gateway *𝛽<sub>1</sub>*. Però abbiamo detto prima che aveva appreso di poterlo fare passando per il
gateway *𝛽<sub>2</sub>* e aveva anche già aggiornato le rotte essendo questo un nuovo miglior percorso.

Nel complesso quindi, la propagazione di questo ETP potrebbe portare a nuove esecuzioni di comandi di
aggiornamento delle rotte nei vari sistemi, ma non li riportiamo qui perché non ci sono variazioni.

#### <a name="Cambio_indirizzo_beta2"></a>Cambio indirizzo per *𝛽<sub>2</sub>*

Ora la nuova identità *𝛽<sub>2</sub>* può assumere un indirizzo reale nel g-nodo destinazione della migrazione,
nel nostro caso 0·1·1.

Come conseguenza il programma *qspnclient* nel sistema *𝛽* aggiunge gli indirizzi IP che prima non aveva
e rimuove gli stessi quali possibili destinazioni da tutte le tabelle. Inoltre aggiorna tutte le rotte nella
tabella per i pacchetti originati localmente (`ntk`) di modo da usare i nuovi indirizzi come *src*.

**sistema 𝛽**
```
ip address add 10.0.0.19 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.19
ip address add 10.0.0.83 dev eth1
ip address add 10.0.0.59 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.41 dev eth1

ip route del 10.0.0.19/32 table ntk
ip route del 10.0.0.83/32 table ntk
ip route del 10.0.0.59/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk

ip route del 10.0.0.19/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.83/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.59/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5

ip route del 10.0.0.19/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.83/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.59/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA

ip route del 10.0.0.19/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.83/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.59/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.19
ip route change 10.0.0.84/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.19
ip route change 10.0.0.60/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.59
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.82/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.58/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.59
ip route change 10.0.0.50/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.41
```

La nuova identità *𝛽<sub>2</sub>* comunica via ETP ai suoi vicini *𝛼* e *𝛾* che non è più possibile
raggiungere tramite lui l'indirizzo 0·1·2; ma che ora è possibile raggiungere tramite lui l'indirizzo 0·1·1.

L'effetto di questo ETP, che contiene solo modifiche a percorsi verso singoli nodi, è comunque limitato ai
nodi del g-nodo di livello 1, cioè al solo *𝛼*.

**sistema 𝛼**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.84/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.60/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.58
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.19/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.83/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.59/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.58
ip route change 10.0.0.51/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.40

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

[Pagina seguente](UseCases14.md)
