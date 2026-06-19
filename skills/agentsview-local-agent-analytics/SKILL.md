---
name: agentsview-local-agent-analytics
description: Local-first session search, analytics, and cost tracking for AI coding agents with SQLite storage
triggers:
  - track my AI agent token usage and costs
  - search through my coding agent sessions
  - analyze my AI coding assistant activity
  - view analytics for my agent conversations
  - export my Claude Code or Cursor sessions
  - set up local agent session tracking
  - monitor my AI coding agent spending
  - get daily cost breakdown for coding agents
---

# agentsview Local Agent Analytics

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

agentsview is a local-first tool for browsing, searching, and tracking costs across 20+ AI coding agents including Claude Code, Codex, Cursor, Copilot, and more. Everything runs locally with no accounts or cloud dependencies. Built in Go with SQLite storage, it provides a web UI for session browsing, full-text search, token usage analytics, and cost tracking.

## Installation

### Quick Install

```bash
# macOS / Linux
curl -fsSL https://agentsview.io/install.sh | bash

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://agentsview.io/install.ps1 | iex"
```

### Homebrew (macOS)

```bash
brew install --cask agentsview
```

### Docker

```bash
docker run --rm -p 127.0.0.1:8080:8080 \
  -v agentsview-data:/data \
  -v "$HOME/.claude/projects:/agents/claude:ro" \
  -v "$HOME/.codex/sessions:/agents/codex:ro" \
  -e CLAUDE_PROJECTS_DIR=/agents/claude \
  -e CODEX_DIR=/agents/codex \
  ghcr.io/kenn-io/agentsview:latest
```

### From Source

```bash
git clone https://github.com/kenn-io/agentsview.git
cd agentsview
go build -o agentsview ./cmd/agentsview
./agentsview serve
```

## Core Commands

### Server Management

```bash
# Start foreground server (blocks terminal)
agentsview serve

# Start background server
agentsview serve --background

# Check server status
agentsview serve status

# Stop background server
agentsview serve stop

# Serve on specific port
agentsview serve --port 9090

# Enable authentication (required for non-loopback)
agentsview serve --require-auth
```

### Token Usage & Cost Tracking

```bash
# Daily cost summary (last 30 days)
agentsview usage daily

# Per-model breakdown
agentsview usage daily --breakdown

# Filter by agent and date range
agentsview usage daily --agent claude --since 2026-04-01 --until 2026-04-30

# All-time summary
agentsview usage daily --all

# JSON output for scripting
agentsview usage daily --all --json

# One-line statusline for shell prompts
agentsview usage statusline

# Per-session usage
agentsview session usage <session-id>
agentsview session usage <session-id> --format json
```

### Session Analytics

```bash
# Human-readable stats (last 28 days)
agentsview stats

# JSON output with custom date range
agentsview stats --format json --since 2026-04-01 --until 2026-04-15

# Filter by agent
agentsview stats --agent claude

# Include git metrics (slow, opt-in)
agentsview stats --include-git-outcomes

# Include GitHub PR metrics (requires gh CLI)
agentsview stats --include-github-outcomes
```

### Database Operations

```bash
# Sync agent sessions into SQLite
agentsview sync

# Export to DuckDB
agentsview duckdb push --full

# Serve DuckDB mirror read-only
agentsview duckdb serve

# PostgreSQL mode
agentsview pg serve
```

## Configuration

### Environment Variables

```bash
# Agent session directories (auto-detected by default)
export CLAUDE_PROJECTS_DIR="$HOME/.claude/projects"
export CODEX_DIR="$HOME/.codex/sessions"
export CURSOR_DIR="$HOME/.cursor/sessions"
export FORGE_DIR="$HOME/.forge"
export COPILOT_DIR="$HOME/.copilot"

# Database
export AGENTSVIEW_DATA_DIR="$HOME/.agentsview"
export AGENTSVIEW_PG_URL="postgres://user:pass@host:5432/agentsview?sslmode=require"
export AGENTSVIEW_DUCKDB_URL="quack:https://duckdb.example.com"
export AGENTSVIEW_DUCKDB_TOKEN="your-quack-token"

# Server
export AGENTSVIEW_PORT="8080"
export AGENTSVIEW_PUBLIC_URL="http://127.0.0.1:8080"
```

### Remote Access Setup

For SSH port-forwarding, reverse proxies, or remote dev environments:

```bash
# Browser opens http://127.0.0.1:18080 via SSH tunnel
agentsview serve --public-url http://127.0.0.1:18080

# Codespaces, exe.dev, etc.
agentsview serve --public-url https://your-workspace.exe.dev

# Trust additional origins
agentsview serve --public-origin https://dev.example.com,https://staging.example.com
```

### Docker Compose Production Setup

