---
name: mercury-agent-deployment
description: Deploy and configure Mercury Agent, a soul-driven AI agent with permission-hardened tools, token budgets, and multi-channel access
triggers:
  - "set up mercury agent"
  - "configure ai agent with telegram"
  - "deploy mercury with daemon mode"
  - "install mercury agent cli"
  - "configure second brain memory"
  - "set up mercury permissions"
  - "create mercury agent skill"
  - "schedule mercury agent tasks"
---

# Mercury Agent Deployment

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Mercury is a soul-driven AI agent framework with permission-hardened tools, token budgets, multi-channel access (CLI + Telegram), persistent memory (Second Brain), and 24/7 daemon mode. It runs TypeScript-based agents with 31 built-in tools, extensible skills, and SQLite-backed memory.

## Installation

### Quick Start (npx)

```bash
npx @cosmicstack/mercury-agent
```

### Global Installation

```bash
npm i -g @cosmicstack/mercury-agent
mercury
```

First run triggers setup wizard for:
- Agent name
- LLM provider (OpenAI, Anthropic, etc.)
- Optional Telegram bot integration
- Permission defaults

## Core Commands

### Daemon & Service Management

```bash
# Recommended: install service + start daemon
mercury up

# Start in foreground
mercury start

# Start as background daemon
mercury start -d

# Daemon control
mercury restart
mercury stop
mercury logs
mercury status

# System service (auto-start on boot)
mercury service install
mercury service status
mercury service uninstall
```

### Configuration

```bash
# Reconfigure setup
mercury doctor

# Platform diagnostics
mercury doctor --platform

# Re-run setup wizard
mercury setup

# Check status
mercury status

# Upgrade to latest
mercury upgrade
```

### Telegram Access Management

```bash
# List all users
mercury telegram list

# Approve pairing code or pending request
mercury telegram approve <code|id>

# Reject pending request
mercury telegram reject <id>

# Remove approved user
mercury telegram remove <id>

# Role management
mercury telegram promote <id>
mercury telegram demote <id>

# Reset all access
mercury telegram reset
```

## Configuration Files

All runtime data in `~/.mercury/`:

```
~/.mercury/
├── mercury.yaml              # Main config
├── .env                      # API keys
├── permissions.yaml          # Tool capabilities
├── token-usage.json          # Budget tracking
├── schedules.yaml            # Scheduled tasks
├── soul/                     # Personality files
│   ├── soul.md
│   ├── persona.md
│   ├── taste.md
│   └── heartbeat.md
├── skills/                   # Installed skills
├── memory/
│   ├── short-term/          # Conversation JSON
│   ├── long-term/           # Extracted facts (JSONL)
│   ├── episodic/            # Event log (JSONL)
│   └── second-brain/        # SQLite + FTS5
└── logs/
```

### Example mercury.yaml

```yaml
agentName: Mercury
soul:
  path: ~/.mercury/soul
providers:
  - type: openai
    model: gpt-4o
    apiKeyEnv: OPENAI_API_KEY
  - type: anthropic
    model: claude-3-5-sonnet-20241022
    apiKeyEnv: ANTHROPIC_API_KEY
channels:
  telegram:
    enabled: true
    tokenEnv: TELEGRAM_BOT_TOKEN
    persistence: ~/.mercury/telegram.json
permissions:
  defaultMode: ask
  filesystem:
    read: ask
    write: ask
    delete: ask
  shell:
    execute: ask
    blocklist:
      - sudo
      - rm -rf /
      - mkfs
      - dd if=
budget:
  daily: 200000
  warningThreshold: 0.7
memory:
  secondBrain:
    enabled: true
    dbPath: ~/.mercury/memory/second-brain/second-brain.db
```

### Example .env

```bash
# LLM Providers
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Telegram
TELEGRAM_BOT_TOKEN=123456:ABC-DEF...

# Optional: Spotify (for music control tools)
SPOTIFY_CLIENT_ID=...
SPOTIFY_CLIENT_SECRET=...

# Optional: Enable inline album art (iTerm only)
MERCURY_SPOTIFY_ART=1

# Optional: Disable Second Brain
SECOND_BRAIN_ENABLED=false
```

