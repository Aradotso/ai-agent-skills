---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent with planning, tool-calling (Tavily/arXiv/Wikipedia), and Postgres task tracking
triggers:
  - how do I run the agentic research agent
  - set up the reflective research agent service
  - how to use the deeplearning ai research workflow
  - create a research task with planning agents
  - how to track research agent progress
  - configure the agentic research FastAPI service
  - build and run the research agent container
  - integrate tavily and arxiv search tools
---

# deeplearning-ai-agentic-research-agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The **Reflective Research Agent** is a FastAPI service that orchestrates multi-step research workflows using AI agents. It combines a **planning agent** (generates subtasks), **executor agents** (research/writer/editor), and **tool-calling** capabilities (Tavily web search, arXiv papers, Wikipedia) with Postgres-backed task state management.

**Key capabilities:**
- `/generate_report` endpoint to start agentic research workflows
- Planning agent breaks down complex research queries into subtasks
- Research, writer, and editor agents collaborate on output
- Real-time progress tracking via `/task_progress/{task_id}`
- Single-container deployment (Postgres + FastAPI)

## Installation

### Prerequisites

- Docker installed
- `.env` file in project root:

```bash
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build the Container

```bash
docker build -t fastapi-postgres-service .
```

### Run the Service

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The entrypoint script:
1. Starts Postgres cluster
2. Creates `app` user and `appdb` database
3. Sets `DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb`
4. Launches Uvicorn on port 8000

## Project Structure

```
.
├── main.py                     # FastAPI app, routes, SQLAlchemy models
├── src/
│   ├── planning_agent.py       # planner_agent(), executor_agent_step()
│   ├── agents.py               # research_agent, writer_agent, editor_agent
│   └── research_tools.py       # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html              # UI for submitting research tasks
├── static/                     # CSS/JS assets
├── docker/
│   └── entrypoint.sh           # DB + Uvicorn startup
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core API Endpoints

### Start a Research Task

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

**Response:**
```json
{"task_id": "uuid-here"}
```

### Poll Task Progress

```python
import requests
import time

task_id = "uuid-here"

while True:
    resp = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    data = resp.json()
    
    if data["status"] in ["completed", "failed"]:
        break
    
    print(f"Current step: {data['current_step']}")
    print(f"Progress: {data['progress']}")
    time.sleep(2)
```

**Response Structure:**
```json
{
  "task_id": "uuid",
  "status": "in_progress",
  "current_step": "research_agent",
  "progress": {
    "planning": "completed",
    "research": "in_progress"
  },
  "substeps": [...]
}
```

### Get Final Report

```python
resp = requests.get(f"http://localhost:8000/task_status/{task_id}")
data = resp.json()

if data["status"] == "completed":
    print(data["report"])
```

## Implementing Custom Agents

### Planning Agent Pattern

