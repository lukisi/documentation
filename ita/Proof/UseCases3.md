# Proof of concept - Casi d'uso - Parte 3

## <a name="Ingresso_gnodo_altra_rete"></a>Ingresso come g-nodo in un'altra rete

Ora assumiamo che il sistema *𝛾* giunga a distanza di rilevamento con la sua interfaccia di rete
per l'interfaccia di rete del sistema *𝛿*.

Fra i compiti del modulo Neighborhood, esso aggiunge nel network namespace default la rotta diretta verso
l'indirizzo IP linklocal del vicino per ogni arco che lo stesso modulo ha realizzato.

**sistema 𝛿**
```
ip route add 169.254.94.223 dev eth1 src 169.254.253.216
```

Assumiamo che il sistema *𝛾* abbia solo una identità *𝛾<sub>0</sub>* che si trova in una diversa rete
*G<sub>𝛾</sub>*.

Il modulo Identities ha creato l'arco-identità principale, cioè quello che collega le due identità
*𝛿<sub>0</sub>* e *𝛾<sub>0</sub>*, senza per questo aggiungere alcuna rotta, perché per tale arco-identità
la rotta è stata aggiunta dal modulo Neighborhood nel network namespace default del sistema *𝛿*.

Ora assumiamo che il g-nodo *𝜒* di livello 1 e di indirizzo Netsukuku 3·1·0·, che comprende *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>*,
decide di entrare in *G<sub>𝛾</sub>*. Per essere precisi, decide di costruire un g-nodo isomorfo
*𝜒'* costituito dalle nuove identità *𝛿<sub>1</sub>* e *𝜇<sub>2</sub>*. Il g-nodo
*𝜒* assume indirizzo *di connettività* 3·1·2· in *G<sub>𝛿</sub>*. Temporaneamente
*𝜒'* assume indirizzo *virtuale* 2·1·2· in *G<sub>𝛾</sub>*.  
Esaminiamo cosa avviene con riferimento al singolo nodo *𝛿<sub>0</sub>* nel sistema *𝛿*.

Il sistema *𝛿* costruisce una nuova identità *𝛿<sub>1</sub>* partendo da *𝛿<sub>0</sub>*.
Viene creato nel sistema *𝛿* un nuovo network namespace "entr03" e in esso viene creata
una pseudo-interfaccia "entr03_eth1" sopra l'interfaccia reale "eth1". Questo nuovo network namespace
sarà gestito da *𝛿<sub>0</sub>* mentre quello precedente (il default) verrà gestito da *𝛿<sub>1</sub>*.  
Ricordiamo che una identità di connettività quale è ora *𝛿<sub>0</sub>*, nel suo network namespace che non è
il default, non detiene alcun indirizzo IP associato al suo indirizzo Netsukuku.

**sistema 𝛿**
```
ip netns add entr03
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 ip link set dev entr03_eth1 address 00:16:3E:B9:77:80
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.83.167 dev entr03_eth1
```

Anche nel sistema *𝜇* partendo da *𝜇<sub>1</sub>* è stata creata una nuova identità *𝜇<sub>2</sub>*.  
L'identità *𝜇<sub>1</sub>* verrà temporaneamente spostata sul network namespace "entr03" del sistema *𝜇*.
Il namespace default è gestito ora da *𝜇<sub>2</sub>*.  
Il modulo Identities del sistema *𝛿*, dal dialogo con i vicini, desume che vanno creati/modificati questi
archi-identità:

