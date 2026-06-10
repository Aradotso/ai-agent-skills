---
name: agentic-ai-research-agent
description: FastAPI research agent service with multi-step planning, Tavily/arXiv/Wikipedia tools, and PostgreSQL task tracking
triggers:
  - set up the agentic AI research agent
  - create a research workflow with planning agents
  - integrate Tavily and arXiv search tools
  - build a reflective research agent with FastAPI
  - track multi-step agent tasks in PostgreSQL
  - implement research planner and executor agents
  - deploy the DeepLearning.AI research agent service
  - configure the agentic workflow research API
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI web service that orchestrates multi-step research workflows using AI agents. It features a planning agent that breaks down research queries into subtasks, executor agents that perform research using tools (Tavily, arXiv, Wikipedia), and writers/editors that synthesize findings. All task state and results are persisted in PostgreSQL, with real-time progress tracking.

**Key capabilities:**
- Multi-agent workflow: planner → researcher → writer → editor
- Research tools: Tavily web search, arXiv papers, Wikipedia
- Async task execution with live progress updates
- PostgreSQL-backed task state management
- Web UI and REST API for task submission/tracking

## Installation

### Prerequisites

- Docker (for containerized deployment)
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Docker Build & Run

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

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

The container runs both PostgreSQL and the FastAPI service. On startup, it:
1. Initializes PostgreSQL cluster
2. Creates the `app` user and `appdb` database
3. Runs SQLAlchemy migrations
4. Starts Uvicorn on port 8000

### Local Development (Python)

If developing outside Docker:

```bash
pip install -r requirements.txt

# Set DATABASE_URL manually
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="sk-..."
export TAVILY_API_KEY="tvly-..."

# Start PostgreSQL separately, then:
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                      # FastAPI app, routes, task management
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## API Reference

### Start a Research Task

**POST** `/generate_report`

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'

# Response:
# {"task_id": "550e8400-e29b-41d4-a716-446655440000"}
```

**Request Body:**
- `prompt` (string, required): Research query
- `model` (string, optional): AI model to use (default: `"openai:gpt-4o"`)

### Check Task Progress

**GET** `/task_progress/{task_id}`

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000

# Response:
# {
#   "task_id": "550e8400-...",
#   "status": "running",
#   "current_step": "research",
#   "steps": [
#     {"name": "planning", "status": "completed", "substeps": [...]},
#     {"name": "research", "status": "running", "substeps": [...]}
#   ]
# }
```

### Get Final Report

**GET** `/task_status/{task_id}`

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000

# Response:
# {
#   "task_id": "550e8400-...",
#   "status": "completed",
#   "report": "# Research Report\n\n## Introduction...",
#   "created_at": "2025-01-15T10:30:00Z",
#   "completed_at": "2025-01-15T10:35:42Z"
# }
```

### Web UI

Navigate to `http://localhost:8000/` to access the interactive UI for submitting queries and viewing results.

## Core Components

### 1. Planning Agent

