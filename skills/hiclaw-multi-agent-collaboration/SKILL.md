---
name: hiclaw-multi-agent-collaboration
description: Set up and manage collaborative multi-agent teams using HiClaw, a Kubernetes-native platform with Matrix rooms for human-in-the-loop AI coordination
triggers:
  - set up a collaborative AI agent team with HiClaw
  - create multiple agents working together in Matrix rooms
  - deploy HiClaw on Kubernetes with Helm
  - configure OpenClaw and QwenPaw workers in HiClaw
  - manage agent teams with human oversight using HiClaw
  - integrate MCP servers with HiClaw agents
  - troubleshoot HiClaw multi-agent deployments
  - orchestrate AI agents with Manager-Workers architecture
---

# HiClaw Multi-Agent Collaboration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

HiClaw is an open-source collaborative multi-agent runtime platform that enables multiple AI agents to work together in Matrix rooms with full human visibility and intervention. It uses a Manager-Workers architecture where a central Manager orchestrates multiple Worker agents, all visible in a shared IM interface.

## Key Concepts

- **Manager-Workers Architecture**: Manager agent coordinates Worker agents, eliminating need to oversee individual workers
- **Matrix Protocol**: Decentralized IM protocol (Tuwunel server + Element client) for agent-human collaboration
- **Multi-Runtime Support**: OpenClaw (deterministic), QwenPaw (Qwen models), and Hermes (autonomous coding) workers
- **MinIO Shared Storage**: File system for inter-agent data exchange, reducing token consumption
- **Higress AI Gateway**: Centralized credential management - workers use consumer tokens, real API keys stay in gateway
- **Kubernetes-Native**: Custom Resource Definitions (CRDs) for declarative agent management

## Installation

### Docker Desktop (Quick Start)

**macOS / Linux:**
```bash
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

**Windows PowerShell 7+:**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
$wc = New-Object Net.WebClient
$wc.Encoding = [Text.Encoding]::UTF8
iex $wc.DownloadString('https://higress.ai/hiclaw/install.ps1')
```

The installer prompts for:
1. LLM provider (OpenAI-compatible APIs supported)
2. API key (stored securely in Higress gateway)
3. Network mode (local-only or external access)

**Access**: Open http://127.0.0.1:18088 in browser, log into Element Web

### Kubernetes (Helm)

**Prerequisites:**
- Kubernetes 1.24+
- Helm 3.7+
- Default StorageClass for PVCs

**Install with OpenAI-compatible API:**
```bash
helm repo add higress.io https://higress.io/helm-charts
helm repo update

helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.llmBaseUrl=https://api.example.com/v1 \
  --set credentials.defaultModel=gpt-4 \
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

**With custom runtimes (QwenPaw Manager + Hermes Workers):**
```bash
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --set manager.runtime=copaw \
  --set worker.defaultRuntime=hermes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

## Declarative Resource Management

HiClaw uses Kubernetes-style CRDs for managing Workers, Teams, and Humans.

### Worker Custom Resource

Create a Worker agent with MCP servers:

```yaml
# worker-example.yaml
apiVersion: hiclaw.higress.io/v1alpha1
kind: Worker
metadata:
  name: data-analyst
  namespace: hiclaw-system
spec:
  runtime: openclaw
  model: gpt-4
  description: "Data analysis specialist with filesystem and database access"
  env:
    - name: CUSTOM_VAR
      value: "custom-value"
  mcp:
    servers:
      - name: filesystem
        command: npx
        args:
          - "-y"
          - "@modelcontextprotocol/server-filesystem"
          - "/workspace"
        env:
          - name: NODE_ENV
            value: production
      - name: postgres
        command: npx
        args:
          - "-y"
          - "@modelcontextprotocol/server-postgres"
        env:
          - name: POSTGRES_CONNECTION
            value: "postgresql://user:pass@postgres:5432/db"
```

Apply the Worker:
```bash
kubectl apply -f worker-example.yaml

# Check status
kubectl get workers -n hiclaw-system
kubectl describe worker data-analyst -n hiclaw-system
```

