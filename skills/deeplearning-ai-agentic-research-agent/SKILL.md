---
name: deeplearning-ai-agentic-research-agent
description: Multi-agent research workflow system using FastAPI, PostgreSQL, and tool-calling agents (Tavily, arXiv, Wikipedia) for automated research report generation
triggers:
  - set up the agentic research agent service
  - how do I use the DeepLearning.AI research agent
  - create a multi-agent research workflow
  - integrate Tavily and arXiv search tools
  - build a reflective research agent with FastAPI
  - deploy the agentic AI research service
  - implement tool-using research agents
  - track multi-step agent task progress
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The DeepLearning.AI Agentic AI Research Agent is a FastAPI-based service that orchestrates multi-agent research workflows. It uses a planning agent to decompose research tasks, then executes research through specialized agents (research, writer, editor) that leverage external tools like Tavily, arXiv, and Wikipedia. Task state and results are stored in PostgreSQL, with real-time progress tracking via API endpoints.

**Key Features:**
- Multi-step agent workflow (planner → research → writer → editor)
- Tool integration: Tavily search, arXiv papers, Wikipedia
- PostgreSQL-backed task state management
- Real-time progress tracking via REST API
- Docker-based deployment (Postgres + API in single container)
- Web UI for submitting research tasks

## Installation

### Prerequisites

- Docker (Desktop or engine)
- API keys: OpenAI and Tavily

### Setup

1. **Clone the repository:**

```bash
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file in project root:**

```bash
# .env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
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

5. **Access the application:**
   - Web UI: http://localhost:8000/
   - API docs: http://localhost:8000/docs

## Project Structure

```
.
├── main.py                    # FastAPI app with routes
├── src/
│   ├── planning_agent.py      # Planner & executor logic
│   ├── agents.py              # Research/writer/editor agents
│   └── research_tools.py      # Tool implementations
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container startup script
├── requirements.txt           # Python dependencies
├── Dockerfile
└── README.md
```

## API Reference

### Generate Research Report

**Endpoint:** `POST /generate_report`

Kicks off a threaded multi-step research workflow.

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

**cURL example:**

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Quantum computing applications in cryptography",
    "model": "openai:gpt-4o"
  }'
```

### Track Task Progress

**Endpoint:** `GET /task_progress/{task_id}`

Returns live status of each workflow step and substep.

```python
import requests
import time

def poll_progress(task_id):
    url = f"http://localhost:8000/task_progress/{task_id}"
    
    while True:
        response = requests.get(url)
        data = response.json()
        
        print(f"Status: {data['status']}")
        print(f"Current Step: {data.get('current_step', 'N/A')}")
        
        if data['status'] in ['completed', 'failed']:
            break
            
        time.sleep(2)
    
    return data

progress = poll_progress(task_id)
```

### Get Final Results

**Endpoint:** `GET /task_status/{task_id}`

Retrieves final status and generated report.

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result['status'] == 'completed':
    print("Research Report:")
    print(result['report'])
else:
    print(f"Task failed: {result.get('error')}")
```

## Core Components

### Planning Agent

The planning agent (`src/planning_agent.py`) decomposes research tasks into actionable steps.

```python
from src.planning_agent import planner_agent

# Define research task
task = "Research the impact of transformer models on NLP"

# Generate plan
plan = planner_agent(task, model="openai:gpt-4o")

# Plan structure:
# {
#   "steps": [
#     {"step": 1, "action": "research", "description": "..."},
#     {"step": 2, "action": "write", "description": "..."},
#     {"step": 3, "action": "edit", "description": "..."}
#   ]
# }
```

### Research Tools

The system includes three primary research tools in `src/research_tools.py`:

**Tavily Search:**

```python
from src.research_tools import tavily_search_tool

results = tavily_search_tool(
    query="latest developments in large language models",
    max_results=5
)

for result in results:
    print(f"Title: {result['title']}")
    print(f"URL: {result['url']}")
    print(f"Content: {result['content']}")
```

