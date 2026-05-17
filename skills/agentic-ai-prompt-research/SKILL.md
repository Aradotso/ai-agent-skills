---
name: agentic-ai-prompt-research
description: Research collection of reconstructed prompt patterns and architectures for agentic AI coding assistants
triggers:
  - "show me agentic AI prompt patterns"
  - "how do AI coding assistants work internally"
  - "explain prompt architecture for autonomous agents"
  - "what are the system prompts for Claude Code"
  - "help me design a multi-agent coding system"
  - "show me security patterns for AI tool approval"
  - "how to build context window management"
  - "explain agent coordination patterns"
---

# Agentic AI Prompt Research

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This project documents reconstructed prompt patterns and architectural designs from modern agentic AI coding assistants. It provides educational insights into how systems like Claude Code assemble dynamic prompts, coordinate multiple agents, manage security, and handle context windows.

## What This Project Provides

A collection of 30+ documented patterns covering:

- **Core Identity**: Main system prompts, simple mode, default agent instructions, security boundaries
- **Orchestration**: Coordinator prompts, multi-agent communication protocols
- **Specialized Agents**: Verification, exploration, agent creation, configuration agents
- **Security**: Permission explainers, auto-approval classifiers, risk assessment
- **Context Management**: Conversation compaction, memory selection, session search
- **Skills**: Reusable multi-agent workflows (simplify, skillify, stuck, remember)

All patterns are based on behavioral observation and reverse engineering, not leaked proprietary code.

## Installation

```bash
# Clone the repository
git clone https://github.com/Leonxlnx/agentic-ai-prompt-research.git
cd agentic-ai-prompt-research

# Browse the prompts directory
ls prompts/
```

No build or installation required — this is a documentation and research repository.

## Repository Structure

```
prompts/
├── 01_main_system_prompt.md          # Dynamic prompt assembly
├── 02_simple_mode.md                 # Minimal operation mode
├── 03_default_agent_prompt.md        # Base agent instructions
├── 04_cyber_risk_instruction.md      # Security boundaries
├── 05_coordinator_system_prompt.md   # Multi-agent orchestration
├── 06_teammate_prompt_addendum.md    # Agent communication
├── 07_verification_agent.md          # Adversarial testing
├── 08_explore_agent.md               # Read-only exploration
├── 09_agent_creation_architect.md    # Dynamic agent generation
├── 10_statusline_setup_agent.md      # Terminal configuration
├── 11_permission_explainer.md        # Risk assessment
├── 12_yolo_auto_mode_classifier.md   # Security classification
├── 13_tool_prompts.md                # Tool-specific instructions
├── 14_tool_use_summary.md            # Action summarization
├── 15_session_search.md              # Semantic search
├── 16_memory_selection.md            # Context selection
├── 17_auto_mode_critique.md          # Classifier review
├── 18_proactive_mode.md              # Autonomous operation
├── 19_simplify_skill.md              # Code review pattern
├── 20_session_title.md               # Title generation
├── 21_compact_service.md             # Context compression
├── 22_away_summary.md                # Session recaps
├── 23_chrome_browser_automation.md   # Browser integration
├── 24_memory_instruction.md          # Memory hierarchy
├── 25_skillify.md                    # Skill creation workflow
├── 26_stuck_skill.md                 # Diagnostic patterns
├── 27_remember_skill.md              # Memory management
├── 28_update_config_skill.md         # Configuration updates
├── 29_agent_summary.md               # Progress updates
└── 30_prompt_suggestion.md           # Follow-up prediction
```

## Key Architectural Patterns

### 1. Dynamic Prompt Assembly

The system assembles prompts from modular components:

```
┌─────────────────────────────────────┐
│   Cacheable Prefix (stable)         │
│   - Identity & safety rules         │
│   - Permission configuration        │
│   - Code style preferences          │
│   - Tool usage patterns             │
├─────────────────────────────────────┤  ← Cache boundary
│   Dynamic Suffix (per-session)      │
│   - Available agents/skills         │
│   - Memory file contents            │
│   - Environment context             │
│   - Active MCP servers              │
└─────────────────────────────────────┘
```

