---
name: litellm-agent-platform
description: Self-hosted platform for running coding agents (Claude Code, Codex, Hermes) in isolated sandboxes with vault proxy and session management
triggers:
  - how do I deploy a coding agent with LiteLLM Agent Platform
  - set up an agent harness with Claude Code or Codex
  - create a scheduled agent with CRON in LiteLLM
  - manage agent sessions and memory across runs
  - configure MCP tools for my agent platform
  - run agents through the LiteLLM API
  - connect GitHub and AWS tools to my agent
  - swap between different agent harnesses
---

# LiteLLM Agent Platform

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

LiteLLM Agent Platform is a self-hosted service for running coding agents (Claude Code, Codex, Hermes, OpenCode) with persistent sessions, CRON scheduling, memory management, and unified tool access through MCP servers. It provides both a UI and REST API for creating, deploying, and managing agents with isolated sandbox execution.

## Installation

### Prerequisites

- Node.js 18+ / TypeScript environment
- Docker (for sandbox isolation)
- LiteLLM API key

### Self-Hosted Setup

```bash
# Clone the repository
git clone https://github.com/LiteLLM-Labs/litellm-agent-platform.git
cd litellm-agent-platform

# Install dependencies
npm install

# Configure environment
cp .env.example .env
```

### Environment Configuration

```bash
# .env
LITELLM_MASTER_KEY=your_master_key_here
DATABASE_URL=postgresql://user:password@localhost:5432/agent_platform
REDIS_URL=redis://localhost:6379
PORT=4000

# Model provider keys (stored in vault)
ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY
OPENAI_API_KEY=$OPENAI_API_KEY
```

### Start the Platform

```bash
# Development
npm run dev

# Production
npm run build
npm start
```

The platform will be available at `http://localhost:4000`.

## Core Concepts

### Harnesses

Harnesses are the underlying agent frameworks:
- **Claude Code**: Anthropic's coding agent
- **Codex**: OpenAI's code generation agent
- **Hermes**: Open-source coding assistant
- **OpenCode**: Community-driven agent

### Tools

Tools are MCP (Model Context Protocol) servers that provide capabilities:
- GitHub integration
- AWS services
- File system access
- Database connections
- Custom MCP servers

### Sessions

Persistent agent sessions maintain context across multiple runs, enabling long-running workflows and memory retention.

## API Usage

### Authentication

All API requests require a Bearer token:

```bash
export LITELLM_KEY="your_api_key"
```

### Create an Agent

```typescript
// TypeScript example
import axios from 'axios';

const createAgent = async () => {
  const response = await axios.post(
    'http://localhost:4000/agents',
    {
      name: 'ci-fixer',
      harness: 'claude-code',
      model: 'anthropic/claude-sonnet-4-5',
      system_prompt: 'You monitor CI and fix failing checks.',
      tools: ['github', 'aws'],
      memory_enabled: true,
      session_timeout: 3600
    },
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data;
};
```

```bash
# cURL example
curl http://localhost:4000/agents -X POST \
  -H "Authorization: Bearer $LITELLM_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "code-reviewer",
    "harness": "codex",
    "model": "openai/gpt-4",
    "system_prompt": "Review code for security and best practices.",
    "tools": ["github", "filesystem"]
  }'
```

### Run an Agent

```typescript
// Single run
const runAgent = async (agentName: string, input: string) => {
  const response = await axios.post(
    `http://localhost:4000/agents/${agentName}/runs`,
    { input },
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data;
};

// Usage
const result = await runAgent('ci-fixer', 'Fix the failing CI check on PR #418');
console.log(result.output);
```

```bash
# cURL example
curl http://localhost:4000/agents/ci-fixer/runs -X POST \
  -H "Authorization: Bearer $LITELLM_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "input": "Fix the failing CI check on PR #418" }'
