---
name: awesome-hermes-agent-ecosystem
description: Navigate the Hermes Agent ecosystem — skills, tools, integrations, deployment, and multi-agent orchestration resources
triggers:
  - "show me hermes agent skills"
  - "how do I extend hermes agent"
  - "what plugins are available for hermes"
  - "deploy hermes agent to production"
  - "find hermes integrations"
  - "set up multi-agent system with hermes"
  - "discover hermes ecosystem tools"
  - "get started with hermes agent resources"
---

# Awesome Hermes Agent Ecosystem

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill provides comprehensive knowledge of the Hermes Agent ecosystem maintained at [0xNyk/awesome-hermes-agent](https://github.com/0xNyk/awesome-hermes-agent). Hermes Agent by Nous Research is a self-improving AI agent with built-in learning loop, autonomous skill curation, and 18-platform messaging support.

## What is Hermes Agent?

Hermes Agent is the only agent with:
- **Built-in learning loop** — creates skills from experience and improves them during use
- **Autonomous Curator (v0.12.0+)** — maintains its own skill library with 7-day grade/consolidate/prune cycles
- **18 messaging platforms** — Telegram, Discord, Slack, WhatsApp, Signal, Feishu/Lark, WeCom, QQBot, Yuanbao, and more
- **7 terminal backends** — local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox
- **Conversational memory** — searches past conversations and builds user model across sessions

Core repository: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) (134k+ stars)

## Quick Start Path

### 1. Install Hermes Agent

```bash
# Clone the repository
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent

# Install dependencies (Python 3.10+)
pip install -e .

# Configure API keys
export ANTHROPIC_API_KEY="your_key_here"
# or use .env file
cp .env.example .env
```

### 2. First Conversation

```bash
# Start interactive CLI
hermes

# Or one-off command
hermes "analyze this codebase and suggest improvements"
```

### 3. Add Your First Skills

**Option A: Cross-platform skills (wondelai)**
```bash
# Clone wondelai/skills (380+ stars, production-ready)
cd ~/.hermes/skills
git clone https://github.com/wondelai/skills.git wondelai

# Skills auto-discovered on next run
hermes "use the youtube search skill to find videos about agent frameworks"
```

**Option B: Literate programming (litprog-skill)**
```bash
# Clone litprog-skill for code + documentation workflows
cd ~/.hermes/skills
git clone https://github.com/tlehman/litprog-skill.git

# Use literate programming
hermes "create a literate program that implements a binary tree"
```

### 4. Add a GUI

**Option A: Hermes-native workspace**
```bash
# Clone hermes-workspace (500+ stars)
git clone https://github.com/outsourc-e/hermes-workspace.git
cd hermes-workspace
npm install
npm run dev
# Access at http://localhost:3000
```

**Option B: Multi-agent orchestration**
```bash
# Clone mission-control (3.7k+ stars)
git clone https://github.com/builderz-labs/mission-control.git
cd mission-control
docker-compose up -d
# Access at http://localhost:8080
```

## Key Configuration

### Profile System (Multi-Instance)

```yaml
# ~/.hermes/profiles/work.yaml
name: work
messaging_platform: slack
terminal_backend: docker
model: claude-opus-4
memory:
  conversation_window: 50
  auto_summarize: true
```

```bash
# Use specific profile
hermes --profile work "analyze the sales dashboard"
```

### Cron Scheduling

```yaml
# ~/.hermes/cron.yaml
jobs:
  - name: daily-standup
    schedule: "0 9 * * 1-5"  # 9 AM weekdays
    command: "summarize yesterday's commits and create standup report"
    
  - name: skill-curator
    schedule: "0 2 * * 0"  # 2 AM Sundays
    command: "hermes curator --grade --consolidate --prune"
```

### Messaging Gateway Setup

```bash
# Telegram
export TELEGRAM_BOT_TOKEN="your_token"
hermes --messaging telegram

# Discord
export DISCORD_BOT_TOKEN="your_token"
hermes --messaging discord

# Slack
export SLACK_BOT_TOKEN="xoxb-your-token"
export SLACK_APP_TOKEN="xapp-your-token"
hermes --messaging slack
```

## Essential Skills & Plugins

### Goal Management & Cost Control

```bash
# Install hermes-plugins (42-evey)
cd ~/.hermes/skills
git clone https://github.com/42-evey/hermes-plugins.git

# Usage examples
hermes "set a goal to refactor the authentication module by Friday"
hermes "show me this week's API cost breakdown by model"
hermes "switch to claude-sonnet for this task to save costs"
```

### Inter-Agent Communication

```bash
# Using hermes-plugins inter-agent bridge
# Terminal 1: Agent A
hermes --profile researcher --port 8001

# Terminal 2: Agent B
hermes --profile writer --port 8002

# Agent A to Agent B
hermes "send to writer agent: use this research summary to write a blog post"
```

