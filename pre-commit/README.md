# 🎯 Pre-commit Taskfile Templates

This directory contains reusable Taskfile templates for pre-commit operations,
including hook installation, cache management, and automated code quality
checks.

## 📋 Prerequisites

- [Task](https://taskfile.dev/) installed (`brew install go-task/tap/go-task`)
- [pre-commit](https://pre-commit.com/) installed (`pip install pre-commit` or
  `brew install pre-commit`)
- Python 3.6+ (for pre-commit)
- Git repository initialized

## 🔐 Configuration

The taskfile requires the following configuration:

1. Ensure you have a `.pre-commit-config.yaml` file in your repository root
   with your desired hooks configured

1. (Optional) Create a `.env` file for any environment-specific settings:

   ```bash
   # Example: Skip specific hooks during local development
   SKIP=pylint,mypy
   ```

## 🎯 Available Tasks

### install-pc-hooks

Installs pre-commit hooks into your git repository. This enables automatic hook
execution on `git commit`.

```bash
task install-pc-hooks
```

### clear-cache

Clears the pre-commit cache. Useful when hooks are misbehaving or after
updating hook versions.

```bash
task clear-cache
```

### run-hooks

Runs all configured pre-commit hooks against all files in the repository. Shows
differences on failure.

```bash
task run-hooks
```

### run-pre-commit

Complete pre-commit workflow: installs hooks, updates them, clears cache, and
runs all hooks. This is the recommended task for CI/CD pipelines or fresh
setups.

```bash
task run-pre-commit
```

### update-hooks

Updates all pre-commit hooks to their latest versions as specified in the configuration.

```bash
task update-hooks
```

## 📝 Example Usage

1. **Initial setup for a new developer:**

   ```bash
   task install-pc-hooks
   ```

1. **Run all hooks before committing:**

   ```bash
   task run-hooks
   ```

1. **Full pre-commit workflow (recommended for CI):**

   ```bash
   task run-pre-commit
   ```

1. **Update hooks to latest versions:**

   ```bash
   task update-hooks
   ```

1. **Fix hook issues by clearing cache:**

   ```bash
   task clear-cache
   task run-hooks
   ```

## 🔧 Extending Tasks

You can extend these tasks in your own Taskfile by importing this template and
adding project-specific tasks. Here's an example:

```yaml
version: "3"

includes:
  precommit:
    taskfile: ./pre-commit.yml
    optional: true

tasks:
  # Override or extend existing tasks
  lint:
    deps: [precommit:run-hooks]
    cmds:
      - echo "Additional linting completed"

  # Add new tasks that use the base tasks
  ci-check:
    desc: "Run all CI checks"
    cmds:
      - task: precommit:run-pre-commit
      - task: test
      - task: build

  # Run specific hooks only
  format:
    desc: "Run only formatting hooks"
    cmds:
      - pre-commit run black --all-files
      - pre-commit run isort --all-files
```

## 🔍 Important Notes

- Pre-commit hooks run automatically on `git commit` after installation
- Use `git commit --no-verify` to bypass hooks in emergencies (not recommended)
- The `run-pre-commit` task includes `install-pc-hooks` as a dependency to
  ensure hooks are always installed
- Hook configurations are stored in `.pre-commit-config.yaml`
- Individual hooks can be skipped using the `SKIP` environment variable:
  `SKIP=flake8 git commit`
- The `--show-diff-on-failure` flag helps identify what changes hooks would make
- Regular hook updates (`task update-hooks`) ensure you're using the latest
  security fixes and features

## 🚀 Best Practices

1. **Run hooks before pushing:** Always run `task run-hooks` before pushing code
1. **Keep hooks updated:** Regularly run `task update-hooks` to get the latest versions
1. **Clear cache when needed:** If hooks behave unexpectedly, run
   `task clear-cache`
1. **CI/CD Integration:** Use `task run-pre-commit` in your CI pipeline for
   consistency
1. **Team consistency:** Ensure all team members run `task install-pc-hooks`
   after cloning the repository
