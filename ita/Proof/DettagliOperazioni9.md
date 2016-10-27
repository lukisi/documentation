# Proof of concept - Dettagli Operazioni - Pagina 9

[Operazione precedente](DettagliOperazioni8.md)

### <a name="Migrazione_ingresso_1"></a> Migrazione per ingresso - Caso 1

#### Migrazione: Popolamento nuove rotte della nuova identità

Poi il programma **qspnclient**, sempre a seguito del comando `migrate_phase_1`, provvede come vediamo
adesso a preparare il vecchio namespace per la nuova identità.

Il programma **qspnclient** calcola tutti i possibili indirizzi IP di destinazione relativi all'indirizzo
della nuova identità. Li memorizza associandoli a questa nuova identità.

Nel vecchio network namespace abbiamo alcune tabelle pre-esistenti (che erano usate per la vecchia identità *𝛽<sub>1</sub>*)
e che vanno mantenute in quanto saranno usate dalla nuova identità *𝛽<sub>2</sub>*. In questo caso si tratta nel namespace default
della tabella `ntk` e delle tabelle di inoltro per gli archi-qspn relativi agli archi-identità duplicati dagli
archi-identità di *𝛽<sub>1</sub>* con *𝛼<sub>1</sub>*, *𝛾<sub>0</sub>* e *𝜀<sub>1</sub>*.

Nel caso in esame, poiché il livello del g-nodo che migra è 0, non ci sono indirizzi IP di possibili destinazioni
che non erano stati rimossi dalle tabelle nella precedente fase. Altrimenti andrebbero considerati.  
Nel nostro caso tutti gli indirizzi IP di destinazione appena calcolati vanno ora aggiunti alle tabelle pre-esistenti
nel vecchio namespace.

**sistema 𝛽**
```
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.20/30 table ntk
ip route add unreachable 10.0.0.84/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.16/31 table ntk
ip route add unreachable 10.0.0.80/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.18/32 table ntk
ip route add unreachable 10.0.0.82/32 table ntk
ip route add unreachable 10.0.0.58/32 table ntk
ip route add unreachable 10.0.0.50/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
ip route add unreachable 10.0.0.19/32 table ntk
ip route add unreachable 10.0.0.83/32 table ntk
ip route add unreachable 10.0.0.59/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.18/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.82/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.58/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.18/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.82/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.58/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.18/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.82/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.58/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33
```

#### Comando add_qspn_arc al sistema *𝛼*

