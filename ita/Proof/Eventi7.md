# Proof of concept - Eventi - Pagina 7

[Pagina precedente](Eventi6.md)

## Migrazione di un singolo nodo

Nel trattare [questo](UseCases12.md) caso d'uso, abbiamo descritto la migrazione di *𝛽* (che aveva
indirizzo 1·1·1) dal g-nodo 1·1· al g-nodo 0·1· (con indirizzo 0·1·1).

Ricordiamo le informazioni salienti di questo ingresso.

**migr01**

Le operazioni **migr01** e **entr05** sono collegate.  
Il singolo nodo *𝛽<sub>1</sub>* aveva indirizzo 2·1·1·1 in *G<sub>𝛾</sub>*. Con questa operazione
di migrazione *𝛽<sub>1</sub>* assume indirizzo *di connettività* 2·1·1·3. Temporaneamente
*𝛽<sub>2</sub>* assume indirizzo *virtuale* 2·0·1·2. Dopo poco *𝛽<sub>2</sub>* assume
indirizzo 2·0·1·1.  
L'identità *𝛽<sub>1</sub>* non viene dismessa finché una verifica con `check_connectivity` non darà
esito positivo.

**entr05**

Il nodo *𝜀<sub>0</sub>* era da solo e aveva indirizzo 3·1·0·0 in *G<sub>𝜀</sub>*. Con questa operazione
di ingresso *𝜀<sub>0</sub>* assume indirizzo *di connettività* 3·1·0·2 in *G<sub>𝜀</sub>*. Temporaneamente
*𝜀<sub>1</sub>* assume indirizzo *virtuale* 2·1·1·2 in *G<sub>𝛾</sub>*. Dopo poco *𝜀<sub>1</sub>* assume
indirizzo 2·1·1·1 in *G<sub>𝛾</sub>*. Naturalmente, dopo poco *𝜀<sub>0</sub>* viene dismesso.

----

Nella trattazione del caso d'uso avevamo accennato al fatto che i dialoghi tra
vicini per scegliere le nuove posizioni e coordinare le operazioni vengono fatte inizialmente
(dopo la realizzazione dell'arco tra *𝜀* e *𝛽*) tramite meccanismi estranei ai moduli di cui stiamo
trattando in questa proof-of-concept. Diciamo in dettaglio come questi dialoghi sono simulati nel
programma **qspnclient**.

L'utente nel sistema *𝜀* dice al **qspnclient** che:
*   l'identità *𝜀<sub>0</sub>* deve produrre una copia (sia essa *𝜀<sub>1</sub>*) che faccia
    ingresso in *G<sub>𝛾</sub>* in 1·1·.
*   l'identità *𝜀<sub>1</sub>* deve prendere temporaneamente il virtuale 1·1·2.
*   è stata individuata una migration path nella quale l'ultimo passaggio ha un dato identificativo
    (che indichiamo con *m<sub>𝛽</sub>*) e libererà per *𝜀<sub>1</sub>* l'indirizzo 1·1·1.

L'utente nel sistema *𝛽* dice al **qspnclient** che:
*   l'identità *𝛽<sub>1</sub>* deve produrre una copia (sia essa *𝛽<sub>2</sub>*) che faccia
    migrazione in 0·1·.
*   l'identità *𝛽<sub>2</sub>* deve prendere temporaneamente il virtuale 0·1·2.
*   è immediatamente disponibile per *𝛽<sub>2</sub>* l'indirizzo 0·1·1.

L'utente nel sistema *𝜀* dice al **qspnclient** che:
*   la migrazione *m<sub>𝛽</sub>* (di cui era in attesa *𝜀<sub>1</sub>*) è stata completata.

----

### Formazione dell'arco

**sistema 𝜀**
```
ip route add 169.254.96.141 dev eth1 src 169.254.163.36
```

**sistema 𝛽**
```
ip route add 169.254.163.36 dev eth1 src 169.254.96.141
```

Sequenza di operazioni eseguita dal modulo Neighborhood.

### entr05: Spostamento vecchia identità in nuovo network namespace


[Pagina seguente](Eventi8.md)
