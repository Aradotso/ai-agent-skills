---
name: deeplearning-ai-agentic-research-agent
description: FastAPI service with multi-step reflective research agents using Tavily, arXiv, Wikipedia, and Postgres for task tracking
triggers:
  - set up agentic research workflow
  - create multi-step research agent with planning
  - use deeplearning ai research agent service
  - build reflective research agent with FastAPI
  - implement task-based research agent workflow
  - configure research agent with Tavily and arXiv
  - deploy agentic AI research service
  - create research planner agent workflow
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This project implements a reflective research agent service built with FastAPI and Postgres. It orchestrates multi-step workflows using a planning agent that coordinates research, writing, and editing agents. The agents use tools like Tavily search, arXiv, and Wikipedia to gather information and generate comprehensive research reports.

## What It Does

- **Multi-step workflow**: Planner agent breaks down research tasks into steps
- **Tool-using agents**: Research, writer, and editor agents use external APIs
- **Task tracking**: Postgres database stores task state, progress, and results
- **Live progress**: Real-time status updates for each workflow step
- **Web UI**: Simple interface to kick off and monitor research tasks
- **RESTful API**: Endpoints for programmatic access

## Installation

### Docker Setup (Recommended)

```bash
# Clone the repository
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Create .env file with required API keys
cat > .env << EOF
OPENAI_API_KEY=${OPENAI_API_KEY}
TAVILY_API_KEY=${TAVILY_API_KEY}
EOF

# Build the Docker image
docker build -t agentic-research-service .

# Run the container (Postgres + FastAPI in one)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name research-agent \
  --env-file .env \
  agentic-research-service
```

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres separately
export DATABASE_URL="postgresql://user:password@localhost:5432/appdb"

# Run the FastAPI app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=your-key-here          # For LLM calls
TAVILY_API_KEY=your-key-here          # For web search

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional overrides
POSTGRES_USER=app                     # Default database user
POSTGRES_PASSWORD=local               # Default database password
POSTGRES_DB=appdb                     # Default database name
RESET_DB_ON_STARTUP=0                 # Set to 1 to drop tables on restart
```

## Project Structure

```
.
├── main.py                    # FastAPI application entry point
├── src/
│   ├── planning_agent.py      # Planner and executor logic
│   ├── agents.py              # Research/writer/editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## API Endpoints

### Generate Research Report

```python
import requests

# Start a research task
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

# Check progress periodically
while True:
    progress = requests.get(
        f"http://localhost:8000/task_progress/{task_id}"
    ).json()
    
    print(f"Status: {progress['status']}")
    print(f"Current step: {progress.get('current_step', 'N/A')}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(5)
```

### Get Final Report

```python
import requests

# Retrieve final status and report
result = requests.get(
    f"http://localhost:8000/task_status/{task_id}"
).json()

print(f"Status: {result['status']}")
print(f"Report:\n{result.get('report', 'No report generated')}")
```

### List All Tasks

```python
import requests

# Get all tasks
tasks = requests.get("http://localhost:8000/tasks").json()

for task in tasks:
    print(f"{task['id']}: {task['status']} - {task['prompt'][:50]}")
```

## Creating Custom Agents

### Research Agent Example

```python
# src/agents.py
from typing import List, Dict
import os

def research_agent(query: str, tools: List[callable]) -> Dict:
    """
    Execute research using available tools.
    
    Args:
        query: Research question
        tools: List of tool functions (tavily_search, arxiv_search, etc.)
    
    Returns:
        Dictionary with findings and sources
    """
    findings = []
    sources = []
    
    for tool in tools:
        try:
            result = tool(query)
            findings.append(result["content"])
            sources.extend(result.get("sources", []))
        except Exception as e:
            print(f"Tool {tool.__name__} failed: {e}")
    
    return {
        "findings": "\n\n".join(findings),
        "sources": sources,
        "query": query
    }
```

### Custom Research Tool

```python
# src/research_tools.py
import requests
import os

def tavily_search_tool(query: str, max_results: int = 5) -> Dict:
    """
    Search using Tavily API.
    
    Args:
        query: Search query
        max_results: Maximum number of results
    
    Returns:
        Dictionary with content and sources
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
    
    data = response.json()
    
    return {
        "content": "\n".join([r["content"] for r in data["results"]]),
        "sources": [r["url"] for r in data["results"]]
    }
```

## Implementing Planning Agent

```python
# src/planning_agent.py
from typing import List, Dict
import json

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> List[Dict]:
    """
    Create a research plan with multiple steps.
    
    Args:
        prompt: Research topic/question
        model: LLM model to use
    
    Returns:
        List of plan steps with agent assignments
    """
    import aisuite as ai
    
    client = ai.Client()
    
    system_prompt = """You are a research planner. Break down the research task
    into specific steps. Return a JSON array of steps, each with:
    - step_name: Brief description
    - agent: research_agent, writer_agent, or editor_agent
    - instructions: Detailed instructions for the agent"""
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": prompt}
        ]
    )
    
    plan = json.loads(response.choices[0].message.content)
    return plan


def executor_agent_step(step: Dict, context: Dict, tools: List) -> Dict:
    """
    Execute a single plan step.
    
    Args:
        step: Step definition from planner
        context: Previous step results
        tools: Available tools for agents
    
    Returns:
        Step execution result
    """
    agent_name = step["agent"]
    
    if agent_name == "research_agent":
        from agents import research_agent
        result = research_agent(step["instructions"], tools)
    elif agent_name == "writer_agent":
        from agents import writer_agent
        result = writer_agent(step["instructions"], context)
    elif agent_name == "editor_agent":
        from agents import editor_agent
        result = editor_agent(context["draft"], step["instructions"])
    else:
        raise ValueError(f"Unknown agent: {agent_name}")
    
    return result
```

