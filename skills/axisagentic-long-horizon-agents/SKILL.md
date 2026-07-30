---
name: axisagentic-long-horizon-agents
description: AxisAgentic framework for building, running, and collecting trajectories from long-horizon AI agents with trace-based execution and SFT export
triggers:
  - build a long-horizon AI agent with AxisAgentic
  - set up agent trajectory collection
  - configure AxisAgentic runtime for agent execution
  - export agent traces for supervised fine-tuning
  - implement web search agent with AxisAgentic
  - replay and evaluate agent execution traces
  - create custom agent recipe with AxisAgentic
  - manage agent context and rollback with AxisAgentic
---

# AxisAgentic Long-Horizon Agents

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

AxisAgentic is an extensible runtime and trajectory-collection framework for long-horizon AI agents. It provides append-only trace execution, multi-turn orchestration, tool management, context budgets, recovery mechanisms, and state-faithful SFT export. The framework preserves runtime visibility so traces can support replay, evaluation, and training data generation.

## Installation

Requires Python 3.12+ and an OpenAI-compatible model endpoint.

```bash
git clone https://github.com/XYZ-AI-Lab/AxisAgentic.git
cd AxisAgentic

# Create virtual environment
python3.12 -m venv .venv
source .venv/bin/activate

# Install dependencies
./setup_env.sh

# Set up environment
source .envs/axis_agentic_env.sh

# Configure provider and credentials
cp .env.example .envs/.env
```

Edit `.envs/.env` with your provider configuration:

```bash
# OpenAI-compatible endpoint
OPENAI_API_KEY=your_api_key
OPENAI_BASE_URL=https://api.openai.com/v1

# Or other providers
# ANTHROPIC_API_KEY=...
# AZURE_OPENAI_API_KEY=...
```

## Core Concepts

### Traces and Visibility

Every task execution creates an append-only trace that records:
- Model requests and responses
- Tool calls and results
- Context compaction events
- Rollback and recovery markers
- Token and timing metrics

Traces preserve what the model saw at each stage, enabling replay, evaluation, and SFT export with correct visibility boundaries.

### Recipes

Recipes define agent behavior by combining:
- Model clients (OpenAI, Anthropic, local)
- Tools (web search, scraping, calculators)
- Orchestrators (multi-turn execution logic)
- Evaluators (correctness, reward functions)
- Context management policies

The framework includes Web Search and WideSearch reference recipes.

## Configuration

Recipes use YAML configuration with strict schemas. Create from templates:

```bash
cp recipe/web_search/configs/default.yaml my-config.yaml
```

### Basic Configuration Structure

```yaml
# Model client configuration
model_client:
  provider: openai
  model: gpt-4o
  temperature: 0.7
  max_tokens: 4096

# Tool configuration
tools:
  - type: web_search
    max_results: 10
  - type: web_scraper
    timeout: 30

# Context management
context:
  max_tokens: 100000
  compaction_threshold: 80000
  compaction_strategy: summarize

# Execution limits
limits:
  max_turns: 50
  max_tool_calls: 100
  timeout_seconds: 600

# Dataset and evaluation
dataset:
  path: data/my_tasks.jsonl
  format: jsonl
  
evaluator:
  type: llm_judge
  judge_model: gpt-4o
```

## Running Agent Tasks

### Dry Run (Validation)

Validate configuration without execution:

```bash
python -m recipe.web_search.runners.run_eval_config \
  --config my-config.yaml \
  --dry-run
```

### Execute Tasks

Run agent on configured dataset:

```bash
python -m recipe.web_search.runners.run_eval_config \
  --config my-config.yaml \
  --output-dir ./outputs/run1
```

Results written to:
- `outputs/run1/traces/` - Full execution traces
- `outputs/run1/metrics/` - Token counts, timing
- `outputs/run1/evaluations/` - Correctness scores
- `outputs/run1/summary.json` - Run summary

### Resume Failed Tasks

Resume from checkpoint:

```bash
python -m recipe.web_search.runners.run_eval_config \
  --config my-config.yaml \
  --output-dir ./outputs/run1 \
  --resume
```

## Working with Traces

### Trace Structure

Each trace is a JSON array of events:

```python
[
  {
    "type": "task_start",
    "timestamp": "2026-07-30T10:00:00Z",
    "task_id": "task_001",
    "query": "What is quantum computing?"
  },
  {
    "type": "model_request",
    "turn": 1,
    "messages": [...],
    "visible_tokens": 1024
  },
  {
    "type": "model_response",
    "turn": 1,
    "content": "...",
    "tool_calls": [...]
  },
  {
    "type": "tool_execution",
    "turn": 1,
    "tool": "web_search",
    "result": {...}
  },
  {
    "type": "context_compaction",
    "turn": 5,
    "before_tokens": 85000,
    "after_tokens": 40000,
    "strategy": "summarize"
  },
  {
    "type": "task_complete",
    "status": "success",
    "final_answer": "..."
  }
]
```

### Replay Traces

Reconstruct execution with visibility rules:

```bash
python -m axis_agentic.tools.replay_trace \
  --trace outputs/run1/traces/task_001.json \
  --output replay_output.json
```

Python API:

```python
from axis_agentic.replay import TraceReplayer

replayer = TraceReplayer()
with open('outputs/run1/traces/task_001.json') as f:
    trace = json.load(f)

# Replay with original visibility
replay = replayer.replay(trace, apply_visibility_markers=True)

# Access state at specific turn
state_at_turn_5 = replayer.get_state_at_turn(trace, turn=5)
```

## SFT Export

Export traces as supervised training data with correct visibility:

```bash
python -m axis_agentic.exporters.sft_exporter \
  --traces-dir outputs/run1/traces \
  --output sft_data.jsonl \
  --format swift_agent \
  --filter-status success \
  --min-turns 3
```

### Export Formats

**Swift Agent format:**

```jsonl
{"messages": [{"role": "user", "content": "..."}, {"role": "assistant", "content": "...", "tool_calls": [...]}, {"role": "tool", "content": "..."}], "metadata": {"task_id": "...", "trace_path": "...", "final_status": "success"}}
```

**Custom exporter:**

```python
from axis_agentic.exporters.base import BaseExporter

class CustomExporter(BaseExporter):
    def export_trace(self, trace, metadata):
        # Apply visibility markers
        visible_history = self.apply_visibility(trace)
        
        # Format for your training system
        return {
            "input": visible_history[0]["content"],
            "output": visible_history[-1]["content"],
            "trajectory": visible_history,
            "custom_metadata": metadata
        }

exporter = CustomExporter()
exporter.export_directory(
    traces_dir='outputs/run1/traces',
    output_path='custom_format.jsonl',
    filters={'status': 'success', 'min_reward': 0.8}
)
```

## Creating Custom Recipes

### Recipe Structure

```
recipe/
  my_agent/
    configs/
      default.yaml
    runners/
      run_eval_config.py
    orchestrators/
      my_orchestrator.py
    tools/
      my_tool.py
    evaluators/
      my_evaluator.py
```

### Custom Tool

```python
from axis_agentic.tools.base import BaseTool
from axis_agentic.types import ToolResult

class MyCustomTool(BaseTool):
    name = "my_tool"
    description = "Description for model to understand when to use"
    
    def __init__(self, config):
        super().__init__(config)
        self.api_key = os.getenv('MY_TOOL_API_KEY')
    
    def parameters_schema(self):
        return {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Query to process"},
                "max_results": {"type": "integer", "default": 5}
            },
            "required": ["query"]
        }
    
    def execute(self, parameters):
        query = parameters['query']
        max_results = parameters.get('max_results', 5)
        
        try:
            # Your tool logic
            results = self._call_api(query, max_results)
            
            return ToolResult(
                success=True,
                content=results,
                metadata={'count': len(results)}
            )
        except Exception as e:
            return ToolResult(
                success=False,
                error=str(e)
            )
```

### Custom Orchestrator

