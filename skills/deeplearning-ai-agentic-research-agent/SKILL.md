---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with multi-step planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres state management
triggers:
  - how do I use the deeplearning.ai research agent
  - set up agentic research workflow with FastAPI
  - create a reflective research agent with planning
  - use Tavily arXiv Wikipedia tools in agent workflow
  - implement multi-step agent task with Postgres
  - build research agent with planner and executor
  - deploy FastAPI agent service with Docker
  - track agent task progress with PostgreSQL
---

# deeplearning-ai-agentic-research-agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The deeplearning.ai Agentic AI Research Agent is a FastAPI-based service that orchestrates multi-step research workflows using planning and execution agents. It combines:

- **Planning Agent**: Breaks down research tasks into subtasks
- **Executor Agents**: Research, writer, and editor agents that use external tools
- **Tool Integration**: Tavily search, arXiv papers, Wikipedia
- **State Management**: PostgreSQL for tracking task progress and results
- **Web UI**: Jinja2 template for submitting research queries

The service runs asynchronously, stores intermediate results, and provides real-time progress tracking via REST API.

## Installation

### Prerequisites

- Docker and Docker Compose
- API keys for OpenAI and Tavily

### Setup

1. **Clone the repository:**

```bash
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file:**

```bash
# .env
OPENAI_API_KEY=your_openai_key_here
TAVILY_API_KEY=your_tavily_key_here
```

3. **Build and run with Docker:**

```bash
docker build -t fastapi-postgres-service .
docker run --rm -it -p 8000:8000 -p 5432:5432 --name fpsvc --env-file .env fastapi-postgres-service
```

The service will:
- Start PostgreSQL on port 5432
- Initialize database schema
- Launch FastAPI on port 8000

## Project Structure

```
src/
├── planning_agent.py      # Planner and executor logic
├── agents.py              # Research, writer, editor agents
└── research_tools.py      # Tavily, arXiv, Wikipedia tools
templates/
└── index.html             # Web UI
main.py                    # FastAPI app entry point
requirements.txt           # Python dependencies
Dockerfile                 # Container definition
docker/entrypoint.sh       # Startup script
```

## API Endpoints

### Submit Research Task

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

### Check Task Progress

```bash
GET /task_progress/{task_id}
```

**Response:**

```json
{
  "task_id": "550e8400-...",
  "status": "running",
  "current_step": "research",
  "steps_completed": 2,
  "total_steps": 5,
  "progress": [
    {"step": "planning", "status": "completed", "result": "..."},
    {"step": "research", "status": "in_progress", "result": null}
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
  "report": "# Research Report\n\n...",
  "created_at": "2025-01-15T10:30:00",
  "completed_at": "2025-01-15T10:35:00"
}
```

## Using the Python Client

### Basic Usage

```python
import requests
import time

# Submit research task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Quantum computing applications in cryptography",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]

# Poll for progress
while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    print(f"Status: {progress['status']} - Step: {progress.get('current_step', 'N/A')}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(5)

# Get final report
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
print(result["report"])
```

## Custom Agent Implementation

### Creating a Research Tool

```python
# src/research_tools.py
import os
from tavily import TavilyClient

def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    """Search the web using Tavily API"""
    client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
    
    try:
        response = client.search(
            query=query,
            search_depth="advanced",
            max_results=max_results
        )
        return {
            "success": True,
            "results": response.get("results", []),
            "query": query
        }
    except Exception as e:
        return {
            "success": False,
            "error": str(e),
            "query": query
        }

def arxiv_search_tool(query: str, max_results: int = 3) -> dict:
    """Search arXiv for research papers"""
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
            "summary": paper.summary,
            "authors": [author.name for author in paper.authors],
            "published": paper.published.isoformat(),
            "pdf_url": paper.pdf_url
        })
    
    return {"success": True, "results": results}
```

### Building Custom Agents

```python
# src/agents.py
import aisuite as ai

