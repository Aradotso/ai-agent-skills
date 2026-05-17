---
name: hyperagents-self-improving-ai
description: Self-referential self-improving AI agents that optimize for any computable task using meta-learning and code generation
triggers:
  - how do I use HyperAgents for self-improving AI
  - set up HyperAgents meta-agent system
  - run HyperAgents optimization loop
  - create custom domain for HyperAgents
  - configure HyperAgents task and meta agents
  - implement HyperAgents self-improvement
  - debug HyperAgents generated code
  - extend HyperAgents with new tasks
---

# HyperAgents Self-Improving AI Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

HyperAgents is a framework for building self-referential self-improving AI agents that can optimize for any computable task. The system uses a meta-agent to iteratively improve a task-agent by generating and evaluating code modifications. The framework supports multiple domains (code generation, reasoning, math, etc.) and uses foundation models to drive the self-improvement loop.

**Key Capabilities:**
- Self-referential meta-learning where agents modify their own code
- Multi-domain support (code, math, reasoning tasks)
- Iterative improvement through generation-evaluation loops
- Integration with OpenAI, Anthropic, and Google Gemini models
- Docker-based safe execution environment

## Installation

### Prerequisites

```bash
# Install system dependencies (Fedora/RHEL)
sudo dnf install -y python3.12-devel graphviz graphviz-devel cmake ninja-build bzip2-devel zlib-devel ncurses-devel libffi-devel

# For Ubuntu/Debian:
# sudo apt-get install -y python3.12-dev graphviz libgraphviz-dev cmake ninja-build libbz2-dev zlib1g-dev libncurses-dev libffi-dev
```

### Setup

```bash
# Clone the repository
git clone https://github.com/facebookresearch/HyperAgents.git
cd HyperAgents

# Create virtual environment
python3.12 -m venv venv_nat
source venv_nat/bin/activate

# Install dependencies
pip install -r requirements.txt
pip install -r requirements_dev.txt

# Build Docker container for safe execution
docker build --network=host -t hyperagents .
```

### Environment Configuration

Create a `.env` file with your API keys:

```bash
# .env file
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here
GEMINI_API_KEY=your_gemini_key_here
```

### Initialize Agents

```bash
# Setup initial agent implementations
bash ./setup_initial.sh
```

## Core Concepts

### Architecture

1. **Task Agent**: Solves domain-specific tasks (code generation, math, etc.)
2. **Meta Agent**: Observes task agent performance and generates improvements
3. **Generation Loop**: Iteratively evolves agents through self-improvement cycles

### File Structure

```
HyperAgents/
├── agent/              # Foundation model interfaces
├── domains/            # Task-specific implementations
├── utils/              # Common utilities
├── meta_agent.py       # Meta-agent implementation
├── task_agent.py       # Task-agent implementation
├── generate_loop.py    # Main entry point
└── run_meta_agent.py   # Meta-agent execution script
```

## Usage

### Running the Self-Improvement Loop

```bash
# Basic usage with default settings
python generate_loop.py --domains code_generation

# Multiple domains
python generate_loop.py --domains math reasoning

# Custom configuration
python generate_loop.py \
    --domains code_generation \
    --max_iterations 10 \
    --output_dir ./my_outputs \
    --model_name gpt-4
```

### Key Command-Line Arguments

```python
# Common arguments for generate_loop.py
--domains           # Domain(s) to optimize (code_generation, math, reasoning, etc.)
--max_iterations    # Maximum improvement iterations
--output_dir        # Directory for outputs (default: outputs/)
--model_name        # Foundation model to use
--baseline          # Baseline agent to compare against
--temperature       # Sampling temperature for generation
--num_samples       # Number of samples per iteration
```

## Working with Task Agents

### Creating a Custom Task Agent

