---
name: awesome-openclaw-agents
description: Production-ready AI agent templates for OpenClaw - 205+ SOUL.md configs across 24 categories for autonomous agents
triggers:
  - "show me openclaw agent templates"
  - "how do I create a SOUL.md agent config"
  - "set up an openclaw productivity agent"
  - "find openclaw agents for development tasks"
  - "deploy an openclaw marketing agent"
  - "configure a crewclaw multi-agent system"
  - "browse openclaw agent examples"
  - "use an openclaw agent template"
---

# awesome-openclaw-agents

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill enables you to work with the **awesome-openclaw-agents** repository: a curated collection of 205+ production-ready AI agent templates for the OpenClaw ecosystem. Each template is a copy-paste ready `SOUL.md` configuration file that defines an autonomous agent's personality, goals, tools, and behavior.

## What It Is

- **205+ agent templates** across 24 categories (Productivity, Development, Marketing, Business, DevOps, Finance, etc.)
- **SOUL.md format**: declarative agent configuration files that work with OpenClaw gateway
- **Production-ready**: each template includes role, goals, backstory, tools, and guardrails
- **Community-driven**: MIT licensed, accepts PRs for new agent templates
- **CrewClaw integration**: optional no-code deploy platform for full agent packages (Dockerfile, docker-compose, bot, README)

## Installation

```bash
# Clone the repository
git clone https://github.com/mergisi/awesome-openclaw-agents.git
cd awesome-openclaw-agents

# Install quickstart dependencies
cd quickstart
npm install

# Copy a template SOUL.md
cp ../agents/productivity/orion/SOUL.md ./SOUL.md

# Start the agent
node bot.js
```

## Repository Structure

```
awesome-openclaw-agents/
├── agents/                    # 205+ agent templates organized by category
│   ├── productivity/          # Task management, email, habits
│   ├── development/           # Code review, docs, bug hunting
│   ├── marketing/             # Content, SEO, social media
│   ├── business/              # Sales, finance, HR
│   ├── devops/                # CI/CD, monitoring, infrastructure
│   └── ...                    # 19 more categories
├── quickstart/                # Minimal Node.js bot runner
├── skills/                    # On-device skills for Gemma & Claude Code
├── use-cases/                 # 132 real-world examples
├── agents.json                # Machine-readable catalog
└── TROUBLESHOOTING.md         # Known issues & recovery
```

## SOUL.md Format

Every agent template follows this structure:

```markdown
# Agent Name

**Role**: {one-line role description}

**Goals**:
- Primary objective
- Secondary objectives
- Success metrics

**Backstory**: {agent's personality and expertise}

**Tools**: {comma-separated list of capabilities}
- tool_name: description

**Guardrails**:
- What the agent should NOT do
- Security boundaries
- Escalation triggers

**Memory**:
- Conversation history depth
- Key facts to remember
- User preferences

**Triggers**:
- Event patterns that activate this agent
- Scheduling (if applicable)

**Output Format**:
- Preferred response structure
- Notification channels
```

## Using Agent Templates

### 1. Browse and Select

```bash
# List all productivity agents
ls agents/productivity/

# View a specific template
cat agents/productivity/orion/SOUL.md

# Search by topic (e.g., "code review")
grep -r "code review" agents/development/
```

### 2. Copy Template

```bash
# Copy to your project
cp agents/development/code-reviewer/SOUL.md ./my-agents/SOUL.md

# Or use multiple agents
mkdir -p ./my-agents
cp agents/productivity/orion/SOUL.md ./my-agents/orion.md
cp agents/development/bug-hunter/SOUL.md ./my-agents/debug.md
```

### 3. Customize Configuration

```markdown
# Edit SOUL.md to match your needs

**Role**: Senior Code Reviewer specializing in TypeScript and React

**Goals**:
- Review PRs within 2 hours
- Flag security vulnerabilities
- Suggest performance improvements
- Maintain 95% approval rating

**Tools**: github, eslint, typescript-analyzer, security-scanner

**Guardrails**:
- Never auto-merge without human approval on main branch
- Escalate if >500 lines changed
- Require tests for new features
```

### 4. Run with OpenClaw

```javascript
// quickstart/bot.js example
const { OpenClaw } = require('openclaw-sdk');

const agent = new OpenClaw({
  soulPath: './SOUL.md',
  provider: 'anthropic', // or 'openai', 'ollama', etc.
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-5-sonnet-20241022'
});

// Start listening
agent.on('message', async (msg) => {
  const response = await agent.process(msg);
  console.log(response);
});

agent.start();
```

## Common Agent Patterns

### Productivity Agent (Daily Standup)

```markdown
# Standup

**Role**: Daily standup coordinator

**Goals**:
- Collect standup updates from team by 10am
- Generate summary for async teams
- Track blockers and escalate

**Tools**: slack, jira, calendar

**Triggers**:
- Schedule: "weekdays at 9:00 AM"
- Manual: "/standup start"

**Output Format**:
```
## 🌅 Daily Standup - {date}

