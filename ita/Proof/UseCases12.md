# Proof of concept - Casi d'uso - Pagina 12

[Pagina precedente](UseCases11.md)

## Migrazione di un singolo nodo

Ora assumiamo un evento che porta alla decisione di far migrare un singolo nodo da un g-nodo
ad un altro all'interno della stessa rete.

Un nuovo sistema *𝜀* viene rilevato solo dal sistema *𝛽*. Assumiamo per convenienza di non voler
costituire un nuovo g-nodo di livello 3. Quando indicheremo gli indirizzi Netsukuku quindi tralasceremo
di indicare la parte iniziale che è sempre 2·.

Il sistema *𝜀* può fare ingresso solo in un g-nodo a cui appartiene anche *𝛽*, poiché esso è il suo unico vicino.
Il g-nodo *g<sub>1</sub>(𝛽)* è saturo. Anche *g<sub>2</sub>(𝛽)* è saturo. Anche *g<sub>3</sub>(𝛽)* è saturo.
E abbiamo escluso di costituire un nuovo g-nodo di livello 3. Si deve quindi ricorrere ad una *migration path*.

**Appunto**: quando un g-nodo di livello *i* vuole fare ingresso in una nuova rete, se tramite i suoi archi
non può trovare un g-nodo di livello *j* non saturo, con *i* + 1 ≤ *j* ≤ *i* + *k* dove *k* è un intero positivo
piccolo a piacere, allora si tenta di ricorrere ad una *migration path*. Si vuole cioè evitare di ricorrere ad una
migration path appena un singolo nodo non trova libero un g-nodo di livello 1, ma allo stesso tempo evitare
di occupare un g-nodo di livello alto per l'ingresso di un singolo nodo. Ragionamento simile per
l'ingresso di un g-nodo di livello *i*.

Si decide di far entrare il sistema *𝜀* in *g<sub>1</sub>(𝛽)* = 1·1· con indirizzo temporaneamente *virtuale* 1·1·2.
Si farà poi migrare il sistema *𝛽* in *g<sub>1</sub>(𝛼)* = 0·1·, con indirizzo dapprima virtuale 0·1·2 e dopo
con indirizzo reale 0·1·1. L'identità di connettività con cui *𝛽* resta in 1·1· avrà indirizzo 1·1·3.
In seguito l'indirizzo di *𝜀* diventerà il *reale* 1·1·1. Il g-nodo 1·1· resta internamente connesso e tutti
i sistemi hanno l'identità principale con un indirizzo IP globale raggiungibile dagli altri.

Tutti i dialoghi tra vicini che portano alla scelta delle nuove posizioni occupate e alla comunicazione delle
relative informazioni ai vari sistemi interessati vengono fatte inizialmente (dopo la realizzazione dell'arco
tra *𝜀* e *𝛽*) tramite meccanismi estranei ai moduli di cui stiamo trattando in questa proof-of-concept.

