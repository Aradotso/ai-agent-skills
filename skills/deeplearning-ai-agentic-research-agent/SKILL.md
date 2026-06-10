---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service using Tavily, arXiv, and Wikipedia with multi-step planning and Postgres state management
triggers:
  - build a research agent with planning workflow
  - use tavily arxiv wikipedia tools in agent
  - create agentic research workflow with fastapi
  - setup multi-step research agent with postgres
  - implement reflective research agent service
  - deploy research agent with docker postgres
  - build task-based research agent api
  - create planning and execution agent workflow
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step agentic workflows with planning, research, writing, and editing phases. Uses Tavily for web search, arXiv for academic papers, Wikipedia for encyclopedic knowledge, and Postgres for task state persistence.

## What It Does

- **Planning Agent**: Creates research plans with multiple steps
- **Executor Agent**: Runs research, writer, and editor agents in sequence
- **Tool Integration**: Tavily search, arXiv search, Wikipedia lookup
- **State Management**: Tracks task progress in Postgres
- **Web UI**: Simple interface to kick off research tasks
- **Live Progress**: Real-time status updates for multi-step workflows

## Installation

### Prerequisites

```bash
# Required tools
docker --version  # Docker 20.10+

# Required API keys in .env file
cat > .env << EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF
```

### Build Docker Image

```bash
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

docker build -t fastapi-postgres-service .
```

### Run Container

```bash
# Run with Postgres + FastAPI in one container
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

Access the app:
- Web UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs
- Postgres: postgresql://app:local@localhost:5432/appdb

## Project Structure

```
.
├── main.py                    # FastAPI app with routes
├── src/
│   ├── planning_agent.py      # planner_agent(), executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # tavily, arxiv, wikipedia tools
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Postgres + Uvicorn startup
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core Components

### Research Tools (src/research_tools.py)

```python
import os
import requests
import wikipedia

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """Search web using Tavily API"""
    api_key = os.getenv("TAVILY_API_KEY")
    url = "https://api.tavily.com/search"
    
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results
    }
    
    response = requests.post(url, json=payload)
    if response.status_code == 200:
        return response.json().get("results", [])
    return []

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv for academic papers"""
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
            "authors": [author.name for author in paper.authors],
            "summary": paper.summary,
            "published": paper.published.isoformat(),
            "pdf_url": paper.pdf_url
        })
    return results

def wikipedia_search_tool(query: str) -> dict:
    """Search Wikipedia and return summary"""
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=3),
            "url": page.url,
            "content": page.content[:1000]  # First 1000 chars
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": "disambiguation", "options": e.options[:5]}
    except wikipedia.exceptions.PageError:
        return {"error": "page_not_found"}
```

### Planning Agent (src/planning_agent.py)

```python
import aisuite as ai
import json

client = ai.Client()

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> dict:
    """Create a research plan with multiple steps"""
    
    system_prompt = """You are a research planning agent. 
    Given a research topic, create a structured plan with 3-5 steps.
    Return JSON: {"steps": [{"name": "...", "description": "...", "tools": [...]}]}
    Available tools: tavily_search, arxiv_search, wikipedia_search"""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": f"Create research plan for: {prompt}"}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    content = response.choices[0].message.content
    try:
        plan = json.loads(content)
        return plan
    except json.JSONDecodeError:
        return {"steps": [{"name": "research", "description": content, "tools": ["tavily_search"]}]}

def executor_agent_step(step: dict, context: dict, model: str = "openai:gpt-4o") -> dict:
    """Execute a single research step using appropriate tools"""
    from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    
    tools_map = {
        "tavily_search": tavily_search_tool,
        "arxiv_search": arxiv_search_tool,
        "wikipedia_search": wikipedia_search_tool
    }
    
    results = {}
    for tool_name in step.get("tools", []):
        if tool_name in tools_map:
            query = f"{context.get('topic', '')} {step.get('description', '')}"
            results[tool_name] = tools_map[tool_name](query)
    
    return {
        "step_name": step["name"],
        "results": results,
        "status": "completed"
    }
```

