---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service using LLMs with planning, tool-calling (Tavily/arXiv/Wikipedia), and PostgreSQL state tracking
triggers:
  - how do I use the agentic research agent service
  - set up the deeplearning ai research agent
  - run a multi-step research workflow with planning agents
  - integrate tavily arxiv and wikipedia tools in research agent
  - track research task progress with postgres
  - deploy the fastapi research agent container
  - create a reflective research agent workflow
  - use the planning and executor agent pattern
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI service that orchestrates multi-step research workflows using LLM-powered agents with planning, tool execution (Tavily search, arXiv papers, Wikipedia), and PostgreSQL-backed task state management. Built for the Agentic Workflow course, this demonstrates reflective agent patterns with planner → executor → research/writer/editor chains.

## What It Does

- **Planning Agent**: Breaks down research questions into structured subtasks
- **Tool-Using Agents**: Research (Tavily/arXiv/Wikipedia), Writer, Editor agents
- **Task Orchestration**: Multi-step workflows with progress tracking
- **State Management**: PostgreSQL stores task state, substeps, and results
- **Web Interface**: Simple UI + REST API for task submission and monitoring
- **Single Container**: Postgres + FastAPI bundled for easy local development

## Installation

### Prerequisites

- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- API keys in `.env` file at project root:

```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build and Run

```bash
# Clone repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build container
docker build -t fastapi-postgres-service .

# Run (exposes ports 8000 for API, 5432 for Postgres)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

Container starts Postgres, creates database schema, and launches Uvicorn on `http://0.0.0.0:8000`.

## Project Architecture

```
src/
├── planning_agent.py      # planner_agent(), executor_agent_step()
├── agents.py              # research_agent, writer_agent, editor_agent
└── research_tools.py      # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

main.py                    # FastAPI app with endpoints
templates/index.html       # Web UI
docker/entrypoint.sh       # Container initialization
```

## Core API Endpoints

### Start Research Task

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

**Response:**
```json
{"task_id": "550e8400-e29b-41d4-a716-446655440000"}
```

### Poll Task Progress

```python
import requests
import time

task_id = "550e8400-e29b-41d4-a716-446655440000"

while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    
    print(f"Status: {progress['status']}")
    print(f"Current step: {progress.get('current_step', 'N/A')}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(2)
```

**Response structure:**
```json
{
  "task_id": "...",
  "status": "running",
  "current_step": "research",
  "substeps": [
    {"name": "Planning", "status": "completed"},
    {"name": "Research", "status": "running"},
    {"name": "Writing", "status": "pending"}
  ],
  "progress_percentage": 45
}
```

### Get Final Report

```python
import requests

task_id = "550e8400-e29b-41d4-a716-446655440000"
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()

if result["status"] == "completed":
    print(result["result"])  # Full research report
else:
    print(f"Error: {result.get('error')}")
```

## Using Research Tools

### Tavily Search Tool

```python
# In src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """Search the web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        raise ValueError("TAVILY_API_KEY not set")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "search_depth": "advanced",
            "max_results": max_results
        }
    )
    response.raise_for_status()
    return response.json().get("results", [])
```

### arXiv Search Tool

```python
import requests
import xml.etree.ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv papers."""
    url = "http://export.arxiv.org/api/query"
    params = {
        "search_query": f"all:{query}",
        "start": 0,
        "max_results": max_results
    }
    
    response = requests.get(url, params=params)
    response.raise_for_status()
    
    root = ET.fromstring(response.content)
    papers = []
    
    for entry in root.findall("{http://www.w3.org/2005/Atom}entry"):
        papers.append({
            "title": entry.find("{http://www.w3.org/2005/Atom}title").text,
            "summary": entry.find("{http://www.w3.org/2005/Atom}summary").text,
            "authors": [author.find("{http://www.w3.org/2005/Atom}name").text 
                       for author in entry.findall("{http://www.w3.org/2005/Atom}author")]
        })
    
    return papers
```

### Wikipedia Search Tool