```yaml
# docker-compose.prod.yaml
version: '3.8'

services:
  agentsview:
    image: ghcr.io/kenn-io/agentsview:latest
    ports:
      - "127.0.0.1:8080:8080"
    volumes:
      - agentsview-data:/data
      - ~/.claude/projects:/agents/claude:ro
      - ~/.codex/sessions:/agents/codex:ro
      - ~/.cursor/sessions:/agents/cursor:ro
    environment:
      - CLAUDE_PROJECTS_DIR=/agents/claude
      - CODEX_DIR=/agents/codex
      - CURSOR_DIR=/agents/cursor
    restart: unless-stopped

volumes:
  agentsview-data:
```

Run with:

```bash
docker compose -f docker-compose.prod.yaml up -d
```

## REST API

### Get Session Usage

```bash
# Fetch per-session token and cost data
curl http://127.0.0.1:8080/api/v1/sessions/<session-id>/usage

# Example response
{
  "session_id": "abc123",
  "agent": "claude",
  "project": "my-app",
  "total_output_tokens": 15420,
  "peak_context_tokens": 98765,
  "has_token_data": true,
  "cost_usd": 0.42,
  "has_cost": true,
  "models": ["claude-3.5-sonnet"],
  "unpriced_models": [],
  "server_running": true
}
```

### Settings & Configuration

```bash
# Get settings
curl http://127.0.0.1:8080/api/v1/settings

# Search sessions (FTS5)
curl "http://127.0.0.1:8080/api/v1/search?q=refactor+authentication"
```

## Common Patterns

### Daily Cost Monitoring in Shell Prompt

Add to `.bashrc` or `.zshrc`:

```bash
# Show today's AI agent cost in prompt
function agent_cost() {
  agentsview usage statusline 2>/dev/null || echo "agentsview offline"
}

PS1='$(agent_cost) $ '
```

### Automated Daily Reports

```bash
#!/bin/bash
# daily-agent-report.sh

DATE=$(date +%Y-%m-%d)
REPORT_FILE="$HOME/reports/agent-usage-$DATE.json"

agentsview usage daily --all --json > "$REPORT_FILE"
agentsview stats --format json >> "$REPORT_FILE"

echo "Report saved to $REPORT_FILE"
```

### CI/CD Cost Tracking

```yaml
# .github/workflows/agent-costs.yml
name: Track Agent Costs

on:
  schedule:
    - cron: '0 0 * * *' # Daily at midnight

jobs:
  track:
    runs-on: ubuntu-latest
    steps:
      - name: Install agentsview
        run: curl -fsSL https://agentsview.io/install.sh | bash
      
      - name: Mount agent sessions
        run: |
          mkdir -p ~/.claude/projects
          # Sync from cloud storage or artifact
      
      - name: Generate report
        run: agentsview usage daily --all --breakdown --json > costs.json
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: agent-costs
          path: costs.json
```

### Querying the SQLite Database Directly

```bash
# Open the database
sqlite3 ~/.agentsview/sessions.db

# Query recent sessions
SELECT 
  id, 
  agent, 
  project_name,
  started_at,
  message_count
FROM sessions
WHERE started_at > datetime('now', '-7 days')
ORDER BY started_at DESC
LIMIT 10;

# Get token stats by model
SELECT 
  model,
  SUM(total_tokens) as total,
  AVG(total_tokens) as avg_per_session
FROM session_usage
GROUP BY model
ORDER BY total DESC;
```

### Export Sessions Programmatically

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
)

type SessionUsage struct {
    SessionID          string   `json:"session_id"`
    Agent              string   `json:"agent"`
    TotalOutputTokens  int      `json:"total_output_tokens"`
    CostUSD            float64  `json:"cost_usd"`
    HasCost            bool     `json:"has_cost"`
}

func fetchSessionUsage(sessionID string) (*SessionUsage, error) {
    url := fmt.Sprintf("http://127.0.0.1:8080/api/v1/sessions/%s/usage", sessionID)
    resp, err := http.Get(url)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var usage SessionUsage
    if err := json.NewDecoder(resp.Body).Decode(&usage); err != nil {
        return nil, err
    }

    return &usage, nil
}

func main() {
    usage, err := fetchSessionUsage("your-session-id")
    if err != nil {
        panic(err)
    }

    fmt.Printf("Agent: %s\n", usage.Agent)
    fmt.Printf("Tokens: %d\n", usage.TotalOutputTokens)
    if usage.HasCost {
        fmt.Printf("Cost: $%.4f\n", usage.CostUSD)
    }
}
```

### Background Server with Systemd

```ini
# ~/.config/systemd/user/agentsview.service
[Unit]
Description=agentsview local agent analytics
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/agentsview serve --port 8080
Restart=on-failure
RestartSec=5s
Environment="AGENTSVIEW_DATA_DIR=%h/.agentsview"

