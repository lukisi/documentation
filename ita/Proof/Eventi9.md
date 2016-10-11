# Proof of concept - Eventi - Pagina 9

[Pagina precedente](Eventi8.md)

### migr01: Popolamento nuove rotte della nuova identità

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
```

Terza parte di operazioni eseguita dal programma **qspnclient** quando
l'utente dà il comando `migrate`.

### migr01: Creazione e popolamento iniziale di tabelle per l'inoltro

**sistema 𝛽**
```
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 248
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
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

Operazioni eseguite dal programma **qspnclient** in occasione dell'aggiunta di un arco-identità.
Questa aggiunta avviene nella quarta parte di operazioni eseguita dal programma **qspnclient** quando
l'utente dà il comando `migrate`.

Notiamo che in questo caso il QspnManager della nuova identità nel sistema *𝛽* riceve degli archi-identità
che erano già noti alla vecchia identità; per questo le relative tabelle di inoltro hanno
già il loro pseudonimo (nel file `rt_tables`) e nel network namespace il kernel è già stato
istruito di marcare i relativi pacchetti IP (`iptables ... set-mark`).

In generale: sia *x* un sistema in cui l'identità *x0* opera nel network namespace *n0*. Sia
*g* un g-nodo di livello *i* di cui fa parte *x0*. Sia *a0* un arco-identità tra *x0* e *y*,
dove *y* è una identità (di un altro sistema) che appartiene a *g*. Sia *a1* un arco-identità tra *x0* e *z*,
dove *z* è una identità (di un altro sistema) che non appartiene a *g*.

Supponiamo che ci sia una migrazione o un ingresso in una diversa rete del g-nodo *g* e che
nel sistema *x* questo comporti che l'identità *x0* si sposta nel network namespace *n1* mentre la
nuova identità *x1* prende possesso di *n0*.

*   Nel caso di ingresso in diversa rete:
    *   In *n1* ci sarà una copia di *a0* che ha un diverso peer-mac e peer-linklocal.
    *   In *n0* nessuna variazione viene apportata per *a0*.
    *   In *n1* ci sarà una copia di *a1* che ha lo stesso peer-mac e peer-linklocal.
    *   In *n0* viene rimossa la tabella-inoltro che si riferiva a *a1*, perché *z* non è nella stessa rete di *x1*.
    *   In *n0* possono venire aggiunti nuovi archi-identità verso nodi della nuova rete.
*   Nel caso di migrazione:
    *   In *n1* ci sarà una copia di *a0* che ha un diverso peer-mac e peer-linklocal.
    *   In *n0* nessuna variazione viene apportata per *a0*.
    *   In *n1* ci sarà una copia di *a1* che ha lo stesso peer-mac e peer-linklocal.
    *   In *n0* nessuna variazione viene apportata per *a1*.

### migr01: Processazione del ETP

In questo esempio abbiamo ipotizzato il caso che il sistema *𝛽* con la sua nuova identità riceva e
processi un ETP da *𝛼* e *𝛾* prima ancora di apprestarsi a cambiare il suo indirizzo nel nuovo g-nodo
in cui è migrato. Evento in realtà molto improbabile quando esista fin da subito un indirizzo Netsukuku
*reale* libero compatibile con gli archi del g-nodo che migra.

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
ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5

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
ip rule add fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
```

### migr01: Cambio di indirizzo della nuova identità

**sistema 𝛽**
```
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

ip address add 10.0.0.41 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.59 dev eth1
ip address add 10.0.0.19 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.19
ip address add 10.0.0.83 dev eth1

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

Quinta parte di operazioni eseguita dal programma **qspnclient** quando l'utente dà il comando `migrate`
nel caso in cui esista fin da subito un indirizzo Netsukuku *reale* libero compatibile con gli archi del g-nodo che migra.

### migr01: Rimozione archi esterni

Dopo qualche istante dalla trasmissione del primo ETP da parte della nuova identità di *𝛽*, il nodo di
connettività che è la vecchia identitià di *𝛽* rimuove il suo arco verso *𝛼* poiché è esterno ai g-nodi di cui
supporta la connettività.

Nei casi di rimozione di un arco-identità, dopo aver rimosso l'arco nel modulo QSPN e aver ricalcolato
i migliori percorsi verso le possibili destinazioni le operazioni che il programma **qspnclient**
fa sono:

*   cambio rotte nelle tabelle
*   rimozione tabella dell'arco (`ntk_from_xxx`) e relativa regola e marcamento
*   rimozione arco (realizzata dal modulo Identities)

**sistema 𝛽**
```
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

ip netns exec migr01 ip rule del fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route flush table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249

ip netns exec migr01 ip route del 169.254.69.30 dev migr01_eth1 src 169.254.27.218
```

[Pagina seguente](Eventi10.md)
