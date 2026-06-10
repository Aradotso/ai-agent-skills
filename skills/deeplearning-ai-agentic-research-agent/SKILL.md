---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with planner/executor workflow, Tavily/arXiv/Wikipedia tools, and PostgreSQL task tracking
triggers:
  - set up the agentic research agent service
  - create a research workflow with planning and execution agents
  - build a multi-step research agent using Tavily and arXiv
  - implement the DeepLearning.AI agentic workflow pattern
  - configure the reflective research agent with FastAPI
  - run research tasks with progress tracking in Postgres
  - deploy the agentic AI research service with Docker
  - use planning agents with research and writer steps
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This project implements a **Reflective Research Agent** service built with FastAPI and PostgreSQL. It demonstrates the agentic workflow pattern: a planner agent breaks down research tasks into steps, then executor agents (research, writer, editor) perform them using tools like Tavily search, arXiv, and Wikipedia. All task state and results are stored in Postgres for live progress tracking.

## What It Does

- **Planning Agent**: Takes a research prompt and generates a multi-step plan
- **Executor Agents**: Research agent (queries Tavily/arXiv/Wikipedia), writer agent (synthesizes findings), editor agent (refines output)
- **Task Tracking**: Stores task status, steps, and results in PostgreSQL
- **Live Progress API**: Poll `/task_progress/{task_id}` for real-time updates
- **Web UI**: Simple interface at `/` to submit research tasks
- **Single Container**: Postgres + FastAPI bundled for easy local development

## Installation

### Prerequisites

- Docker installed
- API keys for OpenAI and Tavily

### Environment Setup

Create a `.env` file in the project root:

```bash
OPENAI_API_KEY=your-openai-key-here
TAVILY_API_KEY=your-tavily-key-here
```

### Build and Run

```bash
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

Access the service:
- Web UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs
- Postgres: `postgresql://app:local@localhost:5432/appdb`

## Project Structure

```
.
├── main.py                      # FastAPI application entry point
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # Tool implementations (Tavily, arXiv, Wikipedia)
├── templates/
│   └── index.html               # Web UI template
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt             # Python dependencies
├── Dockerfile
└── .env                         # API keys (not committed)
```

## Key API Endpoints

### Generate Report (Start Research Task)

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'

# Response: {"task_id": "550e8400-e29b-41d4-a716-446655440000"}
```

### Check Task Progress

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

Response structure:
```json
{
  "task_id": "550e8400-...",
  "status": "in_progress",
  "current_step": 2,
  "total_steps": 5,
  "steps": [
    {"step_number": 1, "description": "Research LLM architectures", "status": "completed"},
    {"step_number": 2, "description": "Gather scientific papers", "status": "in_progress"}
  ]
}
```

### Get Final Status and Report

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

Response includes final report text in the `report` field.

## Code Examples

### Creating Custom Research Tools

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5):
    """Search the web using Tavily API"""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results
    }
    
    response = requests.post(url, json=payload)
    if response.status_code == 200:
        return response.json()
    return {"error": f"Tavily API error: {response.status_code}"}

def arxiv_search_tool(query: str, max_results: int = 5):
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
            "pdf_url": paper.pdf_url,
            "published": str(paper.published)
        })
    
    return {"papers": results}
```

### Implementing the Planning Agent

```python
# src/planning_agent.py
import aisuite as ai
import json
import os

client = ai.Client()

def planner_agent(prompt: str, model: str = "openai:gpt-4o"):
    """Generate a multi-step research plan"""
    system_prompt = """You are a research planning agent. 
Given a research topic, create a detailed step-by-step plan.
Return your plan as JSON: {"steps": [{"step": 1, "action": "...", "agent": "research|writer|editor"}]}"""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": f"Create a research plan for: {prompt}"}
    ]
    
    response = client.chat.completions.create(
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
        # Fallback if LLM doesn't return valid JSON
        return {"steps": [{"step": 1, "action": prompt, "agent": "research"}]}

def executor_agent_step(step: dict, context: dict, model: str = "openai:gpt-4o"):
    """Execute a single step using the appropriate agent"""
    from src.agents import research_agent, writer_agent, editor_agent
    
    agent_type = step.get("agent", "research")
    action = step.get("action", "")
    
    if agent_type == "research":
        return research_agent(action, model=model)
    elif agent_type == "writer":
        return writer_agent(action, context, model=model)
    elif agent_type == "editor":
        return editor_agent(context.get("draft", ""), model=model)
    
    return {"result": "Unknown agent type"}
```

