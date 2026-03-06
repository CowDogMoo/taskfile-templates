# 🐳 Docker Taskfile Tasks

This directory contains reusable Taskfile tasks for Docker operations,
providing simple commands to inspect container mount points, clean up Docker
resources, and manage multi-architecture images.

## 📋 Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed and running
- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### check-docker

Validates that Docker is installed and running on your system.

```bash
task check-docker
```

### list-docker-mounts

Lists all Docker containers with their mount points, showing both volume and
bind mounts with their respective destinations.

```bash
# List all containers with their mount information
task list-docker-mounts
```

**Example output:**

```bash
Listing all containers with their mount points:
----------------------------------------------
/mysql-server

  volume mysql_data => /var/lib/mysql

/nginx-web

  bind /home/user/website => /usr/share/nginx/html
  volume nginx_logs => /var/log/nginx

/redis-cache

  volume redis_data => /data
```

### list-container-mounts

Lists mount points for a specific container, allowing targeted inspection of
volume mappings.

**Variables:**

- `CONTAINER`: Name or ID of the container to inspect (required)

```bash
# List mount points for a specific container by name
task list-container-mounts CONTAINER=my-nginx

# List mount points for a specific container by ID
task list-container-mounts CONTAINER=a1b2c3d4e5f6
```

**Example output:**

```bash
Listing mount points for container 'nginx-web':
------------------------------------------------------
/nginx-web

  bind /home/user/website => /usr/share/nginx/html
  volume nginx_logs => /var/log/nginx
```

### clean-docker 🧹

Performs a comprehensive cleanup of Docker resources, removing all unused
containers, images, networks, volumes, and build cache.

```bash
# Clean up all unused Docker resources
task clean-docker
```

⚠️ **Warning**: This is a destructive operation that will remove:

- All stopped containers
- ALL unused images (including dangling and unreferenced images)
- All unused networks
- ALL unused volumes (including named volumes)
- All build cache

Make sure you have backed up any important data before running this command!

**Example output:**

```bash
🧹 Starting Docker cleanup...
📦 Removing stopped containers...
Deleted Containers:
a1b2c3d4e5f6
g7h8i9j0k1l2

🖼️  Removing all unused images...
Deleted Images:
sha256:123abc...
sha256:456def...
Total reclaimed space: 2.5GB

🌐 Removing unused networks...
Deleted Networks:
old_network
test_network

💾 Removing unused volumes...
Deleted Volumes:
unused_volume_1
old_data_volume
Total reclaimed space: 500MB

🔨 Removing build cache...
Deleted build cache objects:
abc123def456
Total reclaimed space: 1.2GB

✅ Docker cleanup complete!
```

### push-multi-arch

Pushes multi-architecture Docker images to a registry
(default: GitHub Container Registry).

**Required Variables:**

- `NAMESPACE`: Registry namespace (e.g., username or organization)
- `IMAGE_NAME`: Name of the Docker image
- `ARM64_HASH`: Hash/ID of the ARM64 image
- `AMD64_HASH`: Hash/ID of the AMD64 image
- `GITHUB_TOKEN`: GitHub token for authentication
- `GITHUB_USER`: GitHub username

**Optional Variables:**

- `REGISTRY`: Docker registry URL (default: `ghcr.io`)
- `TAG`: Image tag (default: `latest`)

```bash
# Push multi-architecture image to GitHub Container Registry
task push-multi-arch \
  NAMESPACE=myorg \
  IMAGE_NAME=myapp \
  ARM64_HASH=sha256:abc123... \
  AMD64_HASH=sha256:def456... \
  GITHUB_TOKEN=$GITHUB_TOKEN \
  GITHUB_USER=myusername

# Push with custom tag and registry
task push-multi-arch \
  REGISTRY=docker.io \
  NAMESPACE=myorg \
  IMAGE_NAME=myapp \
  TAG=v1.2.3 \
  ARM64_HASH=sha256:abc123... \
  AMD64_HASH=sha256:def456... \
  GITHUB_TOKEN=$DOCKER_TOKEN \
  GITHUB_USER=myusername
```

This task will:

1. Authenticate to the registry (if not already authenticated)
1. Check if images already exist in the registry to avoid redundant pushes
1. Tag and push both ARM64 and AMD64 images
1. Create and push a multi-architecture manifest

### setup-buildx

Creates and configures a Docker buildx builder for multi-architecture builds.

**Optional Variables:**

- `BUILDER_NAME`: Name of the buildx builder (default: `multiarch-builder`)

```bash
# Create default builder
task setup-buildx

# Create builder with custom name
task setup-buildx BUILDER_NAME=my-builder
```

This task is idempotent - it will reuse an existing builder if one with the same
name already exists.

### login

Authenticates to a container registry using a GitHub token.

**Required Environment Variables:**

- `GITHUB_TOKEN`: GitHub token for authentication

**Optional Variables:**

- `REGISTRY`: Docker registry URL (default: `ghcr.io`)

**Optional Environment Variables:**

- `GITHUB_USER`: GitHub username (default: current user)

```bash
# Login to GitHub Container Registry
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx
task login

# Login to a different registry
task login REGISTRY=docker.io
```

### inspect-manifest

Inspects a multi-architecture image manifest to verify platforms and digests.

**Required Variables:**

- `IMAGE`: Full image reference to inspect

```bash
# Inspect a multi-arch image
task inspect-manifest IMAGE=ghcr.io/myorg/myapp:latest
```

**Example output:**

```bash
Name:      ghcr.io/myorg/myapp:latest
MediaType: application/vnd.oci.image.index.v1+json
Digest:    sha256:abc123...

Manifests:
  Name:      ghcr.io/myorg/myapp:latest@sha256:def456...
  MediaType: application/vnd.oci.image.manifest.v1+json
  Platform:  linux/amd64

  Name:      ghcr.io/myorg/myapp:latest@sha256:ghi789...
  MediaType: application/vnd.oci.image.manifest.v1+json
  Platform:  linux/arm64
```

## 🔧 Extending Tasks

Import these tasks in your own Taskfile:

```yaml
---
version: "3"
includes:
  docker: "https://raw.githubusercontent.com/username/taskfile-templates/main/docker/Taskfile.yaml"

tasks:
  check-project-mounts:
    desc: "Check mount points for project containers"
    cmds:
      - echo "Checking database container mounts:"
      - task: docker:list-container-mounts
        vars:
          CONTAINER: project-db

      - echo "Checking web container mounts:"
      - task: docker:list-container-mounts
        vars:
          CONTAINER: project-web

  cleanup-and-rebuild:
    desc: "Clean Docker and rebuild project"
    cmds:
      - task: docker:clean-docker
      - docker-compose build --no-cache
      - docker-compose up -d

  deploy-multiarch:
    desc: "Build and deploy multi-architecture images"
    cmds:
      - docker buildx build --platform linux/arm64 -t myapp:arm64 .
      - docker buildx build --platform linux/amd64 -t myapp:amd64 .
      - task: docker:push-multi-arch
        vars:
          NAMESPACE: "{{.DOCKER_NAMESPACE}}"
          IMAGE_NAME: myapp
          ARM64_HASH: "$(docker images -q myapp:arm64)"
          AMD64_HASH: "$(docker images -q myapp:amd64)"
          GITHUB_TOKEN: "{{.GITHUB_TOKEN}}"
          GITHUB_USER: "{{.GITHUB_USER}}"
```

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Taskfile Documentation](https://taskfile.dev/docs/)
- [Docker Multi-Architecture Images](https://docs.docker.com/build/building/multi-platform/)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
