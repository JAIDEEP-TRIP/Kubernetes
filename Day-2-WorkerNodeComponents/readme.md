
# 1) Commands (ready to paste in your README)

```sh
# --- Prereqs (Linux/amd64) ---
# You should already have: AWS CLI configured (aws configure), an IAM user/role
# with EKS + EC2 permissions, and Git installed.

# 1) Install kubectl (your pinned 1.19.6 build)
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --short --client

# 2) Install eksctl (helper CLI for EKS)
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
  | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin/eksctl
eksctl version

# 3) Create an EKS cluster (name/region/instance-type from your history)
eksctl create cluster --name jaideep --region us-east-1 --node-type t2.small

# 4) Verify cluster + nodes
kubectl get nodes

# --- Workloads: Pod → ReplicaSet → Deployment ---

# 5) Apply a Pod (expects a local file pod.yml with kind: Pod, metadata.name: myapp)
kubectl apply -f pod.yml
kubectl get pods
kubectl describe pod myapp

# Re-apply after changes, or delete the Pod
kubectl apply -f pod.yml       # re-apply if you edited pod.yml
kubectl delete pod myapp
kubectl get pods

# 6) Apply a ReplicaSet (expects replicaset.yml with kind: ReplicaSet, metadata.name: web-rs)
kubectl apply -f replicaset.yml
kubectl get pods
kubectl get rs

# Delete a single Pod managed by the RS (it will be re-created by the RS)
# Replace the name below with one from `kubectl get pods`
kubectl delete pod web-rs-2khb4
kubectl get pods

# Delete the whole ReplicaSet (scales all its Pods to 0 and removes the RS)
kubectl delete rs web-rs
kubectl get pods

# 7) Apply a Deployment (expects deployment.yml with kind: Deployment, metadata.name: web-deploy)
kubectl apply -f deployment.yml
kubectl get pods
kubectl get deployments
kubectl describe deployment web-deploy
kubectl rollout status deployment/web-deploy

# --- Optional cleanup (to avoid costs) ---
# This removes the EKS cluster and its managed node group(s)
# eksctl delete cluster --name jaideep --region us-east-1
```

---

# 2) What each step/command does (plain-English walkthrough)

**Install kubectl**

* `curl -o kubectl …`: downloads the kubectl binary you pinned (v1.19.6 for linux/amd64).
* `chmod +x ./kubectl`: makes the file executable.
* `sudo mv … /usr/local/bin/kubectl`: moves it into your PATH so you can run `kubectl` from anywhere.
* `kubectl version --short --client`: confirms the client version.

**Install eksctl**

* The `curl … | tar xz -C /tmp` pipeline fetches the latest `eksctl` tarball for your OS/arch and extracts it to `/tmp`.
* `sudo mv /tmp/eksctl /usr/local/bin/eksctl`: puts `eksctl` on your PATH.
* `eksctl version`: verifies it’s installed.

**Create the EKS cluster**

* `eksctl create cluster --name jaideep --region us-east-1 --node-type t2.small`:

  * Provisions a control plane (EKS) and a managed node group of EC2 workers (`t2.small`) in `us-east-1`.
  * Also writes a kubeconfig entry so `kubectl` talks to the new cluster.

**Verify connectivity**

* `kubectl get nodes`: confirms your worker nodes have joined and are `Ready`.

**Work with a standalone Pod**

* `kubectl apply -f pod.yml`: creates/updates a Pod resource from your `pod.yml` (e.g., `metadata.name: myapp`).
* `kubectl get pods`: lists Pod(s) and their status (`Pending/ContainerCreating/Running`).
* `kubectl describe pod myapp`: shows detailed info (events, image pulls, restarts).
* Re-apply/delete:

  * `kubectl apply -f pod.yml`: reapplies if you edited the manifest.
  * `kubectl delete pod myapp`: removes the Pod. If it’s standalone (no controller), it stays deleted.

**Use a ReplicaSet**

* `kubectl apply -f replicaset.yml`: creates a ReplicaSet (e.g., `metadata.name: web-rs`) that maintains a fixed number of **identical Pods**.
* `kubectl get pods`: shows Pods created by the RS (names look like `web-rs-xxxxx`).
* `kubectl get rs`: shows the RS and its desired/current/ready replicas.
* `kubectl delete pod <one-pod-name>`: deletes one managed Pod; the RS immediately recreates it (self-healing).
* `kubectl delete rs web-rs`: removes the RS and all Pods it manages.

**Use a Deployment**

* `kubectl apply -f deployment.yml`: creates a Deployment (e.g., `metadata.name: web-deploy`) which manages **ReplicaSets** for rolling updates.
* `kubectl get deployments`: lists Deployments and high-level status.
* `kubectl describe deployment web-deploy`: detailed spec/status/events (e.g., rollout progress).
* `kubectl rollout status deployment/web-deploy`: waits and reports rollout success/failure.

**Optional cleanup**

* `eksctl delete cluster --name jaideep --region us-east-1`: tears down the control plane, node groups, and supporting resources to stop charges.

---




