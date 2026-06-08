---
name: agentic-ai-research-agent
description: FastAPI research agent service with multi-agent workflow, Postgres state storage, and tool integration (Tavily, arXiv, Wikipedia)
triggers:
  - how do I build a research agent with multiple steps
  - set up the agentic AI research workflow service
  - create a multi-agent research system with planning
  - use tavily and arxiv for AI research agents
  - build a reflective research agent with FastAPI
  - implement agent workflow with task state tracking
  - deploy research agent service with Postgres
  - create research report with multi-step agent planning
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI-based service that orchestrates multi-step research workflows using AI agents. It features:

- **Multi-agent architecture**: Planner, research, writer, and editor agents working in sequence
- **Tool integration**: Tavily search, arXiv papers, and Wikipedia lookups
- **State management**: Postgres database for tracking task progress and storing results
- **Async workflows**: Threaded execution with live progress tracking
- **Web UI**: Simple interface for kicking off research tasks

The system follows an agentic workflow pattern: plan → research → write → edit, with each step stored and queryable.

## Installation

### Prerequisites

- Docker (Desktop on Windows/macOS, or Engine on Linux)
- API keys in `.env` file:

```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build and Run

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build Docker image
docker build -t fastapi-postgres-service .

# Run container (foreground)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The container automatically:
- Starts Postgres cluster
- Creates `app` user and `appdb` database
- Runs migrations
- Launches Uvicorn on port 8000

### Access Points

- Web UI: http://localhost:8000/
- API docs: http://localhost:8000/docs
- Postgres: `postgresql://app:local@localhost:5432/appdb`

## Project Structure

```
.
├── main.py                      # FastAPI app, routes, task execution
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily, arxiv, wikipedia tools
├── templates/
│   └── index.html               # UI template (Jinja2)
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Postgres setup + Uvicorn start
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core API Endpoints

### 1. Generate Research Report

Kicks off a multi-step agent workflow in a background thread.

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

### 2. Track Progress

Poll for live updates on task execution.

```python
import requests
import time

def poll_progress(task_id):
    while True:
        resp = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = resp.json()
        
        print(f"Status: {data['status']}")
        if data.get("steps"):
            for step in data["steps"]:
                print(f"  {step['step_name']}: {step['status']}")
        
        if data["status"] in ["completed", "failed"]:
            break
        
        time.sleep(2)

poll_progress(task_id)
```

### 3. Get Final Status

Retrieve completed report and full task history.

```python
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

print(f"Status: {result['status']}")
print(f"Report:\n{result.get('final_report', 'N/A')}")

# Access detailed step results
for step in result.get("steps", []):
    print(f"\n{step['step_name']}:")
    print(f"  Status: {step['status']}")
    print(f"  Result: {step.get('result', '')[:200]}")
```

## Agent Architecture

### Planning Agent

The planner breaks down research into executable steps.

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> list:
    """Generate a structured plan for research."""
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": """You are a research planning agent. Create a step-by-step 
            plan with these agent types: research_agent, writer_agent, editor_agent.
            Return JSON array: [{"step": 1, "agent": "research_agent", "task": "..."}]"""
        },
        {"role": "user", "content": prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    plan = response.choices[0].message.content
    return parse_plan(plan)  # Parse JSON response
```

### Research Agent

Searches multiple sources using tools.

```python
# src/agents.py
from src.research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(task: str, model: str = "openai:gpt-4o") -> str:
    """Execute research using available tools."""
    client = Client()
    
    # Tool definitions for function calling
    tools = [
        {
            "type": "function",
            "function": {
                "name": "tavily_search",
                "description": "Search the web for current information",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string"}
                    },
                    "required": ["query"]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "arxiv_search",
                "description": "Search arXiv for academic papers",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string"},
                        "max_results": {"type": "integer", "default": 5}
                    },
                    "required": ["query"]
                }
            }
        }
    ]
    
    messages = [
        {"role": "system", "content": "You are a research agent. Use tools to gather information."},
        {"role": "user", "content": task}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        tools=tools
    )
    
    # Execute tool calls
    if response.choices[0].message.tool_calls:
        for tool_call in response.choices[0].message.tool_calls:
            if tool_call.function.name == "tavily_search":
                args = json.loads(tool_call.function.arguments)
                result = tavily_search_tool(args["query"])
                # Append result to messages and continue...
    
    return response.choices[0].message.content
```

### Research Tools

```python
# src/research_tools.py
import os
import requests
import arxiv
import wikipedia

def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    """Search using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    return response.json()

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv for papers."""
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for paper in search.results():
        results.append({
            "title": paper.title,
            "authors": [a.name for a in paper.authors],
            "summary": paper.summary[:500],
            "url": paper.pdf_url
        })
    return results

def wikipedia_search_tool(query: str) -> dict:
    """Get Wikipedia summary."""
    try:
        summary = wikipedia.summary(query, sentences=5, auto_suggest=True)
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "summary": summary,
            "url": page.url,
            "title": page.title
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": "disambiguation", "options": e.options[:5]}
    except Exception as e:
        return {"error": str(e)}
```

## Database Models

Task state is stored in Postgres using SQLAlchemy.

```python
# main.py (excerpt)
from sqlalchemy import create_engine, Column, String, DateTime, Text, Integer
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    final_report = Column(Text, nullable=True)

class TaskStep(Base):
    __tablename__ = "task_steps"
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    task_id = Column(String, nullable=False)
    step_name = Column(String, nullable=False)
    status = Column(String, default="pending")
    result = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Complete Workflow Example

```python
import requests
import time
import json

