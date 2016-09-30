# Proof of concept - Dettagli Operazioni - Pagina 4

[Operazione precedente](DettagliOperazioni3.md)

### <a name="Ingresso_rete_1"></a> Ingresso in una rete - Caso 1

Sia *𝛼* un singolo nodo che costituiva una rete a sé; esso si incontra con un
diverso singolo nodo *𝛽*; il nodo *𝛽* appartiene ad un g-nodo di livello 1 che ha una posizione
libera per *𝛼*.

La sequenza di istruzioni che l'utente darà ai singoli nodi *𝛼* e *𝛽* sarà questa:

*   Al sistema *𝛼* dà il comando `prepare_enter_net_phase_1`, indicando queste informazioni:
    *   identità entrante. L'identificativo di una identità di *𝛼*. Sia in questo esempio *𝛼<sub>0</sub>*.
    *   livello g-nodo entrante *k*. Il livello del g-nodo che fa ingresso. In questo esempio è 0.  
        L'identità *𝛼<sub>0</sub>* deve produrre una copia (sia essa *𝛼<sub>1</sub>*) che farà
        ingresso insieme al suo g-nodo di livello *k*. Chiamiamo questo g-nodo *𝜑* e il nuovo g-nodo isomorfo *𝜑'*.
        In questo esempio *𝜑* è costituito dal solo *𝛼<sub>0</sub>* e *𝜑'* è costituito dal solo *𝛼<sub>1</sub>*.
    *   g-nodo ospitante. Le informazioni riguardanti il g-nodo *𝜒*, il quale può ospitare *𝜑'* nella
        nuova rete. Consistono nell'indirizzo Netsukuku di *𝜒* e il suo fingerprint.  
        Sicuramente *𝜒* è di livello maggiore del livello di *𝜑'*, forse anche di più livelli.
    *   nuova posizione 1. Le informazioni riguardanti la posizione da assumere dentro *𝜒*. Consistono
        nella posizione e l'anzianità del g-nodo di livello direttamente inferiore a *𝜒* che potrà
        immediatamente assumere *𝜑'* nella nuova rete. Questa posizione è temporanea e *virtuale*.
    *   nuova posizione 2. Sarà in seguito resa disponibile dentro *𝜒* questa posizione *reale*.
    *   posizione di connettività. Le informazioni riguardanti la posizione di connettività nella
        vecchia rete che dovrà assumere *𝜑*. Consistono nella posizione *virtuale* e l'anzianità del g-nodo di
        livello direttamente superiore a *𝜑*.
    *   l'identificativo di questa operazione di ingresso. Chiamiamolo *m<sub>𝜑</sub>*.
    *   l'identificativo dell'operazione di migrazione (eventuale) al termine della quale si potrà
        prendere la posizione *reale* di cui sopra dentro *𝜒*. Chiamiamolo *m<sub>𝜓</sub>*, ad indicare che
        per liberare la posizione ha migrato il g-nodo *𝜓*. Laddove non vi sia bisogno di alcuna
        migration path, questo identificativo è nullo.
