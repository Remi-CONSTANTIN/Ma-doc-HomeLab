# Présentation rapide
Si vous connaissez déjà Fail2Ban, alors dites vous que CrowdSec et une version plus poussée de celui-ci. Le problème de Fail2Ban est qu'il fonctionne en standalone sur chaque machine, là où CrowdSec permet de mutualiser la liste des IPs bannies entre toutes vos machines et plus généralement avec toute la communauté.

---
# Installation

1. Installer le dépôt CrowdSec sur la machine à protéger
```
curl -s https://install.crowdsec.net | sudo sh
```

2. Installer CrowdSec
```
apt install crowdsec
```

3. Si vous aviez déjà des services sur la machine lors de l'installation de CrowdSec, alors il est possible que des collections aient automatiquement été téléchargées.
Pour vérifier cela : 
```
sudo cscli collections list
```
Exemple :
![[vmcours_crowdsec_installation_collection.png]]
(Oui il y avait beaucoup de chose)
ou simplement, sur une machine fraichement installée :
![[crowdsec_crowdsec_installation_collection.png]]
Les collections sont une composante que nous pouvons ajuster à nos besoins. Nous verrons cela dans la partie `Configuration`

4. Pour finir l'installation , je vous recommande vivement d'installer tout de suite le "Bouncer" iptables afin que CrowdSec puisse bannir automatiquement s'il détecte une activité suspecte
```
sudo apt install crowdsec-firewall-bouncer-iptables -y
```

---
# Configuration d'une machine standalone
Cette partie est consacrée aux paramétrages de base d'une instance CrowdSec sur une machine à protéger. Elle ne porte pas sur une installation multi-serveur gérée par un maître local (LAPI)

