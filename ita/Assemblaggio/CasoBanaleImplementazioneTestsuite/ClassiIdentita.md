# Identità e archi-identità

Nella classe `N.IdentityData` memorizziamo le informazioni relative a ogni identità
assunta dal nodo corrente: una principale e zero o più di connettività.  
Fra queste informazioni ci sono i relativi archi-identità, rappresentati da istanze
della classe `N.IdentityArc`.  
Entrambe le classi sono nel file `main.vala`.

Si era pensato dall'inizio di affidare al modulo `Identities` anche il compito
(oltre ad altri importanti come la gestione degli archi-identità)
di tenere traccia delle istanze di altre classi legate a ogni identità.  
Ma avere nel programma stesso una classe (quale appunto `N.IdentityData`)
a questo scopo è più pratico perché il programma ha conoscenza delle varie
classi dei moduli, mentre il modulo `Identities` è in questo senso agnostico.

## Gestione istanze di IdentityData

La creazione di una istanza di `N.IdentityData` si fa chiamando il metodo
helper `find_or_create_local_identity` nel file `main.vala` e passando
il `NodeID` che è stato assegnato all'identità (dal modulo `Identities`).  
Il metodo usa la variabile globale `next_local_identity_index` per assegnare alla
nuova istanza un identificativo locale numerico progressivo chiamato
`local_identity_index`. Poi mette la nuova istanza nell hashmap `local_identities`
indicizzate per il valore intero che costituisce il `NodeID`.  
L'istanza di `N.IdentityData` ha come membri `nodeid` e `local_identity_index`.

Il metodo helper `find_local_identity` nel file `main.vala` è usato per cercare una
istanza di cui si conosce il `NodeID`. Se esiste la restituisce altrimenti
restituisce *null*.

Il metodo helper `find_local_identity_by_index` nel file `main.vala` è usato per
cercare una istanza di cui si conosce il `local_identity_index`. Se esiste la
restituisce altrimenti restituisce *null*.

Il metodo helper `remove_local_identity` nel file `main.vala` è usato per rimuovere
una istanza dal hashmap dopo aver verificato che era in effetti presente.

La variabile globale `main_identity_data` nel file `main.vala` identifica l'identità
principale.

## Proprietà di una IdentityData

Ogni istanza di `N.IdentityData` memorizza una lista di archi-identità nel membro
`identity_arcs`, rappresentati da istanze di `IdentityArc`.

Ogni istanza di `N.IdentityData` può anche memorizzare:

*   L'indirizzo Netsukuku come istanza `Naddr my_naddr`.
*   Il fingerprint come istanza `Fingerprint my_fp`.
*   I livelli come "identità di connettività" `int connectivity_from_level` e
    `int connectivity_to_level`.
*   L'identità da cui è derivata `weak IdentityData? copy_of_identity`.
*   L'istanza `QspnManager qspn_mgr`.
*   L'istanza `PeersManager peers_mgr`.
*   L'istanza `CoordinatorManager coord_mgr`.
*   L'istanza `HookingManager hook_mgr`.
*   Un getter `bool main_id` dice se è la principale confrontando se stessa con la
    variabile globale `IdentityData main_identity_data` sempre nel file `main.vala`.
*   L'insieme degli indirizzi IP pubblici propri.
*   L'insieme degli indirizzi IP pubblici come destinazioni.
*   Il network namespace (in realtà lo si chiede di volta in volta al IdentityManager)
    in cui si trova a operare.

Alcuni metodi pubblici della classe `N.IdentityData` sono usati per registrare gli
handler dei segnali che produce il modulo Qspn. Le implementazioni sono comunque
funzioni esterne alla classe, per poterle raggruppare nel file `qspn_signals.vala`.

Questi sono:

*  `per_identity_qspn_arc_removed`,
*  `per_identity_qspn_changed_fp`,
*  `per_identity_qspn_changed_nodes_inside`,
*  `per_identity_qspn_destination_added`,
*  `per_identity_qspn_destination_removed`,
*  `per_identity_qspn_gnode_splitted`,
*  `per_identity_qspn_path_added`,
*  `per_identity_qspn_path_changed`,
*  `per_identity_qspn_path_removed`,
*  `per_identity_qspn_presence_notified`,
*  `per_identity_qspn_qspn_bootstrap_complete`,
*  `per_identity_qspn_remove_identity`.

## Gestione istanze di IdentityArc

Le istanze di `N.IdentityArc` che rappresentano gli archi-identità, come detto sopra,
sono memorizzate come liste nel membro `identity_arcs` dell'istanza di
`N.IdentityData` a cui appartengono.

Le funzioni `find_identity_arc` e `find_identity_arc_by_peer_nodeid` facilitano
il recupero di una istanza di `N.IdentityArc` partendo dall'arco-identità come
restituito dal modulo Identities (cioè da una istanza di
`IIdmgmtIdentityArc id_arc`) oppure dalle informazioni note al programma (cioè
tramite `IdentityData identity_data`, `IIdmgmtArc arc`, `NodeID peer_nodeid`).

## Proprietà di un IdentityArc

Ogni istanza di `N.IdentityArc` memorizza immediatamente:

*   l'identità a cui è associata (come indice
    `int local_identity_index`)
*   l'arco su cui poggia (come passato al modulo Identites, cioè `IIdmgmtArc arc`)
*   l'arco identità stesso (come ottenuto dal modulo Identites, cioè
    `IIdmgmtIdentityArc id_arc`)

L'identità è memorizzata in un `IdentityArc` come indice `int local_identity_index`.
Questa classe ha un getter per risalire alla istanza
di `IdentityData` che se viene invocato su una identità che non è più nell hashmap
`local_identities` ha l'effetto di terminare la tasklet corrente.

Ogni istanza può anche memorizzare:

*   `string peer_mac`
*   `string peer_linklocal`
*   `QspnArc? qspn_arc`
*   `HookingIdentityArc? hooking_arc`
*   `int64? network_id` - identifica la rete in cui esiste il peer.
