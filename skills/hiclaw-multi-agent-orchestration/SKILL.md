---
name: hiclaw-multi-agent-orchestration
description: Build and orchestrate collaborative multi-agent teams using HiClaw's Manager-Workers architecture with Matrix rooms, MCP servers, and Kubernetes-native deployment.
triggers:
  - set up a multi-agent collaboration platform with HiClaw
  - create agent teams that work together in Matrix rooms
  - deploy HiClaw manager and worker agents
  - orchestrate multiple AI agents with human-in-the-loop
  - configure MCP servers for agent collaboration
  - build a Kubernetes-native multi-agent system
  - manage agent workers with declarative YAML resources
  - set up secure agent orchestration with Higress gateway
---

# HiClaw Multi-Agent Orchestration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

HiClaw is an open-source collaborative multi-agent runtime platform that enables multiple AI agents to collaborate in controlled, auditable Matrix rooms with full human visibility and intervention. Built on a Manager-Workers architecture, it runs on Docker or Kubernetes and supports OpenClaw, QwenPaw, and Hermes runtimes.

## Installation

### Docker (Local Development)

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

The installer will prompt for:
- LLM provider (OpenAI, Qwen, or OpenAI-compatible)
- API key
- Network mode (local-only or external access)

**Requirements:**
- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- 2 CPU cores + 4 GB RAM minimum (4 cores + 8 GB for multiple workers)

### Kubernetes (Production)

```bash
helm repo add higress.io https://higress.io/helm-charts
helm repo update

# OpenAI / OpenAI-compatible
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=${LLM_API_KEY} \
  --set credentials.adminPassword=${ADMIN_PASSWORD} \
  --set gateway.publicURL=http://localhost:18080

# Non-OpenAI providers (DeepSeek, etc.)
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=${LLM_API_KEY} \
  --set credentials.llmBaseUrl=https://api.deepseek.com/v1 \
  --set credentials.defaultModel=deepseek-chat \
  --set credentials.adminPassword=${ADMIN_PASSWORD} \
  --set gateway.publicURL=http://localhost:18080

# Qwen (通义千问)
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --render-subchart-notes \
  --set credentials.llmApiKey=${QWEN_API_KEY} \
  --set credentials.llmProvider=qwen \
  --set credentials.defaultModel=qwen3.5-plus \
  --set credentials.adminPassword=${ADMIN_PASSWORD} \
  --set gateway.publicURL=http://localhost:18080
```

**Helm Chart Values:**

| Value | Required | Description |
|-------|----------|-------------|
| `credentials.llmApiKey` | yes | API key for LLM provider |
| `gateway.publicURL` | yes | Public URL for Element Web access |
| `credentials.adminPassword` | recommended | Matrix admin password (auto-generated if empty) |
| `credentials.llmProvider` | no | Provider: `openai-compat` (default), `qwen` |
| `credentials.defaultModel` | no | Default model (e.g., `gpt-5.4`, `qwen3.5-plus`) |
| `credentials.llmBaseUrl` | no | OpenAI-compatible base URL |
| `manager.runtime` | no | Manager runtime: `openclaw` (default), `copaw`, `hermes` |
| `worker.defaultRuntime` | no | Worker runtime: `openclaw` (default), `copaw`, `hermes` |

## Core Architecture

HiClaw uses a **Manager-Workers** architecture:

- **Manager**: Central orchestrator that coordinates multiple workers, handles human interaction, and manages task distribution
- **Workers**: Specialized agents that execute specific tasks (OpenClaw for deterministic workflows, Hermes for autonomous coding)
- **Matrix Rooms**: Communication channels where Manager, Workers, and humans collaborate
- **Higress AI Gateway**: Centralized traffic management and credential protection
- **MinIO**: Shared file system for inter-agent data exchange

## Declarative Resource Management

HiClaw uses Kubernetes-style CRDs (Custom Resource Definitions) for managing resources:

### Worker Definition

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: python-dev
spec:
  runtime: hermes  # or openclaw, copaw
  description: "Python development specialist with code execution capabilities"
  
  # Environment variables
  env:
    - name: PYTHON_VERSION
      value: "3.11"
  
  # MCP (Model Context Protocol) servers
  mcp:
    - name: filesystem
      type: stdio
      command: npx
      args:
        - "-y"
        - "@modelcontextprotocol/server-filesystem"
        - "/workspace"
      env:
        - name: NODE_ENV
          value: production
    
    - name: github
      type: sse
      url: http://gateway:8001/mcp/github
      headers:
        - name: Authorization
          value: "Bearer ${GITHUB_TOKEN}"
  
  # Skills from skills.sh registry
  skills:
    - name: python-testing
      version: "1.2.0"
    - name: git-workflow
```

### Team Definition

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: fullstack-dev-team
spec:
  description: "Full-stack development team with frontend, backend, and DevOps specialists"
  
  # Team leader configuration
  leader:
    runtime: copaw  # QwenPaw for intelligent task coordination
    mcp:
      - name: task-coordinator
        type: stdio
        command: node
        args:
          - "/opt/hiclaw/mcp-servers/task-coordinator.js"
  
  # Human coordinators
  humans:
    - name: tech-lead
      matrixId: "@lead:hiclaw.local"
      role: "Technical oversight and code review"
  
  # Worker assignments
  workers:
    - name: frontend-dev
      runtime: hermes
      description: "React and TypeScript specialist"
    
    - name: backend-dev
      runtime: openclaw
      description: "Go API development with structured workflows"
    
    - name: devops-engineer
      runtime: hermes
      description: "Kubernetes and infrastructure automation"
  
  # Shared resources
  sharedMcp:
    - name: jira
      type: sse
      url: http://gateway:8001/mcp/jira
```

### Human Resource

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Human
metadata:
  name: project-manager
spec:
  matrixId: "@pm:hiclaw.local"
  displayName: "Project Manager"
  teams:
    - fullstack-dev-team
    - data-science-team
```

## CLI Usage

HiClaw provides a CLI for resource management:

```bash
# Apply resources from YAML
hiclaw apply -f worker-python.yaml
hiclaw apply -f team-fullstack.yaml

# List resources
hiclaw get workers
hiclaw get teams
hiclaw get humans

# Describe a resource
hiclaw describe worker python-dev
hiclaw describe team fullstack-dev-team

# Delete resources
hiclaw delete worker python-dev
hiclaw delete team fullstack-dev-team

# Check controller status
hiclaw status

# View logs
hiclaw logs manager
hiclaw logs worker python-dev
```

## Configuration Patterns

### Multi-Runtime Collaboration

Use different runtimes for different strengths:

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: ai-development-team
spec:
  leader:
    runtime: copaw  # QwenPaw for flexible task planning
  
  workers:
    - name: code-executor
      runtime: hermes  # Autonomous code execution
      description: "Executes Python/Node.js code autonomously"
    
    - name: workflow-manager
      runtime: openclaw  # Deterministic workflows
      description: "Manages structured development workflows"
    
    - name: research-assistant
      runtime: copaw  # Flexible reasoning
      description: "Research and documentation specialist"
```

### MCP Server Integration

HiClaw supports stdio and SSE MCP servers:

**Stdio MCP (local tools):**
```yaml
mcp:
  - name: filesystem
    type: stdio
    command: npx
    args:
      - "-y"
      - "@modelcontextprotocol/server-filesystem"
      - "/workspace"
  
  - name: postgres
    type: stdio
    command: npx
    args:
      - "-y"
      - "@modelcontextprotocol/server-postgres"
    env:
      - name: DATABASE_URL
        value: "${DATABASE_URL}"
```

**SSE MCP (gateway-proxied external APIs):**
```yaml
mcp:
  - name: github
    type: sse
    url: http://gateway:8001/mcp/github
    headers:
      - name: Authorization
        value: "Bearer ${GITHUB_TOKEN}"
  
  - name: slack
    type: sse
    url: http://gateway:8001/mcp/slack
```

