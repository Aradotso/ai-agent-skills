---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with multi-step planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres state management for agentic workflows
triggers:
  - set up the agentic research agent service
  - create a research workflow with planning agents
  - use the DeepLearning.AI research agent API
  - implement multi-step agent workflows with Tavily and arXiv
  - build a reflective research agent with FastAPI
  - configure the agentic AI research service
  - run research tasks with planning and execution agents
  - integrate Wikipedia, arXiv, and Tavily search tools
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The DeepLearning.AI Agentic Research Agent is a FastAPI-based service that orchestrates multi-step research workflows using AI agents. It implements a reflective agent pattern with planning, research, writing, and editing phases. The system uses Postgres for state management and integrates multiple search tools (Tavily, arXiv, Wikipedia) to gather comprehensive research data.

**Key Features:**
- Multi-agent workflow with planner, researcher, writer, and editor agents
- Tool-using agents for web search (Tavily), academic papers (arXiv), and general knowledge (Wikipedia)
- Async task execution with progress tracking
- Postgres-backed state persistence
- Web UI for task submission and monitoring

## Installation

### Docker Setup (Recommended)

1. **Clone the repository:**

```bash
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file:**

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

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set environment variables
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="your-openai-key"
export TAVILY_API_KEY="your-tavily-key"

# Run the service
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├─ main.py                      # FastAPI app with endpoints
├─ src/
│  ├─ planning_agent.py         # Planner and executor agents
│  ├─ agents.py                 # Research, writer, editor agents
│  └─ research_tools.py         # Tavily, arXiv, Wikipedia tools
├─ templates/
│  └─ index.html                # Web UI
├─ static/                      # CSS/JS assets
├─ docker/
│  └─ entrypoint.sh             # Container startup script
├─ requirements.txt
├─ Dockerfile
└─ README.md
```

## API Usage

### Starting a Research Task

**Endpoint:** `POST /generate_report`

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
print(f"Task ID: {task_id}")
```

**Using curl:**

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications in drug discovery", "model":"openai:gpt-4o"}'
```

### Monitoring Task Progress

**Endpoint:** `GET /task_progress/{task_id}`

```python
import requests
import time

task_id = "your-task-uuid"

while True:
    response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    progress = response.json()
    
    print(f"Status: {progress['status']}")
    print(f"Current Step: {progress.get('current_step', 'N/A')}")
    
    if progress['status'] in ['completed', 'failed']:
        break
    
    time.sleep(2)
```

### Retrieving Final Report

**Endpoint:** `GET /task_status/{task_id}`

```python
import requests

task_id = "your-task-uuid"
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result['status'] == 'completed':
    print("Final Report:")
    print(result['report'])
else:
    print(f"Task status: {result['status']}")
```

## Core Components

### Planning Agent

The planning agent breaks down research queries into executable steps:

```python
# src/planning_agent.py example usage
from src.planning_agent import planner_agent, executor_agent_step

# Generate a research plan
plan = planner_agent(
    query="Analyze the impact of transformers on NLP",
    model="openai:gpt-4o"
)

# Execute each step
for step in plan['steps']:
    result = executor_agent_step(
        step=step,
        context=plan['context'],
        model="openai:gpt-4o"
    )
    print(f"Step completed: {result}")
```

### Research Tools

#### Tavily Search (Web Search)

```python
from src.research_tools import tavily_search_tool

results = tavily_search_tool(
    query="latest advances in reinforcement learning",
    max_results=5
)

for result in results:
    print(f"Title: {result['title']}")
    print(f"URL: {result['url']}")
    print(f"Snippet: {result['content']}\n")
```

#### arXiv Search (Academic Papers)

```python
from src.research_tools import arxiv_search_tool

papers = arxiv_search_tool(
    query="transformer architectures",
    max_results=10
)

for paper in papers:
    print(f"Title: {paper['title']}")
    print(f"Authors: {', '.join(paper['authors'])}")
    print(f"Abstract: {paper['summary']}\n")
```

#### Wikipedia Search

```python
from src.research_tools import wikipedia_search_tool

info = wikipedia_search_tool(
    query="Machine Learning",
    sentences=3
)

print(info['summary'])
print(f"Full URL: {info['url']}")
```

### Agent Implementation Pattern

```python
from src.agents import research_agent, writer_agent, editor_agent

# Research phase
research_results = research_agent(
    topic="Neural Architecture Search",
    tools=["tavily", "arxiv", "wikipedia"],
    model="openai:gpt-4o"
)

# Writing phase
draft_report = writer_agent(
    research_data=research_results,
    outline=plan['outline'],
    model="openai:gpt-4o"
)

# Editing phase
final_report = editor_agent(
    draft=draft_report,
    criteria=["clarity", "accuracy", "completeness"],
    model="openai:gpt-4o"
)
```

## Database Schema

The service uses Postgres to track task state:

```python
from sqlalchemy import Column, String, DateTime, Text, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

Base = declarative_base()

class ResearchTask(Base):
    __tablename__ = 'research_tasks'
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    status = Column(String, default='pending')
    current_step = Column(String)
    report = Column(Text)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)

# Connect to database
DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...                    # OpenAI API key
TAVILY_API_KEY=tvly-...                  # Tavily search API key

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional
POSTGRES_USER=app                         # Postgres username
POSTGRES_PASSWORD=local                   # Postgres password
POSTGRES_DB=appdb                         # Database name
RESET_DB_ON_STARTUP=0                    # Set to 1 to reset DB on start
```

