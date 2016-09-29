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

Per ogni arco che il modulo Neighborhood realizza, le informazioni a disposizione (i due link-local e
i due MAC address) sono visualizzate all'utente. Soltanto agli archi che l'utente decide di accettare
e nell'ordine in cui sono accettati, il programma associa un indice autoincrementante *nodearc_nextindex*,
che parte da 0. In seguito il programma sfrutta questi archi passandoli al modulo Identities.

Sempre per dare all'utente il maggior controllo possibile sulle dinamiche del test, anche il costo
di un arco che viene rilevato dal modulo Neighborhood non è lo stesso che viene usato dal programma
**qspnclient**. L'utente quando accetta un arco dice quale costo gli vuole associare. In seguito può
variarlo a piacimento fino anche a simularne la rimozione.

[Operazione seguente](DettagliOperazioni4.md)
