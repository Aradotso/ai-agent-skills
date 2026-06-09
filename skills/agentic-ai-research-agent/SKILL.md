---
name: agentic-ai-research-agent
description: FastAPI research agent with LLM planning, tool execution (Tavily/arXiv/Wikipedia), and Postgres task tracking for agentic workflows
triggers:
  - set up the research agent service
  - create an agentic research workflow
  - build a FastAPI agent with planning and tools
  - integrate Tavily arXiv Wikipedia search agents
  - implement multi-step research task execution
  - deploy research agent with Postgres tracking
  - use the deeplearning.ai agentic workflow
  - add reflection and planning to my AI agent
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The **agentic-ai-public** project is a FastAPI-based research agent system that demonstrates agentic workflows with planning, reflection, and multi-tool execution. It orchestrates research tasks by:

1. **Planning** - Breaking down user queries into subtasks
2. **Execution** - Running specialized agents (research, writer, editor) with tools (Tavily, arXiv, Wikipedia)
3. **Tracking** - Storing task state and progress in Postgres
4. **Reflection** - Iterating on results through editor feedback

The service runs as a single Docker container with Postgres + FastAPI, exposing REST endpoints and a web UI.

## Installation

### Prerequisites

- Docker installed
- `.env` file with API keys:

```bash
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build and Run

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build Docker image
docker build -t fastapi-postgres-service .

# Run container (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The entrypoint automatically:
- Starts Postgres cluster
- Creates app user/database
- Runs FastAPI with Uvicorn

## Key API Endpoints

### Start Research Task

```http
POST /generate_report
Content-Type: application/json

{
  "prompt": "Large Language Models for scientific discovery",
  "model": "openai:gpt-4o"
}

Response: {"task_id": "uuid-here"}
```

### Poll Task Progress

```http
GET /task_progress/{task_id}

Response: {
  "task_id": "...",
  "status": "in_progress",
  "current_step": "research",
  "substeps": [
    {"name": "tavily_search", "status": "complete"},
    {"name": "arxiv_search", "status": "running"}
  ]
}
```

### Get Final Report

```http
GET /task_status/{task_id}

Response: {
  "task_id": "...",
  "status": "complete",
  "report": "# Research Report\n\n...",
  "steps": [...]
}
```

## Core Architecture

### Project Structure

```
src/
├── planning_agent.py    # planner_agent(), executor_agent_step()
├── agents.py            # research_agent, writer_agent, editor_agent
├── research_tools.py    # Tool implementations
main.py                  # FastAPI app with routes and DB models
templates/index.html     # Web UI
```

### Database Models

```python
from sqlalchemy import Column, String, Text, DateTime, JSON
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    steps = Column(JSON, default=list)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

## Creating Custom Agents

### Research Agent Example

```python
# src/agents.py
from research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

def research_agent(query: str, context: dict) -> dict:
    """Execute research using multiple tools."""
    results = {
        "tavily": [],
        "arxiv": [],
        "wikipedia": []
    }
    
    # Tavily web search
    try:
        tavily_results = tavily_search_tool(query)
        results["tavily"] = tavily_results.get("results", [])
    except Exception as e:
        print(f"Tavily error: {e}")
    
    # arXiv academic papers
    try:
        arxiv_results = arxiv_search_tool(query, max_results=5)
        results["arxiv"] = arxiv_results
    except Exception as e:
        print(f"arXiv error: {e}")
    
    # Wikipedia background
    try:
        wiki_results = wikipedia_search_tool(query)
        results["wikipedia"] = wiki_results
    except Exception as e:
        print(f"Wikipedia error: {e}")
    
    return {
        "status": "complete",
        "results": results,
        "summary": f"Found {len(results['tavily'])} web, {len(results['arxiv'])} papers"
    }
```

### Writer Agent Example

