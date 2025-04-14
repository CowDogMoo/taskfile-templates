# 🤖 Renovate Taskfile Templates

This directory contains reusable Taskfile templates for running Renovate bot
operations in Docker, making it easy to test and debug Renovate configurations.

## 📋 Prerequisites

- [Docker](https://www.docker.com/get-started) installed and running
- [GitHub CLI](https://cli.github.com/) installed and authenticated
- Task installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### check-repository-format

Validates that the repository format matches the expected `org/repo` pattern.

```bash
task check-repository-format REPOSITORY=owner/repo
```

### renovate-docker-debug

Runs Renovate in a Docker container with debugging enabled for a specific repository.

```bash
export REPOSITORY="owner/repo"
GITHUB_TOKEN=$(gh auth token) task renovate:renovate-docker-debug
```

Required variables:

- `REPOSITORY`: GitHub repository in the format `owner/repo`
- `GITHUB_TOKEN`: GitHub personal access token

Optional variables:

- `LOG_LEVEL`: Logging level (defaults to 'debug')
- `DEBUG`: Enable or disable debug mode (defaults to 'true')
- `PLATFORM_COMMIT`: Whether Renovate should commit changes (defaults to 'true')
- `CONFIG_FILE_NAME`: Name of the Renovate config file (defaults to '.github/renovate.json5')
- `CONFIG_FILE_PATH`: Path to the Renovate config file (defaults to '.github/renovate.json5')
- `GIT_AUTHOR`: Git author for commits (defaults to 'Renovate Bot <bot@renovateapp.com>')
- `PLATFORM`: Platform to use (defaults to 'github')
- `ENDPOINT`: API endpoint (defaults to 'https://api.github.com')
- `FORK`: Whether to fork the repository (defaults to 'false')
- `RENOVATE_IMAGE`: Docker image to use (defaults to 'ghcr.io/renovatebot/renovate:latest')
- `LOG_FILE`: File to save logs to (defaults to 'renovate-debug.log')
- `VALIDATE_REPO`: Whether to validate repository existence (defaults to 'true')

## 📝 Example Usage

1. **Running Renovate in debug mode for a specific repository:**

   ```bash
   export REPOSITORY="microsoft/vscode"
   export GITHUB_TOKEN=$(gh auth token)
   task renovate:renovate-docker-debug
   ```

1. **Using a custom Renovate configuration file:**

   ```bash
   export REPOSITORY="facebook/react"
   export GITHUB_TOKEN=$(gh auth token)
   task renovate:renovate-docker-debug CONFIG_FILE_PATH="custom-renovate.json5"
   ```

1. **Using a specific Renovate Docker image version:**

   ```bash
   export REPOSITORY="angular/angular"
   export GITHUB_TOKEN=$(gh auth token)
   task renovate:renovate-docker-debug RENOVATE_IMAGE="ghcr.io/renovatebot/renovate:35.69.3"
   ```

## 🔧 Extending Tasks

You can extend these tasks in your own Taskfile by importing this template and
overriding or adding new tasks:

```yaml
version: "3"

includes:
  renovate:
    taskfile: ./renovate.yml
    optional: true

tasks:
  # Add a convenient wrapper for a specific repository
  renovate-my-repo:
    cmds:
      - export REPOSITORY="myorg/myrepo"
      - GITHUB_TOKEN=$(gh auth token) task renovate:renovate-docker-debug

  # Run Renovate with custom settings
  renovate-custom:
    cmds:
      - task: renovate:renovate-docker-debug
        vars:
          REPOSITORY: "{{.REPO}}"
          LOG_LEVEL: "info"
          FORK: "true"
          CONFIG_FILE_PATH: "custom/path/renovate.json5"
```

## 🔍 Important Notes

- Docker must be running before using these tasks
- GitHub CLI (`gh`) must be authenticated (`gh auth login`)
- The repository must be accessible with your GitHub token
- Cache files are stored in the `.cache` directory
- Logs are saved to `renovate-debug.log` by default
- The Docker container runs as the current user to avoid permission issues
- Use `TASK_X_REMOTE_TASKFILES=1` when including these tasks from remote sources
