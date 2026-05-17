---
name: vercel-open-agents
description: Build and run durable background coding agents with workflow orchestration, isolated sandboxes, and GitHub integration on Vercel.
triggers:
  - how do I build a coding agent with Open Agents
  - set up a durable agent workflow on Vercel
  - create an agent that can make GitHub PRs
  - use Open Agents sandbox orchestration
  - implement agent tools and skills in Open Agents
  - configure GitHub app integration for Open Agents
  - run background agents with Vercel workflows
  - add custom tools to my Open Agents agent
---

# Vercel Open Agents

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Open Agents is an open-source template for building cloud-based coding agents that run as durable workflows on Vercel. It provides web UI, agent runtime, sandbox orchestration, and GitHub integration to execute multi-step coding tasks without local execution.

## Architecture

Open Agents uses a three-layer architecture:

```text
Web App (Next.js) → Agent Workflow (Vercel Workflow SDK) → Sandbox VM (isolated execution)
```

**Key principle**: The agent runs *outside* the sandbox and interacts with it through tools. This separation enables:
- Durable execution beyond single request lifecycles
- Independent sandbox hibernation and resume
- Flexible model/provider swapping
- Clean separation of concerns

## Installation

### Deploy to Vercel

1. **Fork and import** the repository to Vercel
2. **Generate secrets**:
   ```bash
   openssl rand -base64 32  # for BETTER_AUTH_SECRET
   ```

3. **Set required environment variables**:
   ```env
   POSTGRES_URL=your_neon_postgres_url
   BETTER_AUTH_SECRET=your_generated_secret
   ```

4. **Deploy** to get production URL

5. **Create Vercel OAuth app** at https://vercel.com/account/settings/oauth-apps
   - Callback: `https://YOUR_DOMAIN/api/auth/callback/vercel`

6. **Add OAuth credentials**:
   ```env
   NEXT_PUBLIC_VERCEL_APP_CLIENT_ID=your_client_id
   VERCEL_APP_CLIENT_SECRET=your_client_secret
   ```

7. **Create GitHub App** (optional, for repo access):
   - Homepage: `https://YOUR_DOMAIN`
   - Callback: `https://YOUR_DOMAIN/api/auth/callback/github`
   - Setup URL: `https://YOUR_DOMAIN/api/github/app/callback`
   - Make app public for org installs

8. **Add GitHub credentials**:
   ```env
   NEXT_PUBLIC_GITHUB_CLIENT_ID=your_github_app_client_id
   GITHUB_CLIENT_SECRET=your_github_app_client_secret
   GITHUB_APP_ID=your_app_id
   GITHUB_APP_PRIVATE_KEY=your_pem_private_key
   NEXT_PUBLIC_GITHUB_APP_SLUG=your_app_slug
   GITHUB_WEBHOOK_SECRET=your_webhook_secret
   ```

### Local Development

```bash
# Install dependencies
bun install

# Copy environment template
cp apps/web/.env.example apps/web/.env

# Fill in required variables in apps/web/.env

# Start dev server
bun run web
```

For existing Vercel projects:
```bash
vc env pull
```

## Core Concepts

### Workflows

Agent runs execute as Vercel Workflow SDK workflows, enabling:
- **Durable execution**: survives across steps and deployments
- **Streaming**: real-time updates to the UI
- **Cancellation**: user-initiated stop
- **Resume**: reconnect to active runs

Example workflow structure:
```typescript
// apps/web/app/api/workflows/agent/route.ts
import { createWorkflow } from '@vercel/workflow-sdk';

export const POST = createWorkflow(async (context, data) => {
  const { message, sessionId } = data;
  
  // Agent execution happens in workflow steps
  const result = await context.run('agent-step', async () => {
    return await runAgent(message, sessionId);
  });
  
  return result;
});
```

### Sandboxes

Sandboxes provide isolated execution environments with:
- **Filesystem access**: read, write, search files
- **Shell execution**: run commands
- **Git operations**: clone, branch, commit, push
- **Port exposure**: `3000`, `5173`, `4321`, `8000`
- **Snapshot-based resume**: hibernate and restore state

```typescript
// packages/sandbox/src/client.ts
import { createSandbox } from '@vercel-labs/open-agents/sandbox';

const sandbox = await createSandbox({
  baseSnapshotId: process.env.VERCEL_SANDBOX_BASE_SNAPSHOT_ID,
});

// Execute commands
const result = await sandbox.exec('npm install');

// Read files
const content = await sandbox.readFile('package.json');

// Write files
await sandbox.writeFile('src/config.ts', configContent);
```

### Tools

Agents interact with sandboxes through tools:

```typescript
// packages/agent/src/tools/file-edit.ts
export const fileEditTool = {
  name: 'file_edit',
  description: 'Edit a file with search-and-replace',
  parameters: z.object({
    path: z.string(),
    search: z.string(),
    replace: z.string(),
  }),
  execute: async ({ path, search, replace }, context) => {
    const content = await context.sandbox.readFile(path);
    const updated = content.replace(search, replace);
    await context.sandbox.writeFile(path, updated);
    return { success: true };
  },
};
```

