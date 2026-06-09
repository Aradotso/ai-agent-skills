---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with multi-step workflow planning, tool integration (Tavily, arXiv, Wikipedia), and Postgres task tracking
triggers:
  - set up the agentic research agent service
  - use the deeplearning ai research workflow
  - create a multi-step research agent with planning
  - implement reflective research agent with FastAPI
  - integrate tavily arxiv wikipedia tools for research
  - build agentic workflow with task tracking
  - deploy research agent with postgres backend
  - configure multi-agent research pipeline
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic Research Agent is a FastAPI-based service that implements a multi-step research workflow using AI agents. It includes a planning agent that orchestrates research, writing, and editing tasks using external tools (Tavily search, arXiv papers, Wikipedia). All task state and results are stored in PostgreSQL for tracking and retrieval.

**Key capabilities:**
- Multi-step agent workflow (planner → research → writer → editor)
- Tool-using agents for web search, academic papers, and encyclopedia data
- Real-time progress tracking for long-running tasks
- Single-container Docker deployment with Postgres
- RESTful API with web UI

## Installation

### Prerequisites

Create a `.env` file with your API keys:

```bash
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
```

### Docker Deployment (Recommended)

```bash
# Clone the repository
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build the Docker image
docker build -t agentic-research-agent .

# Run the container
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name research-agent \
  --env-file .env \
  agentic-research-agent
```

### Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Start PostgreSQL separately, then set DATABASE_URL
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="your-openai-key"
export TAVILY_API_KEY="your-tavily-key"

# Run the FastAPI server
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## API Endpoints

### Start Research Task

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

### Poll Task Progress

```python
import requests
import time

def check_progress(task_id):
    response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    progress = response.json()
    
    print(f"Status: {progress['status']}")
    print(f"Current step: {progress.get('current_step', 'N/A')}")
    
    if progress.get('steps'):
        for step in progress['steps']:
            print(f"  - {step['name']}: {step['status']}")
    
    return progress['status']

# Poll until complete
while True:
    status = check_progress(task_id)
    if status in ['completed', 'failed']:
        break
    time.sleep(2)
```

### Get Final Report

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result['status'] == 'completed':
    print("Final Report:")
    print(result['result'])
else:
    print(f"Task failed: {result.get('error')}")
```

## Project Structure

```
.
├── main.py                      # FastAPI application entry point
├── src/
│   ├── planning_agent.py        # Planner and executor logic
│   ├── agents.py                # Research, writer, editor agents
│   └── research_tools.py        # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html               # Web UI
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
└── Dockerfile
```

## Core Components

### Planning Agent

The planning agent breaks down research tasks into subtasks:

```python
from src.planning_agent import planner_agent, executor_agent_step

# Generate a plan
plan = planner_agent(
    prompt="Research quantum computing applications in drug discovery",
    model="openai:gpt-4o"
)

# Execute each step
for step in plan['steps']:
    result = executor_agent_step(
        step=step,
        model="openai:gpt-4o",
        task_id="unique-task-id"
    )
    print(f"Step {step['name']}: {result}")
```

### Research Tools

The service provides three research tools:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Web search via Tavily
web_results = tavily_search_tool(
    query="latest developments in transformer models",
    max_results=5
)

# Academic papers from arXiv
papers = arxiv_search_tool(
    query="attention mechanisms neural networks",
    max_results=3
)

# Wikipedia articles
wiki_content = wikipedia_search_tool(
    query="artificial intelligence",
    sentences=10
)
```

### Custom Agent Implementation

Create a custom agent in `src/agents.py`:

```python
from typing import Dict, Any
import os
import aisuite as ai

client = ai.Client()

def custom_analysis_agent(
    data: str,
    model: str = "openai:gpt-4o"
) -> Dict[str, Any]:
    """
    Custom agent for specialized analysis
    """
    messages = [
        {
            "role": "system",
            "content": "You are an expert data analyst."
        },
        {
            "role": "user",
            "content": f"Analyze this data and provide insights:\n\n{data}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return {
        "analysis": response.choices[0].message.content,
        "model_used": model
    }
```

## Database Schema

The service uses PostgreSQL to track tasks:

```python
from sqlalchemy import Column, String, DateTime, Text, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    status = Column(String)  # pending, running, completed, failed
    prompt = Column(Text)
    model = Column(String)
    current_step = Column(String)
    result = Column(Text)
    error = Column(Text)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)

# Initialize database
DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
Base.metadata.create_all(bind=engine)
SessionLocal = sessionmaker(bind=engine)
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Database (defaults provided by entrypoint)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on startup
```

### Model Selection

The service supports multiple models via `aisuite`:

```python
# Available models
models = [
    "openai:gpt-4o",
    "openai:gpt-4-turbo",
    "openai:gpt-3.5-turbo",
    # Add other providers as configured in aisuite
]

# Specify in request
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Your research topic",
        "model": "openai:gpt-4o"  # Choose model here
    }
)
```

## Common Patterns

