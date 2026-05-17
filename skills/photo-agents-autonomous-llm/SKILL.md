---
name: photo-agents-autonomous-llm
description: Build autonomous self-evolving AI agents with vision-grounded memory that operate computers through a perceive-reason-act cycle
triggers:
  - create a photo agent that can see and control my computer
  - set up autonomous agents with visual memory
  - build a self-evolving AI agent with computer control
  - implement vision-grounded agent memory
  - create an agent that writes its own skills
  - set up photo agents with layered memory system
  - build autonomous agents with browser automation
  - configure LLM agents with photographic memory
---

# Photo Agents Autonomous LLM Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

Photo Agents is a Python framework for building autonomous, self-evolving AI agents that ground their understanding in visual observations of the screen. Unlike traditional text-only agents, Photo Agents implements a perceive → reason → act cycle with a layered memory architecture inspired by biological cognition: vision input, bounded observations stored in layers (L1-L4), and skills the agent writes from real successes.

**Key capabilities:**
- Multi-provider LLM routing (Anthropic Claude, OpenAI GPT, failover sessions)
- Layered memory system (working/global/SOP/session archive)
- Physical execution tools (file I/O, sandboxed code, browser automation via Chrome DevTools Protocol)
- Multiple client interfaces (CLI, Streamlit web app, PyQt desktop, chat platform bots)
- Self-evolving through reflection and skill generation

## Installation

### Basic Installation

```bash
pip install photoagents
```

### Full Installation with All Clients

```bash
pip install "photoagents[all]"
```

**Requirements:** Python 3.10+

## API Key Setup

Photo Agents requires a license key validated against `https://photo-agents.com/v1/keys/validate`.

1. Get your key at: https://photo-agents.com/dashboard/keys
2. Configure it (choose one method):

**Environment variable:**
```bash
export PHOTOAGENTS_API_KEY=pk_live_your_key_here
```

**Config file** (`~/.photoagents/config.json`):
```json
{
  "api_key": "pk_live_your_key_here"
}
```

**Interactive prompt:** Run any command and it will prompt you to enter and save the key.

## LLM Provider Configuration

Create a `credentials.py` file in your project root:

```python
# credentials.py
from photoagents.config.keys_template import LLMConfig, ProviderConfig

# Option 1: Anthropic Claude
llm_config = LLMConfig(
    primary=ProviderConfig(
        provider="anthropic",
        api_key="${ANTHROPIC_API_KEY}",  # Use env var
        model="claude-3-5-sonnet-20241022"
    )
)

# Option 2: OpenAI GPT
llm_config = LLMConfig(
    primary=ProviderConfig(
        provider="openai",
        api_key="${OPENAI_API_KEY}",
        model="gpt-4o"
    )
)

# Option 3: Failover configuration
llm_config = LLMConfig(
    primary=ProviderConfig(
        provider="anthropic",
        api_key="${ANTHROPIC_API_KEY}",
        model="claude-3-5-sonnet-20241022"
    ),
    fallback=ProviderConfig(
        provider="openai",
        api_key="${OPENAI_API_KEY}",
        model="gpt-4o"
    )
)
```

Or use JSON format (`credentials.json`):

```json
{
  "primary": {
    "provider": "anthropic",
    "api_key": "${ANTHROPIC_API_KEY}",
    "model": "claude-3-5-sonnet-20241022"
  },
  "fallback": {
    "provider": "openai",
    "api_key": "${OPENAI_API_KEY}",
    "model": "gpt-4o"
  }
}
```

## Core Usage Patterns

### 1. Interactive CLI Mode

```bash
# Start interactive REPL
python -m photoagents

# The agent will prompt for tasks and execute them
# with vision-grounded reasoning
```

### 2. One-Shot Task Execution

```bash
# Execute a single task
python -m photoagents --task my_analysis --input "Analyze the largest files in this directory"

# With custom output path
python -m photoagents --task report --input "Generate system report" --output ./reports/
```

### 3. Reflection/Watchdog Mode

