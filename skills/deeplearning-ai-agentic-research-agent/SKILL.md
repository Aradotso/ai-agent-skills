---
name: deeplearning-ai-agentic-research-agent
description: FastAPI research agent service with planning, tool-using agents (Tavily, arXiv, Wikipedia), and Postgres task tracking
triggers:
  - set up the agentic research agent service
  - use the DeepLearning.AI research agent API
  - create a reflective research workflow with planning agents
  - integrate Tavily arXiv and Wikipedia tools for research
  - run multi-step agent research tasks with FastAPI
  - track research agent task progress with Postgres
  - build a research agent with planner executor and editor agents
  - deploy the agentic AI research service
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The DeepLearning.AI Agentic AI Research Agent is a FastAPI web service that orchestrates multi-step research workflows using AI agents. It implements a reflective planning pattern where a planner agent creates a research strategy, executor agents gather information using tools (Tavily search, arXiv papers, Wikipedia), and writer/editor agents synthesize findings into reports. All task state and results are stored in Postgres for live progress tracking.

**Key capabilities:**
- Multi-agent workflow: planner → research → writer → editor
- Tool integration: Tavily web search, arXiv academic papers, Wikipedia
- Task state management with Postgres
- RESTful API for task creation and progress monitoring
- Web UI for initiating research tasks

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

The service will:
- Start Postgres on port 5432
- Initialize the database schema
- Launch FastAPI on port 8000

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres separately and configure
export DATABASE_URL="postgresql://user:password@localhost:5432/appdb"
export OPENAI_API_KEY="your-openai-key"
export TAVILY_API_KEY="your-tavily-key"

# Run with uvicorn
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                      # FastAPI app with endpoints
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI template
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Postgres + Uvicorn startup
├── requirements.txt
├── Dockerfile
└── README.md
```

## API Reference

### Endpoints

#### `POST /generate_report`
Start a new research task.

**Request:**
```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

#### `GET /task_progress/{task_id}`
Get live progress updates for a running task.

**Request:**
```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "running",
  "current_step": "research",
  "steps_completed": 1,
  "total_steps": 4,
  "substeps": [
    {
      "name": "Planning",
      "status": "completed",
      "result": "Generated 3-step research plan"
    },
    {
      "name": "Research: Tavily search",
      "status": "running",
      "result": null
    }
  ]
}
```

#### `GET /task_status/{task_id}`
Get final task status and generated report.

**Request:**
```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report: Large Language Models...\n\n## Introduction\n...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:23Z"
}
```

#### `GET /`
Serves the web UI (Jinja2 template).

## Code Examples

### Creating Custom Research Tools

```python
# src/research_tools.py
import os
import requests

def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    """Search the web using Tavily API."""
    api_key = os.getenv("TAVILY_API_KEY")
    url = "https://api.tavily.com/search"
    
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results,
        "search_depth": "advanced"
    }
    
    response = requests.post(url, json=payload)
    response.raise_for_status()
    
    results = response.json()
    return {
        "source": "Tavily",
        "query": query,
        "results": results.get("results", [])
    }

def arxiv_search_tool(query: str, max_results: int = 5) -> dict:
    """Search arXiv for academic papers."""
    import arxiv
    
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    papers = []
    for result in search.results():
        papers.append({
            "title": result.title,
            "authors": [author.name for author in result.authors],
            "summary": result.summary,
            "published": result.published.isoformat(),
            "pdf_url": result.pdf_url
        })
    
    return {
        "source": "arXiv",
        "query": query,
        "results": papers
    }

def wikipedia_search_tool(query: str) -> dict:
    """Search Wikipedia for articles."""
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "source": "Wikipedia",
            "query": query,
            "results": [{
                "title": page.title,
                "summary": page.summary,
                "url": page.url,
                "content": page.content[:5000]  # Limit content length
            }]
        }
    except wikipedia.exceptions.DisambiguationError as e:
        # Handle multiple possible pages
        return {
            "source": "Wikipedia",
            "query": query,
            "results": [{"options": e.options[:5]}]
        }
    except wikipedia.exceptions.PageError:
        return {
            "source": "Wikipedia",
            "query": query,
            "results": []
        }
```

### Implementing Custom Agents

