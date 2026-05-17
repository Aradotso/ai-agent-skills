---
name: open-agent-sdk-typescript
description: Build AI agents with in-process agent loops using Anthropic or OpenAI APIs, custom tools, MCP servers, and multi-turn conversations
triggers:
  - create an AI agent with open-agent-sdk
  - set up agent with custom tools and MCP
  - build a conversational agent loop
  - use open-agent-sdk with OpenAI or Anthropic
  - implement agent skills and subagents
  - add lifecycle hooks to my agent
  - create streaming agent queries
  - integrate MCP servers with my agent
---

# Open Agent SDK (TypeScript)

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Open Agent SDK is a TypeScript library that runs full AI agent loops **in-process** without CLI dependencies or subprocesses. It supports both Anthropic and OpenAI-compatible APIs, provides 35+ built-in tools, MCP server integration, custom tool creation, skills system, subagents, and lifecycle hooks. Deploy anywhere: cloud, serverless, Docker, or CI/CD.

## Installation

```bash
npm install @codeany/open-agent-sdk
```

Set your API key as an environment variable:

```bash
export CODEANY_API_KEY=your-api-key-here
```

For OpenAI-compatible models:

```bash
export CODEANY_API_TYPE=openai-completions
export CODEANY_API_KEY=sk-your-key
export CODEANY_BASE_URL=https://api.openai.com/v1
export CODEANY_MODEL=gpt-4o
```

For third-party Anthropic-compatible providers:

```bash
export CODEANY_BASE_URL=https://openrouter.ai/api
export CODEANY_API_KEY=sk-or-your-key
export CODEANY_MODEL=anthropic/claude-sonnet-4
```

## Core Concepts

### API Types

The SDK supports two API types:
- **`anthropic-messages`**: Anthropic Claude models (default)
- **`openai-completions`**: OpenAI, DeepSeek, Qwen, Mistral, or any OpenAI-compatible API

API type is auto-detected from model name. Models containing `gpt-`, `o1`, `o3`, `deepseek`, `qwen`, or `mistral` automatically use `openai-completions`.

### Agent vs Query

- **`query()`**: One-shot streaming query, no session persistence
- **`createAgent()`**: Reusable agent with multi-turn conversation history

## Quick Start Examples

### One-Shot Streaming Query

```typescript
import { query } from "@codeany/open-agent-sdk";

for await (const message of query({
  prompt: "Read package.json and tell me the project name.",
  options: {
    allowedTools: ["Read", "Glob"],
    permissionMode: "bypassPermissions",
  },
})) {
  if (message.type === "assistant") {
    for (const block of message.message.content) {
      if ("text" in block) {
        console.log(block.text);
      }
    }
  }
  
  if (message.type === "result") {
    console.log(`Cost: $${message.total_cost_usd?.toFixed(4)}`);
    console.log(`Turns: ${message.num_turns}`);
  }
}
```

### Simple Blocking Prompt

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent({ model: "claude-sonnet-4-6" });
const result = await agent.prompt("What files are in this project?");

console.log(result.text);
console.log(`Turns: ${result.num_turns}`);
console.log(`Tokens: ${result.usage.input_tokens + result.usage.output_tokens}`);
```

### OpenAI / GPT Models

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent({
  apiType: "openai-completions",
  model: "gpt-4o",
  apiKey: process.env.OPENAI_API_KEY,
  baseURL: "https://api.openai.com/v1",
});

const result = await agent.prompt("Analyze the code structure of this project");
console.log(result.text);
```

### Multi-Turn Conversation

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent({ maxTurns: 5 });

const r1 = await agent.prompt(
  'Create a file /tmp/hello.txt with "Hello World"'
);
console.log("Response 1:", r1.text);

const r2 = await agent.prompt("Read back the file you just created");
console.log("Response 2:", r2.text);

const r3 = await agent.prompt("Delete the file");
console.log("Response 3:", r3.text);

console.log(`Total messages in session: ${agent.getMessages().length}`);

// Clean up
await agent.close();
```

## Custom Tools

### Using Zod Schema (Recommended)

```typescript
import { z } from "zod";
import { query, tool, createSdkMcpServer } from "@codeany/open-agent-sdk";

const getWeather = tool(
  "get_weather",
  "Get the current weather for a city",
  {
    city: z.string().describe("City name"),
    unit: z.enum(["celsius", "fahrenheit"]).optional().describe("Temperature unit"),
  },
  async ({ city, unit = "celsius" }) => {
    // Simulate API call
    const temp = unit === "celsius" ? 22 : 72;
    return {
      content: [
        {
          type: "text",
          text: `Weather in ${city}: ${temp}°${unit === "celsius" ? "C" : "F"}, sunny`,
        },
      ],
    };
  }
);

