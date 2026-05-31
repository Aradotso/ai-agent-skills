---
name: ktx-ai-data-agents-context
description: Context layer for AI data agents - teach Claude Code, Codex, and AI agents to query data warehouses accurately with semantic layer, wiki knowledge, and MCP tools
triggers:
  - set up ktx for AI data agents
  - configure ktx semantic layer for my warehouse
  - teach my agent to query the data warehouse correctly
  - install ktx context layer for data analysis
  - configure ktx with dbt and warehouse connections
  - use ktx to build data agent context
  - integrate ktx MCP server with my AI agent
  - help my agent understand our analytics metrics
---

# ktx AI Data Agents Context

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is an executable context layer that teaches AI agents (Claude Code, Codex, Cursor, OpenCode) how to query your data warehouse accurately. It automatically builds and maintains a semantic layer from your warehouse metadata, dbt projects, LookML, Looker, Metabase, and wiki content - then serves it to agents through CLI and MCP tools with full-text and semantic search.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, dbt docs, Looker, Metabase - organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, detects joinable columns, annotates sources so agents write better queries
- **Builds a semantic layer**: Combines raw tables and metrics through a join graph that resolves chasm and fan traps automatically
- **Serves agents at execution**: Exposes CLI and MCP tools for Claude Code, Codex, Cursor, OpenCode to query context

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install --save-dev @kaelio/ktx
```

Or with pnpm:

```bash
pnpm add -D @kaelio/ktx
```

## Initial Setup

### Interactive Setup (Recommended)

```bash
ktx setup
```

This command:
1. Creates or resumes a local ktx project
2. Configures LLM and embedding providers
3. Configures database connections
4. Configures context sources (dbt, Looker, Metabase, Notion)
5. Builds initial context
6. Installs agent integration (MCP or Skills)

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

## Configuration

### Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

### ktx.yaml Example

```yaml
version: 1
project_id: analytics-warehouse
llm:
  provider: anthropic
  model: claude-sonnet-4-6
embeddings:
  provider: openai
  model: text-embedding-3-small

databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    user: readonly_user
    # Password stored in .ktx/secrets.yaml or env var

context_sources:
  dbt_main:
    type: dbt
    project_dir: ./dbt
    profiles_dir: ~/.dbt
  
  notion_docs:
    type: notion
    # Token stored in .ktx/secrets.yaml or env var NOTION_TOKEN
```

### Environment Variables

ktx respects these environment variables:

```bash
# LLM configuration
export ANTHROPIC_API_KEY=your_key_here
export OPENAI_API_KEY=your_key_here

# Database credentials
export DB_PASSWORD=your_db_password

# Context sources
export NOTION_TOKEN=your_notion_token
export DBT_PROFILES_DIR=~/.dbt

# Project resolution
export KTX_PROJECT_DIR=/path/to/project
```

## Key Commands

### Building Context

```bash
# Build context from all configured sources
ktx ingest

# Build from specific connection
ktx ingest --connection warehouse

# Rebuild everything from scratch
ktx ingest --rebuild
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, sources)
ktx sl "revenue"
ktx sl "customer lifetime value"

# Search wiki content
ktx wiki "refund policy"
ktx wiki "revenue recognition rules"

# Combined search
ktx search "monthly recurring revenue"
```

### MCP Server (for AI Agents)

```bash
# Start MCP server for agent integration
ktx mcp start

# Start with custom project directory
ktx mcp start --project-dir /path/to/project

# List available MCP tools
ktx mcp tools
```

### Managing Semantic Layer

```bash
# Validate semantic layer YAML
ktx sl validate

# List all semantic sources
ktx sl list

# Show specific metric details
ktx sl show monthly_revenue
```

## Database Connection Examples

### PostgreSQL

```yaml
databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    user: readonly_user
    ssl: true
```

### Snowflake

```yaml
databases:
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: readonly_user
    role: ANALYST
```

### BigQuery

```yaml
databases:
  bigquery_prod:
    type: bigquery
    project_id: my-gcp-project
    dataset: analytics
    credentials_file: /path/to/service-account.json
```

### ClickHouse

```yaml
databases:
  clickhouse:
    type: clickhouse
    host: localhost
    port: 9000
    database: analytics
    user: readonly
```

## Context Source Examples

### dbt

```yaml
context_sources:
  dbt_main:
    type: dbt
    project_dir: ./dbt
    profiles_dir: ~/.dbt
    target: prod
```

### LookML

```yaml
context_sources:
  looker:
    type: lookml
    project_dir: ./looker-project
    models:
      - sales
      - marketing
