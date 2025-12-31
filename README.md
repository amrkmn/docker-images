# Docker Images for Act

[![Build Ubuntu](https://github.com/amrkmn/docker-images/actions/workflows/build-ubuntu.yml/badge.svg)](https://github.com/amrkmn/docker-images/actions/workflows/build-ubuntu.yml)
[![Linter](https://github.com/amrkmn/docker-images/actions/workflows/lint.yml/badge.svg)](https://github.com/amrkmn/docker-images/actions/workflows/lint.yml)

Simplified Docker images for use with [nektos/act][nektos/act] - run GitHub Actions locally.

This is a streamlined fork of [catthehacker/docker_images](https://github.com/catthehacker/docker_images) focused on core act functionality without the specialized language tooling variants.

## Images Available

### Act Base Image

[`/linux/ubuntu/act`](./linux/ubuntu/scripts/act.sh) - Medium-sized image for [nektos/act][nektos/act] with essential tools for running most GitHub Actions workflows.

Available tags:
- `ghcr.io/amrkmn/ubuntu:act-22.04`
- `ghcr.io/amrkmn/ubuntu:act-24.04`
- `ghcr.io/amrkmn/ubuntu:act-latest`

### What's Included

- Ubuntu base (22.04 or 24.04)
- Essential build tools and utilities
- Git, curl, wget, and common CLI tools
- Docker-in-Docker support
- YQ (YAML processor)
- Compatible with most GitHub Actions

## Usage with Act

Configure act to use these images in your `.actrc` file:

```bash
-P ubuntu-latest=ghcr.io/amrkmn/ubuntu:act-latest
-P ubuntu-24.04=ghcr.io/amrkmn/ubuntu:act-24.04
-P ubuntu-22.04=ghcr.io/amrkmn/ubuntu:act-22.04
```

Or specify directly when running:

```bash
act -P ubuntu-latest=ghcr.io/amrkmn/ubuntu:act-latest
```

## Building Images

Images are built using the PowerShell script with buildah:

```powershell
pwsh build.ps1 `
  -type act `
  -tag ghcr.io/amrkmn/ubuntu:act-latest `
  -platforms "linux/amd64,linux/arm64" `
  -push
```

See [build.ps1](./build.ps1) for more build options.

## Development

For AI coding agents working on this repository, see [AGENTS.md](./AGENTS.md) for guidelines and best practices.

## License

This repository contains parts of [`actions/runner-images`][actions/runner-images] which is licensed under the [MIT License](https://github.com/actions/runner-images/blob/main/LICENSE).

## Acknowledgments

- Original repository: [catthehacker/docker_images](https://github.com/catthehacker/docker_images)
- Based on work from [actions/runner-images][actions/runner-images]

[nektos/act]: https://github.com/nektos/act
[actions/runner-images]: https://github.com/actions/runner-images