const calculator = tool(
  "calculate",
  "Perform mathematical calculations",
  {
    expression: z.string().describe("Mathematical expression to evaluate"),
  },
  async ({ expression }) => {
    try {
      const result = Function(`'use strict'; return (${expression})`)();
      return {
        content: [{ type: "text", text: `${expression} = ${result}` }],
      };
    } catch (error) {
      return {
        content: [{ type: "text", text: `Error: ${error.message}` }],
        isError: true,
      };
    }
  }
);

const server = createSdkMcpServer({
  name: "custom-tools",
  tools: [getWeather, calculator],
});

for await (const msg of query({
  prompt: "What's the weather in Tokyo? Also calculate 2**16.",
  options: { mcpServers: { "custom-tools": server } },
})) {
  if (msg.type === "assistant") {
    for (const block of msg.message.content) {
      if ("text" in block) console.log(block.text);
    }
  }
}
```

### Low-Level Tool Definition

```typescript
import { createAgent, getAllBaseTools, defineTool } from "@codeany/open-agent-sdk";

const databaseQuery = defineTool({
  name: "DatabaseQuery",
  description: "Execute SQL queries against the database",
  inputSchema: {
    type: "object",
    properties: {
      query: {
        type: "string",
        description: "SQL query to execute",
      },
      readonly: {
        type: "boolean",
        description: "Whether this is a read-only query",
      },
    },
    required: ["query"],
  },
  isReadOnly: false,
  async call(input) {
    // Validate read-only
    if (!input.readonly && /^\s*(SELECT|SHOW|DESCRIBE)/i.test(input.query)) {
      return "Error: Use readonly=true for SELECT queries";
    }
    
    // Execute query (simulated)
    return JSON.stringify({
      rows: [{ id: 1, name: "Example" }],
      rowCount: 1,
    });
  },
});

const agent = createAgent({
  tools: [...getAllBaseTools(), databaseQuery],
});

const result = await agent.prompt(
  "Query the users table and show me all records"
);
console.log(result.text);
```

## Built-in Tools

The SDK includes 35+ built-in tools. Common ones include:

- **File operations**: `Read`, `Write`, `Edit`, `Delete`, `Move`, `Rename`
- **Search**: `Glob`, `Grep`, `Search`
- **Git**: `GitStatus`, `GitDiff`, `GitLog`, `GitCommit`, `GitCheckout`
- **Analysis**: `Lint`, `Symbols`, `CodeGraph`, `Dependencies`
- **Shell**: `Bash`, `BashSession`
- **Utility**: `Ask`, `Attempt`, `Skill`, `Agent` (subagents)

### Restricting Tools

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

// Read-only agent
const readOnlyAgent = createAgent({
  allowedTools: ["Read", "Glob", "Grep", "Search"],
  permissionMode: "dontAsk",
});

// Agent without shell access
const noShellAgent = createAgent({
  disallowedTools: ["Bash", "BashSession"],
});

// Minimal tool set
const minimalAgent = createAgent({
  tools: [], // No tools at all
});
```

## Skills

Skills are reusable prompt templates. Five built-in skills: `simplify`, `commit`, `review`, `debug`, `test`.

### Using Built-in Skills

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent();

// The agent can invoke skills via the Skill tool
const result = await agent.prompt(
  'Use the "review" skill to review src/index.ts'
);
console.log(result.text);
```

### Creating Custom Skills

```typescript
import { registerSkill, getAllSkills, createAgent } from "@codeany/open-agent-sdk";

registerSkill({
  name: "explain",
  description: "Explain a concept in simple terms",
  userInvocable: true,
  async getPrompt(args) {
    return [
      {
        type: "text",
        text: `Explain in simple terms: ${args || "Ask what to explain."}`,
      },
    ];
  },
});

registerSkill({
  name: "optimize",
  description: "Optimize code for performance",
  userInvocable: true,
  async getPrompt(args) {
    return [
      {
        type: "text",
        text: `Analyze and optimize the following for performance:\n${args}\n\nProvide specific improvements and benchmarks.`,
      },
    ];
  },
});

console.log(`${getAllSkills().length} skills registered`);

