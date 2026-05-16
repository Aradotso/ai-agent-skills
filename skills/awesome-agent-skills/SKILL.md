---
name: awesome-agent-skills
description: Curated collection of 1000+ agent skills compatible with Claude Code, Codex, Gemini CLI, Cursor, and more
triggers:
  - find agent skills for my AI coding assistant
  - what agent skills are available for Claude Code
  - show me skills for Cursor AI
  - how do I add skills to my AI agent
  - browse the awesome agent skills collection
  - what official skills exist for Anthropic Claude
  - install agent skills from the community
  - search for AI agent capabilities and tools
---

# Awesome Agent Skills

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A curated collection of 1000+ agent skills from official development teams and the community. Unlike bulk-generated repositories, this collection focuses on real-world agent skills created and used by actual engineering teams from Anthropic, Google Labs, Vercel, Stripe, Cloudflare, Netlify, and more.

Compatible with Claude Code, Codex, Antigravity, Gemini CLI, Cursor, GitHub Copilot, OpenCode, Windsurf, and other AI coding agents.

## What are Agent Skills?

Agent skills are SKILL.md files that extend AI coding agents' capabilities by providing:

- Domain-specific knowledge and best practices
- API reference documentation
- Code patterns and examples
- Tool-specific workflows
- Integration guidelines

When installed, these skills give AI agents expertise in specific frameworks, tools, or domains.

## Installation Paths by Agent

Different AI coding agents look for skills in different locations:

| Agent | Skill Path | Documentation |
|-------|-----------|---------------|
| **Claude Code** | `.claude/skills/` | [Docs](https://docs.anthropic.com/claude/docs/skills) |
| **Codex** | `.codex/skills/` | [Docs](https://codex.ai/docs/skills) |
| **Antigravity** | `.antigravity/skills/` | [Docs](https://antigravity.dev/docs) |
| **Gemini CLI** | `.gemini/skills/` | [Docs](https://gemini.google.com/cli) |
| **Cursor** | `.cursor/skills/` | [Docs](https://cursor.sh/docs) |
| **Windsurf** | `.windsurf/skills/` | [Docs](https://windsurf.ai/docs) |

## Installing Skills

### Method 1: Manual Installation

1. Create the skills directory for your agent (e.g., `.claude/skills/`)
2. Download the SKILL.md file from the repository
3. Place it in your skills directory

```bash
# Example for Claude Code
mkdir -p .claude/skills
cd .claude/skills
curl -O https://raw.githubusercontent.com/VoltAgent/awesome-agent-skills/main/skills/stripe/stripe-best-practices.md
```

### Method 2: Using the officialskills.sh Website

Visit [officialskills.sh](https://officialskills.sh/) to:
- Browse skills by category
- Search for specific skills
- Copy skill URLs for direct installation
- View skill metadata and compatibility

### Method 3: Clone Specific Skills

```bash
# Clone the entire repository
git clone https://github.com/VoltAgent/awesome-agent-skills.git

# Copy specific skills to your agent's directory
cp awesome-agent-skills/skills/voltagent/create-voltagent.md .claude/skills/
```

## Skill Categories

### Official Anthropic Claude Skills

Core skills published by Anthropic for Claude:

```bash
# Document creation and editing
.claude/skills/docx.md          # Word documents
.claude/skills/pptx.md          # PowerPoint presentations
.claude/skills/xlsx.md          # Excel spreadsheets
.claude/skills/pdf.md           # PDF manipulation

# Design and development
.claude/skills/canvas-design.md        # Visual design
.claude/skills/frontend-design.md      # UI/UX development
.claude/skills/web-artifacts-builder.md # React + Tailwind artifacts
.claude/skills/algorithmic-art.md      # p5.js generative art

# Development tools
.claude/skills/mcp-builder.md         # MCP server creation
.claude/skills/webapp-testing.md      # Playwright testing
.claude/skills/skill-creator.md       # Create new skills
```

### Framework & Platform Skills

#### VoltAgent (TypeScript Framework)

```typescript
// Skills available:
// - create-voltagent: Project setup
// - voltagent-best-practices: Architecture patterns
// - voltagent-core-reference: API reference
// - voltagent-docs-bundle: Embedded documentation

// Example usage after installing skills
import { VoltAgent } from '@voltagent/core';

const agent = new VoltAgent({
  name: 'my-agent',
  model: 'claude-3-5-sonnet-20241022',
  apiKey: process.env.ANTHROPIC_API_KEY,
});
```

#### Angular

```typescript
// Skills: angular-developer, angular-new-app

// Generate component with installed skills
// ng generate component my-component --standalone
```

#### React Native (CallStack)

```javascript
// Skills: react-native-best-practices, upgrading-react-native

// Performance optimization patterns from Callstack skill
import { memo, useCallback } from 'react';

const OptimizedComponent = memo(({ data }) => {
  const handlePress = useCallback(() => {
    // Handle press
  }, []);
  
  return <TouchableOpacity onPress={handlePress}>...</TouchableOpacity>;
});
```

### Authentication & Database Skills

#### Better Auth

```typescript
// Skills: best-practices, create-auth, emailAndPassword, twoFactor, organization

import { betterAuth } from 'better-auth';

const auth = betterAuth({
  database: {
    provider: 'pg',
    url: process.env.DATABASE_URL,
  },
  emailAndPassword: {
    enabled: true,
  },
  twoFactor: {
    enabled: true,
  },
});
```

#### Supabase

```typescript
// Skills: postgres-best-practices

import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY
);

// Use PostgreSQL best practices from skill
const { data, error } = await supabase
  .from('users')
  .select('*')
  .eq('active', true);
```

### API & Integration Skills

#### Stripe

```typescript
// Skills: stripe-best-practices, upgrade-stripe

import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: '2024-11-20.acacia',
});

// Create payment intent following best practices
const paymentIntent = await stripe.paymentIntents.create({
  amount: 2000,
  currency: 'usd',
  automatic_payment_methods: { enabled: true },
});
```

#### Composio

```typescript
// Skills: composio (1000+ app integrations)

import { Composio } from 'composio-core';

const composio = new Composio(process.env.COMPOSIO_API_KEY);

// Connect to external apps with managed auth
const connection = await composio.getConnection('github');
```

#### Courier

```typescript
// Skills: courier-skills (multi-channel notifications)

import { CourierClient } from '@trycourier/courier';

const courier = new CourierClient({
  authorizationToken: process.env.COURIER_AUTH_TOKEN,
});

await courier.send({
  message: {
    to: { email: 'user@example.com' },
    content: { title: 'Welcome!', body: 'Thanks for signing up' },
    routing: { method: 'all', channels: ['email', 'sms'] },
  },
});
```

### Data & Analytics Skills

#### Tinybird

```sql
-- Skills: tinybird-best-practices, tinybird-cli-guidelines, 
--         tinybird-python-sdk-guidelines, tinybird-typescript-sdk-guidelines

-- Create a datasource (from skill patterns)
DESCRIPTION >
    User events datasource

SCHEMA >
    `timestamp` DateTime,
    `user_id` String,
    `event_type` String,
    `properties` String

ENGINE "MergeTree"
ENGINE_PARTITION_KEY "toYYYYMM(timestamp)"
ENGINE_SORTING_KEY "timestamp, user_id"
```

#### ClickHouse

```sql
-- Skills: clickhouse

-- Optimized table creation from skill
CREATE TABLE events (
    timestamp DateTime,
    user_id UInt64,
    event_type String,
    properties String
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp, user_id);
```

### Cloud & Infrastructure Skills

#### Google Gemini

```python
# Skills: gemini-api-dev, vertex-ai-api-dev, gemini-live-api-dev, gemini-interactions-api

import google.generativeai as genai

genai.configure(api_key=os.environ["GEMINI_API_KEY"])

model = genai.GenerativeModel('gemini-1.5-pro')
response = model.generate_content('Explain quantum computing')
print(response.text)
```

#### Cloudflare

Skills for Cloudflare Workers, Pages, and services.

#### Netlify

Skills for Netlify deployment and edge functions.

## Creating Custom Skills

Use the `anthropics/skill-creator` skill to create your own:

### SKILL.md Template

```markdown
---
name: my-custom-skill
description: One-line description of what this skill does
triggers:
  - phrase users might say to invoke this skill
  - another trigger phrase
  - how do I use this tool
  - what are best practices for X
---

# My Custom Skill

## Overview

What this skill provides and when to use it.

## Installation

```bash
npm install my-package
```

## Basic Usage

```javascript
// Working code example
import { MyTool } from 'my-package';

const tool = new MyTool({
  apiKey: process.env.MY_API_KEY,
});
```

## Common Patterns

### Pattern 1: Basic Operation

```javascript
// Real example
const result = await tool.doSomething();
```

### Pattern 2: Advanced Usage

```javascript
// Real example with error handling
try {
  const result = await tool.complexOperation({
    param: 'value',
  });
} catch (error) {
  console.error('Operation failed:', error.message);
}
```

## Best Practices

1. Always validate input
2. Handle errors gracefully
3. Use environment variables for secrets

## Troubleshooting

**Error: Connection failed**
- Check your API key
- Verify network connectivity

**Error: Invalid parameter**
- Ensure required fields are provided
- Check parameter types
```

## Quality Standards

Skills in this collection follow these standards:

1. **Hand-picked**: Created by real engineering teams, not AI-generated
2. **Tested**: Used in production by actual developers
3. **Documented**: Clear examples and best practices
4. **Maintained**: Actively updated by project teams
5. **Practical**: Focus on real-world usage patterns

## Contributing

To contribute a skill:

1. Fork the repository
2. Create your skill following the template
3. Test it with at least one AI coding agent
4. Submit a pull request
5. Ensure your skill includes:
   - YAML frontmatter with name, description, triggers
   - Real code examples (no placeholders)
   - Environment variable references for secrets
   - Practical usage patterns

## Compatibility

Skills are plain Markdown files that work across all major AI coding agents:

- ✅ Claude Code (Anthropic)
- ✅ Codex
- ✅ Gemini CLI (Google)
- ✅ Cursor
- ✅ GitHub Copilot
- ✅ OpenCode
- ✅ Windsurf
- ✅ Antigravity

## Resources

- **Website**: [officialskills.sh](https://officialskills.sh/)
- **Repository**: [github.com/VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills)
- **Discord**: Community support and discussions
- **Documentation**: Agent-specific setup guides

## License

MIT License - See repository for full license text.
