---
name: deeplearning-ai-agentic-research-agent
description: FastAPI-based reflective research agent with planning, multi-tool execution (Tavily, arXiv, Wikipedia), and Postgres state management
triggers:
  - set up the agentic research agent service
  - create a reflective research workflow with planning agents
  - use tavily arxiv and wikipedia tools in an agent
  - build a fastapi research agent with postgres
  - implement multi-step agent planning and execution
  - deploy the deeplearning ai research agent
  - integrate openai agents with research tools
  - create a task-based research report generator
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI web service that orchestrates multi-step research workflows using planning agents, tool-using research/writer/editor agents, and Postgres for task state management. Built for the Agentic Workflow course, this service demonstrates reflective agent patterns with external tool integration (Tavily search, arXiv, Wikipedia).

## What It Does

- **Planning Agent**: Breaks down research queries into structured subtasks
- **Executor Agents**: Research, writer, and editor agents work on subtasks
- **Tool Integration**: Tavily web search, arXiv papers, Wikipedia lookups
- **State Management**: Postgres stores task progress, steps, and results
- **Progress Tracking**: Real-time status API for multi-step workflows
- **Web UI**: Simple interface to kick off research tasks

## Installation

### Prerequisites

- Docker and Docker Desktop
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```bash
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Docker Build & Run

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

The entrypoint script automatically:
1. Starts Postgres cluster
2. Creates database and user
3. Launches FastAPI with Uvicorn

### Access Points

- **Web UI**: http://localhost:8000/
- **API Docs**: http://localhost:8000/docs
- **Postgres**: `postgresql://app:local@localhost:5432/appdb`

## Key API Endpoints

### Start Research Task

```bash
POST /generate_report
```

**Request:**
```json
{
  "prompt": "Large Language Models for scientific discovery",
  "model": "openai:gpt-4o"
}
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Poll Progress

```bash
GET /task_progress/{task_id}
```

**Response:**
```json
{
  "task_id": "550e8400-...",
  "status": "in_progress",
  "current_step": "research",
  "steps": [
    {
      "name": "planning",
      "status": "completed",
      "substeps": [...]
    },
    {
      "name": "research",
      "status": "in_progress",
      "substeps": [
        {"description": "Search Tavily for LLM papers", "status": "completed"},
        {"description": "Query arXiv for recent research", "status": "running"}
      ]
    }
  ]
}
```

### Get Final Report

```bash
GET /task_status/{task_id}
```

**Response:**
```json
{
  "task_id": "550e8400-...",
  "status": "completed",
  "report": "# Large Language Models for Scientific Discovery\n\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:22Z"
}
```

## Project Structure

```
.
├── main.py                    # FastAPI app with routes
├── src/
│   ├── planning_agent.py      # planner_agent(), executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # tavily, arxiv, wikipedia tools
├── templates/
│   └── index.html             # Web UI
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Postgres + Uvicorn startup
├── requirements.txt
├── Dockerfile
└── .env                       # API keys (gitignored)
```

## Code Examples

### Using the Planning Agent

```python
from src.planning_agent import planner_agent, executor_agent_step

# Create a plan
plan = planner_agent(
    prompt="Research quantum computing applications in drug discovery",
    model="openai:gpt-4o"
)

# Plan structure:
# {
#   "steps": [
#     {"name": "research", "description": "...", "substeps": [...]},
#     {"name": "writing", "description": "...", "substeps": [...]},
#     {"name": "editing", "description": "...", "substeps": [...]}
#   ]
# }

# Execute a step
result = executor_agent_step(
    step=plan["steps"][0],
    model="openai:gpt-4o"
)
```

### Creating Custom Research Tools

```python
from src.research_tools import tavily_search_tool, arxiv_search_tool

# Use Tavily for web search
tavily_results = tavily_search_tool(
    query="recent advances in transformer architectures",
    max_results=5
)

# Search arXiv papers
arxiv_papers = arxiv_search_tool(
    query="attention mechanisms neural networks",
    max_results=10
)

# Process results
for paper in arxiv_papers:
    print(f"{paper['title']} - {paper['authors']}")
```

### Custom Agent Implementation

```python
from src.agents import research_agent, writer_agent, editor_agent

# Research phase
research_output = research_agent(
    task="Investigate transformer model efficiency improvements",
    tools=[tavily_search_tool, arxiv_search_tool],
    model="openai:gpt-4o"
)

# Writing phase
draft = writer_agent(
    research_data=research_output,
    task="Create a comprehensive report",
    model="openai:gpt-4o"
)

# Editing phase
final_report = editor_agent(
    draft=draft,
    task="Polish and fact-check the report",
    model="openai:gpt-4o"
)
```

### Database Models

```python
from sqlalchemy import Column, String, JSON, DateTime, Enum
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.dialects.postgresql import UUID
import uuid

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    prompt = Column(String, nullable=False)
    model = Column(String, nullable=False)
    status = Column(Enum("pending", "in_progress", "completed", "failed", name="task_status"))
    plan = Column(JSON)
    current_step = Column(String)
    report = Column(String)
    created_at = Column(DateTime)
    completed_at = Column(DateTime)

# Query tasks
from sqlalchemy.orm import Session

def get_task(db: Session, task_id: str):
    return db.query(Task).filter(Task.id == task_id).first()

def update_task_status(db: Session, task_id: str, status: str, report: str = None):
    task = db.query(Task).filter(Task.id == task_id).first()
    task.status = status
    if report:
        task.report = report
    db.commit()
