---
name: cloudflare-agentic-inbox
description: Deploy and manage a self-hosted email client with AI agent on Cloudflare Workers
triggers:
  - set up agentic inbox on cloudflare
  - configure email routing with ai agent
  - deploy self-hosted email client
  - create cloudflare email worker
  - build email client with workers ai
  - set up durable objects for email
  - configure cloudflare access for inbox
  - troubleshoot agentic inbox deployment
---

# Cloudflare Agentic Inbox

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agentic Inbox is a self-hosted email client with an AI agent, running entirely on Cloudflare Workers. It uses Email Routing for receiving emails, Durable Objects with SQLite for per-mailbox storage, R2 for attachments, and Workers AI with the Cloudflare Agents SDK for AI-powered email assistance.

## Installation & Deployment

### Quick Deploy (Recommended)

1. **Deploy via button** (provisions R2, Durable Objects, Workers AI automatically):
   ```bash
   # Visit: https://deploy.workers.cloudflare.com/?url=https://github.com/cloudflare/agentic-inbox
   # When prompted, enter your domain: yourdomain.com
   ```

2. **Configure Cloudflare Access** (required for production):
   - Navigate to Worker Settings → Domains & Routes
   - Enable one-click Cloudflare Access
   - Note the `POLICY_AUD` and `TEAM_DOMAIN` values
   - Set as Worker secrets:
   ```bash
   wrangler secret put POLICY_AUD
   wrangler secret put TEAM_DOMAIN
   ```

3. **Set up Email Routing**:
   - Go to your domain in Cloudflare dashboard
   - Navigate to Email Routing
   - Create a catch-all rule forwarding to this Worker

4. **Enable Email Service**:
   - Add `send_email` binding to `wrangler.jsonc`:
   ```jsonc
   {
     "send_email": [
       {
         "name": "SEB",
         "destination_address": "you@example.com"
       }
     ]
   }
   ```

### Manual Setup

```bash
# Clone repository
git clone https://github.com/cloudflare/agentic-inbox.git
cd agentic-inbox

# Install dependencies
npm install

# Create R2 bucket
wrangler r2 bucket create agentic-inbox

# Configure domain in wrangler.jsonc
# Set DOMAINS variable to your domain

# Deploy
npm run deploy
```

## Configuration

### wrangler.jsonc Structure

```jsonc
{
  "name": "agentic-inbox",
  "main": "worker/index.ts",
  "compatibility_date": "2025-01-01",
  "compatibility_flags": ["nodejs_compat"],
  "vars": {
    "DOMAINS": "yourdomain.com"
  },
  "durable_objects": {
    "bindings": [
      {
        "name": "MAILBOX",
        "class_name": "MailboxDurableObject",
        "script_name": "agentic-inbox"
      },
      {
        "name": "EMAIL_AGENT",
        "class_name": "EmailAgentDurableObject",
        "script_name": "agentic-inbox"
      }
    ]
  },
  "r2_buckets": [
    {
      "binding": "R2",
      "bucket_name": "agentic-inbox"
    }
  ],
  "ai": {
    "binding": "AI"
  },
  "send_email": [
    {
      "name": "SEB",
      "destination_address": "fallback@yourdomain.com"
    }
  ]
}
```

### Environment Variables (Secrets)

```bash
# Required for Cloudflare Access authentication
wrangler secret put POLICY_AUD
wrangler secret put TEAM_DOMAIN

# TEAM_DOMAIN can be either:
# - Your Access team URL: yourteam.cloudflareaccess.com
# - Full certs URL: yourteam.cloudflareaccess.com/cdn-cgi/access/certs
```

## Development

### Local Development

```bash
# Start dev server with hot reload
npm run dev

# Access at http://localhost:8787
# Note: Cloudflare Access is disabled in local development
```

### Project Structure

```
agentic-inbox/
├── app/                    # React frontend
│   ├── routes/             # React Router v7 routes
│   ├── components/         # UI components
│   └── lib/                # Utilities, stores (Zustand)
├── worker/                 # Cloudflare Worker backend
│   ├── index.ts            # Hono router, email handler
│   ├── mailbox-do.ts       # Mailbox Durable Object
│   ├── email-agent-do.ts   # AI Agent Durable Object
│   └── auth.ts             # Access JWT validation
└── wrangler.jsonc          # Cloudflare configuration
```

## Key API Patterns

### Creating a Mailbox

```typescript
// POST /api/mailboxes
const response = await fetch('/api/mailboxes', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    address: 'hello@yourdomain.com'
  })
});

const mailbox = await response.json();
// { id: "uuid", address: "hello@yourdomain.com", createdAt: "..." }
```

### Sending Email

```typescript
// POST /api/mailboxes/:id/send
const response = await fetch(`/api/mailboxes/${mailboxId}/send`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    to: ['recipient@example.com'],
    subject: 'Hello',
    body: '<p>Email content</p>',
    cc: [],
    bcc: [],
    inReplyTo: null,
    references: []
  })
});
```

### Accessing AI Agent

