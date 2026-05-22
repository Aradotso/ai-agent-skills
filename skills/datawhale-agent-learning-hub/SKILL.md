---
name: datawhale-agent-learning-hub
description: AI Agent learning roadmap and curated resources for building production-ready agents with modern patterns like Claude Code, OpenClaw, skills, MCP, and evaluation
triggers:
  - "how do I learn to build AI agents"
  - "show me an agent learning roadmap"
  - "what's the right way to learn agent development"
  - "help me understand modern agent architecture"
  - "recommend agent tutorials and resources"
  - "how to build coding agents like Claude Code"
  - "what are agent skills and MCP"
  - "agent evaluation and safety best practices"
---

# Datawhale Agent Learning Hub

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A curated AI Agent learning roadmap and resource hub maintained by Datawhale. This project provides a structured learning path from basic agent loops to production-ready agent systems, emphasizing modern patterns like agent harnesses, skills, MCP (Model Context Protocol), and evaluation.

## What This Project Provides

- **Structured Learning Path**: 7-stage todo list from basic agent loops to browser/computer-use agents
- **Curated Resources**: Official docs, papers, and proven open-source projects
- **Modern Focus**: Prioritizes Claude Code, OpenClaw, skills, MCP, A2A over legacy role-play frameworks
- **Project Ladder**: Real-world agent projects you can build at each stage
- **Current Best Practices**: What to learn now vs. what's outdated

## Installation & Access

This is a learning resource repository, not a package to install:

```bash
# Clone the repository
git clone https://github.com/datawhalechina/Agent-Learning-Hub.git
cd Agent-Learning-Hub

# Read the README
cat README.md

# Use it as reference while building agents
```

## Key Learning Stages

### Stage 0: Understand What An Agent Is

**Core Concept**: Distinguish chatbot vs workflow vs agent vs multi-agent.

**Required Reading**:
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [OpenAI: A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)

**Deliverable**: One-page note answering "Why does my use case need an agent instead of a workflow?"

### Stage 1: Build A Minimal Agent Loop

**Core Pattern**: observe → think → act → observe

```python
# Minimal agent loop example (Python + OpenAI)
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))

tools = [
    {
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "Perform basic arithmetic",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {"type": "string", "description": "Math expression like '2+2'"}
                },
                "required": ["expression"]
            }
        }
    }
]

def calculate(expression: str) -> str:
    """Execute safe math expression."""
    try:
        return str(eval(expression, {"__builtins__": {}}, {}))
    except Exception as e:
        return f"Error: {e}"

def run_agent(user_message: str, max_steps: int = 5):
    messages = [{"role": "user", "content": user_message}]
    
    for step in range(max_steps):
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools
        )
        
        message = response.choices[0].message
        messages.append(message)
        
        # Check if done
        if not message.tool_calls:
            return message.content
        
        # Execute tool calls
        for tool_call in message.tool_calls:
            if tool_call.function.name == "calculate":
                import json
                args = json.loads(tool_call.function.arguments)
                result = calculate(args["expression"])
                
                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": result
                })
    
    return "Max steps reached"

# Usage
result = run_agent("What is 25 * 4 + 10?")
print(result)
```

**Deliverable**: 50-150 line agent that can choose tools, execute them, and return final answer.

### Stage 2: Tool Use, RAG, and Memory

**Recommended Projects to Study**:

| Project | Focus Area |
|---------|-----------|
| GPT Researcher | Search → scrape → filter → cite → generate report |
| STORM | Multi-perspective research writing with outline |
| Khoj | Personal second brain with semantic search |
| mem0 | Adding long-term memory to agents |

```python
# RAG-enhanced agent example (using LlamaIndex)
import os
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.core.agent import ReActAgent
from llama_index.core.tools import QueryEngineTool, ToolMetadata

# Load and index documents
documents = SimpleDirectoryReader("./docs").load_data()
index = VectorStoreIndex.from_documents(documents)

# Create query engine tool
query_engine = index.as_query_engine()
query_tool = QueryEngineTool(
    query_engine=query_engine,
    metadata=ToolMetadata(
        name="doc_search",
        description="Search company documentation. Use this when user asks about policies, procedures, or technical specs."
    )
)

# Create agent with tools
agent = ReActAgent.from_tools(
    tools=[query_tool],
    verbose=True
)

# Run agent with citation requirement
response = agent.chat(
    "What is our company's remote work policy? Please cite sources."
)
print(response)
```

