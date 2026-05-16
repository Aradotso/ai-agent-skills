---
name: open-multi-agent-orchestration
description: TypeScript-native multi-agent orchestration framework that decomposes goals into task DAGs automatically with MCP and live tracing
triggers:
  - create a multi-agent team
  - orchestrate multiple AI agents
  - set up agent collaboration
  - use open-multi-agent
  - build an agent workflow
  - coordinate AI agents with tasks
  - create agent team with shared memory
  - implement multi-agent system
---

# Open Multi-Agent Orchestration

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Open Multi-Agent is a TypeScript-native multi-agent orchestration framework that automatically decomposes goals into task DAGs, parallelizes independent tasks, and synthesizes results. It supports 10+ LLM providers, built-in tools, MCP server integration, and has only three runtime dependencies.

## Installation

```bash
npm install @open-multi-agent/core
```

**Requirements:** Node.js >= 18

## Core Concepts

### Three Execution Modes

1. **Single Agent** - One agent, one prompt
2. **Auto-orchestrated Team** - Coordinator decomposes goal into tasks automatically
3. **Explicit Pipeline** - You define the task graph and assignments

### Basic Single Agent

```typescript
import { OpenMultiAgent } from '@open-multi-agent/core'

const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
})

const result = await orchestrator.runAgent({
  name: 'coder',
  systemPrompt: 'You are an expert TypeScript developer.',
  tools: ['bash', 'file_write', 'file_read'],
}, 'Create a simple Express server in /tmp/api')

console.log(result.success) // true
console.log(result.content) // agent's final response
console.log(result.totalTokenUsage) // { input_tokens: 1234, output_tokens: 567 }
```

### Auto-Orchestrated Team (Recommended)

The coordinator agent decomposes your goal into a task DAG and executes it:

```typescript
import { OpenMultiAgent, type AgentConfig } from '@open-multi-agent/core'

const agents: AgentConfig[] = [
  {
    name: 'architect',
    model: 'claude-sonnet-4-6',
    systemPrompt: 'Design clean API contracts and data models.',
    tools: ['file_write'],
  },
  {
    name: 'developer',
    model: 'claude-sonnet-4-6',
    systemPrompt: 'Implement runnable TypeScript code.',
    tools: ['bash', 'file_read', 'file_write', 'file_edit'],
  },
  {
    name: 'reviewer',
    model: 'claude-sonnet-4-6',
    systemPrompt: 'Review code for correctness and security.',
    tools: ['file_read', 'grep'],
  },
]

const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
  onProgress: (event) => {
    console.log(event.type, event.task ?? event.agent ?? '')
  },
})

const team = orchestrator.createTeam('api-team', {
  name: 'api-team',
  agents,
  sharedMemory: true, // agents can share context
})

const result = await orchestrator.runTeam(
  team,
  'Create a REST API for a todo list in /tmp/todo-api/'
)

console.log(result.success)
console.log(result.content) // synthesized final result
console.log(result.totalTokenUsage.output_tokens)
```

### Explicit Task Pipeline

When you know the exact workflow:

```typescript
import { OpenMultiAgent, type TaskConfig } from '@open-multi-agent/core'

const tasks: TaskConfig[] = [
  {
    id: 'design',
    description: 'Design the API schema',
    assignedTo: 'architect',
    dependencies: [],
  },
  {
    id: 'implement',
    description: 'Implement the endpoints',
    assignedTo: 'developer',
    dependencies: ['design'],
  },
  {
    id: 'test',
    description: 'Write integration tests',
    assignedTo: 'developer',
    dependencies: ['implement'],
  },
  {
    id: 'review',
    description: 'Security and code review',
    assignedTo: 'reviewer',
    dependencies: ['implement', 'test'],
  },
]

const result = await orchestrator.runTasks(team, tasks)
```

## Provider Configuration

### Environment Variables

```bash
# Anthropic
export ANTHROPIC_API_KEY=sk-ant-...

# OpenAI
export OPENAI_API_KEY=sk-...

# Google Gemini
export GEMINI_API_KEY=...

# DeepSeek
export DEEPSEEK_API_KEY=sk-...

# Azure OpenAI
export AZURE_OPENAI_API_KEY=...
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com
export AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4
export AZURE_OPENAI_API_VERSION=2024-02-15-preview

# Ollama (local)
# No API key needed, runs on localhost:11434 by default
```

### Using Multiple Providers in One Team

