---
name: ktx-context-layer-data-agents
description: Teach AI agents how to query data warehouses accurately using ktx - an executable context layer with skills, memory, and a semantic layer
triggers:
  - setup ktx for data warehouse queries
  - configure ktx context layer
  - build ktx semantic layer from database
  - integrate ktx with claude code
  - query warehouse using ktx
  - ingest dbt metrics into ktx
  - search ktx wiki or semantic layer
  - troubleshoot ktx mcp server
---

# ktx Context Layer for Data Agents

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

ktx is an executable context layer that teaches AI agents how to query data warehouses accurately. It automatically builds a semantic layer from your database, ingests business knowledge from wikis and tools like dbt/Looker, detects joinable columns, resolves fan/chasm traps, and exposes everything through CLI and MCP tools for agent execution.

## Installation

Install ktx globally via npm:

```bash
npm install -g @kaelio/ktx
```

Or add to a project:

```bash
npm install --save-dev @kaelio/ktx
```

## Quick Setup

Run the interactive setup wizard:

```bash
ktx setup
```

This will:
- Create or resume a ktx project in the current directory
- Configure LLM provider (Anthropic API, Google Vertex AI, or Claude Agent SDK)
- Configure embedding provider (OpenAI, Anthropic, Vertex AI)
- Set up database connections (PostgreSQL, Snowflake, BigQuery, etc.)
- Configure context sources (dbt, LookML, Looker, Metabase, Notion)
- Build initial context
- Install agent integration (Codex, Claude Code, etc.)

Check project status:

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

ktx creates this structure:

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local.

## Configuration

### ktx.yaml

Example project configuration:

```yaml
version: 1
project_id: analytics_project
llm_provider: anthropic
embedding_provider: openai

connections:
  - id: warehouse
    type: postgres
    config:
      host: localhost
      port: 5432
      database: analytics
      user: readonly_user
      password_env: POSTGRES_PASSWORD
      ssl: false

context_sources:
  - id: dbt_main
    type: dbt
    config:
      manifest_path: ./dbt/target/manifest.json
      catalog_path: ./dbt/target/catalog.json
  
  - id: looker_metrics
    type: looker
    config:
      base_url_env: LOOKER_BASE_URL
      client_id_env: LOOKER_CLIENT_ID
      client_secret_env: LOOKER_CLIENT_SECRET

agent_integrations:
  - type: codex
    scope: project
```

### Environment Variables

Store secrets in environment variables:

```bash
# LLM Provider
export ANTHROPIC_API_KEY=sk-ant-...
# or for Vertex AI
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json

# Embedding Provider
export OPENAI_API_KEY=sk-...

# Database Credentials
export POSTGRES_PASSWORD=yourpassword
export SNOWFLAKE_PASSWORD=yourpassword

# Tool Integrations
export LOOKER_BASE_URL=https://company.looker.com
export LOOKER_CLIENT_ID=your_client_id
export LOOKER_CLIENT_SECRET=your_secret
export NOTION_TOKEN=secret_...
```

## Core Commands

### Build Context

Ingest from all configured connections and sources:

```bash
ktx ingest
```

Ingest from specific connection:

```bash
ktx ingest --connection warehouse
```

Ingest from specific context source:

```bash
ktx ingest --context-source dbt_main
```

### Search Semantic Layer

Search for metrics, dimensions, and entities:

```bash
ktx sl "revenue"
ktx sl "customer lifetime value"
ktx sl "monthly active users"
```

Example output:

```text
Found 3 semantic sources matching "revenue":

1. metric.monthly_recurring_revenue
   Type: metric
   Connection: warehouse
   Description: Sum of all active subscription values in a given month
   SQL: SUM(subscriptions.monthly_value)
   
2. dimension.revenue_tier
   Type: dimension
   Entity: customer
   Description: Customer revenue bracket (low/medium/high)
   
3. entity.revenue_events
   Type: entity
   Table: prod.revenue_events
   Primary key: event_id
```

### Search Wiki

Search business knowledge and documentation:

```bash
ktx wiki "refund policy"
ktx wiki "customer segmentation"
```

### Query Warehouse

Execute SQL queries through ktx:

```bash
ktx query "SELECT customer_tier, COUNT(*) FROM customers GROUP BY customer_tier"
```

Query with metric resolution:

```bash
ktx query --use-metrics "SELECT monthly_recurring_revenue FROM time WHERE month = '2024-01'"
```

### MCP Server