**Deliverable**: Research assistant that searches, filters, summarizes, and outputs citations.

### Stage 3: Study One Modern Agent Harness

**Key Systems to Learn**:

| System | Learn This For |
|--------|----------------|
| Claude Code | Real coding agent: CLI, tools, permissions, hooks, subagents, MCP |
| learn-claude-code | From-scratch harness implementation |
| claw0 | Building session, gateway, memory, heartbeat, delivery, resilience |
| OpenClaw | Local-first personal agent with skills and system tools |
| LangGraph | Stateful graph orchestration |

**What to Look For in a Harness**:
- Agent loop implementation
- Tool registry and permission gates
- Session/state store
- Context compaction strategy
- Trace/logging system
- Error handling and recovery

```python
# Example: Understanding tool permission gate pattern
class ToolRegistry:
    def __init__(self):
        self.tools = {}
        self.permissions = {}
    
    def register(self, name: str, func: callable, requires_approval: bool = False):
        """Register tool with optional approval gate."""
        self.tools[name] = func
        self.permissions[name] = {
            "requires_approval": requires_approval,
            "allowed_domains": []  # Could expand to domain restrictions
        }
    
    def execute(self, name: str, args: dict, auto_approve: bool = False):
        """Execute tool with permission check."""
        if name not in self.tools:
            raise ValueError(f"Tool {name} not found")
        
        if self.permissions[name]["requires_approval"] and not auto_approve:
            # In real system, this would trigger user confirmation
            print(f"⚠️  Tool {name} requires approval. Args: {args}")
            confirm = input("Approve? (y/n): ")
            if confirm.lower() != 'y':
                return "Tool execution denied by user"
        
        return self.tools[name](**args)

# Usage
registry = ToolRegistry()
registry.register("search_web", lambda query: f"Results for {query}", requires_approval=False)
registry.register("send_email", lambda to, body: f"Email sent to {to}", requires_approval=True)
```

**Deliverable**: Working agent harness demo with README, example runs, and failure logs.

### Stage 4: Multi-Agent Coordination

**Core Principle**: Multi-agent is coordination, not magic. Use supervisor patterns or graphs, not random chat.

```python
# LangGraph multi-agent example
from langgraph.graph import StateGraph, END
from typing import TypedDict, List

class ResearchState(TypedDict):
    topic: str
    outline: List[str]
    research: dict
    draft: str
    review: str
    final: str

def planner(state: ResearchState) -> ResearchState:
    """Create outline for research."""
    # Call LLM to generate outline
    state["outline"] = ["Introduction", "Key Findings", "Conclusion"]
    return state

def researcher(state: ResearchState) -> ResearchState:
    """Research each section."""
    research = {}
    for section in state["outline"]:
        # Call search API and summarize
        research[section] = f"Research for {section}..."
    state["research"] = research
    return state

def writer(state: ResearchState) -> ResearchState:
    """Write draft from research."""
    state["draft"] = "Draft based on research..."
    return state

def reviewer(state: ResearchState) -> ResearchState:
    """Review and suggest improvements."""
    state["review"] = "Needs more citations in section 2"
    return state

def reviser(state: ResearchState) -> ResearchState:
    """Revise based on review."""
    state["final"] = "Final version with improvements..."
    return state

# Build graph
workflow = StateGraph(ResearchState)
workflow.add_node("planner", planner)
workflow.add_node("researcher", researcher)
workflow.add_node("writer", writer)
workflow.add_node("reviewer", reviewer)
workflow.add_node("reviser", reviser)

workflow.set_entry_point("planner")
workflow.add_edge("planner", "researcher")
workflow.add_edge("researcher", "writer")
workflow.add_edge("writer", "reviewer")
workflow.add_edge("reviewer", "reviser")
workflow.add_edge("reviser", END)

app = workflow.compile()
```

**Deliverable**: Multi-agent system with clear roles (e.g., research → write → review → revise).

