---
name: agentic-ai-research-agent
description: FastAPI research agent with multi-step workflow planning, Tavily/arXiv/Wikipedia tools, and Postgres state management
triggers:
  - how do I set up the agentic ai research agent
  - build a research agent with planning and tool execution
  - create a multi-step research workflow with agents
  - implement reflective research agent with fastapi
  - set up tavily arxiv wikipedia research tools
  - deploy research agent with postgres state tracking
  - build agentic workflow with planner and executor agents
  - configure research agent with openai and tavily
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

This is a FastAPI-based research agent system that implements a multi-step agentic workflow with planning, research, writing, and editing capabilities. The system uses:

- **Planning Agent**: Breaks down research tasks into executable steps
- **Research Tools**: Tavily search, arXiv papers, Wikipedia articles
- **Executor Agents**: Research, writer, and editor agents that work in sequence
- **State Management**: Postgres database tracking task progress and results
- **Live Progress**: Real-time status updates for long-running workflows

The architecture runs Postgres and the API in a single Docker container for simplified deployment.

## Installation

### Prerequisites

1. Docker installed
2. API keys for OpenAI and Tavily
3. `.env` file in project root:

```bash
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
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

The container's entrypoint script automatically:
- Starts Postgres cluster
- Creates `app` user with password `local`
- Creates `appdb` database
- Exports `DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb`
- Launches Uvicorn on port 8000

## Project Structure

```
.
├── main.py                    # FastAPI app with endpoints
├── src/
│   ├── planning_agent.py      # planner_agent(), executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # Tool implementations
├── templates/
│   └── index.html             # Web UI
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── Dockerfile
└── requirements.txt
```

## Core API Endpoints

### 1. Generate Report (Start Research Workflow)

```bash
# Start a research task
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'

# Response:
# {"task_id": "550e8400-e29b-41d4-a716-446655440000"}
```

### 2. Poll Task Progress

```bash
# Get live progress updates
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000

# Response:
# {
#   "task_id": "550e8400-e29b-41d4-a716-446655440000",
#   "status": "running",
#   "current_step": "research",
#   "progress": {
#     "planning": "completed",
#     "research": "in_progress",
#     "writing": "pending",
#     "editing": "pending"
#   }
# }
```

### 3. Get Final Status and Report

```bash
# Retrieve completed report
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000

# Response:
# {
#   "task_id": "550e8400-e29b-41d4-a716-446655440000",
#   "status": "completed",
#   "report": "# Research Report\n\n## Executive Summary\n...",
#   "completed_at": "2025-01-15T10:30:00Z"
# }
```

### 4. Web UI

Access the web interface at `http://localhost:8000/` to:
- Submit research prompts via form
- Select AI model
- View task progress in browser
- Download completed reports

## Python Usage Patterns

### Creating a Custom Research Tool

```python
# src/research_tools.py
import requests
import os

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """Search the web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        raise ValueError("TAVILY_API_KEY not set")
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results,
        "search_depth": "advanced"
    }
    
    response = requests.post(url, json=payload)
    response.raise_for_status()
    
    results = response.json().get("results", [])
    return [
        {
            "title": r.get("title"),
            "url": r.get("url"),
            "content": r.get("content")
        }
        for r in results
    ]


def arxiv_search_tool(query: str, max_results: int = 3) -> list:
    """Search arXiv for academic papers."""
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
            "authors": [a.name for a in paper.authors],
            "summary": paper.summary,
            "pdf_url": paper.pdf_url,
            "published": paper.published.isoformat()
        })
    
    return results


def wikipedia_search_tool(query: str) -> dict:
    """Fetch Wikipedia article summary."""
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option from disambiguation
        page = wikipedia.page(e.options[0])
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.PageError:
        return {"error": f"No Wikipedia page found for: {query}"}
```

### Implementing a Planning Agent

```python
# src/planning_agent.py
import aisuite as ai
import os

client = ai.Client()

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> list:
    """
    Generate a research plan broken into steps.
    Returns list of steps with tool assignments.
    """
    system_prompt = """You are a research planner. Break down the research task into 3-5 steps.
    For each step, specify:
    - step_number: int
    - description: what to do
    - tool: one of [tavily_search, arxiv_search, wikipedia_search]
    - query: search query to use
    
    Return as JSON array."""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": f"Research task: {prompt}"}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    import json
    plan = json.loads(response.choices[0].message.content)
    return plan


def executor_agent_step(step: dict, tools: dict) -> dict:
    """
    Execute a single research step using specified tool.
    """
    tool_name = step["tool"]
    query = step["query"]
    
    if tool_name not in tools:
        return {"error": f"Unknown tool: {tool_name}"}
    
    tool_func = tools[tool_name]
    results = tool_func(query)
    
    return {
        "step_number": step["step_number"],
        "description": step["description"],
        "results": results
    }
```

