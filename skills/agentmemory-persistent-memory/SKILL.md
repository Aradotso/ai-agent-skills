---
name: agentmemory-persistent-memory
description: Add persistent memory to AI coding agents using agentmemory - remembers context, preferences, and decisions across sessions
triggers:
  - "set up persistent memory for this project"
  - "remember my coding preferences and architecture decisions"
  - "configure agentmemory to track our conversations"
  - "add memory to this AI agent"
  - "install agentmemory and connect it"
  - "help the agent remember what we discussed before"
  - "set up cross-session memory"
  - "configure memory persistence for Claude/Cursor"
---

# agentmemory-persistent-memory

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

agentmemory provides persistent memory for AI coding agents (Claude Code, Cursor, Gemini CLI, Codex, etc.) so they remember architecture decisions, preferences, bugs, and context across sessions. Built on the iii engine, it achieves 95.2% retrieval accuracy (R@5) and reduces token usage by 92% compared to pasting full context.

## What It Does

- **Cross-session memory**: Agent remembers previous conversations, decisions, and code patterns
- **Zero setup**: No external databases, runs locally
- **Works everywhere**: MCP server + native plugins for Claude Code, Codex, OpenClaw, Hermes, Cursor, and more
- **Hybrid search**: Combines embeddings + BM25 for accurate retrieval
- **Auto-capture**: 12 hooks automatically store context from agent actions
- **Real-time viewer**: Web UI to inspect and manage memories at http://localhost:3112

## Installation

### Global Installation (Recommended)

```bash
# Install globally to get the bare `agentmemory` command
npm install -g @agentmemory/agentmemory

# Start the memory server
agentmemory

# Server runs on http://localhost:3111 (API)
# Viewer runs on http://localhost:3112 (UI)
```

### npx (No Install)

```bash
# Run without installing
npx @agentmemory/agentmemory

# Force latest version if cache is stale
npx -y @agentmemory/agentmemory@latest
```

### Project-Level Installation

```typescript
// package.json
{
  "dependencies": {
    "@agentmemory/agentmemory": "^0.9.16"
  }
}
```

```typescript
// TypeScript/JavaScript usage
import { AgentMemory } from '@agentmemory/agentmemory';

const memory = new AgentMemory({
  port: 3111,
  dataDir: './data/memory'
});

await memory.start();
```

## Quick Start

### 1. Start the Server

```bash
agentmemory
```

Output:
```
✓ agentmemory server running on http://localhost:3111
✓ Real-time viewer at http://localhost:3112
✓ MCP server ready for agent connections
```

### 2. Connect Your Agent

```bash
# Claude Code
agentmemory connect claude-code

# Codex CLI
agentmemory connect codex

# Cursor (via MCP)
agentmemory connect cursor

# Gemini CLI
agentmemory connect gemini-cli

# Generic MCP for any agent
agentmemory connect mcp
```

### 3. Demo Mode (Optional)

```bash
# Seed sample sessions and test recall
agentmemory demo
```

## Agent-Specific Setup

### Claude Code

Native plugin with 12 auto-hooks:

```bash
agentmemory connect claude-code
```

Creates `~/.claude-code/plugins/agentmemory.json`:

```json
{
  "name": "agentmemory",
  "memoryServer": "http://localhost:3111",
  "hooks": {
    "onStart": true,
    "onCommand": true,
    "onFileEdit": true,
    "onSearch": true,
    "onError": true,
    "onDecision": true
  }
}
```

### Cursor (via MCP)

```bash
agentmemory connect cursor
```

Adds to `~/Library/Application Support/Cursor/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "agentmemory": {
      "command": "npx",
      "args": ["-y", "@agentmemory/agentmemory", "mcp"],
      "env": {
        "AGENTMEMORY_URL": "http://localhost:3111"
      }
    }
  }
}
```

### Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "agentmemory": {
      "command": "npx",
      "args": ["-y", "@agentmemory/agentmemory", "mcp"]
    }
  }
}
```

### Codex CLI

```bash
agentmemory connect codex
```

Creates `~/.codex/plugins/agentmemory.js`:

```javascript
module.exports = {
  name: 'agentmemory',
  memoryServer: 'http://localhost:3111',
  hooks: ['onStart', 'onCommand', 'onFileEdit', 'onSearch', 'onError', 'onDecision']
};
```

## Core API Usage

### TypeScript/JavaScript

```typescript
import { AgentMemory } from '@agentmemory/agentmemory';

// Initialize
const memory = new AgentMemory({
  port: 3111,
  dataDir: './data/memory',
  embeddingModel: 'all-MiniLM-L6-v2', // Local, free
  enableViewer: true,
  viewerPort: 3112
});

await memory.start();

// Store a memory
await memory.store({
  sessionId: 'session-001',
  content: 'User prefers JWT auth with jose library for Edge compatibility',
  type: 'preference',
  tags: ['auth', 'jwt', 'edge'],
  confidence: 0.95
});

// Retrieve relevant memories
const results = await memory.search({
  query: 'authentication setup',
  sessionId: 'session-001',
  limit: 5
});

console.log(results);
// [
//   {
//     content: 'User prefers JWT auth with jose library...',
//     score: 0.92,
//     type: 'preference',
//     tags: ['auth', 'jwt', 'edge'],
//     timestamp: '2026-05-16T10:30:00Z'
//   }
// ]

// Store code context
await memory.storeCode({
  sessionId: 'session-001',
  filePath: 'src/middleware/auth.ts',
  language: 'typescript',
  summary: 'JWT middleware using jose, validates tokens on Edge runtime',
  keyDecisions: [
    'Chose jose over jsonwebtoken for Edge compatibility',
    'Token expiry set to 7 days',
    'Refresh tokens stored in httpOnly cookies'
  ]
});

// Recall at session start
const context = await memory.recallForSession('session-002');
console.log(context.relevantMemories);
// Agent automatically gets context from previous sessions
```

### REST API

```bash
# Store memory
curl -X POST http://localhost:3111/api/memory/store \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "session-001",
    "content": "User prefers TypeScript strict mode",
    "type": "preference",
    "tags": ["typescript", "config"],
    "confidence": 0.9
  }'

# Search memories
curl -X POST http://localhost:3111/api/memory/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "typescript configuration",
    "sessionId": "session-001",
    "limit": 5
  }'

# Get session context
curl http://localhost:3111/api/memory/session/session-001

# Delete memory
curl -X DELETE http://localhost:3111/api/memory/delete/mem_abc123xyz

# Health check
curl http://localhost:3111/health
```

## MCP Server (Model Context Protocol)

agentmemory exposes 51 MCP tools for agent integration:

### Available Tools

```typescript
// Memory operations
- memory_store: Store new memory
- memory_search: Search memories by query
- memory_recall: Get context for session
- memory_update: Update existing memory
- memory_delete: Delete specific memory
- memory_list: List all memories (paginated)

// Session management
- session_create: Start new session
- session_get: Get session details
- session_list: List all sessions
- session_archive: Archive completed session

// Code context
- code_store: Store code snippet with context
- code_search: Search code memories
- code_link: Link memories to code files

// Preferences
- preference_set: Store user preference
- preference_get: Retrieve preference
- preference_list: List all preferences

// Knowledge graph
- graph_add_node: Add knowledge node
- graph_add_edge: Connect nodes
- graph_query: Query relationships
- graph_traverse: Walk knowledge graph

// Analytics
- stats_get: Get memory statistics
- stats_session: Session-level stats
- stats_recall_accuracy: Retrieval metrics
```

### MCP Client Example

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const transport = new StdioClientTransport({
  command: 'npx',
  args: ['-y', '@agentmemory/agentmemory', 'mcp']
});

const client = new Client({
  name: 'my-agent',
  version: '1.0.0'
}, {
  capabilities: {}
});

await client.connect(transport);

// Call MCP tool
const result = await client.request({
  method: 'tools/call',
  params: {
    name: 'memory_store',
    arguments: {
      sessionId: 'session-001',
      content: 'User wants error handling with Zod validation',
      type: 'decision',
      tags: ['error-handling', 'validation', 'zod']
    }
  }
});

console.log(result);
```

