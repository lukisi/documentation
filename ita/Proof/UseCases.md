# Proof of concept - Casi d'uso

1.  [Topologia della rete](#Topologia_della_rete)
1.  [Altre convenzioni](#Convenzioni)
1.  [Prime operazioni](#Prime_operazioni)
1.  [Ingresso di un altro singolo nodo nella nostra rete](#Ingresso_altro_nodo)
1.  [Ingresso come g-nodo in un'altra rete](#Ingresso_gnodo_altra_rete)

Attraverso un esame dettagliato dei possibili scenari in cui il programma si trova ad operare
e delle operazioni che devono essere fatte sulle tabelle di routing del sistema per il corretto funzionamento
della rete, giungeremo ad osservare quali operazioni e in quali momenti il programma deve fare
in risposta agli eventi che rileva.

## <a name="Topologia_della_rete"></a>Topologia della rete

Assumiamo di operare con reti Netsukuku che hanno questa comune topologia: 4·2·2·2.

```
Topologia 4·2·2·2

Globali:
10.0.0.0 ... 10.0.0.31
Anonimizzanti:
10.0.0.64 ... 10.0.0.95
Interni al livello 3
10.0.0.56 ... 10.0.0.63
Interni al livello 2
10.0.0.48 ... 10.0.0.51
Interni al livello 1
10.0.0.40 ... 10.0.0.41
```

## <a name="Convenzioni"></a>Altre convenzioni

Al fine di semplificare, assumiamo da subito alcuni dati della rete che si andrà delineando con questi
scenari.

I sistemi che fanno parte di questi esempi sono 7: *𝛼*, *𝛽*, *𝛾*, *𝛿*, *𝜇*, *𝜀*, *𝜆*.

Tutti i sistemi ammettono di essere usati come passi intermedi anonimizzanti e anche di essere
contattati in forma anonima.

Tutti i sistemi hanno una unica interfaccia di rete che chiamiamo "eth1".
Tale interfaccia è di tipo wireless: con questa precisazione si intende sottolineare che, ad esempio, il sistema
*𝛽* con la sua interfaccia "eth1" è direttamente collegato al sistema *𝛼* e al sistema *𝛾*. Invece il sistema *𝛼*
non è direttamente collegato al sistema *𝛾*.

Forniamo un elenco qui, per un comodo riferimento, degli indirizzi MAC delle interfacce di rete reali e
pseudo e degli indirizzi IP linklocal che assumiamo vengano scelti dai moduli Neighborhood e Identities.

I nomi che daremo ai network namespace temporanei e alle relative pseudo-interfacce saranno anche
essi indicati qui sotto e saranno collegati alla migrazione (o ingresso in altra rete) da cui scaturiscono,
sebbene in realtà essi vengono assegnati dal modulo Identities che li compone di un progressivo nel singolo
sistema (e.g. ntkv0, ntkv1, ...).

**sistema 𝛼**
```
eth1         00:16:3E:FD:E2:AA   169.254.69.30
```

**sistema 𝛽**
```
eth1         00:16:3E:EC:A3:E1   169.254.96.141
entr02_eth1  00:16:3E:8E:91:B9   169.254.215.29
ntkv0_eth1   00:16:3E:EE:AF:D1   169.254.27.218
ntkv1_eth1   00:16:3E:BD:34:98   169.254.42.4
```

**sistema 𝛾**
```
eth1         00:16:3E:5B:78:D5   169.254.94.223
ntkv1_eth1   00:16:3E:AF:4C:2A   169.254.24.198
```

**sistema 𝛿**
```
eth1         00:16:3E:1A:C4:45   169.254.253.216
entr03_eth1  00:16:3E:B9:77:80   169.254.83.167
```

**sistema 𝜀**
```
eth1         00:16:3E:3C:14:33   169.254.163.36
ntkv1_eth1   00:16:3E:3B:9F:45   169.254.241.153
```

**sistema 𝜆**
```
eth1         00:16:3E:06:3E:90   169.254.109.22
```

**sistema 𝜇**
```
eth1         00:16:3E:2D:8D:DE   169.254.119.176
entr01_eth1  00:16:3E:71:33:12   169.254.101.161
entr03_eth1  00:16:3E:DF:23:F5   169.254.242.91
```

### Dettagli iniziali

**𝛽<sub>0</sub>** ha indirizzo 1·0·1·0 in *G<sub>𝛽</sub>*.

**𝛾<sub>0</sub>** ha indirizzo 2·1·1·0 in *G<sub>𝛾</sub>*.

**𝛿<sub>0</sub>** ha indirizzo 3·1·0·1 in *G<sub>𝛿</sub>*.

**𝜇<sub>0</sub>** ha indirizzo 1·0·1·1 in *G<sub>𝜇</sub>*.

### Dettagli delle migrazioni/ingressi

**entr01**

Il nodo *𝜇<sub>0</sub>* era da solo e aveva indirizzo 1·0·1·1 in *G<sub>𝜇</sub>*. Con questa operazione
di ingresso *𝜇<sub>0</sub>* assume indirizzo *di connettività* 1·0·1·2 in *G<sub>𝜇</sub>*. Temporaneamente
*𝜇<sub>1</sub>* assume indirizzo *virtuale* 3·1·0·2 in *G<sub>𝛿</sub>*. Dopo poco *𝜇<sub>1</sub>* assume
indirizzo 3·1·0·0 in *G<sub>𝛿</sub>*. Naturalmente, dopo poco *𝜇<sub>0</sub>* viene dismesso.

**entr02**

Il nodo *𝛽<sub>0</sub>* era da solo e aveva indirizzo 1·0·1·0 in *G<sub>𝛽</sub>*. Con questa operazione
di ingresso *𝛽<sub>0</sub>* assume indirizzo *di connettività* 1·0·1·2 in *G<sub>𝛽</sub>*. Temporaneamente
*𝛽<sub>1</sub>* assume indirizzo *virtuale* 2·1·1·2 in *G<sub>𝛾</sub>*. Dopo poco *𝛽<sub>1</sub>* assume
indirizzo 2·1·1·1 in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝛽<sub>0</sub>* viene dismesso.

**entr03**

Il g-nodo *𝜒* di livello 1 e di indirizzo Netsukuku 3·1·0·, che comprende *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>*,
costituiva l'intera rete *G<sub>𝛿</sub>*. Con questa operazione di ingresso si forma il g-nodo isomorfo
*𝜒'* costituito dalle nuove identità *𝛿<sub>1</sub>* e *𝜇<sub>2</sub>*. Il g-nodo
*𝜒* assume indirizzo *di connettività* 3·1·2· in *G<sub>𝛿</sub>*. Temporaneamente
*𝜒'* assume indirizzo *virtuale* 2·1·2· in *G<sub>𝛾</sub>*. Dopo poco *𝜒'* assume
indirizzo 2·1·0· in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝜒* viene dismesso.

## <a name="Prime_operazioni"></a>Prime operazioni

Partiamo dal sistema *𝛿* che ha una identità principale *𝛿<sub>0</sub>* che ha indirizzo Netsukuku 3·1·0·1
in una rete con topologia 4·2·2·2. Tale rete (cioè l'identificativo della rete che si trova nel
fingerprint al livello 4) la chiamiamo *G<sub>𝛿</sub>*.

```
Mio indirizzo 3·1·0·1.
     globale
      10.0.0.29
     anonimizzante
      10.0.0.93
     interno al mio g-nodo di livello 3
      10.0.0.61
     interno al mio g-nodo di livello 2
      10.0.0.49
     interno al mio g-nodo di livello 1
      10.0.0.41

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 1·
     globale
      10.0.0.8/29
     anonimizzante
      10.0.0.72/29
 2·
     globale
      10.0.0.16/29
     anonimizzante
      10.0.0.80/29
 3·0·
     globale
      10.0.0.24/30
     anonimizzante
      10.0.0.88/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 3·1·1·
     globale
      10.0.0.30/31
     anonimizzante
      10.0.0.94/31
     interno al mio g-nodo di livello 3
      10.0.0.62/31
     interno al mio g-nodo di livello 2
      10.0.0.50/31
 3·1·0·0
     globale
      10.0.0.28/32
     anonimizzante
      10.0.0.92/32
     interno al mio g-nodo di livello 3
      10.0.0.60/32
     interno al mio g-nodo di livello 2
      10.0.0.48/32
     interno al mio g-nodo di livello 1
      10.0.0.40/32
```

Fra i compiti del modulo Neighborhood, esso assegna all'interfaccia `eth1` del sistema *𝛿* un indirizzo IP
linklocal.

**sistema 𝛿**
```
ip link set dev eth1 address 00:16:3E:1A:C4:45
ip link set dev eth1 up
ip address add 169.254.253.216 dev eth1
```

Il programma *qspnclient* invece ha il compito, fin dall'inizio, di assegnare alla stessa interfaccia gli
indirizzi IP che sono da associare all'identità principale *𝛿<sub>0</sub>*. Come si spiega nel documento di analisi,
in questo caso abbiamo l'indirizzo IP globale, l'indirizzo IP anonimizzante, l'indirizzo IP interno al g-nodo di
livello 3, di livello 2 e di livello 1.

**sistema 𝛿**
```
ip address add 10.0.0.29 dev eth1
ip address add 10.0.0.93 dev eth1
ip address add 10.0.0.61 dev eth1
ip address add 10.0.0.49 dev eth1
ip address add 10.0.0.41 dev eth1
```

Inoltre il programma *qspnclient* ha da subito il compito, sempre con riferimento alla sua identità principale *𝛿<sub>0</sub>*,
di creare nel network namespace default una tabella `ntk` e di aggiungere una regola che la referenzia. In tale
tabella deve anche mettere tutte le possibili destinazioni, inizialmente in stato "unreachable".

**sistema 𝛿**
```
/etc/iproute2/rt_tables: add table 251: ntk
ip rule add table ntk
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.24/30 table ntk
ip route add unreachable 10.0.0.88/30 table ntk
ip route add unreachable 10.0.0.56/30 table ntk
ip route add unreachable 10.0.0.30/31 table ntk
ip route add unreachable 10.0.0.94/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
ip route add unreachable 10.0.0.28/32 table ntk
ip route add unreachable 10.0.0.92/32 table ntk
ip route add unreachable 10.0.0.60/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
```

Inoltre il programma *qspnclient* ha da subito il compito, sempre con riferimento alla sua identità
principale *𝛿<sub>0</sub>*, siccome decide di prestarsi all'anonimizzazione dei pacchetti IP che inoltra,
di istruire il kernel a questo scopo.

**sistema 𝛿**
```
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.29
```

Trattandosi della prima identità nel sistema, quasi immediatamente il QspnManager associato a
*𝛿<sub>0</sub>* notifica il segnale `bootstrap_complete`. A fronte di esso nel sistema
verranno dati, sebbene superflui, questi comandi:

**sistema 𝛿**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/30 table ntk
ip route change unreachable 10.0.0.88/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.30/31 table ntk
ip route change unreachable 10.0.0.94/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change unreachable 10.0.0.28/32 table ntk
ip route change unreachable 10.0.0.92/32 table ntk
ip route change unreachable 10.0.0.60/32 table ntk
ip route change unreachable 10.0.0.48/32 table ntk
ip route change unreachable 10.0.0.40/32 table ntk
```

## <a name="Ingresso_altro_nodo"></a>Ingresso di un altro singolo nodo nella nostra rete

Ora assumiamo che il sistema *𝜇* giunga a distanza di rilevamento con la sua interfaccia di rete
per l'interfaccia di rete del sistema *𝛿*.

Fra i compiti del modulo Neighborhood, esso aggiunge nel network namespace default la rotta diretta verso
l'indirizzo IP linklocal del vicino per ogni arco che lo stesso modulo ha realizzato.

**sistema 𝛿**
```
ip route add 169.254.119.176 dev eth1 src 169.254.253.216
```

Assumiamo che il sistema *𝜇* abbia solo una identità *𝜇<sub>0</sub>* che si trova in una diversa rete
*G<sub>𝜇</sub>*.

Il modulo Identities ha creato l'arco-identità principale, cioè quello che collega le due identità
*𝛿<sub>0</sub>* e *𝜇<sub>0</sub>*, senza per questo aggiungere alcuna rotta, perché per tale arco-identità
la rotta è stata aggiunta dal modulo Neighborhood nel network namespace default del sistema *𝛿*.

Ora assumiamo che *𝜇<sub>0</sub>* decide di entrare in *G<sub>𝛿</sub>*. Per essere precisi, il sistema *𝜇* decide di
costruire una nuova identità *𝜇<sub>1</sub>* partendo da *𝜇<sub>0</sub>*. L'identità *𝜇<sub>0</sub>* verrà temporaneamente
spostata sul network namespace "entr01" del sistema *𝜇*. Poi *𝜇<sub>1</sub>* farà ingresso nella rete *G<sub>𝛿</sub>*.

Quando il sistema *𝜇* crea la nuova identità, il suo modulo Identities dialoga con il modulo del sistema
*𝛿* per aggiungere l'arco-identità *𝛿<sub>0</sub>-𝜇<sub>1</sub>* e modificare i valori (peer_mac e
peer_linklocal) dell'arco-identità *𝛿<sub>0</sub>-𝜇<sub>0</sub>*. Di fatto, questo comporta che il modulo Identities
nel sistema *𝛿* aggiunge la nuova rotta, sempre nel network namespace default gestito da *𝛿<sub>0</sub>*, verso il nuovo indirizzo
linklocal assunto da *𝜇<sub>0</sub>*.

**sistema 𝛿**
```
ip route add 169.254.101.161 dev eth1 src 169.254.253.216
```

Ora il sistema *𝜇* fa entrare *𝜇<sub>1</sub>* in *G<sub>𝛿</sub>* e, contemporaneamente, il sistema *𝛿* comunica
alla sua identità *𝛿<sub>0</sub>* che sull'arco-identità *𝛿<sub>0</sub>-𝜇<sub>1</sub>* va costruito un QspnArc.

Questo nuovo QspnArc che viene comunicato al modulo QSPN del sistema *𝛿* per l'identità *𝛿<sub>0</sub>*, inizialmente
non comporta operazioni sulle tabelle di routing, fino a quando non si riceve il primo ETP attraverso questo
arco e così si scopre l'indirizzo Netsukuku di questo vicino.

Dopo un po' di tempo il nodo *𝛿<sub>0</sub>* riceverà un ETP dal QspnArc *𝛿<sub>0</sub>-𝜇<sub>1</sub>* e con esso
aggiornerà l'indirizzo Netsukuku di questo vicino. Questo deve produrre due macro-operazioni nel sistema *𝛿*: la creazione di
una tabella per i pacchetti IP ricevuti dall'arco *𝛿<sub>0</sub>-𝜇<sub>1</sub>* e l'aggiornamento (su tutte le tabelle) delle
rotte che adesso possono avere come gateway l'arco *𝛿<sub>0</sub>-𝜇<sub>1</sub>*.

Assumiamo che quando arriva questo primo ETP l'indirizzo Netsukuku di *𝜇<sub>1</sub>* sia ancora quello
temporaneo *virtuale* 3·1·0·2.

**sistema 𝛿**
```
/etc/iproute2/rt_tables: add table 250: ntk_from_00:16:3E:2D:8D:DE
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:2D:8D:DE -j MARK --set-mark 250
ip rule add fwmark 250 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
```

**sistema 𝛿**
```
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/30 table ntk
ip route change unreachable 10.0.0.88/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.30/31 table ntk
ip route change unreachable 10.0.0.94/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change unreachable 10.0.0.28/32 table ntk
ip route change unreachable 10.0.0.92/32 table ntk
ip route change unreachable 10.0.0.60/32 table ntk
ip route change unreachable 10.0.0.48/32 table ntk
ip route change unreachable 10.0.0.40/32 table ntk
```

Poi *𝜇<sub>1</sub>* assume l'indirizzo 3·1·0·0 e comunica un nuovo ETP che giunge a *𝛿<sub>0</sub>*.

**sistema 𝛿**
```
ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/30 table ntk
ip route change unreachable 10.0.0.88/30 table ntk
ip route change unreachable 10.0.0.56/30 table ntk
ip route change unreachable 10.0.0.30/31 table ntk
ip route change unreachable 10.0.0.94/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change 10.0.0.28/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.29
ip route change 10.0.0.92/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.29
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41
```

Poi il sistema *𝜇* rimuoverà l'identità *𝜇<sub>0</sub>* con tutti i suoi archi-identità.
La cosa verrà comunicata al modulo Identities del sistema *𝛿* per via dell'arco-identità
*𝛿<sub>0</sub>-𝜇<sub>0</sub>*. Questo produce la rimozione della rotta.

**sistema 𝛿**
```
ip route del 169.254.101.161 dev eth1 src 169.254.253.216
```

## <a name="Ingresso_gnodo_altra_rete"></a>Ingresso come g-nodo in un'altra rete

Ora assumiamo che il sistema *𝛾* giunga a distanza di rilevamento con la sua interfaccia di rete
per l'interfaccia di rete del sistema *𝛿*.

Fra i compiti del modulo Neighborhood, esso aggiunge nel network namespace default la rotta diretta verso
l'indirizzo IP linklocal del vicino per ogni arco che lo stesso modulo ha realizzato.

**sistema 𝛿**
```
ip route add 169.254.94.223 dev eth1 src 169.254.253.216
```

Assumiamo che il sistema *𝛾* abbia solo una identità *𝛾<sub>0</sub>* che si trova in una diversa rete
*G<sub>𝛾</sub>*.

Il modulo Identities ha creato l'arco-identità principale, cioè quello che collega le due identità
*𝛿<sub>0</sub>* e *𝛾<sub>0</sub>*, senza per questo aggiungere alcuna rotta, perché per tale arco-identità
la rotta è stata aggiunta dal modulo Neighborhood nel network namespace default del sistema *𝛿*.

Ora assumiamo che *𝛿<sub>0</sub>* decide di entrare in *G<sub>𝛾</sub>*. Per essere precisi, il sistema *𝛿* decide di
costruire una nuova identità *𝛿<sub>1</sub>* partendo da *𝛿<sub>0</sub>*. Questa nuova identità scaturisce dalla
migrazione del g-nodo *𝜒*, di livello 1 e di indirizzo Netsukuku 3·1·0·, che comprende anche il vicino
*𝜇<sub>1</sub>*. Poi *𝛿<sub>1</sub>* farà ingresso in *G<sub>𝛾</sub>* come membro del g-nodo *𝜒'*, il quale
avrà in *G<sub>𝛾</sub>* un indirizzo Netsukuku. Per completezza prevediamo che tale indirizzo Netsukuku sia
temporaneamente *virtuale* nella sua componente di livello 1: assumiamo sia 2·1·2·.

All'inizio viene creato nel sistema *𝛿* un nuovo network namespace "entr03" e in esso viene creata
una pseudo-interfaccia "entr03_eth1" sopra l'interfaccia reale "eth1". Questo nuovo network namespace
sarà gestito da *𝛿<sub>0</sub>* mentre quello precedente (il default) verrà gestito da *𝛿<sub>1</sub>*.  
Assumiamo che il g-nodo *𝜒*, che rimane di connettività in *G<sub>𝛿</sub>*, prende l'indirizzo Netsukuku
3·1·2·. Questo non ha una diretta ripercussione negli indirizzi IP del sistema *𝛿* nel nuovo
network namespace: infatti una identità di connettività non detiene (nel suo network namespace che non è
il default) alcun indirizzo IP associato al suo indirizzo Netsukuku.

**sistema 𝛿**
```
ip netns add entr03
ip link add dev entr03_eth1 link eth1 type macvlan
ip link set dev entr03_eth1 netns entr03
ip netns exec entr03 ip link set dev entr03_eth1 address 00:16:3E:B9:77:80
ip netns exec entr03 ip link set dev entr03_eth1 up
ip netns exec entr03 ip address add 169.254.83.167 dev entr03_eth1
```

Anche nel sistema *𝜇* partendo da *𝜇<sub>1</sub>* è stata creata una nuova identità *𝜇<sub>2</sub>*.  
L'identità *𝜇<sub>1</sub>* verrà temporaneamente spostata sul network namespace "entr03" del sistema *𝜇*.
Il namespace default è gestito ora da *𝜇<sub>2</sub>*.  
Il modulo Identities del sistema *𝛿*, dal dialogo con i vicini, desume che vanno creati/modificati questi
archi-identità:

*   *𝛿<sub>0</sub>-𝜇<sub>1</sub>*.  
    Questo arco-identità vede cambiare il `peer_mac` e `peer_linklocal`. Inoltre la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝛿<sub>0</sub>*
    la rotta verso il vicino.  
    I nodi *𝛿<sub>0</sub>* e *𝜇<sub>1</sub>* fanno parte della stessa rete, quindi è prevista una tabella
    per i pacchetti IP da inoltrare che provengono da questo arco. Nel nuovo network namespace gestito da *𝛿<sub>0</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:DF:23:F5` e la relativa regola.
*   *𝛿<sub>1</sub>-𝜇<sub>2</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝛿<sub>1</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝛿<sub>1</sub>* e *𝜇<sub>2</sub>* fanno parte della stessa rete, quindi è prevista una tabella
    per i pacchetti IP da inoltrare che provengono da questo arco. Nel network namespace gestito da *𝛿<sub>1</sub>*
    la tabella `ntk_from_00:16:3E:2D:8D:DE` c'è già, e anche la relativa regola.
*   *𝛿<sub>0</sub>-𝛾<sub>0</sub>*.  
    Per questo arco-identità abbiamo che la sua identità
    di riferimento ha cambiato il suo network namespace e relativo indirizzo IP linklocal. Per questo
    il modulo Identities si occupa di aggiungere nel nuovo network namespace gestito da *𝛿<sub>0</sub>*
    la rotta verso il vicino.  
    I nodi *𝛿<sub>0</sub>* e *𝛾<sub>0</sub>* non fanno parte della stessa rete, quindi non è prevista una tabella
    `ntk_from_xxx`.
*   *𝛿<sub>1</sub>-𝛾<sub>0</sub>*.  
    Questo arco-identità è nuovo.  
    Nel network namespace gestito da *𝛿<sub>1</sub>* la relativa rotta è stata già aggiunta.  
    I nodi *𝛿<sub>1</sub>* e *𝛾<sub>0</sub>* faranno parte della stessa rete, ma solo dopo che il nodo *𝛿<sub>1</sub>*
    avrà costruito la nuova istanza di QspnManager; per fare questo il sistema *𝛿* dovrà costruire un QspnArc
    sull'arco-identità *𝛿<sub>1</sub>-𝛾<sub>0</sub>*. Questo nuovo QspnArc inizialmente non comporta operazioni sulle
    tabelle di routing, fino a quando non si riceve il primo ETP attraverso questo arco e così si scopre l'indirizzo
    Netsukuku di questo vicino. Solo a quel punto, ad esempio, nel network namespace gestito da *𝛿<sub>1</sub>*
    il programma *qspnclient* deve aggiungere la tabella `ntk_from_00:16:3E:5B:78:D5` e la relativa regola.

Il modulo Identities fa queste operazioni:

**sistema 𝛿**
```
ip netns exec entr03 ip route add 169.254.242.91 dev entr03_eth1 src 169.254.83.167
ip netns exec entr03 ip route add 169.254.94.223 dev entr03_eth1 src 169.254.83.167
```

Il programma *qspnclient* fa queste operazioni preliminari:

**sistema 𝛿**
```
ip netns exec entr03 ip rule add table ntk
/etc/iproute2/rt_tables: add table 249: ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:DF:23:F5 -j MARK --set-mark 249
ip netns exec entr03 ip rule add fwmark 249 table ntk_from_00:16:3E:DF:23:F5
```

Il programma *qspnclient* fa queste operazioni sulle rotte verso g-nodi di livello *k*, con `k < 1`, cioè
verso destinazioni interne al g-nodo che ha migrato:

*   Quelle espresse con indirizzi IP interni ad un g-nodo fino al livello 1, vanno mantenute nel network namespace
    vecchio e copiate nel network namespace nuovo.
*   Quelle espresse con indirizzi IP globali o interni ad un g-nodo di livello superiore, vanno rimosse dal
    network namespace vecchio.

**sistema 𝛿**
```
ip netns exec entr03 ip route add unreachable 10.0.0.40/32 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5

ip route del 10.0.0.28/32 table ntk
ip route del 10.0.0.92/32 table ntk
ip route del 10.0.0.60/32 table ntk
ip route del 10.0.0.48/32 table ntk
ip route del 10.0.0.28/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.92/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
```

Il programma *qspnclient* fa queste operazioni sulle rotte verso g-nodi di livello *k*, con `k ≥ 1`, cioè
verso destinazioni esterne al g-nodo che ha migrato:

*   Tutte (siano esse espresse con indirizzi IP globali o interni ad un g-nodo) vanno spostate dal network namespace
    vecchio al network namespace nuovo.

**sistema 𝛿**
```
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk
ip netns exec entr03 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5

ip route del 10.0.0.0/29 table ntk
ip route del 10.0.0.64/29 table ntk
ip route del 10.0.0.8/29 table ntk
ip route del 10.0.0.72/29 table ntk
ip route del 10.0.0.16/29 table ntk
ip route del 10.0.0.80/29 table ntk
ip route del 10.0.0.24/30 table ntk
ip route del 10.0.0.88/30 table ntk
ip route del 10.0.0.56/30 table ntk
ip route del 10.0.0.30/31 table ntk
ip route del 10.0.0.94/31 table ntk
ip route del 10.0.0.62/31 table ntk
ip route del 10.0.0.50/31 table ntk
ip route del 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.16/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.80/29 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.24/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.88/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.30/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.94/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route del 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
```

Il programma *qspnclient* rimuove l'indirizzo globale e gli indirizzi interni ai propri g-nodi di livello
maggiore di 1 dal network namespace vecchio.

**sistema 𝛿**
```
ip address del 10.0.0.29/32 dev eth1
ip address del 10.0.0.93/32 dev eth1
ip address del 10.0.0.61/32 dev eth1
ip address del 10.0.0.49/32 dev eth1
```

Il programma *qspnclient* aggiorna le rotte nel nuovo network namespace sulla base dei migliori percorsi
noti alla vecchia identità *𝛿<sub>0</sub>*.

**sistema 𝛿**
```
ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk
ip netns exec entr03 ip route change 10.0.0.40/32 table ntk via 169.254.242.91 dev entr03_eth1

ip netns exec entr03 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.16/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.80/29 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.24/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.88/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.30/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.94/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:DF:23:F5
ip netns exec entr03 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:DF:23:F5
```

Poi il sistema *𝛿* per la nuova identità *𝛿<sub>1</sub>* istanzia un QspnManager con il
costruttore `enter_net` passandogli un QspnArc per l'arco-identità *𝛿<sub>1</sub>-𝜇<sub>2</sub>*
e uno per l'arco-identità *𝛿<sub>1</sub>-𝛾<sub>0</sub>*. L'indirizzo Netsukuku del nodo
*𝛿<sub>1</sub>* sarà 2·1·2·1: cioè l'indirizzo di *𝜒'* con la parte finale che era già di
*𝛿<sub>0</sub>* in *𝜒*.

Siccome la nuova identità *𝛿<sub>1</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku. Come si spiega nel documento di analisi,
in questo caso abbiamo solo l'indirizzo IP interno al g-nodo di livello 1.  
Deve però tenere presente che alcuni di essi possono essere rimasti assegnati dal
precedente detentore del network namespace default. In questo caso tutti; quindi non ci
sono operazioni da fare.

Inoltre il programma *qspnclient* deve assicurarsi che nel network namespace della nuova identità *𝛿<sub>1</sub>*
siano presenti le regole per la consultazione della tabella `ntk` e delle tabelle `ntk_from_xxx` associate agli
archi-identità di *𝛿<sub>1</sub>* per i quali conosciamo già l'indirizzo Netsukuku del peer. E non altre regole.

Deve poi aggiungere in queste tabelle (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝛿<sub>1</sub>*, inizialmente in stato "unreachable". Deve anche fare in modo che le operazioni di aggiunta
delle destinazioni vengano completate tutte prima di elaborare (ad esempio dopo il segnale `bootstrap_complete`) operazioni
di aggiornamento delle rotte.

```
Mio indirizzo 2·1·2·1.
     globale
      N/A
     anonimizzante
      N/A
     interno al mio g-nodo di livello 3
      N/A
     interno al mio g-nodo di livello 2
      N/A
     interno al mio g-nodo di livello 1
      10.0.0.41

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 1·
     globale
      10.0.0.8/29
     anonimizzante
      10.0.0.72/29
 3·
     globale
      10.0.0.24/29
     anonimizzante
      10.0.0.88/29
 2·0·
     globale
      10.0.0.16/30
     anonimizzante
      10.0.0.80/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 2·1·1·
     globale
      10.0.0.22/31
     anonimizzante
      10.0.0.86/31
     interno al mio g-nodo di livello 3
      10.0.0.62/31
     interno al mio g-nodo di livello 2
      10.0.0.50/31
 2·1·2·0
     globale
      N/A
     anonimizzante
      N/A
     interno al mio g-nodo di livello 3
      N/A
     interno al mio g-nodo di livello 2
      N/A
     interno al mio g-nodo di livello 1
      10.0.0.40/32
```

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
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
```

Dopo un po' di tempo avverranno 2 eventi per *𝛿<sub>1</sub>*, dei quali non possiamo a priori
sapere l'ordine temporale:

*   il QspnManager di *𝛿<sub>1</sub>* emetterà il segnale di `bootstrap_complete` dopo aver ricevuto un ETP dal
    vicino *𝛾<sub>0</sub>*.
*   il programma *qspnclient* nel sistema *𝛿* viene notificato che l'indirizzo Netsukuku di *𝜒'* in *G<sub>𝛾</sub>*, che
    temporaneamente era il *virtuale* 2·1·2·, diventa il *reale* (2·1·0·). Di conseguenza il programma comunica al
    QspnManager di *𝛿<sub>1</sub>* che ora il suo indirizzo Netsukuku è 2·1·0·1.

Assumiamo che si verifichi prima il cambio di indirizzo. Rimandiamo a dopo un esempio dell'altro caso.

Siccome la nuova identità *𝛿<sub>1</sub>* è la *principale*,
il programma *qspnclient* ora ha il compito di assegnare all'interfaccia reale nel network
namespace default gli indirizzi IP che sono da associare al
suo indirizzo Netsukuku. Come si spiega nel documento di analisi,
in questo caso abbiamo l'indirizzo IP globale, l'indirizzo IP anonimizzante, l'indirizzo IP interno al g-nodo di
livello 3, di livello 2 e di livello 1.  
Deve però tenere presente che alcuni di essi possono essere già stati assegnati dal
precedente detentore del network namespace default o dallo stesso nella fase in cui aveva indirizzo virtuale.
In questo caso l'indirizzo IP interno al g-nodo di livello 1.

Deve poi aggiungere in queste tabelle (se non ci sono già) tutte le possibili destinazioni previste dall'indirizzo
di *𝛿<sub>1</sub>*, inizialmente in stato "unreachable".

```
Mio indirizzo 2·1·0·1.
     globale
      10.0.0.21
     anonimizzante
      10.0.0.85
     interno al mio g-nodo di livello 3
      10.0.0.61
     interno al mio g-nodo di livello 2
      10.0.0.49
     interno al mio g-nodo di livello 1
      10.0.0.41

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 1·
     globale
      10.0.0.8/29
     anonimizzante
      10.0.0.72/29
 3·
     globale
      10.0.0.24/29
     anonimizzante
      10.0.0.88/29
 2·0·
     globale
      10.0.0.16/30
     anonimizzante
      10.0.0.80/30
     interno al mio g-nodo di livello 3
      10.0.0.56/30
 2·1·1·
     globale
      10.0.0.22/31
     anonimizzante
      10.0.0.86/31
     interno al mio g-nodo di livello 3
      10.0.0.62/31
     interno al mio g-nodo di livello 2
      10.0.0.50/31
 2·1·0·0
     globale
      10.0.0.20/32
     anonimizzante
      10.0.0.84/32
     interno al mio g-nodo di livello 3
      10.0.0.60/32
     interno al mio g-nodo di livello 2
      10.0.0.48/32
     interno al mio g-nodo di livello 1
      10.0.0.40/32
```

**sistema 𝛿**
```
ip address add 10.0.0.21 dev eth1
ip address add 10.0.0.85 dev eth1
ip address add 10.0.0.61 dev eth1
ip address add 10.0.0.49 dev eth1
```

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
ip route add unreachable 10.0.0.22/31 table ntk
ip route add unreachable 10.0.0.86/31 table ntk
ip route add unreachable 10.0.0.62/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk
ip route add unreachable 10.0.0.20/32 table ntk
ip route add unreachable 10.0.0.84/32 table ntk
ip route add unreachable 10.0.0.60/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
```

Ora il QspnManager di *𝛿<sub>1</sub>* riceve un ETP dal vicino *𝛾<sub>0</sub>*. A questo punto
conosce l'indirizzo Netsukuku del peer sull'arco *𝛿<sub>1</sub>-𝛾<sub>0</sub>*. Per questo il
programma *qspnclient* può creare la tabella `ntk_from_00:16:3E:5B:78:D5` e popolarla con tutte le
destinazioni adatte al suo indirizzo. Poi, ma solo dopo che la tabella sarà stata popolata
e aggiornata, aggiungerà la regola di guardare questa tabella per i pacchetti IP da inoltrare
ricevuti su questo arco.

**sistema 𝛿**
```
/etc/iproute2/rt_tables: add table 248: ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
```

Inoltre il QspnManager di *𝛿<sub>1</sub>* sulla base dell'ETP aggiorna la sua mappa. Assumiamo
che attraverso l'arco *𝛿<sub>1</sub>-𝛾<sub>0</sub>* scopra di poter raggiungere la destinazione
(1, 1), ossia il g-nodo 2·1·1·.

Poi il QspnManager di *𝛿<sub>1</sub>* notifica il segnale `bootstrap_complete`. Per questo
il programma *qspnclient* aggiorna tutte le rotte sulla base dei migliori percorsi noti.

**sistema 𝛿**
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
ip route change 10.0.0.22/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.86/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.62/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.50/31 table ntk via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE
ip route change 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1 src 10.0.0.49
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE
```

Alla fine di questo aggiornamento delle tabelle, siccome era rimasta in attesa la regola di guardare
la nuova tabella, il programma *qspnclient* la aggiunge.

**sistema 𝛿**
```
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 248
ip rule add fwmark 248 table ntk_from_00:16:3E:5B:78:D5
```





