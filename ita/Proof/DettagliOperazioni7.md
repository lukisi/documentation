# Proof of concept - Dettagli Operazioni - Pagina 7

[Operazione precedente](DettagliOperazioni6.md)

### <a name="Migrazione_ingresso_1"></a> Migrazione per ingresso - Caso 1

Sia *𝜀* un sistema. In esso l'identità principale *𝜀<sub>0</sub>* costituisce una rete a sé. Esso si
incontra con un diverso sistema *𝛽*. In esso l'identità principale *𝛽<sub>1</sub>* appartiene (in una
diversa rete) ad un g-nodo di livello 1, *𝜑*, che è saturo.

Il modulo Neighborhood ha già prodotto questi comandi:

**sistema 𝜀**
```
ip route add 169.254.96.141 dev eth1 src 169.254.163.36
```

**sistema 𝛽**
```
ip route add 169.254.163.36 dev eth1 src 169.254.96.141
```

L'utente ha dato i comandi per accettare tale arco su entrambi i sistemi. Il modulo Identities ha
quindi realizzato per le relative identità principali un nuovo arco-identità *𝜀<sub>0</sub>*-*𝛽<sub>1</sub>*.
L'utente ha annotato l'identificativo del nuovo arco-identità nel sistema *𝜀* e il peer-MAC nel sistema *𝛽*.

L'utente stabilisce la migration path che porta a liberare un posto in *𝜑*. Il nodo *𝛽<sub>1</sub>* è
un border-nodo di *𝜑*: infatti esso ha un arco verso *𝛼<sub>1</sub>* (l'identità principale nel
sistema *𝛼*) che appartiene al g-nodo *𝜓* di livello 1. Il g-nodo *𝜓* ha un posto libero. L'utente quindi
si annota l'identificativo dell'arco-identità *𝛽<sub>1</sub>*-*𝛼<sub>1</sub>* nel sistema *𝛽* e
il relativo peer-MAC nel sistema *𝛼*.

La sequenza di istruzioni che l'utente darà ai sistemi sarà questa:

*   Al sistema *𝜀* dà il comando `prepare_enter_net_phase_1`, indicando queste informazioni:
    *   identità entrante: *𝜀<sub>0</sub>*. Sia il duplicato *𝜀<sub>1</sub>*.
    *   livello g-nodo entrante: 0.
    *   g-nodo ospitante *𝜑*:
        *   Livello: 1.
        *   Indirizzo Netsukuku: 2·1·1·.
        *   Fingerprint a livello 1.
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
        *   `identityarc_index`: l'identificativo dell'arco-identità *𝜀<sub>0</sub>*-*𝛽<sub>1</sub>*
            nel sistema *𝜀* prima della duplicazione dell'identità *𝜀<sub>0</sub>*.
    *   identificativo di questa operazione di ingresso: *m<sub>𝜀</sub>*.
    *   identificativo della previa operazione di migrazione: *m<sub>𝛽</sub>*.
*   Al sistema *𝜀* dà il comando `enter_net_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di ingresso *m<sub>𝜀</sub>*.
*   Al sistema *𝛽* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝛽<sub>1</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità *𝜀<sub>0</sub>*-*𝛽<sub>1</sub>* nel
        sistema *𝛽* prima della duplicazione dell'identità *𝜀<sub>0</sub>*.  
        È necessario che il comando `add_qspn_arc` nel sistema *𝛽* venga dato dopo il completamento
        del comando `enter_net_phase_1` nel sistema *𝜀* con sufficiente ritardo, ma in tempi rapidi.
*   Al sistema *𝛽* dà il comando `prepare_migrate_phase_1`, indicando queste informazioni:
    *   identità migrante: *𝛽<sub>1</sub>*. Sia il duplicato *𝛽<sub>2</sub>*.
    *   livello g-nodo migrante: 0.
    *   g-nodo ospitante *𝜓*:
        *   Livello: 1.
        *   Indirizzo Netsukuku: 2·0·1·.
        *   Fingerprint a livello 1.
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 3.
    *   nuova posizione reale:
        *   Identificativo: 1.
        *   Anzianità: 4.
    *   posizione di connettività.
        *   Identificativo: 3.
        *   Anzianità: 3.
    *   identificativo di questa operazione di ingresso: *m<sub>𝛽</sub>*.
    *   identificativo della previa operazione di migrazione: nullo.
*   Al sistema *𝛽* dà il comando `migrate_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di migrazione *m<sub>𝛽</sub>*.
*   Al sistema *𝛼* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝛼<sub>1</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità *𝛽<sub>1</sub>*-*𝛼<sub>1</sub>* nel
        sistema *𝛼* prima della duplicazione dell'identità *𝛽<sub>1</sub>*.  
        È necessario che il comando `add_qspn_arc` nel sistema *𝛼* venga dato dopo il completamento
        del comando `migrate_phase_1` nel sistema *𝛽* con sufficiente ritardo, ma in tempi rapidi.
