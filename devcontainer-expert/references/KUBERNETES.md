# Kubernetes Integration

**Reference for:** Attaching VS Code to running containers in Kubernetes clusters for development and debugging.

## Overview

The Dev Containers extension supports attaching to containers running in Kubernetes clusters. This enables debugging, inspecting, and developing against live cluster environments without recreating infrastructure locally.

**Use cases:**
- Debug microservices running in Kubernetes
- Inspect production/staging pods without SSH
- Develop against cluster-specific resources (databases, services, secrets)
- Test applications in realistic orchestrated environments

## Prerequisites

### Required Tools

1. **kubectl** - Kubernetes command-line tool
   ```bash
   # macOS
   brew install kubectl
   
   # Linux
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl && sudo mv kubectl /usr/local/bin/
   
   # Windows (PowerShell)
   choco install kubernetes-cli
   ```

2. **Kubernetes Extension** (optional but recommended)
   - Install from VS Code Extensions marketplace: `ms-kubernetes-tools.vscode-kubernetes-tools`
   - Provides visual cluster navigation and right-click attach

3. **Dev Containers Extension**
   - Already installed if you're using this skill

### Cluster Access

Ensure kubectl is configured with access to your cluster:

```bash
# Verify connectivity
kubectl cluster-info

# List contexts
kubectl config get-contexts

# Switch context if needed
kubectl config use-context <context-name>
```

## Attaching to Kubernetes Containers

### Method 1: Command Palette

1. **Open Command Palette:** `F1` or `Cmd/Ctrl+Shift+P`
2. **Run:** `Dev Containers: Attach to Running Kubernetes Container...`
3. **Select cluster:** Choose from configured kubectl contexts
4. **Navigate pod structure:**
   - Select namespace
   - Select pod
   - Select container (if pod has multiple containers)
5. **VS Code reloads** and connects to the container

### Method 2: Kubernetes Extension (Visual)

If you have the Kubernetes extension installed:

1. **Open Kubernetes view** from Activity Bar
2. **Expand your cluster** in the Kubernetes explorer
3. **Navigate:** Cluster → Namespace → Pods → Pod Name
4. **Right-click on container** → Select **"Attach Visual Studio Code"**

