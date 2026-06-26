---
name: hiclaw-collaborative-agent-os
description: Deploy and orchestrate collaborative multi-agent teams using HiClaw's Manager-Workers architecture on Docker or Kubernetes with Matrix rooms for human oversight
triggers:
  - "set up HiClaw multi-agent collaboration platform"
  - "deploy collaborative AI agents with HiClaw"
  - "create a team of AI workers with HiClaw"
  - "configure Manager-Workers architecture for agents"
  - "install HiClaw on Kubernetes with Helm"
  - "manage AI agent teams in Matrix rooms"
  - "orchestrate multiple agents with human-in-the-loop"
  - "deploy OpenClaw and QwenPaw workers together"
---

# HiClaw Collaborative Agent OS

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

HiClaw is an open-source collaborative multi-agent runtime platform built on a Manager-Workers architecture. It enables multiple AI agents to collaborate in Matrix-based rooms with full human visibility and intervention capabilities. Workers operate with consumer tokens while real credentials stay in the centralized Higress AI gateway, providing enterprise-grade security.

The platform supports multiple agent runtimes (OpenClaw, QwenPaw/CoPaw, Hermes) working together, uses MinIO for shared file systems to reduce token consumption, and provides Kubernetes-native declarative resource management.

## Installation

### Docker Installation (Local/Single Machine)

**macOS/Linux:**
```bash
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

**Windows (PowerShell 7+):**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
$wc = New-Object Net.WebClient
$wc.Encoding = [Text.Encoding]::UTF8
iex $wc.DownloadString('https://higress.ai/hiclaw/install.ps1')
```

The installer will prompt for:
- LLM provider selection (OpenAI-compatible APIs supported)
- API key
- Network mode (local-only or external access)

**Requirements:**
- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- 2 CPU cores + 4 GB RAM minimum
- 4 cores + 8 GB RAM recommended for multiple Workers

### Kubernetes Installation (Helm)

**Prerequisites:**
- Kubernetes 1.24+
- Helm 3.7+
- Default StorageClass configured

**Install with OpenAI:**
```bash
helm repo add higress.io https://higress.io/helm-charts
helm repo update

helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

**Install with OpenAI-compatible provider:**
```bash
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.llmBaseUrl=https://api.deepseek.com/v1 \
  --set credentials.defaultModel=deepseek-chat \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

**Install with Qwen (通义千问):**
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

**Key Helm Configuration Values:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `credentials.llmApiKey` | Yes | API key from LLM provider |
| `gateway.publicURL` | Yes | Public URL for Element Web access |
| `credentials.adminPassword` | Recommended | Matrix admin password (auto-generated if omitted) |
| `credentials.llmProvider` | No | Provider: `openai-compat` (default), `qwen` |
| `credentials.defaultModel` | No | Model name (default: `gpt-5.4`) |
| `credentials.llmBaseUrl` | No | Base URL for OpenAI-compatible APIs |
| `manager.runtime` | No | Manager runtime: `openclaw` (default), `copaw`, `hermes` |
| `worker.defaultRuntime` | No | Default Worker runtime: `openclaw` (default), `copaw`, `hermes` |

## Accessing HiClaw

After installation, access Element Web (Matrix client):
- Docker: http://127.0.0.1:18088
- Kubernetes (port-forward): `kubectl port-forward -n hiclaw-system svc/element-web 18088:8080`

The Manager agent will greet you in Matrix and guide you through creating your first Worker.

## Declarative Resource Management (Kubernetes)

HiClaw uses Kubernetes-style CRDs (Custom Resource Definitions) for declarative agent management.

### Worker CRD

Create a Worker agent with declarative YAML:

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: code-reviewer
  namespace: hiclaw-system
spec:
  runtime: openclaw  # or copaw, hermes
  profile: |
    You are a senior code reviewer specializing in Go and Python.
    Review code for security issues, performance problems, and best practices.
    Provide constructive feedback with specific suggestions.
  mcp:
    servers:
      - name: github
        type: github
        config:
          GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_TOKEN}
      - name: filesystem
        type: filesystem
        config:
          allowedDirectories:
            - /workspace/code
  env:
    - name: REVIEW_STRICTNESS
      value: "high"
```

Apply the Worker:
```bash
kubectl apply -f worker-code-reviewer.yaml
```

### Team CRD

Create a Team of Workers managed by a Team Leader:

```yaml
apiVersion: hiclaw.io/v1
kind: Team
metadata:
  name: dev-team
  namespace: hiclaw-system
