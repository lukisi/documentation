# Modulo QSPN - Esempio di uso degli indirizzi virtuali

## Passo 10

Il sistema 𝜆 può ora entrare in 1· grazie al suo link con 𝜀<sub>i(2,2)</sub> e assegnarsi l'indirizzo 1·1·. Poiché
esso vuole fare da gateway verso una sottorete autonoma, diciamo che si assegna l'indirizzo 1·1·0 e riserva
l'indirizzo 1·1·1 per un nodo nella sua sottorete.

![grafo12](img/Step10/grafo12.png)

Si vede dal grafo che, pur non essendoci un link diretto tra 𝜆 e 𝛿, il g-nodo 1· resta internamente connesso.

Quindi diamo questi comandi:

**sistema 𝜆**
```
ip route add unreachable 10.0.0.0/29
ip route add unreachable 10.0.2.0/30
ip route add unreachable 10.0.1.0/31
ip address add 10.0.0.6 dev eth1
ip address add 10.0.2.2 dev eth1
ip address add 10.0.1.0 dev eth1
```

Il nodo 𝜆 chiede e riceve un ETP completo da 𝜀<sub>i(2,2)</sub> e uno da 𝜀<sub>N</sub>. Tramite essi:

*   Il nodo 𝜆 sa di poter raggiungere il g-nodo 0· passando per il vicino 𝜀<sub>N</sub>.
*   Il nodo 𝜆 sa di poter raggiungere il g-nodo 1·0· passando per il vicino 𝜀<sub>i(2,2)</sub>.

Quindi diamo questi comandi:

**sistema 𝜆**
```
ip route add 10.0.0.0/30 src 10.0.0.6 via 169.254.163.36 dev eth1
ip route add 10.0.0.4/31 src 10.0.0.6 via 169.254.241.153 dev eth1
ip route add 10.0.2.0/31 src 10.0.2.2 via 169.254.241.153 dev eth1
```

Il nodo 𝜆 ha terminato il bootstrap. Esso ora invia un ETP che informa su come raggiugere 1·1·. In quanto
ricevuto da 𝜀<sub>i(2,2)</sub> (che fa parte di 1·) esso si propaga solo internamente a 1·. In quanto ricevuto
da 𝜀<sub>N</sub> (che fa parte di 0·) esso informa di un nuovo percorso per raggiungere 1· e si propaga internamente a 0·.

Per brevità, supponiamo che ad eccezione di 𝜀<sub>N</sub>, tutti gli altri nodi di [0] preferiscano il vecchio
percorso per raggiungere [1].

Diamo questi comandi ai sistemi:

**sistema 𝜀**
```
# come 𝜀N
ip route change 10.0.0.4/30 via 169.254.109.22 dev eth1 src 10.0.0.1
# come 𝜀i22
ip netns exec ntkv1 ip route add 10.0.0.6/31 via 169.254.109.22 dev ntkv1_eth1
ip netns exec ntkv1 ip route add 10.0.2.2/31 via 169.254.109.22 dev ntkv1_eth1
```
**sistema 𝛽**
```
# come 𝛽i12,i22
ip netns exec ntkv1 ip route add 10.0.0.6/31 via 169.254.241.153 dev ntkv1_eth1
ip netns exec ntkv1 ip route add 10.0.2.2/31 via 169.254.241.153 dev ntkv1_eth1
```
**sistema 𝛾**
```
# come 𝛾i22
ip netns exec ntkv1 ip route add 10.0.0.6/31 via 169.254.42.4 dev ntkv1_eth1
ip netns exec ntkv1 ip route add 10.0.2.2/31 via 169.254.42.4 dev ntkv1_eth1
```
**sistema 𝛿**
```
ip route add 10.0.0.6/31 via 169.254.24.198 dev eth1 src 10.0.0.5
ip route add 10.0.2.2/31 via 169.254.24.198 dev eth1 src 10.0.2.1
```
**sistema 𝜇**
```
ip route add 10.0.0.6/31 via 169.254.253.216 dev eth1 src 10.0.0.4
ip route add 10.0.2.2/31 via 169.254.253.216 dev eth1 src 10.0.2.0
```

Possiamo verificare che il nodo 𝜆 raggiunge tutti gli indirizzi IP dei nodi esistenti.

Inoltre possiamo verificare che i nodi che hanno assunto una identità ''di connettività'' sono in grado di
smistare correttamente pacchetti IP aventi per destinazione un indirizzo ''interno''.

*   Se il nodo 𝜀 invia un pacchetto a 10.0.2.0 (cioè verso il nodo x·0·0 nel g-nodo di livello 1 di cui
    fa parte 𝜀) raggiunge 𝛾.
*   Se il nodo 𝜆 invia un pacchetto a 10.0.2.0 (cioè verso il nodo x·0·0 nel g-nodo di livello 1 di cui
    fa parte 𝜆), sebbene passa da 𝜀 e da 𝛾, raggiunge 𝜇.

Proseguiamo con il [passo 11](Step11.md).

