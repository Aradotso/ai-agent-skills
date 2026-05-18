---
name: agents-best-practices-harness-design
description: Design, audit, and refactor production-safe agentic harnesses with provider-neutral best practices for tools, permissions, planning, context, and observability.
triggers:
  - "design an agent harness"
  - "audit this agent architecture"
  - "how should agent permissions work"
  - "build an MVP agent for"
  - "agent context and memory strategy"
  - "agent tool design and approval gates"
  - "production readiness checklist for agents"
  - "how to structure agentic loops"
---

# agents-best-practices Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill provides **provider-neutral best practices** for designing, auditing, and refactoring **agentic harnesses**—the control plane around a model that validates, authorizes, executes, and observes tool calls. It applies to coding agents, research agents, support, operations, finance, legal, healthcare, education, and workflow automation agents.

**Core principle**: *The model proposes actions; the harness validates, authorizes, executes, records, and returns observations.*

---

## What This Skill Does

- **Generate MVP agent blueprints** for new domains with typed tools, permissions, and launch gates
- **Audit existing agent harnesses** for brittle loops, unbounded tools, missing budgets, and observability gaps
- **Design tools and permissions** with risk-appropriate approval gates
- **Structure planning mode** and goal-like loops with checkpoints and budgets
- **Build context and memory strategies** that preserve active state across compaction
- **Optimize prompt caching** and cost telemetry
- **Integrate skills, MCP, and external connectors** with progressive disclosure
- **Implement security, evals, and observability** for production readiness

---

## Installation

### Option A: Via `skills` CLI (Recommended)

```bash
npx skills add DenisSergeevitch/agents-best-practices -g
```

The `-g` flag installs globally for all projects.

### Option B: Manual Install

**For Codex:**
```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
git clone https://github.com/DenisSergeevitch/agents-best-practices.git \
  "${CODEX_HOME:-$HOME/.codex}/skills/agents-best-practices"
```

**For Claude Code (user-level):**
```bash
mkdir -p "$HOME/.claude/skills"
git clone https://github.com/DenisSergeevitch/agents-best-practices.git \
  "$HOME/.claude/skills/agents-best-practices"
```

**For Claude Code (project-level):**
```bash
mkdir -p .claude/skills
git clone https://github.com/DenisSergeevitch/agents-best-practices.git \
  .claude/skills/agents-best-practices
```

### Verification

After install, verify the skill is discoverable:

```bash
# Codex
ls "${CODEX_HOME:-$HOME/.codex}/skills/agents-best-practices"

# Claude Code
ls "$HOME/.claude/skills/agents-best-practices"
```

You should see `SKILL.md`, `README.md`, `icon.jpeg`, and `references/`.

---

## Repository Structure

```
agents-best-practices/
├── SKILL.md                                  # skill entrypoint (this file)
├── README.md                                 # public-facing overview
├── icon.jpeg                                 # skill icon
└── references/
    ├── mvp-agent-blueprint.md                # MVP harness generator
    ├── architecture.md                       # component model
    ├── agentic-loop.md                       # loop invariants and budgets
    ├── tools-and-permissions.md              # typed tools and risk classes
    ├── planning-and-goals.md                 # planning mode and long-running goals
    ├── context-memory-compaction.md          # context, memory, retrieval
    ├── prompt-caching-and-cost.md            # cache-aware context layout
    ├── skills-and-connectors.md              # Agent Skills, MCP, connectors
    ├── system-prompts-instructions.md        # instruction hierarchy
    ├── provider-api-patterns.md              # OpenAI, Anthropic, compatible APIs
    ├── security-evals-observability.md       # guardrails, tracing, evals
    ├── agent-legibility-feedback-loops.md    # artifacts and cleanup
    ├── checklists.md                         # implementation and audit checklists
    ├── coverage-audit.md                     # topic coverage verification
    └── source-links.md                       # official references
```

---

## Core Concepts

### 1. The Agentic Loop

Every agent follows this pattern:

```
user/task → context builder → model call → typed tool call
→ schema validation → permission check → execution or pause
→ structured observation → next step or final answer
```

**Key invariants:**
- Every tool call gets a result (success, denial, timeout, malformed, abort)
- Risk changes the loop (reads vs. drafts vs. writes vs. external communications)
- Budgets prevent runaway loops (steps, time, tokens, cost, tool calls)
- Active state survives compaction

