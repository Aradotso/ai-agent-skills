---
name: ktx-ai-data-agents-context-layer
description: Build and query an executable context layer for AI data agents using ktx semantic layer, wiki knowledge, and MCP integration
triggers:
  - set up ktx for my data warehouse
  - configure ktx semantic layer for my analytics
  - integrate ktx with Claude Code for data queries
  - build ktx context from my dbt project
  - use ktx to teach agents about my database schema
  - query my warehouse using ktx MCP server
  - add ktx wiki knowledge for my data stack
  - configure ktx for PostgreSQL and Snowflake
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It automatically learns from your company knowledge (wiki, dbt, Looker, Metabase), maps your data stack, builds a semantic layer with approved metrics, and serves agents through CLI and MCP tools.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, dbt docs, LookML, Looker, Metabase, and Notion, organizing and deduplicating automatically
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, and annotates sources for accurate queries
- **Builds semantic layer**: Combines raw tables and metrics through a join graph that resolves chasm and fan traps
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search across wiki and semantic-layer entities
- **Read-only by design**: Never writes to your database, only reads metadata and samples

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Scoped Installation

```bash
npm install --save-dev @kaelio/ktx
```

### Verify Installation

```bash
ktx --version
ktx --help
```

## Quick Setup

### Interactive Setup

```bash
ktx setup
```

This command will:
1. Create or resume a ktx project
2. Configure LLM and embedding providers
3. Add database connections
4. Configure context sources (dbt, Looker, etc.)
5. Build initial context
6. Install agent integration

### Check Project Status

```bash
ktx status
```

Expected output after successful setup:

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
├── ktx.yaml                         # Main configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts
└── .ktx/                            # Local state (git-ignored)
```

### Basic ktx.yaml Configuration

```yaml
version: 1
project:
  name: my-analytics-project
  description: Analytics warehouse for product and revenue metrics

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  apiKeyEnv: ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  apiKeyEnv: OPENAI_API_KEY

connections:
  - id: warehouse
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    user: readonly_user
    passwordEnv: WAREHOUSE_PASSWORD
    schemas:
      - public
      - analytics

contextSources:
  - id: dbt_main
    type: dbt
    manifestPath: ./target/manifest.json
    catalogPath: ./target/catalog.json
```

### Database Connection Examples

#### PostgreSQL

```yaml
connections:
  - id: warehouse
    type: postgres
    host: db.example.com
    port: 5432
    database: analytics
    user: ktx_readonly
    passwordEnv: POSTGRES_PASSWORD
    schemas:
      - public
      - analytics
      - staging
```

#### Snowflake

```yaml
connections:
  - id: snowflake_prod
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: KTX_USER
    passwordEnv: SNOWFLAKE_PASSWORD
    role: ANALYST
```

#### BigQuery

```yaml
connections:
  - id: bigquery_prod
    type: bigquery
    project: my-gcp-project
    dataset: analytics
    credentialsEnv: GOOGLE_APPLICATION_CREDENTIALS
```

### LLM Provider Configuration

#### Anthropic API

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  apiKeyEnv: ANTHROPIC_API_KEY
```

#### Google Vertex AI

```yaml
llm:
  provider: vertex
  model: claude-sonnet-4-6
  project: my-gcp-project
  region: us-central1
  credentialsEnv: GOOGLE_APPLICATION_CREDENTIALS
```

#### Claude Code Session

```yaml
llm:
  provider: claude-code
  # Uses local Claude Code session, no API key needed
```

### Context Source Configuration

#### dbt Project

```yaml
contextSources:
  - id: dbt_main
    type: dbt
    manifestPath: ./target/manifest.json
    catalogPath: ./target/catalog.json
    description: Core dbt models and metrics
```

#### LookML

```yaml
contextSources:
  - id: lookml_main
    type: lookml
    projectPath: ./looker
    description: Looker explores and views
```

#### Notion

