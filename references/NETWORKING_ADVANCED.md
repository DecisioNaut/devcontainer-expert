# Advanced Networking Patterns

**Reference for:** VPN integration, network security, service mesh architectures, load balancing, and performance optimization in dev containers.

> **Prerequisites:** Read [NETWORKING.md](./NETWORKING.md) for fundamental networking concepts including Docker network modes, port forwarding, and Docker Compose basics.

## VPN Considerations

### Split Tunneling

When using a VPN, Docker networking can break if all traffic routes through VPN.

**Solutions:**

1. **Exclude Docker subnets from VPN:**
   - Add `172.17.0.0/16` (default Docker bridge) to VPN exclusions
   - Add custom network subnets if used

2. **Use host network mode (Linux only):**
   ```json
   {
     "runArgs": ["--network=host"]
   }
   ```

3. **Adjust Docker bridge subnet:**
   
   **/etc/docker/daemon.json:**
   ```json
   {
     "bip": "192.168.99.1/24"
   }
   ```
   
   Restart Docker: `sudo systemctl restart docker`

### VPN Inside Container

Run VPN client inside container instead of on host:

```dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu

RUN apt-get update && apt-get install -y openvpn
COPY client.ovpn /etc/openvpn/
```

Requires `--cap-add=NET_ADMIN`:
```json
{
  "runArgs": ["--cap-add=NET_ADMIN"]
}
```

## Network Security

### Internal Networks

Prevent internet access for sensitive containers:

```yaml
services:
  database:
    image: postgres:15
    networks:
      - db-internal

networks:
  db-internal:
    internal: true  # No external internet
```

### Network Policies

Limit container network access:

```json
{
  "runArgs": [
    "--cap-drop=NET_RAW",          // Disable raw sockets
    "--cap-drop=NET_ADMIN"         // Disable network admin
  ]
}
```

### Firewall Rules

Apply iptables rules in container:

```dockerfile
RUN apt-get install -y iptables

# Allow only specific hosts
RUN iptables -A OUTPUT -d 10.0.0.0/8 -j ACCEPT
RUN iptables -A OUTPUT -d 0.0.0.0/0 -j REJECT
```

Requires `--cap-add=NET_ADMIN`.

## Advanced Troubleshooting

### Diagnostic Commands

**Inside Container:**

```bash
# Check network interfaces
ip addr show
ifconfig

# Test connectivity
ping -c 3 google.com
curl -I https://api.example.com

# DNS resolution
nslookup example.com
dig example.com

# Check listening ports
netstat -tulpn
ss -tulpn

# Trace route
traceroute google.com
mtr google.com

# Check active connections
netstat -an | grep ESTABLISHED
```

**On Host:**

```bash
# Inspect Docker networks
docker network ls
docker network inspect bridge

# Container network details
docker inspect <container-id> | grep -A 20 NetworkSettings

# Test port forwarding
curl http://localhost:3000

# Check if port is listening (macOS)
lsof -i :3000

# Check if port is listening (Linux)
netstat -tulpn | grep 3000
```

### Network Performance Analysis

```bash
# Install network tools
apt-get update && apt-get install -y \
  iperf3 tcpdump netcat-openbsd dnsutils

# Test bandwidth between containers
# In server container:
iperf3 -s

# In client container:
iperf3 -c server-container

# Monitor traffic
tcpdump -i eth0 port 3000

# Test specific port connectivity
nc -zv api-server 8080

# Trace DNS queries
dig +trace example.com
```

### Debugging Container-to-Container Communication

```bash
# From app container, test database connection
docker exec -it app-container bash
curl -v telnet://db:5432

# Check if service is reachable by name
ping db

# Verify network membership
docker inspect app-container | grep -A 10 Networks

# Test with netcat
echo "test" | nc db 5432
```

## Service Mesh Architecture

Build complex microservice networks:

```yaml
# docker-compose.yml
services:
  frontend:
    build: ./frontend
    environment:
      API_URL: http://api-gateway:8080
    networks:
      - public

  api-gateway:
    build: ./gateway
    environment:
      AUTH_SERVICE: http://auth:3001
      DATA_SERVICE: http://data:3002
    networks:
      - public
      - backend

  auth:
    build: ./auth-service
    networks:
      - backend
      - database

  data:
    build: ./data-service
    networks:
      - backend
      - database

  db:
    image: postgres:15
    networks:
      - database

networks:
  public:
  backend:
  database:
    internal: true
```

**Network Segmentation Benefits:**
- Frontend can only access API Gateway
- Services communicate through backend network
- Database is fully isolated from public network
- Security through network isolation

## Load Balancing

### Nginx Reverse Proxy

Distribute traffic across multiple app instances:

```yaml
services:
  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "80:80"
    networks:
      - frontend

  app1:
    build: .
    networks:
      - frontend

  app2:
    build: .
    networks:
      - frontend

  app3:
    build: .
    networks:
      - frontend

networks:
  frontend:
```

