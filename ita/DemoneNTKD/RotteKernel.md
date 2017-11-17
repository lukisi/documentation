### <a name="Stack_tcpip"></a>Stack TCP/IP

Il software nella suite Netsukuku non intende sostituire lo stack TCP/IP implementato nel S.O. del nodo. Nemmeno
intende interagire con le librerie TCP/IP del S.O. attraverso sistemi di hook.

Quando un'applicazione vuole avviare una connessione TCP con un indirizzo, o inviare un pacchetto di dati UDP ad un
nodo, o mettersi in ascolto, o qualsiasi altra operazione che coinvolge la rete TCP/IP, questi eventi non sono in
alcun modo notificati al software Netsukuku. Sono invece completamente gestiti alla maniera classica dalle librerie
del S.O. del nodo.

Infatti non è in questo momento che il software Netsukuku intende operare, ma preventivamente. Le sue operazioni
mirano a popolare in anticipo le tabelle di routing del nodo con le rotte di sua pertinenza e mantenerle aggiornate.
Si definiscono rotte di pertinenza di Netsukuku tutte le rotte necessarie a che ogni singolo nodo esistente nella
rete Netsukuku possa essere raggiunto e che ogni indirizzo nello spazio di Netsukuku che non è stato assegnato ad
alcun nodo sia riportato come "irraggiungibile".

Il demone *ntkd* mantiene le rotte di pertinenza di Netsukuku nelle tabelle di routing. Decide quali rotte impostare
sulla base delle informazioni che ha raccolto tramite l'esplorazione della rete.

Per ogni destinazione, o classe di indirizzi *d* di cui è venuto a conoscenza, considerando l'insieme *P* dei percorsi
che sa di poter usare per raggiungerla, il demone *ntkd* mantiene nelle tabelle di routing:

*   La rotta ( *gw* + *nic* + *d* ) che riporta i dati relativi al miglior percorso *p* ∈ *P* , cioè quello con costo
    minore.
*   Per ogni vicino *v* del nodo:
    *   La rotta con i dati del miglior percorso *p<sub>v</sub>* ∈ *P* che non ha fra i suoi passi la classe di
        indirizzi a cui appartiene *v* . Questa sarà usata per i pacchetti che il nodo ha ricevuto dal vicino *v* e
        deve inoltrare alla destinazione *d* .

