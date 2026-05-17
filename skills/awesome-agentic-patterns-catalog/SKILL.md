---
name: awesome-agentic-patterns-catalog
description: Expert knowledge of agentic AI design patterns for autonomous agent development
triggers:
  - "show me agentic patterns for memory management"
  - "what patterns exist for agent orchestration"
  - "help me design an autonomous agent workflow"
  - "which pattern should I use for tool routing"
  - "compare reflection loop and tree of thought patterns"
  - "what are best practices for agent feedback loops"
  - "show examples of multi-agent coordination patterns"
  - "how do I implement context window management"
---

# Awesome Agentic Patterns Catalog

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill provides comprehensive knowledge of **agentic AI patterns** — production-ready architectural patterns, workflows, and techniques for building autonomous and semi-autonomous AI agents. The Awesome Agentic Patterns catalog curates real-world patterns with traceability to implementations, papers, and production use cases.

## What is Awesome Agentic Patterns?

A curated catalogue of repeatable patterns that help AI agents sense, reason, and act effectively in production environments. Each pattern is:

- **Repeatable** — proven by multiple teams
- **Agent-centric** — improves agent capabilities
- **Traceable** — backed by public references (blogs, papers, repos)

**Website**: https://agentic-patterns.com  
**Repository**: https://github.com/nibzard/awesome-agentic-patterns

## Pattern Categories

The catalog organizes patterns into 8 core categories:

1. **Context & Memory** — Managing agent state, memory, and context windows
2. **Feedback Loops** — Self-improvement, reflection, and iterative refinement
3. **Learning & Adaptation** — Skill evolution and reinforcement learning
4. **Orchestration & Control** — Task decomposition, multi-agent coordination
5. **Reliability & Eval** — Testing, monitoring, and fault tolerance
6. **Security & Safety** — Sandboxing, PII protection, guardrails
7. **Tool Use & Environment** — Interacting with shells, browsers, databases
8. **UX & Collaboration** — Human-agent interaction patterns

## Installation & Access

### Browse Online

The primary way to explore patterns is via the website:

```bash
# Visit the interactive pattern explorer
open https://agentic-patterns.com
```

Features available on the website:
- **Pattern Explorer**: Filter by category, complexity, status
- **Compare Tool**: Side-by-side pattern comparison
- **Decision Explorer**: Interactive pattern selection guide
- **Graph Visualization**: Pattern relationship mapping
- **Pattern Packs**: Curated collections for common architectures

### Clone Repository

```bash
git clone https://github.com/nibzard/awesome-agentic-patterns.git
cd awesome-agentic-patterns
```

### Repository Structure

```
awesome-agentic-patterns/
├── patterns/           # Individual pattern markdown files
│   ├── context-window-auto-compaction.md
│   ├── reflection.md
│   ├── plan-then-execute-pattern.md
│   └── ...
├── apps/
│   └── web/           # Astro-based website source
├── README.md          # Main catalog listing
└── LICENSE
```

## Key Patterns Overview

### Context & Memory Patterns

**Curated Code Context Window**
- Dynamically select relevant code files for LLM context
- Use semantic search or dependency analysis
- Example: Pass only modified files + their direct dependencies

**Prompt Caching via Exact Prefix Preservation**
- Structure prompts so static context (system, docs) comes first
- Cache LLM processing of unchanged prefix
- Reduces latency and cost for iterative workflows

**Episodic Memory Retrieval & Injection**
- Store past interactions in vector DB
- Retrieve relevant episodes based on current task
- Inject as context to maintain coherence across sessions

**Working Memory via TodoWrite**
- Agents maintain explicit TODO lists in files
- Track progress, next steps, and blockers
- Provides persistence across crashes/restarts

### Feedback Loop Patterns

**Reflection Loop**
- Agent reviews its own output before finalizing
- Self-critique → revise → validate cycle
- Example workflow:
  ```
  1. Generate initial solution
  2. Critique: "Does this handle edge case X?"
  3. Revise based on critique
  4. Validate against requirements
  ```

