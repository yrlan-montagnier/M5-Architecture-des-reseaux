# TP6 - OSPF

# Objectifs
* Comprendre le fonctionnement d’OSPF en environnement multizones
* Configurer OSPF avec différents types de zones
* Analyser le comportement du protocole et les tables de routage
* Maîtriser les concepts d’Area Border Router (ABR) et de BackboneArea
* Implémenter la redistribution de routes et les routes par défaut

# 1 Introduction à l’Architecture Multi-Zones
Dans ce TP, nous allons configurer une infrastructure réseau complexe utilisant OSPF avec trois zones distinctes :
* **Zone 0 (Backbone) :** Zone centrale reliant toutes les autres zones
* **Zone 1 :** Zone standard contenant des réseaux internes
* **Zone 2 :** Zone de type Stub pour simuler des réseaux d’utilisateurs finaux

# 2 Topologie du Réseau
La topologie comprend :
* 8 routeurs (R0 à R7)
* 3 zones OSPF distinctes
* 1 domaine EIGRP
* Des connexions LAN dans chaque zone

# 3 Plan d’Adressage

| **Zones - Réseau/Masque**     | **Routeur(s) / PC connectés** | **Interface à configurer** | **Adresse IP**                       |
| ----------------------------- | ----------------------------- | -------------------------- | ------------------------------------ |
| **Zone 0 (Backbone)**         |                               |                            |                                      |
| 10.10.0.0/30                  | R10, R1                       | R10 (G0/0) ↔ R1 (G2/0)     | R10: 10.10.0.1, R1: 10.10.0.2        |
| 10.10.0.4/30                  | R1, R3                        | R1 (G1/0) ↔ R3 (G0/0)      | R1: 10.10.0.5, R3: 10.10.0.6         |
| 10.10.0.8/30                  | R3, R2                        | R3 (G1/0) ↔ R2 (G1/0)      | R3: 10.10.0.9, R2: 10.10.0.10        |
| 10.10.0.12/30                 | R2, R10                       | R2 (G2/0) ↔ R10 (G1/0)     | R2: 10.10.0.13, R10: 10.10.0.14      |
| **LAN Zone 0**  - 10.0.0.0/24 | R3, PC2                       | R3 (G2/0) ↔ PC2 (e0)       | R3: 10.1.0.254, PC2: 10.1.0.10       |
| **Zone 1 (Standard)**         |                               |                            |                                      |
| 10.1.10.0/30                  | R1, R4                        | R1 (G0/0) ↔ R4 (G1/0)      | R1: 10.1.10.1, R4: 10.1.10.2         |
| 10.1.10.4/30                  | R4, R5                        | R4 (G0/0) ↔ R5 (G0/0)      | R1: 10.1.10.5, R5: 10.1.10.6         |
| **LAN Zone 1** - 10.1.0.0/24  | R5, PC1                       | R5 (G1/0), PC1 (e0)        | R4: 10.1.0.254, PC1: 10.1.0.10       |
| **Zone 2 (Stub)**             |                               |                            |                                      |
| 10.2.20.0/30                  | R2, R6                        | R2 (G0/0) ↔ R6 (G1/0)      | R2: 10.2.20.1, R6: 10.2.20.2         |
| 10.2.20.4/30                  | R6, R7                        | R6 (G0/0) ↔ R7 (G0/0)      | R6: 10.2.20.5, R7: 10.2.20.6         |
| **LAN Zone 2**  - 10.2.0.0/24 | R7, PC3                       | R7 (G1/0) ↔ PC3 (e0)       | R6: 10.2.0.254, PC3: 10.2.0.10       |
| **Domaine EIGRP**             |                               |                            |                                      |
| 10.100.0.0/30                 | R8, R9                        | R8 (G1/0) ↔ R9 (G1/0)      | R8: 10.100.0.1, R9: 10.100.0.2       |
| 10.100.0.4/30                 | R8, R10                       | R8 (G0/0) ↔ R10 (G2/0)     | R8: 10.100.0.5, R10: 10.100.0.6      |
| 10.100.0.8/30                 | R9, R10                       | R9 (G0/0) ↔ R10 (G3/0)     | R9: 10.100.0.9, R10: 10.100.0.10     |
| **LAN EIGRP 1**               | R8                            | R8 (G2/0) ↔ PC4 (e0)       | R8: 192.168.1.254, PC4: 192.168.1.10 |
| **LAN EIGRP 2**               | R9                            | R9 (G2/0) ↔ PC5 (e0)       | R9: 192.168.2.254  PC5: 192.168.2.10 |

