# Recovery and Debugging

**Reference for:** Debugging container build failures, inspecting failed containers, and recovering from broken dev container setups.

## Overview

Dev container builds don't always succeed on the first try. When things go wrong, VS Code provides powerful recovery tools to debug and fix issues without starting from scratch.

**Common Scenarios:**
- Dockerfile syntax errors
- Failed package installations
- Network timeouts during build
- Misconfigured devcontainer.json
- Feature installation failures

This reference covers strategies for diagnosing and fixing these issues.

## Recovery Containers

When a dev container build **fails**, VS Code offers to open a **Recovery Container**.

### What is a Recovery Container?

A recovery container is a minimal container with:
- ✅ The **filesystem state** at the point of failure
- ✅ **Build logs** available for inspection
- ✅ **Shell access** to debug interactively
- ✅ Ability to **test fixes** before rebuilding

### Activating Recovery Mode

**Automatic Prompt:**
When a container build fails, VS Code shows:

```
Container build failed.
[Reopen in Recovery Container]  [Show Log]  [Cancel]
```

Click **Reopen in Recovery Container**.

**Manual Invocation:**
If you dismissed the prompt, trigger recovery mode via Command Palette:

1. `F1` → `Dev Containers: Reopen in Recovery Container`
2. Select the failed container

### What You Can Do in Recovery Mode

```bash
# 1. View build logs
cat /tmp/vscode-container-*.log

# 2. Inspect filesystem state
ls -la /workspaces
ls -la /.devcontainer

# 3. Test commands that failed
apt-get update
apt-get install -y build-essential  # Test if this was the issue

# 4. Fix configuration files
vi /.devcontainer/devcontainer.json
vi /.devcontainer/Dockerfile

# 5. Test Dockerfile commands manually
RUN apt-get update && apt-get install -y curl
# Becomes:
apt-get update && apt-get install -y curl
```

### Recovery Workflow

1. **Open in Recovery Container** → Minimal environment loads
2. **Inspect Logs** → Identify the failure point
3. **Test Fix** → Run commands manually to verify solution
4. **Update Config** → Edit devcontainer.json or Dockerfile on host
5. **Rebuild** → `Dev Containers: Rebuild Container`
6. **Verify** → Check build succeeds

## Common Build Failures

### 1. Dockerfile Syntax Errors

**Error:**
```
Step 5/10 : RUN apt-get update \
ERROR: Dockerfile parse error at line 5: unexpected token
```