Reference: `references/agentic-loop.md`

### 2. Tools and Permissions

**Risk classes** determine permission requirements:

| Risk Class | Examples | Permission |
|------------|----------|------------|
| `read_private_data` | Read CRM, support tickets | Autonomous with scope |
| `draft_external_message` | Draft email, Slack message | Autonomous with label |
| `write_database` | Update record | Approval gate |
| `external_communication` | Send email, post to Slack | Approval gate |
| `destructive_action` | Delete, archive | Approval gate |
| `privileged_access` | Admin tools, deploy | Approval gate |
| `financial_operation` | Charge card, transfer funds | Approval gate |

**Pattern: Typed Tools**

```typescript
// Good: Narrow, typed, deterministic
interface SendCustomerEmailTool {
  name: "send_customer_email";
  parameters: {
    account_id: string;
    template: "renewal_reminder" | "upgrade_offer" | "support_followup";
    variables: Record<string, string>;
  };
  permission: "approval_gate";
}

// Bad: Generic, untyped, unbounded
interface SendMessageTool {
  name: "send_message";
  parameters: {
    to: string;
    body: string;
  };
}
```

Reference: `references/tools-and-permissions.md`

### 3. Planning and Goals

**Planning mode** separates thinking from acting:

```typescript
interface PlanningResult {
  plan: string;              // What the agent intends to do
  required_approvals: string[];  // Tools needing human approval
  estimated_steps: number;
  estimated_cost_usd: number;
  risk_summary: string;
}

// User approves the plan, then agent executes
```

**Goal-like loops** need:
- Step budget (e.g., max 20 steps)
- Time budget (e.g., 5 minutes)
- Cost budget (e.g., $0.50)
- Checkpoints (e.g., save state every 5 steps)
- Termination reasons (success, budget, validation failure, user abort)

Reference: `references/planning-and-goals.md`

### 4. Context and Memory

**Context hierarchy:**

```typescript
interface AgentContext {
  // Stable, cache-friendly prefix
  system_instructions: string;
  skill_descriptions: string[];
  
  // Active state (outside prompt)
  plan: Plan | null;
  pending_approvals: Approval[];
  todos: Todo[];
  artifacts: Artifact[];
  
  // Recent conversation (compacted)
  messages: Message[];
  
  // Retrieved knowledge
  retrieved_docs: Document[];
}
```

**Compaction rules:**
1. Preserve active state (plan, approvals, todos, artifacts) outside the prompt
2. Summarize conversation, not decisions
3. Rehydrate from state, not chat history
4. Label trust boundaries (user, model, tool, external)

Reference: `references/context-memory-compaction.md`

### 5. Prompt Caching

**Cache-aware layout:**

```typescript
// Stable prefix (cached)
const systemPrefix = [
  systemInstructions,
  allSkillDescriptions,
  allToolSchemas,
  permanentExamples
];

// Dynamic suffix (not cached)
const dynamicSuffix = [
  retrievedDocs,
  recentMessages,
  currentTask
];

// OpenAI: system, cached_user, user
// Anthropic: system (cached), user (cached), user
```

**Cost telemetry:**

```typescript
interface ModelCallTelemetry {
  input_tokens: number;
  output_tokens: number;
  cached_tokens: number;
  cost_usd: number;
  cache_hit_rate: number;
}
```

Reference: `references/prompt-caching-and-cost.md`

### 6. Skills and Connectors

**Progressive disclosure:**

```typescript
// Step 1: Load skill summaries (cached)
const skillIndex = [
  { name: "web-search", description: "Search the web..." },
  { name: "code-analysis", description: "Analyze codebases..." }
];

// Step 2: Load full skill when relevant
if (userNeedsWebSearch) {
  const webSearchSkill = loadSkill("web-search");
  context.push(webSearchSkill.fullInstructions);
}
```

**MCP integration:**

```typescript
// Expose MCP server as typed tools
const mcpTools = await mcpClient.listTools();
for (const tool of mcpTools) {
  registerTool({
    name: tool.name,
    description: tool.description,
    parameters: tool.inputSchema,
    permission: classifyRisk(tool.name)
  });
}
```

Reference: `references/skills-and-connectors.md`

---

## Common Patterns

### Pattern 1: Generate MVP Agent Blueprint

**User request:**
```
Build an agent for account renewal risk. It should read CRM, support tickets,
and usage data, then draft renewal actions.
```

