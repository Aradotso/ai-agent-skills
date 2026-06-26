---
name: agentteams-hiclaw-multi-agent-orchestration
description: Build and orchestrate collaborative multi-agent teams using HiClaw's Manager-Workers architecture on Matrix with Kubernetes-native control.
triggers:
  - set up HiClaw multi-agent collaboration platform
  - create agent teams with HiClaw manager and workers
  - deploy collaborative AI agents using HiClaw
  - orchestrate multiple agents with HiClaw on Kubernetes
  - configure HiClaw workers with custom skills
  - build human-in-the-loop agent workflows with HiClaw
  - manage agent collaboration rooms in HiClaw
  - integrate MCP servers with HiClaw agents
---

# AgentTeams HiClaw Multi-Agent Orchestration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

HiClaw is an open-source collaborative multi-agent runtime platform that enables multiple AI agents to work together in Matrix-based rooms with full human visibility and control. Built on a Manager-Workers architecture, it orchestrates Agent containers (Manager + Workers) for enterprise multi-agent collaboration scenarios.

**Key capabilities:**
- Manager-Workers architecture for hierarchical agent coordination
- Multi-runtime support (OpenClaw, QwenPaw, Hermes)
- Matrix protocol for transparent human-in-the-loop collaboration
- Kubernetes-native declarative resource management
- Shared MinIO file system for efficient inter-agent data exchange
- Higress AI Gateway for secure credential management
- Skills marketplace integration (skills.sh)

## Installation

### Docker Desktop / Docker Engine

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

**Requirements:** 2 CPU cores + 4 GB RAM minimum (4 cores + 8 GB for multiple Workers)

The installer prompts for:
1. LLM provider selection
2. API key (stored securely in environment)
3. Network mode (local-only or external access)

**Access:** Navigate to `http://127.0.0.1:18088` for Element Web client.

### Kubernetes (Helm)

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
  --set credentials.llmApiKey=$OPENAI_API_KEY \
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

**Custom runtime configuration:**
```bash
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --set manager.runtime=copaw \
  --set worker.defaultRuntime=hermes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

## Core Concepts

### Manager-Workers Architecture

- **Manager**: Central orchestrator that coordinates Workers, delegates tasks, manages rooms
- **Workers**: Specialized agents that execute specific tasks
- **Team**: Declarative group of Workers coordinated by Manager
- **Human**: Human participant with full visibility and intervention rights

### Runtime Options

1. **OpenClaw**: Original Lobster-based agent runtime
2. **QwenPaw** (formerly CoPaw): Optimized runtime with 80% less memory
3. **Hermes**: Autonomous coding agent runtime

## Declarative Resource Management

HiClaw uses Kubernetes-style CRDs (Custom Resource Definitions) for declarative agent management.

### Worker Definition

**worker-example.yaml:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: code-reviewer
  namespace: hiclaw-system
spec:
  runtime: hermes
  model: gpt-4o
  description: "Expert code reviewer with focus on Go and Python best practices"
  
  # Custom environment variables
  env:
    - name: HERMES_MAX_ITERATIONS
      value: "10"
    - name: LOG_LEVEL
      value: "debug"
  
  # MCP (Model Context Protocol) servers
  mcp:
    - name: github
      type: nacos
      config:
        serviceName: mcp-github
        namespace: default
    - name: filesystem
      type: stdio
      command: npx
      args:
        - -y
        - "@modelcontextprotocol/server-filesystem"
        - /workspace
  
  # Skills from registry
  skills:
    - source: nacos
      serviceName: skill-code-review
      namespace: default
    - source: nacos
      serviceName: skill-security-scan
      namespace: default
```

**Apply Worker:**
```bash
kubectl apply -f worker-example.yaml
```

### Team Definition

**team-example.yaml:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: backend-dev-team
  namespace: hiclaw-system
