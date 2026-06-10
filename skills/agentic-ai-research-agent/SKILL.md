---
name: agentic-ai-research-agent
description: FastAPI-based reflective research agent with planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres state management
triggers:
  - set up the agentic research agent service
  - create a research workflow with planning and reflection
  - build a multi-step research agent with FastAPI
  - implement research agents using Tavily and arXiv
  - deploy the reflective research agent system
  - configure the agentic workflow research service
  - use the planning agent for research tasks
  - integrate Tavily Wikipedia and arXiv search tools
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI web service that implements a reflective research workflow. It uses a planning agent to orchestrate multiple specialized agents (research, writer, editor) that leverage external tools (Tavily search, arXiv papers, Wikipedia) to conduct comprehensive research and generate reports. All task state and results are stored in PostgreSQL.

**Key Features:**
- Multi-step agent workflow with planning, execution, and reflection
- Tool-using agents for web search, academic papers, and encyclopedia lookups
- Async task management with real-time progress tracking
- Single-container Docker deployment (Postgres + FastAPI)
- RESTful API with web UI

## Installation

### Docker (Recommended)

1. **Clone and prepare environment:**

```bash
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file:**

```bash
cat > .env << EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF
```

3. **Build the container:**

```bash
docker build -t fastapi-postgres-service .
```

4. **Run the service:**

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

### Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres separately
export DATABASE_URL="postgresql://user:pass@localhost:5432/appdb"
export OPENAI_API_KEY="your-key"
export TAVILY_API_KEY="your-key"

# Run the service
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
src/
├── planning_agent.py      # Planner and executor logic
├── agents.py              # Research, writer, editor agents
└── research_tools.py      # Tavily, arXiv, Wikipedia tools
main.py                    # FastAPI application
templates/index.html       # Web UI
docker/entrypoint.sh       # Container startup script
```

## API Reference

### Start Research Task

```bash
POST /generate_report
Content-Type: application/json

{
  "prompt": "Large Language Models for scientific discovery",
  "model": "openai:gpt-4o"
}

# Response: {"task_id": "uuid-string"}
```

**Python example:**

```python
import requests

response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Quantum computing applications in drug discovery",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task started: {task_id}")
```

### Check Progress

```bash
GET /task_progress/{task_id}
```

**Python example:**

```python
import time
import requests

def poll_progress(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = response.json()
        
        print(f"Status: {data['status']}")
        print(f"Current: {data.get('current_step', 'N/A')}")
        
        if data["status"] in ["completed", "failed"]:
            break
            
        time.sleep(2)

poll_progress(task_id)
```

### Get Final Report

```bash
GET /task_status/{task_id}
```

**Python example:**

```python
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result["status"] == "completed":
    print("Report:")
    print(result["report"])
else:
    print(f"Task {result['status']}: {result.get('error', 'Unknown error')}")
```

## Core Components

### Planning Agent

The planning agent breaks down research tasks into subtasks:

```python
# src/planning_agent.py
from typing import List, Dict

def planner_agent(prompt: str, model: str) -> List[Dict]:
    """
    Generate a research plan with multiple steps.
    
    Args:
        prompt: Research question or topic
        model: AI model identifier (e.g., "openai:gpt-4o")
    
    Returns:
        List of steps with agent assignments and descriptions
    """
    # Implementation generates structured plan
    plan = [
        {
            "step": 1,
            "agent": "research_agent",
            "description": "Gather initial sources",
            "tools": ["tavily", "arxiv"]
        },
        {
            "step": 2,
            "agent": "writer_agent",
            "description": "Draft report sections",
            "tools": []
        },
        {
            "step": 3,
            "agent": "editor_agent",
            "description": "Review and refine",
            "tools": []
        }
    ]
    return plan
```

### Research Tools

#### Tavily Search

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> List[Dict]:
    """
    Search the web using Tavily API.
    
    Args:
        query: Search query
        max_results: Maximum number of results
    
    Returns:
        List of search results with title, url, snippet
    """
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        raise ValueError("TAVILY_API_KEY not set")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    return response.json().get("results", [])
```

#### arXiv Search

```python
import requests
from xml.etree import ElementTree

