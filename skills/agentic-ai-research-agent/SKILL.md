---
name: agentic-ai-research-agent
description: FastAPI research agent service with multi-step agentic workflows, Tavily/arXiv/Wikipedia tools, and Postgres task tracking
triggers:
  - set up the agentic AI research agent service
  - build a research agent with planning and execution
  - implement multi-step research workflow with tool use
  - create agentic workflow with Tavily arXiv Wikipedia
  - deploy FastAPI research agent with Postgres
  - use the reflective research agent for report generation
  - integrate planning agent with research tools
  - set up agentic research workflow service
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI-based service that orchestrates multi-step research workflows using planning agents and specialized tool-using agents. It implements a reflective agentic pattern with:

- **Planning Agent**: Decomposes research tasks into structured steps
- **Executor Agents**: Research, writer, and editor agents that use external tools
- **Research Tools**: Tavily search, arXiv papers, Wikipedia integration
- **Task Tracking**: Postgres database for persistent state management
- **Live Progress**: Real-time step/substep status updates

The service runs Postgres + API in a single Docker container for simplified deployment.

## Installation

### Docker Setup (Recommended)

1. **Clone and prepare environment:**

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

### Manual Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres separately
export DATABASE_URL="postgresql://user:pass@localhost:5432/appdb"

# Run the service
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Project Structure

```
src/
├── planning_agent.py         # planner_agent(), executor_agent_step()
├── agents.py                 # research_agent, writer_agent, editor_agent
└── research_tools.py         # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool

main.py                       # FastAPI app with endpoints
templates/index.html          # Web UI
docker/entrypoint.sh          # Container startup script
```

## API Reference

### Start Research Task

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'

# Response: {"task_id": "uuid-here"}
```

### Check Task Progress (Live Updates)

```bash
curl http://localhost:8000/task_progress/<TASK_ID>
```

Response structure:
```json
{
  "task_id": "uuid",
  "status": "running",
  "current_step": 2,
  "total_steps": 5,
  "steps": [
    {
      "step_number": 1,
      "name": "Planning",
      "status": "completed",
      "result": "..."
    },
    {
      "step_number": 2,
      "name": "Research",
      "status": "in_progress",
      "substeps": [...]
    }
  ]
}
```

### Get Final Report

```bash
curl http://localhost:8000/task_status/<TASK_ID>
```

Response:
```json
{
  "task_id": "uuid",
  "status": "completed",
  "report": "# Research Report\n\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:00Z"
}
```

## Core Patterns

### 1. Planning Agent Pattern

```python
# src/planning_agent.py
from aisuite import Client

def planner_agent(user_prompt: str, model: str) -> dict:
    """
    Takes a research prompt and returns a structured plan
    with steps and agent assignments.
    """
    client = Client()
    
    system_prompt = """You are a research planning agent.
    Break down the user's research request into concrete steps.
    Return JSON: {"steps": [{"agent": "research|writer|editor", "task": "..."}]}
    """
    
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return parse_plan(response.choices[0].message.content)
```

### 2. Tool-Using Research Agent

```python
# src/agents.py
from src.research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(task: str, context: dict = None) -> str:
    """
    Research agent that uses multiple tools to gather information.
    """
    client = Client()
    
    # Available tools
    tools = [
        {
            "type": "function",
            "function": {
                "name": "tavily_search",
                "description": "Search the web for current information",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string"}
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
                        "query": {"type": "string"},
                        "max_results": {"type": "integer", "default": 5}
                    },
                    "required": ["query"]
                }
            }
        }
    ]
    
    messages = [
        {"role": "system", "content": "You are a research agent. Use tools to gather information."},
        {"role": "user", "content": task}
    ]
    
    # Tool execution loop
    while True:
        response = client.chat.completions.create(
            model="openai:gpt-4o",
            messages=messages,
            tools=tools
        )
        
        message = response.choices[0].message
        
        if not message.tool_calls:
            return message.content
        
        # Execute tool calls
        for tool_call in message.tool_calls:
            if tool_call.function.name == "tavily_search":
                result = tavily_search_tool(tool_call.function.arguments["query"])
            elif tool_call.function.name == "arxiv_search":
                result = arxiv_search_tool(
                    tool_call.function.arguments["query"],
                    tool_call.function.arguments.get("max_results", 5)
                )
            
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": str(result)
            })
```

### 3. Research Tools Implementation

```python
# src/research_tools.py
import os
import requests
import arxiv
import wikipedia

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search using Tavily API.
    """
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return [{"error": "TAVILY_API_KEY not set"}]
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results
    }
    
    try:
        response = requests.post(url, json=payload, timeout=10)
        response.raise_for_status()
        data = response.json()
        return data.get("results", [])
    except Exception as e:
        return [{"error": str(e)}]


def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search arXiv for academic papers.
    """
    try:
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
                "pdf_url": paper.pdf_url,
                "published": str(paper.published)
            })
        return results
    except Exception as e:
        return [{"error": str(e)}]