### Team Custom Resource

Define a team of Workers with a Team Leader:

```yaml
# team-example.yaml
apiVersion: hiclaw.higress.io/v1alpha1
kind: Team
metadata:
  name: dev-team
  namespace: hiclaw-system
spec:
  description: "Full-stack development team"
  workers:
    - backend-engineer
    - frontend-engineer
    - data-analyst
  teamLeader:
    runtime: openclaw
    model: gpt-4
    description: "Coordinates development tasks across team members"
    mcp:
      servers:
        - name: github
          command: npx
          args:
            - "-y"
            - "@modelcontextprotocol/server-github"
          env:
            - name: GITHUB_PERSONAL_ACCESS_TOKEN
              valueFrom:
                secretKeyRef:
                  name: github-credentials
                  key: token
```

Apply the Team:
```bash
kubectl apply -f team-example.yaml

# List teams
kubectl get teams -n hiclaw-system

# Invite team to Matrix room (done via Manager in Element)
```

### Human Custom Resource

Register human team members:

```yaml
# human-example.yaml
apiVersion: hiclaw.higress.io/v1alpha1
kind: Human
metadata:
  name: alice
  namespace: hiclaw-system
spec:
  displayName: "Alice Smith"
  matrixUserId: "@alice:hiclaw.local"
  email: "alice@example.com"
  role: "coordinator"
```

Apply the Human:
```bash
kubectl apply -f human-example.yaml

# List humans
kubectl get humans -n hiclaw-system
```

## Worker Runtimes

### OpenClaw (Default)
Deterministic, tool-calling agent with explicit reasoning steps. Best for structured tasks.

```yaml
spec:
  runtime: openclaw
  model: gpt-4
```

### QwenPaw (CoPaw)
Optimized for Qwen models, 80% less memory than OpenClaw. Ideal for resource-constrained environments.

```yaml
spec:
  runtime: copaw
  model: qwen3.5-plus
```

### Hermes
Autonomous coding agent with file system access and code execution. Best for development tasks.

```yaml
spec:
  runtime: hermes
  model: gpt-4
```

## MCP Server Configuration

### Declarative MCP on Worker

```yaml
apiVersion: hiclaw.higress.io/v1alpha1
kind: Worker
metadata:
  name: devops-agent
spec:
  runtime: openclaw
  mcp:
    servers:
      - name: docker
        command: npx
        args: ["-y", "@modelcontextprotocol/server-docker"]
      - name: kubernetes
        command: npx
        args: ["-y", "@modelcontextprotocol/server-kubernetes"]
        env:
          - name: KUBECONFIG
            value: /workspace/.kube/config
```

### Declarative MCP on Team Leader

```yaml
apiVersion: hiclaw.higress.io/v1alpha1
kind: Team
metadata:
  name: infra-team
spec:
  teamLeader:
    runtime: openclaw
    mcp:
      servers:
        - name: slack
          command: npx
          args: ["-y", "@modelcontextprotocol/server-slack"]
          env:
            - name: SLACK_BOT_TOKEN
              valueFrom:
                secretKeyRef:
                  name: slack-creds
                  key: bot-token
```

### Nacos Skills Registry

Workers can pull skills from a remote Nacos registry:

```yaml
spec:
  mcp:
    nacos:
      enabled: true
      serverAddr: "nacos.example.com:8848"
      namespace: "hiclaw-skills"
      group: "ai-registry"
```

Skills are pulled on-demand from https://skills.sh (80,000+ community skills). Workers access via consumer tokens - real credentials stay in Higress gateway.

## Common Patterns

### Create Worker via CLI

```bash
# Install hiclaw CLI (if using Kubernetes deployment)
kubectl exec -it deployment/hiclaw-manager -n hiclaw-system -- hiclaw

# Create worker
hiclaw worker create \
  --name code-reviewer \
  --runtime openclaw \
  --model gpt-4 \
  --description "Code review specialist"

# List workers
hiclaw worker list

# Delete worker
hiclaw worker delete code-reviewer
```

