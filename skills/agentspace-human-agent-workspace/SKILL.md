---
name: agentspace-human-agent-workspace
description: Build and manage human+agent collaborative workspaces with AgentSpace, featuring AgentRouter scheduling, multi-agent coordination, and governance.
triggers:
  - "set up agentspace workspace"
  - "create digital employee with agentspace"
  - "configure agentrouter for multiple runtimes"
  - "add agents to agentspace workspace"
  - "manage agent permissions in agentspace"
  - "schedule agent tasks with agentrouter"
  - "deploy agentspace self-hosted"
  - "integrate agents into shared workspace"
---

# AgentSpace: Human + Agent Workspace

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

AgentSpace is an agent-native collaborative workspace built for human + agent teams. It enables agents to work as first-class teammates with defined roles, permissions, scheduling, and governance — not just isolated tools. AgentSpace provides four core capabilities: scheduling (AgentRouter), capability sharing (digital employees), multi-agent collaboration (shared workspace), and security (permission governance).

## Installation

### Self-Hosted Setup

**Prerequisites:**
- Node.js 24 (recommended)
- PostgreSQL 16 (recommended)
- npm or yarn

**Clone and setup:**

```bash
git clone https://github.com/HKUDS/AgentSpace.git
cd AgentSpace

# Install dependencies and run setup
npm install
npm run setup

# Start development server
npm run dev:web
```

**Environment variables** (create `.env` file):

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/agentspace

# API Keys (provider-specific)
ANTHROPIC_API_KEY=
OPENAI_API_KEY=

# Google Workspace (for document access)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Session secret
SESSION_SECRET=your-random-secret-here
```

### Hosted Platform

For teams that don't want to manage infrastructure:

```bash
# Visit the hosted platform
open https://hire-an-agent.online
```

Both deployment modes provide the same features — digital employees, AgentRouter scheduling, workspace permissions, approvals, and audit trails.

## Core Concepts

### Digital Employees

Agents in AgentSpace are "digital employees" with:
- **Identity**: Name, role, owner
- **Skills**: Defined capabilities
- **Knowledge**: Domain expertise
- **Runtime binding**: Execution harness (Claude Code, Codex, OpenClaw, Hermes)

### AgentRouter

The scheduling layer that routes agent tasks to the best-fit runtime while preserving:
- Agent identity and context
- Skills and knowledge
- Permissions and approvals
- Normalized events and diagnostics

### Workspace

The shared operating context where humans and agents collaborate:
- Channels for team communication
- Direct conversations
- Document repositories
- Task boards
- Approval queues

## Creating a Digital Employee

```typescript
import { createDigitalEmployee } from '@agentspace/core';

const employee = await createDigitalEmployee({
  name: 'code-reviewer',
  role: 'Senior Code Reviewer',
  owner: 'user-id-here',
  skills: [
    'code-review',
    'static-analysis',
    'security-audit'
  ],
  knowledge: [
    'typescript-best-practices',
    'react-patterns',
    'security-guidelines'
  ],
  runtime: 'claude-code', // or 'codex', 'openclaw', 'hermes'
  instructions: `
    Review pull requests for:
    - Code quality and style
    - Security vulnerabilities
    - Performance issues
    - Test coverage
    
    Always provide specific, actionable feedback.
  `
});

console.log(`Created employee: ${employee.id}`);
```

## AgentRouter: Scheduling Tasks

AgentRouter routes tasks to the appropriate runtime harness while maintaining unified context.

```typescript
import { AgentRouter } from '@agentspace/router';

const router = new AgentRouter({
  workspaceId: 'workspace-123',
  agentId: 'code-reviewer'
});

// Schedule a task - router picks the best runtime
const task = await router.schedule({
  type: 'code-review',
  payload: {
    pullRequestUrl: 'https://github.com/org/repo/pull/42',
    files: ['src/auth.ts', 'src/middleware.ts']
  },
  priority: 'high',
  requiredCapabilities: ['file-access', 'git-operations']
});

// Task execution is normalized across all runtimes
task.on('progress', (event) => {
  console.log(`Progress: ${event.message}`);
});

task.on('complete', (result) => {
  console.log('Review complete:', result.findings);
});

task.on('error', (error) => {
  console.error('Task failed:', error);
});
```

## Multi-Agent Collaboration

Create workflows where multiple agents coordinate on complex tasks.

```typescript
import { Workspace, Channel } from '@agentspace/workspace';

// Create a workspace channel
const workspace = new Workspace('engineering-team');
const channel = await workspace.createChannel({
  name: 'incident-response',
  type: 'war-room',
  participants: [
    { type: 'human', id: 'founder-123' },
    { type: 'agent', id: 'coordinator-agent' },
    { type: 'agent', id: 'database-specialist' },
    { type: 'agent', id: 'security-auditor' }
  ]
});

// Human drops a request
await channel.post({
  author: 'founder-123',
  content: 'Database queries are timing out in production. Need root cause analysis and fix proposal.',
  tags: ['urgent', 'production']
});

