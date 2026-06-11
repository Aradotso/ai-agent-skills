---
name: deeplearning-ai-agentic-research-agent
description: FastAPI-based research agent service that orchestrates multi-step AI workflows with Tavily, arXiv, and Wikipedia tools using Postgres for task state management
triggers:
  - set up the deeplearning.ai research agent service
  - create a research workflow with planning and execution agents
  - integrate Tavily arXiv and Wikipedia search tools
  - build a FastAPI research agent with Postgres
  - implement agentic workflow with task progress tracking
  - deploy the reflective research agent container
  - use the research agent API to generate reports
  - configure multi-step agent orchestration with planner
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic Research Agent is a FastAPI-based service that implements a reflective, multi-step research workflow. It uses a planner agent to break down research tasks into steps, then executes research/writer/editor agents using external tools (Tavily web search, arXiv papers, Wikipedia). Task state and results are stored in Postgres, with real-time progress tracking.

**Key Features:**
- Multi-agent workflow orchestration (planner → researcher → writer → editor)
- Integration with Tavily, arXiv, and Wikipedia APIs
- Postgres-backed task state management
- Real-time progress tracking via REST API
- Docker deployment with Postgres in a single container
- Web UI for task submission and monitoring

## Installation

### Prerequisites

1. Docker installed
2. `.env` file with required API keys:

```bash
# .env
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
```

### Build and Run

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build Docker image
docker build -t fastapi-postgres-service .

# Run container (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The entrypoint script automatically:
- Starts Postgres cluster
- Creates database and user
- Sets `DATABASE_URL`
- Launches Uvicorn server

## Project Structure

```
agentic-ai-public/
├── main.py                    # FastAPI app with endpoints
├── src/
│   ├── planning_agent.py      # Planner and executor logic
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
└── Dockerfile
```

## API Reference

### Start Research Task

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
```

### Check Progress

```python
progress = requests.get(f"http://localhost:8000/task_progress/{task_id}")
print(progress.json())
# {
#   "task_id": "...",
#   "status": "in_progress",
#   "current_step": 2,
#   "total_steps": 4,
#   "steps": [...]
# }
```

### Get Final Report

```python
result = requests.get(f"http://localhost:8000/task_status/{task_id}")
report = result.json()["report"]
```

## Core Components

### 1. Planning Agent

The planner breaks research tasks into executable steps:

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(prompt: str, model: str = "openai:gpt-4o"):
    """
    Generate a plan with steps for the research task.
    Returns a list of step dictionaries.
    """
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a research planner. Break the task into steps."
        },
        {
            "role": "user",
            "content": f"Plan research steps for: {prompt}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    # Parse response into step structure
    return parse_plan(response.choices[0].message.content)


def executor_agent_step(step: dict, context: dict, model: str):
    """
    Execute a single step using appropriate agent/tool.
    """
    if step["agent"] == "research":
        return research_agent(step, context, model)
    elif step["agent"] == "writer":
        return writer_agent(step, context, model)
    elif step["agent"] == "editor":
        return editor_agent(step, context, model)
```

### 2. Research Tools

```python
# src/research_tools.py
import os
import requests
import wikipedia

def tavily_search_tool(query: str, max_results: int = 5):
    """
    Search the web using Tavily API.
    """
    api_key = os.getenv("TAVILY_API_KEY")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    
    return response.json()["results"]


def arxiv_search_tool(query: str, max_results: int = 5):
    """
    Search arXiv for academic papers.
    """
    import arxiv
    
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for paper in search.results():
        results.append({
            "title": paper.title,
            "summary": paper.summary,
            "authors": [a.name for a in paper.authors],
            "pdf_url": paper.pdf_url
        })
    
    return results


def wikipedia_search_tool(query: str):
    """
    Search Wikipedia and return summary.
    """
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=5),
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option
        return wikipedia_search_tool(e.options[0])
    except wikipedia.exceptions.PageError:
        return {"error": "Page not found"}
```

### 3. Agent Definitions

