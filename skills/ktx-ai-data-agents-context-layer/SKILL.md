---
name: ktx-ai-data-agents-context-layer
description: Context layer for data agents - builds semantic layer, wiki, and warehouse metadata to enable accurate AI-powered analytics queries
triggers:
  - set up ktx for data agent access to our warehouse
  - configure ktx semantic layer for AI queries
  - build ktx context from dbt and warehouse metadata
  - integrate ktx with Claude Code for analytics
  - search ktx wiki or semantic layer
  - troubleshoot ktx ingestion or MCP server
  - add database connection to ktx project
  - query metrics through ktx semantic layer
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It automatically builds a semantic layer from approved metric definitions, detects joinable columns, ingests business knowledge from wikis/dbt/BI tools, and exposes everything through CLI and MCP (Model Context Protocol) for agent execution.

Unlike general-purpose agents that re-explore your warehouse on every question or traditional semantic layers that require constant manual upkeep, ktx learns from company knowledge, maps your data stack automatically, and serves combined context to agents at runtime.

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install --save-dev @kaelio/ktx
```

Or use with `npx`:

```bash
npx @kaelio/ktx setup
```

## Quick Start

### Initial Setup

```bash
# Initialize or resume a ktx project
ktx setup

# Check project readiness
ktx status
```

The `ktx setup` command:
- Creates or resumes a local ktx project
- Configures LLM and embedding providers
- Sets up database connections
- Configures context sources (dbt, Looker, Metabase, Notion)
- Builds initial context
- Installs agent integration

### Expected Status Output

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
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources (commit)
├── wiki/global/                     # Shared business context (commit)
├── wiki/user/<user-id>/             # User-scoped notes (commit)
├── raw-sources/<connection-id>/     # Ingest artifacts and reports (commit)
└── .ktx/                            # Local state and secrets (git-ignore)
```

**Commit**: `ktx.yaml`, `semantic-layer/`, `wiki/`, `raw-sources/`  
**Git-ignore**: `.ktx/`

## Configuration

### ktx.yaml Structure

```yaml
project:
  name: my-analytics-project
  version: 1.0.0

llm:
  provider: anthropic  # or google-vertex, ai-gateway, claude-code
  model: claude-sonnet-4-6
  # API key stored in .ktx/secrets.json

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key stored in .ktx/secrets.json

databases:
  warehouse:
    type: postgres  # or snowflake, bigquery, clickhouse, mysql, mssql, sqlite
    host: db.example.com
    port: 5432
    database: analytics
    schema: public
    # credentials stored in .ktx/secrets.json

context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./target/manifest.json
  
  looker_main:
    type: looker
    base_url: https://company.looker.com
    # API credentials in .ktx/secrets.json
  
  notion_wiki:
    type: notion
    # API token in .ktx/secrets.json
```

### Environment Variables

ktx respects these environment variables:

```bash
export KTX_PROJECT_DIR=/path/to/project
export ANTHROPIC_API_KEY=your-key-here
export OPENAI_API_KEY=your-key-here
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
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

# Force re-ingestion (skip cache)
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, entities)
ktx sl "revenue"
ktx sl "customer lifetime value"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "how to calculate churn"

# List all semantic sources
ktx sl list

# Show specific metric details
ktx sl show "monthly_recurring_revenue"
```

### MCP Server for Agents

```bash
# Start MCP server (required for agent integration)
ktx mcp start

# Start with specific project
ktx mcp start --project-dir /path/to/project

# Check MCP status
ktx mcp status
```

### Project Management

```bash
# Validate project configuration
ktx validate

# Show detailed status
ktx status --verbose

# Clear all context and rebuild
ktx clear && ktx ingest
```

## Using ktx with AI Agents

### Claude Code / Codex Integration

Once `ktx setup` completes, the agent integration is automatic. Ensure the MCP server is running:

```bash
ktx mcp start
```

Then ask the agent:

```text
Query the warehouse using ktx to find monthly recurring revenue for the last 6 months
```

```text
Search ktx wiki for our customer segmentation strategy
```

