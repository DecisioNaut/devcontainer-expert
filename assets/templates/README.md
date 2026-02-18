# Dev Container Templates

This directory contains practical, copy-paste devcontainer.json templates for common scenarios. Each template is fully commented and follows best practices from the [JSON Schema Reference](../references/JSON_SCHEMA_REFERENCE.md).

## Available Templates

### [basic-node.json](basic-node.json)
Simple Node.js/TypeScript setup with:
- Official Microsoft Node.js image
- ESLint and Prettier
- Port forwarding configuration
- Basic lifecycle script

**Best for:** Quick Node.js projects, getting started with dev containers

---

### [python-with-features.json](python-with-features.json)
Python development with Features:
- Base Ubuntu image + Python Feature
- Git, GitHub CLI, and common utilities
- Feature installation order control
- Jupyter and Flask port configuration
- Python extensions and Black formatter

**Best for:** Python data science, web development, learning Features

---

### [docker-compose.json](docker-compose.json)
Multi-service full-stack application:
- Docker Compose integration
- PostgreSQL and Redis services
- Parallel dependency installation
- Service-specific port configuration
- Selective service startup

**Best for:** Full-stack apps, microservices, database-backed applications

---

### [advanced-lifecycle.json](advanced-lifecycle.json)
Complex lifecycle script patterns:
- All lifecycle hooks demonstrated
- Local prerequisite checks
- Parallel command execution
- Background service startup
- Environment variable patterns
- Signal handling with `init`

**Best for:** Complex build processes, monorepos, learning lifecycle scripts

---

### [monorepo-polyglot.json](monorepo-polyglot.json)
Multi-language monorepo setup:
- Node.js, Python, Go, and Rust
- Docker-in-Docker for containerized workflows
- Parallel package installation
- Multiple port forwarding
- Host hardware requirements
- Multi-language extensions

**Best for:** Large monorepos, polyglot teams, full-stack platforms

---

### [security-hardened.json](security-hardened.json)
Security-focused configuration:
- Non-root user setup
- Read-only secret mounting
- Environment variable secrets
- Secrets documentation
- Minimal capabilities
- No privileged mode
- File exclusion patterns

**Best for:** Production-like environments, security-conscious teams, best practices reference

---

### [dockerfile-multistage.json](dockerfile-multistage.json)
Custom Dockerfile with multi-stage builds:
- Multi-stage build target selection
- Build cache configuration
- Build arguments
- Custom workspace mounts
- Host requirements
- Optimized build process

**Best for:** Custom images, optimized builds, CI/CD integration

---

## Usage

1. **Copy a template** to your project's `.devcontainer/devcontainer.json`
2. **Customize** the values (ports, commands, extensions) for your project
3. **Remove comments** if desired (JSON with Comments is supported)
4. **Reopen in Container** using VS Code's Dev Containers extension

## Template Selection Guide

| Scenario | Recommended Template |
|----------|---------------------|
| Simple Node.js app | [basic-node.json](basic-node.json) |
| Python project | [python-with-features.json](python-with-features.json) |
| App + database | [docker-compose.json](docker-compose.json) |
| Complex build | [advanced-lifecycle.json](advanced-lifecycle.json) |
| Multiple languages | [monorepo-polyglot.json](monorepo-polyglot.json) |
| Security focus | [security-hardened.json](security-hardened.json) |
| Custom Dockerfile | [dockerfile-multistage.json](dockerfile-multistage.json) |

## Additional Resources

- **[SKILL.md](../../SKILL.md)** - Core concepts and workflows
- **[JSON Schema Reference](../references/JSON_SCHEMA_REFERENCE.md)** - Complete property documentation
- **[Security Guide](../references/SECURITY.md)** - Security best practices
- **[Official Templates](https://containers.dev/templates)** - Community templates
- **[Official Features](https://containers.dev/features)** - Available Features

## Combining Templates

You can mix patterns from multiple templates:

```json
{
  // Use Docker Compose (from docker-compose.json)
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  
  // Add security hardening (from security-hardened.json)
  "remoteUser": "node",
  "mounts": [
    "source=${localEnv:HOME}/.ssh,target=/home/node/.ssh,readonly,type=bind"
  ],
  
  // Add complex lifecycle (from advanced-lifecycle.json)
  "postCreateCommand": {
    "install-backend": "npm install",
    "install-frontend": "cd frontend && npm install"
  }
}
```

## Contributing

Found an issue or have a suggestion for a new template? See [CONTRIBUTING.md](../../CONTRIBUTING.md) or open an issue on the repository.
