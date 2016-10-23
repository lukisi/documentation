# Proof of concept - Dettagli Operazioni - Pagina 7

[Operazione precedente](DettagliOperazioni6.md)

### <a name="Migrazione_ingresso_1"></a> Migrazione per ingresso - Caso 1

#### Comando prepare_migrate_phase_1 al sistema *𝛽*

Il programma **qspnclient** chiama il metodo `prepare_add_identity` del modulo Identities e memorizza le
informazioni relative alla migrazione.

#### Comando migrate_phase_1 al sistema *𝛽*

Il programma **qspnclient** recupera le informazioni relative alla migrazione e chiama il metodo `add_identity`
del modulo Identities. Il modulo Identities produce quindi queste operazioni:


[Operazione seguente](DettagliOperazioni8.md)
