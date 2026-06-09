---
name: deeplearning-ai-agentic-research
description: FastAPI research agent service with multi-step planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres task tracking
triggers:
  - set up the agentic research workflow service
  - create a research agent with planning and execution
  - implement multi-step research using Tavily and arXiv
  - build a reflective research agent with FastAPI
  - configure the research agent planner and executor
  - use the agentic AI research service
  - integrate research tools into agent workflow
  - deploy the research agent with Postgres
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that plans and executes multi-step research workflows using tool-calling agents (Tavily search, arXiv papers, Wikipedia). Includes a planner agent that breaks down tasks, executor agents (research/writer/editor), and Postgres for task state tracking.

## What It Does

- **Planning Agent**: Breaks research questions into structured subtasks
- **Research Agents**: Execute searches using Tavily (web), arXiv (papers), Wikipedia (encyclopedia)
- **Writer & Editor Agents**: Transform research into cohesive reports
- **Task Tracking**: Stores progress and results in Postgres
- **Web UI**: Simple interface to kick off research tasks and view progress
- **REST API**: Programmatic access to agent workflows

## Installation

### Prerequisites

Create a `.env` file with required API keys:

```bash
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Docker (Recommended)

```bash
# Build the image
docker build -t fastapi-postgres-service .

# Run (includes Postgres + API in one container)
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

# Set up Postgres separately and configure DATABASE_URL
export DATABASE_URL="postgresql://user:password@localhost:5432/appdb"

# Run the service
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                      # FastAPI app entry point
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # UI page
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── .env                         # API keys (not committed)
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

### Poll Task Progress

```python
import requests
import time

def poll_progress(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = response.json()
        
        print(f"Status: {data['status']}")
        print(f"Current step: {data.get('current_step', 'N/A')}")
        
        if data["status"] in ["completed", "failed"]:
            break
        
        time.sleep(2)

poll_progress(task_id)
```

### Get Final Report

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result["status"] == "completed":
    print("Report:", result["report"])
    print("Metadata:", result["metadata"])
```

## Creating Custom Research Agents

### Basic Research Agent

```python
# src/agents.py
import aisuite as ai

def research_agent(query: str, tools: list) -> dict:
    """Execute research using available tools."""
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a research assistant. Use the provided tools to gather information."
        },
        {
            "role": "user",
            "content": f"Research this topic: {query}"
        }
    ]
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages,
        tools=tools
    )
    
    # Process tool calls
    if hasattr(response.choices[0].message, "tool_calls"):
        tool_calls = response.choices[0].message.tool_calls
        # Execute tools and gather results
        results = []
        for call in tool_calls:
            tool_name = call.function.name
            tool_args = json.loads(call.function.arguments)
            # Execute tool...
            results.append(execute_tool(tool_name, tool_args))
        
        return {"findings": results}
    
    return {"answer": response.choices[0].message.content}
```

### Planning Agent

```python
# src/planning_agent.py
import aisuite as ai
import json

def planner_agent(prompt: str) -> list:
    """Break down research task into subtasks."""
    client = ai.Client()
    
    system_prompt = """You are a research planner. Break down the user's research 
    question into 3-5 specific subtasks. Return JSON array of tasks with:
    - task: brief description
    - agent: which agent to use (research/writer/editor)
    - dependencies: list of prior task indices needed"""
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt}
    ]
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages,
        response_format={"type": "json_object"}
    )
    
    plan = json.loads(response.choices[0].message.content)
    return plan["tasks"]
```

### Executor Agent

```python
# src/planning_agent.py
def executor_agent_step(task: dict, context: dict, tools: list) -> dict:
    """Execute a single planned task."""
    agent_map = {
        "research": research_agent,
        "writer": writer_agent,
        "editor": editor_agent
    }
    
    agent_fn = agent_map.get(task["agent"], research_agent)
    
    # Build context from dependencies
    dep_context = ""
    for dep_idx in task.get("dependencies", []):
        if dep_idx in context:
            dep_context += f"\n{context[dep_idx]}"
    
    prompt = f"{task['task']}\n\nContext:{dep_context}"
    
    return agent_fn(prompt, tools)
```

## Research Tools Integration

### Tavily Web Search

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """Search the web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results
        }
    )
    
    data = response.json()
    return data.get("results", [])

# Tool definition for agent
tavily_tool_def = {
    "type": "function",
    "function": {
        "name": "tavily_search",
        "description": "Search the web for current information",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"},
                "max_results": {"type": "integer", "default": 5}
            },
            "required": ["query"]
        }
    }
}
```

### arXiv Paper Search

```python
# src/research_tools.py
import urllib.parse
import feedparser

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """Search arXiv for academic papers."""
    encoded_query = urllib.parse.quote(query)
    url = f"http://export.arxiv.org/api/query?search_query=all:{encoded_query}&max_results={max_results}"
    
    feed = feedparser.parse(url)
    
    results = []
    for entry in feed.entries:
        results.append({
            "title": entry.title,
            "summary": entry.summary,
            "authors": [author.name for author in entry.authors],
            "link": entry.link,
            "published": entry.published
        })
    
    return results

arxiv_tool_def = {
    "type": "function",
    "function": {
        "name": "arxiv_search",
        "description": "Search arXiv for academic papers",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {"type": "string"},
                "max_results": {"type": "integer", "default": 5}
            },
            "required": ["query"]
        }
    }
}
```

### Wikipedia Search

