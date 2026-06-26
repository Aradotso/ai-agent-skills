---
name: hiclaw-multi-agent-orchestration
description: Deploy and orchestrate collaborative multi-agent systems using HiClaw's Manager-Workers architecture with Matrix rooms, Kubernetes CRDs, and human-in-the-loop capabilities.
triggers:
  - set up hiclaw multi-agent system
  - create hiclaw worker agents
  - deploy agents with hiclaw on kubernetes
  - configure hiclaw manager and workers
  - use hiclaw for agent collaboration
  - manage hiclaw teams and matrix rooms
  - integrate mcp servers with hiclaw
  - orchestrate multiple ai agents with hiclaw
---

# HiClaw Multi-Agent Orchestration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

HiClaw is an open-source collaborative multi-agent runtime platform built on a **Manager-Workers architecture**. It orchestrates multiple AI agents in Matrix-based rooms with full human visibility and intervention. HiClaw supports multiple runtimes (OpenClaw, QwenPaw, Hermes), uses MinIO for shared file storage, and integrates with Higress AI Gateway for secure credential management.

## Installation

### Docker (Quick Start)

**macOS / Linux:**
```bash
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

**Windows (PowerShell 7+):**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
$wc=New-Object Net.WebClient
$wc.Encoding=[Text.Encoding]::UTF8
iex $wc.DownloadString('https://higress.ai/hiclaw/install.ps1')
```

The installer prompts for:
- LLM provider (OpenAI, Qwen, or OpenAI-compatible)
- API key
- Network mode (local/external)

**Requirements:** 2 CPU cores, 4 GB RAM minimum (4 cores, 8 GB recommended for multiple workers)

### Kubernetes (Helm)

```bash
helm repo add higress.io https://higress.io/helm-charts
helm repo update

# OpenAI/compatible
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080

# Custom provider
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.llmBaseUrl=https://api.deepseek.com/v1 \
  --set credentials.defaultModel=deepseek-chat \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080

# Qwen (通义千问)
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=$QWEN_API_KEY \
  --set credentials.llmProvider=qwen \
  --set credentials.defaultModel=qwen3.5-plus \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

**Key Helm Values:**
- `credentials.llmApiKey` (required): LLM provider API key
- `gateway.publicURL` (required): Public URL for Element Web
- `credentials.adminPassword`: Matrix admin password (auto-generated if empty)
- `credentials.llmProvider`: Provider name (default: `openai-compat`)
- `credentials.defaultModel`: Default model (default: `gpt-5.4`)
- `credentials.llmBaseUrl`: Base URL for OpenAI-compatible APIs
- `manager.runtime`: Manager runtime (`openclaw`, `copaw`, `hermes`)
- `worker.defaultRuntime`: Default worker runtime

**Kubernetes Requirements:** 1.24+, Helm 3.7+, default StorageClass

## Access

After installation, access Element Web at:
- Docker: `http://127.0.0.1:18088`
- Kubernetes (port-forward): `kubectl port-forward -n hiclaw-system svc/element-web 18088:80`

Login credentials (Docker):
- Username: `admin`
- Password: Check `~/.hiclaw/.env` or installer output

The Manager agent will greet you in a Matrix room.

## Core Concepts

### Architecture

```
┌─────────────────────────────────────────┐
│           Manager Agent                 │
│  (Orchestrates Workers, coordinates     │
│   tasks, interfaces with humans)        │
└──────────────┬──────────────────────────┘
               │
    ┌──────────┴──────────┬──────────┐
    │                     │          │
┌───▼────┐         ┌──────▼───┐  ┌──▼──────┐
│Worker 1│         │Worker 2  │  │Worker N │
│(OpenClaw)        │(QwenPaw) │  │(Hermes) │
└────────┘         └──────────┘  └─────────┘
         │                │          │
         └────────┬───────┴──────────┘
              ┌───▼────┐
              │ MinIO  │
              │(Shared │
              │ Files) │
              └────────┘
```

### Runtimes

1. **OpenClaw**: Deterministic agent with tool calls and structured workflows
2. **QwenPaw** (formerly CoPaw): Qwen-optimized runtime, 80% less memory
3. **Hermes**: Autonomous coding agent with full execution capabilities

### Components

- **Manager**: Central orchestrator, creates/manages workers, coordinates tasks
- **Workers**: Specialized agents with specific skills/tools
- **Teams**: Groups of workers + humans coordinated by a Team Leader
- **Higress Gateway**: AI traffic management, credential isolation
- **Tuwunel**: Matrix protocol server (IM infrastructure)
- **MinIO**: Shared file system for inter-agent data exchange
- **Element Web**: Matrix client UI

