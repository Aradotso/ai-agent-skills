---
name: ai-agent-deep-dive-teaching-framework
description: A minimal teaching framework for understanding AI Agent architecture with core loop, fake LLM interface, and skill discovery system
triggers:
  - how do I build a minimal AI agent in Python
  - show me the ai-agent-deep-dive teaching framework
  - how to structure an AI agent with skills
  - explain the agent main loop and LLM interface
  - how to use the agt teaching agent
  - demonstrate AI agent architecture patterns
  - build a simple agent with skill discovery
  - understand agent core structure with ai-agent-deep-dive
---

# ai-agent-deep-dive-teaching-framework

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

`ai-agent-deep-dive` is a teaching-focused Python project that demonstrates how to build a minimal AI Agent from scratch. It includes:

- A core Agent loop implementation
- A swappable LLM interface (currently using a fake LLM for teaching)
- A skill discovery and loading system
- CLI interface for interaction
- Educational documentation and reports on AI Agent architecture

The project is designed to keep complexity minimal while showing the essential components every AI Agent needs. It's ideal for learning agent architecture before building production systems.

## Installation

The project uses Poetry for dependency management.

```bash
# Clone the repository
git clone https://github.com/tvytlx/ai-agent-deep-dive.git
cd ai-agent-deep-dive

# Install dependencies
poetry install

# Verify installation
poetry run agt --help
```

## Core Architecture

The teaching agent consists of three main components:

1. **Agent Core** (`src/agt/agent.py`) - Main loop and execution logic
2. **CLI Interface** (`src/agt/cli.py`) - Command-line entry point
3. **Skills System** - Pluggable capabilities loaded from a skills directory

## CLI Commands

### Basic Usage

```bash
# Run agent with a simple query
poetry run agt "hello world"

# List available skills
poetry run agt --skills-dir ./skills --list-skills

# Specify custom skills directory
poetry run agt --skills-dir /path/to/skills "your query"
```

### CLI Options

- `query` - The input query for the agent (positional argument)
- `--skills-dir` - Path to directory containing skill modules (default: `./skills`)
- `--list-skills` - Display all discovered skills and exit

## Code Examples

### Basic Agent Usage

```python
from agt.agent import Agent

# Initialize agent with default fake LLM
agent = Agent()

# Run agent with a query
response = agent.run("What can you help me with?")
print(response)
```

### Understanding the Agent Loop

```python
from agt.agent import Agent

# The agent loop follows this pattern:
# 1. Receive user input
# 2. Send to LLM (currently fake)
# 3. Stream response back
# 4. Execute any triggered skills

agent = Agent(skills_dir="./skills")
result = agent.run("Calculate 2+2")
```

### Custom LLM Interface

The project uses a fake LLM for teaching. To replace with a real LLM:

```python
from agt.agent import Agent

class RealLLM:
    """Replace the fake LLM with real API calls"""
    
    def __init__(self, api_key=None):
        self.api_key = api_key or os.getenv("OPENAI_API_KEY")
    
    def stream_response(self, prompt: str):
        """Stream tokens from real LLM"""
        # Implementation would call actual API
        # and yield tokens as they arrive
        pass

# Extend Agent class to use real LLM
class ProductionAgent(Agent):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.llm = RealLLM()
```

### Creating Custom Skills

Skills are Python modules placed in the skills directory:

```python
# skills/calculator.py

def add(a: int, b: int) -> int:
    """Add two numbers together"""
    return a + b

def multiply(a: int, b: int) -> int:
    """Multiply two numbers"""
    return a * b

# Skills metadata (optional)
SKILL_METADATA = {
    "name": "calculator",
    "description": "Basic arithmetic operations",
    "functions": ["add", "multiply"]
}
```

### Programmatic Agent Interaction

```python
from agt.agent import Agent
import os

# Initialize with environment configuration
agent = Agent(
    skills_dir=os.getenv("AGENT_SKILLS_DIR", "./skills")
)

# Process multiple queries
queries = [
    "Hello, what can you do?",
    "Calculate something",
    "List your capabilities"
]

for query in queries:
    print(f"\nUser: {query}")
    response = agent.run(query)
    print(f"Agent: {response}")
```