def research_agent(task: str, tools: list) -> str:
    """Agent that conducts research using available tools"""
    client = ai.Client()
    
    # Build tool descriptions
    tool_descriptions = "\n".join([
        f"- {t['name']}: {t['description']}" for t in tools
    ])
    
    messages = [
        {
            "role": "system",
            "content": f"""You are a research agent. Use these tools:
{tool_descriptions}

For each subtask, decide which tool to use and synthesize findings."""
        },
        {
            "role": "user",
            "content": f"Research task: {task}"
        }
    ]
    
    response = client.chat.completions.create(
        model=os.getenv("MODEL", "openai:gpt-4o"),
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content

def writer_agent(research_data: str, topic: str) -> str:
    """Agent that writes reports from research data"""
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a technical writer. Create comprehensive, well-structured reports."
        },
        {
            "role": "user",
            "content": f"Write a report on '{topic}' using this research:\n\n{research_data}"
        }
    ]
    
    response = client.chat.completions.create(
        model=os.getenv("MODEL", "openai:gpt-4o"),
        messages=messages,
        temperature=0.5
    )
    
    return response.choices[0].message.content

def editor_agent(draft: str) -> str:
    """Agent that edits and refines reports"""
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are an editor. Improve clarity, structure, and accuracy."
        },
        {
            "role": "user",
            "content": f"Edit this report:\n\n{draft}"
        }
    ]
    
    response = client.chat.completions.create(
        model=os.getenv("MODEL", "openai:gpt-4o"),
        messages=messages,
        temperature=0.3
    )
    
    return response.choices[0].message.content
```

### Implementing Planning Agent

```python
# src/planning_agent.py
import aisuite as ai
from typing import List, Dict

def planner_agent(prompt: str) -> List[Dict[str, str]]:
    """Break down research task into subtasks"""
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": """You are a planning agent. Break down research tasks into steps:
1. Information gathering
2. Analysis
3. Synthesis
4. Writing
5. Editing

