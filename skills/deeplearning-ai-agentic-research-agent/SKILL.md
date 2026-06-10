---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with multi-step planning, Tavily/arXiv/Wikipedia tools, and Postgres task tracking
triggers:
  - how do I use the agentic research agent
  - set up the DeepLearning.AI research workflow
  - create a research task with planning agent
  - query task progress in agentic AI service
  - integrate Tavily and arXiv search tools
  - build a reflective research agent with FastAPI
  - deploy the agentic workflow research service
  - track multi-step agent execution status
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The **DeepLearning.AI Agentic Research Agent** is a FastAPI-based service that orchestrates multi-step research workflows using AI agents. It features:

- **Planning Agent**: Breaks down research queries into executable steps
- **Research Tools**: Tavily search, arXiv papers, Wikipedia lookups
- **Agent Types**: Research agent, writer agent, editor agent
- **Task Tracking**: Postgres database stores task state, progress, and results
- **Live Progress**: Real-time status updates for each workflow step
- **Single-Container Deployment**: Postgres + FastAPI bundled for easy local development

The system uses a planner-executor pattern where a planning agent creates a workflow, and executor agents run each step with appropriate tools.

## Installation

### Prerequisites

```bash
# Ensure Docker is installed
docker --version

# Create .env file with API keys
cat > .env << EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF
```

### Build and Run

```bash
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

The container starts Postgres, creates the database, and launches the FastAPI app on http://localhost:8000.

### Verify Installation

```bash
# Check the API health
curl http://localhost:8000/

# View API documentation
open http://localhost:8000/docs
```

## Project Structure

```
agentic-ai-public/
├── main.py                      # FastAPI application entry point
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI for task submission
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt             # Python dependencies
├── Dockerfile
└── README.md
```

## Core Concepts

### Task Lifecycle

1. **Submit**: POST to `/generate_report` with a research prompt
2. **Plan**: Planning agent creates multi-step workflow
3. **Execute**: Each step runs with appropriate agent and tools
4. **Track**: Progress stored in Postgres, queryable via API
5. **Complete**: Final report available at `/task_status/{task_id}`

### Database Schema

The service uses SQLAlchemy models for task tracking:

```python
from sqlalchemy import Column, String, Text, DateTime, Integer
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Task(Base):
    __tablename__ = 'tasks'
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    status = Column(String, default='pending')  # pending, running, completed, failed
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
    result = Column(Text)  # Final report
    model = Column(String)  # e.g., "openai:gpt-4o"
```

## API Reference

### POST /generate_report

Start a new research task with multi-step agent workflow.

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### GET /task_progress/{task_id}

Poll real-time progress of a running task.

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "running",
  "current_step": 2,
  "total_steps": 5,
  "steps": [
    {"step": 1, "description": "Research LLM fundamentals", "status": "completed"},
    {"step": 2, "description": "Find scientific applications", "status": "running"},
    {"step": 3, "description": "Synthesize findings", "status": "pending"}
  ]
}
```

### GET /task_status/{task_id}

Retrieve final status and report for a completed task.

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "prompt": "Large Language Models for scientific discovery",
  "result": "# Research Report\n\n## Introduction\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:22Z"
}
```

## Creating Custom Agents

### Research Agent Example

```python
# src/agents.py
from typing import List, Dict

def research_agent(query: str, tools: List[callable], model: str = "openai:gpt-4o") -> str:
    """
    Agent that uses research tools to gather information.
    
    Args:
        query: Research question
        tools: List of tool functions (tavily_search_tool, arxiv_search_tool, etc.)
        model: LLM model identifier
    
    Returns:
        Research findings as formatted text
    """
    # Use aisuite or your LLM client
    import aisuite as ai
    client = ai.Client()
    
    # Build tool descriptions for the LLM
    tool_descriptions = "\n".join([
        f"- {tool.__name__}: {tool.__doc__}" for tool in tools
    ])
    
    system_prompt = f"""You are a research agent. Use these tools:
{tool_descriptions}

Gather comprehensive information about the query."""
    
    # First, decide which tools to use
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": query}
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    # Execute tool calls and gather results
    findings = []
    for tool in tools:
        try:
            result = tool(query)
            findings.append(result)
        except Exception as e:
            findings.append(f"Error with {tool.__name__}: {str(e)}")
    
    # Synthesize findings
    synthesis_prompt = f"Synthesize these research findings:\n\n" + "\n\n".join(findings)
    final_response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "Synthesize research findings into a coherent summary."},
            {"role": "user", "content": synthesis_prompt}
        ]
    )
    
    return final_response.choices[0].message.content
