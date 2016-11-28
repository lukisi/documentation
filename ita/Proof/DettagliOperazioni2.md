# Proof of concept - Dettagli Operazioni - Pagina 2

[Operazione precedente](DettagliOperazioni1.md)

### <a name="Avvio_programma"></a> Avvio del programma

#### Parte 6

Il programma **qspnclient** computa gli indirizzi IP di tutte le possibili destinazioni relative
all'indirizzo Netsukuku che inizialmente va assegnato al sistema. Per ognuno assegna una rotta
nella tabella `ntk` come `unreachable`.

<a name="computo_indirizzi_ip_destinazioni"></a>
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

#### Parte 8

Il programma **qspnclient**, se il sistema vuole fare da gateway per una sottorete a gestione autonoma, deve
impostare delle regole di rimappatura degli indirizzi IP mittente e destinazione nei pacchetti IP che
transitano per esso. Infatti i nodi interni alla sottorete autonoma hanno cognizione solo degli indirizzi
IP interni al g-nodo di livello `$subnetlevel`.

In seguito nel documento illustreremo la logica di queste rimappature e come sono realizzate.

[Operazione seguente](DettagliOperazioni3.md)
