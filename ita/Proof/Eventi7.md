# Proof of concept - Eventi - Pagina 7

[Pagina precedente](Eventi6.md)

## Migrazione di un singolo nodo

Nel trattare [questo](UseCases12.md) caso d'uso, abbiamo descritto la migrazione di *𝛽* (che aveva
indirizzo 1·1·1) dal g-nodo 1·1· al g-nodo 0·1· (con indirizzo 0·1·1).

Ricordiamo le informazioni salienti di questo ingresso.

**migr01**

Le operazioni **migr01** e **entr05** sono collegate.  
Il singolo nodo *𝛽<sub>1</sub>* aveva indirizzo 2·1·1·1 in *G<sub>𝛾</sub>*. Con questa operazione
di migrazione *𝛽<sub>1</sub>* assume indirizzo *di connettività* 2·1·1·3. Temporaneamente
*𝛽<sub>2</sub>* assume indirizzo *virtuale* 2·0·1·2. Dopo poco *𝛽<sub>2</sub>* assume
indirizzo 2·0·1·1.  
L'identità *𝛽<sub>1</sub>* non viene dismessa finché una verifica con `check_connectivity` non darà
esito positivo.

**entr05**

Il nodo *𝜀<sub>0</sub>* era da solo e aveva indirizzo 3·1·0·0 in *G<sub>𝜀</sub>*. Con questa operazione
di ingresso *𝜀<sub>0</sub>* assume indirizzo *di connettività* 3·1·0·2 in *G<sub>𝜀</sub>*. Temporaneamente
*𝜀<sub>1</sub>* assume indirizzo *virtuale* 2·1·1·2 in *G<sub>𝛾</sub>*. Dopo poco *𝜀<sub>1</sub>* assume
indirizzo 2·1·1·1 in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝜀<sub>0</sub>* viene dismesso.

----

