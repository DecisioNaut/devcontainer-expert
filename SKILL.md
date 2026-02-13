---
name: devcontainer-expert
description: Expert guidance for Development Containers (devcontainers) in VS Code. Create, configure, and manage devcontainer.json files, use Templates and Features, integrate with Docker/Docker Compose, and troubleshoot common issues. Use when working with devcontainers, containerized development environments, or when user mentions devcontainer.json, dev containers, VS Code containers, or Docker development.
license: MIT
metadata:
  version: "1.0.0"
  author: devcontainer-expert contributors
compatibility: Requires Docker (Docker Desktop on Mac/Windows or Docker CE/EE on Linux), VS Code with Dev Containers extension
---

# Development Containers Expert

Expert guidance for working with Development Containers (devcontainers) - a standardized way to create fully-featured, containerized development environments.

## When to Use This Skill

Activate this skill when:
- Creating or configuring devcontainer.json files
- Setting up containerized development environments in VS Code
- Working with dev container Templates or Features
- Troubleshooting devcontainer issues
- Integrating development containers with Docker or Docker Compose
- User mentions: devcontainers, dev containers, devcontainer.json, VS Code containers, containerized development, or development environment setup

## What Are Development Containers?

A development container (dev container) is a running Docker container with a well-defined tool/runtime stack and its prerequisites. The configuration is defined in a `devcontainer.json` file that tells VS Code (or other supporting tools) how to access or create a development container.

**Key Benefits:**
- **Consistency**: Same environment for all developers
- **Repeatability**: Quickly recreate environments
- **Isolation**: Separate tools/libraries per project
- **Onboarding**: New developers get productive faster
- **CI/CD Integration**: Same environment for local dev and testing

