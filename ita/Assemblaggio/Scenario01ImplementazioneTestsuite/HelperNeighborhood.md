# Classi helper modulo Neighborhood

Come per il caso banale. Eccetto quanto segue.

Mentre nel caso banale la classe
`N.NeighborhoodStubFactory` nel metodo `get_unicast` il codice semplicemente
va in errore, in questo scenario il metodo deve
restituire uno stub per mandare messaggi unicast allo stesso modulo `Neighborhood`
in uno specifico nodo diretto vicino attraverso uno specifico
arco `N.N.INeighborhoodArc`.  
Per farlo chiama il metodo `get_stub_whole_node_unicast` di `N.StubFactory` e
ottiene uno stub radice. Con esso poi costruisce una istanza
di `NeighborhoodManagerStubHolder`.

