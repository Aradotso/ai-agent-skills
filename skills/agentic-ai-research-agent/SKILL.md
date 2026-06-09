---
name: agentic-ai-research-agent
description: FastAPI-based reflective research agent with planning, tool use (Tavily/arXiv/Wikipedia), and Postgres task tracking
triggers:
  - build a research agent with planning and reflection
  - set up agentic AI workflow with FastAPI
  - create multi-step research agent with tool calling
  - implement reflective agent with Tavily and arXiv
  - deploy FastAPI research agent with Postgres
  - build agent workflow with task tracking
  - create research agent using OpenAI and tools
  - set up agentic research service with Docker
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI service that implements a multi-step, reflective research workflow. It uses a planning agent to decompose research tasks, then executes specialized agents (research, writer, editor) that leverage external tools (Tavily search, arXiv papers, Wikipedia). All task state and results are persisted in Postgres for progress tracking and final report retrieval.

**Key Features:**
- Multi-agent workflow with planning, research, writing, and editing phases
- Tool integration: Tavily web search, arXiv academic papers, Wikipedia
- Async task execution with real-time progress tracking
- Postgres-backed state persistence
- Single-container Docker deployment (Postgres + FastAPI)
- Web UI and REST API

## Installation

### Prerequisites

1. **Docker** installed on your system
2. **API keys** in a `.env` file at project root:

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

# Run container (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The container starts Postgres, creates the database schema, and launches the FastAPI app on `http://localhost:8000`.

## Project Structure

```
.
├── main.py                      # FastAPI app, routes, DB models
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tool implementations (Tavily, arXiv, Wikipedia)
├── templates/
│   └── index.html               # Web UI
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core API Endpoints

### POST /generate_report

Initiates a research task and returns a unique task ID.

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

### GET /task_progress/{task_id}

Returns live progress of the research workflow (steps and substeps).

```python
progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
print(f"Status: {progress['status']}")
for step in progress.get("steps", []):
    print(f"  {step['step_name']}: {step['status']}")
```

### GET /task_status/{task_id}

Returns final task status and generated report.

```python
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
if result["status"] == "completed":
    print(result["report"])
```

### GET /

Serves the web UI for interactive task submission.

## Configuration

### Environment Variables

**Required:**
- `OPENAI_API_KEY` - OpenAI API key for LLM calls
- `TAVILY_API_KEY` - Tavily API key for web search

**Optional (set by entrypoint, override if needed):**
- `DATABASE_URL` - Postgres connection string (default: `postgresql://app:local@127.0.0.1:5432/appdb`)
- `POSTGRES_USER` - Postgres username (default: `app`)
- `POSTGRES_PASSWORD` - Postgres password (default: `local`)
- `POSTGRES_DB` - Database name (default: `appdb`)

### Database Schema

The app uses SQLAlchemy with two main tables:

```python
# Task table (from main.py)
class Task(Base):
    __tablename__ = "tasks"
    id = Column(String, primary_key=True)  # UUID
    prompt = Column(Text)
    status = Column(String)  # pending, running, completed, failed
    report = Column(Text)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)

# TaskStep table (from main.py)
class TaskStep(Base):
    __tablename__ = "task_steps"
    id = Column(Integer, primary_key=True)
    task_id = Column(String, ForeignKey("tasks.id"))
    step_name = Column(String)
    status = Column(String)
    result = Column(Text)
```

## Agent Workflow

### Planning Agent

The planner decomposes research prompts into structured steps:

```python
# src/planning_agent.py
def planner_agent(prompt: str, model: str):
    """
    Takes a research prompt and returns a structured plan
    with steps like: research → draft → edit
    """
    system_msg = """You are a research planning agent.
    Break down the user's request into clear steps."""
    
    messages = [
        {"role": "system", "content": system_msg},
        {"role": "user", "content": prompt}
    ]
    
    # Returns plan structure with steps
    return parsed_plan
```

