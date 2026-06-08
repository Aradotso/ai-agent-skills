---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service using LLMs, planning workflows, and multiple research tools (Tavily, arXiv, Wikipedia) with PostgreSQL task tracking
triggers:
  - set up the DeepLearning.AI research agent service
  - create a multi-agent research workflow with planning
  - build a FastAPI research agent with Tavily and arXiv
  - implement an agentic research system with task tracking
  - deploy the reflective research agent with PostgreSQL
  - use the DeepLearning.AI research agent API
  - run agentic workflows with planner and executor agents
  - integrate research tools into an agent workflow
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step AI workflows using a planner-executor pattern. The system runs tool-using agents (Tavily search, arXiv papers, Wikipedia) and stores task state/results in PostgreSQL. Designed for the DeepLearning.AI Agentic Workflow course.

## What It Does

- **Planning Agent**: Breaks down research queries into structured workflows
- **Executor Agent**: Runs research, writer, and editor agents in sequence
- **Tool Integration**: Tavily web search, arXiv academic papers, Wikipedia knowledge
- **Task Tracking**: PostgreSQL stores task state, progress, and results
- **Web UI**: Simple interface to kick off research tasks and monitor progress
- **REST API**: Endpoints for task creation, progress polling, and result retrieval

## Installation

### Docker Setup (Recommended)

The project runs Postgres and FastAPI in a single container for local development.

**Prerequisites:**
- Docker installed
- `.env` file with API keys in project root:

```bash
# .env
OPENAI_API_KEY=sk-your-openai-key
TAVILY_API_KEY=tvly-your-tavily-key
```

**Build and Run:**

```bash
# Build the image
docker build -t fastapi-postgres-service .

# Run the container (exposes ports 8000 for API, 5432 for Postgres)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

### Local Development (No Docker)

```bash
# Install dependencies
pip install -r requirements.txt

# Set up PostgreSQL (ensure running locally)
# Update DATABASE_URL in your environment
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="sk-your-key"
export TAVILY_API_KEY="tvly-your-key"

# Run the FastAPI app
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Project Structure

```
.
├── main.py                      # FastAPI app with routes
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI template
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## API Reference

### Start a Research Task

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
  "progress": 45,
  "substeps": [
    {"step": "planning", "status": "completed"},
    {"step": "research", "status": "running"},
    {"step": "writing", "status": "pending"}
  ]
}
```

### Get Final Report

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report on Large Language Models...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:00Z"
}
```

## Core Components

### Planning Agent

The planning agent breaks down research queries into structured workflows:

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(query: str, model: str = "openai:gpt-4o"):
    """
    Generate a research plan with steps.
    Returns a structured plan with research, writing, and editing phases.
    """
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a research planning agent. Break down queries into actionable steps."
        },
        {
            "role": "user",
            "content": f"Create a research plan for: {query}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Research Tools

The system includes three main research tools:

```python
# src/research_tools.py
import os
import requests
from typing import List, Dict

def tavily_search_tool(query: str, max_results: int = 5) -> List[Dict]:
    """
    Search the web using Tavily API.
    """
    api_key = os.getenv("TAVILY_API_KEY")
    url = "https://api.tavily.com/search"
    
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results
    }
    
    response = requests.post(url, json=payload)
    response.raise_for_status()
    
    return response.json().get("results", [])


def arxiv_search_tool(query: str, max_results: int = 5) -> List[Dict]:
    """
    Search arXiv for academic papers.
    """
    import arxiv
    
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for result in search.results():
        results.append({
            "title": result.title,
            "authors": [author.name for author in result.authors],
            "summary": result.summary,
            "pdf_url": result.pdf_url,
            "published": str(result.published)
        })
    
    return results


def wikipedia_search_tool(query: str) -> Dict:
    """
    Search Wikipedia and return summary.
    """
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=5),
            "url": page.url,
            "content": page.content[:2000]  # First 2000 chars
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option from disambiguation
        return wikipedia_search_tool(e.options[0])
    except wikipedia.exceptions.PageError:
        return {"error": "Page not found"}
```

### Research Agent

Orchestrates research using multiple tools:

```python
# src/agents.py
from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

