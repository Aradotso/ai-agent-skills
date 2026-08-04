---
name: nvidia-oo-agents
description: Build AI agents using NVIDIA Object-Oriented Agents framework with Python classes, typed methods, and LLM-driven generation
triggers:
  - create an AI agent with NVIDIA OO Agents
  - build a nooa agent with typed methods
  - use NVIDIA Object-Oriented Agents framework
  - implement generation methods in nooa
  - setup nooa agent with tools and state
  - add LLM-driven methods to Python agent
  - trace and debug nooa agent execution
  - create typed agent workflows with NVIDIA OO Agents
---

# NVIDIA Object-Oriented Agents (NOOA)

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

NVIDIA Object-Oriented Agents (NOOA) is a model-agnostic Python framework for building reliable AI agents using standard object-oriented programming. Unlike traditional agent frameworks that separate prompts, tools, and workflows, NOOA unifies these concepts into Python classes where:

- **Agents are Python objects** with typed fields (state) and methods (capabilities)
- **Methods with `...` bodies** become LLM-driven generation methods
- **Regular methods** remain deterministic Python code
- **Type annotations** define contracts with automatic validation
- **Docstrings** serve as prompts
- **The LLM acts by writing Python code** in a REPL with access to `self` and imports

## Installation

### Core Framework

```bash
# Using uv (recommended)
uv add nooa

# Using pip
pip install nooa
```

### Optional Packages

```bash
# CLI tools and trace viewer
uv add nooa-cli
# or as extra: uv add "nooa[cli]"

# Long-term memory subsystem
uv add nooa-memory
# or as extra: uv add "nooa[memory]"

# Benchmarking tools
uv add nooa-bench
# or as extra: uv add "nooa[bench]"

# Multiple extras at once
uv add "nooa[cli,memory,bench]"
```

### From Source

```bash
# Latest development
uv add "nooa @ git+https://github.com/NVIDIA-NeMo/labs-OO-Agents.git@main"

# Pinned release
uv add "nooa @ git+https://github.com/NVIDIA-NeMo/labs-OO-Agents.git@v0.0.7"
```

## Safety Note