spec:
  leader:
    runtime: openclaw
    profile: |
      You are a technical lead coordinating a development team.
      Break down tasks, assign to appropriate specialists, and synthesize results.
    mcp:
      servers:
        - name: github
          type: github
          config:
            GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_TOKEN}
  members:
    - name: backend-dev
      runtime: copaw
      profile: Go backend development specialist
      mcp:
        servers:
          - name: filesystem
            type: filesystem
            config:
              allowedDirectories:
                - /workspace/backend
    - name: frontend-dev
      runtime: hermes
      profile: React/TypeScript frontend specialist
      mcp:
        servers:
          - name: filesystem
            type: filesystem
            config:
              allowedDirectories:
                - /workspace/frontend
```

Apply the Team:
```bash
kubectl apply -f team-dev.yaml
```

### Human CRD

Invite human users to participate:

```yaml
apiVersion: hiclaw.io/v1
kind: Human
metadata:
  name: alice
  namespace: hiclaw-system
spec:
  displayName: Alice (Product Manager)
  password: ${ALICE_PASSWORD}  # Set via Secret
```

Apply the Human:
```bash
kubectl apply -f human-alice.yaml
```

## Working with the hiclaw CLI

HiClaw v1.1.0+ includes a CLI for managing resources:

```bash
# List all Workers
hiclaw worker list

# Create a Worker from template
hiclaw worker create --name data-analyst --runtime openclaw

# Delete a Worker
hiclaw worker delete data-analyst

# List Teams
hiclaw team list

# Create a Team
hiclaw team create --name research-team --leader-runtime copaw

# Add Worker to Team
hiclaw team add-member research-team --worker web-scraper

# View Team status
hiclaw team status research-team

# Invite human user
hiclaw human invite --username bob --display-name "Bob (Designer)"
```

## Runtime Options

### OpenClaw
- Deterministic, rule-based agent
- Best for: Task orchestration, workflow management
- Profile: Highly structured, follows explicit instructions

### QwenPaw (CoPaw)
- LLM-powered agent optimized for Qwen models
- Best for: Natural language understanding, decision-making
- Profile: Flexible reasoning, context-aware

### Hermes
- Autonomous coding agent runtime
- Best for: Code generation, debugging, autonomous execution
- Profile: Programming tasks, file system operations

## MCP Server Configuration

Model Context Protocol (MCP) servers provide Workers with tools and data access.

### Common MCP Servers

**GitHub Integration:**
```yaml
mcp:
  servers:
    - name: github
      type: github
      config:
        GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_TOKEN}
```

**Filesystem Access:**
```yaml
mcp:
  servers:
    - name: filesystem
      type: filesystem
      config:
        allowedDirectories:
          - /workspace/project
          - /workspace/shared
```

**PostgreSQL Database:**
```yaml
mcp:
  servers:
    - name: postgres
      type: postgres
      config:
        DATABASE_URL: ${DATABASE_URL}
```

**Custom MCP Server:**
```yaml
mcp:
  servers:
    - name: custom-api
      type: custom
      config:
        ENDPOINT: https://api.example.com
        API_KEY: ${CUSTOM_API_KEY}
```

### Nacos Skills Registry

HiClaw integrates with [skills.sh](https://skills.sh) (80,000+ community skills) via Nacos:

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: data-processor
spec:
  runtime: copaw
  profile: Data processing and transformation specialist
  mcp:
    servers:
      - name: nacos-skills
        type: nacos
        config:
          NACOS_SERVER: nacos.hiclaw-system.svc.cluster.local:8848
          NACOS_NAMESPACE: hiclaw
          SKILL_CATEGORY: data-processing
```

Workers pull skills on-demand without exposing credentials (credentials stay in gateway).

## Manager-Workers Pattern

### Basic Workflow

1. **User sends request** to Manager in Matrix room
2. **Manager analyzes** and creates Worker(s) if needed
3. **Manager invites** Worker(s) to task-specific room
4. **Workers collaborate**, using MinIO shared filesystem
5. **Manager synthesizes** results and responds to user
6. **Human observes** all interactions, can intervene anytime

### Example: Multi-Runtime Collaboration

```yaml
apiVersion: hiclaw.io/v1
kind: Team
metadata:
  name: fullstack-project
spec:
  leader:
    runtime: openclaw  # Deterministic orchestration
    profile: Project coordinator - break down tasks and assign work
  members:
    - name: architect
      runtime: copaw  # LLM reasoning for design decisions
      profile: System architect - design scalable solutions
    - name: coder
      runtime: hermes  # Autonomous code execution
      profile: Full-stack developer - implement features
    - name: reviewer
      runtime: openclaw  # Deterministic code review
      profile: Code reviewer - ensure quality and standards
```

