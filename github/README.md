# 🔄 GitHub Taskfile Templates

This directory contains reusable Taskfile templates for GitHub operations,
including managing submodules, handling releases, merging pull requests, and
more.

## 📋 Prerequisites

- [GitHub CLI](https://cli.github.com) installed
- [Node.js](https://nodejs.org) installed (required for some GitHub CLI extensions)
- [jq](https://stedolan.github.io/jq/) installed (for JSON processing)
- Task installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### check-gh-cli

Validates that GitHub CLI is installed on your system.

```bash
task check-gh-cli
```

### add-git-submodules

Adds multiple GitHub repositories as submodules based on a naming pattern.

```bash
task add-git-submodules ORG=myorg REGEXP="^prefix-.*" TARGET_DIR=modules
```

Required variables:

- `ORG`: GitHub organization name
- `REGEXP`: Pattern to match repository names
- `TARGET_DIR`: Directory where submodules will be added

### sync-submodules

Initializes and updates submodules idempotently (safe to run multiple times).

```bash
task sync-submodules
```

### remove-git-submodules

Removes one or more git submodules and cleans up related files.

```bash
task remove-git-submodules TARGET_DIR=modules REGEXP="^prefix-.*"
```

Required variables:

- `TARGET_DIR`: Directory containing the submodules
- `REGEXP`: Pattern to match submodule names to remove

### create-release

Creates a GitHub release using a changelog file.

```bash
task create-release NEXT_VERSION=1.0.0
```

Required variables:

- `NEXT_VERSION`: Version number for the release

### create-release-with-notes

Creates a GitHub release with automatically generated release notes.

```bash
task create-release-with-notes NEXT_VERSION=1.0.0
```

Required variables:

- `NEXT_VERSION`: Version number for the release

### merge-pull-requests

Squash and merges all eligible pull requests, closing those with conflicts or
failing checks.

```bash
task merge-pull-requests
```

### diff-changes

Shows git diff while excluding specified patterns.

```bash
task diff-changes BASE_BRANCH=main EXCLUDES="test,.github,.terraform*,*.md"
```

Required variables:

- `BASE_BRANCH`: Branch to compare against
- `EXCLUDES`: Comma-separated list of patterns to exclude

### pull-git-repos

Pulls updates for all git repositories in the current directory.

```bash
task pull-git-repos
```

## 📝 Example Usage

1. **Adding multiple submodules matching a pattern:**

```bash
task add-git-submodules ORG=mycompany REGEXP="^api-.*" TARGET_DIR=apis
```

1. **Synchronizing all submodules after cloning a repository:**

```bash
task sync-submodules
```

1. **Creating a release with auto-generated notes:**

```bash
task create-release-with-notes NEXT_VERSION=1.2.0
```

1. **Removing submodules matching a pattern:**

```bash
task remove-git-submodules TARGET_DIR=modules REGEXP="^api-.*"
```

1. **Viewing changes excluding certain patterns:**

```bash
task diff-changes BASE_BRANCH=main EXCLUDES="test,docs/*,*.md"
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
      - task: gh:create-release-with-notes
        vars:
          NEXT_VERSION: 1.0.0
      - task: gh:merge-pull-requests
```

## 🔍 Important Notes

- The GitHub CLI must be authenticated (`gh auth login`) before using these
  tasks
- Some tasks require the `gh-submodule-add` extension, which will be
  automatically installed
- All tasks have appropriate error handling and validation
- Use `DEBUG=true` with tasks to enable additional debugging output
- The `merge-pull-requests` task will only merge PRs that have passing checks
  and no conflicts
- Release tasks expect a proper changelog structure in the repository
- The `sync-submodules` task is idempotent and can be run multiple times without
  issues