```python
from axis_agentic.orchestrators.base import BaseOrchestrator
from axis_agentic.types import OrchestratorResult

class MyOrchestrator(BaseOrchestrator):
    def __init__(self, model_client, tools, context_manager, config):
        super().__init__(model_client, tools, context_manager, config)
        self.max_turns = config.get('max_turns', 20)
    
    async def run(self, task):
        trace = self.init_trace(task)
        turn = 0
        
        while turn < self.max_turns:
            # Check context budget
            if self.context_manager.should_compact():
                self.context_manager.compact()
                trace.append(self.context_manager.get_compaction_event())
            
            # Model inference
            response = await self.model_client.complete(
                messages=self.context_manager.get_messages(),
                tools=[tool.schema() for tool in self.tools]
            )
            
            trace.append({
                'type': 'model_response',
                'turn': turn,
                'content': response.content,
                'tool_calls': response.tool_calls
            })
            
            # Execute tools
            if response.tool_calls:
                for tool_call in response.tool_calls:
                    result = await self.execute_tool(tool_call)
                    trace.append({
                        'type': 'tool_execution',
                        'turn': turn,
                        'tool': tool_call.name,
                        'result': result
                    })
                    self.context_manager.add_tool_result(result)
            
            # Check completion
            if self.is_complete(response):
                break
            
            turn += 1
        
        return OrchestratorResult(
            trace=trace,
            final_answer=self.extract_answer(response),
            status='success' if self.is_complete(response) else 'max_turns'
        )
```

### Custom Evaluator

```python
from axis_agentic.evaluators.base import BaseEvaluator

class MyEvaluator(BaseEvaluator):
    def __init__(self, config):
        super().__init__(config)
        self.judge_model = config.get('judge_model', 'gpt-4o')
    
    async def evaluate(self, task, result):
        # Extract components
        question = task['query']
        answer = result.final_answer
        reference = task.get('reference_answer')
        
        # LLM-as-judge evaluation
        judge_prompt = f"""
Question: {question}
Answer: {answer}
Reference: {reference}

Is the answer correct and complete? Respond with:
- "correct" if fully accurate
- "partial" if partially correct
- "incorrect" if wrong

Verdict:"""
        
        verdict = await self.call_judge(judge_prompt)
        
        return {
            'correctness': verdict.lower(),
            'score': 1.0 if verdict == 'correct' else (0.5 if verdict == 'partial' else 0.0),
            'judge_response': verdict
        }
```

### Recipe Configuration

```yaml
# recipe/my_agent/configs/default.yaml
model_client:
  provider: openai
  model: gpt-4o
  temperature: 0.7

orchestrator:
  type: my_orchestrator
  max_turns: 30

tools:
  - type: my_tool
    config:
      timeout: 60

evaluator:
  type: my_evaluator
  judge_model: gpt-4o

dataset:
  path: data/my_tasks.jsonl

output:
  traces_dir: outputs/traces
  metrics_dir: outputs/metrics
```

### Recipe Runner

```python
# recipe/my_agent/runners/run_eval_config.py
import argparse
import yaml
from axis_agentic.runtime import Runtime
from recipe.my_agent.orchestrators.my_orchestrator import MyOrchestrator
from recipe.my_agent.tools.my_tool import MyCustomTool
from recipe.my_agent.evaluators.my_evaluator import MyEvaluator

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--config', required=True)
    parser.add_argument('--output-dir', default='./outputs')
    parser.add_argument('--dry-run', action='store_true')
    parser.add_argument('--resume', action='store_true')
    args = parser.parse_args()
    
    with open(args.config) as f:
        config = yaml.safe_load(f)
    
    runtime = Runtime(
        config=config,
        orchestrator_cls=MyOrchestrator,
        tools=[MyCustomTool],
        evaluator_cls=MyEvaluator,
        output_dir=args.output_dir
    )
    
    if args.dry_run:
        runtime.validate()
        print("Configuration valid")
        return
    
    runtime.run(resume=args.resume)

if __name__ == '__main__':
    main()
```

## Context Management

### Automatic Compaction

```yaml
context:
  max_tokens: 100000
  compaction_threshold: 80000
  compaction_strategy: summarize  # or 'truncate', 'sliding_window'
  preserve_recent_turns: 5
```

### Manual Context Control

```python
from axis_agentic.context import ContextManager

context = ContextManager(config={
    'max_tokens': 100000,
    'compaction_threshold': 80000
})

# Add messages
context.add_user_message("What is AI?")
context.add_assistant_message("AI is...")

# Check budget
if context.should_compact():
    context.compact(strategy='summarize')

# Rollback
context.rollback_to_turn(5)

# Get visible state
messages = context.get_visible_messages()
```

## Evaluation

### Run Evaluation

```bash
python -m recipe.web_search.runners.run_eval_config \
  --config config.yaml \
  --output-dir outputs/eval1
```

### Custom Metrics

