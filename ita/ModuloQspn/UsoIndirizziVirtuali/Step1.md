# Modulo QSPN - Esempio di uso degli indirizzi virtuali

## Premessa

In ogni sistema, il demone *ntkd*, quando inizia la sua attività, si sceglie un identificativo casuale in
uno spazio sufficientemente grande per supporre che sia univoco a livello dei domini di collisione in cui
partecipa con le sue interfacce di rete. Un intero di 32 bit è più che sufficiente.

Inoltre il demone *ntkd*, per ogni interfaccia di rete che gestisce nel sistema, attiva l'interfaccia di
rete e si assegna un indirizzo [link-local](http://en.wikipedia.org/wiki/Link-local_address) in IPv4
scelto a caso. Questo indirizzo serve alle comunicazioni del demone *ntkd* coi sistemi diretti vicini. E anche
per individuare il gateway per una destinazione nelle tabelle di routing. L'associazione tra
l'indirizzo MAC di una interfaccia di rete e il suo indirizzo IP link-local rimane la stessa per tutta
la durata della vita del demone *ntkd*.

Può succedere che il demone *ntkd* in un sistema, per quanto riguarda in particolare le operazioni del modulo QSPN,
assuma diverse identità. Ad esempio il sistema 𝛽 che era nel g-nodo *g* e migra in *h* vuole mantenere una
posizione in *g* e prenderne una nuova in *h*. In questo momento, il demone *ntkd* crea una sua nuova
identità, per la quale si sceglie un altro identificativo casuale. Inoltre il demone *ntkd*, sopra ogni
interfaccia di rete che gestisce, realizza una nuova pseudo-interfaccia con un distinto indirizzo MAC e un
distinto indirizzo IP link-local. Infine, per ogni arco che la precedente identità del demone aveva con un
sistema diretto vicino, la nuova identità ne crea un duplicato.

Avviene inoltre, una sorta di scambio di attributi degli archi tra le due identità. In questo senso. La
vecchia identità assume ora la gestione della posizione che il sistema 𝛽 mantiene in *g*. La nuova identità
assume la gestione della posizione nuova che il sistema 𝛽 prende in *h*. Il controllo delle interfacce di rete
che prima erano gestite dalla vecchia identità passa ora alla nuova identità. La vecchia identità gestisce
ora le pseudo-interfacce create prima, le quali infatti sono temporanee.

Gli archi che erano gestiti dalla vecchia identità rimangono gestiti dalla vecchia identità. Non cambia
il loro *identificativo di arco*, che come sappiamo è un concetto interno al modulo QSPN e di cui il suo
utilizzatore non si deve occupare. Per ognuno di questi archi, però, cambia l'interfaccia di rete su cui
è appoggiato l'arco nel sistema 𝛽 e il relativo indirizzo IP link-local. Infatti ora la vecchia identità
gestisce le pseudo-interfacce create prima, con i loro indirizzi IP link-local. Per quanto riguarda
l'interfaccia di rete su cui è appoggiato l'arco nel vicino 𝛾 all'altro estremo, qualora a migrare è un
intero g-nodo e il vicino 𝛾 ne fa parte, anch'essa diventa la nuova pseudo-interfaccia che anche il sistema
𝛾 ha creato, con il relativo indirizzo IP link-local. Altrimenti resta la stessa.

Gli archi duplicati, invece, mantengono le interfacce di rete su cui sono appoggiati da entrambi gli estremi
e i relativi indirizzi IP link-local. Quando tali archi sono passati alla nuova identità del modulo QSPN,
ricevono anche un nuovo *identificativo di arco* ciascuno.

### Motivazioni della pseudo-interfaccia di rete

Tutte le comunicazioni che i sistemi si scambiano tra i moduli del demone *ntkd*, sia quelle in UDP broadcast,
sia quelle in UDP unicast, sia quelle in TCP, sono tali che il sistema che le riceve è in grado di identificare
l'*arco-identità* su cui sono state inviate (intendendo con *arco-identità* quella entità logica che è
realizzata sopra un dato arco fisico e indica esattamente una specifica identità del sistema corrente e una specifica
identità del sistema collegato). Inoltre le comunicazioni in broadcast contengono informazioni sufficienti a riconoscere
quali identificativi devono considerarsi tra i destinatari del messaggio e quali invece devono ignorarlo. Consideriamo due
sistemi vicini, 𝛼 e 𝛽, collegati tra loro sopra un unico segmento di rete, cioè da un solo arco fisico. Supponiamo
che entrambi abbiano due identità: 𝛼<sub>0</sub>, 𝛽<sub>0</sub>, 𝛼<sub>1</sub> e 𝛽<sub>1</sub>. Le
caratteristiche appena descritte fanno si che sia possibile per 𝛼<sub>0</sub> inviare un messaggio a 𝛽<sub>1</sub>
e accertarsi che venga processato dalla giusta identità di 𝛽 e che questi sappia identificare la giusta identità
di 𝛼 come mittente del messaggio. Inoltre è possibile per 𝛼<sub>0</sub> scegliere di avere solo un arco-identità verso
𝛽<sub>0</sub> oppure solo un arco-identità verso 𝛽<sub>1</sub>, oppure entrambi, oppure nessuno: in ogni caso anche un
messaggio inviato una sola volta in broadcast sull'arco fisico può essere preparato in modo tale che solo le
giuste identità di 𝛽 lo processino.

