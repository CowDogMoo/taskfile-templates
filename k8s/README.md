# 🚢 Kubernetes Taskfile Tasks

This directory contains reusable Taskfile tasks for Kubernetes operations,
including namespace cleanup, pod management, and cluster maintenance tasks.

## 📋 Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed
  and configured
- [jq](https://stedolan.github.io/jq/download/) installed (for JSON processing)
- Task installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### create-kind

Creates a kind cluster with one control plane and one worker node.

```bash
task create-kind
```

### destroy-stuck-ns

Removes finalizers from Kubernetes namespaces stuck in Terminating state. The
task runs in parallel for multiple namespaces and includes proper cleanup of
temporary resources.

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

## 📝 Example Usage

**Create a kind cluster:**

```bash
task create-kind

# Switch to the new cluster
kubectl cluster-info --context kind-test-cluster
```

**Clean up stuck namespaces:**

```bash
task destroy-stuck-ns
```

**List pods on a specific node:**

```bash
task list-node-pods NODE_NAME=worker-1
```

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

## 🔍 Important Notes

- The `destroy-stuck-ns` task requires:
  - `jq` for JSON processing
  - Proper permissions to modify namespace finalizers
  - Available port for kubectl proxy
- Use `--verbose` flag for detailed task execution output
- All tasks assume proper kubectl configuration and cluster access
- Tasks can be combined or chained for more complex operations
