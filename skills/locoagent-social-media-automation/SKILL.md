---
name: locoagent-social-media-automation
description: AI-powered social media agent with real browser automation for autonomous account operation
triggers:
  - "automate social media with AI agent"
  - "set up browser automation for X.com"
  - "create autonomous social media workflow"
  - "build AI agent for Twitter engagement"
  - "use LocoAgent for social posting"
  - "configure platform skills for social automation"
  - "schedule AI-driven social media tasks"
  - "create custom workflow for social platforms"
---

# LocoAgent Social Media Automation

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

LocoAgent is an AI-powered social media agent that autonomously operates social media accounts through real browser automation. It combines an LLM-driven agentic loop with Chrome DevTools Protocol (CDP) to perceive, decide, and act on live web pages — performing tasks like liking posts, writing replies, following users, and publishing content.

**Key capabilities:**
- Real browser automation with Chrome CDP (uses actual login sessions)
- Platform skill system (32+ operations for X.com built-in)
- Workflow engine for deterministic automation pipelines
- Operation log for persistent deduplication across sessions
- Multi-provider LLM support (OpenRouter, DeepSeek, Ollama, etc.)

## Installation

### Prerequisites

Install required dependencies:

```bash
# Install Bun runtime
curl -fsSL https://bun.sh/install | bash

# Install agent-browser CLI
npm install -g @vercel/agent-browser
```

### Project Setup

```bash
git clone https://github.com/LocoreMind/locoagent.git
cd locoagent
bun install
```

### Configuration

Create `.env` file in project root:

```env
# OpenRouter (recommended - access 200+ models)
CLAUDE_CODE_USE_OPENAI=1
OPENAI_API_KEY=sk-or-v1-...
OPENAI_BASE_URL=https://openrouter.ai/api/v1
OPENAI_MODEL=anthropic/claude-sonnet-4.5

# Required for automated mode
SKIP_PERMISSIONS=1
```

Alternative provider configurations:

```env
# DeepSeek (with thinking mode)
CLAUDE_CODE_USE_OPENAI=1
OPENAI_API_KEY=<DEEPSEEK_API_KEY>
OPENAI_BASE_URL=https://api.deepseek.com
OPENAI_MODEL=deepseek-v4-flash

# Ollama (local models)
CLAUDE_CODE_USE_OPENAI=1
OPENAI_API_KEY=ollama
OPENAI_BASE_URL=http://localhost:11434/v1
OPENAI_MODEL=llama3.2

# Anthropic direct (native SDK)
ANTHROPIC_API_KEY=<ANTHROPIC_API_KEY>
```

### Browser Setup

```bash
# One-time: copy Chrome profile and launch with CDP
bun run setup-chrome

# Connect agent-browser to running Chrome
agent-browser connect 9222
```

## Core Commands

### Interactive Mode

```bash
# Start interactive session
bun start

# Load X.com skill and execute task
> /x-com open home timeline, like first 3 posts about AI

# Check operation history
> /operation-log recent --limit 20
```

### Headless Mode

```bash
# Single query execution
bun start -p "open X.com and like the first post about AI agents"

# With specific model
bun start --model anthropic/claude-sonnet-4.5 -p "/x-com like 5 posts about LLMs"

# Platform-specific task
bun start -p "/x-com like 5 posts about 'large language models', then follow the authors"
```

## Platform Skills

Skills inject complete operation playbooks into the agent's context.

### X.com Skill

```bash
# Interactive
> /x-com open home timeline, like first 3 posts about AI, reply to the best one

# Headless
bun start -p "/x-com like 5 posts about 'machine learning', follow authors with >1k followers"
```

Available X.com operations (32+):
- Navigation: home, notifications, messages, profile, search
- Engagement: like, retweet, reply, quote tweet
- Social graph: follow, unfollow, mute, block
- Content: post tweet, post thread, upload media
- Profile: edit bio, change avatar, update banner
- Lists: create, add members, view

### Creating Custom Skills

Create `skills/linkedin/SKILL.md`:

```markdown
---
description: "LinkedIn platform operations playbook"
allowed-tools:
  - Bash
user-invocable: true
---

# LinkedIn Operations

## 1. Navigation

### Open Home Feed
```bash
agent-browser open https://www.linkedin.com/feed
```

### Search Posts
```bash
agent-browser open "https://www.linkedin.com/search/results/content/?keywords=AI%20agents"
agent-browser snapshot -i -c -s 'div[data-post-id]'
```

## 2. Engagement

### Like Post
1. Find post element with `agent-browser snapshot -i`
2. Locate like button (usually `button[aria-label*="Like"]`)
3. Click: `agent-browser click @e<ref>`

### Comment on Post
1. Find comment input (usually `div[role="textbox"]`)
2. Click to focus: `agent-browser click @e<ref>`
3. Type comment: `agent-browser fill @e<ref> "Insightful post!"`
4. Find submit button and click
```

Load the skill:

```bash
bun start
> /linkedin search for posts about 'AI safety', like top 3
```

## Workflow Engine

Workflows are deterministic browser-automation pipelines that run without LLM involvement.

### Built-in Workflows

```bash
# List all workflows
bun run workflow list

# Run once (blocking)
bun run workflow run --id hf-papers-to-x

# Run once (background)
bun run workflow start --id hf-papers-to-x

# Daemon mode (every 3 minutes)
bun run workflow daemon --id x-search-reply --interval 3

# Stop workflow
bun run workflow stop --id x-search-reply

# View status
bun run workflow status

# View execution history
bun run workflow history --id hf-papers-to-x
```

### Creating Custom Workflows

**Step 1:** Create workflow definition `workflows/linkedin-engagement.json`:

```json
{
  "id": "linkedin-engagement",
  "name": "LinkedIn Daily Engagement",
  "description": "Search for AI posts on LinkedIn and engage",
  "schedule": "daily",
  "executor": "executors/linkedin-engagement.ts",
  "config": {
    "searchQuery": "artificial intelligence",
    "maxPosts": 5,
    "cdpPort": 9222
  }
}
```

**Step 2:** Create executor `workflows/executors/linkedin-engagement.ts`:

```typescript
#!/usr/bin/env bun
import { execSync } from 'node:child_process'

// Parse config from workflow engine
const configArg = process.argv.find((_, i, a) => a[i - 1] === '--config')
const config = JSON.parse(configArg!)

// agent-browser helper
function ab(cmd: string): string {
  return execSync(`agent-browser --cdp ${config.cdpPort} ${cmd}`, {
    encoding: 'utf-8', 
    timeout: 30000,
  }).trim()
}

// Helper to check operation log
function hasEngaged(postUrl: string): boolean {
  try {
    execSync(`bun run scripts/log-operation.ts check --platform linkedin --action like --url "${postUrl}"`, {
      encoding: 'utf-8',
      stdio: 'ignore'
    })
    return true // exit 0 = already done
  } catch {
    return false // exit 1 = not done
  }
}

// Helper to log operation
function logOperation(postUrl: string, action: string, status: string, note: string) {
  execSync(`bun run scripts/log-operation.ts add --platform linkedin --action ${action} --url "${postUrl}" --status ${status} --note "${note}"`, {
    encoding: 'utf-8',
    stdio: 'inherit'
  })
}

console.error('[linkedin-engagement] Starting workflow...')

// Step 1: Navigate to search
console.error(`[linkedin-engagement] Searching for: ${config.searchQuery}`)
const searchUrl = `https://www.linkedin.com/search/results/content/?keywords=${encodeURIComponent(config.searchQuery)}`
ab(`open "${searchUrl}"`)
ab('wait 3000')

// Step 2: Get posts
console.error('[linkedin-engagement] Getting posts...')
const snapshot = ab('snapshot -i -c -s \'div[data-post-id]\'')
const posts = JSON.parse(snapshot)

let engaged = 0
const stepsTotal = Math.min(posts.length, config.maxPosts)

// Step 3: Engage with posts
for (let i = 0; i < stepsTotal; i++) {
  const post = posts[i]
  const postUrl = post.attributes?.['data-urn'] || `post-${i}`
  
  // Check if already engaged
  if (hasEngaged(postUrl)) {
    console.error(`[linkedin-engagement] Already engaged with ${postUrl}, skipping`)
    continue
  }
  
  // Find like button
  const likeButton = post.children?.find((el: any) => 
    el.attributes?.['aria-label']?.includes('Like')
  )
  
  if (likeButton?.ref) {
    ab(`click ${likeButton.ref}`)
    logOperation(postUrl, 'like', 'success', `Workflow: ${config.searchQuery}`)
    engaged++
    console.error(`[linkedin-engagement] Liked post ${i + 1}/${stepsTotal}`)
    ab('wait 2000') // Rate limiting
  }
}

