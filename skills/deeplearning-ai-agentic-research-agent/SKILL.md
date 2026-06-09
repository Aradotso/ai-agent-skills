---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with planning, multi-tool search (Tavily/arXiv/Wikipedia), and Postgres-backed task tracking
triggers:
  - how do I run the agentic research agent service
  - set up the deeplearning.ai research agent with Docker
  - create a research task using the agentic workflow API
  - how to track progress of a research agent task
  - configure OpenAI and Tavily keys for the research agent
  - use planning agent and executor in the agentic AI service
  - build and run the FastAPI agentic research container
  - query task status from the research agent Postgres database
---

# deeplearning-ai-agentic-research-agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The **Agentic AI Research Agent** is a FastAPI web service that orchestrates multi-step research workflows using planning and execution agents. It:

- Plans research workflows with a planner agent
- Executes research using specialized tool-using agents (Tavily web search, arXiv papers, Wikipedia)
- Tracks task state and progress in Postgres
- Provides REST API and web UI for kicking off and monitoring research tasks
- Runs Postgres + FastAPI in a single Docker container for easy local development

The service uses a reflective agent pattern: planner → research agent → writer agent → editor agent, with each step storing intermediate results.

## Installation

### Prerequisites

- Docker installed
- `.env` file with API keys in project root:

```env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
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

The service starts Postgres automatically and runs migrations on startup. Access the UI at `http://localhost:8000` and API docs at `http://localhost:8000/docs`.

## Project Structure

```
.
├── main.py                    # FastAPI application entry point
├── src/
│   ├── planning_agent.py      # planner_agent(), executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── Dockerfile
├── requirements.txt
└── .env                       # API keys (not committed)
```

## Key API Endpoints

### POST `/generate_report`

Kick off a new research task:

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

### GET `/task_progress/{task_id}`

Poll task progress in real-time:

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "running",
  "current_step": "research",
  "steps": {
    "planning": {"status": "complete", "progress": 100},
    "research": {"status": "running", "progress": 45},
    "writing": {"status": "pending", "progress": 0},
    "editing": {"status": "pending", "progress": 0}
  }
}
```

### GET `/task_status/{task_id}`

Get final task status and report:

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "complete",
  "report": "# Research Report: Large Language Models...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:22Z"
}
```

## Configuration

### Environment Variables

**Required:**
- `OPENAI_API_KEY` - OpenAI API key for LLM calls
- `TAVILY_API_KEY` - Tavily API key for web search

**Auto-configured by entrypoint:**
- `DATABASE_URL` - Defaults to `postgresql://app:local@127.0.0.1:5432/appdb`
- `POSTGRES_USER` - Defaults to `app`
- `POSTGRES_PASSWORD` - Defaults to `local`
- `POSTGRES_DB` - Defaults to `appdb`

### Override Database Settings

```bash
docker run --rm -it \
  -p 8000:8000 \
  -e DATABASE_URL=postgresql://myuser:mypass@localhost:5432/mydb \
  --env-file .env \
  fastapi-postgres-service
```

## Code Examples

### Creating a Custom Agent

```python
# src/agents.py
from aisuite import Client

def custom_analysis_agent(query: str, context: dict, model: str = "openai:gpt-4o"):
    """Custom agent for specialized analysis."""
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are an expert analyst. Analyze the given data and provide insights."
        },
        {
            "role": "user",
            "content": f"Query: {query}\n\nContext: {context}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

### Adding a Custom Research Tool

```python
# src/research_tools.py
import requests
import os

def pubmed_search_tool(query: str, max_results: int = 5) -> str:
    """Search PubMed for biomedical literature."""
    base_url = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi"
    
    params = {
        "db": "pubmed",
        "term": query,
        "retmax": max_results,
        "retmode": "json"
    }
    
    response = requests.get(base_url, params=params)
    data = response.json()
    
    if "esearchresult" in data and "idlist" in data["esearchresult"]:
        ids = data["esearchresult"]["idlist"]
        return f"Found {len(ids)} PubMed articles: {', '.join(ids)}"
    
    return "No results found"
```

### Implementing the Planner Agent

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> dict:
    """Generate a research plan with sequential steps."""
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": """You are a research planning agent. 
            Create a step-by-step research plan in JSON format with:
            - steps: array of {step_name, description, tools_needed}
            - estimated_time: total time estimate
            """
        },
        {
            "role": "user",
            "content": f"Create a research plan for: {prompt}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    import json
    plan = json.loads(response.choices[0].message.content)
    return plan
```

### Executor Agent Step

