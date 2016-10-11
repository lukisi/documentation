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

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249
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

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 248
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

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:1A:C4:45 -j MARK --set-mark 249
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:3C:14:33 -j MARK --set-mark 248
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

iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 250
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
iptables -t mangle -A PREROUTING -m mac --mac-source 00:16:3E:5B:78:D5 -j MARK --set-mark 248
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

Ora il g-nodo *𝜑<sub>1</sub>* è in fase di bootstrap. I suoi border-nodi ricevono i primi ETP
completi dai vicini esterni a *𝜑<sub>1</sub>*, poi li propagano ai nodi interni di *𝜑<sub>1</sub>*.
Gradualmente tutti i singoli nodi di *𝜑<sub>1</sub>* escono dalla fase di bootstrap, popolano le loro
rotte, producono altri ETP.

*   *𝜀<sub>2</sub>* riceve un ETP da *𝛽<sub>2</sub>*.
*   *𝜀<sub>2</sub>* popola la sua mappa e esce dal bootstrap.
*   *𝜀<sub>2</sub>* trasmette un ETP a *𝛽<sub>2</sub>*.
*   *𝛽<sub>2</sub>* aggiorna la sua mappa.
*   *𝜀<sub>2</sub>* trasmette un ETP a *𝛽<sub>3</sub>*.
*   *𝛽<sub>3</sub>* popola la sua mappa e esce dal bootstrap.
*   *𝛾<sub>1</sub>* riceve un ETP da *𝛽<sub>2</sub>*.
*   *𝛾<sub>1</sub>* popola la sua mappa e esce dal bootstrap.
*   ...

Per brevità indichiamo le varie operazioni nei sistemi come se avvenissero tutte alla fine della
propagazione di tutti gli ETP.  
Ricordiamo che quando si aggiornano le rotte per i pacchetti da inoltrare provenienti da un dato
arco si fa soltanto per gli archi tramite i quali si è già ricevuto almeno un ETP. Inoltre se questo
è il primo aggiornamento dopo aver ricevuto il primo ETP da quell'arco, allora si aggiunge la regola.

**sistema 𝛽**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.241.153 dev eth1 src 10.0.0.19
ip route change 10.0.0.84/30 table ntk via 169.254.241.153 dev eth1 src 10.0.0.19
ip route change 10.0.0.60/30 table ntk via 169.254.241.153 dev eth1 src 10.0.0.59
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.82/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.58/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.59
ip route change 10.0.0.50/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.241.153 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.241.153 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.241.153 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.18/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.82/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.58/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:AF:4C:2A
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:AF:4C:2A

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3B:9F:45
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3B:9F:45
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:3B:9F:45
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:3B:9F:45 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:3B:9F:45 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:3B:9F:45 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.50/32 table ntk_from_00:16:3E:3B:9F:45
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:3B:9F:45

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.241.153 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.241.153 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.241.153 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip rule add fwmark 250 table ntk_from_00:16:3E:5B:78:D5

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.241.153 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.241.153 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.241.153 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
ip rule add fwmark 248 table ntk_from_00:16:3E:3C:14:33
```

**sistema 𝛿**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.24.198 dev eth1 src 10.0.0.21
ip route change 10.0.0.80/30 table ntk via 169.254.24.198 dev eth1 src 10.0.0.21
ip route change 10.0.0.56/30 table ntk via 169.254.24.198 dev eth1 src 10.0.0.61
ip route change unreachable 10.0.0.22/31 table ntk
ip route change unreachable 10.0.0.86/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.119.176 dev eth1
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.119.176 dev eth1
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.119.176 dev eth1
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.119.176 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:AF:4C:2A

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.24.198 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.24.198 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.24.198 dev eth1
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change blackhole 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip rule add fwmark 248 table ntk_from_00:16:3E:5B:78:D5
```

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.84/30 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.60/30 table ntk via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.41/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.40

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
ip rule add fwmark 248 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:1A:C4:45
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
ip rule add fwmark 249 table ntk_from_00:16:3E:1A:C4:45
```

**sistema 𝜀**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.84/30 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.60/30 table ntk via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1
ip rule add fwmark 249 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EE:AF:D1
```

**sistema 𝛽**
```
ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change 10.0.0.18/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.82/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.58/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change 10.0.0.18/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.82/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.58/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
```

