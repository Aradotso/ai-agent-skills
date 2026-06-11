---
name: agentic-ai-research-agent
description: FastAPI-based reflective research agent with planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres state management
triggers:
  - how do I use the agentic AI research agent
  - set up the reflective research agent service
  - run a research task with planning agents
  - create a multi-step research workflow with FastAPI
  - integrate Tavily arXiv and Wikipedia research tools
  - track research agent task progress
  - deploy the agentic workflow research service
  - use the research agent API endpoints
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI web service that orchestrates multi-step research workflows using planning and execution agents. It leverages external research tools (Tavily for web search, arXiv for academic papers, Wikipedia for encyclopedic knowledge) and stores task state/results in Postgres. The system uses a reflective architecture with planner, research, writer, and editor agents working in coordination.

## Installation

### Prerequisites

- Docker installed
- `.env` file with required API keys:

```bash
# .env
OPENAI_API_KEY=your-openai-key-here
TAVILY_API_KEY=your-tavily-key-here
```

### Build and Run

```bash
# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the container (Postgres + FastAPI in one)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The container runs both Postgres (port 5432) and the FastAPI app (port 8000).

### Access Points

- UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs
- Postgres: `postgresql://app:local@localhost:5432/appdb`

## Project Structure

```
.
├── main.py                      # FastAPI application
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily, arxiv, wikipedia search tools
├── templates/
│   └── index.html               # UI template
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── .env
```

## API Endpoints

### POST /generate_report

Initiates a research task with multi-agent workflow.

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
print(f"Task ID: {task_id}")
```

**curl example:**

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Quantum computing applications in drug discovery",
    "model": "openai:gpt-4o"
  }'
```

### GET /task_progress/{task_id}

Poll live progress of each step/substep during execution.

```python
import requests
import time

task_id = "your-task-uuid"

while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    print(f"Status: {progress['status']}")
    print(f"Current step: {progress.get('current_step', 'N/A')}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(2)
```

### GET /task_status/{task_id}

Retrieve final status and generated report.

```python
import requests

task_id = "your-task-uuid"
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()

print(f"Status: {result['status']}")
print(f"Report:\n{result['report']}")
```

## Key Components

### Planning Agent

The planning agent breaks down research tasks into executable steps:

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(prompt: str, model: str):
    """
    Creates a research plan with multiple steps.
    Returns a list of tasks for the executor agents.
    """
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a research planning expert. Break down research tasks into clear, actionable steps."
        },
        {
            "role": "user",
            "content": f"Create a research plan for: {prompt}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Research Tools

#### Tavily Search

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5):
    """Web search using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    
    return response.json()
```

#### arXiv Search

```python
import requests
from xml.etree import ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5):
    """Search academic papers on arXiv."""
    base_url = "http://export.arxiv.org/api/query"
    
    response = requests.get(
        base_url,
        params={
            "search_query": f"all:{query}",
            "max_results": max_results
        }
    )
    
    root = ET.fromstring(response.content)
    papers = []
    
    for entry in root.findall("{http://www.w3.org/2005/Atom}entry"):
        title = entry.find("{http://www.w3.org/2005/Atom}title").text
        summary = entry.find("{http://www.w3.org/2005/Atom}summary").text
        papers.append({"title": title, "summary": summary})
    
    return papers
```

#### Wikipedia Search

```python
import wikipedia

def wikipedia_search_tool(query: str, sentences: int = 5):
    """Search Wikipedia for background information."""
    try:
        summary = wikipedia.summary(query, sentences=sentences)
        return {"query": query, "summary": summary}
    except wikipedia.exceptions.DisambiguationError as e:
        # Handle disambiguation by picking first option
        return wikipedia_search_tool(e.options[0], sentences)
    except wikipedia.exceptions.PageError:
        return {"query": query, "summary": "No results found"}
```

### Agent Workflow

```python
# src/agents.py
from aisuite import Client

def research_agent(task: str, tools: list, model: str):
    """Execute research using available tools."""
    client = Client()
    
    # Use tools to gather information
    research_data = []
    for tool in tools:
        if tool == "tavily":
            research_data.append(tavily_search_tool(task))
        elif tool == "arxiv":
            research_data.append(arxiv_search_tool(task))
        elif tool == "wikipedia":
            research_data.append(wikipedia_search_tool(task))
    
    # Synthesize findings
    messages = [
        {
            "role": "system",
            "content": "You are a research analyst. Synthesize the following research data."
        },
        {
            "role": "user",
            "content": f"Task: {task}\n\nData: {research_data}"
        }
    ]
    
    response = client.chat.completions.create(model=model, messages=messages)
    return response.choices[0].message.content

def writer_agent(research_results: str, model: str):
    """Write a coherent report from research findings."""
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a technical writer. Create a well-structured report."
        },
        {
            "role": "user",
            "content": f"Write a report based on:\n{research_results}"
        }
    ]
    
    response = client.chat.completions.create(model=model, messages=messages)
    return response.choices[0].message.content

def editor_agent(draft: str, model: str):
    """Review and improve the draft report."""
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are an editor. Improve clarity, accuracy, and coherence."
        },
        {
            "role": "user",
            "content": f"Edit this draft:\n{draft}"
        }
    ]
    
    response = client.chat.completions.create(model=model, messages=messages)
    return response.choices[0].message.content
```

## Database Schema

The service uses SQLAlchemy with Postgres:

```python
# main.py or models.py
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Connect to database
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...                    # OpenAI API key
TAVILY_API_KEY=tvly-...                  # Tavily search API key

# Optional (defaults set by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0                    # Set to 1 to drop tables on startup
```

### Custom Models

The service supports different AI models via `aisuite`:

```python
# Use different models
models = [
    "openai:gpt-4o",
    "openai:gpt-4-turbo",
    "openai:gpt-3.5-turbo"
]

for model in models:
    response = requests.post(
        "http://localhost:8000/generate_report",
        json={"prompt": "AI safety research", "model": model}
    )
```

## Common Patterns

### Async Research Workflow

```python
from fastapi import FastAPI, BackgroundTasks
from uuid import uuid4

app = FastAPI()

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background task for multi-step research."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    try:
        # Step 1: Planning
        task.status = "running"
        task.current_step = "planning"
        db.commit()
        
        plan = planner_agent(prompt, model)
        
        # Step 2: Research
        task.current_step = "researching"
        db.commit()
        
        research_results = research_agent(plan, ["tavily", "arxiv", "wikipedia"], model)
        
        # Step 3: Writing
        task.current_step = "writing"
        db.commit()
        
        draft = writer_agent(research_results, model)
        
        # Step 4: Editing
        task.current_step = "editing"
        db.commit()
        
        final_report = editor_agent(draft, model)
        
        # Complete
        task.status = "completed"
        task.report = final_report
        task.current_step = "done"
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = str(e)
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
async def generate_report(prompt: str, model: str, background_tasks: BackgroundTasks):
    task_id = str(uuid4())
    
    db = SessionLocal()
    task = Task(id=task_id, prompt=prompt, model=model)
    db.add(task)
    db.commit()
    db.close()
    
    background_tasks.add_task(run_research_workflow, task_id, prompt, model)
    
    return {"task_id": task_id}
```

### Complete FastAPI Application

```python
# main.py
from fastapi import FastAPI, BackgroundTasks, Request
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel
import os

app = FastAPI()
templates = Jinja2Templates(directory="templates")

# Mount static files if they exist
if os.path.exists("static"):
    app.mount("/static", StaticFiles(directory="static"), name="static")

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.get("/")
async def home(request: Request):
    """Render the UI."""
    return templates.TemplateResponse("index.html", {"request": request})

@app.post("/generate_report")
async def generate_report(req: ResearchRequest, background_tasks: BackgroundTasks):
    """Start a research task."""
    task_id = str(uuid4())
    
    db = SessionLocal()
    task = Task(id=task_id, prompt=req.prompt, model=req.model)
    db.add(task)
    db.commit()
    db.close()
    
    background_tasks.add_task(run_research_workflow, task_id, req.prompt, req.model)
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """Get live progress updates."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.id,
        "status": task.status,
        "current_step": task.current_step
    }

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get final task result."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.id,
        "status": task.status,
        "report": task.report,
        "created_at": task.created_at,
        "updated_at": task.updated_at
    }
