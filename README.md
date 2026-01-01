# TP-33 - Déploiement d'une application Spring Boot sur Kubernetes

## Description
Ce projet est une application Spring Boot REST API simple qui démontre le déploiement d'une application sur Kubernetes avec Docker et Minikube.

## Pré-requis
- Java 17 ou supérieur
- Maven
- Docker
- Minikube (ou autre cluster Kubernetes local)
- kubectl

## Structure du Projet
```
TP33/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/
│       │       └── example/
│       │           └── demok8s/
│       │               ├── DemoK8sApplication.java
│       │               └── api/
│       │                   └── HelloController.java
│       └── resources/
│           └── application.properties
├── Dockerfile
├── k8s-deployment.yaml
├── k8s-service.yaml
├── k8s-configmap.yaml
└── pom.xml
```

## Étapes de Déploiement

### 1. Test Local (Optionnel)
```bash
mvn spring-boot:run
```
Tester avec:
```bash
curl http://localhost:8080/api/hello
```

### 2. Construction du JAR
```bash
mvn clean package -DskipTests
```

### 3. Construction de l'Image Docker

#### Option A: Docker local
```bash
docker build -t demo-k8s:1.0.0 .
```

Test local de l'image:
```bash
docker run -p 8080:8080 demo-k8s:1.0.0
curl http://localhost:8080/api/hello
```

#### Option B: Avec Minikube (Recommandé)
```bash
# Démarrer Minikube
minikube start

# Utiliser le Docker de Minikube
eval $(minikube docker-env)

# Construire l'image dans l'environnement Minikube
docker build -t demo-k8s:1.0.0 .
```

### 4. Créer le Namespace
```bash
kubectl create namespace lab-k8s
kubectl get namespaces
```

### 5. Déployer sur Kubernetes

#### Appliquer la ConfigMap
```bash
kubectl apply -f k8s-configmap.yaml
```

#### Déployer l'application
```bash
kubectl apply -f k8s-deployment.yaml
```

#### Créer le Service
```bash
kubectl apply -f k8s-service.yaml
```

### 6. Vérification du Déploiement

#### Vérifier les pods
```bash
kubectl get pods -n lab-k8s
kubectl describe deployment demo-k8s-deployment -n lab-k8s
```

#### Vérifier le service
```bash
kubectl get svc -n lab-k8s
```

#### Consulter les logs
```bash
# Récupérer le nom du pod
kubectl get pods -n lab-k8s

# Afficher les logs
kubectl logs <nom-du-pod> -n lab-k8s
```

### 7. Tester l'API

#### Récupérer l'IP de Minikube
```bash
minikube ip
```

#### Appeler l'endpoint
```bash
curl http://<minikube-ip>:30080/api/hello
```

Exemple avec IP 192.168.49.2:
```bash
curl http://192.168.49.2:30080/api/hello
```

Réponse attendue:
```json
{
  "message": "Hello from ConfigMap in Kubernetes",
  "status": "OK"
}
```

### 8. Test Inside Cluster (Optionnel)
```bash
kubectl run curl-pod -n lab-k8s --image=alpine/curl -it -- sh
# Dans le pod:
curl http://demo-k8s-service:8080/api/hello
exit
```

### 9. Commandes Utiles de Diagnostic

```bash
# Voir tous les pods
kubectl get pods -n lab-k8s

# Détails d'un pod
kubectl describe pod <nom-pod> -n lab-k8s

# Logs en temps réel
kubectl logs -f <nom-pod> -n lab-k8s

# Exécuter une commande dans un pod
kubectl exec -it <nom-pod> -n lab-k8s -- sh

# Vérifier les événements
kubectl get events -n lab-k8s --sort-by='.lastTimestamp'

# Vérifier la ConfigMap
kubectl get configmap -n lab-k8s
kubectl describe configmap demo-k8s-config -n lab-k8s
```

### 10. Modification de la ConfigMap

Pour modifier le message sans redéployer:

```bash
# Éditer la ConfigMap
kubectl edit configmap demo-k8s-config -n lab-k8s

# Redémarrer les pods pour prendre en compte les changements
kubectl rollout restart deployment demo-k8s-deployment -n lab-k8s

# Suivre le statut du rollout
kubectl rollout status deployment demo-k8s-deployment -n lab-k8s
```

### 11. Scaling

```bash
# Augmenter le nombre de réplicas
kubectl scale deployment demo-k8s-deployment --replicas=3 -n lab-k8s

# Vérifier
kubectl get pods -n lab-k8s
```

## Nettoyage

Pour supprimer toutes les ressources:

```bash
kubectl delete -f k8s-service.yaml
kubectl delete -f k8s-deployment.yaml
kubectl delete -f k8s-configmap.yaml
kubectl delete namespace lab-k8s
```

Pour arrêter Minikube:
```bash
minikube stop
```

Pour supprimer complètement Minikube:
```bash
minikube delete
```

## Endpoints Disponibles

- `GET /api/hello` - Retourne un message JSON
- `GET /actuator/health` - Health check de l'application
- `GET /actuator/info` - Informations sur l'application

## Troubleshooting

### Les pods ne démarrent pas
```bash
kubectl describe pod <nom-pod> -n lab-k8s
kubectl logs <nom-pod> -n lab-k8s
```

### L'image n'est pas trouvée
Assurez-vous d'avoir construit l'image dans l'environnement Docker de Minikube:
```bash
eval $(minikube docker-env)
docker build -t demo-k8s:1.0.0 .
```

### Le service n'est pas accessible
Vérifiez que le service est bien créé:
```bash
kubectl get svc -n lab-k8s
minikube service demo-k8s-service -n lab-k8s --url
```

## Extensions Possibles

1. **Ingress**: Exposer l'application avec un nom de domaine
2. **Secrets**: Stocker des informations sensibles
3. **Persistent Volume**: Ajouter du stockage persistant
4. **HorizontalPodAutoscaler**: Scaling automatique basé sur la charge
5. **CI/CD**: Intégrer dans un pipeline de déploiement automatisé
6. **Multi-services**: Ajouter d'autres microservices et tester la communication

## Auteur
Lab Kubernetes - Formation DevOps
