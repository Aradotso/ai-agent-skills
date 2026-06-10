---
name: deeplearning-ai-agentic-research
description: FastAPI research agent service with multi-step workflow planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres state management
triggers:
  - set up the agentic research agent service
  - create a multi-step research workflow with planning agents
  - integrate Tavily arXiv and Wikipedia search tools
  - build a reflective research agent with FastAPI
  - manage agent task state with Postgres
  - implement planner executor and writer agents
  - deploy the research agent Docker container
  - track research task progress and generate reports
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This project is a FastAPI-based research agent service that orchestrates multi-step agentic workflows. It uses a planner agent to break down research tasks, executor agents that leverage external tools (Tavily web search, arXiv papers, Wikipedia), and stores task state/results in Postgres. The entire stack runs in a single Docker container for development.

## What It Does

- **Multi-Agent Workflow**: Planner → Research → Writer → Editor agents collaborate on research tasks
- **Tool Integration**: Tavily search, arXiv academic papers, Wikipedia knowledge base
- **Task Management**: Threaded execution with progress tracking and state persistence
- **Web Interface**: Simple UI to submit tasks and monitor progress
- **REST API**: Endpoints for task submission, progress polling, and result retrieval

## Installation

### Prerequisites

- Docker (Desktop or Engine)
- API keys for OpenAI and Tavily

### Environment Setup

Create a `.env` file in the project root:

```bash
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
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

Access the application:
- UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs
- Database: `postgresql://app:local@localhost:5432/appdb`

## Project Structure

```
.
├── main.py                    # FastAPI app with routes
├── src/
│   ├── planning_agent.py      # planner_agent(), executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # tool implementations
├── templates/
│   └── index.html             # Web UI
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
└── Dockerfile
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
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Poll Task Progress

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "running",
  "current_step": "research",
  "substeps": [
    {"name": "tavily_search", "status": "completed"},
    {"name": "arxiv_search", "status": "running"}
  ]
}
```

### Get Final Status and Report

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Large Language Models for Scientific Discovery\n\n..."
}
```

## Creating Custom Agents

### Planning Agent Example

```python
# src/planning_agent.py
import aisuite

def planner_agent(prompt: str, model: str = "openai:gpt-4o"):
    """Break down research task into steps."""
    client = aisuite.Client()
    
    planning_prompt = f"""
    You are a research planning agent. Break down this task into steps:
    {prompt}
    
    Return a JSON array of steps with: step_name, description, tools_needed
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": planning_prompt}]
    )
    
    return response.choices[0].message.content
```

### Executor Agent with Tools

```python
# src/agents.py
from src.research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(query: str, tools: list):
    """Execute research using specified tools."""
    results = {}
    
    for tool in tools:
        if tool == "tavily":
            results["web"] = tavily_search_tool(query)
        elif tool == "arxiv":
            results["papers"] = arxiv_search_tool(query)
        elif tool == "wikipedia":
            results["wiki"] = wikipedia_search_tool(query)
    
    return results

def writer_agent(research_data: dict, prompt: str, model: str):
    """Generate report from research data."""
    client = aisuite.Client()
    
    context = "\n\n".join([
        f"## {source}\n{data}" 
        for source, data in research_data.items()
    ])
    
    messages = [
        {"role": "system", "content": "You are a research writer."},
        {"role": "user", "content": f"Topic: {prompt}\n\nData:\n{context}\n\nWrite a comprehensive report."}
    ]
    
    response = client.chat.completions.create(model=model, messages=messages)
    return response.choices[0].message.content
```

## Implementing Research Tools

### Tavily Web Search

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> str:
    """Search the web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return "Error: TAVILY_API_KEY not set"
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results
    }
    
    response = requests.post(url, json=payload)
    if response.status_code == 200:
        results = response.json().get("results", [])
        return "\n\n".join([
            f"**{r['title']}**\n{r['content']}\nSource: {r['url']}"
            for r in results
        ])
    return f"Error: {response.status_code}"
```

### arXiv Paper Search

```python
import requests
import xml.etree.ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5) -> str:
    """Search arXiv for academic papers."""
    url = "http://export.arxiv.org/api/query"
    params = {
        "search_query": f"all:{query}",
        "max_results": max_results,
        "sortBy": "relevance"
    }
    
    response = requests.get(url, params=params)
    root = ET.fromstring(response.content)
    
    papers = []
    for entry in root.findall("{http://www.w3.org/2005/Atom}entry"):
        title = entry.find("{http://www.w3.org/2005/Atom}title").text
        summary = entry.find("{http://www.w3.org/2005/Atom}summary").text
        link = entry.find("{http://www.w3.org/2005/Atom}id").text
        papers.append(f"**{title}**\n{summary}\nLink: {link}")
    
    return "\n\n".join(papers)
```

### Wikipedia Search

```python
import wikipedia

