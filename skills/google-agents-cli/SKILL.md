---
name: google-agents-cli
description: CLI and skills for building, evaluating, and deploying AI agents on Google Cloud's Gemini Enterprise Agent Platform using ADK
triggers:
  - "create a new ADK agent project"
  - "deploy my agent to Google Cloud"
  - "run evaluations on my agent"
  - "scaffold an agents-cli project"
  - "publish my agent to Gemini Enterprise"
  - "set up agent observability and monitoring"
  - "build an agent with ADK"
  - "add CI/CD to my agent project"
---

# google-agents-cli

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

`agents-cli` is the official CLI and skill suite for building, evaluating, and deploying production-grade AI agents on Google Cloud's Gemini Enterprise Agent Platform. It provides commands and coding agent skills that streamline the entire agent development lifecycle — from scaffolding to deployment to observability.

## What It Does

- **Scaffold** agent projects with best-practice structure using ADK (Agent Development Kit)
- **Evaluate** agents with metrics, evalsets, LLM-as-judge, and trajectory scoring
- **Deploy** to Google Cloud (Agent Runtime, Cloud Run, GKE) with CI/CD
- **Publish** agents to Gemini Enterprise for organization-wide access
- **Monitor** with Cloud Trace, logging, and third-party observability integrations
- **Enhance** existing projects with deployment configs, RAG, and CI/CD pipelines

## Installation

### Prerequisites

