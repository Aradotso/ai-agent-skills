---
name: hiclaw-multi-agent-collaboration
description: Build and orchestrate collaborative multi-agent systems with HiClaw's Manager-Workers architecture, Matrix rooms, and human-in-the-loop workflows
triggers:
  - set up multi-agent collaboration with HiClaw
  - create HiClaw agents and teams
  - deploy HiClaw on Kubernetes
  - configure HiClaw workers and managers
  - integrate Matrix chat with AI agents
  - build collaborative agent workflows with HiClaw
  - manage HiClaw agent teams and skills
  - orchestrate multi-agent tasks using HiClaw
---

# HiClaw Multi-Agent Collaboration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

HiClaw is an open-source collaborative multi-agent runtime platform built on a Manager-Workers architecture. It enables multiple AI agents to collaborate in controlled Matrix rooms with full human visibility and intervention capabilities. HiClaw orchestrates Agent containers (Manager + Workers) for enterprise-grade, auditable multi-agent coordination.

## Core Concepts

- **Manager Agent**: Central orchestrator that coordinates Workers and manages task delegation
- **Worker Agents**: Specialized agents with specific skills, orchestrated by the Manager
- **Matrix Rooms**: Communication channels where humans, Manager, and Workers collaborate
- **OpenClaw Runtime**: Python-based agent runtime (default)
- **QwenPaw Runtime**: Lightweight alternative runtime (formerly CoPaw)
- **Hermes Runtime**: Autonomous code execution runtime
- **Shared File System**: MinIO-backed storage for inter-agent information exchange
- **Higress AI Gateway**: Centralized traffic management with credential isolation

## Installation

### Docker Installation (Quick Start)

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
1. LLM provider (OpenAI-compatible APIs supported)
2. API key (stored securely)
3. Network mode (local-only or external access)

**Access the UI:**
- Open http://127.0.0.1:18088 in your browser
- Log in to Element Web (Matrix client)
- The Manager will greet you automatically

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

**Install with custom OpenAI-compatible provider:**

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

**Use alternative runtimes:**

```bash
helm install hiclaw higress.io/hiclaw \
  -n hiclaw-system --create-namespace \
  --set manager.runtime=copaw \
  --set worker.defaultRuntime=hermes \
  --set credentials.llmApiKey=$LLM_API_KEY \
  --set credentials.adminPassword=$ADMIN_PASSWORD \
  --set gateway.publicURL=http://localhost:18080
```

## Configuration

### Helm Chart Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `credentials.llmApiKey` | LLM provider API key | (required) |
| `credentials.llmProvider` | Provider name | `openai-compat` |
| `credentials.defaultModel` | Default model | `gpt-5.4` |
| `credentials.llmBaseUrl` | OpenAI-compatible base URL | (empty for official OpenAI) |
| `credentials.adminPassword` | Matrix admin password | auto-generated |
| `gateway.publicURL` | Public URL for Element Web | (required) |
| `manager.runtime` | Manager runtime | `openclaw` |
| `worker.defaultRuntime` | Default Worker runtime | `openclaw` |

### Docker Environment Variables

The Docker installation creates a `.env` file with:

```bash
LLM_API_KEY=your-api-key
LLM_PROVIDER=openai-compat
DEFAULT_MODEL=gpt-5.4
ADMIN_PASSWORD=your-password
NETWORK_MODE=local
```

## Kubernetes-Native Resource Management

HiClaw 1.0.9+ supports declarative YAML-based resource management for Workers, Teams, and Humans.

### Worker Custom Resource

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: python-dev
  namespace: hiclaw-system
spec:
  runtime: openclaw  # or copaw, hermes
  description: "Python development specialist"
  skills:
    - name: github-search
      type: mcp
      config:
        serverName: github
        toolName: search_repositories
    - name: file-operations
      type: builtin
  env:
    - name: CUSTOM_VAR
      value: "custom-value"
