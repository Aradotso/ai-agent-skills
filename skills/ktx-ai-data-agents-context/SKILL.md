---
name: ktx-ai-data-agents-context
description: Context layer for data agents - enables AI agents to query warehouses accurately using semantic layers, wiki knowledge, and approved metrics through MCP
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer with my warehouse
  - help me install and use ktx for AI data analysis
  - integrate ktx with Claude Code for database queries
  - build a ktx context layer from my dbt project
  - query my data warehouse using ktx semantic layer
  - troubleshoot ktx MCP server connection
  - ingest metadata into ktx from my data sources
---

# ktx AI Data Agents Context Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It automatically builds a semantic layer from your database schema, dbt models, Looker definitions, Metabase metadata, and wiki content—then exposes it through MCP (Model Context Protocol) for AI agents like Claude Code, Codex, Cursor, and OpenCode.

## What ktx Does

- **Automatic context building**: Introspects databases, detects joinable columns, resolves fan/chasm traps
- **Semantic layer**: Combines raw tables and approved metric definitions with automatic join graph resolution
- **Knowledge aggregation**: Ingests from dbt, LookML, Looker, Metabase, Notion, and markdown wikis
- **Contradiction detection**: Flags inconsistencies across different knowledge sources
- **MCP integration**: Exposes context to AI agents through the Model Context Protocol
- **Read-only by design**: Never writes to your database

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, and SQLite.

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
cd your-project
npx @kaelio/ktx setup
```

### Verification

```bash
ktx --version
ktx status
```

## Quick Setup

The `ktx setup` command walks through interactive configuration:

```bash
ktx setup
```

This will:
1. Create or resume a ktx project in the current directory
2. Configure LLM provider (Anthropic, Google Vertex AI, AI Gateway, or Claude Code session)
3. Configure embedding provider (OpenAI, Voyage AI, Google, etc.)
4. Add database connections (warehouse, staging, etc.)
5. Add context sources (dbt, Looker, Metabase, Notion, wiki)
6. Run initial ingestion to build context
7. Install agent integration (MCP server configuration)

### Example Setup Flow

```bash
# Start setup
ktx setup

# Configure LLM (example: Anthropic)
# Set ANTHROPIC_API_KEY environment variable first
export ANTHROPIC_API_KEY=your-api-key

# Configure embeddings (example: OpenAI)
export OPENAI_API_KEY=your-api-key

# Add database connection during setup
# Connection ID: warehouse
# Type: postgres
# Host: localhost
# Port: 5432
# Database: analytics
# Username: readonly_user
# Password: stored in .ktx/secrets/warehouse.env
```

## Project Structure

After setup, your project will have:

```
my-project/
├── ktx.yaml                          # Main configuration (commit)
├── semantic-layer/
│   └── warehouse/                    # YAML semantic sources (commit)
│       ├── tables.yaml
│       ├── metrics.yaml
│       └── joins.yaml
├── wiki/
│   ├── global/                       # Shared knowledge (commit)
│   └── user/<user-id>/               # Personal notes (optional)
├── raw-sources/
│   └── warehouse/                    # Ingestion artifacts
│       ├── tables.json
│       ├── columns.json
│       └── introspection-report.json
└── .ktx/                             # Local state, secrets (gitignore)
    ├── state.db
    └── secrets/
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local (gitignored).

## Configuration

### ktx.yaml Example

```yaml
version: 1
project:
  name: analytics
  description: Analytics warehouse context layer

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  api_key_env: ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  api_key_env: OPENAI_API_KEY

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    user: readonly_user
    password_env: WAREHOUSE_PASSWORD
    ssl: false
  
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: SVC_KTX
    password_env: SNOWFLAKE_PASSWORD

sources:
  dbt_main:
    type: dbt
    path: ./dbt
    connection: warehouse
  
  looker_metrics:
    type: lookml
    path: ./looker
    connection: warehouse
  
  company_wiki:
    type: markdown
    path: ./wiki/global
```

### Environment Variables

Store secrets in environment variables:

```bash
# .env or shell profile
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export WAREHOUSE_PASSWORD=...
export SNOWFLAKE_PASSWORD=...
```

Or use ktx's secret management:

```bash
# Stored in .ktx/secrets/warehouse.env
ktx secrets set warehouse WAREHOUSE_PASSWORD
```

## Core Commands

### Project Management

