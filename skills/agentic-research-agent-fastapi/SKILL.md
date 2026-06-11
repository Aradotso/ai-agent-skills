---
name: agentic-research-agent-fastapi
description: Multi-agent research system with planning, execution, and reflection using FastAPI, PostgreSQL, and LLM tools (Tavily, arXiv, Wikipedia)
triggers:
  - set up a research agent with reflection
  - build an agentic workflow with planning and execution
  - create a multi-step research assistant with FastAPI
  - implement a reflective research agent with tool use
  - deploy an agentic AI research service
  - use planning and research agents with Tavily and arXiv
  - build a research workflow with task tracking
  - create an agent system with planner and executor
---

# Agentic Research Agent (FastAPI)

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What This Project Does

The Agentic AI Research Agent is a FastAPI service that orchestrates multi-step research workflows using LLM-powered agents. It implements an **agentic workflow pattern** with:

- **Planning Agent**: Breaks down research tasks into subtasks
- **Executor Agent**: Runs research, writing, and editing agents with tool use
- **Research Tools**: Tavily search, arXiv papers, Wikipedia lookups
- **Reflection/Iteration**: Multi-step refinement of research outputs
- **Task Tracking**: PostgreSQL-backed state management for async workflows
- **Web UI**: Simple interface to kick off and monitor research tasks

The system runs in a single Docker container with both PostgreSQL and the FastAPI service.

## Installation

### Prerequisites

1. **Docker** installed on your system
2. **API Keys** in a `.env` file at project root:

```env
OPENAI_API_KEY=sk-your-openai-key
TAVILY_API_KEY=tvly-your-tavily-key
```

### Build and Run

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Create .env file with your API keys
cat > .env << EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF

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

The service will start Postgres, create the database, and launch the FastAPI app on `http://localhost:8000`.

## Project Structure

```
.
├── main.py                     # FastAPI app with routes and DB models
├── src/
│   ├── planning_agent.py       # Planner and executor agent logic
│   ├── agents.py               # Research, writer, editor agents
│   └── research_tools.py       # Tool implementations (Tavily, arXiv, Wikipedia)
├── templates/
│   └── index.html              # Web UI
├── static/                     # CSS/JS assets
├── docker/
│   └── entrypoint.sh           # Container startup script
├── requirements.txt            # Python dependencies
└── Dockerfile
```

## Key API Endpoints

### 1. Web UI
```bash
# Open in browser
http://localhost:8000/
```

### 2. Generate Research Report (POST)
```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'

# Response:
# {"task_id": "550e8400-e29b-41d4-a716-446655440000"}
```

### 3. Poll Task Progress (GET)
```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000

# Response:
# {
#   "status": "running",
#   "current_step": "research",
#   "steps_completed": ["planning"],
#   "message": "Running research agent..."
# }
```

### 4. Get Final Report (GET)
```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000

# Response:
# {
#   "status": "completed",
#   "report": "# Research Report\n\n...",
#   "created_at": "2025-01-15T10:30:00",
#   "completed_at": "2025-01-15T10:32:45"
# }
```

## Core Components

### 1. Planning Agent (`src/planning_agent.py`)

The planner breaks down user prompts into structured subtasks:

```python
from src.planning_agent import planner_agent, executor_agent_step

# Generate a plan from a user prompt
plan = planner_agent(
    user_prompt="Research quantum computing applications in drug discovery",
    model="openai:gpt-4o"
)

# Plan structure:
# {
#   "steps": [
#     {"step": "research", "description": "Search for papers on quantum computing + drug discovery"},
#     {"step": "write", "description": "Draft report synthesizing findings"},
#     {"step": "edit", "description": "Refine report for clarity"}
#   ]
# }

# Execute each step
for step in plan["steps"]:
    result = executor_agent_step(
        step=step,
        context={},
        model="openai:gpt-4o"
    )
    print(f"Step {step['step']}: {result}")
```

### 2. Research Tools (`src/research_tools.py`)

Three main tools for information gathering:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Tavily web search (requires TAVILY_API_KEY)
results = tavily_search_tool(query="quantum computing drug discovery")
# Returns: list of {"title": "...", "url": "...", "content": "..."}

