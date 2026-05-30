---
name: ktx-ai-data-agents
description: Context layer for AI data agents - query warehouses accurately with semantic layers, metrics, and wiki knowledge through MCP
triggers:
  - set up ktx for my data warehouse
  - configure ktx semantic layer for AI agents
  - use ktx to query my database with Claude
  - integrate ktx with my analytics workflow
  - build context from dbt and warehouse metadata
  - connect ktx to Snowflake/BigQuery/Postgres
  - search ktx wiki and semantic layer
  - configure ktx MCP server for agents
---

# ktx AI Data Agents Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

**ktx** is an executable context layer that teaches AI agents how to query data warehouses accurately. It automatically builds and maintains:

- **Semantic layer** with approved metric definitions, join graphs, and fan/chasm trap resolution
- **Wiki knowledge** from Notion, dbt docs, BI tools, and team documentation
- **Warehouse metadata** including table schemas, joinable columns, and usage patterns
- **MCP integration** for Claude Code, Codex, Cursor, and other AI agents

Unlike general-purpose agents that reinvent SQL logic on every query, ktx provides agents with canonical definitions and business context through a searchable interface.

## Installation

```bash
# Install globally
npm install -g @kaelio/ktx

# Or use npx
npx @kaelio/ktx setup
```

**Requirements:**
- Node.js 18+
- Access to a SQL warehouse (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, or SQLite)
- LLM API key (Anthropic, Google Vertex AI) or Claude Pro/Max subscription

## Quick Start

### Initial Setup

```bash
# Create or resume a ktx project in current directory
ktx setup

# Check project status
ktx status
```

The `ktx setup` wizard will:
1. Create `ktx.yaml` configuration
2. Configure LLM and embedding providers
3. Set up database connections
4. Configure context sources (dbt, Looker, Metabase, Notion)
5. Build initial context
6. Install agent integration (MCP)

### Project Structure

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML metric/dimension definitions
├── wiki/global/                     # Shared business knowledge
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state (git-ignored)
```

**Commit:** `ktx.yaml`, `semantic-layer/`, `wiki/`  
**Ignore:** `.ktx/`

## Configuration

### ktx.yaml Structure

```yaml
version: 1.0
name: my-analytics-project
llm:
  provider: anthropic
  model: claude-sonnet-4-6
embeddings:
  provider: openai
  model: text-embedding-3-small
connections:
  - id: warehouse
    type: postgres
    host: ${DATABASE_HOST}
    port: 5432
    database: analytics
    user: ${DATABASE_USER}
    password: ${DATABASE_PASSWORD}
    ssl: true
context_sources:
  - id: dbt_main
    type: dbt
    connection_id: warehouse
    manifest_path: ./target/manifest.json
    catalog_path: ./target/catalog.json
  - id: notion_docs
    type: notion
    token: ${NOTION_TOKEN}
    page_ids:
      - 3fa85f64-5717-4562-b3fc-2c963f66afa6
```

### Environment Variables

```bash
# LLM providers
export ANTHROPIC_API_KEY=your-key-here
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Embeddings
export OPENAI_API_KEY=your-key-here

# Database credentials
export DATABASE_HOST=warehouse.example.com
export DATABASE_USER=readonly_user
export DATABASE_PASSWORD=secure-password

# Context sources
export NOTION_TOKEN=secret_notion_token
export LOOKER_API_TOKEN=looker-token
```

### Supported Databases

**PostgreSQL:**
```yaml
connections:
  - id: postgres_warehouse
    type: postgres
    host: ${PG_HOST}
    port: 5432
    database: analytics
    user: ${PG_USER}
    password: ${PG_PASSWORD}
    ssl: true
```

**Snowflake:**
```yaml
connections:
  - id: snowflake_warehouse
    type: snowflake
    account: ${SNOWFLAKE_ACCOUNT}
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
```

**BigQuery:**
```yaml
connections:
  - id: bigquery_warehouse
    type: bigquery
    project_id: ${GCP_PROJECT_ID}
    dataset: analytics
    credentials_path: ${GOOGLE_APPLICATION_CREDENTIALS}
```

## Core Commands

### Context Management

```bash
# Build context from all configured sources
ktx ingest

# Build context for specific connection
ktx ingest --connection warehouse

# Force rebuild ignoring cache
ktx ingest --force

# Dry run to preview changes
ktx ingest --dry-run
```

### Search and Query

```bash
# Search semantic layer (metrics, dimensions)
ktx sl "revenue"
ktx sl "customer churn rate"

# Search wiki knowledge
ktx wiki "refund policy"
ktx wiki "data retention rules"

# Get detailed entity information
ktx describe metric monthly_recurring_revenue
ktx describe dimension customer_segment
```

### MCP Server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status

# Stop MCP server
ktx mcp stop
```

### Project Management

```bash
# Validate configuration
ktx validate

# Show project status
ktx status

# List all configured connections
ktx connections list

# Test connection
ktx connections test warehouse
```

## Semantic Layer Usage

### Defining Metrics

Create `semantic-layer/warehouse/metrics.yaml`:

```yaml
metrics:
  - name: monthly_recurring_revenue
    label: Monthly Recurring Revenue
    description: Sum of all active subscription values normalized to monthly
    type: sum
    sql: |
      CASE 
        WHEN billing_period = 'monthly' THEN amount
        WHEN billing_period = 'annual' THEN amount / 12
      END
    table: subscriptions
    filters:
      - column: status
        operator: equals
        value: 'active'
    dimensions:
      - customer_segment
      - plan_type
    timestamp_column: created_at
    
  - name: customer_count
    label: Active Customers
    description: Count of distinct active customer IDs
    type: count_distinct
    sql: customer_id
    table: subscriptions
    filters:
      - column: status
        operator: equals
        value: 'active'
```

### Defining Dimensions

Create `semantic-layer/warehouse/dimensions.yaml`:

```yaml
dimensions:
  - name: customer_segment
    label: Customer Segment
    description: Business vs. enterprise customer classification
    type: categorical
    sql: |
      CASE 
        WHEN annual_revenue > 100000 THEN 'Enterprise'
        WHEN annual_revenue > 10000 THEN 'Business'
        ELSE 'Startup'
      END
    table: customers
    
  - name: signup_date
    label: Signup Date
    description: Date customer first signed up
    type: time
    sql: DATE(created_at)
    table: customers
    granularities:
      - day
      - week
      - month
      - quarter
      - year
```

### Join Configuration

Define table relationships in `semantic-layer/warehouse/joins.yaml`:

```yaml
joins:
  - left_table: subscriptions
    right_table: customers
    type: left
    conditions:
      - left_column: customer_id
        right_column: id
    
  - left_table: subscriptions
    right_table: plans
    type: left
    conditions:
      - left_column: plan_id
        right_column: id
```

## Wiki Management

### Creating Wiki Pages

```bash
# Create global wiki page
cat > wiki/global/refund-policy.md << 'EOF'
# Refund Policy

## Overview
Customers can request refunds within 30 days of purchase.

## Rules
- Full refund: < 7 days
- Prorated refund: 7-30 days
- No refund: > 30 days

## Database Impact
Refunds update `transactions.status` to 'refunded' and create 
negative entries in `revenue_events`.
EOF

# Create user-scoped note
mkdir -p wiki/user/$(whoami)
cat > wiki/user/$(whoami)/analysis-notes.md << 'EOF'
# Analysis Notes

## 2025-05 Revenue Analysis
Found discrepancy in EMEA revenue - missing Stripe events.
Tracked in JIRA-1234.
EOF
```

### Ingesting from Notion

```yaml
context_sources:
  - id: product_docs
    type: notion
    token: ${NOTION_TOKEN}
    page_ids:
      - 3fa85f64-5717-4562-b3fc-2c963f66afa6  # Product Roadmap
      - 7c9e6679-7425-40de-944b-e07fc1f90ae7  # Data Dictionary
    recursive: true  # Include child pages
```

Run `ktx ingest` to sync Notion content into `wiki/global/`.

## Agent Integration (MCP)

### Starting MCP Server

```bash
# In your ktx project directory
ktx mcp start

# Or specify project location
ktx mcp start --project-dir ~/my-analytics
```

### Claude Desktop Configuration

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

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

Restart Claude Desktop to load the MCP server.

### Using ktx from Claude

Once configured, you can prompt Claude:

```
What was our MRR last month by customer segment?
```

Claude will use ktx MCP tools to:
1. Search semantic layer for `monthly_recurring_revenue` metric
2. Find relevant dimensions (`customer_segment`)
3. Retrieve approved SQL definitions
4. Execute query using canonical metric logic

### Available MCP Tools

- `ktx_search_semantic_layer` - Search metrics, dimensions, tables
- `ktx_search_wiki` - Search business knowledge and documentation
- `ktx_describe_entity` - Get detailed entity information
- `ktx_list_connections` - List available database connections
- `ktx_get_context_summary` - Get project context overview

## Common Patterns

### Initial Project Setup Workflow

```bash
# 1. Navigate to analytics project
cd ~/projects/analytics

# 2. Run setup wizard
ktx setup
# Select: Anthropic Claude, OpenAI embeddings, configure Postgres connection

# 3. Add dbt context source
# Edit ktx.yaml to add dbt manifest/catalog paths

# 4. Build context
ktx ingest

# 5. Verify
ktx status
ktx sl "revenue"

# 6. Start MCP for agents
ktx mcp start
```

### Incremental Context Updates

```bash
# After dbt run or schema changes
dbt run
dbt docs generate
ktx ingest --connection warehouse

# After updating wiki pages
ktx ingest --source notion_docs

# Check for conflicts or issues
ktx validate
```

### Searching Before Agent Queries

```bash
# Find available metrics
ktx sl "churn"

# Output:
# Metrics:
#   - customer_churn_rate (Monthly customer churn percentage)
#   - mrr_churn (Monthly recurring revenue lost to churn)
# 
# Dimensions:
#   - churn_reason (Categorical reason for cancellation)

# Get metric details
ktx describe metric customer_churn_rate

# Now prompt agent with context:
# "Calculate customer_churn_rate for Q1 2025 by churn_reason"
```

