---
name: deeplearning-ai-agentic-research-agent
description: FastAPI service for building reflective research agents with planning, tool integration (Tavily, arXiv, Wikipedia), and Postgres-backed task tracking
triggers:
  - how do I set up the agentic research agent service
  - create a research workflow with planning and tools
  - implement a multi-step research agent with FastAPI
  - use Tavily arXiv and Wikipedia tools in research agents
  - track agent task progress with Postgres
  - build a reflective research agent with planning
  - set up the DeepLearning.AI research agent project
  - how to use the agentic workflow research service
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that implements agentic workflows with planning, execution, and reflection. The system uses a planner agent to decompose research tasks, executes them using specialized agents (research, writer, editor), and integrates external tools (Tavily search, arXiv, Wikipedia). Task state and results are persisted in PostgreSQL.

## What It Does

- **Multi-Agent Workflow**: Coordinates planner, research, writer, and editor agents
- **Tool Integration**: Provides Tavily web search, arXiv academic search, and Wikipedia lookups
- **Task Tracking**: Stores task state, progress, and results in PostgreSQL
- **REST API**: FastAPI endpoints for task submission and status polling
- **Web UI**: Simple HTML interface for kicking off research tasks
- **Reflective Planning**: Agents plan, execute, and reflect on multi-step workflows

## Installation

### Using Docker (Recommended)

1. **Clone the repository**:
```bash
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file** with required API keys:
```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

3. **Build the Docker image**:
```bash
docker build -t fastapi-postgres-service .
```

4. **Run the container**:
```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will start PostgreSQL internally and launch the FastAPI app on port 8000.

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up PostgreSQL separately
export DATABASE_URL="postgresql://user:password@localhost:5432/appdb"

# Run the app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                    # FastAPI app with routes and DB models
├── src/
│   ├── planning_agent.py      # planner_agent() and executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # tavily, arxiv, wikipedia tool functions
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── Dockerfile
└── requirements.txt
```

## Key API Endpoints

### Submit Research Task

**POST** `/generate_report`

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
print(f"Task ID: {task_id}")
```

### Poll Task Progress

**GET** `/task_progress/{task_id}`

```python
import requests
import time

def poll_progress(task_id):
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = response.json()
        
        print(f"Status: {data['status']}")
        print(f"Current Step: {data.get('current_step', 'N/A')}")
        
        if data["status"] in ["completed", "failed"]:
            break
        
        time.sleep(2)

poll_progress(task_id)
```

### Get Final Report

**GET** `/task_status/{task_id}`

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
data = response.json()

print(f"Status: {data['status']}")
if data['status'] == 'completed':
    print(f"Report:\n{data['report']}")
```

## Building Custom Agents

### Creating a Research Tool

```python
# src/research_tools.py
import requests
import os

def custom_search_tool(query: str, max_results: int = 5) -> str:
    """Custom search tool implementation."""
    api_key = os.getenv("CUSTOM_API_KEY")
    
    # Implement your search logic
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query, "limit": max_results},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    
    results = response.json()
    
    # Format results for agent consumption
    formatted = []
    for result in results.get("items", []):
        formatted.append(f"Title: {result['title']}\nURL: {result['url']}\n")
    
    return "\n".join(formatted)
```

### Creating a Specialized Agent

```python
# src/agents.py
import aisuite as ai

def analysis_agent(task: str, context: str, model: str = "openai:gpt-4o") -> str:
    """Agent that performs data analysis."""
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a data analysis expert. Analyze the provided data and extract insights."
        },
        {
            "role": "user",
            "content": f"Task: {task}\n\nContext:\n{context}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

### Implementing a Planning Agent

```python
# src/planning_agent.py
import aisuite as ai
import json

