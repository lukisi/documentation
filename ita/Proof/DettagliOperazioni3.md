# Proof of concept - Dettagli Operazioni - Pagina 3

[Operazione precedente](DettagliOperazioni2.md)

### <a name="Archi_vicini"></a> Archi con i sistemi vicini

Il modulo Neighborhood è quello che si occupa di rilevare i sistemi diretti vicini del sistema corrente
e di formare con essi degli archi. Il comando che realizza questo collegamento fra i vicini è comunque
eseguito dal programma **qspnclient** attraverso un delegato che questi fornisce al modulo Neighborhood.

Sia `$peerlinklocal` l'indirizzo IP link-local del sistema rilevato. Sia `$dev` l'interfaccia di rete
con cui è stato rilevato e `$linklocal` il relativo indirizzo di scheda. Il comando è:

```
ip route add $peerlinklocal dev $dev src $linklocal
```

Tuttavia, l'effettivo utilizzo dell'arco è deciso dal programma **qspnclient**. Il programma demanda all'utente
questa decisione. Questo permette all'utente di dirigere il proprio ambiente di test a piacimento anche in
particolari scenari, come ad esempio un gruppo di macchine virtuali che condividono un unico dominio di broadcast
ma vogliono simulare un gruppo di sistemi wireless disposti in un determinato modo.

Per ogni arco che il modulo Neighborhood realizza, le informazioni a disposizione (il nome dell'interfaccia
di rete reale nel sistema locale, il MAC address e il link-local dell'interfaccia di rete reale nel
sistema vicino) sono visualizzate all'utente.

Sempre per dare all'utente il maggior controllo possibile sulle dinamiche del test, anche il costo
di un arco che viene rilevato dal modulo Neighborhood non è lo stesso che viene usato dal programma
**qspnclient**.

Quando l'utente decide di accettare un arco lo comunica al programma **qspnclient** con il comando
`add_real_arc` e indica quale costo gli vuole associare. Se vuole in seguito variare il costo
lo può fare con il comando `change_real_arc`. Se vuole in seguito simularne la completa rimozione
lo può fare con il comando `remove_real_arc`.

Quando l'utente aggiunge un arco avviene nel programma **qspnclient** quello che nel vero demone *ntkd* avverrebbe
appena il modulo Neighborhood segnala la creazione di un arco.  
Quando l'utente varia il costo di un arco avviene nel programma **qspnclient** quello che nel vero demone *ntkd* avverrebbe
appena il modulo Neighborhood segnala la variazione di costo di un arco.  
Quando l'utente simula la rimozione di un arco avviene nel programma **qspnclient** quello che nel vero demone *ntkd* avverrebbe
appena il modulo Neighborhood segnala la rimozione di un arco.

Alla creazione di un arco (intesa come detto sopra) il programma **qspnclient** associa all'arco un indice
autoincrementante *nodearc_nextindex*, che parte da 0, e lo mostra all'utente. In seguito il programma
sfrutta questi archi passandoli al modulo Identities.

Il modulo Identities a sua volta segnala quando vengono creati o rimossi gli archi-identità. Nel segnalarlo
rende noto al programma:

*   L'identità nel sistema corrente.
*   L'arco fisico su cui poggia.
*   Il MAC address e il link-local dell'identità del vicino.

Il programma **qspnclient** associa ad ogni arco-identità un indice
autoincrementante *identityarc_nextindex*, che parte da 0, e lo mostra all'utente.

[Operazione seguente](DettagliOperazioni4.md)
