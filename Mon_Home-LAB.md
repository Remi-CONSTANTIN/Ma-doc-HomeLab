# Introduction
Vous trouverez ici une présentation de mon Home-Lab.
Chaque grande thématique va être abordée dans sa propre partie.

# Diagrammes isométriques
## Vue physique du Home-lab
<img width="753" alt="home-lab_vue_physique" src="https://github.com/user-attachments/assets/01c680ac-9a93-4e34-891c-ef69205c6bbe" />

## Vue logique des réseaux
<img width="753" alt="Réseaux_vue_logique" src="https://github.com/user-attachments/assets/48d741b4-ee9e-40ae-ad0d-217b094e78f0" />


# Ma vision des choses
Je choisis les outils que je mets dans mon Home-Lab en me basant sur ces critères :  

- **Gratuit**  
Étant étudiant, il m'est toujours plus simple d'accéder à une technologie quand celle-ci ne pose pas de contrainte financière.  
Et a minima avec une Community Edition et dans l'idéal open-source.

- **Simple d'exploitation, d'administration et de réparation**  
Parce qu'un bon outil est un outil qu'on aime manipuler et qu'on ne remet pas à plus tard

- **Intéressant**  
Logique, parce qu'apprendre doit rester un plaisir

- **Utile**  
Pas besoin de m'expliquer...

- **Léger**  
Ou du moins pas une usine à gaz, pour économiser des ressources sur mon cluster Proxmox (c'est pour cela que je n'ai que très peu de Windows)

# Mes outils triés par thématique
## Calcule
J'utilise **Proxmox** comme hyperviseur car gratuit, intéressant et puissant. Je possède actuellement 3 nœuds : 1 nœud Prod (64Go RAM), 1 nœud DEV (64Go RAM) et 1 autre nœud DEV (32Go RAM).  
La plupart des machines virtuelles fonctionnent sous **Debian**, le tout saupoudré d'un peu de LXC (principalement Debian aussi) et accompagné de rares VMs Windows Server  
Voir détails dans [A compléter]()

## Stockage
J'ai choisi l'environnement Synology pour mon stockage car il est éprouvé et documenté. Ce n'est pas la meilleure option pour l'apprentissage mais il me fallait quelque chose de fiable.  
Ayant des contraintes financières j'ai opté pour un modèle d'entrée de gamme DS223j (4 cœurs 1 Go RAM). Le fait qu'il soit faiblement fourni en RAM le cantonne à un usage de stockage uniquement.

## Sauvegarde
Mes sauvegardes sont orchestrées par PVE mais stockées sur un datastore PBS afin de profiter notamment de la déduplication des données

## Réseau
J'ai choisi de placer ma Freebox en bridge avec un Dream Router 7 pour avoir tout l'environnement Ubiquiti, son interface, ses outils et sa sécurité.  
J'accède à mon Home-Lab via un WireGuard fourni par le routeur.

## Sécurité
Chacune des machines Linux possède une suite d'outils de sécurité :
- `CrowdSec` pour détecter les comportements suspects et ainsi bannir en conséquence (fonctionne aussi dans mon réseau local)
- `UFW` pour fermer le maximum de portes sur mes Linux ou Proxmox Firewall si la machine héberge des logiciels contournant les pare-feu locaux (Docker par exemple)
- `Wazuh` pour l'analyse de vulnérabilités (que je n'exploite pas encore à son plein potentiel)
- Un agent `Teleport` pour me simplifier l'accès aux machines

## Automatisation
La plupart des tâches automatiques sont gérées via mon AWX afin d'éviter au maximum de devoir gérer les crons sur chacune de mes machines. Il me sert aussi pour appliquer une configuration sur toue ou partie de mes serveurs 
Je possède tout de même un Ansible sur une Raspberry Pi 4B allumé 24/24h pour l'automatisation de l'extinction / mise en route de mon cluster Proxmox
Voir détails dans [A compléter]()

## Cloud privé
Mes services de compute et de stockage me permettent d'autohéberger un serveur Nextcloud me permettant de me séparer de mes abonnements chez des fournisseurs externes (Onedrive, Google Photo).  
J'en fais d'ailleurs profiter un proche et peut être bientôt plus ?

## Supervision
Et pour avoir une visibilité et des alertes sur d’éventuels problèmes dans mon "parc", je supervise le tout via Checkmk.  
Pourquoi ? Tout simplement parce que je l'utilise en milieu professionnel, c'est donc une bonne opportunité d'apprentissage.   
Voir détails dans [A compléter]()

# Divertissement
Un des premiers services que j'ai commencé à auto-héberger a été FreshRSS pour faire ma veille informatique. C'est un outil simple qui fait très bien ce qu'on lui demande. Je suis tout de même à la recherche d'une autre solution qui pourrait me faire des résumés de l'actualité. Je suppose une solution basée sur IA ?  
J'ai aussi une instance JellyFin pour pouvoir regarder des films sur mes différents supports sans avoir à déplacer un disque dur. On mentionnera le fait que c'est le NAS qui héberge les films afin de ne pas surcharger les SSD de mes nœuds PVE
