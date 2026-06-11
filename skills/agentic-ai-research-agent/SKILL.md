---
name: agentic-ai-research-agent
description: Build and deploy reflective research agents with FastAPI, Postgres, and multi-step planning workflows using Tavily, arXiv, and Wikipedia tools
triggers:
  - create a research agent with planning workflow
  - build agentic AI research system
  - set up reflective research agent with FastAPI
  - implement multi-step agent workflow with tools
  - deploy research agent with Postgres backend
  - use planning agent with research tools
  - build agentic workflow with Tavily and arXiv
  - create task-based research agent API
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI-based service that implements a reflective, multi-step research workflow. It uses planning agents to break down research tasks, executes specialized agents (research, writer, editor) with external tools (Tavily search, arXiv papers, Wikipedia), and stores task state/results in Postgres. The system provides real-time progress tracking and generates comprehensive research reports.

**Key Features:**
- Multi-step agent planning and execution
- Tool-using agents with Tavily, arXiv, Wikipedia integration
- Postgres-backed task state and result storage
- Real-time progress tracking via REST API
- Web UI for launching research tasks
- Single-container Docker deployment with embedded Postgres

## Installation

### Prerequisites

- Docker (Desktop or Engine)
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```bash
# Required API keys
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...

# Optional: Override database settings
# POSTGRES_USER=app
# POSTGRES_PASSWORD=local
# POSTGRES_DB=appdb
# DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
```

### Docker Build and Run

```bash
# Build the image
docker build -t fastapi-postgres-service .

# Run the container (foreground with logs)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service

# Run in detached mode
docker run -d \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

### Access Points

- Web UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs
- Database: `postgresql://app:local@localhost:5432/appdb`

## Project Structure

```
.
├─ main.py                      # FastAPI app with endpoints
├─ src/
│  ├─ planning_agent.py         # Planner and executor agents
│  ├─ agents.py                 # Research, writer, editor agents
│  └─ research_tools.py         # Tavily, arXiv, Wikipedia tools
├─ templates/
│  └─ index.html                # Web UI template
├─ static/                      # CSS/JS assets
├─ docker/
│  └─ entrypoint.sh             # Postgres + Uvicorn startup
├─ requirements.txt
├─ Dockerfile
└─ README.md
```

## Core API Endpoints

### Generate Research Report

```python
# POST /generate_report
import requests

response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Large Language Models for scientific discovery",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task ID: {task_id}")
```

### Poll Task Progress

```python
# GET /task_progress/{task_id}
import requests
import time

def poll_progress(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = response.json()
        
        print(f"Status: {data['status']}")
        print(f"Current step: {data.get('current_step', 'N/A')}")
        print(f"Progress: {data.get('progress_pct', 0)}%")
        
        if data["status"] in ["completed", "failed"]:
            break
        
        time.sleep(2)
    
    return data

progress = poll_progress(task_id)
```

### Get Final Report

```python
# GET /task_status/{task_id}
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result["status"] == "completed":
    print("Final Report:")
    print(result["final_output"])
else:
    print(f"Task failed: {result.get('error')}")
```

## Building Custom Agents

### Creating a Research Tool

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    """Search the web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results
    }
    
    try:
        response = requests.post(url, json=payload, timeout=30)
        response.raise_for_status()
        return response.json()
    except Exception as e:
        return {"error": str(e)}

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv for research papers."""
    import arxiv
    
    try:
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
                "published": paper.published.isoformat(),
                "pdf_url": paper.pdf_url
            })
        return results
    except Exception as e:
        return [{"error": str(e)}]

def wikipedia_search_tool(query: str) -> dict:
    """Search Wikipedia and return summary."""
    import wikipedia
    
    try:
        # Search for the topic
        search_results = wikipedia.search(query, results=3)
        if not search_results:
            return {"error": "No results found"}
        
        # Get the first page
        page = wikipedia.page(search_results[0], auto_suggest=False)
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": f"Disambiguation: {e.options[:5]}"}
    except Exception as e:
        return {"error": str(e)}