### Manager Commands in Matrix Room

In the Element Web interface, chat with the Manager:

```
You: Create a worker named "security-auditor" with runtime openclaw

Manager: [Creates worker and responds with confirmation]

You: Create a team called "security-team" with workers security-auditor and devops-agent

Manager: [Creates team, provides Matrix room invitation link]
```

### Multi-Agent Collaboration Flow

1. **Manager receives task** from human in Matrix room
2. **Manager analyzes task**, determines required Workers
3. **Manager creates Workers** if needed (or uses existing)
4. **Manager creates Team**, invites Workers to new Matrix room
5. **Workers collaborate** in room, sharing files via MinIO
6. **Human monitors** in real-time, intervenes if needed
7. **Manager synthesizes results**, reports back to human

### File Sharing Between Agents

Workers share files via MinIO (S3-compatible):

```go
// Worker writes file to MinIO
package main

import (
    "github.com/minio/minio-go/v7"
    "github.com/minio/minio-go/v7/pkg/credentials"
)

func uploadToSharedStorage(fileName, content string) error {
    client, err := minio.New("minio:9000", &minio.Options{
        Creds:  credentials.NewStaticV4(os.Getenv("MINIO_ACCESS_KEY"), os.Getenv("MINIO_SECRET_KEY"), ""),
        Secure: false,
    })
    if err != nil {
        return err
    }

    _, err = client.FPutObject(context.Background(), "hiclaw-shared", fileName, content, minio.PutObjectOptions{})
    return err
}
```

Workers reference files in Matrix messages - other Workers download from MinIO, avoiding token bloat from embedding file contents.

## Configuration

### Helm Values (Kubernetes)

Key Helm chart values:

```yaml
# values.yaml
credentials:
  llmApiKey: ""              # Required
  llmProvider: openai-compat # openai-compat, qwen
  llmBaseUrl: ""             # For non-OpenAI providers
  defaultModel: gpt-4
  adminPassword: ""          # Auto-generated if empty

gateway:
  publicURL: ""              # Required - where users access Element Web
  
manager:
  runtime: openclaw          # openclaw, copaw, hermes
  
worker:
  defaultRuntime: openclaw   # Default for new Workers

minio:
  persistence:
    enabled: true
    size: 10Gi

tuwunel:
  persistence:
    enabled: true
    size: 5Gi
```

### Environment Variables (Docker)

When using Docker installation, credentials stored in `~/.hiclaw/env`:

```bash
# ~/.hiclaw/env
LLM_API_KEY=sk-...
LLM_BASE_URL=https://api.openai.com/v1
DEFAULT_MODEL=gpt-4
ADMIN_PASSWORD=secure-password
```

### Worker Environment Variables

Custom environment variables for Workers:

```yaml
spec:
  env:
    - name: API_TIMEOUT
      value: "30"
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

## Troubleshooting

### Worker Not Responding in Matrix

```bash
# Check Worker pod status
kubectl get pods -n hiclaw-system -l app=hiclaw-worker

# Check Worker logs
kubectl logs -n hiclaw-system deployment/hiclaw-worker-<name> --tail=100

# Describe Worker CR
kubectl describe worker <name> -n hiclaw-system
```

Common issues:
- **Model not available**: Check `spec.model` matches provider capabilities
- **MCP server failed**: Check MCP server command/args, verify npx package exists
- **Network errors**: Verify Higress gateway connectivity

### Manager Not Creating Workers

```bash
# Check Manager logs
kubectl logs -n hiclaw-system deployment/hiclaw-manager --tail=100

# Check controller logs
kubectl logs -n hiclaw-system deployment/hiclaw-controller --tail=100

# Verify RBAC permissions
kubectl auth can-i create workers --as=system:serviceaccount:hiclaw-system:hiclaw-controller -n hiclaw-system
```

### MinIO File Sharing Issues

```bash
# Check MinIO pod
kubectl get pods -n hiclaw-system -l app=minio

