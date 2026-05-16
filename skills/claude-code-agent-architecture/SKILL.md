---
name: claude-code-agent-architecture
description: Research repository analyzing Claude Code CLI Agent architecture, tools, telemetry, and internal mechanisms
triggers:
  - how does claude code agent work internally
  - analyze claude code architecture
  - what tools does claude code use
  - claude code agent system design
  - understanding coding agent architecture
  - claude code tool permissions and execution
  - implement coding agent like claude code
  - claude code feature flags and telemetry
---

# Claude Code Agent Architecture

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

`sanbuphy/learn-coding-agent` is a research repository that reverse-engineers and documents the architecture of Claude Code (the CLI coding agent). It provides deep technical analysis of:

- **Core agent loop**: How messages, tools, and API calls form the execution cycle
- **Tool system**: 40+ built-in tools with permission flows
- **Telemetry & privacy**: What data is collected and how
- **Hidden features**: Undercover mode, killswitches, remote control
- **Architecture patterns**: Production-grade harness mechanisms

This is a **learning resource** for understanding production coding agents, not the actual Claude Code source code.

## Installation

```bash
# Clone the research repository
git clone https://github.com/sanbuphy/learn-coding-agent.git
cd learn-coding-agent

# Install dependencies (if you want to explore the documented structure)
npm install
# or
bun install
```

**Note**: This is a documentation/research project. The TypeScript files are reconstructed architecture examples, not a runnable agent.

## Core Agent Loop Pattern

The fundamental agent pattern documented in this research:

```typescript
// The minimal agent loop
async function agentLoop(userMessage: string, messages: Message[]) {
  messages.push({ role: "user", content: userMessage });
  
  while (true) {
    const response = await callClaudeAPI(messages);
    
    if (response.stop_reason === "tool_use") {
      // Execute tools
      const toolResults = await executeTools(response.content);
      messages.push({ role: "assistant", content: response.content });
      messages.push({ role: "user", content: toolResults });
      // Loop continues
    } else {
      // Return final text response
      return response.content;
    }
  }
}
```

### The 12 Progressive Harness Mechanisms

Claude Code wraps the basic loop with production features:

1. **Permission System**: User approval for file writes, command execution
2. **Streaming**: Real-time token streaming with UI updates
3. **Concurrency**: Parallel tool execution via `StreamingToolExecutor`
4. **Context Compaction**: Automatic message history compression
5. **Sub-agents**: Specialized agents (verification, review, planning)
6. **Persistence**: Session state saved to disk
7. **MCP Integration**: Model Context Protocol server support
8. **Cost Tracking**: Token usage and API cost accumulation
9. **Analytics**: Telemetry events (OpenTelemetry + Datadog)
10. **Remote Control**: Feature flags and killswitches from server
11. **Error Recovery**: Retry logic with exponential backoff
12. **Multi-transport**: CLI, bridge, SDK interfaces

## Tool System Architecture

### Built-in Tool Categories

The research documents **40+ tools** across categories:

```typescript
// Tool registry structure
const toolRegistry = {
  // File operations
  "read_file": { category: "file", risk: "low" },
  "write_to_file": { category: "file", risk: "high", requiresPermission: true },
  "list_files": { category: "file", risk: "low" },
  
  // Command execution
  "execute_command": { category: "command", risk: "high", requiresPermission: true },
  "spawn_agent": { category: "agent", risk: "medium" },
  
  // Search & analysis
  "search_files": { category: "search", risk: "low" },
  "list_code_definition_names": { category: "code", risk: "low" },
  
  // Browser control (via MCP)
  "browser_navigate": { category: "browser", risk: "medium" },
  "browser_click": { category: "browser", risk: "medium" },
  
  // Memory & context
  "store_memory": { category: "memory", risk: "low" },
  "compact_context": { category: "system", risk: "low" }
};
```

### Permission Flow

```typescript
// Permission checking pattern
interface ToolPermission {
  toolName: string;
  approved: boolean;
  autoApprove?: boolean; // User set in config
  risk: "low" | "medium" | "high";
}

async function checkPermission(tool: Tool, args: any): Promise<boolean> {
  // 1. Check auto-approve rules
  if (config.autoApprove?.[tool.name]) {
    return true;
  }
  
  // 2. Low-risk tools auto-approved
  if (tool.risk === "low") {
    return true;
  }
  
  // 3. Show permission dialog
  const approved = await showPermissionDialog({
    tool: tool.name,
    description: tool.description,
    args: sanitizeArgs(args),
    risk: tool.risk
  });
  
  // 4. Log decision
  analytics.track("tool_permission_decision", {
    tool: tool.name,
    approved,
    autoApprove: false
  });
  
  return approved;
}
```

