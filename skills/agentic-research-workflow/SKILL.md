---
name: agentic-research-workflow
description: FastAPI research agent service that orchestrates multi-step AI workflows with planning, tool use (Tavily, arXiv, Wikipedia), and Postgres state management
triggers:
  - set up agentic research workflow service
  - create a research agent with planning and reflection
  - build multi-step AI research workflow with FastAPI
  - implement research agent with tool calling
  - set up reflective research agent with postgres
  - create research workflow with Tavily and arXiv
  - build agentic workflow with planning and execution
  - implement research report generation agent
---

# Agentic Research Workflow

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic Research Workflow is a FastAPI-based service that implements a reflective, multi-step research agent system. It orchestrates planning, research, writing, and editing agents that work together to generate comprehensive research reports. The system uses Postgres for state management and supports tool-calling agents that can query Tavily (web search), arXiv (academic papers), and Wikipedia.

**Key capabilities:**
- Multi-agent workflow orchestration (planner → research → writer → editor)
- Tool-using agents with external API integration
- Task state tracking and progress monitoring via REST API
- Threaded, non-blocking execution
- Web UI for task submission and monitoring
- Single-container Docker deployment with Postgres

## Installation

### Docker Setup (Recommended)

1. **Clone and prepare environment:**

```bash
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file with required API keys:**

```bash
cat > .env << EOF
OPENAI_API_KEY=your_openai_key
TAVILY_API_KEY=your_tavily_key
EOF
```

3. **Build Docker image:**

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

The service will be available at `http://localhost:8000`.

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set environment variables
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="your_openai_key"
export TAVILY_API_KEY="your_tavily_key"

# Start Postgres (if not using Docker)
# Ensure Postgres is running and database 'appdb' exists

# Run the FastAPI app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                    # FastAPI application and endpoints
├── src/
│   ├── planning_agent.py      # Planner and executor agent logic
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tool definitions (Tavily, arXiv, Wikipedia)
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Docker startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core API Endpoints

### 1. Generate Research Report

```bash
POST /generate_report
```

**Request body:**

```json
{
  "prompt": "Large Language Models for scientific discovery",
  "model": "openai:gpt-4o"
}
```

**Response:**

```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Python example:**

```python
import requests

response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Impact of climate change on marine ecosystems",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task started: {task_id}")
```

### 2. Check Task Progress

```bash
GET /task_progress/{task_id}
```

**Response:**

```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "running",
  "current_step": "research",
  "steps": [
    {
      "step": "planning",
      "status": "completed",
      "substeps": [...]
    },
    {
      "step": "research",
      "status": "running",
      "substeps": [
        {"name": "tavily_search", "status": "completed"},
        {"name": "arxiv_search", "status": "running"}
      ]
    }
  ]
}
```

**Python polling example:**

```python
import requests
import time

task_id = "550e8400-e29b-41d4-a716-446655440000"

while True:
    response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    data = response.json()
    
    print(f"Status: {data['status']} - Current step: {data.get('current_step', 'N/A')}")
    
    if data['status'] in ['completed', 'failed']:
        break
    
    time.sleep(2)
```

### 3. Get Final Report

```bash
GET /task_status/{task_id}
```

**Response:**

```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report\n\n## Introduction...",
  "created_at": "2025-01-15T10:30:00",
  "updated_at": "2025-01-15T10:35:00"
}
```

**Python example:**

```python
import requests

task_id = "550e8400-e29b-41d4-a716-446655440000"
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
data = response.json()

if data['status'] == 'completed':
    print("Report generated successfully:")
    print(data['report'])
else:
    print(f"Task status: {data['status']}")