**arXiv Search:**

```python
from src.research_tools import arxiv_search_tool

papers = arxiv_search_tool(
    query="transformer attention mechanisms",
    max_results=3
)

for paper in papers:
    print(f"Title: {paper['title']}")
    print(f"Authors: {', '.join(paper['authors'])}")
    print(f"Summary: {paper['summary']}")
```

**Wikipedia Search:**

```python
from src.research_tools import wikipedia_search_tool

wiki_content = wikipedia_search_tool(
    query="artificial intelligence",
    sentences=5
)

print(wiki_content)
```

### Agent Execution

Execute individual agent steps with context:

```python
from src.planning_agent import executor_agent_step

# Execute research step
step_result = executor_agent_step(
    step={
        "step": 1,
        "action": "research",
        "description": "Search for recent papers on transformers"
    },
    context={
        "previous_results": [],
        "tools_available": ["tavily", "arxiv", "wikipedia"]
    },
    model="openai:gpt-4o"
)

print(f"Step output: {step_result['output']}")
```

## Database Schema

The service uses PostgreSQL to track task state:

```sql
-- Task tracking table
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    prompt TEXT NOT NULL,
    status VARCHAR(50) NOT NULL,
    current_step INTEGER,
    plan JSONB,
    results JSONB,
    report TEXT,
    error TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Accessing the database:**

```bash
# From host machine
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT id, prompt, status, current_step, created_at 
FROM tasks 
ORDER BY created_at DESC 
LIMIT 10;
```

## Configuration

### Environment Variables

**Required:**
- `OPENAI_API_KEY` - OpenAI API key for LLM calls
- `TAVILY_API_KEY` - Tavily API key for web search

**Optional:**
- `DATABASE_URL` - PostgreSQL connection string (default: `postgresql://app:local@127.0.0.1:5432/appdb`)
- `POSTGRES_USER` - DB user (default: `app`)
- `POSTGRES_PASSWORD` - DB password (default: `local`)
- `POSTGRES_DB` - Database name (default: `appdb`)
- `RESET_DB_ON_STARTUP` - Set to `1` to drop/recreate tables on startup

### Custom Model Configuration

```python
# In your code, specify different models
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Your research topic",
        "model": "openai:gpt-4-turbo"  # Or other aisuite-supported models
    }
)
```

## Common Patterns

### Complete Research Workflow

```python
import requests
import time

class ResearchClient:
    def __init__(self, base_url="http://localhost:8000"):
        self.base_url = base_url
    
    def start_research(self, prompt, model="openai:gpt-4o"):
        response = requests.post(
            f"{self.base_url}/generate_report",
            json={"prompt": prompt, "model": model}
        )
        return response.json()["task_id"]
    
    def wait_for_completion(self, task_id, poll_interval=3):
        while True:
            response = requests.get(
                f"{self.base_url}/task_progress/{task_id}"
            )
            data = response.json()
            
            if data["status"] in ["completed", "failed"]:
                break
            
            print(f"Progress: {data.get('current_step', 'N/A')}")
            time.sleep(poll_interval)
        
        return self.get_results(task_id)
    
    def get_results(self, task_id):
        response = requests.get(
            f"{self.base_url}/task_status/{task_id}"
        )
        return response.json()

# Usage
client = ResearchClient()
task_id = client.start_research(
    "Impact of AI on healthcare diagnostics"
)
results = client.wait_for_completion(task_id)
print(results["report"])
```

### Custom Tool Integration

Add new tools to `src/research_tools.py`:

```python
import requests

def custom_api_tool(query: str, **kwargs):
    """
    Custom research tool example
    """
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query},
        headers={"Authorization": f"Bearer {os.getenv('CUSTOM_API_KEY')}"}
    )
    
    return response.json()

# Register in agents.py
from src.research_tools import custom_api_tool

def research_agent_with_custom_tool(task, context):
    tools = [tavily_search_tool, arxiv_search_tool, custom_api_tool]
    # ... agent implementation
```

