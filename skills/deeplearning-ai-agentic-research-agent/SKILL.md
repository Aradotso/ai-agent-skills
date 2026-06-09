---
name: deeplearning-ai-agentic-research-agent
description: FastAPI-based reflective research agent service with multi-step workflow planning, Tavily/arXiv/Wikipedia tools, and Postgres task tracking
triggers:
  - set up the agentic research agent service
  - create a research workflow with planning agents
  - implement multi-step research tasks with Tavily and arXiv
  - build a FastAPI research agent with task progress tracking
  - deploy the reflective research agent with Postgres
  - use the agentic AI research workflow service
  - configure research agents with Wikipedia and Tavily tools
  - track research task progress with task IDs
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI web service that orchestrates multi-step research workflows using AI agents with tool-calling capabilities. The system includes a planning agent that breaks down research queries, executor agents that use search tools (Tavily, arXiv, Wikipedia), and writer/editor agents that synthesize results. All task state and results are persisted in Postgres.

## What It Does

- **Planning Agent**: Breaks down research queries into structured, multi-step workflows
- **Research Tools**: Integrates Tavily search, arXiv papers, and Wikipedia for comprehensive research
- **Multi-Agent System**: Coordinates research, writing, and editing agents
- **Progress Tracking**: Real-time status updates for each step and substep via REST API
- **Web UI**: Simple interface for kicking off research tasks
- **Postgres Persistence**: Stores all task metadata, progress, and final reports

## Installation

### Prerequisites

1. Docker installed on your system
2. API keys in a `.env` file:

```bash
OPENAI_API_KEY=sk-your-openai-key
TAVILY_API_KEY=tvly-your-tavily-key
```

### Build and Run

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Create .env file with your API keys
cat > .env <<EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF

# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the container (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will start Postgres automatically and create the required database schema.

## Key API Endpoints

### Generate Research Report

```bash
POST /generate_report
```

Kicks off a new research task with multi-step workflow execution.

**Request Body:**
```json
{
  "prompt": "Research topic or query",
  "model": "openai:gpt-4o"
}
```

**Response:**
```json
{
  "task_id": "uuid-task-identifier"
}
```

### Get Task Progress

```bash
GET /task_progress/{task_id}
```

Returns real-time progress updates including current step, substep details, and completion status.

**Response:**
```json
{
  "task_id": "uuid",
  "status": "running",
  "current_step": 2,
  "total_steps": 5,
  "step_details": {
    "name": "research_step",
    "status": "in_progress",
    "substeps": [...]
  }
}
```

### Get Final Task Status

```bash
GET /task_status/{task_id}
```

Returns final status and completed report once the task finishes.

**Response:**
```json
{
  "task_id": "uuid",
  "status": "completed",
  "report": "Full research report text...",
  "created_at": "2025-01-15T10:30:00",
  "completed_at": "2025-01-15T10:35:00"
}
```

### Web UI

```bash
GET /
```

Serves the interactive web interface for submitting research tasks.

## Configuration

### Environment Variables

**Required:**
- `OPENAI_API_KEY`: OpenAI API key for LLM calls
- `TAVILY_API_KEY`: Tavily API key for web search

**Optional:**
- `DATABASE_URL`: Postgres connection string (default: `postgresql://app:local@127.0.0.1:5432/appdb`)
- `POSTGRES_USER`: Database user (default: `app`)
- `POSTGRES_PASSWORD`: Database password (default: `local`)
- `POSTGRES_DB`: Database name (default: `appdb`)
- `RESET_DB_ON_STARTUP`: Set to `1` to drop/recreate tables on startup (dev only)

### Database Configuration

The default setup runs Postgres inside the same container. To connect from outside:

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

For production, set `DATABASE_URL` to point to an external Postgres instance:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -e DATABASE_URL="postgresql://user:pass@external-host:5432/dbname" \
  -e OPENAI_API_KEY="$OPENAI_API_KEY" \
  -e TAVILY_API_KEY="$TAVILY_API_KEY" \
  fastapi-postgres-service
```

## Code Examples

### Basic Research Task Submission (Python)

```python
import requests
import time

# Submit research task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Latest developments in transformer architecture optimization",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task started: {task_id}")

