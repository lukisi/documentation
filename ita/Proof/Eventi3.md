# Proof of concept - Eventi - Pagina 3

[Pagina precedente](Eventi2.md)

## Ingresso di un g-nodo in una diversa rete

Prendiamo ad esempio i valori dell'ingresso di *𝜒* = *𝜇* + *𝛿* in *G<sub>𝛾</sub>*.

Questo ingresso avviene per via di un nuovo arco rilevato dal modulo Neighborhood tra i sistemi
*𝛿* e *𝛾*.

Ricordiamo le informazioni salienti di questo ingresso

**entr03**

Il g-nodo *𝜒* di livello 1 e di indirizzo Netsukuku 3·1·0·, che comprende *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>*,
costituiva l'intera rete *G<sub>𝛿</sub>*. Con questa operazione di ingresso si forma il g-nodo isomorfo
*𝜒'* costituito dalle nuove identità *𝛿<sub>1</sub>* e *𝜇<sub>2</sub>*. Il g-nodo
*𝜒* assume indirizzo *di connettività* 3·1·2· in *G<sub>𝛿</sub>*. Temporaneamente
*𝜒'* assume indirizzo *virtuale* 2·1·2· in *G<sub>𝛾</sub>*. Dopo poco *𝜒'* assume
indirizzo 2·1·0· in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝜒* viene dismesso.

**sistema 𝛾**
```
eth1         00:16:3E:5B:78:D5   169.254.94.223
```

**sistema 𝛿**
```
eth1         00:16:3E:1A:C4:45   169.254.253.216
entr03_eth1  00:16:3E:B9:77:80   169.254.83.167
```

**sistema 𝜇**
```
eth1         00:16:3E:2D:8D:DE   169.254.119.176
entr03_eth1  00:16:3E:DF:23:F5   169.254.242.91
```

### Formazione dell'arco

**sistema 𝛿**
```
ip route add 169.254.94.223 dev eth1 src 169.254.253.216
```

**sistema 𝛾**
```
ip route add 169.254.253.216 dev eth1 src 169.254.94.223
```

Abbiamo già esaminato il momento in cui si esegue questa sequenza di operazioni.

### Spostamento vecchia identità in nuovo network namespace

Ricordiamo cosa avviene nella rete Netsukuku in questo momento: attraverso un meccanismo di coordinamento nel
g-nodo *𝜒* si decide che questo g-nodo farà ingresso in *G<sub>𝛾</sub>*; quindi viene incaricato *𝛿* di dialogare
con *𝛾* per ottenere un posto per *𝜒* in *G<sub>𝛾</sub>*. Tutte queste operazioni, nel caso del programma **qspnclient**,
sono in realtà simulate dall'utente.

A questo punto nella rete Netsukuku avremmo che *𝛿* si occupa di propagare in tutto *𝜒* le informazioni riguardanti questo
ingresso. Questa propagazione, nel caso del programma **qspnclient**, avviene perché l'utente comanda nei vari
sistemi di *𝜒* (cioè *𝜇* + *𝛿*) di prepararsi ad una migrazione. A questo comando il programma **qspnclient** esegue
il metodo `prepare_add_identity` nel modulo Identities, passando l'identificativo di migrazione che l'utente ha scelto
e fornito. Nel dare il comando l'utente fornisce anche altre informazioni che il programma **qspnclient** mantiene in memoria.

Soltanto a questo punto nella rete Netsukuku si avviano le altre operazioni che ora vedremo.

**sistema 𝛿**
```
ip netns add entr03
ip netns exec entr03 sysctl net.ipv4.ip_forward=1
ip netns exec entr03 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.rp_filter=0
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_ignore=1
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_announce=2
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.83.167 dev entr03_eth1
ip netns exec entr03 ip route add 169.254.242.91 dev entr03_eth1 src 169.254.83.167
ip netns exec entr03 ip route add 169.254.94.223 dev entr03_eth1 src 169.254.83.167
```

**sistema 𝜇**
```
ip netns add entr03
ip netns exec entr03 sysctl net.ipv4.ip_forward=1
ip netns exec entr03 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.rp_filter=0
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_ignore=1
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_announce=2
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.242.91 dev entr03_eth1
ip netns exec entr03 ip route add 169.254.83.167 dev entr03_eth1 src 169.254.242.91
```

**sistema 𝛾**
```
ip route add 169.254.83.167 dev eth1 src 169.254.94.223
```

Questa sequenza di operazioni è eseguita dal modulo Identities su richiesta del programma **qspnclient** quando
l'utente comanda (nel sistema *𝜇* e nel sistema *𝛿* in qualsiasi ordine) di completare le operazioni di
migrazione/ingresso prima indicate. In questa prima parte di operazioni
si crea il nuovo network namespace in cui si trasferirà ad operare (per breve tempo) la vecchia
identità.

#### Copia tabelle e regole, spostamento rotte