**nginx.conf:**
```nginx
upstream backend {
    # Load balancing methods
    least_conn;  # Route to least connections
    # ip_hash;   # Sticky sessions based on IP
    # round_robin (default)
    
    server app1:3000;
    server app2:3000;
    server app3:3000;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    location /health {
        access_log off;
        return 200 "healthy\n";
    }
}
```

### HAProxy Load Balancer

Alternative to nginx with advanced health checks:

```yaml
services:
  haproxy:
    image: haproxy:latest
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    ports:
      - "80:80"
      - "8404:8404"  # Stats page
    networks:
      - frontend

  app:
    build: .
    deploy:
      replicas: 3
    networks:
      - frontend
```

**haproxy.cfg:**
```cfg
defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http-in
    bind *:80
    default_backend servers

backend servers
    balance roundrobin
    option httpchk GET /health
    server app1 app1:3000 check
    server app2 app2:3000 check
    server app3 app3:3000 check

listen stats
    bind *:8404
    stats enable
    stats uri /
```

## Environment-Specific Networks

### Development vs Production

Separate configurations for different environments:

```yaml
# docker-compose.yml (base)
services:
  app:
    build: .
    networks:
      - default

# docker-compose.override.yml (development - auto-loaded)
services:
  app:
    ports:
      - "3000:3000"  # Expose for local debugging
    environment:
      DEBUG: "true"
      LOG_LEVEL: debug

# docker-compose.prod.yml (production - explicit)
services:
  app:
    networks:
      - production_network
    environment:
      DEBUG: "false"
      LOG_LEVEL: warn
    # No port exposure - accessed via reverse proxy

networks:
  production_network:
    external: true
```

**Usage:**
```bash
# Development (auto-loads override)
docker-compose up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up

# Testing
docker-compose -f docker-compose.yml -f docker-compose.test.yml up
```

### Multi-Stage Network Configuration

```yaml
# docker-compose.dev.yml
services:
  app:
    build:
      target: development
    networks:
      - dev-network
    volumes:
      - .:/workspace

# docker-compose.staging.yml
services:
  app:
    build:
      target: production
    networks:
      - staging-network
    # No volume mounts

# docker-compose.prod.yml
services:
  app:
    build:
      target: production
    networks:
      - prod-network
    deploy:
      replicas: 3
```

## Performance Optimization

### Adjust MTU (Maximum Transmission Unit)

For performance issues with large packets or VPN:

```bash
# Create network with custom MTU
docker network create \
  --opt com.docker.network.driver.mtu=1400 \
  custom-network
```

```json
{
  "runArgs": ["--network=custom-network"]
}
```

**When to adjust:**
- VPN connections causing packet fragmentation
- Network performance degradation
- Large file transfers failing

**Common MTU values:**
- `1500` - Standard Ethernet
- `1450` - Common with VPN
- `1400` - Conservative for problematic networks
- `9000` - Jumbo frames (advanced setups)

### Enable IPv6

```bash
# Create network with IPv6 support
docker network create \
  --ipv6 \
  --subnet=2001:db8:1::/64 \
  ipv6-network
```

```json
{
  "runArgs": ["--network=ipv6-network"]
}
```

**Docker daemon configuration** (/etc/docker/daemon.json):
```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64",
  "experimental": true,
  "ip6tables": true
}
```

### Network Performance Testing

**Bandwidth testing between containers:**

```bash
# Install iperf3
apt-get update && apt-get install -y iperf3

# Server (in one container)
iperf3 -s

# Client (in another container) - TCP test
iperf3 -c server-container -t 30

# UDP test with bandwidth limit
iperf3 -c server-container -u -b 100M

# Parallel streams
iperf3 -c server-container -P 10
```

**Docker network inspection:**

```bash
# Check network drivers and performance
docker network inspect bridge | grep -i driver

# Monitor network I/O
docker stats --format "table {{.Container}}\t{{.NetIO}}"

# Detailed network stats
docker stats --no-stream container-name
```

### DNS Caching

Improve DNS performance:

```dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# Install dnsmasq for local DNS caching
RUN apt-get update && apt-get install -y dnsmasq

# Configure dnsmasq
RUN echo "cache-size=1000" >> /etc/dnsmasq.conf
RUN echo "no-negcache" >> /etc/dnsmasq.conf

CMD ["dnsmasq", "-k"]
```

## Related Documentation

- [Core Networking Fundamentals](./NETWORKING.md)
- [Docker Networking Overview](https://docs.docker.com/network/)
- [Docker Compose Networking](https://docs.docker.com/compose/networking/)
- [Security Hardening](./SECURITY.md)
- [Container-in-Container Patterns](./CONTAINER_IN_CONTAINER.md)
- [devcontainer.json Reference](https://containers.dev/implementors/json_reference/)
