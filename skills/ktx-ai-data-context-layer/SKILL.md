---
name: ktx-ai-data-context-layer
description: Context layer for AI data agents - teach Claude, Codex, and other agents to query warehouses with approved metrics, semantic layers, and business knowledge through MCP
triggers:
  - set up ktx for data warehouse context
  - configure ktx semantic layer for analytics
  - add ktx to my data project for AI agents
  - create ktx context from dbt and wiki
  - install ktx MCP server for Claude Code
  - build warehouse context layer with ktx
  - integrate ktx with my SQL database
  - use ktx to improve agent data queries
---

# ktx AI Data Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to accurately query data warehouses. It automatically builds semantic layers from dbt, MetricFlow, LookML, Looker, Metabase, and Notion, detects joinable columns, resolves fan/chasm traps, and provides approved metric definitions through CLI and MCP (Model Context Protocol) interfaces.

Use ktx when you need AI agents to query your warehouse with canonical SQL instead of inventing metric logic on every prompt.

## Installation

### Global CLI Install

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Install

```bash
npm install --save-dev @kaelio/ktx
```

### Verify Installation

```bash
ktx --version
ktx --help
```

## Initial Setup

### Interactive Setup

Run the interactive setup wizard to create a new ktx project:

```bash
ktx setup
```

This will:
1. Create or resume a `ktx.yaml` configuration file
2. Configure LLM and embedding providers
3. Set up database connections
4. Configure context sources (dbt, Looker, Notion, etc.)
5. Build initial context
6. Install agent integration (MCP server)

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

## Project Structure

After setup, your project will have:

```
my-project/
├── ktx.yaml                         # Main configuration (commit this)
├── semantic-layer/
│   └── <connection-id>/             # YAML semantic sources (commit)
├── wiki/
│   ├── global/                      # Shared business context (commit)
│   └── user/<user-id>/              # User-scoped notes (commit)
├── raw-sources/
│   └── <connection-id>/             # Ingest artifacts and reports (commit)
└── .ktx/                            # Local state and secrets (DO NOT COMMIT)
```

**Important**: Add `.ktx/` to your `.gitignore`.

## Configuration

### Basic ktx.yaml Structure

```yaml
version: 1
project:
  name: my-analytics-project
  
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key from environment: ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key from environment: OPENAI_API_KEY

databases:
  warehouse:
    type: postgres
    connection:
      host: localhost
      port: 5432
      database: analytics
      user: readonly_user
      # Password from environment: WAREHOUSE_PASSWORD
    schema: public

context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
    database: warehouse
```

### Supported Database Types

- `postgres` - PostgreSQL
- `snowflake` - Snowflake
- `bigquery` - Google BigQuery
- `clickhouse` - ClickHouse
- `mysql` - MySQL
- `sqlserver` - Microsoft SQL Server
- `sqlite` - SQLite

### Supported Context Sources

- `dbt` - dbt projects (models, metrics, sources)
- `metricflow` - MetricFlow semantic layer
- `lookml` - LookML projects
- `looker` - Looker instances
- `metabase` - Metabase instances
- `notion` - Notion workspaces

### LLM Provider Configuration

**Anthropic API:**

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
```

Set environment variable: `ANTHROPIC_API_KEY`

**Google Vertex AI:**

```yaml
llm:
  provider: vertex
  model: claude-sonnet-4-6
  project_id: my-gcp-project
  region: us-east5
```

Set environment variable: `GOOGLE_APPLICATION_CREDENTIALS` (path to service account JSON)

**Claude Code Agent SDK (local session):**

```yaml
llm:
  provider: claude-agent-sdk
