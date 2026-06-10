---
name: agentic-ai-research-agent
description: Build and deploy reflective research agents using FastAPI, Postgres, and LLM-powered planning with Tavily, arXiv, and Wikipedia tools
triggers:
  - how do I set up the agentic AI research agent
  - create a multi-step research workflow with planning agents
  - integrate Tavily arXiv and Wikipedia search tools
  - build a FastAPI research agent with task tracking
  - deploy the reflective research agent in Docker
  - monitor research task progress with task_id
  - use the planner and executor agents for research
  - troubleshoot research agent Postgres database issues
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI web service that orchestrates multi-step research workflows using LLM-powered planning and execution agents. It integrates research tools (Tavily, arXiv, Wikipedia), stores task state in Postgres, and provides real-time progress tracking for complex research tasks.

**Key Features:**
- Planning agent that decomposes research questions into subtasks
- Executor agents (research, writer, editor) with tool access
- Postgres-backed task state and result persistence
- Real-time progress tracking via REST API
- Single-container Docker deployment with embedded Postgres

## Project Structure

```
.
├── main.py                      # FastAPI application entry point
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily, arxiv, wikipedia search tools
├── templates/
│   └── index.html               # UI template
├── static/                      # Static assets (CSS/JS)
├── docker/
│   └── entrypoint.sh            # Postgres + Uvicorn startup script
├── requirements.txt
├── Dockerfile
└── .env                         # API keys (not in repo)
```

## Installation

### Prerequisites

- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build the Docker Image

```bash
docker build -t fastapi-postgres-service .
```

### Run the Container

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The container starts Postgres, creates the database schema, and launches the FastAPI server.

## Core API Endpoints

### 1. Home Page (UI)

```
GET http://localhost:8000/
```

Serves a web interface for submitting research tasks.

### 2. Generate Report (Kick off research)

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 3. Task Progress (Live Updates)

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "running",
  "current_step": "research",
  "substeps": [
    {
      "name": "Search Tavily",
      "status": "completed",
      "result": "Found 5 relevant articles..."
    },
    {
      "name": "Query arXiv",
      "status": "running"
    }
  ]
}
```

### 4. Task Status (Final Result)

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Large Language Models for Scientific Discovery\n\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:23Z"
}
```

## Database Schema

The application uses SQLAlchemy models for task tracking:

```python
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)  # UUID
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    report = Column(Text, nullable=True)
    created_at = Column(DateTime)
    completed_at = Column(DateTime, nullable=True)

# Connect to database
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Working with Agents

### Planning Agent

The planning agent decomposes research tasks into executable steps:

```python
from src.planning_agent import planner_agent

def run_research_workflow(prompt: str, model: str):
    # Generate plan
    plan = planner_agent(
        query=prompt,
        model=model
    )
    
    # Plan structure:
    # {
    #   "steps": [
    #     {"agent": "research", "task": "Search for recent papers on LLMs"},
    #     {"agent": "writer", "task": "Draft introduction section"},
    #     {"agent": "editor", "task": "Review and refine content"}
    #   ]
    # }
    
    return plan
```

### Executor Agent

Execute individual plan steps:

```python
from src.planning_agent import executor_agent_step

def execute_step(step_config: dict, context: dict):
    result = executor_agent_step(
        agent_name=step_config["agent"],
        task=step_config["task"],
        context=context,
        model="openai:gpt-4o"
    )
    return result
```

## Research Tools

### Tavily Search Tool

```python
from src.research_tools import tavily_search_tool

results = tavily_search_tool(
    query="quantum computing applications",
    max_results=5
)
# Returns: List of web search results with titles, URLs, snippets
```

### arXiv Search Tool

```python
from src.research_tools import arxiv_search_tool

papers = arxiv_search_tool(
    query="transformer neural networks",
    max_results=10
)
# Returns: List of academic papers with abstracts, authors, links
```

### Wikipedia Search Tool

```python
from src.research_tools import wikipedia_search_tool

summary = wikipedia_search_tool(
    query="artificial intelligence",
    sentences=5
)
# Returns: Wikipedia article summary
```

## Complete Research Workflow Example

```python
import asyncio
from fastapi import BackgroundTasks
from src.planning_agent import planner_agent, executor_agent_step
from sqlalchemy.orm import Session