**Example pattern from `01_main_system_prompt.md`:**

```markdown
# Core identity established first
You are Claude Code, an agentic AI coding assistant...

# Tool preferences defined
When editing files, prefer multi_file_edit for batching...

# Security boundaries set
Never execute commands that could compromise user data...

# Dynamic sections injected
[AVAILABLE_AGENTS: verification, explore, statusline_setup]
[MEMORY_FILES: .claude/project_rules.md, .claude/preferences.md]
[ENVIRONMENT: OS=linux, SHELL=bash, CWD=/home/user/project]
```

### 2. Multi-Agent Coordination

**Coordinator Pattern** (`05_coordinator_system_prompt.md`):

```markdown
## Phased Workflow

1. **Planning Phase**: Break task into subtasks
2. **Delegation Phase**: Assign workers with specific contexts
3. **Synthesis Phase**: Merge results and resolve conflicts

## Worker Communication

- Workers receive: task description, relevant files, constraints
- Workers return: results, confidence score, blockers
- Coordinator decides: accept, retry, escalate
```

**Implementation approach:**

```python
# Conceptual multi-agent orchestration
class AgentCoordinator:
    def execute_task(self, user_request: str):
        # Phase 1: Planning
        subtasks = self.plan(user_request)
        
        # Phase 2: Delegation
        workers = [
            self.spawn_agent("worker", task=t, context=self.get_context(t))
            for t in subtasks
        ]
        results = [w.execute() for w in workers]
        
        # Phase 3: Synthesis
        return self.merge_results(results)
    
    def spawn_agent(self, agent_type: str, task: str, context: dict):
        # Load base prompt + agent-specific addendum
        base_prompt = self.load_prompt("03_default_agent_prompt.md")
        agent_prompt = self.load_prompt(f"{agent_type}_prompt.md")
        
        return Agent(
            system_prompt=f"{base_prompt}\n\n{agent_prompt}",
            task=task,
            context=context
        )
```

### 3. Security Classification

**Multi-stage auto-approval** (`12_yolo_auto_mode_classifier.md`):

```python
class SecurityClassifier:
    def classify_tool_call(self, tool: str, args: dict) -> str:
        """Returns: 'safe', 'unsafe', or 'uncertain'"""
        
        # Stage 1: Fast predefined rules
        if tool == "bash" and "rm -rf" in args.get("command", ""):
            return "unsafe"
        if tool == "read_file" and not self.accesses_sensitive_path(args["path"]):
            return "safe"
        
        # Stage 2: User-defined overrides
        for rule in self.user_classifier_rules:
            result = rule.evaluate(tool, args)
            if result != "uncertain":
                return result
        
        # Stage 3: Extended reasoning (slower)
        return self.llm_classify_with_reasoning(tool, args)
    
    def accesses_sensitive_path(self, path: str) -> bool:
        sensitive = ["/etc/passwd", "~/.ssh", ".env"]
        return any(s in path for s in sensitive)
```

**User-configurable rules** (`.claude/auto_mode_rules.md`):

```yaml
rules:
  - pattern: "read_file:docs/**"
    verdict: safe
    reason: "Documentation is always safe to read"
  
  - pattern: "bash:git push *"
    verdict: unsafe
    reason: "Always confirm before pushing code"
  
  - pattern: "edit_file:**/test_*.py"
    verdict: safe
    reason: "Test file edits are low-risk"
```

### 4. Memory Hierarchy

**Loading order** (`24_memory_instruction.md`):

```python
class MemoryLoader:
    def load_context(self, project_path: str) -> str:
        """Load memory files in priority order (earliest = lowest priority)"""
        
        layers = [
            # 1. Enterprise/managed configuration
            self.load_if_exists("/etc/claude/enterprise_policy.md"),
            
            # 2. User global preferences
            self.load_if_exists("~/.claude/global_preferences.md"),
            
            # 3. Project-level shared instructions
            self.load_if_exists(f"{project_path}/.claude/project_rules.md"),
            
            # 4. Project rules directory (supports includes)
            *self.load_directory(f"{project_path}/.claude/rules/"),
            
            # 5. Local overrides (private, gitignored)
            self.load_if_exists(f"{project_path}/.claude/local_overrides.md"),
        ]
        
        # Later layers override earlier ones
        return self.merge_with_precedence(layers)
    
    def merge_with_precedence(self, layers: list[str]) -> str:
        """Handle conflicting instructions by priority"""
        merged = {}
        for layer in layers:
            directives = self.parse_directives(layer)
            merged.update(directives)  # Later overwrites earlier
        return self.serialize(merged)
```

