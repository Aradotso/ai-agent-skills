---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with Postgres that orchestrates multi-step AI workflows using Tavily, arXiv, and Wikipedia tools
triggers:
  - how do I use the agentic AI research agent
  - set up the deeplearning.ai research agent service
  - run agentic workflow with Tavily and arXiv
  - create a research task with the planning agent
  - deploy the FastAPI research agent with Postgres
  - configure multi-step agent workflows
  - use the reflective research agent API
  - integrate Tavily arXiv Wikipedia research tools
---

# deeplearning-ai-agentic-research-agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI-based service that orchestrates multi-step research workflows using AI agents. It combines planning, research, writing, and editing agents with external tools (Tavily search, arXiv papers, Wikipedia) to generate comprehensive research reports. The service stores task state and results in Postgres and provides real-time progress tracking.

**Key Features:**
- Multi-agent workflow orchestration (planner → researcher → writer → editor)
- Integration with Tavily, arXiv, and Wikipedia APIs
- PostgreSQL-backed task state management
- Real-time progress tracking via REST API
- Single-container Docker deployment with Postgres + FastAPI
- Web UI for task submission

## Installation

### Prerequisites

- Docker installed
- `.env` file with required API keys:

```bash
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build and Run

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the container
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will be available at:
- API: http://localhost:8000
- Web UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs
- Postgres: localhost:5432

## Project Structure

```
src/
├─ planning_agent.py      # Planner and executor orchestration
├─ agents.py              # Research, writer, editor agents
└─ research_tools.py      # Tavily, arXiv, Wikipedia tool implementations
templates/
└─ index.html            # Web UI template
main.py                  # FastAPI application entry point
Dockerfile               # Single-container build
docker/entrypoint.sh     # Postgres + Uvicorn startup script
```

## API Usage

### Create a Research Task

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

```bash
# Using curl
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications in cryptography", "model":"openai:gpt-4o"}'
```

### Monitor Progress

```python
import requests
import time

def poll_progress(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        progress = response.json()
        
        print(f"Status: {progress['status']}")
        print(f"Current step: {progress.get('current_step', 'N/A')}")
        
        if progress['status'] in ['completed', 'failed']:
            break
        
        time.sleep(2)

poll_progress(task_id)
```

### Retrieve Final Report

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result['status'] == 'completed':
    print("Research Report:")
    print(result['report'])
    print(f"\nSteps executed: {result['steps_completed']}")
else:
    print(f"Task failed: {result.get('error', 'Unknown error')}")
```

## Configuration

### Environment Variables

Required:
```bash
OPENAI_API_KEY=sk-...        # OpenAI API key
TAVILY_API_KEY=tvly-...      # Tavily search API key
```

Optional (defaults provided by entrypoint):
```bash
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb
```

### Database Schema

The service uses SQLAlchemy models for task tracking:

```python
from sqlalchemy import Column, String, Text, DateTime, Integer
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")
    current_step = Column(String)
    steps_completed = Column(Integer, default=0)
    report = Column(Text)
    error = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
```

## Agent Workflow Architecture

### Planning Agent

```python
from src.planning_agent import planner_agent, executor_agent_step

# The planner creates a research plan with steps
plan = planner_agent(
    prompt="Research recent advances in transformer architectures",
    model="openai:gpt-4o"
)

# Executor runs each step sequentially
for step in plan['steps']:
    result = executor_agent_step(
        step=step,
        model="openai:gpt-4o",
        task_id=task_id
    )
    # Results stored in database
```

### Research Tools

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Tavily web search
web_results = tavily_search_tool(
    query="transformer attention mechanisms",
    max_results=5
)

# arXiv academic papers
papers = arxiv_search_tool(
    query="attention is all you need",
    max_results=3
)

# Wikipedia summaries
wiki_info = wikipedia_search_tool(
    query="neural network",
    sentences=5
)
```

### Agent Implementations

```python
from src.agents import research_agent, writer_agent, editor_agent

# Research phase - gathers information
research_results = research_agent(
    topic="Graph Neural Networks",
    tools=[tavily_search_tool, arxiv_search_tool, wikipedia_search_tool],
    model="openai:gpt-4o"
)

# Writing phase - creates draft report
draft = writer_agent(
    research_data=research_results,
    prompt="Write a comprehensive report on GNNs",
    model="openai:gpt-4o"
)

# Editing phase - refines and improves
final_report = editor_agent(
    draft=draft,
    guidelines="Focus on practical applications and recent developments",
    model="openai:gpt-4o"
)
```

## Custom Tool Integration

### Creating a New Research Tool

```python
# In src/research_tools.py
import requests
import os

def custom_api_search_tool(query: str, max_results: int = 5) -> list:
    """
    Custom research tool example
    """
    api_key = os.getenv("CUSTOM_API_KEY")
    if not api_key:
        raise ValueError("CUSTOM_API_KEY not set")
    
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query, "limit": max_results},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    
    if response.status_code != 200:
        return []
    
    results = []
    for item in response.json().get("items", []):
        results.append({
            "title": item["title"],
            "content": item["snippet"],
            "url": item["link"]
        })
    
    return results
