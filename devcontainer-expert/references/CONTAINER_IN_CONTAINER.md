# Docker-in-Docker (Container-in-Container)

**Reference for:** Running Docker or building container images from inside a dev container.

## Overview

Many development workflows require running Docker commands from within a container, such as:

- Building container images (CI/CD pipelines)
- Running Docker Compose for integration tests
- Managing containers from within a dev environment
- Testing containerized applications

There are two primary approaches:

1. **Docker-in-Docker (DinD)** - Run a separate Docker daemon inside the container
2. **Docker-from-Docker (DfD)** - Mount the host's Docker socket into the container

This reference covers the **Docker-in-Docker Feature** provided by the Dev Containers specification.

## Docker-in-Docker Feature

The `docker-in-docker` Feature is the recommended way to enable Docker functionality in dev containers.

### Basic Usage

Add to `.devcontainer/devcontainer.json`:

```json
{
  "name": "Project with Docker",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  }
}
```

Rebuild your container and Docker will be available:

```bash
docker --version
docker ps
docker build -t myapp .
```

### Feature Options

```json
{
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {
      "version": "latest",              // Docker/Moby version
      "moby": true,                      // Use OSS Moby instead of Docker CE
      "dockerDashComposeVersion": "v2",  // v1, v2, or none
      "installDockerBuildx": true,       // Install Buildx plugin
      "azureDnsAutoDetection": true,     // Auto-detect Azure DNS
      "dockerDefaultAddressPool": ""     // Default address pools for networks
    }
  }
}
```

#### Key Options Explained

| Option | Description | Default |
|--------|-------------|---------|
| `version` | Docker/Moby engine version (e.g., `latest`, `24.0`, `23.0`) | `latest` |
| `moby` | Install open-source Moby build instead of Docker CE | `true` |
| `dockerDashComposeVersion` | Docker Compose version to install | `v2` |
| `installDockerBuildx` | Enable Buildx for multi-architecture builds | `true` |
| `dockerDefaultAddressPool` | Define network address pools (e.g., `base=192.168.0.0/16,size=24`) | - |

### Full Example with Options

```json
{
  "name": "CI/CD Environment",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {
      "version": "24.0",
      "moby": true,
      "dockerDashComposeVersion": "v2",
      "installDockerBuildx": true
    },
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "postCreateCommand": "docker --version && docker compose version"
}
```

## How It Works

The Feature installs:
1. **Docker Engine (dockerd)** - Runs as a daemon inside the container
2. **Docker CLI** - Command-line tools (docker, docker compose)
3. **Startup Script** - Automatically starts dockerd when the container starts

Based on the [official Moby Docker-in-Docker wrapper script](https://github.com/moby/moby/blob/master/hack/dind).

### Process Architecture

```
┌─────────────────────────────────────┐
│ Host Docker Daemon                  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │ Dev Container                │  │
│  │                              │  │
│  │  ┌─────────────────────┐    │  │
│  │  │ Inner Docker Daemon │    │  │
│  │  └─────────────────────┘    │  │
│  │           ▲                  │  │
│  │           │                  │  │
│  │  ┌────────┴────────┐        │  │
│  │  │ Docker CLI      │        │  │
│  │  │ $ docker build  │        │  │
│  │  └─────────────────┘        │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
```

## Common Use Cases

### 1. CI/CD Testing Locally

Test GitHub Actions or CI pipelines that build Docker images:

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/github-cli:1": {}
  }
}
```

In terminal:
```bash
# Simulate CI build
docker build -t myapp:test .

# Run tests in container
docker run --rm myapp:test npm test

# Test Docker Compose setup
docker compose up -d
docker compose ps
docker compose down
```

### 2. Building Multi-Architecture Images

Use Buildx for ARM64 and AMD64 builds:

```bash
# Create builder instance
docker buildx create --use

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:multiarch \
  --push \
  .
```

### 3. Integration Testing with Docker Compose

Test applications that depend on multiple services:

**docker-compose.test.yml:**
```yaml
services:
  app:
    build: .
    environment:
      DATABASE_URL: postgres://db:5432/testdb
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: testdb
      POSTGRES_PASSWORD: testpass
```

**In dev container terminal:**
```bash
docker compose -f docker-compose.test.yml up -d
npm run test:integration
docker compose -f docker-compose.test.yml down
```

### 4. Container Management Tools

Build developer tools that manage containers:

```bash
# List containers across different Docker contexts
docker context ls
docker ps --all

