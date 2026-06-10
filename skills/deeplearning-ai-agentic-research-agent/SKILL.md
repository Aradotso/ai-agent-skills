---
name: deeplearning-ai-agentic-research-agent
description: Multi-agent research system with planning, tool use (Tavily, arXiv, Wikipedia), and FastAPI/Postgres backend for agentic workflows
triggers:
  - set up the agentic research agent
  - create a research workflow with planning agents
  - integrate Tavily and arXiv search tools
  - build a multi-step agent research system
  - deploy the FastAPI research agent service
  - configure the reflective research agent
  - use the agentic AI research framework
  - implement tool-using research agents
---

# DeepLearning.AI Agentic Research Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A FastAPI-based multi-agent research system that orchestrates planning, research, writing, and editing agents with tool use (Tavily search, arXiv, Wikipedia). Uses Postgres for task state management and provides a web UI for kicking off research workflows.

## What It Does

- **Planning Agent**: Decomposes research tasks into executable steps
- **Research Agents**: Use external tools (Tavily, arXiv, Wikipedia) to gather information
- **Writer & Editor Agents**: Generate and refine research reports
- **Task Management**: Postgres-backed state tracking with progress monitoring
- **REST API**: FastAPI endpoints for task submission and status checking
- **Web UI**: Simple interface for initiating research tasks

## Installation

### Docker Setup (Recommended)

**1. Clone the repository:**

```bash
git clone https://github.com/https-deeplearning-ai/agentic-ai-public.git
cd agentic-ai-public
```

**2. Create `.env` file:**

```bash
# .env
OPENAI_API_KEY=your_openai_key_here
TAVILY_API_KEY=your_tavily_key_here
```

**3. Build the Docker image:**

```bash
docker build -t fastapi-postgres-service .
```

**4. Run the container:**

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

**5. Access the service:**

- Web UI: http://localhost:8000/
- API Docs: http://localhost:8000/docs

### Local Development Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set environment variables
export DATABASE_URL="postgresql://app:local@127.0.0.1:5432/appdb"
export OPENAI_API_KEY="your_key"
export TAVILY_API_KEY="your_key"

# Run with uvicorn
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Project Structure

```
.
├── main.py                      # FastAPI application entry point
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
print(f"Task ID: {task_id}")
```

**cURL example:**

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications in drug discovery", "model":"openai:gpt-4o"}'
```

### Poll Task Progress

```python
import requests
import time

task_id = "your-task-id-here"

while True:
    response = requests.get(f"http://localhost:8000/task_progress/{task_id}")
    data = response.json()
    
    print(f"Status: {data['status']}")
    print(f"Current Step: {data.get('current_step', 'N/A')}")
    
    if data["status"] in ["completed", "failed"]:
        break
    
    time.sleep(2)
```

### Get Final Report

```python
import requests

task_id = "your-task-id-here"
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
result = response.json()

if result["status"] == "completed":
    print("Final Report:")
    print(result["report"])
else:
    print(f"Task status: {result['status']}")
```

## Creating Custom Research Tools

### Tool Definition Pattern

```python
# src/research_tools.py
import requests
import os

def tavily_search_tool(query: str, max_results: int = 5) -> dict:
    """
    Search using Tavily API for web results.
    
    Args:
        query: Search query string
        max_results: Maximum number of results to return
        
    Returns:
        Dictionary with search results
    """
    api_key = os.getenv("TAVILY_API_KEY")
    if not api_key:
        return {"error": "TAVILY_API_KEY not set"}
    
    url = "https://api.tavily.com/search"
    payload = {
        "api_key": api_key,
        "query": query,
        "max_results": max_results
    }
    
    try:
        response = requests.post(url, json=payload, timeout=30)
        response.raise_for_status()
        return response.json()
    except Exception as e:
        return {"error": str(e)}