**Coding Agent CI Feedback Loop**
- Agent commits code → CI runs → agent reads failures → agent fixes
- Automated self-healing for test failures
- Example integration:
  ```python
  while not tests_pass:
      run_tests()
      if failures:
          agent.analyze_failures(test_output)
          agent.generate_fix()
          commit_and_retry()
  ```

**Self-Critique Evaluator Loop**
- Separate evaluator agent assesses worker agent output
- Worker iterates based on evaluator feedback
- Prevents over-fitting to single perspective

### Orchestration & Control Patterns

**Plan-Then-Execute Pattern**
- Decompose complex task into explicit plan
- Execute steps sequentially with validation
- Update plan based on execution results

```python
# Conceptual implementation
def plan_then_execute(task):
    plan = planner_llm.generate_plan(task)
    results = []
    
    for step in plan.steps:
        result = executor.execute(step)
        if result.needs_replanning:
            plan = planner_llm.replan(task, results, step)
        results.append(result)
    
    return synthesize(results)
```

**Sub-Agent Spawning**
- Main agent delegates subtasks to specialized sub-agents
- Each sub-agent has focused tools and context
- Results aggregated by parent agent

**Dual LLM Pattern**
- Use different models for different tasks
- Fast/cheap model for simple decisions
- Powerful model for complex reasoning
- Example: GPT-4o-mini for routing, GPT-4 for generation

### Tool Use Patterns

**Tool Selection Guide**
- Provide LLM with explicit decision tree for tool selection
- Include when to use each tool, expected inputs/outputs
- Reduces tool misuse and hallucination

```markdown
Tool Selection Guide:
- search_code(query): When user asks "where is X defined"
- run_tests(path): After code changes, before commit
- read_file(path): When context about specific file needed
- edit_file(path, instructions): To modify existing code
```

**Conditional Parallel Tool Execution**
- Execute independent tools concurrently
- Reduce latency for multi-step operations
- Example: Search docs + search code + check tests in parallel

### Reliability Patterns

**Agent Circuit Breaker**
- Track failure rate of agent actions
- Open circuit (disable agent) after threshold
- Fallback to human or simpler system

```python
class AgentCircuitBreaker:
    def __init__(self, failure_threshold=5):
        self.failures = 0
        self.threshold = failure_threshold
        self.state = "closed"  # closed, open, half-open
    
    def call(self, agent_fn, *args):
        if self.state == "open":
            raise CircuitOpenError("Too many failures")
        
        try:
            result = agent_fn(*args)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
    
    def on_failure(self):
        self.failures += 1
        if self.failures >= self.threshold:
            self.state = "open"
```

**LLM Observability**
- Log all prompts, completions, tool calls
- Track latency, token usage, costs
- Essential for debugging and optimization

## Using Patterns in Agent Development

### Pattern Selection Process

1. **Identify your agent's core challenge**
   - Memory/context limits → Context & Memory patterns
   - Quality issues → Feedback Loop patterns
   - Complex multi-step tasks → Orchestration patterns
   - Reliability concerns → Reliability & Eval patterns

2. **Review pattern prerequisites**
   - Check if pattern requires specific infrastructure
   - Assess complexity vs. benefit trade-off

3. **Start simple, iterate**
   - Implement minimal version first
   - Add sophistication based on real failures

### Example: Building a Code Review Agent

