---
name: agentenv-distributed-agent-environments
description: Run and manage agent environments at scale using AgentENV's Firecracker-based microVM platform with snapshot, fork, and distributed storage support.
triggers:
  - "set up AgentENV for agent training"
  - "create a microVM sandbox with AgentENV"
  - "snapshot and fork agent environments"
  - "deploy AgentENV cluster for distributed agents"
  - "manage agent sandboxes with aenv CLI"
  - "integrate E2B with AgentENV"
  - "scale agent environments across machines"
  - "pause and resume agent microVMs"
---

# AgentENV Distributed Agent Environments

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

AgentENV (AENV) is a distributed platform for running agent environments at scale using Firecracker microVMs. It provides fast snapshot/resume (<50ms boot, <100ms pause), native fork support, incremental snapshots to S3/distributed storage, and OCI image loading via overlaybd. Built in Rust, it powers agentic RL training workloads like Kimi K3.

## Prerequisites

- **Linux kernel 6.8+** (Ubuntu 24.04 recommended)
- `/dev/kvm` access for Firecracker
- **Security Warning**: AgentENV has no built-in authorization. Run only on trusted networks or behind an auth proxy.

## Installation

### Option 1: Install Script (Ubuntu 24.04)

Installs both server and CLI, starts server as systemd service:

```bash
curl -fsSL https://raw.githubusercontent.com/kvcache-ai/AgentENV/main/scripts/install.sh | sudo bash
sudo systemctl start aenv
```

Check status:

```bash
sudo systemctl status aenv
```

### Option 2: Docker

```bash
curl -fsSL https://raw.githubusercontent.com/kvcache-ai/AgentENV/main/scripts/docker-setup.sh | sudo bash
docker pull ghcr.io/kvcache-ai/aenv-server:latest
docker run -d --privileged -v /dev:/dev -p 8000:8000 ghcr.io/kvcache-ai/aenv-server:latest
```

### CLI Only (Linux/macOS, x86_64/arm64)

If server is on another machine or using Docker:

```bash
curl -fsSL https://raw.githubusercontent.com/kvcache-ai/AgentENV/main/scripts/install-cli.sh | bash
```

## Authentication

Configure CLI to point at your server:

```bash
aenv auth
# AENV server URL [http://localhost:8000]: http://127.0.0.1:8000
# API key: dummy
```

For production, set:

```bash
export AENV_SERVER_URL=http://your-server:8000
export AENV_API_KEY="${AENV_API_KEY}"
```

## Core Concepts

### Templates

Templates are OCI-compatible images converted to AgentENV format. They serve as base images for sandboxes.

### Sandboxes

Sandboxes are running microVM environments created from templates. They can be paused, resumed, snapshotted, and forked.

## CLI Commands

### Template Management

```bash
# Pull Docker image as template
aenv pull docker.io/library/ubuntu:22.04 --name ubuntu
aenv pull python:3.11-slim --name python311

# List templates
aenv template list
aenv template ls  # alias
```

### Sandbox Lifecycle

```bash
# Start interactive sandbox (attaches shell)
aenv start ubuntu

# Start detached (returns sandbox ID)
aenv start ubuntu --detach
# Output: sandbox-abc123def456

# List running sandboxes
aenv ls
aenv list --output json  # JSON output for scripting
```

### Sandbox Operations

```bash
# Attach to running sandbox
aenv cn sandbox-abc123def456

# Execute command in sandbox
aenv exec sandbox-abc123def456 ls -la /
aenv exec sandbox-abc123def456 python3 script.py

# Pause sandbox (free CPU/memory)
aenv pause sandbox-abc123def456

# Resume paused sandbox
aenv resume sandbox-abc123def456

# Extend TTL (timeout in seconds)
aenv timeout sandbox-abc123def456 600  # 10 minutes from now
aenv timeout sandbox-abc123def456 3600  # 1 hour

# Delete sandbox
aenv delete sandbox-abc123def456
aenv rm sandbox-abc123def456  # alias
```

