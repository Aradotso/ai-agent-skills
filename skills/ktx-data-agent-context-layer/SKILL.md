---
name: ktx-data-agent-context-layer
description: An executable context layer that teaches AI agents to query data warehouses accurately through MCP with skills, memory, and a semantic layer
triggers:
  - set up ktx for data analysis
  - configure ktx semantic layer
  - integrate ktx with this agent
  - build context from my data warehouse
  - search metrics using ktx
  - ingest dbt models into ktx
  - query warehouse through ktx
  - add ktx wiki documentation
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It automatically builds semantic layers from your warehouse metadata, dbt models, BI tools, and company wikis, then exposes that context through MCP (Model Context Protocol) and CLI tools. This enables agents to fetch approved metrics and write better SQL instead of reinventing logic on every query.

## What ktx Does

- **Builds warehouse context automatically** — samples tables, captures metadata, detects joinable columns
- **Ingests company knowledge** — from dbt, LookML, Looker, Metabase, Notion, and wiki content
- **Creates a semantic layer** — with join graphs that resolve fan/chasm traps
- **Flags contradictions** — across different knowledge sources
- **Serves agents via MCP** — with full-text and semantic search across wiki and metrics
- **Read-only by design** — never writes to your database

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite.

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install --save-dev @kaelio/ktx
```

### First-Time Setup

```bash
ktx setup
```

This command:
1. Creates or resumes a local ktx project
2. Configures LLM and embedding providers
3. Sets up database connections
4. Configures context sources (dbt, wikis, etc.)
5. Builds initial context
6. Installs agent integration

## Project Structure

ktx creates the following structure in your project:

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources (commit)
├── wiki/global/                     # Shared business context (commit)
├── wiki/user/<user-id>/             # User-scoped notes (commit)
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignore)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Add `.ktx/` to `.gitignore`.

## Core Commands

### Project Management

```bash
# Check project status
ktx status

# Re-run setup wizard
ktx setup

# Initialize new project manually
ktx init --project-dir ./analytics

# Validate configuration
ktx config validate
```

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific source
ktx ingest --source dbt_main

# Force re-ingestion
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, entities)
ktx sl "revenue"
ktx sl "monthly active users"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "customer churn definition"

# List all semantic sources
ktx sl --list

# Show specific metric details
ktx sl "arr" --show-yaml
```

### MCP Server

```bash
# Start MCP server for agent integration
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir ./analytics

# Check MCP status
ktx mcp status
```

## Configuration

### ktx.yaml Example

```yaml
project:
  name: analytics
  version: 1.0.0

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key from env: ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key from env: OPENAI_API_KEY

connections:
  - id: warehouse
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    user: analyst
    # Password from env: WAREHOUSE_PASSWORD
    schemas:
      - public
      - analytics

  - id: snowflake_prod
    type: snowflake
    account: xy12345.us-east-1
    warehouse: compute_wh
    database: prod
    # User/password from env: SNOWFLAKE_USER, SNOWFLAKE_PASSWORD

context_sources:
  - id: dbt_main
    type: dbt
    connection: warehouse
    manifest_path: ./target/manifest.json
    catalog_path: ./target/catalog.json

  - id: company_wiki
    type: notion
    # Token from env: NOTION_TOKEN
    database_id: abc123def456

  - id: looker_prod
    type: looker
    base_url: https://company.looker.com
    # Client ID/secret from env: LOOKER_CLIENT_ID, LOOKER_CLIENT_SECRET
```

### Environment Variables

Store secrets in environment variables:

```bash
# LLM providers
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Database connections
export WAREHOUSE_PASSWORD=...
export SNOWFLAKE_USER=...
export SNOWFLAKE_PASSWORD=...

# Context sources
export NOTION_TOKEN=secret_...
export LOOKER_CLIENT_ID=...
export LOOKER_CLIENT_SECRET=...

# Optional: custom project directory
export KTX_PROJECT_DIR=/path/to/analytics
```

