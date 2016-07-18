# Proof of concept - Casi d'uso - Parte 4

## <a name="Altri_sistemi"></a>Negli altri sistemi

Prendiamo in esame quello che avviene negli altri sistemi.

*   Sistema *𝜇*. Prime operazioni e ingresso in *G<sub>𝛿</sub>*. [Dettagli](#step1).
*   Sistemi *𝛽* e *𝛾*. Parallelamente essi formano un g-nodo di livello 1 in *G<sub>𝛾</sub>*. [Dettagli](#step2).
*   Ingresso di *𝛿* e *𝜇* (come g-nodo *𝜒'*) in *G<sub>𝛾</sub>*. [Dettagli](#step3).
*   Ingresso di *𝛼* in *G<sub>𝛾</sub>*. [Dettagli](#step4).

## <a name="step1"></a>Sistema 𝜇

La prima identità è *𝜇<sub>0</sub>* con indirizzo Netsukuku 1·0·1·1 in *G<sub>𝜇</sub>*.

```
Mio indirizzo 1·0·1·1.
     globale
      10.0.0.11
     anonimizzante
      10.0.0.75
     interno al mio g-nodo di livello 3
      10.0.0.59
     interno al mio g-nodo di livello 2
      10.0.0.51
     interno al mio g-nodo di livello 1
      10.0.0.41

Possibili destinazioni:
 0·
     globale
      10.0.0.0/29
     anonimizzante
      10.0.0.64/29
 2·
     globale
      10.0.0.16/29
     anonimizzante
      10.0.0.80/29
 3·
     globale
      10.0.0.24/29
     anonimizzante
      10.0.0.88/29
 1·1·
     globale
      10.0.0.12/30
     anonimizzante
      10.0.0.76/30
     interno al mio g-nodo di livello 3
      10.0.0.60/30
 1·0·0·
     globale
      10.0.0.8/31
     anonimizzante
      10.0.0.72/31
     interno al mio g-nodo di livello 3
      10.0.0.56/31
     interno al mio g-nodo di livello 2
      10.0.0.48/31
 1·0·1·0
     globale
      10.0.0.10/32
     anonimizzante
      10.0.0.74/32
     interno al mio g-nodo di livello 3
      10.0.0.58/32
     interno al mio g-nodo di livello 2
      10.0.0.50/32
     interno al mio g-nodo di livello 1
      10.0.0.40/32
```

**sistema 𝜇**
```
ip link set dev eth1 address 00:16:3E:2D:8D:DE
ip link set dev eth1 up
ip address add 169.254.119.176 dev eth1
ip address add 10.0.0.11 dev eth1
ip address add 10.0.0.75 dev eth1
ip address add 10.0.0.59 dev eth1
ip address add 10.0.0.51 dev eth1
ip address add 10.0.0.41 dev eth1
(echo; echo "251 ntk # xxx_table_ntk_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip rule add table ntk
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.16/29 table ntk
ip route add unreachable 10.0.0.80/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.12/30 table ntk
ip route add unreachable 10.0.0.76/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.8/31 table ntk
ip route add unreachable 10.0.0.72/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.10/32 table ntk
ip route add unreachable 10.0.0.74/32 table ntk
ip route add unreachable 10.0.0.58/32 table ntk
ip route add unreachable 10.0.0.50/32 table ntk
ip route add unreachable 10.0.0.40/32 table ntk
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.29
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.16/29 table ntk
ip route change unreachable 10.0.0.80/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change unreachable 10.0.0.12/30 table ntk
ip route change unreachable 10.0.0.76/30 table ntk
ip route change unreachable 10.0.0.60/30 table ntk
ip route change unreachable 10.0.0.8/31 table ntk
ip route change unreachable 10.0.0.72/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change unreachable 10.0.0.10/32 table ntk
ip route change unreachable 10.0.0.74/32 table ntk
ip route change unreachable 10.0.0.58/32 table ntk
ip route change unreachable 10.0.0.50/32 table ntk
ip route change unreachable 10.0.0.40/32 table ntk


```