*   *𝛿<sub>0</sub>-𝜇<sub>1</sub>*.  
    Questo arco-identità vede cambiare il `peer_mac` e `peer_linklocal`. Inoltre la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝛿<sub>0</sub>*
    la rotta verso il vicino.  
    I nodi *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>* fanno parte della stessa rete, quindi è prevista una tabella
    per i pacchetti IP da inoltrare che provengono da questo arco. Nel nuovo network namespace gestito da *𝛿<sub>0</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:DF:23:F5` e la relativa regola.
*   *𝛿<sub>1</sub>-𝜇<sub>2</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝛿<sub>1</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝛿<sub>1</sub>* e *𝜇<sub>2</sub>* fanno parte della stessa rete, quindi è prevista una tabella
    per i pacchetti IP da inoltrare che provengono da questo arco. Nel network namespace gestito da *𝛿<sub>1</sub>*
    la tabella `ntk_from_00:16:3E:2D:8D:DE` c'è già, e anche la relativa regola.
*   *𝛿<sub>0</sub>-𝛾<sub>0</sub>*.  
    Per questo arco-identità abbiamo che la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝛿<sub>0</sub>*
    la rotta verso il vicino.  
    I nodi *𝛿<sub>0</sub>* e *𝛾<sub>0</sub>* non fanno parte della stessa rete, quindi non è prevista una tabella
    `ntk_from_xxx`.
*   *𝛿<sub>1</sub>-𝛾<sub>0</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝛿<sub>1</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝛿<sub>1</sub>* e *𝛾<sub>0</sub>* faranno parte della stessa rete, ma solo dopo che il nodo *𝛿<sub>1</sub>*
    avrà costruito la nuova istanza di QspnManager; per fare questo il sistema *𝛿* dovrà costruire un QspnArc
    sull'arco-identità *𝛿<sub>1</sub>-𝛾<sub>0</sub>*. Questo nuovo QspnArc inizialmente non comporta operazioni sulle
    tabelle di routing, fino a quando non si riceve il primo ETP attraverso questo arco e così si scopre l'indirizzo
    Netsukuku di questo vicino. Solo a quel punto, ad esempio, nel network namespace gestito da *𝛿<sub>1</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:5B:78:D5` e la relativa regola.

Il modulo Identities fa queste operazioni:

**sistema 𝛿**
```
ip netns exec entr03 ip route add 169.254.242.91 dev entr03_eth1 src 169.254.83.167
ip netns exec entr03 ip route add 169.254.94.223 dev entr03_eth1 src 169.254.83.167
```

Il programma *qspnclient* fa queste operazioni preliminari:

**sistema 𝛿**
```
ip netns exec entr03 ip rule add table ntk
(echo; echo "249 ntk_from_00:16:3E:DF:23:F5 # xxx_table_ntk_from_00:16:3E:DF:23:F5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:DF:23:F5 -j MARK --set-mark 249
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:DF:23:F5
```

Il programma *qspnclient* fa queste operazioni sulle rotte verso g-nodi di livello *k*, con `k < 1`, cioè
verso destinazioni interne al g-nodo che ha migrato:

*   Quelle espresse con indirizzi IP interni ad un g-nodo fino al livello 1, vanno mantenute nel network namespace
    vecchio e copiate nel network namespace nuovo.
*   Quelle espresse con indirizzi IP globali o interni ad un g-nodo di livello superiore, vanno rimosse dal
    network namespace vecchio.

**sistema 𝛿**
```
ip netns exec entr03 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5

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

Avendo però preso per la vecchia identità un indirizzo Netsukuku con posizione *virtuale* nel
livello 1, dove prima aveva posizione *reale*, abbiamo che esiste una ulteriore possibile
destinazione da aggiungere nel nuovo network namespace. La 3·1·0·.

```
 3·1·0·
     globale
      10.0.0.28/31
     anonimizzante
      10.0.0.92/31
     interno al mio g-nodo di livello 3
      10.0.0.60/31
     interno al mio g-nodo di livello 2
      10.0.0.48/31
```

**sistema 𝛿**
```
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:DF:23:F5

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

**sistema 𝛿**
```
ip address del 10.0.0.29/32 dev eth1
ip address del 10.0.0.93/32 dev eth1
ip address del 10.0.0.61/32 dev eth1
ip address del 10.0.0.49/32 dev eth1
```

Il programma *qspnclient* aggiorna le rotte nel nuovo network namespace sulla base dei migliori percorsi
noti alla vecchia identità *𝛿<sub>0</sub>*.