*   Al sistema *𝜀* dà il comando `enter_net_phase_2`, indicando queste informazioni:
    *   è stata completata la migrazione *m<sub>𝛽</sub>*; quindi è ora disponibile l'indirizzo *reale* dentro *𝜑*.

#### Comando prepare_enter_net_phase_1 al sistema *𝜀*

Il programma **qspnclient** chiama il metodo `prepare_add_identity` del modulo Identities e memorizza le
informazioni relative all'ingresso.

#### Comando enter_net_phase_1 al sistema *𝜀*

Il programma **qspnclient** recupera le informazioni relative all'ingresso e chiama il metodo `add_identity`
del modulo Identities. Il modulo Identities produce quindi queste operazioni:

**sistema 𝜀**
```
ip netns add entr05
ip netns exec entr05 sysctl net.ipv4.ip_forward=1
ip netns exec entr05 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr05_eth1 link eth1 type macvlan
ip link set dev entr05_eth1 netns entr05
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.rp_filter=0
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.arp_ignore=1
ip netns exec entr05 sysctl net.ipv4.conf.entr05_eth1.arp_announce=2
ip netns exec entr05 ip link set dev entr05_eth1 up
ip netns exec entr05 ip address add 169.254.133.31 dev entr05_eth1
ip netns exec entr05 ip route add 169.254.96.141 dev entr05_eth1 src 169.254.133.31
```

Il programma **qspnclient** memorizza lo scambio di network namespace tra le due identità *𝜀<sub>0</sub>* e
*𝜀<sub>1</sub>*.

Nel sistema *𝛽* il modulo Identities in autonomia (a fronte del comando `enter_net_phase_1` dato nel sistema
*𝜀*) produce queste operazioni:

**sistema 𝛽**
```
ip route add 169.254.133.31 dev eth1 src 169.254.96.141
```

#### <a name="Spostamento_rotte_identita"></a> Spostamento delle rotte della vecchia identità

Il programma **qspnclient** decide quali tabelle di inoltro vanno usate nel nuovo network namespace.
In questo caso, nessuna. Infatti il sistema *𝜀* era isolato.

Il programma **qspnclient** calcola l'indirizzo della vecchia identità nel nuovo namespace a partire
dall'indirizzo Netsukuku precedente della vecchia identità (assumiamo sia 3·1·0·0), sostituendo al
livello *"livello g-nodo entrante"* la *"posizione di connettività"*. Quindi in questo caso abbiamo 3·1·0·2.
Calcola tutti i possibili indirizzi IP di destinazione e li memorizza associandoli a quella identità.

Ma prima recupera tutti i possibili indirizzi IP di destinazione relativi all'indirizzo che quella identità
aveva nel vecchio namespace, cioè 3·1·0·0, e che ora non sono più validi.  
Cioè, relativamente a tutte le destinazioni, gli indirizzi IP globali e quelli interni ai g-nodi di livello
maggiore del livello del g-nodo che fa ingresso in blocco.  
Le rotte verso tali indirizzi vanno rimosse dalle tabelle presenti nel vecchio namespace. Nel caso in
esame si tratta di tutte le rotte (in quanto il g-nodo che fa ingresso è di livello 0) nella tabella `ntk` nel namespace default.

