# Development Containers Expert

An Agent Skill providing expert guidance for working with Development Containers (devcontainers) in VS Code.

**Repository**: https://github.com/DecisioNaut/devcontainer-expert

## Description

This skill provides comprehensive knowledge and step-by-step instructions for creating, configuring, and troubleshooting development containers. It covers the full devcontainer ecosystem including Templates, Features, Docker integration, VS Code workflows, and advanced topics like pre-building, Kubernetes, networking, and security.

**Level 2 Architecture**: Core patterns in [SKILL.md](devcontainer-expert/SKILL.md) with in-depth coverage in [references/](devcontainer-expert/references/).

## Repository Structure

```
devcontainer-expert/                  # Repository root
├── README.md                         # This file (GitHub documentation)
├── CHANGELOG.md                      # Version history
├── LICENSE                           # MIT License
├── .gitignore                        # Git configuration
└── devcontainer-expert/              # Agent Skill package
    ├── SKILL.md                      # Main skill file (533 lines)
    ├── assets/
    │   └── templates/                # 7 ready-to-use templates
    │       ├── README.md
    │       ├── basic-node.json
    │       ├── python-with-features.json
    │       ├── docker-compose.json
    │       ├── advanced-lifecycle.json
    │       ├── monorepo-polyglot.json
    │       ├── security-hardened.json
    │       └── dockerfile-multistage.json
    └── references/                   # Advanced topic guides
        ├── JSON_SCHEMA_REFERENCE.md  # Complete property documentation
        ├── PREBUILDING.md
        ├── KUBERNETES.md
        ├── NETWORKING.md
        ├── NETWORKING_ADVANCED.md
        ├── VOLUME_PERFORMANCE.md
        ├── CONTAINER_IN_CONTAINER.md
        ├── SECURITY.md
        ├── DOTFILES.md
        └── RECOVERY_DEBUGGING.md
```

The skill package is contained in the `devcontainer-expert/` subfolder, keeping the repository root clean for GitHub documentation.

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
- **[JSON Schema Complete Reference](devcontainer-expert/references/JSON_SCHEMA_REFERENCE.md)**: Comprehensive documentation of all devcontainer.json properties
- **[Pre-building Images](devcontainer-expert/references/PREBUILDING.md)**: Image caching, CI/CD workflows, supply-chain security
- **[Kubernetes Integration](devcontainer-expert/references/KUBERNETES.md)**: Attach to K8s pods, cluster development workflows
- **[Networking Fundamentals](devcontainer-expert/references/NETWORKING.md)**: Docker network modes, port forwarding, DNS, proxies
- **[Advanced Networking](devcontainer-expert/references/NETWORKING_ADVANCED.md)**: VPN, network security, service mesh, load balancing
- **[Volume Performance](devcontainer-expert/references/VOLUME_PERFORMANCE.md)**: Optimize disk performance on Windows/macOS
- **[Container-in-Container](devcontainer-expert/references/CONTAINER_IN_CONTAINER.md)**: Docker-in-Docker for CI/CD testing
- **[Security Hardening](devcontainer-expert/references/SECURITY.md)**: Non-root users, secrets management, capabilities
- **[Dotfiles & Personalization](devcontainer-expert/references/DOTFILES.md)**: Shell customization, tool configuration
- **[Recovery & Debugging](devcontainer-expert/references/RECOVERY_DEBUGGING.md)**: Fix build failures, inspect containers

## Installation

### For AI Agents (GitHub Copilot, Claude, etc.)

To install this skill in your agent environment:

1. **Clone or download** the skill package:
   ```bash
   git clone https://github.com/DecisioNaut/devcontainer-expert.git
   ```

2. **Configure your agent** to use the skill:
   - **GitHub Copilot**: Copy the `devcontainer-expert/` folder to your `~/.copilot/skills/` directory
   - **Claude Desktop**: Add to your agent's skills directory (refer to your agent's documentation)
   - **Custom agents**: Point your agent configuration to the `devcontainer-expert/SKILL.md` file

3. **Verify installation**: The skill activates when working with devcontainers or when you mention:
   - "devcontainer", "dev container", "development container"
   - "devcontainer.json"
   - ".devcontainer"
   - Docker development, VS Code containers

### For Human Developers

This skill is designed for AI agents, but humans can use it as a comprehensive reference guide:

- **Quick start**: Browse [devcontainer-expert/SKILL.md](devcontainer-expert/SKILL.md) for core concepts and workflows
- **Copy templates**: Check [devcontainer-expert/assets/templates/](devcontainer-expert/assets/templates/) for ready-to-use configurations
- **Deep dives**: Explore [devcontainer-expert/references/](devcontainer-expert/references/) for advanced scenarios
- **Property reference**: See [devcontainer-expert/references/JSON_SCHEMA_REFERENCE.md](devcontainer-expert/references/JSON_SCHEMA_REFERENCE.md) for all devcontainer.json properties

### File Locations

