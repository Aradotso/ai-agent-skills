---
name: agentic-ai-research-agent
description: FastAPI-based reflective research agent with Postgres that plans multi-step workflows using Tavily, arXiv, and Wikipedia tools
triggers:
  - set up the agentic AI research agent service
  - create a research workflow with planning and execution agents
  - implement a multi-step research task with Tavily and arXiv
  - build a FastAPI research agent with Postgres state management
  - configure the reflective research agent with tool-using capabilities
  - deploy the agentic workflow research service
  - integrate planning agent with research writer and editor agents
  - use the research agent API to generate reports
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI web service that orchestrates multi-step research workflows using AI agents. It features a planning agent that breaks down research tasks, executor agents that gather information from multiple sources (Tavily, arXiv, Wikipedia), and a Postgres database for state management. The system runs reflective workflows with research, writing, and editing phases.

**Key capabilities:**
- Multi-step agent workflow planning and execution
- Integration with Tavily search, arXiv papers, and Wikipedia
- Task state persistence with Postgres
- Real-time progress tracking via REST API
- Threaded execution for non-blocking operations
- Web UI for kicking off research tasks

## Installation

### Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Create .env file with API keys
cat > .env << EOF
OPENAI_API_KEY=${OPENAI_API_KEY}
TAVILY_API_KEY=${TAVILY_API_KEY}
EOF

# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the container (includes Postgres + API)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

### Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres separately
# Then set DATABASE_URL
export DATABASE_URL="postgresql://user:pass@localhost:5432/appdb"
export OPENAI_API_KEY="your-key"
export TAVILY_API_KEY="your-key"

# Run the app
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Project Structure

```
.
├── main.py                    # FastAPI application entry point
├── src/
│   ├── planning_agent.py      # planner_agent(), executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # tavily, arxiv, wikipedia tool functions
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
└── Dockerfile
```

## Core API Endpoints

### Start Research Task

```python
import requests

# Kick off a research workflow
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Large Language Models for scientific discovery",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task started: {task_id}")
```

### Poll Progress

```python
import requests
import time

def poll_progress(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = response.json()
        
        print(f"Status: {data['status']}")
        print(f"Current step: {data.get('current_step', 'N/A')}")
        
        if data['status'] in ['completed', 'failed']:
            break
        
        time.sleep(5)
```

### Get Final Report

```python
import requests

def get_final_report(task_id):
    response = requests.get(f"http://localhost:8000/task_status/{task_id}")
    data = response.json()
    
    if data['status'] == 'completed':
        print(data['final_report'])
    else:
        print(f"Task status: {data['status']}")
        print(f"Error: {data.get('error_message', 'N/A')}")
```

## Agent Architecture

### Planning Agent

The planning agent breaks down research tasks into steps:

```python
# src/planning_agent.py
from typing import List, Dict

def planner_agent(prompt: str, model: str) -> List[Dict]:
    """
    Generates a multi-step research plan.
    
    Returns list of steps like:
    [
        {"step": 1, "action": "research", "query": "LLM architectures"},
        {"step": 2, "action": "write", "topic": "Technical overview"},
        {"step": 3, "action": "edit", "focus": "clarity"}
    ]
    """
    # Implementation using AI model to generate plan
    pass

def executor_agent_step(step: Dict, context: str) -> str:
    """
    Executes a single step from the plan.
    Routes to appropriate agent based on action.
    """
    action = step.get("action")
    
    if action == "research":
        return research_agent(step["query"], context)
    elif action == "write":
        return writer_agent(step["topic"], context)
    elif action == "edit":
        return editor_agent(context, step["focus"])
```

### Research Agent

```python
# src/agents.py
from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

def research_agent(query: str, context: str = "") -> str:
    """
    Performs research using multiple tools.
    """
    results = []
    
    # Search Tavily
    try:
        tavily_results = tavily_search_tool(query)
        results.append(f"Web search results:\n{tavily_results}")
    except Exception as e:
        print(f"Tavily search failed: {e}")
    
    # Search arXiv
    try:
        arxiv_results = arxiv_search_tool(query, max_results=5)
        results.append(f"Academic papers:\n{arxiv_results}")
    except Exception as e:
        print(f"arXiv search failed: {e}")
    
    # Search Wikipedia
    try:
        wiki_results = wikipedia_search_tool(query)
        results.append(f"Wikipedia summary:\n{wiki_results}")
    except Exception as e:
        print(f"Wikipedia search failed: {e}")
    
    return "\n\n".join(results)
```

### Writer and Editor Agents

