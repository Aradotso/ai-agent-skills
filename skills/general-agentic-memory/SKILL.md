---
name: general-agentic-memory
description: Build hierarchical memory systems for AI agents using GAM (General Agentic Memory) with text, video, and long-horizon trajectory support
triggers:
  - how do I add memory to my AI agent
  - build a memory system for LLM agents
  - use GAM for agent memory management
  - create hierarchical memory for long documents
  - implement agentic memory with deep research
  - add video memory to AI agents
  - compress agent trajectories with GAM
  - query agent memory with semantic search
---

# General Agentic Memory (GAM) Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

GAM (General Agentic Memory) is a modular agentic file system framework that provides structured memory and operating environments for Large Language Models. It supports text, video, and long-horizon agent trajectories with four access methods: Python SDK, CLI, REST API, and Web Platform.

### Key Capabilities

- **Intelligent Chunking**: LLM-based semantic text segmentation
- **Memory Generation**: Structured memory summaries (Memory + TLDR) for each chunk
- **Hierarchical Organization**: Automatic taxonomy-based directory structures
- **Incremental Updates**: Append new content without rebuilding
- **Multi-modal**: Text documents, videos, and agent trajectories
- **Flexible Backends**: OpenAI, SGLang, and other inference engines

## Installation

```bash
# Full installation with all features
pip install -e ".[all]"

# Or minimal installation
pip install -e .
```

## Configuration

GAM uses environment variables for API configuration. Set these to avoid repeated parameter input:

```bash
# GAM Agent (memory building)
export GAM_API_KEY="sk-your-api-key"
export GAM_MODEL="gpt-4o-mini"
export GAM_API_BASE="https://api.openai.com/v1"

# Chat Agent (Q&A) — falls back to GAM Agent config when not set
export GAM_CHAT_API_KEY="sk-your-chat-api-key"
export GAM_CHAT_MODEL="gpt-4o"
export GAM_CHAT_API_BASE="https://api.openai.com/v1"
```

Alternatively, pass configuration directly in code or CLI commands.

## Python SDK Usage

### Basic Workflow API

The `Workflow` class provides the simplest interface:

```python
from gam import Workflow

# Initialize workflow for text processing
wf = Workflow(
    task_type="text",
    gam_dir="./my_text_gam",
    model="gpt-4o-mini",
    api_key=None  # Uses GAM_API_KEY env var
)

# Add content to memory
wf.add(input_file="research_paper.pdf")

# Query the memory
result = wf.request("What is the main conclusion of this paper?")
print(result.answer)
print(result.sources)  # Retrieved memory chunks
```

### Video Memory Workflow

```python
from gam import Workflow

# Initialize video workflow
wf = Workflow(
    task_type="video",
    gam_dir="./my_video_gam",
    model="gpt-4o-mini"
)

# Add video content
wf.add(input_file="lecture.mp4")

# Query video memory
result = wf.request("What topics are covered in this lecture?")
print(result.answer)
```

### Long-Horizon Agent Trajectories

```python
from gam import Workflow

# Initialize trajectory workflow
wf = Workflow(
    task_type="long-horizon",
    gam_dir="./agent_trajectory_gam",
    model="gpt-4o-mini"
)

# Add agent trajectory log
wf.add(input_file="agent_execution.jsonl")

# Query the trajectory
result = wf.request("What tools did the agent use to solve the task?")
print(result.answer)
```

### Incremental Memory Addition

```python
from gam import Workflow

wf = Workflow(task_type="text", gam_dir="./my_gam")

# Add initial content
wf.add(input_file="document1.pdf")

# Later, add more content incrementally
wf.add(input_file="document2.pdf")
wf.add(input_file="document3.txt")

# Query across all added content
result = wf.request("Compare the approaches in all three documents")
```

### Advanced: Using Individual Components

```python
from gam.text.chunker import TextChunker
from gam.text.memory_builder import MemoryBuilder
from gam.text.taxonomy_builder import TaxonomyBuilder
from gam.text.chat_agent import ChatAgent

# Step 1: Chunk text
chunker = TextChunker(model="gpt-4o-mini")
chunks = chunker.chunk(text="Long document text here...")

# Step 2: Build memories
memory_builder = MemoryBuilder(model="gpt-4o-mini")
memories = memory_builder.build(chunks)

# Step 3: Create taxonomy
taxonomy_builder = TaxonomyBuilder(model="gpt-4o-mini")
taxonomy = taxonomy_builder.build(memories)

# Step 4: Save to GAM directory
gam_dir = "./my_gam"
taxonomy.save(gam_dir)

# Step 5: Query
chat_agent = ChatAgent(
    gam_dir=gam_dir,
    model="gpt-4o",
    task_type="text"
)
answer = chat_agent.request("Your question here")
print(answer)
```

