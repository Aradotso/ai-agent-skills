---
name: ktx-ai-data-agents-context-layer
description: Context layer for AI data agents that teaches them to query warehouses with approved metrics, semantic understanding, and business knowledge through MCP
triggers:
  - set up ktx for data analysis
  - configure ktx semantic layer
  - help me query the warehouse with ktx
  - integrate ktx with my AI agent
  - build context from my data warehouse
  - create ktx semantic metrics
  - search ktx wiki or semantic layer
  - configure ktx MCP server
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

ktx is an executable context layer that teaches AI agents how to accurately query data warehouses. It automatically builds and maintains approved metric definitions, discovers joinable columns, ingests business knowledge from wikis and documentation, and serves this context to agents through MCP (Model Context Protocol) and CLI tools.

## What ktx Does

ktx bridges the gap between general-purpose AI agents and enterprise data warehouses by:

- **Auto-discovering warehouse structure** — samples tables, captures metadata, detects joinable columns
- **Building a semantic layer** — combines raw tables and metrics with automatic join-graph resolution (fan/chasm trap handling)
- **Ingesting business knowledge** — pulls from dbt, Looker, Metabase, Notion, and organizes/deduplicates it
- **Serving agents** — exposes searchable context via MCP server and CLI with semantic + full-text search

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite.

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install --save-dev @kaelio/ktx
```

## Initial Setup

### Interactive Setup

```bash
ktx setup
```

This command:
1. Creates/resumes a ktx project in the current directory
2. Guides you through LLM provider configuration (Anthropic, Google Vertex, AI Gateway, or Claude Code SDK)
3. Configures database connections (read-only)
4. Sets up context sources (dbt, MetricFlow, LookML, Notion, etc.)
5. Builds initial context
6. Installs agent integration

### Check Project Status

```bash
ktx status
```

Example output:
```text
ktx project: /home/user/analytics
Project ready: yes
LLM ready: yes (claude-sonnet-4-6)
Embeddings ready: yes (text-embedding-3-small)
Databases configured: yes (warehouse)
Context sources configured: yes (dbt_main)
ktx context built: yes
Agent integration ready: yes (codex:project)
```

## Project Structure

```
my-project/
├── ktx.yaml                         # Main configuration (commit this)
├── semantic-layer/<connection-id>/  # YAML semantic sources (commit this)
├── wiki/global/                     # Shared business context (commit this)
├── wiki/user/<user-id>/             # User-scoped notes (commit this)
├── raw-sources/<connection-id>/     # Ingest artifacts and reports (commit this)
└── .ktx/                            # Local state and secrets (git-ignored)
```

## Configuration

### ktx.yaml Structure

```yaml
version: 1

llm:
  provider: anthropic
  model: claude-sonnet-4-20250514
  # API key stored in .ktx/secrets.yaml

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key stored in .ktx/secrets.yaml

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    # Credentials in .ktx/secrets.yaml
    readOnly: true

contextSources:
  dbt_main:
    type: dbt
    path: ./dbt_project
  
  notion_wiki:
    type: notion
    # Token in .ktx/secrets.yaml
    databaseIds:
      - abc123def456

agents:
  codex:
    type: codex
    scope: project
```

### Environment Variables for Secrets

Never hardcode secrets. ktx uses `.ktx/secrets.yaml` (git-ignored) or environment variables:

```bash
export ANTHROPIC_API_KEY=your_key_here
export OPENAI_API_KEY=your_key_here
export NOTION_TOKEN=your_token_here
```

## Core Commands

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific context source
ktx ingest --source dbt_main
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, tables)
ktx sl "monthly recurring revenue"
ktx sl "customer churn rate"

# Search wiki knowledge
ktx wiki "refund policy"
ktx wiki "how to calculate LTV"

# Combined search
ktx search "revenue attribution"
```

### MCP Server for Agents

```bash
# Start MCP server (required for Claude Code, Cursor, etc.)
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP status
ktx mcp status
```

### Managing Semantic Layer

```bash
# List semantic sources
ktx sl list

# Validate semantic layer definitions
ktx sl validate

# Generate SQL from semantic query
ktx sl query --metric revenue --dimensions country,month
```

## Real-World Usage Examples

### Example 1: Setting Up ktx for a Data Warehouse

```typescript
import { execSync } from 'child_process';
import { writeFileSync } from 'fs';

// Create ktx configuration programmatically
const ktxConfig = {
  version: 1,
  llm: {
    provider: 'anthropic',
    model: 'claude-sonnet-4-20250514'
  },
  embeddings: {
    provider: 'openai',
    model: 'text-embedding-3-small'
  },
  connections: {
    analytics_warehouse: {
      type: 'postgres',
      host: process.env.DB_HOST,
      port: 5432,
      database: 'analytics',
      readOnly: true
    }
  },
  contextSources: {
    dbt_models: {
      type: 'dbt',
      path: './dbt'
    }
  }
};

writeFileSync('ktx.yaml', JSON.stringify(ktxConfig, null, 2));

// Build context
execSync('ktx ingest', { stdio: 'inherit' });

// Verify setup
execSync('ktx status', { stdio: 'inherit' });
```

