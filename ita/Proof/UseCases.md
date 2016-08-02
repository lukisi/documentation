# Proof of concept - Casi d'uso - Pagina 1

Nel presente documento facciamo un esempio di una rete di nodi che si va formando. Esaminiamo quali
operazioni ciascun sistema deve fare in risposta ai vari eventi che rileva il programma *qspnclient*
per il corretto funzionamento della rete Netsukuku.  
Per una più facile lettura, questa sequenza di eventi e operazioni è stata divisa in più pagine.

1.  [Convenzioni](#Convenzioni)
1.  [Prime operazioni](UseCases2.md#Prime_operazioni)
1.  [Ingresso di un altro singolo nodo nella nostra rete](UseCases2.md#Ingresso_altro_nodo)
1.  [Ingresso come g-nodo in un'altra rete](UseCases3.md#Ingresso_gnodo_altra_rete)
1.  [Ricezione di un ETP che apporta variazioni alla mappa](UseCases3.md#Elaborazione_etp)
1.  [Negli altri sistemi](UseCases4.md#Altri_sistemi)

## <a name="Convenzioni"></a>Convenzioni

Al fine di semplificare, assumiamo da subito alcuni dati della rete che si andrà delineando con questi
scenari.

I sistemi che fanno parte di questi esempi sono 7: *𝛼*, *𝛽*, *𝛾*, *𝛿*, *𝜇*, *𝜀*, *𝜆*.

Tutti i sistemi adottano questa comune topologia di rete:

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
entr04_eth1  00:16:3E:78:18:0B   169.254.202.128
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

**𝛼<sub>0</sub>** ha indirizzo 3·0·1·0 in *G<sub>𝛼</sub>*.

**𝛽<sub>0</sub>** ha indirizzo 1·0·1·0 in *G<sub>𝛽</sub>*.

**𝛾<sub>0</sub>** ha indirizzo 2·1·1·0 in *G<sub>𝛾</sub>*.

**𝛿<sub>0</sub>** ha indirizzo 3·1·0·1 in *G<sub>𝛿</sub>*.

**𝜇<sub>0</sub>** ha indirizzo 1·0·1·1 in *G<sub>𝜇</sub>*.

### Dettagli delle migrazioni/ingressi

**entr01**

Il nodo *𝜇<sub>0</sub>* era da solo e aveva indirizzo 1·0·1·1 in *G<sub>𝜇</sub>*. Con questa operazione
di ingresso *𝜇<sub>0</sub>* assume indirizzo *di connettività* 1·0·1·2 in *G<sub>𝜇</sub>*. Temporaneamente
*𝜇<sub>1</sub>* assume indirizzo *virtuale* 3·1·0·2 in *G<sub>𝛿</sub>*. Dopo poco *𝜇<sub>1</sub>* assume
indirizzo 3·1·0·0 in *G<sub>𝛿</sub>*.  
Naturalmente, dopo poco *𝜇<sub>0</sub>* viene dismesso. La verifica con `check_connectivity` avrebbe dato
esito positivo perché il nodo costituiva tutta la rete. Inoltre trattandosi di un ingresso in altra rete
la verifica non viene nemmeno fatta.

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

**entr04**

Il nodo *𝛼<sub>0</sub>* era da solo e aveva indirizzo 3·0·1·0 in *G<sub>𝛼</sub>*. Con questa operazione
il singolo nodo abbandona la vecchia rete e entra nella nuova come nuovo g-nodo di livello 2. Per questo
*𝛼<sub>0</sub>* assume indirizzo *di connettività* 3·0·1·2 in *G<sub>𝛼</sub>*. Temporaneamente
*𝛼<sub>1</sub>* assume indirizzo *virtuale* 2·2·1·0 in *G<sub>𝛾</sub>*. Dopo poco *𝛼<sub>1</sub>* assume
indirizzo 2·0·1·0 in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝛼<sub>0</sub>* viene dismesso.

[Pagina seguente](UseCases2.md)