Available tools:
- **file_read**: Read file contents
- **file_write**: Write entire file
- **file_edit**: Search-and-replace edits
- **file_search**: Search file contents
- **shell**: Execute shell commands
- **task**: Create subtasks for the agent
- **web**: Fetch web content
- **skill**: Access domain-specific skills

## Creating Custom Tools

1. **Define the tool schema**:
```typescript
// packages/agent/src/tools/my-tool.ts
import { z } from 'zod';

export const myCustomTool = {
  name: 'my_tool',
  description: 'Does something specific',
  parameters: z.object({
    input: z.string().describe('The input parameter'),
  }),
  execute: async ({ input }, context) => {
    // Access sandbox
    const result = await context.sandbox.exec(`echo ${input}`);
    
    // Return structured data
    return {
      output: result.stdout,
      exitCode: result.exitCode,
    };
  },
};
```

2. **Register the tool**:
```typescript
// packages/agent/src/tools/index.ts
import { myCustomTool } from './my-tool';

export const tools = [
  fileReadTool,
  fileWriteTool,
  shellTool,
  myCustomTool, // Add your tool
  // ...
];
```

## Skills System

Skills are domain-specific knowledge modules loaded into agent context:

```typescript
// packages/agent/src/skills/registry.ts
export interface Skill {
  id: string;
  name: string;
  description: string;
  content: string; // Markdown content
  triggers: string[];
}

// Skills are cached in Redis/KV or in-memory
const skillsCache = new SkillsCache({
  redis: process.env.REDIS_URL,
});

// Retrieve skill by trigger
const skill = await skillsCache.findByTrigger('nextjs app router');
```

Create a custom skill:
```typescript
// packages/agent/src/skills/custom-skill.ts
export const customSkill: Skill = {
  id: 'custom-framework',
  name: 'Custom Framework',
  description: 'Expertise in CustomFramework usage',
  content: `
# Custom Framework Skill

## Installation
\`\`\`bash
npm install custom-framework
\`\`\`

## Usage
\`\`\`typescript
import { Framework } from 'custom-framework';

const app = new Framework();
app.start();
\`\`\`
  `,
  triggers: [
    'use custom framework',
    'setup custom framework',
    'configure custom framework',
  ],
};
```

## GitHub Integration

### Auto-commit and PR Creation

Configure agent to automatically commit and create PRs:

```typescript
// packages/agent/src/workflows/agent-workflow.ts
export async function runAgentWorkflow(
  message: string,
  options: {
    autoCommit?: boolean;
    autoPR?: boolean;
    repository?: string;
    branch?: string;
  }
) {
  const result = await agent.run(message);
  
  if (options.autoCommit && result.success) {
    await sandbox.exec('git add .');
    await sandbox.exec(`git commit -m "Agent: ${message}"`);
    
    if (options.autoPR) {
      await createPullRequest({
        repo: options.repository,
        branch: options.branch,
        title: `AI Agent: ${message}`,
        body: result.summary,
      });
    }
  }
}
```

### GitHub App Installation

```typescript
// apps/web/lib/github/app.ts
import { App } from '@octokit/app';

const app = new App({
  appId: process.env.GITHUB_APP_ID,
  privateKey: process.env.GITHUB_APP_PRIVATE_KEY,
});

// Get installation token
const installation = await app.octokit.request(
  'GET /users/{username}/installation',
  { username: 'user' }
);

const token = await app.octokit.auth({
  type: 'installation',
  installationId: installation.data.id,
});
```

## Configuration

### Resource Profiles

For Hobby tier deployments:
```env
OPEN_AGENTS_RESOURCE_PROFILE=hobby
```

This adjusts chat and sandbox resource limits to Hobby-compatible values.

### Sandbox Base Snapshot

Use a preconfigured sandbox image:
```env
VERCEL_SANDBOX_BASE_SNAPSHOT_ID=your_snapshot_id
```

Create base snapshot:
```bash
bun run sandbox:snapshot-base
```

### Voice Input

Enable voice transcription with ElevenLabs:
```env
ELEVENLABS_API_KEY=your_api_key
```

### Optional Caching

Enable Redis/KV for skills metadata:
```env
REDIS_URL=your_redis_url
KV_URL=your_kv_url
```

## Common Patterns

### Multi-step Agent Task

```typescript
// packages/agent/src/agent.ts
export async function runMultiStepTask(goal: string) {
  const steps = await planSteps(goal);
  
  for (const step of steps) {
    const result = await executeStep(step, {
      tools: availableTools,
      sandbox: currentSandbox,
    });
    
    if (!result.success) {
      // Handle failure, retry, or replan
      return { success: false, error: result.error };
    }
  }
  
  return { success: true, steps };
}
```

### Streaming Agent Response

```typescript
// apps/web/app/api/chat/route.ts
import { streamText } from 'ai';