*   Al sistema *𝛼* dà il comando `enter_net_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di ingresso *m<sub>𝜑</sub>*.
    *   se *m<sub>𝜓</sub>* era nullo: è implicita la richiesta di procedere immediatamente dopo con l'assegnazione
        dell'indirizzo *reale* dentro *𝜒*.
*   Soltanto se *m<sub>𝜓</sub>* non è nullo: al sistema *𝛼* dà il comando `enter_net_phase_2`, indicando queste informazioni:
    *   è stata completata la migrazione *m<sub>𝜓</sub>*; quindi è ora disponibile l'indirizzo *reale* dentro *𝜒*.

Risulta chiaro che in questo caso banale il tutto si sarebbe potuto fare con un solo comando dato dall'utente
nel sistema *𝛼* e che si poteva fare a meno di passare per l'indirizzo temporaneo *virtuale*. Ma per
semplicità manteniamo la sola modalità generica.

#### Comando prepare_enter_net_phase_1

Quando l'utente dà il comando `prepare_enter_net_phase_1` fornisce tutte le informazioni. Il programma
**qspnclient** a questo punto chiama il metodo `prepare_add_identity` del modulo Identities. Oltre a ciò,
non esegue nessuna operazione (comando al S.O.) ma soltanto memorizza le informazioni ricevute.

#### Comando enter_net_phase_1

Quando l'utente dà il comando `enter_net_phase_1` fornisce l'identificativo dell'operazione di
ingresso. Il programma **qspnclient** recupera le informazioni memorizzate prima. Poi chiama il
metodo `add_identity` del modulo Identities. Il modulo Identities produce quindi queste operazioni:

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

Nei comandi sopra riportati, il nome del network namespace (`entr02`) i nomi delle interfacce
di rete reali e pseudo (`eth1`, `entr02_eth1`) i relativi indirizzi di scheda (`169.254.215.29`)
e gli indirizzi link-local dei diretti vicini (`169.254.94.223`) sono tutti gestiti in
autonomia dal modulo Identities.

Dall'esecuzione del metodo `add_identity` sul modulo Identities è necessario reperire:
*   identificativo della nuova identità
*   nome del vecchio network namespace (che passa dalla vecchia identità alla nuova)
*   nome del nuovo network namespace (che nasce per la vecchia identità)
*   associazione tra ogni arco-identità della vecchia identità quando era nel vecchio namespace
    e il corrispettivo arco-identità della vecchia identità nel nuovo namespace (oppure basta
    che data una identità e un arco-identità passato al modulo Qspn possa recuperare le
    relative informazioni attuali?)

#### Copia tabelle e regole, spostamento rotte

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** provvede come vediamo adesso a spostare la vecchia identità
nel nuovo namespace e ripulire il vecchio namespace per la nuova identità.

Il programma **qspnclient** conosce il nome del nuovo namespace. Assumiamo sia `entr02`.

Il programma **qspnclient** conosce il nome del vecchio namespace. Assumiamo sia ``, cioè il default.

Il programma **qspnclient** conosce l'indirizzo Netsukuku che la vecchia identità aveva nel vecchio
namespace. Assumiamo sia 1·0·1·0.

Il programma **qspnclient** conosce l'indirizzo Netsukuku *virtuale* che la vecchia identità come supporto
di connettività assume nel nuovo namespace. Assumiamo sia 1·0·1·3.

Il programma **qspnclient** aggiunge nel nuovo namespace la regola per la tabella primaria `ntk`.

```
ip netns exec entr02 ip rule add table ntk
```

Richiamando le modalità viste [qui](DettagliOperazioni1.md), il programma **qspnclient** calcola tutti i possibili indirizzi IP
di destinazione, ognuno con suffisso CIDR, relativi all'indirizzo della vecchia identità nel nuovo
namespace, cioè 1·0·1·3. E li aggiunge alla tabella primaria `ntk` nel nuovo namespace.

```
ip netns exec entr02 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.12/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.76/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.60/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.8/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.72/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.56/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.11/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.75/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.59/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.51/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.41/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.10/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.74/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.58/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.50/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.40/32 table ntk
```

Con le stesse modalità, il programma **qspnclient** calcola tutti i possibili indirizzi IP
di destinazione relativi all'indirizzo che la vecchia identità aveva nel vecchio
namespace, cioè 1·0·1·0. E li rimuove dalla tabella primaria `ntk` nel vecchio namespace.

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
```

Partendo dal livello del g-nodo entrante *k* (nel nostro caso 0) e salendo fino a
*l* - 1, solo se il vecchio namespace è il default (come nel nostro caso), il programma **qspnclient**
rimuove dal vecchio namespace gli indirizzi IP della vecchia identità che non saranno comuni
con quelli della nuova identità. In particolare, dovendo rimuovere l'indirizzo IP globale, prima
rimuove la regola (se presente) di source-natting per i pacchetti anonimi che transitano per questo
sistema. Rimuove anche (se presente) l'indirizzo IP anonimizzante.

```
ip address del 10.0.0.40/32 dev eth1
ip address del 10.0.0.50/32 dev eth1
ip address del 10.0.0.58/32 dev eth1
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.10
ip address del 10.0.0.10/32 dev eth1
ip address del 10.0.0.74/32 dev eth1
```

Di nuovo con tutti i possibili indirizzi IP di destinazione relativi all'indirizzo della vecchia identità nel nuovo
namespace, il programma **qspnclient** aggiorna tutte le rotte nella tabella primaria `ntk` nel nuovo namespace.

```
ip netns exec entr02 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.12/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.76/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.60/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.8/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.72/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.56/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.11/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.75/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.59/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.51/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.41/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.10/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.74/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.58/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.50/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.40/32 table ntk
```

#### Popolamento nuove rotte della nuova identità

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** provvede come vediamo adesso a preparare il vecchio namespace
per la nuova identità.

Il programma **qspnclient** calcola tutti i possibili indirizzi IP di destinazione relativi all'indirizzo
della nuova identità. Li aggiunge alla tabella primaria `ntk` nel vecchio namespace.

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

#### Creazione e popolamento iniziale di tabelle per l'inoltro

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** crea una istanza di QspnManager per la sua nuova identità e gli
comunica (nel costruttore) che ha un arco-qspn con un altro nodo. Di tale arco-identità il
programma conosce:

*   MAC address del vicino.
*   Indirizzo IP linklocal del vicino.

Tenendo traccia inoltre delle tabelle già inserite nel file `/etc/iproute2/rt_tables`, il programma è in grado di
eseguire le operazioni che seguono. Lo deve fare per tutti gli archi-qspn passati al QspnManager. Come sempre,
vanno eseguite in blocco senza intromissioni da altre tasklet.

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

[Operazione seguente](DettagliOperazioni5.md)
