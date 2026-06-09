---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service using LLM-powered planning, tool execution (Tavily, arXiv, Wikipedia), and PostgreSQL state management
triggers:
  - how do I run the agentic research agent
  - set up the reflective research agent service
  - use the DeepLearning AI research workflow
  - build a multi-step AI research agent
  - run the FastAPI agentic workflow service
  - create a research report with planning agents
  - deploy the research agent Docker container
  - troubleshoot the agentic AI research service
---

# DeepLearning AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI web application that orchestrates multi-agent research workflows. The system uses a planning agent to decompose research tasks, executes specialized agents (research, writer, editor) with tool access (Tavily search, arXiv, Wikipedia), and stores task state/results in PostgreSQL.

## What It Does

- **Planning Agent**: Breaks down research queries into actionable steps
- **Research Agents**: Execute searches across Tavily, arXiv, and Wikipedia
- **Writer/Editor Agents**: Generate and refine research reports
- **State Management**: PostgreSQL-backed task tracking with progress updates
- **Web UI**: Simple interface for kicking off research tasks
- **REST API**: Endpoints for programmatic task management

## Installation

### Prerequisites

- Docker (Desktop or Engine)
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```bash
# Required API keys
OPENAI_API_KEY=your-openai-key-here
TAVILY_API_KEY=your-tavily-key-here

# Optional: Override default database settings
# POSTGRES_USER=app
# POSTGRES_PASSWORD=local
# POSTGRES_DB=appdb
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

The entrypoint script will:
1. Start PostgreSQL cluster
2. Create database and user
3. Set `DATABASE_URL` automatically
4. Launch Uvicorn server

## Key API Endpoints

### Generate Research Report

**POST** `/generate_report`

Starts a threaded research workflow.

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

### Check Task Progress

**GET** `/task_progress/{task_id}`

Returns live status of each workflow step and substep.

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "running",
  "current_step": "research",
  "steps": [
    {
      "name": "planning",
      "status": "completed",
      "started_at": "2025-01-15T10:00:00Z",
      "completed_at": "2025-01-15T10:00:05Z"
    },
    {
      "name": "research",
      "status": "running",
      "started_at": "2025-01-15T10:00:05Z"
    }
  ]
}
```

### Get Final Report

**GET** `/task_status/{task_id}`

Retrieves final status and generated report.

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report\n\n## Introduction\n...",
  "completed_at": "2025-01-15T10:05:00Z"
}
```

### Web UI

**GET** `/`

Serves a Jinja2 template for browser-based task submission.

Access at: http://localhost:8000/

## Project Structure

```
.
├── main.py                      # FastAPI app with endpoints
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI template
├── static/                      # CSS/JS assets (optional)
├── docker/
│   └── entrypoint.sh            # Postgres + Uvicorn startup script
├── requirements.txt             # Python dependencies
├── Dockerfile
└── .env                         # API keys (not committed)
```

## Core Components

### Planning Agent

Located in `src/planning_agent.py`:

```python
from src.planning_agent import planner_agent, executor_agent_step

# Generate a research plan
plan = planner_agent(
    prompt="Quantum computing applications in drug discovery",
    model="openai:gpt-4o"
)

# Execute each step
for step in plan['steps']:
    result = executor_agent_step(
        step=step,
        context=previous_results,
        model="openai:gpt-4o"
    )
```

### Research Tools

Located in `src/research_tools.py`:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Web search via Tavily
web_results = tavily_search_tool(
    query="latest LLM research 2024",
    max_results=5
)

# Academic papers via arXiv
papers = arxiv_search_tool(
    query="transformer architecture",
    max_results=3
)

# Wikipedia summaries
wiki_summary = wikipedia_search_tool(
    query="neural networks"
)
```

### Agent Definitions

Located in `src/agents.py`:

```python
from src.agents import research_agent, writer_agent, editor_agent

# Research agent with tool access
research_result = research_agent(
    task="Find recent developments in AGI safety",
    tools=[tavily_search_tool, arxiv_search_tool],
    model="openai:gpt-4o"
)

# Writer agent
draft = writer_agent(
    research_data=research_result,
    outline=plan_outline,
    model="openai:gpt-4o"
)

# Editor agent
final_report = editor_agent(
    draft=draft,
    guidelines="Academic style, cite sources",
    model="openai:gpt-4o"
)
```

## Database Schema

The service uses SQLAlchemy models in `main.py`:

```python
from sqlalchemy import Column, String, Text, DateTime, Integer
from sqlalchemy.ext.declarative import declarative_base

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

class TaskStep(Base):
    __tablename__ = "task_steps"
    
    id = Column(Integer, primary_key=True)
    task_id = Column(String, nullable=False)
    step_name = Column(String, nullable=False)
    status = Column(String, default="pending")
    result = Column(Text, nullable=True)
    started_at = Column(DateTime, nullable=True)
    completed_at = Column(DateTime, nullable=True)
```

## Configuration

### Database Connection

The entrypoint sets a default `DATABASE_URL`:

```bash
postgresql://app:local@127.0.0.1:5432/appdb
```

To override, set in `.env`:

```bash
DATABASE_URL=postgresql://myuser:mypass@localhost:5432/mydb
```

### Model Selection

Supported models (via `aisuite` or similar):

- `openai:gpt-4o`
- `openai:gpt-4-turbo`
- `openai:gpt-3.5-turbo`

Specify in API requests:

```json
{
  "prompt": "Research topic",
  "model": "openai:gpt-4o"
}
```

## Common Patterns

### Asynchronous Task Execution

The service runs research workflows in background threads:

```python
import threading
import uuid
from datetime import datetime

def run_research_workflow(task_id: str, prompt: str, model: str):
    # Update task status
    task = session.query(Task).filter_by(task_id=task_id).first()
    task.status = "running"
    session.commit()
    
    try:
        # Execute workflow
        plan = planner_agent(prompt, model)
        research = research_agent(plan, model)
        draft = writer_agent(research, model)
        report = editor_agent(draft, model)
        
        # Save results
        task.report = report
        task.status = "completed"
    except Exception as e:
        task.status = "failed"
        task.report = str(e)
    finally:
        session.commit()

@app.post("/generate_report")
async def generate_report(request: ReportRequest):
    task_id = str(uuid.uuid4())
    
    # Create task record
    task = Task(task_id=task_id, prompt=request.prompt, model=request.model)
    session.add(task)
    session.commit()
    
    # Start background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

### Progress Tracking

Store substep progress in `TaskStep` table:

```python
def track_step(task_id: str, step_name: str, status: str, result: str = None):
    step = TaskStep(
        task_id=task_id,
        step_name=step_name,
        status=status,
        result=result,
        started_at=datetime.utcnow() if status == "running" else None,
        completed_at=datetime.utcnow() if status == "completed" else None
    )
    session.add(step)
    session.commit()

# In workflow
track_step(task_id, "planning", "running")
plan = planner_agent(prompt, model)
track_step(task_id, "planning", "completed", result=str(plan))
```

### Error Handling

```python
from sqlalchemy.exc import SQLAlchemyError

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    try:
        task = session.query(Task).filter_by(task_id=task_id).first()
        
        if not task:
            raise HTTPException(status_code=404, detail="Task not found")
        
        return {
            "task_id": task.task_id,
            "status": task.status,
            "report": task.report,
            "created_at": task.created_at.isoformat(),
            "completed_at": task.updated_at.isoformat() if task.status == "completed" else None
        }
    except SQLAlchemyError as e:
        raise HTTPException(status_code=500, detail=f"Database error: {str(e)}")
