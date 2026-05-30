---
name: ktx-ai-data-context-layer
description: Install and configure ktx, an executable context layer for data and analytics agents that enables accurate warehouse queries through MCP with semantic layer and skills
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer for warehouse
  - install ktx context layer with MCP
  - help me query my data warehouse with ktx
  - set up ktx for Claude Code data analysis
  - configure ktx ingestion from dbt and warehouse
  - troubleshoot ktx mcp server connection
  - build ktx context from database metadata
---

# ktx AI Data Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It ingests metadata from databases, dbt, Looker, Metabase, and wikis to build a semantic layer with approved metric definitions, joinable columns, and business knowledge. Agents access this through MCP (Model Context Protocol) tools and CLI commands.

## What ktx Does

- **Learns from company knowledge** — ingests wiki content, removes duplicates, flags contradictions
- **Maps the data stack** — samples tables, detects joinable columns, captures metadata and usage patterns
- **Builds a semantic layer** — combines raw tables and high-level metrics with automatic join resolution
- **Serves agents at execution** — exposes CLI and MCP tools with semantic search across wiki and semantic-layer entities

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, and SQLite. Integrates with dbt, MetricFlow, LookML, Looker, Metabase, and Notion.

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
cd /path/to/your/project
npm install @kaelio/ktx
```

### Verify Installation

```bash
ktx --version
ktx --help
```

## Quick Setup

```bash
# Create or resume a ktx project in current directory
ktx setup

# Check project readiness
ktx status
```

The `ktx setup` command will:
1. Create or resume a local ktx project
2. Configure LLM and embedding providers
3. Set up database connections
4. Configure context sources (dbt, Looker, etc.)
5. Build initial context
6. Install agent integration (MCP)

## Project Structure

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local.

## Configuration

### ktx.yaml Structure

```yaml
version: 1
name: my-analytics-project

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
    # Credentials stored in .ktx/secrets.yaml

context-sources:
  dbt_main:
    type: dbt
    path: ./dbt-project
    target: prod
  
  looker_main:
    type: looker
    base_url: https://company.looker.com
    # API credentials in .ktx/secrets.yaml

agent-integration:
  type: mcp
  clients:
    - codex:project
    - claude-code
```

### LLM Configuration

ktx supports multiple LLM providers:

```bash
# During setup, choose provider
ktx setup

# Supported providers:
# - anthropic (requires ANTHROPIC_API_KEY)
# - vertex-ai (requires Google Cloud credentials)
# - ai-gateway (custom endpoint)
# - claude-agent-sdk (local Claude Code session)
```

Set API keys via environment variables:

```bash
export ANTHROPIC_API_KEY=your-key-here
export OPENAI_API_KEY=your-key-here
```

### Database Connection

Example PostgreSQL connection in `ktx.yaml`:

```yaml
databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    ssl: true
```

Credentials in `.ktx/secrets.yaml` (auto-generated, git-ignored):

```yaml
databases:
  warehouse:
    user: readonly_user
    password: ${DB_PASSWORD}
```

### Context Sources

#### dbt Integration

```yaml
context-sources:
  dbt_main:
    type: dbt
    path: ./dbt-project
    target: prod
    manifest_path: ./dbt-project/target/manifest.json
```

#### Looker Integration

```yaml
context-sources:
  looker_main:
    type: looker
    base_url: https://company.looker.com
    # client_id and client_secret in .ktx/secrets.yaml
```

#### Notion Integration

```yaml
context-sources:
  company_wiki:
    type: notion
    # token in .ktx/secrets.yaml
    database_ids:
      - abc123def456
```

## Key Commands

### Setup and Status

```bash
# Create/resume project
ktx setup

# Check project status
ktx status

# Example output:
# ktx project: /home/user/analytics
# Project ready: yes
# LLM ready: yes (claude-sonnet-4-6)
# Embeddings ready: yes (text-embedding-3-small)
# Databases configured: yes (warehouse)
# Context sources configured: yes (dbt_main)
# ktx context built: yes
# Agent integration ready: yes (codex:project)
```

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
ktx sl "monthly active users"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "data retention"

# Combined search (semantic layer + wiki)
ktx search "customer churn"
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

### Semantic Layer Management

```bash
# List semantic sources
ktx sl list

# Validate semantic layer definitions
ktx sl validate

# Show semantic source details
ktx sl show users
ktx sl show revenue_metrics
```

### Project Management

```bash
# Initialize new project (alternative to setup)
ktx init

# Update project configuration
ktx config set llm.provider anthropic
ktx config set llm.model claude-sonnet-4-6

# Show current configuration
ktx config show

# Clean cached context
ktx clean --cache

