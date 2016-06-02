# Modulo QSPN - Esempio di uso degli indirizzi virtuali

## Passo 5

In questo passo, siccome il g-nodo 0·1· non è pieno, possiamo assegnare a 𝛽<sub>B</sub> un indirizzo
*reale* *definitivo* in esso, cioè modificare il 0·1·2 in 0·1·1.

Dobbiamo aggiungere al sistema 𝛽 l'indirizzo nella classe 10.0.0.0/24 e gli *interni* nelle classi 10.0.1.0/24 e 10.0.2.0/24.
Dopo vanno cambiate le rotte nella tabella principale di modo da avere questi indirizzi come source preferiti, "src",
per i pacchetti originati dal nodo.

Quindi diamo questi comandi:

**sistema 𝛽**
```
ip address add 10.0.0.3 dev eth1
ip address add 10.0.2.3 dev eth1
ip address add 10.0.1.1 dev eth1
ip route change 10.0.0.2/32 src 10.0.0.3 via 169.254.69.30 dev eth1
ip route change 10.0.0.4/30 src 10.0.0.3 via 169.254.94.223 dev eth1
ip route change 10.0.2.2/32 src 10.0.2.3 via 169.254.69.30 dev eth1
ip route change 10.0.1.0/32 src 10.0.1.1 via 169.254.69.30 dev eth1
```

I percorsi che 𝛽<sub>B</sub> aveva memorizzati come nodo con indirizzo 0·1·2 valgono ora per il suo indirizzo 0·1·1.
Il nodo 𝛽<sub>B</sub> ha già completato la fase di bootstrap. Quindi non chiede ETP ai suoi vicini.

Il nodo 𝛽<sub>B</sub> comunica via ETP ai suoi vicini 𝛼 e 𝛾 che non è più possibile raggiungere tramite lui
l'indirizzo 0·1·2; ma che ora è possibile raggiungere tramite lui l'indirizzo 0·1·1.

L'effetto di questo ETP, che contiene solo modifiche a percorsi verso singoli nodi, è comunque limitato ai
nodi del g-nodo di livello 1, cioè al solo 𝛼.

Quindi diamo questi comandi:

**sistema 𝛼**
```
ip route add 10.0.0.3/32 src 10.0.0.2 via 169.254.96.141 dev eth1
ip route add 10.0.2.3/32 src 10.0.2.2 via 169.254.96.141 dev eth1
ip route add 10.0.1.1/32 src 10.0.1.0 via 169.254.96.141 dev eth1
```

Di nuovo, il sistema 𝛽 è in grado di comunicare con gli altri sistemi della rete.

Proseguiamo con il [passo 6](Step6.md).

