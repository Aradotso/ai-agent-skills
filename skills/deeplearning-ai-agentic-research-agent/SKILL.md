---
name: deeplearning-ai-agentic-research-agent
description: FastAPI-based reflective research agent with planning, multi-step workflows, and Postgres task tracking for AI research workflows
triggers:
  - how do I set up the agentic AI research agent
  - use the DeepLearning.AI research agent to analyze a topic
  - create a research workflow with planning and reflection agents
  - configure the FastAPI agentic research service
  - run a multi-step research task with Tavily and arXiv
  - build a reflective AI agent with task progress tracking
  - implement agentic workflows with planner and executor agents
  - deploy the research agent service with Docker and Postgres
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The **DeepLearning.AI Agentic Research Agent** is a FastAPI-based service that implements reflective, multi-step research workflows. It uses a planner-executor architecture where agents collaborate to research topics using tools like Tavily, arXiv, and Wikipedia, then write and edit comprehensive reports. All task state and results are persisted in Postgres for live progress tracking.

**Key capabilities:**
- **Planning Agent**: Breaks down research tasks into structured steps
- **Executor Agents**: Research, writer, and editor agents that collaborate
- **Tool Integration**: Tavily search, arXiv papers, Wikipedia lookups
- **Task Tracking**: Real-time progress via Postgres-backed API endpoints
- **Single-Container Deploy**: Postgres + FastAPI in one Docker image for easy dev/testing

## Installation

### Prerequisites

1. **Docker** installed on your system
2. **API Keys** in a `.env` file at project root:

```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build and Run

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the service (exposes API on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service starts Postgres, creates the database schema, and launches the FastAPI app. Look for:

```
✅ Postgres is ready
🔗 DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
INFO:     Uvicorn running on http://0.0.0.0:8000
```

## Key API Endpoints

### 1. Web UI (GET /)

Open http://localhost:8000/ for a simple web interface to submit research tasks.

### 2. Generate Report (POST /generate_report)

Kick off a research workflow with a prompt and model specification.

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

**cURL example:**

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications in cryptography", "model":"openai:gpt-4o"}'
```

### 3. Task Progress (GET /task_progress/{task_id})

Poll for live updates on planning and execution steps.

```python
import requests
import time

task_id = "your-task-uuid"

while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    
    print(f"Status: {progress['status']}")
    print(f"Current step: {progress.get('current_step', 'N/A')}")
    
    if progress['status'] in ['completed', 'failed']:
        break
    
    time.sleep(2)
```

### 4. Task Status & Report (GET /task_status/{task_id})

Retrieve the final report and full execution details.

```python
import requests

task_id = "your-task-uuid"
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()

if result['status'] == 'completed':
    print("Final Report:")
    print(result['report'])
    print("\nSteps executed:", len(result.get('steps', [])))
else:
    print(f"Task status: {result['status']}")
```

## Project Structure

```
.
├── main.py                    # FastAPI application, endpoints, DB models
├── src/
│   ├── planning_agent.py      # planner_agent() and executor_agent_step()
│   ├── agents.py              # research_agent, writer_agent, editor_agent
│   └── research_tools.py      # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup: Postgres + Uvicorn
├── requirements.txt           # Python dependencies
├── Dockerfile                 # Single-container build
└── .env                       # API keys (not committed)
```

## Agent Architecture

### Planning Agent

The planning agent receives a research prompt and breaks it down into structured steps.

```python
# Example from src/planning_agent.py
def planner_agent(prompt: str, model: str) -> dict:
    """
    Takes a research prompt and returns a structured plan with steps.
    Each step has: type (research/write/edit), description, tools needed.
    """
    # Uses LLM to generate plan
    plan = {
        "steps": [
            {"type": "research", "description": "Search for recent papers", "tools": ["arxiv", "tavily"]},
            {"type": "write", "description": "Draft initial report", "tools": []},
            {"type": "edit", "description": "Refine and fact-check", "tools": ["wikipedia"]}
        ]
    }
    return plan
```

### Executor Agents

Executor agents run each planned step using appropriate tools.

