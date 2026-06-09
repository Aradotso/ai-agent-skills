---
name: deeplearning-ai-agentic-research-agent
description: FastAPI-based reflective research agent with Postgres persistence, multi-step planning, and tool integration (Tavily, arXiv, Wikipedia)
triggers:
  - set up the agentic AI research agent service
  - create a research workflow with planning agents
  - use the reflective research agent API
  - integrate Tavily and arXiv research tools
  - build a multi-step agent workflow with FastAPI
  - deploy the agentic research agent with Docker
  - configure the research agent planning workflow
  - implement task progress tracking for AI agents
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The DeepLearning.AI Agentic Research Agent is a FastAPI web service that orchestrates multi-step research workflows using AI agents. It features a planning agent that breaks down research tasks, executor agents that use various tools (Tavily search, arXiv, Wikipedia), and a persistence layer (Postgres) to track task state and results.

**Key capabilities:**
- Multi-step agent workflow (planner → research → writer → editor)
- Tool-using agents with Tavily, arXiv, and Wikipedia integration
- Postgres-backed task state and progress tracking
- RESTful API with live progress endpoints
- Single-container Docker deployment (Postgres + FastAPI)
- Reflective planning and execution pattern

## Installation

### Prerequisites

- Docker (Desktop on Windows/macOS, or engine on Linux)
- `.env` file with API keys:

```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Quick Start with Docker

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the service (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service starts Postgres automatically and runs database migrations on startup.

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Ensure Postgres is running and DATABASE_URL is set
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="sk-..."
export TAVILY_API_KEY="tvly-..."

# Run the FastAPI app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├─ main.py                      # FastAPI app entry point
├─ src/
│  ├─ planning_agent.py         # planner_agent(), executor_agent_step()
│  ├─ agents.py                 # research_agent, writer_agent, editor_agent
│  └─ research_tools.py         # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├─ templates/
│  └─ index.html                # Web UI template
├─ static/                      # Static assets (CSS/JS)
├─ docker/
│  └─ entrypoint.sh             # Postgres + Uvicorn startup script
├─ requirements.txt
├─ Dockerfile
└─ README.md
```

## Core Concepts

### 1. Planning Agent

The planning agent breaks down a research prompt into discrete steps:

```python
from src.planning_agent import planner_agent

# Generate a research plan
plan = planner_agent(
    prompt="Large Language Models for scientific discovery",
    model="openai:gpt-4o"
)
# Returns: List of steps with substeps
```

### 2. Executor Agent

Executes individual steps from the plan using specialized agents:

```python
from src.planning_agent import executor_agent_step

# Execute a research step
result = executor_agent_step(
    step_description="Search for recent papers on LLMs in science",
    model="openai:gpt-4o"
)
```

### 3. Research Tools

Three main tools are available for agents:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Tavily web search
web_results = tavily_search_tool(query="latest LLM research")

# arXiv academic search
papers = arxiv_search_tool(query="large language models")

# Wikipedia lookup
wiki_summary = wikipedia_search_tool(topic="transformer neural networks")
```

## API Reference

### Start a Research Task

```bash
POST /generate_report
Content-Type: application/json

