---
name: agent-skills-context-engineering
description: Master context engineering principles for building production-grade AI agent systems with effective context management, multi-agent architectures, and memory systems.
triggers:
  - "build agent system with context engineering"
  - "optimize agent context window usage"
  - "implement multi-agent architecture"
  - "design agent memory system"
  - "compress agent context for long sessions"
  - "debug agent context degradation"
  - "evaluate agent performance with LLM-as-judge"
  - "build hosted coding agent with sandboxes"
---

# Agent Skills for Context Engineering

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A comprehensive collection of Agent Skills for context engineering, multi-agent architectures, and production agent systems. This skill teaches principles for managing LLM context windows, designing effective agent architectures, and building production-grade agent systems.

## What This Project Does

Agent Skills for Context Engineering provides battle-tested patterns for:

- **Context Management**: Managing limited attention budgets, avoiding lost-in-middle degradation
- **Multi-Agent Systems**: Orchestrator, peer-to-peer, and hierarchical architectures
- **Memory Systems**: Short-term, long-term, and graph-based memory patterns
- **Tool Design**: Building tools that agents can use effectively
- **Evaluation**: LLM-as-judge frameworks for measuring agent quality
- **Production Systems**: Hosted agents with sandboxed VMs and multiplayer support

Unlike prompt engineering (crafting instructions), context engineering addresses holistic curation of all information in the context window: system prompts, tool definitions, retrieved documents, message history, and tool outputs.

## Installation

### For Claude Code (Recommended)

**Step 1: Add the Marketplace**

```bash
/plugin marketplace add muratcankoylan/Agent-Skills-for-Context-Engineering
```

**Step 2: Install the Plugin**

```bash
/plugin install context-engineering@context-engineering-marketplace
```

Or browse and install:
1. Select `Browse and install plugins`
2. Select `context-engineering-marketplace`
3. Select `context-engineering`
4. Select `Install now`

### For Cursor (Open Plugins)

Add to your `.cursor/plugins.json`:

```json
{
  "plugins": [
    {
      "name": "context-engineering",
      "repository": "muratcankoylan/Agent-Skills-for-Context-Engineering"
    }
  ]
}
```

### For Individual Skills

Copy specific skills to your project:

```bash
# Create skills directory
mkdir -p .claude/skills

# Add a specific skill (example: context-fundamentals)
curl -o .claude/skills/context-fundamentals.md \
  https://raw.githubusercontent.com/muratcankoylan/Agent-Skills-for-Context-Engineering/main/skills/context-fundamentals/SKILL.md
```

Available skills: `context-fundamentals`, `context-degradation`, `context-compression`, `context-optimization`, `latent-briefing`, `multi-agent-patterns`, `memory-systems`, `tool-design`, `filesystem-context`, `hosted-agents`, `evaluation`, `advanced-evaluation`, `project-development`, `bdi-mental-states`

## Core Concepts

### Context Window Management

The fundamental challenge: context windows are constrained by attention mechanics, not raw token capacity.

**Key degradation patterns:**
- **Lost-in-the-middle**: Models lose track of information in the middle of long contexts
- **U-shaped attention**: Strong attention at beginning and end, weak in middle
- **Attention scarcity**: As context grows, attention per token decreases

**Solution**: Find the smallest possible set of high-signal tokens.

### Progressive Disclosure

Load information only when needed:

```python
# skills/__init__.py - Lazy loading pattern
class SkillRegistry:
    def __init__(self):
        self._skills = {}
        self._loaded = set()
    
    def get_skill_summary(self, skill_name: str) -> dict:
        """Load only name and description initially."""
        return {
            "name": skill_name,
            "description": self._get_description(skill_name)
        }
    
    def load_skill(self, skill_name: str) -> dict:
        """Load full skill content only when activated."""
        if skill_name not in self._loaded:
            self._skills[skill_name] = self._read_skill_file(skill_name)
            self._loaded.add(skill_name)
        return self._skills[skill_name]
```

### Context Compression Strategies

**Sliding Window:**
```python
def sliding_window_context(messages: list, window_size: int = 10) -> list:
    """Keep only recent messages."""
    if len(messages) <= window_size:
        return messages
    
    # Always keep system message
    system_msgs = [m for m in messages if m["role"] == "system"]
    recent_msgs = messages[-window_size:]
    
    return system_msgs + recent_msgs
```

