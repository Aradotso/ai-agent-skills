---
name: agentic-ai-research-agent
description: FastAPI research agent with planner/executor workflow, Tavily/arXiv/Wikipedia tools, Postgres state tracking
triggers:
  - set up agentic AI research agent
  - create research workflow with planning agent
  - build multi-step research task with Tavily and arXiv
  - implement FastAPI agent with Postgres state tracking
  - deploy reflective research agent service
  - use planning and executor agents for research
  - integrate Tavily arXiv Wikipedia research tools
  - track agent task progress with Postgres
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI service that orchestrates multi-step research workflows using a planning agent that delegates to specialized research, writer, and editor agents. It integrates with Tavily (web search), arXiv (academic papers), and Wikipedia, storing all task state and progress in Postgres. The system uses a planner-executor pattern with live progress tracking.

## Project Structure

```
.
├─ main.py                      # FastAPI app with endpoints
├─ src/
│  ├─ planning_agent.py         # planner_agent(), executor_agent_step()
│  ├─ agents.py                 # research_agent, writer_agent, editor_agent
│  └─ research_tools.py         # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├─ templates/
│  └─ index.html                # UI page
├─ static/                      # CSS/JS assets
├─ docker/
│  └─ entrypoint.sh             # Postgres + Uvicorn startup
├─ requirements.txt
├─ Dockerfile
└─ .env                         # API keys
```

## Installation

### Prerequisites

- Docker (Desktop or Engine)
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```bash
# .env
OPENAI_API_KEY=your-openai-key-here
TAVILY_API_KEY=your-tavily-key-here
```

### Build and Run

```bash
# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the container (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The entrypoint automatically:
- Starts Postgres cluster
- Creates database and role (`app:local@appdb`)
- Initializes tables
- Launches Uvicorn on port 8000

## API Endpoints

### Start a Research Task

```bash
POST /generate_report
Content-Type: application/json

{
  "prompt": "Large Language Models for scientific discovery",
  "model": "openai:gpt-4o"
}

# Response: {"task_id": "uuid-here"}
```

### Check Task Progress (Live Updates)

```bash
GET /task_progress/{task_id}

# Response:
{
  "task_id": "uuid",
  "status": "running",
  "current_step": "research",
  "steps": [
    {"step": "planning", "status": "completed", "message": "Plan created"},
    {"step": "research", "status": "running", "message": "Searching Tavily..."}
  ]
}
```

### Get Final Report

```bash
GET /task_status/{task_id}

# Response:
{
  "task_id": "uuid",
  "status": "completed",
  "report": "# Research Report\n\n...",
  "created_at": "2025-01-15T10:30:00",
  "completed_at": "2025-01-15T10:35:23"
}
```

### UI Access

- Main UI: `http://localhost:8000/`
- API Docs: `http://localhost:8000/docs`

## Core Components

### 1. Planning Agent (`src/planning_agent.py`)

The planner agent breaks down research prompts into subtasks:

```python
from src.planning_agent import planner_agent, executor_agent_step
import aisuite as ai

# Initialize AI client
client = ai.Client()

# Generate a plan
plan = planner_agent(
    client=client,
    prompt="Research quantum computing applications in drug discovery",
    model="openai:gpt-4o"
)

# plan contains: {"steps": [{"agent": "research", "task": "..."}, ...]}

# Execute each step
for step in plan["steps"]:
    result = executor_agent_step(
        client=client,
        agent_name=step["agent"],
        task=step["task"],
        model="openai:gpt-4o"
    )
    print(f"Step {step['agent']}: {result}")
```

### 2. Research Tools (`src/research_tools.py`)

Three primary research tools are available:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Tavily web search (requires TAVILY_API_KEY)
tavily_results = tavily_search_tool("latest AI breakthroughs 2024")
# Returns: [{"title": "...", "url": "...", "content": "..."}, ...]

# arXiv academic papers
arxiv_results = arxiv_search_tool("transformer architecture")
# Returns: [{"title": "...", "authors": [...], "summary": "...", "pdf_url": "..."}, ...]