```python
# src/research_tools.py
import wikipedia

def wikipedia_search_tool(query: str) -> dict:
    """Search Wikipedia for encyclopedic information."""
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=3),
            "url": page.url,
            "content": page.content[:2000]  # First 2000 chars
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Return first option
        return wikipedia_search_tool(e.options[0])
    except wikipedia.exceptions.PageError:
        return {"error": "Page not found"}

wikipedia_tool_def = {
    "type": "function",
    "function": {
        "name": "wikipedia_search",
        "description": "Search Wikipedia for encyclopedic information",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {"type": "string"}
            },
            "required": ["query"]
        }
    }
}
```

## Database Models

```python
# main.py
from sqlalchemy import Column, String, Text, DateTime, JSON
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
import os
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="planning")  # planning, executing, completed, failed
    current_step = Column(String, nullable=True)
    plan = Column(JSON, nullable=True)
    report = Column(Text, nullable=True)
    metadata = Column(JSON, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Initialize database
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base.metadata.create_all(bind=engine)
```

## FastAPI Endpoints

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates
from pydantic import BaseModel
import uuid
import threading

app = FastAPI()
templates = Jinja2Templates(directory="templates")

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(request: ResearchRequest, background_tasks: BackgroundTasks):
    """Start a new research task."""
    task_id = str(uuid.uuid4())
    
    # Create task in database
    db = SessionLocal()
    task = Task(
        task_id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="planning"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Start background processing
    thread = threading.Thread(
        target=execute_research_workflow,
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
        "task_id": task_id,
        "status": task.status,
        "current_step": task.current_step,
        "plan": task.plan
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
        "task_id": task_id,
        "status": task.status,
        "report": task.report,
        "metadata": task.metadata
    }
```

## Complete Workflow Example

```python
def execute_research_workflow(task_id: str, prompt: str, model: str):
    """Execute the full research workflow."""
    db = SessionLocal()
    
    try:
        # Step 1: Planning
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "planning"
        task.current_step = "Creating research plan"
        db.commit()
        
        plan = planner_agent(prompt)
        task.plan = plan
        db.commit()
        
        # Step 2: Execute each subtask
        task.status = "executing"
        context = {}
        all_tools = [tavily_tool_def, arxiv_tool_def, wikipedia_tool_def]
        
        for idx, subtask in enumerate(plan):
            task.current_step = f"Step {idx+1}/{len(plan)}: {subtask['task']}"
            db.commit()
            
            result = executor_agent_step(subtask, context, all_tools)
            context[idx] = result
        
        # Step 3: Generate final report
        task.status = "finalizing"
        task.current_step = "Generating final report"
        db.commit()
        
        final_report = writer_agent(prompt, context, all_tools)
        
        # Step 4: Complete
        task.status = "completed"
        task.report = final_report
        task.metadata = {"steps_completed": len(plan)}
        task.current_step = None
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.metadata = {"error": str(e)}
        db.commit()
    
    finally:
        db.close()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...                    # OpenAI API key
TAVILY_API_KEY=tvly-...                  # Tavily search API key

# Database (set automatically by Docker entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional Postgres config (for custom setup)
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional: Reset DB on startup (dev only)
RESET_DB_ON_STARTUP=0
```

### Docker Compose Alternative

```yaml
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
      - postgres-data:/var/lib/postgresql/17/main

volumes:
  postgres-data:
```

## Common Patterns

### Custom Writer Agent

```python
def writer_agent(topic: str, research_data: dict, tools: list) -> str:
    """Transform research into a cohesive report."""
    client = ai.Client()
    
    # Combine all research findings
    findings = "\n\n".join([str(v) for v in research_data.values()])
    
    messages = [
        {
            "role": "system",
            "content": "You are a technical writer. Create a comprehensive, well-structured report."
        },
        {
            "role": "user",
            "content": f"Topic: {topic}\n\nFindings:\n{findings}\n\nWrite a detailed report."
        }
    ]
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    return response.choices[0].message.content
```

### Reflection/Editing Loop

```python
def editor_agent(draft: str, criteria: list) -> str:
    """Review and improve a draft report."""
    client = ai.Client()
    
    criteria_text = "\n".join([f"- {c}" for c in criteria])
    
    messages = [
        {
            "role": "system",
            "content": f"You are an editor. Improve the draft based on:\n{criteria_text}"
        },
        {
            "role": "user",
            "content": f"Draft:\n{draft}\n\nProvide an improved version."
        }
    ]
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=messages
    )
    
    return response.choices[0].message.content
```

## Troubleshooting

### Database Connection Issues

```python
# Verify connection
from sqlalchemy import text

def test_db_connection():
    engine = create_engine(os.getenv("DATABASE_URL"))
    with engine.connect() as conn:
        result = conn.execute(text("SELECT 1"))
        print("Database connected:", result.fetchone())

test_db_connection()
```

### API Key Errors

```bash
# Verify environment variables are loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Check .env file is mounted correctly
docker exec -it fpsvc bash -lc "cat /app/.env"
```

### Task Stuck in "Planning"

```python
# Check logs in database
db = SessionLocal()
task = db.query(Task).filter(Task.task_id == task_id).first()
print(f"Status: {task.status}")
print(f"Metadata: {task.metadata}")
db.close()

# Check container logs
# docker logs -f fpsvc
```

### Tavily Rate Limits

```python
def tavily_search_tool(query: str, max_results: int = 5):
    """Search with retry logic."""
    import time
    
    for attempt in range(3):
        try:
            # ... existing code ...
            return data.get("results", [])
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429:
                time.sleep(2 ** attempt)  # Exponential backoff
            else:
                raise
    
    return []
```

### Database Tables Not Created

```python
# Force table creation
from main import Base, engine

Base.metadata.drop_all(bind=engine)  # Careful: deletes data
Base.metadata.create_all(bind=engine)
```

### Connect to Postgres from Host

```bash
# Install psql client, then:
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT task_id, status, current_step FROM tasks;
```