```python
# task_agent.py - Basic structure
from typing import Any, Dict, List
from agent.base_agent import BaseAgent

class MyTaskAgent(BaseAgent):
    """Custom task agent for specific domain."""
    
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.domain = config.get('domain', 'custom')
        
    def solve_task(self, task_input: str) -> str:
        """
        Main method to solve a task.
        
        Args:
            task_input: Input task specification
            
        Returns:
            Solution to the task
        """
        # Generate prompt for the model
        prompt = self._create_prompt(task_input)
        
        # Get model response
        response = self.model.generate(
            prompt=prompt,
            temperature=self.config.get('temperature', 0.7),
            max_tokens=self.config.get('max_tokens', 2048)
        )
        
        # Post-process response
        solution = self._parse_solution(response)
        return solution
    
    def _create_prompt(self, task_input: str) -> str:
        """Create prompt for the model."""
        return f"""Solve the following task:

Task: {task_input}

Solution:"""
    
    def _parse_solution(self, response: str) -> str:
        """Extract solution from model response."""
        # Custom parsing logic
        return response.strip()
    
    def evaluate(self, task_input: str, solution: str) -> float:
        """
        Evaluate solution quality.
        
        Returns:
            Score between 0 and 1
        """
        # Domain-specific evaluation
        return self._compute_score(task_input, solution)
```

### Using the Task Agent

```python
from task_agent import MyTaskAgent

# Initialize agent
config = {
    'domain': 'custom',
    'model_name': 'gpt-4',
    'temperature': 0.7,
    'max_tokens': 2048
}

agent = MyTaskAgent(config)

# Solve a task
task = "Write a function to compute Fibonacci numbers"
solution = agent.solve_task(task)
score = agent.evaluate(task, solution)

print(f"Solution: {solution}")
print(f"Score: {score}")
```

## Working with Meta Agents

### Meta Agent Structure