```

### FastAPI Route Implementation

```python
from fastapi import FastAPI, BackgroundTasks, HTTPException
from pydantic import BaseModel
import threading

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    task_id = str(uuid.uuid4())
    
    # Store task in DB
    with SessionLocal() as db:
        task = Task(
            id=task_id,
            prompt=request.prompt,
            model=request.model,
            status="pending",
            created_at=datetime.utcnow()
        )
        db.add(task)
        db.commit()
    
    # Run workflow in background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background worker for research workflow"""
    try:
        # Generate plan
        plan = planner_agent(prompt, model)
        
        # Execute each step
        for step in plan["steps"]:
            result = executor_agent_step(step, model)
            # Update DB with step results
        
        # Mark complete
        with SessionLocal() as db:
            update_task_status(db, task_id, "completed", report=final_report)
    except Exception as e:
        with SessionLocal() as db:
            update_task_status(db, task_id, "failed")
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key
TAVILY_API_KEY=tvly-...            # Tavily search API key

# Optional - Database (defaults set by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional - Development
RESET_DB_ON_STARTUP=0              # Set to 1 to drop tables on startup
```

### Custom Tool Configuration

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5):
    """Search the web using Tavily API"""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        raise ValueError("TAVILY_API_KEY not set")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results,
            "search_depth": "advanced"
        }
    )
    return response.json()
```

## Common Patterns

### Multi-Step Workflow with Progress Tracking

```python
def run_research_workflow(task_id: str, prompt: str, model: str):
    with SessionLocal() as db:
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "in_progress"
        db.commit()
        
        # Step 1: Planning
        task.current_step = "planning"
        db.commit()
        plan = planner_agent(prompt, model)
        task.plan = plan
        db.commit()
        
        # Step 2: Research
        task.current_step = "research"
        db.commit()
        research_data = executor_agent_step(plan["steps"][0], model)
        
        # Step 3: Writing
        task.current_step = "writing"
        db.commit()
        draft = executor_agent_step(plan["steps"][1], model)
        
        # Step 4: Editing
        task.current_step = "editing"
        db.commit()
        final_report = executor_agent_step(plan["steps"][2], model)
        
        # Complete
        task.status = "completed"
        task.report = final_report
        task.completed_at = datetime.utcnow()
        db.commit()
```

### Polling for Task Completion (Client Side)

```python
import requests
import time

def wait_for_task(task_id: str, max_wait: int = 300):
    """Poll task status until completion"""
    start_time = time.time()
    
    while time.time() - start_time < max_wait:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = response.json()
        
        if data["status"] in ["completed", "failed"]:
            return data
        
        print(f"Current step: {data.get('current_step')}")
        time.sleep(5)
    
    raise TimeoutError("Task did not complete in time")
```

### Error Handling in Agents

```python
def safe_executor(step, model):
    """Execute agent step with error handling"""
    max_retries = 3
    
    for attempt in range(max_retries):
        try:
            return executor_agent_step(step, model)
        except Exception as e:
            if attempt == max_retries - 1:
                return {
                    "status": "failed",
                    "error": str(e),
                    "step": step["name"]
                }
            time.sleep(2 ** attempt)  # Exponential backoff
```

## Troubleshooting

### Container Won't Start

**Symptom**: "Postgres not ready" or permission errors

**Solution**:
```bash
# Check logs
docker logs fpsvc

# Ensure entrypoint is executable
chmod +x docker/entrypoint.sh

# Verify Postgres version detection
docker exec -it fpsvc bash -lc "psql --version"
```

### Templates Not Found

**Symptom**: `TemplateNotFound: index.html`

**Solution**: Verify Dockerfile copies templates correctly:
```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

Check inside container:
```bash
docker exec -it fpsvc ls -la /app/templates/
```

### Database Connection Errors

**Symptom**: `could not connect to server`

**Solution**:
```bash
# Check DATABASE_URL is set correctly
docker exec -it fpsvc bash -lc "echo \$DATABASE_URL"

# Test connection manually
docker exec -it fpsvc bash -lc "psql postgresql://app:local@127.0.0.1:5432/appdb -c 'SELECT 1;'"

# Ensure Postgres is running
docker exec -it fpsvc bash -lc "pg_lsclusters"
```

### API Key Errors

**Symptom**: `API key not found` or authentication failures

**Solution**:
```bash
# Verify .env file is loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Restart with explicit env vars
docker run --rm -it \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  -p 8000:8000 \
  fastapi-postgres-service
```

### Tasks Stuck in "in_progress"

**Symptom**: Task never completes, no errors logged

**Solution**:
```python
# Add timeout to background threads
import signal

def timeout_handler(signum, frame):
    raise TimeoutError("Task execution timeout")

signal.signal(signal.SIGALRM, timeout_handler)
signal.alarm(600)  # 10 minute timeout

try:
    run_research_workflow(task_id, prompt, model)
finally:
    signal.alarm(0)
```

### Tool Rate Limits

**Symptom**: Wikipedia or arXiv errors

**Solution**: Implement retry logic with backoff:
```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
def resilient_tool_call(tool_func, *args, **kwargs):
    return tool_func(*args, **kwargs)
```

## Development Tips

### Hot Reload for Development

```bash
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

# Query task status
SELECT id, prompt, status, current_step, created_at FROM tasks;
```

### Disable Table Drops on Startup

```python
# main.py
import os

if os.getenv("RESET_DB_ON_STARTUP") != "1":
    # Don't drop tables
    pass
else:
    Base.metadata.drop_all(bind=engine)

Base.metadata.create_all(bind=engine)
```
