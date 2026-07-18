---
name: agent-as-a-router-coding
description: Use Agent-as-a-Router (ACRouter) to intelligently route coding tasks to optimal models under performance-cost tradeoffs
triggers:
  - route this coding task to the best model
  - use ACRouter to select a model for this problem
  - set up agent-as-a-router for my project
  - evaluate models with CodeRouterBench
  - integrate ACRouter into my coding workflow
  - run the ACRouter baselines and benchmarks
  - implement agentic model routing for code tasks
  - add intelligent model selection to my agent
---

# Agent-as-a-Router Coding Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

ACRouter is an agentic model routing system that intelligently selects backend models for coding tasks, balancing performance and cost. It uses verifier feedback and escalation strategies to route problems through a hierarchy of models (cheap → strong), stopping when a solution passes verification. This skill covers installation, reproduction of benchmark results, runtime integration, and custom inference patterns.

## What ACRouter Does

- **Agentic Routing**: Routes coding tasks to different models (cheap-first, escalate on failure)
- **CodeRouterBench**: Public benchmark with ID (in-distribution) and OOD176 (out-of-distribution) tasks
- **Verifier-Driven**: Uses test execution or static analysis to validate solutions before escalating
- **Cost-Performance Tradeoff**: Optimizes for high performance per dollar spent
- **Runtime Integration**: Ships with plugins for Claude Code Router, cc-switch, and generic OpenRouter-compatible APIs

## Installation

```bash
# Clone the repository
git clone https://github.com/LanceZPF/agent-as-a-router.git
cd agent-as-a-router

# Create conda environment
conda create -n acrouter python=3.11 -y
conda activate acrouter

# Install dependencies
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m pip install -e .

# Run tests to verify installation
python -m unittest discover -s tests
```

## Reproduce Benchmark Results

### ID (In-Distribution) Evaluation

```bash
python scripts/run_id.py --output-dir outputs/tmp/id
```

Expected output: `ID n=2919 AvgPerf=50.14 CumReg=202.0 $Total=22.31 Perf/$=2.25 rAcc=0.2395`

### OOD176 (Out-of-Distribution) with ACRouter

```bash
python scripts/run_acrouter_ood176.py --output-dir outputs/tmp/acrouter_ood176
```

Expected output: `ACRouter-OOD176 n=176 AvgPerf=73.30 CumReg=15.9 $Total=86.72 Perf/$=0.85`

### OOD176 Baselines Comparison

```bash
python scripts/run_baselines_ood176.py --output-dir outputs/tmp/baselines_ood176
```

Generates a comparison table with Oracle, Single-Model, Round-Robin, and ACRouter strategies.

## Download Hugging Face Assets

### Minimal Dataset (OOD176 replay only)

```bash
python scripts/download_hf_assets.py --minimal --dataset-dir .hf/CodeRouterBench
```

### With Optional Trained Router Model

```bash
python scripts/download_hf_assets.py \
  --minimal \
  --with-router-model \
  --dataset-dir .hf/CodeRouterBench \
  --model-dir .hf/router_model
```

### Run from Downloaded Snapshot

```bash
python scripts/run_acrouter_ood176.py \
  --hf-dataset-dir .hf/CodeRouterBench \
  --output-dir outputs/tmp/acrouter_ood176_hf

python scripts/run_baselines_ood176.py \
  --hf-dataset-dir .hf/CodeRouterBench \
  --output-dir outputs/tmp/baselines_ood176_hf
```

## Runtime Integration: Inference API

### Basic ACRouter Usage

