---
name: lark-coding-agent-bridge
description: Bridge Feishu/Lark messenger with local Claude Code or Codex CLI for chat-based coding assistance with streaming cards and session management
triggers:
  - set up lark bridge for claude code
  - connect feishu to my local coding agent
  - configure lark channel bridge
  - manage lark bot workspaces and sessions
  - troubleshoot lark coding agent bridge
  - add users to lark bot access control
  - switch workspace in lark bridge
  - debug lark bridge connection issues
---

# lark-coding-agent-bridge

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A lightweight bot that bridges Feishu/Lark messenger with your local Claude Code or Codex CLI. Send messages in Feishu and get streaming responses with tool calls updating in real-time cards. Each chat/topic maintains its own session with support for multiple workspaces, queueing, and file attachments.

## What It Does

- **Message forwarding**: DM the bot or `@mention` in groups to talk to your local coding agent
- **Streaming cards**: Real-time updates of text replies and tool calls on a single Lark card
- **Session continuity**: Each chat, topic, or document comment thread maintains its own session
- **Queue management**: Messages sent quickly are batched; commands like `/new`, `/cd`, `/stop` interrupt current runs
- **Multiple workspaces**: Switch projects with `/cd` and save common directories with `/ws`
- **Media support**: Send images and files directly to the bot
- **Access control**: Private by default with granular user/group permissions

## Prerequisites

