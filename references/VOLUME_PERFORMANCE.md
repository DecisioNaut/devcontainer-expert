# Volume Performance Optimization

**Reference for:** Improving file system performance in dev containers on macOS and Windows through volume strategies.

## Overview

On macOS and Windows, Docker runs containers in a Linux VM. **Bind mounts** (mounting local filesystem directories into containers) have significant performance overhead due to cross-VM file system translation. This manifests as:

- Slow `npm install`, `pip install`, or similar package operations
- Sluggish file watching (auto-reload delays)
- Poor compile times for large codebases

This reference covers strategies to dramatically improve performance using **named volumes**, **WSL 2 filesystem**, and **targeted volume mounts**.

## Performance Comparison

| Strategy | Windows (non-WSL) | Windows (WSL 2) | macOS | Linux |
|----------|-------------------|-----------------|-------|-------|
| **Bind Mount** | ❌ Slow | ⚠️ Moderate | ❌ Slow | ✅ Fast |
| **Named Volume** | ✅ Fast | ✅ Fast | ✅ Fast | ✅ Fast |
| **WSL 2 Filesystem** | ✅ Fast | N/A | N/A | Native |

**Recommendation:**
- **Windows:** Use WSL 2 + store source code in WSL filesystem
- **macOS:** Use Clone Repository in Container Volume or named volumes
- **Linux:** Bind mounts are already fast; no optimization needed

## Strategy 1: WSL 2 Filesystem (Windows Only)

### Best Performance on Windows

Windows 10 2004+ and Windows 11 include WSL 2 with significantly improved performance. Docker Desktop 2.3+ supports the WSL 2 backend.

### Setup

1. **Enable WSL 2:**
   ```powershell
   wsl --install
   wsl --set-default-version 2
   ```

2. **Enable Docker Desktop WSL 2 Backend:**
   - Right-click Docker Desktop system tray icon
   - Select **Settings** → **General**
   - Check **"Use the WSL 2 based engine"**
   - **Resources** → **WSL Integration** → Enable for your distributions

3. **Store Code in WSL Filesystem:**
   ```bash
   # In WSL terminal
   cd ~
   mkdir projects
   cd projects
   git clone https://github.com/yourorg/yourrepo.git
   ```

4. **Open Folder in Container:**
   - In VS Code, install **[Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)** extension
   - Run: `Dev Containers: Reopen in Container` from WSL folder

### Performance Impact

| Operation | Before (Windows FS) | After (WSL 2 FS) |
|-----------|---------------------|------------------|
| `npm install` | 120s | 15s |
| `pip install` | 90s | 12s |
| File watching | Multiple seconds delay | Near-instant |
| Large file operations | 10-60x slower | Native speed |

### Video Tutorial

