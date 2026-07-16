# Introduction
Nous allons ici utiliser les outils SAN d'un NAS Synology DS223j. L'objectif est de créer un datastore sur un cluster PVE.  
Je parle ici de simulation, car les SAN d'entreprise habituels se basent sur des infrastructures bien plus conséquentes, avec des baies dédiées, des switchs et des liaisons fibre. 
Je n'ai rien de tout ça, je vais donc utiliser un boîtier Synology DS223j équipé de disques HDD, ainsi que sa liaison ethernet.

---

# Création rapide d'un espace de stockage

1. Rendez-vous dans l'application "SAN Manager"
2. Si c'est la première fois que vous vous rendez dans l'application, il vous sera proposé de créer votre premier stockage. Vous pouvez tout de même vous rendre dans l'onglet dédié si vous le souhaitez.
<img width="817" height="440" alt="SAN_manager_dashboard" src="https://github.com/user-attachments/assets/29a575a1-472e-490d-94df-c6f1eb8c0cac" />
  
> [!note]
> `LUN (Logical Unit Number)` : Unité logique de stockage. Celui qui se connecte au SAN le voit comme un disque physique.

3. Quoi qu'il en soit, commencez la création et adaptez les champs à votre convenance
<img width="479" height="453" alt="LUN-Test" src="https://github.com/user-attachments/assets/09f622a1-60d7-4235-86b4-c5216696cb40" />

- `Nom` : Nom de votre LUN  
- `Description` : Une simple description de celui-ci  
- `Emplacement` : Le volume sur lequel placer ce LUN. Dépends de votre contexte  
- `Capacité totale (Go)` : Capacité en Go du nouveau LUN  
- `Allocation d'espace` : Type de provisionnement du stockage (`Thick` pré-réserve la place et fournis les meilleures performances, `Thin` alloue dynamiquement en fonction de la place réellement utilisée)  

4. À l'étape suivante, il vous sera demandé de sélectionner une cible iSCSI. Dans mon cas, je n'ai que `Synology iSCSI Target`, car je n'en ai pas créé d'autres au préalable.
On partira ici du principe que c'est aussi votre première fois.
Passez donc cette étape
<img width="477" height="451" alt="iscsi-target_selection" src="https://github.com/user-attachments/assets/14b1f9b7-d219-4870-b57f-8ce56bbcf8df" />

> [!note]
> `Target` : Composant qui permet aux machines externes de se connecter au LUN. Sans lui, le LUN est innaccessible car non proposé aux machines exterieures

5. À cette étape, vous avez la possibilité de définir des autorisations d'accès pour certaines machines. Il faut savoir que les identifiants utilisés ne sont pas les IP, mais les `Initiator Name`.
Mais ici, ne vous embêtez pas : autorisez tout le monde   
<img width="478" height="451" alt="initiator_autorisation" src="https://github.com/user-attachments/assets/beb64b9c-658a-4d7c-a89c-fdd13515f558" />  

> [!note]
> `Initiator Name` : C'est la machine qui se connecte au SAN et qui consomme le stockage. Elles possèdent toutes un nom d'initiateur unique (IQN) du type : `iqn.1993-08.org.debian:01:abcdef`

6. Vous avez terminé, plus qu'à valider la synthèse. Attendez quelques secondes et vous verrez devant vous votre premier LUN

<img width="810" height="434" alt="LUN_summary" src="https://github.com/user-attachments/assets/9656f88c-38f4-4bfd-8b35-4634737ce7d9" />  

# La suite ?
En l'état des choses, vous pouvez déjà utiliser ce que nous venons de faire pour, par exemple, créer un datastore sur votre cluster Proxmox, comme je vous montre [ici](https://github.com/tadaron/Mon-Home-Lab/blob/main/Infra/Services/Proxmox/Proxmox_Virtual_Environment/Stockage/proxmox_iscsi_lun_datastore.md). 


Si vous vouliez aller plus loin dans le SAN, alors vous pourriez approfondir chacun de ses aspects :
- `LUN` 
  1. Étudier les cas d'utilisation du `Thick` ou du `Thin Provisionning`
  2. Regarder comment paramétrer des snapshots sur des LUN en `Thin provisionning`

- `Target`
  1. Créer une nouvelle cible pour un/des nouveau(x) LUN(s) afin de restreindre les accès à certaines machines à partir de leur `IQN`
  2. Ajouter une couche d'authentification avec `CHAP`
  3. Restreindre les accès à cette cible en passant uniquement par certaines interfaces réseau de votre NAS (si vous avez la chance d'en avoir plusieurs)
