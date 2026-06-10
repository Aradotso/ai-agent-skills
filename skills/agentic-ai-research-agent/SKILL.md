---
name: agentic-ai-research-agent
description: FastAPI service for multi-agent research workflows with planning, tool execution (Tavily, arXiv, Wikipedia), and PostgreSQL state management
triggers:
  - build a research agent workflow
  - create an agentic research service
  - implement multi-step AI planning and execution
  - set up FastAPI research agent with tools
  - use planning agent with research tools
  - build reflective research workflow
  - create task-based AI agent system
  - implement research agent with Tavily and arXiv
---

# Agentic AI Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

Agentic AI Research Agent is a FastAPI-based service that implements a multi-agent workflow for research tasks. The system uses a planning agent to break down research queries, executes specialized agents (research, writer, editor) with tool access (Tavily search, arXiv, Wikipedia), and stores task state in PostgreSQL. The entire stack runs in a single Docker container for easy development and deployment.

**Key Features:**
- Multi-step agentic workflow with planner → executor pattern
- Research tools integration (Tavily, arXiv, Wikipedia)
- Task state management with PostgreSQL
- Real-time progress tracking via API
- Web UI for initiating research tasks
- Threaded execution for non-blocking operations

## Installation

### Prerequisites

- Docker (Desktop or Engine)
- OpenAI API key
- Tavily API key (for web search)

### Environment Setup

Create a `.env` file in the project root:

```bash
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Build the Container

```bash
docker build -t fastapi-postgres-service .
```

### Run the Service

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

The service will:
1. Start PostgreSQL cluster
2. Create database and user
3. Initialize tables
4. Launch FastAPI on port 8000

Access the UI at `http://localhost:8000/` or API docs at `http://localhost:8000/docs`.

## Project Structure

```
├─ main.py                      # FastAPI app with routes
├─ src/
│  ├─ planning_agent.py         # planner_agent(), executor_agent_step()
│  ├─ agents.py                 # research_agent, writer_agent, editor_agent
│  └─ research_tools.py         # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├─ templates/
│  └─ index.html                # Web UI
├─ static/                      # CSS/JS assets
├─ docker/
│  └─ entrypoint.sh             # Container startup script
├─ requirements.txt
├─ Dockerfile
└─ README.md
```

## Core API Endpoints

### Generate Research Report

Initiates a new research task with multi-agent workflow:

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Large Language Models for scientific discovery",
    "model": "openai:gpt-4o"
  }'
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Check Task Progress

Get real-time status of each workflow step:

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "in_progress",
  "current_step": "research",
  "steps": [
    {
      "step": "planning",
      "status": "completed",
      "timestamp": "2025-01-15T10:30:00Z"
    },
    {
      "step": "research",
      "status": "in_progress",
      "progress": 50
    }
  ]
}
```

### Get Final Report

Retrieve completed research report and final status:

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report\n\n## Executive Summary...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:45:00Z"
}
```

## Implementing Custom Agents

### Research Agent Example

```python
# src/agents.py
import aisuite as ai
from src.research_tools import tavily_search_tool, arxiv_search_tool

def research_agent(query: str, model: str) -> dict:
    """
    Research agent that uses multiple tools to gather information.
    """
    client = ai.Client()
    
    messages = [
        {
            "role": "system",
            "content": "You are a research agent. Use available tools to gather comprehensive information."
        },
        {
            "role": "user",
            "content": f"Research the following topic: {query}"
        }
    ]
    
    tools = [
        tavily_search_tool,
        arxiv_search_tool,
        wikipedia_search_tool
    ]
    
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        tools=tools,
        tool_choice="auto"
    )
    
    # Handle tool calls
    if response.choices[0].message.tool_calls:
        for tool_call in response.choices[0].message.tool_calls:
            function_name = tool_call.function.name
            arguments = json.loads(tool_call.function.arguments)
            
            # Execute tool
            tool_result = execute_tool(function_name, arguments)
            
            # Add tool response to messages
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": json.dumps(tool_result)
            })
    
    # Get final response with tool results
    final_response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    
    return {
        "findings": final_response.choices[0].message.content,
        "sources": extract_sources(messages)
    }
```

