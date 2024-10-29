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
task terraform:check-terraform
```

### check-terragrunt

Validates that Terragrunt is installed on your system.

```bash
task terraform:check-terragrunt
```

### terragrunt-apply

Applies Terraform configurations using Terragrunt. Can be used for specific
modules or all modules.

**Default apply (all modules):**

```bash
task terraform:terragrunt-apply
```

**Apply specific module:**

```bash
task terraform:terragrunt-apply \
    DEPLOYMENT=mydeploy \
    ENV=dev \
    REGION=us-east-2 \
    MODULE=terraform-mydeploy-module
```

### terragrunt-destroy

Destroys Terraform infrastructure using Terragrunt. Can target specific modules
or all modules.

**Default destroy (all modules):**

```bash
task terraform:terragrunt-destroy
```

**Destroy specific module:**

```bash
task terraform:terragrunt-destroy \
    DEPLOYMENT=mydeploy \
    ENV=dev \
    REGION=us-east-2 \
    MODULE=terraform-mydeploy-module
```

### run-terratest

Runs Terratest infrastructure tests. Supports customizable timeout and destroy behavior.

**Basic usage:**

```bash
export TASK_X_REMOTE_TASKFILES=1 && task terraform:run-terratest -y
```

**With custom timeout and destroy flag:**

```bash
export TASK_X_REMOTE_TASKFILES=1 && task terraform:run-terratest TIMEOUT=30m DESTROY=false -y
```

### validate

Runs `terraform validate` to ensure configurations are syntactically correct.

```bash
task terraform:validate
```

### format

Formats Terraform code using `terraform fmt -recursive`.

```bash
task terraform:format
```

## 📝 Example Usage

1. **Running Terratest without destroying infrastructure:**

```bash
export TASK_X_REMOTE_TASKFILES=1 && task terraform:run-terratest DESTROY=false
```

1. **Applying a specific module in development:**

```bash
export TASK_X_REMOTE_TASKFILES=1 && task terraform:terragrunt-apply \
  DEPLOYMENT=mydeploy \
  ENV=dev \
  REGION=us-east-2 \
  MODULE=terraform-mydeploy-module
```

1. **Format and validate Terraform code:**

```bash
export TASK_X_REMOTE_TASKFILES=1 && task terraform:format
export TASK_X_REMOTE_TASKFILES=1 && task terraform:validate
```

## 🔍 Important Notes

- Always ensure you have the remote taskfiles enabled by setting `TASK_X_REMOTE_TASKFILES=1`
- The terragrunt tasks expect a specific directory structure: `infra/<deployment>/<env>/<region>/<module>/terragrunt.hcl`
- Terratest requires a `test` directory with Go files and proper module initialization
- All tasks that modify infrastructure (apply/destroy) use `-auto-approve` and
  `-lock=false` flags by default