### FastAPI Main App (main.py)

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, String, DateTime, Text, JSON
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
import os
import uuid
import threading
from datetime import datetime
from src.planning_agent import planner_agent, executor_agent_step

app = FastAPI(title="Agentic Research Agent")

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class TaskDB(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text)
    model = Column(String)
    status = Column(String, default="pending")
    plan = Column(JSON)
    progress = Column(JSON, default={})
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

Base.metadata.create_all(bind=engine)

# Templates
templates = Jinja2Templates(directory="templates")
app.mount("/static", StaticFiles(directory="static"), name="static")

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.get("/")
async def index(request: Request):
    """Render web UI"""
    return templates.TemplateResponse("index.html", {"request": request})

@app.post("/generate_report")
async def generate_report(req: ResearchRequest):
    """Start a research task"""
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    task = TaskDB(
        task_id=task_id,
        prompt=req.prompt,
        model=req.model,
        status="planning"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background
    thread = threading.Thread(target=run_research_workflow, args=(task_id, req.prompt, req.model))
    thread.start()
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """Get live progress updates"""
    db = SessionLocal()
    task = db.query(TaskDB).filter(TaskDB.task_id == task_id).first()
    db.close()
    
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    return {
        "task_id": task_id,
        "status": task.status,
        "progress": task.progress,
        "plan": task.plan
    }

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get final task status and report"""
    db = SessionLocal()
    task = db.query(TaskDB).filter(TaskDB.task_id == task_id).first()
    db.close()
    
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    return {
        "task_id": task_id,
        "status": task.status,
        "report": task.report,
        "created_at": task.created_at.isoformat(),
        "updated_at": task.updated_at.isoformat()
    }

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Execute the full research workflow"""
    db = SessionLocal()
    task = db.query(TaskDB).filter(TaskDB.task_id == task_id).first()
    
    try:
        # Step 1: Planning
        plan = planner_agent(prompt, model)
        task.plan = plan
        task.status = "executing"
        db.commit()
        
        # Step 2: Execute each step
        context = {"topic": prompt}
        all_results = []
        
        for i, step in enumerate(plan.get("steps", [])):
            task.progress = {"current_step": i + 1, "total_steps": len(plan["steps"]), "step_name": step["name"]}
            db.commit()
            
            result = executor_agent_step(step, context, model)
            all_results.append(result)
        
        # Step 3: Generate final report
        task.status = "finalizing"
        db.commit()
        
        report = generate_final_report(prompt, all_results, model)
        task.report = report
        task.status = "completed"
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()

def generate_final_report(prompt: str, results: list, model: str) -> str:
    """Compile research results into a final report"""
    import aisuite as ai
    
    client = ai.Client()
    
    system_prompt = "You are a research report writer. Synthesize the research findings into a coherent report."
    user_prompt = f"Topic: {prompt}\n\nFindings:\n{json.dumps(results, indent=2)}\n\nWrite a comprehensive report."
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

## API Usage

### Start Research Task

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'

# Response:
# {"task_id": "550e8400-e29b-41d4-a716-446655440000"}
```

### Check Progress

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000

# Response:
# {
#   "task_id": "550e8400-e29b-41d4-a716-446655440000",
#   "status": "executing",
#   "progress": {
#     "current_step": 2,
#     "total_steps": 4,
#     "step_name": "literature_review"
#   },
#   "plan": {...}
# }
```

### Get Final Report

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000

# Response:
# {
#   "task_id": "...",
#   "status": "completed",
#   "report": "# Research Report\n\n...",
#   "created_at": "2025-01-15T10:30:00",
#   "updated_at": "2025-01-15T10:35:00"
# }
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Database (optional, defaults provided)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on startup
```

### Database Connection from Host

```bash
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT task_id, status, created_at FROM tasks ORDER BY created_at DESC LIMIT 10;
```

## Common Patterns

### Custom Research Agent

```python
# src/agents.py
import aisuite as ai

def research_agent(query: str, tools_results: dict, model: str = "openai:gpt-4o") -> str:
    """Analyze tool results and extract key insights"""
    client = ai.Client()
    
    system_prompt = """You are a research analyst. 
    Review the search results and extract key insights, facts, and citations."""
    
    user_prompt = f"Query: {query}\n\nResults:\n{json.dumps(tools_results, indent=2)}\n\nProvide analysis."
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.5
    )
    
    return response.choices[0].message.content

