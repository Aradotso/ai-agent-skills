---
name: awesome-long-horizon-agents-survey
description: Comprehensive resource and taxonomy for long-horizon AI agents research, covering foundations, harnesses, optimization, and applications
triggers:
  - "show me research on long-horizon agents"
  - "what are the latest papers on AI agent capabilities"
  - "how do I build agents that work on multi-step tasks"
  - "explain long-horizon agent architectures"
  - "find papers about agent memory and context"
  - "what's the difference between agent harness and optimization"
  - "benchmark datasets for evaluating AI agents"
  - "research on agent reasoning over long time horizons"
---

# Awesome Long-Horizon Agents Survey

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill helps you navigate and utilize the **RUC-NLPIR/Awesome-Long-Horizon-Agents** repository, a comprehensive survey and curated reading list covering the landscape of long-horizon AI agents. The resource organizes research into two main pillars: **externalized harness engineering** (loops, memory, tools, orchestration) and **internalized model optimization** (training, RL, self-evolution).

## Overview

The repository accompanies the paper ["Towards Long-Horizon Agents: A Survey"](https://openreview.net/pdf?id=HyhfhlbWGh) and provides:

- **Three-level formalization** (H1→H2→H3) of long-horizon tasks
- **Evolution timeline** from prompt engineering (2020-2023) to runtime harnesses (2025+)
- **Systematic taxonomy** of agent capabilities and implementation patterns
- **Curated paper lists** organized by technique and application domain
- **Benchmarks and evaluation frameworks** for long-horizon tasks

### Key Concept: Long-Horizon Defined

The survey defines long-horizon agents across three nested levels:

| Level | Horizon | Capability Required |
|-------|---------|---------------------|
| **H1** | Intra-context (~minutes) | Interactive reasoning within one context window |
| **H2** | Cross-context (~hours-days) | State persistence and memory across sessions |
| **H3** | Cross-task (open-ended) | Experience accumulation and self-evolution |

**Agent Formula**: `Agent = π_θ ⊕ H` (base policy + harness)

## Installation & Access

### Clone the Repository

```bash
git clone https://github.com/RUC-NLPIR/Awesome-Long-Horizon-Agents.git
cd Awesome-Long-Horizon-Agents
```

### Access the Resources

The repository is primarily a curated markdown document with links to papers, not executable code. Use it as:

1. **Reference guide** for researching agent techniques
2. **Reading list** organized by capability area
3. **Taxonomy** for categorizing your own agent implementations

### Quick Links

- **Paper**: [OpenReview](https://openreview.net/pdf?id=HyhfhlbWGh) | [Preprint](https://www.preprints.org/manuscript/202607.1328)
- **Website**: [Long-Horizon-Agents.github.io](https://Long-Horizon-Agents.github.io)
- **Discussion**: [X Thread](https://x.com/kakakbibibi/status/2078076130037514640)

## Key Research Areas & Paper Collections

### 1. Foundations: Formalizing Agents

**Core Papers**:
- **ReAct** (ICLR 2023): Synergizing reasoning and acting - [arxiv:2210.03629](https://arxiv.org/abs/2210.03629)
- **Chain-of-Thought** (NeurIPS 2022): Multi-step reasoning - [arxiv:2201.11903](https://arxiv.org/abs/2201.11903)
- **Tree of Thoughts** (NeurIPS 2023): Deliberate problem solving - [arxiv:2305.10601](https://arxiv.org/abs/2305.10601)

**Use Case**: Understanding foundational agent reasoning patterns

```python
# Example: Implementing ReAct pattern
class ReActAgent:
    def __init__(self, llm, tools):
        self.llm = llm
        self.tools = tools
    
    def run(self, task):
        thought = self.llm.generate(f"Thought: {task}")
        action = self.parse_action(thought)
        observation = self.tools[action['name']].execute(action['args'])
        
        # Iterate: Thought → Action → Observation
        return self.llm.generate(f"Based on {observation}, the answer is...")
    
    def parse_action(self, thought):
        # Extract action from "Action: tool_name[arg1, arg2]"
        return {'name': 'search', 'args': ['query']}
```

### 2. Harnesses: External Capabilities (Pillar I)

#### Loops and Workflows

**Key Papers**:
- **Reflexion** - Self-reflection for iterative improvement
- **Voyager** - Curriculum learning in open worlds
- **AutoGPT** - Autonomous task execution loops

**Pattern**: Agent control flow architecture

```python
# Example: Multi-agent orchestration harness
class AgentHarness:
    def __init__(self, agents: dict, memory: Memory):
        self.agents = agents
        self.memory = memory
    
    async def execute_workflow(self, task):
        """H2-level: Cross-context workflow"""
        # 1. Planning phase
        plan = await self.agents['planner'].decompose(task)
        self.memory.save('plan', plan)
        
        # 2. Execution loop
        for step in plan.steps:
            context = self.memory.retrieve_relevant(step)
            result = await self.agents['executor'].run(step, context)
            self.memory.save(f'step_{step.id}', result)
            
            # Verification hook
            if not self.verify(result):
                result = await self.agents['refiner'].fix(result, context)
        
        # 3. Synthesis
        return self.agents['synthesizer'].combine(self.memory.get_all())
    
    def verify(self, result):
        return result.confidence > 0.8
```

#### Context and Memory

**Key Papers**:
- **RAG** (NeurIPS 2020): Retrieval-Augmented Generation
- **RAPTOR** (ICLR 2024): Tree-organized hierarchical retrieval
- **MemGPT**: Operating system for LLM memory management

**Pattern**: Long-term memory systems

```python
# Example: Hierarchical memory with RAPTOR-style retrieval
from sentence_transformers import SentenceTransformer

class HierarchicalMemory:
    def __init__(self, embedding_model='all-MiniLM-L6-v2'):
        self.encoder = SentenceTransformer(embedding_model)
        self.short_term = []  # Recent context
        self.long_term = {}   # Episodic memory
        self.semantic = {}    # Indexed by topic clusters
    
    def add(self, text, metadata=None):
        """Add to short-term, periodically consolidate"""
        embedding = self.encoder.encode(text)
        self.short_term.append({
            'text': text,
            'embedding': embedding,
            'metadata': metadata,
            'timestamp': time.time()
        })
        
        if len(self.short_term) > 100:
            self.consolidate()
    
    def retrieve_relevant(self, query, k=5):
        """H2 capability: Cross-context retrieval"""
        query_emb = self.encoder.encode(query)
        
        # Search across both short and long-term
        all_memories = self.short_term + list(self.long_term.values())
        scores = [cosine_similarity(query_emb, m['embedding']) 
                  for m in all_memories]
        
        top_k = sorted(zip(scores, all_memories), reverse=True)[:k]
        return [m['text'] for _, m in top_k]
    
    def consolidate(self):
        """Move short-term to long-term with summarization"""
        summary = self.summarize_cluster(self.short_term)
        self.long_term[summary['id']] = summary
        self.short_term = []
```

#### Tools, MCP, and Skills

**Model Context Protocol (MCP)**: Standard for agent-tool integration

```python
# Example: MCP-compliant tool integration
from typing import Protocol

class MCPTool(Protocol):
    """Standard interface for agent tools"""
    name: str
    description: str
    
    def get_schema(self) -> dict:
        """Return JSON schema for tool parameters"""
        ...
    
    async def execute(self, **kwargs) -> dict:
        """Execute tool and return structured result"""
        ...

class WebSearchTool:
    name = "web_search"
    description = "Search the web for current information"
    
    def get_schema(self):
        return {
            "type": "object",
            "properties": {
                "query": {"type": "string"},
                "num_results": {"type": "integer", "default": 5}
            },
            "required": ["query"]
        }
    
    async def execute(self, query: str, num_results: int = 5):
        # Integration with search API
        import os
        api_key = os.getenv('SEARCH_API_KEY')
        results = await search_api.query(query, limit=num_results)
        return {"results": results, "count": len(results)}

# Agent uses tools via MCP
class MCPAgent:
    def __init__(self, llm, tools: list[MCPTool]):
        self.llm = llm
        self.tools = {t.name: t for t in tools}
    
    async def run(self, task):
        # LLM selects tool based on schemas
        tool_schemas = {n: t.get_schema() for n, t in self.tools.items()}
        decision = self.llm.select_tool(task, tool_schemas)
        
        # Execute via MCP interface
        result = await self.tools[decision['tool']].execute(**decision['args'])
        return self.llm.synthesize(task, result)
```

#### Verification

**Key Papers**:
- **Self-Consistency**: Multiple sampling for verification
- **Critic models**: Learned verification functions
- **Process supervision**: Step-by-step correctness checking

```python
# Example: Multi-stage verification
class VerificationHarness:
    def __init__(self, critic_model, test_suite):
        self.critic = critic_model
        self.tests = test_suite
    
    async def verify_with_feedback(self, agent_output, task):
        """Multi-level verification with repair loop"""
        # 1. Syntax/format check
        if not self.validate_format(agent_output):
            return {'valid': False, 'feedback': 'Format error'}
        
        # 2. Semantic correctness (critic model)
        critique = await self.critic.evaluate(agent_output, task)
        if critique.score < 0.7:
            return {'valid': False, 'feedback': critique.reasoning}
        
        # 3. Execution tests (for code generation tasks)
        if self.tests:
            test_results = self.tests.run(agent_output)
            if not test_results.all_passed():
                return {
                    'valid': False,
                    'feedback': f'Failed tests: {test_results.failures}'
                }
        
        return {'valid': True}
    
    def validate_format(self, output):
        # Check structure, required fields, etc.
        return True
```

### 3. Optimization: Internal Capabilities (Pillar II)

#### Agentic Reinforcement Learning

**Key Papers**:
- **AgentQ**: Online RL for web agents
- **Agent-FLAN**: Multi-task instruction tuning
- **Reflexion**: Reinforcement via verbal feedback

**Pattern**: Training agents with trajectory feedback

```python
# Conceptual example: RL training loop for agents
class AgentRLTrainer:
    def __init__(self, base_model, environment, reward_model):
        self.policy = base_model
        self.env = environment
        self.reward_model = reward_model
    
    def train_episode(self, task):
        """Single training episode with trajectory collection"""
        trajectory = []
        state = self.env.reset(task)
        
        for step in range(max_steps):
            # Agent takes action
            action = self.policy.generate_action(state)
            next_state, env_reward = self.env.step(action)
            
            # Compute learned reward (outcome + process)
            reward = self.reward_model.score(
                state=state,
                action=action,
                outcome=next_state,
                success=env_reward
            )
            
            trajectory.append({
                'state': state,
                'action': action,
                'reward': reward
            })
            
            if self.env.is_done():
                break
            state = next_state
        
        # Update policy using PPO/DPO
        self.update_policy(trajectory)
        return sum(t['reward'] for t in trajectory)
    
    def update_policy(self, trajectory):
        """Update using preference optimization"""
        # Implement DPO, PPO, or GRPO update
        pass
```

#### Self-Evolution

**Key Papers**:
- **Voyager**: Skill library via self-play
- **AutoGPT**: Autonomous capability expansion
- **Self-Instruct**: Bootstrap via self-generated data

**Pattern**: H3-level cross-task learning

```python
# Example: Skill library with self-evolution
class EvolvingAgent:
    def __init__(self, base_model):
        self.model = base_model
        self.skill_library = {}
        self.experience_buffer = []
    
    def execute_and_learn(self, task):
        """H3: Learn from task execution"""
        # Try to use existing skills
        relevant_skills = self.match_skills(task)
        
        result = self.model.run(
            task,
            context=relevant_skills,
            exploration=True
        )
        
        # Store successful patterns
        if result.success:
            self.experience_buffer.append({
                'task': task,
                'solution': result.trajectory,
                'performance': result.metrics
            })
            
            # Periodically distill into reusable skills
            if len(self.experience_buffer) > 100:
                self.evolve_skills()
        
        return result
    
    def evolve_skills(self):
        """Abstract common patterns into skills"""
        # Cluster similar successful trajectories
        clusters = self.cluster_experiences(self.experience_buffer)
        
        for cluster in clusters:
            # Generate skill from cluster
            skill = self.model.abstract_skill(cluster)
            skill_name = f"skill_{len(self.skill_library)}"
            
            # Validate on held-out tasks
            if self.validate_skill(skill):
                self.skill_library[skill_name] = skill
        
        self.experience_buffer = []  # Clear after learning
    
    def match_skills(self, task):
        """Retrieve relevant skills for task"""
        return [s for s in self.skill_library.values() 
                if s.is_applicable(task)]
```

## Application Domains

### Software Engineering

**Key Projects**:
- **SWE-agent**: Solves GitHub issues via command-line interface
- **Devin**: Autonomous software engineer
- **AutoCodeRover**: Automated program repair

```python
# Pattern: Code generation with verification
class CodeAgent:
    def solve_issue(self, repo_path, issue_description):
        # 1. Understand codebase
        codebase = self.analyze_repo(repo_path)
        
        # 2. Generate fix with context
        fix = self.model.generate_code(
            issue=issue_description,
            context=codebase.relevant_files,
            constraints=codebase.style_guide
        )
        
        # 3. Verify via tests
        test_results = self.run_tests(repo_path, fix)
        
        if not test_results.passed:
            # Iterative refinement (H1)
            fix = self.model.refine_code(fix, test_results.errors)
        
        return fix
```

### Computer Use

**Key Projects**:
- **Claude Computer Use**: Control desktop via screenshots
- **OpenHands**: General computer control agent
- **OS-Copilot**: Operating system automation

```python
# Pattern: Visual grounding for GUI control
class ComputerUseAgent:
    def __init__(self, vision_model, action_model):
        self.vision = vision_model
        self.action = action_model
    
    async def execute_task(self, instruction):
        """Control computer via vision + actions"""
        for _ in range(max_steps):
            # Perceive current state
            screenshot = self.capture_screen()
            state = self.vision.parse_ui(screenshot)
            
            # Decide next action
            action = self.action.plan(
                goal=instruction,
                current_state=state
            )
            
            # Execute (click, type, etc.)
            await self.execute_action(action)
            
            if self.task_complete(instruction):
                break
```

## Benchmark Datasets

The survey references key evaluation benchmarks:

| Benchmark | Focus | Horizon Level |
|-----------|-------|---------------|
| **SWE-bench** | GitHub issue resolution | H1-H2 |
| **WebArena** | Web task completion | H1 |
| **GAIA** | General assistant tasks | H2 |
| **AgentBench** | Multi-domain evaluation | H1-H2 |
| **METR Task Standard** | Time-to-completion metric | H1-H3 |

### Using Benchmarks

```python
# Example: Evaluate on multi-step benchmark
from agent_benchmark import SWEBench

def evaluate_agent(agent, benchmark='swe-bench-lite'):
    dataset = SWEBench.load(benchmark)
    results = []
    
    for task in dataset:
        result = agent.solve_issue(
            repo=task.repo,
            issue=task.issue_description
        )
        
        # Check if fix resolves issue and passes tests
        success = benchmark.verify(result, task.ground_truth)
        results.append({
            'task_id': task.id,
            'success': success,
            'steps': len(result.trajectory),
            'time': result.duration
        })
    
    # Calculate 50%-completion horizon
    horizon = calculate_horizon(results, success_rate=0.5)
    return {
        'success_rate': sum(r['success'] for r in results) / len(results),
        'avg_steps': sum(r['steps'] for r in results) / len(results),
        'horizon_50': horizon
    }
```

## Common Patterns & Best Practices

### Pattern 1: ReAct Loop with Memory

```python
class PersistentReActAgent:
    """Combines ReAct reasoning with cross-context memory (H2)"""
    
    def __init__(self, llm, tools, memory):
        self.llm = llm
        self.tools = tools
        self.memory = memory
    
    async def run(self, task, session_id):
        # Retrieve relevant past experiences
        context = self.memory.retrieve_by_session(session_id)
        
        max_iterations = 20
        for i in range(max_iterations):
            # ReAct cycle
            prompt = self.build_prompt(task, context, i)
            response = await self.llm.generate(prompt)
            
            if response.is_final_answer():
                self.memory.save(session_id, {
                    'task': task,
                    'answer': response.answer,
                    'trajectory': context
                })
                return response.answer
            
            # Execute action and observe
            action = self.parse_action(response)
            observation = await self.tools[action.tool].execute(**action.args)
            
            # Add to context for next iteration
            context.append({
                'thought': response.thought,
                'action': action,
                'observation': observation
            })
        
        return "Task exceeded max iterations"
```

### Pattern 2: Hierarchical Planning

```python
class HierarchicalAgent:
    """Decompose long-horizon tasks into subtasks (H2→H1)"""
    
    async def solve(self, complex_task):
        # 1. High-level planning
        plan = await self.planner.decompose(complex_task)
        
        # 2. Execute subtasks with specialized agents
        results = {}
        for subtask in plan.subtasks:
            agent = self.select_specialist(subtask.type)
            results[subtask.id] = await agent.execute(subtask)
            
            # Update plan based on results (adaptive)
            plan = self.planner.replan(plan, results)
        
        # 3. Synthesize results
        return self.synthesizer.combine(results, complex_task)
    
    def select_specialist(self, task_type):
        """Route to specialized agent (orchestration)"""
        specialists = {
            'code': self.code_agent,
            'search': self.search_agent,
            'analysis': self.analysis_agent
        }
        return specialists.get(task_type, self.general_agent)
```

### Pattern 3: Verification-Driven Refinement

```python
async def verified_generation(agent, task, verifier, max_attempts=3):
    """Generate with verification loop"""
    for attempt in range(max_attempts):
        result = await agent.generate(task)
        verification = await verifier.check(result, task)
        
        if verification.passed:
            return result
        
        # Refine based on feedback
        task = task.with_feedback(verification.errors)
    
    raise Exception("Could not generate valid result")
```

## Troubleshooting

### Issue: Agent Gets Stuck in Loops

**Solution**: Implement loop detection and breaking

```python
class LoopDetector:
    def __init__(self, window=5):
        self.history = []
        self.window = window
    
    def add_step(self, state):
        self.history.append(state)
        if len(self.history) > self.window:
            self.history.pop(0)
    
    def is_looping(self):
        """Detect repeated states"""
        if len(self.history) < self.window:
            return False
        
        # Check for exact repetition
        recent = self.history[-self.window//2:]
        earlier = self.history[:self.window//2]
        return recent == earlier
```

### Issue: Context Window Overflow (H1→H2 Transition)

**Solution**: Implement memory consolidation

```python
def summarize_context(long_context, max_tokens=4000):
    """Compress context when approaching limits"""
    if len(long_context) < max_tokens:
        return long_context
    
    # Keep most recent + summarize earlier
    recent = long_context[-1000:]
    earlier = long_context[:-1000]
    
    summary = llm.summarize(earlier, max_length=max_tokens - 1000)
    return summary + "\n\n[Recent context]\n" + recent
```

### Issue: Poor Tool Selection

**Solution**: Provide better tool descriptions and examples

```python
def enhance_tool_schema(tool):
    """Add usage examples to tool schema"""
    schema = tool.get_schema()
    schema['examples'] = [
        {
            'input': {'query': 'current weather in Paris'},
            'output': {'temp': 18, 'condition': 'cloudy'},
            'when_to_use': 'User asks about current conditions'
        }
    ]
    return schema
```

## Research & Development Workflow

### 1. Exploring Papers by Topic

```bash
# Clone and search the repository
git clone https://github.com/RUC-NLPIR/Awesome-Long-Horizon-Agents.git
cd Awesome-Long-Horizon-Agents

# Search for specific topics
grep -n "memory" README.md | head -20
grep -n "reinforcement learning" README.md
```

### 2. Building on Survey Taxonomy

When designing your own agent:

1. **Identify horizon level** (H1/H2/H3) for your target tasks
2. **Choose harness components** (which loops, memory, tools needed)
3. **Select optimization strategy** (fine-tuning, RL, self-evolution)
4. **Reference relevant papers** from corresponding sections

### 3. Contributing to the Survey

```bash
# Fork and add missing papers
git checkout -b add-paper-xyz

# Follow the format: [Venue Year] Title. [[paper](url)] [[code](url)]
echo "- **\`NeurIPS 2026\`** Your Paper Title. [[paper](https://arxiv.org/abs/xxxx)]" >> README.md

git commit -m "Add: Your Paper Title"
git push origin add-paper-xyz
# Open PR on GitHub
```

## Key Takeaways for Implementation

1. **Start with H1**: Build intra-context agents before tackling cross-context tasks
2. **Harness first, optimize later**: Externalize capabilities in the harness, then consider internalizing via training
3. **Memory is critical for H2**: Implement effective retrieval for cross-session persistence
4. **Verification enables iteration**: Add verification loops to improve reliability
5. **Measure horizon empirically**: Use time-to-completion at fixed success rate (METR methodology)

## Citation

When using this survey in your research:

```bibtex
@article{dong2026longhorizon,
    title={Towards Long-Horizon Agents: A Survey},
    author={Dong, Guanting and Song, Xiaoshuai and Hu, Yuyang and others},
    journal={Preprints},
    year={2026},
    doi={10.20944/preprints202607.1328.v1},
    url={https://openreview.net/pdf?id=HyhfhlbWGh}
}
```

## Additional Resources

- **Website**: [Long-Horizon-Agents.github.io](https://Long-Horizon-Agents.github.io)
- **Discussion**: [X/Twitter thread](https://x.com/kakakbibibi/status/2078076130037514640)
- **Chinese coverage**: [机器之心微信文章](https://mp.weixin.qq.com/s/r9YJYlVAyBZtfMXvAOh5ig)
- **Related surveys**: AgentBench, Agent AI survey papers linked in repository

This skill provides the conceptual framework and implementation patterns for building long-horizon agents based on the comprehensive survey taxonomy.
