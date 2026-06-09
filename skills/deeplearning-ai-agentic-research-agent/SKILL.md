---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service using LangChain, Tavily, arXiv, Wikipedia with PostgreSQL state management
triggers:
  - how do I use the agentic research agent
  - set up deeplearning ai research agent service
  - create a research workflow with planning and execution
  - how to build multi-step research agents with FastAPI
  - integrate Tavily arXiv Wikipedia research tools
  - deploy research agent with PostgreSQL state tracking
  - create reflective research agent workflow
  - how to monitor research task progress
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step agentic workflows with planning, research, writing, and editing phases. Uses PostgreSQL for task state management and integrates Tavily search, arXiv, and Wikipedia research tools.

## What It Does

- **Planning Agent**: Decomposes research queries into structured workflows
- **Executor Agents**: Research, writer, and editor agents that use external tools
- **State Management**: PostgreSQL database tracks task progress and results
- **Live Progress**: Real-time status updates for multi-step workflows
- **Web UI**: Simple interface to kick off research tasks
- **REST API**: Full API for programmatic access

## Installation

### Prerequisites

- Docker (required)
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your-openai-key-here
TAVILY_API_KEY=your-tavily-key-here
```

### Build and Run with Docker

```bash
# Build the image
docker build -t fastapi-postgres-service .

# Run the container (exposes both API and Postgres)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The container runs both PostgreSQL and the FastAPI service in a single container for development.

## Key API Endpoints

### Start a Research Task

```bash
POST /generate_report
Content-Type: application/json

{
  "prompt": "Large Language Models for scientific discovery",
  "model": "openai:gpt-4o"
}

# Response: {"task_id": "uuid-here"}
```

### Check Task Progress

```bash
GET /task_progress/{task_id}

# Returns live status of each workflow step/substep
```

### Get Final Report

```bash
GET /task_status/{task_id}

# Returns final status and generated report
```

### Web UI

Navigate to `http://localhost:8000/` for the browser interface.

## Core Architecture

### Project Structure

```
.
├── main.py                    # FastAPI app entry point
├── src/
│   ├── planning_agent.py      # Planner and executor orchestration
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI template
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
└── Dockerfile
```

### Database Models

The service uses SQLAlchemy models to track task state:

```python
from sqlalchemy import Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    status = Column(String)  # "running", "completed", "failed"
    prompt = Column(Text)
    model = Column(String)
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

## Building Custom Agents

### Research Tool Example

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5):
    """Search the web using Tavily API"""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        raise ValueError("TAVILY_API_KEY not set")
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results
    }
    
    response = requests.post(url, json=payload)
    response.raise_for_status()
    return response.json()

def arxiv_search_tool(query: str, max_results: int = 3):
    """Search arXiv for academic papers"""
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
            "summary": result.summary,
            "authors": [author.name for author in result.authors],
            "pdf_url": result.pdf_url
        })
    return results
```

### Planning Agent Pattern

```python
# src/planning_agent.py
import aisuite as ai

def planner_agent(prompt: str, model: str = "openai:gpt-4o"):
    """Generate a research plan from user prompt"""
    client = ai.Client()
    
    system_prompt = """You are a research planning agent. 
    Break down the research task into clear steps:
    1. Research phase (what to search for)
    2. Writing phase (what to include)
    3. Editing phase (what to refine)
    
    Return a structured JSON plan."""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Executor Agent Pattern

```python
# src/agents.py
from research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(topic: str, plan: dict):
    """Execute research phase using available tools"""
    results = []
    
    # Web search
    web_results = tavily_search_tool(topic, max_results=5)
    results.append({"source": "web", "data": web_results})
    
    # Academic search
    arxiv_results = arxiv_search_tool(topic, max_results=3)
    results.append({"source": "arxiv", "data": arxiv_results})
    
    return {
        "phase": "research",
        "topic": topic,
        "results": results
    }

def writer_agent(research_data: dict, model: str = "openai:gpt-4o"):
    """Generate report from research data"""
    client = ai.Client()
    
    system_prompt = "You are a technical writer. Create a comprehensive report."
    user_prompt = f"Research data:\n{research_data}\n\nWrite a detailed report."
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt}
        ]
    )
    
    return response.choices[0].message.content
