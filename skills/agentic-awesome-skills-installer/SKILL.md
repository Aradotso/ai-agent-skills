---
name: agentic-awesome-skills-installer
description: Install and manage 1,935+ AI agent skills for Claude Code, Cursor, Gemini CLI, Codex, Antigravity, and other coding assistants
triggers:
  - install agentic awesome skills
  - set up agent skills library
  - add skills to my AI coding assistant
  - install cursor skills from agentic awesome
  - configure claude code with skill library
  - browse available agent skills
  - install specialized plugin for web development
  - update my agent skills collection
---

# Agentic Awesome Skills Installer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

Agentic Awesome Skills is an installable GitHub library of 1,935+ reusable `SKILL.md` playbooks for AI coding assistants. It provides structured operating instructions, constraints, and outputs for recurring development tasks across planning, coding, debugging, testing, security, infrastructure, product work, and growth.

**Key capabilities:**
- Install full skill library or specialized domain plugins
- Support for Claude Code, Cursor, Codex CLI, Autohand Code, Gemini CLI, Antigravity, Kiro, OpenCode, GitHub Copilot
- 14 specialized plugins (Web App Builder, Security Engineer, Data Analytics, Agent & MCP Builder, etc.)
- Bundles and workflows for role-based recommendations
- Plugin-safe distributions for marketplace installs

**Repository:** https://github.com/sickn33/agentic-awesome-skills  
**License:** MIT  
**Current version:** v13.13.0

## Installation

### Prerequisites

- Node.js 14+ (for npx installer)
- Git (for clone-based installs)

### Quick Install (Default Path)

```bash
# Install to ~/.agents/skills (Antigravity 2.0 global default)
npx agentic-awesome-skills

# Verify installation
test -d ~/.agents/skills && echo "✓ Skills installed successfully"
ls ~/.agents/skills | head -5
```

### Tool-Specific Installation

```bash
# Claude Code (uses Claude plugin directory)
npx agentic-awesome-skills --claude

# Cursor (uses Cursor plugin directory)
npx agentic-awesome-skills --cursor

# Gemini CLI
npx agentic-awesome-skills --gemini

# Codex CLI
npx agentic-awesome-skills --codex

# Autohand Code
npx agentic-awesome-skills --path ~/.autohand/skills

# Antigravity IDE
npx agentic-awesome-skills --antigravity

# Antigravity CLI (agy) - uses ~/.gemini/antigravity-cli/skills/
npx agentic-awesome-skills --agy

# Kiro CLI
npx agentic-awesome-skills --kiro

# Kiro IDE
npx agentic-awesome-skills --path ~/.kiro/skills

# OpenCode (with filters)
npx agentic-awesome-skills --path .agents/skills --category development,backend --risk safe,none

# Custom path
npx agentic-awesome-skills --path ./my-custom-skills
```

### Install Specialized Plugin

```bash
# Install Web App Builder plugin (10 curated skills)
npx agentic-awesome-skills --plugin web-app-builder

# Install Security Engineer plugin
npx agentic-awesome-skills --plugin security-engineer

# Install Data Analytics plugin
npx agentic-awesome-skills --plugin data-analytics

# Install Agent & MCP Builder plugin
npx agentic-awesome-skills --plugin agent-mcp-builder

# List all available plugins
npx agentic-awesome-skills --list-plugins
```

### Manual Installation (Git Clone)

```bash
# Clone repository
git clone --depth 1 https://github.com/sickn33/agentic-awesome-skills.git

# Create symlink to default path
ln -s "$(pwd)/agentic-awesome-skills/skills" ~/.agents/skills

# Or copy skills directory
cp -r agentic-awesome-skills/skills ~/.agents/skills
```

## Using Installed Skills

### Claude Code

```bash
# In Claude Code chat
>> /brainstorming help me plan a feature for user authentication

# Use specialized skill
>> /api-design design a REST API for user management

# Chain skills
>> /planning create a project plan, then /testing add test coverage strategy
```

### Cursor

```python
# In Cursor chat (use @ prefix)
@brainstorming help me plan a feature for user authentication

# Reference skill explicitly
@api-security-audit review this endpoint for vulnerabilities

# Multi-skill workflow
@planning create architecture, then @code-review analyze implementation
```

### Gemini CLI

```bash
# Invoke skill in natural language
gemini "Use brainstorming to plan a feature for user authentication"

# Specialized skill
gemini "Use api-design to create OpenAPI spec for user endpoints"

# Workflow execution
gemini "Use planning to draft architecture, then code-review to analyze it"
```