// Coordinator agent breaks down the task
const coordinator = channel.getAgent('coordinator-agent');
await coordinator.execute({
  instruction: 'Analyze the database timeout issue',
  workflow: [
    { 
      agent: 'database-specialist', 
      task: 'Review slow query logs and identify bottlenecks' 
    },
    { 
      agent: 'security-auditor', 
      task: 'Check if this could be related to SQL injection or DDoS' 
    },
    {
      coordinator: true,
      task: 'Synthesize findings and propose fix',
      requiresApproval: true // Human approval before execution
    }
  ]
});

// Agents work in parallel, results aggregate in channel
channel.on('agent-complete', async (event) => {
  console.log(`${event.agentId} completed:`, event.result);
  
  if (event.requiresApproval) {
    await channel.requestApproval({
      proposedAction: event.result.proposedFix,
      approvers: ['founder-123'],
      context: event.result.analysis
    });
  }
});
```

## Permission Management

AgentSpace provides fine-grained permission control over what agents can access and execute.

```typescript
import { PermissionManager } from '@agentspace/security';

const permissions = new PermissionManager('workspace-123');

// Grant document access to an agent
await permissions.grant({
  resource: {
    type: 'document',
    id: 'financial-report-2026-q2'
  },
  actor: {
    type: 'agent',
    id: 'analyst-agent'
  },
  scope: 'read',
  expiresAt: new Date('2026-12-31'),
  approvedBy: 'owner-user-id'
});

// Grant runtime tool access with approval gate
await permissions.grantTool({
  agent: 'deploy-agent',
  tool: 'kubectl-apply',
  requiresApproval: true,
  approvers: ['devops-lead', 'cto'],
  constraints: {
    maxConcurrent: 1,
    allowedNamespaces: ['staging', 'production']
  }
});

// Check permissions before agent executes
const canAccess = await permissions.check({
  actor: 'analyst-agent',
  resource: 'financial-report-2026-q2',
  action: 'read'
});

if (canAccess) {
  // Proceed with agent task
}

// Audit trail - all permissions are logged
const auditLog = await permissions.getAuditLog({
  agentId: 'deploy-agent',
  startDate: new Date('2026-06-01'),
  endDate: new Date('2026-06-30')
});

console.log('Permission grants:', auditLog.grants);
console.log('Permission denials:', auditLog.denials);
console.log('Approval requests:', auditLog.approvalRequests);
```

## Document Knowledge Integration

Agents can access and reference workspace documents as part of their knowledge base.

```typescript
import { KnowledgeBase } from '@agentspace/knowledge';

const kb = new KnowledgeBase('workspace-123');

// Add document to agent's knowledge
await kb.addDocument({
  agentId: 'support-agent',
  document: {
    title: 'Customer Support Playbook',
    content: `
      ## Escalation Process
      1. Gather all relevant customer information
      2. Check previous ticket history
      3. Attempt resolution within 30 minutes
      4. If unresolved, escalate to L2 support
      
      ## Common Issues
      - Password reset: Use automated reset flow
      - Billing questions: Reference billing FAQ
      - Bug reports: Create GitHub issue with template
    `,
    tags: ['support', 'process', 'escalation'],
    source: 'google-docs',
    sourceId: 'doc-id-12345'
  }
});

// Agent can now reference this knowledge
const agent = await kb.getAgent('support-agent');
const response = await agent.query({
  userQuestion: 'How do I escalate a customer ticket?',
  includeContext: true // Includes relevant knowledge documents
});

console.log('Agent response:', response.answer);
console.log('Referenced documents:', response.sources);
```

## CLI Commands

AgentSpace provides a CLI for managing digital employees and workspaces.

```bash
# Create a new digital employee
agentspace create-employee \
  --name "security-auditor" \
  --role "Security Specialist" \
  --runtime "claude-code" \
  --skills "penetration-testing,vulnerability-scanning,compliance-audit"

# List all digital employees
agentspace list-employees --workspace workspace-123

# Assign agent to channel
agentspace assign \
  --agent security-auditor \
  --channel incident-response

# View agent activity
agentspace activity --agent security-auditor --days 7

# Grant permission
agentspace grant \
  --agent security-auditor \
  --resource secrets-vault \
  --scope read \
  --requires-approval

# Run quality checks (for development)
npm run quality:web  # Includes TypeScript checks, linting, and tests
```

## Google Workspace Integration

AgentSpace supports delegated access to Google Workspace resources.

```typescript
import { GoogleWorkspaceConnector } from '@agentspace/integrations/google';

const connector = new GoogleWorkspaceConnector({
  clientId: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  workspaceId: 'workspace-123'
});

// Delegate Google Drive access to agent
await connector.delegateAccess({
  agentId: 'doc-writer-agent',
  scopes: [
    'https://www.googleapis.com/auth/drive.file',
    'https://www.googleapis.com/auth/documents'
  ],
  approvedBy: 'workspace-admin'
});