**sistema 𝛿**
```
ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr03 ip route change 10.0.0.40/32 table ntk via 169.254.242.91 dev entr03_eth1

ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5
```

Poi il sistema *𝛿* per la nuova identità *𝛿<sub>1</sub>* istanzia un QspnManager con il
costruttore `enter_net` passandogli un QspnArc per l'arco-identità *𝛿<sub>1</sub>-𝜇<sub>2</sub>*
e uno per l'arco-identità *𝛿<sub>1</sub>-𝛾<sub>0</sub>*. L'indirizzo Netsukuku del nodo
*𝛿<sub>1</sub>* sarà 2·1·2·1: cioè l'indirizzo di *𝜒'* con la parte finale che era già di
*𝛿<sub>0</sub>* in *𝜒*.

Siccome la nuova identità *𝛿<sub>1</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku. Come si spiega nel documento di analisi,
in questo caso abbiamo solo l'indirizzo IP interno al g-nodo di livello 1.  
Deve però tenere presente che alcuni di essi possono essere rimasti assegnati dal
precedente detentore del network namespace default. In questo caso tutti; quindi non ci
sono operazioni da fare.

Inoltre il programma *qspnclient* deve assicurarsi che nel network namespace della nuova identità *𝛿<sub>1</sub>*
siano presenti le regole per la consultazione della tabella `ntk` e delle tabelle `ntk_from_xxx` associate agli
archi-identità di *𝛿<sub>1</sub>* per i quali conosciamo già l'indirizzo Netsukuku del peer. E non altre regole.