### Multi-Warehouse Setup

```yaml
connections:
  - id: production
    type: snowflake
    account: ${SNOWFLAKE_PROD_ACCOUNT}
    # ... prod credentials
    
  - id: staging
    type: snowflake
    account: ${SNOWFLAKE_STAGING_ACCOUNT}
    # ... staging credentials

context_sources:
  - id: dbt_prod
    type: dbt
    connection_id: production
    manifest_path: ./prod/target/manifest.json
    
  - id: dbt_staging
    type: dbt
    connection_id: staging
    manifest_path: ./staging/target/manifest.json
```

```bash
# Ingest specific warehouse
ktx ingest --connection production

# Search scoped to connection
ktx sl "revenue" --connection production
```

## Troubleshooting

### MCP Server Not Starting

**Check project directory:**
```bash
ktx status
# If shows "ktx mcp start --project-dir ...", copy and run that command
```

**Verify ktx.yaml exists:**
```bash
ls ktx.yaml
# If missing, run: ktx setup
```

**Check Claude Desktop config:**
```bash
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json
# Ensure path is absolute, not relative
```

### Database Connection Issues

```bash
# Test connection directly
ktx connections test warehouse

# Common fixes:
# - Verify environment variables are set
# - Check firewall/VPN for warehouse access
# - Ensure user has SELECT permissions
# - For Snowflake, verify warehouse is running
```

### Context Ingestion Failures

```bash
# Enable verbose logging
ktx ingest --verbose

# Check specific source
ktx ingest --source dbt_main --verbose

# Validate configuration
ktx validate

# Common issues:
# - dbt manifest/catalog paths incorrect
# - Missing environment variables
# - LLM API rate limits (retry with backoff)
```

### Semantic Layer Errors

**Undefined metric:**
```bash
# List all metrics
ktx sl "*" --type metric

# Ensure YAML is valid
cat semantic-layer/warehouse/metrics.yaml
ktx validate
```

**Join graph errors:**
```bash
# Check join definitions
cat semantic-layer/warehouse/joins.yaml

# Common issues:
# - Missing join between tables
# - Ambiguous join paths (fan trap)
# - Incorrect column names
```

### Agent Not Finding Context

**Restart MCP server:**
```bash
ktx mcp stop
ktx mcp start
```

**Rebuild context:**
```bash
ktx ingest --force
```

**Check search results:**
```bash
ktx sl "your search term"
ktx wiki "your search term"
# If empty, context may not have been ingested
```

## TypeScript API Usage

For programmatic usage in Node.js:

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load project
const project = await KtxProject.load('/path/to/project');

// Search semantic layer
const metrics = await project.searchSemanticLayer('revenue', {
  type: 'metric',
  limit: 10
});

// Search wiki
const wikiPages = await project.searchWiki('refund policy', {
  scope: 'global',
  limit: 5
});

// Get entity details
const metric = await project.describeEntity('metric', 'monthly_recurring_revenue');

// Execute ingestion
await project.ingest({
  connectionId: 'warehouse',
  force: false
});
```

## Best Practices

1. **Version control semantic layer:** Commit `semantic-layer/` and `wiki/global/` to track metric definitions
2. **Use environment variables:** Never hardcode credentials in `ktx.yaml`
3. **Regular ingestion:** Run `ktx ingest` after dbt runs or schema changes
4. **Descriptive names:** Use clear metric/dimension names that match business language
5. **Document assumptions:** Add descriptions to all semantic layer entities
6. **Test connections:** Use `ktx connections test` before ingestion
7. **User wiki for ephemeral notes:** Keep temporary analysis in `wiki/user/`
8. **Review ingest logs:** Check for warnings about duplicate or conflicting definitions

## Advanced Configuration

### Custom LLM Configuration

```yaml
llm:
  provider: vertex
  project_id: ${GCP_PROJECT_ID}
  location: us-central1
  model: claude-3-5-sonnet@20241022
  max_tokens: 8192
  temperature: 0.0
```

### Embedding Configuration

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-large
  dimensions: 1536
  batch_size: 100
```

### Ingest Scheduling

```bash
# Cron example: Daily at 2 AM
0 2 * * * cd /path/to/project && ktx ingest --connection warehouse >> /var/log/ktx.log 2>&1
```

### Custom Python Queries

ktx includes a Python semantic layer query planner:

```python
from ktx_sl import SemanticLayer

# Load semantic layer
sl = SemanticLayer.from_project("/path/to/project")

# Build query
query = sl.query(
    metrics=["monthly_recurring_revenue"],
    dimensions=["customer_segment", "signup_month"],
    filters=[
        {"dimension": "signup_date", "operator": ">=", "value": "2025-01-01"}
    ],
    order_by=[{"metric": "monthly_recurring_revenue", "desc": True}]
)

# Get SQL
sql = query.to_sql()
print(sql)
```

## Resources

- **Documentation:** https://docs.kaelio.com/ktx
- **CLI Reference:** https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Setup:** https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
- **Slack Community:** https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues:** https://github.com/Kaelio/ktx/issues
