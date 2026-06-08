---
name: agentic-ai-research-agent
description: Build multi-step research agents using FastAPI, Postgres, and tool-calling workflows with planning, research, writing, and editing agents.
triggers:
  - how do I build a research agent with agentic AI
  - set up the agentic research agent service
  - create a multi-step research workflow
  - use planning and executor agents
  - integrate Tavily arXiv Wikipedia tools
  - run agentic AI research agent with Docker
  - implement reflective research agent workflow
  - query task progress in agentic AI
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI service that orchestrates multi-step research workflows using planning and executor agents. It combines tool-calling agents (Tavily search, arXiv, Wikipedia) with a Postgres backend to track task state and results. The system uses a planner agent to break down research tasks into substeps, then executes them with specialized agents (research, writer, editor).

**Key capabilities:**
- Plan complex research workflows automatically
- Execute multi-agent research pipelines (planner → research → writer → editor)
- Track task progress in real-time via REST API
- Store results in Postgres for persistence
- Simple web UI for kicking off research tasks

## Installation

### Docker Setup (Recommended)

1. **Clone the repository:**
```bash
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file with API keys:**
```bash
# .env
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
```

3. **Build the Docker image:**
```bash
docker build -t fastapi-postgres-service .
```

4. **Run the container:**
```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service runs Postgres and FastAPI in a single container for local development.

### Manual Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres (ensure running locally)
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"

# Run the service
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Project Structure

```
.
├── main.py                      # FastAPI app with endpoints
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## API Endpoints

### 1. Start a Research Task

**Endpoint:** `POST /generate_report`

```python
import requests
import json

# Kick off a research task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Large Language Models for scientific discovery",
        "model": "openai:gpt-4o"
    }
)

task_data = response.json()
task_id = task_data["task_id"]
print(f"Task started: {task_id}")
```

**cURL example:**
```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing advances in 2024", "model":"openai:gpt-4o"}'
```

### 2. Poll Task Progress

**Endpoint:** `GET /task_progress/{task_id}`

```python
import time

# Poll for progress updates
while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    
    print(f"Status: {progress['status']}")
    print(f"Current step: {progress.get('current_step', 'N/A')}")
    
    if progress['status'] in ['completed', 'failed']:
        break
    
    time.sleep(2)
```

**Response structure:**
```json
{
  "task_id": "uuid-here",
  "status": "in_progress",
  "current_step": "research",
  "steps": {
    "planning": "completed",
    "research": "in_progress",
    "writing": "pending",
    "editing": "pending"
  },
  "progress_percentage": 35
}
```

### 3. Get Final Results

**Endpoint:** `GET /task_status/{task_id}`

```python
# Get final report and status
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()

print(f"Status: {result['status']}")
print(f"Report:\n{result['report']}")
print(f"Metadata: {result['metadata']}")
```

## Core Components

### Planning Agent

The planner breaks down research tasks into executable substeps:

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(prompt: str, model: str = "openai:gpt-4o"):
    """
    Generate a research plan with substeps.
    
    Args:
        prompt: User's research question
        model: AI model to use
        
    Returns:
        List of substeps with tool assignments
    """
    client = Client()
    
    planning_prompt = f"""
    Given this research task: "{prompt}"
    
    Create a step-by-step plan using these agents:
    - research_agent: Gathers information from Tavily, arXiv, Wikipedia
    - writer_agent: Synthesizes findings into coherent report
    - editor_agent: Reviews and refines the final report
    
    Return JSON array of steps with: step_name, agent, tools_needed, description
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": planning_prompt}]
    )
    
    # Parse and return plan
    plan = parse_plan(response.choices[0].message.content)
    return plan
```

### Executor Agent

Executes individual plan steps:

```python
def executor_agent_step(step: dict, context: dict, model: str):
    """
    Execute a single planned step.
    
    Args:
        step: Step definition from planner
        context: Accumulated context from previous steps
        model: AI model to use
        
    Returns:
        Step result to add to context
    """
    agent_name = step['agent']
    
    if agent_name == 'research_agent':
        from src.agents import research_agent
        result = research_agent(step['description'], context, model)
    elif agent_name == 'writer_agent':
        from src.agents import writer_agent
        result = writer_agent(context, model)
    elif agent_name == 'editor_agent':
        from src.agents import editor_agent
        result = editor_agent(context, model)
    
    return result
```

