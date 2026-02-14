# Changelog

All notable changes to the devcontainer-expert skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-02-14

### Added
- **Comprehensive monorepo support** - Critical for users working with monorepos
  - Pattern 1: Separate package containers for different tech stacks
  - Pattern 2: Unified polyglot container with all tools
  - Pattern 3: Docker Compose multi-container setup
- **Multi-root workspace limitation documentation** - Important constraint that all folders in `.code-workspace` files open in same container
- Workflow guidance for opening multiple packages
- Practical examples for frontend/backend monorepo scenarios

### Changed
- Optimized Dev Container CLI section for brevity
- Condensed CI/CD integration examples
- Streamlined Popular Features list

### Improved
- Line count: 517 lines (from 484) - balanced comprehensive coverage with brevity
- Added advanced patterns requested by community

## [1.0.0] - 2026-02-14

### Added
- Initial release of devcontainer-expert skill
- Comprehensive devcontainer.json configuration guide
- Instructions for using Templates and Features
- Docker and Docker Compose integration examples
- VS Code workflow guidance
- Lifecycle scripts documentation
- Advanced configuration examples (mounts, environment variables, non-root users)
- Troubleshooting section with common issues and solutions
- CI/CD integration examples (GitHub Actions, Azure DevOps)
- Dev Container CLI usage guide
- Best practices and security recommendations
- Quick reference section for VS Code commands

### Coverage
- Development Containers Specification (https://containers.dev)
- VS Code Dev Containers extension
- devcontainers GitHub organization resources
- Official templates and features
- Docker Hub images

[1.1.0]: https://github.com/yourusername/devcontainer-expert/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/yourusername/devcontainer-expert/releases/tag/v1.0.0
