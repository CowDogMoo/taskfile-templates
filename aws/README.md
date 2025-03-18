# 🌩️ AWS Taskfile Templates

This directory contains reusable Taskfile templates for AWS operations,
including listing EC2 instances, cleaning up KMS keys, and managing EFS file
systems.

## 📋 Prerequisites

- [AWS CLI](https://aws.amazon.com/cli/) installed and configured
- [jq](https://stedolan.github.io/jq/) installed (for JSON processing)
- Task installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### check-aws

Validates that AWS CLI is installed on your system.

```bash
task check-aws
```

### check-jq

Validates that jq is installed on your system.

```bash
task check-jq
```

### list-running-instances

Lists all running EC2 instances with detailed information.

```bash
task list-running-instances REGION=us-east-2
```

Optional variables:

- `REGION`: AWS region to query (defaults to `AWS_DEFAULT_REGION` environment
  variable or 'us-east-2')

### cleanup-kms

Finds and schedules deletion of unused KMS keys (those without Name tags).

```bash
task cleanup-kms REGION=us-east-2 DAYS_TO_DELETION=7
```

Optional variables:

- `REGION`: AWS region to query (defaults to 'us-east-2')
- `DAYS_TO_DELETION`: Number of days before keys are permanently deleted
  (defaults to '7')

### cleanup-efs

Deletes an EFS file system and its mount targets.

```bash
task cleanup-efs FILE_SYSTEM_ID=fs-1234567890abcdef0 REGION=us-east-2
```

Required variables:

- `FILE_SYSTEM_ID`: ID of the EFS file system to delete

Optional variables:

- `REGION`: AWS region where the EFS file system is located (defaults to 'us-east-2')

## 📝 Example Usage

1. **Listing all running EC2 instances in a specific region:**

```bash
task list-running-instances REGION=us-west-2
```

2. **Finding and scheduling deletion of unused KMS keys with a custom deletion window:**

```bash
task cleanup-kms REGION=us-east-1 DAYS_TO_DELETION=14
```

3. **Deleting an EFS file system:**

```bash
task cleanup-efs FILE_SYSTEM_ID=fs-0123456789abcdef REGION=us-east-2
```

## 🔧 Extending Tasks

You can extend these tasks in your own Taskfile by importing this template and
overriding or adding new tasks:

```yaml
version: "3"

includes:
  aws:
    taskfile: ./aws.yml
    optional: true

tasks:
  # Override or extend existing tasks
  list-running-instances-enhanced:
    deps: [aws:list-running-instances]
    cmds:
      - echo "Additional processing of instance data..."

  # Add new tasks that use the base tasks
  cleanup-all:
    cmds:
      - task: aws:cleanup-kms
        vars:
          REGION: us-east-2
      - task: aws:cleanup-efs
        vars:
          FILE_SYSTEM_ID: fs-0123456789abcdef
          REGION: us-east-2
```

## 🔍 Important Notes

- The AWS CLI must be properly configured (`aws configure`) before using these tasks
- Appropriate IAM permissions are required for each task
- The `cleanup-kms` task only targets keys without a Name tag
- The `cleanup-efs` task will first delete all mount targets before deleting
  the file system
- All tasks include appropriate error handling
- Use caution with deletion tasks as they can permanently remove AWS resources
- Consider using `AWS_PROFILE` environment variable to specify different AWS profiles