## In-Chat Commands

These work in both CLI and Telegram without consuming tokens:

```bash
/help                 # Show manual
/status              # Config, budget, usage
/tools               # List loaded tools
/skills              # List installed skills
/budget              # Token budget status
/budget override     # Override budget once
/budget reset        # Reset usage to zero
/budget set <n>      # Change daily limit
/permissions         # Toggle Ask Me / Allow All
/view                # Toggle balanced/detailed progress
/stream              # Toggle Telegram streaming
/code agent <task>   # Delegate coding task to sub-agent
/ws exit             # Exit workspace IDE mode
/tasks               # List scheduled tasks
/memory              # View/manage Second Brain
/unpair              # Telegram: reset access
```

## Built-in Tools

### Filesystem Tools

```typescript
// Mercury auto-requests permission for file operations
// Example conversation:
// User: "Read package.json"
// Mercury uses: read_file

// Approve folder scope for batch operations
// User: "Read all TypeScript files in src/"
// Mercury prompts: approve_scope for src/

// File operations
read_file({ path: "package.json" })
write_file({ path: "config.json", content: "{...}" })
create_file({ path: "new.ts", content: "export ..." })
edit_file({ path: "app.ts", operations: [...] })
list_dir({ path: "src/" })
delete_file({ path: "temp.txt" })
send_file({ path: "report.pdf" })  // Telegram only
```

### Shell Tools

```typescript
// Blocklist prevents dangerous commands
// Blocked: sudo, rm -rf /, mkfs, dd if=, etc.

run_command({ command: "npm install" })
cd({ path: "/path/to/project" })
approve_command({ command: "git push" })  // Pre-approve
```

### Git Tools

```typescript
git_status()
git_diff({ staged: true })
git_log({ limit: 10 })
git_add({ files: ["src/app.ts"] })
git_commit({ message: "feat: add feature" })
git_push()
```

### Web Tools

```typescript
fetch_url({ url: "https://api.example.com/data" })
```

### Messaging Tools

```typescript
// Send proactive messages
send_message({ 
  channel: "telegram", 
  content: "Build complete!" 
})
```

### Scheduler Tools

```typescript
// Recurring task (cron syntax)
schedule_task({
  name: "morning-standup",
  cron: "0 9 * * *",  // 9am daily
  task: "Send standup reminder"
})

// One-shot delayed task
schedule_task({
  name: "reminder",
  delay_seconds: 900,  // 15 minutes
  task: "Check deployment status"
})

list_scheduled_tasks()
cancel_scheduled_task({ name: "morning-standup" })
```

### Skill Management Tools

```typescript
// Install community skill
install_skill({ name: "web-search" })

// List installed
list_skills()

// Execute skill
use_skill({ 
  name: "web-search", 
  query: "latest TypeScript features" 
})
```

## Creating Custom Skills

Skills are markdown files in `~/.mercury/skills/<skill-name>/SKILL.md`:

### Example: GitHub Integration Skill

```bash
mkdir -p ~/.mercury/skills/github-pr-review
```

`~/.mercury/skills/github-pr-review/SKILL.md`:

```markdown
---
name: github-pr-review
description: Review GitHub pull requests and provide feedback
triggers:
  - "review this pull request"
  - "check github pr"
  - "analyze code changes"
---

# GitHub PR Review Skill

## Prerequisites

```bash
# Install GitHub CLI
brew install gh  # macOS
# or: sudo apt install gh  # Linux

# Authenticate
gh auth login
```

## Usage

When user requests PR review:

1. Fetch PR diff: `gh pr diff <number>`
2. Analyze changes for:
   - Code quality
   - Security issues
   - Breaking changes
   - Test coverage
3. Post review: `gh pr review <number> --comment -b "feedback"`

## Example Flow

```bash
# Get PR list
gh pr list

# Get diff
gh pr diff 123

# Review with approval
gh pr review 123 --approve -b "LGTM! Great work on error handling."

