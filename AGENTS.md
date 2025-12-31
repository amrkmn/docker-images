# Agent Guidelines for Docker Images Repository

This document provides guidelines for AI coding agents working in this repository.

## Project Overview

This is a Docker images repository for building Ubuntu-based container images used with [nektos/act](https://github.com/nektos/act) and GitHub Actions. The project creates various specialized Docker images (act, runner, js, rust, pwsh, go, dotnet, etc.) based on Ubuntu 22.04 and 24.04.

## Repository Structure

```
docker_images/
├── linux/ubuntu/
│   ├── Dockerfile              # Main Dockerfile with build args
│   └── scripts/                # Installation scripts
│       ├── act.sh              # Base act image installer
│       ├── yq.sh               # YQ tool installer
│       └── helpers/            # Helper functions
│           ├── install.sh      # Download and installation helpers
│           ├── etc-environment.sh  # Environment variable helpers
│           └── os.sh           # OS detection helpers
├── build.ps1                   # PowerShell build script using buildah
├── .mega-linter.yml            # MegaLinter configuration
├── .prettierrc.yml             # Prettier configuration
└── .editorconfig               # Editor configuration
```

## Build Commands

### Building Docker Images

```bash
# Build using PowerShell script (requires buildah)
pwsh build.ps1 \
  -type act \
  -tag ghcr.io/catthehacker/ubuntu:act-latest \
  -platforms "linux/amd64,linux/arm64" \
  -push
```

### Build Parameters

- `-type`: Image type (act, runner, js, rust, pwsh, go, dotnet, etc.)
- `-tag`: Docker image tag
- `-platforms`: Target platforms (linux/amd64, linux/arm64)
- `-node`: Node.js versions to install
- `-from_image`: Base image
- `-from_tag`: Base image tag
- `-push`: Push to registry after build

### Linting

```bash
# Run MegaLinter (lints Dockerfiles, shell scripts, etc.)
mega-linter-runner

# Or using Docker
docker run --rm -v /path/to/repo:/tmp/lint oxsecurity/megalinter:latest
```

## Code Style Guidelines

### Shell Scripts

#### Shebang and Options

```bash
#!/bin/bash -e
# Use -e to exit on error
# Use set -Eeuxo pipefail for stricter error handling in main scripts
```

#### Script Headers

All shell scripts should include a header comment block:

```bash
#!/bin/bash -e
################################################################################
##  File:  script-name.sh
##  Desc:  Brief description of what this script does
################################################################################
# source: [original source URL if applicable]
```

#### Shellcheck Directives

Use shellcheck disable comments when necessary:

```bash
# shellcheck disable=SC2174
```

#### Functions

- Use `function` keyword for function definitions
- Use camelCase for function names
- Use descriptive names (e.g., `getEtcEnvironmentVariable`, `download_with_retries`)
- Add comments explaining complex logic

```bash
function downloadWithRetries {
    local URL="$1"
    local DEST="${2:-.}"
    local NAME="${3:-${URL##*/}}"
    
    # Function implementation
}
```

#### Variables

- Use double quotes around variables: `"${variable_name}"`
- Use local variables in functions: `local variable_name="value"`
- Use UPPERCASE for constants and environment variables
- Use lowercase or snake_case for local variables

#### Error Handling

- Always check exit codes for critical operations
- Use `set -e` to exit on errors
- Provide meaningful error messages

```bash
if [ $http_code -eq 200 ] && [ $exit_code -eq 0 ]; then
    echo "Download completed"
    return 0
else
    echo "Error — HTTP response code is wrong - '$http_code'"
    return 1
fi
```

#### Sourcing Helper Scripts

```bash
# Source the helpers for use with the script
. /imagegeneration/installers/helpers/install.sh
```

### Dockerfile

#### Build Arguments

- Always declare ARGs at the top after FROM
- ARGs before FROM are not accessible in the build context
- Use descriptive ARG names

```dockerfile
ARG FROM_IMAGE
ARG FROM_TAG
FROM ${FROM_IMAGE}:${FROM_TAG}

ARG TARGETARCH
ARG NODE_VERSION="22 24"
ARG DISTRO=ubuntu
ARG TYPE=act
```

#### Labels

Include OCI-compliant labels:

```dockerfile
LABEL org.opencontainers.image.created="${BUILD_DATE}"
LABEL org.opencontainers.image.vendor="${BUILD_OWNER}"
LABEL org.opencontainers.image.source="https://github.com/${BUILD_OWNER}/${BUILD_REPO}"
```

### PowerShell Scripts

#### Indentation

- Use spaces (2 spaces per indent level) for PowerShell files

#### Error Handling

```powershell
function exec() {
    $path, $myargs = $args | Flatten-Array
    & "$path" $myargs
    if($LASTEXITCODE -ne 0) {
        throw "$($args | Flatten-Array) failed with exit code $LASTEXITCODE"
    }
}
```

### General Formatting

#### Indentation (.editorconfig)

- **Default**: Tabs (4 spaces wide)
- **PowerShell (.ps1)**: Spaces
- **YAML/JSON/JS/Shell/Markdown**: 2 spaces
- **End of line**: LF (Unix-style)
- **Charset**: UTF-8
- **Trim trailing whitespace**: Yes
- **Insert final newline**: Yes

#### Prettier (.prettierrc.yml)

```yaml
singleQuote: true
tabWidth: 2
```

## Testing

This repository does not have traditional unit tests. Testing is done through:

1. **Build verification**: Ensure Docker images build successfully
2. **MegaLinter**: Validates shell scripts, Dockerfiles, and configuration files
3. **Manual testing**: Test images with act or GitHub Actions

## Error Handling and Logging

### Shell Scripts

- Use `set -Eeuxo pipefail` for verbose error tracking
- Print progress messages with clear formatting:

```bash
printf "\n\t🐋 Build started 🐋\t\n"
printf "\n\t🐋 Installing packages 🐋\t\n"
```

- Use descriptive error messages with context

### Download Functions

- Implement retry logic (default: 20 retries with 30s intervals)
- Verify checksums for security-critical downloads
- Log HTTP status codes and exit codes

## Common Patterns

### Architecture Detection

```bash
node_arch() {
  case "$(uname -m)" in
    'aarch64') echo 'arm64' ;;
    'x86_64') echo 'x64' ;;
    'armv7l') echo 'armv7l' ;;
    *) exit 1 ;;
  esac
}
```

### Environment Variables

```bash
# Append to /etc/environment
{
  echo "IMAGE_OS=$ImageOS"
  echo "AGENT_TOOLSDIRECTORY=${AGENT_TOOLSDIRECTORY}"
} | tee -a "/etc/environment"
```

### Directory Creation

```bash
# Create with specific permissions and ownership
mkdir -m 0777 -p "${AGENT_TOOLSDIRECTORY}"
chown -R 1001:1000 "${AGENT_TOOLSDIRECTORY}"
```

## Git and Version Control

- Follow conventional commit messages
- Reference issue numbers when applicable
- Keep commits focused and atomic

## MegaLinter Configuration

MegaLinter is configured to:

- Disable COPYPASTE checks
- Disable SPELL_CSPELL checks
- Validate Dockerfiles, shell scripts, YAML, and more

## Notes for Agents

1. **Image builds are expensive**: Test changes locally before building full multi-platform images
2. **Supply chain security**: Always verify checksums for downloaded binaries
3. **Compatibility**: Images support both amd64 and arm64 architectures
4. **Base images**: Scripts assume Ubuntu base images (22.04 or 24.04)
5. **User context**: Some images run as root, others as specific users (e.g., runner)
6. **Toolcache**: Use `/opt/hostedtoolcache` or `/opt/acttoolcache` for tool installations