### Model Selection

The service supports multiple AI models through `aisuite`:

```python
# Available models
models = [
    "openai:gpt-4o",
    "openai:gpt-4-turbo",
    "openai:gpt-3.5-turbo",
    # Add other aisuite-compatible models
]

# Specify in request
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Your research query",
        "model": "openai:gpt-4o"  # Choose model
    }
)
```

## Common Workflows

### Complete Research Pipeline

```python
import requests
import time

def run_research_task(prompt, model="openai:gpt-4o"):
    # Start task
    response = requests.post(
        "http://localhost:8000/generate_report",
        json={"prompt": prompt, "model": model}
    )
    task_id = response.json()["task_id"]
    
    # Poll for completion
    while True:
        status_response = requests.get(
            f"http://localhost:8000/task_progress/{task_id}"
        )
        status = status_response.json()
        
        print(f"Step: {status.get('current_step', 'initializing')}")
        
        if status['status'] == 'completed':
            break
        elif status['status'] == 'failed':
            raise Exception("Task failed")
        
        time.sleep(3)
    
    # Get final report
    final = requests.get(
        f"http://localhost:8000/task_status/{task_id}"
    )
    return final.json()['report']

# Use it
report = run_research_task(
    "Analyze recent breakthroughs in protein folding prediction"
)
print(report)
```

### Web UI Usage

1. Navigate to `http://localhost:8000/`
2. Enter your research query
3. Select model
4. Click "Generate Report"
5. Monitor progress in real-time
6. Download or copy the final report

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs fpsvc

# Verify environment variables
docker exec -it fpsvc env | grep -E 'OPENAI|TAVILY|DATABASE'

# Check Postgres
docker exec -it fpsvc su -s /bin/bash postgres -c "psql -c '\l'"
```

### Database Connection Issues

```python
# Test database connection
from sqlalchemy import create_engine
import os

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)

try:
    connection = engine.connect()
    print("✅ Database connected")
    connection.close()
except Exception as e:
    print(f"❌ Database error: {e}")
```

### API Key Errors

```bash
# Verify keys are loaded
docker exec -it fpsvc bash -c 'echo $OPENAI_API_KEY | cut -c1-7'
docker exec -it fpsvc bash -c 'echo $TAVILY_API_KEY | cut -c1-7'

# Reload with env file
docker run --rm -it -p 8000:8000 --env-file .env fastapi-postgres-service
```

### Tool Integration Issues

```python
# Test Tavily connection
from src.research_tools import tavily_search_tool

try:
    results = tavily_search_tool("test query", max_results=1)
    print("✅ Tavily working")
except Exception as e:
    print(f"❌ Tavily error: {e}")

# Test arXiv
from src.research_tools import arxiv_search_tool

try:
    papers = arxiv_search_tool("test", max_results=1)
    print("✅ arXiv working")
except Exception as e:
    print(f"❌ arXiv error: {e}")
```

### Rate Limiting

```python
import time

def retry_with_backoff(func, max_retries=3):
    for i in range(max_retries):
        try:
            return func()
        except Exception as e:
            if "rate limit" in str(e).lower():
                wait_time = 2 ** i
                print(f"Rate limited, waiting {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise
    raise Exception("Max retries exceeded")

# Use with API calls
result = retry_with_backoff(
    lambda: tavily_search_tool("query", max_results=5)
)
```

## Advanced Usage

### Custom Agent Configuration

```python
# Configure agents with custom parameters
agent_config = {
    "planner": {
        "model": "openai:gpt-4o",
        "max_steps": 5,
        "temperature": 0.7
    },
    "researcher": {
        "tools": ["tavily", "arxiv"],
        "max_results_per_tool": 10
    },
    "writer": {
        "style": "academic",
        "word_count_target": 2000
    },
    "editor": {
        "iterations": 2,
        "focus": ["clarity", "citations"]
    }
}
```

### Direct Database Access

```bash
# Connect from host
psql "postgresql://app:local@localhost:5432/appdb"

# Query tasks
SELECT task_id, status, current_step, created_at 
FROM research_tasks 
ORDER BY created_at DESC 
LIMIT 10;

# Get task details
SELECT * FROM research_tasks WHERE task_id = 'your-uuid';
```

### Batch Processing

```python
import asyncio
import aiohttp

async def submit_task(session, prompt):
    async with session.post(
        "http://localhost:8000/generate_report",
        json={"prompt": prompt, "model": "openai:gpt-4o"}
    ) as resp:
        return await resp.json()

async def batch_research(prompts):
    async with aiohttp.ClientSession() as session:
        tasks = [submit_task(session, p) for p in prompts]
        return await asyncio.gather(*tasks)

# Submit multiple research tasks
prompts = [
    "AI in healthcare",
    "Quantum computing applications",
    "Climate change solutions"
]

results = asyncio.run(batch_research(prompts))
for result in results:
    print(f"Started task: {result['task_id']}")
```