```typescript
const agents: AgentConfig[] = [
  {
    name: 'planner',
    model: 'claude-sonnet-4-6', // Anthropic
    systemPrompt: 'Create detailed plans.',
  },
  {
    name: 'coder',
    model: 'gpt-4o', // OpenAI
    systemPrompt: 'Write production-grade code.',
    tools: ['bash', 'file_write'],
  },
  {
    name: 'local-reviewer',
    model: 'ollama:qwen2.5-coder:32b', // Local Ollama
    systemPrompt: 'Review code for bugs.',
    tools: ['file_read', 'grep'],
  },
]
```

### Ollama (Local Models)

```typescript
const orchestrator = new OpenMultiAgent({
  defaultModel: 'ollama:qwen2.5-coder:7b',
})

const result = await orchestrator.runAgent({
  name: 'local-coder',
  model: 'ollama:deepseek-coder-v2:16b',
  systemPrompt: 'You write Python code.',
  tools: ['bash', 'file_write'],
}, 'Create a FastAPI hello world in /tmp/api.py')
```

## Tools

### Built-in Tools

Available out of the box:

- `bash` - Execute shell commands
- `file_read` - Read file contents
- `file_write` - Write files
- `file_edit` - Edit files using search/replace
- `grep` - Search file contents
- `glob` - List files matching patterns

```typescript
const agent: AgentConfig = {
  name: 'dev',
  systemPrompt: 'You are a developer.',
  tools: ['bash', 'file_read', 'file_write', 'file_edit', 'grep'],
}
```

### Custom Tools with Zod

```typescript
import { defineTool } from '@open-multi-agent/core'
import { z } from 'zod'

const weatherTool = defineTool({
  name: 'get_weather',
  description: 'Get current weather for a city',
  parameters: z.object({
    city: z.string().describe('City name'),
    units: z.enum(['celsius', 'fahrenheit']).default('celsius'),
  }),
  execute: async ({ city, units }) => {
    // Your implementation
    const temp = units === 'celsius' ? 22 : 72
    return `Weather in ${city}: ${temp}°${units === 'celsius' ? 'C' : 'F'}`
  },
})

const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
  customTools: [weatherTool],
})

const result = await orchestrator.runAgent({
  name: 'assistant',
  tools: ['get_weather'],
}, 'What is the weather in London?')
```

### MCP Server Integration

Connect Model Context Protocol servers:

```typescript
import { connectMCPTools } from '@open-multi-agent/core'

const mcpTools = await connectMCPTools({
  command: 'npx',
  args: ['-y', '@modelcontextprotocol/server-github'],
  env: {
    GITHUB_PERSONAL_ACCESS_TOKEN: process.env.GITHUB_TOKEN,
  },
})

const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
  customTools: mcpTools,
})

const result = await orchestrator.runAgent({
  name: 'github-agent',
  tools: ['create_or_update_file', 'search_repositories'], // MCP tools
}, 'Search for TypeScript agent frameworks and create a comparison in repo.md')
```

### Agent Delegation Tool

Allow agents to delegate to other agents:

```typescript
const team = orchestrator.createTeam('dev-team', {
  name: 'dev-team',
  agents: [
    {
      name: 'lead',
      systemPrompt: 'You coordinate work.',
      tools: ['delegate_to_agent'],
    },
    {
      name: 'specialist',
      systemPrompt: 'You implement features.',
      tools: ['bash', 'file_write'],
    },
  ],
})

// The 'lead' agent can now call delegate_to_agent to hand off tasks
```

## Structured Output

Get Zod-validated responses:

```typescript
import { z } from 'zod'

const resultSchema = z.object({
  files: z.array(z.object({
    path: z.string(),
    purpose: z.string(),
  })),
  commands: z.array(z.string()),
  summary: z.string(),
})

const result = await orchestrator.runAgent({
  name: 'architect',
  systemPrompt: 'You design project structures.',
}, 'Design a TypeScript library structure', {
  resultSchema,
})

// result.parsedContent is now typed and validated
console.log(result.parsedContent.files) // TypeScript knows the shape
console.log(result.parsedContent.commands)
```

## Shared Memory

### In-Memory (Default)

```typescript
const team = orchestrator.createTeam('team', {
  name: 'team',
  agents: [...],
  sharedMemory: true, // default in-memory store
})
```

### Custom Memory Store (Redis)

