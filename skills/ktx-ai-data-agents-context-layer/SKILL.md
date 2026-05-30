---
name: ktx-ai-data-agents-context-layer
description: Install and configure ktx, the executable context layer for data and analytics agents with MCP skills, memory and semantic layer
triggers:
  - install and configure ktx for data agents
  - set up ktx context layer for warehouse queries
  - add ktx semantic layer to this project
  - configure ktx for Claude Code data analysis
  - help me query my data warehouse with ktx
  - set up ktx MCP server for AI agents
  - build ktx context from my dbt project
  - integrate ktx with my analytics stack
---

# ktx AI Data Agents Context Layer Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is an executable context layer that teaches AI agents how to query data warehouses accurately. It builds and maintains approved metric definitions, joinable columns, and business knowledge automatically by ingesting from dbt, MetricFlow, LookML, Looker, Metabase, Notion, and warehouse introspection. Exposes this context to agents via CLI and MCP (Model Context Protocol) tools with semantic search.

## Installation

### Quick Setup (Interactive)

```bash
npm install -g @kaelio/ktx
ktx setup
```

The `ktx setup` command walks through:
- Creating or resuming a ktx project in the current directory
- Configuring LLM provider (Anthropic, Vertex AI, AI Gateway, or Claude Code session)
- Configuring embedding provider (OpenAI, Voyage, local Claude Code)
- Adding database connections (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite)
- Adding context sources (dbt, MetricFlow, LookML, Looker, Metabase, Notion)
- Building initial context
- Installing agent integration (Codex, Claude Code, Cursor, etc.)

### Manual Installation in Existing Project

```bash
npm install -g @kaelio/ktx
cd /path/to/your/analytics-project
ktx init
```

Then edit `ktx.yaml` to configure providers and connections.

## Project Structure

After setup, ktx creates this structure:

```
my-project/
├── ktx.yaml                         # Main configuration
├── semantic-layer/<connection-id>/  # Generated YAML semantic sources
├── wiki/global/                     # Shared business context pages
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Commit:** `ktx.yaml`, `semantic-layer/`, `wiki/`  
**Ignore:** `.ktx/`

## Configuration

### ktx.yaml Structure

```yaml
project:
  name: analytics
  version: 1.0.0

providers:
  llm:
    type: anthropic
    model: claude-sonnet-4-6
    # API key read from ANTHROPIC_API_KEY env var
  
  embeddings:
    type: openai
    model: text-embedding-3-small
    # API key read from OPENAI_API_KEY env var

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    # Credentials read from WAREHOUSE_USER and WAREHOUSE_PASSWORD

context_sources:
  dbt_main:
    type: dbt
    project_dir: ./dbt
    profiles_dir: ~/.dbt
    profile: default
    target: prod

agent_integration:
  type: codex
  scope: project
```

### Supported Database Types

```typescript
// PostgreSQL
connections:
  warehouse:
    type: postgres
    host: ${POSTGRES_HOST}
    port: 5432
    database: analytics
    user: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}
    schema: public

// Snowflake
connections:
  warehouse:
    type: snowflake
    account: ${SNOWFLAKE_ACCOUNT}
    warehouse: ${SNOWFLAKE_WAREHOUSE}
    database: ANALYTICS
    schema: PUBLIC
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}

// BigQuery
connections:
  warehouse:
    type: bigquery
    project_id: ${GCP_PROJECT_ID}
    dataset: analytics
    credentials_path: ${GOOGLE_APPLICATION_CREDENTIALS}

// ClickHouse
connections:
  warehouse:
    type: clickhouse
    host: ${CLICKHOUSE_HOST}
    port: 8123
    database: default
    user: ${CLICKHOUSE_USER}
    password: ${CLICKHOUSE_PASSWORD}
```

### Supported Context Source Types

```yaml
# dbt project
context_sources:
  dbt_main:
    type: dbt
    project_dir: ./dbt
    profiles_dir: ~/.dbt
    profile: default
    target: prod

# MetricFlow
context_sources:
  metrics:
    type: metricflow
    config_path: ./semantic-layer.yaml

# LookML
context_sources:
  lookml:
    type: lookml
    project_dir: ./lookml

# Looker API
context_sources:
  looker:
    type: looker
    base_url: ${LOOKER_BASE_URL}
    client_id: ${LOOKER_CLIENT_ID}
    client_secret: ${LOOKER_CLIENT_SECRET}

# Metabase
context_sources:
  metabase:
    type: metabase
    base_url: ${METABASE_URL}
    username: ${METABASE_USER}
    password: ${METABASE_PASSWORD}

