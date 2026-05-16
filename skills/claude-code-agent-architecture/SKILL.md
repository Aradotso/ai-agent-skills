---
name: claude-code-agent-architecture
description: Build and understand CLI coding agents using Claude Code's production-grade architecture patterns
triggers:
  - how do I build a coding agent like Claude Code
  - explain the Claude Code agent loop architecture
  - implement a tool execution system for my AI agent
  - add MCP protocol support to my coding agent
  - set up streaming tool execution with permission checks
  - understand Claude Code's query engine pattern
  - build a REPL-based AI coding assistant
  - implement agent task state management
---

# Claude Code Agent Architecture

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What This Project Provides

This repository contains research and architectural analysis of Claude Code, a production CLI coding agent. It documents:

- **The 12 Progressive Harness Mechanisms** that layer production features on top of the core agent loop
- **40+ built-in tools** with permission system and sub-agent orchestration
- **Query Engine** pattern for headless/SDK agent execution
- **MCP (Model Context Protocol)** integration for extensible tool systems
- **Streaming execution** with concurrent tool orchestration
- **Context compaction** and memory management strategies
- **Telemetry, feature flags, and remote control** infrastructure

**Use this for:** Learning production-grade agent architecture patterns, building CLI coding agents, implementing tool execution systems, understanding agent state management.

## Core Architecture Pattern

### The Agent Loop

Every coding agent follows this minimal loop:

```typescript
// Simplified agent loop pattern
async function agentLoop(messages: Message[]) {
  while (true) {
    const response = await claude.messages.create({
      model: "claude-3-5-sonnet",
      messages,
      tools: availableTools,
    });

    if (response.stop_reason === "tool_use") {
      // Execute all tool uses in parallel
      const toolResults = await executeTools(response.content);
      
      // Append results and continue
      messages.push({ role: "assistant", content: response.content });
      messages.push({ role: "user", content: toolResults });
    } else {
      // Done - return text response
      return response.content;
    }
  }
}
```

### The 12 Production Harness Layers

Claude Code wraps the minimal loop with:

1. **Permission system** - User approval for dangerous operations
2. **Streaming execution** - Real-time output as tools run
3. **Concurrent orchestration** - Parallel tool execution with dependency tracking
4. **Context compaction** - Automatic message pruning when approaching limits
5. **Sub-agents** - Specialized agents for verification, review, planning
6. **State persistence** - Resume interrupted sessions
7. **MCP integration** - Dynamic tool loading from external servers
8. **Cost tracking** - Token usage and API cost monitoring
9. **Error recovery** - Automatic retry with exponential backoff
10. **Telemetry** - Usage analytics and performance monitoring
11. **Remote control** - Feature flags and managed settings
12. **Multi-modal UI** - Terminal UI with React/Ink components

## Building Your Own Agent

### 1. Query Engine Pattern

The `QueryEngine` separates agent logic from UI:

```typescript
import { QueryEngine, QueryContext } from "./QueryEngine";
import { Tool, buildTool } from "./Tool";

// Define your tools
const tools: Tool[] = [
  buildTool({
    name: "read_file",
    description: "Read contents of a file",
    input_schema: {
      type: "object",
      properties: {
        path: { type: "string", description: "File path" }
      },
      required: ["path"]
    },
    async execute({ path }) {
      return await fs.readFile(path, "utf-8");
    }
  }),
  
  buildTool({
    name: "write_file",
    description: "Write content to a file",
    input_schema: {
      type: "object",
      properties: {
        path: { type: "string" },
        content: { type: "string" }
      },
      required: ["path", "content"]
    },
    async execute({ path, content }) {
      await fs.writeFile(path, content);
      return `Written to ${path}`;
    },
    // Mark as requiring permission
    needsPermission: true
  })
];

// Initialize engine
const engine = new QueryEngine({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  model: "claude-3-5-sonnet-20241022",
  tools,
  onPermissionRequest: async (tool, input) => {
    // Your permission UI logic
    const approved = await askUser(`Allow ${tool.name}?`);
    return approved;
  }
});

// Run a query
const result = await engine.query({
  message: "Read package.json and update the version to 2.0.0",
  conversationHistory: []
});

console.log(result.finalResponse);
```

### 2. Streaming Tool Executor

Handle multiple tools executing in parallel:

