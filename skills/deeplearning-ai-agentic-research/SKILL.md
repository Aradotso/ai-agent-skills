---
name: deeplearning-ai-agentic-research
description: FastAPI research agent service with multi-step planning, Tavily/arXiv/Wikipedia tools, and Postgres task tracking
triggers:
  - set up the agentic research agent service
  - how do I run the research agent with FastAPI
  - create a research workflow with planning agent
  - integrate Tavily and arXiv search tools
  - build a multi-step research task with progress tracking
  - deploy the reflective research agent API
  - use the research agent to generate reports
  - configure the agentic AI research service
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step research workflows using planning agents, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres for task state management. The system breaks down research queries into planned steps, executes them with specialized agents (researcher, writer, editor), and tracks progress in real-time.

## What This Project Does

- **Multi-Agent Research Pipeline**: Planner agent creates workflow → Research agent gathers data → Writer agent drafts → Editor agent refines
- **Tool Integration**: Tavily web search, arXiv academic papers, Wikipedia knowledge base
- **Task Management**: Postgres-backed task tracking with live progress updates
- **REST API**: FastAPI endpoints for kicking off research, polling progress, retrieving results
- **Single-Container Deploy**: Runs Postgres + FastAPI in one Docker container for local development

## Installation

### Prerequisites

- Docker (Desktop or Engine)
- API keys for OpenAI and Tavily

### Environment Setup

Create a `.env` file in the project root:

```bash
# .env
OPENAI_API_KEY=sk-your-openai-key
TAVILY_API_KEY=tvly-your-tavily-key
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

The container's entrypoint automatically:
- Starts Postgres cluster
- Creates application user and database
- Runs database migrations
- Launches FastAPI with Uvicorn

## Key API Endpoints

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
print(f"Task started: {task_id}")
```

### Poll Task Progress

```python
# Check live progress (substeps, tool calls, etc.)
progress = requests.get(f"http://localhost:8000/task_progress/{task_id}")
print(progress.json())
# Returns: {"status": "running", "steps": [...], "current_step": "research"}
```

### Get Final Report

```python
# Retrieve completed research report
status = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = status.json()
if result["status"] == "completed":
    print(result["report"])
```

### UI Access

- Web Interface: `http://localhost:8000/`
- API Docs (Swagger): `http://localhost:8000/docs`

## Project Structure

```
.
├── main.py                    # FastAPI app, endpoints, DB models
├── src/
│   ├── planning_agent.py      # planner_agent(), executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html             # Web UI
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
├── Dockerfile
└── .env                       # API keys (not committed)
```

## Core Patterns

### Creating a Research Tool

```python
# src/research_tools.py
import os
from typing import Dict, Any

def tavily_search_tool(query: str, max_results: int = 5) -> Dict[str, Any]:
    """Search the web using Tavily API"""
    import requests
    
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    return response.json()

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv for academic papers"""
    import requests
    from xml.etree import ElementTree as ET
    
    url = f"http://export.arxiv.org/api/query?search_query=all:{query}&max_results={max_results}"
    response = requests.get(url)
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

### Building a Planning Agent

```python
# src/planning_agent.py
import os
import aisuite as ai

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> dict:
    """Create a multi-step research plan"""
    client = ai.Client()
    
    planning_prompt = f"""
    You are a research planning agent. Break down this research task into steps:
    "{prompt}"
    
    Provide a JSON plan with steps like:
    {{"steps": [
        {{"type": "research", "query": "...", "tools": ["tavily", "arxiv"]}},
        {{"type": "write", "instruction": "..."}},
        {{"type": "edit", "focus": "..."}}
    ]}}
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": planning_prompt}],
        response_format={"type": "json_object"}
    )
    
    import json
    return json.loads(response.choices[0].message.content)

def executor_agent_step(step: dict, context: dict) -> dict:
    """Execute a single planned step"""
    from src.research_tools import tavily_search_tool, arxiv_search_tool
    
    if step["type"] == "research":
        results = {}
        for tool in step["tools"]:
            if tool == "tavily":
                results["tavily"] = tavily_search_tool(step["query"])
            elif tool == "arxiv":
                results["arxiv"] = arxiv_search_tool(step["query"])
        return {"type": "research", "results": results}
    
    elif step["type"] == "write":
        # Use writer_agent to draft content
        from src.agents import writer_agent
        return writer_agent(step["instruction"], context)
    
    elif step["type"] == "edit":
        # Use editor_agent to refine
        from src.agents import editor_agent
        return editor_agent(context["draft"], step["focus"])
```

### Database Models (SQLAlchemy)

```python
# main.py
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text)
    model = Column(String)
    status = Column(String)  # pending, running, completed, failed
    current_step = Column(String)
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Create tables
Base.metadata.create_all(bind=engine)
```

### FastAPI Endpoint Implementation

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import uuid
import threading

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background task that runs the full research workflow"""
    from src.planning_agent import planner_agent, executor_agent_step
    
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    
    try:
        # Update status
        task.status = "running"
        task.current_step = "planning"
        db.commit()
        
        # Generate plan
        plan = planner_agent(prompt, model)
        
        # Execute steps
        context = {}
        for i, step in enumerate(plan["steps"]):
            task.current_step = f"step_{i}_{step['type']}"
            db.commit()
            
            result = executor_agent_step(step, context)
            context[f"step_{i}"] = result
        
        # Final report
        task.report = context.get("final_report", "Research completed")
        task.status = "completed"
        task.current_step = "done"
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
    
    finally:
        db.commit()
        db.close()

