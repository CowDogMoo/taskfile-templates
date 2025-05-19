# 🌍 Terraform Taskfile Templates

This directory contains reusable Taskfile templates for **Terraform** and
**Terragrunt** operations, supporting infrastructure deployment, linting,
testing, and state management.

## 📋 Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) installed
- [Terragrunt](https://terragrunt.gruntwork.io) installed
- [Go](https://go.dev/dl/) (for running Terratest)
- [Terratest](https://terratest.gruntwork.io) in your `test` Go module
- [Task](https://taskfile.dev/#/installation)
  (`brew install go-task/tap/go-task`) installed

---

## 🎯 Available Tasks

### check-terraform

Checks if Terraform is installed on your system.

```bash
task check-terraform
```

### check-terragrunt

Checks if Terragrunt is installed on your system.

```bash
task check-terragrunt
```

### lint

Runs `terraform fmt -check -recursive` and `tflint` for all `.tf` files, in all directories.

```bash
task lint
```

### format

Applies `terraform fmt -recursive` to format code in place.

```bash
task format
```

### validate

Runs `terraform validate` in the current directory to check syntax and config correctness.

```bash
task validate
```

### terragrunt-init

Initializes one or all Terragrunt modules.
Required: `DEPLOYMENT`, `ENV`, `REGION`
Optional: `MODULE`

**All modules**:

```bash
task terragrunt-init DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Specific module**:

```bash
task terragrunt-init DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

### terragrunt-plan

Runs `terragrunt plan` (`run-all plan`) for all or a specific module.
**All modules**:

```bash
task terragrunt-plan DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Specific module**:

```bash
task terragrunt-plan DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

### terragrunt-apply

Runs `terragrunt apply` (`run-all apply`) for all or a specific module.
All apply and destroy operations are non-interactive (`-auto-approve`).

**All modules**:

```bash
task terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Specific module**:

```bash
task terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

### terragrunt-destroy

Runs `terragrunt destroy` (`run-all destroy`) for all or a specific module.

**All modules**:

```bash
task terragrunt-destroy DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Specific module**:

```bash
task terragrunt-destroy DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

### terragrunt-state-remove-all

Removes all resources from the Terraform state for a specified module or all
modules.
Required: `DEPLOYMENT`, `ENV`, `REGION`
Optional: `MODULE`

**Entire deployment**:

```bash
task terragrunt-state-remove-all DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Specific module only**:

```bash
task terragrunt-state-remove-all DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

### run-terratest

Runs Terratest integration tests in the `test/` directory.

#### Supported options (all optional)

- `TIMEOUT`: Max duration (default: `60m`)
- `DESTROY`: Destroy infra after test (default: `true`)
- `VERBOSE`: Show verbose logs (default: `false`)
- `LOG_TO_FILE`: Tee logs to a file (default: `false`)
- `LOG_PATH`: Log output file path (default: `/tmp/terratest.log`)

**Simple usage:**

```bash
task run-terratest
```

**Advanced usage:**

```bash
task run-terratest TIMEOUT=30m DESTROY=false VERBOSE=true LOG_TO_FILE=true LOG_PATH=output.log
```

---

## 📝 Example Usage

**Format and validate code:**

```bash
task format
task validate
```

**Plan and apply infrastructure:**

```bash
task terragrunt-plan DEPLOYMENT=myenv ENV=dev REGION=us-east-2
task terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Apply for a single module:**

```bash
task terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

**Remove state for a module:**

```bash
task terragrunt-state-remove-all DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

**Terratest, verbose with logs:**

```bash
task run-terratest VERBOSE=true LOG_TO_FILE=true LOG_PATH=/tmp/test.log
```

---

## ⚙️ Extending Tasks

You can import this template in your own Taskfile and add or extend tasks as desired:

```yaml
includes:
  tf:
    taskfile: ./terraform.yml

tasks:
  my-custom-task:
    cmds:
      - task: tf:terragrunt-plan
        vars:
          DEPLOYMENT: myenv
          ENV: dev
          REGION: us-west-1
```

---

## 🔍 Notes

- `DEPLOYMENT`, `ENV`, and `REGION` are required for all Terragrunt tasks.
- `MODULE` is optional, to target a specific submodule directory.
- Terragrunt commands default to non-interactive, auto-approved, and disable
  state file locking.
- Terratest assumes your tests are in a `test` subdirectory.
- `tflint` must be installed to use the `lint` task.

## 🤝 Contributing

1. Fork the repository
1. Create a new branch for your changes
1. Ensure tests pass: `task run-molecule-tests`
1. Lint changes: `task lint-ansible`
1. Update changelog: `task gen-changelog NEXT_VERSION=x.y.z`
1. Submit as a PR

## 📜 License

MIT License. See [LICENSE](LICENSE) file for details.