### Tool Execution with Streaming

```typescript
// StreamingToolExecutor pattern
class StreamingToolExecutor {
  async executeTools(toolCalls: ToolUse[]): Promise<ToolResult[]> {
    const results: ToolResult[] = [];
    
    // Group by parallelizable vs sequential
    const parallel = toolCalls.filter(t => this.canRunInParallel(t));
    const sequential = toolCalls.filter(t => !this.canRunInParallel(t));
    
    // Execute parallel tools concurrently
    if (parallel.length > 0) {
      const parallelResults = await Promise.all(
        parallel.map(tc => this.executeSingleTool(tc))
      );
      results.push(...parallelResults);
    }
    
    // Execute sequential tools one by one
    for (const toolCall of sequential) {
      const result = await this.executeSingleTool(toolCall);
      results.push(result);
    }
    
    return results;
  }
  
  private async executeSingleTool(toolCall: ToolUse): Promise<ToolResult> {
    const tool = toolRegistry.get(toolCall.name);
    
    // Permission check
    const approved = await checkPermission(tool, toolCall.input);
    if (!approved) {
      return {
        tool_use_id: toolCall.id,
        content: "Permission denied by user",
        is_error: true
      };
    }
    
    // Execute
    try {
      const output = await tool.execute(toolCall.input);
      return {
        tool_use_id: toolCall.id,
        content: output
      };
    } catch (error) {
      return {
        tool_use_id: toolCall.id,
        content: error.message,
        is_error: true
      };
    }
  }
}
```

## Key Architectural Components

### 1. Query Engine

```typescript
// QueryEngine.ts - SDK/headless query lifecycle
class QueryEngine {
  private messages: Message[] = [];
  private state: TaskState;
  
  async runQuery(input: string, options?: QueryOptions): Promise<QueryResult> {
    // Add user message
    this.messages.push({ role: "user", content: input });
    
    // Context compaction if needed
    if (this.shouldCompact()) {
      await this.compactContext();
    }
    
    // Main loop
    while (true) {
      const response = await this.callAPI();
      
      if (response.stop_reason === "tool_use") {
        const toolResults = await this.toolExecutor.executeTools(
          response.content.filter(c => c.type === "tool_use")
        );
        this.messages.push({ role: "assistant", content: response.content });
        this.messages.push({ role: "user", content: toolResults });
      } else {
        return { output: response.content, messages: this.messages };
      }
    }
  }
  
  private async compactContext(): Promise<void> {
    const compactedMessages = await this.compactionService.compact(
      this.messages,
      { strategy: "adaptive" }
    );
    this.messages = compactedMessages;
  }
}
```

### 2. MCP Integration

```typescript
// MCP (Model Context Protocol) server management
interface MCPServer {
  name: string;
  command: string;
  args: string[];
  env?: Record<string, string>;
}

class MCPManager {
  private servers: Map<string, MCPServerInstance> = new Map();
  
  async connectServer(config: MCPServer): Promise<void> {
    const instance = await this.spawn({
      command: config.command,
      args: config.args,
      env: { ...process.env, ...config.env }
    });
    
    // Register tools from server
    const tools = await instance.listTools();
    tools.forEach(tool => {
      toolRegistry.register(`mcp_${config.name}_${tool.name}`, tool);
    });
    
    this.servers.set(config.name, instance);
  }
  
  async callMCPTool(serverName: string, toolName: string, args: any): Promise<any> {
    const server = this.servers.get(serverName);
    return await server.callTool(toolName, args);
  }
}

// Example MCP configuration
const mcpConfig = {
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    },
    "filesystem": {
      "command": "node",
      "args": ["/path/to/mcp-server-filesystem/dist/index.js"],
      "env": {
        "ALLOWED_PATHS": "/workspace"
      }
    }
  }
};
```

### 3. Telemetry System

```typescript
// Analytics infrastructure
interface AnalyticsEvent {
  event: string;
  properties: Record<string, any>;
  userId?: string;
  sessionId: string;
  timestamp: number;
}

class AnalyticsService {
  private sinks = {
    firstParty: new OpenTelemetryClient(),
    datadog: new DatadogClient()
  };
  
  track(event: string, properties: Record<string, any>): void {
    const payload: AnalyticsEvent = {
      event,
      properties: {
        ...properties,
        // Automatic fingerprinting
        os: process.platform,
        arch: process.arch,
        nodeVersion: process.version,
        appVersion: packageJson.version,
        repoHash: this.getRepoHash(), // SHA256 of git remote URL
      },
      sessionId: this.sessionId,
      timestamp: Date.now()
    };
    
    // Send to both sinks
    this.sinks.firstParty.send(payload);
    this.sinks.datadog.send(payload);
    
    // Detailed tool logging if enabled
    if (process.env.OTEL_LOG_TOOL_DETAILS === "1") {
      this.logToolDetails(payload);
    }
  }
  
  trackToolUse(tool: string, args: any, result: any): void {
    this.track("tool_executed", {
      tool,
      argsSize: JSON.stringify(args).length,
      resultSize: JSON.stringify(result).length,
      success: !result.is_error
    });
  }
}
```