```python
# Example from src/agents.py
def research_agent(query: str, tools: list) -> str:
    """
    Executes research step using specified tools (tavily, arxiv, wikipedia).
    Returns aggregated research findings.
    """
    results = []
    
    if "tavily" in tools:
        results.append(tavily_search_tool(query))
    
    if "arxiv" in tools:
        results.append(arxiv_search_tool(query))
    
    if "wikipedia" in tools:
        results.append(wikipedia_search_tool(query))
    
    return "\n\n".join(results)

def writer_agent(research_data: str, topic: str) -> str:
    """
    Synthesizes research into a cohesive report draft.
    """
    # Uses LLM to generate report from research data
    pass

def editor_agent(draft: str, feedback: str = "") -> str:
    """
    Refines report for clarity, accuracy, and completeness.
    """
    # Uses LLM to edit and improve draft
    pass
```

### Research Tools

```python
# Example from src/research_tools.py
import os
import requests
import arxiv
import wikipedia

def tavily_search_tool(query: str) -> str:
    """Search web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    response = requests.post(
        "https://api.tavily.com/search",
        json={"query": query, "api_key": api_key}
    )
    return response.json().get("results", "No results found")

def arxiv_search_tool(query: str, max_results: int = 5) -> str:
    """Search arXiv for academic papers."""
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for paper in search.results():
        results.append(f"Title: {paper.title}\nSummary: {paper.summary}\n")
    
    return "\n---\n".join(results)

def wikipedia_search_tool(query: str) -> str:
    """Retrieve Wikipedia summary."""
    try:
        return wikipedia.summary(query, sentences=5)
    except wikipedia.exceptions.DisambiguationError as e:
        return f"Ambiguous query. Options: {e.options[:5]}"
    except wikipedia.exceptions.PageError:
        return "No Wikipedia page found."
```

## Configuration

### Environment Variables

```bash
# Required for LLM calls
OPENAI_API_KEY=sk-...

# Required for Tavily web search
TAVILY_API_KEY=tvly-...

# Database (auto-configured by entrypoint, override if needed)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional: prevent table drop on restart
RESET_DB_ON_STARTUP=0
```

### Custom Model Selection

The `/generate_report` endpoint accepts a `model` parameter:

```python
# Use GPT-4o
{"prompt": "...", "model": "openai:gpt-4o"}

# Use GPT-3.5
{"prompt": "...", "model": "openai:gpt-3.5-turbo"}
```

## Common Patterns

### Complete Research Workflow

```python
import requests
import time

API_BASE = "http://localhost:8000"

def run_research_workflow(prompt: str, model: str = "openai:gpt-4o"):
    """Execute a complete research workflow with progress tracking."""
    
    # 1. Start the task
    response = requests.post(
        f"{API_BASE}/generate_report",
        json={"prompt": prompt, "model": model}
    )
    task_id = response.json()["task_id"]
    print(f"✅ Task created: {task_id}")
    
    # 2. Poll for progress
    while True:
        progress = requests.get(f"{API_BASE}/task_progress/{task_id}").json()
        status = progress["status"]
        
        print(f"📊 Status: {status} | Step: {progress.get('current_step', 'N/A')}")
        
        if status == "completed":
            break
        elif status == "failed":
            print(f"❌ Task failed: {progress.get('error', 'Unknown error')}")
            return None
        
        time.sleep(3)
    
    # 3. Retrieve final report
    result = requests.get(f"{API_BASE}/task_status/{task_id}").json()
    return result["report"]

# Example usage
report = run_research_workflow(
    "How can transformer models improve drug discovery?"
)

if report:
    print("\n" + "="*60)
    print("FINAL REPORT")
    print("="*60)
    print(report)
```

### Direct Database Access

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Connect to the database
engine = create_engine("postgresql://app:local@localhost:5432/appdb")
Session = sessionmaker(bind=engine)
session = Session()

# Query task history
from main import Task  # Import Task model from main.py

recent_tasks = session.query(Task).order_by(Task.created_at.desc()).limit(10).all()