```

### Metabase

```yaml
context_sources:
  metabase:
    type: metabase
    url: https://metabase.company.com
    # Token in .ktx/secrets.yaml or METABASE_TOKEN env var
    collections:
      - Sales Dashboards
      - Marketing Analytics
```

### Notion

```yaml
context_sources:
  notion_wiki:
    type: notion
    # Token in .ktx/secrets.yaml or NOTION_TOKEN env var
    page_ids:
      - abc123def456
      - ghi789jkl012
```

## Semantic Layer YAML

### Defining a Metric

```yaml
# semantic-layer/warehouse/metrics/revenue.yaml
type: metric
name: monthly_revenue
description: Total revenue aggregated by month
sql: |
  SELECT 
    DATE_TRUNC('month', order_date) as month,
    SUM(amount) as revenue
  FROM orders
  WHERE status = 'completed'
  GROUP BY 1
dimensions:
  - month
measures:
  - revenue
tags:
  - finance
  - reporting
```

### Defining a Dimension

```yaml
# semantic-layer/warehouse/dimensions/customer_segment.yaml
type: dimension
name: customer_segment
description: Customer segmentation based on lifetime value
sql: |
  CASE
    WHEN lifetime_value >= 10000 THEN 'Enterprise'
    WHEN lifetime_value >= 1000 THEN 'Mid-Market'
    ELSE 'SMB'
  END
source_table: customers
```

### Defining a Joinable Source

```yaml
# semantic-layer/warehouse/sources/orders.yaml
type: source
name: orders
description: All customer orders
table: public.orders
primary_key: order_id
joins:
  - to: customers
    type: many_to_one
    on: orders.customer_id = customers.customer_id
columns:
  - name: order_id
    type: integer
    description: Unique order identifier
  - name: customer_id
    type: integer
    description: Foreign key to customers
  - name: amount
    type: decimal
    description: Order total in USD
  - name: order_date
    type: timestamp
    description: When order was placed
```

## Agent Integration

### Using with Claude Code

After running `ktx setup`, ktx installs MCP integration automatically. The MCP server provides these tools:

- `ktx_search_semantic_layer`: Search metrics, dimensions, sources
- `ktx_search_wiki`: Search wiki content
- `ktx_get_metric`: Get full metric definition
- `ktx_query_preview`: Preview SQL for a semantic query

In Claude Code:
```text
Use ktx to find our revenue metrics
What wiki pages discuss refund policies?
Show me the SQL for monthly_recurring_revenue metric
```

### Using with Codex

```bash
# Install ktx skill
npx skills add Kaelio/ktx --skill ktx

# Then in your project
ktx setup
```

Codex can now use ktx tools:
```text
Query our warehouse for Q1 2024 revenue by customer segment
Find documentation about our churn calculation
```

### Using with Cursor or OpenCode

Start the MCP server manually:
```bash
ktx mcp start
```

Then configure your agent's MCP settings to connect to the ktx server.

## Common Workflows

### Setting Up a New Analytics Project

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Navigate to your project
cd /path/to/analytics-project

# 3. Run interactive setup
ktx setup

# 4. Verify configuration
ktx status

# 5. Build initial context
ktx ingest

# 6. Test search
ktx sl "revenue"
ktx wiki "metric definitions"
```

### Updating Context After Schema Changes

```bash
# Rebuild context for specific connection
ktx ingest --connection warehouse --rebuild

# Or rebuild everything
ktx ingest --rebuild
```

### Adding a New Metric

```bash
# 1. Create metric YAML file
cat > semantic-layer/warehouse/metrics/arr.yaml <<EOF
type: metric
name: annual_recurring_revenue
description: ARR calculated from active subscriptions
sql: |
  SELECT 
    DATE_TRUNC('year', subscription_start) as year,
    SUM(monthly_amount * 12) as arr
  FROM subscriptions
  WHERE status = 'active'
  GROUP BY 1
dimensions:
  - year
measures:
  - arr
tags:
  - finance
  - saas
EOF

# 2. Validate
ktx sl validate

# 3. Rebuild context
ktx ingest --connection warehouse

# 4. Test search
ktx sl "annual recurring revenue"
```

### Writing Wiki Documentation

```bash
# Create a wiki page
mkdir -p wiki/global/metrics
cat > wiki/global/metrics/revenue-recognition.md <<EOF
# Revenue Recognition

## Overview
Our revenue recognition follows ASC 606 guidelines.

## Key Rules
1. Revenue is recognized when service is delivered
2. Refunds are deducted in the month issued
3. Deferred revenue is amortized monthly

## Related Metrics
- monthly_revenue
- deferred_revenue
- recognized_revenue
EOF

# Rebuild wiki index
ktx ingest

# Search it
ktx wiki "revenue recognition"
```

