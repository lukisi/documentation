# Proof of concept - Eventi - Pagina 4

[Pagina precedente](Eventi3.md)

Le sequenze di operazioni presentate in questa pagina sono eseguite dal programma **qspnclient** quando
l'utente comanda di completare le operazioni di migrazione/ingresso prima indicate, in seguito alle
operazioni viste prima.

### Popolamento nuove rotte della nuova identità

**sistema 𝛿**
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
ip route add unreachable 10.0.0.22/31 table ntk
ip route add unreachable 10.0.0.86/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
```

**sistema 𝜇**
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
ip route add unreachable 10.0.0.22/31 table ntk
ip route add unreachable 10.0.0.86/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
```

Nella terza parte di operazioni la nuova identità popola
il suo network namespace (quello preesistente ereditato dalla vecchia identità) con le
rotte che sono consone al suo nuovo indirizzo.

In questo esempio, sia nel sistema *𝛿* che in *𝜇*, abbiamo come vecchio namespace il
default e un indirizzo che ha una sola componente *virtuale* al livello 1. L'indirizzo
IP interno al livello 1 per la destinazione valida come g-nodo di livello 0 (cioè
10.0.0.40/32 per *𝛿* e 10.0.0.41/32 per *𝜇*) è già presente nelle tabelle del namespace vecchio.

### Creazione e popolamento iniziale di tabelle per l'inoltro

**sistema 𝛿**
```
(echo; echo "248 ntk_from_00:16:3E:5B:78:D5 # xxx_table_ntk_from_00:16:3E:5B:78:D5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 248
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
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
```

**sistema 𝛾**
```
(echo; echo "249 ntk_from_00:16:3E:1A:C4:45 # xxx_table_ntk_from_00:16:3E:1A:C4:45_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:1A:C4:45 -j MARK --set-mark 249
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
```

Nella quarta parte delle operazioni, il **qspnmanager** del sistema *𝛿*, siccome crea il QspnManager
della nuova identità indicando l'arco-identità con il sistema *𝛾*, crea e popola la relativa tabella
di inoltro, pur senza aggiungere la regola. Analoga cosa fa il **qspnmanager** del sistema *𝛾*,
siccome aggiunge al suo QspnManager l'arco-identità con il sistema *𝛿*.

### Cambio di indirizzo della nuova identità

**sistema 𝛿**
```
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/32 table ntk
ip route add unreachable 10.0.0.84/32 table ntk
ip route add unreachable 10.0.0.60/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5

ip address add 10.0.0.49 dev eth1
ip address add 10.0.0.61 dev eth1
ip address add 10.0.0.21 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.21
ip address add 10.0.0.85 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.22/31 table ntk
ip route change unreachable 10.0.0.86/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41
```

**sistema 𝜇**
```
ip route del 10.0.0.20/31 table ntk
ip route del 10.0.0.84/31 table ntk
ip route del 10.0.0.60/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.21/32 table ntk
ip route add unreachable 10.0.0.85/32 table ntk
ip route add unreachable 10.0.0.61/32 table ntk
ip route add unreachable 10.0.0.49/32 table ntk
ip route add unreachable 10.0.0.21/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.85/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.61/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45

ip address add 10.0.0.48 dev eth1
ip address add 10.0.0.60 dev eth1
ip address add 10.0.0.20 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.20
ip address add 10.0.0.84 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.22/31 table ntk
ip route change unreachable 10.0.0.86/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change 10.0.0.21/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.85/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.20
ip route change 10.0.0.61/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.60
ip route change 10.0.0.49/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.48
ip route change 10.0.0.41/32 table ntk via 169.254.253.216 dev eth1 src 10.0.0.40
```

Nell'indicare le operazioni di migrazione/ingresso, l'utente aveva specificato
che la migration path più breve è di zero passi. Cioè aveva indicato che esiste fin da subito un
indirizzo Netsukuku *reale* libero compatibile con gli archi del g-nodo. Esso è `2·1·0·`.  
Quindi il programma **qspnclient**, dopo aver fatto passare la nuova identità per l'indirizzo
*virtuale* al livello 1 `2·1·2·X`, immediatamente (o pochi istanti dopo) cambia il suo identificativo di livello 1
da `2` a `0`. Questa passaggio induce il programma **qspnclient** a eseguire la quinta parte delle operazioni.

Nella quinta parte delle operazioni il programma **qspnclient** rimuove, nel network namespace della
nuova identità cioè il default, dalla tabella `ntk` e da
tutte le tabelle di inoltro (anche quelle il cui arco non ha ancora ricevuto alcun ETP) le
rotte verso gli indirizzi che non sono più validi a causa di questo nuovo identificativo *reale*.  
In seguito, siccome il livello in cui l'identificativo passa a *reale* è maggiore di 0, in questo
caso 1, aggiunge nelle stesse tabelle le rotte verso altri indirizzi che ora può computare. Si tratta
degli indirizzi IP interni ai livelli superiori per le destinazioni di g-nodi di livello inferiore
rispetto al g-nodo che migra. (Come tali, non sarebbe dannoso lasciarli come *irraggiungibili* fino alla
processazione del primo ETP; ma non siamo sicuri che questa avvenga dopo l'esecuzione di queste operazioni.)  
Poi, siccome la nuova identità è la *principale*, il programma **qspnclient** fa queste operazioni:

*   Aggiunge ad ogni interfaccia di rete reale nel network
    namespace default gli indirizzi IP propri che prima non era possibile computare e adesso invece sì,
    partendo dal livello più basso e salendo finché possibile.
*   Poi, siccome l'indirizzo è del tutto *reale*, aggiunge (opzionalmente) la regola di source-natting
    e (opzionalmente) l'indirizzo IP anonimizzante.
*   Infine deve aggiornare tutte le rotte nella tabella `ntk` per fare in modo di mettere in esse
    (se disponibile) un *src* preferito. Ma poiché potremmo aver aggiunto rotte nelle altre tabelle,
    per sicurezza aggiorniamo tutte le rotte in tutte le tabelle.

[Pagina seguente](Eventi5.md)