- Python 3.11+
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- [Node.js](https://nodejs.org/en/download) (for skills installation)
- Google Cloud account (for deployment) or [AI Studio API key](https://aistudio.google.com/apikey) (for local development)

### Install CLI and Skills

```bash
uvx google-agents-cli setup
```

This installs the CLI globally and adds skills to your coding agents (Gemini CLI, Claude Code, Cursor, etc.).

### Install Just the Skills

```bash
npx skills add google/agents-cli
```

Your coding agent will handle the rest.

## Authentication

### Authenticate with Google Cloud

```bash
agents-cli login
```

### Use AI Studio for Local Development

Set your API key:

```bash
export GOOGLE_API_KEY=your-api-key
```

### Check Authentication Status

```bash
agents-cli login --status
```

## Key Commands

### Project Scaffolding

#### Create a New Agent Project

```bash
agents-cli scaffold my-agent
cd my-agent
```

This creates a complete ADK agent project with:
- `agent.py` - Main agent definition
- `pyproject.toml` - Dependencies
- `eval/` - Evaluation configuration
- `tests/` - Unit tests
- `.github/workflows/` - CI/CD (optional)

#### Enhance Existing Project

Add deployment, CI/CD, or RAG to an existing agent:

```bash
agents-cli scaffold enhance
```

Choose from:
- Cloud Run deployment
- GKE deployment
- CI/CD pipeline (staging + prod)
- RAG (Retrieval-Augmented Generation)

#### Upgrade Project

Upgrade to the latest agents-cli version:

```bash
agents-cli scaffold upgrade
```

### Development

#### Install Dependencies

```bash
agents-cli install
```

This uses `uv` to install Python dependencies from `pyproject.toml`.

#### Run Agent Locally

```bash
agents-cli run "What's the weather in Tokyo?"
```

Single-turn execution with your agent.

#### Code Quality

```bash
agents-cli lint
```

Runs Ruff for linting and formatting checks.

### Evaluation

#### Run Evaluations

```bash
agents-cli eval run
```

Runs evaluations defined in `eval/evalset.yaml` against your agent.

#### Compare Evaluation Results

```bash
agents-cli eval compare results-v1.json results-v2.json
```

Compare two evaluation runs to see performance deltas.

### Deployment

#### Deploy to Google Cloud

```bash
agents-cli deploy
```

Deploys to the configured target (Agent Runtime, Cloud Run, or GKE).

#### Provision Infrastructure

Single-project setup:

```bash
agents-cli infra single-project
```

Multi-environment CI/CD (staging + prod):

```bash
agents-cli infra cicd
```

### Publish

#### Register with Gemini Enterprise

```bash
agents-cli publish gemini-enterprise
```

Makes your agent available to your organization through Gemini Enterprise.

### Data & RAG

#### Provision Datastore

```bash
agents-cli infra datastore
```

Sets up vector stores and databases for RAG.

#### Run Data Ingestion

```bash
agents-cli data-ingestion
```

Ingests documents into your RAG datastore.

### Utilities

#### Project Info

```bash
agents-cli info
```

Shows project configuration and CLI version.

#### Update Skills

Force reinstall skills to all coding agents:

```bash
agents-cli update
```

## ADK Agent Code Patterns

### Basic Agent Structure

```python
# agent.py
from adk.agents import Agent
from adk.tools import Tool

def search_tool(query: str) -> str:
    """Search for information."""
    # Implementation
    return f"Results for: {query}"

agent = Agent(
    name="my-agent",
    model="gemini-2.0-flash",
    description="A helpful assistant",
    tools=[Tool(search_tool)],
    instructions="""You are a helpful assistant.
    Use the search tool to find information when needed."""
)

if __name__ == "__main__":
    result = agent.run("What is ADK?")
    print(result.content)
```

### Agent with State

```python
from adk.agents import Agent
from adk.state import State
from typing import TypedDict

class ConversationState(TypedDict):
    user_name: str
    message_count: int

def increment_counter(state: State[ConversationState]) -> None:
    """Track message count."""
    state.data["message_count"] = state.data.get("message_count", 0) + 1

agent = Agent(
    name="stateful-agent",
    model="gemini-2.0-flash",
    state_schema=ConversationState,
    instructions="Track conversation history and personalize responses."
)

# Run with state
initial_state = {"user_name": "Alice", "message_count": 0}
result = agent.run("Hello!", state=initial_state)
```

### Multi-Agent Orchestration

```python
from adk.agents import Agent
from adk.orchestration import SequentialOrchestrator

researcher = Agent(
    name="researcher",
    model="gemini-2.0-flash",
    instructions="Research the topic thoroughly."
)

writer = Agent(
    name="writer",
    model="gemini-2.0-flash",
    instructions="Write a comprehensive article based on research."
)

orchestrator = SequentialOrchestrator(agents=[researcher, writer])
result = orchestrator.run("Write an article about quantum computing")
```

### Custom Tools

```python
from adk.tools import Tool
from adk.agents import Agent

def calculate_tax(amount: float, rate: float = 0.20) -> dict:
    """
    Calculate tax on an amount.
    
    Args:
        amount: The base amount
        rate: Tax rate (default 0.20 for 20%)
    
    Returns:
        Dictionary with tax and total
    """
    tax = amount * rate
    return {
        "base": amount,
        "tax": tax,
        "total": amount + tax
    }

agent = Agent(
    name="tax-calculator",
    model="gemini-2.0-flash",
    tools=[Tool(calculate_tax)],
    instructions="Help users calculate taxes."
)
```

### Callbacks for Observability

```python
from adk.agents import Agent
from adk.callbacks import Callback

class LoggingCallback(Callback):
    def on_tool_start(self, tool_name: str, inputs: dict) -> None:
        print(f"🔧 Starting tool: {tool_name}")
        print(f"   Inputs: {inputs}")
    
    def on_tool_end(self, tool_name: str, outputs: dict) -> None:
        print(f"✅ Tool completed: {tool_name}")
        print(f"   Outputs: {outputs}")
    
    def on_error(self, error: Exception) -> None:
        print(f"❌ Error: {error}")

agent = Agent(
    name="monitored-agent",
    model="gemini-2.0-flash",
    callbacks=[LoggingCallback()]
)
```

## Configuration

### Project Configuration

`pyproject.toml` includes agents-cli settings:

```toml
[tool.agents-cli]
agent_module = "agent:agent"  # Path to agent instance
deployment_target = "cloud-run"  # cloud-run, gke, agent-runtime
region = "us-central1"
project_id = "my-gcp-project"

[tool.agents-cli.eval]
evalset_path = "eval/evalset.yaml"
metrics = ["accuracy", "latency", "cost"]
```

### Evaluation Configuration

`eval/evalset.yaml`:

```yaml
version: "1.0"
evalset:
  - input: "What is the capital of France?"
    expected_output: "Paris"
    metadata:
      category: "geography"
      difficulty: "easy"
  
  - input: "Explain quantum entanglement"
    evaluator: "llm-as-judge"
    criteria:
      - accuracy
      - clarity
      - completeness
    metadata:
      category: "science"
      difficulty: "hard"
```

### Environment Variables

```bash
# Required for deployment
export GOOGLE_CLOUD_PROJECT=my-project-id
export GOOGLE_CLOUD_REGION=us-central1

# For local development with AI Studio
export GOOGLE_API_KEY=your-api-key

# Optional: Custom model
export ADK_MODEL=gemini-2.0-flash

# Optional: Observability
export GOOGLE_CLOUD_TRACE_ENABLED=true
```

## Deployment Patterns

### Cloud Run Deployment

Automatically configured when you scaffold with Cloud Run:

```yaml
# .agents-cli/deploy.yaml
target: cloud-run
service_name: my-agent
region: us-central1
min_instances: 0
max_instances: 10
memory: 512Mi
cpu: 1
```

Deploy:

```bash
agents-cli deploy
```

### GKE Deployment

For high-scale, production workloads:

```yaml
# .agents-cli/deploy.yaml
target: gke
cluster_name: agents-cluster
namespace: production
replicas: 3
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
```

### Agent Runtime Deployment

Managed runtime for ADK agents:

```yaml
# .agents-cli/deploy.yaml
target: agent-runtime
agent_id: my-agent
version: v1
scaling:
  min_replicas: 1
  max_replicas: 10
```

### CI/CD Pipeline

Generated `.github/workflows/deploy.yaml`:

```yaml
name: Deploy Agent
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}
      - name: Deploy
        run: |
          uvx google-agents-cli deploy
```

Set required secrets in GitHub:
- `GCP_SA_KEY`: Service account JSON key with deployment permissions

## Common Patterns

### Multi-Turn Conversations

```python
from adk.agents import Agent

agent = Agent(
    name="conversational-agent",
    model="gemini-2.0-flash",
    instructions="Maintain context across conversation turns."
)

# Multi-turn interaction
session_id = "user-123"
messages = [
    "My name is Alice",
    "What's my name?",
    "What did I just tell you?"
]

for msg in messages:
    result = agent.run(msg, session_id=session_id)
    print(f"User: {msg}")
    print(f"Agent: {result.content}\n")
```

### Structured Output

```python
from adk.agents import Agent
from pydantic import BaseModel

class MovieRecommendation(BaseModel):
    title: str
    year: int
    genre: str
    reason: str

agent = Agent(
    name="movie-recommender",
    model="gemini-2.0-flash",
    output_schema=MovieRecommendation,
    instructions="Recommend movies based on user preferences."
)

result = agent.run("Recommend a sci-fi movie from the 1980s")
recommendation = result.structured_output
print(f"{recommendation.title} ({recommendation.year})")
```

### RAG Integration

```python
from adk.agents import Agent
from adk.tools import Tool
from google.cloud import aiplatform

def search_knowledge_base(query: str) -> list[str]:
    """Search vector database for relevant documents."""
    # Initialize vector search
    index = aiplatform.MatchingEngineIndex("projects/.../indexes/...")
    results = index.find_neighbors(query, num_neighbors=5)
    return [doc.content for doc in results]

agent = Agent(
    name="rag-agent",
    model="gemini-2.0-flash",
    tools=[Tool(search_knowledge_base)],
    instructions="""Use the knowledge base to answer questions.
    Always cite sources when providing information."""
)
```

### Error Handling

```python
from adk.agents import Agent
from adk.exceptions import ToolExecutionError, ModelError

agent = Agent(name="robust-agent", model="gemini-2.0-flash")

try:
    result = agent.run("Complex query")
except ToolExecutionError as e:
    print(f"Tool failed: {e.tool_name} - {e.message}")
    # Fallback logic
except ModelError as e:
    print(f"Model error: {e.message}")
    # Retry or use backup model
except Exception as e:
    print(f"Unexpected error: {e}")
```

## Evaluation Strategies

### Custom Metrics

```python
# eval/custom_metrics.py
from adk.eval import Metric

class BusinessLogicMetric(Metric):
    def evaluate(self, output: str, expected: str, metadata: dict) -> float:
        """Custom scoring logic."""
        # Check if output contains required business terms
        required_terms = metadata.get("required_terms", [])
        score = sum(term.lower() in output.lower() for term in required_terms)
        return score / len(required_terms) if required_terms else 1.0
```

Reference in evalset:

```yaml
# eval/evalset.yaml
- input: "Explain our refund policy"
  evaluator: "custom:BusinessLogicMetric"
  metadata:
    required_terms: ["30 days", "receipt", "original condition"]
```

### LLM-as-Judge

```yaml
# eval/evalset.yaml
- input: "Write a haiku about coding"
  evaluator: "llm-as-judge"
  judge_model: "gemini-2.0-flash"
  criteria:
    - name: "structure"
      description: "Has 5-7-5 syllable structure"
      weight: 0.5
    - name: "creativity"
      description: "Original and creative"
      weight: 0.3
    - name: "relevance"
      description: "Related to coding"
      weight: 0.2
```

## Troubleshooting

### Authentication Issues

**Problem:** `agents-cli login` fails

**Solution:**
```bash
# Clear existing credentials
gcloud auth application-default revoke
gcloud auth revoke

# Re-authenticate
gcloud auth login
gcloud auth application-default login

# Verify
agents-cli login --status
```

### Deployment Failures

**Problem:** Deploy fails with permission errors

**Solution:**
```bash
# Ensure service account has required roles
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SA_EMAIL" \
  --role="roles/run.admin"

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SA_EMAIL" \
  --role="roles/iam.serviceAccountUser"
```

### Agent Import Errors

**Problem:** `ModuleNotFoundError` when running agent

**Solution:**
```bash
# Reinstall dependencies
agents-cli install

# Verify agent module path in pyproject.toml
# [tool.agents-cli]
# agent_module = "agent:agent"  # Should match your file structure
```

### Evaluation Not Running

**Problem:** `agents-cli eval run` finds no test cases

**Solution:**
```bash
# Verify evalset path
cat pyproject.toml | grep evalset_path

# Check evalset format
cat eval/evalset.yaml

# Ensure YAML is valid
python -c "import yaml; yaml.safe_load(open('eval/evalset.yaml'))"
```

### Model Rate Limits

**Problem:** `429 Too Many Requests` errors

**Solution:**
```python
# Add retry logic with exponential backoff
from adk.agents import Agent

agent = Agent(
    name="rate-limited-agent",
    model="gemini-2.0-flash",
    retry_config={
        "max_retries": 5,
        "initial_delay": 1.0,
        "exponential_base": 2.0
    }
)
```

### Local Development with AI Studio

**Problem:** Want to develop without Google Cloud project

**Solution:**
```bash
# Get API key from https://aistudio.google.com/apikey
export GOOGLE_API_KEY=your-api-key

# Create agent without deployment config
agents-cli scaffold my-agent --no-deploy

# Run locally
agents-cli run "test query"
```

### Observability Not Working

**Problem:** No traces appearing in Cloud Trace

**Solution:**
```bash
# Enable Cloud Trace API
gcloud services enable cloudtrace.googleapis.com

# Verify environment variable
export GOOGLE_CLOUD_TRACE_ENABLED=true

# Add explicit callback
python -c "
from adk.agents import Agent
from adk.callbacks import CloudTraceCallback

agent = Agent(
    name='traced-agent',
    model='gemini-2.0-flash',
    callbacks=[CloudTraceCallback()]
)
"
```

## Additional Resources

- [Official Documentation](https://google.github.io/agents-cli/)
- [ADK Documentation](https://adk.dev)
- [Release Notes](https://github.com/google/agents-cli/blob/main/RELEASE_NOTES.md)
- [GitHub Issues](https://github.com/google/agents-cli/issues)
- [Google Cloud Agent Platform](https://cloud.google.com/gemini-enterprise-agent-platform)
