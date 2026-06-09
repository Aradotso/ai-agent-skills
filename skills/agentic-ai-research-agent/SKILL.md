---
name: agentic-ai-research-agent
description: Research Agent service using FastAPI, Postgres, and AI tools (Tavily, arXiv, Wikipedia) for multi-step agentic workflows
triggers:
  - how do I build a research agent with planning and execution
  - set up the agentic AI research workflow service
  - deploy a multi-agent research system with FastAPI
  - create a reflective research agent with tool-using capabilities
  - implement agentic workflow with planner and executor agents
  - build a research report generator using AI agents
  - configure Tavily arXiv Wikipedia tools for AI research
  - run the deeplearning.ai agentic research service
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This project provides a FastAPI web service that orchestrates multi-step AI research workflows. It uses a planning agent to decompose research tasks, executes them with specialized agents (research, writer, editor), and stores task state/results in Postgres. The system integrates with Tavily (web search), arXiv (academic papers), and Wikipedia tools.

## What It Does

- **Multi-agent workflow**: Planner agent creates research steps, executor agents perform tasks
- **Tool integration**: Web search (Tavily), academic papers (arXiv), encyclopedic knowledge (Wikipedia)
- **Async task tracking**: Track progress of multi-step research workflows via task IDs
- **Persistent storage**: Postgres stores task state, intermediate results, and final reports
- **Web UI + API**: Simple web interface and REST API for kicking off and monitoring research tasks

## Installation

### Prerequisites

- Docker
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build and Run with Docker

```bash
# Build the image
docker build -t fastapi-postgres-service .

# Run the service (exposes both API and Postgres)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will:
1. Start Postgres cluster
2. Create database and user
3. Initialize tables via SQLAlchemy
4. Launch FastAPI on port 8000

### Access Points

- **Web UI**: http://localhost:8000/
- **API Docs**: http://localhost:8000/docs
- **Postgres**: `postgresql://app:local@localhost:5432/appdb`

## Project Structure

```
.
├── main.py                      # FastAPI app with endpoints
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

## API Usage

### Start a Research Task

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
print(f"Task started: {task_id}")
```

```bash
# Via curl
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications in drug discovery", "model":"openai:gpt-4o"}'
```

### Poll Task Progress

```python
import requests
import time

def poll_progress(task_id):
    while True:
        resp = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = resp.json()
        
        print(f"Status: {data['status']}")
        print(f"Current step: {data.get('current_step', 'N/A')}")
        
        if data['status'] in ['completed', 'failed']:
            break
        
        time.sleep(2)

poll_progress(task_id)
```

### Retrieve Final Report

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result['status'] == 'completed':
    print("Report:")
    print(result['report'])
else:
    print(f"Task {result['status']}: {result.get('error', 'No error info')}")
```

## Building Custom Agents

### Research Tool Example

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    """Search the web using Tavily API"""
    api_key = os.getenv("TAVILY_API_KEY")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "search_depth": "advanced",
            "max_results": max_results
        }
    )
    
    return response.json()


def arxiv_search_tool(query: str, max_results: int = 3) -> list:
    """Search arXiv for academic papers"""
    import arxiv
    
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for paper in search.results():
        results.append({
            "title": paper.title,
            "authors": [a.name for a in paper.authors],
            "summary": paper.summary,
            "url": paper.entry_id,
            "published": paper.published.isoformat()
        })
    
    return results


def wikipedia_search_tool(query: str) -> dict:
    """Get Wikipedia summary"""
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": "disambiguation", "options": e.options[:5]}
    except wikipedia.exceptions.PageError:
        return {"error": "not_found"}
```

### Planning Agent Pattern

