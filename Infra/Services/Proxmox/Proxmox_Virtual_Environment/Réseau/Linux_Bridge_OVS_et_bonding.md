# Introduction
Nous parlerons ici de la partie réseau des noeuds PVE sans se pencher sur la partie SDN que nous verrons dans une autre partie.  


# Linux Bridge VS OVS
## Linux Bridge
Le Linux Bridge est le commutateur logiciel historique du noyau Linux, il a donc été épprouvé par des millions d'utilisateurs. Il supporte les VLAN (ce qui n'a pas toujours été le cas) et le STP (assez daté et pas très efficace). Il est d'ailleurs utilisé dans le SDN de Proxmox.  
Convient très bien à 90% des utilisateurs mais reste assez limité

## OVS
Plus qu'un simple "Bridge", c'est un commutateur multicouche virtuel complet, conçu par et pour l'industrie du Cloud. Il possède sa propre base de données interne (ovsdb) pour stocker sa configuration.  
Permet beaucoup de choses comme : brider la bande passante par VM, être pilotable via API par des contrôleurs SDN externes (OpenDaylight) grâce à OpenFlow, supporte le RSTP. Il permet aussi d'envoyer des statistiques détaillées de tous les flux qui le traversent vers un serveur de supervision, ou ordonner la copie exacte du trafic d'une VM vers une autre pour qu'un pare-feu ou une sonde de sécurité puisse l'analyser.  
C'est une solution complète, plus poussée mais plus complexe que le Linux bridge et pas forcément utile à tout le monde  

# Bond (aggrégat de lien)

## Les différents types

- `Round-robin (balance-rr)`: Transmet les paquet une fois par interface (paquet 1 sur interface A, paquet 2 sur interface B, paquet 3 sur interface A et ansi de suite). Cela necessite que le switch physique soit configuré avec une agrégation statique (Static EtherChannel). Nous avons donc une tolérance de panne et une répartition de charge
- `Active-backup (active-backup)`: Basiquement un mode avec une interface principale et une interface de secours. Cela ne fournis que de la tolérance de panne.  
- `XOR (balance-xor)`: Fait de la répartition de charge en se basant sur un hash (calculé sur les adresses MAC ou IP source/destination). Contrairement au Round-Robin, ce mode garantit que tous les paquets d'une même session (par exemple, un transfert de fichier) passeront toujours par le même câble, évitant ainsi le problème des paquets désordonnés. Cela nécessite que le switch doit être configuré en agrégation statique  
- `Broadcast (broadcast)`: Envoie les paquets sur toutes les interfaces afin de prévenir toute perte de paquets. Cela gaspille beaucoup de bande passante mais fournis une fiabilité extrème. Ici acune répartition de charge mais une tolérance de panne avancée
- `IEEE 802.3ad Dynamic link aggregation (802.3ad)(LACP)`: Protocole standard en datacenter. Permet l'aggrégation de lien de façon intelligente et l'additionnement de la bande passante*. Nous avons donc tolérance de panne et addition de la bande passante
- `Adaptive transmit load balancing (balance-tlb)`: Fonctionne comme `Active-backup` à la différence que ça répartis le traffic sortant sur les interfaces en fonction de la charge. Nous avons donc une répartition de charge pour les flux sortants et une tolérance de panne. De plus il ne necessite pas de toucher à la configuration des switchs
- `Adaptive load balancing (balance-alb)`: Fonctionne comme le `balance-tlb`, mais ajoute la répartition de charge pour les flux entrants. On peut profiter de cette répartition de charge sans toucher au switch grâce une mécanismee qui simule le LACP. Peu recommandé. Comme pour le `LACP`, permet d'augmenter la bande passante maximum et d'avoir une tolérance de panne.

> [!note]
> Le LACP ne permet pas d'augmenter la bande passante sur un seul flux. Il l'augmente en ajoutant le nombre de flux simultanés (métaphore : on ajoute une voie à l'autoroute, on n'augmente pas sa vitesse)

---

## Cas pratique
Afin de tester la théorie vue précédemment, nous allons mettre en place une redondance sur l'interface réseau qui nous permet d'accéder au noeud PVE (interface web, ssh etc..). Nous partons ici du principe que vous manipulez sur une machine ayant une configuration basique de son réseau et ne possédant par conséquent qu'une seule interface réseau.  
Pas besoin ici de cluster car les manipulations ne sont relative qu'à une seule machine.  