#### <a name="Rimozione_archi_esterni_phi0"></a>Rimozione archi da *𝜑<sub>0</sub>* all'esterno

Ogni border-nodo del g-nodo *𝜑* con la vecchia identità che ora è di connettività,
dopo qualche istante dalla trasmissione del primo ETP da parte della nuova identità
che l'ha sostituita, rimuove i suoi archi esterni ai g-nodi di cui
supporta la connettività. Nel nostro caso il nodo *𝜀<sub>1</sub>* rimuove il suo arco
verso *𝛽<sub>2</sub>*. Analogamente il nodo *𝛾<sub>0</sub>* rimuove il suo arco
verso *𝛽<sub>2</sub>*. Questi sono i dati:

*   *𝜀<sub>1</sub>* (epsilon migr02) - `MAC=00:16:3E:3B:9F:45` `IP=169.254.241.153`
*   *𝛾<sub>0</sub>* (gamma migr02) - `MAC=00:16:3E:AF:4C:2A` `IP=169.254.24.198`
*   *𝛽<sub>2</sub>* (beta default) - `MAC=00:16:3E:EC:A3:E1` `IP=169.254.96.141`

In ogni sistema, dopo aver rimosso l'arco nel modulo QSPN e aver ricalcolato
i migliori percorsi verso le possibili destinazioni, le operazioni che il programma *qspnclient*
fa sono:

*   cambio rotte nelle tabelle
*   rimozione tabella dell'arco (`ntk_from_xxx`) e relativa regola e marcamento
*   rimozione arco (realizzata dal modulo Identities)

**sistema 𝜀**
```
ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:06:3E:90
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:06:3E:90

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98

ip netns exec migr02 ip rule del fwmark 249 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route flush table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 249

ip netns exec migr02 ip route del 169.254.96.141 dev migr02_eth1 src 169.254.241.153
```

**sistema 𝛾**
```
ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:1A:C4:45
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk_from_00:16:3E:BD:34:98 via 169.254.253.216 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:BD:34:98
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:BD:34:98

ip netns exec migr02 ip rule del fwmark 248 table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 ip route flush table ntk_from_00:16:3E:EC:A3:E1
ip netns exec migr02 iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:EC:A3:E1 -j MARK --set-mark 248

ip netns exec migr02 ip route del 169.254.96.141 dev migr02_eth1 src 169.254.24.198
```

**sistema 𝛽**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.19
ip route change 10.0.0.84/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.19
ip route change 10.0.0.60/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.59
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.82/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.58/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.59
ip route change 10.0.0.50/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.18/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.82/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.58/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33

ip rule del fwmark 246 table ntk_from_00:16:3E:3B:9F:45
ip route flush table ntk_from_00:16:3E:3B:9F:45
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:3B:9F:45 -j MARK --set-mark 246

ip rule del fwmark 247 table ntk_from_00:16:3E:AF:4C:2A
ip route flush table ntk_from_00:16:3E:AF:4C:2A
iptables -t mangle -D PREROUTING -m mac --mac-source 00:16:3E:AF:4C:2A -j MARK --set-mark 247

ip route del 169.254.24.198 dev eth1 src 169.254.96.141
ip route del 169.254.241.153 dev eth1 src 169.254.96.141
```

Poi ogni nodo, avendo rimosso un suo arco, comunica le variazioni apportate alla sua mappa tramite
un ETP agli altri vicini. La propagazione di questi ETP aggiorna le mappe di diversi nodi.

**sistema 𝛽**
```
ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.20/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.84/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.60/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:AF:4C:2A
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:AF:4C:2A

ip netns exec migr02 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change 10.0.0.16/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.80/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.56/30 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.20/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.84/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.60/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change 10.0.0.48/31 table ntk_from_00:16:3E:3B:9F:45 via 169.254.24.198 dev migr02_eth1
ip netns exec migr02 ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:3B:9F:45
ip netns exec migr02 ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:3B:9F:45
```

**sistema 𝜀**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.84/30 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.60/30 table ntk via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:EE:AF:D1
```

