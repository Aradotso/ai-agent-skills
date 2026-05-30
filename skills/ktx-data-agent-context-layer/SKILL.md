---
name: ktx-data-agent-context-layer
description: Install and configure ktx, the self-improving context layer that teaches AI agents to query data warehouses accurately with approved metrics, semantic layer, and business knowledge.
triggers:
  - set up ktx for data analysis
  - configure ktx semantic layer
  - install ktx context layer for warehouse
  - use ktx to query database with approved metrics
  - integrate ktx with this data project
  - build ktx context from dbt and warehouse
  - search ktx semantic layer and wiki
  - configure ktx MCP server for agents
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It automatically builds approved metric definitions, detects joinable columns, ingests business knowledge from wikis and dbt, and exposes everything through CLI and MCP interfaces.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, dbt docs, LookML, Metabase, and Notion; organizes it and flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, resolves fan/chasm traps
- **Builds semantic layer**: Combines raw tables and high-level metrics through a join graph
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search
- **Read-only by design**: Never writes to your warehouse

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite. Integrates with dbt, MetricFlow, LookML, Looker, Metabase, Notion.

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Setup

```bash
# Create or resume ktx project in current directory
ktx setup

# Check project readiness
ktx status
```

### Programmatic Installation (TypeScript)

```typescript
import { execSync } from 'child_process';

// Install ktx globally
execSync('npm install -g @kaelio/ktx', { stdio: 'inherit' });

// Initialize project
execSync('ktx setup', { 
  cwd: process.cwd(),
  stdio: 'inherit' 
});
```

## Project Structure

After `ktx setup`, your project will contain:

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local and git-ignored.

## Configuration

### ktx.yaml Structure

```yaml
version: 1.0.0
projectName: my-analytics-project

# LLM Provider Configuration
llmProvider:
  type: anthropic  # or 'google-vertex', 'ai-gateway', 'claude-code'
  model: claude-sonnet-4-6
  # API key set via environment variable ANTHROPIC_API_KEY

# Embedding Provider
embeddingProvider:
  type: openai
  model: text-embedding-3-small
  # API key set via environment variable OPENAI_API_KEY

# Database Connections
connections:
  - id: warehouse
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    # Credentials set via WAREHOUSE_USER and WAREHOUSE_PASSWORD

  - id: snowflake_prod
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: PROD
    schema: PUBLIC
    # Credentials via SNOWFLAKE_USER and SNOWFLAKE_PASSWORD

# Context Sources
contextSources:
  - id: dbt_main
    type: dbt
    manifestPath: ./target/manifest.json
    catalogPath: ./target/catalog.json
    
  - id: notion_wiki
    type: notion
    # Token via NOTION_INTEGRATION_TOKEN
    databaseIds:
      - a1b2c3d4e5f6
      
  - id: metabase_analytics
    type: metabase
    url: https://metabase.company.com
    # Credentials via METABASE_USER and METABASE_PASSWORD
```

### Environment Variables

Store credentials securely:

```bash
# LLM Providers
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Database Connections
export WAREHOUSE_USER=analytics_user
export WAREHOUSE_PASSWORD=...
export SNOWFLAKE_USER=...
export SNOWFLAKE_PASSWORD=...

# Context Sources
export NOTION_INTEGRATION_TOKEN=secret_...
export METABASE_USER=...
export METABASE_PASSWORD=...

# Optional: Specify project directory
export KTX_PROJECT_DIR=/path/to/project
```

## Key Commands

### Setup and Status

```bash
# Interactive setup wizard
ktx setup

# Check project configuration and readiness
ktx status

# Example output:
# ktx project: /home/user/analytics
# Project ready: yes
# LLM ready: yes (claude-sonnet-4-6)
# Databases configured: yes (warehouse)
# ktx context built: yes
# Agent integration ready: yes (codex:project)
```

### Building Context

```bash
# Build context for all configured connections
ktx ingest

# Build context for specific connection
ktx ingest --connection warehouse

# Force rebuild even if up-to-date
ktx ingest --force

# Dry run to see what would be ingested
ktx ingest --dry-run
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, tables)
ktx sl "revenue"
ktx sl "customer churn rate"
ktx sl "monthly active users"

# Search wiki content (business knowledge)
ktx wiki "refund policy"
ktx wiki "fiscal year definition"
ktx wiki "customer segmentation"

# Limit results
ktx sl "orders" --limit 5
ktx wiki "pricing" --limit 3
```

### MCP Server

```bash
# Start MCP server for agent integration
ktx mcp start

# Specify project directory
ktx mcp start --project-dir /path/to/project

# Check MCP status
ktx mcp status

# Stop MCP server
ktx mcp stop
```

### Maintenance

```bash
# Clear all cached context
ktx clear-cache

# Clear cache for specific connection
ktx clear-cache --connection warehouse

# Validate configuration without building
ktx validate

# Show version
ktx --version
```