```python
# src/planning_agent.py
from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

def executor_agent_step(step: dict, context: str, model: str = "openai:gpt-4o") -> str:
    """Execute a single research step using appropriate tools."""
    tools_map = {
        "tavily": tavily_search_tool,
        "arxiv": arxiv_search_tool,
        "wikipedia": wikipedia_search_tool
    }
    
    results = []
    for tool_name in step.get("tools_needed", []):
        if tool_name in tools_map:
            tool_result = tools_map[tool_name](step["description"])
            results.append(f"{tool_name}: {tool_result}")
    
    # Synthesize results with LLM
    client = Client()
    messages = [
        {
            "role": "system",
            "content": "Synthesize research findings into a coherent summary."
        },
        {
            "role": "user",
            "content": f"Step: {step['step_name']}\n\nFindings:\n" + "\n\n".join(results)
        }
    ]
    
    response = client.chat.completions.create(model=model, messages=messages)
    return response.choices[0].message.content
```

### Database Schema Setup

```python
# main.py
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

Base = declarative_base()

class ResearchTask(Base):
    __tablename__ = "research_tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    status = Column(String, default="pending")
    current_step = Column(String)
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)

# Initialize database
engine = create_engine(os.getenv("DATABASE_URL"))
Base.metadata.create_all(bind=engine)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

### FastAPI Endpoint Implementation

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import uuid

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    task_id = str(uuid.uuid4())
    
    # Store task in database
    db = SessionLocal()
    task = ResearchTask(
        task_id=task_id,
        prompt=request.prompt,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background
    background_tasks.add_task(run_research_workflow, task_id, request.prompt, request.model)
    
    return {"task_id": task_id}

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background task for running the full workflow."""
    db = SessionLocal()
    task = db.query(ResearchTask).filter(ResearchTask.task_id == task_id).first()
    
    try:
        task.status = "running"
        task.current_step = "planning"
        db.commit()
        
        # Planning phase
        plan = planner_agent(prompt, model)
        
        # Execution phases
        research_results = []
        for step in plan["steps"]:
            task.current_step = step["step_name"]
            db.commit()
            result = executor_agent_step(step, "", model)
            research_results.append(result)
        
        # Final report
        task.status = "complete"
        task.report = "\n\n".join(research_results)
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()
```

## Common Patterns

### Multi-Tool Research Pattern

```python
def comprehensive_research(topic: str) -> dict:
    """Use all available tools for comprehensive research."""
    from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    
    return {
        "web_search": tavily_search_tool(topic, max_results=5),
        "academic_papers": arxiv_search_tool(topic, max_results=3),
        "background": wikipedia_search_tool(topic)
    }
```

### Reflective Agent Loop

```python
def reflective_writing(content: str, model: str, max_iterations: int = 3) -> str:
    """Iteratively improve content through reflection."""
    from src.agents import writer_agent, editor_agent
    
    current_draft = writer_agent(content, model)
    
    for i in range(max_iterations):
        feedback = editor_agent(current_draft, model)
        if "no changes needed" in feedback.lower():
            break
        current_draft = writer_agent(f"{content}\n\nFeedback: {feedback}", model)
    
    return current_draft
```

### Progress Tracking

```python
def update_task_progress(task_id: str, step: str, progress: int):
    """Update task progress in database."""
    db = SessionLocal()
    task = db.query(ResearchTask).filter(ResearchTask.task_id == task_id).first()
    if task:
        task.current_step = step
        # Store progress in JSON field or separate table
        db.commit()
    db.close()
```

## Troubleshooting

### Container won't start / Postgres errors

Check that the entrypoint script has execute permissions:
```bash
chmod +x docker/entrypoint.sh
```

View container logs:
```bash
docker logs -f fpsvc
```

### Database connection refused

Verify `DATABASE_URL` is set correctly. The entrypoint exports a default, but you can override:
```bash
docker exec -it fpsvc env | grep DATABASE_URL
```

### Tables not created

The app calls `Base.metadata.create_all()` on startup. To reset:
```python
# main.py - add this guard
if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
Base.metadata.create_all(bind=engine)
```

Then run:
```bash
docker run --rm -it -e RESET_DB_ON_STARTUP=1 --env-file .env -p 8000:8000 fastapi-postgres-service
```

### Tavily/arXiv/Wikipedia tool failures

**Tavily:** Ensure `TAVILY_API_KEY` is set in `.env`
```bash
docker exec -it fpsvc env | grep TAVILY_API_KEY
```

**arXiv:** No API key needed, but check network connectivity
```bash
docker exec -it fpsvc curl https://export.arxiv.org/api/query
```

**Wikipedia:** Rate limits may apply; implement retry logic:
```python
import time
import wikipedia

def wikipedia_search_tool(query: str, retries: int = 3) -> str:
    for attempt in range(retries):
        try:
            return wikipedia.summary(query, sentences=3)
        except wikipedia.exceptions.WikipediaException as e:
            if attempt < retries - 1:
                time.sleep(2 ** attempt)
            else:
                return f"Error: {str(e)}"
```

### Hot reload for development

Mount your code and run with `--reload`:
```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Accessing Postgres from host

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

Query tasks:
```sql
SELECT task_id, status, current_step, created_at FROM research_tasks ORDER BY created_at DESC LIMIT 10;
```

### Missing templates/static files

Verify files are copied into container:
```bash
docker exec -it fpsvc ls -la /app/templates /app/static
```

Add to Dockerfile if missing:
```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```