### Batch Research Tasks

```python
import asyncio
import aiohttp

async def submit_task(session, prompt):
    async with session.post(
        "http://localhost:8000/generate_report",
        json={"prompt": prompt, "model": "openai:gpt-4o"}
    ) as response:
        data = await response.json()
        return data["task_id"]

async def batch_research(prompts):
    async with aiohttp.ClientSession() as session:
        tasks = [submit_task(session, p) for p in prompts]
        task_ids = await asyncio.gather(*tasks)
        return task_ids

# Usage
prompts = [
    "Machine learning in drug discovery",
    "Renewable energy storage solutions",
    "Quantum computing algorithms"
]

task_ids = asyncio.run(batch_research(prompts))
print(f"Started {len(task_ids)} research tasks")
```

## Troubleshooting

### Container Issues

**Problem:** Container fails to start or exits immediately

```bash
# Check logs
docker logs fpsvc

# Verify environment variables
docker exec -it fpsvc env | grep -E '(OPENAI|TAVILY|DATABASE)'

# Check Postgres status
docker exec -it fpsvc bash -lc "pg_lsclusters"
```

**Problem:** Database connection errors

```bash
# Verify DATABASE_URL
docker exec -it fpsvc bash -lc 'echo $DATABASE_URL'

# Test connection
docker exec -it fpsvc bash -lc "psql $DATABASE_URL -c 'SELECT 1;'"
```

### API Errors

**Problem:** 500 errors when submitting tasks

Check for missing API keys:

```python
import os
from dotenv import load_dotenv

load_dotenv()

required_keys = ["OPENAI_API_KEY", "TAVILY_API_KEY"]
missing = [k for k in required_keys if not os.getenv(k)]

if missing:
    print(f"Missing API keys: {', '.join(missing)}")
```

**Problem:** Tasks stuck in "running" state

```bash
# Check task details
curl http://localhost:8000/task_progress/{task_id}

# Inspect database
docker exec -it fpsvc psql postgresql://app:local@127.0.0.1:5432/appdb \
  -c "SELECT id, status, current_step, error FROM tasks WHERE id = '{task_id}';"
```

### Tool-Specific Issues

**Tavily rate limits:**

```python
# Add retry logic
import time
from functools import wraps

def retry_on_rate_limit(max_retries=3, delay=5):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if "rate limit" in str(e).lower() and attempt < max_retries - 1:
                        time.sleep(delay * (attempt + 1))
                        continue
                    raise
        return wrapper
    return decorator

@retry_on_rate_limit()
def safe_tavily_search(query):
    return tavily_search_tool(query)
```

**Wikipedia timeouts:**

```python
# Set timeout in requests
import requests

def wikipedia_search_with_timeout(query, timeout=10):
    try:
        response = requests.get(
            f"https://en.wikipedia.org/api/rest_v1/page/summary/{query}",
            timeout=timeout
        )
        return response.json()
    except requests.Timeout:
        print(f"Wikipedia search timed out for: {query}")
        return None
```

### Development Mode

Run with hot reload for development:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "
    pg_ctlcluster \$(psql -V | awk '{print \$3}' | cut -d. -f1) main start && \
    uvicorn main:app --host 0.0.0.0 --port 8000 --reload
  "
```

### Reset Database

```python
# In main.py, add conditional reset
import os
from sqlalchemy import create_engine
from models import Base

engine = create_engine(os.getenv("DATABASE_URL"))

if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
    print("⚠️  Database tables dropped")

Base.metadata.create_all(bind=engine)
```

Then run:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  --env-file .env \
  -e RESET_DB_ON_STARTUP=1 \
  --name fpsvc \
  fastapi-postgres-service
```