```python
from acrouter_repro.inference import ACRouter

# Initialize router with model hierarchy
router = ACRouter(
    candidate_models=["gpt-4o-mini", "gpt-4o", "claude-3.5-sonnet"],
    cheap_chain=["gpt-4o-mini"],
    escalate_to="gpt-4o",
    k=1,  # Number of cheap attempts before escalation
)

# Define your backend model caller
def call_model(model: str, task: dict) -> str:
    """Call your actual model API (OpenRouter, OpenAI, etc.)"""
    # Example: use OpenRouter
    import openai
    client = openai.OpenAI(
        base_url="https://openrouter.ai/api/v1",
        api_key=os.environ["OPENROUTER_API_KEY"]
    )
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": task["prompt"]}]
    )
    return response.choices[0].message.content

# Define your verifier (tests, static analysis, etc.)
def verify_solution(response: str, task: dict, model: str) -> bool:
    """Validate the generated code"""
    # Example: run pytest or static checks
    # Return True if solution passes, False to escalate
    return run_tests(response, task["test_file"])

# Route a task
task = {
    "task_id": "two_sum",
    "dimension": "algorithm",
    "prompt": "Write a function that solves two-sum problem...",
    "test_file": "tests/test_two_sum.py"
}

decision = router.run_with_verifier(
    task=task,
    call_model=call_model,
    verify=verify_solution
)

print(f"Chosen model: {decision.chosen_model}")
print(f"Solution: {decision.final_response}")
print(f"Cost: ${decision.total_cost:.4f}")
```

### Complete Inference Example

```python
import os
from acrouter_repro.inference import ACRouter

def main():
    # Set up router
    router = ACRouter(
        candidate_models=["deepseek-coder-v2", "claude-3.5-sonnet"],
        cheap_chain=["deepseek-coder-v2"],
        escalate_to="claude-3.5-sonnet",
        k=2  # Try cheap model twice before escalating
    )
    
    # Mock model caller (replace with real API)
    def call_model(model: str, task: dict) -> str:
        print(f"[ACRouter] Calling {model}...")
        # Your actual API call here
        return f"def solution(): pass  # Generated by {model}"
    
    # Mock verifier (replace with real test runner)
    def verify(response: str, task: dict, model: str) -> bool:
        print(f"[ACRouter] Verifying solution from {model}...")
        # Run actual tests: subprocess.run(["pytest", task["test_file"]])
        return "claude" in model  # Mock: only strong model passes
    
    # Task definition
    task = {
        "task_id": "bug_fix_001",
        "dimension": "bug_fixing",
        "prompt": "Fix the null pointer exception in src/parser.py"
    }
    
    # Route and solve
    decision = router.run_with_verifier(
        task=task,
        call_model=call_model,
        verify=verify
    )
    
    print(f"\n[Result]")
    print(f"  Model: {decision.chosen_model}")
    print(f"  Success: {decision.verified}")
    print(f"  Attempts: {len(decision.attempt_history)}")

if __name__ == "__main__":
    main()
```

Run: `python examples/inference_demo.py`

## Demo: API Coding Solver

Route a programming problem through multiple models until verification passes.

### Setup

```bash
export OPENROUTER_API_KEY="your-key-here"
```

### Configuration

Create `demos/api_coding_solver/models.json`:

```json
{
  "models": [
    {
      "name": "deepseek/deepseek-coder",
      "cost_per_1k_tokens": 0.0002,
      "provider": "openrouter"
    },
    {
      "name": "anthropic/claude-3.5-sonnet",
      "cost_per_1k_tokens": 0.015,
      "provider": "openrouter"
    }
  ],
  "verifier": {
    "command": "python",
    "args": ["-m", "pytest", "--tb=short"]
  }
}
```

### Run with Dry-Run

```bash
python demos/api_coding_solver/solve.py \
  --config demos/api_coding_solver/models.example.json \
  --problem-file demos/api_coding_solver/problems/two_sum.txt \
  --dry-run
```

### Solve a Problem

```bash
python demos/api_coding_solver/solve.py \
  --config demos/api_coding_solver/models.example.json \
  --problem-file demos/api_coding_solver/problems/two_sum.txt
```

## Demo: Commercial CLI Router

Route prompts to Codex, Claude Code, or Opencode CLI tools.

### Setup

```bash
# Set command prefixes (optional wrappers)
export ACROUTER_CODEX_PREFIX="ccswitch codex --"
export ACROUTER_CLAUDE_PREFIX="ccswitch claude --"
export ACROUTER_OPENCODE_PREFIX="ccswitch opencode --"
```