```

No API key needed - uses local Claude Code session.

## Core Commands

### Build Context

Ingest all configured sources and build the context layer:

```bash
ktx ingest
```

Ingest a specific connection:

```bash
ktx ingest --connection warehouse
```

Force re-ingestion (skip cache):

```bash
ktx ingest --force
```

### Search Semantic Layer

Search for metrics, dimensions, and entities:

```bash
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer retention"
```

With result limit:

```bash
ktx sl "orders" --limit 5
```

### Search Wiki

Search business knowledge and documentation:

```bash
ktx wiki "refund policy"
ktx wiki "how to calculate churn"
ktx wiki "fiscal year definition"
```

### Query Execution

Execute SQL against configured databases:

```bash
ktx query --connection warehouse "SELECT COUNT(*) FROM orders"
```

Read query from file:

```bash
ktx query --connection warehouse --file query.sql
```

### MCP Server

Start the MCP server for agent integration:

```bash
ktx mcp start
```

Start with specific project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

The MCP server exposes tools for:
- Searching semantic layer entities
- Searching wiki content
- Executing queries
- Getting connection schemas

### Project Management

Reset project (clear all context):

```bash
ktx reset
```

Export project configuration:

```bash
ktx export > backup.yaml
```

## Real-World Usage Examples

### Example 1: Setting Up ktx for dbt Project

```bash
# Navigate to your dbt project
cd /path/to/dbt-project

# Initialize ktx
ktx setup

# When prompted, configure:
# - LLM: Anthropic Claude Sonnet 4
# - Embeddings: OpenAI text-embedding-3-small
# - Database: PostgreSQL (your warehouse)
# - Context source: dbt (./dbt directory)

# Build context
ktx ingest

# Test semantic layer search
ktx sl "revenue"

# Start MCP server for agents
ktx mcp start
```

### Example 2: Programmatic Configuration

Create `ktx.yaml` manually:

```yaml
version: 1
project:
  name: ecommerce-analytics

llm:
  provider: anthropic
  model: claude-sonnet-4-6

embeddings:
  provider: openai
  model: text-embedding-3-small

databases:
  warehouse:
    type: snowflake
    connection:
      account: xy12345.us-east-1
      database: ANALYTICS
      warehouse: COMPUTE_WH
      schema: PUBLIC
      user: ktx_readonly

context_sources:
  dbt_models:
    type: dbt
    path: ./dbt
    database: warehouse
    
  notion_docs:
    type: notion
    database_id: abc123def456
```

Set environment variables:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export SNOWFLAKE_PASSWORD=...
export NOTION_API_KEY=secret_...
```

Then run:

```bash
ktx ingest
ktx status
```

### Example 3: Searching and Querying from TypeScript

While ktx is primarily a CLI tool, you can shell out to it from code:

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function searchSemanticLayer(query: string): Promise<string> {
  const { stdout } = await execAsync(`ktx sl "${query}" --limit 10`);
  return stdout;
}

async function searchWiki(query: string): Promise<string> {
  const { stdout } = await execAsync(`ktx wiki "${query}"`);
  return stdout;
}

async function executeQuery(sql: string, connection: string): Promise<string> {
  const { stdout } = await execAsync(
    `ktx query --connection ${connection} "${sql}"`
  );
  return stdout;
}

// Usage
const metrics = await searchSemanticLayer('monthly revenue');
console.log('Found metrics:', metrics);

const docs = await searchWiki('how to calculate LTV');
console.log('Wiki results:', docs);

const results = await executeQuery(
  'SELECT COUNT(*) FROM orders WHERE status = \'completed\'',
  'warehouse'
);
console.log('Query results:', results);
```

### Example 4: Multi-Source Context Setup

```yaml
version: 1
project:
  name: multi-source-analytics

llm:
  provider: anthropic
  model: claude-sonnet-4-6

embeddings:
  provider: openai
  model: text-embedding-3-small

databases:
  warehouse:
    type: postgres
    connection:
      host: db.example.com
      port: 5432
      database: analytics
      user: ktx_user
      
  clickhouse_events:
    type: clickhouse
    connection:
      host: clickhouse.example.com
      port: 8123
      database: events
      user: readonly

