---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres state management for agentic AI workflows
triggers:
  - "set up the agentic research agent"
  - "how do I use the research agent API"
  - "create a research workflow with planning agents"
  - "configure Tavily arXiv Wikipedia research tools"
  - "run a multi-step research task with agents"
  - "track research task progress and status"
  - "build a reflective research agent with FastAPI"
  - "implement planner executor agent pattern"
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic Research Agent is a FastAPI-based service that implements a reflective, multi-agent research workflow. It uses a planning agent to break down research tasks, executes them using specialized agents (research, writer, editor), and integrates tools like Tavily search, arXiv papers, and Wikipedia. All task state and results are persisted in PostgreSQL.

**Key capabilities:**
- **Planning agent**: Decomposes research questions into actionable steps
- **Executor agents**: Research, writer, and editor agents that use external tools
- **Tool integration**: Tavily web search, arXiv academic papers, Wikipedia knowledge
- **State management**: PostgreSQL-backed task tracking and progress monitoring
- **Live progress**: Real-time status updates for long-running research workflows

## Installation

### Prerequisites

1. **Docker** installed on your system
2. **API keys** in a `.env` file at the project root:

```bash
# .env
OPENAI_API_KEY=sk-your-key-here
TAVILY_API_KEY=tvly-your-key-here
```

### Build and Run with Docker

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the container (Postgres + API in one)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service starts Postgres internally, creates the database schema, and launches the FastAPI app on port 8000.

### Access Points

- **Web UI**: http://localhost:8000/
- **API Docs**: http://localhost:8000/docs
- **Postgres**: `postgresql://app:local@localhost:5432/appdb`

## Project Structure

```
.
├── main.py                      # FastAPI application with routes
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, etc.
├── templates/
│   └── index.html               # Web UI template
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt             # Python dependencies
└── Dockerfile
```

## Core API Endpoints

### 1. Generate Report (Start Research Task)

**POST** `/generate_report`

Kicks off a threaded, multi-step agent workflow.

```python
import requests

response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Large Language Models for scientific discovery",
        "model": "openai:gpt-4o"
    }
)

task_data = response.json()
task_id = task_data["task_id"]
print(f"Task started: {task_id}")
```

**Request body:**
```json
{
  "prompt": "Research topic or question",
  "model": "openai:gpt-4o"
}
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 2. Task Progress (Live Updates)

**GET** `/task_progress/{task_id}`

Returns current status, steps completed, and substep details.

```python
import requests
import time

def poll_progress(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        progress = response.json()
        
        print(f"Status: {progress['status']}")
        print(f"Current step: {progress['current_step']}")
        
        if progress['status'] in ['completed', 'failed']:
            break
            
        time.sleep(5)

poll_progress(task_id)
```

**Response:**
```json
{
  "task_id": "550e8400-...",
  "status": "in_progress",
  "current_step": "research",
  "steps": ["planning", "research", "writing", "editing"],
  "substeps": [
    {
      "step": "research",
      "tool": "tavily",
      "result": "Found 5 sources..."
    }
  ]
}
```

### 3. Task Status (Final Result)

**GET** `/task_status/{task_id}`

Returns final status and complete report.

```python
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

print(f"Status: {result['status']}")
if result['status'] == 'completed':
    print(f"Report:\n{result['report']}")
```

**Response:**
```json
{
  "task_id": "550e8400-...",
  "status": "completed",
  "report": "# Research Report\n\nLarge Language Models...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:00Z"
}
```

## Agent Architecture

### Planning Agent

Located in `src/planning_agent.py`. Decomposes research questions into executable steps.

```python
from src.planning_agent import planner_agent

# The planner breaks down a prompt into steps
plan = planner_agent(
    prompt="Explain quantum computing applications in drug discovery",
    model="openai:gpt-4o"
)

# Returns a structured plan with steps like:
# 1. Research quantum computing fundamentals
# 2. Search for drug discovery use cases
# 3. Find academic papers on quantum chemistry
# 4. Synthesize findings into report
```

### Executor Agent

Runs individual steps from the plan using specialized agents.

```python
from src.planning_agent import executor_agent_step

result = executor_agent_step(
    step_description="Research quantum computing fundamentals",
    model="openai:gpt-4o"
)
# Automatically routes to appropriate agent (research/writer/editor)
```

### Research Tools

Located in `src/research_tools.py`. Three main tools:

**Tavily Web Search:**
```python
from src.research_tools import tavily_search_tool

results = tavily_search_tool(query="quantum computing drug discovery")
# Returns web search results with URLs, snippets, relevance scores
```

**arXiv Academic Papers:**
```python
from src.research_tools import arxiv_search_tool

papers = arxiv_search_tool(query="quantum machine learning")
# Returns academic papers with titles, abstracts, authors, URLs
```

**Wikipedia Knowledge:**
```python
from src.research_tools import wikipedia_search_tool

info = wikipedia_search_tool(query="CRISPR gene editing")
# Returns Wikipedia article summary and content
```

## Database Schema

The service uses SQLAlchemy with PostgreSQL. Key models:

```python
from sqlalchemy import Column, String, Text, DateTime, Integer
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Task(Base):
    __tablename__ = 'tasks'
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default='pending')  # pending, in_progress, completed, failed
    current_step = Column(String)
    report = Column(Text)
    created_at = Column(DateTime)
    completed_at = Column(DateTime)
```

## Configuration

### Environment Variables

Create a `.env` file in the project root:

```bash
# Required
OPENAI_API_KEY=sk-your-openai-key
TAVILY_API_KEY=tvly-your-tavily-key