```

### Writer Agent Example

```python
def writer_agent(research_content: str, model: str = "openai:gpt-4o") -> str:
    """
    Agent that transforms research into a structured report.
    """
    import aisuite as ai
    client = ai.Client()
    
    system_prompt = """You are a technical writer. Transform research findings into a well-structured report with:
- Executive Summary
- Key Findings
- Detailed Analysis
- References
"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": research_content}
        ]
    )
    
    return response.choices[0].message.content
```

### Editor Agent Example

```python
def editor_agent(draft: str, model: str = "openai:gpt-4o") -> str:
    """
    Agent that refines and polishes the final report.
    """
    import aisuite as ai
    client = ai.Client()
    
    system_prompt = """You are an editor. Review and improve:
- Clarity and conciseness
- Grammar and style
- Logical flow
- Citation accuracy
"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": f"Edit this draft:\n\n{draft}"}
        ]
    )
    
    return response.choices[0].message.content
```

## Research Tools

### Tavily Search Tool

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> str:
    """
    Search the web using Tavily API for research-quality results.
    
    Requires: TAVILY_API_KEY environment variable
    """
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return "Error: TAVILY_API_KEY not set"
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results,
        "include_answer": True
    }
    
    try:
        response = requests.post(url, json=payload)
        response.raise_for_status()
        data = response.json()
        
        results = []
        if "answer" in data:
            results.append(f"Summary: {data['answer']}\n")
        
        for result in data.get("results", []):
            results.append(f"- {result['title']}: {result['content']}\n  URL: {result['url']}")
        
        return "\n".join(results)
    except Exception as e:
        return f"Tavily search error: {str(e)}"
```

### arXiv Search Tool

```python
import requests
import xml.etree.ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5) -> str:
    """
    Search arXiv for academic papers.
    """
    base_url = "http://export.arxiv.org/api/query"
    params = {
        "search_query": f"all:{query}",
        "start": 0,
        "max_results": max_results,
        "sortBy": "relevance"
    }
    
    try:
        response = requests.get(base_url, params=params)
        response.raise_for_status()
        
        root = ET.fromstring(response.content)
        ns = {"atom": "http://www.w3.org/2005/Atom"}
        
        results = []
        for entry in root.findall("atom:entry", ns):
            title = entry.find("atom:title", ns).text.strip()
            summary = entry.find("atom:summary", ns).text.strip()
            link = entry.find("atom:id", ns).text.strip()
            
            authors = [author.find("atom:name", ns).text 
                      for author in entry.findall("atom:author", ns)]
            
            results.append(f"**{title}**\nAuthors: {', '.join(authors)}\n{summary}\n{link}")
        
        return "\n\n".join(results) if results else "No arXiv results found."
    except Exception as e:
        return f"arXiv search error: {str(e)}"
```

### Wikipedia Search Tool

```python
import wikipedia

def wikipedia_search_tool(query: str) -> str:
    """
    Search Wikipedia for background information.
    """
    try:
        # Search for relevant pages
        search_results = wikipedia.search(query, results=3)
        
        if not search_results:
            return "No Wikipedia results found."
        
        # Get summary of top result
        page = wikipedia.page(search_results[0], auto_suggest=False)
        summary = wikipedia.summary(search_results[0], sentences=5)
        
        return f"**{page.title}**\n\n{summary}\n\nURL: {page.url}"
    except wikipedia.exceptions.DisambiguationError as e:
        # Handle disambiguation pages
        return f"Wikipedia disambiguation: {', '.join(e.options[:5])}"
    except Exception as e:
        return f"Wikipedia search error: {str(e)}"
```

