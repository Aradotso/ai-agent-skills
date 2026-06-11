---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service using Tavily, arXiv, and Wikipedia with multi-step planning and task tracking
triggers:
  - how do I use the agentic research agent
  - set up the DeepLearning AI research agent service
  - create a research workflow with planning agents
  - integrate Tavily and arXiv search tools
  - track multi-step agent task progress
  - build a reflective research agent with FastAPI
  - use the agentic AI research workflow
  - implement multi-agent research system
---

# DeepLearning AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step research workflows using planning agents and specialized tools (Tavily, arXiv, Wikipedia). Features task state management with Postgres, live progress tracking, and a complete web UI for research report generation.

## What It Does

This project implements a **reflective research agent** that:

- Plans multi-step research workflows using a planner agent
- Executes research using specialized agents (research, writer, editor)
- Integrates multiple search tools (Tavily web search, arXiv papers, Wikipedia)
- Tracks task progress in real-time via Postgres database
- Provides both API and web UI interfaces
- Runs entirely in a single Docker container (Postgres + FastAPI)

The architecture follows an agentic workflow pattern: planner → executor → substeps → final report.

## Installation

### Docker Setup (Recommended)

1. **Clone the repository:**

```bash
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
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

The service will start Postgres, create the database schema, and launch the FastAPI app.

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set environment variables
export OPENAI_API_KEY=your-openai-key
export TAVILY_API_KEY=your-tavily-key
export DATABASE_URL=postgresql://user:password@localhost:5432/appdb

# Run the application
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
agentic-ai-public/
├── main.py                    # FastAPI app, routes, database models
├── src/
│   ├── planning_agent.py      # Planner and executor agent logic
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── Dockerfile
├── requirements.txt
└── .env                       # API keys (create this)
```

## API Reference

### Start a Research Task

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

**Curl equivalent:**

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications", "model":"openai:gpt-4o"}'
```

### Poll Task Progress

```python
import requests
import time

task_id = "your-task-id"

while True:
    progress = requests.get(
        f"http://localhost:8000/task_progress/{task_id}"
    ).json()
    
    print(f"Status: {progress['status']}")
    print(f"Current step: {progress.get('current_step', 'N/A')}")
    
    if progress['status'] in ['completed', 'failed']:
        break
    
    time.sleep(2)
```

### Get Final Report

```python
import requests

task_id = "your-task-id"
result = requests.get(
    f"http://localhost:8000/task_status/{task_id}"
).json()

if result['status'] == 'completed':
    print(result['final_report'])
else:
    print(f"Task failed: {result.get('error')}")
```

## Core Components

### Planning Agent Implementation

The planner agent breaks down research queries into executable steps:

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(prompt: str, model: str = "openai:gpt-4o"):
    """Generate a research plan from a user prompt."""
    client = Client()
    
    system_prompt = """You are a research planning agent. 
    Break down the research query into clear, actionable steps.
    Return a structured plan with specific tasks."""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content
```

### Research Tools

**Tavily Web Search:**

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> str:
    """Search the web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return "Error: TAVILY_API_KEY not set"
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    
    if response.status_code == 200:
        results = response.json().get("results", [])
        return "\n\n".join([
            f"**{r['title']}**\n{r['content']}\nSource: {r['url']}"
            for r in results
        ])
    else:
        return f"Error: {response.status_code}"
```

**arXiv Paper Search:**

```python
import requests
from xml.etree import ElementTree

def arxiv_search_tool(query: str, max_results: int = 5) -> str:
    """Search arXiv for academic papers."""
    url = "http://export.arxiv.org/api/query"
    params = {
        "search_query": f"all:{query}",
        "max_results": max_results
    }
    
    response = requests.get(url, params=params)
    root = ElementTree.fromstring(response.content)
    
    entries = []
    for entry in root.findall('{http://www.w3.org/2005/Atom}entry'):
        title = entry.find('{http://www.w3.org/2005/Atom}title').text
        summary = entry.find('{http://www.w3.org/2005/Atom}summary').text
        link = entry.find('{http://www.w3.org/2005/Atom}id').text
        
        entries.append(f"**{title}**\n{summary}\n{link}")
    
    return "\n\n".join(entries)
```

**Wikipedia Search:**

```python
import wikipedia

def wikipedia_search_tool(query: str) -> str:
    """Search Wikipedia for background information."""
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return f"**{page.title}**\n\n{page.summary}\n\nURL: {page.url}"
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option if ambiguous
        return wikipedia_search_tool(e.options[0])
    except wikipedia.exceptions.PageError:
        return f"No Wikipedia page found for '{query}'"
```

### Agent Implementations

**Research Agent:**

```python
# src/agents.py
from aisuite import Client
from research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(task: str, model: str = "openai:gpt-4o") -> str:
    """Execute research using available tools."""
    client = Client()
    
    # Gather information from multiple sources
    web_results = tavily_search_tool(task, max_results=3)
    arxiv_results = arxiv_search_tool(task, max_results=3)
    
    # Synthesize findings
    prompt = f"""Research task: {task}