```python
# meta_agent.py - Core implementation
from typing import Dict, List, Any
import difflib

class MetaAgent:
    """Meta-agent that improves task agents."""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.model = self._initialize_model()
        self.history = []
        
    def generate_improvement(
        self,
        current_code: str,
        performance_data: List[Dict[str, Any]]
    ) -> str:
        """
        Generate improved version of task agent.
        
        Args:
            current_code: Current task agent implementation
            performance_data: Performance metrics from recent runs
            
        Returns:
            Improved code implementation
        """
        # Analyze performance
        insights = self._analyze_performance(performance_data)
        
        # Generate improvement prompt
        prompt = self._create_meta_prompt(current_code, insights)
        
        # Generate new code
        improved_code = self.model.generate(
            prompt=prompt,
            temperature=self.config.get('meta_temperature', 0.8),
            max_tokens=self.config.get('meta_max_tokens', 4096)
        )
        
        # Validate and extract code
        validated_code = self._validate_code(improved_code)
        
        # Store in history
        self.history.append({
            'original': current_code,
            'improved': validated_code,
            'insights': insights
        })
        
        return validated_code
    
    def _analyze_performance(
        self,
        performance_data: List[Dict[str, Any]]
    ) -> Dict[str, Any]:
        """Analyze performance metrics to identify improvement areas."""
        # Compute statistics
        scores = [d['score'] for d in performance_data]
        avg_score = sum(scores) / len(scores)
        
        # Identify failure patterns
        failures = [d for d in performance_data if d['score'] < 0.5]
        
        return {
            'average_score': avg_score,
            'num_failures': len(failures),
            'failure_patterns': self._extract_patterns(failures)
        }
    
    def _create_meta_prompt(
        self,
        current_code: str,
        insights: Dict[str, Any]
    ) -> str:
        """Create prompt for meta-level improvement."""
        return f"""You are a meta-agent tasked with improving an AI task agent.

Current Implementation:
```python
{current_code}
```

Performance Analysis:
- Average Score: {insights['average_score']:.2f}
- Failures: {insights['num_failures']}
- Common Issues: {insights.get('failure_patterns', 'None identified')}

Generate an improved version that addresses these issues.
Output only the complete improved code.

Improved Implementation:
```python"""
    
    def _validate_code(self, code: str) -> str:
        """Validate and extract code from response."""
        # Extract code block
        if '```python' in code:
            code = code.split('```python')[1].split('```')[0]
        
        # Basic syntax validation
        try:
            compile(code, '<string>', 'exec')
        except SyntaxError as e:
            raise ValueError(f"Generated code has syntax error: {e}")
        
        return code.strip()
    
    def compute_diff(self, old_code: str, new_code: str) -> List[str]:
        """Compute diff between code versions."""
        diff = difflib.unified_diff(
            old_code.splitlines(keepends=True),
            new_code.splitlines(keepends=True),
            fromfile='old_agent.py',
            tofile='new_agent.py'
        )
        return list(diff)
```

### Running Meta Agent

```python
# run_meta_agent.py - Example usage
from meta_agent import MetaAgent
from task_agent import MyTaskAgent
import json

def run_meta_improvement_cycle(
    initial_agent_code: str,
    test_tasks: List[str],
    num_iterations: int = 5
):
    """Run multiple iterations of meta-improvement."""
    
    # Initialize meta-agent
    meta_config = {
        'model_name': 'gpt-4',
        'meta_temperature': 0.8,
        'meta_max_tokens': 4096
    }
    meta_agent = MetaAgent(meta_config)
    
    current_code = initial_agent_code
    
    for iteration in range(num_iterations):
        print(f"\n=== Iteration {iteration + 1} ===")
        
        # Evaluate current agent
        performance_data = evaluate_agent(current_code, test_tasks)
        
        avg_score = sum(d['score'] for d in performance_data) / len(performance_data)
        print(f"Current Performance: {avg_score:.3f}")
        
        # Generate improvement
        improved_code = meta_agent.generate_improvement(
            current_code,
            performance_data
        )
        
        # Show diff
        diff = meta_agent.compute_diff(current_code, improved_code)
        print("Changes:")
        print(''.join(diff[:20]))  # Show first 20 lines
        
        # Update current code
        current_code = improved_code
        
        # Save checkpoint
        with open(f'agent_iteration_{iteration}.py', 'w') as f:
            f.write(current_code)
    
    return current_code

def evaluate_agent(agent_code: str, test_tasks: List[str]) -> List[Dict[str, Any]]:
    """Evaluate agent on test tasks."""
    # Create agent from code
    namespace = {}
    exec(agent_code, namespace)
    AgentClass = namespace['MyTaskAgent']
    
    agent = AgentClass({'model_name': 'gpt-4'})
    
    results = []
    for task in test_tasks:
        solution = agent.solve_task(task)
        score = agent.evaluate(task, solution)
        results.append({
            'task': task,
            'solution': solution,
            'score': score
        })
    
    return results

# Usage
if __name__ == '__main__':
    # Read initial agent code
    with open('initial_agent.py', 'r') as f:
        initial_code = f.read()
    
    # Define test tasks
    test_tasks = [
        "Implement binary search",
        "Write a function to reverse a linked list",
        "Create a trie data structure"
    ]
    
    # Run improvement loop
    final_code = run_meta_improvement_cycle(
        initial_code,
        test_tasks,
        num_iterations=5
    )
    
    print("\nFinal agent saved!")
```

## Domain-Specific Implementation

### Code Generation Domain

```python
# domains/code_generation/agent.py
from typing import Dict, Any, List
import ast
import subprocess

class CodeGenerationAgent:
    """Agent specialized for code generation tasks."""
    
    def generate_code(self, specification: str) -> str:
        """Generate code from specification."""
        prompt = f"""Generate Python code for the following specification:

{specification}

Requirements:
- Include proper error handling
- Add docstrings
- Follow PEP 8 style guide

Code:
```python"""
        
        code = self.model.generate(prompt)
        return self._extract_code(code)
    
    def test_code(self, code: str, test_cases: List[Dict[str, Any]]) -> float:
        """Test generated code against test cases."""
        try:
            # Create temporary module
            namespace = {}
            exec(code, namespace)
            
            passed = 0
            for test in test_cases:
                func_name = test['function']
                inputs = test['inputs']
                expected = test['expected']
                
                func = namespace[func_name]
                result = func(*inputs)
                
                if result == expected:
                    passed += 1
            
            return passed / len(test_cases)
            
        except Exception as e:
            print(f"Test error: {e}")
            return 0.0
    
    def _extract_code(self, response: str) -> str:
        """Extract code from model response."""
        if '```python' in response:
            code = response.split('```python')[1].split('```')[0]
        else:
            code = response
        
        # Validate syntax
        try:
            ast.parse(code)
        except SyntaxError:
            raise ValueError("Generated code has syntax errors")
        
        return code.strip()
```

### Math Reasoning Domain

```python
# domains/math/agent.py
import re
from typing import Optional

