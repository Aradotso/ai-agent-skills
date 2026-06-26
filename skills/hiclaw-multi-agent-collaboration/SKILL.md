---
name: hiclaw-multi-agent-collaboration
description: Deploy and orchestrate collaborative multi-agent teams using HiClaw's Manager-Workers architecture with Matrix rooms, Kubernetes CRDs, and human-in-the-loop coordination
triggers:
  - set up a multi-agent collaboration system with HiClaw
  - create AI agent teams that work together in Matrix rooms
  - deploy HiClaw workers and managers for agent orchestration
  - configure Kubernetes-based agent collaboration with HiClaw
  - build transparent agent systems with human oversight
  - orchestrate multiple AI agents using HiClaw platform
  - integrate OpenClaw QwenPaw and Hermes agents together
  - manage agent credentials securely with Higress gateway
---

# HiClaw Multi-Agent Collaboration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

HiClaw is an open-source collaborative multi-agent runtime platform built on a Manager-Workers architecture. It enables multiple AI agents to collaborate in controlled, auditable Matrix rooms with full human visibility and intervention capabilities. HiClaw orchestrates agent containers (Manager + Workers) rather than implementing agent logic itself, supporting OpenClaw, QwenPaw, and Hermes runtimes.

## Core Concepts

- **Manager-Workers Architecture**: Central Manager orchestrates multiple Worker agents
- **Matrix Protocol**: All collaboration happens in Matrix IM rooms (Element client + Tuwunel server)
- **Kubernetes Native**: Custom Resource Definitions (CRDs) for declarative agent management
- **Human-in-the-Loop**: Every room includes humans who can observe and intervene
- **Shared File System**: MinIO-based storage reduces token consumption in multi-agent scenarios
- **Security**: Higress AI Gateway centralizes credentials; Workers only get consumer tokens

## Quick Installation (Docker)

### macOS / Linux

```bash
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

### Windows (PowerShell 7+)

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
$wc=New-Object Net.WebClient
$wc.Encoding=[Text.Encoding]::UTF8
iex $wc.DownloadString('https://higress.ai/hiclaw/install.ps1')
```

**Requirements**: Docker Desktop/Engine, 2 CPU cores, 4 GB RAM (8 GB recommended for multiple Workers)

The installer will:
1. Prompt for LLM provider (OpenAI, Qwen, or compatible API)
2. Request API key (stored securely)
3. Ask for network mode (local-only or external access)
4. Deploy all components (gateway, Matrix server, MinIO, Manager)

After installation, access Element Web at: `http://127.0.0.1:18088`

## Kubernetes Installation (Helm)

### Prerequisites

- Kubernetes 1.24+
- Helm 3.7+
- Default StorageClass configured

### Install with OpenAI-Compatible Provider

```bash
helm repo add higress.io https://higress.io/helm-charts
helm repo update

helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.llmBaseUrl=https://api.deepseek.com/v1 \
  --set credentials.defaultModel=deepseek-reasoner \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

### Install with Qwen (通义千问)

```bash
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=$QWEN_API_KEY \
  --set credentials.llmProvider=qwen \
  --set credentials.defaultModel=qwen3.5-plus \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

### Install with Custom Runtimes

```bash
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --set manager.runtime=copaw \
  --set worker.defaultRuntime=hermes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.llmBaseUrl=$LLM_BASE_URL \
  --set credentials.defaultModel=$MODEL_NAME \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

**Key Helm Values:**

| Value | Required | Description |
|-------|----------|-------------|
| `credentials.llmApiKey` | Yes | API key for LLM provider |
| `gateway.publicURL` | Yes | Public URL for Element Web access |
| `credentials.adminPassword` | Recommended | Matrix admin password (auto-generated if empty) |
| `credentials.llmProvider` | No | Provider: `openai-compat` (default), `qwen` |
| `credentials.defaultModel` | No | Default model (e.g., `gpt-5.4`, `qwen3.5-plus`) |
| `credentials.llmBaseUrl` | No | Base URL for OpenAI-compatible APIs |
| `manager.runtime` | No | Manager runtime: `openclaw`, `copaw`, `hermes` |
| `worker.defaultRuntime` | No | Default Worker runtime |

## Kubernetes Custom Resource Definitions

HiClaw uses Kubernetes CRDs for declarative agent management:

### Worker CRD

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: code-reviewer
  namespace: hiclaw-system
spec:
  runtime: openclaw
  systemPrompt: |
    You are an expert code reviewer specializing in Go and Python.
    Review code for security issues, performance problems, and best practices.
  mcp:
    servers:
      filesystem:
        url: docker://mcp/filesystem
        env:
          ALLOWED_DIRECTORIES: /workspace/projects
      github:
        url: docker://modelcontextprotocol/servers/github
        env:
          GITHUB_PERSONAL_ACCESS_TOKEN:
            secretKeyRef:
              name: github-pat
              key: token
  env:
    - name: CUSTOM_CONFIG
      value: "production"
  package: # Optional: pre-built Worker package
    source: https://skills.sh/packages/code-reviewer.zip
```