## Configuration

### Environment Variables

```bash
# Server settings
AGENTMEMORY_PORT=3111
AGENTMEMORY_VIEWER_PORT=3112
AGENTMEMORY_DATA_DIR=./data/memory

# Embedding model (local)
AGENTMEMORY_EMBEDDING_MODEL=all-MiniLM-L6-v2

# Or use OpenAI embeddings (requires API key)
OPENAI_API_KEY=your_openai_key_here
AGENTMEMORY_EMBEDDING_PROVIDER=openai
AGENTMEMORY_EMBEDDING_MODEL=text-embedding-3-small

# Search tuning
AGENTMEMORY_HYBRID_ALPHA=0.7  # 0.7 = 70% embeddings, 30% BM25
AGENTMEMORY_MAX_RESULTS=10

# Memory lifecycle
AGENTMEMORY_AUTO_ARCHIVE_DAYS=30
AGENTMEMORY_MIN_CONFIDENCE=0.5

# Hooks (for Claude Code, Codex)
AGENTMEMORY_ENABLE_HOOKS=true
AGENTMEMORY_HOOK_ON_FILE_EDIT=true
AGENTMEMORY_HOOK_ON_COMMAND=true
AGENTMEMORY_HOOK_ON_ERROR=true
```

### Configuration File

Create `agentmemory.config.json`:

```json
{
  "server": {
    "port": 3111,
    "viewerPort": 3112,
    "dataDir": "./data/memory"
  },
  "embedding": {
    "provider": "local",
    "model": "all-MiniLM-L6-v2"
  },
  "search": {
    "hybridAlpha": 0.7,
    "maxResults": 10,
    "minConfidence": 0.5
  },
  "lifecycle": {
    "autoArchiveDays": 30,
    "pruneInactive": true
  },
  "hooks": {
    "onStart": true,
    "onCommand": true,
    "onFileEdit": true,
    "onSearch": true,
    "onError": true,
    "onDecision": true
  }
}
```

Load config:

```bash
agentmemory --config ./agentmemory.config.json
```

## Common Patterns

### Pattern 1: Architecture Decision Capture

```typescript
// When agent makes architectural decision, store it
await memory.store({
  sessionId: currentSession,
  content: 'Decided to use PostgreSQL with Prisma ORM for type safety',
  type: 'decision',
  tags: ['architecture', 'database', 'prisma', 'postgresql'],
  confidence: 0.95,
  metadata: {
    rationale: 'Team familiar with Prisma, need strong typing',
    alternatives: ['MySQL + Drizzle', 'MongoDB'],
    impact: 'high',
    reversible: false
  }
});

// Next session, agent recalls this automatically
const context = await memory.recallForSession(newSession);
// Agent knows: "Previously decided on PostgreSQL + Prisma for type safety"
```

### Pattern 2: Code Pattern Memory

```typescript
// Store reusable code pattern
await memory.storeCode({
  sessionId: currentSession,
  filePath: 'src/lib/api-client.ts',
  language: 'typescript',
  summary: 'API client with retry logic and exponential backoff',
  pattern: `
    export async function fetchWithRetry(url: string, options?: RequestInit) {
      let attempt = 0;
      while (attempt < 3) {
        try {
          return await fetch(url, options);
        } catch (err) {
          if (attempt === 2) throw err;
          await new Promise(r => setTimeout(r, 2 ** attempt * 1000));
          attempt++;
        }
      }
    }
  `,
  tags: ['api', 'retry', 'fetch', 'resilience'],
  keyDecisions: [
    '3 retries with exponential backoff',
    'Throws on final failure',
    'No retry on 4xx errors (only network failures)'
  ]
});

// Later, when user asks about API calls
const codeMemories = await memory.search({
  query: 'api error handling',
  type: 'code',
  limit: 3
});
// Agent suggests: "We use fetchWithRetry from api-client.ts for resilience"
```