In conclusione, i messaggi che due sistemi vicini si scambiano tra i moduli del demone *ntkd* possono raggiungere
la corretta identità dei sistemi senza bisogno di creare nuove pseudo-interfacce di rete.

Ma ci sono dei motivi per cui si rende necessario creare nuove pseudo-interfacce allo scopo di individuare
la giusta identità del sistema alla quale è stato trasmesso un generico pacchetto IP da inoltrare. Si individuano questi motivi:

*   Corretto instradamento di pacchetti IP che hanno come destinazione un indirizzo *interno* ad un g-nodo.
*   Realizzazione di un circuito fisso tra due end-point, per lo sfruttamento dei percorsi disgiunti acquisiti.

### Distinti stack di network

Quando in un sistema 𝛽 una identità 𝛽<sub>0</sub> determina la creazione di una identità aggiuntiva 𝛽<sub>1</sub>,
oltre a creare delle pseudo-interfacce apposite coi loro nuovi indirizzi MAC, il sistema 𝛽 crea un nuovo completo stack
di network, o network namespace. Questo permette alle due distinte identità, le quali esistono in distinti
g-nodi *g<sub>0</sub>* e *g<sub>1</sub>* di livello *j*, di avere indirizzi IP *interni* al g-nodo di livello *j*
che non si influenzino a vicenda.

Un pacchetto IP che ha per destinazione un indirizzo IP *interno* al g-nodo *g<sub>0</sub>* può giungere al
sistema 𝛽 attraverso una delle pseudo-interfacce di *𝛽<sub>0</sub>*. Quindi seguirà le regole iscritte nel network
namespace gestito dall'identità 𝛽<sub>0</sub>.  
Anche se l'identità 𝛽<sub>1</sub> dovesse avere in *g<sub>1</sub>* lo stesso indirizzo IP *interno* che il
pacchetto IP riporta come indirizzo destinazione (oppure sorgente) questo non interferirà con le operazioni che
il network namespace di 𝛽<sub>0</sub> porta avanti.

## Passo 1
Consideriamo un grafo connesso *G* = (*V*, *E*).

*   *V* = {𝛼, 𝛽, 𝛾, 𝛿, 𝜇}
*   *E* = {𝛼-𝛽, 𝛽-𝛾, 𝛾-𝛿, 𝛿-𝜇}

![grafo1](img/Step1/grafo1.png)

Ogni elemento di *V* è un sistema. Diciamo, per semplicità, che ogni sistema ha una interfaccia di rete, che chiamiamo "eth1".

Immaginiamola come una interfaccia radio, nel senso che tramite essa il singolo sistema rileva soltanto gli altri
sistemi che sono sufficientemente vicini. Con questo intendo dire che, ad esempio, il sistema 𝛽 con la sua interfaccia
"eth1" è direttamente collegato al sistema 𝛼 e al sistema 𝛾. Invece il sistema 𝛼 non è direttamente collegato al sistema 𝛾.

Scriviamo l'elenco degli indirizzi link-local che i sistemi si sono assegnati sull'interfaccia "eth1":

*   *IP(𝛼,eth1)* = 169.254.69.30
*   *IP(𝛽,eth1)* = 169.254.96.141
*   *IP(𝛾,eth1)* = 169.254.94.223
*   *IP(𝛿,eth1)* = 169.254.253.216
*   *IP(𝜇,eth1)* = 169.254.119.176

Ricordiamo l'elenco degli archi attualmente formatisi:

*   𝛼-𝛽
*   𝛽-𝛾
*   𝛾-𝛿
*   𝛿-𝜇

Per ogni arco che realizza, inoltre, ogni sistema aggiunge una rotta verso i diretti vicini.

Diamo questi comandi ai sistemi:

**sistema 𝛼**
```
ip link set eth1 up
ip address add 169.254.69.30 dev eth1
ip route add 169.254.96.141 dev eth1 src 169.254.69.30
```
**sistema 𝛽**
```
ip link set eth1 up
ip address add 169.254.96.141 dev eth1
ip route add 169.254.69.30 dev eth1 src 169.254.96.141
ip route add 169.254.94.223 dev eth1 src 169.254.96.141
```
**sistema 𝛾**
```
ip link set eth1 up
ip address add 169.254.94.223 dev eth1
ip route add 169.254.96.141 dev eth1 src 169.254.94.223
ip route add 169.254.253.216 dev eth1 src 169.254.94.223
```
**sistema 𝛿**
```
ip link set eth1 up
ip address add 169.254.253.216 dev eth1
ip route add 169.254.94.223 dev eth1 src 169.254.253.216
ip route add 169.254.119.176 dev eth1 src 169.254.253.216
```
**sistema 𝜇**
```
ip link set eth1 up
ip address add 169.254.119.176 dev eth1
ip route add 169.254.253.216 dev eth1 src 169.254.119.176
```

Proseguiamo con il [passo 2](Step2.md).