### Example 2: Defining a Semantic Metric

Create `semantic-layer/warehouse/metrics.yaml`:

```yaml
metrics:
  - name: monthly_recurring_revenue
    label: Monthly Recurring Revenue
    description: Sum of all active subscription revenue in a given month
    type: simple
    sql: SUM(subscription_amount)
    table: subscriptions
    filters:
      - field: status
        operator: equals
        value: active
    dimensions:
      - name: plan_type
        sql: plan_type
      - name: region
        sql: customer_region

  - name: customer_lifetime_value
    label: Customer Lifetime Value
    description: Average total revenue per customer over their lifetime
    type: derived
    sql: SUM(total_revenue) / COUNT(DISTINCT customer_id)
    table: customer_revenue_summary
```

### Example 3: Querying Semantic Layer from Code

```typescript
import { execSync } from 'child_process';

function queryMetric(metric: string, dimensions: string[], filters?: Record<string, any>) {
  const dimensionsArg = dimensions.join(',');
  const cmd = `ktx sl query --metric ${metric} --dimensions ${dimensionsArg}`;
  
  const result = execSync(cmd, { encoding: 'utf-8' });
  return JSON.parse(result);
}

// Query MRR by region and plan
const mrrData = queryMetric('monthly_recurring_revenue', ['region', 'plan_type']);

console.log('MRR Analysis:', mrrData);
```

### Example 4: Searching Context Programmatically

```typescript
import { execSync } from 'child_process';

function searchSemanticLayer(query: string): any[] {
  const result = execSync(`ktx sl "${query}" --json`, { encoding: 'utf-8' });
  return JSON.parse(result);
}

function searchWiki(query: string): any[] {
  const result = execSync(`ktx wiki "${query}" --json`, { encoding: 'utf-8' });
  return JSON.parse(result);
}

// Search for revenue-related metrics
const revenueMetrics = searchSemanticLayer('revenue');

// Search for business context about refunds
const refundContext = searchWiki('refund policy');

console.log('Found metrics:', revenueMetrics.length);
console.log('Found wiki pages:', refundContext.length);
```

### Example 5: Integrating with MCP-Enabled Agent

For Claude Code, Cursor, Codex, or OpenCode:

1. Ensure MCP server is running:

```bash
ktx mcp start --project-dir /path/to/project
```