# Clean everything (reset project)
ktx clean --all
```

## Real Usage Examples

### Example 1: Set Up ktx for PostgreSQL Warehouse

```bash
# Install globally
npm install -g @kaelio/ktx

# Navigate to project
cd ~/my-analytics-project

# Set LLM API key
export ANTHROPIC_API_KEY=sk-ant-...

# Run setup (interactive)
ktx setup
# Choose: anthropic, claude-sonnet-4-6
# Choose: openai, text-embedding-3-small
# Add database: postgres
# Host: localhost
# Port: 5432
# Database: analytics
# User: readonly_user
# Password: [enter securely]

# Verify setup
ktx status
```

### Example 2: Ingest dbt and Build Context

```bash
# Configure dbt source in ktx.yaml
cat >> ktx.yaml <<EOF
context-sources:
  dbt_main:
    type: dbt
    path: ./dbt-project
    target: prod
EOF

# Run ingestion
ktx ingest

# Search for dbt models
ktx sl "dim_customers"
ktx sl "fct_orders"
```

### Example 3: Query Semantic Layer from TypeScript

```typescript
import { execSync } from 'child_process';

// Search semantic layer
function searchSemanticLayer(query: string): string {
  const result = execSync(`ktx sl "${query}"`, { encoding: 'utf-8' });
  return result;
}

// Example usage
const revenueMetrics = searchSemanticLayer('revenue');
console.log(revenueMetrics);

// Search wiki
function searchWiki(query: string): string {
  const result = execSync(`ktx wiki "${query}"`, { encoding: 'utf-8' });
  return result;
}

const refundPolicy = searchWiki('refund policy');
console.log(refundPolicy);
```

### Example 4: Set Up MCP for Claude Code

```bash
# Navigate to project
cd ~/my-analytics-project

# Run setup with MCP integration
ktx setup
# During setup, select: "Install MCP integration? yes"
# Select agent: "Claude Code (codex)"

# Start MCP server
ktx mcp start --project-dir ~/my-analytics-project

# Keep terminal open, open Claude Code in another window
# Claude Code will auto-discover the MCP server
```

### Example 5: Create Semantic Layer Definition

Create `semantic-layer/warehouse/customers.yaml`:

```yaml
kind: SemanticSource
name: customers
type: entity

description: Customer dimension with lifetime metrics

columns:
  - name: customer_id
    type: dimension
    data_type: integer
    primary_key: true
    
  - name: email
    type: dimension
    data_type: string
    
  - name: created_at
    type: dimension
    data_type: timestamp
    
  - name: lifetime_value
    type: metric
    data_type: numeric
    sql: SUM(order_total)
    aggregation: sum
    
  - name: total_orders
    type: metric
    data_type: integer
    sql: COUNT(DISTINCT order_id)
    aggregation: count

sql: |
  SELECT 
    c.id as customer_id,
    c.email,
    c.created_at,
    COALESCE(SUM(o.total), 0) as lifetime_value,
    COUNT(o.id) as total_orders
  FROM customers c
  LEFT JOIN orders o ON c.id = o.customer_id
  GROUP BY c.id, c.email, c.created_at

joins:
  - to: orders
    type: one_to_many
    on: customers.customer_id = orders.customer_id
```

Validate and ingest:

```bash
ktx sl validate
ktx ingest --force
ktx sl show customers
```

### Example 6: Add Wiki Context

Create `wiki/global/data-definitions.md`:

```markdown
# Data Definitions

## Revenue Recognition

Revenue is recognized at the point of sale, not when payment is received.

**Key Metrics:**
- `gross_revenue`: Total sales before discounts
- `net_revenue`: Sales minus refunds and discounts
- `arr`: Annual Recurring Revenue (subscription only)

## Customer Classification

- **Active**: Made a purchase in last 90 days
- **Churned**: No purchase in 90+ days
- **New**: First purchase within last 30 days

## Refund Policy

Refunds are processed within 7 days and deducted from revenue in the same period.
```

Ingest wiki:

```bash
ktx ingest
ktx wiki "revenue recognition"
```

## Common Patterns

### Pattern 1: Daily Context Refresh

```bash
#!/bin/bash
# refresh-context.sh

cd /path/to/ktx/project
ktx ingest --force
ktx sl validate
```

Run via cron:

```bash
# Run daily at 2am
0 2 * * * /path/to/refresh-context.sh
```

### Pattern 2: Multi-Environment Setup

```yaml
# ktx.yaml
databases:
  warehouse_prod:
    type: postgres
    host: prod.example.com
    database: analytics
    
  warehouse_staging:
    type: postgres
    host: staging.example.com
    database: analytics
```

```bash
# Ingest only production
ktx ingest --connection warehouse_prod

