# 🚢 Kubernetes Taskfile Tasks

This repository contains reusable Taskfile tasks for Kubernetes operations,
including cluster management, GitOps tooling (ArgoCD/Flux), and troubleshooting
utilities.

## 📋 Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) -
  Kubernetes CLI
- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) - Local
  Kubernetes clusters (optional)
- [Helm](https://helm.sh/docs/intro/install/) - Kubernetes package manager
- [Task](https://taskfile.dev) - Task runner (`brew install go-task/tap/go-task`)
- [jq](https://stedolan.github.io/jq/download/) - JSON processor
- [Python3](https://www.python.org/downloads/) with `bcrypt` - For ArgoCD
  password hashing
- [Flux CLI](https://fluxcd.io/flux/installation/#install-the-flux-cli) - For
  Flux operations (optional)

## 📁 Project Structure

```text
.
├── gitops/              # GitOps tooling tasks
│   ├── argo/            # ArgoCD specific tasks
│   └── flux/            # Flux specific tasks
├── helm/                # Helm management tasks
└── Taskfile.yaml        # Main task definitions
```

## 🎯 Available Tasks

### Cluster Management

#### create-kind

Creates a local Kubernetes cluster using kind with one control plane and one
worker node.

```bash
# Create with defaults
task create-kind

# Custom cluster name
task create-kind CLUSTER_NAME=dev-cluster

# Custom config path
task create-kind CONFIG_PATH=./my-config.yaml
```

#### destroy-kind

Removes a kind cluster and cleans up Docker resources.

```bash
# Remove default cluster
task destroy-kind

# Remove specific cluster
task destroy-kind CLUSTER_NAME=dev-cluster
```

### Node Operations

#### describe-node

Get detailed information about a specific node.

```bash
task describe-node NODE=kind-control-plane
```

#### drain-node

Safely evict all pods from a node for maintenance.

```bash
task drain-node NODE=kind-worker
```

#### uncordon-node

Mark a node as schedulable again after maintenance.

```bash
task uncordon-node NODE=kind-worker
```

#### list-node-pods

List all pods running on a specific node.

```bash
task list-node-pods NODE_NAME=kind-worker
```

### Troubleshooting

#### destroy-stuck-ns

Removes finalizers from namespaces stuck in `Terminating` state.

```bash
# Use default proxy port
task destroy-stuck-ns

# Custom proxy port
task destroy-stuck-ns PROXY_PORT=8002
```

#### get-events

Get cluster events sorted by timestamp.

```bash
# All namespaces
task get-events

# Specific namespace
task get-events NAMESPACE=default
```

#### get-pods-all-ns

List all pods across all namespaces.

```bash
task get-pods-all-ns
```

### GitOps - ArgoCD

#### gitops:argo:install

Install ArgoCD with secure password management.

```bash
# Default installation
task gitops:argo:install

# Custom configuration
task gitops:argo:install \
  ARGOCD_NAMESPACE=argocd \
  ARGOCD_VERSION=5.51.6 \
  ARGOCD_PASSWORD=mySecurePassword
```

#### gitops:argo:uninstall

Remove ArgoCD installation.

```bash
task gitops:argo:uninstall ARGOCD_NAMESPACE=argocd
```

### GitOps - Flux

#### gitops:flux:install

Install Flux GitOps toolkit.

```bash
# Default installation
task gitops:flux:install

# Custom configuration
task gitops:flux:install \
  FLUX_NAMESPACE=flux-system \
  FLUX_VERSION=2.14.1 \
  FLUX_VALUES=./flux-values.yaml
```

#### gitops:flux:uninstall

Remove Flux installation.

```bash
task gitops:flux:uninstall FLUX_NAMESPACE=flux-system
```

#### gitops:flux:status

Check the status of all Flux resources.

```bash
task gitops:flux:status
```

#### gitops:flux:logs

View Flux controller logs.

```bash
# View all controller logs
task gitops:flux:logs

# Follow logs
task gitops:flux:logs FOLLOW=true

# Specific controller
task gitops:flux:logs CONTROLLER=source-controller
```

#### gitops:flux:sync-all

Sync all Flux resources (GitRepositories, Kustomizations, HelmReleases).

```bash
task gitops:flux:sync-all
```

#### gitops:flux:sync-gitrepositories

Sync all GitRepository resources.

```bash
task gitops:flux:sync-gitrepositories
```

#### gitops:flux:sync-kustomizations

Sync all Kustomization resources.

```bash
task gitops:flux:sync-kustomizations
```

#### gitops:flux:sync-helmreleases

Sync all HelmRelease resources.

```bash
task gitops:flux:sync-helmreleases
```

#### gitops:flux:get-not-ready

Get all Flux resources that are not in ready state.

```bash
task gitops:flux:get-not-ready
```

### Helm Operations

#### helm:setup-helm

Generic Helm chart installation with repository management.

```bash
task helm:setup-helm \
  HELM_RELEASE=nginx \
  HELM_CHART=bitnami/nginx \
  HELM_NAMESPACE=web \
  HELM_REPO_NAME=bitnami \
  HELM_REPO_URL=https://charts.bitnami.com/bitnami \
  HELM_VERSION=15.0.0 \
  HELM_VALUES=./values.yaml
```

## 🔧 Usage Examples

### Complete Development Stack with ArgoCD

```yaml
# In your Taskfile.yaml
version: "3"
includes:
  k8s:
    taskfile: ./path/to/k8s/tasks

tasks:
  setup-dev:
    cmds:
      - task: k8s:create-kind
        vars:
          CLUSTER_NAME: dev
      - task: k8s:gitops:argo:install
        vars:
          ARGOCD_PASSWORD: "{{.PASSWORD}}"
```

### Flux GitOps Workflow

```bash
# Install Flux
task gitops:flux:install

# Check status
task gitops:flux:status

# Watch logs
task gitops:flux:logs FOLLOW=true

# Force sync all resources
task gitops:flux:sync-all

# Troubleshoot issues
task gitops:flux:get-not-ready
```

### Cluster Maintenance

```bash
# Drain node for updates
task drain-node NODE=worker-1

# Perform maintenance...

# Make node schedulable again
task uncordon-node NODE=worker-1
```

## 🔍 Important Notes

- **ArgoCD Installation**: Includes secure bcrypt password hashing,
  LoadBalancer service, and Kustomize plugin support
- **Flux Installation**: Automatic controller health checks and multi-tenant support
- **Error Handling**: All tasks include proper error handling and cleanup
- **Proxy Management**: Automatic proxy lifecycle management for API operations
- **Namespace Cleanup**: Handles resource conflicts and retries automatically

## 📚 Task Namespacing

Tasks are organized hierarchically:

- Root tasks: Core Kubernetes operations
- `gitops:*`: GitOps tooling (ArgoCD, Flux)
- `helm:*`: Helm package management

Access nested tasks using colon notation:

```bash
task gitops:flux:status
task gitops:argo:install
```