```python
# src/agents.py
from aisuite import Client
from research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

def research_agent(step: dict, context: dict, model: str):
    """
    Research agent that uses tools to gather information.
    """
    query = step["query"]
    
    # Gather from multiple sources
    web_results = tavily_search_tool(query)
    arxiv_results = arxiv_search_tool(query)
    wiki_results = wikipedia_search_tool(query)
    
    # Synthesize findings
    client = Client()
    messages = [
        {
            "role": "system",
            "content": "Synthesize research findings into coherent insights."
        },
        {
            "role": "user",
            "content": f"Query: {query}\n\nWeb: {web_results}\n\nArXiv: {arxiv_results}\n\nWiki: {wiki_results}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content


def writer_agent(step: dict, context: dict, model: str):
    """
    Writer agent that drafts content from research.
    """
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a technical writer. Draft clear, well-structured content."
        },
        {
            "role": "user",
            "content": f"Write section on: {step['topic']}\n\nResearch: {context['research']}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content


def editor_agent(step: dict, context: dict, model: str):
    """
    Editor agent that refines and improves draft.
    """
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are an editor. Improve clarity, flow, and accuracy."
        },
        {
            "role": "user",
            "content": f"Edit this draft:\n\n{context['draft']}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

### 4. FastAPI Endpoints

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
import uuid
import threading
from datetime import datetime

app = FastAPI()
templates = Jinja2Templates(directory="templates")

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()


class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text)
    status = Column(String)  # pending, in_progress, completed, failed
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow)


Base.metadata.create_all(bind=engine)


class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"


@app.get("/", response_class=HTMLResponse)
async def index():
    return templates.TemplateResponse("index.html", {"request": {}})


@app.post("/generate_report")
async def generate_report(req: ResearchRequest, background_tasks: BackgroundTasks):
    """
    Start a research task in the background.
    """
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    task = Task(id=task_id, prompt=req.prompt, status="pending")
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background thread
    thread = threading.Thread(
        target=run_workflow,
        args=(task_id, req.prompt, req.model)
    )
    thread.start()
    
    return {"task_id": task_id}


@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """
    Get current progress of a task.
    """
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task_id,
        "status": task.status,
        "updated_at": task.updated_at
    }


@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """
    Get final status and report.
    """
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task_id,
        "status": task.status,
        "report": task.report,
        "created_at": task.created_at,
        "updated_at": task.updated_at
    }


def run_workflow(task_id: str, prompt: str, model: str):
    """
    Execute the full agentic workflow.
    """
    from src.planning_agent import planner_agent, executor_agent_step
    
    db = SessionLocal()
    
    try:
        # Update status
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "in_progress"
        db.commit()
        
        # Generate plan
        plan = planner_agent(prompt, model)
        
        # Execute steps
        context = {}
        for step in plan:
            result = executor_agent_step(step, context, model)
            context[step["name"]] = result
        
        # Final report is in context
        final_report = context.get("final_report", "")
        
        # Update task
        task.status = "completed"
        task.report = final_report
        task.updated_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    
    finally:
        db.close()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...

# Optional (defaults set by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0  # Set to 1 to drop tables on startup
```

### Database Connection from Host

```bash
# Connect to Postgres running in container
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT id, status, created_at FROM tasks ORDER BY created_at DESC;
```

## Usage Patterns

### Complete Research Workflow

```python
import requests
import time

# Start task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Impact of transformers on natural language processing",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]

# Poll until complete
while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    status = progress.json()["status"]
    
    if status == "completed":
        result = requests.get(f"http://localhost:8000/task_status/{task_id}")
        print(result.json()["report"])
        break
    elif status == "failed":
        print("Task failed")
        break
    
    time.sleep(5)
```

### Custom Tool Integration

```python
# Add custom tool to research_tools.py
def custom_api_tool(query: str):
    """
    Query your custom API.
    """
    api_key = os.getenv("CUSTOM_API_KEY")
    
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    
    return response.json()

# Use in research agent
from research_tools import custom_api_tool

def research_agent(step: dict, context: dict, model: str):
    # ... existing code ...
    custom_results = custom_api_tool(query)
    # Incorporate into synthesis
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs fpsvc

# Verify Postgres is running inside container
docker exec -it fpsvc bash -lc "pg_isready"

# Manually inspect database
docker exec -it fpsvc bash -lc "psql -U app -d appdb -c 'SELECT * FROM tasks;'"
```

### API Key Errors

```bash
# Verify environment variables are loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Check .env file is in correct location
ls -la .env

# Restart container with verbose logging
docker run --rm -it -p 8000:8000 --env-file .env fastapi-postgres-service
```

### Tasks Stuck in "pending"

```python
# Check if workflow thread started
import requests

task_id = "your-task-id"
response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
print(response.json())

# Check database directly
# docker exec -it fpsvc bash -lc "psql -U app -d appdb"
# SELECT * FROM tasks WHERE id = 'your-task-id';
```

### Rate Limiting Issues

```python
# Add retry logic for external APIs
import time
from functools import wraps

def retry_on_rate_limit(max_retries=3, delay=5):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except requests.exceptions.HTTPError as e:
                    if e.response.status_code == 429:
                        time.sleep(delay * (attempt + 1))
                    else:
                        raise
            return func(*args, **kwargs)
        return wrapper
    return decorator

@retry_on_rate_limit()
def tavily_search_tool(query: str):
    # ... implementation ...
```

### Development with Hot Reload

```bash
# Mount code directory and enable reload
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Additional Resources

- **API Documentation**: http://localhost:8000/docs (Swagger UI)
- **Course**: DeepLearning.AI Agentic Workflow course
- **Repository**: https://github.com/https-deeplearning-ai/agentic-ai-public