def arxiv_search_tool(query: str, max_results: int = 5) -> list:
    """
    Search arXiv for academic papers.
    
    Args:
        query: Search query string
        max_results: Maximum number of papers to return
        
    Returns:
        List of paper dictionaries
    """
    import arxiv
    
    search = arxiv.Search(
        query=query,
        max_results=max_results,
        sort_by=arxiv.SortCriterion.Relevance
    )
    
    results = []
    for paper in search.results():
        results.append({
            "title": paper.title,
            "summary": paper.summary,
            "authors": [author.name for author in paper.authors],
            "published": paper.published.isoformat(),
            "pdf_url": paper.pdf_url
        })
    
    return results


def wikipedia_search_tool(query: str) -> dict:
    """
    Search Wikipedia and return page summary.
    
    Args:
        query: Search query string
        
    Returns:
        Dictionary with page title and summary
    """
    import wikipedia
    
    try:
        page = wikipedia.page(query, auto_suggest=True)
        return {
            "title": page.title,
            "summary": page.summary,
            "url": page.url
        }
    except wikipedia.exceptions.DisambiguationError as e:
        return {"error": "disambiguation", "options": e.options[:5]}
    except wikipedia.exceptions.PageError:
        return {"error": "page_not_found"}
    except Exception as e:
        return {"error": str(e)}
```

## Creating Custom Agents

### Research Agent Pattern

```python
# src/agents.py
from typing import Dict, List, Any
import os

def research_agent(task: str, tools: List[callable]) -> Dict[str, Any]:
    """
    Agent that conducts research using provided tools.
    
    Args:
        task: Research task description
        tools: List of tool functions to use
        
    Returns:
        Dictionary with research findings
    """
    import aisuite as ai
    
    client = ai.Client()
    
    # Build tool descriptions
    tool_descriptions = []
    for tool in tools:
        tool_descriptions.append(f"- {tool.__name__}: {tool.__doc__.strip()}")
    
    prompt = f"""You are a research agent. Your task: {task}

Available tools:
{chr(10).join(tool_descriptions)}

Conduct thorough research and provide findings."""

    response = client.chat.completions.create(
        model=os.getenv("MODEL", "openai:gpt-4o"),
        messages=[{"role": "user", "content": prompt}]
    )
    
    findings = response.choices[0].message.content
    
    # Execute tool calls based on agent's plan
    tool_results = []
    for tool in tools:
        try:
            result = tool(task)
            tool_results.append({
                "tool": tool.__name__,
                "result": result
            })
        except Exception as e:
            tool_results.append({
                "tool": tool.__name__,
                "error": str(e)
            })
    
    return {
        "agent_reasoning": findings,
        "tool_results": tool_results
    }