spec:
  description: "Backend development team for microservices"
  
  # Team leader configuration
  leader:
    runtime: copaw
    model: qwen3.5-plus
    mcp:
      - name: jira
        type: nacos
        config:
          serviceName: mcp-jira
          namespace: tools
  
  # Team members
  workers:
    - name: go-developer
      runtime: hermes
      model: gpt-4o
      skills:
        - source: nacos
          serviceName: skill-golang-expert
    - name: db-specialist
      runtime: openclaw
      model: claude-3.5-sonnet
      skills:
        - source: nacos
          serviceName: skill-postgres-admin
  
  # Human coordinators
  humans:
    - matrix_id: "@alice:matrix.example.com"
      role: tech-lead
    - matrix_id: "@bob:matrix.example.com"
      role: reviewer
```

**Apply Team:**
```bash
kubectl apply -f team-example.yaml
```

### Human Definition

**human-example.yaml:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Human
metadata:
  name: senior-engineer
  namespace: hiclaw-system
spec:
  matrix_id: "@engineer:matrix.example.com"
  display_name: "Senior Engineer"
  teams:
    - backend-dev-team
    - frontend-dev-team
  permissions:
    - interrupt_workflows
    - approve_deployments
    - access_credentials
```

## CLI Commands (hiclaw)

### Worker Management

**List Workers:**
```bash
kubectl get workers -n hiclaw-system
```

**Get Worker details:**
```bash
kubectl describe worker code-reviewer -n hiclaw-system
```

**Delete Worker:**
```bash
kubectl delete worker code-reviewer -n hiclaw-system
```

### Team Management

**List Teams:**
```bash
kubectl get teams -n hiclaw-system
```

**Scale Team (add Worker):**
```yaml
# Edit team-example.yaml to add new worker
spec:
  workers:
    - name: go-developer
    - name: db-specialist
    - name: api-tester  # New worker
      runtime: hermes
      model: gpt-4o
```

```bash
kubectl apply -f team-example.yaml
```

### Logs and Debugging

**View Manager logs:**
```bash
kubectl logs -n hiclaw-system deployment/hiclaw-manager -f
```

**View Worker logs:**
```bash
# Get worker pod name
kubectl get pods -n hiclaw-system -l worker=code-reviewer

# Stream logs
kubectl logs -n hiclaw-system <pod-name> -f
```

**View controller logs:**
```bash
kubectl logs -n hiclaw-system deployment/hiclaw-controller -f
```

## MCP Server Integration

HiClaw supports two types of MCP servers:

### 1. Nacos-based MCP (Recommended)

**Register MCP server in Nacos:**
```bash
# MCP server configuration stored in Nacos
# Access via Higress AI Gateway STS scope: sts-hiclaw or ai-registry
```

**Use in Worker/Manager:**
```yaml
mcp:
  - name: github
    type: nacos
    config:
      serviceName: mcp-github
      namespace: default
      group: DEFAULT_GROUP  # Optional
```

**Benefits:**
- Centralized credential management
- Workers use consumer tokens only
- Zero credential exposure to agents
- Dynamic service discovery

### 2. Stdio MCP

**Direct command execution:**
```yaml
mcp:
  - name: filesystem
    type: stdio
    command: npx
    args:
      - -y
      - "@modelcontextprotocol/server-filesystem"
      - /workspace
  
  - name: git
    type: stdio
    command: python
    args:
      - -m
      - mcp_server_git
      - --repository
      - /workspace/repo
```

## Skills Integration

### Using Nacos Skills Registry

**Worker with remote skills:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: devops-engineer
spec:
  runtime: hermes
  skills:
    - source: nacos
      serviceName: skill-docker-compose
      namespace: devops
    - source: nacos
      serviceName: skill-kubernetes-deploy
      namespace: devops
```

### Custom Skills Package

**Directory structure:**
```
my-worker-package/
├── SOUL.md          # Optional: Worker personality/behavior
├── skill-1.sh
├── skill-2.py
└── README.md
```

**Deploy custom Worker with skills:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: custom-worker
spec:
  runtime: openclaw
  # Mount custom skills via ConfigMap or PVC
  volumeMounts:
    - name: custom-skills
      mountPath: /skills
  volumes:
    - name: custom-skills
      configMap:
        name: custom-worker-skills
```

## Configuration Patterns

