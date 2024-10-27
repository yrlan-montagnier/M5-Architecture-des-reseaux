# TP3

# Schéma de l’architecture
![image](https://github.com/user-attachments/assets/86b1c811-38e8-4189-8009-1c8aec477b34)

# Plan d’adressage respectant le principe de VLSM pour tous les VLANs.

Il prend en compte les informations suivantes : 
* Marketing : 8 personnes, utilisation de ressources cloud uniquement
* Développement : 17 personnes, 3 serveurs
* IT : 4 personnes, 5 serveurs
* RH : 3 personnes, 1 serveur
* Administration : 4 personnes, utilisation de ressources cloud uniquement

| VLAN           | Adresse réseau | Adresses IP disponibles       | Adresse Broadcast |
| -------------- | -------------- | ----------------------------- | ----------------- |
| Marketing (10)      | 10.10.10.0/28  | 14 (10.10.10.1 - 10.10.10.14)  | 10.10.10.15        |
| Developpement (20)  | 10.20.20.0/27  | 30 (10.20.20.1 - 10.20.20.30) | 10.20.20.31       |
| IT (30)             | 10.30.30.0/28  | 14 (10.30.30.1 - 10.30.30.14)   | 10.30.30.15        |
| RH (40)            | 10.40.40.0/29  | 6 (10.40.40.1 - 10.40.40.6)   | 10.40.40.7        |
| Administration (50) | 10.50.50.0/29  | 6 (10.50.50.1 - 10.50.50.6)   | 10.50.50.7        |

# Configurations complètes des équipements: 
* IOU1 : 
* IOU2 : 
* SW3 : 
* SW4 : 
* SW5 : 
* SW6 : 
* SW7 : 
* SW8 : 
* SW9 : 
* SW10 : 


# Interconnexions et Topologie
Les équipements sont interconnectés via des liens trunk et des agrégations (Port-Channels) pour transporter les VLANs à travers le réseau. 

# Mise en place HSRP
Le protocole HSRP (Hot Standby Router Protocol) est utilisé pour assurer la redondance des passerelles pour chaque VLAN.

## Configuration HSRP - IOU1
```
int vlan10
no shut
ip add 10.10.10.1 255.255.255.0
standby 10 ip 10.10.10.254
standby 10 preempt

int vlan20
no shut
ip add 10.20.20.1 255.255.255.0
standby 20 ip 10.20.20.254
standby 20 preempt

int vlan30
no shut
ip add 10.30.30.1 255.255.255.0
standby 30 ip 10.30.30.254
standby 30 preempt

int vlan40
no shut
ip add 10.40.40.1 255.255.255.0
standby 40 ip 10.40.40.254
standby 40 preempt

int vlan50
no shut
ip add 10.50.50.1 255.255.255.0
standby 50 ip 10.50.50.254
standby 50 preempt
```
## Configuration HSRP - IOU2
```
int vlan10
no shut
ip add 10.10.10.2 255.255.255.0
standby 10 ip 10.10.10.254
standby 10 preempt

int vlan20
no shut
ip add 10.20.20.2 255.255.255.0
standby 20 ip 10.20.20.254
standby 20 preempt

int vlan30
no shut
ip add 10.30.30.2 255.255.255.0
standby 30 ip 10.30.30.254
standby 30 preempt

int vlan40
no shut
ip add 10.40.40.2 255.255.255.0
standby 40 ip 10.40.40.254
standby 40 preempt

int vlan50
no shut
ip add 10.50.50.2 255.255.255.0
interface vlan50
standby 50 ip 10.50.50.254
standby 50 preempt
```

# Mise en place des ACL's
## 1. — Le VLAN IT (30) doit pouvoir communiquer avec tous les autres VLANs et tous les serveurs.
```
ip access-list extended VLAN30_IT_ALLOW_ANY
    permit ip 10.30.30.0 0.0.0.255 any
```

## 2. — Les autres VLANs ne doivent pas pouvoir communiquer directement entre eux.

## 3. — Le VLAN RH (40) doit pouvoir accéder à son serveur dédié dans le VLAN IT.
```
ip access-list extended VLAN40_RH_ALLOW-SRV-RH
    permit udp any host 224.0.0.2 eq 1985
    permit icmp host 10.40.40.1 host 10.40.40.2
    permit icmp host 10.40.40.2 host 10.40.40.1
    permit ip host 10.40.40.1 host 10.40.40.2
    permit ip host 10.40.40.2 host 10.40.40.1
    permit ip 10.40.40.0 0.0.0.255 10.30.30.100 0.0.0.0
    deny ip 10.40.40.0 0.0.0.255 10.10.10.0 0.0.0.255
    deny ip 10.40.40.0 0.0.0.255 10.20.20.0 0.0.0.255
    deny ip 10.40.40.0 0.0.0.255 10.30.30.0 0.0.0.255
    deny ip 10.40.40.0 0.0.0.255 10.50.50.0 0.0.0.255
```

## 4. — Le VLAN Développement (20) doit pouvoir accéder à ses 3 serveurs dédiés dans le VLAN IT.
```
ip access-list extended VLAN20_Developpement_ALLOW-SRV-DEV*
    permit udp any host 224.0.0.2 eq 1985
    permit icmp host 10.20.20.1 host 10.20.20.2
    permit icmp host 10.20.20.2 host 10.20.20.1
    permit ip host 10.20.20.1 host 10.20.20.2
    permit ip host 10.20.20.2 host 10.20.20.1
    permit ip 10.20.20.0 0.0.0.255 10.30.30.101 0.0.0.0
    permit ip 10.20.20.0 0.0.0.255 10.30.30.102 0.0.0.0
    permit ip 10.20.20.0 0.0.0.255 10.30.30.103 0.0.0.0
    deny ip 10.20.20.0 0.0.0.255 10.10.10.0 0.0.0.255
    deny ip 10.20.20.0 0.0.0.255 10.30.30.0 0.0.0.255
    deny ip 10.20.20.0 0.0.0.255 10.40.40.0 0.0.0.255
    deny ip 10.20.20.0 0.0.0.255 10.50.50.0 0.0.0.255
```

## 5. — Les VLANs Marketing et Administration doivent pouvoir accéder à Internet pour utiliser les ressources cloud.
### ACL pour le VLAN 10 (Marketing)
```
ip access-list extended VLAN10_Marketing_ALLOW-INTERNET
    permit udp any host 224.0.0.2 eq 1985
    permit icmp host 10.10.10.1 host 10.10.10.2
    permit icmp host 10.10.10.2 host 10.10.10.1
    permit ip host 10.10.10.1 host 10.10.10.2
    permit ip host 10.10.10.2 host 10.10.10.1
    permit ip 10.10.10.0 0.0.0.255 192.168.100.1 0.0.0.0
    deny ip 10.10.10.0 0.0.0.255 10.20.20.0 0.0.0.255
    deny ip 10.10.10.0 0.0.0.255 10.30.30.0 0.0.0.255
    deny ip 10.10.10.0 0.0.0.255 10.40.40.0 0.0.0.255
    deny ip 10.10.10.0 0.0.0.255 10.50.50.0 0.0.0.255
```

### ACL pour le VLAN 50 (Administration)
```
ip access-list extended VLAN50_Administration_ALLOW-INTERNET
    permit udp any host 224.0.0.2 eq 1985
    permit icmp host 10.50.50.1 host 10.50.50.2
    permit icmp host 10.50.50.2 host 10.50.50.1
    permit ip host 10.50.50.1 host 10.50.50.2
    permit ip host 10.50.50.2 host 10.50.50.1
    permit ip 10.50.50.0 0.0.0.255 192.168.100.1 0.0.0.0
    deny ip 10.50.50.0 0.0.0.255 10.10.10.0 0.0.0.255
    deny ip 10.50.50.0 0.0.0.255 10.20.20.0 0.0.0.255
    deny ip 10.50.50.0 0.0.0.255 10.30.30.0 0.0.0.255
    deny ip 10.50.50.0 0.0.0.255 10.40.40.0 0.0.0.255
```

## Applications des ACLs sur les interfaces VLAN sur IOU1/IOU2 (switchs L3)
```
interface vlan 10
    ip access-group VLAN10_Marketing_ALLOW-INTERNET in
interface vlan 20
    ip access-group VLAN20_Developpement_ALLOW-SRV-DEV* in
interface vlan 30
    ip access-group VLAN30_IT_ALLOW_ANY in
interface vlan 40
    ip access-group VLAN40_RH_ALLOW-SRV-RH in
interface vlan 50
    ip access-group VLAN50_Administration_ALLOW-INTERNET in
```

## Vérifications : 
```
SW1# sh access-lists
Extended IP access list VLAN10_Marketing_ALLOW-INTERNET
    10 permit udp any host 224.0.0.2 eq 1985 (2439 matches)
    20 permit icmp host 10.10.10.1 host 10.10.10.2
    30 permit icmp host 10.10.10.2 host 10.10.10.1
    40 permit ip host 10.10.10.1 host 10.10.10.2
    50 permit ip host 10.10.10.2 host 10.10.10.1
    60 permit ip 10.10.10.0 0.0.0.255 host 192.168.100.1
    70 deny ip 10.10.10.0 0.0.0.255 10.20.20.0 0.0.0.255
    80 deny ip 10.10.10.0 0.0.0.255 10.30.30.0 0.0.0.255
    90 deny ip 10.10.10.0 0.0.0.255 10.40.40.0 0.0.0.255
    100 deny ip 10.10.10.0 0.0.0.255 10.50.50.0 0.0.0.255
Extended IP access list VLAN20_Developpement_ALLOW-SRV-DEV*
    10 permit udp any host 224.0.0.2 eq 1985 (2443 matches)
    20 permit icmp host 10.20.20.1 host 10.20.20.2
    30 permit icmp host 10.20.20.2 host 10.20.20.1
    40 permit ip host 10.20.20.1 host 10.20.20.2
    50 permit ip host 10.20.20.2 host 10.20.20.1
    60 permit ip 10.20.20.0 0.0.0.255 host 10.30.30.101
    70 permit ip 10.20.20.0 0.0.0.255 host 10.30.30.102
    80 permit ip 10.20.20.0 0.0.0.255 host 10.30.30.103
    90 deny ip 10.20.20.0 0.0.0.255 10.10.10.0 0.0.0.255
    100 deny ip 10.20.20.0 0.0.0.255 10.30.30.0 0.0.0.255
    110 deny ip 10.20.20.0 0.0.0.255 10.40.40.0 0.0.0.255
    120 deny ip 10.20.20.0 0.0.0.255 10.50.50.0 0.0.0.255
Extended IP access list VLAN30_IT_ALLOW_ANY
    10 permit ip 10.30.30.0 0.0.0.255 any (30 matches)
Extended IP access list VLAN40_RH_ALLOW-SRV-RH
    10 permit udp any host 224.0.0.2 eq 1985 (2443 matches)
    20 permit icmp host 10.40.40.1 host 10.40.40.2
    30 permit icmp host 10.40.40.2 host 10.40.40.1
    40 permit ip host 10.40.40.1 host 10.40.40.2
    50 permit ip host 10.40.40.2 host 10.40.40.1
    60 permit ip 10.40.40.0 0.0.0.255 host 10.30.30.100
    70 deny ip 10.40.40.0 0.0.0.255 10.10.10.0 0.0.0.255
    80 deny ip 10.40.40.0 0.0.0.255 10.20.20.0 0.0.0.255
    90 deny ip 10.40.40.0 0.0.0.255 10.30.30.0 0.0.0.255
    100 deny ip 10.40.40.0 0.0.0.255 10.50.50.0 0.0.0.255
Extended IP access list VLAN50_Administration_ALLOW-INTERNET
    10 permit udp any host 224.0.0.2 eq 1985 (2440 matches)
    20 permit icmp host 10.50.50.1 host 10.50.50.2
    30 permit icmp host 10.50.50.2 host 10.50.50.1
    40 permit ip host 10.50.50.1 host 10.50.50.2
    50 permit ip host 10.50.50.2 host 10.50.50.1
    60 permit ip 10.50.50.0 0.0.0.255 host 192.168.100.1
    70 deny ip 10.50.50.0 0.0.0.255 10.10.10.0 0.0.0.255
    80 deny ip 10.50.50.0 0.0.0.255 10.20.20.0 0.0.0.255
    90 deny ip 10.50.50.0 0.0.0.255 10.30.30.0 0.0.0.255
    100 deny ip 10.50.50.0 0.0.0.255 10.40.40.0 0.0.0.255
```

# Multiple Spanning Tree
```
conf t
spanning-tree mode mst

spanning-tree mst configuration
    name LAB-YRLAN
    revision 1
    instance 1 vlan 10,20
    instance 2 vlan 40,50
    instance 3 vlan 30
exit
```
# Port-Security pour VLAN IT (30)
## 1. Activation de la sécurité par port (IOU8/IOU9)
Pour sécuriser le VLAN IT, nous allons activer la sécurité par port sur les interfaces connectées aux PC de ce VLAN. 

La commande suivante doit être appliquée sur chaque interface appropriée sur les deux switchs:
### IOU8
```
interface range Ethernet1/1-2
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

### IOU9
```
interface range Ethernet1/0 , Ethernet1/2
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

## 2. Configuration d'un maximum de 2 adresses MAC par port
Dans cette configuration, nous spécifions que chaque port peut apprendre un maximum de 2 adresses MAC. Cela est fait avec la ligne switchport port-security maximum 2.

## 3. Paramétrage de l'action en cas de branchement d'un troisième appareil
En cas de violation (c’est-à-dire si un troisième appareil tente de se connecter), le port sera désactivé grâce à la commande switchport port-security violation shutdown. Cela protège le réseau en empêchant des accès non autorisés.

# VLANs et trunks
On laisse uniquement passer les VLANs nécessaires

## Création et nommage des VLANs sur les switches

```
vlan 10
 name Marketing
vlan 20
 name Developpement
vlan 30
 name IT
vlan 40
 name RH
vlan 50
 name Administration
```

## Configuration des switchs L3 (IOU1/IOU2)
### IOU1/IOU2 (Switchs L3)
```
interface port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50

interface port-channel 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50

interface port-channel 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30

interface port-channel 4
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30,40,50
```

## Configuration des switchs de distribution L2 (IOU3 / IOU4 / IOU5 / IOU6) - Liaisons trunk spécifiques)
On configure les interfaces des switchs de distribution :
* Vers le deuxième switch de distribution (redondance) et le switch L3
* Vers les switchs d'accès

### IOU3
#### Vers le deuxième switch de distribution et switch L3
```
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
!
interface Port-channel2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
 channel-group 2 mode active
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
 channel-group 2 mode active
!

```

#### Vers les switchs d'accès
```
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 20,30
 switchport mode trunk
!
```
### IOU4
#### Vers le deuxième switch de distribution et switch L3
```
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
!
interface Port-channel2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
 channel-group 2 mode active
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
 channel-group 2 mode active
!
```

#### Vers les switchs d'accès
```
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 20,30
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10
 switchport mode trunk
!
```
### IOU5
#### Vers le deuxième switch de distribution et switch L3
```
!
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
!
interface Port-channel2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
 channel-group 2 mode active
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
 channel-group 2 mode active
!
```

#### Vers les switchs d'accès
```
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 50
 switchport mode trunk
!
```

### IOU6
#### Vers le deuxième switch de distribution et switch L3
```
!
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
!
interface Port-channel2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
 channel-group 2 mode active
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,50
 switchport mode trunk
 channel-group 2 mode active
!

```

#### Vers les switchs d'accès
```
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 50
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40
 switchport mode trunk
!
```

## Configuration des switchs d'accès (IOU7, IOU8, IOU9, IOU10) pour assigner les VLANs appropriés.
### IOU7 (Switch d'accès pour le VLAN Marketing) :
#### Interfaces Trunk
```
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10
 switchport mode trunk
!
```
#### Interfaces d'accès
```
!
interface Ethernet1/0
 switchport access vlan 10
 switchport mode access
!
```

### IOU8 (Switch d'accès pour le VLAN Développement et IT) :
#### Interfaces Trunk
```
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 20,30
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 20,30
 switchport mode trunk
!
```
#### Interfaces d'accès
```
!
interface Ethernet1/0
 switchport access vlan 20
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 30
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 30
 switchport mode access
!
```

### IOU9 (Switch d'accès pour le VLAN IT et RH) :
#### Interfaces Trunk
```
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40
 switchport mode trunk
!
```
#### Interfaces d'accès
```
!
interface Ethernet1/0
 switchport access vlan 30
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 40
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 30
 switchport mode access
!
```

### IOU10 (Switch d'accès pour le VLAN Administration) :
#### Interfaces Trunk
```
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 50
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 50
 switchport mode trunk
!
```
#### Interfaces d'accès
```
!
interface Ethernet1/0
 switchport access vlan 50
 switchport mode access
!
```
