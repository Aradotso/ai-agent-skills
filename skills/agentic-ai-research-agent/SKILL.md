---
name: agentic-ai-research-agent
description: FastAPI research agent service with planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres task tracking
triggers:
  - build a research agent with planning and tools
  - create agentic workflow for research tasks
  - set up FastAPI research agent with Postgres
  - implement multi-step agent research workflow
  - use Tavily arXiv and Wikipedia agents
  - create reflective research agent service
  - build task-based research agent API
  - implement planning and executor agents
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI-based service that orchestrates multi-step research workflows using planning and executor agents. It integrates research tools (Tavily search, arXiv papers, Wikipedia) with LLMs, stores task state in Postgres, and provides real-time progress tracking. The system uses a planner agent to decompose research tasks into subtasks, then executes them using specialized agents (research, writer, editor).

## Installation

### Docker (Recommended)

1. **Clone the repository:**
```bash
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file:**
```bash
cat > .env << EOF
OPENAI_API_KEY=your-openai-key-here
TAVILY_API_KEY=your-tavily-key-here
EOF
```

3. **Build the Docker image:**
```bash
docker build -t fastapi-postgres-service .
```

4. **Run the container:**
```bash
docker run --rm -it -p 8000:8000 -p 5432:5432 --name fpsvc --env-file .env fastapi-postgres-service
```

### Local Development

```bash
pip install -r requirements.txt
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="your-key"
export TAVILY_API_KEY="your-key"
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                    # FastAPI application entry point
├── src/
│   ├── planning_agent.py      # Planner and executor agent logic
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
└── Dockerfile
```

## Core Concepts

### 1. Task Lifecycle

Tasks flow through states: `pending` → `in_progress` → `completed` / `failed`

Each task has:
- Unique `task_id` (UUID)
- Research prompt
- Model selection (e.g., `openai:gpt-4o`)
- Progress tracking with substeps
- Final report output

### 2. Agent Workflow

```
User Prompt → Planner Agent → [Subtasks] → Executor Agent → Research/Writer/Editor → Final Report
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
```

**cURL:**
```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications", "model": "openai:gpt-4o"}'
```

### Check Task Progress

```python
import requests
import time

task_id = "uuid-from-previous-call"

while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    print(f"Status: {progress['status']}, Step: {progress['current_step']}")
    
    if progress['status'] in ['completed', 'failed']:
        break
    
    time.sleep(2)
```

### Get Final Report

```python
status = requests.get(f"http://localhost:8000/task_status/{task_id}").json()

if status['status'] == 'completed':
    print(status['report'])
else:
    print(f"Error: {status.get('error')}")
```

## Database Models

### Task Table Schema

```python
from sqlalchemy import Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
import uuid
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, in_progress, completed, failed
    current_step = Column(String, default="initialization")
    current_substep = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

## Building Custom Agents

### Planning Agent

```python
import os
import aisuite as ai

client = ai.Client()

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> dict:
    """
    Creates a research plan with subtasks.
    
    Returns:
        {
            "research_question": str,
            "subtasks": [
                {"id": "1", "description": "Search academic papers", "agent": "research"},
                {"id": "2", "description": "Synthesize findings", "agent": "writer"}
            ]
        }
    """
    messages = [
        {
            "role": "system",
            "content": """You are a research planning agent. Break down research questions 
            into actionable subtasks. Assign each to: research_agent, writer_agent, or editor_agent.
            Return JSON with research_question and subtasks array."""
        },
        {"role": "user", "content": prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    import json
    return json.loads(response.choices[0].message.content)
```

### Research Agent with Tools

```python
from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

def research_agent(query: str, model: str = "openai:gpt-4o") -> str:
    """
    Performs research using available tools.
    """
    # Determine which tool to use
    tool_prompt = f"""
    For the query: "{query}"
    
    Choose the best research tool:
    1. tavily - general web search, recent news
    2. arxiv - academic papers, scientific research
    3. wikipedia - encyclopedic background information
    
    Respond with just the tool name.
    """
    
    messages = [{"role": "user", "content": tool_prompt}]
    response = client.chat.completions.create(model=model, messages=messages)
    tool_choice = response.choices[0].message.content.strip().lower()
    
    # Execute tool
    if "tavily" in tool_choice:
        results = tavily_search_tool(query)
    elif "arxiv" in tool_choice:
        results = arxiv_search_tool(query)
    else:
        results = wikipedia_search_tool(query)
    
    # Synthesize results
    synthesis_prompt = f"""
    Query: {query}
    
    Research Results:
    {results}
    
    Provide a comprehensive summary of the findings.
    """
    
    messages = [{"role": "user", "content": synthesis_prompt}]
    response = client.chat.completions.create(model=model, messages=messages)
    
    return response.choices[0].message.content
```

