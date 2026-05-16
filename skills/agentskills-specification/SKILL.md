---
name: agentskills-specification
description: Specification and SDK for creating Agent Skills - a standard format for giving AI agents new capabilities through discoverable instruction bundles
triggers:
  - create an agent skill
  - write a SKILL.md file
  - build agent skills
  - agent skills specification
  - package agent capabilities
  - make my project agent-ready
  - agent skills format
  - write skills for AI agents
---

# Agent Skills Specification

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agent Skills is an open format for packaging instructions, scripts, and resources that AI agents can discover and use to perform better at specific tasks. Skills are folders containing a `SKILL.md` file with structured information that agents can load to gain expertise in specific domains or tools.

## What Agent Skills Are

Agent Skills provide:
- **Standardized format** for agent capabilities via Markdown with YAML frontmatter
- **Discoverability** through triggers and structured metadata
- **Portability** across different AI agent platforms
- **Composability** allowing agents to combine multiple skills

## Installation

Agent Skills are typically distributed as folders containing at minimum a `SKILL.md` file:

```bash
# Clone the reference implementation
git clone https://github.com/agentskills/agentskills.git

# Or see example skills
git clone https://github.com/anthropics/skills.git
```

Individual skills are installed by placing them in a directory that your AI agent monitors (implementation-specific).

## SKILL.md Format

Every Agent Skill must have a `SKILL.md` file with YAML frontmatter followed by Markdown content.

### Basic Structure

```markdown
---
name: skill-name-in-kebab-case
description: One-line description of what this skill does
triggers:
  - natural phrase users might say
  - another trigger phrase
  - how to use this skill
---

# Skill Name

Detailed documentation in Markdown format...
```

### Required Frontmatter Fields

```yaml
---
name: database-query-optimizer
description: Optimize SQL queries for PostgreSQL and MySQL databases
triggers:
  - optimize my database query
  - improve SQL performance
  - analyze query execution plan
  - find slow queries
  - index suggestions
  - database performance tuning
---
```

**Field Specifications:**
- `name`: kebab-case identifier, descriptive and unique
- `description`: Single line, under 200 characters
- `triggers`: 6-8 natural language phrases that should activate this skill

### Optional Frontmatter Fields

```yaml
---
name: docker-deployment
description: Deploy applications using Docker and Docker Compose
version: 1.0.0
author: Your Name
requires:
  - docker >= 20.10.0
  - docker-compose >= 2.0.0
platform:
  - linux
  - macos
triggers:
  - deploy with docker
  - create docker container
  - write dockerfile
  - docker compose setup
  - containerize application
  - build docker image
---
```

## Creating a Complete Skill

### 1. Start with Frontmatter

```yaml
---
name: fastapi-rest-api
description: Build REST APIs with FastAPI framework including async operations, validation, and documentation
triggers:
  - create fastapi endpoint
  - build rest api with fastapi
  - fastapi async route
  - add pydantic validation
  - setup fastapi project
  - fastapi dependency injection
version: 1.0.0
requires:
  - fastapi >= 0.100.0
  - pydantic >= 2.0.0
---
```

### 2. Add Core Documentation

```markdown
# FastAPI REST API

## Overview

FastAPI is a modern, fast web framework for building APIs with Python 3.7+ based on standard Python type hints.

## Installation

```bash
pip install fastapi uvicorn[standard]
```

## Basic Usage

### Simple Endpoint

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

### Run the Server

```bash
uvicorn main:app --reload
```
```

### 3. Include Real Code Examples

```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel, Field
from typing import List, Optional
import os

app = FastAPI(title="My API", version="1.0.0")

# Models with validation
class Item(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    description: Optional[str] = None
    price: float = Field(..., gt=0)
    tax: Optional[float] = None

class ItemResponse(BaseModel):
    id: int
    name: str
    price: float

# Dependency injection
def get_api_key(api_key: str = Depends(lambda: os.getenv("API_KEY"))):
    if not api_key:
        raise HTTPException(status_code=401, detail="API key required")
    return api_key

# CRUD endpoints
items_db = []

@app.post("/items/", response_model=ItemResponse, status_code=201)
async def create_item(item: Item, api_key: str = Depends(get_api_key)):
    item_dict = item.dict()
    item_dict["id"] = len(items_db) + 1
    items_db.append(item_dict)
    return item_dict

@app.get("/items/", response_model=List[ItemResponse])
async def list_items(skip: int = 0, limit: int = 10):
    return items_db[skip : skip + limit]

@app.get("/items/{item_id}", response_model=ItemResponse)
async def get_item(item_id: int):
    if item_id > len(items_db) or item_id < 1:
        raise HTTPException(status_code=404, detail="Item not found")
    return items_db[item_id - 1]
```

