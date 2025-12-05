# 🎭 Ansible Taskfile Tasks

This directory contains reusable Taskfile tasks for Ansible development
operations, including Molecule testing, linting, changelog generation, and
release automation.

## 📋 Prerequisites

You must have the following installed for full functionality:

- [Task](https://taskfile.dev) (`brew install go-task/tap/go-task`)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [Molecule](https://molecule.readthedocs.io/en/latest/installation.html)
- [act](https://github.com/nektos/act) (for GitHub Actions local testing)
- [jq](https://stedolan.github.io/jq/download/) (for JSON processing)
- [antsibull-changelog](https://github.com/ansible-community/antsibull-changelog)
- [yq](https://github.com/mikefarah/yq) (for YAML editing)
- Docker (required for Molecule and act)

## 🎯 Available Tasks

### Quick Reference

**Collection Development:**

- `build-collection` - Build collection artifact with cleanup
- `install-collection` - Install collection from tarball
- `build-install-collection` - **Recommended:** Build + install in one command
- `uninstall-collection` - Remove installed collection

**Testing & Quality:**

- `run-molecule-tests` - Run Molecule tests for roles
- `run-molecule-action` - Run Molecule workflow locally via act
- `lint-ansible` - Run Ansible Lint
- `ping` - Cross-platform Ansible ping

**Changelog & Versioning:**

- `gen-changelog` - Generate changelog for release
- `changelog-lint` - Lint changelog
- `changelog-release` - Generate changelog release
- `update-galaxy-version` - Update version in galaxy.yml

**Utilities:**

- `check-ansible` - Validate Ansible installation

---

### changelog-lint

Lints the changelog using `antsibull-changelog` to ensure proper structure.

```bash
task changelog-lint
```

### changelog-release

Generates a changelog release for the specified version.
**Requires:** `NEXT_VERSION` variable.
**Automatically runs:** `update-galaxy-version`.

```bash
task changelog-release NEXT_VERSION=1.0.0
```

### check-ansible

Checks if Ansible is installed; errors out with instructions for installation
if not.

```bash
task check-ansible
```

### build-collection

Builds an Ansible collection artifact with automatic cleanup of problematic directories.

**What it does:**

- Removes old `*.tar.gz` build artifacts
- Cleans up `.ansible/` directories from roles (prevents recursive nesting issues)
- Builds the collection with `--force` flag
- Uses incremental builds (only rebuilds when sources change)

**Sources tracked:**

- `galaxy.yml`
- `roles/**/*`
- `plugins/**/*`
- `meta/**/*`

**Optional Variables:**

- `BUILD_DIR`: Output directory for tarball (default: `.`)

```bash
task ansible:build-collection

# Custom output directory
task ansible:build-collection BUILD_DIR=dist
```

**Note:** Requires `galaxy.yml` in current directory.

### install-collection

Installs an Ansible collection from a tarball using `ansible-galaxy`.

**Optional Variables:**

- `COLLECTION_PATH`: Installation path (default: `~/.ansible/collections`)
- `TARBALL`: Path to tarball (auto-detects latest if not specified)

```bash
# Auto-detect latest tarball
task ansible:install-collection

# Specify tarball
task ansible:install-collection TARBALL=cowdogmoo-workstation-1.0.0.tar.gz

# Custom installation path
task ansible:install-collection COLLECTION_PATH=./.ansible/collections
```

### build-install-collection

**Recommended for local development.** Builds and installs an Ansible collection
in one command with full cleanup. This is the primary task you should use to
avoid recursive `.ansible/` directory issues during development.

**What it does:**

1. Cleans old build artifacts
2. Removes `.ansible/` directories from roles
3. Builds collection
4. Installs to specified path (uses `--force` to overwrite)
5. Optionally installs globally
6. Cleans up tarball unless `KEEP_TARBALL=true`

**Optional Variables:**

- `COLLECTION_PATH`: Installation path (default: `~/.ansible/collections`)
- `INSTALL_GLOBAL`: Set to `true` to also install to `~/.ansible/collections`
- `KEEP_TARBALL`: Set to `true` to keep the tarball after installation

```bash
# Basic usage (installs to ~/.ansible/collections)
task ansible:build-install-collection

# Install to project-local collections
task ansible:build-install-collection COLLECTION_PATH=./.ansible/collections

# Install both locally and globally
task ansible:build-install-collection COLLECTION_PATH=./.ansible/collections INSTALL_GLOBAL=true

# Keep tarball after installation
task ansible:build-install-collection KEEP_TARBALL=true
```

**Example:** This replaces manual commands like:

```bash
# Old way (manual)
ansible-galaxy collection build --force && \
ansible-galaxy collection install cowdogmoo-workstation-*.tar.gz --force

# New way (automated with cleanup)
task ansible:build-install-collection
```

### uninstall-collection

Removes an installed Ansible collection. Since `ansible-galaxy` doesn't provide
an uninstall command, this task handles manual removal.

**Required Variables:**

- `NAMESPACE`: Collection namespace (e.g., `cowdogmoo`)
- `COLLECTION`: Collection name (e.g., `workstation`)

**Optional Variables:**

- `COLLECTION_PATH`: Path to collections (default: `~/.ansible/collections`)

```bash
task ansible:uninstall-collection NAMESPACE=cowdogmoo COLLECTION=workstation

# From custom path
task ansible:uninstall-collection NAMESPACE=cowdogmoo COLLECTION=workstation COLLECTION_PATH=./.ansible/collections
```

### gen-changelog

Validates your NEXT_VERSION, runs changelog lint and release steps.

**Required variables:**

- `NEXT_VERSION` — version string for the new release (e.g., `1.0.0`)

```bash
# Standard usage
task gen-changelog NEXT_VERSION=1.0.0

# With remote task inclusion
TASK_X_REMOTE_TASKFILES=1 NEXT_VERSION=2.0.0 task ansible:gen-changelog
```

**Example output:**

```bash
task: [ansible:gen-changelog] Generating changelog for release 2.0.0
task: [ansible:changelog-lint] antsibull-changelog lint
task: [ansible:changelog-release] antsibull-changelog release --version $NEXT_VERSION
```

### lint-ansible

Runs Ansible Lint with project configuration from
`.hooks/linters/ansible-lint.yaml`, for fast feedback.

```bash
task lint-ansible
```

### ping

Performs a robust, cross-platform Ansible ping. Supports:

- Explicit OS selection (`OS_TYPE=windows`, `linux`)
- Automatic OS detection (`OS_TYPE=auto`, the default)
- Custom host/inventory/grouping via `INVENTORY` and `HOSTS`
- Smart mktemp fallback (Linux/macOS)
- Auto-detected targets (Windows/Linux/macOS), including localhost

**Optional Variables:**

- `INVENTORY`: Ansible inventory file path (required)
- `HOSTS`: Target hosts or group (default: `all`)
- `OS_TYPE`: Override OS autodetection (`windows`, `linux`, or `auto`)
- `DEBUG`: Set to `true` for verbose output

**Usage Examples:**

```bash
# Ping all (auto-detect)
task ping INVENTORY=hosts.ini

# Explicitly windows hosts
task ping INVENTORY=hosts.ini OS_TYPE=windows

# A group, verbose
task ping INVENTORY=hosts.ini HOSTS=datacenter DEBUG=true
```

Sample playbook automatically used for detection:

```yaml
---
- hosts: all
  gather_facts: yes
  tasks:
    - name: Ping Windows hosts
      win_ping:
      when: ansible_facts.os_family == 'Windows'
    - name: Ping macOS hosts
      ping:
      when: ansible_facts['system'] == 'Darwin'
    - name: Ping Linux hosts
      ping:
      when: ansible_facts['system'] == 'Linux'
```

**Sample Output:**

```bash
Auto-detecting host types and pinging each…
===== PING RESULTS =====
PLAY [all] *********************************************************************
TASK [Gathering Facts] ********************************************************
ok: [host1-ubuntu]
ok: [host2-mac]
...
```

**Notes:**

- Uses `ANSIBLE_PYTHON_INTERPRETER=auto_silent` unless `DEBUG=true`
- Mixed environments supported (Windows+Linux+macOS)
- Temporary playbooks are cleaned up after run

### run-molecule-action

Runs the Molecule workflow GitHub Action _locally_ via `act`, supporting:

- Full suite, specific role, or specific playbook testing
- ARM64 support on macOS (transparently set)

**Optional Variables:**

- `ROLE`: Only test a specific role (folder name)
- `PLAYBOOK`: Only test a specific playbook

```bash
# Run all tests
task run-molecule-action

# Just a role
task run-molecule-action ROLE=auth

# Just a playbook
task run-molecule-action PLAYBOOK=setup_workstation
```

**Workflow requirements:**
Requires `.github/workflows/molecule.yaml` (see [workstation-collection example](https://github.com/CowDogMoo/ansible-collection-workstation/blob/main/.github/workflows/molecule.yaml),
[arsenal-collection example](https://github.com/l50/ansible-collection-arsenal/blob/main/.github/workflows/molecule.yaml))

### run-molecule-tests

Runs `molecule test` for one or more roles in the `roles/` directory. Supports
testing individual roles, multiple roles, or all roles at once. Each role's
output is logged to a separate file in the `logs/` directory.

**Optional Variables:**

- `ROLES`: Comma-separated list of role names, or `*` for all roles (default: `*`)

**Usage Examples:**

```bash
# Run tests for all roles (default)
task run-molecule-tests

# Run tests for all roles explicitly
task run-molecule-tests ROLES="*"

# Run test for a single role
task run-molecule-tests ROLES="logging"

# Run tests for multiple specific roles
task run-molecule-tests ROLES="logging,asdf,user_setup"
```

**Output:**

- Individual log files per role: `logs/molecule_tests_<role_name>.log`
- Clear visual separation between role tests in console output
- Fails fast if any role test fails

**Example Output:**

````bash
Running molecule tests for specified roles: logging,asdf
=========================================
Testing role: logging
=========================================
--> Test matrix

└── default
    ├── dependency
    ├── cleanup
    ├── destroy
    ├── syntax
    ├── create
    ├── prepare
    ├── converge
    ├── idempotence
    ├── side_effect
    ├── verify
    ├── cleanup
    └── destroy
...
=========================================
Testing role: asdf
=========================================
...
=========================================
All molecule tests completed successfully!
Logs available in: logs/

### update-galaxy-version

Updates the `version` field in `galaxy.yml` (if present) to match NEXT_VERSION.

**Requires:**

- `NEXT_VERSION` environment variable
- `galaxy.yml` file must exist

```bash
task update-galaxy-version NEXT_VERSION=1.0.0
````

If `galaxy.yml` is not present, the step is skipped.

---

## 🔍 Important Notes

- All tasks use preconditions for validation with clear error messages
- Collection tasks automatically clean `.ansible/` directories to prevent recursive nesting
- `build-collection` uses incremental builds (only rebuilds when sources change)
- All Molecule output stored in `logs/molecule_tests.log`
- `run-molecule-action` supports macOS ARM64 containers automatically
- Docker containers are cleaned up between test runs
- **All changelog-related tasks require the `NEXT_VERSION` variable**
- **Use `build-install-collection` for local development** to avoid manual cleanup

## 🔄 Typical Workflow

When working on an Ansible collection that uses these tasks:

```bash
# Make changes to your collection
vim roles/myapp/tasks/main.yml

# Build and install to test locally
task ansible:build-install-collection COLLECTION_PATH=./.ansible/collections

# Run tests
task ansible:run-molecule-tests

# Lint
task ansible:lint-ansible

# When ready to release
task ansible:gen-changelog NEXT_VERSION=1.2.3
```

## 🔧 Importing Tasks

Include from another Taskfile like this:

```yaml
version: "3"
includes:
  ansible: "./path/to/ansible/Taskfile.yaml"
tasks:
  dev:
    desc: Build and install collection for local development
    cmds:
      - task: ansible:build-install-collection
        vars:
          COLLECTION_PATH: ./.ansible/collections

  test-and-release:
    cmds:
      - task: ansible:run-molecule-tests
      - task: ansible:gen-changelog
        vars: { NEXT_VERSION: 1.0.0 }
      - task: ansible:lint-ansible
```

## 📜 License

MIT License. See [LICENSE](LICENSE) file for details.