## 3.1 Zone 0 (Backbone)
* LAN0 : 10.0.0.0/24
* Liens entre routeurs : 10.10.0.0/30 à 10.10.0.12/30
## 3.2 Zone 1
* LAN1 : 10.1.0.0/24
* Liens entre routeurs : 10.1.10.0/30 et 10.1.10.4/30
## 3.3 Zone 2 (Stub)
* LAN2 : 10.2.0.0/24
* Liens entre routeurs : 10.2.10.0/30 et 10.1.10.4/30
## 3.4 EIGRP
* LAN3 : 192.168.1.0/24
* LAN4 : 192.168.2.0/24
* Liens entre routeurs : 10.100.0.0/30 à 10.100.0.8/30
* AS EIGRP : 100


# Sujet
## Configuration de Base
### 1. Configurer les adresses IP sur toutes les interfaces
Exemple sur R10 : 
```
hostname R10
!
interface GigabitEthernet0/0
ip address 10.10.0.1 255.255.255.252
no shutdown
!
interface GigabitEthernet1/0
ip address 10.10.0.14 255.255.255.252
no shutdown
!
interface GigabitEthernet2/0
ip address 10.100.0.6 255.255.255.252
no shutdown
!
interface GigabitEthernet3/0
ip address 10.100.0.10 255.255.255.252
no shutdown
!
```
Pour R2 : 
```
hostname R2
!
interface GigabitEthernet0/0
ip address 10.2.20.1 255.255.255.252
no shutdown
!
interface GigabitEthernet1/0
ip address 10.10.0.10 255.255.255.252
no shutdown
!
interface GigabitEthernet2/0
ip address 10.10.0.13 255.255.255.252
no shutdown
!
```

### 2. Activer OSPF sur chaque routeur avec l’identifiant de processus 1

* `router ospf 1` : On active OSPF avec l'ID de processus 1
```
router ospf 1
```

### 3. Configurer les router-id de manière appropriée
* `0.0.0` : On indique que le routeur est dans la zone 0
* `1` : ID unique du routeur, adapté au nom du routeur. Pour R1 : `1`
```
router-id 0.0.0.1
```

### 4. Vérifier la connectivité de base entre les routeurs directement connectés
Depuis R10 (G1/0), on tente un ping vers R2 (G2/0) sur l'IP 10.10.0.13/30
```
R10# ping 10.10.0.13
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.0.13, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/22/36 ms
```

## Configuration des Zones OSPF
* `router ospf 1` : On active OSPF avec l'ID de processus 1
* `network x.x.x.x x.x.x.x` : On déclare les adresses réseaux des réseaux connectés à chaque routeur
* `area x` : On définit la zone (area) dans laquelle se trouve le réseau

### 1. Configurer la Zone 0 (Backbone)
#### R1
```
router ospf 1
router-id 0.0.0.1
network 10.10.0.0 0.0.0.3 area 0
network 10.10.0.4 0.0.0.3 area 0
```

#### R2
```
router ospf 1
router-id 0.0.0.2
network 10.10.0.8 0.0.0.3 area 0
network 10.10.0.12 0.0.0.3 area 0
```

#### R3
```
router ospf 1
router-id 0.0.0.3
network 10.10.0.4 0.0.0.3 area 0
network 10.10.0.8 0.0.0.3 area 0
network 10.0.0.0 0.0.0.255 area 0
```

#### R10
```
router ospf 1 
router-id 0.0.0.10
network 10.10.0.0 0.0.0.3 area 0
network 10.10.0.12 0.0.0.3 area 0
```

### 2. Configurer la Zone 1 comme zone standard
#### R4
```
router ospf 1
router-id 1.1.1.4
network 10.1.10.0 0.0.0.3 area 1
network 10.1.10.4 0.0.0.3 area 1
```
#### R5
```
router ospf 1
router-id 1.1.1.5
network 10.1.10.4 0.0.0.3 area 1
network 10.1.0.0 0.0.0.255 area 1
```

### 3. Configurer la Zone 2 comme zone stub
* `area 2 stub` : On définit la zone `2` comme étant la **zone "Stub"**
#### R6
```
router ospf 1
area 2 stub
router-id 2.2.2.6
network 10.2.20.0 0.0.0.3 area 2
network 10.2.20.4 0.0.0.3 area 2
```

#### R7
```
router ospf 1
area 2 stub
router-id 2.2.2.7
network 10.2.20.4 0.0.0.3 area 2
network 10.2.0.0 0.0.0.255 area 2
```

