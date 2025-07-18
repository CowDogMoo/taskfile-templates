# 🔄 GitHub Taskfile Templates

This directory contains reusable Taskfile templates for GitHub operations,
including managing submodules, handling releases, merging pull requests,
working with GitHub Container Registry, and more.

## 📋 Prerequisites

- [GitHub CLI](https://cli.github.com) installed
- [Docker](https://www.docker.com) installed (for GHCR tasks)
- [jq](https://stedolan.github.io/jq/) installed (for JSON processing)
- Task installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### add-git-submodules

Adds a GitHub repository as a submodule.

```bash
task add-git-submodules ORG=myorg REGEXP=my-repo TARGET_DIR=modules
```

Required variables:

- `ORG`: GitHub organization name
- `REGEXP`: Repository name (exact match)
- `TARGET_DIR`: Directory where submodule will be added

### check-gh-cli

Validates that GitHub CLI is installed on your system.

```bash
task check-gh-cli
```

### create-release

Creates a GitHub release using a changelog file or auto-generated notes.

```bash
# Using changelog file (default: changelogs/CHANGELOG.rst)
task create-release NEXT_VERSION=1.0.0

# Using custom changelog file
task create-release NEXT_VERSION=1.0.0 CHANGELOG_FILE=CHANGELOG.md

# Force auto-generated notes
task create-release NEXT_VERSION=1.0.0 AUTO_NOTES=true
```

Required variables:

- `NEXT_VERSION`: Version number for the release

Optional variables:

- `CHANGELOG_FILE`: Path to changelog file (default: `changelogs/CHANGELOG.rst`)
- `AUTO_NOTES`: Force use of auto-generated notes

### diff-changes

Shows git diff while excluding specified patterns.

```bash
task diff-changes BASE_BRANCH=main EXCLUDES="test,.github,.terraform*,*.md"
```

Required variables:

- `BASE_BRANCH`: Branch to compare against

Optional variables:

- `EXCLUDES`: Comma-separated list of patterns to exclude

### ghcr-setup

Sets up GitHub authentication for GitHub Container Registry access.

```bash
task ghcr-setup
```

### ghcr-pull-image

Pulls and tags Docker images from GitHub Container Registry.

```bash
# Pull with default settings
task ghcr-pull-image IMAGE=my-app

# Pull with custom tag
task ghcr-pull-image IMAGE=my-app TAG=v1.2.3

# Pull from custom registry/namespace
task ghcr-pull-image IMAGE=my-app REGISTRY=ghcr.io NAMESPACE=myorg
```

Required variables:

- `IMAGE`: Image name to pull

Optional variables:

- `TAG`: Image tag (defaults to value in taskfile)
- `REGISTRY`: Container registry URL
- `NAMESPACE`: Registry namespace
- `LOCAL_IMAGE`: Local image name (defaults to IMAGE)
- `LOCAL_TAG`: Local tag name (defaults to TAG)

### ghcr-push-image

Tags and pushes Docker images to GitHub Container Registry.

```bash
# Push with default settings
task ghcr-push-image IMAGE=my-app

# Push with custom tag
task ghcr-push-image IMAGE=my-app TAG=v1.2.3

# Push with additional tags
task ghcr-push-image IMAGE=my-app ADDITIONAL_TAGS="latest,stable"

# Push from custom local tag
task ghcr-push-image IMAGE=my-app LOCAL_TAG=dev
```

Required variables:

- `IMAGE`: Image name to push

Optional variables:

- `TAG`: Remote image tag (defaults to value in taskfile)
- `LOCAL_TAG`: Local tag to push from (default: `latest`)
- `ADDITIONAL_TAGS`: Comma-separated list of additional tags to push
- `REGISTRY`: Container registry URL
- `NAMESPACE`: Registry namespace

### merge-pull-requests

Squash and merges all eligible pull requests, closing those with conflicts or
failing checks.

```bash
task merge-pull-requests
```

### pull-git-repos

Pulls updates for all git repositories in the current directory.

```bash
task pull-git-repos
```

### remove-git-submodules

Removes one or more git submodules and cleans up related files.

```bash
task remove-git-submodules TARGET_DIR=modules REGEXP="^prefix-.*"
```

Required variables:

- `TARGET_DIR`: Directory containing the submodules
- `REGEXP`: Pattern to match submodule names to remove

### sync-submodules

Initializes and updates submodules idempotently (safe to run multiple times).

```bash
task sync-submodules
```

## 📝 Example Usage

1. **Adding a submodule:**

   ```bash
   task add-git-submodules ORG=mycompany REGEXP=api-service TARGET_DIR=services
   ```

1. **Synchronizing all submodules after cloning a repository:**

   ```bash
   task sync-submodules
   ```

1. **Creating a release with auto-generated notes:**

   ```bash
   task create-release NEXT_VERSION=1.2.0 AUTO_NOTES=true
   ```

1. **Removing submodules matching a pattern:**

   ```bash
   task remove-git-submodules TARGET_DIR=modules REGEXP="^api-.*"
   ```

1. **Viewing changes excluding certain patterns:**

   ```bash
   task diff-changes BASE_BRANCH=main EXCLUDES="test,docs/*,*.md"
   ```

1. **Working with GitHub Container Registry:**

   ```bash
   # First-time setup
   task ghcr-setup

   # Pull an image
   task ghcr-pull-image IMAGE=my-app TAG=v1.0.0

   # Push an image with multiple tags
   task ghcr-push-image IMAGE=my-app TAG=v1.0.0 ADDITIONAL_TAGS="latest,stable"
   ```

## 🔧 Extending Tasks

You can extend these tasks in your own Taskfile by importing this template and
overriding or adding new tasks:

```yaml
version: "3"

includes:
  gh:
    taskfile: ./github.yml
    optional: true

tasks:
  # Override or extend existing tasks
  create-release:
    deps: [gh:create-release]
    cmds:
      - echo "Additional steps after release creation..."

  # Add new tasks that use the base tasks
  release-and-merge:
    cmds:
      - task: gh:create-release
        vars:
          NEXT_VERSION: 1.0.0
      - task: gh:merge-pull-requests
```

## 🔍 Important Notes

- The GitHub CLI must be authenticated (`gh auth login`) before using these
  tasks
- GHCR tasks require Docker to be installed and running
- The `ghcr-setup` task configures GitHub authentication with the necessary
  scopes for package access
- All tasks have appropriate error handling and validation
- The `merge-pull-requests` task will only merge PRs that have passing checks
  and no conflicts
- Release tasks will use a changelog file if it exists, otherwise will generate
  notes automatically
- The `sync-submodules` task is idempotent and can be run multiple times without
  issues
- The `pull-git-repos` task skips repositories with uncommitted changes