```

### Implementing a Research Agent

```python
# src/agents.py
import os
from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

def research_agent(query: str, tools: list = None) -> dict:
    """
    Research agent that gathers information using multiple tools.
    """
    if tools is None:
        tools = ["tavily", "arxiv", "wikipedia"]
    
    results = {
        "query": query,
        "sources": []
    }
    
    # Use Tavily for web search
    if "tavily" in tools:
        tavily_results = tavily_search_tool(query, max_results=5)
        if "error" not in tavily_results:
            results["sources"].append({
                "tool": "tavily",
                "data": tavily_results
            })
    
    # Use arXiv for academic papers
    if "arxiv" in tools:
        arxiv_results = arxiv_search_tool(query, max_results=5)
        results["sources"].append({
            "tool": "arxiv",
            "data": arxiv_results
        })
    
    # Use Wikipedia for general knowledge
    if "wikipedia" in tools:
        wiki_results = wikipedia_search_tool(query)
        results["sources"].append({
            "tool": "wikipedia",
            "data": wiki_results
        })
    
    return results

def writer_agent(research_data: dict, style: str = "academic") -> str:
    """
    Writer agent that creates content from research data.
    """
    # This would typically use an LLM to synthesize the research
    # into a coherent report. Example structure:
    
    import aisuite as ai
    client = ai.Client()
    
    prompt = f"""
    Based on the following research data, write a {style} report:
    
    {research_data}
    
    Structure the report with:
    1. Executive Summary
    2. Key Findings
    3. Detailed Analysis
    4. Conclusions
    """
    
    messages = [{"role": "user", "content": prompt}]
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    return response.choices[0].message.content

def editor_agent(draft: str, feedback: str = None) -> str:
    """
    Editor agent that refines and improves the draft.
    """
    import aisuite as ai
    client = ai.Client()
    
    prompt = f"""
    Edit and improve the following draft. Focus on:
    - Clarity and conciseness
    - Logical flow
    - Grammar and style
    - Factual accuracy
    
    {f'Specific feedback to address: {feedback}' if feedback else ''}
    
    Draft:
    {draft}
    """
    
    messages = [{"role": "user", "content": prompt}]
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Planning Agent Implementation

```python
# src/planning_agent.py
import json
from typing import List, Dict

def planner_agent(user_prompt: str, model: str = "openai:gpt-4o") -> List[Dict]:
    """
    Plans a multi-step research workflow based on user prompt.
    Returns a list of steps with agent assignments and parameters.
    """
    import aisuite as ai
    client = ai.Client()
    
    planning_prompt = f"""
    Create a step-by-step research plan for the following task:
    "{user_prompt}"
    
    Break it into concrete steps. For each step, specify:
    - step_id: unique identifier
    - agent: which agent to use (research_agent, writer_agent, editor_agent)
    - action: brief description
    - parameters: dict of parameters for the agent
    - dependencies: list of step_ids this step depends on
    
    Return ONLY a JSON array of steps.
    """
    
    messages = [{"role": "user", "content": planning_prompt}]
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    # Parse the plan
    plan_text = response.choices[0].message.content
    try:
        # Extract JSON from markdown code blocks if present
        if "```json" in plan_text:
            plan_text = plan_text.split("```json")[1].split("```")[0]
        elif "```" in plan_text:
            plan_text = plan_text.split("```")[1].split("```")[0]
        
        plan = json.loads(plan_text.strip())
        return plan
    except json.JSONDecodeError:
        # Fallback: create a simple default plan
        return [
            {
                "step_id": "research",
                "agent": "research_agent",
                "action": "Gather information",
                "parameters": {"query": user_prompt, "tools": ["tavily", "arxiv", "wikipedia"]},
                "dependencies": []
            },
            {
                "step_id": "write",
                "agent": "writer_agent",
                "action": "Write initial draft",
                "parameters": {"style": "academic"},
                "dependencies": ["research"]
            },
            {
                "step_id": "edit",
                "agent": "editor_agent",
                "action": "Edit and refine",
                "parameters": {},
                "dependencies": ["write"]
            }
        ]