### Nacos Skills Registry

Pull skills from the community registry:

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: fullstack-dev
spec:
  runtime: hermes
  
  skills:
    - name: react-development
      version: "2.1.0"
      registry: https://skills.sh
    
    - name: kubernetes-ops
      version: "1.5.3"
      registry: https://skills.sh
    
    # Custom registry
    - name: internal-tools
      version: "0.9.0"
      registry: http://nacos:8848/nacos
      namespace: production
```

### Environment Variables

Pass custom environment to workers:

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: data-processor
spec:
  runtime: hermes
  
  env:
    - name: SPARK_VERSION
      value: "3.4.0"
    
    - name: AWS_REGION
      value: "us-west-2"
    
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: connection-string
```

## Code Examples

### Go: Create Worker Programmatically

```go
package main

import (
    "context"
    "fmt"
    
    hiclawv1alpha1 "github.com/higress-group/hiclaw/api/v1alpha1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/kubernetes/scheme"
    "sigs.k8s.io/controller-runtime/pkg/client"
    "sigs.k8s.io/controller-runtime/pkg/client/config"
)

func createWorker(ctx context.Context) error {
    // Register HiClaw scheme
    _ = hiclawv1alpha1.AddToScheme(scheme.Scheme)
    
    // Create Kubernetes client
    cfg, err := config.GetConfig()
    if err != nil {
        return fmt.Errorf("failed to get config: %w", err)
    }
    
    k8sClient, err := client.New(cfg, client.Options{Scheme: scheme.Scheme})
    if err != nil {
        return fmt.Errorf("failed to create client: %w", err)
    }
    
    // Define worker
    worker := &hiclawv1alpha1.Worker{
        ObjectMeta: metav1.ObjectMeta{
            Name:      "go-backend-dev",
            Namespace: "hiclaw-system",
        },
        Spec: hiclawv1alpha1.WorkerSpec{
            Runtime:     "hermes",
            Description: "Go backend development specialist",
            Env: []hiclawv1alpha1.EnvVar{
                {
                    Name:  "GO_VERSION",
                    Value: "1.21",
                },
            },
            Mcp: []hiclawv1alpha1.McpServer{
                {
                    Name:    "git",
                    Type:    "stdio",
                    Command: "npx",
                    Args:    []string{"-y", "@modelcontextprotocol/server-git"},
                },
            },
            Skills: []hiclawv1alpha1.Skill{
                {
                    Name:    "go-testing",
                    Version: "1.0.0",
                },
            },
        },
    }
    
    // Create worker
    if err := k8sClient.Create(ctx, worker); err != nil {
        return fmt.Errorf("failed to create worker: %w", err)
    }
    
    fmt.Printf("Worker %s created successfully\n", worker.Name)
    return nil
}
```

### Go: Watch Worker Status

```go
package main

import (
    "context"
    "fmt"
    
    hiclawv1alpha1 "github.com/higress-group/hiclaw/api/v1alpha1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/watch"
    "k8s.io/client-go/kubernetes/scheme"
    "sigs.k8s.io/controller-runtime/pkg/client"
    "sigs.k8s.io/controller-runtime/pkg/client/config"
)

func watchWorkers(ctx context.Context) error {
    _ = hiclawv1alpha1.AddToScheme(scheme.Scheme)
    
    cfg, err := config.GetConfig()
    if err != nil {
        return err
    }
    
    k8sClient, err := client.New(cfg, client.Options{Scheme: scheme.Scheme})
    if err != nil {
        return err
    }
    
    // Create watcher
    workerList := &hiclawv1alpha1.WorkerList{}
    watcher, err := k8sClient.Watch(ctx, workerList, &client.ListOptions{
        Namespace: "hiclaw-system",
    })
    if err != nil {
        return err
    }
    defer watcher.Stop()
    
    // Process events
    for event := range watcher.ResultChan() {
        worker, ok := event.Object.(*hiclawv1alpha1.Worker)
        if !ok {
            continue
        }
        
        switch event.Type {
        case watch.Added:
            fmt.Printf("Worker added: %s (runtime: %s)\n", 
                worker.Name, worker.Spec.Runtime)
        case watch.Modified:
            fmt.Printf("Worker modified: %s (status: %s)\n", 
                worker.Name, worker.Status.Phase)
        case watch.Deleted:
            fmt.Printf("Worker deleted: %s\n", worker.Name)
        }
    }
    
    return nil
}
```