## Troubleshooting

### ktx status shows "LLM ready: no"

Configure LLM provider:
```bash
ktx setup
# Follow prompts to configure Anthropic, OpenAI, or Vertex AI
```

Or set environment variable:
```bash
export ANTHROPIC_API_KEY=your_key_here
ktx status
```

### Database connection fails

Test connection:
```bash
# Check credentials in ktx.yaml or .ktx/secrets.yaml
ktx ingest --connection warehouse --verbose
```

Common fixes:
- Verify user has SELECT permissions on all tables
- Check firewall/network access to database host
- Ensure SSL settings match your database requirements
- For Snowflake: verify warehouse is running

### Semantic layer validation errors

```bash
# Validate specific file
ktx sl validate semantic-layer/warehouse/metrics/revenue.yaml

# Common issues:
# - Invalid YAML syntax
# - Missing required fields (name, type, sql)
# - Referenced tables don't exist
# - Invalid join definitions
```

### MCP server won't start

```bash
# Check if project is properly initialized
ktx status

# Ensure context is built
ktx ingest

# Start with verbose logging
ktx mcp start --verbose

# Check for port conflicts (default: 3000)
ktx mcp start --port 3001
```

### Search returns no results

```bash
# Rebuild search indexes
ktx ingest --rebuild

# Check if context sources are configured
cat ktx.yaml

# Verify semantic layer files exist
ls -la semantic-layer/

# Verify wiki files exist
ls -la wiki/global/
```

### Agent can't access ktx tools

1. Verify MCP server is running: `ktx status`
2. Check agent's MCP configuration points to correct ktx server
3. Restart agent after starting MCP server
4. Check MCP server logs: `ktx mcp start --verbose`

## Best Practices

### 1. Organize Semantic Layer by Domain

```text
semantic-layer/
  warehouse/
    metrics/
      finance/
        revenue.yaml
        arr.yaml
      product/
        dau.yaml
        retention.yaml
    dimensions/
      time.yaml
      geography.yaml
    sources/
      orders.yaml
      customers.yaml
```

### 2. Document Metric Business Logic in Wiki

Link semantic layer metrics to wiki documentation:

```yaml
# semantic-layer/warehouse/metrics/revenue.yaml
type: metric
name: monthly_revenue
description: See wiki/global/metrics/revenue-recognition.md for full details
# ... rest of definition
```

### 3. Use Tags for Discovery

```yaml
type: metric
name: customer_churn_rate
tags:
  - saas
  - retention
  - executive-dashboard
  - monthly-reporting
```

### 4. Version Control Everything

```bash
# .gitignore
.ktx/secrets.yaml
.ktx/cache/
.ktx/*.db
```

Commit:
- `ktx.yaml`
- `semantic-layer/`
- `wiki/global/`

### 5. Set Up Read-Only Database Users

ktx never writes to your database, but use dedicated read-only credentials:

```sql
-- PostgreSQL example
CREATE USER ktx_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE analytics TO ktx_readonly;
GRANT USAGE ON SCHEMA public TO ktx_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ktx_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
  GRANT SELECT ON TABLES TO ktx_readonly;
```

### 6. Regular Context Updates

Set up a cron job or CI workflow:

```bash
#!/bin/bash
# daily-context-update.sh
cd /path/to/project
ktx ingest --connection warehouse
ktx sl validate
```

## Advanced Usage

### Custom LLM Configuration

```yaml
llm:
  provider: vertex
  model: claude-3-5-sonnet-v2@20250219
  project_id: my-gcp-project
  region: us-central1
  temperature: 0.1
  max_tokens: 4096
```

### Multiple Database Connections

```yaml
databases:
  prod_warehouse:
    type: snowflake
    account: prod.us-east-1
    database: ANALYTICS_PROD
  
  staging_warehouse:
    type: snowflake
    account: staging.us-east-1
    database: ANALYTICS_STAGING
  
  postgres_app:
    type: postgres
    host: app-db.internal
    database: application
```

### Programmatic Usage (TypeScript)

```typescript
import { KtxContext } from '@kaelio/ktx';

const ctx = await KtxContext.load('/path/to/project');

// Search semantic layer
const results = await ctx.searchSemanticLayer('revenue');
console.log(results);

// Get metric definition
const metric = await ctx.getMetric('monthly_revenue');
console.log(metric.sql);

// Search wiki
const wikiResults = await ctx.searchWiki('refund policy');
console.log(wikiResults);
```

## Resources

- **Documentation**: https://docs.kaelio.com/ktx
- **GitHub**: https://github.com/Kaelio/ktx
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Setup Guide**: https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
