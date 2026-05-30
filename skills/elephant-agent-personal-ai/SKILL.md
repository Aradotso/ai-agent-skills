---
name: elephant-agent-personal-ai
description: Build and interact with Elephant Agent, a self-evolving personal AI that grows a Personal Model and manages long-running Paths
triggers:
  - how do I set up elephant agent
  - use elephant agent to create a personal ai assistant
  - configure elephant personal model
  - create a path with elephant agent
  - interact with elephant agent cli
  - integrate elephant agent with my workflow
  - build a self-evolving ai agent with elephant
  - manage elephant agent skills and herd
---

# Elephant Agent Personal AI Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Elephant Agent is a Personal-Model First Self Evolving AI Agent that starts from understanding the person, not the task. It builds a correctable **Personal Model** across four lenses (Identity, World, Pulse, Journey) and helps design living **Paths** for work, health, habits, learning, and other long-running directions.

## What Elephant Agent Does

- **Personal Model**: Builds understanding of who you are (Identity), what surrounds you (World), what's alive right now (Pulse), and what your path has taught (Journey)
- **Paths**: Long-running arcs that break down into Steps with Checkpoints for human judgment
- **Herd Management**: Coordinates Mother elephant and baby elephants under one understanding
- **Skills & Tools**: Extensible system for browser, filesystem, MCP, and operator actions
- **Multi-Surface**: Native macOS app + CLI + Dashboard for different workflows

## Installation

### macOS Desktop App (Recommended)

```bash
# Download from GitHub releases
curl -L -o elephant-agent.dmg https://github.com/agentic-in/elephant-agent/releases/latest/download/Elephant-Agent.dmg

# Or visit releases page
open https://github.com/agentic-in/elephant-agent/releases/latest
```

### CLI + Dashboard (Linux/Cloud/SSH)

```bash
# Install via install script
curl -fsSL https://elephant.agentic-in.ai/install.sh | bash

# Or manual installation
git clone https://github.com/agentic-in/elephant-agent.git
cd elephant-agent
pip install -e .
```

## Key CLI Commands

### Initialization and Setup

```bash
# Initialize Elephant Agent (first run)
elephant init

# Check system readiness
elephant status

# Configure provider settings
elephant config set provider openai
elephant config set model gpt-4
elephant config set curiosity_effort medium

# List all configuration
elephant config list
```

### Daily Interaction

```bash
# Enter the chat TUI (main interaction mode)
elephant wake

# Quick query without entering TUI
elephant ask "What should I focus on today?"

# View Personal Model
elephant model show

# View specific model lens
elephant model show --lens identity
elephant model show --lens world
elephant model show --lens pulse
elephant model show --lens journey
```

### Paths Management

```bash
# List all paths
elephant paths list

# Create a new path
elephant paths create "Launch new product feature"

# View path details
elephant paths show <path-id>

# Update path status
elephant paths update <path-id> --status active

# Archive completed path
elephant paths archive <path-id>
```

### Herd Management

```bash
# List all agents in herd
elephant herd list

# Create a baby elephant for specific task
elephant herd spawn --role researcher --context "market analysis"

# View agent details
elephant herd show <agent-id>

# Remove agent from herd
elephant herd remove <agent-id>
```

### Skills and Tools

```bash
# List available skills
elephant skills list

# Enable a skill
elephant skills enable filesystem

# Disable a skill
elephant skills disable browser

# View skill documentation
elephant skills info filesystem
```

### Dashboard

```bash
# Open dashboard in browser
elephant dashboard

# Run dashboard without opening browser (for remote/SSH)
elephant dashboard --no-open

# Dashboard on custom port
elephant dashboard --port 8080
```

## Configuration

### Provider Configuration

```bash
# OpenAI
elephant config set provider openai
export OPENAI_API_KEY=your-key-here

# Anthropic
elephant config set provider anthropic
export ANTHROPIC_API_KEY=your-key-here

# Local models (Ollama)
elephant config set provider ollama
elephant config set model llama2
```

### Configuration File

Elephant Agent stores configuration in `~/.elephant/config.yaml`:

```yaml
# ~/.elephant/config.yaml
provider: openai
model: gpt-4
curiosity_effort: medium  # low, medium, high
language: en
boundaries:
  - respect_privacy
  - ask_before_destructive
  - explain_reasoning
posture: collaborative  # directive, collaborative, suggestive
```

