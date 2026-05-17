---
name: agent-orchestrator-parallel-coding
description: Orchestrate parallel AI coding agents across git worktrees for autonomous CI fixes, code reviews, and PR management
triggers:
  - set up agent orchestrator for parallel coding
  - spawn multiple AI agents to work on different issues
  - configure autonomous CI failure handling
  - create isolated git worktrees for parallel agents
  - automate code review responses with AI agents
  - orchestrate multi-agent coding tasks
  - manage fleet of coding agents with reactions
  - run parallel AI agents on different branches
---

# Agent Orchestrator — Parallel AI Coding Agents

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agent Orchestrator is an agentic orchestration layer that spawns parallel AI coding agents, each in its own git worktree. Agents autonomously fix CI failures, address review comments, and open PRs while you supervise from a unified dashboard.

## Core Concepts

- **Orchestrator Agent**: Plans tasks, spawns worker agents, monitors progress
- **Worker Agents**: Isolated agents working on individual issues/features
- **Git Worktrees**: Each agent gets its own filesystem workspace
- **Reactions**: Automated responses to CI failures, review comments, approvals
- **Plugin Architecture**: Extensible runtime, agent, workspace, tracker, SCM, notifier, terminal plugins

## Installation

### Global CLI Installation

```bash
npm install -g @aoagents/ao
```

For nightly builds from main:

```bash
npm install -g @aoagents/ao@nightly
```

### From Source (Contributors)

```bash
git clone https://github.com/ComposioHQ/agent-orchestrator.git
cd agent-orchestrator
bash scripts/setup.sh
```

### Prerequisites

- Node.js 20+
- Git 2.25+ (worktree support)
- `gh` CLI (authenticated)
- **macOS/Linux**: tmux (`brew install tmux` or `sudo apt install tmux`)
- **Windows**: PowerShell 7+ (uses native ConPTY, no tmux needed)

## Quick Start

### Start from GitHub URL

```bash
ao start https://github.com/your-org/your-repo
```

### Start from Local Repo

```bash
cd ~/your-project
ao start
```

### Start Multiple Projects

```bash
ao start ~/path/to/project-one
ao start https://github.com/org/project-two
```

The dashboard opens at `http://localhost:3000` by default.

## Configuration

On first run, `ao start` generates `agent-orchestrator.yaml`:

```yaml
$schema: https://raw.githubusercontent.com/ComposioHQ/agent-orchestrator/main/schema/config.schema.json
port: 3000

defaults:
  runtime: tmux       # tmux on macOS/Linux, process on Windows
  agent: claude-code  # or codex, aider, cursor, opencode, kimicode
  workspace: worktree # or clone
  notifiers: [desktop]

projects:
  my-app:
    repo: owner/my-app
    path: ~/my-app
    defaultBranch: main
    sessionPrefix: app
    
reactions:
  ci-failed:
    auto: true
    action: send-to-agent
    retries: 2
  changes-requested:
    auto: true
    action: send-to-agent
    escalateAfter: 30m
  approved-and-green:
    auto: false  # Set to true for auto-merge
    action: notify

power:
  preventIdleSleep: true  # Keeps Mac awake for remote access
```

### Multi-Project Configuration

```yaml
projects:
  frontend:
    repo: org/frontend
    path: ~/work/frontend
    defaultBranch: main
    sessionPrefix: fe
    
  backend:
    repo: org/backend
    path: ~/work/backend
    defaultBranch: develop
    sessionPrefix: be
    
  docs:
    repo: org/documentation
    path: ~/work/docs
    defaultBranch: main
    sessionPrefix: docs
```

## CLI Commands

### Session Management

```bash
# List all active sessions
ao list

# Start a new agent session
ao new "Fix login validation bug" --project my-app --issue 123

# Attach to existing session
ao attach my-app-fix-login

# Stop a session
ao stop my-app-fix-login

# Stop all sessions for a project
ao stop --project my-app --all
```

### Project Management

```bash
# Add project
ao add-project ~/path/to/repo

# Remove project
ao remove-project my-app

# Show configuration
ao config-help
```

### Status & Monitoring

```bash
# Show all sessions and their status
ao status

# Show detailed session info
ao status my-app-fix-login

# View logs
ao logs my-app-fix-login
```

### Zsh Completion

```bash
# Generate completion file
mkdir -p ~/.zsh/completions
ao completion zsh > ~/.zsh/completions/_ao

# Add to ~/.zshrc before compinit
fpath=(~/.zsh/completions $fpath)
autoload -Uz compinit
compinit
```

## Plugin Configuration

### Runtime Plugins

```yaml
defaults:
  runtime: tmux  # or process, docker
```

**tmux** (default on macOS/Linux):
- Persistent sessions
- Attach from multiple terminals
- Scrollback history

**process** (default on Windows):
- Native ConPTY support
- No tmux dependency
- Set `AO_SHELL=bash` for Git Bash

**docker**:
- Complete isolation
- Reproducible environments

### Agent Plugins

```yaml
defaults:
  agent: claude-code  # or codex, aider, cursor, opencode, kimicode
```

Each agent plugin implements the `AgentPlugin` interface:

```typescript
export interface AgentPlugin {
  name: string;
  start(config: AgentConfig): Promise<void>;
  stop(sessionId: string): Promise<void>;
  sendMessage(sessionId: string, message: string): Promise<void>;
  getStatus(sessionId: string): Promise<AgentStatus>;
}
```

### Workspace Plugins

```yaml
defaults:
  workspace: worktree  # or clone
```

**worktree**: Single repo, multiple working directories (recommended)
**clone**: Full repository clone per session

### Tracker Plugins

```yaml
defaults:
  tracker: github  # or linear, gitlab
```

Integration with issue tracking systems to fetch context and update status.

### Notifier Plugins

```yaml
defaults:
  notifiers: [desktop, slack]  # Available: desktop, slack, discord, composio, webhook, openclaw
```

Multiple notifiers can be active simultaneously.

## Reaction Patterns

### Auto-Fix CI Failures

```yaml
reactions:
  ci-failed:
    auto: true
    action: send-to-agent
    retries: 2
    message: "CI failed with the following errors: {{errors}}"
```

When CI fails, the orchestrator sends logs to the agent automatically. After 2 retries, escalation occurs.

### Auto-Address Review Comments

```yaml
reactions:
  changes-requested:
    auto: true
    action: send-to-agent
    escalateAfter: 30m
    message: "Reviewer requested: {{comments}}"
```

The agent gets review comments and attempts to address them. If not resolved in 30 minutes, you're notified.

### Auto-Merge on Approval

```yaml
reactions:
  approved-and-green:
    auto: true  # Enable auto-merge
    action: merge
    strategy: squash  # or merge, rebase
    deleteAfter: true
```

When PR is approved and CI passes, automatically merge and delete the branch.

### Custom Reactions

```yaml
reactions:
  label-added:
    auto: true
    condition: "label == 'needs-tests'"
    action: send-to-agent
    message: "Please add tests for this change"
```

## Working with Sessions

### Creating a Session Programmatically

```typescript
import { SessionManager } from '@aoagents/core';

const manager = new SessionManager();

const session = await manager.create({
  projectId: 'my-app',
  name: 'fix-auth-bug',
  issue: 'GH-456',
  branch: 'fix/auth-validation',
  agent: 'claude-code',
  runtime: 'tmux',
  workspace: 'worktree'
});

// Session is now running in isolated worktree
console.log(`Session ${session.id} started at ${session.workspacePath}`);
```

### Sending Messages to Agent

```bash
# Via CLI
ao send my-app-fix-auth "Review the authentication flow in src/auth.ts"

# Via API
ao api POST /sessions/my-app-fix-auth/message -d '{"content":"Check the auth flow"}'
```

### Monitoring Session Status

```typescript
import { SessionManager } from '@aoagents/core';

const manager = new SessionManager();
const status = await manager.getStatus('my-app-fix-auth');

console.log(`Agent: ${status.agent}`);
console.log(`State: ${status.state}`);  // idle, working, waiting, error
console.log(`Current task: ${status.currentTask}`);
console.log(`Branch: ${status.branch}`);
console.log(`Workspace: ${status.workspacePath}`);
```

## Git Worktree Management

Agent Orchestrator uses git worktrees to isolate agents:

```bash
# Worktrees are automatically created under:
~/my-app/.worktrees/fix-auth-bug/

# Each worktree has its own:
# - Working directory
# - HEAD pointer
# - Index
# - Branch
```

### Worktree Cleanup

```bash
# Cleanup happens automatically on session stop
ao stop my-app-fix-auth

# Manual cleanup if needed
cd ~/my-app
git worktree remove .worktrees/fix-auth-bug
git branch -D fix/auth-validation
```

## Dashboard API

The orchestrator exposes a REST API (default port 3000):

### List Sessions

```bash
curl http://localhost:3000/api/sessions
```

### Create Session

```bash
curl -X POST http://localhost:3000/api/sessions \
  -H "Content-Type: application/json" \
  -d '{
    "projectId": "my-app",
    "issue": "GH-789",
    "title": "Add user export feature"
  }'
```

### Get Session Status

```bash
curl http://localhost:3000/api/sessions/my-app-export/status
```

### Send Message

```bash
curl -X POST http://localhost:3000/api/sessions/my-app-export/message \
  -H "Content-Type: application/json" \
  -d '{"content": "Export to CSV format"}'
```

### Stop Session

```bash
curl -X DELETE http://localhost:3000/api/sessions/my-app-export
```

## Building Plugins

### Agent Plugin Example

```typescript
// plugins/agent-custom/src/index.ts
import { AgentPlugin, AgentConfig, AgentStatus } from '@aoagents/core';

export class CustomAgentPlugin implements AgentPlugin {
  name = 'custom-agent';
  
  async start(config: AgentConfig): Promise<void> {
    // Initialize agent in workspace
    const { workspacePath, sessionId, task } = config;
    
    // Launch agent process
    // Set up communication channel
    // Return when ready
  }
  
  async stop(sessionId: string): Promise<void> {
    // Cleanup agent process
    // Save state if needed
  }
  
  async sendMessage(sessionId: string, message: string): Promise<void> {
    // Forward message to agent
  }
  
  async getStatus(sessionId: string): Promise<AgentStatus> {
    return {
      state: 'working',
      currentTask: 'Analyzing codebase',
      lastActivity: new Date()
    };
  }
}

export default {
  agent: new CustomAgentPlugin()
};
```