## CLI Commands

HiClaw provides a CLI for management tasks:

```bash
# Docker installations
hiclaw version              # Show version
hiclaw status              # Check service status
hiclaw logs [service]      # View logs
hiclaw restart [service]   # Restart services
hiclaw upgrade             # Upgrade to latest version

# Kubernetes installations
kubectl get workers -n hiclaw-system           # List workers
kubectl get teams -n hiclaw-system             # List teams
kubectl get managers -n hiclaw-system          # List managers
kubectl describe worker <name> -n hiclaw-system
kubectl logs -n hiclaw-system deployment/hiclaw-manager
```

## Declarative Resource Management (Kubernetes)

HiClaw uses Kubernetes-style CRDs for resource management.

### Worker CRD

Create a worker with specific runtime and skills:

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: python-developer
  namespace: hiclaw-system
spec:
  runtime: openclaw  # openclaw, copaw, or hermes
  displayName: "Python Developer"
  personality: |
    You are an expert Python developer specializing in FastAPI,
    data processing, and testing. You write clean, idiomatic code.
  skills:
    - name: file-operations
      type: builtin
    - name: web-search
      type: mcp
      config:
        command: "npx"
        args: ["-y", "@modelcontextprotocol/server-brave-search"]
        env:
          BRAVE_API_KEY:
            valueFrom:
              secretKeyRef:
                name: brave-credentials
                key: api-key
  env:
    - name: PYTHONPATH
      value: "/workspace/lib"
```

Apply:
```bash
kubectl apply -f worker.yaml
```

### Team CRD

Create a team with multiple workers and a human coordinator:

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: backend-team
  namespace: hiclaw-system
spec:
  displayName: "Backend Development Team"
  description: "Team for building and maintaining backend services"
  leader:
    personality: |
      You coordinate backend development tasks. Break complex work
      into subtasks and delegate to appropriate team members.
    mcpServers:
      - name: github-integration
        config:
          command: "npx"
          args: ["-y", "@modelcontextprotocol/server-github"]
          env:
            GITHUB_PERSONAL_ACCESS_TOKEN:
              valueFrom:
                secretKeyRef:
                  name: github-credentials
                  key: token
  members:
    workers:
      - python-developer
      - database-expert
      - devops-specialist
    humans:
      - matrixUserId: "@alice:hiclaw.local"
        role: "Tech Lead"
```

Apply:
```bash
kubectl apply -f team.yaml
```

### Manager CRD

Customize the Manager (usually one per cluster):

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Manager
metadata:
  name: primary-manager
  namespace: hiclaw-system
spec:
  runtime: copaw  # Use QwenPaw for efficiency
  personality: |
    You are a helpful AI assistant that coordinates multiple
    specialized agents. You understand user needs and delegate
    tasks effectively.
  mcpServers:
    - name: nacos-skills
      type: nacos
      config:
        serverAddr: "nacos.hiclaw-system:8848"
        namespace: "ai-registry"
        username:
          valueFrom:
            secretKeyRef:
              name: nacos-credentials
              key: username
        password:
          valueFrom:
            secretKeyRef:
              name: nacos-credentials
              key: password
```

## MCP Server Integration

HiClaw supports Model Context Protocol (MCP) servers for extensibility.

### Built-in MCP Servers

Configure at Worker/Team level:

```yaml
spec:
  mcpServers:
    - name: filesystem
      config:
        command: "npx"
        args: ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
    
    - name: brave-search
      config:
        command: "npx"
        args: ["-y", "@modelcontextprotocol/server-brave-search"]
        env:
          BRAVE_API_KEY:
            valueFrom:
              secretKeyRef:
                name: brave-credentials
                key: api-key
    
    - name: github
      config:
        command: "npx"
        args: ["-y", "@modelcontextprotocol/server-github"]
        env:
          GITHUB_PERSONAL_ACCESS_TOKEN:
            valueFrom:
              secretKeyRef:
                name: github-credentials
                key: token
```

### Nacos Skills Registry

Connect to a Nacos-based skills registry (80,000+ community skills from skills.sh):

```yaml
spec:
  mcpServers:
    - name: nacos-skills
      type: nacos
      config:
        serverAddr: "nacos.hiclaw-system:8848"
        namespace: "ai-registry"  # or "sts-hiclaw" for scoped access
        username:
          valueFrom:
            secretKeyRef:
              name: nacos-credentials
              key: username
        password:
          valueFrom:
            secretKeyRef:
              name: nacos-credentials
              key: password