### Implementing Agent Functions

```python
# src/agents.py
import aisuite as ai
from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

client = ai.Client()

def research_agent(query: str, model: str = "openai:gpt-4o"):
    """Research agent that uses multiple tools"""
    # Gather information from multiple sources
    tavily_results = tavily_search_tool(query, max_results=3)
    arxiv_results = arxiv_search_tool(query, max_results=3)
    
    # Synthesize findings with LLM
    context = f"""
Tavily Search Results: {tavily_results}
ArXiv Papers: {arxiv_results}
"""
    
    messages = [
        {"role": "system", "content": "You are a research assistant. Synthesize information from multiple sources."},
        {"role": "user", "content": f"Research topic: {query}\n\nSources:\n{context}\n\nProvide a comprehensive summary."}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.5
    )
    
    return {"findings": response.choices[0].message.content}

def writer_agent(topic: str, research_context: dict, model: str = "openai:gpt-4o"):
    """Writer agent that creates structured reports"""
    findings = research_context.get("findings", "")
    
    messages = [
        {"role": "system", "content": "You are a technical writer. Create clear, well-structured reports."},
        {"role": "user", "content": f"Write a report on: {topic}\n\nResearch findings:\n{findings}"}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return {"draft": response.choices[0].message.content}

def editor_agent(draft: str, model: str = "openai:gpt-4o"):
    """Editor agent that refines and improves drafts"""
    messages = [
        {"role": "system", "content": "You are an editor. Improve clarity, fix errors, enhance structure."},
        {"role": "user", "content": f"Edit and improve this draft:\n\n{draft}"}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    return {"final_report": response.choices[0].message.content}
```

### Database Models and Task Management

```python
# main.py (excerpt)
from sqlalchemy import create_engine, Column, String, Integer, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
import uuid
from datetime import datetime

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, in_progress, completed, failed
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

class TaskStep(Base):
    __tablename__ = "task_steps"
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    task_id = Column(String, nullable=False)
    step_number = Column(Integer, nullable=False)
    description = Column(Text, nullable=False)
    status = Column(String, default="pending")
    result = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)

Base.metadata.create_all(bind=engine)
```

### FastAPI Endpoint Implementation

```python
# main.py (excerpt)
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import threading

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """Start a new research task"""
    db = SessionLocal()
    
    # Create task record
    task = Task(
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    task_id = task.id
    db.close()
    
    # Run workflow in background thread
    thread = threading.Thread(target=run_research_workflow, args=(task_id, request.prompt, request.model))
    thread.start()
    
    return {"task_id": task_id}

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Execute the complete research workflow"""
    from src.planning_agent import planner_agent, executor_agent_step
    
    db = SessionLocal()
    
    try:
        # Update status
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "in_progress"
        db.commit()
        
        # Generate plan
        plan = planner_agent(prompt, model=model)
        steps = plan.get("steps", [])
        
        # Create step records
        for i, step in enumerate(steps, 1):
            task_step = TaskStep(
                task_id=task_id,
                step_number=i,
                description=step.get("action", ""),
                status="pending"
            )
            db.add(task_step)
        db.commit()
        
        # Execute steps
        context = {}
        for step in steps:
            step_record = db.query(TaskStep).filter(
                TaskStep.task_id == task_id,
                TaskStep.step_number == step.get("step", 0)
            ).first()
            
            step_record.status = "in_progress"
            db.commit()
            
            result = executor_agent_step(step, context, model=model)
            context.update(result)
            
            step_record.status = "completed"
            step_record.result = str(result)
            db.commit()
        
        # Store final report
        final_report = context.get("final_report", context.get("draft", ""))
        task.report = final_report
        task.status = "completed"
        db.commit()
        
    except Exception as e:
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    
    finally:
        db.close()
```

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENAI_API_KEY` | OpenAI API key for LLM calls | Required |
| `TAVILY_API_KEY` | Tavily search API key | Required |
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://app:local@127.0.0.1:5432/appdb` |
| `POSTGRES_USER` | PostgreSQL username | `app` |
| `POSTGRES_PASSWORD` | PostgreSQL password | `local` |
| `POSTGRES_DB` | PostgreSQL database name | `appdb` |

