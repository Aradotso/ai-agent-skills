---
name: agentic-harness-engineering
description: Evolve coding agent harnesses automatically using observability-driven iteration with NexAU components
triggers:
  - How do I set up agentic harness engineering?
  - Help me evolve a coding agent harness with AHE
  - Show me how to run harness evolution experiments
  - Configure AHE for automatic agent improvement
  - How do I analyze agent traces with AHE?
  - Run iterative harness optimization with NexAU
  - Set up E2B templates for AHE experiments
  - Debug and improve my coding agent with AHE
---

# Agentic Harness Engineering

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

**Agentic Harness Engineering (AHE)** is an observability-driven system for automatically evolving the harness around a coding agent. The base LLM model remains frozen while AHE iteratively improves the harness components: system prompts, tool descriptions, tool implementations, middleware, skills, sub-agents, and long-term memory.

AHE operates through a three-phase loop:
1. **Evaluate** — Run the agent over a dataset, capture full traces
2. **Analyze** — Distill traces into root-cause reports using Agent Debugger
3. **Improve** — Evolve Agent proposes evidence-backed harness edits

Key achievements:
- Lifts GPT-5.4 from 69.7% → 77.0% pass@1 on Terminal-Bench 2 over 10 iterations
- Reaches 84.7% ± 2.1% with GPT-5.5 (ranked #3 on TB2 leaderboard)
- Produces transferable harnesses that work across models and benchmarks

## Installation

### Prerequisites

- Python ≥ 3.13
- [uv](https://docs.astral.sh/uv/) package manager
- tmux (for background experiment runs)

```bash
# macOS
brew install uv tmux

# Linux
curl -LsSf https://astral.sh/uv/install.sh | sh
sudo apt install -y tmux
```

### Clone and Install

```bash
git clone https://github.com/china-qijizhifeng/agentic-harness-engineering.git
cd agentic-harness-engineering
uv sync
```

### Environment Configuration

```bash
cp .env.example .env
```

Required environment variables:

```bash
# Main LLM endpoint (used by code_agent and evolve_agent)
LLM_API_KEY="your_api_key_here"
LLM_BASE_URL="https://api.openai.com/v1"

# E2B sandbox (required for safe code execution)
E2B_API_KEY="your_e2b_key"

# Web search for evolve_agent
SERPER_API_KEY="your_serper_key"
```

Optional specialized endpoints:

```bash
# Stronger model for Agent Debugger
ADB_LLM_API_KEY="your_key"
ADB_LLM_BASE_URL="https://api.anthropic.com/v1"

# Specific model for GPT-5.4 experiments
GPT54_LLM_API_KEY="your_key"
GPT54_LLM_BASE_URL="https://api.openai.com/v1"
```

### E2B Sandbox Setup

AHE supports two E2B deployment modes:

**SaaS E2B (default):**
```bash
# Only set E2B_API_KEY, leave E2B_API_URL unset
E2B_API_KEY="your_e2b_key"
```

**Self-hosted E2B:**
```bash
E2B_API_KEY="your_e2b_key"
E2B_API_URL="https://your-e2b-host.example.com"
E2B_DOMAIN="your-e2b-host.example.com"
```

### Build E2B Templates

Before running experiments, build sandbox templates for your dataset:

```bash
# Build all templates (16 parallel jobs)
uv run python scripts/build_templates.py \
  --dataset-dir /path/to/harbor-datasets/terminal-bench-2 \
  -j 16

# Retry failed builds only
uv run python scripts/build_templates.py \
  --dataset-dir /path/to/harbor-datasets/terminal-bench-2 \
  --retry-failed

# Build specific tasks
uv run python scripts/build_templates.py \
  --dataset-dir /path/to/harbor-datasets/terminal-bench-2 \
  task_001 task_002
```

For private Docker registries:
```bash
export DOCKER_REGISTRY_USERNAME="your_username"
export DOCKER_REGISTRY_PASSWORD="your_password"
```

## Key Commands

### Running Experiments

```bash
# Launch single experiment in tmux background
./scripts/evolve.sh configs/experiments/exp-003-simple-code-gpt54.yaml

# Launch and attach to see logs in real-time
./scripts/evolve.sh --attach configs/experiments/exp-003-simple-code-gpt54.yaml

# Batch launch all experiments
./scripts/evolve.sh --batch

# Resume a paused/crashed experiment
./scripts/evolve-resume.sh runs/exp-003-simple-code-gpt54
```

### Tmux Session Management

```bash
# List running experiments
tmux ls

# Attach to running experiment
tmux attach -t exp-003-simple-code-gpt54

# Detach from session (keeps running): Ctrl-b d

# Kill experiment
tmux kill-session -t exp-003-simple-code-gpt54
```

### Manual Execution Steps

```bash
# Run just the evaluation phase
uv run python evolve.py \
  --config configs/experiments/exp-003-simple-code-gpt54.yaml \
  --run-dir runs/exp-003-simple-code-gpt54 \
  --phase evaluate

# Run just the analysis phase
uv run python evolve.py \
  --config configs/experiments/exp-003-simple-code-gpt54.yaml \
  --run-dir runs/exp-003-simple-code-gpt54 \
  --phase analyze

# Run just the improvement phase
uv run python evolve.py \
  --config configs/experiments/exp-003-simple-code-gpt54.yaml \
  --run-dir runs/exp-003-simple-code-gpt54 \
  --phase improve
```

## Configuration

### Experiment Configuration Structure

Experiments use a base + overlay pattern:

```yaml
# configs/base.yaml - shared defaults
dataset:
  path: "/path/to/harbor-datasets/terminal-bench-2"
  
harbor:
  max_workers: 8
  timeout: 600
  
evolution:
  max_iterations: 10
  target_pass_rate: 0.85

# configs/experiments/my-experiment.yaml - overlay
extends: ../base.yaml

experiment:
  name: "my-experiment"
  description: "Testing new middleware configuration"

llm:
  model: "gpt-5.4-turbo"
  temperature: 0.0
  
harbor:
  max_workers: 16  # Override base setting
```

### Key Configuration Sections

**Dataset Configuration:**
```yaml
dataset:
  path: "/path/to/harbor-datasets/terminal-bench-2"
  tasks:  # Optional: run subset
    - "task_001"
    - "task_002"
```

**Harbor Executor Settings:**
```yaml
harbor:
  max_workers: 8  # Parallel sandbox limit (respect E2B tier quota)
  timeout: 600    # Per-task timeout in seconds
  retry_failed: true
```

**Evolution Loop Settings:**
```yaml
evolution:
  max_iterations: 10
  target_pass_rate: 0.85
  early_stop_patience: 3  # Stop if no improvement for N iterations
```

**LLM Configuration:**
```yaml
llm:
  model: "gpt-5.4-turbo"
  temperature: 0.0
  max_tokens: 8192
  api_key_env: "LLM_API_KEY"  # References .env variable
  base_url_env: "LLM_BASE_URL"
```

## Code Examples

### Basic Evolution Loop (Python)

```python
from pathlib import Path
import yaml
from evolve import EvolutionOrchestrator

# Load configuration
config_path = Path("configs/experiments/my-experiment.yaml")
with open(config_path) as f:
    config = yaml.safe_load(f)

# Initialize orchestrator
orchestrator = EvolutionOrchestrator(
    config=config,
    run_dir=Path("runs/my-experiment"),
    resume=False
)

# Run full evolution loop
results = orchestrator.run()

print(f"Final pass rate: {results['final_pass_rate']:.2%}")
print(f"Iterations completed: {results['iterations']}")
print(f"Best iteration: {results['best_iteration']}")
```

### Custom Middleware Component

```python
# agents/code_agent_simple/workspace/middleware/custom_logger.py
from typing import Any, Dict
from nexau.middleware import Middleware

class CustomLogger(Middleware):
    """Log all tool calls with timing information."""
    
    def __init__(self, log_file: str = "tool_calls.jsonl"):
        self.log_file = log_file
        
    async def before_tool_call(
        self,
        tool_name: str,
        arguments: Dict[str, Any],
        context: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Called before each tool execution."""
        import time
        context["start_time"] = time.time()
        return context
        
    async def after_tool_call(
        self,
        tool_name: str,
        result: Any,
        context: Dict[str, Any]
    ) -> Any:
        """Called after tool execution."""
        import time
        import json
        
        elapsed = time.time() - context.get("start_time", 0)
        
        log_entry = {
            "tool": tool_name,
            "elapsed_ms": round(elapsed * 1000, 2),
            "success": not isinstance(result, Exception)
        }
        
        with open(self.log_file, "a") as f:
            f.write(json.dumps(log_entry) + "\n")
            
        return result
```

### Custom Evolution Skill

```python
# agents/evolve_agent/workspace/skills/analyze_performance.py
from nexau.skills import Skill
from typing import Dict, Any

class AnalyzePerformance(Skill):
    """Analyze pass rate trends across iterations."""
    
    name = "analyze_performance"
    description = "Analyze performance trends and identify regression patterns"
    
    def __init__(self, run_dir: str):
        self.run_dir = Path(run_dir)
        
    async def execute(self, **kwargs) -> Dict[str, Any]:
        """
        Analyze pass rates across iterations.
        
        Returns:
            trend: "improving", "degrading", or "stable"
            current_rate: Current pass rate
            best_rate: Best pass rate achieved
            recommendations: List of actionable recommendations
        """
        iterations = sorted(self.run_dir.glob("iteration_*"))
        
        pass_rates = []
        for iter_dir in iterations:
            results_file = iter_dir / "evaluation" / "results.json"
            if results_file.exists():
                with open(results_file) as f:
                    data = json.load(f)
                    pass_rates.append(data["pass_rate"])
        
        if len(pass_rates) < 2:
            return {"trend": "insufficient_data"}
            
        # Calculate trend
        recent_avg = sum(pass_rates[-3:]) / len(pass_rates[-3:])
        earlier_avg = sum(pass_rates[:-3]) / len(pass_rates[:-3])
        
        if recent_avg > earlier_avg + 0.02:
            trend = "improving"
        elif recent_avg < earlier_avg - 0.02:
            trend = "degrading"
        else:
            trend = "stable"
            
        recommendations = []
        if trend == "degrading":
            recommendations.append("Review recent changes for regressions")
            recommendations.append("Consider reverting last iteration's edits")
        elif trend == "stable":
            recommendations.append("Try more aggressive harness modifications")
            
        return {
            "trend": trend,
            "current_rate": pass_rates[-1],
            "best_rate": max(pass_rates),
            "recommendations": recommendations
        }
```

### Programmatic Trace Analysis

```python
from pathlib import Path
import json

def analyze_failure_patterns(iteration_dir: Path):
    """Extract common failure patterns from evaluation traces."""
    
    traces_dir = iteration_dir / "evaluation" / "tasks"
    failures = []
    
    for task_dir in traces_dir.iterdir():
        reward_file = task_dir / "verifier" / "reward.txt"
        trace_file = task_dir / "agent" / "nexau_in_memory_tracer.cleaned.json"
        
        # Check if task failed
        if reward_file.exists():
            with open(reward_file) as f:
                if f.read().strip() != "1.0":  # Failed
                    # Load trace
                    with open(trace_file) as tf:
                        trace = json.load(tf)
                        
                    failures.append({
                        "task": task_dir.name,
                        "steps": len(trace.get("steps", [])),
                        "last_error": extract_last_error(trace)
                    })
    
    # Group by error type
    error_counts = {}
    for failure in failures:
        error = failure["last_error"]
        error_counts[error] = error_counts.get(error, 0) + 1
        
    return {
        "total_failures": len(failures),
        "error_distribution": error_counts,
        "examples": failures[:5]
    }

def extract_last_error(trace: dict) -> str:
    """Extract the last error message from a trace."""
    steps = trace.get("steps", [])
    for step in reversed(steps):
        if "error" in step:
            return step["error"]
        if step.get("tool_result", {}).get("success") is False:
            return step["tool_result"].get("error", "unknown_error")
    return "no_error_found"
```

## Common Patterns

### Pattern 1: Iterative Refinement

```python
# Start with a baseline experiment
./scripts/evolve.sh configs/experiments/baseline.yaml

# After 10 iterations, fork the best harness
cp -r runs/baseline/iteration_007/input runs/baseline-v2/iteration_000/input

# Run refinement with different configuration
./scripts/evolve.sh configs/experiments/baseline-v2.yaml
```

### Pattern 2: A/B Testing Middleware

```yaml
# configs/experiments/test-middleware-a.yaml
experiment:
  name: "test-middleware-a"

harness_overrides:
  middleware:
    - "context_compaction"
    - "error_recovery"

# configs/experiments/test-middleware-b.yaml
experiment:
  name: "test-middleware-b"
  
harness_overrides:
  middleware:
    - "context_compaction"
    - "ralph_loop"  # Different middleware
```

### Pattern 3: Transfer Learning Across Benchmarks

```python
# Train on Terminal-Bench 2
./scripts/evolve.sh configs/experiments/train-tb2.yaml

# Freeze the best harness
best_iter = "runs/train-tb2/iteration_008"
frozen_harness = "harnesses/tb2-frozen"
cp -r f"{best_iter}/input" frozen_harness

# Test on SWE-bench-Verified (no further evolution)
uv run python evolve.py \
  --config configs/experiments/test-swebench.yaml \
  --workspace frozen_harness \
  --max-iterations 1  # Single evaluation, no evolution
```

### Pattern 4: Multi-Model Comparison

```python
models = ["gpt-5.4-turbo", "claude-4-opus", "gpt-5.5-preview"]

for model in models:
    config = {
        "experiment": {"name": f"compare-{model}"},
        "llm": {"model": model},
        "extends": "configs/base.yaml"
    }
    
    with open(f"configs/experiments/compare-{model}.yaml", "w") as f:
        yaml.dump(config, f)
    
    # Launch experiment
    os.system(f"./scripts/evolve.sh configs/experiments/compare-{model}.yaml")
```

## Troubleshooting

### E2B Sandbox Issues

**Problem:** Sandboxes fail to start with "concurrent limit exceeded"

```bash
# Check your E2B tier's concurrent sandbox limit
# Reduce max_workers in config to stay under the limit

harbor:
  max_workers: 4  # Conservative for free tier
```

**Problem:** Template build fails with Docker errors

```bash
# Ensure Docker credentials are set for private registries
export DOCKER_REGISTRY_USERNAME="your_username"
export DOCKER_REGISTRY_PASSWORD="your_password"

# Rebuild specific failed template
uv run python scripts/build_templates.py \
  --dataset-dir /path/to/dataset \
  --retry-failed \
  task_name
```

### Agent Debugger Issues

**Problem:** Analysis phase produces empty reports

```bash
# Check Agent Debugger has access to traces
ls runs/my-exp/iteration_001/evaluation/tasks/*/agent/nexau_in_memory_tracer.cleaned.json

# Verify ADB_LLM environment variables
echo $ADB_LLM_API_KEY
echo $ADB_LLM_BASE_URL

# Run analysis manually with verbose output
uv run python evolve.py \
  --config configs/experiments/my-exp.yaml \
  --phase analyze \
  --verbose
```

### Evolution Stalls

**Problem:** Evolve Agent makes no changes across iterations

```bash
# Check if workspace is read-only
ls -la runs/my-exp/iteration_NNN/evolve/workspace/

# Verify Evolve Agent has proper permissions
chmod -R u+w runs/my-exp/iteration_NNN/evolve/workspace/

# Increase evolution temperature for more exploration
# In config:
llm:
  temperature: 0.3  # Higher = more creative changes
```

### Performance Degradation

**Problem:** Pass rate decreases after certain iteration

```bash
# Identify the regression iteration
cat runs/my-exp/iteration_*/evaluation/results.json | grep pass_rate

# Rollback to last good iteration
cp -r runs/my-exp/iteration_005/input runs/my-exp/iteration_007/input

# Resume from that point
./scripts/evolve-resume.sh runs/my-exp --from-iteration 7
```

### Trace Conversion Errors

**Problem:** `trace_converter.py` fails on certain tasks

```python
# Debug trace conversion manually
from trace_converter import convert_trace

task_dir = Path("runs/my-exp/iteration_001/evaluation/tasks/task_042")
trace_file = task_dir / "agent" / "nexau_in_memory_tracer.cleaned.json"

try:
    with open(trace_file) as f:
        trace = json.load(f)
    converted = convert_trace(trace)
    print(f"Conversion successful: {len(converted['steps'])} steps")
except Exception as e:
    print(f"Conversion failed: {e}")
    # Examine raw trace
    print(json.dumps(trace, indent=2)[:1000])
```

## Advanced Usage

### Custom Evolution Objectives

```python
# agents/evolve_agent/workspace/skills/custom_objective.py
class OptimizeForSpeed(Skill):
    """Optimize harness for execution speed, not just pass rate."""
    
    async def execute(self, iteration_data: dict) -> dict:
        """
        Analyze execution time and suggest optimizations.
        
        Focus on:
        - Reducing redundant tool calls
        - Parallelizing independent operations
        - Caching frequently accessed data
        """
        tasks = iteration_data["evaluation"]["tasks"]
        
        slow_tasks = [
            t for t in tasks 
            if t["elapsed_time"] > 120  # Over 2 minutes
        ]
        
        optimizations = []
        for task in slow_tasks:
            # Analyze trace for inefficiencies
            trace = load_trace(task["trace_file"])
            redundant_calls = find_redundant_calls(trace)
            
            if redundant_calls:
                optimizations.append({
                    "task": task["name"],
                    "issue": "redundant_tool_calls",
                    "suggestion": f"Cache results of {redundant_calls[0]}"
                })
                
        return {"optimizations": optimizations}
```

### Integration with External Tools

```python
# Send evolution progress to monitoring dashboard
import requests

def send_metrics(iteration: int, pass_rate: float):
    """Push metrics to external monitoring."""
    requests.post(
        "https://your-dashboard.example.com/api/metrics",
        json={
            "experiment": "my-experiment",
            "iteration": iteration,
            "pass_rate": pass_rate,
            "timestamp": time.time()
        },
        headers={"Authorization": f"Bearer {os.getenv('DASHBOARD_API_KEY')}"}
    )
```

## References

- [Official Documentation](https://arxiv.org/abs/2604.25850)
- [NexAU Framework](https://github.com/nex-agi/NexAU)
- [Harbor Datasets](https://github.com/laude-institute/harbor-datasets)
- [E2B Documentation](https://e2b.dev/docs)
- [Terminal-Bench 2 Leaderboard](https://www.tbench.ai/leaderboard/terminal-bench/2.0)