### Incident Response (SRE)

```bash
# Install hermes-incident-commander
cd ~/.hermes/skills
git clone https://github.com/Lethe044/hermes-incident-commander.git

# Cron-based monitoring
hermes "monitor production services and alert on failures"

# Manual incident handling
hermes "diagnose why the API is returning 503 errors and apply a fix"
```

### Spotify Control (Linux/Raspberry Pi)

```bash
# Install hermes-spotify-skill (Linux-native)
cd ~/.hermes/skills
git clone https://github.com/Alexeyisme/hermes-spotify-skill.git

# Configure Spotify credentials
export SPOTIPY_CLIENT_ID="your_client_id"
export SPOTIPY_CLIENT_SECRET="your_client_secret"
export SPOTIPY_REDIRECT_URI="http://localhost:8888/callback"

# Usage
hermes "play lo-fi hip hop on Spotify"
hermes "skip to the next track"
hermes "transfer playback to my Raspberry Pi speaker"
```

### Self-Hosted Cloud Integration (Nextcloud)

```bash
# Install hermes-nextcloud
cd ~/.hermes/skills
git clone https://github.com/adnw-vinc/hermes-nextcloud.git

# Configure
export NEXTCLOUD_URL="https://your-nextcloud.example.com"
export NEXTCLOUD_USERNAME="your_username"
export NEXTCLOUD_APP_PASSWORD="your_app_password"

# Usage
hermes "list my nextcloud files"
hermes "create a note about today's meeting in nextcloud"
hermes "show my calendar for next week"
```

## Deployment Patterns

### Production VPS Deployment

```bash
# Systemd service
cat > /etc/systemd/system/hermes.service <<EOF
[Unit]
Description=Hermes Agent
After=network.target

[Service]
Type=simple
User=hermes
WorkingDirectory=/opt/hermes-agent
Environment="ANTHROPIC_API_KEY=your_key"
ExecStart=/usr/bin/hermes --messaging telegram --profile production
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable hermes
sudo systemctl start hermes
```

### Docker Deployment

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY . .
RUN pip install -e .

ENV ANTHROPIC_API_KEY=""
ENV TELEGRAM_BOT_TOKEN=""

CMD ["hermes", "--messaging", "telegram"]
```

```bash
# Build and run
docker build -t hermes-agent .
docker run -d \
  -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
  -e TELEGRAM_BOT_TOKEN="$TELEGRAM_BOT_TOKEN" \
  -v ~/.hermes:/root/.hermes \
  --name hermes \
  hermes-agent
```

### Vercel Sandbox Deployment

```bash
# Use Vercel Sandbox backend (serverless)
hermes --terminal vercel-sandbox "deploy this Next.js app"
```

### Modal Deployment (Serverless GPU)

```bash
# Use Modal for GPU-intensive tasks
hermes --terminal modal "train this model on the full dataset"
```

## Multi-Agent Orchestration

### Using oh-my-hermes Suite

```bash
# Install oh-my-hermes (witt3rd)
cd ~/.hermes/skills
git clone https://github.com/witt3rd/oh-my-hermes.git

# Deep research workflow
hermes "research the state of agent frameworks using deep-research skill"

# Consensus planning (Planner → Architect → Critic)
hermes "create a ralplan for building a distributed task queue"

# Verified execution (execute → verify → iterate)
hermes "implement the task queue using ralph skill"

# Full autopilot
hermes "autopilot: research agent frameworks, plan architecture, implement MVP"
```

### Inter-Agent Bridge Pattern

```python
# In agent A's skill
def send_to_agent(target_agent: str, message: str, port: int):
    """Send message to another Hermes instance"""
    import requests
    
    response = requests.post(
        f"http://localhost:{port}/api/message",
        json={"message": message}
    )
    return response.json()

# Usage
send_to_agent("writer", "Use this data to write a report", port=8002)
```

## MCP (Model Context Protocol) Integration

```bash
# List available MCP servers
hermes "list mcp servers"

# Use MCP server
hermes "use the filesystem MCP server to read project files"

# Configure custom MCP server
cat > ~/.hermes/mcp-servers.json <<EOF
{
  "servers": {
    "custom-db": {
      "command": "node",
      "args": ["/path/to/db-mcp-server/index.js"],
      "env": {
        "DB_CONNECTION_STRING": "postgresql://..."
      }
    }
  }
}
EOF
```

## Memory System

### Conversational Memory

```bash
# Search past conversations
hermes "what did we discuss about authentication last week?"

# Summarize conversation history
hermes "summarize all our discussions about the API redesign"
```

### Skill Memory

```bash
# Skills automatically improve from experience
# After using a skill multiple times:
hermes "show me how the 'api-integration' skill has evolved"