**Agent response using this skill:**

```markdown
## MVP Blueprint: Account Renewal Risk Agent

**Harness level:** Approval-gated Level 2

**Core loop:**
user/task → context builder → model call → typed tool call
→ schema validation → permission check → execution or pause
→ structured observation → next step or final brief

**Minimal tools:**

1. `read_account_profile` - permission: `read_private_data`
   - Input: account_id
   - Output: { name, tier, arr, contract_end_date, renewal_probability }

2. `list_support_tickets` - permission: `read_private_data`
   - Input: account_id, days: 90
   - Output: Ticket[]

3. `fetch_usage_summary` - permission: `read_private_data`
   - Input: account_id, days: 30
   - Output: { active_users, feature_adoption, last_login }

4. `draft_customer_email` - permission: `draft_external_message`
   - Input: account_id, template, variables
   - Output: { subject, body, draft_id }

5. `request_approval` - permission: `approval_gate`
   - Input: action, rationale, draft_id
   - Output: { approval_id, status: "pending" }

**Launch gate:**
- 20 historical accounts processed
- Trace review confirms no unapproved external sends
- Human acceptance on ≥80% of draft actions
- No unbounded tool results or timeout loops

**Implementation checklist:**
- [ ] Typed tools with JSON schemas
- [ ] Permission check before every execution
- [ ] Step budget (max 15 steps per account)
- [ ] Cost telemetry and budget ($0.10 per account)
- [ ] Structured observations with trust labels
- [ ] Eval: injection, timeout, missing result, budget exhaustion
```

Reference: `references/mvp-agent-blueprint.md`

---

### Pattern 2: Audit Existing Agent

**User request:**
```
Our research agent sometimes runs tools forever and forgets why it made
a decision after context compaction. Audit the harness.
```

**Agent response using this skill:**

```markdown
## Audit: Research Agent Harness

**Failure points identified:**

1. **No loop budgets** (agentic-loop.md)
   - Missing: step budget, time budget, cost budget
   - Risk: Runaway loops, cost overruns
   - Fix: Add `max_steps: 30`, `timeout_seconds: 300`, `max_cost_usd: 2.0`

2. **Compaction loses active state** (context-memory-compaction.md)
   - Current: Summarizes chat history, drops plan and approvals
   - Risk: Agent forgets what it's doing mid-task
   - Fix: Store plan, approvals, todos, artifacts outside prompt; rehydrate on load

3. **Tool results unbounded** (tools-and-permissions.md)
   - Current: Returns full API responses (some 50KB+)
   - Risk: Context overflow, cost explosion
   - Fix: Add `max_result_bytes: 4096` and structured summarization

4. **No event trace** (security-evals-observability.md)
   - Current: Logs model input/output only
   - Risk: Cannot debug tool call → execution → observation flow
   - Fix: Add structured trace: `{ step, tool, args, result, duration, cost }`

5. **Trust boundaries unlabeled** (context-memory-compaction.md)
   - Current: Mixes user input, model output, tool results
   - Risk: Injection attacks, confused deputy
   - Fix: Label every message: `source: "user" | "model" | "tool" | "external"`

**Fix priority:**
1. Add loop budgets and termination reasons
2. Move active state outside prompt
3. Bound tool results and add structured observations
4. Add event trace
5. Add evals for injection, timeout, budget exhaustion
```

References: `references/agentic-loop.md`, `references/context-memory-compaction.md`, `references/security-evals-observability.md`

---

### Pattern 3: Design Tools and Permissions

**User request:**
```
The ops agent needs Slack, Linear, Google Drive, and our internal deploy API.
How should tools and permissions work?
```

**Agent response using this skill:**

```markdown
## Tool Design: Ops Agent

**Risk classification:**

| Tool | Risk Class | Permission |
|------|------------|------------|
| `search_linear_issues` | `read_private_data` | Autonomous (scoped) |
| `read_gdrive_doc` | `read_private_data` | Autonomous (scoped) |
| `draft_slack_message` | `draft_external_message` | Autonomous (labeled) |
| `post_slack_message` | `external_communication` | Approval gate |
| `create_linear_issue` | `write_database` | Approval gate |
| `trigger_deploy` | `privileged_access` | Approval gate |

**Tool schemas:**

```typescript
// Good: Narrow, typed
interface PostSlackMessageTool {
  name: "post_slack_message";
  parameters: {
    channel: string;  // Must match allow-list
    message: string;
    thread_ts?: string;
  };
  permission: "approval_gate";
}

