---
name: agent-native-framework
description: Build agent-native applications with shared actions, SQL-backed state, tools, skills, and UI surfaces that work together.
triggers:
  - how do I build an agent-native app
  - create an agent native application with actions
  - set up agent-native framework with database
  - define actions that work with UI and agents
  - add agent skills to my coding assistant
  - build agents that act inside real apps
  - integrate agent runtime with SQL state
  - create shared actions for agents and UI
---

# Agent-Native Framework

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agent-Native is an open-source framework for building robust agents that act inside real apps, not just chat next to them. It provides primitives for product-grade agentic software: shared actions, SQL-backed state, identity, tools, skills, jobs, observability, and UI surfaces.

## Installation

### Create a New Project

```bash
npx @agent-native/core@latest create my-app
cd my-app
pnpm install
pnpm dev
```

The `create` command offers three starting points:
- **Full template(s)**: Clone complete apps (Mail, Calendar, Forms, Clips, etc.)
- **Chat**: Minimal chat UI with browser shell
- **Headless**: Action-first app with no UI shell

Use flags to skip the prompt:
```bash
npx @agent-native/core@latest create my-app --template mail
npx @agent-native/core@latest create my-app --headless
npx @agent-native/core@latest create my-app --standalone
```

### Add Skills to Existing Coding Agent

Install visual planning and PR recap skills:

```bash
npx @agent-native/core@latest skills add visual-plan
```

This adds `/visual-plan` and `/visual-recap` slash commands to Claude Code, Cursor, Codex, Pi, OpenCode, GitHub Copilot, and similar agents.

## Core Concepts

### Actions

Actions are the fundamental building block. Define work once, use it everywhere: UI, agent, HTTP API, MCP, A2A, and CLI.

```typescript
// actions/reply-to-email.ts
import { defineAction } from '@agent-native/core';
import { z } from 'zod';
import { db } from '~/db';
import { replies } from '~/db/schema';

export default defineAction({
  schema: z.object({
    emailId: z.string(),
    body: z.string(),
  }),
  run: async ({ emailId, body }) => {
    await db.insert(replies).values({ emailId, body });
  },
});
```

### Calling Actions

**From UI:**
```typescript
import { useAction } from '@agent-native/react';
import replyToEmail from '~/actions/reply-to-email';

function ReplyButton({ emailId }: { emailId: string }) {
  const { execute, loading } = useAction(replyToEmail);
  
  return (
    <button 
      onClick={() => execute({ emailId, body: 'Thanks for reaching out!' })}
      disabled={loading}
    >
      Send Reply
    </button>
  );
}
```

**From Agent:**
```typescript
// The agent automatically gets access to all defined actions
// and can call them based on user intent
```

**From CLI:**
```bash
pnpm agent-native action reply-to-email --emailId "123" --body "Hello"
```

**From HTTP API:**
```bash
curl -X POST http://localhost:3000/api/actions/reply-to-email \
  -H "Content-Type: application/json" \
  -d '{"emailId": "123", "body": "Hello"}'
```

## Agent Configuration

### Basic Agent Setup

```typescript
// agent/config.ts
import { defineAgent } from '@agent-native/core';

export default defineAgent({
  name: 'my-assistant',
  model: 'gpt-4',
  systemPrompt: `You are a helpful assistant that helps users manage their emails.
You have access to actions like reply-to-email, archive-email, and search-emails.`,
  tools: [
    // Actions are automatically exposed as tools
  ],
  memory: {
    // SQL-backed conversation memory
    enabled: true,
  },
});
```

### Agent with Custom Tools

```typescript
import { defineAgent, defineTool } from '@agent-native/core';
import { z } from 'zod';

const searchWeb = defineTool({
  name: 'search-web',
  description: 'Search the web for information',
  schema: z.object({
    query: z.string(),
  }),
  execute: async ({ query }) => {
    // Integration with search API
    const response = await fetch(
      `https://api.search.com/search?q=${encodeURIComponent(query)}`,
      {
        headers: { 'Authorization': `Bearer ${process.env.SEARCH_API_KEY}` }
      }
    );
    return response.json();
  },
});

