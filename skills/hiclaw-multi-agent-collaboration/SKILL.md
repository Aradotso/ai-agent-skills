---
name: hiclaw-multi-agent-collaboration
description: Build and orchestrate collaborative AI agent teams using HiClaw's Manager-Workers architecture with Matrix rooms, MCP servers, and Kubernetes-native control plane
triggers:
  - set up a multi-agent collaboration environment with HiClaw
  - create AI agent teams that work together in Matrix rooms
  - deploy HiClaw workers and managers on Kubernetes
  - configure MCP servers for HiClaw agents
  - orchestrate multiple AI agents with human oversight
  - build a collaborative agent system with HiClaw
  - integrate OpenClaw and QwenPaw workers in HiClaw
  - manage AI agent teams with Matrix protocol
---

# HiClaw Multi-Agent Collaboration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

HiClaw is an open-source collaborative multi-agent runtime platform built on a Manager-Workers architecture. It enables multiple AI agents to collaborate in controlled, auditable Matrix rooms with full human visibility and intervention capabilities. HiClaw orchestrates agent containers (OpenClaw, QwenPaw, Hermes) rather than implementing agent logic itself, focusing on enterprise-grade multi-agent coordination.

## Key Concepts

- **Manager-Workers Architecture**: Central Manager orchestrates multiple Worker agents
- **Matrix Protocol**: Uses Element IM client + Tuwunel server for agent communication
- **Kubernetes-Native**: CRDs for Worker, Manager, Team, and Human resources
- **MinIO File System**: Shared storage for inter-agent information exchange
- **Higress AI Gateway**: Centralized traffic management with credential isolation
- **MCP Servers**: Model Context Protocol integration for external tools/data
- **Multiple Runtimes**: OpenClaw (deterministic), QwenPaw (Alibaba Cloud), Hermes (autonomous coding)

## Installation

### Docker Desktop Quick Start

**macOS / Linux:**
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

The installer prompts for:
1. LLM provider (OpenAI-compatible APIs supported)
2. API key
3. Network mode (local-only or external access)

Access Element Web at: `http://127.0.0.1:18088`

### Kubernetes Helm Installation

**Prerequisites**: Kubernetes 1.24+, Helm 3.7+, default StorageClass

**OpenAI / OpenAI-Compatible:**
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

**Qwen (通义千问):**
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

**Custom Provider:**
```bash
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.llmBaseUrl=https://api.deepseek.com/v1 \
  --set credentials.defaultModel=deepseek-chat \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

**Alternative Runtimes (QwenPaw Manager + Hermes Workers):**
```bash
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace --devel \
  --set manager.runtime=copaw \
  --set worker.defaultRuntime=hermes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.llmBaseUrl=$LLM_BASE_URL \
  --set credentials.defaultModel=$DEFAULT_MODEL \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

## Kubernetes CRD Resources

### Worker Resource

Workers are individual AI agents with specific skills and runtimes.

**Example: OpenClaw Worker**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: python-dev
  namespace: hiclaw-system
spec:
  runtime: openclaw
  description: "Python development specialist"
  env:
    - name: CUSTOM_VAR
      value: "custom-value"
  mcp:
    servers:
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
```

**Example: QwenPaw Worker**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: data-analyst
  namespace: hiclaw-system
spec:
  runtime: copaw
  description: "Data analysis and visualization expert"
  mcp:
    servers:
      - name: postgres
        type: stdio
        command: npx
        args:
          - "-y"
          - "@modelcontextprotocol/server-postgres"
        env:
          - name: DATABASE_URL
            valueFrom:
              secretKeyRef:
                name: db-credentials
                key: url
```

**Example: Hermes Worker (Autonomous Coding)**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: hermes-coder
  namespace: hiclaw-system
spec:
  runtime: hermes
  description: "Autonomous coding agent for complex implementations"
  env:
    - name: MAX_ITERATIONS
      value: "10"