```

Workers can discover and use skills on-demand without credential exposure.

## Configuration

### Environment Variables (Docker)

Located in `~/.hiclaw/.env`:

```bash
# LLM Configuration
LLM_PROVIDER=openai-compat
LLM_API_KEY=your-api-key-here
LLM_BASE_URL=https://api.openai.com/v1
DEFAULT_MODEL=gpt-4o

# Manager Runtime
MANAGER_RUNTIME=openclaw  # openclaw, copaw, hermes

# Worker Defaults
WORKER_DEFAULT_RUNTIME=openclaw

# Matrix Configuration
MATRIX_ADMIN_PASSWORD=your-secure-password
MATRIX_SERVER_NAME=hiclaw.local

# Network
HICLAW_PUBLIC_URL=http://localhost:18080
```

Restart after changes:
```bash
hiclaw restart
```

### Custom Worker Package

Create a custom worker with skills and personality:

```
my-worker/
├── SOUL.md          # Optional personality/instructions
├── skills/
│   ├── skill1.py
│   └── skill2.py
└── worker.yaml      # Worker CRD manifest
```

**worker.yaml:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: custom-worker
spec:
  runtime: openclaw
  displayName: "Custom Worker"
  packagePath: /path/to/my-worker  # Local path or Git URL
```

**SOUL.md:**
```markdown
# Custom Worker Personality

You are a specialized agent for data analysis. You:
- Process CSV/JSON datasets efficiently
- Generate visualizations with matplotlib
- Provide statistical insights
- Export results in multiple formats
```

## Common Patterns

### Creating a Worker via Matrix Chat

In Docker installations, message the Manager:

```
Create a worker named "code-reviewer" that specializes in
Go code reviews, focuses on error handling and testing,
and has access to GitHub.
```

The Manager will create the worker and invite you to a new room.

### Multi-Runtime Collaboration

Use QwenPaw as task coordinator, Hermes for code execution:

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: dev-team
spec:
  leader:
    runtime: copaw  # QwenPaw for orchestration
  members:
    workers:
      - name: executor
        runtime: hermes  # Autonomous execution
      - name: reviewer
        runtime: openclaw  # Deterministic validation
```

### Shared File Exchange

Workers use MinIO for large data:

```python
# Worker 1: Generate data
import json
from minio import Minio

client = Minio(
    "minio:9000",
    access_key=os.getenv("MINIO_ACCESS_KEY"),
    secret_key=os.getenv("MINIO_SECRET_KEY"),
    secure=False
)

data = {"results": [...]}
client.put_object(
    "shared",
    "analysis.json",
    json.dumps(data).encode(),
    len(json.dumps(data))
)

# Send to room: "Data saved to minio://shared/analysis.json"
```

```python
# Worker 2: Process data
response = client.get_object("shared", "analysis.json")
data = json.loads(response.read())
# Process data...
```

### Declarative Team Updates

Modify team membership without recreation:

```bash
# Add worker
kubectl patch team backend-team -n hiclaw-system --type=merge -p '
spec:
  members:
    workers:
      - python-developer
      - database-expert
      - devops-specialist
      - security-auditor
'

# Add human coordinator
kubectl patch team backend-team -n hiclaw-system --type=merge -p '
spec:
  members:
    humans:
      - matrixUserId: "@alice:hiclaw.local"
        role: "Tech Lead"
      - matrixUserId: "@bob:hiclaw.local"
        role: "Product Owner"
'
```

### Credential Management

Store secrets in Kubernetes, reference in CRDs:

```bash
# Create secret
kubectl create secret generic github-credentials \
  -n hiclaw-system \
  --from-literal=token=$GITHUB_TOKEN

# Reference in Worker
```

```yaml
spec:
  mcpServers:
    - name: github
      config:
        env:
          GITHUB_PERSONAL_ACCESS_TOKEN:
            valueFrom:
              secretKeyRef:
                name: github-credentials
                key: token
```

Workers see only a consumer token; real credentials stay in Higress Gateway.

## Troubleshooting

### Worker Not Appearing in Matrix

**Symptom:** Worker created but no Matrix room invitation

**Check:**
```bash
# Docker
hiclaw logs manager
hiclaw logs tuwunel

