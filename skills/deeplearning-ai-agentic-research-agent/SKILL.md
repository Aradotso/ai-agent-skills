---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with planning, multi-tool execution (Tavily, arXiv, Wikipedia), and Postgres state management for agentic AI workflows
triggers:
  - set up the research agent service
  - create an agentic research workflow
  - build a multi-step research agent with planning
  - use tavily arxiv and wikipedia tools in research
  - implement reflective research agent with fastapi
  - deploy research agent with postgres backend
  - create task-based research agent system
  - build planning and executor agents
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that implements agentic workflows with planning, execution, and reflection. The system uses a planner agent to decompose research tasks, executes research/writing/editing steps using multiple tools (Tavily search, arXiv, Wikipedia), and persists task state in Postgres. Designed for the Agentic Workflow course by DeepLearning.AI.

## What It Does

- **Multi-agent workflow**: Planner → Research → Writer → Editor pipeline
- **Tool integration**: Tavily web search, arXiv academic papers, Wikipedia knowledge
- **Async task execution**: Threaded execution with progress tracking
- **State persistence**: SQLAlchemy + Postgres for task storage
- **REST API**: FastAPI endpoints for task submission, progress monitoring, and results
- **Web UI**: Simple Jinja2 template interface for launching research tasks

## Installation

### Docker Setup (Recommended)

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Create .env file with API keys
cat > .env << EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF

# Build the Docker image
docker build -t agentic-research-agent .

# Run (Postgres + FastAPI in one container)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name research-agent \
  --env-file .env \
  agentic-research-agent
```

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres separately
createdb appdb

# Export environment variables
export DATABASE_URL="postgresql://user:password@localhost:5432/appdb"
export OPENAI_API_KEY="your-openai-key"
export TAVILY_API_KEY="your-tavily-key"

# Run the application
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                    # FastAPI app with endpoints
├── src/
│   ├── planning_agent.py      # Planner and executor agents
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tool implementations (Tavily, arXiv, Wikipedia)
├── templates/
│   └── index.html             # Web UI
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
└── Dockerfile
```

## Key API Endpoints

### POST /generate_report

Start a new research task:

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

### GET /task_progress/{task_id}

Monitor real-time progress:

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

### GET /task_status/{task_id}

Get final status and report:

```python
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
data = response.json()

print(f"Status: {data['status']}")
if data['status'] == 'completed':
    print(f"Final Report:\n{data['final_report']}")
```

## Implementing Custom Agents

### Research Agent Example

```python
# src/agents.py
from typing import Dict, Any

def research_agent(task: str, tools: list) -> Dict[str, Any]:
    """
    Conducts research using available tools.
    
    Args:
        task: Research task description
        tools: List of tool functions (tavily_search, arxiv_search, etc.)
    
    Returns:
        Dict with research findings
    """
    results = []
    
    # Use Tavily for web search
    if 'tavily_search' in [t.__name__ for t in tools]:
        tavily_results = tavily_search_tool(task)
        results.append({
            "source": "tavily",
            "data": tavily_results
        })
    
    # Search academic papers
    if 'arxiv_search' in [t.__name__ for t in tools]:
        arxiv_results = arxiv_search_tool(task)
        results.append({
            "source": "arxiv",
            "data": arxiv_results
        })
    
    return {
        "status": "completed",
        "findings": results,
        "summary": generate_summary(results)
    }
```

### Planning Agent Example

```python
# src/planning_agent.py
def planner_agent(prompt: str, model: str) -> list:
    """
    Breaks down research task into steps.
    
    Returns:
        List of step dictionaries with agent assignments
    """
    # Use LLM to generate plan
    plan_prompt = f"""
    Break down this research task into steps:
    {prompt}
    
    Available agents: research_agent, writer_agent, editor_agent
    Output JSON array of steps.
    """
    
    # Call LLM (using aisuite or similar)
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": plan_prompt}]
    )
    
    steps = parse_plan(response.choices[0].message.content)
    return steps

def executor_agent_step(step: dict, context: dict) -> dict:
    """
    Executes a single step from the plan.
    """
    agent_name = step["agent"]
    task = step["task"]
    
    if agent_name == "research_agent":
        return research_agent(task, context.get("tools", []))
    elif agent_name == "writer_agent":
        return writer_agent(task, context.get("research_results", []))
    elif agent_name == "editor_agent":
        return editor_agent(task, context.get("draft", ""))
    
    raise ValueError(f"Unknown agent: {agent_name}")
```

## Implementing Research Tools

### Tavily Search Tool

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search the web using Tavily API.
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
    response.raise_for_status()
    
    results = response.json().get("results", [])
    return [{
        "title": r["title"],
        "url": r["url"],
        "content": r["content"]
    } for r in results]