```

## Troubleshooting

### Container Won't Start

**Symptom:** Container exits immediately or shows Postgres errors.

**Solution:**
```bash
# Check logs
docker logs fpsvc

# Verify entrypoint permissions
docker run --rm -it --entrypoint bash fastapi-postgres-service -c "ls -l /docker/entrypoint.sh"

# Ensure Postgres is properly configured
docker exec -it fpsvc su -s /bin/bash postgres -c "psql -l"
```

### Database Connection Fails

**Symptom:** `DATABASE_URL not set` or connection refused.

**Solution:**
```python
# Verify DATABASE_URL
import os
print(os.getenv("DATABASE_URL"))

# Test connection
from sqlalchemy import create_engine
engine = create_engine(os.getenv("DATABASE_URL"))
with engine.connect() as conn:
    result = conn.execute("SELECT 1")
    print(result.fetchone())
```

### API Keys Not Working

**Symptom:** 401/403 errors from Tavily or OpenAI.

**Solution:**
```bash
# Verify .env file is loaded
docker exec -it fpsvc env | grep API_KEY

# Re-run with explicit env vars
docker run --rm -it \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  -p 8000:8000 -p 5432:5432 \
  fastapi-postgres-service
```

### Templates Not Found

**Symptom:** `TemplateNotFound` error when accessing `/`.

**Solution:**
```dockerfile
# Verify Dockerfile copies templates
COPY templates/ /app/templates/
COPY static/ /app/static/

# Check in running container
docker exec -it fpsvc ls -R /app/templates
```

### Task Stays in "running" Forever

**Symptom:** Progress endpoint shows task stuck in one step.

**Solution:**
```python
# Add error handling and logging
import logging
logging.basicConfig(level=logging.INFO)

def run_research_workflow(task_id: str, prompt: str, model: str):
    logger = logging.getLogger(__name__)
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    try:
        logger.info(f"Starting task {task_id}")
        # ... workflow steps ...
    except Exception as e:
        logger.error(f"Task {task_id} failed: {str(e)}", exc_info=True)
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()
```

### Rate Limiting Issues

**Symptom:** Wikipedia or arXiv returns errors intermittently.

**Solution:**
```python
import time
from functools import wraps

def retry_with_backoff(max_retries=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    time.sleep(delay * (2 ** attempt))
            return None
        return wrapper
    return decorator

@retry_with_backoff()
def wikipedia_search_tool(query: str, sentences: int = 5):
    return wikipedia.summary(query, sentences=sentences)
```

### Hot Reload for Development

```bash
# Mount source code and enable auto-reload
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Query Database Directly

```bash
# From host
psql "postgresql://app:local@localhost:5432/appdb"

# From container
docker exec -it fpsvc su -s /bin/bash postgres -c "psql -d appdb -c 'SELECT * FROM tasks;'"
```