### 4. Vérifier les relations d’adjacence entre les routeurs
```
R1#show ip ospf neigh

Neighbor ID     Pri   State           Dead Time   Address         Interface
0.0.0.3           1   FULL/BDR        00:00:39    10.10.0.6       GigabitEthernet1/0
0.0.0.10          1   FULL/BDR        00:00:33    10.10.0.1       GigabitEthernet2/0
1.1.1.4           1   FULL/BDR        00:00:32    10.1.10.2       GigabitEthernet0/0
----------------------------------------------------------------------------------------
R2#show ip ospf neigh

Neighbor ID     Pri   State           Dead Time   Address         Interface
0.0.0.10          1   FULL/BDR        00:00:35    10.10.0.14      GigabitEthernet2/0
0.0.0.3           1   FULL/DR         00:00:31    10.10.0.9       GigabitEthernet1/0
10.10.10.6        1   FULL/DR         00:00:35    10.2.20.2       GigabitEthernet0/0
----------------------------------------------------------------------------------------
R4#show ip ospf neigh

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.5           1   FULL/DR         00:00:31    10.1.10.6       GigabitEthernet0/0
0.0.0.1           1   FULL/DR         00:00:33    10.1.10.1       GigabitEthernet1/0
----------------------------------------------------------------------------------------
R6# show ip ospf neigh

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.10.10.7        1   FULL/DR         00:00:32    10.2.20.6       GigabitEthernet0/0
0.0.0.2           1   FULL/BDR        00:00:36    10.2.20.1       GigabitEthernet1/0
----------------------------------------------------------------------------------------
R10#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
0.0.0.2           1   FULL/DR         00:00:39    10.10.0.13      GigabitEthernet1/0
0.0.0.1           1   FULL/DR         00:00:37    10.10.0.2       GigabitEthernet0/0
----------------------------------------------------------------------------------------
R10#show ip ospf database

            OSPF Router with ID (0.0.0.10) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.1         0.0.0.1         1147        0x80000005 0x00FEBE 2
0.0.0.2         0.0.0.2         1051        0x80000005 0x001B81 2
0.0.0.3         0.0.0.3         1526        0x80000005 0x00F39C 3
0.0.0.10        0.0.0.10        1065        0x80000003 0x002A72 2

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.0.2       0.0.0.1         1147        0x80000002 0x000D0D
10.10.0.5       0.0.0.1         1647        0x80000002 0x008C91
10.10.0.9       0.0.0.3         1526        0x80000002 0x005EB8
10.10.0.13      0.0.0.2         1051        0x80000002 0x00A26A

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.0.0        0.0.0.1         885         0x80000002 0x00111C
10.1.10.0       0.0.0.1         1647        0x80000002 0x007CAB
10.1.10.4       0.0.0.1         1647        0x80000002 0x005EC4
10.2.0.0        0.0.0.2         307         0x80000002 0x00FE2C
10.2.20.0       0.0.0.2         1553        0x80000002 0x00FB20
10.2.20.4       0.0.0.2         307         0x80000002 0x00DD39

                Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
10.100.0.0      0.0.0.10        1065        0x80000002 0x00DCCD 0
10.100.0.4      0.0.0.10        1065        0x80000002 0x00B4F1 0
10.100.0.8      0.0.0.10        1065        0x80000002 0x008C16 0
192.168.1.0     0.0.0.10        1065        0x80000002 0x006942 0
192.168.2.0     0.0.0.10        1065        0x80000002 0x005E4C 0

```