Start the Model Context Protocol server for agent integration:

```bash
ktx mcp start
```

Specify project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

The MCP server exposes these tools to agents:
- `ktx_search_semantic_layer` - Search metrics and dimensions
- `ktx_search_wiki` - Search business knowledge
- `ktx_query` - Execute SQL queries
- `ktx_get_schema` - Retrieve table schemas
- `ktx_get_metric_definition` - Get canonical metric SQL

## Working with Semantic Sources

ktx automatically generates semantic sources during ingestion. You can also define them manually.

### Metric Definition

```yaml
# semantic-layer/warehouse/metrics/mrr.yaml
name: monthly_recurring_revenue
type: metric
description: Sum of all active subscription values in a given month
entity: subscription
sql: SUM(subscriptions.monthly_value)
filters:
  - sql: subscriptions.status = 'active'
dimensions:
  - customer_tier
  - plan_type
time_dimension: subscription_start_date
```

### Entity Definition

```yaml
# semantic-layer/warehouse/entities/customer.yaml
name: customer
type: entity
table: prod.customers
primary_key: customer_id
description: Customer master table
dimensions:
  - name: customer_tier
    type: categorical
    sql: tier
  - name: signup_date
    type: time
    sql: created_at
```

### Join Configuration

ktx auto-detects joins, but you can override:

```yaml
# semantic-layer/warehouse/joins/customer_subscription.yaml
from_entity: customer
to_entity: subscription
type: one_to_many
join_sql: customers.customer_id = subscriptions.customer_id
```

## Common Patterns

### Pattern 1: Setup New Project

```bash
# Navigate to analytics directory
cd ~/analytics

# Initialize ktx
ktx setup

# Follow prompts to configure:
# - LLM provider (Anthropic API recommended)
# - Embedding provider (OpenAI recommended)
# - Database connection (read-only credentials)
# - dbt integration (point to manifest.json)

# Verify setup
ktx status

# Build initial context
ktx ingest
```

### Pattern 2: Query with Agent Context

From Claude Code, Cursor, Codex, or OpenCode:

```text
User: What was our MRR in January 2024?

Agent uses ktx:
1. ktx sl "monthly recurring revenue" → finds metric definition
2. ktx query --use-metrics "SELECT monthly_recurring_revenue FROM time WHERE month = '2024-01'"
3. Returns accurate result using canonical metric logic
```

### Pattern 3: Add Business Context

Create wiki pages for business knowledge:

```bash
# Create global wiki page
mkdir -p wiki/global
cat > wiki/global/refund-policy.md << 'EOF'
# Refund Policy

Customers can request refunds within 30 days of purchase.

## Refund Eligibility
- Full refund if < 7 days
- Prorated refund if 7-30 days
- No refund if > 30 days

## Accounting Treatment
Refunds are recorded as negative revenue in the month issued, not the original purchase month.
EOF

# Ingest wiki content
ktx ingest
```

### Pattern 4: Integrate dbt Metrics

```bash
# Ensure dbt artifacts exist
cd dbt-project
dbt compile
dbt docs generate

# Configure in ktx.yaml
cat >> ktx.yaml << 'EOF'
context_sources:
  - id: dbt_main
    type: dbt
    config:
      manifest_path: ./dbt-project/target/manifest.json
      catalog_path: ./dbt-project/target/catalog.json
EOF

# Ingest dbt metrics
ktx ingest --context-source dbt_main

# Search for dbt metrics
ktx sl "customers"
```

### Pattern 5: Agent Integration

For Codex:

```bash
# Install during setup or manually
ktx setup
# Select "codex" when prompted for agent integration

# Or install explicitly
npx skills add Kaelio/ktx --skill ktx
```

For Claude Code:

```bash
# ktx setup handles this automatically
# Adds MCP server config to Claude Code settings
```

Manual MCP configuration for Claude Desktop:

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

## TypeScript API (Programmatic Usage)

While ktx is primarily a CLI tool, you can use it programmatically:

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load existing project
const project = await KtxProject.load('/path/to/project');

// Search semantic layer
const results = await project.searchSemanticLayer('revenue');
console.log(results);

// Search wiki
const wikiResults = await project.searchWiki('refund policy');
console.log(wikiResults);

// Execute query
const queryResult = await project.query(
  'warehouse',
  'SELECT * FROM customers LIMIT 10'
);
console.log(queryResult.rows);