### Async Task Pattern

Handle long-running research tasks:

```python
import requests
import time
from typing import Optional

class ResearchClient:
    def __init__(self, base_url: str = "http://localhost:8000"):
        self.base_url = base_url
    
    def start_research(self, prompt: str, model: str = "openai:gpt-4o") -> str:
        response = requests.post(
            f"{self.base_url}/generate_report",
            json={"prompt": prompt, "model": model}
        )
        return response.json()["task_id"]
    
    def wait_for_completion(
        self,
        task_id: str,
        poll_interval: int = 3,
        timeout: int = 600
    ) -> Optional[dict]:
        start_time = time.time()
        
        while time.time() - start_time < timeout:
            response = requests.get(
                f"{self.base_url}/task_status/{task_id}"
            )
            result = response.json()
            
            if result["status"] == "completed":
                return result
            elif result["status"] == "failed":
                raise Exception(f"Task failed: {result.get('error')}")
            
            time.sleep(poll_interval)
        
        raise TimeoutError(f"Task {task_id} did not complete within {timeout}s")

# Usage
client = ResearchClient()
task_id = client.start_research("AI safety research overview")
result = client.wait_for_completion(task_id)
print(result["result"])
```

### Batch Research Pattern

Process multiple research topics:

```python
import asyncio
import aiohttp

async def research_topic(session, prompt: str, model: str):
    async with session.post(
        "http://localhost:8000/generate_report",
        json={"prompt": prompt, "model": model}
    ) as response:
        data = await response.json()
        return data["task_id"]

async def get_result(session, task_id: str):
    while True:
        async with session.get(
            f"http://localhost:8000/task_status/{task_id}"
        ) as response:
            data = await response.json()
            if data["status"] in ["completed", "failed"]:
                return data
        await asyncio.sleep(3)

async def batch_research(topics: list):
    async with aiohttp.ClientSession() as session:
        # Start all tasks
        task_ids = await asyncio.gather(*[
            research_topic(session, topic, "openai:gpt-4o")
            for topic in topics
        ])
        
        # Wait for all results
        results = await asyncio.gather(*[
            get_result(session, task_id)
            for task_id in task_ids
        ])
        
        return dict(zip(topics, results))

# Usage
topics = [
    "Machine learning in healthcare",
    "Quantum computing applications",
    "Renewable energy technologies"
]
results = asyncio.run(batch_research(topics))
```

## Troubleshooting

### Database Connection Issues

```bash
# Check if Postgres is running
docker exec -it research-agent bash -lc "pg_lsclusters"

# Test database connection
docker exec -it research-agent bash -lc "psql postgresql://app:local@127.0.0.1:5432/appdb -c 'SELECT 1;'"

# View database logs
docker logs research-agent | grep postgres
```

### API Key Errors

```python
# Verify API keys are loaded
import os
from dotenv import load_dotenv

load_dotenv()

assert os.getenv("OPENAI_API_KEY"), "OPENAI_API_KEY not set"
assert os.getenv("TAVILY_API_KEY"), "TAVILY_API_KEY not set"
```

### Task Stuck in Running State

```python
# Manually check task state in database
import psycopg2
import os

conn = psycopg2.connect(os.getenv("DATABASE_URL"))
cur = conn.cursor()

# Find stuck tasks
cur.execute("""
    SELECT id, status, current_step, updated_at 
    FROM tasks 
    WHERE status = 'running' 
    AND updated_at < NOW() - INTERVAL '10 minutes'
""")

stuck_tasks = cur.fetchall()
print("Stuck tasks:", stuck_tasks)

# Reset if needed
for task_id, *_ in stuck_tasks:
    cur.execute(
        "UPDATE tasks SET status = 'failed', error = 'Timeout' WHERE id = %s",
        (task_id,)
    )

conn.commit()
conn.close()
```

### Memory Issues with Large Reports

```python
# Stream large results instead of loading all at once
def stream_task_result(task_id: str):
    response = requests.get(
        f"http://localhost:8000/task_status/{task_id}",
        stream=True
    )
    
    for chunk in response.iter_content(chunk_size=8192):
        if chunk:
            yield chunk
```

### Rate Limiting

```python
# Implement exponential backoff for API calls
import time
from functools import wraps

def retry_with_backoff(retries=3, backoff_in_seconds=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            x = 0
            while True:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if x == retries:
                        raise
                    wait = backoff_in_seconds * 2 ** x
                    time.sleep(wait)
                    x += 1
        return wrapper
    return decorator

@retry_with_backoff(retries=3, backoff_in_seconds=2)
def call_research_api(prompt: str):
    return requests.post(
        "http://localhost:8000/generate_report",
        json={"prompt": prompt, "model": "openai:gpt-4o"}
    )
```

## Web UI Access

Access the built-in web interface at `http://localhost:8000/` to:
- Submit research tasks through a form
- View real-time progress updates
- Download completed reports

The UI provides a user-friendly alternative to the API for manual research tasks.