**Summarization:**
```python
async def compress_with_summary(messages: list, llm_client) -> list:
    """Compress old messages into summary."""
    if len(messages) < 20:
        return messages
    
    # Keep recent messages uncompressed
    to_compress = messages[1:-10]  # Skip system message and recent 10
    recent = messages[-10:]
    
    # Generate summary
    summary_prompt = f"Summarize these messages concisely:\n{to_compress}"
    summary = await llm_client.complete(summary_prompt)
    
    return [
        messages[0],  # System message
        {"role": "assistant", "content": f"[Summary of previous conversation: {summary}]"},
        *recent
    ]
```

## Multi-Agent Patterns

### Orchestrator Pattern

Single coordinator delegates to specialized workers:

```python
from typing import List, Dict

class OrchestratorAgent:
    def __init__(self, workers: Dict[str, Agent]):
        self.workers = workers
    
    async def process_task(self, task: str) -> str:
        # Determine which worker to use
        worker_name = await self._route_task(task)
        worker = self.workers[worker_name]
        
        # Delegate to worker with minimal context
        result = await worker.execute(task)
        
        return result
    
    async def _route_task(self, task: str) -> str:
        """Use LLM to determine which worker handles task."""
        routing_prompt = f"""Given this task: {task}

Available workers:
- code_writer: Writes and modifies code
- researcher: Gathers information and analyzes data
- reviewer: Reviews code and provides feedback

Which worker should handle this? Respond with just the worker name."""
        
        return await self.llm.complete(routing_prompt)

# Usage
orchestrator = OrchestratorAgent({
    "code_writer": CodeWriterAgent(),
    "researcher": ResearcherAgent(),
    "reviewer": ReviewerAgent()
})

result = await orchestrator.process_task("Add error handling to the API client")
```

### Peer-to-Peer Pattern

Agents collaborate directly:

```python
class PeerAgent:
    def __init__(self, name: str, peers: List['PeerAgent']):
        self.name = name
        self.peers = peers
        self.messages = []
    
    async def broadcast(self, message: str):
        """Send message to all peers."""
        for peer in self.peers:
            await peer.receive(self.name, message)
    
    async def receive(self, sender: str, message: str):
        """Receive message from peer."""
        self.messages.append({
            "from": sender,
            "content": message,
            "timestamp": time.time()
        })
```

## Memory Systems

### Short-term Memory (Working Context)

```python
class WorkingMemory:
    def __init__(self, max_items: int = 5):
        self.items = []
        self.max_items = max_items
    
    def add(self, item: dict):
        """Add item, removing oldest if at capacity."""
        self.items.append(item)
        if len(self.items) > self.max_items:
            self.items.pop(0)
    
    def get_context(self) -> str:
        """Format for inclusion in prompt."""
        return "\n".join([
            f"- {item['key']}: {item['value']}"
            for item in self.items
        ])
```

### Long-term Memory (Retrieval)

```python
import chromadb
from typing import List, Dict

class LongTermMemory:
    def __init__(self):
        self.client = chromadb.Client()
        self.collection = self.client.create_collection("agent_memory")
    
    def store(self, content: str, metadata: dict = None):
        """Store information for later retrieval."""
        self.collection.add(
            documents=[content],
            metadatas=[metadata or {}],
            ids=[str(hash(content))]
        )
    
    def recall(self, query: str, n_results: int = 3) -> List[Dict]:
        """Retrieve relevant memories."""
        results = self.collection.query(
            query_texts=[query],
            n_results=n_results
        )
        
        return [
            {
                "content": doc,
                "metadata": meta
            }
            for doc, meta in zip(results['documents'][0], results['metadatas'][0])
        ]
```

### Graph-based Memory

```python
import networkx as nx

class GraphMemory:
    def __init__(self):
        self.graph = nx.DiGraph()
    
    def add_entity(self, entity: str, properties: dict):
        """Add or update entity node."""
        self.graph.add_node(entity, **properties)
    
    def add_relation(self, from_entity: str, to_entity: str, relation: str):
        """Add relationship between entities."""
        self.graph.add_edge(from_entity, to_entity, relation=relation)
    
    def get_neighbors(self, entity: str, max_depth: int = 2) -> dict:
        """Get connected entities within depth."""
        if entity not in self.graph:
            return {}
        
        # BFS to find neighbors
        neighbors = {}
        for node in nx.single_source_shortest_path_length(
            self.graph, entity, cutoff=max_depth
        ):
            neighbors[node] = self.graph.nodes[node]
        
        return neighbors
```