```

### Manager Resource

The Manager orchestrates Workers and handles task delegation.

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Manager
metadata:
  name: team-coordinator
  namespace: hiclaw-system
spec:
  runtime: openclaw
  description: "Coordinates multi-agent tasks"
  mcp:
    servers:
      - name: github
        type: stdio
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

### Team Resource

Teams group Workers and assign them to collaborative tasks.

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: web-dev-team
  namespace: hiclaw-system
spec:
  description: "Full-stack web development team"
  workers:
    - python-dev
    - data-analyst
    - hermes-coder
  leader:
    runtime: copaw
    mcp:
      servers:
        - name: git
          type: stdio
          command: npx
          args:
            - "-y"
            - "@modelcontextprotocol/server-git"
          env:
            - name: GIT_REPO_PATH
              value: "/workspace/project"
```

### Human Resource

Register human users for collaboration.

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Human
metadata:
  name: developer-alice
  namespace: hiclaw-system
spec:
  matrixUserId: "@alice:matrix.example.com"
  displayName: "Alice (Senior Developer)"
  teams:
    - web-dev-team
```

## CLI Commands (hiclaw)

The `hiclaw` CLI replaces shell scripts for resource management.

**Create Worker:**
```bash
hiclaw worker create \
  --name frontend-expert \
  --runtime openclaw \
  --description "React and TypeScript specialist"
```

**Create Worker with MCP Server:**
```bash
hiclaw worker create \
  --name db-worker \
  --runtime copaw \
  --mcp-server postgres \
  --mcp-command "npx -y @modelcontextprotocol/server-postgres" \
  --env DATABASE_URL=$DATABASE_URL
```

**List Workers:**
```bash
hiclaw worker list
```

**Delete Worker:**
```bash
hiclaw worker delete frontend-expert
```

**Create Team:**
```bash
hiclaw team create \
  --name backend-team \
  --workers "python-dev,db-worker" \
  --description "Backend development team"
```

**Add Worker to Team:**
```bash
hiclaw team add-worker backend-team hermes-coder
```

**Create Human User:**
```bash
hiclaw human create \
  --name bob \
  --matrix-id "@bob:matrix.example.com" \
  --display-name "Bob (Product Manager)"
```

## MCP Server Configuration

MCP (Model Context Protocol) servers provide external tools and data access.

### Filesystem Server

```yaml
mcp:
  servers:
    - name: filesystem
      type: stdio
      command: npx
      args:
        - "-y"
        - "@modelcontextprotocol/server-filesystem"
        - "/workspace"
```

### GitHub Server

```yaml
mcp:
  servers:
    - name: github
      type: stdio
      command: npx
      args:
        - "-y"
        - "@modelcontextprotocol/server-github"
      env:
        - name: GITHUB_PERSONAL_ACCESS_TOKEN
          valueFrom:
            secretKeyRef:
              name: github-creds
              key: token
```

### Postgres Server

```yaml
mcp:
  servers:
    - name: postgres
      type: stdio
      command: npx
      args:
        - "-y"
        - "@modelcontextprotocol/server-postgres"
      env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-creds
              key: url
```

### Custom MCP Server

```yaml
mcp:
  servers:
    - name: custom-api
      type: stdio
      command: python
      args:
        - "/opt/mcp-servers/custom_api.py"
      env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: api-creds
              key: key
        - name: API_BASE_URL
          value: "https://api.example.com"
```

## Nacos Skills Registry

HiClaw supports remote skill loading from Nacos registry.

**Register Skill:**
```bash
# Nacos API endpoint
curl -X POST "http://nacos-server:8848/nacos/v1/cs/configs" \
  -d "dataId=my-skill.py" \
  -d "group=ai-registry" \
  -d "content=$(cat my_skill.py)"
```

**Worker with Nacos Skills:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: nacos-worker
spec:
  runtime: openclaw
  env:
    - name: NACOS_SERVER
      value: "http://nacos-server:8848"
    - name: NACOS_NAMESPACE
      value: "ai-registry"
    - name: SKILL_IDS
      value: "my-skill.py,another-skill.py"
```

## Runtime-Specific Patterns

### OpenClaw Workers

Best for deterministic, rule-based tasks with explicit tool calling.

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: openclaw-analyzer
spec:
  runtime: openclaw
  description: "Log analysis and pattern detection"
  mcp:
    servers:
      - name: filesystem
        type: stdio
        command: npx
        args: ["-y", "@modelcontextprotocol/server-filesystem", "/logs"]
