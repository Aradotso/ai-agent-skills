---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with multi-step planning, tool integration (Tavily, arXiv, Wikipedia), and Postgres state management
triggers:
  - set up agentic research workflow
  - create a research agent with planning and tools
  - build multi-step agent with tavily and arxiv
  - implement reflective research agent service
  - deploy fastapi agent with postgres state
  - use deeplearning ai agentic workflow
  - create tool-using research agents
  - build planning executor agent system
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that implements a multi-step agentic workflow with planning, execution, and reflection capabilities. The system uses a planner agent to decompose research tasks, executor agents (research, writer, editor) to perform work, and integrates external tools (Tavily search, arXiv, Wikipedia) for information gathering. Task state and results are persisted in PostgreSQL.

## What It Does

- **Multi-Agent Workflow**: Orchestrates planner → research → writer → editor agents
- **Tool Integration**: Connects to Tavily API, arXiv, and Wikipedia for research
- **State Management**: Tracks task progress and substeps in PostgreSQL
- **Async Execution**: Runs research workflows in background threads
- **Web UI**: Simple interface for kicking off tasks and monitoring progress
- **RESTful API**: Endpoints for task creation, progress tracking, and result retrieval

## Installation

### With Docker (Recommended)

1. **Clone the repository**:
```bash
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file** with required API keys:
```bash
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
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

### Local Development

1. **Install dependencies**:
```bash
pip install -r requirements.txt
```

2. **Set up PostgreSQL** and export `DATABASE_URL`:
```bash
export DATABASE_URL="postgresql://user:password@localhost:5432/appdb"
export OPENAI_API_KEY="your-openai-key"
export TAVILY_API_KEY="your-tavily-key"
```

3. **Run the service**:
```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Project Structure

```
├── main.py                    # FastAPI application, endpoints, DB models
├── src/
│   ├── planning_agent.py      # Planner and executor agent logic
│   ├── agents.py              # Research, writer, editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── Dockerfile
└── requirements.txt
```

## Key API Endpoints

### 1. Start Research Task

**POST** `/generate_report`

Kicks off an asynchronous research workflow.

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

**Request Body**:
```json
{
  "prompt": "Your research query",
  "model": "openai:gpt-4o"
}
```

**Response**:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 2. Poll Task Progress

**GET** `/task_progress/{task_id}`

Returns live status of each workflow step and substep.

```python
progress = requests.get(
    f"http://localhost:8000/task_progress/{task_id}"
).json()

print(f"Status: {progress['status']}")
print(f"Current step: {progress['current_step']}")
for substep in progress.get("substeps", []):
    print(f"  - {substep['agent']}: {substep['status']}")
```

**Response**:
```json
{
  "task_id": "550e8400...",
  "status": "in_progress",
  "current_step": 2,
  "total_steps": 5,
  "substeps": [
    {
      "agent": "planner",
      "status": "completed",
      "result": "Research plan created"
    },
    {
      "agent": "research_agent",
      "status": "running",
      "result": null
    }
  ]
}
```

### 3. Get Final Report

**GET** `/task_status/{task_id}`

Returns final status and generated report.

```python
result = requests.get(
    f"http://localhost:8000/task_status/{task_id}"
).json()

if result["status"] == "completed":
    print(result["report"])
```

**Response**:
```json
{
  "task_id": "550e8400...",
  "status": "completed",
  "report": "# Research Report\n\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-01-15T10:35:00Z"
}
```

## Building Custom Agents

### Creating a Research Tool

Tools are defined in `src/research_tools.py`:

```python
from typing import Dict, Any

def custom_search_tool(query: str) -> Dict[str, Any]:
    """
    Custom research tool for your data source.
    
    Args:
        query: Search query string
        
    Returns:
        Dictionary with 'results' key containing findings
    """
    # Implement your search logic
    results = your_api.search(query)
    
    return {
        "results": [
            {"title": r.title, "content": r.content, "url": r.url}
            for r in results
        ]
    }