### Team CRD (Multi-Agent Collaboration)

```yaml
apiVersion: hiclaw.io/v1
kind: Team
metadata:
  name: backend-dev-team
  namespace: hiclaw-system
spec:
  leader:
    systemPrompt: |
      You coordinate a backend development team. Break tasks into subtasks
      and assign them to the database-expert and api-developer Workers.
    runtime: copaw
    mcp:
      servers:
        jira:
          url: docker://custom/jira-mcp
          env:
            JIRA_API_TOKEN:
              secretKeyRef:
                name: jira-creds
                key: token
  workers:
    - name: database-expert
      runtime: openclaw
      systemPrompt: |
        You are a PostgreSQL and MongoDB expert. Design schemas,
        write optimized queries, and handle migrations.
    - name: api-developer
      runtime: hermes
      systemPrompt: |
        You build REST and GraphQL APIs in Go. Write code, tests,
        and documentation following best practices.
      mcp:
        servers:
          filesystem:
            url: docker://mcp/filesystem
            env:
              ALLOWED_DIRECTORIES: /workspace/api-service
```

### Human CRD (Room Participants)

```yaml
apiVersion: hiclaw.io/v1
kind: Human
metadata:
  name: alice-engineer
  namespace: hiclaw-system
spec:
  name: "Alice"
  email: alice@example.com
  password: # Auto-generated if not specified
    secretKeyRef:
      name: alice-matrix-creds
      key: password
```

## HiClaw CLI Commands

### Worker Management

```bash
# List all Workers
hiclaw worker list

# Create Worker from YAML
hiclaw worker create -f worker-config.yaml

# Create Worker interactively
hiclaw worker create --name security-scanner \
  --runtime openclaw \
  --prompt "You scan code for security vulnerabilities"

# Delete Worker
hiclaw worker delete security-scanner

# View Worker logs
hiclaw worker logs security-scanner

# Update Worker
hiclaw worker update -f updated-worker.yaml
```

### Team Management

```bash
# List all Teams
hiclaw team list

# Create Team from YAML
hiclaw team create -f team-config.yaml

# Delete Team (removes all Workers and Matrix room)
hiclaw team delete backend-dev-team

# View Team status
hiclaw team describe backend-dev-team
```

### Human Management

```bash
# List all Humans
hiclaw human list

# Create Human user
hiclaw human create --name bob --email bob@example.com

# Delete Human user
hiclaw human delete bob

# Reset Human password
hiclaw human reset-password bob
```

### System Operations

```bash
# Check HiClaw status
hiclaw status

# View controller logs
hiclaw logs controller

# Restart components
hiclaw restart manager
hiclaw restart worker code-reviewer

# Upgrade HiClaw
hiclaw upgrade --version v1.1.2

# Uninstall HiClaw
hiclaw uninstall
```

## Docker Compose Configuration

HiClaw uses Docker Compose for local deployments. Configuration is stored in `~/.hiclaw/.env`:

```bash
# LLM Configuration
LLM_API_KEY=sk-xxxxx
LLM_BASE_URL=https://api.openai.com/v1
LLM_MODEL=gpt-4
LLM_PROVIDER=openai-compat

# Manager Configuration
MANAGER_RUNTIME=openclaw
MANAGER_SYSTEM_PROMPT="You are the HiClaw Manager..."

# Worker Defaults
WORKER_DEFAULT_RUNTIME=openclaw

# Matrix Server
MATRIX_SERVER_NAME=hiclaw.local
MATRIX_ADMIN_PASSWORD=secure-password-here

# MinIO Storage
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin123

# Higress Gateway
GATEWAY_PUBLIC_URL=http://localhost:18080

# Network Mode
EXPOSE_EXTERNAL=false
```

## Real-World Usage Patterns

### Pattern 1: Code Review Team

Create a team where OpenClaw Leader coordinates specialized Workers:

```yaml
apiVersion: hiclaw.io/v1
kind: Team
metadata:
  name: code-review-squad
spec:
  leader:
    runtime: openclaw
    systemPrompt: |
      Coordinate code reviews. Assign security checks to security-worker,
      performance analysis to perf-worker, and style checks to style-worker.
  workers:
    - name: security-worker
      runtime: openclaw
      systemPrompt: "Find security vulnerabilities: SQL injection, XSS, secrets in code"
    - name: perf-worker
      runtime: hermes
      systemPrompt: "Analyze performance: O(n) complexity, memory leaks, DB query optimization"
      mcp:
        servers:
          filesystem:
            url: docker://mcp/filesystem
            env:
              ALLOWED_DIRECTORIES: /workspace/repos
    - name: style-worker
      runtime: copaw
      systemPrompt: "Check code style against project guidelines, naming conventions, documentation"
```

### Pattern 2: Autonomous Development with Hermes

Deploy a Hermes Worker for autonomous coding tasks:

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: autonomous-developer
spec:
  runtime: hermes
  systemPrompt: |
    You are an autonomous developer. Write code, run tests, fix bugs,
    and iterate until tests pass. Use the filesystem MCP to read/write code.
  mcp:
    servers:
      filesystem:
        url: docker://mcp/filesystem
        env:
          ALLOWED_DIRECTORIES: /workspace/projects,/workspace/tests
      shell:
        url: docker://mcp/shell
        env:
          ALLOWED_COMMANDS: go,python3,npm,pytest,make
```

Chat with the Worker in Matrix:

```
You: Create a REST API in Go for a todo app with PostgreSQL backend

autonomous-developer: I'll create the API with the following structure:
1. Models (todo.go)
2. Handlers (handlers.go)
3. Database (db.go)
4. Main server (main.go)

[Worker writes code files via filesystem MCP]
[Worker runs tests via shell MCP]
[Worker iterates until tests pass]

autonomous-developer: ✅ API complete. 15 tests passing. Files created:
- cmd/api/main.go
- internal/handlers/todo.go
- internal/models/todo.go
- internal/db/postgres.go
```

### Pattern 3: Multi-Runtime Collaboration

Combine different runtimes for optimal task distribution:

```yaml
apiVersion: hiclaw.io/v1
kind: Team
metadata:
  name: full-stack-team
spec:
  leader:
    runtime: copaw  # QwenPaw for coordination
    systemPrompt: "Coordinate full-stack development tasks"
  workers:
    - name: frontend-dev
      runtime: hermes  # Autonomous code execution
      systemPrompt: "Build React/TypeScript frontends"
      mcp:
        servers:
          filesystem:
            url: docker://mcp/filesystem
            env:
              ALLOWED_DIRECTORIES: /workspace/frontend
    - name: backend-dev
      runtime: openclaw  # Deterministic API development
      systemPrompt: "Build Go REST APIs with proper error handling"
    - name: devops-engineer
      runtime: hermes  # Infrastructure automation
      systemPrompt: "Write Dockerfiles, K8s manifests, CI/CD pipelines"
      mcp:
        servers:
          filesystem:
            url: docker://mcp/filesystem
            env:
              ALLOWED_DIRECTORIES: /workspace/infra
          shell:
            url: docker://mcp/shell
            env:
              ALLOWED_COMMANDS: kubectl,docker,terraform
```

## MCP Server Integration

HiClaw supports Model Context Protocol (MCP) servers for extending agent capabilities:

### Built-in MCP Servers

```yaml
mcp:
  servers:
    # Filesystem access
    filesystem:
      url: docker://mcp/filesystem
      env:
        ALLOWED_DIRECTORIES: /workspace/project1,/workspace/project2
    
    # GitHub integration
    github:
      url: docker://modelcontextprotocol/servers/github
      env:
        GITHUB_PERSONAL_ACCESS_TOKEN:
          secretKeyRef:
            name: github-credentials
            key: pat
    
    # PostgreSQL access
    postgres:
      url: docker://modelcontextprotocol/servers/postgres
      env:
        POSTGRES_CONNECTION_STRING:
          secretKeyRef:
            name: db-credentials
            key: connection-string
```

### Nacos Skills Registry

Pull community skills from skills.sh (80,000+ skills):

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: k8s-operator
spec:
  runtime: openclaw
  systemPrompt: "You manage Kubernetes clusters"
  mcp:
    servers:
      nacos-skills:
        url: nacos://skills.sh/registry
        env:
          NACOS_NAMESPACE: kubernetes
          SKILLS_FILTER: "k8s-*,kubectl-*,helm-*"
```

## Credential Management

HiClaw separates credentials from Workers for security:

### Kubernetes Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: github-pat
  namespace: hiclaw-system