```python
# Combining multiple patterns

from typing import List, Dict

class CodeReviewAgent:
    """
    Combines:
    - Curated Code Context Window (Context & Memory)
    - Reflection Loop (Feedback)
    - Tool Selection Guide (Tool Use)
    """
    
    def __init__(self, llm, code_retriever):
        self.llm = llm
        self.retriever = code_retriever
        self.tool_guide = self._load_tool_guide()
    
    def review_pr(self, pr_diff: str) -> Dict:
        # 1. Curated Context: Fetch relevant files
        context_files = self.retriever.get_relevant_context(
            pr_diff, 
            max_files=10
        )
        
        # 2. Initial review with tool guide
        initial_review = self.llm.generate(
            prompt=f"""
            {self.tool_guide}
            
            Review this PR diff:
            {pr_diff}
            
            Context files:
            {context_files}
            
            Use available tools to verify claims.
            """,
            tools=["run_tests", "search_similar_code", "check_style"]
        )
        
        # 3. Reflection loop: Self-critique
        critique = self.llm.generate(
            prompt=f"""
            Review this code review for:
            - Are all concerns valid?
            - Any false positives?
            - Missing critical issues?
            
            Review: {initial_review}
            """
        )
        
        # 4. Final review incorporating critique
        final_review = self.llm.generate(
            prompt=f"""
            Original review: {initial_review}
            Self-critique: {critique}
            
            Produce final review addressing critique points.
            """
        )
        
        return {
            "review": final_review,
            "context_used": context_files,
            "critique": critique
        }
    
    def _load_tool_guide(self) -> str:
        return """
        Tool Selection for Code Review:
        - run_tests(test_path): If PR touches test files or claims fix
        - search_similar_code(pattern): To find similar patterns/bugs
        - check_style(file_path): For style/lint violations
        - get_git_history(file): For understanding change context
        """
```

### Example: Multi-Agent Research Assistant

```python
# Implementing Planner-Worker Separation + Sub-Agent Spawning

class ResearchOrchestrator:
    """
    Patterns:
    - Planner-Worker Separation
    - Sub-Agent Spawning
    - Plan-Then-Execute
    """
    
    def __init__(self, planner_llm, worker_llm):
        self.planner = planner_llm
        self.worker = worker_llm
    
    async def research(self, query: str) -> Dict:
        # 1. Planner creates research plan
        plan = self.planner.generate(
            prompt=f"""
            Create research plan for: {query}
            
            Output as JSON with steps:
            [
              {{"type": "search", "query": "...", "sources": [...]}},
              {{"type": "analyze", "focus": "..."}},
              {{"type": "synthesize", "format": "..."}}
            ]
            """
        )
        
        # 2. Spawn sub-agents for parallel search
        search_tasks = [
            step for step in plan if step["type"] == "search"
        ]
        
        search_results = await asyncio.gather(*[
            self._spawn_search_agent(task)
            for task in search_tasks
        ])
        
        # 3. Worker agent analyzes results
        analysis = self.worker.generate(
            prompt=f"""
            Analyze these search results for: {query}
            
            Results: {search_results}
            
            Focus: {[s['focus'] for s in plan if s['type'] == 'analyze']}
            """
        )
        
        # 4. Synthesize final report
        report = self.worker.generate(
            prompt=f"""
            Synthesize research report:
            Query: {query}
            Analysis: {analysis}
            Format: {plan[-1]['format']}
            """
        )
        
        return {
            "report": report,
            "sources": search_results,
            "plan_used": plan
        }
    
    async def _spawn_search_agent(self, task: Dict):
        """Dedicated sub-agent for single search task"""
        agent = SearchAgent(self.worker, sources=task["sources"])
        return await agent.search(task["query"])
```

## Common Pattern Combinations

### Autonomous Coding Agent Stack

```
1. Curated Code Context Window (manage context size)
2. Plan-Then-Execute (break down complex changes)
3. Coding Agent CI Feedback Loop (validate changes)
4. Reflection Loop (self-review before commit)
5. Agent Circuit Breaker (prevent infinite loops)
```

### Long-Running Agent Architecture

```
1. Filesystem-Based Agent State (persist state)
2. Working Memory via TodoWrite (track progress)
3. Planner-Worker Separation (long-term planning)
4. Signal-Driven Agent Activation (efficient wake-up)
5. LLM Observability (monitor over time)
```

### Multi-Agent System

```
1. Declarative Multi-Agent Topology (define structure)
2. Economic Value Signaling (coordinate via incentives)
3. Sub-Agent Spawning (dynamic creation)
4. Opponent Processor (debate for quality)
5. Cross-Cycle Consensus Relay (agreement protocol)
```