[Install]
WantedBy=default.target
```

Enable and start:

```bash
systemctl --user enable agentsview.service
systemctl --user start agentsview.service
systemctl --user status agentsview.service
```

## Supported Agents & Session Paths

| Agent             | Default Session Directory                                      |
|-------------------|----------------------------------------------------------------|
| Claude Code       | `~/.claude/projects/`                                          |
| Codex             | `~/.codex/sessions/`                                           |
| Cursor            | `~/.cursor/sessions/`                                          |
| Copilot CLI       | `~/.copilot/`                                                  |
| Forge             | `~/.forge/`                                                    |
| Amp               | `~/.local/share/amp/threads/`                                  |
| Claude Cowork     | `~/Library/Application Support/Claude/local-agent-mode-sessions/` (macOS) |
| Antigravity       | `~/.gemini/antigravity/`                                       |
| Cortex Code       | `~/.snowflake/cortex/conversations/`                           |

All paths are auto-detected. Override with environment variables if needed.

## Troubleshooting

### Server Returns 403 Forbidden

**Symptom:** API requests return `403 Forbidden` when accessing through SSH tunnel, reverse proxy, or remote dev environment.

**Cause:** Host header validation fails when the browser's Host header doesn't match the server's expected origin.

**Solution:**

```bash
# Set public URL to match browser origin
agentsview serve --public-url http://127.0.0.1:18080

# Or trust multiple origins
agentsview serve --public-origin https://dev.example.com,https://codespaces.github.dev
```

### Sessions Not Appearing

**Symptom:** Agent sessions don't show up in the UI.

**Cause:** Session directory not mounted (Docker) or not auto-detected.

**Solution:**

```bash
# Verify session directory exists
ls ~/.claude/projects/

# Docker: ensure volume is mounted
docker run -v "$HOME/.claude/projects:/agents/claude:ro" \
  -e CLAUDE_PROJECTS_DIR=/agents/claude \
  ghcr.io/kenn-io/agentsview:latest

# Manual sync
agentsview sync
```

### Cost Data Missing

**Symptom:** `has_cost: false` or `cost_usd: 0`.

**Cause:** Model pricing not available in LiteLLM database or no token data recorded.

**Solution:**

```bash
# Check if token data exists
agentsview session usage <id> --format json | jq '.has_token_data'

# Update agentsview to latest version
curl -fsSL https://agentsview.io/install.sh | bash
```

### Background Server Not Starting

**Symptom:** `agentsview serve --background` fails or exits immediately.

**Solution:**

```bash
# Check logs
cat ~/.agentsview/serve.log

# Verify port not in use
lsof -i :8080

# Try foreground to see errors
agentsview serve
```

### Slow Git Outcome Metrics

**Symptom:** `agentsview stats --include-git-outcomes` hangs or takes too long.

**Cause:** Large repositories or missing git history.

**Solution:**

```bash
# Omit git metrics (default)
agentsview stats

# Or limit date range
agentsview stats --include-git-outcomes --since 2026-06-01
```

### Docker Permission Issues

**Symptom:** Root-owned files in `~/.agentsview` after Docker run.

**Solution:**

```bash
# Use named volume instead of bind mount
docker run -v agentsview-data:/data ghcr.io/kenn-io/agentsview:latest

# Or pre-create with correct ownership
mkdir -p ~/.agentsview
docker run -v "$HOME/.agentsview:/data" ghcr.io/kenn-io/agentsview:latest
```

## Advanced Usage

### PostgreSQL Backend

```bash
# Set PostgreSQL URL
export AGENTSVIEW_PG_URL="postgres://user:pass@host:5432/agentsview?sslmode=require"

# Start PostgreSQL mode
agentsview pg serve

# Docker
docker run -e PG_SERVE=1 \
  -e AGENTSVIEW_PG_URL="$AGENTSVIEW_PG_URL" \
  ghcr.io/kenn-io/agentsview:latest
```

### DuckDB Mirror

```bash
# Export SQLite to DuckDB
agentsview duckdb push --full

# Serve DuckDB read-only
agentsview duckdb serve

# Quack remote access
QUACK_TOKEN="$(openssl rand -base64 32)"
agentsview duckdb quack serve \
  --bind quack:0.0.0.0:9494 \
  --token "$QUACK_TOKEN" \
  --allow-insecure

# Connect to remote Quack
export AGENTSVIEW_DUCKDB_URL="quack:https://duckdb.example.com"
export AGENTSVIEW_DUCKDB_TOKEN="$QUACK_TOKEN"
agentsview duckdb serve
```

### Custom Timezone for Reports

```bash
# Use specific timezone for daily bucketing
agentsview usage daily --timezone America/New_York

# Or set environment variable
export TZ=Europe/London
agentsview usage daily
```

## Keyboard Shortcuts (Web UI)

- `j` / `k` — Navigate sessions
- `[` / `]` — Previous/next session
- `Cmd+K` / `Ctrl+K` — Open search
- `?` — Show all shortcuts
- `Esc` — Close dialogs/modals

## License

MIT
