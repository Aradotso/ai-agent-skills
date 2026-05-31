---
name: ktx-ai-data-agents-mcp-context-skills
description: Executable context layer for data and analytics agents - enables Claude Code, Codex, and AI agents to query data accurately through MCP with skills, memory and a semantic layer
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer
  - integrate ktx with my warehouse
  - use ktx to query analytics data
  - build ktx context from dbt and wiki
  - enable agent access to warehouse with ktx
  - install ktx for AI data analysis
  - connect ktx to my database
---

# ktx AI Data Agents MCP Context Skills

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It combines approved metric definitions, joinable columns, wiki knowledge, and semantic layers into a single searchable surface exposed through MCP and CLI tools.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, dbt docs, Looker/Metabase metadata
- **Maps the data stack**: Samples tables, detects joinable columns, annotates sources
- **Builds a semantic layer**: Combines raw tables and high-level metrics with automatic join resolution
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite. Integrates with dbt, MetricFlow, LookML, Looker, Metabase, and Notion.

## Installation

```bash
# Install globally
npm install -g @kaelio/ktx

# Or use npx
npx @kaelio/ktx --version
```

For local development:

```bash
git clone https://github.com/kaelio/ktx.git
cd ktx
pnpm install
uv sync --all-groups
pnpm run build
```

## Initial Setup

```bash
# Interactive setup - creates or resumes ktx project
ktx setup

# Check project status
ktx status
```

The setup wizard will:
1. Create `ktx.yaml` configuration
2. Configure LLM provider (Anthropic, Google Vertex, AI Gateway, or Claude Code)
3. Configure embedding provider
4. Set up database connections
5. Add context sources (dbt, Looker, Metabase, Notion, wikis)
6. Build initial context
7. Install agent integration

## Project Structure

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts
└── .ktx/                            # Local state (git-ignored)
```

Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local.

## Configuration

### ktx.yaml Structure

```yaml
version: "1.0"
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  api_key_env: ANTHROPIC_API_KEY
embeddings:
  provider: openai
  model: text-embedding-3-small
  api_key_env: OPENAI_API_KEY
database_connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    username: readonly_user
    password_env: WAREHOUSE_PASSWORD
context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./dbt/target/manifest.json
    catalog_path: ./dbt/target/catalog.json
  company_wiki:
    type: local_wiki
    path: ./docs
```

### Database Connection Types

```typescript
// PostgreSQL
{
  type: "postgres",
  host: "localhost",
  port: 5432,
  database: "analytics",
  username: "readonly",
  password_env: "PG_PASSWORD"
}

// Snowflake
{
  type: "snowflake",
  account: "xy12345.us-east-1",
  warehouse: "ANALYTICS_WH",
  database: "ANALYTICS",
  schema: "PUBLIC",
  username: "readonly",
  password_env: "SNOWFLAKE_PASSWORD"
}

// BigQuery
{
  type: "bigquery",
  project_id: "my-project",
  dataset: "analytics",
  credentials_path_env: "GOOGLE_APPLICATION_CREDENTIALS"
}
```

### Context Source Types

```yaml
# dbt
dbt_main:
  type: dbt
  manifest_path: ./dbt/target/manifest.json
  catalog_path: ./dbt/target/catalog.json

# LookML
lookml_main:
  type: lookml
  project_path: ./lookml

# Looker instance
looker_api:
  type: looker
  base_url: https://company.looker.com
  client_id_env: LOOKER_CLIENT_ID
  client_secret_env: LOOKER_CLIENT_SECRET

# Metabase
metabase_api:
  type: metabase
  base_url: https://metabase.company.com
  username_env: METABASE_USER
  password_env: METABASE_PASSWORD

# Notion
notion_wiki:
  type: notion
  api_key_env: NOTION_API_KEY
  page_ids:
    - "a1b2c3d4e5f6"

# Local wiki
local_docs:
  type: local_wiki
  path: ./docs
```

## Key Commands

### Context Building

```bash
# Build context for all configured sources
ktx ingest

# Build for specific connection
ktx ingest --connection warehouse

# Force rebuild
ktx ingest --force
```

### Search Commands

```bash
# Search semantic layer sources
ktx sl "revenue"
ktx sl "monthly active users"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "customer segmentation"

# Combined search
ktx search "revenue metrics"
```

### MCP Server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP status
ktx mcp status
```

### Project Management

```bash
# Check project readiness
ktx status

# Validate configuration
ktx validate

# Update project
ktx update
```

## Usage Patterns

### Setting Up for a New Project