### Docker Configuration

The `docker/entrypoint.sh` script handles:
- Starting PostgreSQL cluster
- Creating user and database
- Setting `DATABASE_URL`
- Launching Uvicorn server

To customize startup behavior, modify `entrypoint.sh` or override with your own:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v ./custom-entrypoint.sh:/custom-entrypoint.sh \
  --env-file .env \
  fastapi-postgres-service \
  /custom-entrypoint.sh
```

## Common Patterns

### Adding a New Research Tool

1. Create tool function in `src/research_tools.py`:

```python
def custom_search_tool(query: str):
    """Your custom search implementation"""
    # Implementation here
    return {"results": [...]}
```

2. Import and use in `src/agents.py`:

```python
from src.research_tools import custom_search_tool

def research_agent(query: str, model: str = "openai:gpt-4o"):
    custom_results = custom_search_tool(query)
    # Use results...
```

### Extending the Agent Workflow

Add new agent types by:

1. Implementing agent function in `src/agents.py`
2. Updating `executor_agent_step()` in `src/planning_agent.py`:

```python
def executor_agent_step(step: dict, context: dict, model: str = "openai:gpt-4o"):
    agent_type = step.get("agent", "research")
    
    if agent_type == "fact_checker":
        from src.agents import fact_checker_agent
        return fact_checker_agent(context, model=model)
    # ... existing agents
```

### Database Migrations

For production, use Alembic for migrations:

```bash
# Install Alembic
pip install alembic

# Initialize
alembic init migrations

# Create migration
alembic revision --autogenerate -m "Add new column"

# Apply migration
alembic upgrade head
```

## Troubleshooting

### Container Won't Start

Check logs:
```bash
docker logs fpsvc
```

Verify Postgres is running inside container:
```bash
docker exec -it fpsvc pg_isready
```

### API Keys Not Working

Ensure `.env` file exists and is passed to container:
```bash
# Check if env vars are set
docker exec -it fpsvc env | grep API_KEY
```

### Database Connection Errors

Test connection from inside container:
```bash
docker exec -it fpsvc bash -lc \
  "psql postgresql://app:local@127.0.0.1:5432/appdb -c 'SELECT 1;'"
```

From host machine:
```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

### Tables Not Created

The app calls `Base.metadata.create_all()` on startup. If tables don't exist:

```python
# In main.py, ensure this runs:
Base.metadata.create_all(bind=engine)
```

To reset database (DEV ONLY):
```python
Base.metadata.drop_all(bind=engine)
Base.metadata.create_all(bind=engine)
```

### Hot Reload for Development

Mount code volume and run with `--reload`:

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

### Rate Limiting from Tools

Handle exceptions gracefully:

```python
def tavily_search_tool(query: str, max_results: int = 5):
    try:
        # API call
        response = requests.post(url, json=payload, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        return {"error": f"Tavily search failed: {str(e)}"}
```

### Task Stuck in "in_progress"

Add timeout handling to workflow:

```python
import signal

def timeout_handler(signum, frame):
    raise TimeoutError("Task execution timeout")

signal.signal(signal.SIGALRM, timeout_handler)
signal.alarm(300)  # 5 minute timeout

try:
    run_research_workflow(task_id, prompt, model)
finally:
    signal.alarm(0)  # Cancel timeout
```
