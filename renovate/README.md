# 🤖 Renovate Taskfile Templates

This directory contains reusable [Taskfile](https://taskfile.dev/) templates
for running Renovate bot operations in Docker, making it easy to test and debug
Renovate configurations—locally and reliably.

---

## 📋 Prerequisites

- [Docker](https://www.docker.com/get-started) installed and running
- [GitHub CLI](https://cli.github.com/) installed and authenticated (`gh auth login`)
- [Task](https://taskfile.dev/) installed
  (`brew install go-task/tap/go-task` or see official docs)
- Node.js (including `npx`) is required for `lint`
  (`node -v` and `npx -v` should work)

---

## 🎯 Available Tasks

### check-repository-format

Validates that the repository matches `org/repo` pattern.

```sh
task check-repository-format REPOSITORY=owner/repo
```

---

### docker-debug

Runs Renovate in Docker with debugging enabled for a specific GitHub repository.

```sh
export REPOSITORY="owner/repo"
GITHUB_TOKEN=$(gh auth token) task renovate:docker-debug
```

**Required variables:**

- `REPOSITORY` (format: `owner/repo`)
- `GITHUB_TOKEN` (GitHub personal access token or `gh auth token`)

**Optional variables:**

- `LOG_LEVEL`: Logging level (default: `debug`)
- `DEBUG`: Enable or disable debug logs (`true`/`false`, default: `true`)
- `PLATFORM_COMMIT`: Allow Renovate to commit changes (default: `true`)
- `CONFIG_FILE_NAME`: Name of the Renovate config file (default: `.github/renovate.json5`)
- `CONFIG_FILE_PATH`: Path to the Renovate config file (default: `.github/renovate.json5`)
- `GIT_AUTHOR`: Author for Renovate commits (default: `Renovate Bot <bot@renovateapp.com>`)
- `PLATFORM`: Platform to use, defaults to `github`
- `ENDPOINT`: API endpoint (default: `https://api.github.com`)
- `FORK`: Whether to fork the repo (`true`/`false`, default: `false`)
- `RENOVATE_IMAGE`: Renovate Docker image (default: `ghcr.io/renovatebot/renovate:latest`)
- `LOG_FILE`: File to save logs (default: `renovate-debug.log`)
- `VALIDATE_REPO`: Verify repository existence and authentication (default: `true`)
- `BRANCH`: Branch to run from (default: `main`)
- `INIT_SUBMODULES`: Initialize git submodules (`true`/`false`, default: `true`)
- `INCLUDE_SUBMODULES`: Whether to include repository submodules
  (`true`/`false`, default: `true`)
- `CLEANUP`: Clean up credentials/files after run (`true`/`false`, default: `true`)
- `CLEANUP_ALL`: Remove all cache, not just the git repo
  (`true`/`false`, default: `false`)
- `DOCKER_EXTRA_FLAGS`: Append extra Docker flags as string

---

### lint

Validate a Renovate JSON(5) config file using the official Renovate validator.

```sh
task renovate:lint CONFIG_FILE_PATH=path/to/renovate.json5
```

(Default: `.github/renovate.json5`)

---

## 📝 Example Usage

1. **Run Renovate in debug mode for a repository:**

   ```bash
   export GITHUB_TOKEN=$(gh auth token)
   task renovate:docker-debug REPOSITORY=org/repo
   ```

1. **Run Renovate with a specific branch:**

   ```bash
   export GITHUB_TOKEN=$(gh auth token)
   task renovate:docker-debug REPOSITORY=org/repo BRANCH=feat/my-branch
   ```

1. **Use a custom Renovate config file:**

   ```bash
   export GITHUB_TOKEN=$(gh auth token)
   task renovate:docker-debug REPOSITORY=org/repo CONFIG_FILE_PATH="custom-renovate.json5"
   ```

1. **Run with a specific Renovate Docker image version:**

   ```bash
   export REPOSITORY="angular/angular"
   export GITHUB_TOKEN=$(gh auth token)
   task renovate:docker-debug REPOSITORY=org/repo RENOVATE_IMAGE="ghcr.io/renovatebot/renovate:35.69.3"
   ```

---

## 🔧 Extending Tasks

You can import and extend these tasks in your own Taskfile:

```yaml
version: "3"
includes:
  renovate:
    taskfile: ./renovate.yml
    optional: true

tasks:
  renovate-my-repo:
    cmds:
      - export REPOSITORY="myorg/myrepo"
      - GITHUB_TOKEN=$(gh auth token) task renovate:docker-debug

  renovate-custom:
    cmds:
      - task: renovate:docker-debug
        vars:
          REPOSITORY: "{{.REPO}}"
          LOG_LEVEL: "info"
          FORK: "true"
          CONFIG_FILE_PATH: "custom/path/renovate.json5"
```

_If including Taskfiles remotely, you may need `TASK_X_REMOTE_TASKFILES=1`
as an environment variable._

---

## 🔍 Important Notes

- Docker must be up and running before use
- `gh` (GitHub CLI) must be authenticated
- The GitHub token must have access to the target repository
- Temporary files/cache/logs are stored in `.cache`
- Logs are saved as `renovate-debug.log` by default
- The container runs as your current local user (avoids permission errors)
- **To test _local_ config changes for `.github/renovate.json5`, you must either:**
  1. Push your config changes to the repo first, _or_
  1. Specify your config path with `CONFIG_FILE_PATH` pointing to a local file
- Uses `ghcr.io/renovatebot/renovate:latest` by default (override with `RENOVATE_IMAGE`)
- The debug run clones the repository to a temp dir and overlays your config if specified