**Transitive includes:**

```markdown
<!-- project_rules.md -->
# Project Rules

@include ./rules/code_style.md
@include ./rules/testing_requirements.md

<!-- Conditional inclusion -->
@include ./rules/python_specific.md if file_extension == ".py"
```

### 5. Context Window Management

**Compaction strategy** (`21_compact_service.md`):

```python
class ContextCompactor:
    def compact_conversation(self, messages: list[dict]) -> list[dict]:
        """Summarize old messages to fit within context window"""
        
        # Keep recent messages verbatim
        recent_cutoff = len(messages) - 10
        recent = messages[recent_cutoff:]
        old = messages[:recent_cutoff]
        
        # Identify which old messages to keep fully
        important = self.filter_important(old)  # Tool uses, errors, decisions
        
        # Summarize the rest
        summaries = self.batch_summarize(
            [m for m in old if m not in important],
            max_tokens_per_summary=150
        )
        
        return summaries + important + recent
    
    def filter_important(self, messages: list[dict]) -> list[dict]:
        """Keep tool uses, errors, and key decisions"""
        important = []
        for msg in messages:
            if msg.get("tool_use"):
                important.append(msg)
            elif "error" in msg.get("content", "").lower():
                important.append(msg)
            elif msg.get("flagged_as_important"):
                important.append(msg)
        return important
```

### 6. Specialized Agent Patterns

**Verification Agent** (`07_verification_agent.md`):

```markdown
## Your Role

You are an adversarial testing agent. After another agent implements a feature,
your job is to break it.

## Testing Strategy

1. **Read the implementation** - Understand what was built
2. **Generate test cases** - Focus on edge cases and error conditions
3. **Execute tests** - Run them and document failures
4. **Report findings** - Clear reproduction steps

## Test Categories

- Boundary conditions (empty input, max values)
- Error handling (invalid input, network failures)
- Race conditions (concurrent access)
- Security (injection, unauthorized access)

## Constraints

- Read-only access to implementation
- Create test files in `tests/` directory
- Use project's testing framework
- No modifications to implementation code
```

**Explore Agent** (`08_explore_agent.md`):

```markdown
## Your Role

You explore codebases to answer questions. You have read-only access.

## Available Tools

- `read_file`: Read any file
- `list_directory`: Browse directory structure
- `search_code`: Semantic code search
- `grep`: Pattern matching across files

## Constraints

- NEVER use edit_file or write_file
- NEVER use bash to modify files
- Focus on understanding, not changing

## Exploration Strategy

1. Start broad (directory structure, README)
2. Identify entry points (main files, key modules)
3. Follow dependencies
4. Document findings concisely
```

### 7. Skill Patterns

**Simplify Skill** (`19_simplify_skill.md`) - Multi-agent parallel review:

```python
class SimplifySkill:
    """Spawn multiple agents to review code in parallel"""
    
    def execute(self, target_files: list[str]):
        # Spawn review agents in parallel
        agents = [
            self.spawn_agent("reviewer", {
                "file": f,
                "focus": "complexity",
                "constraints": "suggest simplifications, not rewrites"
            })
            for f in target_files
        ]
        
        # Collect suggestions
        suggestions = [a.execute() for a in agents]
        
        # Coordinator merges and deduplicates
        return self.merge_suggestions(suggestions)
```

**Skillify Skill** (`25_skillify.md`) - Interview-based skill creation:

```markdown
## Process

1. **Interview user** about the skill they want to create
   - What problem does it solve?
   - What tools/agents are needed?
   - What are success criteria?

2. **Generate skill specification**
   ```yaml
   name: custom-skill-name
   description: One-line description
   triggers: [list of natural language triggers]
   agents: [required agent types]
   tools: [required tool access]
   workflow: [step-by-step process]
   ```

3. **Write skill implementation** as markdown file

4. **Test skill** with sample scenarios

5. **Save to** `.claude/skills/custom-skill-name.md`
```