## Tool Design Principles

### Minimal Interface

```python
from typing import Dict, Any

def search_documentation(query: str, max_results: int = 5) -> list[Dict[str, Any]]:
    """Search documentation with minimal parameters.
    
    Args:
        query: Search query string
        max_results: Maximum number of results to return (default: 5)
    
    Returns:
        List of matching documentation sections with title and content
    
    Example:
        results = search_documentation("authentication")
    """
    # Implementation
    pass
```

### Clear Output Format

```python
def analyze_code(code: str) -> dict:
    """Analyze code and return structured results.
    
    Returns:
        {
            "issues": [{"line": int, "severity": str, "message": str}],
            "metrics": {"complexity": int, "lines": int},
            "suggestions": [str]
        }
    """
    return {
        "issues": [
            {"line": 15, "severity": "warning", "message": "Unused variable 'x'"}
        ],
        "metrics": {
            "complexity": 7,
            "lines": 42
        },
        "suggestions": [
            "Consider extracting this logic into a separate function"
        ]
    }
```

### Context Offloading

```python
import json
from pathlib import Path

def analyze_large_dataset(data_path: str, output_dir: str = ".agent_context") -> str:
    """Analyze data and write detailed results to file.
    
    Returns reference to results file instead of full data in context.
    """
    # Create context directory
    Path(output_dir).mkdir(exist_ok=True)
    
    # Analyze data
    results = perform_analysis(data_path)
    
    # Write detailed results to file
    results_file = f"{output_dir}/analysis_results.json"
    with open(results_file, 'w') as f:
        json.dump(results, f, indent=2)
    
    # Return only summary in context
    summary = {
        "total_records": results["count"],
        "key_findings": results["top_insights"][:3],
        "full_results": results_file
    }
    
    return f"Analysis complete. Summary: {summary}\nFull results in {results_file}"
```

## Filesystem-based Context Management

### Dynamic Discovery

```python
from pathlib import Path
import yaml

def discover_tools(tools_dir: str = ".agent_tools") -> dict:
    """Dynamically discover available tools from filesystem."""
    tools = {}
    
    for tool_file in Path(tools_dir).glob("*.yaml"):
        with open(tool_file) as f:
            tool_spec = yaml.safe_load(f)
            tools[tool_spec["name"]] = tool_spec
    
    return tools

# Tool definition file: .agent_tools/github_search.yaml
"""
name: github_search
description: Search GitHub repositories
parameters:
  - name: query
    type: string
    required: true
  - name: language
    type: string
    required: false
"""
```

### Plan Persistence

```python
import json
from datetime import datetime

class PlanTracker:
    def __init__(self, plan_file: str = ".agent_context/current_plan.json"):
        self.plan_file = plan_file
    
    def save_plan(self, steps: list):
        """Persist plan to disk."""
        plan = {
            "created_at": datetime.now().isoformat(),
            "steps": steps,
            "completed": []
        }
        
        with open(self.plan_file, 'w') as f:
            json.dump(plan, f, indent=2)
    
    def mark_complete(self, step_index: int):
        """Mark step as complete."""
        with open(self.plan_file, 'r') as f:
            plan = json.load(f)
        
        plan["completed"].append(step_index)
        
        with open(self.plan_file, 'w') as f:
            json.dump(plan, f, indent=2)
    
    def get_next_step(self) -> dict:
        """Get next incomplete step."""
        with open(self.plan_file, 'r') as f:
            plan = json.load(f)
        
        for i, step in enumerate(plan["steps"]):
            if i not in plan["completed"]:
                return {"index": i, "step": step}
        
        return None
```

## LLM-as-Judge Evaluation

### Direct Scoring

