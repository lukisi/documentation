# Modulo QSPN - Esempio di uso degli indirizzi virtuali

## Passo 2

In questo passo specifichiamo la topologia che è usata nella rete.

Consideriamo una topologia a 3 livelli con *gsize* ad ogni livello = 2. Quindi abbiamo a disposizione solo
2<sup>3</sup> = 8 indirizzi, da 10.0.0.0 a 10.0.0.7 in IPv4.

Usiamo anche gli indirizzi IP *interni* ai g-nodi:

*   dentro ogni g-nodo di livello 1 abbiamo 2 indirizzi: da 10.0.1.0 a 10.0.1.1.
*   dentro ogni g-nodo di livello 2 abbiamo 4 indirizzi: da 10.0.2.0 a 10.0.2.3.

Inoltre supponiamo che le fasi iniziali della formazione della rete sono state già eseguite e che si sia giunti
ad una assegnazione di indirizzi e percorsi come preciseremo sotto.

Indichiamo gli indirizzi dei singoli nodi con una tupla di 3 elementi separati da "·",
p<sub>2</sub>·p<sub>1</sub>·p<sub>0</sub>, dove p<sub>0</sub> è la posizione del nodo nel g-nodo
di livello 1, ..., p<sub>2</sub> è la posizione del g-nodo di livello 2 nella rete.

Indichiamo i g-nodi di livello 1 con una tupla di 2 elementi terminati con un "·", p<sub>2</sub>·p<sub>1</sub>·.

Indichiamo i g-nodi di livello 2 con una tupla di 1 elemento terminato con un "·", p<sub>2</sub>·.

Indichiamo con la scrittura *g<sub>1</sub>(𝛼)*, *g<sub>2</sub>(𝛼)*, ..., il g-nodo di livello 1, 2, ..., a cui appartiene il nodo 𝛼.

Assegnamo questi indirizzi:

*   𝛼 = 0·1·0
*   𝛽 = 1·1·1
*   𝛾 = 1·1·0
*   𝛿 = 1·0·1
*   𝜇 = 1·0·0

Quindi abbiamo che i g-nodi di livello 1 sono:

*   g<sub>1</sub>(𝛼) = 0·1· = {𝛼}
*   g<sub>1</sub>(𝛽) = 1·1· = {𝛽, 𝛾} e gli archi {𝛽-𝛾} quindi è connesso
*   g<sub>1</sub>(𝛿) = 1·0· = {𝛿, 𝜇} e gli archi {𝛿-𝜇} quindi è connesso

mentre i g-nodi di livello 2 sono:

*   g<sub>2</sub>(𝛼) = 0· = {𝛼}
*   g<sub>2</sub>(𝛽) = 1· = {𝛽, 𝛾, 𝛿, 𝜇} e gli archi {𝛽-𝛾, 𝛾-𝛿, 𝛿-𝜇} quindi è connesso

![grafo2](img/Step2/grafo2.png)

Dopo le trasmissioni dei messaggi ETP:

*   Il nodo 𝛼 sa di poter raggiungere il g-nodo 1· passando per l'arco 𝛼-𝛽.
*   Il nodo 𝛽 sa di poter raggiungere il g-nodo 0· passando per l'arco 𝛽-𝛼.
*   Il nodo 𝛽 sa di poter raggiungere il g-nodo 1·0· passando per l'arco 𝛽-𝛾.
*   Il nodo 𝛽 sa di poter raggiungere il nodo 1·1·0 passando per l'arco 𝛽-𝛾.
*   Il nodo 𝛾 sa di poter raggiungere il g-nodo 0· passando per l'arco 𝛾-𝛽.
*   Il nodo 𝛾 sa di poter raggiungere il g-nodo 1·0· passando per l'arco 𝛾-𝛿.
*   Il nodo 𝛾 sa di poter raggiungere il nodo 1·1·1 passando per l'arco 𝛾-𝛽.
*   Il nodo 𝛿 sa di poter raggiungere il g-nodo 0· passando per l'arco 𝛿-𝛾.
*   Il nodo 𝛿 sa di poter raggiungere il g-nodo 1·1· passando per l'arco 𝛿-𝛾.
*   Il nodo 𝛿 sa di poter raggiungere il nodo 1·0·0 passando per l'arco 𝛿-𝜇.
*   Il nodo 𝜇 sa di poter raggiungere il g-nodo 0· passando per l'arco 𝜇-𝛿.
*   Il nodo 𝜇 sa di poter raggiungere il g-nodo 1·1· passando per l'arco 𝜇-𝛿.
*   Il nodo 𝜇 sa di poter raggiungere il nodo 1·0·1 passando per l'arco 𝜇-𝛿.