### Cursor / OpenCode Integration

Add ktx as an MCP server in your agent configuration, then use natural language:

```text
Use ktx to explore available metrics related to user engagement
```

## Real-World Examples

### Defining a Semantic Metric

Create a file in `semantic-layer/warehouse/metrics.yaml`:

```yaml
metrics:
  - name: monthly_recurring_revenue
    description: Sum of all active subscription revenue for the month
    type: simple
    sql: "SUM(subscription_amount)"
    base_entity: subscription
    filters:
      - field: status
        operator: equals
        value: active
    dimensions:
      - plan_type
      - customer_segment
    
  - name: customer_ltv
    description: Customer lifetime value - total revenue per customer
    type: derived
    sql: |
      SUM(order_total) / COUNT(DISTINCT customer_id)
    base_entity: order
    dimensions:
      - acquisition_channel
      - cohort_month
```

### Adding Wiki Context

Create a file in `wiki/global/analytics-definitions.md`:

```markdown
# Analytics Definitions

## Revenue Recognition

We recognize revenue on a cash basis when payment is received, not when 
the invoice is sent.

## Customer Churn

A customer is considered churned if they have not had an active subscription 
for 90 consecutive days. Paused subscriptions do not count as churn.

## Cohort Analysis

Customer cohorts are defined by their first purchase month. Use the 
`first_order_date` field to group customers into cohorts.
```

After adding wiki content, re-ingest:

```bash
ktx ingest --source wiki
```

### Querying from TypeScript/Node

```typescript
import { execSync } from 'child_process';

// Search semantic layer
const metrics = JSON.parse(
  execSync('ktx sl "revenue" --json', { encoding: 'utf-8' })
);

console.log('Available revenue metrics:', metrics);

// Search wiki
const wikiResults = JSON.parse(
  execSync('ktx wiki "churn" --json', { encoding: 'utf-8' })
);

console.log('Wiki pages about churn:', wikiResults);
```

### Programmatic Context Building

```typescript
import { KtxProject } from '@kaelio/ktx';

async function buildContext() {
  const project = await KtxProject.load('/path/to/project');
  
  // Ingest all sources
  await project.ingest({ force: false });
  
  // Query semantic layer
  const revenueMetrics = await project.semanticLayer.search('revenue');
  
  console.log('Found metrics:', revenueMetrics.map(m => m.name));
}

buildContext();
```

### Integrating with dbt

Ensure your dbt project has a compiled `manifest.json`:

```bash
cd /path/to/dbt-project
dbt compile
```

Configure in `ktx.yaml`:

```yaml
context_sources:
  dbt_main:
    type: dbt
    manifest_path: ../dbt-project/target/manifest.json
    include_models: true
    include_sources: true
    include_metrics: true
```

Then ingest:

```bash
ktx ingest --source dbt_main
```

ktx will:
- Extract model descriptions and column definitions
- Import dbt metrics as semantic-layer metrics
- Detect relationships from `ref()` and foreign keys
- Flag contradictions between dbt docs and warehouse metadata

## Common Patterns

### Multi-Warehouse Setup

```yaml
databases:
  production:
    type: snowflake
    account: xy12345.us-east-1
    database: PROD_DB
    warehouse: COMPUTE_WH
  
  staging:
    type: postgres
    host: staging-db.internal
    database: staging_analytics
```

Ingest separately:

```bash
ktx ingest --connection production
ktx ingest --connection staging
```

### Custom Semantic Entities

Define entities (e.g., tables or views) in `semantic-layer/warehouse/entities.yaml`:

```yaml
entities:
  - name: customer
    description: Customer dimension table
    table: public.customers
    primary_key: customer_id
    columns:
      - name: customer_id
        type: integer
      - name: email
        type: string
      - name: plan_type
        type: string
      - name: created_at
        type: timestamp

  - name: subscription
    description: Active and historical subscriptions
    table: public.subscriptions
    primary_key: subscription_id
    foreign_keys:
      - column: customer_id
        references_entity: customer
        references_column: customer_id
```

### Searching with Filters