# Poll for progress
while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    status = progress.get("status")
    print(f"Status: {status}, Step: {progress.get('current_step', 0)}/{progress.get('total_steps', 0)}")
    
    if status in ["completed", "failed"]:
        break
    time.sleep(2)

# Get final report
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
print(f"\n{result['report']}")
```

### Creating Custom Research Tools

```python
# src/research_tools.py
import os
from tavily import TavilyClient

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search the web using Tavily API.
    
    Args:
        query: Search query string
        max_results: Maximum number of results to return
    
    Returns:
        List of search results with title, url, and content
    """
    client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
    response = client.search(query=query, max_results=max_results)
    return response.get("results", [])

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search arXiv for academic papers.
    
    Args:
        query: Search query string
        max_results: Maximum number of papers to return
    
    Returns:
        List of paper metadata (title, authors, summary, pdf_url)
    """
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
            "pdf_url": paper.pdf_url,
            "published": paper.published.isoformat()
        })
    return results

def wikipedia_search_tool(query: str) -> dict:
    """
    Search Wikipedia and return page summary.
    
    Args:
        query: Topic to search for
    
    Returns:
        Dictionary with title, summary, and url
    """
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.PageError:
        return {"error": f"No Wikipedia page found for '{query}'"}
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": f"Ambiguous query. Options: {e.options[:5]}"}
```

### Implementing a Planning Agent

```python
# src/planning_agent.py
import os
import json
from typing import List, Dict

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> List[Dict]:
    """
    Break down a research query into structured workflow steps.
    
    Args:
        prompt: User's research query
        model: LLM model to use for planning
    
    Returns:
        List of workflow steps with agent assignments and tool selections
    """
    import aisuite as ai
    
    client = ai.Client()
    
    planning_prompt = f"""
You are a research planning agent. Break down this research query into concrete steps:

Query: {prompt}

Return a JSON array of steps, each with:
- step_number: integer
- agent: one of [research_agent, writer_agent, editor_agent]
- action: brief description
- tools: array of tools to use [tavily, arxiv, wikipedia]
- output: what this step produces

Example:
[
  {{"step_number": 1, "agent": "research_agent", "action": "Search recent papers", "tools": ["arxiv", "tavily"], "output": "list of relevant papers"}},
  {{"step_number": 2, "agent": "writer_agent", "action": "Draft summary", "tools": [], "output": "research summary"}},
  {{"step_number": 3, "agent": "editor_agent", "action": "Refine and format", "tools": [], "output": "final report"}}
]
"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": planning_prompt}],
        temperature=0.3
    )
    
    plan_text = response.choices[0].message.content
    # Extract JSON from response (may be wrapped in markdown)
    if "```json" in plan_text:
        plan_text = plan_text.split("```json")[1].split("```")[0]
    
    return json.loads(plan_text)


def executor_agent_step(step: Dict, context: str, model: str = "openai:gpt-4o") -> str:
    """
    Execute a single workflow step using the specified agent and tools.
    
    Args:
        step: Step configuration from planner
        context: Previous steps' results
        model: LLM model to use
    
    Returns:
        Step output as string
    """
    from .research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    
    # Gather tool results
    tool_results = []
    for tool in step.get("tools", []):
        if tool == "tavily":
            results = tavily_search_tool(context, max_results=3)
            tool_results.append(f"Tavily results: {json.dumps(results, indent=2)}")
        elif tool == "arxiv":
            results = arxiv_search_tool(context, max_results=3)
            tool_results.append(f"arXiv results: {json.dumps(results, indent=2)}")
        elif tool == "wikipedia":
            results = wikipedia_search_tool(context)
            tool_results.append(f"Wikipedia: {json.dumps(results, indent=2)}")
    
    # Combine context and tool results
    combined_context = f"{context}\n\n" + "\n\n".join(tool_results)
    
    # Execute agent action
    import aisuite as ai
    client = ai.Client()
    
    agent_prompt = f"""
You are a {step['agent']}.
Task: {step['action']}
Expected output: {step['output']}

Context and research materials:
{combined_context}

Provide your output:
"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": agent_prompt}],
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

### FastAPI Integration with Progress Tracking

