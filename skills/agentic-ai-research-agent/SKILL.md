---
name: agentic-ai-research-agent
description: FastAPI research agent service using LLMs, planning, reflection, and multi-tool workflows with Postgres state management
triggers:
  - how do I use the agentic AI research agent
  - set up the reflective research agent with FastAPI
  - run a research workflow with Tavily and arXiv
  - build a multi-step agent with planning and reflection
  - implement research agents with tool calling
  - create a research report using the agentic workflow
  - integrate Tavily arXiv Wikipedia for research tasks
  - deploy the research agent service with Docker
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What This Project Does

The `agentic-ai-research-agent` is a FastAPI-based research automation service that orchestrates multi-step agentic workflows. It combines:

- **Planning agent**: Breaks down research questions into structured subtasks
- **Executor agents**: Research, writer, and editor agents with tool-calling capabilities
- **Research tools**: Tavily search, arXiv papers, Wikipedia lookups
- **State management**: Postgres database tracks task progress and results
- **Reflection patterns**: Agents review and improve their outputs iteratively

The service exposes REST endpoints to kick off research tasks, poll progress, and retrieve final reports. Perfect for building AI agents that need to perform multi-step research with reflection.

## Installation & Setup

### Prerequisites

1. Docker installed
2. API keys in `.env` file:

```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build and Run

```bash
# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the service (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The container runs Postgres and the FastAPI app together. On startup, it:
- Initializes Postgres cluster
- Creates `app` user and `appdb` database
- Runs SQLAlchemy migrations
- Starts Uvicorn on port 8000

### Verify Installation

```bash
# Check service health
curl http://localhost:8000/

# View API docs
open http://localhost:8000/docs
```

## Project Structure

```
.
├── main.py                      # FastAPI app, endpoints, DB models
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI for kicking off tasks
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core API Endpoints

### 1. Start a Research Task

```python
import requests

response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Large Language Models for scientific discovery",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task started: {task_id}")
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 2. Poll Task Progress

```python
import time

while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    print(f"Status: {progress['status']}")
    print(f"Step: {progress['current_step']}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(2)
```

**Response:**
```json
{
  "task_id": "550e8400-...",
  "status": "processing",
  "current_step": "research",
  "substeps": [
    {"name": "tavily_search", "status": "completed"},
    {"name": "arxiv_search", "status": "in_progress"}
  ],
  "progress_percentage": 45
}
```

### 3. Get Final Report

```python
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
print(result["report"])
```

**Response:**
```json
{
  "task_id": "550e8400-...",
  "status": "completed",
  "report": "# Research Report: Large Language Models...\n\n## Introduction\n...",
  "metadata": {
    "sources_count": 12,
    "duration_seconds": 45.2
  }
}
```

## Database Models

The service uses SQLAlchemy with Postgres:

```python
from sqlalchemy import create_engine, Column, String, Text, DateTime, JSON
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class ResearchTask(Base):
    __tablename__ = "research_tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, processing, completed, failed
    current_step = Column(String)  # planning, research, writing, editing
    substeps = Column(JSON, default=list)
    report = Column(Text)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
```

## Building Research Tools

