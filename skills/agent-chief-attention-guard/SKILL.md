---
name: agent-chief-attention-guard
description: Use Agent Chief to filter and manage AI agent notifications, alerts, and events with local-first LLM-based attention management
triggers:
  - set up agent chief to filter my notifications
  - configure chief to manage agent interruptions
  - add event sources to agent chief
  - create a custom chief policy for my workflow
  - integrate my agent with chief's dispatch system
  - check what chief would interrupt me about
  - tune chief's decision thresholds
  - review chief's learned preferences
---

# Agent Chief — Local-First Attention Guard

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agent Chief is a local-first attention management layer that sits between you and all your agents, alerts, and event sources. It uses a three-stage filtering engine (hard rules → similarity classifier → LLM judge) to decide whether to interrupt you, batch into a digest, dispatch to agents, or curate into memory. Everything runs locally with explainable decisions and zero telemetry.

## Installation

```bash
# Quick demo (no config needed)
uvx agent-chief demo

# Install persistently
uv tool install agent-chief

# Or via pip
pip install agent-chief
```

## Core Commands

```bash
# Interactive setup wizard (all questions skippable)
chief init

# Start the resident daemon + local console (127.0.0.1:7800)
chief run

# Run offline demo (24 events → 1 interrupt)
chief demo

# Trace a specific decision with full breakdown
chief trace evt_20260706_1040_ab12

# View learned policy and preferences
chief policy

# Generate trust report after shadow mode
chief report

# Run evaluation suite
chief eval                    # routing regression on demo events
chief eval --learning         # single-user feedback loop
chief eval --cohort          # 100-user benchmark
chief eval --compare v1 v2   # prompt version diff
```

## Configuration

Chief stores everything under `~/.chief/` (configurable via `CHIEF_HOME`):

```
~/.chief/
├── config.toml           # Core configuration
├── POLICY.md            # Human-readable learned policy (editable!)
├── chief.db             # SQLite: events, decisions, memory
└── .env                 # LLM API keys (optional for local models)
```

### Minimal config.toml

```toml
[llm]
# DeepSeek (cheap, good quality)
backend = "deepseek"
api_key_env = "DEEPSEEK_API_KEY"

# Or use local models
# backend = "ollama"
# model = "llama3.2:latest"

[scenes]
# Define your contexts with interrupt thresholds (0-1)
deep_work = { threshold = 0.85, active_hours = "9-12,14-17" }
available = { threshold = 0.60 }
sleeping = { threshold = 0.95, active_hours = "23-7" }
commute = { threshold = 0.75, active_hours = "8-9,17-18" }

[behavior]
shadow_days = 7          # Don't actually interrupt for first 7 days
batch_window_minutes = 30
max_batch_size = 10
```

### Environment Variables

```bash
# LLM API keys (only if using hosted models)
export DEEPSEEK_API_KEY="your_key_here"
export OPENAI_API_KEY="your_key_here"

# Override home directory
export CHIEF_HOME="/custom/path"
```

## Sending Events to Chief

### HTTP API

```python
import httpx

# Send an event to Chief's resident daemon
response = httpx.post("http://127.0.0.1:7800/v1/events", json={
    "source": "my-monitor",
    "title": "Disk usage above 90%",
    "body": "The /var partition is at 92% capacity on prod-server-3",
    "topic": "infra.alerts",  # optional, Chief infers if omitted
    "severity": "warning",     # info|warning|error|critical
    "metadata": {
        "server": "prod-server-3",
        "partition": "/var",
        "usage_pct": 92
    }
})

decision = response.json()
print(f"Route: {decision['route']}")  # interrupt|digest|dispatch|block
print(f"Score: {decision['score']:.2f}")
print(f"Reason: {decision['reason']}")
```

### Python SDK

```python
from agent_chief import ChiefClient

client = ChiefClient()  # connects to http://127.0.0.1:7800

# Send event and get routing decision
decision = client.propose_event(
    source="github-watcher",
    title="PR #482 merged to main",
    body="Feature: Add OAuth login flow\nAuthor: @alice\n+240 -18",
    topic="dev.github",
    metadata={"pr_number": 482, "author": "alice"}
)

if decision.route == "interrupt":
    print(f"🔔 Chief says interrupt: {decision.reason}")
elif decision.route == "dispatch":
    print(f"🤖 Dispatched to: {decision.agent_id}")
```

### Model Context Protocol (MCP)

Chief exposes an MCP server for Claude Desktop and other MCP clients:

```json
// Add to your MCP config (e.g., ~/Library/Application Support/Claude/claude_desktop_config.json)
{
  "mcpServers": {
    "chief": {
      "command": "chief",
      "args": ["mcp"]
    }
  }
}
```