### Planning Agent Pattern

```python
# src/planning_agent.py
def planner_agent(prompt: str, model: str) -> list[dict]:
    """
    Creates a multi-step execution plan for a research task.
    """
    client = ai.Client()
    
    planning_prompt = f"""
    Break down this research task into discrete steps:
    {prompt}
    
    Return a JSON array of steps with:
    - step_name: Short identifier (e.g., 'research', 'analyze', 'write')
    - description: What this step accomplishes
    - agent_type: Which agent to use ('research', 'writer', 'editor')
    - dependencies: List of step_names that must complete first
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a planning agent. Create structured execution plans."},
            {"role": "user", "content": planning_prompt}
        ],
        response_format={"type": "json_object"}
    )
    
    plan = json.loads(response.choices[0].message.content)
    return plan["steps"]


def executor_agent_step(step: dict, context: dict, model: str) -> dict:
    """
    Executes a single step from the plan using the appropriate agent.
    """
    agent_map = {
        "research": research_agent,
        "writer": writer_agent,
        "editor": editor_agent
    }
    
    agent_func = agent_map.get(step["agent_type"])
    if not agent_func:
        raise ValueError(f"Unknown agent type: {step['agent_type']}")
    
    result = agent_func(
        query=step["description"],
        context=context,
        model=model
    )
    
    return {
        "step_name": step["step_name"],
        "result": result,
        "status": "completed"
    }
```

## Research Tools Implementation

### Tavily Search Tool

```python
# src/research_tools.py
import os
import requests

tavily_search_tool = {
    "type": "function",
    "function": {
        "name": "tavily_search",
        "description": "Search the web using Tavily API for current information",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Search query"
                },
                "max_results": {
                    "type": "integer",
                    "description": "Maximum number of results",
                    "default": 5
                }
            },
            "required": ["query"]
        }
    }
}

def execute_tavily_search(query: str, max_results: int = 5) -> dict:
    """Execute Tavily search and return results."""
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    response = requests.post(
        "https://api.tavily.com/search",
        json={
            "api_key": api_key,
            "query": query,
            "max_results": max_results,
            "include_answer": True
        }
    )
    
    if response.status_code == 200:
        data = response.json()
        return {
            "results": data.get("results", []),
            "answer": data.get("answer", "")
        }
    
    return {"error": f"Tavily API error: {response.status_code}"}
```

### arXiv Search Tool

```python
arxiv_search_tool = {
    "type": "function",
    "function": {
        "name": "arxiv_search",
        "description": "Search arXiv for academic papers",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Search query for papers"
                },
                "max_results": {
                    "type": "integer",
                    "description": "Maximum papers to return",
                    "default": 5
                }
            },
            "required": ["query"]
        }
    }
}

def execute_arxiv_search(query: str, max_results: int = 5) -> list[dict]:
    """Search arXiv and return paper summaries."""
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
            "pdf_url": result.pdf_url,
            "arxiv_id": result.entry_id.split("/")[-1]
        })
    
    return papers
```

### Wikipedia Search Tool

```python
wikipedia_search_tool = {
    "type": "function",
    "function": {
        "name": "wikipedia_search",
        "description": "Search Wikipedia for background information",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Topic to search"
                }
            },
            "required": ["query"]
        }
    }
}

def execute_wikipedia_search(query: str) -> dict:
    """Search Wikipedia and return summary."""
    import wikipedia
    
    try:
        # Get page summary
        summary = wikipedia.summary(query, sentences=5)
        page = wikipedia.page(query)
        
        return {
            "title": page.title,
            "summary": summary,
            "url": page.url,
            "sections": page.sections[:5]  # First 5 sections
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {
            "error": "disambiguation",
            "options": e.options[:5]
        }
    except wikipedia.exceptions.PageError:
        return {"error": "page_not_found"}
```

## Database Models

```python
# main.py or models.py
from sqlalchemy import Column, String, Text, DateTime, JSON
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime
import uuid

Base = declarative_base()

class ResearchTask(Base):
    __tablename__ = "research_tasks"
    
    task_id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    status = Column(String, default="pending")  # pending, in_progress, completed, failed
    plan = Column(JSON)  # Execution plan from planner_agent
    results = Column(JSON)  # Results from each step
    report = Column(Text)  # Final report
    error = Column(Text)  # Error message if failed
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    completed_at = Column(DateTime)
```

