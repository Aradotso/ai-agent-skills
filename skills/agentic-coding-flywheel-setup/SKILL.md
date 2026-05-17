---
name: agentic-coding-flywheel-setup
description: Bootstrap a fresh Ubuntu VPS into a complete multi-agent AI development environment with safety tools and coordination infrastructure in 30 minutes
triggers:
  - set up an agentic coding environment on my VPS
  - install the ACFS stack on Ubuntu
  - bootstrap my server for AI coding agents
  - configure a VPS with Claude Code and agentic tools
  - run the agentic coding flywheel setup
  - install the Dicklesworthstone agent stack
  - set up NTM and MCP Agent Mail
  - configure vibe mode for AI development
---

# Agentic Coding Flywheel Setup (ACFS)

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection

## Overview

**Agentic Coding Flywheel Setup (ACFS)** is a bootstrapping system that transforms a fresh Ubuntu VPS into a professional AI-powered development environment in ~30 minutes. It installs 30+ tools, configures three AI coding agents (Claude Code, Codex CLI, Gemini CLI), and sets up a complete agent coordination stack.

**Key Features:**
- **One-liner install**: `curl | bash` to full environment
- **Idempotent**: Resume from interruptions automatically
- **Vibe mode**: Pre-configured for maximum development velocity
- **Manifest-driven**: Single source of truth for all tools and configs
- **Battle-tested stack**: 10 Dicklesworthstone tools + 20+ developer utilities

**What Gets Installed:**
- Modern shell: zsh + oh-my-zsh + powerlevel10k
- Language runtimes: bun, uv/Python, Rust, Go
- AI agents: Claude Code, Codex CLI, Gemini CLI
- Agent coordination: NTM, MCP Agent Mail, SLB
- Developer tools: tmux, neovim, ripgrep, fzf, git, gh
- Cloud CLIs: Vault, Wrangler, Supabase, Vercel
- Security: checksummed upstream installers, verified downloads

## Installation

### Quick Install (Latest)

```bash
# Default mode (interactive)
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh?$(date +%s)" | bash

# Vibe mode (non-interactive, passwordless sudo, all dangerous flags enabled)
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh?$(date +%s)" | bash -s -- --yes --mode vibe
```

### Production Install (Pinned Version)

```bash
# Pin to a tagged release (recommended)
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/v0.7.0/install.sh" | bash -s -- --yes --mode vibe --ref v0.7.0

# Pin to a specific commit
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/abc1234/install.sh" | bash -s -- --yes --mode vibe --ref abc1234
```

### Installation Flags

```bash
--yes              # Non-interactive mode (auto-confirm all prompts)
--mode vibe        # Enable vibe mode (passwordless sudo, dangerous flags)
--ref <tag|sha>    # Pin to specific version (tag or commit SHA)
--skip-check       # Skip system requirements check
--debug            # Enable verbose debug output
```

### Requirements

- **OS**: Ubuntu 25.10 (installer auto-upgrades from 24.04/24.10)
- **User**: Non-root user with sudo access
- **Network**: Internet connection for package downloads
- **Disk**: ~10GB free space
- **Memory**: 2GB+ RAM recommended

## Post-Install Commands

After installation, ACFS provides several management commands:

### Health Check

```bash
# Run comprehensive system health check
acfs doctor

# Check specific category
acfs doctor --category shells
acfs doctor --category language_runtimes
acfs doctor --category agent_clis
```

### Update ACFS

```bash
# Update to latest version
acfs update

# Update to specific version
acfs update --ref v0.7.0
```

### Service Setup

```bash
# Configure agent coordination services (NTM, MCP Agent Mail, etc.)
acfs services-setup
```

### Onboarding

```bash
# Interactive tutorial for new users
onboard
```

## Key Tools & Usage

### AI Coding Agents

