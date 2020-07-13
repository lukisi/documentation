# Classi helper modulo Identities

Come per il caso banale. Eccetto quanto segue.

Mentre nel caso banale non è stato necessario definire la classe `N.IdmgmtArc`,
che implementa l'interfaccia `N.I.IIdmgmtArc`, in questo scenario essa viene definita
per rappresentare un arco tra due nodi diretti vicini.

I diversi metodi richiesti da questa interfaccia sono eseguiti in questa classe
usando le conoscenze del modulo `Neighborhood`.