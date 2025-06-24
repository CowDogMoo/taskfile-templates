# 📦 Taskfile Templates Repository

This repository contains reusable **Taskfile templates** to standardize and
simplify common tasks like running pre-commit hooks, generating changelogs,
creating GitHub releases, and more.

---

## 🚀 Getting Started

### 1. Add the Remote Taskfiles

To use the Taskfile templates in your project, include the remote Taskfiles in
your project's `Taskfile.yaml`:

```yaml
version: "3"
includes:
  pre-commit:
    taskfile: "https://raw.githubusercontent.com/CowDogMoo/taskfile-templates/main/pre-commit/Taskfile.yaml"
  github:
    taskfile: "https://raw.githubusercontent.com/CowDogMoo/taskfile-templates/main/github/Taskfile.yaml"
tasks:
  default:
    cmds:
      - task: pre-commit:update-hooks
      - task: github:create-release
```

This setup automatically pulls in both the `pre-commit` and `github` tasks from
the remote repository.

### 2. Enable Remote Taskfiles

Enable remote Taskfiles by setting this environment variable:

```bash
export TASK_X_REMOTE_TASKFILES=1
```

For more details, refer to the [Taskfile documentation](https://taskfile.dev/experiments/remote-taskfiles/).

### 3. Use the Tasks

Once you've included the Taskfiles in your project, you can use the tasks like so:

```bash
task pre-commit:update-hooks
task pre-commit:clear-cache
task pre-commit:run-hooks
task github:create-release
```

---

## 🔄 Variables and Configuration

### Environment Variables

Many of the templates support configuration through environment variables. When
you export an environment variable in your shell, Task will automatically pick
it up:

```bash
# Set environment variable
export GITHUB_TOKEN=your-token-here

# Task will use this value automatically
task github:create-release
```

### Variable Precedence

Task follows this precedence order (highest to lowest priority):

1. **CLI variables**: `task github:create-release NEXT_VERSION=1.0.0`
1. **Task-level variables**: Variables defined in the task itself
1. **Global variables**: Variables in the `vars:` section of your Taskfile
1. **Environment variables**: Variables from your shell environment

This means you can override any default configuration by passing variables directly:

```bash
# Override with CLI variable (highest priority)
task github:create-release NEXT_VERSION=2.0.0

# Or set via environment (lower priority)
export NEXT_VERSION=2.0.0
task github:create-release
```

### Template Variable Syntax

If your Taskfile uses template syntax like this:

```yaml
vars:
  MY_VAR: '{{.MY_VAR | default (env "MY_VAR") | default "fallback"}}'
```

It will check in order:

1. Task variable `.MY_VAR`
1. Environment variable `MY_VAR`
1. Default value `"fallback"`

This allows maximum flexibility in how you configure the templates.

---

## 📂 Available Taskfiles

Each taskfile template includes its own README with detailed documentation
about available tasks and configuration options. Here are the available
templates:

### 📚 Template Categories

- **[ansible/](ansible/)** - Ansible automation tasks including playbook
  execution, inventory management, and role testing
- **[aws/](aws/)** - AWS CLI operations, resource management, and deployment tasks
- **[docker/](docker/)** - Docker container and image management, compose
  operations, and registry tasks
- **[github/](github/)** - GitHub repository management, release creation, PR
  automation, and submodule handling
- **[k8s/](k8s/)** - Kubernetes deployment, service management, and cluster operations
- **[packer/](packer/)** - Packer image building, validation, and artifact management
- **[pre-commit/](pre-commit/)** - Pre-commit hook management, updates, and execution
- **[renovate/](renovate/)** - Renovate dependency update automation and configuration
- **[secrets/](secrets/)** - Secret management with support for:
  - **[local/](secrets/local/)** - Local secret storage and retrieval
  - **[onepassword/](secrets/onepassword/)** - 1Password integration for team
    secret management
- **[terraform/](terraform/)** - Terraform/Terragrunt operations including
  apply, destroy, validation, and testing
- **[unifi/](unifi/)** - UniFi network controller management and configuration tasks

### 📖 Usage

To see available tasks for any template, check the README in the respective
directory. For example:

- View Docker tasks: [docker/README.md](docker/README.md)
- View GitHub tasks: [github/README.md](github/README.md)
- View Terraform tasks: [terraform/README.md](terraform/README.md)

Each README includes:

- Complete list of available tasks
- Required and optional variables
- Usage examples
- Configuration options

---

## 🤝 Contributing

Have an idea for a useful task? Submit a pull request! Ensure your Taskfiles
are well-documented and follow the structure of existing files.

---

## 📜 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file
for details.