def executor_agent_step(step: Dict, step_results: Dict) -> any:
    """
    Execute a single step in the plan.
    Uses step_results to access outputs from dependency steps.
    """
    from src.agents import research_agent, writer_agent, editor_agent
    
    agent_name = step["agent"]
    parameters = step.get("parameters", {})
    dependencies = step.get("dependencies", [])
    
    # Inject dependency results into parameters
    for dep_id in dependencies:
        if dep_id in step_results:
            parameters[f"{dep_id}_output"] = step_results[dep_id]
    
    # Execute the appropriate agent
    if agent_name == "research_agent":
        return research_agent(**parameters)
    elif agent_name == "writer_agent":
        # Use research output if available
        research_data = parameters.get("research_output", {})
        return writer_agent(research_data, style=parameters.get("style", "academic"))
    elif agent_name == "editor_agent":
        # Use writer output if available
        draft = parameters.get("write_output", "")
        return editor_agent(draft, feedback=parameters.get("feedback"))
    else:
        raise ValueError(f"Unknown agent: {agent_name}")
```

## Database Models and Task Management

### SQLAlchemy Models

```python
# main.py or models.py
from sqlalchemy import Column, String, Text, DateTime, Float, JSON, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime
import os

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, default="openai:gpt-4o")
    status = Column(String, default="pending")  # pending, running, completed, failed
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    current_step = Column(String, nullable=True)
    progress_pct = Column(Float, default=0.0)
    plan = Column(JSON, nullable=True)
    step_results = Column(JSON, nullable=True)
    final_output = Column(Text, nullable=True)
    error = Column(Text, nullable=True)

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

### Task Execution with Threading

```python
# main.py
import uuid
import threading
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class GenerateReportRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

def execute_workflow(task_id: str):
    """
    Background thread function that executes the full workflow.
    """
    from src.planning_agent import planner_agent, executor_agent_step
    
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        if not task:
            return
        
        # Step 1: Planning
        task.status = "running"
        task.current_step = "planning"
        db.commit()
        
        plan = planner_agent(task.prompt, model=task.model)
        task.plan = plan
        db.commit()
        
        # Step 2: Execute each step
        step_results = {}
        total_steps = len(plan)
        
        for idx, step in enumerate(plan):
            step_id = step["step_id"]
            task.current_step = step_id
            task.progress_pct = (idx / total_steps) * 100
            db.commit()
            
            # Execute the step
            result = executor_agent_step(step, step_results)
            step_results[step_id] = result
            
            task.step_results = step_results
            db.commit()
        
        # Step 3: Finalize
        task.status = "completed"
        task.progress_pct = 100.0
        task.final_output = step_results.get(plan[-1]["step_id"], "")
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
def generate_report(request: GenerateReportRequest):
    """
    Kick off a research workflow in a background thread.
    """
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    try:
        task = Task(
            task_id=task_id,
            prompt=request.prompt,
            model=request.model,
            status="pending"
        )
        db.add(task)
        db.commit()
    finally:
        db.close()
    
    # Start background thread
    thread = threading.Thread(target=execute_workflow, args=(task_id,))
    thread.daemon = True
    thread.start()
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
def task_progress(task_id: str):
    """
    Get real-time progress for a task.
    """
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        if not task:
            raise HTTPException(status_code=404, detail="Task not found")
        
        return {
            "task_id": task.task_id,
            "status": task.status,
            "current_step": task.current_step,
            "progress_pct": task.progress_pct,
            "plan": task.plan
        }
    finally:
        db.close()

@app.get("/task_status/{task_id}")
def task_status(task_id: str):
    """
    Get final status and report for a task.
    """
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        if not task:
            raise HTTPException(status_code=404, detail="Task not found")
        
        return {
            "task_id": task.task_id,
            "status": task.status,
            "prompt": task.prompt,
            "final_output": task.final_output,
            "error": task.error,
            "created_at": task.created_at.isoformat(),
            "updated_at": task.updated_at.isoformat()
        }
    finally:
        db.close()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key for LLM
TAVILY_API_KEY=tvly-...         # Tavily API key for web search

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on startup
```