# 1. Start a research task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Latest advances in transformer architectures for NLP",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Started task: {task_id}")

# 2. Poll until complete
while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    print(f"\nStatus: {progress['status']}")
    
    if progress.get("steps"):
        for step in progress["steps"]:
            print(f"  [{step['status']}] {step['step_name']}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(3)

# 3. Get final report
final = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
print(f"\n{'='*60}")
print(f"Final Report:")
print(f"{'='*60}")
print(final.get("final_report", "No report generated"))

# 4. Review step-by-step results
for step in final.get("steps", []):
    print(f"\n## {step['step_name']}")
    print(f"Status: {step['status']}")
    if step.get("result"):
        print(f"Output: {step['result'][:300]}...")
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key
TAVILY_API_KEY=tvly-...            # Tavily search API key

# Optional (defaults set by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0              # Set to 1 to drop tables on start
```

### Custom Agent Models

Override the default model in API calls:

```python
# Use Claude via aisuite
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Your research question",
        "model": "anthropic:claude-3-5-sonnet-20241022"
    }
)

# Use local model
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Your research question",
        "model": "ollama:llama3.1"
    }
)
```

## Advanced Patterns

### Custom Step Execution

```python
# src/planning_agent.py
def executor_agent_step(step: dict, model: str) -> str:
    """Execute a single step in the plan."""
    agent_type = step["agent"]
    task = step["task"]
    
    if agent_type == "research_agent":
        from src.agents import research_agent
        return research_agent(task, model)
    
    elif agent_type == "writer_agent":
        from src.agents import writer_agent
        return writer_agent(task, model)
    
    elif agent_type == "editor_agent":
        from src.agents import editor_agent
        return editor_agent(task, model)
    
    else:
        raise ValueError(f"Unknown agent type: {agent_type}")
```

### Task Cancellation

```python
from fastapi import HTTPException

@app.post("/cancel_task/{task_id}")
def cancel_task(task_id: str):
    """Cancel a running task."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    if task.status == "completed":
        raise HTTPException(status_code=400, detail="Task already completed")
    
    task.status = "cancelled"
    db.commit()
    db.close()
    
    return {"message": "Task cancelled", "task_id": task_id}
```

### Direct Database Queries

```python
import psycopg2
import os

conn = psycopg2.connect(os.getenv("DATABASE_URL"))
cur = conn.cursor()

# Get all tasks from last 24 hours
cur.execute("""
    SELECT id, prompt, status, created_at 
    FROM tasks 
    WHERE created_at > NOW() - INTERVAL '24 hours'
    ORDER BY created_at DESC
""")

for row in cur.fetchall():
    print(f"{row[0]}: {row[2]} - {row[1][:50]}...")

cur.close()
conn.close()
```

## Troubleshooting

### Container Won't Start

**Check logs:**
```bash
docker logs fpsvc
```

**Postgres won't start:**
- Ensure ports 5432 and 8000 aren't in use: `lsof -i :5432`
- Check if `/var/lib/postgresql` has correct permissions

**DATABASE_URL not set:**
- The entrypoint exports a default; verify with: `docker exec fpsvc env | grep DATABASE`

### API Errors

**"TAVILY_API_KEY not set":**
```bash
# Verify .env file exists and is loaded
docker run --rm --env-file .env fastapi-postgres-service env | grep TAVILY
```

**Tool calls failing:**
- Check API key validity
- Verify network access from container: `docker exec fpsvc curl https://api.tavily.com`
- Wikipedia rate limits: implement retry logic with exponential backoff

### Database Issues

**Tables disappear on restart:**
```python
# main.py - comment out for persistence
# Base.metadata.drop_all(bind=engine)  # DON'T drop tables
Base.metadata.create_all(bind=engine)   # Only create if not exist
```

**Connection errors:**
```bash
# Test connection from host
psql "postgresql://app:local@localhost:5432/appdb"

# Test from container
docker exec -it fpsvc psql -U app -d appdb -c "SELECT COUNT(*) FROM tasks;"
```

### Hot Reload for Development

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "
    pg_ctlcluster 17 main start && 
    uvicorn main:app --host 0.0.0.0 --port 8000 --reload
  "
```

### Memory Issues

Large research tasks can consume memory. Monitor with:

```bash
docker stats fpsvc
```

Limit container memory:
```bash
docker run --memory="4g" --memory-swap="4g" ...
```

## Testing

```python
# test_research_agent.py
import requests
import pytest

BASE_URL = "http://localhost:8000"

def test_generate_report():
    response = requests.post(
        f"{BASE_URL}/generate_report",
        json={"prompt": "Test query", "model": "openai:gpt-4o"}
    )
    assert response.status_code == 200
    assert "task_id" in response.json()

def test_task_progress():
    # First create a task
    create_resp = requests.post(
        f"{BASE_URL}/generate_report",
        json={"prompt": "Test", "model": "openai:gpt-4o"}
    )
    task_id = create_resp.json()["task_id"]
    
    # Check progress
    progress_resp = requests.get(f"{BASE_URL}/task_progress/{task_id}")
    assert progress_resp.status_code == 200
    assert "status" in progress_resp.json()
```

Run tests:
```bash
pytest test_research_agent.py -v
```
