# Modulo Migrations - Dettagli Tecnici

1.  [Operazioni relative ad un arco-identità](#Operazioni_arco_identita)

## <a name="Operazioni_arco_identita"></a>Operazioni relative ad un arco-identità

Il modulo migrations, quando aggiunge un arco-identità, avvia una nuova tasklet. In essa eseguirà
tutte le operazioni relative a quell'arco-identità. Quando rimuove un arco-identità abortisce
la tasklet relativa.

La tasklet per prima cosa esamina il tipo di identità su cui è in esecuzione. Se si tratta di una
identità *di connettività* la tasklet termina. Se si tratta di una identità *principale* ma con
indirizzo *virtuale* la tasklet attende un tempo breve e poi guarda di nuovo. Se si tratta di una
identità *principale* con indirizzo *reale* allora prosegue. Questa verifica viene fatta all'inizio
e in seguito ogni volta che la tasklet intende chiamare il metodo remoto `retrieve_network_data`.

La tasklet chiama il metodo remoto `retrieve_network_data` sul suo arco-identità. La risposta può essere
l'eccezione `NotPrincipalError` oppure l'eccezione `VirtualAddressError` oppure informazioni sulla rete
di appartenenza del vicino.

Se rileva che l'identità vicina è *di connettività*, allora la tasklet termina.

Se rileva una identità *principale* ma con indirizzo *virtuale*, allora attende un tempo breve e poi
ripete la stessa operazione (cioè chiama di nuovo il metodo remoto `retrieve_network_data` dopo aver
verificato il tipo di identità su cui è in esecuzione). Infatti l'identità vicina dovrebbe presto
assumere un indirizzo *reale*.

Se l'identità vicina appartiene alla nostra stessa rete, allora attende un tempo lungo (10 minuti) prima
di ripetere la stessa operazione. Infatti l'identità vicina potrebbe diventare di un'altra rete.

Se l'identità vicina appartiene ad una rete diversa che ha una topologia diversa, allora la tasklet termina.

Se l'identità vicina appartiene ad una rete diversa che ha la stessa topologia, allora la tasklet procede
con la valutazione.