```

### Creating a Custom Agent

Agents are defined in `src/agents.py`:

```python
import aisuite as ai

def analysis_agent(prompt: str, context: str, model: str = "openai:gpt-4o") -> str:
    """
    Agent that performs data analysis.
    
    Args:
        prompt: User's analysis request
        context: Data/context to analyze
        model: LLM model to use
        
    Returns:
        Analysis results as string
    """
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a data analysis expert. Analyze the provided data and generate insights."
        },
        {
            "role": "user",
            "content": f"Request: {prompt}\n\nData: {context}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content
```

### Implementing the Planning Agent

The planner decomposes tasks into steps:

```python
# In src/planning_agent.py
import aisuite as ai
import json

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> list:
    """
    Creates a research plan with ordered steps.
    
    Args:
        prompt: Research question/task
        model: LLM model to use
        
    Returns:
        List of step dictionaries
    """
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": """You are a research planning expert. Break down the research task 
            into ordered steps. Return JSON array of steps with 'agent', 'action', and 'inputs' keys."""
        },
        {
            "role": "user",
            "content": f"Create a research plan for: {prompt}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    plan = json.loads(response.choices[0].message.content)
    return plan["steps"]
```

## Database Models

The service uses SQLAlchemy models defined in `main.py`:

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
    status = Column(String, default="pending")  # pending, in_progress, completed, failed
    report = Column(Text, nullable=True)
    current_step = Column(Integer, default=0)
    total_steps = Column(Integer, default=0)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

class SubTask(Base):
    __tablename__ = "subtasks"
    
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    task_id = Column(String, nullable=False)
    agent = Column(String, nullable=False)  # planner, research_agent, writer_agent, etc.
    status = Column(String, default="pending")
    result = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
```

### Querying Task State

```python
from sqlalchemy.orm import Session

def get_task_with_substeps(db: Session, task_id: str):
    """Get task and all its substeps."""
    task = db.query(Task).filter(Task.id == task_id).first()
    subtasks = db.query(SubTask).filter(SubTask.task_id == task_id).all()
    
    return {
        "task": task,
        "substeps": [
            {
                "agent": st.agent,
                "status": st.status,
                "result": st.result
            }
            for st in subtasks
        ]
    }
```

## Configuration

### Environment Variables

Required:
- `OPENAI_API_KEY`: OpenAI API key for LLM calls
- `TAVILY_API_KEY`: Tavily API key for web search

Optional:
- `DATABASE_URL`: PostgreSQL connection string (default: `postgresql://app:local@127.0.0.1:5432/appdb`)
- `POSTGRES_USER`: Database username (default: `app`)
- `POSTGRES_PASSWORD`: Database password (default: `local`)
- `POSTGRES_DB`: Database name (default: `appdb`)

### Model Configuration

The service supports multiple LLM providers through `aisuite`:

```python
# OpenAI
response = requests.post(
    "http://localhost:8000/generate_report",
    json={"prompt": "...", "model": "openai:gpt-4o"}
)

# Anthropic
response = requests.post(
    "http://localhost:8000/generate_report",
    json={"prompt": "...", "model": "anthropic:claude-3-opus"}
)
```

## Common Patterns

### Building a Complete Workflow

```python
from fastapi import BackgroundTasks
from sqlalchemy.orm import Session
import threading

def execute_research_workflow(task_id: str, prompt: str, model: str, db: Session):
    """
    Complete research workflow execution.
    """
    # Update task status
    task = db.query(Task).filter(Task.id == task_id).first()
    task.status = "in_progress"
    db.commit()
    
    try:
        # Step 1: Planning
        plan = planner_agent(prompt, model)
        task.total_steps = len(plan)
        db.commit()
        
        context = {"prompt": prompt}
        
        # Step 2: Execute each step
        for i, step in enumerate(plan):
            task.current_step = i + 1
            
            # Create subtask
            subtask = SubTask(
                task_id=task_id,
                agent=step["agent"],
                status="running"
            )
            db.add(subtask)
            db.commit()
            
            # Execute agent
            if step["agent"] == "research_agent":
                result = research_agent(step["inputs"], context, model)
            elif step["agent"] == "writer_agent":
                result = writer_agent(context, model)
            elif step["agent"] == "editor_agent":
                result = editor_agent(context, model)
            
            # Update subtask
            subtask.status = "completed"
            subtask.result = result
            context[step["agent"]] = result
            db.commit()
        
        # Step 3: Finalize
        task.status = "completed"
        task.report = context.get("editor_agent", "")
        db.commit()
        
    except Exception as e:
        task.status = "failed"
        task.report = f"Error: {str(e)}"
        db.commit()

# In your endpoint
@app.post("/generate_report")
async def generate_report(request: ReportRequest, background_tasks: BackgroundTasks):
    task_id = str(uuid.uuid4())
    
    # Create task in DB
    task = Task(id=task_id, prompt=request.prompt, total_steps=0)
    db.add(task)
    db.commit()
    
    # Start background execution
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, request.prompt, request.model, db)
    )
    thread.start()
    
    return {"task_id": task_id}
```

### Implementing Tool Calls

```python
from src.research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(inputs: dict, context: dict, model: str) -> str:
    """
    Research agent that uses multiple tools.
    """
    query = inputs.get("query", context["prompt"])
    
    # Use Tavily for web search
    web_results = tavily_search_tool(query)
    
    # Use arXiv for academic papers
    arxiv_results = arxiv_search_tool(query, max_results=5)
    
    # Combine and synthesize
    combined_context = f"""
    Web Search Results:
    {web_results}
    
    Academic Papers:
    {arxiv_results}
    """
    
    client = ai.Client()
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "Synthesize research findings into a coherent summary."},
            {"role": "user", "content": f"Query: {query}\n\nSources:\n{combined_context}"}
        ]
    )
    
    return response.choices[0].message.content
```

## Troubleshooting

### Database Connection Issues

**Problem**: `DATABASE_URL not set` error

**Solution**: Ensure the environment variable is exported:
```bash
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
```

For Docker, the entrypoint sets this automatically. If overriding, ensure format is correct.

### Tables Not Persisting

**Problem**: Tables disappear on restart

**Solution**: Comment out `Base.metadata.drop_all()` in `main.py`:
```python
# Remove or guard this line
# Base.metadata.drop_all(bind=engine)
Base.metadata.create_all(bind=engine)
```

Or use an environment flag:
```python
if os.getenv("RESET_DB_ON_STARTUP") != "1":
    Base.metadata.create_all(bind=engine)
```

### API Key Errors

**Problem**: `TAVILY_API_KEY` not found

**Solution**: Verify `.env` file exists and is loaded:
```bash
# .env file
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

Load it when running:
```bash
docker run --env-file .env ...
```

### Container Port Conflicts

**Problem**: Port 8000 or 5432 already in use

**Solution**: Map to different host ports:
```bash
docker run -p 8080:8000 -p 5433:5432 ...
```

Access at `http://localhost:8080`

### Hot Reload for Development

Mount your code as a volume and enable Uvicorn reload:
```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -c "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Connecting to Database from Host

```bash
psql "postgresql://app:local@localhost:5432/appdb"
```

Query task state:
```sql
SELECT id, prompt, status, current_step, total_steps FROM tasks;
SELECT task_id, agent, status FROM subtasks WHERE task_id = 'your-task-id';
```

### Template Not Found Errors

**Problem**: 404 or template errors when accessing `/`

**Solution**: Verify templates are copied to container:
```dockerfile
# In Dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

Check inside container:
```bash
docker exec -it fpsvc ls -l /app/templates
```