// Output final summary (required)
console.log(JSON.stringify({ 
  stepsCompleted: engaged, 
  stepsTotal,
  searchQuery: config.searchQuery 
}))
```

**Step 3:** Run workflow:

```bash
bun run workflow run --id linkedin-engagement
```

## Operation Log

Persistent memory prevents duplicate actions across sessions.

### Check Before Acting

```typescript
import { execSync } from 'node:child_process'

function hasLiked(postUrl: string): boolean {
  try {
    execSync(`bun run scripts/log-operation.ts check --platform x --action like --url "${postUrl}"`, {
      encoding: 'utf-8',
      stdio: 'ignore'
    })
    return true // exit 0 = already done
  } catch {
    return false // exit 1 = not done
  }
}

const url = "https://x.com/user/status/123"
if (hasLiked(url)) {
  console.log("Already liked this post")
} else {
  // Perform like action
  execSync(`agent-browser click @e5`)
  
  // Log operation
  execSync(`bun run scripts/log-operation.ts add --platform x --action like --url "${url}" --status success --note "AI research post"`)
}
```

### CLI Operations

```bash
# Check if operation was performed (exit 0 = done, exit 1 = not done)
bun run scripts/log-operation.ts check \
  --platform x \
  --action like \
  --url "https://x.com/user/status/123"

# Record operation
bun run scripts/log-operation.ts add \
  --platform x \
  --action like \
  --url "https://x.com/user/status/123" \
  --status success \
  --note "AI agents research post"

# View recent operations
bun run scripts/log-operation.ts recent --limit 20

# 30-day summary
bun run scripts/log-operation.ts summary --days 30
```

State stored in `persona/operation-log.json`.

## Task Scheduling

Structure daily/weekly tasks instead of ad-hoc prompts.

### Define Tasks

Edit `persona/tasks.md`:

```markdown
## Daily Tasks
1. Engage with AI research content (like 5-10 posts)
2. Monitor project mentions and respond
3. Leave 1-2 technical comments on relevant posts

## Weekly Tasks (Monday)
4. Follow 3-5 relevant researchers or developers
5. Post 1 original tweet about recent findings

## Session Constraints
| Action   | Max per session |
|----------|----------------|
| Likes    | 10             |
| Comments | 2              |
| Follows  | 5              |
| Posts    | 1              |
```

### Run Tasks

```bash
# Execute today's tasks
bun run run-tasks

# Preview prompt without running
bun run run-tasks:dry

# Restrict to one platform
bun run run-tasks -- --platform x
```

## Real-time Trajectory Monitor

Watch live execution status instead of black-box `--print` mode.

```bash
# Terminal 1: start monitor
bun run tail

# Terminal 2: run agent
bun start -p "/x-com open timeline, like first post"
```

Output shows live execution:

```
═══ New Task ═══
/x-com open timeline, like first post

[6:30:47 PM] ⚡ Bash: agent-browser connect 9222
[6:30:47 PM] ✓ Result: Done
[6:31:10 PM] ⚡ Bash: agent-browser open https://x.com/home
[6:31:27 PM] ⚡ Bash: agent-browser snapshot -i -c -s 'article'
[6:31:44 PM] ● Agent: Found first post, like button ref=e136
[6:31:44 PM] ⚡ Bash: agent-browser click e136
[6:31:45 PM] ✓ Result: Done
```

Additional commands:

```bash
# Replay latest session from beginning
bun run tail:history

# List recent sessions
bun run tail:list

# Watch specific session
bun run tail <session-id>
```

## Common Patterns

### Pattern: Safe Engagement Loop

```typescript
#!/usr/bin/env bun
import { execSync } from 'node:child_process'

function ab(cmd: string): string {
  return execSync(`agent-browser --cdp 9222 ${cmd}`, {
    encoding: 'utf-8',
    timeout: 30000,
  }).trim()
}

function hasEngaged(platform: string, action: string, url: string): boolean {
  try {
    execSync(`bun run scripts/log-operation.ts check --platform ${platform} --action ${action} --url "${url}"`, {
      stdio: 'ignore'
    })
    return true
  } catch {
    return false
  }
}

