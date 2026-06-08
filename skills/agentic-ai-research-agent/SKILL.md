---
name: agentic-ai-research-agent
description: Build and run reflective research agents with FastAPI, Postgres, and multi-step workflows using Tavily, arXiv, and Wikipedia tools
triggers:
  - how do I create a research agent workflow
  - set up the agentic AI research service
  - build a multi-step research agent with reflection
  - use tavily arxiv wikipedia in an agent workflow
  - deploy research agent with FastAPI and Postgres
  - create a task-based research agent with progress tracking
  - implement planning and execution agents
  - build reflective research agent with tool use
---

# Agentic AI Research Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The **agentic-ai-public** project is a Research Agent service that implements a reflective, multi-step workflow for conducting research tasks. It combines planning agents, execution agents (research, writer, editor), and tool-using capabilities (Tavily search, arXiv papers, Wikipedia) in a FastAPI web service backed by Postgres for state management.

Key features:
- Multi-agent workflow with planner, research, writer, and editor agents
- Tool integration: Tavily web search, arXiv academic papers, Wikipedia
- Task state tracking with Postgres
- Real-time progress monitoring via REST API
- Web UI for initiating research tasks
- Single-container Docker deployment

## Installation

### Prerequisites

```bash
# Required: Docker installed
# API keys in .env file:
cat > .env << EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF
```

### Build and Run

```bash
# Clone the repository
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Build Docker image
docker build -t fastapi-postgres-service .

# Run container (exposes FastAPI on 8000, Postgres on 5432)
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

### Access Points

- Web UI: `http://localhost:8000/`
- API Docs: `http://localhost:8000/docs`
- Postgres: `postgresql://app:local@localhost:5432/appdb`

## Project Structure

```
.
├── main.py                    # FastAPI app with routes
├── src/
│   ├── planning_agent.py      # Planner and executor logic
│   ├── agents.py              # Research/writer/editor agents
│   └── research_tools.py      # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html             # Web UI template
├── static/                    # CSS/JS assets
├── docker/
│   └── entrypoint.sh          # Container initialization
├── requirements.txt
├── Dockerfile
└── README.md
```

## Core API Usage

### Generate Research Report

```python
import requests
import json

# Initiate a research task
response = requests.post(
    "http://localhost:8000/generate_report",
    headers={"Content-Type": "application/json"},
    json={
        "prompt": "Large Language Models for scientific discovery",
        "model": "openai:gpt-4o"
    }
)

task_data = response.json()
task_id = task_data["task_id"]
print(f"Task initiated: {task_id}")
```

### Monitor Progress

```python
# Poll task progress
progress_response = requests.get(
    f"http://localhost:8000/task_progress/{task_id}"
)
progress = progress_response.json()

print(f"Status: {progress['status']}")
print(f"Current step: {progress['current_step']}")
print(f"Steps completed: {progress['steps_completed']}")

# Get detailed step information
for step in progress.get('steps', []):
    print(f"  - {step['name']}: {step['status']}")
```

### Retrieve Final Report

```python
# Get final task status and report
status_response = requests.get(
    f"http://localhost:8000/task_status/{task_id}"
)
result = status_response.json()

if result['status'] == 'completed':
    print("Research Report:")
    print(result['report'])
else:
    print(f"Task status: {result['status']}")
    print(f"Error: {result.get('error', 'N/A')}")
```

## Building Custom Research Tools

### Creating a New Research Tool

```python
# src/research_tools.py
import requests
from typing import Dict, List

def custom_search_tool(query: str, max_results: int = 5) -> List[Dict]:
    """
    Custom research tool template.
    
    Args:
        query: Search query string
        max_results: Maximum number of results to return
        
    Returns:
        List of result dictionaries with 'title', 'content', 'url'
    """
    results = []
    
    # Implement your search logic
    # Example: API call to custom data source
    response = requests.get(
        "https://api.example.com/search",
        params={"q": query, "limit": max_results}
    )
    
    if response.status_code == 200:
        data = response.json()
        for item in data.get("results", []):
            results.append({
                "title": item.get("title", ""),
                "content": item.get("snippet", ""),
                "url": item.get("url", "")
            })
    
    return results
```

### Using Built-in Tools

```python
# Example: Using Tavily search
from src.research_tools import tavily_search_tool
import os

# Ensure TAVILY_API_KEY is set in environment
results = tavily_search_tool(
    query="quantum computing applications",
    max_results=10
)

for result in results:
    print(f"Title: {result['title']}")
    print(f"Content: {result['content'][:200]}...")
    print(f"URL: {result['url']}\n")
```

```python
# Example: Searching arXiv
from src.research_tools import arxiv_search_tool

papers = arxiv_search_tool(
    query="machine learning optimization",
    max_results=5
)

for paper in papers:
    print(f"Title: {paper['title']}")
    print(f"Authors: {paper['authors']}")
    print(f"Summary: {paper['summary'][:300]}...")
    print(f"PDF: {paper['pdf_url']}\n")
```