```python
from typing import List, Dict

async def score_response(
    response: str,
    criteria: List[Dict[str, any]],
    llm_client
) -> dict:
    """Score response against weighted criteria.
    
    Args:
        response: Response to evaluate
        criteria: List of {name, description, weight, scale}
        llm_client: LLM client for evaluation
    
    Returns:
        {
            "total_score": float,
            "criteria_scores": [{name, score, reasoning}]
        }
    """
    scores = []
    
    for criterion in criteria:
        prompt = f"""Evaluate this response on {criterion['name']}:

{criterion['description']}

Response to evaluate:
{response}

Score from 1-{criterion['scale']} and explain your reasoning.
Format: SCORE: X | REASONING: explanation"""
        
        evaluation = await llm_client.complete(prompt)
        
        # Parse score and reasoning
        score_line = [l for l in evaluation.split('\n') if 'SCORE:' in l][0]
        score = int(score_line.split('SCORE:')[1].split('|')[0].strip())
        reasoning = evaluation.split('REASONING:')[1].strip()
        
        scores.append({
            "name": criterion['name'],
            "score": score,
            "max_score": criterion['scale'],
            "weight": criterion['weight'],
            "reasoning": reasoning
        })
    
    # Calculate weighted total
    total = sum(s['score'] / s['max_score'] * s['weight'] for s in scores)
    total_weight = sum(c['weight'] for c in criteria)
    
    return {
        "total_score": total / total_weight,
        "criteria_scores": scores
    }

# Usage
result = await score_response(
    response="The API returns JSON with user data...",
    criteria=[
        {
            "name": "accuracy",
            "description": "Is the information factually correct?",
            "weight": 2.0,
            "scale": 5
        },
        {
            "name": "completeness",
            "description": "Does it cover all aspects of the question?",
            "weight": 1.5,
            "scale": 5
        }
    ],
    llm_client=client
)
```

### Pairwise Comparison

```python
async def compare_responses(
    response_a: str,
    response_b: str,
    criteria: str,
    llm_client,
    mitigate_bias: bool = True
) -> dict:
    """Compare two responses with position bias mitigation.
    
    Args:
        response_a: First response
        response_b: Second response
        criteria: Evaluation criteria
        llm_client: LLM client
        mitigate_bias: Run comparison both ways and aggregate
    
    Returns:
        {"winner": "A" | "B" | "tie", "reasoning": str, "confidence": float}
    """
    async def single_comparison(first: str, second: str) -> dict:
        prompt = f"""Compare these two responses based on: {criteria}

Response 1:
{first}

Response 2:
{second}

Which is better? Respond with: WINNER: [1|2|TIE] | REASONING: explanation"""
        
        result = await llm_client.complete(prompt)
        winner_line = [l for l in result.split('\n') if 'WINNER:' in l][0]
        winner = winner_line.split('WINNER:')[1].split('|')[0].strip()
        reasoning = result.split('REASONING:')[1].strip()
        
        return {"winner": winner, "reasoning": reasoning}
    
    # First comparison (A then B)
    comp1 = await single_comparison(response_a, response_b)
    
    if not mitigate_bias:
        return {
            "winner": "A" if comp1["winner"] == "1" else "B" if comp1["winner"] == "2" else "tie",
            "reasoning": comp1["reasoning"],
            "confidence": 1.0
        }
    
    # Second comparison (B then A) to mitigate position bias
    comp2 = await single_comparison(response_b, response_a)
    
    # Aggregate results
    if comp1["winner"] == "1" and comp2["winner"] == "2":
        return {"winner": "A", "reasoning": comp1["reasoning"], "confidence": 1.0}
    elif comp1["winner"] == "2" and comp2["winner"] == "1":
        return {"winner": "B", "reasoning": comp1["reasoning"], "confidence": 1.0}
    elif comp1["winner"] == "TIE" or comp2["winner"] == "TIE":
        return {"winner": "tie", "reasoning": "Evaluations were mixed", "confidence": 0.5}
    else:
        return {"winner": "tie", "reasoning": "Evaluations disagreed", "confidence": 0.3}
```

## Hosted Agents (Sandboxed Execution)

### Modal-based Background Agent

