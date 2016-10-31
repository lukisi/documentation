# Proof of concept - Dettagli Operazioni - Pagina 12

[Operazione precedente](DettagliOperazioni11.md)

### <a name="Migrazione_ingresso_2"></a> Migrazione per ingresso - Caso 2

Ricordiamo che la sequenza di operazioni deve essere questa: prima il comando `prepare_migrate_phase_1`
viene dato ai sistemi *𝜀*, *𝛽* e *𝛾*. Poi in tempi radidi il comando `migrate_phase_1`
viene dato ai sistemi *𝜀*, *𝛽* e *𝛾*. Poi dopo alcuni istanti, ma comunque in tempi radidi,
il comando `add_qspn_arc` viene dato ai sistemi *𝛽* (per 2 archi-qspn), *𝛿* e *𝜆*.  
Ciò premesso, mostreremo le relative annotazioni solo per i comandi `prepare_migrate_phase_1`
e `migrate_phase_1` nel sistema *𝜀*.

#### Comando prepare_migrate_phase_1 al sistema *𝜀*

Analogo a quanto visto nel caso 1.

#### Comando migrate_phase_1 al sistema *𝜀*

Creazione del nuovo network namespace: analogo a quanto visto nel caso 1.

#### Migrazione: Spostamento delle rotte della vecchia identità nel sistema *𝜀*

Il programma **qspnclient** decide quali tabelle di inoltro vanno usate nel nuovo network namespace.
In questo caso abbiamo l'arco-identità *𝜀<sub>1</sub>*-*𝛽<sub>1</sub>* che era associato ad un arco-qspn
interno al g-nodo migrante *𝜑*, il quale vede modificarsi anche le sue proprietà (cioè il peer-MAC).
Poi abbiamo l'arco-identità *𝜀<sub>1</sub>*-*𝛽<sub>2</sub>* che
era associato ad un arco-qspn (che era l'arco verso l'altro nodo in *𝜔*) e che mantiene inalterate le
sue proprietà. Abbiamo anche l'arco-identità *𝜀<sub>1</sub>*-*𝜆<sub>0</sub>* ma questo non era associato
ad un arco-qspn. Abbiamo infine l'arco-identità *𝜀<sub>1</sub>*-*𝜆<sub>1</sub>* che era associato
ad un arco-qspn (prodotto dal comando `add_qspn_arc` di poco fa) e che mantiene inalterate le
sue proprietà.

Il comportamento è analogo a quanto visto nel caso 1, a parte il fatto che esiste un arco-qspn interno
al g-nodo migrante, il quale vede modificarsi il peer-MAC, quindi abbiamo l'aggiunta di una tabella
di inoltro.

**sistema 𝜀**
```
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1

ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:06:3E:90 -j MARK --set-mark 248
ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:06:3E:90

(echo; echo "247 ntk_from_00:16:3E:BD:34:98 # xxx_table_ntk_from_00:16:3E:BD:34:98_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip netns exec migr02 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:BD:34:98 -j MARK --set-mark 247
ip netns exec migr02 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.42.4 dev migr02_eth1
ip netns exec migr02 ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.96.141 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip rule add fwmark 247 table ntk_from_00:16:3E:BD:34:98
```

#### Migrazione: Aggiornamento dei gateway che si sono spostati in un diverso namespace

Analogo a quanto visto nel caso 1.

#### Migrazione: Pulizia del vecchio namespace per la nuova identità nel sistema *𝜀*

Analogo a quanto visto nel caso 1.

#### Migrazione: Cambio di indirizzo della vecchia identità nel sistema *𝜀*

Analogo a quanto visto nel caso 1.

#### Migrazione: Popolamento nuove rotte della nuova identità nel sistema *𝜀*

Il comportamento è analogo a quanto visto nel caso 1, a parte il fatto che, siccome il livello
del g-nodo che entra in blocco è 1, nella fase precedente non erano stati rimossi gli indirizzi IP interni
ai g-nodi di livello 1 o minore. Perciò quelli non vanno aggiunti alle tabelle pre-esistenti nel
vecchio network namespace.

#### Comando add_qspn_arc ai vari sistemi

Analogo a quanto visto nel caso 1.

#### Migrazione: Cambio di indirizzo di una identità nel sistema *𝜀*

