---
name: awesome-agentic-ai-zh-learning
description: Structured learning roadmap for AI Agent development from LLM basics to multi-agent systems (bilingual Chinese/English)
triggers:
  - how do I learn AI agents from scratch
  - show me the agentic AI learning path
  - what's the roadmap for building AI agents
  - guide me through learning LLM and agent frameworks
  - I want to build my first AI agent
  - explain the AI agent learning stages
  - what resources for learning agentic AI
  - help me understand MCP and Claude Code ecosystem
---

# awesome-agentic-ai-zh Learning Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

`awesome-agentic-ai-zh` is a comprehensive, structured learning roadmap for AI Agent development that takes you from **LLM basics to building multi-agent systems**. It provides:

- **Two learning tracks**: Track A (CLI Power User) and Track B (Agent Builder)
- **8 core stages** with 145+ curated projects and resources
- **27 hands-on exercises** with working code examples
- **Bilingual content** (Traditional Chinese, Simplified Chinese, English)
- **5 specialized branches** for researchers, developers, teachers, knowledge workers, and everyday users

The project is particularly valuable for understanding the **Claude Code ecosystem** (MCP, Skills, Plugins, Subagents) and modern agent interfaces (Computer Use, Browser Use, Code Sandbox).

## Installation & Setup

```bash
# Clone the repository
git clone https://github.com/WenyuChiou/awesome-agentic-ai-zh.git
cd awesome-agentic-ai-zh

# No additional dependencies for reading the roadmap
# Individual exercises may require Python and specific libraries
```

For complete setup (first-time learners):
```bash
# Read the setup guide first
cat resources/setup-guide.md

# Install Python 3.8+ if needed
python --version

# Install common dependencies for exercises
pip install anthropic openai langchain chromadb
```

## Learning Path Structure

### Shared Foundation (Stage 0-2)

**Stage 0: Foundations** (`stages/00-foundations.md`)
- Python, CLI, git, API basics, JSON
- Duration: 1-2 weeks

**Stage 1: LLM Basics** (`stages/01-llm-basics.md`)
- Token concepts, API usage, LLM comparison, local LLM (Ollama)
- Duration: 1 week

**Stage 2: Prompt Engineering** (`stages/02-prompt-engineering.md`)
- System prompts, few-shot learning, Chain-of-Thought
- Duration: 1-2 weeks

### Track A: CLI Power User

```bash
# Navigate Track A
cat tracks/cli/A1-cli-intro.md           # CLI agent comparison & setup
cat tracks/cli/A2-cli-workflow.md        # Workflow patterns
cat tracks/cli/A3-cli-production.md      # Production integration

# Key resource
cat resources/cli-agents-guide.md
```

**Total duration**: 8-10 weeks (including shared foundation)

### Track B: Agent Builder

```bash
# Navigate Track B
cat stages/03-tool-use-and-hello-agent.md    # Function calling, ReAct
cat stages/04-agent-frameworks.md             # LangGraph, AutoGen, CrewAI
cat stages/05-claude-code-ecosystem.md        # MCP, Skills, Plugins (SHARED HUB)
cat stages/06-memory-rag.md                   # Context engineering, RAG
cat stages/07-multi-agent-production.md       # Multi-agent orchestration
cat stages/07.5-advanced-agentic-concepts.md  # Advanced concepts (reading)
cat stages/08-agent-interfaces.md             # Computer Use, Browser Use (SHARED HUB)
```

**Total duration**: 16-22 weeks minimum, 5-7 months realistically (5-8 hrs/week)

## Key Commands & Navigation

### Finding Resources

```bash
# List all stage files
ls stages/

# View glossary of terms
cat resources/glossary.md

# Check CLI agents comparison
cat resources/cli-agents-guide.md

# Browse exercises
ls exercises/stage-*/
```

### Running Exercises

Each stage has 1-5 exercises in `exercises/stage-X/`:

```python
# Example: Stage 1 - First LLM API call
# exercises/stage-1/01-first-llm-call/main.py

import os
from anthropic import Anthropic

def main():
    client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
    
    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[
            {"role": "user", "content": "Explain AI agents in one sentence."}
        ]
    )
    
    print(message.content[0].text)

if __name__ == "__main__":
    main()
```

```bash
# Set up environment
export ANTHROPIC_API_KEY="your-key-here"

# Run exercise
python exercises/stage-1/01-first-llm-call/main.py
```

