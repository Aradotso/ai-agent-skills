---
name: agentic-ai-research-agent
description: FastAPI research agent service with planning, execution, and reflection using Tavily, arXiv, and Wikipedia tools
triggers:
  - how do I build a research agent with planning and reflection
  - set up the agentic AI research workflow service
  - create a multi-step research agent with Tavily and arXiv
  - deploy the FastAPI research agent with Postgres
  - implement a reflective research workflow agent
  - use the DeepLearning.AI agentic workflow service
  - build an agent that plans research tasks and generates reports
  - run a research agent with task tracking and progress updates
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This project is a FastAPI-based research agent service that implements an agentic workflow with planning, execution, and reflection. It orchestrates multi-step research tasks using tools like Tavily search, arXiv papers, and Wikipedia, storing task state and results in PostgreSQL.

## What It Does

- **Planning Agent**: Breaks down research queries into structured subtasks
- **Research Tools**: Integrates Tavily web search, arXiv academic papers, and Wikipedia
- **Multi-Agent Workflow**: Coordinates research, writer, and editor agents
- **Task Tracking**: Stores progress and results in Postgres with live status updates
- **Web UI**: Simple interface to kick off research tasks and monitor progress
- **REST API**: Programmatic access to generate reports and track task status

## Installation

### Prerequisites

Create a `.env` file in the project root:

```bash
OPENAI_API_KEY=your_openai_key_here
TAVILY_API_KEY=your_tavily_key_here
```

### Docker Setup (Recommended)

```bash
# Build the image
docker build -t fastapi-postgres-service .

# Run the container (Postgres + API in one)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will be available at `http://localhost:8000`.

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up PostgreSQL separately
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"

# Run the service
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
├── main.py                     # FastAPI application with routes
├── src/
│   ├── planning_agent.py       # Planner and executor agents
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
└── Dockerfile
```

## Key API Endpoints

### Generate Report

Initiates a research task with planning and execution:

```python
import requests

response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Large Language Models for scientific discovery",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task started: {task_id}")
```

### Poll Task Progress

Get live updates on task execution:

```python
import requests
import time

