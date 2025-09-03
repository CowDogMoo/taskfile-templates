# 🤖 Renovate Taskfile Templates

This directory contains reusable [Taskfile](https://taskfile.dev/) templates
for running Renovate bot operations in Docker, making it easy to test and debug
Renovate configurations locally.

---

## 📋 Prerequisites

- [Docker](https://www.docker.com/get-started) installed and running
- [GitHub CLI](https://cli.github.com/) installed and authenticated
  (`gh auth login`) or `GITHUB_TOKEN` environment variable set
- [Task](https://taskfile.dev/) installed
  (`brew install go-task/tap/go-task` or see official docs)
- Git repository (for `test-local` task)

---

## 🎯 Available Tasks

### test-local

Tests Renovate configuration locally using Docker against your current repository.

```bash
task renovate:test-local
```

**Optional variables:**

- `LOG_LEVEL`: Logging level (default: `debug`)
- `CONFIG_FILE`: Path to the Renovate config file (default: `.github/renovate.json5`)
- `LOG_FILE`: File to save logs (default: `renovate-debug.log`)
- `GITHUB_TOKEN`: GitHub personal access token (if not using `gh` CLI)

**Features:**

- Automatically detects GitHub token from `gh` CLI or `GITHUB_TOKEN`
  environment variable
- Validates that you're in a git repository
- Checks that the config file exists before running
- Saves detailed logs for debugging
- Provides helpful commands to analyze results

---

### lint

Validate a Renovate JSON(5) config file using the official Renovate validator.

```sh
task renovate:lint
```

**Optional variables:**

- `CONFIG_FILE`: Path to the Renovate config file (default: `.github/renovate.json5`)

---

## 📝 Example Usage

1. **Test Renovate configuration in your local repository:**

   ```bash
   # From your repository root
   task renovate:test-local
   ```

1. **Test with a custom config file:**

   ```bash
   task renovate:test-local CONFIG_FILE="custom-renovate.json5"
   ```

1. **Test with different log level:**

   ```bash
   task renovate:test-local LOG_LEVEL=info
   ```

1. **Validate your Renovate configuration:**

   ```bash
   task renovate:lint
   ```

1. **Validate a custom config file:**

   ```bash
   task renovate:lint CONFIG_FILE="path/to/renovate.json"
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
  test-my-config:
    cmds:
      - task: renovate:test-local
        vars:
          CONFIG_FILE: "custom/renovate.json5"
          LOG_LEVEL: "info"

  validate-all:
    cmds:
      - task: renovate:lint
      - task: renovate:test-local
```

_If including Taskfiles remotely, you may need `TASK_X_REMOTE_TASKFILES=1`
as an environment variable._

---

## 🔍 Important Notes

- Docker must be up and running before use
- The `test-local` task must be run from within a git repository
- GitHub token is automatically detected from `gh` CLI or `GITHUB_TOKEN`
  environment variable
- The `test-local` task uses `--platform=local` to test against your local repository
- Logs are saved as `renovate-debug.log` by default
- The lint task validates configuration syntax and schema
- Uses `renovate/renovate` Docker image (latest version)

### Analyzing Results

After running `test-local`, you can analyze the results:

```bash
# Check for detected package updates
grep -i 'packageFiles with updates' renovate-debug.log

# Check custom managers
grep -i 'customManager' renovate-debug.log

# Check for errors
grep -i 'error' renovate-debug.log
```
