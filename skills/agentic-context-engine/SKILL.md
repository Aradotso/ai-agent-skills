---
name: agentic-context-engine
description: Add persistent learning and self-improvement to AI agents using ACE framework
triggers:
  - make my agent learn from mistakes
  - add memory to my AI agent
  - implement agent learning loop
  - use ACE framework for agent improvement
  - create self-improving AI agent
  - add persistent context to my agent
  - build agent with skillbook memory
  - implement recursive reflection for agents
---

# Agentic Context Engine (ACE)

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

ACE is a framework that adds persistent learning capabilities to AI agents. Unlike traditional agents that forget everything between sessions, ACE maintains a **Skillbook** — a living collection of strategies extracted from execution traces. The framework uses a **Recursive Reflector** that writes and executes Python code to analyze traces and extract actionable patterns.

## Installation

```bash
# Basic installation
uv add ace-framework

# With optional integrations
uv add 'ace-framework[browser-use]'    # Browser automation
uv add 'ace-framework[langchain]'      # LangChain integration
uv add 'ace-framework[logfire]'        # Observability
uv add 'ace-framework[mcp]'            # MCP server for IDE
uv add 'ace-framework[deduplication]'  # Embedding-based deduplication
```

## Configuration

Interactive setup (recommended):
```bash
ace setup
```

Or set environment variables manually:
```bash
export OPENAI_API_KEY="your-key-here"
# OR
export ANTHROPIC_API_KEY="your-key-here"
# Supports 100+ providers via LiteLLM
```

## Core Concepts

**Skillbook**: Persistent collection of learned strategies  
**Agent**: Executes tasks enhanced with Skillbook strategies  
**Reflector**: Analyzes execution traces to extract insights  
**SkillManager**: Curates the Skillbook (adds, refines, removes strategies)

## Quick Start: LiteLLM Runner

The simplest way to add learning to any LLM:

```python
from ace import ACELiteLLM

# Initialize with any LiteLLM model
agent = ACELiteLLM(model="gpt-4o-mini")

# First attempt - may hallucinate
answer = agent.ask("Is there a seahorse emoji?")
print(answer)  # May incorrectly say yes

# Provide corrective feedback
agent.learn_from_feedback("There is no seahorse emoji in Unicode.")

# Second attempt - benefits from learned strategy
answer = agent.ask("Is there a seahorse emoji?")
print(answer)  # Now correctly says no

# Inspect what was learned
strategies = agent.get_strategies()
for strategy in strategies:
    print(f"Strategy: {strategy.name}")
    print(f"Content: {strategy.content}")

# Save skillbook for later
agent.save("my_skillbook.json")

# Load skillbook in new session
agent = ACELiteLLM(model="gpt-4o-mini", skillbook_path="my_skillbook.json")
```

## Learning from Existing Traces

Extract strategies from pre-recorded execution traces:

```python
from ace import ACELiteLLM

agent = ACELiteLLM(model="gpt-4o-mini")

# Your existing traces (list of conversation histories)
traces = [
    [
        {"role": "user", "content": "What's 2+2?"},
        {"role": "assistant", "content": "4"},
    ],
    [
        {"role": "user", "content": "What's 3+3?"},
        {"role": "assistant", "content": "The answer is 6"},
    ]
]

# Learn from traces without re-running tasks
agent.learn_from_traces(traces)

# View extracted strategies
print(agent.get_strategies())
```

## Core Runner: Full Learning Loop

For complete control with batch epochs and evaluation:

```python
from ace import ACE, Skillbook
from pydantic_ai.models.openai import OpenAIModel

# Create skillbook and agent
skillbook = Skillbook()
ace = ACE(
    model=OpenAIModel("gpt-4o-mini"),
    skillbook=skillbook,
    environment=your_env,  # Custom environment
    max_attempts=3
)

# Define your task
async def my_task():
    return "What is the capital of France?"

# Run learning epoch
result = await ace.run_epoch(
    task=my_task,
    task_id="geography_001",
    num_iterations=5
)

print(f"Success rate: {result.success_rate}")
print(f"Strategies learned: {len(skillbook.get_all_strategies())}")
```

## Custom Agent Integration

