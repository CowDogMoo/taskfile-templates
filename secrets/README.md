# 🔐 Secrets Taskfile Tasks

Reusable Taskfile helpers for **secure secret management** with
• 1Password CLI (`op`) – remote, team-ready vaults
• sops + age – local, Git-friendly file encryption

Works on macOS, Linux and Windows (WSL/PowerShell via Task).

---

## 📋 Prerequisites

Global
• [Task](https://taskfile.dev) ≥ v3
• [jq](https://stedolan.github.io/jq) (optional but recommended for nicer JSON parsing)

onepassword/ tasks
• [1Password CLI](https://developer.1password.com/docs/cli/get-started)
(`brew install --cask 1password/tap/1password-cli`)

local/ tasks
• [sops](https://github.com/mozilla/sops) (`brew install sops` / `apt …`)
• [age](https://github.com/FiloSottile/age) (`brew install age` / `apt …`)

---

## 🎯 Available Tasks

### onepassword/

1. ensure-dependencies
   Verify that `op` is on the PATH (auto-runs for every other task).

1. setup-account
   Add/sign-in to a 1Password account once per workstation.
   Variables
   • `OP_ACCOUNT` – your domain (default `my`).

   ```bash
   # first time
   task onepassword:setup-account OP_ACCOUNT=my-team.1password.com
   ```

1. list-vaults

   ```bash
   task onepassword:list-vaults
   ```

1. list-secrets
   List items with optional filters.
   • `VAULT_NAME` – vault slug / UUID
   • `CATEGORY` – Login, Password, API_Credential …

   ```bash
   task onepassword:list-secrets VAULT_NAME=Development CATEGORY=API_Credential
   ```

1. get-secret
   Retrieve a field (defaults to a password) by item **name or UUID**.
   Optional vars
   • `VAULT` – vault slug / UUID
   • `FIELD` – field label / ID / purpose (default `password`)
   • `FORMAT` – `json|text` (fallback)
   • `OUTPUT_FILE` – save to file instead of stdout

   ```bash
   # Print the password for “github-token” in vault “CI”
   task onepassword:get-secret SECRET_NAME=github-token VAULT=CI

   # Save an entire item as JSON for debugging
   task onepassword:get-secret SECRET_NAME=abcd1234 OUTPUT_FILE=/tmp/item.json FORMAT=json
   ```

1. create-vault

   ```bash
   task onepassword:create-vault VAULT_NAME=Staging
   ```

---

### local/

1. ensure-dependencies
   Guard against missing `sops` / `age`.

1. create-age-keypair
   Generates `~/.config/sops/age/keys.txt` (if absent) and prints the public key.

   ```bash
   task local:create-age-keypair
   ```

1. encrypt-file
   Encrypt **YAML / any file** with sops-age.
   Vars
   • `INPUT_FILE` (required) – path of plaintext
   • `OUTPUT_FILE` – override destination
   • `AGE_FILE` – custom private key path
   • `ENCRYPTED_REGEX` – only match keys (default covers common secret names)

   ```bash
   task local:encrypt-file INPUT_FILE=secrets.yaml
   # → secrets.yaml.sops.yaml
   ```

1. decrypt-file
   Vars
   • `INPUT_FILE` (required) – encrypted file
   • `OUTPUT_FILE` – plaintext destination (`*.dec` default)
   • `AGE_FILE` – override key location

   ```bash
   task local:decrypt-file INPUT_FILE=secrets.yaml.sops.yaml
   ```

---

## 🔍 Important Notes

- All tasks exit early with helpful messages when prerequisites or input vars
  are missing.
- `get-secret` tries **three strategies** (field ID, field label, reference
  path) before failing.
- `encrypt-file` automatically builds the output filename (`.sops.yaml`) if
  none is supplied.
- `decrypt-file` respects `SOPS_AGE_KEY_FILE` or falls back to the generated
  key.
- No secrets are written to disk unless you supply `OUTPUT_FILE`.

---

## 🔧 Importing Into Your Project

Task supports multiple included Taskfiles. Add to your root `Taskfile.yaml`:

```yaml
version: "3"
includes:
  secrets-onepassword: "./onepassword/Taskfile.yaml"
  secrets-local: "./local/Taskfile.yaml"

tasks:
  bootstrap-secrets:
    desc: "Ensure secrets tooling is ready"
    cmds:
      - task: secrets-onepassword:setup-account
        vars: { OP_ACCOUNT: my-team.1password.com }
      - task: secrets-local:create-age-keypair
```

Now run `task bootstrap-secrets` from anywhere in your repo.

---

## 📝 Directory Layout

```text
.
├── onepassword/
│   └── Taskfile.yaml   # 1Password helpers
├── local/
│   └── Taskfile.yaml   # sops-age helpers
└── README.md           # (this file)
```

---

## 🤝 Contributing

1. Fork & branch.
1. Add/adjust tasks – keep them idempotent and cross-platform.
1. `task secrets-onepassword:ensure-dependencies` and
   `task secrets-local:ensure-dependencies` must pass.
1. Document any new variables in this README.
1. Open a PR – GitHub Actions will lint and test.

---

## 📜 License

MIT – see `LICENSE` for details.
