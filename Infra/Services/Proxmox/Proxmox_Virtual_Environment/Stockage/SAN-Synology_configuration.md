# Introduction
Nous allons ici utiliser un NAS Synology pour simuler un SAN afin de créer un datastore sur le cluster PVE. L'objectif est d'utiliser le protocole iSCSI.  
Je parle ici de simulation car les SAN d'entreprises habituels se bases sur des infrastructures bien plus conséquentes avec baies dediées, switchs et liaisons fibres. 
Je n'ai rien de tout ça, je vais donc utiliser un boitier Synology DS223j ainsi que sa liaison ethernet.

---

# Création d'un LUN

1. Rendez-vous sur l'application "SAN Manager"
2. Si c'est la première fois que vous vous rendez dans l'application,il vous sera proposé de créer votre premier stockage. Vous pouvez tout de même vous rendre dans l'onglet dedié si vous le désirez
<img width="1168" height="629" alt="SAN_manager_dashboard" src="https://github.com/user-attachments/assets/29a575a1-472e-490d-94df-c6f1eb8c0cac" />
  
> [!note]
> Point définition `LUN` : Insérer définition

3. Quoi qu'il en soit commencez la création et adaptez les champs à votre convenance
<img width="685" height="648" alt="LUN-Test" src="https://github.com/user-attachments/assets/09f622a1-60d7-4235-86b4-c5216696cb40" />

- `Nom` : Nom de votre LUN  
- `Description` : Une simple déscription de celui-ci  
- `Emplacement` : Le volume sur lequel placer ce LUN. Dépends de votre contexte  
- `Capacité totale (Go)` : Capacité en Go du nouveau LUN  
- `Allocation d'espace` : Type de provisionnement du stockage (`Thick` pré-reserve la place et fournis les meilleures performances, `Thin` alloue dynamiquement en fonction de la place rééllement utilisée)  

4. A l'étape suivante, il vous sera demander de selectionner une cible iSCSI. Dans mon cas je n'ai que `Synology iSCSI Target` car je n'en ai pas créé d'autres au préalable.
On partira ici du principe que c'est aussi votre première fois.
Passez donc cette étape
<img width="682" height="644" alt="iscsi-target_selection" src="https://github.com/user-attachments/assets/14b1f9b7-d219-4870-b57f-8ce56bbcf8df" />

5. A cette étape vous avez la possibilité de définir des autorisations d'accès pour certaines machines. Il faut savoir que les identifiants utilisés ne sont pas les IPs mais les `Initiator Name`.
Mais ici ne vous embêtez pas, autorisez pour tout le monde   
<img width="684" height="645" alt="initiator_autorisation" src="https://github.com/user-attachments/assets/beb64b9c-658a-4d7c-a89c-fdd13515f558" />  

> [!note]
> Point définition `Initiator Name` : Insérer définition

6. Vous avez terminé, plus qu'à valider la synthèse. Attendez quelque secondes et vous verrez devant vous votre premier LUN

<img width="1158" height="620" alt="LUN_summary" src="https://github.com/user-attachments/assets/9656f88c-38f4-4bfd-8b35-4634737ce7d9" />  