```python
# src/agents.py
def writer_agent(topic: str, research_context: str) -> str:
    """
    Synthesizes research into a draft report.
    """
    # Use LLM to generate content from research context
    pass

def editor_agent(draft: str, focus: str) -> str:
    """
    Refines and improves the draft.
    focus: 'clarity', 'conciseness', 'technical_accuracy', etc.
    """
    # Use LLM to edit and improve the draft
    pass
```

## Research Tools

### Tavily Search

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> str:
    """
    Searches the web using Tavily API.
    """
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return "Tavily API key not configured"
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    
    if response.status_code == 200:
        results = response.json().get("results", [])
        return "\n".join([f"- {r['title']}: {r['content']}" for r in results])
    
    return f"Tavily search failed: {response.status_code}"
```

### arXiv Search

```python
# src/research_tools.py
import requests
import xml.etree.ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5) -> str:
    """
    Searches arXiv for academic papers.
    """
    base_url = "http://export.arxiv.org/api/query"
    params = {
        "search_query": f"all:{query}",
        "start": 0,
        "max_results": max_results
    }
    
    response = requests.get(base_url, params=params)
    
    if response.status_code != 200:
        return f"arXiv search failed: {response.status_code}"
    
    root = ET.fromstring(response.content)
    entries = root.findall("{http://www.w3.org/2005/Atom}entry")
    
    papers = []
    for entry in entries:
        title = entry.find("{http://www.w3.org/2005/Atom}title").text
        summary = entry.find("{http://www.w3.org/2005/Atom}summary").text
        papers.append(f"**{title}**\n{summary[:200]}...")
    
    return "\n\n".join(papers)
```

### Wikipedia Search

```python
# src/research_tools.py
import wikipedia

def wikipedia_search_tool(query: str) -> str:
    """
    Searches Wikipedia and returns summary.
    """
    try:
        # Set user agent to avoid rate limiting
        wikipedia.set_user_agent("ResearchAgent/1.0")
        
        # Get page summary
        summary = wikipedia.summary(query, sentences=5, auto_suggest=True)
        return summary
    except wikipedia.exceptions.DisambiguationError as e:
        # Multiple options, pick first
        return wikipedia.summary(e.options[0], sentences=5)
    except wikipedia.exceptions.PageError:
        return f"No Wikipedia page found for: {query}"
    except Exception as e:
        return f"Wikipedia search error: {str(e)}"
```

## Database Models

```python
# main.py
from sqlalchemy import Column, String, DateTime, Text, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    current_step = Column(String, nullable=True)
    final_report = Column(Text, nullable=True)
    error_message = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Setup database
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## FastAPI Application

```python
# main.py
from fastapi import FastAPI, HTTPException
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates
from pydantic import BaseModel
import uuid
import threading

app = FastAPI(title="Research Agent Service")
templates = Jinja2Templates(directory="templates")

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.get("/", response_class=HTMLResponse)
async def root(request: Request):
    """Serve the web UI."""
    return templates.TemplateResponse("index.html", {"request": request})

@app.post("/generate_report")
async def generate_report(req: ResearchRequest):
    """Start a new research task."""
    task_id = str(uuid.uuid4())
    
    # Create task in database
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        prompt=req.prompt,
        model=req.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, req.prompt, req.model)
    )
    thread.daemon = True
    thread.start()
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """Get current progress of a task."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "current_step": task.current_step,
        "created_at": task.created_at.isoformat()
    }

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get final status and report."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "final_report": task.final_report,
        "error_message": task.error_message,
        "created_at": task.created_at.isoformat(),
        "updated_at": task.updated_at.isoformat()
    }

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Execute the research workflow in background."""
    db = SessionLocal()
    
    try:
        # Update status to running
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "running"
        db.commit()
        
        # Step 1: Planning
        task.current_step = "Planning research workflow"
        db.commit()
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute each step
        context = ""
        for step in plan:
            task.current_step = f"Step {step['step']}: {step['action']}"
            db.commit()
            
            result = executor_agent_step(step, context)
            context += f"\n\n{result}"
        
        # Step 3: Complete
        task.status = "completed"
        task.final_report = context
        task.current_step = "Completed"
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error_message = str(e)
        db.commit()
    
    finally:
        db.close()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Optional - Database (defaults work for Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional - Development
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on startup
```

### Docker Compose (Optional)

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
      - "5432:5432"
    env_file:
      - .env
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

## Common Patterns

### Custom Agent Implementation

