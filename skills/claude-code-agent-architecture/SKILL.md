---
name: claude-code-agent-architecture
description: Expert skill for understanding and working with Claude Code's internal architecture, tool system, and agent patterns
triggers:
  - how does claude code agent architecture work
  - explain claude code tool system
  - show me claude code internal structure
  - implement a custom tool for claude code
  - understand claude code agent loop
  - debug claude code tool execution
  - create custom claude code command
  - analyze claude code's query engine
---

# Claude Code Agent Architecture

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Expert knowledge for understanding, debugging, and extending the Claude Code CLI agent architecture based on the `learn-coding-agent` research repository.

## What This Project Is

A comprehensive research and learning repository that reverse-engineers and documents the architecture of the Claude Code CLI agent (v2.1.88). It provides:

- Deep analysis of the core agent loop and tool system
- Documentation of 40+ built-in tools and their permission model
- Insight into telemetry, feature flags, and remote control mechanisms
- Architectural patterns for building production-grade coding agents
- Multi-language analysis reports (EN/JA/KO/ZH)

**Key Statistics:**
- ~1,884 TypeScript files
- ~512,664 lines of code
- 40+ built-in tools
- 80+ slash commands
- Built on Bun/Node.js ≥18

## Core Architecture

### The Agent Loop

Claude Code implements a standard tool-using agent pattern:

```typescript
// Conceptual flow - src/query.ts (~785KB)
async function agentLoop(messages: Message[]) {
  while (true) {
    const response = await claudeAPI.chat({
      messages,
      tools: availableTools,
      // ... system prompt, model config
    });

    if (response.stop_reason === "end_turn") {
      return response.content; // Done
    }

    if (response.stop_reason === "tool_use") {
      // Execute tools in parallel
      const toolResults = await executeTools(response.tool_uses);
      
      // Append results to conversation
      messages.push({
        role: "user",
        content: toolResults
      });
      
      // Loop continues
    }
  }
}
```

### Directory Structure

```
src/
├── main.tsx              # REPL entry (4,683 lines)
├── QueryEngine.ts        # SDK/headless query engine
├── query.ts              # Main agent loop (785KB)
├── Tool.ts               # Tool interface factory
├── tools.ts              # Tool registry
├── commands.ts           # Slash command definitions
│
├── bridge/               # Claude Desktop integration
├── commands/             # 80+ slash commands
├── components/           # React/Ink terminal UI
├── services/             # Business logic
│   ├── api/              # Claude API client
│   ├── analytics/        # Telemetry
│   ├── mcp/              # Model Context Protocol
│   └── tools/            # Tool execution engine
└── state/                # Application state management
```

## Tool System Architecture

### Tool Interface

Tools are defined using the `buildTool` factory from `Tool.ts`:

```typescript
import { buildTool } from './Tool';

const myTool = buildTool({
  name: "my_custom_tool",
  
  description: "What this tool does - shown to Claude",
  
  input_schema: {
    type: "object",
    properties: {
      path: { type: "string", description: "File path" },
      content: { type: "string", description: "Content to write" }
    },
    required: ["path", "content"]
  },
  
  // Permission category
  category: "file_write",
  
  // Execution handler
  async execute({ input, context, signal }) {
    const { path, content } = input;
    
    // Check permissions
    if (!context.hasPermission("file_write")) {
      throw new Error("Permission denied");
    }
    
    // Do the work
    await fs.writeFile(path, content);
    
    return {
      success: true,
      message: `Wrote ${content.length} bytes to ${path}`
    };
  }
});
```

### Built-in Tool Categories

```typescript
// From tools.ts
export const TOOL_CATEGORIES = {
  file_read: ["read_file", "list_files", "search_files"],
  file_write: ["write_file", "edit_file", "delete_file"],
  shell: ["execute_command", "bg_process_start", "bg_process_stop"],
  browser: ["browser_action", "screenshot"],
  mcp: ["mcp_call_tool", "mcp_get_prompt"],
  agent: ["sub_agent_start", "sub_agent_stop"],
  // ... 40+ tools total
};
```

### Permission Flow

```typescript
// From hooks/useCanUseTool.tsx
function checkToolPermission(
  toolName: string,
  category: string,
  context: ExecutionContext
): PermissionResult {
  
  // 1. Check if tool is in allowed set
  if (!context.allowedTools.includes(toolName)) {
    return { allowed: false, reason: "Tool not in allowed set" };
  }
  
  // 2. Check category permissions
  const categoryPermission = context.permissions[category];
  if (categoryPermission === "never") {
    return { allowed: false, reason: "Category blocked" };
  }
  
  // 3. Check if prompt required
  if (categoryPermission === "ask") {
    return { 
      allowed: false, 
      needsPrompt: true,
      promptMessage: `Allow ${toolName}?`
    };
  }
  
  // 4. Check remote killswitches
  if (context.remoteConfig.killswitches[toolName]) {
    return { allowed: false, reason: "Remote killswitch active" };
  }
  
  return { allowed: true };
}
```