Then in Claude:

```
Use Chief to propose this event: "Nightly backup completed successfully"
```

## Agent Integration Patterns

### Heartbeat Agent with Chief

```python
import time
from agent_chief import ChiefClient

chief = ChiefClient()

def check_system_health():
    """Your existing health check logic"""
    return {
        "status": "healthy",
        "cpu": 45.2,
        "memory": 62.1,
        "disk": 78.5
    }

# Instead of always alerting, let Chief decide
while True:
    health = check_system_health()
    
    # Chief will block "all clear" reports automatically
    decision = chief.propose_event(
        source="health-monitor",
        title=f"System health: {health['status']}",
        body=f"CPU: {health['cpu']}% | Memory: {health['memory']}% | Disk: {health['disk']}%",
        topic="infra.monitoring",
        severity="info" if health['status'] == "healthy" else "warning"
    )
    
    # Only noisy if Chief decides it's worth attention
    if decision.route == "interrupt":
        print(f"🚨 {decision.reason}")
    
    time.sleep(300)  # Check every 5 minutes
```

### CI/CD Integration

```python
# In your CI post-hook
from agent_chief import ChiefClient

def on_build_complete(build_result):
    chief = ChiefClient()
    
    decision = chief.propose_event(
        source="github-actions",
        title=f"Build {'✓ passed' if build_result.success else '✗ failed'}: {build_result.branch}",
        body=f"Commit: {build_result.commit_sha[:8]}\nDuration: {build_result.duration}s",
        topic="dev.ci",
        severity="info" if build_result.success else "error",
        metadata={
            "branch": build_result.branch,
            "commit": build_result.commit_sha,
            "duration": build_result.duration,
            "tests_failed": build_result.failures
        }
    )
    
    # Chief batches passing builds, interrupts failures on main
    return decision
```

### Dispatch with Verification

Chief can dispatch work to your agents and verify completion:

```python
from agent_chief import ChiefClient

client = ChiefClient()

# Register agent capability
client.register_agent(
    agent_id="log-analyzer",
    capabilities=["analyze_logs", "suggest_fix"],
    verification="command:grep -q 'Analysis complete' /tmp/analysis.log"
)

# When Chief dispatches to this agent, it will:
# 1. Call your agent's handler
# 2. Run the verification command
# 3. Only mark "done" if verification passes
# 4. Report back if verification fails

def handle_dispatch(task):
    # Your agent's work
    analyze_logs(task.event.metadata["log_file"])
    write_report("/tmp/analysis.log")
```

## Policy Editing

Chief learns from your ±1 feedback and distills preferences into `POLICY.md`. You can hand-edit this file:

```markdown
# Chief Attention Policy

## Topic Preferences (EMA-learned)
- dev.ci: +0.15 (↑ after 3 positive signals)
- infra.monitoring: -0.20 (↓ after 5 negative signals)
- social.twitter: -0.50 (manually set)

## Hard Rules
- BLOCK: source=heartbeat AND body contains "all clear"
- ALWAYS_INTERRUPT: severity=critical AND topic=infra.prod
- BATCH: topic=dev.github.prs AND severity=info
```

Changes take effect immediately — no restart needed. Chief reconciles your edits with learned weights.

## Tracing Decisions

Every decision is fully traceable:

```bash
$ chief trace evt_20260706_1040_ab12
CI failed on main: test_auth_flow broken by PR #482  dev.ci · github-actions
route dispatch at stage 3 in scene deep_work (confidence 0.85)
score 0.87  urgency=0.90 relevance=0.90 actionability=0.85 novelty=0.80 confidence=0.90
┌────────────┬───────┬──────────────────────┐
│ stage      │    ms │ note                 │
│ stage1     │   0.1 │ no hard rule fired   │
│ associate  │   1.2 │ 0 memory hits        │
│ judge      │ 812.4 │ backend deepseek     │
│ route      │   0.3 │ routed dispatch      │
└────────────┴───────┴──────────────────────┘
tokens: 1104 in (704 cached) / 96 out · prompt v1 · cost $0.000301
```

## Working with Scenes

Scenes are your contexts (working, sleeping, commuting). Chief uses them to adjust interrupt thresholds:

```python
from agent_chief import ChiefClient

client = ChiefClient()

# Manually switch scene (or let Chief infer from time/calendar)
client.set_scene("deep_work")

# Same event scores differently in different scenes
decision1 = client.propose_event(
    source="slack", 
    title="New message in #random",
    scene="available"  # threshold 0.60
)
# → digest (score 0.65, below deep_work but above available)

decision2 = client.propose_event(
    source="slack",
    title="New message in #random", 
    scene="deep_work"  # threshold 0.85
)
# → block (same event, stricter scene)
```