## Configuration

### Environment Variables

```bash
# Example .env file
AGENT_SKILLS_DIR=./skills
OPENAI_API_KEY=your-api-key-here  # For when you replace fake LLM
LOG_LEVEL=INFO
```

### Agent Configuration

```python
from agt.agent import Agent

agent = Agent(
    skills_dir="./custom_skills",  # Custom skills location
    verbose=True,                   # Enable detailed logging
    max_iterations=10               # Limit agent loop iterations
)
```

## Project Structure

```
ai-agent-deep-dive/
├── src/
│   └── agt/
│       ├── agent.py      # Core agent implementation
│       ├── cli.py        # CLI entry point
│       └── __init__.py
├── skills/               # Skill modules directory
├── docs/                 # Teaching documentation
├── pyproject.toml       # Poetry dependencies
└── README.md
```

## Common Patterns

### Pattern 1: Agent with Skill Discovery

```python
from agt.agent import Agent
import os

# Agent automatically discovers skills in directory
agent = Agent(skills_dir="./skills")

# Skills are loaded and available during agent execution
response = agent.run("Use available skills to help me")
```

### Pattern 2: Streaming Response Handler

```python
from agt.agent import Agent

agent = Agent()

# The fake LLM demonstrates streaming pattern
# Real implementation would stream from API
def handle_stream(query: str):
    for chunk in agent.stream_query(query):
        print(chunk, end="", flush=True)
    print()  # Newline after stream completes
```

### Pattern 3: Extending the Agent

```python
from agt.agent import Agent

class CustomAgent(Agent):
    """Extended agent with custom behavior"""
    
    def pre_process(self, query: str) -> str:
        """Custom preprocessing"""
        return query.strip().lower()
    
    def post_process(self, response: str) -> str:
        """Custom postprocessing"""
        return response.upper()
    
    def run(self, query: str) -> str:
        query = self.pre_process(query)
        response = super().run(query)
        return self.post_process(response)
```

## Learning Resources

The repository includes extensive documentation:

- **PDF Reports**: Deep dive analysis of AI Agent architectures
  - `ai-agent-deep-dive-v2.1.pdf` - Includes memory systems chapter
  - `ai-agent-deep-dive-v2.pdf` - Core agent analysis
- **Teaching Docs**: `/docs` directory with architectural explanations

## Troubleshooting

### Skills Not Loading

```python
# Verify skills directory exists and has valid Python modules
import os

skills_dir = "./skills"
if not os.path.exists(skills_dir):
    os.makedirs(skills_dir)

# List skills to debug
from agt.cli import main
# Run: poetry run agt --list-skills
```

### Import Errors

```bash
# Ensure you're in the project directory
cd ai-agent-deep-dive

# Reinstall dependencies
poetry install

# Verify installation
poetry run python -c "from agt.agent import Agent; print('OK')"
```

### Poetry Not Found

```bash
# Install poetry
curl -sSL https://install.python-poetry.org | python3 -

# Or via pip
pip install poetry

# Verify
poetry --version
```

## Next Steps

This teaching framework uses a **fake LLM** to demonstrate structure. To build a production agent:

1. Replace the fake LLM with real API calls (OpenAI, Anthropic, etc.)
2. Implement proper error handling and retries
3. Add memory/context management
4. Enhance skill execution with function calling
5. Add logging and monitoring

**Example Real LLM Integration:**

```python
import os
from openai import OpenAI

class OpenAILLM:
    def __init__(self):
        self.client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
    
    def stream_response(self, prompt: str):
        stream = self.client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            stream=True
        )
        for chunk in stream:
            if chunk.choices[0].delta.content:
                yield chunk.choices[0].delta.content
```

This teaching project provides the **architectural foundation** — real integrations follow the same patterns but with production-grade implementations.