```typescript
import { StreamingToolExecutor } from "./services/tools/StreamingToolExecutor";

class MyAgent {
  private executor: StreamingToolExecutor;
  
  constructor() {
    this.executor = new StreamingToolExecutor({
      tools: myTools,
      maxConcurrent: 5, // Run up to 5 tools simultaneously
      onToolStart: (toolUse) => {
        console.log(`▶ ${toolUse.name}`);
      },
      onToolComplete: (toolUse, result) => {
        console.log(`✓ ${toolUse.name} completed`);
      },
      onToolError: (toolUse, error) => {
        console.error(`✗ ${toolUse.name} failed:`, error);
      }
    });
  }
  
  async executeToolBatch(toolUses: ToolUse[]) {
    // Automatically handles concurrency, dependencies, streaming
    const results = await this.executor.executeBatch(toolUses);
    return results;
  }
}
```

### 3. MCP Integration

Load tools from MCP servers:

```typescript
import { MCPManager } from "./services/mcp/MCPManager";

const mcpManager = new MCPManager({
  configPath: "~/.config/my-agent/mcp.json"
});

// Load all configured MCP servers
await mcpManager.initialize();

// Get tools from all connected servers
const mcpTools = await mcpManager.getAllTools();

// Combine with built-in tools
const allTools = [...builtInTools, ...mcpTools];

// Execute MCP tool
const result = await mcpManager.executeTool({
  server: "filesystem",
  tool: "read_file",
  arguments: { path: "/etc/hosts" }
});
```

MCP configuration file (`~/.config/my-agent/mcp.json`):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
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

### 4. Permission System

Implement tiered permission checking:

```typescript
type PermissionLevel = "always" | "once" | "session" | "never";

class PermissionManager {
  private grants = new Map<string, PermissionLevel>();
  
  async checkPermission(
    tool: Tool,
    input: Record<string, any>
  ): Promise<boolean> {
    const key = `${tool.name}:${JSON.stringify(input)}`;
    const existing = this.grants.get(key);
    
    if (existing === "always") return true;
    if (existing === "never") return false;
    
    // Show permission dialog
    const { granted, remember } = await this.showDialog(tool, input);
    
    if (remember) {
      this.grants.set(key, granted ? "always" : "never");
    }
    
    return granted;
  }
  
  private async showDialog(tool: Tool, input: any) {
    // Your UI implementation
    console.log(`\n⚠️  Permission required:`);
    console.log(`   Tool: ${tool.name}`);
    console.log(`   Args: ${JSON.stringify(input, null, 2)}`);
    
    const answer = await prompt("Allow? (y/n/always/never): ");
    
    return {
      granted: ["y", "yes", "always"].includes(answer.toLowerCase()),
      remember: ["always", "never"].includes(answer.toLowerCase())
    };
  }
}
```

### 5. Context Compaction

Prevent hitting context limits:

```typescript
import { compactMessages } from "./services/compact/compactor";

class AgentSession {
  private messages: Message[] = [];
  private readonly maxTokens = 180000; // Leave room for response
  
  async addMessage(message: Message) {
    this.messages.push(message);
    
    // Check if we're approaching limit
    const tokenCount = await this.estimateTokens(this.messages);
    
    if (tokenCount > this.maxTokens) {
      console.log("📦 Compacting context...");
      
      this.messages = await compactMessages({
        messages: this.messages,
        targetTokens: Math.floor(this.maxTokens * 0.7),
        preserveRecent: 5, // Keep last 5 exchanges
        summarizeOld: true // Summarize older messages
      });
      
      console.log(`✓ Reduced to ~${await this.estimateTokens(this.messages)} tokens`);
    }
  }
  
  private async estimateTokens(messages: Message[]): Promise<number> {
    // Rough estimate: 1 token ≈ 4 chars
    const chars = messages.reduce((sum, m) => 
      sum + JSON.stringify(m.content).length, 0
    );
    return Math.ceil(chars / 4);
  }
}
```

### 6. Sub-Agent Pattern

Delegate specialized tasks to focused agents:

```typescript
class CodeReviewAgent {
  async review(code: string, context: string): Promise<string> {
    const response = await claude.messages.create({
      model: "claude-3-5-sonnet-20241022",
      max_tokens: 4000,
      system: `You are a code review specialist. Focus on:
- Security vulnerabilities
- Performance issues
- Best practices
- Edge cases`,
      messages: [{
        role: "user",
        content: `Review this code:\n\n${code}\n\nContext: ${context}`
      }]
    });
    
    return response.content[0].text;
  }
}

class MainAgent {
  private reviewer = new CodeReviewAgent();
  
  async handleCodeChange(file: string, diff: string) {
    // Main agent decides to delegate
    if (this.shouldReview(diff)) {
      const review = await this.reviewer.review(diff, file);
      return `Changes to ${file}:\n\n${review}`;
    }
    
    return `Applied changes to ${file}`;
  }
}
```