### Multi-Model Strategy

**Use different models for different Workers:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: cost-optimized-team
spec:
  leader:
    runtime: copaw
    model: gpt-4o-mini  # Cheap model for orchestration
  
  workers:
    - name: code-generator
      runtime: hermes
      model: claude-3.5-sonnet  # Premium model for complex coding
    
    - name: code-reviewer
      runtime: openclaw
      model: gpt-4o-mini  # Cheap model for review
    
    - name: test-writer
      runtime: hermes
      model: deepseek-coder  # Specialized model for testing
```

### Environment-Specific Configuration

**Development Team:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: dev-team
  namespace: hiclaw-dev
spec:
  leader:
    env:
      - name: LOG_LEVEL
        value: debug
      - name: MAX_ITERATIONS
        value: "20"
  workers:
    - name: dev-worker
      env:
        - name: ENVIRONMENT
          value: development
        - name: ENABLE_EXPERIMENTAL
          value: "true"
```

**Production Team:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: prod-team
  namespace: hiclaw-prod
spec:
  leader:
    env:
      - name: LOG_LEVEL
        value: info
      - name: MAX_ITERATIONS
        value: "5"
  workers:
    - name: prod-worker
      env:
        - name: ENVIRONMENT
          value: production
        - name: ENABLE_EXPERIMENTAL
          value: "false"
```

### Shared File System Usage

**Workers automatically share files via MinIO:**
```yaml
# Worker A generates report
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: data-analyst
spec:
  runtime: hermes
  description: "Generate analysis reports and save to /workspace/reports/"

---
# Worker B processes the report
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: report-formatter
spec:
  runtime: hermes
  description: "Read reports from /workspace/reports/ and format as PDF"
```

**In Matrix room:**
```
@manager: data-analyst, generate quarterly sales report
data-analyst: Report saved to /workspace/reports/q4-2026.json
@manager: report-formatter, convert q4-2026.json to PDF
report-formatter: PDF saved to /workspace/reports/q4-2026.pdf
```

## Troubleshooting

### Worker Not Responding

**Check Worker status:**
```bash
kubectl get worker my-worker -n hiclaw-system -o yaml
```

**Look for status conditions:**
```yaml
status:
  conditions:
    - type: Ready
      status: "False"
      reason: ImagePullError
      message: "Failed to pull image"
```

**Common fixes:**
- Verify runtime image is available
- Check resource limits (CPU/memory)
- Inspect logs: `kubectl logs -n hiclaw-system <worker-pod>`

### MCP Server Connection Failed

**Verify Nacos registration:**
```bash
# Check if MCP service is registered
curl http://nacos-server:8848/nacos/v1/ns/instance/list?serviceName=mcp-github
```

**Test MCP connectivity:**
```bash
# From Worker pod
kubectl exec -it <worker-pod> -n hiclaw-system -- \
  curl http://mcp-github.default.svc.cluster.local:8080/health
```

**Check credentials:**
- Ensure STS scope `sts-hiclaw` or `ai-registry` is configured in Higress
- Verify consumer tokens have access to MCP services
- Review gateway logs: `kubectl logs -n higress-system deployment/higress-gateway`

### Manager Cannot Create Room

**Check Matrix server status:**
```bash
kubectl get pods -n hiclaw-system -l app=synapse
```

**Verify Matrix credentials:**
```bash
kubectl get secret -n hiclaw-system hiclaw-credentials -o yaml
```

**Test Matrix API:**
```bash
# Get admin token from secret
ADMIN_TOKEN=$(kubectl get secret -n hiclaw-system hiclaw-credentials \
  -o jsonpath='{.data.matrix-admin-token}' | base64 -d)

# Test API
curl -H "Authorization: Bearer $ADMIN_TOKEN" \
  http://matrix.hiclaw-system.svc.cluster.local:8008/_matrix/client/v3/account/whoami
```

### High Token Consumption

**Enable MinIO file sharing:**
- Ensure Workers save large outputs to `/workspace/` instead of Matrix messages
- Configure Workers to reference file paths in messages

**Optimize model selection:**
```yaml
# Use cheaper models for simple tasks
spec:
  workers:
    - name: log-parser
      model: gpt-4o-mini  # Instead of gpt-4o