```bash
# Check project status
ktx status

# Initialize or update project
ktx setup

# Show configuration
ktx config show

# Validate configuration
ktx config validate
```

### Context Building

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific source
ktx ingest --source dbt_main

# Force re-ingestion (skip cache)
ktx ingest --force

# Dry run (show what would be ingested)
ktx ingest --dry-run
```

### Searching Context

```bash
# Search semantic layer (metrics, tables, columns)
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "metric definitions"

# Combined search
ktx search "payment processing"
```

### MCP Server for Agents

```bash
# Start MCP server (required for agent integration)
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status

# Stop MCP server
ktx mcp stop
```

## Agent Integration

### Claude Code / Codex / Cursor

After running `ktx setup`, the MCP server configuration is automatically added to your agent's config file.

**Manual configuration** (if needed):

For Claude Desktop (`~/Library/Application Support/Claude/claude_desktop_config.json`):

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

For Codex (`.codex/project.json`):

```json
{
  "mcp": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "${workspaceFolder}"]
    }
  }
}
```

### Verifying Agent Connection

1. Start MCP server: `ktx mcp start`
2. Open agent (Claude Code, Codex, etc.)
3. Ask: "What ktx semantic sources are available?"
4. Agent should list metrics, tables from your context

## TypeScript API (Programmatic Usage)

### Basic Context Query

```typescript
import { KtxContext } from '@kaelio/ktx';

async function queryContext() {
  const context = await KtxContext.load('/path/to/project');
  
  // Search semantic layer
  const metrics = await context.searchSemanticLayer('revenue', {
    limit: 10,
    types: ['metric', 'table']
  });
  
  console.log(metrics);
  
  // Search wiki
  const pages = await context.searchWiki('refund policy', {
    limit: 5
  });
  
  console.log(pages);
}
```

### Custom Ingestion

```typescript
import { KtxIngestor } from '@kaelio/ktx';

async function customIngest() {
  const ingestor = new KtxIngestor({
    projectDir: '/path/to/project',
    llmProvider: 'anthropic',
    llmModel: 'claude-sonnet-4-6'
  });
  
  // Ingest specific connection
  await ingestor.ingestConnection('warehouse', {
    force: true,
    verbose: true
  });
  
  // Ingest dbt source
  await ingestor.ingestSource('dbt_main', {
    detectJoins: true,
    sampleData: true
  });
}
```

### Semantic Layer Query Planning

```typescript
import { SemanticLayerEngine } from '@kaelio/ktx';

async function planQuery() {
  const engine = new SemanticLayerEngine('/path/to/project');
  
  // Declarative metric query
  const plan = await engine.plan({
    metrics: ['revenue', 'active_users'],
    dimensions: ['country', 'month'],
    filters: [
      { column: 'signup_date', operator: '>=', value: '2024-01-01' }
    ]
  });
  
  console.log('Generated SQL:', plan.sql);
  console.log('Join graph:', plan.joins);
  console.log('Warnings:', plan.warnings); // fan traps, chasm traps, etc.
}
```

## Common Workflows

### Setting Up ktx for Existing dbt Project

```bash
# Navigate to dbt project
cd my-dbt-project

# Install ktx globally
npm install -g @kaelio/ktx

# Run setup
ktx setup

# During setup:
# - Add postgres/snowflake connection to your warehouse
# - Add dbt source pointing to current directory
# - Choose to ingest immediately

# Verify semantic layer
ktx sl "orders"
ktx sl "revenue"

# Start MCP for agent
ktx mcp start
```

### Adding Looker Metrics

```bash
# Add LookML source to existing project
ktx setup

# Select "Add a new context source"
# Type: lookml
# Path: ./looker
# Connection: warehouse

# Ingest Looker definitions
ktx ingest --source looker_metrics

# Search for Looker metrics
ktx sl "ltv"
```

### Creating Custom Wiki Pages

```bash
# Create wiki page
mkdir -p wiki/global
cat > wiki/global/metrics.md << 'EOF'
# Metric Definitions

## Revenue
Total payment amount excluding refunds and credits.
Calculated as `SUM(payments.amount) WHERE payments.status = 'succeeded'`

## Active Users
Users who logged in during the period.
Distinct count of `user_events.user_id WHERE event_type = 'login'`
EOF

# Ingest wiki
ktx ingest

