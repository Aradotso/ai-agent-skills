---
name: oh-my-openagent-orchestration
description: Orchestrate multi-agent AI workflows with ultrawork, discipline agents, team mode, and hash-anchored editing for autonomous code development
triggers:
  - "set up oh-my-openagent for multi-agent orchestration"
  - "use ultrawork to build this feature"
  - "configure team mode with multiple AI agents"
  - "enable discipline agents for autonomous development"
  - "use hash-anchored editing to prevent stale line errors"
  - "run ralph loop until task is complete"
  - "orchestrate parallel AI agents with oh-my-openagent"
  - "configure IntentGate for better agent understanding"
---

# Oh My OpenAgent Orchestration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Oh My OpenAgent (omo) is an AI agent orchestration framework that coordinates multiple specialized AI agents (Sisyphus, Hephaestus, Oracle, Librarian, Explore) to work in parallel on complex development tasks. It provides hash-anchored editing to prevent stale-line errors, LSP integration for IDE-level precision, background agents for parallel work, and Team Mode for coordinating up to 8 agents simultaneously.

## Installation

### Quick Install (Recommended)

```bash
# Install via npm
npm install -g oh-my-opencode

# Or via npx (no install)
npx oh-my-opencode
```

### Configuration Setup

Create `opencode.json` or `opencode.jsonc` in your project root:

```jsonc
{
  "mcpServers": {
    "oh-my-openagent": {
      "command": "npx",
      "args": ["-y", "oh-my-opencode"]
    }
  }
}
```

For legacy compatibility, `oh-my-opencode` entries are still supported but will show a deprecation warning.

### Environment Variables

```bash
# Disable anonymous telemetry (optional)
export OMO_SEND_ANONYMOUS_TELEMETRY=0
# or
export OMO_DISABLE_POSTHOG=1

# Custom configuration path (optional)
export OMO_CONFIG_PATH=/path/to/custom/config.json
```

## Key Commands

### `ultrawork` / `ulw`

The primary command that activates all discipline agents and doesn't stop until the task is complete.

```bash
# In your AI agent (Claude Code, Cursor, etc.)
ultrawork: Implement user authentication with OAuth2
```

or shorthand:

```bash
ulw: Refactor the payment module to use Stripe
```

### Ralph Loop (`/ulw-loop`)

Self-referential loop that continues iterating until 100% done.

```bash
/ulw-loop: Fix all TypeScript errors in the codebase
```

### Team Mode Commands

Team Mode (v4.0+) allows a lead agent to coordinate up to 8 parallel member agents.

```typescript
// Enable team mode in your configuration
{
  "teamMode": {
    "enabled": true,
    "maxMembers": 8,
    "visualization": "tmux"
  }
}
```

Available team tools:
- `team_spawn` - Create a new team member agent
- `team_message` - Send message to a team member
- `team_status` - Check status of all team members
- `team_terminate` - Terminate a team member

Example team mode session:

```bash
# Use hyperplan (5 hostile critics)
hyperplan: Review this API design for security vulnerabilities

# Use security-research (3 hunters + 2 PoC engineers)
security-research: Analyze this authentication flow
```

## Discipline Agents

Oh My OpenAgent includes specialized agents that work in parallel:

- **Sisyphus** - Lead orchestrator, coordinates other agents
- **Hephaestus** - Code implementation and refactoring
- **Oracle** - Architecture and design decisions
- **Librarian** - Documentation and knowledge retrieval
- **Explore** - Codebase exploration and analysis

These agents activate automatically with `ultrawork`.

## Hash-Anchored Editing

Prevents stale-line errors by validating content with hash anchors:

```typescript
// Traditional edit (prone to stale-line errors)
edit_file({
  path: "src/utils.ts",
  old_content: "function oldName() {",
  new_content: "function newName() {"
});

// Hash-anchored edit (omo)
edit_file({
  path: "src/utils.ts",
  line_id: "L42#a8f3b2c1", // Content hash validates the line
  old_content: "function oldName() {",
  new_content: "function newName() {"
});
```

The `LINE#ID` format includes a content hash that ensures the line hasn't changed since it was read.

## LSP Integration

Access IDE-level features through the agent:

```typescript
// Workspace-wide rename
lsp_rename({
  file: "src/models/User.ts",
  position: { line: 10, character: 15 },
  newName: "UserAccount"
});

// Get diagnostics before building
const diagnostics = await lsp_diagnostics({
  file: "src/app.ts"
});

// AST-aware code search
ast_grep({
  pattern: "function $NAME($PARAMS) { $BODY }",
  language: "typescript"
});
```