# Kubernetes
kubectl logs -n hiclaw-system deployment/hiclaw-manager
kubectl get worker <name> -n hiclaw-system -o yaml
```

**Common causes:**
- Manager not connected to Matrix (check logs)
- Worker CRD status shows errors (`kubectl describe worker`)
- Matrix server issues (`kubectl logs -n hiclaw-system statefulset/tuwunel`)

### MCP Server Fails to Start

**Symptom:** Worker logs show MCP connection errors

**Check:**
```bash
kubectl logs -n hiclaw-system deployment/<worker-name>
```

**Common causes:**
- Missing `npx` (ensure Node.js in worker image)
- Secret not found (verify secretKeyRef name/key)
- Command typo in `spec.mcpServers[].config.command`

**Fix for missing secrets:**
```bash
kubectl create secret generic brave-credentials \
  -n hiclaw-system \
  --from-literal=api-key=$BRAVE_API_KEY
```

### High Memory Usage

**Symptom:** Workers consuming excessive memory

**Solution:** Switch to QwenPaw runtime (80% less memory):

```yaml
spec:
  runtime: copaw
```

Or adjust resource limits:

```yaml
spec:
  resources:
    limits:
      memory: 512Mi
    requests:
      memory: 256Mi
```

### Agent Not Responding

**Symptom:** Messages sent but no response

**Check:**
1. LLM API connectivity:
```bash
kubectl logs -n hiclaw-system deployment/higress-gateway
```

2. Worker status:
```bash
kubectl get worker <name> -n hiclaw-system
```

3. Manager logs:
```bash
kubectl logs -n hiclaw-system deployment/hiclaw-manager -f
```

**Common causes:**
- LLM API key invalid/expired
- Rate limiting
- Worker crashed (check status/logs)

### Upgrade Issues

**Symptom:** Upgrade fails or services don't start

**Docker:**
```bash
# Full reinstall (preserves data)
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)

# Or specific version
HICLAW_VERSION=v1.1.2 bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

**Kubernetes:**
```bash
# Check current chart version
helm list -n hiclaw-system

# Upgrade
helm repo update
helm upgrade hiclaw higress.io/hiclaw -n hiclaw-system
```

### Matrix Login Fails

**Symptom:** Cannot login to Element Web

**Reset password (Docker):**
```bash
# Check current password
cat ~/.hiclaw/.env | grep MATRIX_ADMIN_PASSWORD

# Reset (requires restart)
docker exec hiclaw-tuwunel register_new_matrix_user \
  -c /data/homeserver.yaml \
  -u admin -p newpassword --admin
```

**Kubernetes:**
```bash
kubectl exec -n hiclaw-system statefulset/tuwunel -- \
  register_new_matrix_user \
  -c /data/homeserver.yaml \
  -u admin -p newpassword --admin
```

## Advanced Usage

### Custom Runtime Images

Build worker with custom dependencies:

```dockerfile
FROM hiclaw/worker-openclaw:latest

RUN pip install pandas scikit-learn torch

COPY ./skills /app/skills
```

Deploy:
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: ml-worker
spec:
  runtime: openclaw
  image: myregistry.com/ml-worker:v1
  imagePullPolicy: Always
```

### External Access (Ingress)

Expose Element Web and API:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hiclaw-ingress
  namespace: hiclaw-system
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - hiclaw.example.com
      secretName: hiclaw-tls
  rules:
    - host: hiclaw.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: element-web
                port:
                  number: 80
```

Update Helm installation:
```bash
helm upgrade hiclaw higress.io/hiclaw -n hiclaw-system \
  --set gateway.publicURL=https://hiclaw.example.com
```

### Monitoring and Metrics

HiClaw controller exposes Prometheus metrics:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hiclaw-controller-metrics
  namespace: hiclaw-system
spec:
  selector:
    app: hiclaw-controller
  ports:
    - name: metrics
      port: 8080
      targetPort: 8080
```

Key metrics:
- `hiclaw_controller_reconcile_total`: Reconciliation count
- `hiclaw_controller_reconcile_duration_seconds`: Reconciliation latency
- `hiclaw_worker_status`: Worker status (0=unhealthy, 1=healthy)

## Resources

- **Homepage:** https://hiclaw.io
- **GitHub:** https://github.com/agentscope-ai/HiClaw
- **Discord:** https://discord.com/invite/NVjNA4BAVw
- **Skills Registry:** https://skills.sh
- **Documentation:** Check `docs/` and `blog/` in the repository
