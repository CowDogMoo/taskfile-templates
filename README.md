# Taskfile Templates Repository

This repository contains reusable Taskfile templates that can be included in
other repositories to standardize and simplify common tasks like running
pre-commit hooks, generating changelogs, and more.

## Getting Started

### 1. Add the Remote Taskfile

To use the Taskfile templates in your project, simply include the remote
Taskfile in your project’s `Taskfile.yaml`.

For example, to include the `pre-commit` Taskfile:

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

This will automatically pull in the `pre-commit` tasks from the remote
repository and make them available in your project.

### 2. Enable remote taskfiles

See the [Taskfile documentation](https://taskfile.dev/experiments/remote-taskfiles/)
for more information.

```bash
export TASK_X_REMOTE_TASKFILES=1
```

### 3. Use the Tasks

Once you’ve included the Taskfile in your project, you can start using the
tasks defined in it. For example:

```bash
task pre-commit:update-hooks
task pre-commit:clear-cache
task pre-commit:run-hooks
```

These commands will run the respective tasks from the included Taskfile.

## Available Taskfiles

### Pre-Commit Tasks

The `pre-commit` Taskfile provides tasks to update, clear the cache, and run
all pre-commit hooks locally.

## Contributing

If you have additional tasks that you think would be useful for others, feel
free to submit a pull request. Ensure that your Taskfiles are well-documented
and follow the structure of the existing files.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE)
file for details.
