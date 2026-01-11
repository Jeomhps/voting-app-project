# Notes : Déploiement de la voting-app avec Kind

## 1. Création du cluster Kubernetes local avec Kind
**Commande :**
```bash
kind create cluster --name voting-app
```
**Explication :**
- **Kind** = "Kubernetes IN Docker" → Un outil qui crée un cluster Kubernetes **local** en utilisant des conteneurs Docker comme "noeuds".
- Un **cluster** = Un ensemble de machines (appelées **nodes**) qui exécutent Kubernetes.
  - Ici, Kind crée **1 node** (par défaut) qui simule un vrai cluster.
  - *Pourquoi ?* Pour tester Kubernetes sans avoir besoin de machines physiques ou de cloud.

**Vérification :**
```bash
kubectl cluster-info --context kind-voting-app  # Affiche les infos du cluster
kubectl get nodes                              # Liste les nodes (ici, 1 node dans Docker)
```
→ Résultat attendu :
```
NAME                     STATUS   ROLES           AGE   VERSION
voting-app-control-plane Ready    control-plane  2m    v1.27.3
```
*(Le `control-plane` gère le cluster, et c’est aussi un `node` qui peut exécuter des pods.)*

---

## 2. Déploiement des manifests Kubernetes
**Commande :**
```bash
kubectl apply -f k8s-specifications/
```
**Explication :**
- `kubectl apply` = "Applique ces fichiers YAML pour créer des ressources dans le cluster".
- Les fichiers dans `k8s-specifications/` décrivent :
  - **Pods** : Le plus petit élément Kubernetes. Un pod contient **1 ou plusieurs conteneurs** (ex: un conteneur pour le frontend, un pour Redis).
  - **Deployments** : Gèrent les pods (redémarrage si crash, scaling, etc.).
  - **Services** : Permettent aux pods de communiquer entre eux ou avec l’extérieur.
    - Exemple : Le service `vote` expose le pod du frontend pour qu’on puisse y accéder.

**Vérification :**
```bash
kubectl get pods -A  # Liste tous les pods dans tous les namespaces
```
→ Exemple de sortie :
```
NAMESPACE   NAME                       READY   STATUS    RESTARTS   AGE
default     vote-7d5f6d89b7-abc12      1/1     Running   0          3m
default     redis-6f7d4d86f4-def34     1/1     Running   0          3m
```
- **READY 1/1** = 1 conteneur dans le pod est prêt.
- **Running** = Le pod tourne correctement.

---

## 3. Accès à l’UI web avec port-forward
**Commande :**
```bash
kubectl port-forward svc/vote 8080:80
```
**Explication :**
- **Service (svc/vote)** : Une ressource Kubernetes qui expose le pod `vote` (frontend) sur un port interne (ex: 80).
- **Port-forward** : Redirige un port de **ta machine locale** (8080) vers le port du service dans le cluster (80).
  - *Pourquoi ?* Parce que Kind est isolé dans Docker. Sans ça, impossible d’accéder à l’UI depuis ton navigateur.
- **Résultat** : L’UI est disponible sur [http://localhost:8080](http://localhost:8080).

**Schéma :**
```
navigateur → localhost:8080 → (port-forward) → Service "vote" (port 80) → Pod "vote"
```