**sistema 𝜀**
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
ip route del 10.0.0.29/32 table ntk
ip route del 10.0.0.93/32 table ntk
ip route del 10.0.0.61/32 table ntk
ip route del 10.0.0.49/32 table ntk
ip route del 10.0.0.41/32 table ntk
```

Partendo dal livello del nuovo g-nodo che si è costituito nella nuova rete (nel nostro caso 0) e salendo fino a
*l* - 1, solo se il vecchio namespace è il default (come nel nostro caso), il programma **qspnclient**
rimuove dal vecchio namespace gli indirizzi IP della vecchia identità che non saranno comuni
con quelli della nuova identità. In particolare, dovendo rimuovere l'indirizzo IP globale, prima
rimuove la regola (se presente) di source-natting per i pacchetti anonimi che transitano per questo
sistema. Rimuove anche (se presente) l'indirizzo IP anonimizzante.

**sistema 𝜀**
```
ip address del 10.0.0.40/32 dev eth1
ip address del 10.0.0.48/32 dev eth1
ip address del 10.0.0.60/32 dev eth1
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.28
ip address del 10.0.0.28/32 dev eth1
ip address del 10.0.0.92/32 dev eth1
```

#### Popolamento nuove rotte della nuova identità

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** provvede come vediamo adesso a preparare il vecchio namespace
per la nuova identità.

Il programma **qspnclient** calcola tutti i possibili indirizzi IP di destinazione relativi all'indirizzo
della nuova identità. Li memorizza associandoli a questa nuova identità.

Nel vecchio network namespace abbiamo alcune tabelle pre-esistenti (che erano usate per la vecchia
identità) e che vanno mantenute in quanto saranno usate dalla nuova identità. In questo caso si tratta della
sola tabella `ntk` nel namespace default.

Nel caso in esame, poiché il livello del g-nodo che entra è 0, non ci sono indirizzi IP di possibili
destinazioni che non erano stati rimossi dalle tabelle nella precedente fase. Altrimenti andrebbero considerati.  
Nel nostro caso tutti gli indirizzi IP di destinazione appena calcolati vanno ora aggiunti alle tabelle
pre-esistenti nel vecchio namespace.

**sistema 𝜀**
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
comunica (nel costruttore) di quali archi-qspn dispone.

Nel nostro esempio ci sarà un nuovo
arco-qspn, comunicato dall'utente nel comando `prepare_enter_net_phase_1`. Di tale arco-identità il
programma conosce:

*   MAC address del vicino.
*   Indirizzo IP linklocal del vicino.

Tenendo traccia inoltre delle tabelle già inserite nel file `/etc/iproute2/rt_tables`, il programma è in grado di
eseguire le operazioni che seguono. Lo deve fare per tutti gli archi-qspn passati al QspnManager. Come sempre,
vanno eseguite in blocco senza intromissioni da altre tasklet.

**sistema 𝜀**
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
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

#### Comando add_qspn_arc al sistema *𝛽*

Quando l'utente dà il comando `add_qspn_arc` nel sistema *𝛽*, il programma **qspnclient**
aggiunge sulla relativa istanza di QspnManager un nuovo arco-qspn. Di tale arco-identità il
programma conosce:

*   MAC address del vicino.
*   Indirizzo IP linklocal del vicino.

Il programma è in grado di eseguire le operazioni che seguono. Come sempre,
vanno eseguite in blocco senza intromissioni da altre tasklet.

**sistema 𝛽**
```
(echo; echo "248 ntk_from_00:16:3E:3C:14:33 # xxx_table_ntk_from_00:16:3E:3C:14:33_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3C:14:33 -j MARK --set-mark 248
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
```

#### Dismissione identità

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** dismette la vecchia identità.

**sistema 𝜀**
```
ip netns exec entr05 ip route flush table main
ip netns exec entr05 ip link delete entr05_eth1 type macvlan
ip netns del entr05
```

Nel sistema *𝛽* il modulo Identities in autonomia (a fronte della dismissione della vecchia identità di
*𝜀*) produce queste operazioni:

**sistema 𝛽**
```
ip route del 169.254.133.31 dev eth1 src 169.254.96.141
```

[Operazione seguente](DettagliOperazioni8.md)
