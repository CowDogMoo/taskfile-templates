# 🎭 Ansible Taskfile Tasks

This directory contains reusable Taskfile tasks for Ansible development
operations, including molecule testing, linting, and changelog management.

## 📋 Prerequisites

- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
  installed
- [Molecule](https://molecule.readthedocs.io/en/latest/installation.html) installed
- [act](https://github.com/nektos/act) installed (for GitHub Actions local testing)
- [jq](https://stedolan.github.io/jq/download/) installed (for JSON processing)
- [antsibull-changelog](https://github.com/ansible-community/antsibull-changelog)
  installed
- Docker installed (required for Molecule and act)

## 🎯 Available Tasks

### changelog-lint

Lints the changelog using antsibull-changelog to ensure it follows the proper format.

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

### lint-ansible

Runs Ansible Lint with custom configuration from `.hooks/linters/ansible-lint.yaml`.

```bash
task lint-ansible
```

### run-molecule-action

Runs GitHub Actions molecule workflow locally using act. Supports testing
specific components. This task expects a GitHub Actions workflow file at
`.github/workflows/molecule.yaml`. See example workflow implementations:

- [workstation-collection/molecule.yaml](https://github.com/CowDogMoo/ansible-collection-workstation/blob/main/.github/workflows/molecule.yaml)
  - Example workflow for testing workstation configuration roles
- [arsenal-collection/molecule.yaml](https://github.com/l50/ansible-collection-arsenal/blob/main/.github/workflows/molecule.yaml)
  - Example workflow for testing security tooling roles

**Optional Variables:**

- `ROLE`: Name of the specific role to test
- `PLAYBOOK`: Name of the specific playbook to test

```bash
# Run all tests
task run-molecule-action

# Test specific role
task run-molecule-action ROLE=asdf

# Test specific playbook
task run-molecule-action PLAYBOOK=workstation
```

### run-molecule-tests

Executes Molecule tests for all roles in the collection sequentially.

```bash
task run-molecule-tests
```

## 🔍 Important Notes

- All tasks include proper error handling and logging to `logs/molecule_tests.log`
- Molecule tests use the project-specific `ansible.cfg` configuration
- The `run-molecule-action` task automatically handles ARM64 architecture on macOS
- Docker containers are automatically cleaned up between test runs
- All changelog operations require proper version specification

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