```python
def writer_agent(research_data: dict, prompt: str, model: str) -> str:
    """Generate report from research data."""
    import aisuite
    
    client = aisuite.Client()
    
    # Format research into context
    context = []
    for source, items in research_data.get("results", {}).items():
        context.append(f"\n## {source.upper()} Results:")
        for item in items[:3]:  # Top 3 from each
            if isinstance(item, dict):
                context.append(f"- {item.get('title', '')}: {item.get('snippet', '')}")
    
    system_msg = """You are a research writer. Create a comprehensive report 
    synthesizing the provided research data. Use markdown formatting with sections, 
    citations, and clear conclusions."""
    
    user_msg = f"Topic: {prompt}\n\nResearch Data:{''.join(context)}"
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_msg},
            {"role": "user", "content": user_msg}
        ],
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

### Editor Agent Example

```python
def editor_agent(draft: str, prompt: str, model: str) -> dict:
    """Review and improve the draft report."""
    import aisuite
    
    client = aisuite.Client()
    
    system_msg = """You are an editor. Review the draft and provide:
    1. A quality score (1-10)
    2. Specific improvement suggestions
    3. An edited version if score < 8"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_msg},
            {"role": "user", "content": f"Original query: {prompt}\n\nDraft:\n{draft}"}
        ],
        temperature=0.3
    )
    
    content = response.choices[0].message.content
    
    # Parse response (simplified)
    score = 8  # Extract from content
    needs_revision = score < 8
    
    return {
        "score": score,
        "needs_revision": needs_revision,
        "feedback": content,
        "edited_draft": content if needs_revision else draft
    }
```

## Planning Agent Pattern

```python
# src/planning_agent.py
def planner_agent(prompt: str, model: str) -> list:
    """Generate execution plan with subtasks."""
    import aisuite
    
    client = aisuite.Client()
    
    system_msg = """You are a research planner. Break down the query into subtasks.
    Output JSON array of steps with: {name, description, agent, dependencies}"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_msg},
            {"role": "user", "content": prompt}
        ],
        response_format={"type": "json_object"}
    )
    
    import json
    plan = json.loads(response.choices[0].message.content)
    
    # Default plan structure
    return plan.get("steps", [
        {"name": "research", "agent": "research_agent", "description": "Gather sources"},
        {"name": "write", "agent": "writer_agent", "description": "Create draft", "depends_on": ["research"]},
        {"name": "edit", "agent": "editor_agent", "description": "Review and refine", "depends_on": ["write"]}
    ])


def executor_agent_step(step: dict, context: dict, model: str) -> dict:
    """Execute a single plan step."""
    from agents import research_agent, writer_agent, editor_agent
    
    agents = {
        "research_agent": research_agent,
        "writer_agent": writer_agent,
        "editor_agent": editor_agent
    }
    
    agent_func = agents.get(step["agent"])
    if not agent_func:
        return {"status": "error", "error": f"Unknown agent: {step['agent']}"}
    
    # Execute with context from previous steps
    result = agent_func(context.get("prompt", ""), context)
    
    return {
        "status": "complete",
        "step": step["name"],
        "result": result
    }
```

## Implementing Custom Tools

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    """Search web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results,
        "search_depth": "advanced"
    }
    
    response = requests.post(url, json=payload)
    response.raise_for_status()
    return response.json()


def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv papers."""
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
    """Get Wikipedia summary."""
    import wikipedia
    
    try:
        # Search for relevant page
        search_results = wikipedia.search(query, results=1)
        if not search_results:
            return {"error": "No results found"}
        
        # Get page content
        page = wikipedia.page(search_results[0], auto_suggest=False)
        
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Handle disambiguation
        return wikipedia_search_tool(e.options[0])
    except Exception as e:
        return {"error": str(e)}
```

## FastAPI Integration

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import uuid
import threading

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """Start async research task."""
    task_id = str(uuid.uuid4())
    
    # Create DB task
    task = Task(
        id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    
    # Execute in background thread
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}


def execute_research_workflow(task_id: str, prompt: str, model: str):
    """Background worker for research execution."""
    from planning_agent import planner_agent, executor_agent_step
    
    # Update task status
    task = db.query(Task).filter(Task.id == task_id).first()
    task.status = "planning"
    db.commit()
    
    # Generate plan
    plan = planner_agent(prompt, model)
    task.steps = [{"name": s["name"], "status": "pending"} for s in plan]
    db.commit()
    
    # Execute steps
    context = {"prompt": prompt}
    for step in plan:
        task.current_step = step["name"]
        task.status = "in_progress"
        db.commit()
        
        result = executor_agent_step(step, context, model)
        context[step["name"]] = result
        
        # Update step status
        for s in task.steps:
            if s["name"] == step["name"]:
                s["status"] = "complete"
        db.commit()
    
    # Finalize
    task.status = "complete"
    task.report = context.get("edit", {}).get("result", {}).get("edited_draft", "")
    db.commit()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-proj-...           # OpenAI API key
TAVILY_API_KEY=tvly-...              # Tavily search API key

