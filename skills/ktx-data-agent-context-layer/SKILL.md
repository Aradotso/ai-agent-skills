---
name: ktx-data-agent-context-layer
description: Use ktx to build a context layer for AI data agents with semantic search, metric definitions, and MCP integration
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer
  - build context with ktx for database queries
  - integrate ktx with claude code or cursor
  - search ktx wiki and semantic layer
  - ingest database metadata into ktx
  - query warehouse through ktx mcp server
  - create ktx skills for data analysis
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

ktx is an executable context layer that teaches AI agents how to query data warehouses accurately. It automatically builds a semantic layer from approved metric definitions, discovers joinable columns, ingests wiki content, and exposes everything through MCP tools and CLI commands. Agents get consistent, reusable definitions instead of reinventing SQL on every prompt.

## Installation

```bash
npm install -g @kaelio/ktx
```

Or use directly with npx:

```bash
npx @kaelio/ktx setup
```

## Quick Start

Initialize a new ktx project in your analytics directory:

```bash
ktx setup
```

This interactive command will:
- Create or resume a ktx project
- Configure LLM provider (Anthropic, Google Vertex AI, or Claude Code session)
- Configure embedding provider (OpenAI, Anthropic, Google)
- Connect to databases (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite)
- Add context sources (dbt, MetricFlow, LookML, Looker, Metabase, Notion)
- Build initial context
- Install agent integration (Codex, Claude Code, etc.)

Check project status:

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

```text
my-project/
├── ktx.yaml                         # Main configuration
├── semantic-layer/warehouse/        # Generated semantic layer YAML
│   ├── sources/
│   │   ├── raw_users.yaml
│   │   └── raw_orders.yaml
│   └── metrics/
│       └── revenue.yaml
├── wiki/global/                     # Shared business context
│   └── metrics/revenue-definition.md
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/warehouse/           # Ingest artifacts
│   ├── table-samples/
│   └── column-analysis.json
└── .ktx/                            # Local state (git-ignored)
    ├── config.yaml
    └── secrets/
```

**Commit:** `ktx.yaml`, `semantic-layer/`, `wiki/global/`  
**Ignore:** `.ktx/`

## Core Commands

### Build Context

Ingest metadata from all configured connections and sources:

```bash
ktx ingest
```

For a specific connection:

```bash
ktx ingest --connection warehouse
```

This will:
- Sample database tables
- Detect joinable columns
- Extract metadata from dbt, Looker, etc.
- Build semantic layer YAML
- Index wiki content
- Generate embeddings

### Search Semantic Layer

Search for metrics, dimensions, and sources:

```bash
ktx sl "revenue"
ktx sl "active users by region"
```

Example output:
```text
📊 revenue (metric)
  Description: Total revenue in USD
  SQL: SUM(orders.amount)
  Dimensions: date, product_id, region
  File: semantic-layer/warehouse/metrics/revenue.yaml

📋 orders (source)
  Description: All customer orders
  Columns: order_id, user_id, amount, created_at
  Joinable to: users via user_id
  File: semantic-layer/warehouse/sources/raw_orders.yaml
```

### Search Wiki

Search business context and documentation:

```bash
ktx wiki "refund policy"
ktx wiki "how to calculate ARR"
```

### MCP Server

Start the MCP server for agent integration:

```bash
ktx mcp start
```

With custom project directory:

```bash
ktx mcp start --project-dir /path/to/analytics
```

The MCP server exposes tools that agents can call to search semantic layer and wiki content.

## Configuration

### ktx.yaml

Main project configuration:

```yaml
version: 1
project:
  name: my-analytics
  description: Analytics warehouse for Acme Corp

llm:
  provider: anthropic
  model: claude-sonnet-4-6

embeddings:
  provider: openai
  model: text-embedding-3-small

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    sslmode: prefer

sources:
  dbt_main:
    type: dbt
    path: ./dbt_project
    profiles_dir: ~/.dbt

  looker_main:
    type: looker
    base_url: https://company.looker.com
    project_name: analytics

  notion_wiki:
    type: notion
    database_ids:
      - abc123
```

### Environment Variables

Credentials should be stored in environment variables:

```bash
# Database credentials
export POSTGRES_USER=analytics_user
export POSTGRES_PASSWORD=...

# LLM provider
export ANTHROPIC_API_KEY=...
export OPENAI_API_KEY=...

# Notion integration
export NOTION_TOKEN=...

# Looker API
export LOOKER_CLIENT_ID=...
export LOOKER_CLIENT_SECRET=...
```

ktx reads these automatically during setup and execution.

### Local Secrets

Sensitive configuration is stored in `.ktx/config.yaml` (git-ignored):