```python
# main.py (excerpt)
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, String, Text, DateTime, Integer
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
import uuid
from datetime import datetime
import threading

app = FastAPI()

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text)
    model = Column(String)
    status = Column(String)  # pending, running, completed, failed
    current_step = Column(Integer, default=0)
    total_steps = Column(Integer, default=0)
    report = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

Base.metadata.create_all(bind=engine)

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background task that executes the full research workflow."""
    from src.planning_agent import planner_agent, executor_agent_step
    
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "running"
        db.commit()
        
        # Step 1: Plan workflow
        plan = planner_agent(prompt, model)
        task.total_steps = len(plan)
        db.commit()
        
        # Step 2: Execute each step
        context = prompt
        for step in plan:
            task.current_step = step["step_number"]
            db.commit()
            
            output = executor_agent_step(step, context, model)
            context += f"\n\nStep {step['step_number']} output:\n{output}"
        
        # Step 3: Store final report
        task.report = context
        task.status = "completed"
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """Start a new research task."""
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """Get current progress of a task."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "current_step": task.current_step,
        "total_steps": task.total_steps,
        "updated_at": task.updated_at.isoformat()
    }

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get final status and report."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "report": task.report,
        "created_at": task.created_at.isoformat(),
        "completed_at": task.updated_at.isoformat()
    }
```

## Common Patterns

### Long-Running Task Management

```python
# Client-side polling with timeout
import time
import requests

def wait_for_task(task_id: str, timeout: int = 300, poll_interval: int = 2):
    """Wait for task completion with timeout."""
    start_time = time.time()
    
    while time.time() - start_time < timeout:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = response.json()
        
        if data["status"] in ["completed", "failed"]:
            return requests.get(f"http://localhost:8000/task_status/{task_id}").json()
        
        time.sleep(poll_interval)
    
    raise TimeoutError(f"Task {task_id} did not complete within {timeout} seconds")
```

### Batch Research Tasks

```python
import asyncio
import aiohttp

async def submit_batch_research(queries: list[str], model: str = "openai:gpt-4o"):
    """Submit multiple research queries concurrently."""
    async with aiohttp.ClientSession() as session:
        tasks = []
        for query in queries:
            task = session.post(
                "http://localhost:8000/generate_report",
                json={"prompt": query, "model": model}
            )
            tasks.append(task)
        
        responses = await asyncio.gather(*tasks)
        task_ids = []
        for resp in responses:
            data = await resp.json()
            task_ids.append(data["task_id"])
        
        return task_ids

# Usage
queries = [
    "Quantum computing applications in drug discovery",
    "Recent advances in protein folding prediction",
    "Climate models and machine learning"
]
task_ids = asyncio.run(submit_batch_research(queries))
```

### Custom Agent Implementation

```python
# src/agents.py
from typing import List, Dict

def research_agent(query: str, tools: List[str], model: str = "openai:gpt-4o") -> str:
    """
    Research agent that uses multiple tools to gather information.
    """
    from .research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    import aisuite as ai
    
    # Gather information from tools
    context_parts = []
    
    if "tavily" in tools:
        results = tavily_search_tool(query, max_results=5)
        context_parts.append(f"Web search results:\n{results}")
    
    if "arxiv" in tools:
        papers = arxiv_search_tool(query, max_results=3)
        context_parts.append(f"Academic papers:\n{papers}")
    
    if "wikipedia" in tools:
        wiki_data = wikipedia_search_tool(query)
        context_parts.append(f"Wikipedia:\n{wiki_data}")
    
    # Synthesize findings
    context = "\n\n".join(context_parts)
    client = ai.Client()
    
    prompt = f"""Analyze the following research materials and provide a comprehensive summary:

Query: {query}

Materials:
{context}

Provide a structured summary with key findings, important details, and sources."""
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content

def writer_agent(research_summary: str, model: str = "openai:gpt-4o") -> str:
    """
    Writer agent that creates polished reports from research summaries.
    """
    import aisuite as ai
    client = ai.Client()
    
    prompt = f"""You are a technical writer. Transform this research summary into a well-structured report:

{research_summary}

Format with:
- Executive summary
- Key findings (bulleted)
- Detailed analysis
- Conclusions
- References"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content

def editor_agent(draft_report: str, model: str = "openai:gpt-4o") -> str:
    """
    Editor agent that refines and polishes the final report.
    """
    import aisuite as ai
    client = ai.Client()
    
    prompt = f"""You are an editor. Refine this report for clarity, accuracy, and professionalism:

{draft_report}

Improve:
- Grammar and style
- Flow and structure
- Clarity and precision
- Citation formatting"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content
```

