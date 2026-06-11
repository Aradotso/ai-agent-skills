---
name: agentic-research-agent-deeplearning
description: Build and deploy FastAPI-based research agents with planning, tool-using (Tavily, arXiv, Wikipedia), and Postgres state management in a single container.
triggers:
  - "set up the agentic research agent service"
  - "create a research workflow with planning agents"
  - "build a FastAPI research agent with Postgres"
  - "implement multi-step research workflow with agents"
  - "deploy the reflective research agent container"
  - "use Tavily and arXiv tools in research agents"
  - "create task progress tracking for agent workflows"
  - "build agentic AI research service with state management"
---

# Agentic Research Agent (DeepLearning.AI)

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Reflective Research Agent is a FastAPI-based service that orchestrates multi-step research workflows using AI agents. It features:

- **Planning Agent**: Breaks down research tasks into executable steps
- **Tool-Using Agents**: Research, writer, and editor agents with access to Tavily search, arXiv papers, and Wikipedia
- **State Management**: Postgres database tracks task state, progress, and results
- **Single Container**: All-in-one Docker image with Postgres + FastAPI
- **Live Progress**: Real-time status updates for long-running research tasks

The service exposes a web UI and REST API for kicking off research workflows, monitoring progress, and retrieving final reports.

## Installation

### Prerequisites

```bash
# Ensure Docker is installed
docker --version

# Create .env file with API keys
cat > .env << EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF
```

### Clone and Build

```bash
# Clone the repository
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build Docker image
docker build -t fastapi-postgres-service .
```

### Run the Service

```bash
# Run container with ports exposed
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

Access the service:
- **Web UI**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs
- **Postgres**: postgresql://app:local@localhost:5432/appdb

## Project Structure

```
.
├── main.py                    # FastAPI application entry point
├── src/
│   ├── planning_agent.py     # Planner and executor logic
│   ├── agents.py             # Research, writer, editor agents
│   └── research_tools.py     # Tool implementations (Tavily, arXiv, Wikipedia)
├── templates/
│   └── index.html            # Web UI template
├── static/                   # CSS/JS assets
├── docker/
│   └── entrypoint.sh         # Container startup script
├── requirements.txt
└── Dockerfile
```

## Key API Endpoints

### Start Research Task

```bash
# Kick off a research workflow
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'

# Response: {"task_id": "550e8400-e29b-41d4-a716-446655440000"}
```

### Poll Progress

```bash
# Get live progress updates
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000

# Response shows current step, substeps, and status
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "in_progress",
  "current_step": "research",
  "steps_completed": 1,
  "total_steps": 4,
  "substeps": [
    {"name": "tavily_search", "status": "completed"},
    {"name": "arxiv_search", "status": "in_progress"}
  ]
}
```

### Get Final Report

```bash
# Retrieve completed research report
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000

# Response includes full report and metadata
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report\n\n## Introduction\n...",
  "created_at": "2024-01-15T10:30:00Z",
  "completed_at": "2024-01-15T10:35:00Z"
}
```

## Core Code Patterns

### Creating Custom Research Tools

```python
# src/research_tools.py
import requests
from typing import Dict, Any

def tavily_search_tool(query: str, max_results: int = 5) -> Dict[str, Any]:
    """Search using Tavily API"""
    api_key = os.getenv("TAVILY_API_KEY")
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "query": query,
            "max_results": max_results,
            "search_depth": "advanced"
        },
        headers={"Authorization": f"Bearer {api_key}"}
    )
    return response.json()

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv papers"""
    import arxiv
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    return [
        {
            "title": result.title,
            "summary": result.summary,
            "authors": [author.name for author in result.authors],
            "pdf_url": result.pdf_url
        }
        for result in search.results()
    ]
```

### Building Custom Agents

```python
# src/agents.py
from aisuite.client import Client

def research_agent(prompt: str, tools: list, model: str = "openai:gpt-4o") -> str:
    """Agent that conducts research using available tools"""
    client = Client()
    
    # Prepare tool descriptions for the agent
    tool_descriptions = "\n".join([
        f"- {tool['name']}: {tool['description']}" 
        for tool in tools
    ])
    
    system_prompt = f"""You are a research agent. You have access to:
{tool_descriptions}

