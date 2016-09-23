# Proof of concept - Eventi - Pagina 11

[Pagina precedente](Eventi10.md)

## Migrazione di un g-nodo

Nel trattare [questo](UseCases15.md) caso d'uso, abbiamo descritto la migrazione di *𝜑* (che aveva
indirizzo 1·1·) dal g-nodo 1· al g-nodo 0· (con indirizzo 0·0·).

Ricordiamo le informazioni salienti di questo ingresso.

**migr02**

Le operazioni **migr02** e **entr06** sono collegate.  
Indichiamo con *𝜑<sub>0</sub>* il g-nodo di livello 1 composto da *𝛽<sub>1</sub>* *𝛾<sub>0</sub>* e *𝜀<sub>1</sub>*
che aveva indirizzo 2·1·1· in *G<sub>𝛾</sub>*.  
Con questa migrazione *𝜑<sub>0</sub>* assume indirizzo *di connettività* 2·1·3· e si costituisce un
*nuovo g-nodo* *𝜑<sub>1</sub>* composto da *𝛽<sub>3</sub>* *𝛾<sub>1</sub>* e *𝜀<sub>2</sub>*.
Temporaneamente *𝜑<sub>1</sub>* assume indirizzo *virtuale* 2·0·2·. Dopo poco *𝜑<sub>1</sub>* assume
indirizzo 2·0·0·.  
L'identità *𝜑<sub>0</sub>* non viene dismessa finché una verifica con `check_connectivity` non darà
esito positivo.

**entr06**

Il sistema *𝜆* gestisce una rete privata di 2 indirizzi.  
Il nodo *𝜆<sub>0</sub>* era da solo e aveva indirizzo 1·1·0·0 in *G<sub>𝜆</sub>*. Con questa operazione
di ingresso *𝜆<sub>0</sub>* assume indirizzo *di connettività* 1·1·2·0 in *G<sub>𝜆</sub>*. Temporaneamente
*𝜆<sub>1</sub>* assume indirizzo *virtuale* 2·1·2·0 in *G<sub>𝛾</sub>*. Dopo poco *𝜆<sub>1</sub>* assume
indirizzo 2·1·1·0 in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝜆<sub>0</sub>* viene dismesso.

----

Vediamo come vanno simulate nel programma **qspnclient** le operazioni di coordinamento tra i vari
singoli nodi che portano alle operazioni di migrazione e ingresso che interessano i moduli di cui stiamo
trattando in questa proof-of-concept.

L'utente nel sistema *𝜆* dà il comando `enter_net_phase1` dicendo al **qspnclient** che:
*   l'identità *𝜆<sub>0</sub>* deve produrre una copia (sia essa *𝜆<sub>1</sub>*) che faccia
    ingresso in *G<sub>𝛾</sub>* (insieme al suo g-nodo di livello 1 in quanto è un **gateway di rete autonoma**) in 1·.
*   l'identità *𝜆<sub>1</sub>* deve prendere temporaneamente il virtuale 1·2·X. Dove con X indichiamo
    il suo attuale identificativo in quella posizione.
*   è stata individuata una migration path nella quale l'ultimo passaggio ha un dato identificativo
    (che indichiamo con *m<sub>𝜑</sub>*) e libererà per *𝜆<sub>1</sub>* l'indirizzo 1·1·X.

L'utente nel sistema *𝜀* dà il comando `prepare_migrate` dicendo al **qspnclient** che:
*   l'identità *𝜀<sub>1</sub>* deve produrre una copia (sia essa *𝜀<sub>2</sub>*) che faccia
    migrazione insieme al suo g-nodo di livello 1 in 0·.
*   l'identità *𝜀<sub>2</sub>* deve prendere temporaneamente il virtuale 0·2·X.
*   è immediatamente disponibile per *𝜀<sub>2</sub>* l'indirizzo 0·0.X.
*   l'identificativo di questa migrazione è *m<sub>𝜑</sub>*.

L'utente nel sistema *𝛽* dà il comando `prepare_migrate` dicendo al **qspnclient** che:
*   l'identità *𝛽<sub>1</sub>* deve produrre una copia (sia essa *𝛽<sub>3</sub>*) che faccia
    migrazione insieme al suo g-nodo di livello 1 in 0·.
*   l'identità *𝛽<sub>3</sub>* deve prendere temporaneamente il virtuale 0·2·X.
*   è immediatamente disponibile per *𝛽<sub>3</sub>* l'indirizzo 0·0.X.
*   l'identificativo di questa migrazione è *m<sub>𝜑</sub>*.