async def background_research_task(task_id: str, prompt: str, model: str, db: Session):
    try:
        # Update status
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "running"
        db.commit()
        
        # Step 1: Plan
        plan = planner_agent(query=prompt, model=model)
        
        # Step 2: Execute each step
        context = {"prompt": prompt, "results": []}
        
        for step in plan["steps"]:
            result = executor_agent_step(
                agent_name=step["agent"],
                task=step["task"],
                context=context,
                model=model
            )
            context["results"].append(result)
        
        # Step 3: Compile final report
        final_report = context["results"][-1]  # Editor output
        
        # Update task
        task.status = "completed"
        task.report = final_report
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
```

## FastAPI Integration

### Main Application Setup

```python
from fastapi import FastAPI, BackgroundTasks, Depends
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel
import uuid

app = FastAPI(title="Research Agent Service")

# Mount static files and templates
app.mount("/static", StaticFiles(directory="static"), name="static")
templates = Jinja2Templates(directory="templates")

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(
    request: ResearchRequest,
    background_tasks: BackgroundTasks,
    db: Session = Depends(get_db)
):
    task_id = str(uuid.uuid4())
    
    # Create task record
    new_task = Task(
        id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending",
        created_at=datetime.utcnow()
    )
    db.add(new_task)
    db.commit()
    
    # Start background task
    background_tasks.add_task(
        background_research_task,
        task_id,
        request.prompt,
        request.model,
        db
    )
    
    return {"task_id": task_id}
```

## Configuration

### Database URL Override

```bash
# Custom database connection
docker run --rm -it \
  -p 8000:8000 \
  -e DATABASE_URL="postgresql://user:pass@host:5432/dbname" \
  --env-file .env \
  fastapi-postgres-service
```

### Postgres Credentials

```bash
# Override default Postgres settings
docker run --rm -it \
  -p 8000:8000 \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydb \
  --env-file .env \
  fastapi-postgres-service
```

### Development Mode with Hot Reload

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Troubleshooting

### Tables Disappear on Restart

The app may drop tables on startup. To prevent this:

```python
# main.py - Modify startup logic
import os

if os.getenv("RESET_DB_ON_STARTUP") != "1":
    # Don't drop tables
    pass
else:
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)
```

Run without reset:

```bash
docker run --rm -it -p 8000:8000 --env-file .env fastapi-postgres-service
```

### Database Connection Issues

Check the DATABASE_URL:

```bash
docker exec -it fpsvc bash -lc 'echo $DATABASE_URL'
```

Test connection from host:

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

### Tavily API Rate Limits

Handle errors gracefully:

```python
from src.research_tools import tavily_search_tool

try:
    results = tavily_search_tool(query="AI research", max_results=5)
except Exception as e:
    print(f"Tavily search failed: {e}")
    results = []  # Fall back to other sources
```

### Templates Not Found

Verify templates are in the container:

```bash
docker exec -it fpsvc bash -lc "ls -l /app/templates"
```

Ensure Dockerfile copies them:

```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

### View Container Logs

```bash
docker logs -f fpsvc
```

### Access Container Shell

```bash
docker exec -it fpsvc bash
```

### Check Postgres Status

```bash
docker exec -it fpsvc bash -lc "pg_lsclusters"
```

## Common Patterns

### Polling for Task Completion

```python
import time
import requests

def wait_for_task(task_id: str, timeout: int = 300):
    start = time.time()
    while time.time() - start < timeout:
        response = requests.get(f"http://localhost:8000/task_status/{task_id}")
        data = response.json()
        
        if data["status"] in ["completed", "failed"]:
            return data
        
        time.sleep(2)
    
    raise TimeoutError(f"Task {task_id} did not complete in {timeout}s")
```

### Custom Agent with Tools

```python
def custom_agent(task: str, context: dict, model: str):
    # Use multiple tools in sequence
    tavily_results = tavily_search_tool(task, max_results=3)
    arxiv_results = arxiv_search_tool(task, max_results=3)
    
    # Combine results
    combined = {
        "web": tavily_results,
        "academic": arxiv_results
    }
    
    # Pass to LLM for synthesis
    # (Implementation depends on your aisuite client)
    
    return combined
```

### Streaming Progress Updates

```python
from fastapi.responses import StreamingResponse
import json

@app.get("/task_progress_stream/{task_id}")
async def stream_progress(task_id: str, db: Session = Depends(get_db)):
    async def event_stream():
        while True:
            task = db.query(Task).filter(Task.id == task_id).first()
            yield f"data: {json.dumps({'status': task.status})}\n\n"
            
            if task.status in ["completed", "failed"]:
                break
            
            await asyncio.sleep(1)
    
    return StreamingResponse(event_stream(), media_type="text/event-stream")
```

This skill provides comprehensive coverage of building, deploying, and extending the Agentic AI Research Agent service.
