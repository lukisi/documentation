# Proof of concept - Dettagli Operazioni - Pagina 7

[Operazione precedente](DettagliOperazioni6.md)

### <a name="Migrazione_ingresso_1"></a> Migrazione per ingresso - Caso 1

#### Comando prepare_migrate_phase_1 al sistema *𝛽*

Il programma **qspnclient** chiama il metodo `prepare_add_identity` del modulo Identities e memorizza le
informazioni relative alla migrazione.

#### Comando migrate_phase_1 al sistema *𝛽*

Il programma **qspnclient** recupera le informazioni relative alla migrazione e chiama il metodo `add_identity`
del modulo Identities. Il modulo Identities produce quindi queste operazioni:

**sistema 𝛽**
```
ip netns add migr01
ip netns exec migr01 sysctl net.ipv4.ip_forward=1
ip netns exec migr01 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev migr01_eth1 link eth1 type macvlan
ip link set dev migr01_eth1 netns migr01
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.rp_filter=0
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.arp_ignore=1
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.arp_announce=2
ip netns exec migr01 ip link set dev migr01_eth1 up
ip netns exec migr01 ip address add 169.254.27.218 dev migr01_eth1
ip netns exec migr01 ip route add 169.254.69.30 dev migr01_eth1 src 169.254.27.218
ip netns exec migr01 ip route add 169.254.94.223 dev migr01_eth1 src 169.254.27.218
ip netns exec migr01 ip route add 169.254.163.36 dev migr01_eth1 src 169.254.27.218
```

**sistema 𝛼**
```
ip route add 169.254.27.218 dev eth1 src 169.254.69.30
```

**sistema 𝛾**
```
ip route add 169.254.27.218 dev eth1 src 169.254.94.223
```

**sistema 𝜀**
```
ip route add 169.254.27.218 dev eth1 src 169.254.163.36
```

#### <a name="Spostamento_rotte_identita"></a> Spostamento delle rotte della vecchia identità

Il programma **qspnclient** decide quali tabelle di inoltro vanno usate nel nuovo network namespace.
In questo caso abbiamo l'arco-identità *𝛽<sub>1</sub>*-*𝛼<sub>1</sub>* che era associato ad un arco-qspn
e che mantiene inalterate le sue proprietà. Abbiamo l'arco-identità *𝛽<sub>1</sub>*-*𝛾<sub>0</sub>* che
era associato ad un arco-qspn (che era l'arco verso l'altro nodo in *𝜑*) e che mantiene inalterate le
sue proprietà. Abbiamo anche l'arco-identità *𝛽<sub>1</sub>*-*𝜀<sub>0</sub>* ma questo non era associato
ad un arco-qspn. Abbiamo infine l'arco-identità *𝛽<sub>1</sub>*-*𝜀<sub>1</sub>* che era associato
ad un arco-qspn (prodotto dal comando `add_qspn_arc` di poco fa) e che mantiene inalterate le
sue proprietà.

Il programma **qspnclient** calcola l'indirizzo della vecchia identità nel nuovo namespace. Questo
si calcola a partire dall'indirizzo Netsukuku precedente della vecchia identità, sostituendo al
livello *"livello g-nodo migrante"* la *"posizione di connettività"*. Quindi in questo caso
dall'indirizzo precedente di *𝛽<sub>1</sub>* che era 2·1·1·1 si passa al 2·1·1·3.

Il programma **qspnclient** calcola tutti i possibili indirizzi IP di destinazione, ognuno con suffisso CIDR,
(come riportato [qui](DettagliOperazioni1.md#computo_indirizzi_ip_destinazioni))
relativi all'indirizzo della vecchia identità nel nuovo namespace. Il programma li memorizza associandoli a
quella identità.

Inizialmente il programma aggiunge le rotte verso questi indirizzi IP nello stato `unreachable` in tutte le
tabelle di inoltro che vanno usate nel nuovo network namespace. Poi, per gli archi-qspn che la vecchia
identità già conosceva e dai quali aveva ricevuto un ETP, secondo le sue conoscenze aggiorna le rotte.

**sistema 𝛽**
```
ip netns exec migr01 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250
ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5

ip netns exec migr01 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249
ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA

ip netns exec migr01 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3C:14:33 -j MARK --set-mark 248
ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev migr01_eth1
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
ip netns exec migr01 ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change 10.0.0.22/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.86/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.62/32 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change blackhole 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip rule add fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
```


[Operazione seguente](DettagliOperazioni8.md)
