# TP3
# Adaptation de l'architecture
## Schéma de l’infra
![image](https://github.com/user-attachments/assets/86b1c811-38e8-4189-8009-1c8aec477b34)

## Plan d’adressage respectant le principe de VLSM pour tous les VLANs.

Il prend en compte les informations suivantes : 
* Marketing : 8 personnes, utilisation de ressources cloud uniquement
* Développement : 17 personnes, 3 serveurs
* IT : 4 personnes, 5 serveurs
* RH : 3 personnes, 1 serveur
* Administration : 4 personnes, utilisation de ressources cloud uniquement

J'ai utilisé des masques de sous réseaux en /24, mais voici le plan qui aurait du être utilisé pour respecter le principe de VLSM.

## Plan d'adressage que j'aurais du utiliser
|        VLAN         | Adresse réseau |    Adresses IP disponibles    |   Gateway   | Adresse Broadcast |
|:-------------------:|:--------------:|:-----------------------------:|:-----------:|:-----------------:|
|   Marketing (10)    | 10.10.10.0/28  | 13(10.10.10.1 - 10.10.10.13)  | 10.10.10.14 |    10.10.10.15    |
| Developpement (20)  | 10.20.20.0/27  | 29 (10.20.20.1 - 10.20.20.29) | 10.20.20.30 |    10.20.20.31    |
|       IT (30)       | 10.30.30.0/28  | 13 (10.30.30.1 - 10.30.30.13) | 10.30.30.14 |    10.30.30.15    |
|       RH (40)       | 10.40.40.0/29  |  5 (10.40.40.1 - 10.40.40.5)  | 10.40.40.6  |    10.40.40.7     |
| Administration (50) | 10.50.50.0/29  |  5 (10.50.50.1 - 10.50.50.5)  | 10.50.50.6  |    10.50.50.7     |

## Tableau des machines que j'aurais du utiliser
|        VLAN         |       Hôte        |   Adresse IP    |
|:-------------------:|:-----------------:|:---------------:|
|   Marketing (10)    |     PC-MARKET     |  10.10.10.1/28  |
| Developpement (20)  |      PC-DEV       |  10.20.20.1/27  |
|       IT (30)       |      PC-ADM1      |  10.30.30.1/28  |
|       IT (30)       |      PC-ADM2      |  10.30.30.2/28  |
|       IT (30)       |      SRV-DEV      | 10.30.30.100/28 |
|       IT (30)       |      SRV-RH       | 10.30.30.101/28 |
|       RH (40)       |       PC-RH       |  10.40.40.0/29  |
| Administration (50) | PC-ADMINISTRATION |  10.50.50.0/29  |

## Tableau d'adressage que j'ai utilisé
|        VLAN         |       Hôte        |   Adresse IP    |
|:-------------------:|:-----------------:|:---------------:|
|   Marketing (10)    |     PC-MARKET     | 10.10.10.10/24  |
| Developpement (20)  |      PC-DEV       | 10.20.20.10/24  |
|       IT (30)       |      PC-ADM1      | 10.30.30.10/24  |
|       IT (30)       |      PC-ADM2      | 10.30.30.20/24  |
|       IT (30)       |      SRV-DEV      | 10.30.30.100/24 |
|       IT (30)       |      SRV-RH       | 10.30.30.101/24 |
|       RH (40)       |       PC-RH       | 10.40.40.10/24  |
| Administration (50) | PC-ADMINISTRATION | 10.50.50.10/24  |

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

Pour chaque **Interface VLAN**, on **configure une adresse IP statique**, puis, on lui donne une IP "standby" identifiée par un groupe (10) avec : `standby 10 ip 10.10.10.254`

Cette adresse IP Virtuelle sera partagée entre les deux switchs

|        VLAN         |  Switch L3   | Adresse IP | Adresse IP virtuelle -  HSRP |
|:-------------------:|:------------:|:----------:|:----------------------------:|
|   Marketing (10)    | SW1 (Passif) | 10.10.10.1 |         10.10.10.254         |
|   Marketing (10)    | SW2 (Actif)  | 10.10.10.2 |         10.10.10.254         |
| Developpement (20)  | SW1 (Passif) | 10.20.20.1 |         10.20.20.254         |
| Developpement (20)  | SW2 (Actif)  | 10.20.20.2 |         10.20.20.254         |
|       IT (30)       | SW1 (Passif) | 10.30.30.1 |         10.30.30.254         |
|       IT (30)       | SW2 (Actif)  | 10.30.30.2 |         10.30.30.254         |
|       RH (40)       | SW1 (Passif) | 10.40.40.1 |         10.40.40.254         |
|       RH (40)       | SW2 (Actif)  | 10.40.40.2 |         10.40.40.254         |
| Administration (50) | SW1 (Passif) | 10.50.50.1 |         10.50.50.254         |
| Administration (50) | SW2 (Actif)  | 10.50.50.2 |         10.50.50.254         |

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

## Vérifications
### SW1
On voit que le switch 1 est passif : `State is Standby`

On retrouve l'adresse IP virtuelle : `Virtual IP address is 10.10.10.254`

Le switch passif (SW1) : `Standby router is local`

Le switch actif (SW2) : `Active router is 10.10.10.2, priority 100 (expires in 9.648 sec)`

La priorité : `Priority 100 (default 100)`
```
SW1# sh standby
Vlan10 - Group 10
  State is Standby
    1 state change, last state change 00:02:24
  Virtual IP address is 10.10.10.254
  Active virtual MAC address is 0000.0c07.ac0a (MAC Not In Use)
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.008 secs
  Preemption enabled
  Active router is 10.10.10.2, priority 100 (expires in 9.648 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Vl10-10" (default)
Vlan20 - Group 20
  State is Standby
    1 state change, last state change 00:02:26
  Virtual IP address is 10.20.20.254
  Active virtual MAC address is 0000.0c07.ac14 (MAC Not In Use)
    Local virtual MAC address is 0000.0c07.ac14 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.656 secs
  Preemption enabled
  Active router is 10.20.20.2, priority 100 (expires in 10.384 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Vl20-20" (default)
Vlan30 - Group 30
  State is Standby
    1 state change, last state change 00:02:28
  Virtual IP address is 10.30.30.254
  Active virtual MAC address is 0000.0c07.ac1e (MAC Not In Use)
    Local virtual MAC address is 0000.0c07.ac1e (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.328 secs
  Preemption enabled
  Active router is 10.30.30.2, priority 100 (expires in 9.728 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Vl30-30" (default)
Vlan40 - Group 40
  State is Standby
    1 state change, last state change 00:02:26
  Virtual IP address is 10.40.40.254
  Active virtual MAC address is 0000.0c07.ac28 (MAC Not In Use)
    Local virtual MAC address is 0000.0c07.ac28 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.416 secs
  Preemption enabled
  Active router is 10.40.40.2, priority 100 (expires in 10.768 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Vl40-40" (default)
Vlan50 - Group 50
  State is Standby
    1 state change, last state change 00:02:27
  Virtual IP address is 10.50.50.254
  Active virtual MAC address is 0000.0c07.ac32 (MAC Not In Use)
    Local virtual MAC address is 0000.0c07.ac32 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.512 secs
  Preemption enabled
  Active router is 10.50.50.2, priority 100 (expires in 8.656 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Vl50-50" (default)
```
### SW2
On voit que le switch 2 est actif : `State is Active`

On retrouve l'adresse IP virtuelle : `Virtual IP address is 10.10.10.254`

Le routeur actif : `Active router is local`

Le routeur passif (SW1) : `Standby router is 10.10.10.1, priority 100 (expires in 9.504 sec)`

La priorité : `Priority 100 (default 100)`
```
SW2#$ sh standby
Vlan10 - Group 10
  State is Active
    2 state changes, last state change 00:02:28
  Virtual IP address is 10.10.10.254
  Active virtual MAC address is 0000.0c07.ac0a (MAC In Use)
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.912 secs
  Preemption enabled
  Active router is local
  Standby router is 10.10.10.1, priority 100 (expires in 9.504 sec)
  Priority 100 (default 100)
  Group name is "hsrp-Vl10-10" (default)
Vlan20 - Group 20
  State is Active
    2 state changes, last state change 00:02:28
  Virtual IP address is 10.20.20.254
  Active virtual MAC address is 0000.0c07.ac14 (MAC In Use)
    Local virtual MAC address is 0000.0c07.ac14 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.736 secs
  Preemption enabled
  Active router is local
  Standby router is 10.20.20.1, priority 100 (expires in 9.888 sec)
  Priority 100 (default 100)
  Group name is "hsrp-Vl20-20" (default)
Vlan30 - Group 30
  State is Active
    2 state changes, last state change 00:02:17
  Virtual IP address is 10.30.30.254
  Active virtual MAC address is 0000.0c07.ac1e (MAC In Use)
    Local virtual MAC address is 0000.0c07.ac1e (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.704 secs
  Preemption enabled
  Active router is local
  Standby router is 10.30.30.1, priority 100 (expires in 10.384 sec)
  Priority 100 (default 100)
  Group name is "hsrp-Vl30-30" (default)
Vlan40 - Group 40
  State is Active
    2 state changes, last state change 00:02:27
  Virtual IP address is 10.40.40.254
  Active virtual MAC address is 0000.0c07.ac28 (MAC In Use)
    Local virtual MAC address is 0000.0c07.ac28 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.576 secs
  Preemption enabled
  Active router is local
  Standby router is 10.40.40.1, priority 100 (expires in 10.592 sec)
  Priority 100 (default 100)
  Group name is "hsrp-Vl40-40" (default)
Vlan50 - Group 50
  State is Active
    2 state changes, last state change 00:02:29
  Virtual IP address is 10.50.50.254
  Active virtual MAC address is 0000.0c07.ac32 (MAC In Use)
    Local virtual MAC address is 0000.0c07.ac32 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.696 secs
  Preemption enabled
  Active router is local
  Standby router is 10.50.50.1, priority 100 (expires in 9.152 sec)
  Priority 100 (default 100)
  Group name is "hsrp-Vl50-50" (default)
```

# Mise en place des ACL's
## 1. — Le VLAN IT (30) doit pouvoir communiquer avec tous les autres VLANs et tous les serveurs.
```
ip access-list extended VLAN30_IT_ALLOW_ANY
	permit ip 10.30.30.0 0.0.0.255 any (1318 matches)
	permit icmp 10.30.30.0 0.0.0.255 any
	permit tcp 10.30.30.0 0.0.0.255 any
	permit udp 10.30.30.0 0.0.0.255 any
```

## 2. — Les autres VLANs ne doivent pas pouvoir communiquer directement entre eux.

## 3. — Le VLAN RH (40) doit pouvoir accéder à son serveur dédié dans le VLAN IT.
### Configuration de l'ACL
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
### Vérifications
Les RH peuvent ping leur serveur dédié **(10.30.30.100)** mais ne peuvent pas ping le serveur de développement **(10.30.30.101)**
```
PC-RH> ping 10.30.30.100

84 bytes from 10.30.30.100 icmp_seq=1 ttl=63 time=8.788 ms
84 bytes from 10.30.30.100 icmp_seq=2 ttl=63 time=9.104 ms
^C
PC-RH> ping 10.30.30.101

*10.40.40.2 icmp_seq=1 ttl=255 time=10.781 ms (ICMP type:3, code:13, Communication administratively prohibited)
*10.40.40.2 icmp_seq=2 ttl=255 time=7.494 ms (ICMP type:3, code:13, Communication administratively prohibited)
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
### Vérifications
Les DEV peuvent ping leur serveur dédié **(10.20.20.100)** mais ne peuvent pas ping le serveur de développement **(10.20.20.101)**
```
PC-DEV> ping 10.20.20.100

84 bytes from 10.20.20.100 icmp_seq=1 ttl=63 time=8.788 ms
84 bytes from 10.20.20.100 icmp_seq=2 ttl=63 time=9.104 ms
^C
PC-DEV> ping 10.20.20.101

*10.40.40.2 icmp_seq=1 ttl=255 time=10.781 ms (ICMP type:3, code:13, Communication administratively prohibited)
*10.40.40.2 icmp_seq=2 ttl=255 time=7.494 ms (ICMP type:3, code:13, Communication administratively prohibited)
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

## Appliquer les priorités
On applique les priorités appropriées sur chaque switch avec les commandes : 
```
# Configuration pour l'instance 1
spanning-tree mst 1 priority 8192

# Configuration pour l'instance 2
spanning-tree mst 2 priority 8192

# Configuration pour l'instance 3
spanning-tree mst 3 priority 8192
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