### Pattern 3: Bug Fix Memory

```typescript
// When bug is found and fixed
await memory.store({
  sessionId: currentSession,
  content: 'Fixed race condition in WebSocket reconnection logic',
  type: 'bug_fix',
  tags: ['websocket', 'race-condition', 'concurrency'],
  confidence: 1.0,
  metadata: {
    symptom: 'Duplicate connections causing message echoes',
    rootCause: 'reconnect() called before disconnect() completed',
    fix: 'Added connectionLock mutex to serialize connect/disconnect',
    filePath: 'src/websocket/connection.ts',
    linesChanged: '45-67',
    testAdded: 'tests/websocket/reconnect.test.ts'
  }
});

// Prevents re-introducing similar bugs
const pastBugs = await memory.search({
  query: 'websocket connection issues',
  type: 'bug_fix'
});
// Agent warns: "Watch out - we had a race condition here before (see connectionLock)"
```

### Pattern 4: Preference Learning

```typescript
// Capture user preferences over time
await memory.store({
  sessionId: currentSession,
  content: 'User prefers named exports over default exports',
  type: 'preference',
  tags: ['code-style', 'exports', 'modules'],
  confidence: 0.85
});

await memory.store({
  sessionId: currentSession,
  content: 'User wants all async errors wrapped in Result<T, E> type',
  type: 'preference',
  tags: ['error-handling', 'types', 'functional'],
  confidence: 0.9
});

// Agent adapts to preferences automatically
const prefs = await memory.search({
  query: 'code style preferences',
  type: 'preference'
});
// Agent generates code matching user's style without being told
```

### Pattern 5: Cross-File Context

```typescript
// Link related code across files
await memory.storeCode({
  sessionId: currentSession,
  filePath: 'src/middleware/auth.ts',
  language: 'typescript',
  summary: 'JWT authentication middleware',
  tags: ['auth', 'jwt', 'middleware']
});

await memory.linkCode({
  from: 'src/middleware/auth.ts',
  to: 'src/routes/api.ts',
  relationship: 'used_by',
  context: 'All /api/* routes use JWT auth middleware'
});

await memory.linkCode({
  from: 'src/middleware/auth.ts',
  to: 'tests/middleware/auth.test.ts',
  relationship: 'tested_by',
  context: 'Test coverage: token validation, expiry, refresh'
});

// Agent understands impact of changes
const linkedFiles = await memory.getCodeLinks('src/middleware/auth.ts');
// Agent warns: "Changing auth.ts affects 12 API routes and has test coverage"
```

## Real-Time Viewer

Open http://localhost:3112 to see:

- **Memory timeline**: Visual history of all stored memories
- **Session explorer**: Browse by session, see context evolution
- **Search interface**: Test queries, see retrieval scores
- **Knowledge graph**: Visual map of connected memories
- **Stats dashboard**: Recall accuracy, token savings, storage usage

### Viewer API

```typescript
// Programmatic access to viewer data
const viewerData = await fetch('http://localhost:3112/api/viewer/data').then(r => r.json());

console.log(viewerData);
// {
//   sessions: [...],
//   memories: [...],
//   stats: { totalMemories: 1247, avgRecall: 0.952, tokensSaved: 168420 }
// }
```

## CLI Commands

```bash
# Start server
agentmemory

# Start with custom port
agentmemory --port 4000 --viewer-port 4001

# Start in background
agentmemory --daemon

# Stop daemon
agentmemory stop

# Connect agent
agentmemory connect <agent-name>

# Run demo
agentmemory demo

# Export memories
agentmemory export --format json --output memories.json

# Import memories
agentmemory import --input memories.json

# Clear all memories (with confirmation)
agentmemory clear

# Show stats
agentmemory stats

# Health check
agentmemory health

# Update to latest
agentmemory update
```

