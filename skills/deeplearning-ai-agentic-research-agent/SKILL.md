---
name: deeplearning-ai-agentic-research-agent
description: FastAPI service for multi-agent research workflows with planning, tool execution (Tavily, arXiv, Wikipedia), and Postgres task tracking
triggers:
  - how do I use the agentic research agent
  - set up the DeepLearning.AI research agent service
  - run a multi-step research workflow with agents
  - configure the reflective research agent API
  - build and deploy the agentic AI research service
  - work with the planning and executor agents
  - track research task progress with this agent system
  - integrate Tavily and arXiv into agent workflows
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step workflows using a planner agent and specialized research/writer/editor agents. The system uses tools like Tavily (web search), arXiv (academic papers), and Wikipedia, storing all task state and results in Postgres.

## What It Does

- **Planning Agent**: Decomposes research questions into structured workflows
- **Executor Agents**: Research, writer, and editor agents that use external tools
- **Tool Integration**: Tavily search, arXiv academic search, Wikipedia lookups
- **Task Tracking**: Postgres-backed state management with real-time progress updates
- **REST API**: FastAPI endpoints for task submission, progress polling, and result retrieval
- **Single Container**: Docker setup with Postgres + API in one container for easy local dev

## Installation & Setup

### Prerequisites

1. **Docker** installed on your system
2. **API Keys** in a `.env` file at project root:

```bash
# .env
OPENAI_API_KEY=sk-your-openai-key
TAVILY_API_KEY=tvly-your-tavily-key
```

### Build the Container

```bash
docker build -t fastapi-postgres-service .
```

### Run the Service

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will:
- Start Postgres on port 5432
- Create the database and user
- Launch FastAPI on port 8000

### Verify Running

```bash
# Check logs
docker logs -f fpsvc

# Access UI
open http://localhost:8000

# Access API docs
open http://localhost:8000/docs
```

## Key API Endpoints

### 1. Generate Research Report

**POST** `/generate_report`

Initiates a multi-step research workflow.

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 2. Poll Task Progress

**GET** `/task_progress/{task_id}`

Returns real-time status of each workflow step.

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "running",
  "current_step": "research_agent",
  "steps_completed": 2,
  "total_steps": 5,
  "progress": {
    "planning": "completed",
    "research": "running",
    "writing": "pending",
    "editing": "pending"
  }
}
```

### 3. Get Final Report

**GET** `/task_status/{task_id}`

Retrieves completed task status and final report.

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report\n\n## Introduction\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:00Z"
}
```

## Project Structure & Key Components

### Directory Layout

```
.
├── main.py                    # FastAPI app with endpoints
├── src/
│   ├── planning_agent.py      # Planner and executor logic
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
├── Dockerfile
└── .env                       # API keys (not committed)
```

### Custom Agent Implementation

**src/agents.py** - Define specialized agents:

```python
from aisuite import Client

client = Client()

def research_agent(query: str, tools: list) -> str:
    """
    Research agent that uses external tools to gather information.
    """
    messages = [
        {
            "role": "system",
            "content": "You are a research agent. Use tools to find accurate information."
        },
        {
            "role": "user",
            "content": f"Research this topic: {query}"
        }
    ]
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages,
        tools=tools
    )
    
    return response.choices[0].message.content


def writer_agent(research_data: str) -> str:
    """
    Writer agent that synthesizes research into a report.
    """
    messages = [
        {
            "role": "system",
            "content": "You are a technical writer. Create clear, structured reports."
        },
        {
            "role": "user",
            "content": f"Write a report based on: {research_data}"
        }
    ]
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    return response.choices[0].message.content


def editor_agent(draft: str) -> str:
    """
    Editor agent that refines and improves the draft.
    """
    messages = [
        {
            "role": "system",
            "content": "You are an editor. Improve clarity, accuracy, and structure."
        },
        {
            "role": "user",
            "content": f"Edit this draft: {draft}"
        }
    ]
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Research Tools

**src/research_tools.py** - Tool implementations:

```python
import os
import requests
import wikipedia

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search the web using Tavily API.
    """
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
    
    return response.json().get("results", [])


def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search arXiv for academic papers.
    """
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
            "authors": [author.name for author in paper.authors],
            "summary": paper.summary,
            "url": paper.entry_id,
            "published": paper.published.isoformat()
        })
    
    return results


def wikipedia_search_tool(query: str, sentences: int = 3) -> str:
    """
    Get Wikipedia summary for a topic.
    """
    try:
        summary = wikipedia.summary(query, sentences=sentences)
        return summary
    except wikipedia.exceptions.DisambiguationError as e:
        # Return summary of first option
        return wikipedia.summary(e.options[0], sentences=sentences)
    except wikipedia.exceptions.PageError:
        return f"No Wikipedia page found for '{query}'"