## Creating Custom Tools

### Step 1: Define the Tool

```typescript
// my-tools/customFileAnalyzer.ts
import { buildTool } from '../src/Tool';
import { readFile } from 'fs/promises';

export const customFileAnalyzer = buildTool({
  name: "analyze_code_complexity",
  
  description: 
    "Analyzes code complexity metrics for a given file. " +
    "Returns cyclomatic complexity, line count, and function count.",
  
  input_schema: {
    type: "object",
    properties: {
      file_path: {
        type: "string",
        description: "Path to the code file to analyze"
      },
      language: {
        type: "string",
        enum: ["typescript", "javascript", "python"],
        description: "Programming language"
      }
    },
    required: ["file_path"]
  },
  
  category: "file_read",
  
  async execute({ input, context, signal }) {
    const { file_path, language = "typescript" } = input;
    
    // Read file
    const content = await readFile(file_path, 'utf-8');
    
    // Simple analysis
    const lines = content.split('\n').length;
    const functions = (content.match(/function\s+\w+/g) || []).length;
    const complexity = calculateComplexity(content, language);
    
    return {
      file: file_path,
      metrics: {
        lines,
        functions,
        complexity,
        complexityPerFunction: functions > 0 ? complexity / functions : 0
      }
    };
  }
});

function calculateComplexity(code: string, lang: string): number {
  // Count decision points
  const patterns = {
    typescript: /\b(if|while|for|case|\?\?|&&|\|\|)\b/g,
    javascript: /\b(if|while|for|case|\?\?|&&|\|\|)\b/g,
    python: /\b(if|while|for|elif|and|or)\b/g
  };
  
  const matches = code.match(patterns[lang] || patterns.typescript);
  return (matches?.length || 0) + 1; // +1 base complexity
}
```

### Step 2: Register the Tool

```typescript
// src/tools.ts or custom registry
import { customFileAnalyzer } from '../my-tools/customFileAnalyzer';

export function registerCustomTools(registry: ToolRegistry) {
  registry.register(customFileAnalyzer);
  
  // Add to appropriate preset
  registry.addToPreset("code_analysis", [
    "analyze_code_complexity",
    "read_file",
    "search_files"
  ]);
}
```

### Step 3: Use in Agent Context

```typescript
// From QueryEngine.ts pattern
import { QueryEngine } from './src/QueryEngine';

const engine = new QueryEngine({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: "claude-sonnet-4-20250514",
  
  tools: [
    customFileAnalyzer,
    // ... other tools
  ],
  
  systemPrompt: `You are a code analysis assistant.
  Use analyze_code_complexity to provide insights about code quality.`,
});

const result = await engine.query(
  "Analyze the complexity of src/query.ts and suggest refactoring"
);
```

## Slash Commands

### Creating a Custom Command

```typescript
// commands/custom/complexity.tsx
import { Command } from '../../commands';
import React from 'react';

export const complexityCommand: Command = {
  name: "complexity",
  aliases: ["cx", "analyze"],
  
  description: "Analyze code complexity for current project",
  
  async execute({ args, context, render }) {
    const targetPath = args[0] || context.workingDirectory;
    
    render(
      <Text color="cyan">
        Analyzing complexity in {targetPath}...
      </Text>
    );
    
    // Trigger agent with custom tool
    const query = `Please analyze code complexity for all TypeScript files in ${targetPath} using the analyze_code_complexity tool. Provide a summary and highlight files that need refactoring.`;
    
    return context.sendQuery(query);
  }
};
```

## Model Context Protocol (MCP) Integration

### Connecting to MCP Servers

```typescript
// From services/mcp/
import { MCPClient } from './services/mcp/client';

const mcpClient = new MCPClient({
  serverCommand: "npx",
  serverArgs: ["-y", "@modelcontextprotocol/server-filesystem"],
  env: {
    ...process.env,
    FILESYSTEM_ROOT: "/home/user/projects"
  }
});

await mcpClient.connect();

// List available tools from MCP server
const tools = await mcpClient.listTools();

// Call MCP tool
const result = await mcpClient.callTool({
  name: "read_file",
  arguments: { path: "/home/user/projects/src/main.ts" }
});
```

### Adding MCP Server to Config

```typescript
// ~/.config/claude-code/mcp.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "env": {
        "FILESYSTEM_ROOT": "/home/user/projects"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

## Configuration Management

### User Settings Structure

```typescript
// From services/settingsSync/
interface ClaudeCodeSettings {
  // Model configuration
  model: "claude-sonnet-4" | "claude-opus-4" | string;
  maxTokens: number;
  temperature: number;
  