# Search wiki
ktx wiki "revenue definition"
```

### Querying Across Multiple Sources

```typescript
import { KtxContext } from '@kaelio/ktx';

async function crossSourceQuery() {
  const ctx = await KtxContext.load();
  
  // Get metric from dbt semantic layer
  const dbtMetric = await ctx.getMetric('dbt_main', 'monthly_revenue');
  
  // Get corresponding Looker definition
  const lookerMetric = await ctx.getMetric('looker_metrics', 'revenue_monthly');
  
  // Compare definitions
  if (dbtMetric.sql !== lookerMetric.sql) {
    console.warn('Metric definition mismatch!');
    console.log('dbt:', dbtMetric.sql);
    console.log('Looker:', lookerMetric.sql);
  }
}
```

## Troubleshooting

### MCP Server Not Starting

**Problem**: Agent can't connect to ktx MCP server

**Solution**:
```bash
# Check status
ktx mcp status

# Check project directory
ktx status

# Start manually with verbose logging
ktx mcp start --verbose

# Verify agent config points to correct project directory
# Should be absolute path, not relative
```

### Ingestion Failing

**Problem**: `ktx ingest` errors or incomplete context

**Solutions**:

```bash
# Check connection credentials
ktx config validate

# Test database connection
ktx test connection warehouse

# View detailed ingestion logs
ktx ingest --verbose

# Force re-ingestion (clear cache)
ktx ingest --force

# Ingest one source at a time
ktx ingest --source dbt_main
```

### LLM Rate Limits

**Problem**: Context building fails with rate limit errors

**Solution**:
```bash
# Use smaller model for context building
# Edit ktx.yaml:
llm:
  provider: anthropic
  model: claude-haiku-3-5  # Faster, cheaper
  
# Or reduce batch size
ktx ingest --batch-size 10
```

### Missing Semantic Sources

**Problem**: `ktx sl` returns no results for known metrics

**Solutions**:

```bash
# Verify source was ingested
ktx status

# Check raw-sources directory for artifacts
ls -la raw-sources/warehouse/

# Re-run ingestion
ktx ingest --source dbt_main --force

# Check dbt/LookML paths in ktx.yaml
ktx config show
```

### Agent Not Using ktx Context

**Problem**: Agent doesn't use ktx semantic layer in responses

**Solutions**:

1. Verify MCP server is running:
```bash
ktx mcp status
```

2. Check agent MCP config includes ktx server

3. Restart agent after config changes

4. Ask agent directly: "List ktx semantic sources" to verify connection

5. Check project directory is absolute path in MCP config

### Join Graph Errors

**Problem**: Semantic layer query planning fails with join errors

**Solution**:

```typescript
// Check join graph manually
import { SemanticLayerEngine } from '@kaelio/ktx';

const engine = new SemanticLayerEngine('/path/to/project');
const graph = await engine.getJoinGraph();

console.log('Available joins:', graph.joins);
console.log('Warnings:', graph.warnings);

// Add manual join definition if needed
// Edit semantic-layer/warehouse/joins.yaml:
```

```yaml
joins:
  - from: orders
    to: customers
    type: left
    on:
      - orders.customer_id = customers.id
```

### Database Permissions

**Problem**: Ingestion fails with permission errors

**Solution**:

```sql
-- Grant read-only access for ktx user
GRANT USAGE ON SCHEMA public TO ktx_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ktx_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA analytics TO ktx_readonly;

-- For Snowflake
GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ktx_readonly;
GRANT USAGE ON DATABASE ANALYTICS TO ktx_readonly;
GRANT USAGE ON SCHEMA ANALYTICS.PUBLIC TO ktx_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA ANALYTICS.PUBLIC TO ktx_readonly;
```

## Best Practices

1. **Start small**: Ingest one connection and one source first, verify results
2. **Use read-only credentials**: ktx never writes, but use least-privilege DB users
3. **Commit semantic layer**: Track `semantic-layer/` and `wiki/` in git for team sharing
4. **Keep secrets local**: Never commit `.ktx/` directory
5. **Regular re-ingestion**: Run `ktx ingest` when schema or metrics change
6. **Test locally first**: Verify MCP server works before deploying to team
7. **Document contradictions**: When ktx flags metric mismatches, resolve in wiki

## Additional Resources

- [Documentation](https://docs.kaelio.com/ktx/docs/)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Setup Guide](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
