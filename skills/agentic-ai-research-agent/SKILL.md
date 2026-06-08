---
name: agentic-ai-research-agent
description: FastAPI research agent service with multi-step workflow planning, tool integration (Tavily, arXiv, Wikipedia), and PostgreSQL state management
triggers:
  - set up the agentic AI research agent
  - create a reflective research workflow with agents
  - build a research agent with planning and execution
  - implement multi-step agent workflow with FastAPI
  - use Tavily and arXiv for agent research tasks
  - configure the agentic research agent service
  - deploy research agent with Postgres and FastAPI
  - integrate planning agent with research tools
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill covers the DeepLearning.AI Agentic AI Research Agent - a FastAPI-based service that orchestrates multi-step research workflows using planning agents, research tools (Tavily, arXiv, Wikipedia), and PostgreSQL for state management. The system runs planner → research → writer → editor agents in a threaded workflow with live progress tracking.

## What This Project Does

The Agentic AI Research Agent is a reflective research system that:
- Plans multi-step research workflows using a planning agent
- Executes research using tool-enabled agents (Tavily search, arXiv papers, Wikipedia)
- Maintains task state and results in PostgreSQL
- Provides real-time progress tracking via REST API
- Generates comprehensive research reports through agent collaboration
- Runs in a single Docker container (Postgres + FastAPI)

## Installation

### Prerequisites

Ensure you have Docker installed and create a `.env` file with required API keys:

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

# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the container
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will start Postgres, create the database, and launch the FastAPI app on port 8000.

## Project Structure

```
agentic-ai-public/
├── main.py                    # FastAPI application entry point
├── src/
│   ├── planning_agent.py      # Planner and executor agents
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tool integrations (Tavily, arXiv, Wikipedia)
├── templates/
│   └── index.html             # Web UI
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core API Endpoints

### Generate Research Report

```python
import requests

# Start a research task
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

### Poll Task Progress

```python
# Get real-time progress
progress = requests.get(f"http://localhost:8000/task_progress/{task_id}")
print(progress.json())
# Returns: {"status": "running", "current_step": "research", ...}
```

### Get Final Report

```python
# Get completed report
status = requests.get(f"http://localhost:8000/task_status/{task_id}")
report_data = status.json()
print(f"Status: {report_data['status']}")
print(f"Report: {report_data['report']}")
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

### Database Connection

The container automatically configures PostgreSQL. To connect from host:

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

## Building Custom Agents

### Creating a Planning Agent

```python
# src/planning_agent.py
from typing import List, Dict

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> List[Dict]:
    """Generate a multi-step research plan."""
    import aisuite as ai
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a research planner. Break down research tasks into steps."
        },
        {
            "role": "user",
            "content": f"Create a research plan for: {prompt}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    # Parse plan into structured steps
    plan = parse_plan(response.choices[0].message.content)
    return plan

def executor_agent_step(step: Dict, context: Dict, model: str) -> Dict:
    """Execute a single research step with available tools."""
    step_type = step.get("type")
    
    if step_type == "research":
        return research_agent(step["query"], context, model)
    elif step_type == "write":
        return writer_agent(step["topic"], context, model)
    elif step_type == "edit":
        return editor_agent(context["draft"], context, model)
    
    return {"error": f"Unknown step type: {step_type}"}
```

### Creating Research Tools

```python
# src/research_tools.py
import requests
import os

def tavily_search_tool(query: str, max_results: int = 5) -> List[Dict]:
    """Search using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        raise ValueError("TAVILY_API_KEY not set")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    response.raise_for_status()
    return response.json().get("results", [])

def arxiv_search_tool(query: str, max_results: int = 3) -> List[Dict]:
    """Search arXiv papers."""
    import arxiv
    
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for result in search.results():
        results.append({
            "title": result.title,
            "authors": [a.name for a in result.authors],
            "summary": result.summary,
            "pdf_url": result.pdf_url,
            "published": result.published.isoformat()
        })
    return results

def wikipedia_search_tool(query: str) -> Dict:
    """Search Wikipedia."""
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url,
            "content": page.content[:2000]  # Truncate
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option
        page = wikipedia.page(e.options[0])
        return {"title": page.title, "summary": page.summary, "url": page.url}
    except Exception as e:
        return {"error": str(e)}
```

### Creating Specialized Agents

```python
# src/agents.py
def research_agent(query: str, context: Dict, model: str) -> Dict:
    """Agent that conducts research using available tools."""
    from src.research_tools import (
        tavily_search_tool,
        arxiv_search_tool,
        wikipedia_search_tool
    )
    
    # Gather information from multiple sources
    tavily_results = tavily_search_tool(query, max_results=5)
    arxiv_results = arxiv_search_tool(query, max_results=3)
    wiki_result = wikipedia_search_tool(query)
    
    # Synthesize findings
    import aisuite as ai
    client = ai.Client()
    
    synthesis_prompt = f"""
    Research Query: {query}
    
    Tavily Results: {tavily_results}
    arXiv Papers: {arxiv_results}
    Wikipedia: {wiki_result}
    
    Synthesize these findings into a comprehensive research summary.
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a research synthesizer."},
            {"role": "user", "content": synthesis_prompt}
        ]
    )
    
    return {
        "query": query,
        "findings": response.choices[0].message.content,
        "sources": {
            "tavily": len(tavily_results),
            "arxiv": len(arxiv_results),
            "wikipedia": bool(wiki_result)
        }
    }