## Background Agents

Fire multiple specialist agents in parallel while keeping context lean:

```typescript
// Example: Launch 5 parallel research agents
const tasks = [
  background_agent({ role: "security", task: "Audit auth flow" }),
  background_agent({ role: "performance", task: "Analyze bottlenecks" }),
  background_agent({ role: "testing", task: "Generate test cases" }),
  background_agent({ role: "docs", task: "Update API documentation" }),
  background_agent({ role: "refactor", task: "Clean up legacy code" })
];

// Results returned when ready, doesn't block main agent
```

## Built-in MCPs (Model Context Protocol)

Oh My OpenAgent includes three built-in MCPs:

### Exa (Web Search)

```bash
# Search the web for current information
exa: latest TypeScript 5.4 features
```

### Context7 (Official Documentation)

```bash
# Fetch official documentation
context7: React useEffect hook documentation
```

### Grep.app (GitHub Code Search)

```bash
# Search public GitHub repositories
grep.app: how to implement OAuth2 refresh token rotation
```

## Real-World Usage Examples

### Example 1: Full Feature Implementation

```bash
ultrawork: Build a REST API for a todo app with:
- User authentication (JWT)
- CRUD operations for todos
- PostgreSQL database
- Input validation
- Unit tests
- API documentation
```

The agent will:
1. Analyze requirements through IntentGate
2. Spawn discipline agents (Hephaestus for code, Oracle for architecture, Librarian for docs)
3. Use LSP for workspace-aware refactoring
4. Validate all edits with hash-anchored changes
5. Run until 100% complete

### Example 2: Security Audit

```bash
# Use team mode with security-research preset
security-research: Audit the authentication system for vulnerabilities

# This spawns:
# - 3 security hunter agents
# - 2 PoC engineering agents
# All working in parallel via tmux visualization
```

### Example 3: Large-Scale Refactoring

```bash
/ulw-loop: Migrate the entire codebase from JavaScript to TypeScript

# Ralph loop will:
# - Convert files iteratively
# - Fix type errors
# - Update imports
# - Run tests after each batch
# - Continue until 100% complete
```

### Example 4: Automated Code Review

```typescript
// Use hyperplan for hostile code review
hyperplan: Review this pull request for:
- Security vulnerabilities
- Performance issues
- Code quality
- Test coverage
- Documentation completeness

// This spawns 5 hostile critic agents who will:
// - Challenge every decision
// - Find edge cases
// - Suggest improvements
// - Validate assumptions
```

## Configuration

### Basic Configuration (`oh-my-opencode.json`)

```jsonc
{
  // Enable team mode
  "teamMode": {
    "enabled": true,
    "maxMembers": 8,
    "visualization": "tmux",
    "presets": ["hyperplan", "security-research"]
  },
  
  // Configure discipline agents
  "agents": {
    "sisyphus": { "enabled": true },
    "hephaestus": { "enabled": true },
    "oracle": { "enabled": true },
    "librarian": { "enabled": true },
    "explore": { "enabled": true }
  },
  
  // LSP settings
  "lsp": {
    "enabled": true,
    "languages": ["typescript", "python", "rust", "go"],
    "features": ["rename", "diagnostics", "hover", "definition"]
  },
  
  // Hash-anchored editing
  "hashAnchoring": {
    "enabled": true,
    "algorithm": "sha256"
  },
  
  // Built-in MCPs
  "mcps": {
    "exa": { "enabled": true },
    "context7": { "enabled": true },
    "grepApp": { "enabled": true }
  },
  
  // Background agents
  "backgroundAgents": {
    "maxConcurrent": 5,
    "timeout": 3600
  }
}
```

### Model Recommendations

For optimal performance with `ultrawork`:

```bash
# Recommended subscriptions (not affiliated)
# - ChatGPT ($20/month) - gpt-4, gpt-5.5
# - Kimi Code ($19/month) - kimi-k2.6
# - GLM Coding ($10/month) - glm-4-plus

# Or pay-per-token:
# - Kimi models (via API)
# - Gemini models (via Google AI Studio)
```

## Common Patterns

### Pattern 1: Iterative Development

```bash
# Start with broad requirements
ulw: Build a blog platform

# Let it work, then refine
ulw: Add markdown support and syntax highlighting

# Continue iterating
ulw: Implement comment system with moderation
```

### Pattern 2: Multi-Phase Projects

