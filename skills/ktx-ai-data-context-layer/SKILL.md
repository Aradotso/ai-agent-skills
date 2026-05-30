---
name: ktx-ai-data-context-layer
description: Configure and use ktx to build an executable context layer for AI agents querying data warehouses with semantic layers, wiki knowledge, and approved metrics
triggers:
  - set up ktx for data agent context
  - configure ktx semantic layer and warehouse
  - ingest data context with ktx
  - search ktx semantic layer and wiki
  - connect ai agent to ktx mcp server
  - build warehouse context for claude code
  - query approved metrics with ktx
  - troubleshoot ktx setup and ingestion
---

# ktx AI Data Context Layer Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It automatically builds and maintains approved metric definitions, detects joinable columns, absorbs business knowledge from wikis and semantic layers, and exposes everything through CLI and MCP (Model Context Protocol) tools.

**Key capabilities:**
- Ingests from dbt, MetricFlow, LookML, Looker, Metabase, Notion
- Auto-detects joinable columns and resolves fan/chasm traps
- Combines warehouse metadata, semantic layers, and wiki knowledge
- Exposes search and query tools via MCP for agent execution
- Read-only by design — never writes to your warehouse

**Supported warehouses:** PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite

## Installation

### Global CLI Install

```bash
npm install -g @kaelio/ktx
```

### Project-Scoped Install

```bash
npm install --save-dev @kaelio/ktx
npx ktx setup
```

### Verify Installation

```bash
ktx --version
ktx status
```

## Initial Setup

### Interactive Setup Wizard

```bash
ktx setup
```

The setup wizard will:
1. Create or resume a `ktx.yaml` project configuration
2. Configure LLM provider (Anthropic API, Vertex AI, AI Gateway, or Claude Code)
3. Configure embedding provider (OpenAI, Voyage AI, or Google)
4. Set up database connections (warehouse credentials)
5. Configure context sources (dbt, Looker, Metabase, Notion, etc.)
6. Run initial ingestion to build context
7. Install agent integration (Codex, Claude Code, Cursor, OpenCode)

### Project Structure

```
my-analytics-project/
├── ktx.yaml                         # Main configuration (commit this)
├── semantic-layer/warehouse/        # Generated semantic sources (commit)
├── wiki/global/                     # Shared business context (commit)
├── wiki/user/alice/                 # User-scoped notes (commit)
├── raw-sources/warehouse/           # Ingest artifacts (commit)
└── .ktx/                            # Local state and secrets (git-ignore)
```

### Manual Configuration

Create `ktx.yaml`:

```yaml
version: 1
project:
  name: my-analytics
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
    schema: public
    # Credentials from env vars or .ktx/secrets.yaml

context_sources:
  dbt_main:
    type: dbt
    project_dir: ../dbt-project
    profiles_dir: ~/.dbt
    target: prod
```

Store secrets in `.ktx/secrets.yaml` (never commit):

```yaml
databases:
  warehouse:
    user: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}

llm:
  anthropic_api_key: ${ANTHROPIC_API_KEY}

embeddings:
  openai_api_key: ${OPENAI_API_KEY}
```

## Core Commands

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
Context sources configured: yes (dbt_main, looker_main)
ktx context built: yes
Agent integration ready: yes (codex:project)
```

### Build Context (Ingestion)

Ingest from all configured sources:

```bash
ktx ingest
```

Ingest specific connection:

```bash
ktx ingest --connection warehouse
```

Ingest specific context source:

```bash
ktx ingest --source dbt_main
```

Force re-ingestion:

```bash
ktx ingest --force
```

### Search Semantic Layer

```bash
# Search for metrics and dimensions
ktx sl "revenue"
ktx sl "customer lifetime value"

# JSON output for scripting
ktx sl "churn rate" --json
```

### Search Wiki

```bash
# Search business knowledge
ktx wiki "refund policy"
ktx wiki "how to calculate mrr"

# JSON output
ktx wiki "data retention" --json
```

### Start MCP Server

```bash
# Auto-detect project and start MCP server
ktx mcp start

# Specify project directory
ktx mcp start --project-dir /path/to/project

# Custom port
ktx mcp start --port 3000
```

## LLM Configuration

### Anthropic API

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  api_key: ${ANTHROPIC_API_KEY}  # In .ktx/secrets.yaml
```

### Google Vertex AI

```yaml
llm:
  provider: vertex
  model: claude-sonnet-4-6@20250514
  project_id: my-gcp-project
  region: us-central1
  # Uses Application Default Credentials
```