### 4. Feature Flags & Remote Control

```typescript
// Remote settings management
interface RemoteSettings {
  killswitches: {
    disable_bypass_permissions?: boolean;
    disable_fast_mode?: boolean;
    disable_analytics?: boolean;
  };
  modelOverrides?: {
    defaultModel?: string;
    enabledModels?: string[];
  };
  experimentFlags?: Record<string, any>;
}

class SettingsSyncService {
  private pollInterval = 60 * 60 * 1000; // 1 hour
  
  async poll(): Promise<void> {
    const settings = await fetch(
      "https://api.claude.ai/api/claude_code/settings",
      { headers: { Authorization: `Bearer ${this.token}` } }
    ).then(r => r.json());
    
    // Apply killswitches
    if (settings.killswitches?.disable_bypass_permissions) {
      config.bypassPermissions = false;
    }
    
    // Show blocking dialog for dangerous changes
    if (this.isDangerousChange(settings)) {
      const accepted = await showBlockingDialog({
        title: "Critical Update Required",
        message: "Claude Code settings have changed. Accept to continue.",
        actions: ["Accept", "Reject (Exit)"]
      });
      
      if (!accepted) {
        process.exit(1); // User rejected = app exits
      }
    }
    
    // Apply GrowthBook experiments
    this.applyExperiments(settings.experimentFlags);
  }
}
```

## Common Patterns

### Building a Custom Tool

```typescript
// Tool.ts pattern
import { buildTool } from "./Tool";

const customTool = buildTool({
  name: "analyze_code_quality",
  description: "Analyzes code quality metrics for a given file",
  parameters: {
    type: "object",
    properties: {
      filePath: { type: "string", description: "Path to file" },
      metrics: { 
        type: "array", 
        items: { type: "string" },
        description: "Metrics to analyze (complexity, coverage, etc.)"
      }
    },
    required: ["filePath"]
  },
  
  async execute({ filePath, metrics = ["complexity"] }) {
    // Read file
    const content = await fs.readFile(filePath, "utf-8");
    
    // Run analysis
    const results = await analyzeCode(content, metrics);
    
    // Return structured output
    return {
      file: filePath,
      metrics: results,
      summary: `Analyzed ${metrics.length} metrics for ${filePath}`
    };
  },
  
  // Tool metadata
  category: "code",
  risk: "low",
  requiresPermission: false
});

// Register tool
toolRegistry.register("analyze_code_quality", customTool);
```

### Implementing Sub-agents

```typescript
// Sub-agent pattern (verification, review, planning)
class SubAgent {
  constructor(
    private role: string,
    private systemPrompt: string
  ) {}
  
  async run(task: string, context: any): Promise<string> {
    const messages = [
      { role: "system", content: this.systemPrompt },
      { role: "user", content: this.formatTask(task, context) }
    ];
    
    const response = await callClaudeAPI({
      messages,
      model: "claude-sonnet-4.0", // Sub-agents use specific models
      max_tokens: 4096
    });
    
    return response.content[0].text;
  }
  
  private formatTask(task: string, context: any): string {
    return `Task: ${task}\n\nContext:\n${JSON.stringify(context, null, 2)}`;
  }
}

// Example: Code review sub-agent
const reviewAgent = new SubAgent(
  "code_reviewer",
  `You are a code review expert. Analyze the provided code changes for:
   - Security issues
   - Performance problems
   - Best practice violations
   Provide specific, actionable feedback.`
);

const reviewResult = await reviewAgent.run(
  "Review this pull request",
  { diff, files, description }
);
```

### Context Compaction Strategy

```typescript
// Adaptive context compaction
class CompactionService {
  async compact(messages: Message[], options: CompactOptions): Promise<Message[]> {
    const totalTokens = this.estimateTokens(messages);
    
    if (totalTokens < options.maxTokens) {
      return messages; // No compaction needed
    }
    
    // Strategy: Keep recent messages, summarize old ones
    const recentMessages = messages.slice(-10); // Keep last 10
    const oldMessages = messages.slice(0, -10);
    
    // Summarize old conversation
    const summary = await this.summarizeMessages(oldMessages);
    
    return [
      { role: "system", content: `Previous conversation summary:\n${summary}` },
      ...recentMessages
    ];
  }
  
  private async summarizeMessages(messages: Message[]): Promise<string> {
    const response = await callClaudeAPI({
      messages: [
        { role: "user", content: `Summarize this conversation concisely:\n${JSON.stringify(messages)}` }
      ],
      model: "claude-haiku-4.0", // Use fast model for summaries
      max_tokens: 1024
    });
    
    return response.content[0].text;
  }
}
```

