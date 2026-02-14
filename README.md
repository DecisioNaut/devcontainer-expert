# Development Containers Expert

An Agent Skill providing expert guidance for working with Development Containers (devcontainers) in VS Code.

## Description

This skill provides comprehensive knowledge and step-by-step instructions for creating, configuring, and troubleshooting development containers. It covers the full devcontainer ecosystem including Templates, Features, Docker integration, VS Code workflows, and advanced topics like pre-building, Kubernetes, networking, and security.

**Level 2 Architecture**: Core patterns in [SKILL.md](SKILL.md) with in-depth coverage in [references/](references/).

## What You'll Learn

### Core Topics (SKILL.md)
- **Core Concepts**: Understanding development containers and when to use them
- **Configuration**: Creating and customizing devcontainer.json files
- **Templates & Features**: Using pre-built templates and adding tools with Features
- **Monorepos**: Multiple devcontainer strategies for monorepo projects
- **Docker Integration**: Docker Compose, non-root users, custom mounts
- **Common Workflows**: Open in container, clone in volume, rebuild, attach
- **Quick Troubleshooting**: Common issues and solutions

### Advanced Topics (references/)
- **[Pre-building Images](references/PREBUILDING.md)**: Image caching, CI/CD workflows, supply-chain security
- **[Kubernetes Integration](references/KUBERNETES.md)**: Attach to K8s pods, cluster development workflows
- **[Networking Fundamentals](references/NETWORKING.md)**: Docker network modes, port forwarding, DNS, proxies
- **[Advanced Networking](references/NETWORKING_ADVANCED.md)**: VPN, network security, service mesh, load balancing
- **[Volume Performance](references/VOLUME_PERFORMANCE.md)**: Optimize disk performance on Windows/macOS
- **[Container-in-Container](references/CONTAINER_IN_CONTAINER.md)**: Docker-in-Docker for CI/CD testing
- **[Security Hardening](references/SECURITY.md)**: Non-root users, secrets management, capabilities
- **[Dotfiles & Personalization](references/DOTFILES.md)**: Shell customization, tool configuration
- **[Recovery & Debugging](references/RECOVERY_DEBUGGING.md)**: Fix build failures, inspect containers

## Installation

### For Agents

Read [SKILL.md](SKILL.md) for core devcontainer patterns and common workflows. For advanced topics (pre-building, Kubernetes, networking, security, etc.), consult the specialized guides in [references/](references/).

### For Human Developers

This skill is designed for AI agents, but humans can also use it as a comprehensive reference guide:
- Start with [SKILL.md](SKILL.md) for core concepts and workflows
- Explore [references/](references/) for in-depth coverage of advanced scenarios

## Prerequisites

To work with devcontainers, you need:
- Docker (Docker Desktop on Mac/Windows, Docker CE/EE on Linux)
- Visual Studio Code with the Dev Containers extension
- 2GB RAM and 2-core CPU (recommended)

## Quick Start Example

Here's a simple devcontainer.json to get started:

```json
{
  "name": "My Project",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20",
  "forwardPorts": [3000],
  "features": {
    "ghcr.io/devcontainers/features/git:1": {},
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode"
      ]
    }
  },
  "postCreateCommand": "npm install"
}
```

## Topics Covered

### Core Topics (SKILL.md)
- ✅ devcontainer.json configuration
- ✅ Templates and Features
- ✅ Docker and Docker Compose integration
- ✅ Monorepo strategies (multiple containers)
- ✅ VS Code workflows and commands
- ✅ Lifecycle scripts (onCreate, postCreate, postStart, etc.)
- ✅ Port forwarding and networking basics
- ✅ Environment variables and mounts
- ✅ Git credentials sharing
- ✅ Common troubleshooting scenarios

### Advanced Topics (references/)
- ✅ Pre-building images for CI/CD
- ✅ Kubernetes pod attachment
- ✅ Custom Docker networks and DNS
- ✅ Volume performance optimization
- ✅ Docker-in-Docker (container-in-container)
- ✅ Security hardening and secrets management
- ✅ Dotfiles and personalization
- ✅ Recovery containers and debugging

## Resources

- **Specification**: https://containers.dev/
- **VS Code Documentation**: https://code.visualstudio.com/docs/devcontainers/containers
- **GitHub Organization**: https://github.com/devcontainers
- **Templates**: https://containers.dev/templates
- **Features**: https://containers.dev/features
- **CLI**: https://github.com/devcontainers/cli

## Contributing

This is an Agent Skill following the [Agent Skills Specification](https://agentskills.io/). 

**Level 2 Architecture**: This skill uses a Level 2 structure with core patterns in SKILL.md (~450 lines) and advanced topics in references/ (~300-650 lines each).

To contribute:
1. Fork this repository
2. For core patterns: Edit [SKILL.md](SKILL.md) (keep focused ~400-500 lines)
3. For advanced topics: Add or edit files in [references/](references/)
4. Ensure clear, actionable instructions with examples
5. Add appropriate cross-references between files
6. Test that all links work
7. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) for details.

## Specification Compliance

This skill follows the [Agent Skills Specification v1.0](https://agentskills.io/specification):
- ✅ Valid devcontainer-expert name (lowercase with hyphens)
- ✅ Structured YAML frontmatter with required fields
- ✅ Progressive disclosure (skill discovery via name/description)
- ✅ Clear activation criteria
- ✅ Level 2 architecture (main file ~475 lines + references/)
- ✅ Comprehensive coverage via specialized reference files
- ✅ Actionable, step-by-step guidance
- ✅ Cross-referenced for easy navigation

## Feedback

Issues and feedback can be submitted via GitHub Issues. For questions about the Agent Skills specification itself, visit https://agentskills.io/

---

**Made for AI agents** | **Useful for humans too** | **Following the Agent Skills Specification**