**sistema 𝛼**
```
(echo; echo "249 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

#### Comando add_qspn_arc al sistema *𝛾*

**sistema 𝛾**
```
(echo; echo "248 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 248
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

#### Comando add_qspn_arc al sistema *𝜀*

**sistema 𝜀**
```
(echo; echo "249 ntk_from_00:16:3E:EC:A3:E1 # xxx_table_ntk_from_00:16:3E:EC:A3:E1_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
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

#### Migrazione: Cambio di indirizzo di una identità

Sempre quando l'utente dà il comando `migrate_phase_1`, in seguito alle operazioni viste prima, il
programma **qspnclient** sulla base delle istruzioni che l'utente aveva dato nel comando `prepare_migrate_phase_1`,
dopo aver fatto passare la nuova identità per l'indirizzo con posizione *virtuale* al livello 0, cambia il
suo identificativo di livello 0 con la posizione *reale* dentro il g-nodo destinazione. Cioè comunica questa
variazione alla relativa istanza del modulo Qspn.

Oltre a ciò, il programma **qspnclient** esegue di seguito le due sequenze di operazioni che vedremo
adesso: la prima perché un identificativo dell'indirizzo di una sua identità passa da *virtuale*
a *reale* e la seconda perchè questo cambio avviene nell'identità *principale*.

##### Un identificativo passa da virtuale a reale

In questa occasione il programma **qspnclient** rimuove, nel network namespace interessato, da tutte le
tabelle presenti (anche quelle di inoltro il cui arco non ha ancora ricevuto alcun ETP) le rotte verso
gli indirizzi che non sono più validi a causa di questo nuovo identificativo *reale*.

Inoltre il programma **qspnclient** aggiunge, nel network namespace interessato, su tutte le tabelle
presenti (anche quelle di inoltro il cui arco non ha ancora ricevuto alcun ETP) le rotte verso gli indirizzi
IP che sono diventati validi come possibili destinazioni a causa di questo nuovo identificativo reale.  
Inoltre, per le rotte aggiunte il programma **qspnclient** subito le aggiorna in base alle conoscenze di
routing del relativo modulo Qspn.

Notiamo che questo avviene anche nelle identità *di connettività* (che gestiscono un namespace diverso
dal default) e anche se l'indirizzo Netsukuku non è del tutto *reale*.

**sistema 𝛽**
```
ip route del 10.0.0.19/32 table ntk
ip route del 10.0.0.83/32 table ntk
ip route del 10.0.0.59/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
ip route del 10.0.0.19/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.83/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.59/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.19/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.83/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.59/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:FD:E2:AA
ip route del 10.0.0.19/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.83/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.59/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33
```

##### L'identità interessata dal cambio di indirizzo è la principale

In questa occasione il programma **qspnclient** aggiunge ad ogni interfaccia di rete reale nel network
namespace default gli indirizzi propri che prima non era possibile computare e adesso invece sì, partendo
dal livello del nuovo g-nodo che si è costituito nella nuova rete e salendo finché possibile.

Poi, solo se l'indirizzo è ora del tutto *reale*, aggiunge (opzionalmente) la regola di source-natting e
(opzionalmente) l'indirizzo IP anonimizzante.

Infine aggiorna tutte le rotte nella tabella `ntk` per fare in modo di mettere in esse (se disponibile) un src preferito.

**sistema 𝛽**
```
ip address add 10.0.0.41 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.59 dev eth1
ip address add 10.0.0.19 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.19
ip address add 10.0.0.83 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.20/30 table ntk
ip route change unreachable 10.0.0.84/30 table ntk
ip route change unreachable 10.0.0.60/30 table ntk
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.18/32 table ntk
ip route change unreachable 10.0.0.82/32 table ntk
ip route change unreachable 10.0.0.58/32 table ntk
ip route change unreachable 10.0.0.50/32 table ntk
ip route change unreachable 10.0.0.40/32 table ntk
```

#### Migrazione: Rimozione archi esterni

A questo punto delle operazioni svolte a fronte del comando `migrate_phase_1`, il programma **qspnclient**
avvia una nuova tasklet che attenda la produzione del primo ETP da parte della nuova identità.

Nel frattempo avremo che la processazione di ETP provenienti dai vicini fa aggiornare le rotte nelle varie
tabelle di routing e le regole per le tabelle di inoltro. Secondo le modalità che abbiamo visto in precedenza
e quindi con altri comandi che non riporteremo qui nuovamente.

Dopo che la nuova identità *𝛽<sub>2</sub>* ha prodotto e trasmesso il suo primo ETP, dopo aver atteso
qualche istante affinché i vicini abbiano il tempo di processarlo, il programma **qspnclient** (nella tasklet
di cui abbiamo detto sopra) rimuove dalla vecchia identità *𝛽<sub>1</sub>* gli archi esterni al g-nodo *di connettività*.

Questa operazione viene fatta attraverso l'istanza di QspnManager associata a *𝛽<sub>1</sub>*. Se questa operazione
effettivamente rimuove alcuni archi-identità (che sicuramente erano associati ad archi-qspn) allora il
programma **qspnclient** riceverà il segnale `arc_removing` del modulo Identities e si comporterà di
conseguenza come descritto in precedenza [qui](DettagliOperazioni5.md#Rimosso_vicino_stessa_rete).

**sistema 𝛽**
```
ip netns exec migr01 ip rule del fwmark 249 table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 ip route flush table ntk_from_00:16:3E:FD:E2:AA
ip netns exec migr01 iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:FD:E2:AA -j MARK --set-mark 249

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip netns exec migr01 ip route del 169.254.69.30 dev migr01_eth1 src 169.254.27.218
```

[Operazione seguente](DettagliOperazioni10.md)