def writer_agent(topic: str, context: Dict, model: str) -> Dict:
    """Agent that writes research reports."""
    import aisuite as ai
    client = ai.Client()
    
    research_data = context.get("research_findings", "")
    
    messages = [
        {
            "role": "system",
            "content": "You are an expert technical writer. Create comprehensive, well-structured reports."
        },
        {
            "role": "user",
            "content": f"Write a research report on: {topic}\n\nResearch Data:\n{research_data}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return {
        "topic": topic,
        "draft": response.choices[0].message.content
    }

def editor_agent(draft: str, context: Dict, model: str) -> Dict:
    """Agent that edits and improves drafts."""
    import aisuite as ai
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are an expert editor. Improve clarity, structure, and accuracy."
        },
        {
            "role": "user",
            "content": f"Edit and improve this draft:\n\n{draft}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.5
    )
    
    return {
        "edited_report": response.choices[0].message.content
    }
```

## FastAPI Integration

### Main Application Setup

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
from datetime import datetime
import threading

app = FastAPI(title="Research Agent Service")
templates = Jinja2Templates(directory="templates")

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text)
    model = Column(String)
    status = Column(String, default="pending")
    current_step = Column(String)
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Create tables
Base.metadata.create_all(bind=engine)

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.get("/", response_class=HTMLResponse)
async def root(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})

@app.post("/generate_report")
async def generate_report(req: ResearchRequest, background_tasks: BackgroundTasks):
    task_id = str(uuid.uuid4())
    
    # Save task to DB
    db = SessionLocal()
    task = Task(task_id=task_id, prompt=req.prompt, model=req.model)
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background
    background_tasks.add_task(run_workflow, task_id, req.prompt, req.model)
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "current_step": task.current_step,
        "updated_at": task.updated_at.isoformat()
    }

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.task_id,
        "prompt": task.prompt,
        "status": task.status,
        "report": task.report,
        "created_at": task.created_at.isoformat()
    }

def run_workflow(task_id: str, prompt: str, model: str):
    """Execute the complete research workflow."""
    from src.planning_agent import planner_agent, executor_agent_step
    
    db = SessionLocal()
    
    try:
        # Update status
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "planning"
        task.current_step = "planning"
        db.commit()
        
        # Generate plan
        plan = planner_agent(prompt, model)
        
        # Execute each step
        context = {"prompt": prompt}
        for i, step in enumerate(plan):
            task.current_step = f"step_{i+1}_{step['type']}"
            db.commit()
            
            result = executor_agent_step(step, context, model)
            context.update(result)
        
        # Mark complete
        task.status = "completed"
        task.report = context.get("edited_report", context.get("draft", "No report generated"))
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    
    finally:
        db.close()
```

