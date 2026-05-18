---
name: gam-agentic-memory
description: Build structured hierarchical memory systems for LLM agents using GAM (General Agentic Memory) with support for text, video, and agent trajectories
triggers:
  - how do I create a memory system for my agent
  - set up GAM for long document processing
  - build hierarchical memory with general agentic memory
  - add video memory to my AI agent
  - use GAM to handle long context
  - implement agent trajectory compression
  - query GAM memory with natural language
  - integrate agentic memory into my workflow
---

# GAM (General Agentic Memory) Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

GAM is a modular agentic file system framework that provides structured memory and operating environments for Large Language Models. It automatically chunks content, generates memory summaries, and organizes them hierarchically. Supports text documents, long videos, and agent trajectories with access via Python SDK, CLI, REST API, and Web interface.

## Installation

```bash
# Full installation with all features
pip install -e ".[all]"

# Basic installation (text only)
pip install -e .

# With specific features
pip install -e ".[video]"  # Video support
pip install -e ".[api]"    # REST API support
pip install -e ".[web]"    # Web interface support
```

## Configuration

GAM uses environment variables for LLM configuration. Set these to avoid passing API keys in code:

```bash
# GAM Agent (for memory building)
export GAM_API_KEY="sk-your-api-key"
export GAM_MODEL="gpt-4o-mini"
export GAM_API_BASE="https://api.openai.com/v1"

# Chat Agent (for Q&A) — optional, falls back to GAM Agent config
export GAM_CHAT_API_KEY="sk-your-chat-api-key"
export GAM_CHAT_MODEL="gpt-4o"
export GAM_CHAT_API_BASE="https://api.openai.com/v1"
```

## Python SDK Usage

### Basic Workflow API

The `Workflow` class is the primary high-level interface:

```python
from gam import Workflow

# Create a text workflow
wf = Workflow(
    task_type="text",
    gam_dir="./my_gam",
    model="gpt-4o-mini",  # Optional if GAM_MODEL is set
    api_key=None  # Uses GAM_API_KEY env var
)

# Add content to GAM
wf.add(input_file="research_paper.pdf")

# Query the memory
result = wf.request("What is the main conclusion of this paper?")
print(result.answer)
print(result.context)  # Retrieved memory context
```

### Incremental Memory Building

Add new content to existing GAM without rebuilding:

```python
from gam import Workflow

wf = Workflow("text", gam_dir="./knowledge_base")

# Add initial document
wf.add(input_file="doc1.pdf")

# Later, add more documents incrementally
wf.add(input_file="doc2.pdf")
wf.add(input_file="doc3.txt")

# Query across all added documents
result = wf.request("Summarize the key points from all documents")
```

### Video Memory Workflow

```python
from gam import Workflow

# Create video workflow
wf = Workflow(
    task_type="video",
    gam_dir="./video_gam",
    model="gpt-4o-mini"
)

# Add video content
wf.add(input_file="lecture.mp4")

# Query video content
result = wf.request("What topics were covered in the first 10 minutes?")
print(result.answer)
```

### Long-Horizon Agent Trajectory

```python
from gam import Workflow

# Create long-horizon workflow for agent trajectories
wf = Workflow(
    task_type="long-horizon",
    gam_dir="./agent_memory",
    model="gpt-4o-mini"
)

# Add agent trajectory data
wf.add(input_file="agent_trace.jsonl")

# Query trajectory
result = wf.request("What tool was used to solve the math problem?")
```

### Advanced Component Usage

For more control, use individual components:

```python
from gam.text.core import TextGAM
from gam.text.chat import TextChatAgent

# Initialize GAM with custom config
gam = TextGAM(
    gam_dir="./custom_gam",
    model="gpt-4o-mini",
    api_key=None,  # Uses env var
    chunk_size=2000,
    max_workers=4
)

# Build memory from text
gam.add(input_file="document.pdf")

# Initialize chat agent with different model
chat_agent = TextChatAgent(
    gam_dir="./custom_gam",
    model="gpt-4o",  # More powerful model for Q&A
    api_key=None,
    top_k=10,  # Number of memory chunks to retrieve
    rerank=True  # Enable reranking
)

# Query
response = chat_agent.chat("What are the main findings?")
print(response)
```

