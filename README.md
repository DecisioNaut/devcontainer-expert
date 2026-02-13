# Development Containers Expert

An Agent Skill providing expert guidance for working with Development Containers (devcontainers) in VS Code.

## Description

This skill provides comprehensive knowledge and step-by-step instructions for creating, configuring, and troubleshooting development containers. It covers the full devcontainer ecosystem including Templates, Features, Docker integration, and VS Code workflows.

## What You'll Learn

- **Core Concepts**: Understanding development containers and when to use them
- **Configuration**: Creating and customizing devcontainer.json files
- **Templates**: Using pre-built templates for common tech stacks
- **Features**: Adding tools and runtimes with dev container Features
- **Integration**: Working with Docker, Docker Compose, and VS Code
- **Advanced Topics**: Lifecycle scripts, custom mounts, environment variables
- **Troubleshooting**: Solving common devcontainer issues
- **CI/CD**: Using devcontainers in GitHub Actions and Azure DevOps

## Installation

### For Agents

If you're an AI agent, simply read the [SKILL.md](SKILL.md) file to access all the devcontainer expertise.

### For Human Developers

This skill is designed for AI agents, but humans can also use it as a comprehensive reference guide. Read [SKILL.md](SKILL.md) for detailed instructions on working with devcontainers.

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

- ✅ devcontainer.json configuration
- ✅ Templates and Features
- ✅ Docker and Docker Compose integration
- ✅ VS Code workflows and commands
- ✅ Lifecycle scripts (onCreate, postCreate, postStart, etc.)
- ✅ Port forwarding and networking
- ✅ Environment variables and mounts
- ✅ Git credentials sharing
- ✅ CI/CD integration
- ✅ Common troubleshooting scenarios
- ✅ Best practices and security

## Resources

- **Specification**: https://containers.dev/
- **VS Code Documentation**: https://code.visualstudio.com/docs/devcontainers/containers
- **GitHub Organization**: https://github.com/devcontainers
- **Templates**: https://containers.dev/templates
- **Features**: https://containers.dev/features
- **CLI**: https://github.com/devcontainers/cli

## Contributing

This is an Agent Skill following the [Agent Skills Specification](https://agentskills.io/). 

To contribute:
1. Fork this repository
2. Make your changes to SKILL.md
3. Ensure the skill stays under 500 lines
4. Test that instructions are clear and actionable
5. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) for details.

## Specification Compliance

This skill follows the [Agent Skills Specification v1.0](https://agentskills.io/specification):
- ✅ Valid devcontainer-expert name (lowercase with hyphens)
- ✅ Structured YAML frontmatter with required fields
- ✅ Progressive disclosure (skill discovery via name/description)
- ✅ Clear activation criteria
- ✅ Comprehensive but focused instructions
- ✅ Under 500 lines for optimal agent processing
- ✅ Actionable, step-by-step guidance

## Feedback

Issues and feedback can be submitted via GitHub Issues. For questions about the Agent Skills specification itself, visit https://agentskills.io/

---

**Made for AI agents** | **Useful for humans too** | **Following the Agent Skills Specification**