```python
from axis_agentic.evaluators.metrics import register_metric

@register_metric('custom_f1')
def compute_custom_f1(predictions, references):
    # Your metric logic
    return {
        'f1': f1_score,
        'precision': precision,
        'recall': recall
    }
```

## Common Patterns

### Error Recovery

```python
class RecoveryOrchestrator(BaseOrchestrator):
    async def run(self, task):
        max_retries = 3
        attempt = 0
        
        while attempt < max_retries:
            try:
                result = await self.execute_task(task)
                if self.is_valid(result):
                    return result
                else:
                    # Rollback and retry
                    self.context_manager.rollback_to_last_valid()
                    attempt += 1
            except Exception as e:
                self.trace.append({'type': 'error', 'error': str(e)})
                attempt += 1
        
        return self.create_failure_result()
```

### Multi-Model Orchestration

```python
from axis_agentic.clients import create_model_client

class MultiModelOrchestrator(BaseOrchestrator):
    def __init__(self, config):
        self.fast_model = create_model_client({
            'provider': 'openai',
            'model': 'gpt-4o-mini'
        })
        self.smart_model = create_model_client({
            'provider': 'openai',
            'model': 'gpt-4o'
        })
    
    async def run(self, task):
        # Fast model for tool selection
        tool_plan = await self.fast_model.complete(
            messages=[{'role': 'user', 'content': f'Plan tools for: {task["query"]}'}]
        )
        
        # Execute tools
        results = await self.execute_tools(tool_plan)
        
        # Smart model for final answer
        answer = await self.smart_model.complete(
            messages=[
                {'role': 'user', 'content': task['query']},
                {'role': 'assistant', 'content': str(results)}
            ]
        )
        
        return answer
```

### Selective SFT Export

```python
from axis_agentic.exporters import SFTExporter

exporter = SFTExporter(format='swift_agent')

# Export only high-quality traces
exporter.export_directory(
    traces_dir='outputs/traces',
    output_path='training_data.jsonl',
    filters={
        'status': 'success',
        'min_reward': 0.8,
        'max_turns': 30,
        'min_turns': 5,
        'evaluator_verdict': 'correct'
    }
)

# Add custom selection logic
def custom_filter(trace, metadata):
    return (
        metadata['status'] == 'success' and
        metadata['token_count'] < 50000 and
        'error' not in str(trace)
    )

exporter.export_directory(
    traces_dir='outputs/traces',
    output_path='filtered_data.jsonl',
    custom_filter=custom_filter
)
```

## Troubleshooting

### High Token Usage

```yaml
# Reduce context window
context:
  max_tokens: 50000
  compaction_threshold: 40000
  compaction_strategy: truncate

# Limit tool results
tools:
  - type: web_scraper
    max_content_length: 5000
```

### Slow Execution

```python
# Use async execution for parallel tools
async def execute_tools_parallel(self, tool_calls):
    tasks = [self.execute_tool(tc) for tc in tool_calls]
    results = await asyncio.gather(*tasks)
    return results
```

### Memory Issues with Large Datasets

```bash
# Process in batches
python -m recipe.web_search.runners.run_eval_config \
  --config config.yaml \
  --batch-size 50 \
  --checkpoint-interval 10
```

### Trace Replay Mismatch

```python
# Verify visibility markers
from axis_agentic.replay import verify_visibility

with open('trace.json') as f:
    trace = json.load(f)

issues = verify_visibility(trace)
if issues:
    print(f"Visibility issues: {issues}")
```

### Model Client Errors

```python
# Add retry logic
from axis_agentic.clients import create_model_client

client = create_model_client({
    'provider': 'openai',
    'model': 'gpt-4o',
    'max_retries': 5,
    'retry_delay': 2.0,
    'timeout': 60
})
```

### Missing Environment Variables

```bash
# Verify all required variables
python -m axis_agentic.tools.check_env

# Expected output lists missing variables
```

## References

- [Full Documentation](https://github.com/XYZ-AI-Lab/AxisAgentic/tree/main/docs)
- [Recipe Examples](https://github.com/XYZ-AI-Lab/AxisAgentic/tree/main/recipe)
- [Technical Report](https://xyz-lab.ai/blogs/ai4ai-at-scale/)
- [Contributing Guide](https://github.com/XYZ-AI-Lab/AxisAgentic/blob/main/CONTRIBUTING.md)