### AI Gateway

```yaml
llm:
  provider: ai-gateway
  model: claude-sonnet-4-6
  base_url: https://gateway.example.com/v1
  api_key: ${GATEWAY_API_KEY}
```

### Claude Code Session (Local)

```yaml
llm:
  provider: claude-code
  # Uses local Claude Code session via SDK
```

## Database Connections

### PostgreSQL

```yaml
databases:
  analytics:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    user: ${PG_USER}
    password: ${PG_PASSWORD}
```

### Snowflake

```yaml
databases:
  warehouse:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}
    role: ANALYST
```

### BigQuery

```yaml
databases:
  bigquery:
    type: bigquery
    project_id: my-gcp-project
    dataset: analytics
    credentials_path: ${GOOGLE_APPLICATION_CREDENTIALS}
```

### ClickHouse

```yaml
databases:
  events:
    type: clickhouse
    host: localhost
    port: 8123
    database: analytics
    user: ${CLICKHOUSE_USER}
    password: ${CLICKHOUSE_PASSWORD}
```

## Context Sources

### dbt

```yaml
context_sources:
  dbt_prod:
    type: dbt
    project_dir: ../dbt-project
    profiles_dir: ~/.dbt
    target: prod
    include_tests: true
    include_docs: true
```

### Looker

```yaml
context_sources:
  looker_main:
    type: looker
    base_url: https://company.looker.com
    client_id: ${LOOKER_CLIENT_ID}
    client_secret: ${LOOKER_CLIENT_SECRET}
    project: analytics
```

### Metabase

```yaml
context_sources:
  metabase:
    type: metabase
    base_url: https://metabase.company.com
    username: ${METABASE_USER}
    password: ${METABASE_PASSWORD}
    database_id: 1
```

### Notion

```yaml
context_sources:
  notion_wiki:
    type: notion
    api_key: ${NOTION_API_KEY}
    database_ids:
      - abc123def456
      - ghi789jkl012
```

## Semantic Layer Usage

### Search for Metrics

```typescript
// Via CLI
$ ktx sl "monthly recurring revenue"
```

Output structure:
```json
{
  "results": [
    {
      "type": "metric",
      "name": "mrr",
      "description": "Monthly Recurring Revenue",
      "calculation": "SUM(subscription_amount)",
      "dimensions": ["customer_id", "plan_type"],
      "filters": ["status = 'active'"],
      "source": "dbt_prod",
      "score": 0.95
    }
  ]
}
```

### Query Planning (Python API)

```python
# Install ktx semantic layer engine
# uv add ktx-sl

from ktx_sl import SemanticLayer, Query

# Load semantic layer from ktx project
sl = SemanticLayer.from_project("/path/to/ktx-project")

# Define query
query = Query(
    metrics=["mrr", "customer_count"],
    dimensions=["plan_type", "region"],
    filters=[{"field": "signup_date", "op": ">=", "value": "2024-01-01"}],
    order_by=[{"field": "mrr", "direction": "desc"}]
)

# Generate SQL
sql = sl.plan(query)
print(sql)
```

### Semantic Layer YAML Format

Example `semantic-layer/warehouse/subscriptions.yaml`:

```yaml
version: 1
source: warehouse
table: public.subscriptions

metrics:
  - name: mrr
    description: Monthly Recurring Revenue
    type: sum
    sql: amount
    filters:
      - status = 'active'
      - billing_period = 'monthly'
    
  - name: customer_count
    description: Distinct active customers
    type: count_distinct
    sql: customer_id
    filters:
      - status = 'active'

dimensions:
  - name: customer_id
    type: string
    sql: customer_id
    
  - name: plan_type
    type: string
    sql: plan_type
    
  - name: region
    type: string
    sql: customer_region

joins:
  - to: customers
    type: left
    on: subscriptions.customer_id = customers.id
```

## Wiki Management

### Add Wiki Content Manually

Create `wiki/global/refund-policy.md`:

```markdown
---
title: Refund Policy
tags: [finance, customer-success]
---

# Refund Policy

Customers can request refunds within 30 days of purchase.

## Calculation Rules

- Full refund: within 7 days
- Prorated refund: 8-30 days
- No refund: after 30 days

## Impact on Metrics

Refunds reduce `net_revenue` but not `gross_revenue`.
```

### Search Wiki from CLI

```bash
ktx wiki "refund policy"
ktx wiki "how to calculate churn"
```

### Wiki Structure