### Antigravity IDE

```python
# Use @ prefix in chat
@brainstorming help me plan a microservices architecture

# Chain skills
@api-design create REST API spec, then @testing add integration tests

# Specialized plugin skill
@secure-coding-review audit this authentication module
```

### Antigravity CLI (agy)

```bash
# Use slash commands
agy /brainstorming help me plan a feature

# Specialized skill
agy /api-security-audit check this endpoint

# View skill help
agy /brainstorming --help
```

## Available Specialized Plugins

| Plugin Name | Skill Count | Use Case | Install Command |
|------------|-------------|----------|-----------------|
| **web-app-builder** | 10 | Frontend/fullstack web development | `npx agentic-awesome-skills --plugin web-app-builder` |
| **product-design-studio** | 10 | Product UI, brand, accessibility | `npx agentic-awesome-skills --plugin product-design-studio` |
| **security-engineer** | 10 | Security testing, audit, hardening | `npx agentic-awesome-skills --plugin security-engineer` |
| **secure-app-builder** | 10 | Security-embedded development | `npx agentic-awesome-skills --plugin secure-app-builder` |
| **documents-presentations** | 9 | Office files, decks, conversions | `npx agentic-awesome-skills --plugin documents-presentations` |
| **data-analytics** | 10 | SQL, dashboards, experiments | `npx agentic-awesome-skills --plugin data-analytics` |
| **agent-mcp-builder** | 10 | Agentic apps, MCP tools, RAG | `npx agentic-awesome-skills --plugin agent-mcp-builder` |
| **oss-maintainer** | 10 | PRs, releases, contributor work | `npx agentic-awesome-skills --plugin oss-maintainer` |
| **qa-test-automation** | 10 | Test suites, browser automation | `npx agentic-awesome-skills --plugin qa-test-automation` |
| **devops-cloud** | 10 | Infrastructure, deployments | `npx agentic-awesome-skills --plugin devops-cloud` |
| **accessibility-ux** | 8 | WCAG audits, screen readers | `npx agentic-awesome-skills --plugin accessibility-ux` |
| **api-platform-builder** | 10 | API design, auth, observability | `npx agentic-awesome-skills --plugin api-platform-builder` |
| **saas-launch-revenue** | 10 | SaaS MVPs, pricing, analytics | `npx agentic-awesome-skills --plugin saas-launch-revenue` |
| **ai-product-eval-ops** | 10 | AI metrics, evals, experiments | `npx agentic-awesome-skills --plugin ai-product-eval-ops` |

## Configuration

### Environment Variables

```bash
# Override default installation path
export AGENTIC_SKILLS_PATH=~/.my-custom-skills

# Use specific branch/tag
export AGENTIC_SKILLS_TAG=main

# Skip verification prompts
export AGENTIC_SKILLS_AUTO_CONFIRM=true
```

### Configuration File (.agenticrc)

Create `~/.agenticrc` for persistent settings:

```json
{
  "defaultPath": "~/.agents/skills",
  "autoUpdate": true,
  "updateInterval": "weekly",
  "preferredPlugins": [
    "web-app-builder",
    "security-engineer",
    "data-analytics"
  ],
  "excludeCategories": ["experimental"],
  "riskLevel": "safe"
}
```

### Tool-Specific Configuration

#### Claude Code Plugin

```json
// ~/.claude/plugins/agentic-awesome-skills/config.json
{
  "enabled": true,
  "autoLoad": true,
  "categories": ["development", "security", "testing"],
  "maxSkills": 1935
}
```

#### Cursor Settings

```json
// .cursor/settings.json
{
  "cursor.skills.path": "~/.agents/skills",
  "cursor.skills.autoLoad": true,
  "cursor.skills.categories": ["all"]
}
```

## Real-World Examples

### Example 1: Install Full Library and Use Planning Skill

```bash
# Install to default path
npx agentic-awesome-skills

# In Claude Code
>> /planning I need to architect a new payment processing system with Stripe
```

**Expected output:** Detailed project plan with phases, dependencies, milestones, security considerations, and testing strategy.

### Example 2: Install Security Plugin and Audit Code

```bash
# Install security-focused plugin
npx agentic-awesome-skills --plugin security-engineer

# In Cursor, open a file with authentication logic
@security-audit review this JWT implementation for vulnerabilities
```

**Expected output:** Security assessment covering token validation, expiration, secret management, XSS/CSRF risks, and remediation steps.