```bash
# Phase 1: Architecture
ultrawork: Design the database schema and API structure

# Phase 2: Implementation
ultrawork: Implement the backend API

# Phase 3: Frontend
ultrawork: Build the React frontend

# Phase 4: Testing & Deployment
ultrawork: Add comprehensive tests and deploy to production
```

### Pattern 3: Background Research

```bash
# Fire research agents while working
background_agent({ 
  task: "Research best practices for Redis caching strategies"
})

# Continue with main work
ulw: Implement user session management

# Research results available when ready
```

## Troubleshooting

### Issue: Stale Line Errors

**Problem**: Traditional agents fail with "line changed" errors.

**Solution**: Oh My OpenAgent's hash-anchored editing prevents this automatically. No action needed.

### Issue: Agents Getting Stuck

**Problem**: Agent stops making progress.

**Solution**: Use ralph loop for self-correcting behavior:

```bash
/ulw-loop: Complete the remaining tasks
```

### Issue: Team Members Not Coordinating

**Problem**: Team mode agents working in silos.

**Solution**: Check tmux visualization:

```bash
# Attach to team session
tmux attach-session -t omo-team

# Review agent communication
team_status
```

### Issue: LSP Features Not Working

**Problem**: Rename, diagnostics not available.

**Solution**: Ensure language server is installed:

```bash
# For TypeScript
npm install -g typescript typescript-language-server

# For Python
pip install python-lsp-server

# Check LSP status in config
{
  "lsp": {
    "enabled": true,
    "debug": true
  }
}
```

### Issue: High API Costs

**Problem**: Too many concurrent agents.

**Solution**: Adjust background agent limits:

```jsonc
{
  "backgroundAgents": {
    "maxConcurrent": 3,  // Reduce from default 5
    "timeout": 1800      // Shorter timeout
  }
}
```

### Issue: IntentGate Misunderstanding

**Problem**: Agent misinterprets user intent.

**Solution**: Be more explicit in your prompts:

```bash
# Vague
ulw: fix the bug

# Better
ultrawork: Fix the null pointer exception in src/auth/login.ts line 42 
when user email is undefined
```

## Advanced Usage

### Custom Agent Presets

Create custom team configurations:

```jsonc
{
  "teamMode": {
    "customPresets": {
      "fullstack-sprint": {
        "members": [
          { "role": "backend", "model": "claude-sonnet-4" },
          { "role": "frontend", "model": "gpt-4" },
          { "role": "database", "model": "kimi-k2.6" },
          { "role": "testing", "model": "claude-sonnet-4" },
          { "role": "docs", "model": "gpt-4" }
        ]
      }
    }
  }
}
```

Usage:

```bash
fullstack-sprint: Build a SaaS dashboard for analytics
```

### Programmatic API

```typescript
import { OhMyOpenAgent } from 'oh-my-opencode';

const omo = new OhMyOpenAgent({
  teamMode: { enabled: true, maxMembers: 8 },
  lsp: { enabled: true }
});

// Start ultrawork programmatically
await omo.ultrawork({
  task: "Implement payment processing",
  context: { 
    provider: "stripe",
    features: ["subscriptions", "refunds"]
  }
});

// Monitor progress
omo.on('progress', (status) => {
  console.log(`Progress: ${status.percentage}%`);
});

// Wait for completion
await omo.waitForCompletion();
```

## Integration with Popular AI Agents

### Claude Code

```bash
# In Claude Code, simply type:
ultrawork: your task here
```

### Cursor

```bash
# In Cursor's composer:
@oh-my-openagent ultrawork: your task here
```

### AmpCode

```bash
# In AmpCode:
/ulw: your task here
```

## Best Practices

1. **Start with `ultrawork`** - Let the orchestration handle complexity
2. **Use ralph loop for completion** - Ensures tasks finish to 100%
3. **Leverage team mode for large tasks** - Parallel agents = faster results
4. **Trust hash-anchored editing** - No need to manually verify line changes
5. **Use background agents for research** - Keep main agent focused
6. **Be specific with IntentGate** - Clear intent = better results
7. **Monitor team status in tmux** - Visual feedback prevents confusion
8. **Use presets for common workflows** - `hyperplan`, `security-research`, etc.

## Resources

- [Documentation](https://ohmyopenagent.com/)
- [Team Mode Guide](https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/refs/heads/dev/docs/guide/team-mode.md)
- [Discord Community](https://discord.gg/PUwSMR9XNk)
- [GitHub Repository](https://github.com/code-yeongyu/oh-my-openagent)
- [Installation Guide](https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/refs/heads/dev/docs/guide/installation.md)