### Tool Structure

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str) -> dict:
    """Search the web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "search_depth": "advanced",
            "max_results": 5
        }
    )
    return response.json()

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv for academic papers."""
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
    """Search Wikipedia and return summary."""
    import wikipedia
    
    try:
        summary = wikipedia.summary(query, sentences=3)
        page = wikipedia.page(query)
        return {
            "title": page.title,
            "summary": summary,
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": "disambiguation", "options": e.options[:5]}
    except wikipedia.exceptions.PageError:
        return {"error": "page_not_found"}
```

## Agent Implementation Patterns

### Planning Agent

```python
# src/planning_agent.py
import aisuite as ai

def planner_agent(prompt: str, model: str) -> dict:
    """Generate a research plan with subtasks."""
    client = ai.Client()
    
    system_prompt = """You are a research planning agent. Break down the research question 
    into structured subtasks. Return JSON with this structure:
    {
      "plan": [
        {"step": "research", "tools": ["tavily", "arxiv"], "goal": "..."},
        {"step": "synthesis", "tools": [], "goal": "..."},
        {"step": "writing", "tools": [], "goal": "..."},
        {"step": "editing", "tools": [], "goal": "..."}
      ]
    }"""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": f"Create a research plan for: {prompt}"}
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

### Research Agent

```python
# src/agents.py
def research_agent(task: str, tools: list, model: str) -> dict:
    """Execute research using specified tools."""
    from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    
    results = {
        "task": task,
        "sources": []
    }
    
    if "tavily" in tools:
        web_results = tavily_search_tool(task)
        if "results" in web_results:
            results["sources"].extend(web_results["results"])
    
    if "arxiv" in tools:
        papers = arxiv_search_tool(task, max_results=3)
        results["sources"].extend(papers)
    
    if "wikipedia" in tools:
        wiki = wikipedia_search_tool(task)
        if "summary" in wiki:
            results["sources"].append(wiki)
    
    # Synthesize findings with LLM
    client = ai.Client()
    synthesis_prompt = f"""Based on these research sources, synthesize key findings for: {task}
    
    Sources:
    {json.dumps(results['sources'], indent=2)}
    
    Provide a clear, concise summary of the most important information."""
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": synthesis_prompt}],
        temperature=0.5
    )
    
    results["synthesis"] = response.choices[0].message.content
    return results
```

### Writer Agent with Reflection

```python
def writer_agent(research_data: dict, prompt: str, model: str) -> str:
    """Write a comprehensive report from research data."""
    client = ai.Client()
    
    messages = [
        {"role": "system", "content": "You are an expert research writer. Create clear, well-structured reports."},
        {"role": "user", "content": f"""Write a comprehensive report on: {prompt}
        
        Research findings:
        {research_data['synthesis']}
        
        Sources available: {len(research_data['sources'])}
        
        Structure: Introduction, Key Findings, Analysis, Conclusion"""}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content

def editor_agent(draft: str, model: str) -> str:
    """Review and improve the draft report."""
    client = ai.Client()
    
    messages = [
        {"role": "system", "content": "You are an expert editor. Improve clarity, coherence, and accuracy."},
        {"role": "user", "content": f"""Review and improve this research report:

{draft}

Focus on:
- Clarity and readability
- Logical flow
- Factual accuracy
- Professional tone"""}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.4
    )
    
    return response.choices[0].message.content
```

## Orchestrating the Workflow

```python
# main.py (simplified workflow)
import threading
from uuid import uuid4

def execute_research_workflow(task_id: str, prompt: str, model: str):
    """Execute the full research workflow in a background thread."""
    db = SessionLocal()
    
    try:
        # Update status: planning
        task = db.query(ResearchTask).filter_by(task_id=task_id).first()
        task.status = "processing"
        task.current_step = "planning"
        db.commit()
        
        # Step 1: Plan
        plan = planner_agent(prompt, model)
        task.substeps = plan["plan"]
        db.commit()
        
        # Step 2: Research
        task.current_step = "research"
        research_steps = [s for s in plan["plan"] if s["step"] == "research"]
        research_results = research_agent(
            task=prompt,
            tools=research_steps[0]["tools"],
            model=model
        )
        db.commit()
        
        # Step 3: Write
        task.current_step = "writing"
        draft = writer_agent(research_results, prompt, model)
        db.commit()
        
        # Step 4: Edit
        task.current_step = "editing"
        final_report = editor_agent(draft, model)
        
        # Complete
        task.status = "completed"
        task.report = final_report
        task.current_step = "done"
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
async def generate_report(request: ResearchRequest):
    task_id = str(uuid4())
    
    # Create DB record
    db = SessionLocal()
    task = ResearchTask(
        task_id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Start background thread
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key for LLM calls
TAVILY_API_KEY=tvly-...            # Tavily search API key

# Optional (Postgres config)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app                  # Default: app
POSTGRES_PASSWORD=local            # Default: local
POSTGRES_DB=appdb                  # Default: appdb

# Optional (dev mode)
RESET_DB_ON_STARTUP=0              # Set to 1 to drop tables on startup
```

### Database Connection

```python
import os
from sqlalchemy import create_engine

# Read from environment
DATABASE_URL = os.getenv("DATABASE_URL")

# Create engine with connection pooling
engine = create_engine(
    DATABASE_URL,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True  # Verify connections before use
)
```

## Common Patterns

### Custom Tool Integration

```python
# Add a new research tool
def custom_search_tool(query: str) -> dict:
    """Your custom search implementation."""
    # Implement your tool logic
    return {"results": [...]}

# Register in agents.py
TOOL_REGISTRY = {
    "tavily": tavily_search_tool,
    "arxiv": arxiv_search_tool,
    "wikipedia": wikipedia_search_tool,
    "custom": custom_search_tool
}

# Use in research agent
def research_agent(task: str, tools: list, model: str):
    for tool_name in tools:
        if tool_name in TOOL_REGISTRY:
            tool_func = TOOL_REGISTRY[tool_name]
            results = tool_func(task)
            # Process results...
```

### Streaming Progress Updates

```python
# Use Server-Sent Events for real-time progress
from fastapi.responses import StreamingResponse
import asyncio

@app.get("/task_progress_stream/{task_id}")
async def stream_progress(task_id: str):
    async def event_stream():
        while True:
            db = SessionLocal()
            task = db.query(ResearchTask).filter_by(task_id=task_id).first()
            
            yield f"data: {json.dumps({
                'status': task.status,
                'current_step': task.current_step,
                'substeps': task.substeps
            })}\n\n"
            
            db.close()
            
            if task.status in ["completed", "failed"]:
                break
            
            await asyncio.sleep(1)
    
    return StreamingResponse(event_stream(), media_type="text/event-stream")
```

## Troubleshooting

### Templates Not Found

If you see `TemplateNotFound` errors:

```bash
# Verify templates directory in container
docker exec -it fpsvc ls -la /app/templates

# Ensure Dockerfile copies templates
# Add to Dockerfile:
COPY templates /app/templates
```

### Database Connection Errors

```bash
# Check DATABASE_URL is set correctly
docker exec -it fpsvc env | grep DATABASE_URL

# Test Postgres connection
docker exec -it fpsvc psql "postgresql://app:local@127.0.0.1:5432/appdb" -c "\dt"

# View Postgres logs
docker exec -it fpsvc tail -f /var/log/postgresql/postgresql-17-main.log
```

### API Key Issues

```bash
# Verify environment variables are loaded
docker exec -it fpsvc env | grep API_KEY

# Re-run with explicit env vars
docker run --rm -it \
  -p 8000:8000 \
  -e OPENAI_API_KEY=sk-... \
  -e TAVILY_API_KEY=tvly-... \
  fastapi-postgres-service
```

### Task Stuck in "Processing"

```python
# Query database directly
import psycopg2

conn = psycopg2.connect("postgresql://app:local@localhost:5432/appdb")
cur = conn.cursor()
cur.execute("SELECT task_id, status, current_step FROM research_tasks WHERE status = 'processing'")
print(cur.fetchall())

# Check for exceptions in logs
docker logs fpsvc | grep -i error
```

### Rate Limiting (Tavily/Wikipedia)

```python
# Add retry logic with exponential backoff
import time
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    delay = base_delay * (2 ** attempt)
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3)
def tavily_search_tool(query: str):
    # Implementation...
```

### Memory Issues with Large Reports

```python
# Stream large reports instead of loading all at once
from fastapi.responses import StreamingResponse

@app.get("/task_report_stream/{task_id}")
async def stream_report(task_id: str):
    async def report_chunks():
        db = SessionLocal()
        task = db.query(ResearchTask).filter_by(task_id=task_id).first()
        
        # Stream in chunks
        chunk_size = 1024
        for i in range(0, len(task.report), chunk_size):
            yield task.report[i:i+chunk_size]
        
        db.close()
    
    return StreamingResponse(report_chunks(), media_type="text/plain")
```

## Development Mode

Run with hot reload for local development:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

This mounts your local code and enables Uvicorn's auto-reload on file changes.
