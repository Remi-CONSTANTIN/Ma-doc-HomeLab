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
Ne permet pas la communication des VMs entre les noeuds (comme un vmbr)

**Configuration**
Pour créer une zone de ce type voici un exemple de configuration :  
<img width="1252" height="655" alt="simple_zone" src="https://github.com/user-attachments/assets/894ddb7e-1300-4a05-afaf-ac3d1442e596" />  
- `ID` : Un simple nom d'affichage limité à 8 caractères
- `MTU` : Taille des paquets, laissez en auto dans le cas d'une zone simple
- `Nodes` : Vous pouvez limiter la création de cette zone à certains noeuds de votre cluster. A vous d'adapter en fonction de vos besoins
- `IPAM` : Par défaut `pve` car natif à Proxmox. Vous pouvez en choisir un autre. Le sujet sera abordé en détails dans la partie `IPAM`
- `DNS Server` + `Reverse DNS Server` + `DNS Zone` : Utile si vous avez un service DNS externe à Proxmox. Pas abordé ici
- `Automatic DHCP` : Active le DHCP sur la zone. L'IPAM ne sert à rien si pas coché 

### VLAN
Lorem Ipsum

### QinQ
Niche

### VXLAN
Simple en mieux : 
Communication des VMs inter-noeud

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
Lorem Ipsum

## VXLAN
Lorem ipsum

## A voir
Lorem ipsum