class MathReasoningAgent:
    """Agent for mathematical reasoning tasks."""
    
    def solve_math_problem(self, problem: str) -> Dict[str, Any]:
        """Solve a math problem with step-by-step reasoning."""
        prompt = f"""Solve the following math problem step by step:

Problem: {problem}

Show your work clearly. Format your final answer as: ANSWER: <value>

Solution:"""
        
        response = self.model.generate(prompt)
        
        return {
            'reasoning': response,
            'answer': self._extract_answer(response)
        }
    
    def _extract_answer(self, response: str) -> Optional[str]:
        """Extract final answer from reasoning."""
        # Look for ANSWER: pattern
        match = re.search(r'ANSWER:\s*([^\n]+)', response, re.IGNORECASE)
        if match:
            return match.group(1).strip()
        
        # Look for boxed answer (LaTeX)
        match = re.search(r'\\boxed\{([^}]+)\}', response)
        if match:
            return match.group(1).strip()
        
        # Try to find number at end
        numbers = re.findall(r'-?\d+\.?\d*', response)
        if numbers:
            return numbers[-1]
        
        return None
    
    def evaluate_answer(
        self,
        predicted: str,
        ground_truth: str,
        tolerance: float = 1e-5
    ) -> bool:
        """Evaluate if answer is correct."""
        try:
            pred_val = float(predicted)
            true_val = float(ground_truth)
            return abs(pred_val - true_val) < tolerance
        except (ValueError, TypeError):
            # Fallback to string comparison
            return predicted.strip() == ground_truth.strip()
```

## Configuration Patterns

### Agent Configuration

```python
# config.py - Common configuration patterns
from dataclasses import dataclass
from typing import Optional

@dataclass
class AgentConfig:
    """Configuration for task agents."""
    model_name: str = 'gpt-4'
    temperature: float = 0.7
    max_tokens: int = 2048
    top_p: float = 1.0
    frequency_penalty: float = 0.0
    presence_penalty: float = 0.0
    timeout: int = 60
    max_retries: int = 3

@dataclass
class MetaAgentConfig:
    """Configuration for meta-agents."""
    model_name: str = 'gpt-4'
    meta_temperature: float = 0.8
    meta_max_tokens: int = 4096
    improvement_iterations: int = 5
    min_improvement_threshold: float = 0.05
    use_reflection: bool = True

@dataclass
class ExperimentConfig:
    """Configuration for experiments."""
    domain: str
    num_iterations: int = 10
    num_eval_samples: int = 100
    output_dir: str = './outputs'
    save_checkpoints: bool = True
    checkpoint_interval: int = 1
    seed: Optional[int] = None
    
# Usage
agent_config = AgentConfig(
    model_name='gpt-4-turbo',
    temperature=0.5,
    max_tokens=4096
)

meta_config = MetaAgentConfig(
    improvement_iterations=10,
    min_improvement_threshold=0.02
)
```

### Loading Models

```python
# agent/base_agent.py - Model initialization
from typing import Dict, Any
import os
from dotenv import load_dotenv

class BaseAgent:
    """Base class for all agents."""
    
    def __init__(self, config: Dict[str, Any]):
        load_dotenv()
        self.config = config
        self.model = self._initialize_model()
    
    def _initialize_model(self):
        """Initialize the foundation model."""
        model_name = self.config.get('model_name', 'gpt-4')
        
        if 'gpt' in model_name.lower():
            from openai import OpenAI
            api_key = os.getenv('OPENAI_API_KEY')
            client = OpenAI(api_key=api_key)
            return OpenAIModel(client, model_name)
            
        elif 'claude' in model_name.lower():
            from anthropic import Anthropic
            api_key = os.getenv('ANTHROPIC_API_KEY')
            client = Anthropic(api_key=api_key)
            return AnthropicModel(client, model_name)
            
        elif 'gemini' in model_name.lower():
            import google.generativeai as genai
            api_key = os.getenv('GEMINI_API_KEY')
            genai.configure(api_key=api_key)
            return GeminiModel(model_name)
        
        else:
            raise ValueError(f"Unsupported model: {model_name}")