```

### Registering Tool with Agents

```python
# In src/agents.py
from src.research_tools import custom_api_search_tool

def research_agent(topic: str, tools: list, model: str):
    # Tools are passed to the agent
    available_tools = tools + [custom_api_search_tool]
    
    # Agent execution logic
    for tool in available_tools:
        try:
            results = tool(query=topic)
            # Process results
        except Exception as e:
            print(f"Tool {tool.__name__} failed: {e}")
            continue
```

## FastAPI Application Structure

### Main Application

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import os
import uuid

app = FastAPI(title="Research Agent API")

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Templates
templates = Jinja2Templates(directory="templates")

# Request/Response models
class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

class TaskResponse(BaseModel):
    task_id: str

@app.post("/generate_report", response_model=TaskResponse)
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    task_id = str(uuid.uuid4())
    
    # Create task in database
    db = SessionLocal()
    task = Task(
        id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background
    background_tasks.add_task(run_research_workflow, task_id, request.prompt, request.model)
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.id,
        "status": task.status,
        "current_step": task.current_step,
        "steps_completed": task.steps_completed
    }

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.id,
        "status": task.status,
        "report": task.report,
        "error": task.error,
        "steps_completed": task.steps_completed
    }
```

### Background Workflow Execution

```python
# In main.py or separate module
from src.planning_agent import planner_agent, executor_agent_step
from sqlalchemy.orm import Session

def run_research_workflow(task_id: str, prompt: str, model: str):
    db = SessionLocal()
    
    try:
        # Update status
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "running"
        task.current_step = "planning"
        db.commit()
        
        # Planning phase
        plan = planner_agent(prompt=prompt, model=model)
        
        # Execute steps
        results = []
        for idx, step in enumerate(plan['steps']):
            task.current_step = step['name']
            task.steps_completed = idx
            db.commit()
            
            result = executor_agent_step(
                step=step,
                model=model,
                task_id=task_id
            )
            results.append(result)
        
        # Mark complete
        task.status = "completed"
        task.report = results[-1].get('final_report', '')
        task.steps_completed = len(plan['steps'])
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
        db.commit()
    
    finally:
        db.close()
```

## Database Operations

### Direct Database Access

```bash
# Connect to Postgres from host
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT id, status, current_step, created_at FROM tasks ORDER BY created_at DESC LIMIT 10;

# View task details
SELECT * FROM tasks WHERE id = 'your-task-id';
```

### Programmatic Database Queries

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Get all pending tasks
db = SessionLocal()
pending_tasks = db.query(Task).filter(Task.status == "pending").all()

for task in pending_tasks:
    print(f"{task.id}: {task.prompt[:50]}...")