### Implementing Agent Workflow

```python
# src/agents.py
import aisuite as ai

client = ai.Client()

def research_agent(plan_results: list, model: str = "openai:gpt-4o") -> str:
    """Synthesize research findings into a coherent summary."""
    
    findings = "\n\n".join([
        f"Step {r['step_number']}: {r['description']}\n{r['results']}"
        for r in plan_results
    ])
    
    messages = [
        {
            "role": "system",
            "content": "You are a research analyst. Synthesize the findings into a comprehensive research summary."
        },
        {
            "role": "user",
            "content": f"Research findings:\n\n{findings}\n\nProvide a detailed synthesis."
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content


def writer_agent(research_summary: str, original_prompt: str, model: str = "openai:gpt-4o") -> str:
    """Write a polished report from research summary."""
    
    messages = [
        {
            "role": "system",
            "content": "You are a technical writer. Create a well-structured report with sections, headings, and clear prose."
        },
        {
            "role": "user",
            "content": f"Topic: {original_prompt}\n\nResearch:\n{research_summary}\n\nWrite a comprehensive report."
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.8
    )
    
    return response.choices[0].message.content


def editor_agent(draft_report: str, model: str = "openai:gpt-4o") -> str:
    """Review and refine the report for clarity and accuracy."""
    
    messages = [
        {
            "role": "system",
            "content": "You are an editor. Improve clarity, fix errors, and enhance readability without changing meaning."
        },
        {
            "role": "user",
            "content": f"Draft report:\n\n{draft_report}\n\nProvide edited version."
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.5
    )
    
    return response.choices[0].message.content
```

### Database Models with SQLAlchemy

```python
# main.py (database setup)
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
import datetime

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()


class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.datetime.utcnow, onupdate=datetime.datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)


# Create tables
Base.metadata.create_all(bind=engine)


# Database helper functions
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


def create_task(db, task_id: str, prompt: str, model: str):
    task = Task(task_id=task_id, prompt=prompt, model=model)
    db.add(task)
    db.commit()
    return task


def update_task_status(db, task_id: str, status: str, current_step: str = None):
    task = db.query(Task).filter(Task.task_id == task_id).first()
    if task:
        task.status = status
        if current_step:
            task.current_step = current_step
        task.updated_at = datetime.datetime.utcnow()
        db.commit()
    return task


def save_task_report(db, task_id: str, report: str):
    task = db.query(Task).filter(Task.task_id == task_id).first()
    if task:
        task.report = report
        task.status = "completed"
        task.completed_at = datetime.datetime.utcnow()
        db.commit()
    return task
```

### Complete FastAPI Workflow Implementation

```python
# main.py (endpoints)
from fastapi import FastAPI, BackgroundTasks, Depends
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
from pydantic import BaseModel
from sqlalchemy.orm import Session
import uuid
import threading

app = FastAPI()
templates = Jinja2Templates(directory="templates")
app.mount("/static", StaticFiles(directory="static"), name="static")


class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"


@app.get("/", response_class=HTMLResponse)
async def home(request: Request):
    """Render web UI."""
    return templates.TemplateResponse("index.html", {"request": request})


@app.post("/generate_report")
async def generate_report(req: ResearchRequest, db: Session = Depends(get_db)):
    """Start a research workflow in background thread."""
    task_id = str(uuid.uuid4())
    
    # Create task record
    create_task(db, task_id, req.prompt, req.model)
    
    # Start background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, req.prompt, req.model)
    )
    thread.start()
    
    return {"task_id": task_id}


@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str, db: Session = Depends(get_db)):
    """Get current task progress."""
    task = db.query(Task).filter(Task.task_id == task_id).first()
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task_id,
        "status": task.status,
        "current_step": task.current_step,
        "updated_at": task.updated_at.isoformat()
    }


@app.get("/task_status/{task_id}")
async def task_status(task_id: str, db: Session = Depends(get_db)):
    """Get final task status and report."""
    task = db.query(Task).filter(Task.task_id == task_id).first()
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task_id,
        "status": task.status,
        "report": task.report,
        "completed_at": task.completed_at.isoformat() if task.completed_at else None
    }


def run_research_workflow(task_id: str, prompt: str, model: str):
    """Execute the full research workflow."""
    from src.planning_agent import planner_agent, executor_agent_step
    from src.agents import research_agent, writer_agent, editor_agent
    from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    
    db = SessionLocal()
    
    try:
        # Step 1: Planning
        update_task_status(db, task_id, "running", "planning")
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute research steps
        update_task_status(db, task_id, "running", "research")
        tools = {
            "tavily_search": tavily_search_tool,
            "arxiv_search": arxiv_search_tool,
            "wikipedia_search": wikipedia_search_tool
        }
        
        plan_results = []
        for step in plan:
            result = executor_agent_step(step, tools)
            plan_results.append(result)
        
        # Step 3: Synthesize research
        research_summary = research_agent(plan_results, model)
        
        # Step 4: Write report
        update_task_status(db, task_id, "running", "writing")
        draft = writer_agent(research_summary, prompt, model)
        
        # Step 5: Edit report
        update_task_status(db, task_id, "running", "editing")
        final_report = editor_agent(draft, model)
        
        # Save final report
        save_task_report(db, task_id, final_report)
        
    except Exception as e:
        update_task_status(db, task_id, "failed", f"error: {str(e)}")
    finally:
        db.close()
```

