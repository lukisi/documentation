# Proof of concept - Casi d'uso - Pagina 19

[Pagina precedente](UseCases18.md)

Ora il singolo nodo **𝛽<sub>migr01</sub>** si domanda se la rimozione dell'indirizzo 0·0·3 che è
*di connettività* ai livelli da 1 a 1, provoca lo split del g-nodo 0·0·. Si veda la prima
[migrazione](UseCases12.md#Migrazione_beta) di *𝛽* e poi la successiva
[migrazione](UseCases15.md#Migrazione_phi) di tutto il g-nodo *𝜑*, in particolare l'indirizzo
[definitivo](UseCases16.md#Cambio_indirizzo_phi1) che prende *𝜑<sub>1</sub>* di cui fa parte
l'identità di *𝛽* che ora gestisce il network namespace migr01.

Applichiamo la regola generale esposta nel documento di
[analisi](../ModuloQspn/AnalisiFunzionale.md#ImplementazioneVerificaRimozione)
con *i* = 0 e *j* = 1, dove *i* è il livello del g-nodo che si dovrebbe rimuovere, il quale è *di connettività* ai
livelli da *i* + 1 a *j*.

Per prima cosa si cerca il livello più piccolo *k*, partendo da 1, in cui il g-nodo
*g<sub>k-1</sub>(𝛽<sub>migr01</sub>)* ha almeno un vicino in *g<sub>k</sub>(𝛽<sub>migr01</sub>)*.
Nel nostro caso è 1, in quanto 0·0·3 ha i vicini 0·0·1 e 0·0·0 in 0·0·.

Questo è l'insieme dei g-nodi di livello da 0 a 0 che **𝛽<sub>migr01</sub>** vede nella sua
mappa: {0·0·0, 0·0·1}.

Questo è l'insieme dei g-nodi di livello 0 che **𝛽<sub>migr01</sub>** vede nella sua mappa come
diretti vicini di 0·0·3: {0·0·0, 0·0·1}.

Per ogni elemento *x* del primo set, per ogni elemento *y* del secondo set, se *x* ≠ *y*, il nodo **𝛽<sub>migr01</sub>** è
a conoscenza di un percorso per *x* che passa per *y*. Quindi **𝛽<sub>migr01</sub>** sa che è possibile rimuovere
l'indirizzo *di connettività* 0·0·3.

Quando un singolo nodo (quello che ha questo compito) scopre che un indirizzo *di connettività* può essere
rimosso ogni singolo nodo che appartiene a quel g-nodo rimuove la sua relativa identità *di connettività*.

Inoltre, quando viene rimossa una identità *di connettività* che era nata come copia di un'altra identità
*di connettività*, anche questa va rimossa immediatamente.

Nel nostro caso l'identità del sistema *𝛽* che ora si trova nel network namespace *migr01* è nata, durante
la migrazione di *𝜑*, come copia dell'identità del sistema *𝛽* che ora si trova nel
network namespace *migr02*. Va quindi rimossa anche quella.

----

Per dismettere l'identità *𝛽<sub>migr01</sub>* il programma *qspnclient* nel sistema *𝛽* usa il modulo Identities.
Vengono coinvolti dal modulo anche i sistemi vicini. Le operazioni svolte nei vari sistemi sono:

**sistema 𝛽**
```
ip netns exec migr01 ip route flush table main
ip netns exec migr01 ip link delete migr01_eth1 type macvlan
ip netns del migr01
```

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.16
ip route change 10.0.0.84/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.16
ip route change 10.0.0.60/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.56
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.16
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.16
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.56
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.48
ip route change 10.0.0.17/32 table ntk via 169.254.163.36 dev eth1 src 10.0.0.16
ip route change 10.0.0.81/32 table ntk via 169.254.163.36 dev eth1 src 10.0.0.16
ip route change 10.0.0.57/32 table ntk via 169.254.163.36 dev eth1 src 10.0.0.56
ip route change 10.0.0.49/32 table ntk via 169.254.163.36 dev eth1 src 10.0.0.48
ip route change 10.0.0.41/32 table ntk via 169.254.163.36 dev eth1 src 10.0.0.40

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.253.216 dev eth1
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.17/32 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.81/32 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.57/32 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.49/32 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.17/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.163.36 dev eth1
ip route change 10.0.0.81/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.163.36 dev eth1
ip route change 10.0.0.57/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.163.36 dev eth1
ip route change 10.0.0.49/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.163.36 dev eth1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.17/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.163.36 dev eth1
ip route change 10.0.0.81/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.163.36 dev eth1
ip route change 10.0.0.57/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.163.36 dev eth1
ip route change blackhole 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45

ip rule del fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
ip route flush table ntk_from_00:16:3E:EE:AF:D1
sed -i '/xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx/d' /etc/iproute2/rt_tables
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
```

**sistema 𝜀**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.109.22 dev eth1 src 10.0.0.17
ip route change 10.0.0.84/30 table ntk via 169.254.109.22 dev eth1 src 10.0.0.17
ip route change 10.0.0.60/30 table ntk via 169.254.109.22 dev eth1 src 10.0.0.57
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.17
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.17
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.57
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.49
ip route change 10.0.0.16/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.17
ip route change 10.0.0.80/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.17
ip route change 10.0.0.56/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.57
ip route change 10.0.0.48/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.109.22 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.109.22 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.109.22 dev eth1
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.16/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.109.22 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.109.22 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.109.22 dev eth1
ip route change unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.16/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.94.223 dev eth1
ip route change 10.0.0.80/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.94.223 dev eth1
ip route change 10.0.0.56/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.94.223 dev eth1
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.94.223 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:06:3E:90
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:06:3E:90
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:06:3E:90
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:06:3E:90 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:06:3E:90 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:06:3E:90 via 169.254.96.141 dev eth1
ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:06:3E:90
ip route change 10.0.0.16/32 table ntk_from_00:16:3E:06:3E:90 via 169.254.94.223 dev eth1
ip route change 10.0.0.80/32 table ntk_from_00:16:3E:06:3E:90 via 169.254.94.223 dev eth1
ip route change 10.0.0.56/32 table ntk_from_00:16:3E:06:3E:90 via 169.254.94.223 dev eth1
ip route change blackhole 10.0.0.48/32 table ntk_from_00:16:3E:06:3E:90
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:06:3E:90

ip rule del fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
ip route flush table ntk_from_00:16:3E:EE:AF:D1
sed -i '/xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx/d' /etc/iproute2/rt_tables
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
```

Ricordiamo che, a fronte della rimozione di un network namespace, il programma deve tenere traccia se qualche
particolare tabella `ntk_from_XXX` presente ha cessato di essere referenziata. In quel caso va
rimossa. Nel caso di *𝛽<sub>migr01</sub>* nessuna.

----

Per dismettere l'identità *𝛽<sub>migr02</sub>* il programma *qspnclient* nel sistema *𝛽* usa il modulo Identities.
Vengono coinvolti dal modulo anche i sistemi vicini. Le operazioni svolte nei vari sistemi sono:

**sistema 𝛽**
```
ip netns exec migr02 ip route flush table main
ip netns exec migr02 ip link delete migr02_eth1 type macvlan
ip netns del migr02
```

**sistema 𝛾**
```
ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.22/31 table ntk via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.86/31 table ntk via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.62/31 table ntk via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.50/31 table ntk via 169.254.241.153 dev migr02_eth1

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45 via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.241.153 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.241.153 dev migr02_eth1

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:3B:9F:45

ip netns exec migr02 ip rule del fwmark 247 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route flush table ntk_from_00:16:3E:BD:34:98
sed -i '/xxx_table_ntk_from_00:16:3E:BD:34:98_xxx/d' /etc/iproute2/rt_tables
ip netns exec migr02 iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:BD:34:98 -j MARK --set-mark 247
```

**sistema 𝜀**
```
ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.22/31 table ntk via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.86/31 table ntk via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.62/31 table ntk via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.50/31 table ntk via 169.254.109.22 dev migr02_eth1

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:06:3E:90 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:06:3E:90 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:06:3E:90 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:06:3E:90 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:06:3E:90 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:06:3E:90 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:06:3E:90

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change 10.0.0.22/31 table ntk_from_00:16:3E:AF:4C:2A via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.86/31 table ntk_from_00:16:3E:AF:4C:2A via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.62/31 table ntk_from_00:16:3E:AF:4C:2A via 169.254.109.22 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.50/31 table ntk_from_00:16:3E:AF:4C:2A via 169.254.109.22 dev migr02_eth1

ip netns exec migr02 ip rule del fwmark 247 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route flush table ntk_from_00:16:3E:BD:34:98
sed -i '/xxx_table_ntk_from_00:16:3E:BD:34:98_xxx/d' /etc/iproute2/rt_tables
ip netns exec migr02 iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:BD:34:98 -j MARK --set-mark 247
```

Ricordiamo che, a fronte della rimozione di un network namespace, il programma deve tenere traccia se qualche
particolare tabella `ntk_from_XXX` presente ha cessato di essere referenziata. In quel caso va
rimossa. Nel caso di *𝛽<sub>migr02</sub>* ce ne sono due.

**sistema 𝛽**
```
sed -i '/xxx_table_ntk_from_00:16:3E:AF:4C:2A_xxx/d' /etc/iproute2/rt_tables
sed -i '/xxx_table_ntk_from_00:16:3E:3B:9F:45_xxx/d' /etc/iproute2/rt_tables
```

----

L'intero sistema e ogni singolo g-nodo sono ancora internamente connessi e ogni singolo nodo è
raggiungibile. La nuova rete si presenta come nel disegno seguente.

##### <a name="grafo_5"></a>Grafo 5

![missing image](img/grafo5.png "grafo 5")

[Pagina seguente](UseCases20.md)
