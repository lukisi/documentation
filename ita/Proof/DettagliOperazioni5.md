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

Anche dalla prospettiva del nodo *𝛽* avremo subito un ETP da processare. Non ripetiamo le operazioni che il
programma **qspnclient** deve fare a seguito della completata processazione di un ETP, sono le stesse viste
in precedenza nel nodo *𝛼*.

### <a name="Rimosso_vicino_stessa_rete"></a> Un arco con un vicino nella stessa rete viene rimosso

#### Rimozione di un arco fisico non più funzionante

Supponiamo che il programma **qspnclient** riceve dall'utente istruzioni di rimuovere un arco fisico
verso un diretto vicino. Sarebbe la stessa cosa anche se il programma riceve il segnale `arc_removing`
dal modulo Neighborhood che significa che un arco fisico non è più funzionante.

**TODO: verificare nel modulo Identities** Il programma istruisce il modulo Identities che questo arco fisico sta per essere rimosso:
cioè gli dice che sta per essere rimosso dalla tabella `main` del kernel nel network namespace `default` la rotta diretta *principale*,
cioè quella che collega l'identità principale di questo sistema all'identità principale del sistema vicino.

Il modulo Identities per ogni arco-identità che fa riferimento a questo arco fisico (anche per quello principale)
emette il **suo** segnale `arc_removing`. Poi, non per l'arco-identità principale ma solo per gli altri,
rimuove dalla tabella `main` del kernel nel network namespace interessato la rotta diretta. Infine, per tutti gli
archi-identità, emette il suo segnale `arc_removed`.

Quando riceve il segnale `arc_removing` dal modulo Identities, il programma **qspnclient**, poiché quel
particolare arco-identità era associato ad un arco-qspn, esegue queste azioni sulla relativa tabella di
inoltro:

*   Se la relativa tabella di inoltro era stata attivata:
    *   Rimuove la regola.
*   Svuota la tabella.
*   Rimuove la marcatura dei pacchetti.
*   Rimuove l'associazione tra questa identità e questa tabella. **TODO: spostare questa informazione:** Esiste
    quindi una associazione tra una identità e un elenco di tabelle (ad esempio una lista di identificativi
    numerici, i quali sono nel file `rt_tables`; inoltre esiste una mappa globale dell'applicazione che associa
    il numero alla stringa del MAC).
*   Se nessuna identità ha una associazione con questa tabella:
    *   Rimuove lo pseudonimo della tabella nel file `rt_tables`, liberando il numero identificativo.

```
ip rule del fwmark 250 table ntk_from_00:16:3E:EE:AF:D1
ip route flush table ntk_from_00:16:3E:EE:AF:D1
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EE:AF:D1 -j MARK --set-mark 250
sed -i '/xxx_table_ntk_from_00:16:3E:EE:AF:D1_xxx/d' /etc/iproute2/rt_tables
```

Vanno eseguite in blocco.

Poi il programma rimuove l'arco-qspn dal QspnManager dell'identità interessata.

#### Rimozione di un arco-identità su richiesta dell'identità vicina

Supponiamo invece che sia il modulo Identities del vicino a decidere di rimuovere un arco-identità
con noi. Può avvenire perché una identità di connettività nel sistema vicino rimuove i
suoi archi esterni, oppure perché viene dismessa, oppure perché l'intero sistema vicino si sta
disconnettendo dalla rete.

Il modulo Identities del sistema vicino lo comunica al modulo Identities nel nostro sistema, il
quale adesso emette il suo segnale `arc_removing`.

Allo stesso modo in cui abbiamo visto prima, il programma **qspnclient** reagisce a questo segnale
se quel particolare arco-identità era associato ad un arco-qspn.

#### Rimozione di un arco-identità per scelta del programma

Supponiamo invece che sia il programma **qspnclient** nel nostro sistema a decidere di rimuovere un
arco-identità con un vicino. Cioè quando rimuoviamo gli archi esterni di una nostra identità di connettività.

In questo caso è il programma stesso a sapere che deve eseguire le operazioni viste prima.

[Operazione seguente](DettagliOperazioni6.md)
