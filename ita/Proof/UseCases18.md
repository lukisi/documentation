# Proof of concept - Casi d'uso - Pagina 18

[Pagina precedente](UseCases17.md)

## Rimozione identità di connettività

Ora analiziamo alcune delle dinamiche nella rete che possono portare alla rimozione di identità *di connettività* sui sistemi.

Supponiamo che a causa di variazioni nelle condizioni atmosferiche, ora il sistema *𝜀* e il sistema *𝛾* sono a distanza di
rilevamento con la loro interfaccia "eth1". Lo visualiziamo nel disegno seguente con una linea verde.

##### <a name="grafo_3"></a>Grafo 3

![missing image](img/grafo3.png "grafo 3")

Siccome entrambi i sistemi hanno due identità devono valutare quali archi-identità vanno formati. L'identità principale di un
sistema è sempre disponibile a formare un arco-identità con qualsiasi altra identità. Una identità di supporto alla connettività
dei suoi g-nodi dal livello *i* al livello *j* è disposta a formare archi-identità solo con identità che appartengono al suo
stesso g-nodo di livello *j*. L'arco-identità tra due identità si forma solo se entrambe le identità sono disposte.

Analiziamo il nostro caso. Indichiamo con *𝜀<sub>P</sub>* e con *𝛾<sub>P</sub>* le due identità *principali*. Indichiamo con
*𝜀<sub>migr02</sub>* e con *𝛾<sub>migr02</sub>* le due identità nel namespace *migr02*.

*   Si forma l'arco-identità fra le due identità *principali* nel sistema *𝜀* e nel sistema *𝛾*.
*   L'identità *𝜀<sub>migr02</sub>* supporta la connettività di 1·. L'identità *𝛾<sub>P</sub>* invece
    appartiene a 0·. Quindi non si forma questo arco.
*   Analogamente, l'identità *𝛾<sub>migr02</sub>* supporta la connettività di 1·. L'identità *𝜀<sub>P</sub>*
    invece appartiene a 0·. Quindi non si forma questo arco.
*   L'identità *𝜀<sub>migr02</sub>* supporta la connettività di 1·. L'identità *𝛾<sub>migr02</sub>* appartiene a 1·,
    quindi *𝜀<sub>migr02</sub>* è disposta a formare l'arco.  
    Allo stesso tempo, *𝛾<sub>migr02</sub>* supporta la connettività di 1·. Siccome *𝜀<sub>migr02</sub>* appartiene a 1·,
    anche *𝛾<sub>migr02</sub>* è disposta a formare l'arco. Quindi si forma questo arco-identità.

Lo visualiziamo nel disegno seguente con una linea verde.

##### <a name="grafo_4"></a>Grafo 4

![missing image](img/grafo4.png "grafo 4")

[Pagina seguente](UseCases19.md)