## Planning Agent Integration

### Planner Agent

```python
# src/planning_agent.py
from typing import List, Dict

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> List[Dict[str, str]]:
    """
    Create a multi-step research plan.
    
    Returns:
        List of steps with agent type and description
    """
    import aisuite as ai
    client = ai.Client()
    
    system_prompt = """You are a research planning agent. Break down the research task into 3-5 steps.
For each step, specify:
1. agent_type: "research", "writer", or "editor"
2. description: What this step accomplishes
3. tools: Comma-separated list of tools needed (tavily, arxiv, wikipedia)

Output as JSON array."""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": f"Plan research for: {prompt}"}
        ],
        response_format={"type": "json_object"}
    )
    
    import json
    plan = json.loads(response.choices[0].message.content)
    return plan.get("steps", [])
```

### Executor Agent Step

```python
def executor_agent_step(
    step: Dict[str, str],
    context: str,
    model: str = "openai:gpt-4o"
) -> str:
    """
    Execute a single workflow step.
    
    Args:
        step: Dict with agent_type, description, tools
        context: Previous step results
        model: LLM model to use
    
    Returns:
        Step output
    """
    from src.agents import research_agent, writer_agent, editor_agent
    from src.research_tools import (
        tavily_search_tool,
        arxiv_search_tool,
        wikipedia_search_tool
    )
    
    agent_type = step.get("agent_type", "research")
    description = step.get("description", "")
    tools_str = step.get("tools", "")
    
    # Map tool names to functions
    tool_map = {
        "tavily": tavily_search_tool,
        "arxiv": arxiv_search_tool,
        "wikipedia": wikipedia_search_tool
    }
    
    tools = [tool_map[t.strip()] for t in tools_str.split(",") if t.strip() in tool_map]
    
    # Execute based on agent type
    if agent_type == "research":
        return research_agent(description, tools, model)
    elif agent_type == "writer":
        return writer_agent(context, model)
    elif agent_type == "editor":
        return editor_agent(context, model)
    else:
        return f"Unknown agent type: {agent_type}"
```

## Full Workflow Example

```python
# main.py - Full workflow implementation
import uuid
import threading
from datetime import datetime
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = 'tasks'
    task_id = Column(String, primary_key=True)
    prompt = Column(Text)
    status = Column(String)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
    result = Column(Text)
    model = Column(String)

Base.metadata.create_all(bind=engine)

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

def run_workflow(task_id: str, prompt: str, model: str):
    """Background task that executes the full research workflow."""
    from src.planning_agent import planner_agent, executor_agent_step
    
    db = SessionLocal()
    try:
        # Update status
        task = db.query(Task).filter(Task.task_id == task_id).first()
        task.status = "running"
        task.updated_at = datetime.utcnow()
        db.commit()
        
        # Step 1: Plan
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute each step
        context = prompt
        for i, step in enumerate(plan):
            step_result = executor_agent_step(step, context, model)
            context = step_result  # Pass to next step
            
            # Update progress in DB (optional: store step details)
            task.updated_at = datetime.utcnow()
            db.commit()
        
        # Step 3: Store final result
        task.status = "completed"
        task.result = context
        task.updated_at = datetime.utcnow()
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.result = f"Error: {str(e)}"
        task.updated_at = datetime.utcnow()
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
        status="pending",
        model=request.model,
        created_at=datetime.utcnow(),
        updated_at=datetime.utcnow()
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Start workflow in background
    background_tasks.add_task(run_workflow, task_id, request.prompt, request.model)
    
    return {"task_id": task_id}

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get final task status and result."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.task_id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}
    
    return {
        "task_id": task.task_id,
        "status": task.status,
        "prompt": task.prompt,
        "result": task.result,
        "created_at": task.created_at,
        "updated_at": task.updated_at
    }
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key
TAVILY_API_KEY=tvly-...            # Tavily search API key

# Optional
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb  # Postgres connection
POSTGRES_USER=app                   # DB username (default: app)
POSTGRES_PASSWORD=local             # DB password (default: local)
POSTGRES_DB=appdb                   # DB name (default: appdb)
RESET_DB_ON_STARTUP=0              # Set to 1 to drop tables on startup
```

