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

[Pagina seguente](Eventi4.md)