### Example 3: Install Web Plugin for Full-Stack Development

```python
# Install web development plugin
import subprocess
subprocess.run(["npx", "agentic-awesome-skills", "--plugin", "web-app-builder"])

# In Antigravity IDE
@react-component-builder create a user profile card with avatar, name, bio, and edit button
```

**Expected output:** React component with TypeScript, proper props interface, accessibility attributes, and CSS modules.

### Example 4: Update Existing Installation

```bash
# Update to latest version
npx agentic-awesome-skills --update

# Or reinstall from scratch
rm -rf ~/.agents/skills
npx agentic-awesome-skills
```

### Example 5: Filter Skills by Category and Risk

```bash
# Install only safe development skills
npx agentic-awesome-skills \
  --category development,backend,testing \
  --risk safe,none \
  --path ./filtered-skills

# Verify filtered installation
ls ./filtered-skills | wc -l
```

### Example 6: Programmatic Installation (Python)

```python
import subprocess
import os
from pathlib import Path

def install_agentic_skills(path=None, plugin=None):
    """Install agentic awesome skills library."""
    cmd = ["npx", "agentic-awesome-skills"]
    
    if path:
        cmd.extend(["--path", str(Path(path).expanduser())])
    if plugin:
        cmd.extend(["--plugin", plugin])
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    
    if result.returncode == 0:
        print(f"✓ Skills installed successfully")
        return True
    else:
        print(f"✗ Installation failed: {result.stderr}")
        return False

# Install full library
install_agentic_skills(path="~/.agents/skills")

# Install specialized plugin
install_agentic_skills(
    path="~/.agents/skills",
    plugin="data-analytics"
)
```

### Example 7: Verify Installation and List Skills

```bash
# Verify installation
test -d ~/.agents/skills && echo "✓ Installed" || echo "✗ Not installed"

# Count installed skills
find ~/.agents/skills -name "SKILL.md" | wc -l

# List all skill names
find ~/.agents/skills -name "SKILL.md" -exec basename {} .md \; | sort

# Search for specific skills
find ~/.agents/skills -name "*security*.md"
find ~/.agents/skills -name "*test*.md"
find ~/.agents/skills -name "*api*.md"
```

## Common Patterns

### Pattern 1: Domain-Specific Installation

```bash
# Web development team
npx agentic-awesome-skills --plugin web-app-builder
npx agentic-awesome-skills --plugin qa-test-automation

# Security team
npx agentic-awesome-skills --plugin security-engineer
npx agentic-awesome-skills --plugin secure-app-builder

# Data team
npx agentic-awesome-skills --plugin data-analytics
npx agentic-awesome-skills --plugin api-platform-builder

# Platform/SRE team
npx agentic-awesome-skills --plugin devops-cloud
npx agentic-awesome-skills --plugin oss-maintainer
```

### Pattern 2: Skill Chaining

```bash
# In Claude Code or Cursor
>> /planning create architecture for user authentication
>> /api-design design REST endpoints for the planned architecture
>> /secure-coding-review audit the authentication flow
>> /testing create test plan for auth endpoints
>> /code-review final review before merge
```

### Pattern 3: Project-Local Installation

```bash
# Install skills in project directory
cd my-project
npx agentic-awesome-skills --path .agents/skills

# Add to .gitignore
echo ".agents/skills/" >> .gitignore

# Document in README
echo "Run 'npx agentic-awesome-skills --path .agents/skills' to install agent skills" >> README.md
```

### Pattern 4: Conditional Installation in CI/CD

```yaml
# .github/workflows/ci.yml
name: Install Agent Skills
on: [push]

jobs:
  install-skills:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install agent skills
        run: |
          npx agentic-awesome-skills \
            --path .agents/skills \
            --category development,testing \
            --risk safe
      - name: Verify installation
        run: |
          test -d .agents/skills
          find .agents/skills -name "SKILL.md" | wc -l
```

## Troubleshooting

### Issue: Installation Fails with Permission Error

```bash
# Error: EACCES: permission denied
# Solution: Use user-writable path or fix permissions
npx agentic-awesome-skills --path ~/my-skills

# Or fix permissions on default path
mkdir -p ~/.agents/skills
chmod -R u+w ~/.agents/skills
```

### Issue: Skills Not Loading in Claude Code

```bash
# 1. Verify installation path matches Claude's expectation
npx agentic-awesome-skills --claude

# 2. Restart Claude Code

# 3. Check plugin is enabled
# In Claude: Settings > Plugins > Agentic Awesome Skills > Enable

# 4. Verify skill files exist
ls ~/.claude/plugins/agentic-awesome-skills/
```