# arXiv academic paper search
papers = arxiv_search_tool(query="quantum machine learning", max_results=5)
# Returns: list of {"title": "...", "authors": [...], "summary": "...", "pdf_url": "..."}

# Wikipedia lookup
article = wikipedia_search_tool(query="Quantum computing")
# Returns: {"title": "...", "summary": "...", "url": "..."}
```

### 3. Agent Implementations (`src/agents.py`)

Specialized agents for different workflow stages:

```python
from src.agents import research_agent, writer_agent, editor_agent

# Research agent: Gathers information using tools
research_output = research_agent(
    topic="quantum computing applications",
    tools=[tavily_search_tool, arxiv_search_tool],
    model="openai:gpt-4o"
)

# Writer agent: Synthesizes research into a draft
draft = writer_agent(
    research_data=research_output,
    outline="Introduction, Applications, Conclusion",
    model="openai:gpt-4o"
)

# Editor agent: Refines and polishes the draft
final_report = editor_agent(
    draft=draft,
    style_guide="academic, concise",
    model="openai:gpt-4o"
)
```

### 4. Database Models (`main.py`)

Task tracking with SQLAlchemy:

```python
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    status = Column(String, default="pending")  # pending, running, completed, failed
    prompt = Column(Text)
    model = Column(String)
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime)
    completed_at = Column(DateTime, nullable=True)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Building a Custom Research Workflow

### Example: Multi-Step Research Task

```python
import threading
from uuid import uuid4
from datetime import datetime
from fastapi import FastAPI, BackgroundTasks
from src.planning_agent import planner_agent, executor_agent_step
from src.research_tools import tavily_search_tool, arxiv_search_tool

app = FastAPI()

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background task that runs the full agentic workflow"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    try:
        # Update status
        task.status = "running"
        db.commit()
        
        # Step 1: Planning
        plan = planner_agent(user_prompt=prompt, model=model)
        
        # Step 2: Execute each step in plan
        context = {}
        for step_info in plan["steps"]:
            result = executor_agent_step(
                step=step_info,
                context=context,
                model=model
            )
            context[step_info["step"]] = result
        
        # Step 3: Store final report
        task.report = context.get("final_report", context.get("edit", ""))
        task.status = "completed"
        task.completed_at = datetime.utcnow()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
    
    finally:
        db.commit()
        db.close()

@app.post("/generate_report")
async def generate_report(
    background_tasks: BackgroundTasks,
    request: dict
):
    task_id = str(uuid4())
    prompt = request["prompt"]
    model = request.get("model", "openai:gpt-4o")
    
    # Create task record
    db = SessionLocal()
    task = Task(
        id=task_id,
        prompt=prompt,
        model=model,
        status="pending",
        created_at=datetime.utcnow()
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, prompt, model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key for LLM calls
TAVILY_API_KEY=tvly-...         # Tavily API for web search

# Optional (defaults provided by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on startup
```

### Model Selection

The system supports multiple LLM providers via `aisuite`:

```python
# OpenAI models
model = "openai:gpt-4o"
model = "openai:gpt-4o-mini"

# Anthropic models (if configured)
model = "anthropic:claude-3-sonnet"

# Pass to any agent or planning function
plan = planner_agent(prompt, model="openai:gpt-4o")
```

## Common Patterns

### 1. Custom Research Tool

```python
def custom_search_tool(query: str) -> list[dict]:
    """Add your own search/retrieval logic"""
    import requests
    
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query},
        headers={"Authorization": f"Bearer {os.getenv('CUSTOM_API_KEY')}"}
    )
    
    return response.json()["results"]

# Register in agents.py
research_output = research_agent(
    topic="AI safety",
    tools=[tavily_search_tool, custom_search_tool],
    model="openai:gpt-4o"
)
```

### 2. Streaming Task Updates

```python
@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """Real-time progress endpoint"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    if not task:
        return {"error": "Task not found"}
    
    # You can store step-by-step progress in a separate table
    # or use a JSONB column for detailed state
    return {
        "status": task.status,
        "current_step": get_current_step(task_id),  # Custom logic
        "steps_completed": get_completed_steps(task_id),
        "message": f"Running {task.status}..."
    }
```

### 3. Reflection/Critique Loop

