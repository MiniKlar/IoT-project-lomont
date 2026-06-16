# IoT-project-lomont

> GitOps manifests for Part 3 (P3) of **Inception-of-Things** (42 school project).
> This repository is the Git source of truth that **Argo CD** watches and synchronises into a lightweight **k3d/k3s** Kubernetes cluster.
>
> Manifestes GitOps pour la partie 3 (P3) du projet **Inception-of-Things** (école 42).
> Ce dépôt est la source Git que **Argo CD** surveille et synchronise dans un cluster Kubernetes léger **k3d/k3s**.

---

## 🇬🇧 English

### Role of this repo

In Inception-of-Things P3, you install **Argo CD** on a **k3d** cluster and configure it to follow a Git repository. **This repository is that GitOps source.** It contains nothing to build or run locally — it only holds the Kubernetes manifests describing the application. Argo CD reads them and reconciles the cluster state so that the live deployment always matches what is committed here. Pushing a change here (for example bumping the image tag) is what triggers a redeploy.

### What's inside

Only two manifest files, both targeting the `dev` namespace:

- **`deployment.yaml`** — a `Deployment` named `kyaubry-app` with `1` replica, running the container `kyaubry42` from the image `pouletenfuite/red-tetris:v1`, exposing container port `8888`.
- **`service.yaml`** — a `Service` named `kyaubry-app-service` of type `LoadBalancer`, forwarding TCP `8888` → target port `8888`, selecting pods labelled `app: kyaubry-app`.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kyaubry-app
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kyaubry-app
  template:
    metadata:
      labels:
        app: kyaubry-app
    spec:
      containers:
      - name: kyaubry42
        image: pouletenfuite/red-tetris:v1
        ports:
        - containerPort: 8888
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: kyaubry-app-service
  namespace: dev
spec:
  selector:
    app: kyaubry-app
  ports:
    - protocol: TCP
      port: 8888
      targetPort: 8888
  type: LoadBalancer
```

### How it's used

There is **no local build step**. The workflow is GitOps:

1. A k3d cluster is created and Argo CD is installed in it (this is done by the P3 setup, in the main Inception-of-Things repo — not here).
2. An Argo CD `Application` is configured to point at **this** repository (the manifests above) and at the `dev` namespace.
3. Argo CD continuously syncs: it applies `deployment.yaml` and `service.yaml` to the cluster, creating the `kyaubry-app` Deployment and its `kyaubry-app-service` Service.
4. Any commit pushed to this repo (e.g. changing `image: pouletenfuite/red-tetris:v1` to a `:v2` tag) is detected by Argo CD, which rolls out the new version automatically. The Git history of this repo shows exactly these image-tag flips used to demonstrate the sync.

The application is reachable through the `LoadBalancer` Service on port `8888`.

### What I learned

- Writing declarative **Kubernetes** manifests (`Deployment`, `Service`) and how their labels/selectors tie pods to a Service.
- Running a lightweight Kubernetes cluster with **k3d / k3s**.
- The **GitOps** model with **Argo CD**: Git as the single source of truth, automatic reconciliation, and redeploys driven by commits rather than manual `kubectl apply`.
- Exposing a containerised app through a `LoadBalancer` Service and namespacing workloads (`dev`).

---

## 🇫🇷 Français

### Rôle du dépôt

Dans la partie 3 (P3) d'Inception-of-Things, on installe **Argo CD** sur un cluster **k3d** et on le configure pour suivre un dépôt Git. **Ce dépôt est cette source GitOps.** Il n'y a rien à compiler ni à lancer en local : il ne contient que les manifestes Kubernetes décrivant l'application. Argo CD les lit et réconcilie l'état du cluster pour que le déploiement en cours corresponde toujours à ce qui est commité ici. Pousser une modification ici (par exemple changer le tag de l'image) déclenche un redéploiement.

### Contenu

Seulement deux fichiers de manifeste, ciblant tous deux le namespace `dev` :

- **`deployment.yaml`** — un `Deployment` nommé `kyaubry-app` avec `1` réplica, exécutant le conteneur `kyaubry42` à partir de l'image `pouletenfuite/red-tetris:v1`, exposant le port conteneur `8888`.
- **`service.yaml`** — un `Service` nommé `kyaubry-app-service` de type `LoadBalancer`, qui redirige le TCP `8888` → port cible `8888`, en sélectionnant les pods étiquetés `app: kyaubry-app`.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kyaubry-app
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kyaubry-app
  template:
    metadata:
      labels:
        app: kyaubry-app
    spec:
      containers:
      - name: kyaubry42
        image: pouletenfuite/red-tetris:v1
        ports:
        - containerPort: 8888
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: kyaubry-app-service
  namespace: dev
spec:
  selector:
    app: kyaubry-app
  ports:
    - protocol: TCP
      port: 8888
      targetPort: 8888
  type: LoadBalancer
```

### Utilisation

Il n'y a **aucune étape de build local**. Le fonctionnement est en GitOps :

1. Un cluster k3d est créé et Argo CD y est installé (réalisé par la mise en place de la P3, dans le dépôt principal d'Inception-of-Things, pas ici).
2. Une `Application` Argo CD est configurée pour pointer sur **ce** dépôt (les manifestes ci-dessus) et sur le namespace `dev`.
3. Argo CD synchronise en continu : il applique `deployment.yaml` et `service.yaml` au cluster, créant le Deployment `kyaubry-app` et son Service `kyaubry-app-service`.
4. Tout commit poussé sur ce dépôt (par exemple passer `image: pouletenfuite/red-tetris:v1` à un tag `:v2`) est détecté par Argo CD, qui déploie la nouvelle version automatiquement. L'historique Git de ce dépôt montre précisément ces changements de tag d'image utilisés pour démontrer la synchronisation.

L'application est accessible via le Service `LoadBalancer` sur le port `8888`.

### Ce que ça m'a apporté

- Écrire des manifestes **Kubernetes** déclaratifs (`Deployment`, `Service`) et comprendre comment les labels/selectors relient les pods à un Service.
- Faire tourner un cluster Kubernetes léger avec **k3d / k3s**.
- Le modèle **GitOps** avec **Argo CD** : Git comme source de vérité unique, réconciliation automatique, et redéploiements pilotés par les commits plutôt que par un `kubectl apply` manuel.
- Exposer une application conteneurisée via un Service `LoadBalancer` et cloisonner les charges de travail dans un namespace (`dev`).
