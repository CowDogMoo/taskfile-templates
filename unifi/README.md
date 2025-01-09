# 🎯 Unifi Controller Taskfile Templates

This directory contains reusable Taskfile templates for Unifi Controller
operations, including device management, client monitoring, and network
administration tasks.

## 📋 Prerequisites

- [Task](https://taskfile.dev/) installed (`brew install go-task/tap/go-task`)
- [curl](https://curl.se/) installed
- [jq](https://stedolan.github.io/jq/) installed
- [1Password CLI](https://1password.com/downloads/command-line/) installed and configured

## 🔐 Configuration

The taskfile requires the following configuration:

1. Set your Unifi Controller host either through:

   - Environment variable: `export UBIQUITI_CONTROLLER_HOST=192.168.1.1`
   - Command line argument: `UBIQUITI_CONTROLLER_HOST=192.168.1.1 task list-sites`
   - `.env` file (add to .gitignore):

     ```bash
     UBIQUITI_CONTROLLER_HOST=192.168.1.1
     ```

1. Store your Unifi API key in 1Password:

   - Create an item named "Ubiquiti Account"
   - Add a field named "control-plane-api-key" with your API key

## 🎯 Available Tasks

### check-dependencies

Validates that required tools (curl, jq, 1Password CLI) are installed on your system.

```bash
task check-dependencies
```

### list-sites

Lists all Ubiquiti sites accessible to your account.

```bash
task list-sites
```

### list-unifi-devices

Lists all adopted Unifi devices in the default site.

```bash
task list-unifi-devices
```

### list-clients

Lists all clients in the default site. Optionally filter by network name.

**List all clients:**

```bash
task list-clients
```

**Filter by network:**

```bash
task list-clients NETWORK=IoT
```

### list-fw-rules

Lists all firewall rules configured in the default site.

```bash
task list-fw-rules
```

## 📝 Example Usage

1. **List all devices in your network:**

   ```bash
   export UBIQUITI_CONTROLLER_HOST=192.168.1.1
   task list-unifi-devices
   ```

1. **View clients on a specific network:**

   ```bash
   task list-clients NETWORK=Guest
   ```

1. **Check firewall rules:**

   ```bash
   UBIQUITI_CONTROLLER_HOST=192.168.1.1 task list-fw-rules
   ```

## 🔧 Extending Tasks

You can extend these tasks in your own Taskfile by importing this template and
overriding or adding new tasks. Here's an example:

```yaml
version: "3"

includes:
  unifi:
    taskfile: ./unifi.yml
    optional: true

tasks:
  # Override or extend existing tasks
  list-devices:
    deps: [unifi:list-unifi-devices]
    cmds:
      - echo "Additional processing of device list..."

  # Add new tasks that use the base tasks
  monitor-network:
    cmds:
      - task: unifi:list-clients
      - task: unifi:list-fw-rules
```

## 🔍 Important Notes

- All sensitive information (API keys, credentials) should be stored in
  1Password
- The taskfile uses the 1Password CLI to securely retrieve credentials at
  runtime
- API requests use `-k` flag with curl to handle self-signed certificates
  (common in Unifi setups)
- Proper error handling is implemented with `set -euo pipefail`
- JSON responses are automatically formatted using jq for better readability