Each runtime excels at different tasks, working together in the same room.

## Upgrade

### Docker Upgrade

Preserves all data (Matrix rooms, MinIO files, configuration):

```bash
# Latest version
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)

# Specific version
HICLAW_VERSION=v1.1.2 bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

### Kubernetes Upgrade

```bash
helm repo update

helm upgrade hiclaw higress.io/hiclaw \
  -n hiclaw-system \
  --reuse-values
```

Use `--set` to override specific values during upgrade.

## Configuration

### Environment Variables (Docker)

Configuration stored in `.hiclaw/hiclaw.env`:

```bash
# LLM Configuration
LLM_PROVIDER=openai-compat
LLM_API_KEY=sk-...
LLM_BASE_URL=https://api.openai.com/v1
DEFAULT_MODEL=gpt-4

# Manager Runtime
MANAGER_RUNTIME=openclaw

# Worker Default Runtime
WORKER_DEFAULT_RUNTIME=openclaw

# Network
PUBLIC_URL=http://localhost:18080

# Matrix Admin
MATRIX_ADMIN_PASSWORD=...
```

### Custom Worker Environment Variables

Pass environment variables to specific Workers:

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: data-analyst
spec:
  runtime: copaw
  profile: Data analysis specialist
  env:
    - name: PANDAS_COMPUTE_BACKEND
      value: dask
    - name: MAX_MEMORY_GB
      value: "8"
    - name: API_TIMEOUT
      value: "30"
```

### Custom Resource Limits (Kubernetes)

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: ml-trainer
spec:
  runtime: hermes
  profile: Machine learning model training
  resources:
    requests:
      memory: "4Gi"
      cpu: "2"
    limits:
      memory: "8Gi"
      cpu: "4"
```

## Token Budget Management

Configure Token Plans for Workers (v1.1.1+):

```yaml
apiVersion: hiclaw.io/v1
kind: Worker
metadata:
  name: budget-constrained-worker
spec:
  runtime: copaw
  profile: Cost-conscious assistant
  tokenPlan:
    provider: qwen
    plan: basic  # or premium
    maxTokensPerRequest: 4000
    dailyLimit: 100000
```

## Common Patterns

### Pattern: Task Decomposition

Manager breaks complex task into subtasks, assigns to specialist Workers:

```yaml
apiVersion: hiclaw.io/v1
kind: Team
metadata:
  name: research-project
spec:
  leader:
    runtime: openclaw
    profile: |
      Break down research requests into:
      1. Data collection (assign to web-scraper)
      2. Data analysis (assign to analyst)
      3. Report generation (assign to writer)
  members:
    - name: web-scraper
      runtime: hermes
      profile: Web scraping and data extraction
    - name: analyst
      runtime: copaw
      profile: Statistical analysis and insights
    - name: writer
      runtime: copaw
      profile: Technical report writing
```

### Pattern: Human Review Checkpoint

Include Human in Team for critical decisions:

```yaml
apiVersion: hiclaw.io/v1
kind: Team
metadata:
  name: deployment-pipeline
spec:
  leader:
    runtime: openclaw
    profile: CI/CD pipeline coordinator
  members:
    - name: tester
      runtime: hermes
      profile: Automated testing
    - name: reviewer
      runtime: copaw
      profile: Code review
  humans:
    - name: devops-lead
      role: approver
      requiredFor:
        - production-deployment
        - infrastructure-changes
```

### Pattern: Shared File Exchange

Workers use MinIO for large data exchange:

```go
// Worker A writes data
package main

import (
    "github.com/minio/minio-go/v7"
    "github.com/minio/minio-go/v7/pkg/credentials"
)