const agent = createAgent();
const result = await agent.prompt(
  'Use the "optimize" skill on src/heavy-computation.ts'
);
console.log(result.text);
```

## MCP Server Integration

### External MCP Servers

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent({
  mcpServers: {
    filesystem: {
      command: "npx",
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
    },
    postgres: {
      command: "npx",
      args: ["-y", "@modelcontextprotocol/server-postgres"],
      env: {
        DATABASE_URL: process.env.DATABASE_URL,
      },
    },
  },
});

const result = await agent.prompt("List files in /tmp and query the database");
console.log(result.text);

await agent.close(); // Important: closes MCP connections
```

### In-Process MCP Servers

```typescript
import { z } from "zod";
import { createAgent, tool, createSdkMcpServer } from "@codeany/open-agent-sdk";

const httpGet = tool(
  "http_get",
  "Make HTTP GET requests",
  {
    url: z.string().url().describe("URL to fetch"),
    headers: z.record(z.string()).optional().describe("HTTP headers"),
  },
  async ({ url, headers }) => {
    const response = await fetch(url, { headers });
    const text = await response.text();
    return {
      content: [
        {
          type: "text",
          text: `Status: ${response.status}\n\n${text}`,
        },
      ],
    };
  }
);

const mcpServer = createSdkMcpServer({
  name: "http-tools",
  tools: [httpGet],
});

const agent = createAgent({
  mcpServers: {
    http: mcpServer,
  },
});

const result = await agent.prompt("Fetch https://api.github.com/repos/codeany-ai/open-agent-sdk-typescript");
console.log(result.text);
```

## Subagents

Subagents are specialized agents with restricted tools and custom prompts.

```typescript
import { query } from "@codeany/open-agent-sdk";

for await (const msg of query({
  prompt: "Use the security-reviewer to audit src/auth.ts for vulnerabilities",
  options: {
    agents: {
      "security-reviewer": {
        description: "Expert security code reviewer",
        prompt: "You are a security expert. Analyze code for vulnerabilities, focusing on authentication, authorization, input validation, and data exposure.",
        tools: ["Read", "Glob", "Grep", "Symbols"],
      },
      "performance-optimizer": {
        description: "Performance optimization specialist",
        prompt: "You optimize code for speed and efficiency. Identify bottlenecks, suggest algorithmic improvements, and recommend caching strategies.",
        tools: ["Read", "Symbols", "Dependencies"],
      },
    },
  },
})) {
  if (msg.type === "assistant") {
    for (const block of msg.message.content) {
      if ("text" in block) console.log(block.text);
    }
  }
}
```

## Lifecycle Hooks

20 available hook events for observability and control.

```typescript
import { createAgent, createHookRegistry } from "@codeany/open-agent-sdk";

const hooks = createHookRegistry({
  PreToolUse: [
    {
      handler: async (input) => {
        console.log(`[HOOK] About to use tool: ${input.toolName}`);
        console.log(`[HOOK] Arguments:`, input.toolInput);
        
        // Block dangerous operations
        if (input.toolName === "Bash" && input.toolInput.command?.includes("rm -rf")) {
          console.warn("[HOOK] Blocking dangerous command");
          return { block: true };
        }
      },
    },
  ],
  PostToolUse: [
    {
      handler: async (input) => {
        console.log(`[HOOK] Tool ${input.toolName} completed`);
        console.log(`[HOOK] Result length:`, input.toolResult.length);
      },
    },
  ],
  PostToolUseFailure: [
    {
      handler: async (input) => {
        console.error(`[HOOK] Tool ${input.toolName} failed:`, input.error);
      },
    },
  ],
  SessionStart: [
    {
      handler: async () => {
        console.log("[HOOK] Session started");
      },
    },
  ],
  SessionEnd: [
    {
      handler: async () => {
        console.log("[HOOK] Session ended");
      },
    },
  ],
  FileChanged: [
    {
      handler: async (input) => {
        console.log(`[HOOK] File changed: ${input.path}`);
      },
    },
  ],
});

const agent = createAgent({
  hooks,
  model: "claude-sonnet-4-6",
});

const result = await agent.prompt("List files and create a new test file");
console.log(result.text);
```

### Available Hook Events

- **Tool lifecycle**: `PreToolUse`, `PostToolUse`, `PostToolUseFailure`
- **Session**: `SessionStart`, `SessionEnd`, `Stop`
- **Subagents**: `SubagentStart`, `SubagentStop`
- **User interaction**: `UserPromptSubmit`, `PermissionRequest`, `PermissionDenied`
- **Tasks**: `TaskCreated`, `TaskCompleted`
- **Config**: `ConfigChange`, `CwdChanged`
- **File system**: `FileChanged`
- **System**: `Notification`, `PreCompact`, `PostCompact`, `TeammateIdle`