Use these tools to gather comprehensive information about the topic.
Synthesize findings into a cohesive research summary."""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content

def writer_agent(research_content: str, model: str = "openai:gpt-4o") -> str:
    """Agent that transforms research into a formatted report"""
    client = Client()
    
    messages = [
        {"role": "system", "content": "Transform research findings into a well-structured report with sections, headings, and citations."},
        {"role": "user", "content": research_content}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.5
    )
    
    return response.choices[0].message.content

def editor_agent(draft_report: str, model: str = "openai:gpt-4o") -> str:
    """Agent that refines and polishes the report"""
    client = Client()
    
    messages = [
        {"role": "system", "content": "Review and improve the report for clarity, coherence, and professionalism."},
        {"role": "user", "content": draft_report}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    return response.choices[0].message.content
```

### Implementing Planning Agent

```python
# src/planning_agent.py
import json
from aisuite.client import Client

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> dict:
    """Create a multi-step plan for research workflow"""
    client = Client()
    
    system_prompt = """You are a research planning agent. Create a step-by-step plan to complete the research task.
Output JSON with structure:
{
  "steps": [
    {"name": "research", "description": "...", "tools": ["tavily", "arxiv"]},
    {"name": "write", "description": "..."},
    {"name": "edit", "description": "..."}
  ]
}"""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": f"Plan research for: {prompt}"}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return json.loads(response.choices[0].message.content)

def executor_agent_step(step: dict, context: dict, model: str) -> str:
    """Execute a single step from the plan"""
    step_name = step["name"]
    
    if step_name == "research":
        # Execute research with tools
        results = []
        if "tavily" in step.get("tools", []):
            results.append(tavily_search_tool(context["prompt"]))
        if "arxiv" in step.get("tools", []):
            results.append(arxiv_search_tool(context["prompt"]))
        return research_agent(context["prompt"], results, model)
    
    elif step_name == "write":
        return writer_agent(context["research_results"], model)
    
    elif step_name == "edit":
        return editor_agent(context["draft_report"], model)
    
    return ""
```

### Database Models and State Management

```python
# main.py or models.py
from sqlalchemy import create_engine, Column, String, Text, DateTime, Enum
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import enum
from datetime import datetime
import os

Base = declarative_base()

class TaskStatus(enum.Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"
    FAILED = "failed"

class ResearchTask(Base):
    __tablename__ = "research_tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(Enum(TaskStatus), default=TaskStatus.PENDING)
    current_step = Column(String)
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    completed_at = Column(DateTime)

# Initialize database
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

### FastAPI Application Setup

```python
# main.py
from fastapi import FastAPI, BackgroundTasks, HTTPException
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel
import uuid
import threading

app = FastAPI(title="Reflective Research Agent")

# Mount templates and static files
templates = Jinja2Templates(directory="templates")
app.mount("/static", StaticFiles(directory="static"), name="static")

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.get("/")
async def home(request: Request):
    """Render web UI"""
    return templates.TemplateResponse("index.html", {"request": request})

@app.post("/generate_report")
async def generate_report(req: ResearchRequest, background_tasks: BackgroundTasks):
    """Kick off research workflow in background"""
    task_id = str(uuid.uuid4())
    
    # Create task in database
    db = SessionLocal()
    task = ResearchTask(
        task_id=task_id,
        prompt=req.prompt,
        model=req.model,
        status=TaskStatus.PENDING
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Start workflow in thread
    thread = threading.Thread(target=run_workflow, args=(task_id, req.prompt, req.model))
    thread.start()
    
    return {"task_id": task_id}

def run_workflow(task_id: str, prompt: str, model: str):
    """Execute the full research workflow"""
    db = SessionLocal()
    task = db.query(ResearchTask).filter_by(task_id=task_id).first()
    
    try:
        # Update status
        task.status = TaskStatus.IN_PROGRESS
        db.commit()
        
        # Step 1: Plan
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute plan
        context = {"prompt": prompt}
        for step in plan["steps"]:
            task.current_step = step["name"]
            db.commit()
            
            result = executor_agent_step(step, context, model)
            context[f"{step['name']}_results"] = result
        
        # Step 3: Store final report
        task.report = context.get("edit_results", "")
        task.status = TaskStatus.COMPLETED
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = TaskStatus.FAILED
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """Get live progress for a task"""
    db = SessionLocal()
    task = db.query(ResearchTask).filter_by(task_id=task_id).first()
    db.close()
    
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    return {
        "task_id": task.task_id,
        "status": task.status.value,
        "current_step": task.current_step,
        "created_at": task.created_at.isoformat()
    }

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get final status and report"""
    db = SessionLocal()
    task = db.query(ResearchTask).filter_by(task_id=task_id).first()
    db.close()
    
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    return {
        "task_id": task.task_id,
        "status": task.status.value,
        "report": task.report,
        "created_at": task.created_at.isoformat(),
        "completed_at": task.completed_at.isoformat() if task.completed_at else None
    }
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...                    # OpenAI API key
TAVILY_API_KEY=tvly-...                  # Tavily search API key