L'utente nel sistema *𝛾* dà il comando `prepare_migrate` dicendo al **qspnclient** che:
*   l'identità *𝛾<sub>0</sub>* deve produrre una copia (sia essa *𝛾<sub>1</sub>*) che faccia
    migrazione insieme al suo g-nodo di livello 1 in 0·.
*   l'identità *𝛾<sub>1</sub>* deve prendere temporaneamente il virtuale 0·2·X.
*   è immediatamente disponibile per *𝛾<sub>1</sub>* l'indirizzo 0·0.X.
*   l'identificativo di questa migrazione è *m<sub>𝜑</sub>*.

L'utente nel sistema *𝜀* dà il comando `migrate` dicendo al **qspnclient** che:
*   va completata la migrazione di g-nodo il cui identificativo è *m<sub>𝜑</sub>*.

L'utente nel sistema *𝛽* dà il comando `migrate` dicendo al **qspnclient** che:
*   va completata la migrazione di g-nodo il cui identificativo è *m<sub>𝜑</sub>*.

L'utente nel sistema *𝛾* dà il comando `migrate` dicendo al **qspnclient** che:
*   va completata la migrazione di g-nodo il cui identificativo è *m<sub>𝜑</sub>*.

L'utente nel sistema *𝜆* dà il comando `enter_net_phase2` dicendo al **qspnclient** che:
*   la migrazione *m<sub>𝜑</sub>* (di cui era in attesa *𝜆<sub>1</sub>*) è stata completata.

----

### Operazioni iniziali in *𝜆*

**sistema 𝜆**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip address add 10.0.0.32 dev lo

sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2

ip link set dev eth1 up
ip address add 169.254.109.22 dev eth1

(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk

ip address add 10.0.0.12 dev eth1
ip address add 10.0.0.76 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
ip address add 10.0.0.40 dev eth1

ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.8/30 table ntk
ip route add unreachable 10.0.0.72/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.14/31 table ntk
ip route add unreachable 10.0.0.78/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
# PREROUTING => map destination
iptables -t nat -A PREROUTING -d 10.0.0.12/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.76/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.60/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A PREROUTING -d 10.0.0.48/31 -j NETMAP --to 10.0.0.40/31
# POSTROUTING => map source
iptables -t nat -A POSTROUTING -d 10.0.0.48/30 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.48/31
iptables -t nat -A POSTROUTING -d 10.0.0.56/29 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.60/31
iptables -t nat -A POSTROUTING -d 10.0.0.0/27  -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31

iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.12

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.8/30 table ntk
ip route change unreachable 10.0.0.72/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.14/31 table ntk
ip route change unreachable 10.0.0.78/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
```

In questa sequenza di operazioni eseguita all'avvio del programma **qspnclient** notiamo
che la differenza (rispetto ad un nodo comune trattandosi di un nodo che fa da gateway ad una rete
autonoma che costituisce un g-nodo di livello 1) si trova nella parte che assegna le rotte
nella tabella `ntk`. Gli indirizzi IP delle possibili destinazioni sono calcolati solo
fino al livello del g-nodo autonomo (nel nostro esempio: 1) e poi si opera con `iptables`
la mappatura della sottorete come si evince dall'esempio.

Sono lasciate a qualche altro meccanismo nel sistema *𝜆* le seguenti operazioni:

**sistema 𝜆**
```
ip route add 169.254.110.188 dev eth1 src 169.254.109.22
ip route add 10.0.0.41/32 table ntk via 169.254.110.188 dev eth1 src 10.0.0.40
```

Queste vanno eseguite pochi istanti dopo l'avvio del programma **qspnclient** di modo che la
tabella `ntk` sia stata predisposta e l'indirizzo IP interno al g-nodo di livello 1 sia
stato assegnato alle interfacce di rete gestite.

### Formazione dell'arco tra *𝜆* e *𝜀*

Non ripetiamo questa trattazione, in quanto del tutto simile a quanto visto nella migrazione di un singolo nodo.

### entr06: Spostamento vecchia identità di *𝜆* in nuovo network namespace

Non ripetiamo questa trattazione, in quanto del tutto simile a quanto visto nella migrazione di un singolo nodo.

L'unica differenza nelle operazioni è dovuta al fatto che *𝜆* è un nodo che fa da gateway
verso una rete a gestione autonoma di un g-nodo di livello 1. La differenza si trova nella parte che
ripulisce il vecchio network namespace dalle rotte della vecchia identità, ed è analoga a quanto visto
nelle operazioni iniziali.



[Pagina seguente](Eventi12.md)