Wrap your existing agent with ACE learning:

```python
from ace import ACE, Skillbook
from pydantic_ai import Agent

# Your existing PydanticAI agent
my_agent = Agent(
    model="openai:gpt-4o",
    system_prompt="You are a helpful assistant."
)

# Wrap with ACE
skillbook = Skillbook()
ace_agent = ACE(
    agent=my_agent,
    skillbook=skillbook,
    environment=your_env
)

# Agent now learns from execution
result = await ace_agent.run_epoch(task=your_task)
```

## Browser Automation with Learning

ACE integrates with browser-use for self-improving browser automation:

```python
from ace.runners.browser_use import BrowserUse
from browser_use import Agent as BrowserAgent

# Create browser agent with learning
browser_ace = BrowserUse(
    agent=BrowserAgent(
        task="Find flights from NYC to LAX",
        llm=your_llm
    ),
    skillbook_path="browser_skills.json"
)

# Run with learning enabled
result = await browser_ace.run()

# Each run improves the agent
# Strategies are saved to browser_skills.json
```

## LangChain Integration

Add learning to any LangChain chain or agent:

```python
from ace.runners.langchain import LangChain
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

# Your existing LangChain setup
prompt = PromptTemplate.from_template("Translate {text} to {language}")
chain = LLMChain(llm=your_llm, prompt=prompt)

# Wrap with ACE
ace_chain = LangChain(
    chain=chain,
    skillbook_path="translation_skills.json"
)

# Run with learning
result = await ace_chain.run(
    text="Hello world",
    language="Spanish"
)
```

## Trace Analysis

Analyze traces without re-running tasks:

```python
from ace.runners.trace_analyser import TraceAnalyser

analyzer = TraceAnalyser(
    model="gpt-4o-mini",
    skillbook_path="analyzed_skills.json"
)

# Analyze a batch of traces
traces = load_your_traces()  # List of execution traces
strategies = await analyzer.analyze(traces)

print(f"Extracted {len(strategies)} strategies")
for s in strategies:
    print(f"- {s.name}: {s.content}")
```

## CLI Commands

```bash
# Interactive setup
ace setup

# Search available models
ace models gpt
ace models --provider anthropic

# Validate model connection
ace validate gpt-4o-mini
ace validate claude-3-5-sonnet-20241022

# Show configuration
ace config

# Hosted API (requires account at kayba.ai)
kayba login
kayba upload-traces traces.json
kayba fetch-insights
kayba install-prompt my-skill
```

## Custom Pipeline

Build custom learning pipelines with composable steps:

```python
from ace import Pipeline, AgentStep, EvaluateStep, learning_tail
from ace.core.reflector import Reflector
from ace.core.skill_manager import SkillManager

# Create pipeline components
agent_step = AgentStep(agent, skillbook)
eval_step = EvaluateStep(environment)

# Add standard learning tail (reflect + update + deduplicate)
reflector = Reflector(model="gpt-4o")
skill_manager = SkillManager(model="gpt-4o")

steps = [
    agent_step,
    eval_step
] + learning_tail(reflector, skill_manager, skillbook)

# Create and run pipeline
pipeline = Pipeline(steps)
context = await pipeline.run({"task": your_task})

print(context.get("strategies_learned"))
```

## Configuration Options

### Model Selection

ACE supports 100+ providers via LiteLLM:

```python
# OpenAI
ACELiteLLM(model="gpt-4o-mini")
ACELiteLLM(model="gpt-4o")

# Anthropic
ACELiteLLM(model="claude-3-5-sonnet-20241022")
ACELiteLLM(model="claude-3-5-haiku-20241022")

# Google
ACELiteLLM(model="gemini/gemini-2.0-flash-exp")

# Local models
ACELiteLLM(model="ollama/llama3")

# Any LiteLLM supported model
ACELiteLLM(model="bedrock/anthropic.claude-v2")
```

### Skillbook Persistence

```python
# Save skillbook
agent.save("path/to/skillbook.json")

# Load skillbook
agent = ACELiteLLM(
    model="gpt-4o-mini",
    skillbook_path="path/to/skillbook.json"
)

# Programmatic access
skillbook = Skillbook()
skillbook.load_from_file("skillbook.json")
strategies = skillbook.get_all_strategies()
skillbook.add_strategy(new_strategy)
skillbook.save_to_file("updated.json")
```