```

## Building Custom Agents

### Research Tool Implementation

```python
# src/research_tools.py
import requests
import os

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search the web using Tavily API.
    
    Args:
        query: Search query string
        max_results: Maximum number of results to return
        
    Returns:
        List of search results with title, url, and snippet
    """
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        raise ValueError("TAVILY_API_KEY not set")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results,
            "search_depth": "advanced"
        }
    )
    response.raise_for_status()
    return response.json().get("results", [])


def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search arXiv for academic papers.
    
    Args:
        query: Search query
        max_results: Maximum papers to retrieve
        
    Returns:
        List of papers with title, authors, summary, and pdf_url
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
    Search Wikipedia and return summary.
    
    Args:
        query: Topic to search
        
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
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first suggestion
        page = wikipedia.page(e.options[0])
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.PageError:
        return {"error": f"No Wikipedia page found for '{query}'"}
```

### Agent Implementation Pattern

```python
# src/agents.py
import aisuite as ai

client = ai.Client()

def research_agent(topic: str, model: str = "openai:gpt-4o") -> dict:
    """
    Research agent that uses multiple tools to gather information.
    
    Args:
        topic: Research topic
        model: LLM model to use
        
    Returns:
        Dictionary with research findings
    """
    from src.research_tools import (
        tavily_search_tool,
        arxiv_search_tool,
        wikipedia_search_tool
    )
    
    # Define available tools
    tools = [
        {
            "type": "function",
            "function": {
                "name": "tavily_search",
                "description": "Search the web for current information",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string", "description": "Search query"}
                    },
                    "required": ["query"]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "arxiv_search",
                "description": "Search arXiv for academic papers",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string", "description": "Research topic"}
                    },
                    "required": ["query"]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "wikipedia_search",
                "description": "Get Wikipedia summary on a topic",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string", "description": "Topic name"}
                    },
                    "required": ["query"]
                }
            }
        }
    ]
    
    messages = [
        {
            "role": "system",
            "content": "You are a research assistant. Use available tools to gather comprehensive information."
        },
        {
            "role": "user",
            "content": f"Research the following topic and provide comprehensive findings: {topic}"
        }
    ]
    
    # Initial call
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        tools=tools,
        tool_choice="auto"
    )
    
    # Handle tool calls
    tool_results = []
    while response.choices[0].message.tool_calls:
        for tool_call in response.choices[0].message.tool_calls:
            function_name = tool_call.function.name
            arguments = eval(tool_call.function.arguments)
            
            # Execute tool
            if function_name == "tavily_search":
                result = tavily_search_tool(arguments["query"])
            elif function_name == "arxiv_search":
                result = arxiv_search_tool(arguments["query"])
            elif function_name == "wikipedia_search":
                result = wikipedia_search_tool(arguments["query"])
            else:
                result = {"error": "Unknown tool"}
            
            tool_results.append({
                "tool": function_name,
                "query": arguments.get("query"),
                "result": result
            })
            
            # Add tool response to messages
            messages.append({
                "role": "assistant",
                "content": None,
                "tool_calls": [tool_call]
            })
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": str(result)
            })
        
        # Continue conversation
        response = client.chat.completions.create(
            model=model,
            messages=messages,
            tools=tools
        )
    
    return {
        "findings": response.choices[0].message.content,
        "tool_results": tool_results
    }


def writer_agent(research_data: dict, model: str = "openai:gpt-4o") -> str:
    """
    Writer agent that creates a structured report from research findings.
    
    Args:
        research_data: Dictionary containing research findings
        model: LLM model to use
        
    Returns:
        Formatted research report as markdown
    """
    messages = [
        {
            "role": "system",
            "content": (
                "You are an expert technical writer. Create a comprehensive, "
                "well-structured research report in markdown format with proper "
                "sections, citations, and clear explanations."
            )
        },
        {
            "role": "user",
            "content": f"Create a research report from these findings:\n\n{research_data}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content


def editor_agent(draft_report: str, model: str = "openai:gpt-4o") -> str:
    """
    Editor agent that refines and improves the draft report.
    
    Args:
        draft_report: Draft report text
        model: LLM model to use
        
    Returns:
        Edited and polished report
    """
    messages = [
        {
            "role": "system",
            "content": (
                "You are an expert editor. Review and improve the report for "
                "clarity, coherence, accuracy, and professional presentation. "
                "Maintain markdown formatting."
            )
        },
        {
            "role": "user",
            "content": f"Edit and improve this research report:\n\n{draft_report}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Planning Agent Pattern

```python
# src/planning_agent.py
import aisuite as ai
import json

client = ai.Client()

def planner_agent(task_prompt: str, model: str = "openai:gpt-4o") -> dict:
    """
    Planning agent that creates a multi-step execution plan.
    
    Args:
        task_prompt: User's research request
        model: LLM model to use
        
    Returns:
        Dictionary with execution plan
    """
    messages = [
        {
            "role": "system",
            "content": (
                "You are a planning agent. Create a detailed execution plan "
                "with clear steps (research, writing, editing). Return a JSON "
                "object with 'steps' array containing objects with 'name', "
                "'description', and 'substeps' fields."
            )
        },
        {
            "role": "user",
            "content": f"Create an execution plan for: {task_prompt}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        response_format={"type": "json_object"}
    )
    
    plan = json.loads(response.choices[0].message.content)
    return plan


def executor_agent_step(step: dict, context: dict, model: str = "openai:gpt-4o") -> dict:
    """
    Execute a single step from the plan.
    
    Args:
        step: Step definition from planner
        context: Current execution context
        model: LLM model to use
        
    Returns:
        Step execution results
    """
    from src.agents import research_agent, writer_agent, editor_agent
    
    step_name = step.get("name", "").lower()
    
    if "research" in step_name:
        result = research_agent(context.get("topic"), model)
        context["research_data"] = result
        return result
    
    elif "writ" in step_name:
        result = writer_agent(context.get("research_data"), model)
        context["draft_report"] = result
        return {"report": result}
    
    elif "edit" in step_name:
        result = editor_agent(context.get("draft_report"), model)
        context["final_report"] = result
        return {"report": result}
    
    else:
        return {"error": f"Unknown step type: {step_name}"}
```

## Database Models

```python
# main.py (excerpt)
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True, index=True)
    status = Column(String, default="pending")  # pending, running, completed, failed
    prompt = Column(Text)
    model = Column(String)
    report = Column(Text, nullable=True)
    progress = Column(Text, nullable=True)  # JSON string
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | Postgres connection string (set by entrypoint) |
| `OPENAI_API_KEY` | Yes | OpenAI API key for LLM calls |
| `TAVILY_API_KEY` | Yes | Tavily API key for web search |
| `POSTGRES_USER` | No | Postgres username (default: `app`) |
| `POSTGRES_PASSWORD` | No | Postgres password (default: `local`) |
| `POSTGRES_DB` | No | Database name (default: `appdb`) |

### Docker Configuration

**Dockerfile excerpt:**

```dockerfile
FROM python:3.11-slim

# Install Postgres
RUN apt-get update && apt-get install -y \
    postgresql \
    postgresql-contrib \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000 5432

ENTRYPOINT ["docker/entrypoint.sh"]
```

**entrypoint.sh pattern:**

```bash
#!/bin/bash
set -e

# Start Postgres
pg_ctlcluster $(pg_lsclusters -h | awk '{print $1}') main start

# Wait for Postgres
until su -s /bin/bash postgres -c "psql -c 'SELECT 1'" > /dev/null 2>&1; do
  sleep 1
done

# Create user and database
su -s /bin/bash postgres -c "psql -c \"CREATE ROLE app WITH LOGIN PASSWORD 'local';\""
su -s /bin/bash postgres -c "psql -c \"CREATE DATABASE appdb OWNER app;\""

# Set DATABASE_URL
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"

# Start FastAPI
exec uvicorn main:app --host 0.0.0.0 --port 8000
```

## Common Patterns

### Threaded Task Execution

```python
import threading
import uuid
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

def execute_research_workflow(task_id: str, prompt: str, model: str):
    """
    Background task that executes the full research workflow.
    """
    db = SessionLocal()
    try:
        # Update status
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "running"
        db.commit()
        
        # Execute plan
        plan = planner_agent(prompt, model)
        context = {"topic": prompt, "plan": plan}
        
        for step in plan.get("steps", []):
            result = executor_agent_step(step, context, model)
            # Update progress in DB
            task.progress = json.dumps({"current_step": step["name"], "result": result})
            db.commit()
        
        # Save final report
        task.report = context.get("final_report", "No report generated")
        task.status = "completed"
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()


@app.post("/generate_report")
async def generate_report(request: dict, background_tasks: BackgroundTasks):
    """
    Kick off a research workflow in the background.
    """
    task_id = str(uuid.uuid4())
    prompt = request.get("prompt")
    model = request.get("model", "openai:gpt-4o")
    
    # Create task record
    db = SessionLocal()
    task = Task(task_id=task_id, prompt=prompt, model=model)
    db.add(task)
    db.commit()
    db.close()
    
    # Start background thread
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, prompt, model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

### Progress Tracking

```python
import json

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """
    Get detailed progress for a running task.
    """
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    progress_data = {}
    if task.progress:
        progress_data = json.loads(task.progress)
    
    return {
        "task_id": task_id,
        "status": task.status,
        "current_step": progress_data.get("current_step"),
        "progress": progress_data
    }
```

### Error Handling Pattern

```python
def safe_tool_call(tool_func, *args, **kwargs):
    """
    Safely execute a tool call with error handling.
    """
    try:
        return {
            "success": True,
            "data": tool_func(*args, **kwargs)
        }
    except requests.exceptions.RequestException as e:
        return {
            "success": False,
            "error": f"Network error: {str(e)}"
        }
    except Exception as e:
        return {
            "success": False,
            "error": f"Tool error: {str(e)}"
        }

# Usage in agent
result = safe_tool_call(tavily_search_tool, "quantum computing")
if result["success"]:
    data = result["data"]
else:
    print(f"Tool failed: {result['error']}")
```

## Troubleshooting

### Database Connection Issues

**Problem:** `DATABASE_URL not set` error

**Solution:**
```bash
# Verify environment variable is set
docker exec -it fpsvc bash -c 'echo $DATABASE_URL'

# Should output: postgresql://app:local@127.0.0.1:5432/appdb

# If empty, check entrypoint.sh exports it correctly
```

**Problem:** Tables not created

**Solution:**
```python
# In main.py, ensure tables are created on startup
from sqlalchemy import inspect

def init_db():
    inspector = inspect(engine)
    if not inspector.has_table("tasks"):
        Base.metadata.create_all(bind=engine)
        print("✅ Database tables created")
    else:
        print("✅ Database tables already exist")

# Call on startup
init_db()
```

### API Key Issues

**Problem:** Tavily search fails with authentication error

**Solution:**
```bash
# Verify API key is loaded
docker exec -it fpsvc bash -c 'echo $TAVILY_API_KEY | head -c 20'

# Ensure .env file is passed to docker run
docker run --env-file .env ...

# Or set explicitly
docker run -e TAVILY_API_KEY=your_key ...
```

**Problem:** OpenAI API errors

**Solution:**
```python
# Add retry logic for API calls
import time
from openai import RateLimitError

def call_llm_with_retry(client, **kwargs):
    max_retries = 3
    for attempt in range(max_retries):
        try:
            return client.chat.completions.create(**kwargs)
        except RateLimitError:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)  # Exponential backoff
                continue
            raise
```

### Task Execution Issues

**Problem:** Tasks stuck in "running" state

**Solution:**
```python
# Add timeout to background tasks
import signal

class TimeoutError(Exception):
    pass

def timeout_handler(signum, frame):
    raise TimeoutError("Task execution timeout")

def execute_with_timeout(task_func, timeout=300):
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(timeout)
    try:
        return task_func()
    finally:
        signal.alarm(0)
```

**Problem:** Memory issues with large reports

**Solution:**
```python
# Stream large responses instead of loading all at once
from fastapi.responses import StreamingResponse

@app.get("/task_report_stream/{task_id}")
async def stream_report(task_id: str):
    def generate():
        db = SessionLocal()
        task = db.query(Task).filter(Task.task_id == task_id).first()
        db.close()
        
        if task and task.report:
            # Stream in chunks
            chunk_size = 1024
            for i in range(0, len(task.report), chunk_size):
                yield task.report[i:i+chunk_size]
    
    return StreamingResponse(generate(), media_type="text/markdown")
```

### Docker Issues

**Problem:** Container exits immediately

**Solution:**
```bash
# Check logs
docker logs fpsvc

# Run interactively to debug
docker run --rm -it --entrypoint bash fastapi-postgres-service

# Inside container, manually run commands from entrypoint.sh
```

**Problem:** Postgres fails to start

**Solution:**
```bash
# Check Postgres logs
docker exec -it fpsvc tail -f /var/log/postgresql/postgresql-*-main.log

# Verify cluster exists
docker exec -it fpsvc pg_lsclusters

# Restart cluster if needed
docker exec -it fpsvc pg_ctlcluster 17 main restart
```

### Performance Optimization

**Problem:** Slow research operations

**Solution:**
```python
# Parallelize tool calls
import concurrent.futures

def parallel_research(topic: str):
    """
    Execute multiple research tools in parallel.
    """
    with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
        future_tavily = executor.submit(tavily_search_tool, topic)
        future_arxiv = executor.submit(arxiv_search_tool, topic)
        future_wiki = executor.submit(wikipedia_search_tool, topic)
        
        results = {
            "tavily": future_tavily.result(),
            "arxiv": future_arxiv.result(),
            "wikipedia": future_wiki.result()
        }
    
    return results
```

## Advanced Usage

### Custom Agent Chain

```python
def create_custom_workflow(task_id: str, prompt: str):
    """
    Create a custom agent workflow with validation and reflection.
    """
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    
    # Step 1: Research
    task.progress = json.dumps({"step": "research", "status": "running"})
    db.commit()
    research_data = research_agent(prompt)
    
    # Step 2: Validation
    task.progress = json.dumps({"step": "validation", "status": "running"})
    db.commit()
    
    messages = [
        {
            "role": "system",
            "content": "Validate research findings for accuracy and completeness."
        },
        {
            "role": "user",
            "content": f"Validate these findings:\n{research_data}"
        }
    ]
    validation = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    # Step 3: Write report
    task.progress = json.dumps({"step": "writing", "status": "running"})
    db.commit()
    draft = writer_agent(research_data)
    
    # Step 4: Reflection and improvement
    task.progress = json.dumps({"step": "reflection", "status": "running"})
    db.commit()
    
    messages = [
        {
            "role": "system",
            "content": "Reflect on the report and suggest improvements."
        },
        {
            "role": "user",
            "content": draft
        }
    ]
    reflection = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    # Step 5: Final edit
    final_report = editor_agent(draft + "\n\nReflections:\n" + reflection.choices[0].message.content)
    
    task.report = final_report
    task.status = "completed"
    db.commit()
    db.close()
```

### Web UI Integration

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Research Agent</title>
    <script>
        async function startResearch() {
            const prompt = document.getElementById('prompt').value;
            const model = document.getElementById('model').value;
            
            const response = await fetch('/generate_report', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({prompt, model})
            });
            
            const data = await response.json();
            const taskId = data.task_id;
            
            