## Configuration

### Environment Variables

```bash
# API Configuration
ANTHROPIC_API_KEY=sk-ant-xxx  # Required for Claude API

# Analytics Control
OTEL_LOG_TOOL_DETAILS=1       # Enable detailed tool logging (default: 0)
ANALYTICS_ENABLED=true         # Enable telemetry (default: true)

# Feature Flags
ENABLE_FAST_MODE=true          # Enable fast mode (Haiku for simple tasks)
ENABLE_VOICE_MODE=false        # Enable voice input (experimental)
ENABLE_UNDERCOVER_MODE=false   # Hide AI attribution in commits

# MCP Configuration
MCP_SERVER_PATH=/path/to/mcp-servers
MCP_ALLOWED_PATHS=/workspace,/home/user/projects

# Debug
DEBUG=claude:*                 # Enable debug logging
LOG_LEVEL=info                 # Log level (debug|info|warn|error)
```

### Configuration File

```json
// ~/.claude/config.json
{
  "defaultModel": "claude-sonnet-4.0",
  "autoApprove": {
    "read_file": true,
    "list_files": true,
    "search_files": true
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
    }
  },
  "maxTokens": 8192,
  "compactionThreshold": 100000,
  "telemetry": {
    "enabled": true,
    "userId": "anonymous"
  }
}
```

## Research Reports

The repository includes deep-dive reports in 4 languages:

### Key Reports

**01-telemetry-and-privacy**: Documents two analytics sinks (OpenTelemetry + Datadog), environment fingerprinting, and why opt-out isn't exposed in UI.

**02-hidden-features-and-codenames**: Reveals animal codenames (Capybara, Tengu, Numbat), feature flags using random word pairs, and internal vs external user differences.

**03-undercover-mode**: Official employees auto-enter mode that instructs the model to hide AI authorship in public repos ("Do not blow your cover").

**04-remote-control-and-killswitches**: Hourly polling of settings API, blocking dialogs that exit the app on rejection, 6+ killswitches.

**05-future-roadmap**: Upcoming Numbat model, KAIROS autonomous mode with `<tick>` heartbeats, voice mode, 17 unreleased tools.

Access reports in `docs/[en|ja|ko|zh]/` directories.

## Troubleshooting

### Understanding Agent Behavior

```typescript
// Enable verbose logging to see agent decision process
process.env.DEBUG = "claude:*";
process.env.LOG_LEVEL = "debug";

// This will log:
// - API requests/responses
// - Tool execution decisions
// - Permission checks
// - Context compaction triggers
```

### Analyzing Tool Permissions

```typescript
// Check which tools require permission
const highRiskTools = Array.from(toolRegistry.entries())
  .filter(([_, tool]) => tool.risk === "high")
  .map(([name, _]) => name);

console.log("High-risk tools:", highRiskTools);
// Output: ["write_to_file", "execute_command", "delete_file", ...]
```

### Debugging MCP Connections

```typescript
// Test MCP server connection
const mcpManager = new MCPManager();

try {
  await mcpManager.connectServer({
    name: "test-server",
    command: "npx",
    args: ["-y", "@modelcontextprotocol/server-puppeteer"]
  });
  
  const tools = await mcpManager.listTools("test-server");
  console.log("Available MCP tools:", tools);
} catch (error) {
  console.error("MCP connection failed:", error.message);
  // Common issues: command not found, port conflicts, auth failures
}
```

### Inspecting Telemetry Events

```bash
# Enable full tool argument logging
export OTEL_LOG_TOOL_DETAILS=1

# Run agent and grep for tool events
claude-code | grep "tool_executed"

# You'll see:
# {"event":"tool_executed","tool":"write_to_file","argsSize":1234,...}
```

## Additional Resources

- **Repository**: https://github.com/sanbuphy/learn-coding-agent
- **Deep Analysis Reports**: `docs/` directory (EN/JA/KO/ZH)
- **Architecture Diagrams**: Referenced in research reports
- **Related Projects**:
  - Model Context Protocol: https://modelcontextprotocol.io
  - Anthropic Claude API: https://docs.anthropic.com

---

**Note**: This is a research and educational repository. It documents publicly available information about coding agent architecture. For production coding agent development, refer to official Anthropic documentation and Claude API guides.