### 5. Analyser les LSA dans chaque zone
```
R2#show ip ospf database

            OSPF Router with ID (0.0.0.2) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.1         0.0.0.1         1636        0x80000005 0x00FEBE 2
0.0.0.2         0.0.0.2         1537        0x80000005 0x001B81 2
0.0.0.3         0.0.0.3         2013        0x80000005 0x00F39C 3
0.0.0.10        0.0.0.10        1554        0x80000003 0x002A72 2

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.0.2       0.0.0.1         1636        0x80000002 0x000D0D
10.10.0.5       0.0.0.1         139         0x80000003 0x008A92
10.10.0.9       0.0.0.3         2013        0x80000002 0x005EB8
10.10.0.13      0.0.0.2         1537        0x80000002 0x00A26A

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.0.0        0.0.0.1         1374        0x80000002 0x00111C
10.1.10.0       0.0.0.1         139         0x80000003 0x007AAC
10.1.10.4       0.0.0.1         139         0x80000003 0x005CC5
10.2.0.0        0.0.0.2         794         0x80000002 0x00FE2C
10.2.20.0       0.0.0.2         40          0x80000003 0x00F921
10.2.20.4       0.0.0.2         794         0x80000002 0x00DD39

                Router Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.2         0.0.0.2         794         0x80000006 0x009B4C 1
10.10.10.6      10.10.10.6      935         0x80000008 0x00C67F 2
10.10.10.7      10.10.10.7      1428        0x80000006 0x00C7B4 2

                Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
10.2.20.2       10.10.10.6      935         0x80000002 0x00527F
10.2.20.6       10.10.10.7      1427        0x80000004 0x00CBDB

                Summary Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         0.0.0.2         794         0x80000002 0x00A397
10.0.0.0        0.0.0.2         794         0x80000003 0x002906
10.1.0.0        0.0.0.2         794         0x80000003 0x003BEF
10.1.10.0       0.0.0.2         794         0x80000003 0x00A67F
10.1.10.4       0.0.0.2         794         0x80000003 0x008898
10.10.0.0       0.0.0.2         794         0x80000004 0x009C8A
10.10.0.4       0.0.0.2         794         0x80000003 0x0076AD
10.10.0.8       0.0.0.2         794         0x80000003 0x0044DC
10.10.0.12      0.0.0.2         794         0x80000003 0x001C01

                Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
10.100.0.0      0.0.0.10        1554        0x80000002 0x00DCCD 0
10.100.0.4      0.0.0.10        1554        0x80000002 0x00B4F1 0
10.100.0.8      0.0.0.10        1554        0x80000002 0x008C16 0
192.168.1.0     0.0.0.10        1554        0x80000002 0x006942 0
192.168.2.0     0.0.0.10        1554        0x80000002 0x005E4C 0
------------------------------------------------------------------------------
R6#show ip ospf database

            OSPF Router with ID (10.10.10.6) (Process ID 1)

                Router Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.2         0.0.0.2         773         0x80000006 0x009B4C 1
10.10.10.6      10.10.10.6      912         0x80000008 0x00C67F 2
10.10.10.7      10.10.10.7      1404        0x80000006 0x00C7B4 2

                Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
10.2.20.2       10.10.10.6      912         0x80000002 0x00527F
10.2.20.6       10.10.10.7      1404        0x80000004 0x00CBDB

                Summary Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         0.0.0.2         773         0x80000002 0x00A397
10.0.0.0        0.0.0.2         773         0x80000003 0x002906
10.1.0.0        0.0.0.2         773         0x80000003 0x003BEF
10.1.10.0       0.0.0.2         773         0x80000003 0x00A67F
10.1.10.4       0.0.0.2         773         0x80000003 0x008898
10.10.0.0       0.0.0.2         773         0x80000004 0x009C8A
10.10.0.4       0.0.0.2         773         0x80000003 0x0076AD
10.10.0.8       0.0.0.2         773         0x80000003 0x0044DC
10.10.0.12      0.0.0.2         773         0x80000003 0x001C01
------------------------------------------------------------------------------
R10#show ip ospf database

            OSPF Router with ID (0.0.0.10) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.1         0.0.0.1         1147        0x80000005 0x00FEBE 2
0.0.0.2         0.0.0.2         1051        0x80000005 0x001B81 2
0.0.0.3         0.0.0.3         1526        0x80000005 0x00F39C 3
0.0.0.10        0.0.0.10        1065        0x80000003 0x002A72 2

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.0.2       0.0.0.1         1147        0x80000002 0x000D0D
10.10.0.5       0.0.0.1         1647        0x80000002 0x008C91
10.10.0.9       0.0.0.3         1526        0x80000002 0x005EB8
10.10.0.13      0.0.0.2         1051        0x80000002 0x00A26A

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.0.0        0.0.0.1         885         0x80000002 0x00111C
10.1.10.0       0.0.0.1         1647        0x80000002 0x007CAB
10.1.10.4       0.0.0.1         1647        0x80000002 0x005EC4
10.2.0.0        0.0.0.2         307         0x80000002 0x00FE2C
10.2.20.0       0.0.0.2         1553        0x80000002 0x00FB20
10.2.20.4       0.0.0.2         307         0x80000002 0x00DD39

                Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
10.100.0.0      0.0.0.10        1065        0x80000002 0x00DCCD 0
10.100.0.4      0.0.0.10        1065        0x80000002 0x00B4F1 0
10.100.0.8      0.0.0.10        1065        0x80000002 0x008C16 0
192.168.1.0     0.0.0.10        1065        0x80000002 0x006942 0
192.168.2.0     0.0.0.10        1065        0x80000002 0x005E4C 0

```