**Diagnosis:**
- Missing continuation character (`\`)
- Mismatched quotes
- Invalid instruction

**Recovery:**
```bash
# In recovery container, inspect Dockerfile
cat /.devcontainer/Dockerfile

# Fix on host, then rebuild
```

**Fix Example:**

❌ **Before:**
```dockerfile
RUN apt-get update
    apt-get install -y curl  # Missing \
```

✅ **After:**
```dockerfile
RUN apt-get update \
    && apt-get install -y curl
```

### 2. Package Installation Failures

**Error:**
```
Step 7/10 : RUN apt-get install -y nonexistent-package
E: Unable to locate package nonexistent-package
```

**Recovery:**
```bash
# In recovery container, test package availability
apt-cache search <package-name>

# Test installation
apt-get update
apt-get install -y <correct-package-name>
```

**Fix:**
Update Dockerfile with correct package name.

### 3. Network Timeouts

**Error:**
```
Could not connect to archive.ubuntu.com
Temporary failure resolving 'archive.ubuntu.com'
```

**Recovery:**
```bash
# Test network connectivity
ping -c 3 8.8.8.8
curl -I https://google.com

# Check DNS
cat /etc/resolv.conf
```

**Fix:**
- Check Docker Desktop network settings
- Try again (transient issue)
- Use different mirror in Dockerfile:
  ```dockerfile
  RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
  ```

### 4. Feature Installation Failures

**Error:**
```
Feature 'ghcr.io/devcontainers/features/nonexistent:1' not found
```

**Recovery:**
```bash
# Check Feature URL is correct
# Verify Feature exists: https://containers.dev/features
```

**Fix:**
Correct Feature ID in devcontainer.json:

```json
{
  "features": {
    "ghcr.io/devcontainers/features/node:1": {}  // Correct path
  }
}
```

### 5. Permission Errors

**Error:**
```
mkdir: cannot create directory '/workspace': Permission denied
```

**Recovery:**
```bash
# Check user
whoami

# Check directory ownership
ls -la /workspace

# Test with sudo
sudo mkdir /workspace/test
```

**Fix:**
Set correct `remoteUser` or fix ownership in `postCreateCommand`:

```json
{
  "remoteUser": "vscode",
  "postCreateCommand": "sudo chown -R vscode:vscode /workspace"
}
```

## Inspecting Running Containers

### Show Container Log

View real-time container logs:

```
Command Palette → Dev Containers: Show Container Log
```

**What it shows:**
- Container startup process
- Feature installation output
- Lifecycle script execution (postCreateCommand, etc.)
- Error messages

**Use for:**
- Diagnosing slow startup
- Debugging postCreateCommand failures
- Monitoring Feature installations

### Attach to Running Container

Inspect a container without modifying it:

```
Command Palette → Dev Containers: Attach to Running Container...
```

**Use for:**
- Debugging production containers
- Inspecting containers created by other tools
- Read-only investigation

## Debugging Lifecycle Scripts

Lifecycle scripts (`postCreateCommand`, `postStartCommand`, etc.) can fail silently.

### Add Debugging

```json
{
  "postCreateCommand": "set -x && npm install",  // Bash verbose mode
  "postStartCommand": "sh -x ./setup.sh"         // Shell verbose mode
}
```

**OR** use arrays for better error handling:

```json
{
  "postCreateCommand": ["bash", "-c", "set -ex && npm install && npm run setup"]
}
```

### Test Scripts Manually

```bash
# In container terminal
bash -x .devcontainer/scripts/setup.sh

# Or step-by-step
npm install
npm run setup
```

### Common Issues

**Script not executable:**
```json
{
  "postCreateCommand": "chmod +x ./setup.sh && ./setup.sh"
}
```

**Wrong working directory:**
```json
{
  "postCreateCommand": "cd /workspace && npm install"
}
```

**Missing environment variables:**
```json
{
  "remoteEnv": {
    "NODE_ENV": "development"
  },
  "postCreateCommand": "npm install"
}
```

## Cleaning Up

### Remove Broken Containers

```bash
# Stop and remove container
docker ps -a  # Find container ID
docker stop <container-id>
docker rm <container-id>

# Remove volumes (if using named volumes)
docker volume ls
docker volume rm <volume-name>
```

### Clean Docker System

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Nuclear option: Remove everything
docker system prune -a --volumes
```

**Warning:** This deletes ALL stopped containers, unused images, and volumes.

### Rebuild Without Cache

Force complete rebuild:

```
Command Palette → Dev Containers: Rebuild Container Without Cache
```

OR with CLI:

```bash
devcontainer build --no-cache --workspace-folder .
```

## Advanced Debugging

### Enable Docker BuildKit

BuildKit provides better error messages:

```bash
export DOCKER_BUILDKIT=1
devcontainer build --workspace-folder .
```

### Dockerfile Layer Inspection

```bash
# Build and inspect each layer
docker build --target <stage-name> .

# View image history
docker history <image-name>

# Inspect specific layer
docker inspect <image-id>
```

### Container Shell Access (No VS Code)

```bash
# Run container manually
docker run -it --rm <image-name> bash

# Execute command in running container
docker exec -it <container-id> bash
```

## Preventing Build Failures

### 1. Test Dockerfile Separately

```bash
# Build Dockerfile directly
docker build -f .devcontainer/Dockerfile .

# Run image
docker run -it --rm <image-id> bash
```

### 2. Use Well-Tested Base Images

✅ **Prefer official Dev Container images:**
```json
{
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20"
}
```

❌ **Avoid untested custom images:**
```json
{
  "image": "random-dockerhub-user/untested-image:latest"
}
```

### 3. Pin Versions

```json
{
  "image": "mcr.microsoft.com/devcontainers/typescript-node:1-20-bookworm",
  "features": {
    "ghcr.io/devcontainers/features/node:1.0.0": {},
    "ghcr.io/devcontainers/features/git:1.0.1": {}
  }
}
```

### 4. Test Features Individually

Add Features one at a time to isolate failures:

```json
{
  "features": {
    "ghcr.io/devcontainers/features/node:1": {}
    // Add more after testing this works
  }
}
```

### 5. Use Templates as Starting Points

```
Command Palette → Dev Containers: Add Dev Container Configuration Files...
```

Select a Template matching your stack. Templates are tested and maintained.

## Diagnostic Commands

### Inside Container

```bash
# Check environment
env | sort

# Verify tools installed
which node python docker git

# Check running processes
ps aux

# Disk usage
df -h

# Network connectivity
ping -c 3 google.com
curl -I https://github.com

# DNS resolution
nslookup github.com
```

### On Host

```bash
# Docker info
docker info
docker version

# Check running containers
docker ps -a

# Resource usage
docker stats

# View logs
docker logs <container-id>

# Inspect container
docker inspect <container-id>
```

## Getting Help

### Check Official Docs

- [Create a Dev Container](https://code.visualstudio.com/docs/devcontainers/create-dev-container)
- [Tips and Tricks](https://code.visualstudio.com/docs/devcontainers/tips-and-tricks)
- [FAQ](https://code.visualstudio.com/docs/devcontainers/faq)

### Community Support

- [Stack Overflow (vscode-remote tag)](https://stackoverflow.com/questions/tagged/vscode-remote)
- [VS Code GitHub Issues](https://github.com/microsoft/vscode-remote-release/issues)
- [Dev Containers Spec Discussion](https://github.com/devcontainers/spec/discussions)

### Report Bugs

If you've found a bug in the Dev Containers extension:

1. **Reproduce in recovery container** or minimal example
2. **Collect logs:** `Dev Containers: Show Container Log`
3. **Report:** [vscode-remote-release/issues](https://github.com/microsoft/vscode-remote-release/issues)

Include:
- VS Code version
- Docker version
- devcontainer.json
- Dockerfile (if applicable)
- Error messages / logs

## Troubleshooting Checklist

When debugging container issues, check:

- [ ] Dockerfile syntax is valid
- [ ] Features use correct IDs
- [ ] Base image exists and is pullable
- [ ] Network connectivity (curl/ping works)
- [ ] Lifecycle scripts are executable
- [ ] Sufficient disk space (`df -h`)
- [ ] Docker daemon is running (`docker ps`)
- [ ] Correct devcontainer.json location
- [ ] No typos in property names
- [ ] Logs reviewed (`Show Container Log`)
- [ ] Recovery container tested

## Related Documentation

- [Container Build Troubleshooting](https://code.visualstudio.com/docs/devcontainers/tips-and-tricks#_troubleshooting-containers)
- [Attach to Running Container](https://code.visualstudio.com/docs/devcontainers/attach-container)
- [Docker Troubleshooting](https://docs.docker.com/config/daemon/troubleshoot/)
- [Reduce Dockerfile Warnings](https://code.visualstudio.com/remote/advancedcontainers/reduce-docker-warnings)
