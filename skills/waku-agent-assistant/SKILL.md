---
name: waku-agent-assistant
description: Local-first personal AI agent with harness, loop, memory, and eval pillars
triggers:
  - "build a personal AI assistant"
  - "set up waku agent"
  - "implement AI agent memory system"
  - "create local-first AI agent"
  - "add semantic memory to agent"
  - "run waku agent dashboard"
  - "build agent with SQLite memory"
  - "implement agent eval harness"
---

# Waku Agent Assistant

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What Waku Agent Does

Waku is a local-first personal AI assistant built on four core pillars:
- **Harness**: Gateway interface (CLI, Telegram, voice, web dashboard)
- **Loop**: ~95 lines of plain Python reasoning loop (LLM ↔ tools)
- **Memory**: Three-layer system (semantic facts, episodic events, procedural skills) in SQLite
- **Eval/LLM-Ops**: Built-in deterministic tests and LLM-as-judge evaluation with release gates

Your memory lives in a single `state.db` SQLite file that you own and can inspect. No frameworks hiding the implementation.

## Installation

```bash
# Quick install (use pre-built package)
pip install waku-agent

# Development install (clone and modify)
git clone https://github.com/ShenSeanChen/waku-agent
cd waku-agent
uv venv && uv pip install -e .
cp .env.example .env
```

## Configuration

Create `.env` file with your chosen provider (only one key needed):

```bash
# Choose one provider:
WAKU_PROVIDER=anthropic  # default, or: openai, gemini, deepseek, openrouter
ANTHROPIC_API_KEY=your_key_here

# Optional: Telegram integration
TELEGRAM_BOT_TOKEN=your_bot_token

# Optional: Web search capability
TAVILY_API_KEY=your_tavily_key
```

Supported providers: Anthropic (Claude), OpenAI, Gemini, DeepSeek, MiniMax, Kimi, GLM, OpenRouter, OpenCode Zen, OpenCode Go.

## Key Commands

```bash
# Terminal chat interface
waku

# Web dashboard (localhost:7777)
waku dashboard

# Run with uv (no venv activation needed)
uv run waku
uv run waku dashboard

# Development shortcuts
make run         # terminal interface
make dashboard   # web interface
```

## Architecture Overview

```
Gateway → Working Memory → LLM Loop → Tools → Reply
                ↑                        ↓
         Retrieval Gate ← Memory (state.db)
                              ↓
                        Consolidation
```

### Core Components

1. **Gateway** (`waku/gateway/`): Multiple input channels
2. **Session** (`waku/runtime/session.py`): Working memory per turn
3. **Agent Loop** (`waku/loop/agent.py`): Reasoning and tool execution
4. **Memory** (`waku/memory/`): Three-pillar storage system
5. **Tools** (`waku/tools/`): Calendar, notes, search, messaging
6. **Ops** (`waku/ops/`): Tracing, eval, release gates

## Memory System

### Semantic Memory (Facts)

```python
from waku.memory.semantic import SemanticMemory

mem = SemanticMemory()

# Save a fact
mem.save_fact("Alex prefers morning meetings", tags=["preferences", "scheduling"])

# Search facts
results = mem.search("Alex meeting time")
# Returns: [{"content": "Alex prefers morning meetings", "tags": [...], ...}]
```

### Episodic Memory (Events)

```python
from waku.memory.episodic import EpisodicMemory

ep_mem = EpisodicMemory()

# Save an event
ep_mem.save_event(
    description="Tennis game with Raj",
    timestamp="2026-08-05T08:00:00",
    metadata={"location": "Park courts"}
)

# Retrieve recent events
recent = ep_mem.get_recent_events(days=7)
```

### Procedural Memory (Skills)

Skills live in `skills/*.md` files and `.waku/SOUL.md`:

```markdown
# SOUL.md - Your agent's personality and core instructions

## Identity
You are Waku, a helpful personal assistant.

## Communication Style
- Be concise and friendly
- Ask clarifying questions when needed

## Capabilities
- Calendar management
- Note-taking
- Web search
```