## Configuration

### Environment Variables

Required:
```bash
OPENAI_API_KEY=sk-...           # OpenAI API key
TAVILY_API_KEY=tvly-...         # Tavily search API key
```

Optional (auto-configured by entrypoint):
```bash
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb
```

### Dockerfile Configuration

```dockerfile
FROM postgres:17

# Install Python and dependencies
RUN apt-get update && apt-get install -y \
    python3.11 python3-pip python3-venv \
    postgresql-contrib

WORKDIR /app

COPY requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt

COPY . .

# Copy entrypoint script
COPY docker/entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh

EXPOSE 8000 5432

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
```

### Entrypoint Script

```bash
#!/bin/bash
# docker/entrypoint.sh

set -e

# Start Postgres
echo "🚀 Starting Postgres cluster..."
pg_ctlcluster $(psql -V | awk '{print $3}' | cut -d. -f1) main start

# Wait for Postgres
until su -s /bin/bash postgres -c "psql -c 'SELECT 1' > /dev/null 2>&1"; do
    sleep 1
done
echo "✅ Postgres is ready"

# Create user and database
su -s /bin/bash postgres -c "psql -c \"CREATE ROLE ${POSTGRES_USER:-app} WITH LOGIN PASSWORD '${POSTGRES_PASSWORD:-local}';\""
su -s /bin/bash postgres -c "psql -c \"CREATE DATABASE ${POSTGRES_DB:-appdb} OWNER ${POSTGRES_USER:-app};\""

# Export DATABASE_URL
export DATABASE_URL="postgresql://${POSTGRES_USER:-app}:${POSTGRES_PASSWORD:-local}@127.0.0.1:5432/${POSTGRES_DB:-appdb}"
echo "🔗 DATABASE_URL=$DATABASE_URL"

# Start FastAPI
exec uvicorn main:app --host 0.0.0.0 --port 8000
```

## Common Patterns

### Add Custom Agent Step

```python
def fact_checker_agent(report: str, model: str = "openai:gpt-4o") -> dict:
    """Verify facts in the report."""
    
    messages = [
        {
            "role": "system",
            "content": "You are a fact-checker. Identify claims that need verification and rate confidence."
        },
        {
            "role": "user",
            "content": f"Report:\n{report}\n\nIdentify claims and confidence levels."
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    return {
        "verification": response.choices[0].message.content,
        "timestamp": datetime.datetime.utcnow().isoformat()
    }

# Add to workflow after writing
fact_check = fact_checker_agent(draft, model)
```

### Stream Progress Updates

```python
from fastapi.responses import StreamingResponse
import asyncio
import json

@app.get("/stream_progress/{task_id}")
async def stream_progress(task_id: str):
    """Server-sent events for real-time progress."""
    
    async def event_generator():
        db = SessionLocal()
        try:
            while True:
                task = db.query(Task).filter(Task.task_id == task_id).first()
                if not task:
                    break
                
                data = {
                    "status": task.status,
                    "current_step": task.current_step
                }
                yield f"data: {json.dumps(data)}\n\n"
                
                if task.status in ["completed", "failed"]:
                    break
                
                await asyncio.sleep(2)
        finally:
            db.close()
    
    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream"
    )
```

### Retry Failed Tasks

```python
@app.post("/retry_task/{task_id}")
async def retry_task(task_id: str, db: Session = Depends(get_db)):
    """Retry a failed task."""
    task = db.query(Task).filter(Task.task_id == task_id).first()
    if not task:
        return {"error": "Task not found"}
    
    if task.status != "failed":
        return {"error": "Only failed tasks can be retried"}
    
    # Reset task status
    task.status = "pending"
    task.current_step = None
    task.updated_at = datetime.datetime.utcnow()
    db.commit()
    
    # Restart workflow
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, task.prompt, task.model)
    )
    thread.start()
    
    return {"task_id": task_id, "status": "restarted"}
```

