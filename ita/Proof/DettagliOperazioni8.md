# Proof of concept - Dettagli Operazioni - Pagina 8

[Operazione precedente](DettagliOperazioni7.md)

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

#### <a name="Migrazione_Spostamento_rotte_identita"></a> Migrazione: Spostamento delle rotte della vecchia identità

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
(come riportato [qui](DettagliOperazioni2.md#computo_indirizzi_ip_destinazioni))
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

#### Migrazione: Aggiornamento dei gateway che si sono spostati in un diverso namespace

In questa situazione notiamo che la preparazione del vecchio namespace per la nuova identità
viene rimandata a dopo che i vicini di *𝛽<sub>1</sub>* abbiano modificato le
rotte che prevedono di usare lui come gateway.  
Tali operazioni nei nodi vicini sono avviate automaticamente dalla duplicazione degli archi-identità
di *𝛽<sub>1</sub>* operata dal modulo Identities. Quindi al programma **qspnclient** nel sistema
*𝛽* è sufficiente attendere un adeguato lasso di tempo prima di procedere con le operazioni a seguito
del comando `migrate_phase_1`.

Nei sistemi dei nodi vicini abbiamo che il modulo Identities notifica il cambio delle proprietà peer-MAC
e peer-linklocal per un arco-identità. Se il programma **qspnclient** vede che a quell'arco-identità è
associato un arco-qspn allora esegue queste operazioni:

*   rimozione della vecchia tabella di inoltro.
    *   con rimozione della regola se da quell'arco-qspn era stato ricevuto almeno un ETP.
*   aggiunta della nuova tabella di inoltro con lo stesso set di possibili rotte di destinazione.
*   aggiornamento delle rotte su tutte le tabelle.
*   aggiunta della regola per la nuova tabella di inoltro se da quell'arco-qspn era stato ricevuto almeno un ETP.

**sistema 𝛼**
```
ip rule del fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
ip route flush table ntk_from_00:16:3E:EC:A3:E1
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
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
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

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
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1
ip rule add fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
```

**sistema 𝛾**
```
ip rule del fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
ip route flush table ntk_from_00:16:3E:EC:A3:E1
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.22
ip route change 10.0.0.80/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.22
ip route change 10.0.0.56/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.62
ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.50
ip route change 10.0.0.23/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.40

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1
ip rule add fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
```

**sistema 𝜀**
```
ip route flush table ntk_from_00:16:3E:EC:A3:E1
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1
```

#### Migrazione: Pulizia del vecchio namespace per la nuova identità

Poi il programma **qspnclient** rimuove dalle tabelle presenti nel vecchio network namespace le rotte
verso i possibili indirizzi IP di destinazione relativi all'indirizzo che la vecchia identità aveva nel
vecchio namespace e che ora non sono più validi.  
Cioè, relativamente a tutte le destinazioni, gli indirizzi IP globali e quelli interni ai g-nodi di livello
maggiore del livello del g-nodo che fa ingresso in blocco; nel nostro caso del livello 0, cioè tutti.

**sistema 𝛽**
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
ip route del 10.0.0.40/32 table ntk

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA

ip route del 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.22/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.86/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.62/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
```

Poi il programma **qspnclient**, solo se il vecchio namespace è il default (come nel nostro caso),
rimuove dal vecchio namespace gli indirizzi IP della vecchia identità che non saranno comuni
con quelli della nuova identità. Cioè quelli interni ai g-nodi di livello maggiore del
livello del nuovo g-nodo che si è costituito nella rete; nel nostro caso del livello 0.

**sistema 𝛽**
```
ip address del 10.0.0.41/32 dev eth1
ip address del 10.0.0.51/32 dev eth1
ip address del 10.0.0.63/32 dev eth1
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.23
ip address del 10.0.0.23/32 dev eth1
ip address del 10.0.0.87/32 dev eth1
```

Infine il programma **qspnclient** controlla se alcuni archi-qspn sono verso vicini che sono esterni
al g-nodo che ha migrato. Lo capisce dal fatto che il relativo arco-identità non ha cambiato le sue proprietà
peer-MAC e peer-linklocal. Se per tali archi-qspn era stata aggiunta la regola (cioè se avevano ricevuto
almeno un ETP) essa va rimossa in attesa di un nuovo ETP che andrà valutato in base al nuovo indirizzo
Netsukuku della nuova identità.

**sistema 𝛽**
```
ip rule del fwmark 250 table ntk_from_00:16:3E:5B:78:D5
ip rule del fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
```

#### Migrazione: Cambio di indirizzo della vecchia identità

Poi il programma **qspnclient**, sempre a seguito del comando `migrate_phase_1`, istruisce
il QspnManager sul cambio di indirizzo della vecchia identità. Nell'esempio corrente abbiamo
che tale identità assume un identificativo *virtuale* al livello 0.

Questo determina la produzione di un ETP che viene trasmesso dal sistema *𝛽* ai suoi vicini e che
interessa solo il sistema *𝛾*. Avremo quindi una serie di operazioni eseguite dal programma **qspnclient**
nel sistema *𝛾* a seguito della processazione di un ETP. Queste non vengono riportate qui in quanto sono
state trattate in precedenza.

[Operazione seguente](DettagliOperazioni9.md)