def wikipedia_search_tool(query: str, sentences: int = 3) -> dict:
    """
    Search Wikipedia and return summary.
    """
    try:
        summary = wikipedia.summary(query, sentences=sentences)
        page = wikipedia.page(query)
        return {
            "title": page.title,
            "summary": summary,
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": "Disambiguation", "options": e.options[:5]}
    except wikipedia.exceptions.PageError:
        return {"error": "Page not found"}
    except Exception as e:
        return {"error": str(e)}
```

### 4. FastAPI Task Management

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
import uuid
from datetime import datetime

app = FastAPI()

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()


class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text)
    model = Column(String)
    status = Column(String, default="pending")
    plan = Column(Text, nullable=True)
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)


Base.metadata.create_all(bind=engine)


def execute_research_workflow(task_id: str, prompt: str, model: str):
    """
    Background task that executes the full research workflow.
    """
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    
    try:
        # Step 1: Planning
        task.status = "planning"
        db.commit()
        
        plan = planner_agent(prompt, model)
        task.plan = str(plan)
        db.commit()
        
        # Step 2: Execute each step
        task.status = "executing"
        results = []
        
        for step in plan["steps"]:
            if step["agent"] == "research":
                result = research_agent(step["task"])
            elif step["agent"] == "writer":
                result = writer_agent(step["task"], context=results)
            elif step["agent"] == "editor":
                result = editor_agent(step["task"], context=results)
            
            results.append(result)
        
        # Step 3: Compile report
        task.status = "completed"
        task.report = results[-1]  # Final output from editor
        task.completed_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()
    finally:
        db.close()


@app.post("/generate_report")
async def generate_report(
    request: dict,
    background_tasks: BackgroundTasks
):
    """
    Start a research task in the background.
    """
    task_id = str(uuid.uuid4())
    prompt = request.get("prompt")
    model = request.get("model", "openai:gpt-4o")
    
    db = SessionLocal()
    task = Task(id=task_id, prompt=prompt, model=model)
    db.add(task)
    db.commit()
    db.close()
    
    background_tasks.add_task(
        execute_research_workflow,
        task_id,
        prompt,
        model
    )
    
    return {"task_id": task_id}


@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """
    Get final task status and report.
    """
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.id,
        "status": task.status,
        "report": task.report,
        "created_at": task.created_at,
        "completed_at": task.completed_at
    }
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key
TAVILY_API_KEY=tvly-...            # Tavily search API key

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional overrides
POSTGRES_USER=app                  # Default: app
POSTGRES_PASSWORD=local            # Default: local
POSTGRES_DB=appdb                  # Default: appdb

# Development
RESET_DB_ON_STARTUP=0              # Set to 1 to drop tables on startup
```

### Model Configuration

The service supports any model compatible with `aisuite.Client()`:

```python
# OpenAI models
"openai:gpt-4o"
"openai:gpt-4-turbo"
"openai:gpt-3.5-turbo"

# Anthropic models (if configured)
"anthropic:claude-3-opus"
"anthropic:claude-3-sonnet"
```

## Common Workflows

### Basic Research Report

```python
import requests

# Start task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Analyze recent advances in quantum computing",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]

# Poll for completion
import time
while True:
    status = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
    if status["status"] in ["completed", "failed"]:
        print(status["report"])
        break
    time.sleep(2)
```

### Custom Agent Integration

```python
# Add a new agent to src/agents.py
def critic_agent(draft: str, criteria: list) -> str:
    """
    Critic agent that evaluates draft against criteria.
    """
    client = Client()
    
    prompt = f"""Review this draft against criteria: {criteria}
    
    Draft:
    {draft}
    
    Provide constructive feedback and suggest improvements.
    """
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=[
            {"role": "system", "content": "You are a critic agent."},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content
```

### Database Direct Access

```bash
# Connect to Postgres from host
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT id, status, created_at FROM tasks ORDER BY created_at DESC LIMIT 10;

# View task details
SELECT prompt, report FROM tasks WHERE id = 'uuid-here';
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs fpsvc

# Verify Postgres is running inside container
docker exec -it fpsvc bash -lc "pg_isready"

# Test database connection
docker exec -it fpsvc bash -lc "psql \$DATABASE_URL -c 'SELECT 1'"
```

### API Keys Not Working

```bash
# Verify environment variables are set
docker exec -it fpsvc env | grep API_KEY

# Restart with explicit environment
docker run --rm -it \
  -p 8000:8000 \
  -e OPENAI_API_KEY=sk-... \
  -e TAVILY_API_KEY=tvly-... \
  fastapi-postgres-service
```

### Template Not Found

```bash
# Verify templates directory exists in container
docker exec -it fpsvc ls -la /app/templates

# Rebuild if missing
docker build --no-cache -t fastapi-postgres-service .
```

### Tool Errors

```python
# Tavily rate limit - add retry logic
def tavily_search_tool(query: str, max_results: int = 5, retries: int = 3):
    for attempt in range(retries):
        try:
            response = requests.post(url, json=payload, timeout=10)
            response.raise_for_status()
            return response.json().get("results", [])
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429 and attempt < retries - 1:
                time.sleep(2 ** attempt)
                continue
            return [{"error": str(e)}]

# Wikipedia disambiguation
def wikipedia_search_tool(query: str, sentences: int = 3):
    try:
        return wikipedia.summary(query, sentences=sentences, auto_suggest=False)
    except wikipedia.exceptions.DisambiguationError as e:
        # Try first option
        return wikipedia.summary(e.options[0], sentences=sentences)
```

### Development Mode with Hot Reload

```bash
# Mount local code and enable reload
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  fastapi-postgres-service \
  bash -lc "service postgresql start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Reset Database

```python
# In main.py, add flag-controlled reset
if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)
```