# Optional - Database (defaults set by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional - Development
RESET_DB_ON_STARTUP=0                # Set to 1 to drop tables on start
```

### Database Connection

```python
import os
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Get connection string
DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql://app:local@127.0.0.1:5432/appdb"
)

# Create engine
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Common Patterns

### Polling Task Status

```python
import requests
import time

def wait_for_task(task_id: str, timeout: int = 300) -> dict:
    """Poll until task completes or timeout."""
    start = time.time()
    
    while time.time() - start < timeout:
        response = requests.get(f"http://localhost:8000/task_status/{task_id}")
        data = response.json()
        
        if data["status"] in ["complete", "failed"]:
            return data
        
        time.sleep(5)  # Poll every 5 seconds
    
    raise TimeoutError(f"Task {task_id} did not complete in {timeout}s")

# Usage
response = requests.post("http://localhost:8000/generate_report", json={
    "prompt": "Quantum computing applications in drug discovery",
    "model": "openai:gpt-4o"
})
task_id = response.json()["task_id"]

result = wait_for_task(task_id)
print(result["report"])
```

### Adding Custom Agent

```python
# 1. Define agent function in src/agents.py
def custom_data_agent(query: str, context: dict) -> dict:
    """Custom agent that processes specific data sources."""
    # Your implementation
    return {"status": "complete", "data": [...]}

# 2. Register in executor
def executor_agent_step(step: dict, context: dict, model: str) -> dict:
    from agents import research_agent, writer_agent, editor_agent, custom_data_agent
    
    agents = {
        "research_agent": research_agent,
        "writer_agent": writer_agent,
        "editor_agent": editor_agent,
        "custom_data_agent": custom_data_agent  # Add here
    }
    
    agent_func = agents.get(step["agent"])
    return agent_func(context.get("prompt", ""), context)

# 3. Include in plan
def planner_agent(prompt: str, model: str) -> list:
    # Include custom agent in plan
    return [
        {"name": "custom_data", "agent": "custom_data_agent"},
        {"name": "research", "agent": "research_agent"},
        # ... rest of plan
    ]
```

### Streaming Progress Updates

```python
from fastapi.responses import StreamingResponse
import asyncio

@app.get("/task_progress_stream/{task_id}")
async def stream_progress(task_id: str):
    """Server-sent events for live progress."""
    async def event_generator():
        while True:
            task = db.query(Task).filter(Task.id == task_id).first()
            if not task:
                break
            
            yield f"data: {json.dumps({
                'status': task.status,
                'current_step': task.current_step,
                'steps': task.steps
            })}\n\n"
            
            if task.status in ["complete", "failed"]:
                break
            
            await asyncio.sleep(2)
    
    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

## Troubleshooting

### Database Connection Issues

```bash
# Check if Postgres is running
docker exec -it fpsvc pg_isready

# Connect to database manually
docker exec -it fpsvc psql -U app -d appdb

# View tables
docker exec -it fpsvc psql -U app -d appdb -c "\dt"

# Check DATABASE_URL
docker exec -it fpsvc printenv DATABASE_URL
```

### Tables Not Persisting

Remove the `drop_all` call from `main.py`:

```python
# Comment out or guard with env check
# Base.metadata.drop_all(bind=engine)  # DON'T drop on every start

# Only create missing tables
Base.metadata.create_all(bind=engine)
```

### API Key Errors

```python
# Verify keys are loaded
import os
print(f"OpenAI key present: {bool(os.getenv('OPENAI_API_KEY'))}")
print(f"Tavily key present: {bool(os.getenv('TAVILY_API_KEY'))}")

# Check within container
docker exec -it fpsvc printenv | grep API_KEY
```

### Rate Limiting

```python
import time
from functools import wraps

def rate_limit(calls_per_minute: int):
    """Decorator to rate limit tool calls."""
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            wait_time = min_interval - elapsed
            if wait_time > 0:
                time.sleep(wait_time)
            last_called[0] = time.time()
            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(calls_per_minute=10)
def tavily_search_tool(query: str) -> dict:
    # Implementation with rate limiting
    pass
```

### Container Logs

```bash
# View all logs
docker logs fpsvc

# Follow logs in real-time
docker logs -f fpsvc

# Last 100 lines
docker logs --tail 100 fpsvc

# Filter for errors
docker logs fpsvc 2>&1 | grep -i error
```

### Development Mode with Hot Reload

```bash
# Mount code as volume for live editing
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

This skill provides comprehensive coverage of building agentic research workflows with planning, tool execution, and state management using the agentic-ai-public framework.