```python
# src/planning_agent.py
from typing import List, Dict
import json

def planner_agent(research_prompt: str, ai_client, model: str) -> List[Dict]:
    """
    Generate a research plan with discrete steps
    Returns list of steps: [{"type": "research", "description": "..."}, ...]
    """
    
    system_prompt = """You are a research planning agent. Break down the research task into steps.
Output JSON array of steps with: {"type": "research|write|edit", "description": "...", "tools": [...]}
Available tools: tavily_search, arxiv_search, wikipedia_search
"""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": f"Create research plan for: {research_prompt}"}
    ]
    
    response = ai_client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    plan_text = response.choices[0].message.content
    
    # Parse JSON plan
    try:
        plan = json.loads(plan_text)
        return plan
    except json.JSONDecodeError:
        # Fallback: simple 3-step plan
        return [
            {"type": "research", "description": "Research the topic", "tools": ["tavily_search", "arxiv_search"]},
            {"type": "write", "description": "Write comprehensive report", "tools": []},
            {"type": "edit", "description": "Edit and refine report", "tools": []}
        ]


def executor_agent_step(step: Dict, context: str, ai_client, model: str, tools_map: Dict) -> str:
    """
    Execute a single research step
    Returns the result/output of this step
    """
    
    step_type = step.get("type", "research")
    description = step.get("description", "")
    tool_names = step.get("tools", [])
    
    # Gather tool results
    tool_results = []
    for tool_name in tool_names:
        if tool_name in tools_map:
            tool_fn = tools_map[tool_name]
            try:
                result = tool_fn(description)
                tool_results.append({
                    "tool": tool_name,
                    "result": result
                })
            except Exception as e:
                tool_results.append({
                    "tool": tool_name,
                    "error": str(e)
                })
    
    # Generate agent response using tool results + context
    system_prompt = f"""You are a {step_type} agent. 
Previous context: {context}
Tool results: {json.dumps(tool_results, indent=2)}

Task: {description}
"""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": f"Complete this step: {description}"}
    ]
    
    response = ai_client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

### Main FastAPI Application Pattern

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates
from pydantic import BaseModel
import uuid
import os
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

app = FastAPI()
templates = Jinja2Templates(directory="templates")

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()


class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text)
    model = Column(String)
    status = Column(String)  # pending, running, completed, failed
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)


Base.metadata.create_all(bind=engine)


class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"


def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background task that runs the full research workflow"""
    from src.planning_agent import planner_agent, executor_agent_step
    from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    import aisuite as ai
    
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    try:
        task.status = "running"
        db.commit()
        
        # Initialize AI client
        client = ai.Client()
        
        # Tool mapping
        tools_map = {
            "tavily_search": tavily_search_tool,
            "arxiv_search": arxiv_search_tool,
            "wikipedia_search": wikipedia_search_tool
        }
        
        # Step 1: Planning
        task.current_step = "Planning research workflow"
        db.commit()
        
        plan = planner_agent(prompt, client, model)
        
        # Step 2: Execute each step
        context = f"Research topic: {prompt}\n\n"
        
        for i, step in enumerate(plan):
            task.current_step = f"Step {i+1}/{len(plan)}: {step.get('description', 'Processing')}"
            db.commit()
            
            step_result = executor_agent_step(step, context, client, model, tools_map)
            context += f"\n\n{step.get('description')}:\n{step_result}"
        
        # Final result
        task.status = "completed"
        task.report = context
        task.current_step = "Completed"
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
    
    finally:
        db.commit()
        db.close()


@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """Start a new research task"""
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    task = Task(
        id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    background_tasks.add_task(run_research_workflow, task_id, request.prompt, request.model)
    
    return {"task_id": task_id}


@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """Get current progress of a task"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.id,
        "status": task.status,
        "current_step": task.current_step,
        "created_at": task.created_at.isoformat()
    }


@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get final status and report"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.id,
        "status": task.status,
        "report": task.report,
        "error": task.error,
        "created_at": task.created_at.isoformat()
    }
```

## Configuration

### Environment Variables

Required:
- `OPENAI_API_KEY`: OpenAI API key for LLM calls
- `TAVILY_API_KEY`: Tavily API key for web search

Optional:
- `DATABASE_URL`: Postgres connection string (default: `postgresql://app:local@127.0.0.1:5432/appdb`)
- `POSTGRES_USER`: Database user (default: `app`)
- `POSTGRES_PASSWORD`: Database password (default: `local`)
- `POSTGRES_DB`: Database name (default: `appdb`)

### Disable Table Reset on Startup