![Kubernetes Explorer Attach](https://code.visualstudio.com/assets/docs/devcontainers/attach-container/k8s-attach.png)

VS Code will reload and open the container environment.

## What You Can Do

Once attached to a K8s container:

- ✅ Open terminals (runs inside the container)
- ✅ Install VS Code extensions in the container
- ✅ Forward ports (expose container ports to localhost)
- ✅ Edit files with full IntelliSense
- ✅ Debug applications running in the pod
- ✅ Run commands via integrated terminal

## Limitations

### No devcontainer.json Support

⚠️ **Important:** Attached container configuration files (like devcontainer.json) are **not supported** for Kubernetes containers.

You cannot:
- Specify extensions to auto-install
- Define forwarded ports declaratively
- Configure lifecycle scripts
- Set environment variables via devcontainer.json

**Workaround:** Use image-level configuration or manually configure after attaching.

### Ephemeral Connections

- Configuration is not persisted between attaches
- Extensions must be reinstalled each time you attach
- Port forwards are temporary

### Pod Restarts

If the pod restarts or is rescheduled, your connection will be lost. You'll need to reattach to the new pod instance.

## Configuration (Attached Container Config)

While devcontainer.json isn't supported, VS Code maintains an **attached container configuration** based on image/container name. This config persists some settings between attaches.

### Viewing Attached Config

After attaching, run from Command Palette:
- `Dev Containers: Open Container Configuration File` (image-level)
- `Dev Containers: Open Named Configuration File` (container name-level)

### Supported Properties

```json
{
  "workspaceFolder": "/path/to/code/in/container",
  "settings": {
    "terminal.integrated.defaultProfile.linux": "bash"
  },
  "extensions": ["dbaeumer.vscode-eslint"],
  "forwardPorts": [8000, 5432],
  "remoteUser": "vscode",
  "remoteEnv": {
    "MY_VARIABLE": "some-value"
  },
  "postAttachCommand": "npm install"
}
```

**Note:** These are applied to **future attaches** to containers with matching image/name, not retroactively.

## Port Forwarding

### Automatic Port Detection

VS Code automatically detects ports that applications listen on inside the container and suggests forwarding them.

### Manual Port Forwarding

1. **Open Command Palette:** `F1`
2. **Run:** `Forward a Port`
3. **Enter port number** (e.g., `8080`)
4. **Access via** `http://localhost:<local-port>`

## Common Workflows

### Debug a Microservice in a Cluster

```bash
# 1. Ensure kubectl context is correct
kubectl config current-context

# 2. Find your pod
kubectl get pods -n <namespace>

# 3. Attach VS Code (via Command Palette or K8s Explorer)

# 4. In VS Code terminal (now inside pod):
ps aux  # Find process
cd /app
ls -la

# 5. Install debugger extension if needed
# 6. Set breakpoints in code
# 7. Forward application port
# 8. Trigger requests to debug
```

### Inspect Cluster-Specific Configuration

```bash
# Attach to pod

# Check environment variables
env | grep DATABASE

# Check mounted secrets
ls /var/run/secrets/
cat /var/run/secrets/kubernetes.io/serviceaccount/token

# Test service connectivity
curl http://backend-service:8080/health
```

### Temporary Development Against Live Data

⚠️ **Use extreme caution** - you're modifying a live environment!

```bash
# Attach to pod

# Make temporary changes for testing
vi /app/config.yaml

# Restart process (if possible)
kill -HUP <pid>

# Test changes
curl localhost:8080/endpoint
```

**Best Practice:** Only do this in dev/staging clusters, never production.

## Advanced Scenarios

### Multi-Container Pods

If a pod has multiple containers (e.g., app + sidecar), you'll be prompted to select which container to attach to:

```yaml
# Pod definition
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:latest
  - name: proxy
    image: nginx:latest
```

When attaching, choose `app` or `proxy` as needed.

### Namespace Isolation

If your cluster has many namespaces, specify the namespace explicitly when listing pods:

```bash
# List pods in specific namespace
kubectl get pods -n production

# Attach via command palette will prompt for namespace
```

### Remote Clusters (Not Local)

Attaching works with any cluster kubectl can reach:
- Cloud-managed clusters (EKS, GKE, AKS)
- Self-hosted remote clusters
- Local clusters (minikube, kind, k3s)

Ensure your `~/.kube/config` has the context configured.

## Kubernetes Development Patterns

### Pattern 1: Local Dev with Minikube/Kind

Develop locally using a local Kubernetes cluster:

```bash
# Start minikube
minikube start

# Deploy your app
kubectl apply -f k8s/deployment.yaml

# Attach VS Code to running pod
# Make changes, test in realistic K8s environment
```

### Pattern 2: Bridge to Staging Cluster

Use port forwarding to route traffic from staging cluster services to your local dev container:

```bash
# In terminal: Forward staging database to localhost
kubectl port-forward -n staging svc/postgres 5432:5432

# In dev container: Connect to localhost:5432
# Test against real staging data
```

### Pattern 3: Debugging Production Issues

**For urgent production debugging only:**

```bash
# Attach to production pod (READ-ONLY mode)
# Inspect logs, environment, network
# DO NOT modify files or restart processes
# Use insights to fix issue in dev environment
```

## Comparison: Dev Containers vs K8s Attach

| Feature | Dev Container (devcontainer.json) | K8s Attach |
|---------|-----------------------------------|------------|
| **Configuration** | Full devcontainer.json support | No devcontainer.json |
| **Persistence** | Reproducible across team | Ephemeral |
| **Use Case** | Primary development environment | Debugging/inspection |
| **Lifecycle Control** | VS Code manages container | K8s manages pod |
| **Customization** | Features, extensions, scripts | Manual after attach |
| **Best For** | Daily development | Production debugging |

## Troubleshooting

**"No clusters found"**
- Verify kubectl is installed: `kubectl version --client`
- Check kubeconfig: `kubectl config view`
- Ensure context is set: `kubectl config get-contexts`

**"Unable to connect to pod"**
- Verify pod is running: `kubectl get pods`
- Check pod logs: `kubectl logs <pod-name>`
- Ensure network connectivity: `kubectl exec <pod-name> -- ping google.com`

**Extensions not working after attach**
- Manually install extensions from Extensions view
- Check for `glibc` dependencies (Alpine Linux compatibility issues)
- Verify extension supports remote development

**Connection drops frequently**
- Check pod health: `kubectl describe pod <pod-name>`
- Look for OOMKilled or CrashLoopBackOff status
- Review resource limits in pod definition

## Security Considerations

### RBAC Permissions

Attaching to pods requires appropriate Kubernetes RBAC permissions:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-attach
rules:
- apiGroups: [""]
  resources: ["pods", "pods/exec"]
  verbs: ["get", "list", "create"]
```

### Audit Logging

Be aware that attaching to pods may be logged by cluster audit systems. Ensure you have authorization for the clusters you're accessing.

### Best Practices

1. **Never attach to production** unless absolutely necessary for critical debugging
2. **Use read-only operations** when inspecting production pods
3. **Create debug replicas** instead of attaching to live production pods
4. **Log your actions** when working with sensitive environments

## Related Documentation

- [kubectl Reference](https://kubernetes.io/docs/reference/kubectl/)
- [Attach to Running Container](https://code.visualstudio.com/docs/devcontainers/attach-container)
- [Kubernetes Extension](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools)
- [Attached Container Config Reference](https://code.visualstudio.com/docs/devcontainers/attach-container#_attached-container-configuration-reference)
