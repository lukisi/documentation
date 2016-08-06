# Proof of concept - Casi d'uso - Pagina 12

[Pagina precedente](UseCases11.md)

## Migrazione di un singolo nodo

Ora assumiamo un evento che porta alla decisione di far migrare un singolo nodo da un g-nodo
ad un altro all'interno della stessa rete.

Un nuovo sistema *𝜀* viene rilevato solo dal sistema *𝛽*. Assumiamo per convenienza di non voler
costituire un nuovo g-nodo di livello 3.

Il sistema *𝜀* può fare ingresso solo in un g-nodo a cui appartiene anche *𝛽*, poiché esso è il suo unico vicino.
Il g-nodo *g<sub>1</sub>(𝛽)* è saturo. Anche *g<sub>2</sub>(𝛽)* è saturo. Anche *g<sub>3</sub>(𝛽)* è saturo.
E abbiamo escluso di costituire un nuovo g-nodo di livello 3. Si deve quindi ricorrere ad una *migration path*.

**Appunto**: quando un g-nodo di livello *i* vuole fare ingresso in una nuova rete, se tramite i suoi archi
non può trovare un g-nodo di livello *j* non saturo, con *i* + 1 ≤ *j* ≤ *i* + *k* dove *k* è un intero positivo
piccolo a piacere, allora si tenta di ricorrere ad una *migration path*. Si vuole cioè evitare di ricorrere ad una
migration path appena un singolo nodo non trova libero un g-nodo di livello 1, ma allo stesso tempo evitare
di occupare un g-nodo di livello alto per l'ingresso di un singolo nodo. Ragionamento simile per
l'ingresso di un g-nodo di livello *i*.

Si decide di far entrare il sistema *𝜀* in *g<sub>1</sub>(𝛽)* = 1·1· con indirizzo temporaneamente *virtuale*.
Si farà poi migrare il sistema *𝛽* in *g<sub>1</sub>(𝛼)* = 0·1·, ma che *𝛽* resti come virtuale in 1·1·. In questo
modo il sistema *𝜀* in 1·1· potrà assumere un indirizzo *reale*.

**completare**

[Pagina seguente](UseCases13.md)
