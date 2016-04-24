# Zero Configuration Dispatchers - Analisi Funzionale

## Obiettivo
I nodi di una rete Netsukuku per eseguire l'algoritmo distribuito di esplorazione della rete si devono passare dei messaggi.

Il passaggio di messaggi tra due nodi avviene sempre sotto forma di chiamata a una procedura (o metodo) remota. Si ha un
nodo *n* chiamante, cioè che inizia la comunicazione. Il messaggio che il nodo *n* trasmette è costituito dal metodo che
chiama e dai parametri che tale metodo accetta in ingresso.

**TODO**