**Claude Code** (Anthropic)
```bash
# Set API key
export ANTHROPIC_API_KEY="sk-ant-..."

# Start interactive session
claude

# One-shot command
claude "refactor this function to use async/await" < input.js

# Edit files directly
claude --edit main.py "add error handling"
```

**Codex CLI** (OpenAI)
```bash
# Set API key
export OPENAI_API_KEY="sk-..."

# Interactive mode
codex

# Generate code
codex "write a REST API with Express"

# Code review
codex --review src/
```

**Gemini CLI** (Google)
```bash
# Set API key
export GEMINI_API_KEY="..."

# Start session
gemini

# Multi-modal (with images)
gemini "analyze this screenshot" --image screenshot.png
```

### Agent Coordination Stack

**NTM (Note-Taking Manager)**
```bash
# Initialize notes directory
ntm init

# Create note
ntm new "project-ideas"

# Search notes
ntm search "api design"

# List all notes
ntm list

# Edit note
ntm edit "project-ideas"
```

**MCP Agent Mail** (inter-agent communication)
```bash
# Start mail server
mcp_agent_mail serve --port 3000

# Send message between agents
mcp_agent_mail send --to claude@agents.local --subject "Review PR" --body "..."

# Check inbox
mcp_agent_mail inbox
```

**SLB (Session Ledger Bot)**
```bash
# Initialize session tracking
slb init

# Start new session
slb start "implementing auth feature"

# Log activity
slb log "completed OAuth flow"

# End session
slb end

# View session history
slb history
```

**BV (Beads Viewer)** - Session visualization
```bash
# View current session beads
bv show

# Export session graph
bv export --format svg --output session.svg

# Analyze session patterns
bv analyze
```

**UBS (Universal Backup System)**
```bash
# Configure backup
ubs init --dest s3://my-backups

# Create backup
ubs backup ~/projects

# List backups
ubs list

# Restore backup
ubs restore --backup 2025-01-15 --dest ~/restored
```

### Developer Tools

**Tmux** (terminal multiplexer)
```bash
# ACFS includes pre-configured tmux.conf

# Start new session
tmux new -s work

# Detach: Ctrl+b d
# List sessions
tmux ls

# Attach to session
tmux attach -t work

# Split panes
# Horizontal: Ctrl+b "
# Vertical: Ctrl+b %
```

**Neovim** (text editor)
```bash
# Launch neovim
nvim file.py

# ACFS includes sensible defaults
# Basic commands:
# :w  - save
# :q  - quit
# :wq - save and quit
# i   - insert mode
# ESC - normal mode
```

**Ripgrep** (fast search)
```bash
# Search current directory
rg "function.*async"

# Search with context
rg -C 3 "TODO"

# Search specific file types
rg --type py "class.*Model"

# Case-insensitive
rg -i "error"
```

**FZF** (fuzzy finder)
```bash
# Interactive file finder (Ctrl+T in shell)
# Interactive history search (Ctrl+R in shell)

# In scripts
selected_file=$(fzf)
echo "Selected: $selected_file"

# Filter command output
ps aux | fzf
```

## Configuration

### ACFS Config Files

All ACFS configuration lives in `~/.acfs/`:

```bash
~/.acfs/
├── zshrc              # Shell configuration
├── tmux.conf          # Tmux configuration
├── state.json         # Installation state
├── onboard/           # Tutorial lessons
└── scripts/           # Utility scripts
```

### Environment Variables

Key environment variables to configure:

```bash
# AI Agent API Keys
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
export GEMINI_API_KEY="..."

# Cloud Provider Keys
export CLOUDFLARE_API_TOKEN="..."
export SUPABASE_ACCESS_TOKEN="..."
export VERCEL_TOKEN="..."

# Vault (secrets management)
export VAULT_ADDR="https://vault.example.com"
export VAULT_TOKEN="..."

# ACFS behavior
export ACFS_MODE="vibe"           # Enable vibe mode features
export ACFS_AUTO_UPDATE="true"    # Auto-update ACFS
export ACFS_TELEMETRY="false"     # Disable telemetry
```

