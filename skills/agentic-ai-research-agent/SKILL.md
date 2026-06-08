---
name: agentic-ai-research-agent
description: FastAPI research agent service with planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres state management
triggers:
  - set up the agentic research agent service
  - run a research workflow with planning agents
  - create a multi-step research task with tavily and arxiv
  - implement reflective research agent with fastapi
  - build an agent workflow with planning and execution
  - configure the research agent postgres backend
  - generate a research report using agentic ai
  - debug the fastapi research agent service
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research Agent is a FastAPI service that orchestrates multi-step research workflows using AI agents. It features:

- **Planning Agent**: Breaks down research tasks into executable steps
- **Executor Agents**: Research, writer, and editor agents that use tools
- **Research Tools**: Integration with Tavily search, arXiv papers, and Wikipedia
- **State Management**: PostgreSQL tracks task progress and results
- **Async Execution**: Threaded workflow execution with live progress tracking
- **Web UI**: Simple interface to kick off research tasks

The system uses a reflective workflow pattern where agents plan, execute, and refine research outputs iteratively.

## Installation

### Docker Setup (Recommended)

1. **Clone the repository:**

```bash
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

2. **Create `.env` file with API keys:**

```bash
# .env
OPENAI_API_KEY=your-openai-key-here
TAVILY_API_KEY=your-tavily-key-here
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

The container runs both PostgreSQL and the FastAPI service in a single container for development.

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up PostgreSQL separately
export DATABASE_URL="postgresql://user:password@localhost:5432/appdb"

# Run the service
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                      # FastAPI app with routes
├── src/
│   ├── planning_agent.py        # Planner and executor logic
│   ├── agents.py                # Research, writer, editor agents
│   └── research_tools.py        # Tool implementations
├── templates/
│   └── index.html               # Web UI
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh           # Container startup script
├── requirements.txt
└── Dockerfile
```

## Core Concepts

### Task Lifecycle

1. **Submit** → Task created with UUID
2. **Planning** → Planner agent breaks down research query
3. **Execution** → Each step runs through appropriate agent (research/writer/editor)
4. **Completion** → Final report generated and stored

### Database Schema

The service uses SQLAlchemy models:

```python
from sqlalchemy import Column, String, Text, DateTime, Integer
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class TaskStatus(Base):
    __tablename__ = "task_status"
    task_id = Column(String, primary_key=True)
    status = Column(String)  # "running", "completed", "failed"
    prompt = Column(Text)
    model = Column(String)
    final_report = Column(Text, nullable=True)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)

class TaskProgress(Base):
    __tablename__ = "task_progress"
    id = Column(Integer, primary_key=True, autoincrement=True)
    task_id = Column(String)
    step_number = Column(Integer)
    step_name = Column(String)
    status = Column(String)
    result = Column(Text, nullable=True)
    timestamp = Column(DateTime)
```

## API Reference

### POST `/generate_report`

Kick off a research task:

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

**Request Body:**
```json
{
  "prompt": "Research topic or question",
  "model": "openai:gpt-4o"
}
```

**Response:**
```json
{
  "task_id": "uuid-string"
}
```

### GET `/task_progress/{task_id}`

Poll for live progress updates:

```python
import requests
import time

task_id = "your-task-id"

while True:
    response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    data = response.json()
    
    print(f"Status: {data['status']}")
    for step in data['progress']:
        print(f"  Step {step['step_number']}: {step['step_name']} - {step['status']}")
    
    if data['status'] in ['completed', 'failed']:
        break
    
    time.sleep(2)
```

**Response:**
```json
{
  "task_id": "uuid",
  "status": "running",
  "prompt": "Research question",
  "progress": [
    {
      "step_number": 1,
      "step_name": "Planning",
      "status": "completed",
      "result": "Step output",
      "timestamp": "2025-01-01T12:00:00"
    }
  ]
}
```

### GET `/task_status/{task_id}`

Get final status and report:

```python
import requests

response = requests.get(f"http://localhost:8000/task_status/{task_id}")
data = response.json()

if data['status'] == 'completed':
    print(data['final_report'])
```

**Response:**
```json
{
  "task_id": "uuid",
  "status": "completed",
  "prompt": "Research question",
  "final_report": "Generated research report...",
  "created_at": "2025-01-01T12:00:00",
  "updated_at": "2025-01-01T12:05:00"
}
```

## Implementing Custom Agents

### Research Agent Pattern

```python
from typing import List, Dict
import os

