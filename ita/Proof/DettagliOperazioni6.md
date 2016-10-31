# Proof of concept - Dettagli Operazioni - Pagina 6

[Operazione precedente](DettagliOperazioni5.md)

### <a name="Ingresso_rete_2"></a> Ingresso in una rete - Caso 2

Sia *𝜑* un g-nodo di livello 1 costituito da *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>*. Queste sono
le identità principali dei sistemi *𝛿* e *𝜇*. In particolare questo ci interessa del sistema *𝛿*.  
Sia *𝜒* un g-nodo di livello 2 di una distinta rete. In particolare sia *𝛾* un sistema in cui
l'identità principale *𝛾<sub>0</sub>* è un singolo nodo in *𝜒*.  
Assumiamo che venga a formarsi un collegamento diretto tra i sistemi *𝛿* e *𝛾*.  
L'arco-identità principale sopra questo arco fisico è *𝛿<sub>0</sub>*-*𝛾<sub>0</sub>*.  
Assumiamo che *𝜑* voglia fare ingresso nell'altra rete dentro *𝜒* per mezzo di questo arco-identità.  
Indichiamo con *𝜑'* il nuovo g-nodo isomorfo di *𝜑* composto da *𝛿<sub>1</sub>* e *𝜇<sub>2</sub>*.
Indichiamo con *m<sub>𝜑</sub>* l'identificativo assegnato all'operazione di ingresso.
Assumiamo che il g-nodo *𝜒* assegna all'ingresso di *𝜑'* la posizione *virtuale*
2 e immediatamente disponibile la posizione *reale* 0.

Il modulo Neighborhood ha già prodotto questi comandi:

**sistema 𝛿**
```
ip route add 169.254.94.223 dev eth1 src 169.254.253.216
```

**sistema 𝛾**
```
ip route add 169.254.253.216 dev eth1 src 169.254.94.223
```

L'utente ha dato i comandi per accettare tale arco su entrambi i sistemi. Il modulo Identities ha
quindi realizzato per le relative identità principali un nuovo arco-identità *𝛿<sub>0</sub>*-*𝛾<sub>0</sub>*. L'utente ha
annotato l'identificativo del nuovo arco-identità nel sistema *𝛿* e il peer-MAC nel
sistema *𝛾*.

La sequenza di istruzioni che l'utente darà ai sistemi *𝛿* e *𝜇* sarà questa:

*   Al sistema *𝛿* dà il comando `prepare_enter_net_phase_1`, indicando queste informazioni:
    *   identità entrante: *𝛿<sub>0</sub>*.
    *   livello g-nodo entrante: 1.
    *   g-nodo ospitante *𝜒*:
        *   Livello: 2.
        *   Indirizzo Netsukuku: 2·1·.
        *   Fingerprint a livello 2. **TODO verificare**
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 1.
    *   nuova posizione reale:
        *   Identificativo: 0.
        *   Anzianità: 2.
    *   posizione di connettività.
        *   Identificativo: 2.
        *   Anzianità: 1.
    *   nuovi archi-qspn: 1.
        *   `identityarc_index`: 4. Assumiamo che questo sia il valore assegnato come identificativo
            all'arco-identità *𝛿<sub>0</sub>*-*𝛾<sub>0</sub>* nel sistema *𝛿* prima della duplicazione
            dell'identità *𝛿<sub>0</sub>*.
    *   identificativo di questa operazione di ingresso: *m<sub>𝜑</sub>*.
    *   identificativo della previa operazione di migrazione: nullo.
*   Al sistema *𝜇* dà il comando `prepare_enter_net_phase_1`, indicando queste informazioni:
    *   identità entrante: *𝜇<sub>1</sub>*.
    *   livello g-nodo entrante: 1.
    *   g-nodo ospitante *𝜒*:
        *   Livello: 2.
        *   Indirizzo Netsukuku: 2·1·.
        *   Fingerprint a livello 2. **TODO verificare**
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 1.
    *   nuova posizione reale:
        *   Identificativo: 0.
        *   Anzianità: 2.
    *   posizione di connettività.
        *   Identificativo: 2.
        *   Anzianità: 1.
    *   nuovi archi-qspn: nessuno.
    *   identificativo di questa operazione di ingresso: *m<sub>𝜑</sub>*.
    *   identificativo della previa operazione di migrazione: nullo.