def arxiv_search_tool(query: str, max_results: int = 5) -> List[Dict]:
    """
    Search arXiv for academic papers.
    
    Args:
        query: Search query
        max_results: Maximum number of papers
    
    Returns:
        List of papers with title, authors, summary, url
    """
    params = {
        "search_query": f"all:{query}",
        "start": 0,
        "max_results": max_results
    }
    
    response = requests.get(
        "http://export.arxiv.org/api/query",
        params=params
    )
    
    # Parse XML response
    root = ElementTree.fromstring(response.content)
    papers = []
    
    for entry in root.findall("{http://www.w3.org/2005/Atom}entry"):
        paper = {
            "title": entry.find("{http://www.w3.org/2005/Atom}title").text.strip(),
            "summary": entry.find("{http://www.w3.org/2005/Atom}summary").text.strip(),
            "url": entry.find("{http://www.w3.org/2005/Atom}id").text.strip()
        }
        papers.append(paper)
    
    return papers
```

#### Wikipedia Search

```python
import wikipedia

def wikipedia_search_tool(query: str, sentences: int = 3) -> Dict:
    """
    Search Wikipedia and return summary.
    
    Args:
        query: Search term
        sentences: Number of summary sentences
    
    Returns:
        Dictionary with title, summary, url
    """
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=sentences),
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Handle disambiguation by taking first option
        return wikipedia_search_tool(e.options[0], sentences)
    except wikipedia.exceptions.PageError:
        return {"error": f"No Wikipedia page found for '{query}'"}
```

### Specialized Agents

#### Research Agent

```python
# src/agents.py
def research_agent(topic: str, tools: List[str]) -> Dict:
    """
    Conduct research using specified tools.
    
    Args:
        topic: Research topic
        tools: List of tool names to use
    
    Returns:
        Compiled research findings
    """
    findings = {"sources": [], "key_points": []}
    
    if "tavily" in tools:
        web_results = tavily_search_tool(topic)
        findings["sources"].extend(web_results)
    
    if "arxiv" in tools:
        papers = arxiv_search_tool(topic)
        findings["sources"].extend(papers)
    
    if "wikipedia" in tools:
        wiki_info = wikipedia_search_tool(topic)
        findings["sources"].append(wiki_info)
    
    return findings
```

#### Writer Agent

```python
def writer_agent(research_data: Dict, outline: List[str]) -> str:
    """
    Generate report sections from research data.
    
    Args:
        research_data: Compiled research findings
        outline: Report structure
    
    Returns:
        Drafted report text
    """
    # Use LLM to synthesize findings into coherent report
    # Implementation would call AI model with research context
    report = ""
    for section in outline:
        # Generate section content
        report += f"\n## {section}\n\n"
        # ... LLM generation logic
    return report
```

#### Editor Agent

```python
def editor_agent(draft: str, criteria: List[str]) -> str:
    """
    Review and refine draft report.
    
    Args:
        draft: Initial report draft
        criteria: Editing criteria (clarity, accuracy, etc.)
    
    Returns:
        Edited report
    """
    # Use LLM to review and improve draft
    # Implementation would call AI model for editing
    return edited_report
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Optional (Docker sets defaults)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on startup
```

### Database Configuration

The service uses SQLAlchemy with PostgreSQL:

```python
# main.py
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class ResearchTask(Base):
    __tablename__ = "research_tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text)
    status = Column(String)  # pending, running, completed, failed
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Common Patterns

### Background Task Execution

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
import threading

app = FastAPI()