```yaml
llm:
  apiKey: ${ANTHROPIC_API_KEY}

embeddings:
  apiKey: ${OPENAI_API_KEY}

connections:
  warehouse:
    user: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}

sources:
  notion_wiki:
    token: ${NOTION_TOKEN}
```

## Semantic Layer

ktx automatically generates a semantic layer from your warehouse and existing tools.

### Sources

Raw table definitions with joinable columns:

```yaml
# semantic-layer/warehouse/sources/raw_users.yaml
source:
  name: raw_users
  description: User account data
  database: analytics
  schema: public
  table: users
  
  columns:
    - name: user_id
      type: integer
      description: Primary key
      
    - name: email
      type: varchar
      description: User email address
      
    - name: created_at
      type: timestamp
      description: Account creation timestamp
  
  joins:
    - to: raw_orders
      type: one_to_many
      on:
        - from: user_id
          to: user_id
```

### Metrics

Approved metric definitions:

```yaml
# semantic-layer/warehouse/metrics/revenue.yaml
metric:
  name: revenue
  description: Total revenue in USD
  type: sum
  sql: ${raw_orders.amount}
  
  dimensions:
    - ${raw_orders.created_at}
    - ${raw_users.region}
    - ${raw_orders.product_id}
  
  filters:
    - ${raw_orders.status} = 'completed'
```

Agents can query metrics declaratively:

```typescript
// Instead of writing SQL, agents ask:
// "What was revenue by region last month?"
// ktx resolves joins and applies canonical SQL
```

## TypeScript API

For programmatic usage:

```typescript
import { KtxProject, ProjectConfig } from '@kaelio/ktx';

// Load existing project
const project = await KtxProject.load('/path/to/analytics');

// Search semantic layer
const results = await project.semanticLayer.search('revenue');
for (const result of results) {
  console.log(result.name, result.type, result.description);
}

// Search wiki
const wikiResults = await project.wiki.search('refund policy');

// Get metric definition
const metric = await project.semanticLayer.getMetric('revenue');
console.log(metric.sql);
console.log(metric.dimensions);

// Build context
await project.ingest({ connectionId: 'warehouse' });
```

Create new project programmatically:

```typescript
import { createProject } from '@kaelio/ktx';

const config: ProjectConfig = {
  version: 1,
  project: {
    name: 'my-analytics',
    description: 'Analytics warehouse',
  },
  llm: {
    provider: 'anthropic',
    model: 'claude-sonnet-4-6',
    apiKey: process.env.ANTHROPIC_API_KEY,
  },
  embeddings: {
    provider: 'openai',
    model: 'text-embedding-3-small',
    apiKey: process.env.OPENAI_API_KEY,
  },
  connections: {
    warehouse: {
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      database: 'analytics',
      user: process.env.POSTGRES_USER,
      password: process.env.POSTGRES_PASSWORD,
    },
  },
};

await createProject('/path/to/analytics', config);
```

## Agent Integration

### Codex Integration

Install ktx as a Codex project skill:

```bash
cd /path/to/analytics
ktx setup
# Choose "codex:project" when prompted for agent integration
```

This adds ktx to your Codex skills registry. Agents can now:

```text
Use ktx to find the revenue metric definition
Query the wiki for our refund policy
Search the semantic layer for user-related tables
```

### Claude Code Session

Use Claude Code's local session as LLM provider:

```bash
ktx setup
# Choose "Claude Code session" as LLM provider
```

No API key needed — ktx uses the active Claude Code conversation.

### MCP Tools

ktx exposes these MCP tools to agents:

- `ktx_search_semantic_layer` — Search metrics, sources, dimensions
- `ktx_search_wiki` — Search business context
- `ktx_get_metric` — Get full metric definition
- `ktx_get_source` — Get table schema and joins
- `ktx_list_connections` — List available databases

Agents call these automatically when you ask data questions.

## Common Patterns

### Add a New Database Connection

```bash
ktx setup
# Choose "Add database connection"
# Follow prompts to configure
ktx ingest --connection new_warehouse
```

Or edit `ktx.yaml` directly:

```yaml
connections:
  new_warehouse:
    type: snowflake
    account: abc12345
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}
```

Then ingest:

```bash
ktx ingest --connection new_warehouse
```

### Ingest dbt Metadata

```yaml
sources:
  dbt_main:
    type: dbt
    path: ./dbt_project
    profiles_dir: ~/.dbt
    target: prod
```

```bash
ktx ingest --source dbt_main
```

ktx extracts:
- Model definitions and SQL
- Column descriptions
- Tests and constraints
- Lineage graphs

### Add Wiki Content

Create markdown files in `wiki/global/`:

```bash
mkdir -p wiki/global/metrics
cat > wiki/global/metrics/arr-definition.md <<EOF
# Annual Recurring Revenue (ARR)

ARR is calculated as the sum of all active subscription MRR * 12.

## Business Rules
- Only count subscriptions with status = 'active'
- Exclude one-time purchases
- Include discounts in MRR calculation

## SQL
\`\`\`sql
SELECT 
  SUM(monthly_amount * 12) as arr
FROM subscriptions
WHERE status = 'active'
  AND subscription_type != 'one_time'
\`\`\`
EOF

ktx ingest
```

Agents can now find this definition:

```bash
ktx wiki "how to calculate ARR"
```

### Manual Semantic Layer Edits

Edit generated YAML to add business logic:

```yaml
# semantic-layer/warehouse/metrics/active_users.yaml
metric:
  name: active_users
  description: Users who logged in during period
  type: count_distinct
  sql: ${raw_events.user_id}
  
  # Add custom filter
  filters:
    - ${raw_events.event_name} = 'login'
    - ${raw_events.created_at} >= CURRENT_DATE - INTERVAL '30 days'
  
  # Document business rule
  notes: |
    Active users are defined as users who have logged in
    at least once in the last 30 days. This is our standard
    retention metric.
```

Re-index after manual edits:

```bash
ktx ingest
```

### Query Planning

For complex queries, ktx resolves joins automatically:

```text
User: "Show me revenue by user region last month"

ktx resolves:
1. revenue metric requires raw_orders.amount
2. user region requires raw_users.region
3. Join path: raw_orders -> raw_users via user_id
4. Apply time filter to raw_orders.created_at
5. Return canonical SQL
```

Agents see the resolved query plan and can execute it directly.

## Troubleshooting

### "ktx project not initialized"

Run `ktx setup` in your project directory or specify path:

```bash
ktx setup --project-dir /path/to/analytics
```

Or set environment variable:

```bash
export KTX_PROJECT_DIR=/path/to/analytics
ktx status
```

### "LLM provider not configured"

Check `.ktx/config.yaml` has valid credentials:

```yaml
llm:
  provider: anthropic
  apiKey: ${ANTHROPIC_API_KEY}
```

Verify environment variable is set:

```bash
echo $ANTHROPIC_API_KEY
```

Re-run setup if needed:

```bash
ktx setup
```

### "Database connection failed"

Test connection manually:

```bash
psql -h localhost -U analytics_user -d analytics
```

Check `.ktx/config.yaml` credentials match. For Snowflake/BigQuery, verify service account permissions.

### "No semantic layer results"

Ensure context was built:

```bash
ktx ingest
ktx status  # Should show "ktx context built: yes"
```

Check `semantic-layer/` directory has YAML files:

```bash
ls -R semantic-layer/
```

If empty, verify database connection has readable tables:

```bash
ktx ingest --connection warehouse --verbose
```

### MCP Server Won't Start

Check if port is in use:

```bash
lsof -i :3000
```

Start with custom port:

```bash
ktx mcp start --port 3001
```

Verify project is initialized:

```bash
ktx status
```

### Embeddings Not Working

Check embeddings provider configuration:

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-small
  apiKey: ${OPENAI_API_KEY}
```

Re-build embeddings:

```bash
ktx ingest --rebuild-embeddings
```

## Advanced Usage

### Custom Connector

Add a custom database connector by extending the base class:

```typescript
import { DatabaseConnector } from '@kaelio/ktx';

class CustomConnector extends DatabaseConnector {
  async connect(): Promise<void> {
    // Custom connection logic
  }
  
  async getTables(): Promise<TableMetadata[]> {
    // Return table list
  }
  
  async sampleTable(table: string): Promise<any[]> {
    // Return sample rows
  }
}
```

Register in `ktx.yaml`:

```yaml
connections:
  custom_db:
    type: custom
    connector: ./connectors/custom.ts
```

### Programmatic Ingestion

```typescript
import { KtxProject } from '@kaelio/ktx';

const project = await KtxProject.load();

// Ingest specific connection
await project.ingest({
  connectionId: 'warehouse',
  tables: ['users', 'orders'],
  skipSampling: false,
  rebuildEmbeddings: true,
});

// Ingest specific source
await project.ingest({
  sourceId: 'dbt_main',
  extractLineage: true,
});
```

### Multi-Warehouse Setup

Configure multiple warehouses:

```yaml
connections:
  prod_warehouse:
    type: snowflake
    account: prod_account
    database: ANALYTICS
    
  dev_warehouse:
    type: postgres
    host: localhost
    database: analytics_dev
```

Agents can query either:

```text
Use ktx to search prod_warehouse for revenue metrics
Use ktx to search dev_warehouse for test data
```

## Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