  // Tool permissions (per category)
  permissions: {
    file_read: "always" | "ask" | "never";
    file_write: "always" | "ask" | "never";
    shell: "always" | "ask" | "never";
    browser: "always" | "ask" | "never";
    // ...
  };
  
  // Feature flags
  features: {
    undercover_mode: boolean;
    voice_mode: boolean;
    sub_agents: boolean;
    mcp_enabled: boolean;
  };
  
  // Analytics
  telemetry: {
    enabled: boolean;
    datadog_enabled: boolean;
  };
  
  // MCP servers
  mcpServers: Record<string, MCPServerConfig>;
}
```

### Programmatic Config Access

```typescript
// Reading settings
import { getSettings } from './services/settingsSync/store';

const settings = await getSettings();
console.log("Current model:", settings.model);
console.log("File write permission:", settings.permissions.file_write);

// Updating settings
import { updateSettings } from './services/settingsSync/store';

await updateSettings({
  permissions: {
    ...settings.permissions,
    shell: "ask" // Require prompt for shell commands
  }
});
```

## Streaming Tool Execution

### Parallel Tool Execution

```typescript
// From services/tools/StreamingToolExecutor.ts
class StreamingToolExecutor {
  async executeToolsInParallel(
    toolUses: ToolUse[],
    context: ExecutionContext
  ): Promise<ToolResult[]> {
    
    // Group tools by dependency
    const batches = this.createDependencyBatches(toolUses);
    const results: ToolResult[] = [];
    
    for (const batch of batches) {
      // Execute batch in parallel
      const batchResults = await Promise.all(
        batch.map(async (toolUse) => {
          try {
            const tool = this.registry.get(toolUse.name);
            
            // Stream progress
            this.emit('tool:start', { 
              id: toolUse.id, 
              name: toolUse.name 
            });
            
            const result = await tool.execute({
              input: toolUse.input,
              context,
              signal: context.abortSignal
            });
            
            this.emit('tool:complete', { 
              id: toolUse.id, 
              result 
            });
            
            return { id: toolUse.id, result };
            
          } catch (error) {
            this.emit('tool:error', { 
              id: toolUse.id, 
              error 
            });
            
            return { 
              id: toolUse.id, 
              error: error.message,
              is_error: true 
            };
          }
        })
      );
      
      results.push(...batchResults);
    }
    
    return results;
  }
}
```

## Analytics and Telemetry

### Event Tracking Structure

```typescript
// From services/analytics/
interface TelemetryEvent {
  // Event identity
  event_name: string;
  event_id: string;
  
  // Timing
  timestamp: string;
  session_id: string;
  
  // Environment fingerprint
  os: string;
  arch: string;
  node_version: string;
  app_version: string;
  
  // Repository context (hashed)
  repo_hash?: string;
  project_type?: string;
  
  // Event-specific properties
  properties: Record<string, any>;
  
  // User context
  user_id?: string;
  is_internal?: boolean;
}

// Example: Tool execution event
const toolEvent: TelemetryEvent = {
  event_name: "tool_executed",
  event_id: crypto.randomUUID(),
  timestamp: new Date().toISOString(),
  session_id: context.sessionId,
  
  os: process.platform,
  arch: process.arch,
  node_version: process.version,
  app_version: "2.1.88",
  
  properties: {
    tool_name: "write_file",
    duration_ms: 234,
    success: true,
    permission_mode: "always"
  }
};
```

### Disabling Telemetry

```bash
# Note: First-party telemetry cannot be fully disabled
# Only third-party sinks can be disabled

# Disable Datadog sink
export CLAUDE_CODE_DISABLE_DATADOG=1

# Disable detailed tool logging
export OTEL_LOG_TOOL_DETAILS=0
```

## Common Patterns

### Pattern 1: Multi-Step File Operations

```typescript
// Agent conversation pattern for complex file tasks
const systemPrompt = `
When modifying multiple files:
1. Use read_file to understand current state
2. Use search_files to find related files
3. Use write_file for each change
4. Use execute_command to run tests
5. Summarize changes made

Always verify changes before confirming completion.
`;

const query = `
Refactor the authentication system:
- Move auth logic from src/main.ts to src/auth/
- Create separate files for login, logout, token refresh
- Update imports in all affected files
- Run tests to verify nothing broke
`;
```

### Pattern 2: Sub-Agent Delegation

```typescript
// From commands/agents/
async function delegateToSubAgent(
  task: string,
  context: ExecutionContext
) {
  // Start sub-agent with constrained permissions
  const subAgent = await context.startSubAgent({
    task,
    allowedTools: [
      "read_file",
      "search_files",
      "execute_command"
    ],
    permissions: {
      file_read: "always",
      file_write: "never", // Read-only
      shell: "ask"
    },
    maxSteps: 10
  });
  
  // Wait for completion
  const result = await subAgent.waitForCompletion();
  
  return result;
}