```python
import wikipedia

def wikipedia_search_tool(query: str, sentences: int = 5) -> str:
    """Search Wikipedia and return summary."""
    try:
        return wikipedia.summary(query, sentences=sentences)
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option summary
        return wikipedia.summary(e.options[0], sentences=sentences)
    except wikipedia.exceptions.PageError:
        return f"No Wikipedia page found for: {query}"
```

## Creating Agents

### Planning Agent Pattern

```python
# In src/planning_agent.py
import aisuite as ai

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> dict:
    """Generate research plan with subtasks."""
    client = ai.Client()
    
    system_prompt = """You are a research planning assistant. 
    Break down the research question into 3-5 concrete subtasks.
    Return a JSON object with 'subtasks' array."""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    import json
    plan = json.loads(response.choices[0].message.content)
    return plan
```

### Research Agent with Tools

```python
# In src/agents.py
from src.research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(subtask: str, model: str = "openai:gpt-4o") -> str:
    """Execute research using available tools."""
    client = ai.Client()
    
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
                "description": "Search academic papers on arXiv",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string", "description": "Paper topic"}
                    },
                    "required": ["query"]
                }
            }
        }
    ]
    
    messages = [
        {"role": "system", "content": "You are a research assistant with web and academic search tools."},
        {"role": "user", "content": subtask}
    ]
    
    # Tool-calling loop
    while True:
        response = client.chat.completions.create(
            model=model,
            messages=messages,
            tools=tools
        )
        
        message = response.choices[0].message
        
        if not message.tool_calls:
            return message.content
        
        # Execute tool calls
        for tool_call in message.tool_calls:
            function_name = tool_call.function.name
            arguments = json.loads(tool_call.function.arguments)
            
            if function_name == "tavily_search":
                result = tavily_search_tool(arguments["query"])
            elif function_name == "arxiv_search":
                result = arxiv_search_tool(arguments["query"])
            
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": json.dumps(result)
            })
```

## Database Models

```python
# In main.py
from sqlalchemy import Column, String, Text, DateTime, JSON
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.dialects.postgresql import UUID
import uuid

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    prompt = Column(Text, nullable=False)
    model = Column(String(100), nullable=False)
    status = Column(String(50), default="pending")
    current_step = Column(String(100))
    result = Column(Text)
    error = Column(Text)
    created_at = Column(DateTime, server_default="now()")
    updated_at = Column(DateTime, server_default="now()", onupdate="now()")

class Substep(Base):
    __tablename__ = "substeps"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    task_id = Column(UUID(as_uuid=True), nullable=False)
    name = Column(String(200), nullable=False)
    status = Column(String(50), default="pending")
    result = Column(Text)
    order = Column(Integer)
```

## FastAPI Endpoint Implementation

```python
# In main.py
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import threading

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """Start research workflow in background."""
    task_id = str(uuid.uuid4())
    
    # Create task in database
    with SessionLocal() as session:
        task = Task(
            id=task_id,
            prompt=request.prompt,
            model=request.model,
            status="running"
        )
        session.add(task)
        session.commit()
    
    # Run workflow in thread
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.daemon = True
    thread.start()
    
    return {"task_id": task_id}

def execute_research_workflow(task_id: str, prompt: str, model: str):
    """Execute multi-step research workflow."""
    try:
        # Step 1: Planning
        update_task_status(task_id, "running", "planning")
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute substasks
        results = []
        for idx, subtask in enumerate(plan["subtasks"]):
            update_task_status(task_id, "running", f"executing_{idx}")
            result = research_agent(subtask, model)
            results.append(result)
        
        # Step 3: Write report
        update_task_status(task_id, "running", "writing")
        report = writer_agent(prompt, results, model)
        
        # Step 4: Edit
        update_task_status(task_id, "running", "editing")
        final_report = editor_agent(report, model)
        
        # Complete
        update_task_result(task_id, "completed", final_report)
        
    except Exception as e:
        update_task_error(task_id, str(e))
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Optional (defaults shown)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb
```

