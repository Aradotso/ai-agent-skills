---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres task tracking for agentic workflows
triggers:
  - set up the agentic research agent service
  - create a research workflow with planning agents
  - use tavily arxiv and wikipedia research tools
  - build a multi-step agent research system
  - implement reflective research agent with fastapi
  - configure postgres backed agent task tracking
  - run research agents with tool calling
  - create agentic workflow research reports
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic Research Agent is a FastAPI-based service that orchestrates multi-step research workflows using tool-using agents. It includes a planner agent that breaks down research tasks, executor agents that perform research (via Tavily web search, arXiv papers, Wikipedia), writing, and editing. All task state and results are persisted in Postgres, enabling live progress tracking and final report retrieval.

**Key Features:**
- Planning agent that generates multi-step workflows
- Research tools: Tavily search, arXiv search, Wikipedia lookup
- Writer and editor agents for report generation
- Postgres-backed task state management
- Real-time progress tracking via API endpoints
- Web UI for task submission and monitoring
- Single Docker container deployment (Postgres + FastAPI)

## Installation

### Prerequisites

1. **Docker** installed and running
2. **API Keys** in a `.env` file in the project root:

```env
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
```

### Quick Start

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Create .env file with your API keys
cat > .env << EOF
OPENAI_API_KEY=${OPENAI_API_KEY}
TAVILY_API_KEY=${TAVILY_API_KEY}
EOF

# Build the Docker image
docker build -t agentic-research-agent .

# Run the container
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name research-agent \
  --env-file .env \
  agentic-research-agent
```

The service will be available at:
- **Web UI:** http://localhost:8000/
- **API Docs:** http://localhost:8000/docs
- **Postgres:** localhost:5432

## API Reference

### Generate Research Report

**POST** `/generate_report`

Initiates a research task with multi-step agent workflow.

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Latest advances in quantum computing for drug discovery",
    "model": "openai:gpt-4o"
  }'
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Check Task Progress

**GET** `/task_progress/{task_id}`

Returns live status of each step and substep in the workflow.

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "in_progress",
  "current_step": "research",
  "steps": [
    {"name": "planning", "status": "completed"},
    {"name": "research", "status": "in_progress"},
    {"name": "writing", "status": "pending"}
  ]
}
```

### Get Final Report

**GET** `/task_status/{task_id}`

Retrieves final status and completed research report.

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report\n\n## Introduction...",
  "metadata": {
    "started_at": "2025-01-15T10:30:00Z",
    "completed_at": "2025-01-15T10:35:23Z"
  }
}
```

## Core Components

### Planning Agent

Located in `src/planning_agent.py`, the planner creates a structured workflow from a user prompt.

```python
from src.planning_agent import planner_agent, executor_agent_step

# Generate a research plan
plan = planner_agent(
    prompt="Analyze the impact of transformer models on NLP",
    model="openai:gpt-4o"
)

# Execute each step in the plan
for step in plan["steps"]:
    result = executor_agent_step(
        step=step,
        context={},
        model="openai:gpt-4o"
    )
    print(f"Step {step['name']}: {result['status']}")
```

### Research Tools

Located in `src/research_tools.py`, provides three main research capabilities:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Web search via Tavily
web_results = tavily_search_tool(
    query="recent breakthroughs in protein folding",
    max_results=5
)

# Academic papers via arXiv
papers = arxiv_search_tool(
    query="attention mechanisms neural networks",
    max_results=10
)

# Encyclopedia lookup via Wikipedia
wiki_content = wikipedia_search_tool(
    query="transformer architecture",
    sentences=5
)
```

### Agent Implementations

Located in `src/agents.py`:

```python
from src.agents import research_agent, writer_agent, editor_agent

# Research phase
research_results = research_agent(
    topic="carbon capture technologies",
    tools=["tavily", "arxiv", "wikipedia"],
    model="openai:gpt-4o"
)

# Writing phase
draft = writer_agent(
    research_data=research_results,
    outline=plan["outline"],
    model="openai:gpt-4o"
)

# Editing phase
final_report = editor_agent(
    draft=draft,
    criteria=["clarity", "accuracy", "structure"],
    model="openai:gpt-4o"
)
```

## Database Schema

The service uses Postgres to track task state. Key tables:

```sql
-- Tasks table
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    prompt TEXT NOT NULL,
    model VARCHAR(100),
    status VARCHAR(50) DEFAULT 'pending',
    report TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Task steps table
CREATE TABLE task_steps (
    id SERIAL PRIMARY KEY,
    task_id UUID REFERENCES tasks(id),
    step_name VARCHAR(100),
    step_order INTEGER,
    status VARCHAR(50) DEFAULT 'pending',
    result JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);
```

Connect to the database from your host:

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key for LLM calls
TAVILY_API_KEY=tvly-...            # Tavily API key for web search

# Optional - Database (defaults provided by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0              # Set to 1 to drop/recreate tables
```

### Custom Models

The service uses `aisuite` for model abstraction. Supported formats:

```python
# OpenAI models
model = "openai:gpt-4o"
model = "openai:gpt-4-turbo"

# Anthropic models (if configured)
model = "anthropic:claude-3-opus"

# Google models (if configured)
model = "google:gemini-pro"
```

## Development Patterns

### Adding Custom Research Tools

Create a new tool in `src/research_tools.py`:

```python
import requests
import os

def custom_search_tool(query: str, **kwargs) -> dict:
    """
    Custom search tool implementation.
    """
    api_key = os.getenv("CUSTOM_API_KEY")
    response = requests.get(
        "https://api.custom-search.com/search",
        params={"q": query, "key": api_key},
        timeout=10
    )
    
    if response.status_code == 200:
        return {
            "status": "success",
            "results": response.json().get("items", [])
        }
    else:
        return {
            "status": "error",
            "message": f"API error: {response.status_code}"
        }
