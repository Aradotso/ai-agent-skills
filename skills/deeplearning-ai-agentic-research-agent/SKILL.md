---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service using Tavily, arXiv, and Wikipedia with Postgres storage for agentic workflows
triggers:
  - how do I use the agentic research agent
  - set up the reflective research agent service
  - run a research task with planning agent
  - query the research agent API
  - build the FastAPI research workflow
  - configure the agentic AI research service
  - track research task progress
  - use Tavily and arXiv tools in research agent
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

The **Reflective Research Agent** is a FastAPI web service that orchestrates multi-step research workflows using AI agents. It combines a planning agent with specialized research, writing, and editing agents that use tools like Tavily search, arXiv, and Wikipedia. All task state and results are stored in Postgres, with live progress tracking via REST endpoints.

## What It Does

- **Planning Agent**: Breaks down research queries into structured subtasks
- **Executor Agents**: Research, writer, and editor agents with tool access
- **Research Tools**: Tavily web search, arXiv papers, Wikipedia queries
- **Task Management**: Postgres-backed state tracking with live progress updates
- **Web UI + API**: Jinja2 template UI and REST endpoints for task submission/monitoring

## Installation

### Docker (Recommended)

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
docker build -t fastapi-postgres-service .

# Run the container (Postgres + API in one)
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
export DATABASE_URL="postgresql://user:pass@localhost:5432/appdb"
export OPENAI_API_KEY="your-openai-key"
export TAVILY_API_KEY="your-tavily-key"

# Run the FastAPI server
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Key API Endpoints

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
print(f"Task started: {task_id}")
```

### Check Task Progress

```python
import requests
import time

def poll_progress(task_id):
    while True:
        resp = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        data = resp.json()
        
        print(f"Status: {data['status']}")
        print(f"Current step: {data.get('current_step', 'N/A')}")
        
        if data["status"] in ["completed", "error"]:
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
    print("Final Report:")
    print(result["report"])
else:
    print(f"Task failed: {result.get('error', 'Unknown error')}")