- Node.js >= 20.12.0
- At least one local agent installed and authenticated:
  - Claude Code: `claude` (https://docs.anthropic.com/en/docs/claude-code/quickstart)
  - Codex CLI: `codex` (https://developers.openai.com/codex/cli)
- A Feishu/Lark PersonalAgent app (QR wizard creates one on first run)

## Installation

```bash
# Global installation (recommended for background service)
npm i -g lark-channel-bridge

# Or with pnpm
pnpm add -g lark-channel-bridge

# Or run directly with npx (foreground only)
npx lark-channel-bridge run
```

## First-Time Setup

```bash
# Start the QR wizard
lark-channel-bridge run

# With existing app ID
lark-channel-bridge run --app-id cli_xxx

# For Lark global apps
lark-channel-bridge run --app-id cli_xxx --tenant lark

# Specify agent type explicitly
lark-channel-bridge run --agent claude

# Set initial workspace
lark-channel-bridge run --workspace /path/to/project
```

The QR wizard:
1. Displays QR code in terminal
2. Scan with Feishu/Lark app
3. Pick/create a PersonalAgent app
4. Choose agent to initialize (claude or codex)
5. Config saved to `~/.lark-channel/config.json`

## Background Service Management

```bash
# Start background service (macOS: launchd, Linux: systemd, Windows: Task Scheduler)
lark-channel-bridge start

# Check status
lark-channel-bridge status

# Stop service
lark-channel-bridge stop

# Restart service
lark-channel-bridge restart

# Remove service registration
lark-channel-bridge unregister
```

**Important**: Install globally before using service commands. The daemon records the CLI path, and `npx` temp cache paths break when cleaned.

### Multiple Profiles (Claude + Codex)

```bash
# Create separate profiles for different agents
lark-channel-bridge profile create claude --agent claude
lark-channel-bridge profile create codex --agent codex

# List all profiles
lark-channel-bridge profile list

# Switch active profile
lark-channel-bridge profile use claude

# Start specific profile
lark-channel-bridge start --profile claude
lark-channel-bridge start --profile codex

# Check specific profile status
lark-channel-bridge status --profile codex

# Remove profile (archives by default)
lark-channel-bridge profile remove codex

# Permanently delete profile
lark-channel-bridge profile remove codex --purge --yes

# Export profile (secrets redacted)
lark-channel-bridge profile export claude --output ./claude-profile.json

# Export with secrets
lark-channel-bridge profile export claude --include-secrets --yes
```

## In-Chat Slash Commands

### Session Management

```
/new              # Clear current session
/reset            # Alias for /new
/cd /path/to/dir  # Switch working directory and reset session
/resume           # Resume compatible history for same agent/directory/permissions
/timeout 30       # Set 30-minute idle timeout
/timeout off      # Disable timeout
/timeout default  # Restore default timeout
/stop             # Stop current run
```

### Workspace Management

```
/ws list                    # List all named workspaces
/ws save myproject         # Save current directory as "myproject"
/ws use myproject          # Switch to "myproject" workspace
/ws remove myproject       # Delete "myproject" workspace
```

### Access Control

```
/invite user @username      # Allow user to DM the bot
/invite admin @username     # Add access-control admin
/invite group              # Allow current group to use bot
/invite all group          # Allow all groups bot has joined
/remove user @username     # Remove user access
/remove admin @username    # Remove admin
/remove group              # Remove current group access
```

### Diagnostics & Info

```
/status        # Show profile, agent, working directory, session state
/config        # Adjust preferences and view access panel
/help          # Display help card
/ps            # List local bridge processes
/exit <id|#>   # Stop a bridge process
/reconnect     # Force WebSocket reconnect
/doctor        # Run diagnostics (include description for context)
```

## Configuration

### Profile Structure

Config location: `~/.lark-channel/config.json`

```json
{
  "activeProfile": "default",
  "profiles": {
    "default": {
      "name": "default",
      "agent": "claude",
      "feishu": {
        "appId": "cli_xxx",
        "tenant": "feishu"
      },
      "workspaces": {
        "default": "/Users/me/.lark-channel-workspaces/claude/default",
        "named": {
          "myproject": "/Users/me/projects/myproject",
          "webapp": "/Users/me/work/webapp"
        }
      },
      "permissions": {
        "defaultAccess": "full",
        "maxAccess": "full"
      }
    }
  }
}
```

### Permission Modes

Edit the profile's `permissions` field:

```json
{
  "permissions": {
    "defaultAccess": "full",
    "maxAccess": "full"
  }
}
```

Mode mappings:

| Bridge Access | Claude Mode | Codex Mode | Capabilities |
|--------------|-------------|------------|--------------|
| `full` | `bypassPermissions` | `danger-full-access` | All tools, auth flows, file writes |
| `workspace` | `acceptEdits` | `workspace-write` | Limited to workspace directory |
| `read-only` | `plan` | `read-only` | No file writes or dangerous operations |

### Working Directories

The bridge validates directories exist, are actual directories, and aren't overly broad (not `/`, home root, system dirs, temp roots).

```typescript
// Profile field snippet - edit matching profile's workspaces
{
  "workspaces": {
    "default": "/Users/me/projects/default",
    "named": {
      "backend": "/Users/me/work/backend-api",
      "frontend": "/Users/me/work/web-client",
      "mobile": "/Users/me/work/mobile-app"
    }
  }
}
```

Switch in chat:
```
/cd /Users/me/work/new-project
/ws save new-project
/ws use backend
```

## Process Management

```bash
# List all running bridge processes
lark-channel-bridge ps

# Kill specific process by ID or index
lark-channel-bridge kill 12345
lark-channel-bridge kill 1  # First in list
```

In Feishu chat:
```
/ps         # List processes
/exit 1     # Stop first process
/exit 12345 # Stop by PID
```

## Data Directories

```
~/.lark-channel/
├── config.json                              # Root config with profiles
├── active-profile                           # Last selected profile
├── profiles/
│   └── <profile>/
│       ├── sessions.json                    # Session state
│       ├── sessions.json.catalog.json       # Agent-aware session catalog
│       ├── workspaces.json                  # Workspace bindings
│       ├── secrets.enc                      # Encrypted secrets
│       ├── media/                           # Attachment cache
│       └── logs/
│           ├── daemon/                      # Background service logs
│           └── <timestamp>/                 # Structured run logs
└── registry/
    ├── processes.json                       # Process registry
    └── locks/                               # Profile and app locks
```

### Environment Variables

```bash
# Move all local bridge state
export LARK_CHANNEL_HOME=/custom/path/to/state

# Override log retention (default: 7 days)
export LARK_CHANNEL_LOG_DAYS=30
```

## Code Examples

### Starting Multiple Agent Profiles

```bash
#!/bin/bash
# setup-dual-agents.sh - Run both Claude and Codex bridges

# Create profiles if they don't exist
lark-channel-bridge profile create claude --agent claude --workspace ~/projects
lark-channel-bridge profile create codex --agent codex --workspace ~/projects

# Start both services
lark-channel-bridge start --profile claude
lark-channel-bridge start --profile codex

# Verify both are running
lark-channel-bridge status --profile claude
lark-channel-bridge status --profile codex
```

### Scripted Deployment

```bash
#!/bin/bash
# deploy-lark-bridge.sh - Automated deployment

set -e

PROFILE_NAME="${1:-production}"
AGENT_TYPE="${2:-claude}"
WORKSPACE_PATH="${3:-$HOME/workspace}"
APP_ID="${LARK_APP_ID}"  # From environment
APP_SECRET="${LARK_APP_SECRET}"  # From environment

if [ -z "$APP_ID" ] || [ -z "$APP_SECRET" ]; then
  echo "Error: Set LARK_APP_ID and LARK_APP_SECRET"
  exit 1
fi

# Create profile
lark-channel-bridge profile create "$PROFILE_NAME" \
  --agent "$AGENT_TYPE" \
  --workspace "$WORKSPACE_PATH"

# Use the new profile
lark-channel-bridge profile use "$PROFILE_NAME"

# Start with app credentials (will prompt for secret if interactive)
lark-channel-bridge start --profile "$PROFILE_NAME" --app-id "$APP_ID"

echo "Bridge deployed as profile: $PROFILE_NAME"
```

### Health Check Script

```bash
#!/bin/bash
# healthcheck.sh - Monitor bridge health

PROFILE="${1:-default}"

# Check if service is running
if ! lark-channel-bridge status --profile "$PROFILE" | grep -q "running"; then
  echo "Bridge not running, attempting restart..."
  lark-channel-bridge restart --profile "$PROFILE"
  
  # Wait and verify
  sleep 5
  if lark-channel-bridge status --profile "$PROFILE" | grep -q "running"; then
    echo "Bridge restarted successfully"
    exit 0
  else
    echo "Bridge failed to restart"
    exit 1
  fi
fi

echo "Bridge healthy"
exit 0
```

### Programmatic Configuration Update

```typescript
// update-config.ts - Modify profile settings programmatically
import { readFileSync, writeFileSync } from 'fs';
import { homedir } from 'os';
import { join } from 'path';

interface Profile {
  name: string;
  agent: 'claude' | 'codex';
  permissions?: {
    defaultAccess: 'full' | 'workspace' | 'read-only';
    maxAccess: 'full' | 'workspace' | 'read-only';
  };
  workspaces?: {
    default: string;
    named?: Record<string, string>;
  };
}

interface Config {
  activeProfile: string;
  profiles: Record<string, Profile>;
}

const configPath = join(homedir(), '.lark-channel', 'config.json');

function updateProfilePermissions(
  profileName: string,
  mode: 'full' | 'workspace' | 'read-only'
): void {
  const config: Config = JSON.parse(readFileSync(configPath, 'utf-8'));
  
  if (!config.profiles[profileName]) {
    throw new Error(`Profile ${profileName} not found`);
  }
  
  config.profiles[profileName].permissions = {
    defaultAccess: mode,
    maxAccess: mode,
  };
  
  writeFileSync(configPath, JSON.stringify(config, null, 2));
  console.log(`Updated ${profileName} to ${mode} mode`);
}

function addWorkspace(
  profileName: string,
  name: string,
  path: string
): void {
  const config: Config = JSON.parse(readFileSync(configPath, 'utf-8'));
  
  if (!config.profiles[profileName]) {
    throw new Error(`Profile ${profileName} not found`);
  }
  
  if (!config.profiles[profileName].workspaces) {
    config.profiles[profileName].workspaces = {
      default: path,
      named: {},
    };
  }
  
  if (!config.profiles[profileName].workspaces!.named) {
    config.profiles[profileName].workspaces!.named = {};
  }
  
  config.profiles[profileName].workspaces!.named![name] = path;
  
  writeFileSync(configPath, JSON.stringify(config, null, 2));
  console.log(`Added workspace ${name}: ${path} to ${profileName}`);
}

// Example usage
updateProfilePermissions('production', 'workspace');
addWorkspace('production', 'api', '/home/user/projects/api');
```

## Common Patterns

### Multi-Workspace Development

```
# Start with default workspace
lark-channel-bridge run

# In Feishu, save current projects
/cd ~/work/backend
/ws save backend

/cd ~/work/frontend
/ws save frontend

/cd ~/work/mobile
/ws save mobile

# Switch between them quickly
/ws use backend
# ... do backend work ...

/ws use frontend
# ... do frontend work ...

# View all workspaces
/ws list
```

### Team Setup with Access Control

```
# Creator sets up bot
lark-channel-bridge run --app-id cli_xxx

# Add team leads as admins (they can manage access)
/invite admin @tech-lead
/invite admin @product-lead

# Allow specific developers DM access
/invite user @developer1
/invite user @developer2

# Allow bot in team channels
# (In each channel, as creator or admin)
/invite group

# Or allow all groups at once
/invite all group
```

### Locked-Down Production Setup

```bash
# Create production profile with restricted permissions
lark-channel-bridge profile create production \
  --agent claude \
  --workspace /opt/production/workspace

# Edit config to lock down permissions
cat > /tmp/update-perms.json << 'EOF'
{
  "permissions": {
    "defaultAccess": "read-only",
    "maxAccess": "workspace"
  }
}
EOF

# Merge into config (manual edit or script)
# Then start with read-only mode
lark-channel-bridge start --profile production
```

In chat, users can still ask questions and get code suggestions, but the agent won't execute dangerous operations or write files outside the workspace.

### Session Recovery After Restart

```
# After bridge restart, sessions persist automatically
# Resume last session in current chat
/resume

# Or start fresh
/new

# Check current state
/status
```

## Troubleshooting

### Bridge Won't Start

```bash
# Check if another instance is running
lark-channel-bridge ps

# Kill conflicting processes
lark-channel-bridge kill 1

# Check service status
lark-channel-bridge status

# View daemon logs
tail -f ~/.lark-channel/profiles/default/logs/daemon/*.log

# Verify Node.js version
node --version  # Must be >= 20.12.0

# Reinstall globally
npm uninstall -g lark-channel-bridge
npm i -g lark-channel-bridge
```

### Bot Not Responding in Feishu

```
# In chat, check connection
/status
/reconnect

# Verify app credentials
cat ~/.lark-channel/config.json | grep appId

# Run diagnostics
/doctor Cannot receive messages in group chat

# Check if group is allowed
/config  # View access panel
/invite group  # Add current group
```

### Wrong Agent Type

```bash
# Stop service first
lark-channel-bridge stop --profile wrong-agent

# Remove profile
lark-channel-bridge profile remove wrong-agent

# Recreate with correct agent
lark-channel-bridge profile create correct-agent --agent codex

# Start new profile
lark-channel-bridge start --profile correct-agent
```

### Session Not Resuming

```
# Sessions are agent + workspace + permission mode specific
# Check current state
/status

# If workspace or permissions changed, /resume won't find history
# Either:
# 1. Switch back to original workspace
/ws use original-project

# 2. Or start fresh
/new
```

### Permission Denied Errors

```
# Check current permission mode
/status

# If agent complains about restricted access:
# 1. Increase permission level (edit config)
{
  "permissions": {
    "defaultAccess": "workspace",  # or "full"
    "maxAccess": "workspace"       # or "full"
  }
}

# 2. Restart bridge
lark-channel-bridge restart

# 3. Start new session with new permissions
/new
```

### Working Directory Validation Failed

```
# Common causes:
# - Directory doesn't exist
# - Path is too broad (/, ~, /tmp, /usr, etc.)
# - Not a directory (is a file)

# Check current directory
/status

# Fix:
mkdir -p /path/to/valid/workspace
/cd /path/to/valid/workspace

# Or use an existing project
/cd ~/projects/my-app
```

### Media Files Not Working

```
# Media downloaded to profile-specific cache
~/.lark-channel/profiles/<profile>/media/

# If files aren't reaching agent:
# 1. Check bridge logs
tail -f ~/.lark-channel/profiles/default/logs/<latest>/bridge.log

# 2. Verify file permissions
ls -la ~/.lark-channel/profiles/default/media/

# 3. Ensure working directory allows file access
/status  # Check permission mode
```

### Process Won't Stop

```bash
# Service won't stop cleanly
lark-channel-bridge stop  # May hang

# Force kill
lark-channel-bridge ps
lark-channel-bridge kill 1  # Or specific PID

# Unregister and re-register service
lark-channel-bridge unregister
lark-channel-bridge start
```

### Log Retention Issues

```bash
# Logs filling disk
du -sh ~/.lark-channel/profiles/*/logs/

# Set retention period
export LARK_CHANNEL_LOG_DAYS=3

# Restart bridge
lark-channel-bridge restart

# Or manually clean old logs
find ~/.lark-channel/profiles/*/logs -type f -mtime +7 -delete
```

### Multiple Profiles Conflict

```bash
# Check all running profiles
lark-channel-bridge ps

# Ensure each profile uses unique app ID
cat ~/.lark-channel/config.json | jq '.profiles[].feishu.appId'

# If same app ID in multiple profiles, one must be removed
lark-channel-bridge profile remove duplicate
```

This skill enables AI coding agents to help developers set up, configure, and troubleshoot the Lark Channel Bridge for seamless chat-based coding assistance.
