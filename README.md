# NGINX sur Google Kubernetes Engine

Ce projet déploie une page de démonstration NGINX sur un cluster Google Kubernetes Engine (GKE), avec une exposition HTTP via la Gateway API et une `GatewayClass` GKE managée.

## Contenu

- `Deployment` `nginx-hello` avec 2 réplicas de l’image `nginxdemos/hello:latest` ;
- sondes de readiness et de liveness HTTP ;
- limites et réservations de ressources CPU/mémoire ;
- `Service` interne de type `ClusterIP` ;
- `Gateway` externe GKE ;
- `HTTPRoute` qui achemine les requêtes reçues sur `/` vers le service NGINX.

## Prérequis

- un cluster GKE accessible avec `kubectl` configuré ;
- les droits nécessaires pour créer des ressources Kubernetes et une Gateway externe ;
- la Gateway API activée et une `GatewayClass` nommée `gke-l7-global-external-managed` disponible dans le cluster.

Vérifier la connexion au cluster :

```bash
kubectl cluster-info
kubectl get gatewayclass gke-l7-global-external-managed
```

## Déploiement

Appliquer le manifeste :

```bash
kubectl apply -f nginx.yaml
```

Vérifier les ressources créées :

```bash
kubectl get deployment, pods, service
kubectl get gateway nginx-gateway
kubectl get httproute nginx-hello-route
```

La création d’une Gateway externe peut prendre quelques minutes. Attendre qu’une adresse soit attribuée :

```bash
kubectl get gateway nginx-gateway -w
```

Lorsque la colonne `ADDRESS` est renseignée, accéder à cette adresse dans un navigateur ou tester le service :

```bash
curl http://<ADRESSE-DE-LA-GATEWAY>/
```

## Vérification des pods

```bash
kubectl rollout status deployment/nginx-hello
kubectl describe deployment nginx-hello
kubectl logs deployment/nginx-hello
```

## Nettoyage

Supprimer toutes les ressources déclarées par le manifeste :

```bash
kubectl delete -f nginx.yaml
```

> La Gateway externe peut entraîner des coûts liés aux ressources Google Cloud associées. Pensez à la supprimer après utilisation.

## Configuration principale

| Ressource | Nom | Rôle |
| --- | --- | --- |
| Deployment | `nginx-hello` | Exécute 2 pods NGINX |
| Service | `nginx-hello` | Expose NGINX dans le cluster sur le port 80 |
| Gateway | `nginx-gateway` | Fournit l’entrée HTTP externe |
| HTTPRoute | `nginx-hello-route` | Dirige `/` vers le service NGINX |