Add to `~/.zshrc`:
```bash
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc
source ~/.zshrc
```

### Vibe Mode

Vibe mode enables maximum development velocity:

```bash
# Enable during install
curl -fsSL "..." | bash -s -- --mode vibe

# Features enabled in vibe mode:
# - Passwordless sudo for your user
# - All dangerous agent flags enabled
# - Auto-confirm for tool installations
# - Optimized shell aliases
# - Pre-configured git settings
```

**Security Note**: Vibe mode is designed for personal development VPS instances, not production servers or shared environments.

## Common Patterns

### Pattern 1: Initial VPS Setup

```bash
# 1. SSH into fresh Ubuntu VPS
ssh user@vps-ip

# 2. Run ACFS installer
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh?$(date +%s)" | bash -s -- --yes --mode vibe

# 3. Set up API keys
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc
echo 'export OPENAI_API_KEY="sk-..."' >> ~/.zshrc
source ~/.zshrc

# 4. Verify installation
acfs doctor

# 5. Start coding with agents
claude "create a new Express API project"
```

### Pattern 2: Agent Session Workflow

```bash
# Start session tracking
slb start "building user authentication"

# Use AI agent for scaffolding
claude "create user model with email and password fields"

# Track progress
slb log "created user model"

# Review with another agent
codex --review models/user.py

# Visualize session
bv show

# End session
slb end
```

### Pattern 3: Multi-Agent Collaboration

```bash
# Agent 1: Generate implementation
claude "implement JWT authentication" > auth.py

# Agent 2: Review for security
codex --review auth.py > review.md

# Agent 3: Optimize performance
gemini "optimize this authentication flow" < auth.py > auth_optimized.py

# Track collaboration via MCP Agent Mail
mcp_agent_mail send --to review-agent --body "$(cat review.md)"
```

### Pattern 4: Backup Before Experimentation

```bash
# Backup current state
ubs backup ~/projects/myapp

# Let agent make risky changes
claude --edit src/ "refactor entire codebase to use TypeScript" --allow-dangerous

# If things go wrong
ubs restore --backup latest --dest ~/projects/myapp
```

### Pattern 5: Reproducible Team Environments

```bash
# Team lead: Create bootstrap script
cat > setup-team-env.sh << 'EOF'
#!/bin/bash
# Install ACFS with pinned version
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/v0.7.0/install.sh" \
  | bash -s -- --yes --mode vibe --ref v0.7.0

# Configure team-specific tools
git config --global user.name "$GIT_NAME"
git config --global user.email "$GIT_EMAIL"

# Clone team repos
gh repo clone myorg/backend ~/projects/backend
gh repo clone myorg/frontend ~/projects/frontend
EOF

# Team members: Run identical setup
export GIT_NAME="Alice" GIT_EMAIL="alice@company.com"
bash setup-team-env.sh
```

## Architecture

### Manifest-Driven Design

ACFS uses a single source of truth: `acfs.manifest.yaml`

```yaml
# Example manifest structure (simplified)
modules:
  - id: bun
    category: language_runtimes
    install:
      bash: |
        curl -fsSL https://bun.sh/install | bash
    verify:
      bash: |
        command -v bun >/dev/null 2>&1
    
  - id: claude_code
    category: agent_clis
    install:
      bash: |
        npm install -g @anthropic-ai/claude-code
    verify:
      bash: |
        command -v claude >/dev/null 2>&1
```

### Generated Scripts (Reference)

The manifest generates reference scripts in `scripts/generated/`:

```bash
scripts/generated/
├── install_shells.sh
├── install_language_runtimes.sh
├── install_agent_clis.sh
├── install_agent_coordination.sh
├── install_developer_tools.sh
├── install_cloud_clis.sh
├── doctor_checks.sh
└── install_all.sh
```

### Security

ACFS implements multiple security layers:

1. **Checksummed installers**: `checksums.yaml` verifies upstream scripts
2. **HTTPS-only downloads**: All sources fetched over TLS
3. **Isolated environments**: Each tool runs in its own context
4. **API key isolation**: Keys stored in environment, not config files
5. **Audit logging**: All installations logged to `~/.acfs/state.json`

## Troubleshooting

### Installation Fails Midway

```bash
# ACFS is idempotent - just re-run
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh?$(date +%s)" | bash -s -- --yes --mode vibe

# Check installation state
cat ~/.acfs/state.json | jq '.completed_phases'

# Run doctor to identify issues
acfs doctor
```

### Doctor Check Failures

```bash
# Run full diagnostic
acfs doctor --verbose

# Fix common issues:
# - Missing PATH entries
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# - Incomplete installations
acfs update

# - Permission issues
sudo chown -R $USER:$USER ~/.acfs
```

### Agent CLI Not Found

```bash
# Verify installation
acfs doctor --category agent_clis

# Reinstall specific agent
# (Manually, as ACFS doesn't have per-tool reinstall yet)
npm install -g @anthropic-ai/claude-code

# Check API key
echo $ANTHROPIC_API_KEY  # Should not be empty
```

### Slow Performance

```bash
# Check system resources
htop

# Verify zsh compilation
rm ~/.zcompdump
zsh -c 'autoload -U compinit && compinit'

# Disable plugins if needed
# Edit ~/.acfs/zshrc and comment out heavy plugins
```

### SSH Connection Issues After Install

```bash
# If you set up SSH key during install but can't connect:

# From local machine - use password one more time
ssh user@vps-ip

# On VPS - verify SSH key was added
cat ~/.ssh/authorized_keys

# If missing, add your public key
echo "ssh-ed25519 AAAA..." >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Try key-based auth again from local
ssh -i ~/.ssh/id_ed25519 user@vps-ip
```

### Services Won't Start

```bash
# Check service status
acfs services-setup --status

# View logs
journalctl -u ntm -f
journalctl -u mcp_agent_mail -f

# Restart services
sudo systemctl restart ntm
sudo systemctl restart mcp_agent_mail
```

## Advanced Usage

### Custom Tool Installation

Add custom tools to your environment post-install:

```bash
# Install additional language runtime
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install additional CLI tool
cargo install ripgrep

# Update PATH if needed
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Verify with doctor
acfs doctor
```

### Extending the Manifest

To add tools to ACFS itself, edit `acfs.manifest.yaml`:

```yaml
modules:
  - id: my_custom_tool
    category: developer_tools
    description: "My custom development tool"
    install:
      bash: |
        curl -fsSL https://example.com/install.sh | bash
    verify:
      bash: |
        command -v my_tool >/dev/null 2>&1
```

Then regenerate scripts:

```bash
cd packages/manifest
npm run generate
```

### Integration Testing

ACFS includes Docker-based integration tests:

```bash
# Run full install test in Docker
bash tests/vm/test_install_ubuntu.sh

# Test specific version
ACFS_REF=v0.7.0 bash tests/vm/test_install_ubuntu.sh

# Test vibe mode
ACFS_MODE=vibe bash tests/vm/test_install_ubuntu.sh
```

### Monitoring Agent Activity

```bash
# Track all agent sessions
slb history --all

# Export session data
slb export --format json > sessions.json

# Visualize patterns
bv analyze --input sessions.json --output report.html

# Monitor in real-time
tail -f ~/.acfs/logs/agent-activity.log
```

## Version Compatibility

- **v0.7.0**: Current stable (Ubuntu 25.10, all features)
- **v0.6.x**: Legacy (Ubuntu 24.10, limited stack tools)
- **v0.5.x**: Deprecated (Ubuntu 24.04, no vibe mode)

Always use the latest tagged release for production environments.

## Additional Resources

- **Website**: https://agent-flywheel.com (interactive wizard)
- **Repository**: https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup
- **Documentation**: See `docs/` directory in repo
- **Issues**: https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/issues
