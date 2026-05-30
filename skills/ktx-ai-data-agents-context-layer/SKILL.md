---
name: ktx-ai-data-agents-context-layer
description: Context layer for AI data agents with semantic layer, wiki, and MCP integration for accurate warehouse queries
triggers:
  - set up ktx for data agent context
  - configure ktx semantic layer for AI queries
  - integrate ktx with Claude Code or Cursor
  - build ktx context from warehouse and wiki
  - use ktx MCP tools for data analysis
  - query semantic layer with ktx
  - ingest dbt or LookML into ktx
  - troubleshoot ktx agent integration
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It combines semantic layer definitions, wiki knowledge, and warehouse metadata into a searchable surface exposed via CLI and MCP tools. Agents get approved metric definitions, joinable columns, and business context instead of re-exploring schemas on every question.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content (Notion, local docs), organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds semantic layer**: Combines raw tables and metrics through a join graph that auto-resolves chasm and fan traps
- **Serves agents**: Exposes CLI and MCP tools with full-text + semantic search across wiki and semantic-layer entities

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Project Setup

```bash
ktx setup
```

Interactive setup will:
1. Create or resume a local ktx project
2. Configure LLM provider (Anthropic, Google Vertex, AI Gateway, Claude Code session)
3. Configure embedding provider (OpenAI, Voyage, Google Vertex)
4. Configure database connections (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite)
5. Configure context sources (dbt, MetricFlow, LookML, Looker, Metabase, Notion)
6. Build initial context
7. Install agent integration (MCP)

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
project:
  name: analytics-warehouse
  
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  
embeddings:
  provider: openai
  model: text-embedding-3-small

connections:
  - id: warehouse
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    ssl: false

context_sources:
  - id: dbt_main
    type: dbt
    path: ./dbt
    connection_id: warehouse
```

### Environment Variables

Store secrets in environment variables or `.ktx/.env`:

```bash
# LLM Provider
ANTHROPIC_API_KEY=your_api_key_here
# or
GOOGLE_CLOUD_PROJECT=your_project_id
GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json

# Embeddings Provider
OPENAI_API_KEY=your_api_key_here
# or
VOYAGE_API_KEY=your_api_key_here

# Database Connections
POSTGRES_PASSWORD=your_db_password
SNOWFLAKE_PASSWORD=your_snowflake_password
```

## Key Commands

### Build Context

Ingest all configured connections and sources:

```bash
ktx ingest
```

Ingest specific connection:

```bash
ktx ingest --connection-id warehouse
```

### Search Semantic Layer

```bash
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer lifetime value"
```

Returns metrics, dimensions, and join paths with descriptions and SQL.

### Search Wiki

```bash
ktx wiki "refund policy"
ktx wiki "revenue recognition"
ktx wiki "data quality SLA"
```

Searches all wiki content (global and user-scoped) with semantic + full-text ranking.

### MCP Server

Start MCP server for agent clients:

```bash
ktx mcp start
```

With custom project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

The MCP server exposes tools:
- `ktx_sl_search`: Search semantic layer
- `ktx_wiki_search`: Search wiki
- `ktx_execute_query`: Execute read-only SQL queries
- `ktx_get_schema`: Get table schema details

### Manage Wiki

Add wiki page from file:

```bash
ktx wiki add --file ./docs/metrics-guide.md
```

Add wiki page with content:

```bash
ktx wiki add --title "Revenue Recognition Policy" --content "Revenue is recognized when..."
```

List wiki pages:

```bash
ktx wiki list
```

### Manage Semantic Sources

List semantic sources:

```bash
ktx sl list
```

Validate semantic source YAML:

```bash
ktx sl validate --file semantic-layer/warehouse/revenue.yaml
```

## Real Code Examples

### TypeScript: Using ktx in Node.js Script

```typescript
import { execSync } from 'child_process';

// Search semantic layer programmatically
function searchSemanticLayer(query: string): string {
  const result = execSync(`ktx sl "${query}"`, { encoding: 'utf-8' });
  return result;
}

// Execute query through ktx
function executeQuery(sql: string, connectionId: string): string {
  const result = execSync(
    `ktx execute --connection-id ${connectionId} --sql "${sql}"`,
    { encoding: 'utf-8' }
  );
  return result;
}

// Example usage
const revenueMetrics = searchSemanticLayer('revenue');
console.log(revenueMetrics);
```

### Python: Calling ktx CLI

```python
import subprocess
import json