### Dockerfile Example

```dockerfile
FROM python:3.11-slim

# Install Postgres
RUN apt-get update && \
    apt-get install -y postgresql postgresql-contrib && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose ports
EXPOSE 8000 5432

# Copy and set entrypoint
COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

### Entrypoint Script

```bash
#!/bin/bash
# docker/entrypoint.sh

set -e

# Start Postgres
echo "🚀 Starting Postgres cluster..."
pg_ctlcluster 17 main start

# Wait for Postgres
until su -s /bin/bash postgres -c "psql -c 'SELECT 1'" &>/dev/null; do
  sleep 1
done
echo "✅ Postgres is ready"

# Create user and database
su -s /bin/bash postgres -c "psql -c \"CREATE ROLE ${POSTGRES_USER:-app} WITH LOGIN PASSWORD '${POSTGRES_PASSWORD:-local}';\""
su -s /bin/bash postgres -c "psql -c \"CREATE DATABASE ${POSTGRES_DB:-appdb} OWNER ${POSTGRES_USER:-app};\""

# Set DATABASE_URL
export DATABASE_URL="postgresql://${POSTGRES_USER:-app}:${POSTGRES_PASSWORD:-local}@127.0.0.1:5432/${POSTGRES_DB:-appdb}"
echo "🔗 DATABASE_URL=$DATABASE_URL"

# Start FastAPI
exec uvicorn main:app --host 0.0.0.0 --port 8000
```

## Common Patterns

### Client-Side Progress Polling

```javascript
// static/app.js
async function submitResearch(prompt, model) {
    const response = await fetch('/generate_report', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({prompt, model})
    });
    const {task_id} = await response.json();
    
    // Poll for progress
    const interval = setInterval(async () => {
        const progress = await fetch(`/task_progress/${task_id}`).then(r => r.json());
        
        console.log(`Status: ${progress.status}, Step: ${progress.current_step}, Progress: ${progress.progress_pct}%`);
        
        if (progress.status === 'completed' || progress.status === 'failed') {
            clearInterval(interval);
            
            // Get final result
            const result = await fetch(`/task_status/${task_id}`).then(r => r.json());
            console.log('Final report:', result.final_output);
        }
    }, 2000);
}
```

### Custom Tool Integration

```python
# Add a new tool to src/research_tools.py
def pubmed_search_tool(query: str, max_results: int = 10) -> list:
    """Search PubMed for biomedical literature."""
    from Bio import Entrez
    
    Entrez.email = "your.email@example.com"  # Required by NCBI
    
    try:
        handle = Entrez.esearch(db="pubmed", term=query, retmax=max_results)
        record = Entrez.read(handle)
        handle.close()
        
        id_list = record["IdList"]
        results = []
        
        if id_list:
            handle = Entrez.efetch(db="pubmed", id=id_list, rettype="abstract", retmode="xml")
            records = Entrez.read(handle)
            handle.close()
            
            for article in records['PubmedArticle']:
                medline = article['MedlineCitation']
                results.append({
                    "pmid": medline['PMID'],
                    "title": medline['Article']['ArticleTitle'],
                    "abstract": medline['Article'].get('Abstract', {}).get('AbstractText', [''])[0]
                })
        
        return results
    except Exception as e:
        return [{"error": str(e)}]
```

### Multi-Model Support

```python
# Use different models for different agents
def execute_workflow_with_specialized_models(task_id: str):
    """Use specialized models for each agent type."""
    
    # Planning with GPT-4
    plan = planner_agent(task.prompt, model="openai:gpt-4o")
    
    # Research with cheaper model
    research_results = research_agent(task.prompt)
    
    # Writing with GPT-4
    import aisuite as ai
    client = ai.Client()
    
    draft = writer_agent(research_results, style="academic")
    
    # Editing with Claude for better prose
    editing_prompt = f"Edit and improve this draft:\n\n{draft}"
    response = client.chat.completions.create(
        model="anthropic:claude-3-5-sonnet-20241022",
        messages=[{"role": "user", "content": editing_prompt}]
    )
    final_output = response.choices[0].message.content