def research_agent(query: str, model: str = "openai:gpt-4o") -> str:
    """
    Research agent that uses tools to gather information.
    """
    from src.research_tools import (
        tavily_search_tool,
        arxiv_search_tool,
        wikipedia_search_tool
    )
    
    # Define available tools
    tools = [
        tavily_search_tool,
        arxiv_search_tool,
        wikipedia_search_tool
    ]
    
    # Use aisuite or similar to call LLM with tools
    client = get_ai_client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a research assistant. Use tools to gather information."
        },
        {
            "role": "user",
            "content": query
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        tools=tools,
        tool_choice="auto"
    )
    
    # Handle tool calls and aggregate results
    return process_tool_results(response)
```

### Writer Agent Pattern

```python
def writer_agent(research_data: str, topic: str, model: str = "openai:gpt-4o") -> str:
    """
    Writer agent that synthesizes research into a report.
    """
    client = get_ai_client()
    
    prompt = f"""
    Based on the following research data, write a comprehensive report on: {topic}
    
    Research Data:
    {research_data}
    
    Format the report with:
    - Executive Summary
    - Key Findings
    - Detailed Analysis
    - Conclusion
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are an expert technical writer."},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content
```

### Editor Agent Pattern

```python
def editor_agent(draft: str, model: str = "openai:gpt-4o") -> str:
    """
    Editor agent that refines and improves the draft.
    """
    client = get_ai_client()
    
    prompt = f"""
    Review and improve this draft report:
    
    {draft}
    
    Focus on:
    - Clarity and coherence
    - Factual accuracy
    - Proper structure
    - Professional tone
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are an expert editor."},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content
```

## Research Tools Implementation

### Tavily Search Tool

```python
import requests
import os

def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    """
    Search the web using Tavily API.
    """
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results,
        "search_depth": "advanced"
    }
    
    response = requests.post(url, json=payload)
    
    if response.status_code == 200:
        data = response.json()
        return {
            "results": [
                {
                    "title": r["title"],
                    "url": r["url"],
                    "content": r["content"]
                }
                for r in data.get("results", [])
            ]
        }
    
    return {"error": f"Tavily API error: {response.status_code}"}
```

### arXiv Search Tool

```python
import requests
import xml.etree.ElementTree as ET

def arxiv_search_tool(query: str, max_results: int = 5) -> dict:
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
    
    if response.status_code != 200:
        return {"error": f"arXiv API error: {response.status_code}"}
    
    root = ET.fromstring(response.content)
    ns = {"atom": "http://www.w3.org/2005/Atom"}
    
    papers = []
    for entry in root.findall("atom:entry", ns):
        papers.append({
            "title": entry.find("atom:title", ns).text.strip(),
            "authors": [
                author.find("atom:name", ns).text
                for author in entry.findall("atom:author", ns)
            ],
            "summary": entry.find("atom:summary", ns).text.strip(),
            "link": entry.find("atom:id", ns).text,
            "published": entry.find("atom:published", ns).text
        })
    
    return {"papers": papers}
```

### Wikipedia Search Tool

```python
import wikipedia

def wikipedia_search_tool(query: str, sentences: int = 3) -> dict:
    """
    Search Wikipedia for information.
    """
    try:
        # Search for pages
        search_results = wikipedia.search(query, results=3)
        
        if not search_results:
            return {"error": "No Wikipedia results found"}
        
        # Get summary of top result
        page = wikipedia.page(search_results[0], auto_suggest=False)
        
        return {
            "title": page.title,
            "summary": wikipedia.summary(query, sentences=sentences),
            "url": page.url,
            "related": search_results[1:]
        }
    
    except wikipedia.exceptions.DisambiguationError as e:
        return {
            "error": "Disambiguation needed",
            "options": e.options[:5]
        }
    except wikipedia.exceptions.PageError:
        return {"error": "Page not found"}
