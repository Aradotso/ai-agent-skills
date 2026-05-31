---
name: ktx-ai-data-agents-context-layer
description: Use ktx to give AI agents accurate data warehouse querying through semantic layers, metrics, and business knowledge via MCP
triggers:
  - set up ktx for data agent querying
  - configure ktx semantic layer for warehouse
  - use ktx to query data with approved metrics
  - integrate ktx with Claude Code for analytics
  - build context in ktx from dbt and warehouse
  - search ktx semantic layer and wiki
  - configure ktx MCP server for agents
  - ingest warehouse metadata into ktx
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

ktx is a self-improving context layer that teaches AI agents how to query data warehouses accurately using approved metric definitions, joinable columns, and business knowledge. It ingests metadata from dbt, LookML, Looker, Metabase, Notion, and raw warehouse tables, then exposes this context through CLI and MCP tools for AI agents to use.

## What ktx Does

- **Learns from company knowledge** — ingests wiki content, organizes it, removes duplicates, flags contradictions
- **Maps the data stack** — samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds a semantic layer** — combines raw tables and metrics through a join graph that resolves chasm and fan traps
- **Serves agents** — exposes CLI and MCP tools with semantic search across wiki and semantic-layer entities

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-local Installation

```bash
npm install --save-dev @kaelio/ktx
```

### Setup Wizard

```bash
ktx setup
```

The setup wizard:
- Creates or resumes a ktx project
- Configures LLM and embedding providers
- Sets up database connections
- Configures context sources (dbt, Looker, etc.)
- Builds initial context
- Installs agent integration

## Configuration

### Project Structure

```
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
llm:
  provider: anthropic
  model: claude-sonnet-4-6
embeddings:
  provider: openai
  model: text-embedding-3-small
connections:
  warehouse:
    type: postgresql
    host: localhost
    port: 5432
    database: analytics
    schema: public
    user: readonly_user
    ssl: true
context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
    connection: warehouse
  looker_main:
    type: looker
    project_id: my_project
    connection: warehouse
```

### Environment Variables

```bash
# LLM Provider
export ANTHROPIC_API_KEY=sk-ant-...
# or
export GOOGLE_VERTEX_PROJECT_ID=my-project
export GOOGLE_VERTEX_LOCATION=us-central1

# Embeddings Provider
export OPENAI_API_KEY=sk-...

# Database credentials (if not in ktx.yaml)
export WAREHOUSE_PASSWORD=...
export SNOWFLAKE_ACCOUNT=...
export BIGQUERY_CREDENTIALS_PATH=/path/to/service-account.json
```

## Key Commands

### Check Project Status

```bash
ktx status
```

Example output:
```
ktx project: /home/user/analytics
Project ready: yes
LLM ready: yes (claude-sonnet-4-6)
Embeddings ready: yes (text-embedding-3-small)
Databases configured: yes (warehouse)
Context sources configured: yes (dbt_main)
ktx context built: yes
Agent integration ready: yes (codex:project)
```

### Build Context

Ingest metadata from all configured sources:

```bash
ktx ingest
```

Ingest from specific connection:

```bash
ktx ingest --connection warehouse
```

Force re-ingestion:

```bash
ktx ingest --force
```

### Search Semantic Layer

```bash
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn rate"
```

Returns metric definitions, measures, dimensions, and their SQL.

### Search Wiki

```bash
ktx wiki "refund policy"
ktx wiki "how to calculate ARR"
ktx wiki "data quality issues"
```

Returns relevant wiki pages from global and user-scoped knowledge.

### MCP Server

Start the MCP server for AI agent clients:

```bash
ktx mcp start
```

With custom project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

The MCP server exposes tools for agents:
- `ktx_search_semantic_layer` — search metrics and dimensions
- `ktx_search_wiki` — search business knowledge
- `ktx_get_metric` — fetch full metric definition
- `ktx_list_connections` — list available data sources

## Using ktx with AI Agents

### Claude Code Integration

From your project directory, tell Claude Code:

```
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install and configure ktx in this project.
```

### Codex Integration

ktx auto-detects and integrates with Codex projects. After `ktx setup`, the MCP server is registered in `.codex/mcp.json`.

### Cursor Integration

Add to your Cursor MCP settings:

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

## Common Patterns

### Setting Up a New Analytics Project

```bash
# Create project directory
mkdir my-analytics-project
cd my-analytics-project

# Initialize ktx
ktx setup

# Follow prompts to:
# 1. Configure LLM (Anthropic, Google Vertex, or AI Gateway)
# 2. Configure embeddings (OpenAI, Google, or AI Gateway)
# 3. Add warehouse connection (PostgreSQL, Snowflake, BigQuery, etc.)
# 4. Add context sources (dbt, Looker, Metabase, Notion)
# 5. Build initial context

# Verify setup
ktx status

# Start MCP server for agents
ktx mcp start
```

### Adding dbt Context

```typescript
// ktx.yaml
connections:
  warehouse:
    type: postgresql
    host: localhost
    database: analytics

context_sources:
  dbt_main:
    type: dbt
    path: ./dbt  // Path to dbt project with manifest.json
    connection: warehouse
```

```bash
# Ingest dbt metadata
ktx ingest --connection warehouse
```

### Adding Looker Context

```yaml
# ktx.yaml
context_sources:
  looker_main:
    type: looker
    project_id: my_project
    connection: warehouse
    # ktx will parse LookML files for views, explores, dimensions, measures
```