## Common Patterns

### RSS Feed Integration

```python
import feedparser
from agent_chief import ChiefClient

chief = ChiefClient()

for entry in feedparser.parse("https://example.com/feed.xml").entries:
    chief.propose_event(
        source="rss.example",
        title=entry.title,
        body=entry.summary[:500],
        topic="news.tech",
        metadata={"url": entry.link}
    )
```

### Cron Job Notifications

```python
#!/usr/bin/env python3
from agent_chief import ChiefClient
import subprocess

chief = ChiefClient()
result = subprocess.run(["backup-script.sh"], capture_output=True)

chief.propose_event(
    source="cron.backups",
    title=f"Nightly backup {'succeeded' if result.returncode == 0 else 'FAILED'}",
    body=result.stdout.decode() if result.returncode == 0 else result.stderr.decode(),
    topic="infra.backups",
    severity="info" if result.returncode == 0 else "error"
)
```

### Training the Policy

```python
from agent_chief import ChiefClient

client = ChiefClient()

# Give feedback on decisions (trains per-topic weights)
decision = client.propose_event(
    source="twitter",
    title="New mention",
    body="@you nice work on that PR!"
)

# If you check the digest and it WAS worth attention:
client.feedback(decision.id, thumbs_up=True)

# If it was noise:
client.feedback(decision.id, thumbs_up=False)

# Chief adjusts topic=social.twitter weight accordingly
```

## Troubleshooting

### Chief won't interrupt in shadow mode
This is intentional. For the first 7 days (or 50 graded samples), Chief only simulates interrupts. Check the digest for `⚡ would have: interrupted` annotations and grade them with ✓/✗. Run `chief report` to see graduation status.

### High LLM costs
1. Use DeepSeek (cheap) or local Ollama models
2. Check cache hit rate: `chief trace <id>` shows "X cached" tokens
3. Tune stage-1 rules to block more before LLM: edit `POLICY.md`
4. Most events (75%) should die at stage 1/2 (microseconds/milliseconds)

### Events not routing as expected
```bash
# Check what Chief saw
chief trace evt_<id>

# Verify topic inference
chief debug --event '{"title": "...", "body": "..."}'

# Check learned weights
chief policy

# Reset learning (careful!)
chief reset --learning-only
```

### Daemon won't start
```bash
# Check if port 7800 is in use
lsof -i :7800

# Use different port
chief run --port 8000

# Check logs
tail -f ~/.chief/logs/chief.log
```

### LLM backend errors
Chief degrades gracefully to rules-only routing when the LLM is unavailable. Check:
```bash
# Test LLM connectivity
chief debug --test-llm

# Force rules-only mode for testing
chief run --rules-only
```

### Verification failing for dispatched tasks
```python
# Check verification command runs successfully
import subprocess
result = subprocess.run(["your-verify-command"], capture_output=True)
print(result.returncode, result.stdout, result.stderr)

# Use LLM verification instead of command
client.register_agent(
    agent_id="my-agent",
    capabilities=["task"],
    verification="llm:Did the agent complete the task successfully?"
)
```

## Testing Your Integration

```python
# test_chief_integration.py
import pytest
from agent_chief import ChiefClient

@pytest.fixture
def chief():
    # Use test mode (in-memory DB, no actual interrupts)
    return ChiefClient(test_mode=True)

def test_critical_alert_interrupts(chief):
    decision = chief.propose_event(
        source="test",
        title="Production down",
        severity="critical",
        topic="infra.prod"
    )
    assert decision.route == "interrupt"
    assert decision.score >= 0.90

def test_heartbeat_blocked(chief):
    decision = chief.propose_event(
        source="heartbeat",
        title="System check",
        body="All systems operational, no issues to report"
    )
    assert decision.route == "block"
```

## Advanced: Custom Judge Prompts

Chief's LLM judge uses versioned prompt templates. To experiment:

```bash
# Copy current prompt
cp ~/.chief/judge/templates/v1/system.jinja2 ~/.chief/judge/templates/v2/system.jinja2

# Edit v2
vim ~/.chief/judge/templates/v2/system.jinja2

# Test against golden set
chief eval --compare v1 v2

# Use in production
chief config set judge.prompt_version v2
```

## Resources

- **Demo replay**: `chief demo` (24 events, fully offline)
- **Evaluation suite**: `chief eval --help`
- **Policy reference**: `~/.chief/POLICY.md`
- **Metrics**: Every decision includes token count and USD cost
- **Learning curve**: `chief eval --learning` shows 0% → 100% convergence
- **Cohort benchmark**: `chief eval --cohort` (100-user dataset)