### Stage 5: Skills, MCP, and Capability Packaging

**Key Concepts**:
- **Skill**: Reusable procedural knowledge (how to do X)
- **Tool**: Callable interface (function/API)
- **MCP**: Model Context Protocol for connecting external tools/data
- **A2A**: Agent-to-Agent protocol
- **ACP**: Agent Client Protocol

**Skill File Structure** (Claude Code style):

```markdown
# SKILL.md

## Name
code-review

## Description
Perform thorough code review following team standards

## When to Use
- User asks "review this code"
- PR is opened (via webhook)
- Code changes detected in staging branch

## Steps
1. Read code changes (use git diff or file_read tool)
2. Check against style guide in `.code-standards.md`
3. Run linter: `npm run lint` or `python -m pylint`
4. Check for common issues:
   - Hardcoded secrets
   - Missing error handling
   - Unhandled edge cases
   - Performance anti-patterns
5. Generate structured feedback with severity levels

## Tools Required
- file_read
- execute_command
- (optional) github_api for posting comments

## Acceptance Criteria
- All files reviewed
- At least 3 specific suggestions
- Severity level assigned (blocker/major/minor)
- Code style compliance checked
```

**MCP Server Example**:

```typescript
// MCP server for custom tools
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  {
    name: "my-tools-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Register tool
server.setRequestHandler("tools/list", async () => {
  return {
    tools: [
      {
        name: "search_codebase",
        description: "Search company codebase using semantic search",
        inputSchema: {
          type: "object",
          properties: {
            query: { type: "string" },
            language: { type: "string", enum: ["python", "typescript", "all"] }
          },
          required: ["query"]
        }
      }
    ]
  };
});

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "search_codebase") {
    const { query, language } = request.params.arguments;
    // Implement search logic
    return {
      content: [{ type: "text", text: `Results for ${query}...` }]
    };
  }
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

**Deliverable**: Reusable skill (e.g., code-review, research-report, migration-helper) with clear structure.

### Stage 6: Browser and Computer-Use Agents

**Key Patterns**:
- DOM observation and element selection
- Click/type/scroll actions
- Screenshot-based fallback
- Safety boundaries (no sensitive logins, respect robots.txt)

```python
# Browser agent using browser-use
import asyncio
from browser_use import Agent
from langchain_openai import ChatOpenAI

async def main():
    agent = Agent(
        task="Go to news.ycombinator.com and get the top 5 story titles",
        llm=ChatOpenAI(model="gpt-4o"),
    )
    
    result = await agent.run()
    print(result)

asyncio.run(main())
```

**Anthropic Computer Use Example**:

```python
import anthropic
import os

client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    tools=[
        {
            "type": "computer_20241022",
            "name": "computer",
            "display_width_px": 1024,
            "display_height_px": 768,
        }
    ],
    messages=[
        {
            "role": "user",
            "content": "Open a browser and search for 'AI agent frameworks'"
        }
    ]
)

print(response)
```

**Safety Checklist**:
- [ ] No login to sensitive accounts
- [ ] No financial transactions without explicit confirmation
- [ ] Respect robots.txt and rate limits
- [ ] Log all actions with screenshots
- [ ] Human-in-the-loop for risky actions

**Deliverable**: Browser agent that operates on public pages (e.g., extract info, generate summary).

### Stage 7: Evaluation, Observability, and Safety

**Core Metrics**:
- Success rate
- Failure reason distribution
- Tool call count
- Cost per task
- Latency (p50, p95, p99)

```python
# Evaluation harness example
import json
from typing import List, Dict
from dataclasses import dataclass
from datetime import datetime

@dataclass
class TestCase:
    id: str
    input: str
    expected_output: str
    max_steps: int = 10

@dataclass
class EvalResult:
    test_id: str
    success: bool
    actual_output: str
    steps_taken: int
    cost_usd: float
    latency_ms: float
    failure_reason: str = None
    trace: List[Dict] = None

