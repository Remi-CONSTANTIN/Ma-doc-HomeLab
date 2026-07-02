# Introduction
Vous trouverez ici une présentation de mon Home-Lab.
Chaque grande thématiques va être abordée dans sa propre partie.

# Ma vision des choses
Je choisis les outils que je mets dans mon home-lab en me basant sur ces critères :  

- **Gratuit**  
Etant étudiant il m'est toujours plus simple d'accéder à une technologie quand celle-ci ne pose pas de contrainte financière
Et à minima avec une Community Edition et dans l'idéal open-source

- **Simple d'exploitation, d'administration et de réparation**  
Parce qu'un bon outil est un outil qu'on aime manipuler et qu'on ne remet pas à plus tard parce que c'est un enfer à debugger

- **Intéressant**  
Logique, parce qu'apprendre doit rester un plaisir

- **Utile**  
Pas besoin de m'expliquer...

- **Léger**  
Ou du moins pas une usine à gaz pour économiser des ressources sur mon cluster Proxmox (c'est pour cela que je n'ai que très peu de Windows)

# Mes outils triés par thématique
## Calcule
J'utilise **Promox** comme hyperviseur car gratuit, intéressant et puissant. Je possède actuellement 3 nœuds : 1 nœud Prod (64Go RAM), 1 nœud DEV (64Go RAM) et 1 autre nœud DEV (32Go RAM).  
La plupart des machines virtuelles fonctionnent sous **Debian** saupoudré d'un peu de LXC (principalement Debian aussi) et accompagné de rares VMs Windows Server

## Stockage
J'ai choisi l'environnement Synology pour mon stockage car éprouvé, documenté. Ce n'est pas la meilleure option pour l'apprentissage mais il me fallait quelque chose de fiable.  
Ayant des contraintes financières j'ai opté pour un modèle entrée de gamme DS223j (4 cœurs 1 Go RAM). Le fait qu'il soit faiblement fourni en RAM le cantonne à un usage de stockage uniquement.

## Sauvegarde
Mes sauvegardes sont orchestrées par PVE mais stockées sur un datastore PBS afin de profiter notamment de la déduplication des données

## Réseau
J'ai choisi de placer ma Freebox en bridge avec un Dream Router pour avoir tout l'environnement Ubiquiti, son interface, ses outils et sa sécurité.  
J'accède à mon Home-Lab via un WireGuard fourni par le routeur.

## Sécurité
Chacune des machines Linux possède une suite d'outils de sécurité :
- CrowdSec pour détecter les comportements suspects et ainsi bannir en conséquence (fonctionne aussi dans mon réseau local)
- UFW pour fermer le maximum de portes sur mes Linux ou Proxmox Firewall si la machine héberge des logiciels contournant les pare-feu locaux (Docker par exemple)
- Wazuh pour l'analyse de vulnérabilité (que je n'exploite pas encore à son plein potentiel)
- Un agent Teleport pour me simplifier l'accès aux machines

## Automatisation
La plupart des tâches automatiques sont gérées via mon AWX afin d'éviter au maximum de devoir gérer les cron sur chacune de mes machines. Il me sert aussi pour appliquer une configuration sur toue ou partie de mes serveurs 
 
Je possède tout de même un Ansible sur une Raspberry Pi 4B allumé 24/24h pour l'automatisation de l'extinction/mise en route de mon cluster Proxmox

## Cloud privé
Mes services de compute et de stockage me permettent d'autoheberger un serveur Nextcloud ce qui m'a permis de me séparer de mon abonnement Onedrive.  
J'en fais d'ailleurs profiter mes proches

## Supervision
Et pour avoir une visibilité et des alertes sur d'eventuels problèmes dans mon "parc", je supervise le tout via Checkmk.  
Pourquoi ? Tout simplement parce que je l'utilise en milieu professionel, c'est donc une bonne opportunité d'apprentissage 
