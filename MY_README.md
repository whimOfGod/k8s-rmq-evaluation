### [ KS8-RMQ-EVALUATION ] 


Après avoir cloner le projet en locale, j'ai supprimé le dossier .GIT pour pouvoir refaire un : 
    [  git init  ]

par la suite j'ai commit et push sur mon repository github



Nous entrons désormais dans le vif du sujet,

### [ EXAMEN ] 

1) Récupéreration du fichier kubeconfig.yml puis exportation de cette dernière

```bash
        $env:KUBECONFIG="D:\CFA-INSTA-COURS\CODE\DEVOPSCRV\k8s-rmq-evaluation\kubeconfig.yml"

         # puis on vérifie si les données sont biens accessibles avec un :
        kubectl get nodes
```



2) Création d'un namespace Pour éviter les collisions avec mes collègues étudiants, il est essentiel de créer un namespace spécifique.

```bash
        kubectl create namespace aaronkolins
```
3) Nous construisons l'image de notre backend 

```bash
       cd k8s-rmq-evaluation/database
       docker build -t aaronkolins77/backend:latest .

        # puis nous faisons un push
        docker push aaronkolins77/backend:latest
```

4) Nous passons ensuite au Déploiement des Services sur Kubernetes
    
### Déploiement de RabbitMQ 

pour ce faire nous allons créer notre fichier YAML :  rabbitmq-cluster.yml

```sh
        # on applique
       kubectl apply -f rabbitmq-cluster.yml

```

### Déployer PostgreSQL

pour ce faire nous allons créer notre fichier YAML :  postgres-deployment.yml   

```sh
kubectl apply -f postgres-deployment.yml
```



