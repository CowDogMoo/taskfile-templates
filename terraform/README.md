# 🌍 Terraform Taskfile Templates

This directory contains reusable Taskfile templates for Terraform and
Terragrunt operations, including infrastructure deployment, testing, and
maintenance tasks.

## 📋 Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) installed
- [Terragrunt](https://terragrunt.gruntwork.io) installed
- [Go](https://go.dev/dl/) installed (for Terratest)
- [Terratest](https://terratest.gruntwork.io) installed
- Task installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### check-terraform

Validates that Terraform is installed on your system.

```bash
task check-terraform
```

### check-terragrunt

Validates that Terragrunt is installed on your system.

```bash
task check-terragrunt
```

### terragrunt-init

Initializes Terraform configurations using Terragrunt. Can be used for specific
modules or all modules.

```bash
task terragrunt-init DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

### terragrunt-apply

Applies Terraform configurations using Terragrunt. Can be used for specific
modules or all modules.

**Apply all modules:**

```bash
task terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2 -y
```

**Apply specific module:**

```bash
task --verbose terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=cruciboard -y
```

### terragrunt-destroy

Destroys Terraform infrastructure using Terragrunt. Can target specific modules
or all modules.

**Destroy all modules:**

```bash
task terragrunt-destroy DEPLOYMENT=myenv ENV=dev REGION=us-east-2 -y
```

**Destroy specific module:**

```bash
task terragrunt-destroy DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=cruciboard -y
```

### run-terratest

Runs Terratest infrastructure tests. The `run-terratest` task supports the
following variables with defaults:

- `TIMEOUT`: Default is `60m`
- `DESTROY`: Default is `true`
- `VERBOSE`: Default is `false`
- `LOG_TO_FILE`: Default is `false`
- `LOG_PATH`: Default is `/tmp/terratest.log`

When `VERBOSE` is set to `true`, it will:

- Enable Terraform debug logging (`TF_LOG=DEBUG`)
- Run tests with verbose Go testing flags

When `LOG_TO_FILE` is set to `true`, it will:

- Save Terraform logs to `./terraform.log` (if `VERBOSE` is also `true`)
- Save test output to the specified `LOG_PATH`

**Basic usage:**

```bash
task run-terratest
```

**With custom options:**

```bash
task run-terratest TIMEOUT=30m DESTROY=false VERBOSE=true LOG_TO_FILE=true
```

### validate

Runs `terraform validate` to ensure configurations are syntactically correct.

```bash
task validate
```

### format

Formats Terraform code using `terraform fmt -recursive`.

```bash
task format
```

### lint

Runs `terraform fmt` check and `tflint` on all Terraform files.

```bash
task lint
```

### changelog-init

Initializes a changelog for Terraform modules.

```bash
task changelog-init
```

### changelog-release

Updates changelog for release.

```bash
task changelog-release NEXT_VERSION=1.0.0
```

### release

Prepares and validates a Terraform release.

```bash
task release NEXT_VERSION=1.0.0
```

## 📝 Example Usage

1. **Running Terratest without destroying infrastructure:**

```bash
task run-terratest DESTROY=false
```

2. **Running Terratest with verbose output and logging to disk:**

```bash
task run-terratest DESTROY=false VERBOSE=true LOG_TO_FILE=true LOG_PATH=/tmp/output.log
```

3. **Applying infrastructure changes:**

```bash
# Apply all modules
task terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2 -y

# Apply specific module with verbose output
task --verbose terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=cruciboard -y
```

4. **Format and validate Terraform code:**

```bash
task format
task validate
```

## 🔧 Extending Tasks

You can extend these tasks in your own Taskfile by importing this template and
overriding or adding new tasks. Here's an example:

```yaml
version: "3"

includes:
  tf:
    taskfile: ./terraform.yml
    optional: true

tasks:
  # Override or extend existing tasks
  terragrunt-apply:
    deps: [tf:terragrunt-apply]
    cmds:
      - echo "Additional steps after terraform apply..."

  # Add new tasks that use the base tasks
  deploy-all:
    cmds:
      - task: tf:terragrunt-apply
        vars:
          DEPLOYMENT: myenv
          ENV: dev
          REGION: us-east-2
```

## 🔍 Important Notes

- The terragrunt tasks require the following environment variables:
  - `DEPLOYMENT`: The name of your deployment
  - `ENV`: The environment (e.g., dev, prod)
  - `REGION`: The AWS region
- The `MODULE` variable is optional and allows targeting specific modules
- All tasks that modify infrastructure (apply/destroy) use `-auto-approve` and
  `-lock=false` flags by default
- Use the `-y` flag to automatically approve task execution
- Add `--verbose` flag for detailed task execution output
- Terratest requires a `test` directory with Go files and proper module initialization