The planning agent breaks down research queries into subtasks:

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(query: str, model: str = "openai:gpt-4o") -> dict:
    """
    Generate a research plan with subtasks.
    Returns: {"plan": "...", "subtasks": [...]}
    """
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a research planning agent. Break down queries into actionable subtasks."
        },
        {
            "role": "user",
            "content": f"Create a research plan for: {query}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    # Parse structured response
    return parse_plan(response.choices[0].message.content)
```

### 2. Research Tools

#### Tavily Web Search

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """Search the web using Tavily API."""
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
```

#### arXiv Academic Search

```python
import requests
import xml.etree.ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv for academic papers."""
    base_url = "http://export.arxiv.org/api/query"
    
    params = {
        "search_query": f"all:{query}",
        "max_results": max_results,
        "sortBy": "relevance"
    }
    
    response = requests.get(base_url, params=params)
    root = ET.fromstring(response.content)
    
    papers = []
    for entry in root.findall("{http://www.w3.org/2005/Atom}entry"):
        papers.append({
            "title": entry.find("{http://www.w3.org/2005/Atom}title").text,
            "summary": entry.find("{http://www.w3.org/2005/Atom}summary").text,
            "link": entry.find("{http://www.w3.org/2005/Atom}id").text
        })
    
    return papers
```

#### Wikipedia Search

```python
import wikipedia

def wikipedia_search_tool(query: str) -> dict:
    """Search Wikipedia for background information."""
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=3),
            "url": page.url,
            "content": page.content[:2000]  # First 2000 chars
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Handle disambiguation by picking first option
        page = wikipedia.page(e.options[0])
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.PageError:
        return {"error": "Page not found"}
```

### 3. Executor Agent

The executor agent runs each subtask using appropriate tools:

```python
# src/planning_agent.py
def executor_agent_step(subtask: dict, model: str = "openai:gpt-4o") -> dict:
    """
    Execute a single subtask using available tools.
    
    Args:
        subtask: {"id": 1, "action": "research", "query": "LLM architectures"}
        model: AI model identifier
    
    Returns:
        {"subtask_id": 1, "status": "completed", "result": "..."}
    """
    query = subtask.get("query", "")
    action = subtask.get("action", "research")
    
    # Select appropriate tool
    if action == "web_search":
        results = tavily_search_tool(query)
    elif action == "academic_search":
        results = arxiv_search_tool(query)
    elif action == "background_info":
        results = wikipedia_search_tool(query)
    else:
        results = {"error": "Unknown action"}
    
    # Synthesize results with LLM
    client = Client()
    synthesis = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "Synthesize research findings."},
            {"role": "user", "content": f"Summarize: {results}"}
        ]
    )
    
    return {
        "subtask_id": subtask["id"],
        "status": "completed",
        "result": synthesis.choices[0].message.content
    }
```

### 4. Task Management with SQLAlchemy

```python
# main.py
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class ResearchTask(Base):
    __tablename__ = "research_tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)

Base.metadata.create_all(bind=engine)
```

### 5. Threaded Task Execution

```python
# main.py
import threading
import uuid
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background thread for research workflow."""
    db = SessionLocal()
    task = db.query(ResearchTask).filter_by(task_id=task_id).first()
    
    try:
        # Step 1: Planning
        task.status = "running"
        task.current_step = "planning"
        db.commit()
        
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute subtasks
        task.current_step = "research"
        db.commit()
        
        results = []
        for subtask in plan["subtasks"]:
            result = executor_agent_step(subtask, model)
            results.append(result)
        
        # Step 3: Write report
        task.current_step = "writing"
        db.commit()
        
        report = writer_agent(results, prompt, model)
        
        # Step 4: Edit report
        task.current_step = "editing"
        db.commit()
        
        final_report = editor_agent(report, model)
        
        # Complete
        task.status = "completed"
        task.report = final_report
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
def generate_report(request: ResearchRequest):
    """Start a new research task."""
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    task = ResearchTask(
        task_id=task_id,
        prompt=request.prompt,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Start background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

## Configuration

### Environment Variables

| Variable | Required | Description | Default |
|----------|----------|-------------|---------|
| `OPENAI_API_KEY` | Yes | OpenAI API key | - |
| `TAVILY_API_KEY` | Yes | Tavily web search API key | - |
| `DATABASE_URL` | No | PostgreSQL connection string | `postgresql://app:local@127.0.0.1:5432/appdb` |
| `POSTGRES_USER` | No | PostgreSQL username | `app` |
| `POSTGRES_PASSWORD` | No | PostgreSQL password | `local` |
| `POSTGRES_DB` | No | PostgreSQL database name | `appdb` |

### Docker Environment Variables

Pass environment variables when running the container:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -e OPENAI_API_KEY="sk-..." \
  -e TAVILY_API_KEY="tvly-..." \
  -e DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb" \
  --name fpsvc \
  fastapi-postgres-service
```

Or use an environment file:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service
```

## Common Patterns

### Custom Agent Implementation

```python
# src/agents.py
from aisuite import Client

def research_agent(query: str, tools: list, model: str = "openai:gpt-4o") -> dict:
    """
    Research agent that uses multiple tools.
    
    Args:
        query: Research query
        tools: List of tool functions [tavily_search_tool, arxiv_search_tool, ...]
        model: AI model
    
    Returns:
        {"findings": [...], "summary": "..."}
    """
    client = Client()
    findings = []
    
    # Execute all tools
    for tool in tools:
        try:
            result = tool(query)
            findings.append(result)
        except Exception as e:
            findings.append({"error": str(e)})
    
    # Synthesize with LLM
    messages = [
        {
            "role": "system",
            "content": "You are a research synthesizer. Combine findings into a coherent summary."
        },
        {
            "role": "user",
            "content": f"Query: {query}\n\nFindings: {findings}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.5
    )
    
    return {
        "findings": findings,
        "summary": response.choices[0].message.content
    }
```

### Writer Agent

```python
def writer_agent(research_results: list, original_query: str, model: str = "openai:gpt-4o") -> str:
    """Generate a research report from findings."""
    client = Client()
    
    combined_findings = "\n\n".join([
        f"Subtask {r['subtask_id']}: {r['result']}"
        for r in research_results
    ])
    
    messages = [
        {
            "role": "system",
            "content": "You are a technical writer. Create comprehensive, well-structured research reports."
        },
        {
            "role": "user",
            "content": f"Original Query: {original_query}\n\nResearch Findings:\n{combined_findings}\n\nWrite a detailed report."
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

### Editor Agent

```python
def editor_agent(draft_report: str, model: str = "openai:gpt-4o") -> str:
    """Review and improve a draft report."""
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are an editor. Improve clarity, fix errors, and enhance structure."
        },
        {
            "role": "user",
            "content": f"Edit this report:\n\n{draft_report}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    return response.choices[0].message.content
```

### Database Query Helpers

```python
# main.py
def get_task(task_id: str) -> ResearchTask:
    """Retrieve a task from the database."""
    db = SessionLocal()
    task = db.query(ResearchTask).filter_by(task_id=task_id).first()
    db.close()
    
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    return task

@app.get("/task_status/{task_id}")
def get_task_status(task_id: str):
    """Get final task status and report."""
    task = get_task(task_id)
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "report": task.report,
        "created_at": task.created_at.isoformat(),
        "completed_at": task.completed_at.isoformat() if task.completed_at else None
    }