### Go: Create Team with Workers

```go
package main

import (
    "context"
    "fmt"
    
    hiclawv1alpha1 "github.com/higress-group/hiclaw/api/v1alpha1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

func createTeam(ctx context.Context, k8sClient client.Client) error {
    team := &hiclawv1alpha1.Team{
        ObjectMeta: metav1.ObjectMeta{
            Name:      "microservices-team",
            Namespace: "hiclaw-system",
        },
        Spec: hiclawv1alpha1.TeamSpec{
            Description: "Microservices development team",
            Leader: hiclawv1alpha1.TeamLeader{
                Runtime: "copaw",
                Mcp: []hiclawv1alpha1.McpServer{
                    {
                        Name:    "jira",
                        Type:    "sse",
                        Url:     "http://gateway:8001/mcp/jira",
                    },
                },
            },
            Humans: []hiclawv1alpha1.TeamHuman{
                {
                    Name:     "architect",
                    MatrixId: "@architect:hiclaw.local",
                    Role:     "System architecture and design review",
                },
            },
            Workers: []hiclawv1alpha1.TeamWorker{
                {
                    Name:        "backend-service",
                    Runtime:     "hermes",
                    Description: "Go microservice development",
                },
                {
                    Name:        "api-gateway",
                    Runtime:     "openclaw",
                    Description: "API gateway configuration",
                },
            },
            SharedMcp: []hiclawv1alpha1.McpServer{
                {
                    Name:    "kubernetes",
                    Type:    "stdio",
                    Command: "kubectl",
                    Args:    []string{"mcp"},
                },
            },
        },
    }
    
    if err := k8sClient.Create(ctx, team); err != nil {
        return fmt.Errorf("failed to create team: %w", err)
    }
    
    fmt.Printf("Team %s created with %d workers\n", 
        team.Name, len(team.Spec.Workers))
    return nil
}
```

## Common Workflows

### 1. Deploy Complete Development Environment

```bash
# Create worker definitions
cat > workers.yaml <<EOF
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: frontend-dev
spec:
  runtime: hermes
  description: "React/TypeScript frontend specialist"
---
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: backend-dev
spec:
  runtime: hermes
  description: "Go backend API developer"
---
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: qa-tester
spec:
  runtime: openclaw
  description: "Automated testing and QA"
EOF

# Apply workers
hiclaw apply -f workers.yaml

# Create team
cat > team.yaml <<EOF
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: webapp-team
spec:
  description: "Full web application development team"
  leader:
    runtime: copaw
  workers:
    - name: frontend-dev
    - name: backend-dev
    - name: qa-tester
EOF

hiclaw apply -f team.yaml

# Access via Element Web
# Open http://127.0.0.1:18088
# Join #webapp-team room
```

### 2. Add MCP Server to Existing Worker

```bash
# Update worker with new MCP server
cat > worker-update.yaml <<EOF
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: python-dev
spec:
  runtime: hermes
  mcp:
    - name: postgres
      type: stdio
      command: npx
      args:
        - "-y"
        - "@modelcontextprotocol/server-postgres"
      env:
        - name: DATABASE_URL
          value: "${DATABASE_URL}"
    
    - name: github
      type: sse
      url: http://gateway:8001/mcp/github
EOF

hiclaw apply -f worker-update.yaml
```

### 3. Scale Team with Additional Workers