## Tool Development

### Tool Structure

```typescript
import { buildTool } from "./Tool";
import { z } from "zod";

export const myTool = buildTool({
  name: "my_tool",
  description: "What this tool does - be specific for Claude to choose it correctly",
  
  // JSON Schema for parameters
  input_schema: {
    type: "object",
    properties: {
      required_param: {
        type: "string",
        description: "Describe what this parameter does"
      },
      optional_param: {
        type: "number",
        description: "Optional parameter",
        default: 42
      }
    },
    required: ["required_param"]
  },
  
  // Execution logic
  async execute(input: { required_param: string; optional_param?: number }) {
    // Return string or structured data
    return {
      success: true,
      result: `Processed ${input.required_param}`
    };
  },
  
  // Optional: requires user permission
  needsPermission: true,
  
  // Optional: can this tool run concurrently with others?
  allowConcurrent: false,
  
  // Optional: tool category for organization
  category: "filesystem"
});
```

### Tool Categories in Claude Code

```typescript
// From tools.ts - common tool categories:
const categories = {
  filesystem: ["read_file", "write_file", "list_directory"],
  execution: ["execute_command", "spawn_agent"],
  search: ["grep_search", "search_and_replace"],
  browser: ["open_browser", "screenshot"],
  git: ["git_commit", "git_status", "git_diff"],
  memory: ["store_memory", "recall_memory"],
  analysis: ["code_review", "ask_followup"],
  mcp: [], // Dynamically loaded
};
```

### Advanced Tool: File Watcher

```typescript
export const watchFilesTool = buildTool({
  name: "watch_files",
  description: "Watch files for changes and notify when modified",
  input_schema: {
    type: "object",
    properties: {
      paths: {
        type: "array",
        items: { type: "string" },
        description: "File paths to watch"
      },
      action: {
        type: "string",
        enum: ["start", "stop", "status"],
        description: "Watch action"
      }
    },
    required: ["paths", "action"]
  },
  
  // State persists across tool calls
  state: {
    watchers: new Map<string, fs.FSWatcher>()
  },
  
  async execute({ paths, action }, { state }) {
    if (action === "start") {
      for (const path of paths) {
        if (state.watchers.has(path)) continue;
        
        const watcher = fs.watch(path, (event) => {
          console.log(`\n📝 File changed: ${path} (${event})`);
        });
        
        state.watchers.set(path, watcher);
      }
      return `Watching ${paths.length} files`;
    }
    
    if (action === "stop") {
      for (const path of paths) {
        state.watchers.get(path)?.close();
        state.watchers.delete(path);
      }
      return `Stopped watching ${paths.length} files`;
    }
    
    return `Currently watching: ${Array.from(state.watchers.keys()).join(", ")}`;
  },
  
  needsPermission: true
});
```

## Configuration

### Agent Configuration File

```typescript
// config.ts - Agent configuration schema
interface AgentConfig {
  // Claude API
  apiKey: string;              // from ANTHROPIC_API_KEY
  model: string;               // "claude-3-5-sonnet-20241022"
  maxTokens: number;           // 8000 default
  temperature: number;         // 0.0-1.0
  
  // Tools
  enabledTools: string[];      // Tool names to enable
  disabledTools: string[];     // Tools to disable
  autoApproveTools: string[];  // Tools that don't need permission
  
  // MCP
  mcpServers: Record<string, {
    command: string;
    args: string[];
    env?: Record<string, string>;
  }>;
  
  // Behavior
  maxConcurrentTools: number;  // 5 default
  autoCompact: boolean;        // true
  compactThreshold: number;    // 0.8 (80% of context)
  
  // Memory
  persistHistory: boolean;     // true
  historyPath: string;         // "~/.agent/history"
  
  // Telemetry
  telemetryEnabled: boolean;   // true (opt-out not supported in Claude Code)
}

// Load from ~/.config/my-agent/config.json
export async function loadConfig(): Promise<AgentConfig> {
  const configPath = path.join(os.homedir(), ".config/my-agent/config.json");
  
  if (!fs.existsSync(configPath)) {
    return getDefaultConfig();
  }
  
  const data = JSON.parse(await fs.readFile(configPath, "utf-8"));
  return { ...getDefaultConfig(), ...data };
}
```

### Environment Variables