## Usage Examples

### Building a Custom Agent System

```python
# Using patterns from this research to build your own agent

import anthropic

class CustomAgentSystem:
    def __init__(self, api_key: str):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.prompts = self.load_prompt_library()
    
    def load_prompt_library(self) -> dict:
        """Load reconstructed prompts from this repository"""
        return {
            "coordinator": open("prompts/05_coordinator_system_prompt.md").read(),
            "default_agent": open("prompts/03_default_agent_prompt.md").read(),
            "security": open("prompts/04_cyber_risk_instruction.md").read(),
        }
    
    def create_coordinator(self, task: str) -> str:
        """Create a coordinator agent for a complex task"""
        system_prompt = f"""
{self.prompts['default_agent']}

{self.prompts['coordinator']}

{self.prompts['security']}

Available sub-agents: explore, verification, implementation
Current task: {task}
"""
        
        response = self.client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=4000,
            system=system_prompt,
            messages=[{"role": "user", "content": task}]
        )
        
        return response.content[0].text
```

### Implementing Auto-Approval Logic

```python
import re

class ToolApprovalSystem:
    def __init__(self):
        # Based on 12_yolo_auto_mode_classifier.md
        self.safe_patterns = [
            (r"read_file", lambda args: not self._is_sensitive(args["path"])),
            (r"list_directory", lambda args: True),
            (r"search_code", lambda args: True),
        ]
        
        self.unsafe_patterns = [
            (r"bash:rm -rf", lambda args: True),
            (r"bash:sudo", lambda args: True),
            (r"edit_file:.env", lambda args: True),
            (r"bash:git push", lambda args: True),
        ]
    
    def classify(self, tool: str, args: dict) -> str:
        """Returns: 'approve', 'reject', or 'ask_user'"""
        
        tool_str = f"{tool}:{args.get('command', args.get('path', ''))}"
        
        # Check unsafe patterns first
        for pattern, condition in self.unsafe_patterns:
            if re.search(pattern, tool_str) and condition(args):
                return "reject"
        
        # Check safe patterns
        for pattern, condition in self.safe_patterns:
            if re.search(pattern, tool_str) and condition(args):
                return "approve"
        
        # Uncertain - ask user
        return "ask_user"
    
    def _is_sensitive(self, path: str) -> bool:
        sensitive = [".env", ".ssh", "password", "secret", "/etc/"]
        return any(s in path.lower() for s in sensitive)
```

### Memory System Implementation

```python
import os
from pathlib import Path

class MemorySystem:
    def __init__(self, project_root: str):
        self.project_root = Path(project_root)
        self.memory_dir = self.project_root / ".claude"
    
    def load_all_context(self) -> str:
        """Load memory files in precedence order"""
        
        memory_files = [
            Path.home() / ".claude" / "global_preferences.md",
            self.memory_dir / "project_rules.md",
            *self.memory_dir.glob("rules/*.md"),
            self.memory_dir / "local_overrides.md",
        ]
        
        context_parts = []
        for file_path in memory_files:
            if file_path.exists():
                content = file_path.read_text()
                # Process includes
                content = self._process_includes(content, file_path.parent)
                context_parts.append(f"## From {file_path.name}\n\n{content}")
        
        return "\n\n".join(context_parts)
    
    def _process_includes(self, content: str, base_dir: Path) -> str:
        """Handle @include directives"""
        import re
        
        def replace_include(match):
            include_path = match.group(1)
            full_path = base_dir / include_path
            if full_path.exists():
                return full_path.read_text()
            return f"<!-- Include not found: {include_path} -->"
        
        return re.sub(r'@include\s+(.+)', replace_include, content)
    
    def save_memory(self, name: str, content: str):
        """Save a new memory file"""
        self.memory_dir.mkdir(exist_ok=True)
        (self.memory_dir / f"{name}.md").write_text(content)
```

## Common Patterns for AI Agent Builders

### 1. Modular Prompt Assembly