### Custom LLM Backend

```python
from gam import Workflow

# Use custom API endpoint (e.g., local vLLM server)
wf = Workflow(
    task_type="text",
    gam_dir="./my_gam",
    model="meta-llama/Llama-3-8B",
    api_base="http://localhost:8000/v1",
    api_key="EMPTY"  # Some local servers don't require keys
)

wf.add(input_file="document.pdf")
result = wf.request("Summarize this document")
```

## CLI Usage

### Adding Content with `gam-add`

```bash
# Add text document
gam-add --type text \
  --gam-dir ./my_gam \
  --input research_paper.pdf \
  --model gpt-4o-mini

# Add video
gam-add --type video \
  --gam-dir ./video_gam \
  --input lecture.mp4

# Add long-horizon trajectory
gam-add --type long-horizon \
  --gam-dir ./trajectory_gam \
  --input agent_log.jsonl

# Use environment variables for API config
export GAM_API_KEY="sk-xxx"
export GAM_MODEL="gpt-4o-mini"
gam-add --type text --gam-dir ./my_gam --input document.txt
```

### Querying with `gam-request`

```bash
# Query text memory
gam-request --type text \
  --gam-dir ./my_gam \
  --question "What is the main conclusion?" \
  --model gpt-4o

# Query video memory
gam-request --type video \
  --gam-dir ./video_gam \
  --question "What happens at 5 minutes?"

# Query with custom chat model
export GAM_CHAT_MODEL="gpt-4o"
export GAM_CHAT_API_KEY="sk-xxx"
gam-request --type text \
  --gam-dir ./my_gam \
  --question "Summarize the key findings"
```

### CLI Options

Common options for both `gam-add` and `gam-request`:

- `--type`: Task type (`text`, `video`, `long-horizon`)
- `--gam-dir`: Directory to store/read GAM memory
- `--model`: LLM model name
- `--api-key`: API key (or use `GAM_API_KEY` env var)
- `--api-base`: API base URL (or use `GAM_API_BASE` env var)

## REST API Usage

### Starting the Server

```python
# examples/run_api.py
from gam.api import create_app
import uvicorn

app = create_app()

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=5001)
```

```bash
# Run the API server
python examples/run_api.py --port 5001

# Interactive API docs available at:
# http://localhost:5001/docs
```

### Using the API

```python
import requests

API_BASE = "http://localhost:5001"

# Add content
add_response = requests.post(
    f"{API_BASE}/add",
    json={
        "task_type": "text",
        "gam_dir": "./my_gam",
        "input_file": "document.pdf",
        "model": "gpt-4o-mini",
        "api_key": None  # Uses server's env vars
    }
)
print(add_response.json())

# Query memory
query_response = requests.post(
    f"{API_BASE}/request",
    json={
        "task_type": "text",
        "gam_dir": "./my_gam",
        "question": "What are the key findings?",
        "model": "gpt-4o"
    }
)
result = query_response.json()
print(result["answer"])
print(result["sources"])
```

### API Endpoints

- `POST /add`: Add content to a GAM
- `POST /request`: Query a GAM
- `GET /health`: Health check
- `GET /docs`: Interactive API documentation (Swagger UI)
- `GET /redoc`: Alternative API documentation

## Web Interface

```bash
# Start web interface
python examples/run_web.py \
  --model gpt-4o-mini \
  --port 5000

# Access at http://localhost:5000
```

The web interface provides:
- Visual GAM management
- File upload for text/video/trajectories
- Interactive Q&A interface
- Memory exploration and visualization

## Common Patterns

### Multi-Document Knowledge Base

```python
from gam import Workflow

# Create a knowledge base from multiple documents
wf = Workflow(task_type="text", gam_dir="./knowledge_base")

documents = [
    "research/paper1.pdf",
    "research/paper2.pdf",
    "research/paper3.pdf",
    "notes/summary.txt"
]

for doc in documents:
    wf.add(input_file=doc)

# Cross-document queries
result = wf.request("Compare the methodologies across all papers")
```

### Agent Trajectory Compression

```python
from gam import Workflow

# Compress long agent execution traces
wf = Workflow(task_type="long-horizon", gam_dir="./agent_memory")

# Add trajectory
wf.add(input_file="agent_trace.jsonl")

# Query specific actions
result = wf.request("What API calls did the agent make?")

# Query reasoning
result = wf.request("Why did the agent choose this approach?")
```

### Video Analysis Pipeline

```python
from gam import Workflow

# Build video memory
wf = Workflow(task_type="video", gam_dir="./video_memory")
wf.add(input_file="tutorial.mp4")

# Time-based queries
result = wf.request("What is demonstrated in the first 10 minutes?")

# Content-based queries
result = wf.request("Find all mentions of error handling")
```

