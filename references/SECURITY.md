# Security Hardening

**Reference for:** Securing dev containers through non-root users, secrets management, capability restrictions, and supply-chain best practices.

## Overview

While dev containers provide isolation, proper security practices are essential, especially when:

- Working with sensitive data or credentials
- Connecting to production systems from dev environments
- Sharing devcontainer.json configurations across teams
- Using third-party Features or base images

This reference covers security best practices for dev containers.

## Workspace Trust

VS Code's [Workspace Trust](https://code.visualstudio.com/docs/editing/workspaces/workspace-trust) feature protects you from malicious code in devcontainer.json files.

### How It Works

When opening a folder with a devcontainer.json, VS Code asks you to trust the workspace:

- **Trust** - Executes lifecycle scripts and container configuration
- **Restricted Mode** - Opens file system but doesn't execute code

### Trust Scenarios

**Reopening folder locally:**
- You'll be prompted before the container starts
- Recent entries bypass the prompt

**Attaching to existing container:**
- Prompted once per container
- Confirms you trust the container's contents

**Cloning in container volume:**
- Prompted once per repository
- Confirms you trust the repository

### Best Practices

1. **Review devcontainer.json before trusting**
   - Check `postCreateCommand`, `postStartCommand`, `postAttachCommand`
   - Inspect Dockerfile and scripts
   - Verify Feature sources

2. **Don't trust unfamiliar repositories**
   - Fork and review code first
   - Be cautious with PR previews from unknown contributors

3. **Use Restricted Mode for inspection**
   - Inspect devcontainer configuration without executing it
   - Review lifecycle scripts before trusting

## Running as Non-Root User

By default, containers often run as `root`. Running as a non-root user reduces security risks.

### Configure Non-Root User

**Method 1: remoteUser Property**

```json
{
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20",
  "remoteUser": "node"
}
```

**Method 2: Use common-utils Feature**

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/common-utils:2": {
      "username": "vscode",
      "userUid": "1000",
      "userGid": "1000"
    }
  },
  "remoteUser": "vscode"
}
```

**Method 3: Dockerfile USER Directive**

```dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# Create non-root user
RUN useradd -m -s /bin/bash devuser

# Switch to non-root user
USER devuser
```

```json
{
  "build": {
    "dockerfile": "Dockerfile"
  },
  "remoteUser": "devuser"
}
```

### File Ownership Issues

When running as non-root, volumes may be owned by root:

```json
{
  "remoteUser": "vscode",
  "postCreateCommand": "sudo chown -R vscode:vscode /workspace"
}
```

**For named volumes:**
```json
{
  "mounts": [
    "source=myproject-node_modules,target=/workspace/node_modules,type=volume"
  ],
  "postCreateCommand": "sudo chown vscode /workspace/node_modules"
}
```

### Benefits

✅ Limits blast radius of compromised processes  
✅ Prevents accidental system modifications  
✅ Aligns with principle of least privilege  
✅ Required for rootless Docker environments  

## Secrets Management

**Never commit secrets in devcontainer.json or Dockerfiles.**

### Strategy 1: Environment Variables (User Settings)

Store secrets in user settings (not committed to Git):

**settings.json (User scope):**
```json
{
  "terminal.integrated.env.linux": {
    "DATABASE_PASSWORD": "my-secret-password",
    "API_KEY": "abc123"
  }
}
```

Reference in devcontainer.json:
```json
{
  "remoteEnv": {
    "DATABASE_URL": "postgres://user:${localEnv:DATABASE_PASSWORD}@localhost/db"
  }
}
```

### Strategy 2: .env File (Gitignored)

Create `.env` file (add to `.gitignore`):

```env
DATABASE_PASSWORD=my-secret-password
API_KEY=abc123
```

**For Docker Compose:**

```yaml
services:
  app:
    env_file: .env