## HTTP API Usage

AgentENV exposes a REST API compatible with E2B. Base URL defaults to `http://localhost:8000`.

### Create Sandbox

```bash
curl -X POST http://localhost:8000/sandboxes \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${AENV_API_KEY}" \
  -d '{
    "template": "ubuntu",
    "timeout": 600
  }'
```

Response:

```json
{
  "sandbox_id": "sandbox-abc123def456",
  "status": "running"
}
```

### Execute Command

```bash
curl -X POST http://localhost:8000/sandboxes/sandbox-abc123def456/exec \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${AENV_API_KEY}" \
  -d '{
    "cmd": ["python3", "-c", "print(\"Hello from AgentENV\")"]
  }'
```

Response:

```json
{
  "exit_code": 0,
  "stdout": "Hello from AgentENV\n",
  "stderr": ""
}
```

### Pause/Resume

```bash
# Pause
curl -X POST http://localhost:8000/sandboxes/sandbox-abc123def456/pause \
  -H "X-API-Key: ${AENV_API_KEY}"

# Resume
curl -X POST http://localhost:8000/sandboxes/sandbox-abc123def456/resume \
  -H "X-API-Key: ${AENV_API_KEY}"
```

### Delete Sandbox

```bash
curl -X DELETE http://localhost:8000/sandboxes/sandbox-abc123def456 \
  -H "X-API-Key: ${AENV_API_KEY}"
```

## E2B Compatibility

AgentENV implements the E2B API. Use the official E2B SDK without code changes.

### Python SDK

```bash
pip install e2b
```

```python
import os
from e2b import Sandbox

# Point to AgentENV server
os.environ["E2B_API_URL"] = "http://localhost:8000"
os.environ["E2B_API_KEY"] = os.getenv("AENV_API_KEY", "dummy")

# Create sandbox from template
sandbox = Sandbox(template="ubuntu", timeout=600)

try:
    # Execute commands
    result = sandbox.commands.run("ls -la /")
    print(result.stdout)
    
    # Write and execute Python script
    sandbox.filesystem.write("/tmp/test.py", "print('Hello from E2B on AgentENV')")
    output = sandbox.commands.run("python3 /tmp/test.py")
    print(output.stdout)
    
finally:
    sandbox.close()
```

### TypeScript SDK

```bash
npm install @e2b/sdk
```

```typescript
import { Sandbox } from '@e2b/sdk';

process.env.E2B_API_URL = 'http://localhost:8000';
process.env.E2B_API_KEY = process.env.AENV_API_KEY || 'dummy';

const sandbox = await Sandbox.create({ 
  template: 'ubuntu',
  timeout: 600 
});

try {
  const result = await sandbox.commands.run('ls -la /');
  console.log(result.stdout);
  
  await sandbox.filesystem.write('/tmp/test.js', 'console.log("Hello from E2B on AgentENV")');
  const output = await sandbox.commands.run('node /tmp/test.js');
  console.log(output.stdout);
} finally {
  await sandbox.close();
}
```

## Snapshot and Fork Patterns

### Creating Checkpoints

Snapshots complete in <100ms even with heavy disk modifications:

```bash
# Via CLI (requires API call or direct server access)
curl -X POST http://localhost:8000/sandboxes/sandbox-abc123def456/snapshot \
  -H "X-API-Key: ${AENV_API_KEY}" \
  -d '{"name": "checkpoint-training-epoch-5"}'
```

### Forking Environments

Fork a running sandbox for parallel workflows:

```bash
curl -X POST http://localhost:8000/sandboxes/sandbox-abc123def456/fork \
  -H "X-API-Key: ${AENV_API_KEY}"
```

Response:

```json
{
  "sandbox_id": "sandbox-xyz789ghi012",
  "parent_id": "sandbox-abc123def456"
}
```

## Agent Training Workflow Example