export default defineAgent({
  name: 'research-assistant',
  model: 'gpt-4-turbo',
  tools: [searchWeb],
});
```

## Database Setup

Agent-Native uses Drizzle ORM with any SQL database (PostgreSQL, MySQL, SQLite).

### Schema Definition

```typescript
// db/schema.ts
import { pgTable, serial, text, timestamp, varchar } from 'drizzle-orm/pg-core';

export const emails = pgTable('emails', {
  id: serial('id').primaryKey(),
  subject: text('subject').notNull(),
  body: text('body').notNull(),
  from: varchar('from', { length: 255 }).notNull(),
  to: varchar('to', { length: 255 }).notNull(),
  createdAt: timestamp('created_at').defaultNow(),
});

export const replies = pgTable('replies', {
  id: serial('id').primaryKey(),
  emailId: serial('email_id').references(() => emails.id),
  body: text('body').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
});
```

### Database Connection

```typescript
// db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const connectionString = process.env.DATABASE_URL;

if (!connectionString) {
  throw new Error('DATABASE_URL is not set');
}

const client = postgres(connectionString);
export const db = drizzle(client, { schema });
```

### Environment Variables

```env
# .env
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
OPENAI_API_KEY=your-key-here
```

## Advanced Action Patterns

### Action with Database Query

```typescript
import { defineAction } from '@agent-native/core';
import { z } from 'zod';
import { db } from '~/db';
import { emails } from '~/db/schema';
import { eq } from 'drizzle-orm';

export default defineAction({
  name: 'get-email',
  schema: z.object({
    id: z.string(),
  }),
  run: async ({ id }) => {
    const email = await db
      .select()
      .from(emails)
      .where(eq(emails.id, parseInt(id)))
      .limit(1);
    
    return email[0] || null;
  },
});
```

### Action with Side Effects

```typescript
import { defineAction } from '@agent-native/core';
import { z } from 'zod';
import { db } from '~/db';
import { emails } from '~/db/schema';

export default defineAction({
  name: 'send-email',
  schema: z.object({
    to: z.string().email(),
    subject: z.string(),
    body: z.string(),
  }),
  run: async ({ to, subject, body }) => {
    // Store in database
    const [email] = await db
      .insert(emails)
      .values({ to, subject, body, from: 'me@example.com' })
      .returning();
    
    // Send via email service
    await fetch('https://api.emailservice.com/send', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.EMAIL_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ to, subject, body }),
    });
    
    return email;
  },
});
```

### Action with Validation and Error Handling

```typescript
import { defineAction } from '@agent-native/core';
import { z } from 'zod';

export default defineAction({
  name: 'process-payment',
  schema: z.object({
    amount: z.number().positive(),
    currency: z.enum(['USD', 'EUR', 'GBP']),
    customerId: z.string(),
  }),
  run: async ({ amount, currency, customerId }) => {
    try {
      const response = await fetch('https://api.payment.com/charge', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${process.env.PAYMENT_API_KEY}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ amount, currency, customer: customerId }),
      });
      
      if (!response.ok) {
        throw new Error(`Payment failed: ${response.statusText}`);
      }
      
      return response.json();
    } catch (error) {
      console.error('Payment error:', error);
      throw error;
    }
  },
});
```

## Skills System

### Installing Skills

```bash
# Install a specific skill
npx @agent-native/core@latest skills add visual-plan

# List available skills
npx @agent-native/core@latest skills list

# Remove a skill
npx @agent-native/core@latest skills remove visual-plan
```

### Using Visual Plan Skill

After installing `visual-plan`, use these slash commands in your coding agent:

- **`/visual-plan`**: Generate a structured plan with diagrams, wireframes, and file-by-file implementation maps before coding
- **`/visual-recap`**: Generate a visual recap of changes with a shareable review link after PR or git diff

### Creating Custom Skills

```typescript
// skills/custom-skill.ts
import { defineSkill } from '@agent-native/core';