**Done Yesterday**:
- @alice: Completed PR #123
- @bob: Fixed bug in payment flow

**Planned Today**:
- @alice: Start feature X
- @bob: Code review sprint

**Blockers**: 
- @charlie: Waiting on design assets
```
```

### Development Agent (Code Reviewer)

```markdown
# Lens (Code Reviewer)

**Role**: Senior code reviewer with security focus

**Goals**:
- Review all PRs within 30 minutes
- Catch security vulnerabilities before merge
- Maintain code quality standards
- Provide constructive feedback

**Tools**: github, eslint, sonarqube, snyk

**Guardrails**:
- Never approve PRs that introduce CVEs
- Escalate if test coverage drops below 80%
- Require two human approvals for auth changes

**Triggers**:
- GitHub webhook: pull_request.opened
- GitHub webhook: pull_request.synchronize

**Output Format**:
✅ **APPROVED** | ⚠️ **CHANGES REQUESTED** | ❌ **BLOCKED**

**Summary**: {one-line verdict}

**Security**: {findings}
**Performance**: {findings}
**Style**: {suggestions}
```

### Marketing Agent (Content Writer)

```markdown
# Echo (Content Writer)

**Role**: SEO-focused content writer

**Goals**:
- Generate blog posts from topics
- Optimize for target keywords
- Include internal links
- Match brand voice

**Tools**: openai, serp-api, wordpress, grammarly

**Backstory**: I'm a content strategist who's written 500+ articles...

**Output Format**:
# {SEO Title}

{Hook paragraph}

## {H2 Section}
{Content with keyword density 1-2%}

**Meta Description**: {155 chars}
**Keywords**: {comma-separated}
**Internal Links**: {3-5 suggestions}
```

## Multi-Agent Crews

```javascript
// Use multiple agents together
const crew = new OpenClaw.Crew({
  agents: [
    { soul: './agents/orion.md', role: 'manager' },
    { soul: './agents/lens.md', role: 'reviewer' },
    { soul: './agents/scribe.md', role: 'writer' }
  ],
  process: 'sequential' // or 'hierarchical', 'consensus'
});

// Manager delegates to reviewers and writers
const result = await crew.execute({
  task: 'Review PR #123 and update documentation'
});
```

## Environment Configuration

```bash
# .env file
OPENCLAW_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...

# Optional: CrewClaw for no-code deploys
CREWCLAW_API_KEY=cc_...

# Tool integrations
GITHUB_TOKEN=ghp_...
SLACK_BOT_TOKEN=xoxb-...
JIRA_API_TOKEN=...

# Model routing (optional)
OLLAMA_HOST=http://localhost:11434
OPENAI_API_KEY=sk-...
```

## CLI Quick Reference

```bash
# Register an agent
openclaw agents add ./SOUL.md

# List registered agents
openclaw agents list

# Start gateway
openclaw gateway start

# Run a one-off task
openclaw run --agent=orion --task="Prioritize today's tasks"

# Deploy to CrewClaw (optional)
crewclaw deploy --agent=./SOUL.md --platform=docker
```

## Working with Templates

### Customize an Existing Template

```bash
# Start with a template
cp agents/productivity/orion/SOUL.md ./my-pm-agent.md

# Edit key sections
vim my-pm-agent.md

# Test locally
node quickstart/bot.js --soul=./my-pm-agent.md
```

### Combine Multiple Agents

```javascript
// Sequential workflow: bug hunter → code reviewer → PR merger
const pipeline = [
  { agent: 'bug-hunter', input: 'github_issue' },
  { agent: 'code-reviewer', input: 'pull_request' },
  { agent: 'pr-merger', condition: 'approved' }
];

await crew.executePipeline(pipeline, { issueId: 456 });
```

### Create a New Template

```markdown
# My Custom Agent

**Role**: {specific role}

**Goals**:
- Goal 1 with measurable outcome
- Goal 2
- Goal 3

**Backstory**: {1-2 paragraphs establishing expertise}

**Tools**: {comma-separated, be specific}
- tool_name: what it does
- another_tool: its purpose

**Guardrails**:
- Security boundary
- Escalation rule
- Error handling

**Memory**:
- Retain: user preferences, past decisions
- Forget: sensitive data after task complete

**Triggers**:
- Event: when to activate
- Schedule: (if recurring)

**Output Format**:
{Show exact structure with examples}
```

## Integration Examples

### GitHub PR Review Bot

```javascript
const express = require('express');
const app = express();

app.post('/webhook/github', async (req, res) => {
  const { action, pull_request } = req.body;
  
  if (action === 'opened' || action === 'synchronize') {
    const reviewer = new OpenClaw({ soulPath: './agents/development/code-reviewer/SOUL.md' });
    
    const review = await reviewer.process({
      prNumber: pull_request.number,
      files: pull_request.changed_files,
      diff: await fetchDiff(pull_request.diff_url)
    });
    
    await postGitHubComment(pull_request.number, review);
  }
  
  res.status(200).send('OK');
});
```

