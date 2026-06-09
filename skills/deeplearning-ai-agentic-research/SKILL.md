---
name: deeplearning-ai-agentic-research
description: FastAPI research agent service with planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres state management for agentic workflows
triggers:
  - how do I run the agentic research agent
  - set up the DeepLearning AI research workflow
  - use the reflective research agent with FastAPI
  - create a research task with planning agents
  - integrate Tavily and arXiv search tools
  - build agentic workflows with tool-using agents
  - deploy the research agent with Docker and Postgres
  - track multi-step agent task progress
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step agentic workflows. The system uses a planner agent to decompose research tasks, executes tool-using agents (Tavily search, arXiv, Wikipedia), and manages task state in Postgres. Designed for the Agentic Workflow course, it demonstrates reflective research patterns with step-by-step progress tracking.

## What It Does

- **Planning Agent**: Breaks down research queries into structured workflows
- **Research Agents**: Execute searches using Tavily (web), arXiv (papers), and Wikipedia
- **Writer/Editor Agents**: Draft and refine research reports
- **Progress Tracking**: Real-time task status via `/task_progress/{task_id}`
- **Postgres State**: Persistent storage of tasks, steps, and results
- **Web UI**: Simple Jinja2 interface for kicking off research tasks

## Installation

### Docker Setup (Recommended)

The project runs Postgres and FastAPI in a single container for local development.

**Prerequisites:**
- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- `.env` file with API keys

**Create `.env` file:**

```bash
# .env
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
```

**Build the image:**

```bash
docker build -t fastapi-postgres-service .
```

**Run the container:**

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

**Access the application:**
- UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs
- Postgres: `postgresql://app:local@localhost:5432/appdb`

### Local Development Setup

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
export DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
export OPENAI_API_KEY=your-openai-key
export TAVILY_API_KEY=your-tavily-key

# Run with Uvicorn
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Key API Endpoints

### `/generate_report` - Kick Off Research Task

Starts a threaded multi-step agent workflow.

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

### `/task_progress/{task_id}` - Live Status

Monitor step-by-step progress in real-time.

```python
import requests
import time

def poll_progress(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        progress = response.json()
        
        print(f"Status: {progress['status']}")
        print(f"Current Step: {progress['current_step']}")
        
        if progress['status'] in ['completed', 'failed']:
            break
        
        time.sleep(2)

poll_progress(task_id)
```

### `/task_status/{task_id}` - Final Results

Retrieve the completed report and metadata.

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result['status'] == 'completed':
    print("Report:")
    print(result['report'])
    print(f"\nCompleted at: {result['completed_at']}")
```

## Project Structure

```
.
├── main.py                      # FastAPI app with routes and DB models
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI for task creation
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core Components

### Planning Agent