func uploadResults(data []byte, filename string) error {
    client, err := minio.New("minio.hiclaw-system.svc.cluster.local:9000", &minio.Options{
        Creds:  credentials.NewEnvAWS(),
        Secure: false,
    })
    if err != nil {
        return err
    }

    _, err = client.PutObject(
        context.Background(),
        "hiclaw-shared",
        filename,
        bytes.NewReader(data),
        int64(len(data)),
        minio.PutObjectOptions{ContentType: "application/octet-stream"},
    )
    return err
}
```

```go
// Worker B reads data
func downloadResults(filename string) ([]byte, error) {
    client, err := minio.New("minio.hiclaw-system.svc.cluster.local:9000", &minio.Options{
        Creds:  credentials.NewEnvAWS(),
        Secure: false,
    })
    if err != nil {
        return nil, err
    }

    object, err := client.GetObject(
        context.Background(),
        "hiclaw-shared",
        filename,
        minio.GetObjectOptions{},
    )
    if err != nil {
        return nil, err
    }
    defer object.Close()

    return io.ReadAll(object)
}
```

## Troubleshooting

### Workers Not Starting

**Check Worker status:**
```bash
kubectl get workers -n hiclaw-system
kubectl describe worker <worker-name> -n hiclaw-system
```

**Common causes:**
- Invalid runtime specified (must be `openclaw`, `copaw`, or `hermes`)
- MCP server configuration errors
- Resource constraints (CPU/memory limits)

**View controller logs:**
```bash
kubectl logs -n hiclaw-system -l app=hiclaw-controller --tail=100
```

### Manager Not Responding

**Check Manager pod:**
```bash
kubectl get pods -n hiclaw-system -l app=hiclaw-manager
kubectl logs -n hiclaw-system -l app=hiclaw-manager --tail=100
```

**Verify LLM connectivity:**
```bash
kubectl exec -n hiclaw-system -it deploy/hiclaw-manager -- sh
curl -H "Authorization: Bearer $LLM_API_KEY" $LLM_BASE_URL/models
```

### Matrix Connection Issues

**Check Tuwunel (Matrix server):**
```bash
kubectl get pods -n hiclaw-system -l app=tuwunel
kubectl logs -n hiclaw-system -l app=tuwunel --tail=100
```

**Test Matrix API:**
```bash
kubectl port-forward -n hiclaw-system svc/tuwunel 8008:8008
curl http://localhost:8008/_matrix/client/versions
```

### MCP Server Failures

**Invalid credential reference:**
- Ensure environment variables are set in Secrets
- Credentials stay in gateway, Workers receive consumer tokens only

**Check gateway configuration:**
```bash
kubectl get configmap -n hiclaw-system higress-config -o yaml
```

**Verify MCP server pod (if deployed separately):**
```bash
kubectl get pods -n hiclaw-system -l mcp-server=<server-name>
```

### MinIO File Access Issues

**Check MinIO service:**
```bash
kubectl get pods -n hiclaw-system -l app=minio
kubectl logs -n hiclaw-system -l app=minio --tail=100
```

**Test MinIO connectivity:**
```bash
kubectl port-forward -n hiclaw-system svc/minio 9000:9000
mc alias set hiclaw http://localhost:9000 $MINIO_ROOT_USER $MINIO_ROOT_PASSWORD
mc ls hiclaw/hiclaw-shared
```

### Docker Installation Issues

**Check container status:**
```bash
docker ps -a | grep hiclaw
docker logs hiclaw-manager
docker logs hiclaw-matrix
```

**Verify network:**
```bash
docker network inspect hiclaw-network
```

**Check disk space:**
```bash
docker system df
```

**Reset installation (preserves data):**
```bash
docker restart hiclaw-manager hiclaw-matrix hiclaw-minio
```

### Performance Optimization

**Reduce token consumption:**
- Use MinIO shared files instead of passing large data in messages
- Configure appropriate `maxTokensPerRequest` in Token Plans
- Use deterministic runtimes (OpenClaw) for structured tasks

**Scale Workers horizontally:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: worker-scaler
  namespace: hiclaw-system
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: worker-<worker-name>
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

## Uninstallation

### Docker

**macOS/Linux:**
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/higress-group/hiclaw/main/install/hiclaw-install.sh) uninstall
```

**Windows:**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
$wc = New-Object Net.WebClient
$wc.Encoding = [Text.Encoding]::UTF8
$s = $wc.DownloadString('https://raw.githubusercontent.com/higress-group/hiclaw/main/install/hiclaw-install.ps1')
& ([scriptblock]::Create($s)) uninstall
```

Removes all containers, volumes, networks, and configuration.

### Kubernetes

```bash
helm uninstall hiclaw -n hiclaw-system
kubectl delete namespace hiclaw-system
```

To also remove PVCs (caution: deletes all Matrix rooms and MinIO files):
```bash
kubectl delete pvc -n hiclaw-system --all
```

## Resources

- **Homepage:** https://hiclaw.io
- **GitHub:** https://github.com/agentscope-ai/HiClaw
- **Discord:** https://discord.com/invite/NVjNA4BAVw
- **Skills Registry:** https://skills.sh
- **Documentation:** https://github.com/higress-group/hiclaw/tree/main/docs
- **License:** Apache-2.0
