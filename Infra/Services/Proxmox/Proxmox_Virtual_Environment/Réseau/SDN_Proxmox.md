# Introduction
Lorem ipsum

# La théorie
Lorem ipsum

## Zones
Lorem Ipsum  
Exemple d'utilisation : Une Zone = un client

### Simple
Comme un linux bridge mais en mieux :  
Profites de l'IPAM  
Gestion centralisée  
Ne permet pas la communication des VMs entre les nœuds (comme un vmbr)

### VLAN
Lorem Ipsum

### QinQ
Niche

### VXLAN
Simple en mieux : 
Communication des VMs inter-nœud

### EVPN
Lorem Ipsum

## VTNets
Lorem ipsum

## A voir
Lorem ipsum

## IPAM
pve (default native)  
netbox (api)  
php-ipam (api)  

---

# La pratique
Lorem ipsum

## Simple
Pour créer une zone de ce type voici un exemple de configuration :  

1. La première étape est de créer la zone simple dans `Datacenter` --> `SDN` --> `Zones`
<img width="1252" height="655" alt="simple_zone" src="https://github.com/user-attachments/assets/894ddb7e-1300-4a05-afaf-ac3d1442e596" />

- `ID` : Un simple nom d'affichage limité à 8 caractères
- `MTU` : Taille des paquets, laissez en auto dans le cas d'une zone simple
- `Nodes` : Vous pouvez limiter la création de cette zone à certains nœuds de votre cluster. A vous d'adapter en fonction de vos besoins
- `IPAM` : Par défaut `pve` car natif à Proxmox. Vous pouvez en choisir un autre. Le sujet sera abordé en détails dans la partie `IPAM`
- `DNS Server` + `Reverse DNS Server` + `DNS Zone` : Utile si vous avez un service DNS externe à Proxmox. Pas abordé ici
- `Automatic DHCP` : Active le DHCP sur la zone. L'IPAM ne sert à rien si pas coché 
<br><br>
2. Lorem Ipsum

## VXLAN
Lorem ipsum

## A voir
Lorem ipsum

---

# Erreurs connues

### Erreur 500 lors de la suppression d'un subnet dans un Vnet
1. Passez en CLI sur un des nœuds et éditez le fichier `/etc/pve/sdn/subnets.cfg`
2. Supprimez les lignes qui correspondent à votre subnet
3. Vérifiez que le subnet est passé dans l'état `Deleted` dans `Datacenter` --> `SDN` --> `VNets` --> <votre-VNet>
4. Si bien supprimé, appliquez les changements dans `Datacenter` --> `SDN`