2. Configure agent client (e.g., Claude Code `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "ktx": {
      "command": "npx",
      "args": [
        "@kaelio/ktx",
        "mcp",
        "start",
        "--project-dir",
        "/path/to/project"
      ]
    }
  }
}
```

3. Agent can now use ktx tools directly in conversation:

```
User: What is our monthly recurring revenue by region?
Agent: [uses ktx_search_semantic_layer("monthly recurring revenue")]
Agent: [uses ktx_query_metric with metric="monthly_recurring_revenue" dimensions=["region"]]
```

### Example 6: Adding Wiki Knowledge

Create `wiki/global/revenue-definitions.md`:

```markdown
# Revenue Definitions

## Monthly Recurring Revenue (MRR)

MRR is calculated as the sum of all active subscription amounts in a given month.

**Exclusions:**
- One-time fees
- Paused subscriptions
- Trial subscriptions

**Data Source:** `subscriptions` table, filtered to `status = 'active'`

## Annual Recurring Revenue (ARR)

ARR = MRR × 12

## Revenue Recognition

Revenue is recognized on the first day of each subscription period.
```

Ingest wiki content:

```bash
ktx ingest --source wiki
```

### Example 7: dbt Integration

If you have a dbt project at `./dbt`, add to `ktx.yaml`:

```yaml
contextSources:
  dbt_main:
    type: dbt
    path: ./dbt
    profilesDir: ~/.dbt
    target: prod
```

ktx will automatically ingest:
- Model definitions
- Metric definitions (dbt Semantic Layer / MetricFlow)
- Documentation
- Source freshness
- Test results

```bash
ktx ingest --source dbt_main
```

### Example 8: Multi-Warehouse Setup

```yaml
connections:
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: ANALYTICS_WH
    database: ANALYTICS
    schema: PUBLIC
    readOnly: true
  
  postgres_staging:
    type: postgres
    host: staging-db.internal
    port: 5432
    database: staging
    readOnly: true

contextSources:
  dbt_snowflake:
    type: dbt
    path: ./dbt
    target: prod
  
  dbt_staging:
    type: dbt
    path: ./dbt
    target: staging
```

Build context for specific connection:

```bash
ktx ingest --connection snowflake_prod
```

## Common Patterns

### Pattern 1: Scheduled Context Updates

```bash
#!/bin/bash
# cron job to keep ktx context fresh

cd /path/to/project
ktx ingest --connection warehouse
ktx sl validate
```

### Pattern 2: Pre-Commit Hook for Semantic Layer Validation

```bash
#!/bin/bash
# .git/hooks/pre-commit

if git diff --cached --name-only | grep -q "^semantic-layer/"; then
  echo "Validating semantic layer changes..."
  ktx sl validate || exit 1
fi
```

### Pattern 3: Agent Prompting Strategy

When using ktx with an AI agent:

```
Before answering data questions:
1. Use ktx_search_semantic_layer to find relevant metrics
2. Use ktx_search_wiki for business context
3. Use ktx_query_metric to get actual data
4. Cross-reference metric definitions to ensure accuracy
```

### Pattern 4: Custom Metric Development Workflow

1. Define metric in YAML:
   ```yaml
   # semantic-layer/warehouse/custom_metrics.yaml
   metrics:
     - name: revenue_per_customer
       sql: SUM(revenue) / COUNT(DISTINCT customer_id)
       table: orders
   ```

2. Validate:
   ```bash
   ktx sl validate
   ```

3. Test query:
   ```bash
   ktx sl query --metric revenue_per_customer --dimensions month
   ```

4. Commit to git

5. Re-ingest:
   ```bash
   ktx ingest --source semantic-layer
   ```

## Troubleshooting

### Issue: "LLM not ready"

**Cause:** API key not configured or invalid

**Solution:**
```bash
# Check configuration
ktx status

# Reconfigure LLM provider
ktx setup

# Or set environment variable
export ANTHROPIC_API_KEY=your_key_here
```

### Issue: "Database connection failed"

**Cause:** Credentials incorrect or network unreachable

**Solution:**
```bash
# Test connection manually
psql -h localhost -p 5432 -U username -d analytics

# Verify ktx.yaml connection settings
cat ktx.yaml | grep -A 5 "connections:"

# Check .ktx/secrets.yaml for credentials
cat .ktx/secrets.yaml
```

### Issue: "MCP server not responding"

**Cause:** Server not started or project directory mismatch

**Solution:**
```bash
# Check if MCP server is running
ktx mcp status

# Start MCP server explicitly
ktx mcp start --project-dir $(pwd)

# Restart agent client after starting MCP
```

### Issue: "No semantic sources found"

**Cause:** Context not built or semantic-layer directory empty

**Solution:**
```bash
# Check if semantic-layer directory exists
ls -la semantic-layer/

# Run ingestion
ktx ingest

# Verify status
ktx status
```

### Issue: "Embedding search returns no results"

**Cause:** Embeddings not generated or query too specific

**Solution:**
```bash
# Rebuild embeddings
ktx ingest --rebuild-embeddings

# Try broader search query
ktx sl "revenue"  # instead of "monthly_recurring_revenue_by_region_and_plan"
```

### Issue: "Context ingestion fails for dbt"

**Cause:** dbt project path incorrect or profiles not found

**Solution:**
```bash
# Verify dbt project compiles
cd ./dbt && dbt compile

# Check ktx.yaml dbt path
cat ktx.yaml | grep -A 3 "type: dbt"

# Provide explicit profiles directory
# In ktx.yaml:
# contextSources:
#   dbt_main:
#     type: dbt
#     path: ./dbt
#     profilesDir: ~/.dbt
```

### Issue: "Agent can't find ktx tools"

**Cause:** MCP server configuration incorrect in agent client

**Solution:**

For Claude Code, check `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "/absolute/path/to/project"]
    }
  }
}
```

Restart Claude Code after updating config.

## Advanced Usage

### Custom Connector for Proprietary System

```typescript
// Define custom connector in ktx.yaml
// Then implement in TypeScript

import { BaseConnector } from '@kaelio/ktx/connectors';

export class CustomConnector extends BaseConnector {
  async connect() {
    // Connection logic
  }
  
  async introspect() {
    // Return table/column metadata
    return {
      tables: [
        {
          name: 'custom_table',
          columns: [
            { name: 'id', type: 'integer', isPrimaryKey: true },
            { name: 'value', type: 'decimal' }
          ]
        }
      ]
    };
  }
}
```

### Programmatic Context Building

```typescript
import { ContextEngine } from '@kaelio/ktx/context';

const engine = new ContextEngine({
  projectDir: '/path/to/project',
  llmProvider: 'anthropic',
  embeddingsProvider: 'openai'
});

await engine.ingest(['warehouse']);
const results = await engine.search('revenue', { type: 'semantic' });

console.log('Found:', results.length, 'semantic sources');
```

## Key Commands Reference

| Command | Purpose |
|---------|---------|
| `ktx setup` | Interactive project setup |
| `ktx status` | Check project readiness |
| `ktx ingest` | Build context from all sources |
| `ktx sl <query>` | Search semantic layer |
| `ktx wiki <query>` | Search wiki knowledge |
| `ktx search <query>` | Combined search |
| `ktx mcp start` | Start MCP server |
| `ktx mcp status` | Check MCP server status |
| `ktx sl list` | List all semantic sources |
| `ktx sl validate` | Validate semantic layer YAML |
| `ktx sl query` | Generate SQL from semantic query |

For complete CLI reference: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