// Bad: Generic, unbounded
interface SendMessageTool {
  name: "send_message";
  parameters: {
    platform: string;
    destination: string;
    content: string;
  };
}
```

**Approval flow:**

```typescript
// Agent proposes
const proposal = {
  tool: "post_slack_message",
  args: { channel: "#incidents", message: "Deploy complete." },
  rationale: "Notify team of successful rollout."
};

// Harness pauses and stores
const approval = await requestApproval(proposal);

// Human reviews in UI
// On approval, harness executes and returns observation
const result = await executeWithApproval(approval.id);
```

**Connector governance:**

```typescript
// Wrap external APIs as typed tools
class SlackConnector {
  async postMessage(channel: string, message: string): Promise<ToolResult> {
    // Validate channel against allow-list
    if (!this.allowedChannels.includes(channel)) {
      return { status: "denied", reason: "Channel not in allow-list" };
    }
    
    // Execute
    const response = await this.slackClient.chat.postMessage({
      channel,
      text: message
    });
    
    // Return structured observation
    return {
      status: "success",
      data: { ts: response.ts, channel: response.channel },
      metadata: { timestamp: Date.now(), cost_usd: 0 }
    };
  }
}
```

Reference: `references/tools-and-permissions.md`, `references/skills-and-connectors.md`

---

### Pattern 4: Production Readiness Checklist

**User request:**
```
What should I check before launching this agent?
```

**Agent response using this skill:**

```markdown
## Production Readiness Checklist

**Runtime:**
- [ ] Loop budgets enforced (step, time, cost, tool calls)
- [ ] Termination reasons logged (success, budget, validation, abort)
- [ ] Retries with exponential backoff for transient failures
- [ ] Active state (plan, approvals, todos, artifacts) outside prompt
- [ ] Context compaction preserves active state

**Tools:**
- [ ] Every tool has JSON schema and docstring
- [ ] Risk classification for every tool
- [ ] Permission check before every execution
- [ ] Tool results bounded (max 4KB per result)
- [ ] Structured observations with trust labels
- [ ] No generic tools (execute_anything, send_message, write_database)

**Security:**
- [ ] Input validation for all tool arguments
- [ ] Approval gates for high-risk tools
- [ ] Injection evals pass (prompt injection, jailbreak, PII leakage)
- [ ] Sandboxing for code execution tools
- [ ] Rate limits and abuse detection

**Observability:**
- [ ] Structured event trace (step, tool, args, result, duration, cost)
- [ ] Cost telemetry per task (input tokens, output tokens, cached tokens, USD)
- [ ] Error categorization (validation, permission, execution, timeout)
- [ ] Dashboards for cost, latency, success rate, approval rate

**Evals:**
- [ ] Historical task replay (≥50 tasks)
- [ ] Adversarial inputs (injection, confused deputy, unbounded loops)
- [ ] Edge cases (empty results, timeouts, malformed args, missing approvals)
- [ ] Human eval on ≥80% of high-risk tool calls

**Launch gates:**
- [ ] Shadow mode with human review for 1 week
- [ ] No unapproved external communications
- [ ] No cost overruns (≤10% over estimate)
- [ ] Incident response plan documented
```

Reference: `references/checklists.md`, `references/security-evals-observability.md`

---

## Provider API Patterns

### OpenAI (Compatible)

```typescript
import OpenAI from "openai";

const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: systemInstructions },
    { role: "user", content: task }
  ],
  tools: tools.map(t => ({
    type: "function",
    function: {
      name: t.name,
      description: t.description,
      parameters: t.parameters
    }
  })),
  tool_choice: "auto"
});