```

```json
{
  "dockerComposeFile": "docker-compose.yml",
  "service": "app"
}
```

**For Dockerfile:**

Pass as build args (use sparingly):

```json
{
  "build": {
    "dockerfile": "Dockerfile",
    "args": {
      "API_KEY": "${localEnv:API_KEY}"
    }
  }
}
```

### Strategy 3: Secret Files (Mounted)

Mount secret files from host:

```json
{
  "mounts": [
    "source=${localEnv:HOME}/.aws,target=/home/vscode/.aws,readonly,type=bind",
    "source=${localEnv:HOME}/.ssh,target=/home/vscode/.ssh,readonly,type=bind"
  ],
  "remoteUser": "vscode"
}
```

✅ Read-only mount prevents accidental modification  
✅ Keeps secrets on host, not in container image  

### Strategy 4: Secret Management Services

For production-like environments:

**AWS Secrets Manager:**
```bash
# In postCreateCommand or manually
aws secretsmanager get-secret-value --secret-id myapp/database --query SecretString --output text > /tmp/db-secret
```

**HashiCorp Vault:**
```bash
vault kv get -field=password secret/myapp/database
```

**Azure Key Vault:**
```bash
az keyvault secret show --vault-name myvault --name database-password --query value -o tsv
```

### Best Practices

1. **Use .gitignore liberally**
   ```gitignore
   .env
   .env.local
   secrets/
   *.pem
   *.key
   ```

2. **Document required secrets**
   Create `.env.example`:
   ```env
   DATABASE_PASSWORD=<your-password-here>
   API_KEY=<your-api-key>
   ```

3. **Avoid secrets in lifecycle scripts**
   ❌ Bad:
   ```json
   {
     "postCreateCommand": "export API_KEY=abc123 && npm install"
   }
   ```
   
   ✅ Good:
   ```json
   {
     "remoteEnv": {
       "API_KEY": "${localEnv:API_KEY}"
     },
     "postCreateCommand": "npm install"
   }
   ```

4. **Rotate secrets regularly**
   - Change credentials periodically
   - Update .env files after rotation

## Container Capabilities

Linux capabilities allow fine-grained control over container permissions.

### Adding Capabilities

Some tools require specific capabilities:

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "capAdd": [
    "SYS_PTRACE",     // For debuggers (gdb, lldb)
    "NET_ADMIN",      // For network configuration tools
    "SYS_ADMIN"       // For mounting filesystems (use sparingly)
  ]
}
```

### Common Capabilities

| Capability | Use Case | Risk Level |
|------------|----------|------------|
| `SYS_PTRACE` | Debugging with gdb/lldb | Low |
| `NET_ADMIN` | Network tools (iptables, tcpdump) | Medium |
| `NET_RAW` | Raw socket operations (ping) | Medium |
| `SYS_ADMIN` | Mount filesystems, chroot | **High** |

### Removing Capabilities

For hardened environments, drop unnecessary capabilities:

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "capDrop": [
    "NET_RAW",
    "SYS_ADMIN"
  ]
}
```

### Best Practices

1. **Grant minimum required capabilities**
2. **Document why capabilities are needed**
3. **Avoid SYS_ADMIN if possible** - It's nearly equivalent to root
4. **Test without capabilities first** - Many tools don't actually need them

## Pre-built Image Security

### Supply-Chain Security Benefits

Pre-building images provides security advantages (see [PREBUILDING.md](./PREBUILDING.md)):

1. **Version pinning** - Lock exact tool versions
2. **Vulnerability scanning** - Scan images before use
3. **Audit trail** - Know exactly what's in each image
4. **Reduced build-time attacks** - Fewer external fetches during build

### Image Scanning

Scan pre-built images for vulnerabilities:

**Trivy:**
```bash
# Scan image
trivy image ghcr.io/myorg/myproject-devcontainer:latest

# Fail on high/critical vulnerabilities
trivy image --severity HIGH,CRITICAL --exit-code 1 myimage:latest
```

**Snyk:**
```bash
snyk container test myimage:latest
```

**Docker Scout (Docker Desktop):**
```bash
docker scout cves myimage:latest
```

### Pin Base Image Versions

❌ **Avoid:**
```json
{
  "image": "node:latest"  // Unpredictable updates
}
```

✅ **Prefer:**
```json
{
  "image": "mcr.microsoft.com/devcontainers/typescript-node:1-20-bookworm"
}
```

Or use digest pinning:
```json
{
  "image": "mcr.microsoft.com/devcontainers/typescript-node@sha256:abc123..."
}
```

### Feature Version Pinning

```json
{
  "features": {
    "ghcr.io/devcontainers/features/node:1.0.0": {},     // Exact version
    "ghcr.io/devcontainers/features/python:1.0": {},     // Minor version
    "ghcr.io/devcontainers/features/go:1": {}            // Major version (riskier)
  }
}
```

## Network Security

### Limit Network Access

Use Docker networks to restrict communication:

```yaml
services:
  app:
    networks:
      - backend
  db:
    networks:
      - backend