class AgentEvaluator:
    def __init__(self, agent_fn, test_cases: List[TestCase]):
        self.agent_fn = agent_fn
        self.test_cases = test_cases
        self.results: List[EvalResult] = []
    
    def run_evaluation(self) -> Dict:
        """Run all test cases and collect metrics."""
        for test in self.test_cases:
            start_time = datetime.now()
            
            try:
                result = self.agent_fn(test.input, max_steps=test.max_steps)
                success = self._check_success(result, test.expected_output)
                
                latency = (datetime.now() - start_time).total_seconds() * 1000
                
                eval_result = EvalResult(
                    test_id=test.id,
                    success=success,
                    actual_output=result["output"],
                    steps_taken=result["steps"],
                    cost_usd=result["cost"],
                    latency_ms=latency,
                    trace=result.get("trace")
                )
            except Exception as e:
                eval_result = EvalResult(
                    test_id=test.id,
                    success=False,
                    actual_output="",
                    steps_taken=0,
                    cost_usd=0,
                    latency_ms=0,
                    failure_reason=str(e)
                )
            
            self.results.append(eval_result)
        
        return self._compute_metrics()
    
    def _check_success(self, result: Dict, expected: str) -> bool:
        """Check if output matches expected (implement your logic)."""
        return expected.lower() in result["output"].lower()
    
    def _compute_metrics(self) -> Dict:
        """Aggregate metrics across all tests."""
        total = len(self.results)
        successful = sum(1 for r in self.results if r.success)
        
        return {
            "success_rate": successful / total if total > 0 else 0,
            "total_tests": total,
            "total_cost_usd": sum(r.cost_usd for r in self.results),
            "avg_latency_ms": sum(r.latency_ms for r in self.results) / total if total > 0 else 0,
            "failure_reasons": [r.failure_reason for r in self.results if not r.success]
        }
    
    def export_report(self, filename: str):
        """Export detailed report as JSON."""
        report = {
            "metrics": self._compute_metrics(),
            "results": [
                {
                    "test_id": r.test_id,
                    "success": r.success,
                    "steps": r.steps_taken,
                    "cost": r.cost_usd,
                    "latency_ms": r.latency_ms,
                    "failure_reason": r.failure_reason
                }
                for r in self.results
            ]
        }
        
        with open(filename, "w") as f:
            json.dump(report, f, indent=2)

# Usage
test_cases = [
    TestCase(
        id="research-basic",
        input="Research recent AI agent frameworks",
        expected_output="langchain"
    ),
    TestCase(
        id="research-citation",
        input="Find papers on agent evaluation",
        expected_output="citation"
    )
]

evaluator = AgentEvaluator(agent_fn=my_agent_function, test_cases=test_cases)
metrics = evaluator.run_evaluation()
evaluator.export_report("eval_results.json")

print(f"Success rate: {metrics['success_rate']:.2%}")
print(f"Avg cost: ${metrics['total_cost_usd']:.4f}")
```

**Safety Patterns**:

```python
# Dangerous tool approval gate
DANGEROUS_TOOLS = ["delete_file", "send_email", "make_payment", "publish_content"]

def execute_tool_with_approval(tool_name: str, args: dict):
    """Execute tool with human-in-the-loop for dangerous actions."""
    if tool_name in DANGEROUS_TOOLS:
        print(f"\n⚠️  APPROVAL REQUIRED")
        print(f"Tool: {tool_name}")
        print(f"Args: {json.dumps(args, indent=2)}")
        
        approval = input("\nApprove this action? (yes/no): ")
        if approval.lower() != "yes":
            return {"status": "rejected", "reason": "User denied approval"}
    
    # Execute tool
    return tools_registry[tool_name](**args)