# Notion
context_sources:
  notion:
    type: notion
    token: ${NOTION_TOKEN}
    database_ids:
      - abc123def456
```

## Key Commands

### Status and Setup

```bash
# Check project readiness
ktx status

# Initialize new project
ktx init

# Run interactive setup
ktx setup

# Validate configuration
ktx validate
```

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific context source
ktx ingest --source dbt_main

# Force rebuild (bypass cache)
ktx ingest --force

# Dry run (show what would be ingested)
ktx ingest --dry-run
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, tables)
ktx sl "revenue"
ktx sl "active users by cohort"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "customer segmentation"

# Search with JSON output
ktx sl "revenue" --json
ktx wiki "refund policy" --json
```

### MCP Server

```bash
# Start MCP server (for agent integration)
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Start on custom port
ktx mcp start --port 3001

# Check MCP server status
ktx mcp status

# Stop MCP server
ktx mcp stop
```

### Wiki Management

```bash
# List wiki pages
ktx wiki list

# Show specific page
ktx wiki show global/metrics-definitions

# Create new page
ktx wiki create global/new-topic "Content here"

# Update page
ktx wiki update global/metrics-definitions "Updated content"

# Delete page
ktx wiki delete global/old-topic
```

## Agent Integration

### Using with Claude Code

After running `ktx setup`, the MCP server is automatically configured. Start it before opening Claude Code:

```bash
ktx mcp start --project-dir /path/to/your/project
```

Then in Claude Code, ask:

```
What metrics are available in ktx?
Query ktx for total revenue last quarter
What's our refund policy according to ktx wiki?
```

### Using with Codex

```bash
# Install ktx skill in Codex project scope
npx skills add Kaelio/ktx --skill ktx

# Or use this skill to set up ktx
ktx setup
```

### Using with Cursor or OpenCode

Configure the MCP server in your agent's settings, then use ktx tools from chat.

## Semantic Layer Patterns

### Generated Semantic Source Example

After ingestion, ktx generates YAML files in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/revenue_metrics.yaml
type: metric
id: revenue_metrics.total_revenue
name: Total Revenue
description: Sum of all revenue from completed orders
sql: |
  SELECT SUM(amount)
  FROM orders
  WHERE status = 'completed'
dimensions:
  - time_period
  - customer_segment
  - product_category
joins:
  - table: orders
    on: orders.id = revenue.order_id
  - table: customers
    on: customers.id = orders.customer_id
metadata:
  owner: analytics-team
  source: dbt_main
  last_updated: 2026-05-30T10:00:00Z
```

### Querying Metrics Declaratively

Instead of agents writing raw SQL, they use ktx semantic layer:

```typescript
// Agent uses ktx to find and execute metric
const result = await ktx.executeMetric({
  metric: 'total_revenue',
  dimensions: ['time_period', 'customer_segment'],
  filters: {
    time_period: { gte: '2026-01-01', lt: '2026-04-01' },
    customer_segment: { in: ['enterprise', 'mid-market'] }
  }
});
```

The semantic layer automatically:
- Resolves joins between tables
- Handles fan/chasm traps
- Applies metric logic consistently
- Uses approved definitions

## Common Patterns

### Full Project Setup from Scratch

```bash
# Navigate to analytics project
cd /path/to/analytics

# Install ktx globally
npm install -g @kaelio/ktx

# Run interactive setup
ktx setup
# Select: Anthropic for LLM
# Select: OpenAI for embeddings
# Add: PostgreSQL connection (warehouse)
# Add: dbt context source
# Build: Yes, build context now
# Install: Codex project integration

# Verify everything is ready
ktx status

# Start MCP server for agents
ktx mcp start
```

### Updating Context After dbt Changes

```bash
# After updating dbt models, re-ingest
ktx ingest --source dbt_main

# Force full rebuild if needed
ktx ingest --source dbt_main --force

# Verify new metrics are available
ktx sl "new_metric_name"
```

### Adding Wiki Documentation

```bash
# Create global business context
ktx wiki create global/revenue-recognition <<EOF
# Revenue Recognition Policy

We recognize revenue when:
1. Order is marked as completed
2. Payment is received
3. Goods are shipped

## Edge Cases
- Refunds reduce revenue in the period issued
- Subscription revenue is recognized monthly
EOF

# Verify agents can find it
ktx wiki "revenue recognition"
```

### Multi-Environment Setup

```yaml
# ktx.yaml with dev and prod connections
connections:
  warehouse_dev:
    type: postgres
    host: ${DEV_POSTGRES_HOST}
    database: analytics_dev
    user: ${DEV_POSTGRES_USER}
    password: ${DEV_POSTGRES_PASSWORD}
  
  warehouse_prod:
    type: postgres
    host: ${PROD_POSTGRES_HOST}
    database: analytics_prod
    user: ${PROD_POSTGRES_USER}
    password: ${PROD_POSTGRES_PASSWORD}