```

### List Agents

```typescript
const listAgents = async () => {
  const response = await axios.get('http://localhost:4000/agents', {
    headers: {
      Authorization: `Bearer ${process.env.LITELLM_KEY}`
    }
  });
  
  return response.data.agents;
};
```

### Get Agent Status

```typescript
const getAgentStatus = async (agentName: string) => {
  const response = await axios.get(
    `http://localhost:4000/agents/${agentName}`,
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data;
};
```

### Update Agent Configuration

```typescript
const updateAgent = async (agentName: string, updates: any) => {
  const response = await axios.patch(
    `http://localhost:4000/agents/${agentName}`,
    updates,
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data;
};

// Example: Switch harness
await updateAgent('ci-fixer', {
  harness: 'hermes',
  model: 'nous/hermes-2-pro'
});
```

### Delete an Agent

```typescript
const deleteAgent = async (agentName: string) => {
  await axios.delete(`http://localhost:4000/agents/${agentName}`, {
    headers: {
      Authorization: `Bearer ${process.env.LITELLM_KEY}`
    }
  });
};
```

## Session Management

### Create a Persistent Session

```typescript
const createSession = async (agentName: string) => {
  const response = await axios.post(
    `http://localhost:4000/agents/${agentName}/sessions`,
    {
      session_id: 'debug-session-001',
      metadata: {
        project: 'api-service',
        branch: 'main'
      }
    },
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data.session_id;
};
```

### Run with Session Context

```typescript
const runWithSession = async (
  agentName: string,
  sessionId: string,
  input: string
) => {
  const response = await axios.post(
    `http://localhost:4000/agents/${agentName}/runs`,
    {
      input,
      session_id: sessionId
    },
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data;
};

// Multi-turn conversation with memory
const sessionId = await createSession('code-reviewer');
await runWithSession(sessionId, 'code-reviewer', 'Review src/auth.ts');
await runWithSession(sessionId, 'code-reviewer', 'Now check for SQL injection risks');
```

## CRON Scheduling

### Schedule an Agent

```typescript
const scheduleAgent = async (agentName: string, schedule: string, task: string) => {
  const response = await axios.post(
    `http://localhost:4000/agents/${agentName}/schedules`,
    {
      cron: schedule,
      input: task,
      enabled: true
    },
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data;
};

// Run every day at 9 AM
await scheduleAgent(
  'ci-fixer',
  '0 9 * * *',
  'Check all open PRs for failing CI and fix them'
);

// Run every hour
await scheduleAgent(
  'security-scanner',
  '0 * * * *',
  'Scan dependencies for vulnerabilities'
);
```

### List Schedules

```typescript
const listSchedules = async (agentName: string) => {
  const response = await axios.get(
    `http://localhost:4000/agents/${agentName}/schedules`,
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data.schedules;
};
```

## Tool Configuration

### Register MCP Tools

```typescript
const registerTool = async (toolConfig: any) => {
  const response = await axios.post(
    'http://localhost:4000/tools',
    toolConfig,
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data;
};

// Register GitHub MCP
await registerTool({
  name: 'github',
  type: 'mcp',
  config: {
    server_url: 'https://github-mcp.example.com',
    credentials: {
      token: process.env.GITHUB_TOKEN
    }
  }
});

// Register custom MCP
await registerTool({
  name: 'custom-api',
  type: 'mcp',
  config: {
    server_url: 'http://localhost:8080/mcp',
    credentials: {
      api_key: process.env.CUSTOM_API_KEY
    }
  }
});
```

### List Available Tools

```typescript
const listTools = async () => {
  const response = await axios.get('http://localhost:4000/tools', {
    headers: {
      Authorization: `Bearer ${process.env.LITELLM_KEY}`
    }
  });
  
  return response.data.tools;
};
```

## Common Patterns

### CI/CD Integration Agent

```typescript
const setupCIAgent = async () => {
  // Create the agent
  const agent = await axios.post(
    'http://localhost:4000/agents',
    {
      name: 'ci-cd-helper',
      harness: 'claude-code',
      model: 'anthropic/claude-sonnet-4-5',
      system_prompt: `You are a CI/CD expert. Monitor build failures, 
        analyze logs, and suggest fixes. You have access to GitHub and AWS.`,
      tools: ['github', 'aws'],
      memory_enabled: true
    },
    {
      headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
    }
  );
  
  // Schedule daily checks
  await axios.post(
    `http://localhost:4000/agents/ci-cd-helper/schedules`,
    {
      cron: '0 */6 * * *', // Every 6 hours
      input: 'Check recent CI failures and create tickets for recurring issues',
      enabled: true
    },
    {
      headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
    }
  );
  
  return agent.data;
};
```

### Code Review Automation

```typescript
const setupCodeReviewer = async () => {
  const agent = await axios.post(
    'http://localhost:4000/agents',
    {
      name: 'security-reviewer',
      harness: 'codex',
      model: 'openai/gpt-4',
      system_prompt: `Review code for security vulnerabilities, 
        best practices, and performance issues. Be thorough but constructive.`,
      tools: ['github', 'filesystem'],
      memory_enabled: true
    },
    {
      headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
    }
  );
  
  // Trigger on new PRs via webhook or schedule
  return agent.data;
};
```

### Multi-Step Workflow with Sessions

```typescript
const runMultiStepWorkflow = async () => {
  const agentName = 'code-refactor';
  const sessionId = `refactor-${Date.now()}`;
  
  // Step 1: Analyze codebase
  const analysis = await axios.post(
    `http://localhost:4000/agents/${agentName}/runs`,
    {
      input: 'Analyze src/ for code smells and technical debt',
      session_id: sessionId
    },
    {
      headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
    }
  );
  
  // Step 2: Create refactoring plan
  const plan = await axios.post(
    `http://localhost:4000/agents/${agentName}/runs`,
    {
      input: 'Create a detailed refactoring plan based on the analysis',
      session_id: sessionId
    },
    {
      headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
    }
  );
  
  // Step 3: Execute refactoring
  const execution = await axios.post(
    `http://localhost:4000/agents/${agentName}/runs`,
    {
      input: 'Execute the first three refactoring tasks from the plan',
      session_id: sessionId
    },
    {
      headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
    }
  );
  
  return { analysis, plan, execution };
};
```

## Monitoring and Logs

### Get Run History

```typescript
const getRunHistory = async (agentName: string, limit = 10) => {
  const response = await axios.get(
    `http://localhost:4000/agents/${agentName}/runs?limit=${limit}`,
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data.runs;
};
```

### Get Run Details

```typescript
const getRunDetails = async (agentName: string, runId: string) => {
  const response = await axios.get(
    `http://localhost:4000/agents/${agentName}/runs/${runId}`,
    {
      headers: {
        Authorization: `Bearer ${process.env.LITELLM_KEY}`
      }
    }
  );
  
  return response.data;
};
```

## Troubleshooting

### Agent Not Starting

**Issue**: Agent creation returns success but runs fail

**Solution**: Check model availability and credentials:

```typescript
const debugAgent = async (agentName: string) => {
  const status = await axios.get(
    `http://localhost:4000/agents/${agentName}/debug`,
    {
      headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
    }
  );
  
  console.log('Model available:', status.data.model_available);
  console.log('Tools connected:', status.data.tools_status);
  console.log('Last error:', status.data.last_error);
};
```

### Session Memory Issues

**Issue**: Agent not remembering context across runs

**Solution**: Ensure session_id is consistent and memory is enabled:

```typescript
// Verify session configuration
const checkSession = async (agentName: string) => {
  const agent = await axios.get(
    `http://localhost:4000/agents/${agentName}`,
    {
      headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
    }
  );
  
  if (!agent.data.memory_enabled) {
    await axios.patch(
      `http://localhost:4000/agents/${agentName}`,
      { memory_enabled: true },
      {
        headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
      }
    );
  }
};
```

### Tool Connection Failures

**Issue**: Agent cannot access tools (GitHub, AWS, etc.)

**Solution**: Verify tool registration and credentials:

```bash
# Check tool status
curl http://localhost:4000/tools \
  -H "Authorization: Bearer $LITELLM_KEY"

# Test tool connection
curl http://localhost:4000/tools/github/test \
  -H "Authorization: Bearer $LITELLM_KEY"
```

### Rate Limiting

**Issue**: Hitting model provider rate limits

**Solution**: Configure retry logic and backoff:

```typescript
const runWithRetry = async (agentName: string, input: string, maxRetries = 3) => {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await axios.post(
        `http://localhost:4000/agents/${agentName}/runs`,
        { input, retry_on_rate_limit: true },
        {
          headers: { Authorization: `Bearer ${process.env.LITELLM_KEY}` }
        }
      );
      return response.data;
    } catch (error: any) {
      lastError = error;
      if (error.response?.status === 429) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
      } else {
        throw error;
      }
    }
  }
  
  throw lastError;
};
```

## Best Practices

1. **Use sessions for multi-turn interactions**: Enable memory and use consistent session IDs
2. **Set appropriate timeouts**: Configure `session_timeout` based on workflow complexity
3. **Monitor run history**: Track agent performance and failures
4. **Secure credentials**: Store API keys in environment variables, use the vault proxy
5. **Start with Claude Code or Codex**: Most reliable harnesses for production use
6. **Test tools separately**: Verify MCP connections before deploying agents
7. **Use descriptive system prompts**: Clear instructions improve agent behavior
8. **Schedule wisely**: Avoid overlapping CRON jobs for the same agent