```bash
# Run with reflection scheduler (self-evolving)
python -m photoagents --reflect photoagents/evolution/scheduler.py
```

### 4. Programmatic Agent Session

```python
from photoagents.core.loop import run_agent_session
from photoagents.llm.router import LLMSession
from photoagents.config.keys_template import LLMConfig, ProviderConfig

# Configure LLM
llm_config = LLMConfig(
    primary=ProviderConfig(
        provider="anthropic",
        api_key="${ANTHROPIC_API_KEY}",
        model="claude-3-5-sonnet-20241022"
    )
)

# Create session
session = LLMSession(llm_config)

# Run agent loop
result = run_agent_session(
    task_name="file_analysis",
    user_input="Find and summarize all Python files in the current directory",
    session=session,
    max_turns=10
)

print(f"Final output: {result}")
```

### 5. Custom Tool Integration

```python
from photoagents.core.tool_dispatcher import register_tool
from typing import Dict, Any

@register_tool
def custom_analysis_tool(data: str, options: Dict[str, Any]) -> str:
    """
    Custom tool for specialized analysis.
    
    Args:
        data: Input data to analyze
        options: Configuration options
        
    Returns:
        Analysis results
    """
    # Your custom logic here
    result = f"Analyzed: {data} with options {options}"
    return result

# Tool is now available to the agent
```

## GUI Client Options

### Streamlit Web App + WebView

```bash
# Launch web interface with native window
pythonw -m photoagents.cli.launcher
```

### Service Hub (Start/Stop Services)

```bash
# Launch control hub
pythonw -m photoagents.cli.hub
```

### Desktop PyQt Application

```bash
python -m photoagents.clients.desktop_app
```

### Desktop Companion

```bash
pythonw -m photoagents.clients.companion_v2
```

### Chat Platform Bots

```bash
# Telegram
python -m photoagents.clients.telegram_client

# Feishu (Lark)
python -m photoagents.clients.feishu_client

# WeCom
python -m photoagents.clients.wecom_client

# DingTalk
python -m photoagents.clients.dingtalk_client

# QQ
python -m photoagents.clients.qq_client
```

## Layered Memory System

Photo Agents uses a 4-layer memory architecture:

### L1: Working Memory
Short-term context for the current task (conversation turns, immediate observations).

### L2: Global Memory
Long-term facts stored in `~/.photoagents/global_mem.txt`.

```python
from photoagents.core.memory import add_global_fact, search_global_memory

# Add a fact
add_global_fact("Project uses Python 3.11 and requires PostgreSQL 14+")

# Search memory
results = search_global_memory("database requirements")
```

### L3: Skills & SOPs
Standard Operating Procedures the agent writes from successful executions.

```python
from photoagents.skills.skill_manager import save_skill, load_skill

# Save a new skill
save_skill(
    name="web_scraping_pattern",
    code="""
def scrape_structured_data(url: str) -> dict:
    # Implementation
    pass
""",
    description="Reliable pattern for scraping structured web data"
)

# Load and use
skill = load_skill("web_scraping_pattern")
```

### L4: Session Archive
Full raw session logs in `~/.photoagents/sessions/`.

## Browser Automation with CDP

Photo Agents includes Chrome DevTools Protocol integration for browser control:

```python
from photoagents.web.cdp_bridge import CDPBridge

async def automate_browser():
    async with CDPBridge() as browser:
        # Navigate
        await browser.navigate("https://example.com")
        
        # Take screenshot
        screenshot = await browser.screenshot()
        
        # Execute JavaScript
        result = await browser.evaluate("document.title")
        
        # Click element
        await browser.click("button.submit")
        
        # Fill form
        await browser.type("input[name='query']", "search term")
        
    return result
```

## Vision-Grounded Operations

### Screenshot Analysis

```python
from photoagents.skills.vision import analyze_screenshot

# Agent automatically captures and analyzes screen
analysis = analyze_screenshot(
    region=(0, 0, 1920, 1080),  # x, y, width, height
    question="What UI elements are visible?"
)
```

