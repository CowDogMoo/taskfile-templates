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

This setup automatically pulls in both the `pre-commit` and `github` tasks from the remote repository.

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

### 🏷️ GitHub Tasks

The `github` Taskfile provides the following tasks:

- **Create Release:** Creates a new release on GitHub using the `gh` CLI.
  It validates that the `gh` command is installed and checks that the
  `NEXT_VERSION` environment variable is set before proceeding with the release.

---

## 🤝 Contributing

Have an idea for a useful task? Submit a pull request! Ensure your Taskfiles
are well-documented and follow the structure of existing files.

---

## 📜 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file
for details.
