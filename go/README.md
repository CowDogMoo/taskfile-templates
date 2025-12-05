# Go Taskfile Tasks

This directory contains reusable Taskfile tasks for Go development operations,
including building, testing, linting, and release automation with
cross-platform support.

## Prerequisites

You must have the following installed for full functionality:

- [Task](https://taskfile.dev) (`brew install go-task/tap/go-task`)
- [Go](https://golang.org/doc/install) (1.21+)
- [golangci-lint](https://golangci-lint.run/usage/install/) (for linting)
- [gh CLI](https://cli.github.com/) (for release tasks)
- [goreleaser](https://goreleaser.com/install/) (optional, for local release testing)

## Configuration

This Taskfile uses customizable variables that you can override in your
project's Taskfile:

```yaml
vars:
  BINARY_NAME: "myapp" # Default: "app"
  CMD_PATH: "./cmd/myapp" # Default: "./cmd"
```

The following variables are auto-generated:

- `VERSION`: Git tag or "dev"
- `COMMIT`: Short git commit hash
- `DATE`: Build timestamp
- `LDFLAGS`: Linker flags with version info
- `HOST_OS`: Current operating system
- `HOST_ARCH`: Current architecture

## Available Tasks

### Core Build Tasks

#### build

Build binary with optional OS, ARCH, and GOARM parameters. Defaults to current platform.

```bash
# Build for current platform (outputs to current directory)
task build

# Build for specific platform (outputs to dist/ directory)
task build OS=linux ARCH=amd64
task build OS=darwin ARCH=arm64
task build OS=windows ARCH=amd64

# Build for ARM with specific version
task build OS=linux ARCH=arm GOARM=7
```

**Parameters:**

- `OS`: Target operating system (linux, darwin, windows) - defaults to host OS
- `ARCH`: Target architecture (amd64, arm64, arm) - defaults to host architecture
- `GOARM`: ARM version (6, 7) - only applicable when ARCH=arm

**Output behavior:**

- Native builds (host OS/ARCH): Output to current directory as `{{.BINARY_NAME}}`
- Cross-platform builds: Output to `dist/` as `{{.BINARY_NAME}}-{OS}-{ARCH}[.exe]`

#### build-all

Build for all supported platforms (outputs to `dist/` directory):

- Linux: amd64, arm64, armv7
- macOS: amd64, arm64
- Windows: amd64

```bash
task build-all
```

### Development Tasks

#### run

Build and run the binary with optional arguments.

```bash
task run
task run -- --config=myconfig.yaml
```

#### dev

Run in development mode with verbose logging.

```bash
task dev
task dev -- --port=8080
```

#### fmt

Format code using `go fmt`.

```bash
task fmt
```

#### tidy

Tidy and verify module dependencies.

```bash
task tidy
```

#### install

Install the binary to `$GOPATH/bin`.

```bash
task install
```

### Testing Tasks

#### test

Run tests with race detection and coverage.

```bash
task test
```

#### test-coverage

Run tests and generate an HTML coverage report.

```bash
task test-coverage
```

Opens `coverage.html` with a visual representation of code coverage.

#### lint

Run golangci-lint to check code quality.

```bash
task lint
```

### Utility Tasks

#### clean

Remove build artifacts and clean Go cache.

```bash
task clean
```

#### show-arch

Display detected architecture and build information.

```bash
task show-arch
```

Example output:

```text
Host OS: darwin
Host ARCH: arm64
Version: v1.2.3
Build strategy: native
```

### CI/CD Tasks

#### ci-test

Run tests with JSON output for CI/CD systems.

```bash
task ci-test
```

Generates `test-results.json` and displays coverage summary.

#### ci-build

Build and test in CI/CD environment.

```bash
task ci-build
```

#### ci-build-native

Build for native architecture with file type detection.

```bash
task ci-build-native
```

#### ci-matrix

Display recommended CI/CD build matrix configuration.

```bash
task ci-matrix
```

Example output:

```yaml
Recommended CI/CD build matrix:
  - arch: amd64
    runner: ubuntu-latest
    strategy: native

  - arch: arm64
    runner: ubuntu-24.04-arm
    strategy: native
```

### Release Tasks

These tasks support the GoReleaser-based release workflow where tagging
triggers automated builds. For projects with changelog files or different
release workflows, see the `github:create-release` task in the
[github taskfile](https://github.com/CowDogMoo/taskfile-templates/tree/main/github).

#### release

Create and push a new release tag, triggering GoReleaser GitHub Action.

**Prerequisites:**

- Clean working directory
- gh CLI installed
- TAG variable specified

```bash
task release TAG=v1.0.0
```

This will:

1. Create an annotated git tag
2. Push the tag to the remote repository
3. Trigger GoReleaser GitHub Action workflow
4. Display the workflow run URL

#### release-check

Check release status or list recent releases.

```bash
# Check specific release
task release-check TAG=v1.0.0

# List latest releases
task release-check
```

#### release-watch

Watch the GoReleaser workflow run in real-time.

```bash
task release-watch
```

#### release-test

Test GoReleaser locally without publishing.

```bash
task release-test
```

Builds artifacts in snapshot mode and outputs to `dist/` directory.

#### release-draft

Create a draft release on GitHub for manual review before publishing.

```bash
# With auto-generated notes
task release-draft TAG=v1.0.0

# With custom notes
task release-draft TAG=v1.0.0 TITLE="Release 1.0.0" NOTES="Custom release notes"
```

**Note:** This creates a draft release that you can edit before publishing. For
the standard GoReleaser workflow, use `task release` which tags and triggers
automated release builds via GitHub Actions.

#### release-delete

Delete a release and its associated tag (local and remote).

```bash
task release-delete TAG=v1.0.0
```

**Warning:** This operation is destructive and will prompt for confirmation.

#### release-changelog

Generate changelog between two git references.

```bash
# Since last tag
task release-changelog

# Between specific tags
task release-changelog FROM=v1.0.0 TO=v1.1.0

# From tag to HEAD
task release-changelog FROM=v1.0.0 TO=HEAD
```

### Default Task

Running `task` without arguments executes the default build, test, and clean workflow:

```bash
task
```

This runs (in order):

1. `clean` - Remove old artifacts
2. `build` - Build for current platform (native build to current directory)
3. `test` - Run test suite with race detection

## Usage in Your Project

### Basic Setup

Create a `Taskfile.yaml` in your project root:

```yaml
version: "3"

vars:
  BINARY_NAME: "myapp"
  CMD_PATH: "./cmd/myapp"

includes:
  go:
    taskfile: "https://raw.githubusercontent.com/CowDogMoo/taskfile-templates/main/go/Taskfile.yaml"

tasks:
  # Add project-specific tasks here
  deploy:
    desc: Deploy the application
    deps:
      - go:build
    cmds:
      - ./deploy.sh
```

### Usage Examples

```bash
# Build for current platform
task go:build

# Run tests
task go:test

# Build for all platforms
task go:build-all

# Create a release
task go:release TAG=v1.0.0

# Run with custom binary name
BINARY_NAME=mycli task go:build
```

## Additional Taskfiles

The Go taskfile focuses on Go-specific development tasks. For additional
functionality, you can include other taskfiles from this repository:

```yaml
version: "3"

vars:
  BINARY_NAME: "myapp"
  CMD_PATH: "./cmd/myapp"

includes:
  go:
    taskfile: "https://raw.githubusercontent.com/CowDogMoo/taskfile-templates/main/go/Taskfile.yaml"
  github:
    taskfile: "https://raw.githubusercontent.com/CowDogMoo/taskfile-templates/main/github/Taskfile.yaml"
  docker:
    taskfile: "https://raw.githubusercontent.com/CowDogMoo/taskfile-templates/main/docker/Taskfile.yaml"
  pre-commit:
    taskfile: "https://raw.githubusercontent.com/CowDogMoo/taskfile-templates/main/pre-commit/Taskfile.yaml"
```

Available complementary taskfiles:

- **github**: GitHub CLI tasks (PR management, submodules, releases with
  changelog files, GHCR operations)
- **docker**: Docker build and management tasks
- **pre-commit**: Pre-commit hook management
- **secrets**: Secret scanning and management
- **renovate**: Renovate bot configuration

## Directory Structure

Expected Go project structure:

```text
.
├── cmd/
│   └── myapp/
│       └── main.go
├── pkg/
│   └── ...
├── internal/
│   └── ...
├── dist/                 # Generated by build-all
│   ├── myapp-linux-amd64
│   ├── myapp-darwin-arm64
│   └── ...
├── coverage.out          # Generated by test
├── coverage.html         # Generated by test-coverage
├── test-results.json     # Generated by ci-test
├── go.mod
├── go.sum
├── Taskfile.yaml
└── README.md
```

## Version Embedding

The Taskfile automatically embeds version information into your binary using
ldflags. To use it in your Go code:

```go
package main

var (
    version = "dev"
    commit  = "unknown"
    date    = "unknown"
)

func main() {
    fmt.Printf("Version: %s\nCommit: %s\nBuilt: %s\n", version, commit, date)
}
```

When built with this Taskfile:

- `version`: Git tag or "dev"
- `commit`: Short git commit hash
- `date`: ISO 8601 build timestamp

## GoReleaser Integration

The release tasks are designed to work with GoReleaser GitHub Actions. Example `.github/workflows/goreleaser.yaml`:

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

permissions:
  contents: write

jobs:
  goreleaser:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-go@v5
        with:
          go-version: "1.21"
      - uses: goreleaser/goreleaser-action@v5
        with:
          distribution: goreleaser
          version: latest
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Important Notes

- All build tasks automatically run `go mod tidy` as a dependency
- Cross-compilation is native (no CGO by default)
- Source tracking ensures builds only run when files change
- Release tasks require a clean git working directory
- Test tasks use race detection for concurrency issues
- Coverage reports are generated in HTML format for easy viewing

## Contributing

1. Fork the repository
2. Create a new branch for your changes
3. Ensure tests pass: `task test`
4. Format code: `task fmt`
5. Run linter: `task lint`
6. Submit as a PR

## License

MIT License. See [LICENSE](LICENSE) file for details.