**sistema 𝛾**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.84/30 table ntk via 169.254.253.216 dev eth1
ip route change 10.0.0.60/30 table ntk via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk
ip route change unreachable 10.0.0.80/31 table ntk
ip route change unreachable 10.0.0.56/31 table ntk
ip route change unreachable 10.0.0.48/31 table ntk
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1
ip route change 10.0.0.41/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.40

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change 10.0.0.50/31 table ntk_from_00:16:3E:EE:AF:D1 via 169.254.96.141 dev eth1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EE:AF:D1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.253.216 dev eth1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.18/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.82/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.58/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:1A:C4:45
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:1A:C4:45
ip route change blackhole 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.18/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.82/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change 10.0.0.58/31 table ntk_from_00:16:3E:1A:C4:45 via 169.254.96.141 dev eth1
ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:1A:C4:45
ip route change blackhole 10.0.0.41/32 table ntk_from_00:16:3E:1A:C4:45
```

**sistema 𝛽**
```
ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change 10.0.0.18/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.82/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.58/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.50/31 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:3C:14:33

ip netns exec migr01 ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change 10.0.0.18/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.82/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.58/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change 10.0.0.41/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
```

**sistema 𝛿**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.16/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.80/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.21
ip route change 10.0.0.56/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.61
ip route change unreachable 10.0.0.22/31 table ntk
ip route change unreachable 10.0.0.86/31 table ntk
ip route change unreachable 10.0.0.62/31 table ntk
ip route change unreachable 10.0.0.50/31 table ntk
ip route change 10.0.0.20/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.84/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.21
ip route change 10.0.0.60/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.61
ip route change 10.0.0.48/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.119.176 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:AF:4C:2A
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.94.223 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.94.223 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:AF:4C:2A via 169.254.94.223 dev eth1
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:AF:4C:2A
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.119.176 dev eth1
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.119.176 dev eth1
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.119.176 dev eth1
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:AF:4C:2A via 169.254.119.176 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:AF:4C:2A

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:2D:8D:DE
ip route change 10.0.0.16/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change 10.0.0.80/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change 10.0.0.56/30 table ntk_from_00:16:3E:2D:8D:DE via 169.254.94.223 dev eth1
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.50/31 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.20/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.84/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.60/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:2D:8D:DE
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:2D:8D:DE

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.22/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.86/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.62/31 table ntk_from_00:16:3E:5B:78:D5
ip route change blackhole 10.0.0.50/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.20/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change 10.0.0.84/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change 10.0.0.60/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.119.176 dev eth1
ip route change blackhole 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5
```

#### <a name="Cambio_indirizzo_phi1"></a>Cambio indirizzo per *𝜑<sub>1</sub>*

Ora le nuove identità in *𝜑<sub>1</sub>* cambiano indirizzo: il loro identificativo al livello
del g-nodo che ha migrato diventa un identificativo reale nel g-nodo destinazione della migrazione,
nel nostro caso passano da 0·2· a 0·0·.

Ricordiamo che nel nostro esempio queste nuove identità sono:

*   *𝜀<sub>2</sub>* - network namespace default del sistema epsilon - indirizzo 0·0·1.
*   *𝛽<sub>3</sub>* - network namespace migr01 del sistema beta - indirizzo 0·0·3.
*   *𝛾<sub>1</sub>* - network namespace default del sistema gamma - indirizzo 0·0·0.

Come conseguenza, per ogni nuova identità il programma *qspnclient* fa queste operazioni:

*   Rimuove da tutte le tabelle le rotte verso i vari indirizzi IP (globale, anonimizzante, interni)
    riferiti alla precedente possibile destinazione che ora non è più valida, cioè il nuovo
    identificativo reale appena assegnato a *𝜑<sub>1</sub>*.
*   Aggiunge su tutte le tabelle (dapprima unreachable e poi aggiorna) le rotte verso i vari
    indirizzi IP (globale, anonimizzante, interni) riferiti alle destinazioni interne a *𝜑<sub>1</sub>*
    e che prima non erano valorizzate perché l'identificativo al livello di *𝜑<sub>1</sub>* era
    virtuale.
*   Se l'identità è nel network namespace default, aggiunge gli indirizzi IP che prima non aveva.
*   Se l'identità è nel network namespace default, aggiorna tutte le rotte nella per i pacchetti
    originati localmente (`ntk`) di modo da usare i nuovi indirizzi come *src*.

