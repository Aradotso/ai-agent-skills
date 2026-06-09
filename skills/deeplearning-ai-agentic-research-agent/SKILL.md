---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with planning, multi-step workflows, and tool integration (Tavily, arXiv, Wikipedia) backed by Postgres
triggers:
  - set up agentic research workflow
  - create research agent with planning and tools
  - build multi-agent research system
  - implement reflective research agent with FastAPI
  - integrate Tavily arXiv Wikipedia research tools
  - deploy containerized research agent service
  - create task-based research agent workflow
  - build planning executor research pipeline
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step research workflows using a planner-executor pattern. The system runs tool-using agents (Tavily for web search, arXiv for papers, Wikipedia for encyclopedic knowledge) and stores task state/results in Postgres. Designed for the Agentic Workflow course by DeepLearning.AI.

## What It Does

- **Planning Agent**: Breaks down research tasks into steps
- **Executor Agents**: Research, writer, and editor agents that execute planned steps
- **Tool Integration**: Tavily search, arXiv academic search, Wikipedia lookup
- **Task Management**: Threaded execution with progress tracking via Postgres
- **REST API**: FastAPI endpoints for task submission, progress polling, and results
- **Web UI**: Simple Jinja2 template for initiating research tasks

## Installation

### Prerequisites

- Docker (for containerized deployment)
- API keys: OpenAI and Tavily
- `.env` file with credentials

### Create `.env` File

```bash
# Required API keys
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build and Run with Docker

```bash
# Build the container
docker build -t fastapi-postgres-service .

# Run (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres separately, then set DATABASE_URL
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="sk-..."
export TAVILY_API_KEY="tvly-..."

# Run the app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Key API Endpoints

### Start a Research Task

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

### Poll Task Progress

```bash
GET /task_progress/{task_id}
```

**Response:**
```json
{
  "task_id": "550e8400-...",
  "status": "in_progress",
  "current_step": "research",
  "steps_completed": 2,
  "total_steps": 5,
  "substeps": [
    {"name": "tavily_search", "status": "completed"},
    {"name": "arxiv_search", "status": "in_progress"}
  ]
}
```

### Get Final Status and Report

```bash
GET /task_status/{task_id}
```

**Response:**
```json
{
  "task_id": "550e8400-...",
  "status": "completed",
  "report": "# Research Report\n\n## Introduction\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:00Z"
}
```

## Project Structure

```
.
├─ main.py                      # FastAPI app entry point
├─ src/
│  ├─ planning_agent.py         # planner_agent(), executor_agent_step()
│  ├─ agents.py                 # research_agent, writer_agent, editor_agent
│  └─ research_tools.py         # tavily, arxiv, wikipedia search tools
├─ templates/
│  └─ index.html                # Web UI
├─ static/                      # CSS/JS assets
├─ docker/
│  └─ entrypoint.sh             # Container startup script
├─ requirements.txt
├─ Dockerfile
└─ README.md
```

## Core Components

### Planning Agent

The planner breaks down research tasks into executable steps:

```python
from src.planning_agent import planner_agent

# Generate a plan
plan = planner_agent(
    prompt="Research quantum computing applications in drug discovery",
    model="openai:gpt-4o"
)

# plan contains:
# {
#   "steps": [
#     {"name": "research", "description": "Gather sources..."},
#     {"name": "analyze", "description": "Synthesize findings..."},
#     {"name": "write", "description": "Draft report..."},
#     {"name": "edit", "description": "Refine and polish..."}
#   ]
# }
```

### Research Tools

#### Tavily Search

```python
from src.research_tools import tavily_search_tool

results = tavily_search_tool(
    query="latest advances in transformer models",
    max_results=5
)

# Returns: List of dicts with {title, url, snippet, score}
```

#### arXiv Search

```python
from src.research_tools import arxiv_search_tool

papers = arxiv_search_tool(
    query="attention mechanisms neural networks",
    max_results=10
)

# Returns: List of papers with {title, authors, summary, pdf_url}
```

#### Wikipedia Search

```python
from src.research_tools import wikipedia_search_tool

content = wikipedia_search_tool(
    query="Large language model",
    sentences=5
)

# Returns: String with Wikipedia summary
```

### Agent Implementation Pattern

```python
from src.agents import research_agent, writer_agent, editor_agent

# Research phase
research_results = research_agent(
    topic="AI safety alignment",
    tools=[tavily_search_tool, arxiv_search_tool, wikipedia_search_tool]
)

# Writing phase
draft = writer_agent(
    research_data=research_results,
    outline=plan["steps"]
)

# Editing phase
final_report = editor_agent(
    draft=draft,
    criteria=["clarity", "accuracy", "coherence"]
)
```

## Database Models

### Task Table

```python
from sqlalchemy import Column, String, Text, DateTime, Enum
from sqlalchemy.dialects.postgresql import UUID
import enum

class TaskStatus(str, enum.Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"
    FAILED = "failed"

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    prompt = Column(Text, nullable=False)
    model = Column(String(100), nullable=False)
    status = Column(Enum(TaskStatus), default=TaskStatus.PENDING)
    current_step = Column(String(50))
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)
```

## Complete FastAPI Example