```

**Use cases**: File processing, API integration, structured data analysis

### QwenPaw Workers

Best for Chinese language tasks and Alibaba Cloud ecosystem.

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: qwen-translator
spec:
  runtime: copaw
  description: "中英文翻译专家"
  env:
    - name: QWEN_MODEL
      value: "qwen3.5-plus"
```

**Use cases**: Translation, Chinese NLP, Alibaba Cloud service integration

### Hermes Workers

Best for autonomous coding tasks requiring multiple iterations.

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: hermes-dev
spec:
  runtime: hermes
  description: "Autonomous full-stack developer"
  env:
    - name: MAX_ITERATIONS
      value: "15"
    - name: CODE_REVIEW_ENABLED
      value: "true"
  mcp:
    servers:
      - name: git
        type: stdio
        command: npx
        args: ["-y", "@modelcontextprotocol/server-git"]
```

**Use cases**: Feature implementation, bug fixing, code refactoring

## Configuration Patterns

### Environment Variables for Workers

```yaml
spec:
  env:
    # Simple key-value
    - name: LOG_LEVEL
      value: "debug"
    
    # From Secret
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: api-credentials
          key: key
    
    # From ConfigMap
    - name: CONFIG_FILE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: config.json
```

### Token Plans (LLM Quota Management)

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: budget-worker
spec:
  runtime: openclaw
  tokenPlan:
    name: basic-plan
    maxTokens: 100000
    resetPeriod: "daily"
```

### Worker Template Marketplace

**Install from Template:**
```bash
# Browse available templates
hiclaw template list

# Install template
hiclaw worker create-from-template \
  --template github-pr-reviewer \
  --name my-pr-reviewer
```

**Custom Template:**
```yaml
# templates/my-template.yaml
apiVersion: hiclaw.io/v1alpha1
kind: WorkerTemplate
metadata:
  name: data-pipeline-worker
spec:
  runtime: copaw
  description: "ETL data pipeline specialist"
  mcp:
    servers:
      - name: postgres
        type: stdio
        command: npx
        args: ["-y", "@modelcontextprotocol/server-postgres"]
  requiredSecrets:
    - DATABASE_URL
```

## Multi-Agent Collaboration Example

**Scenario**: Build a web application with human oversight

**1. Create Workers:**
```yaml
---
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: frontend-dev
spec:
  runtime: openclaw
  description: "React frontend specialist"
---
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: backend-dev
spec:
  runtime: hermes
  description: "Python FastAPI backend developer"
  mcp:
    servers:
      - name: postgres
        type: stdio
        command: npx
        args: ["-y", "@modelcontextprotocol/server-postgres"]
---
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: devops
spec:
  runtime: openclaw
  description: "Docker and Kubernetes deployment expert"
  mcp:
    servers:
      - name: kubernetes
        type: stdio
        command: kubectl
        args: ["proxy"]
```

**2. Create Team:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: webapp-team
spec:
  description: "Full-stack web application team"
  workers:
    - frontend-dev
    - backend-dev
    - devops
  leader:
    runtime: copaw
    description: "Team coordinator and task planner"
    mcp:
      servers:
        - name: git
          type: stdio
          command: npx
          args: ["-y", "@modelcontextprotocol/server-git"]
```

**3. Add Human Coordinator:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Human
metadata:
  name: tech-lead
spec:
  matrixUserId: "@lead:matrix.local"
  displayName: "Tech Lead"
  teams:
    - webapp-team
```

**4. Interact via Matrix:**
Open Element Web, join the `webapp-team` room, and send:
```
@team-leader: Create a new e-commerce website with user authentication, 
product catalog, and payment integration. Use React for frontend, 
FastAPI for backend, and deploy on Kubernetes.
```

The Team Leader will delegate tasks to Workers, who collaborate in the room while you monitor and intervene as needed.

## Upgrade

**Docker:**
```bash
# Latest version
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)

# Specific version
HICLAW_VERSION=v1.1.2 bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

**Kubernetes:**
```bash
helm repo update
helm upgrade hiclaw higress.io/hiclaw \
  -n hiclaw-system \
  --reuse-values
