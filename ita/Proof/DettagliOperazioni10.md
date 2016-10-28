# Proof of concept - Dettagli Operazioni - Pagina 10

[Operazione precedente](DettagliOperazioni9.md)

### <a name="Migrazione_ingresso_1"></a> Migrazione per ingresso - Caso 1

#### Comando enter_net_phase_2 al sistema *𝜀*

Il programma **qspnclient** recupera le informazioni relative all'ingresso in corso che era
in attesa della migrazione *m<sub>𝛽</sub>*. Deduce quindi che è ora disponibile la posizione
*reale* che gli era stata comunicata all'inizio.

C'è da dire che in precedenza a questo evento è possibile che il nodo *𝜀<sub>1</sub>* abbia
ricevuto i primi ETP e sia uscito dalla fase di bootstrap. Potrebbe quindi aver valorizzato alcune
rotte e abilitato le tabelle di inoltro con le relative regole.

**sistema 𝜀**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.20/31 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.84/31 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.60/31 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.48/31 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.22/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.86/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.62/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.50/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.23/32 table ntk
ip route change unreachable 10.0.0.87/32 table ntk
ip route change unreachable 10.0.0.63/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.84/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.60/31 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.22/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.86/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.62/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.22/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.86/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.62/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1
ip rule add fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
```

Quando viene dato il comando `enter_net_phase_2` il programma **qspnclient** provvede a modificare
la posizione dell'indirizzo Netsukuku da *virtuale* a *reale*. Come abbiamo visto in precedenza,
oltre a comunicare tale variazione al modulo Qspn dell'identità interessata, questo cambio comporta
che il programma esegua due sequenze di operazioni: la prima perché un identificativo dell'indirizzo
di una sua identità passa da *virtuale* a *reale* e la seconda perchè questo cambio avviene nell'identità *principale*.

**sistema 𝜀**
```
ip route del 10.0.0.23/32 table ntk
ip route del 10.0.0.87/32 table ntk
ip route del 10.0.0.63/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
ip route del 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.23/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.87/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.63/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip address add 10.0.0.41 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.63 dev eth1
ip address add 10.0.0.23 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.23
ip address add 10.0.0.87 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.23
ip route change 10.0.0.80/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.23
ip route change 10.0.0.56/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.63
ip route change 10.0.0.20/31 table ntk via 169.254.27.218 dev eth1 src 10.0.0.23
ip route change 10.0.0.84/31 table ntk via 169.254.27.218 dev eth1 src 10.0.0.23
ip route change 10.0.0.60/31 table ntk via 169.254.27.218 dev eth1 src 10.0.0.63
ip route change 10.0.0.48/31 table ntk via 169.254.27.218 dev eth1 src 10.0.0.51
ip route change 10.0.0.22/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.23
ip route change 10.0.0.86/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.23
ip route change 10.0.0.62/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.63
ip route change 10.0.0.50/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41
```

[Operazione seguente](DettagliOperazioni11.md)