# Optional (override defaults)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0  # Set to 1 to drop/recreate tables on startup
```

### Database Connection

The entrypoint script sets a default `DATABASE_URL`. To connect from external tools:

```bash
# From host machine
psql "postgresql://app:local@localhost:5432/appdb"

# From Python
from sqlalchemy import create_engine
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@localhost:5432/appdb")
engine = create_engine(DATABASE_URL)
```

## Complete Example Workflow

```python
import requests
import time
import json

# Configuration
API_BASE = "http://localhost:8000"

# Step 1: Start a research task
def start_research(prompt, model="openai:gpt-4o"):
    response = requests.post(
        f"{API_BASE}/generate_report",
        json={"prompt": prompt, "model": model}
    )
    return response.json()["task_id"]

# Step 2: Monitor progress
def wait_for_completion(task_id, poll_interval=5):
    while True:
        response = requests.get(f"{API_BASE}/task_progress/{task_id}")
        progress = response.json()
        
        print(f"[{progress['status']}] Step: {progress.get('current_step', 'N/A')}")
        
        # Print substep details
        if progress.get('substeps'):
            latest = progress['substeps'][-1]
            print(f"  → {latest['step']}: {latest.get('tool', 'N/A')}")
        
        if progress['status'] in ['completed', 'failed']:
            return progress['status']
        
        time.sleep(poll_interval)

# Step 3: Retrieve final report
def get_report(task_id):
    response = requests.get(f"{API_BASE}/task_status/{task_id}")
    result = response.json()
    
    if result['status'] == 'completed':
        return result['report']
    else:
        return f"Task failed or incomplete: {result['status']}"

# Run the workflow
if __name__ == "__main__":
    # Start research
    task_id = start_research(
        prompt="Impact of AI on climate change mitigation strategies"
    )
    print(f"Task ID: {task_id}\n")
    
    # Wait for completion
    status = wait_for_completion(task_id)
    
    # Get final report
    if status == 'completed':
        report = get_report(task_id)
        print("\n" + "="*80)
        print("FINAL REPORT")
        print("="*80)
        print(report)
```

## Custom Agent Integration

### Adding a New Tool

Create a new tool in `src/research_tools.py`:

```python
import requests

def custom_api_search_tool(query: str) -> dict:
    """Custom API integration tool."""
    api_key = os.getenv("CUSTOM_API_KEY")
    
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    
    return {
        "tool": "custom_api",
        "query": query,
        "results": response.json()
    }
```

### Creating a Custom Agent

Add to `src/agents.py`:

```python
def analysis_agent(task: str, context: dict, model: str):
    """Agent specialized in data analysis."""
    from src.research_tools import custom_api_search_tool
    
    # Use tools
    data = custom_api_search_tool(task)
    
    # Process with LLM
    prompt = f"""
    Analyze the following data and provide insights:
    
    Task: {task}
    Data: {data}
    """
    
    # Call your LLM client (aisuite, OpenAI, etc.)
    # response = client.chat.completions.create(...)
    
    return {
        "agent": "analysis",
        "result": "Analysis complete..."
    }
```

## Troubleshooting

### Container Won't Start

Check logs:
```bash
docker logs -f fpsvc
```

Ensure Postgres starts successfully:
```bash
docker exec -it fpsvc bash -lc "pg_lsclusters"
```

### Database Tables Missing

The app may drop tables on startup. Disable with:
```python
# In main.py, comment out or guard:
# if os.getenv("RESET_DB_ON_STARTUP") != "1":
#     Base.metadata.drop_all(bind=engine)
```

### API Key Errors

Verify environment variables are loaded:
```bash
docker exec -it fpsvc bash -lc "env | grep API_KEY"
```

Ensure `.env` file is in the correct location and passed with `--env-file`.

### Tavily Rate Limits

Handle rate limiting gracefully:
```python
from src.research_tools import tavily_search_tool
import time

def search_with_retry(query, max_retries=3):
    for attempt in range(max_retries):
        try:
            return tavily_search_tool(query)
        except Exception as e:
            if "rate limit" in str(e).lower():
                wait = 2 ** attempt
                print(f"Rate limited, waiting {wait}s...")
                time.sleep(wait)
            else:
                raise
    raise Exception("Max retries exceeded")
```

### Task Stuck in Progress

Check the background thread status:
```python
# Add logging to main.py executor thread
import logging
logging.basicConfig(level=logging.DEBUG)
```

Inspect database directly:
```sql
SELECT id, status, current_step, created_at 
FROM tasks 
WHERE status = 'in_progress' 
ORDER BY created_at DESC;
```

### Hot Reload for Development

Mount source code for live changes:
```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Production Considerations

### Separate Database

Use an external PostgreSQL instance:
```bash
docker run --rm -it \
  -p 8000:8000 \
  -e DATABASE_URL=postgresql://user:pass@db-host:5432/proddb \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  fastapi-postgres-service
```

### Background Workers

For production, use Celery or similar for task execution instead of threads.

### Monitoring

Add health check endpoint:
```python
@app.get("/health")
async def health_check():
    # Check DB connection
    try:
        db.execute("SELECT 1")
        return {"status": "healthy"}
    except:
        return {"status": "unhealthy"}, 503
```

This skill provides comprehensive guidance for using the DeepLearning.AI Agentic Research Agent service, covering setup, API usage, agent architecture, and troubleshooting for effective AI-powered research workflows.