### Notifier Plugin Example

```typescript
// plugins/notifier-telegram/src/index.ts
import { NotifierPlugin, Notification } from '@aoagents/core';

export class TelegramNotifierPlugin implements NotifierPlugin {
  name = 'telegram';
  
  constructor(private botToken: string, private chatId: string) {}
  
  async notify(notification: Notification): Promise<void> {
    const { title, message, level, sessionId } = notification;
    
    const text = `*${title}*\n${message}\nSession: ${sessionId}`;
    
    await fetch(`https://api.telegram.org/bot${this.botToken}/sendMessage`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        chat_id: this.chatId,
        text,
        parse_mode: 'Markdown'
      })
    });
  }
}

export default {
  notifier: new TelegramNotifierPlugin(
    process.env.TELEGRAM_BOT_TOKEN!,
    process.env.TELEGRAM_CHAT_ID!
  )
};
```

## Environment Variables

```bash
# Agent credentials
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...

# GitHub integration
export GITHUB_TOKEN=ghp_...

# Notifier credentials
export SLACK_WEBHOOK_URL=https://hooks.slack.com/...
export DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...

# Windows shell preference
export AO_SHELL=bash  # Use Git Bash instead of PowerShell

# Custom config location
export AO_CONFIG=~/custom/agent-orchestrator.yaml
```

## Troubleshooting

### Worktree Already Exists

```bash
# Error: worktree already exists
# Solution: Clean up manually
git worktree remove .worktrees/session-name --force
git branch -D branch-name
```

### tmux Not Found (macOS/Linux)

```bash
# Install tmux
brew install tmux       # macOS
sudo apt install tmux   # Ubuntu/Debian

# Or switch to process runtime
ao config set defaults.runtime process
```

### Agent Not Responding

```bash
# Check session logs
ao logs session-name

# Check agent process
ao status session-name

# Restart session
ao stop session-name
ao new "Same task" --project my-app --issue 123
```

### Port Already in Use

```yaml
# agent-orchestrator.yaml
port: 3001  # Change from default 3000
```

### GitHub Auth Issues

```bash
# Re-authenticate gh CLI
gh auth login

# Verify token has required scopes
gh auth status
```

### Merge Conflicts in Worktree

```bash
# Agent handles automatically via reaction
# Or attach and resolve manually
ao attach session-name
# Inside session: resolve conflicts, commit, push
```

## Best Practices

1. **One issue per session**: Each agent works on a single, well-defined task
2. **Use worktrees**: Better isolation and performance than clones
3. **Configure reactions**: Automate CI fixes and review responses
4. **Monitor dashboard**: Track progress across all agents
5. **Escalate complex decisions**: Let agents handle routine, you handle judgment calls
6. **Clean up regularly**: Remove merged branches and completed worktrees
7. **Test reactions**: Start with `auto: false`, validate behavior, then enable
8. **Use session prefixes**: Organize sessions by project for clarity

## Remote Access

```yaml
# Keep Mac awake for remote dashboard access
power:
  preventIdleSleep: true  # Default on macOS
```

Access dashboard remotely via Tailscale or VPN:
- `http://your-mac-tailscale-ip:3000`

**Note**: Lid-close sleep cannot be prevented on macOS. Use clamshell mode (external display + power) for lid-closed access.

## Development & Testing

```bash
# Clone and build
git clone https://github.com/ComposioHQ/agent-orchestrator.git
cd agent-orchestrator
pnpm install
pnpm build

# Run tests (3,288 test cases)
pnpm test

# Start dev server
pnpm dev

# Build plugins
cd packages/plugin-agent-custom
pnpm build
```

## Advanced: Custom Orchestrator Agent

Create a custom orchestrator that uses different planning logic:

```typescript
// orchestrator-custom.ts
import { OrchestratorAgent, Task, SessionManager } from '@aoagents/core';

class CustomOrchestrator extends OrchestratorAgent {
  async plan(goal: string): Promise<Task[]> {
    // Custom planning logic
    const tasks = await this.breakdownGoal(goal);
    
    return tasks.map(task => ({
      id: this.generateId(),
      title: task.title,
      description: task.description,
      dependencies: task.deps,
      estimatedEffort: task.effort
    }));
  }
  
  async spawn(task: Task): Promise<string> {
    const manager = new SessionManager();
    
    const session = await manager.create({
      projectId: this.projectId,
      name: this.toSessionName(task.title),
      issue: task.id,
      branch: this.toBranchName(task.title),
      agent: this.selectAgent(task),
      runtime: 'tmux',
      workspace: 'worktree'
    });
    
    return session.id;
  }
}
```

This skill covers the complete usage of Agent Orchestrator for spawning and managing parallel AI coding agents with autonomous reactions, git worktree isolation, and extensible plugin architecture.