### Dual-Path SDK Examples

Most exercises provide both **Anthropic SDK** and **Ollama** (local) implementations:

```python
# Using Anthropic Claude
from anthropic import Anthropic

client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
)

# Using Ollama (local)
import ollama

response = ollama.chat(
    model="llama3.2",
    messages=[{"role": "user", "content": "Hello"}]
)
```

## Configuration Patterns

### API Key Management

```bash
# Set environment variables (recommended)
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."

# Or use .env file
cat > .env << EOF
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
EOF

# Load in Python
from dotenv import load_dotenv
load_dotenv()
```

### Local LLM Setup (Ollama)

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull a model
ollama pull llama3.2

# Test
ollama run llama3.2 "Explain what an AI agent is"
```

## Common Usage Patterns

### Pattern 1: Following the Learning Path

```bash
# For complete beginners
cat stages/00-foundations.md
# → Complete exercises in exercises/stage-0/

# Then proceed sequentially
cat stages/01-llm-basics.md
cat stages/02-prompt-engineering.md

# Choose your track
cat tracks/cli/A1-cli-intro.md    # OR
cat stages/03-tool-use-and-hello-agent.md
```

### Pattern 2: Quick Reference for Specific Topics

```bash
# Need MCP information?
cat stages/05-claude-code-ecosystem.md

# Need multi-agent patterns?
cat stages/07-multi-agent-production.md

# Need Computer Use examples?
cat stages/08-agent-interfaces.md
```

### Pattern 3: Building Your First Agent

Follow the comprehensive walkthrough:

```bash
cat walkthroughs/build-first-agent-in-7-steps.md
```

Example from the walkthrough (Stage 3: Tool Use):

```python
# exercises/stage-3/02-function-calling/main.py
import os
import json
from anthropic import Anthropic

def get_weather(city: str) -> dict:
    """Mock weather API - returns fake data"""
    return {
        "city": city,
        "temperature": 22,
        "condition": "sunny"
    }

def main():
    client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
    
    tools = [{
        "name": "get_weather",
        "description": "Get current weather for a city",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "City name"}
            },
            "required": ["city"]
        }
    }]
    
    response = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        tools=tools,
        messages=[
            {"role": "user", "content": "What's the weather in Tokyo?"}
        ]
    )
    
    # Handle tool use
    if response.stop_reason == "tool_use":
        tool_use = next(block for block in response.content if block.type == "tool_use")
        
        if tool_use.name == "get_weather":
            result = get_weather(**tool_use.input)
            
            # Send result back
            response = client.messages.create(
                model="claude-3-5-sonnet-20241022",
                max_tokens=1024,
                tools=tools,
                messages=[
                    {"role": "user", "content": "What's the weather in Tokyo?"},
                    {"role": "assistant", "content": response.content},
                    {
                        "role": "user",
                        "content": [{
                            "type": "tool_result",
                            "tool_use_id": tool_use.id,
                            "content": json.dumps(result)
                        }]
                    }
                ]
            )
    
    print(response.content[0].text)

if __name__ == "__main__":
    main()
```

### Pattern 4: ReAct Agent Implementation

```python
# exercises/stage-3/04-react-agent/main.py
import os
from anthropic import Anthropic

def search_papers(query: str) -> list:
    """Mock paper search"""
    return [
        {"title": "Attention Is All You Need", "year": 2017},
        {"title": "BERT: Pre-training of Deep Bidirectional Transformers", "year": 2018}
    ]

def summarize_paper(title: str) -> str:
    """Mock paper summarizer"""
    return f"Summary of '{title}': A foundational paper in NLP..."

def react_loop(user_query: str, max_iterations: int = 5):
    client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
    
    tools = [
        {
            "name": "search_papers",
            "description": "Search academic papers by query",
            "input_schema": {
                "type": "object",
                "properties": {
                    "query": {"type": "string"}
                },
                "required": ["query"]
            }
        },
        {
            "name": "summarize_paper",
            "description": "Get summary of a paper by title",
            "input_schema": {
                "type": "object",
                "properties": {
                    "title": {"type": "string"}
                },
                "required": ["title"]
            }
        }
    ]
    
    messages = [{"role": "user", "content": user_query}]
    
    for i in range(max_iterations):
        response = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=2048,
            tools=tools,
            messages=messages
        )
        
        messages.append({"role": "assistant", "content": response.content})
        
        if response.stop_reason == "end_turn":
            return response.content[0].text
        
        # Execute tools
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                if block.name == "search_papers":
                    result = search_papers(**block.input)
                elif block.name == "summarize_paper":
                    result = summarize_paper(**block.input)
                
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": str(result)
                })
        
        messages.append({"role": "user", "content": tool_results})
    
    return "Max iterations reached"