### Configuration

Edit `demos/commercial_cli_router/tools.example.json`:

```json
{
  "tools": {
    "codex": {
      "command": "codex",
      "args": ["--workdir", "{workdir}", "--prompt", "{prompt}"]
    },
    "claude": {
      "command": "claude-code",
      "args": ["--cwd", "{workdir}", "{prompt}"]
    },
    "opencode": {
      "command": "opencode",
      "args": ["{prompt}", "--directory", "{workdir}"]
    }
  },
  "default_tool": "codex"
}
```

### Route a Prompt

```bash
# Dry-run (show command without execution)
python demos/commercial_cli_router/router_mvp.py \
  --prompt "Patch this repository so pytest passes" \
  --dry-run

# Execute with selected tool
python demos/commercial_cli_router/router_mvp.py \
  --tool codex \
  --workdir /path/to/project \
  --prompt "Run the tests and fix the failing parser case"
```

## Config-Driven Pipeline

Use when you have precomputed task/model results.

### Example Config

Create `configs/my_eval.json`:

```json
{
  "input": {
    "matrix_file": "data/matrices/phase2_ood/unified/matrix_acrouter_ood176.json"
  },
  "router": {
    "type": "acrouter",
    "candidate_models": ["gpt-4o-mini", "gpt-4o"],
    "cheap_chain": ["gpt-4o-mini"],
    "escalate_to": "gpt-4o",
    "k": 1
  },
  "output": {
    "dir": "outputs/my_eval",
    "formats": ["csv", "json", "table"]
  }
}
```

### Run Pipeline

```bash
python scripts/run_pipeline.py --config configs/my_eval.json
```

## Add Custom Benchmark

### Prepare Input Data

Create `my_tasks.jsonl`:

```jsonl
{"task_id": "task_001", "dimension": "bug_fixing", "prompt": "Fix the parser..."}
{"task_id": "task_002", "dimension": "feature", "prompt": "Add CSV export..."}
```

Create `my_results.jsonl`:

```jsonl
{"task_id": "task_001", "model": "gpt-4o-mini", "resolved": true, "input_tokens": 150, "output_tokens": 300}
{"task_id": "task_001", "model": "gpt-4o", "resolved": true, "input_tokens": 150, "output_tokens": 280}
{"task_id": "task_002", "model": "gpt-4o-mini", "resolved": false, "input_tokens": 200, "output_tokens": 150}
{"task_id": "task_002", "model": "gpt-4o", "resolved": true, "input_tokens": 200, "output_tokens": 400}
```

### Pipeline Config

```json
{
  "input": {
    "tasks_file": "my_tasks.jsonl",
    "results_file": "my_results.jsonl"
  },
  "router": {
    "type": "acrouter",
    "candidate_models": ["gpt-4o-mini", "gpt-4o"],
    "cheap_chain": ["gpt-4o-mini"],
    "escalate_to": "gpt-4o",
    "k": 1
  },
  "output": {
    "dir": "outputs/my_custom_eval"
  }
}
```

### Run Evaluation

```bash
python scripts/run_pipeline.py --config configs/my_custom_eval.json
```

## Key Configuration Options

### ACRouter Parameters

```python
ACRouter(
    candidate_models=["model-a", "model-b", "model-c"],  # All available models
    cheap_chain=["model-a"],                              # Try these first (in order)
    escalate_to="model-c",                                # Fallback strong model
    k=1,                                                  # Max attempts per cheap model
    timeout_seconds=300,                                  # Per-model timeout
    max_total_cost=10.0,                                  # Budget limit (USD)
)
```

### Baseline Routers

```python
from src.routing.baselines import SingleModelRouter, RoundRobinRouter, OracleRouter

# Always use one model
single = SingleModelRouter(model="gpt-4o")

# Cycle through models
rr = RoundRobinRouter(models=["gpt-4o-mini", "gpt-4o"])

# Perfect hindsight (for upper bound analysis)
oracle = OracleRouter(results_matrix=results_df)
```

