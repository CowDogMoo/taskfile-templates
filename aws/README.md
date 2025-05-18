# 🌩️ AWS Taskfile Templates

Reusable Taskfile configurations for AWS operations—manage EC2 instances, EFS
file systems, KMS keys, S3 buckets, and IAM resources efficiently and safely.

---

## 📋 Prerequisites

Before you start, ensure you have:

- [AWS CLI](https://aws.amazon.com/cli/) installed and configured.
- [jq](https://stedolan.github.io/jq/) installed (for managing JSON data).
- [Task](https://taskfile.dev/) installed. Installation via Homebrew:

  ```bash
  brew install go-task/tap/go-task
  ```

---

## 🎯 Available Tasks

### ✅ check-aws

Check to confirm the AWS CLI is correctly installed.

```bash
task check-aws
```

---

### ✅ check-jq

Confirm the `jq` JSON processor is installed on your system.

```bash
task check-jq
```

---

### 🗑️ cleanup-bucket

Safely delete AWS S3 buckets matching a keyword.

**Example:**

```bash
# Dry run to list buckets matching keyword without deleting
task cleanup-bucket KEYWORD=test-bucket DRY_RUN=true REGION=us-east-1

# Actually delete matching buckets
task cleanup-bucket KEYWORD=test-bucket REGION=us-east-1
```

**Parameters:**

| Parameter | Required | Default                             | Description                        |
| --------- | -------- | ----------------------------------- | ---------------------------------- |
| `DRY_RUN` | ❌ No    | `false`                             | Set to `true` to preview deletions |
| `KEYWORD` | ✅ Yes   | None — must be specified            | Keyword to match bucket names      |
| `REGION`  | ❌ No    | `AWS_DEFAULT_REGION` or `us-east-2` | AWS region for bucket deletion     |

**Process:**

- Lists and empties buckets matching keyword
- Deletes buckets after emptying
- Offers detailed logging and error reporting

---

### 🗑️ cleanup-efs

Delete EFS file systems along with their mount targets.

**Example:**

```bash
task cleanup-efs FILE_SYSTEM_ID=fs-0123456789abcdef REGION=us-west-2 DRY_RUN=true
```

**Parameters:**

| Parameter        | Required | Default                             | Description                        |
| ---------------- | -------- | ----------------------------------- | ---------------------------------- |
| `DRY_RUN`        | ❌ No    | `false`                             | Set to `true` to preview deletions |
| `FILE_SYSTEM_ID` | ✅ Yes   | None — must be specified            | EFS File System ID to delete       |
| `REGION`         | ❌ No    | `AWS_DEFAULT_REGION` or `us-east-2` | AWS region for EFS deletion        |

**Process:**

- Finds and removes mount targets before the EFS filesystem
- Waits until all mount targets are fully deleted
- Deletes the file system if no mount targets remain

---

### 🛡️ cleanup-iam

Clean IAM roles and policies matching a keyword safely.

**Example:**

```bash
task cleanup-iam KEYWORD=temp-policy DRY_RUN=true
```

**Parameters:**

| Parameter | Required | Default                             | Description                                     |
| --------- | -------- | ----------------------------------- | ----------------------------------------------- |
| `DRY_RUN` | ❌ No    | `false`                             | Set to `true` to preview action before deletion |
| `KEYWORD` | ✅ Yes   | None — must be specified            | Keyword matching IAM resource names             |
| `REGION`  | ❌ No    | `AWS_DEFAULT_REGION` or `us-east-2` | AWS region to perform IAM operations            |

**Process:**

1. Identifies IAM resources (roles & policies) matching keyword.
2. Detaches and removes dependencies, such as:
   - Instance profiles from roles.
   - Attached policies.
   - Inline policies.
3. Deletes IAM roles and policies securely.

---

### 🔑 cleanup-kms

Identify and schedule deletion of unused KMS keys (keys without Name tags) or
keys matching a keyword.

**Example:**

```bash
task cleanup-kms REGION=us-west-2 DAYS_TO_DELETION=10 DRY_RUN=true
```

**Parameters:**

| Parameter          | Required | Default                                       | Description                                  |
| ------------------ | -------- | --------------------------------------------- | -------------------------------------------- |
| `DAYS_TO_DELETION` | ❌ No    | `7`                                           | Waiting period before permanent key deletion |
| `DRY_RUN`          | ❌ No    | `false`                                       | Set to `true` for a preview                  |
| `KEYWORD`          | ❌ No    | None (Keys without Name tags chosen if blank) | Keyword to find specific KMS keys            |
| `REGION`           | ❌ No    | `AWS_DEFAULT_REGION` or `us-east-2`           | AWS region for KMS operations                |

**Process:**

- Searches and identifies unused keys or matches keyword keys
- Disables keys securely before scheduling deletion
- Provides detailed error handling and reports any permissions issues

---

### 🖥️ list-running-instances

List detailed info about all currently running EC2 instances.

**Example:**

```bash
task list-running-instances REGION=us-east-1
```

**Parameters:**

| Parameter | Required | Default                             | Description                               |
| --------- | -------- | ----------------------------------- | ----------------------------------------- |
| `REGION`  | ❌ No    | `AWS_DEFAULT_REGION` or `us-east-2` | AWS region to target for EC2 descriptions |

**Output includes:**

- Instance IDs
- VPC IDs & Subnet IDs
- Public IP & Private IP addresses
- Instance Names based on "Name" tags

---

## 📝 Example Usage Scenarios

- **Preview buckets deletion:**

  ```bash
  task cleanup-bucket KEYWORD=dev-bucket DRY_RUN=true
  ```

- **Delete IAM related test resources:**

  ```bash
  task cleanup-iam KEYWORD=integration-tests
  ```

- **Schedule unused KMS keys for deletion after 2 weeks:**

  ```bash
  task cleanup-kms REGION=us-east-1 DAYS_TO_DELETION=14
  ```

---

## ⚙️ Extending & Customizing Tasks

Include and extend these tasks within your project's Taskfile:

```yaml
version: "3"

includes:
  aws:
    taskfile: ./aws.yml
    optional: true

tasks:
  custom-instance-report:
    deps: [aws:list-running-instances]
    cmds:
      - echo "Generating custom instance report..."

  cleanup-all-resources:
    cmds:
      - task: aws:cleanup-kms
        vars:
          REGION: us-west-2
      - task: aws:cleanup-efs
        vars:
          FILE_SYSTEM_ID: fs-123456789abcdef0
          REGION: us-west-2
```

---

## ⚠️ Important Notes & Precautions

- Verify AWS CLI configuration via `aws configure` set with relevant permissions.
- Dry run actions (`DRY_RUN=true`) are highly recommended before executing
  destructive tasks.
- Environment variable `AWS_PROFILE` can designate alternate AWS accounts/profiles.
- Be cautious as tasks such as `cleanup-bucket`, `cleanup-efs`, `cleanup-kms`,
  and `cleanup-iam` perform destructive operations capable of deleting data and
  AWS resources permanently. Always review before proceeding.