Web search results:
{web_results}

Academic papers:
{arxiv_results}

Synthesize these findings into a comprehensive research summary."""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a research analyst."},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content
```

**Writer Agent:**

```python
def writer_agent(research_data: str, model: str = "openai:gpt-4o") -> str:
    """Transform research into a structured report."""
    client = Client()
    
    prompt = f"""Based on this research data:

{research_data}

Write a clear, well-structured report with:
- Executive summary
- Key findings
- Detailed analysis
- Conclusions"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a technical writer."},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content
```

**Editor Agent:**

```python
def editor_agent(draft: str, model: str = "openai:gpt-4o") -> str:
    """Review and refine the report."""
    client = Client()
    
    prompt = f"""Review this draft report and improve it:

{draft}

Focus on:
- Clarity and coherence
- Grammar and style
- Logical flow
- Accuracy of claims"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a technical editor."},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content
```

## Database Models

The service uses SQLAlchemy for task tracking:

```python
# main.py
from sqlalchemy import Column, String, Text, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text)
    status = Column(String)  # 'running', 'completed', 'failed'
    current_step = Column(String)
    plan = Column(Text)
    research_data = Column(Text)
    draft_report = Column(Text)
    final_report = Column(Text)
    error = Column(Text, nullable=True)

# Initialize database
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Complete Workflow Example

```python
import threading
from uuid import uuid4
from sqlalchemy.orm import Session

def execute_workflow(task_id: str, prompt: str, model: str, db: Session):
    """Execute complete research workflow in background."""
    try:
        # Step 1: Planning
        db.query(Task).filter_by(task_id=task_id).update({
            "status": "running",
            "current_step": "planning"
        })
        db.commit()
        
        plan = planner_agent(prompt, model)
        db.query(Task).filter_by(task_id=task_id).update({"plan": plan})
        db.commit()
        
        # Step 2: Research
        db.query(Task).filter_by(task_id=task_id).update({
            "current_step": "researching"
        })
        db.commit()
        
        research_data = research_agent(prompt, model)
        db.query(Task).filter_by(task_id=task_id).update({
            "research_data": research_data
        })
        db.commit()
        
        # Step 3: Writing
        db.query(Task).filter_by(task_id=task_id).update({
            "current_step": "writing"
        })
        db.commit()
        
        draft = writer_agent(research_data, model)
        db.query(Task).filter_by(task_id=task_id).update({
            "draft_report": draft
        })
        db.commit()
        
        # Step 4: Editing
        db.query(Task).filter_by(task_id=task_id).update({
            "current_step": "editing"
        })
        db.commit()
        
        final_report = editor_agent(draft, model)
        db.query(Task).filter_by(task_id=task_id).update({
            "final_report": final_report,
            "status": "completed",
            "current_step": "done"
        })
        db.commit()
        
    except Exception as e:
        db.query(Task).filter_by(task_id=task_id).update({
            "status": "failed",
            "error": str(e)
        })
        db.commit()

# FastAPI endpoint
@app.post("/generate_report")
def generate_report(request: dict):
    task_id = str(uuid4())
    prompt = request["prompt"]
    model = request.get("model", "openai:gpt-4o")
    
    # Create task record
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        prompt=prompt,
        status="queued"
    )
    db.add(task)
    db.commit()
    
    # Start background thread
    thread = threading.Thread(
        target=execute_workflow,
        args=(task_id, prompt, model, db)
    )
    thread.start()
    
    return {"task_id": task_id}
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key
TAVILY_API_KEY=tvly-...            # Tavily search API key

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional PostgreSQL overrides
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0              # Set to 1 to drop tables on start
```

### Docker Compose (Alternative)

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
      - "5432:5432"
    env_file:
      - .env
    volumes:
      - pgdata:/var/lib/postgresql/17/main

volumes:
  pgdata:
```

Run with: `docker-compose up`

## Common Patterns

### Custom Tool Integration

Add a new research tool:

```python
# src/research_tools.py
def custom_api_tool(query: str) -> str:
    """Integrate a custom data source."""
    api_key = os.getenv("CUSTOM_API_KEY")
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    return response.json()["results"]

# Use in research agent
def research_agent(task: str, model: str = "openai:gpt-4o") -> str:
    web = tavily_search_tool(task)
    papers = arxiv_search_tool(task)
    custom = custom_api_tool(task)  # Add custom tool
    
    # Synthesize all sources...
```

### Multi-Model Support

Use different models for different agents:

```python
def execute_workflow(task_id: str, prompt: str, db: Session):
    # Use GPT-4 for planning
    plan = planner_agent(prompt, model="openai:gpt-4o")
    
    # Use Claude for research
    research_data = research_agent(prompt, model="anthropic:claude-3-sonnet")
    
    # Use GPT-4 mini for writing/editing
    draft = writer_agent(research_data, model="openai:gpt-4o-mini")
    final = editor_agent(draft, model="openai:gpt-4o-mini")
```

### Progress Callbacks

Add detailed progress tracking:

```python
def update_progress(task_id: str, step: str, substep: str, db: Session):
    """Update task progress with substep details."""
    db.query(Task).filter_by(task_id=task_id).update({
        "current_step": step,
        "substep": substep  # Add substep column to model
    })
    db.commit()

def research_agent(task: str, model: str, task_id: str, db: Session) -> str:
    update_progress(task_id, "researching", "web_search", db)
    web = tavily_search_tool(task)
    
    update_progress(task_id, "researching", "arxiv_search", db)
    papers = arxiv_search_tool(task)
    
    update_progress(task_id, "researching", "synthesis", db)
    # Synthesize...
```

## Troubleshooting

### "TAVILY_API_KEY not set" Error

Ensure `.env` file exists and is loaded:

```bash
# Verify environment variables
docker exec -it fpsvc env | grep TAVILY

# If missing, recreate container with --env-file
docker run --rm -it -p 8000:8000 --env-file .env fastapi-postgres-service
```

### Database Connection Issues

Check DATABASE_URL format:

```python
# Correct format
DATABASE_URL=postgresql://user:password@host:port/database

# For Docker internal Postgres
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
```

Verify Postgres is running:

```bash
docker exec -it fpsvc pg_lsclusters
# Should show cluster 17/main as "online"
```

### Tasks Stuck in "running" Status

Check for exceptions in logs:

```bash
docker logs -f fpsvc
```

Add error handling to workflow:

```python
try:
    research_data = research_agent(prompt, model)
except Exception as e:
    db.query(Task).filter_by(task_id=task_id).update({
        "status": "failed",
        "error": f"Research failed: {str(e)}"
    })
    db.commit()
    return
```

### Web UI Not Loading

Verify template files exist:

```bash
docker exec -it fpsvc ls -la /app/templates/
docker exec -it fpsvc ls -la /app/static/
```

Check Dockerfile copies them:

```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

### Rate Limiting from APIs

Implement exponential backoff:

```python
import time

def tavily_search_tool(query: str, max_retries: int = 3) -> str:
    for attempt in range(max_retries):
        response = requests.post(...)
        
        if response.status_code == 429:  # Rate limited
            wait = 2 ** attempt
            time.sleep(wait)
            continue
        
        return response.json()
    
    return "Error: Rate limit exceeded"
```

### Memory Issues with Large Reports

Stream results instead of storing everything in memory:

```python
# Use generator for large datasets
def research_agent_streaming(task: str, model: str):
    for chunk in client.chat.completions.create(
        model=model,
        messages=[...],
        stream=True
    ):
        yield chunk.choices[0].delta.content
```

### Hot Reload for Development

Mount code volume and use `--reload`:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -v "$PWD":/app \
  --env-file .env \
  fastapi-postgres-service \
  bash -c "uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Testing

```python
# test_agents.py
import pytest
from src.research_tools import tavily_search_tool, arxiv_search_tool

def test_tavily_search():
    result = tavily_search_tool("quantum computing", max_results=2)
    assert "quantum" in result.lower()
    assert len(result) > 0

def test_arxiv_search():
    result = arxiv_search_tool("neural networks", max_results=2)
    assert "arxiv" in result.lower()

# Run tests
pytest test_agents.py
```

## Additional Resources

- **Web UI:** Access at `http://localhost:8000/`
- **API Docs:** Interactive docs at `http://localhost:8000/docs`
- **Database:** Connect with `psql "postgresql://app:local@localhost:5432/appdb"`
- **Logs:** `docker logs -f fpsvc`