## Project Structure Reference

```
agent-as-a-router/
├── src/
│   ├── acrouter_repro/          # Core reproduction package
│   │   ├── inference.py          # ACRouter class
│   │   ├── evaluation.py         # Metrics and reporting
│   │   └── data_loader.py        # Load tasks/matrices
│   └── routing/
│       └── baselines.py          # Baseline router implementations
├── scripts/
│   ├── run_id.py                 # Run ID benchmark
│   ├── run_acrouter_ood176.py    # Run OOD176 with ACRouter
│   ├── run_baselines_ood176.py   # Compare all baselines
│   ├── run_pipeline.py           # Config-driven evaluation
│   └── download_hf_assets.py     # Fetch CodeRouterBench
├── demos/
│   ├── api_coding_solver/        # OpenRouter routing demo
│   └── commercial_cli_router/    # Codex/Claude/Opencode router
├── data/
│   ├── coderouterbench/          # Public benchmark tables
│   └── matrices/                 # Task-model result matrices
├── outputs/                      # Reference outputs (baselines_ood176, etc.)
└── configs/                      # Example pipeline configs
```

## Common Patterns

### Pattern 1: Cheap-First with Escalation

```python
router = ACRouter(
    candidate_models=["cheap-model", "medium-model", "strong-model"],
    cheap_chain=["cheap-model", "medium-model"],
    escalate_to="strong-model",
    k=2  # Try each cheap model twice
)
```

### Pattern 2: Budget-Constrained Routing

```python
router = ACRouter(
    candidate_models=["gpt-4o-mini", "gpt-4o"],
    cheap_chain=["gpt-4o-mini"],
    escalate_to="gpt-4o",
    k=1,
    max_total_cost=5.0  # Stop if cost exceeds $5
)

decision = router.run_with_verifier(task, call_model, verify)
if decision.total_cost > 5.0:
    print("Budget exceeded, using best available solution")
```

### Pattern 3: Dimension-Aware Routing

```python
def route_by_dimension(task: dict):
    if task["dimension"] == "bug_fixing":
        return ACRouter(
            candidate_models=["deepseek-coder", "claude-3.5-sonnet"],
            cheap_chain=["deepseek-coder"],
            escalate_to="claude-3.5-sonnet",
            k=2
        )
    elif task["dimension"] == "feature_addition":
        return ACRouter(
            candidate_models=["gpt-4o", "o1"],
            cheap_chain=["gpt-4o"],
            escalate_to="o1",
            k=1
        )
    else:
        return SingleModelRouter(model="gpt-4o")

router = route_by_dimension(task)
decision = router.run_with_verifier(task, call_model, verify)
```

### Pattern 4: Retry Logic with Verifier

```python
def verify_with_retry(response: str, task: dict, model: str) -> bool:
    """Run tests up to 3 times (flaky test tolerance)"""
    for attempt in range(3):
        result = subprocess.run(
            ["pytest", task["test_file"]],
            capture_output=True,
            timeout=60
        )
        if result.returncode == 0:
            return True
        time.sleep(1)
    return False

decision = router.run_with_verifier(task, call_model, verify_with_retry)
```

## Troubleshooting

### Issue: Import Error for `acrouter_repro`

**Solution**: Ensure you installed in editable mode:

```bash
python -m pip install -e .
```

### Issue: Missing Hugging Face Assets

**Solution**: Download the minimal dataset:

```bash
python scripts/download_hf_assets.py --minimal --dataset-dir .hf/CodeRouterBench
```

Then run with `--hf-dataset-dir`:

```bash
python scripts/run_acrouter_ood176.py --hf-dataset-dir .hf/CodeRouterBench --output-dir outputs/tmp
```

### Issue: API Key Not Found

**Solution**: Set the required environment variable:

```bash
export OPENROUTER_API_KEY="your-key-here"
# or for other providers
export OPENAI_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"
```

