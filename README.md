# TP2 - Redondance (RSTP, LACP, HSRP)

## Objectifs
* Comprendre les concepts de redondance réseau
* Configurer des protocoles de redondance (STP, EtherChannel, HSRP)
* Analyser l’impact de la redondance sur la disponibilité du réseau

## Sujet
![image](https://hackmd.io/_uploads/SyptuZ7kJl.png)

### 1. Introduction à la Redondance Réseau
La redondance réseau est cruciale pour assurer la continuité des services et
minimiser les temps d’arrêt. Dans ce TP, nous allons explorer trois mécanismes
de redondance essentiels :
1. Spanning Tree Protocol (STP) : Prévient les boucles de commutation.
2. EtherChannel : Agrège plusieurs liens physiques en un seul lien
logique.
3. Hot Standby Router Protocol (HSRP) : Fournit une redondance
de passerelle par défaut.

### 2. Étude de Cas : Entreprise ABC
Vous êtes chargé d’implémenter la redondance réseau pour l’entreprise ABC,
une société de commerce électronique qui nécessite une haute disponibilité.

L’entreprise a les besoins suivants :
* 2 commutateurs de distribution connectés à 4 commutateurs d’accès
* Liens redondants entre les commutateurs de distribution et d’accès
* 2 routeurs pour la connectivité Internet redondante
* Une architecture qui minimise les temps d’arrêt en cas de panne d’un équipement ou d’un lien

### 3. Tâches à Réaliser
1. Configurez RSTP sur les commutateurs pour prévenir les boucles de
couche 2.
2. Mettez en place des EtherChannels entre les commutateurs de distribution
et d’accès.
3. Configurez HSRP sur les routeurs pour la redondance de la passerelle
par défaut.
4. Testez la résilience de votre configuration en simulant des pannes de
liens et d’équipements.

### Ressources
* Commutation Ethernet : https://cisco.goffinet.org/ccna/ethernet/commutation-ethernet-cisco/
* EtherChannel : https://cisco.goffinet.org/ccna/redondance-de-liens/cisco-etherchannel-configuration-verification-depannage/
* Spaning-Tree : https://cisco.goffinet.org/ccna/disponibilite-lan/lab-repartition-charge-rapid-spanning-tree-rspt-load-balancing/
* HSRP : https://cisco.goffinet.org/ccna/disponibilite-lan/redondance-de-passerelle-host-standby-router-protocol-hsrp/


### Rendus attendus
Dans le rendu sont attendus :
* Un schéma de l’architecture réseau implémentée
![image](https://hackmd.io/_uploads/H1gOAMmkyx.png)
* Les configurations STP, EtherChannel, et HSRP pour chaque équipement concerné
* Un rapport de test détaillant les scénarios de panne simulés et les résultats observés : Voir ci-dessous


## Rendu

### RSTP
Le Spanning Tree Protocol (STP) évite les boucles de niveau 2 dans les topologies redondantes. Le RSTP est une version améliorée, beaucoup plus rapide, ce qui est crucial dans les environnements nécessitant une haute disponibilité.

#### Configuration sur les commutateurs de distribution et d'accès : 
Tous les commutateurs doivent être configurés en mode RSTP pour garantir une convergence rapide en cas de panne.
```
spanning-tree mode rapid-pvst
```
Le mode Rapid PVST active une instance STP par VLAN avec une convergence rapide. Ce mode améliore la tolérance aux pannes en assurant que les boucles sont rapidement détectées et résolues.


### LACP
L'**EtherChannel** est utilisé pour **agréger plusieurs liens physiques entre les commutateurs en un seul lien logique**, **augmentant** ainsi **la bande passante** *(en mode actif)* et offrant de la **redondance**. Si un des liens tombe, le trafic est **redirigé automatiquement** vers les autres.

On aurait pu configurer le **LACP** en **mode passif**, dans ce cas, les **liens secondaires** ne sont utilisés qu'**en cas de panne d'un lien actif**.

Le LACP a été mis en place sur les interfaces **entre les commutateurs de distribution et d’accès**.

#### Switchs de distribution ( `IOU1` / `IOU2` ) : 
```
interface range Ethernet 0/0-1
channel-group 1 mode active

interface range Ethernet 0/2-3
channel-group 2 mode active

interface range Ethernet 1/0-1
channel-group 3 mode active

interface range Ethernet 1/2-3
channel-group 4 mode active

interface range Ethernet 2/0-1
channel-group 5 mode active

interface port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
interface port-channel 2
switchport trunk encapsulation dot1q
switchport mode trunk
interface port-channel 3
switchport trunk encapsulation dot1q
switchport mode trunk
interface port-channel 4
switchport trunk encapsulation dot1q
switchport mode trunk
interface port-channel 5
switchport trunk encapsulation dot1q
switchport mode trunk
```

#### Switchs d'accés ( `IOU3` / `IOU4` / `IOU5` / `IOU6` ) :
```
interface range Ethernet 0/0-1
channel-group 1 mode active

interface range Ethernet 0/2-3
channel-group 2 mode active

interface port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
interface port-channel 2
switchport trunk encapsulation dot1q
switchport mode trunk
```

### Mise en place HSRP
Le **HSRP** fournit une **redondance de passerelle par défaut** en permettant à **plusieurs routeurs** de partager **une adresse IP virtuelle**, qui sera **visible par le reste du réseau comme une seule passerelle virtuelle**. Si l’un des routeurs échoue, l’autre prend automatiquement le relais.

A noter : 
* On définit une **adresse IP virtuelle** qui sera **partagée entre les routeurs**
* Pour les deux routeurs, il faut configurer une **adresse IP statique dans le même sous-réseau que l'adresse IP virtuelle qui sera utilisée pour HSRP**  *(`gigabitEthernet 3/0` dans ce cas)*
* **Un seul routeur** est **actif** lorsqu'on utilise **HSRP**. Le **routeur actif** est celui qui possède la **priorité la plus élevée** *(`100` par défaut)*. 

Dans mon cas : 
* L'adresse IP virtuelle partagée par les routeurs sera 10.1.1.254 : `standby 10 ip 10.1.1.254`
* L'adresse IP de R1 est 10.1.1.1
* L'adresse IP de R2 est 10.1.1.2
* La priorité de R1 est de 110 (C'est donc le routeur principal) - `standby 10 priority 110`
* La priorité de R2 est de 100 (par défaut) - `standby 10 priority 100`
* `standby 10` : Numéro du **groupe HSRP** *(Nos deux routeurs seront dans le même groupe)*
#### R1

Sur R1, on configure une adresse IP pour l'interface `GigabitEthernet3/0` (vers le LAN) dans le même sous réseau que l'adresse IP virtuelle utilisée pour le HSRP (`10.1.1.1`)
On configure l'adresse IP virtuelle : `standby 10 ip 10.1.1.254`
On définit une **priorité** de `110`, ce sera donc le routeur principal.
```
interface  gigabitEthernet 3/0
interface GigabitEthernet3/0
ip address 10.1.1.1 255.255.255.0
standby 10 ip 10.1.1.254
standby 10 priority 110
standby 10 preempt
```
#### R2
* Sur **R2**, on configure également une adresse IP différente pour l'interface `GigabitEthernet3/0` (vers le LAN) dans le même sous réseau que l'adresse IP virtuelle utilisée pour le HSRP (`10.1.1.2`)
* On configure l'adresse IP virtuelle : `standby 10 ip 10.1.1.254`
* On définit une **priorité** de `110`, ce sera donc le routeur principal.
```
interface GigabitEthernet3/0
ip address 10.1.1.2 255.255.255.0
standby 10 ip 10.1.1.254
standby 10 priority 100
standby 10 preempt
```



#### Explication de la configuration
`standby 10 ip 10.1.1.254` configure l'adresse IP virtuelle (`10.1.1.254`) que les clients utiliseront comme passerelle. Ce sera l'adresse IP unique de notre routeur 'virtuel'.

La **priorité** de **R1** (`110`) est **plus élevée** que celle de **R2** (`100`), ce qui fait de **R1** le **routeur principal**. Elle est définie à l'aide de la commande suivante : 
* R1 :`standby 10 priority 110` 
* R2 :`standby 10 priority 100`

`standby 10 preempt` assure que si le routeur principal revient en ligne après une panne, il redeviendra automatiquement le routeur actif.

### Vérifications 
#### LACP  - Test de deconnexion d'un lien

![image](https://hackmd.io/_uploads/B1hRS7Qkkg.png)
![image](https://hackmd.io/_uploads/SkJZUXX1kx.png)

On se place sur un VPC pour lancer un ping continu a travers un aggrégat.
```
ping -t 10.1.1.25
```

On débranche un lien `Ethernet 0/0` entre un commutateur de distribution et un commutateur d'accès pour observer le comportement du LACP. 
```
interface Ethernet 0/0
shutdown
```

Le trafic continue à circuler sur le lien restant `Ethernet 0/1`
Le LACP est fonctionnel.

#### Panne d’un routeur (vérification HSRP)
**R1** est le routeur **principal** et **actif** et l'interface sur laquelle j'ai configuré le HSRP est la `GigabitEthernet3/0`, on la repère avec cette ligne
![image](https://hackmd.io/_uploads/H10m2GX1Jx.png)

![image](https://hackmd.io/_uploads/SJ520MmJyx.png)

![image](https://hackmd.io/_uploads/BkDUbXmkkl.png)


##### Explications
Depuis un VPC, on lance un ping en continu vers l'adresse IP virtuelle (`10.1.1.254`)

Sur R1, on **éteint** l'interface `GigabitEthernet3/0` pour **simuler une panne** au bout de quelques secondes (`icmp_seq=5`)
![image](https://hackmd.io/_uploads/H1h6gQQk1x.png)

Le statut n'est plus actif : `*Oct  8 21:05:18.699: %LINK-5-CHANGED: Interface GigabitEthernet3/0, changed state to administratively down`

Dans le même temps, on constate qu'il y'a eut une légère interruption sur le ping :
```
PC1> ping 10.1.1.254 -t

84 bytes from 10.1.1.254 icmp_seq=1 ttl=255 time=9.345 ms
84 bytes from 10.1.1.254 icmp_seq=2 ttl=255 time=6.330 ms
84 bytes from 10.1.1.254 icmp_seq=3 ttl=255 time=6.840 ms
84 bytes from 10.1.1.254 icmp_seq=4 ttl=255 time=6.814 ms
10.1.1.254 icmp_seq=5 timeout
10.1.1.254 icmp_seq=6 timeout
```
Puis au bout de quelques secondes, le routeur R2 prend le relai et le ping reprend rapidement
![image](https://hackmd.io/_uploads/rJnKeX7Jke.png)

```
10.1.1.254 icmp_seq=6 timeout
10.1.1.254 icmp_seq=7 timeout
10.1.1.254 icmp_seq=8 timeout
10.1.1.254 icmp_seq=9 timeout
84 bytes from 10.1.1.254 icmp_seq=10 ttl=255 time=14.601 ms
84 bytes from 10.1.1.254 icmp_seq=11 ttl=255 time=5.516 ms
84 bytes from 10.1.1.254 icmp_seq=12 ttl=255 time=5.999 ms
84 bytes from 10.1.1.254 icmp_seq=13 ttl=255 time=6.828 ms
84 bytes from 10.1.1.254 icmp_seq=14 ttl=255 time=6.252 ms
84 bytes from 10.1.1.254 icmp_seq=15 ttl=255 time=7.620 ms
84 bytes from 10.1.1.254 icmp_seq=16 ttl=255 time=4.538 ms
```

#### Panne d’un commutateur
Éteins un commutateur de distribution pour voir si le réseau continue à fonctionner via l'autre commutateur de distribution grâce au RSTP.