### Research Tools

Tools available to agents:

```python
# src/research_tools.py
import os
import requests
import wikipedia

def tavily_search_tool(query: str, max_results: int = 5):
    """Search using Tavily API for current web information."""
    api_key = os.getenv("TAVILY_API_KEY")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    
    return response.json()["results"]


def arxiv_search_tool(query: str, max_results: int = 3):
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
            "url": paper.entry_id
        })
    
    return results


def wikipedia_search_tool(query: str):
    """Search Wikipedia for background information."""
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=3),
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option
        return wikipedia_search_tool(e.options[0])
    except:
        return {"error": "No Wikipedia page found"}
```

### Research Agent Example

```python
# src/agents.py
from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

def research_agent(query: str, context: dict, model: str):
    """
    Execute research using multiple tools.
    """
    results = {
        "query": query,
        "sources": []
    }
    
    # Search Tavily for current info
    tavily_results = tavily_search_tool(query)
    results["sources"].extend([
        {"type": "web", "data": r} for r in tavily_results
    ])
    
    # Search arXiv for papers
    arxiv_results = arxiv_search_tool(query)
    results["sources"].extend([
        {"type": "academic", "data": r} for r in arxiv_results
    ])
    
    # Search Wikipedia for background
    wiki_result = wikipedia_search_tool(query)
    if "error" not in wiki_result:
        results["sources"].append({
            "type": "encyclopedia",
            "data": wiki_result
        })
    
    return results


def writer_agent(context: dict, model: str):
    """Synthesize research into a report."""
    client = Client()
    
    sources = context.get("research_results", {}).get("sources", [])
    
    writing_prompt = f"""
    Based on these sources, write a comprehensive research report:
    
    {json.dumps(sources, indent=2)}
    
    Structure: Introduction, Key Findings, Analysis, Conclusion
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": writing_prompt}]
    )
    
    return response.choices[0].message.content


def editor_agent(context: dict, model: str):
    """Review and refine the report."""
    client = Client()
    
    draft = context.get("draft_report", "")
    
    editing_prompt = f"""
    Review this research report and improve clarity, structure, and accuracy:
    
    {draft}
    
    Provide the final edited version.
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": editing_prompt}]
    )
    
    return response.choices[0].message.content
```

## Database Schema

Task tracking is handled via SQLAlchemy models:

```python
# main.py
from sqlalchemy import Column, String, Text, DateTime, create_engine
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
    status = Column(String, default="pending")  # pending, in_progress, completed, failed
    prompt = Column(Text)
    model = Column(String)
    plan = Column(Text)  # JSON string
    current_step = Column(String)
    result = Column(Text)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...                    # OpenAI API key
TAVILY_API_KEY=tvly-...                  # Tavily search API key

# Optional (defaults provided)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb
```

### Model Selection

Specify model when starting tasks:

```python
# Use GPT-4
{"prompt": "Your query", "model": "openai:gpt-4o"}

# Use GPT-3.5 (faster, cheaper)
{"prompt": "Your query", "model": "openai:gpt-3.5-turbo"}
```

## Common Patterns

### Async Task Processing

```python
# main.py
import threading
from fastapi import FastAPI, BackgroundTasks
import uuid

app = FastAPI()

def execute_research_workflow(task_id: str, prompt: str, model: str):
    """Background thread that runs the full workflow."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    
    try:
        # Update status
        task.status = "in_progress"
        db.commit()
        
        # Step 1: Planning
        task.current_step = "planning"
        db.commit()
        plan = planner_agent(prompt, model)
        task.plan = json.dumps(plan)
        db.commit()
        
        # Step 2: Execute plan
        context = {}
        for step in plan:
            task.current_step = step['step_name']
            db.commit()
            
            result = executor_agent_step(step, context, model)
            context[step['step_name']] = result
        
        # Step 3: Store final result
        task.status = "completed"
        task.result = context.get("final_report", "")
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
        db.commit()
    finally:
        db.close()


@app.post("/generate_report")
async def generate_report(request: dict):
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        prompt=request["prompt"],
        model=request.get("model", "openai:gpt-4o"),
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Start background thread
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, request["prompt"], request.get("model", "openai:gpt-4o"))
    )
    thread.start()
    
    return {"task_id": task_id}
```