### Research Tools Implementation

```python
# src/research_tools.py
import os
import requests
import arxiv
import wikipedia

def tavily_search_tool(query: str, max_results: int = 5) -> str:
    """Search using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    
    results = response.json().get("results", [])
    return "\n\n".join([
        f"Title: {r['title']}\nURL: {r['url']}\nContent: {r['content']}"
        for r in results
    ])

def arxiv_search_tool(query: str, max_results: int = 5) -> str:
    """Search arXiv papers."""
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for paper in search.results():
        results.append(
            f"Title: {paper.title}\n"
            f"Authors: {', '.join([a.name for a in paper.authors])}\n"
            f"Summary: {paper.summary}\n"
            f"URL: {paper.entry_id}"
        )
    
    return "\n\n".join(results)

def wikipedia_search_tool(query: str) -> str:
    """Search Wikipedia."""
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return f"Title: {page.title}\n\nSummary:\n{page.summary}"
    except wikipedia.exceptions.DisambiguationError as e:
        # Take first option
        page = wikipedia.page(e.options[0])
        return f"Title: {page.title}\n\nSummary:\n{page.summary}"
    except Exception as e:
        return f"Wikipedia search failed: {str(e)}"
```

## Executor Agent Pattern

```python
from sqlalchemy.orm import Session
from datetime import datetime

def executor_agent_step(db: Session, task_id: str, subtask: dict, model: str):
    """
    Execute a single subtask and update task progress.
    """
    task = db.query(Task).filter(Task.id == task_id).first()
    
    # Update progress
    task.current_substep = subtask["description"]
    task.updated_at = datetime.utcnow()
    db.commit()
    
    # Route to appropriate agent
    agent_type = subtask.get("agent", "research")
    query = subtask["description"]
    
    if agent_type == "research":
        result = research_agent(query, model)
    elif agent_type == "writer":
        result = writer_agent(query, model)
    elif agent_type == "editor":
        result = editor_agent(query, model)
    else:
        result = f"Unknown agent type: {agent_type}"
    
    return result
```

## Complete Workflow Example

```python
import threading
from sqlalchemy.orm import Session

def research_workflow(task_id: str, prompt: str, model: str, db: Session):
    """
    Full research workflow: plan → execute → report.
    """
    try:
        task = db.query(Task).filter(Task.id == task_id).first()
        
        # Step 1: Planning
        task.status = "in_progress"
        task.current_step = "planning"
        db.commit()
        
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute subtasks
        task.current_step = "research"
        db.commit()
        
        results = []
        for subtask in plan["subtasks"]:
            result = executor_agent_step(db, task_id, subtask, model)
            results.append({"subtask": subtask["description"], "result": result})
        
        # Step 3: Generate final report
        task.current_step = "writing_report"
        db.commit()
        
        report_prompt = f"""
        Research Question: {plan['research_question']}
        
        Findings:
        {chr(10).join([f"{i+1}. {r['subtask']}: {r['result']}" for i, r in enumerate(results)])}
        
        Create a comprehensive research report.
        """
        
        messages = [{"role": "user", "content": report_prompt}]
        response = client.chat.completions.create(model=model, messages=messages)
        
        # Save final report
        task.status = "completed"
        task.current_step = "done"
        task.report = response.choices[0].message.content
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
        db.commit()

# Start workflow in background thread
def start_research_task(task_id: str, prompt: str, model: str, db: Session):
    thread = threading.Thread(
        target=research_workflow,
        args=(task_id, prompt, model, db)
    )
    thread.start()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Optional
DATABASE_URL=postgresql://user:pass@host:5432/db  # Postgres connection
POSTGRES_USER=app               # DB username (Docker)
POSTGRES_PASSWORD=local         # DB password (Docker)
POSTGRES_DB=appdb              # Database name (Docker)
RESET_DB_ON_STARTUP=0          # Set to 1 to drop/recreate tables
```

### Model Selection

Supported models via `aisuite`:
- `openai:gpt-4o`
- `openai:gpt-4-turbo`
- `openai:gpt-3.5-turbo`
- Other providers supported by aisuite