// Usage
const analysisResult = await delegateToSubAgent(
  "Analyze all TypeScript files and report common patterns",
  context
);
```

### Pattern 3: Context Compaction

```typescript
// From services/compact/
import { compactContext } from './services/compact';

async function manageContextSize(
  messages: Message[],
  maxTokens: number = 100000
) {
  const currentTokens = estimateTokens(messages);
  
  if (currentTokens > maxTokens) {
    // Compact old messages
    const compacted = await compactContext({
      messages,
      targetTokens: maxTokens * 0.8,
      preserveRecent: 10, // Keep last 10 messages
      strategy: "summarize" // or "truncate"
    });
    
    return compacted.messages;
  }
  
  return messages;
}
```

## Troubleshooting

### Issue: Tool Not Executing

```typescript
// Debug tool availability
import { tools } from './src/tools';

const tool = tools.find(t => t.name === "my_custom_tool");
if (!tool) {
  console.error("Tool not registered!");
}

// Check permissions
const canUse = context.permissions[tool.category];
console.log(`Permission for ${tool.category}:`, canUse);

// Enable verbose logging
process.env.DEBUG = "claude-code:tools";
```

### Issue: MCP Server Not Connecting

```bash
# Test MCP server manually
npx -y @modelcontextprotocol/server-filesystem

# Check config file
cat ~/.config/claude-code/mcp.json

# Enable MCP debug logs
export DEBUG=claude-code:mcp
```

### Issue: Rate Limiting

```typescript
// From services/api/withRetry.ts
const apiClient = new ClaudeClient({
  apiKey: process.env.ANTHROPIC_API_KEY,
  
  retry: {
    maxRetries: 3,
    backoff: "exponential",
    initialDelay: 1000,
    maxDelay: 60000
  },
  
  rateLimit: {
    requestsPerMinute: 50,
    tokensPerMinute: 40000
  }
});
```

### Issue: Memory/Context Too Large

```typescript
// Monitor context size
import { estimateTokens } from './services/api/tokens';

function checkContextSize(messages: Message[]) {
  const tokens = estimateTokens(messages);
  const warningThreshold = 150000;
  
  if (tokens > warningThreshold) {
    console.warn(`Context size: ${tokens} tokens (approaching limit)`);
    
    // Trigger compaction
    return compactContext(messages, 100000);
  }
  
  return messages;
}
```

## Advanced: Custom Query Engine

```typescript
// Building a custom agent using QueryEngine
import { QueryEngine } from './src/QueryEngine';
import { customFileAnalyzer } from './my-tools/customFileAnalyzer';

class CustomCodeAgent {
  private engine: QueryEngine;
  
  constructor() {
    this.engine = new QueryEngine({
      apiKey: process.env.ANTHROPIC_API_KEY,
      model: "claude-sonnet-4-20250514",
      
      tools: [
        customFileAnalyzer,
        // Add more custom tools
      ],
      
      systemPrompt: `You are a specialized code analysis agent.
      Your role is to help developers understand and improve code quality.
      
      When analyzing code:
      - Use analyze_code_complexity for metrics
      - Provide specific, actionable recommendations
      - Highlight security concerns
      - Suggest modern best practices`,
      
      onToolExecute: (tool, result) => {
        console.log(`[TOOL] ${tool.name}:`, result);
      },
      
      onError: (error) => {
        console.error("[ERROR]", error);
      }
    });
  }
  
  async analyzeProject(projectPath: string) {
    const query = `
    Please perform a comprehensive analysis of the project at ${projectPath}:
    1. Analyze complexity of all source files
    2. Identify files that need refactoring
    3. Check for common anti-patterns
    4. Suggest architectural improvements
    `;
    
    const response = await this.engine.query(query);
    return response;
  }
}

// Usage
const agent = new CustomCodeAgent();
const analysis = await agent.analyzeProject("/path/to/project");
console.log(analysis);
```

## Key Takeaways

1. **Core Loop**: Message → Claude API → Tool Use → Execute → Loop
2. **Tool System**: 40+ built-in tools, category-based permissions, parallel execution
3. **Extensibility**: Custom tools via `buildTool`, custom commands, MCP integration
4. **Production Harness**: Streaming, retries, compaction, sub-agents, telemetry
5. **Remote Control**: Hourly settings sync, killswitches, feature flags via GrowthBook
6. **Privacy Considerations**: Telemetry captures environment, repo hash, tool usage (see `docs/01-telemetry-and-privacy.md`)

For implementation details, refer to the full research reports in `docs/` (EN/JA/KO/ZH).