```

Register the tool in `src/agents.py`:

```python
AVAILABLE_TOOLS = {
    "tavily": tavily_search_tool,
    "arxiv": arxiv_search_tool,
    "wikipedia": wikipedia_search_tool,
    "custom": custom_search_tool,  # Add your tool
}
```

### Creating Custom Agents

Extend the agent system in `src/agents.py`:

```python
def fact_checker_agent(content: str, sources: list, model: str) -> dict:
    """
    Verify claims against source materials.
    """
    import aisuite as ai
    
    client = ai.Client()
    
    prompt = f"""
    Verify the following content against the provided sources:
    
    Content:
    {content}
    
    Sources:
    {sources}
    
    Identify any claims that are not supported by the sources.
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return {
        "verified": True,
        "corrections": response.choices[0].message.content
    }
```

### Extending the Workflow

Modify `src/planning_agent.py` to add custom steps:

```python
def executor_agent_step(step: dict, context: dict, model: str) -> dict:
    """
    Execute a single workflow step.
    """
    step_type = step.get("type")
    
    if step_type == "research":
        return research_agent(step["params"], model)
    elif step_type == "write":
        return writer_agent(context, model)
    elif step_type == "edit":
        return editor_agent(context, model)
    elif step_type == "fact_check":  # New step type
        return fact_checker_agent(
            content=context.get("draft"),
            sources=context.get("sources"),
            model=model
        )
    else:
        return {"status": "error", "message": f"Unknown step type: {step_type}"}
```

## Troubleshooting

### Container Won't Start

**Check Postgres initialization:**

```bash
docker logs research-agent | grep Postgres
```

Expected output:
```
🚀 Starting Postgres cluster 17/main...
✅ Postgres is ready
```

**If Postgres fails to start:**

```bash
# Remove any stale PID files
docker exec research-agent rm -f /var/run/postgresql/.s.PGSQL.5432.lock

# Restart the cluster
docker exec research-agent pg_ctlcluster 17 main start
```

### Templates Not Found

Verify templates are copied into the container:

```bash
docker exec research-agent ls -la /app/templates/
```

If missing, check your `Dockerfile`:

```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

### API Key Errors

**Tavily API errors:**

```python
# Check if key is set
docker exec research-agent bash -c 'echo $TAVILY_API_KEY'

# Test Tavily directly
curl -X POST https://api.tavily.com/search \
  -H "Content-Type: application/json" \
  -d "{\"api_key\": \"${TAVILY_API_KEY}\", \"query\": \"test\"}"
```

**OpenAI API errors:**

```bash
# Verify key format (should start with sk-)
docker exec research-agent bash -c 'echo $OPENAI_API_KEY | cut -c1-3'
```

### Task Stuck in Progress

Check the task state in the database:

```sql
-- Connect to database
psql "postgresql://app:local@localhost:5432/appdb"

-- Check task status
SELECT id, prompt, status, updated_at 
FROM tasks 
WHERE status = 'in_progress'
ORDER BY updated_at DESC;

-- Check step details
SELECT ts.step_name, ts.status, ts.result 
FROM task_steps ts 
JOIN tasks t ON ts.task_id = t.id 
WHERE t.id = '550e8400-e29b-41d4-a716-446655440000'
ORDER BY ts.step_order;

-- Manually reset a stuck task
UPDATE tasks 
SET status = 'failed', updated_at = NOW() 
WHERE id = '550e8400-e29b-41d4-a716-446655440000';
```

### Database Connection Issues

**Test connection from within container:**

```bash
docker exec research-agent psql -U app -d appdb -c "SELECT 1;"
```

**Reset the database:**

```bash
# Stop and remove container
docker stop research-agent
docker rm research-agent

# Restart with fresh database
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  --name research-agent \
  --env-file .env \
  -e RESET_DB_ON_STARTUP=1 \
  agentic-research-agent
```

### Enable Debug Logging

Add to your `main.py`:

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

@app.post("/generate_report")
async def generate_report(request: ReportRequest):
    logger.debug(f"Received request: {request.prompt}")
    # ... rest of function
```

### Hot Reload for Development

Mount your code for live updates:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --name research-agent \
  --env-file .env \
  agentic-research-agent \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Production Considerations

### Separate Postgres Container

For production, use a separate managed Postgres instance:

```bash
# Update DATABASE_URL
export DATABASE_URL="postgresql://user:pass@postgres-host:5432/dbname"

# Run API only
docker run -d \
  -p 8000:8000 \
  --name research-agent-api \
  -e DATABASE_URL=$DATABASE_URL \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  agentic-research-agent
```

### Rate Limiting

Add rate limiting to prevent API abuse:

```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.post("/generate_report")
@limiter.limit("5/minute")
async def generate_report(request: Request, report_request: ReportRequest):
    # ... implementation
```

### Background Task Queue

For long-running tasks, use Celery or similar:

```python
from celery import Celery

celery_app = Celery('tasks', broker='redis://localhost:6379')

@celery_app.task
def run_research_workflow(task_id: str, prompt: str, model: str):
    # Execute workflow
    plan = planner_agent(prompt, model)
    # ... rest of workflow
    
    # Update database with results
    update_task_status(task_id, "completed", report)

@app.post("/generate_report")
async def generate_report(request: ReportRequest):
    task_id = str(uuid.uuid4())
    run_research_workflow.delay(task_id, request.prompt, request.model)
    return {"task_id": task_id}
```
