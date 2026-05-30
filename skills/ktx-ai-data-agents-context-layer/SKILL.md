---
name: ktx-ai-data-agents-context-layer
description: Context layer for data agents - teach AI agents to query warehouses accurately with semantic layers, metrics, and business knowledge
triggers:
  - set up ktx for my data warehouse
  - configure ktx semantic layer for my database
  - help me integrate ktx with Claude Code
  - build context for my analytics warehouse with ktx
  - create ktx metrics and wiki for agent queries
  - troubleshoot ktx MCP server connection
  - ingest dbt models into ktx semantic layer
  - search ktx wiki for business definitions
---

# ktx AI Data Agents Context Layer Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is an executable context layer that teaches AI agents to query data warehouses accurately. It automatically builds a semantic layer from raw tables, dbt models, LookML, Looker, Metabase, and Notion content, resolves joins, defines canonical metrics, and serves everything through CLI and MCP interfaces.

**Key capabilities:**
- Auto-detects joinable columns and resolves fan/chasm traps
- Ingests wiki/Notion content and flags contradictions
- Builds reusable metric definitions from dbt, MetricFlow, LookML
- Exposes read-only MCP tools for agent execution
- Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite

## Installation

### Global CLI installation

```bash
npm install -g @kaelio/ktx
```

### Project-local installation

```bash
npm install --save-dev @kaelio/ktx
npx ktx --version
```

### Initialize a ktx project

```bash
# Interactive setup - creates ktx.yaml, configures providers, builds context
ktx setup

# Non-interactive with specific project directory
ktx setup --project-dir ./analytics --non-interactive
```

## Project Structure

After `ktx setup`, your project will have:

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML metric definitions
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Git best practices:**
- Commit: `ktx.yaml`, `semantic-layer/`, `wiki/`
- Ignore: `.ktx/` (contains secrets)

## Configuration

### Basic ktx.yaml structure

```yaml
version: 1
project:
  name: analytics
  description: Company analytics warehouse

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key stored in .ktx/config.json, not here

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key stored in .ktx/config.json

databases:
  warehouse:
    type: postgres
    connection:
      host: localhost
      port: 5432
      database: analytics
      # Credentials stored in .ktx/connections.json

context_sources:
  dbt_main:
    type: dbt
    path: ./dbt_project
    database_id: warehouse
```

### Supported database types

- `postgres`, `snowflake`, `bigquery`, `clickhouse`, `mysql`, `sqlserver`, `sqlite`

### Supported context source types

- `dbt` - dbt project with models and metrics
- `metricflow` - MetricFlow semantic layer
- `lookml` - LookML projects
- `looker` - Looker instance metadata
- `metabase` - Metabase collections
- `notion` - Notion pages and databases

## Key Commands

### Project status and health

```bash
# Check if project is ready for agent use
ktx status

# Expected output:
# ktx project: /home/user/analytics
# Project ready: yes
# LLM ready: yes (claude-sonnet-4-6)
# Embeddings ready: yes (text-embedding-3-small)
# Databases configured: yes (warehouse)
# Context sources configured: yes (dbt_main)
# ktx context built: yes
# Agent integration ready: yes (codex:project)
```

### Building context

```bash
# Ingest all configured connections and sources
ktx ingest

# Ingest specific database only
ktx ingest --database warehouse

# Ingest specific context source only
ktx ingest --source dbt_main

# Force rebuild even if unchanged
ktx ingest --force
```

### Searching context

```bash
# Search semantic layer (metrics, dimensions, entities)
ktx sl "revenue"
ktx sl "monthly active users"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "customer segmentation"

# Limit search results
ktx sl "revenue" --limit 5
```

### MCP server for agents

```bash
# Start MCP server (required before opening agent client)
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status

# Stop MCP server
ktx mcp stop
```

### Managing semantic layer

```bash
# List all semantic entities
ktx sl list

# Show specific metric details
ktx sl show "monthly_recurring_revenue"

# Validate semantic layer YAML
ktx sl validate
```

## Real-World Usage Patterns

### Pattern 1: Setting up ktx for an existing dbt project

```bash
# Navigate to your dbt project root
cd /path/to/my-dbt-project

# Initialize ktx
ktx setup

# During setup, select:
# - LLM provider: Anthropic
# - Database type: postgres (or your warehouse)
# - Context source: dbt (point to current directory)

# Build initial context
ktx ingest

# Verify setup
ktx status

# Start MCP for agent access
ktx mcp start
```

### Pattern 2: Adding a custom metric definition

Create `semantic-layer/warehouse/metrics/mrr.yaml`:

```yaml
version: 1
type: metric
name: monthly_recurring_revenue
description: Monthly recurring revenue from active subscriptions
label: Monthly Recurring Revenue
sql: SUM(subscription_amount)
timestamp: subscription_start_date
time_granularity: month
dimensions:
  - plan_type
  - customer_segment
filters:
  - field: subscription_status
    operator: equals
    value: active
sources:
  - subscriptions
```

Then reload:

```bash
ktx ingest --source dbt_main
ktx sl show "monthly_recurring_revenue"
```

### Pattern 3: Adding business context to wiki

Create `wiki/global/customer-segmentation.md`:

```markdown
# Customer Segmentation

Our customer base is divided into three segments:

## Enterprise
- Annual contract value > $100,000
- Dedicated account manager
- Custom SLA

## Mid-Market
- Annual contract value $10,000 - $100,000
- Standard support
- Quarterly business reviews

## SMB
- Annual contract value < $10,000
- Self-service support
- Monthly usage tier

Note: Segmentation is based on the `customers.segment` field in the warehouse.
```

Index it:

```bash
ktx ingest
ktx wiki "customer segment"
```

### Pattern 4: Connecting to Snowflake

Update `ktx.yaml`:

```yaml
databases:
  snowflake_prod:
    type: snowflake
    connection:
      account: xy12345.us-east-1
      warehouse: COMPUTE_WH
      database: ANALYTICS
      schema: PUBLIC
      role: ANALYST
      # user and password stored in .ktx/connections.json
```

Set credentials:

```bash
# ktx setup will prompt for credentials, or manually edit .ktx/connections.json
ktx setup --update

# Test connection
ktx ingest --database snowflake_prod --dry-run
```

### Pattern 5: Integrating with Claude Code

After running `ktx setup`, if prompted:

```bash
# Install ktx skill for Claude Code
npx skills add Kaelio/ktx --skill ktx

# Start MCP server (required for Claude Code integration)
ktx mcp start
```

In Claude Code, ask:

```
Search the ktx semantic layer for revenue metrics
```

Claude Code will use the `ktx_search_semantic_layer` MCP tool to query your context.

## Agent Integration Examples

### TypeScript: Using ktx programmatically

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

interface SemanticSearchResult {
  name: string;
  description: string;
  type: string;
  score: number;
}

async function searchSemanticLayer(query: string): Promise<SemanticSearchResult[]> {
  const { stdout } = await execAsync(`ktx sl "${query}" --json`);
  return JSON.parse(stdout);
}

async function searchWiki(query: string): Promise<any[]> {
  const { stdout } = await execAsync(`ktx wiki "${query}" --json`);
  return JSON.parse(stdout);
}

// Example usage
async function findRevenueMetrics() {
  const metrics = await searchSemanticLayer('revenue');
  console.log('Revenue metrics:', metrics);
  
  const context = await searchWiki('revenue recognition policy');
  console.log('Business context:', context);
}

findRevenueMetrics();
```

### Python: Calling ktx from a Jupyter notebook

```python
import subprocess
import json

def search_semantic_layer(query: str) -> list:
    """Search ktx semantic layer from Python"""
    result = subprocess.run(
        ['ktx', 'sl', query, '--json'],
        capture_output=True,
        text=True
    )
    return json.loads(result.stdout)

def search_wiki(query: str) -> list:
    """Search ktx wiki from Python"""
    result = subprocess.run(
        ['ktx', 'wiki', query, '--json'],
        capture_output=True,
        text=True
    )
    return json.loads(result.stdout)

# Example: Find customer-related metrics
metrics = search_semantic_layer("customer churn")
for metric in metrics:
    print(f"{metric['name']}: {metric['description']}")

# Example: Find business definitions
wiki_pages = search_wiki("churn definition")
for page in wiki_pages:
    print(f"{page['title']}: {page['excerpt']}")
```

## Common Configuration Scenarios

### Multiple databases

```yaml
databases:
  prod_warehouse:
    type: snowflake
    connection:
      account: prod.us-east-1
      database: PROD_ANALYTICS
  
  staging_warehouse:
    type: postgres
    connection:
      host: staging-db.internal
      database: staging_analytics

context_sources:
  dbt_prod:
    type: dbt
    path: ./dbt
    database_id: prod_warehouse
  
  dbt_staging:
    type: dbt
    path: ./dbt
    database_id: staging_warehouse
```

### Using AI Gateway instead of direct Anthropic API

```yaml
llm:
  provider: ai-gateway
  model: claude-sonnet-4-6
  endpoint: https://gateway.ai.example.com/v1
  # API key stored in .ktx/config.json as AI_GATEWAY_API_KEY