### Research Agent

Searches external sources using configured tools:

```python
# src/agents.py
def research_agent(query: str, tools: list):
    """
    Uses Tavily, arXiv, Wikipedia to gather information.
    Returns aggregated research findings.
    """
    results = []
    for tool in tools:
        if tool == "tavily":
            results.append(tavily_search_tool(query))
        elif tool == "arxiv":
            results.append(arxiv_search_tool(query))
        elif tool == "wikipedia":
            results.append(wikipedia_search_tool(query))
    return combine_results(results)
```

### Writer & Editor Agents

```python
# src/agents.py
def writer_agent(research_data: str, prompt: str):
    """Drafts a report from research findings."""
    return draft_report

def editor_agent(draft: str, feedback: str):
    """Refines and polishes the draft."""
    return final_report
```

## Tool Implementations

### Tavily Search Tool

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5):
    """Web search using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    url = "https://api.tavily.com/search"
    
    response = requests.post(
        url,
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    return response.json().get("results", [])
```

### arXiv Search Tool

```python
# src/research_tools.py
import requests
import xml.etree.ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5):
    """Search arXiv for academic papers."""
    url = f"http://export.arxiv.org/api/query?search_query=all:{query}&max_results={max_results}"
    
    response = requests.get(url)
    root = ET.fromstring(response.content)
    
    papers = []
    for entry in root.findall("{http://www.w3.org/2005/Atom}entry"):
        title = entry.find("{http://www.w3.org/2005/Atom}title").text
        summary = entry.find("{http://www.w3.org/2005/Atom}summary").text
        papers.append({"title": title, "summary": summary})
    
    return papers
```

### Wikipedia Search Tool

```python
# src/research_tools.py
import wikipedia

def wikipedia_search_tool(query: str):
    """Search Wikipedia and return summary."""
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=3),
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"title": query, "summary": f"Multiple results: {e.options[:3]}"}
    except Exception as e:
        return {"error": str(e)}
```

## Common Patterns

### Async Task Execution with Threading

```python
# main.py
import threading
from uuid import uuid4

@app.post("/generate_report")
def generate_report(request: ResearchRequest):
    task_id = str(uuid4())
    
    # Create DB task record
    task = Task(
        id=task_id,
        prompt=request.prompt,
        status="pending",
        created_at=datetime.utcnow()
    )
    db.add(task)
    db.commit()
    
    # Run agent workflow in background thread
    thread = threading.Thread(
        target=run_agent_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}

def run_agent_workflow(task_id: str, prompt: str, model: str):
    """Executes planning → research → write → edit pipeline."""
    # Update task status to running
    update_task_status(task_id, "running")
    
    # Step 1: Plan
    plan = planner_agent(prompt, model)
    log_step(task_id, "planning", "completed", plan)
    
    # Step 2: Research
    research_data = research_agent(plan["research_query"], tools=["tavily", "arxiv"])
    log_step(task_id, "research", "completed", research_data)
    
    # Step 3: Write
    draft = writer_agent(research_data, prompt)
    log_step(task_id, "writing", "completed", draft)
    
    # Step 4: Edit
    final_report = editor_agent(draft, feedback="")
    log_step(task_id, "editing", "completed", final_report)
    
    # Mark complete
    update_task_status(task_id, "completed", report=final_report)
```

### Database Session Management

```python
# main.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# In routes
@app.get("/task_status/{task_id}")
def task_status(task_id: str, db: Session = Depends(get_db)):
    task = db.query(Task).filter(Task.id == task_id).first()
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    return task
```

### Progress Tracking

```python
def log_step(task_id: str, step_name: str, status: str, result: str):
    """Log a workflow step to database."""
    db = SessionLocal()
    step = TaskStep(
        task_id=task_id,
        step_name=step_name,
        status=status,
        result=result
    )
    db.add(step)
    db.commit()
    db.close()

@app.get("/task_progress/{task_id}")
def task_progress(task_id: str, db: Session = Depends(get_db)):
    task = db.query(Task).filter(Task.id == task_id).first()
    steps = db.query(TaskStep).filter(TaskStep.task_id == task_id).all()
    
    return {
        "task_id": task_id,
        "status": task.status,
        "steps": [
            {
                "step_name": s.step_name,
                "status": s.status,
                "result": s.result
            }
            for s in steps
        ]
    }
```

## Development Workflow

### Hot Reload for Development

Mount your code directory for live updates:

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

### Connect to Postgres from Host

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

Query tasks:

```sql
SELECT id, prompt, status, created_at FROM tasks ORDER BY created_at DESC;
SELECT * FROM task_steps WHERE task_id = 'your-uuid-here';
```

### Reset Database on Startup

To drop/recreate tables on startup (useful for dev):

```python
# main.py
import os

if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)

Base.metadata.create_all(bind=engine)
```

Then run with:

```bash
docker run --rm -it -p 8000:8000 -p 5432:5432 \
  -e RESET_DB_ON_STARTUP=1 \
  --env-file .env \
  fastapi-postgres-service
```

## Troubleshooting

### Templates Not Found

If you see `TemplateNotFound` errors:

```bash
# Verify templates exist in container
docker exec -it fpsvc ls -la /app/templates
docker exec -it fpsvc ls -la /app/static
```

Ensure `Dockerfile` copies them:

```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

### Tavily API Errors

Check API key is set:

```python
import os
print(os.getenv("TAVILY_API_KEY"))  # Should not be None
```

Handle rate limits gracefully:

```python
def tavily_search_tool(query: str):
    try:
        response = requests.post(url, json=payload, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        return {"error": f"Tavily search failed: {str(e)}"}
```

### Wikipedia Rate Limiting

Implement retry logic:

```python
import time

def wikipedia_search_tool(query: str, retries: int = 3):
    for attempt in range(retries):
        try:
            return wikipedia.summary(query, sentences=3)
        except Exception as e:
            if attempt < retries - 1:
                time.sleep(2 ** attempt)
                continue
            return {"error": str(e)}
```

### Task Stuck in "Running"

Add timeout handling:

```python
import signal

def timeout_handler(signum, frame):
    raise TimeoutError("Agent workflow timeout")

def run_agent_workflow(task_id: str, prompt: str, model: str):
    try:
        signal.signal(signal.SIGALRM, timeout_handler)
        signal.alarm(300)  # 5 minute timeout
        
        # ... workflow steps ...
        
        signal.alarm(0)  # Cancel timeout
    except TimeoutError:
        update_task_status(task_id, "failed", report="Workflow timeout")
    except Exception as e:
        update_task_status(task_id, "failed", report=str(e))
```

### Container Won't Start

Check logs:

```bash
docker logs fpsvc
```

Common issues:
- Port 5432 already in use (stop existing Postgres)
- Missing `.env` file (provide `--env-file` or `-e` flags)
- Entrypoint script errors (check `docker/entrypoint.sh` has LF line endings, not CRLF)

### Database Connection Refused

Verify Postgres is running inside container:

```bash
docker exec -it fpsvc pg_lsclusters
docker exec -it fpsvc psql -U app -d appdb -c "SELECT 1;"
```

Check `DATABASE_URL` matches container setup:

```bash
docker exec -it fpsvc env | grep DATABASE_URL
```

## Example: Complete Research Task

```python
import requests
import time

# 1. Start research task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Explain the impact of transformer architectures on NLP",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task started: {task_id}")

# 2. Poll for progress
while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    print(f"Status: {progress['status']}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(5)

# 3. Get final report
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
if result["status"] == "completed":
    print("\n=== Final Report ===")
    print(result["report"])
else:
    print(f"Task failed: {result.get('report', 'Unknown error')}")
```

This skill covers the essential patterns for building, deploying, and extending the agentic research agent system.