## Troubleshooting

### Postgres Connection Issues

**Problem**: `DATABASE_URL not set` or connection refused

**Solution**:
```bash
# Check Postgres is running in container
docker exec -it fpsvc bash -lc "pg_isready"

# Verify DATABASE_URL is exported
docker exec -it fpsvc bash -lc "echo \$DATABASE_URL"

# Test connection manually
docker exec -it fpsvc bash -lc "psql postgresql://app:local@127.0.0.1:5432/appdb -c 'SELECT 1'"
```

### Tables Not Created

**Problem**: SQLAlchemy tables don't exist

**Solution**:
```python
# Force table creation on startup (main.py)
from sqlalchemy import inspect

def init_database():
    inspector = inspect(engine)
    if not inspector.has_table("tasks"):
        Base.metadata.create_all(bind=engine)
        print("✅ Database tables created")

# Call in startup event
@app.on_event("startup")
async def startup_event():
    init_database()
```

### API Key Errors

**Problem**: `TAVILY_API_KEY not set` or invalid key errors

**Solution**:
```bash
# Verify env vars are loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Check .env file is mounted/copied
docker exec -it fpsvc bash -lc "cat .env"

# Explicitly pass env vars
docker run --rm -it \
  -p 8000:8000 \
  -e OPENAI_API_KEY=sk-... \
  -e TAVILY_API_KEY=tvly-... \
  fastapi-postgres-service
```

### Task Hangs or Never Completes

**Problem**: Task stuck in "running" status

**Solution**:
```python
# Add timeout to research workflow
import signal

class TimeoutError(Exception):
    pass

def timeout_handler(signum, frame):
    raise TimeoutError("Task exceeded time limit")

def run_research_workflow(task_id: str, prompt: str, model: str):
    # Set 10 minute timeout
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(600)
    
    db = SessionLocal()
    try:
        # ... workflow steps ...
    except TimeoutError:
        update_task_status(db, task_id, "failed", "timeout")
    finally:
        signal.alarm(0)  # Cancel alarm
        db.close()
```

### Hot Reload for Development

```bash
# Mount code and use --reload flag
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  fastapi-postgres-service \
  bash -lc "
    pg_ctlcluster 17 main start && \
    sleep 2 && \
    su -s /bin/bash postgres -c \"psql -c 'CREATE ROLE app WITH LOGIN PASSWORD 'local'; CREATE DATABASE appdb OWNER app;'\" || true && \
    export DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb && \
    uvicorn main:app --host 0.0.0.0 --port 8000 --reload
  "
```

### Check Agent Execution Logs

```python
# Add detailed logging to agents.py
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def research_agent(plan_results: list, model: str = "openai:gpt-4o") -> str:
    logger.info(f"Starting research synthesis with {len(plan_results)} findings")
    
    findings = "\n\n".join([
        f"Step {r['step_number']}: {r['description']}\n{r['results']}"
        for r in plan_results
    ])
    
    logger.debug(f"Findings length: {len(findings)} chars")
    
    # ... agent execution ...
    
    logger.info("Research synthesis completed")
    return response.choices[0].message.content
```

### Rate Limiting Handling

```python
# src/research_tools.py
import time
from functools import wraps

def rate_limit(calls_per_minute=10):
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            wait_time = min_interval - elapsed
            if wait_time > 0:
                time.sleep(wait_time)
            result = func(*args, **kwargs)
            last_called[0] = time.time()
            return result
        return wrapper
    return decorator

@rate_limit(calls_per_minute=5)
def wikipedia_search_tool(query: str) -> dict:
    # ... implementation ...
    pass
```

## Advanced Usage

### Multi-Model Support

```python
# Use different models for different agents
def run_research_workflow(task_id: str, prompt: str, model: str):
    # Use faster model for planning
    plan = planner_agent(prompt, "openai:gpt-3.5-turbo")
    
    # Use powerful model for research
    research_summary = research_agent(plan_results, "openai:gpt-4o")
    
    # Use specialized model for writing
    draft = writer_agent(research_summary, prompt, "anthropic:claude-3-opus-20240229")
```

### Batch Research Tasks

```python
@app.post("/batch_research")
async def batch_research(prompts: list[str], model: str = "openai:gpt-4o", db: Session = Depends(get_db)):
    """Start multiple research tasks."""
    task_ids = []
    
    for prompt in prompts:
        task_id = str(uuid.uuid4())
        create_task(db, task_id, prompt, model)
        
        thread = threading.Thread(
            target=run_research_workflow,
            args=(task_id, prompt, model)
        )
        thread.start()
        task_ids.append(task_id)
    
    return {"task_ids": task_ids, "count": len(task_ids)}
```
