= Modulo Identities - Analisi Funzionale =
<<TableOfContents(4)>>

== Terminologia ==
Usiamo la parola ''sistema'' per indicare una macchina (un computer, un dispositivo, una macchina virtuale, ...) sulla quale è in esecuzione il programma che usa il modulo Identities.

Usiamo il termine ''nodo del grafo'', o più brevemente ''nodo'', per indicare una singola entità della rete Netsukuku. Questa concretamente è una identità all'interno di un sistema.

== Ruolo del modulo ==
Il modulo Identities si occupa di organizzare le diverse identità che vivono nel sistema.

Il modulo Identities consente la creazione di una prima ''identità'' all'avvio del programma nel sistema.

In seguito il modulo, data una identità ''j'', consente di creare una nuova identità ''i'' che prende il posto di ''j''. In questo momento l'identità ''j'' resta in vita, ma in modo temporaneo. L'identità ''j'' diventa una identità ''di connettività''. Si veda la trattazione del modulo [[Netsukuku/ita/docs/ModuloQSPN/AnalisiFunzionale#Nodi_virtuali|QSPN]] per comprendere la funzione di una identità ''di connettività'' e per quale motivo essa sia temporanea.

In contrapposizione al significato di una identità ''di connettività'' introduciamo il concetto di ''identità principale''. Un sistema ha sempre una e una sola identità ''principale''. La prima identità del sistema che viene creata all'avvio è dunque l'identità ''principale'' del sistema.

Abbiamo detto che in seguito il sistema può creare una nuova identità ''i'' partendo da una precedente identità ''j''. L'identità ''j'' poteva essere in precedenza l'identità ''principale'' oppure una identità ''di connettività''. Dicendo che l'identità ''i'' prende il posto di ''j'', intendiamo significare, tra le altre cose, che se ''j'' era la ''principale'' allora adesso ''i'' diventa la ''principale''. Mentre se ''j'' era una identità ''di connettività'' allora adesso ''i'' diventa anch'essa una identità ''di connettività''.

Ogni volta che il modulo crea una identità, inizializza un identificativo per tale identità, cioè una istanza di NodeID.

 . '''Nota.''' L'identificativo deve essere univoco a livello dei domini di broadcast dei diretti vicini del sistema. Questo significa che deve essere univoco nel sistema stesso e che nessuno dei suoi diretti vicini deve avere ''attualmente'' conoscenza di un identificativo equivalente. Per una prima implementazione possiamo assumere che un intero di 32 bit scelto a caso sia sufficiente a questo scopo, ma in futuro potrebbe essere fatta una implementazione che offre maggiori garanzie.

Il modulo Identities associa ad ogni identità che viene creata un numero di istanze di classi di altri moduli, chiamati ''membri dell'identità''. Questi ''membri di identità'' sono semplicemente degli '''Object''' per il modulo Identities, in quanto esso non ha dipendenza sugli altri moduli dell'applicazione. Ogni ''membro'' è individuato con una stringa. Cioè, data una stringa ed una identità abbiamo un solo oggetto.

Il modulo Identities consente inoltre di gestire gli ''archi-identità''. Il modulo viene portato a conoscenza di archi che collegano il sistema corrente ad un altro sistema diretto vicino. Sopra ogni arco poi l'utilizzatore può richiedere al modulo di aggiungere (o rimuovere) uno o più ''archi-identità'' che collegano una certa identità del sistema corrente con una certa identità del sistema vicino. In seguito il modulo potrà in autonomia aggiungere/modificare/rimuovere un arco-identità sulla base di operazioni di aggiunta/modifica/rimozione di identità (di norma a seguito di migrazioni) nel sistema corrente e/o nel sistema vicino. Le modifiche che il modulo Identities apporta in autonomia ad un arco-identità (cioè che non sono state richieste direttamente dall'utilizzatore del modulo) vengono notificate attraverso un segnale.

Sopra ogni arco ''arc'' esiste sempre un particolare ''arco-identità'': quello che collega le due identità principali dei due sistemi collegati dall'arco ''arc''. Questo particolare arco-identità lo chiamiamo ''arco-identità principale'' dell'arco ''arc''. Un tale arco-identità c'è sempre sopra ogni arco ''arc'' (fino alla completa rimozione dell'arco ''arc'') anche se può cambiare nel tempo: infatti l'identità ''principale'' del sistema corrente o del sistema collegato tramite ''arc'' può cambiare.

Il modulo Identities consente infine al suo utilizzatore di richiedere la rimozione di una identità ''di connettività'' dal sistema corrente.

== Network namespace ==
Il modulo Identities assume che la creazione di una nuova identità comporta nel sistema la replicazione dell'intero stack di gestione del networking (detto ''network stack''). Quindi il modulo richiede che il Sistema Operativo sia in grado di realizzare questa replicazione. Volendo però essere agnostico rispetto al Sistema Operativo che gestisce il sistema, il modulo lascia che sia il suo utilizzatore ad eseguire le operazioni necessarie a richiedere al Sistema Operativo questa replicazione.

Il modulo assume che ogni replica del network stack possa essere associata ad un nome, che d'ora in poi chiamiamo ''network namespace''. Il termine è preso in prestito dalla implementazione di questa feature presente nelle versioni recenti del kernel Linux. Inoltre assume di poter indicare con il nome "" (stringa vuota) il primo network namespace (o network namespace default) che è lo stesso sul quale è in esecuzione l'applicazione stessa.

Il modulo richiede che il Sistema Operativo sia in grado di realizzare la costruzione di una pseudo-interfaccia di rete sopra una interfaccia di rete reale e di rendere operativa tale pseudo-interfaccia in una particolare replica del network stack (associarla ad un namespace). Anche in questo caso, il modulo demanda al suo utilizzatore l'esecuzione delle operazioni necessarie a richiedere al Sistema Operativo questa costruzione.

== Archi-identità ==
Abbiamo detto che il modulo sa quali archi collegano il sistema corrente ad un altro sistema diretto vicino. Per la precisione un arco ''arc~-,,i,,-~'' collega una certa interfaccia di rete reale ''if~-,,j,,-~'' del sistema corrente ad una certa interfaccia di rete reale del sistema vicino, di cui il sistema corrente conosce il MAC address ''arc~-,,i,,-~.peer_mac'' e l'indirizzo link-local ''arc~-,,i,,-~.peer_linklocal''.

Sopra l'arco ''arc~-,,i,,-~'' il modulo consente la creazione di uno o più ''archi-identità'' che collegano una certa identità del sistema corrente con una certa identità del sistema vicino. Consideriamo ad esempio l'identità ''id~-,,k,,-~''. Il modulo Identities può creare l'arco-identità ''id-arc~-,,0,,-~'' associato all'arco ''arc~-,,i,,-~'' e alla sua identità ''id~-,,k,,-~'' e che collega alla identità del vicino ''b~-,,q,,-~''. Può anche creare un altro arco-identità ''id-arc~-,,1,,-~'', sempre associato all'arco ''arc~-,,i,,-~'' e alla sua identità ''id~-,,k,,-~'', ma che collega alla identità del vicino ''b~-,,w,,-~''.

Limitiamoci a considerare l'arco-identità ''id-arc~-,,0,,-~''. Consideriamo il vertice nel nostro sistema di ''id-arc~-,,0,,-~''. Sappiamo che esso è associato all'arco ''arc~-,,i,,-~'', il quale è stato fatto sulla interfaccia di rete reale ''if~-,,j,,-~'' del sistema corrente. Sappiamo anche che esso è associato all'identità ''id~-,,k,,-~'', la quale è associata ad un particolare network namespace ''ns~-,,k,,-~''. In questo network namespace, se non è quello default, opera una certa pseudo-interfaccia di rete costruita sull'interfaccia di rete reale ''if~-,,j,,-~'', chiamiamola ''if~-,,j,k,,-~''. Sappiamo che tale pseudo-interfaccia ha un suo MAC address e un suo indirizzo link-local che sono distinti da quelli di ''if~-,,j,,-~''.

Facciamo delle considerazioni analoghe per il vertice nel sistema vicino di ''id-arc~-,,0,,-~''. Sappiamo che esso è associato all'identità del vicino ''b~-,,q,,-~''. Questa potrebbe non essere l'identità associata al network namespace default: quindi vogliamo mantenere l'informazione ''id-arc~-,,0,,-~.peer_mac'' e ''id-arc~-,,0,,-~.peer_linklocal'' che possono differire da ''arc~-,,i,,-~.peer_mac'' e ''arc~-,,i,,-~.peer_linklocal''.

Riassumendo, un arco-identità ''id-arc~-,,0,,-~'' è associato ad una identità ''id~-,,k,,-~'' e ad un arco ''arc~-,,i,,-~'' ed ha queste caratteristiche (o membri):
 * ''peer_nodeid'' - L'identificativo dell'identità del vicino, cioè ''b~-,,q,,-~''.
 * ''peer_mac'' - Il MAC address dell'interfaccia (reale o pseudo) associata a ''b~-,,q,,-~''.
 * ''peer_linklocal'' - L'indirizzo link-local dell'interfaccia (reale o pseudo) associata a ''b~-,,q,,-~''.

Aggiungiamo infine che, siccome vedremo in seguito che nel tempo nel sistema ''b'' può cambiare l'associazione fra l'identità ''b~-,,q,,-~'' e un network namespace, nel tempo i valori ''peer_mac'' e ''peer_linklocal'' di ''id-arc~-,,0,,-~'' possono cambiare. Eventuali cambiamenti sono gestiti in autonomia dal modulo Identities, cioè non è l'utilizzatore del modulo a richiederli.

== Operazioni ==
Il modulo fa uso delle [[Netsukuku/ita/docs/Librerie/TaskletSystem|tasklet]], un sistema di multithreading cooperativo.

Il modulo fa uso del framework [[Netsukuku/ita/docs/Librerie/ZCD|ZCD]], precisamente appoggiandosi alla libreria di livello intermedio ''ntkdrpc'' prodotta con questo framework per formalizzare i metodi remoti usati nel demone ''ntkd''.

Le operazioni del modulo sono implementate per la maggior parte in metodi di una classe chiamata !IdentityManager. Di essa viene creata una sola istanza dall'applicazione in esecuzione su un sistema. Di seguito ci possiamo riferire a tale istanza semplicemente con il termine ''manager''.

Poiché il manager è uno solo nel sistema, diciamo che il modulo Identities è un modulo ''di sistema''. Si veda nella trattazione del modulo [[Netsukuku/ita/docs/ModuloNeighborhood/AnalisiFunzionale#Identit.2BAOA_multiple_in_un_nodo|Neighborhood]] la differenza tra moduli ''di sistema'' e moduli ''di identità''.

Nel costruire l'istanza del manager, l'utilizzatore del modulo specifica quali sono i nomi delle interfacce di rete reali gestite dal sistema, i relativi MAC e gli indirizzi link-local (che gli sono stati definitivamente assegnati dal modulo Neighborhood). Inoltre gli passa un manager di network namespace, o ''netns-manager'', cioè un oggetto (si veda sotto la descrizione dell'interfaccia IIdmgmtNetnsManager) per svolgere le operazioni sui network namespace e le interfacce di rete. Inoltre gli passa una ''stub-factory'', cioè un oggetto (si veda sotto la descrizione dell'interfaccia IIdmgmtStubFactory) per ottenere uno stub per comunicare con un vicino attraverso un arco.

Nel costruttore il manager crea la prima identità, chiamiamola ''id~-,,0,,-~''. Il manager genera un identificativo per la prima identità, che è la ''principale''. Chiamiamo identità principale quella associata al network namespace default e che gestisce quindi in esso le interfacce di rete reali. Il manager gli associa un ''namespace'' uguale a stringa vuota. Il manager inoltre, per ogni interfaccia di rete reale ''dev~-,,i,,-~'' che gli è stata segnalata, associa alla coppia ''id~-,,0,,-~-dev~-,,i,,-~'' una struttura dati con le informazioni della interfaccia gestita dalla identità: ''dev'', ''mac'', ''linklocal''. Per la identità principale queste sono le interfacce reali.

L'utilizzatore dice al manager di associare alcuni oggetti alla identità appena creata: un oggetto per "qspn", un oggetto per "peerservices", eccetera.

In seguito, quando il sistema rileva la presenza di vicini, l'utilizzatore dice al manager che è stato formato un arco con un vicino, chiamiamolo ''arc~-,,0,,-~''. Dall'oggetto ''arco'' (si veda sotto la descrizione dell'interfaccia IIdmgmtArc) si può risalire alla interfaccia di rete reale su cui è stato realizzato, chiamiamola ''ir(arc~-,,0,,-~)''. La prima volta questo evento accadrà quando il manager ha creato una sola identità. Ma in seguito nel tempo questo evento può accadere anche quando il manager ha più di una identità.

Il manager associa alla coppia ''id~-,,0,,-~-arc~-,,0,,-~'' un contenitore, inizialmente vuoto, nel quale potranno essere aggiunte strutture dati che rappresentano un ''arco-identità''.

Poi il manager realizza automaticamente il primo ''arco-identità'', che è il ''principale'' di ''arc~-,,0,,-~''. Cioè quello tra la propria identità principale ''id~-,,0,,-~'' e l'identità principale del sistema vicino. Per farlo prima comunica con il vicino tramite l'arco ''arc~-,,0,,-~'' e gli chiede l'identificativo della sua identità principale, chiamiamola ''b~-,,j,,-~''. Di questa identità del vicino il sistema corrente conosce il MAC address e l'indirizzo link-local, in quanto tale identità gestisce le interfacce di rete reali. Dall'istanza ''arc~-,,0,,-~'' (si veda sotto la descrizione dell'interfaccia IIdmgmtArc) si può risalire anche a questi dati.

Nel tempo, l'utilizzatore può:
 * Aggiungere un arco che è stato realizzato dal sistema.
 * Rimuovere un arco non più funzionante.
 * Aggiungere su un arco già segnalato un arco-identità.
 * Rimuovere da un arco già segnalato un arco-identità.
 * Aggiungere una interfaccia di rete reale a quelle gestite dal sistema. In questo caso indica il nome, il MAC e l'indirizzo link-local.
 * Rimuovere una interfaccia di rete reale da quelle gestite dal sistema. In questo caso indica il nome.

Può capitare (vedremo fra poco in quali occasioni) che il modulo Identities in autonomia aggiunga o rimuova o modifichi un arco-identità. Questo evento è notificato all'utilizzatore del modulo con un segnale che esso può ascoltare.

----
Ad un certo punto l'utilizzatore del modulo può richiedere al manager di realizzare la duplicazione di una sua identità a causa di una migrazione.

Se la migrazione è di un singolo ''nodo del grafo'' allora le operazioni da fare sono alquanto semplici. Tuttavia anche in questo caso vedremo che un certo numero di comunicazioni vengono fatte ai sistemi vicini e di questo si occupa il manager.

Se invece la migrazione è di un cluster di nodi (detto ''g-nodo'') allora le operazioni sono alquanto complesse. Si rende necessaria una concertazione delle operazioni, inizialmente con tutti i nodi del g-nodo e in seguito con i diretti vicini del sistema. La prima parte, cioè la concertazione con tutti i nodi del g-nodo, mira a far conoscere a tutti i nodi del g-nodo alcune informazioni sulla migrazione stessa e fra queste un ''identificativo di migrazione'' condiviso: questa parte non è di competenza del modulo Identities. La seconda parte, cioè la concertazione coi diretti vicini, mira a far sì che la duplicazione degli archi-identità avvenga in modo corretto anche fra due nodi che appartengono entrambi al g-nodo che migra.

Ad esempio, supponiamo che l'identità ''a~-,,0,,-~'' nel sistema ''a'' e l'identità ''b~-,,0,,-~'' nel sistema ''b'' siano collegate con un arco-identità. Supponiamo che entrambe le identità appartengano ad un g-nodo che migra e si formino a causa di questa migrazione le identità ''a~-,,1,,-~'' nel sistema ''a'' e ''b~-,,1,,-~'' nel sistema ''b''. Allora deve persistere l'arco-identità ''a~-,,0,,-~-b~-,,0,,-~'' e si deve aggiungere l'arco-identità ''a~-,,1,,-~-b~-,,1,,-~''. Non devono invece formarsi né ''a~-,,0,,-~-b~-,,1,,-~'' né ''a~-,,1,,-~-b~-,,0,,-~''.

Invece, supponiamo che l'identità ''a~-,,0,,-~'' nel sistema ''a'' e l'identità ''b~-,,0,,-~'' nel sistema ''b'' siano collegate con un arco-identità. Supponiamo che solo l'identità ''a~-,,0,,-~'' appartenga ad un g-nodo che migra e si formi a causa di questa migrazione l'identità ''a~-,,1,,-~''. Allora deve persistere l'arco-identità ''a~-,,0,,-~-b~-,,0,,-~'' e si deve aggiungere l'arco-identità ''a~-,,1,,-~-b~-,,0,,-~''.

Quest'ultimo esempio vale anche per i casi in cui la migrazione sia di un singolo nodo, in questo esempio ''a~-,,0,,-~'' nel sistema ''a''.

Della concertazione coi diretti vicini si occupa il modulo Identities. Questa è particolarmente complessa quando entrambi i vicini partecipano (con una o più identità) alla migrazione.

Per realizzare questa concertazione si ha una prima fase in cui, tramite un meccanismo che non è di pertinenza del modulo Identities, su tutti i nodi del g-nodo viene chiamato il metodo ''prepare_add_identity(migration_id, old_id)''. Soltanto al termine, cioè quando tutti i nodi del g-nodo hanno processato il metodo ''prepare_add_identity'', viene chiamato su tutti i nodi il metodo ''add_identity(migration_id, old_id)''.

Al resto pensa il modulo Identities.

Al termine delle operazioni, cioè al ritorno del metodo ''add_identity'', l'utilizzatore del modulo viene portato a conoscenza dell'identificativo della nuova identità.

----
Ad un certo punto l'utilizzatore del modulo può richiedere al manager di rimuovere una sua identità ''id~-,,j,,-~''.

Il manager per prima cosa rimuove tutti gli archi-identità associati a ''id~-,,j,,-~''. Per ogni arco-identità ''id-arc~-,,k,,-~'' associato a ''id~-,,j,,-~'', indicando con ''arc~-,,q,,-~'' l'arco su cui esso è formato, il manager tenta anche (ma non è necessario che vi riesca) di comunicare al sistema vicino attraverso l'arco ''arc~-,,q,,-~'' che va rimosso l'arco tra ''id~-,,j,,-~'' e ''id-arc~-,,k,,-~.peer_id''. Se la comunicazione riesce, il manager nel sistema vicino ha l'opportunità di rimuovere l'arco-identità e notificarlo al suo utilizzatore con un apposito segnale.

Rimossi tutti gli archi, il manager usa il ''netns-manager'' per rimuovere le pseudo-interfacce e il network namespace associati a ''id~-,,j,,-~''.

Rimuove infine dalla sua memoria tutte le associazioni che manteneva per ''id~-,,j,,-~''.

== Memoria del modulo ==
Il modulo mantiene un elenco, chiamato ''dev_list'', dei nomi delle interfacce di rete reali attualmente gestite dal sistema.

Il modulo mantiene un elenco, chiamato ''arc_list'', degli archi attualmente realizzati dal sistema con i suoi vicini. Da un arco può risalire al nome dell'interfaccia di rete reale su cui è stato realizzato e ai dati (MAC address e indirizzo link-local) dell'interfaccia di rete reale nel sistema diretto vicino.

Il modulo mantiene un elenco, chiamato ''id_list'', delle identità attualmente presenti nel sistema. Inoltre mantiene un riferimento, chiamato ''main_id'', alla identità ''principale''.

Il modulo mantiene una associazione chiamata ''namespaces'' che partendo da una identità ''id~-,,0,,-~'' del sistema corrente, con ''id~-,,0,,-~ ∈ id_list'', individua il nome del network namespace che essa gestisce.

Il modulo mantiene una associazione chiamata ''handled_nics'' che partendo dalla coppia ''id~-,,0,,-~-dev~-,,i,,-~'' (una identità e il nome di una interfaccia di rete reale), con ''id~-,,0,,-~ ∈ id_list'' e ''dev~-,,i,,-~ ∈ dev_list'', individua la relativa istanza di !HandledNic.

Il modulo mantiene una associazione chiamata ''identity_arcs'' che partendo dalla coppia ''id~-,,0,,-~-arc~-,,0,,-~'' (una identità e un arco), con ''id~-,,0,,-~ ∈ id_list'' e ''arc~-,,0,,-~ ∈ arc_list'', individua il relativo contenitore di istanze di !IdentityArc.

== Requisiti ==
L'utilizzatore deve fornire al modulo fin dalla sua inizializzazione questi requisiti:
 * L'elenco delle interfacce reali gestite dal sistema nel network namespace default. Di ognuna di esse va indicato il nome, il MAC, il link-local assegnato dal modulo Neighborhood.
 * Un manager di network namespace, o ''netns-manager'' (si veda sotto la descrizione dell'interfaccia IIdmgmtNetnsManager).
 * Factory per creare uno "stub" per invocare metodi remoti in un vicino attraverso un dato arco, o ''stub-factory'' (si veda sotto la descrizione dell'interfaccia IIdmgmtStubFactory).
 . Questo oggetto può anche essere usato per recuperare un arco che è quello da cui è stata ricevuta una chiamata a metodo remoto.

Durante il tempo l'utilizzatore potrà fornire al modulo altre informazioni:
 * Aggiungere una interfaccia di rete reale gestita dal sistema.
 . L'interfaccia appena aggiunta sarà immediatamente usata solo dalla identità ''principale''. Se ci fossero al momento altre identità nel sistema, ognuna con un suo network namespace, non viene creata un pseudo-interfaccia per ogni identità ''di connettività''.
 * Rimuovere una interfaccia di rete reale che non è più gestita dal sistema.
 . Questo causa la rimozione anche di tutte le pseudo-interfacce costruite su di essa e al momento in uso da eventuali identità ''di connettività''.
 * Aggiungere un arco che il sistema ha realizzato.
 * Rimuovere un arco che non è più valido.

== Deliverable ==
Il modulo Identities risponde a queste richieste:
 * ''NodeID get_main_id()'' - Ottenere il NodeID della identità ''principale'' del sistema.
 * ''Gee.List<NodeID> get_id_list()'' - Ottenere i NodeID di tutte le identità del sistema.
 * ''string get_namespace(NodeID id~-,,0,,-~)'' - Ottenere il nome del network namespace gestito dalla identità ''id~-,,0,,-~''.
 * ''string get_pseudodev(NodeID id~-,,0,,-~, string dev)'' - Ottenere il nome dell'interfaccia di rete (reale o pseudo) gestita dalla identità ''id~-,,0,,-~'' al posto dell'interfaccia reale ''dev''.
 * ''Gee.List<IIdmgmtIdentityArc> get_identity_arcs(IIdmgmtArc arc~-,,0,,-~, NodeID id~-,,0,,-~)'' - Dato un arco e una identità nel sistema (passata come NodeID) ottenere i dati di tutti gli ''archi-identità'' formati su questo arco da questa identità.

Il modulo Identities permette queste operazioni:
 * Creazione della prima identità. Nel costruttore.
 * Duplicazione di una identità a seguito di una migrazione. Eventualmente in due fasi se la migrazione è di un g-nodo. Metodi ''prepare_add_identity'' e ''add_identity''.
 * Associazione di un oggetto ad una identità. Metodi ''set_identity_module'' e ''get_identity_module''.
 . Come detto prima, ad ogni identità si possono associare diversi oggetti, ognuno identificato da un nome. Di norma un oggetto per ogni modulo ''di identità''.
 * Aggiunta di un arco-identità. Metodo ''add_identity_arc''.
 * Rimozione di un arco-identità. Metodo ''remove_identity_arc''.
 * Rimozione di una identità che era ''di connettività''. Metodo ''remove_identity''.

Il modulo Identities segnala questi eventi:
 * Aggiunta di un arco-identità. Questo può anche avvenire senza che ci sia stata una diretta richiesta da parte dell'utilizzatore del modulo. Segnale ''identity_arc_added''.
 * Modifica dei valori ''peer_mac'' e ''peer_linklocal'' di un arco-identità. Questo avviene senza che ci sia stata una diretta richiesta da parte dell'utilizzatore del modulo. Segnale ''identity_arc_changed''.
 * Rimozione di un arco-identità. Questo può anche avvenire senza che ci sia stata una diretta richiesta da parte dell'utilizzatore del modulo. Segnale ''identity_arc_removed''.

== Classi e interfacce ==
Il netns-manager è un oggetto di cui il modulo conosce l'interfaccia IIdmgmtNetnsManager. Tramite essa il modulo può:
 * Creare un network namespace con un dato nome. Metodo ''create_namespace''.
 * Costruire una pseudo-interfaccia di rete sopra una data interfaccia di rete reale e metterla su un dato network namespace. Metodo ''create_pseudodev''.
 . Sulla chiamata va specificato il nome da dare alla pseudo-interfaccia. La chiamata restituisce il MAC address assegnato alla pseudo-interfaccia.
 * Assegnare un dato indirizzo IP link-local ad una data pseudo-interfaccia di rete su un dato network namespace. Metodo ''add_address''.
 * Aggiungere/rimuovere un collegamento diretto da un dato indirizzo IP link-local locale ad un dato indirizzo IP link-local attraverso una data interfaccia di rete (pseudo o reale) su un dato network namespace. Metodi ''add_gateway'' e ''remove_gateway''.
 . Un tale collegamento diretto non va mai "cambiato". Infatti un indirizzo link-local di un vicino individua sempre univocamente un arco tra una specifica interfaccia del sistema corrente e una specifica interfaccia del sistema vicino. Non può formarsi un altro arco verso lo stesso indirizzo link-local da una diversa interfaccia di rete del sistema corrente. L'unico scenario possibile è la creazione di una diversa identità nel sistema corrente (ma comunque si tratta di una aggiunta su un diverso network namespace) o nel sistema vicino (ma si tratta di una aggiunta con un diverso indirizzo link-local del vicino).
 * Svuotare del tutto la routing table ''main'' di un dato network namespace diverso dal default. Metodo ''flush_table''. Il modulo lo usa prima di eliminare tutte le pseudo-interfacce e l'intero network namespace.
 * Eliminare una data pseudo-interfaccia di rete da un dato network namespace. Metodo ''delete_pseudodev''.
 * Eliminare un dato network namespace. Metodo ''delete_namespace''.

----
La stub-factory è un oggetto di cui il modulo conosce l'interfaccia IIdmgmtStubFactory. Tramite essa il modulo può:
 * Ottenere uno stub per comunicazioni reliable ad un vicino dato un arco. Metodo ''get_stub''.
 * Ottenere l'arco da cui è stata ricevuta una chiamata a metodo remoto, passando il ''caller'' ricevuto nei paramtri del metodo remoto. Metodo ''get_arc''.

----
La classe Identity è interna al modulo. All'esterno del modulo una identità è rappresentata da un NodeID, sia per le identità del sistema corrente che per quelle dei vicini.

La classe Identity incapsula un NodeID. Fornisce inoltre un metodo ''to_string'' che produce (a partire dal NodeID) una stringa che la identifica univocamente.

Nella classe Identity vengono memorizzate le istanze delle classi dei moduli ''di identità''. Cioè i ''membri'' dell'identità.

----
La classe usata per l'identificativo di una identità, cioè NodeID, è una classe serializzabile definita nella libreria [[Netsukuku/ita/docs/Librerie/Common|Common]]. Il modulo Neighborhood ha una dipendenza su questa libreria, quindi conosce tale classe.

Il modulo Identities crea le istanze di questa classe relative alle identità di questo sistema.

Il modulo Identities riceve anche istanze di questa classe relative alle identità di sistemi vicini. Naturalmente sa accedere alle informazioni in essa contenute.

----
Un arco che il sistema ha realizzato è rappresentato con una classe che il modulo non conosce. Il modulo espone l'interfaccia IIdmgmtArc che tale oggetto deve implementare.

L'interfaccia IIdmgmtArc consente di:
 * Leggere il nome dell'interfaccia di rete reale su cui l'arco è realizzato (metodo ''get_dev'').
 * Leggere l'indirizzo link-local e il MAC address del vertice di questo arco sul sistema vicino (metodi ''get_peer_linklocal'' e ''get_peer_mac'').
 . Questi servono sia per realizzare in automatico l'arco-identità principale di un arco, sia per riconoscere se un dato arco-identità è il principale. Infatti il modulo Identities assume che non sono di sua competenza né la realizzazione né la rimozione del collegamento diretto (sul ''network namespace default'') tra i relativi indirizzi link-local che concretizza l'arco-identità ''principale'' di un arco.

----
La classe !HandledNic è una struttura dati interna al modulo. Essa si riferisce ad una interfaccia di rete (reale o pseudo) la quale è associata ad una data interfaccia di rete reale ed è gestita da una data identità del sistema corrente.

Ad esempio, se il modulo nel sistema ''a'' vuole recuperare il nome della pseudo-interfaccia di rete che la sua identità ''id~-,,0,,-~'' (la quale non è la principale) gestisce e che è stata costruita sulla interfaccia reale "eth0", il modulo esamina l'associazione ''handled_nics(id~-,,0,,-~-eth0)'' e ottiene una istanza di !HandledNic; quindi guarda il suo membro ''dev'', che ad esempio contiene "ntkv0_eth0".

Relativamente a questa interfaccia di rete (reale o pseudo) la struttura dati contiene:
 * ''dev'' - Una stringa. Il nome.
 * ''mac'' - Una stringa. Il MAC address.
 * ''linklocal'' - Una stringa. L'indirizzo link-local assegnato.

----
La classe !IdentityArc è una struttura dati interna al modulo. Essa si riferisce ad una interfaccia di rete (reale o pseudo) gestita da una identità di un sistema vicino. Il vicino è collegato al sistema corrente tramite un dato arco e esiste un arco-identità tramite quella identità del sistema vicino e una data identità del sistema corrente.

Ad esempio, supponiamo che nel sistema corrente ''a'' abbiamo l'identità ''id~-,,0,,-~''. Inoltre abbiamo un arco ''arc~-,,1,,-~'' che collega una interfaccia del sistema ''a'', diciamo ''if~-,,a1,,-~'', ad una interfaccia del sistema ''b'', diciamo ''if~-,,b1,,-~''. Se voglio vedere quali archi-identità collegano l'identità ''id~-,,0,,-~'' di ''a'' ad altre identità di ''b'' tramite l'arco ''arc~-,,1,,-~'', allora il modulo esamina l'associazione ''identity_arcs(id~-,,0,,-~-arc~-,,1,,-~)'' e ottiene una lista di istanze di !IdentityArc; le esamina una ad una accedendo ai suoi membri.

La classe !IdentityArc contiene questi dati:
 * ''peer_nodeid'' - Un NodeID. L'identificativo di una identità nel sistema ''b'' per la quale esiste un arco-identità con la nostra ''id~-,,0,,-~'' costruito sull'arco ''arc~-,,1,,-~''.
 * ''peer_mac'' - Una stringa. Il MAC address dell'interfaccia gestita dalla ''peer_nodeid'' nel sistema ''b'' costruita sulla ''if~-,,b1,,-~''.
 * ''peer_linklocal'' - Una stringa. L'indirizzo link-local assegnato all'interfaccia gestita dalla ''peer_nodeid'' nel sistema ''b'' costruita sulla ''if~-,,b1,,-~''.

Forniamo la classe !IdentityArc del metodo ''copy'' per facilitare la duplicazione degli archi-identità quando viene aggiunta una nuova identità al sistema.

L'interfaccia dell'oggetto arco-identità nota all'esterno del modulo, IIdmgmtIdentityArc, permette solo la lettura dei suddetti dati:
 * Leggere l'identificativo della identità nel sistema vicino. Metodo ''get_peer_nodeid()''.
 * Leggere il MAC address dell'interfaccia gestita dalla identità nel sistema vicino. Metodo ''get_peer_mac()''.
 * Leggere l'indirizzo link-local assegnato all'interfaccia gestita dalla identità nel sistema vicino. Metodo ''get_peer_linklocal()''.

----
Il modulo Identities definisce la classe interna !DuplicationData. Essa è serializzabile ed è usata come valore di ritorno nel metodo remoto ''match_duplication'' esposto dalla classe !IdentityManager. Nella definizione dei metodi RPC si usa come segnaposto l'interfaccia IDuplicationData.

Questo metodo remoto è chiamato dal sistema corrente ''a'' sul sistema vicino ''b'' attraverso un dato arco ''arc~-,,0,,-~''. Sopra tale arco esiste un arco-identità che collega l'identità ''a~-,,0,,-~'' alla identità ''b~-,,j,,-~''. Nel sistema ''a'' l'identità ''a~-,,0,,-~'' è stata appena duplicata (a causa di una migrazione). La chiamata di questo metodo remoto serve a comunicare al sistema ''b'' le informazioni relative a questa duplicazione e a sapere dal sistema ''b'' se la stessa migrazione ha prodotto una duplicazione dell'identità ''b~-,,j,,-~''. Se è così viene restituita una istanza di !DuplicationData, altrimenti ''null''.

La classe !DuplicationData contiene:
 * ''peer_new_id'' - L'identificativo della nuova identità frutto della duplicazione (migrazione) di ''b~-,,j,,-~'' nel sistema vicino ''b''.
 * ''peer_old_id_new_mac'' - Il MAC della nuova pseudo-interfaccia gestita ora da ''b~-,,j,,-~'' nel sistema vicino ''b''.
 * ''peer_old_id_new_linklocal'' - L'indirizzo link-local della nuova pseudo-interfaccia gestita ora da ''b~-,,j,,-~'' nel sistema vicino ''b''.