```yaml
contextSources:
  - id: notion_docs
    type: notion
    apiKeyEnv: NOTION_API_KEY
    databaseId: abc123def456
    description: Analytics documentation
```

## Key Commands

### Building Context

#### Build All Context

```bash
ktx ingest
```

Runs ingestion for all configured connections and context sources.

#### Build Specific Connection

```bash
ktx ingest --connection warehouse
```

#### Build Specific Context Source

```bash
ktx ingest --source dbt_main
```

#### Force Rebuild

```bash
ktx ingest --force
```

### Searching Context

#### Search Semantic Layer

```bash
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn rate"
```

Returns metrics, dimensions, and entities matching the query.

#### Search Wiki

```bash
ktx wiki "refund policy"
ktx wiki "data retention"
ktx wiki "how to calculate LTV"
```

Returns wiki pages with relevant business knowledge.

#### Combined Search

```bash
ktx search "customer revenue" --limit 10
```

Searches both semantic layer and wiki content.

### Managing Semantic Layer

#### List Metrics

```bash
ktx sl list --type metric
```

#### List Entities

```bash
ktx sl list --type entity
```

#### Validate Semantic Layer

```bash
ktx sl validate
```

Checks for join path issues, orphaned metrics, and contradictions.

### MCP Server for Agents

#### Start MCP Server

```bash
ktx mcp start --project-dir /path/to/project
```

Starts the MCP server that agents connect to.

#### Test MCP Connection

```bash
ktx mcp test
```

### Project Management

#### Initialize New Project

```bash
mkdir my-ktx-project
cd my-ktx-project
ktx init
```

#### Update Project Configuration

```bash
ktx setup
```

Re-runs setup wizard to modify configuration.

#### Validate Project

```bash
ktx validate
```

Checks configuration, connection health, and context integrity.

## Code Examples

### TypeScript: Programmatic Usage

```typescript
import { KtxProject } from '@kaelio/ktx';

async function querySemanticLayer() {
  const project = await KtxProject.load('/path/to/project');
  
  // Search for metrics
  const results = await project.search({
    query: 'revenue',
    type: 'metric',
    limit: 5
  });
  
  console.log('Found metrics:', results);
  
  // Get metric definition
  const metric = await project.getMetric('total_revenue');
  console.log('SQL:', metric.sql);
  console.log('Dimensions:', metric.dimensions);
}

async function addWikiPage() {
  const project = await KtxProject.load('/path/to/project');
  
  await project.wiki.addPage({
    title: 'Revenue Calculation Guide',
    content: `
# Revenue Calculation Guide

## Monthly Recurring Revenue (MRR)
Sum of all active subscriptions normalized to monthly amounts.

## Annual Recurring Revenue (ARR)
MRR × 12

## Important Notes
- Exclude one-time charges
- Include only active subscriptions
- Use subscription.status = 'active'
    `,
    tags: ['revenue', 'metrics', 'definitions']
  });
}

async function validateConnections() {
  const project = await KtxProject.load('/path/to/project');
  
  for (const conn of project.connections) {
    const isHealthy = await conn.test();
    console.log(`${conn.id}: ${isHealthy ? 'OK' : 'FAIL'}`);
  }
}
```

### Python: Using ktx-sl Query Planner

```python
from ktx_sl import SemanticLayer, QueryRequest

# Load semantic layer
sl = SemanticLayer.from_directory('./semantic-layer/warehouse')

# Build query plan
request = QueryRequest(
    metrics=['total_revenue', 'customer_count'],
    dimensions=['date_month', 'product_category'],
    where=['date_month >= 2024-01-01']
)

plan = sl.plan_query(request)

print("Generated SQL:")
print(plan.sql)

print("\nJoin path:")
for join in plan.join_path:
    print(f"  {join.left_entity} -> {join.right_entity} on {join.condition}")

# Detect issues
issues = sl.validate()
for issue in issues:
    print(f"[{issue.severity}] {issue.message}")
```

### Shell: Common Workflows