context_sources:
  dbt_models:
    type: dbt
    path: ./dbt
    database: warehouse
    
  metricflow_layer:
    type: metricflow
    path: ./semantic-layer
    database: warehouse
    
  looker_instance:
    type: looker
    base_url: https://looker.example.com
    database: warehouse
    
  team_wiki:
    type: notion
    database_id: notion-db-id
```

Build context from all sources:

```bash
ktx ingest
```

This will:
1. Scan both database connections
2. Parse dbt models and metrics
3. Import MetricFlow definitions
4. Fetch Looker LookML
5. Pull Notion pages
6. Detect contradictions across sources
7. Build unified semantic layer

## Agent Integration Patterns

### Claude Code / Claude Desktop

After running `ktx setup`, the MCP server is auto-configured. Ensure the server is running:

```bash
ktx mcp start --project-dir /path/to/project
```

In Claude, you can now ask:

```
Search ktx for revenue metrics and show me how to query monthly revenue by product category
```

```
Use ktx to find the definition of customer lifetime value in our wiki
```

### Codex

Install the ktx skill:

```bash
npx skills add Kaelio/ktx --skill ktx
```

Or ask Codex:

```
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to set up ktx in this project
```

### Cursor / OpenCode

From your project directory with `ktx.yaml`:

```bash
ktx mcp start
```

Configure your agent's MCP settings to point to the ktx server socket (printed by `ktx mcp start`).

## Common Patterns

### Pattern 1: Incremental Context Updates

After adding new dbt models or wiki pages:

```bash
# Quick re-ingest (uses caching)
ktx ingest

# Force full rebuild
ktx ingest --force
```

### Pattern 2: Connection-Specific Operations

Work with a specific database:

```bash
# Ingest only warehouse connection
ktx ingest --connection warehouse

# Query specific connection
ktx query --connection warehouse "SELECT * FROM customers LIMIT 10"

# Search metrics from specific connection
ktx sl "revenue" --connection warehouse
```

### Pattern 3: Project Directory Override

Run ktx commands from anywhere:

```bash
ktx status --project-dir /path/to/analytics
ktx ingest --project-dir /path/to/analytics
ktx mcp start --project-dir /path/to/analytics
```

Or set environment variable:

```bash
export KTX_PROJECT_DIR=/path/to/analytics
ktx status
ktx ingest
```

### Pattern 4: CI/CD Integration

In GitHub Actions or similar:

```yaml
- name: Install ktx
  run: npm install -g @kaelio/ktx

- name: Build ktx context
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    WAREHOUSE_PASSWORD: ${{ secrets.WAREHOUSE_PASSWORD }}
  run: |
    ktx ingest --project-dir ./analytics
    ktx status --project-dir ./analytics
```

### Pattern 5: Multiple Environment Setup

Development:

```yaml
# ktx.dev.yaml
databases:
  warehouse:
    connection:
      host: dev-db.example.com
```

Production:

```yaml
# ktx.prod.yaml
databases:
  warehouse:
    connection:
      host: prod-db.example.com
