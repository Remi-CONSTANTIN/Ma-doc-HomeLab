# Introduction
Lorem ipsum

# La théorie
Lorem ipsum

## Zones
Lorem Ipsum  
Exemple d'utilisation : Une Zone = un client

### Simple
Utiliser un Linux bridge (vmbr) ou un VNet issue d'une zone simple est assez similaire sur certains points mais en y regardant de plus près on remarque que le SDN arrive à tirer son épingle du jeu.  

Pour comprendre cela, comparons les : 
|Linux bridge|Zone simple|
|------------|-----------|
|Permet de créer un réseau isolé|Idem|
|Ne permet pas de faire communiquer des VMs situées sur des PVE différents|Idem|
|Peut donner un accès à internet en l'associant à une carte physique|Peut donner un accès à internet à internet en activant le `SNAT` dans le `subnet` de son `Vnet`|
|Pour avoir le même "vmbr" sur tout vos nœuds, il faut tous les créer à la main|Doit être configuré dans l'onglet Datacenter puis est répliqué sur tout les nœuds|
|Nécessite une machine connectée au "vmbr" pour avoir le DHCP|Profite du DHCP et de l'IPAM du SDN Proxmox|

En résumé, les deux sont assez similaire dans les fonctionnalités qu'ils proposent mais le SDN apporte quelques fonctionnalités supplémentaires non négligeables. L'IPAM intégré est particulièrement utile pour les laboratoires.
<br><br>
<br><br>

### VLAN
Contrairement à la zone simple ou au Linux bridge qui créés un réseau local au PVE, le Linux Bridge (tagué) et la zone VLAN n'isolent pas eux même les flux et délèguent cela à l'infrastructure réseau de l'entreprise.  
Comme pour la zone simple, la zone VLAN ne révolutionne rien mais vient apporter quelques avantages liés au SDN

Encore une fois, une comparaison des deux sera plus simples pour comprendre : 
|Linux Bridge|Zone VLAN|
|-|-|
|Permet de taguer les flux réseau d'une machine|Idem|
|Nécessite de taguer la carte réseau de la machine ou de créer un Linux VLAN le PVE pour avoir des flux tagués (Linux VLAN moins utilisé)|Nécessite d'utiliser le VNet de la zone VLAN comme carte réseau pour avoir des flux tagués|
|N'attribue pas d'IP car consiste juste en un tag ou un Linux VLAN dédié qui n'apporte pas la fonctionnalité|Ne fournis pas non plus de DHCP car consiste seulement en un tag de flux|

En résumé, les deux fournissent le même résultat, c'est à dire de taguer les flux d'une carte réseau afin que l'infrastructure réseau de la société puisse assigner les flux à un VLAN.
Ils sont donc très similaire dans les fonctionnalités qu'ils proposent mais le SDN apporte quelques fonctionnalités supplémentaires non négligeables. Très utile pour les environnements d'entreprise qui utilisent la plupart du temps les VLANs pour segmenter leur réseau.
<br><br>
<br><br>

### QinQ
La zone QinQ (802.1Q in 802.1Q) permet de d'outrepasser la limite des 4096 Vlan imposée par la norme 802.1Q en mettant un tag Vlan dans un autre tag Vlan.  
Le `C-VLAN` correspond au tag interne et le `S-VLAN` à l'externe, ajouté et géré par le fournisseur ou l'hébergeur.  

 **Exemple**  
1. Le routeur de l'opérateur à Paris reçoit les données des clients  
2. Il enferme toutes les trames du Client A dans un S-VLAN 100 (On aura donc le tag 10 dans le tag 100)
3. Il enferme toutes les trames du Client B dans un S-VLAN 200 (On aura donc le tag 10 dans le tag 200)
4. Sur le câble en fibre optique, les switchs de l'opérateur ne regardent que l'étiquette externe (100 ou 200). Il n'y a donc plus aucun conflit
5. À Lyon, l'opérateur retire l'étiquette externe (100 et 200) et livre les données. Le Client A récupère son VLAN 10 intact, sans jamais savoir qu'il a traversé le pays dans une plus grosse enveloppe
<br><br>
<br><br>

### VXLAN
Simple en mieux : 
Communication des VMs inter-nœud
<br><br>
<br><br>

### EVPN
Lorem Ipsum
<br><br>
<br><br>

## VTNets
Lorem ipsum
<br><br>
<br><br>

## IPAM
pve (default native)  
netbox (api)  
php-ipam (api)  
<br><br>
<br><br>

## A voir
Lorem ipsum
<br><br>
<br><br>

---

# En pratique
Afin d'appliquer ces connaissances et de mieux comprendre comment utiliser ce savoir nouvellement apprit, vous êtes libre de suivre ces tutoriels de mise en place.

## Simple
Il sera abordé ici la création et l'utilisation d'une zone de type `simple`
<br><br>
1. La première étape est de créer la zone simple dans `Datacenter` --> `SDN` --> `Zones`
<img width="800" height="419" alt="simple_zone" src="https://github.com/user-attachments/assets/894ddb7e-1300-4a05-afaf-ac3d1442e596" />