## Configuration EIGRP et Redistribution
### 1. Configurer EIGRP AS 100 sur R8, R9 et R10
#### R8 : 
```
router eigrp 100
network 10.100.0.0 0.0.0.3
network 10.100.0.4 0.0.0.3
network 192.168.1.0 0.0.0.255
```
#### R9 : 
```
router eigrp 100
network 10.100.0.0 0.0.0.3
network 10.100.0.8 0.0.0.3
network 192.168.2.0 0.0.0.255
```

#### R10 : 
```
router eigrp 100
network 10.100.0.4 0.0.0.3
network 10.100.0.8 0.0.0.3
redistribute ospf 1 metric 1500 100 255 1 1500
```

### 2. Établir la connectivité EIGRP entre R8, R9 et R10
* `network` : On déclare tous les réseaux connectés au rout

Exemple pour R10
```
network 10.100.0.4 0.0.0.3
network 10.100.0.8 0.0.0.3
```

### 3. Configurer la redistribution bi-directionnelle et configurer les métriques de redistribution appropriées
Sur R10, on rentre dans la configuration `EIGRP` et `OSPF` et on passe la commande `redistribute` appropriée.
#### Redistribuer EIGRP dans OSPF sur R10
```
router eigrp 100
redistribute ospf 1 metric 1500 100 255 1 1500
```
#### Redistribuer OSPF dans EIGRP sur R10
```
router ospf 1 
redistribute eigrp 100 subnets metric-type 1
```

### 4. Vérifier la propagation des routes entre les deux protocoles
Sur R10, on affiche les routes avec `show ip route` :
* D : Routes découvertes via EIGRP
* O : Routes découvertes via OSPF
```
R10#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 18 subnets, 3 masks
O        10.0.0.0/24 [110/3] via 10.10.0.13, 01:04:32, GigabitEthernet1/0
                     [110/3] via 10.10.0.2, 01:04:32, GigabitEthernet0/0
O IA     10.1.0.0/24 [110/4] via 10.10.0.2, 00:59:21, GigabitEthernet0/0
O IA     10.1.10.0/30 [110/2] via 10.10.0.2, 01:04:32, GigabitEthernet0/0
O IA     10.1.10.4/30 [110/3] via 10.10.0.2, 01:04:32, GigabitEthernet0/0
O IA     10.2.0.0/24 [110/4] via 10.10.0.13, 00:50:59, GigabitEthernet1/0
O IA     10.2.20.0/30 [110/2] via 10.10.0.13, 01:04:32, GigabitEthernet1/0
O IA     10.2.20.4/30 [110/3] via 10.10.0.13, 00:50:59, GigabitEthernet1/0
C        10.10.0.0/30 is directly connected, GigabitEthernet0/0
L        10.10.0.1/32 is directly connected, GigabitEthernet0/0
O        10.10.0.4/30 [110/2] via 10.10.0.2, 01:04:32, GigabitEthernet0/0
O        10.10.0.8/30 [110/2] via 10.10.0.13, 01:04:32, GigabitEthernet1/0
C        10.10.0.12/30 is directly connected, GigabitEthernet1/0
L        10.10.0.14/32 is directly connected, GigabitEthernet1/0
D        10.100.0.0/30 [90/3072] via 10.100.0.9, 01:06:00, GigabitEthernet3/0
                       [90/3072] via 10.100.0.5, 01:06:00, GigabitEthernet2/0
C        10.100.0.4/30 is directly connected, GigabitEthernet2/0
L        10.100.0.6/32 is directly connected, GigabitEthernet2/0
C        10.100.0.8/30 is directly connected, GigabitEthernet3/0
L        10.100.0.10/32 is directly connected, GigabitEthernet3/0
D     192.168.1.0/24 [90/3072] via 10.100.0.5, 01:06:00, GigabitEthernet2/0
D     192.168.2.0/24 [90/3072] via 10.100.0.9, 01:06:00, GigabitEthernet3/0

```

## Tests et Analyse
### 1. Effectuer des tests de connectivité entre tous les réseaux
Depuis PC2 dans la zone 0, on tente de ping les PC des zones 1 et 2 ainsi que dans le LAN, derrière les routeurs configurés en EIGRP
```
PC2> ping 10.1.0.10

84 bytes from 10.1.0.10 icmp_seq=1 ttl=60 time=164.810 ms
84 bytes from 10.1.0.10 icmp_seq=2 ttl=60 time=125.220 ms

PC2> ping 10.2.0.10

84 bytes from 10.2.0.10 icmp_seq=1 ttl=60 time=148.373 ms
84 bytes from 10.2.0.10 icmp_seq=2 ttl=60 time=148.932 ms

PC2> ping 192.168.1.10

84 bytes from 192.168.1.10 icmp_seq=1 ttl=60 time=148.767 ms
84 bytes from 192.168.1.10 icmp_seq=2 ttl=60 time=86.188 ms

PC2> ping 192.168.2.10

84 bytes from 192.168.2.10 icmp_seq=1 ttl=60 time=157.616 ms
84 bytes from 192.168.2.10 icmp_seq=2 ttl=60 time=112.664 ms
```