**sistema 𝜀**
```
ip route del 10.0.0.16/31 table ntk
ip route del 10.0.0.80/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:06:3E:90
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:06:3E:90

ip route add unreachable 10.0.0.16/32 table ntk
ip route add unreachable 10.0.0.80/32 table ntk
ip route add unreachable 10.0.0.56/32 table ntk
ip route add unreachable 10.0.0.48/32 table ntk
ip route change 10.0.0.16/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.80/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.56/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.48/32 table ntk via 169.254.27.218 dev eth1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.16/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.80/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.56/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.48/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.16/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.80/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.56/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:EE:AF:D1

ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:06:3E:90
ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:06:3E:90

ip address add 10.0.0.17 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.17
ip address add 10.0.0.81 dev eth1
ip address add 10.0.0.57 dev eth1
ip address add 10.0.0.49 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.84/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.60/30 table ntk via 169.254.27.218 dev eth1 src 10.0.0.57
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.17
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.17
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.57
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.49
ip route change 10.0.0.16/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.80/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.17
ip route change 10.0.0.56/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.57
ip route change 10.0.0.48/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.49
ip route change 10.0.0.40/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.41
```

**sistema 𝛽**
```
ip netns exec migr01 ip route del 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route del 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route del 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route del 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33

ip netns exec migr01 ip route del 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route del 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route del 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route del 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5

ip netns exec migr01 ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.17/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.81/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.57/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change 10.0.0.16/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.80/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.56/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.48/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.94.223 dev migr01_eth1
ip netns exec migr01 ip route change unreachable 10.0.0.17/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.81/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.57/32 table ntk_from_00:16:3E:3C:14:33
ip netns exec migr01 ip route change unreachable 10.0.0.49/32 table ntk_from_00:16:3E:3C:14:33

ip netns exec migr01 ip route add unreachable 10.0.0.16/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.80/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.56/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.17/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.81/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.57/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.16/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.80/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.56/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change unreachable 10.0.0.48/32 table ntk_from_00:16:3E:5B:78:D5
ip netns exec migr01 ip route change 10.0.0.17/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.81/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.57/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
ip netns exec migr01 ip route change 10.0.0.49/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.163.36 dev migr01_eth1
```

**sistema 𝛾**
```
ip route del 10.0.0.16/31 table ntk
ip route del 10.0.0.80/31 table ntk
ip route del 10.0.0.56/31 table ntk
ip route del 10.0.0.48/31 table ntk

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:EE:AF:D1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EE:AF:D1

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1

ip route del 10.0.0.16/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.80/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.56/31 table ntk_from_00:16:3E:1A:C4:45
ip route del 10.0.0.48/31 table ntk_from_00:16:3E:1A:C4:45

ip route add unreachable 10.0.0.17/32 table ntk
ip route add unreachable 10.0.0.81/32 table ntk
ip route add unreachable 10.0.0.57/32 table ntk
ip route add unreachable 10.0.0.49/32 table ntk
ip route change 10.0.0.17/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.81/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.57/32 table ntk via 169.254.27.218 dev eth1
ip route change 10.0.0.49/32 table ntk via 169.254.27.218 dev eth1

ip route add unreachable 10.0.0.17/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.81/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.57/32 table ntk_from_00:16:3E:EE:AF:D1
ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.17/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.81/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.57/32 table ntk_from_00:16:3E:EE:AF:D1
ip route change unreachable 10.0.0.49/32 table ntk_from_00:16:3E:EE:AF:D1

ip route add unreachable 10.0.0.17/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.81/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.57/32 table ntk_from_00:16:3E:EC:A3:E1
ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change 10.0.0.17/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.81/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.57/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1
ip route change 10.0.0.49/32 table ntk_from_00:16:3E:EC:A3:E1 via 169.254.27.218 dev eth1

ip route add unreachable 10.0.0.17/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.81/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.57/32 table ntk_from_00:16:3E:1A:C4:45
ip route add unreachable 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45
ip route change 10.0.0.17/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.81/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change 10.0.0.57/32 table ntk_from_00:16:3E:1A:C4:45 via 169.254.27.218 dev eth1
ip route change blackhole 10.0.0.49/32 table ntk_from_00:16:3E:1A:C4:45

ip address add 10.0.0.16 dev eth1
iptables -t nat -A POSTROUTING -d 10.0.0.64/27 -j SNAT --to 10.0.0.16
ip address add 10.0.0.80 dev eth1
ip address add 10.0.0.56 dev eth1
ip address add 10.0.0.48 dev eth1

ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.16
ip route change 10.0.0.84/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.16
ip route change 10.0.0.60/30 table ntk via 169.254.253.216 dev eth1 src 10.0.0.56
ip route change 10.0.0.18/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.16
ip route change 10.0.0.82/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.16
ip route change 10.0.0.58/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.56
ip route change 10.0.0.50/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.48
ip route change 10.0.0.17/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.16
ip route change 10.0.0.81/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.16
ip route change 10.0.0.57/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.56
ip route change 10.0.0.49/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.48
ip route change 10.0.0.41/32 table ntk via 169.254.27.218 dev eth1 src 10.0.0.40
```