## Permission Modes

Control how the agent handles permission requests:

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

// Bypass all permissions (default)
const autoAgent = createAgent({
  permissionMode: "bypassPermissions",
});

// Never ask, deny if permission needed
const denyAgent = createAgent({
  permissionMode: "dontAsk",
});

// Ask for permission (requires custom handler)
const askAgent = createAgent({
  permissionMode: "ask",
  // Implement permission handler via hooks
});

// Change mode mid-session
autoAgent.setPermissionMode("dontAsk");
```

## Configuration Options

### Agent Options

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent({
  // API configuration
  apiType: "anthropic-messages", // or "openai-completions"
  model: "claude-sonnet-4-6",
  apiKey: process.env.CODEANY_API_KEY,
  baseURL: "https://api.anthropic.com",
  
  // Session
  sessionId: "my-session-id",
  maxTurns: 10,
  
  // Working directory
  cwd: process.cwd(),
  
  // System prompt
  systemPrompt: "You are a helpful coding assistant",
  appendSystemPrompt: "\n\nAlways explain your reasoning.",
  
  // Tools
  tools: [], // Custom tool list
  allowedTools: ["Read", "Write", "Glob"],
  disallowedTools: ["Bash"],
  
  // Permissions
  permissionMode: "bypassPermissions",
  
  // MCP servers
  mcpServers: {},
  
  // Subagents
  agents: {},
  
  // Hooks
  hooks: createHookRegistry({}),
  
  // Model parameters
  temperature: 0.7,
  maxTokens: 4096,
});
```

### Query Options

```typescript
import { query } from "@codeany/open-agent-sdk";

for await (const msg of query({
  prompt: "Your prompt here",
  options: {
    model: "gpt-4o",
    apiType: "openai-completions",
    apiKey: process.env.OPENAI_API_KEY,
    cwd: "/path/to/project",
    allowedTools: ["Read", "Glob"],
    permissionMode: "bypassPermissions",
    maxTurns: 5,
    temperature: 0.5,
  },
})) {
  // Handle messages
}
```

## Advanced Patterns

### Session Management

```typescript
import { createAgent, listSessions, forkSession } from "@codeany/open-agent-sdk";

// Create agent with persistent session
const agent = createAgent({
  sessionId: "my-project-session",
});

await agent.prompt("Analyze the codebase structure");
await agent.prompt("Now suggest improvements");

// List all sessions
const sessions = await listSessions();
console.log("Saved sessions:", sessions);

// Fork a session for experimentation
const forkedId = await forkSession("my-project-session");
const forkedAgent = createAgent({ sessionId: forkedId });

await forkedAgent.prompt("Try refactoring with a different approach");
```

### Interrupting Long-Running Queries

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent();

// Start a long-running query
const queryPromise = agent.prompt("Analyze all files in the project");

// Interrupt after 5 seconds
setTimeout(() => {
  agent.interrupt();
  console.log("Query interrupted");
}, 5000);

try {
  const result = await queryPromise;
  console.log(result.text);
} catch (error) {
  console.log("Query was interrupted");
}
```

### Model Switching

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent({ model: "claude-sonnet-4-6" });

await agent.prompt("Quick analysis");

// Switch to more powerful model for complex task
agent.setModel("claude-opus-4");
await agent.prompt("Perform deep architectural analysis");

// Switch back to cheaper model
agent.setModel("claude-sonnet-4-6");
await agent.prompt("Summarize findings");
```

### Streaming with Full Control

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent();

for await (const message of agent.query("Analyze this codebase")) {
  switch (message.type) {
    case "assistant":
      for (const block of message.message.content) {
        if ("text" in block) {
          process.stdout.write(block.text);
        } else if ("tool_use" in block) {
          console.log(`\n[Tool: ${block.tool_use.name}]`);
        }
      }
      break;
      
    case "tool_use":
      console.log(`\nExecuting: ${message.toolName}`);
      break;
      
    case "tool_result":
      console.log(`Tool result length: ${message.result.length} chars`);
      break;
      
    case "result":
      console.log(`\n\nCompleted in ${message.num_turns} turns`);
      console.log(`Cost: $${message.total_cost_usd?.toFixed(4)}`);
      console.log(`Tokens: ${message.usage.input_tokens + message.usage.output_tokens}`);
      break;
      
    case "error":
      console.error(`Error: ${message.error}`);
      break;
  }
}
```

## Error Handling

```typescript
import { createAgent } from "@codeany/open-agent-sdk";