The planning agent decomposes a research prompt into subtasks:

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(prompt: str, model: str) -> dict:
    """
    Returns: {
      "plan": ["step1", "step2", ...],
      "raw_response": "..."
    }
    """
    client = Client()
    
    messages = [
        {"role": "system", "content": "You are a research planning agent..."},
        {"role": "user", "content": f"Create a research plan for: {prompt}"}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    # Parse response into structured plan
    plan_steps = parse_plan(response.choices[0].message.content)
    
    return {
        "plan": plan_steps,
        "raw_response": response.choices[0].message.content
    }
```

### Research Agent with Tools

```python
# src/agents.py
from src.research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(query: str, model: str) -> str:
    """Execute research using available tools"""
    
    # Web search
    web_results = tavily_search_tool(query)
    
    # Academic papers
    paper_results = arxiv_search_tool(query)
    
    # Synthesize findings
    client = Client()
    messages = [
        {"role": "system", "content": "Synthesize research findings..."},
        {"role": "user", "content": f"Web: {web_results}\n\nPapers: {paper_results}"}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Tool Implementation

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """Search web using Tavily API"""
    api_key = os.getenv("TAVILY_API_KEY")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    
    return response.json().get("results", [])

def arxiv_search_tool(query: str, max_results: int = 3) -> list:
    """Search arXiv papers"""
    import arxiv
    
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    return [
        {
            "title": result.title,
            "summary": result.summary,
            "url": result.entry_id
        }
        for result in search.results()
    ]
```

## Database Models

The service uses SQLAlchemy with Postgres:

```python
# main.py
from sqlalchemy import Column, String, Text, DateTime, Integer, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, in_progress, completed, failed
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...

# Database (auto-configured by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional overrides
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb
RESET_DB_ON_STARTUP=0  # Set to 1 to drop tables on restart
```

### Custom Models

The service uses `aisuite` client, supporting multiple providers:

```python
# Use OpenAI
{"model": "openai:gpt-4o"}

# Use Anthropic
{"model": "anthropic:claude-3-opus-20240229"}

# Use other aisuite-supported providers
{"model": "provider:model-name"}
```

## Common Patterns

### Async Task Execution

```python
# main.py
import threading
from uuid import uuid4

@app.post("/generate_report")
async def generate_report(request: ResearchRequest):
    task_id = str(uuid4())
    
    # Store initial task
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background thread
    thread = threading.Thread(
        target=run_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}

def run_workflow(task_id: str, prompt: str, model: str):
    """Execute multi-agent workflow"""
    db = SessionLocal()
    
    try:
        # Update status
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "in_progress"
        task.current_step = "planning"
        db.commit()
        
        # Planning phase
        plan = planner_agent(prompt, model)
        
        # Research phase
        task.current_step = "research"
        db.commit()
        research_output = research_agent(prompt, model)
        
        # Writing phase
        task.current_step = "writing"
        db.commit()
        draft = writer_agent(research_output, model)
        
        # Editing phase
        task.current_step = "editing"
        db.commit()
        final_report = editor_agent(draft, model)
        
        # Mark complete
        task.status = "completed"
        task.report = final_report
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()
```

### Substep Progress Tracking

```python
# Store granular progress in a separate table
class TaskProgress(Base):
    __tablename__ = "task_progress"
    
    id = Column(Integer, primary_key=True)
    task_id = Column(String, nullable=False)
    step = Column(String, nullable=False)
    substep = Column(String, nullable=True)
    status = Column(String, default="pending")
    output = Column(Text, nullable=True)
    timestamp = Column(DateTime, default=datetime.utcnow)
```

## Troubleshooting

### Database Connection Errors

**Issue:** `DATABASE_URL not set` or connection refused

```bash
# Check if Postgres is running inside container
docker exec -it fpsvc bash -c "pg_isready"

# Verify DATABASE_URL
docker exec -it fpsvc bash -c 'echo $DATABASE_URL'

# Connect manually to test
docker exec -it fpsvc bash -c 'psql $DATABASE_URL -c "SELECT 1;"'
```

### Missing Templates

**Issue:** 404 on `/` or template errors

```bash
# Verify templates are copied
docker exec -it fpsvc ls -la /app/templates/

# Check Dockerfile includes:
# COPY templates /app/templates
# COPY static /app/static
```

### Tool API Failures

**Issue:** Tavily/arXiv returning errors

```python
# Add error handling to tools
def tavily_search_tool(query: str, max_results: int = 5) -> list:
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return [{"error": "TAVILY_API_KEY not set"}]
    
    try:
        response = requests.post(
            "https://api.tavily.com/search",
            json={"api_key": api_key, "query": query, "max_results": max_results},
            timeout=10
        )
        response.raise_for_status()
        return response.json().get("results", [])
    except Exception as e:
        return [{"error": f"Tavily API error: {str(e)}"}]
```

### Hot Reload for Development

```bash
# Mount code and run with --reload
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Direct Database Access

```bash
# From host machine
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT task_id, status, current_step FROM tasks ORDER BY created_at DESC LIMIT 10;
```

## Advanced Usage

### Custom Executor Step

```python
# src/planning_agent.py
def executor_agent_step(step_description: str, context: dict, model: str) -> str:
    """Execute a single planned step with context"""
    client = Client()
    
    messages = [
        {"role": "system", "content": "You are executing a research step..."},
        {"role": "user", "content": f"Step: {step_description}\n\nContext: {context}"}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Streaming Responses

```python
from fastapi.responses import StreamingResponse

@app.get("/stream_report/{task_id}")
async def stream_report(task_id: str):
    def generate():
        db = SessionLocal()
        task = db.query(Task).filter(Task.task_id == task_id).first()
        
        while task.status == "in_progress":
            yield f"data: {json.dumps({'step': task.current_step})}\n\n"
            time.sleep(1)
            db.refresh(task)
        
        yield f"data: {json.dumps({'status': task.status, 'report': task.report})}\n\n"
        db.close()
    
    return StreamingResponse(generate(), media_type="text/event-stream")
```
