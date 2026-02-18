# Networking Fundamentals

**Reference for:** Docker network modes, port forwarding, Docker Compose basics, DNS configuration, and proxy setup in dev containers.

> **For advanced topics** like VPN integration, network security, service mesh, and load balancing, see [NETWORKING_ADVANCED.md](./NETWORKING_ADVANCED.md).

## Overview

Dev containers use Docker networking to enable communication between:
- Container and host machine
- Multiple containers (microservices, databases, etc.)
- Container and external networks

Understanding networking is essential for:
- Multi-container architectures (Docker Compose)
- Accessing external services (databases, APIs)
- Debugging connectivity issues
- Working behind corporate proxies/firewalls

## Docker Network Modes

Docker supports several network modes, configurable via `runArgs` in devcontainer.json:

### Bridge Mode (Default)

Containers get their own network namespace with NAT to the host.

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "runArgs": ["--network=bridge"]  // Default, not required
}
```

**Characteristics:**
- ✅ Container isolation
- ✅ Port forwarding/publishing required for host access
- ✅ Containers on same bridge can communicate
- ✅ Most common mode

### Host Mode

Container shares the host's network namespace directly.

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "runArgs": ["--network=host"]
}
```

**Characteristics:**
- ✅ Best performance (no NAT overhead)
- ✅ Container ports directly accessible on host
- ❌ Reduced isolation
- ❌ Port conflicts with host services
- ⚠️ Linux only (not supported on Docker Desktop for Mac/Windows)

**Use cases:**
- Performance-critical applications
- Services requiring direct network access
- Testing network-level tools

### None Mode

No network connectivity.

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "runArgs": ["--network=none"]
}
```

**Use cases:**
- Maximum isolation
- Security-sensitive workloads
- Testing offline scenarios

### Custom Networks

Create custom Docker networks for controlled communication:

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "runArgs": ["--network=my-custom-network"]
}
```

Create the network beforehand:
```bash
docker network create my-custom-network
```

## Port Forwarding vs Publishing

Understanding the difference is critical for proper network configuration.

### Port Forwarding (Recommended for Dev Containers)

VS Code forwards container ports to localhost **without modifying the container**.

**In devcontainer.json:**
```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "forwardPorts": [3000, 8080, 5432]
}
```

**How it works:**
1. Application listens on `0.0.0.0:3000` or `localhost:3000` in container
2. VS Code creates SSH tunnel: `localhost:3000` (host) → container port 3000
3. Access via `http://localhost:3000` on your machine

**Benefits:**
- ✅ Applications can bind to `localhost` (more secure)
- ✅ No container rebuild needed to change ports
- ✅ Port mapping happens at VS Code level
- ✅ Automatic port detection

### Port Publishing (Traditional Docker)

Docker maps container ports to host ports at container creation.

**In devcontainer.json:**
```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "appPort": [3000, "8080:8080", "5432:5432"]
}
```

**OR with Docker Compose:**
```yaml
services:
  app:
    ports:
      - "3000:3000"      # host:container
      - "8080:8080"
      - "5432:5432"
```

**How it works:**
- Docker exposes container port to host network
- Must rebuild container to change mappings

**When to use:**
- Containers need to be accessible outside VS Code
- Other tools/services need direct access
- Working without VS Code (CLI, Docker Compose only)

### Port Attributes

Fine-tune port behavior:

```json
{
  "forwardPorts": [3000, 5432],
  "portsAttributes": {
    "3000": {
      "label": "Application",
      "protocol": "https",
      "onAutoForward": "notify"
    },
    "5432": {
      "label": "PostgreSQL",
      "onAutoForward": "silent"
    }
  },
  "otherPortsAttributes": {
    "onAutoForward": "ignore"
  }
}
```

**Attributes:**
- `label` - Display name in Ports view
- `protocol` - `http` or `https`
- `onAutoForward` - `notify`, `openBrowser`, `openPreview`, `silent`, `ignore`
- `requireLocalPort` - Force specific host port
- `elevateIfNeeded` - Request elevated permissions for ports < 1024

