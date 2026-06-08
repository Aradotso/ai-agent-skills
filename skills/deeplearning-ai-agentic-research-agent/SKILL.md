---
name: deeplearning-ai-agentic-research-agent
description: Research Agent service using FastAPI, Postgres, and multi-step planning with Tavily, arXiv, and Wikipedia tools
triggers:
  - set up agentic research workflow with FastAPI
  - build a reflective research agent with planning
  - create multi-agent research system with Tavily and arXiv
  - implement agentic AI research workflow
  - deploy research agent with Postgres task tracking
  - use planning agent with research and writing steps
  - build multi-step agent workflow with task progress
  - create research report generation with agent orchestration
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based research agent service that orchestrates multi-step workflows using planning, research, writing, and editing agents. Built for the Agentic Workflow course, it demonstrates planning-driven agent execution with tool usage (Tavily search, arXiv papers, Wikipedia) and Postgres-backed task state management.

## What It Does

- **Planning Agent**: Generates a multi-step research plan from a user prompt
- **Executor Agents**: Research, writer, and editor agents that execute planned steps
- **Tool Integration**: Tavily web search, arXiv academic papers, Wikipedia lookups
- **Task Tracking**: Postgres database stores task state, progress, and results
- **Live Progress**: Real-time status updates for each substep in the workflow
- **REST API**: FastAPI endpoints for kicking off tasks and polling progress
- **Web UI**: Simple Jinja2 template for interactive report generation

## Installation

### Docker Setup (Recommended)

Build the Docker image with Postgres and FastAPI in a single container:

```bash
docker build -t fastapi-postgres-service .
```

Create a `.env` file in the project root:

```bash
# .env
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
```

Run the container:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

Access the application:
- UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs
- Postgres: `postgresql://app:local@localhost:5432/appdb`

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up Postgres locally
createdb appdb

# Set environment variables
export DATABASE_URL="postgresql://user:password@localhost:5432/appdb"
export OPENAI_API_KEY="your-key"
export TAVILY_API_KEY="your-key"

# Run the app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                      # FastAPI app with endpoints
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily, arxiv, wikipedia tools
├── templates/
│   └── index.html               # Web UI template
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Postgres + Uvicorn startup
├── requirements.txt
├── Dockerfile
└── README.md
```

## Key API Endpoints

### Generate Report

Start a new research task:

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

Poll for live updates on substeps:

```bash
curl http://localhost:8000/task_progress/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "in_progress",
  "current_step": "research",
  "steps_completed": ["planning"],
  "progress": {
    "planning": "completed",
    "research": "running",
    "writing": "pending"
  }
}
```

### Get Final Report

Retrieve the completed report:

```bash
curl http://localhost:8000/task_status/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "report": "# Research Report on LLMs...",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:35:00Z"
}
```

## Core Components

### Planning Agent

Located in `src/planning_agent.py`, generates execution plan:

```python
from src.planning_agent import planner_agent

# Generate a plan from user prompt
plan = planner_agent(
    prompt="Research quantum computing applications",
    model="openai:gpt-4o"
)

# Returns structured plan:
# {
#   "steps": [
#     {"type": "research", "query": "quantum computing applications"},
#     {"type": "write", "section": "introduction"},
#     {"type": "edit", "content": "..."}
#   ]
# }
```

### Executor Agent

Execute individual plan steps:

```python
from src.planning_agent import executor_agent_step

result = executor_agent_step(
    step_type="research",
    step_data={"query": "quantum computing"},
    context={},
    model="openai:gpt-4o"
)
```

### Research Tools

Use individual tools from `src/research_tools.py`:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Web search
web_results = tavily_search_tool(
    query="quantum computing breakthroughs 2024",
    max_results=5
)

# Academic papers
papers = arxiv_search_tool(
    query="quantum machine learning",
    max_results=3
)

# Wikipedia summary
wiki_info = wikipedia_search_tool(
    query="Quantum supremacy"
)
```

### Database Models

Task tracking in Postgres:

```python
from sqlalchemy import Column, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True)
    prompt = Column(Text, nullable=False)
    status = Column(String, default="pending")
    plan = Column(Text)
    report = Column(Text)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
```

## Workflow Patterns

### Complete Research Workflow