# Curator grades skills automatically (v0.12.0+)
hermes curator --report
```

## Advanced Patterns

### Self-Improving Skill Factory

```bash
# Install hermes-skill-factory
cd ~/.hermes/skills
git clone https://github.com/Romanescu11/hermes-skill-factory.git

# Auto-generate skill from workflow
hermes "watch me deploy this app, then create a reusable deployment skill"
```

### Personal Obsidian API

```bash
# Install personal-api skill
cd ~/.hermes/skills
git clone https://github.com/beiyuii/personal-api-skill.git

# Point to your Obsidian vault
export OBSIDIAN_VAULT_PATH="$HOME/Documents/Obsidian"

# Usage
hermes "read my daily notes and suggest priorities"
hermes "what did I write about agent frameworks?"
```

### Accumulator Betting Tracker

```bash
# Install acca-tracker (sports betting multi-leg tracking)
cd ~/.hermes/skills
git clone https://github.com/svenmedina07-ship-it/skills.git
cd skills/acca-tracker

# Configure cron for 15-minute updates
hermes "set up acca tracking cron job"

# Manual check
hermes "check my accumulator bet status"
```

## API Server Mode

```bash
# Start Hermes as API server
hermes serve --port 8080

# Use REST API
curl -X POST http://localhost:8080/api/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "analyze this code", "context": {"file": "main.py"}}'
```

```python
# Python client
import requests

def ask_hermes(message: str, context: dict = None):
    response = requests.post(
        "http://localhost:8080/api/chat",
        json={"message": message, "context": context or {}}
    )
    return response.json()["response"]

# Usage
result = ask_hermes("refactor this function", context={"file": "utils.py"})
```

## Troubleshooting

### Skills Not Loading

```bash
# Check skills directory
ls -la ~/.hermes/skills/

# Verify skill structure (needs skill.md or SKILL.md)
cat ~/.hermes/skills/your-skill/skill.md

# Force reload
hermes --reload-skills "list available skills"
```

### Memory Issues

```bash
# Clear conversation history
hermes clear-history

# Reduce memory window in profile
# ~/.hermes/profiles/default.yaml
memory:
  conversation_window: 20  # Reduce from default 50
```

### Curator Failures

```bash
# Manual curator run with debug
hermes curator --grade --verbose

# Skip consolidation if stuck
hermes curator --grade --prune --skip-consolidate

# Reset curator state
rm ~/.hermes/curator/state.json
```

### Messaging Gateway Issues

```bash
# Test connection
hermes --messaging telegram --test-connection

# Debug webhook (for platforms like Telegram)
export HERMES_WEBHOOK_URL="https://your-domain.com/webhook"
hermes --messaging telegram --debug
```

### Terminal Backend Failures

```bash
# Test Docker backend
docker ps  # Ensure Docker is running
hermes --terminal docker "echo test"

# Fallback to local execution
hermes --terminal local "echo test"

# SSH backend with custom key
hermes --terminal ssh \
  --ssh-host "192.168.1.100" \
  --ssh-user "ubuntu" \
  --ssh-key "~/.ssh/hermes_key"
```

## Common Workflows

### Daily Standup Automation

```yaml
# ~/.hermes/cron.yaml
jobs:
  - name: daily-standup
    schedule: "0 9 * * 1-5"
    command: |
      summarize yesterday's git commits,
      check open GitHub issues assigned to me,
      create standup report and post to #standup Slack channel
```

### Code Review Assistant

```bash
# Review pull request
hermes "review PR #123 on GitHub and provide feedback"

# Continuous code quality
hermes "monitor the main branch and flag potential issues after each push"
```

### Documentation Generator

```bash
# Generate docs from code
hermes "analyze src/ and generate API documentation in docs/"

# Keep docs in sync
hermes "watch src/ and update docs/ whenever code changes"
```

## Ecosystem Resources

- **Official docs**: https://hermes-agent.nousresearch.com/docs/
- **Discord community**: https://discord.gg/NousResearch
- **Skills hub**: https://agentskills.io
- **Release notes**: https://github.com/NousResearch/hermes-agent/releases
- **Awesome list**: https://github.com/0xNyk/awesome-hermes-agent

## Maturity Tags

When exploring the ecosystem, note these maturity indicators:
- **production** — Stable, documented, actively maintained
- **beta** — Works but evolving, expect some rough edges
- **experimental** — Proof of concept, learn from it but don't depend on it

## Getting Help

```bash
# Built-in help
hermes --help

# Skill-specific help
hermes "explain the youtube-search skill"

# Community support
# Post in Discord #hermes-agent channel with:
hermes "generate a debug report" > debug.txt
```