*   Al sistema *𝛿* dà il comando `enter_net_phase_1`, indicando queste informazioni:
    *   identificativo di operazione di ingresso: *m<sub>𝜑</sub>*.
*   Al sistema *𝜇* dà il comando `enter_net_phase_1`, indicando queste informazioni:
    *   identificativo di operazione di ingresso: *m<sub>𝜑</sub>*.
*   Al sistema *𝛾* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝛾<sub>0</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità *𝛿<sub>0</sub>*-*𝛾<sub>0</sub>* nel sistema *𝛾*.  
        È necessario che il comando `add_qspn_arc` nel sistema *𝛾* venga dato dopo il completamento
        del comando `enter_net_phase_1` nel sistema *𝛿* con sufficiente ritardo affinché il peer-MAC che
        l'utente specifica sia ormai associato all'arco-identità *𝛿<sub>1</sub>*-*𝛾<sub>0</sub>*, cioè
        l'arco con la nuova identità di *𝛿*. Allo stesso tempo è necessario che sia dato in tempi rapidi
        dopo il comando `enter_net_phase_1` nel sistema *𝛿* perché non deve scadere il tempo di rilevamento
        dell'arco per il modulo Qspn.
*   Poiché *"identificativo della previa operazione di migrazione"* è nullo, non serve dare il
    comando `enter_net_phase_2` ai sistemi *𝛿* e *𝜇*.

#### Comando prepare_enter_net_phase_1

Sul comando `prepare_enter_net_phase_1` il programma **qspnclient** memorizza le informazioni ricevute.

Prima di avviare le operazioni di questo ingresso, il programma **qspnclient** memorizza le proprietà
correnti dell'identità interessata. In particolare l'indirizzo Netsukuku, che nel nostro caso assumiamo
essere 3·1·0·X, dove X vale 0 per *𝛿<sub>0</sub>* e 1 per *𝜇<sub>1</sub>*.

Poi il programma **qspnclient** verifica per ogni arco-identità associato all'identità interessata
all'ingresso se questo è associato ad un arco-qspn. Cioè se tale arco-identità faceva parte della
vecchia rete. In questo caso memorizza questo arco e le sue proprietà correnti: MAC address e IP link-local.

Poi il programma **qspnclient** chiama il metodo `prepare_add_identity`
del modulo Identities.

#### Comando enter_net_phase_1

Sul comando `enter_net_phase_1` il programma **qspnclient** recupera le informazioni memorizzate prima. Poi chiama il
metodo `add_identity` del modulo Identities. Il modulo Identities produce quindi queste operazioni:

**sistema 𝛿**
```
ip netns add entr03
ip netns exec entr03 sysctl net.ipv4.ip_forward=1
ip netns exec entr03 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 ip link set dev entr03_eth1 address 00:16:3E:B9:77:80
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.rp_filter=0
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_ignore=1
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_announce=2
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.83.167 dev entr03_eth1
ip netns exec entr03 ip route add 169.254.242.91 dev entr03_eth1 src 169.254.83.167
ip netns exec entr03 ip route add 169.254.94.223 dev entr03_eth1 src 169.254.83.167
```

**sistema 𝜇**
```
ip netns add entr03
ip netns exec entr03 sysctl net.ipv4.ip_forward=1
ip netns exec entr03 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 ip link set dev entr03_eth1 address 00:16:3E:DF:23:F5
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.rp_filter=0
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_ignore=1
ip netns exec entr03 sysctl net.ipv4.conf.entr03_eth1.arp_announce=2
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.242.91 dev entr03_eth1
ip netns exec entr03 ip route add 169.254.83.167 dev entr03_eth1 src 169.254.242.91
```