```bash
# Get current team definition
hiclaw get team data-science-team -o yaml > team-current.yaml

# Edit to add workers (manually or programmatically)
# Then apply
hiclaw apply -f team-current.yaml
```

## Troubleshooting

### Worker Not Starting

```bash
# Check worker status
hiclaw describe worker <worker-name>

# Check controller logs
hiclaw logs controller

# Check worker container logs
docker logs hiclaw-worker-<worker-name>  # Docker mode
kubectl logs -n hiclaw-system deploy/<worker-name>  # K8s mode

# Common issues:
# - Invalid runtime specified (must be: openclaw, copaw, hermes)
# - MCP server command not found
# - Missing environment variables
# - Insufficient resources
```

### Matrix Connection Issues

```bash
# Check Matrix server status
docker ps | grep synapse  # Docker mode
kubectl get pods -n hiclaw-system | grep synapse  # K8s mode

# Test Matrix connectivity
curl http://localhost:8008/_matrix/client/versions

# Reset Matrix admin password
docker exec -it hiclaw-synapse \
  register_new_matrix_user \
  -c /data/homeserver.yaml \
  http://localhost:8008
```

### MCP Server Not Working

```bash
# Verify MCP server in worker spec
hiclaw describe worker <worker-name> | grep -A 10 mcp

# Check MCP server logs
# For stdio MCP: check worker container logs
docker logs hiclaw-worker-<worker-name> 2>&1 | grep mcp

# For SSE MCP: check gateway logs
docker logs hiclaw-gateway 2>&1 | grep mcp

# Test MCP server manually
npx -y @modelcontextprotocol/server-filesystem /workspace
```

### Team Leader Not Coordinating

```bash
# Check team status
hiclaw describe team <team-name>

# Verify leader configuration
hiclaw get team <team-name> -o yaml | grep -A 5 leader

# Check leader logs
hiclaw logs team <team-name> --component=leader

# Ensure all workers are ready
hiclaw get workers --selector=team=<team-name>
```

### Upgrade Issues

```bash
# Check current version
docker ps --format '{{.Image}}' | grep hiclaw

# Backup before upgrade
docker exec hiclaw-synapse tar czf /backup/synapse-data.tar.gz /data
docker exec hiclaw-minio tar czf /backup/minio-data.tar.gz /data

# Upgrade with specific version
HICLAW_VERSION=v1.1.2 bash <(curl -sSL https://higress.ai/hiclaw/install.sh)

# Rollback if needed
docker-compose down
docker-compose -f docker-compose.v1.1.1.yaml up -d
```

### Performance Issues

```bash
# Check resource usage
docker stats  # Docker mode
kubectl top pods -n hiclaw-system  # K8s mode

# Increase worker resources (Kubernetes)
kubectl patch worker python-dev -n hiclaw-system --type=merge -p '
spec:
  resources:
    requests:
      memory: "2Gi"
      cpu: "1000m"
    limits:
      memory: "4Gi"
      cpu: "2000m"
'

# Monitor MinIO storage
docker exec hiclaw-minio mc admin info local

# Check gateway metrics
curl http://localhost:18001/metrics
```

## Security Best Practices

1. **Never expose real credentials to workers**:
   - Use Higress gateway for credential management
   - Workers receive only consumer tokens
   - Configure SSE MCP servers at gateway level

2. **Network isolation**:
   ```yaml
   # workers.yaml
   spec:
     networkPolicy:
       enabled: true
       allowedEgress:
         - gateway:8001  # Only allow gateway access
   ```

3. **Audit all agent actions**:
   - All interactions logged in Matrix rooms
   - Human can review and intervene anytime
   - Export audit logs: `hiclaw audit export`

4. **Resource limits**:
   ```yaml
   spec:
     resources:
       limits:
         memory: "2Gi"
         cpu: "1000m"
   ```

5. **Regular updates**:
   ```bash
   # Check for updates
   hiclaw version --check
   
   # Auto-upgrade (preserves data)
   bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
   ```