```python
import threading
from src.planning_agent import planner_agent, executor_agent_step

def run_research_workflow(task_id, prompt, model):
    # 1. Generate plan
    plan = planner_agent(prompt=prompt, model=model)
    update_task(task_id, plan=plan, status="planning_complete")
    
    # 2. Execute each step
    context = {}
    for step in plan["steps"]:
        result = executor_agent_step(
            step_type=step["type"],
            step_data=step,
            context=context,
            model=model
        )
        context[step["type"]] = result
        update_task_progress(task_id, step["type"], "completed")
    
    # 3. Finalize report
    final_report = context.get("editor", context.get("writer", ""))
    update_task(task_id, report=final_report, status="completed")

# Run in background thread
thread = threading.Thread(
    target=run_research_workflow,
    args=(task_id, prompt, model)
)
thread.start()
```

### Custom Agent Definition

Define your own agents in `src/agents.py`:

```python
def custom_analysis_agent(query, context, model="openai:gpt-4o"):
    """
    Performs specialized analysis on research data.
    """
    prompt = f"""
    You are an expert analyst. Given the following research context:
    {context}
    
    Analyze the key findings related to: {query}
    Provide structured insights with citations.
    """
    
    # Use your LLM client (aisuite, langchain, etc.)
    response = llm_client.generate(prompt, model=model)
    return response
```

### Tool-Augmented Agent

Combine multiple tools in agent logic:

```python
from src.research_tools import tavily_search_tool, arxiv_search_tool

def comprehensive_research_agent(topic, model="openai:gpt-4o"):
    # Gather from multiple sources
    web_data = tavily_search_tool(topic, max_results=5)
    academic_data = arxiv_search_tool(topic, max_results=3)
    
    # Synthesize with LLM
    prompt = f"""
    Synthesize the following research on {topic}:
    
    Web Sources:
    {web_data}
    
    Academic Papers:
    {academic_data}
    
    Create a comprehensive summary with key insights.
    """
    
    return llm_client.generate(prompt, model=model)
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional
POSTGRES_USER=app
POSTGRES_PASSWORD=local
POSTGRES_DB=appdb
RESET_DB_ON_STARTUP=0  # Set to 1 to drop tables on startup
```

### Model Selection

Specify models when calling agents:

```python
# OpenAI GPT-4
report = generate_report(prompt="...", model="openai:gpt-4o")

# Anthropic Claude
report = generate_report(prompt="...", model="anthropic:claude-3-opus")

# Default model in agents.py
DEFAULT_MODEL = os.getenv("DEFAULT_MODEL", "openai:gpt-4o")
```

## Troubleshooting

### Container Won't Start

Check that templates exist:

```bash
docker exec -it fpsvc bash -lc "ls -l /app/templates"
```

Verify entrypoint script has execute permissions:

```bash
chmod +x docker/entrypoint.sh
```

### Database Connection Issues

Test connection string:

```bash
psql "postgresql://app:local@localhost:5432/appdb" -c "SELECT 1;"
```

If tables are missing, check startup logs for migration errors:

```bash
docker logs fpsvc | grep -i "create table"
```

### Tavily API Errors

Verify API key is set:

```python
import os
print(os.getenv("TAVILY_API_KEY"))  # Should not be None
```

Rate limit handling:

```python
def safe_tavily_search(query, max_retries=3):
    for attempt in range(max_retries):
        try:
            return tavily_search_tool(query)
        except Exception as e:
            if "rate limit" in str(e).lower():
                time.sleep(2 ** attempt)
            else:
                raise
    return None
```

### Task Stuck in Progress

Manually check task status in DB:

```sql
SELECT id, status, created_at, updated_at 
FROM tasks 
WHERE status = 'in_progress' 
  AND updated_at < NOW() - INTERVAL '10 minutes';
```

Reset stuck task:

```python
from sqlalchemy.orm import Session
from models import Task

def reset_stuck_task(session: Session, task_id: str):
    task = session.query(Task).filter_by(id=task_id).first()
    if task:
        task.status = "failed"
        task.report = "Task timed out"
        session.commit()
```

### Memory Issues with Large Reports

Stream results instead of loading all at once:

```python
def stream_research_results(task_id):
    # Store intermediate results
    for step in plan["steps"]:
        result = executor_agent_step(...)
        # Save to DB incrementally
        save_step_result(task_id, step["type"], result)
        del result  # Free memory
```

### Hot Reload for Development

Mount code volume for live updates:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --env-file .env \
  --name fpsvc \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

## Best Practices

1. **Always use environment variables** for API keys—never hardcode
2. **Run agents in background threads** to avoid blocking API responses
3. **Store intermediate results** in the database for resume capability
4. **Add retry logic** for external API calls (Tavily, arXiv, Wikipedia)
5. **Set reasonable timeouts** for each agent step (5-10 minutes max)
6. **Log all agent decisions** for debugging and transparency
7. **Validate plan structure** before executing steps
8. **Use structured output** from LLMs when possible (JSON mode)