Return JSON array of subtasks with 'step' and 'description' fields."""
        },
        {
            "role": "user",
            "content": f"Plan research for: {prompt}"
        }
    ]
    
    response = client.chat.completions.create(
        model=os.getenv("MODEL", "openai:gpt-4o"),
        messages=messages,
        temperature=0.3
    )
    
    # Parse JSON response
    import json
    plan = json.loads(response.choices[0].message.content)
    return plan

def executor_agent_step(step: Dict[str, str], context: Dict) -> str:
    """Execute a single step of the plan"""
    step_name = step["step"]
    description = step["description"]
    
    if step_name == "research":
        from src.research_tools import tavily_search_tool, arxiv_search_tool
        from src.agents import research_agent
        
        tools = [
            {"name": "tavily", "description": "Web search", "fn": tavily_search_tool},
            {"name": "arxiv", "description": "Academic papers", "fn": arxiv_search_tool}
        ]
        return research_agent(description, tools)
    
    elif step_name == "writing":
        from src.agents import writer_agent
        return writer_agent(context.get("research_data", ""), description)
    
    elif step_name == "editing":
        from src.agents import editor_agent
        return editor_agent(context.get("draft", ""))
    
    return ""
```

## Database Schema

The service uses SQLAlchemy models for state management:

```python
# main.py (excerpt)
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
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)

# Connect to database
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Configuration

### Environment Variables

- `OPENAI_API_KEY` - Required for LLM calls
- `TAVILY_API_KEY` - Required for web search
- `DATABASE_URL` - PostgreSQL connection string (default: `postgresql://app:local@127.0.0.1:5432/appdb`)
- `MODEL` - Default model (default: `openai:gpt-4o`)
- `POSTGRES_USER` - DB user (default: `app`)
- `POSTGRES_PASSWORD` - DB password (default: `local`)
- `POSTGRES_DB` - DB name (default: `appdb`)
- `RESET_DB_ON_STARTUP` - Set to `1` to drop/recreate tables on startup

### Database Connection

To connect from host machine:

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

Query task status:

```sql
SELECT task_id, status, created_at, completed_at FROM tasks ORDER BY created_at DESC;
```

## Common Patterns

### Asynchronous Task Processing

```python
import threading
from fastapi import FastAPI, BackgroundTasks
from uuid import uuid4

app = FastAPI()

def process_research_task(task_id: str, prompt: str, model: str):
    """Background worker function"""
    # Update status to running
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    task.status = "running"
    db.commit()
    
    try:
        # Execute planning
        plan = planner_agent(prompt)
        
        # Execute each step
        context = {}
        for step in plan:
            result = executor_agent_step(step, context)
            context[step["step"]] = result
        
        # Save final report
        task.report = context.get("editing", context.get("writing", ""))
        task.status = "completed"
        task.completed_at = datetime.utcnow()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
    
    finally:
        db.commit()
        db.close()

@app.post("/generate_report")
def generate_report(request: dict):
    task_id = str(uuid4())
    
    # Create task record
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        prompt=request["prompt"],
        model=request.get("model", "openai:gpt-4o"),
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Start background thread
    thread = threading.Thread(
        target=process_research_task,
        args=(task_id, request["prompt"], request.get("model", "openai:gpt-4o"))
    )
    thread.start()
    
    return {"task_id": task_id}
```

### Web UI Integration

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Research Agent</title>
</head>
<body>
    <h1>Agentic Research Agent</h1>
    <form id="researchForm">
        <label>Research Topic:</label>
        <input type="text" id="prompt" required>
        <button type="submit">Start Research</button>
    </form>
    <div id="status"></div>
    <div id="report"></div>
    
    <script>
        document.getElementById('researchForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            const prompt = document.getElementById('prompt').value;
            
            const res = await fetch('/generate_report', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({prompt, model: 'openai:gpt-4o'})
            });
            
            const {task_id} = await res.json();
            pollProgress(task_id);
        });
        
        async function pollProgress(taskId) {
            const interval = setInterval(async () => {
                const res = await fetch(`/task_progress/${taskId}`);
                const data = await res.json();
                
                document.getElementById('status').textContent = 
                    `Status: ${data.status} - Step: ${data.current_step || 'N/A'}`;
                
                if (data.status === 'completed' || data.status === 'failed') {
                    clearInterval(interval);
                    const final = await fetch(`/task_status/${taskId}`);
                    const report = await final.json();
                    document.getElementById('report').innerHTML = report.report;
                }
            }, 2000);
        }
    </script>
</body>
</html>
```

## Troubleshooting

### Database Connection Issues

**Error: `DATABASE_URL not set`**

Ensure the entrypoint script exports the variable:

```bash
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
```

Or set explicitly:

```bash
docker run --rm -it -p 8000:8000 -p 5432:5432 \
  -e DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb" \
  --env-file .env fastapi-postgres-service
```

### Tables Not Persisting

By default, tables are dropped on startup. Disable with:

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

### API Key Errors

**Tavily:** Verify key is set:

```bash
docker exec -it fpsvc bash -lc 'echo $TAVILY_API_KEY'
```

**OpenAI:** Check aisuite configuration loads the key from environment.

### Template Not Found

Ensure `templates/` directory is copied in Dockerfile:

```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

Verify inside container:

```bash
docker exec -it fpsvc ls -la /app/templates/
```

### Hot Reload for Development

Mount source code for live updates:

```bash
docker run --rm -it -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Rate Limiting

Wikipedia and arXiv may rate limit. Implement exponential backoff:

```python
import time
from functools import wraps

def retry_with_backoff(retries=3, backoff_in_seconds=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            x = 0
            while True:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if x == retries:
                        raise
                    wait = backoff_in_seconds * 2 ** x
                    time.sleep(wait)
                    x += 1
        return wrapper
    return decorator

@retry_with_backoff(retries=3)
def wikipedia_search_tool(query: str):
    import wikipedia
    return wikipedia.summary(query, sentences=3)
```