### OCR Text Extraction

```python
from photoagents.skills.ocr import extract_text_from_region

# Extract text from screen region
text = extract_text_from_region(
    x=100, y=200, width=500, height=300
)
```

## Sandboxed Code Execution

```python
from photoagents.core.sandbox import execute_code

# Python execution
result = execute_code(
    code="""
import json
data = {"status": "success"}
print(json.dumps(data))
""",
    language="python",
    timeout=30
)

# PowerShell (Windows)
ps_result = execute_code(
    code="Get-Process | Select-Object -First 5",
    language="powershell"
)

# Bash (Linux/Mac)
bash_result = execute_code(
    code="ls -la | head -n 10",
    language="bash"
)
```

## File I/O Operations

```python
from photoagents.core.file_ops import read_file, write_file, list_directory

# Read file
content = read_file("~/project/config.json")

# Write file
write_file("~/output/report.txt", "Analysis complete\n")

# List directory with filters
files = list_directory(
    path="~/project",
    pattern="*.py",
    recursive=True
)
```

## Observability with Langfuse

```python
from photoagents.integrations.langfuse_tracer import init_langfuse, trace_agent_step

# Initialize
tracer = init_langfuse(
    public_key="${LANGFUSE_PUBLIC_KEY}",
    secret_key="${LANGFUSE_SECRET_KEY}",
    host="https://cloud.langfuse.com"
)

# Trace agent steps
with trace_agent_step("file_analysis", metadata={"task": "analyze_logs"}):
    # Agent operations here
    pass
```

## Configuration Files

### On-Disk State Locations

| Path | Purpose |
|------|---------|
| `~/.photoagents/config.json` | API key + license validation cache |
| `~/.photoagents/global_mem.txt` | L2 long-term facts |
| `~/.photoagents/sessions/` | L4 raw session archives |
| `~/.photoagents/skill_index/` | Vector index for skill/SOP search |
| `~/.photoagents/temp/` | Per-task scratch (logs, intermediate output) |

### Custom System Prompt

Override the default system prompt:

```python
from photoagents.core.loop import run_agent_session

custom_prompt = """
You are a specialized data analysis agent.
Focus on: statistical analysis, visualization, and reporting.
Always verify data integrity before processing.
"""

result = run_agent_session(
    task_name="analysis",
    user_input="Analyze sales data",
    system_prompt_override=custom_prompt
)
```

## Common Patterns

### Pattern 1: Autonomous Research Agent

```python
from photoagents.core.loop import run_agent_session
from photoagents.llm.router import LLMSession

def create_research_agent(topic: str):
    session = LLMSession.from_env()
    
    result = run_agent_session(
        task_name=f"research_{topic}",
        user_input=f"""
        Research {topic} and create a comprehensive report:
        1. Search for recent information
        2. Analyze credibility of sources
        3. Synthesize findings
        4. Save report with citations
        """,
        session=session,
        max_turns=50
    )
    
    return result

# Use it
report = create_research_agent("quantum computing advances 2026")
```

### Pattern 2: Self-Evolving Monitor

```python
# monitor.py
from photoagents.evolution.scheduler import schedule_check

def check() -> bool:
    """
    Watchdog function that triggers agent tasks.
    Return True to execute a task.
    """
    import os
    import time
    
    # Check if it's time to run daily backup
    last_run = os.path.getmtime("~/.photoagents/last_backup")
    if time.time() - last_run > 86400:  # 24 hours
        return True
    
    return False

def get_task() -> str:
    """Return the task to execute when check() returns True."""
    return "Backup all project files to ~/backups/ and verify integrity"

# Run with:
# python -m photoagents --reflect monitor.py
```

### Pattern 3: Multi-Step Workflow