export default defineSkill({
  name: 'custom-analyzer',
  description: 'Analyzes code patterns and suggests improvements',
  commands: [
    {
      name: 'analyze',
      description: 'Analyze current file for patterns',
      execute: async (context) => {
        const currentFile = context.getCurrentFile();
        // Analysis logic here
        return {
          suggestions: ['Use const instead of let', 'Extract this function'],
        };
      },
    },
  ],
});
```

## React Integration

### Setup React App

```typescript
// app/root.tsx
import { AgentProvider } from '@agent-native/react';
import { Outlet } from '@remix-run/react';

export default function Root() {
  return (
    <AgentProvider>
      <Outlet />
    </AgentProvider>
  );
}
```

### Using Agent in Components

```typescript
import { useAgent, useAction } from '@agent-native/react';
import sendEmail from '~/actions/send-email';

function EmailComposer() {
  const { chat, messages, isLoading } = useAgent();
  const { execute: send } = useAction(sendEmail);
  
  const handleSend = async () => {
    await send({
      to: 'user@example.com',
      subject: 'Hello',
      body: 'Message body',
    });
  };
  
  const askAgent = async (prompt: string) => {
    await chat(prompt);
  };
  
  return (
    <div>
      <button onClick={handleSend}>Send Email</button>
      <button onClick={() => askAgent('Draft a follow-up email')}>
        Ask Agent to Draft
      </button>
      
      <div>
        {messages.map((msg, i) => (
          <div key={i}>{msg.content}</div>
        ))}
      </div>
    </div>
  );
}
```

### Real-time State Sync

```typescript
import { useRealtimeState } from '@agent-native/react';
import { emails } from '~/db/schema';

function EmailList() {
  // Automatically syncs with database and agent changes
  const { data: emailList, mutate } = useRealtimeState(emails);
  
  return (
    <ul>
      {emailList?.map(email => (
        <li key={email.id}>{email.subject}</li>
      ))}
    </ul>
  );
}
```

## Jobs and Background Tasks

```typescript
// jobs/process-attachments.ts
import { defineJob } from '@agent-native/core';
import { z } from 'zod';

export default defineJob({
  name: 'process-attachments',
  schedule: 'every 5 minutes', // or cron: '*/5 * * * *'
  schema: z.object({
    emailId: z.string(),
  }),
  run: async ({ emailId }) => {
    // Process attachments for email
    console.log(`Processing attachments for email ${emailId}`);
  },
});
```

### Triggering Jobs from Actions

```typescript
import { defineAction, scheduleJob } from '@agent-native/core';
import { z } from 'zod';
import processAttachments from '~/jobs/process-attachments';

export default defineAction({
  name: 'receive-email',
  schema: z.object({
    emailId: z.string(),
  }),
  run: async ({ emailId }) => {
    // Schedule background job
    await scheduleJob(processAttachments, { emailId });
  },
});
```

## Observability

### Logging

```typescript
import { defineAction, log } from '@agent-native/core';

export default defineAction({
  name: 'complex-action',
  schema: z.object({ data: z.string() }),
  run: async ({ data }) => {
    log.info('Starting complex action', { data });
    
    try {
      // Work here
      log.debug('Processing step 1');
      log.debug('Processing step 2');
      
      return { success: true };
    } catch (error) {
      log.error('Action failed', { error, data });
      throw error;
    }
  },
});
```

### Tracing

```typescript
import { defineAgent, trace } from '@agent-native/core';

export default defineAgent({
  name: 'traced-agent',
  onMessage: async (message, context) => {
    const span = trace.startSpan('agent-message');
    
    try {
      // Agent logic
      span.setAttributes({ messageLength: message.length });
      return await context.complete(message);
    } finally {
      span.end();
    }
  },
});
```

## MCP (Model Context Protocol) Integration

Agent-Native actions automatically expose as MCP tools:

```typescript
// Automatically available via MCP server
// No additional configuration needed
```

### A2A (Agent-to-Agent) Communication

```typescript
import { defineAgent, callAgent } from '@agent-native/core';

