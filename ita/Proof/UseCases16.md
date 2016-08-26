# Proof of concept - Casi d'uso - Pagina 16

[Pagina precedente](UseCases15.md)

#### <a name="Costituzione_phi1"></a>Costituzione *𝜑<sub>1</sub>*

Nei sistemi che fanno parte del g-nodo *𝜑<sub>1</sub>*, le nuove identità *𝜀<sub>2</sub>* *𝛽<sub>3</sub>* e
*𝛾<sub>1</sub>* prendono possesso del precedente network namespace assumendo un indirizzo virtuale nel g-nodo
destinazione della migrazione, nel nostro caso 0·2·. Seguiamo i loro passi fino alla produzione del primo ETP.

I vari moduli QSPN hanno questi archi:

*   *𝜀<sub>2</sub>* (epsilon default) :
    *   *𝛽<sub>2</sub>* (beta default) - `MAC=00:16:3E:EC:A3:E1` `IP=169.254.96.141` esterno a *𝜑<sub>1</sub>*
    *   *𝜆<sub>1</sub>* (lambda default) - `MAC=00:16:3E:06:3E:90` `IP=169.254.109.22` esterno a *𝜑<sub>1</sub>*
    *   *𝛽<sub>3</sub>* (beta migr01) - `MAC=00:16:3E:EE:AF:D1` `IP=169.254.27.218` interno a *𝜑<sub>1</sub>*
*   *𝛽<sub>3</sub>* (beta migr01) :
    *   *𝜀<sub>2</sub>* (epsilon default) - `MAC=00:16:3E:3C:14:33` `IP=169.254.163.36` interno a *𝜑<sub>1</sub>*
    *   *𝛾<sub>1</sub>* (gamma default) - `MAC=00:16:3E:5B:78:D5` `IP=169.254.94.223` interno a *𝜑<sub>1</sub>*
*   *𝛾<sub>1</sub>* (gamma default) :
    *   *𝛽<sub>3</sub>* (beta migr01) - `MAC=00:16:3E:EE:AF:D1` `IP=169.254.27.218` interno a *𝜑<sub>1</sub>*
    *   *𝛽<sub>2</sub>* (beta default) - `MAC=00:16:3E:EC:A3:E1` `IP=169.254.96.141` esterno a *𝜑<sub>1</sub>*
    *   *𝛿<sub>1</sub>* (delta default) - `MAC=00:16:3E:1A:C4:45` `IP=169.254.253.216` esterno a *𝜑<sub>1</sub>*

Per gli archi esterni a *𝜑<sub>1</sub>* aspettiamo di avere un ETP da loro prima di usare le relative tabelle ntk_from_xxx.

**sistema 𝜀**
```
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.20/30 table ntk
ip route add unreachable 10.0.0.84/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.16/31 table ntk
ip route add unreachable 10.0.0.80/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.18/31 table ntk
ip route add unreachable 10.0.0.82/31 table ntk
ip route add unreachable 10.0.0.58/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:06:3E:90

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1
```

**sistema 𝛽**
```
ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.20/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.84/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.60/30 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.16/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.80/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.56/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.18/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.82/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.58/31 table ntk
ip netns exec migr01 ip route add unreachable 10.0.0.50/31 table ntk

ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33

ip netns exec migr01 ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
```

**sistema 𝛾**
```
ip route add unreachable 10.0.0.0/29 table ntk
ip route add unreachable 10.0.0.64/29 table ntk
ip route add unreachable 10.0.0.8/29 table ntk
ip route add unreachable 10.0.0.72/29 table ntk
ip route add unreachable 10.0.0.24/29 table ntk
ip route add unreachable 10.0.0.88/29 table ntk
ip route add unreachable 10.0.0.20/30 table ntk
ip route add unreachable 10.0.0.84/30 table ntk
ip route add unreachable 10.0.0.60/30 table ntk
ip route add unreachable 10.0.0.16/31 table ntk
ip route add unreachable 10.0.0.80/31 table ntk
ip route add unreachable 10.0.0.56/31 table ntk
ip route add unreachable 10.0.0.48/31 table ntk
ip route add unreachable 10.0.0.18/31 table ntk
ip route add unreachable 10.0.0.82/31 table ntk
ip route add unreachable 10.0.0.58/31 table ntk
ip route add unreachable 10.0.0.50/31 table ntk

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.18/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.82/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.58/31 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
```

**sistema 𝛽**
```
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.18/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.82/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.58/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33

ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.18/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.82/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.58/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
```

**sistema 𝜆**
```
(echo; echo "249 ntk_from_00:16:3E:3C:14:33 # xxx_table_ntk_from_00:16:3E:3C:14:33_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.20/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.84/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.60/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33
ip route add unreachable 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33
```

**sistema 𝛿**
```
(echo; echo "248 ntk_from_00:16:3E:5B:78:D5 # xxx_table_ntk_from_00:16:3E:5B:78:D5_xxx") | tee -a /etc/iproute2/rt_tables >/dev/null
ip route add unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip route add unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
```

**TODO**

#### <a name="Rimozione_archi_esterni_phi0"></a>Rimozione archi da *𝜑<sub>0</sub>* all'esterno

**TODO**

#### <a name="Cambio_indirizzo_phi1"></a>Cambio indirizzo per *𝜑<sub>1</sub>*

**TODO**


[Pagina seguente](UseCases17.md)