### Database Connection

```python
# In main.py
import os
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = os.getenv("DATABASE_URL")
if not DATABASE_URL:
    raise ValueError("DATABASE_URL environment variable not set")

engine = create_engine(DATABASE_URL, pool_pre_ping=True)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Common Patterns

### Workflow Executor Pattern

```python
def executor_agent_step(task_id: str, step_name: str, agent_func, *args):
    """Execute a single agent step with error handling."""
    with SessionLocal() as session:
        task = session.query(Task).filter_by(id=task_id).first()
        task.current_step = step_name
        session.commit()
    
    try:
        result = agent_func(*args)
        return result
    except Exception as e:
        with SessionLocal() as session:
            task = session.query(Task).filter_by(id=task_id).first()
            task.status = "failed"
            task.error = str(e)
            session.commit()
        raise
```

### Progress Tracking

```python
@app.get("/task_progress/{task_id}")
async def get_task_progress(task_id: str):
    """Get real-time task progress."""
    with SessionLocal() as session:
        task = session.query(Task).filter_by(id=task_id).first()
        if not task:
            return {"error": "Task not found"}
        
        substeps = session.query(Substep).filter_by(
            task_id=task_id
        ).order_by(Substep.order).all()
        
        completed = sum(1 for s in substeps if s.status == "completed")
        total = len(substeps)
        progress = (completed / total * 100) if total > 0 else 0
        
        return {
            "task_id": str(task.id),
            "status": task.status,
            "current_step": task.current_step,
            "progress_percentage": progress,
            "substeps": [
                {"name": s.name, "status": s.status}
                for s in substeps
            ]
        }
```

## Troubleshooting

### Container Won't Start

**Symptom:** Postgres password errors or connection failures

**Solution:** Ensure entrypoint uses UNIX socket authentication:
```bash
# In docker/entrypoint.sh
su -s /bin/bash postgres -c "psql -c \"CREATE ROLE ${POSTGRES_USER} LOGIN PASSWORD '${POSTGRES_PASSWORD}';\""
```

### API Key Errors

**Symptom:** `TAVILY_API_KEY not set` or authentication failures

**Solution:** 
```bash
# Verify .env file exists and is loaded
docker run --env-file .env ...

# Or pass explicitly
docker run -e OPENAI_API_KEY=sk-... -e TAVILY_API_KEY=tvly-...
```

### Tables Not Persisting

**Symptom:** Database resets on container restart

**Solution:** Comment out drop tables in production:
```python
# In main.py - remove or guard this:
# Base.metadata.drop_all(bind=engine)
Base.metadata.create_all(bind=engine)
```

### Template Not Found

**Symptom:** 404 errors or Jinja2 template errors

**Solution:** Ensure Dockerfile copies templates:
```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

### Connect to Database from Host

```bash
# Access Postgres directly
psql "postgresql://app:local@localhost:5432/appdb"

# Inspect running container
docker exec -it fpsvc bash
psql -U app -d appdb
```

### Enable Hot Reload for Development

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -c "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Wikipedia Rate Limiting

**Symptom:** `HTTPError 429` from Wikipedia API

**Solution:** Add retry logic with backoff:
```python
import time

def wikipedia_search_tool(query: str, sentences: int = 5, max_retries: int = 3) -> str:
    for attempt in range(max_retries):
        try:
            return wikipedia.summary(query, sentences=sentences)
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429 and attempt < max_retries - 1:
                time.sleep(2 ** attempt)
            else:
                raise
```

## Testing the API

```bash
# Start task
TASK_ID=$(curl -s -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications in drug discovery", "model":"openai:gpt-4o"}' \
  | jq -r '.task_id')

echo "Task ID: $TASK_ID"

# Monitor progress
watch -n 2 "curl -s http://localhost:8000/task_progress/$TASK_ID | jq"

# Get final result
curl -s http://localhost:8000/task_status/$TASK_ID | jq -r '.result'
```