Don't hardcode monolithic prompts. Use composition:

```python
def build_agent_prompt(role: str, context: dict) -> str:
    return "\n\n".join([
        load_prompt("base_identity"),
        load_prompt(f"role_{role}"),
        load_prompt("security_boundaries"),
        format_dynamic_context(context),
    ])
```

### 2. Tool Call Batching

Reduce round-trips by batching related operations:

```python
# Instead of: read file1, read file2, read file3
# Prefer: read multiple files in one call
{
    "tool": "multi_file_read",
    "files": ["file1.py", "file2.py", "file3.py"]
}
```

### 3. Progressive Disclosure

Start simple, add complexity as needed:

```python
if context_window_usage < 0.5:
    # Full verbose mode
    system_prompt = build_full_prompt()
else:
    # Compact mode with summarized history
    system_prompt = build_simple_prompt()
```

### 4. Adversarial Validation

Always use a separate agent to verify work:

```python
def implement_and_verify(task: str):
    # Agent 1: Implementation
    implementation = implementation_agent.execute(task)
    
    # Agent 2: Verification (adversarial)
    verification = verification_agent.test(implementation)
    
    if verification.passed:
        return implementation
    else:
        return implementation_agent.fix(verification.issues)
```

## Configuration Patterns

### Project-Level Configuration

```markdown
<!-- .claude/project_rules.md -->

# Code Style

- Use TypeScript strict mode
- Prefer functional components
- Maximum line length: 100 characters

# Testing Requirements

- All public functions must have unit tests
- Minimum coverage: 80%
- Use Jest for testing

# Auto-Approval Rules

@include ./rules/auto_approve.yaml

# Memory Organization

@include ./rules/memory_structure.md
```

### User-Level Preferences

```markdown
<!-- ~/.claude/global_preferences.md -->

# Output Style

- Be concise
- Use emojis for status indicators
- Prefer markdown tables for structured data

# Tool Preferences

- Use multi_file_edit over single edit_file
- Prefer ripgrep over grep when available
- Always confirm before git push
```

## Troubleshooting

### Issue: Prompt Assembly Not Working

**Symptom:** Dynamic context not appearing in agent responses

**Solution:** Check cache boundaries and invalidation:

```python
# Ensure dynamic content comes AFTER cache boundary
prompt = f"""
{CACHEABLE_PREFIX}

--- CACHE BOUNDARY ---

{dynamic_context}  # This must change per session
"""
```

### Issue: Security Classifier Too Restrictive

**Symptom:** Safe operations being blocked

**Solution:** Add project-specific overrides:

```yaml
# .claude/auto_mode_rules.yaml
rules:
  - pattern: "bash:npm install"
    verdict: safe
    reason: "Package installation is safe in this project"
```

### Issue: Context Window Overflow

**Symptom:** Errors about exceeding token limits

**Solution:** Implement compaction earlier:

```python
if total_tokens > MAX_TOKENS * 0.7:
    messages = compact_old_messages(messages)
```

### Issue: Agent Coordination Failures

**Symptom:** Sub-agents producing conflicting results

**Solution:** Use explicit coordination protocol:

```markdown
## Coordinator Instructions

When spawning sub-agents:
1. Assign non-overlapping file scopes
2. Provide explicit merge strategy
3. Define conflict resolution rules
```

## Resources

- **Full pattern documentation**: Browse `prompts/` directory
- **Research discussions**: Check project issues and PRs
- **Related projects**: 
  - MCP (Model Context Protocol) for tool integration
  - LangChain for agent frameworks
  - Autogen for multi-agent systems

## Best Practices

1. **Start with simple mode** - Use minimal prompts for straightforward tasks
2. **Layer complexity gradually** - Add specialized agents only when needed
3. **Test security boundaries** - Always validate auto-approval rules
4. **Monitor context usage** - Implement compaction before hitting limits
5. **Version your prompts** - Track changes to system prompts over time
6. **Validate with adversarial testing** - Use verification agents liberally
7. **Document agent behaviors** - Keep notes on what works and what doesn't

This research provides architectural patterns, not prescriptive solutions. Adapt these patterns to your specific use case and constraints.