// Get metric definition
const metric = await project.getMetric('monthly_recurring_revenue');
console.log(metric.sql);
```

## Troubleshooting

### MCP Server Not Starting

**Symptom**: Agent can't connect to ktx

**Solution**:

```bash
# Check status
ktx status

# Start MCP server manually
ktx mcp start --project-dir /path/to/project

# Check logs
tail -f ~/.ktx/logs/mcp.log
```

### Ingestion Failures

**Symptom**: `ktx ingest` fails with connection errors

**Solution**:

```bash
# Test database connection
ktx test-connection warehouse

# Verify environment variables
env | grep -E 'POSTGRES|SNOWFLAKE|ANTHROPIC|OPENAI'

# Check ktx.yaml syntax
ktx validate

# Ingest with verbose logging
ktx ingest --verbose
```

### Missing Metrics

**Symptom**: `ktx sl "metric_name"` returns no results

**Solution**:

```bash
# Re-ingest context
ktx ingest --force

# Check semantic layer directory
ls -la semantic-layer/warehouse/metrics/

# Manually create metric if needed
mkdir -p semantic-layer/warehouse/metrics
cat > semantic-layer/warehouse/metrics/my_metric.yaml << 'EOF'
name: my_metric
type: metric
description: My custom metric
entity: my_entity
sql: COUNT(*)
EOF

# Rebuild context
ktx ingest
```

### LLM Provider Issues

**Symptom**: Context building fails with API errors

**Solution**:

```bash
# Verify API key
echo $ANTHROPIC_API_KEY

# Test LLM connection
ktx test-llm

# Switch provider if needed
ktx setup
# Select different LLM provider

# Check rate limits and quota
# Anthropic: https://console.anthropic.com
# OpenAI: https://platform.openai.com/usage
```

### Permission Errors

**Symptom**: Can't write to project directory

**Solution**:

```bash
# Check directory permissions
ls -la .

# Fix ownership
sudo chown -R $USER:$USER .

# Or specify writable project directory
ktx setup --project-dir ~/my-ktx-project
```

### Semantic Layer Contradictions

**Symptom**: ktx flags conflicting metric definitions

**Solution**:

```bash
# Review contradictions report
cat raw-sources/warehouse/contradictions.json

# Resolve by editing semantic sources
vim semantic-layer/warehouse/metrics/revenue.yaml

# Or update source (dbt, Looker, etc.)
# Then re-ingest
ktx ingest --force
```

## Advanced Configuration

### Custom Sampling Strategy

Control how ktx samples tables during ingestion:

```yaml
# ktx.yaml
connections:
  - id: warehouse
    type: postgres
    config:
      host: localhost
      database: analytics
      sample_strategy: adaptive
      max_sample_rows: 10000
      min_sample_rows: 100
```

### Join Detection Tuning

Adjust automatic join detection:

```yaml
# ktx.yaml
semantic_layer:
  join_detection:
    min_confidence: 0.8
    sample_size: 1000
    detect_fan_traps: true
    detect_chasm_traps: true
```

### Wiki Organization

Structure wiki for better retrieval:

```bash
wiki/
├── global/
│   ├── metrics/
│   │   ├── revenue-definitions.md
│   │   └── user-engagement.md
│   ├── policies/
│   │   ├── data-retention.md
│   │   └── refund-policy.md
│   └── glossary/
│       └── business-terms.md
└── user/
    └── <user-id>/
        └── scratch.md
```

## Best Practices

1. **Use read-only database credentials** - ktx never writes, but enforce it at the DB level
2. **Commit semantic layer and wiki** - Share context across team
3. **Keep .ktx/ local** - Contains secrets and local state
4. **Re-ingest after schema changes** - `ktx ingest` after dbt runs or migrations
5. **Document metrics in wiki** - Add business context beyond SQL definitions
6. **Use environment variables for secrets** - Never commit credentials
7. **Test queries before agents use them** - `ktx query` validates SQL
8. **Review contradiction reports** - Resolve conflicting definitions promptly

## Project Resolution

ktx finds projects in this order:

1. `--project-dir` flag
2. `KTX_PROJECT_DIR` environment variable
3. Nearest `ktx.yaml` in parent directories
4. Current working directory

For scripting, always use explicit project dir:

```bash
ktx ingest --project-dir /opt/analytics
ktx mcp start --project-dir /opt/analytics
```

---

**Documentation**: https://docs.kaelio.com/ktx  
**GitHub**: https://github.com/Kaelio/ktx  
**Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