```python
# Example: Wikipedia search
from src.research_tools import wikipedia_search_tool

wiki_results = wikipedia_search_tool(
    query="artificial intelligence",
    max_results=3
)

for article in wiki_results:
    print(f"Title: {article['title']}")
    print(f"Summary: {article['content'][:400]}...")
    print(f"URL: {article['url']}\n")
```

## Implementing Custom Agents

### Research Agent Pattern

```python
# src/agents.py
from typing import Dict, List
import aisuite

client = aisuite.Client()

def research_agent(topic: str, tools: List) -> Dict:
    """
    Research agent that uses multiple tools to gather information.
    
    Args:
        topic: Research topic
        tools: List of tool functions to use
        
    Returns:
        Dictionary with research findings
    """
    findings = []
    
    for tool in tools:
        try:
            results = tool(topic)
            findings.extend(results)
        except Exception as e:
            print(f"Tool error: {e}")
            continue
    
    # Synthesize findings using LLM
    prompt = f"""
    Based on the following research findings about "{topic}":
    
    {format_findings(findings)}
    
    Provide a comprehensive summary of key insights.
    """
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    
    return {
        "topic": topic,
        "raw_findings": findings,
        "summary": response.choices[0].message.content
    }

def format_findings(findings: List[Dict]) -> str:
    """Format findings for LLM consumption."""
    formatted = []
    for i, finding in enumerate(findings, 1):
        formatted.append(f"{i}. {finding.get('title', 'Untitled')}")
        formatted.append(f"   {finding.get('content', '')[:200]}...")
    return "\n".join(formatted)
```

### Writer Agent Pattern

```python
def writer_agent(research_data: Dict, style: str = "academic") -> str:
    """
    Writer agent that creates structured reports from research.
    
    Args:
        research_data: Output from research agent
        style: Writing style (academic, blog, technical)
        
    Returns:
        Formatted report string
    """
    prompt = f"""
    Write a {style} report on "{research_data['topic']}" based on this research:
    
    {research_data['summary']}
    
    Structure the report with:
    1. Introduction
    2. Key Findings
    3. Analysis
    4. Conclusion
    5. References
    """
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content
```

### Editor Agent Pattern

```python
def editor_agent(draft_report: str, criteria: List[str]) -> Dict:
    """
    Editor agent that reviews and improves reports.
    
    Args:
        draft_report: Initial report draft
        criteria: List of editing criteria to check
        
    Returns:
        Dictionary with edited report and feedback
    """
    criteria_text = "\n".join([f"- {c}" for c in criteria])
    
    prompt = f"""
    Review and edit this report according to these criteria:
    {criteria_text}
    
    Original Report:
    {draft_report}
    
    Provide:
    1. Edited version
    2. List of changes made
    3. Quality score (1-10)
    """
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    
    content = response.choices[0].message.content
    
    return {
        "edited_report": content,
        "original": draft_report
    }
```

## Planning Agent Workflow

### Implementing a Planner

```python
# src/planning_agent.py
from typing import List, Dict

def planner_agent(user_prompt: str) -> List[Dict]:
    """
    Planner agent that breaks down research into steps.
    
    Args:
        user_prompt: User's research request
        
    Returns:
        List of execution steps with tool assignments
    """
    prompt = f"""
    Create a research plan for: "{user_prompt}"
    
    Break this into steps with format:
    1. [RESEARCH] Topic to research
    2. [WRITE] Section to write
    3. [EDIT] Criteria to check
    
    Return as JSON array with step_number, action, description, tools.
    """
    
    response = client.chat.completions.create(
        model="openai:gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    
    # Parse response into structured plan
    plan_text = response.choices[0].message.content
    # Implementation would parse JSON and return structured steps
    
    return parse_plan(plan_text)

def executor_agent_step(step: Dict, context: Dict) -> Dict:
    """
    Execute a single step from the plan.
    
    Args:
        step: Step dictionary from planner
        context: Accumulated context from previous steps
        
    Returns:
        Step result with outputs
    """
    action = step["action"]
    
    if action == "RESEARCH":
        from src.research_tools import tavily_search_tool, arxiv_search_tool
        tools = [tavily_search_tool, arxiv_search_tool]
        result = research_agent(step["description"], tools)
        
    elif action == "WRITE":
        result = writer_agent(context.get("research_data", {}))
        
    elif action == "EDIT":
        result = editor_agent(
            context.get("draft", ""),
            step.get("criteria", ["clarity", "accuracy"])
        )
    
    return {
        "step_number": step["step_number"],
        "action": action,
        "result": result,
        "status": "completed"
    }
```

## Database Integration

### Task Management with SQLAlchemy

```python
# main.py
from sqlalchemy import create_engine, Column, String, Text, DateTime, Integer
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")
    current_step = Column(Integer, default=0)
    total_steps = Column(Integer, default=0)
    report = Column(Text)
    error = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

Base.metadata.create_all(bind=engine)

# Create a new task
def create_task(prompt: str, model: str) -> str:
    import uuid
    task_id = str(uuid.uuid4())
    
    db = SessionLocal()
    task = Task(
        id=task_id,
        prompt=prompt,
        model=model,
        status="pending"
    )
    db.add(task)
    db.commit()
    db.close()
    
    return task_id

# Update task progress
def update_task_progress(task_id: str, step: int, total: int, status: str):
    db = SessionLocal()
    task = db.query(Task).filter(Task.id == task_id).first()
    if task:
        task.current_step = step
        task.total_steps = total
        task.status = status
        db.commit()
    db.close()
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key for LLM calls
TAVILY_API_KEY=tvly-...         # Tavily search API key

# Database (optional, defaults provided)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb

# Optional
RESET_DB_ON_STARTUP=0           # Set to 1 to drop tables on start
```

