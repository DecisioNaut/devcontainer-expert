# Changelog

All notable changes to the devcontainer-expert skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2026-02-14

### Changed
- **BREAKING**: Evolved to Level 2 architecture for comprehensive enterprise coverage
  - Core patterns remain in SKILL.md (~475 lines)
  - Advanced topics moved to specialized references/ directory
  - Progressive disclosure: discovery in main file, deep dives in references

### Added
- **9 comprehensive reference files** covering advanced scenarios:
  - `references/PREBUILDING.md` (427 lines) - Pre-building images, caching strategies, CI/CD workflows, supply-chain security
  - `references/KUBERNETES.md` (365 lines) - Attaching to K8s pods, kubectl integration, cluster development
  - `references/NETWORKING.md` (573 lines) - Docker network modes, port forwarding, DNS, proxies, Docker Compose basics
  - `references/NETWORKING_ADVANCED.md` (557 lines) - VPN, network security, service mesh, load balancing, performance tuning
  - `references/VOLUME_PERFORMANCE.md` (397 lines) - WSL 2 optimization, named volumes, platform-specific performance
  - `references/CONTAINER_IN_CONTAINER.md` (380 lines) - Docker-in-Docker Feature, DinD vs DfD, CI/CD testing
  - `references/SECURITY.md` (574 lines) - Workspace Trust, non-root users, secrets management, capabilities, network security
  - `references/DOTFILES.md` (452 lines) - Built-in dotfiles support, shell customization, tool automation
  - `references/RECOVERY_DEBUGGING.md` (547 lines) - Recovery containers, build failure debugging, diagnostic commands

### Improved
- SKILL.md focused on core patterns with cross-references to advanced topics
- **Networking split into fundamentals and advanced patterns** - Better agent digestibility (573 + 557 lines vs 819 lines)
- Comprehensive coverage of enterprise devcontainer scenarios
- Better separation of concerns: discovery vs detailed implementation
- Enhanced navigation with internal cross-linking
- README updated to document Level 2 architecture
- Contributing guidelines updated for multi-file structure

### Coverage
- All v1.1.0 content retained in accessible format
- Expanded coverage: pre-building, Kubernetes, networking fundamentals & advanced patterns, performance, security, Docker-in-Docker, dotfiles, recovery
- Total documentation: ~4,750 lines across all files (optimized for agent token budgets)
- Optimized for agent discovery and human reference

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

[2.0.0]: https://github.com/yourusername/devcontainer-expert/compare/v1.1.0...v2.0.0
[1.1.0]: https://github.com/yourusername/devcontainer-expert/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/yourusername/devcontainer-expert/releases/tag/v1.0.0
