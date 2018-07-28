# Netsukuku - Documenti di analisi

La suite di software per un nodo nella rete Netsukuku si compone di:

*   Un demone (`ntkd`) che svolge le seguenti mansioni:

    *   Rileva i vicini.
    *   Comunicando con i vicini implementa il protocollo distribuito di routing.
    *   Assegna gli indirizzi al nodo e mantiene le tabelle di routing del kernel.
    *   Svolge il ruolo di server ANDNA, accedendo al database distribuito dei nomi.

    **Nota**: le mansioni del demone ntkd sono meglio illustrate all'inizio del documento
    [analisi funzionale](DemoneNTKD/AnalisiFunzionale.md).

*   Un resolver di nomi (`libnss_andna.so`) che interroga il server ANDNA su richiesta delle applicazioni che
    girano nel nodo, le quali lo richiamano in modo del tutto trasparente tramite l'uso del
    [Name Service Switch](https://en.wikipedia.org/wiki/Name_Service_Switch).

*   Un server (`dns-to-andna`) che traduce le richieste DNS in richieste ANDNA e interroga il server ANDNA.  
    Di norma questo è in esecuzione su un nodo sul quale gira anche il demone `ntkd` ed interroga il
    server ANDNA del nodo locale. Questo server `dns-to-andna` si rende necessario per questi casi:

    *   Applicazioni nel nodo stesso, che vogliono interrogare direttamente il DNS senza passare attraverso il NSS.
    *   Applicazioni nel nodo stesso, in un Sistema Operativo che non ha un NSS.
    *   Applicazioni in un nodo che fa parte di una sottorete a gestione autonoma (cioè sul quale non è in
        esecuzione la suite Netsukuku) che esce attraverso un gateway nella rete Netsukuku.

*   Un tool di risoluzione nomi (`ntk-resolv`) che interroga direttamente il server ANDNA. Può essere utile durante
    un troubleshooting in una rete.

*   Alcune librerie di supporto.

Oltre al software suddetto, per il corretto funzionamento di un nodo nella rete Netsukuku il sistema operativo
deve soddisfare alcuni requisiti. Nel documento [Requisiti](Sistema/Requisiti.md) prendiamo in
esame questi requisiti annotando per ogni sistema operativo quali sono le impostazioni da configurare per soddisfarli.

## ntkd

*   [Analisi funzionale](DemoneNTKD/AnalisiFunzionale.md)
*   [Dettagli Tecnici](DemoneNTKD/DettagliTecnici.md)

La logica delle operazioni svolte dal demone ntkd è stata suddivisa in moduli, ognuno dei quali si occupa di un
determinato aspetto. Essi sono isolati, indipendenti gli uni dagli altri. Questo rende più facile verificarne
il corretto funzionamento e apportare modifiche o correzioni.

Il software si compone dei moduli seguenti:

*   Neighborhood
    *   [Analisi funzionale](ModuloNeighborhood/AnalisiFunzionale.md)
    *   [Dettagli Tecnici](ModuloNeighborhood/DettagliTecnici.md)
*   Identities
    *   [Analisi funzionale](ModuloIdentities/AnalisiFunzionale.md)
    *   [Dettagli Tecnici](ModuloIdentities/DettagliTecnici.md)
*   QSPN
    *   [Analisi funzionale](ModuloQspn/AnalisiFunzionale.md)
    *   [Esplorazione rete](ModuloQspn/EsplorazioneRete.md)
    *   [Percorsi disgiunti](ModuloQspn/PercorsiDisgiunti.md)
    *   [Dettagli Tecnici](ModuloQspn/DettagliTecnici.md)
*   PeerServices
    *   [Analisi funzionale](ModuloPeers/AnalisiFunzionale.md)
    *   [Dettagli Tecnici](ModuloPeers/DettagliTecnici.md)
*   Coordinator
    *   [Analisi funzionale](ModuloCoordinator/AnalisiFunzionale.md)
    *   [Dettagli Tecnici](ModuloCoordinator/DettagliTecnici.md)
*   Hooking
    *   [Analisi funzionale](ModuloHooking/AnalisiFunzionale.md)
    *   [Dettagli Tecnici](ModuloHooking/DettagliTecnici.md)

Poi ci sono alcune librerie che offrono funzionalità che vengono usate da quasi tutti i moduli. Su queste librerie
tutti i moduli hanno una dipendenza.

*   [tasklet-system](Librerie/TaskletSystem.md). Con una dipendenza su questa libreria, un modulo può usare un generico sistema di
    tasklet (thread cooperativi) senza dover conoscere la specifica implementazione che il programma
    adotterà.
*   [ZCD](Librerie/ZCD.md). Questo framework è usato per realizzare il passaggio di messaggi tra nodi della rete.  
    Il framework comprende una libreria di basso livello, la quale però non è direttamente tra le dipendenze dei moduli
    o del demone ntkd. Infatti per usare questo framework, partendo da una definizione dei messaggi specifici
    dell'applicazione, si costruisce una libreria intermedia
    (nel nostro caso [ntkdrpc](https://github.com/lukisi/ntkdrpc/blob/master/build_sources/interfaces.rpcidl))
    che sarà tra le dipendenze dirette dei moduli e del demone ntkd.  
    I moduli utilizzano direttamente questa libreria in quanto chiamano i metodi degli stub e implementano i
    metodi delle classi che realizzano le interfacce skeleton
    ([classi stub e skeleton](https://en.wikipedia.org/wiki/Distributed_object_communication#Class_stubs_and_skeletons)).  
    Il demone ntkd utilizza direttamente questa libreria per costruire gli stub e invocare gli skeleton
    secondo le modalità descritte nel documento [RPC](DemoneNTKD/RPC.md).
*   [ntkd-common](Librerie/Common.md). Questa libreria contiene alcune classi e funzioni che sono note a più di un
    modulo.

#### Vecchi documenti (deprecati) del proof-of-concept

*   Proof-of-concept
    *   [Analisi funzionale](Proof/AnalisiFunzionale.md)
    *   [Dettagli Tecnici](Proof/DettagliTecnici.md)
    *   [Casi d'uso](Proof/UseCases.md)
    *   [Eventi](Proof/Eventi.md)
    *   [Operazioni](Proof/DettagliOperazioni1.md)
    *   [Indirizzi IP](Proof/IndirizziIP.md)

## libnss_andna

**TODO**

## dns-to-andna

Per i software che non supportano NSS o per offrire connettività a sistemi operativi al momento non supportati
dalla suite software Netsukuku.

