# Introduction
Lorem ipsum

# Linux Bridge VS OVS
## Linux Bridge
Lorem ipsum
## OVS
Lorem ipsum

# Bond (aggrégat de lien)

## Les différents types

- `Round-robin (balance-rr)`: Transmit network packets in sequential order from the first available network interface (NIC) slave through the last. This mode provides load balancing and fault tolerance.
- `Active-backup (active-backup)`: Only one NIC slave in the bond is active. A different slave becomes active if, and only if, the active slave fails. The single logical bonded interface’s MAC address is externally visible on only one NIC (port) to avoid distortion in the network switch. This mode provides fault tolerance.
- `XOR (balance-xor)`: Transmit network packets based on [(source MAC address XOR’d with destination MAC address) modulo NIC slave count]. This selects the same NIC slave for each destination MAC address. This mode provides load balancing and fault tolerance.
- `Broadcast (broadcast)`: Transmit network packets on all slave network interfaces. This mode provides fault tolerance.
- `IEEE 802.3ad Dynamic link aggregation (802.3ad)(LACP)`: Creates aggregation groups that share the same speed and duplex settings. Utilizes all slave network interfaces in the active aggregator group according to the 802.3ad specification.
- `Adaptive transmit load balancing (balance-tlb)`: Linux bonding driver mode that does not require any special network-switch support. The outgoing network packet traffic is distributed according to the current load (computed relative to the speed) on each network interface slave. Incoming traffic is received by one currently designated slave network interface. If this receiving slave fails, another slave takes over the MAC address of the failed receiving slave.
- `Adaptive load balancing (balance-alb)`: Includes balance-tlb plus receive load balancing (rlb) for IPV4 traffic, and does not require any special network switch support. The receive load balancing is achieved by ARP negotiation. The bonding driver intercepts the ARP Replies sent by the local system on their way out and overwrites the source hardware address with the unique hardware address of one of the NIC slaves in the single logical bonded interface such that different network-peers use different MAC addresses for their network packet traffic.


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
<img width="866" height="540" alt="empty_vmbr0" src="https://github.com/user-attachments/assets/db525872-03e6-44ab-b4a3-9b52e716fcb9" />  
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
A terminer !!!


> [!note]
> On assigne une IP directement à des cartes réseaux physique ou des aggrégats si on est sûr de ne pas les utiliser pour des machines virtuelles et quelles seront dediées aux flux hyperviseurs (migration, réplication ZFS, Corosync, Ceph etc...)

### Simulation de panne
1. Arrêt d'une interface