def execute_research_workflow(task_id: str, prompt: str, model: str):
    """Background task for research workflow."""
    db = SessionLocal()
    
    try:
        # Update status
        task = db.query(ResearchTask).filter_by(task_id=task_id).first()
        task.status = "running"
        db.commit()
        
        # Step 1: Planning
        task.current_step = "Planning"
        db.commit()
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute each step
        for step in plan:
            task.current_step = f"{step['agent']}: {step['description']}"
            db.commit()
            
            if step["agent"] == "research_agent":
                findings = research_agent(prompt, step["tools"])
            elif step["agent"] == "writer_agent":
                draft = writer_agent(findings, outline=[])
            elif step["agent"] == "editor_agent":
                final_report = editor_agent(draft, criteria=[])
        
        # Complete
        task.status = "completed"
        task.report = final_report
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
async def generate_report(request: dict, background_tasks: BackgroundTasks):
    task_id = str(uuid.uuid4())
    
    # Create task record
    db = SessionLocal()
    task = ResearchTask(
        task_id=task_id,
        prompt=request["prompt"],
        status="pending",
        created_at=datetime.utcnow()
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Start background execution
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, request["prompt"], request["model"])
    )
    thread.start()
    
    return {"task_id": task_id}
```

### Streaming Progress Updates

```python
@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    db = SessionLocal()
    task = db.query(ResearchTask).filter_by(task_id=task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}, 404
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "current_step": task.current_step,
        "updated_at": task.updated_at.isoformat() if task.updated_at else None
    }
```

## Troubleshooting

### Container Won't Start

**Postgres initialization fails:**

```bash
# Check logs
docker logs fpsvc

# Verify entrypoint is executable
docker exec -it fpsvc ls -la /docker-entrypoint.sh

# Test Postgres manually
docker exec -it fpsvc bash -lc "pg_ctlcluster 17 main status"
```

### Database Connection Issues

**"DATABASE_URL not set" error:**

```python
# Verify in container
docker exec -it fpsvc bash -lc "echo \$DATABASE_URL"

# Set explicitly if needed
docker run --rm -it \
  -e DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb" \
  -p 8000:8000 \
  fastapi-postgres-service
```

**Connection refused:**

```bash
# Check if Postgres is running
docker exec -it fpsvc bash -lc "pg_isready"

# Connect manually
docker exec -it fpsvc bash -lc "psql postgresql://app:local@127.0.0.1:5432/appdb -c 'SELECT 1;'"
```

### API Rate Limits

**Tavily rate limiting:**

```python
import time

def tavily_search_with_retry(query: str, max_retries: int = 3):
    for attempt in range(max_retries):
        try:
            return tavily_search_tool(query)
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429:
                wait_time = 2 ** attempt
                time.sleep(wait_time)
            else:
                raise
    raise Exception("Max retries exceeded for Tavily API")
```

**Wikipedia rate limiting:**

```python
import time

def safe_wikipedia_search(query: str):
    try:
        time.sleep(1)  # Respect rate limits
        return wikipedia_search_tool(query)
    except Exception as e:
        return {"error": f"Wikipedia search failed: {str(e)}"}
```

### Memory Issues with Large Reports

**Stream results instead of storing in memory:**

```python
from fastapi.responses import StreamingResponse

@app.get("/task_report/{task_id}/stream")
async def stream_report(task_id: str):
    def generate():
        db = SessionLocal()
        task = db.query(ResearchTask).filter_by(task_id=task_id).first()
        
        if task and task.report:
            # Stream in chunks
            chunk_size = 1024
            for i in range(0, len(task.report), chunk_size):
                yield task.report[i:i+chunk_size]
        
        db.close()
    
    return StreamingResponse(generate(), media_type="text/plain")
```

### Hot Reload for Development

```bash
# Mount code volume for live updates
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Connect to DB from Host

```bash
# Direct connection
psql "postgresql://app:local@localhost:5432/appdb"

# View task records
psql "postgresql://app:local@localhost:5432/appdb" -c "SELECT task_id, status, current_step FROM research_tasks;"
```

## Web UI Usage

Access the web interface at `http://localhost:8000/`:

1. Enter research topic in the form
2. Select AI model (default: openai:gpt-4o)
3. Submit to start workflow
4. Monitor progress on the status page
5. View final report when complete

The UI automatically polls `/task_progress/{task_id}` for updates and displays the report from `/task_status/{task_id}` when finished.