Nella trattazione del caso d'uso avevamo accennato al fatto che i dialoghi tra
vicini per scegliere le nuove posizioni e coordinare le operazioni vengono fatte inizialmente
(dopo la realizzazione dell'arco tra *𝜀* e *𝛽*) tramite meccanismi estranei ai moduli di cui stiamo
trattando in questa proof-of-concept. Diciamo in dettaglio come questi dialoghi sono simulati nel
programma **qspnclient**.

L'utente nel sistema *𝜀* dà il comando `enter_net_phase1` dicendo al **qspnclient** che:
*   l'identità *𝜀<sub>0</sub>* deve produrre una copia (sia essa *𝜀<sub>1</sub>*) che faccia
    ingresso in *G<sub>𝛾</sub>* in 1·1·.
*   l'identità *𝜀<sub>1</sub>* deve prendere temporaneamente il virtuale 1·1·2.
*   è stata individuata una migration path nella quale l'ultimo passaggio ha un dato identificativo
    (che indichiamo con *m<sub>𝛽</sub>*) e libererà per *𝜀<sub>1</sub>* l'indirizzo 1·1·1.

L'utente nel sistema *𝛽* dà il comando `migrate` dicendo al **qspnclient** che:
*   l'identità *𝛽<sub>1</sub>* deve produrre una copia (sia essa *𝛽<sub>2</sub>*) che faccia
    migrazione in 0·1·.
*   l'identità *𝛽<sub>2</sub>* deve prendere temporaneamente il virtuale 0·1·2.
*   è immediatamente disponibile per *𝛽<sub>2</sub>* l'indirizzo 0·1·1.

L'utente nel sistema *𝜀* dà il comando `enter_net_phase2` dicendo al **qspnclient** che:
*   la migrazione *m<sub>𝛽</sub>* (di cui era in attesa *𝜀<sub>1</sub>*) è stata completata.

----

### Formazione dell'arco

**sistema 𝜀**
```
ip route add 169.254.96.141 dev eth1 src 169.254.163.36
```

**sistema 𝛽**
```
ip route add 169.254.163.36 dev eth1 src 169.254.96.141
```

Sequenza di operazioni eseguita dal modulo Neighborhood.

### entr05: Spostamento vecchia identità in nuovo network namespace

**sistema 𝜀**
```
ip netns add entr05
ip netns exec entr05 sysctl net.ipv4.ip_forward=1
ip netns exec entr05 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr05_eth1 link eth1 type macvlan
ip link set dev entr05_eth1 netns entr05
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.rp_filter=0
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.arp_ignore=1
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.arp_announce=2
ip netns exec entr05 ip link set dev entr05_eth1 up
ip netns exec entr05 ip address add 169.254.133.31 dev entr05_eth1
ip netns exec entr05 ip route add 169.254.96.141 dev entr05_eth1 src 169.254.133.31
```

**sistema 𝛽**
```
ip route add 169.254.133.31 dev eth1 src 169.254.96.141
```

Sequenza di operazioni eseguita dal modulo Identities su richiesta del programma **qspnclient** quando
l'utente dà il comando `enter_net_phase1`.

#### entr05: Copia tabelle e regole, spostamento rotte

**sistema 𝜀**
```
ip netns exec entr05 ip rule add table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.24/30 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.88/30 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.30/31 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.94/31 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.50/31 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.28/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.92/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.60/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.48/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.29/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.93/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.61/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.49/32 table ntk
ip netns exec entr05 ip route add unreachable 10.0.0.41/32 table ntk
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
ip route del 10.0.0.29/32 table ntk
ip route del 10.0.0.93/32 table ntk
ip route del 10.0.0.61/32 table ntk
ip route del 10.0.0.49/32 table ntk
ip route del 10.0.0.41/32 table ntk
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address del 10.0.0.28/32 dev eth1
ip address del 10.0.0.92/32 dev eth1
ip address del 10.0.0.60/32 dev eth1
ip address del 10.0.0.48/32 dev eth1
ip address del 10.0.0.40/32 dev eth1
ip netns exec entr05 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.24/30 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.88/30 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.56/30 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.30/31 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.94/31 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.50/31 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.28/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.92/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.60/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.48/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.40/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.29/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.93/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.61/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.49/32 table ntk
ip netns exec entr05 ip route change unreachable 10.0.0.41/32 table ntk
```

Seconda parte di operazioni eseguita dal programma **qspnclient** quando
l'utente dà il comando `enter_net_phase1`.

### entr05: Popolamento nuove rotte della nuova identità

**sistema 𝜀**
```
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.16/30 table ntk
ip route add unreachable 10.0.0.80/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.20/31 table ntk
ip route add unreachable 10.0.0.84/31 table ntk
ip route add unreachable 10.0.0.60/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.22/32 table ntk
ip route add unreachable 10.0.0.86/32 table ntk
ip route add unreachable 10.0.0.62/32 table ntk
ip route add unreachable 10.0.0.50/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
ip route add unreachable 10.0.0.23/32 table ntk
ip route add unreachable 10.0.0.87/32 table ntk
ip route add unreachable 10.0.0.63/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
```

Terza parte di operazioni eseguita dal programma **qspnclient** quando
l'utente dà il comando `enter_net_phase1`.

### entr05: Creazione e popolamento iniziale di tabelle per l'inoltro

**sistema 𝜀**
```
(echo; echo "250 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250
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

**sistema 𝛽**
```
(echo; echo "248 ntk_from_00:16:3E:3C:14:33 # xxx_table_ntk_from_00:16:3E:3C:14:33_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3C:14:33 -j MARK --set-mark 248
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
```

Operazioni eseguite dal programma **qspnclient** in occasione dell'aggiunta di un arco-identità.
Questa aggiunta avviene nella quarta parte di operazioni eseguita dal programma **qspnclient** quando
l'utente dà il comando `enter_net_phase1`.

### entr05: Dismissione identità

Nell'ultima parte di operazioni a seguito del comando `enter_net_phase1`,
il sistema *𝜀* verifica che la vecchia identità di connettività possa essere dismessa.

**sistema 𝜀**
```
ip netns exec entr05 ip route flush table main
ip netns exec entr05 ip link delete entr05_eth1 type macvlan
ip netns del entr05
```

**sistema 𝛽**
```
ip route del 169.254.133.31 dev eth1 src 169.254.96.141
ip netns exec migr01 ip route del 169.254.133.31 dev migr01_eth1 src 169.254.27.218
```

[Pagina seguente](Eventi8.md)
