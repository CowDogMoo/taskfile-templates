# 📦 Packer Taskfile Tasks

This directory contains reusable [Taskfile](https://taskfile.dev) tasks for
HashiCorp Packer development workflows, including formatting, validation,
template management, plugin control, and more.

## 📋 Prerequisites

- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)
- [Packer](https://www.packer.io/downloads) installed and in your PATH

## 🎯 Available Tasks

---

### check-packer

Checks for the existence of the `packer` binary in your environment.

```bash
task check-packer
```

---

### lint

Formats and validates all Packer files recursively. Runs `packer fmt` and
`packer validate` for all templates.

```bash
task lint
```

---

### format

Auto-formats all packer templates recursively using `packer fmt`.

```bash
task format
```

---

### validate

Runs `packer validate` for every Packer template directory in the repo to
ensure syntax correctness.

```bash
task validate
```

---

### packer-fmt

Formats packer template files with additional options.

**Optional Variables:**

- `CHECK` (`true`/`false`): Only check formatting, do not write changes.
- `RECURSIVE` (`true`/`false`): Recurse into subdirectories.
- `TEMPLATE_DIR`: Directory containing templates.

```bash
task packer-fmt
task packer-fmt CHECK=true
task packer-fmt RECURSIVE=false TEMPLATE_DIR=subdir/
```

---

### packer-init

Initializes a Packer working directory (useful for plugin management).

**Optional Variables:**

- `TEMPLATE_DIR`
- `TEMPLATE`

```bash
task packer-init TEMPLATE_DIR=foo/
```

---

### packer-validate

Validates a specific Packer template or directory.

**Optional Variables:**

- `TEMPLATE_DIR`
- `TEMPLATE`
- `VAR_FILE`

```bash
task packer-validate TEMPLATE_DIR=foo/ TEMPLATE=main.pkr.hcl
```

---

### packer-build

Builds images from a template, supports filtering and forcing.

**Optional Variables:**

- `TEMPLATE_DIR`
- `TEMPLATE`
- `VAR_FILE`
- `ONLY`
- `EXCEPT`
- `FORCE` (set to `true` to force a build)

```bash
task packer-build TEMPLATE_DIR=foo/ ONLY=amazon-ebs FORCE=true
```

---

### packer-console

Launches an interactive Packer console for variable interpolation and testing.

**Optional Variables:**

- `TEMPLATE_DIR`
- `TEMPLATE`
- `VAR_FILE`

```bash
task packer-console TEMPLATE_DIR=foo/ TEMPLATE=main.pkr.hcl
```

---

### packer-inspect

Inspects a template and prints details about its configuration.

**Required Variable:**

- `TEMPLATE`

**Optional Variable:**

- `TEMPLATE_DIR`

```bash
task packer-inspect TEMPLATE_DIR=foo/ TEMPLATE=main.pkr.hcl
```

---

### packer-hcl2-upgrade

Converts a legacy JSON template to HCL2 and writes it to a target directory.

**Required Variable:**

- `JSON_TEMPLATE`

**Optional Variable:**

- `OUTPUT_DIR` (default: `./hcl-templates`)

```bash
task packer-hcl2-upgrade JSON_TEMPLATE=old-template.json
```

---

### packer-plugin-install

Installs a Packer plugin.

**Required Variable:**

- `PLUGIN_NAME`

**Optional Variable:**

- `PLUGIN_VERSION` (default: `latest`)

```bash
task packer-plugin-install PLUGIN_NAME=github.com/hashicorp/amazon
```

---

### packer-plugin-list

Lists installed Packer plugins in the environment.

```bash
task packer-plugin-list
```

---

## 🛠 Template Variables

- Most commands accept `TEMPLATE_DIR`, `TEMPLATE`, `VAR_FILE` for flexible
  template management.
- You can build, validate, or inspect specific templates or apply commands to
  entire directories.

---

## 📂 Typical Directory Structure

```bash
.
├── packer/
│   ├── main.pkr.hcl
│   └── variables.pkr.hcl
├── old-templates/
│   └── deprecated.json
├── hcl-templates/
├── Taskfile.yaml
└── README.md
```

---

## 🤝 Contributing

1. Fork the repository.
1. Create a new branch.
1. Ensure all tasks run: `task lint` and `task validate`
1. Format your code: `task format`
1. Submit your PR!

---

## 📜 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