```python
# src/agents.py
from typing import Dict, List

def research_agent(topic: str, tools: List[callable]) -> Dict:
    """Execute research using available tools."""
    results = []
    
    for tool in tools:
        try:
            tool_result = tool(topic)
            results.append(tool_result)
        except Exception as e:
            results.append({
                "source": tool.__name__,
                "error": str(e)
            })
    
    return {
        "topic": topic,
        "findings": results,
        "sources_count": len(results)
    }

def writer_agent(research_data: Dict, model: str = "openai:gpt-4o") -> str:
    """Generate a structured report from research findings."""
    import aisuite as ai
    
    client = ai.Client()
    
    # Prepare context from research
    context = "\n\n".join([
        f"## Source: {finding.get('source', 'Unknown')}\n{str(finding.get('results', ''))}"
        for finding in research_data.get('findings', [])
    ])
    
    messages = [
        {
            "role": "system",
            "content": "You are a research report writer. Create comprehensive, well-structured reports."
        },
        {
            "role": "user",
            "content": f"Topic: {research_data.get('topic')}\n\nResearch findings:\n{context}\n\nWrite a detailed report."
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7
    )
    
    return response.choices[0].message.content

def editor_agent(draft_report: str, model: str = "openai:gpt-4o") -> str:
    """Review and improve the draft report."""
    import aisuite as ai
    
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are an editor. Improve clarity, structure, and accuracy. Fix any errors."
        },
        {
            "role": "user",
            "content": f"Please edit and improve this report:\n\n{draft_report}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.3
    )
    
    return response.choices[0].message.content
```

### Planning Agent Workflow

```python
# src/planning_agent.py
from typing import List, Dict
import aisuite as ai

def planner_agent(prompt: str, model: str = "openai:gpt-4o") -> List[Dict]:
    """Create a research plan with steps and tool selections."""
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": """You are a research planner. Create a step-by-step research plan.
            
Available tools:
- tavily_search: Web search for current information
- arxiv_search: Academic papers and research
- wikipedia_search: Encyclopedia articles and background

Output JSON array of steps:
[
  {"step": 1, "action": "tavily_search", "query": "...", "purpose": "..."},
  {"step": 2, "action": "arxiv_search", "query": "...", "purpose": "..."}
]"""
        },
        {
            "role": "user",
            "content": f"Create a research plan for: {prompt}"
        }
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.5
    )
    
    import json
    plan = json.loads(response.choices[0].message.content)
    return plan

def executor_agent_step(step: Dict, tools_map: Dict) -> Dict:
    """Execute a single research step."""
    action = step.get("action")
    query = step.get("query")
    
    tool = tools_map.get(action)
    if not tool:
        return {"error": f"Tool {action} not found"}
    
    try:
        result = tool(query)
        return {
            "step": step.get("step"),
            "action": action,
            "query": query,
            "result": result,
            "status": "completed"
        }
    except Exception as e:
        return {
            "step": step.get("step"),
            "action": action,
            "error": str(e),
            "status": "failed"
        }
```

### Database Models

```python
# main.py (excerpt)
from sqlalchemy import Column, String, DateTime, Text, Integer
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.dialects.postgresql import UUID
import uuid
from datetime import datetime

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    prompt = Column(Text, nullable=False)
    model = Column(String(100), nullable=False)
    status = Column(String(50), default="pending")  # pending, running, completed, failed
    report = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)
    
class TaskStep(Base):
    __tablename__ = "task_steps"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    task_id = Column(UUID(as_uuid=True), nullable=False)
    step_number = Column(Integer, nullable=False)
    step_name = Column(String(200), nullable=False)
    status = Column(String(50), default="pending")
    result = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
```

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | Postgres connection string (auto-set in Docker) |
| `OPENAI_API_KEY` | Yes | OpenAI API key for LLM calls |
| `TAVILY_API_KEY` | Yes | Tavily search API key |
| `POSTGRES_USER` | No | Postgres username (default: `app`) |
| `POSTGRES_PASSWORD` | No | Postgres password (default: `local`) |
| `POSTGRES_DB` | No | Database name (default: `appdb`) |
| `RESET_DB_ON_STARTUP` | No | Set to `1` to drop/recreate tables on startup |

### Default DATABASE_URL (Docker)

```bash
postgresql://app:local@127.0.0.1:5432/appdb
```

## Common Patterns

### Async Task Execution

The service runs research tasks in background threads to avoid blocking the API:

```python
import threading
from fastapi import BackgroundTasks

def run_research_workflow(task_id: str, prompt: str, model: str):
    """Background task for research workflow."""
    # Update task status
    update_task_status(task_id, "running")
    
    # 1. Planning
    plan = planner_agent(prompt, model)
    save_task_step(task_id, 1, "planning", "completed", str(plan))
    
    # 2. Execute research steps
    tools_map = {
        "tavily_search": tavily_search_tool,
        "arxiv_search": arxiv_search_tool,
        "wikipedia_search": wikipedia_search_tool
    }
    
    research_results = []
    for i, step in enumerate(plan):
        result = executor_agent_step(step, tools_map)
        research_results.append(result)
        save_task_step(task_id, i + 2, f"research_{step['action']}", "completed", str(result))
    
    # 3. Write draft
    draft = writer_agent({"topic": prompt, "findings": research_results}, model)
    save_task_step(task_id, len(plan) + 2, "writing", "completed", draft)
    
    # 4. Edit final report
    final_report = editor_agent(draft, model)
    save_task_step(task_id, len(plan) + 3, "editing", "completed", final_report)
    
    # Update task with final report
    update_task_status(task_id, "completed", final_report)

@app.post("/generate_report")
def generate_report(request: ResearchRequest):
    task_id = str(uuid.uuid4())
    create_task(task_id, request.prompt, request.model)
    
    # Start background thread
    thread = threading.Thread(
        target=run_research_workflow,
        args=(task_id, request.prompt, request.model)
    )
    thread.daemon = True
    thread.start()
    
    return {"task_id": task_id}
```

### Polling for Progress

Client-side polling pattern:

```javascript
async function pollTaskProgress(taskId) {
    const maxAttempts = 60;
    let attempts = 0;
    
    while (attempts < maxAttempts) {
        const response = await fetch(`/task_progress/${taskId}`);
        const data = await response.json();
        
        console.log(`Status: ${data.status}, Steps: ${data.steps_completed}/${data.total_steps}`);
        
        if (data.status === 'completed' || data.status === 'failed') {
            break;
        }
        
        await new Promise(resolve => setTimeout(resolve, 2000));
        attempts++;
    }
    
    // Get final report
    const finalResponse = await fetch(`/task_status/${taskId}`);
    const finalData = await finalResponse.json();
    return finalData.report;
}
```

## Troubleshooting

### Container Fails to Start Postgres

**Symptom:** `pg_ctlcluster` errors or password prompts

**Solution:** Ensure the entrypoint uses UNIX socket authentication:
```bash
su -s /bin/bash postgres -c "psql -c 'CREATE DATABASE appdb;'"
```

Don't use `-h 127.0.0.1` for admin commands during initialization.

### Tables Not Persisting

**Symptom:** Data disappears on container restart

**Solution:** Remove or guard the `drop_all` call in `main.py`:
```python
import os

if os.getenv("RESET_DB_ON_STARTUP") == "1":
    Base.metadata.drop_all(bind=engine)

Base.metadata.create_all(bind=engine)
```

### Tavily Rate Limiting

**Symptom:** 429 errors from Tavily API

**Solution:** Implement exponential backoff:
```python
import time
from requests.exceptions import HTTPError

def tavily_search_with_retry(query: str, max_retries: int = 3):
    for attempt in range(max_retries):
        try:
            return tavily_search_tool(query)
        except HTTPError as e:
            if e.response.status_code == 429 and attempt < max_retries - 1:
                wait_time = 2 ** attempt
                time.sleep(wait_time)
                continue
            raise
```

### Missing Templates/Static Files

**Symptom:** 404 errors or template not found

**Solution:** Verify files are copied in Dockerfile:
```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

Check inside container:
```bash
docker exec -it fpsvc ls -la /app/templates
```

### Wikipedia DisambiguationError

**Symptom:** Multiple pages match the query

**Solution:** Handle in the tool:
```python
try:
    page = wikipedia.page(query, auto_suggest=True)
except wikipedia.exceptions.DisambiguationError as e:
    # Take the first option
    page = wikipedia.page(e.options[0])
```

### Database Connection Errors

**Symptom:** `could not connect to server`

**Solution:** Check `DATABASE_URL` format and that Postgres is running:
```bash
docker exec -it fpsvc pg_isready
docker exec -it fpsvc psql -U app -d appdb -c '\dt'
```

### Out of Memory During Research

**Symptom:** Container crashes or becomes unresponsive

**Solution:** Limit result sizes and add memory constraints:
```bash
docker run --memory="2g" --memory-swap="2g" ...
```

Truncate large results:
```python
def arxiv_search_tool(query: str, max_results: int = 3):  # Reduce from 5
    # ... truncate summaries
    papers.append({
        "summary": result.summary[:500]  # Limit length
    })
```
