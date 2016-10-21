# Proof of concept - Dettagli Operazioni - Pagina 7

[Operazione precedente](DettagliOperazioni6.md)

### <a name="Migrazione_ingresso_1"></a> Migrazione per ingresso - Caso 1

Sia *𝜀* un sistema. In esso l'identità principale *𝜀<sub>0</sub>* costituisce una rete a sé. Esso si
incontra con un diverso sistema *𝛽*. In esso l'identità principale *𝛽<sub>1</sub>* appartiene (in una
diversa rete) ad un g-nodo di livello 1, *𝜑*, che è saturo.

Il modulo Neighborhood ha già prodotto questi comandi:

**sistema 𝜀**
```
ip route add 169.254.96.141 dev eth1 src 169.254.163.36
```

**sistema 𝛽**
```
ip route add 169.254.163.36 dev eth1 src 169.254.96.141
```

L'utente ha dato i comandi per accettare tale arco su entrambi i sistemi. Il modulo Identities ha
quindi realizzato per le relative identità principali un nuovo arco-identità *𝜀<sub>0</sub>*-*𝛽<sub>1</sub>*.
L'utente ha annotato l'identificativo del nuovo arco-identità nel sistema *𝜀* e il peer-MAC nel sistema *𝛽*.

L'utente stabilisce la migration path che porta a liberare un posto in *𝜑*. Il nodo *𝛽<sub>1</sub>* è
un border-nodo di *𝜑*: infatti esso ha un arco verso *𝛼<sub>1</sub>* (l'identità principale nel
sistema *𝛼*) che appartiene al g-nodo *𝜓* di livello 1 che ha un posto libero. L'utente quindi
si annota l'identificativo dell'arco-identità *𝛽<sub>1</sub>*-*𝛼<sub>1</sub>* nel sistema *𝛽* e
il relativo peer-MAC nel sistema *𝛼*.

La sequenza di istruzioni che l'utente darà ai sistemi sarà questa:

*   Al sistema *𝜀* dà il comando `prepare_enter_net_phase_1`, indicando queste informazioni:
    *   identità entrante: *𝜀<sub>0</sub>*. Sia il duplicato *𝜀<sub>1</sub>*.
    *   livello g-nodo entrante: 0.
    *   g-nodo ospitante *𝜑*:
        *   Livello: 1.
        *   Indirizzo Netsukuku: 2·1·1·.
        *   Fingerprint a livello 1.
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 3.
    *   nuova posizione reale:
        *   Identificativo: 1.
        *   Anzianità: 4.
    *   posizione di connettività.
        *   Identificativo: 2.
        *   Anzianità: 1.
    *   nuovi archi-qspn: 1.
        *   `identityarc_index`: l'identificativo dell'arco-identità *𝜀<sub>0</sub>*-*𝛽<sub>1</sub>*
            nel sistema *𝜀* prima della duplicazione dell'identità *𝜀<sub>0</sub>*.
    *   identificativo di questa operazione di ingresso: *m<sub>𝜀</sub>*.
    *   identificativo della previa operazione di migrazione: *m<sub>𝛽</sub>*.
*   Al sistema *𝜀* dà il comando `enter_net_phase_1`, indicando queste informazioni:
    *   si proceda con l'operazione di ingresso *m<sub>𝜀</sub>*.
*   Al sistema *𝛽* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝛽<sub>0</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità *𝜀<sub>0</sub>*-*𝛽<sub>1</sub>* nel
        sistema *𝛽* prima della duplicazione dell'identità *𝜀<sub>0</sub>*.
*   Al sistema *𝛽* dà il comando `prepare_migrate`, indicando queste informazioni:
    *   identità migrante: *𝛽<sub>1</sub>*. Sia il duplicato *𝛽<sub>2</sub>*.
    *   livello g-nodo migrante: 0.
    *   g-nodo ospitante *𝜓*:
        *   Livello: 1.
        *   Indirizzo Netsukuku: 2·0·1·.
        *   Fingerprint a livello 1.
    *   nuova posizione virtuale:
        *   Identificativo: 2.
        *   Anzianità: 3.
    *   nuova posizione reale:
        *   Identificativo: 1.
        *   Anzianità: 4.
    *   posizione di connettività.
        *   Identificativo: 3.
        *   Anzianità: 3.
    *   nuovi archi-qspn: 1.
        *   `identityarc_index`: l'identificativo dell'arco-identità *𝛽<sub>1</sub>*-*𝛼<sub>1</sub>*
            nel sistema *𝛽* prima della duplicazione dell'identità *𝛽<sub>1</sub>*.
    *   identificativo di questa operazione di ingresso: *m<sub>𝛽</sub>*.
    *   identificativo della previa operazione di migrazione: nullo.
*   Al sistema *𝛽* dà il comando `migrate`, indicando queste informazioni:
    *   si proceda con l'operazione di migrazione *m<sub>𝛽</sub>*.
*   Al sistema *𝛼* dà il comando `add_qspn_arc`, indicando queste informazioni:
    *   identità locale. *𝛼<sub>1</sub>*.
    *   nuovo arco-qspn. Il peer-MAC dell'arco-identità *𝛽<sub>1</sub>*-*𝛼<sub>1</sub>* nel
        sistema *𝛼* prima della duplicazione dell'identità *𝛽<sub>1</sub>*.
*   Al sistema *𝜀* dà il comando `enter_net_phase_2`, indicando queste informazioni:
    *   è stata completata la migrazione *m<sub>𝛽</sub>*; quindi è ora disponibile l'indirizzo *reale* dentro *𝜑*.

[Operazione seguente](DettagliOperazioni8.md)
