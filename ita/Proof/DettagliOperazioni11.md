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
*   Indichiamo con *subnetlevel* il livello del g-nodo rappresentato dalla sottorete autonoma.
*   Se `$subnetlevel` > 0:
    *   Sia *range1* l'indirizzo IP con suffisso CIDR che contiene tutti e solo gli indirizzi IP
        di tipo interno di livello *subnetlevel*; in altre parole, che rappresenta la sottorete
        autonoma dentro il suo g-nodo di livello *subnetlevel*.  
        Nell'esempio di *𝜆*: `10.0.0.40/31`.
    *   Per *i* che sale da `$subnetlevel` a *l* - 1:
        *   Se *i* < *l* - 1:
            *   Sia *range2* l'indirizzo IP con suffisso CIDR che rappresenta la sottorete
                autonoma dentro il suo g-nodo di livello *i* + 1. Si basa sulle posizioni di *n*
                da *subnetlevel* a *i*.  
                Nell'esempio `10.0.0.50/31` per *i* = 1.  
                E `10.0.0.62/31` per *i* = 2.
            *   Sia *g* il g-nodo di livello *i* + 1 di cui fa parte *n*.  
                Sia *range3* l'indirizzo IP con suffisso CIDR che comprende l'insieme di tutti
                i nodi in *g* rappresentati con un indirizzo IP interno al g-nodo *g*. Si basa sulla posizione di *n*
                al livello *i*.  
                Nell'esempio `10.0.0.48/30` per *i* = 1.  
                E `10.0.0.56/29` per *i* = 2.
            *   Il programma **qspnclient** esegue:  
                `iptables -t nat -A PREROUTING -d $range2 -j NETMAP --to $range1`  
                `iptables -t nat -A POSTROUTING -d $range3 -s $range1 -j NETMAP --to $range2`
        *   Altrimenti (cioè per *i* = *l* - 1):
            *   Sia *range2* l'indirizzo IP con suffisso CIDR che rappresenta la sottorete
                autonoma dentro tutta la rete Netsukuku. Si basa sulle posizioni di *n*
                da *subnetlevel* a *l* - 1.  
                Nell'esempio `10.0.0.22/31`.
            *   Sia *range3* l'indirizzo IP con suffisso CIDR che comprende l'insieme di tutti
                i nodi nella rete Netsukuku rappresentati con indirizzo IP globale.  
                Nell'esempio `10.0.0.0/27`.
            *   Il programma **qspnclient** esegue:  
                `iptables -t nat -A PREROUTING -d $range2 -j NETMAP --to $range1`  
                `iptables -t nat -A POSTROUTING -d $range3 -s $range1 -j NETMAP --to $range2`
            *   Se ogni sistema nella sottorete autonoma accetta di essere contattato in forma anonima:
                *   Sia *range4* l'indirizzo IP con suffisso CIDR che rappresenta la sottorete
                    autonoma dentro tutta la rete Netsukuku con indirizzo IP anonimizzante. Si basa sulle posizioni di *n*
                    da *subnetlevel* a *l* - 1.  
                    Nell'esempio `10.0.0.86/31`.
                *   Il programma **qspnclient** esegue:  
                    `iptables -t nat -A PREROUTING -d $range4 -j NETMAP --to $range1`  
            *   Sia *range5* l'indirizzo IP con suffisso CIDR che comprende l'insieme di tutti
                i nodi nella rete Netsukuku rappresentati con indirizzo IP anonimizzante.  
                Nell'esempio `10.0.0.64/27`.
            *   Il programma **qspnclient** esegue:  
                `iptables -t nat -A POSTROUTING -d $range5 -s $range1 -j NETMAP --to $range2`

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

#### Comando prepare_enter_net_phase_1 al sistema *𝜆*

Analogo a quanto visto nel caso 1.

#### Comando enter_net_phase_1 al sistema *𝜆*

Creazione del nuovo network namespace: analogo a quanto visto nel caso 1.

#### Spostamento delle rotte della vecchia identità

Analogo a quanto visto nel caso 1, con alcune accortezze.

Visto che il livello della sottorete autonoma è 1, si considerano le possibili destinazioni
solo fino ai g-nodi di livello 1. Inoltre per ogni possibile g-nodo destinazione, visto che il livello
del g-nodo che entra in blocco è 1, si rimuovono gli indirizzi IP interni ai g-nodi solo di livello maggiore 
di 1.

Infine, siccome si rimuoveranno i propri indirizzi IP interni (oltre al globale e anonimizzante) ai
g-nodi di livello maggiore di 1, si rimuovono anche le relative regole di rimappatura.

**sistema 𝜆**
```
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/29 table ntk
ip route del 10.0.0.88/29 table ntk
ip route del 10.0.0.8/30 table ntk
ip route del 10.0.0.72/30 table ntk
ip route del 10.0.0.56/30 table ntk
ip route del 10.0.0.14/31 table ntk
ip route del 10.0.0.78/31 table ntk
ip route del 10.0.0.62/31 table ntk
ip route del 10.0.0.50/31 table ntk

iptables -t nat -D PREROUTING -d 10.0.0.48/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -D POSTROUTING -d 10.0.0.48/30 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.48/31

iptables -t nat -D PREROUTING -d 10.0.0.60/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -D POSTROUTING -d 10.0.0.56/29 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.60/31

iptables -t nat -D PREROUTING -d 10.0.0.12/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -D POSTROUTING -d 10.0.0.0/27  -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31

iptables -t nat -D PREROUTING -d 10.0.0.76/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.12/31

ip address del 10.0.0.48/32 dev eth1
ip address del 10.0.0.60/32 dev eth1
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.12
ip address del 10.0.0.12/32 dev eth1
ip address del 10.0.0.76/32 dev eth1
```

#### Popolamento nuove rotte della nuova identità

Analogo a quanto visto nel caso 1.

#### Creazione e popolamento iniziale di tabelle per l'inoltro

Analogo a quanto visto nel caso 1.

#### Comando add_qspn_arc al sistema *𝜀*

Analogo a quanto visto nel caso 1.

#### Dismissione identità

Analogo a quanto visto nel caso 1.

[Operazione seguente](DettagliOperazioni12.md)