**The Specification:**
Dev containers follow the open [Development Containers Specification](https://containers.dev/) maintained by the devcontainers community.

## Prerequisites

### Required Software

1. **Docker**
   - **Windows**: Docker Desktop 2.0+ (Windows 10 Pro/Enterprise or Windows 10 Home 2004+ with WSL 2)
   - **macOS**: Docker Desktop 2.0+
   - **Linux**: Docker CE/EE 18.06+ and Docker Compose 1.21+

2. **VS Code**
   - Visual Studio Code or VS Code Insiders
   - Dev Containers extension (`ms-vscode-remote.remote-containers`)

3. **System Resources**
   - Minimum: 1 GB RAM
   - Recommended: 2 GB RAM and 2-core CPU

### Installation Steps

**Windows/macOS:**
1. Install Docker Desktop from https://www.docker.com/products/docker-desktop
2. For Windows WSL 2: Enable "Use the WSL 2 based engine" in Docker Desktop settings
3. Install VS Code from https://code.visualstudio.com/
4. Install the Dev Containers extension from the VS Code Extensions marketplace

**Linux:**
1. Install Docker CE/EE following official instructions for your distribution
2. Add your user to the docker group: `sudo usermod -aG docker $USER`
3. Sign out and back in for changes to take effect
4. Install VS Code
5. Install the Dev Containers extension

## Creating a devcontainer.json File

The `devcontainer.json` file is the core configuration for your dev container. It can be placed in one of two locations:
- `.devcontainer/devcontainer.json` (recommended for complex setups)
- `.devcontainer.json` (root of project)

### Quick Start: Using a Template

The easiest way to create a dev container is using a pre-built Template:

1. **Open Command Palette** in VS Code: `F1` or `Cmd/Ctrl+Shift+P`
2. **Run**: `Dev Containers: Add Dev Container Configuration Files...`
3. **Select a Template** based on your tech stack (e.g., Node.js, Python, Go, Java)
4. **Customize with Features** (optional) - add tools like Git, GitHub CLI, Docker-in-Docker
5. **Reopen in Container**: Run `Dev Containers: Reopen in Container`

VS Code will build the container and reopen your workspace inside it.

### Basic devcontainer.json Structure

**Minimal Example (Using Pre-built Image):**

```json
{
  "name": "My Project",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20",
  "forwardPorts": [3000],
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode"
      ]
    }
  }
}
```

**Using a Dockerfile:**

```json
{
  "name": "My Custom Container",
  "build": {
    "dockerfile": "Dockerfile",
    "context": "..",
    "args": {
      "VARIANT": "18"
    }
  },
  "forwardPorts": [3000, 5000],
  "postCreateCommand": "npm install",
  "customizations": {
    "vscode": {
      "settings": {
        "terminal.integrated.defaultProfile.linux": "bash"
      },
      "extensions": ["dbaeumer.vscode-eslint"]
    }
  }
}
```

**Using Docker Compose:**

```json
{
  "name": "My Multi-Container App",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "forwardPorts": [3000],
  "postCreateCommand": "npm install",
  "customizations": {
    "vscode": {
      "extensions": ["dbaeumer.vscode-eslint"]
    }
  }
}
```

### Key Configuration Properties

**Container Definition:**
- `name`: Display name for the container
- `image`: Pre-built Docker image to use
- `build`: Build configuration (dockerfile, context, args)
- `dockerComposeFile`: Path to docker-compose.yml file
- `service`: Docker Compose service name to connect to

**Environment Setup:**
- `workspaceFolder`: Path where workspace is mounted (default: `/workspaces/<project-name>`)
- `workspaceMount`: Custom workspace mount configuration
- `mounts`: Additional volume/bind mounts
- `containerEnv`: Environment variables for the container
- `remoteUser`: User to run as inside container (default: root or image default)

**Port Forwarding:**
- `forwardPorts`: Array of ports to forward (e.g., `[3000, 8080]`)
- `portsAttributes`: Configure port labels, protocols, and visibility

**Lifecycle Scripts:**
- `onCreateCommand`: Runs once when container is created
- `updateContentCommand`: Runs when container config changes
- `postCreateCommand`: Runs after container creation (often used for `npm install`, `pip install`)
- `postStartCommand`: Runs every time the container starts
- `postAttachCommand`: Runs every time you attach to the container

**VS Code Customization:**
- `customizations.vscode.extensions`: Extensions to install in the container
- `customizations.vscode.settings`: VS Code settings for the container

## Working with Dev Container Features

Features are self-contained, shareable units of installation code that add tools, runtimes, or libraries to your container.

### Adding Features

**Method 1: During Setup**
When running `Dev Containers: Add Dev Container Configuration Files...`, you'll be prompted to select Features.

**Method 2: Add to Existing Config**
1. Open Command Palette: `F1`
2. Run: `Dev Containers: Configure Container Features`
3. Select Features to add

**Method 3: Edit devcontainer.json Directly**

```json
{
  "name": "My Project",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    },
    "ghcr.io/devcontainers/features/git:1": {
      "version": "latest"
    },
    "ghcr.io/devcontainers/features/github-cli:1": {},
    "ghcr.io/devcontainers/features/docker-in-docker:2": {
      "version": "latest",
      "moby": true
    }
  }
}
```

### Popular Features

- **ghcr.io/devcontainers/features/node** - Node.js
- **ghcr.io/devcontainers/features/python** - Python
- **ghcr.io/devcontainers/features/go** - Go
- **ghcr.io/devcontainers/features/git** - Git
- **ghcr.io/devcontainers/features/github-cli** - GitHub CLI
- **ghcr.io/devcontainers/features/docker-in-docker** - Docker inside container
- **ghcr.io/devcontainers/features/kubectl-helm-minikube** - Kubernetes tools
- **ghcr.io/devcontainers/features/aws-cli** - AWS CLI
- **ghcr.io/devcontainers/features/azure-cli** - Azure CLI

Browse all Features at: https://containers.dev/features

### Pinning Feature Versions

Features follow semantic versioning. You can pin to specific versions:

```json
"features": {
  "ghcr.io/devcontainers/features/node:1": {},           // Latest v1.x.x
  "ghcr.io/devcontainers/features/python:1.0": {},       // Latest v1.0.x
  "ghcr.io/devcontainers/features/go:1.0.0": {}          // Exact version 1.0.0
}
```

### Creating Custom Features

You can create your own Features using the [feature-starter template](https://github.com/devcontainers/feature-starter). Each Feature is a folder with:
- `devcontainer-feature.json` - Metadata and options
- `install.sh` - Installation script

## Common Workflows

**Open Local Project in Container:**
1. Open project in VS Code
2. Run: `Dev Containers: Reopen in Container` (F1)
3. Select Template if no devcontainer.json exists
4. VS Code builds and reopens in container

**Clone Repository in Container Volume** (better performance on Windows/macOS):
1. Run: `Dev Containers: Clone Repository in Container Volume...`
2. Enter Git URL or select from GitHub
3. Choose Template if needed

**Open GitHub PR:**
1. Run: `Dev Containers: Clone Repository in Container Volume...`
2. Paste GitHub PR URL
3. VS Code opens PR with GitHub Pull Requests extension

**Rebuild Container** (after config changes):
- `Dev Containers: Rebuild Container`
- OR `Dev Containers: Rebuild Container Without Cache` (full rebuild)

**Attach to Running Container:**
1. Run: `Dev Containers: Attach to Running Container...`
2. Select container from list

## Advanced Configuration

### Using Docker Compose

**docker-compose.yml:**
```yaml
services:
  app:
    build: .
    volumes:
      - .:/workspace:cached
    command: sleep infinity
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: example
```

**devcontainer.json:**
```json
{
  "name": "App with Database",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "postCreateCommand": "npm install"
}
```

### Non-Root User

```json
{
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20",
  "remoteUser": "node",
  "features": {
    "ghcr.io/devcontainers/features/common-utils:2": {
      "username": "node"
    }
  }
}
```

### Custom Mounts and Environment

```json
{
  "image": "mcr.microsoft.com/devcontainers/python:3.11",
  "mounts": [
    "source=${localEnv:HOME}/.ssh,target=/home/node/.ssh,readonly,type=bind"
  ],
  "containerEnv": {
    "DATABASE_URL": "postgresql://localhost/mydb"
  }
}
```

### Lifecycle Scripts

```json
{
  "onCreateCommand": "echo 'First time setup'",
  "updateContentCommand": "npm install",
  "postCreateCommand": "npm run setup",
  "postStartCommand": "npm run start-services"
}
```

Use strings for simple commands, arrays for args, or objects for multiple commands.

## Troubleshooting

**Docker Not Running**: Ensure Docker Desktop is running; check `docker ps` works. On Linux: `sudo systemctl status docker`

**Port Already in Use**: Check with `lsof -i :<port>` (macOS/Linux) or `netstat -ano | findstr :<port>` (Windows). Change port or stop conflicting service.

**Slow File Performance (Windows/macOS)**: Use `Clone Repository in Container Volume` instead of bind mounts. Add `:cached` or `:delegated` flags to mounts.

**Extension Not Working**: Add to `customizations.vscode.extensions` in devcontainer.json and rebuild container. Check extension docs for additional dependencies.

**Changes Not Applied**: Run `Dev Containers: Rebuild Container` or `Rebuild Container Without Cache` for Dockerfile changes.

**Git Credentials Not Working**: VS Code forwards credentials automatically. For SSH, mount: `"mounts": ["source=${localEnv:HOME}/.ssh,target=/home/node/.ssh,readonly,type=bind"]`

**Permission Denied**: Set `remoteUser` in devcontainer.json. Use `common-utils` Feature for proper user setup.

**Container Build Fails**: Check logs with `Dev Containers: Show Container Log`. Test Dockerfile separately: `docker build -f .devcontainer/Dockerfile .`

**OutOfMemory Error**: Increase Docker memory in settings. Use multi-stage builds and run `docker system prune` to free space.

## Best Practices

1. **Version Control devcontainer.json**: Always commit your `.devcontainer/` folder so all developers use the same environment

2. **Use Pre-built Images**: Start with official images from `mcr.microsoft.com/devcontainers/` for faster builds

3. **Layer Features Thoughtfully**: Add most common/stable Features first, more volatile ones last

4. **Pin Versions**: Pin Feature and base image versions for reproducibility

5. **Minimize Image Size**: Use multi-stage builds and clean up package caches

6. **Security**:
   - Run as non-root user when possible
   - Keep base images updated
   - Don't commit secrets in devcontainer.json (use environment variables or mounts)

7. **Performance**:
   - Use volume mounts for better performance on Windows/macOS
   - Leverage layer caching in Dockerfiles
   - Use `postCreateCommand` for expensive one-time setup

8. **Documentation**: Add comments to devcontainer.json explaining custom configurations

9. **Test Regularly**: Periodically test that new developers can build the container from scratch

10. **Use .dockerignore**: Exclude unnecessary files from Docker build context

## Using the Dev Container CLI

The Dev Container CLI (`@devcontainers/cli`) allows you to work with dev containers outside of VS Code.

### Installation

```bash
npm install -g @devcontainers/cli
```

### Common Commands

```bash
# Build a dev container
devcontainer build --workspace-folder .

# Open a dev container
devcontainer up --workspace-folder .

# Execute a command in the container
devcontainer exec --workspace-folder . npm test

# Apply a template
devcontainer templates apply --workspace-folder . \
  --template-id ghcr.io/devcontainers/templates/typescript-node:latest

# List available templates
devcontainer templates list
```

## CI/CD Integration

Use dev containers in CI/CD pipelines for consistent testing environments.

### GitHub Actions

Use the [devcontainers/ci](https://github.com/devcontainers/ci) action:

```yaml
name: CI
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build and run dev container
        uses: devcontainers/ci@v0.3
        with:
          runCmd: npm test
```

### Azure DevOps

Install the Dev Containers task from the marketplace and use:

```yaml
- task: DevContainers@0
  inputs:
    runCmd: 'npm test'
```

## Resources

- **Specification**: https://containers.dev/
- **VS Code Docs**: https://code.visualstudio.com/docs/devcontainers/containers
- **Templates**: https://containers.dev/templates
- **Features**: https://containers.dev/features
- **GitHub Org**: https://github.com/devcontainers
- **CLI Reference**: https://github.com/devcontainers/cli
- **Docker Images**: https://github.com/devcontainers/images

## Quick Reference

**VS Code Commands** (open with `F1` or `Cmd/Ctrl+Shift+P`):
- `Dev Containers: Reopen in Container`
- `Dev Containers: Rebuild Container`
- `Dev Containers: Add Dev Container Configuration Files...`
- `Dev Containers: Configure Container Features`
- `Dev Containers: Clone Repository in Container Volume...`
- `Dev Containers: Attach to Running Container...`
- `Dev Containers: Show Container Log`

**Key Configuration Locations**:
- `.devcontainer/devcontainer.json` (recommended)
- `.devcontainer.json` (alternative)
- Dockerfile: `.devcontainer/Dockerfile`
- Docker Compose: `.devcontainer/docker-compose.yml`
