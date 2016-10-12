# Proof of concept - Dettagli Operazioni - Pagina 4

[Operazione precedente](DettagliOperazioni3.md)

### <a name="Ingresso_rete_1"></a> Ingresso in una rete - Caso 1

Sia *𝛼* un singolo nodo che costituiva una rete a sé; esso si incontra con un
diverso singolo nodo *𝛽*; il nodo *𝛽* appartiene ad un g-nodo di livello 1 che ha una posizione
libera per *𝛼*.

La sequenza di istruzioni che l'utente darà ai singoli nodi *𝛼* e *𝛽* sarà questa:

*   Al sistema *𝛼* dà il comando `prepare_enter_net_phase_1`, indicando queste informazioni:
    *   identità entrante. L'identificativo di una identità di *𝛼*. Sia in questo esempio *𝛼<sub>0</sub>*.
    *   livello g-nodo entrante *k*. Il livello del g-nodo che fa ingresso. In questo esempio è 0.  
        L'identità *𝛼<sub>0</sub>* deve produrre una copia (sia essa *𝛼<sub>1</sub>*) che farà
        ingresso insieme al suo g-nodo di livello *k*. Chiamiamo questo g-nodo *𝜑* e il nuovo g-nodo isomorfo *𝜑'*.
        In questo esempio *𝜑* è costituito dal solo *𝛼<sub>0</sub>* e *𝜑'* è costituito dal solo *𝛼<sub>1</sub>*.
    *   g-nodo ospitante. Le informazioni riguardanti il g-nodo *𝜒*, il quale può ospitare *𝜑'* nella
        nuova rete. Consistono nell'indirizzo Netsukuku di *𝜒* e il suo fingerprint.  
        Sicuramente *𝜒* è di livello maggiore del livello di *𝜑'*, forse anche di più livelli.
    *   nuova posizione 1. Le informazioni riguardanti la posizione da assumere dentro *𝜒*. Consistono
        nella posizione e l'anzianità del g-nodo di livello direttamente inferiore a *𝜒* che potrà
        immediatamente assumere *𝜑'* nella nuova rete. Questa posizione è temporanea e *virtuale*.
    *   nuova posizione 2. Sarà in seguito resa disponibile dentro *𝜒* questa posizione *reale*.
    *   posizione di connettività. Le informazioni riguardanti la posizione di connettività nella
        vecchia rete che dovrà assumere *𝜑*. Consistono nella posizione *virtuale* e l'anzianità del g-nodo di
        livello direttamente superiore a *𝜑*.
    *   nuovi archi-qspn. Cioè quali archi-identità fra quelli di *𝛼<sub>0</sub>* (l'identità entrante)
        saranno archi nella nuova rete. Oltre a quelli che sono già noti in quanto interni al g-nodo entrante *𝜑'*.  
        Ricordiamo che ogni arco-identità riceve un identificativo che viene mostrato all'utente.
    *   l'identificativo di questa operazione di ingresso. Chiamiamolo *m<sub>𝜑</sub>*.
    *   l'identificativo dell'operazione di migrazione (eventuale) al termine della quale si potrà
        prendere la posizione *reale* di cui sopra dentro *𝜒*. Chiamiamolo *m<sub>𝜓</sub>*, ad indicare che
        per liberare la posizione ha migrato il g-nodo *𝜓*. Laddove non vi sia bisogno di alcuna
        migration path, questo identificativo è nullo.
*   Al sistema *𝛼* dà il comando `enter_net_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di ingresso *m<sub>𝜑</sub>*.
    *   se *m<sub>𝜓</sub>* era nullo: è implicita la richiesta di procedere immediatamente dopo con l'assegnazione
        dell'indirizzo *reale* dentro *𝜒*.
*   Soltanto se *m<sub>𝜓</sub>* non è nullo: al sistema *𝛼* dà il comando `enter_net_phase_2`, indicando queste informazioni:
    *   è stata completata la migrazione *m<sub>𝜓</sub>*; quindi è ora disponibile l'indirizzo *reale* dentro *𝜒*.

Risulta chiaro che in questo caso banale il tutto si sarebbe potuto fare con un solo comando dato dall'utente
nel sistema *𝛼* e che si poteva fare a meno di passare per l'indirizzo temporaneo *virtuale*. Ma per
semplicità manteniamo la sola modalità generica.

#### Comando prepare_enter_net_phase_1

Quando l'utente dà il comando `prepare_enter_net_phase_1` fornisce tutte le informazioni. Il programma
**qspnclient** a questo punto chiama il metodo `prepare_add_identity` del modulo Identities. Oltre a ciò,
non esegue nessuna operazione (comando al S.O.) ma soltanto memorizza le informazioni ricevute.

#### Comando enter_net_phase_1

Quando l'utente dà il comando `enter_net_phase_1` fornisce l'identificativo dell'operazione di
ingresso. Il programma **qspnclient** recupera le informazioni memorizzate prima. Poi chiama il
metodo `add_identity` del modulo Identities. Il modulo Identities produce quindi queste operazioni:

```
ip netns add entr02
ip netns exec entr02 sysctl net.ipv4.ip_forward=1
ip netns exec entr02 sysctl net.ipv4.conf.all.rp_filter=0
ip link add dev entr02_eth1 link eth1 type macvlan
ip link set dev entr02_eth1 netns entr02
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.rp_filter=0
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.arp_ignore=1
ip netns exec entr02 sysctl net.ipv4.conf.entr02_eth1.arp_announce=2
ip netns exec entr02 ip link set dev entr02_eth1 up
ip netns exec entr02 ip address add 169.254.215.29 dev entr02_eth1
ip netns exec entr02 ip route add 169.254.94.223 dev entr02_eth1 src 169.254.215.29
```

Nei comandi sopra riportati, il nome del network namespace (`entr02`) i nomi delle interfacce
di rete reali e pseudo (`eth1`, `entr02_eth1`) i relativi indirizzi di scheda (`169.254.215.29`)
e gli indirizzi link-local dei diretti vicini (`169.254.94.223`) sono tutti gestiti in
autonomia dal modulo Identities.

Prima di avviare il metodo `add_identity` il programma **qspnclient** conosce il nome del vecchio
network namespace, perché esso è associato all'identità vecchia.

Dall'esecuzione del metodo `add_identity` sul modulo Identities è necessario reperire:

*   identificativo della nuova identità
*   nome del nuovo network namespace (che nasce per la vecchia identità)
*   associazione tra ogni arco-identità della vecchia identità quando era nel vecchio namespace
    e il corrispettivo arco-identità della vecchia identità nel nuovo namespace (oppure basta
    che data una identità e un arco-identità passato al modulo Qspn possa recuperare le
    relative informazioni attuali?)

Il nome del vecchio network namespace viene ora associato alla nuova identità, mentre quello del
nuovo network namespace viene associato alla vecchia identità.

#### Copia tabelle e regole, spostamento rotte

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** provvede come vediamo adesso a spostare la vecchia identità
nel nuovo namespace e ripulire il vecchio namespace per la nuova identità.

Il programma **qspnclient** conosce il nome del nuovo namespace. Assumiamo sia `entr02`.

Il programma **qspnclient** conosce il nome del vecchio namespace. Assumiamo sia "" (stringa vuota), cioè il default.

Il programma **qspnclient** conosce l'indirizzo Netsukuku che la vecchia identità aveva nel vecchio
namespace. Assumiamo sia 1·0·1·0.

Il programma **qspnclient** conosce l'indirizzo Netsukuku *virtuale* che la vecchia identità come supporto
di connettività assume nel nuovo namespace. Assumiamo sia 1·0·1·3.

Richiamando le modalità viste [qui](DettagliOperazioni1.md), il programma **qspnclient** calcola tutti i possibili indirizzi IP
di destinazione, ognuno con suffisso CIDR, relativi all'indirizzo della vecchia identità nel nuovo
namespace, cioè 1·0·1·3. Li memorizza associandoli a quella identità, ma in realtà in questo caso specifico
non li usa in quanto la vecchia identità non ha nessun arco ed è destinata a sparire immediatamente.

Se la vecchia identità avesse avuto degli archi-qspn avrebbe usato nel nuovo network namespace le relative
tabelle di inoltro. In esse il programma avrebbe dovuto aggiungere (in stato `unreachable`) tutti gli
indirizzi IP di cui sopra. **TODO**: invece di dire questo, si rimandi al "caso_X" in cui si mostra come si fa.

Con le stesse modalità, il programma **qspnclient** calcola tutti i possibili indirizzi IP
di destinazione relativi all'indirizzo che la vecchia identità aveva nel vecchio
namespace, cioè 1·0·1·0. (**TODO**: in effetti erano precedentemente associati a quella identità)

Nel vecchio namespace questi indirizzi erano stati aggiunti alle tabelle presenti. Nel caso in
esame si tratta della tabella `ntk` nel namespace default. Vanno ora rimossi.

```
ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/29 table ntk
ip route del 10.0.0.88/29 table ntk
ip route del 10.0.0.12/30 table ntk
ip route del 10.0.0.76/30 table ntk
ip route del 10.0.0.60/30 table ntk
ip route del 10.0.0.8/31 table ntk
ip route del 10.0.0.72/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk
ip route del 10.0.0.11/32 table ntk
ip route del 10.0.0.75/32 table ntk
ip route del 10.0.0.59/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
```

Partendo dal livello del g-nodo entrante *k* (nel nostro caso 0) e salendo fino a
*l* - 1, solo se il vecchio namespace è il default (come nel nostro caso), il programma **qspnclient**
rimuove dal vecchio namespace gli indirizzi IP della vecchia identità che non saranno comuni
con quelli della nuova identità. In particolare, dovendo rimuovere l'indirizzo IP globale, prima
rimuove la regola (se presente) di source-natting per i pacchetti anonimi che transitano per questo
sistema. Rimuove anche (se presente) l'indirizzo IP anonimizzante.

```
ip address del 10.0.0.40/32 dev eth1
ip address del 10.0.0.50/32 dev eth1
ip address del 10.0.0.58/32 dev eth1
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.10
ip address del 10.0.0.10/32 dev eth1
ip address del 10.0.0.74/32 dev eth1
```

Prendiamo di nuovo in considerazione tutti i possibili indirizzi IP di destinazione relativi all'indirizzo
della vecchia identità nel nuovo namespace.
Se la vecchia identità avesse avuto degli archi-qspn avrebbe usato nel nuovo network namespace le relative
tabelle di inoltro. In esse, ma soltanto nelle tabelle di inoltro che hanno come nodo vicino un altro
nodo che partecipa alla migrazione/ingresso, il programma avrebbe dovuto aggiornare lo stato delle rotte su tutti gli
indirizzi IP di cui sopra. **TODO**: invece di dire questo, si rimandi al "caso_X" in cui si mostra come si fa.

#### Popolamento nuove rotte della nuova identità

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** provvede come vediamo adesso a preparare il vecchio namespace
per la nuova identità.

Il programma **qspnclient** calcola tutti i possibili indirizzi IP di destinazione relativi all'indirizzo
della nuova identità. Li memorizza associandoli a questa nuova identità.

Questi vanno ora aggiunti alle tabelle presenti nel vecchio namespace. In questo caso si tratta della
tabella `ntk` nel namespace default.

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
ip route add unreachable 10.0.0.22/32 table ntk
ip route add unreachable 10.0.0.86/32 table ntk
ip route add unreachable 10.0.0.62/32 table ntk
ip route add unreachable 10.0.0.50/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
ip route add unreachable 10.0.0.23/32 table ntk
ip route add unreachable 10.0.0.87/32 table ntk
ip route add unreachable 10.0.0.63/32 table ntk
ip route add unreachable 10.0.0.51/32 table ntk
ip route add unreachable 10.0.0.41/32 table ntk
```

#### Creazione e popolamento iniziale di tabelle per l'inoltro

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** crea una istanza di QspnManager per la sua nuova identità e gli
comunica (nel costruttore) di quali archi-qspn dispone. Nel nostro esempio ci sarà un nuovo
arco-qspn, comunicato dall'utente nel comando `prepare_enter_net_phase_1`. Di tale arco-identità il
programma conosce:

*   MAC address del vicino.
*   Indirizzo IP linklocal del vicino.

Tenendo traccia inoltre delle tabelle già inserite nel file `/etc/iproute2/rt_tables`, il programma è in grado di
eseguire le operazioni che seguono. Lo deve fare per tutti gli archi-qspn passati al QspnManager. Come sempre,
vanno eseguite in blocco senza intromissioni da altre tasklet.

```
(echo; echo "250 ntk_from_00:16:3E:5B:78:D5 # xxx_table_ntk_from_00:16:3E:5B:78:D5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250
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
ip route add unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
```

#### Processazione di un ETP

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** potrebbe dover attendere il completamento di una migration
path prima di poter cambiare l'indirizzo della nuova identità; oppure no. Questo a seconda se
l'utente nel comando `prepare_enter_net_phase_1` ha specificato una migration path oppure ha
richiesto di procedere immediatamente con l'assegnazione dell'indirizzo *reale* dentro il g-nodo destinazione.

Se deve attendere la migration path, allora è possibile che prima del cambio di indirizzo avvenga
la ricezione di un ETP e la conseguente uscita dalla fase di bootstrap della nuova identità.

In ogni caso le operazioni eseguite dal programma **qspnclient** quando un suo QspnManager ha terminato
di processare un ETP sono le stesse. Verranno delineate sotto.

#### Cambio di indirizzo di una identità

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** sulla base delle istruzioni che l'utente aveva dato nel
comando `prepare_enter_net_phase_1`, dopo aver fatto passare la nuova identità per l'indirizzo
con posizione *virtuale* al livello 0, cambia il suo identificativo di livello 0 con la posizione
*reale* dentro il g-nodo destinazione. Cioè comunica questa variazione alla relativa istanza del modulo Qspn.

Oltre a ciò, il programma **qspnclient** esegue di seguito le due sequenze di
operazioni che vedremo adesso: la prima perché un identificativo dell'indirizzo di una sua
identità passa da *virtuale* a *reale* e la seconda perchè questo cambio avviene nell'identità *principale*.

##### Un identificativo passa da virtuale a reale

In questa occasione il programma **qspnclient** rimuove, nel network namespace interessato, da tutte le tabelle
presenti (anche quelle di inoltro il cui arco non ha ancora ricevuto alcun ETP) le
rotte verso gli indirizzi che non sono più validi a causa di questo nuovo identificativo *reale*.

Notiamo che questo avviene anche nelle identità *di connettività* (che gestiscono un namespace
diverso dal default) e anche se l'indirizzo Netsukuku non è del tutto *reale*.

```
ip route del 10.0.0.23/32 table ntk
ip route del 10.0.0.87/32 table ntk
ip route del 10.0.0.63/32 table ntk
ip route del 10.0.0.51/32 table ntk
ip route del 10.0.0.41/32 table ntk
ip route del 10.0.0.23/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.87/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.63/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.51/32 table ntk_from_00:16:3E:5B:78:D5
ip route del 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5
```

##### L'identità interessata dal cambio di indirizzo è la principale

In questa occasione il programma **qspnclient** aggiunge ad ogni interfaccia di rete reale nel network
namespace default gli indirizzi propri che prima non era possibile computare e adesso invece sì,
partendo dal livello più basso e salendo finché possibile.

Poi, solo se l'indirizzo è ora del tutto *reale*, aggiunge (opzionalmente) la regola di source-natting
e (opzionalmente) l'indirizzo IP anonimizzante.

Infine aggiorna tutte le rotte nella tabella `ntk` per fare in modo di mettere in esse (se disponibile)
un src preferito. In questo caso non avevamo nessuna rotta nota poiché ancora nessun ETP era stato ricevuto
e l'ingresso era da parte di un singolo nodo. Comunque riportiamo la sequenza di operazioni completa.

```
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
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.22/32 table ntk
ip route change unreachable 10.0.0.86/32 table ntk
ip route change unreachable 10.0.0.62/32 table ntk
ip route change unreachable 10.0.0.50/32 table ntk
ip route change unreachable 10.0.0.40/32 table ntk
```

#### Processazione di un ETP

Quando in una identità del sistema il QspnManager ha terminato di processare un ETP, esso
deve segnalarlo al programma. Inoltre il QspnManager deve indicare, con un metodo, per un
dato arco-qspn se ha già ricevuto tramite di esso almeno un ETP.

Il programma **qspnclient** per ogni arco-qspn tiene traccia se ha già aggiunto la regola
per la relativa tabella di inoltro nel relativo network namespace.

Questa sequenza di operazioni è eseguita dal programma **qspnclient** quando un suo
QspnManager ha terminato di processare un ETP. In essa, relativamente al network namespace
associato all'identità a cui il QspnManager appartiene, vengono aggiornate tutte le rotte di
tutte le tabelle presenti, ma per quelle di inoltro solo se abbiamo ricevuto almeno un
ETP da quell'arco; in seguito viene aggiunta la regola per le tabelle di inoltro il cui arco ha ricevuto
proprio adesso il primo ETP.

```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.16/30 table ntk
ip route change unreachable 10.0.0.80/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.20/31 table ntk
ip route change unreachable 10.0.0.84/31 table ntk
ip route change unreachable 10.0.0.60/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.22/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.86/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.23
ip route change 10.0.0.62/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.63
ip route change 10.0.0.50/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.94.223 dev eth1 src 10.0.0.41
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5
```

#### Dismissione identità

Sempre quando l'utente dà il comando `enter_net_phase_1`, in seguito alle operazioni viste
prima, il programma **qspnclient** dismette la vecchia identità.

```
ip netns exec entr02 ip route flush table main
ip netns exec entr02 ip link delete entr02_eth1 type macvlan
ip netns del entr02
```

[Operazione seguente](DettagliOperazioni5.md)