```python
import os
import time
from e2b import Sandbox

os.environ["E2B_API_URL"] = "http://localhost:8000"
os.environ["E2B_API_KEY"] = os.getenv("AENV_API_KEY")

def train_agent_episode(template: str, agent_code: str, episode_num: int):
    """Run single training episode in isolated sandbox."""
    sandbox = Sandbox(template=template, timeout=3600)
    
    try:
        # Install dependencies
        sandbox.commands.run("pip install numpy gymnasium torch")
        
        # Deploy agent code
        sandbox.filesystem.write("/workspace/agent.py", agent_code)
        
        # Run training episode
        result = sandbox.commands.run(
            f"python /workspace/agent.py --episode {episode_num}",
            timeout=1800
        )
        
        # Collect metrics
        metrics = sandbox.filesystem.read("/workspace/metrics.json")
        
        return {
            "episode": episode_num,
            "exit_code": result.exit_code,
            "metrics": metrics,
            "sandbox_id": sandbox.id
        }
        
    finally:
        sandbox.close()

# Run parallel episodes
agent_code = open("my_agent.py").read()
results = []

for i in range(10):
    result = train_agent_episode("python311", agent_code, i)
    results.append(result)
    print(f"Episode {i} completed: {result['metrics']}")
```

## Distributed Cluster Deployment

### Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  aenv-server:
    image: ghcr.io/kvcache-ai/aenv-server:latest
    privileged: true
    volumes:
      - /dev:/dev
    ports:
      - "8000:8000"
    environment:
      - AENV_BIND_ADDRESS=0.0.0.0:8000
      - AENV_STORAGE_BACKEND=s3
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - AWS_REGION=${AWS_REGION}
      - AENV_S3_BUCKET=${AENV_S3_BUCKET}
    restart: unless-stopped
```

Deploy:

```bash
docker-compose up -d
```

### Kubernetes (Helm)

```bash
helm repo add aenv https://kvcache-ai.github.io/AgentENV/charts
helm install aenv aenv/agentenv \
  --set storage.backend=s3 \
  --set storage.s3.bucket="${AENV_S3_BUCKET}" \
  --set storage.s3.region="${AWS_REGION}"
```

## Configuration

### Environment Variables

```bash
# Server configuration
export AENV_BIND_ADDRESS=0.0.0.0:8000
export AENV_LOG_LEVEL=info  # debug, info, warn, error

# Storage backend (local, s3, distributed-fs)
export AENV_STORAGE_BACKEND=s3
export AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}"
export AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}"
export AWS_REGION=us-west-2
export AENV_S3_BUCKET=my-aenv-snapshots

# Resource limits
export AENV_MAX_SANDBOXES=100
export AENV_DEFAULT_TIMEOUT=600  # seconds
export AENV_MAX_MEMORY_MB=2048
export AENV_MAX_VCPUS=2

# Overlaybd cache
export AENV_CACHE_SIZE_GB=50
export AENV_CACHE_DIR=/var/cache/aenv
```

## Advanced Patterns

### Custom Template Creation

```bash
# Pull base image
aenv pull ubuntu:22.04 --name base-ubuntu

# Start sandbox and customize
SANDBOX_ID=$(aenv start base-ubuntu --detach)

# Install software
aenv exec $SANDBOX_ID apt-get update
aenv exec $SANDBOX_ID apt-get install -y python3-pip git

# Create template from running sandbox
curl -X POST http://localhost:8000/templates \
  -H "X-API-Key: ${AENV_API_KEY}" \
  -d "{\"name\": \"custom-python\", \"sandbox_id\": \"$SANDBOX_ID\"}"

# Clean up
aenv rm $SANDBOX_ID
```

### Long-Running Agent with Auto-Pause

```python
import time
from e2b import Sandbox

sandbox = Sandbox(template="ubuntu", timeout=86400)  # 24h

