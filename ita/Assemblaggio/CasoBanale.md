# Caso banale

Consideriamo un caso banale. Un nodo in cui il programma si avvia e dopo un po' viene
terminato senza che incontri altri nodi.

Nel presente documento trattiamo la sequenza di operazioni che fa il programma in
questo caso. In particolare le operazioni di creazione e rimozione delle singole
istanze delle classi dei vari moduli.

La parte iniziale sarà comune a tutti i casi. Ogni nodo appena il programma si avvia
si considera l'unico membro di una nuova rete.

La parte finale sarà comune a qualsiasi nodo che al termine del programma si trovi
a essere l'ultimo rimasto della sua rete.

## Panoramica delle operazioni

All'avvio del programma si crea una istanza di `NeighborhoodManager`.  
Questa istanza viene subito usata per mettere il nodo in ascolto sulle varie interfacce
di rete che il programma dovrà gestire. Come risultato verranno rilevati su questa
istanza alcuni segnali per mezzo dei quali il programma avrà accesso a informazioni
che sono necessarie alla creazione di una istanza di `IdentityManager`.

Subito dopo si crea una istanza di `IdentityManager`.  
La prima identità del nodo, che è una *identità principale*, nasce nel momento stesso
in cui si avvia il programma. L'istanza di `IdentityManager` quando viene costruita
assegna subito a questa prima identità il suo identificativo. Quindi il programma,
subito dopo aver costruito l'istanza di `IdentityManager`, la interroga per conoscere
la prima identità.

Poi il programma associa alla prima identità del nodo una istanza di `QspnManager`
che è creata in modalità `create_net`. In questa modalità viene creata dal modulo `Qspn`
una nuova rete composta dal solo nodo.

Ora il programma dovrà associare alla prima identità del nodo una istanza di `PeersManager`.
Questa è necessaria per poi associare alla prima identità del nodo una istanza
di `CoordinatorManager`. Infine questa è necessaria per associare alla prima identità del
nodo una istanza di `HookingManager`.  
Va notato che le prime istanze di `PeersManager` e`CoordinatorManager` si troveranno a
operare in un contesto molto "semplice" essendo la prima identità del nodo unico membro
di una nuova rete.