```

## Planning Agent Workflow

The planning agent coordinates the entire research workflow:

```python
from typing import List, Dict
import json

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> List[Dict]:
    """
    Plans the research workflow by breaking down the task.
    """
    client = get_ai_client()
    
    planning_prompt = f"""
    Create a research plan for: {prompt}
    
    Break this down into steps using these agent types:
    - research: Gather information using search tools
    - writer: Synthesize information into content
    - editor: Review and improve content
    
    Return a JSON array of steps:
    [
      {{"step": 1, "agent": "research", "task": "description"}},
      {{"step": 2, "agent": "writer", "task": "description"}},
      {{"step": 3, "agent": "editor", "task": "description"}}
    ]
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a research planning expert."},
            {"role": "user", "content": planning_prompt}
        ]
    )
    
    plan_text = response.choices[0].message.content
    return json.loads(plan_text)


def executor_agent_step(step: Dict, context: str, model: str) -> str:
    """
    Executes a single step of the plan.
    """
    agent_type = step["agent"]
    task = step["task"]
    
    if agent_type == "research":
        return research_agent(task, model)
    elif agent_type == "writer":
        return writer_agent(context, task, model)
    elif agent_type == "editor":
        return editor_agent(context, model)
    else:
        return f"Unknown agent type: {agent_type}"
```

## Database Operations

### Recording Progress

```python
from sqlalchemy.orm import Session
from datetime import datetime

def record_step_progress(
    session: Session,
    task_id: str,
    step_number: int,
    step_name: str,
    status: str,
    result: str = None
):
    """
    Record progress for a specific step.
    """
    progress = TaskProgress(
        task_id=task_id,
        step_number=step_number,
        step_name=step_name,
        status=status,
        result=result,
        timestamp=datetime.utcnow()
    )
    session.add(progress)
    session.commit()


def update_task_status(
    session: Session,
    task_id: str,
    status: str,
    final_report: str = None
):
    """
    Update overall task status.
    """
    task = session.query(TaskStatus).filter_by(task_id=task_id).first()
    if task:
        task.status = status
        task.updated_at = datetime.utcnow()
        if final_report:
            task.final_report = final_report
        session.commit()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional overrides
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Development
RESET_DB_ON_STARTUP=0  # Set to 1 to drop tables on startup
```

### Model Configuration

The service supports different AI models via the `aisuite` pattern:

```python
# In requests, specify model as:
{
  "model": "openai:gpt-4o",        # OpenAI GPT-4
  "model": "openai:gpt-4o-mini",   # OpenAI GPT-4 Mini
  "model": "anthropic:claude-3",   # Anthropic Claude (if configured)
}
```

## Common Patterns

### Complete Research Workflow

```python
import requests
import time

def run_research_task(prompt: str, model: str = "openai:gpt-4o") -> str:
    """
    Submit a research task and wait for completion.
    """
    # Start task
    response = requests.post(
        "http://localhost:8000/generate_report",
        json={"prompt": prompt, "model": model}
    )
    task_id = response.json()["task_id"]
    print(f"Task {task_id} started")
    
    # Poll until complete
    while True:
        response = requests.get(
            f"http://localhost:8000/task_progress/{task_id}"
        )
        data = response.json()
        
        print(f"Status: {data['status']}")
        
        if data['status'] == 'completed':
            # Get final report
            response = requests.get(
                f"http://localhost:8000/task_status/{task_id}"
            )
            return response.json()['final_report']
        
        elif data['status'] == 'failed':
            raise Exception("Task failed")
        
        time.sleep(3)

# Example usage
report = run_research_task(
    "Explain the latest developments in quantum computing for drug discovery"
)
print(report)
```

### Batch Research Tasks

```python
import requests
from concurrent.futures import ThreadPoolExecutor, as_completed

def submit_batch_tasks(prompts: list) -> dict:
    """
    Submit multiple research tasks in parallel.
    """
    task_ids = {}
    
    for prompt in prompts:
        response = requests.post(
            "http://localhost:8000/generate_report",
            json={"prompt": prompt, "model": "openai:gpt-4o"}
        )
        task_id = response.json()["task_id"]
        task_ids[task_id] = prompt
    
    return task_ids


def wait_for_batch(task_ids: dict, timeout: int = 600) -> dict:
    """
    Wait for all tasks to complete.
    """
    results = {}
    start_time = time.time()
    
    while task_ids and (time.time() - start_time) < timeout:
        for task_id in list(task_ids.keys()):
            response = requests.get(
                f"http://localhost:8000/task_status/{task_id}"
            )
            data = response.json()
            
            if data['status'] == 'completed':
                results[task_id] = data['final_report']
                del task_ids[task_id]
            elif data['status'] == 'failed':
                results[task_id] = None
                del task_ids[task_id]
        
        time.sleep(5)
    
    return results

# Example usage
prompts = [
    "AI safety alignment research",
    "Multimodal learning architectures",
    "Efficient transformers for edge devices"
]

task_ids = submit_batch_tasks(prompts)
results = wait_for_batch(task_ids)

for task_id, report in results.items():
    print(f"\n{'='*60}")
    print(f"Task {task_id}:")
    print(report)
```

## Troubleshooting

### Database Connection Issues

**Problem:** `DATABASE_URL not set` error

```python
# Verify environment variable
import os
print(os.getenv("DATABASE_URL"))

# Should output something like:
# postgresql://app:local@127.0.0.1:5432/appdb
```

**Solution:** Ensure the entrypoint script runs or manually set:

```bash
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
```

### Tables Not Persisting

**Problem:** Tables disappear after restart

Check if `main.py` drops tables on startup:

```python
# In main.py, comment out or guard:
# Base.metadata.drop_all(bind=engine)  # ← Remove this

# Or guard with environment variable:
if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)
```

### API Key Errors

**Problem:** `TAVILY_API_KEY not set` or search failures

```bash
# Verify .env file exists
cat .env

# Should contain:
# OPENAI_API_KEY=sk-...
# TAVILY_API_KEY=tvly-...

# Ensure Docker run includes --env-file
docker run --env-file .env ...
```

### Task Stuck in "Running" Status

**Problem:** Task never completes

```python
# Check logs for errors
docker logs -f fpsvc

# Query database directly
import psycopg2

conn = psycopg2.connect(
    "postgresql://app:local@localhost:5432/appdb"
)
cur = conn.cursor()

# Check task status
cur.execute("SELECT * FROM task_status WHERE task_id = %s", (task_id,))
print(cur.fetchone())

# Check progress steps
cur.execute(
    "SELECT * FROM task_progress WHERE task_id = %s ORDER BY step_number",
    (task_id,)
)
for row in cur.fetchall():
    print(row)
```

### Tool Execution Failures

**Problem:** Research tools return errors

```python
# Test tools individually
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Test Tavily
result = tavily_search_tool("machine learning")
print(result)

# Test arXiv
result = arxiv_search_tool("neural networks")
print(result)

# Test Wikipedia
result = wikipedia_search_tool("artificial intelligence")
print(result)
```

### Memory Issues with Large Reports

**Problem:** Container runs out of memory

```bash
# Increase Docker memory limits
docker run --memory=4g --memory-swap=4g ...

# Or limit context passed between steps
def truncate_context(text: str, max_chars: int = 10000) -> str:
    """Truncate context to prevent memory issues."""
    if len(text) > max_chars:
        return text[:max_chars] + "\n\n[Content truncated...]"
    return text
```

### Port Conflicts

**Problem:** `Address already in use` error

```bash
# Check what's using the ports
lsof -i :8000
lsof -i :5432

# Use different ports
docker run -p 8001:8000 -p 5433:5432 ...
```

## Testing

### Unit Test Pattern

```python
import pytest
from unittest.mock import Mock, patch
from src.research_tools import tavily_search_tool

def test_tavily_search_success():
    """Test Tavily search with mocked response."""
    with patch('requests.post') as mock_post:
        mock_post.return_value.status_code = 200
        mock_post.return_value.json.return_value = {
            "results": [
                {
                    "title": "Test",
                    "url": "http://test.com",
                    "content": "Test content"
                }
            ]
        }
        
        result = tavily_search_tool("test query")
        assert "results" in result
        assert len(result["results"]) == 1


def test_research_agent():
    """Test research agent workflow."""
    with patch('src.agents.get_ai_client') as mock_client:
        mock_response = Mock()
        mock_response.choices[0].message.content = "Research findings"
        mock_client.return_value.chat.completions.create.return_value = mock_response
        
        from src.agents import research_agent
        result = research_agent("test query")
        assert isinstance(result, str)
        assert len(result) > 0
```

### Integration Test

```python
import requests
import time

def test_end_to_end_workflow():
    """Test complete research workflow."""
    # Submit task
    response = requests.post(
        "http://localhost:8000/generate_report",
        json={
            "prompt": "Test research topic",
            "model": "openai:gpt-4o"
        }
    )
    assert response.status_code == 200
    task_id = response.json()["task_id"]
    
    # Wait for completion (with timeout)
    max_wait = 300  # 5 minutes
    start = time.time()
    
    while time.time() - start < max_wait:
        response = requests.get(
            f"http://localhost:8000/task_status/{task_id}"
        )
        data = response.json()
        
        if data["status"] == "completed":
            assert data["final_report"] is not None
            assert len(data["final_report"]) > 0
            return
        
        time.sleep(5)
    
    pytest.fail("Task did not complete in time")
```

## Advanced Usage

### Custom Agent Implementation

```python
from typing import Callable
from src.agents import research_agent, writer_agent, editor_agent

# Define custom agent
def fact_checker_agent(content: str, model: str = "openai:gpt-4o") -> str:
    """
    Custom agent that verifies factual claims.
    """
    client = get_ai_client()
    
    prompt = f"""
    Verify the factual claims in this content:
    
    {content}
    
    For each claim:
    1. Assess accuracy
    2. Provide sources
    3. Flag any inaccuracies
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a fact-checking expert."},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content


# Register custom agent in executor
AGENT_REGISTRY = {
    "research": research_agent,
    "writer": writer_agent,
    "editor": editor_agent,
    "fact_checker": fact_checker_agent,  # Add custom agent
}

def executor_agent_step(step: dict, context: str, model: str) -> str:
    agent_type = step["agent"]
    agent_func = AGENT_REGISTRY.get(agent_type)
    
    if not agent_func:
        raise ValueError(f"Unknown agent type: {agent_type}")
    
    return agent_func(context, model=model)
```

This skill provides comprehensive coverage of the Agentic AI Research Agent service for AI coding agents to effectively assist developers in using and extending the system.
