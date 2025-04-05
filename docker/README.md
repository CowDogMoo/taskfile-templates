# 🐳 Docker Taskfile Tasks

This directory contains reusable Taskfile tasks for Docker operations,
providing simple commands to inspect and list container mount points.

## 📋 Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed and running
- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### list-docker-mounts

Lists all Docker containers with their mount points, showing both volume and
bind mounts with their respective destinations.

```bash
# List all containers with their mount information
task list-docker-mounts

# The output includes:
# - Container names
# - Mount types (bind, volume)
# - Source paths (for bind mounts)
# - Volume names (for volume mounts)
# - Destination paths in the container
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

# If container is not found, the task will:
# - Display an error message
# - Show usage instructions
# - Exit with an error code
```

## 🔍 Example Output

When you run `task list-docker-mounts`, you'll see output similar to:

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

When you run `task list-container-mounts CONTAINER=nginx-web`:

```bash
Listing mount points for container 'nginx-web':
------------------------------------------------------
/nginx-web

  bind /home/user/website => /usr/share/nginx/html
  volume nginx_logs => /var/log/nginx
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
```
