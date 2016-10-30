# Proof of concept - Dettagli Operazioni - Pagina 11

[Operazione precedente](DettagliOperazioni10.md)

### <a name="Migrazione_ingresso_2"></a> Migrazione per ingresso - Caso 2

Sia *𝜆* un sistema che è configurato per fare da gateway ad una sottorete autonoma della dimensione di un
g-nodo di livello 1. In tale sistema al momento sia *𝜆<sub>0</sub>* l'identità principale che non
ha nessun arco verso altri nodi Netsukuku.

#### Fasi iniziali in *𝜆*

Ripassiamo le fasi iniziali in un sistema. Sono corrette anche nel caso di un sistema che fa da
gateway.

**sistema 𝜆**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip address add 10.0.0.32 dev lo

sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2

ip link set dev eth1 up
ip address add 169.254.109.22 dev eth1

(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk

ip address add 10.0.0.12 dev eth1
ip address add 10.0.0.76 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.48 dev eth1
ip address add 10.0.0.40 dev eth1

ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.8/30 table ntk
ip route add unreachable 10.0.0.72/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.14/31 table ntk
ip route add unreachable 10.0.0.78/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk

iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.12
```

Alla fine vanno aggiunte queste operazioni:

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *n* l'indirizzo Netsukuku del sistema.
*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Se `$subnetlevel` > 0:
    *   Per *i* che sale da `$subnetlevel` a *l* - 1:
        *   Se *pos_n(i)* ≥ *gsize(i)*, cioè se la posizione è *virtuale* al livello *i* (questa condizione
            è sempre falsa nel caso delle operazioni iniziali del sistema):
            *   Esci dal ciclo *i*.
        *   Se *i* < *l* - 1:
            *   Per i pacchetti IP che passano per questo sistema e sono destinati ad
                un indirizzo IP interno al g-nodo di livello *i* + 1 che identifica un nodo
                interno alla mia sottorete autonoma (di livello `$subnetlevel`) e che quindi
                necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
                dell'IP di destinazione affinché risulti nel range degli indirizzi IP
                interni al g-nodo di livello `$subnetlevel`.
            *   Per i pacchetti IP che passano per questo sistema, che hanno per IP di mittente
                un indirizzo IP interno al g-nodo di livello `$subnetlevel` (che cioè
                provengono dall'interno della sottorete autonoma) e che sono destinati ad
                un indirizzo IP interno al g-nodo di livello *i* + 1, esegui la rimappatura
                dell'IP di mittente affinché risulti nel range degli indirizzi IP
                interni al g-nodo di livello *i* e identifichi un nodo
                interno alla mia sottorete autonoma.
        *   Altrimenti (cioè per *i* = *l* - 1):
            *   Per i pacchetti IP che passano per questo sistema e sono destinati ad
                un indirizzo IP globale che identifica un nodo
                interno alla mia sottorete autonoma (di livello `$subnetlevel`) e che quindi
                necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
                dell'IP di destinazione affinché risulti nel range degli indirizzi IP
                interni al g-nodo di livello `$subnetlevel`.
            *   Per i pacchetti IP che passano per questo sistema, che hanno per IP di mittente
                un indirizzo IP interno al g-nodo di livello `$subnetlevel` (che cioè
                provengono dall'interno della sottorete autonoma) e che sono destinati ad
                un indirizzo IP globale, esegui la rimappatura
                dell'IP di mittente affinché risulti nel range degli indirizzi IP
                globali e identifichi un nodo
                interno alla mia sottorete autonoma.
            *   Se si vuole che ogni sistema nella sottorete autonoma ammetta di essere contattato in forma anonima:
                *   Per i pacchetti IP che passano per questo sistema e sono destinati ad
                    un indirizzo IP anonimizzante che identifica un nodo
                    interno alla mia sottorete autonoma (di livello `$subnetlevel`) e che quindi
                    necessariamente provengono dall'esterno della sottorete, esegui la rimappatura
                    dell'IP di destinazione affinché risulti nel range degli indirizzi IP
                    interni al g-nodo di livello `$subnetlevel`.  
                    Non è possibile ammettere che qualche nodo sia contattabile in forma anonima
                    senza di fatto renderlo possibile a tutti; in quanto l'indirizzo anonimizzante di
                    destinazione viene rimappato all'indirizzo interno esattamente come viene
                    rimappato l'indirizzo globale.
            *   Naturalmente, il gateway permette ai sistemi interni di contattare un sistema esterno in forma anonima. Quindi:
                *   Per i pacchetti IP che passano per questo sistema, che hanno per IP di mittente
                    un indirizzo IP interno al g-nodo di livello `$subnetlevel` (che cioè
                    provengono dall'interno della sottorete autonoma) e che sono destinati ad
                    un indirizzo IP anonimizzante, esegui la rimappatura
                    dell'IP di mittente affinché risulti nel range degli indirizzi IP
                    globali e identifichi un nodo
                    interno alla mia sottorete autonoma.

Le rimappature sono fatte con l'operazione "NETMAP" fornita da `iptables`.  
L'operazione "NETMAP" nella catena PREROUTING modifica l'IP di destinazione del pacchetto. La
stessa operazione nella catena POSTROUTING modifica l'IP di mittente del pacchetto.

**sistema 𝜆**
```
# Per i = 1
iptables -t nat -A PREROUTING -d 10.0.0.48/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.48/30 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.48/31

# Per i = 2
iptables -t nat -A PREROUTING -d 10.0.0.60/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.56/29 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.60/31

# Per i = 3
iptables -t nat -A PREROUTING -d 10.0.0.12/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.0/27  -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31

# Se vogliamo che ogni sistema nella sottorete autonoma accetti di essere contattato in forma anonima:
iptables -t nat -A PREROUTING -d 10.0.0.76/31 -j NETMAP --to 10.0.0.40/31

# Sicuramente il gateway permette ai sistemi interni di contattare un sistema esterno in forma anonima.
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31
```

Sono lasciate a qualche altro meccanismo nel sistema *𝜆* le operazioni che riguardano l'instradamento
dei pacchetti internamente alla sottorete a gestione autonoma.

Ad esempio, assumiamo che il sistema *𝜆* sia collegato al sistema
*𝜌* attraverso l'interfaccia di rete `eth2`. Allora questa sequenza di operazioni nel sistema
*𝜆* potrebbe essere eseguita (senza alcuna coordinazione con il programma **qspnclient**) all'avvio del sistema:

**sistema 𝜆**
```
ip address add 10.0.111.222 dev eth2
ip route add 169.254.110.188 dev eth1 src 169.254.111.222
ip address add 10.0.0.40 dev eth2
ip route add 10.0.0.41/32 via 169.254.110.188 dev eth2 src 10.0.0.40
```

Analogamente, questa sequenza di operazioni nel sistema
*𝜌* potrebbe essere eseguita all'avvio del sistema:

**sistema 𝜌**
```
sysctl net.ipv4.ip_forward=1
ip address add 10.0.0.32 dev lo
ip link set dev eth1 up
ip address add 169.254.110.188 dev eth1
ip route add 169.254.109.22 dev eth1 src 169.254.110.188
ip address add 10.0.0.41 dev eth1
ip route add 10.0.0.40/32 via 169.254.109.22 dev eth1 src 10.0.0.41
ip route add 10.0.0.0/25 via 169.254.109.22 dev eth1 src 10.0.0.41
```

#### Formazione arco-fisico *𝜆*-*𝜀*

Ora, si formi l'arco-fisico *𝜆*-*𝜀*, dove *𝜀* è un sistema la cui identità principale *𝜀<sub>1</sub>* è
già in una rete. Assumiamo che *𝜆<sub>0</sub>* voglia entrare in questa rete.

In particolare, in questa rete sia *𝜑* un g-nodo di livello 1 composto da *𝜀<sub>1</sub>*, *𝛽<sub>1</sub>* e
*𝛾<sub>0</sub>*. In particolare abbiamo che *𝛽<sub>1</sub>* è un g-nodo di livello 0 di connettività con
la componente a livello 0 *virtuale*.  
Assumiamo che *𝜑* appartiene al g-nodo di livello 2 *𝜓* che è saturo. Assumiamo che *𝜔* sia un g-nodo di
livello 2 non saturo adiacente a *𝜑*. Per permettere l'ingresso di *𝜆* si decide di far migrare *𝜑* da *𝜓*
in *𝜔* per lasciare un posto per *𝜆* dentro *𝜓*.

Dalla rivelazione dell'arco-fisico *𝜆*-*𝜀*, il modulo Neighborhood ha già prodotto questi comandi:

**sistema 𝜆**
```
ip route add 169.254.163.36 dev eth1 src 169.254.109.22
```

**sistema 𝜀**
```
ip route add 169.254.109.22 dev eth1 src 169.254.163.36
```

L'utente ha dato i comandi per accettare tale arco su entrambi i sistemi. Il modulo Identities ha
quindi realizzato per le relative identità principali un nuovo arco-identità *𝜆<sub>0</sub>*-*𝜀<sub>1</sub>*.
L'utente ha annotato l'identificativo del nuovo arco-identità nel sistema *𝜆* e il peer-MAC nel sistema *𝜀*.

Indichiamo con *𝜑<sub>0</sub>* la vecchia identità del g-nodo *𝜑*, composta come dicevamo
da *𝜀<sub>1</sub>*, *𝛽<sub>1</sub>* e *𝛾<sub>0</sub>* in *𝜓*.  
Assumiamo che l'indirizzo di *𝜓* sia 2·1·. Assumiamo che in esso l'indirizzo di *𝜑<sub>0</sub>* sia 2·1·1·.  
Assumiamo che l'indirizzo di *𝜔* sia 2·0·. Assumiamo che in esso sia libero l'indirizzo 2·0·2·.  

L'utente stabilisce la migration path che porta a liberare un posto in *𝜓*.  
Con questa migrazione *𝜑<sub>0</sub>* assume indirizzo *di connettività* 2·1·3· e si costituisce *𝜑<sub>1</sub>*,
un nuovo g-nodo *isomorfo*, composto da *𝜀<sub>2</sub>*, *𝛽<sub>3</sub>* e *𝛾<sub>1</sub>*.  
Temporaneamente *𝜑<sub>1</sub>* assume indirizzo virtuale 2·0·2·. Dopo poco *𝜑<sub>1</sub>* assume indirizzo 2·0·0·.  
L'identità *𝜑<sub>0</sub>* non viene dismessa finché una verifica con `check_connectivity` non darà esito positivo.

La sequenza di istruzioni che l'utente darà ai sistemi sarà questa:

*   Al sistema *𝜆* dà il comando `prepare_enter_net_phase_1`, indicando queste informazioni:
    *   identità entrante: *𝜆<sub>0</sub>*. Sia il duplicato *𝜆<sub>1</sub>*.
    *   livello g-nodo entrante: 1.
    *   g-nodo ospitante *𝜓*:
        *   Livello: 1.
        *   Indirizzo Netsukuku: 2·1·.
        *   Fingerprint a livello 2.
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 3.
    *   nuova posizione reale:
        *   Identificativo: 1.
        *   Anzianità: 4.
    *   posizione di connettività.
        *   Identificativo: 2.
        *   Anzianità: 1.
    *   nuovi archi-qspn: 1.
        *   `identityarc_index`: l'identificativo dell'arco-identità *𝜆<sub>0</sub>*-*𝜀<sub>1</sub>*
            nel sistema *𝜆* prima della duplicazione dell'identità *𝜆<sub>0</sub>*.
    *   identificativo di questa operazione di ingresso: *m<sub>𝜆</sub>*.
    *   identificativo della previa operazione di migrazione: *m<sub>𝜑</sub>*.
*   Al sistema *𝜆* dà il comando `enter_net_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di ingresso *m<sub>𝜆</sub>*.
*   Al sistema *𝜀* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝜀<sub>1</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità con *𝜆<sub>0</sub>* prima della duplicazione.
*   Al sistema *𝜀* dà il comando `prepare_migrate_phase_1`, indicando queste informazioni:
    *   identità migrante: *𝜀<sub>1</sub>*. Sia il duplicato *𝜀<sub>2</sub>*.
    *   livello g-nodo migrante: 1.
    *   g-nodo ospitante *𝜔*:
        *   Livello: 2.
        *   Indirizzo Netsukuku: 2·0·.
        *   Fingerprint a livello 2.
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 3.
    *   nuova posizione reale:
        *   Identificativo: 0.
        *   Anzianità: 4.
    *   posizione di connettività.
        *   Identificativo: 3.
        *   Anzianità: 3.
    *   identificativo di questa operazione di migrazione: *m<sub>𝜑</sub>*.
    *   identificativo della previa operazione di migrazione: nullo.
*   Al sistema *𝛽* dà il comando `prepare_migrate_phase_1`, indicando queste informazioni:
    *   identità migrante: *𝛽<sub>1</sub>*. Sia il duplicato *𝛽<sub>3</sub>*.
    *   livello g-nodo migrante: 1.
    *   g-nodo ospitante *𝜔*:
        *   Livello: 2.
        *   Indirizzo Netsukuku: 2·0·.
        *   Fingerprint a livello 2.
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 3.
    *   nuova posizione reale:
        *   Identificativo: 0.
        *   Anzianità: 4.
    *   posizione di connettività.
        *   Identificativo: 3.
        *   Anzianità: 3.
    *   identificativo di questa operazione di migrazione: *m<sub>𝜑</sub>*.
    *   identificativo della previa operazione di migrazione: nullo.
*   Al sistema *𝛾* dà il comando `prepare_migrate_phase_1`, indicando queste informazioni:
    *   identità migrante: *𝛾<sub>0</sub>*. Sia il duplicato *𝛾<sub>1</sub>*.
    *   livello g-nodo migrante: 1.
    *   g-nodo ospitante *𝜔*:
        *   Livello: 2.
        *   Indirizzo Netsukuku: 2·0·.
        *   Fingerprint a livello 2.
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 3.
    *   nuova posizione reale:
        *   Identificativo: 0.
        *   Anzianità: 4.
    *   posizione di connettività.
        *   Identificativo: 3.
        *   Anzianità: 3.
    *   identificativo di questa operazione di migrazione: *m<sub>𝜑</sub>*.
    *   identificativo della previa operazione di migrazione: nullo.
*   Al sistema *𝜀* dà il comando `migrate_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di migrazione *m<sub>𝜑</sub>*.
*   Al sistema *𝛽* dà il comando `migrate_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di migrazione *m<sub>𝜑</sub>*.
*   Al sistema *𝛾* dà il comando `migrate_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di migrazione *m<sub>𝜑</sub>*.
*   Al sistema *𝛽* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝛽<sub>2</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità con *𝛾<sub>0</sub>* prima della duplicazione.
*   Al sistema *𝛽* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝛽<sub>2</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità con *𝜀<sub>1</sub>* prima della duplicazione.
*   Al sistema *𝛿* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝛿<sub>1</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità con *𝛾<sub>0</sub>* prima della duplicazione.
*   Al sistema *𝜆* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝜆<sub>1</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità con *𝜀<sub>1</sub>* prima della duplicazione.
*   Al sistema *𝜆* dà il comando `enter_net_phase_2`, indicando queste informazioni:
    *   è stata completata la migrazione *m<sub>𝜑</sub>*; quindi è ora disponibile l'indirizzo *reale* dentro *𝜓*.

[Operazione seguente](DettagliOperazioni12.md)