### Querying from TypeScript

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function searchMetrics(query: string): Promise<string> {
  const { stdout } = await execAsync(`ktx sl "${query}"`);
  return stdout;
}

async function searchWiki(query: string): Promise<string> {
  const { stdout } = await execAsync(`ktx wiki "${query}"`);
  return stdout;
}

// Example usage
const revenueMetrics = await searchMetrics('revenue');
const refundPolicy = await searchWiki('refund policy');
```

### Building Semantic Layer YAML

ktx auto-generates semantic sources, but you can also define them manually:

```yaml
# semantic-layer/warehouse/revenue.yaml
version: 1
sources:
  - name: monthly_revenue
    description: Total revenue by month
    base_table: analytics.fact_orders
    dimensions:
      - name: month
        type: time
        sql: "DATE_TRUNC('month', order_date)"
    measures:
      - name: revenue
        type: sum
        sql: "amount"
        description: Sum of all order amounts
    metrics:
      - name: total_revenue
        type: simple
        measure: revenue
        description: Total revenue across all orders
```

### Adding Wiki Pages

Create markdown files in `wiki/global/`:

```bash
# wiki/global/refund-policy.md
---
title: Refund Policy
tags: [policy, finance]
---

# Refund Policy

Customers can request refunds within 30 days of purchase.
Refunds are processed within 5-7 business days.

## Revenue Impact

Refunds are subtracted from gross revenue to calculate net revenue.
See the `net_revenue` metric in the semantic layer.
```

ktx ingests and indexes these for semantic search.

### Using with Multiple Warehouses

```yaml
# ktx.yaml
connections:
  prod_warehouse:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS_PROD
    schema: PUBLIC
  
  dev_warehouse:
    type: postgresql
    host: localhost
    database: analytics_dev

context_sources:
  dbt_prod:
    type: dbt
    path: ./dbt
    connection: prod_warehouse
  
  dbt_dev:
    type: dbt
    path: ./dbt
    connection: dev_warehouse
```

```bash
# Ingest from specific warehouse
ktx ingest --connection prod_warehouse
ktx ingest --connection dev_warehouse
```

## Troubleshooting

### "Project ready: no" in ktx status

```bash
# Re-run setup wizard
ktx setup

# Or manually create ktx.yaml
touch ktx.yaml
```

### "LLM ready: no"

Ensure API keys are set:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
# or
export GOOGLE_VERTEX_PROJECT_ID=my-project
export GOOGLE_VERTEX_LOCATION=us-central1

ktx status
```

### "Context sources configured: no"

Add at least one context source in `ktx.yaml`:

```yaml
context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
    connection: warehouse
```

### Database Connection Fails

Check credentials and network access:

```bash
# Test PostgreSQL connection
psql -h localhost -U readonly_user -d analytics

# Test Snowflake connection
snowsql -a xy12345.us-east-1 -u readonly_user -d ANALYTICS

# ktx uses read-only operations only
```

### MCP Server Won't Start

Ensure project is properly configured:

```bash
ktx status
# All checks should be "yes"

# If agent integration shows "no", re-run setup
ktx setup
```

### Ingestion Takes Too Long

Limit table sampling:

```yaml
# ktx.yaml
connections:
  warehouse:
    type: postgresql
    # ... connection details
    sample_limit: 100  # Limit rows sampled per table
    skip_tables:
      - logs_*
      - temp_*
```

### Semantic Search Returns No Results

Rebuild embeddings:

```bash
ktx ingest --force
```

Verify embeddings provider is configured:

```bash
export OPENAI_API_KEY=sk-...
ktx status
```

### Agent Can't Find ktx Tools

Ensure MCP server is running:

```bash
ktx mcp start --project-dir /absolute/path/to/project
```

For Codex/Claude Code, verify `.codex/mcp.json` or equivalent config exists.

## Advanced Usage

### Custom LLM Provider

```yaml
# ktx.yaml
llm:
  provider: ai_gateway
  endpoint: https://gateway.example.com/v1
  model: claude-sonnet-4
```

```bash
export AI_GATEWAY_API_KEY=...
```

### Python API (via ktx-sl)

```python
from ktx_sl import SemanticLayerEngine

engine = SemanticLayerEngine(project_dir="/path/to/project")

# Query metrics
results = engine.query_metric(
    metric_name="total_revenue",
    dimensions=["month"],
    filters={"year": 2024}
)
```

### Telemetry Opt-out

```bash
export KTX_TELEMETRY_DISABLED=1
```

Or in `ktx.yaml`:

```yaml
telemetry:
  enabled: false
```

## Supported Data Sources

### Databases
- PostgreSQL
- Snowflake
- BigQuery
- ClickHouse
- MySQL
- SQL Server
- SQLite

### Semantic Layers
- dbt (via manifest.json)
- MetricFlow
- LookML (Looker)

### BI Tools
- Looker
- Metabase

### Wikis
- Notion
- Local markdown files

## Best Practices

1. **Keep context fresh** — run `ktx ingest` regularly or after schema changes
2. **Commit semantic-layer/** — version control your metric definitions
3. **Use read-only database credentials** — ktx never writes to your warehouse
4. **Organize wiki pages** — use clear titles and tags for better search results
5. **Review contradictions** — ktx flags conflicts between dbt, Looker, and wiki sources
6. **Test metrics with agents** — verify AI agents use correct definitions after ingestion