```bash
# Search only metrics (not dimensions or entities)
ktx sl "conversion" --type metric

# Search with semantic similarity threshold
ktx sl "customer value" --threshold 0.7

# Full-text search in wiki
ktx wiki "refund AND policy"
```

## Troubleshooting

### MCP Server Not Starting

**Symptom**: `ktx mcp start` exits immediately or shows connection errors.

**Solutions**:

```bash
# Check project status first
ktx status

# Ensure LLM and embeddings are configured
ktx setup

# Verify project directory
ktx mcp start --project-dir /path/to/project

# Check for port conflicts (default 3000)
lsof -i :3000

# Start on different port
ktx mcp start --port 3001
```

### Ingestion Fails for Database

**Symptom**: `ktx ingest --connection warehouse` fails with authentication or timeout errors.

**Solutions**:

```bash
# Verify credentials in .ktx/secrets.json
cat .ktx/secrets.json

# Test connection manually
psql -h db.example.com -U your_user -d analytics

# Check firewall / network access
telnet db.example.com 5432

# Use read-only user for safety
# ktx never writes, but explicit read-only is best practice
```

### No Results from Search

**Symptom**: `ktx sl "revenue"` returns empty results.

**Solutions**:

```bash
# Ensure context is built
ktx status

# Re-ingest if needed
ktx ingest --force

# Check semantic-layer directory
ls semantic-layer/warehouse/

# Verify embeddings are configured
ktx status --verbose
```

### dbt Manifest Not Found

**Symptom**: `ktx ingest --source dbt_main` reports missing manifest.

**Solutions**:

```bash
# Compile dbt project first
cd /path/to/dbt-project
dbt compile

# Verify manifest path in ktx.yaml
cat ktx.yaml | grep manifest_path

# Use absolute path if needed
context_sources:
  dbt_main:
    type: dbt
    manifest_path: /absolute/path/to/target/manifest.json
```

### Contradictions Flagged

**Symptom**: ktx reports contradictions between dbt docs and warehouse metadata.

**Expected behavior**: ktx flags these for human review. Check:

```bash
# Review flagged contradictions
cat raw-sources/warehouse/contradictions.json

# Common causes:
# - dbt docs out of sync with warehouse
# - Recent schema changes not yet in dbt
# - Typos in column names or descriptions
```

Resolve by updating dbt docs or warehouse schema, then re-ingest.

### Agent Can't Access ktx

**Symptom**: Agent says "ktx not found" or doesn't see semantic layer.

**Solutions**:

```bash
# Ensure MCP server is running
ktx mcp start

# Check agent integration status
ktx status | grep "Agent integration"

# Re-run setup to refresh integration
ktx setup

# Verify agent is configured to use MCP
# (check agent settings for MCP server configuration)
```

## Advanced Usage

### Custom LLM Configuration

```yaml
llm:
  provider: ai-gateway
  endpoint: https://gateway.example.com/v1
  model: custom-model-v2
  max_tokens: 4096
  temperature: 0.1
```

### Scripting with ktx

```bash
#!/bin/bash
# Daily context refresh script

export KTX_PROJECT_DIR=/path/to/project

# Pull latest dbt changes
cd /path/to/dbt-project && git pull && dbt compile

# Rebuild ktx context
cd $KTX_PROJECT_DIR
ktx ingest --force

# Restart MCP server
pkill -f "ktx mcp" || true
ktx mcp start --daemon
```

### Filtering Ingestion

```yaml
databases:
  warehouse:
    type: postgres
    # ... connection details ...
    include_schemas:
      - public
      - analytics
    exclude_tables:
      - tmp_*
      - staging_*
    sample_size: 1000  # rows to sample per table
```

## Key Resources

- **Documentation**: https://docs.kaelio.com/ktx/docs/
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Setup Guide**: https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues**: https://github.com/Kaelio/ktx/issues

## Security Notes

- ktx connections are **read-only** by design
- Credentials stored in `.ktx/secrets.json` (git-ignored)
- No data sent to hosted services - only to your configured LLM provider
- Use environment variables for CI/CD: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, etc.
