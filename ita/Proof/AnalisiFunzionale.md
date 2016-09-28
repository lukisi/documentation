# Proof of concept - Analisi Funzionale

1.  [Ruolo del qspnclient](#Ruolo_del_qspnclient)
1.  [Operazioni del qspnclient](#Operazioni_del_qspnclient)
    1.  [Interazione programma-utente](#Interazione_programma_utente)
    1.  [Avvio del programma](#Avvio_programma)
    1.  [Primo segnale `bootstrap_complete`](#Primo_bootstrap_complete)
1.  [Vecchio](#Vecchio)
    1.  [Primi passi](#Primi_passi)
    1.  [Da riordinare](#Da_riordinare)
1.  [Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IPv4](#Mappatura_indirizzi_ip)
1.  [Identità](#Identita)
    1.  [Identità principale](#Identita_principale)
    1.  [Identità di connettività](#Identita_di_connettivita)
1.  [Indirizzi IP di ogni identità nel sistema](#Indirizzi_del_sistema)
1.  [Rotte nelle tabelle di routing](#Rotte_nelle_tabelle_di_routing)
    1.  [Source NATting](#Source_natting)
    1.  [Routing](#Routing)

Ci proponiamo di realizzare un programma, **qspnclient**, che si avvale del modulo QSPN e altri
moduli a sostegno (Neighborhood, Identities) per stabilire come impostare le rotte nelle tabelle
di routing del kernel di una macchina. Si tratta di un programma specifico per un sistema Linux.

Per giungere a delineare le regole generali descritte in questo documento si è proceduto ad un
esame dettagliato delle operazioni da fare su un sistema a fronte dei vari casi d'uso. Tale
esame è riportato nel documento [Casi d'uso](UseCases.md).

In seguito si è cercato di associare ogni operazione ad uno specifico momento del ciclo di vita
del programma **qspnclient**: il suo avvio, la rilevazione di qualche segnale da un dato modulo,
le richieste da parte dell'utente. Questo lavoro è riportato nel documento [Eventi](Eventi.md).

## <a name="Ruolo_del_qspnclient"></a>Ruolo del qspnclient

Questo programma permette all'utente di fare le veci del demone *ntkd*, simulando le sue operazioni
e quelle di pertinenza di altri moduli:

*   Il dialogo con identità in sistemi vicini appartenenti ad altre reti.
*   Il coordinamento di un g-nodo (moduli PeerServices e Coordinator).
*   La strategia di ingresso in una rete, cioè la scelta del g-nodo *g* in cui chiedere un posto.
*   La ricerca della più breve *migration path* per liberare un posto in *g* (modulo Migrations).
*   La comunicazione delle informazioni a tutte le identità interessate dalla migration path trovata.

Il programma *qspnclient* interagisce con l'utente (il quale ha a disposizione alcuni meccanismi per dare istruzioni
al programma) e con i moduli suddetti: QSPN, Neighborhood, Identities. Sulla base delle informazioni ottenute tramite
queste interazioni, interviene sulle configurazioni di rete del sistema. L'utente sarà quindi in grado di verificare che il
sistema riesca effettivamente a stabilire connessioni con gli altri sistemi della rete, che le rotte
siano quelle che ci si attende, eccetera. Inoltre il programma consente all'utente di chiedere la visualizzazione
di tutte le informazioni che il modulo QSPN ha raccolto.

## <a name="Operazioni_del_qspnclient"></a>Operazioni del qspnclient

### <a name="Interazione_programma_utente"></a>Interazione programma-utente

L'utente avvia il programma *qspnclient* su un sistema 𝛼 eseguendo su una shell il comando *qspnclient init*. In
questo momento fornisce alcuni dati iniziali come argomenti. Il comando avvia il programma *qspnclient* e
non restituisce il controllo della shell all'utente; la utilizza invece per visualizzare alcune informazioni
utili durante le sue operazioni.

Per le successive comunicazioni con il programma, l'utente dovrà aprire una nuova shell sul sistema 𝛼 e
da questa dare altri comandi (ad esempio *qspnclient enter_net*, ...) e con essi altri dati. Questi comandi
e informazioni saranno comunicate al programma *qspnclient* già in esecuzione, poi il comando restituirà
all'utente la shell, eventualmente dopo aver visualizzato le informazioni pertinenti.

Ai parametri che saranno individuati in modo autonomo dai moduli (ad esempio gli identificativi di
nodo, gli indirizzi di scheda, ...) verranno associati degli indici progressivi che saranno visualizzati
all'utente. L'utente si riferirà ad essi tramite questi indici. Questo per rendere più facilmente
riproducibili gli ambienti di test.

### <a name="Avvio_programma"></a> Avvio del programma

All'avvio del programma **qspnclient** l'utente specifica alcune informazioni:

*   Topologia della rete.
*   Se il sistema vuole fare da gateway per una sottorete a gestione autonoma, tale sottorete può
    avere la dimensione di un g-nodo di livello *i* nella topologia di rete specificata. L'utente
    specifica il livello di tale g-nodo. Default 0.
*   Lista di interfacce di rete da gestire.
*   Se il sistema ammette di essere usato come anonimizzatore.
*   Se il sistema (e tutta la eventuale sottorete autonoma dietro di lui) ammette di essere contattato
    in forma anonima.

*   Indirizzo Netsukuku **iniziale** del sistema.

#### Parte 1

All'avvio il programma computa l'indirizzo IP sinonimo di localhost per la topologia di rete
specificata. Il modo di calcolare questo indirizzo è spiegato [sotto](#Mappatura_indirizzi_ip).
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
livello *l* - 1 (dove *l* è il numero di livelli della topologia) e scendendo fino al livello 1.

```
for dev in $devlist; do
 ip address add $globalip dev $dev
done

if $allow_anonymous; then
 for dev in $devlist; do
  ip address add $anonymousip dev $dev
 done
fi

for i = l-1 to 1 step -1
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
*   Indichiamo con *subnet_level* il livello del g-nodo in cui la rete è a gestione autonoma
    dietro questo gateway.
*   Per *i* che scende da *l* - 1 a *subnet_level*, per *j* da 0 a *gsize(i)* - 1, se *pos_n(i)* ≠ *j*:
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

### <a name="Primo_bootstrap_complete"></a> Primo segnale `bootstrap_complete`

Immediatamente, poiché il sistema è inizialmente isolato, l'identità principale riceve dal QspnManager
il segnale `bootstrap_complete`.

Alla ricezione del segnale `bootstrap_complete` (da una qualsiasi identità) il programma **qspnclient**
computa gli indirizzi IP di tutte le possibili destinazioni relative all'indirizzo Netsukuku di
quella identità. Per ognuno, in base alle conoscenze di quella identità, aggiorna la rotta
nella tabella `ntk` e nelle tabelle di inoltro (`ntk_from_xxx`) presenti nel network namespace di
gestione di quella identità.

Nel presente caso assisteremo ad un aggiornamento della sola tabella `ntk`, in cui tutte le destinazioni
sono irraggiungibili. Ad esempio:

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

Questo blocco di comandi va eseguito senza intromissione di altri comandi da altre tasklet.

### <a name="Archi_vicini"></a> Archi con i sistemi vicini

Il modulo Neighborhood è quello che si occupa di rilevare i sistemi diretti vicini del sistema corrente
e di formare con essi degli archi. Il comando che realizza questo collegamento fra i vicini è comunque
eseguito dal programma **qspnclient** attraverso un delegato che questi fornisce al modulo Neighborhood.

Sia `$peerlinklocal` l'indirizzo IP link-local del sistema rilevato. Sia `$dev` l'interfaccia di rete
con cui è stato rilevato e `$linklocal` il relativo indirizzo di scheda. Il comando è:

```
ip route add $peerlinklocal dev $dev src $linklocal
```

Tuttavia, l'effettivo utilizzo dell'arco è deciso dal programma **qspnclient**. Il programma demanda all'utente
questa decisione. Questo permette all'utente di dirigere il proprio ambiente di test a piacimento anche in
particolari scenari, come ad esempio un gruppo di macchine virtuali che condividono un unico dominio di broadcast
ma vogliono simulare un gruppo di sistemi wireless disposti in un determinato modo.

Per ogni arco che il modulo Neighborhood realizza, le informazioni a disposizione (i due link-local e
i due MAC address) sono visualizzate all'utente. Soltanto agli archi che l'utente decide di accettare
e nell'ordine in cui sono accettati, il programma associa un indice autoincrementante *nodearc_nextindex*,
che parte da 0. In seguito il programma sfrutta questi archi passandoli al modulo Identities.

Sempre per dare all'utente il maggior controllo possibile sulle dinamiche del test, anche il costo
di un arco che viene rilevato dal modulo Neighborhood non è lo stesso che viene usato dal programma
**qspnclient**. L'utente quando accetta un arco dice quale costo gli vuole associare. In seguito può
variarlo a piacimento fino anche a simularne la rimozione.

### <a name="Ingresso_rete_1"></a> Ingresso in una rete - Caso 1

Quando due reti si incontrano, il programma **qspnclient** prevede che sia l'utente a simulare i meccanismi
di dialogo che portano a decidere:

*   Quale g-nodo fa ingresso in blocco nell'altra rete.
*   Quale g-nodo lo ospiterà nella nuova rete e con che posizione.
*   Quale migration path (eventualmente) porterà a liberare quella posizione.

Sarà ancora l'utente a simulare anche i meccanismi che portano all'avvio delle operazioni nei
vari singoli sistemi con le dovute informazioni.

Esaminiamo il caso più banale: sia *𝛼* un singolo nodo che costituiva una rete a sé; esso si incontra con un
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

Dall'esecuzione del metodo `add_identity` sul modulo Identities è necessario reperire:
*   identificativo della nuova identità
*   nome del vecchio network namespace (che passa dalla vecchia identità alla nuova)
*   nome del nuovo network namespace (che nasce per la vecchia identità)
*   associazione tra ogni arco-identità della vecchia identità quando era nel vecchio namespace
    e il corrispettivo arco-identità della vecchia identità nel nuovo namespace (oppure basta
    che data una identità e un arco-identità passato al modulo Qspn possa recuperare le
    relative informazioni attuali?)

#### Copia tabelle e regole, spostamento rotte

Il programma **qspnclient** conosce il nome del nuovo namespace. Assumiamo sia `entr02`.

Il programma **qspnclient** conosce il nome del vecchio namespace. Assumiamo sia ``, cioè il default.

Il programma **qspnclient** conosce l'indirizzo Netsukuku che la vecchia identità aveva nel vecchio
namespace. Assumiamo sia 1·0·1·0.

Il programma **qspnclient** conosce l'indirizzo Netsukuku *virtuale* che la vecchia identità come supporto
di connettività assume nel nuovo namespace. Assumiamo sia 1·0·1·3.

Il programma **qspnclient** aggiunge nel nuovo namespace la regola per la tabella primaria `ntk`.

```
ip netns exec entr02 ip rule add table ntk
```

Richiamando le modalità viste sopra, il programma **qspnclient** calcola tutti i possibili indirizzi IP
di destinazione, ognuno con suffisso CIDR, relativi all'indirizzo della vecchia identità nel nuovo
namespace, cioè 1·0·1·3. E li aggiunge alla tabella primaria `ntk` nel nuovo namespace.

```
ip netns exec entr02 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.12/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.76/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.60/30 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.8/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.72/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.56/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.11/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.75/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.59/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.51/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.41/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.10/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.74/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.58/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.50/32 table ntk
ip netns exec entr02 ip route add unreachable 10.0.0.40/32 table ntk
```

**sistema 𝛽**
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
iptables -t nat -D POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.10
ip address del 10.0.0.10/32 dev eth1
ip address del 10.0.0.74/32 dev eth1
ip address del 10.0.0.58/32 dev eth1
ip address del 10.0.0.50/32 dev eth1
ip address del 10.0.0.40/32 dev eth1
ip netns exec entr02 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.24/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.88/29 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.12/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.76/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.60/30 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.8/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.72/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.56/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.48/31 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.11/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.75/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.59/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.51/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.41/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.10/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.74/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.58/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.50/32 table ntk
ip netns exec entr02 ip route change unreachable 10.0.0.40/32 table ntk
```


## <a name="Vecchio"></a>Vecchio

### <a name="Primi_passi"></a>Primi passi

All'avvio del programma nel sistema viene creata l'istanza di NeighborhoodManager. Poi gli sono passati i nomi delle
interfacce di rete che dovrà gestire, tramite il metodo *start_monitor*. Ad ogni interfaccia, il modulo Neighborhood associa un
indirizzo IP link-local scelto a caso. In realtà l'assegnazione dell'indirizzo è fatta proprio
dal programma attraverso una classe che implementa l'interfaccia INeighborhoodIPRouteManager passata
al modulo Neighborhood. Quello che fa il modulo Neighborhood è scegliere l'indirizzo.

Il programma si avvede dell'indirizzo scelto perché il NeighborhoodManager lo notifica con il segnale
*nic_address_set*. Il programma associa questo proprio indirizzo link-local all'indice
autoincrementante *linklocal_nextindex*, che parte da 0.

* * *

L'istanza di NeighborhoodManager deve essere istruita sul numero massimo di archi che può accettare.
Il programma *qspnclient* specifica un numero elevato. Nonostante questo, ogni arco che il modulo
Neighborhood realizza passa al vaglio dell'utente che decide se utilizzarlo o meno. Questo permette
di dirigere il proprio ambiente di test a piacimento anche in particolari scenari, come ad esempio
un gruppo di macchine virtuali che condividono un unico dominio di broadcast ma vogliono simulare un
gruppo di sistemi wireless disposti in un determinato modo.

Per ogni arco che il modulo Neighborhood realizza, le informazioni a disposizione (i due link-local e
i due MAC address) sono visualizzate all'utente. Soltanto agli archi che l'utente decide di accettare
e nell'ordine in cui sono accettati, il programma associa un indice autoincrementante *nodearc_nextindex*,
che parte da 0. In seguito il programma sfrutta questi archi passandoli al modulo Identities.

Sempre per dare all'utente il maggior controllo possibile sulle dinamiche del test, anche il costo
di un arco che viene rilevato dal modulo Neighborhood non è lo stesso che viene usato dal programma
*qspnclient*. L'utente quando accetta un arco dice quale costo gli vuole associare. In seguito può
variarlo a piacimento fino anche a simularne la rimozione.

Quindi gli effettivi segnali di *arc_changed* del modulo Neighborhood sono in realtà ignorati dal programma.

* * *

All'avvio del programma nel sistema viene creata l'istanza di IdentityManager. Questi nel
costruttore crea la prima identità *principale* del sistema e per essa genera un NodeID casuale. Il programma
recupera tale NodeID col metodo *get_main_id()* e lo associa all'indice autoincrementante *nodeid_nextindex*,
che parte da 0. In seguito il programma quando crea una nuova identità col metodo *add_identity* associa la
nuova istanza di NodeID al prossimo valore di *nodeid_nextindex*. Quindi l'utente può usare questo indice per
dare dalla console comandi concernenti una certa identità.

### <a name="Da_riordinare"></a>Da riordinare

Alla creazione di una nuova identità, il modulo Identities crea un nuovo network namespace. In realtà la creazione
del network namespace è fatta proprio dal programma attraverso una classe che implementa l'interfaccia
IIdmgmtNetnsManager passata al modulo Identities. Quello che fa il modulo Identities è scegliere il nome di
tale namespace.

Il nome di tale namespace è già costruito con un indice progressivo, quindi il programma non gli associa un
indice.

Comunque (per il momento) non c'è nessun comando interattivo in cui l'utente debba specificare un
namespace. Quindi tale nome non è nemmeno mostrato all'utente.

* * *

Sempre alla creazione di una nuova identità, il modulo Identities, di nuovo con chiamate all'interfaccia
IIdmgmtNetnsManager, crea per ogni interfaccia di rete reale una pseudo-interfaccia. Anche qui il nome
della pseudo-interfaccia non è casuale ma generato con lo stesso indice progressivo usato per il
network namespace, quindi il programma non gli associa un indice.

Inoltre (per il momento) non c'è nessun comando interattivo in cui l'utente debba specificare una
pseudo-interfaccia. Quindi tale nome non è nemmeno mostrato all'utente.

Invece il modulo Identities genera casualmente un indirizzo IP link-local e lo associa alla pseudo-interfaccia
col metodo *add_address* di IIdmgmtNetnsManager. Il programma vede tale indirizzo e lo associa all'indice
autoincrementante *linklocal_nextindex*, che avevamo introdotto prima.

* * *

Quando al modulo Identities viene comunicato un arco, esso vi costruisce sopra automaticamente un
*arco-identità*. Al modulo Identities può venire espressamente richiesto dal suo utilizzatore (cioè dal
programma *qspnclient*) di aggiungere un arco-identità, ma di norma questo non si fa. Inoltre quando
viene creata una nuova identità coi metodi *prepare_add_identity* e *add_identity* (a seguito di una
migrazione o un ingresso in una rete) tutti gli archi-identità della precedente identità vengono duplicati.
Infine quando una identità nuova si crea in un vicino (a seguito di una migrazione che coinvolge l'identità
del vicino ma non coinvolge la nostra identità) il modulo Identities riceve direttamente dalla rete
istruzioni per creare un nuovo arco-identità.

In ogni occasione in cui viene aggiunto un arco-identità, il modulo Identities emette un segnale
*identity_arc_added* con i dati "arco" (istanza di IIdmgmtArc), "propria identità" (istanza di NodeID)
e "arco-identità" (istanza di IIdmgmtIdentityArc). In questo momento il programma *qspnclient* può
identificare il nuovo arco-identità con l'indice autoincrementante *identityarc_nextindex*, che parte da 0.
Ad ogni indice rimane associato sia l'arco, sia la propria identità, sia l'identità nel sistema vicino.
Ricordiamo che dall'associazione "arco + propria identità" si può risalire al link-local dell'identità nel proprio sistema,
che nel tempo può cambiare. Ricordiamo che dall' "arco-identità" si può risalire sia al NodeID del vicino
(che non cambia nel tempo) sia al link-local dell'identità nel sistema vicino, che nel tempo può cambiare.

* * *

All'avvio del programma nel sistema viene creata la prima istanza di QspnManager con il costruttore *create_net* e viene
associata alla prima *identità principale* del sistema. Così si costruisce inizialmente una rete nuova che
comprende solo questa identità. I dati che servono sono forniti dall'utente sulla riga di comando: i dati della topologia e il
primo indirizzo Netsukuku che questa identità si assegna. L'identificativo del fingerprint a livello 0 è scelto
a caso dal programma e le anzianità sono a zero (primo g-nodo) a tutti i livelli.

Siccome il QSPN è un modulo di identità, ogni istanza di QspnManager viene memorizzata come membro di una
identità nel modulo Identities. Quindi non è necessario un ulteriore indice perché l'utente possa
referenziare una istanza di QspnManager: è sufficiente l'indice *nodeid_nextindex*.

* * *

Supponiamo ora che l'utente nel sistema *A* vuole simulare l'ingresso dell'identità *A<sub>0</sub>* in un'altra rete esistente
attraverso un arco tra i sistemi *A* e *B* e in particolare che collega le identità *A<sub>0</sub>* e *B<sub>0</sub>*.

L'utente chiederà al programma in esecuzione su *A* di costruire una nuova *identità* *A<sub>1</sub>*
basata su *A<sub>0</sub>*. Facendo questo si duplica anche l'arco-identità *A<sub>0</sub>*-*B<sub>0</sub>*
in un nuovo arco-identità *A<sub>1</sub>*-*B<sub>0</sub>* e di questo fatto si avvede autonomamente
l'IdentityManager sia nel sistema *A*, sia nel sistema *B*.

Poi l'utente chiederà al programma in esecuzione su *A*, con un comando interattivo che chiamiamo
*enter_net*, di costruire per essa una nuova istanza di QspnManager con il costruttore *enter_net*. I dati che
servono sono forniti dall'utente in modo interattivo: l'indirizzo del nuovo g-nodo prenotato nella
rete appena scoperta, la sua anzianità, gli archi-identità che ci collegano alla nuova rete, e altri.

Esaminiamo gli archi-identità. Per l'arco-identità *A<sub>1</sub>*-*B<sub>0</sub>* che in questo caso l'utente specifica (tramite
il suo indice) quando da il comando *enter_net* sul sistema *A*, il programma crea una istanza di IQspnArc
tale che il modulo QSPN di *A<sub>1</sub>* possa effettuare comunicazioni con il modulo QSPN
di *B<sub>0</sub>*.

Immediatamente l'utente dovrà dare un comando al programma in esecuzione su *B*, che chiamiamo *add_qspnarc*,
con il quale gli chiede di costruire e passare al QspnManager di *B<sub>0</sub>* una istanza di IQspnArc
tale che il modulo QSPN di *B<sub>0</sub>* possa effettuare comunicazioni con il modulo QSPN
di *A<sub>1</sub>*.

Poi l'utente chiederà al programma in esecuzione su *A* di rendere la vecchia *identità* *A<sub>0</sub>*
una identità di connettività che andrà da lì a breve a scomparire. Per prima cosa l'utente usa il comando
*make_connectivity*, il quale con l'omonimo metodo del QspnManager di *A<sub>0</sub>* rende quella identità
di connettività. L'utente specifica a quale livello l'indirizzo Netsukuku di *A<sub>0</sub>* deve diventare
*virtuale* (nel nostro caso 0), quale indirizzo virtuale deve prendere e con quale anzianità (dati che sarebbero
da concordare con il Coordinator) e fino a quale livello arriva la migrazione (nel nostro caso equivale al
numero totale dei livelli poiché si è entrati in una diversa rete).  
Ora l'utente deve attendere un po' (un breve istante è sufficiente) per simulare l'attesa che il demone
*ntkd* farebbe a questo punto per due motivi: bisogna attendere un istante per permettere che l'ETP prodotto
da *A<sub>0</sub>* per segnalare ai vicini la rimozione del vecchio identificativo *reale*, venga da essi elaborato;
bisogna inoltre attendere che *A<sub>1</sub>* notifichi il segnale `presence_notified`.  
Poi l'utente con il comando *remove_outer_arcs* chiede al programma di eseguire l'omonimo metodo del
QspnManager di *A<sub>0</sub>* per rimuovere gli archi-identità, se ci sono, esterni al massimo g-nodo
per il quale si rimane di supporto alla connettività. In questo caso per forza non vi sono archi, essendo
questo g-nodo l'intera vecchia rete.  
Poi l'utente verifica con il comando *check_connectivity*, il quale usa l'omonimo metodo del
QspnManager di *A<sub>0</sub>*, che la permanenza di quella identità è ora superflua. Il programma
scriverà a video l'esito di questa verifica.  
Infine l'utente con il comando *remove_identity* chiederà al programma nel sistema A di rimuovere l'identità
*A<sub>0</sub>*.

* * *

Per ogni *identità* creata, nel relativo network namespace, sulla base dell'indirizzo Netsukuku
ad essa associato, il programma computa un numero di indirizzi IP (in seguito verrà dettagliato
come siano computati) e se li assegna.

Inoltre, sempre per ogni *identità* e nel relativo network namespace, basandosi sui segnali notificati
dalla relativa istanza di QspnManager, il programma popola le tabelle di routing del kernel.

* * *

Al lancio del programma *qspnclient* l'utente indica attraverso appositi flag come vuole che il sistema si
comporti riguardo le forme di contatto anonimo. Questo concetto verrà spiegato più sotto. Il comportamento
di default del programma, se l'utente non indica alcun flag a riguardo, è di:

*   abilitare l'anonimizzazione dei pacchetti che transitano per il sistema;
*   non accettare richieste indirizzate al sistema da un client in forma anonima.

## <a name="Mappatura_indirizzi_ip"></a>Mappatura dello spazio di indirizzi Netsukuku nello spazio di indirizzi IPv4

Gli indirizzi Netsukuku dei *nodi del grafo* vanno mappati in un range di indirizzi IP che si
decide di destinare alla rete Netsukuku. Nell'attuale implementazione si presume che questo
range sia la classe IPv4 10.0.0.0/8.

La rete viene suddivisa in un numero arbitrario di livelli.
La [notazione CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) usata per individuare
classi di indirizzi nelle reti IPv4, ci obbliga ad usare come gsize ad ogni livello una potenza di 2.
In teoria, ad esempio, potremmo avere un livello 0 di gsize 5. Ma quando vogliamo indicare nelle
tabelle di routing tutti gli indirizzi di un g-nodo di livello 1 (ad esempio da 10.1.2.0 a 10.1.2.4) non
potremmo farlo in forma compatta. Invece se usiamo un livello 0 di gsize 8, per riferirci agli indirizzi
nel g-nodo da 10.1.2.0 a 10.1.2.7 possiamo usare la notazione 10.1.2.0/29; per gli indirizzi da 10.1.2.8
a 10.1.2.15 useremo 10.1.2.8/29; e così via.

Indichiamo con *l* il numero dei livelli. Indichiamo con *gsize(i)* la dimensione dei g-nodi di livello
*i* + 1. Tali dimensioni devono essere potenze di 2. Indichiamo con *g-exp(i)* l'esponente della potenza
di 2 che dà la dimensione dei g-nodi di livello *i* + 1. Il numero di bit necessari a coprire lo spazio
di indirizzi è dato dalla sommatoria per *i* da 0 a *l* - 1 di *g-exp(i)*. 𝛴 *<sub>0 ≤ i < l</sub>* *g-exp(i)*.

Questo numero di bit non può essere maggiore di 22. Gli indirizzi IP nella classe 10.0.0.0/8 hanno 24 bit
a disposizione. Però ai bit sfruttati per la rappresentazione di un indirizzo Netsukuku vanno aggiunti 2 bit,
nella posizione più significativa, che riserviamo per notazioni particolari (routing interno ai g-nodi e
forma anonima).

Inoltre dobbiamo assicurarci che *gsize(l-1)* ≥ *l*. Cioè che nello spazio destinato al livello più alto
sia possibile rappresentare un numero da 0 a *l* - 1. Questo pure ci serve per la notazione usata per il
routing interno ai g-nodi.

Una volta scelti i valori di *l* e di *g-exp(i)* rispettando i vincoli prima esposti, questa mappatura
associa ad un indirizzo Netsukuku *reale* un numero di indirizzi IP:

*   Un indirizzo IP globale.  
    Questo indirizzo IP identifica un preciso *sistema* univocamente all'interno
    di tutta la rete.
*   Un indirizzo IP globale *anonimizzante*.  
    Questo indirizzo IP identifica un preciso *sistema* univocamente all'interno
    di tutta la rete. È una diversa rappresentazione, rispetto all'indirizzo IP globale,
    che identifica lo stesso *sistema*; ma questa convoglia in più l'informazione che si vuole
    contattare quel *sistema* restando anonimi.
*   Un indirizzo IP interno al livello *i* per ogni valore di *i* da 0 a *l* - 1.  
    Questo indirizzo IP identifica un preciso *sistema* univocamente all'interno
    di un g-nodo *g* di livello *i*. Questo indirizzo IP si può utilizzare come indirizzo di
    destinazione di un pacchetto IP quando sia il *sistema* mittente che il *sistema*
    identificato (il destinatario) appartengono allo stesso g-nodo *g* di livello *i*. La
    peculiarità di questi indirizzi IP (che si riflette sui pacchetti IP trasmessi a questi
    indirizzi) è che essi non cambiano quando il g-nodo *g* o uno dei suoi g-nodi superiori
    *migra* all'interno della rete o anche in una diversa rete Netsukuku.  
    Anche per il livello 0 si calcola un tale indirizzo IP. Questo indirizzo ha un significato
    simile a `localhost` nel senso che identifica come destinazione lo stesso sistema mittente.

Ricordiamo che un indirizzo Netsukuku identifica un *nodo del grafo*, cioè una specifica identità
all'interno di un *sistema*. Però in ogni sistema, in ogni momento, esiste una ed una sola
*identità principale*, che è l'unica che detiene un indirizzo Netsukuku *reale*.

Per l'esattezza, l'identità principale di un sistema può detenere per brevi istanti un
indirizzo Netsukuku *virtuale*, durante le operazioni di una migrazione coinvolta in una
*migration path*. Durante questo periodo quel sistema (e insieme a quello anche tutti
gli altri sistemi che appartengono al g-nodo *g* che sta migrando) non può avere un indirizzo
IP globale. Questo però non inficia sulla possibilità di quel sistema di avere un indirizzo
IP interno al livello *i* per ogni valore di *i* da 1 a *k*, dove *k* è il livello
del g-nodo *g*.

Questo permette, come detto prima, che le connessioni realizzate tra due sistemi appartenenti
al g-nodo *g* non vengano compromesse. Per questo aggiungiamo che la mappatura associa alcuni
indirizzi IP anche ad un indirizzo Netsukuku *virtuale*, purché siano *reali* i suoi
identificativi da 0 a *k* - 1. Questi sono:

*   Un indirizzo IP interno al livello *i* per ogni valore di *i* da 1 a *k*.

Gli algoritmi di calcolo dei vari tipi di indirizzo IP sono descritti nel documento [IndirizziIP](IndirizziIP.md).

## <a name="Identita"></a>Identità

Ogni identità che vive nel sistema ha un suo indirizzo Netsukuku. Inoltre ha una mappa di percorsi, ognuno
che ha come destinazione (e come passi) un g-nodo *visibile* dal suo indirizzo Netsukuku.

Un sistema ha sempre una identità principale e zero o più identità di connettività.

### <a name="Identita_principale"></a>Identità principale

L'identità principale gestisce il network namespace default. L'identità principale ha un indirizzo
Netsukuku *definitivo* che può essere *reale* o *virtuale*.

Se è *reale*, nel network namespace default:

*   Il sistema si assegna l'indirizzo IP globale di *n*.
*   Il sistema può (opzionalmente) assegnarsi l'indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *l* - 1:
    *   Il sistema si assegna l'indirizzo IP interno al livello *j* + 1 di *n*
        (non quando *j* + 1 = *l*; in quel caso abbiamo solo l'indirizzo IP globale).
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.  
                Si tenga presente che se un processo locale vuole inviare un pacchetto a questo
                indirizzo IP, allora come *src* preferito dovrà usare l'indirizzo IP globale di *n*.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.  
                Si tenga presente che se un processo locale vuole inviare un pacchetto a questo
                indirizzo IP, allora come *src* preferito dovrà usare l'indirizzo IP globale di *n*.
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    In questo caso è garantito che tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*, cosa necessaria affinché
                    l'indirizzo IP interno al livello *t* del g-nodo *d* abbia significato.  
                    Si tenga presente che se un processo locale vuole inviare un pacchetto a questo
                    indirizzo IP, allora come *src* preferito dovrà usare l'indirizzo IP
                    di *n* interno al livello *t*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d* e con *n<sub>x</sub>* l'indirizzo IP che
            il sistema *n* intende usare come *src* preferito.
            *   Il sistema imposta una rotta in *partenza* verso *d<sub>x</sub>*. In essa specifica
                l'indirizzo *n<sub>x</sub>* come *src* preferito.  
                Ricordiamo che nelle tabelle di routing del kernel si riporta come informazione solo
                il primo gateway di una rotta, sebbene l'identità sia a conoscenza di altre informazioni.  
                Viene impostata la rotta identificata dal miglior percorso noto all'identità per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile"
                perché abbiamo appena detto che l'identità conosce la destinazione *d*, quindi almeno
                un percorso verso *d*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*: l'identità potrebbe non conoscere nessun percorso
                    verso *d* che non passi per il massimo distinto g-nodo di *m* per *n*.
            *   Il sistema dovrebbe impostare una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                In realtà, per come funziona lo stack TCP/IP di Linux, tale rotta è superflua. Infatti
                essa sarebbe identica a quella che è stata impostata per i pacchetti in *partenza* verso
                *d<sub>x</sub>* e lo stack TCP/IP prende in considerazione questa anche per i pacchetti
                in *inoltro*. Perciò il programma *qspnclient* non imposta una ulteriore rotta.

Se è *virtuale*, significa che l'indirizzo ha una o più componenti virtuali. Sia *i* il livello
più basso in cui la componente è virtuale. Sia *k* il livello più alto in cui la componente è virtuale.

In questo caso, nel network namespace default:

*   NON esiste un indirizzo IP globale di *n*.
*   NON esiste un indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *i*-1:
    *   Il sistema si assegna l'indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    Questo indirizzo IP va calcolato solo se tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*, cosa necessaria affinché
                    l'indirizzo IP interno al livello *t* del g-nodo *d* abbia significato.  
                    Si tenga presente che se un processo locale vuole inviare un pacchetto a questo
                    indirizzo IP, allora come *src* preferito dovrà usare l'indirizzo IP
                    di *n* interno al livello *t*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d* e con *n<sub>x</sub>* l'indirizzo IP che
            il sistema *n* intende usare come *src* preferito.
            *   Il sistema imposta una rotta in *partenza* verso *d<sub>x</sub>*. In essa specifica
                l'indirizzo *n<sub>x</sub>* come *src* preferito.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema dovrebbe impostare una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce. Ma come detto
                sopra lo stack TCP/IP di Linux si avvale della rotta che è stata impostata per i pacchetti
                in *partenza* verso *d<sub>x</sub>*. Perciò il programma *qspnclient* non imposta una ulteriore rotta.
*   Per ogni livello *j* da *i* a *k*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    Questo indirizzo IP va calcolato solo se tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*.  
                    In questo caso è impossibile per un processo locale inviare un pacchetto a questo
                    indirizzo IP, non potendo usare un indirizzo IP
                    di *n* interno al livello *t*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d*.
            *   Il sistema NON ha una rotta in *partenza* verso *d<sub>x</sub>*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".
*   Per ogni livello *j* da *k* a *l*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.  
                In questo caso è impossibile per un processo locale inviare un pacchetto a questo
                indirizzo IP, non potendo usare l'indirizzo IP globale di *n*.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.  
                In questo caso è impossibile per un processo locale inviare un pacchetto a questo
                indirizzo IP, non potendo usare l'indirizzo IP globale di *n*.
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    In questo caso è garantito che tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*.  
                    In questo caso è impossibile per un processo locale inviare un pacchetto a questo
                    indirizzo IP, non potendo usare un indirizzo IP
                    di *n* interno al livello *t*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d*.
            *   Il sistema NON ha una rotta in *partenza* verso *d<sub>x</sub>*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".

### <a name="Identita_di_connettivita"></a>Identità di connettività

Un sistema può avere 0 o più identità di connettività. L'identità di connettività gestisce un certo
network namespace. L'identità di connettività ha un indirizzo Netsukuku *di connettività* che è *virtuale*.

Significa che l'indirizzo ha una o più componenti virtuali. Sia *i* il livello più basso in cui
la componente è virtuale. Sia *k* il livello più alto in cui la componente è virtuale.

Nel network namespace gestito da questa identità:

*   NON esiste un indirizzo IP globale di *n*.
*   NON esiste un indirizzo IP globale anonimizzante di *n*.
*   Per ogni livello *j* da 0 a *k* - 1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    Questo indirizzo IP va calcolato solo se tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*.  
                    In questo caso è impossibile che un processo locale voglia inviare un pacchetto a questo
                    indirizzo IP, poiché questa identità non è nel network namespace default.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d*.
            *   Il sistema NON ha una rotta in *partenza* verso *d<sub>x</sub>*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".
*   Per ogni livello *j* da *k* a *l*-1:
    *   NON esiste un indirizzo IP interno al livello *j* + 1 di *n*.
    *   Per ogni g-nodo *d* di livello *j* che l'identità conosce, e solo quelli la cui
        componente (a livello *j*) è *reale*:
        *   Il sistema computa questi indirizzi IP:
            *   *d<sub>g</sub>* - Indirizzo IP globale del g-nodo *d*.  
                Di nuovo, è impossibile che un processo locale voglia inviare un pacchetto a questi
                indirizzi IP, poiché questa identità non è nel network namespace default.
            *   *d<sub>a</sub>* - Indirizzo IP globale anonimizzante del g-nodo *d*.  
            *   Per ogni valore *t* da *j* + 1 a *l* - 1 inclusi:
                *   *d<sub>i[t]</sub>* - Indirizzo IP interno al livello *t* del g-nodo *d*.  
                    In questo caso è garantito che tutte le componenti dell'indirizzo di *n*
                    dal livello *j* + 1 al livello *t* - 1 sono *reali*.
        *   Esaminiamo ognuno di questi indirizzi IP. Indichiamo con *d<sub>x</sub>*
            questo indirizzo riferito a *d*.
            *   Il sistema NON ha una rotta in *partenza* verso *d<sub>x</sub>*.
            *   Per ogni MAC address *m* di diretto vicino che l'identità conosce:
                *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                    provenienti da *m*.  
                    Viene impostata la rotta identificata dal miglior percorso noto per quella
                    destinazione che non passi per il massimo distinto g-nodo di *m* per *n*.  
                    La destinazione *d<sub>x</sub>* può essere "non raggiungibile" per i pacchetti
                    in *inoltro* provenienti da *m*.
            *   Il sistema imposta una rotta in *inoltro* verso *d<sub>x</sub>* per i pacchetti
                provenienti da un MAC address che non è fra quelli che l'identità conosce.  
                Viene impostata la rotta identificata dal miglior percorso noto per quella
                destinazione. La destinazione *d<sub>x</sub>* non può essere "non raggiungibile".

## <a name="Indirizzi_del_sistema"></a>Indirizzi IP di ogni identità nel sistema

Come abbiamo visto prima, in un sistema possono esistere diverse identità. Ogni identità detiene un
indirizzo Netsukuku. A seconda del tipo, sulla base del suo indirizzo Netsukuku ogni identità
può volersi assegnare zero o più indirizzi IPv4.

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa il programma *qspnclient*.

In un sistema Linux il sistema si assegna un indirizzo associandolo ad una interfaccia di rete. In realtà
ogni interfaccia può avere più indirizzi e anche lo stesso indirizzo può essere associato a più interfacce;
inoltre anche se un indirizzo *x* viene associato alla interfaccia *nicA* e non all'interfaccia *nicB*, un
pacchetto destinato a *x* ricevuto tramite l'interfaccia *nicB* giunge ugualmente al processo che sta in
ascolto sull'indirizzo *x*.

Si noti che il fatto di associare un indirizzo IP ad una specifica interfaccia di rete
ha la sua importanza in relazione al protocollo di risoluzione degli indirizzi IP in
indirizzi MAC ([Address Resolution Protocol](https://en.wikipedia.org/wiki/Address_Resolution_Protocol)).
Infatti quando un sistema *a* vuole trasmettere un pacchetto IP ad un suo diretto vicino *b*, esso conosce
l'indirizzo IP di *b* e l'interfaccia di rete di *a* dove trasmettere. Il sistema *a* trasmette un frame Ethernet broadcast
su quel segmento di rete una richiesta: «chi ha l'indirizzo IP XYZ?». Il sistema *b* risponde indicando al
sistema *a* l'indirizzo MAC della sua interfaccia di rete. Quindi il sistema *a* può incapsulare il pacchetto
IP in un frame Ethernet unicast che riporta gli indirizzi MAC dell'interfaccia che trasmette e dell'interfaccia che deve ricevere.

Se il sistema *b* ha diverse interfacce di rete (all'interno di un unico network namespace) tutte
collegate allo stesso segmento di rete, il fatto
di associare diversi indirizzi IP a diverse interfacce può fornire un modo di identificare una precisa
interfaccia di rete a cui un pacchetto va trasmesso. Questo è usato dal modulo Qspn per distinguere
i pacchetti che vanno ricevuti da una pseudo-interfaccia, per questo gli indirizzi link-local sono diversi
su ogni interfaccia e sulle pseudo-interfacce.

Fatta questa premessa, come si comporta il programma?

Il programma *qspnclient*, quando crea una *identità*, a seconda del tipo di identità e del suo indirizzo
Netsukuku, come visto prima, computa gli indirizzi IP che si deve assegnare e li associa tutti
a ognuna delle interfacce di rete che gestisce quell'identità.

Inoltre, prima di rimuovere una identità, li rimuove da tutte le interfacce che gestisce quell'identità.

## <a name="Rotte_nelle_tabelle_di_routing"></a>Rotte nelle tabelle di routing

Il programma deve istruire le policy di routing del sistema (che di norma significa impostare delle
rotte nelle tabelle di routing) in modo da garantire questi comportamenti:

*   Se un processo locale vuole inviare un pacchetto ad un indirizzo IP *x* nello spazio destinato alla rete Netsukuku:
    *   L'indirizzo IP *x* identifica un indirizzo Netsukuku reale *d*. Può trattarsi di un
        indirizzo IP globale, globale anonimizzante o interno.
    *   Se il modulo QSPN non ha alcun percorso verso la destinazione *d*:
        *   Il sistema non trova nelle tabelle del kernel alcuna rotta e segnala al processo
            che la destinazione *x* è irraggiungibile.
    *   Altrimenti:
        *   Il sistema trova nelle tabelle del kernel una rotta il cui gateway è costituito dal
            primo hop del miglior percorso (fra tutti quelli che il modulo QSPN ha scoperto) verso
            la destinazione *d*.
*   Se un pacchetto è ricevuto da un vicino *v* ed è destinato ad un indirizzo IP *x* nello spazio
    destinato alla rete Netsukuku che non è un indirizzo IP del sistema:
    *   L'indirizzo IP *x* identifica un indirizzo Netsukuku reale *d*. Può trattarsi di un
        indirizzo IP globale, globale anonimizzante o interno.
    *   Se il modulo QSPN non ha alcun percorso verso la destinazione *d*, tale che non contenga
        fra i suoi hop il *massimo distinto g-nodo* del vicino *v*:
        *   Il sistema non trova nelle tabelle del kernel alcuna rotta; quindi scarta il pacchetto
            e invia un pacchetto ICMP «host *x* irraggiungibile» al mittente.
    *   Altrimenti:
        *   Il sistema trova nelle tabelle del kernel una rotta il cui gateway è costituito dal primo
            hop per il miglior percorso verso la destinazione *d*, tale che non contenga il g-nodo del vicino.
        *   Se l'indirizzo IP *x* è globale anonimizzante:
            *   Opzionalmente, il sistema trova una regola che gli impone di mascherare l'indirizzo
                del mittente (cioè di indicare se stesso come mittente e inoltrare le comunicazioni di
                risposta che riceverà al vero mittente).

* * *

Vediamo come queste impostazioni si configurano in un sistema Linux e quindi quali operazioni fa
il programma *qspnclient*.

Abbiamo già avuto modo di evidenziare che un sistema Linux è in grado di replicare il suo
intero network stack in molti distinti network namespace. E che i moduli che stiamo esaminando
assumono come requisito una capacità di questo tipo. Nella trattazione che segue parliamo sempre
di concetti (come le tabelle di routing, l'assegnazione degli indirizzi alle interfacce di rete,
la manipolazione dei pacchetti, ...) che sono da riferirsi ad un particolare network stack.

Esaminiamo prima l'aspetto del source natting e poi del routing.

### <a name="Source_natting"></a>Source NATting

Il [source NATting](https://en.wikipedia.org/wiki/Network_address_translation) in un sistema Linux
può essere realizzato istruendo il kernel con il comando `iptables` (utilizzando una regola con
l'estensione `SNAT` nella catena `POSTROUTING` della tabella `nat`). Quando un pacchetto
da inoltrare rientra nei parametri di questa regola (un esempio di parametro che si può usare è
l'indirizzo di destinazione del pacchetto che rientra in un dato range) allora il kernel modifica
l'indirizzo mittente nel pacchetto sostituendolo con uno dei suoi indirizzi pubblici. Inoltre compie
una serie di altre operazioni volte a mantenere inalterate il più possibile le caratteristiche della
comunicazione (ad esempio la connessione stabilita dal protocollo TCP).

Ad esempio, con il comando «`iptables -t nat -A POSTROUTING -d 10.128.0.0/9 -j SNAT --to 10.1.2.3`»
si ottiene che tutti i pacchetti da inoltrare alle destinazioni 10.128.0.0/9 vanno rimappati al mio indirizzo 10.1.2.3

Fatta questa premessa, come si comporta il programma?

Il programma, all'avvio, opzionalmente, istruisce il kernel per il source natting. Con questa configurazione
il sistema si rende disponibile ad anonimizzare i pacchetti che riceve e che vanno inoltrati verso una
destinazione che accetta richieste anonime.

L'opzione di rendere anonimi i pacchetti che transitano per il sistema nel percorso verso un'altra destinazione
è distinta e indipendente dall'opzione di accettare richieste anonime, che è stata discussa sopra.

*   **NOTA**: La seguente spiegazione sui motivi per cui l'operazione è opzionale va spostata in un
    documento che affronta ad alto livello le implicazioni sul detenere un sistema nella rete Netsukuku. Il documento
    presente si limita a illustrare i dettagli implementativi del programma *qspnclient* o del demone *ntkd*.  
    Questa azione è opzionale perché il proprietario di un sistema può avere remore a nascondere il vero mittente
    di un messaggio prendendo il suo posto. In realtà questo timore sarebbe infondato, vediamo perché. Per far
    funzionare bene l'operazione di contatto anonimo da parte del client, occorre che il sistema che fa da server
    (fornisce un servizio) si assegni anche gli indirizzi per essere contattato in forma anonima. Se fa questa
    operazione opzionale, significa che è pronto a ricevere alcune richieste dalle quali saprà di non
    poter risalire al mittente. Sarà quindi responsabile di rispondere o meno a tali richieste e non
    potrà far ricadere tale responsabilità sugli altri sistemi.  
    Anche considerando quindi non rischiosa l'azione di implementare nel proprio sistema il source natting,
    l'azione è opzionale perché il sistema che la implementa si carica di un onere che costa un po' in
    termini di memoria. Se il sistema quindi ha scarse risorse (si intende molto scarse, come pochi mega
    di RAM) conviene che non la implementi.  
    Va considerato che se un sistema decide di non implementare questa azione, comunque il meccanismo di
    trasmissione anonima risulta efficace se nel percorso tra il mittente e il destinatario almeno un sistema è
    disposto a implementarla. Invece, se un sistema decide di implementare l'azione e ad un certo punto le sue risorse
    di memoria venissero meno, in questo caso la comunicazione in corso ne verrebbe compromessa.

Se il sistema decide di implementare il source natting, calcola lo spazio di indirizzi che indicano una
risorsa da raggiungere in forma anonima. Una volta calcolato il numero di bit necessari a codificare
un indirizzo Netsukuku *reale* nella topologia della nostra rete, va considerato che nei successivi
2 bit in testa va codificato (per gli indirizzi IP globali *anonimizzanti*) il numero 2, in binario `|1|0|`.

Facciamo un esempio. Supponiamo di destinare alla rete Netsukuku tutta la classe 10.0.0.0/8 di IPv4.
Consideriamo una topologia di rete con 4 livelli. Diamo 2 bit al livello 3, 4 bit al livello 2, 8 bit
ai livelli 1 e 0. Sono soddisfatti i vincoli esposti sopra.

In questo esempio, il range di indirizzi che individuano a livello globale una risorsa da
raggiungere in forma anonima è `10.128.0.0/10`.

Supponiamo che l'indirizzo Netsukuku del nostro sistema in questa topologia sia 3·10·123·45.
L'indirizzo IP globale del sistema è 10.58.123.45. L'indirizzo IP globale *anonimizzante* del sistema è 10.186.123.45.

Allora il programma istruisce il kernel di modificare i pacchetti destinati al range `10.128.0.0/10`
indicando come nuovo indirizzo mittente il suo indirizzo globale (non quello *anonimizzante*). Il comando è il seguente:

```
   iptables -t nat -A POSTROUTING -d 10.128.0.0/10 -j SNAT --to 10.58.123.45
```

Quando il programma termina, se aveva istruito il kernel per fare il source natting, rimuove le regole
che aveva messe nella catena `POSTROUTING` della tabella `nat`.

### <a name="Routing"></a>Routing

In un sistema Linux le rotte vengono memorizzate in diverse tabelle. Queste tabelle hanno un
identificativo che è un numero da 0 a 255. Hanno anche un nome descrittivo: l'associazione del
nome al numero (che in realtà è il vero identificativo) è fatta nel file `/etc/iproute2/rt_tables`.

In ogni tabella possono esserci diverse rotte. Ogni rotta ha alcune informazioni importanti:

*   Destinazione. È in formato CIDR. Indica una classe di indirizzi. Solo se il pacchetto è destinato
    a quella classe allora la rotta va presa in considerazione. Se ci sono più classi che soddisfano il
    pacchetto, allora si prende quella più restrittiva.
*   Unreachable. Se presente indica che la destinazione è irraggiungibile.
*   Gateway (gw) e interfaccia (dev). Dice dove trasmettere il pacchetto.
*   Mittente preferito (src). Questa informazione è usata solo per i pacchetti trasmessi dal sistema
    locale, non per i pacchetti da inoltrare. È un indirizzo del sistema locale. Dice quale indirizzo
    locale usare come mittente, se non viene espressamente specificato un indirizzo locale dal processo
    che richiede la trasmissione.

Quando un pacchetto va inviato ad una certa destinazione, ci sono delle regole che dicono al sistema su
quali tabelle guardare. Queste regole, visibili con il comando `ip rule list`, di default dicono di
guardare per prima la tabella `local`, per penultima la tabella `main` e per ultima la tabella
`default`. Tra la regola che dice di guardare la `local` e quella che dice di guardare la `main` possono
essere inserite altre regole.

Ogni regola può dire semplicemente di guardare una tabella, oppure di guardarla solo a determinate
condizioni. Una particolare condizione che ci torna utile è questa: «guarda la tabella `XXX` se il
pacchetto da trasmettere è marcato con il numero `YYY`». La marcatura del pacchetto è virtuale, nel
senso che i dati del pacchetto non sono affatto modificati, ma solo il sistema locale lo vede come
marcato; ed è sempre il sistema locale che lo ha precedentemente marcato. Questa marcatura viene
fatta da una parte del kernel che può essere istruita usando l'azione `MARK` del comando `iptables`.

Occorre evidenziare che, in presenza di molteplici network namespace, (di default) il file che
associa il nome mnemonico della tabella al suo numero, `/etc/iproute2/rt_tables`, è comune a tutti
i namespace. Invece le regole di scelta della tabella da esaminare e il contenuto delle tabelle
è distinto in ogni namespace.

Fatta questa premessa, come si comporta il programma?

Il programma, attraverso i moduli Neighborhood e Identities, ha già automaticamente ottenuto
che nella tabella `main` di ogni network namespace siano memorizzate rotte dirette (cioè senza gateway)
verso ogni suo diretto vicino (più precisamente verso ogni *identità* sua vicina). In queste rotte
sono indicati gli indirizzi di scheda delle proprie interfacce e di quelle dei vicini.

Il programma crea una tabella `ntk` con identificativo `YYY`, dove `YYY` è il primo identificativo
libero nel file `/etc/iproute2/rt_tables`. Tale tabella sarà inizialmente vuota; in essa il programma
andrà a mettere le rotte di pertinenza della rete Netsukuku, cioè quelle con destinazione nello
spazio 10.0.0.0/8. Inoltre aggiunge una regola che dice di guardare la tabella `ntk` prima della `main`.

Il programma, per ogni suo arco, crea un'altra tabella chiamata `ntk_from_XXX` con identificativo `YYY`,
dove `XXX` è il MAC address del sistema vicino, `YYY` è il primo identificativo libero nel
file `/etc/iproute2/rt_tables`. Questa tabella conterrà rotte da esaminare solo per i pacchetti da
inoltrare che ci sono pervenuti attraverso questo arco. Il programma quindi aggiunge una regola che
dice di guardare la tabella `ntk_from_XXX` se il pacchetto da trasmettere è marcato con il numero
`YYY`. Inoltre istruisce il kernel di marcare con il numero `YYY` i pacchetti che hanno `XXX` come MAC di provenienza.

Anche le varie tabelle `ntk_from_XXX` conterranno solo rotte di pertinenza della rete Netsukuku, cioè
quelle con destinazione nello spazio 10.0.0.0/8.

Inoltre, sia la tabella `ntk` sia le varie `ntk_from_XXX` conterranno la rotta "unreachable 10.0.0.0/8".
Questa rotta verrà presa in esame solo se un pacchetto ha una destinazione all'interno dello spazio
di Netsukuku, ma per tale destinazione non esistono altre rotte valide con una classe più restrittiva.
In altre parole, una destinazione per la quale non si conosce nessun percorso. Questa particolare rotta dice
che il pacchetto non potrà giungere a destinazione e il suo mittente ne va informato.

Sulla base degli eventi segnalati dal modulo QSPN, e se necessario richiamando i suoi metodi pubblici, il
programma *qspnclient* popola e mantiene le rotte nelle tabelle `ntk` e `ntk_from_XXX`. I percorsi
segnalati dal modulo QSPN contengono sempre un arco che parte dal sistema corrente come passo iniziale e da tale arco
si può risalire all'indirizzo di scheda del vicino. Le rotte nelle tabelle `ntk` e `ntk_from_XXX` infatti
devono avere come campo gateway (gw) l'indirizzo di scheda del vicino, non il suo indirizzo Netsukuku.

Ogni destinazione nota ad una identità è un g-nodo. Per ogni destinazione il programma *qspnclient*
sceglie (per ogni tabella come descritto sopra) il miglior percorso.

Per ogni percorso scelto dal programma *qspnclient* per entrare in una tabella, in realtà il programma
inserisce nella tabella un numero di rotte pari al numero di indirizzi IP che la mappatura di
cui abbiamo parlato sopra associa ad un indirizzo Netsukuku *reale*. Sia *i* il livello del g-nodo
destinazione del percorso. Queste sono le rotte:

*   Indirizzo IP globale del g-nodo.
*   Indirizzo IP globale anonimizzante del g-nodo.
*   Per ogni valore *t* da *i* + 1 a *l* - 1:
    *   Indirizzo IP interno al livello *t* del g-nodo.

Quando il programma ha finito di usare una tabella (ad esempio se un arco che conosceva non è più presente,
oppure se il programma termina) svuota la tabella, poi rimuove la regola, poi rimuove il record
relativo dal file `/etc/iproute2/rt_tables`.

