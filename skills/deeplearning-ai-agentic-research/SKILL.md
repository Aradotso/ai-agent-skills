---
name: deeplearning-ai-agentic-research
description: FastAPI research agent service with multi-step planning, tool use (Tavily, arXiv, Wikipedia), and PostgreSQL state management
triggers:
  - set up agentic research workflow
  - create a research agent with planning capabilities
  - build multi-step AI research service
  - use Tavily and arXiv for agent research
  - implement reflective research agents
  - deploy FastAPI agent with database
  - create research workflow with planner and executor
  - build AI research report generator
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic AI Research service is a FastAPI application that implements a multi-agent research workflow with planning, execution, and reflection capabilities. It orchestrates specialized agents (planner, researcher, writer, editor) to autonomously research topics using tools like Tavily search, arXiv, and Wikipedia, then generates comprehensive reports. All task state and results are persisted in PostgreSQL.

Key features:
- **Planning Agent**: Decomposes research tasks into executable steps
- **Research Tools**: Tavily web search, arXiv academic papers, Wikipedia lookups
- **Multi-Agent Workflow**: Specialized agents for research, writing, and editing
- **State Persistence**: PostgreSQL tracks task progress and substeps
- **Async Execution**: Background threads handle long-running research tasks
- **REST API + Web UI**: Simple interface to kick off and monitor research

## Installation

### Docker (Recommended)

The project runs Postgres + FastAPI in a single container for local development.

```bash
# Clone the repository
git clone https://github.com/deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public

# Create .env file with API keys
cat > .env << EOF
OPENAI_API_KEY=your-openai-key
TAVILY_API_KEY=your-tavily-key
EOF

# Build the Docker image
docker build -t fastapi-postgres-service .

# Run the container
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

# Set up environment variables
export OPENAI_API_KEY=your-openai-key
export TAVILY_API_KEY=your-tavily-key
export DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Start PostgreSQL separately
# Then run the app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                      # FastAPI app, routes, database models
├── src/
│   ├── planning_agent.py        # planner_agent(), executor_agent_step()
│   ├── agents.py                # research_agent, writer_agent, editor_agent
│   └── research_tools.py        # tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
├── templates/
│   └── index.html               # Web UI template
├── static/                      # CSS/JS assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt
├── Dockerfile
└── README.md
```

## API Reference

### Generate Research Report

Start a new research task with automatic workflow planning and execution.

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

**cURL:**
```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications in drug discovery", "model":"openai:gpt-4o"}'
```

### Monitor Task Progress

Get real-time status of each step and substep in the workflow.

```python
import requests
import time

task_id = "your-task-uuid"

while True:
    response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    data = response.json()
    
    print(f"Status: {data['status']}")
    print(f"Current Step: {data['current_step']}")
    
    if data["status"] in ["completed", "failed"]:
        break
    
    time.sleep(2)
```

### Get Final Report

Retrieve the completed research report and metadata.

```python
import requests

task_id = "your-task-uuid"
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result["status"] == "completed":
    print("Final Report:")
    print(result["report"])
    print(f"\nCompleted in {result['execution_time']}s")
```

## Core Components

### Planning Agent

The planning agent breaks down research prompts into executable steps.

```python
from src.planning_agent import planner_agent

# Generate a research plan
plan = planner_agent(
    user_prompt="Analyze recent developments in transformer architectures",
    model="openai:gpt-4o"
)

# Plan contains structured steps
for step in plan["steps"]:
    print(f"Step {step['step_number']}: {step['description']}")
    print(f"  Agent: {step['agent']}")
    print(f"  Expected output: {step['expected_output']}")
```

### Research Tools

Three built-in tools for information gathering:

```python
from src.research_tools import (
    tavily_search_tool,
    arxiv_search_tool,
    wikipedia_search_tool
)

# Web search via Tavily
web_results = tavily_search_tool(
    query="latest developments in reinforcement learning from human feedback"
)

# Academic papers from arXiv
papers = arxiv_search_tool(
    query="transformer attention mechanisms",
    max_results=5
)

# Wikipedia summaries
wiki_info = wikipedia_search_tool(
    query="neural architecture search"
)
```

### Specialized Agents

Each agent has a specific role in the research workflow:

```python
from src.agents import research_agent, writer_agent, editor_agent

# Research agent: gathers and synthesizes information
research_output = research_agent(
    task="Find recent papers on multimodal learning",
    context={"previous_findings": "..."}
)

# Writer agent: creates structured content
draft = writer_agent(
    task="Write an introduction to the research findings",
    research_data=research_output
)

# Editor agent: refines and polishes output
final_report = editor_agent(
    draft=draft,
    guidelines="Academic tone, cite sources"
)
```

### Executor Agent

Runs individual steps from the plan:

```python
from src.planning_agent import executor_agent_step

step_result = executor_agent_step(
    step={
        "step_number": 1,
        "description": "Research transformer architectures",
        "agent": "research_agent",
        "expected_output": "Summary of key papers"
    },
    context={},
    model="openai:gpt-4o"
)

print(step_result["output"])
```

## Configuration

### Environment Variables

Required:
```bash
OPENAI_API_KEY=sk-...          # OpenAI API key for LLM calls
TAVILY_API_KEY=tvly-...        # Tavily API key for web search
```

Optional:
```bash
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
POSTGRES_USER=app              # Database user (default: app)
POSTGRES_PASSWORD=local        # Database password (default: local)
POSTGRES_DB=appdb              # Database name (default: appdb)
RESET_DB_ON_STARTUP=0          # Set to 1 to drop tables on startup
```

### Database Schema

The app uses SQLAlchemy models for task tracking:

```python
from sqlalchemy import Column, String, Integer, DateTime, Text
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class ResearchTask(Base):
    __tablename__ = "research_tasks"
    
    task_id = Column(String, primary_key=True)
    prompt = Column(Text)
    status = Column(String)  # pending, running, completed, failed
    current_step = Column(Integer)
    report = Column(Text)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
```

## Common Patterns

### End-to-End Research Workflow

```python
import requests
import time
import json

# 1. Start research task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Impact of federated learning on privacy-preserving AI",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]

# 2. Poll for completion
while True:
    progress = requests.get(
        f"http://localhost:8000/task_progress/{task_id}"
    ).json()
    
    print(f"Step {progress['current_step']}: {progress['status']}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(5)

# 3. Retrieve final report
result = requests.get(
    f"http://localhost:8000/task_status/{task_id}"
).json()

with open("research_report.txt", "w") as f:
    f.write(result["report"])

print(f"Report saved! Took {result['execution_time']}s")
```

### Custom Agent Implementation

Extend the agent system with custom agents:

```python
from aisuite import Client

def custom_analysis_agent(task, context, model="openai:gpt-4o"):
    """Custom agent for data analysis tasks"""
    
    client = Client()
    
    system_prompt = """You are a data analysis expert.
    Analyze the provided data and extract key insights.
    Use statistical reasoning and cite specific numbers."""
    
    user_message = f"""
    Task: {task}
    
    Context: {json.dumps(context)}
    
    Provide a detailed analysis with:
    1. Key findings
    2. Statistical evidence
    3. Recommendations
    """
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_message}
        ],
        temperature=0.3
    )
    
    return {
        "output": response.choices[0].message.content,
        "agent": "custom_analysis_agent"
    }
```

### Direct Database Access

Query task state directly from Python:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import os

engine = create_engine(os.getenv("DATABASE_URL"))
Session = sessionmaker(bind=engine)
session = Session()

# Find all completed tasks
from main import ResearchTask

completed_tasks = session.query(ResearchTask).filter(
    ResearchTask.status == "completed"
).all()

for task in completed_tasks:
    print(f"Task: {task.prompt}")
    print(f"Completed: {task.updated_at}")
    print(f"Report length: {len(task.report)} chars")
    print("---")
```

## Troubleshooting

### Postgres Connection Issues

**Error:** `DATABASE_URL not set`

The entrypoint should set this automatically. If running locally:
```bash
export DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
```

**Error:** `FATAL: password authentication failed`

Use UNIX socket authentication in Docker or verify credentials:
```bash
# Inside container
su -s /bin/bash postgres -c "psql -c 'ALTER USER app WITH PASSWORD '\''local'\'';'"
```

### Tavily API Errors

**Error:** `401 Unauthorized` from Tavily

Ensure your API key is set:
```bash
echo $TAVILY_API_KEY  # Should show your key
```

Add to `.env`:
```
TAVILY_API_KEY=tvly-your-actual-key
```

### Template Not Found

**Error:** `TemplateNotFound: index.html`

Verify templates are copied into the Docker image. In `Dockerfile`:
```dockerfile
COPY templates/ /app/templates/
COPY static/ /app/static/
```

Check inside container:
```bash
docker exec -it fpsvc ls -la /app/templates
```

### Agent Timeout Issues

For long-running research tasks, increase timeouts:

```python
# In main.py, adjust thread timeout or add progress checkpoints
def run_research_workflow(task_id, prompt, model):
    try:
        # Add timeout handling
        import signal
        signal.alarm(1800)  # 30 minute timeout
        
        # ... existing workflow code ...
        
    except TimeoutError:
        update_task_status(task_id, "failed", error="Timeout after 30 minutes")
```

### Wikipedia Rate Limiting

**Error:** `HTTPError 429` from Wikipedia

Add retry logic with exponential backoff:

```python
import time
from requests.exceptions import HTTPError

def wikipedia_search_tool(query, retries=3):
    for attempt in range(retries):
        try:
            # ... existing Wikipedia search code ...
            return results
        except HTTPError as e:
            if e.response.status_code == 429 and attempt < retries - 1:
                wait_time = 2 ** attempt
                time.sleep(wait_time)
                continue
            raise
```

### Database Schema Reset

To reset tables without restarting:

```bash
# Connect to database
docker exec -it fpsvc psql "postgresql://app:local@127.0.0.1:5432/appdb"

# Drop and recreate
DROP TABLE research_tasks;
DROP TABLE task_steps;
# Then restart app to recreate
```

Or use the environment variable:
```bash
docker run -e RESET_DB_ON_STARTUP=1 ...
```

## Advanced Usage

### Stream Progress Updates

Implement SSE (Server-Sent Events) for real-time updates:

```python
from fastapi import FastAPI
from sse_starlette.sse import EventSourceResponse

@app.get("/task_stream/{task_id}")
async def stream_task_progress(task_id: str):
    async def event_generator():
        while True:
            task = get_task_status(task_id)
            yield {
                "event": "progress",
                "data": json.dumps({
                    "status": task.status,
                    "current_step": task.current_step
                })
            }
            if task.status in ["completed", "failed"]:
                break
            await asyncio.sleep(1)
    
    return EventSourceResponse(event_generator())
```

### Parallel Research Execution

Run multiple research tools concurrently:

```python
import concurrent.futures

def parallel_research(query):
    with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
        futures = {
            executor.submit(tavily_search_tool, query): "tavily",
            executor.submit(arxiv_search_tool, query): "arxiv",
            executor.submit(wikipedia_search_tool, query): "wikipedia"
        }
        
        results = {}
        for future in concurrent.futures.as_completed(futures):
            source = futures[future]
            try:
                results[source] = future.result()
            except Exception as e:
                results[source] = {"error": str(e)}
        
        return results
```