```

## Troubleshooting

### Workers Not Appearing in Matrix Room

**Check Worker status:**
```bash
kubectl get workers -n hiclaw-system
kubectl describe worker <worker-name> -n hiclaw-system
```

**Check controller logs:**
```bash
kubectl logs -n hiclaw-system deployment/hiclaw-controller -f
```

**Common cause**: MCP server command not available in container. Verify `npx` or custom binary is in PATH.

### MCP Server Connection Failures

**Check environment variables:**
```bash
kubectl get secret <secret-name> -n hiclaw-system -o yaml
```

**Test MCP server manually:**
```bash
kubectl exec -it deployment/<worker-name> -n hiclaw-system -- \
  npx -y @modelcontextprotocol/server-filesystem /workspace
```

**Fix**: Ensure secrets exist and valueFrom references are correct.

### Manager Not Delegating Tasks

**Check Manager logs:**
```bash
kubectl logs -n hiclaw-system deployment/hiclaw-manager -f
```

**Verify Team configuration:**
```bash
kubectl get team <team-name> -n hiclaw-system -o yaml
```

**Common cause**: Workers not listed in Team spec or Worker names misspelled.

### High Token Consumption

**Check MinIO file sharing:**
```bash
kubectl port-forward -n hiclaw-system svc/minio 9000:9000
# Access http://localhost:9000 (credentials in install log)
```

**Verify workers use file system instead of inline content:**
```yaml
mcp:
  servers:
    - name: filesystem
      type: stdio
      command: npx
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
```

**Fix**: Configure shared MinIO buckets for large data exchange.

### Element Web 404 Error

**Docker Desktop:**
```bash
docker ps | grep element
# Should show element-web container running
```

**Kubernetes:**
```bash
kubectl get pods -n hiclaw-system | grep element
kubectl port-forward -n hiclaw-system svc/element-web 18088:80
```

**Access**: `http://localhost:18088`

### Credential Exposure Concerns

**Verify gateway isolation:**
```bash
kubectl get configmap -n hiclaw-system higress-config -o yaml
# Check that credentials are in gateway config, not Worker env
```

**Best practice**: Always use `valueFrom.secretKeyRef` in MCP env, never plain `value` for secrets.

### Namespace-Scoped RBAC Issues

**Check controller ServiceAccount:**
```bash
kubectl get sa -n hiclaw-system hiclaw-controller
kubectl get rolebinding -n hiclaw-system
```

**Fix**: Ensure RoleBinding exists for `hiclaw-controller` SA in `hiclaw-system` namespace.

## Advanced Patterns

### Cross-Namespace Workers (Not Recommended)

HiClaw uses namespace-scoped RBAC (v1.1.1+). For multi-tenant setups, install separate HiClaw instances per namespace instead of cross-namespace Workers.

### Custom Runtime Images

**Build custom Worker image:**
```dockerfile
FROM hiclaw/worker-openclaw:v1.1.2
RUN pip install custom-library
COPY custom_skill.py /app/skills/
```

**Deploy:**
```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: custom-worker
spec:
  runtime: openclaw
  image: myregistry.io/custom-worker:latest
```

### Graceful Shutdown

**Controller metrics:**
```bash
kubectl port-forward -n hiclaw-system svc/hiclaw-controller-metrics 8080:8080
curl http://localhost:8080/metrics | grep hiclaw_reconcile
```

**Graceful termination**: Controller handles SIGTERM and drains in-flight reconciles before exit (v1.1.2+).

## Key Files and Directories

**Docker Installation:**
- Config: `~/.hiclaw/hiclaw.env` (or `%USERPROFILE%\.hiclaw\hiclaw.env` on Windows)
- Workspace: `~/.hiclaw/workspace/`
- Install log: `~/.hiclaw/install.log`

**Kubernetes:**
- Helm values: `helm get values hiclaw -n hiclaw-system`
- Controller config: ConfigMap `hiclaw-controller-config`
- Credentials: Secret `hiclaw-credentials`
- MinIO data: PVC `minio-data`
- Matrix data: PVC `synapse-data`

## Resources

- **Documentation**: https://hiclaw.io
- **GitHub**: https://github.com/agentscope-ai/HiClaw
- **Helm Charts**: https://higress.io/helm-charts
- **Skills Registry**: https://skills.sh
- **Discord**: https://discord.com/invite/NVjNA4BAVw
- **Matrix Protocol**: https://matrix.org
