# 🚢 Kubernetes Taskfile Tasks

This directory contains reusable Taskfile tasks for Kubernetes operations,
including namespace cleanup, pod management, and cluster maintenance tasks.

## 📋 Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed
  and configured
- [jq](https://stedolan.github.io/jq/download/) installed (for JSON processing)
- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)
- [Helm](https://helm.sh/docs/intro/install/) installed (for package management)
- [Python3](https://www.python.org/downloads/) with `bcrypt` package (for
  password hashing)

## 🎯 Available Tasks

### create-kind

Creates a kind cluster with one control plane and one worker node using a
temporary configuration file.

**Variables:**

- `CLUSTER_NAME`: Name for the cluster (default: test-cluster)

```bash
# Create a new cluster with default name
task create-kind

# Create a cluster with custom name
task create-kind CLUSTER_NAME=my-dev-cluster

# The cluster will include:
# - One control-plane node
# - One worker node
```

### destroy-kind

Deletes a kind cluster and cleans up associated Docker resources.

**Variables:**

- `CLUSTER_NAME`: Name of the cluster to delete (default: test-cluster)

```bash
# Delete the default test cluster
task destroy-kind

# Delete a specific cluster
task destroy-kind CLUSTER_NAME=my-custom-cluster

# The task will:
# - Check if the cluster exists before attempting deletion
# - Remove the kind cluster
# - Clean up related Docker resources
```

### destroy-stuck-ns

Removes finalizers from Kubernetes namespaces stuck in the `Terminating` state.
Handles proxy management and cleanup automatically.

**Variables:**

- `PROXY_PORT`: Port to use for kubectl proxy (default: 8001)

```bash
# Remove finalizers using default proxy port
task destroy-stuck-ns

# Use custom proxy port
task destroy-stuck-ns PROXY_PORT=8002

# Run with verbose output to debug issues
task destroy-stuck-ns --verbose
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

### setup-helm

Generic Helm chart installation task that handles repository setup and chart deployment.

**Variables:**

- `HELM_RELEASE`: Name of the Helm release (required)
- `HELM_CHART`: Chart to install (required)
- `HELM_NAMESPACE`: Target namespace (default: default)
- `HELM_VERSION`: Specific chart version to install (optional)
- `HELM_VALUES`: Path to values file (optional)
- `HELM_REPO_NAME`: Name for the Helm repository (required)
- `HELM_REPO_URL`: URL of the Helm repository (required)

```bash
# Install nginx from Bitnami repo
task setup-helm \
  HELM_RELEASE=my-nginx \
  HELM_CHART=bitnami/nginx \
  HELM_NAMESPACE=web \
  HELM_REPO_NAME=bitnami \
  HELM_REPO_URL=https://charts.bitnami.com/bitnami

# Install specific version with custom values
task setup-helm \
  HELM_RELEASE=my-app \
  HELM_CHART=stable/app \
  HELM_VERSION=1.2.3 \
  HELM_VALUES=my-values.yaml \
  HELM_REPO_NAME=stable \
  HELM_REPO_URL=https://charts.helm.sh/stable
```

### setup-argocd

Installs ArgoCD using Helm with secure password management and health checks.

**Variables:**

- `ARGOCD_NAMESPACE`: Target namespace (default: argocd)
- `ARGOCD_VERSION`: Version of ArgoCD to install (default: 5.51.6)
- `ARGOCD_PASSWORD`: Admin password (default: admin)

```bash
# Install with default settings
task setup-argocd

# Custom installation with specific version and namespace
task setup-argocd \
  ARGOCD_NAMESPACE=gitops \
  ARGOCD_VERSION=5.51.6 \
  ARGOCD_PASSWORD=superSecret123

# After installation, access UI with:
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### setup-flux

Installs Flux using Helm with automatic controller health verification.

**Variables:**

- `FLUX_NAMESPACE`: Target namespace (default: flux-system)
- `FLUX_VERSION`: Version of Flux to install (default: 2.14.1)
- `FLUX_VALUES`: Path to custom values file (optional)

```bash
# Install with default settings
task setup-flux

# Custom installation
task setup-flux \
  FLUX_NAMESPACE=gitops \
  FLUX_VERSION=2.14.1 \
  FLUX_VALUES=./flux-values.yaml
```

### uninstall-argocd

Cleanly removes ArgoCD installation and namespace.

**Variables:**

- `ARGOCD_NAMESPACE`: Namespace to remove (default: argocd)

```bash
# Remove default installation
task uninstall-argocd

# Remove from custom namespace
task uninstall-argocd ARGOCD_NAMESPACE=gitops
```

### uninstall-flux

Cleanly removes Flux installation and namespace.

**Variables:**

- `FLUX_NAMESPACE`: Namespace to remove (default: flux-system)

```bash
# Remove default installation
task uninstall-flux

# Remove from custom namespace
task uninstall-flux FLUX_NAMESPACE=gitops
```

## 🔍 Important Notes

- All tasks include proper error handling and cleanup
- ArgoCD installation includes:
  - Secure password hashing using bcrypt
  - LoadBalancer service type
  - Insecure mode enabled for testing
  - Kustomize plugins support
  - Automatic readiness checks
- Flux installation includes:
  - Custom values support
  - Automatic controller health verification
  - Multi-tenant capabilities

## 🔧 Extending Tasks

Import these tasks in your own Taskfile:

```yaml
---
version: "3"
includes:
  k8s: "https://raw.githubusercontent.com/CowDogMoo/taskfile-templates/main/k8s/Taskfile.yaml"

tasks:
  deploy-argo-dev-stack:
    cmds:
      # Create local cluster
      - task: k8s:create-kind

      # Setup GitOps tools
      - task: k8s:setup-argocd
        vars:
          ARGOCD_NAMESPACE: argocd
          ARGOCD_PASSWORD: "{{.CLUSTER_PASSWORD}}"

  deploy-flux-dev-stack:
    cmds:
      # Create local cluster
      - task: k8s:create-kind

      # Setup GitOps tools
      - task: k8s:setup-flux
        vars:
          FLUX_NAMESPACE: flux-system
          FLUX_VALUES: ./config/flux-values.yaml
```