networks:
  backend:
    internal: true  # No external internet access
```

### Firewall Rules

For containers needing limited external access:

```json
{
  "runArgs": [
    "--dns=8.8.8.8",
    "--dns-search=example.com"
  ]
}
```

### Avoid Privileged Mode

❌ **Never use unless absolutely required:**
```json
{
  "runArgs": ["--privileged"]  // Disables ALL security features
}
```

✅ **Use specific capabilities instead:**
```json
{
  "capAdd": ["SYS_PTRACE"]  // Only what's needed
}
```

## Git Credentials

VS Code automatically forwards Git credentials to containers. For enhanced security:

### SSH Key Sharing

```json
{
  "mounts": [
    "source=${localEnv:HOME}/.ssh,target=/home/vscode/.ssh,readonly,type=bind"
  ]
}
```

### Git Credential Manager

Windows/macOS credential helpers are forwarded automatically. For Linux:

```bash
# Install git-credential-manager
curl -L https://aka.ms/gcm/linux-install-source.sh | sh

# Configure
git config --global credential.helper manager
```

### Best Practices

1. **Use SSH keys over HTTPS tokens when possible**
2. **Protect SSH keys with passphrases**
3. **Use read-only mounts** for sensitive credential files
4. **Rotate tokens/keys periodically**

## Audit and Compliance

### Log Lifecycle Scripts

Ensure lifecycle commands are logged:

```json
{
  "postCreateCommand": "set -x && npm install",  // Bash verbose mode
  "postStartCommand": "sh -x ./setup.sh"
}
```

### Review devcontainer.json Changes

Require PR reviews for devcontainer.json changes:

**.github/CODEOWNERS:**
```
.devcontainer/* @org/security-team
```

### Compliance Scanning

Scan Dockerfiles and configurations:

**hadolint (Dockerfile linter):**
```bash
hadolint .devcontainer/Dockerfile
```

**dockle (Image security):**
```bash
dockle myimage:latest
```

## Incident Response

### Compromised Container

If a container is compromised:

1. **Stop the container immediately:**
   ```bash
   docker stop <container-id>
   ```

2. **Inspect container logs:**
   ```bash
   docker logs <container-id>
   ```

3. **Remove the container:**
   ```bash
   docker rm <container-id>
   ```

4. **Scan images:**
   ```bash
   trivy image myimage:latest
   ```

5. **Rebuild from clean source:**
   ```bash
   git clean -fdx
   devcontainer build --no-cache
   ```

### Post-Incident

- Rotate all credentials that were accessible in the container
- Review audit logs for unauthorized access
- Update security policies based on root cause

## Checklist

Use this checklist for security reviews:

- [ ] Run as non-root user (`remoteUser` configured)
- [ ] No secrets in devcontainer.json or Dockerfile
- [ ] Secrets in .gitignore
- [ ] Base images pinned to specific versions
- [ ] Features pinned to specific versions
- [ ] Vulnerability scanning configured in CI
- [ ] Workspace Trust enabled
- [ ] Only necessary capabilities granted
- [ ] Read-only mounts for sensitive files
- [ ] Lifecycle scripts reviewed and safe
- [ ] .devcontainer/ changes require review (CODEOWNERS)

## Related Documentation

- [Workspace Trust](https://code.visualstudio.com/docs/editing/workspaces/workspace-trust)
- [Add Non-Root User](https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user)
- [Sharing Git Credentials](https://code.visualstudio.com/remote/advancedcontainers/sharing-git-credentials)
- [Pre-building Images](./PREBUILDING.md)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