## Docker Compose Networking

### Default Network

Docker Compose creates a default network automatically:

```yaml
# docker-compose.yml
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
  
  backend:
    build: ./backend
    ports:
      - "8080:8080"
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: example
```

**Communication:**
- Frontend can reach backend at `http://backend:8080`
- Backend can reach database at `postgres://db:5432`
- Service names become DNS hostnames

### Custom Networks

Create isolated networks for better control:

```yaml
services:
  frontend:
    build: ./frontend
    networks:
      - public
      - backend
  
  backend:
    build: ./backend
    networks:
      - backend
      - database
  
  db:
    image: postgres:15
    networks:
      - database

networks:
  public:
    driver: bridge
  backend:
    driver: bridge
  database:
    driver: bridge
    internal: true  # No internet access
```

**Benefits:**
- ✅ Frontend can access backend, but not database directly
- ✅ Database is isolated from external networks
- ✅ Microservices segmentation

### Network Aliases

Provide multiple DNS names for a service:

```yaml
services:
  api:
    build: ./api
    networks:
      backend:
        aliases:
          - api-server
          - api.local
          - backend-api

networks:
  backend:
```

Access API via `http://api:8080`, `http://api-server:8080`, or `http://api.local:8080`.

### External Networks

Connect to existing Docker networks:

```yaml
services:
  app:
    build: .
    networks:
      - existing-network

networks:
  existing-network:
    external: true
```

Create the external network:
```bash
docker network create existing-network
```

## DNS Configuration

### Custom DNS Servers

Override default DNS resolution:

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "runArgs": [
    "--dns=8.8.8.8",
    "--dns=1.1.1.1"
  ]
}
```

**Docker Compose:**
```yaml
services:
  app:
    build: .
    dns:
      - 8.8.8.8
      - 1.1.1.1
```

### DNS Search Domains

Add search domains for short hostnames:

```json
{
  "runArgs": [
    "--dns-search=example.com",
    "--dns-search=internal.local"
  ]
}
```

Now `ping db` resolves to `db.example.com` if it exists.

### Custom Hosts

Add entries to `/etc/hosts`:

```json
{
  "runArgs": [
    "--add-host=api.local:192.168.1.100",
    "--add-host=db.local:192.168.1.101"
  ]
}
```

**Docker Compose:**
```yaml
services:
  app:
    extra_hosts:
      - "api.local:192.168.1.100"
      - "db.local:192.168.1.101"
```

### Accessing Host Services from Container

Refer to the host machine from within the container:

**Linux:**
```bash
# Access service on host:3000 from container
curl http://host.docker.internal:3000
```

**macOS/Windows:**
```bash
# Docker Desktop provides host.docker.internal automatically
curl http://host.docker.internal:3000
```

**Linux (if host.docker.internal not available):**
```json
{
  "runArgs": [
    "--add-host=host.docker.internal:host-gateway"
  ]
}
```

**OR** get host IP:
```bash
# From container
ip route show default | awk '{print $3}'  # Host IP
```

## Proxy Configuration

### HTTP/HTTPS Proxy

Configure proxy for container internet access:

**In devcontainer.json:**
```json
{
  "containerEnv": {
    "HTTP_PROXY": "http://proxy.company.com:8080",
    "HTTPS_PROXY": "http://proxy.company.com:8080",
    "NO_PROXY": "localhost,127.0.0.1,.local"
  }
}
```

**For Docker builds (Dockerfile):**
```dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# Use build args for proxy
ARG HTTP_PROXY
ARG HTTPS_PROXY
ARG NO_PROXY

# Configure apt to use proxy
RUN if [ -n "$HTTP_PROXY" ]; then \
      echo "Acquire::http::Proxy \"$HTTP_PROXY\";" > /etc/apt/apt.conf.d/proxy.conf; \
    fi