## TypeScript API Usage

### Programmatic Context Search

```typescript
import { KtxClient } from '@kaelio/ktx';

// Initialize client
const ktx = new KtxClient({
  projectDir: process.cwd(),
  // Credentials from environment variables
});

// Search semantic layer
async function searchMetrics(query: string) {
  const results = await ktx.searchSemanticLayer(query, {
    limit: 10,
    includeScore: true
  });
  
  for (const result of results) {
    console.log(`${result.name} (score: ${result.score})`);
    console.log(`  Type: ${result.type}`);
    console.log(`  SQL: ${result.sql}`);
    console.log(`  Description: ${result.description}`);
  }
  
  return results;
}

// Search wiki
async function searchWiki(query: string) {
  const results = await ktx.searchWiki(query, {
    limit: 5,
    namespace: 'global'  // or 'user/<user-id>'
  });
  
  for (const result of results) {
    console.log(`${result.title}`);
    console.log(`  Path: ${result.path}`);
    console.log(`  Excerpt: ${result.excerpt}`);
  }
  
  return results;
}

// Usage
await searchMetrics('monthly revenue');
await searchWiki('customer retention strategy');
```

### Building Context Programmatically

```typescript
import { KtxClient } from '@kaelio/ktx';

async function buildContext() {
  const ktx = new KtxClient({
    projectDir: process.cwd()
  });
  
  // Ingest all sources
  await ktx.ingest({
    force: false,
    dryRun: false
  });
  
  // Ingest specific connection
  await ktx.ingest({
    connectionId: 'warehouse',
    force: true
  });
  
  console.log('Context built successfully');
}

await buildContext();
```

### Validating Configuration

```typescript
import { KtxClient } from '@kaelio/ktx';

async function validateProject() {
  const ktx = new KtxClient({
    projectDir: process.cwd()
  });
  
  const status = await ktx.getStatus();
  
  console.log('Project ready:', status.projectReady);
  console.log('LLM ready:', status.llmReady);
  console.log('Embeddings ready:', status.embeddingsReady);
  console.log('Databases configured:', status.databasesConfigured);
  console.log('Context built:', status.contextBuilt);
  console.log('Agent integration:', status.agentIntegration);
  
  if (!status.projectReady) {
    console.error('Issues:', status.issues);
  }
  
  return status.projectReady;
}

await validateProject();
```

## Common Patterns

### Setting Up ktx for dbt Projects

```bash
# Navigate to dbt project
cd my-dbt-project

# Initialize ktx
ktx setup

# During setup, configure:
# 1. LLM provider (Anthropic recommended)
# 2. Embedding provider (OpenAI recommended)
# 3. Database connection (your data warehouse)
# 4. dbt context source (point to target/manifest.json)

# Build initial context
ktx ingest

# Search dbt models
ktx sl "customer lifetime value"
```

### Integrating with Multiple Warehouses

```yaml
# ktx.yaml
connections:
  - id: prod_warehouse
    type: snowflake
    account: prod.us-east-1
    database: ANALYTICS
    
  - id: staging_warehouse
    type: postgres
    host: staging-db.internal
    database: analytics_staging
    
  - id: local_dev
    type: postgres
    host: localhost
    database: dev_analytics
```

```bash
# Ingest each warehouse
ktx ingest --connection prod_warehouse
ktx ingest --connection staging_warehouse
ktx ingest --connection local_dev

# Search across all warehouses
ktx sl "daily_revenue"
```

### Creating Semantic Layer Definitions

Create `semantic-layer/<connection-id>/metrics.yaml`:

```yaml
metrics:
  - name: monthly_recurring_revenue
    description: Total MRR from active subscriptions
    type: metric
    sql: |
      SELECT 
        DATE_TRUNC('month', subscription_start) as month,
        SUM(monthly_amount) as mrr
      FROM subscriptions
      WHERE status = 'active'
      GROUP BY 1
    dimensions:
      - month
    measures:
      - mrr
    grain: monthly
    
  - name: customer_churn_rate
    description: Percentage of customers who churned in period
    type: metric
    sql: |
      SELECT
        DATE_TRUNC('month', churn_date) as month,
        COUNT(DISTINCT customer_id)::float / 
        LAG(COUNT(DISTINCT customer_id)) OVER (ORDER BY DATE_TRUNC('month', churn_date)) as churn_rate
      FROM customers
      WHERE churned = true
      GROUP BY 1
    dimensions:
      - month
    measures:
      - churn_rate
    grain: monthly
```

```bash
# Rebuild to pick up new definitions
ktx ingest --force
```

### Adding Wiki Knowledge

Create `wiki/global/pricing-policy.md`:

```markdown
# Pricing Policy

## Standard Pricing Tiers

- **Basic**: $29/month, up to 10 users
- **Professional**: $99/month, up to 50 users
- **Enterprise**: Custom pricing, unlimited users

## Discounts

- Annual commitment: 20% off
- Non-profit organizations: 30% off
- Educational institutions: 50% off

## Refund Policy

Full refund within 30 days of purchase, no questions asked.
After 30 days, prorated refund for annual plans only.

## Fiscal Year

Our fiscal year runs from February 1 to January 31.
Q1 = Feb-Apr, Q2 = May-Jul, Q3 = Aug-Oct, Q4 = Nov-Jan
```

```bash
# Rebuild to index wiki content
ktx ingest

# Search wiki
ktx wiki "refund policy"
ktx wiki "fiscal year Q1"
```

### Agent Integration Workflow

```bash
# 1. Setup ktx in your project
ktx setup

# 2. Build context
ktx ingest

# 3. Start MCP server
ktx mcp start --project-dir $(pwd)

# 4. In your agent (Claude Code, Codex, etc.), use ktx tools:
# - search_semantic_layer: Find metrics and tables
# - search_wiki: Find business knowledge
# - get_metric_definition: Get approved SQL for a metric
# - list_connections: See available data sources

# Example agent prompt:
# "Use ktx to search for revenue metrics and show me the approved SQL definition"
```

## Troubleshooting

### ktx setup fails with "LLM provider not configured"

**Solution**: Set required environment variables:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
# or
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

### ktx ingest fails with "Connection refused"

**Problem**: Database connection credentials incorrect or host unreachable.

**Solution**: Verify connection in `ktx.yaml` and test manually:

```bash
# For PostgreSQL
psql -h localhost -U analytics_user -d analytics

# For Snowflake
snowsql -a xy12345.us-east-1 -u username -d PROD
```

### ktx status shows "Agent integration ready: no"

**Problem**: MCP server not started or not configured for agent.

**Solution**: Start MCP server:

```bash
ktx mcp start

# If using Codex, ensure .codex/skills.yaml includes ktx
# If using Claude Code, ensure Claude can access MCP server
```

### Search returns no results

**Problem**: Context not built or out of date.

**Solution**: Rebuild context:

```bash
ktx ingest --force
```

### "Module not found" when importing @kaelio/ktx

**Problem**: Package not installed or not in node_modules.

**Solution**:

```bash
# Install globally for CLI
npm install -g @kaelio/ktx

# Install locally for programmatic use
npm install @kaelio/ktx
```

### Semantic layer definitions not appearing

**Problem**: YAML files in wrong location or invalid syntax.

**Solution**: Verify structure:

```bash
# Files must be in semantic-layer/<connection-id>/*.yaml
ls -la semantic-layer/warehouse/

# Validate YAML syntax
ktx validate
```

### MCP server crashes on startup

**Problem**: Port conflict or invalid project directory.

**Solution**:

```bash
# Check if another ktx instance is running
ktx mcp status

# Stop existing instance
ktx mcp stop

# Verify project directory
ktx status --project-dir $(pwd)
```

### "Read-only violation" error

**Problem**: ktx attempted to write to database (should never happen).

**Solution**: This is a bug. Report it:

```bash
# Check ktx version
ktx --version

# Report to GitHub Issues with connection type and error details
```

## Development and Testing

### Running ktx in Development Mode

```bash
# Clone repository
git clone https://github.com/kaelio/ktx.git
cd ktx

# Install dependencies
pnpm install
uv sync --all-groups

# Build project
pnpm run build

# Run type checks
pnpm run type-check

# Run tests
pnpm run test
uv run pytest -q

# Link development CLI
pnpm run setup:dev
pnpm run link:dev

# Use development version
ktx-dev --help
```

### Testing ktx Integration

```typescript
import { KtxClient } from '@kaelio/ktx';
import { describe, it, expect } from 'vitest';

describe('ktx integration', () => {
  const ktx = new KtxClient({
    projectDir: './test-project'
  });
  
  it('should search semantic layer', async () => {
    const results = await ktx.searchSemanticLayer('revenue', {
      limit: 5
    });
    
    expect(results).toBeDefined();
    expect(results.length).toBeGreaterThan(0);
    expect(results[0]).toHaveProperty('name');
    expect(results[0]).toHaveProperty('sql');
  });
  
  it('should search wiki', async () => {
    const results = await ktx.searchWiki('pricing', {
      limit: 3,
      namespace: 'global'
    });
    
    expect(results).toBeDefined();
    expect(results[0]).toHaveProperty('title');
    expect(results[0]).toHaveProperty('excerpt');
  });
  
  it('should validate project status', async () => {
    const status = await ktx.getStatus();
    
    expect(status.projectReady).toBe(true);
    expect(status.llmReady).toBe(true);
    expect(status.contextBuilt).toBe(true);
  });
});
```

## Additional Resources

- **Documentation**: https://docs.kaelio.com/ktx
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Setup Guide**: https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues**: https://github.com/Kaelio/ktx/issues
- **Contributing Guide**: https://docs.kaelio.com/ktx/docs/community/contributing
