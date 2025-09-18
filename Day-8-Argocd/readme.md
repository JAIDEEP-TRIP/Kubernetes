
````markdown
# 🚀 ArgoCD Overview & Practical Example

Argo CD (short for *Argo Continuous Delivery*) is a **Kubernetes-native GitOps continuous deployment tool**.  
It automates the deployment of applications to Kubernetes clusters, continuously ensuring that the deployed state matches the Git repository state.

---

## 📖 What is ArgoCD?

Argo CD works by **pulling the desired state** (application manifests, Helm charts, or Kustomize configs) from Git repositories and applying them to Kubernetes.  

Unlike traditional **push-based CD tools** (e.g., Jenkins pipelines pushing to Kubernetes), Argo CD is **pull-based**:  
- The desired configuration lives in Git.  
- Argo CD watches Git for changes.  
- Argo CD reconciles Kubernetes resources to always match Git.  

---

## 🔹 What is GitOps?

**GitOps** is an operational model for Kubernetes and cloud-native apps:  
- Git = the *single source of truth* for infrastructure and applications.  
- Declarative configs describe the desired state.  
- Automated tools (like Argo CD) ensure the cluster matches Git.  

Benefits of GitOps:  
✅ Reliable rollbacks (via Git history)  
✅ Audit trail of all changes  
✅ Self-healing, consistent deployments  
✅ Improved collaboration between Dev, Ops, and DevOps  

---

## ✨ Benefits of ArgoCD

1. **Improved developer productivity** – self-service deployments, less manual work.  
2. **Compliance & security** – central Git repo with policies and RBAC.  
3. **Team collaboration** – shared visibility into deployments.  
4. **Faster deployments** – automate rollouts across clusters and clouds.  

---

## ⚡ Prerequisites

- A running Kubernetes cluster (e.g., Minikube, Kind, or cloud provider).  
- `kubectl` CLI configured to access the cluster.  

---

## 🛠️ Installation of ArgoCD

### 1. Create Namespace
```bash
kubectl create namespace argocd
````

### 2. Install ArgoCD Components

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3. Verify Installation

```bash
kubectl get all -n argocd
```

You should see pods like:

* `argocd-server`
* `argocd-repo-server`
* `argocd-application-controller`

---

## 🌐 Accessing the ArgoCD UI

### 1. Expose the ArgoCD Server

Edit the service to use `NodePort`:

```bash
kubectl edit svc argocd-server -n argocd
```

Change:

```yaml
type: ClusterIP
```

to:

```yaml
type: NodePort
```

### 2. Get Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

Decode it:

```bash
echo "<base64-secret>" | base64 --decode
```

* Username: `admin`
* Password: `<decoded value>`

Now login via browser → `http://<NodeIP>:<NodePort>`

---

## 📦 Example Application with ArgoCD

### 1. Kubernetes Manifests

**Deployment (`deployment.yaml`)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: swiggy-app
  labels:
    app: swiggy-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: swiggy-app
  template:
    metadata:
      labels:
        app: swiggy-app
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: swiggy-app
        image: veeranarni/hotstar:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
```

**Service (`service.yaml`)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: swiggy-app
  labels:
    app: swiggy-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: swiggy-app
```

---

### 2. ArgoCD Application Manifest

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/JAIDEEP-TRIP/Kubernetes.git
    targetRevision: HEAD
    path: Day-8-Argocd
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
    - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```

---

### 3. Apply the ArgoCD Application

```bash
kubectl apply -f application.yaml
```

Check in ArgoCD UI → application should appear.

---

## 🔄 Auto-Sync and Updates

* **selfHeal: true** → Reverts manual changes (`kubectl edit`) to match Git.
* **prune: true** → Deletes resources removed from Git.

Example: Increase replicas in `deployment.yaml`:

```yaml
replicas: 4
```

Commit & push → ArgoCD auto-syncs → Cluster updates to 4 pods:

```bash
kubectl get pods -n myapp
```

---

## 🎯 Summary

* **ArgoCD** = Kubernetes-native GitOps CD tool.
* **Git is the source of truth** – any change in Git is reflected in the cluster.
* Benefits: automated rollouts, version control, collaboration, compliance.
* Practical workflow:

  1. Install ArgoCD
  2. Connect Git repo
  3. Define `Application` CRD
  4. ArgoCD syncs & manages deployments automatically

---

## 📚 References

* [ArgoCD Docs](https://argo-cd.readthedocs.io/en/stable/)
* [GitOps Principles](https://www.gitops.tech/)
* [Argo Project GitHub](https://github.com/argoproj/argo-cd)

```

---

Would you like me to also add a **diagram (ASCII or Mermaid)** showing the GitOps workflow (Git → ArgoCD → Kubernetes → App) so that this `README.md` is more visually intuitive?
```