```

## Troubleshooting

### Container Won't Start

**Problem:** Container exits immediately or fails to start PostgreSQL.

**Solution:**
- Check logs: `docker logs fpsvc`
- Ensure port 5432 isn't already in use: `lsof -i :5432`
- Verify the entrypoint script is executable:
  ```bash
  docker exec -it fpsvc ls -l /docker/entrypoint.sh
  # Should show: -rwxr-xr-x
  ```

### Database Connection Errors

**Problem:** `sqlalchemy.exc.OperationalError: could not connect to server`

**Solution:**
- Verify `DATABASE_URL` format: `postgresql://user:password@host:port/database`
- Check if PostgreSQL is running inside container:
  ```bash
  docker exec -it fpsvc pg_isready -h 127.0.0.1 -p 5432
  ```
- Ensure the database and user were created:
  ```bash
  docker exec -it fpsvc psql -U app -d appdb -c '\dt'
  ```

### Missing API Keys

**Problem:** `TAVILY_API_KEY not set` or Tavily returns 401 errors.

**Solution:**
- Verify `.env` file exists and contains keys:
  ```bash
  cat .env | grep TAVILY_API_KEY
  ```
- Check if environment variables are passed to container:
  ```bash
  docker exec -it fpsvc env | grep API_KEY
  ```
- Restart container with `--env-file`:
  ```bash
  docker stop fpsvc
  docker run --rm -it -p 8000:8000 --env-file .env --name fpsvc fastapi-postgres-service
  ```

### Tables Not Created

**Problem:** `relation "research_tasks" does not exist`

**Solution:**
- Check if `Base.metadata.create_all()` is called in `main.py`
- Manually create tables:
  ```python
  from main import Base, engine
  Base.metadata.create_all(bind=engine)
  ```
- Or reset the database (WARNING: deletes all data):
  ```python
  Base.metadata.drop_all(bind=engine)
  Base.metadata.create_all(bind=engine)
  ```

### Task Stuck in "running" State

**Problem:** Task shows `"status": "running"` indefinitely.

**Solution:**
- Check for thread exceptions in logs: `docker logs fpsvc | grep -i error`
- Manually inspect task in database:
  ```bash
  docker exec -it fpsvc psql -U app -d appdb -c \
    "SELECT task_id, status, current_step, created_at FROM research_tasks WHERE status='running';"
  ```
- Add exception handling to `run_research_workflow()` to catch and log errors
- Consider adding a timeout mechanism:
  ```python
  import signal
  
  def timeout_handler(signum, frame):
      raise TimeoutError("Task execution timed out")
  
  signal.signal(signal.SIGALRM, timeout_handler)
  signal.alarm(600)  # 10-minute timeout
  ```