# Ingest only staging
ktx ingest --connection warehouse_staging
```

### Pattern 3: CI/CD Validation

```yaml
# .github/workflows/validate-ktx.yml
name: Validate ktx Semantic Layer

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install ktx
        run: npm install -g @kaelio/ktx
        
      - name: Validate semantic layer
        run: ktx sl validate
```

### Pattern 4: Environment-Specific Secrets

```bash
# .env.production
ANTHROPIC_API_KEY=sk-ant-prod-...
DB_PASSWORD=prod_password

# .env.staging
ANTHROPIC_API_KEY=sk-ant-staging-...
DB_PASSWORD=staging_password
```

Load before running:

```bash
source .env.production
ktx ingest
```

## Troubleshooting

### Issue: `ktx command not found`

**Solution**: Ensure global installation or use npx:

```bash
npm install -g @kaelio/ktx
# or
npx @kaelio/ktx setup
```

### Issue: MCP server not connecting

**Solution**: Check server is running and project directory is correct:

```bash
ktx mcp status

# If not running:
ktx mcp start --project-dir /absolute/path/to/project

# Verify in Claude Code settings:
# MCP servers should show ktx with status "Connected"
```

### Issue: Database connection fails

**Solution**: Verify credentials and network access:

```bash
# Test connection manually
psql -h localhost -p 5432 -U readonly_user -d analytics

# Check ktx configuration
ktx config show

# Re-run setup to update credentials
ktx setup
```

### Issue: Ingestion fails with "LLM error"

**Solution**: Verify API key and quota:

```bash
# Check API key is set
echo $ANTHROPIC_API_KEY

# Verify in ktx config
ktx config show

# Try with different model
ktx config set llm.model claude-sonnet-3-5-20240620
ktx ingest
```

### Issue: Semantic layer validation errors

**Solution**: Check YAML syntax and required fields:

```bash
# Run validation with verbose output
ktx sl validate --verbose

# Common issues:
# - Missing 'kind' field
# - Invalid column types (must be: dimension, metric, or attribute)
# - Invalid join syntax
# - Missing SQL or columns definition
```

### Issue: Wiki search returns no results

**Solution**: Verify wiki ingestion completed:

```bash
# Re-run ingestion
ktx ingest --force

# Check wiki files exist
ls -la wiki/global/

# Verify embeddings configured
ktx config show | grep embeddings
```

### Issue: Slow ingestion performance

**Solution**: Use incremental ingestion and tune sampling:

```bash
# Skip force refresh (use cache)
ktx ingest

# Reduce table sampling (in ktx.yaml)
databases:
  warehouse:
    sampling:
      max_rows: 1000  # default is 10000
```

### Issue: Agent can't find ktx MCP tools

**Solution**: Ensure MCP server is running and configured:

```bash
# Start MCP server in background
ktx mcp start --project-dir $(pwd) &

# In Claude Code, verify MCP connection:
# Settings > MCP Servers > ktx should show "Connected"

# If using Cursor/other, check their MCP configuration
```

## Advanced Usage

### Custom LLM Provider (AI Gateway)

```yaml
llm:
  provider: ai-gateway
  endpoint: https://gateway.example.com/v1/chat/completions
  model: custom-model
  api_key_env: CUSTOM_API_KEY
```

### Multiple Semantic Layer Sources

```yaml
context-sources:
  dbt_core:
    type: dbt
    path: ./dbt-core
    
  dbt_marketing:
    type: dbt
    path: ./dbt-marketing
    
  metabase:
    type: metabase
    url: https://metabase.example.com
```

### Programmatic Access (Node.js)

```typescript
import { spawn } from 'child_process';

async function queryKtx(query: string): Promise<string> {
  return new Promise((resolve, reject) => {
    const proc = spawn('ktx', ['search', query]);
    let output = '';
    
    proc.stdout.on('data', (data) => {
      output += data.toString();
    });
    
    proc.on('close', (code) => {
      if (code === 0) {
        resolve(output);
      } else {
        reject(new Error(`ktx exited with code ${code}`));
      }
    });
  });
}

// Usage
const results = await queryKtx('customer metrics');
console.log(results);
```

## Project Resolution

ktx resolves the project directory in this order:

1. `--project-dir` flag
2. `KTX_PROJECT_DIR` environment variable
3. Nearest `ktx.yaml` (walk up from current directory)
4. Current directory

```bash
# Explicit project directory
ktx status --project-dir /path/to/project

# Using environment variable
export KTX_PROJECT_DIR=/path/to/project
ktx status

# Auto-discover from current directory
cd /path/to/project
ktx status
```

## Resources

- [Documentation](https://docs.kaelio.com/ktx/docs/)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [GitHub Repository](https://github.com/Kaelio/ktx)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
