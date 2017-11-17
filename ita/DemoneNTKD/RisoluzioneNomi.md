### <a name="Associazione_nomi_indirizzi"></a>Associazione nomi - indirizzi

La suite di software Netsukuku offre un meccanismo di risoluzione dei nomi degli host chiamato ANDNA (A Netsukuku
Domain Name Architecture).

ANDNA si propone come meccanismo alternativo o in aggiunta al classico DNS. Tra i due sistemi si evidenziano queste
differenze:

 * DNS è un sistema centralizzato. ANDNA è distribuito.
    *   Nel sistema DNS esiste un numero di server centrali, i root-nameserver. Solo questi sono autoritativi,
        senza di essi il sistema non può funzionare. Infatti, per risolvere il problema del Single Point Of Failure
        e assicurare la ridondanza, questi server sono molti e vanno tra loro coordinati. Una unica autorità centrale
        statunitense, ICANN, controllata dal Dipartimento del Commercio statunitense, gestisce le informazioni che
        sono su questi server.  
        I root-nameserver decidono quali siano i domini di più alto livello validi (.com, .edu, .it, ...) e per
        ognuno indicano quali altri server siano autoritativi per i nomi appartenenti a quel dominio.
    *   Nel sistema ANDNA non esistono server basilari. Ogni singolo nodo può essere chiamato a memorizzare una parte
        delle informazioni che costituiscono la mappatura nomi-indirizzi.  
        Il funzionamento del sistema è tale per cui viene assicurata ridondanza.  
        Le modifiche alla topologia della rete di fatto cambiano con frequenza il nodo che viene individuato come
        autoritativo per un certo nome, rendendo molto arduo un attacco.
 * DNS è un sistema fortemente gerarchico. ANDNA è flat.
    *   Nel sistema DNS i server centrali demandano la gestione di un dominio top-level (ad esempio .it) ad altri
        server. Questi a loro volta dicono quali sotto-domini esistono all'interno di .it e demandano ad altri server
        la gestione di ogni sotto-dominio. E così via.  
        Per ogni livello esiste quindi una gestione, cioè una autorità che ha il controllo. Di conseguenza una serie
        di adempimenti (almeno burocratici, spesso economici) che chi vuole registrare un nome di host o un dominio
        deve fare.
    *   Nel sistema ANDNA un nome di host è una sequenza di caratteri validi (lettere case-insensitive, numeri, pochi
        altri simboli) con una lunghezza massima prefissata.  
        Se un nome non è stato già assegnato, qualunque nodo può registrarlo per se nel database distribuito.
 * ANDNA reagisce molto più rapidamente di DNS agli aggiornamenti di una associazione nome-indirizzo.
    *   Nel sistema DNS se ogni singola richiesta fosse inviata ai server autoritativi questo comporterebbe un carico
        di lavoro troppo pesante per i root-nameservers e quindi un collo di bottiglia.  
        Per questo il sistema DNS prevede la presenza di numerosi *caching* DNS server. Questi memorizzano le passate
        ricerche e quando interrogati rispondono direttamente sulla base della loro cache. Soltanto periodicamente vanno
        ad aggiornare i record nella loro cache interrogando i server autoritativi.  
        Spesso ogni Internet Service Provider predispone un caching server che fornisce il servizio ai suoi clienti.  
        Questo comporta un ritardo, spesso di alcune ore o giorni, tra il momento in cui viene apportata una variazione
        ad una associazione nome-indirizzo e il momento in cui tutti i nodi di Internet vedono il nuovo indirizzo.
    *   Essendo ANDNA fortemente distribuito, il problema del collo di bottiglia presso i server autoritativi è
        drasticamente ridotto. Sono comunque previsti dei meccanismi di caching, soprattutto per migliorare le
        prestazioni del sistema, ma i tempi di aggiornamento delle cache possono essere ridotti di molto.  
        Questo comporta che al variare di una associazione nome-indirizzo, dopo tempi molto brevi possiamo presumere
        che tutti i nodi della rete vedranno il nuovo indirizzo.  
        Questa caratteristica, d'altra parte, è necessaria in una rete Netsukuku. Infatti in tale rete, poiché non ha
        bisogno di una entità centrale di coordinamento, ogni nodo può essere obbligato a cambiare il suo indirizzo
        in qualsiasi momento senza che questo comporti alcun intervento manuale di un amministratore.

Ogni singolo nodo della rete Netsukuku può cercare di registrare per se stesso un numero illimitato di nomi.
L'algoritmo distribuito consente ad ogni singolo nodo di vedere accettata la registrazione di un totale di 256 nomi
che non siano già stati assegnati ad altri nodi.

Quando un nodo ottiene la registrazione di un nome, questa rimane attiva per 30 giorni. Fino a quando la registrazione
è attiva nessun altro nodo potrà registrare per se lo stesso nome. La registrazione può essere aggiornata dal nodo
che la detiene, il quale si autentica al sistema distribuito ANDNA attraverso un meccanismo di cifratura asimmetrica
(chiave pubblica / chiave privata). Quindi il conteggio dei 30 giorni riparte da capo.

**Attenzione:** Un nodo può essere parte di una rete Netsukuku e fare allo stesso tempo parte anche di un'altra rete
TCP/IP privata o di Internet. In questo caso, oltre a usare il sistema ANDNA nella rete Netsukuku, può continuare ad
usare sull'altra rete il sistema DNS.

Tutto il back-end di ANDNA (la registrazione dei nomi, l'applicazione delle politiche delineate sopra, la ricerca dei
nomi) è realizzato attraverso due servizi peer-to-peer chiamati Andna e Counter. La spiegazione del loro funzionamento
non rientra nell'ambito di questo documento.

Il front-end di ANDNA, cioè quello che succede quando una applicazione chiede una informazione al cosidetto
*resolver* dei nomi del S.O. del nodo su cui è in esecuzione, si compone di due parti: una client e una server.

La parte server è realizzata dallo stesso demone *ntkd*. Esso riceve le richieste dalla parte client e risponde dopo
aver interrogato il back-end.

La parte client può essere interpretata da diversi moduli software, i quali si interfacciano con le applicazioni in
diversi modi a seconda dei meccanismi messi a disposizione dal sistema operativo. I moduli client forniti dalla suite
Netsukuku (`libnss_andna`, `dns-to-andna`, `ntk-resolv`) sono descritti in dettaglio in un altro documento.

