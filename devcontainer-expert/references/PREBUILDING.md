# Pre-building Dev Container Images

**Reference for:** Advanced image caching and pre-building workflows to improve team performance and CI/CD integration.

## Overview

Pre-building dev container images rather than building them on demand provides several critical benefits:

- **Faster container startup** - Team members wait seconds instead of minutes
- **Supply-chain security** - Pin exact tool versions, avoid surprises from base image updates
- **CI/CD integration** - Use the same environment for local development and automated testing
- **Consistency** - All team members use identical pre-built images

This reference covers strategies for pre-building images with the Dev Container CLI, leveraging layer caching, embedding metadata, and automating builds with CI/CD systems.

## Pre-building with the Dev Container CLI

The [@devcontainers/cli](https://github.com/devcontainers/cli) is the recommended tool for pre-building images. It stays in sync with the VS Code extension's capabilities and supports Dev Container Features.

### Installation

```bash
npm install -g @devcontainers/cli
```

### Basic Build Command

```bash
# Build image from current directory's devcontainer.json
devcontainer build --workspace-folder .

# Build and push to registry
devcontainer build \
  --workspace-folder . \
  --push true \
  --image-name ghcr.io/myorg/myproject-devcontainer:latest
```

### Build with Caching

```bash
# Use cacheFrom for layer caching
devcontainer build \
  --workspace-folder . \
  --push true \
  --image-name ghcr.io/myorg/myproject-devcontainer:latest \
  --cache-from ghcr.io/myorg/myproject-devcontainer:latest
```

**How Layer Caching Works:**
- Docker reuses layers from `cacheFrom` image if build steps match
- Dramatically reduces build time (minutes → seconds)
- Works with any Docker registry (GHCR, ACR, Docker Hub)

## Image Metadata Inheritance

Pre-built images can embed dev container configuration via the `devcontainer.metadata` label. This allows settings to **automatically merge** when the image is referenced.

### How It Works

1. **Pre-build with metadata:** CLI automatically adds `devcontainer.metadata` label containing settings from devcontainer.json
2. **Reference in simplified config:** Repositories can use minimal devcontainer.json that just references the image
3. **Settings merge at runtime:** VS Code merges embedded metadata with repository-specific config

**Example Pre-build Configuration** (`.devcontainer/devcontainer.json` in base repo):

```json
{
  "name": "Python Data Science",
  "build": {
    "dockerfile": "Dockerfile",
    "args": { "VARIANT": "3.11" }
  },
  "features": {
    "ghcr.io/devcontainers/features/python:1": {
      "version": "3.11"
    },
    "ghcr.io/devcontainers/features/git:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance"
      ]
    }
  },
  "postCreateCommand": "pip install --user -r requirements.txt",
  "remoteUser": "vscode"
}
```

**Simplified Repository Config** (using pre-built image):

```json
{
  "image": "ghcr.io/myorg/python-datascience:latest",
  "postCreateCommand": "pip install --user -r project-requirements.txt"
}
```

**Result:** The repository inherits Features, extensions, and base configuration from the pre-built image, only specifying project-specific customizations.

### Manually Adding Metadata (Optional)

You can manually add metadata to a Dockerfile instead of using the CLI:

```dockerfile
LABEL devcontainer.metadata='[{ \
  "capAdd": ["SYS_PTRACE"], \
  "remoteUser": "devcontainer", \
  "postCreateCommand": "npm install" \
}]'
```

## CI/CD Integration

### Using devcontainers/ci GitHub Action

The [devcontainers/ci](https://github.com/devcontainers/ci) repository provides GitHub Actions and Azure DevOps tasks for pre-building and running dev containers in workflows.

**Pattern 1: Pre-build Image Only**

```yaml
name: Pre-build Dev Container

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # Weekly Monday 2am

jobs:
  prebuild:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: devcontainers/ci@v0.3
        with:
          imageName: ghcr.io/${{ github.repository }}-devcontainer
          cacheFrom: ghcr.io/${{ github.repository }}-devcontainer
          push: always
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Pattern 2: Run CI Tests in Dev Container**

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: devcontainers/ci@v0.3
        with:
          runCmd: |
            npm test
            npm run lint
```

**Pattern 3: Pre-build AND Run Tests**

```yaml
name: Pre-build and Test

on:
  push:
    branches: [main]

jobs:
  devcontainer:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: devcontainers/ci@v0.3
        with:
          imageName: ghcr.io/${{ github.repository }}-devcontainer
          cacheFrom: ghcr.io/${{ github.repository }}-devcontainer
          push: always
          runCmd: |
            npm install
            npm test
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### devcontainers/ci Action Options

| Property | Description |
|----------|-------------|
| `imageName` | Target image name (e.g., `ghcr.io/myorg/myrepo-devcontainer`) |
| `cacheFrom` | Source image for layer caching (typically same as `imageName`) |
| `push` | When to push: `always`, `never`, or `filter` (only on specific branches) |
| `runCmd` | Command(s) to run inside the container after build |
| `env` | Environment variables to set in the container |

### Azure DevOps Integration

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: DevContainers@0
    inputs:
      runCmd: 'npm test'
      imageName: 'myregistry.azurecr.io/myproject-devcontainer'
      push: 'always'
```

## Caching Strategies

### Strategy 1: Registry-Based Caching (Recommended)

Push images to a registry and use `cacheFrom` to reuse layers:

```json
{
  "build": {
    "dockerfile": "Dockerfile",
    "cacheFrom": "ghcr.io/myorg/myproject-devcontainer:cache"
  }
}
```

**Benefits:**
- Works in CI/CD environments
- Team members benefit from cache
- No local cache bloat

### Strategy 2: Tagged Cache Images

Use separate tags for cache vs. production:

```bash
# Build and push cache image
devcontainer build \
  --workspace-folder . \
  --push true \
  --image-name ghcr.io/myorg/myproject:cache

# Build production image using cache
devcontainer build \
  --workspace-folder . \
  --push true \
  --image-name ghcr.io/myorg/myproject:latest \
  --cache-from ghcr.io/myorg/myproject:cache
```

### Strategy 3: Multi-Stage Build Optimization

Use multi-stage Dockerfiles to cache build dependencies separately:

```dockerfile
# Build stage (cached separately)
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM mcr.microsoft.com/devcontainers/typescript-node:20
COPY --from=builder /app/dist /workspace/dist
COPY --from=builder /app/node_modules /workspace/node_modules
```

## Best Practices

### 1. Schedule Regular Rebuilds

Pre-build on a schedule to incorporate base image updates:

```yaml
on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # Weekly on Monday at 2am UTC
```

### 2. Pin Feature Versions

Avoid surprises by pinning exact Feature versions:

```json
"features": {
  "ghcr.io/devcontainers/features/node:1.0.0": {},  // Exact version
  "ghcr.io/devcontainers/features/python:1.0": {}   // Minor version
}
```

### 3. Layer Order Optimization

Order Dockerfile instructions from least to most frequently changing:

```dockerfile
# 1. Base image (rarely changes)
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# 2. System packages (changes occasionally)
RUN apt-get update && apt-get install -y build-essential

# 3. Application dependencies (changes more often)
COPY package.json package-lock.json ./
RUN npm ci

# 4. Application code (changes frequently)
COPY . .
```

### 4. Use BuildKit for Better Caching

Enable Docker BuildKit for improved caching and performance:

```bash
export DOCKER_BUILDKIT=1
devcontainer build --workspace-folder .
```

Or in devcontainer.json:

```json
{
  "build": {
    "dockerfile": "Dockerfile",
    "options": ["--progress=plain"]
  }
}
```

### 5. Registry Authentication

For private registries, authenticate before pushing:

**GitHub Container Registry:**
```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
```

**Azure Container Registry:**
```bash
az acr login --name myregistry
```

**Docker Hub:**
```bash
docker login -u USERNAME
```

## Supply-Chain Security Benefits

Pre-building provides important security advantages:

1. **Version Pinning** - Lock specific tool versions, preventing unexpected updates
2. **Image Scanning** - Scan pre-built images for vulnerabilities before distribution
3. **Audit Trail** - Track exactly what's in each image version
4. **Reduced Attack Surface** - Fewer build-time fetches from external sources

**Example**: Weekly pre-build + scan workflow:

```yaml
name: Pre-build and Scan

on:
  schedule:
    - cron: '0 2 * * 1'

jobs:
  prebuild:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: devcontainers/ci@v0.3
        with:
          imageName: ghcr.io/${{ github.repository }}-devcontainer
          cacheFrom: ghcr.io/${{ github.repository }}-devcontainer
          push: always
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Scan image for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/${{ github.repository }}-devcontainer:latest
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
```

## Troubleshooting

**Build fails with "cacheFrom image not found"**
- Ensure the cache image exists in the registry
- Check authentication for private registries
- Use `--no-cache` flag to build without caching if needed

**Changes not reflected in pre-built image**
- Verify the image tag being pushed matches what repositories reference
- Check that CI workflow successfully completed
- Clear local image cache: `docker rmi <image-name>`

**Slow builds despite caching**
- Verify `cacheFrom` image digest matches recent build
- Check Dockerfile layer ordering (frequently changing steps should be last)
- Ensure BuildKit is enabled: `export DOCKER_BUILDKIT=1`

## Related Documentation

- [Dev Container CLI](https://github.com/devcontainers/cli)
- [devcontainers/ci GitHub Action](https://github.com/devcontainers/ci)
- [Image Metadata Specification](https://containers.dev/implementors/spec/#image-metadata)
- [Docker Layer Caching](https://docs.docker.com/build/cache/)
- [Supply-Chain Security Guide](../SKILL.md#best-practices)