## Common Patterns

### Workflow with Custom Steps

```python
def custom_workflow(task_id: str, prompt: str):
    """Custom research workflow with specific steps."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    
    # Step 1: Background research
    task.current_step = "background_research"
    db.commit()
    background = research_agent(f"background on {prompt}", {}, "openai:gpt-4o")
    
    # Step 2: Deep dive research
    task.current_step = "deep_research"
    db.commit()
    deep_research = research_agent(f"detailed analysis of {prompt}", background, "openai:gpt-4o")
    
    # Step 3: Write report
    task.current_step = "writing"
    db.commit()
    draft = writer_agent(prompt, deep_research, "openai:gpt-4o")
    
    # Step 4: Edit
    task.current_step = "editing"
    db.commit()
    final = editor_agent(draft["draft"], {}, "openai:gpt-4o")
    
    # Save result
    task.status = "completed"
    task.report = final["edited_report"]
    db.commit()
    db.close()
```

### Adding Custom Research Tools

```python
# src/research_tools.py (extended)
def custom_api_tool(query: str) -> Dict:
    """Integrate a custom API as a research tool."""
    api_key = os.getenv("CUSTOM_API_KEY")
    
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    
    return response.json()

# Use in research agent
def enhanced_research_agent(query: str, context: Dict, model: str) -> Dict:
    """Research agent with custom tool."""
    # Use all available tools
    results = {
        "tavily": tavily_search_tool(query),
        "arxiv": arxiv_search_tool(query),
        "custom": custom_api_tool(query)
    }
    
    # Synthesize with LLM
    # ... synthesis logic
    return {"findings": synthesized_content}
```

## Troubleshooting

### Database Connection Issues

```python
# Verify DATABASE_URL is set
import os
print(f"DATABASE_URL: {os.getenv('DATABASE_URL')}")

# Test connection
from sqlalchemy import create_engine
engine = create_engine(os.getenv("DATABASE_URL"))
try:
    connection = engine.connect()
    print("✅ Database connection successful")
    connection.close()
except Exception as e:
    print(f"❌ Database error: {e}")
```

### API Key Not Found

```bash
# Verify .env file is loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Check from Python
import os
print(f"OPENAI_API_KEY: {'✅ Set' if os.getenv('OPENAI_API_KEY') else '❌ Not set'}")
print(f"TAVILY_API_KEY: {'✅ Set' if os.getenv('TAVILY_API_KEY') else '❌ Not set'}")
```

### Task Stuck in Running State

```python
# Check task status directly in database
import psycopg2
conn = psycopg2.connect("postgresql://app:local@localhost:5432/appdb")
cur = conn.cursor()
cur.execute("SELECT task_id, status, current_step FROM tasks WHERE status = 'running'")
print(cur.fetchall())
cur.close()
conn.close()

# Reset stuck task
db = SessionLocal()
task = db.query(Task).filter(Task.task_id == task_id).first()
task.status = "failed"
task.report = "Task timed out"
db.commit()
db.close()
```

### Tool Rate Limiting

```python
import time
from functools import wraps

def rate_limit(calls_per_minute: int):
    """Decorator to rate limit tool calls."""
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            if elapsed < min_interval:
                time.sleep(min_interval - elapsed)
            result = func(*args, **kwargs)
            last_called[0] = time.time()
            return result
        return wrapper
    return decorator

# Apply to tools
@rate_limit(calls_per_minute=10)
def tavily_search_tool(query: str, max_results: int = 5):
    # ... existing implementation
```

### Development Mode with Hot Reload

```bash
# Mount code and run with reload
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Viewing Container Logs

```bash
# Follow logs
docker logs -f fpsvc

# View specific component
docker exec -it fpsvc tail -f /var/log/postgresql/postgresql-17-main.log
```

This skill provides comprehensive guidance for implementing and extending the Agentic AI Research Agent service with custom workflows, tools, and agents.