def wikipedia_search_tool(query: str) -> str:
    """Search Wikipedia for background information."""
    try:
        # Get summary
        summary = wikipedia.summary(query, sentences=5)
        page = wikipedia.page(query)
        return f"**{page.title}**\n\n{summary}\n\nURL: {page.url}"
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option if ambiguous
        return wikipedia_search_tool(e.options[0])
    except wikipedia.exceptions.PageError:
        return f"No Wikipedia page found for: {query}"
```

## Database Models and State Management

```python
# main.py
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    current_step = Column(String)
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)

# Create tables
Base.metadata.create_all(bind=engine)
```

## FastAPI Route Implementation

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import uuid
import threading

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
def generate_report(req: ResearchRequest, background_tasks: BackgroundTasks):
    """Start a new research task."""
    task_id = str(uuid.uuid4())
    
    # Save to DB
    db = SessionLocal()
    task = Task(task_id=task_id, prompt=req.prompt, model=req.model)
    db.add(task)
    db.commit()
    db.close()
    
    # Run in background thread
    thread = threading.Thread(target=run_workflow, args=(task_id, req.prompt, req.model))
    thread.daemon = True
    thread.start()
    
    return {"task_id": task_id}

def run_workflow(task_id: str, prompt: str, model: str):
    """Execute the multi-agent workflow."""
    db = SessionLocal()
    
    try:
        # Update status
        task = db.query(Task).filter_by(task_id=task_id).first()
        task.status = "running"
        task.current_step = "planning"
        db.commit()
        
        # Plan
        plan = planner_agent(prompt, model)
        
        # Research
        task.current_step = "research"
        db.commit()
        research_data = research_agent(prompt, ["tavily", "arxiv", "wikipedia"])
        
        # Write
        task.current_step = "writing"
        db.commit()
        draft = writer_agent(research_data, prompt, model)
        
        # Edit
        task.current_step = "editing"
        db.commit()
        final_report = editor_agent(draft, model)
        
        # Complete
        task.status = "completed"
        task.report = final_report
        task.current_step = None
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = str(e)
        db.commit()
    finally:
        db.close()
```

## Configuration

### Database Configuration

The default DATABASE_URL is set by the entrypoint script:

```bash
# Override in .env or docker run
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
```

To change Postgres credentials:

```bash
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
POSTGRES_DB=mydb
```

### Reset Database on Startup

```python
# main.py
import os

if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)
```

Run with:
```bash
docker run --env RESET_DB_ON_STARTUP=1 ...
```

## Common Patterns

### Streaming Agent Responses

```python
from fastapi.responses import StreamingResponse

@app.get("/stream_report/{task_id}")
def stream_report(task_id: str):
    def generate():
        db = SessionLocal()
        task = db.query(Task).filter_by(task_id=task_id).first()
        
        while task.status != "completed":
            yield f"data: {task.current_step}\n\n"
            time.sleep(1)
            db.refresh(task)
        
        yield f"data: {task.report}\n\n"
        db.close()
    
    return StreamingResponse(generate(), media_type="text/event-stream")
```

### Retry Logic for Tools

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay * (attempt + 1))
            return None
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2)
def tavily_search_tool(query: str):
    # Implementation with automatic retry
    ...
```

## Troubleshooting

### Templates Not Found

```bash
# Verify templates are in the container
docker exec -it fpsvc ls -l /app/templates

# Ensure Dockerfile copies them
# COPY templates/ /app/templates/
```

### Database Connection Errors

```bash
# Check DATABASE_URL is set correctly
docker exec -it fpsvc env | grep DATABASE_URL

# Connect manually to verify
docker exec -it fpsvc psql "postgresql://app:local@127.0.0.1:5432/appdb" -c "\dt"
```

### API Key Issues

```bash
# Verify environment variables are loaded
docker exec -it fpsvc env | grep API_KEY

# Check .env file is being passed
docker run --env-file .env ...
```

### Tavily Rate Limits

```python
# Add rate limiting
import time

class RateLimiter:
    def __init__(self, calls_per_minute=10):
        self.calls_per_minute = calls_per_minute
        self.last_call = 0
    
    def wait(self):
        elapsed = time.time() - self.last_call
        if elapsed < 60 / self.calls_per_minute:
            time.sleep(60 / self.calls_per_minute - elapsed)
        self.last_call = time.time()

tavily_limiter = RateLimiter(calls_per_minute=10)

def tavily_search_tool(query: str):
    tavily_limiter.wait()
    # Make API call
    ...
```

### Hot Reload for Development

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

### Inspect Task State

```bash
# Connect to database
docker exec -it fpsvc psql "postgresql://app:local@127.0.0.1:5432/appdb"

# Query tasks
SELECT task_id, status, current_step, created_at FROM tasks ORDER BY created_at DESC LIMIT 10;
```