# Wikipedia articles
wiki_results = wikipedia_search_tool("neural networks")
# Returns: {"title": "...", "summary": "...", "url": "..."}
```

### 3. Specialized Agents (`src/agents.py`)

```python
from src.agents import research_agent, writer_agent, editor_agent
import aisuite as ai

client = ai.Client()

# Research agent - gathers information
research_data = research_agent(
    client=client,
    topic="climate change mitigation strategies",
    model="openai:gpt-4o"
)

# Writer agent - creates draft report
draft = writer_agent(
    client=client,
    research_data=research_data,
    model="openai:gpt-4o"
)

# Editor agent - refines and polishes
final_report = editor_agent(
    client=client,
    draft=draft,
    model="openai:gpt-4o"
)
```

## FastAPI Integration Pattern

### Task State Management

```python
from fastapi import FastAPI, BackgroundTasks
from sqlalchemy.orm import Session
from uuid import uuid4
import threading

app = FastAPI()

@app.post("/generate_report")
async def generate_report(
    request: dict,
    background_tasks: BackgroundTasks,
    db: Session = Depends(get_db)
):
    task_id = str(uuid4())
    
    # Create task record
    task = Task(
        id=task_id,
        prompt=request["prompt"],
        model=request.get("model", "openai:gpt-4o"),
        status="queued"
    )
    db.add(task)
    db.commit()
    
    # Run workflow in background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request["prompt"], request["model"])
    )
    thread.start()
    
    return {"task_id": task_id}

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background task that runs the full agent workflow"""
    client = ai.Client()
    db = SessionLocal()
    
    try:
        # Update status
        update_task_status(db, task_id, "planning")
        
        # Step 1: Planning
        plan = planner_agent(client, prompt, model)
        
        # Step 2: Execute plan
        results = []
        for step in plan["steps"]:
            update_task_status(db, task_id, f"running_{step['agent']}")
            result = executor_agent_step(client, step["agent"], step["task"], model)
            results.append(result)
        
        # Step 3: Generate final report
        update_task_status(db, task_id, "generating_report")
        report = generate_final_report(results)
        
        # Save and complete
        task = db.query(Task).filter(Task.id == task_id).first()
        task.report = report
        task.status = "completed"
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
        db.commit()
    finally:
        db.close()
```

## Database Schema

The service uses SQLAlchemy with Postgres:

```python
from sqlalchemy import Column, String, DateTime, Text, JSON
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, default="openai:gpt-4o")
    status = Column(String, default="queued")  # queued, running, completed, failed
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)
    
class TaskProgress(Base):
    __tablename__ = "task_progress"
    
    id = Column(String, primary_key=True)
    task_id = Column(String, nullable=False)
    step = Column(String, nullable=False)  # planning, research, writing, editing
    status = Column(String, nullable=False)  # running, completed, failed
    message = Column(Text, nullable=True)
    data = Column(JSON, nullable=True)
    timestamp = Column(DateTime, default=datetime.utcnow)
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key for AI models
TAVILY_API_KEY=tvly-...            # Tavily API key for web search

# Database (defaults provided by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app                  # Database user
POSTGRES_PASSWORD=local            # Database password
POSTGRES_DB=appdb                  # Database name

# Optional
RESET_DB_ON_STARTUP=0              # Set to 1 to drop/recreate tables on start
```

### Model Selection

The service supports any `aisuite`-compatible model:

```python
# OpenAI models
model = "openai:gpt-4o"
model = "openai:gpt-4o-mini"

# Anthropic models
model = "anthropic:claude-3-5-sonnet-20241022"

# Other providers supported by aisuite
```

## Common Usage Patterns

### Simple Research Query

```python
import requests

# Start research
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "What are the latest developments in quantum computing?",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]

# Poll for completion
import time
while True:
    status = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
    if status["status"] in ["completed", "failed"]:
        break
    time.sleep(2)

# Get final report
print(status["report"])
```

### Custom Agent Workflow

```python
from src.planning_agent import planner_agent, executor_agent_step
from src.research_tools import tavily_search_tool, arxiv_search_tool
import aisuite as ai

client = ai.Client()

# Step 1: Manual planning
steps = [
    {"agent": "research", "task": "Find recent papers on reinforcement learning"},
    {"agent": "research", "task": "Search web for RL applications in robotics"},
    {"agent": "writer", "task": "Synthesize findings into a technical summary"}
]

# Step 2: Execute with tool selection
results = []
for step in steps:
    if step["agent"] == "research":
        # Use appropriate tool
        if "papers" in step["task"]:
            data = arxiv_search_tool(step["task"])
        else:
            data = tavily_search_tool(step["task"])
        results.append(data)
    else:
        # Use LLM agent
        result = executor_agent_step(client, step["agent"], step["task"], "openai:gpt-4o")
        results.append(result)
```

### Streaming Progress Updates

```python
from fastapi.responses import StreamingResponse
import json
import asyncio

@app.get("/stream_progress/{task_id}")
async def stream_progress(task_id: str, db: Session = Depends(get_db)):
    async def event_generator():
        while True:
            # Query latest progress
            progress = db.query(TaskProgress).filter(
                TaskProgress.task_id == task_id
            ).order_by(TaskProgress.timestamp.desc()).first()
            
            if progress:
                yield f"data: {json.dumps({'step': progress.step, 'status': progress.status})}\n\n"
            
            task = db.query(Task).filter(Task.id == task_id).first()
            if task.status in ["completed", "failed"]:
                break
                
            await asyncio.sleep(1)
    
    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

## Troubleshooting

### Database Connection Issues

```bash
# Check if Postgres is running inside container
docker exec -it fpsvc bash -lc "pg_isready"

# Connect to database manually
docker exec -it fpsvc psql -U app -d appdb

# Verify tables exist
docker exec -it fpsvc psql -U app -d appdb -c "\dt"
```

### Missing API Keys

```bash
# Verify environment variables are loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Re-run with explicit env vars
docker run --rm -it -p 8000:8000 -p 5432:5432 \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  --name fpsvc fastapi-postgres-service
```

### Task Stuck in "running" State

```python
# Reset task status manually
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine("postgresql://app:local@localhost:5432/appdb")
Session = sessionmaker(bind=engine)
db = Session()

# Find stuck task
task = db.query(Task).filter(Task.id == "your-task-id").first()
task.status = "failed"
task.error = "Manually reset"
db.commit()
```

### Template Not Found

```bash
# Verify templates directory in container
docker exec -it fpsvc ls -la /app/templates/

# Rebuild if missing
docker build --no-cache -t fastapi-postgres-service .
```

### Tool-Specific Errors

```python
# Tavily API rate limit
# Solution: Add retry logic with exponential backoff
import time

def tavily_search_with_retry(query, max_retries=3):
    for attempt in range(max_retries):
        try:
            return tavily_search_tool(query)
        except Exception as e:
            if "rate limit" in str(e).lower() and attempt < max_retries - 1:
                time.sleep(2 ** attempt)
            else:
                raise

# Wikipedia timeout
# Solution: Use shorter summaries
def wikipedia_search_tool(query, sentences=3):
    import wikipedia
    wikipedia.set_lang("en")
    try:
        return wikipedia.summary(query, sentences=sentences)
    except wikipedia.exceptions.DisambiguationError as e:
        return wikipedia.summary(e.options[0], sentences=sentences)
```

## Development Mode

```bash
# Run with hot reload (mount local code)
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "
    pg_ctlcluster 17 main start && \
    uvicorn main:app --host 0.0.0.0 --port 8000 --reload
  "
```

## Testing

```python
# Test individual components
import pytest
from src.research_tools import tavily_search_tool

def test_tavily_search():
    results = tavily_search_tool("artificial intelligence")
    assert len(results) > 0
    assert "title" in results[0]
    assert "content" in results[0]

def test_workflow_execution():
    from src.planning_agent import planner_agent
    import aisuite as ai
    
    client = ai.Client()
    plan = planner_agent(
        client,
        "Research machine learning trends",
        "openai:gpt-4o"
    )
    
    assert "steps" in plan
    assert len(plan["steps"]) > 0
```