### 4. Add Configuration Section

```markdown
## Configuration

### Environment Variables

- `API_KEY`: Authentication key for protected endpoints
- `DATABASE_URL`: Connection string for database
- `LOG_LEVEL`: Logging level (DEBUG, INFO, WARNING, ERROR)

### CORS Setup

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```
```

### 5. Include Common Patterns

```markdown
## Common Patterns

### Background Tasks

```python
from fastapi import BackgroundTasks

def write_log(message: str):
    with open("log.txt", "a") as f:
        f.write(f"{message}\n")

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_log, f"Notification sent to {email}")
    return {"message": "Notification sent"}
```

### Error Handling

```python
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    return JSONResponse(
        status_code=400,
        content={"message": str(exc)},
    )
```
```

### 6. Add Troubleshooting

```markdown
## Troubleshooting

### Port Already in Use
```bash
# Change the port
uvicorn main:app --port 8001
```

### Import Errors
Ensure all dependencies are installed:
```bash
pip install -r requirements.txt
```

### Auto-reload Not Working
Use the `--reload` flag and ensure you're editing the correct file:
```bash
uvicorn main:app --reload --reload-dir ./app
```
```

## Skills Directory Structure

```
my-skill/
├── SKILL.md           # Required: Main skill documentation
├── resources/         # Optional: Additional files
│   ├── templates/
│   ├── examples/
│   └── scripts/
└── README.md          # Optional: Human-readable documentation
```

## Python SDK Usage

If there's a Python SDK for working with Agent Skills:

```python
from agentskills import Skill, SkillRegistry

# Load a skill
skill = Skill.from_file("path/to/SKILL.md")

# Access metadata
print(skill.name)
print(skill.description)
print(skill.triggers)

# Parse content
content = skill.parse_markdown()

# Registry for managing multiple skills
registry = SkillRegistry()
registry.discover("./skills")  # Auto-discover skills in directory

# Find skills by trigger
matched_skills = registry.match_trigger("create fastapi endpoint")

# Load skill content
skill_content = registry.get_skill("fastapi-rest-api")
```

## Best Practices

### 1. Descriptive Naming
Use kebab-case names that clearly identify the project or capability:
- ✅ `postgresql-database-optimization`
- ✅ `react-component-library`
- ❌ `database` (too generic)
- ❌ `myskill` (not descriptive)

### 2. Effective Triggers
Write triggers as natural phrases users would say:
```yaml
triggers:
  - optimize postgres queries  # ✅ Specific and natural
  - improve database performance  # ✅ Common phrasing
  - db perf  # ❌ Too abbreviated
  - use this skill  # ❌ Too generic
```

### 3. Real, Working Code
Always provide complete, executable examples:

```python
# ✅ Complete example with imports
import os
from typing import Optional

def connect_db(db_url: Optional[str] = None):
    url = db_url or os.getenv("DATABASE_URL")
    if not url:
        raise ValueError("DATABASE_URL not set")
    return connect(url)
```

### 4. Environment Variables for Secrets
Never include actual secrets:

```python
# ✅ Use environment variables
api_key = os.getenv("OPENAI_API_KEY")

# ❌ Never hardcode
api_key = "sk-1234567890abcdef"
```

### 5. Focus on Practical Usage
Prioritize common use cases over comprehensive API documentation:
- Installation and setup
- Most frequently used features
- Common patterns and workflows
- Troubleshooting actual problems

## Skill Discovery

Agents discover skills through:
1. **Directory scanning**: Looking for `SKILL.md` files
2. **Trigger matching**: Comparing user input to trigger phrases
3. **Context relevance**: Using skill metadata to determine applicability

## Validation

Basic validation checklist:
- [ ] YAML frontmatter is valid
- [ ] Required fields present: `name`, `description`, `triggers`
- [ ] Name is in kebab-case
- [ ] 6-8 trigger phrases provided
- [ ] Code examples are in correct language
- [ ] No hardcoded secrets
- [ ] Markdown renders correctly

## Resources

- **Specification**: https://agentskills.io/specification
- **Example Skills**: https://github.com/anthropics/skills
- **Documentation**: https://agentskills.io
- **Community**: https://discord.gg/MKPE9g8aUy

## Common Issues

### Frontmatter Not Parsing
Ensure three dashes on their own lines:
```markdown
---
name: my-skill
---
```

### Triggers Not Matching
Make triggers conversational and specific to the domain:
```yaml
triggers:
  - "create a react component"  # Better than "react"
  - "setup webpack configuration"  # Better than "webpack"
```

### Skills Not Loading
Check file structure and ensure `SKILL.md` is at the root of the skill directory.