# Request changes
gh pr review 123 --request-changes -b "Please add tests for the new endpoint."
```

## Best Practices

- Always check CI status before review
- Look for security vulnerabilities in dependencies
- Verify backward compatibility
- Check for adequate test coverage
```

### Install Custom Skill

```bash
# Mercury auto-detects skills in ~/.mercury/skills/
# Or install programmatically:
mercury  # Start agent
# In chat:
# User: "Install the github-pr-review skill"
# Mercury uses: install_skill
```

## Second Brain Memory System

Mercury's persistent memory system with automatic extraction and recall.

### Memory Types

1. **identity** — Core facts about user (name, role, location)
2. **preference** — User preferences (tools, coding style, communication)
3. **goal** — Objectives and targets
4. **project** — Active projects and context
5. **habit** — Recurring patterns and routines
6. **decision** — Important choices made
7. **constraint** — Limitations and boundaries
8. **relationship** — People and connections
9. **episode** — Significant events
10. **reflection** — Patterns and insights

### Memory Lifecycle

```typescript
// Automatic after each conversation:
// 1. Extract 0-3 facts with confidence/importance/durability scores
// 2. Store in SQLite with FTS5 full-text search
// 3. Auto-consolidation every 60 minutes
// 4. Conflict resolution (higher confidence wins)
// 5. Auto-pruning (stale after 21 days for active-scope)
```

### Memory Commands

```bash
# In-chat memory management
/memory               # Show overview
/memory search AI     # Search memories
/memory pause         # Pause extraction
/memory resume        # Resume extraction
/memory clear         # Clear all memories

# Disable Second Brain entirely
# In .env:
SECOND_BRAIN_ENABLED=false

# Or in mercury.yaml:
memory:
  secondBrain:
    enabled: false
```

### Memory Data Structure

```typescript
// ~/.mercury/memory/second-brain/second-brain.db (SQLite)
// Tables:
// - memories: id, type, content, confidence, importance, durability
// - memories_fts: FTS5 index for full-text search
// - consolidations: profile summaries, active state, reflections

// Example memory record:
{
  type: "preference",
  content: "User prefers TypeScript over JavaScript for new projects",
  confidence: 0.95,
  importance: 0.8,
  durability: "durable",  // or "transient"
  scope: "active",        // or "background"
  extractedAt: "2026-05-16T10:30:00Z",
  lastAccessedAt: "2026-05-16T10:30:00Z"
}
```

## Permission System

### Permission Modes

```bash
# Ask Me: Prompt for each tool use
# Allow All: Auto-approve all tools

# Toggle in-chat:
/permissions

# Or configure default in mercury.yaml:
permissions:
  defaultMode: ask  # or: allow
```

### Permission Configuration

`~/.mercury/permissions.yaml`:

```yaml
filesystem:
  read: ask          # ask, allow, deny
  write: ask
  delete: ask
  scopes:
    - path: ~/projects/safe-dir
      read: allow
      write: allow

shell:
  execute: ask
  blocklist:
    - sudo
    - rm -rf /
    - mkfs
    - dd if=
    - "> /dev/"
  allowlist:
    - npm
    - git
    - node

messaging:
  send: allow

git:
  read: allow        # status, diff, log
  write: ask         # commit, push

web:
  fetch: ask
```

### Scope Approval

```bash
# User: "Update all TypeScript files in src/ to use strict mode"
# Mercury prompts:
# ┌─────────────────────────────────────────┐
# │ Approve folder scope?                   │
# │ Path: ~/project/src                     │
# │ Read: allow                             │
# │ Write: allow                            │
# │ → Yes  No  Allow All                    │
# └─────────────────────────────────────────┘

# Keyboard shortcuts:
# Y - Yes (approve this scope)
# N - No (deny)
# A - Allow All (switch to Allow All mode)
# Arrow keys + Enter
```

## Telegram Integration

### Bot Setup