## Common Patterns

### Pattern 1: Iterative Improvement

```python
from ace import ACELiteLLM

agent = ACELiteLLM(model="gpt-4o-mini")

# Run task multiple times with feedback
for attempt in range(5):
    result = agent.ask("Complex reasoning task")
    
    # Provide feedback on errors
    if not validate(result):
        agent.learn_from_feedback(f"Error: {get_error(result)}")
    
    print(f"Attempt {attempt + 1}: {result}")

# Save learned strategies
agent.save("improved_agent.json")
```

### Pattern 2: Multi-Task Learning

```python
from ace import ACELiteLLM

agent = ACELiteLLM(model="gpt-4o-mini")

tasks = [
    "Task 1: Data analysis",
    "Task 2: Code generation", 
    "Task 3: Writing"
]

for task in tasks:
    result = agent.ask(task)
    # Agent accumulates strategies across tasks
    
# Single skillbook learns from diverse tasks
agent.save("multi_task_skills.json")
```

### Pattern 3: Batch Trace Analysis

```python
from ace import TraceAnalyser

# Load historical traces
traces = load_traces_from_database()

analyzer = TraceAnalyser(model="gpt-4o")

# Extract all strategies at once
strategies = await analyzer.analyze(traces)

# Use in new agent
agent = ACELiteLLM(
    model="gpt-4o-mini",
    skillbook=analyzer.skillbook
)
```

## Troubleshooting

### API Key Issues

```python
# Verify configuration
import subprocess
result = subprocess.run(["ace", "config"], capture_output=True, text=True)
print(result.stdout)

# Test connection
subprocess.run(["ace", "validate", "gpt-4o-mini"])
```

### Empty Skillbook

If no strategies are learned:

```python
# Check if reflector is enabled
agent = ACELiteLLM(
    model="gpt-4o-mini",
    enable_reflection=True  # Ensure this is True
)

# Provide explicit feedback
agent.learn_from_feedback("Specific error description")

# Verify strategies were added
print(len(agent.get_strategies()))
```

### Performance Issues

```python
# Use cheaper models for reflection
from ace import ACE, Skillbook
from pydantic_ai.models.openai import OpenAIModel

ace = ACE(
    model=OpenAIModel("gpt-4o"),  # Expensive for main task
    reflector_model=OpenAIModel("gpt-4o-mini"),  # Cheap for reflection
    skillbook=Skillbook()
)
```

### Trace Format Issues

Traces must be in chat format:

```python
# Correct format
valid_trace = [
    {"role": "user", "content": "Question"},
    {"role": "assistant", "content": "Answer"}
]

# Learn from properly formatted traces
agent.learn_from_traces([valid_trace])
```

## Advanced: Custom Reflector

Customize the reflection process:

```python
from ace.core.reflector import Reflector
from ace.core.skill_manager import SkillManager
from ace import Skillbook

# Custom reflector with specific system prompt
reflector = Reflector(
    model="gpt-4o",
    system_prompt="Focus on error patterns and edge cases"
)

skill_manager = SkillManager(model="gpt-4o-mini")
skillbook = Skillbook()

# Use in custom pipeline
from ace import Pipeline, learning_tail

pipeline = Pipeline(
    learning_tail(reflector, skill_manager, skillbook)
)
```

## Environment Variables

```bash
# Required (one of):
export OPENAI_API_KEY=sk-...
export ANTHROPIC_API_KEY=sk-ant-...
export GOOGLE_API_KEY=...

# Optional:
export ACE_DEFAULT_MODEL=gpt-4o-mini
export ACE_SKILLBOOK_PATH=/path/to/default.json
export ACE_LOG_LEVEL=INFO

# Hosted API (kayba.ai):
export KAYBA_API_KEY=kb-...
```

## Resources

- Documentation: https://kayba-ai.github.io/agentic-context-engine/latest/
- Repository: https://github.com/kayba-ai/agentic-context-engine
- Hosted Solution: https://kayba.ai
- Discord: https://discord.gg/mqCqH7sTyK
