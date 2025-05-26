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

Runs `molecule test` for **all roles** in `roles/`, logs output to `logs/molecule_tests.log`.

```bash
task run-molecule-tests
```

### update-galaxy-version

Updates the `version` field in `galaxy.yml` (if present) to match NEXT_VERSION.

**Requires:**

- `NEXT_VERSION` environment variable
- `galaxy.yml` file must exist

```bash
task update-galaxy-version NEXT_VERSION=1.0.0
```

If `galaxy.yml` is not present, the step is skipped.

---

## 🔍 Important Notes

- All tasks provide error handling
- All Molecule output stored in `logs/molecule_tests.log`
- `run-molecule-action` supports macOS ARM64 containers automatically
- Docker containers are cleaned up between test runs
- **All changelog-related tasks require the `NEXT_VERSION` variable**

## 🔧 Importing Tasks

Include from another Taskfile like this:

```yaml
version: "3"
includes:
  ansible: "./path/to/ansible/Taskfile.yaml"
tasks:
  test-and-release:
    cmds:
      - task: ansible:run-molecule-tests
      - task: ansible:gen-changelog
        vars: { NEXT_VERSION: 1.0.0 }
      - task: ansible:lint-ansible
```

## 📝 Directory Structure

```bash
.
├── ansible.cfg
├── roles/
│   ├── asdf/
│   ├── user_setup/
│   └── zsh_setup/
├── playbooks/
│   └── workstation/
├── logs/
│   └── molecule_tests.log
├── .hooks/
│   └── linters/
│       └── ansible-lint.yaml
├── galaxy.yml
├── README.md
└── Taskfile.yaml
```

## 🤝 Contributing

1. Fork the repository
1. Create a new branch for your changes
1. Ensure tests pass: `task run-molecule-tests`
1. Lint changes: `task lint-ansible`
1. Update changelog: `task gen-changelog NEXT_VERSION=x.y.z`
1. Submit as a PR

## 📜 License

MIT License. See [LICENSE](LICENSE) file for details.