```typescript
// WebSocket connection to agent
const ws = new WebSocket(`wss://yourapp.workers.dev/agents/${mailboxId}`);

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'message',
    content: 'Summarize my unread emails'
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  // Stream: { type: 'text-delta', text: '...' }
  // Tools: { type: 'tool-call', toolName: 'read_inbox', args: {...} }
  // Result: { type: 'tool-result', result: {...} }
};
```

## Durable Object Implementation

### Mailbox Durable Object

```typescript
import { DurableObject } from 'cloudflare:workers';

export class MailboxDurableObject extends DurableObject {
  async fetch(request: Request) {
    const url = new URL(request.url);
    
    if (url.pathname === '/emails' && request.method === 'GET') {
      const stmt = this.ctx.storage.sql.exec(
        'SELECT * FROM emails ORDER BY receivedAt DESC LIMIT 50'
      );
      return Response.json(stmt.toArray());
    }
    
    if (url.pathname === '/emails' && request.method === 'POST') {
      const email = await request.json();
      const result = this.ctx.storage.sql.exec(
        `INSERT INTO emails (id, subject, from_address, to_address, body, receivedAt)
         VALUES (?, ?, ?, ?, ?, ?)`,
        email.id, email.subject, email.from, email.to, email.body, Date.now()
      );
      return Response.json({ success: true });
    }
    
    return new Response('Not found', { status: 404 });
  }
}
```

### Email Agent Durable Object

```typescript
import { AIChatAgent } from '@cloudflare/agents-sdk';
import { DurableObject } from 'cloudflare:workers';

export class EmailAgentDurableObject extends DurableObject {
  private agent?: AIChatAgent;
  
  async fetch(request: Request) {
    if (!this.agent) {
      this.agent = new AIChatAgent({
        model: '@cf/moonshotai/kimi-k2.5',
        binding: this.env.AI,
        tools: [
          {
            name: 'read_inbox',
            description: 'Read emails from the inbox',
            parameters: {
              type: 'object',
              properties: {
                limit: { type: 'number', default: 10 }
              }
            },
            handler: async ({ limit }) => {
              // Fetch from mailbox DO
              const mailboxId = this.ctx.id.toString();
              const emails = await this.fetchMailboxEmails(mailboxId, limit);
              return { emails };
            }
          },
          {
            name: 'send_email',
            description: 'Send an email',
            parameters: {
              type: 'object',
              properties: {
                to: { type: 'array', items: { type: 'string' } },
                subject: { type: 'string' },
                body: { type: 'string' }
              },
              required: ['to', 'subject', 'body']
            },
            handler: async ({ to, subject, body }) => {
              // Send via Email Service
              await this.env.SEB.send({
                from: this.getMailboxAddress(),
                to,
                subject,
                content: [{ type: 'text/html', value: body }]
              });
              return { success: true };
            }
          }
        ],
        systemPrompt: 'You are an email assistant...'
      });
    }
    
    // Handle WebSocket upgrade for streaming
    const upgradeHeader = request.headers.get('Upgrade');
    if (upgradeHeader === 'websocket') {
      const [client, server] = Object.values(new WebSocketPair());
      this.ctx.acceptWebSocket(server);
      return new Response(null, { status: 101, webSocket: client });
    }
    
    return new Response('Expected WebSocket', { status: 400 });
  }
  
  async webSocketMessage(ws: WebSocket, message: string) {
    const { content } = JSON.parse(message);
    
    for await (const chunk of this.agent.stream(content)) {
      ws.send(JSON.stringify(chunk));
    }
  }
}
```

## Email Routing Handler

```typescript
// worker/index.ts
import { EmailMessage } from 'cloudflare:email';

export default {
  async email(message: EmailMessage, env: Env) {
    const to = message.to;
    const mailboxId = await env.KV.get(`address:${to}`);
    
    if (!mailboxId) {
      message.setReject('Mailbox not found');
      return;
    }
    
    // Forward to Mailbox DO
    const id = env.MAILBOX.idFromString(mailboxId);
    const stub = env.MAILBOX.get(id);
    
    const emailData = {
      id: crypto.randomUUID(),
      from: message.from,
      to: message.to,
      subject: message.headers.get('subject'),
      body: await message.text(),
      receivedAt: Date.now()
    };
    
    await stub.fetch('https://mailbox/emails', {
      method: 'POST',
      body: JSON.stringify(emailData)
    });
  }
};
```

## Common Patterns

### Access Authentication Middleware

```typescript
// worker/auth.ts
import * as jose from 'jose';

