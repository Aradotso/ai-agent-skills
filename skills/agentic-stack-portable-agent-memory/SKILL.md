---
name: agentic-stack-portable-agent-memory
description: Portable .agent/ folder with memory, skills, and protocols that works across Claude Code, Cursor, Windsurf, and other AI coding harnesses
triggers:
  - set up agentic stack for this project
  - install portable agent memory
  - configure agent harness with agentic stack
  - switch between cursor and claude code without losing memory
  - add agentic stack brain to my project
  - manage multiple AI coding agents in one project
  - create portable agent skills folder
  - set up cross-harness agent memory
---

# agentic-stack-portable-agent-memory

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What It Does

agentic-stack creates a portable `.agent/` folder that maintains memory, skills, and protocols across different AI coding harnesses (Claude Code, Cursor, Windsurf, OpenCode, OpenClaw, Copilot CLI, Gemini, Hermes, Pi, Codex, Antigravity). When you switch tools, your agent keeps its knowledge, preferences, and accumulated context.

It includes:
- **Memory layers**: personal, semantic, episodic, working memory
- **Skills system**: reusable capabilities with manifest tracking
- **Data layer**: monitor harness activity, token/cost estimates, KPI summaries
- **Data flywheel**: turn approved runs into training artifacts
- **Brain integration**: optional git-backed long-term memory
- **Harness manager**: TUI for managing multiple adapters simultaneously

## Installation

### macOS / Linux

```bash
# Tap and install (both lines required)
brew tap codejunkie99/agentic-stack https://github.com/codejunkie99/agentic-stack
brew install agentic-stack

# Install into your project
cd your-project
agentic-stack claude-code
```

### Windows (PowerShell)

```powershell
git clone https://github.com/codejunkie99/agentic-stack.git
cd agentic-stack
.\install.ps1 claude-code C:\path\to\your-project
```

### Clone Method

```bash
git clone https://github.com/codejunkie99/agentic-stack.git
cd agentic-stack
./install.sh claude-code
```

### Upgrade Existing Installation

```bash
brew update && brew upgrade agentic-stack
cd your-project
agentic-stack upgrade --dry-run
agentic-stack upgrade --yes
```

## Available Adapters

- `claude-code` - Anthropic Claude Code
- `cursor` - Cursor IDE
- `windsurf` - Windsurf
- `opencode` - OpenCode
- `openclaw` - OpenClaw
- `copilot-cli` - GitHub Copilot CLI
- `gemini` - Google Gemini CLI
- `hermes` - Hermes
- `pi` - Pi Coding Agent
- `codex` - Codex
- `standalone-python` - DIY Python loop
- `antigravity` - Antigravity

## Key Commands

### Project Management

```bash
# Interactive dashboard
agentic-stack dashboard

# Web-based mission control (beta)
agentic-stack mission-control

# Check adapter status
agentic-stack status

# Audit installation health
agentic-stack doctor

# Interactive adapter manager
agentic-stack manage

# Add additional adapter
agentic-stack add cursor

# Remove adapter
agentic-stack remove cursor
```

### Upgrade & Sync

```bash
# Preview infrastructure updates
agentic-stack upgrade --dry-run

# Apply updates (preserves memory and custom skills)
agentic-stack upgrade --yes

# Rebuild skill manifest
agentic-stack sync-manifest
```

### Brain Integration (Optional)

```bash
# Install Brain CLI
brew install codejunkie99/tap/brain

# Check Brain status
agentic-stack brain status

# Onboard project to Brain
agentic-stack brain onboard --agents codex,cursor --yes

# Query Brain memory
agentic-stack brain ask "auth decisions"

# Write to Brain memory
agentic-stack brain note "Use PKCE for local OAuth flows."

# Get MCP command
agentic-stack brain mcp-command
```

### Memory Transfer

```bash
# Export/import memory between projects
agentic-stack transfer
```

## Project Structure

After installation, your project will have:

```
.agent/
├── memory/
│   ├── personal/PREFERENCES.md    # User preferences (read first)
│   ├── semantic/                  # Conceptual knowledge
│   ├── episodic/                  # Session history
│   ├── working/                   # Active context
│   └── .features.json             # Feature toggles
├── skills/
│   ├── _manifest.jsonl            # Skill registry
│   ├── brain/                     # Brain integration skill
│   ├── data-layer/                # Harness monitoring
│   ├── data-flywheel/             # Training artifact export
│   ├── design-md/                 # DESIGN.md workflows
│   └── tldraw/                    # Visual canvas (opt-in)
├── tools/
│   └── brain_bridge.py            # Brain API wrapper
├── harness/
│   └── adapters/                  # Adapter-specific configs
└── install.json                   # Installed adapters manifest
```

## Configuration

### Initial Setup (Onboarding Wizard)

The wizard populates `.agent/memory/personal/PREFERENCES.md`:

```bash
# Accept all defaults (CI mode)
agentic-stack claude-code --yes

# Re-run wizard on existing project
agentic-stack claude-code --reconfigure
```

Wizard questions:
1. What should I call you? *(optional)*
2. Primary language(s)? *(default: unspecified)*
3. Explanation style? *(default: concise)*
4. Test strategy? *(default: test-after)*
5. Commit message style? *(default: conventional commits)*
6. Code review depth? *(default: critical issues only)*

Optional features (off by default):
- FTS memory search `[BETA]`
- tldraw visual canvas `[BETA]`

### Manual Preference Editing

Edit `.agent/memory/personal/PREFERENCES.md` directly:

```markdown
---
name: Alex
languages: [Python, TypeScript]
explanation_style: detailed
test_strategy: test-first
commit_style: conventional commits
review_depth: thorough
---

# Personal Preferences

I prefer async/await over callbacks. Use descriptive variable names.
Always include error handling for network requests.
```

### Feature Toggles

Edit `.agent/memory/.features.json`:

```json
{
  "fts_search": false,
  "tldraw": false
}
```

## Working with Memory Layers

### Personal Memory

```bash
# View preferences
cat .agent/memory/personal/PREFERENCES.md

# Add personal context
echo "## Code Style\n\nPrefer functional composition over inheritance." >> .agent/memory/personal/PREFERENCES.md
```

### Semantic Memory

```python
# .agent/memory/semantic/architecture_decisions.md
"""
Document lasting project knowledge
"""
```

### Episodic Memory

Automatically managed by harness; tracks session history and decisions.

### Working Memory

Active context during current session; cleared when appropriate.

## Creating Custom Skills

### Skill Structure

```
.agent/skills/my-custom-skill/
├── SKILL.md              # Main skill definition
├── README.md             # Documentation
└── tools/                # Optional tool scripts
    └── helper.py
```

### SKILL.md Format

```markdown
---
name: my-custom-skill
description: Does something specific for this project
version: 1.0.0
triggers:
  - use my custom skill
  - apply custom workflow
---

# My Custom Skill

## What It Does

Detailed explanation...

## When to Use

- Trigger condition 1
- Trigger condition 2

## Examples

\```python
# Working code example
\```
```

### Sync Manifest After Adding Skills

```bash
agentic-stack sync-manifest
```

## Multi-Adapter Setup

### Install Multiple Harnesses

```bash
# Interactive multi-select
cd your-project
./install.sh

# Or add individually
agentic-stack add claude-code
agentic-stack add cursor
agentic-stack add windsurf
```

### Check Installed Adapters

```bash
agentic-stack status
```

Output:
```
✓ claude-code (active)
✓ cursor (active)
✗ windsurf (not installed)
```

### Manage via TUI

```bash
agentic-stack manage
```

## Data Layer Usage

### Generate Dashboard

```python
# From within agent session, trigger:
# "show me the harness dashboard"
# "generate daily report"

# Or manually:
from pathlib import Path
import sys
sys.path.insert(0, str(Path.cwd() / ".agent" / "skills" / "data-layer"))
from dashboard import generate_dashboard

generate_dashboard()
```

### View Dashboard

```bash
open .agent/data/dashboard.html
cat .agent/data/daily-report.md
```

## Data Flywheel Usage

### Export Approved Runs

```python
from pathlib import Path
import sys
sys.path.insert(0, str(Path.cwd() / ".agent" / "skills" / "data-flywheel"))
from export import export_run

# Export specific run
export_run(
    run_id="2026-05-16-auth-refactor",
    redact_secrets=True,
    include_context=True
)
```