**NOOA agents can execute LLM-generated code.** Always run in a sandboxed environment (container, VM, or [NVIDIA OpenShell](https://github.com/NVIDIA/OpenShell)). Built-in AST validation and module deny-lists are defense-in-depth, not containment boundaries.

## Quick Start

### 1. Configure an LLM Client

```python
from nooa.unifiedllm.registry import get_llm_client

# Anthropic (requires ANTHROPIC_API_KEY env var)
llm = get_llm_client("claude-haiku-4-5")

# OpenAI (requires OPENAI_API_KEY env var)
llm = get_llm_client("gpt-5-mini")

# Local Ollama (no key required)
llm = get_llm_client(
    "ollama_chat/qwen3:1.7b",
    api_base="http://localhost:11434"
)

# Local vLLM (no key required)
llm = get_llm_client(
    "hosted_vllm/Qwen/Qwen3-1.7B",
    api_base="http://localhost:8000/v1"
)
```

### 2. Create Your First Agent

```python
import asyncio
from nooa import Agent

class FeedbackAgent(Agent, llm=llm):
    """You are an agent specializing in analyzing customer feedback."""

    async def analyze_feedback(self, text: str) -> str:
        """Analyze customer feedback for sentiment and key topics in one sentence."""
        ...

async def main():
    agent = FeedbackAgent()
    result = await agent.analyze_feedback("Great product, but shipping was slow")
    print(result)

asyncio.run(main())
```

## Core Concepts

### Generation Methods (LLM-Driven)

Methods with `...` bodies are implemented by the LLM at runtime:

```python
class SupportAgent(Agent, llm=llm):
    """You are a customer support agent."""

    async def triage(self, message: str) -> str:
        """Classify this support message and suggest next steps."""
        ...
```

The method name, parameters, return type, and docstring all contribute to the prompt.

### Deterministic Methods (Regular Python)

Regular Python methods provide tools and logic:

```python
from datetime import datetime, timedelta

class OrderAgent(Agent, llm=llm):
    """You help manage customer orders."""

    def is_refund_eligible(self, order_date: datetime) -> bool:
        """Check if order is within 30-day refund window."""
        days_since = (datetime.now() - order_date).days
        return days_since <= 30

    async def process_refund_request(self, order_id: str, order_date: datetime) -> str:
        """Decide whether to approve refund and explain why."""
        ...
```

The LLM can call `self.is_refund_eligible()` when implementing `process_refund_request()`.

### Typed State

Fields on the agent class hold state:

```python
from dataclasses import dataclass
from typing import List

@dataclass
class Order:
    id: str
    total: float
    delivered: bool
    days_since_delivery: int

class SupportAgent(Agent, llm=llm):
    """You are a support agent with access to order database."""

    order_db: dict[str, Order]  # Typed state

    def get_order(self, order_id: str) -> Order | None:
        return self.order_db.get(order_id)

    async def handle_inquiry(self, order_id: str, question: str) -> str:
        """Answer customer question about their order."""
        ...
```

### Structured Output with Pydantic

Use Pydantic models for typed, validated returns:

```python
from pydantic import BaseModel, Field

class Ticket(BaseModel):
    category: str = Field(description="Support category: refund, shipping, technical")
    priority: int = Field(ge=1, le=5, description="Priority from 1 (low) to 5 (critical)")
    summary: str = Field(description="One-sentence summary")

class SupportAgent(Agent, llm=llm):
    """You create support tickets from customer messages."""

    async def create_ticket(self, message: str) -> Ticket:
        """Create a structured support ticket."""
        ...

# Usage
async def main():
    agent = SupportAgent()
    ticket = await agent.create_ticket("My order never arrived and I need it urgently!")
    print(f"Category: {ticket.category}, Priority: {ticket.priority}")
    print(f"Summary: {ticket.summary}")
```

## Advanced Patterns

### Context Blocks for Runtime Information

Use `Context` to provide runtime information without cluttering the agent class:

```python
from nooa import Agent, Context

class ResearchAgent(Agent, llm=llm):
    """You are a research assistant."""

    async def answer_question(self, question: str) -> str:
        """Answer the question using available context."""
        ...

async def main():
    agent = ResearchAgent()

    with Context.user_message("The meeting is scheduled for 3 PM today."):
        result = await agent.answer_question("When is the meeting?")
        print(result)  # Will use the context provided
```

### Multiple Generation Strategies

Agents can use different orchestration strategies:

```python
from nooa import Agent

# ReAct strategy (default): iterative think-act loops
class ReactAgent(Agent, llm=llm, strategy="react"):
    """You solve problems step by step."""

    async def solve(self, problem: str) -> str:
        """Solve this problem."""
        ...

# Code-first strategy: generates Python code to execute
class CodeAgent(Agent, llm=llm, strategy="code"):
    """You solve problems by writing Python code."""

    async def calculate(self, expression: str) -> float:
        """Calculate the result."""
        ...
```

### Progressive Disclosure with doc()

Use `doc()` to provide detailed information only when the LLM requests it:

```python
from nooa import Agent, doc

class AnalyticsAgent(Agent, llm=llm):
    """You analyze sales data."""

    def get_sales_schema(self) -> str:
        return doc("""
        Sales database schema:
        - orders table: id, customer_id, total, date
        - customers table: id, name, email, segment
        - products table: id, name, category, price
        """)

    async def query_sales(self, question: str) -> str:
        """Answer questions about sales data. Use get_sales_schema() if you need schema details."""
        ...
```

The documentation in `doc()` is only shown to the LLM when it calls `get_sales_schema()`.

### Agent Composition

Agents can delegate to other agents:

```python
class EmailAgent(Agent, llm=llm):
    """You write professional emails."""

    async def compose(self, topic: str, recipient: str) -> str:
        """Write an email."""
        ...

class CommunicationAgent(Agent, llm=llm):
    """You manage customer communications."""

    email_agent: EmailAgent

    async def send_update(self, customer_name: str, order_status: str) -> str:
        """Compose and send an order status update."""
        email = await self.email_agent.compose(
            topic=f"Order Status Update: {order_status}",
            recipient=customer_name
        )
        # Send email logic here
        return email

# Usage
async def main():
    agent = CommunicationAgent(email_agent=EmailAgent())
    result = await agent.send_update("John Doe", "Shipped")
```

### Event System

Agents can emit and respond to events:

```python
from nooa import Agent

class MonitorAgent(Agent, llm=llm):
    """You monitor system health."""

    async def on_error(self, error_message: str) -> str:
        """Handle an error event."""
        ...

async def main():
    agent = MonitorAgent()

    # Emit an event
    await agent.emit("error", error_message="Database connection failed")
```

### Long-Term Memory

With `nooa-memory` installed:

```python
from nooa import Agent
from nooa_memory import MemoryManager

class AssistantAgent(Agent, llm=llm):
    """You are a helpful assistant with memory."""

    memory: MemoryManager

    async def chat(self, message: str) -> str:
        """Chat with the user, remembering previous conversations."""
        # Memory is automatically managed
        ...

# Usage
async def main():
    memory = MemoryManager(llm=llm)
    agent = AssistantAgent(memory=memory)

    response1 = await agent.chat("My name is Alice")
    response2 = await agent.chat("What's my name?")  # Will remember "Alice"
```

## CLI Tools

### Start Trace Viewer

```bash
# Start development trace viewer (http://localhost:5001)
uv run nooa start-dev

# Or with custom port
uv run nooa start-dev --port 8080
```

### View Traces

All agent execution is automatically traced (LLM calls, code execution, method invocations) when the viewer is running. Open `http://localhost:5001` to browse traces with parent-child span relationships.

## Configuration

### Environment Variables

```bash
# For Anthropic models
export ANTHROPIC_API_KEY=your_key_here

# For OpenAI models
export OPENAI_API_KEY=your_key_here

# For custom API endpoints (Ollama, vLLM)
# Pass api_base directly to get_llm_client()
```

### LLM Client Options

```python
from nooa.unifiedllm.registry import get_llm_client

llm = get_llm_client(
    "claude-haiku-4-5",
    temperature=0.7,           # Control randomness
    max_tokens=4096,           # Maximum response length
    timeout=60.0,              # Request timeout in seconds
)
```

## Real-World Example: Research Assistant

```python
import asyncio
from typing import List
from pydantic import BaseModel, Field
from nooa import Agent, Context, doc

class Source(BaseModel):
    title: str
    summary: str
    relevance: int = Field(ge=1, le=5, description="Relevance score 1-5")

class ResearchReport(BaseModel):
    topic: str
    key_findings: List[str]
    sources: List[Source]
    conclusion: str

class ResearchAgent(Agent, llm=llm):
    """You are a research assistant who gathers and synthesizes information."""

    search_history: List[str] = []

    def record_search(self, query: str) -> None:
        """Record a search query in history."""
        self.search_history.append(query)
        print(f"Searching: {query}")

    def get_research_guidelines(self) -> str:
        return doc("""
        Research best practices:
        1. Use multiple diverse sources
        2. Verify claims across sources
        3. Note conflicting information
        4. Prioritize recent, authoritative sources
        5. Clearly distinguish facts from opinions
        """)

    async def research_topic(self, topic: str, depth: str = "comprehensive") -> ResearchReport:
        """
        Research a topic and produce a structured report.

        Args:
            topic: The research topic
            depth: "quick" for overview, "comprehensive" for detailed analysis
        """
        ...

async def main():
    agent = ResearchAgent()

    with Context.user_message("Focus on developments from the last 6 months."):
        report = await agent.research_topic(
            topic="Recent advances in multimodal AI models",
            depth="comprehensive"
        )

    print(f"\n=== Research Report: {report.topic} ===")
    print(f"\nKey Findings:")
    for finding in report.key_findings:
        print(f"  - {finding}")

    print(f"\nSources ({len(report.sources)}):")
    for source in report.sources:
        print(f"  - {source.title} (relevance: {source.relevance}/5)")
        print(f"    {source.summary}")

    print(f"\nConclusion:\n{report.conclusion}")
    print(f"\nSearch history: {agent.search_history}")

asyncio.run(main())
```

## Troubleshooting

### Agent Not Calling Methods

**Problem**: Generation method completes without calling helper methods.

**Solution**: Make docstrings more directive:

```python
# Less effective
async def analyze(self, text: str) -> str:
    """Analyze the text."""
    ...

# More effective
async def analyze(self, text: str) -> str:
    """
    Analyze the text for sentiment and topics.
    Use check_language() first to validate the input language.
    """
    ...
```

### Type Validation Errors

**Problem**: Pydantic validation fails on LLM output.

**Solution**: Add field descriptions and constraints:

```python
class Report(BaseModel):
    # Less constrained
    score: int

    # More constrained
    score: int = Field(ge=0, le=100, description="Confidence score from 0 to 100")
```

### Code Execution Failures

**Problem**: Generated code fails or is rejected by validators.

**Solution**:
1. Ensure required imports are available to the agent
2. Check that method signatures match what the LLM expects
3. Review trace viewer to see exactly what code was generated
4. Use `strategy="react"` if code generation is problematic

### Memory Issues

**Problem**: Agent doesn't remember previous interactions.

**Solution**: Install and configure `nooa-memory`:

```python
from nooa_memory import MemoryManager

agent = MyAgent(memory=MemoryManager(llm=llm))
```

### Trace Viewer Not Working

**Problem**: Traces not appearing in viewer.

**Solution**:
1. Ensure `nooa-cli` is installed: `uv add nooa-cli`
2. Start viewer before running agent: `uv run nooa start-dev`
3. Check viewer is running: `curl http://localhost:5001`

### Local Model Issues

**Problem**: Local Ollama/vLLM models not responding.

**Solution**:
```bash
# For Ollama, ensure it's running
ollama serve
ollama pull qwen3:1.7b

# For vLLM, check the endpoint
curl http://localhost:8000/v1/models

# Verify api_base in get_llm_client
llm = get_llm_client(
    "ollama_chat/qwen3:1.7b",
    api_base="http://localhost:11434"  # Must match Ollama port
)
```

## Testing Agents

```python
import pytest
from nooa import Agent

class CalculatorAgent(Agent, llm=llm):
    """You perform calculations."""

    async def add(self, a: float, b: float) -> float:
        """Add two numbers."""
        ...

@pytest.mark.asyncio
async def test_calculator_addition():
    agent = CalculatorAgent()
    result = await agent.add(2.5, 3.5)
    assert abs(result - 6.0) < 0.01  # Allow for floating point precision

@pytest.mark.asyncio
async def test_calculator_with_mock_llm():
    # Use a mock LLM for deterministic testing
    from unittest.mock import AsyncMock

    mock_llm = AsyncMock()
    mock_llm.generate.return_value = "6.0"

    agent = CalculatorAgent(llm=mock_llm)
    result = await agent.add(2.0, 4.0)
    assert result == 6.0
```

## Additional Resources

- **Paper**: [NVIDIA OO Agents: Native Python Object-Oriented Agents](https://arxiv.org/abs/2607.20709)
- **Examples**: [examples/README.md](https://github.com/NVIDIA-NeMo/labs-OO-Agents/blob/main/examples/README.md)
- **Blog**: [Six Agent Harness Capabilities for Higher Model Performance](https://developer.nvidia.com/blog/six-agent-harness-capabilities-for-higher-model-performance/)
- **Repository**: [NVIDIA-NeMo/labs-OO-Agents](https://github.com/NVIDIA-NeMo/labs-OO-Agents)