// Handle tool calls
if (response.choices[0].message.tool_calls) {
  for (const toolCall of response.choices[0].message.tool_calls) {
    const result = await executeToolWithPermission(
      toolCall.function.name,
      JSON.parse(toolCall.function.arguments)
    );
    
    messages.push({
      role: "tool",
      tool_call_id: toolCall.id,
      content: JSON.stringify(result)
    });
  }
}
```

### Anthropic

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

const response = await client.messages.create({
  model: "claude-3-7-sonnet-20250219",
  max_tokens: 4096,
  system: [
    { type: "text", text: systemInstructions, cache_control: { type: "ephemeral" } }
  ],
  messages: [
    { role: "user", content: task }
  ],
  tools: tools.map(t => ({
    name: t.name,
    description: t.description,
    input_schema: t.parameters
  }))
});

// Handle tool calls
if (response.stop_reason === "tool_use") {
  for (const block of response.content) {
    if (block.type === "tool_use") {
      const result = await executeToolWithPermission(block.name, block.input);
      
      messages.push({
        role: "user",
        content: [{
          type: "tool_result",
          tool_use_id: block.id,
          content: JSON.stringify(result)
        }]
      });
    }
  }
}

// Track cache metrics
console.log({
  input_tokens: response.usage.input_tokens,
  cache_read_tokens: response.usage.cache_read_input_tokens,
  cache_creation_tokens: response.usage.cache_creation_input_tokens
});
```

Reference: `references/provider-api-patterns.md`

---

## Configuration Example

**Harness config:**

```typescript
interface AgentConfig {
  model: {
    provider: "openai" | "anthropic" | "openai-compatible";
    model: string;
    base_url?: string;
    api_key_env: string;
  };
  
  loop: {
    max_steps: number;           // e.g., 30
    timeout_seconds: number;     // e.g., 300
    max_cost_usd: number;        // e.g., 1.0
    max_tool_calls_per_step: number;  // e.g., 5
  };
  
  context: {
    max_messages: number;        // e.g., 50
    compaction_threshold: number; // e.g., 40
    max_tool_result_bytes: number; // e.g., 4096
  };
  
  permissions: {
    auto_approve: string[];      // e.g., ["read_private_data", "draft_external_message"]
    require_approval: string[];  // e.g., ["external_communication", "destructive_action"]
  };
  
  observability: {
    trace_enabled: boolean;
    cost_tracking_enabled: boolean;
    eval_mode: "shadow" | "production";
  };
}

const config: AgentConfig = {
  model: {
    provider: "anthropic",
    model: "claude-3-7-sonnet-20250219",
    api_key_env: "ANTHROPIC_API_KEY"
  },
  loop: {
    max_steps: 30,
    timeout_seconds: 300,
    max_cost_usd: 1.0,
    max_tool_calls_per_step: 5
  },
  context: {
    max_messages: 50,
    compaction_threshold: 40,
    max_tool_result_bytes: 4096
  },
  permissions: {
    auto_approve: ["read_private_data", "draft_external_message"],
    require_approval: ["external_communication", "write_database", "destructive_action"]
  },
  observability: {
    trace_enabled: true,
    cost_tracking_enabled: true,
    eval_mode: "shadow"
  }
};
```

---

## Troubleshooting

### Issue: Agent loops forever

**Symptoms:** Agent exceeds step budget or timeout without completing task.

**Diagnosis:**
- Check `references/agentic-loop.md` for loop invariants
- Verify step budget, time budget, and termination conditions are enforced
- Review tool results: are they bounded? Are failures properly observed?

**Fix:**
```typescript
// Add hard budgets
if (step >= config.loop.max_steps) {
  return { status: "budget_exhausted", reason: "max_steps" };
}

if (Date.now() - startTime > config.loop.timeout_seconds * 1000) {
  return { status: "timeout", reason: "time_budget" };
}

// Add stop rules
if (allTodosComplete() || userAborted() || criticalError()) {
  return { status: "terminated", reason: ... };
}
```

---

### Issue: Context compaction loses active work

**Symptoms:** Agent forgets plan, pending approvals, or todos after compaction.

**Diagnosis:**
- Check `references/context-memory-compaction.md`
- Verify active state is stored outside the prompt

**Fix:**
```typescript
interface ActiveState {
  plan: Plan | null;
  pending_approvals: Approval[];
  todos: Todo[];
  artifacts: Artifact[];
}

// Store outside prompt
const state = loadState(taskId);

// Rehydrate after compaction
const context = buildContext({
  system: systemInstructions,
  plan: state.plan,
  todos: state.todos,
  messages: compactedMessages
});
```

---

### Issue: Approval gates bypassed

**Symptoms:** High-risk tool executed without approval record.

**Diagnosis:**
- Check `references/tools-and-permissions.md`
- Verify permission check runs before every execution

