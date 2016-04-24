# Common

La libreria _ntkd-common_ contiene alcune classi note a più di un modulo.

Siccome i moduli tendono ad essere abbastanza complessi, si cerca di isolarli in modo da poter verificare la correttezza di un modulo senza tenere conto delle problematiche di cui si occupa un altro modulo. Quindi le classi che sono note a più di un modulo non devono costituire una dipendenza di un modulo su un altro.

Per questo non sono messe all'interno di uno dei moduli, ma dentro una libreria molto semplice su cui alcuni moduli avranno una dipendenza comune.

## HCoord
La classe HCoord è nota a questi moduli:
 * Qspn
 * PeerServices

E' una classe serializzabile, cioè le cui istanze sono adatte al passaggio di dati a metodi remoti (vedi framework [ZCD](../../zcd/wiki/ita_ZCD)). Una sua istanza contiene le coordinate gerarchiche di un g-nodo nella mappa del nodo: livello e identificativo nel livello.

## NodeID
La classe NodeID è nota a questi moduli:
 * Neighborhood
 * ...

E' una classe serializzabile, cioè le cui istanze sono adatte al passaggio di dati a metodi remoti (vedi framework [ZCD](../../zcd/wiki/ita_ZCD)). Una sua istanza contiene un intero a 32 bit che identifica, in modo presumibilmente univoco, una identità che vive in un nodo.
