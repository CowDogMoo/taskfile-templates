# 🌩️ AWS Taskfile Templates

This directory contains reusable Taskfile templates for AWS operations,
including listing EC2 instances, cleaning up KMS keys, managing EFS file
systems, deleting S3 buckets, and cleaning up IAM resources.

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

### cleanup-bucket

Deletes S3 buckets that match a specified prefix pattern.

```bash
task cleanup-bucket BUCKET_PREFIX=attack-box-bucket- REGION=us-west-1 DRY_RUN=true
```

Optional variables:

- `REGION`: AWS region where the buckets are located (defaults to 'us-west-1')
- `BUCKET_PREFIX`: Prefix pattern to match bucket names (defaults to 'attack-box-bucket-')
- `DRY_RUN`: When set to 'true', lists buckets without deleting them (defaults
  to 'false')

This task:

1. Lists all S3 buckets matching the specified name prefix
1. Empties each bucket's contents (required before deletion)
1. Deletes the buckets
1. Provides detailed progress and error reporting

### cleanup-iam

Cleans up IAM resources (roles and policies) that match a specified keyword.

```bash
task cleanup-iam KEYWORD=test-role DRY_RUN=true
```

Required variables:

- `KEYWORD`: Keyword to match IAM resource names

Optional variables:

- `REGION`: AWS region to use (defaults to `AWS_DEFAULT_REGION` environment
  variable or 'us-east-2')
- `DRY_RUN`: When set to 'true', lists resources without deleting them
  (defaults to 'false')

This task:

1. Searches for IAM roles and policies containing the specified keyword
1. For roles: removes from instance profiles, detaches policies, deletes inline
   policies, then deletes the role
1. For policies: detaches from roles, then deletes the policy
1. Provides comprehensive logging of all actions taken

## 📝 Example Usage

1. **Listing all running EC2 instances in a specific region:**

   ```bash
   task list-running-instances REGION=us-west-2
   ```

1. **Finding and scheduling deletion of unused KMS keys with a custom deletion window:**

   ```bash
   task cleanup-kms REGION=us-east-1 DAYS_TO_DELETION=14
   ```

1. **Deleting an EFS file system:**

   ```bash
   task cleanup-efs FILE_SYSTEM_ID=fs-0123456789abcdef REGION=us-east-2
   ```

1. **Listing S3 buckets matching a pattern without deleting (dry run):**

   ```bash
   task cleanup-bucket BUCKET_PREFIX=test-bucket- DRY_RUN=true
   ```

1. **Identifying IAM resources to clean up without deleting (dry run):**

   ```bash
   task cleanup-iam KEYWORD=temporary-access DRY_RUN=true
   ```

1. **Cleaning up all IAM resources containing a specific keyword:**

   ```bash
   task cleanup-iam KEYWORD=test-automation
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
- The `cleanup-bucket` task will permanently delete matching buckets and their
  contents
- The `cleanup-iam` task handles the complex dependencies between IAM resources
  before deletion
- All tasks include appropriate error handling
- Use caution with deletion tasks as they can permanently remove AWS resources
- Consider using `AWS_PROFILE` environment variable to specify different AWS profiles
- Always use `DRY_RUN=true` first with deletion tasks to verify which resources
  will be affected