def writer_agent(research_findings: Dict[str, Any]) -> str:
    """
    Agent that writes a report based on research findings.
    
    Args:
        research_findings: Dictionary of research results
        
    Returns:
        Written report as string
    """
    import aisuite as ai
    
    client = ai.Client()
    
    prompt = f"""You are a technical writer. Create a comprehensive report based on these research findings:

{research_findings}

Write a well-structured, informative report."""

    response = client.chat.completions.create(
        model=os.getenv("MODEL", "openai:gpt-4o"),
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content


def editor_agent(draft_report: str) -> str:
    """
    Agent that edits and refines a report.
    
    Args:
        draft_report: Draft report text
        
    Returns:
        Edited report as string
    """
    import aisuite as ai
    
    client = ai.Client()
    
    prompt = f"""You are an editor. Review and improve this report:

{draft_report}

Edit for clarity, accuracy, and coherence. Return the final version."""

    response = client.chat.completions.create(
        model=os.getenv("MODEL", "openai:gpt-4o"),
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content
```

## Database Models

```python
# main.py or models.py
from sqlalchemy import Column, String, Text, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import datetime
import os

Base = declarative_base()

class TaskState(Base):
    __tablename__ = "task_states"
    
    task_id = Column(String, primary_key=True)
    status = Column(String, nullable=False)  # pending, running, completed, failed
    prompt = Column(Text, nullable=False)
    model = Column(String, nullable=False)
    current_step = Column(String, nullable=True)
    report = Column(Text, nullable=True)
    error = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.datetime.utcnow, onupdate=datetime.datetime.utcnow)

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:local@127.0.0.1:5432/appdb")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Create tables
Base.metadata.create_all(bind=engine)
```

## Planning Agent Pattern

```python
# src/planning_agent.py
from typing import List, Dict, Any
import os

def planner_agent(prompt: str, model: str) -> List[Dict[str, str]]:
    """
    Create a research plan broken into steps.
    
    Args:
        prompt: Research topic/question
        model: LLM model to use
        
    Returns:
        List of step dictionaries with type and description
    """
    import aisuite as ai
    
    client = ai.Client()
    
    planning_prompt = f"""You are a research planner. Break down this research task into steps:

Task: {prompt}

Create a plan with these step types:
- research: Gather information
- write: Draft content
- edit: Refine and finalize

Return a numbered list of steps."""

    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": planning_prompt}]
    )
    
    plan_text = response.choices[0].message.content
    
    # Parse plan into structured steps
    steps = []
    for line in plan_text.split('\n'):
        line = line.strip()
        if line and line[0].isdigit():
            # Extract step type and description
            if 'research' in line.lower():
                step_type = 'research'
            elif 'write' in line.lower():
                step_type = 'write'
            elif 'edit' in line.lower():
                step_type = 'edit'
            else:
                step_type = 'research'
            
            steps.append({
                'type': step_type,
                'description': line
            })
    
    return steps


def executor_agent_step(step: Dict[str, str], context: Dict[str, Any]) -> Dict[str, Any]:
    """
    Execute a single step from the plan.
    
    Args:
        step: Step dictionary with type and description
        context: Context from previous steps
        
    Returns:
        Dictionary with step results
    """
    from src.research_tools import tavily_search_tool, arxiv_search_tool, wikipedia_search_tool
    from src.agents import research_agent, writer_agent, editor_agent
    
    step_type = step['type']
    description = step['description']
    
    if step_type == 'research':
        tools = [tavily_search_tool, arxiv_search_tool, wikipedia_search_tool]
        result = research_agent(description, tools)
        context['research_findings'] = result
        return result
    
    elif step_type == 'write':
        findings = context.get('research_findings', {})
        draft = writer_agent(findings)
        context['draft_report'] = draft
        return {'draft': draft}
    
    elif step_type == 'edit':
        draft = context.get('draft_report', '')
        final = editor_agent(draft)
        context['final_report'] = final
        return {'final_report': final}
    
    return {}
```

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...              # OpenAI API key for LLM calls
TAVILY_API_KEY=tvly-...            # Tavily API for web search

# Database (auto-configured in Docker)
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb

# Optional
POSTGRES_USER=app                  # Database user (default: app)
POSTGRES_PASSWORD=local            # Database password (default: local)
POSTGRES_DB=appdb                  # Database name (default: appdb)
MODEL=openai:gpt-4o                # Default model for agents
RESET_DB_ON_STARTUP=0              # Set to 1 to drop tables on startup
```

### Database Connection from Host

```bash
# Connect to Postgres running in container
psql "postgresql://app:local@localhost:5432/appdb"

# Query task states
SELECT task_id, status, current_step, created_at FROM task_states ORDER BY created_at DESC LIMIT 10;
```

## Common Workflows

### Complete Research Workflow