### Custom Memory Organization

```python
from gam.text.taxonomy_builder import TaxonomyBuilder
from gam.text.memory_builder import MemoryBuilder

# Build memories with custom chunking
memory_builder = MemoryBuilder(model="gpt-4o-mini")
memories = memory_builder.build(your_chunks)

# Organize with custom taxonomy strategy
taxonomy_builder = TaxonomyBuilder(
    model="gpt-4o-mini",
    max_depth=4  # Control hierarchy depth
)
taxonomy = taxonomy_builder.build(memories)

# Save to specific location
taxonomy.save("./custom_gam")
```

## Troubleshooting

### API Key Issues

**Problem**: `AuthenticationError` or missing API key

**Solution**: Ensure environment variables are set:

```bash
export GAM_API_KEY="sk-your-key"
export GAM_MODEL="gpt-4o-mini"

# Verify
echo $GAM_API_KEY
```

Or pass explicitly in code:

```python
wf = Workflow(
    task_type="text",
    gam_dir="./my_gam",
    api_key="sk-your-key",  # Explicit key
    model="gpt-4o-mini"
)
```

### Model Not Found

**Problem**: Model name not recognized by API

**Solution**: Check model availability with your API provider:

```python
# For OpenAI
wf = Workflow(model="gpt-4o-mini")  # Correct

# For local vLLM
wf = Workflow(
    model="meta-llama/Llama-3-8B",  # Full model path
    api_base="http://localhost:8000/v1"
)
```

### Empty or Invalid Responses

**Problem**: GAM returns empty results or errors during querying

**Solution**: Verify GAM directory structure:

```python
import os

gam_dir = "./my_gam"
if not os.path.exists(gam_dir):
    print("GAM directory doesn't exist - need to run add() first")

# Check for memory files
if not os.path.exists(f"{gam_dir}/taxonomy.json"):
    print("No taxonomy found - GAM may be corrupted")
```

### Video Processing Failures

**Problem**: Video GAM fails during processing

**Solution**: Ensure video dependencies are installed:

```bash
pip install -e ".[all]"  # Includes video dependencies

# Verify ffmpeg is available (required for video)
which ffmpeg
```

### Performance Issues with Large Documents

**Problem**: Memory building takes too long

**Solution**: Use more capable models for building, lighter models for querying:

```python
# Use powerful model for memory building (one-time cost)
wf = Workflow(
    task_type="text",
    gam_dir="./my_gam",
    model="gpt-4o"  # Better chunking and summarization
)
wf.add(input_file="large_document.pdf")

# Use efficient model for queries (frequent operation)
from gam.text.chat_agent import ChatAgent
chat = ChatAgent(
    gam_dir="./my_gam",
    model="gpt-4o-mini",  # Faster and cheaper
    task_type="text"
)
```

### Docker Environment Issues

**Problem**: Running GAM in containers

**Solution**: Mount GAM directory as volume:

```bash
docker run -v $(pwd)/my_gam:/app/my_gam \
  -e GAM_API_KEY="sk-xxx" \
  -e GAM_MODEL="gpt-4o-mini" \
  your-image
```

## Best Practices

### Memory Organization

- Use descriptive `gam_dir` names for different projects/topics
- Keep related documents in the same GAM for better cross-referencing
- Rebuild GAM when document structure changes significantly

### Model Selection

- **Building memory**: Use `gpt-4o` or `gpt-4o-mini` for quality
- **Querying**: Use `gpt-4o-mini` for cost-effectiveness
- **Local inference**: Use SGLang or vLLM for privacy/cost

### Incremental Updates

```python
# Good: Add documents incrementally
wf = Workflow(task_type="text", gam_dir="./docs")
wf.add(input_file="doc1.pdf")
wf.add(input_file="doc2.pdf")

# Avoid: Rebuilding entire GAM for new documents
# (GAM handles incremental addition efficiently)
```

### Error Handling

```python
from gam import Workflow

try:
    wf = Workflow(task_type="text", gam_dir="./my_gam")
    wf.add(input_file="document.pdf")
    result = wf.request("What is this about?")
    print(result.answer)
except Exception as e:
    print(f"Error: {e}")
    # Handle appropriately (retry, log, etc.)
```

## Research Implementation

For academic benchmarking and the original dual-agent implementation:

```bash
cd research
pip install -e .
```

```python
from gam_research import MemoryAgent, ResearchAgent

# Use research implementation
memory_agent = MemoryAgent(model="gpt-4o")
research_agent = ResearchAgent(model="gpt-4o")
```

See `research/README.md` for benchmark evaluation scripts.