## CLI Usage

### Adding Content to GAM

```bash
# Add text document
gam-add --type text --gam-dir ./my_gam --input paper.pdf

# Add with custom model
gam-add --type text --gam-dir ./my_gam --input paper.pdf --model gpt-4o

# Add video
gam-add --type video --gam-dir ./video_gam --input lecture.mp4

# Add agent trajectory
gam-add --type long-horizon --gam-dir ./agent_gam --input trace.jsonl
```

### Querying GAM

```bash
# Basic query
gam-request --type text --gam-dir ./my_gam --question "What is the main conclusion?"

# Query with custom chat model
gam-request --type text --gam-dir ./my_gam --question "Explain the methodology" \
  --model gpt-4o --top-k 15

# Video query
gam-request --type video --gam-dir ./video_gam --question "Summarize the lecture"

# Long-horizon query
gam-request --type long-horizon --gam-dir ./agent_gam --question "What actions were taken?"
```

### CLI with Environment Variables

```bash
# Set once
export GAM_API_KEY="sk-your-key"
export GAM_MODEL="gpt-4o-mini"

# Then use without specifying credentials
gam-add --type text --gam-dir ./my_gam --input doc.pdf
gam-request --type text --gam-dir ./my_gam --question "Summary?"
```

## REST API Usage

### Starting the API Server

```python
# examples/run_api.py
from gam.api import create_app
import uvicorn

app = create_app()

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=5001)
```

```bash
# Start server
python examples/run_api.py --port 5001

# Interactive API docs at http://localhost:5001/docs
```

### REST API Client Example

```python
import requests

BASE_URL = "http://localhost:5001"

# Add content via API
add_response = requests.post(
    f"{BASE_URL}/add",
    json={
        "task_type": "text",
        "gam_dir": "./api_gam",
        "input_file": "document.pdf",
        "model": "gpt-4o-mini"
    }
)
print(add_response.json())

# Query via API
query_response = requests.post(
    f"{BASE_URL}/request",
    json={
        "task_type": "text",
        "gam_dir": "./api_gam",
        "question": "What are the key findings?",
        "model": "gpt-4o",
        "top_k": 10
    }
)
result = query_response.json()
print(result["answer"])
print(result["context"])
```

### REST API Endpoints

- `POST /add` - Add content to GAM
- `POST /request` - Query GAM memory
- `GET /health` - Health check
- `GET /docs` - Interactive API documentation

## Web Interface Usage

### Starting the Web Platform

```python
# examples/run_web.py
from gam.web import app

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```

```bash
# Start with environment variables
export GAM_MODEL="gpt-4o-mini"
export GAM_API_KEY="sk-your-key"
python examples/run_web.py

# Or pass directly
python examples/run_web.py --model gpt-4o-mini --api-key sk-your-key --port 5000
```

Access at `http://localhost:5000` for visual GAM management and querying.

## Common Patterns

### Multi-Document Knowledge Base

```python
from gam import Workflow

# Create knowledge base
kb = Workflow("text", gam_dir="./knowledge_base")

# Add multiple documents
documents = ["finance.pdf", "legal.pdf", "tech.pdf"]
for doc in documents:
    kb.add(input_file=doc)

# Query across all documents
result = kb.request("What are the legal implications mentioned in any document?")
```

### Video Analysis Pipeline

```python
from gam import Workflow

# Process video lectures
wf = Workflow("video", gam_dir="./lectures")

# Add multiple lectures
wf.add(input_file="lecture1.mp4")
wf.add(input_file="lecture2.mp4")

# Ask questions about content
topics = wf.request("What are all the topics covered?")
timeline = wf.request("When was machine learning discussed?")
```

### Agent Memory with Search/Recall

```python
from gam import Workflow

# Agent trajectory memory
agent_mem = Workflow("long-horizon", gam_dir="./agent_traces")

# Add agent execution trace
agent_mem.add(input_file="execution_log.jsonl")

# Recall specific actions
result = agent_mem.request("What was the sequence of tool calls for solving the problem?")

# Search for patterns
patterns = agent_mem.request("How many times did the agent retry failed operations?")
```