### Prérequis
- 1 noeud PVE sans configuration réseau existante (pas besoin de cluster)
- Avoir 1 carte réseau supplémentaire à disposition

### Ajout des cartes réseaux
1. Si ce n'est pas déjà fait, branchez votre carte réseau/adaptateur sur votre machine physique.  
Si vous avez choisis de virtualisez le Pve alors je pars du principe que vous savez ajouter une carte réseau sur une machine virtuelle...

2. Rendez-vous dans l'interface de gestion des interfaces réseaux en suivant le chemin : Votre-noeud --> `System` --> `Network`  
3. Vérifiez que votre nouvelle interface réseau apparaisse bien, même s'il elle n'est pas encore dans l'état `Active : yes` car elle sera activée automatiqement suite à nos manipulations.  <img width="1274" height="214" alt="new_ens19" src="https://github.com/user-attachments/assets/4af7a7c8-b6de-46b2-9478-f240cf58f1c3" />  

Dans mon cas, ma nouvelle interface `ens19` est présentee et pas encore activée.  
On notera aussi la présence de ma première carte réseau `nic0` et de son Linux Bridge asscocié `vmbr0` par lequel je passe actuellement pour avoir accès à l'interface web.

### Création du bond

> [!caution]
> N'appliquez pas les modifications que nous allons faire avant que je vous l'ai explicité précisé, au risque de perdre l'accès distant à votre noeud !

1. Avant de pouvoir créer le bond avec nos deux cartes réseaux (`ens19` et `nic0`) nous devons d'abord désassocier `nic0` du `vmbr0` en éditant le Linux Bridge :  
<img width="779" height="486" alt="empty_vmbr0" src="https://github.com/user-attachments/assets/db525872-03e6-44ab-b4a3-9b52e716fcb9" />  

Le champs `Bridge Port` doit être vide (pour l'instant)

2. Une fois cela fait, créez un `Bond` en cliquant sur le bouton `Create` puis en séléctionnant `Linux Bond`  
Il Vous sera demandé de compléter plusieurs champs :  
- `Name` : Nom de votre aggrégat  
- `IPv4/CIDR` : Ne mettez pas d'IP directement sur le bond car nous partons du principe que nous allons aussi l'utiliser pour des machines virtuelles  
- `Gateway (IPv4)` : Pas d'IP pour les mêmes raisons  
- `IPv6/CIDR` : Idem  
- `Gateway (IPv6)` : Idem  
- `Autostart` : On laisse coché car on veut que l'interface démarre automatiquement avec la machine  
- `Slaves` : C'est ici que nous associons nos deux cartes `ens19` et `nic0`  
- `Mode` : Comme vu précédemment, plusieurs modes s'offrent à nous, mais pour faire simple choisissez `Active-Backup` afin de ne pas dépendre de l'infrastructure réseau comme avec le `LACP`  
- `bond-primary` : Choisissez l'interface qui sera utilisée en priorité si les deux interfaces sont UP  
- `Comment` : Un commentaire, si besoin  

> [!note]
> On assigne une IP directement à des cartes réseaux physique ou des aggrégats si on est sûr de ne pas les utiliser pour des machines virtuelles et quelles seront dediées aux flux hyperviseurs (migration, réplication ZFS, Corosync, Ceph etc...)

<img width="737" height="386" alt="linux_bond" src="https://github.com/user-attachments/assets/3adb6322-5a52-4474-b0bb-2d200cc89ad4" />

3. Plus qu'à assigner ce bond à notre interface `vmbr0` afin d'avoir une interface redondée.  
Modifiez donc votre `vmbr0` et ajoutez `bond0` dans le champs `Bridge ports`  
<img width="6172" height="2564" alt="vmbr0_bond0" src="https://github.com/user-attachments/assets/d0a8ffa4-916b-4a55-a64d-1d228b95eb31" />  

**Vous avez maintenant terminé la mise en place de la redondance. Plus qu'à tester.**  

### Simulation de panne
Afin de tester si la redondance des interfaces fonctionne, vous n'avez qu'à tester d'éteindre votre interface principale (`bond-primary`) et vérifier que vous avez toujours accès à l'interface Web.  
- Si vous vitualisez les Pve, désactivez l'interface dans la partie `Hardware` de la machine  
- Si vous avez une machine physique, vous pouvez utiliser `ifdown`