## Web UI Integration

The service includes a Jinja2 template for browser-based interaction:

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Research Agent</title>
</head>
<body>
    <h1>Agentic Research Agent</h1>
    <form id="researchForm">
        <label>Research Question:</label>
        <textarea name="prompt" rows="4" cols="50"></textarea>
        
        <label>Model:</label>
        <select name="model">
            <option value="openai:gpt-4o">GPT-4o</option>
            <option value="openai:gpt-4-turbo">GPT-4 Turbo</option>
        </select>
        
        <button type="submit">Start Research</button>
    </form>
    
    <div id="progress"></div>
    <div id="report"></div>
    
    <script>
        document.getElementById('researchForm').onsubmit = async (e) => {
            e.preventDefault();
            const formData = new FormData(e.target);
            const data = {
                prompt: formData.get('prompt'),
                model: formData.get('model')
            };
            
            const response = await fetch('/generate_report', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify(data)
            });
            
            const {task_id} = await response.json();
            pollProgress(task_id);
        };
        
        async function pollProgress(taskId) {
            const interval = setInterval(async () => {
                const res = await fetch(`/task_progress/${taskId}`);
                const progress = await res.json();
                
                document.getElementById('progress').innerHTML = 
                    `Status: ${progress.status}<br>Step: ${progress.current_step}`;
                
                if (progress.status === 'completed' || progress.status === 'failed') {
                    clearInterval(interval);
                    const finalRes = await fetch(`/task_status/${taskId}`);
                    const final = await finalRes.json();
                    document.getElementById('report').innerHTML = 
                        `<h2>Report:</h2><pre>${final.report || final.error}</pre>`;
                }
            }, 2000);
        }
    </script>
</body>
</html>
```

## Troubleshooting

### Database Connection Issues

```python
# Verify DATABASE_URL format
import os
print(os.getenv("DATABASE_URL"))
# Should be: postgresql://user:password@host:port/database

# Test connection
from sqlalchemy import create_engine
engine = create_engine(os.getenv("DATABASE_URL"))
with engine.connect() as conn:
    print("Connection successful")
```

### Tables Not Created

```python
# Force table creation
from main import Base, engine

Base.metadata.drop_all(bind=engine)  # Caution: deletes data
Base.metadata.create_all(bind=engine)
```

### Tool API Errors

```python
# Test Tavily
import os
import requests

response = requests.post(
    "https://api.tavily.com/search",
    json={
        "api_key": os.getenv("TAVILY_API_KEY"),
        "query": "test",
        "max_results": 1
    }
)
print(response.json())

# Test arXiv (no API key needed)
import arxiv
search = arxiv.Search(query="machine learning", max_results=1)
for result in search.results():
    print(result.title)
```

### Container Won't Start

```bash
# Check logs
docker logs fpsvc

# Verify Postgres is running
docker exec -it fpsvc bash -lc "pg_isready"

# Check if tables exist
docker exec -it fpsvc bash -lc "psql -U app -d appdb -c '\dt'"
```

### Missing Templates

```bash
# Verify templates are in container
docker exec -it fpsvc ls -la /app/templates

# If missing, rebuild with correct COPY in Dockerfile
# Dockerfile should have:
# COPY templates/ /app/templates/
# COPY static/ /app/static/
```

## Performance Optimization

### Background Task Management

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor(max_workers=4)

@app.post("/generate_report")
async def generate_report_async(request: ResearchRequest):
    task_id = str(uuid.uuid4())
    
    # Create task record
    db = SessionLocal()
    task = Task(id=task_id, prompt=request.prompt, model=request.model)
    db.add(task)
    db.commit()
    db.close()
    
    # Run in thread pool
    loop = asyncio.get_event_loop()
    loop.run_in_executor(executor, research_workflow, task_id, request.prompt, request.model, SessionLocal())
    
    return {"task_id": task_id}
```

### Caching Research Results

```python
import hashlib
import json

def cache_key(query: str) -> str:
    return hashlib.md5(query.encode()).hexdigest()

def cached_research(query: str, tool_func, ttl: int = 3600):
    """Cache research tool results."""
    from redis import Redis
    redis = Redis(host='localhost', port=6379, decode_responses=True)
    
    key = f"research:{cache_key(query)}"
    cached = redis.get(key)
    
    if cached:
        return cached
    
    result = tool_func(query)
    redis.setex(key, ttl, result)
    return result
```
