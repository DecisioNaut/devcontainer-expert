# devcontainer.json Schema Complete Reference

This document provides comprehensive documentation for all properties available in `devcontainer.json` based on the official [Development Containers Specification](https://containers.dev/).

## Table of Contents

- [General Properties](#general-properties)
- [Container Source](#container-source)
- [Lifecycle Scripts](#lifecycle-scripts)
- [Port Configuration](#port-configuration)
- [Environment Variables](#environment-variables)
- [User Configuration](#user-configuration)
- [Features](#features)
- [Build Options](#build-options)
- [Docker Compose](#docker-compose)
- [Container Runtime](#container-runtime)
- [Host Requirements](#host-requirements)
- [Tool Customizations](#tool-customizations)
- [Variable Substitution](#variable-substitution)

## General Properties

### `name`
**Type:** `string`  
**Description:** Display name for the dev container shown in the UI.

```json
{
  "name": "My Awesome Project"
}
```

### `$schema`
**Type:** `string` (URI)  
**Description:** JSON schema reference for validation and IntelliSense.

```json
{
  "$schema": "https://raw.githubusercontent.com/devcontainers/spec/main/schemas/devContainer.schema.json"
}
```

### `workspaceFolder`
**Type:** `string`  
**Description:** Path where the workspace folder is mounted inside the container.

**Default:** `/workspaces/<project-name>` (for image/Dockerfile) or as specified in docker-compose.yml (for Compose)

```json
{
  "workspaceFolder": "/workspace"
}
```

### `workspaceMount`
**Type:** `string`  
**Description:** Custom mount configuration for the workspace folder using Docker's `--mount` syntax.

```json
{
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind,consistency=cached"
}
```

## Container Source

You must specify exactly **one** container source: `image`, `build`/`dockerfile`, or `dockerComposeFile`.

### Using a Pre-built Image

#### `image`
**Type:** `string`  
**Description:** Docker image to use for the container.

```json
{
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20"
}
```

### Using a Dockerfile

#### `build`
**Type:** `object`  
**Description:** Docker build configuration.

**Properties:**
- `dockerfile` (string, required): Path to Dockerfile relative to `.devcontainer/`
- `context` (string): Build context path relative to `.devcontainer/` (default: `.`)
- `target` (string): Multi-stage build target
- `args` (object): Build arguments (key-value pairs)
- `cacheFrom` (string or array): Image(s) to use as cache source
- `options` (array): Additional docker build arguments

```json
{
  "build": {
    "dockerfile": "Dockerfile",
    "context": "..",
    "target": "development",
    "args": {
      "NODE_VERSION": "20",
      "VARIANT": "bullseye"
    },
    "cacheFrom": [
      "ghcr.io/myorg/myproject:cache"
    ],
    "options": ["--no-cache"]
  }
}
```

#### `dockerFile` (deprecated)
**Type:** `string`  
**Description:** Legacy property. Use `build.dockerfile` instead.

### Using Docker Compose

#### `dockerComposeFile`
**Type:** `string` or `array`  
**Description:** Path(s) to docker-compose file(s). When using an array, files are merged in order.

```json
{
  "dockerComposeFile": ["docker-compose.yml", "docker-compose.dev.yml"]
}
```

#### `service`
**Type:** `string` (required with `dockerComposeFile`)  
**Description:** The service name to connect to.

```json
{
  "service": "app"
}
```

#### `runServices`
**Type:** `array`  
**Description:** Services to start. If omitted, all services defined in Compose file(s) start.

```json
{
  "runServices": ["app", "db", "redis"]
}
```

## Lifecycle Scripts

Lifecycle scripts run at different stages of the container lifecycle. Each can be:
- **String**: Runs in a shell (e.g., `"npm install"`)
- **Array**: Runs as single command without shell (e.g., `["npm", "install"]`)
- **Object**: Runs multiple commands in parallel

### `initializeCommand`
**Type:** `string` | `array` | `object`  
**Description:** Runs on the **local machine** before creating the container.

```json
{
  "initializeCommand": "echo 'Preparing local environment'"
}
```

**Parallel execution:**
```json
{
  "initializeCommand": {
    "check-docker": "docker --version",
    "prepare-env": ["node", "scripts/prepare.js"]
  }
}
```

### `onCreateCommand`
**Type:** `string` | `array` | `object`  
**Description:** Runs once when the container is created. Runs after `initializeCommand`.

```json
{
  "onCreateCommand": "mkdir -p /workspace/temp"
}
```

### `updateContentCommand`
**Type:** `string` | `array` | `object`  
**Description:** Runs when creating the container and when workspace content is updated. Runs after `onCreateCommand`.

```json
{
  "updateContentCommand": "git submodule update --init"
}
```

### `postCreateCommand`
**Type:** `string` | `array` | `object`  
**Description:** Runs after creating the container. Most common for installing dependencies. Runs after `updateContentCommand`.

```json
{
  "postCreateCommand": "npm install"
}
```

**Parallel installation:**
```json
{
  "postCreateCommand": {
    "backend": "cd backend && npm install",
    "frontend": "cd frontend && npm install"
  }
}
```

### `postStartCommand`
**Type:** `string` | `array` | `object`  
**Description:** Runs every time the container starts. Runs after `postCreateCommand`.

```json
{
  "postStartCommand": "npm run start-services"
}
```

### `postAttachCommand`
**Type:** `string` | `array` | `object`  
**Description:** Runs every time you attach to the container. Runs after `postStartCommand`.

```json
{
  "postAttachCommand": "echo 'Welcome to the dev container!'"
}
```

### `waitFor`
**Type:** `string`  
**Description:** Which lifecycle script to wait for before starting the UI.

**Options:** `initializeCommand` | `onCreateCommand` | `updateContentCommand` | `postCreateCommand` | `postStartCommand`  
**Default:** `updateContentCommand`

```json
{
  "waitFor": "postCreateCommand"
}
```

## Port Configuration

### `forwardPorts`
**Type:** `array` of integers or strings  
**Description:** Ports to forward from container to local machine.

```json
{
  "forwardPorts": [3000, 5000, "8080-8090"]
}
```

**String format:** `"host:container"` or port ranges

```json
{
  "forwardPorts": [3000, "4000:3001", "8080-8090"]
}
```

### `appPort` (deprecated)
**Type:** `integer` | `string` | `array`  
**Description:** Deprecated in favor of `forwardPorts`. Application ports exposed by the container.

### `portsAttributes`
**Type:** `object`  
**Description:** Configure behavior for specific ports, port ranges, or patterns.

**Keys:** Port number, port range (e.g., `"8000-9000"`), or regex pattern (e.g., `".+\\/server\\.js"`)

**Properties per port:**
- `label` (string): Display name in UI
- `protocol` (string): `"http"` | `"https"`
- `onAutoForward` (string): Action when port is auto-forwarded
  - `"notify"` (default): Show notification
  - `"openBrowser"`: Open in browser
  - `"openBrowserOnce"`: Open in browser once per session
  - `"openPreview"`: Open preview in same window
  - `"silent"`: No notification
  - `"ignore"`: Don't auto-forward
- `elevateIfNeeded` (boolean): Auto-elevate for privileged ports (< 1024)
- `requireLocalPort` (boolean): Show modal if local port unavailable

```json
{
  "portsAttributes": {
    "3000": {
      "label": "Web Application",
      "onAutoForward": "openBrowser",
      "protocol": "https",
      "elevateIfNeeded": true,
      "requireLocalPort": true
    },
    "5000-5999": {
      "onAutoForward": "ignore"
    },
    ".+\\/api-server\\.js": {
      "label": "API Server",
      "onAutoForward": "notify"
    }
  }
}
```

### `otherPortsAttributes`
**Type:** `object`  
**Description:** Default properties for ports not explicitly configured in `portsAttributes`.

**Properties:** Same as `portsAttributes` (except no port key)

```json
{
  "otherPortsAttributes": {
    "onAutoForward": "silent"
  }
}
```

## Environment Variables

### `containerEnv`
**Type:** `object`  
**Description:** Environment variables set in the container for all processes.

```json
{
  "containerEnv": {
    "NODE_ENV": "development",
    "DATABASE_URL": "postgresql://db:5432/mydb"
  }
}
```

### `remoteEnv`
**Type:** `object`  
**Description:** Environment variables for remote processes spawned by the editor/IDE and lifecycle scripts. Supports `null` values to unset variables.

```json
{
  "remoteEnv": {
    "EDITOR": "code --wait",
    "PATH": "${containerEnv:PATH}:/custom/bin",
    "UNWANTED_VAR": null
  }
}
```

## User Configuration

### `remoteUser`
**Type:** `string`  
**Description:** User for spawning processes inside the container (lifecycle scripts, IDE server processes).

**Default:** Container's default user

```json
{
  "remoteUser": "vscode"
}
```

### `containerUser`
**Type:** `string`  
**Description:** User for starting the container itself.

**Default:** Image's default user

```json
{
  "containerUser": "node"
}
```

### `updateRemoteUserUID`
**Type:** `boolean`  
**Description:** On Linux, update the container user's UID/GID to match the local user's UID/GID.

**Default:** `true` when opening from local folder, `false` otherwise

```json
{
  "updateRemoteUserUID": true
}
```

### `userEnvProbe`
**Type:** `string`  
**Description:** How to probe the user's shell environment.

**Options:**
- `"none"` - Don't probe
- `"loginShell"` - Source login shell profile
- `"loginInteractiveShell"` (default) - Source login and interactive profiles
- `"interactiveShell"` - Source interactive profile only

```json
{
  "userEnvProbe": "loginShell"
}
```

## Features

### `features`
**Type:** `object`  
**Description:** Dev Container Features to install. Keys are Feature IDs, values are option objects.

```json
{
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    },
    "ghcr.io/devcontainers/features/docker-in-docker:2": {
      "version": "latest",
      "moby": true
    },
    "ghcr.io/devcontainers/features/github-cli:1": {}
  }
}
```

**Feature Version Pinning:**
- `"ghcr.io/devcontainers/features/node:1"` - Latest v1.x.x
- `"ghcr.io/devcontainers/features/node:1.2"` - Latest v1.2.x
- `"ghcr.io/devcontainers/features/node:1.2.3"` - Exact version

Browse Features at https://containers.dev/features

### `overrideFeatureInstallOrder`
**Type:** `array`  
**Description:** Control the order in which Features are installed. Array of Feature IDs (without version).

```json
{
  "features": {
    "ghcr.io/devcontainers/features/common-utils:1": {},
    "ghcr.io/devcontainers/features/node:1": {}
  },
  "overrideFeatureInstallOrder": [
    "ghcr.io/devcontainers/features/common-utils",
    "ghcr.io/devcontainers/features/node"
  ]
}
```

## Build Options

See [Container Source - Using a Dockerfile](#using-a-dockerfile) for build configuration details.

## Docker Compose

See [Container Source - Using Docker Compose](#using-docker-compose) for Compose-specific properties.

### `shutdownAction` (Compose)
**Type:** `string`  
**Description:** Action when disconnecting from the container.

**Options:**
- `"stopCompose"` (default) - Stop all Compose services
- `"none"` - Leave containers running

```json
{
  "shutdownAction": "none"
}
```

## Container Runtime

### `mounts`
**Type:** `array`  
**Description:** Additional mounts for the container. Can be Docker mount objects or strings using `--mount` syntax.

```json
{
  "mounts": [
    "source=${localEnv:HOME}/.ssh,target=/home/node/.ssh,readonly,type=bind",
    {
      "type": "bind",
      "source": "${localWorkspaceFolder}/../shared",
      "target": "${containerWorkspaceFolder}/shared"
    },
    {
      "type": "volume",
      "source": "node-modules",
      "target": "/workspace/node_modules"
    }
  ]
}
```

### `runArgs`
**Type:** `array`  
**Description:** Additional arguments passed to `docker run` or `docker create`.

```json
{
  "runArgs": [
    "--network=host",
    "--add-host=api.local:127.0.0.1",
    "--env-file=.env.local"
  ]
}
```

### `init`
**Type:** `boolean`  
**Description:** Pass `--init` flag to Docker. Uses [tini](https://github.com/krallin/tini) for proper PID 1 signal handling.

**Recommended:** `true` for applications that spawn child processes

```json
{
  "init": true
}
```

### `privileged`
**Type:** `boolean`  
**Description:** Run container in privileged mode.

**⚠️ Security Warning:** Only use when absolutely necessary (e.g., Docker-in-Docker)

```json
{
  "privileged": true
}
```

### `capAdd`
**Type:** `array`  
**Description:** Linux capabilities to add to the container.

```json
{
  "capAdd": ["SYS_PTRACE", "NET_ADMIN"]
}
```

Common capabilities:
- `SYS_PTRACE` - Debugging with ptrace (e.g., gdb)
- `NET_ADMIN` - Network administration
- `SYS_ADMIN` - Various administrative operations

### `securityOpt`
**Type:** `array`  
**Description:** Docker security options.

```json
{
  "securityOpt": ["seccomp=unconfined", "apparmor=unconfined"]
}
```

**⚠️ Security Warning:** Relaxing security should be done carefully

### `overrideCommand`
**Type:** `boolean`  
**Description:** Whether to override the default command from the image.

**Default:** `true` for image/Dockerfile, `false` for Compose

```json
{
  "overrideCommand": false
}
```

Set to `false` to preserve the image's entrypoint/command.

### `shutdownAction` (Non-Compose)
**Type:** `string`  
**Description:** Action when disconnecting.

**Options:**
- `"stopContainer"` (default) - Stop the container
- `"none"` - Leave container running

```json
{
  "shutdownAction": "none"
}
```

## Host Requirements

### `hostRequirements`
**Type:** `object`  
**Description:** Minimum host machine hardware requirements.

**Properties:**
- `cpus` (integer): Number of required CPU cores (minimum: 1)
- `memory` (string): Required RAM (e.g., `"8gb"`, `"4096mb"`)
- `storage` (string): Required disk space (e.g., `"32gb"`)
- `gpu` (boolean | string | object): GPU requirements

```json
{
  "hostRequirements": {
    "cpus": 4,
    "memory": "8gb",
    "storage": "50gb",
    "gpu": true
  }
}
```

### GPU Configuration

**Simple:**
```json
{
  "hostRequirements": {
    "gpu": true  // Required
  }
}
```

```json
{
  "hostRequirements": {
    "gpu": "optional"  // Optional
  }
}
```

**Detailed:**
```json
{
  "hostRequirements": {
    "gpu": {
      "cores": 2,
      "memory": "4gb"
    }
  }
}
```

## Secrets

### `secrets`
**Type:** `object`  
**Description:** Document recommended secrets for the dev container. This is **documentation only** - it doesn't create or inject secrets.

**Properties per secret:**
- `description` (string): What the secret is for
- `documentationUrl` (string): Link to documentation

```json
{
  "secrets": {
    "GITHUB_TOKEN": {
      "description": "Personal access token for GitHub API access",
      "documentationUrl": "https://docs.github.com/en/authentication"
    },
    "DATABASE_PASSWORD": {
      "description": "Password for PostgreSQL database"
    }
  }
}
```

Users must provide secrets via:
- `remoteEnv` with `${localEnv:SECRET_NAME}`
- Mounted files
- Docker secrets (when using Compose)

## Tool Customizations

### `customizations`
**Type:** `object`  
**Description:** Tool-specific configuration. Each tool uses a unique subproperty.

### VS Code Customizations

#### `customizations.vscode.extensions`
**Type:** `array`  
**Description:** VS Code extensions to install in the container.

```json
{
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "ms-python.python"
      ]
    }
  }
}
```

#### `customizations.vscode.settings`
**Type:** `object`  
**Description:** VS Code settings for the container.

```json
{
  "customizations": {
    "vscode": {
      "settings": {
        "terminal.integrated.defaultProfile.linux": "zsh",
        "editor.formatOnSave": true,
        "python.defaultInterpreterPath": "/usr/local/bin/python"
      }
    }
  }
}
```

### Other Tool Customizations

Other supporting tools can have their own customizations:

```json
{
  "customizations": {
    "vscode": {
      "extensions": ["ms-python.python"]
    },
    "codespaces": {
      "openFiles": ["README.md", "src/main.py"]
    }
  }
}
```

## Variable Substitution

Dev containers support variable substitution in string values using `${variable}` syntax.

### Available Variables

#### Local Machine Variables
- `${localEnv:NAME}` - Environment variable from local machine
- `${localWorkspaceFolder}` - Workspace folder path on local machine
- `${localWorkspaceFolderBasename}` - Basename of workspace folder on local machine

#### Container Variables
- `${containerEnv:NAME}` - Environment variable from container (set via `containerEnv`)
- `${containerWorkspaceFolder}` - Workspace folder path inside container
- `${containerWorkspaceFolderBasename}` - Basename of workspace folder inside container

### Default Values

Use `:` to specify a default value:

```json
{
  "remoteEnv": {
    "API_URL": "${localEnv:API_URL:https://api.example.com}"
  }
}
```

If `API_URL` is not set locally, `https://api.example.com` is used.

### Usage Examples

**Passing local environment to container:**
```json
{
  "remoteEnv": {
    "GITHUB_TOKEN": "${localEnv:GITHUB_TOKEN}",
    "NPM_TOKEN": "${localEnv:NPM_TOKEN}"
  }
}
```

**Extending container PATH:**
```json
{
  "remoteEnv": {
    "PATH": "${containerEnv:PATH}:${containerWorkspaceFolder}/bin"
  }
}
```

**Mounting local SSH keys:**
```json
{
  "mounts": [
    "source=${localEnv:HOME}/.ssh,target=/home/node/.ssh,readonly,type=bind"
  ]
}
```

**Referencing workspace folders:**
```json
{
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace/${localWorkspaceFolderBasename},type=bind"
}
```

## Additional Properties

### `additionalProperties`
**Type:** `object`  
**Description:** The spec allows additional custom properties for tool-specific extensions. However, prefer using `customizations` for tool-specific configuration.

## Schema Validation

Add `$schema` property for editor validation and IntelliSense:

```json
{
  "$schema": "https://raw.githubusercontent.com/devcontainers/spec/main/schemas/devContainer.schema.json",
  "name": "My Dev Container"
}
```

## Resources

- **Official Schema**: https://github.com/devcontainers/spec/blob/main/schemas/devContainer.schema.json
- **JSON Reference**: https://containers.dev/implementors/json_reference/
- **Specification**: https://containers.dev/
- **VS Code Docs**: https://code.visualstudio.com/docs/devcontainers/containers

## Best Practices

1. **Always specify `$schema`** for validation and IntelliSense
2. **Use Features** instead of installing tools in lifecycle scripts when possible
3. **Pin versions** of Features and base images for reproducibility
4. **Document secrets** using the `secrets` property
5. **Use `waitFor`** wisely - only wait for commands that block essential dependencies
6. **Minimize `privileged`** and `securityOpt` relaxations for security
7. **Use `init: true`** for applications that spawn child processes
8. **Leverage parallel execution** in lifecycle scripts for faster builds
9. **Use variable substitution** to keep configuration portable
10. **Specify `hostRequirements`** when you have specific hardware needs