### Whitelist
[Source](https://docs.crowdsec.net/u/getting_started/post_installation/whitelists/)
Par défaut, toutes les IP de réseaux privés sont whitelistés, mais en fonction de votre contexte, cela n'est pas toujours appréciable. Dans mon cas, je préfère prévenir aussi prévenir les attaques venants de mes réseaux locaux, alors je vais être plus fin dans ma whitelist.

Pour ce faire, rendez-vous sur votre/vos machines où vous avez installé CrowdSec, et plus précisément dans le fichier dédié `/etc/crowdsec/parsers/s02-enrich/whitelists.yaml`.
(Chez Windows c'est `c:/programdata/crowdsec/config/parsers/s02-enrich/)

Par défaut, le fichier ressemble à cela :
![[crowdsec_linux_whitelists.yaml.png|417]]
Comme dit précédemment, cela ne me convient pas car cela ne protège pas en cas de mouvements latéraux dans un même réseau. 

Dans mon cas, je préfère whitelister seulement les IPs vraiment dignes de confiances. Cela peut par exemple, être votre serveur ansible ou votre PC Personnel...

Pour faire votre configuration, il n'est pas conseillé de toucher au fichier `whitelists.yaml` par défaut car celui-ci pourrait être écrasé lors d'un mise à jour.
Alors ce que nous allons faire c'est de le désactiver en le renommant puis d'en créer un nouveau.

1. Renommez le fichier par défaut
```
mv /etc/crowdsec/parsers/s02-enrich/whitelists.yaml /etc/crowdsec/parsers/s02-enrich/whitelists.yaml.desactivated
```
(.desactivated est juste un exemple, tout ce qui compte c'est que le fichier ne soit plus en .yaml)

2. Créez le nouveau fichier et customisez le comme-suit, par exemple
```
nano /etc/crowdsec/parsers/s02-enrich/custom-whitelist.yaml
```

3. Redémarrez ensuite le service pour appliquer les changements
```
sudo systemctl restart crowdsec.service
```
Pour savoir si la manipulation à fonctionnée. Essayez de vous munir d'une machine sous kali linux afin de tester le bruforce ssh (du type hydra)

Si vous le faites, voici la commande pour voir si une décision de ban à été prise par CrowdSec
```
sudo cscli decisions list
```
Exemple :
![[cscli decisions list.png]]
La durée de ban par défaut est de 4h (nous verrons comment la changer dans la partie `Profiles`)

Pour supprimer le bannissement après les tests
```
# Via l'IP
sudo cscli decisions delete --ip <IP>
# Via l'ID qu'on voit dans la commande `sudo cscli decisions list`
sudo cscli decisions delete --id <ID>
```

## Profiles
[Source](https://docs.crowdsec.net/u/getting_started/post_installation/profiles/)
Les profiles sont des listes de règles qui déterminent quelles actions CrowdSec va faire après avoir détecté des actions suspectes.

Vous pouvez retrouver la configuration par défaut dans le fichier `/etc/crowdsec/profiles.yaml`
(Pour Windows c'est `C:\ProgramData\CrowdSec\config\profiles.yaml`)
![[profiles.yaml.png]]
Comme vous pouvez le voir ,la décision par défaut est un ban de 4h sans plus d'actions.
Mais en vérité plus d'options s'offrent à nous.
Par exemple, les plus intéressantes (à mon sens) sont :
- L'augmentation du temps de ban en fonction du nombre total de ban
- La possibilité de recevoir une notification avec le ban
Voyons donc comment nous pouvons faire cela.

#### Scale automatique du temps de bannissement 
CrowdSec propose une ligne déjà toute faite dans le fichier de profil. Nous avons donc juste à la décommentée et le tour est joué !
```
duration_expr: Sprintf('%dh', (GetDecisionsCount(Alert.GetValue()) + 1) * 4)
```
**Explication :** 
A chaque fois qu'une IP va être bannie son compteur d'alerte va être incrémenté de 1.
Son temps de bannissement sera définie par `ce compteur x la durée de ban`.
Par exemple une IP s'ai fait bannir une première fois pendant 4h mais reviens à la charge une deuxième fois. Et bien cette fois il est bannie 2 x 4h = 8h. Pareil s'il revient une troisième fois, il sera banni 3 x 4h = 12h et ainsi de suite.

#### Notifications Discord
[Source](https://enchantedcode.co.uk/blog/crowdsec-discord-alerts/)

Bien qu'il y ai plusieurs façon d'être notifié lorsque CrowdSec fait une action, nous ne verrons ici que la notification discord qui nécessite un peu plus de manipulations que les autres.

Pour mettre en place les notification dans un salon discord, nous aurions pu passer par la console Web si nous avions eu un abonnement premium. Mais nous partons ici du principe que ce n'est pas le cas, nous allons donc bricoler nous même et utiliser l'API discord (webhook) !

**Prérequis :**
- Un serveur discord avec assez de droits pour gérer les intégrations

**Procédure :**
1. Commençons par créer le fichier de notification dans le répertoire dédié `/etc/crowdsec/notifications/discord.yaml`
Nous allons y mettre le contenu suivant :
```
type: http
name: discord
log_level: info
format: |
  {
    "username": "CrowdSec",
    "content": "{{range . -}}{{$alert := . -}}{{range .Decisions -}}:hammer: <@&id-role> {{$alert.MachineID}} - **[{{.Scenario | replace "crowdsecurity/" ""}}]** >
  }
url: <VOTRE-URL-WEBHOOK>
method: POST
headers:
  Content-Type: "application/json"

```

> [!warning]
> Veillez à remplacer `<VOTRE-URL-WEBHOOK>` et  `id-role` avec vos informations.

- Vous pouvez générer cette URL en créant un bot dans `Discord --> votre serveur --> Paramètres du serveur --> Intégration --> Webhooks --> Créer webhook`.
- Vous êtes libre d'enlever la partie `id-role` mais dans le cas ù vous le garderiez, alors allez dans `Discord --> votre serveur --> Paramètres du serveur --> Rôles` et cliquez sur les `...` à côté du rôle que vous voulez mentionner pour obtenir son ID

2. Retourner dans le fichier `/etc/crowdsec/profiles.yaml` et modifiez la partie `notifications` pour ne renseigner que discord
```
notifications:
  - discord
```

Si vous décidez de redémarrer le service `sudo systemctl restart crowdsec.service` pour appliquer la configuration et de tester le brutforce (par exemple), votre notification ressemblera à ça : 
(Le nom et l'image sont des paramètres que vous définissez lors de la création du bot)
![[discord_notification_id-machine.png]]
Comme vous pouvez le voir, vous avez l'ID de la machine et non pas un nom compréhensible comme son hostname.

Pour corriger cela nous allons désenregistrer notre machine du moteur CrowdSec local (LAPI) puis le réenregistrer en spécifiant le nom.

3. Prenez l'ID que vous avez du recevoir dans votre salon ou faite la commande suivante (login)
```
cat /etc/crowdsec/local_api_credentials.yaml
```

4. Faites ensuite la commande suivante pour supprimer votre machine de son propre moteur CrowdSec local (LAPI)
```
sudo cscli machines delete <ID-machine>
```

5. Réenregistrez ensuite votre machine mais en utilisant son hostname directement
```
sudo cscli lapi register --machine $(hostname)
```

6. Validez l'enrôlement de la machine dans le moteur local (même logique pour la console web)
```
sudo cscli machines validate $(hostname)
```

7. Redémarrez le service
```
systemctl restart crowdsec
```

8. Si vous arrivez à faire la commande `sudo cscli machines list` sans que cela vous retourne un message d'erreur alors vous avez terminé. Plus qu'à re-tester le brut-force.

Vous devriez avoir ce résultat :
![[discord_notification_hostname-machine.png]]
(On remarque aussi que le temps à augmenté grâce au scaling automatique que l'on a activé dans `/etc/crowdsec/profiles.yaml` )


---
# Enregistrer l'agent dans la console Web
Cette étape est optionnelle mais permet de retrouver les statistiques de vos CrowdSec installée dans un même Dashboard. Plutôt sympathique je trouve.

Pour ce faire :
1. Rendez vous dans l'onglet `Engines`  de votre [console CrowdSec](https://app.crowdsec.net/security-engines)
	Si c'est votre première connexion, il vous faudra alors créer un compte. Il vous sera ensuite proposé d'enregistrer votre premier agent. 

2. Défilez le tutoriel qui vous proposera d'installer l'agent comme nous l'avons fait au dessus, pas besoin de refaire les manipulations

3. Enfin, il vous sera proposé d'enregistrer votre machine dans la console. Exécutez la commande affichée, elle contient le token spécifique à votre console
```
sudo cscli console enroll <votre-token>
```

4. Plus qu'à approuver l'enrôlement dans la console Web, pensez à renommer vos machines sinon la gestion sera compliquée quand vous en aurez plusieurs


---

# Configuration multi-serveur (1 maître pour X clients)
Cette partie porte sur le cas où vous avez plusieurs machines à gérer avec crowdSec.
Ce n'est que dans ce cas de figure que vous pourrez profitez de la liste commune de bannissement d'IP entre toute vos machines.

Une installation par défaut de CrowdSec installe un serveur indépendant où il faut faire toutes les manipulations vues au dessus alors qu'une installation maître-clients permet de centraliser certaines configurations sur le serveur.

> [!warning]
> La liste commune de ban entre vos VMs n'est pas la même chose que les blocklist fournis par CrowdSec !

### Prérequis
- Un serveur linux/Windows maître 

### Procédure

#### Configuration du serveur maître
La configuration du serveur maître est exactement la même que si vous configuriez une machine en standalone, à la seule différence qu'il faudra ouvrir l'API de la LAPI du serveur à vos machines clientes.
Donc commencer par suivre le tutoriel `Configuration d'une machine standalone` puis reprenez ici à l'étape ci-dessous !

##### Configuration machine cliente (agent)
Commençons par préparer le serveur où CrowdSec servira de maître en modifiant son fichier de configuration général `/etc/crowdsec/config.yaml`

1. Changer le périmètre d'écoute de l'API afin que la LAPI accepte les communications issues de tout les réseaux (vous pouvez bien entendu filtrer plus)
```
api:
  client:
    insecure_skip_verify: false
    credentials_path: /etc/crowdsec/local_api_credentials.yaml
  server:
    log_level: info
    listen_uri: 0.0.0.0:8080  # <--- C'est cette ligne qui change
    profiles_path: /etc/crowdsec/profiles.yaml
```
N'oubliez pas d'ouvrir le port 8080 sur votre pares-feux local et/ou réseau

2. Redémarrez ensuite le service
```
sudo systemctl restart crowdsec
```

Passons maintenant à la configuration côté client

3. Enregistrer votre machine cliente sur le LAPI de votre serveur maître
```
sudo cscli lapi register --url http://<IP_DU_MASTER>:8080 --machine <hostname>
```
ou encore `sudo cscli lapi register --url http://<IP_DU_MASTER>:8080 --machine $(hostname)` si on veut utiliser le hostname de la machine cliente

4. Rendez-vous ensuite dans les paramètres de la machine cliente pour désactiver la LAPI inutile car c'est le serveur maître qui va s'occuper de réfléchir `/etc/crowdsec/config.yaml`
```
api:
  client:
    insecure_skip_verify: false
    credentials_path: /etc/crowdsec/local_api_credentials.yaml
  server:
    enable: false  # <--- On désactive le cerveau local
    log_level: info
```

5. Redémarrez ensuite le service
```
sudo systemctl restart crowdsec
```

6. Pensez ensuite à vous rendre sur le maître pour accepter l'enregistrement du client dans la LAPI

Pour afficher les machines (dont celles en attente de validation)
```
sudo cscli machines list
```

Pour valider
```
sudo cscli machines validate <nom_du_client>
```


##### Configuration machine cliente "Bouncer"
La configuration de l'agent seul ne permet pas de synchroniser la liste des bannissements, il faut aussi configurer le `bouncer` qui est un composant à part.

1. Rendez-vous sur le maître et créer un token pour votre client
```
sudo cscli bouncers add <nom_du_client>
```

2. Allez sur la machine cliente et modifiez le fichier `/etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml`pour y ajouter l'adresse du serveur maître et le token de connexion
```
mode: iptables
update_frequency: 10s
log_mode: file
log_dir: /var/log/
log_level: info
log_compression: true
log_max_size: 100
log_max_backups: 3
log_max_age: 30
api_url: http://<IP_maître>:8080/
api_key: <votre-token>
```

3. Comme d'habitude, redémarrez le service pour appliquer les changements
```
sudo systemctl restart crowdsec-firewall-bouncer
```

4. Vous pouvez maintenant tester d'ajouter une décision manuellement sur votre client pour voir si elle remonte sur le maître

Décision manuelle sur client :
```
sudo cscli decisions add --ip 8.8.4.4 --duration 1h --reason "Test de synchronisation"
```

Vérifier sur le maître :
```
sudo cscli decisions list
```


# Commandes utiles

- Lister les machines enregistrées dans la LAPI
```
cscli machine list
```
- Supprimer une machine de la LAPI
```
cscli machine delete <NOM-OU-ID-MACHINE>
```
- Lister les bouncers enregistrés
```
sudo cscli bouncers list
```
- Supprimer un bouncer
- ```
  sudo cscli bouncers delete <NOM-Bouncer>
  ```