## TypeScript Integration

### Using ktx Programmatically

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load project
const project = await KtxProject.load({ projectDir: './analytics' });

// Search semantic layer
const metrics = await project.searchSemanticLayer('revenue', {
  limit: 10,
  threshold: 0.7
});

for (const result of metrics) {
  console.log(`${result.name}: ${result.description}`);
  console.log(`Type: ${result.type}`);
  console.log(`Score: ${result.score}`);
}

// Search wiki
const wikiPages = await project.searchWiki('customer churn', {
  limit: 5
});

for (const page of wikiPages) {
  console.log(`${page.title}: ${page.content.substring(0, 100)}...`);
}
```

### Working with Semantic Sources

```typescript
import { SemanticLayer } from '@kaelio/ktx';

const sl = await SemanticLayer.load({
  projectDir: './analytics',
  connectionId: 'warehouse'
});

// Get metric definition
const metric = await sl.getMetric('arr');
console.log(metric.sql);
console.log(metric.dimensions);

// List all entities
const entities = await sl.listEntities();
for (const entity of entities) {
  console.log(`${entity.name} (${entity.table})`);
}

// Get join graph
const graph = await sl.getJoinGraph();
console.log(graph.paths);
```

## Common Patterns

### Integrating dbt Models

```bash
# 1. Build dbt manifest and catalog
cd /path/to/dbt
dbt compile
dbt docs generate

# 2. Configure in ktx.yaml
# context_sources:
#   - id: dbt_main
#     type: dbt
#     connection: warehouse
#     manifest_path: ./dbt/target/manifest.json
#     catalog_path: ./dbt/target/catalog.json

# 3. Ingest
ktx ingest --source dbt_main
```

### Adding Wiki Documentation

```bash
# Create wiki page for business concept
mkdir -p wiki/global
cat > wiki/global/revenue-definitions.md << 'EOF'
# Revenue Definitions

## Annual Recurring Revenue (ARR)
Sum of all active subscription values normalized to annual amounts.
Excludes one-time fees and professional services.

## Monthly Recurring Revenue (MRR)
ARR divided by 12. Updated daily based on active subscriptions.
EOF

# Ingest wiki content
ktx ingest --source wiki
```

### Querying Through MCP in Claude Code

After running `ktx mcp start`, use in Claude Code:

```
Use the ktx skill to search for metrics related to "revenue"
```

```
Use ktx to find our definition of "customer churn"
```

```
Use ktx to get the SQL for calculating ARR
```

### Creating Custom Metrics

```yaml
# semantic-layer/warehouse/metrics.yaml
metrics:
  - name: arr
    description: Annual Recurring Revenue
    type: metric
    sql: |
      SELECT 
        DATE_TRUNC('month', subscription_start_date) as period,
        SUM(CASE 
          WHEN billing_period = 'annual' THEN amount
          WHEN billing_period = 'monthly' THEN amount * 12
          WHEN billing_period = 'quarterly' THEN amount * 4
        END) as arr
      FROM subscriptions
      WHERE status = 'active'
      GROUP BY 1
    dimensions:
      - period
      - customer_segment
      - plan_type
    entity: subscription
```

### Detecting Join Paths

```typescript
import { JoinGraphBuilder } from '@kaelio/ktx';

const builder = new JoinGraphBuilder({
  connection: 'warehouse',
  projectDir: './analytics'
});

// Build join graph from sampled data
await builder.buildFromIntrospection();

// Get possible join paths between entities
const paths = await builder.findPaths('customer', 'order');