@app.post("/generate_report")
def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """Start a research task"""
    task_id = str(uuid.uuid4())
    
    # Create DB record
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run in background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
def task_progress(task_id: str):
    """Get live progress of a task"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "current_step": task.current_step,
        "created_at": task.created_at.isoformat()
    }

@app.get("/task_status/{task_id}")
def task_status(task_id: str):
    """Get final status and report"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "report": task.report,
        "prompt": task.prompt,
        "model": task.model
    }
```

## Configuration

### Database Connection

Override the default DATABASE_URL:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -e DATABASE_URL="postgresql://user:pass@host:5432/db" \
  --env-file .env \
  fastapi-postgres-service
```

### Postgres Credentials

Set custom database credentials:

```bash
# In .env or via -e flags
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
POSTGRES_DB=research_db
```

### Disable DB Reset on Startup

By default, `main.py` may drop tables on startup (dev mode):

```python
# main.py
import os

# Guard the drop operation
if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)

Base.metadata.create_all(bind=engine)
```

Set `RESET_DB_ON_STARTUP=0` to preserve data between restarts.

## Common Patterns

### Custom Agent Implementation

```python
# src/agents.py
import os
import aisuite as ai

def research_agent(query: str, context: dict, tools: list) -> dict:
    """Gather information using specified tools"""
    from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    
    results = []
    
    if "tavily" in tools:
        web_results = tavily_search_tool(query)
        results.append({"source": "tavily", "data": web_results})
    
    if "arxiv" in tools:
        papers = arxiv_search_tool(query)
        results.append({"source": "arxiv", "data": papers})
    
    if "wikipedia" in tools:
        wiki_results = wikipedia_search_tool(query)
        results.append({"source": "wikipedia", "data": wiki_results})
    
    return {"query": query, "results": results}

def writer_agent(instruction: str, context: dict, model: str = "openai:gpt-4o") -> dict:
    """Draft content based on research"""
    client = ai.Client()
    
    research_data = context.get("research", {})
    prompt = f"""
    {instruction}
    
    Based on this research:
    {research_data}
    
    Write a comprehensive draft.
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return {"draft": response.choices[0].message.content}

def editor_agent(draft: str, focus: str, model: str = "openai:gpt-4o") -> dict:
    """Refine and edit the draft"""
    client = ai.Client()
    
    prompt = f"""
    Edit this draft with focus on: {focus}
    
    Draft:
    {draft}
    
    Provide an improved version.
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return {"final_report": response.choices[0].message.content}
```

### Connecting to Postgres from Host

```bash
# Install psql on host, then:
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT task_id, status, current_step FROM tasks ORDER BY created_at DESC;
```

### Hot Reload for Development

```bash
# Mount source code and run with --reload
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && sleep 2 && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Troubleshooting

### Container Fails to Start Postgres

**Symptom**: Logs show permission errors or cluster not found

**Solution**: Ensure the entrypoint script has correct permissions:

```bash
# In Dockerfile
COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
```

### API Returns "DATABASE_URL not set"

**Symptom**: App crashes on startup with database connection error

**Solution**: Verify the entrypoint exports DATABASE_URL:

```bash
# docker/entrypoint.sh
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
```

### Tavily Search Fails

**Symptom**: Research agent returns `{"error": "TAVILY_API_KEY not set"}`

**Solution**: Ensure `.env` file is passed to container:

```bash
# Check env inside container
docker exec -it fpsvc env | grep TAVILY

# If missing, rebuild with --env-file
docker run --env-file .env ...
```

### arXiv Rate Limiting

**Symptom**: HTTP 429 errors from arXiv API

**Solution**: Add exponential backoff in `arxiv_search_tool`:

```python
import time
import requests

def arxiv_search_tool(query: str, max_results: int = 5, retry: int = 3):
    for attempt in range(retry):
        try:
            response = requests.get(url, timeout=10)
            if response.status_code == 200:
                return parse_results(response)
        except:
            pass
        
        time.sleep(2 ** attempt)  # 1s, 2s, 4s
    
    return {"error": "arXiv request failed after retries"}
```

### Tasks Stuck in "running" Status

**Symptom**: Task never completes, `current_step` doesn't update

**Solution**: Add exception handling and timeouts:

```python
import signal

def timeout_handler(signum, frame):
    raise TimeoutError("Step exceeded time limit")

def executor_agent_step(step: dict, context: dict):
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(300)  # 5 minute timeout per step
    
    try:
        # ... execute step
        pass
    except TimeoutError:
        return {"error": "Step timeout"}
    finally:
        signal.alarm(0)
```

### Web UI Not Loading

**Symptom**: `http://localhost:8000/` returns 404 or template error

**Solution**: Verify templates are copied in Dockerfile:

```dockerfile
# Dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

Check inside container:

```bash
docker exec -it fpsvc ls -la /app/templates
```

### Postgres Data Persistence

**Symptom**: Tasks disappear after container restart

**Solution**: Use Docker volumes:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v postgres-data:/var/lib/postgresql \
  --env-file .env \
  fastapi-postgres-service
```

Or disable table drops (see Configuration section).