### Issue: Verifier Always Returns False

**Solution**: Check that your verifier command is correct and the test environment is set up:

```python
def verify(response: str, task: dict, model: str) -> bool:
    # Debug: print what's being verified
    print(f"[DEBUG] Verifying response from {model}")
    print(f"[DEBUG] Response length: {len(response)} chars")
    
    # Ensure test file exists
    if not os.path.exists(task["test_file"]):
        print(f"[ERROR] Test file not found: {task['test_file']}")
        return False
    
    # Run tests with verbose output
    result = subprocess.run(
        ["pytest", task["test_file"], "-v"],
        capture_output=True,
        text=True
    )
    
    print(f"[DEBUG] Test output:\n{result.stdout}")
    return result.returncode == 0
```

### Issue: OOD176 Results Don't Match Paper

**Solution**: Ensure you're using the correct matrix and pricing:

```bash
# Use the official unified matrix
python scripts/run_acrouter_ood176.py \
  --matrix data/matrices/phase2_ood/unified/matrix_acrouter_ood176.json \
  --output-dir outputs/tmp/acrouter_ood176
```

### Issue: Pipeline Config Not Found

**Solution**: Use absolute paths or ensure working directory is project root:

```bash
cd /path/to/agent-as-a-router
python scripts/run_pipeline.py --config configs/eval_pipeline.example.json
```

### Issue: High Memory Usage During Evaluation

**Solution**: Process tasks in batches:

```python
from acrouter_repro.data_loader import load_tasks

tasks = load_tasks("data/coderouterbench/ood176_tasks.jsonl")
batch_size = 50

for i in range(0, len(tasks), batch_size):
    batch = tasks[i:i+batch_size]
    for task in batch:
        decision = router.run_with_verifier(task, call_model, verify)
        # Process decision immediately
        save_result(decision)
```

## Advanced: Custom Router Implementation

```python
from typing import List, Dict, Any
from acrouter_repro.inference import BaseRouter, RoutingDecision

class MyCustomRouter(BaseRouter):
    """Route based on task difficulty heuristic"""
    
    def __init__(self, models: List[str], difficulty_threshold: int = 100):
        self.models = models
        self.difficulty_threshold = difficulty_threshold
    
    def route(self, task: Dict[str, Any]) -> str:
        """Select model based on prompt length (simple heuristic)"""
        difficulty = len(task.get("prompt", ""))
        
        if difficulty < self.difficulty_threshold:
            return self.models[0]  # Use cheap model
        else:
            return self.models[-1]  # Use strong model
    
    def run_with_verifier(self, task, call_model, verify):
        chosen_model = self.route(task)
        response = call_model(chosen_model, task)
        verified = verify(response, task, chosen_model)
        
        return RoutingDecision(
            task_id=task["task_id"],
            chosen_model=chosen_model,
            final_response=response,
            verified=verified,
            total_cost=0.0,  # Calculate based on your pricing
            attempt_history=[{"model": chosen_model, "verified": verified}]
        )

# Use it
router = MyCustomRouter(models=["gpt-4o-mini", "gpt-4o"], difficulty_threshold=200)
decision = router.run_with_verifier(task, call_model, verify)
```

## Testing Your Integration

```bash
# Run unit tests
python -m unittest discover -s tests

# Validate installation
python -c "from acrouter_repro.inference import ACRouter; print('✓ Import OK')"

# Check syntax of all Python files
python -m py_compile $(find src scripts demos examples -name '*.py' -print)

# Verify no secrets leaked (if contributing)
bash scripts/check_no_secrets.sh
```

## Resources

- **Homepage**: https://www.omnisource.cn/agent-as-a-router
- **Paper**: https://arxiv.org/abs/2606.22902
- **Dataset**: https://huggingface.co/datasets/Lance1573/CodeRouterBench
- **Router Model**: https://huggingface.co/Lance1573/acrouter-qwen35-08b-router-lora
- **Full Documentation**: `docs/HANDBOOK.md` in the repository