### Custom Agent Integration

Add your own specialized agents:

```python
# src/agents.py
def data_analysis_agent(data_context: dict, model: str):
    """Analyze numerical data and create visualizations."""
    import pandas as pd
    
    # Process data from context
    df = pd.DataFrame(data_context["raw_data"])
    
    # Generate analysis prompt
    analysis_prompt = f"""
    Analyze this dataset:
    {df.describe().to_string()}
    
    Provide key insights and trends.
    """
    
    client = Client()
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": analysis_prompt}]
    )
    
    return {
        "statistics": df.describe().to_dict(),
        "insights": response.choices[0].message.content
    }
```

### Tool Result Caching

Avoid redundant API calls:

```python
# src/research_tools.py
from functools import lru_cache

@lru_cache(maxsize=100)
def cached_tavily_search(query: str, max_results: int = 5):
    """Cached version of Tavily search."""
    return tavily_search_tool(query, max_results)
```

## Troubleshooting

### Database Connection Issues

**Problem:** `DATABASE_URL not set` error

**Solution:**
```bash
# Ensure DATABASE_URL is exported in entrypoint
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"

# Or verify in Docker logs
docker logs fpsvc | grep DATABASE_URL
```

### API Key Errors

**Problem:** `401 Unauthorized` from Tavily/OpenAI

**Solution:**
```bash
# Verify .env file exists and is loaded
cat .env

# Check keys are passed to container
docker run --rm -it --env-file .env fastapi-postgres-service \
  bash -c 'echo $OPENAI_API_KEY && echo $TAVILY_API_KEY'
```

### Task Stuck in Progress

**Problem:** Task status never updates

**Solution:**
```python
# Check for exceptions in background thread
# Add logging to executor
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def execute_research_workflow(task_id: str, prompt: str, model: str):
    logger.info(f"Starting workflow for task {task_id}")
    try:
        # ... workflow code
        logger.info(f"Step completed: {step['step_name']}")
    except Exception as e:
        logger.error(f"Workflow failed: {e}", exc_info=True)
```

### Templates Not Found

**Problem:** `TemplateNotFound: index.html`

**Solution:**
```bash
# Verify templates copied into container
docker exec -it fpsvc ls -la /app/templates/

# Ensure Dockerfile includes:
# COPY templates/ /app/templates/
```

### Rate Limiting

**Problem:** Wikipedia/arXiv returning errors

**Solution:**
```python
# Add retry logic with exponential backoff
import time
from functools import wraps

def retry_with_backoff(max_retries=3, backoff_factor=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    wait_time = backoff_factor ** attempt
                    time.sleep(wait_time)
        return wrapper
    return decorator

@retry_with_backoff()
def wikipedia_search_tool(query: str):
    # ... existing code
```

### Postgres Not Starting

**Problem:** Container exits with Postgres errors

**Solution:**
```bash
# Check entrypoint script has correct permissions
chmod +x docker/entrypoint.sh

# Verify Postgres version in entrypoint
# Use: pg_ctlcluster $(pg_lsclusters -h | awk '{print $1}') main start

# Check logs
docker logs fpsvc 2>&1 | grep -i postgres
```

## Web UI Access

Open browser to `http://localhost:8000/` to use the simple UI:

1. Enter research prompt
2. Select model (GPT-4 or GPT-3.5)
3. Click "Start Research"
4. View real-time progress updates
5. Download final report when complete

## Production Considerations

- **Separate containers:** Run Postgres in dedicated container
- **Connection pooling:** Use SQLAlchemy pool settings
- **Task queue:** Replace threads with Celery or Redis Queue
- **API rate limiting:** Add rate limiters to prevent abuse
- **Result storage:** Archive completed reports to S3/object storage
- **Monitoring:** Add Prometheus metrics for task completion rates