for (const path of paths) {
  console.log(`Path: ${path.tables.join(' -> ')}`);
  console.log(`Join keys: ${path.joinKeys.join(', ')}`);
  console.log(`Cardinality: ${path.cardinality}`);
}
```

## Agent Integration

### Claude Code Integration

ktx integrates automatically with Claude Code via MCP:

1. Run `ktx setup` and select Claude Code integration
2. ktx updates your Claude Code config at `~/.claude/claude_desktop_config.json`
3. Restart Claude Code
4. Use natural language to query: "Show me our revenue metrics"

### Codex/Cursor Integration

For Codex or Cursor, manually configure MCP:

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

## Troubleshooting

### "ktx mcp start --project-dir ..." in Status

If `ktx status` prints a command to run:

```bash
# Copy and run the exact command shown
ktx mcp start --project-dir /path/to/project
```

This starts the MCP server. Keep it running while using agent clients.

### Connection Failures

```bash
# Test connection manually
ktx config validate --connection warehouse

# Check environment variables
echo $WAREHOUSE_PASSWORD
echo $ANTHROPIC_API_KEY

# Re-run setup with verbose output
ktx setup --verbose
```

### Ingestion Issues

```bash
# Clear cache and re-ingest
ktx ingest --force --source dbt_main

# Check ingestion logs
cat raw-sources/warehouse/ingestion.log

# Validate source configuration
ktx config validate --source dbt_main
```

### Semantic Search Not Working

```bash
# Check embeddings configuration
ktx config validate

# Rebuild embeddings index
ktx ingest --rebuild-index

# Test search directly
ktx sl "test query" --debug
```

### MCP Server Not Responding

```bash
# Check MCP status
ktx mcp status

# Restart MCP server
ktx mcp stop
ktx mcp start

# Check logs
ktx mcp logs
```

### Missing Metrics After dbt Update

```bash
# Regenerate dbt artifacts
cd /path/to/dbt
dbt compile
dbt docs generate

# Re-ingest
ktx ingest --source dbt_main --force
```

## Advanced Usage

### Custom LLM Configuration

```yaml
# Use Claude Code session (no API key needed)
llm:
  provider: claude_code_session

# Use Google Vertex AI
llm:
  provider: vertex_ai
  model: claude-3-5-sonnet-20241022
  project_id: my-gcp-project
  region: us-central1

# Use AI Gateway
llm:
  provider: ai_gateway
  api_url: https://gateway.company.com/v1
  model: claude-sonnet-4-6
```

### Multi-Connection Setup

```yaml
connections:
  - id: prod
    type: snowflake
    account: prod.snowflakecomputing.com
    warehouse: prod_wh
    database: analytics

  - id: staging
    type: postgres
    host: staging-db.company.com
    database: analytics_staging

context_sources:
  - id: prod_dbt
    type: dbt
    connection: prod
    manifest_path: ./dbt/prod/target/manifest.json

  - id: staging_dbt
    type: dbt
    connection: staging
    manifest_path: ./dbt/staging/target/manifest.json
```

### Scripting with ktx

```bash
#!/bin/bash
set -e

# Daily context refresh
export KTX_PROJECT_DIR=/home/analytics/ktx-project

# Update dbt
cd /home/dbt
dbt compile
dbt docs generate

# Re-ingest everything
ktx ingest --all --force

# Generate reports
ktx sl --list > /tmp/metrics-catalog.txt
ktx wiki --list > /tmp/wiki-index.txt

# Notify team
echo "Context refreshed: $(date)" | slack-cli send analytics
```

## Best Practices

1. **Commit semantic layer and wiki**: These are your source of truth
2. **Gitignore `.ktx/`**: Contains secrets and local cache
3. **Use environment variables**: Never commit API keys or passwords
4. **Run `ktx ingest` regularly**: Keep context fresh (daily or after dbt runs)
5. **Review contradiction reports**: ktx flags conflicts between sources
6. **Start MCP server before agent**: Required for MCP integration
7. **Document in wiki/**: Add business context agents can search
8. **Validate after changes**: Run `ktx config validate` after editing `ktx.yaml`

## Resources

- **Documentation**: https://docs.kaelio.com/ktx
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues**: https://github.com/Kaelio/ktx/issues
- **License**: Apache 2.0