See [Speed up Dev Containers on Windows](https://code.visualstudio.com/remote/advancedcontainers/improve-performance#_video-speed-up-dev-containers-on-windows) for a visual walkthrough.

## Strategy 2: Clone Repository in Container Volume

### Universal Performance Boost (macOS + Windows)

The `Dev Containers: Clone Repository in Container Volume...` command uses an **isolated Docker named volume** instead of binding to the local filesystem.

### Benefits

✅ **Improved performance** - Disk operations run at native Linux speed  
✅ **Cleaner file tree** - No local project files polluting your file system  
✅ **Isolated environments** - Each clone is independent  

### Usage

1. **Run Command:**
   - Open Command Palette: `F1`
   - Select: `Dev Containers: Clone Repository in Container Volume...`

2. **Enter Repository:**
   - GitHub URL: `https://github.com/microsoft/vscode-remote-try-node`
   - GitHub PR: `https://github.com/microsoft/vscode/pull/12345`
   - Repo identifier: `microsoft/vscode-remote-try-node`

3. **Select Template** (if no devcontainer.json exists)

4. **VS Code builds and opens** the repository in a container backed by a named volume

### How It Works

```bash
# Docker creates a named volume
docker volume create vscode-remote-try-node-unique-id

# Code is cloned into the volume (not your local filesystem)
# Container mounts the volume, achieving native Linux performance
```

### Accessing Volume Contents

To inspect or modify the volume outside VS Code:

```bash
# List volumes
docker volume ls | grep vscode

# Explore volume in dev container
# Command Palette → Dev Containers: Explore a Volume in a Dev Container...
```

## Strategy 3: Targeted Named Volumes (node_modules)

### Problem: Package Folders Slow

Even with bind mounts, package installation folders like `node_modules`, `vendor/`, or `.venv/` benefit from being in named volumes.

### Solution: Mount Specific Folders to Named Volumes

**For Dockerfile/Image Projects:**

Add to `.devcontainer/devcontainer.json`:

```json
{
  "name": "Node.js Project",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20",
  "mounts": [
    "source=${localWorkspaceFolderBasename}-node_modules,target=${containerWorkspaceFolder}/node_modules,type=volume"
  ],
  "postCreateCommand": "sudo chown node node_modules && npm install"
}
```

**For Docker Compose Projects:**

Update `docker-compose.yml`:

```yaml
services:
  app:
    build: .
    volumes:
      # Bind mount source code
      - .:/workspace:cached
      # Named volume for node_modules
      - myproject-node_modules:/workspace/node_modules

volumes:
  myproject-node_modules:
```

Update `.devcontainer/devcontainer.json`:

```json
{
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "postCreateCommand": "sudo chown node node_modules && npm install"
}
```

### How It Works

1. Source code is bind-mounted (e.g., `/workspaces/myproject`)
2. `node_modules/` subfolder is **overlaid** with a named volume
3. Package operations run at native speed
4. An empty `node_modules/` folder appears locally (harmless)

### Important Notes

**Don't Delete the Folder Itself:**
```bash
# ❌ This breaks the volume mount
rm -rf node_modules

# ✅ Delete contents instead
rm -rf node_modules/* node_modules/.*
```

**Non-Root Users:**
The volume is created as root. If your container runs as a non-root user (e.g., `node`), use `postCreateCommand` to fix ownership:

```json
{
  "remoteUser": "node",
  "postCreateCommand": "sudo chown node node_modules"
}
```

### Performance Impact

| Operation | Bind Mount | Named Volume |
|-----------|------------|--------------|
| `npm install` (1000 packages) | 180s | 18s |
| `pip install -r requirements.txt` | 90s | 10s |
| `composer install` | 120s | 15s |

### Video Tutorial

See [Speed up npm install in a dev container](https://code.visualstudio.com/remote/advancedcontainers/improve-performance#_video-speed-up-npm-install-in-a-dev-container).

## Strategy 4: Named Volume for Entire Source Tree

### Ultimate Performance (No Local Files)

Mount the **entire workspace** into a named volume instead of binding to local filesystem.

### Configuration

**For Dockerfile/Image:**

```json
{
  "name": "My Project",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20",
  "workspaceMount": "source=myproject-workspace,target=/workspace,type=volume",
  "workspaceFolder": "/workspace"
}
```

**For Docker Compose:**

```yaml
services:
  app:
    build: .
    volumes:
      - myproject-workspace:/workspace

volumes:
  myproject-workspace:
```

```json
{
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace"
}
```

### Cloning Source Code

Since the volume is empty initially, clone your repo:

1. **Rebuild and reopen** in container
2. **Run in terminal:**
   ```bash
   git clone https://github.com/yourorg/yourrepo.git /workspace
   ```
3. **Reopen folder:**
   - File → Open Folder → `/workspace`

**Tip:** Use `postCreateCommand` to automate:

```json
{
  "postCreateCommand": "git clone https://github.com/yourorg/yourrepo.git . || true"
}
```

### Trade-Offs

| Pros | Cons |
|------|------|
| ✅ Maximum performance | ❌ Source code not accessible on host |
| ✅ No cross-platform issues | ❌ Must use container tools for Git/file operations |
| ✅ Consistent across machines | ❌ Slightly more complex workflow |

**Best for:** Projects where all work happens in the container.

## Platform-Specific Optimizations

### macOS: :cached and :delegated Flags

Bind mounts on macOS support consistency flags to trade accuracy for speed:

```json
{
  "mounts": [
    "source=${localWorkspaceFolder},target=/workspace,type=bind,consistency=cached"
  ]
}
```

**Flags:**
- `cached` - Host writes visible to container with delay (faster reads)
- `delegated` - Container writes visible to host with delay (faster writes)
- `consistent` (default) - Strict consistency (slowest)

**Docker Compose:**
```yaml
volumes:
  - .:/workspace:cached  # Read-heavy workloads
  - ./data:/workspace/data:delegated  # Write-heavy workloads
```

**Note:** These flags have minimal impact on modern Docker Desktop versions but can still help in older setups.

### Windows: Avoid Antivirus Scanning

Windows Defender and third-party antivirus can slow Docker volumes:

1. **Exclude Docker Directories:**
   - Add exclusions for:
     - `%LOCALAPPDATA%\Docker`
     - `C:\ProgramData\Docker`
     - WSL 2 distributions (if applicable)

2. **Windows Security Settings:**
   - Open **Windows Security** → **Virus & threat protection**
   - **Manage settings** → **Exclusions** → **Add or remove exclusions**

### Linux: Already Optimal

Linux runs containers natively, so bind mounts have no overhead. Named volumes provide no performance benefit and add complexity. **Use bind mounts on Linux.**

## Monitoring Performance

### Docker Stats

```bash
# Monitor container resource usage
docker stats

# Check disk I/O
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.BlockIO}}"
```

### Benchmarking

```bash
# Test write performance
time dd if=/dev/zero of=/workspace/testfile bs=1M count=1000

# Test read performance  
time cat /workspace/largefile > /dev/null

# Test package install time
time npm install
```

## Best Practices

1. **Windows Users:**
   - Always use WSL 2 + WSL filesystem for best performance
   - Don't access WSL files from Windows Explorer (use `\\wsl$\` if needed)

2. **macOS Users:**
   - Prefer "Clone Repository in Container Volume" for new projects
   - Use targeted named volumes for package folders in existing projects

3. **Multi-Platform Teams:**
   - Document volume strategy in README
   - Consider using named volumes for consistency across platforms

4. **CI/CD:**
   - Named volumes provide consistent performance in CI
   - Avoid bind mounts in GitHub Actions/Azure DevOps

## Troubleshooting

**"node_modules empty after mount"**
- Ensure `postCreateCommand` reinstalls packages
- Check volume was created: `docker volume ls`

**"Permission denied" errors**
- Fix ownership: `sudo chown -R $USER:$USER /workspace/node_modules`
- Or use `remoteUser` + `postCreateCommand`

**"Changes not reflected"**
- Named volumes don't sync to host - this is expected
- Use `docker cp` to extract files if needed: `docker cp <container>:/workspace/file.txt ./`

**"Volume disappeared"**
- Volumes persist until explicitly deleted: `docker volume rm <volume-name>`
- Check with: `docker volume ls`

## Related Documentation

- [Improve Disk Performance (Official)](https://code.visualstudio.com/remote/advancedcontainers/improve-performance)
- [WSL 2 Backend](https://docs.docker.com/desktop/wsl/)
- [Docker Volumes](https://docs.docker.com/storage/volumes/)
- [Clone Repository in Container Volume](../SKILL.md#common-workflows)
