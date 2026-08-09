# Introduction
Vous trouverez ici mes notes sur l'utilisation du système de fichier `ZFS` sur Proxmox

# Pratique

## Supprimer un nœud PVE de l'architecture ZFS
L'objectif ici est de supprimer un nœud Pve de mon architecture ZFS car son seul disque disponible pour le ZFS n'est pas adapté à ce système de fichier.  

> [!warning]
> En effet les SSD SATA grand publiques ne possèdent pas assez de cache pour le ZFS. On préférera du NVME type entreprise (même si les modèles grand publiques fonctionnent) et du HDD de type "CRM"

1. Retirez tout le jobs de réplication ZFS vers ce nœud dans `Datacenter` --> `Replication`
<img width="1342" height="567" alt="poseidon_zfs_replication_job" src="https://github.com/user-attachments/assets/dd41ebee-acb8-4f05-9dc5-7066b0863a9f" />
Dans mon cas, j'ai supprimé tout mes jobs à destination du nœud Poseidon
<br><br>

2. Une fois cela fait, retirez votre PVE des nœuds disponibles pour le ZFS dans `Datacenter` --> `Storage` --> cliquez sur votre stockage zfs puis retirez le de la liste
<img width="2115" height="644" alt="delete_zfs_node" src="https://github.com/user-attachments/assets/88063e8c-1de6-451c-8062-6815660e9309" />
<br>

3. Nous devons maintenant supprimer le pool ZFS du nœud dans son entièreté car celui-ci n'est constitué que du seul disque que nous allons réinitialiser

   a. Rendez-vous donc ensuite dans la gestion ZFS de votre nœud --> `Disks` --> `ZFS`
   
   b. Cliquez sur le pool ZFS --> Cliquez sur `More` en haut à droite puis supprimez votre pool en sélectionnant `destroy`
<img width="9920" height="3240" alt="zsf_disk_destroy" src="https://github.com/user-attachments/assets/33569325-b6c1-4c73-8df3-f340058b28e6" />

Vous pouvez ici choisir de réinitialiser le(s) disque(s) après le(s) avoir sortis du pool. On choisira de le faire ici afin de pouvoir les réutiliser pour autre chose directement après

4. Vérifiez ensuite que votre/vos disque(s) ne contiennent plus aucune trace de ZFS dans l'onglet `Disks` de votre PVE
<br>

**Vous avez terminé !**  

Pour résumer tout ce que nous avons fait :
- vous avez sortis un nœud Pve de votre architecture ZFS
- supprimer son pool ZFS
- nettoyer son/ses disque(s) afin de pouvoir les réutiliser pour autre chose