const agent = createAgent();

try {
  const result = await agent.prompt("Your prompt here");
  console.log(result.text);
} catch (error) {
  if (error.message.includes("rate limit")) {
    console.error("Rate limit exceeded, retry later");
  } else if (error.message.includes("context length")) {
    console.error("Prompt too long, try simplifying");
  } else {
    console.error("Agent error:", error.message);
  }
} finally {
  await agent.close();
}
```

## Troubleshooting

### API Key Not Found

```
Error: API key not found
```

**Solution**: Set the `CODEANY_API_KEY` environment variable or pass `apiKey` in options:

```typescript
const agent = createAgent({
  apiKey: process.env.CODEANY_API_KEY,
});
```

### Model Not Found / Invalid API Type

```
Error: Model not found: gpt-4o
```

**Solution**: Explicitly set `apiType` for OpenAI models:

```typescript
const agent = createAgent({
  apiType: "openai-completions",
  model: "gpt-4o",
  baseURL: "https://api.openai.com/v1",
});
```

### MCP Server Connection Failed

```
Error: Failed to start MCP server
```

**Solution**: Ensure MCP server package is available and command is correct:

```bash
npm install -g @modelcontextprotocol/server-filesystem
```

```typescript
const agent = createAgent({
  mcpServers: {
    filesystem: {
      command: "npx",
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
    },
  },
});
```

### Permission Denied Errors

```
Permission denied: Cannot execute Bash
```

**Solution**: Adjust permission mode or allow specific tools:

```typescript
const agent = createAgent({
  permissionMode: "bypassPermissions",
  allowedTools: ["Bash", "Read", "Write"],
});
```

### Session Persistence Issues

Sessions are stored in `.codeany/sessions/` by default. Ensure write permissions exist.

### Rate Limiting

Implement retry logic with exponential backoff:

```typescript
async function queryWithRetry(agent, prompt, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await agent.prompt(prompt);
    } catch (error) {
      if (error.message.includes("rate limit") && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

## Web UI

The SDK includes a built-in web chat interface for testing:

```bash
npx tsx examples/web/server.ts
```

Then open `http://localhost:8081` in your browser.

## Complete Example: Code Review Agent

```typescript
import { z } from "zod";
import { createAgent, tool, createSdkMcpServer, createHookRegistry } from "@codeany/open-agent-sdk";

// Custom tool for static analysis
const analyzeComplexity = tool(
  "analyze_complexity",
  "Analyze code complexity metrics",
  {
    filePath: z.string().describe("Path to file to analyze"),
  },
  async ({ filePath }) => {
    // Simplified complexity analysis
    return {
      content: [
        {
          type: "text",
          text: JSON.stringify({
            cyclomaticComplexity: 12,
            cognitiveComplexity: 8,
            linesOfCode: 245,
          }),
        },
      ],
    };
  }
);

const mcpServer = createSdkMcpServer({
  name: "code-analysis",
  tools: [analyzeComplexity],
});

// Hook to log all tool usage
const hooks = createHookRegistry({
  PreToolUse: [
    {
      handler: async (input) => {
        console.log(`[Review] Using tool: ${input.toolName}`);
      },
    },
  ],
  PostToolUse: [
    {
      handler: async (input) => {
        console.log(`[Review] Tool ${input.toolName} completed`);
      },
    },
  ],
});

const agent = createAgent({
  model: "claude-sonnet-4-6",
  systemPrompt: "You are an expert code reviewer. Focus on code quality, security, performance, and maintainability.",
  allowedTools: ["Read", "Glob", "Grep", "Symbols", "analyze_complexity"],
  mcpServers: { analysis: mcpServer },
  hooks,
  permissionMode: "bypassPermissions",
});

async function reviewCode(pattern) {
  console.log(`\n=== Starting code review for: ${pattern} ===\n`);
  
  const result = await agent.prompt(`
Review all files matching ${pattern}. For each file:
1. Analyze code quality and structure
2. Check for security vulnerabilities
3. Assess performance implications
4. Suggest improvements

Use analyze_complexity to get metrics.
Provide a summary with prioritized recommendations.
  `);
  
  console.log("\n=== Review Results ===\n");
  console.log(result.text);
  console.log(`\n=== Stats: ${result.num_turns} turns, ${result.usage.input_tokens + result.usage.output_tokens} tokens ===`);
}

reviewCode("src/**/*.ts").then(() => agent.close());
```