## Working with the Agent Loop

The loop is in `waku/loop/agent.py` (~95 lines):

```python
from waku.loop.agent import run_agent_loop
from waku.runtime.session import Session

session = Session(user_id="demo")
response = run_agent_loop(
    user_message="Schedule tennis with Raj on Saturday at 8am",
    session=session
)

print(response)  # Agent's final reply after tool calls
```

The loop does:
1. Calls LLM with messages and available tools
2. Executes any tool calls
3. Appends results back to messages
4. Repeats until LLM returns text (no more tool calls)

## Building Custom Tools

Tools are Python functions with docstrings describing their purpose:

```python
from waku.tools.base import tool

@tool
def calculate_tax(amount: float, rate: float) -> dict:
    """Calculate tax on an amount.
    
    Args:
        amount: The base amount in dollars
        rate: Tax rate as decimal (e.g., 0.08 for 8%)
    
    Returns:
        dict with 'total', 'tax', 'base' keys
    """
    tax = amount * rate
    return {
        "base": amount,
        "tax": tax,
        "total": amount + tax
    }
```

Register it:

```python
from waku.tools import register_tool

register_tool(calculate_tax)
```

## Retrieval Gate

The gate decides whether to retrieve memory for a turn:

```python
from waku.memory.retrieval_gate import should_retrieve

# Simple query that doesn't need context
should_retrieve("What's 2 + 2?")  # → False

# Query that needs memory lookup
should_retrieve("When is my meeting with Alex?")  # → True
```

Check gate decisions in the dashboard **Ops** tab or the **Overview** gate bar.

## Graph Workflows

For structured multi-step tasks, use graph workflows (`waku/graph/`):

```python
from waku.graph.triage import run_triage_workflow

result = run_triage_workflow(
    user_message="Search for World Cup games and add them to my calendar",
    session=session
)

# The workflow will:
# 1. Classify the intent (search + calendar)
# 2. Execute search tool multiple times
# 3. Parse results
# 4. Create calendar events for each game
```

## Dashboard Usage

```bash
waku dashboard
# Opens http://localhost:7777
```

### Dashboard Tabs

- **Overview**: Architecture diagram, costs, latency, gate metrics
- **Gateway**: Unified conversation across all input channels
- **Loop**: Turn-by-turn execution with tool calls and tokens
- **Graph**: Workflow topology visualization
- **Memory**: Browse semantic facts, episodes, skills
- **Tools**: Available tools and their results
- **Data**: Live SQLite browser for `state.db`
- **Ops**: Eval history, gate decisions, traces

### Chat in Dashboard

The chat dock (right side) supports:
- Text input
- Voice input
- New conversation
- Message history
- Multi-channel tagging (shows if message came from CLI, Telegram, etc.)

## Evaluation System

### Deterministic Tests

```python
# evals/deterministic/test_memory.py
from waku.memory.semantic import SemanticMemory

def test_fact_storage():
    mem = SemanticMemory()
    mem.save_fact("Test fact")
    results = mem.search("Test")
    assert len(results) > 0
    assert "Test fact" in results[0]["content"]
```

Run tests:

```bash
pytest evals/deterministic/
```

### LLM-as-Judge Evals

```python
# evals/judge/scenarios.py
SCENARIOS = [
    {
        "input": "Remember that Alex prefers morning meetings",
        "expected_behavior": "Should save a semantic fact about Alex's preference",
        "judge_prompt": "Did the agent store this preference in memory?"
    }
]
```

Run judge evals:

```bash
python evals/judge/run_judge.py
```

## Common Patterns

### Multi-Tool Coordination

```python
# Agent automatically chains tools:
"Search for Python conferences in 2026 and add them to my calendar"

# Execution flow:
# 1. search_web("Python conferences 2026")
# 2. search_web("PyCon 2026 dates")
# 3. create_event("PyCon", "2026-04-15")
# 4. create_event("EuroPython", "2026-07-20")
# ... (multiple iterations in one turn)
```

