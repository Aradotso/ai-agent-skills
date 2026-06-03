---
name: duel-agents-routing
description: CLI, SDK, and IDE plugins for Duel Agents - an LLM routing layer that runs prompts against multiple models and picks the cheapest winning answer
triggers:
  - "set up duel agents routing"
  - "install duel agents for cursor"
  - "configure duel agents api"
  - "use duel agents sdk"
  - "route llm requests through duel"
  - "integrate duel agents with claude code"
  - "add multi-model llm routing"
  - "configure openclaw with duel agents"
---

# Duel Agents Routing

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

Duel Agents is an IDE-native routing layer that runs prompts against multiple LLM models simultaneously and selects the cheapest answer that meets quality thresholds. It provides:

- **Automatic model selection** via `duel-auto` model name
- **Cost optimization** by choosing cheaper models when quality is sufficient
- **OpenAI-compatible** and **Anthropic-compatible** APIs
- **IDE integrations** for Claude Code, Cursor, Codex, and OpenClaw
- **TypeScript SDK** for building custom agents and applications

The service routes traffic through `https://duelagents.com/v1` using Duel API keys (`duel_<prefix>_<secret>`).

## Prerequisites

**You must have a Duel API key.** Raw Anthropic or OpenAI keys will not work.

1. Subscribe at [https://duelagents.com/dashboard/settings](https://duelagents.com/dashboard/settings)
2. Create an API key from the dashboard
3. Key format: `duel_` + 8 characters + `_` + 32 characters

## Installation

### Quick Start (All Tools)

```bash
# Set your API key
export DUEL_API_KEY=duel_yourprefix_yoursecret

# Install for all supported tools
npx @duel-agents/install all

# Verify installation
npx @duel-agents/install doctor
```

### Per-Tool Installation

```bash
# Claude Code
npx @duel-agents/install claude-code

# Cursor IDE
npx @duel-agents/install cursor

# Codex CLI
npx @duel-agents/install codex

# OpenClaw
npx @duel-agents/install openclaw
```

### SDK for Custom Applications

```bash
npm install @duel-agents/sdk
```

## Configuration

### Environment Variables

| Variable | Required | Purpose |
|----------|----------|---------|
| `DUEL_API_KEY` | Yes | Your Duel API key from the dashboard |
| `DUEL_AGENTS_API_KEY` | No | Alias for `DUEL_API_KEY` |
| `DUEL_PROXY_URL` | No | Override proxy URL (staging environments only) |
| `OPENCLAW_CONFIG_PATH` | No | Custom OpenClaw config file path |

### Setting Environment Variables

**.env file:**
```bash
DUEL_API_KEY=duel_abc12345_def67890ghijklmnopqrstuvwxyz12
```

**Shell export:**
```bash
export DUEL_API_KEY=duel_abc12345_def67890ghijklmnopqrstuvwxyz12
```

## SDK Usage

### OpenAI-Compatible API

```typescript
import { DuelClient } from "@duel-agents/sdk";

const duel = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!, // required
});

// Chat completions
const response = await duel.chat.completions.create({
  model: "duel-auto", // automatic model selection
  messages: [
    { role: "system", content: "You are a helpful coding assistant." },
    { role: "user", content: "Explain TypeScript generics briefly." }
  ],
  temperature: 0.7,
  max_tokens: 500,
});

console.log(response.choices[0].message.content);
```

### Anthropic-Compatible API

```typescript
import { DuelClient } from "@duel-agents/sdk";

const duel = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!,
});

const message = await duel.messages.create({
  model: "duel-auto",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "Write a Python function to parse JSON safely." }
  ],
});

console.log(message.content);
```

### Streaming Responses

```typescript
const stream = await duel.chat.completions.create({
  model: "duel-auto",
  messages: [{ role: "user", content: "Count to 10" }],
  stream: true,
});

for await (const chunk of stream) {
  const content = chunk.choices[0]?.delta?.content;
  if (content) {
    process.stdout.write(content);
  }
}
```

### Error Handling

```typescript
import { DuelClient } from "@duel-agents/sdk";

const duel = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!,
});

try {
  const response = await duel.chat.completions.create({
    model: "duel-auto",
    messages: [{ role: "user", content: "Hello" }],
  });
} catch (error) {
  if (error.status === 401) {
    console.error("Invalid API key or subscription inactive");
  } else if (error.status === 429) {
    console.error("Rate limit exceeded");
  } else {
    console.error("Request failed:", error.message);
  }
}
```

## IDE Integrations

### Claude Code

```bash
# Clone repo and install plugin
git clone https://github.com/2aronS/Duel-Agents.git
cd duel-agents
claude plugin install ./integrations/claude-plugin

# Run installer
npx @duel-agents/install claude-code
```

**In Claude Code:**
```
/duel-agents:setup
```

Follow the guided setup to configure your API key and preferences.

### Cursor

The installer copies a skill to `.cursor/skills/duel-agents/` and writes `DUEL_API_KEY` to your project `.env`.

**Manual configuration required:**

1. Open Cursor Settings → Models
2. Set **Override OpenAI Base URL** to: `https://duelagents.com/v1`
3. Set **API Key** to your Duel key: `duel_yourprefix_yoursecret`
4. Select `duel-auto` as your model

```bash
npx @duel-agents/install cursor
```

### Codex CLI

The installer writes `OPENAI_BASE_URL` and `OPENAI_API_KEY` to `.env`:

```bash
npx @duel-agents/install codex
# Restart Codex after installation
```

### OpenClaw

Patches `~/.openclaw/openclaw.json` with a `duel` provider and sets the default model:

```bash
npx @duel-agents/install openclaw

# Validate configuration
openclaw config validate
```

**Manual configuration** (`~/.openclaw/openclaw.json`):
```json
{
  "providers": {
    "duel": {
      "apiKey": "duel_yourprefix_yoursecret",
      "baseURL": "https://duelagents.com/v1"
    }
  },
  "defaultModel": "duel/duel-auto"
}
```

A backup is created at `openclaw.json.bak` before patching.

## CLI Commands

### Install Command

```bash
# Install for specific tool
npx @duel-agents/install <tool>

# Available tools:
# - claude-code
# - cursor
# - codex
# - openclaw
# - all
```

### Doctor Command

Validates your setup and connectivity:

```bash
npx @duel-agents/install doctor
```

Checks:
- API key format validity
- Connectivity to `duelagents.com/v1`
- Subscription status
- Environment variable configuration

## Common Patterns

### Multi-Agent System

```typescript
import { DuelClient } from "@duel-agents/sdk";

const duel = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!,
});

async function coordinatedAgents(task: string) {
  // Planner agent
  const plan = await duel.chat.completions.create({
    model: "duel-auto",
    messages: [
      { role: "system", content: "You are a task planning agent." },
      { role: "user", content: `Create a plan for: ${task}` }
    ],
  });

  const steps = plan.choices[0].message.content;

  // Executor agent
  const execution = await duel.chat.completions.create({
    model: "duel-auto",
    messages: [
      { role: "system", content: "You are a task execution agent." },
      { role: "user", content: `Execute this plan:\n${steps}` }
    ],
  });

  return execution.choices[0].message.content;
}
```

### Cost-Aware Routing

```typescript
import { DuelClient } from "@duel-agents/sdk";

const duel = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!,
});

async function analyzeCode(code: string, complexity: "simple" | "complex") {
  // Duel automatically routes to cheaper models for simple tasks
  const response = await duel.chat.completions.create({
    model: "duel-auto",
    messages: [
      {
        role: "user",
        content: `Analyze this ${complexity} code:\n\n${code}`
      }
    ],
    // Duel uses prompt complexity to select optimal model
  });

  return response.choices[0].message.content;
}
```

### Use with Existing OpenAI Code

Replace only the client initialization:

```typescript
// Before:
// import OpenAI from "openai";
// const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// After:
import { DuelClient } from "@duel-agents/sdk";
const client = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!,
});

// All existing OpenAI code works unchanged
const response = await client.chat.completions.create({
  model: "duel-auto", // or keep your existing model name
  messages: [{ role: "user", content: "Hello" }],
});
```

### Fallback Strategy

```typescript
import { DuelClient } from "@duel-agents/sdk";

const duel = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!,
});

async function robustCompletion(prompt: string) {
  try {
    // Try Duel routing first (cost-optimized)
    return await duel.chat.completions.create({
      model: "duel-auto",
      messages: [{ role: "user", content: prompt }],
    });
  } catch (error) {
    console.warn("Duel routing failed, using direct fallback");
    // Fallback logic here if needed
    throw error;
  }
}
```

## Troubleshooting

### Invalid API Key Format

**Symptom:** `Invalid API key format` error

**Solution:**
- Key must match pattern: `duel_` + 8 chars + `_` + 32 chars
- Create a new key at [https://duelagents.com/dashboard/settings](https://duelagents.com/dashboard/settings)
- Verify no extra whitespace: `echo "$DUEL_API_KEY" | wc -c` should be 46

### 401 Unauthorized

**Symptom:** `401` error when running doctor or making requests

**Solution:**
- Key may be revoked or subscription inactive
- Check billing status at dashboard
- Generate a new API key
- Ensure key is exported: `echo $DUEL_API_KEY`

### Could Not Reach Duel API

**Symptom:** Connection errors to `duelagents.com/v1`

**Solution:**
- Verify internet connectivity
- Check if `duelagents.com` is accessible: `curl https://duelagents.com/v1/health`
- Proxy may be temporarily down; retry after a few minutes
- Check for firewall or VPN restrictions

### Cursor Still Uses OpenAI

**Symptom:** Cursor not routing through Duel

**Solution:**
1. Open Settings → Models → Override OpenAI Base URL
2. Set to: `https://duelagents.com/v1`
3. Set API key field to your `duel_*` key (not OpenAI key)
4. Restart Cursor
5. Select `duel-auto` or `duel/duel-auto` as model

### OpenClaw Won't Start

**Symptom:** OpenClaw fails after Duel installation

**Solution:**
```bash
# Validate configuration
openclaw config validate

# Restore from backup if needed
cp ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json

# Reinstall
npx @duel-agents/install openclaw
```

### Skill Copy Failed

**Symptom:** Cursor skill not found after npm install

**Solution:**
```bash
# Rebuild the package
git clone https://github.com/2aronS/Duel-Agents.git
cd duel-agents
npm install
npm run build

# Reinstall
npx @duel-agents/install cursor
```

### TypeScript Type Errors

**Symptom:** TypeScript errors with SDK

**Solution:**
```bash
# Ensure latest version
npm install @duel-agents/sdk@latest

# Check TypeScript version (>= 4.5 recommended)
npm list typescript
```

## Model Selection

### Available Models

- `duel-auto` - Automatic routing (recommended)
- Any OpenAI model name (routed through Duel)
- Any Anthropic model name (routed through Duel)

### How Auto-Routing Works

Duel analyzes:
1. Prompt complexity
2. Required capabilities (code, reasoning, creative)
3. Token length
4. Historical quality metrics

Then selects the cheapest model that meets the quality threshold for that specific request.

## Advanced Configuration

### Custom Base URL (Staging)

```typescript
import { DuelClient } from "@duel-agents/sdk";

const duel = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!,
  baseURL: process.env.DUEL_PROXY_URL || "https://duelagents.com/v1",
});
```

### Timeout Configuration

```typescript
import { DuelClient } from "@duel-agents/sdk";

const duel = new DuelClient({
  apiKey: process.env.DUEL_API_KEY!,
  timeout: 60000, // 60 seconds
});
```

## Repository Structure

```
packages/
  core/       @duel-agents/core    - Validation, env handling, connectivity
  cli/        @duel-agents/install - Installer CLI tool
  sdk/        @duel-agents/sdk     - TypeScript API client

integrations/
  claude-plugin/  - Claude Code plugin
  cursor-skill/   - Cursor IDE skill
  openclaw-skill/ - OpenClaw integration

templates/
  cursor-models.override.md - Cursor configuration example
  openclaw.duel.json5       - OpenClaw configuration example
```

## Development

```bash
# Clone repository
git clone https://github.com/2aronS/Duel-Agents.git
cd duel-agents

# Install dependencies
npm install

# Build all packages
npm run build

# Run tests
npm test

# Run installer locally
npm run cli -- install doctor
```

## License

MIT License - see repository for full text.