Deve poi aggiungere in queste tabelle (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝛿<sub>1</sub>*, inizialmente in stato "unreachable". Deve anche fare in modo che le operazioni di aggiunta
delle destinazioni vengano completate tutte prima di elaborare operazioni di aggiornamento delle rotte: infatti
queste potrebbero essere avviate, ad esempio, dopo il segnale `bootstrap_complete` che viene notificato da un'altra tasklet.

```
Mio indirizzo 2·1·2·1.
     globale
      N/A
     anonimizzante
      N/A
     interno al mio g-nodo di livello 3
      N/A
     interno al mio g-nodo di livello 2
      N/A
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
 3·
     globale
      10.0.0.24/29
     anonimizzante
      10.0.0.88/29
 2·0·
     globale
      10.0.0.16/30
     anonimizzante
      10.0.0.80/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 2·1·0·
     globale
      10.0.0.20/31
     anonimizzante
      10.0.0.84/31
     interno al mio g-nodo di livello 3
      10.0.0.60/31
     interno al mio g-nodo di livello 2
      10.0.0.48/31
 2·1·1·
     globale
      10.0.0.22/31
     anonimizzante
      10.0.0.86/31
     interno al mio g-nodo di livello 3
      10.0.0.62/31
     interno al mio g-nodo di livello 2
      10.0.0.50/31
 2·1·2·0
     globale
      N/A
     anonimizzante
      N/A
     interno al mio g-nodo di livello 3
      N/A
     interno al mio g-nodo di livello 2
      N/A
     interno al mio g-nodo di livello 1
      10.0.0.40/32
```

**sistema 𝛿**
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
ip route add unreachable 10.0.0.22/31 table ntk
ip route add unreachable 10.0.0.86/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
```

Dopo un po' di tempo avverranno 2 eventi per *𝛿<sub>1</sub>*, dei quali non possiamo a priori
sapere l'ordine temporale:

*   il QspnManager di *𝛿<sub>1</sub>* emetterà il segnale di `bootstrap_complete` dopo aver ricevuto un ETP dal
    vicino *𝛾<sub>0</sub>*.
*   il programma *qspnclient* nel sistema *𝛿* viene notificato che l'indirizzo Netsukuku di *𝜒'* in *G<sub>𝛾</sub>*, che
    temporaneamente era il *virtuale* 2·1·2·, diventa il *reale* (2·1·0·). Di conseguenza il programma comunica al
    QspnManager di *𝛿<sub>1</sub>* che ora il suo indirizzo Netsukuku è 2·1·0·1.

Assumiamo che si verifichi prima il cambio di indirizzo. Rimandiamo a dopo un esempio dell'altro caso.

Siccome la nuova identità *𝛿<sub>1</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku. Come si spiega nel documento di analisi,
in questo caso abbiamo l'indirizzo IP globale, l'indirizzo IP anonimizzante, l'indirizzo IP interno al g-nodo di
livello 3, di livello 2 e di livello 1.  
Deve però tenere presente che alcuni di essi possono essere già stati assegnati dal
precedente detentore del network namespace default o dallo stesso nella fase in cui aveva indirizzo virtuale.
In questo caso l'indirizzo IP interno al g-nodo di livello 1.

Deve poi aggiungere in queste tabelle (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝛿<sub>1</sub>*, inizialmente in stato "unreachable".  
Deve però tenere presente che alcune di esse possono essere già state aggiunte dal
precedente detentore del network namespace default o dallo stesso nella fase in cui aveva indirizzo virtuale.
In questo caso le destinazioni verso g-nodi di livello 1 e superiori.  
Deve inoltre considerare che prima la sua attuale posizione *reale* al livello 1 (2·1·0·) poteva essere
stata aggiunta come destinazione nelle tabelle (se le posizioni superiori erano tutte *reali*). In questo
caso essa va rimossa.

```
Mio indirizzo 2·1·0·1.
     globale
      10.0.0.21
     anonimizzante
      10.0.0.85
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
 3·
     globale
      10.0.0.24/29
     anonimizzante
      10.0.0.88/29
 2·0·
     globale
      10.0.0.16/30
     anonimizzante
      10.0.0.80/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 2·1·0·  era valida prima ma non lo è più.
     globale
      10.0.0.20/31
     anonimizzante
      10.0.0.84/31
     interno al mio g-nodo di livello 3
      10.0.0.60/31
     interno al mio g-nodo di livello 2
      10.0.0.48/31
 2·1·1·
     globale
      10.0.0.22/31
     anonimizzante
      10.0.0.86/31
     interno al mio g-nodo di livello 3
      10.0.0.62/31
     interno al mio g-nodo di livello 2
      10.0.0.50/31
 2·1·0·0
     globale
      10.0.0.20/32
     anonimizzante
      10.0.0.84/32
     interno al mio g-nodo di livello 3
      10.0.0.60/32
     interno al mio g-nodo di livello 2
      10.0.0.48/32
     interno al mio g-nodo di livello 1
      10.0.0.40/32
```

**sistema 𝛿**
```
ip address add 10.0.0.21 dev eth1
ip address add 10.0.0.85 dev eth1
ip address add 10.0.0.61 dev eth1
ip address add 10.0.0.49 dev eth1
```

**sistema 𝛿**
```
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.20/32 table ntk
ip route add unreachable 10.0.0.84/32 table ntk
ip route add unreachable 10.0.0.60/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk

ip route del 10.0.0.20/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
```

Ora il QspnManager di *𝛿<sub>1</sub>* riceve un ETP dal vicino *𝛾<sub>0</sub>*. A questo punto
conosce l'indirizzo Netsukuku del peer sull'arco *𝛿<sub>1</sub>-𝛾<sub>0</sub>*. Per questo il
programma *qspnclient* può creare la tabella `ntk_from_00:16:3E:5B:78:D5` e popolarla con tutte le
destinazioni adatte al suo indirizzo. Poi, ma solo dopo che la tabella sarà stata popolata
e aggiornata, aggiungerà la regola di guardare questa tabella per i pacchetti IP da inoltrare
ricevuti su questo arco.  
**Nota**: una volta che il modulo Qspn conosce l'indirizzo Netsukuku del peer tramite un arco, il
programma *qspnclient* nella tabella per i pacchetti da inoltrare che arrivano tramite quell'arco
può impostare come "blackhole" le rotte per gli indirizzi IP di tipo interno al g-nodo di livello
*i* dove *i* è minore o uguale al massimo distinto g-nodo di tale indirizzo. Questo come misura di
sicurezza, sebbene non dovrebbe ricevere pacchetti da quell'arco per un tale indirizzo IP.

**sistema 𝛿**
```
(echo; echo "248 ntk_from_00:16:3E:5B:78:D5 # xxx_table_ntk_from_00:16:3E:5B:78:D5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
```

Inoltre il QspnManager di *𝛿<sub>1</sub>* sulla base dell'ETP aggiorna la sua mappa. Assumiamo
che attraverso l'arco *𝛿<sub>1</sub>-𝛾<sub>0</sub>* scopra di poter raggiungere la destinazione
(1, 1), ossia il g-nodo 2·1·1·.

Poi il QspnManager di *𝛿<sub>1</sub>* notifica il segnale `bootstrap_complete`. Per questo
il programma *qspnclient* aggiorna tutte le rotte sulla base dei migliori percorsi noti.

**sistema 𝛿**
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
ip route change 10.0.0.22/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.86/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.62/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.50/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
```

Alla fine di questo aggiornamento delle tabelle, siccome era rimasta in attesa la regola di guardare
la nuova tabella, il programma *qspnclient* la aggiunge.

**sistema 𝛿**
```
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 248
ip rule add fwmark 248 table ntk_from_00:16:3E:5B:78:D5
```

Infine, dopo un po' di tempo, il g-nodo *𝜒* in *G<sub>𝛿</sub>* viene dismesso. Di fatto questo si
identifica con la dismissione di ogni identità di connettività che lo costituisce.

*   Se il g-nodo di connettività si era formato per la scelta da parte del g-nodo originario di
    fare ingresso in una diversa rete, allora immediatamente il g-nodo di connettività va
    dismesso. Ogni singolo nodo che lo componeva ne è al corrente.
*   Se il g-nodo di connettività si era formato per la scelta da parte del g-nodo originario di
    migrare dal suo g-nodo superiore in uno diverso della stessa rete, allora viene eletto un
    singolo nodo che gli appartiene a controllare periodicamente se il g-nodo di connettività può
    essere dismesso. Quando tale verifica è positiva il singolo nodo inizia la propagazione di
    questa informazione all'interno del g-nodo e dopo alcuni secondi dismette la sua identità di connettività.  
    La propagazione dell'informazione di dismissione avviene nel modulo QSPN, i singoli nodi che la ricevono
    ne notificano l'evento con un segnale al programma *qspnclient* del loro sistema.

Comunque questo avvenga, quando il programma *qspnclient* di un sistema sa di dover rimuovere una
identità di connettività esegue queste operazioni tramite il modulo Identities:

**sistema 𝛿**
```
ip netns exec entr03 ip route flush table main
ip netns exec entr03 ip link delete entr03_eth1 type macvlan
ip netns del entr03
```

Inoltre, a fronte della rimozione di un network namespace, il programma deve tenere traccia se qualche
particolare tabella `ntk_from_XXX` presente ha cessato di essere referenziata. In quel caso va
rimossa. Nel nostro esempio:

**sistema 𝛿**
```
sed -i '/xxx_table_ntk_from_00:16:3E:DF:23:F5_xxx/d' /etc/iproute2/rt_tables
```

## <a name="Elaborazione_etp"></a>Ricezione di un ETP che apporta variazioni alla mappa

A processazione ultimata di un ETP che apporta qualche variazione alla mappa di una identità nel
sistema, ad esempio *𝛿<sub>1</sub>*, è sufficiente per il sistema eseguire su tutte le destinazioni
in tutte le tabelle un aggiornamento delle rotte.

Assumiamo ad esempio che un ETP fornisce a *𝛿<sub>1</sub>* un nuovo percorso verso la destinazione
(2, 0), ossia il g-nodo 2·0·, passando per l'arco *𝛿<sub>1</sub>-𝛾<sub>0</sub>*.

**sistema 𝛿**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.80/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.56/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.22/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.86/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.62/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.50/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
```