### 2. Analyser les chemins choisis par OSPF et EIGRP
```
R1#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 17 subnets, 3 masks
O        10.0.0.0/24 [110/2] via 10.10.0.6, 01:29:32, GigabitEthernet1/0
O        10.1.0.0/24 [110/3] via 10.1.10.2, 01:16:53, GigabitEthernet0/0
O        10.1.10.4/30 [110/2] via 10.1.10.2, 01:17:03, GigabitEthernet0/0
O IA     10.2.0.0/24 [110/5] via 10.10.0.6, 01:08:30, GigabitEthernet1/0
                     [110/5] via 10.10.0.1, 01:08:30, GigabitEthernet2/0
O IA     10.2.20.0/30 [110/3] via 10.10.0.6, 01:28:50, GigabitEthernet1/0
                      [110/3] via 10.10.0.1, 01:22:02, GigabitEthernet2/0
O IA     10.2.20.4/30 [110/4] via 10.10.0.6, 01:08:30, GigabitEthernet1/0
                      [110/4] via 10.10.0.1, 01:08:30, GigabitEthernet2/0
O        10.10.0.8/30 [110/2] via 10.10.0.6, 01:29:32, GigabitEthernet1/0
O        10.10.0.12/30 [110/2] via 10.10.0.1, 01:22:02, GigabitEthernet2/0
O E1     10.100.0.0/30 [110/21] via 10.10.0.1, 01:22:02, GigabitEthernet2/0
O E1     10.100.0.4/30 [110/21] via 10.10.0.1, 01:22:02, GigabitEthernet2/0
O E1     10.100.0.8/30 [110/21] via 10.10.0.1, 01:22:02, GigabitEthernet2/0
O E1  192.168.1.0/24 [110/21] via 10.10.0.1, 01:22:02, GigabitEthernet2/0
O E1  192.168.2.0/24 [110/21] via 10.10.0.1, 01:22:02, GigabitEthernet2/0

R4# sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 16 subnets, 3 masks
O IA     10.0.0.0/24 [110/3] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O        10.1.0.0/24 [110/2] via 10.1.10.6, 01:18:08, GigabitEthernet0/0
O IA     10.2.0.0/24 [110/6] via 10.1.10.1, 01:09:35, GigabitEthernet1/0
O IA     10.2.20.0/30 [110/4] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O IA     10.2.20.4/30 [110/5] via 10.1.10.1, 01:09:35, GigabitEthernet1/0
O IA     10.10.0.0/30 [110/2] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O IA     10.10.0.4/30 [110/2] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O IA     10.10.0.8/30 [110/3] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O IA     10.10.0.12/30 [110/3] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O E1     10.100.0.0/30 [110/22] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O E1     10.100.0.4/30 [110/22] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O E1     10.100.0.8/30 [110/22] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O E1  192.168.1.0/24 [110/22] via 10.1.10.1, 01:19:05, GigabitEthernet1/0
O E1  192.168.2.0/24 [110/22] via 10.1.10.1, 01:19:05, GigabitEthernet1/0

R10# sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 18 subnets, 3 masks
O        10.0.0.0/24 [110/3] via 10.10.0.13, 01:23:35, GigabitEthernet1/0
                     [110/3] via 10.10.0.2, 01:23:35, GigabitEthernet0/0
O IA     10.1.0.0/24 [110/4] via 10.10.0.2, 01:18:24, GigabitEthernet0/0
O IA     10.1.10.0/30 [110/2] via 10.10.0.2, 01:23:35, GigabitEthernet0/0
O IA     10.1.10.4/30 [110/3] via 10.10.0.2, 01:23:35, GigabitEthernet0/0
O IA     10.2.0.0/24 [110/4] via 10.10.0.13, 01:10:02, GigabitEthernet1/0
O IA     10.2.20.0/30 [110/2] via 10.10.0.13, 01:23:35, GigabitEthernet1/0
O IA     10.2.20.4/30 [110/3] via 10.10.0.13, 01:10:02, GigabitEthernet1/0
C        10.10.0.0/30 is directly connected, GigabitEthernet0/0
L        10.10.0.1/32 is directly connected, GigabitEthernet0/0
O        10.10.0.4/30 [110/2] via 10.10.0.2, 01:23:35, GigabitEthernet0/0
O        10.10.0.8/30 [110/2] via 10.10.0.13, 01:23:35, GigabitEthernet1/0
C        10.10.0.12/30 is directly connected, GigabitEthernet1/0
L        10.10.0.14/32 is directly connected, GigabitEthernet1/0
D        10.100.0.0/30 [90/3072] via 10.100.0.9, 01:25:03, GigabitEthernet3/0
                       [90/3072] via 10.100.0.5, 01:25:03, GigabitEthernet2/0
C        10.100.0.4/30 is directly connected, GigabitEthernet2/0
L        10.100.0.6/32 is directly connected, GigabitEthernet2/0
C        10.100.0.8/30 is directly connected, GigabitEthernet3/0
L        10.100.0.10/32 is directly connected, GigabitEthernet3/0
D     192.168.1.0/24 [90/3072] via 10.100.0.5, 01:25:03, GigabitEthernet2/0
D     192.168.2.0/24 [90/3072] via 10.100.0.9, 01:25:03, GigabitEthernet3/0

```