# Clean up resources
docker system prune -af
docker volume prune -f
```

## Limitations

### Platform Requirements

⚠️ **Must run on Docker host** - The Feature is designed for Docker (or Moby) hosts. Other container engines (Podman, Containerd) may not work.

⚠️ **Same architecture** - Host and container must use the same chip architecture:

❌ **This will NOT work:**
```json
{
  "build": {
    "dockerfile": "Dockerfile",
    "args": {
      "PLATFORM": "linux/amd64"  // Emulated x86 on Apple Silicon
    }
  },
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  }
}
```

✅ **This will work:**
```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu"  // Native ARM64 on Apple Silicon
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  }
}
```

### OS Support

Supported operating systems:
- ✅ Debian/Ubuntu (with `apt`)
- ✅ Recent Debian-based distributions
- ❌ Alpine Linux (use Docker-from-Docker alternative)
- ❌ Windows containers

Requires `bash` to execute installation script.

### Security Considerations

**Privileged mode not required** - Unlike some DinD implementations, the Feature runs without `--privileged` flag, reducing security risks.

**Isolated Docker daemon** - Inner Docker daemon is separate from host, providing isolation.

**Resource overhead** - Running a second Docker daemon consumes additional CPU/memory.

## Alternative: Docker-from-Docker (Socket Mount)

If the Docker-in-Docker Feature doesn't meet your needs, you can mount the host Docker socket instead.

### Docker Socket Mount Approach

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/docker-outside-of-docker:1": {}
  },
  "mounts": [
    "source=/var/run/docker.sock,target=/var/run/docker.sock,type=bind"
  ]
}
```

### Pros and Cons

| Aspect | Docker-in-Docker | Docker-from-Docker |
|--------|------------------|-------------------|
| **Isolation** | ✅ Separate daemon | ❌ Shares host daemon |
| **Security** | ✅ Better isolation | ⚠️ Full Docker access on host |
| **Performance** | ⚠️ Overhead from nested daemon | ✅ No daemon overhead |
| **Cleanup** | ✅ Automatic on container stop | ❌ Must clean up manually |
| **Compatibility** | ⚠️ Requires Docker host | ✅ Works everywhere |

**Recommendation:** Use Docker-in-Docker unless you have specific reasons to use socket mounting (e.g., need to access existing host containers).

## Troubleshooting

**"Cannot connect to the Docker daemon"**
- Ensure the Feature is installed: Check `.devcontainer/devcontainer.json`
- Rebuild container: `Dev Containers: Rebuild Container`
- Check daemon status: `ps aux | grep dockerd`

**"docker: command not found"**
- Feature installation failed - check logs during container build
- Verify bash is available: `which bash`
- Ensure OS is Debian/Ubuntu: `cat /etc/os-release`

**Cross-architecture build fails**
- Verify native architecture: `uname -m`
- Remove `--platform` directives that force emulation
- Use multi-architecture base images

**Out of disk space**
```bash
# Clean up Docker resources
docker system prune -af
docker volume prune -f

# Check available space
df -h
```

**Slow Docker builds inside container**
- Inner Docker daemon uses container's filesystem
- Ensure adequate resources: Increase Docker Desktop CPU/memory limits
- Use smaller base images to reduce build context

## Performance Optimization

### Enable BuildKit

BuildKit provides better caching and parallelism:

```bash
export DOCKER_BUILDKIT=1
docker build -t myapp .
```

Or globally in devcontainer.json:

```json
{
  "containerEnv": {
    "DOCKER_BUILDKIT": "1"
  }
}
```

### Use Layer Caching

```bash
docker build \
  --cache-from myapp:latest \
  -t myapp:new \
  .
```

### Prune Regularly

Add a lifecycle script:

```json
{
  "postStartCommand": "docker system prune -f"
}
```

## Best Practices

1. **Use for CI/CD simulation** - Perfect for testing Docker workflows locally
2. **Avoid in production** - DinD adds complexity; use native Docker in production
3. **Monitor resources** - Inner daemon consumes additional CPU/memory
4. **Pin versions** - Specify Feature version for reproducibility:
   ```json
   "ghcr.io/devcontainers/features/docker-in-docker:2.0.0": {}
   ```
5. **Clean up regularly** - Remove unused images/containers to free space

## Related Documentation

- [Docker-in-Docker Feature](https://github.com/devcontainers/features/tree/main/src/docker-in-docker)
- [Moby DinD Script](https://github.com/moby/moby/blob/master/hack/dind)
- [Docker Buildx](https://docs.docker.com/buildx/working-with-buildx/)
- [Security Hardening](./SECURITY.md)