```bash
# Required
export ANTHROPIC_API_KEY=sk-ant-xxxxx

# Optional - override config
export AGENT_MODEL=claude-3-5-sonnet-20241022
export AGENT_MAX_TOKENS=8000
export AGENT_TEMPERATURE=0.0

# MCP servers can reference env vars
export GITHUB_TOKEN=ghp_xxxxx
export DATABASE_URL=postgresql://localhost/mydb

# Telemetry (Claude Code doesn't respect opt-out, but you can in yours)
export AGENT_TELEMETRY=false

# Debug
export AGENT_DEBUG=true
export AGENT_LOG_LEVEL=debug
```

## Common Patterns

### Pattern: Agentic Task Execution

```typescript
interface Task {
  id: string;
  type: "implement" | "review" | "test" | "debug";
  description: string;
  context: Record<string, any>;
  status: "pending" | "running" | "completed" | "failed";
}

class TaskAgent {
  async executeTask(task: Task): Promise<void> {
    task.status = "running";
    
    try {
      const systemPrompt = this.getSystemPromptForTask(task.type);
      
      const response = await this.engine.query({
        system: systemPrompt,
        message: task.description,
        context: task.context,
        tools: this.getToolsForTask(task.type)
      });
      
      task.status = "completed";
      task.result = response.finalResponse;
      
    } catch (error) {
      task.status = "failed";
      task.error = error.message;
    }
  }
  
  private getSystemPromptForTask(type: Task["type"]): string {
    const prompts = {
      implement: "You are a senior software engineer implementing features...",
      review: "You are a code reviewer focusing on quality and security...",
      test: "You are a testing specialist writing comprehensive tests...",
      debug: "You are a debugging expert finding and fixing issues..."
    };
    return prompts[type];
  }
  
  private getToolsForTask(type: Task["type"]): Tool[] {
    const toolsets = {
      implement: [readFile, writeFile, executeCommand],
      review: [readFile, codeReview],
      test: [readFile, writeFile, executeCommand],
      debug: [readFile, grepSearch, executeCommand]
    };
    return toolsets[type];
  }
}
```

### Pattern: Progressive Permission Approval

```typescript
// Start restrictive, learn user preferences
class AdaptivePermissionManager {
  private autoApprove = new Set<string>();
  private autoDeny = new Set<string>();
  
  async checkPermission(tool: Tool, input: any): Promise<boolean> {
    const signature = `${tool.name}:${this.normalizeInput(input)}`;
    
    if (this.autoApprove.has(signature)) return true;
    if (this.autoDeny.has(signature)) return false;
    
    const response = await this.askUser(tool, input);
    
    if (response.remember) {
      if (response.granted) {
        this.autoApprove.add(signature);
      } else {
        this.autoDeny.add(signature);
      }
      this.persistPreferences();
    }
    
    return response.granted;
  }
  
  private normalizeInput(input: any): string {
    // Generalize paths, URLs for pattern matching
    const str = JSON.stringify(input);
    return str
      .replace(/\/[^/]+\//g, "/**/")  // Generalize paths
      .replace(/https?:\/\/[^/]+/, "https://*/");  // Generalize URLs
  }
}
```

### Pattern: Conversation Resume

```typescript
interface SavedSession {
  id: string;
  timestamp: number;
  messages: Message[];
  context: Record<string, any>;
  checksum: string;
}

class SessionManager {
  async saveSession(session: SavedSession) {
    const path = this.getSessionPath(session.id);
    await fs.writeFile(path, JSON.stringify(session, null, 2));
  }
  
  async resumeSession(sessionId: string): Promise<SavedSession | null> {
    const path = this.getSessionPath(sessionId);
    
    if (!fs.existsSync(path)) {
      return null;
    }
    
    const session = JSON.parse(await fs.readFile(path, "utf-8"));
    
    // Verify integrity
    if (this.computeChecksum(session) !== session.checksum) {
      throw new Error("Session corrupted");
    }
    
    return session;
  }
  
  private computeChecksum(session: SavedSession): string {
    const data = session.messages.map(m => m.content).join("");
    return crypto.createHash("sha256").update(data).digest("hex");
  }
  
  private getSessionPath(id: string): string {
    return path.join(os.homedir(), ".agent/sessions", `${id}.json`);
  }
}
```

## Troubleshooting

### Context Length Errors

```typescript
// Error: maximum context length exceeded
// Solution: Enable auto-compaction
const engine = new QueryEngine({
  maxTokens: 180000,
  autoCompact: true,
  compactThreshold: 0.7, // Compact at 70% usage
  onCompact: (oldCount, newCount) => {
    console.log(`Compacted: ${oldCount} → ${newCount} tokens`);
  }
});
```

