# Introduction
Cette procédure fait suite à la création d'un LUN sur mon NAS Synolgy que vous pouvez retrouvez [ici](https://github.com/tadaron/Mon-Home-Lab/blob/4146b9909a38964018f17ecdcbde4efbe6cc987b/Infra/Services/Synology/SAN-Synology_configuration.md).  
L'objectif est d'utiliser un LUN pour créer un datastore Proxmox pour y mettre, par exemple, des machines virtuelles, ISOs ou images de conteneurs.  

# Procédure
Pouvoir utiliser un LUN comme datastore Proxmox ne se fait malheureusement pas en 2 clique et necessite dans un premier temps la connexion du LUN puis son formatage.
Nous verrons comment faire cela ici.

## 1. Connexion du LUN
Rendez-vous sur votre cluster PVE dans l'onglet `Datacenter` --> `Storage` --> Cliquez sur `Add` --> Sélectionnez `iSCSI`

<img width="1100" height="344" alt="iSCSI_pve" src="https://github.com/user-attachments/assets/6eb0712e-05c1-4b99-a7e0-f65b065e5adf" />  

Complétez avec vos informations :  
- `ID` : Nom d'affichage de la cible `iSCSI` dans l'inteface proxmox (purement esthétique)  
- `Portal` : IP/Nom DNS de votre machine qui fournit le LUN (dans mon cas mon NAS Synology)   
- `Target` : Si l'IP/Nom DNS est bon, vous devriez avoir la liste des cible iSCSI proposée par votre SAN  
- `Nodes` : Vous pouvez restreindre ce stockage à certains noeuds PVE
- `Use LUNs directly` : **Important !** Si vous laissez coché cette case Proxmox va utiliser ce disque brut pour une seule machine virtuelle géante ou vous pourrez juste l'assigner directement à une VM.  
**Ce n'est pas du tout notre objectif ici car nous voulons l'utiliser comme datastore**

Une fois cela validé, restez dans le même onglet et passez à la prochaine étape

## 2. Formatage du nouveau stockage
Re-cliquez sur `Add`, puis, cette fois, séléctionnez `LVM`  
<img width="605" height="395" alt="lvm_format_lun-proxmox1" src="https://github.com/user-attachments/assets/6cc44384-b9d7-48a2-a4ec-3cf2269d7de8" />

Remplissez toutes les informations relative au LUN à formater : 
- `ID` : Comme d'habitude c'est le nom d'affichage de ce stockage dans Proxmox
- `Base Storage` : Séléctionnez la cible iSCSI sur laquelle aller chercher le lUN (j'ai nommé la mienne `LUN-Proxmox1` par abus de langage)
- `Base Volume` : Le LUN visé. Vous pouvez avoir plusieurs LUN sous la même cible iSCSI
- `Volume group` : A vous de choisir le nom du VG qui va accueillir les données
- `Content` : Le type de contenu que vous voulez y mettre (`Disk image` et/ou `Container`)
- `Shared` : Permet de spécifier au cluster PVE que c'est un emplacement partagé et pas un stockage local. Cela permet la migration de machine inter-noeuds sans coupure
- `Wipe Removed Disk` : Habituellement, lors d'une suppression de disque seul l'index est modifié mais pas rééllement les données sur le disque. Cette option permet d'écraser les données par des zéros, cela peut etre utile dans des environnements sensibles (bancaire, médical etc...)
- `Allow Snapshots as Volume-Chain` :  En Beta sur PVE 9.2.3. Permet de faire des "snapshots" sur un stockage LVM qui ne le supporte habituellement pas.
