---
name: genericagent-self-evolving-ai-agent
description: Self-evolving autonomous agent framework with skill tree growth, browser/desktop/mobile control, and hierarchical memory system
triggers:
  - set up GenericAgent for autonomous task automation
  - create a self-evolving AI agent with GenericAgent
  - configure GenericAgent with browser and system control
  - build skills and memory layers with GenericAgent
  - automate desktop tasks using GenericAgent
  - integrate GenericAgent with Claude/Gemini/GPT models
  - implement autonomous web browsing with GenericAgent
  - create custom agent skills in GenericAgent
---

# GenericAgent Self-Evolving AI Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

GenericAgent is a minimal (~3K LOC) self-evolving autonomous agent framework that grants LLMs system-level control over computers. It features 9 atomic tools for browser, terminal, filesystem, keyboard/mouse, screen vision, and mobile (ADB) control. The core innovation is automatic skill crystallization: every solved task becomes a reusable skill, forming a personal skill tree that grows with usage while consuming 6x fewer tokens than traditional agents.

## Installation

### Quick Install (Recommended)

**Windows PowerShell:**
```powershell
powershell -ExecutionPolicy Bypass -c "$env:GLOBAL=1; irm http://fudankw.cn:9000/files/ga_install.ps1 | iex"
```

**Linux/macOS:**
```bash
GLOBAL=1 bash -c "$(curl -fsSL http://fudankw.cn:9000/files/ga_install.sh)"
```

### Developer Install

```bash
# Clone repository
git clone https://github.com/lsdefine/GenericAgent.git
cd GenericAgent

# Create virtual environment (Python 3.11 or 3.12 required)
uv venv
uv pip install -e ".[ui]"

# Configure API key
cp mykey_template.py mykey.py
# Edit mykey.py with your LLM API credentials
```

**Important:** Use Python 3.11 or 3.12. Python 3.14 is incompatible with `pywebview` and other dependencies.

## Configuration

### API Key Setup

Edit `mykey.py`:

```python
# For Claude
ANTHROPIC_API_KEY = "your-key-here"

# For Gemini
GEMINI_API_KEY = "your-key-here"

# For OpenAI-compatible APIs
OPENAI_API_KEY = "your-key-here"
OPENAI_BASE_URL = "https://api.openai.com/v1"

# For Kimi
MOONSHOT_API_KEY = "your-key-here"

# For MiniMax
MINIMAX_API_KEY = "your-key-here"
MINIMAX_GROUP_ID = "your-group-id"
```

Better practice using environment variables:

```python
import os

ANTHROPIC_API_KEY = os.getenv("ANTHROPIC_API_KEY")
GEMINI_API_KEY = os.getenv("GEMINI_API_KEY")
```

### Memory System Configuration

GenericAgent uses a 4-layer hierarchical memory system (L1-L4):

```python
# In your agent configuration
memory_config = {
    "l1_working_memory": True,      # Current conversation context
    "l2_episodic_memory": True,     # Recent task history
    "l3_skill_library": True,       # Crystallized skills
    "l4_session_archive": True      # Long-term archives
}
```

## Core Architecture

### Agent Loop (100 lines)

The core agent loop is minimal:

```python
from genericagent import GenericAgent

# Initialize agent
agent = GenericAgent(
    model="claude-sonnet-4.6",  # or gpt-5.4, gemini-2.0-flash, etc.
    working_dir="./workspace"
)

# Execute task
result = agent.run("Order me a milk tea from the delivery app")
```

### 9 Atomic Tools

GenericAgent provides 9 atomic tools for system control:

1. **Browser Control** - Real browser injection (preserves sessions)
2. **Terminal Execution** - Shell command execution
3. **File Operations** - Read/write/search filesystem
4. **Screen Vision** - Screenshot capture and analysis
5. **Keyboard Input** - Direct keyboard control
6. **Mouse Control** - Click, drag, move operations
7. **ADB Mobile** - Android device control
8. **Python REPL** - Interactive Python execution
9. **Memory Operations** - Read/write skill library

## Usage Patterns

### Basic Task Execution

```python
from genericagent import GenericAgent

# Create agent instance
agent = GenericAgent(
    model="claude-sonnet-4.6",
    verbose=True
)

# Single-turn task
agent.run("Find all PDF files in ~/Documents and move them to ~/PDFs")

# Multi-turn conversation
agent.chat("Install the requests library")
agent.chat("Now use it to fetch https://api.github.com/repos/lsdefine/GenericAgent")
agent.chat("Save the star count to stars.txt")
```

### Skill Crystallization

Skills are automatically created when tasks complete:

```python
# First time: Agent explores and learns
agent.run("Read my WeChat messages")
# Agent installs deps, reverses DB schema, writes script, saves skill

# Every subsequent time: Direct skill invocation
agent.run("Read my WeChat messages")
# Agent loads existing skill, executes instantly
```