```

## Project Structure

```
agentic-ai-public/
├── main.py                    # FastAPI app with endpoints
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

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key for LLM calls
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Optional (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on startup
```

### Database Setup

The Docker container auto-configures Postgres. For local setup:

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Create tables
Base.metadata.create_all(bind=engine)
```

## Using Research Tools

### Tavily Search Tool

```python
from src.research_tools import tavily_search_tool

results = tavily_search_tool("quantum computing breakthroughs 2024")
# Returns: List of search results with titles, URLs, and snippets
```

### arXiv Search Tool

```python
from src.research_tools import arxiv_search_tool

papers = arxiv_search_tool("transformer architecture", max_results=5)
# Returns: List of academic papers with abstracts and metadata
```

### Wikipedia Search Tool

```python
from src.research_tools import wikipedia_search_tool

summary = wikipedia_search_tool("artificial intelligence")
# Returns: Wikipedia page summary
```

## Creating Custom Agents

### Research Agent Example

```python
from src.agents import research_agent
from src.research_tools import tavily_search_tool, arxiv_search_tool

def custom_research_task(query):
    # Research agent uses tools to gather information
    context = {
        "query": query,
        "tools": [tavily_search_tool, arxiv_search_tool]
    }
    
    result = research_agent(context)
    return result
```

### Writer Agent Pattern

```python
from src.agents import writer_agent

def generate_report_section(research_data, section_title):
    context = {
        "research": research_data,
        "section": section_title,
        "style": "academic"
    }
    
    written_content = writer_agent(context)
    return written_content
```

### Editor Agent Workflow

```python
from src.agents import editor_agent

def refine_content(draft, requirements):
    context = {
        "draft": draft,
        "requirements": requirements,
        "tone": "professional"
    }
    
    edited = editor_agent(context)
    return edited
```

## Planning Agent Usage

```python
from src.planning_agent import planner_agent, executor_agent_step

# Generate a plan
plan = planner_agent("Research recent advances in quantum computing")
# Returns: Structured plan with steps and substeps

# Execute plan steps
for step in plan["steps"]:
    result = executor_agent_step(
        step_description=step["description"],
        step_type=step["type"],  # "research", "write", or "edit"
        context={}
    )
    print(f"Completed: {step['description']}")
```

## Web UI Integration

Access the web interface at `http://localhost:8000/`:

```html
<!-- templates/index.html example usage -->
<form id="research-form">
  <input type="text" name="prompt" placeholder="Enter research topic">
  <select name="model">
    <option value="openai:gpt-4o">GPT-4</option>
    <option value="openai:gpt-3.5-turbo">GPT-3.5</option>
  </select>
  <button type="submit">Generate Report</button>
</form>

<script>
document.getElementById('research-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const formData = new FormData(e.target);
  const response = await fetch('/generate_report', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify(Object.fromEntries(formData))
  });
  const {task_id} = await response.json();
  // Poll progress...
});
</script>
```

## Common Patterns

### Full Workflow Execution

```python
import requests
import time

def execute_research_workflow(topic, model="openai:gpt-4o"):
    # 1. Start task
    resp = requests.post(
        "http://localhost:8000/generate_report",
        json={"prompt": topic, "model": model}
    )
    task_id = resp.json()["task_id"]
    
    # 2. Poll until completion
    while True:
        status = requests.get(
            f"http://localhost:8000/task_progress/{task_id}"
        ).json()
        
        if status["status"] == "completed":
            break
        elif status["status"] == "error":
            raise Exception(status.get("error"))
        
        time.sleep(3)
    
    # 3. Retrieve final report
    result = requests.get(
        f"http://localhost:8000/task_status/{task_id}"
    ).json()
    
    return result["report"]

# Use it
report = execute_research_workflow(
    "Impact of AI on healthcare diagnostics"
)
print(report)
```

### Custom Tool Integration

```python
def custom_tool(query: str) -> str:
    """Add your own research tool"""
    # Your custom logic here
    return f"Results for: {query}"

# Register in research_tools.py
def get_available_tools():
    return [
        tavily_search_tool,
        arxiv_search_tool,
        wikipedia_search_tool,
        custom_tool  # Add your tool
    ]
```

### Database Query Example

```python
from sqlalchemy.orm import Session
from main import SessionLocal, Task

def get_recent_tasks(limit=10):
    db: Session = SessionLocal()
    try:
        tasks = db.query(Task)\
            .order_by(Task.created_at.desc())\
            .limit(limit)\
            .all()
        return [{"id": t.id, "status": t.status} for t in tasks]
    finally:
        db.close()
```

## Troubleshooting

### Tables Not Persisting

```python
# In main.py, disable table dropping on startup
import os

# Comment out or guard with env var
if os.getenv("RESET_DB_ON_STARTUP") != "1":
    # Don't drop tables
    pass
else:
    Base.metadata.drop_all(bind=engine)

Base.metadata.create_all(bind=engine)
```

### API Key Errors

```bash
# Verify environment variables are loaded
docker exec -it fpsvc bash -lc "env | grep API_KEY"

# Check .env file format (no spaces around =)
cat .env
```

### Postgres Connection Issues

```bash
# Check if Postgres is running in container
docker exec -it fpsvc pg_lsclusters

# Connect to database directly
docker exec -it fpsvc psql -U app -d appdb

# Verify DATABASE_URL
docker exec -it fpsvc bash -lc "echo \$DATABASE_URL"
```

### Tool Rate Limiting

```python
import time
from functools import wraps

def rate_limit(calls=5, period=60):
    def decorator(func):
        calls_made = []
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            calls_made[:] = [c for c in calls_made if c > now - period]
            
            if len(calls_made) >= calls:
                sleep_time = period - (now - calls_made[0])
                time.sleep(sleep_time)
            
            calls_made.append(time.time())
            return func(*args, **kwargs)
        
        return wrapper
    return decorator

@rate_limit(calls=3, period=60)
def wikipedia_search_tool(query):
    # Tool implementation
    pass
```

### Container Logs

```bash
# View real-time logs
docker logs -f fpsvc

# Check startup sequence
docker logs fpsvc | grep -E "Starting|ready|ERROR"

# Debug database initialization
docker logs fpsvc | grep -E "CREATE|DATABASE"
```

### Hot Reload for Development

```bash
# Mount code directory for live editing
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```