```python
from fastapi import FastAPI, BackgroundTasks, HTTPException
from pydantic import BaseModel
import threading
from src.planning_agent import planner_agent, executor_agent_step
from database import SessionLocal, Task, TaskStatus

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background thread executing the research workflow"""
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = TaskStatus.IN_PROGRESS
        db.commit()
        
        # Step 1: Generate plan
        plan = planner_agent(prompt=prompt, model=model)
        
        # Step 2: Execute each step
        results = []
        for step in plan["steps"]:
            task.current_step = step["name"]
            db.commit()
            
            result = executor_agent_step(
                step=step,
                previous_results=results,
                model=model
            )
            results.append(result)
        
        # Step 3: Mark complete
        task.status = TaskStatus.COMPLETED
        task.report = results[-1]  # Final report
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = TaskStatus.FAILED
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
async def generate_report(request: ResearchRequest):
    db = SessionLocal()
    task = Task(prompt=request.prompt, model=request.model)
    db.add(task)
    db.commit()
    task_id = str(task.id)
    db.close()
    
    # Start background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional overrides
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0  # Set to 1 to drop/recreate tables
```

### Model Selection

Supports any `aisuite`-compatible model:

```python
# OpenAI
{"model": "openai:gpt-4o"}
{"model": "openai:gpt-4-turbo"}

# Anthropic
{"model": "anthropic:claude-3-opus-20240229"}

# Other providers via aisuite
{"model": "google:gemini-pro"}
```

## Common Patterns

### Custom Agent with Tools

```python
from src.research_tools import tavily_search_tool, arxiv_search_tool

def custom_research_agent(query: str, domain: str):
    """Domain-specific research agent"""
    
    # Use different tools based on domain
    if domain == "academic":
        papers = arxiv_search_tool(query, max_results=10)
        return analyze_papers(papers)
    elif domain == "news":
        articles = tavily_search_tool(query, max_results=15)
        return summarize_news(articles)
    else:
        # Combined approach
        web = tavily_search_tool(query, max_results=5)
        wiki = wikipedia_search_tool(query, sentences=3)
        papers = arxiv_search_tool(query, max_results=5)
        return synthesize_sources(web, wiki, papers)
```

### Progress Tracking Pattern

```python
def update_substep_progress(task_id: str, substep: str, status: str):
    """Track fine-grained progress"""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    # Store substeps as JSON
    substeps = task.metadata or []
    substeps.append({
        "name": substep,
        "status": status,
        "timestamp": datetime.utcnow().isoformat()
    })
    task.metadata = substeps
    db.commit()
    db.close()

# Usage in executor
def executor_agent_step(step, previous_results, model):
    update_substep_progress(task_id, "tavily_search", "started")
    web_results = tavily_search_tool(step["query"])
    update_substep_progress(task_id, "tavily_search", "completed")
    
    update_substep_progress(task_id, "arxiv_search", "started")
    papers = arxiv_search_tool(step["query"])
    update_substep_progress(task_id, "arxiv_search", "completed")
    
    return synthesize(web_results, papers)
```

### Error Handling and Retries

```python
import time
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
def resilient_tool_call(tool_func, *args, **kwargs):
    """Retry tool calls with exponential backoff"""
    try:
        return tool_func(*args, **kwargs)
    except Exception as e:
        print(f"Tool call failed: {e}, retrying...")
        raise

# Usage
results = resilient_tool_call(
    tavily_search_tool,
    query="machine learning",
    max_results=10
)
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs fpsvc

# Verify Postgres is running
docker exec -it fpsvc bash -lc "pg_isready"

# Check database connection
docker exec -it fpsvc bash -lc "psql postgresql://app:local@127.0.0.1:5432/appdb -c 'SELECT 1'"
```

### Missing Templates Error

```bash
# Verify templates exist in container
docker exec -it fpsvc ls -l /app/templates/

# If missing, check Dockerfile COPY directive
COPY templates/ /app/templates/
COPY static/ /app/static/
```

### API Key Errors

```bash
# Verify env vars are loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Check .env file syntax (no spaces around =)
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Tasks Stuck in IN_PROGRESS

```python
# Add timeout handling
import signal

def timeout_handler(signum, frame):
    raise TimeoutError("Task exceeded maximum execution time")

signal.signal(signal.SIGALRM, timeout_handler)
signal.alarm(600)  # 10 minute timeout

try:
    run_research_workflow(task_id, prompt, model)
finally:
    signal.alarm(0)  # Disable alarm
```

### Database Connection Pool Exhausted

```python
# Configure SQLAlchemy pool
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True  # Verify connections before use
)
```

### Wikipedia Rate Limiting

```python
import time

def rate_limited_wikipedia_search(query, sentences=5, delay=1.0):
    """Add delay between Wikipedia requests"""
    time.sleep(delay)
    return wikipedia_search_tool(query, sentences)
```

## Testing

### Test Individual Tools

```bash
# Test from Python REPL
python3 -c "
from src.research_tools import tavily_search_tool
import os
os.environ['TAVILY_API_KEY'] = 'tvly-...'
results = tavily_search_tool('AI safety', max_results=3)
print(results)
"
```

### Test API Endpoints

```bash
# Submit task
TASK_ID=$(curl -s -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "test query", "model": "openai:gpt-4o"}' | jq -r '.task_id')

# Poll progress
watch -n 2 "curl -s http://localhost:8000/task_progress/$TASK_ID | jq"

# Get final result
curl -s http://localhost:8000/task_status/$TASK_ID | jq '.report'
```