## Database Models

```python
# main.py or models.py
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import datetime
import os

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.datetime.utcnow)

# Initialize database
DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
Base.metadata.create_all(bind=engine)
SessionLocal = sessionmaker(bind=engine)
```

## FastAPI Integration

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import uuid
import threading

app = FastAPI()

class ResearchRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"

@app.post("/generate_report")
async def generate_report(
    request: ResearchRequest,
    background_tasks: BackgroundTasks
):
    """Start a research task."""
    task_id = str(uuid.uuid4())
    
    # Create task in database
    db = SessionLocal()
    task = Task(
        id=task_id,
        prompt=request.prompt,
        model=request.model,
        status="running"
    )
    db.add(task)
    db.commit()
    db.close()
    
    # Run workflow in background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.start()
    
    return {"task_id": task_id}

@app.get("/task_progress/{task_id}")
async def task_progress(task_id: str):
    """Get live progress of a task."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}, 404
    
    return {
        "task_id": task.id,
        "status": task.status,
        "current_step": task.current_step,
        "created_at": task.created_at.isoformat()
    }

@app.get("/task_status/{task_id}")
async def task_status(task_id: str):
    """Get final status and report."""
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    db.close()
    
    if not task:
        return {"error": "Task not found"}, 404
    
    return {
        "task_id": task.id,
        "status": task.status,
        "report": task.report,
        "error": task.error
    }
```

## Common Patterns

### Multi-Step Workflow Execution

```python
def run_research_workflow(task_id: str, prompt: str, model: str):
    """Execute complete research workflow."""
    from src.planning_agent import planner_agent, executor_agent_step
    from src.research_tools import tavily_search_tool, arxiv_search_tool
    
    tools = [tavily_search_tool, arxiv_search_tool]
    db = SessionLocal()
    
    try:
        # Generate plan
        plan = planner_agent(prompt, model)
        
        # Execute each step
        context = {}
        for i, step in enumerate(plan):
            # Update progress
            task = db.query(Task).filter(Task.id == task_id).first()
            task.current_step = f"{i+1}/{len(plan)}: {step['step_name']}"
            db.commit()
            
            # Execute step
            result = executor_agent_step(step, context, tools)
            context[step["step_name"]] = result
        
        # Mark complete
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "completed"
        task.report = context.get("final_report", str(context))
        db.commit()
        
    except Exception as e:
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "failed"
        task.error = str(e)
        db.commit()
    
    finally:
        db.close()
```

### Tool Error Handling

```python
def safe_tool_call(tool_func, *args, **kwargs):
    """Wrapper for tool calls with error handling."""
    try:
        return tool_func(*args, **kwargs)
    except requests.RequestException as e:
        return {
            "content": f"Network error: {e}",
            "sources": [],
            "error": True
        }
    except Exception as e:
        return {
            "content": f"Tool error: {e}",
            "sources": [],
            "error": True
        }
```

## Troubleshooting

### Database Connection Issues

```bash
# Check if Postgres is running
docker exec -it research-agent pg_isready

# Connect to database manually
docker exec -it research-agent psql -U app -d appdb

# View task table
docker exec -it research-agent psql -U app -d appdb -c "SELECT * FROM tasks;"
```

### API Key Errors

```python
# Verify API keys are loaded
import os
from dotenv import load_dotenv

load_dotenv()

required_keys = ["OPENAI_API_KEY", "TAVILY_API_KEY"]
for key in required_keys:
    if not os.getenv(key):
        print(f"Missing: {key}")
    else:
        print(f"{key}: configured")
```

### Container Won't Start

```bash
# Check logs
docker logs research-agent

# Verify entrypoint script
docker exec -it research-agent cat /docker/entrypoint.sh

# Manual Postgres start
docker exec -it research-agent bash -lc "pg_ctlcluster 17 main start"
```

### Tables Missing After Restart

```python
# Disable table dropping in main.py
# Comment out or guard with environment variable:
if os.getenv("RESET_DB_ON_STARTUP") != "1":
    # Base.metadata.drop_all(bind=engine)  # Comment this line
    pass
Base.metadata.create_all(bind=engine)
```

### Rate Limiting Issues

```python
import time
from functools import wraps

def rate_limit(calls_per_minute=10):
    """Decorator to rate limit function calls."""
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            wait_time = min_interval - elapsed
            if wait_time > 0:
                time.sleep(wait_time)
            last_called[0] = time.time()
            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(calls_per_minute=5)
def wikipedia_search_tool(query: str):
    # Tool implementation
    pass
```

### Testing Workflow Locally

```bash
# Start service
docker run --rm -it -p 8000:8000 --env-file .env agentic-research-service

# In another terminal, test the API
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Quantum computing applications in drug discovery",
    "model": "openai:gpt-4o"
  }'

# Poll progress
TASK_ID="<task-id-from-above>"
watch -n 2 "curl -s http://localhost:8000/task_progress/$TASK_ID | jq"
```