## Troubleshooting & Best Practices

### Context Window Issues

**Problem**: Agent loses track of earlier conversation

**Solutions**:
- Apply **Context Window Auto-Compaction**: Summarize old context
- Use **Episodic Memory Retrieval**: Store + retrieve relevant history
- Implement **Progressive Disclosure**: Only show needed details
- Try **Prompt Caching**: Preserve expensive prefix computation

### Reliability Issues

**Problem**: Agent produces inconsistent results

**Solutions**:
- Add **Reflection Loop**: Self-review catches errors
- Implement **Agent Circuit Breaker**: Prevent cascading failures
- Use **LLM Observability**: Log everything to debug
- Apply **Failover-Aware Model Fallback**: Backup models

### Tool Misuse

**Problem**: Agent uses wrong tools or hallucinates tool calls

**Solutions**:
- Provide **Tool Selection Guide**: Explicit decision rules
- Use **Tool Capability Compartmentalization**: Limit tool access per task
- Implement **Tool Use Incentivization**: Reward correct usage
- Apply **Conditional Parallel Tool Execution**: Validate before executing

### Planning Failures

**Problem**: Agent creates poor plans or gets stuck

**Solutions**:
- Apply **Plan-Then-Execute Pattern**: Separate planning from execution
- Use **Tree-of-Thought Reasoning**: Explore multiple plans
- Implement **Explicit Posterior-Sampling Planner**: Probabilistic planning
- Try **Language Agent Tree Search (LATS)**: Search plan space

## Contributing Patterns

To add a new pattern to the catalog:

1. **Validate it's repeatable**: Used by 2+ teams/projects
2. **Document traceability**: Link to blog, paper, or repo
3. **Create pattern file**: `patterns/your-pattern-name.md`
4. **Follow template structure**:
   ```markdown
   # Pattern Name
   
   ## Problem
   What challenge does this solve?
   
   ## Solution
   How does the pattern work?
   
   ## Implementation
   Code examples, architecture diagrams
   
   ## References
   - [Source 1](url)
   - [Source 2](url)
   ```
5. **Submit PR**: https://github.com/nibzard/awesome-agentic-patterns

## Resources

- **Website**: https://agentic-patterns.com
- **GitHub**: https://github.com/nibzard/awesome-agentic-patterns
- **License**: Apache-2.0
- **Pattern Explorer**: Filter 50+ patterns by category, complexity
- **Decision Tool**: Interactive guide for pattern selection
- **Graph View**: Visual pattern relationships

## Advanced Usage

### Pattern Selection Matrix

| Your Challenge | Primary Pattern | Supporting Patterns |
|---------------|----------------|-------------------|
| Exceeding context limits | Curated Code Context Window | Prompt Caching, Progressive Disclosure |
| Low quality outputs | Reflection Loop | Self-Critique Evaluator, CriticGPT |
| Complex multi-step tasks | Plan-Then-Execute | Sub-Agent Spawning, Tree-of-Thought |
| Unreliable behavior | Agent Circuit Breaker | LLM Observability, Failover Fallback |
| Tool confusion | Tool Selection Guide | Tool Compartmentalization |
| Multi-agent coordination | Declarative Topology | Economic Value Signaling |

### Pattern Anti-Patterns

**Don't**:
- Stack too many patterns initially (start simple)
- Use orchestration patterns for simple tasks (overhead)
- Skip observability (you'll regret it)
- Ignore context window limits (use memory patterns)
- Over-engineer before proving need (iterate)

**Do**:
- Measure before optimizing (observability first)
- Start with single-agent patterns (orchestrate later)
- Document why you chose each pattern (maintainability)
- Test patterns in isolation (debug complexity)
- Review catalog regularly (patterns evolve)

This skill provides comprehensive knowledge of production-ready agentic patterns. Use the website's interactive tools for pattern discovery and the repository examples for implementation guidance.