```

**Deliverable**: Eval suite with fixed test set, success rate tracking, and cost/latency metrics.

## Common Patterns

### When to Use Agents vs. Workflows

**Use Agent When**:
- Task requires dynamic tool selection
- Steps depend on runtime information
- Need to handle unexpected situations
- Task involves exploration or research

**Use Workflow When**:
- Steps are predictable
- Process is well-defined
- Speed and cost matter more than flexibility
- You need guarantees about execution path

### Context Management

```python
# Simple context compaction strategy
class ContextManager:
    def __init__(self, max_tokens: int = 8000):
        self.max_tokens = max_tokens
        self.messages = []
    
    def add_message(self, message: dict):
        """Add message and compact if needed."""
        self.messages.append(message)
        
        # Estimate tokens (rough: 1 token ≈ 4 chars)
        total_chars = sum(len(str(m)) for m in self.messages)
        estimated_tokens = total_chars // 4
        
        if estimated_tokens > self.max_tokens:
            self._compact()
    
    def _compact(self):
        """Keep system message, last user message, and recent history."""
        system = [m for m in self.messages if m["role"] == "system"]
        recent = self.messages[-10:]  # Keep last 10 messages
        
        # Summarize middle messages (in production, use LLM)
        if len(self.messages) > 12:
            summary = {
                "role": "system",
                "content": f"[Previous conversation summarized: {len(self.messages) - 12} messages]"
            }
            self.messages = system + [summary] + recent
```

## Troubleshooting

### Agent Loops Forever

**Cause**: No max_steps limit or unclear stopping criteria.

**Solution**:
```python
def run_agent_with_limits(task: str, max_steps: int = 10, max_cost: float = 1.0):
    total_cost = 0.0
    
    for step in range(max_steps):
        if total_cost > max_cost:
            return {"error": "Cost limit exceeded", "partial_result": current_state}
        
        # Run agent step
        result = agent.step()
        total_cost += result.cost
        
        if result.is_final:
            return result
    
    return {"error": "Max steps exceeded", "partial_result": current_state}
```

### Tools Return Empty/Error Frequently

**Cause**: Input validation issues or tool design mismatch.

**Solution**:
- Add input schema validation
- Provide clear tool descriptions
- Show examples in tool metadata
- Add retry logic with backoff

```python
from pydantic import BaseModel, Field

class SearchInput(BaseModel):
    query: str = Field(..., min_length=3, description="Search query, at least 3 characters")
    max_results: int = Field(5, ge=1, le=20, description="Number of results, 1-20")

def search_tool(input: SearchInput) -> dict:
    """Type-safe search tool."""
    # Validation happens automatically via Pydantic
    return {"results": [...]}
```

### Agent Hallucinates Citations

**Cause**: No grounding mechanism, agent invents sources.

**Solution**:
- Return citations with every retrieval
- Use structured output for citations
- Post-process to verify citation validity

```python
def verify_citations(text: str, sources: List[str]) -> bool:
    """Check if all citations in text exist in sources."""
    import re
    
    cited = re.findall(r'\[(\d+)\]', text)
    max_source_idx = len(sources) - 1
    
    for citation in cited:
        if int(citation) > max_source_idx:
            return False
    
    return True
```

## Related Resources

**Official Documentation**:
- [Claude Code Docs](https://code.claude.com/docs)
- [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)
- [Anthropic Tool Use](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/overview)
- [Model Context Protocol](https://modelcontextprotocol.io/)

**Key Papers**:
- ReAct: Synergizing Reasoning and Acting in Language Models
- WebArena: A Realistic Web Environment for Building Autonomous Agents
- ToolBench: Tool Learning with Foundation Models

**Recommended Open Source Projects** (curated in repo):
- GPT Researcher, STORM, Khoj (RAG/research)
- learn-claude-code, claw0, OpenClaw (agent harnesses)
- browser-use (browser agents)
- mem0, Letta (memory systems)

## Best Practices

1. **Start Simple**: Build minimal loop before adding frameworks
2. **Add Safety Early**: Approval gates, logging, cost limits from day one
3. **Evaluate Continuously**: Fixed test set, track regressions
4. **Study Harnesses**: Learn from Claude Code, OpenClaw, not just toy examples
5. **Prefer Skills Over Prompts**: Package reusable knowledge formally
6. **Use MCP for Integration**: Standard protocol beats custom tool wrappers
7. **Log Everything**: Traces are essential for debugging agent failures
8. **Human-in-Loop for Risk**: Never auto-approve delete/send/publish

## When to Use This Skill

An AI coding agent should use this skill when:
- User asks "how do I learn AI agents"
- User wants structured agent learning path
- User needs modern agent architecture guidance
- User asks about specific stage (tool use, RAG, multi-agent, evaluation)
- User wants curated agent resources or project recommendations
- User needs code examples for agent loops, tools, or harnesses