```

**Apply Worker:**

```bash
kubectl apply -f worker-python-dev.yaml
```

### Team Custom Resource

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: backend-team
  namespace: hiclaw-system
spec:
  description: "Backend development team"
  leader:
    runtime: copaw
    model: gpt-4o
    mcpServers:
      - name: github
        enabled: true
      - name: filesystem
        enabled: true
  workers:
    - name: python-dev
    - name: database-expert
  humans:
    - name: alice
      role: reviewer
```

**Create Team:**

```bash
kubectl apply -f team-backend.yaml
```

### Human Custom Resource

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Human
metadata:
  name: alice
  namespace: hiclaw-system
spec:
  matrixUserId: "@alice:hiclaw.local"
  role: developer
  teams:
    - backend-team
    - frontend-team
```

**Register Human:**

```bash
kubectl apply -f human-alice.yaml
```

## CLI Commands

HiClaw provides a CLI for resource management (replaces shell scripts in v1.1.0+).

### Worker Management

```bash
# List Workers
kubectl get workers -n hiclaw-system

# Describe Worker
kubectl describe worker python-dev -n hiclaw-system

# Delete Worker
kubectl delete worker python-dev -n hiclaw-system
```

### Team Management

```bash
# List Teams
kubectl get teams -n hiclaw-system

# Describe Team
kubectl describe team backend-team -n hiclaw-system

# Delete Team
kubectl delete team backend-team -n hiclaw-system
```

### Check Controller Status

```bash
# View controller logs
kubectl logs -n hiclaw-system -l app=hiclaw-controller -f

# Check reconciliation metrics
kubectl get events -n hiclaw-system --sort-by='.lastTimestamp'
```

## Working with Skills

### MCP Server Skills

HiClaw supports declarative MCP (Model Context Protocol) server configuration.

**Worker with MCP skills:**

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: github-worker
spec:
  runtime: openclaw
  skills:
    - name: search-repos
      type: mcp
      config:
        serverName: github
        toolName: search_repositories
    - name: create-pr
      type: mcp
      config:
        serverName: github
        toolName: create_pull_request
```

**Team Leader with MCP servers:**

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: devops-team
spec:
  leader:
    runtime: copaw
    mcpServers:
      - name: github
        enabled: true
      - name: filesystem
        enabled: true
      - name: kubernetes
        enabled: true
        config:
          namespace: default
```

### Nacos Skills Registry

HiClaw supports pulling skills from Nacos registries (e.g., skills.sh with 80,000+ community skills).

**Worker with Nacos skills:**

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: cloud-worker
spec:
  runtime: openclaw
  skills:
    - name: aws-s3-upload
      type: nacos
      config:
        registry: https://skills.sh
        scope: sts-hiclaw
        skillId: aws-s3-upload-v1
```

### Custom Skills in Worker Packages

Workers can include custom skills via `SOUL.md` files in their packages.

**Directory structure:**

```
worker-package/
├── SOUL.md           # Worker personality and behavior
├── skills/
│   ├── custom-skill-1.py
│   └── custom-skill-2.py
└── requirements.txt
```

## Multi-Runtime Collaboration

HiClaw supports mixing different runtimes in the same Team.

### Example: QwenPaw Leader + Hermes Workers

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: autonomous-coding-team
spec:
  description: "Autonomous coding with human oversight"
  leader:
    runtime: copaw        # QwenPaw for deterministic coordination
    model: qwen3.5-plus
  workers:
    - name: hermes-coder  # Hermes for autonomous execution
      runtime: hermes
    - name: code-reviewer
      runtime: openclaw   # OpenClaw for review tasks
```

**Use case:**
- **QwenPaw Leader**: Task planning and coordination (deterministic)
- **Hermes Worker**: Autonomous code execution
- **OpenClaw Worker**: Code review and validation

## Code Examples

### Python: Interacting with HiClaw API

```python
import requests
import os

