# 🎭 Ansible Taskfile Tasks

This directory contains reusable Taskfile tasks for Ansible development
operations, including molecule testing, linting, host reachability validation,
and changelog management.

## 📋 Prerequisites

- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
  installed
- [Molecule](https://molecule.readthedocs.io/en/latest/installation.html) installed
- [act](https://github.com/nektos/act) installed (for GitHub Actions local testing)
- [jq](https://stedolan.github.io/jq/download/) installed (for JSON processing)
- [antsibull-changelog](https://github.com/ansible-community/antsibull-changelog)
  installed
- Docker installed (required for Molecule and `act`)

## 🎯 Available Tasks

### changelog-lint

Lints the changelog using `antsibull-changelog` to ensure it follows the proper format.

```bash
task changelog-lint
```

### changelog-release

Generates a changelog release for the specified version.

```bash
task changelog-release NEXT_VERSION=1.0.0
```

### gen-changelog

Generates the complete changelog for the next release, including linting and
release generation steps.

**Required Variables:**

- `NEXT_VERSION`: Version number for the release (e.g., 1.0.0)

```bash
# Standard usage
task gen-changelog NEXT_VERSION=1.0.0

# Using with remote taskfiles
TASK_X_REMOTE_TASKFILES=1 NEXT_VERSION=2.0.0 task ansible:gen-changelog
```

Example output:

```bash
task: [ansible:gen-changelog] Generating changelog for release 2.0.0
task: [ansible:changelog-lint] antsibull-changelog lint
task: [ansible:changelog-release] antsibull-changelog release --version $NEXT_VERSION
```

### ping

Performs a cross-platform ping using Ansible. This task supports:

- Explicit OS detection using the `OS_TYPE` variable (`windows`, `linux`)
- Automatic OS detection (default) using Ansible facts
- Robust output display and smart fallback for `mktemp` compatibility on Linux/macOS
- Windows, Linux, and macOS targets, including localhost

**Optional Variables:**

- `INVENTORY`: Path to your Ansible inventory file (required)
- `HOSTS`: Target hosts or group (default: `all`)
- `OS_TYPE`: Explicit OS type override (`windows`, `linux`, or `auto`)
- `DEBUG`: Set to `true` for more verbose output

#### 🔧 Usage Examples

```bash
# Auto-detect host types and ping all hosts
task ping INVENTORY=mashup.ini

# Explicitly ping only Windows hosts
task ping INVENTORY=mashup.ini OS_TYPE=windows

# Ping specific group with verbose output
task ping INVENTORY=mashup.ini HOSTS=macos DEBUG=true
```

#### 🛠 What It Does

If `OS_TYPE` is set explicitly, it uses the appropriate Ansible module:

```bash
# Example shortcut output (windows)
Using win_ping module for all (OS_TYPE=windows)
ansible all -i "mashup.ini" -m win_ping
```

Otherwise, it builds a minimal playbook to auto-detect each host's OS using facts:

```yaml
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

#### ✅ Sample Output

```bash
Auto-detecting host types and pinging each…
===== PING RESULTS =====

PLAY [all] *********************************************************************

TASK [Gathering Facts] ********************************************************
ok: [ashley-kali]
ok: [vincent-kali]
ok: [srv03]
ok: [localhost]

TASK [Ping Windows hosts] *****************************************************
ok: [srv03]
ok: [dc01]
skipping: [vincent-kali]
skipping: [localhost]

TASK [Ping macOS hosts] *******************************************************
ok: [localhost]
skipping: [srv03]
skipping: [dc01]

TASK [Ping Linux hosts] *******************************************************
ok: [vincent-kali]
ok: [ashley-kali]
skipping: [localhost]
skipping: [srv03]

PLAY RECAP ********************************************************************
ashley-kali      : ok=2  changed=0  failed=0  skipped=2
vincent-kali     : ok=2  changed=0  failed=0  skipped=2
srv03            : ok=2  changed=0  failed=0  skipped=2
localhost        : ok=2  changed=0  failed=0  skipped=2
```

#### 🧠 Notes

- Uses `ANSIBLE_PYTHON_INTERPRETER=auto_silent` unless `DEBUG=true`
- Works even with mixed environments (e.g., Windows DCs + Linux attackers +
  local macOS)
- Cleans up temporary playbooks after execution

Let me know if you want a table view of variables or more output formatting examples.

### run-molecule-action

Runs GitHub Actions Molecule workflow locally using `act`. Useful for testing
roles and playbooks prior to pushing.

**Optional Variables:**

- `ROLE`: Name of the specific role to test
- `PLAYBOOK`: Name of the specific playbook to test

```bash
# Run all tests
task run-molecule-action

# Test a specific role
task run-molecule-action ROLE=zsh_setup

# Test a specific playbook
task run-molecule-action PLAYBOOK=workstation
```

See example workflows:

- [workstation-collection/molecule.yaml](https://github.com/CowDogMoo/ansible-collection-workstation/blob/main/.github/workflows/molecule.yaml)
- [arsenal-collection/molecule.yaml](https://github.com/l50/ansible-collection-arsenal/blob/main/.github/workflows/molecule.yaml)

### run-molecule-tests

Runs Molecule tests for all roles in the collection sequentially and logs the output.

```bash
task run-molecule-tests
```

## 🔍 Important Notes

- All tasks include proper error handling and logging to `logs/molecule_tests.log`
- Molecule tests rely on the local `ansible.cfg` configuration
- `ping` task includes robust OS detection logic (Windows/macOS/Linux)
- `run-molecule-action` supports macOS ARM64 compatibility automatically
- Docker containers are cleaned up between test runs
- Changelog generation requires explicit `NEXT_VERSION`

## 🔧 Importing Tasks

Include these tasks in your own Taskfile:

```yaml
version: "3"
includes:
  ansible: "./path/to/ansible/Taskfile.yaml"
tasks:
  test-and-release:
    cmds:
      - task: ansible:run-molecule-tests
      - task: ansible:gen-changelog
        vars:
          NEXT_VERSION: 1.0.0
      - task: ansible:lint-ansible
```

## 📝 Directory Structure

```bash
.
├── ansible.cfg
├── roles/
│   ├── asdf/
│   ├── user_setup/
│   ├── package_management/
│   └── zsh_setup/
├── playbooks/
│   └── workstation/
├── logs/
│   └── molecule_tests.log
└── .hooks/
    └── linters/
        └── ansible-lint.yaml
```

## 🤝 Contributing

1. Fork the repository
2. Create a new branch for your changes
3. Ensure all tests pass: `task run-molecule-tests`
4. Lint your changes: `task lint-ansible`
5. Update changelog: `task gen-changelog NEXT_VERSION=x.y.z`
6. Submit your PR

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.