### Browser Automation

```python
from genericagent import GenericAgent

agent = GenericAgent(model="claude-sonnet-4.6")

# Browser tasks preserve login sessions
agent.run("""
Navigate to gmail.com, compose an email to john@example.com
with subject 'Q4 Report' and attach the file ~/reports/q4.pdf
""")

# Multi-step web workflows
agent.run("""
1. Go to Amazon
2. Search for 'wireless keyboard'
3. Filter by 4+ stars and under $50
4. Take screenshots of top 3 results
5. Save product names and prices to products.csv
""")
```

### Desktop Automation

```python
agent = GenericAgent(model="gemini-2.0-flash")

# Combine vision + mouse/keyboard
agent.run("""
Open my expense tracking spreadsheet,
find all transactions over $2000 in the last 3 months,
and create a summary chart
""")

# System-level automation
agent.run("""
Set up a cron job that runs every day at 9 AM
to backup ~/Documents to ~/Backups
""")
```

### Mobile Device Control (ADB)

```python
agent = GenericAgent(model="claude-sonnet-4.6")

# Android automation via ADB
agent.run("""
Open Alipay on my phone,
navigate to transaction history,
find expenses over ¥2000 in last 3 months,
take screenshots
""")
```

### Quantitative Analysis Example

```python
agent = GenericAgent(model="claude-opus-4.6")

# First run: Agent installs mootdx, builds screening logic
agent.run("""
Find GEM stocks with:
- EXPMA golden cross
- Turnover > 5%
- Save results to stocks.csv
""")

# Skill is crystallized, future runs are instant
# The screening logic is now in your personal skill tree
```

## Frontends

### Desktop GUI

```bash
# Launch desktop app (after one-line install)
frontends/GenericAgent.exe

# Or for developers
python launch.pyw
```

### Terminal UI (TUI v2)

```bash
# Textual-based interface with streaming support
python frontends/tuiapp_v2.py
```

**TUI Commands:**
- `Ctrl+N` - New session
- `Ctrl+S` - Save current session
- `Ctrl+L` - Load session
- `/llm <model>` - Switch LLM model
- `/export` - Export conversation
- `/continue` - Resume previous session

### Streamlit Web UI

```bash
python launch.pyw
```

### IM Bot Frontends

```bash
# Telegram bot
python frontends/tgapp.py

# WeChat bot
python frontends/wechatapp.py

# QQ bot
python frontends/qqapp.py

# Feishu/Lark bot
python frontends/fsapp.py

# WeCom bot
python frontends/wecomapp.py

# DingTalk bot
python frontends/dingtalkapp.py
```

**Bot Commands:**
- `/new` - Start fresh conversation
- `/continue` - List recoverable snapshots
- `/continue N` - Restore snapshot N

## Advanced Features

### Conductor Sub-Agent Orchestration

```python
from genericagent import GenericAgent, Conductor

# Main agent spawns sub-agents for parallel tasks
main_agent = GenericAgent(model="claude-sonnet-4.6")

# Conductor manages sub-agent lifecycle
conductor = Conductor(main_agent)

# Parallel task execution
conductor.spawn_agent("research", "Research competitors in AI agent space")
conductor.spawn_agent("analysis", "Analyze our user feedback from last month")
conductor.spawn_agent("report", "Draft Q1 roadmap based on research and analysis")

# Auto-cleanup and result aggregation
results = conductor.wait_all()
```

### Custom Skill Creation

```python
# Skills are stored in memory/L3_skills/
# Create custom skill manually:

skill_code = """
def check_stock_alerts():
    '''Monitor stocks and send alerts'''
    import mootdx
    from mootdx.quotes import Quotes
    
    client = Quotes.factory(market='std')
    # Custom screening logic
    symbols = client.stocks(market='cyb')
    
    for stock in symbols:
        # Check conditions
        if meets_criteria(stock):
            send_alert(stock)
    
    return results
"""

# Save to skill library
agent.save_skill("stock_monitoring", skill_code)

# Invoke skill
agent.run("Run my stock monitoring skill")
```

### Session Management

```python
# Save current session
agent.save_session("project_setup")

# List available sessions
sessions = agent.list_sessions()

# Load previous session
agent.load_session("project_setup")

# Continue from L4 archive
agent.continue_from_archive(session_id=3)
```

### Scheduler Integration

```python
from genericagent import GenericAgent
import schedule

agent = GenericAgent(model="claude-sonnet-4.6")

# Define recurring task
def daily_report():
    agent.run("Generate daily sales report and email to team@company.com")

# Schedule with cron-like syntax
schedule.every().day.at("09:00").do(daily_report)

# Or let agent set it up
agent.run("""
Set up a scheduled task that runs every morning at 9 AM
to generate a sales report and email it to the team
""")
```

### Side Questions with /btw