```python
from photoagents.core.loop import run_agent_session
from photoagents.core.memory import add_global_fact

def execute_workflow(project_path: str):
    # Step 1: Analyze codebase
    analysis = run_agent_session(
        task_name="code_analysis",
        user_input=f"Analyze Python code structure in {project_path}"
    )
    
    # Save insight to global memory
    add_global_fact(f"Project at {project_path}: {analysis}")
    
    # Step 2: Generate documentation
    docs = run_agent_session(
        task_name="generate_docs",
        user_input=f"Create API documentation for {project_path}"
    )
    
    # Step 3: Run tests
    tests = run_agent_session(
        task_name="run_tests",
        user_input=f"Execute test suite and report coverage"
    )
    
    return {
        "analysis": analysis,
        "documentation": docs,
        "tests": tests
    }
```

## Troubleshooting

### API Key Issues

**Problem:** `PhotoAgentsAuthError: Invalid or missing API key`

**Solution:**
```bash
# Verify key is set
echo $PHOTOAGENTS_API_KEY

# Or check config file
cat ~/.photoagents/config.json

# Clear cache if key was recently updated
rm ~/.photoagents/config.json
```

### LLM Provider Errors

**Problem:** `LLM provider authentication failed`

**Solution:**
```bash
# Verify environment variables are set
echo $ANTHROPIC_API_KEY
echo $OPENAI_API_KEY

# Test credentials.py is in correct location
ls credentials.py

# Check credentials.py syntax
python -c "from credentials import llm_config; print(llm_config)"
```

### Memory Issues

**Problem:** Agent can't recall previous facts

**Solution:**
```python
# Check global memory file exists
import os
print(os.path.exists(os.path.expanduser("~/.photoagents/global_mem.txt")))

# Manually verify content
with open(os.path.expanduser("~/.photoagents/global_mem.txt")) as f:
    print(f.read())

# Rebuild skill index if corrupted
from photoagents.skills.skill_manager import rebuild_index
rebuild_index()
```

### Browser Automation Fails

**Problem:** CDP bridge cannot connect to Chrome

**Solution:**
```bash
# Ensure Chrome is installed and accessible
which google-chrome
which chrome

# Launch Chrome with remote debugging manually
google-chrome --remote-debugging-port=9222

# Check port availability
lsof -i :9222
```

### Session Archive Growth

**Problem:** `~/.photoagents/sessions/` consuming too much disk

**Solution:**
```bash
# Clean old sessions (older than 30 days)
find ~/.photoagents/sessions/ -type f -mtime +30 -delete

# Or configure auto-cleanup
python -c "
from photoagents.core.cleanup import configure_auto_cleanup
configure_auto_cleanup(max_age_days=30, max_size_mb=1000)
"
```

### Permission Errors

**Problem:** Cannot write to `~/.photoagents/`

**Solution:**
```bash
# Fix ownership
sudo chown -R $USER:$USER ~/.photoagents/

# Fix permissions
chmod -R 755 ~/.photoagents/
```

## Advanced Configuration

### Custom Tool Schema

```python
from photoagents.resources.tool_schema import register_custom_tool

schema = {
    "name": "analyze_metrics",
    "description": "Analyze system metrics and generate report",
    "parameters": {
        "type": "object",
        "properties": {
            "metric_type": {
                "type": "string",
                "enum": ["cpu", "memory", "disk", "network"]
            },
            "duration_hours": {
                "type": "integer",
                "minimum": 1,
                "maximum": 168
            }
        },
        "required": ["metric_type"]
    }
}

register_custom_tool(schema, implementation_function)
```

### Environment Variables Reference

```bash
# Required
export PHOTOAGENTS_API_KEY=pk_live_xxx

# LLM Providers (choose one or both for fallback)
export ANTHROPIC_API_KEY=sk-ant-xxx
export OPENAI_API_KEY=sk-xxx

# Optional integrations
export LANGFUSE_PUBLIC_KEY=pk-lf-xxx
export LANGFUSE_SECRET_KEY=sk-lf-xxx
export LANGFUSE_HOST=https://cloud.langfuse.com

# Chat platform bots (if using)
export TELEGRAM_BOT_TOKEN=xxx
export FEISHU_APP_ID=xxx
export FEISHU_APP_SECRET=xxx
```
