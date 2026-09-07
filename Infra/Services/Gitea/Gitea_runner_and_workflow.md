# Guide de mise en place d'un runner sur Gitea
Ce guide vise à vous montrer comment mettre en place un runner afin de pouvoir profiter du système de pipeline de Gitea.  
J'ai pour objectif de mettre en place un environnement de build d'images OCI à partir d'une instance de Gitea déjà existante.

Si vous connaissez déjà le concept et les outils de CI/CD, alors vous saurez manipuler Gitea Actions car celui ci est basé sur Github Actions


# Mise en place

## Prérequis
Assurez vous d'avoir un dépôt (ou d'en créer un) pour pouvoir tester un "workflow" (pipeline).  
Je vais personnellement manipuler dans mon dépôt `awx-ee` afin de pouvoir builder mes conteneurs d’exécution d'ansible pour mon AWX.

---

## Création du token Gitea pour le runner
La première véritable étape consiste à aller chercher le token d'enregistrement pour que notre runner soit associé à notre Gitea

1. Pour ce faire, connectez vous à votre gitea avec l'utilisateur qui a les droits sur le dépôt

2. Pour créer le token, cliquez sur votre icône en haut à droite, ensuite sur `configuration` puis sur `Actions` et pour finir `Exécuteurs`

3. Une fois dans le bon onglet, cliquez sur `Créer un nouvel exécuteur` et copiez simplement le `Registration Token` affiché.  
Mettez le de côté, nous allons en avoir besoin dans l'étape suivante

---

## Installation du runner

1. Créez vous un répertoire de travail et placez vous dedans
```
mkdir gitea-runner && cd gitea-runner
```

2. Commencez par créer un espace de nom pour le runner
```
kubectl create namespace gitea-runner
```

3. Créez le secret qui contiendra le token Gitea du runner en veillant à bien remplacer <votre-token> par ce que vous avez trouvé dans Gitea
```
kubectl create secret generic runner-secret --from-literal=token="<votre-token>" -n gitea-runner
```

4. Créer le fichier qui contiendra les paramètres de notre runner

```
nano runner.yml
```

Mettez-y : 
```
# 1. Le disque dur pour le cache Docker (5 Go)
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: docker-vol
  namespace: gitea-runner
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
# 2. Le disque dur pour les données du Runner (1 Go)
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: runner-vol
  namespace: gitea-runner
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
# 3. Le Pod contenant le Runner et DinD
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: runner
  name: runner
  namespace: gitea-runner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: runner
  template:
    metadata:
      labels:
        app: runner
    spec:
      restartPolicy: Always
      volumes:
      - name: docker-socket
        emptyDir: {}
      - name: runner-data
        persistentVolumeClaim:
          claimName: runner-vol
      - name: docker-data
        persistentVolumeClaim:
          claimName: docker-vol
      
      # Le conteneur 1 : Le moteur Docker (DinD)
      initContainers:
      - name: docker
        image: docker:dind
        securityContext:
          privileged: true
        volumeMounts:
        - name: docker-socket
          mountPath: /var/run
        - name: docker-data
          mountPath: /var/lib/docker
        restartPolicy: Always
      
      # Le conteneur 2 : Le Runner Gitea
      containers:
      - name: runner
        image: gitea/act_runner:latest
        env:
        - name: GITEA_INSTANCE_URL
          value: "http://<URL-interne-gitea:port>"      # A remplacer
        - name: GITEA_RUNNER_REGISTRATION_TOKEN
          valueFrom:
            secretKeyRef:
              name: runner-secret
              key: token
        volumeMounts:
        - name: runner-data
          mountPath: /data
        - name: docker-socket
          mountPath: /var/run
```

**Comment trouver `<URL-interne-gitea:port>` ?**  
En regardant le résultat de la commande `kubectl get svc -n gitea`.  

Je peux en déduire dans mon cas : `http://gitea-http.gitea.svc.cluster.local:3000`
<img width="1357" height="260" alt="gitea-http" src="https://github.com/user-attachments/assets/47858b49-d68a-4c4f-a6cb-2e520ff328ba" />

5. Lancez l'installation du runner
```
kubectl apply -f runner.yml
```
L'installation va prendre quelques secondes puis vérifier qui passe bien dans l'état `running`
```
kubectl get pods -n gitea-runner
```

6. Si vous retournez dans l'onglet des exécuteurs (Voir `Création du token Gitea pour le runner`), vous deviez voir votre runner
<img width="1048" height="200" alt="gitea-runner" src="https://github.com/user-attachments/assets/4b93b692-4d0f-4fe0-87e2-2d59754e4510" />

---

## Test d'un workflow
Maintenant que vous avez un runner prêt à l'emploi, plus qu'à créer un workflow pour le tester

1. Retournez dans l'interface web de gitea et allez dans un de vos dépôt

2. Créez y un nouveau fichier que vous appellerez exactement `.gitea/workflows/test.yaml` (cela va créer l'arborescence de dossiers automatiquement)

3. Ajoutez y le contenu suivant : 
```
name: Mon Premier Pipeline

# 1. Quand doit-il s'exécuter ? (Ici, à chaque "push" sur le code)
on: [push]

# 2. Que doit-il faire ?
jobs:
  test-basique:
    # On demande explicitement l'environnement déclaré par notre Runner
    runs-on: ubuntu-latest
    
    # 3. La liste des tâches à accomplir, dans l'ordre
    steps:
      - name: Message de victoire
        run: echo "🚀 Félicitations, votre Runner Gitea fonctionne parfaitement !"
        
      - name: Vérifier l'environnement de la machine virtuelle
        run: |
          echo "Voici les informations sur le système jetable créé par DinD :"
          cat /etc/os-release
```

**Détails :**
- `on: [push]` : Déclenche automatiquement le workflow à chaque nouvelle modification dans le dépôt
- `jobs` : Décrit les tâches exécutées. Ici, c'est juste un simple affichage d'un message bidon et des informations sur la distribution du système

4. Plus qu'à envoyer ce nouveau fichier et cela devrait déclencher notre workflow dès maintenant !

5. Pour voir la tâche en action et le résultat, allez dans l'onglet `Actions` de votre dépôt

</br><br/>
**Et voilà c'est tout pour la mise en place et le test d'un runner sur Gitea !**
