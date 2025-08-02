# 🌍 Terraform Taskfile Templates

This directory contains reusable Taskfile templates for **Terraform** and
**Terragrunt** operations, supporting infrastructure deployment, linting,
testing, and state management.

## 📋 Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) installed
- [Terragrunt](https://terragrunt.gruntwork.io) installed (for Terragrunt tasks)
- [Go](https://go.dev/dl/) (for running Terratest)
- [Terratest](https://terratest.gruntwork.io) in your `test` Go module
- [Task](https://taskfile.dev/#/installation)
  (`brew install go-task/tap/go-task`) installed
- [tflint](https://github.com/terraform-linters/tflint) (for linting)

---

## 🎯 Available Tasks

### Core Terraform Tasks

#### check-terraform

Checks if Terraform is installed on your system.

```bash
task check-terraform
```

#### format

Applies `terraform fmt -recursive` to format code in place.

```bash
task format
```

#### validate

Runs `terraform validate` in the current directory to check syntax and config correctness.

```bash
task validate
```

#### lint

Runs `terraform fmt -check -recursive` and `tflint` for all `.tf` files, in all directories.

```bash
task lint
```

### Standard Terraform Workflow Tasks

#### tf-init-upgrade

Initialize Terraform with upgrade flag.

```bash
task tf-init-upgrade
```

#### plan

Run terraform plan with tfvars file.

```bash
# Default terraform.tfvars
task -y terraform:plan

# Custom tfvars file
task -y terraform:plan VAR_FILE=production.tfvars

# With verbose output
task -y terraform:plan VERBOSE=true
```

#### apply

Run terraform apply with tfvars file.

```bash
# Interactive apply
task -y terraform:apply

# Auto-approve
task -y terraform:apply AUTO_APPROVE=true

# Custom tfvars file with auto-approve
task -y terraform:apply VAR_FILE=production.tfvars AUTO_APPROVE=true
```

#### destroy

Run terraform destroy with tfvars file.

```bash
# Interactive destroy
task -y terraform:destroy

# Auto-approve destroy
task -y terraform:destroy AUTO_APPROVE=true
```

#### refresh

Refresh Terraform state.

```bash
task -y terraform:refresh

# With custom tfvars
task -y terraform:refresh VAR_FILE=production.tfvars
```

### Terraform State Management

#### state-list

List resources in Terraform state.

```bash
task -y terraform:state-list
```

#### state-show

Show details of a resource in Terraform state.

```bash
task -y terraform:state-show RESOURCE=aws_instance.example
```

#### import

Import existing infrastructure into Terraform state.

```bash
task -y terraform:import RESOURCE=aws_instance.example ID=i-1234567890abcdef0

# With custom tfvars
task -y terraform:import RESOURCE=aws_instance.example ID=i-1234567890abcdef0 VAR_FILE=production.tfvars
```

### Terraform Output

#### output

Show Terraform outputs.

```bash
# Show all outputs
task -y terraform:output

# Show specific output
task -y terraform:output OUTPUT=instance_ip

# Output as JSON
task -y terraform:output FORMAT=json
```

### Composite Workflow Tasks (Recommended)

#### tf-check

Format and validate Terraform configuration.

```bash
task tf-check
```

#### tf-plan

Full Terraform plan workflow (init, format, validate, plan).

```bash
task tf-plan

# With custom tfvars
task tf-plan VAR_FILE=production.tfvars

# With verbose output
task tf-plan VERBOSE=true
```

#### tf-apply

Full Terraform apply workflow (init, format, validate, apply).

```bash
# Interactive apply
task tf-apply

# Auto-approve
task tf-apply AUTO_APPROVE=true

# Custom tfvars with auto-approve
task tf-apply VAR_FILE=production.tfvars AUTO_APPROVE=true
```

#### tf-destroy

Full Terraform destroy workflow (init, plan destroy, destroy).

```bash
# Interactive destroy
task tf-destroy

# Auto-approve
task tf-destroy AUTO_APPROVE=true
```

### Terragrunt Tasks

#### check-terragrunt

Checks if Terragrunt is installed on your system.

```bash
task check-terragrunt
```

#### terragrunt-init

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

#### terragrunt-plan

Runs `terragrunt plan` (`run-all plan`) for all or a specific module.

**All modules**:

```bash
task terragrunt-plan DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Specific module**:

```bash
task terragrunt-plan DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

#### terragrunt-apply

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

#### terragrunt-destroy

Runs `terragrunt destroy` (`run-all destroy`) for all or a specific module.

**All modules**:

```bash
task terragrunt-destroy DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Specific module**:

```bash
task terragrunt-destroy DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
```

#### terragrunt-state-remove-all

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

### Testing

#### run-terratest

Runs Terratest integration tests in the `test/` directory.

##### Supported options (all optional)

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

### Standard Terraform Workflow

**Quick apply with safety checks:**

```bash
# See what will change
task tf-plan

# Apply changes
task tf-apply AUTO_APPROVE=true
```

**Working with different environments:**

```bash
# Development
task tf-apply VAR_FILE=dev.tfvars AUTO_APPROVE=true

# Production (interactive for safety)
task tf-plan VAR_FILE=production.tfvars
task tf-apply VAR_FILE=production.tfvars
```

**State management:**

```bash
# List all resources
task -y terraform:state-list

# Import existing infrastructure
task -y terraform:import RESOURCE=unifi_firewall_rule.example ID=12345

# Show resource details
task -y terraform:state-show RESOURCE=unifi_firewall_rule.example
```

### Terragrunt Workflow

**Plan and apply infrastructure:**

```bash
task terragrunt-plan DEPLOYMENT=myenv ENV=dev REGION=us-east-2
task terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2
```

**Apply for a single module:**

```bash
task terragrunt-apply DEPLOYMENT=myenv ENV=dev REGION=us-east-2 MODULE=mymodule
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
      - task: tf:terraform-plan
        vars:
          VAR_FILE: custom.tfvars
```

---

## 🔍 Notes

- Standard Terraform tasks use `terraform.tfvars` by default, but support
  custom tfvars files via `VAR_FILE`
- `AUTO_APPROVE` flag is available for apply and destroy operations
- `VERBOSE` flag enables detailed Terraform debug logging
- Terragrunt tasks require `DEPLOYMENT`, `ENV`, and `REGION` variables
- `MODULE` is optional for Terragrunt tasks, to target a specific submodule directory
- Terragrunt commands default to non-interactive, auto-approved, and disable
  state file locking
- Terratest assumes your tests are in a `test` subdirectory
- `tflint` must be installed to use the `lint` task

## 🤝 Contributing

1. Fork the repository
1. Create a new branch for your changes
1. Ensure tests pass: `task run-molecule-tests`
1. Lint changes: `task lint-ansible`
1. Update changelog: `task gen-changelog NEXT_VERSION=x.y.z`
1. Submit as a PR

## 📜 License

MIT License. See [LICENSE](LICENSE) file for details.