class OpenAIModel:
    """Wrapper for OpenAI models."""
    
    def __init__(self, client, model_name: str):
        self.client = client
        self.model_name = model_name
    
    def generate(
        self,
        prompt: str,
        temperature: float = 0.7,
        max_tokens: int = 2048,
        **kwargs
    ) -> str:
        """Generate response from OpenAI model."""
        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=[{"role": "user", "content": prompt}],
            temperature=temperature,
            max_tokens=max_tokens,
            **kwargs
        )
        return response.choices[0].message.content
```

## Advanced Patterns

### Batched Evaluation

```python
# utils/evaluation.py
from typing import List, Dict, Any
from concurrent.futures import ThreadPoolExecutor, as_completed
import numpy as np

class BatchEvaluator:
    """Efficiently evaluate agents on multiple tasks."""
    
    def __init__(self, max_workers: int = 10):
        self.max_workers = max_workers
    
    def evaluate_batch(
        self,
        agent,
        tasks: List[str],
        ground_truths: List[Any]
    ) -> Dict[str, Any]:
        """Evaluate agent on batch of tasks in parallel."""
        results = []
        
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            futures = {
                executor.submit(self._evaluate_single, agent, task, truth): idx
                for idx, (task, truth) in enumerate(zip(tasks, ground_truths))
            }
            
            for future in as_completed(futures):
                idx = futures[future]
                try:
                    result = future.result()
                    results.append((idx, result))
                except Exception as e:
                    print(f"Task {idx} failed: {e}")
                    results.append((idx, {'score': 0.0, 'error': str(e)}))
        
        # Sort by original order
        results.sort(key=lambda x: x[0])
        results = [r[1] for r in results]
        
        return {
            'results': results,
            'mean_score': np.mean([r['score'] for r in results]),
            'std_score': np.std([r['score'] for r in results]),
            'success_rate': sum(1 for r in results if r['score'] > 0.5) / len(results)
        }
    
    def _evaluate_single(self, agent, task: str, ground_truth: Any) -> Dict[str, Any]:
        """Evaluate single task."""
        solution = agent.solve_task(task)
        score = agent.evaluate(task, solution, ground_truth)
        
        return {
            'task': task,
            'solution': solution,
            'score': score,
            'correct': score > 0.5
        }
```

### Checkpointing

```python
# utils/checkpointing.py
import json
import pickle
from pathlib import Path
from typing import Any, Dict

class CheckpointManager:
    """Manage experiment checkpoints."""
    
    def __init__(self, output_dir: str):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
    
    def save_checkpoint(
        self,
        iteration: int,
        agent_code: str,
        performance_data: Dict[str, Any],
        meta_data: Dict[str, Any]
    ):
        """Save checkpoint for an iteration."""
        checkpoint_dir = self.output_dir / f'iteration_{iteration}'
        checkpoint_dir.mkdir(exist_ok=True)
        
        # Save agent code
        with open(checkpoint_dir / 'agent.py', 'w') as f:
            f.write(agent_code)
        
        # Save performance data
        with open(checkpoint_dir / 'performance.json', 'w') as f:
            json.dump(performance_data, f, indent=2)
        
        # Save metadata
        with open(checkpoint_dir / 'metadata.pkl', 'wb') as f:
            pickle.dump(meta_data, f)
        
        print(f"Checkpoint saved to {checkpoint_dir}")
    
    def load_checkpoint(self, iteration: int) -> Dict[str, Any]:
        """Load checkpoint from an iteration."""
        checkpoint_dir = self.output_dir / f'iteration_{iteration}'
        
        with open(checkpoint_dir / 'agent.py', 'r') as f:
            agent_code = f.read()
        
        with open(checkpoint_dir / 'performance.json', 'r') as f:
            performance_data = json.load(f)
        
        with open(checkpoint_dir / 'metadata.pkl', 'rb') as f:
            meta_data = pickle.load(f)
        
        return {
            'agent_code': agent_code,
            'performance_data': performance_data,
            'meta_data': meta_data
        }
    
    def list_checkpoints(self) -> List[int]:
        """List available checkpoints."""
        iterations = []
        for path in self.output_dir.glob('iteration_*'):
            if path.is_dir():
                iteration = int(path.name.split('_')[1])
                iterations.append(iteration)
        return sorted(iterations)
```

## Troubleshooting

### Common Issues

**1. API Key Errors**
```python
# Verify environment variables
import os
from dotenv import load_dotenv