def search_wiki(query: str) -> dict:
    """Search ktx wiki and parse JSON output."""
    result = subprocess.run(
        ['ktx', 'wiki', query, '--json'],
        capture_output=True,
        text=True,
        check=True
    )
    return json.loads(result.stdout)

def ingest_sources():
    """Trigger ktx context ingestion."""
    subprocess.run(['ktx', 'ingest'], check=True)

# Example usage
wiki_results = search_wiki('customer churn definition')
for page in wiki_results.get('pages', []):
    print(f"{page['title']}: {page['excerpt']}")
```

### Semantic Layer YAML Definition

Create `semantic-layer/warehouse/customers.yaml`:

```yaml
version: 1
sources:
  - id: customers_monthly
    type: metric
    name: Monthly Active Customers
    description: Count of unique customers with at least one purchase in the month
    sql: |
      SELECT
        DATE_TRUNC('month', order_date) AS month,
        COUNT(DISTINCT customer_id) AS value
      FROM orders
      WHERE order_date >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '12 months')
      GROUP BY 1
    dimensions:
      - month
    metrics:
      - value
    grain: month
    connection_id: warehouse

  - id: customer_lifetime_value
    type: metric
    name: Customer Lifetime Value
    description: Total revenue per customer over their lifetime
    sql: |
      SELECT
        customer_id,
        SUM(order_total) AS value
      FROM orders
      GROUP BY 1
    dimensions:
      - customer_id
    metrics:
      - value
    grain: customer_id
    connection_id: warehouse
```

### Wiki Page Markdown

Create `wiki/global/revenue-recognition.md`:

```markdown
---
title: Revenue Recognition Policy
tags: [finance, revenue, policy]
---

# Revenue Recognition Policy

## Overview

Revenue is recognized when services are delivered and payment is received or reasonably assured.

## Monthly Recurring Revenue (MRR)

- **Definition**: Normalized monthly value of active subscriptions
- **Calculation**: Annual contracts divided by 12, monthly contracts at face value
- **Exclusions**: One-time fees, usage overages, credits

## Annual Recurring Revenue (ARR)

- **Definition**: MRR × 12
- **When to Use**: Board metrics, investor reporting

## Related Metrics

- See `revenue_mrr` metric in semantic layer
- See `revenue_arr` metric in semantic layer
```

## Agent Integration Patterns

### Claude Code / Codex / Cursor

After running `ktx setup`, the MCP integration is automatically configured. In your agent:

```text
What are our revenue metrics?
```

The agent will:
1. Call `ktx_sl_search` with query "revenue"
2. Get metric definitions with SQL
3. Optionally execute queries via `ktx_execute_query`

### Custom MCP Client

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const transport = new StdioClientTransport({
  command: 'ktx',
  args: ['mcp', 'start', '--project-dir', '/path/to/project'],
});

const client = new Client(
  { name: 'my-agent', version: '1.0.0' },
  { capabilities: {} }
);

await client.connect(transport);

// Search semantic layer
const result = await client.callTool({
  name: 'ktx_sl_search',
  arguments: { query: 'revenue', limit: 5 },
});

console.log(result);
```

## Common Patterns

### Adding dbt Project as Context Source

```bash
ktx setup
# During setup, select "dbt" as context source type
# Provide path to dbt project directory
# ktx will ingest models, metrics, and documentation
```

Or manually edit `ktx.yaml`:

```yaml
context_sources:
  - id: dbt_main
    type: dbt
    path: ./dbt
    connection_id: warehouse
```

Then run:

```bash
ktx ingest --connection-id warehouse
```

### Adding Notion as Context Source

```bash
ktx setup
# Select "notion" as context source
# Provide Notion integration token and page IDs
```

Or in `ktx.yaml`:

```yaml
context_sources:
  - id: notion_docs
    type: notion
    integration_token: ${NOTION_INTEGRATION_TOKEN}
    page_ids:
      - abc123
      - def456
```

### Continuous Context Updates

Set up a cron job or CI pipeline:

```bash
#!/bin/bash
# update-ktx-context.sh

cd /path/to/project
ktx ingest --connection-id warehouse
ktx ingest --connection-id dbt_main
```

Run daily or after dbt runs to keep context fresh.

### Multi-Warehouse Setup

