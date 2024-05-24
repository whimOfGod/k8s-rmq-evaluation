### [ KS8-RMQ-EVALUATION ] 


Après avoir cloner le projet en locale, j'ai supprimé le dossier .GIT caché pour pouvoir refaire un : 
    [  git init  ]

par la suite j'ai commit et push sur mon repository github



Nous entrons désormais dans le vif du sujet,

- un dossier : image est à disposition pour les illustrations et les preuves de test

### [ EXAMEN ] 

### Contextualisation

Ce projet nous amène à mettre en œuvre une infrastructure complète en utilisant Docker et Kubernetes, incluant la création et déploiement de conteneurs pour un backend Node.js et une base de données PostgreSQL, ainsi que la configuration et l'utilisation d'un cluster RabbitMQ

### A. connexion au serveur kubernetes à distance 

1) Récupéreration du fichier kubeconfig.yml puis exportation de cette dernière 
   nous allons par la suite nous connecter au serveur kubernetes à distance 

```sh
$env:KUBECONFIG="D:\CFA-INSTA-COURS\CODE\DEVOPSCRV\k8s-rmq-evaluation\kubeconfig.yml"
```

2) puis on vérifie si on reçoit bien les données/packets de la part de notre serveur, si notre serveur distant est accessibles avec un :

```sh
        kubectl get nodes
```


### B. Creation d'un namespace spécifique

1) Création d'un namespace Pour éviter les collisions avec mes collègues étudiants, il est essentiel de créer un namespace spécifique.

```sh
        kubectl create namespace aaronkolins
```

2) Puis je vérifie que le namespace existe bel et bien



### C. Dockerfile 

1) Nous préparons nos dockefiles : du backend et de database

2) constructions des images :

- Création de Dockerfiles pour le backend Node.js et la base de données PostgreSQL.


- nous construisons l'image de notre backend et pushons cette images avec les commandes suivantes

```sh
# on se dirige dans le bon répertoire
    cd k8s-rmq-evaluation/backend


# nous construisons l'image de notre docker avec un tag
    docker build -t aaronkolins77/backend:v1 .

# puis nous faisons un push
    docker push aaronkolins77/backend:v1

```


- nous construisons l'image de notre postgres(database) et pushons cette images avec les commandes suivantes

```sh
# on se dirige dans le bon répertoire
    cd ..
    cd k8s-rmq-evaluation/database


# nous construisons l'image de notre docker avec un tag
    docker build -t aaronkolins77/postgres:v1 .

# puis nous faisons un push
    docker push aaronkolins77/postgres:v1

```

### D. Configuration Kubernetes et déploiements

 Nous procédons à la configuration et au Déploiement des Services sur Kubernetes
    
### Déploiement de RabbitMQ 

pour ce faire nous avons créé notre fichier YAML :  rabbitmq.yml

- Nous Déployons un cluster RabbitMQ sur Kubernetes.

- Configuration de RabbitMQ pour l'utilisation par l'application backend.

```sh
        # on applique
       kubectl apply -f rabbitmq.yml

```

### Déploiement de PostgreSQL

- pour ce faire nous allons créer notre fichier YAML :  postgres-deployment.yml

- nous vérifions bien le nom de nos métadata: 
    #### postgres-deployment
- surtout le namespace 
    #### aaronkolins
- au niveau de l'image de notre conteneurs on vérifie bien à rajouté le login de notre docker en locale sinon nous aurons des problèmes d'accès au de droits
    ### image: aaronkolins77/postgres:v1
- puis le port etc
- ainsi que la configuration de notre service


- commande de déploiement :
```sh
# On se dirige dans le bon chemin sinon cela ne fonctionne pas 

    kubectl apply -f postgres-deployment.yml
```

### Déploiement du backend

- C'est à peu près le même processus

```sh
    kubectl apply -f backend-deployment.yml
```

### D. Débogage et Résolution des Problèmes :

- Pour ^tre sûre et avancé dans nos implémentations , nous vérifions quand mêmes nos pods et nos services :


```sh
    kubectl get pods --namespace=aaronkolins

    kubectl get services --namespace=aaronkolins

    kubectl get deployments --namespace=aaronkolins
```

- On vérifie le statut de nos services s'il y'a une erreur on doit pouvoir consulter les logs et corriger cette erreur, cela peut être dû à une saturation de port, à l'inaccessibilité du serveur distant, à une mauvaise adresse que nous avons exposé

- si tout nos pods sont opérationnelles 1/1 nous pouvons évoluer

- on peut également consulter tous nos évènement avec  :

```sh
    kubectl get events --namespace=aaronkolins
```

- ou une erreur spécifique sur l'un de nos dépoiements avec cette commande, par exemple : 

```sh

kubectl logs backend-deployment-6cdc8554c5-vflgq --namespace=aaronkolins 
```

### E. Ajout de l'autoscaler

1) Partie Test avec le Compteur

- c'est une partie importante et consiste à tester la communication et l'intégration des différents services déployés sur Kubernetes. Le test principal est réalisé à travers un script Node.js appelé count.js, qui sert de démonstration de l'utilisation et de l'intégration de RabbitMQ avec le backend et la base de données PostgreSQL

### Communication entre Services :

- Nous Vérifions à ce niveau que le backend peut se connecter à RabbitMQ et envoyer des messages à une file d'attente spécifique.


2) Création d'un compteur

```sh

kubectl apply -f backend-autoscaler.yml

```
- On s'assure que RabbitMQ est correctement configuré pour recevoir et gérer les messages provenant du backend.

```sh
curl localhost:4040/count/create -X POST
```

3) puis nous Configurerons les variables d'environnement pour count.js et nous exécutons ce script dans un terminal bash pour éviter les erreurs sur pwshl :

```bash

    export RABBITMQ_URL=amqp://rabbitmq:5672
    export QUEUE=count
    export COUNTER=<UUID_FROM_PREVIOUS_STEP>
    node count.js

```

- Nous Évaluons par par suite la capacité du système à gérer des charges de travail variables en utilisant un : 
#### Horizontal Pod Autoscaler 
pour le backend.

- nous Observons le comportement de l'application sous différentes conditions de charge.

### Les résultats attendus

- Le server kubernetes distant a été fermé malheureseusemnt ahah je n'ai pas pu tout tester

- cependant 
Les logs du pod backend devraient afficher des messages indiquant qu'il tente de se connecter à RabbitMQ.
Une fois connecté, des messages "send" devraient apparaître périodiquement, indiquant que des messages sont envoyés à la file d'attente.

Messages dans RabbitMQ :

on doit être en mesure de voir ces messages dans RabbitMQ via son interface de gestion (accessible via le port configuré).
Les messages devraient contenir la valeur définie par COUNTER.

- j'ai également oublié de spcifier que pour lancer notre application dans notre terminale nous exécutons : 

```bash

    yarn start 

```
on pourra consulter ainsi voir si notre application envoie bien les requ^tes et si par exemple on arrive pas à accéder au postgres nous aurions une trace de cette erreur dans nos logs 

- également pour les tests il ne faut pas hésiter à supprimer ses images docker, deploiements et pods et refaire toute les commandes pour s'assurer que tout est ok, 

- Il est vrai que cela peut être rédibitoire raison pour laquelle il est préférable de créer un test à exécuter à chaque pour tous les builds, déploiments et suppressions


### Merci pour cet Examen superiche en technologie 








