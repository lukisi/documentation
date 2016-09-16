# Proof of concept - Eventi - Pagina 2

[Pagina precedente](Eventi.md)

## Ingresso di un singolo nodo (rete a sè) in una diversa rete

Prendiamo ad esempio i valori dell'ingresso di *𝛽* in *G<sub>𝛾</sub>*.

Questo ingresso avviene per via di un nuovo arco rilevato dal modulo Neighborhood tra i sistemi
*𝛽* e *𝛾*.

Il nodo *𝛽<sub>0</sub>* era da solo e aveva indirizzo 1·0·1·0 in *G<sub>𝛽</sub>*. Con questa operazione
di ingresso *𝛽<sub>0</sub>* assume indirizzo *di connettività* 1·0·1·2 in *G<sub>𝛽</sub>*. Temporaneamente
*𝛽<sub>1</sub>* assume indirizzo *virtuale* 2·1·1·2 in *G<sub>𝛾</sub>*. Dopo poco *𝛽<sub>1</sub>* assume
indirizzo 2·1·1·1 in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝛽<sub>0</sub>* viene dismesso.

**sistema 𝛽**
```
eth1         00:16:3E:EC:A3:E1   169.254.96.141
entr02_eth1  00:16:3E:8E:91:B9   169.254.215.29
```

**sistema 𝛾**
```
eth1         00:16:3E:5B:78:D5   169.254.94.223
```

### Formazione dell'arco

**sistema 𝛽**
```
ip route add 169.254.94.223 dev eth1 src 169.254.96.141
```

Questa sequenza di operazioni è eseguita dal modulo Neighborhood (attraverso un delegato fornito dal
programma **qspnclient**) quando l'interfaccia di rete di un altro sistema è raggiungibile.
La stessa cosa avviene nel sistema opposto:

**sistema 𝛾**
```
ip route add 169.254.96.141 dev eth1 src 169.254.94.223
```

### Spostamento vecchia identità in nuovo network namespace

**TODO**

[Pagina seguente](Eventi3.md)