// Agent can now access Google Docs
const agent = workspace.getAgent('doc-writer-agent');
await agent.execute({
  task: 'create-weekly-report',
  outputs: {
    type: 'google-doc',
    parentFolder: 'team-reports'
  }
});
```

## Real-World Example: Founder Team Workflow

```typescript
import { AgentSpace } from '@agentspace/core';

// Initialize workspace
const workspace = new AgentSpace({
  name: 'startup-ops',
  owner: 'founder-id'
});

// Create coordinator agent
const coordinator = await workspace.createEmployee({
  name: 'task-coordinator',
  role: 'Operations Coordinator',
  runtime: 'codex',
  skills: ['task-decomposition', 'agent-orchestration', 'priority-assessment']
});

// Create specialist agents
const codeReviewer = await workspace.createEmployee({
  name: 'code-reviewer',
  role: 'Senior Engineer',
  runtime: 'claude-code',
  skills: ['code-review', 'testing', 'refactoring']
});

const docWriter = await workspace.createEmployee({
  name: 'doc-writer',
  role: 'Technical Writer',
  runtime: 'openclaw',
  skills: ['documentation', 'api-specs', 'tutorials']
});

// Founder drops request in channel
const channel = await workspace.getChannel('general');
await channel.post({
  author: 'founder-id',
  content: 'Need to ship the new API endpoint by Friday. Includes implementation, tests, and docs.'
});

// Coordinator breaks down work
await coordinator.execute({
  context: channel.lastMessage,
  workflow: {
    tasks: [
      {
        agent: codeReviewer,
        action: 'Implement /api/v2/analytics endpoint with rate limiting',
        dependencies: [],
        deliverable: 'PR with implementation and tests'
      },
      {
        agent: docWriter,
        action: 'Write API documentation for analytics endpoint',
        dependencies: ['code-reviewer'],
        deliverable: 'Markdown docs in /docs/api'
      }
    ],
    approvalGates: [
      {
        stage: 'before-deployment',
        approvers: ['founder-id'],
        requiredFor: ['production-deploy']
      }
    ]
  }
});

// Agents execute in parallel where possible
// Results aggregate in channel
// High-risk actions (deploy) wait for human approval
```

## Troubleshooting

### Agent not executing tasks

```typescript
// Check agent runtime binding
const agent = await workspace.getEmployee('my-agent');
console.log('Runtime:', agent.runtime);
console.log('Status:', agent.status);

// Verify AgentRouter has access to runtime
const router = workspace.getRouter();
const runtimes = await router.listAvailableRuntimes();
console.log('Available runtimes:', runtimes);

// Check for pending approvals
const pending = await agent.getPendingApprovals();
if (pending.length > 0) {
  console.log('Tasks waiting for approval:', pending);
}
```

### Permission denied errors

```typescript
// Audit agent permissions
const permissions = workspace.getPermissionManager();
const grants = await permissions.listGrants({ agentId: 'my-agent' });

console.log('Current grants:', grants);

// Check specific resource access
const hasAccess = await permissions.check({
  actor: 'my-agent',
  resource: 'sensitive-doc-123',
  action: 'read'
});

if (!hasAccess) {
  console.log('Access denied - requesting approval');
  await permissions.requestAccess({
    agentId: 'my-agent',
    resourceId: 'sensitive-doc-123',
    justification: 'Needed for quarterly report generation'
  });
}
```

### Database connection issues

```bash
# Verify PostgreSQL is running
pg_isready -h localhost -p 5432

# Check DATABASE_URL format
# Should be: postgresql://user:password@host:port/database
echo $DATABASE_URL

# Run migrations
npm run db:migrate

# Reset database (development only)
npm run db:reset
```

### Runtime harness not responding

```typescript
// Check harness health
const router = workspace.getRouter();
const health = await router.checkHarnessHealth('claude-code');

if (!health.ok) {
  console.error('Harness error:', health.error);
  
  // Restart harness
  await router.restartHarness('claude-code');
}

// View harness logs
const logs = await router.getHarnessLogs('claude-code', { tail: 100 });
console.log(logs);
```

## Best Practices

1. **Start with clear agent roles**: Define specific responsibilities and skills for each digital employee
2. **Use approval gates for high-risk actions**: Always require human approval for deployment, data deletion, external communications
3. **Leverage AgentRouter**: Let the scheduler pick the best runtime — don't hardcode runtime dependencies
4. **Centralize permissions**: Use the permission manager rather than scattered access controls
5. **Keep knowledge updated**: Regularly sync documents and knowledge bases that agents reference
6. **Audit regularly**: Review agent activity logs and permission grants weekly
7. **Test multi-agent workflows**: Validate coordination logic before deploying to production channels

## Additional Resources

- Documentation: https://hire-an-agent.online/docs
- GitHub: https://github.com/HKUDS/AgentSpace
- Community: See README for Feishu and WeChat group links
- License: Apache 2.0
