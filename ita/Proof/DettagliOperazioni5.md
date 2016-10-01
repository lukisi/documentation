# Proof of concept - Dettagli Operazioni - Pagina 5

[Operazione precedente](DettagliOperazioni4.md)

### <a name="Nuovo_vicino_stessa_rete"></a> Un nuovo vicino nella stessa rete viene rilevato

Abbiamo visto l'ingresso di *𝛼* nella rete di *𝛽* dalla sua prospettiva. Dall'altra parte,
il nodo *𝛽* rileva un nuovo vicino *𝛼* che inizialmente non fa parte della sua rete. Poi viene
a sapere (tramite il modulo Identities) che una nuova identità di *𝛼* fa adesso parte della
sua rete.

In questo momento il programma **qspnclient** passa all'istanza di QspnManager della sua identità
coinvolta un nuovo arco-qspn con un altro nodo. Di tale arco-identità il programma conosce per
mezzo del modulo Identities:

*   MAC address del vicino.
*   Indirizzo IP linklocal del vicino.

Deve quindi eseguire queste operazioni in blocco:

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
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

#### Processazione di un ETP

Anche dalla prospettiva del nodo *𝛽* avremo subito un ETP da processare. Per questo evento
non ripetiamo le operazioni che il programma **qspnclient** deve fare, sono le stesse viste
in precedenza nel nodo *𝛼*.

[Operazione seguente](DettagliOperazioni6.md)