```bash
#!/bin/bash

# Complete setup workflow
ktx setup

# Build all context
ktx ingest

# Validate everything
ktx validate

# Search for revenue metrics
ktx sl "revenue" --format json > revenue_metrics.json

# Start MCP server for agents
ktx mcp start --project-dir $(pwd)
```

### Agent Integration: Using ktx with Claude Code

Add to your Claude Code configuration:

```json
{
  "mcpServers": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "/path/to/project"]
    }
  }
}
```

Then in Claude Code:

```text
Use ktx to find all revenue-related metrics

Query the total_revenue metric grouped by month for 2024

Show me the join path from customers to orders
```

## Common Patterns

### Pattern 1: Initial Project Setup

```bash
# Create project directory
mkdir analytics-ktx && cd analytics-ktx

# Initialize ktx
ktx init

# Run interactive setup
ktx setup
# - Select Anthropic as LLM provider
# - Add PostgreSQL connection with readonly credentials
# - Configure dbt context source pointing to ./target/manifest.json
# - Build initial context

# Verify setup
ktx status

# Start MCP server for agents
ktx mcp start
```

### Pattern 2: Adding New Database Connection

```typescript
import { execSync } from 'child_process';
import fs from 'fs';
import yaml from 'yaml';

function addConnection() {
  // Read current config
  const config = yaml.parse(fs.readFileSync('ktx.yaml', 'utf8'));
  
  // Add new connection
  config.connections.push({
    id: 'analytics_replica',
    type: 'postgres',
    host: 'replica.example.com',
    port: 5432,
    database: 'analytics',
    user: 'readonly',
    passwordEnv: 'REPLICA_PASSWORD',
    schemas: ['public', 'staging']
  });
  
  // Write back
  fs.writeFileSync('ktx.yaml', yaml.stringify(config));
  
  // Rebuild context
  execSync('ktx ingest --connection analytics_replica');
}
```

### Pattern 3: Creating Custom Metrics

Create `semantic-layer/warehouse/metrics/revenue.yaml`:

```yaml
metrics:
  - name: total_revenue
    description: Sum of all completed order amounts
    type: simple
    sql: SUM(orders.amount)
    entity: order
    dimensions:
      - date_day
      - customer_id
      - product_category
    filters:
      - orders.status = 'completed'
    
  - name: arr
    description: Annual Recurring Revenue
    type: derived
    sql: mrr * 12
    dependsOn:
      - mrr
    
  - name: revenue_per_customer
    description: Average revenue per customer
    type: ratio
    numerator: total_revenue
    denominator: customer_count
```

Then rebuild:

```bash
ktx ingest --source warehouse
ktx sl validate
```

### Pattern 4: Adding Wiki Knowledge

Create `wiki/global/revenue-definitions.md`:

```markdown
# Revenue Definitions

## Total Revenue
Sum of all completed orders. Excludes refunded, cancelled, or pending orders.

**Source**: `orders` table
**Filter**: `status = 'completed'`

## Monthly Recurring Revenue (MRR)
Normalized monthly subscription revenue.

**Calculation**:
- Annual plans: `amount / 12`
- Monthly plans: `amount`
- Quarterly plans: `amount / 3`

**Important**: Only include active subscriptions (`subscription.status = 'active'`)

## Customer Lifetime Value (LTV)
Average revenue per customer over their lifetime.

**Formula**: `(Average Order Value) × (Purchase Frequency) × (Customer Lifespan)`

**Typical Lifespan**: 24 months for SaaS products

## Related Metrics
- [[ARR]]: Annual Recurring Revenue (MRR × 12)
- [[Churn Rate]]: Percentage of customers who cancel
- [[CAC]]: Customer Acquisition Cost
```

Ingest wiki content:

```bash
ktx ingest
```

### Pattern 5: Querying with MCP from Agent

In your agent (Claude Code, Cursor, etc.):