def planner_agent(query: str, model: str = "openai:gpt-4o") -> dict:
    """Plans multi-step research workflow."""
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": """You are a research planning expert. Break down the research query into logical steps.
Return a JSON object with:
- steps: array of step objects with {step_number, action, description}
- estimated_time: overall time estimate
"""
        },
        {
            "role": "user",
            "content": f"Plan research for: {query}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    plan = json.loads(response.choices[0].message.content)
    return plan

def executor_agent_step(step: dict, context: str, model: str = "openai:gpt-4o") -> str:
    """Executes a single planned step."""
    client = ai.Client()
    
    # Determine which tools to use based on step action
    tools_output = ""
    if "search" in step["action"].lower():
        from src.research_tools import tavily_search_tool
        tools_output = tavily_search_tool(step["description"])
    elif "arxiv" in step["action"].lower():
        from src.research_tools import arxiv_search_tool
        tools_output = arxiv_search_tool(step["description"])
    
    messages = [
        {
            "role": "system",
            "content": "Execute this research step and provide results."
        },
        {
            "role": "user",
            "content": f"Step: {step['description']}\n\nTool Output:\n{tools_output}\n\nContext:\n{context}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return response.choices[0].message.content
```

## Database Models

The service uses SQLAlchemy models to track task state:

```python
from sqlalchemy import Column, String, Text, DateTime, Integer
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime
import uuid

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, running, completed, failed
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Database (set automatically in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional
POSTGRES_USER=app               # PostgreSQL username
POSTGRES_PASSWORD=local         # PostgreSQL password
POSTGRES_DB=appdb               # Database name
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on startup
```

### Custom Model Configuration

```python
# main.py
from pydantic import BaseModel

class GenerateReportRequest(BaseModel):
    prompt: str
    model: str = "openai:gpt-4o"  # Default model
    max_steps: int = 5            # Maximum planning steps
    temperature: float = 0.7      # LLM temperature

@app.post("/generate_report")
async def generate_report(request: GenerateReportRequest):
    # Use custom configuration
    task = Task(
        prompt=request.prompt,
        model=request.model,
        status="pending"
    )
    # ... rest of implementation
```

## Common Patterns

### Implementing a Complete Research Workflow

```python
import threading
from sqlalchemy.orm import Session
from src.planning_agent import planner_agent, executor_agent_step
from src.agents import research_agent, writer_agent, editor_agent

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Complete research workflow in background thread."""
    from main import SessionLocal, Task
    
    db = SessionLocal()
    
    try:
        # Update status
        task = db.query(Task).filter(Task.id == task_id).first()
        task.status = "running"
        task.current_step = "planning"
        db.commit()
        
        # Step 1: Plan
        plan = planner_agent(prompt, model)
        
        # Step 2: Execute steps
        context = ""
        for step in plan["steps"]:
            task.current_step = f"step_{step['step_number']}"
            db.commit()
            
            result = executor_agent_step(step, context, model)
            context += f"\n\nStep {step['step_number']} Result:\n{result}"
        
        # Step 3: Research
        task.current_step = "research"
        db.commit()
        research_output = research_agent(prompt, context, model)
        
        # Step 4: Write
        task.current_step = "writing"
        db.commit()
        draft = writer_agent(prompt, research_output, model)
        
        # Step 5: Edit
        task.current_step = "editing"
        db.commit()
        final_report = editor_agent(draft, model)
        
        # Complete
        task.status = "completed"
        task.report = final_report
        task.current_step = "done"
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
        db.commit()
    finally:
        db.close()

# Start workflow in background
thread = threading.Thread(
    target=run_research_workflow,
    args=(task_id, prompt, model)
)
thread.daemon = True
thread.start()
```

### Using Research Tools

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> str:
    """Search using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "query": query,
            "api_key": api_key,
            "max_results": max_results
        }
    )
    
    results = response.json()
    formatted = []
    for result in results.get("results", []):
        formatted.append(
            f"Title: {result['title']}\n"
            f"URL: {result['url']}\n"
            f"Snippet: {result['content']}\n"
        )
    
    return "\n".join(formatted)

def arxiv_search_tool(query: str, max_results: int = 5) -> str:
    """Search arXiv for academic papers."""
    import arxiv
    
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for paper in search.results():
        results.append(
            f"Title: {paper.title}\n"
            f"Authors: {', '.join(a.name for a in paper.authors)}\n"
            f"Published: {paper.published}\n"
            f"Summary: {paper.summary}\n"
            f"URL: {paper.entry_id}\n"
        )
    
    return "\n".join(results)

def wikipedia_search_tool(query: str) -> str:
    """Search Wikipedia."""
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return f"Title: {page.title}\n\nSummary:\n{page.summary}\n\nURL: {page.url}"
    except wikipedia.exceptions.DisambiguationError as e:
        return f"Multiple results found: {', '.join(e.options[:5])}"
    except wikipedia.exceptions.PageError:
        return f"No Wikipedia page found for: {query}"
```

## Troubleshooting

### Container won't start

```bash
# Check logs
docker logs fpsvc

# Verify .env file exists and has valid keys
cat .env

# Ensure ports aren't already in use
lsof -i :8000
lsof -i :5432
```

### Database connection errors

```bash
# Verify DATABASE_URL is set correctly
docker exec -it fpsvc bash -c 'echo $DATABASE_URL'

# Connect to DB manually to test
docker exec -it fpsvc bash -c 'psql $DATABASE_URL -c "SELECT 1;"'

# Check if tables exist
docker exec -it fpsvc bash -c 'psql $DATABASE_URL -c "\dt"'
```

### Task stuck in "running" state

```python
# Query database to check task status
import psycopg2
import os

conn = psycopg2.connect(os.getenv("DATABASE_URL"))
cur = conn.cursor()

cur.execute("SELECT id, status, current_step, error FROM tasks WHERE id = %s", (task_id,))
print(cur.fetchone())

# Reset stuck task
cur.execute("UPDATE tasks SET status = 'pending', current_step = NULL WHERE id = %s", (task_id,))
conn.commit()
```

### API key errors

```bash
# Verify API keys are loaded
docker exec -it fpsvc bash -c 'echo $OPENAI_API_KEY | head -c 10'
docker exec -it fpsvc bash -c 'echo $TAVILY_API_KEY | head -c 10'

# Test Tavily API directly
curl -X POST https://api.tavily.com/search \
  -H "Content-Type: application/json" \
  -d "{\"query\": \"test\", \"api_key\": \"$TAVILY_API_KEY\"}"
```

### Hot reload for development

```bash
# Mount code and run with reload
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Access PostgreSQL from host

```bash
# Connect using psql
psql "postgresql://app:local@localhost:5432/appdb"

# Or use environment variable
export DATABASE_URL="postgresql://app:local@localhost:5432/appdb"
psql $DATABASE_URL
```