# HiClaw API base URL (gateway)
API_BASE = os.getenv("HICLAW_API_URL", "http://localhost:18080/api")
API_KEY = os.getenv("HICLAW_API_KEY")  # Consumer token

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# Create a Worker
def create_worker(name: str, runtime: str = "openclaw", skills: list = None):
    payload = {
        "apiVersion": "hiclaw.io/v1alpha1",
        "kind": "Worker",
        "metadata": {"name": name},
        "spec": {
            "runtime": runtime,
            "description": f"{name} agent",
            "skills": skills or []
        }
    }
    response = requests.post(
        f"{API_BASE}/workers",
        headers=headers,
        json=payload
    )
    return response.json()

# Create a Team
def create_team(name: str, workers: list, leader_runtime: str = "copaw"):
    payload = {
        "apiVersion": "hiclaw.io/v1alpha1",
        "kind": "Team",
        "metadata": {"name": name},
        "spec": {
            "leader": {"runtime": leader_runtime},
            "workers": [{"name": w} for w in workers]
        }
    }
    response = requests.post(
        f"{API_BASE}/teams",
        headers=headers,
        json=payload
    )
    return response.json()

# Example usage
worker = create_worker(
    name="python-expert",
    runtime="openclaw",
    skills=[
        {"name": "file-operations", "type": "builtin"},
        {"name": "github-search", "type": "mcp", "config": {"serverName": "github"}}
    ]
)

team = create_team(
    name="dev-team",
    workers=["python-expert"],
    leader_runtime="copaw"
)

print(f"Worker: {worker}")
print(f"Team: {team}")
```

### Go: Kubernetes Controller Pattern

```go
package main

import (
    "context"
    "fmt"
    
    hiclawv1alpha1 "github.com/higress-group/hiclaw/api/v1alpha1"
    corev1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/client"
)

// WorkerReconciler reconciles a Worker object
type WorkerReconciler struct {
    client.Client
    Scheme *runtime.Scheme
}

func (r *WorkerReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    var worker hiclawv1alpha1.Worker
    if err := r.Get(ctx, req.NamespacedName, &worker); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // Create Worker Pod
    pod := &corev1.Pod{
        ObjectMeta: metav1.ObjectMeta{
            Name:      worker.Name,
            Namespace: worker.Namespace,
            Labels: map[string]string{
                "app":     "hiclaw-worker",
                "worker":  worker.Name,
                "runtime": worker.Spec.Runtime,
            },
        },
        Spec: corev1.PodSpec{
            Containers: []corev1.Container{
                {
                    Name:  "worker",
                    Image: fmt.Sprintf("hiclaw/worker-%s:latest", worker.Spec.Runtime),
                    Env: append(worker.Spec.Env, corev1.EnvVar{
                        Name: "WORKER_NAME",
                        Value: worker.Name,
                    }),
                },
            },
        },
    }

    if err := ctrl.SetControllerReference(&worker, pod, r.Scheme); err != nil {
        return ctrl.Result{}, err
    }

    if err := r.Create(ctx, pod); err != nil {
        return ctrl.Result{}, err
    }

    return ctrl.Result{}, nil
}

func main() {
    mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), ctrl.Options{
        Scheme: scheme,
    })
    if err != nil {
        panic(err)
    }

    if err := (&WorkerReconciler{
        Client: mgr.GetClient(),
        Scheme: mgr.GetScheme(),
    }).SetupWithManager(mgr); err != nil {
        panic(err)
    }

    if err := mgr.Start(ctrl.SetupSignalHandler()); err != nil {
        panic(err)
    }
}
```

### Shell: Worker Template Marketplace

```bash
#!/bin/bash

# List available Worker templates
kubectl get workertemplates -n hiclaw-system

# Create Worker from template
cat <<EOF | kubectl apply -f -
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: my-python-worker
  namespace: hiclaw-system
spec:
  templateRef:
    name: python-developer-template
    version: v1.2.0
  customization:
    env:
      - name: PYTHON_VERSION
        value: "3.11"
EOF