```python
import requests
import time

# 1. Submit research task
response = requests.post(
    "http://localhost:8000/generate_report",
    json={
        "prompt": "Impact of transformer models on NLP",
        "model": "openai:gpt-4o"
    }
)
task_id = response.json()["task_id"]
print(f"Task started: {task_id}")

# 2. Monitor progress
while True:
    progress = requests.get(f"http://localhost:8000/task_progress/{task_id}").json()
    print(f"Status: {progress['status']} | Step: {progress.get('current_step', 'N/A')}")
    
    if progress["status"] in ["completed", "failed"]:
        break
    
    time.sleep(3)

# 3. Retrieve final report
result = requests.get(f"http://localhost:8000/task_status/{task_id}").json()

if result["status"] == "completed":
    print("\n=== Final Report ===")
    print(result["report"])
    
    # Optionally save to file
    with open(f"report_{task_id}.md", "w") as f:
        f.write(result["report"])
else:
    print(f"Task failed: {result.get('error', 'Unknown error')}")
```

### Batch Research Tasks

```python
import requests
import concurrent.futures

topics = [
    "Large language models in healthcare",
    "Quantum computing breakthroughs 2024",
    "AI alignment research progress"
]

def submit_task(topic):
    response = requests.post(
        "http://localhost:8000/generate_report",
        json={"prompt": topic, "model": "openai:gpt-4o"}
    )
    return response.json()["task_id"]

# Submit all tasks
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    task_ids = list(executor.map(submit_task, topics))

print(f"Submitted {len(task_ids)} tasks")

# Monitor all tasks
for task_id in task_ids:
    print(f"Monitoring {task_id}...")
    # Add monitoring logic here
```

## Troubleshooting

### Container Issues

**Templates not found (404 on UI):**

```bash
# Verify templates are in container
docker exec -it fpsvc ls -la /app/templates

# Check Dockerfile copies templates
# Should have: COPY templates/ /app/templates/
```

**Database connection errors:**

```bash
# Check DATABASE_URL is set
docker exec -it fpsvc env | grep DATABASE_URL

# Verify Postgres is running
docker exec -it fpsvc pg_isready -h 127.0.0.1

# Check tables exist
docker exec -it fpsvc psql postgresql://app:local@127.0.0.1:5432/appdb -c "\dt"
```

### API Key Issues

**Tavily search failing:**

```python
# Test Tavily API key
import os
import requests

api_key = os.getenv("TAVILY_API_KEY")
response = requests.post(
    "https://api.tavily.com/search",
    json={"api_key": api_key, "query": "test", "max_results": 1}
)
print(response.status_code, response.json())
```

**OpenAI rate limits:**

```python
# Add retry logic with exponential backoff
import time
from openai import RateLimitError

def call_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError:
            wait = 2 ** attempt
            print(f"Rate limited, waiting {wait}s...")
            time.sleep(wait)
    raise Exception("Max retries exceeded")
```

### Task Not Progressing

**Check task status in database:**

```bash
docker exec -it fpsvc psql postgresql://app:local@127.0.0.1:5432/appdb -c \
  "SELECT task_id, status, current_step, error FROM task_states WHERE task_id='YOUR_TASK_ID';"
```

**View application logs:**

```bash
# Follow logs in real-time
docker logs -f fpsvc

# Search for errors
docker logs fpsvc 2>&1 | grep -i error
```

### Reset Database

```bash
# Drop and recreate tables
docker exec -it fpsvc psql postgresql://app:local@127.0.0.1:5432/appdb -c \
  "DROP TABLE IF EXISTS task_states CASCADE;"

# Restart container to recreate tables
docker restart fpsvc
```

## Performance Tips

- **Concurrent requests**: FastAPI handles concurrent research tasks automatically
- **Database connection pooling**: SQLAlchemy manages connection pool (default 5 connections)
- **Tool timeouts**: Set appropriate timeouts for external APIs (default 30s)
- **Model selection**: Use `gpt-4o-mini` for faster/cheaper tasks, `gpt-4o` for complex research

## Additional Resources

- FastAPI Docs: https://fastapi.tiangolo.com/
- Tavily API: https://tavily.com/
- arXiv API: https://arxiv.org/help/api/
- Wikipedia API: https://wikipedia.readthedocs.io/