def writer_agent(research_findings: list, model: str = "openai:gpt-4o") -> str:
    """Write a draft report from research findings"""
    client = ai.Client()
    
    system_prompt = "You are a technical writer. Create a well-structured report from research findings."
    user_prompt = f"Findings:\n{json.dumps(research_findings, indent=2)}\n\nWrite report."
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content

def editor_agent(draft: str, model: str = "openai:gpt-4o") -> str:
    """Edit and polish the draft report"""
    client = ai.Client()
    
    system_prompt = "You are an editor. Improve clarity, fix errors, ensure coherent structure."
    user_prompt = f"Draft:\n{draft}\n\nEdit and finalize."
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    return response.choices[0].message.content
```

### Add Custom Tool

```python
# src/research_tools.py
def pubmed_search_tool(query: str, max_results: int = 5) -> list:
    """Search PubMed for medical research"""
    from Bio import Entrez
    
    Entrez.email = "your-email@example.com"  # Required by NCBI
    
    handle = Entrez.esearch(db="pubmed", term=query, retmax=max_results)
    record = Entrez.read(handle)
    handle.close()
    
    results = []
    for pmid in record["IdList"]:
        handle = Entrez.efetch(db="pubmed", id=pmid, rettype="abstract", retmode="text")
        results.append({
            "pmid": pmid,
            "abstract": handle.read()
        })
        handle.close()
    
    return results
```

### Streaming Progress Updates

```python
from fastapi.responses import StreamingResponse
import json
import asyncio

@app.get("/task_progress_stream/{task_id}")
async def task_progress_stream(task_id: str):
    """Server-Sent Events stream for live progress"""
    
    async def event_generator():
        db = SessionLocal()
        while True:
            task = db.query(TaskDB).filter(TaskDB.task_id == task_id).first()
            if not task:
                break
            
            data = json.dumps({
                "status": task.status,
                "progress": task.progress
            })
            yield f"data: {data}\n\n"
            
            if task.status in ["completed", "failed"]:
                break
            
            await asyncio.sleep(1)
        
        db.close()
    
    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

## Troubleshooting

### Templates Not Found

```bash
# Verify templates exist in container
docker exec -it fpsvc ls -l /app/templates

# Ensure Dockerfile copies templates
# Dockerfile should have:
# COPY templates/ /app/templates/
```

### Database Connection Failed

```bash
# Check DATABASE_URL is set
docker exec -it fpsvc bash -lc 'echo $DATABASE_URL'

# Test connection
docker exec -it fpsvc bash -lc 'psql $DATABASE_URL -c "SELECT 1"'

# Check Postgres is running
docker exec -it fpsvc pg_lsclusters
```

### API Keys Not Working

```bash
# Verify env vars are passed
docker exec -it fpsvc bash -lc 'env | grep API_KEY'

# Re-run with explicit env vars
docker run --rm -it \
  -p 8000:8000 \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  fastapi-postgres-service
```

### Tasks Stuck in Pending

```bash
# Check logs for exceptions
docker logs -f fpsvc

# Query task status directly
docker exec -it fpsvc bash -lc "psql \$DATABASE_URL -c 'SELECT task_id, status, updated_at FROM tasks ORDER BY created_at DESC LIMIT 5;'"

# Check thread execution
# Add logging to run_research_workflow()
```

### Tool Rate Limits

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
def tavily_search_tool(query: str, max_results: int = 5) -> list:
    # ... existing implementation
    pass
```

### Development with Hot Reload

```bash
# Mount code as volume for live changes
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Production Deployment

```bash
# Use separate Postgres instance
export DATABASE_URL="postgresql://user:pass@prod-db:5432/appdb"

# Run without exposing Postgres port
docker run -d \
  -p 8000:8000 \
  -e DATABASE_URL=$DATABASE_URL \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  --name research-agent \
  fastapi-postgres-service

# Use process manager for production
# Install gunicorn in requirements.txt
# CMD: gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```