```typescript
// 1. Initialize in your analytics project
// Run: ktx setup

// 2. Configure LLM provider
// Choose Anthropic (recommended) or Google Vertex
// Set ANTHROPIC_API_KEY environment variable

// 3. Add database connection
// Connection ID: warehouse
// Type: postgres (or snowflake, bigquery, etc.)
// Set connection details and WAREHOUSE_PASSWORD env var

// 4. Add dbt as context source
// Source ID: dbt_main
// Type: dbt
// Manifest path: ./dbt/target/manifest.json
// Catalog path: ./dbt/target/catalog.json

// 5. Build context
// Run: ktx ingest

// 6. Verify setup
// Run: ktx status
```

### Using with Claude Code

```bash
# Install ktx skill in your project
# Ask Claude Code:
# "Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install and configure ktx in this project."

# Start MCP server
ktx mcp start --project-dir .

# Open Claude Code - ktx tools will be available
# Ask Claude: "What metrics are available for revenue?"
# Ask Claude: "Query monthly revenue by product category"
```

### Creating Semantic Layer Definitions

```yaml
# semantic-layer/warehouse/metrics.yaml
metrics:
  - name: revenue
    label: Total Revenue
    type: measure
    sql: SUM(${orders.amount})
    description: Total order revenue excluding refunds
    
  - name: monthly_active_users
    label: Monthly Active Users
    type: metric
    sql: COUNT(DISTINCT ${users.id})
    filters:
      - field: ${users.last_active_at}
        operator: ">="
        value: "CURRENT_DATE - INTERVAL '30 days'"
    description: Users active in the last 30 days

dimensions:
  - name: product_category
    label: Product Category
    type: string
    sql: ${products.category}
    
  - name: order_date
    label: Order Date
    type: time
    sql: ${orders.created_at}
    time_intervals: [day, week, month, quarter, year]
```

### Adding Wiki Content

```markdown
<!-- wiki/global/revenue-definitions.md -->
# Revenue Metrics

## Total Revenue

Sum of all order amounts excluding:
- Refunded orders
- Test orders (order_id < 1000)
- Internal employee orders

Formula: `SUM(orders.amount) WHERE status = 'completed' AND is_refunded = false`

## ARR (Annual Recurring Revenue)

Monthly recurring revenue × 12

Only includes subscription orders, not one-time purchases.
```

### Querying from Agents

```typescript
// MCP tools available to agents:

// 1. Search semantic layer
{
  name: "ktx_sl_search",
  input: { query: "revenue metrics" }
}

// 2. Search wiki
{
  name: "ktx_wiki_search",
  input: { query: "revenue definition" }
}

// 3. Get metric definition
{
  name: "ktx_get_metric",
  input: { metric_name: "revenue" }
}

// 4. List available metrics
{
  name: "ktx_list_metrics",
  input: { connection_id: "warehouse" }
}
```

### Programmatic Access

```typescript
// packages/cli/src/context/engine.ts
import { ContextEngine } from '@kaelio/ktx/context';

const engine = new ContextEngine({
  projectDir: '/path/to/project',
  llmConfig: {
    provider: 'anthropic',
    model: 'claude-sonnet-4-6',
    apiKey: process.env.ANTHROPIC_API_KEY
  }
});

// Search semantic layer
const slResults = await engine.searchSemanticLayer('revenue');
console.log(slResults);

// Search wiki
const wikiResults = await engine.searchWiki('refund policy');
console.log(wikiResults);

// Get specific metric
const metric = await engine.getMetric('warehouse', 'revenue');
console.log(metric);
```

## Advanced Configuration

### Multi-Warehouse Setup

```yaml
database_connections:
  production:
    type: snowflake
    account: prod.us-east-1
    warehouse: ANALYTICS_WH
    database: PROD
  staging:
    type: snowflake
    account: staging.us-east-1
    warehouse: ANALYTICS_WH
    database: STAGING

context_sources:
  dbt_prod:
    type: dbt
    connection: production
    manifest_path: ./dbt/target/prod/manifest.json
  dbt_staging:
    type: dbt
    connection: staging
    manifest_path: ./dbt/target/staging/manifest.json
```

### Custom Embedding Provider

```yaml
embeddings:
  provider: custom
  endpoint: https://embeddings.company.com/v1/embed
  api_key_env: CUSTOM_EMBEDDINGS_KEY
  model: company-embeddings-v2
  dimensions: 1536
```

### LLM Gateway Configuration

```yaml
llm:
  provider: ai_gateway
  endpoint: https://gateway.company.com/v1
  api_key_env: GATEWAY_API_KEY
  model: claude-sonnet-4-6
```

## Common Workflows

### Initial Project Setup

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Navigate to analytics project
cd ~/projects/analytics

# 3. Run setup wizard
ktx setup