for task in recent_tasks:
    print(f"ID: {task.id}")
    print(f"Prompt: {task.prompt}")
    print(f"Status: {task.status}")
    print(f"Created: {task.created_at}")
    print("---")
```

### Custom Agent Implementation

```python
# Extend src/agents.py with a custom agent

def fact_checker_agent(report: str, sources: list) -> dict:
    """
    Verifies claims in the report against source material.
    Returns verification results with confidence scores.
    """
    import aisuite
    
    client = aisuite.Client()
    
    prompt = f"""
    Review this report and verify each major claim against the provided sources.
    
    Report:
    {report}
    
    Sources:
    {chr(10).join(sources)}
    
    Return JSON with: claim, verified (bool), confidence (0-1), source_ref
    """
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=[{"role": "user", "content": prompt}],
        response_format={"type": "json_object"}
    )
    
    return response.choices[0].message.content
```

## Troubleshooting

### Container Won't Start

**Symptom:** Container exits immediately or shows Postgres errors.

**Solution:**

```bash
# Check logs
docker logs fpsvc

# Ensure entrypoint script is executable
chmod +x docker/entrypoint.sh

# Verify Postgres port isn't already in use
lsof -i :5432
```

### Missing Templates/Static Files

**Symptom:** `TemplateNotFound` error when accessing `/`.

**Solution:** Verify Dockerfile copies templates:

```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

Check inside container:

```bash
docker exec -it fpsvc ls -la /app/templates
```

### API Key Errors

**Symptom:** `TAVILY_API_KEY not found` or OpenAI authentication errors.

**Solution:**

```bash
# Verify .env file exists and is passed to container
cat .env

# Check environment inside container
docker exec -it fpsvc env | grep API_KEY

# Restart with explicit environment variables
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -e OPENAI_API_KEY=sk-... \
  -e TAVILY_API_KEY=tvly-... \
  fastapi-postgres-service
```

### Database Tables Missing After Restart

**Symptom:** Tasks don't persist across container restarts.

**Solution:** The default `main.py` may drop tables on startup. Comment out:

```python
# In main.py
# Base.metadata.drop_all(bind=engine)  # Remove this line
Base.metadata.create_all(bind=engine)
```

Or use a persistent volume:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v postgres-data:/var/lib/postgresql/17/main \
  --env-file .env \
  fastapi-postgres-service
```

### Task Stuck in "running" Status

**Symptom:** Progress endpoint shows task running indefinitely.

**Solution:**

```bash
# Check application logs
docker logs -f fpsvc

# Verify task execution thread is running
# Look for exceptions in planner_agent or executor_agent_step

# Manually update task status if needed (emergency fix)
docker exec -it fpsvc psql -U app -d appdb -c \
  "UPDATE tasks SET status='failed' WHERE id='<task-uuid>';"
```

### Rate Limiting from External APIs

**Symptom:** Wikipedia or arXiv tools fail intermittently.

**Solution:** Implement retry logic with exponential backoff:

```python
import time
from functools import wraps

def retry_with_backoff(max_retries=3, backoff_factor=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    wait_time = backoff_factor ** attempt
                    print(f"Retry {attempt+1}/{max_retries} after {wait_time}s")
                    time.sleep(wait_time)
        return wrapper
    return decorator

@retry_with_backoff()
def wikipedia_search_tool(query: str) -> str:
    # existing implementation
    pass
```

## Development Tips

### Hot Reload for Development

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$(pwd)":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

### Interactive API Testing

Use the built-in FastAPI docs at http://localhost:8000/docs for interactive endpoint testing with Swagger UI.

### Direct Postgres Access

```bash
# From host machine
psql "postgresql://app:local@localhost:5432/appdb"

# Inside container
docker exec -it fpsvc psql -U app -d appdb
```

Useful queries:

```sql
-- View all tasks
SELECT id, prompt, status, created_at FROM tasks ORDER BY created_at DESC;

-- View task steps
SELECT task_id, step_number, step_type, status FROM task_steps WHERE task_id = '<uuid>';

-- Clear old tasks
DELETE FROM tasks WHERE created_at < NOW() - INTERVAL '7 days';
```
