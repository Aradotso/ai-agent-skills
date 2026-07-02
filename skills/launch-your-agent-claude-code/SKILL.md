---
name: launch-your-agent-claude-code
description: Build, deploy, and iterate Claude Managed Agents from idea to production using Claude Code
triggers:
  - build a claude managed agent
  - launch my agent to production
  - create a scheduled claude agent
  - interview me about my agent idea
  - deploy agent to anthropic console
  - grade and iterate my agent
  - set up a recurring agent workflow
  - help me build on claude managed agents
---

# launch-your-agent-claude-code

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

**launch-your-agent** is a Claude Code skill that takes you from an agent idea to a live Claude Managed Agent (CMA) in your Anthropic Console. It orchestrates the full lifecycle: interview → scope v0 → launch in your account → grade against success criteria → iterate → schedule (if recurring).

You walk away with:
- A live managed agent in your Console
- Complete build artifacts in `my-agent/` (build sheet, API payloads, eval scaffold, launch script)
- A graded initial run
- Scheduled deployment (if your task recurs)
- A `NEXT-DIRECTIONS.md` with v1/v2 roadmap

**Primary Language:** HTML (documentation/templates), with JavaScript/TypeScript for agent logic and shell scripts for deployment.

## Installation

```bash
git clone https://github.com/anthropics/launch-your-agent.git
cd launch-your-agent
claude
```

The skills in `.claude/skills/` are auto-loaded when you run Claude Code inside this directory.

## Prerequisites

- **Claude Code** installed and authenticated
- **Anthropic API key** for your own account (create at platform.claude.com → API keys)
- API key stored in local `.env` file (never committed):

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-api03-...
```

## Key Commands

### Main Flow

```
/launch-your-agent
```

Starts the full 4-phase flow:

1. **Interview** – Asks about your use case, inputs, outputs, success criteria, scheduling needs
2. **Stage & Launch** – Generates agent config, creates environment, deploys to your Console
3. **Grade & Iterate** – Runs eval against your success criteria, suggests improvements
4. **Automate** – Sets up scheduled deployment if task is recurring

### Status Check / Close-Out

```
/wrap-up
```

Regenerates overview page, recaps all primitives you built, suggests next 1-2 upgrades.

## Configuration

### CMA Primitives

Claude Managed Agents support:

- **Agent definition**: System prompt, tools, model (`claude-3-5-sonnet-20241022` or `claude-3-7-sonnet-20250219`)
- **Environments**: Isolated runtime contexts (dev/staging/prod)
- **Secrets**: API keys, credentials (encrypted at rest)
- **Scheduled deployments**: Cron-based recurring runs
- **Tool use**: API calls, database queries, file operations

Limits (as of documentation):
- Max system prompt: ~200k tokens
- Tools: Up to 2048 per agent
- Secrets: String values only, accessed via tool parameters

Reference: `cma-primitives.md` in repo root.

### Interview Mapping

The interview phase captures:

| Question | Maps to CMA primitive |
|----------|----------------------|
| What problem does this solve? | Agent system prompt (goals section) |
| What inputs does it need? | Tool definitions + environment secrets |
| What's a successful output? | Eval criteria + success rubric |
| How often should it run? | Scheduled deployment config (cron) |
| What should it NOT do? | System prompt (constraints/guardrails) |

See `interview-to-config.md` for full mapping.

## Code Examples

### Agent Configuration Shape

```typescript
// Generated in my-agent/agent-config.json
{
  "name": "daily-standup-summarizer",
  "model": "claude-3-5-sonnet-20241022",
  "system_prompt": "You are a standup summarizer. Each morning, you...",
  "tools": [
    {
      "name": "fetch_github_prs",
      "description": "Fetch open PRs from team repos",
      "input_schema": {
        "type": "object",
        "properties": {
          "repo": { "type": "string" },
          "github_token": { "type": "string" }
        },
        "required": ["repo", "github_token"]
      }
    }
  ],
  "max_tokens": 4096
}
```

### Launch Script

```bash
#!/bin/bash
# my-agent/launch.sh

set -e

source .env

# 1. Create agent
AGENT_ID=$(curl -s https://api.anthropic.com/v1/agents \
  -H "anthropic-version: 2023-06-01" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "content-type: application/json" \
  -d @agent-config.json | jq -r '.id')

echo "Agent created: $AGENT_ID"

# 2. Create environment
ENV_ID=$(curl -s https://api.anthropic.com/v1/environments \
  -H "anthropic-version: 2023-06-01" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d "{\"name\": \"production\", \"agent_id\": \"$AGENT_ID\"}" | jq -r '.id')

echo "Environment created: $ENV_ID"

# 3. Set secrets
curl -s https://api.anthropic.com/v1/environments/$ENV_ID/secrets \
  -H "anthropic-version: 2023-06-01" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d "{\"key\": \"GITHUB_TOKEN\", \"value\": \"$GITHUB_TOKEN\"}"

# 4. Deploy
curl -s https://api.anthropic.com/v1/agents/$AGENT_ID/deploy \
  -H "anthropic-version: 2023-06-01" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d "{\"environment_id\": \"$ENV_ID\"}"

