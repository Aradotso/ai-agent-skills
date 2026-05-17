---
name: hermes-agent-self-evolution
description: Evolutionary self-improvement for Hermes Agent using DSPy + GEPA to optimize skills, prompts, and code
triggers:
  - evolve a hermes agent skill
  - optimize hermes agent prompts with GEPA
  - run self-evolution on agent skills
  - improve hermes agent with evolutionary search
  - generate evaluation data for skill optimization
  - use DSPy to optimize agent capabilities
  - run GEPA optimizer on tool descriptions
  - automate hermes agent improvement
---

# Hermes Agent Self-Evolution

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Hermes Agent Self-Evolution is an evolutionary optimization framework for [Hermes Agent](https://github.com/NousResearch/hermes-agent) that uses DSPy + GEPA (Genetic-Pareto Prompt Evolution) to automatically improve agent skills, tool descriptions, system prompts, and code through reflective evolutionary search. No GPU training required — everything operates via API calls with execution trace analysis.

## Installation

```bash
# Clone the repository
git clone https://github.com/NousResearch/hermes-agent-self-evolution.git
cd hermes-agent-self-evolution

# Install with development dependencies
pip install -e ".[dev]"

# Set environment variables
export HERMES_AGENT_REPO=~/.hermes/hermes-agent  # Path to your hermes-agent repo
export OPENAI_API_KEY=your_openai_api_key        # For DSPy/GEPA optimizer
```

## Key Commands

### Evolving Skills

```bash
# Basic skill evolution with synthetic eval data
python -m evolution.skills.evolve_skill \
    --skill github-code-review \
    --iterations 10 \
    --eval-source synthetic

# Use real session history from agent databases
python -m evolution.skills.evolve_skill \
    --skill github-code-review \
    --iterations 10 \
    --eval-source sessiondb

# Specify custom hermes-agent repo path
python -m evolution.skills.evolve_skill \
    --skill web-scraping \
    --iterations 5 \
    --hermes-repo /path/to/hermes-agent

# Use specific model for optimization
python -m evolution.skills.evolve_skill \
    --skill data-analysis \
    --iterations 8 \
    --model gpt-4-turbo
```

### Generating Evaluation Data

```bash
# Generate synthetic evaluation dataset for a skill
python -m evolution.eval.generate_eval_data \
    --skill github-code-review \
    --num-examples 50 \
    --output eval_data/github-code-review.json

# Extract real usage examples from session database
python -m evolution.eval.extract_sessions \
    --skill web-scraping \
    --min-quality 0.7 \
    --output eval_data/web-scraping-real.json
```

## Core Concepts

### Evolution Pipeline

The optimization process follows this flow:

1. **Read current artifact** (skill file, prompt, tool description)
2. **Generate evaluation dataset** (synthetic or from real sessions)
3. **GEPA optimization loop**:
   - Generate candidate variants through mutation
   - Execute variants against eval dataset
   - Analyze execution traces for failure patterns
   - Propose targeted improvements
   - Apply constraint gates (tests, size limits, benchmarks)
4. **Select best variant** based on multi-objective criteria
5. **Create PR** against hermes-agent repository

### Evaluation Sources

- **`synthetic`**: LLM-generated scenarios based on skill description
- **`sessiondb`**: Real usage traces from Claude Code, Copilot, Hermes sessions
- **`benchmark`**: Predefined test suites (SWE-bench, HumanEval, etc.)

## Python API

```python
from evolution.skills.optimizer import SkillOptimizer
from evolution.eval.dataset import EvalDataset

# Initialize optimizer
optimizer = SkillOptimizer(
    hermes_repo_path="~/.hermes/hermes-agent",
    model="gpt-4-turbo",
    temperature=0.7
)

# Load or generate evaluation data
eval_dataset = EvalDataset.from_synthetic(
    skill_name="github-code-review",
    num_examples=30
)

# Run optimization
result = optimizer.optimize(
    skill_name="github-code-review",
    eval_dataset=eval_dataset,
    iterations=10,
    population_size=8
)

print(f"Original score: {result.baseline_score}")
print(f"Optimized score: {result.best_score}")
print(f"Improvement: {result.improvement_pct}%")

# Access the optimized skill content
optimized_skill = result.best_variant.content

# Create PR with changes
result.create_pull_request(
    title="[Self-Evolution] Optimize github-code-review skill",
    branch="evolution/github-code-review"
)
```

### Custom Evaluation Functions

```python
from evolution.eval.metrics import EvalMetric

class TaskSuccessRate(EvalMetric):
    """Custom metric for task completion rate."""
    
    def evaluate(self, prediction, example):
        """Score prediction against ground truth."""
        # Extract success indicators from execution trace
        completed_steps = self.count_completed_steps(prediction.trace)
        total_steps = len(example.expected_steps)
        
        return {
            "score": completed_steps / total_steps,
            "metadata": {
                "completed": completed_steps,
                "total": total_steps
            }
        }

# Use custom metric
optimizer = SkillOptimizer(
    metrics=[TaskSuccessRate(), "accuracy", "response_quality"]
)
```

### Constraint Gates

```python
from evolution.constraints import ConstraintGate

# Define custom constraints
constraints = [
    ConstraintGate.max_size(15_000),  # 15KB max for skill files
    ConstraintGate.test_suite_passes(),  # Must pass pytest
    ConstraintGate.no_semantic_drift(threshold=0.85),  # Preserve purpose
    ConstraintGate.benchmark_threshold("swe_bench_lite", min_score=0.6)
]

optimizer = SkillOptimizer(constraints=constraints)
```

## Configuration

Create `evolution_config.yaml` in your project root:

```yaml
# Model configuration
model:
  provider: openai
  name: gpt-4-turbo
  temperature: 0.7
  max_tokens: 4096

# GEPA optimizer settings
gepa:
  population_size: 8
  mutation_rate: 0.3
  crossover_rate: 0.5
  elite_count: 2
  max_generations: 10

# Evaluation settings
evaluation:
  default_source: synthetic
  num_examples: 30
  min_quality_threshold: 0.7
  use_execution_traces: true

# Constraints
constraints:
  max_skill_size: 15360  # 15KB
  max_tool_description: 500  # chars
  require_test_pass: true
  semantic_drift_threshold: 0.85

# Repository paths
paths:
  hermes_agent: ~/.hermes/hermes-agent
  eval_data: ./eval_data
  results: ./evolution_results

# PR automation
pull_requests:
  auto_create: false
  branch_prefix: evolution/
  require_approval: true
  assignees: []
```

Load configuration:

```python
from evolution.config import load_config

config = load_config("evolution_config.yaml")
optimizer = SkillOptimizer.from_config(config)
```

## Common Patterns

### Batch Optimization

Optimize multiple skills in sequence:

```python
from evolution.batch import BatchOptimizer

skills = [
    "github-code-review",
    "web-scraping",
    "data-analysis",
    "api-integration"
]

batch = BatchOptimizer(
    skills=skills,
    hermes_repo_path="~/.hermes/hermes-agent",
    iterations=10
)

results = batch.run(
    eval_source="synthetic",
    parallel=True,
    max_workers=4
)

# Generate comparison report
batch.generate_report("evolution_results/batch_report.html")
```

### Continuous Evolution Pipeline

Set up automated continuous improvement:

```python
from evolution.continuous import ContinuousPipeline
from datetime import timedelta

pipeline = ContinuousPipeline(
    hermes_repo_path="~/.hermes/hermes-agent",
    check_interval=timedelta(days=7)
)

# Watch for new session data
pipeline.watch_sessions(
    min_sessions=100,  # Wait for 100 new sessions
    quality_threshold=0.7
)

# Run evolution when triggered
pipeline.on_trigger(
    callback=lambda skill: optimizer.optimize(skill),
    create_pr=True
)

pipeline.start()
```

### A/B Testing Variants

Compare multiple evolved variants:

```python
from evolution.testing import ABTest

# Generate multiple variants
variants = optimizer.evolve_variants(
    skill_name="github-code-review",
    num_variants=5,
    iterations=10
)

# Run A/B test
ab_test = ABTest(
    baseline=original_skill,
    variants=variants,
    eval_dataset=eval_dataset
)

results = ab_test.run(num_trials=100)

# Statistical significance testing
winner = results.get_winner(confidence=0.95)
print(f"Winner: {winner.variant_id} (p={winner.p_value})")
```

## Real-World Examples

### Example 1: Optimize GitHub Code Review Skill

```python
from evolution.skills.optimizer import SkillOptimizer
from evolution.eval.dataset import EvalDataset

# Initialize
optimizer = SkillOptimizer(
    hermes_repo_path="~/.hermes/hermes-agent",
    model="gpt-4-turbo"
)

# Generate synthetic eval data based on skill description
eval_dataset = EvalDataset.from_synthetic(
    skill_name="github-code-review",
    num_examples=40,
    scenarios=[
        "review PR with security vulnerabilities",
        "review PR with performance issues",
        "review PR with style violations",
        "review PR with breaking changes"
    ]
)

# Run optimization
result = optimizer.optimize(
    skill_name="github-code-review",
    eval_dataset=eval_dataset,
    iterations=15,
    constraints=[
        "max_size:15000",
        "test_suite_passes",
        "no_semantic_drift:0.85"
    ]
)

# Review improvements
print(f"Baseline: {result.baseline_score:.2f}")
print(f"Optimized: {result.best_score:.2f}")
print(f"\nKey improvements:")
for improvement in result.improvements:
    print(f"  - {improvement}")

# Create PR if improvement is significant
if result.improvement_pct > 10:
    result.create_pull_request(
        title="[Self-Evolution] Optimize github-code-review skill (+{:.1f}%)".format(
            result.improvement_pct
        ),
        branch="evolution/github-code-review-v2"
    )
```

### Example 2: Use Real Session Data

```python
from evolution.eval.sessions import SessionExtractor

# Extract high-quality sessions from agent database
extractor = SessionExtractor(
    session_db_path="~/.hermes/sessions.db"
)

real_examples = extractor.extract(
    skill_name="web-scraping",
    min_quality=0.8,
    min_session_length=5,  # At least 5 turns
    max_examples=50
)

# Convert to eval dataset
eval_dataset = EvalDataset.from_sessions(real_examples)

# Optimize using real user data
result = optimizer.optimize(
    skill_name="web-scraping",
    eval_dataset=eval_dataset,
    iterations=10
)
```

### Example 3: Multi-Objective Optimization

```python
from evolution.objectives import MultiObjective

# Define multiple objectives
objectives = MultiObjective([
    ("task_success", weight=0.5),
    ("response_quality", weight=0.3),
    ("efficiency", weight=0.2)
])

result = optimizer.optimize(
    skill_name="data-analysis",
    eval_dataset=eval_dataset,
    objectives=objectives,
    pareto_optimal=True  # Use Pareto frontier selection
)

# Visualize trade-offs
result.plot_pareto_frontier(
    x_axis="task_success",
    y_axis="efficiency",
    output="pareto_analysis.png"
)
```

## Troubleshooting

### Optimization Stalls

If evolution isn't improving scores:

```python
# Increase mutation diversity
optimizer = SkillOptimizer(
    mutation_rate=0.5,  # More aggressive mutations
    temperature=0.9     # More creative variations
)

# Or use trace-guided mutations
optimizer.enable_trace_analysis(
    focus_on_failures=True,
    extract_error_patterns=True
)
```

### Constraint Failures

Check which constraints are failing:

```python
result = optimizer.optimize(skill_name="github-code-review")

for variant in result.rejected_variants:
    print(f"Variant {variant.id} failed:")
    for constraint, passed in variant.constraint_results.items():
        if not passed:
            print(f"  ✗ {constraint}: {variant.constraint_errors[constraint]}")
```

### API Rate Limits

Handle rate limiting gracefully:

```python
from evolution.utils import RateLimiter

optimizer = SkillOptimizer(
    rate_limiter=RateLimiter(
        max_requests_per_minute=50,
        backoff_strategy="exponential"
    )
)
```

### Evaluation Dataset Quality

Validate dataset before optimization:

```python
from evolution.eval.validation import DatasetValidator

validator = DatasetValidator()
issues = validator.validate(eval_dataset)

if issues:
    print("Dataset issues found:")
    for issue in issues:
        print(f"  - {issue}")
    
    # Auto-fix common issues
    eval_dataset = validator.auto_fix(eval_dataset)
```

## Integration with Hermes Agent

Evolved skills automatically integrate with Hermes Agent:

```bash
# After optimization, test the evolved skill
cd $HERMES_AGENT_REPO
git checkout evolution/github-code-review

# Run hermes with the evolved skill
hermes --skill github-code-review "Review this PR: https://github.com/..."

# Compare with baseline
hermes-benchmark compare \
    --baseline main \
    --variant evolution/github-code-review \
    --skill github-code-review
```

## Cost Estimation

Typical costs per optimization run:

- **Skill optimization** (10 iterations, 30 examples): ~$2-5
- **Tool description** (5 iterations, 20 examples): ~$1-2
- **System prompt** (15 iterations, 50 examples): ~$5-10

Reduce costs:

```python
optimizer = SkillOptimizer(
    model="gpt-3.5-turbo",  # Cheaper model
    cache_predictions=True,  # Reuse evaluations
    early_stopping=True      # Stop if no improvement
)
```