RUN apt-get update && apt-get install -y curl
```

**Pass build args:**
```json
{
  "build": {
    "dockerfile": "Dockerfile",
    "args": {
      "HTTP_PROXY": "${localEnv:HTTP_PROXY}",
      "HTTPS_PROXY": "${localEnv:HTTPS_PROXY}",
      "NO_PROXY": "${localEnv:NO_PROXY}"
    }
  }
}
```

### Docker Daemon Proxy

Configure Docker daemon itself to use a proxy:

**~/.docker/config.json:**
```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.company.com:8080",
      "httpsProxy": "http://proxy.company.com:8080",
      "noProxy": "localhost,127.0.0.1"
    }
  }
}
```

**Docker Desktop:**
1. Settings → Resources → Proxies
2. Enable manual proxy configuration
3. Enter proxy details

### Corporate Certificate Authorities

Trust custom CA certificates:

```dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# Copy corporate CA certificate
COPY corporate-ca.crt /usr/local/share/ca-certificates/
RUN update-ca-certificates
```

**Mount dynamically:**
```json
{
  "mounts": [
    "source=${localEnv:HOME}/certs/corporate-ca.crt,target=/usr/local/share/ca-certificates/corporate-ca.crt,readonly,type=bind"
  ],
  "postCreateCommand": "sudo update-ca-certificates"
}
```

## Basic Troubleshooting

### Quick Diagnostic Commands

**Test container connectivity:**

```bash
# Check if service is reachable
ping -c 3 database-container
curl http://api-service:8080/health

# DNS resolution
nslookup database-container
dig database-container

# Check listening ports
netstat -tulpn | grep 3000
ss -tulpn
```

**Check Docker network configuration:**

```bash
# List networks
docker network ls

# Inspect network details
docker network inspect bridge

# View container network settings
docker inspect <container-id> | grep -A 20 NetworkSettings
```

### Common Issues

**"Cannot connect to service"**

1. Check service is running: `docker ps`
2. Verify port is listening inside container: `docker exec <container> netstat -tulpn`
3. Test connectivity: `docker exec <container> curl http://service-name:port`
4. Check network membership: `docker inspect <container> | grep Networks`

**"Connection refused"**

- Service binding to `127.0.0.1` instead of `0.0.0.0`
- Fix: Configure service to listen on `0.0.0.0` or all interfaces
  
**Example fixes:**
```bash
# Node.js/Express
app.listen(3000, '0.0.0.0')

# Python Flask
app.run(host='0.0.0.0', port=5000)

# Python HTTP server
python3 -m http.server 8000 --bind 0.0.0.0
```

**"Name resolution failed"**

- DNS not configured properly in container
- Check `/etc/resolv.conf` inside container
- Override DNS servers: `"runArgs": ["--dns=8.8.8.8"]`

**"Port already in use"**

- Another process using the port on host
- Find process: 
  - macOS: `lsof -i :3000`
  - Linux: `netstat -tulpn | grep 3000`
- Either kill the process or change the port mapping

## Related Documentation

- [Docker Networking Overview](https://docs.docker.com/network/)
- [Docker Compose Networking](https://docs.docker.com/compose/networking/)
- [Port Forwarding (VS Code)](https://code.visualstudio.com/docs/devcontainers/containers#_forwarding-or-publishing-a-port)
- [devcontainer.json Reference](https://containers.dev/implementors/json_reference/)
- [Security Hardening](./SECURITY.md)
Next Steps

For more advanced networking scenarios, see:

- **[Advanced Networking Patterns](./NETWORKING_ADVANCED.md)** - VPN integration, network security, service mesh, load balancing, performance tuning
- **[Security Hardening](./SECURITY.md)** - Network security policies and container isolation
- **[Container-in-Container](./CONTAINER_IN_CONTAINER.md)** - Docker-in-Docker networking considerations

## Related Documentation

- [Docker Networking Overview](https://docs.docker.com/network/)
- [Docker Compose Networking](https://docs.docker.com/compose/networking/)
- [Port Forwarding (VS Code)](https://code.visualstudio.com/docs/devcontainers/containers#_forwarding-or-publishing-a-port)
- [devcontainer.json Reference](https://containers.dev/implementors/json_reference/