```python
import modal

# Create Modal app
app = modal.App("coding-agent")

# Define image with dependencies
image = modal.Image.debian_slim().pip_install(
    "anthropic",
    "requests"
)

@app.function(
    image=image,
    secrets=[modal.Secret.from_name("anthropic-api-key")],
    timeout=3600
)
async def run_coding_task(task: str, files: dict) -> dict:
    """Execute coding task in sandboxed environment.
    
    Args:
        task: Task description
        files: Initial files as {path: content}
    
    Returns:
        {
            "status": "success" | "error",
            "files": {path: content},
            "output": str
        }
    """
    import os
    from anthropic import Anthropic
    
    # Initialize Claude
    client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
    
    # Write initial files
    for path, content in files.items():
        os.makedirs(os.path.dirname(path), exist_ok=True)
        with open(path, 'w') as f:
            f.write(content)
    
    # Execute task with agent
    messages = [
        {
            "role": "user",
            "content": f"Complete this task: {task}\n\nAvailable files: {list(files.keys())}"
        }
    ]
    
    response = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=4096,
        messages=messages
    )
    
    # Collect modified files
    output_files = {}
    for path in files.keys():
        if os.path.exists(path):
            with open(path, 'r') as f:
                output_files[path] = f.read()
    
    return {
        "status": "success",
        "files": output_files,
        "output": response.content[0].text
    }

# Client usage
@app.local_entrypoint()
def main():
    result = run_coding_task.remote(
        task="Add error handling to the API client",
        files={
            "client.py": "def fetch_data():\n    response = requests.get(url)\n    return response.json()"
        }
    )
    
    print(result["output"])
    print("\nModified files:")
    for path, content in result["files"].items():
        print(f"\n{path}:")
        print(content)
```

## Common Patterns

### Task-Model Fit Analysis

```python
def analyze_task_model_fit(task_description: str) -> dict:
    """Determine if LLM is appropriate for task.
    
    Returns:
        {
            "recommended": bool,
            "model_size": "small" | "medium" | "large",
            "approach": "single_shot" | "chain_of_thought" | "multi_agent",
            "reasoning": str
        }
    """
    # Decision tree
    is_structured = "json" in task_description.lower() or "schema" in task_description.lower()
    is_creative = any(w in task_description.lower() for w in ["generate", "write", "create"])
    is_complex = any(w in task_description.lower() for w in ["analyze", "reason", "solve"])
    
    if is_structured:
        return {
            "recommended": True,
            "model_size": "small",
            "approach": "single_shot",
            "reasoning": "Structured output tasks work well with smaller models"
        }
    elif is_complex:
        return {
            "recommended": True,
            "model_size": "large",
            "approach": "chain_of_thought",
            "reasoning": "Complex reasoning requires larger models with step-by-step thinking"
        }
    elif is_creative:
        return {
            "recommended": True,
            "model_size": "medium",
            "approach": "single_shot",
            "reasoning": "Creative tasks benefit from medium-sized models"
        }
    else:
        return {
            "recommended": False,
            "model_size": None,
            "approach": None,
            "reasoning": "Task may be better suited for traditional programming"
        }
```

### Append-Only Memory Pattern

```python
import json
from datetime import datetime
from pathlib import Path

class AppendOnlyMemory:
    """JSONL-based memory with schema-first line for agent parsing."""
    
    def __init__(self, file_path: str, schema: dict):
        self.file_path = Path(file_path)
        self.schema = schema
        
        # Create file with schema if doesn't exist
        if not self.file_path.exists():
            self.file_path.parent.mkdir(parents=True, exist_ok=True)
            with open(self.file_path, 'w') as f:
                f.write(json.dumps({"_schema": schema}) + '\n')
    
    def append(self, data: dict):
        """Append entry with timestamp."""
        entry = {
            **data,
            "_timestamp": datetime.now().isoformat()
        }
        
        with open(self.file_path, 'a') as f:
            f.write(json.dumps(entry) + '\n')
    
    def read_recent(self, n: int = 10) -> list:
        """Read n most recent entries."""
        with open(self.file_path, 'r') as f:
            lines = f.readlines()
        
        # Skip schema line, get recent entries
        entries = [json.loads(line) for line in lines[1:]]
        return entries[-n:]
    
    def query(self, filter_fn) -> list:
        """Query entries with filter function."""
        with open(self.file_path, 'r') as f:
            lines = f.readlines()
        
        entries = [json.loads(line) for line in lines[1:]]
        return [e for e in entries if filter_fn(e)]

# Usage
memory = AppendOnlyMemory(
    ".agent_memory/interactions.jsonl",
    schema={
        "user": "string",
        "action": "string",
        "result": "string"
    }
)

memory.append({
    "user": "developer_123",
    "action": "deployed_feature",
    "result": "success"
})

recent = memory.read_recent(5)
```

## Troubleshooting

### Lost-in-Middle Degradation