By default, the app may drop/recreate tables on startup. To disable:

```python
# In main.py
import os

# Only reset DB if explicitly requested
if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)

Base.metadata.create_all(bind=engine)
```

## Common Patterns

### Custom Agent Implementation

```python
# src/agents.py
def research_agent(query: str, ai_client, model: str, tools: list) -> str:
    """Specialized research agent that gathers information"""
    
    # Gather results from all tools
    all_results = []
    for tool_fn in tools:
        result = tool_fn(query)
        all_results.append(result)
    
    # Synthesize results
    messages = [
        {"role": "system", "content": "You are a research agent. Synthesize the tool results into coherent findings."},
        {"role": "user", "content": f"Query: {query}\n\nTool results: {json.dumps(all_results, indent=2)}"}
    ]
    
    response = ai_client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content


def writer_agent(research_context: str, ai_client, model: str) -> str:
    """Write comprehensive report from research"""
    
    messages = [
        {"role": "system", "content": "You are a technical writer. Create a comprehensive, well-structured report."},
        {"role": "user", "content": f"Research findings:\n{research_context}\n\nWrite a detailed report."}
    ]
    
    response = ai_client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content


def editor_agent(draft: str, ai_client, model: str) -> str:
    """Edit and refine the report"""
    
    messages = [
        {"role": "system", "content": "You are an editor. Improve clarity, fix errors, enhance structure."},
        {"role": "user", "content": f"Draft report:\n{draft}\n\nEdit and refine this report."}
    ]
    
    response = ai_client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.5
    )
    
    return response.choices[0].message.content
```

### Database Query Patterns

```python
from sqlalchemy.orm import Session

def get_recent_tasks(db: Session, limit: int = 10):
    """Get recent tasks"""
    return db.query(Task).order_by(Task.created_at.desc()).limit(limit).all()

def get_tasks_by_status(db: Session, status: str):
    """Get all tasks with given status"""
    return db.query(Task).filter(Task.status == status).all()

def cleanup_old_tasks(db: Session, days: int = 7):
    """Delete tasks older than N days"""
    from datetime import timedelta
    cutoff = datetime.utcnow() - timedelta(days=days)
    db.query(Task).filter(Task.created_at < cutoff).delete()
    db.commit()
```

## Troubleshooting

### Container Won't Start

Check logs:
```bash
docker logs -f fpsvc
```

Verify Postgres is running inside container:
```bash
docker exec -it fpsvc bash -lc "pg_isready"
```

### Database Connection Issues

Test connection from host:
```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

Check `DATABASE_URL` is set correctly:
```bash
docker exec -it fpsvc bash -lc "echo \$DATABASE_URL"
```

### Tool API Errors

**Tavily 401/403**: Verify `TAVILY_API_KEY` is set and valid
```bash
docker exec -it fpsvc bash -lc "echo \$TAVILY_API_KEY"
```

**Wikipedia rate limiting**: Add retry logic or delays:
```python
import time
import wikipedia

def wikipedia_search_tool_safe(query: str, retries: int = 3):
    for attempt in range(retries):
        try:
            return wikipedia_search_tool(query)
        except Exception as e:
            if attempt < retries - 1:
                time.sleep(2 ** attempt)
            else:
                raise e
```

### Tasks Stuck in "Running"

Tasks may get stuck if the background worker crashes. Add timeout logic:

```python
from datetime import timedelta

def cleanup_stuck_tasks(db: Session, timeout_minutes: int = 30):
    """Mark tasks as failed if running too long"""
    cutoff = datetime.utcnow() - timedelta(minutes=timeout_minutes)
    
    stuck_tasks = db.query(Task).filter(
        Task.status == "running",
        Task.updated_at < cutoff
    ).all()
    
    for task in stuck_tasks:
        task.status = "failed"
        task.error = "Task timeout"
    
    db.commit()
    return len(stuck_tasks)
```

### Hot Reload for Development

Mount source code and run with reload:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Check Template/Static Files

Verify files are in container:
```bash
docker exec -it fpsvc bash -lc "ls -la /app/templates && ls -la /app/static"
```
