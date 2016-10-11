# Proof of concept - Eventi - Pagina 2

[Pagina precedente](Eventi.md)

## Ingresso di un singolo nodo (rete a sè) in una diversa rete

Prendiamo ad esempio i valori dell'ingresso di *𝛽* in *G<sub>𝛾</sub>*.

Questo ingresso avviene per via di un nuovo arco rilevato dal modulo Neighborhood tra i sistemi
*𝛽* e *𝛾*.

Il nodo *𝛽<sub>0</sub>* era da solo e aveva indirizzo 1·0·1·0 in *G<sub>𝛽</sub>*. Con questa operazione
di ingresso *𝛽<sub>0</sub>* assume indirizzo *di connettività* 1·0·1·2 in *G<sub>𝛽</sub>*. Temporaneamente
*𝛽<sub>1</sub>* assume indirizzo *virtuale* 2·1·1·2 in *G<sub>𝛾</sub>*. Dopo poco *𝛽<sub>1</sub>* assume
indirizzo 2·1·1·1 in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝛽<sub>0</sub>* viene dismesso.

**sistema 𝛽**
```
eth1         00:16:3E:EC:A3:E1   169.254.96.141
entr02_eth1  00:16:3E:8E:91:B9   169.254.215.29
```

**sistema 𝛾**
```
eth1         00:16:3E:5B:78:D5   169.254.94.223
```

### Formazione dell'arco

**sistema 𝛽**
```
ip route add 169.254.94.223 dev eth1 src 169.254.96.141
```

Questa sequenza di operazioni è eseguita dal modulo Neighborhood (attraverso un delegato fornito dal
programma **qspnclient**) quando l'interfaccia di rete di un altro sistema è raggiungibile.
La stessa cosa avviene nel sistema opposto:

**sistema 𝛾**
```
ip route add 169.254.96.141 dev eth1 src 169.254.94.223
```

### Spostamento vecchia identità in nuovo network namespace

**sistema 𝛽**
```
ip netns add entr02
ip netns exec entr02 sysctl net.ipv4.ip_forward=1
ip netns exec entr02 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr02_eth1 link eth1 type macvlan
ip link set dev entr02_eth1 netns entr02
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.rp_filter=0
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.arp_ignore=1
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.arp_announce=2
ip netns exec entr02 ip link set dev entr02_eth1 up
ip netns exec entr02 ip address add 169.254.215.29 dev entr02_eth1
ip netns exec entr02 ip route add 169.254.94.223 dev entr02_eth1 src 169.254.215.29
```

**sistema 𝛾**
```
ip route add 169.254.215.29 dev eth1 src 169.254.94.223
```

Questa sequenza di operazioni è eseguita dal modulo Identities di *𝛽* su richiesta del programma **qspnclient** quando
l'utente comanda di avviare l'ingresso come singolo nodo in altra rete. In questa prima parte di operazioni
si crea il nuovo network namespace in cui si trasferirà ad operare (per breve tempo) la vecchia
identità.

Il nome `entr02` è un nome che il modulo Identities sceglie in questo momento ed è
sufficiente che sia inutilizzato in questo sistema. Ad esempio *ntkvX* dove *X* è un contatore
autoincrementante unico per il processo.

Anche le altre variabili sono di competenza del modulo Identities, il quale conosce quali
interfacce reali sono gestite e quali archi vanno duplicati.

#### Copia tabelle e regole, spostamento rotte

**sistema 𝛽**
```
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/29 table ntk
ip route del 10.0.0.88/29 table ntk
ip route del 10.0.0.12/30 table ntk
ip route del 10.0.0.76/30 table ntk
ip route del 10.0.0.60/30 table ntk
ip route del 10.0.0.8/31 table ntk
ip route del 10.0.0.72/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.11/32 table ntk
ip route del 10.0.0.75/32 table ntk
ip route del 10.0.0.59/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.10
ip address del 10.0.0.10/32 dev eth1
ip address del 10.0.0.74/32 dev eth1
ip address del 10.0.0.58/32 dev eth1
ip address del 10.0.0.50/32 dev eth1
ip address del 10.0.0.40/32 dev eth1
```

Questa sequenza di operazioni è eseguita dal programma **qspnclient**, sempre quando
l'utente comanda di avviare l'ingresso come singolo nodo in altra rete, in seguito alle
operazioni viste prima. In questa seconda parte di operazioni il programma, per la vecchia identità, replica
nel nuovo network namespace la situazione che era prima nel suo vecchio network namespace
(in questo esempio dal namespace *default* al namespace *entr02*) considerando però questi cambiamenti:

*   La tabella `ntk` (per i pacchetti generati localmente) è presente solo nel network namespace default.
*   Tutti gli archi-qspn che la vecchia identità aveva restano gli stessi. Però le relative
    tabelle `ntk_from_xxx` passando da un namespace ad un altro possono cambiare nome, cioè
    se la migrazione coinvolge anche il nodo vicino cambia anche il MAC address associato al vicino.
*   L'indirizzo Netsukuku della vecchia identità assume una componente *virtuale* ad un
    livello in cui prima era *reale* e questo aggiunge una possibile destinazione a quel livello.
*   ...

Nell'esempio presente la vecchia identità non aveva nessun arco-qspn.

Inoltre il programma prepara il vecchio network namespace che dovrà ospitare
la nuova identità: cioè elimina le vecchie rotte e gli indirizzi propri e la (eventuale)
regola di source-natting.

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

Questa sequenza di operazioni è eseguita dal programma **qspnclient**, sempre quando
l'utente comanda di avviare l'ingresso come singolo nodo in altra rete, in seguito alle
operazioni viste prima. In questa terza parte di operazioni la nuova identità popola
il suo network namespace (quello preesistente ereditato dalla vecchia identità) con le
rotte che sono consone al suo nuovo indirizzo.

Siccome in questo momento trattiamo l'ingresso in nuova rete di un singolo nodo, è
certo che il vecchio namespace è il *default* e che le rotte saranno quelle di un indirizzo
che ha una sola componente *virtuale* al livello 0.

### Creazione e popolamento iniziale di tabelle per l'inoltro

**sistema 𝛽**
```
(echo; echo "250 ntk_from_00:16:3E:5B:78:D5 # xxx_table_ntk_from_00:16:3E:5B:78:D5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
```