### Tool Execution Timeouts

```typescript
// Problem: Long-running tools block the agent
// Solution: Set per-tool timeouts
export const longRunningTool = buildTool({
  name: "compile_project",
  timeout: 300000, // 5 minutes
  async execute(input) {
    return await withTimeout(
      actualCompile(input),
      300000,
      "Compilation timed out"
    );
  }
});

async function withTimeout<T>(
  promise: Promise<T>,
  ms: number,
  error: string
): Promise<T> {
  const timeout = new Promise<never>((_, reject) =>
    setTimeout(() => reject(new Error(error)), ms)
  );
  return Promise.race([promise, timeout]);
}
```

### Permission Dialog UX

```typescript
// Problem: Too many permission prompts
// Solution: Batch similar requests
class BatchPermissionManager {
  private pending: Array<{
    tool: Tool;
    input: any;
    resolve: (value: boolean) => void;
  }> = [];
  
  private batchTimer: NodeJS.Timeout | null = null;
  
  async checkPermission(tool: Tool, input: any): Promise<boolean> {
    return new Promise((resolve) => {
      this.pending.push({ tool, input, resolve });
      
      if (!this.batchTimer) {
        this.batchTimer = setTimeout(() => this.processBatch(), 100);
      }
    });
  }
  
  private async processBatch() {
    const batch = this.pending.splice(0);
    this.batchTimer = null;
    
    console.log(`\n⚠️  ${batch.length} tools need permission:`);
    batch.forEach((req, i) => {
      console.log(`  ${i + 1}. ${req.tool.name}`);
    });
    
    const answer = await prompt("Approve all? (y/n): ");
    const approved = answer.toLowerCase() === "y";
    
    batch.forEach(req => req.resolve(approved));
  }
}
```

### MCP Server Connection Issues

```bash
# Problem: MCP server won't connect
# Debug steps:

# 1. Test server directly
npx -y @modelcontextprotocol/server-filesystem /tmp

# 2. Check logs
export AGENT_DEBUG=true
export MCP_LOG_LEVEL=debug

# 3. Verify server config
cat ~/.config/my-agent/mcp.json

# 4. Check environment variables are accessible
echo $GITHUB_TOKEN

# 5. Test with minimal config
{
  "mcpServers": {
    "test": {
      "command": "node",
      "args": ["--version"]
    }
  }
}
```

### Memory Leaks in Long Sessions

```typescript
// Problem: Agent consumes too much memory over time
// Solution: Periodic cleanup
class AgentWithCleanup {
  private cleanupInterval = setInterval(() => {
    this.cleanup();
  }, 60000); // Every minute
  
  private cleanup() {
    // Clear old tool results
    this.messages = this.messages.filter((m, i) => {
      if (m.role === "user" && m.content.includes("tool_result")) {
        return i >= this.messages.length - 100; // Keep last 100
      }
      return true;
    });
    
    // Force GC if available
    if (global.gc) {
      global.gc();
    }
  }
  
  destroy() {
    clearInterval(this.cleanupInterval);
  }
}
```

## Key Architectural Insights

1. **Separation of Concerns**: Query engine (logic) vs REPL (UI) enables headless operation
2. **Tool Orchestration**: Parallel execution with dependency tracking improves performance
3. **Permission Layering**: Different trust levels for different operations
4. **Sub-Agent Delegation**: Specialized prompts for focused tasks
5. **Context Management**: Proactive compaction prevents runtime errors
6. **Streaming Everything**: Real-time feedback improves UX dramatically
7. **MCP as Plugin System**: External tools without rebuilding agent
8. **State Persistence**: Resume interrupted work seamlessly
9. **Telemetry by Default**: Understand how users actually use your agent
10. **Remote Control**: Feature flags enable safe rollouts and killswitches

## References

- **Core Loop**: `src/query.ts` (~785KB - main agent logic)
- **Tool System**: `src/Tool.ts`, `src/tools.ts`
- **Query Engine**: `src/QueryEngine.ts` (headless SDK)
- **Streaming Executor**: `src/services/tools/StreamingToolExecutor.ts`
- **MCP Manager**: `src/services/mcp/`
- **Permission System**: `src/hooks/toolPermission/`
- **Context Compaction**: `src/services/compact/`
- **Deep Analysis**: `docs/en/` (telemetry, hidden features, remote control, roadmap)

This architecture represents production-grade patterns for building reliable, scalable coding agents.