def monitor_task(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = response.json()
        
        print(f"Status: {data['status']}")
        print(f"Current step: {data.get('current_step', 'N/A')}")
        
        if data["status"] in ["completed", "failed"]:
            break
        
        time.sleep(2)

monitor_task(task_id)
```

### Get Final Report

Retrieve completed research report:

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result["status"] == "completed":
    print("Report:", result["report"])
    print("Steps completed:", result["steps_completed"])
else:
    print("Error:", result.get("error"))
```

## Agent Architecture

### Planning Agent

The planning agent breaks down research queries into structured subtasks:

```python
from src.planning_agent import planner_agent

# Example usage in your custom workflow
plan = planner_agent(
    query="Explain quantum computing applications",
    model="openai:gpt-4o"
)

# Plan contains structured steps
for step in plan["steps"]:
    print(f"Step {step['id']}: {step['action']}")
    print(f"Tools: {step['tools']}")
```

### Research Tools

The service provides three research tools:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Tavily web search
web_results = tavily_search_tool(
    query="latest advances in transformer models",
    max_results=5
)

# arXiv academic papers
papers = arxiv_search_tool(
    query="attention mechanisms neural networks",
    max_results=3
)

# Wikipedia lookup
wiki_info = wikipedia_search_tool(
    query="artificial neural network"
)
```

### Executor Agent

Execute individual research steps:

```python
from src.planning_agent import executor_agent_step

result = executor_agent_step(
    step={
        "id": 1,
        "action": "search_web",
        "query": "transformer architecture",
        "tools": ["tavily"]
    },
    model="openai:gpt-4o",
    task_id="uuid-here"
)

print(result["findings"])
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...                    # OpenAI API key
TAVILY_API_KEY=tvly-...                  # Tavily search API key

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional Postgres overrides
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0                    # Set to 1 to drop tables on restart
```

### Model Selection

The service supports multiple models via `aisuite`:

```python
# OpenAI models
{"prompt": "Your query", "model": "openai:gpt-4o"}
{"prompt": "Your query", "model": "openai:gpt-4-turbo"}
{"prompt": "Your query", "model": "openai:gpt-3.5-turbo"}

# Anthropic (if configured)
{"prompt": "Your query", "model": "anthropic:claude-3-opus"}
```

## Common Patterns

### Full Research Workflow

```python
import requests
import time
import json

def run_research_workflow(query, model="openai:gpt-4o"):
    # Start the task
    response = requests.post(
        "http://localhost:8000/generate_report",
        json={"prompt": query, "model": model}
    )
    task_id = response.json()["task_id"]
    
    # Monitor progress
    while True:
        progress = requests.get(
            f"http://localhost:8000/task_progress/{task_id}"
        ).json()
        
        print(f"[{progress['status']}] {progress.get('current_step', '')}")
        
        if progress["status"] in ["completed", "failed"]:
            break
        
        time.sleep(3)
    
    # Get final report
    result = requests.get(
        f"http://localhost:8000/task_status/{task_id}"
    ).json()
    
    return result

# Execute
report = run_research_workflow(
    "How does CRISPR gene editing work?",
    model="openai:gpt-4o"
)

print(json.dumps(report, indent=2))
```

### Custom Agent Integration

Extend the workflow with custom agents:

```python
from src.agents import research_agent, writer_agent, editor_agent

def custom_research_pipeline(query, model):
    # Research phase
    research_findings = research_agent(
        query=query,
        tools=["tavily", "arxiv"],
        model=model
    )
    
    # Writing phase
    draft_report = writer_agent(
        findings=research_findings,
        model=model
    )
    
    # Editing phase
    final_report = editor_agent(
        draft=draft_report,
        model=model
    )
    
    return final_report
```

### Database Query Patterns

```python
from sqlalchemy.orm import Session
from main import Task, engine

def get_recent_tasks(limit=10):
    with Session(engine) as session:
        tasks = session.query(Task).order_by(
            Task.created_at.desc()
        ).limit(limit).all()
        
        return [
            {
                "id": str(task.id),
                "prompt": task.prompt,
                "status": task.status,
                "created_at": task.created_at
            }
            for task in tasks
        ]

def get_task_by_id(task_id):
    with Session(engine) as session:
        task = session.query(Task).filter(
            Task.id == task_id
        ).first()
        
        return task
```

## Troubleshooting

### Container Won't Start

Check if Postgres initialized correctly:

```bash
docker logs fpsvc

# Should see:
# 🚀 Starting Postgres cluster 17/main...
# ✅ Postgres is ready
```

If it fails, ensure no other Postgres is running on port 5432.

### API Key Errors

Verify environment variables are loaded:

```bash
docker exec -it fpsvc bash -lc "env | grep API_KEY"
```

Should show both `OPENAI_API_KEY` and `TAVILY_API_KEY`.

### Template Not Found

Ensure templates are copied to the container:

```bash
docker exec -it fpsvc bash -lc "ls -l /app/templates"

# Should show index.html
```

If missing, check your Dockerfile `COPY` commands.

### Database Connection Issues

Test database connectivity:

```bash
# From host
psql "postgresql://app:local@localhost:5432/appdb"

# Inside container
docker exec -it fpsvc bash -lc "psql \$DATABASE_URL -c 'SELECT 1;'"
```

### Tables Not Persisting

If tables disappear on restart, disable automatic drop in `main.py`:

```python
# In main.py startup
# Comment out or guard:
# Base.metadata.drop_all(bind=engine)

# Only drop if explicitly requested
import os
if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
    
Base.metadata.create_all(bind=engine)
```

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

### Task Stuck in Processing

Check task status in database:

```python
from sqlalchemy.orm import Session
from main import Task, engine

def reset_stuck_tasks():
    with Session(engine) as session:
        stuck = session.query(Task).filter(
            Task.status == "processing"
        ).all()
        
        for task in stuck:
            task.status = "failed"
            task.error = "Task timed out"
        
        session.commit()
        print(f"Reset {len(stuck)} stuck tasks")
```

### Hot Reload for Development

Run with code mounting for live updates:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Advanced Usage

### Custom Tool Integration

Add your own research tools:

```python
# In src/research_tools.py

def custom_search_tool(query: str, **kwargs):
    """Your custom search implementation"""
    # Implementation here
    return {
        "source": "custom",
        "results": results
    }

# Register in planning_agent.py
AVAILABLE_TOOLS = {
    "tavily": tavily_search_tool,
    "arxiv": arxiv_search_tool,
    "wikipedia": wikipedia_search_tool,
    "custom": custom_search_tool
}
```

### Async Task Processing

For production, consider using Celery or similar:

```python
from celery import Celery

celery_app = Celery(
    "research_tasks",
    broker=os.getenv("REDIS_URL"),
    backend=os.getenv("REDIS_URL")
)

@celery_app.task
def async_research_task(task_id, prompt, model):
    # Execute research workflow
    # Update database with results
    pass
```