export async function validateAccessToken(request: Request, env: Env) {
  if (!env.POLICY_AUD || !env.TEAM_DOMAIN) {
    throw new Error('Cloudflare Access must be configured in production');
  }
  
  const token = request.headers.get('Cf-Access-Jwt-Assertion');
  if (!token) {
    throw new Error('Missing Access token');
  }
  
  const certsUrl = env.TEAM_DOMAIN.includes('/cdn-cgi/access/certs')
    ? env.TEAM_DOMAIN
    : `https://${env.TEAM_DOMAIN}/cdn-cgi/access/certs`;
  
  const jwks = jose.createRemoteJWKSet(new URL(certsUrl));
  
  const { payload } = await jose.jwtVerify(token, jwks, {
    audience: env.POLICY_AUD,
    issuer: env.TEAM_DOMAIN
  });
  
  return payload;
}
```

### Agent System Prompt Customization

```typescript
// Update system prompt per mailbox
const response = await fetch(`/api/mailboxes/${mailboxId}/agent/system-prompt`, {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    systemPrompt: `You are a professional email assistant for sales@company.com.
    Always be polite and concise. When drafting replies, maintain a friendly tone.`
  })
});
```

### Attachment Storage in R2

```typescript
// Store attachment in R2
async function storeAttachment(env: Env, emailId: string, file: File) {
  const key = `attachments/${emailId}/${file.name}`;
  await env.R2.put(key, file.stream(), {
    httpMetadata: {
      contentType: file.type
    }
  });
  return key;
}

// Retrieve attachment
async function getAttachment(env: Env, key: string) {
  const object = await env.R2.get(key);
  if (!object) return null;
  
  return new Response(object.body, {
    headers: {
      'Content-Type': object.httpMetadata?.contentType || 'application/octet-stream'
    }
  });
}
```

## Troubleshooting

### Invalid or Expired Access Token

**Issue**: `Invalid or expired Access token` error when accessing deployed app.

**Solution**:
```bash
# 1. Turn Access off and back on in Worker Settings → Domains & Routes
# 2. Copy new POLICY_AUD and TEAM_DOMAIN values from modal
# 3. Reset secrets
wrangler secret put POLICY_AUD
# Paste new value
wrangler secret put TEAM_DOMAIN
# Paste new value (can be team URL or full certs URL)

# 4. Redeploy
npm run deploy
```

### Emails Not Arriving

**Issue**: Catch-all rule configured but emails not appearing in inbox.

**Checklist**:
1. Verify Email Routing is enabled for your domain
2. Check catch-all rule forwards to the Worker (not an email address)
3. Confirm mailbox exists for the recipient address:
   ```bash
   # Check via API
   curl https://yourapp.workers.dev/api/mailboxes
   ```
4. Check Worker logs:
   ```bash
   wrangler tail
   ```

### Agent Not Responding

**Issue**: WebSocket connection fails or agent doesn't respond.

**Solution**:
```typescript
// Verify Workers AI binding in wrangler.jsonc
{
  "ai": {
    "binding": "AI"
  }
}

// Check agent initialization
const ws = new WebSocket(`wss://yourapp.workers.dev/agents/${mailboxId}`);
ws.onerror = (error) => console.error('WebSocket error:', error);
ws.onclose = (event) => console.log('Closed:', event.code, event.reason);
```

### R2 Bucket Not Found

**Issue**: Attachment upload fails with R2 error.

**Solution**:
```bash
# Create bucket if missing
wrangler r2 bucket create agentic-inbox

# Verify binding in wrangler.jsonc
{
  "r2_buckets": [
    {
      "binding": "R2",
      "bucket_name": "agentic-inbox"
    }
  ]
}

# Redeploy
npm run deploy
```

### Send Email Not Working

**Issue**: `send_email` binding not available or emails not sending.

**Solution**:
1. Enable Email Service in Cloudflare dashboard for your account
2. Add binding to `wrangler.jsonc`:
   ```jsonc
   {
     "send_email": [
       {
         "name": "SEB",
         "destination_address": "verified@yourdomain.com"
       }
     ]
   }
   ```
3. Verify domain ownership for sending
4. Redeploy Worker

### Local Development Access Errors

**Issue**: Access errors when running `npm run dev`.

**Note**: Cloudflare Access is intentionally disabled in local development. If you see Access-related errors in production mode locally, set:

```typescript
// worker/auth.ts - for local testing only
if (env.ENVIRONMENT === 'development') {
  return { email: 'dev@localhost' }; // Skip Access validation
}
```

## CLI Commands

```bash
# Development
npm run dev              # Start local dev server
npm run build            # Build frontend and worker
npm run deploy           # Deploy to Cloudflare

# Wrangler commands
wrangler tail            # Stream Worker logs
wrangler secret put KEY  # Set environment secret
wrangler r2 bucket list  # List R2 buckets
wrangler d1 execute DB --command "SELECT * FROM emails" # Query D1 (if using D1)

# Debugging
wrangler dev --remote    # Debug against production resources
wrangler kv:key list --binding=KV  # List KV keys (if using KV)
```

## MCP Server Integration

Agentic Inbox includes an MCP server at `/mcp` for external AI tools (Claude Code, Cursor):

```json
// claude_desktop_config.json or similar
{
  "mcpServers": {
    "agentic-inbox": {
      "url": "https://yourapp.workers.dev/mcp",
      "headers": {
        "Cf-Access-Client-Id": "YOUR_SERVICE_TOKEN_ID",
        "Cf-Access-Client-Secret": "YOUR_SERVICE_TOKEN_SECRET"
      }
    }
  }
}
```

MCP tools have access to all mailboxes by passing `mailboxId` parameter. Security relies on the Cloudflare Access policy.