### Environment Variables

```bash
# Provider API keys
export OPENAI_API_KEY=sk-...
export ANTHROPIC_API_KEY=sk-ant-...
export DEEPSEEK_API_KEY=...

# Elephant-specific settings
export ELEPHANT_HOME=~/.elephant
export ELEPHANT_LOG_LEVEL=info
export ELEPHANT_DASHBOARD_PORT=3000
```

## Python API Usage

### Basic Interaction

```python
from elephant_agent import ElephantAgent, PersonalModel

# Initialize agent
agent = ElephantAgent(
    provider="openai",
    model="gpt-4",
    curiosity_effort="medium"
)

# Load or create Personal Model
model = agent.get_personal_model()

# Ask a question
response = agent.ask("What should I prioritize today?")
print(response.message)
print(f"Confidence: {response.confidence}")
```

### Working with Personal Model

```python
from elephant_agent import PersonalModel

# Access model lenses
model = PersonalModel.load()

# View identity
identity = model.get_lens("identity")
print(f"Values: {identity.values}")
print(f"Decision style: {identity.decision_style}")

# Update world lens
model.update_lens("world", {
    "people": ["Alice (mentor)", "Bob (colleague)"],
    "projects": ["Product launch", "Team onboarding"],
    "tools": ["VS Code", "Linear", "Slack"]
})

# Check pulse
pulse = model.get_lens("pulse")
print(f"Current focus: {pulse.focus}")
print(f"Energy level: {pulse.energy}")
print(f"Constraints: {pulse.constraints}")

# Save changes
model.save()
```

### Creating and Managing Paths

```python
from elephant_agent import Path, Step

# Create a new path
path = Path.create(
    title="Launch API v2",
    description="Ship new API with auth and webhooks",
    context=model
)

# Add steps
path.add_step(Step(
    title="Design API schema",
    description="Define endpoints, auth flow, webhook events",
    checkpoint=True  # Requires human review
))

path.add_step(Step(
    title="Implement core endpoints",
    description="Build CRUD operations with auth",
    checkpoint=False
))

# Start the path
path.start()

# Get next step
next_step = path.get_next_step()
print(f"Next: {next_step.title}")

# Mark step complete
path.complete_step(next_step.id)

# Save path
path.save()
```

### Working with the Herd

```python
from elephant_agent import Herd, BabyElephant

# Get the herd
herd = Herd.load()

# Spawn a baby elephant for a specific task
researcher = BabyElephant(
    role="researcher",
    context={
        "focus": "competitor analysis",
        "constraints": ["public data only"],
        "reporting_to": "mother"
    }
)

herd.add(researcher)

# Assign task to baby
task = researcher.assign_task(
    "Research top 3 competitors' API offerings"
)

# Check status
status = researcher.get_status()
print(f"Progress: {status.progress}%")

# Get results
if task.is_complete():
    results = task.get_results()
    print(results.summary)
```

### Skills Integration

```python
from elephant_agent.skills import FileSystemSkill, BrowserSkill

# Initialize skills
fs_skill = FileSystemSkill(
    allowed_paths=["/home/user/projects"],
    readonly=False
)

browser_skill = BrowserSkill(
    headless=True,
    timeout=30
)

# Register skills with agent
agent.register_skill(fs_skill)
agent.register_skill(browser_skill)

# Use skills in context
response = agent.ask(
    "Read the README.md and create a summary document",
    skills=["filesystem"]
)
```

### Advanced: Custom Skills

```python
from elephant_agent.skills import Skill, SkillParameter

class GitSkill(Skill):
    name = "git"
    description = "Execute git commands safely"
    
    parameters = [
        SkillParameter("command", str, "Git command to run"),
        SkillParameter("args", list, "Command arguments", required=False)
    ]
    
    def execute(self, command: str, args: list = None):
        """Execute git command with safety checks."""
        import subprocess
        
        # Safety: only allow read operations
        safe_commands = ["status", "log", "diff", "branch"]
        if command not in safe_commands:
            return {
                "success": False,
                "error": f"Command '{command}' not allowed"
            }
        
        cmd = ["git", command] + (args or [])
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True
        )
        
        return {
            "success": result.returncode == 0,
            "output": result.stdout,
            "error": result.stderr
        }

# Register custom skill
git_skill = GitSkill()
agent.register_skill(git_skill)
```

## Common Patterns