# Access MinIO console
kubectl port-forward -n hiclaw-system svc/minio 9001:9001
# Open http://localhost:9001

# Check bucket exists
kubectl exec -n hiclaw-system deployment/minio -- mc ls local/
```

### Matrix Room Not Accessible

```bash
# Check Tuwunel (Matrix server) status
kubectl get pods -n hiclaw-system -l app=tuwunel

# Check Matrix federation (if using external access)
kubectl logs -n hiclaw-system deployment/tuwunel --tail=50

# Reset admin password
kubectl delete secret -n hiclaw-system tuwunel-admin-credentials
kubectl rollout restart deployment/tuwunel -n hiclaw-system
```

### Upgrade Issues

```bash
# Upgrade to latest version
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)

# Upgrade to specific version
HICLAW_VERSION=v1.1.2 bash <(curl -sSL https://higress.ai/hiclaw/install.sh)

# Helm upgrade (Kubernetes)
helm upgrade hiclaw higress.io/hiclaw \
  -n hiclaw-system \
  --reuse-values \
  --version v1.1.2
```

Data preserved across upgrades - all Workers, Teams, and chat history retained.

## Advanced Usage

### Custom Worker Package with SOUL.md

Create Worker with personality/instructions:

```bash
# Create Worker package directory
mkdir -p my-worker-package
cd my-worker-package

# Create SOUL.md (optional)
cat > SOUL.md <<EOF
# Code Reviewer Agent

You are a meticulous code reviewer specializing in Go and Kubernetes.

## Review Checklist
- Error handling completeness
- Resource cleanup (defer statements)
- Context propagation
- Security vulnerabilities
- Performance implications

Always provide constructive feedback with code examples.
EOF

# Create Worker spec
cat > worker.yaml <<EOF
apiVersion: hiclaw.higress.io/v1alpha1
kind: Worker
metadata:
  name: go-code-reviewer
spec:
  runtime: openclaw
  model: gpt-4
  description: "Go and Kubernetes code review specialist"
EOF

# Package and apply
kubectl apply -f worker.yaml
```

### Token Budget Management

Configure token plans for cost control:

```yaml
spec:
  tokenBudget:
    daily: 100000
    perRequest: 4000
    alertThreshold: 0.8  # Alert at 80% usage
```

### Namespace-Scoped Deployments

Deploy HiClaw with namespace-scoped permissions:

```bash
helm install hiclaw higress.io/hiclaw \
  -n team-alpha --create-namespace \
  --set rbac.scope=namespace \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set gateway.publicURL=http://localhost:18080
```

Isolates Workers/Teams to specific namespace, useful for multi-tenant environments.

## Uninstallation

### Docker

**macOS / Linux:**
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/higress-group/hiclaw/main/install/hiclaw-install.sh) uninstall
```

**Windows PowerShell:**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
$wc = New-Object Net.WebClient
$wc.Encoding = [Text.Encoding]::UTF8
$s = $wc.DownloadString('https://raw.githubusercontent.com/higress-group/hiclaw/main/install/hiclaw-install.ps1')
& ([scriptblock]::Create($s)) uninstall
```

Removes all containers, volumes, networks, and workspace directory.

### Kubernetes

```bash
helm uninstall hiclaw -n hiclaw-system

# Remove CRDs (optional - deletes all Workers/Teams/Humans)
kubectl delete crd workers.hiclaw.higress.io
kubectl delete crd teams.hiclaw.higress.io
kubectl delete crd humans.hiclaw.higress.io

# Remove namespace
kubectl delete namespace hiclaw-system
```

## Resources

- **Official Docs**: https://hiclaw.io
- **GitHub**: https://github.com/agentscope-ai/HiClaw
- **Helm Charts**: https://higress.io/helm-charts
- **Skills Marketplace**: https://skills.sh
- **Discord**: https://discord.com/invite/NVjNA4BAVw
