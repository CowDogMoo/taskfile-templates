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

#### argo:install

Install ArgoCD with secure password management.

```bash
# Default installation
task argo:install

# Custom configuration
task argo:install \
  ARGOCD_NAMESPACE=argocd \
  ARGOCD_VERSION=5.51.6 \
  ARGOCD_PASSWORD=mySecurePassword
```

#### argo:uninstall

Remove ArgoCD installation.

```bash
task argo:uninstall ARGOCD_NAMESPACE=argocd
```

### GitOps - Flux

#### flux:install

Install Flux GitOps toolkit.

```bash
# Default installation
task flux:install

# Custom configuration
task flux:install \
  FLUX_NAMESPACE=flux-system \
  FLUX_VERSION=2.14.1 \
  FLUX_VALUES=./flux-values.yaml
```

#### flux:uninstall

Remove Flux installation.

```bash
task flux:uninstall FLUX_NAMESPACE=flux-system
```

#### flux:status

Check the status of all Flux resources.

```bash
task flux:status
```

#### flux:logs

View Flux controller logs.

```bash
# View all controller logs
task flux:logs

# Follow logs
task flux:logs FOLLOW=true

# Specific controller
task flux:logs CONTROLLER=source-controller
```

#### flux:sync-all

Force reconciliation of all Flux and ExternalSecrets resources.

```bash
task flux:sync-all
```

This syncs:

- GitRepositories
- OCIRepositories
- Kustomizations
- HelmReleases (with force upgrade)
- ExternalSecrets

#### flux:sync-gitrepositories

Force reconciliation of all GitRepository resources.

```bash
task flux:sync-gitrepositories
```

#### flux:sync-ocirepositories

Force reconciliation of all OCIRepository resources.

```bash
task flux:sync-ocirepositories
```

#### flux:sync-kustomizations

Force reconciliation of all Kustomization resources.

```bash
task flux:sync-kustomizations
```

#### flux:sync-helmreleases

Force reconciliation and upgrade of all HelmRelease resources.

```bash
task flux:sync-helmreleases
```

**Note**: This applies both `requestedAt` and `forceAt` annotations to ensure
upgrades are performed even when no changes are detected.

#### flux:sync-externalsecrets

Force synchronization of all ExternalSecrets resources.

```bash
task flux:sync-externalsecrets
```

#### flux:get-not-ready

Get all Flux resources that are not in ready state.

```bash
task flux:get-not-ready
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
      - task: k8s:argo:install
        vars:
          ARGOCD_PASSWORD: "{{.PASSWORD}}"
```

### Flux GitOps Workflow

```bash
# Install Flux
task flux:install

# Check status
task flux:status

# Watch logs
task flux:logs FOLLOW=true

# Force sync all resources
task flux:sync-all

# Force sync specific resource types
task flux:sync-helmreleases  # Forces helm upgrades
task flux:sync-externalsecrets  # Refreshes secrets from external sources

# Troubleshoot issues
task flux:get-not-ready
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

### Flux Synchronization

- **Resource Annotations**: All sync tasks use proper field managers
  (`flux-client-side-apply`) for ownership tracking
- **HelmRelease Sync**: Applies both `requestedAt` and `forceAt` annotations to
  ensure upgrades happen even without changes
- **ExternalSecrets**: Uses the `force-sync` annotation specific to the
  external-secrets operator
- **Timestamp-based**: All sync operations use Unix timestamps to ensure unique
  reconciliation requests

### General Notes

- **ArgoCD Installation**: Includes secure bcrypt password hashing,
  LoadBalancer service, and Kustomize plugin support
- **Flux Installation**: Automatic controller health checks and multi-tenant support
- **Error Handling**: All tasks include proper error handling and cleanup
- **Proxy Management**: Automatic proxy lifecycle management for API operations
- **Namespace Cleanup**: Handles resource conflicts and retries automatically

## 📚 Task Namespacing

Tasks are organized hierarchically:

- Root tasks: Core Kubernetes operations
- `argo:*`: ArgoCD GitOps tooling
- `flux:*`: Flux GitOps tooling
- `helm:*`: Helm package management

Access nested tasks using colon notation:

```bash
task flux:status
task argo:install
```