db.close()

# Get completed tasks with reports
db = SessionLocal()
completed = db.query(Task).filter(
    Task.status == "completed",
    Task.report.isnot(None)
).order_by(Task.created_at.desc()).limit(5).all()

for task in completed:
    print(f"\n=== {task.prompt} ===")
    print(task.report[:200] + "...")

db.close()
```

## Troubleshooting

### Container Won't Start

**Problem:** Postgres initialization fails

```bash
# Check logs
docker logs fpsvc

# Verify entrypoint permissions
docker exec -it fpsvc ls -la /docker/entrypoint.sh

# Manual Postgres start
docker exec -it fpsvc bash -lc "pg_ctlcluster 17 main start"
```

### Database Connection Errors

**Problem:** `DATABASE_URL not set` or connection refused

```bash
# Verify DATABASE_URL inside container
docker exec -it fpsvc bash -lc "echo \$DATABASE_URL"

# Test Postgres connectivity
docker exec -it fpsvc bash -lc "psql -U app -d appdb -c 'SELECT 1;'"

# Check if Postgres is running
docker exec -it fpsvc bash -lc "pg_lsclusters"
```

### API Key Issues

**Problem:** Tavily or OpenAI authentication fails

```bash
# Verify env vars are loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Rebuild with fresh .env
docker build --no-cache -t fastapi-postgres-service .
docker run --rm -it -p 8000:8000 --env-file .env fastapi-postgres-service
```

### Task Stuck in "Running" State

**Problem:** Workflow hangs or crashes without updating status

```python
# Check task in database
import requests

task_id = "your-task-id"
response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
print(response.json())

# Manually update task status (emergency fix)
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine("postgresql://app:local@localhost:5432/appdb")
SessionLocal = sessionmaker(bind=engine)
db = SessionLocal()

task = db.query(Task).filter(Task.id == task_id).first()
task.status = "failed"
task.error = "Manual reset - workflow timeout"
db.commit()
db.close()
```

### Missing Templates or Static Files

**Problem:** 404 on root path or UI not loading

```bash
# Verify files in container
docker exec -it fpsvc ls -la /app/templates/
docker exec -it fpsvc ls -la /app/static/

# Rebuild ensuring files are copied
# Check Dockerfile has:
# COPY templates/ /app/templates/
# COPY static/ /app/static/
```

### Rate Limiting from External APIs

**Problem:** Tavily, arXiv, or Wikipedia returning errors

```python
# Add retry logic to tools
import time
from functools import wraps

def retry_on_rate_limit(max_retries=3, delay=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if "rate limit" in str(e).lower() and attempt < max_retries - 1:
                        time.sleep(delay * (attempt + 1))
                        continue
                    raise
            return None
        return wrapper
    return decorator

@retry_on_rate_limit(max_retries=3, delay=2)
def tavily_search_tool(query: str, max_results: int = 5):
    # Implementation
    pass
```

## Development and Testing

### Hot Reload for Development

```bash
# Mount code volume for live changes
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Running Tests

```python
# test_agents.py
import pytest
from src.research_tools import tavily_search_tool, arxiv_search_tool

def test_tavily_search():
    results = tavily_search_tool("machine learning", max_results=2)
    assert len(results) <= 2
    assert all("title" in r for r in results)

def test_arxiv_search():
    results = arxiv_search_tool("neural networks", max_results=1)
    assert len(results) <= 1
    if results:
        assert "title" in results[0]
        assert "authors" in results[0]
```

### Reset Database on Startup

```python
# In main.py - conditional DB reset
import os
from sqlalchemy import create_engine

if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)
    print("⚠️  Database reset on startup")
```

```bash
# Run with reset flag
docker run --rm -it \
  -p 8000:8000 \
  --env-file .env \
  -e RESET_DB_ON_STARTUP=1 \
  fastapi-postgres-service
```