**Symptom**: Agent ignores information from middle of context.

**Solution**: Use U-shaped context organization:

```python
def organize_context(system_msg: str, key_facts: list, messages: list) -> list:
    """Place critical info at beginning and end."""
    return [
        {"role": "system", "content": system_msg},
        # Key facts at the top
        {"role": "system", "content": "Key facts:\n" + "\n".join(key_facts)},
        # Conversation in middle
        *messages[:-3],
        # Recent context at the end
        *messages[-3:],
        # Reminder of key facts at very end
        {"role": "system", "content": "Remember:\n" + "\n".join(key_facts[:3])}
    ]
```

### Context Clash

**Symptom**: Agent confused by contradictory information.

**Solution**: Timestamp and prioritize information:

```python
def merge_contexts(contexts: list[dict]) -> str:
    """Merge contexts with priority and timestamps."""
    # Sort by priority (higher first) then timestamp (newer first)
    sorted_contexts = sorted(
        contexts,
        key=lambda x: (-x.get('priority', 0), -x.get('timestamp', 0))
    )
    
    merged = []
    for ctx in sorted_contexts:
        timestamp_str = datetime.fromtimestamp(ctx['timestamp']).strftime('%Y-%m-%d %H:%M')
        merged.append(f"[{timestamp_str}, priority={ctx.get('priority', 0)}] {ctx['content']}")
    
    return "\n\n".join(merged)
```

### Attention Scarcity

**Symptom**: Performance degrades with long contexts.

**Solution**: Implement KV-cache compaction:

```python
def compact_messages(messages: list, target_tokens: int) -> list:
    """Compress messages to target token budget."""
    current_tokens = sum(estimate_tokens(m["content"]) for m in messages)
    
    if current_tokens <= target_tokens:
        return messages
    
    # Keep system message and recent messages
    system = [m for m in messages if m["role"] == "system"]
    recent = messages[-5:]
    
    # Compress middle messages
    middle = messages[len(system):-5]
    compression_ratio = (target_tokens - estimate_tokens(system + recent)) / estimate_tokens(middle)
    
    if compression_ratio < 0.5:
        # Aggressive summarization needed
        summary = summarize_messages(middle)
        return system + [{"role": "assistant", "content": f"[Earlier: {summary}]"}] + recent
    else:
        # Keep important messages, drop others
        important = extract_important_messages(middle, keep_ratio=compression_ratio)
        return system + important + recent

def estimate_tokens(text: str) -> int:
    """Rough estimate: 1 token per 4 characters."""
    return len(text) // 4
```

### Tool Output Bloat

**Symptom**: Tool outputs fill context window.

**Solution**: Offload to filesystem:

```python
def execute_tool_with_offloading(
    tool_name: str,
    params: dict,
    context_dir: str = ".agent_context/tool_outputs"
) -> str:
    """Execute tool and offload detailed results."""
    Path(context_dir).mkdir(parents=True, exist_ok=True)
    
    # Execute tool
    result = execute_tool(tool_name, params)
    
    # Write full result to file
    output_file = f"{context_dir}/{tool_name}_{int(time.time())}.json"
    with open(output_file, 'w') as f:
        json.dump(result, f, indent=2)
    
    # Return summary + file reference
    summary = {
        "status": result.get("status"),
        "key_findings": result.get("summary", "See file for details"),
        "record_count": len(result.get("data", [])),
        "full_output": output_file
    }
    
    return json.dumps(summary)
```

## Best Practices

1. **Progressive Disclosure**: Load information only when needed
2. **Context Budget**: Track token usage, compress proactively
3. **Tool Minimalism**: Design tools with minimal interface, maximum clarity
4. **Filesystem Offloading**: Keep detailed data out of context
5. **Evaluation First**: Build evaluation before building features
6. **Platform Agnostic**: Focus on transferable principles over vendor APIs
7. **Append-Only Memory**: Use JSONL with schema-first for agent-friendly persistence
8. **U-shaped Organization**: Place critical info at start and end of context

## Resources

- [Repository](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering)
- [DeepWiki Documentation](https://deepwiki.com/muratcankoylan/Agent-Skills-for-Context-Engineering)
- [Cursor Plugin Directory](https://cursor.directory/plugins/context-engineering)
- [Examples Directory](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering/tree/main/examples)

## License

MIT
