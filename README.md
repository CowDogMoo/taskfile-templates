# 📦 Taskfile Templates Repository

This repository contains reusable **Taskfile templates** to standardize and
simplify common tasks like running pre-commit hooks, generating changelogs, and
more.

---

## 🚀 Getting Started

### 1. Add the Remote Taskfile

To use the Taskfile templates in your project, include the remote Taskfile in
your project’s `Taskfile.yaml`:

```yaml
version: "3"

includes:
  pre-commit:
    taskfile: "https://raw.githubusercontent.com/l50/taskfile-templates/main/pre-commit/Taskfile.yaml"

tasks:
  default:
    cmds:
      - task: pre-commit:update-hooks
```

This setup automatically pulls in the `pre-commit` tasks from the remote repository.

### 2. Enable Remote Taskfiles

Enable remote Taskfiles by setting this environment variable:

```bash
export TASK_X_REMOTE_TASKFILES=1
```

For more details, refer to the [Taskfile documentation](https://taskfile.dev/experiments/remote-taskfiles/).

### 3. Use the Tasks

Once you’ve included the Taskfile in your project, you can use the tasks like so:

```bash
task pre-commit:update-hooks
task pre-commit:clear-cache
task pre-commit:run-hooks
```

---

## 📂 Available Taskfiles

### 🔧 Pre-Commit Tasks

The `pre-commit` Taskfile provides the following tasks:

- **Update Hooks:** Automatically updates pre-commit hooks.
- **Clear Cache:** Clears the pre-commit cache.
- **Run Hooks:** Runs all pre-commit hooks locally.

---

## 🤝 Contributing

Have an idea for a useful task? Submit a pull request! Ensure your Taskfiles
are well-documented and follow the structure of existing files.

---

## 📜 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file
for details.