## FastAPI Integration Pattern

```python
# main.py
from fastapi import FastAPI, BackgroundTasks
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import os
import threading

app = FastAPI(title="Research Agent Service")

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# Initialize database
Base.metadata.create_all(bind=engine)


def execute_research_workflow(task_id: str, prompt: str, model: str):
    """Background worker for research workflow."""
    db = SessionLocal()
    
    try:
        task = db.query(ResearchTask).filter_by(task_id=task_id).first()
        task.status = "in_progress"
        db.commit()
        
        # Step 1: Planning
        plan = planner_agent(prompt, model)
        task.plan = plan
        db.commit()
        
        # Step 2: Execute each step
        context = {}
        results = []
        
        for step in plan:
            step_result = executor_agent_step(step, context, model)
            results.append(step_result)
            context[step["step_name"]] = step_result["result"]
            
            # Update progress
            task.results = results
            db.commit()
        
        # Step 3: Generate final report (could be an editor_agent step)
        task.report = generate_final_report(results, model)
        task.status = "completed"
        task.completed_at = datetime.utcnow()
        
    except Exception as e:
        task.status = "failed"
        task.error = str(e)
    
    finally:
        db.commit()
        db.close()


@app.post("/generate_report")
async def generate_report(request: dict, background_tasks: BackgroundTasks):
    """Initiate a research workflow."""
    db = SessionLocal()
    
    task = ResearchTask(
        prompt=request["prompt"],
        model=request.get("model", "openai:gpt-4o")
    )
    
    db.add(task)
    db.commit()
    task_id = task.task_id
    db.close()
    
    # Start workflow in background thread
    thread = threading.Thread(
        target=execute_research_workflow,
        args=(task_id, request["prompt"], task.model)
    )
    thread.start()
    
    return {"task_id": task_id}
```

## Configuration

### Database Configuration

The container uses these defaults (override via environment variables):

```bash
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
```

### Model Configuration

Supports any `aisuite` compatible model:

```python
# OpenAI models
model = "openai:gpt-4o"
model = "openai:gpt-4o-mini"

# Anthropic models
model = "anthropic:claude-3-5-sonnet-20241022"

# Other providers via aisuite
model = "groq:llama-3.1-70b-versatile"
```

### Tool Configuration

Enable/disable tools by modifying the tools list:

```python
# All tools
tools = [tavily_search_tool, arxiv_search_tool, wikipedia_search_tool]

# Only web search
tools = [tavily_search_tool]

# Custom tool subset
tools = [arxiv_search_tool, wikipedia_search_tool]
```

## Common Patterns

### Reflective Agent Pattern

Implement self-correction by having agents review their own output:

```python
def reflective_writer_agent(query: str, research_data: dict, model: str) -> str:
    """Writer agent with reflection step."""
    client = ai.Client()
    
    # Initial draft
    draft_messages = [
        {"role": "system", "content": "You are a research writer."},
        {"role": "user", "content": f"Write a report based on: {research_data}"}
    ]
    
    draft_response = client.chat.completions.create(
        model=model,
        messages=draft_messages
    )
    draft = draft_response.choices[0].message.content
    
    # Reflection step
    reflection_messages = [
        {"role": "system", "content": "You are an editor reviewing a draft."},
        {"role": "user", "content": f"Review this draft and suggest improvements:\n\n{draft}"}
    ]
    
    reflection = client.chat.completions.create(
        model=model,
        messages=reflection_messages
    )
    
    # Revision step
    revision_messages = draft_messages + [
        {"role": "assistant", "content": draft},
        {"role": "user", "content": f"Revise based on feedback: {reflection.choices[0].message.content}"}
    ]
    
    final_response = client.chat.completions.create(
        model=model,
        messages=revision_messages
    )
    
    return final_response.choices[0].message.content
```

### Tool Result Aggregation

Combine results from multiple tools:

```python
def aggregate_research_results(task_id: str, db_session) -> dict:
    """Aggregate all tool results for a task."""
    task = db_session.query(ResearchTask).filter_by(task_id=task_id).first()
    
    aggregated = {
        "web_sources": [],
        "academic_papers": [],
        "wikipedia_info": [],
        "key_findings": []
    }
    
    for result in task.results:
        if result.get("tool") == "tavily":
            aggregated["web_sources"].extend(result["data"]["results"])
        elif result.get("tool") == "arxiv":
            aggregated["academic_papers"].extend(result["data"])
        elif result.get("tool") == "wikipedia":
            aggregated["wikipedia_info"].append(result["data"])
    
    return aggregated
```

### Progressive Context Building

Build context incrementally across workflow steps:

```python
def execute_workflow_with_context(plan: list[dict], model: str) -> dict:
    """Execute plan with cumulative context."""
    context = {
        "original_query": "",
        "research_findings": {},
        "draft_sections": {},
        "final_report": ""
    }
    
    for step in plan:
        if step["agent_type"] == "research":
            # Research agent adds findings to context
            findings = research_agent(step["description"], model)
            context["research_findings"][step["step_name"]] = findings
            
        elif step["agent_type"] == "writer":
            # Writer uses all research findings
            draft = writer_agent(
                query=step["description"],
                research_data=context["research_findings"],
                model=model
            )
            context["draft_sections"][step["step_name"]] = draft
            
        elif step["agent_type"] == "editor":
            # Editor refines all drafts
            context["final_report"] = editor_agent(
                drafts=context["draft_sections"],
                model=model
            )
    
    return context
```

## Troubleshooting

### Container Won't Start

Check PostgreSQL initialization:

```bash
docker logs fpsvc | grep -i postgres
```

Verify entrypoint is executable:

```bash
docker exec fpsvc ls -l /docker/entrypoint.sh
```

### Database Connection Errors

Verify `DATABASE_URL` format:

```bash
docker exec fpsvc bash -c 'echo $DATABASE_URL'
```

Test connection manually:

```bash
docker exec fpsvc psql "postgresql://app:local@127.0.0.1:5432/appdb" -c '\dt'
```

### Tables Not Created

Check if `Base.metadata.create_all()` is called at startup. Enable SQL logging:

```python
engine = create_engine(DATABASE_URL, echo=True)
```

### Tavily API Errors

Verify API key is set:

```bash
docker exec fpsvc bash -c 'echo $TAVILY_API_KEY | head -c 10'
```

Test Tavily endpoint manually:

```python
import requests
import os

response = requests.post(
    "https://api.tavily.com/search",
    json={
        "api_key": os.getenv("TAVILY_API_KEY"),
        "query": "test query",
        "max_results": 1
    }
)
print(response.status_code, response.json())
```

### Rate Limiting Issues

Implement exponential backoff for external APIs:

```python
import time
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    delay = base_delay * (2 ** attempt)
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3)
def execute_tavily_search(query: str, max_results: int = 5):
    # Implementation...
```

### Memory Issues with Long Workflows

Implement context windowing for long conversations:

```python
def trim_context(messages: list[dict], max_tokens: int = 8000) -> list[dict]:
    """Keep only recent messages within token budget."""
    # Simple heuristic: ~4 chars per token
    max_chars = max_tokens * 4
    
    total_chars = 0
    trimmed = []
    
    for msg in reversed(messages):
        msg_chars = len(msg["content"])
        if total_chars + msg_chars > max_chars:
            break
        trimmed.insert(0, msg)
        total_chars += msg_chars
    
    return trimmed
```

### Task Progress Not Updating

Ensure database commits after each step:

```python
def executor_agent_step(step: dict, task_id: str, db_session):
    """Execute step and update progress."""
    task = db_session.query(ResearchTask).filter_by(task_id=task_id).first()
    
    try:
        result = execute_step(step)
        
        # Update progress immediately
        task.results = task.results or []
        task.results.append(result)
        db_session.commit()  # Commit after each step
        
        return result
        
    except Exception as e:
        db_session.rollback()
        raise
```

### Hot Reload for Development

Mount code directory for live updates:

```bash
docker run --rm -it \
  -p 8000:8000 -p 5432:5432 \
  -v "$PWD":/app \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service \
  bash -c "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```
