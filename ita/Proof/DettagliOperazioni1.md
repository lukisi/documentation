# Proof of concept - Dettagli Operazioni - Pagina 1

### <a name="Avvio_programma"></a> Avvio del programma

All'avvio del programma **qspnclient** l'utente specifica alcune informazioni persistenti:

*   Topologia della rete.
*   Se il sistema vuole fare da gateway per una sottorete a gestione autonoma, tale sottorete può
    avere la dimensione di un g-nodo di livello *i* nella topologia di rete specificata. L'utente
    specifica il livello di tale g-nodo. Indichiamolo in questa trattazione con la variabile `$subnetlevel`. Default 0.
*   Lista di interfacce di rete da gestire.
*   Se il sistema ammette di essere usato come anonimizzatore.
*   Se il sistema (e tutta la eventuale sottorete autonoma dietro di lui) ammette di essere contattato
    in forma anonima.

Inoltre l'utente specifica alcune informazioni temporanee:

*   Indirizzo Netsukuku **iniziale** del sistema.

#### Parte 1

All'avvio il programma computa l'indirizzo IP sinonimo di localhost per la topologia di rete
specificata. Il modo di calcolare questo indirizzo è spiegato [qui](AnalisiFunzione.md#Mappatura_indirizzi_ip).
Sia `$ntklocalhost` questo indirizzo IP, ad esempio `10.0.0.32`.

Il programma **qspnclient** esegue queste operazioni:

```
sysctl net.ipv4.ip_forward=1
sysctl net.ipv4.conf.all.rp_filter=0
ip address add $ntklocalhost dev lo
```

#### Parte 2

Sia `$devlist` la lista di interfacce di rete da gestire, ad esempio `eth1 wlan0`.

Il programma **qspnclient** esegue queste operazioni:

```
for dev in $devlist; do
 sysctl net.ipv4.conf.$dev.rp_filter=0
 sysctl net.ipv4.conf.$dev.arp_ignore=1
 sysctl net.ipv4.conf.$dev.arp_announce=2
done
```

#### Parte 3

Il programma **qspnclient** informa il modulo Neighborhood di ogni interfaccia di rete che
deve gestire, tramite il metodo `start_monitor`. Il modulo Neighborhood produrrà per questo alcuni comandi al sistema operativo.

Il programma si avvede dell'indirizzo scelto perché il NeighborhoodManager lo notifica con il segnale
*nic_address_set*. Il programma associa questo proprio indirizzo link-local all'indice
autoincrementante *linklocal_nextindex*, che parte da 0.

#### Parte 4

Il programma **qspnclient** tiene traccia delle tabelle che ha creato nel file `/etc/iproute2/rt_tables`.
Ogni volta che deve aggiungere una tabella gli assegna il primo numero identificativo libero
partendo da 251 e scendendo verso il basso. Ogni volta che deve rimuovere una tabella tiene traccia
del numero che è stato liberato.

Sia `$tbid_free` il primo identificativo libero, che all'avvio del programma sarà `251`.

Il programma **qspnclient** aggiunge la tabella `ntk` e imposta la relativa regola di
default (senza fwmark) sul namespace default. Cioè, esegue queste operazioni:

```
(echo; echo $tbid_free "ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
```

#### Parte 5

Il programma **qspnclient** computa i propri indirizzi IP, equivalenti all'indirizzo Netsukuku che inizialmente
il sistema si assegna.

Il primo indirizzo IP assegnato è quello globale. Viene assegnato sempre. Sia esso `$globalip`.

Il secondo indirizzo IP assegnato è quello anonimizzante. Viene assegnato opzionalmente, cioè solo
se il sistema ammette di essere contattato in forma anonima. Sia esso `$anonymousip`.

I successivi indirizzi IP sono quelli interni. Vengono calcolati partendo da quello interno al
livello *l* - 1 (dove *l* è il numero di livelli della topologia) e scendendo fino al livello `$subnetlevel` + 1.

```
for dev in $devlist; do
 ip address add $globalip dev $dev
done

if $allow_anonymous; then
 for dev in $devlist; do
  ip address add $anonymousip dev $dev
 done
fi

for i = l - 1 to $subnetlevel + 1 step -1
 internalip = indirizzo IP interno al g-nodo di livello $i
 for dev in $devlist; do
  ip address add $internalip dev $dev
 done
next
```

#### Parte 6

Il programma **qspnclient** computa gli indirizzi IP di tutte le possibili destinazioni relative
all'indirizzo Netsukuku che inizialmente va assegnato al sistema. Per ognuno assegna una rotta
nella tabella `ntk` come `unreachable`.

I possibili indirizzi IP di destinazione, ognuno con suffisso CIDR, sono calcolati in questo modo:

*   Indichiamo con *l* il numero di livelli nella topologia.
*   Indichiamo con *n* l'indirizzo Netsukuku del sistema.
*   Indichiamo con *pos_n(i)* l'identificativo al livello *i* dell'indirizzo Netsukuku *n*.
*   Per *i* che scende da *l* - 1 a `$subnetlevel`, per *j* da 0 a *gsize(i)* - 1, se *pos_n(i)* ≠ *j*:
    *   Calcola, se possibile, indirizzo IP globale di (*i*, *j*) rispetto a *n*.  
        È possibile se tutte le posizioni di *n* sono *reali* dal livello *i* + 1 al livello *l* - 1.
    *   Calcola, se possibile, indirizzo IP anonimizzante di (*i*, *j*) rispetto a *n*.
    *   Per *k* che scende da *l* - 1 a *i* + 1:
        *   Calcola, se possibile, indirizzo IP interno al livello *k* di (*i*, *j*) rispetto a *n*.  
            È possibile se tutte le posizioni di *n* sono *reali* dal livello *i* + 1 al livello *k* - 1.

Per ogni indirizzo IP calcolato in questo ciclo, sia esso `$ipaddr` (ad esempio `10.0.0.0/29`),
il programma **qspnclient** esegue queste operazioni:

```
ip route add unreachable $ipaddr table ntk
```

Questo blocco di comandi va eseguito senza intromissione di altri comandi da altre tasklet.

#### Parte 7

Il programma **qspnclient**, se il sistema ammette di essere usato come anonimizzatore, esegue questi
comandi per istruire il kernel di mascherare il source dei pacchetti IP che vengono inoltrati verso
indirizzi IP anonimizzanti.

Sia `$anonymousrange` il range di indirizzi IP anonimizzanti, calcolato dal programma sulla base
della topologia, ad esempio `10.0.0.64/27`. L'indirizzo IP che viene rimpiazzato è quello globale del sistema,
che avevamo già computato, `$globalip`.

```
iptables -t nat -A POSTROUTING -d $anonymousrange -j SNAT --to $globalip
```

[Operazione seguente](DettagliOperazioni2.md)