### 3. Simuler des pannes de liens et observer la convergence
On tente des ping continu et on éteint des interfaces de routeur pour tester la redondance.

On constate que les pings reviennent très rapidement en passant par une autre route.

### 4. Comparer le contenu des bases de données OSPF dans chaque zone
On voit qu'en fonction du routeur sur lequel on lance la commande, le retour est différent.

Les routeurs de bordure de zone contiennent les informations des deux zones qu'ils relient.

Par exemple, sur **R4**, on aura `Router Link States (Area 1)`. Sur **R6**, on aura `Router Link States (Area 2)`

#### Zone 0 - Backbone
##### Sur R2 - Routeur de bordure entre zone 0 (Backbone) et zone 2 (Stub)
```
R2#show ip ospf database

            OSPF Router with ID (0.0.0.2) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.1         0.0.0.1         1636        0x80000005 0x00FEBE 2
0.0.0.2         0.0.0.2         1537        0x80000005 0x001B81 2
0.0.0.3         0.0.0.3         2013        0x80000005 0x00F39C 3
0.0.0.10        0.0.0.10        1554        0x80000003 0x002A72 2

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.0.2       0.0.0.1         1636        0x80000002 0x000D0D
10.10.0.5       0.0.0.1         139         0x80000003 0x008A92
10.10.0.9       0.0.0.3         2013        0x80000002 0x005EB8
10.10.0.13      0.0.0.2         1537        0x80000002 0x00A26A

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.0.0        0.0.0.1         1374        0x80000002 0x00111C
10.1.10.0       0.0.0.1         139         0x80000003 0x007AAC
10.1.10.4       0.0.0.1         139         0x80000003 0x005CC5
10.2.0.0        0.0.0.2         794         0x80000002 0x00FE2C
10.2.20.0       0.0.0.2         40          0x80000003 0x00F921
10.2.20.4       0.0.0.2         794         0x80000002 0x00DD39

                Router Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.2         0.0.0.2         794         0x80000006 0x009B4C 1
10.10.10.6      10.10.10.6      935         0x80000008 0x00C67F 2
10.10.10.7      10.10.10.7      1428        0x80000006 0x00C7B4 2

                Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
10.2.20.2       10.10.10.6      935         0x80000002 0x00527F
10.2.20.6       10.10.10.7      1427        0x80000004 0x00CBDB

                Summary Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         0.0.0.2         794         0x80000002 0x00A397
10.0.0.0        0.0.0.2         794         0x80000003 0x002906
10.1.0.0        0.0.0.2         794         0x80000003 0x003BEF
10.1.10.0       0.0.0.2         794         0x80000003 0x00A67F
10.1.10.4       0.0.0.2         794         0x80000003 0x008898
10.10.0.0       0.0.0.2         794         0x80000004 0x009C8A
10.10.0.4       0.0.0.2         794         0x80000003 0x0076AD
10.10.0.8       0.0.0.2         794         0x80000003 0x0044DC
10.10.0.12      0.0.0.2         794         0x80000003 0x001C01

                Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
10.100.0.0      0.0.0.10        1554        0x80000002 0x00DCCD 0
10.100.0.4      0.0.0.10        1554        0x80000002 0x00B4F1 0
10.100.0.8      0.0.0.10        1554        0x80000002 0x008C16 0
192.168.1.0     0.0.0.10        1554        0x80000002 0x006942 0
192.168.2.0     0.0.0.10        1554        0x80000002 0x005E4C 0

```