```yaml
connections:
  - id: prod_warehouse
    type: snowflake
    account: myaccount
    database: PROD
    warehouse: COMPUTE_WH
    schema: PUBLIC

  - id: staging_warehouse
    type: snowflake
    account: myaccount
    database: STAGING
    warehouse: COMPUTE_WH
    schema: PUBLIC

context_sources:
  - id: dbt_prod
    type: dbt
    path: ./dbt
    connection_id: prod_warehouse

  - id: dbt_staging
    type: dbt
    path: ./dbt
    connection_id: staging_warehouse
```

## Troubleshooting

### MCP Server Not Starting

**Symptom**: Agent can't connect to ktx MCP server

**Solution**:
1. Check `ktx status` output for MCP command
2. Run the exact command shown (e.g., `ktx mcp start --project-dir /path`)
3. Verify MCP server is running before opening agent client

```bash
ktx status
# If output shows: "Run: ktx mcp start --project-dir /home/user/project"
ktx mcp start --project-dir /home/user/project
```

### LLM Provider Authentication Errors

**Symptom**: `Error: Authentication failed`

**Solution**:
1. Check environment variables are set correctly
2. For Anthropic: `ANTHROPIC_API_KEY`
3. For Google Vertex: `GOOGLE_APPLICATION_CREDENTIALS` and `GOOGLE_CLOUD_PROJECT`
4. Verify API key has proper permissions

```bash
# Check if env var is set
echo $ANTHROPIC_API_KEY

# Re-run setup to reconfigure
ktx setup
```

### Database Connection Failures

**Symptom**: `Error: Connection refused` or timeout errors

**Solution**:
1. Verify database is accessible from ktx machine
2. Check firewall rules and network policies
3. Test connection manually (e.g., `psql`, `snowsql`)
4. Verify credentials in `.ktx/.env` or environment

```bash
# Test PostgreSQL connection
psql -h localhost -p 5432 -U username -d database

# Verify ktx can connect
ktx execute --connection-id warehouse --sql "SELECT 1"
```

### Context Not Building

**Symptom**: `ktx ingest` completes but `ktx sl` returns no results

**Solution**:
1. Check `raw-sources/` directory for ingest artifacts
2. Review ingestion logs for errors
3. Verify dbt/LookML paths are correct
4. Ensure database connection works

```bash
# Re-run ingest with verbose output
ktx ingest --connection-id warehouse --verbose

# Check raw sources directory
ls -la raw-sources/warehouse/
```

### Semantic Layer Join Errors

**Symptom**: Agent gets incorrect results or join failures

**Solution**:
1. Review join graph in semantic layer YAML
2. Check for fan/chasm traps in metrics
3. Validate grain definitions match actual data
4. Use `ktx sl validate` to check YAML syntax

```bash
ktx sl validate --file semantic-layer/warehouse/revenue.yaml
```

### Wiki Search Returns Stale Content

**Symptom**: Updated wiki pages don't appear in search results

**Solution**:
1. Re-ingest wiki sources
2. Check file timestamps in `wiki/` directory
3. Verify embeddings provider is working

```bash
# Re-ingest all sources
ktx ingest

# Or re-add specific wiki page
ktx wiki add --file ./docs/updated-policy.md --force
```

### Project Not Found

**Symptom**: `Error: No ktx project found`

**Solution**:
1. Ensure `ktx.yaml` exists in project directory
2. Use `--project-dir` flag to specify location
3. Set `KTX_PROJECT_DIR` environment variable

```bash
# Explicit project directory
ktx status --project-dir /path/to/project

# Or set environment variable
export KTX_PROJECT_DIR=/path/to/project
ktx status
```

## Advanced Usage

### Custom Embedding Model

```yaml
embeddings:
  provider: voyage
  model: voyage-large-2-instruct
```

Requires `VOYAGE_API_KEY` environment variable.

### Read-Only Database User

ktx only needs SELECT permissions. Create restricted user:

```sql
-- PostgreSQL example
CREATE USER ktx_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE analytics TO ktx_readonly;
GRANT USAGE ON SCHEMA public TO ktx_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ktx_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO ktx_readonly;
```

### Project-Specific LLM Configuration

Override global LLM settings per project:

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  max_tokens: 4096
  temperature: 0.0
```

### Telemetry Opt-Out

Disable anonymous usage telemetry:

```bash
export KTX_TELEMETRY_DISABLED=1
ktx setup
```

Or add to `.ktx/.env`:

```bash
KTX_TELEMETRY_DISABLED=1
```

## Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