### Slack Daily Standup

```javascript
const { WebClient } = require('@slack/web-api');
const slack = new WebClient(process.env.SLACK_BOT_TOKEN);

const standup = new OpenClaw({ soulPath: './agents/productivity/daily-standup/SOUL.md' });

// Every weekday at 9 AM
cron.schedule('0 9 * * 1-5', async () => {
  const channel = 'C01234567'; // #engineering
  
  await slack.chat.postMessage({
    channel,
    text: '🌅 Time for standup! Reply with: Done | Doing | Blockers'
  });
  
  // Collect responses for 1 hour
  setTimeout(async () => {
    const messages = await fetchStandupReplies(channel);
    const summary = await standup.process({ messages });
    
    await slack.chat.postMessage({
      channel,
      text: summary,
      thread_ts: originalMessage.ts
    });
  }, 60 * 60 * 1000);
});
```

### Email Inbox Zero

```javascript
const { gmail } = require('googleapis').google.gmail('v1');

const inbox = new OpenClaw({ soulPath: './agents/productivity/inbox-zero/SOUL.md' });

async function processInbox() {
  const messages = await gmail.users.messages.list({
    userId: 'me',
    q: 'is:unread',
    maxResults: 50
  });
  
  for (const msg of messages.data.messages) {
    const email = await gmail.users.messages.get({ userId: 'me', id: msg.id });
    
    const triage = await inbox.process({
      subject: email.payload.headers.find(h => h.name === 'Subject').value,
      from: email.payload.headers.find(h => h.name === 'From').value,
      body: email.snippet
    });
    
    // Apply label based on triage
    if (triage.action === 'archive') {
      await gmail.users.messages.modify({
        userId: 'me',
        id: msg.id,
        resource: { removeLabelIds: ['UNREAD'] }
      });
    }
  }
}
```

## Common Troubleshooting

### Agent Not Responding

```bash
# Check gateway status
openclaw gateway status

# Verify SOUL.md syntax
openclaw validate ./SOUL.md

# Check logs
tail -f ~/.openclaw/logs/gateway.log
```

### Rate Limiting

```javascript
// Add rate limiting to agent
const agent = new OpenClaw({
  soulPath: './SOUL.md',
  rateLimit: {
    maxRequests: 100,
    perMinutes: 1
  }
});
```

### Tool Integration Failures

```markdown
# In SOUL.md, add error handling

**Guardrails**:
- If GitHub API fails, retry 3x with exponential backoff
- If Slack is down, queue messages for later
- If critical tool unavailable, escalate to human
```

## Categories Overview

| Category | Agent Count | Example Use Cases |
|----------|-------------|-------------------|
| Productivity | 7 | Task management, email triage, habits |
| Development | 16 | Code review, docs, testing, migrations |
| Marketing | 15 | Content, SEO, social media, brand monitoring |
| Business | 12 | Sales, proposals, CRM, competitive intelligence |
| DevOps | 8 | CI/CD, monitoring, incident response |
| Finance | 6 | Invoicing, expenses, forecasting |
| Customer Success | 10 | Support tickets, onboarding, retention |
| Security | 5 | Vulnerability scanning, compliance audits |

## Best Practices

1. **Start with templates**: Don't write from scratch - customize existing agents
2. **Clear goals**: Define measurable success criteria in SOUL.md
3. **Explicit guardrails**: Document what agent should NOT do
4. **Test locally**: Use quickstart bot before production deploy
5. **Version control**: Track SOUL.md changes like code
6. **Monitor behavior**: Log all agent actions for audit trail
7. **Escalation paths**: Define when humans should intervene
8. **Memory hygiene**: Clear sensitive data, retain preferences

## Resources

- **Repository**: https://github.com/mergisi/awesome-openclaw-agents
- **Deploy Platform**: https://crewclaw.com
- **agents.json**: Machine-readable catalog of all 205+ templates
- **TROUBLESHOOTING.md**: Known issues and recovery procedures
- **Community**: Submit PRs to add your agent templates

## Example: Full Agent Lifecycle

```bash
# 1. Find template
cat agents/development/code-reviewer/SOUL.md

# 2. Customize
cp agents/development/code-reviewer/SOUL.md ./my-reviewer.md
vim my-reviewer.md  # Edit goals, tools, guardrails

# 3. Test locally
cd quickstart
npm install
node bot.js --soul=../my-reviewer.md

# 4. Register
openclaw agents add ../my-reviewer.md --name=my-reviewer

# 5. Start gateway
openclaw gateway start

# 6. Integrate
# Add webhook handler for GitHub PRs

# 7. Monitor
tail -f ~/.openclaw/logs/my-reviewer.log

# 8. Deploy (optional)
crewclaw deploy --agent=../my-reviewer.md --platform=railway
```

This skill covers the essential knowledge for working with awesome-openclaw-agents: browsing templates, customizing SOUL.md files, running agents locally, integrating with tools, and deploying production systems.
