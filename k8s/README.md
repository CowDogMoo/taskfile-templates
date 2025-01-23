# 🚢 Kubernetes Taskfile Tasks

This directory contains reusable Taskfile tasks for Kubernetes operations,
including namespace cleanup, pod management, and cluster maintenance tasks.

## 📋 Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed
  and configured
- [jq](https://stedolan.github.io/jq/download/) installed (for JSON processing)
- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)
- [Helm](https://helm.sh/docs/intro/install/) installed (for package management)
- [Flux](https://fluxcd.io/flux/installation/) (optional, for GitOps workflow)
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/)
  (optional, for GitOps workflow)

---

## 🎯 Available Tasks

### create-kind

Creates a kind cluster with one control plane and one worker node.

```bash
task create-kind
```

### destroy-stuck-ns

Removes finalizers from Kubernetes namespaces stuck in the `Terminating` state.
The task runs in parallel for multiple namespaces and includes proper cleanup
of temporary resources.

**Variables:**

- `PROXY_PORT`: Port for kubectl proxy (default: 8001)

```bash
# Run with default proxy port
task destroy-stuck-ns

# Run with custom proxy port
task destroy-stuck-ns PROXY_PORT=8002
```

### list-node-pods

Lists all pods running on a specific Kubernetes node across all namespaces.

**Variables:**

- `NODE_NAME`: Target node name (default: node1)

```bash
# List pods on default node
task list-node-pods

# List pods on specific node
task list-node-pods NODE_NAME=k8s6
```

### setup-helm-in-kind

Installs Helm and deploys a test chart (e.g., Nginx) in the kind cluster.

```bash
task setup-helm-in-kind
```

### setup-flux-in-kind

Bootstraps Flux in the kind cluster for GitOps-based deployments.

```bash
task setup-flux-in-kind
```

### setup-argocd-in-kind

Installs ArgoCD in the kind cluster and exposes the UI.

```bash
task setup-argocd-in-kind
```

---

## 📝 Example Usage

### **Create a kind cluster**

```bash
task create-kind

# Switch to the new cluster
kubectl cluster-info --context kind-test-cluster
```

---

### **📦 Using Helm with kind**

Once the kind cluster is up, you can install Helm and deploy a chart:

```bash
task setup-helm-in-kind

# Verify Helm installation
helm version

# Install an example Nginx chart
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-nginx bitnami/nginx

# Check the status of the deployment
kubectl get pods -n default
```

**Uninstalling the Helm chart:**

```bash
helm uninstall my-nginx
```

---

### **🚀 Using Flux with kind**

To set up Flux in the kind cluster for GitOps workflows:

```bash
task setup-flux-in-kind

# Check Flux components
kubectl get pods -n flux-system
```

**If using a GitHub repository for GitOps:**

```bash
flux bootstrap github \
  --owner=my-github-user \
  --repository=my-flux-repo \
  --branch=main \
  --path=clusters/kind-cluster
```

**If using a local source for Helm releases:**

```bash
flux create source helm podinfo \
  --url=https://stefanprodan.github.io/podinfo

flux create helmrelease podinfo \
  --source=HelmRepository/podinfo \
  --chart=podinfo \
  --namespace=default
```

---

### **🚢 Using ArgoCD with kind**

To install ArgoCD in the kind cluster:

```bash
task setup-argocd-in-kind

# Check ArgoCD components
kubectl get pods -n argocd
```

**Retrieve the ArgoCD admin password:**

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**Expose the ArgoCD UI:**

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Now, open a browser and go to **https://localhost:8080**, then log in with:

- **Username:** `admin`
- **Password:** (output from the previous command)

**Deploy an application with ArgoCD:**

```bash
argocd app create my-app \
  --repo https://github.com/my-org/my-repo.git \
  --path my-app \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

**Sync the application:**

```bash
argocd app sync my-app
```

---

## 🔧 Extending Tasks

You can extend these tasks in your own Taskfile by importing this template and
overriding or adding new tasks. Here's an example:

```yaml
version: "3"

includes:
  k8s:
    taskfile: ./kubernetes.yml
    optional: true

tasks:
  # Override or extend existing tasks
  list-node-pods:
    deps: [k8s:list-node-pods]
    cmds:
      - echo "Additional pod filtering steps..."

  # Add new tasks that use the base tasks
  cleanup-node:
    cmds:
      - task: k8s:list-node-pods
        vars:
          NODE_NAME: worker-1
      - echo "Performing additional cleanup..."
```

---

## 🔍 Important Notes

- The `destroy-stuck-ns` task requires:
  - `jq` for JSON processing
  - Proper permissions to modify namespace finalizers
  - An available port for kubectl proxy
- `setup-helm-in-kind` assumes Helm is installed on your system
- `setup-flux-in-kind` assumes Flux CLI is installed and authenticated for
  GitOps operations
- `setup-argocd-in-kind` assumes ArgoCD CLI is installed for application
  management
- Use `--verbose` for detailed task execution output
- All tasks assume proper `kubectl` configuration and cluster access
- Tasks can be combined or chained for more complex operations