After installation, key files are located at:
- Main skill: `devcontainer-expert/SKILL.md`
- Templates: `devcontainer-expert/assets/templates/`
- References: `devcontainer-expert/references/`

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
- ✅ devcontainer.json configuration (all properties documented)
- ✅ Templates and Features (with installation order control)
- ✅ Docker and Docker Compose integration
- ✅ Monorepo strategies (multiple containers)
- ✅ VS Code workflows and commands
- ✅ Lifecycle scripts (with waitFor and parallel execution)
- ✅ Port forwarding and networking basics (with advanced attributes)
- ✅ Environment variables and mounts (containerEnv vs remoteEnv)
- ✅ Variable substitution (localEnv, containerEnv, workspace paths)
- ✅ User configuration (remoteUser, containerUser, UID mapping)
- ✅ Host requirements (CPU, memory, storage, GPU)
- ✅ Security options (privileged, capabilities, init)
- ✅ Secrets management documentation
- ✅ Git credentials sharing
- ✅ Common troubleshooting scenarios

### Advanced Topics (references/)
- **[JSON Schema Complete Reference](devcontainer-expert/references/JSON_SCHEMA_REFERENCE.md)**: Comprehensive documentation of all devcontainer.json properties
- **[Pre-building Images](devcontainer-expert/references/PREBUILDING.md)**: Image caching, CI/CD workflows, supply-chain security
- **[Kubernetes Integration](devcontainer-expert/references/KUBERNETES.md)**: Attach to K8s pods, cluster development workflows
- **[Networking Fundamentals](devcontainer-expert/references/NETWORKING.md)**: Docker network modes, port forwarding, DNS, proxies
- **[Advanced Networking](devcontainer-expert/references/NETWORKING_ADVANCED.md)**: VPN, network security, service mesh, load balancing
- **[Volume Performance](devcontainer-expert/references/VOLUME_PERFORMANCE.md)**: Optimize disk performance on Windows/macOS
- **[Container-in-Container](devcontainer-expert/references/CONTAINER_IN_CONTAINER.md)**: Docker-in-Docker for CI/CD testing
- **[Security Hardening](devcontainer-expert/references/SECURITY.md)**: Non-root users, secrets management, capabilities
- **[Dotfiles & Personalization](devcontainer-expert/references/DOTFILES.md)**: Shell customization, tool configuration
- **[Recovery & Debugging](devcontainer-expert/references/RECOVERY_DEBUGGING.md)**: Fix build failures, inspect containers

### Templates (assets/templates/)
Ready-to-use devcontainer.json templates:
- **[basic-node.json](devcontainer-expert/assets/templates/basic-node.json)**: Simple Node.js/TypeScript starter
- **[python-with-features.json](devcontainer-expert/assets/templates/python-with-features.json)**: Python with Features
- **[docker-compose.json](devcontainer-expert/assets/templates/docker-compose.json)**: Multi-service full-stack app
- **[advanced-lifecycle.json](devcontainer-expert/assets/templates/advanced-lifecycle.json)**: Complex lifecycle patterns
- **[monorepo-polyglot.json](devcontainer-expert/assets/templates/monorepo-polyglot.json)**: Multi-language monorepo
- **[security-hardened.json](devcontainer-expert/assets/templates/security-hardened.json)**: Security best practices
- **[dockerfile-multistage.json](devcontainer-expert/assets/templates/dockerfile-multistage.json)**: Multi-stage builds

See [devcontainer-expert/assets/templates/README.md](devcontainer-expert/assets/templates/README.md) for usage guide.

## Resources

- **Specification**: https://containers.dev/
- **VS Code Documentation**: https://code.visualstudio.com/docs/devcontainers/containers
- **GitHub Organization**: https://github.com/devcontainers
- **Templates**: https://containers.dev/templates
- **Features**: https://containers.dev/features
- **CLI**: https://github.com/devcontainers/cli

## Contributing

This is an Agent Skill following the [Agent Skills Specification](https://agentskills.io/). 

**Level 2 Architecture**: This skill uses a Level 2 structure with core patterns in SKILL.md (~533 lines) and advanced topics in references/ (~300-850 lines each).

To contribute:
1. Fork the repository: https://github.com/DecisioNaut/devcontainer-expert
2. For core patterns: Edit [devcontainer-expert/SKILL.md](devcontainer-expert/SKILL.md) (keep focused ~500 lines)
3. For advanced topics: Add or edit files in [devcontainer-expert/references/](devcontainer-expert/references/)
4. For templates: Add or update files in [devcontainer-expert/assets/templates/](devcontainer-expert/assets/templates/)
5. Ensure clear, actionable instructions with examples
6. Add appropriate cross-references between files
7. Test that all links work
8. Submit a pull request

**Note**: The skill package is located in the `devcontainer-expert/` subfolder. Repository documentation (README.md, CHANGELOG.md) stays at the root for GitHub.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Specification Compliance

This skill follows the [Agent Skills Specification v1.0](https://agentskills.io/specification):
- ✅ Valid devcontainer-expert name (lowercase with hyphens)
- ✅ Structured YAML frontmatter with required fields only
- ✅ Progressive disclosure (skill discovery via name/description)
- ✅ Clear activation criteria
- ✅ Level 2 architecture (main file 533 lines + references/)
- ✅ Comprehensive coverage via specialized reference files
- ✅ Actionable, step-by-step guidance
- ✅ Cross-referenced for easy navigation
- ✅ Clean separation: skill package in subfolder, repo docs at root

## Feedback

Issues and feedback can be submitted via [GitHub Issues](https://github.com/DecisioNaut/devcontainer-expert/issues). For questions about the Agent Skills specification itself, visit https://agentskills.io/

---

**Made for AI agents** | **Useful for humans too** | **Following the Agent Skills Specification**