```

## Troubleshooting

### Database Connection Issues

```bash
# Check if Postgres is running in container
docker exec -it fpsvc bash -lc "pg_lsclusters"

# Test database connection
docker exec -it fpsvc bash -lc "psql postgresql://app:local@127.0.0.1:5432/appdb -c 'SELECT 1'"

# View Postgres logs
docker exec -it fpsvc bash -lc "tail -f /var/log/postgresql/postgresql-17-main.log"
```

### API Key Errors

```python
# Validate environment variables on startup
import os
from fastapi import FastAPI

app = FastAPI()

@app.on_event("startup")
def validate_config():
    required_vars = ["OPENAI_API_KEY", "TAVILY_API_KEY"]
    missing = [var for var in required_vars if not os.getenv(var)]
    
    if missing:
        raise RuntimeError(f"Missing required environment variables: {', '.join(missing)}")
    
    print("✅ All required API keys are configured")
```

### Task Stuck in Running State

```python
# Add timeout handling
import time
from datetime import datetime, timedelta

def cleanup_stale_tasks():
    """Mark tasks as failed if they've been running too long."""
    db = SessionLocal()
    try:
        timeout = datetime.utcnow() - timedelta(hours=1)
        stale_tasks = db.query(Task).filter(
            Task.status == "running",
            Task.updated_at < timeout
        ).all()
        
        for task in stale_tasks:
            task.status = "failed"
            task.error = "Task timeout (>1 hour)"
        
        db.commit()
        print(f"Cleaned up {len(stale_tasks)} stale tasks")
    finally:
        db.close()
```

### Memory Issues with Long Reports

```python
# Stream results instead of storing in memory
def execute_workflow_streaming(task_id: str):
    """Execute workflow and stream results to database."""
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        
        # Store results incrementally
        for idx, step in enumerate(plan):
            result = executor_agent_step(step, step_results)
            
            # Save immediately instead of accumulating
            step_results[step_id] = result
            task.step_results = step_results
            db.commit()
            
            # Clear large objects from memory
            if len(str(result)) > 100000:  # >100KB
                step_results[step_id] = {"truncated": True, "size": len(str(result))}
    finally:
        db.close()
```

### Rate Limiting External APIs

```python
# Add rate limiting for external tools
import time
from functools import wraps

def rate_limit(calls_per_minute=10):
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            if elapsed < min_interval:
                time.sleep(min_interval - elapsed)
            result = func(*args, **kwargs)
            last_called[0] = time.time()
            return result
        return wrapper
    return decorator

@rate_limit(calls_per_minute=5)
def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    # ... implementation
    pass
```

## CLI Usage

### Start Service

```bash
# Start in development mode with auto-reload
docker run --rm -it -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Database Management

```bash
# Connect to database
docker exec -it fpsvc psql postgresql://app:local@127.0.0.1:5432/appdb

# View all tasks
docker exec -it fpsvc psql postgresql://app:local@127.0.0.1:5432/appdb -c "SELECT task_id, status, current_step, created_at FROM tasks;"

# Delete completed tasks
docker exec -it fpsvc psql postgresql://app:local@127.0.0.1:5432/appdb -c "DELETE FROM tasks WHERE status = 'completed' AND created_at < NOW() - INTERVAL '7 days';"
```

### Logs and Debugging

```bash
# Follow application logs
docker logs -f fpsvc

# Check container health
docker exec -it fpsvc bash -lc "curl -s http://localhost:8000/docs | head -n 20"

# Inspect running processes
docker exec -it fpsvc bash -lc "ps aux"
```

This skill provides comprehensive coverage of building, deploying, and managing reflective research agents with the Agentic AI framework. The multi-step planning workflow, tool integration, and database-backed task management make it suitable for complex research automation tasks.