### Custom Model Configuration

```python
# Using different models
models = {
    "openai": "openai:gpt-4o",
    "openai-mini": "openai:gpt-4o-mini",
    "anthropic": "anthropic:claude-3-5-sonnet-20241022",
}

# Make request with specific model
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "AI safety research",
        "model": models["anthropic"]
    }
)
```

## Common Patterns

### Async Research Workflow

```python
import asyncio
import aiohttp

async def async_research_workflow(prompts: List[str]):
    """Run multiple research tasks concurrently."""
    async with aiohttp.ClientSession() as session:
        tasks = []
        for prompt in prompts:
            task = session.post(
                "http://localhost:8000/generate_report",
                json={"prompt": prompt, "model": "openai:gpt-4o"}
            )
            tasks.append(task)
        
        responses = await asyncio.gather(*tasks)
        task_ids = [await r.json() for r in responses]
        
        return task_ids

# Usage
prompts = [
    "Quantum computing breakthroughs",
    "AI in healthcare",
    "Renewable energy innovations"
]
task_ids = asyncio.run(async_research_workflow(prompts))
```

### Polling with Timeout

```python
import time

def wait_for_completion(task_id: str, timeout: int = 300) -> Dict:
    """
    Poll task until completion or timeout.
    
    Args:
        task_id: Task identifier
        timeout: Maximum seconds to wait
        
    Returns:
        Final task result
    """
    start_time = time.time()
    
    while time.time() - start_time < timeout:
        response = requests.get(f"http://localhost:8000/task_status/{task_id}")
        result = response.json()
        
        if result['status'] in ['completed', 'failed']:
            return result
        
        time.sleep(5)  # Poll every 5 seconds
    
    raise TimeoutError(f"Task {task_id} did not complete within {timeout}s")
```

### Streaming Progress Updates

```python
def stream_progress(task_id: str):
    """Stream progress updates to console."""
    last_step = -1
    
    while True:
        response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
        progress = response.json()
        
        if progress['current_step'] > last_step:
            print(f"Step {progress['current_step']}/{progress.get('total_steps', '?')}: "
                  f"{progress.get('current_step_name', 'Processing...')}")
            last_step = progress['current_step']
        
        if progress['status'] in ['completed', 'failed']:
            break
        
        time.sleep(2)
    
    # Get final report
    final = requests.get(f"http://localhost:8000/task_status/{task_id}").json()
    return final
```

## Troubleshooting

### Container Issues

```bash
# Check if Postgres is running inside container
docker exec fpsvc pg_lsclusters

# View application logs
docker logs -f fpsvc

# Connect to Postgres from host
psql "postgresql://app:local@localhost:5432/appdb"

# Inspect database tables
docker exec -it fpsvc psql -U app -d appdb -c "\dt"
```

### API Key Errors

```python
# Verify API keys are loaded
import os
from dotenv import load_dotenv

load_dotenv()

if not os.getenv("OPENAI_API_KEY"):
    raise ValueError("OPENAI_API_KEY not found in environment")

if not os.getenv("TAVILY_API_KEY"):
    print("Warning: TAVILY_API_KEY not set, Tavily tool will fail")
```

### Database Connection Issues

```python
# Test database connection
from sqlalchemy import create_engine, text

try:
    engine = create_engine(os.getenv("DATABASE_URL"))
    with engine.connect() as conn:
        result = conn.execute(text("SELECT 1"))
        print("Database connection successful")
except Exception as e:
    print(f"Database connection failed: {e}")
```

### Tool Failures

```python
# Graceful tool failure handling
def safe_tool_call(tool_func, query: str, default=None):
    """Wrapper for tool calls with error handling."""
    try:
        return tool_func(query)
    except Exception as e:
        print(f"Tool {tool_func.__name__} failed: {e}")
        return default or []

# Usage in research agent
from src.research_tools import tavily_search_tool, arxiv_search_tool

results = []
results.extend(safe_tool_call(tavily_search_tool, "query"))
results.extend(safe_tool_call(arxiv_search_tool, "query"))
```

### Template Not Found

```bash
# Verify templates directory in container
docker exec fpsvc ls -la /app/templates/

# If missing, rebuild ensuring COPY directive in Dockerfile:
# COPY templates/ /app/templates/
# COPY static/ /app/static/
```

### Rate Limiting

```python
import time
from functools import wraps

def rate_limit(calls_per_minute: int):
    """Rate limiting decorator for API calls."""
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

@rate_limit(calls_per_minute=10)
def call_wikipedia_api(query: str):
    # Wikipedia API call implementation
    pass
```