### Daily Check-in Pattern

```python
from elephant_agent import ElephantAgent
from datetime import datetime

agent = ElephantAgent.load()

# Morning routine
def morning_checkin():
    model = agent.get_personal_model()
    
    # Update pulse
    model.update_pulse({
        "timestamp": datetime.now(),
        "energy": "high",
        "focus": ["API launch", "team sync"],
        "constraints": ["3hr meeting block 2-5pm"]
    })
    
    # Get prioritized guidance
    guidance = agent.ask(
        "Given my current pulse and active paths, what should I prioritize today?"
    )
    
    return guidance

checkin = morning_checkin()
print(checkin.message)
```

### Path Progress Review

```python
from elephant_agent import Path

def review_active_paths():
    paths = Path.list(status="active")
    
    for path in paths:
        print(f"\n=== {path.title} ===")
        print(f"Progress: {path.progress}%")
        
        next_step = path.get_next_step()
        if next_step:
            print(f"Next: {next_step.title}")
            
            if next_step.checkpoint:
                print("⚠️  Requires your review")
                
        blockers = path.get_blockers()
        if blockers:
            print(f"Blockers: {', '.join(blockers)}")
```

### Correcting the Personal Model

```python
from elephant_agent import PersonalModel

model = PersonalModel.load()

# Correct a misunderstanding
correction = model.correct(
    lens="identity",
    field="values",
    current="achievement, efficiency",
    corrected="learning, collaboration, impact",
    reason="I value learning and collaboration over pure efficiency"
)

# The model learns from corrections
print(f"Correction applied: {correction.applied}")
print(f"Impact: {correction.impact_summary}")
```

### Integrating with Workflows

```python
from elephant_agent import ElephantAgent
import json

agent = ElephantAgent.load()

# Export context for external tools
def export_context_for_tool(tool_name: str):
    model = agent.get_personal_model()
    
    context = {
        "current_focus": model.pulse.focus,
        "active_projects": model.world.projects,
        "constraints": model.pulse.constraints,
        "tool": tool_name
    }
    
    return json.dumps(context, indent=2)

# Import learnings back
def import_learning(source: str, content: dict):
    model = agent.get_personal_model()
    
    model.add_journey_entry({
        "source": source,
        "lesson": content["lesson"],
        "context": content["context"],
        "timestamp": content["timestamp"]
    })
```

## Troubleshooting

### Provider Connection Issues

```bash
# Test provider connectivity
elephant config test-provider

# Reset provider configuration
elephant config reset-provider

# Check API key
echo $OPENAI_API_KEY  # Should not be empty

# Verify model availability
elephant models list
```

### Personal Model Not Loading

```python
from elephant_agent import PersonalModel

# Reset model if corrupted
PersonalModel.reset()

# Reinitialize
agent.init(force=True)

# Verify model structure
model = PersonalModel.load()
assert model.is_valid(), "Model structure invalid"
```

### Dashboard Connection Issues

```bash
# Check if dashboard is running
elephant dashboard status

# Kill existing dashboard
elephant dashboard stop

# Restart on different port
elephant dashboard --port 8080

# For remote access, use tunnel
ssh -L 3000:localhost:3000 user@remote-host
```

### Performance Optimization

```python
from elephant_agent import ElephantAgent

# Reduce model calls with caching
agent = ElephantAgent(
    cache_responses=True,
    cache_ttl=300  # 5 minutes
)

# Lower curiosity effort for faster responses
agent.set_curiosity_effort("low")

# Use smaller model for routine tasks
agent.set_model("gpt-3.5-turbo")  # Faster, cheaper

# Batch questions
questions = [
    "What's my next step?",
    "Any blockers?",
    "Energy check?"
]
responses = agent.ask_batch(questions)
```

### Debugging

```bash
# Enable debug logging
export ELEPHANT_LOG_LEVEL=debug
elephant wake

# View logs
tail -f ~/.elephant/logs/elephant.log

# Check runtime state
elephant debug state

# Verify skills
elephant skills test filesystem
```

## Resources

- **Documentation**: https://elephant.agentic-in.ai/docs/
- **Paper**: https://elephant.agentic-in.ai/paper/
- **Blog**: https://elephant.agentic-in.ai/blog/personal-ai-you-create/
- **GitHub**: https://github.com/agentic-in/elephant-agent
- **Homepage**: https://elephant.agentic-in.ai/