export default defineAgent({
  name: 'coordinator',
  onMessage: async (message, context) => {
    // Call another agent
    const result = await callAgent('specialist-agent', {
      task: 'analyze',
      data: message,
    });
    
    return `Coordinated result: ${result}`;
  },
});
```

## Deployment

### Environment Setup

```env
# Production environment variables
DATABASE_URL=postgresql://prod-user:password@prod-host:5432/prod-db
OPENAI_API_KEY=sk-prod-key
NODE_ENV=production
PORT=3000
```

### Build and Deploy

```bash
# Build the application
pnpm build

# Start production server
pnpm start
```

Agent-Native works with Nitro-compatible hosting providers:
- Vercel
- Netlify
- AWS Lambda
- Cloudflare Workers
- Node.js servers

## Troubleshooting

### Action Not Found

**Problem:** Agent can't find defined action.

**Solution:** Ensure action is exported as default and file is in `actions/` directory:
```typescript
// actions/my-action.ts
export default defineAction({ /* ... */ });
```

### Database Connection Issues

**Problem:** `DATABASE_URL is not set` error.

**Solution:** Check `.env` file exists and is loaded:
```bash
# Verify environment variable
echo $DATABASE_URL

# Create .env if missing
cp .env.example .env
```

### Agent Not Responding

**Problem:** Agent doesn't call actions or respond to prompts.

**Solution:** 
1. Check API key is set: `OPENAI_API_KEY` or relevant model provider key
2. Verify system prompt includes action descriptions
3. Check action schemas are valid Zod schemas

### Type Errors with Actions

**Problem:** TypeScript errors when using actions.

**Solution:** Regenerate types after schema changes:
```bash
pnpm generate
```

### Real-time State Not Updating

**Problem:** UI doesn't reflect agent changes.

**Solution:** Ensure `useRealtimeState` is used and WebSocket connection is active:
```typescript
const { data, connected } = useRealtimeState(table);
console.log('Connected:', connected); // Should be true
```

## Common Patterns

### Multi-step Agent Workflow

```typescript
import { defineAgent } from '@agent-native/core';
import searchEmails from '~/actions/search-emails';
import replyToEmail from '~/actions/reply-to-email';

export default defineAgent({
  name: 'email-assistant',
  systemPrompt: `You help manage emails. Follow these steps:
1. Search for relevant emails using search-emails
2. Read the content and understand context
3. Draft appropriate replies using reply-to-email`,
  tools: [searchEmails, replyToEmail],
});
```

### Shared State Between UI and Agent

```typescript
// Action modifies state
export const updateDocument = defineAction({
  schema: z.object({ id: z.string(), content: z.string() }),
  run: async ({ id, content }) => {
    return await db.update(documents)
      .set({ content })
      .where(eq(documents.id, id))
      .returning();
  },
});

// UI component observes same state
function DocumentEditor({ id }: { id: string }) {
  const { data: doc } = useRealtimeState(documents, eq(documents.id, id));
  const { execute: update } = useAction(updateDocument);
  
  // Both user edits and agent edits show in real-time
  return <Editor value={doc?.content} onChange={(val) => update({ id, content: val })} />;
}
```

### Authentication and Authorization

```typescript
import { defineAction } from '@agent-native/core';
import { z } from 'zod';

export default defineAction({
  name: 'delete-email',
  schema: z.object({ id: z.string() }),
  auth: true, // Requires authentication
  run: async ({ id }, context) => {
    // Check user owns this email
    if (!context.user) {
      throw new Error('Unauthorized');
    }
    
    const email = await db.query.emails.findFirst({
      where: eq(emails.id, id),
    });
    
    if (email?.userId !== context.user.id) {
      throw new Error('Forbidden');
    }
    
    await db.delete(emails).where(eq(emails.id, id));
  },
});
```