Dopo l'esecuzione del metodo `add_identity` il programma verifica tra tutti gli archi-identità che erano
in precedenza associati ad un arco-qspn quali hanno cambiato le loro proprietà MAC address e IP link-local.

Nel sistema *𝛾* il modulo Identities in autonomia (a fronte del comando `enter_net_phase_1` dato nel sistema
*𝛿*) produce queste operazioni:

**sistema 𝛾**
```
ip route add 169.254.83.167 dev eth1 src 169.254.94.223
```

#### <a name="Spostamento_rotte_identita"></a> Spostamento delle rotte della vecchia identità

Abbiamo precedentemente detto quali sono le informazioni che il programma **qspnclient** ha relativamente
all'operazione di ingresso e quali arriva a conoscere con l'esecuzione del metodo `add_identity`.

Il programma **qspnclient** decide quali tabelle di inoltro vanno usate nel nuovo network namespace.
In questo caso abbiamo l'arco-qspn tra *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>* che cambia le sue
proprietà.

Il programma **qspnclient** calcola l'indirizzo della vecchia identità nel nuovo namespace. Questo
si calcola a partire dall'indirizzo Netsukuku precedente della vecchia identità, sostituendo al
livello *"livello g-nodo entrante"* la *"posizione di connettività"*. Quindi in questo caso
abbiamo 3·1·2·X, dove X vale 0 per *𝛿<sub>0</sub>* e 1 per *𝜇<sub>1</sub>*.