```

**Limit context:**
```yaml
spec:
  workers:
    - name: focused-worker
      env:
        - name: MAX_CONTEXT_MESSAGES
          value: "10"  # Reduce history size
```

### Team Leader Not Coordinating

**Verify Team CR status:**
```bash
kubectl get team backend-dev-team -n hiclaw-system -o yaml
```

**Check leader logs:**
```bash
kubectl logs -n hiclaw-system deployment/hiclaw-manager -f | grep backend-dev-team
```

**Common issues:**
- Leader MCP servers not accessible
- Workers not in Ready state
- Insufficient permissions for humans in team

### Upgrade Issues

**Backup before upgrade:**
```bash
# Export all resources
kubectl get workers,teams,humans -n hiclaw-system -o yaml > backup.yaml

# Backup Matrix data
kubectl exec -n hiclaw-system synapse-0 -- tar czf /tmp/matrix-backup.tar.gz /data
kubectl cp hiclaw-system/synapse-0:/tmp/matrix-backup.tar.gz ./matrix-backup.tar.gz
```

**Upgrade Helm chart:**
```bash
helm upgrade hiclaw higress.io/hiclaw \
  -n hiclaw-system \
  --reuse-values \
  --render-subchart-notes
```

**Restore if needed:**
```bash
kubectl apply -f backup.yaml
```

## Best Practices

1. **Start with simple Workers** — Test with one Worker before creating complex Teams
2. **Use Nacos MCP** for production — Avoid stdio MCP for security-sensitive operations
3. **Leverage file sharing** — Save large outputs to `/workspace/` to reduce token costs
4. **Monitor resource usage** — Set appropriate CPU/memory limits for Workers
5. **Human oversight** — Always include humans in Teams for critical workflows
6. **Version control CRDs** — Store Worker/Team YAMLs in Git for reproducibility
7. **Test locally first** — Use Docker installation for development, Kubernetes for production

## Example: Complete Development Workflow

**1. Define Team:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: microservice-team
  namespace: hiclaw-system
spec:
  description: "Build and deploy microservices"
  
  leader:
    runtime: copaw
    model: gpt-4o
    mcp:
      - name: jira
        type: nacos
        config:
          serviceName: mcp-jira
  
  workers:
    - name: backend-dev
      runtime: hermes
      model: claude-3.5-sonnet
      mcp:
        - name: github
          type: nacos
          config:
            serviceName: mcp-github
      skills:
        - source: nacos
          serviceName: skill-go-microservice
    
    - name: devops
      runtime: hermes
      model: gpt-4o
      mcp:
        - name: kubernetes
          type: nacos
          config:
            serviceName: mcp-k8s
      skills:
        - source: nacos
          serviceName: skill-helm-deploy
  
  humans:
    - matrix_id: "@tech-lead:matrix.example.com"
      role: lead
```

**2. Apply and verify:**
```bash
kubectl apply -f microservice-team.yaml
kubectl get team microservice-team -n hiclaw-system
```

**3. Interact in Matrix:**
```
@manager: Create a new Go microservice for user authentication

manager: I'll coordinate backend-dev and devops for this task.
         backend-dev, please create the authentication service.

backend-dev: Created auth-service with:
             - JWT token handling
             - PostgreSQL user store
             - gRPC API
             Code pushed to /workspace/auth-service/

manager: devops, please containerize and prepare for deployment.

devops: Created:
        - Dockerfile saved to /workspace/auth-service/Dockerfile
        - Helm chart in /workspace/auth-service/chart/
        Ready for deployment.

@tech-lead: Looks good! Deploy to staging.

manager: devops, deploy auth-service to staging environment.

devops: Deployed successfully to staging.
        Service URL: https://auth-staging.example.com
```

This workflow demonstrates:
- Manager delegation to specialized Workers
- Shared file system for code artifacts
- MCP integration (GitHub, Kubernetes)
- Human oversight and approval
- End-to-end microservice delivery