### Artifacts Generated

- `traces/` - Execution traces
- `context-cards/` - Contextual summaries
- `eval-cases/` - Test cases for evaluation
- `training.jsonl` - Training-ready format
- `metrics.json` - Flywheel readiness metrics

## Brain Integration

### Bridge from Agent Code

```python
#!/usr/bin/env python3
from pathlib import Path
import sys
sys.path.insert(0, str(Path.cwd() / ".agent" / "tools"))
from brain_bridge import query_brain, write_brain

# Query across projects
results = query_brain("authentication patterns")
for hit in results:
    print(f"{hit['project']}: {hit['content']}")

# Write durable note
write_brain(
    content="OAuth refresh tokens expire after 30 days",
    tags=["auth", "security"]
)
```

### MCP Integration

```bash
# Get MCP stdio command for harness config
agentic-stack brain mcp-command
```

Add to `.claude/mcp.json` or equivalent:

```json
{
  "mcpServers": {
    "brain": {
      "command": "brain",
      "args": ["serve"]
    }
  }
}
```

## Troubleshooting

### Doctor Reports Issues

```bash
agentic-stack doctor
```

Common issues:
- **Missing hooks**: Re-run `agentic-stack add <adapter>`
- **Orphaned files**: Use `agentic-stack remove <adapter>` then reinstall
- **Version drift**: Run `agentic-stack upgrade --yes`

### Manifest Out of Sync

```bash
# Rebuild from installed SKILL.md files
agentic-stack sync-manifest
```

### Python 3.9 Compatibility

Ensure Python 3.9+ is available:

```bash
python3 --version
```

If using brew Python:

```bash
brew install python@3.11
```

### Migration from Pre-v0.9

```bash
# Synthesize install.json from on-disk adapters
./install.sh doctor
```

### Brain Not Found

```bash
# Install Brain CLI first
brew install codejunkie99/tap/brain

# Verify installation
brain --version
```

### Adapter Conflicts

```bash
# Remove conflicting adapter
agentic-stack remove cursor

# Clean reinstall
agentic-stack add cursor
```

## Environment Variables

Common environment variables referenced:

```bash
# Optional: Custom Brain path
export BRAIN_PATH="$HOME/.config/brain"

# Optional: Override .agent directory
export AGENTIC_STACK_DIR=".agent"
```

## Common Patterns

### Single Harness Setup

```bash
cd my-project
agentic-stack claude-code
# Work exclusively in Claude Code
```

### Multi-Harness Setup

```bash
cd my-project
agentic-stack add claude-code
agentic-stack add cursor
agentic-stack add windsurf
# Switch between tools without losing context
```

### CI/Automation

```bash
# Scripted install (no interactive prompts)
agentic-stack claude-code --yes

# Verify installation
agentic-stack doctor || exit 1
```

### Project Migration

```bash
# Export from old project
cd old-project
agentic-stack transfer
# Follow export prompts, save curl command

# Import to new project
cd new-project
# Paste and run curl command
```

### Skill Development Workflow

```bash
# 1. Create skill directory
mkdir -p .agent/skills/my-skill

# 2. Write SKILL.md with frontmatter
cat > .agent/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: Custom skill for X
version: 1.0.0
triggers: ["use my skill"]
---
# My Skill
...
EOF

# 3. Sync manifest
agentic-stack sync-manifest

# 4. Verify
grep "my-skill" .agent/skills/_manifest.jsonl
```

### Safe Upgrades

```bash
# 1. Preview changes
agentic-stack upgrade --dry-run

# 2. Review output
# 3. Apply if safe
agentic-stack upgrade --yes

# 4. Verify
agentic-stack doctor
```

## Version History Highlights

- **v0.18.0**: External Brain memory integration
- **v0.17.0**: Adapters, Mission Control, lesson retraction
- **v0.16.0**: Safe project upgrades via `upgrade` command
- **v0.12.0**: tldraw visual canvas (opt-in beta)
- **v0.11.0**: Data layer + data flywheel
- **v0.10.0**: design-md skill, Python 3.9 fix
- **v0.9.0**: Manifest-driven harness manager