load_dotenv()

required_keys = ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'GEMINI_API_KEY']
for key in required_keys:
    value = os.getenv(key)
    if value:
        print(f"{key}: {'*' * 20} (set)")
    else:
        print(f"{key}: NOT SET")
```

**2. Docker Execution Errors**
```bash
# Verify Docker is running
docker ps

# Rebuild container if needed
docker build --no-cache --network=host -t hyperagents .

# Check container logs
docker logs <container_id>
```

**3. Code Generation Syntax Errors**
```python
# Add validation wrapper
import ast

def validate_generated_code(code: str) -> bool:
    """Validate Python syntax before execution."""
    try:
        ast.parse(code)
        return True
    except SyntaxError as e:
        print(f"Syntax error at line {e.lineno}: {e.msg}")
        print(f"Text: {e.text}")
        return False

# Use in meta-agent
if validate_generated_code(improved_code):
    current_code = improved_code
else:
    print("Generated code has errors, keeping current version")
```

**4. Performance Degradation**
```python
# Track performance over iterations
def monitor_performance(history: List[Dict[str, float]]):
    """Monitor for performance degradation."""
    if len(history) < 3:
        return
    
    recent_scores = [h['score'] for h in history[-3:]]
    if all(recent_scores[i] < recent_scores[i-1] for i in range(1, len(recent_scores))):
        print("WARNING: Performance degrading for 3 consecutive iterations")
        print("Consider:")
        print("  - Reducing temperature")
        print("  - Changing meta-agent prompt")
        print("  - Rolling back to earlier checkpoint")
```

**5. Memory Issues with Large Contexts**
```python
# Implement context truncation
def truncate_context(
    context: str,
    max_tokens: int = 8000,
    tokenizer=None
) -> str:
    """Truncate context to fit within token limit."""
    if tokenizer is None:
        # Rough approximation: 4 chars per token
        max_chars = max_tokens * 4
        if len(context) > max_chars:
            return context[:max_chars] + "\n... (truncated)"
    else:
        tokens = tokenizer.encode(context)
        if len(tokens) > max_tokens:
            truncated = tokenizer.decode(tokens[:max_tokens])
            return truncated + "\n... (truncated)"
    
    return context
```

### Debugging Tips

```python
# Enable verbose logging
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('hyperagents.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger('hyperagents')

# Use in code
logger.debug(f"Generating improvement for iteration {iteration}")
logger.info(f"Current score: {current_score:.3f}")
logger.warning(f"Low performance detected: {score:.3f}")
logger.error(f"Failed to generate valid code: {error}")
```

### Safety Checks

```python
# Implement safety checks before executing generated code
import re

def safety_check(code: str) -> Dict[str, bool]:
    """Check for potentially dangerous operations."""
    checks = {
        'no_file_deletion': 'os.remove' not in code and 'shutil.rmtree' not in code,
        'no_system_calls': 'os.system' not in code and 'subprocess.call' not in code,
        'no_network': 'requests.' not in code and 'urllib' not in code,
        'no_eval': 'eval(' not in code and 'exec(' not in code,
    }
    
    all_safe = all(checks.values())
    
    return {
        'safe': all_safe,
        'checks': checks
    }

# Use before execution
safety_result = safety_check(generated_code)
if not safety_result['safe']:
    print("WARNING: Potentially unsafe code detected!")
    print(f"Failed checks: {[k for k, v in safety_result['checks'].items() if not v]}")
    # Decide whether to proceed
```

## Best Practices

1. **Always use environment variables** for API keys, never hardcode
2. **Checkpoint frequently** to avoid losing progress
3. **Validate generated code** before execution
4. **Monitor performance** across iterations to detect degradation
5. **Use Docker containers** for safe code execution
6. **Implement timeouts** for long-running operations
7. **Log extensively** for debugging and analysis
8. **Test on small batches** before full-scale runs

## Resources

- [Official Repository](https://github.com/facebookresearch/HyperAgents)
- [Research Paper](https://arxiv.org/abs/2603.19461)
- [Blog Post](https://ai.meta.com/research/publications/hyperagents/)
- [Experiment Logs](https://drive.google.com/drive/folders/164fKQWgLM18foOzSnpv0F_I3TNpX8u8-?usp=sharing)
