# 📦 Taskfile Templates Repository

This repository contains reusable **Taskfile templates** to standardize and
simplify common tasks like running pre-commit hooks, generating changelogs,
creating GitHub releases, and more.

---

## 🚀 Getting Started

### 1. Add the Remote Taskfiles

To use the Taskfile templates in your project, include the remote Taskfiles in
your project’s `Taskfile.yaml`:

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

Once you’ve included the Taskfiles in your project, you can use the tasks like so:

```bash
task pre-commit:update-hooks
task pre-commit:clear-cache
task pre-commit:run-hooks
task github:create-release
```

---

## 📂 Available Taskfiles

### 🔧 Pre-Commit Tasks

The `pre-commit` Taskfile provides the following tasks:

- **Update Hooks:** Automatically updates pre-commit hooks.
- **Clear Cache:** Clears the pre-commit cache.
- **Run Hooks:** Runs all pre-commit hooks locally.

### ☁️ Terraform Tasks

- **check-terraform:** Validate that Terraform is installed.
- **check-terragrunt:** Validate that Terragrunt is installed.
- **terragrunt-apply:** Apply a specific module in a specific environment using Terragrunt.
- **terragrunt-destroy:** Destroy a specific module in a specific environment
  using Terragrunt.
- **run-terratest:** Run Terratest for infrastructure testing.
- **validate:** Run Terraform validate to ensure configuration is syntactically correct.
- **format:** Format Terraform code.

### 🏷️ GitHub Tasks

The `github` Taskfile provides the following tasks:

- **add-git-submodule:** Add a new submodule to the repository.
- **check-gh-cli:** Validate that the `gh` CLI is installed.
- **create-release:** Creates a new release on GitHub using the `gh` CLI.
- **remove-git-submodules:** Remove specified submodule(s) from the repository

---

## 🤝 Contributing

Have an idea for a useful task? Submit a pull request! Ensure your Taskfiles
are well-documented and follow the structure of existing files.

---

## 📜 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file
for details.
