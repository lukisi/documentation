# Scenario #1

Il primo test consisterà in due nodi che si incontrano. Cioè una interfaccia
di rete del nodo A si collega ad un segmento a cui è collegata anche una interfaccia
di rete del nodo B. Per variare un po' diciamo che i due nodi hanno due interfacce
di rete.

Il test consiste nel simulare questo scenario:

```
          Nodo 123                          Nodo 456          
          --------                          --------          
   eth1  |        |  eth2            eth1  |        |  eth2   
  ---====|        |====----------------====|        |====-----
         |        |                        |        |         
          --------                          --------          
```

Il nome dell'eseguibile sarà `sys_ntkd_test1`.

Per rendere le operazioni riproducibili in modo deterministico
seguiamo sempre questa sequenza:

*   Nella directory dove si trova l'eseguibile, aprire 5 terminali.
*   Sul terminale #1 dare il comando `eth_domain -i 123_eth1 -v`.  
    Il processo resta appeso.
*   Sul terminale #2 dare il comando `eth_domain -i 456_eth2 -v`.  
    Il processo resta appeso.
*   Sul terminale #3 dare il comando `eth_domain -i 123_eth2 -i 456_eth1 -v`.  
    Il processo resta appeso.
*   Sul terminale #4 dare il comando `./sys_ntkd_test1 -p 123 -i eth1 -i eth2`.  
    Il processo produce alcuni output.
*   Sul terminale #5 dare il comando `./sys_ntkd_test1 -p 456 -i eth1 -i eth2`.  
    Il processo produce alcuni output.
*   Sul terminale #4 interrompiamo il processo.
*   Sul terminale #5 interrompiamo il processo.

Nel presente documento trattiamo la sequenza di operazioni che il programma fa in
questo caso in entrambi i nodi.

Si assume una conoscenza di quanto trattato nel
documento [Caso banale](CasoBanale.md). Verranno solo riportate le implementazioni
che sono state aggiunte rispetto a quello.

## Panoramica delle operazioni

Come nel caso banale fino alla costruzione della prima identità (sia nel nodo 123
che nel nodo 456) con le relative istanze dei vari manager dei moduli di identità.

In seguito all'incontro...


## Implementazione testsuite locale

Come per il caso banale, useremo il supporto del medium `system` fornito
dal framework ZCD.  
Il nome dell'eseguibile sarà `sys_ntkd_test1`.  
I file sorgente sono nella dir `sys-ntkd-test1`.

### Comunicazioni in rete

Diversamente dal caso banale:

*   Il programma si troverà a trasmettere in unicast (stream) alcuni
    messaggi, poiché il nodo costituirà un arco con un altro nodo.
*   Il programma si troverà a rilevare alcuni messaggi in broadcast (datagram)
    sull'una o l'altra interfaccia. Alcuni saranno messaggi provenienti dal
    nodo stesso **e** altri da un altro nodo.
*   Il programma si troverà a rilevare alcuni messaggi in unicast (stream).

* * *

Le parti dell'implementazione che riguardano le comunicazioni in rete sono trattate
nel documento [ComunicazioniRete](Scenario01ImplementazioneTestsuite/ComunicazioniRete.md).

### Identità e archi-identità

Come per il caso banale.

### Routine main

Come per il caso banale.

### Gestione segnali dai moduli

Come per il caso banale.

### Classi per l'integrazione dei moduli

Come per il caso banale.

#### Integrazione modulo Neighborhood

Le classi helper per il modulo `Neighborhood` sono trattate nel documento
[HelperNeighborhood](Scenario01ImplementazioneTestsuite/HelperNeighborhood.md).