```python
# During complex task, ask side questions without losing context
agent.chat("Deploy the new feature to production")
# Mid-task: check something
agent.chat("/btw what's the current server load?")
# Returns to main task automatically
```

## Real-World Examples

### Autonomous Web Data Collection

```python
agent = GenericAgent(model="claude-sonnet-4.6")

agent.run("""
Visit techcrunch.com, browse the latest AI articles,
summarize the top 5 stories, and save summaries to ai_news.md.
Check back every hour and update the file.
""")
```

### Expense Tracking with Mobile App

```python
agent.run("""
Connect to my Android phone via ADB,
open Alipay, navigate to bill details,
extract all transactions from last quarter,
categorize by type (food, transport, shopping),
create a pie chart visualization,
save report as Q1_expenses.pdf
""")
```

### Bulk Messaging

```python
agent.run("""
Read contacts from team_contacts.csv,
send a WeChat message to each person:
'Reminder: Team meeting tomorrow at 2 PM'
""")
```

### Custom Automation Workflow

```python
agent.run("""
1. Monitor my Gmail for emails with 'URGENT' in subject
2. When found, extract key points
3. Create a task in my todo.txt file
4. Send me a desktop notification
5. Run this check every 15 minutes
""")
```

## Troubleshooting

### Python Version Issues

**Problem:** Installation fails with dependency conflicts

**Solution:** Ensure Python 3.11 or 3.12:
```bash
python --version  # Should show 3.11.x or 3.12.x
# If wrong version, install correct Python and recreate venv
```

### TUI Rendering Issues on Windows

**Problem:** TUI displays broken characters or doesn't respond to input

**Solution:**
```bash
# Update textual
pip install -U textual

# Use Git Bash instead of PowerShell/cmd
# Or ask GenericAgent to fix it:
python frontends/tuiapp_v2.py
# In chat: "Fix TUI rendering issues for Windows terminal"
```

### Browser Automation Not Working

**Problem:** Browser control fails or doesn't preserve sessions

**Solution:**
```python
# Check if browser driver is installed
agent.run("Install Chrome WebDriver for browser automation")

# Verify browser path
agent.run("Check if Chrome is installed and accessible")

# For Firefox users
agent.run("Configure Firefox profile for persistent sessions")
```

### Skill Not Crystallizing

**Problem:** Task completes but no skill is saved

**Solution:**
```python
# Manually save skill after successful execution
agent.run("Save the last task execution as a skill named 'email_reports'")

# Check skill library
agent.run("List all available skills in my library")

# Verify L3 memory is enabled
agent.config['l3_skill_library'] = True
```

### Memory Context Issues

**Problem:** Agent forgets previous context or hallucinates

**Solution:**
```python
# Check active memory layers
agent.run("Show current memory configuration")

# Clear and rebuild memory
agent.clear_l1_memory()  # Working memory
agent.rebuild_l2_memory()  # Episodic memory

# Reduce context by archiving old sessions
agent.archive_session()
```

### ADB Device Not Found

**Problem:** Mobile automation fails with "device not found"

**Solution:**
```bash
# Check ADB connection
adb devices

# Enable USB debugging on Android device
# Connect device and authorize computer

# Let agent diagnose
agent.run("Troubleshoot ADB connection to my Android device")
```

### High Token Usage

**Problem:** Consuming too many tokens per task

**Solution:**
```python
# GenericAgent is designed for efficiency, but check:

# 1. Ensure skills are being reused
agent.run("List my most frequently used skills")

# 2. Archive old sessions to L4
agent.run("Archive conversations older than 1 week")

# 3. Use lighter model for simple tasks
agent = GenericAgent(model="gemini-2.0-flash")  # vs claude-opus-4.6

# 4. Check if unnecessary tools are being called
agent.config['verbose'] = True  # Log tool calls
```

## Best Practices

1. **Let Skills Grow Organically:** Don't pre-install everything. Let GenericAgent install dependencies as needed and crystallize skills.

2. **Use Appropriate Models:** Use lighter models (Gemini Flash) for simple tasks, heavier (Claude Opus) for complex reasoning.

3. **Leverage Memory Layers:** Regularly archive old sessions to L4 to keep L1/L2 context clean.

4. **Session Management:** Save important sessions with descriptive names for easy recovery.

5. **Environment Variables:** Always use env vars for API keys, never hardcode.

6. **Incremental Complexity:** Start with simple tasks, build to complex workflows as skills accumulate.

7. **Monitor Token Usage:** Track token consumption to optimize model selection and skill reuse.

## Resources

- **GitHub:** https://github.com/lsdefine/GenericAgent
- **Technical Report:** https://arxiv.org/abs/2604.17091
- **Tutorial (Chinese):** https://datawhalechina.github.io/hello-generic-agent/
- **Skill Library:** Released March 2026 (million-scale)
- **Evaluation Data:** https://github.com/JinyiHan99/GA-Technical-Report
