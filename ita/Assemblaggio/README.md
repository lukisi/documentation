Appunti per facilitare le operazioni di assemblaggio dei vari moduli
sotto il coordinamento del programma `ntkd`.

Nel documento [Interdipendenze](Interdipendenze.md) vengono descritte
le dipendenze funzionali tra i vari moduli e i loro compiti.

Il documento [Ciclo di vita](CicloDiVita.md) dice quando le singole istanze
delle classi dei vari moduli sono create e quanto restano attive.

### Caso banale

Può essere utile per facilitare l'implementazione partire con un caso banale. Un nodo
in cui il programma si avvia e dopo un po' viene terminato senza che incontri altri
nodi.

Nel documento [Caso banale](CasoBanale.md) tratteremo la sequenza di operazioni che
il programma fa in questo caso e che coinvolgono la creazione
e la rimozione delle singole istanze delle classi dei vari moduli.

### Testsuite #1

Subito dopo aver realizzato l'implementazione di una testsuite per il caso banale
sopra descritto, proviamo a implementare un primo test significativo.

Nel programma che realizza il caso banale, alcuni metodi saranno rimasti non
implementati perché non necessari in un nodo che non incontra nessun altro nodo.  
Quando il programma viene eseguito in uno scenario in cui questi metodi vengono
chiamati, il codice va in errore e questo ci indica in quali punti dobbiamo
proseguire il lavoro.

Nel documento [Scenario #1](Scenario01.md) tratteremo la sequenza di operazioni
che il programma fa in un semplice scenario in cui due nodi si incontrano.