```python
# Add a new specialized agent
def code_review_agent(code: str, language: str) -> str:
    """
    Specialized agent for code review.
    """
    prompt = f"""
    Review this {language} code for:
    - Security issues
    - Performance problems
    - Best practices
    
    Code:
    {code}
    """
    # Call LLM with prompt
    return llm_call(prompt)

# Register in executor
def executor_agent_step(step: Dict, context: str) -> str:
    action = step.get("action")
    
    if action == "code_review":
        return code_review_agent(step["code"], step["language"])
    # ... other agents
```

### Streaming Progress Updates

```python
from fastapi import WebSocket

@app.websocket("/ws/task/{task_id}")
async def websocket_progress(websocket: WebSocket, task_id: str):
    """Stream real-time progress updates via WebSocket."""
    await websocket.accept()
    
    db = SessionLocal()
    last_step = None
    
    try:
        while True:
            task = db.query(Task).filter(Task.task_id == task_id).first()
            
            if task.current_step != last_step:
                await websocket.send_json({
                    "status": task.status,
                    "current_step": task.current_step
                })
                last_step = task.current_step
            
            if task.status in ["completed", "failed"]:
                break
            
            await asyncio.sleep(1)
    finally:
        db.close()
        await websocket.close()
```

### Batch Processing

```python
@app.post("/batch_research")
async def batch_research(prompts: List[str], model: str = "openai:gpt-4o"):
    """Process multiple research tasks in parallel."""
    task_ids = []
    
    for prompt in prompts:
        task_id = str(uuid.uuid4())
        db = SessionLocal()
        task = Task(task_id=task_id, prompt=prompt, model=model)
        db.add(task)
        db.commit()
        db.close()
        
        thread = threading.Thread(
            target=run_research_workflow,
            args=(task_id, prompt, model)
        )
        thread.daemon = True
        thread.start()
        
        task_ids.append(task_id)
    
    return {"task_ids": task_ids}
```

## Troubleshooting

### Database Connection Issues

```python
# Test database connectivity
from sqlalchemy import text

def test_db_connection():
    try:
        engine = create_engine(DATABASE_URL)
        with engine.connect() as conn:
            result = conn.execute(text("SELECT 1"))
            print("Database connected successfully")
    except Exception as e:
        print(f"Database connection failed: {e}")
```

### API Key Not Found

```bash
# Verify environment variables are loaded
docker exec -it fpsvc bash -c 'echo $OPENAI_API_KEY'

# Check .env file is in the right location
docker exec -it fpsvc bash -c 'ls -la /app/.env'
```

### Tavily Rate Limiting

```python
# Add retry logic for Tavily
import time
from functools import wraps

def retry_on_rate_limit(max_retries=3, delay=5):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if "rate limit" in str(e).lower() and attempt < max_retries - 1:
                        time.sleep(delay * (attempt + 1))
                    else:
                        raise
        return wrapper
    return decorator

@retry_on_rate_limit()
def tavily_search_tool(query: str, max_results: int = 5) -> str:
    # Implementation
    pass
```

### Task Stuck in Running State

```python
# Add timeout mechanism
import signal
from contextlib import contextmanager

@contextmanager
def timeout(seconds):
    def timeout_handler(signum, frame):
        raise TimeoutError()
    
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(seconds)
    try:
        yield
    finally:
        signal.alarm(0)

def run_research_workflow(task_id: str, prompt: str, model: str):
    db = SessionLocal()
    
    try:
        with timeout(600):  # 10 minute timeout
            # Execute workflow
            pass
    except TimeoutError:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "failed"
        task.error_message = "Task timed out after 10 minutes"
        db.commit()
    finally:
        db.close()
```

### View Container Logs

```bash
# Follow logs in real-time
docker logs -f fpsvc

# View last 100 lines
docker logs --tail 100 fpsvc

# Search for specific errors
docker logs fpsvc 2>&1 | grep -i error
```

### Connect to Database Directly

```bash
# From host machine
psql "postgresql://app:local@localhost:5432/appdb"

# Inside container
docker exec -it fpsvc psql -U app -d appdb

# Query task status
docker exec -it fpsvc psql -U app -d appdb -c "SELECT task_id, status, current_step FROM tasks;"
```

### Clear Stale Tasks

```python
# Utility endpoint to clean up old tasks
from datetime import timedelta

@app.post("/admin/cleanup_tasks")
async def cleanup_tasks(days_old: int = 7):
    """Remove tasks older than specified days."""
    db = SessionLocal()
    cutoff = datetime.utcnow() - timedelta(days=days_old)
    
    deleted = db.query(Task).filter(Task.created_at < cutoff).delete()
    db.commit()
    db.close()
    
    return {"deleted_tasks": deleted}
```