1. Create bot with [@BotFather](https://t.me/botfather)
2. Get token from BotFather
3. Add to `~/.mercury/.env`:

```bash
TELEGRAM_BOT_TOKEN=123456:ABC-DEF...
```

4. Enable in `mercury.yaml`:

```yaml
channels:
  telegram:
    enabled: true
    tokenEnv: TELEGRAM_BOT_TOKEN
    persistence: ~/.mercury/telegram.json
```

### First-Time Pairing

```bash
# 1. Start Mercury
mercury up

# 2. In Telegram, send to your bot:
/start

# 3. Bot responds with pairing code: TG-ABC123

# 4. In CLI, approve:
mercury telegram approve TG-ABC123

# You're now the first admin!
```

### Multi-User Access

```bash
# Admin workflow:
# 1. New user sends /start to bot
# 2. Admin sees pending request in:
mercury telegram list

# 3. Approve or reject:
mercury telegram approve <user-id>
mercury telegram reject <user-id>

# Promote to admin:
mercury telegram promote <user-id>

# Demote to member:
mercury telegram demote <user-id>

# Remove user:
mercury telegram remove <user-id>
```

### Telegram Features

```typescript
// HTML formatting (bold, italic, code)
// Editable streaming messages (live token updates)
// File uploads (send_file tool)
// Typing indicators
// Multi-user with role-based access (admin/member)
// Private chat only (groups ignored)

// Example interaction:
// User: "Show me the last commit"
// Mercury: git_log → formatted HTML response with code blocks
```

## Token Budget Management

### Budget Configuration

```yaml
# mercury.yaml
budget:
  daily: 200000              # tokens per day
  warningThreshold: 0.7      # 70% warning
```

### Budget Commands

```bash
# Check status
/budget

# Override for one request
/budget override

# Reset usage
/budget reset

# Change daily limit
/budget set 300000
```

### Auto-Concise Mode

```typescript
// When usage > 70%, Mercury automatically:
// 1. Switches to concise response mode
// 2. Notifies user: "⚠️ 75% of daily budget used. Responses will be concise."
// 3. Continues normally but shorter responses

// Usage tracked in:
// ~/.mercury/token-usage.json
{
  "2026-05-16": {
    "used": 150000,
    "limit": 200000,
    "requests": 42
  }
}
```

## CLI Workspace Mode

### Keyboard Shortcuts

```bash
# In workspace/coding mode:
Ctrl+P        # Switch to Plan mode
Ctrl+X        # Switch to Execute mode
Esc / Ctrl+Q  # Exit workspace → general chat
Ctrl+V        # Toggle progress view

# Progress view modes:
# - balanced: compact, key steps only
# - detailed: full tool calls, reasoning, output
```

### Code Agent Delegation

```bash
# In-chat command:
/code agent implement user authentication with JWT

# Mercury spawns sub-agent in background:
# 1. Plans implementation
# 2. Executes code changes
# 3. Reports back with file changes
# 4. You review and approve
```

## Daemon Mode Production Setup

### Systemd (Linux)

```bash
# Install service
mercury service install

# Service file created at:
# ~/.config/systemd/user/mercury-agent.service

# Enable linger (start on boot without login)
sudo loginctl enable-linger $USER

# Service commands
systemctl --user status mercury-agent
systemctl --user restart mercury-agent
systemctl --user stop mercury-agent

# View logs
journalctl --user -u mercury-agent -f
```

### LaunchAgent (macOS)

```bash
# Install service
mercury service install

# Plist created at:
# ~/Library/LaunchAgents/org.cosmicstack.mercury-agent.plist

# Service commands
launchctl list | grep mercury
launchctl unload ~/Library/LaunchAgents/org.cosmicstack.mercury-agent.plist
launchctl load ~/Library/LaunchAgents/org.cosmicstack.mercury-agent.plist

# View logs
mercury logs
```

### Task Scheduler (Windows)

```bash
# Install service
mercury service install

# Task created via schtasks
# Location: Task Scheduler Library

# View task
schtasks /query /tn "MercuryAgent"

# Service commands via mercury CLI
mercury service status
mercury service uninstall
```

### Crash Recovery

```typescript
// Built-in exponential backoff:
// - Max 10 restarts per minute
// - Delay doubles on each crash: 1s, 2s, 4s, 8s, ...
// - Resets after 60s of stable run

// Logs crash details to:
// ~/.mercury/logs/daemon.log
```

## Common Patterns

### Morning Standup Automation

```bash
# Schedule daily standup report
mercury  # Start agent

# In chat:
# "Schedule a daily standup at 9am that checks:
# - Yesterday's git commits
# - Open GitHub issues assigned to me
# - Calendar for today
# and sends a summary to Telegram"

# Mercury uses: schedule_task with cron: "0 9 * * *"
```

### Project Context Persistence

```typescript
// Mercury's Second Brain remembers:
// - Current project: "Working on Mercury agent skills"
// - Preferences: "Prefers TypeScript, avoids any types"
// - Coding style: "Uses async/await, ESM imports"
// - Active goals: "Ship v1.2.0 by end of month"

// Example conversation:
// User: "Add a new skill for git operations"
// Mercury recalls project context, checks existing skills,
// suggests implementation that matches established patterns
```

### File Scope Workflow

```bash
# Efficient batch operations:
# User: "Refactor all components in src/components/ to use hooks"

# Mercury prompts once:
# "Approve folder scope for src/components/?"
# → Yes

# Then processes all files without re-prompting
```

### Multi-Channel Notifications

```typescript
// Use send_message for proactive updates:
// 1. Long-running build starts
// 2. Mercury sends: "Build started for v1.2.0"
// 3. Build completes
// 4. Mercury sends: "✅ Build complete in 3m 42s"

// Works in both CLI and Telegram
send_message({ 
  channel: "telegram",  // or "cli"
  content: "Deployment to production successful!" 
})
```

## Troubleshooting

### Agent Not Responding

```bash
# Check daemon status
mercury status

# View logs
mercury logs

# Restart daemon
mercury restart

# Or kill and start fresh
mercury stop
mercury up
```

### Telegram Bot Not Working

```bash
# Verify token
cat ~/.mercury/.env | grep TELEGRAM_BOT_TOKEN

# Check bot status with BotFather
# Ensure bot is not blocked

# Reset access and re-pair
mercury telegram reset
# Then send /start to bot again
```

### Permission Errors

```bash
# Check permissions.yaml
cat ~/.mercury/permissions.yaml

# Reset to defaults
mercury doctor

# Or edit manually:
nano ~/.mercury/permissions.yaml
```

### Token Budget Exceeded

```bash
# Check usage
/budget

# Reset today's usage
/budget reset

# Increase limit
/budget set 500000

# Or override once
/budget override
```

### Memory Database Issues

```bash
# Check Second Brain status
/memory

# Clear and rebuild
/memory clear

# Or disable entirely
# In .env:
echo "SECOND_BRAIN_ENABLED=false" >> ~/.mercury/.env
mercury restart
```

### Service Won't Start on Boot

```bash
# Linux: Enable linger
sudo loginctl enable-linger $USER

# macOS: Check LaunchAgent
launchctl list | grep mercury

# Windows: Verify task exists
schtasks /query /tn "MercuryAgent"

# Reinstall service
mercury service uninstall
mercury service install
```

### High Memory Usage

```bash
# Check daemon stats
ps aux | grep mercury

# Clear old conversation memory
rm -rf ~/.mercury/memory/short-term/*.json

# Restart daemon
mercury restart
```

### Skill Not Loading

```bash
# List installed skills
/skills

# Check skill directory
ls -la ~/.mercury/skills/

# Reinstall skill
mercury  # Start agent
# In chat: "Install the web-search skill"

# Or manually:
mkdir -p ~/.mercury/skills/my-skill
nano ~/.mercury/skills/my-skill/SKILL.md
```

## Resources

- **Homepage**: https://mercury.cosmicstack.org/
- **GitHub**: https://github.com/cosmicstack-labs/mercury-agent
- **npm**: https://www.npmjs.com/package/@cosmicstack/mercury-agent
- **Agent Skills**: https://agentskills.io
- **License**: MIT