## Troubleshooting

### Container Won't Start

**Symptom:** Container exits immediately or fails to start Postgres.

**Solution:**
```bash
# Check logs
docker logs fpsvc

# Verify entrypoint script is executable
docker exec -it fpsvc bash -lc "ls -l /docker/entrypoint.sh"

# Manual startup for debugging
docker run --rm -it --entrypoint bash fastapi-postgres-service
```

### Database Connection Errors

**Symptom:** `DATABASE_URL not set` or connection refused.

**Solution:**
```bash
# Verify DATABASE_URL is set correctly
docker exec -it fpsvc bash -lc "echo \$DATABASE_URL"

# Check Postgres is running
docker exec -it fpsvc bash -lc "pg_isready -h 127.0.0.1 -p 5432"

# Test connection manually
docker exec -it fpsvc bash -lc "psql \$DATABASE_URL -c 'SELECT 1'"
```

### API Key Errors

**Symptom:** `Authentication failed` or `API key not found`.

**Solution:**
```bash
# Verify .env file is being loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Ensure .env file exists and has correct format
cat .env

# Restart container with explicit env vars
docker run --rm -it \
  -p 8000:8000 \
  -e OPENAI_API_KEY="$OPENAI_API_KEY" \
  -e TAVILY_API_KEY="$TAVILY_API_KEY" \
  fastapi-postgres-service
```

### Tavily Rate Limiting

**Symptom:** `429 Too Many Requests` from Tavily API.

**Solution:**
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
                    if "429" in str(e) and attempt < max_retries - 1:
                        delay = base_delay * (2 ** attempt)
                        time.sleep(delay)
                        continue
                    raise
            return None
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3, base_delay=2)
def tavily_search_tool(query: str, max_results: int = 5):
    # ... existing implementation
    pass
```

### Tasks Stuck in "Running" State

**Symptom:** Task progress never reaches "completed" or "failed".

**Solution:**
```python
# Add timeout and error handling to workflow
import signal

class TimeoutException(Exception):
    pass

def timeout_handler(signum, frame):
    raise TimeoutException("Task execution timeout")

def run_research_workflow(task_id: str, prompt: str, model: str, timeout: int = 600):
    """Background task with timeout protection."""
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(timeout)
    
    db = SessionLocal()
    try:
        # ... existing workflow code
        signal.alarm(0)  # Cancel alarm
    except TimeoutException:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "failed"
        task.report = f"Task timeout after {timeout} seconds"
        db.commit()
    except Exception as e:
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        signal.alarm(0)
        db.close()
```

### Hot Reload for Development

**Symptom:** Need to rebuild Docker image for every code change.

**Solution:**
```bash
# Mount source code as volume for hot reload
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD:/app" \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Memory Issues with Large Reports

**Symptom:** Container crashes or becomes unresponsive with long research reports.

**Solution:**
```python
# Implement pagination for large reports
from sqlalchemy import func

@app.get("/task_report/{task_id}")
async def get_task_report(task_id: str, page: int = 1, page_size: int = 1000):
    """Get report with pagination for large outputs."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    
    if not task or not task.report:
        return {"error": "Report not found"}
    
    report = task.report
    total_length = len(report)
    start = (page - 1) * page_size
    end = start + page_size
    
    return {
        "task_id": task_id,
        "page": page,
        "total_pages": (total_length // page_size) + 1,
        "report_chunk": report[start:end],
        "status": task.status
    }
```

## Additional Resources

- **FastAPI Documentation**: https://fastapi.tiangolo.com/
- **Tavily API Docs**: https://tavily.com/docs
- **arXiv API**: https://arxiv.org/help/api/
- **Wikipedia Python Library**: https://wikipedia.readthedocs.io/
- **SQLAlchemy ORM**: https://docs.sqlalchemy.org/