Il programma **qspnclient** calcola tutti i possibili indirizzi IP di destinazione, ognuno con suffisso CIDR,
(come riportato [qui](DettagliOperazioni2.md#computo_indirizzi_ip_destinazioni))
relativi all'indirizzo della vecchia identità nel nuovo namespace. Il programma li memorizza associandoli a
quella identità.

Inizialmente il programma aggiunge le rotte verso questi indirizzi IP nello stato `unreachable` in tutte le
tabelle di inoltro che vanno usate nel nuovo network namespace.

**sistema 𝛿**
```
(echo; echo "249 ntk_from_00:16:3E:DF:23:F5 # xxx_table_ntk_from_00:16:3E:DF:23:F5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:DF:23:F5 -j MARK --set-mark 249
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
ip netns exec entr03 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5
```

Va notato che, vista la posizione *virtuale* dell'indirizzo Netsukuku al livello 1, abbiamo una
ulteriore destinazione possibile come g-nodo di livello 1; mentre per le possibili destinazioni
come g-nodo di livello 0 abbiamo che soltanto gli indirizzi IP interni al livello 1 (e inferiori)
sono validi.

Poi il programma **qspnclient** rimuove dalle tabelle presenti nel vecchio network namespace le rotte
verso i possibili indirizzi IP di destinazione relativi all'indirizzo che la vecchia identità aveva nel
vecchio namespace e che ora non sono più validi.  
Consideriamo le destinazioni di livello minore del livello del g-nodo che fa ingresso in blocco. Relativamente
a queste consideriamo gli indirizzi IP interni ai g-nodi di livello minore o uguale al livello del g-nodo che
fa ingresso in blocco. Tali rotte non vanno rimosse. In questo
caso gli indirizzi IP interni al g-nodo di livello 1 che portano a destinazioni che sono g-nodi
di livello 0. Cioè 10.0.0.40/32 per *𝛿* e 10.0.0.41/32 per *𝜇*.

**sistema 𝛿**
```
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
ip route del 10.0.0.28/32 table ntk
ip route del 10.0.0.92/32 table ntk
ip route del 10.0.0.60/32 table ntk
ip route del 10.0.0.48/32 table ntk
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
ip route del 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
```

Poi il programma **qspnclient**, solo se il vecchio namespace è il default (come nel nostro caso),
rimuove dal vecchio namespace gli indirizzi IP della vecchia identità che non saranno comuni
con quelli della nuova identità. Cioè quelli interni ai g-nodi di livello maggiore del
livello del nuovo g-nodo che si è costituito nella nuova rete.

**sistema 𝛿**
```
ip address del 10.0.0.49/32 dev eth1
ip address del 10.0.0.61/32 dev eth1
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.29
ip address del 10.0.0.29/32 dev eth1
ip address del 10.0.0.93/32 dev eth1
```

* * *

**Osservazione.**
In generale può succedere che un g-nodo di livello *i* fa ingresso in altra rete
andando a costituire un nuovo g-nodo di livello *j* in un g-nodo esistente di livello *j* + 1, con *j* ≥ *i*.  
Ad esempio un singolo nodo potrebbe costituire un nuovo singolo nodo all'interno di un g-nodo esistente
di livello 1, oppure potrebbe trovarsi a costituire un nuovo g-nodo di livello 1 all'interno di un g-nodo esistente
di livello 2.

Quando un g-nodo *𝜑* di livello *i* migra o fa ingresso in una rete, sicuramente il suo indirizzo Netsukuku
è tale che tutte le posizioni maggiori o uguali a *i* sono *reali*. Perciò quando *𝜑'* prende
posto in un g-nodo *𝜒* di livello *k* maggiore di *i*, comporremo il nuovo indirizzo Netsukuku di ogni
singolo nodo in *𝜑'* in questo modo:

*   Gli identificativi di *𝜒* per i livelli da *l* - 1 a *k*.
*   L'identificativo assegnato a *𝜑'* al livello *k* - 1.
*   Gli identificativi che aveva la precedente identità in *𝜑* per i livelli da *k* - 2 a 0.

Questo ci assicura che i livelli da *l* - 1 a *i* sono tutti *reali*.

* * *

Prendiamo di nuovo in considerazione tutti i possibili indirizzi IP di destinazione relativi all'indirizzo
della vecchia identità nel nuovo namespace. In tutte le tabelle di inoltro che vanno usate nel nuovo
network namespace, ma soltanto nelle tabelle di inoltro che hanno come nodo vicino un altro
nodo che partecipa alla migrazione/ingresso, il programma aggiorna lo stato delle rotte sulla
base delle conoscenze che l'identità aveva da prima e aggiunge la regola.

**sistema 𝛿**
```
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
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:DF:23:F5
```

Analogamente avremo queste operazioni nell'altro sistema la cui identità fa ingresso.

**sistema 𝜇**
```
(echo; echo "249 ntk_from_00:16:3E:B9:77:80 # xxx_table_ntk_from_00:16:3E:B9:77:80_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:B9:77:80 -j MARK --set-mark 249
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
ip netns exec entr03 ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:B9:77:80

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
ip route del 10.0.0.29/32 table ntk
ip route del 10.0.0.93/32 table ntk
ip route del 10.0.0.61/32 table ntk
ip route del 10.0.0.49/32 table ntk
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
ip route del 10.0.0.29/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.93/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45

ip address del 10.0.0.48/32 dev eth1
ip address del 10.0.0.60/32 dev eth1
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address del 10.0.0.28/32 dev eth1
ip address del 10.0.0.92/32 dev eth1

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
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:B9:77:80
```

#### Popolamento nuove rotte della nuova identità

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** provvede come vediamo adesso a preparare il vecchio namespace
per la nuova identità.

Il programma **qspnclient** calcola tutti i possibili indirizzi IP di destinazione relativi all'indirizzo
della nuova identità. Li memorizza associandoli a questa nuova identità.

Nel vecchio network namespace abbiamo alcune tabelle pre-esistenti (che erano usate per la vecchia
identità) e che vanno mantenute in quanto saranno usate dalla nuova identità. In questo caso si tratta della
tabella `ntk` nel namespace default e della tabella di inoltro `ntk_from_00:16:3E:2D:8D:DE` che resta
valida per la nuova identità in quanto identifica un arco interno al g-nodo che ha fatto ingresso (il programma
lo capisce dal fatto che l'arco-identità originante ha cambiato il peer-MAC nel nuovo network namespace).

Tra gli indirizzi IP di possibili destinazioni associate alla nuova identità, alcuni erano presenti nelle tabelle che erano
pre-esistenti nel vecchio network namespace e non sono stati rimossi nella precedente fase.  
Cioè, relativamente alle destinazioni di livello minore del livello del g-nodo che fa ingresso in blocco,
gli indirizzi IP interni ai g-nodi di livello minore o uguale al livello del g-nodo che
fa ingresso in blocco. Cioè 10.0.0.40/32 per *𝛿* e 10.0.0.41/32 per *𝜇*.  
Gli altri vanno ora aggiunti alle tabelle pre-esistenti nel vecchio namespace.

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

Analogamente avremo queste operazioni nell'altro sistema la cui identità fa ingresso.

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

#### Creazione e popolamento iniziale di tabelle per l'inoltro

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** crea una istanza di QspnManager per la sua nuova identità e gli
comunica (nel costruttore) di quali archi-qspn dispone.

Nel nostro esempio ci sarà un nuovo
arco-qspn, comunicato dall'utente nel comando `prepare_enter_net_phase_1`. Di tale arco-identità il
programma conosce:

*   MAC address del vicino.
*   Indirizzo IP linklocal del vicino.

Tenendo traccia inoltre delle tabelle già inserite nel file `/etc/iproute2/rt_tables`, il programma è in grado di
eseguire le operazioni che seguono. Lo deve fare per tutti gli archi-qspn passati al QspnManager. Come sempre,
vanno eseguite in blocco senza intromissioni da altre tasklet.

**sistema 𝛿**
```
(echo; echo "248 ntk_from_00:16:3E:5B:78:D5 # xxx_table_ntk_from_00:16:3E:5B:78:D5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 248
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
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
```

#### Comando add_qspn_arc

Quando l'utente dà il comando `add_qspn_arc` nel sistema *𝛾*, il programma **qspnclient**
aggiunge sulla relativa istanza di QspnManager un nuovo arco-qspn. Di tale arco-identità il
programma conosce:

*   MAC address del vicino.
*   Indirizzo IP linklocal del vicino.

Il programma è in grado di eseguire le operazioni che seguono. Come sempre,
vanno eseguite in blocco senza intromissioni da altre tasklet.

**sistema 𝛾**
```
(echo; echo "249 ntk_from_00:16:3E:1A:C4:45 # xxx_table_ntk_from_00:16:3E:1A:C4:45_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:1A:C4:45 -j MARK --set-mark 249
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
```

#### Cambio di indirizzo di una identità

Come abbiamo detto in precedenza nel ["caso 1"](DettagliOperazioni4.md),
se l'utente nel comando `prepare_enter_net_phase_1` non ha
specificato una migration path, cioè ha richiesto di procedere immediatamente con l'assegnazione
dell'indirizzo *reale* dentro il g-nodo destinazione, allora tale cambio di indirizzo avviene
immediatamente, prima cioè che possa essere ricevuto un ETP.

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** sulla base delle istruzioni che l'utente aveva dato nel
comando `prepare_enter_net_phase_1`, dopo aver fatto passare la nuova identità per l'indirizzo
con posizione *virtuale* al livello 1, cambia il suo identificativo di livello 1 con la posizione
*reale* dentro il g-nodo destinazione. Cioè comunica questa variazione alla relativa istanza del modulo Qspn.

Oltre a ciò, il programma **qspnclient** esegue di seguito le due sequenze di
operazioni che vedremo adesso: la prima perché un identificativo dell'indirizzo di una sua
identità passa da *virtuale* a *reale* e la seconda perchè questo cambio avviene nell'identità *principale*.

##### Un identificativo passa da virtuale a reale

In questa occasione il programma **qspnclient** rimuove, nel network namespace interessato, da tutte le tabelle
presenti (anche quelle di inoltro il cui arco non ha ancora ricevuto alcun ETP) le
rotte verso gli indirizzi IP che non sono più validi come possibili destinazioni a causa di questo nuovo identificativo *reale*.

Inoltre il programma **qspnclient** aggiunge, nel network namespace interessato, su tutte le tabelle
presenti (anche quelle di inoltro il cui arco non ha ancora ricevuto alcun ETP) le
rotte verso gli indirizzi IP che sono diventati validi come possibili destinazioni a causa di questo nuovo
identificativo *reale*.  
Inoltre, per le rotte aggiunte il programma **qspnclient** subito le aggiorna in base alle conoscenze di routing
del relativo modulo Qspn.

Notiamo che questo avviene anche nelle identità *di connettività* (che gestiscono un namespace
diverso dal default) e anche se l'indirizzo Netsukuku non è del tutto *reale*.

**sistema 𝛿**
```
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/32 table ntk
ip route add unreachable 10.0.0.84/32 table ntk
ip route add unreachable 10.0.0.60/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
```

Analogamente avremo queste operazioni nell'altro sistema la cui identità fa ingresso.

**sistema 𝜇**
```
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.21/32 table ntk
ip route add unreachable 10.0.0.85/32 table ntk
ip route add unreachable 10.0.0.61/32 table ntk
ip route add unreachable 10.0.0.49/32 table ntk
ip route add unreachable 10.0.0.21/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.85/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.21/32 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.85/32 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.61/32 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.49/32 table ntk via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.21/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.85/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
```

##### L'identità interessata dal cambio di indirizzo è la principale

In questa occasione il programma **qspnclient** aggiunge ad ogni interfaccia di rete reale nel network
namespace default gli indirizzi propri che prima non era possibile computare e adesso invece sì,
partendo dal livello del nuovo g-nodo che si è costituito nella nuova rete e salendo finché possibile.

Poi, solo se l'indirizzo è ora del tutto *reale*, aggiunge (opzionalmente) la regola di source-natting
e (opzionalmente) l'indirizzo IP anonimizzante.

Infine aggiorna tutte le rotte nella tabella `ntk` per fare in modo di mettere in esse (se disponibile)
un src preferito.

**sistema 𝛿**
```
ip address add 10.0.0.49 dev eth1
ip address add 10.0.0.61 dev eth1
ip address add 10.0.0.21 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.21
ip address add 10.0.0.85 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.22/31 table ntk
ip route change unreachable 10.0.0.86/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41
```

Analogamente avremo queste operazioni nell'altro sistema la cui identità fa ingresso.

**sistema 𝜇**
```
ip address add 10.0.0.48 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.20 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.20
ip address add 10.0.0.84 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.22/31 table ntk
ip route change unreachable 10.0.0.86/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change 10.0.0.21/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.85/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.61/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.60
ip route change 10.0.0.49/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.48
ip route change 10.0.0.41/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.40
```

#### Processazione di un ETP

Le operazioni da fare al termine della processazione di un ETP sono già state esaminate e
non ci sono ulteriori dettagli nel caso in esame (di ingresso di un g-nodo).

#### Dismissione identità

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** dismette la vecchia identità.

**sistema 𝛿**
```
ip netns exec entr03 ip route flush table main
ip netns exec entr03 ip link delete entr03_eth1 type macvlan
ip netns del entr03
sed -i '/xxx_table_ntk_from_00:16:3E:DF:23:F5_xxx/d' /etc/iproute2/rt_tables
```

Analogamente avremo queste operazioni nell'altro sistema la cui identità fa ingresso.

**sistema 𝜇**
```
ip netns exec entr03 ip route flush table main
ip netns exec entr03 ip link delete entr03_eth1 type macvlan
ip netns del entr03
sed -i '/xxx_table_ntk_from_00:16:3E:B9:77:80_xxx/d' /etc/iproute2/rt_tables
```

[Operazione seguente](DettagliOperazioni7.md)