context_sources:
  dbt_dev:
    type: dbt
    project_dir: ./dbt
    target: dev
  
  dbt_prod:
    type: dbt
    project_dir: ./dbt
    target: prod
```

```bash
# Ingest dev environment
ktx ingest --connection warehouse_dev --source dbt_dev

# Ingest prod environment
ktx ingest --connection warehouse_prod --source dbt_prod
```

## Troubleshooting

### "ktx mcp start --project-dir ..." in status output

If `ktx status` shows this message, the MCP server isn't running. Start it:

```bash
ktx mcp start --project-dir /path/to/your/project
```

Keep this running in a separate terminal or background process while using agents.

### No Results from ktx sl or ktx wiki

Context hasn't been built yet:

```bash
# Build context for all sources
ktx ingest

# Check status
ktx status
```

### Database Connection Errors

Verify credentials are set:

```bash
# Check environment variables
echo $POSTGRES_HOST
echo $POSTGRES_USER

# Test connection manually
psql -h $POSTGRES_HOST -U $POSTGRES_USER -d analytics

# Update ktx.yaml with correct values
# Then re-run setup or ingest
ktx ingest --connection warehouse
```

### LLM Provider Issues

```bash
# Verify API key is set
echo $ANTHROPIC_API_KEY

# Test with a simple query
ktx sl "test" --json

# If using Claude Code session, ensure Claude Code is running
# and you've configured agent_integration.type: claude_code
```

### Conflicting Metric Definitions

ktx flags contradictions across sources during ingestion. Review reports:

```bash
# Check ingest reports
cat raw-sources/warehouse/ingest-report.json | jq '.contradictions'

# Resolve by:
# 1. Updating source definitions (dbt, Looker, etc.)
# 2. Adding clarification to wiki
ktx wiki create global/metrics-conflicts "Revenue metric uses..."
```

### MCP Server Won't Start

```bash
# Check if port is in use
lsof -i :3000

# Start on different port
ktx mcp start --port 3001

# Check logs
ktx mcp status
```

### Context Ingestion Fails

```bash
# Run with verbose output
ktx ingest --verbose

# Validate configuration first
ktx validate

# Test specific source
ktx ingest --source dbt_main --dry-run

# Check raw-sources directory for error details
cat raw-sources/warehouse/errors.log
```

## Project Resolution

ktx finds your project in this order:

1. `KTX_PROJECT_DIR` environment variable
2. Nearest `ktx.yaml` file walking up from current directory
3. Current directory

Override with `--project-dir`:

```bash
ktx ingest --project-dir /path/to/analytics
ktx mcp start --project-dir /path/to/analytics
```

## Development Mode

For local ktx development:

```bash
git clone https://github.com/kaelio/ktx.git
cd ktx
pnpm install
uv sync --all-groups
pnpm run build

# Create dev CLI alias
pnpm run setup:dev
pnpm run link:dev

# Use dev version
ktx-dev --help
ktx-dev setup
```

## Environment Variables Reference

```bash
# LLM Providers
export ANTHROPIC_API_KEY=sk-ant-...
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/vertex-ai-key.json
export AI_GATEWAY_URL=https://your-gateway.com
export AI_GATEWAY_API_KEY=...

# Embedding Providers
export OPENAI_API_KEY=sk-...
export VOYAGE_API_KEY=...

# Database Connections
export POSTGRES_HOST=localhost
export POSTGRES_PORT=5432
export POSTGRES_USER=analytics_user
export POSTGRES_PASSWORD=...
export SNOWFLAKE_ACCOUNT=abc12345
export SNOWFLAKE_WAREHOUSE=COMPUTE_WH
export SNOWFLAKE_USER=...
export SNOWFLAKE_PASSWORD=...
export GCP_PROJECT_ID=my-project
export CLICKHOUSE_HOST=localhost
export CLICKHOUSE_PORT=8123
export CLICKHOUSE_USER=default
export CLICKHOUSE_PASSWORD=...

# Context Sources
export LOOKER_BASE_URL=https://company.looker.com
export LOOKER_CLIENT_ID=...
export LOOKER_CLIENT_SECRET=...
export METABASE_URL=https://metabase.company.com
export METABASE_USER=...
export METABASE_PASSWORD=...
export NOTION_TOKEN=secret_...

# ktx Project
export KTX_PROJECT_DIR=/path/to/your/project
```