echo "Deployed to $ENV_ID"
```

### Eval Scaffold

```javascript
// my-agent/eval.js
const Anthropic = require('@anthropic-ai/sdk');

async function gradeRun(runId, successCriteria) {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  // Fetch run output
  const run = await client.agents.runs.retrieve(runId);
  const output = run.output;

  // Grade against criteria
  const gradePrompt = `
Success criteria:
${successCriteria.join('\n')}

Agent output:
${output}

Did the agent meet all criteria? Respond with JSON: {"pass": true/false, "feedback": "..."}
  `;

  const gradeResponse = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 1024,
    messages: [{ role: 'user', content: gradePrompt }],
  });

  return JSON.parse(gradeResponse.content[0].text);
}

module.exports = { gradeRun };
```

### Scheduled Deployment

```json
// my-agent/schedule-config.json
{
  "agent_id": "agt_abc123",
  "environment_id": "env_xyz789",
  "cron": "0 9 * * 1-5",
  "timezone": "America/Los_Angeles"
}
```

```bash
# Apply schedule
curl https://api.anthropic.com/v1/schedules \
  -H "anthropic-version: 2023-06-01" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d @schedule-config.json
```

## Common Patterns

### Pattern: Internal Workflow Agent

**Use case:** Daily standup summary from GitHub + Slack

1. Interview answers:
   - Problem: "Summarize team activity each morning"
   - Inputs: GitHub API token, Slack webhook
   - Success: "Covers all PRs, mentions blockers, <500 words"
   - Schedule: "Every weekday at 9am PT"

2. Generated tools: `fetch_github_prs`, `post_slack_message`
3. System prompt includes: goals, tone (concise), constraints (no speculation)
4. Scheduled deployment with cron `0 9 * * 1-5`

### Pattern: Customer-Facing Agent

**Use case:** Support ticket triage

1. Interview answers:
   - Problem: "Categorize incoming tickets, suggest help articles"
   - Inputs: Zendesk API, knowledge base embeddings
   - Success: "95% category accuracy, links 2+ relevant articles"
   - Schedule: "Real-time via webhook"

2. Generated tools: `search_kb`, `update_ticket_tags`
3. System prompt includes: customer empathy guidelines, escalation rules
4. No cron schedule (webhook-triggered)

### Pattern: Data Pipeline Agent

**Use case:** Weekly analytics rollup

1. Interview answers:
   - Problem: "Aggregate usage metrics, detect anomalies"
   - Inputs: Postgres connection, previous week's baseline
   - Success: "Flags any >20% change, generates CSV"
   - Schedule: "Mondays at 6am UTC"

2. Generated tools: `query_db`, `write_csv`, `send_email`
3. System prompt includes: statistical thresholds, alert format
4. Scheduled deployment with cron `0 6 * * 1`

## Troubleshooting

### "API key not found"

Ensure `.env` file exists in repo root with:
```bash
ANTHROPIC_API_KEY=sk-ant-api03-...
```

**Never** commit `.env` — it's in `.gitignore` by default.

### "Agent creation failed: invalid tool schema"

Check `my-agent/agent-config.json`:
- `input_schema` must be valid JSON Schema
- `required` fields must be present in `properties`
- Tool names must be lowercase with underscores

### "Eval keeps failing"

Refine success criteria in `my-agent/build-sheet.md`:
- Make criteria measurable (e.g., "mentions 3+ PRs" not "comprehensive")
- Add edge cases to test set
- Iterate system prompt with `/launch-your-agent` (picks up where it left off)

### "Schedule not triggering"

Verify:
- Cron syntax valid (use https://crontab.guru)
- Timezone correct (defaults to UTC)
- Environment has required secrets set
- Check Console → Agents → Runs for error logs

### "Rate limit hit during launch"

CMA API respects standard Anthropic rate limits. If hitting during deploy:
- Add `sleep 2` between curl commands in `launch.sh`
- Or batch secret creation into single call

## File Structure After Launch

```
my-agent/
├── build-sheet.md          # Interview answers + scoping decisions
├── agent-config.json       # Agent definition (system prompt, tools, model)
├── environment-config.json # Environment + secrets references
├── schedule-config.json    # Cron schedule (if recurring)
├── launch.sh               # Resumable deploy script
├── eval.js                 # Grading logic
├── OVERVIEW.md             # Human-readable status page
└── NEXT-DIRECTIONS.md      # v1/v2 roadmap

.env                        # API keys (git-ignored)
```

## API Reference

See official docs: https://platform.claude.com/docs/en/managed-agents/overview

Key endpoints:
- `POST /v1/agents` – Create agent
- `POST /v1/environments` – Create environment
- `POST /v1/environments/{id}/secrets` – Set secrets
- `POST /v1/agents/{id}/deploy` – Deploy to environment
- `POST /v1/schedules` – Create cron schedule
- `GET /v1/agents/runs/{id}` – Fetch run output

## Additional Resources

- `cma-primitives.md` – Full inventory of CMA features and limits
- `interview-to-config.md` – Detailed interview → config mapping
- `examples-bank.md` – Sourced agent examples and proof points
- `ui/` – Example overview page templates

## License

Apache 2.0 (see LICENSE in repo root)
