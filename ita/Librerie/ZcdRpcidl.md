# ZCD - Formato RPC-IDL

La libreria di livello intermedio può essere realizzata con il tool `rpcdesign` partendo da una descrizione
delle interfacce degli oggetti remoti. Questa viene preparata dallo sviluppatore in un file con un formato
apposito, detto RPC-IDL.

Tale formato prevede la presenza di una classe radice, chiamata *root-dispatcher*. Questa ha un numero di
membri che rappresentano ognuno un *oggetto remoto* che implementa un particolare modulo dell'applicazione.
Ogni oggetto remoto ha un numero di metodi remoti.

La prima riga del file è una *riga radice*. Cioè contiene il nome di una classe radice. Si distingue perché
non contiene spazi all'inizio. Il nome della classe è seguito da uno spazio e poi dal nome dell'istanza.

Nell'esempio sotto la riga `Operatore op` indica la classe radice `Operatore` che si chiama `op`.

Le righe che seguono una riga radice, fino ad incontrare una nuova riga radice o fino alla *riga speciale*
`Errors` di cui parleremo sotto, sono riferite a quella particolare classe radice.

La prima riga dopo una riga radice è una *riga oggetto-remoto*, o *riga modulo*. Cioè contiene il nome di
una classe modulo. Si distingue perché contiene uno spazio all'inizio, seguito dal nome della classe, seguito
da uno spazio e poi dal nome dell'istanza.

Nell'esempio sotto la riga `Notificatore note` indica il modulo `Notificatore` nella classe radice `Operatore`,
che si indica come `op.note`.

Le righe che seguono una riga modulo, fino ad incontrare una nuova riga radice o una nuova riga modulo o la
*riga speciale* `Errors`, sono riferite a quel particolare modulo.

Una riga riferita ad un modulo è una *riga metodo*. Cioè contiene la firma di un metodo remoto. Si distingue
perché contiene due spazi all'inizio, seguiti dalla firma del metodo.

Nell'esempio sotto la riga `void scrivi(string msg)` indica il metodo `scrivi` nel modulo `Notificatore` nella
classe radice `Operatore`, che si chiama con `op.note.scrivi`. Questa riceve un argomento stringa e non
restituisce alcun risultato.

Un metodo può dichiarare di poter rilanciare una o più eccezioni. Nell'esempio sotto con la
riga `float divisione(int dividendo, int divisore) throws DivisionePerZeroError`.

Ogni eccezione può avere dei codici in Vala, una specie di sottoclassi. I codici delle eccezioni previste dai
vari metodi remoti vanno specificate in dettaglio in fondo al file di descrizione, dopo la riga speciale `Errors`.

Ogni *riga errore* sotto la riga speciale `Errors` è composta da uno spazio, seguito dal nome dell'error-domain,
seguito tra parentesi da tutti i possibili codici separati da virgole. 

Nell'esempio sotto la riga `DivisionePerZeroError(INDEFINITO,IMPOSSIBILE)`. Indica che
l'errore `DivisionePerZeroError` può avere i codici `INDEFINITO` e `IMPOSSIBILE`.

## Esempio

```
Operatore op
 Notificatore note
  void scrivi(string msg)
 Calcolatore cal
  float divisione(int dividendo, int divisore) throws DivisionePerZeroError
Errors
 DivisionePerZeroError(INDEFINITO,IMPOSSIBILE)
```