```
wiki/
├── global/                    # Shared team knowledge
│   ├── metrics-glossary.md
│   ├── refund-policy.md
│   └── data-quality-rules.md
└── user/alice/                # User-scoped notes
    └── analysis-notes.md
```

## MCP Integration

### Start MCP Server

```bash
# Auto-detect project
ktx mcp start

# Explicit project path
ktx mcp start --project-dir /path/to/analytics
```

### Claude Desktop Configuration

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "ktx-analytics": {
      "command": "ktx",
      "args": [
        "mcp",
        "start",
        "--project-dir",
        "/path/to/analytics"
      ]
    }
  }
}
```

### Codex Integration

```bash
# Install ktx skill in Codex project
npx skills add Kaelio/ktx --skill ktx

# Or via natural language
# "Run npx skills add Kaelio/ktx --skill ktx and configure ktx for this project"
```

### Available MCP Tools

Once MCP server is running, agents can use:

- `ktx_search_semantic_layer` — Search metrics and dimensions
- `ktx_search_wiki` — Search business knowledge
- `ktx_get_metric_definition` — Get detailed metric spec
- `ktx_list_connections` — List configured warehouses
- `ktx_validate_query` — Validate SQL against semantic layer

## Common Patterns

### Full Setup Workflow

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Create project
cd /path/to/analytics
ktx setup

# 3. Configure via interactive prompts:
#    - LLM: Anthropic API with claude-sonnet-4-6
#    - Embeddings: OpenAI text-embedding-3-small
#    - Database: PostgreSQL connection
#    - Sources: dbt project

# 4. Verify setup
ktx status

# 5. Build context
ktx ingest

# 6. Test search
ktx sl "revenue"
ktx wiki "metric definitions"

# 7. Start MCP for agents
ktx mcp start
```

### Incremental Updates

```bash
# Add new dbt models
cd ../dbt-project
dbt run --models new_model

# Ingest updates into ktx
cd ../analytics
ktx ingest --source dbt_prod

# Verify new metrics are available
ktx sl "new_metric"
```

### Multi-Warehouse Setup

```yaml
databases:
  production:
    type: postgres
    host: prod.db.company.com
    database: analytics
    schema: public
    user: ${PROD_DB_USER}
    password: ${PROD_DB_PASSWORD}
    
  events:
    type: clickhouse
    host: events.company.com
    database: analytics
    user: ${CLICKHOUSE_USER}
    password: ${CLICKHOUSE_PASSWORD}

context_sources:
  dbt_prod:
    type: dbt
    project_dir: ../dbt-project
    target: prod
    connection: production
    
  events_raw:
    type: raw
    connection: events
```

### CI/CD Integration

```yaml
# .github/workflows/ktx-ingest.yml
name: Update ktx Context
on:
  push:
    branches: [main]
    paths:
      - 'dbt-project/**'
      - 'wiki/**'

jobs:
  ingest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          
      - name: Install ktx
        run: npm install -g @kaelio/ktx
        
      - name: Configure secrets
        run: |
          mkdir -p .ktx
          echo "databases:" > .ktx/secrets.yaml
          echo "  warehouse:" >> .ktx/secrets.yaml
          echo "    user: ${{ secrets.DB_USER }}" >> .ktx/secrets.yaml
          echo "    password: ${{ secrets.DB_PASSWORD }}" >> .ktx/secrets.yaml
          
      - name: Ingest context
        run: ktx ingest --project-dir .
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          
      - name: Commit updates
        run: |
          git config user.name "ktx-bot"
          git config user.email "bot@company.com"
          git add semantic-layer/ raw-sources/
          git commit -m "Update ktx context" || exit 0
          git push
```

## Troubleshooting

### "No LLM provider configured"

**Solution:** Run `ktx setup` to configure LLM provider, or manually add to `ktx.yaml`:

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
```

Add API key to `.ktx/secrets.yaml`:

```yaml
llm:
  anthropic_api_key: ${ANTHROPIC_API_KEY}