# Optional - Database (defaults set by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional - Behavior
RESET_DB_ON_STARTUP=0                    # Set to 1 to drop tables on startup
```

### Requirements

```txt
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.0
python-dotenv>=1.0.0
jinja2>=3.1.0
requests>=2.31.0
wikipedia>=1.4.0
arxiv>=2.0.0
aisuite>=0.1.0
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs fpsvc

# Verify Postgres initialization
docker exec -it fpsvc bash -lc "pg_lsclusters"

# Test database connection
docker exec -it fpsvc bash -lc "psql $DATABASE_URL -c 'SELECT 1'"
```

### Tables Missing After Restart

The default `main.py` may drop tables on startup. Disable this:

```python
# In main.py, comment out or guard:
# Base.metadata.drop_all(bind=engine)

# Or use environment flag:
if os.getenv("RESET_DB_ON_STARTUP") != "1":
    Base.metadata.create_all(bind=engine)
```

### API Key Errors

```bash
# Verify .env is loaded
docker exec -it fpsvc bash -lc "echo \$OPENAI_API_KEY"

# If empty, restart with explicit env vars
docker run --rm -it \
  -p 8000:8000 \
  -e OPENAI_API_KEY=sk-... \
  -e TAVILY_API_KEY=tvly-... \
  --name fpsvc \
  fastapi-postgres-service
```

### Tool Failures (Tavily, arXiv, Wikipedia)

```python
# Add error handling to tools
def tavily_search_tool(query: str, max_results: int = 5) -> Dict[str, Any]:
    try:
        api_key = os.getenv("TAVILY_API_KEY")
        if not api_key:
            return {"error": "TAVILY_API_KEY not set"}
        # ... rest of implementation
    except Exception as e:
        return {"error": f"Tavily search failed: {str(e)}"}

# Wikipedia rate limiting - add retry logic
import time
def wikipedia_search_tool(query: str, retries: int = 3) -> list:
    for attempt in range(retries):
        try:
            # ... search logic
            return results
        except Exception as e:
            if attempt < retries - 1:
                time.sleep(2 ** attempt)  # Exponential backoff
            else:
                raise
```

### Development with Hot Reload

```bash
# Mount source code for live updates
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Debugging Long-Running Tasks

```python
# Add detailed logging
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def run_workflow(task_id: str, prompt: str, model: str):
    logger.info(f"Starting workflow for task {task_id}")
    
    for step in plan["steps"]:
        logger.info(f"Executing step: {step['name']}")
        result = executor_agent_step(step, context, model)
        logger.info(f"Step {step['name']} completed: {len(result)} chars")
```

## Best Practices

1. **Always use environment variables** for API keys, never hardcode
2. **Implement retries** for external API calls (Tavily, arXiv)
3. **Add timeouts** to prevent stuck tasks
4. **Log progress** to database for observability
5. **Handle partial failures** gracefully in multi-step workflows
6. **Use background tasks** or message queues for production deployments
7. **Add rate limiting** to prevent API quota exhaustion
8. **Cache results** for repeated queries to reduce costs
