# TP5 - RIPng EIGRP

## Objectifs
* Comprendre les mécanismes de RIPng et EIGRP
* Configurer RIPng pour IPv6
* Configurer EIGRP pour IPv4
* Analyser le comportement des protocoles lors de changements topologiques
* Comparer les caractéristiques des deux protocoles

# RIPng

## Topologie RIPng
![image](https://github.com/user-attachments/assets/32567ad2-d68c-4564-bc52-301ab57ab0d0)

## Plan d’adressage IPv6
* **Liens entre routeurs :** `2001:db8:1::/64` à `2001:db8:8::/64`
* **LAN1 (`R1`) :** `2001:db8:A::/64`
* **LAN2 (`R2`) :** `2001:db8:B::/64`
* **LAN3 (`R5`) :** `2001:db8:C::/64`

## 1. Configurer les adresses IPv6 sur toutes les interfaces des routeurs
### R1
```
ipv6 unicast-routing
ipv6 router rip LAB-RIPng
interface GigabitEthernet0/0
  no shut
  ipv6 address 2001:db8:1::1/64
  ipv6 address FE80::1 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet1/0
  no shut
  ipv6 address 2001:db8:2::1/64
  ipv6 address FE80::2 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet2/0
  no shut
  ipv6 address 2001:db8:3::1/64
  ipv6 address FE80::3 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet3/0
  no shut
  ipv6 address 2001:db8:A::1/64
  ipv6 address FE80::4 link-local
  ipv6 rip LAB-RIPng enable
```
### R2
```
ipv6 unicast-routing
ipv6 router rip LAB-RIPng
interface GigabitEthernet0/0
  no shut
  ipv6 address 2001:db8:1::2/64
  ipv6 address FE80::5 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet1/0
  no shut
  ipv6 address 2001:db8:4::1/64
  ipv6 address FE80::6 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet2/0
  no shut
  ipv6 address 2001:db8:5::1/64
  ipv6 address FE80::7 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet3/0
  no shut
  ipv6 address 2001:db8:B::1/64
  ipv6 address FE80::8 link-local
  ipv6 rip LAB-RIPng enable
```
### R3
```
ipv6 unicast-routing
ipv6 router rip LAB-RIPng
interface GigabitEthernet0/0
  no shut
  ipv6 address 2001:db8:6::1/64
  ipv6 address FE80::9 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet1/0
  no shut
  ipv6 address 2001:db8:2::2/64
  ipv6 address FE80::10 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet2/0
  no shut
  ipv6 address 2001:db8:5::2/64
  ipv6 address FE80::11 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet3/0
  no shut
  ipv6 address 2001:db8:7::1/64
  ipv6 address FE80::12 link-local
  ipv6 rip LAB-RIPng enable
```
### R4
```
ipv6 unicast-routing
ipv6 router rip LAB-RIPng
interface GigabitEthernet0/0
  no shut
  ipv6 address 2001:db8:6::2/64
  ipv6 address FE80::13 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet1/0
  no shut
  ipv6 address 2001:db8:4::2/64
  ipv6 address FE80::14 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet2/0
  no shut
  ipv6 address 2001:db8:3::2/64
  ipv6 address FE80::15 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet3/0
  no shut
  ipv6 address 2001:db8:8::1/64
  ipv6 address FE80::16 link-local
  ipv6 rip LAB-RIPng enable
```
### R5
```
ipv6 unicast-routing
ipv6 router rip LAB-RIPng
interface GigabitEthernet0/0
  no shut
  ipv6 address 2001:db8:7::2/64
  ipv6 address FE80::17 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet1/0
  no shut
  ipv6 address 2001:db8:8::2/64
  ipv6 address FE80::18 link-local
  ipv6 rip LAB-RIPng enable
interface GigabitEthernet2/0
  no shut
  ipv6 address 2001:db8:C::1/64
  ipv6 address FE80::19 link-local
  ipv6 rip LAB-RIPng enable
```

## 2. Activer RIPng sur chaque routeur
### Au niveau de la conf globale
```
ipv6 unicast-routing
ipv6 router rip LAB-RIPng
```
## 3. Configurer RIPng sur les interfaces appropriées
### Au niveau de chaque interface
```
interface GigabitEthernet0/0
  ipv6 rip LAB-RIPng enable
```
## 4. Vérifier les tables de routage 
```
R1#show ipv6 route rip
IPv6 Routing Table - default - 17 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, R - RIP, H - NHRP, I1 - ISIS L1
       I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary, D - EIGRP
       EX - EIGRP external, ND - ND Default, NDp - ND Prefix, DCE - Destination
       NDr - Redirect, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
R   2001:DB8:4::/64 [120/2]
     via FE80::5, GigabitEthernet0/0
     via FE80::15, GigabitEthernet2/0
R   2001:DB8:6::/64 [120/2]
     via FE80::10, GigabitEthernet1/0
     via FE80::15, GigabitEthernet2/0
R   2001:DB8:7::/64 [120/2]
     via FE80::10, GigabitEthernet1/0
R   2001:DB8:8::/64 [120/2]
     via FE80::15, GigabitEthernet2/0
R   2001:DB8:B::/64 [120/2]
     via FE80::5, GigabitEthernet0/0
R   2001:DB8:C::/64 [120/3]
     via FE80::10, GigabitEthernet1/0
     via FE80::15, GigabitEthernet2/0
     
R3#show ipv6 route rip
IPv6 Routing Table - default - 16 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, R - RIP, H - NHRP, I1 - ISIS L1
       I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary, D - EIGRP
       EX - EIGRP external, ND - ND Default, NDp - ND Prefix, DCE - Destination
       NDr - Redirect, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
R   2001:DB8:1::/64 [120/2]
     via FE80::7, GigabitEthernet2/0
R   2001:DB8:3::/64 [120/2]
     via FE80::13, GigabitEthernet0/0
     via FE80::2, GigabitEthernet1/0
R   2001:DB8:4::/64 [120/2]
     via FE80::13, GigabitEthernet0/0
     via FE80::7, GigabitEthernet2/0
R   2001:DB8:8::/64 [120/2]
     via FE80::13, GigabitEthernet0/0
     via FE80::17, GigabitEthernet3/0
R   2001:DB8:A::/64 [120/2]
     via FE80::2, GigabitEthernet1/0
R   2001:DB8:B::/64 [120/2]
     via FE80::7, GigabitEthernet2/0
R   2001:DB8:C::/64 [120/2]
     via FE80::17, GigabitEthernet3/0

R4#show ipv6 route rip
IPv6 Routing Table - default - 16 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, R - RIP, H - NHRP, I1 - ISIS L1
       I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary, D - EIGRP
       EX - EIGRP external, ND - ND Default, NDp - ND Prefix, DCE - Destination
       NDr - Redirect, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
R   2001:DB8:1::/64 [120/2]
     via FE80::6, GigabitEthernet1/0
R   2001:DB8:2::/64 [120/2]
     via FE80::9, GigabitEthernet0/0
     via FE80::3, GigabitEthernet2/0
R   2001:DB8:5::/64 [120/2]
     via FE80::9, GigabitEthernet0/0
     via FE80::6, GigabitEthernet1/0
     via FE80::3, GigabitEthernet2/0
R   2001:DB8:7::/64 [120/2]
     via FE80::18, GigabitEthernet3/0
     via FE80::9, GigabitEthernet0/0
R   2001:DB8:A::/64 [120/2]
     via FE80::3, GigabitEthernet2/0
R   2001:DB8:B::/64 [120/2]
     via FE80::6, GigabitEthernet1/0
R   2001:DB8:C::/64 [120/2]
     via FE80::18, GigabitEthernet3/0
```
## 5. Tester la connectivité entre les LANs
Depuis `PC1`, on tente de ping `PC2` et `PC3`
```
PC1> ping 2001:db8:b::2

2001:db8:b::2 icmp6_seq=1 ttl=60 time=84.643 ms
2001:db8:b::2 icmp6_seq=2 ttl=60 time=30.136 ms
2001:db8:b::2 icmp6_seq=3 ttl=60 time=31.285 ms
2001:db8:b::2 icmp6_seq=4 ttl=60 time=41.555 ms
2001:db8:b::2 icmp6_seq=5 ttl=60 time=34.301 ms

PC1> ping 2001:db8:c::2

2001:db8:c::2 icmp6_seq=1 ttl=58 time=69.720 ms
2001:db8:c::2 icmp6_seq=2 ttl=58 time=56.797 ms
2001:db8:c::2 icmp6_seq=3 ttl=58 time=36.158 ms
2001:db8:c::2 icmp6_seq=4 ttl=58 time=53.518 ms
2001:db8:c::2 icmp6_seq=5 ttl=58 time=110.292 ms
```
## 6. Analyser le comportement en cas de panne d’un lien
On coupe le lien GigabitEthernet0/0 qui va de R1 vers R2
![image](https://github.com/user-attachments/assets/b43613bf-cdfa-4077-ac52-e9aa6aec640f)

Au bout de quelques secondes, le ping passe par une autre route et reprend
```
R1(config)#int GigabitEthernet 0/0
R1(config-if)#shut
*Oct 29 09:41:49.431: %LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to administratively down
*Oct 29 09:41:50.431: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to down


PC1> ping 2001:db8:b::2

2001:db8:b::2 icmp6_seq=1 ttl=59 time=42.988 ms 
2001:db8:b::2 icmp6_seq=2 ttl=59 time=41.732 ms
2001:db8:b::2 icmp6_seq=3 ttl=59 time=50.200 ms
2001:db8:b::2 icmp6_seq=4 timeout
2001:db8:b::2 icmp6_seq=5 timeout
```

# EIGRP
![image](https://github.com/user-attachments/assets/58d45fce-5882-41cb-a057-fb2f10235dfc)

## Plan d’adressage IPv4
* Liens entre routeurs : `10.0.0.0/30` à `10.0.0.28/30`
* LAN1 (`R1`) : `192.168.10.0/24`
* LAN2 (`R2`) : `192.168.20.0/24`
* LAN3 (`R2`) : `192.168.30.0/24`

## 1. Configurer les adresses IPv4 sur toutes les interfaces des routeurs
### R1
```
interface GigabitEthernet0/0
  no shut
  ip address 10.0.0.1 255.255.255.252
interface GigabitEthernet1/0
  no shut
  ip address 10.0.0.5 255.255.255.252
interface GigabitEthernet2/0
  no shut
  ip address 10.0.0.9 255.255.255.252
interface GigabitEthernet3/0
  no shut
  ip address 192.168.10.254 255.255.255.0
```
### R2
```
interface GigabitEthernet0/0
  no shut
  ip address 10.0.0.2 255.255.255.252
interface GigabitEthernet1/0
  no shut
  ip address 10.0.0.13 255.255.255.252
interface GigabitEthernet2/0
  no shut
  ip address 10.0.0.17 255.255.255.252
interface GigabitEthernet3/0
  no shut
  ip address 192.168.20.254 255.255.255.0
```
### R3
```
interface GigabitEthernet0/0
  no shut
  ip address 10.0.0.21 255.255.255.252
interface GigabitEthernet1/0
  no shut
  ip address 10.0.0.6 255.255.255.252
interface GigabitEthernet2/0
  no shut
  ip address 10.0.0.18 255.255.255.252
interface GigabitEthernet3/0
  no shut
  ip address 10.0.0.25 255.255.255.252
```
### R4
```
interface GigabitEthernet0/0
  no shut
  ip address 10.0.0.22 255.255.255.252
interface GigabitEthernet1/0
  no shut
  ip address 10.0.0.14 255.255.255.252
interface GigabitEthernet2/0
  no shut
  ip address 10.0.0.10 255.255.255.252
interface GigabitEthernet3/0
  no shut
  ip address 10.0.0.29 255.255.255.252
```
### R5
```
interface GigabitEthernet0/0
  no shut
  ip address 10.0.0.26 255.255.255.252
interface GigabitEthernet1/0
  no shut
  ip address 10.0.0.30 255.255.255.252
interface GigabitEthernet2/0
  no shut
  ip address 192.168.30.254 255.255.255.0
```

## 2. Configurer EIGRP avec le numéro de système autonome 100
On déclare EIGRP en s'assurant d'inclure tous les réseaux connectés à chaque routeur

> **Remarque :** Chaque `network` doit correspondre aux réseaux auxquels le routeur est connecté, en utilisant le masque inverse (wildcard mask). Le masque inverse pour un /30 est 0.0.0.3, ce qui permet de ne cibler que les adresses IP sur ces sous-réseaux spécifiques.

### R1
```
router eigrp 100
 network 192.168.10.0 0.0.0.255
 network 10.0.0.0 0.0.0.3
 network 10.0.0.4 0.0.0.3
 network 10.0.0.8 0.0.0.3
 no auto-summary
```
### R2
```
router eigrp 100
 network 192.168.20.0 0.0.0.255
 network 10.0.0.0 0.0.0.3
 network 10.0.0.12 0.0.0.3
 network 10.0.0.16 0.0.0.3
 no auto-summary
```
### R3
```
router eigrp 100
 network 10.0.0.4 0.0.0.3
 network 10.0.0.16 0.0.0.3
 network 10.0.0.20 0.0.0.3
 network 10.0.0.24 0.0.0.3
 no auto-summary
```
### R4
```
router eigrp 100
 network 10.0.0.8 0.0.0.3
 network 10.0.0.12 0.0.0.3
 network 10.0.0.20 0.0.0.3
 network 10.0.0.28 0.0.0.3
 no auto-summary
```
### R5
```
router eigrp 100
 network 192.168.30.0 0.0.0.255
 network 10.0.0.24 0.0.0.3
 network 10.0.0.28 0.0.0.3
 no auto-summary
```

## 3. Identifier les réseaux à annoncer dans EIGRP
Les réseaux à annoncer sont déjà identifiés dans les commandes ci-dessus (les network statements). Pour rappel, voici les réseaux annoncés par chaque routeur :
* **R1 :** `192.168.10.0/24`, `10.0.0.0/30`, `10.0.0.4/30`, `10.0.0.8/30`
* **R2 :** `192.168.20.0/24`, `10.0.0.0/30`, `10.0.0.12/30`, `10.0.0.16/30`
* **R3 :** `10.0.0.4/30`, `10.0.0.16/30`, `10.0.0.20/30`, `10.0.0.24/30`
* **R4 :** `10.0.0.8/30`, `10.0.0.12/30`, `10.0.0.20/30`, `10.0.0.28/30`
* **R5 :** `192.168.30.0/24`, `10.0.0.24/30`, `10.0.0.28/30`

## 4. Vérifier les relations de voisinage EIGRP
Après avoir configuré EIGRP, vérifie les relations de voisinage (ou adjacences) avec la commande suivante sur chaque routeur :

```
R3#show ip eigrp neighbors
EIGRP-IPv4 Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
3   10.0.0.5                Gi1/0                    11 00:00:48   17   102  0  9
2   10.0.0.17               Gi2/0                    14 00:00:54   13   100  0  9
1   10.0.0.26               Gi3/0                    10 00:01:01 1016  5000  0  13
0   10.0.0.22               Gi0/0                    14 00:01:01 1016  5000  0  17

R4#show ip eigrp neighbors
EIGRP-IPv4 Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
3   10.0.0.9                Gi2/0                    11 00:01:06   14   100  0  7
2   10.0.0.13               Gi1/0                    10 00:01:12   17   102  0  10
1   10.0.0.21               Gi0/0                    12 00:01:18   17   102  0  15
0   10.0.0.30               Gi3/0                    11 00:01:23  821  4926  0  12
```

Cette commande doit afficher les voisins EIGRP directement connectés. Chaque routeur devrait avoir plusieurs voisins en fonction de la topologie.

## 5. Analyser les tables de topologie et de routage

Pour voir la table de topologie EIGRP, on utilise la commande suivante : `show ip eigrp topology`

Cela permet de voir tous les réseaux connus par EIGRP, ainsi que leurs métriques.

Ensuite, pour voir la table de routage et confirmer que toutes les routes sont bien apprises, on utilise : `show ip route eigrp`

Les routes `D` indiquent celles apprises via **EIGRP** dans la table de routage. 

S'assurer que tous les réseaux des LAN (`192.168.10.0/24`, `192.168.20.0/24` et `192.168.30.0/24`) et les liens entre routeurs sont bien présents dans chaque table de routage.

```
R3#show ip eigrp topology
EIGRP-IPv4 Topology Table for AS(100)/ID(10.0.0.25)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status

P 10.0.0.12/30, 2 successors, FD is 3072
        via 10.0.0.17 (3072/2816), GigabitEthernet2/0
        via 10.0.0.22 (3072/2816), GigabitEthernet0/0
P 192.168.10.0/24, 1 successors, FD is 3072
        via 10.0.0.5 (3072/2816), GigabitEthernet1/0
P 10.0.0.20/30, 1 successors, FD is 2816
        via Connected, GigabitEthernet0/0
P 10.0.0.8/30, 2 successors, FD is 3072
        via 10.0.0.5 (3072/2816), GigabitEthernet1/0
        via 10.0.0.22 (3072/2816), GigabitEthernet0/0
P 192.168.30.0/24, 1 successors, FD is 3072
        via 10.0.0.26 (3072/2816), GigabitEthernet3/0
P 10.0.0.0/30, 2 successors, FD is 3072
        via 10.0.0.5 (3072/2816), GigabitEthernet1/0
        via 10.0.0.17 (3072/2816), GigabitEthernet2/0
P 10.0.0.4/30, 1 successors, FD is 2816
        via Connected, GigabitEthernet1/0
P 10.0.0.16/30, 1 successors, FD is 2816
        via Connected, GigabitEthernet2/0
P 10.0.0.28/30, 2 successors, FD is 3072
        via 10.0.0.22 (3072/2816), GigabitEthernet0/0
        via 10.0.0.26 (3072/2816), GigabitEthernet3/0
P 10.0.0.24/30, 1 successors, FD is 2816
        via Connected, GigabitEthernet3/0
P 192.168.20.0/24, 1 successors, FD is 3072
        via 10.0.0.17 (3072/2816), GigabitEthernet2/0
```

### R3
```
R3#show ip route eigrp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
D        10.0.0.0/30 [90/3072] via 10.0.0.17, 00:03:02, GigabitEthernet2/0
                     [90/3072] via 10.0.0.5, 00:03:02, GigabitEthernet1/0
D        10.0.0.8/30 [90/3072] via 10.0.0.22, 00:03:02, GigabitEthernet0/0
                     [90/3072] via 10.0.0.5, 00:03:02, GigabitEthernet1/0
D        10.0.0.12/30 [90/3072] via 10.0.0.22, 00:02:57, GigabitEthernet0/0
                      [90/3072] via 10.0.0.17, 00:02:57, GigabitEthernet2/0
D        10.0.0.28/30 [90/3072] via 10.0.0.26, 00:02:57, GigabitEthernet3/0
                      [90/3072] via 10.0.0.22, 00:02:57, GigabitEthernet0/0
D     192.168.10.0/24 [90/3072] via 10.0.0.5, 00:03:02, GigabitEthernet1/0
D     192.168.20.0/24 [90/3072] via 10.0.0.17, 00:02:57, GigabitEthernet2/0
D     192.168.30.0/24 [90/3072] via 10.0.0.26, 00:03:15, GigabitEthernet3/0
```

### R4
```
R4# show ip route eigrp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
D        10.0.0.0/30 [90/3072] via 10.0.0.13, 00:03:47, GigabitEthernet1/0
                     [90/3072] via 10.0.0.9, 00:03:47, GigabitEthernet2/0
D        10.0.0.4/30 [90/3072] via 10.0.0.21, 00:03:47, GigabitEthernet0/0
                     [90/3072] via 10.0.0.9, 00:03:47, GigabitEthernet2/0
D        10.0.0.16/30 [90/3072] via 10.0.0.21, 00:03:42, GigabitEthernet0/0
                      [90/3072] via 10.0.0.13, 00:03:42, GigabitEthernet1/0
D        10.0.0.24/30 [90/3072] via 10.0.0.30, 00:03:42, GigabitEthernet3/0
                      [90/3072] via 10.0.0.21, 00:03:42, GigabitEthernet0/0
D     192.168.10.0/24 [90/3072] via 10.0.0.9, 00:03:47, GigabitEthernet2/0
D     192.168.20.0/24 [90/3072] via 10.0.0.13, 00:03:42, GigabitEthernet1/0
D     192.168.30.0/24 [90/3072] via 10.0.0.30, 00:03:54, GigabitEthernet3/0
```

## 6. Tester la connectivité entre les réseaux
On utilise la commande ping pour vérifier la connectivité entre les réseaux :

* Depuis PC1, on essaie de faire un ping vers PC2 (`192.168.20.1`) et PC3 (`192.168.30.1`).
* Depuis PC2, on essaie de faire un ping vers PC1 (`192.168.10.1`) et PC3.
* Depuis PC3, on fais un ping vers PC1 et PC2.

Si la connectivité est en place, cela signifie qu’EIGRP fonctionne correctement et que les routeurs connaissent bien tous les réseaux de destination.


```
PC1> ping 192.168.20.10

84 bytes from 192.168.20.10 icmp_seq=1 ttl=62 time=51.612 ms
84 bytes from 192.168.20.10 icmp_seq=2 ttl=62 time=32.418 ms

PC1> ping 192.168.30.10

84 bytes from 192.168.30.10 icmp_seq=1 ttl=61 time=68.337 ms
84 bytes from 192.168.30.10 icmp_seq=2 ttl=61 time=35.257 ms

PC2> ping 192.168.10.10

84 bytes from 192.168.10.10 icmp_seq=1 ttl=61 time=54.841 ms
84 bytes from 192.168.10.10 icmp_seq=2 ttl=61 time=42.347 ms

PC2> ping 192.168.30.10

84 bytes from 192.168.30.10 icmp_seq=1 ttl=61 time=55.355 ms
84 bytes from 192.168.30.10 icmp_seq=2 ttl=61 time=45.848 ms

PC3> ping 192.168.10.10

84 bytes from 192.168.10.10 icmp_seq=1 ttl=61 time=72.832 ms
84 bytes from 192.168.10.10 icmp_seq=2 ttl=61 time=59.292 ms

PC3> ping 192.168.20.10

84 bytes from 192.168.20.10 icmp_seq=1 ttl=61 time=62.093 ms
84 bytes from 192.168.20.10 icmp_seq=2 ttl=61 time=53.788 ms
```

## 7. Observer le comportement d’EIGRP lors d’une panne de lien
![image](https://github.com/user-attachments/assets/f8fd173b-3e03-44c9-8369-3278a0129027)

Pour tester la tolérance de panne d’EIGRP, déconnecte l’un des liens entre deux routeurs (par exemple, entre R1 et R2). EIGRP devrait automatiquement recalculer les routes pour trouver un chemin alternatif.

Les chemins alternatifs devraient apparaître dans la table de routage, et après quelques secondes, la connectivité devrait être rétablie.

Le ping ne passe plus temporairement puis passe par une autre route

```
R1(config)#int GigabitEthernet 0/0
R1(config-if)#shut
*Oct 29 10:41:54.207: %DUAL-5-NBRCHANGE: EIGRP-IPv4 100: Neighbor 10.0.0.2 (GigabitEthernet0/0) is down: interface down
*Oct 29 10:41:56.163: %LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to administratively down
*Oct 29 10:41:57.163: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to down


PC1> ping 192.168.20.10

84 bytes from 192.168.20.10 icmp_seq=1 ttl=62 time=51.612 ms
84 bytes from 192.168.20.10 icmp_seq=2 ttl=62 time=31.218 ms
192.168.20.10 icmp_seq=3 timeout
192.168.20.10 icmp_seq=4 timeout
84 bytes from 192.168.20.10 icmp_seq=5 ttl=61 time=50.709 ms
```