function logEngagement(platform: string, action: string, url: string, note: string) {
  execSync(`bun run scripts/log-operation.ts add --platform ${platform} --action ${action} --url "${url}" --status success --note "${note}"`, {
    stdio: 'inherit'
  })
}

// Navigate to page
ab('open https://x.com/search?q=AI%20agents&f=live')
ab('wait 3000')

// Get posts
const snapshot = JSON.parse(ab('snapshot -i -c -s \'article\''))
const posts = snapshot.slice(0, 5)

for (const post of posts) {
  const postUrl = post.attributes?.['data-testid'] || `post-${Math.random()}`
  
  // Skip if already engaged
  if (hasEngaged('x', 'like', postUrl)) {
    console.error(`Already liked ${postUrl}`)
    continue
  }
  
  // Find like button
  const likeBtn = post.children?.find((el: any) => 
    el.attributes?.['data-testid'] === 'like'
  )
  
  if (likeBtn?.ref) {
    ab(`click ${likeBtn.ref}`)
    logEngagement('x', 'like', postUrl, 'AI agents search result')
    ab('wait 2000') // Rate limiting
  }
}
```

### Pattern: Multi-Step Workflow with Checkpoints

```typescript
#!/usr/bin/env bun
import { execSync } from 'node:child_process'
import { writeFileSync, readFileSync, existsSync } from 'fs'

const CHECKPOINT_FILE = '/tmp/workflow-checkpoint.json'

function loadCheckpoint(): any {
  if (existsSync(CHECKPOINT_FILE)) {
    return JSON.parse(readFileSync(CHECKPOINT_FILE, 'utf-8'))
  }
  return { step: 0, data: {} }
}

function saveCheckpoint(step: number, data: any) {
  writeFileSync(CHECKPOINT_FILE, JSON.stringify({ step, data }))
}

const checkpoint = loadCheckpoint()
let currentStep = checkpoint.step

// Step 1: Fetch data
if (currentStep === 0) {
  console.error('[workflow] Step 1: Fetching data...')
  const data = { papers: ['paper1', 'paper2', 'paper3'] }
  saveCheckpoint(1, data)
  currentStep = 1
}

// Step 2: Process data
if (currentStep === 1) {
  console.error('[workflow] Step 2: Processing data...')
  const { data } = loadCheckpoint()
  // Process papers
  saveCheckpoint(2, { ...data, processed: true })
  currentStep = 2
}

// Step 3: Post to social
if (currentStep === 2) {
  console.error('[workflow] Step 3: Posting to social...')
  const { data } = loadCheckpoint()
  // Post each paper
  saveCheckpoint(3, data)
  currentStep = 3
}

// Cleanup checkpoint on success
if (existsSync(CHECKPOINT_FILE)) {
  execSync(`rm ${CHECKPOINT_FILE}`)
}

console.log(JSON.stringify({ stepsCompleted: 3, stepsTotal: 3 }))
```

## Troubleshooting

### Browser Connection Issues

```bash
# Check if Chrome is running with CDP
ps aux | grep chrome | grep remote-debugging-port

# Kill existing Chrome and restart
pkill -f chrome
bun run setup-chrome

# Verify CDP port is accessible
curl http://localhost:9222/json/version
```

### Operation Log Not Working

```bash
# Check log file exists and is readable
cat persona/operation-log.json

# Reset log if corrupted
echo '[]' > persona/operation-log.json

# Verify log script works
bun run scripts/log-operation.ts recent --limit 5
```

### Workflow Execution Fails

```bash
# Check workflow status
bun run workflow status

# View detailed logs
bun run workflow history --id <workflow-id>

# Run with debug output
DEBUG=1 bun run workflow run --id <workflow-id>

# Check executor is executable
chmod +x workflows/executors/<executor>.ts
```

### LLM Provider Errors

```bash
# Verify API key is set
echo $OPENAI_API_KEY

# Test connection
curl -H "Authorization: Bearer $OPENAI_API_KEY" \
  $OPENAI_BASE_URL/models

# Check model availability
bun start --model <model-name> -p "test"
```

### Agent Not Finding Elements

```bash
# Get detailed snapshot
agent-browser snapshot -i -c -s 'article' > snapshot.json

# Check element refs are valid
cat snapshot.json | jq '.[] | .ref'

# Try broader selector
agent-browser snapshot -i -c -s 'div'

# Wait for page to load
agent-browser wait 5000
agent-browser snapshot -i
```
