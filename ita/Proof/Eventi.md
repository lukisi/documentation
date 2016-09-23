# Proof of concept - Eventi - Pagina 1

Nel presente documento associamo ad ogni operazione l'evento che nel programma scatena la
sua esecuzione.

## Sequenza iniziale

Prendiamo ad esempio i valori del nodo *𝛼*. Questo all'avvio ha indirizzo Netsukuku `3·0·1·0`.
Il modulo Neighborhood sceglie per l'interfaccia `eth1` l'indirizzo di scheda `169.254.69.30`.

### Preparazione sistema

**sistema 𝛼**
```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip address add 10.0.0.32 dev lo
```

Questa sequenza di operazioni avviene all'avvio del programma **qspnclient**.

L'indirizzo `10.0.0.32` usato come sinonimo di `localhost` viene calcolato in base alla topologia
di rete, la quale viene passata dall'utente come argomento del comando di avvio.

### Preparazione interfacce di rete

**sistema 𝛼**
```
sysctl net.ipv4.conf.eth1.rp_filter=0
sysctl net.ipv4.conf.eth1.arp_ignore=1
sysctl net.ipv4.conf.eth1.arp_announce=2
```

Questa sequenza di operazioni avviene all'avvio del programma **qspnclient**. Viene fatta per ogni
interfaccia di rete (nell'esempio `eth1`) le quali sono passate dall'utente come argomento del comando di avvio.

### Assegnazione indirizzo di scheda

**sistema 𝛼**
```
ip link set dev eth1 up
ip address add 169.254.69.30 dev eth1
```

Questa sequenza di operazioni è eseguita dal modulo Neighborhood, il quale viene istruito all'avvio
del programma **qspnclient** su quali interfacce di rete deve gestire.

### Creazione della tabella di routing `ntk`

**sistema 𝛼**
```
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
```

Questa sequenza di operazioni avviene all'avvio del programma **qspnclient**.

Il programma **qspnclient** tiene traccia delle tabelle che ha creato nel file `/etc/iproute2/rt_tables`.
Ogni volta che deve aggiungere una tabella gli assegna il primo numero identificativo libero
partendo da 251 e scendendo verso il basso. Ogni volta che deve rimuovere una tabella tiene traccia
del numero che è stato liberato.

Il numero identificativo della tabella `ntk` è sempre 251 poiché è la prima tabella ad essere
codificata nel file `/etc/iproute2/rt_tables`.

### Assegnazione indirizzi IP del primo indirizzo Netsukuku scelto

**sistema 𝛼**
```
ip address add 10.0.0.26 dev eth1
ip address add 10.0.0.90 dev eth1
ip address add 10.0.0.58 dev eth1
ip address add 10.0.0.50 dev eth1
ip address add 10.0.0.40 dev eth1
```

Questa sequenza di operazioni avviene all'avvio del programma **qspnclient**. Viene fatta per ogni
interfaccia di rete gestita (nell'esempio `eth1`).

Sia *n* l'indirizzo Netsukuku scelto per il nodo. Questa informazione è passata
dall'utente come argomento del comando di avvio.

Il primo indirizzo IP assegnato è quello globale di *n*. Viene assegnato sempre.

Il secondo indirizzo IP assegnato è quello anonimizzante di *n*. Viene assegnato opzionalmente, cioè solo
se il sistema ammette di essere contattato in forma anonima. Questa informazione è passata
dall'utente come argomento del comando di avvio.

I successivi indirizzi IP sono quelli interni di *n*. Vengono calcolati partendo da quello interno al
livello *l* - 1 (dove *l* è il numero di livelli della topologia, nel nostro esempio 4) e
scendendo fino al livello 1.

### Popolamento iniziale della tabella di routing `ntk`

**sistema 𝛼**
```
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.28/30 table ntk
ip route add unreachable 10.0.0.92/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.24/31 table ntk
ip route add unreachable 10.0.0.88/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.27/32 table ntk
ip route add unreachable 10.0.0.91/32 table ntk
ip route add unreachable 10.0.0.59/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
```

Questa sequenza di operazioni avviene all'avvio del programma **qspnclient**.

La tabella `ntk` viene popolata fin da subito con tutte le possibili destinazioni, inizialmente segnate
come `unreachable`. I possibili indirizzi IP di destinazione, ognuno con suffisso CIDR, sono calcolati in questo modo:

*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Per *i* che scende da *l* - 1 a 0, per *j* da 0 a *gsize(i)*, se *pos_n(i)* ≠ *j*:
    *   Calcola indirizzo IP globale di (*i*, *j*) rispetto a *n*.
    *   Calcola indirizzo IP anonimizzante di (*i*, *j*) rispetto a *n*.
    *   Per *k* che scende da *l* - 1 a *i* + 1:
        *   Calcola indirizzo IP interno al livello *k* di (*i*, *j*) rispetto a *n*.

### Mascheramento del source nei pacchetti *anonimi*

**sistema 𝛼**
```
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.26
```

Questa sequenza di operazioni avviene, opzionalmente, all'avvio del programma **qspnclient**. Serve ad istruire
il kernel di mascherare il source dei pacchetti IP che vengono inoltrati verso indirizzi IP anonimizzanti; per questo
è opzionale. L'informazione della disponibilità del sistema ad anonimizzare tali pacchetti IP è passata
dall'utente come argomento del comando di avvio.

Il criterio di selezione sulla destinazione, nell'esempio `10.0.0.64/27`, viene calcolato in base alla topologia
di rete. L'indirizzo IP che viene rimpiazzato, nell'esempio `10.0.0.26`, è quello globale del sistema.

### Primo aggiornamento di tutte le rotte conosciute

**sistema 𝛼**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.28/30 table ntk
ip route change unreachable 10.0.0.92/30 table ntk
ip route change unreachable 10.0.0.60/30 table ntk
ip route change unreachable 10.0.0.24/31 table ntk
ip route change unreachable 10.0.0.88/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.27/32 table ntk
ip route change unreachable 10.0.0.91/32 table ntk
ip route change unreachable 10.0.0.59/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
```

Questa sequenza di operazioni avviene quando il programma **qspnclient** riceve il segnale
`bootstrap_complete` dal QspnManager. Certamente, nella rete appena creata dal nodo che si sta
avviando, nessuna destinazione è raggiungibile.

[Pagina seguente](Eventi2.md)
