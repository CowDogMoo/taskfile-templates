# 🎭 Ansible Taskfile Tasks

This directory contains reusable Taskfile tasks for Ansible development operations,
including molecule testing, linting, and changelog management.

## 📋 Prerequisites

- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
  installed
- [Molecule](https://molecule.readthedocs.io/en/latest/installation.html) installed
- [act](https://github.com/nektos/act) installed (for GitHub Actions local testing)
- [jq](https://stedolan.github.io/jq/download/) installed (for JSON processing)
- [antsibull-changelog](https://github.com/ansible-community/antsibull-changelog)
  installed

## 🎯 Available Tasks

### act

Runs GitHub Actions molecule workflow locally using act, with optional
component targeting.

**Variables:**

- `COMPONENT`: Specific role or playbook to test (optional)

```bash
# Run all tests
task act

# Test specific role
task act COMPONENT=asdf

# Test specific playbook
task act COMPONENT=workstation
```

### changelog-lint

Lints the changelog using antsibull-changelog.

```bash
# Lint changelog
task changelog-lint
```

### changelog-release

Generates a changelog release with the specified version.

```bash
# Generate release changelog
task changelog-release
```

### gen-changelog

Generates the changelog for the next release, including linting and release generation.

**Variables:**

- `NEXT_VERSION`: Version number for the release (required)

```bash
# Generate changelog for version 1.0.0
task gen-changelog NEXT_VERSION=1.0.0
```

### lint-ansible

Runs Ansible Lint with custom configuration.

```bash
# Run ansible-lint
task lint-ansible
```

### run-molecule-tests

Runs Molecule tests for all roles in the collection.

```bash
# Run all molecule tests
task run-molecule-tests

# Tests will:
# - Create logs directory
# - Use project-specific ansible.cfg
# - Test each role sequentially
# - Log output to logs/molecule_tests.log
```

## 🔍 Important Notes

- All tasks include proper error handling and logging
- Molecule tests are run with project-specific ansible.cfg
- Changelog generation requires proper version specification
- GitHub Actions local testing supports ARM64 architecture on macOS

## 🔧 Extending Tasks

Import these tasks in your own Taskfile:

```yaml
version: "3"
includes:
  ansible: "./path/to/ansible/Taskfile.yaml"

tasks:
  test-and-release:
    cmds:
      # Run all tests
      - task: ansible:run-molecule-tests

      # Generate changelog
      - task: ansible:gen-changelog
        vars:
          NEXT_VERSION: 1.0.0

      # Lint everything
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
└── .hooks/
    └── linters/
        └── ansible-lint.yaml
```

## 🤝 Contributing

1. Ensure all tests pass: `task run-molecule-tests`
2. Lint your changes: `task lint-ansible`
3. Update changelog: `task gen-changelog NEXT_VERSION=x.y.z`
4. Submit your PR

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.