1.  [Operazioni iniziali in *𝜀*](#Operazioni_iniziali_epsilon)
1.  [Arco tra *𝜀* e *𝛽*](#Arco_epsilon_beta)
1.  [Ingresso di *𝜀* - Prima fase](#Ingresso_epsilon_prima_fase)
1.  [Migrazione di *𝛽*](#Migrazione_beta)
    1.  [Cambio namespace per *𝛽<sub>1</sub>*](#Cambio_namespace_beta1)
    1.  [Costituzione *𝛽<sub>2</sub>*](UseCases13.md#Costituzione_beta2)
    1.  [Rimozione arco *𝛽<sub>1</sub>* - *𝛼*](UseCases13.md#Rimozione_arco_beta1_alfa)
    1.  [Cambio indirizzo per *𝛽<sub>2</sub>*](UseCases13.md#Cambio_indirizzo_beta2)
1.  [Ingresso di *𝜀* - Seconda fase](UseCases14.md#Ingresso_epsilon_seconda_fase)

### <a name="Operazioni_iniziali_epsilon"></a>Operazioni iniziali in *𝜀*

**sistema 𝜀**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip address add 10.0.0.32 dev lo
ip link set dev eth1 address 00:16:3E:3C:14:33
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
ip link set dev eth1 up
ip address add 169.254.163.36 dev eth1
ip address add 10.0.0.28 dev eth1
ip address add 10.0.0.92 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
ip address add 10.0.0.40 dev eth1
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.24/30 table ntk
ip route add unreachable 10.0.0.88/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.30/31 table ntk
ip route add unreachable 10.0.0.94/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
ip route add unreachable 10.0.0.29/32 table ntk
ip route add unreachable 10.0.0.93/32 table ntk
ip route add unreachable 10.0.0.61/32 table ntk
ip route add unreachable 10.0.0.49/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/30 table ntk
ip route change unreachable 10.0.0.88/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.30/31 table ntk
ip route change unreachable 10.0.0.94/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change unreachable 10.0.0.29/32 table ntk
ip route change unreachable 10.0.0.93/32 table ntk
ip route change unreachable 10.0.0.61/32 table ntk
ip route change unreachable 10.0.0.49/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
```

### <a name="Arco_epsilon_beta"></a>Arco tra *𝜀* e *𝛽*

Il modulo Neighborhood produce questi comandi:

**sistema 𝜀**
```
ip route add 169.254.96.141 dev eth1 src 169.254.163.36
```

**sistema 𝛽**
```
ip route add 169.254.163.36 dev eth1 src 169.254.96.141
```

### <a name="Ingresso_epsilon_prima_fase"></a>Ingresso di *𝜀* - Prima fase

Il modulo Identities produce questi comandi per preparare il nuovo network
namespace nel sistema *𝜀* per la vecchia identità:

**sistema 𝜀**
```
ip netns add entr05
ip netns exec entr05 sysctl net.ipv4.ip_forward=1
ip netns exec entr05 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr05_eth1 link eth1 type macvlan
ip link set dev entr05_eth1 netns entr05
ip netns exec entr05 ip link set dev entr05_eth1 address 00:16:3E:13:28:B2
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

**Spostare** La vecchia identità mantiene il suo indirizzo Netsukuku inalterato, ad eccezione della posizione
in cui diventa *virtuale* per mantenere la connettività dei g-nodi superiori. Mantiene inalterate
anche le sue possibili destinazioni, aggiungendone una che era la sua vecchia posizione *reale*.  
Nel nuovo network namespace la vecchia identità non tiene più traccia del suo proprio indirizzo Netsukuku,
perché essendo ora una identità di connettività non è mai l'end-point di una qualsiasi comunicazione che
ha luogo nella rete Netsukuku, né come *src* né come *dest*.  
Se la vecchia identità era in precedenza l'identità principale, vale a dire se il precedente network namespace
era quello default, allora tale identità poteva essere in precedenza l'end-point di una qualche comunicazione
in atto nella rete Netsukuku. In questo caso vanno aggiunte le seguenti considerazioni. Il vecchio indirizzo
Netsukuku nel suo complesso non è più riferibile a questo sistema. Ma se a migrare (o fare ingresso) è un intero
g-nodo, allora la parte interna di tale indirizzo Netsukuku è ancora riferibile a questo sistema. Pertanto vanno
rimossi dal network namespace default gli indirizzi IP globale e interni nei g-nodi di livello alto (che saranno
rimpiazzati da nuovi indirizzi) ma vanno mantenuti gli indirizzi IP interni nei g-nodi di livello basso a
partire dal livello del g-nodo che ha migrato.  
Il vecchio network namespace era stato gestito dalla vecchia identità e per questo era stato popolato con
alcune possibili destinazioni. Le operazioni che riguardano le rotte nel vecchio network namespace vanno avviate
solo dopo che: a) la vecchia identità ha predisposto il nuovo network namespace; b) i vicini esterni al g-nodo
che ha migrato hanno cambiato i gateway nelle loro rotte sulla base del cambio di peer-linklocal del loro
arco-identità.  
Nel vecchio network namespace vanno rimosse tutte le rotte tranne quelle verso indirizzi IP interni al
proprio g-nodo di livello inferiore o uguale al livello del g-nodo che ha migrato.

Il programma *qspnclient* nel sistema *𝜀*, per trasferire le impostazioni relative alla sua vecchia identità
nel nuovo network namespace, produce questi comandi:

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

**Spostare** La nuova identità conosce subito il suo nuovo indirizzo Netsukuku, che inizialmente
è *virtuale*. Quindi conosce anche le sue nuove possibili destinazioni. Pertanto le aggiunge
nel vecchio network namespace; tranne quelle che c'erano, cioè quelle verso indirizzi IP interni al
proprio g-nodo di livello inferiore o uguale al livello del g-nodo che ha migrato.

Il programma *qspnclient* nel sistema *𝜀*, per aggiungere le impostazioni relative alla sua nuova identità
nel precedente network namespace, produce questi comandi:

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

Il programma *qspnclient* nel sistema *𝜀*, relativamente alla sua nuova identità, fornisce al modulo QSPN
un arco verso il vicino nodo *𝛽*. Per esso costituisce anche la tabella `ntk_from_xxx`, che però
non sarà menzionata in alcuna regola prima di aver ricevuto e processato un ETP da questo arco.

**sistema 𝜀**
```
(echo; echo "250 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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

Analogamente nel sistema *𝛽* viene fornito al modulo QSPN un arco verso il vicino sistema *𝜀*.

**sistema 𝛽**
```
(echo; echo "248 ntk_from_00:16:3E:3C:14:33 # xxx_table_ntk_from_00:16:3E:3C:14:33_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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

### <a name="Migrazione_beta"></a>Migrazione di *𝛽*

#### <a name="Cambio_namespace_beta1"></a>Cambio namespace per *𝛽<sub>1</sub>*

La vecchia identità *𝛽<sub>1</sub>* aveva indirizzo 1·1·1, era la principale nel network namespace
default. Con questa operazione si sposta nel network namespace **migr01** e diventa di connettività
con indirizzo 1·1·3, cioè con un indirizzo virtuale nel g-nodo origine della migrazione.

Verifichiamo che il *passaggio* della vecchia identità *𝛽<sub>1</sub>* ad un nuovo network
namespace nel suo sistema è fatto in modo tale che in nessun momento si interrompe il servizio di
forwarding offerto dal sistema *𝛽*. Ad esempio, una connessione tra il sistema 𝛼 e il sistema 𝛾
non si interrompe e non perde pacchetti.

Il modulo Identities produce questi comandi per preparare il nuovo network
namespace per la vecchia identità:

**sistema 𝛽**
```
ip netns add migr01
ip netns exec migr01 sysctl net.ipv4.ip_forward=1
ip netns exec migr01 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev migr01_eth1 link eth1 type macvlan
ip link set dev migr01_eth1 netns migr01
ip netns exec migr01 ip link set dev migr01_eth1 address 00:16:3E:EE:AF:D1
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.rp_filter=0
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.arp_ignore=1
ip netns exec migr01 sysctl net.ipv4.conf.migr01_eth1.arp_announce=2
ip netns exec migr01 ip link set dev migr01_eth1 up
ip netns exec migr01 ip address add 169.254.27.218 dev migr01_eth1
ip netns exec migr01 ip route add 169.254.69.30 dev migr01_eth1 src 169.254.27.218
ip netns exec migr01 ip route add 169.254.94.223 dev migr01_eth1 src 169.254.27.218
ip netns exec migr01 ip route add 169.254.163.36 dev migr01_eth1 src 169.254.27.218
ip netns exec migr01 ip route add 169.254.133.31 dev migr01_eth1 src 169.254.27.218
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
ip netns exec entr05 ip route add 169.254.27.218 dev entr05_eth1 src 169.254.133.31
```

Il programma *qspnclient*, per trasferire le impostazioni relative alla sua vecchia identità
nel nuovo network namespace, produce questi comandi:

**sistema 𝛽**
```
ip netns exec migr01 ip rule add table ntk
ip netns exec migr01 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250
ip netns exec migr01 ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249
ip netns exec migr01 ip rule add fwmark 249 table ntk_from_00:16:3E:FD:E2:AA

ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.16/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.80/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.20/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.84/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.22/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.86/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.62/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.50/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.23/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.87/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.63/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.51/32 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.41/32 table ntk

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

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec migr01 ip route change 10.0.0.16/30 table ntk via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.80/30 table ntk via 169.254.69.30 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.56/30 table ntk via 169.254.69.30 dev migr01_eth1
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
```

Ora il programma *qspnclient* nel sistema *𝛽* si accerta che i suoi vicini abbiano modificato le
rotte che prevedono di usare *𝛽<sub>1</sub>* come gateway.

Nel nostro esempio si tratta dei sistemi *𝛼* e *𝛾*. In essi occorre cambiare le tabelle e le regole per via
del cambio di indirizzo MAC del peer.  
Prima si rimuove la vecchia regola. Poi si svuota/elimina la vecchia tabella. Poi si toglie la marcatura dei pacchetti
provenienti dal vecchio MAC. Poi si aggiunge la nuova tabella con le rotte unreachable. Poi si aggiunge la marcatura
dei pacchetti provenienti dal nuovo MAC.  
Poi vanno aggiornate tutte le rotte in tutte le tabelle. Infine si aggiunge la nuova regola.

**sistema 𝛼**
```
ip rule del fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
ip route flush table ntk_from_00:16:3E:EC:A3:E1
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250

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
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 250

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250

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

Anche il sistema *𝜀* è un vicino di *𝛽*, ma esso non ha completato il bootstrap quindi non ha ancora aggiunto
una regola per la tabella `ntk_from_00:16:3E:EC:A3:E1` e nemmeno alcuna rotta che usa *𝛽<sub>1</sub>* come gateway.  
Quindi per tale sistema si svuota/elimina la vecchia tabella. Poi si aggiunge la nuova tabella con le rotte unreachable.

**sistema 𝜀**
```
ip route flush table ntk_from_00:16:3E:EC:A3:E1
sed -i '/xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx/d' /etc/iproute2/rt_tables

(echo; echo "250 ntk_from_00:16:3E:EE:AF:D1 # xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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

Dopo il programma *qspnclient* nel sistema *𝛽* proseguirà con questi comandi:

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

iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.23
ip address del 10.0.0.23/32 dev eth1
ip address del 10.0.0.87/32 dev eth1
ip address del 10.0.0.63/32 dev eth1
ip address del 10.0.0.51/32 dev eth1
ip address del 10.0.0.41/32 dev eth1
```

Siccome i vicini *𝛼* e *𝛾* sono esterni al g-nodo che ha migrato, dal vecchio network namespace
della vecchia identità vanno rimosse anche le regole per le tabelle `ntk_from_xxx` relative.
Infatti la nuova identità dovrà attendere un ETP da questi archi prima di poter aggiornare le tabelle.

**sistema 𝛽**
```
ip rule del fwmark 250 table ntk_from_00:16:3E:5B:78:D5
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250

ip rule del fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249
```

Ora consideriamo che la vecchia identità *𝛽<sub>1</sub>* comunica con un ETP ai suoi vicini *𝛼*
e *𝛾* che non è più possibile raggiungere tramite lui l'indirizzo 1·1·1; e che ora è possibile
raggiungere tramite lui l'indirizzo 1·1·2.

Questo ETP viene recepito solo dai nodi del g-nodo di livello 1, trattandosi di due percorsi verso
singoli nodi, cioè solo dal nodo *𝛾*.

Quindi diamo questi comandi:

**sistema 𝛾**
```
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
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
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
```

[Pagina seguente](UseCases13.md)