```text
User: What's our total revenue by month for Q1 2024?

Agent uses ktx MCP:
1. ktx.search(query: "total revenue metric")
2. ktx.getMetric(name: "total_revenue")
3. Generate SQL:
   SELECT
     DATE_TRUNC('month', orders.created_at) as month,
     SUM(orders.amount) as total_revenue
   FROM orders
   WHERE orders.status = 'completed'
     AND orders.created_at >= '2024-01-01'
     AND orders.created_at < '2024-04-01'
   GROUP BY 1
   ORDER BY 1
```

## Troubleshooting

### ktx setup fails with "LLM connection error"

**Cause**: Missing or invalid API key

**Solution**:
```bash
# Check environment variable
echo $ANTHROPIC_API_KEY

# Set it if missing
export ANTHROPIC_API_KEY=sk-ant-...

# Or update ktx.yaml to reference correct env var
```

### Database connection fails during ingest

**Cause**: Network/credentials issue

**Solution**:
```bash
# Test connection directly
ktx validate --connection warehouse

# Check credentials
echo $WAREHOUSE_PASSWORD

# Verify network access
psql -h localhost -U readonly_user -d analytics
```

### "No semantic layer found" when searching

**Cause**: Context not built yet

**Solution**:
```bash
# Build all context
ktx ingest

# Or build specific source
ktx ingest --source dbt_main

# Verify context
ktx status
ktx sl list
```

### MCP server fails to start

**Cause**: Port conflict or missing project directory

**Solution**:
```bash
# Specify project directory explicitly
ktx mcp start --project-dir /path/to/project

# Check if another instance is running
ps aux | grep "ktx mcp"

# Kill existing instance
pkill -f "ktx mcp"
```

### Metrics show contradictory definitions

**Cause**: Multiple sources defining same metric differently

**Solution**:
```bash
# Validate semantic layer
ktx sl validate

# Review contradictions report
cat ./raw-sources/warehouse/contradictions.json

# Resolve in wiki/global/metric-decisions.md
```

### Out of memory during large database ingestion

**Cause**: Sampling too many tables at once

**Solution**:
```yaml
# In ktx.yaml, limit sampling
connections:
  - id: warehouse
    type: postgres
    # ... other config
    samplingConfig:
      maxRowsPerTable: 1000
      maxTablesPerSchema: 50
      excludePatterns:
        - "_temp_*"
        - "staging_*"
```

### Search returns no results

**Cause**: Embeddings not generated or outdated

**Solution**:
```bash
# Rebuild embeddings
ktx ingest --force

# Verify embeddings config
ktx status

# Test search directly
ktx search "revenue" --debug
```

### Agent can't access ktx MCP tools

**Cause**: MCP server not started or wrong configuration

**Solution**:
```bash
# Start MCP server
ktx mcp start --project-dir $(pwd)

# Test MCP connection
ktx mcp test

# Check agent MCP config points to correct command
# For Claude Code: .claude/config.json
# For Cursor: cursor.config.json
```

## Advanced Usage

### Custom Embedding Models

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-large
  dimensions: 3072
  apiKeyEnv: OPENAI_API_KEY
```

### Multi-Warehouse Setup

```yaml
connections:
  - id: prod_postgres
    type: postgres
    # ... config
    
  - id: prod_snowflake
    type: snowflake
    # ... config
    
  - id: analytics_bigquery
    type: bigquery
    # ... config
```

### CI/CD Integration

```yaml
# .github/workflows/ktx-validate.yml
name: Validate ktx Context

on:
  pull_request:
    paths:
      - 'semantic-layer/**'
      - 'wiki/**'
      - 'ktx.yaml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm install -g @kaelio/ktx
      - run: ktx validate
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          WAREHOUSE_PASSWORD: ${{ secrets.WAREHOUSE_PASSWORD }}
```

## Resources

- [Documentation](https://docs.kaelio.com/ktx/docs/)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [GitHub Repository](https://github.com/Kaelio/ktx)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