- `ID` : Un simple nom d'affichage limité à 8 caractères
- `MTU` : Taille des paquets, laissez en auto dans le cas d'une zone simple
- `Nodes` : Vous pouvez limiter la création de cette zone à certains nœuds de votre cluster. A vous d'adapter en fonction de vos besoins
- `IPAM` : Par défaut `pve` car natif à Proxmox. Vous pouvez en choisir un autre. Le sujet sera abordé en détails dans la partie `IPAM`
- `DNS Server` + `Reverse DNS Server` + `DNS Zone` : Utile si vous avez un service DNS externe à Proxmox. Pas abordé ici
- `Automatic DHCP` : Active le DHCP sur la zone. L'IPAM ne sert à rien si vous choisissez de ne pas activer le DHCP
<br><br>

2. Allez dans `Datacenter` --> `SDN` --> `VNets` et créez un nouveau `Vnet`
<img width="779" height="419" alt="vnets_simple" src="https://github.com/user-attachments/assets/8532beba-d4fb-4ee9-92c5-4440d8fa1cfb" />


- `Name` : Nom d'affichage du Vnet
- `Alias` : Description ou alias du Vnet
- `Zone` : La zone à utiliser. Dans notre cas, la zone `Simple1`
- `Isolate Ports` : Isole les machines entre elles dans le réseau sans couper l'accès à la passerelle
- `VLAN Aware` : Autorise les flux tagués si vous utilisez un routeur virtuel dans le réseau
<br><br>

3. Dans ce même Vnet, créez un réseau dans la partie `Subnets` on complétant les deux onglets : 

**General**  
<img width="1141" height="383" alt="subnet1_vnets_simple" src="https://github.com/user-attachments/assets/2cda92d4-804f-435e-ad16-68051a30ed29" />

- `Subnet` : L'adressage IP du réseau à créér
- `Gateway` : La passerelle associée
- `SNAT` : Permet d'autoriser le NAT à travers le nœud pour avoir accès à internet par exemple
- `DNS Zone Prefix` : Permet d'associer un prefix DNS aux machines du réseau si vous l'avez configuré dans la zone 
<br><br>

**DHCP Ranges**  
Vous pouvez ajouter un/des étendue(s) DHCP dans le second onglet
<img width="1267" height="430" alt="dhcp_subnet1_vnets_simple" src="https://github.com/user-attachments/assets/69d28267-aa82-4f24-8009-6919b940b1d7" />
<br><br>

4. Une fois tout cela fait, il faut appliquer les modifications dans `Datacenter` --> `SDN`
<br><br>

5. Pour utiliser ce nouveau switch virtuel, il nous suffit de créer/modifier une carte réseau et de choisir `VNsimp1` au lieu d'un Linux Bridge (vmbr)
<img width="1051" height="364" alt="vnet_VNsimp1_debian1" src="https://github.com/user-attachments/assets/20bbabd6-467e-48b1-b4cd-fe9938ca7b4e" />
<br><br>

6. Démarrez la machine pour vérifier l'IP
<img width="1325" height="262" alt="debian1_ip-a" src="https://github.com/user-attachments/assets/dc808bfb-b0dd-488f-a690-6736e6d0c280" />

On voit ici que nous avons obtenus la première IP de l'étendue DHCP
<br><br>

7. Pour être sur que cette IP a bien été distribuée par le DHCP, vous n'avez qu'à confirmer la présence du bail dans l'IPAM.
Pour cela rendez-vous dans `Datacenter` --> `SDN` --> `IPAM`
<img width="1361" height="479" alt="debian1_ipam_dhcp_lease" src="https://github.com/user-attachments/assets/b80d194b-9afa-4798-8a9f-09990b69f561" />

On retrouve bien notre machine et son IP !
<br><br>
C'est tout pour la mise en place d'un zone simple
<br><br>
Le routeur de l'opérateur à Paris reçoit les données des clients.

Il enferme toutes les trames du Client A dans un S-VLAN 100. (On aura donc le tag 10 dans le tag 100).

Il enferme toutes les trames du Client B dans un S-VLAN 200. (On aura donc le tag 10 dans le tag 200).

Sur le câble en fibre optique, les switchs de l'opérateur ne regardent que l'étiquette externe (100 ou 200). Il n'y a donc plus aucun conflit.

À Lyon, l'opérateur retire l'étiquette externe (100 et 200) et livre les données. Le Client A récupère son VLAN 10 intact, sans jamais savoir qu'il a traversé le pays dans une plus grosse enveloppe.
## VLAN
**A mettre en place sur proxmox physique car marche pas sur maquette**

## QinQ
Lorem Ipsum

## VXLAN
Lorem ipsum

## EVPN
Lorem ipsum

---

# Erreurs connues

### Erreur 500 lors de la suppression d'un subnet dans un Vnet
1. Passez en CLI sur un des nœuds et éditez le fichier `/etc/pve/sdn/subnets.cfg`
2. Supprimez les lignes qui correspondent à votre subnet
3. Vérifiez que le subnet est passé dans l'état `Deleted` dans `Datacenter` --> `SDN` --> `VNets` --> <votre-VNet>
4. Si bien supprimé, appliquez les changements dans `Datacenter` --> `SDN`