### Issue: Skills Not Found in Cursor

```bash
# 1. Check Cursor settings
# .cursor/settings.json should have correct path

# 2. Reinstall to Cursor's expected location
npx agentic-awesome-skills --cursor

# 3. Reload Cursor window
# Cmd/Ctrl+Shift+P > "Reload Window"

# 4. Test skill invocation
# @brainstorming test
```

### Issue: npx Command Not Found

```bash
# Install Node.js if missing
# macOS
brew install node

# Ubuntu/Debian
sudo apt-get update
sudo apt-get install nodejs npm

# Verify installation
node --version
npm --version
npx --version
```

### Issue: Installation Hangs or Times Out

```bash
# Use shallow clone (default)
npx agentic-awesome-skills --shallow

# Or specify release tag instead of main branch
npx agentic-awesome-skills --tag v13.13.0

# Or download release directly
curl -L https://github.com/sickn33/agentic-awesome-skills/archive/refs/tags/v13.13.0.tar.gz | tar xz
mv agentic-awesome-skills-13.13.0/skills ~/.agents/skills
```

### Issue: Cannot Find Specific Skill

```bash
# Search for skill by name
find ~/.agents/skills -iname "*auth*.md"
find ~/.agents/skills -iname "*security*.md"

# List all available skills
find ~/.agents/skills -name "SKILL.md" -exec basename {} .md \;

# Check if skill is in a specialized plugin
npx agentic-awesome-skills --list-plugins
npx agentic-awesome-skills --plugin security-engineer --dry-run
```

### Issue: Update Not Working

```bash
# Force reinstall
rm -rf ~/.agents/skills
npx agentic-awesome-skills

# Or use update flag
npx agentic-awesome-skills --update --force

# Clear npm cache if needed
npx clear-npx-cache
npm cache clean --force
```

## Advanced Usage

### Custom Skill Repository

```bash
# Fork the repository and install from your fork
npx agentic-awesome-skills \
  --repo https://github.com/YOUR_USERNAME/agentic-awesome-skills \
  --path ~/.agents/skills

# Or clone and install locally
git clone https://github.com/YOUR_USERNAME/agentic-awesome-skills
cd agentic-awesome-skills
npm install
npm link
agentic-awesome-skills --path ~/.agents/skills
```

### Skill Manifest Export

```bash
# Export skill manifest for tooling integration
npx agentic-awesome-skills --export-manifest skills-manifest.json

# Use manifest in custom tools
cat skills-manifest.json | jq '.skills[] | select(.category=="security")'
```

### Programmatic API (Node.js)

```javascript
const { installSkills, listSkills, searchSkills } = require('agentic-awesome-skills');

// Install skills programmatically
await installSkills({
  path: '~/.agents/skills',
  categories: ['development', 'security'],
  riskLevel: 'safe'
});

// List installed skills
const skills = await listSkills('~/.agents/skills');
console.log(`Installed ${skills.length} skills`);

// Search skills
const securitySkills = await searchSkills({
  path: '~/.agents/skills',
  query: 'security audit'
});
console.log('Security skills:', securitySkills);
```

## Best Practices

1. **Start small:** Install specialized plugins before full library
2. **Use tool-specific flags:** `--claude`, `--cursor`, etc. for optimal paths
3. **Update regularly:** Run `npx agentic-awesome-skills --update` monthly
4. **Project-local installs:** Use `.agents/skills` for team consistency
5. **Filter by risk:** Use `--risk safe` for production environments
6. **Document for team:** Add install command to project README
7. **Skill chaining:** Chain multiple skills for complex workflows
8. **Test first:** Verify installation with simple skill before complex ones

## Additional Resources

- **Homepage:** https://sickn33.github.io/agentic-awesome-skills/
- **Repository:** https://github.com/sickn33/agentic-awesome-skills
- **Plugin Comparison:** https://sickn33.github.io/agentic-awesome-skills/plugins
- **Specialized Plugin Roadmap:** [docs/users/specialized-plugin-roadmap.md](https://github.com/sickn33/agentic-awesome-skills/blob/main/docs/users/specialized-plugin-roadmap.md)
- **Plugin Guide:** [docs/users/plugins.md](https://github.com/sickn33/agentic-awesome-skills/blob/main/docs/users/plugins.md)
- **Community:** Twitter [@AASkills_](https://x.com/AASkills_)
- **License:** MIT