## Troubleshooting

### Server Won't Start

```bash
# Check if port is in use
lsof -i :3111

# Kill existing process
kill -9 $(lsof -t -i:3111)

# Start on different port
agentmemory --port 4000
```

### Agent Not Connecting

```bash
# Verify server is running
curl http://localhost:3111/health

# Check MCP configuration
cat ~/Library/Application\ Support/Cursor/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json

# Test MCP connection manually
npx @agentmemory/agentmemory mcp
```

### Poor Recall Accuracy

```typescript
// Increase hybrid search weight toward embeddings
AGENTMEMORY_HYBRID_ALPHA=0.85  // 85% embeddings, 15% BM25

// Or switch to OpenAI embeddings for better quality
OPENAI_API_KEY=your_key_here
AGENTMEMORY_EMBEDDING_PROVIDER=openai
AGENTMEMORY_EMBEDDING_MODEL=text-embedding-3-large
```

### Memory Storage Growing Large

```bash
# Archive old sessions
agentmemory archive --older-than 30d

# Prune low-confidence memories
agentmemory prune --min-confidence 0.5

# Compact database
agentmemory compact
```

### Viewer Not Loading

```bash
# Check viewer port
curl http://localhost:3112

# Restart with viewer explicitly enabled
agentmemory --enable-viewer --viewer-port 3112
```

### npx Cache Issues

```bash
# Clear npx cache (macOS/Linux)
rm -rf ~/.npm/_npx

# Windows
# Delete %LOCALAPPDATA%\npm-cache\_npx

# Force latest version
npx -y @agentmemory/agentmemory@latest
```

## Advanced: iii Engine Integration

agentmemory is built on the [iii engine](https://github.com/iii-hq/iii), a knowledge processing system. You can use iii directly for custom memory backends:

```typescript
import { IIIEngine } from '@iii-hq/iii';

const engine = new IIIEngine({
  storage: 'local',
  embeddings: 'all-MiniLM-L6-v2'
});

// Store knowledge node
await engine.store({
  content: 'User prefers Tailwind CSS for styling',
  type: 'preference',
  tags: ['ui', 'css', 'tailwind'],
  confidence: 0.9
});

// Query with iii's advanced graph traversal
const results = await engine.query({
  semantic: 'css framework choice',
  graphDepth: 2,
  includeRelated: true
});
```

## Environment Setup Example

```bash
# .env file for production
AGENTMEMORY_PORT=3111
AGENTMEMORY_VIEWER_PORT=3112
AGENTMEMORY_DATA_DIR=/var/lib/agentmemory
AGENTMEMORY_EMBEDDING_PROVIDER=local
AGENTMEMORY_EMBEDDING_MODEL=all-MiniLM-L6-v2
AGENTMEMORY_HYBRID_ALPHA=0.75
AGENTMEMORY_AUTO_ARCHIVE_DAYS=90
AGENTMEMORY_MIN_CONFIDENCE=0.6
AGENTMEMORY_ENABLE_HOOKS=true
AGENTMEMORY_LOG_LEVEL=info
```

## Performance Notes

- **Embedding**: Local model (all-MiniLM-L6-v2) processes ~500 tokens/sec on M1 Mac
- **Search**: Hybrid search returns results in <50ms for 10K memories
- **Storage**: ~1KB per memory average, ~10MB for 10K memories
- **Startup**: Cold start ~2 seconds, warm start <500ms
- **Token savings**: 92% reduction vs. full context pasting (see benchmarks)

## Resources

- [GitHub Repository](https://github.com/rohitg00/agentmemory)
- [Design Doc Gist](https://gist.github.com/rohitg00/2067ab416f7bbe447c1977edaaa681e2) (1200+ stars)
- [Product Hunt Launch](https://www.producthunt.com/products/agent-memory-dev)
- [iii Engine](https://github.com/iii-hq/iii)
- [npm Package](https://www.npmjs.com/package/@agentmemory/agentmemory)
- [Landing Site](https://agent-memory.dev)