# Usage
result = react_loop("Find papers about transformers and summarize the most important one")
print(result)
```

## MCP (Model Context Protocol) Integration

Stage 5 covers the Claude Code ecosystem. Key MCP concepts:

```python
# Example MCP server structure
# See stages/05-claude-code-ecosystem.md for details

from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("my-mcp-server")

@app.tool()
async def get_document(doc_id: str) -> str:
    """Fetch document by ID"""
    # Your implementation
    return f"Document content for {doc_id}"

@app.tool()
async def search_database(query: str) -> list:
    """Search internal database"""
    # Your implementation
    return [{"id": "1", "title": "Result"}]
```

### Using MCP with Claude Desktop

```json
// ~/Library/Application Support/Claude/claude_desktop_config.json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/path/to/your/mcp_server.py"]
    }
  }
}
```

## Specialized Branches

### For Researchers

```bash
cat branches/for-researcher.md
```

Focus: Literature review automation, paper writing assistance, multi-agent review systems

### For Developers

```bash
cat branches/for-developer.md
```

Focus: Cursor/Aider integration, CLI delegation, code review agents

### For Everyday Users

```bash
cat branches/for-everyday-users.md
```

Focus: Using ChatGPT/Claude.ai effectively, privacy scenarios, CLI agent introduction (no coding required)

## Troubleshooting

### Common Issues

**Issue: "API key not found"**
```bash
# Verify environment variable
echo $ANTHROPIC_API_KEY

# Set it if missing
export ANTHROPIC_API_KEY="sk-ant-..."
```

**Issue: "Module not found"**
```bash
# Install required package
pip install anthropic

# For exercises requiring multiple packages
pip install -r requirements.txt
```

**Issue: "Ollama connection refused"**
```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# Start Ollama if needed
ollama serve
```

**Issue: "Which stage should I start from?"**
- Have Python/git basics? → Start Stage 1
- Complete beginner? → Start Stage 0
- Want to use CLI agents without coding? → Go directly to Track A (A1-cli-intro.md)

### Stage-Specific Help

```bash
# Each stage has a glossary
cat resources/glossary.md

# Setup troubleshooting
cat resources/setup-guide.md

# CLI-specific issues
cat resources/cli-agents-guide.md
```

## Best Practices

1. **Follow the path sequentially** — Each stage builds on previous knowledge
2. **Complete the exercises** — 27 hands-on exercises are designed for learning by doing
3. **Use dual-path approach** — Try both cloud APIs (Anthropic/OpenAI) and local models (Ollama)
4. **Check the glossary** — `resources/glossary.md` has all terminology in Chinese + English
5. **Join the community** — The project welcomes contributions and questions
6. **Set realistic expectations** — Track B takes 5-7 months part-time; Track A takes 8-10 weeks

## Example: Complete Agent Build

See `walkthroughs/build-first-agent-in-7-steps.md` for a 350-line Paper Summary Bot that evolves from Stage 1 to Stage 7, demonstrating:

- LLM API basics (Stage 1)
- Prompt engineering (Stage 2)
- Tool use and ReAct (Stage 3)
- Framework integration with LangGraph (Stage 4)
- Memory and RAG (Stage 6)
- Multi-agent orchestration (Stage 7)

## Additional Resources

```bash
# Full glossary
cat resources/glossary.md

# Diagrams
ls resources/diagrams/
# - learning-map.png
# - branch-decision-tree.png
# - banner.png

# Complete setup guide
cat resources/setup-guide.md
```

## Project Metadata

- **License**: MIT
- **Languages**: Traditional Chinese (primary), Simplified Chinese, English
- **Scope**: 145+ curated projects, 27 exercises, 8 stages, 2 tracks, 5 branches
- **Repository**: https://github.com/WenyuChiou/awesome-agentic-ai-zh
- **Stars**: 1462+ (as of 2026-05-16)