# 4. Configure environment variables
export ANTHROPIC_API_KEY="your-key"
export WAREHOUSE_PASSWORD="your-password"

# 5. Build context
ktx ingest

# 6. Verify
ktx status
```

### Adding a New Metric

```bash
# 1. Create metric definition
cat > semantic-layer/warehouse/new-metric.yaml << EOF
metrics:
  - name: customer_lifetime_value
    label: Customer Lifetime Value
    type: measure
    sql: AVG(${customers.total_spent})
    description: Average total spend per customer
EOF

# 2. Rebuild context
ktx ingest --connection warehouse

# 3. Search for it
ktx sl "lifetime value"
```

### Updating Context from dbt

```bash
# 1. Run dbt
cd dbt
dbt run
dbt docs generate

# 2. Rebuild ktx context
cd ..
ktx ingest --source dbt_main

# 3. Verify new models
ktx sl "new_model_name"
```

### Integrating with CI/CD

```bash
#!/bin/bash
# .github/workflows/ktx-update.sh

# Install ktx
npm install -g @kaelio/ktx

# Set environment variables from secrets
export ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY"
export WAREHOUSE_PASSWORD="$WAREHOUSE_PASSWORD"

# Build context
ktx ingest --project-dir .

# Validate semantic layer
ktx validate

# Commit updated semantic layer
git add semantic-layer/ wiki/
git commit -m "Update ktx context [skip ci]"
git push
```

## Troubleshooting

### ktx status shows "LLM ready: no"

```bash
# Check LLM configuration in ktx.yaml
cat ktx.yaml | grep -A 5 "llm:"

# Verify API key environment variable is set
echo $ANTHROPIC_API_KEY

# Test connection
ktx validate --check-llm
```

### Database connection fails

```bash
# Test connection manually
ktx validate --check-connections

# Check credentials
echo $WAREHOUSE_PASSWORD

# Verify network access
psql -h localhost -U readonly -d analytics -c "SELECT 1"

# Check ktx.yaml connection config
cat ktx.yaml | grep -A 10 "database_connections:"
```

### Context ingestion errors

```bash
# Check specific source
ktx ingest --source dbt_main --verbose

# Verify source paths exist
ls -la dbt/target/manifest.json
ls -la dbt/target/catalog.json

# Force rebuild
ktx ingest --force --source dbt_main
```

### MCP server won't start

```bash
# Check if already running
ktx mcp status

# Kill existing process
pkill -f "ktx mcp"

# Start with debug logging
ktx mcp start --log-level debug

# Check project directory is correct
ktx mcp start --project-dir $(pwd)
```

### Semantic search returns no results

```bash
# Rebuild embeddings
ktx ingest --rebuild-embeddings

# Check embedding provider config
cat ktx.yaml | grep -A 5 "embeddings:"

# Verify API key
echo $OPENAI_API_KEY

# Try exact match search
ktx sl "exact_metric_name"
```

### Wiki content not appearing

```bash
# Check wiki path configuration
cat ktx.yaml | grep -A 5 "local_wiki"

# Verify files exist
ls -la wiki/global/

# Rebuild wiki index
ktx ingest --source company_wiki --force

# Check file format (markdown only)
file wiki/global/*.md
```

### Agent integration not working

```bash
# Verify MCP server is running
ktx mcp status

# Check Claude Code config
cat ~/.claude/config.json

# Restart MCP server
ktx mcp start --project-dir .

# Check logs
tail -f .ktx/logs/mcp-server.log
```

## Environment Variables

```bash
# LLM Providers
export ANTHROPIC_API_KEY="sk-ant-..."
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/credentials.json"

# Embedding Providers
export OPENAI_API_KEY="sk-..."

# Database Connections
export WAREHOUSE_PASSWORD="..."
export SNOWFLAKE_PASSWORD="..."
export BIGQUERY_CREDENTIALS="/path/to/bigquery-key.json"

# BI Tools
export LOOKER_CLIENT_ID="..."
export LOOKER_CLIENT_SECRET="..."
export METABASE_USER="..."
export METABASE_PASSWORD="..."

# Wikis
export NOTION_API_KEY="secret_..."

# Custom endpoints
export GATEWAY_API_KEY="..."
export CUSTOM_EMBEDDINGS_KEY="..."

# Project location (optional)
export KTX_PROJECT_DIR="/path/to/project"
```

## Development Commands

```bash
# Clone and setup
git clone https://github.com/kaelio/ktx.git
cd ktx
pnpm install
uv sync --all-groups

# Build
pnpm run build

# Type check
pnpm run type-check

# Run tests
pnpm run test
uv run pytest -q

# Link for local development
pnpm run link:dev
ktx-dev --help

# Check for dead code
pnpm run dead-code
```

## Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