# Check Worker status
kubectl get worker my-python-worker -n hiclaw-system -o yaml
```

## Common Patterns

### Pattern 1: Human-in-the-Loop Review

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: code-review-team
spec:
  description: "Code review with mandatory human approval"
  leader:
    runtime: copaw
  workers:
    - name: code-analyzer
    - name: security-scanner
  humans:
    - name: senior-dev
      role: reviewer
  workflow:
    approvalRequired: true
    approvers:
      - senior-dev
```

### Pattern 2: Autonomous + Deterministic Hybrid

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Team
metadata:
  name: hybrid-dev-team
spec:
  leader:
    runtime: copaw          # Deterministic planning
  workers:
    - name: hermes-coder    # Autonomous execution
      runtime: hermes
    - name: task-planner
      runtime: openclaw     # Structured task breakdown
```

### Pattern 3: Shared File System for Context

```yaml
apiVersion: hiclaw.io/v1alpha1
kind: Worker
metadata:
  name: document-analyzer
spec:
  runtime: openclaw
  storage:
    minio:
      enabled: true
      bucket: shared-docs
  skills:
    - name: analyze-document
      type: custom
      config:
        inputPath: /mnt/minio/shared-docs/input
        outputPath: /mnt/minio/shared-docs/output
```

**Usage in Worker code:**

```python
import os
from minio import Minio

# MinIO client (credentials injected by HiClaw)
minio_client = Minio(
    os.getenv("MINIO_ENDPOINT"),
    access_key=os.getenv("MINIO_ACCESS_KEY"),
    secret_key=os.getenv("MINIO_SECRET_KEY"),
    secure=False
)

# Read shared file
bucket = "shared-docs"
object_name = "input/document.pdf"
minio_client.fget_object(bucket, object_name, "/tmp/document.pdf")

# Process and upload result
minio_client.fput_object(bucket, "output/analysis.json", "/tmp/analysis.json")
```

## Troubleshooting

### Docker Installation Issues

**Problem:** Installer fails with "Docker not found"

```bash
# Verify Docker is running
docker ps

# On macOS, ensure Docker Desktop is started
open -a Docker

# On Linux, start Docker service
sudo systemctl start docker
```

**Problem:** Port conflicts (18080, 18088 already in use)

```bash
# Check what's using the port
lsof -i :18080

# Stop HiClaw and restart with custom ports
docker-compose -f ~/.hiclaw/docker-compose.yml down
# Edit ~/.hiclaw/.env to change ports
docker-compose -f ~/.hiclaw/docker-compose.yml up -d
```

### Kubernetes/Helm Issues

**Problem:** "no matches for kind Worker"

```bash
# CRDs not installed, reinstall HiClaw
helm uninstall hiclaw -n hiclaw-system
helm install hiclaw higress.io/hiclaw -n hiclaw-system --create-namespace
```

**Problem:** Worker Pod stuck in Pending

```bash
# Check events
kubectl describe pod <worker-pod> -n hiclaw-system

# Common issue: no default StorageClass
kubectl get storageclass
kubectl patch storageclass <name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

**Problem:** Controller not reconciling resources

```bash
# Check controller logs
kubectl logs -n hiclaw-system -l app=hiclaw-controller --tail=100

# Check RBAC permissions
kubectl auth can-i create workers --as=system:serviceaccount:hiclaw-system:hiclaw-controller -n hiclaw-system
```

### Agent Communication Issues

**Problem:** Workers not responding in Matrix room

```bash
# Check Worker Pod status
kubectl get pods -n hiclaw-system -l app=hiclaw-worker

# Check Worker logs
kubectl logs -n hiclaw-system <worker-pod-name>

# Verify Matrix server connection
kubectl exec -n hiclaw-system <worker-pod-name> -- curl http://tuwunel:8008/_matrix/client/versions
```

**Problem:** Manager not creating Workers

```bash
# Check Manager logs
kubectl logs -n hiclaw-system -l app=hiclaw-manager

# Verify Manager can access HiClaw API
kubectl exec -n hiclaw-system <manager-pod-name> -- curl http://hiclaw-controller:8080/api/health
```

### Credential Issues