Diamo questi comandi ai sistemi:

**sistema 𝛼**
```
ip route add unreachable 10.0.0.0/29
ip route add unreachable 10.0.2.0/30
ip route add unreachable 10.0.1.0/31
ip address add 10.0.0.2 dev eth1
ip address add 10.0.2.2 dev eth1
ip address add 10.0.1.0 dev eth1
ip route add 10.0.0.4/30 src 10.0.0.2 via 169.254.96.141 dev eth1
```
**sistema 𝛽**
```
ip route add unreachable 10.0.0.0/29
ip route add unreachable 10.0.2.0/30
ip route add unreachable 10.0.1.0/31
ip address add 10.0.0.7 dev eth1
ip address add 10.0.2.3 dev eth1
ip address add 10.0.1.1 dev eth1
ip route add 10.0.0.0/30 src 10.0.0.7 via 169.254.69.30 dev eth1
ip route add 10.0.0.4/31 src 10.0.0.7 via 169.254.94.223 dev eth1
ip route add 10.0.2.0/31 src 10.0.2.3 via 169.254.94.223 dev eth1
ip route add 10.0.0.6/32 src 10.0.0.7 via 169.254.94.223 dev eth1
ip route add 10.0.2.2/32 src 10.0.2.3 via 169.254.94.223 dev eth1
ip route add 10.0.1.0/32 src 10.0.1.1 via 169.254.94.223 dev eth1
```
**sistema 𝛾**
```
ip route add unreachable 10.0.0.0/29
ip route add unreachable 10.0.2.0/30
ip route add unreachable 10.0.1.0/31
ip address add 10.0.0.6 dev eth1
ip address add 10.0.2.2 dev eth1
ip address add 10.0.1.0 dev eth1
ip route add 10.0.0.0/30 src 10.0.0.6 via 169.254.96.141 dev eth1
ip route add 10.0.0.4/31 src 10.0.0.6 via 169.254.253.216 dev eth1
ip route add 10.0.2.0/31 src 10.0.2.2 via 169.254.253.216 dev eth1
ip route add 10.0.0.7/32 src 10.0.0.6 via 169.254.96.141 dev eth1
ip route add 10.0.2.3/32 src 10.0.2.2 via 169.254.96.141 dev eth1
ip route add 10.0.1.1/32 src 10.0.1.0 via 169.254.96.141 dev eth1
```
**sistema 𝛿**
```
ip route add unreachable 10.0.0.0/29
ip route add unreachable 10.0.2.0/30
ip route add unreachable 10.0.1.0/31
ip address add 10.0.0.5 dev eth1
ip address add 10.0.2.1 dev eth1
ip address add 10.0.1.1 dev eth1
ip route add 10.0.0.0/30 src 10.0.0.5 via 169.254.94.223 dev eth1
ip route add 10.0.0.6/31 src 10.0.0.5 via 169.254.94.223 dev eth1
ip route add 10.0.2.2/31 src 10.0.2.1 via 169.254.94.223 dev eth1
ip route add 10.0.0.4/32 src 10.0.0.5 via 169.254.119.176 dev eth1
ip route add 10.0.2.0/32 src 10.0.2.1 via 169.254.119.176 dev eth1
ip route add 10.0.1.0/32 src 10.0.1.1 via 169.254.119.176 dev eth1
```
**sistema 𝜇**
```
ip route add unreachable 10.0.0.0/29
ip route add unreachable 10.0.2.0/30
ip route add unreachable 10.0.1.0/31
ip address add 10.0.0.4 dev eth1
ip address add 10.0.2.0 dev eth1
ip address add 10.0.1.0 dev eth1
ip route add 10.0.0.0/30 src 10.0.0.4 via 169.254.253.216 dev eth1
ip route add 10.0.0.6/31 src 10.0.0.4 via 169.254.253.216 dev eth1
ip route add 10.0.2.2/31 src 10.0.2.0 via 169.254.253.216 dev eth1
ip route add 10.0.0.5/32 src 10.0.0.4 via 169.254.253.216 dev eth1
ip route add 10.0.2.1/32 src 10.0.2.0 via 169.254.253.216 dev eth1
ip route add 10.0.1.1/32 src 10.0.1.0 via 169.254.253.216 dev eth1
```

Proseguiamo con il [passo 3](Step3.md).