{
  "prompt": "Large Language Models for scientific discovery",
  "model": "openai:gpt-4o"
}
```

Response:
```json
{
  "task_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

### Poll Task Progress

```bash
GET /task_progress/{task_id}
```

Response:
```json
{
  "task_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "running",
  "current_step": 2,
  "total_steps": 5,
  "steps": [
    {
      "step": 1,
      "description": "Research recent papers",
      "status": "completed",
      "substeps": [...]
    },
    {
      "step": 2,
      "description": "Synthesize findings",
      "status": "running",
      "substeps": [...]
    }
  ]
}
```

### Get Final Report

```bash
GET /task_status/{task_id}
```

Response:
```json
{
  "task_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "completed",
  "report": "# Research Report\n\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:00Z"
}
```

### Access Web UI

Navigate to `http://localhost:8000/` for the browser-based interface.

## Code Examples

### Custom Research Agent

```python
from fastapi import FastAPI
from src.planning_agent import planner_agent, executor_agent_step
import os

app = FastAPI()

@app.post("/custom_research")
async def custom_research(prompt: str):
    """Custom research endpoint with specific model."""
    
    # Generate plan
    plan = planner_agent(
        prompt=prompt,
        model="openai:gpt-4o"
    )
    
    results = []
    for step in plan["steps"]:
        # Execute each step
        result = executor_agent_step(
            step_description=step["description"],
            model="openai:gpt-4o"
        )
        results.append({
            "step": step["step"],
            "result": result
        })
    
    return {"plan": plan, "results": results}
```

### Using Research Tools Directly

```python
from src.research_tools import tavily_search_tool, arxiv_search_tool
import os

# Ensure API key is set
os.environ["TAVILY_API_KEY"] = os.getenv("TAVILY_API_KEY")

def multi_source_research(query: str):
    """Search multiple sources and combine results."""
    
    # Web search
    web_results = tavily_search_tool(query=query)
    
    # Academic papers
    papers = arxiv_search_tool(query=query)
    
    # Combine findings
    combined = {
        "query": query,
        "web_sources": web_results,
        "academic_papers": papers
    }
    
    return combined

# Usage
findings = multi_source_research("transformer architecture improvements")
```

### Custom Agent Implementation

```python
from src.agents import research_agent, writer_agent, editor_agent

def create_research_workflow(topic: str, model: str = "openai:gpt-4o"):
    """End-to-end research workflow."""
    
    # Step 1: Research
    research_data = research_agent(
        topic=topic,
        model=model
    )
    
    # Step 2: Write draft
    draft_report = writer_agent(
        research_data=research_data,
        model=model
    )
    
    # Step 3: Edit and refine
    final_report = editor_agent(
        draft=draft_report,
        model=model
    )
    
    return final_report

# Usage
report = create_research_workflow("quantum computing applications in ML")
```

### Database Model Extension

```python
from sqlalchemy import Column, String, Integer, JSON, DateTime
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class ResearchTask(Base):
    __tablename__ = "research_tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(String, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    plan = Column(JSON)
    results = Column(JSON)
    report = Column(String)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)

# Query tasks
from sqlalchemy.orm import Session

def get_recent_tasks(db: Session, limit: int = 10):
    """Retrieve recent research tasks."""
    return db.query(ResearchTask)\
        .order_by(ResearchTask.created_at.desc())\
        .limit(limit)\
        .all()
```

### Async Task Processing

```python
import threading
from uuid import uuid4
from sqlalchemy.orm import Session

def async_research_task(task_id: str, prompt: str, model: str, db: Session):
    """Background research task execution."""
    
    # Update status to running
    task = db.query(ResearchTask).filter_by(id=task_id).first()
    task.status = "running"
    db.commit()
    
    try:
        # Generate plan
        plan = planner_agent(prompt=prompt, model=model)
        task.plan = plan
        db.commit()
        
        # Execute steps
        results = []
        for step in plan["steps"]:
            result = executor_agent_step(
                step_description=step["description"],
                model=model
            )
            results.append(result)
        
        task.results = results
        task.status = "completed"
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = str(e)
        db.commit()

# Start background task
task_id = str(uuid4())
thread = threading.Thread(
    target=async_research_task,
    args=(task_id, "AI safety research", "openai:gpt-4o", db_session)
)
thread.start()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key for LLM calls
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Optional
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb  # Postgres connection
POSTGRES_USER=app               # Database user (default: app)
POSTGRES_PASSWORD=local         # Database password (default: local)
POSTGRES_DB=appdb               # Database name (default: appdb)
RESET_DB_ON_STARTUP=0           # Set to 1 to drop/recreate tables on startup
```

### Docker Volume Mounting (Development)

```bash
# Mount source code for hot reload
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Model Configuration

The service supports multiple models via `aisuite`:

```python
# In requests, specify model as "provider:model_name"
models = [
    "openai:gpt-4o",
    "openai:gpt-4-turbo",
    "openai:gpt-3.5-turbo",
    # Add other providers as needed
]
```

## Common Patterns

### Error Handling in Agents

```python
from src.planning_agent import planner_agent

def safe_plan_generation(prompt: str, model: str, max_retries: int = 3):
    """Generate plan with retry logic."""
    
    for attempt in range(max_retries):
        try:
            plan = planner_agent(prompt=prompt, model=model)
            return plan
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            print(f"Retry {attempt + 1}/{max_retries} after error: {e}")
    
    return None
```

### Streaming Progress Updates

```python
from fastapi import WebSocket
from sqlalchemy.orm import Session

async def stream_task_progress(websocket: WebSocket, task_id: str, db: Session):
    """Stream real-time progress updates via WebSocket."""
    
    await websocket.accept()
    
    while True:
        task = db.query(ResearchTask).filter_by(id=task_id).first()
        
        await websocket.send_json({
            "status": task.status,
            "progress": task.plan if task.plan else {},
            "timestamp": datetime.utcnow().isoformat()
        })
        
        if task.status in ["completed", "failed"]:
            break
        
        await asyncio.sleep(2)
```

### Custom Tool Integration

```python
def custom_search_tool(query: str, source: str = "all"):
    """Unified search across multiple sources."""
    
    results = {}
    
    if source in ["all", "web"]:
        results["web"] = tavily_search_tool(query=query)
    
    if source in ["all", "academic"]:
        results["academic"] = arxiv_search_tool(query=query)
    
    if source in ["all", "wiki"]:
        results["wiki"] = wikipedia_search_tool(topic=query)
    
    return results
```

## Troubleshooting

### Templates Not Found

If you see `TemplateNotFound` errors:

```bash
# Verify templates exist in container
docker exec -it fpsvc ls -l /app/templates

# Ensure Dockerfile copies templates
# Add to Dockerfile if missing:
COPY templates/ /app/templates/
COPY static/ /app/static/
```

### Database Connection Issues

```bash
# Test database connectivity from host
psql "postgresql://app:local@localhost:5432/appdb"

# Check DATABASE_URL in container
docker exec -it fpsvc env | grep DATABASE_URL

# View Postgres logs
docker exec -it fpsvc tail -f /var/log/postgresql/postgresql-17-main.log
```

### Tables Not Created

```python
# In main.py, ensure tables are created on startup
from sqlalchemy import create_engine
from models import Base

engine = create_engine(os.getenv("DATABASE_URL"))
Base.metadata.create_all(bind=engine)
```

### Tavily API Errors

```bash
# Verify API key is set
docker exec -it fpsvc env | grep TAVILY_API_KEY

# Test Tavily directly
python -c "from src.research_tools import tavily_search_tool; print(tavily_search_tool('test'))"
```

### Rate Limiting

```python
import time

def rate_limited_search(query: str, delay: float = 1.0):
    """Search with rate limiting."""
    time.sleep(delay)
    return tavily_search_tool(query=query)
```

### Memory Issues with Large Tasks

```python
# Process steps in batches
def batch_execute_steps(steps: list, batch_size: int = 5):
    """Execute steps in batches to manage memory."""
    
    results = []
    for i in range(0, len(steps), batch_size):
        batch = steps[i:i + batch_size]
        batch_results = [executor_agent_step(s["description"]) for s in batch]
        results.extend(batch_results)
        
        # Clear memory between batches if needed
        import gc
        gc.collect()
    
    return results
```

### Debugging Agent Execution

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

# Add logging to agent steps
def debug_executor_step(step_description: str, model: str):
    logger.debug(f"Executing step: {step_description}")
    result = executor_agent_step(step_description=step_description, model=model)
    logger.debug(f"Step result: {result[:200]}...")  # Log first 200 chars
    return result
```

## Advanced Usage

### Multi-Model Comparison

```python
def compare_models(prompt: str, models: list):
    """Generate research plans with different models."""
    
    results = {}
    for model in models:
        plan = planner_agent(prompt=prompt, model=model)
        results[model] = plan
    
    return results

# Usage
comparison = compare_models(
    prompt="Quantum machine learning",
    models=["openai:gpt-4o", "openai:gpt-3.5-turbo"]
)
```

### Custom Persistence Layer

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Create custom session factory
engine = create_engine(
    os.getenv("DATABASE_URL"),
    pool_size=10,
    max_overflow=20
)
SessionLocal = sessionmaker(bind=engine)

# Use in dependency injection
from fastapi import Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```