try:
    while True:
        # Do work
        result = sandbox.commands.run("python agent_step.py")
        
        # Pause during idle periods
        if result.stdout.strip() == "idle":
            sandbox.pause()
            time.sleep(60)
            sandbox.resume()
        
        time.sleep(5)
        
finally:
    sandbox.close()
```

### Batch Processing with Sandbox Pool

```python
from concurrent.futures import ThreadPoolExecutor
from e2b import Sandbox

def process_task(task_id: int, template: str):
    sandbox = Sandbox(template=template, timeout=600)
    try:
        result = sandbox.commands.run(f"python process.py --task {task_id}")
        return {"task_id": task_id, "output": result.stdout}
    finally:
        sandbox.close()

# Process 50 tasks across 10 parallel sandboxes
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(process_task, i, "python311") for i in range(50)]
    results = [f.result() for f in futures]

print(f"Processed {len(results)} tasks")
```

## Troubleshooting

### Sandbox Won't Start

Check KVM access:

```bash
ls -l /dev/kvm
# Should show: crw-rw---- 1 root kvm

# Add user to kvm group
sudo usermod -aG kvm $USER
```

Check kernel version:

```bash
uname -r
# Should be 6.8.0 or higher
```

### Server Not Responding

Check service status:

```bash
sudo systemctl status aenv
sudo journalctl -u aenv -n 50
```

For Docker:

```bash
docker ps
docker logs <container-id>
```

### Template Pull Fails

Check overlaybd setup:

```bash
# Verify overlaybd is running
sudo systemctl status overlaybd

# Check disk space
df -h /var/cache/aenv
```

### Sandbox Timeout Issues

Extend timeout before starting:

```bash
# CLI
aenv start ubuntu --detach
aenv timeout sandbox-abc123def456 7200  # 2 hours

# API
curl -X POST http://localhost:8000/sandboxes \
  -H "X-API-Key: ${AENV_API_KEY}" \
  -d '{"template": "ubuntu", "timeout": 7200}'
```

### Memory/CPU Limits

Configure per-sandbox resources:

```bash
curl -X POST http://localhost:8000/sandboxes \
  -H "X-API-Key: ${AENV_API_KEY}" \
  -d '{
    "template": "ubuntu",
    "mem_size_mib": 4096,
    "vcpu_count": 4
  }'
```

### Check Sandbox Logs

```bash
# Via CLI
aenv exec sandbox-abc123def456 cat /var/log/syslog

# Via API
curl http://localhost:8000/sandboxes/sandbox-abc123def456/logs \
  -H "X-API-Key: ${AENV_API_KEY}"
```

## Performance Optimization

### Pre-warm Templates

Pull templates before workload starts:

```bash
aenv pull ubuntu:22.04 --name ubuntu
aenv pull python:3.11-slim --name python311
aenv pull nvidia/cuda:12.2.0-runtime-ubuntu22.04 --name cuda
```

### Use Pause/Resume for Idle Periods

Pausing releases CPU and most memory in <100ms:

```python
# Pause during long waits
sandbox.pause()
time.sleep(300)  # Wait for external event
sandbox.resume()
```

### Leverage Forking for Parallel Workflows

Fork instead of creating new sandboxes for faster startup:

```bash
# Create base sandbox with loaded dependencies
BASE_ID=$(aenv start python311 --detach)
aenv exec $BASE_ID pip install torch numpy pandas

# Fork for parallel tasks
FORK1=$(curl -X POST http://localhost:8000/sandboxes/$BASE_ID/fork | jq -r .sandbox_id)
FORK2=$(curl -X POST http://localhost:8000/sandboxes/$BASE_ID/fork | jq -r .sandbox_id)
```

## Resources

- **Documentation**: https://kvcache-ai.github.io/AgentENV/
- **GitHub**: https://github.com/kvcache-ai/AgentENV
- **Issues**: https://github.com/kvcache-ai/AgentENV/issues
- **E2B SDK**: https://github.com/e2b-dev/e2b