### Custom Memory Configuration

```python
from gam.text.core import TextGAM
from gam.text.chat import TextChatAgent

# High-performance memory building
gam = TextGAM(
    gam_dir="./optimized_gam",
    model="gpt-4o-mini",
    chunk_size=3000,  # Larger chunks
    max_workers=8,     # More parallel workers
    embedding_model="text-embedding-3-large"
)

gam.add(input_file="large_corpus.txt")

# Precision-focused retrieval
chat = TextChatAgent(
    gam_dir="./optimized_gam",
    model="gpt-4o",
    top_k=20,         # Retrieve more candidates
    rerank=True,      # Enable reranking
    temperature=0.0   # Deterministic responses
)

response = chat.chat("Find all mentions of quantum computing")
```

### Docker Workspace Support

```python
from gam import Workflow

# Create GAM in Docker container workspace
wf = Workflow(
    task_type="text",
    gam_dir="/workspace/gam",  # Container path
    model="gpt-4o-mini"
)

# Works seamlessly in containerized environments
wf.add(input_file="/workspace/data/document.pdf")
result = wf.request("Summarize the document")
```

## Troubleshooting

### API Key Issues

If you get authentication errors:

```python
# Check environment variables
import os
print(os.getenv("GAM_API_KEY"))
print(os.getenv("GAM_MODEL"))

# Or pass explicitly
wf = Workflow("text", gam_dir="./gam", model="gpt-4o-mini", api_key="sk-your-key")
```

### Missing Dependencies

```bash
# If video processing fails
pip install -e ".[video]"

# If API server fails
pip install -e ".[api]"

# If web interface fails
pip install -e ".[web]"

# Install everything
pip install -e ".[all]"
```

### Memory Not Found

```python
# Ensure GAM directory exists and has content
import os
gam_dir = "./my_gam"

if not os.path.exists(gam_dir):
    print("GAM directory not found. Run add() first.")

if not os.path.exists(f"{gam_dir}/chunks"):
    print("No chunks found. GAM may be empty.")
```

### Slow Performance

```python
# Increase parallel workers for faster building
from gam.text.core import TextGAM

gam = TextGAM(
    gam_dir="./gam",
    max_workers=16,  # More workers
    chunk_size=2500  # Adjust chunk size
)

# Use smaller, faster model for building
gam = TextGAM(gam_dir="./gam", model="gpt-4o-mini")

# Use larger model only for querying
from gam.text.chat import TextChatAgent
chat = TextChatAgent(gam_dir="./gam", model="gpt-4o")
```

### Model Configuration

```python
# Use different models for different stages
from gam import Workflow

# Fast model for memory building
wf = Workflow("text", gam_dir="./gam", model="gpt-4o-mini")
wf.add(input_file="doc.pdf")

# Advanced model for complex queries
from gam.text.chat import TextChatAgent
chat = TextChatAgent(gam_dir="./gam", model="gpt-4o")
response = chat.chat("Complex analytical question?")
```

## Directory Structure

After building a GAM, the structure looks like:

```
./my_gam/
├── chunks/          # Segmented content chunks
├── memories/        # Generated memory summaries
├── taxonomy/        # Hierarchical organization
├── embeddings/      # Vector embeddings for retrieval
└── metadata.json    # GAM configuration
```

## Research Codebase

For academic research and benchmarking:

```bash
cd research
pip install -e .
```

```python
from gam_research import MemoryAgent, ResearchAgent

# Dual-agent implementation from paper
memorizer = MemoryAgent(gam_dir="./research_gam")
researcher = ResearchAgent(gam_dir="./research_gam")

# Run benchmarks (LoCoMo, HotpotQA, RULER, NarrativeQA)
# See research/README.md for details
```

## Key Concepts

- **GAM Directory**: File system location where memory is stored
- **Chunks**: Semantically segmented pieces of content
- **Memories**: LLM-generated summaries with TLDR for each chunk
- **Taxonomy**: Hierarchical directory structure organizing memories
- **Workflow**: High-level API combining add and request operations
- **Incremental Addition**: Add new content without rebuilding existing memory
