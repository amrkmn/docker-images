# GitHub Actions Workflows

This directory contains GitHub Actions workflows for building and maintaining the act Docker images.

## Workflows

### build-ubuntu.yml

Builds the act Docker image for Ubuntu 22.04 and 24.04 on multiple architectures.

**Triggers:**
- **Push**: Automatically builds when changes are pushed to `main`/`master` branch
- **Pull Request**: Builds on PRs to validate changes (without pushing)
- **Manual Dispatch**: Allows manual triggering with custom parameters
- **Schedule**: Weekly builds on Sundays at 2 AM UTC

**Manual Dispatch Parameters:**
- `ubuntu_version`: Select Ubuntu version (22.04, 24.04, or both)
- `push_image`: Whether to push the built image to the registry

**Features:**
- Multi-platform builds (linux/amd64, linux/arm64)
- Automatic tagging (version-specific and latest)
- Build caching for faster builds
- Image testing after build
- Pushes to GitHub Container Registry (ghcr.io)

**Image Tags:**
- `ghcr.io/<owner>/ubuntu:act-22.04` - Ubuntu 22.04 based image
- `ghcr.io/<owner>/ubuntu:act-24.04` - Ubuntu 24.04 based image
- `ghcr.io/<owner>/ubuntu:act-latest` - Latest version (24.04)

### lint.yml

Runs MegaLinter to validate code quality and style.

**Triggers:**
- **Push**: On pushes to `main`/`master`
- **Pull Request**: On all PRs
- **Manual Dispatch**: Can be triggered manually

**Features:**
- Lints Dockerfiles, shell scripts, YAML files, and more
- Uploads results as artifacts
- Integrates with GitHub Security (SARIF)
- Follows `.mega-linter.yml` configuration

## Usage Examples

### Manual Build via GitHub UI

1. Go to **Actions** tab in GitHub
2. Select **Build Ubuntu Docker Images**
3. Click **Run workflow**
4. Choose parameters:
   - Ubuntu version: `both` (builds 22.04 and 24.04)
   - Push image: `true`
5. Click **Run workflow**

### Manual Build via GitHub CLI

```bash
# Build for both Ubuntu versions and push
gh workflow run build-ubuntu.yml \
  -f ubuntu_version=both \
  -f push_image=true

# Build for Ubuntu 24.04 only (no push)
gh workflow run build-ubuntu.yml \
  -f ubuntu_version=24.04 \
  -f push_image=false
```

### Run Linter Locally

```bash
# Using Docker
docker run --rm -v $(pwd):/tmp/lint oxsecurity/megalinter:v7

# Using mega-linter-runner
mega-linter-runner
```

## Permissions

The workflows require the following permissions:
- `contents: read` - Read repository contents
- `packages: write` - Push images to GitHub Container Registry
- `issues: write` - Post linter comments (lint.yml)
- `pull-requests: write` - Post linter comments (lint.yml)

## Secrets

The workflows use `GITHUB_TOKEN` which is automatically provided by GitHub Actions.

## Troubleshooting

### Build Fails

1. Check the build logs in the Actions tab
2. Verify the Dockerfile and scripts in `linux/ubuntu/`
3. Test locally using `build.ps1`

### Image Not Pushed

1. Ensure `push_image` is set to `true` (for manual dispatch)
2. Verify you have write permissions to GitHub Container Registry
3. Check that the event is not a pull request (PRs don't push by default)

### Linter Errors

1. Review the MegaLinter report artifacts
2. Run MegaLinter locally to see detailed errors
3. Fix issues according to `.mega-linter.yml` configuration
