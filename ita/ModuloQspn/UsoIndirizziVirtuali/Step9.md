# Modulo QSPN - Esempio di uso degli indirizzi virtuali

## Passo 9

In questo passo, siccome il g-nodo 0· non è pieno, possiamo assegnare a 𝜑<sub>N</sub> un indirizzo *reale* *definitivo*
in esso, cioè modificare il 0·2· in 0·0·. O meglio, ciascun nodo che ha una identità in 𝜑<sub>N</sub> (vale a
dire 𝛽<sub>i(1,2),N</sub>, 𝛾<sub>N</sub> ed 𝜀<sub>N</sub>) modifica l'indirizzo gestito da quella identità.

Alcuni comandi vanno dati sui sistemi che hanno in 𝜑<sub>N</sub> la loro identità che gestisce l'indirizzo *definitivo*,
vale a dire solo 𝛾<sub>N</sub> ed 𝜀<sub>N</sub>. Ad essi dobbiamo aggiungere l'indirizzo IP globale e gli indirizzi IP
*interni* ai propri g-nodi di livello maggiore del livello di 𝜑, cioè 1. Nel nostro caso, l'indirizzo nella
classe 10.0.0.0/24 e quello nella classe 10.0.2.0/24. Dopo vanno cambiate le rotte nella relativa tabella di modo
da avere questi indirizzi come source preferiti, "src", per i pacchetti originati dal nodo.

Inoltre, su ogni sistema che ha una identità in 𝜑<sub>N</sub>, per ogni rotta verso indirizzi IP *interni* ai propri
g-nodi di livello minore o uguale al livello di 𝜑, cioè 1, va aggiunta una analoga rotta che abbia quella destinazione
espressa come indirizzo IP globale e una per ogni indirizzo IP *interno* ai propri g-nodi di livello maggiore del livello di 𝜑.

Quindi diamo questi comandi:

**sistema 𝛾**
```
ip address add 10.0.0.0 dev eth1
ip address add 10.0.2.0 dev eth1
ip route change 10.0.0.2/31 via 169.254.96.141 dev eth1 src 10.0.0.0
ip route change 10.0.0.4/30 via 169.254.253.216 dev eth1 src 10.0.0.0
ip route change 10.0.2.2/31 via 169.254.96.141 dev eth1 src 10.0.2.0
ip route add 10.0.0.1 via 169.254.27.218 dev eth1 src 10.0.0.0
ip route add 10.0.2.1 via 169.254.27.218 dev eth1 src 10.0.2.0
```
**sistema 𝜀**
```
ip address add 10.0.0.1 dev eth1
ip address add 10.0.2.1 dev eth1
ip route change 10.0.0.2/31 via 169.254.96.141 dev eth1 src 10.0.0.1
ip route change 10.0.0.4/30 via 169.254.96.141 dev eth1 src 10.0.0.1
ip route change 10.0.2.2/31 via 169.254.96.141 dev eth1 src 10.0.2.1
ip route add 10.0.0.0 via 169.254.27.218 dev eth1 src 10.0.0.1
ip route add 10.0.2.0 via 169.254.27.218 dev eth1 src 10.0.2.1
```
**sistema 𝛽**
```
ip netns exec ntkv0 ip route add 10.0.0.0 via 169.254.94.223 dev ntkv0_eth1
ip netns exec ntkv0 ip route add 10.0.2.0 via 169.254.94.223 dev ntkv0_eth1
ip netns exec ntkv0 ip route add 10.0.0.1 via 169.254.163.36 dev ntkv0_eth1
ip netns exec ntkv0 ip route add 10.0.2.1 via 169.254.163.36 dev ntkv0_eth1
```

I percorsi che ogni nodo in 𝜑<sub>N</sub> aveva memorizzati come nodo con indirizzo 0·2·X valgono
ora per il suo indirizzo 0·0·X.

Ogni border-nodo in 𝜑<sub>N</sub> comunica via ETP ai suoi vicini esterni a 𝜑<sub>N</sub>, cioè
a 𝛽<sub>B</sub> e 𝛿, che non è più possibile raggiungere tramite lui l'indirizzo 0·2·; ma che ora
è possibile raggiungere tramite lui l'indirizzo 0·0·.

L'effetto di questo ETP, che contiene solo modifiche a percorsi verso g-nodi di livello 1, è comunque
limitato ai nodi del g-nodo di livello 2, cioè a  𝛼 e 𝛽<sub>B</sub>. Nel senso che quando esso raggiunge
il nodo 𝛿 non contiene nessuna novità.

Quindi diamo questi comandi:

**sistema 𝛼**
```
ip route add 10.0.0.0/31 src 10.0.0.2 via 169.254.96.141 dev eth1
ip route add 10.0.2.0/31 src 10.0.2.2 via 169.254.96.141 dev eth1
```
**sistema 𝛽**
```
ip route add 10.0.0.0/31 src 10.0.0.3 via 169.254.94.223 dev eth1
ip route add 10.0.2.0/31 src 10.0.2.3 via 169.254.94.223 dev eth1
```

Di nuovo, i sistemi in 𝜑<sub>N</sub> sono in grado di comunicare con gli altri sistemi della rete.

Proseguiamo con il [passo 10](Step10.md).

