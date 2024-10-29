# TP4 : Configuration IPv6

# Objectifs
* Comprendre les concepts fondamentaux d’IPv6
* Configurer des adresses IPv6 sur des routeurs et des commutateurs
* Mettre en place le routage IPv6 statique
* Tester la connectivité IPv6 de base

# Sujet
![image](https://hackmd.io/_uploads/ryM1bkpl1g.png)

## Introduction à IPv6
IPv6 est conçu pour remplacer IPv4 et offre de nombreux avantages, notamment un espace d’adressage beaucoup plus grand et des fonctionnalités de sécurité intégrées. 

Dans ce TP, nous allons explorer la configuration de base d’IPv6 sur des équipements réseau.

## Étude de Cas : Introduction d’IPv6 dans une Petite Entreprise
Vous êtes chargé d’introduire IPv6 dans le réseau de la petite entreprise DEF. 

L’entreprise souhaite commencer à utiliser IPv6 sur une partie de son réseau tout en maintenant son infrastructure IPv4 existante. Voici les besoins :
* Configurer IPv6 sur 3 routeurs et 2 commutateurs
* Mettre en place un routage IPv6 statique entre les routeurs
* Configurer des adresses IPv6 link-local et globales
* Tester la connectivité IPv6 de base

# Tâches à Réaliser 
## 1. Configurez les adresses IPv6 link-local et globales sur les interfaces des routeurs et des commutateurs.
Chaque routeur aura ses interfaces configurées avec des adresses Link-Local et Globales pour chaque sous-réseau en suivant cette nouvelle logique.

### Routeur R1
```
interface GigabitEthernet0/0
 no ip address
 media-type gbic
 speed 1000
 duplex full
 negotiation auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:1::1/64
!
interface GigabitEthernet1/0
 no ip address
 negotiation auto
 ipv6 address FE80::2 link-local
 ipv6 address 2001:DB8:3::2/64
!
interface GigabitEthernet2/0
 no ip address
 negotiation auto
 ipv6 address FE80::3 link-local
 ipv6 address 2001:DB8:4::1/64
```
Explications :
* GigabitEthernet0/0 (Vers R2) utilise le sous-réseau 2001:db8:1::/64. 
* GigabitEthernet1/0 (Vers R3) utilise le sous-réseau 2001:db8:3::/64.
* GigabitEthernet2/0 (Vers SW1 pour PC1 et PC2) utilise le sous-réseau 2001:db8:4::/64.

### Routeur R2
```
interface GigabitEthernet0/0
 ipv6 address FE80::4 link-local
 ipv6 address 2001:DB8:1::2/64
!
interface GigabitEthernet1/0
 ipv6 address FE80::5 link-local
 ipv6 address 2001:DB8:2::2/64
!
interface GigabitEthernet2/0
 ipv6 address FE80::6 link-local
 ipv6 address 2001:DB8:5::1/64
```

Explications :
* `GigabitEthernet0/0` (connexion à R1) utilise le sous-réseau 2001:db8:1::/64. 
* `GigabitEthernet1/0` (connexion à R3) utilise le sous-réseau 2001:db8:2::/64.
* `GigabitEthernet2/0` (connexion à SW2 pour PC3 et PC4) utilise le sous-réseau 2001:db8:5::/64.
### Routeur R3
```
interface GigabitEthernet0/0
 ipv6 address FE80::7 link-local
 ipv6 address 2001:DB8:3::1/64
!
interface GigabitEthernet1/0
 ipv6 address FE80::8 link-local
 ipv6 address 2001:DB8:2::1/64
```
Explications :
* GigabitEthernet0/0 (connexion à R1) utilise le sous-réseau 2001:db8:3::/64.
* GigabitEthernet0/1 (connexion à R2) utilise le sous-réseau 2001:db8:2::/64.

## 2. Mettez en place un routage IPv6 statique entre les routeurs.
### R1
```
ipv6 route 2001:DB8:2::/64 GigabitEthernet0/0 FE80::4
ipv6 route 2001:DB8:2::/64 GigabitEthernet1/0 FE80::7
ipv6 route 2001:DB8:5::/64 GigabitEthernet0/0 FE80::4
```
### R2
```
ipv6 route 2001:DB8:3::/64 GigabitEthernet0/0 FE80::1
ipv6 route 2001:DB8:3::/64 GigabitEthernet1/0 FE80::8
ipv6 route 2001:DB8:4::/64 GigabitEthernet0/0 FE80::1
```
### R3
```
ipv6 route 2001:DB8:1::/64 GigabitEthernet0/0 FE80::1
ipv6 route 2001:DB8:1::/64 GigabitEthernet1/0 FE80::4
ipv6 route 2001:DB8:4::/64 GigabitEthernet0/0 FE80::2
ipv6 route 2001:DB8:5::/64 GigabitEthernet1/0 FE80::5
```

## 3. Implémentez le Dual Stack sur un routeur pour permettre la coexistence d’IPv4 et IPv6.
Dual stack est un mode permetttant le fonctionnement simmultané d'IPv4 et IPv6


## 4. Configurez le protocole ICMPv6 pour permettre la découverte des voisins (Neighbor Discovery).
ICMPv6 est activé par défaut sur les routeurs Cisco, il n'est donc pas nécessaire de configurer explicitement ce protocole. 

Cependant, il faut vérifier que les interfaces et le routage IPv6 est activé pour s'assurer du bon fonctionnement du Neighbor Discovery Protocol (NDP).


## 5. Testez la connectivité IPv6 end-to-end en utilisant les commandes ping et traceroute pour IPv6.
On utilise les commandes suivantes pour vérifier la connectivité entre les routeurs :

Depuis R1 vers R2 (adresse globale) : `R1# ping ipv6 2001:DB8:1::2`

Depuis R2 vers R3 (adresse link-local) : `R2# ping ipv6 FE80::8`

Traceroute entre R1 et R3 pour tester le chemin : `R1# traceroute ipv6 2001:DB8:3::2`

Ces commandes permettent de vérifier la connectivité et le bon fonctionnement du routage IPv6.

## 6. Vérifiez la table de routage IPv6 et assurez-vous que les routes statiques sont correctement configurées.
```
show ipv6 route
```
