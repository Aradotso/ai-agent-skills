---
name: agentic-ai-research-agent
description: FastAPI research agent service that orchestrates multi-step AI workflows using Tavily, arXiv, and Wikipedia for research tasks with Postgres state management
triggers:
  - how do I use the agentic AI research agent
  - set up the reflective research agent service
  - create a research workflow with planning and execution agents
  - integrate Tavily arXiv and Wikipedia tools in research agents
  - build a multi-step AI research pipeline with FastAPI
  - configure the research agent with task progress tracking
  - run research agents with tool orchestration
  - deploy the agentic research workflow service
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI-based service that orchestrates multi-step AI research workflows. It uses a planning agent to break down research tasks, then executes them using specialized agents (research, writer, editor) with access to tools like Tavily search, arXiv, and Wikipedia. All task state and results are stored in Postgres, with real-time progress tracking.

**Key Features:**
- Multi-agent workflow orchestration (planner → research → writer → editor)
- Tool integration: Tavily API, arXiv, Wikipedia
- Postgres-backed task state management
- Real-time progress tracking via REST API
- Single-container Docker deployment (Postgres + FastAPI)
- Web UI for task submission and monitoring

## Installation

### Prerequisites

- Docker installed
- API keys for OpenAI and Tavily

### Environment Setup

Create a `.env` file in the project root:

```bash
# Required API keys
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...

# Optional database configuration (defaults are set by entrypoint)
# DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
# POSTGRES_USER=app
# POSTGRES_PASSWORD=local
# POSTGRES_DB=appdb
```

### Build and Run

```bash
# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the service (exposes port 8000 for API, 5432 for Postgres)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will:
1. Start Postgres inside the container
2. Create the database and user
3. Initialize SQLAlchemy tables
4. Launch Uvicorn on port 8000

## Project Structure

```
agentic-ai-public/
├── main.py                      # FastAPI app with routes
├── src/
│   ├── planning_agent.py        # Planner and executor logic
│   ├── agents.py                # Research/writer/editor agents
│   └── research_tools.py        # Tool implementations
├── templates/
│   └── index.html               # Web UI
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── .env                         # API keys (not committed)
```

## API Reference

### Start a Research Task

```python
import requests

# Submit a research task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Large Language Models for scientific discovery",
        "model": "openai:gpt-4o"
    }
)

task_data = response.json()
task_id = task_data["task_id"]
print(f"Task started: {task_id}")
```

**cURL equivalent:**
```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Large Language Models for scientific discovery", "model":"openai:gpt-4o"}'
```

### Poll Task Progress

```python
import requests
import time

def check_progress(task_id):
    response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    progress = response.json()
    
    print(f"Status: {progress['status']}")
    print(f"Current step: {progress.get('current_step', 'N/A')}")
    
    if progress.get('steps'):
        for step in progress['steps']:
            print(f"  - {step['name']}: {step['status']}")
    
    return progress['status']

# Poll until complete
while True:
    status = check_progress(task_id)
    if status in ['completed', 'failed']:
        break
    time.sleep(2)
```

### Get Final Report

```python
import requests

def get_final_report(task_id):
    response = requests.get(f"http://localhost:8000/task_status/{task_id}")
    result = response.json()
    
    if result['status'] == 'completed':
        print("Final Report:")
        print(result['report'])
        return result['report']
    else:
        print(f"Task status: {result['status']}")
        return None

report = get_final_report(task_id)
```

## Creating Custom Agents

### Research Tool Example

```python
# src/research_tools.py
import os
import requests
from typing import Dict, Any

def tavily_search_tool(query: str) -> Dict[str, Any]:
    """
    Search using Tavily API for recent, factual information.
    """
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "search_depth": "advanced",
        "max_results": 5
    }
    
    try:
        response = requests.post(url, json=payload, timeout=10)
        response.raise_for_status()
        data = response.json()
        
        results = []
        for item in data.get("results", []):
            results.append({
                "title": item.get("title", ""),
                "url": item.get("url", ""),
                "content": item.get("content", "")
            })
        
        return {"results": results}
    except Exception as e:
        return {"error": str(e)}