```

### "Database connection failed"

**Check:**

1. Credentials in `.ktx/secrets.yaml` are correct
2. Network access to database host
3. Read-only user has SELECT permissions

Test connection:

```bash
# PostgreSQL example
psql -h localhost -U ${POSTGRES_USER} -d analytics -c "SELECT 1"
```

### "ktx context built: no"

**Solution:** Run ingestion:

```bash
ktx ingest --force
```

Check for errors in output. Common issues:
- Missing database permissions
- Invalid dbt profiles.yml
- Unreachable context source URLs

### "No results found" when searching

**Check:**

1. Ingestion completed successfully: `ktx status`
2. Semantic layer YAML files exist: `ls semantic-layer/`
3. Wiki content exists: `ls wiki/global/`

Re-run ingestion:

```bash
ktx ingest --source dbt_prod --force
```

### MCP Server Not Starting

**Check:**

1. No other process on port 3000: `lsof -i :3000`
2. Project directory exists and has `ktx.yaml`
3. Agent client configuration points to correct project path

Specify explicit project:

```bash
ktx mcp start --project-dir /full/path/to/project
```

### dbt Ingestion Fails

**Common issues:**

1. **profiles.yml not found:** Set `profiles_dir` in `ktx.yaml`
2. **Target not found:** Verify `target` name matches profiles.yml
3. **Compilation errors:** Run `dbt compile` manually to verify dbt project

Debug:

```bash
# Verify dbt configuration
cd ../dbt-project
dbt debug --profiles-dir ~/.dbt --target prod

# Check ktx can read dbt
ktx ingest --source dbt_prod --log-level debug
```

### Embedding Search Returns Poor Results

**Tune embedding quality:**

1. Use higher-quality embedding model:

```yaml
embeddings:
  provider: voyage
  model: voyage-3
```

2. Add more descriptive metadata to semantic layer YAML:

```yaml
metrics:
  - name: mrr
    description: Monthly Recurring Revenue from active subscriptions
    tags: [revenue, subscription, saas]
    business_context: |
      MRR is the primary metric for tracking subscription health.
      Includes only active, monthly-billed customers.
```

### Large Projects Performance

For projects with 100+ tables:

1. **Limit ingestion scope:**

```yaml
context_sources:
  dbt_prod:
    type: dbt
    include_patterns:
      - "marts/**"
      - "metrics/**"
    exclude_patterns:
      - "staging/**"
      - "tests/**"
```

2. **Increase sampling limits:**

```yaml
databases:
  warehouse:
    type: postgres
    sampling:
      max_tables: 500
      rows_per_table: 10000
```

3. **Run incremental ingestion:**

```bash
# Only ingest changed sources
ktx ingest --incremental
```

## Advanced Configuration

### Custom Sampling Strategy

```yaml
databases:
  warehouse:
    type: postgres
    sampling:
      enabled: true
      max_tables: 200
      rows_per_table: 5000
      include_patterns:
        - "public.*"
        - "analytics.*"
      exclude_patterns:
        - "*_tmp"
        - "*_staging"
```

### Join Detection Tuning

```yaml
project:
  join_detection:
    enabled: true
    min_confidence: 0.85
    max_cardinality_ratio: 0.1
    require_foreign_keys: false
```

### Wiki Ingestion from Git

```yaml
context_sources:
  team_wiki:
    type: git
    repo_url: https://github.com/company/analytics-wiki
    branch: main
    path: docs/
    auth_token: ${GITHUB_TOKEN}
```

## Project Resolution

ktx resolves project directory in this order:

1. `--project-dir` flag
2. `KTX_PROJECT_DIR` environment variable
3. Nearest `ktx.yaml` in current or parent directories
4. Current working directory

For scripting, always use explicit path:

```bash
ktx ingest --project-dir /path/to/project
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `KTX_PROJECT_DIR` | Default project directory |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `OPENAI_API_KEY` | OpenAI API key |
| `POSTGRES_USER` | PostgreSQL username |
| `POSTGRES_PASSWORD` | PostgreSQL password |
| `SNOWFLAKE_USER` | Snowflake username |
| `SNOWFLAKE_PASSWORD` | Snowflake password |
| `GOOGLE_APPLICATION_CREDENTIALS` | GCP service account key path |
| `GITHUB_TOKEN` | GitHub personal access token |

Store in `.env` (git-ignored) or `.ktx/secrets.yaml`.

## CLI Reference Summary

| Command | Purpose |
|---------|---------|
| `ktx setup` | Create/resume project, configure providers |
| `ktx status` | Check project readiness |
| `ktx ingest` | Build context from configured sources |
| `ktx sl <query>` | Search semantic layer |
| `ktx wiki <query>` | Search wiki knowledge |
| `ktx mcp start` | Start MCP server for agents |
| `ktx config get` | Show current configuration |
| `ktx config set` | Update configuration |
| `ktx connections list` | List database connections |
| `ktx sources list` | List context sources |

Full reference: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