```

## FastAPI Integration

### Full Workflow Example

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import uuid
import os

app = FastAPI()

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

def execute_research_workflow(task_id: str, prompt: str, model: str):
    """Background task to run full research workflow"""
    db = SessionLocal()
    
    try:
        # Update task status
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "planning"
        db.commit()
        
        # Step 1: Planning
        plan = planner_agent(prompt, model)
        
        task.status = "researching"
        db.commit()
        
        # Step 2: Research
        research_results = research_agent(prompt, plan)
        
        task.status = "writing"
        db.commit()
        
        # Step 3: Writing
        draft = writer_agent(research_results, model)
        
        task.status = "editing"
        db.commit()
        
        # Step 4: Editing
        final_report = editor_agent(draft, model)
        
        # Save final report
        task.status = "completed"
        task.report = final_report
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
async def generate_report(request: dict, background_tasks: BackgroundTasks):
    """Start a new research task"""
    task_id = str(uuid.uuid4())
    prompt = request.get("prompt")
    model = request.get("model", "openai:gpt-4o")
    
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        status="queued",
        prompt=prompt,
        model=model
    )
    db.add(task)
    db.commit()
    db.close()
    
    background_tasks.add_task(execute_research_workflow, task_id, prompt, model)
    
    return {"task_id": task_id}

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get task status and report"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "prompt": task.prompt,
        "report": task.report,
        "created_at": task.created_at
    }
```

## Configuration

### Database Connection

Set `DATABASE_URL` environment variable:

```bash
export DATABASE_URL="postgresql://user:password@host:port/database"
```

Default (in container): `postgresql://app:local@127.0.0.1:5432/appdb`

### Model Selection

The service supports any model via `aisuite`. Common options:

- `openai:gpt-4o`
- `openai:gpt-4-turbo`
- `openai:gpt-3.5-turbo`

### Disable DB Reset on Startup

By default, `main.py` may drop tables on startup. To prevent:

```python
# In main.py
if os.getenv("RESET_DB_ON_STARTUP") != "1":
    # Base.metadata.drop_all(bind=engine)  # Comment out
    pass
Base.metadata.create_all(bind=engine)
```

## Common Patterns

### Streaming Progress Updates

```python
from fastapi import WebSocket

@app.websocket("/ws/task/{task_id}")
async def websocket_task_progress(websocket: WebSocket, task_id: str):
    await websocket.accept()
    db = SessionLocal()
    
    while True:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        await websocket.send_json({
            "status": task.status,
            "updated_at": str(task.updated_at)
        })
        
        if task.status in ["completed", "failed"]:
            break
        
        await asyncio.sleep(2)
    
    db.close()
    await websocket.close()
```

### Custom Research Tools

```python
def custom_tool(query: str, **kwargs):
    """Template for custom research tool"""
    # Your tool logic here
    return {"results": "..."}

# Register in agents.py
def research_agent(topic: str, plan: dict):
    results = []
    
    # Add your custom tool
    custom_results = custom_tool(topic)
    results.append({"source": "custom", "data": custom_results})
    
    return {"phase": "research", "results": results}
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

### Database Connection Errors

Test connection string:
```bash
docker exec -it fpsvc bash -lc "psql \$DATABASE_URL -c 'SELECT 1;'"
```

Verify environment variable is set:
```bash
docker exec -it fpsvc printenv DATABASE_URL
```

### API Key Errors

Ensure `.env` file is in the correct location and loaded:
```bash
docker run --rm -it --env-file .env fastapi-postgres-service printenv | grep API_KEY
```

### Tables Not Created

Force table creation by connecting to the database:
```python
# In Python shell inside container
from main import Base, engine
Base.metadata.create_all(bind=engine)
```

### Tavily Rate Limits

Handle gracefully in code:
```python
def tavily_search_tool(query: str, max_results: int = 5):
    try:
        # ... API call
        return response.json()
    except requests.HTTPError as e:
        if e.response.status_code == 429:
            return {"error": "Rate limit exceeded", "results": []}
        raise
```

### Connect to DB from Host

```bash
psql "postgresql://app:local@localhost:5432/appdb"

# Or with docker exec
docker exec -it fpsvc psql -U app -d appdb
```

### Hot Reload for Development

Mount source code and run with `--reload`:
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

## Advanced Usage

### Multi-Model Research

```python
def multi_model_research(prompt: str):
    """Use multiple models for different phases"""
    plan = planner_agent(prompt, model="openai:gpt-4o")
    research = research_agent(prompt, plan)
    draft = writer_agent(research, model="openai:gpt-4-turbo")
    final = editor_agent(draft, model="openai:gpt-4o")
    return final
```

### Custom Workflow Steps

```python
def custom_workflow(task_id: str, prompt: str, steps: list):
    """Execute custom workflow steps"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    
    for step in steps:
        task.status = step["name"]
        db.commit()
        
        result = step["function"](prompt, step.get("args", {}))
        # Store intermediate results
    
    db.close()
```
