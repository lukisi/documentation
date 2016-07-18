# Proof of concept - Casi d'uso - Parte 2

## <a name="Prime_operazioni"></a>Prime operazioni

Prendiamo in esame quello che avviene nel sistema *𝛿*, partendo dal momento in cui avvia
il programma *qspnclient*. Assumiamo che la sua prima identità principale, *𝛿<sub>0</sub>*,
ha indirizzo Netsukuku 3·1·0·1. La sua rete iniziale (cioè l'identificativo della rete che si trova nel
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
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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
(echo; echo "250 ntk_from_00:16:3E:2D:8D:DE # xxx_table_ntk_from_00:16:3E:2D:8D:DE_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
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