Le nuove identità in *𝜑<sub>1</sub>* comunicano via ETP ai loro vicini il loro nuovo indirizzo, cioè che non è più possibile
raggiungere tramite loro l'indirizzo 0·2·X; ma che ora è possibile raggiungere tramite loro l'indirizzo 0·0·X.

La propagazione di questi ETP è comunque limitata ai nodi del g-nodo di livello 2, cioè apporta variazioni alle mappe
del nodo *𝛽* e *𝛼*.

**sistema 𝛽**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.19
ip route change 10.0.0.84/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.19
ip route change 10.0.0.60/30 table ntk via 169.254.94.223 dev eth1 src 10.0.0.59
ip route change 10.0.0.16/31 table ntk via 169.254.163.36 dev eth1 src 10.0.0.19
ip route change 10.0.0.80/31 table ntk via 169.254.163.36 dev eth1 src 10.0.0.19
ip route change 10.0.0.56/31 table ntk via 169.254.163.36 dev eth1 src 10.0.0.59
ip route change 10.0.0.48/31 table ntk via 169.254.163.36 dev eth1 src 10.0.0.51
ip route change 10.0.0.18/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.82/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.19
ip route change 10.0.0.58/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.59
ip route change 10.0.0.50/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.51
ip route change 10.0.0.40/32 table ntk via 169.254.69.30 dev eth1 src 10.0.0.41

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:FD:E2:AA
ip route change 10.0.0.20/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.84/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.60/30 table ntk_from_00:16:3E:FD:E2:AA via 169.254.94.223 dev eth1
ip route change 10.0.0.16/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.163.36 dev eth1
ip route change 10.0.0.80/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.163.36 dev eth1
ip route change 10.0.0.56/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.163.36 dev eth1
ip route change 10.0.0.48/31 table ntk_from_00:16:3E:FD:E2:AA via 169.254.163.36 dev eth1
ip route change unreachable 10.0.0.18/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.82/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.58/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.50/32 table ntk_from_00:16:3E:FD:E2:AA
ip route change unreachable 10.0.0.40/32 table ntk_from_00:16:3E:FD:E2:AA

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:5B:78:D5
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:5B:78:D5
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change 10.0.0.50/32 table ntk_from_00:16:3E:5B:78:D5 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:5B:78:D5

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:3C:14:33
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:3C:14:33
ip route change 10.0.0.18/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.82/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.58/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change 10.0.0.50/32 table ntk_from_00:16:3E:3C:14:33 via 169.254.69.30 dev eth1
ip route change blackhole 10.0.0.40/32 table ntk_from_00:16:3E:3C:14:33
```

**sistema 𝛼**
```
ip route change unreachable 10.0.0.0/29 table ntk
ip route change unreachable 10.0.0.64/29 table ntk
ip route change unreachable 10.0.0.8/29 table ntk
ip route change unreachable 10.0.0.72/29 table ntk
ip route change unreachable 10.0.0.24/29 table ntk
ip route change unreachable 10.0.0.88/29 table ntk
ip route change 10.0.0.20/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.84/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.60/30 table ntk via 169.254.96.141 dev eth1 src 10.0.0.58
ip route change 10.0.0.16/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.80/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.56/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.58
ip route change 10.0.0.48/31 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change 10.0.0.19/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.83/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.18
ip route change 10.0.0.59/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.58
ip route change 10.0.0.51/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.50
ip route change 10.0.0.41/32 table ntk via 169.254.96.141 dev eth1 src 10.0.0.40

ip route change unreachable 10.0.0.0/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.64/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.8/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.72/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.24/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.88/29 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.20/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.84/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.60/30 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.16/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.80/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.56/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.48/31 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.19/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.83/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.59/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.51/32 table ntk_from_00:16:3E:EC:A3:E1
ip route change unreachable 10.0.0.41/32 table ntk_from_00:16:3E:EC:A3:E1
```

[Pagina seguente](UseCases17.md)