**Fix:**
```typescript
async function executeToolWithPermission(tool: string, args: any): Promise<ToolResult> {
  const permission = getToolPermission(tool);
  
  if (permission === "approval_gate") {
    const approval = await findApproval(tool, args);
    if (!approval || approval.status !== "approved") {
      return { status: "denied", reason: "Requires human approval" };
    }
  }
  
  // Execute only after permission check passes
  return await executeTool(tool, args);
}
```

---

### Issue: Cost explosion

**Symptoms:** Task costs 10x estimate; cached_tokens = 0.

**Diagnosis:**
- Check `references/prompt-caching-and-cost.md`
- Verify stable prefix is cached

**Fix:**
```typescript
// OpenAI: Use system + cached_user pattern
const messages = [
  { role: "system", content: stablePrefix },
  { role: "user", content: cachedKnowledge, cache_control: { type: "ephemeral" } },
  { role: "user", content: dynamicTask }
];

// Anthropic: Cache system blocks
const system = [
  { type: "text", text: stablePrefix, cache_control: { type: "ephemeral" } }
];

// Track cache hit rate
if (cacheHitRate < 0.7) {
  console.warn("Low cache hit rate; review context layout");
}
```

---

### Issue: Injection attack

**Symptoms:** Agent executes unintended tool calls from user input.

**Diagnosis:**
- Check `references/security-evals-observability.md`
- Verify input validation and trust labels

**Fix:**
```typescript
// Label trust boundaries
const messages = [
  { role: "user", content: userInput, metadata: { source: "user", trusted: false } },
  { role: "assistant", content: modelOutput, metadata: { source: "model" } },
  { role: "tool", content: toolResult, metadata: { source: "tool", trusted: true } }
];

// Validate tool arguments against schema
const valid = validateSchema(tool.parameters, args);
if (!valid) {
  return { status: "validation_failed", errors: valid.errors };
}

// Run injection evals
await runEval("prompt_injection", testCases);
```

---

## When to Use This Skill

This skill activates when conversations involve:

- **Agent architecture**: harness, loop, runtime, control plane
- **Tool design**: permissions, approvals, typed tools, risk classes
- **Planning**: planning mode, goal loops, checkpoints, budgets
- **Context**: memory, compaction, retrieval, active state
- **Security**: injection, guardrails, evals, sandboxing
- **Observability**: tracing, cost telemetry, launch gates
- **Connectors**: skills, MCP, external APIs, progressive disclosure
- **Production readiness**: checklists, incident response, audits

---

## Key References

All detailed references live in `references/`:

- **MVP Blueprint**: `mvp-agent-blueprint.md` — domain-specific harness generator
- **Loop Design**: `agentic-loop.md` — invariants, retries, budgets, stopping
- **Tools**: `tools-and-permissions.md` — typed tools, risk classes, approvals
- **Planning**: `planning-and-goals.md` — planning mode, goal loops
- **Context**: `context-memory-compaction.md` — context, memory, retrieval
- **Caching**: `prompt-caching-and-cost.md` — cache-aware layout, cost telemetry
- **Connectors**: `skills-and-connectors.md` — Agent Skills, MCP, progressive disclosure
- **APIs**: `provider-api-patterns.md` — OpenAI, Anthropic, compatible
- **Security**: `security-evals-observability.md` — guardrails, tracing, evals
- **Checklists**: `checklists.md` — implementation and audit checklists

---

## Philosophy Summary

1. **The harness acts, not the model** — the model proposes; harness validates, authorizes, executes, records
2. **Every tool call gets a result** — denial, timeout, malformed, abort are observations too
3. **Risk changes the loop** — reads, drafts, writes, external comms, destructive, privileged need different paths
4. **Draft and commit are separate** — high-risk side effects require approval records outside prompt
5. **Context is built, not dumped** — retrieve just enough, label trust, preserve active state
6. **Long-running work needs budgets** — step, time, token, cost, tool-call budgets are product features
7. **Skills and connectors are progressively disclosed** — expose names first, load details when relevant
8. **Repeated failures become harness features** — validators, tools, docs, evals, policies beat repeating prompt advice

---

## License

MIT License — see repository for details.

---

## Learn More

- Repository: [github.com/DenisSergeevitch/agents-best-practices](https://github.com/DenisSergeevitch/agents-best-practices)
- Agent Skills Spec: [agentskills.io/specification](https://agentskills.io/specification)
- Official API docs: `references/source-links.md`