The planning agent decomposes research queries into executable steps.

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(user_query: str, model: str) -> dict:
    """Generate a research plan with steps."""
    client = Client()
    
    prompt = f"""
    Create a research plan for: {user_query}
    
    Return a JSON object with:
    - steps: list of research steps
    - tools: which tools to use (tavily, arxiv, wikipedia)
    - expected_output: what the final report should contain
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return parse_plan(response.choices[0].message.content)
```

### Research Tools

Tool-using agents for different knowledge sources.

```python
# src/research_tools.py
import os
import requests
from tavily import TavilyClient

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """Search the web using Tavily."""
    client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
    
    response = client.search(
        query=query,
        search_depth="advanced",
        max_results=max_results
    )
    
    return response.get("results", [])

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
            "authors": [author.name for author in paper.authors],
            "summary": paper.summary,
            "url": paper.entry_id,
            "published": paper.published.isoformat()
        })
    
    return results

def wikipedia_search_tool(query: str) -> dict:
    """Search Wikipedia for background information."""
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url,
            "content": page.content[:2000]  # First 2000 chars
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": "disambiguation", "options": e.options[:5]}
    except wikipedia.exceptions.PageError:
        return {"error": "page_not_found"}
```

### Database Models

SQLAlchemy models for task state management.

```python
# main.py (excerpt)
from sqlalchemy import Column, String, Text, DateTime, Enum
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.dialects.postgresql import UUID
import uuid
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    prompt = Column(Text, nullable=False)
    model = Column(String(100), nullable=False)
    status = Column(
        Enum('pending', 'running', 'completed', 'failed', name='task_status'),
        default='pending'
    )
    current_step = Column(String(200))
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)
```

## Usage Patterns

### Complete Research Workflow

```python
import requests
import time
import json

class ResearchClient:
    def __init__(self, base_url="http://localhost:8000"):
        self.base_url = base_url
    
    def start_research(self, prompt: str, model: str = "openai:gpt-4o"):
        """Initiate a research task."""
        response = requests.post(
            f"{self.base_url}/generate_report",
            json={"prompt": prompt, "model": model}
        )
        response.raise_for_status()
        return response.json()["task_id"]
    
    def wait_for_completion(self, task_id: str, poll_interval: int = 3):
        """Poll until task completes."""
        while True:
            progress = requests.get(
                f"{self.base_url}/task_progress/{task_id}"
            ).json()
            
            print(f"[{progress['status']}] {progress.get('current_step', 'N/A')}")
            
            if progress['status'] in ['completed', 'failed']:
                return progress['status']
            
            time.sleep(poll_interval)
    
    def get_report(self, task_id: str):
        """Retrieve final report."""
        response = requests.get(f"{self.base_url}/task_status/{task_id}")
        response.raise_for_status()
        return response.json()

# Usage
client = ResearchClient()

# Start research
task_id = client.start_research(
    "Impact of transformer models on NLP benchmarks"
)

# Wait for completion
status = client.wait_for_completion(task_id)

# Get results
if status == 'completed':
    result = client.get_report(task_id)
    print("\n=== RESEARCH REPORT ===")
    print(result['report'])
```

### Custom Agent Step Execution

```python
# src/planning_agent.py (excerpt)
def executor_agent_step(step: dict, context: dict, model: str) -> dict:
    """Execute a single research step with appropriate tools."""
    step_type = step.get("type")
    
    if step_type == "web_search":
        from research_tools import tavily_search_tool
        results = tavily_search_tool(step["query"])
        return {"results": results, "source": "tavily"}
    
    elif step_type == "academic_search":
        from research_tools import arxiv_search_tool
        results = arxiv_search_tool(step["query"])
        return {"results": results, "source": "arxiv"}
    
    elif step_type == "background_research":
        from research_tools import wikipedia_search_tool
        results = wikipedia_search_tool(step["query"])
        return {"results": results, "source": "wikipedia"}
    
    elif step_type == "synthesis":
        # Use LLM to synthesize gathered information
        client = Client()
        response = client.chat.completions.create(
            model=model,
            messages=[
                {"role": "system", "content": "You are a research synthesizer."},
                {"role": "user", "content": f"Synthesize: {context}"}
            ]
        )
        return {"synthesis": response.choices[0].message.content}
    
    return {"error": "unknown_step_type"}
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...                          # OpenAI API key
TAVILY_API_KEY=tvly-...                        # Tavily search API key

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional overrides
POSTGRES_USER=app                              # Default: app
POSTGRES_PASSWORD=local                        # Default: local
POSTGRES_DB=appdb                              # Default: appdb

# Development
RESET_DB_ON_STARTUP=0                          # Set to 1 to drop tables on start
```

### Docker Environment

```bash
# Run with custom environment
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  -e RESET_DB_ON_STARTUP=1 \
  --name fpsvc \
  fastapi-postgres-service
```

### Hot Reload for Development

```bash
# Mount local code for live updates
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Troubleshooting

### Database Connection Issues

**Problem:** `DATABASE_URL not set` error

**Solution:** The entrypoint sets a default. If overriding, ensure correct format:

```bash
export DATABASE_URL=postgresql://user:password@host:port/database
```

**Problem:** Tables disappear on restart

**Solution:** Remove `Base.metadata.drop_all()` from `main.py` or guard it:

```python
import os
if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
```

### API Key Errors

**Problem:** `tavily_search_tool` raises authentication error

**Solution:** Verify `.env` file is loaded:

```bash
# Check inside container
docker exec -it fpsvc bash -c 'echo $TAVILY_API_KEY'

# Or pass explicitly
docker run --rm -it -e TAVILY_API_KEY=tvly-... ...
```

### Template/Static Files Not Found

**Problem:** UI shows 404 or blank page

**Solution:** Verify files are copied in Dockerfile:

```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

Check inside container:

```bash
docker exec -it fpsvc ls -la /app/templates
```

### Postgres Connection from Host

**Problem:** Can't connect to Postgres from host machine

**Solution:** Use the exposed port:

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

### Agent Timeout Issues

**Problem:** Long-running research tasks timeout

**Solution:** Increase timeout in `main.py`:

```python
import threading

def run_research_task(task_id, prompt, model):
    # Set longer timeout for LLM calls
    client = Client(timeout=120)  # 2 minutes
    # ... rest of implementation
```

### Rate Limiting

**Problem:** Wikipedia or arXiv returns rate limit errors

**Solution:** Add exponential backoff:

```python
import time
from functools import wraps

def with_retry(max_retries=3, backoff=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if i == max_retries - 1:
                        raise
                    wait = backoff ** i
                    print(f"Retry {i+1}/{max_retries} after {wait}s")
                    time.sleep(wait)
        return wrapper
    return decorator

@with_retry(max_retries=3)
def arxiv_search_tool(query: str, max_results: int = 5):
    # ... implementation
```

## Advanced Usage

### Custom Tool Integration

Add new research tools by extending `research_tools.py`:

```python
# src/research_tools.py
def pubmed_search_tool(query: str, max_results: int = 5) -> list:
    """Search PubMed for medical research papers."""
    from Bio import Entrez
    
    Entrez.email = os.getenv("ENTREZ_EMAIL", "user@example.com")
    
    handle = Entrez.esearch(db="pubmed", term=query, retmax=max_results)
    record = Entrez.read(handle)
    handle.close()
    
    ids = record["IdList"]
    # Fetch details...
    
    return results
```

Register in executor:

```python
# src/planning_agent.py
elif step_type == "medical_search":
    from research_tools import pubmed_search_tool
    results = pubmed_search_tool(step["query"])
    return {"results": results, "source": "pubmed"}
```
