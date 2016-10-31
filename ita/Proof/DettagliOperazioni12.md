# Proof of concept - Dettagli Operazioni - Pagina 12

[Operazione precedente](DettagliOperazioni11.md)

### <a name="Migrazione_ingresso_2"></a> Migrazione per ingresso - Caso 2

#### Comando prepare_migrate_phase_1 al sistema *𝜀*

Analogo a quanto visto nel caso 1.

#### Comando migrate_phase_1 al sistema *𝜀*

Creazione del nuovo network namespace: analogo a quanto visto nel caso 1.

#### Migrazione: Spostamento delle rotte della vecchia identità

Il programma **qspnclient** decide quali tabelle di inoltro vanno usate nel nuovo network namespace.
In questo caso abbiamo l'arco-identità *𝜀<sub>1</sub>*-*𝛽<sub>1</sub>* che era associato ad un arco-qspn
interno al g-nodo migrante *𝜑*, il quale vede modificarsi anche le sue proprietà (cioè il peer-MAC).
Poi abbiamo l'arco-identità *𝜀<sub>1</sub>*-*𝛽<sub>2</sub>* che
era associato ad un arco-qspn (che era l'arco verso l'altro nodo in *𝜔*) e che mantiene inalterate le
sue proprietà. Abbiamo anche l'arco-identità *𝜀<sub>1</sub>*-*𝜆<sub>0</sub>* ma questo non era associato
ad un arco-qspn. Abbiamo infine l'arco-identità *𝜀<sub>1</sub>*-*𝜆<sub>1</sub>* che era associato
ad un arco-qspn (prodotto dal comando `add_qspn_arc` di poco fa) e che mantiene inalterate le
sue proprietà.

Il comportamento è analogo a quanto visto nel caso 1, a parte il fatto che esiste un arco-qspn interno
al g-nodo migrante, il quale vede modificarsi il peer-MAC, quindi abbiamo l'aggiunta di una tabella
di inoltro.

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

[Operazione seguente](DettagliOperazioni13.md)
