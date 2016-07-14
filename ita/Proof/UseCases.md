# Proof of concept - Casi d'uso

1.  [Topologia della rete](#Topologia_della_rete)
1.  [Prime operazioni](#Prime_operazioni)

Attraverso un esame dettagliato dei possibili scenari in cui il programma si trova ad operare
e delle operazioni che devono essere fatte sulle tabelle di routing del sistema per il corretto funzionamento
della rete, giungeremo ad osservare quali operazioni e in quali momenti il programma deve fare
in risposta agli eventi che rileva.

## <a name="Topologia_della_rete"></a>Topologia della rete

Assumiamo di operare con reti Netsukuku che hanno questa comune topologia: 4·2·2·2.

```
Topologia 4·2·2·2

Globali:
10.0.0.0 ... 10.0.0.31
Anonimizzanti:
10.0.0.64 ... 10.0.0.95
Interni al livello 3
10.0.0.56 ... 10.0.0.63
Interni al livello 2
10.0.0.48 ... 10.0.0.51
Interni al livello 1
10.0.0.40 ... 10.0.0.41
```

## <a name="Prime_operazioni"></a>Prime operazioni

Partiamo dal sistema *𝛼* che ha una identità principale *𝛼<sub>0</sub>* che ha indirizzo Netsukuku 3·1·0·1
in una rete con topologia 4·2·2·2. Tale rete (cioè l'identificativo della rete che si trova nel
fingerprint al livello 4) la chiamiamo *G<sub>0</sub>*. Diciamo inoltre che tale sistema ha una sola
interfaccia di rete che chiamiamo `eth1`. Diciamo anche che questo nodo ammette la possibilità di essere
contattato in forma anonima.

```
Mio indirizzo 3·1·0·1.
     globale
      10.0.0.29
     anonimizzante
      10.0.0.93
     interno al mio g-nodo di livello 3
      10.0.0.61
     interno al mio g-nodo di livello 2
      10.0.0.49
     interno al mio g-nodo di livello 1
      10.0.0.41

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 1·
     globale
      10.0.0.8/29
     anonimizzante
      10.0.0.72/29
 2·
     globale
      10.0.0.16/29
     anonimizzante
      10.0.0.80/29
 3·0·
     globale
      10.0.0.24/30
     anonimizzante
      10.0.0.88/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 3·1·1·
     globale
      10.0.0.30/31
     anonimizzante
      10.0.0.94/31
     interno al mio g-nodo di livello 3
      10.0.0.62/31
     interno al mio g-nodo di livello 2
      10.0.0.50/31
 3·1·0·0
     globale
      10.0.0.28/32
     anonimizzante
      10.0.0.92/32
     interno al mio g-nodo di livello 3
      10.0.0.60/32
     interno al mio g-nodo di livello 2
      10.0.0.48/32
     interno al mio g-nodo di livello 1
      10.0.0.40/32
```

Fra i compiti del modulo Neighborhood, esso assegna all'interfaccia `eth1` del sistema *𝛼* un indirizzo IP
linklocal, sia ad esempio 169.254.35.112.

**sistema 𝛼**
```
ip link set eth1 up
ip address add 169.254.35.112 dev eth1
```

Il programma *qspnclient* invece ha il compito, fin dall'inizio, di assegnare alla stessa interfaccia gli
indirizzi IP che sono da associare all'identità principale *𝛼<sub>0</sub>*. Come si spiega nel documento di analisi,
in questo caso abbiamo l'indirizzo IP globale, l'indirizzo IP anonimizzante, l'indirizzo IP interno al g-nodo di
livello 3, di livello 2 e di livello 1.

**sistema 𝛼**
```
ip address add 10.0.0.29 dev eth1
ip address add 10.0.0.93 dev eth1
ip address add 10.0.0.61 dev eth1
ip address add 10.0.0.49 dev eth1
ip address add 10.0.0.41 dev eth1
```

Inoltre il programma *qspnclient* ha da subito il compito, sempre con riferimento alla sua identità principale *𝛼<sub>0</sub>*,
di creare nel network namespace default una tabella "ntk" e di aggiungere una regola che la referenzia. In tale
tabella deve anche mettere tutte le possibili destinazioni, inizialmente in stato "unreachable".

**sistema 𝛼**
```
/etc/iproute2/rt_tables: add table 251: ntk
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
ip route add unreachable 10.0.0.28/32 table ntk
ip route add unreachable 10.0.0.92/32 table ntk
ip route add unreachable 10.0.0.60/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
```

Inoltre il programma *qspnclient* ha da subito il compito, sempre con riferimento alla sua identità
principale *𝛼<sub>0</sub>*, se decide di prestarsi all'anonimizzazione dei pacchetti IP che inoltra,
di istruire il kernel a questo scopo.

**sistema 𝛼**
```
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.29
```

## Ingresso di un altro singolo nodo nella nostra rete

Ora assumiamo che un sistema *𝛽* giunga a distanza di rilevamento con la sua interfaccia di rete che
ha MAC address 00:16:3E:2D:8D:DE e indirizzo IP linklocal (assegnatogli dal modulo Neighborhood) 169.254.43.192.

Fra i compiti del modulo Neighborhood, esso aggiunge nel network namespace default la rotta diretta verso
l'indirizzo IP linklocal del vicino per ogni arco che lo stesso modulo ha realizzato.

**sistema 𝛼**
```
ip route add 169.254.43.192 dev eth1 src 169.254.35.112
```

Assumiamo che il sistema *𝛽* abbia solo una identità *𝛽<sub>0</sub>* che si trova in una diversa rete
*G<sub>1</sub>*.

Il modulo Identities ha creato l'arco-identità principale, cioè quello che collega le due identità
*𝛼<sub>0</sub>* e *𝛽<sub>0</sub>*, senza per questo aggiungere alcuna rotta, perché per tale arco-identità
la rotta è stata aggiunta dal modulo Neighborhood nel network namespace default del sistema *𝛼*.

Ora assumiamo che *𝛽<sub>0</sub>* decide di entrare in *G<sub>0</sub>*. Per essere precisi, il sistema *𝛽* decide di
costruire una nuova identità *𝛽<sub>1</sub>* partendo da *𝛽<sub>0</sub>*. Poi *𝛽<sub>1</sub>* farà ingresso nella
rete *G<sub>0</sub>*.

Quando il sistema *𝛽* crea la nuova identità, il suo modulo Identities dialoga con il modulo del sistema
*𝛼* per aggiungere l'arco-identità *𝛼<sub>0</sub>-𝛽<sub>1</sub>* e modificare i valori (peer_mac e
peer_linklocal) dell'arco-identità *𝛼<sub>0</sub>-𝛽<sub>0</sub>*. Di fatto, questo comporta che il modulo Identities
nel sistema *𝛼* aggiunge la nuova rotta, sempre nel network namespace default gestito da *𝛼<sub>0</sub>*, verso il nuovo indirizzo
linklocal assunto da *𝛽<sub>0</sub>*, assumiamo ad esempio 169.254.101.161.

**sistema 𝛼**
```
ip route add 169.254.101.161 dev eth1 src 169.254.35.112
```

Ora il sistema *𝛽* fa entrare *𝛽<sub>1</sub>* in *G<sub>0</sub>* e, contemporaneamente, il sistema *𝛼* comunica
alla sua identità *𝛼<sub>0</sub>* che sull'arco-identità *𝛼<sub>0</sub>-𝛽<sub>1</sub>* va costruito un QspnArc.

Questo nuovo QspnArc che viene comunicato al modulo QSPN del sistema *𝛼* per l'identità *𝛼<sub>0</sub>*, inizialmente
non comporta operazioni sulle tabelle di routing, fino a quando non si riceve il primo ETP attraverso questo
arco e così si scopre l'indirizzo Netsukuku di questo vicino.

Dopo un po' di tempo il nodo *𝛼<sub>0</sub>* riceverà un ETP dal QspnArc *𝛼<sub>0</sub>-𝛽<sub>1</sub>* e con esso
aggiornerà l'indirizzo Netsukuku di questo vicino. Questo deve produrre due macro-operazioni nel sistema *𝛼*: la creazione di
una tabella per i pacchetti IP ricevuti dall'arco *𝛼<sub>0</sub>-𝛽<sub>1</sub>* e l'aggiornamento (su tutte le tabelle) delle
rotte che adesso possono avere come gateway l'arco *𝛼<sub>0</sub>-𝛽<sub>1</sub>*.

Assumiamo che l'indirizzo Netsukuku di *𝛽<sub>1</sub>* sia 3·1·0·0.

**sistema 𝛼**
```
/etc/iproute2/rt_tables: add table 250: ntk_from_00:16:3E:2D:8D:DE
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:2D:8D:DE -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
```

**sistema 𝛼**
```
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE

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
ip route change 10.0.0.28/32 table ntk via 169.254.43.192 dev eth1 src 10.0.0.29
ip route change 10.0.0.92/32 table ntk via 169.254.43.192 dev eth1 src 10.0.0.29
ip route change 10.0.0.60/32 table ntk via 169.254.43.192 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.43.192 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.43.192 dev eth1 src 10.0.0.41
```

Poi il sistema *𝛽* rimuoverà l'identità *𝛽<sub>0</sub>* con tutti i suoi archi-identità.
La cosa verrà comunicata al modulo Identities del sistema *𝛼* per via dell'arco-identità
*𝛼<sub>0</sub>-𝛽<sub>0</sub>*. Questo produce la rimozione della rotta.

**sistema 𝛼**
```
ip route del 169.254.101.161 dev eth1 src 169.254.35.112
```

Ora assumiamo che un sistema *𝛾* giunga a distanza di rilevamento con la sua interfaccia di rete che
ha MAC address 00:16:3E:5B:78:D5 e indirizzo IP linklocal (assegnatogli dal modulo Neighborhood) 169.254.103.81.

Fra i compiti del modulo Neighborhood, esso aggiunge nel network namespace default la rotta diretta verso
l'indirizzo IP linklocal del vicino per ogni arco che lo stesso modulo ha realizzato.

**sistema 𝛼**
```
ip route add 169.254.103.81 dev eth1 src 169.254.35.112
```

Assumiamo che il sistema *𝛾* abbia solo una identità *𝛾<sub>0</sub>* che si trova in una diversa rete
*G<sub>2</sub>*.

Il modulo Identities ha creato l'arco-identità principale, cioè quello che collega le due identità
*𝛼<sub>0</sub>* e *𝛾<sub>0</sub>*, senza per questo aggiungere alcuna rotta, perché per tale arco-identità
la rotta è stata aggiunta dal modulo Neighborhood nel network namespace default del sistema *𝛼*.

Ora assumiamo che *𝛼<sub>0</sub>* decide di entrare in *G<sub>2</sub>*. Per essere precisi, il sistema *𝛼* decide di
costruire una nuova identità *𝛼<sub>1</sub>* partendo da *𝛼<sub>0</sub>*. Questa nuova identità scaturisce dalla
migrazione del g-nodo *𝜑*, di livello 1 e di indirizzo Netsukuku 3·1·0·, che comprende anche il vicino
*𝛽<sub>1</sub>*. Poi *𝛼<sub>1</sub>* farà ingresso in *G<sub>2</sub>* come membro del g-nodo *𝜑*.

All'inizio viene creato nel sistema *𝛼* un nuovo network namespace "ntkv0" e in esso viene creata
una pseudo-interfaccia "ntkv0_eth1" sopra l'interfaccia reale "eth1". Questo nuovo network namespace
sarà gestito da *𝛼<sub>0</sub>* mentre quello precedente (il default) verrà gestito da *𝛼<sub>1</sub>*.  
Assumiamo che alla nuova pseudo-interfaccia il modulo Identities assegna l'indirizzo IP linklocal 169.254.83.167.  
Ricordiamo inoltre che una identità di connettività non detiene (nel suo network namespace che non è
il default) alcun indirizzo IP associato al suo indirizzo Netsukuku.

**sistema 𝛼**
```
ip netns add ntkv0
ip link add dev ntkv0_eth1 link eth1 type macvlan
ip link set dev ntkv0_eth1 netns ntkv0
ip netns exec ntkv0 ip link set dev ntkv0_eth1 up
ip netns exec ntkv0 ip address add 169.254.83.167 dev ntkv0_eth1
```

Anche nel sistema *𝛽* partendo da *𝛽<sub>1</sub>* è stata creata una nuova identità *𝛽<sub>2</sub>*.  
Assumiamo che la nuova pseudo-interfaccia gestita ora da *𝛽<sub>1</sub>* prende MAC address 00:16:3E:DF:23:F5 e
indirizzo IP linklocal 169.254.242.91. La vecchia è gestita ora da *𝛽<sub>2</sub>*.  
Il modulo Identities del sistema *𝛼*, dal dialogo con i vicini, desume che vanno creati/modificati questi
archi-identità:

*   *𝛼<sub>0</sub>-𝛽<sub>1</sub>*.  
    Questo arco-identità vede cambiare il `peer_mac` e `peer_linklocal`. Inoltre la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝛼<sub>0</sub>*
    la rotta verso il vicino.  
    I nodi *𝛼<sub>0</sub>* e *𝛽<sub>1</sub>* fanno parte della stessa rete, quindi è prevista una tabella
    per i pacchetti IP da inoltrare che provengono da questo arco. Nel nuovo network namespace gestito da *𝛼<sub>0</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:DF:23:F5` e la relativa regola.
*   *𝛼<sub>1</sub>-𝛽<sub>2</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝛼<sub>1</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝛼<sub>1</sub>* e *𝛽<sub>2</sub>* fanno parte della stessa rete, quindi è prevista una tabella
    per i pacchetti IP da inoltrare che provengono da questo arco. Nel network namespace gestito da *𝛼<sub>1</sub>*
    la tabella `ntk_from_00:16:3E:2D:8D:DE` c'è già, e anche la relativa regola.
*   *𝛼<sub>0</sub>-𝛾<sub>0</sub>*.  
    Per questo arco-identità abbiamo che la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝛼<sub>0</sub>*
    la rotta verso il vicino.  
    I nodi *𝛼<sub>0</sub>* e *𝛾<sub>0</sub>* non fanno parte della stessa rete, quindi non è prevista una tabella
    `ntk_from_xxx`.
*   *𝛼<sub>1</sub>-𝛾<sub>0</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝛼<sub>1</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝛼<sub>1</sub>* e *𝛾<sub>0</sub>* faranno parte della stessa rete, ma solo dopo che il nodo *𝛼<sub>1</sub>*
    avrà costruito la nuova istanza di QspnManager; per fare questo il sistema *𝛼* dovrà costruire un QspnArc
    sull'arco-identità *𝛼<sub>1</sub>-𝛾<sub>0</sub>*. Questo nuovo QspnArc inizialmente non comporta operazioni sulle
    tabelle di routing, fino a quando non si riceve il primo ETP attraverso questo arco e così si scopre l'indirizzo
    Netsukuku di questo vicino. Solo a quel punto, ad esempio, nel network namespace gestito da *𝛼<sub>1</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:5B:78:D5` e la relativa regola.

Il modulo Identities fa queste operazioni:

**sistema 𝛼**
```
ip netns exec ntkv0 ip route add 169.254.242.91 dev ntkv0_eth1 src 169.254.83.167
ip netns exec ntkv0 ip route add 169.254.103.81 dev ntkv0_eth1 src 169.254.83.167
```

Il programma *qspnclient* fa queste operazioni preliminari:

**sistema 𝛼**
```
ip netns exec ntkv0 ip rule add table ntk
/etc/iproute2/rt_tables: add table 249: ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:DF:23:F5 -j MARK --set-mark 249
ip netns exec ntkv0 ip rule add fwmark 249 table ntk_from_00:16:3E:DF:23:F5
```

Il programma *qspnclient* fa queste operazioni sulle rotte verso g-nodi di livello *k*, con `k < 1`, cioè
verso destinazioni interne al g-nodo che ha migrato:

*   Quelle espresse con indirizzi IP interni ad un g-nodo fino al livello 1, vanno mantenute nel network namespace
    vecchio e copiate nel network namespace nuovo.
*   Quelle espresse con indirizzi IP globali o interni ad un g-nodo di livello superiore, vanno rimosse dal
    network namespace vecchio.

**sistema 𝛼**
```
ip netns exec ntkv0 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5

ip route del 10.0.0.28/32 table ntk
ip route del 10.0.0.92/32 table ntk
ip route del 10.0.0.60/32 table ntk
ip route del 10.0.0.48/32 table ntk
ip route del 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
```

Il programma *qspnclient* fa queste operazioni sulle rotte verso g-nodi di livello *k*, con `k ≥ 1`, cioè
verso destinazioni esterne al g-nodo che ha migrato:

*   Tutte (siano esse espresse con indirizzi IP globali o interni ad un g-nodo) vanno spostate dal network namespace
    vecchio al network namespace nuovo.

**sistema 𝛼**
```
ip netns exec ntkv0 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.24/30 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.88/30 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.30/31 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.94/31 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.50/31 table ntk
ip netns exec ntkv0 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5

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
ip route del 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
```

Il programma *qspnclient* rimuove l'indirizzo globale e gli indirizzi interni ai propri g-nodi di livello
maggiore di 1 dal network namespace vecchio.

**sistema 𝛼**
```
ip address del 10.0.0.29/32 dev eth1
ip address del 10.0.0.93/32 dev eth1
ip address del 10.0.0.61/32 dev eth1
ip address del 10.0.0.49/32 dev eth1
```

Il programma *qspnclient* aggiorna le rotte nel nuovo network namespace sulla base dei migliori percorsi
noti alla vecchia identità *𝛼<sub>0</sub>*.

**sistema 𝛼**
```
ip netns exec ntkv0 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.24/30 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.88/30 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.56/30 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.30/31 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.94/31 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec ntkv0 ip route change unreachable 10.0.0.50/31 table ntk
ip netns exec ntkv0 ip route change 10.0.0.40/32 table ntk via 169.254.242.91 dev ntkv0_eth1

ip netns exec ntkv0 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec ntkv0 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5
```