```

### arXiv Search Tool

```python
import requests
import xml.etree.ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search arXiv for academic papers.
    """
    base_url = "http://export.arxiv.org/api/query"
    params = {
        "search_query": f"all:{query}",
        "start": 0,
        "max_results": max_results
    }
    
    response = requests.get(base_url, params=params)
    response.raise_for_status()
    
    root = ET.fromstring(response.content)
    ns = {"atom": "http://www.w3.org/2005/Atom"}
    
    papers = []
    for entry in root.findall("atom:entry", ns):
        papers.append({
            "title": entry.find("atom:title", ns).text.strip(),
            "summary": entry.find("atom:summary", ns).text.strip(),
            "authors": [a.find("atom:name", ns).text for a in entry.findall("atom:author", ns)],
            "url": entry.find("atom:id", ns).text
        })
    
    return papers
```

### Wikipedia Search Tool

```python
import wikipedia

def wikipedia_search_tool(query: str, sentences: int = 3) -> dict:
    """
    Search Wikipedia and get summary.
    """
    try:
        # Search for the topic
        search_results = wikipedia.search(query, results=1)
        if not search_results:
            return {"error": "No results found"}
        
        # Get page summary
        page = wikipedia.page(search_results[0], auto_suggest=False)
        summary = wikipedia.summary(search_results[0], sentences=sentences)
        
        return {
            "title": page.title,
            "summary": summary,
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Handle disambiguation
        return {
            "error": "disambiguation",
            "options": e.options[:5]
        }
    except Exception as e:
        return {"error": str(e)}
```

## Database Models

```python
# main.py or models.py
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")
    current_step = Column(String, nullable=True)
    plan = Column(Text, nullable=True)  # JSON string
    progress = Column(Text, nullable=True)  # JSON string
    final_report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Complete Workflow Example

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import uuid
import threading

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

def execute_research_workflow(task_id: str, prompt: str, model: str):
    """
    Background task to execute the full research workflow.
    """
    db = SessionLocal()
    try:
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "running"
        db.commit()
        
        # Step 1: Planning
        task.current_step = "planning"
        db.commit()
        plan = planner_agent(prompt, model)
        task.plan = json.dumps(plan)
        db.commit()
        
        # Step 2: Execute each step
        context = {"tools": [tavily_search_tool, arxiv_search_tool, wikipedia_search_tool]}
        results = []
        
        for i, step in enumerate(plan):
            task.current_step = f"step_{i+1}_{step['agent']}"
            db.commit()
            
            result = executor_agent_step(step, context)
            results.append(result)
            
            # Update context for next step
            if step["agent"] == "research_agent":
                context["research_results"] = result["findings"]
            elif step["agent"] == "writer_agent":
                context["draft"] = result["draft"]
        
        # Final step
        task.status = "completed"
        task.final_report = results[-1].get("final_report", "")
        task.progress = json.dumps(results)
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
        db.commit()
    finally:
        db.close()

@app.post("/generate_report")
async def generate_report(request: ResearchRequest):
    task_id = str(uuid.uuid4())
    
    # Create task record
    db = SessionLocal()
    task = Task(id=task_id, prompt=request.prompt, model=request.model)
    db.add(task)
    db.commit()
    db.close()
    
    # Start background execution
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...                  # OpenAI API key
TAVILY_API_KEY=tvly-...                # Tavily search API key

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://user:pass@host:5432/dbname

# Optional
POSTGRES_USER=app                      # Postgres username
POSTGRES_PASSWORD=local                # Postgres password
POSTGRES_DB=appdb                      # Database name
RESET_DB_ON_STARTUP=0                  # Set to 1 to drop tables on start
```

### Docker Environment

```bash
# Run with custom settings
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  -e TAVILY_API_KEY=$TAVILY_API_KEY \
  -e POSTGRES_PASSWORD=custom_pass \
  --name research-agent \
  agentic-research-agent
```

## Troubleshooting

### Database Connection Issues

```python
# Test database connection
from sqlalchemy import create_engine, text

def test_db_connection():
    try:
        engine = create_engine(os.getenv("DATABASE_URL"))
        with engine.connect() as conn:
            result = conn.execute(text("SELECT 1"))
            print("✅ Database connection successful")
    except Exception as e:
        print(f"❌ Database connection failed: {e}")

test_db_connection()
```

### API Key Validation

```python
def validate_api_keys():
    """Check if required API keys are set."""
    required_keys = ["OPENAI_API_KEY", "TAVILY_API_KEY"]
    missing = [key for key in required_keys if not os.getenv(key)]
    
    if missing:
        raise ValueError(f"Missing required API keys: {', '.join(missing)}")
    
    print("✅ All required API keys are set")

# Add to app startup
@app.on_event("startup")
async def startup_event():
    validate_api_keys()
```

### Task Not Progressing

```bash
# Check container logs
docker logs -f research-agent

# Connect to database to inspect task state
docker exec -it research-agent bash -c "psql postgresql://app:local@127.0.0.1:5432/appdb -c 'SELECT id, status, current_step, error FROM tasks;'"
```

### Tool Rate Limits

```python
import time
from functools import wraps

def rate_limit(calls_per_minute=5):
    """Decorator to rate-limit tool calls."""
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            if elapsed < min_interval:
                time.sleep(min_interval - elapsed)
            
            result = func(*args, **kwargs)
            last_called[0] = time.time()
            return result
        return wrapper
    return decorator

@rate_limit(calls_per_minute=5)
def wikipedia_search_tool(query: str) -> dict:
    # ... implementation
    pass
```

### Memory Issues with Large Tasks

```python
# Limit result sizes
def research_agent(task: str, tools: list, max_results: int = 3) -> Dict[str, Any]:
    results = []
    
    for tool in tools:
        try:
            tool_results = tool(task, max_results=max_results)
            # Truncate large content
            if isinstance(tool_results, list):
                for r in tool_results:
                    if "content" in r and len(r["content"]) > 2000:
                        r["content"] = r["content"][:2000] + "..."
            results.append(tool_results)
        except Exception as e:
            print(f"Tool {tool.__name__} failed: {e}")
    
    return {"findings": results}
```

## Web UI Access

```bash
# Open browser to UI
open http://localhost:8000

# Or use curl to test endpoints
curl http://localhost:8000/docs  # OpenAPI documentation
```