**sistema 𝛿**
```
(echo; echo "249 ntk_from_00:16:3E:DF:23:F5 # xxx_table_ntk_from_00:16:3E:DF:23:F5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:DF:23:F5 -j MARK --set-mark 249
ip netns exec entr03 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5
ip route del 10.0.0.28/32 table ntk
ip route del 10.0.0.92/32 table ntk
ip route del 10.0.0.60/32 table ntk
ip route del 10.0.0.48/32 table ntk
ip route del 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:DF:23:F5
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.8/29 table ntk
ip route del 10.0.0.72/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/30 table ntk
ip route del 10.0.0.88/30 table ntk
ip route del 10.0.0.56/30 table ntk
ip route del 10.0.0.30/31 table ntk
ip route del 10.0.0.94/31 table ntk
ip route del 10.0.0.62/31 table ntk
ip route del 10.0.0.50/31 table ntk
ip route del 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.29
ip address del 10.0.0.29/32 dev eth1
ip address del 10.0.0.93/32 dev eth1
ip address del 10.0.0.61/32 dev eth1
ip address del 10.0.0.49/32 dev eth1
ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:DF:23:F5
```

**sistema 𝜇**
```
(echo; echo "249 ntk_from_00:16:3E:B9:77:80 # xxx_table_ntk_from_00:16:3E:B9:77:80_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:B9:77:80 -j MARK --set-mark 249
ip netns exec entr03 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:B9:77:80
ip route del 10.0.0.29/32 table ntk
ip route del 10.0.0.93/32 table ntk
ip route del 10.0.0.61/32 table ntk
ip route del 10.0.0.49/32 table ntk
ip route del 10.0.0.29/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.93/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:B9:77:80
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.8/29 table ntk
ip route del 10.0.0.72/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/30 table ntk
ip route del 10.0.0.88/30 table ntk
ip route del 10.0.0.56/30 table ntk
ip route del 10.0.0.30/31 table ntk
ip route del 10.0.0.94/31 table ntk
ip route del 10.0.0.62/31 table ntk
ip route del 10.0.0.50/31 table ntk
ip route del 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.16/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.80/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.24/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.88/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.30/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.94/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address del 10.0.0.28/32 dev eth1
ip address del 10.0.0.92/32 dev eth1
ip address del 10.0.0.60/32 dev eth1
ip address del 10.0.0.48/32 dev eth1
ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:B9:77:80
```

Questa sequenza di operazioni è eseguita dal programma **qspnclient**, sempre quando
l'utente comanda di completare le operazioni di migrazione/ingresso prima indicate, in seguito alle
operazioni viste prima. In questa seconda parte di operazioni la vecchia identità replica
nel nuovo network namespace la situazione che era prima nel suo vecchio network namespace
e prepara il vecchio network namespace che dovrà ospitare la nuova identità.

Trattandosi di un g-nodo di livello 1, rispetto a prima nel caso di ingresso di un singolo nodo, notiamo che
vengono fatte queste operazioni:

*   Vengono aggiunte nel namespace nuovo le marcature per i pacchetti provenienti dagli archi-qspn con il
    codice delle relative tabelle di inoltro (`ntk_from_xxx`).  
    Le tabelle di inoltro riferite ad altri nodi appartenenti al g-nodo che migra cambiano nome: ad esempio in *𝛿*
    la tabella di inoltro che prima era `ntk_from_00:16:3E:2D:8D:DE` diventa `ntk_from_00:16:3E:DF:23:F5`
    nel network namespace `entr03`. Infatti l'arco-identità con *𝜇*, che ha partecipato anch'esso alla
    migrazione, cambia il suo peer-mac. La stessa cosa accade in *𝜇* per la tabella di inoltro
    dei pacchetti provenienti dal MAC di *𝛿*.
*   Nello spostare le rotte si parte da quelle che hanno per destinazione g-nodi di livello inferiore al livello del
    g-nodo che migra, in questo caso solo quelle di livello 0. Gli indirizzi IP interni a un livello minore o uguale
    al livello del g-nodo che migra (in questo caso solo il livello 1) vengono mantenuti nel namespace vecchio e aggiunti
    (inizialmente come *irraggiungibili*) nel namespace nuovo. Gli altri indirizzi IP (interni, globale, anonimizzante)
    vengono rimossi dal namespace vecchio.
*   Poi si passa a quelle che hanno per destinazione g-nodi di livello maggiore o uguale al livello del
    g-nodo che migra. Per queste, come visto nel caso di ingresso di un singolo nodo, tutti gli indirizzi IP
    vengono aggiunti nelle tabelle del namespace nuovo (considerando che per il cambio di indirizzo ora si ha una
    destinazione in più al livello 1) e rimossi dalle tabelle del namespace vecchio.
*   Poi, se il vecchio namespace era il default, si rimuove da esso la (eventuale) regola di source-natting e
    gli indirizzi IP propri tranne quelli interni a un livello minore o uguale al livello del g-nodo che migra.
*   Infine, per le tabelle di inoltro riferite ad altri nodi appartenenti al g-nodo che migra, senza attendere
    da questi archi un nuovo ETP, si aggiornano nel namespace nuovo in tali tabelle tutte le rotte e si aggiunge
    la relativa regola.  
    Invece, per le tabelle di inoltro riferite ad altri nodi non appartenenti al g-nodo che migra, dovremo
    attendere un nuovo ETP da questi archi prima di aggiornare le rotte e aggiungere le regole.

[Pagina seguente](Eventi4.md)
