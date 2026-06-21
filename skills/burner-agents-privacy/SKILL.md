---
name: burner-agents-privacy
description: Deploy disposable AI agents for unattributable web interaction with automatic identity destruction
triggers:
  - create a burner agent to browse anonymously
  - set up disposable agents for web automation
  - use burner agents for privacy-preserving web interaction
  - configure swarm of unattributable agents
  - automate web tasks with disposable identities
  - deploy privacy agents that destroy themselves after completion
  - set up isolated browser contexts for each agent
  - orchestrate multiple burner agents for a task
---

# Burner Agents Privacy Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

Burner Agents is a Python framework for deploying disposable AI agents that interact with the web on your behalf. Each agent runs in an isolated browser context with a unique fingerprint and is destroyed immediately after completing its task, preventing tracking and profile building. The system provides non-attribution, non-linkability, and ensures no persistent profile is created.

## Installation

```bash
# Clone the repository
git clone https://github.com/NotPBShaw/burner-agents.git
cd burner-agents

# Install dependencies
pip install -r requirements.txt

# Or install via pip (if published)
pip install burner-agents
```

## Core Architecture

The project has four main components:

- **isolation/**: Fresh, separable browser context per agent
- **reasoning/**: Turns intent into actions, reasoning over each page live
- **orchestration/**: Decomposes tasks, fans across agents, reconciles results
- **identity/**: Instantiated on task start, destroyed on completion

## Basic Usage

### Single Agent Task

```python
from burner.agent import BurnerAgent
from burner.task import Task

# Create a task with natural language intent
task = Task(
    intent="Find the top 3 Python AI frameworks on GitHub",
    max_agents=1
)

# Deploy a burner agent
agent = BurnerAgent()
result = agent.execute(task)

print(result.data)
# Agent and all state destroyed automatically after completion
```

### Multi-Agent Swarm

```python
from burner.orchestration import Swarm
from burner.task import Task

# Define a complex task that benefits from multiple agents
task = Task(
    intent="Compare pricing across 5 cloud providers",
    max_agents=5,
    parallel=True
)

# Orchestrate across multiple disposable agents
swarm = Swarm(size=5)
results = swarm.execute(task)

# Each agent visited one provider with unique identity
for result in results:
    print(f"Provider: {result.provider}, Price: {result.price}")
    
# All agents destroyed, no persistent state retained
```

## Configuration

### Environment Variables

```bash
# Required: LLM provider for reasoning
export BURNER_LLM_PROVIDER="anthropic"
export ANTHROPIC_API_KEY="your-api-key"

# Optional: Browser configuration
export BURNER_HEADLESS=true
export BURNER_TIMEOUT=30000

# Optional: Network isolation
export BURNER_PROXY_ROTATION=true
export BURNER_PROXY_POOL_SIZE=10

# Optional: Identity generation
export BURNER_FINGERPRINT_RANDOMIZATION=true
```

### Configuration File

```python
# config.py
from burner.config import BurnerConfig

config = BurnerConfig(
    # Isolation settings
    isolation={
        "browser": "chromium",  # chromium, firefox, webkit
        "headless": True,
        "separate_contexts": True,
        "clear_on_complete": True
    },
    
    # Reasoning engine
    reasoning={
        "provider": "anthropic",
        "model": "claude-3-5-sonnet-20241022",
        "max_steps": 50,
        "live_page_analysis": True
    },
    
    # Orchestration
    orchestration={
        "max_parallel_agents": 10,
        "task_decomposition": "auto",
        "result_reconciliation": True
    },
    
    # Identity management
    identity={
        "fingerprint_source": "random",
        "destroy_on_complete": True,
        "no_state_retention": True
    }
)
```

## Agent Isolation

### Creating Isolated Browser Contexts

```python
from burner.isolation import IsolatedContext
from burner.identity import generate_identity

# Generate unique identity for this agent
identity = generate_identity()

# Create isolated browser context
context = IsolatedContext(
    fingerprint=identity.fingerprint,
    user_agent=identity.user_agent,
    viewport=identity.viewport,
    timezone=identity.timezone,
    locale=identity.locale,
    device_characteristics=identity.device
)

# Use context for browsing
async with context.browser() as browser:
    page = await browser.new_page()
    await page.goto("https://example.com")
    
    # Perform actions...
    
# Context destroyed automatically on exit
```

### Custom Fingerprinting

```python
from burner.identity import Fingerprint

# Create custom fingerprint
fingerprint = Fingerprint(
    canvas_noise=True,
    webgl_vendor="random",
    audio_context_noise=True,
    client_rects_noise=True,
    screen_resolution=(1920, 1080),
    color_depth=24,
    hardware_concurrency=8,
    device_memory=8
)

context = IsolatedContext(fingerprint=fingerprint)
```

## Reasoning Engine

### Natural Language to Actions

```python
from burner.reasoning import ReasoningEngine
from burner.isolation import IsolatedContext

# Initialize reasoning engine
engine = ReasoningEngine(
    provider="anthropic",
    model="claude-3-5-sonnet-20241022"
)

# Execute intent with live page reasoning
async with IsolatedContext() as context:
    page = await context.new_page()
    
    result = await engine.reason_and_act(
        page=page,
        intent="Navigate to Hacker News and find the top story about AI",
        max_steps=20
    )
    
    print(result.final_answer)
    print(f"Steps taken: {len(result.steps)}")
```

### Step-by-Step Reasoning

```python
from burner.reasoning import Step

# Manual step control for complex workflows
async with IsolatedContext() as context:
    page = await context.new_page()
    
    steps = []
    current_intent = "Find pricing information"
    
    while not engine.is_complete(steps, current_intent):
        # Analyze current page state
        analysis = await engine.analyze_page(page)
        
        # Decide next action
        action = await engine.decide_action(
            page_state=analysis,
            intent=current_intent,
            history=steps
        )
        
        # Execute action
        result = await engine.execute_action(page, action)
        steps.append(Step(action=action, result=result))
        
    final_result = engine.synthesize(steps)
```

## Orchestration Patterns

### Task Decomposition

```python
from burner.orchestration import TaskDecomposer, Swarm

# Decompose complex task into subtasks
decomposer = TaskDecomposer()
subtasks = decomposer.decompose(
    intent="Research and compare 10 project management tools",
    max_agents=10
)

# Each subtask assigned to separate agent
swarm = Swarm(size=len(subtasks))
results = await swarm.execute_parallel(subtasks)

# Reconcile results
final_report = decomposer.reconcile(results)
```

### Fan-Out Pattern

```python
from burner.orchestration import FanOut

# Fan out single task to multiple agents for redundancy
fanout = FanOut(
    task=Task(intent="Check if service is available"),
    agent_count=3,
    reconciliation_strategy="majority"
)

results = await fanout.execute()

# Returns reconciled result based on majority agreement
print(f"Service available: {results.consensus}")
print(f"Agreement: {results.agreement_percentage}%")
```

### Sequential Pipeline

```python
from burner.orchestration import Pipeline

# Create pipeline where each agent's output feeds the next
pipeline = Pipeline([
    Task(intent="Find the top AI research papers this week"),
    Task(intent="Summarize the key findings from these papers"),
    Task(intent="Identify practical applications")
])

# Each stage uses a fresh agent
result = await pipeline.execute()
print(result.final_output)
```

## Identity Management

### Identity Lifecycle

```python
from burner.identity import IdentityManager

# Create identity manager
manager = IdentityManager()

# Generate identity for task
identity = manager.create()
print(f"Identity ID: {identity.id}")
print(f"Created: {identity.created_at}")

# Use identity
agent = BurnerAgent(identity=identity)
result = await agent.execute(task)

# Destroy identity (automatic on task completion)
manager.destroy(identity.id)

# Verify destruction
assert manager.get(identity.id) is None
```

### Custom Identity Attributes

```python
from burner.identity import Identity, DeviceProfile

# Create identity with specific characteristics
device = DeviceProfile(
    device_type="desktop",
    os="macos",
    browser="chrome",
    version="120.0.0.0"
)

identity = Identity(
    device=device,
    location="US-CA-SanFrancisco",
    language="en-US",
    connection_type="wifi"
)

agent = BurnerAgent(identity=identity)
```

## Complete Example: Privacy-Preserving Research

```python
import asyncio
from burner.agent import BurnerAgent
from burner.task import Task
from burner.orchestration import Swarm
from burner.config import BurnerConfig

async def privacy_research():
    # Configure for maximum privacy
    config = BurnerConfig(
        isolation={"headless": True, "separate_contexts": True},
        identity={"destroy_on_complete": True, "no_state_retention": True}
    )
    
    # Define research task
    task = Task(
        intent="""
        Research the top 5 privacy-focused email providers.
        For each: pricing, features, jurisdiction, encryption standards.
        """,
        max_agents=5
    )
    
    # Deploy swarm
    swarm = Swarm(size=5, config=config)
    results = await swarm.execute(task)
    
    # Results are collected; all agents destroyed
    for i, result in enumerate(results, 1):
        print(f"\nProvider {i}:")
        print(f"  Name: {result.provider_name}")
        print(f"  Pricing: {result.pricing}")
        print(f"  Jurisdiction: {result.jurisdiction}")
        print(f"  Encryption: {result.encryption}")
    
    # Verify no state retained
    assert swarm.active_agents() == 0
    assert swarm.retained_state() is None

if __name__ == "__main__":
    asyncio.run(privacy_research())
```

## Advanced Patterns

### Rate Limiting & Throttling

```python
from burner.orchestration import ThrottledSwarm
import asyncio

# Prevent detection through timing analysis
swarm = ThrottledSwarm(
    size=10,
    requests_per_minute=30,
    jitter=True,  # Random delays between requests
    delay_range=(2, 8)  # Seconds between actions
)

results = await swarm.execute(task)
```

### Rotating Proxy Integration

```python
from burner.isolation import ProxyRotator

# Rotate network egress for each agent
rotator = ProxyRotator(
    proxy_list=[
        "http://proxy1.example.com:8080",
        "http://proxy2.example.com:8080",
        "http://proxy3.example.com:8080"
    ],
    rotation_strategy="random"  # or "round-robin", "least-used"
)

context = IsolatedContext(proxy_rotator=rotator)
```

### Result Validation

```python
from burner.orchestration import Validator

# Validate results across multiple agents
validator = Validator(
    consensus_threshold=0.7,  # 70% agreement required
    min_agents=3
)

# Deploy multiple agents for same task
swarm = Swarm(size=5)
raw_results = await swarm.execute(task)

# Validate and reconcile
validated_result = validator.validate(raw_results)

if validated_result.consensus_reached:
    print(f"Validated result: {validated_result.data}")
else:
    print(f"No consensus. Confidence: {validated_result.confidence}")
```

## Troubleshooting

### Agent Fails to Complete

```python
from burner.agent import BurnerAgent
from burner.exceptions import AgentTimeoutError, ReasoningError

try:
    agent = BurnerAgent(timeout=60000)  # 60 seconds
    result = await agent.execute(task)
except AgentTimeoutError as e:
    print(f"Agent timed out after {e.elapsed}ms")
    print(f"Last action: {e.last_action}")
except ReasoningError as e:
    print(f"Reasoning failed: {e.message}")
    print(f"Page state: {e.page_state}")
```

### Browser Context Not Isolated

```python
# Verify isolation
from burner.isolation import verify_isolation

context1 = IsolatedContext()
context2 = IsolatedContext()

# Check fingerprints are different
assert context1.fingerprint != context2.fingerprint

# Check no shared cookies/storage
isolation_report = verify_isolation([context1, context2])
print(f"Fully isolated: {isolation_report.is_isolated}")
print(f"Shared elements: {isolation_report.shared_elements}")
```

### Identity Not Destroyed

```python
from burner.identity import IdentityManager

manager = IdentityManager(strict_destruction=True)

identity = manager.create()
agent = BurnerAgent(identity=identity)

# Ensure destruction even on error
try:
    result = await agent.execute(task)
finally:
    manager.destroy(identity.id, verify=True)
    
# Verify no remnants
assert manager.get(identity.id) is None
assert identity.has_remnants() is False
```

### High Memory Usage with Many Agents

```python
from burner.orchestration import ResourceConstrainedSwarm

# Limit concurrent agents
swarm = ResourceConstrainedSwarm(
    max_size=20,
    max_concurrent=5,  # Only 5 running at once
    memory_limit_mb=2048
)

# Agents queued and executed in batches
results = await swarm.execute(task)
```

## Best Practices

1. **Always destroy identities**: Use context managers or try/finally blocks
2. **Validate agent isolation**: Verify fingerprints differ between agents
3. **Use environment variables**: Never hardcode API keys or secrets
4. **Monitor resource usage**: Limit concurrent agents based on system capacity
5. **Enable result validation**: Use multiple agents for critical tasks
6. **Implement rate limiting**: Avoid detection through timing analysis
7. **Rotate network egress**: Use proxy rotation for additional anonymity
8. **Check destruction**: Verify no state retained after task completion

## Legal & Compliance Notice

This tool is for privacy-preserving web interaction. You are responsible for:
- Complying with websites' terms of service
- Following applicable laws in your jurisdiction
- Using the tool ethically and responsibly

The framework provides technical capabilities; legal compliance is the user's responsibility.