Questa sequenza di operazioni è eseguita dal programma **qspnclient** quando un arco-identità
viene indicato al QspnManager (o nel costruttore o con il metodo `add_arc`) di una sua
identità. In essa viene aggiunto nel file `rt_tables` (se non c'era) un nuovo pseudonimo di
tabella `ntk_from_XXX`; viene istruito il kernel che i pacchetti ricevuti (in un certo network namespace)
da quell'indirizzo MAC vanno *marcati*; la tabella viene popolata con le possibili destinazioni
tutte segnate inizialmente come *irraggiungibili*. Non viene però aggiunta ancora la regola
che dice di guardare quella tabella, poiché ancora non si è ricevuto alcun ETP da questo
arco-identità.

Nel nostro caso il programma **qspnclient** di *𝛽*, sempre poiché
l'utente ha comandato di avviare l'ingresso come singolo nodo in altra rete, in seguito alle
operazioni viste prima, crea il QspnManager della nuova identità indicando l'arco-identità
con il sistema *𝛾*. Questa è la quarta parte delle operazioni.

Analogamente si ha nel sistema *𝛾* che il programma **qspnclient** comunica al QspnManager della
sua identità il nuovo arco-identità con il sistema *𝛽*.

**sistema 𝛾**
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
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

### Processazione di un ETP

**sistema 𝛽**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.22/32 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.86/32 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.62/32 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.50/32 table ntk via 169.254.94.223 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.94.223 dev eth1
ip route change unreachable 10.0.0.23/32 table ntk
ip route change unreachable 10.0.0.87/32 table ntk
ip route change unreachable 10.0.0.63/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5
```

Questa sequenza di operazioni è eseguita dal programma **qspnclient** quando un suo
QspnManager ha terminato di processare un ETP. In essa, relativamente al network namespace
associato all'identità a cui il QspnManager appartiene, vengono aggiornate tutte le rotte
della tabella `ntk` e di quelle tabelle di inoltro dai cui archi abbiamo ricevuto almeno un
ETP; in seguito viene aggiunta la regola per le tabelle di inoltro il cui arco ha ricevuto
proprio adesso il primo ETP.

Nel nostro caso il programma **qspnclient** di *𝛽* riceve per primo un ETP dal sistema *𝛾*, poiché
prima di riceverlo non può completare la fase di bootstrap.

In seguito *𝛽* trasmette un ETP. Analogamente si ha nel sistema *𝛾* che il programma **qspnclient**,
per via della processazione dell'ETP da parte del suo QspnManager, esegue queste operazioni. Notiamo
che non ci sono ancora destinazioni raggiungibili per *𝛾* perché ancora *𝛽* ha un indirizzo *virtuale*.

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.23/32 table ntk
ip route change unreachable 10.0.0.87/32 table ntk
ip route change unreachable 10.0.0.63/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip rule add fwmark 250 table ntk_from_00:16:3E:EC:A3:E1
```

### Cambio di indirizzo di una identità

#### Un identificativo passa da virtuale a reale (a livello *i*)

**sistema 𝛽**
```
ip route del 10.0.0.23/32 table ntk
ip route del 10.0.0.87/32 table ntk
ip route del 10.0.0.63/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
ip route del 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
```

Questa sequenza di operazioni è eseguita dal programma **qspnclient** quando una sua identità
cambia il suo indirizzo Netsukuku, passando uno dei suoi identificativi da *virtuale* a
*reale*. In essa si rimuovono, nel network namespace interessato, dalla tabella `ntk` e da
tutte le tabelle di inoltro (anche quelle il cui arco non ha ancora ricevuto alcun ETP) le
rotte verso gli indirizzi che non sono più validi a causa di questo nuovo identificativo *reale*.

Notiamo che questo avviene anche nelle identità *di connettività* (che gestiscono un namespace
diverso dal default) e anche se l'indirizzo Netsukuku non è del tutto *reale*.

#### L'identità interessata era la principale

**sistema 𝛽**
```
ip address add 10.0.0.41 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.63 dev eth1
ip address add 10.0.0.23 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.23
ip address add 10.0.0.87 dev eth1
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.22/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.86/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.62/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.63
ip route change 10.0.0.50/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.41
```

Questa sequenza di operazioni è eseguita, in seguito alle operazioni viste prima, dal
programma **qspnclient** quando la sua identità *principale* cambia il suo indirizzo Netsukuku
passando uno dei suoi identificativi da *virtuale* a *reale*, e solo se al livello 0 l'identificativo
è *reale*.

Per prima cosa il programma **qspnclient** aggiunge ad ogni interfaccia di rete reale nel network
namespace default gli indirizzi propri che prima non era possibile computare e adesso invece sì,
partendo dal livello più basso e salendo finché possibile.

Poi, solo se l'indirizzo è ora del tutto *reale*, aggiunge (opzionalmente) la regola di source-natting
e (opzionalmente) l'indirizzo IP anonimizzante.

Infine aggiorna tutte le rotte nella tabella `ntk` per fare in modo di mettere in esse (se disponibile) un src preferito.

Nel nostro caso, l'utente ha comandato di avviare l'ingresso come singolo nodo in altra rete specificando
che la migration path più breve è di zero passi. Cioè indicando che esiste fin da subito un
indirizzo Netsukuku *reale* libero compatibile con gli archi del nodo. Esso è `2·1·1·1`.  
Quindi il programma **qspnclient**, dopo aver fatto passare la nuova identità per l'indirizzo
*virtuale* al livello 0 `2·1·1·2`, immediatamente (o pochi istanti dopo) cambia il suo identificativo di livello 0
da `2` a `1`. Questa è la quinta parte delle operazioni.

### Processazione di un ETP, ancora

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.23/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.40
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

Di nuovo assistiamo ad una sequenza di operazioni eseguita dal programma **qspnclient** perché un suo
QspnManager ha terminato di processare un ETP. Infatti il sistema *𝛾* riceve un ETP trasmesso dal
sistema *𝛽* a seguito del suo cambio di indirizzo.

### Dismissione identità

**sistema 𝛽**
```
ip netns exec entr02 ip route flush table main
ip netns exec entr02 ip link delete entr02_eth1 type macvlan
ip netns del entr02
```

Questa sequenza di operazioni è eseguita dal modulo Identities di *𝛽* su richiesta del
programma **qspnclient**, sempre quando l'utente comanda di avviare l'ingresso come singolo nodo
in altra rete, qualche istante dopo il completamento delle operazioni viste prima. In questa
sesta parte di operazioni viene dismessa la vecchia identità del sistema *𝛽*.

Inoltre il modulo Identities lo comunica al sistema *𝛾*:

**sistema 𝛾**
```
ip route del 169.254.215.29 dev eth1 src 169.254.94.223
```

[Pagina seguente](Eventi3.md)