```python
def reflective_research_loop(prompt: str, model: str, max_iterations: int = 3):
    """Iteratively improve research output with reflection"""
    draft = None
    
    for i in range(max_iterations):
        # Research or refine
        if draft is None:
            draft = research_agent(topic=prompt, model=model)
        else:
            # Critique current draft
            critique = editor_agent(
                draft=draft,
                instruction="Identify gaps and areas for improvement",
                model=model
            )
            
            # Research to fill gaps
            additional_research = research_agent(
                topic=critique["gaps"],
                model=model
            )
            
            # Re-write with new info
            draft = writer_agent(
                research_data=draft + additional_research,
                model=model
            )
    
    return draft
```

## Troubleshooting

### Container won't start / Postgres issues

```bash
# Check logs
docker logs fpsvc

# Common issue: port conflicts
# Solution: change port mapping
docker run -p 8001:8000 -p 5433:5432 --env-file .env fastapi-postgres-service

# Verify Postgres is running inside container
docker exec -it fpsvc bash -lc "pg_isready"
```

### Database connection errors

```python
# Verify DATABASE_URL is set correctly
import os
print(os.getenv("DATABASE_URL"))

# Test connection manually
from sqlalchemy import create_engine
engine = create_engine(os.getenv("DATABASE_URL"))
conn = engine.connect()
print("Connection successful!")
conn.close()
```

### API key not found

```bash
# Verify .env file is loaded
docker exec -it fpsvc env | grep API_KEY

# If missing, restart with explicit env vars
docker run --rm -it \
  -p 8000:8000 \
  -e OPENAI_API_KEY=sk-... \
  -e TAVILY_API_KEY=tvly-... \
  fastapi-postgres-service
```

### Tavily rate limits

```python
import time

def tavily_search_with_retry(query: str, max_retries: int = 3):
    """Add retry logic for rate limits"""
    for attempt in range(max_retries):
        try:
            return tavily_search_tool(query)
        except Exception as e:
            if "rate limit" in str(e).lower() and attempt < max_retries - 1:
                time.sleep(2 ** attempt)  # Exponential backoff
                continue
            raise
```

### Hot reload for development

```bash
# Mount code as volume for live updates
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "
    pg_ctlcluster \$(psql -V | awk '{print \$3}' | cut -d. -f1) main start && \
    uvicorn main:app --host 0.0.0.0 --port 8000 --reload
  "
```

### Connect to database from host

```bash
# Use psql client
psql "postgresql://app:local@localhost:5432/appdb"

# Or with explicit connection string
psql -h localhost -p 5432 -U app -d appdb

# List all tasks
SELECT id, status, prompt, created_at FROM tasks;
```

### Reset database

```python
# Add to main.py startup
import os
from sqlalchemy import create_engine

if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)
    print("✅ Database reset complete")
```

Then run:

```bash
docker run -e RESET_DB_ON_STARTUP=1 --env-file .env fastapi-postgres-service
```

## Best Practices

1. **Always use environment variables** for API keys (never hardcode)
2. **Implement retry logic** for external API calls (Tavily, OpenAI)
3. **Store intermediate results** in the database for long-running tasks
4. **Use background threads** for async workflows (don't block API requests)
5. **Log extensively** to track agent decision-making
6. **Set timeouts** for tool calls to prevent hanging tasks
7. **Validate user input** before passing to agents (injection risks)
8. **Monitor token usage** when using paid LLM APIs

## Advanced Usage

### Custom Agent Chain

```python
from src.planning_agent import planner_agent, executor_agent_step

def custom_workflow(prompt: str, model: str):
    """Define your own agent sequence"""
    
    # 1. Break down the task
    plan = planner_agent(prompt, model)
    
    # 2. Add custom pre-processing step
    context = {"original_prompt": prompt}
    
    # 3. Execute with custom logic
    for step_info in plan["steps"]:
        # Add your own conditionals
        if step_info["step"] == "research":
            # Use specialized research logic
            context["research"] = deep_research(step_info, model)
        else:
            # Use default executor
            result = executor_agent_step(step_info, context, model)
            context[step_info["step"]] = result
    
    # 4. Post-process
    final_output = synthesize_results(context)
    return final_output
```

This skill provides comprehensive knowledge for AI agents to work with the Agentic Research Agent system, from basic setup to advanced customization.
