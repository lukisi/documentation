# Proof of concept - Casi d'uso - Parte 6

## <a name="step3"></a>Ingresso di 𝛿 e 𝜇 in G𝛾

Abbiamo già visto cosa avviene nel sistema *𝛿* quando il g-nodo *𝜒'* entra in *G<sub>𝛾</sub>*. Vediamo cosa
avviene negli altri sistemi.

### Nel sistema *𝜇*

Ricordiamo che l'identità *𝜇<sub>1</sub>* (la principale ed unica al momento nel sistema *𝜇*)
ha indirizzo 3·1·0·0 in *G<sub>𝛿</sub>*. Appartiene a *𝜒*.

Il g-nodo *𝜒* di livello 1 e di indirizzo Netsukuku 3·1·0·, che comprende *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>*,
decide di entrare in *G<sub>𝛾</sub>*. Per essere precisi, decide di costruire un g-nodo isomorfo
*𝜒'* costituito dalle nuove identità *𝛿<sub>1</sub>* e *𝜇<sub>2</sub>*. Il g-nodo
*𝜒* assume indirizzo *di connettività* 3·1·2· in *G<sub>𝛿</sub>*. Temporaneamente
*𝜒'* assume indirizzo *virtuale* 2·1·2· in *G<sub>𝛾</sub>*.

Queste informazioni giungono a *𝜇<sub>1</sub>* da parte di *𝛿<sub>0</sub>*. Non si sono formati
nuovi archi che interessano il sistema *𝜇*.

Il sistema *𝜇* costruisce una nuova identità *𝜇<sub>2</sub>* partendo da *𝜇<sub>1</sub>*.
Viene creato nel sistema *𝜇* un nuovo network namespace "entr03" e in esso viene creata
una pseudo-interfaccia "entr03_eth1" sopra l'interfaccia reale "eth1". Questo nuovo network namespace
sarà gestito da *𝜇<sub>1</sub>* mentre quello precedente (il default) verrà gestito da *𝜇<sub>2</sub>*.  
Ricordiamo che una identità di connettività quale è ora *𝜇<sub>1</sub>*, nel suo network namespace che non è
il default, non detiene alcun indirizzo IP associato al suo indirizzo Netsukuku.

**sistema 𝜇**
```
ip netns add entr03
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 ip link set dev entr03_eth1 address 00:16:3E:DF:23:F5
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.242.91 dev entr03_eth1
```

Anche nel sistema *𝛿* partendo da *𝛿<sub>0</sub>* è stata creata una nuova identità *𝛿<sub>1</sub>*.  
L'identità *𝛿<sub>0</sub>* verrà temporaneamente spostata sul network namespace "entr03" del sistema *𝛿*.
Il namespace default è gestito ora da *𝛿<sub>1</sub>*.  
Il modulo Identities del sistema *𝜇*, dal dialogo con i vicini, desume che vanno creati/modificati questi
archi-identità:

*   *𝜇<sub>1</sub>-𝛿<sub>0</sub>*.  
    Questo arco-identità vede cambiare il `peer_mac` e `peer_linklocal`. Inoltre la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝜇<sub>1</sub>*
    la rotta verso il vicino.  
    I nodi *𝜇<sub>1</sub>* e *𝛿<sub>0</sub>* fanno parte della stessa rete, quindi è prevista una tabella
    per i pacchetti IP da inoltrare che provengono da questo arco. Nel nuovo network namespace gestito da *𝜇<sub>1</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:B9:77:80` e la relativa regola.
*   *𝜇<sub>2</sub>-𝛿<sub>1</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝜇<sub>2</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝜇<sub>2</sub>* e *𝛿<sub>1</sub>* fanno parte della stessa rete, quindi è prevista una tabella
    per i pacchetti IP da inoltrare che provengono da questo arco. Nel network namespace gestito da *𝛿<sub>1</sub>*
    la tabella `ntk_from_00:16:3E:1A:C4:45` c'è già, e anche la relativa regola.

Il modulo Identities fa queste operazioni:

**sistema 𝜇**
```
ip netns exec entr03 ip route add 169.254.83.167 dev entr03_eth1 src 169.254.242.91
```

Il programma *qspnclient* fa queste operazioni preliminari:

**sistema 𝜇**
```
ip netns exec entr03 ip rule add table ntk
(echo; echo "249 ntk_from_00:16:3E:B9:77:80 # xxx_table_ntk_from_00:16:3E:B9:77:80_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:B9:77:80 -j MARK --set-mark 249
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:B9:77:80
```

Il programma *qspnclient* fa queste operazioni sulle rotte verso g-nodi di livello *k*, con `k < 1`, cioè
verso destinazioni interne al g-nodo che ha migrato:

*   Quelle espresse con indirizzi IP interni ad un g-nodo fino al livello 1, vanno mantenute nel network namespace
    vecchio e copiate nel network namespace nuovo.
*   Quelle espresse con indirizzi IP globali o interni ad un g-nodo di livello superiore, vanno rimosse dal
    network namespace vecchio.

**sistema 𝜇**
```
ip netns exec entr03 ip route add unreachable 10.0.0.41/32 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:B9:77:80

ip route del 10.0.0.29/32 table ntk
ip route del 10.0.0.93/32 table ntk
ip route del 10.0.0.61/32 table ntk
ip route del 10.0.0.49/32 table ntk
ip route del 10.0.0.29/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.93/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
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

**sistema 𝜇**
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
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.28/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.92/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:B9:77:80

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
ip route del 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.16/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.80/29 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.24/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.88/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.30/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.94/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
```

Il programma *qspnclient* rimuove l'indirizzo globale e gli indirizzi interni ai propri g-nodi di livello
maggiore di 1 dal network namespace vecchio.

**sistema 𝜇**
```
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address del 10.0.0.28/32 dev eth1
ip address del 10.0.0.92/32 dev eth1
ip address del 10.0.0.60/32 dev eth1
ip address del 10.0.0.48/32 dev eth1
```

Il programma *qspnclient* aggiorna le rotte nel nuovo network namespace sulla base dei migliori percorsi
noti alla vecchia identità *𝜇<sub>1</sub>*.

**sistema 𝜇**
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
ip netns exec entr03 ip route change 10.0.0.41/32 table ntk via 169.254.83.167 dev entr03_eth1

ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.28/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.92/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:B9:77:80
ip netns exec entr03 ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:B9:77:80
```

Poi il sistema *𝜇* per la nuova identità *𝜇<sub>2</sub>* istanzia un QspnManager con il
costruttore `enter_net` passandogli un QspnArc per l'arco-identità *𝜇<sub>2</sub>-𝛿<sub>1</sub>*.
L'indirizzo Netsukuku del nodo *𝜇<sub>2</sub>* sarà 2·1·2·0: cioè l'indirizzo di *𝜒'* con la parte finale che era già di
*𝜇<sub>1</sub>* in *𝜒*.

Siccome la nuova identità *𝜇<sub>2</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku. Come si spiega nel documento di analisi,
in questo caso abbiamo solo l'indirizzo IP interno al g-nodo di livello 1.  
Deve però tenere presente che alcuni di essi possono essere rimasti assegnati dal
precedente detentore del network namespace default. In questo caso tutti; quindi non ci
sono operazioni da fare.

Inoltre il programma *qspnclient* deve assicurarsi che nel network namespace della nuova identità *𝜇<sub>2</sub>*
siano presenti le regole per la consultazione della tabella `ntk` e delle tabelle `ntk_from_xxx` associate agli
archi-identità di *𝛿<sub>1</sub>* per i quali conosciamo già l'indirizzo Netsukuku del peer. E non altre regole.

Deve poi aggiungere in queste tabelle (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝜇<sub>2</sub>*, inizialmente in stato "unreachable". Deve anche fare in modo che le operazioni di aggiunta
delle destinazioni vengano completate tutte prima di elaborare operazioni di aggiornamento delle rotte: infatti
queste potrebbero essere avviate, ad esempio, dopo il segnale `bootstrap_complete` che viene notificato da un'altra tasklet.

**sistema 𝜇**
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

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
```

Poi l'indirizzo Netsukuku di *𝜒'* in *G<sub>𝛾</sub>*, che
temporaneamente era il *virtuale* 2·1·2·, diventa il *reale* 2·1·0·. Di conseguenza il programma comunica al
QspnManager di *𝜇<sub>2</sub>* che ora il suo indirizzo Netsukuku è 2·1·0·0.

Il programma *qspnclient* assegna all'interfaccia reale nel network
namespace default gli indirizzi IP che mancano ancora. Poi aggiunge nelle tabelle le possibili destinazioni
previste dall'indirizzo di *𝜇<sub>2</sub>* ancora mancanti. Rimuove, se le aveva aggiunte, quelle per
l'indirizzo reale che ha ora assunto.

**sistema 𝜇**
```
ip address add 10.0.0.20 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.20
ip address add 10.0.0.84 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
```

**sistema 𝜇**
```
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.21/32 table ntk
ip route add unreachable 10.0.0.85/32 table ntk
ip route add unreachable 10.0.0.61/32 table ntk
ip route add unreachable 10.0.0.49/32 table ntk

ip route del 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.21/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.85/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
```

Ora il QspnManager di *𝜇<sub>2</sub>* riceve un ETP dal vicino *𝛿<sub>1</sub>*. Sulla base dell'ETP aggiorna la sua
mappa. Assumiamo che attraverso l'arco *𝜇<sub>2</sub>-𝛿<sub>1</sub>* scopra di poter raggiungere la destinazione
(1, 1), ossia il g-nodo 2·1·1·.

Poi il QspnManager di *𝜇<sub>2</sub>* notifica il segnale `bootstrap_complete`. Per questo
il programma *qspnclient* aggiorna tutte le rotte sulla base dei migliori percorsi noti.

**sistema 𝜇**
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
ip route change 10.0.0.22/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.86/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.62/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.60
ip route change 10.0.0.50/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.48
ip route change 10.0.0.21/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.85/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.61/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.60
ip route change 10.0.0.49/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.48
ip route change 10.0.0.41/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.40

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.21/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.85/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
```

Infine, dopo un po' di tempo, il g-nodo *𝜒* in *G<sub>𝛿</sub>* viene dismesso.
Il programma *qspnclient* nel sistema *𝜇* sa di dover rimuovere *𝜇<sub>1</sub>*.
Esegue queste operazioni tramite il modulo Identities:

**sistema 𝜇**
```
ip netns exec entr03 ip route flush table main
ip netns exec entr03 ip link delete entr03_eth1 type macvlan
ip netns del entr03
```

Inoltre, a fronte della rimozione di un network namespace, il programma deve tenere traccia se qualche
particolare tabella `ntk_from_XXX` presente ha cessato di essere referenziata. In quel caso va
rimossa. Nel nostro esempio:

**sistema 𝜇**
```
sed -i '/xxx_table_ntk_from_00:16:3E:B9:77:80_xxx/d' /etc/iproute2/rt_tables
```

### Nel sistema *𝛾*

Abbiamo detto che il sistema *𝛾* giunge a distanza di rilevamento con la sua interfaccia di rete
per l'interfaccia di rete del sistema *𝛿*. Il modulo Neighborhood aggiunge nel network namespace default
la rotta diretta verso l'indirizzo IP linklocal del vicino.

**sistema 𝛾**
```
ip route add 169.254.253.216 dev eth1 src 169.254.94.223
```

Abbiamo detto che in precedenza *𝛽* entra in *G<sub>𝛾</sub>*, quindi le identità
che esistono ora sono *𝛽<sub>1</sub>* e *𝛾<sub>0</sub>*.

Quando il sistema *𝛿* entra (insieme al suo g-nodo) in *G<sub>𝛾</sub>*, crea un nuovo network namespace
in cui temporaneamente sposta (primi di dismetterla) l'identità *𝛿<sub>0</sub>*. Per questo il modulo
Identities nel sistema *𝛾* crea una nuova rotta che presto verrà rimossa.

**sistema 𝛾**
```
ip route add 169.254.83.167 dev eth1 src 169.254.94.223
```

Il modulo Qspn del sistema *𝛾* riceve un nuovo arco, ma fino a quando non riceve tramite esso un ETP
non apporta modifiche alle regole. Intanto però crea una tabella per i pacchetti da inoltrare che
arrivano tramite quell'arco. Poi riceve un ETP che dice che *𝛿<sub>1</sub>* ha in *G<sub>𝛾</sub>*
l'indirizzo virtuale 2·1·2·1. Infine un ETP che dice che *𝛿<sub>1</sub>* ha in *G<sub>𝛾</sub>*
l'indirizzo reale 2·1·0·1.  
**Nota**: una volta che il modulo Qspn conosce l'indirizzo Netsukuku del peer tramite un arco, il
programma *qspnclient* nella tabella per i pacchetti da inoltrare che arrivano tramite quell'arco
può impostare come "blackhole" le rotte per gli indirizzi IP di tipo interno al g-nodo di livello
*i* dove *i* è minore o uguale al massimo distinto g-nodo di tale indirizzo. Questo come misura di
sicurezza, sebbene non dovrebbe ricevere pacchetti da quell'arco per un tale indirizzo IP.

**sistema 𝛾**
```
(echo; echo "249 ntk_from_00:16:3E:1A:C4:45 # xxx_table_ntk_from_00:16:3E:1A:C4:45_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
# Popola tabella con possibili destinazioni.
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
# Aggiorna sulla base del primo ETP che dice che 𝛿1 è virtuale al livello 1
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
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
# tabella from_xxx è popolata e aggiornata
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:1A:C4:45 -j MARK --set-mark 249
ip rule add fwmark 249 table ntk_from_00:16:3E:1A:C4:45
# Aggiorna sulla base del nuovo ETP che dice che 𝛿1 è reale al livello 1
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1 src 10.0.0.50
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev eth1 src 10.0.0.50
ip route change 10.0.0.23/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.87/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.22
ip route change 10.0.0.63/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.62
ip route change 10.0.0.51/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.40
# Dismissione 𝛿0
ip route del 169.254.83.167 dev eth1 src 169.254.94.223
```

### Nel sistema *𝛽*

Il sistema *𝛽* non rileva direttamente alcun cambiamento. La sua identità *𝛽<sub>1</sub>* vede arrivare un
ETP dal vicino *𝛾<sub>0</sub>* con il quale scopre un percorso verso la destinazione 2·1·0·.

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
ip route change 10.0.0.20/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.84/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.60/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.63
ip route change 10.0.0.48/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.51
ip route change 10.0.0.22/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.86/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.62/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.63
ip route change 10.0.0.50/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.41
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
```