### Docker Environment

```bash
# Run with custom environment
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -e OPENAI_API_KEY=${OPENAI_API_KEY} \
  -e TAVILY_API_KEY=${TAVILY_API_KEY} \
  -e POSTGRES_PASSWORD=custompassword \
  fastapi-postgres-service
```

## Common Patterns

### Accessing Postgres from Host

```bash
# Connect to the database from your local machine
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT task_id, status, created_at FROM tasks ORDER BY created_at DESC LIMIT 10;
```

### Hot Reload Development

```bash
# Mount local code for live reloading
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Custom Tool Integration

```python
# Add a new research tool
def custom_database_tool(query: str) -> str:
    """Search a custom database."""
    # Your implementation
    return "Custom results"

# Register in executor_agent_step
tool_map = {
    "tavily": tavily_search_tool,
    "arxiv": arxiv_search_tool,
    "wikipedia": wikipedia_search_tool,
    "custom_db": custom_database_tool  # Add here
}
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs fpsvc

# Verify Postgres is starting
docker exec -it fpsvc bash -lc "pg_lsclusters"

# Test Postgres connection
docker exec -it fpsvc bash -lc "psql -U app -d appdb -c 'SELECT 1;'"
```

### API Key Errors

```bash
# Verify environment variables are loaded
docker exec -it fpsvc bash -lc "env | grep -E '(OPENAI|TAVILY)_API_KEY'"

# Check .env file is mounted correctly
docker run --rm -it --env-file .env fastapi-postgres-service env | grep API_KEY
```

### Database Connection Issues

```python
# Test DATABASE_URL in Python
from sqlalchemy import create_engine
import os

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)

try:
    with engine.connect() as conn:
        result = conn.execute("SELECT 1")
        print("Database connected!")
except Exception as e:
    print(f"Connection error: {e}")
```

### Task Stuck in "Running"

```bash
# Check task status directly in DB
docker exec -it fpsvc bash -lc "psql -U app -d appdb -c \"SELECT task_id, status, updated_at FROM tasks WHERE status='running';\""

# View application logs for errors
docker logs -f fpsvc | grep ERROR
```

### Tables Not Created

```python
# Ensure Base.metadata.create_all() is called
# In main.py, after model definitions:

Base.metadata.create_all(bind=engine)

# Verify tables exist
docker exec -it fpsvc bash -lc "psql -U app -d appdb -c '\dt'"
```

### Tool Rate Limiting

```python
# Add retry logic to tools
import time
from functools import wraps

def retry_on_rate_limit(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        for attempt in range(3):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                if "rate limit" in str(e).lower() and attempt < 2:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
                raise
        return None
    return wrapper

@retry_on_rate_limit
def tavily_search_tool(query: str, max_results: int = 5) -> str:
    # Implementation
    pass
```

### Memory Issues with Large Reports

```python
# Stream large results instead of storing in memory
from fastapi.responses import StreamingResponse

@app.get("/task_result_stream/{task_id}")
async def stream_result(task_id: str):
    def generate():
        db = SessionLocal()
        task = db.query(Task).filter(Task.task_id == task_id).first()
        if task and task.result:
            # Stream in chunks
            chunk_size = 1024
            for i in range(0, len(task.result), chunk_size):
                yield task.result[i:i+chunk_size]
        db.close()
    
    return StreamingResponse(generate(), media_type="text/plain")
```

## Additional Resources

- **FastAPI Documentation**: https://fastapi.tiangolo.com/
- **SQLAlchemy ORM Guide**: https://docs.sqlalchemy.org/
- **Tavily API Docs**: https://tavily.com/docs
- **arXiv API Manual**: https://arxiv.org/help/api/
- **AISuite (if used)**: Check your LLM client library docs