```typescript
import { MemoryStore } from '@open-multi-agent/core'
import Redis from 'ioredis'

class RedisMemoryStore implements MemoryStore {
  private redis: Redis

  constructor() {
    this.redis = new Redis(process.env.REDIS_URL)
  }

  async get(key: string): Promise<string | null> {
    return this.redis.get(key)
  }

  async set(key: string, value: string): Promise<void> {
    await this.redis.set(key, value)
  }

  async delete(key: string): Promise<void> {
    await this.redis.del(key)
  }

  async clear(): Promise<void> {
    await this.redis.flushdb()
  }
}

const team = orchestrator.createTeam('team', {
  name: 'team',
  agents: [...],
  sharedMemory: true,
  memoryStore: new RedisMemoryStore(),
})
```

## Observability

### Progress Events

```typescript
const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
  onProgress: (event) => {
    switch (event.type) {
      case 'agent_start':
        console.log(`Starting agent: ${event.agent}`)
        break
      case 'task_start':
        console.log(`Task started: ${event.task}`)
        break
      case 'task_complete':
        console.log(`Task complete: ${event.task}`)
        break
      case 'agent_complete':
        console.log(`Agent finished: ${event.agent}`)
        break
    }
  },
})
```

### Trace Observability

```typescript
const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
  onTrace: (span) => {
    console.log('Span:', span.type, span.name)
    console.log('Duration:', span.endTime - span.startTime, 'ms')
    if (span.metadata?.tokens) {
      console.log('Tokens:', span.metadata.tokens)
    }
  },
})
```

### Post-Run Dashboard

Generate HTML report of executed task DAG:

```typescript
const result = await orchestrator.runTeam(team, goal)

// result.trace contains all execution data
// Render to HTML (implementation in docs/observability.md)
```

## Plan-Only Mode

Preview the task DAG without executing:

```typescript
const plan = await orchestrator.runTeam(team, goal, { planOnly: true })

console.log('Tasks:', plan.tasks.map(t => ({
  id: t.id,
  description: t.description,
  assignedTo: t.assignedTo,
  dependencies: t.dependencies,
})))
```

## Fan-Out / MapReduce Pattern

Parallel execution without dependencies:

```typescript
import { AgentPool } from '@open-multi-agent/core'

const pool = new AgentPool({
  defaultModel: 'claude-sonnet-4-6',
  agents: [
    { name: 'analyzer-1', systemPrompt: 'Analyze data source 1.' },
    { name: 'analyzer-2', systemPrompt: 'Analyze data source 2.' },
    { name: 'analyzer-3', systemPrompt: 'Analyze data source 3.' },
  ],
})

const results = await pool.runParallel([
  { agent: 'analyzer-1', prompt: 'Analyze feed A' },
  { agent: 'analyzer-2', prompt: 'Analyze feed B' },
  { agent: 'analyzer-3', prompt: 'Analyze feed C' },
])

// Aggregate results
const aggregator = new OpenMultiAgent({ defaultModel: 'claude-sonnet-4-6' })
const summary = await aggregator.runAgent({
  name: 'aggregator',
  systemPrompt: 'Merge analysis results.',
}, `Merge these findings:\n${results.map(r => r.content).join('\n\n')}`)
```

## Production Checklist

### Context Management

```typescript
const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
  maxContextTokens: 100000, // prevent context overflow
  contextStrategy: 'truncate', // or 'summarize'
})
```

### Task Retry with Backoff

```typescript
const tasks: TaskConfig[] = [
  {
    id: 'api-call',
    description: 'Fetch external data',
    assignedTo: 'fetcher',
    dependencies: [],
    retry: {
      maxAttempts: 3,
      backoffMs: 1000, // exponential backoff
    },
  },
]
```

### Loop Detection

```typescript
const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
  maxIterations: 20, // prevent infinite loops
})
```

### Tool Output Truncation

```typescript
import { defineTool } from '@open-multi-agent/core'
import { z } from 'zod'

const largeTool = defineTool({
  name: 'fetch_logs',
  description: 'Fetch system logs',
  parameters: z.object({ lines: z.number() }),
  execute: async ({ lines }) => {
    const logs = getLogs(lines) // potentially huge
    return logs.slice(0, 10000) // truncate to 10k chars
  },
})
```

## Common Patterns

### Contract Review Workflow

Four-task DAG with parallel branches:

```typescript
const tasks: TaskConfig[] = [
  {
    id: 'extract',
    description: 'Extract key terms from contract',
    assignedTo: 'extractor',
    dependencies: [],
  },
  {
    id: 'risk-check',
    description: 'Identify legal risks',
    assignedTo: 'legal-expert',
    dependencies: ['extract'],
  },
  {
    id: 'compliance-check',
    description: 'Check regulatory compliance',
    assignedTo: 'compliance-expert',
    dependencies: ['extract'],
  },
  {
    id: 'report',
    description: 'Generate final report',
    assignedTo: 'reporter',
    dependencies: ['risk-check', 'compliance-check'],
  },
]

const result = await orchestrator.runTasks(team, tasks)
```

### Translation with Back-Translation

```typescript
const team = orchestrator.createTeam('translation', {
  name: 'translation',
  agents: [
    {
      name: 'translator',
      model: 'claude-sonnet-4-6',
      systemPrompt: 'Translate English to target language.',
    },
    {
      name: 'back-translator',
      model: 'gpt-4o',
      systemPrompt: 'Translate back to English.',
    },
    {
      name: 'validator',
      model: 'claude-sonnet-4-6',
      systemPrompt: 'Compare original and back-translation, flag drift.',
    },
  ],
  sharedMemory: true,
})

const tasks: TaskConfig[] = [
  { id: 'translate', description: 'Translate to Spanish', assignedTo: 'translator', dependencies: [] },
  { id: 'back-translate', description: 'Translate back to English', assignedTo: 'back-translator', dependencies: ['translate'] },
  { id: 'validate', description: 'Compare and flag drift', assignedTo: 'validator', dependencies: ['translate', 'back-translate'] },
]

const result = await orchestrator.runTasks(team, tasks)
```

## Troubleshooting

### "Model not found" Error

Make sure the correct API key is set:

```bash
# Check which provider your model uses
export ANTHROPIC_API_KEY=sk-ant-...  # for claude-*
export OPENAI_API_KEY=sk-...         # for gpt-*
export GEMINI_API_KEY=...            # for gemini-*
```

### Ollama Connection Refused

```bash
# Start Ollama daemon
ollama serve

# Pull the model
ollama pull qwen2.5-coder:7b

# Verify it's running
curl http://localhost:11434/api/tags
```

### Task Hangs / Times Out

Enable progress logging to see where it's stuck:

```typescript
const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6',
  onProgress: (event) => console.log(event), // debug all events
  timeout: 300000, // 5 minutes per task
})
```

### "Too many tokens" Error

Reduce context or use a larger context window model:

```typescript
const orchestrator = new OpenMultiAgent({
  defaultModel: 'claude-sonnet-4-6', // 200k context
  maxContextTokens: 100000,
  contextStrategy: 'truncate', // keep recent messages only
})
```

### MCP Server Won't Connect

Check stderr from the MCP process:

```typescript
const mcpTools = await connectMCPTools({
  command: 'npx',
  args: ['-y', '@modelcontextprotocol/server-github'],
  env: { GITHUB_PERSONAL_ACCESS_TOKEN: process.env.GITHUB_TOKEN },
  onStderr: (data) => console.error('MCP stderr:', data),
})
```

### Agents Not Sharing Context

Ensure `sharedMemory: true` is set:

```typescript
const team = orchestrator.createTeam('team', {
  name: 'team',
  agents: [...],
  sharedMemory: true, // required for context sharing
})
```

## CLI Usage

For shell and CI workflows:

```bash
# Install globally
npm install -g @open-multi-agent/core

# Run a team
oma run --team api-team --goal "Create REST API" --config team.json

# Plan only
oma plan --team api-team --goal "Create REST API" --config team.json

# Output JSON for parsing
oma run --team api-team --goal "..." --json > result.json
```

See [docs/cli.md](https://github.com/open-multi-agent/open-multi-agent/blob/main/docs/cli.md) for full CLI reference.

## Resources

- **Documentation**: https://open-multi-agent.com
- **GitHub**: https://github.com/open-multi-agent/open-multi-agent
- **Examples**: https://github.com/open-multi-agent/open-multi-agent/tree/main/examples
- **Provider Setup**: https://github.com/open-multi-agent/open-multi-agent/blob/main/docs/providers.md
- **Tool Configuration**: https://github.com/open-multi-agent/open-multi-agent/blob/main/docs/tool-configuration.md
- **Observability Guide**: https://github.com/open-multi-agent/open-multi-agent/blob/main/docs/observability.md