Il comportamento è analogo a quanto visto nel caso 1, a parte il fatto che in questa occasione si
rende chiaro che quando il livello a cui l'identificativo diventa *reale* è maggiore di 0
abbiamo nuovi indirizzi IP validi, che prima non lo erano, e quindi vanno aggiunti.

**sistema 𝜀**
```
ip route del 10.0.0.16/31 table ntk
ip route del 10.0.0.80/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90

ip route add unreachable 10.0.0.16/32 table ntk
ip route add unreachable 10.0.0.80/32 table ntk
ip route add unreachable 10.0.0.56/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route change 10.0.0.16/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.80/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.56/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.48/32 table ntk via 169.254.27.218 dev eth1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.16/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.80/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.56/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EE:AF:D1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:06:3E:90

ip address add 10.0.0.49 dev eth1
ip address add 10.0.0.57 dev eth1
ip address add 10.0.0.17 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.17
ip address add 10.0.0.81 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.84/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.60/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.57
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.17
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.17
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.57
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.49
ip route change 10.0.0.16/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.80/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.56/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.57
ip route change 10.0.0.48/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41
```

#### Migrazione: Rimozione archi esterni nel sistema *𝜀*

Analogo a quanto visto nel caso 1.

#### Migrazione: Controllo di persistenza dell'identità di connettività

Analogo a quanto visto nel caso 1.

#### Comando enter_net_phase_2 al sistema *𝜆*

Analogo a quanto visto nel caso 1, con alcune accortezze.

Visto che il sistema è gateway ad una sottorete autonoma di dimensione di un g-nodo di livello 1, 
poiché si aggiungono nel network namespace default i propri indirizzi IP, si impostano anche le relative regole di rimappatura.

**sistema 𝜆**
```
ip route del 10.0.0.22/31 table ntk
ip route del 10.0.0.86/31 table ntk
ip route del 10.0.0.62/31 table ntk
ip route del 10.0.0.50/31 table ntk
ip route del 10.0.0.22/31 table ntk_from_00:16:3E:3B:9F:45
ip route del 10.0.0.86/31 table ntk_from_00:16:3E:3B:9F:45
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:3B:9F:45
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:3B:9F:45
ip route del 10.0.0.22/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.86/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:3C:14:33
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33

ip address add 10.0.0.50 dev eth1
ip address add 10.0.0.62 dev eth1
ip address add 10.0.0.22 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.22
ip address add 10.0.0.86 dev eth1

# il nuovo indirizzo è valido almeno dentro il g-nodo di livello 2
iptables -t nat -A PREROUTING -d 10.0.0.50/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.48/30 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.50/31

# il nuovo indirizzo è valido almeno dentro il g-nodo di livello 3
iptables -t nat -A PREROUTING -d 10.0.0.62/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.56/29 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.62/31

# il nuovo indirizzo è valido in tutta la rete
iptables -t nat -A PREROUTING -d 10.0.0.22/31 -j NETMAP --to 10.0.0.40/31
iptables -t nat -A POSTROUTING -d 10.0.0.0/27  -s 10.0.0.40/31 -j NETMAP --to 10.0.0.22/31
# Se vogliamo che ogni sistema nella sottorete autonoma accetti di essere contattato in forma anonima:
iptables -t nat -A PREROUTING -d 10.0.0.86/31 -j NETMAP --to 10.0.0.40/31
# Sicuramente il gateway permette ai sistemi interni di contattare un sistema esterno in forma anonima.
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -s 10.0.0.40/31 -j NETMAP --to 10.0.0.22/31

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.22
ip route change 10.0.0.80/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.22
ip route change 10.0.0.56/30 table ntk via 169.254.163.36 dev eth1 src 10.0.0.62
ip route change 10.0.0.20/31 table ntk via 169.254.241.153 dev eth1 src 10.0.0.22
ip route change 10.0.0.84/31 table ntk via 169.254.241.153 dev eth1 src 10.0.0.22
ip route change 10.0.0.60/31 table ntk via 169.254.241.153 dev eth1 src 10.0.0.62
ip route change 10.0.0.48/31 table ntk via 169.254.241.153 dev eth1 src 10.0.0.50
```

[Operazione seguente](DettagliOperazioni13.md)