### Wikipedia Rate Limiting

**Problem:** `wikipedia.exceptions.HTTPTimeoutError` or 429 errors.

**Solution:**
- Add retry logic with exponential backoff:
  ```python
  import time
  from wikipedia.exceptions import HTTPTimeoutError
  
  def wikipedia_search_tool_with_retry(query: str, retries: int = 3) -> dict:
      for attempt in range(retries):
          try:
              return wikipedia_search_tool(query)
          except HTTPTimeoutError:
              if attempt < retries - 1:
                  time.sleep(2 ** attempt)  # Exponential backoff
              else:
                  return {"error": "Wikipedia timeout after retries"}
  ```

### Templates Not Found

**Problem:** `TemplateNotFound: index.html`

**Solution:**
- Verify templates directory exists in container:
  ```bash
  docker exec -it fpsvc ls -la /app/templates/
  ```
- Check Dockerfile copies templates:
  ```dockerfile
  COPY templates/ /app/templates/
  COPY static/ /app/static/
  ```
- Ensure FastAPI Jinja2Templates points to correct path:
  ```python
  from fastapi.templating import Jinja2Templates
  templates = Jinja2Templates(directory="templates")
  ```

### Hot Reload During Development

**Problem:** Need to rebuild Docker image for every code change.

**Solution:**
- Mount source code as volume:
  ```bash
  docker run --rm -it \
    -p 8000:8000 \
    -v "$(pwd)":/app \
    --env-file .env \
    --name fpsvc \
    fastapi-postgres-service \
    bash -lc "uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
  ```

### Direct PostgreSQL Access

Connect to the database from your host machine:

```bash
# Using psql
psql "postgresql://app:local@localhost:5432/appdb"

# Or from within the container
docker exec -it fpsvc psql -U app -d appdb

# View all tasks
appdb=# SELECT task_id, status, current_step FROM research_tasks;

# Clear failed tasks
appdb=# DELETE FROM research_tasks WHERE status='failed';
```

## Advanced Usage

### Custom Tool Integration

Add a new research tool:

```python
# src/research_tools.py
import os
import requests

def custom_api_search_tool(query: str) -> dict:
    """Search a custom API endpoint."""
    api_key = os.getenv("CUSTOM_API_KEY")
    api_url = os.getenv("CUSTOM_API_URL", "https://api.example.com/search")
    
    response = requests.post(
        api_url,
        headers={"Authorization": f"Bearer {api_key}"},
        json={"query": query, "limit": 10}
    )
    
    return response.json()

# Register in planning agent
# src/planning_agent.py
AVAILABLE_TOOLS = {
    "web_search": tavily_search_tool,
    "academic_search": arxiv_search_tool,
    "wiki_search": wikipedia_search_tool,
    "custom_search": custom_api_search_tool
}
```

### Streaming Progress Updates

Implement WebSocket for real-time updates:

```python
# main.py
from fastapi import WebSocket

@app.websocket("/ws/task/{task_id}")
async def websocket_task_updates(websocket: WebSocket, task_id: str):
    await websocket.accept()
    
    while True:
        task = get_task(task_id)
        
        await websocket.send_json({
            "status": task.status,
            "current_step": task.current_step,
            "timestamp": datetime.utcnow().isoformat()
        })
        
        if task.status in ["completed", "failed"]:
            break
        
        await asyncio.sleep(1)  # Poll every second
    
    await websocket.close()
```

### Multi-Model Support

Switch between different AI providers:

```python
# src/planning_agent.py
MODEL_CONFIGS = {
    "openai:gpt-4o": {"provider": "openai", "temperature": 0.7},
    "openai:gpt-3.5-turbo": {"provider": "openai", "temperature": 0.7},
    "anthropic:claude-3-opus": {"provider": "anthropic", "temperature": 0.7}
}

def planner_agent(query: str, model: str = "openai:gpt-4o") -> dict:
    config = MODEL_CONFIGS.get(model, MODEL_CONFIGS["openai:gpt-4o"])
    client = Client()
    
    response = client.chat.completions.create(
        model=model,
        messages=[...],
        temperature=config["temperature"]
    )
    
    return parse_plan(response.choices[0].message.content)
```