```

Use with `--config` flag:

```bash
ktx ingest --config ktx.dev.yaml
ktx ingest --config ktx.prod.yaml
```

## Troubleshooting

### MCP Server Won't Start

**Issue**: `ktx mcp start` fails or agents can't connect.

**Solution**:

1. Check if another instance is running:
   ```bash
   ps aux | grep "ktx mcp"
   ```

2. Kill existing processes:
   ```bash
   pkill -f "ktx mcp"
   ```

3. Restart with explicit project directory:
   ```bash
   ktx mcp start --project-dir $(pwd)
   ```

4. Check agent configuration points to correct socket path (shown in `ktx status` output).

### Database Connection Fails

**Issue**: `ktx ingest` fails with connection error.

**Solution**:

1. Test connection manually (e.g., `psql` for PostgreSQL):
   ```bash
   psql -h localhost -U readonly_user -d analytics
   ```

2. Verify credentials are in environment:
   ```bash
   echo $WAREHOUSE_PASSWORD
   ```

3. Check firewall/network access to database.

4. Ensure user has read permissions on target schemas.

### Slow Ingestion

**Issue**: `ktx ingest` takes too long.

**Solution**:

1. Use connection-specific ingestion:
   ```bash
   ktx ingest --connection warehouse
   ```

2. Reduce table sampling (edit `ktx.yaml`):
   ```yaml
   databases:
     warehouse:
       sampling:
         max_rows: 1000  # default is 10000
   ```

3. Exclude large tables:
   ```yaml
   databases:
     warehouse:
       exclude_tables:
         - raw_events
         - clickstream_logs
   ```

### Search Returns No Results

**Issue**: `ktx sl "metric"` or `ktx wiki "topic"` returns nothing.

**Solution**:

1. Rebuild context:
   ```bash
   ktx ingest --force
   ```

2. Check that sources are configured:
   ```bash
   ktx status
   ```

3. Verify source files exist (for dbt, check `./dbt` directory).

4. Try broader search terms:
   ```bash
   ktx sl "revenue"  # instead of "daily_revenue_by_region"
   ```

### LLM Rate Limits

**Issue**: Ingestion fails with rate limit errors.

**Solution**:

1. For Anthropic, use tier-appropriate API key with higher limits.

2. Use Vertex AI instead (no rate limits):
   ```yaml
   llm:
     provider: vertex
     model: claude-sonnet-4-6
     project_id: my-gcp-project
     region: us-east5
   ```

3. Run ingestion during off-peak hours.

4. Use Claude Code session (local, no limits):
   ```yaml
   llm:
     provider: claude-agent-sdk
   ```

### Contradicting Metric Definitions

**Issue**: `ktx ingest` reports contradictions between sources.

**Solution**:

This is expected! ktx flags inconsistencies for human review.

1. Check ingestion report:
   ```bash
   cat raw-sources/<connection-id>/report.json
   ```

2. Review contradictions and pick canonical definition.

3. Update wiki to clarify:
   ```bash
   echo "# Revenue Definition\n\nUse dbt metric revenue_total, not Looker's total_revenue." > wiki/global/revenue-definition.md
   ```

4. Re-ingest:
   ```bash
   ktx ingest --force
   ```

## Advanced Configuration

### Custom Embedding Dimensions

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-small
  dimensions: 1536  # default, can reduce to 512 for speed
```

### Sampling Strategy

```yaml
databases:
  warehouse:
    sampling:
      max_rows: 5000
      strategy: random  # or 'top'
```

### Wiki Organization

Place files in `wiki/global/` for team-wide context:

```
wiki/global/
├── metrics-definitions.md
├── data-quality-rules.md
└── business-glossary.md
```

User-specific notes go in `wiki/user/<user-id>/`:

```
wiki/user/alice/
├── analysis-notes.md
└── query-templates.md
```

### Exclude Patterns

```yaml
databases:
  warehouse:
    exclude_schemas:
      - temp
      - staging
    exclude_tables:
      - audit_logs
      - raw_*
```

## Environment Variables Reference

| Variable | Description | Example |
|----------|-------------|---------|
| `ANTHROPIC_API_KEY` | Anthropic API key | `sk-ant-api03-...` |
| `OPENAI_API_KEY` | OpenAI API key | `sk-proj-...` |
| `GOOGLE_APPLICATION_CREDENTIALS` | Path to GCP service account JSON | `/path/to/key.json` |
| `WAREHOUSE_PASSWORD` | Database password | `secure_password` |
| `SNOWFLAKE_PASSWORD` | Snowflake password | `snowflake_pass` |
| `NOTION_API_KEY` | Notion integration token | `secret_...` |
| `KTX_PROJECT_DIR` | Override project directory | `/path/to/project` |

## Further Resources

- [Official Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Building Context Guide](https://docs.kaelio.com/ktx/docs/guides/building-context)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