def research_agent(topic: str, model: str = "openai:gpt-4o") -> Dict:
    """
    Conduct research using multiple sources.
    """
    # Gather information from multiple sources
    web_results = tavily_search_tool(topic, max_results=3)
    arxiv_results = arxiv_search_tool(topic, max_results=3)
    wiki_results = wikipedia_search_tool(topic)
    
    # Synthesize findings
    from aisuite import Client
    client = Client()
    
    context = f"""
    Web Search Results: {web_results}
    
    Academic Papers: {arxiv_results}
    
    Wikipedia: {wiki_results}
    """
    
    messages = [
        {
            "role": "system",
            "content": "You are a research agent. Synthesize information from multiple sources."
        },
        {
            "role": "user",
            "content": f"Research topic: {topic}\n\nSources:\n{context}\n\nProvide a comprehensive summary."
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return {
        "research": response.choices[0].message.content,
        "sources": {
            "web": web_results,
            "arxiv": arxiv_results,
            "wikipedia": wiki_results
        }
    }


def writer_agent(research_data: Dict, model: str = "openai:gpt-4o") -> str:
    """
    Write a report based on research findings.
    """
    from aisuite import Client
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a technical writer. Create well-structured reports from research data."
        },
        {
            "role": "user",
            "content": f"Write a comprehensive report based on: {research_data['research']}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content


def editor_agent(draft: str, model: str = "openai:gpt-4o") -> str:
    """
    Edit and refine a draft report.
    """
    from aisuite import Client
    client = Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are an editor. Improve clarity, coherence, and formatting."
        },
        {
            "role": "user",
            "content": f"Edit this report:\n\n{draft}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Task Management with SQLAlchemy

```python
# main.py (excerpt)
from sqlalchemy import create_engine, Column, String, DateTime, Text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()


class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)


# Create tables
Base.metadata.create_all(bind=engine)
```

### FastAPI Workflow Endpoint

```python
# main.py (excerpt)
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import uuid
import threading

app = FastAPI()


class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"


def run_research_workflow(task_id: str, prompt: str, model: str):
    """
    Background task that runs the full research workflow.
    """
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    try:
        # Step 1: Planning
        task.status = "planning"
        db.commit()
        plan = planner_agent(prompt, model)
        
        # Step 2: Research
        task.status = "researching"
        db.commit()
        research_data = research_agent(prompt, model)
        
        # Step 3: Writing
        task.status = "writing"
        db.commit()
        draft = writer_agent(research_data, model)
        
        # Step 4: Editing
        task.status = "editing"
        db.commit()
        final_report = editor_agent(draft, model)
        
        # Complete
        task.status = "completed"
        task.report = final_report
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()


@app.post("/generate_report")
def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """
    Kick off a research workflow in the background.
    """
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    task = Task(
        id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run in background thread
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
OPENAI_API_KEY=sk-your-openai-key
TAVILY_API_KEY=tvly-your-tavily-key

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional overrides for Docker container
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb
```

### Model Configuration

The system supports multiple LLM providers through `aisuite`. Specify models in API calls:

```python
# OpenAI
{"model": "openai:gpt-4o"}
{"model": "openai:gpt-3.5-turbo"}

# Anthropic (if configured)
{"model": "anthropic:claude-3-opus"}

# Other providers supported by aisuite
```

## Common Patterns

### Custom Research Workflow

```python
# Create a specialized research workflow
from src.planning_agent import planner_agent
from src.agents import research_agent, writer_agent

def custom_research_workflow(topic: str, model: str = "openai:gpt-4o"):
    # 1. Create plan
    plan = planner_agent(f"Research {topic} focusing on recent developments", model)
    
    # 2. Execute research
    research = research_agent(topic, model)
    
    # 3. Generate report
    report = writer_agent(research, model)
    
    return {
        "plan": plan,
        "research": research,
        "report": report
    }
```

### Accessing Task Data

```python
# Query tasks from database
from main import SessionLocal, Task

db = SessionLocal()

# Get all completed tasks
completed = db.query(Task).filter(Task.status == "completed").all()

# Get task by ID
task = db.query(Task).filter(Task.id == task_id).first()

# Update task status
task.status = "processing"
db.commit()

db.close()
```

### Adding Custom Tools

```python
# src/research_tools.py
def custom_api_tool(query: str) -> Dict:
    """
    Add your own research tool.
    """
    import requests
    
    api_key = os.getenv("CUSTOM_API_KEY")
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    
    return response.json()
```

## Troubleshooting

### Database Connection Issues

```bash
# Check if Postgres is running in container
docker exec -it fpsvc bash -c "pg_isready"

# Connect to database manually
docker exec -it fpsvc bash
psql -U app -d appdb

# Verify tables exist
\dt
```

### API Key Errors

```python
# Verify environment variables are loaded
import os
print(f"OpenAI Key exists: {bool(os.getenv('OPENAI_API_KEY'))}")
print(f"Tavily Key exists: {bool(os.getenv('TAVILY_API_KEY'))}")
```

### Task Stuck in "Running" Status

```python
# Check task logs in database
from main import SessionLocal, Task

db = SessionLocal()
task = db.query(Task).filter(Task.id == "task-id-here").first()
print(f"Status: {task.status}")
print(f"Created: {task.created_at}")
print(f"Report: {task.report}")
```

### Rate Limiting from Research Tools

```python
# Add retry logic with exponential backoff
import time
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
def tavily_search_with_retry(query: str):
    return tavily_search_tool(query)
```

### Container Doesn't Start

```bash
# Check logs
docker logs fpsvc

# Verify entrypoint script
docker run --rm -it fastapi-postgres-service cat /docker/entrypoint.sh

# Start container with shell for debugging
docker run --rm -it --env-file .env fastapi-postgres-service bash
```

### Templates Not Found

```bash
# Verify templates directory in container
docker exec -it fpsvc ls -la /app/templates/

# Check Dockerfile COPY commands
docker run --rm -it fastapi-postgres-service ls -la /app/
```
