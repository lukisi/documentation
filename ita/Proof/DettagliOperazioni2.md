# Proof of concept - Dettagli Operazioni - Pagina 2

[Operazione precedente](DettagliOperazioni1.md)

### <a name="Primo_bootstrap_complete"></a> Primo segnale `bootstrap_complete`

Immediatamente, poiché il sistema è inizialmente isolato, l'identità principale riceve dal QspnManager
il segnale `bootstrap_complete`.

Alla ricezione del segnale `bootstrap_complete` (da una qualsiasi identità) il programma **qspnclient**
computa gli indirizzi IP di tutte le possibili destinazioni relative all'indirizzo Netsukuku di
quella identità. Per ognuno, in base alle conoscenze di quella identità, aggiorna la rotta
nelle tabelle (`ntk*`) presenti nel network namespace di gestione di quella identità.

Nel presente caso assisteremo ad un aggiornamento della sola tabella `ntk`, in cui tutte le destinazioni
sono irraggiungibili. Ad esempio:

```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.28/30 table ntk
ip route change unreachable 10.0.0.92/30 table ntk
ip route change unreachable 10.0.0.60/30 table ntk
ip route change unreachable 10.0.0.24/31 table ntk
ip route change unreachable 10.0.0.88/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.27/32 table ntk
ip route change unreachable 10.0.0.91/32 table ntk
ip route change unreachable 10.0.0.59/32 table ntk
ip route change unreachable 10.0.0.51/32 table ntk
ip route change unreachable 10.0.0.41/32 table ntk
```

Questo blocco di comandi va eseguito senza intromissione di altri comandi da altre tasklet.

[Operazione seguente](DettagliOperazioni3.md)