type: Opaque
stringData:
  token: ghp_xxxxxxxxxxxx
---
apiVersion: v1
kind: Secret
metadata:
  name: database-creds
  namespace: hiclaw-system
type: Opaque
stringData:
  connection-string: postgresql://user:pass@host:5432/db
```

### Reference in Worker

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: github-worker
spec:
  mcp:
    servers:
      github:
        url: docker://modelcontextprotocol/servers/github
        env:
          GITHUB_PERSONAL_ACCESS_TOKEN:
            secretKeyRef:
              name: github-pat
              key: token
```

**Security Note**: Workers receive consumer tokens only. Real credentials stay in Higress Gateway—Workers never see them.

## Upgrade HiClaw

### Docker Installation

```bash
# Upgrade to latest version (preserves data)
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)

# Upgrade to specific version
HICLAW_VERSION=v1.1.2 bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

### Helm Installation

```bash
# Update Helm repo
helm repo update

# Upgrade to latest chart
helm upgrade hiclaw higress.io/hiclaw \
  -n hiclaw-system \
  --reuse-values

# Upgrade to specific version
helm upgrade hiclaw higress.io/hiclaw \
  -n hiclaw-system \
  --reuse-values \
  --version 1.1.2
```

## Troubleshooting

### Workers Not Responding

```bash
# Check Worker status
hiclaw worker describe <worker-name>

# View Worker logs
hiclaw worker logs <worker-name>

# Restart Worker
hiclaw worker restart <worker-name>
```

### Matrix Connection Issues

```bash
# Check Tuwunel (Matrix server) status
docker ps | grep tuwunel

# View Matrix server logs
docker logs hiclaw-tuwunel

# Restart Matrix server
docker restart hiclaw-tuwunel
```

### LLM API Failures

```bash
# Check Higress Gateway logs
docker logs hiclaw-gateway

# Test LLM connectivity
curl -X POST http://localhost:18080/v1/chat/completions \
  -H "Authorization: Bearer consumer-token-xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "messages": [{"role": "user", "content": "test"}]
  }'
```

### Storage Issues (MinIO)

```bash
# Check MinIO status
docker ps | grep minio

# Access MinIO console
open http://localhost:9001
# Login with credentials from ~/.hiclaw/.env

# View MinIO logs
docker logs hiclaw-minio
```

### Kubernetes Issues

```bash
# Check HiClaw controller status
kubectl get pods -n hiclaw-system

# View controller logs
kubectl logs -n hiclaw-system deployment/hiclaw-controller

# Check CRD status
kubectl get workers -n hiclaw-system
kubectl get teams -n hiclaw-system
kubectl get humans -n hiclaw-system

# Describe specific resource
kubectl describe worker <worker-name> -n hiclaw-system
```

### Common Error: "Worker Creation Failed"

**Cause**: Insufficient resources or invalid runtime specification

**Solution**:
```bash
# Check available resources
docker stats

# Verify runtime is valid
hiclaw worker create --name test --runtime openclaw --prompt "test"

# If using custom image, ensure it exists
docker images | grep hiclaw
```

### Common Error: "MCP Server Unreachable"

**Cause**: MCP server URL incorrect or container not running

**Solution**:
```bash
# List available MCP servers
docker ps | grep mcp

# Check MCP server logs
docker logs <mcp-container-name>

# Verify MCP server URL format
# Correct: docker://mcp/filesystem
# Incorrect: http://mcp-filesystem:8080
```

## Best Practices

1. **Use QwenPaw (copaw) for Leaders**: Better at task decomposition and coordination
2. **Use Hermes for Code Execution**: Autonomous iteration until tests pass
3. **Use OpenClaw for Deterministic Tasks**: API design, documentation, planning
4. **Limit Filesystem Access**: Only grant MCP permissions to required directories
5. **Store Credentials in Secrets**: Never hardcode API keys in Worker specs
6. **Monitor Resource Usage**: Each Worker consumes ~500MB-1GB RAM
7. **Use Teams for Complex Tasks**: Let Leader delegate instead of managing Workers manually
8. **Enable Human Oversight**: Always include humans in critical Matrix rooms

## Resources

- **Official Website**: https://hiclaw.io
- **GitHub Repository**: https://github.com/agentscope-ai/HiClaw
- **Documentation**: https://github.com/agentscope-ai/HiClaw/tree/main/docs
- **Skills Marketplace**: https://skills.sh
- **Discord Community**: https://discord.com/invite/NVjNA4BAVw
- **Helm Charts**: https://higress.io/helm-charts