```

### Using local Claude Code session (no API key needed)

```yaml
llm:
  provider: claude-agent-sdk
  # No API key required - uses local Claude Code session
```

### Configuring Notion integration

```yaml
context_sources:
  company_wiki:
    type: notion
    notion:
      database_ids:
        - a1b2c3d4e5f6
        - f6e5d4c3b2a1
      # NOTION_API_KEY stored in .ktx/config.json
```

## Troubleshooting

### MCP server not starting

**Error:** `ktx mcp start` hangs or fails

**Solution:**

```bash
# Check if port is already in use
lsof -i :3000

# Stop existing MCP server
ktx mcp stop

# Start with verbose logging
ktx mcp start --log-level debug

# Check MCP status
ktx mcp status
```

### Agent can't find ktx tools

**Error:** Claude Code says "ktx tools not available"

**Solution:**

```bash
# Ensure MCP server is running
ktx mcp status

# If not running, check ktx status for instructions
ktx status

# Start MCP with explicit project directory
ktx mcp start --project-dir $(pwd)

# Restart Claude Code after MCP server starts
```

### Context not updating after changes

**Error:** Changes to dbt models or wiki not reflected in searches

**Solution:**

```bash
# Force rebuild context
ktx ingest --force

# Verify specific source
ktx ingest --source dbt_main --verbose

# Check for errors
ktx status
```

### Database connection failures

**Error:** `Could not connect to database 'warehouse'`

**Solution:**

```bash
# Verify connection settings
cat ktx.yaml

# Check credentials in .ktx/connections.json
cat .ktx/connections.json | jq '.databases.warehouse'

# Test connection with dry-run
ktx ingest --database warehouse --dry-run

# Update credentials
ktx setup --update
```

### LLM provider errors

**Error:** `LLM provider 'anthropic' not configured`

**Solution:**

```bash
# Verify LLM configuration
cat ktx.yaml | grep -A 3 "llm:"

# Check API key is set in .ktx/config.json
cat .ktx/config.json | jq '.llm'

# Reconfigure LLM provider
ktx setup --update

# Or set environment variable
export ANTHROPIC_API_KEY=your-api-key-here
```

### Semantic layer validation errors

**Error:** `Invalid semantic layer YAML in metrics/revenue.yaml`

**Solution:**

```bash
# Validate specific file
ktx sl validate semantic-layer/warehouse/metrics/revenue.yaml

# Check YAML syntax
cat semantic-layer/warehouse/metrics/revenue.yaml

# Common issues:
# - Missing required fields: name, description, sql
# - Invalid type values (must be: metric, dimension, entity, source)
# - Malformed YAML indentation
```

### Search returning no results

**Error:** `ktx sl "revenue"` returns empty results

**Solution:**

```bash
# Rebuild search indices
ktx ingest --force

# Verify context was built
ls -la semantic-layer/

# Check if metrics exist
ktx sl list

# Try broader search term
ktx sl "rev"

# Search with verbose output
ktx sl "revenue" --verbose
```

## Environment Variables

ktx respects these environment variables:

- `KTX_PROJECT_DIR` - Override default project directory resolution
- `ANTHROPIC_API_KEY` - Anthropic API key (alternative to .ktx/config.json)
- `OPENAI_API_KEY` - OpenAI API key for embeddings
- `NO_COLOR` - Disable colored output
- `KTX_TELEMETRY_DISABLED` - Opt out of anonymous telemetry

Example:

```bash
export KTX_PROJECT_DIR=/path/to/project
export ANTHROPIC_API_KEY=sk-ant-...
ktx ingest
```

## Advanced Usage

### Scripting ktx ingestion in CI/CD

```bash
#!/bin/bash
set -e

# Build context in CI pipeline
export KTX_PROJECT_DIR=/workspace/analytics
export ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}"

ktx ingest --non-interactive
ktx sl validate
ktx status --json > ktx-status.json
```

### Monitoring ktx context freshness

```typescript
import { execSync } from 'child_process';

function getContextFreshness(): { lastIngest: Date; isStale: boolean } {
  const status = execSync('ktx status --json').toString();
  const parsed = JSON.parse(status);
  
  const lastIngest = new Date(parsed.lastIngestTimestamp);
  const hoursSinceIngest = (Date.now() - lastIngest.getTime()) / (1000 * 60 * 60);
  
  return {
    lastIngest,
    isStale: hoursSinceIngest > 24
  };
}

const freshness = getContextFreshness();
if (freshness.isStale) {
  console.warn('ktx context is stale - run `ktx ingest`');
}
```

## Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [GitHub Repository](https://github.com/Kaelio/ktx)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
