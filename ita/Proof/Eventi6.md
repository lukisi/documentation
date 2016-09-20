# Proof of concept - Eventi - Pagina 6

[Pagina precedente](Eventi5.md)

## Ingresso di un singolo nodo come nuovo g-nodo di livello 2.

In generale un g-nodo di livello *i* può fare ingresso in altra rete (o in generale una migrazione) 
andando a costituire un nuovo g-nodo di livello *j* in un g-nodo esistente di livello *j* + 1, con *j* > *i*.

Prendiamo ad esempio i valori dell'ingresso di *𝛼* in *G<sub>𝛾</sub>*.

Ricordiamo le informazioni salienti di questo ingresso.

**entr04**

Il nodo *𝛼<sub>0</sub>* era da solo e aveva indirizzo 3·0·1·0 in *G<sub>𝛼</sub>*. Con questa operazione
il singolo nodo abbandona la vecchia rete e entra nella nuova come nuovo g-nodo di livello 2. Per questo
*𝛼<sub>0</sub>* assume indirizzo *di connettività* 3·0·1·2 in *G<sub>𝛼</sub>*. Temporaneamente
*𝛼<sub>1</sub>* assume indirizzo *virtuale* 2·2·1·0 in *G<sub>𝛾</sub>*. Dopo poco *𝛼<sub>1</sub>* assume
indirizzo 2·0·1·0 in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝛼<sub>0</sub>* viene dismesso.

**sistema 𝛼**
```
eth1         00:16:3E:FD:E2:AA   169.254.69.30
entr04_eth1  00:16:3E:78:18:0B   169.254.202.128
```

**sistema 𝛽**
```
eth1         00:16:3E:EC:A3:E1   169.254.96.141
```

### Formazione dell'arco

**sistema 𝛼**
```
ip route add 169.254.96.141 dev eth1 src 169.254.69.30
```

**sistema 𝛽**
```
ip route add 169.254.69.30 dev eth1 src 169.254.96.141
```

Sequenza di operazioni eseguita dal modulo Neighborhood.

### Spostamento vecchia identità in nuovo network namespace

**sistema 𝛼**
```
ip netns add entr04
ip netns exec entr04 sysctl net.ipv4.ip_forward=1
ip netns exec entr04 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr04_eth1 link eth1 type macvlan
ip link set dev entr04_eth1 netns entr04
ip netns exec entr04 sysctl net.ipv4.conf.entr04_eth1.rp_filter=0
ip netns exec entr04 sysctl net.ipv4.conf.entr04_eth1.arp_ignore=1
ip netns exec entr04 sysctl net.ipv4.conf.entr04_eth1.arp_announce=2
ip netns exec entr04 ip link set dev entr04_eth1 up
ip netns exec entr04 ip address add 169.254.202.128 dev entr04_eth1
ip netns exec entr04 ip route add 169.254.96.141 dev entr04_eth1 src 169.254.202.128
```

**sistema 𝛽**
```
ip route add 169.254.202.128 dev eth1 src 169.254.96.141
```

Sequenza di operazioni eseguita dal modulo Identities su richiesta del programma **qspnclient** quando
l'utente comanda di avviare l'ingresso in altra rete.

#### Copia tabelle e regole, spostamento rotte

**sistema 𝛼**
```
ip netns exec entr04 ip rule add table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.28/30 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.92/30 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.60/30 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.24/31 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.88/31 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.56/31 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.27/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.91/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.59/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.51/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.41/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.26/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.90/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.58/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.50/32 table ntk
ip netns exec entr04 ip route add unreachable 10.0.0.40/32 table ntk
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.8/29 table ntk
ip route del 10.0.0.72/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.28/30 table ntk
ip route del 10.0.0.92/30 table ntk
ip route del 10.0.0.60/30 table ntk
ip route del 10.0.0.24/31 table ntk
ip route del 10.0.0.88/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.27/32 table ntk
ip route del 10.0.0.91/32 table ntk
ip route del 10.0.0.59/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.26
ip address del 10.0.0.26/32 dev eth1
ip address del 10.0.0.90/32 dev eth1
ip address del 10.0.0.58/32 dev eth1
ip netns exec entr04 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.28/30 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.92/30 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.60/30 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.24/31 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.88/31 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.56/31 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.27/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.91/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.59/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.51/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.41/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.26/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.90/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.58/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.50/32 table ntk
ip netns exec entr04 ip route change unreachable 10.0.0.40/32 table ntk
```

Seconda parte di operazioni eseguita dal programma **qspnclient** quando
l'utente comanda di avviare l'ingresso in altra rete.

In questa situazione notiamo che nel preparare il vecchio namespace per la nuova identità
vengono eliminati i propri indirizzi IP globale e anonimizzante e quelli interni solo
ai livelli superiori al g-nodo che viene creato (non il livello del g-nodo che migra).
In questo caso fino al livello 3, mentre restano in essere gli indirizzi `10.0.0.50` e `10.0.0.40`.

### Popolamento nuove rotte della nuova identità

**sistema 𝛽**
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
ip route add unreachable 10.0.0.20/30 table ntk
ip route add unreachable 10.0.0.84/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
```

Terza parte di operazioni eseguita dal programma **qspnclient** quando
l'utente comanda di avviare l'ingresso in altra rete.

[Pagina seguente](Eventi7.md)