**Problem:** "LLM API authentication failed"

```bash
# Docker: Check .env file
cat ~/.hiclaw/.env

# Kubernetes: Check Secret
kubectl get secret hiclaw-credentials -n hiclaw-system -o jsonpath='{.data.llmApiKey}' | base64 -d

# Update credentials
kubectl create secret generic hiclaw-credentials \
  --from-literal=llmApiKey=$NEW_API_KEY \
  --dry-run=client -o yaml | kubectl apply -f -

# Restart Manager
kubectl rollout restart deployment hiclaw-manager -n hiclaw-system
```

### Performance Issues

**Problem:** High memory usage with multiple Workers

```bash
# Switch to QwenPaw runtime (80% less memory than OpenClaw)
kubectl patch worker <worker-name> -n hiclaw-system \
  --type=merge -p '{"spec":{"runtime":"copaw"}}'

# Or set as default for new Workers
helm upgrade hiclaw higress.io/hiclaw -n hiclaw-system \
  --reuse-values \
  --set worker.defaultRuntime=copaw
```

**Problem:** Slow task execution

```bash
# Check if Workers are using shared file system (reduces token consumption)
kubectl get worker <worker-name> -n hiclaw-system -o jsonpath='{.spec.storage.minio.enabled}'

# Enable MinIO for Worker
kubectl patch worker <worker-name> -n hiclaw-system \
  --type=merge -p '{"spec":{"storage":{"minio":{"enabled":true,"bucket":"shared"}}}}'
```

## Upgrade Process

### Docker Upgrade

```bash
# Upgrade to latest (preserves data)
bash <(curl -sSL https://higress.ai/hiclaw/install.sh)

# Upgrade to specific version
HICLAW_VERSION=v1.1.2 bash <(curl -sSL https://higress.ai/hiclaw/install.sh)
```

### Kubernetes Upgrade

```bash
# Update Helm repo
helm repo update higress.io

# Upgrade (keeps existing values)
helm upgrade hiclaw higress.io/hiclaw -n hiclaw-system --reuse-values

# Or upgrade with new values
helm upgrade hiclaw higress.io/hiclaw -n hiclaw-system \
  --reuse-values \
  --set manager.runtime=copaw
```

## Uninstall

### Docker Uninstall

```bash
# macOS / Linux
bash <(curl -fsSL https://raw.githubusercontent.com/higress-group/hiclaw/main/install/hiclaw-install.sh) uninstall

# Windows
Set-ExecutionPolicy Bypass -Scope Process -Force
$wc=New-Object Net.WebClient
$wc.Encoding=[Text.Encoding]::UTF8
$s=$wc.DownloadString('https://raw.githubusercontent.com/higress-group/hiclaw/main/install/hiclaw-install.ps1')
& ([scriptblock]::Create($s)) uninstall
```

### Kubernetes Uninstall

```bash
# Remove HiClaw (keeps CRDs and PVCs by default)
helm uninstall hiclaw -n hiclaw-system

# Remove CRDs (WARNING: deletes all Workers, Teams, Humans)
kubectl delete crd workers.hiclaw.io teams.hiclaw.io humans.hiclaw.io

# Remove namespace (deletes all data)
kubectl delete namespace hiclaw-system
```

## Security Best Practices

1. **Credential Isolation**: Workers receive consumer tokens only. Real API keys stay in the Higress gateway.
2. **Namespace Scoping**: Use namespace-scoped RBAC for the HiClaw controller.
3. **Matrix Federation**: Disable federation for internal deployments to prevent external access.
4. **MCP Server Permissions**: Restrict MCP server access to necessary Workers only.
5. **Human Approval**: Enable `approvalRequired` in Team workflows for sensitive operations.

## Resources

- **Official Docs**: https://hiclaw.io
- **GitHub**: https://github.com/higress-group/hiclaw
- **Discord**: https://discord.com/invite/NVjNA4BAVw
- **Skills Marketplace**: https://skills.sh
- **Helm Charts**: https://higress.io/helm-charts