```

## Troubleshooting

### Container Fails to Start Postgres

**Symptom**: Logs show `pg_ctlcluster` errors or permission denied.

**Solution**: Ensure the Dockerfile installs `postgresql` and `postgresql-contrib`:

```dockerfile
RUN apt-get update && apt-get install -y \
    postgresql-17 \
    postgresql-contrib-17 \
    && rm -rf /var/lib/apt/lists/*
```

### Tables Not Persisting

**Symptom**: Data disappears on container restart.

**Solution**: Comment out `drop_all()` in `main.py`:

```python
# Remove or guard this line
# Base.metadata.drop_all(bind=engine)

Base.metadata.create_all(bind=engine)
```

Or use a Docker volume:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -v pgdata:/var/lib/postgresql/17/main \
  --env-file .env \
  fastapi-postgres-service
```

### Tavily API Rate Limits

**Symptom**: `429 Too Many Requests` errors.

**Solution**: Implement retry logic in `research_tools.py`:

```python
import time
from requests.exceptions import HTTPError

def tavily_search_tool(query: str, max_results: int = 5, retries: int = 3):
    for attempt in range(retries):
        try:
            response = requests.post(
                "https://api.tavily.com/search",
                json={"query": query, "max_results": max_results},
                headers={"Authorization": f"Bearer {os.getenv('TAVILY_API_KEY')}"}
            )
            response.raise_for_status()
            return response.json()
        except HTTPError as e:
            if e.response.status_code == 429 and attempt < retries - 1:
                time.sleep(2 ** attempt)  # Exponential backoff
                continue
            raise
```

### Missing Templates/Static Files

**Symptom**: 404 or `TemplateNotFound` errors.

**Solution**: Ensure Dockerfile copies them:

```dockerfile
COPY templates /app/templates
COPY static /app/static
```

Verify inside container:

```bash
docker exec -it fpsvc ls -la /app/templates
```

### Database Connection Refused

**Symptom**: `psycopg2.OperationalError: could not connect to server`.

**Solution**: Check `DATABASE_URL` uses `127.0.0.1` (not `localhost`) inside container:

```bash
# In entrypoint.sh
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
```

### Hot Reload for Development

Mount source code and run with `--reload`:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "
    pg_ctlcluster 17 main start && \
    uvicorn main:app --host 0.0.0.0 --port 8000 --reload
  "
```

### Connect to Database from Host

Use `psql` or any PostgreSQL client:

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

Or using environment variables:

```bash
PGPASSWORD=local psql -h localhost -U app -d appdb
```

## Advanced Usage

### Custom Agent Workflows

Extend `src/agents.py` with domain-specific agents:

```python
def analysis_agent(data: dict, model: str):
    """Analyzes research data for insights."""
    prompt = f"""
    Analyze the following research data and extract key insights:
    
    {data}
    
    Provide:
    1. Main findings
    2. Patterns and trends
    3. Gaps in research
    """
    
    response = llm_client.generate(prompt=prompt, model=model)
    return response

def citation_agent(report: str, sources: list, model: str):
    """Adds proper citations to report."""
    prompt = f"""
    Add citations to this report using the provided sources:
    
    Report: {report}
    
    Sources: {sources}
    
    Format: APA style
    """
    
    response = llm_client.generate(prompt=prompt, model=model)
    return response
```

### Multi-Model Orchestration

Use different models for different tasks:

```python
plan = planner_agent(prompt, model="openai:gpt-4o")  # Complex reasoning
research = research_agent(plan, model="openai:gpt-4-turbo")  # Balanced
draft = writer_agent(research, model="openai:gpt-3.5-turbo")  # Fast drafting
report = editor_agent(draft, model="openai:gpt-4o")  # Quality refinement
```

### Webhook Notifications

Notify on task completion:

```python
import requests

def notify_completion(task_id: str, webhook_url: str):
    task = session.query(Task).filter_by(task_id=task_id).first()
    
    requests.post(webhook_url, json={
        "task_id": task_id,
        "status": task.status,
        "report_url": f"https://yourservice.com/reports/{task_id}"
    })

# In workflow
try:
    # ... execute workflow ...
    task.status = "completed"
    session.commit()
    
    if webhook_url:
        notify_completion(task_id, webhook_url)
except Exception as e:
    # ... handle error ...
```