```

### Custom Agent Implementation

```python
# src/agents.py
import aisuite as ai
import os

client = ai.Client()

def research_agent(task: str, tools_results: list) -> str:
    """
    Agent that synthesizes research from tool outputs.
    """
    model = os.getenv("RESEARCH_MODEL", "openai:gpt-4o")
    
    # Format tool results into context
    context = "\n\n".join([
        f"Source: {r.get('source', 'Unknown')}\n{r.get('content', '')}"
        for r in tools_results
    ])
    
    messages = [
        {
            "role": "system",
            "content": "You are a research assistant. Synthesize the provided sources into a comprehensive analysis."
        },
        {
            "role": "user",
            "content": f"Task: {task}\n\nSources:\n{context}\n\nProvide a detailed research summary."
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

### Planning Agent Pattern

```python
# src/planning_agent.py
import aisuite as ai
import json

client = ai.Client()

def planner_agent(user_prompt: str, model: str = "openai:gpt-4o") -> dict:
    """
    Creates a multi-step research plan from user prompt.
    Returns a structured plan with steps and tools.
    """
    system_prompt = """You are a research planning agent. Break down the user's 
    research request into concrete steps. For each step, specify:
    - step_name: Brief description
    - tool: Which tool to use (tavily, arxiv, wikipedia, or none)
    - query: Search query for the tool
    
    Return a JSON object with a 'steps' array."""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    plan_text = response.choices[0].message.content
    
    # Extract JSON from response
    try:
        # Handle markdown code blocks
        if "```json" in plan_text:
            plan_text = plan_text.split("```json")[1].split("```")[0]
        elif "```" in plan_text:
            plan_text = plan_text.split("```")[1].split("```")[0]
        
        plan = json.loads(plan_text.strip())
        return plan
    except json.JSONDecodeError as e:
        return {"error": f"Failed to parse plan: {e}", "raw": plan_text}
```

## Database Models

```python
# main.py
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import datetime
import os

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.datetime.utcnow, onupdate=datetime.datetime.utcnow)

# Initialize database
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Complete FastAPI Route Example

```python
# main.py
from fastapi import FastAPI, HTTPException, BackgroundTasks
from pydantic import BaseModel
import uuid

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """
    Kick off a research task in the background.
    """
    task_id = str(uuid.uuid4())
    
    # Create task record
    db = SessionLocal()
    try:
        task = Task(
            id=task_id,
            prompt=request.prompt,
            model=request.model,
            status="planning"
        )
        db.add(task)
        db.commit()
    finally:
        db.close()
    
    # Start background workflow
    background_tasks.add_task(run_research_workflow, task_id, request.prompt, request.model)
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """
    Get real-time progress of a task.
    """
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.id == task_id).first()
        if not task:
            raise HTTPException(status_code=404, detail="Task not found")
        
        return {
            "task_id": task.id,
            "status": task.status,
            "current_step": task.current_step,
            "created_at": task.created_at.isoformat(),
            "updated_at": task.updated_at.isoformat()
        }
    finally:
        db.close()

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """
    Get final status and report.
    """
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.id == task_id).first()
        if not task:
            raise HTTPException(status_code=404, detail="Task not found")
        
        return {
            "task_id": task.id,
            "status": task.status,
            "report": task.report,
            "error": task.error
        }
    finally:
        db.close()
```

## Background Workflow Execution

```python
# main.py (continued)
from src.planning_agent import planner_agent
from src.agents import research_agent, writer_agent, editor_agent
from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

def update_task_status(task_id: str, status: str, current_step: str = None, report: str = None, error: str = None):
    """Helper to update task in database."""
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.id == task_id).first()
        if task:
            task.status = status
            if current_step:
                task.current_step = current_step
            if report:
                task.report = report
            if error:
                task.error = error
            db.commit()
    finally:
        db.close()

def run_research_workflow(task_id: str, prompt: str, model: str):
    """
    Execute the multi-step research workflow.
    """
    try:
        # Step 1: Planning
        update_task_status(task_id, "planning", "Creating research plan")
        plan = planner_agent(prompt, model)
        
        if "error" in plan:
            update_task_status(task_id, "failed", error=plan["error"])
            return
        
        # Step 2: Execute research steps
        update_task_status(task_id, "researching", "Gathering information")
        research_results = []
        
        for step in plan.get("steps", []):
            tool = step.get("tool", "none")
            query = step.get("query", "")
            
            if tool == "tavily":
                result = tavily_search_tool(query)
            elif tool == "arxiv":
                result = arxiv_search_tool(query)
            elif tool == "wikipedia":
                result = wikipedia_search_tool(query)
            else:
                result = {"content": "No tool specified"}
            
            research_results.append({
                "source": tool,
                "content": str(result)
            })
        
        # Step 3: Synthesize research
        update_task_status(task_id, "writing", "Synthesizing research")
        synthesis = research_agent(prompt, research_results)
        
        # Step 4: Write report
        update_task_status(task_id, "editing", "Writing final report")
        draft_report = writer_agent(prompt, synthesis)
        
        # Step 5: Edit report
        final_report = editor_agent(draft_report)
        
        # Complete
        update_task_status(task_id, "completed", "Done", report=final_report)
        
    except Exception as e:
        update_task_status(task_id, "failed", error=str(e))
```

## Configuration

### Database Connection

The service uses `DATABASE_URL` environment variable:

```bash
# Default (set by entrypoint.sh)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Custom configuration
export DATABASE_URL=postgresql://myuser:mypass@db-host:5432/mydb
```

### Model Selection

Specify the AI model in the request:

```python
# Use different models for different tasks
research_request = {
    "prompt": "Quantum computing applications",
    "model": "openai:gpt-4o"  # or "openai:gpt-3.5-turbo"
}
```

### Tool Configuration

Tools are configured via environment variables:

```bash
# Tavily (required for web search)
TAVILY_API_KEY=tvly-...

# OpenAI (required for agent logic)
OPENAI_API_KEY=sk-...
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs fpsvc

# Verify Postgres is running inside container
docker exec -it fpsvc pg_lsclusters

# Manually test DB connection
docker exec -it fpsvc psql -U app -d appdb -c "SELECT 1;"
```

### API Key Errors

```bash
# Verify environment variables are loaded
docker exec -it fpsvc env | grep API_KEY

# Restart with explicit env vars if .env not working
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -e OPENAI_API_KEY=sk-... \
  -e TAVILY_API_KEY=tvly-... \
  --name fpsvc \
  fastapi-postgres-service
```

### Database Connection Issues

```python
# Test database connectivity
from sqlalchemy import create_engine, text
import os

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)

with engine.connect() as conn:
    result = conn.execute(text("SELECT current_database();"))
    print(result.fetchone())
```

### Tasks Stuck in Progress

```sql
-- Connect to database
psql "postgresql://app:local@localhost:5432/appdb"

-- Check task status
SELECT id, status, current_step, created_at, updated_at FROM tasks;

-- Manually reset stuck task
UPDATE tasks SET status = 'failed', error = 'Manual reset' WHERE id = 'task-uuid';
```

### Rate Limiting with Tools

```python
# Add retry logic for external APIs
import time
from typing import Dict, Any

def tavily_search_tool_with_retry(query: str, max_retries: int = 3) -> Dict[str, Any]:
    for attempt in range(max_retries):
        try:
            result = tavily_search_tool(query)
            if "error" not in result:
                return result
        except Exception as e:
            if attempt == max_retries - 1:
                return {"error": f"Failed after {max_retries} attempts: {e}"}
            time.sleep(2 ** attempt)  # Exponential backoff
    
    return {"error": "Max retries exceeded"}
```

### Development Hot Reload

```bash
# Mount code for live editing
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Best Practices

1. **Always use environment variables** for API keys (never hardcode)
2. **Implement retry logic** for external API calls (Tavily, arXiv)
3. **Set reasonable timeouts** on tool calls to prevent hanging tasks
4. **Log extensively** in background tasks for debugging
5. **Use database transactions** when updating task status
6. **Implement task cleanup** for old/completed tasks
7. **Monitor Postgres performance** for high-volume deployments