##### Sur R3 : 
```
R3#show ip ospf database

            OSPF Router with ID (0.0.0.3) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.1         0.0.0.1         1129        0x80000005 0x00FEBE 2
0.0.0.2         0.0.0.2         1033        0x80000005 0x001B81 2
0.0.0.3         0.0.0.3         1506        0x80000005 0x00F39C 3
0.0.0.10        0.0.0.10        1049        0x80000003 0x002A72 2

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.0.2       0.0.0.1         1129        0x80000002 0x000D0D
10.10.0.5       0.0.0.1         1628        0x80000002 0x008C91
10.10.0.9       0.0.0.3         1506        0x80000002 0x005EB8
10.10.0.13      0.0.0.2         1033        0x80000002 0x00A26A

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.0.0        0.0.0.1         867         0x80000002 0x00111C
10.1.10.0       0.0.0.1         1629        0x80000002 0x007CAB
10.1.10.4       0.0.0.1         1628        0x80000002 0x005EC4
10.2.0.0        0.0.0.2         289         0x80000002 0x00FE2C
10.2.20.0       0.0.0.2         1535        0x80000002 0x00FB20
10.2.20.4       0.0.0.2         289         0x80000002 0x00DD39

                Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
10.100.0.0      0.0.0.10        1049        0x80000002 0x00DCCD 0
10.100.0.4      0.0.0.10        1049        0x80000002 0x00B4F1 0
10.100.0.8      0.0.0.10        1049        0x80000002 0x008C16 0
192.168.1.0     0.0.0.10        1049        0x80000002 0x006942 0
192.168.2.0     0.0.0.10        1049        0x80000002 0x005E4C 0
```

#### Zone 1 - Standard
##### Sur R4 : 
```
R4#show ip ospf database

            OSPF Router with ID (1.1.1.4) (Process ID 1)

                Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.1         0.0.0.1         89          0x80000007 0x00906D 1
1.1.1.4         1.1.1.4         47          0x80000007 0x00E7C4 2
1.1.1.5         1.1.1.5         1974        0x80000004 0x00E9E3 2

                Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.10.1       0.0.0.1         89          0x80000005 0x00DE3B
10.1.10.6       1.1.1.5         1973        0x80000002 0x00C247

                Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.0.0.0        0.0.0.1         836         0x80000003 0x00111D
10.2.0.0        0.0.0.1         1560        0x80000002 0x001911
10.2.20.0       0.0.0.1         836         0x80000003 0x001406
10.2.20.4       0.0.0.1         1560        0x80000002 0x00F71E
10.10.0.0       0.0.0.1         836         0x80000003 0x007CAB
10.10.0.4       0.0.0.1         836         0x80000003 0x0054CF
10.10.0.8       0.0.0.1         836         0x80000003 0x0036E8
10.10.0.12      0.0.0.1         343         0x80000004 0x000C0E

                Summary ASB Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.10        0.0.0.1         342         0x80000003 0x001717

                Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
10.100.0.0      0.0.0.10        248         0x80000003 0x00DACE 0
10.100.0.4      0.0.0.10        248         0x80000003 0x00B2F2 0
10.100.0.8      0.0.0.10        248         0x80000003 0x008A17 0
192.168.1.0     0.0.0.10        248         0x80000003 0x006743 0
192.168.2.0     0.0.0.10        248         0x80000003 0x005C4D 0

```

#### Zone 2 - Stub
##### Sur R6 : 
```
R6#show ip ospf database

            OSPF Router with ID (10.10.10.6) (Process ID 1)

                Router Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum Link count
0.0.0.2         0.0.0.2         1510        0x80000006 0x009B4C 1
10.10.10.6      10.10.10.6      1649        0x80000008 0x00C67F 2
10.10.10.7      10.10.10.7      94          0x80000007 0x00C5B5 2

                Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
10.2.20.2       10.10.10.6      1649        0x80000002 0x00527F
10.2.20.6       10.10.10.7      94          0x80000005 0x00C9DC

                Summary Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         0.0.0.2         1510        0x80000002 0x00A397
10.0.0.0        0.0.0.2         1510        0x80000003 0x002906
10.1.0.0        0.0.0.2         1510        0x80000003 0x003BEF
10.1.10.0       0.0.0.2         1510        0x80000003 0x00A67F
10.1.10.4       0.0.0.2         1510        0x80000003 0x008898
10.10.0.0       0.0.0.2         1510        0x80000004 0x009C8A
10.10.0.4       0.0.0.2         1510        0x80000003 0x0076AD
10.10.0.8       0.0.0.2         1510        0x80000003 0x0044DC
10.10.0.12      0.0.0.2         1510        0x80000003 0x001C01

```