### Memory Consolidation

After every N chat turns, Waku consolidates episodic memory into semantic facts:

```python
from waku.memory.consolidation import consolidate_memory

# Runs automatically, but you can trigger manually:
consolidate_memory(session)

# Converts patterns like:
# Episodes: "Meeting with Alex (9am)", "Call with Alex (10am)"
# → Fact: "Alex prefers morning communication"
```

### Cross-Channel Conversations

Start a conversation in CLI, continue in dashboard, respond via Telegram — all tracked in one thread:

```bash
# Terminal
$ waku
You: Schedule tennis on Saturday
Waku: What time?

# Dashboard (localhost:7777)
You: 8am please

# Telegram
You: /status
Waku: Your Saturday 8am tennis game is confirmed.
```

## Inspecting Memory

### Via Dashboard

**Data** tab → `facts` or `episodes` table → browse or run SQL:

```sql
SELECT * FROM facts WHERE content LIKE '%Alex%';
```

### Via Code

```python
import sqlite3

conn = sqlite3.connect(".waku/state.db")
cursor = conn.cursor()

# View all facts
cursor.execute("SELECT * FROM facts")
for row in cursor.fetchall():
    print(row)

# Full-text search
cursor.execute("SELECT * FROM facts WHERE content MATCH 'meeting'")
```

### Via File System

```bash
cat .waku/MEMORY.md  # Human-readable mirror of state.db
cat .waku/SOUL.md    # Agent personality and instructions
ls skills/           # Procedural skills (*.md)
```

## Troubleshooting

### "No API key found"

Set one provider key in `.env`:

```bash
WAKU_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...
```

### Memory not persisting

Check that `.waku/state.db` exists and is writable:

```bash
ls -la .waku/
sqlite3 .waku/state.db "SELECT COUNT(*) FROM facts;"
```

### Dashboard won't start

Port 7777 already in use:

```bash
# Find process using port
lsof -i :7777
# Kill it or change port in waku/gateway/dashboard_server.py
```

### Agent not using tools

Check tool registration in `waku/tools/__init__.py` and verify tools appear in dashboard **Tools** tab.

### Gate always skipping retrieval

Check gate threshold in `waku/memory/retrieval_gate.py`:

```python
# Adjust sensitivity (0.0 = always retrieve, 1.0 = never retrieve)
GATE_THRESHOLD = 0.5  # Default
```

## Advanced: Custom Gateway

Add a new input channel:

```python
# waku/gateway/slack.py
from waku.runtime.session import Session
from waku.loop.agent import run_agent_loop

def handle_slack_message(user_id: str, message: str):
    session = Session(user_id=user_id, channel="slack")
    response = run_agent_loop(message, session)
    return response  # Send back to Slack
```

## File Structure

```
waku-agent/
├── waku/
│   ├── gateway/          # CLI, Telegram, dashboard, voice
│   ├── loop/             # agent.py (main loop), models.py (LLM adapters)
│   ├── graph/            # Structured workflows
│   ├── memory/           # semantic/, episodic/, procedural/, consolidation
│   ├── tools/            # Built-in tools (calendar, notes, search)
│   ├── runtime/          # session.py (working memory)
│   └── ops/              # tracing.py, release_gate.py
├── evals/
│   ├── deterministic/    # Pytest-based tests
│   └── judge/            # LLM-as-judge scenarios
├── skills/               # Procedural memory (.md files)
├── .waku/
│   ├── state.db          # SQLite database (your memory)
│   ├── SOUL.md           # Agent personality
│   └── MEMORY.md         # Human-readable memory mirror
└── .env                  # API keys (gitignored)
```

## Resources

- [GitHub Repository](https://github.com/ShenSeanChen/waku-agent)
- [20-min Code Walkthrough](https://www.youtube.com/watch?v=rvRyBhILrls)
- [Architecture Diagrams](docs/whiteboards/) (editable .excalidraw files)
- [Full Architecture Docs](docs/architecture.md)

## License

MIT License — code you own and can modify freely.