export async function POST(req: Request) {
  const { messages, sessionId } = await req.json();
  
  const result = await streamText({
    model: openai('gpt-4'),
    messages,
    tools: {
      file_read: fileReadTool,
      file_write: fileWriteTool,
      shell: shellTool,
    },
    onFinish: async ({ text, toolCalls }) => {
      // Save to session
      await saveMessage(sessionId, { text, toolCalls });
    },
  });
  
  return result.toDataStreamResponse();
}
```

### Session Management

```typescript
// apps/web/lib/sessions.ts
import { db } from './db';

export async function createSession(userId: string, metadata: object) {
  const session = await db.session.create({
    data: {
      userId,
      metadata,
      createdAt: new Date(),
    },
  });
  
  return session;
}

export async function shareSession(sessionId: string) {
  const shareToken = await generateShareToken(sessionId);
  
  return {
    url: `${process.env.NEXT_PUBLIC_VERCEL_PROJECT_PRODUCTION_URL}/share/${shareToken}`,
    token: shareToken,
  };
}
```

### Sandbox Lifecycle

```typescript
// packages/sandbox/src/lifecycle.ts
export class SandboxLifecycle {
  async create() {
    this.sandbox = await createSandbox({
      baseSnapshotId: this.baseSnapshotId,
    });
    return this.sandbox;
  }
  
  async hibernate() {
    const snapshot = await this.sandbox.createSnapshot();
    await this.sandbox.stop();
    return snapshot.id;
  }
  
  async resume(snapshotId: string) {
    this.sandbox = await createSandbox({
      baseSnapshotId: snapshotId,
    });
    return this.sandbox;
  }
  
  async destroy() {
    await this.sandbox.stop();
    this.sandbox = null;
  }
}
```

## Troubleshooting

### Workflow Execution Fails

**Problem**: Workflow steps timeout or fail
```typescript
// Check workflow logs
const logs = await getWorkflowLogs(workflowId);
console.log(logs);

// Increase timeout
export const POST = createWorkflow(
  async (context, data) => {
    // ...
  },
  { timeout: 300000 } // 5 minutes
);
```

### Sandbox Connection Issues

**Problem**: Cannot connect to sandbox
```typescript
// Verify sandbox is running
const status = await sandbox.status();
console.log('Sandbox status:', status);

// Recreate sandbox
await sandbox.stop();
const newSandbox = await createSandbox();
```

### GitHub App Permissions

**Problem**: Cannot access repositories
- Verify app installation: `https://github.com/settings/installations`
- Check app permissions include:
  - Repository contents: Read & write
  - Pull requests: Read & write
  - Metadata: Read-only

### Database Migrations

**Problem**: Schema out of sync
```bash
# Check migration status
bun run ci

# Apply migrations manually if needed
cd apps/web
npx drizzle-kit push:pg
```

### Authentication Issues

**Problem**: OAuth callback fails
- Verify callback URLs match exactly (including protocol)
- Check `BETTER_AUTH_SECRET` is set and consistent across deployments
- Confirm OAuth app credentials are correct

### Port Conflicts in Sandbox

**Problem**: Preview port already in use
```typescript
// Use alternative ports
const ports = [3000, 5173, 4321, 8000];
for (const port of ports) {
  try {
    await sandbox.exec(`lsof -ti:${port} | xargs kill -9`);
  } catch {
    // Port was free
  }
}
```

## Development Commands

```bash
# Start development server
bun run web

# Lint and format check
bun run check

# Auto-fix linting and formatting
bun run fix

# Type check all packages
bun run typecheck

# Full CI suite
bun run ci

# Refresh sandbox base snapshot
bun run sandbox:snapshot-base
```

## API Reference

### Core Types

```typescript
// Agent context passed to tools
interface AgentContext {
  sandbox: Sandbox;
  sessionId: string;
  userId: string;
  metadata: Record<string, unknown>;
}

// Tool definition
interface Tool<T = unknown> {
  name: string;
  description: string;
  parameters: z.ZodSchema<T>;
  execute: (params: T, context: AgentContext) => Promise<unknown>;
}

// Sandbox interface
interface Sandbox {
  exec(command: string): Promise<ExecResult>;
  readFile(path: string): Promise<string>;
  writeFile(path: string, content: string): Promise<void>;
  search(pattern: string): Promise<SearchResult[]>;
  createSnapshot(): Promise<Snapshot>;
  status(): Promise<SandboxStatus>;
  stop(): Promise<void>;
}
```

## Resources

- **Homepage**: https://open-agents.dev
- **Repository**: https://github.com/vercel-labs/open-agents
- **Vercel Workflow SDK**: https://vercel.com/docs/workflow
- **Better Auth**: https://www.better-auth.com/
- **GitHub Apps**: https://docs.github.com/en/apps