```

### Planning Agent Workflow

**src/planning_agent.py** - Orchestrate multi-step workflows:

```python
from typing import Dict, List
from aisuite import Client
import json

client = Client()

def planner_agent(task: str) -> List[Dict]:
    """
    Decompose a research task into steps.
    """
    messages = [
        {
            "role": "system",
            "content": """You are a research planner. Break down tasks into steps.
            Return a JSON array of steps with: agent_type, description, dependencies."""
        },
        {
            "role": "user",
            "content": f"Plan research workflow for: {task}"
        }
    ]
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    plan = json.loads(response.choices[0].message.content)
    return plan


def executor_agent_step(step: Dict, context: Dict) -> str:
    """
    Execute a single workflow step.
    """
    agent_type = step["agent_type"]
    description = step["description"]
    
    if agent_type == "research":
        from agents import research_agent
        from research_tools import tavily_search_tool, arxiv_search_tool
        
        tools = [tavily_search_tool, arxiv_search_tool]
        return research_agent(description, tools)
    
    elif agent_type == "writer":
        from agents import writer_agent
        research_data = context.get("research_data", "")
        return writer_agent(research_data)
    
    elif agent_type == "editor":
        from agents import editor_agent
        draft = context.get("draft", "")
        return editor_agent(draft)
    
    return ""
```

## Database Models

The service uses SQLAlchemy models to track tasks:

```python
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime
import os

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | Yes | OpenAI API key for LLM calls |
| `TAVILY_API_KEY` | Yes | Tavily API key for web search |
| `DATABASE_URL` | No | Postgres connection string (default set by entrypoint) |
| `POSTGRES_USER` | No | Postgres username (default: `app`) |
| `POSTGRES_PASSWORD` | No | Postgres password (default: `local`) |
| `POSTGRES_DB` | No | Postgres database name (default: `appdb`) |

## Common Patterns

### Background Task Execution

```python
import threading
from uuid import uuid4

def run_research_workflow(task_id: str, prompt: str, model: str):
    """
    Background thread that executes the full workflow.
    """
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    
    try:
        task.status = "running"
        db.commit()
        
        # Plan workflow
        plan = planner_agent(prompt)
        
        context = {}
        for step in plan:
            result = executor_agent_step(step, context)
            context[step["agent_type"] + "_data"] = result
        
        # Final report is the editor output
        task.report = context.get("editor_data", "")
        task.status = "completed"
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
    
    finally:
        db.commit()
        db.close()


@app.post("/generate_report")
def generate_report(request: dict):
    task_id = str(uuid4())
    prompt = request["prompt"]
    model = request["model"]
    
    # Create task in DB
    db = SessionLocal()
    task = Task(task_id=task_id, prompt=prompt, model=model)
    db.add(task)
    db.commit()
    db.close()
    
    # Start background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, prompt, model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

### Direct Database Access

```bash
# Connect from host machine
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT task_id, status, created_at FROM tasks ORDER BY created_at DESC;

# View specific task
SELECT * FROM tasks WHERE task_id = '550e8400-e29b-41d4-a716-446655440000';
```

## Troubleshooting

### Tables Not Persisting

If tables disappear on restart, comment out the drop statement in `main.py`:

```python
# Don't do this in production
# Base.metadata.drop_all(bind=engine)

# Only create tables if they don't exist
Base.metadata.create_all(bind=engine)
```

Or use an environment flag:

```python
if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
Base.metadata.create_all(bind=engine)
```

### API Key Errors

Ensure `.env` file is in the project root and passed to Docker:

```bash
# Check environment variables in container
docker exec fpsvc env | grep API_KEY

# Verify .env file exists
cat .env
```

### Tavily Rate Limits

Handle rate limiting gracefully:

```python
def tavily_search_tool(query: str, max_results: int = 5) -> list:
    try:
        # ... API call ...
    except requests.exceptions.HTTPError as e:
        if e.response.status_code == 429:
            return [{"error": "Rate limited, try again later"}]
        raise
```

### Container Won't Start

Check Postgres initialization:

```bash
# View full logs
docker logs fpsvc

# Enter container
docker exec -it fpsvc bash

# Check Postgres status
pg_ctlcluster 17 main status

# Manually start if needed
pg_ctlcluster 17 main start
```

### Hot Reload for Development

Mount source code and use Uvicorn reload:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Missing Templates/Static Files

Verify files are copied into the container:

```bash
docker exec fpsvc ls -la /app/templates
docker exec fpsvc ls -la /app/static
```

Ensure `Dockerfile` includes:

```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```
