---
name: hermes-agent-framework
description: Expert guide for Nous Research's Hermes Agent framework with self-improving learning loops, three-layer memory, and automatic Skill creation
triggers:
  - "help me set up Hermes Agent"
  - "how do I use Hermes Agent framework"
  - "configure Hermes Agent memory system"
  - "create custom Skills for Hermes"
  - "build an AI agent with Hermes"
  - "Hermes Agent learning loop and tools"
  - "integrate Hermes Agent into my project"
  - "troubleshoot Hermes Agent issues"
---

# Hermes Agent Framework

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Expert knowledge for working with [Hermes Agent](https://github.com/NousResearch/hermes-agent), the open-source AI Agent framework by Nous Research featuring built-in self-improving learning loops, three-layer memory system (episodic, semantic, procedural), and automatic Skill creation and evolution.

## What is Hermes Agent?

Hermes Agent is a production-ready AI Agent framework released in February 2026 that differs from traditional agents (like OpenClaw/Claude Code) by implementing:

- **Self-improving learning loop**: Automatically learns from interactions and improves over time
- **Three-layer memory system**: Episodic (conversations), semantic (knowledge), procedural (Skills)
- **Automatic Skill creation**: Generates and evolves reusable capabilities
- **Built-in tool ecosystem**: Extensible plugin architecture for custom tools

Core philosophy: Agents should learn and improve themselves, not just execute tasks.

## Installation

### Prerequisites

- Python 3.10 or higher
- OpenAI API key (or compatible provider like Anthropic, local models)
- Git

### Quick Start

```bash
# Clone the repository
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent

# Install dependencies
pip install -r requirements.txt

# Or use poetry
poetry install

# Set up environment variables
cp .env.example .env
# Edit .env with your API keys
```

### Environment Configuration

Create a `.env` file:

```bash
# Required: Your LLM provider API key
OPENAI_API_KEY=your_key_here
# Or for Anthropic
ANTHROPIC_API_KEY=your_key_here

# Optional: Model selection
HERMES_MODEL=gpt-4-turbo
# or
HERMES_MODEL=claude-3-5-sonnet-20241022

# Memory configuration
HERMES_MEMORY_PATH=~/.hermes/memory
HERMES_ENABLE_SEMANTIC_MEMORY=true

# Skill storage
HERMES_SKILL_PATH=~/.hermes/skills
```

## Core Architecture

### Three-Layer Memory System

```python
from hermes_agent import HermesAgent, MemoryConfig

# Configure memory layers
memory_config = MemoryConfig(
    episodic_enabled=True,      # Conversation history
    semantic_enabled=True,       # Knowledge base
    procedural_enabled=True,     # Skills/procedures
    memory_path="~/.hermes/memory"
)

agent = HermesAgent(
    model="gpt-4-turbo",
    memory_config=memory_config
)
```

### Self-Improving Learning Loop

```python
from hermes_agent import HermesAgent, LearningConfig

learning_config = LearningConfig(
    enable_auto_learning=True,
    reflection_interval=5,       # Reflect every 5 interactions
    skill_creation_threshold=3,  # Create skill after 3 similar tasks
    feedback_sensitivity=0.7
)

agent = HermesAgent(
    model="gpt-4-turbo",
    learning_config=learning_config
)

# The agent will automatically:
# 1. Detect patterns in user requests
# 2. Create Skills for repeated tasks
# 3. Improve existing Skills based on feedback
# 4. Store knowledge in semantic memory
```

## Basic Usage

### Simple Conversation

```python
from hermes_agent import HermesAgent

# Initialize agent
agent = HermesAgent(
    model="gpt-4-turbo",
    api_key=os.getenv("OPENAI_API_KEY")
)

# Single interaction
response = agent.chat("Help me analyze this Python code for bugs")
print(response)

# Conversational context is maintained
response = agent.chat("Now optimize it for performance")
print(response)
```

### Using the CLI

```bash
# Start interactive mode
python -m hermes_agent

# Or use the CLI directly
hermes chat "What's the weather today?"

# Load specific Skill
hermes chat --skill code-reviewer "Review my Python script"

# Show agent's memory
hermes memory list

# Show available Skills
hermes skills list

# Export learned knowledge
hermes memory export --format json --output knowledge.json
```

## Skill System

### Creating Custom Skills

Skills are reusable procedures stored in the agent's procedural memory.

```python
from hermes_agent import Skill, SkillParameter

# Define a custom Skill
code_review_skill = Skill(
    name="code_reviewer",
    description="Reviews code for bugs, style, and best practices",
    parameters=[
        SkillParameter(
            name="code",
            type="string",
            description="The code to review",
            required=True
        ),
        SkillParameter(
            name="language",
            type="string",
            description="Programming language",
            required=False,
            default="python"
        )
    ],
    instructions="""
    1. Analyze the code for syntax errors
    2. Check for common bugs and anti-patterns
    3. Review style and formatting
    4. Suggest performance improvements
    5. Provide specific line-by-line feedback
    """,
    examples=[
        {
            "input": {"code": "def foo():\n  x=1\n  return x", "language": "python"},
            "output": "Style: Use spaces around operators. Function name could be more descriptive."
        }
    ]
)

# Register Skill with agent
agent.register_skill(code_review_skill)

# Use the Skill
result = agent.execute_skill("code_reviewer", {
    "code": "def calculate(a,b):\n  return a+b",
    "language": "python"
})
print(result)
```

### Skill YAML Definition

Skills can also be defined in YAML files:

```yaml
# ~/.hermes/skills/code-reviewer.yaml
name: code_reviewer
description: Reviews code for bugs, style, and best practices
version: 1.0.0

parameters:
  - name: code
    type: string
    required: true
    description: The code to review
  - name: language
    type: string
    required: false
    default: python
    description: Programming language

instructions: |
  1. Analyze the code for syntax errors
  2. Check for common bugs and anti-patterns
  3. Review style and formatting
  4. Suggest performance improvements
  5. Provide specific line-by-line feedback

examples:
  - input:
      code: "def foo():\n  x=1\n  return x"
      language: python
    output: "Style: Use spaces around operators. Function name could be more descriptive."

metadata:
  author: HuaShu
  tags: [code, review, python]
  auto_improve: true
```

Load from YAML:

```python
agent.load_skill_from_file("~/.hermes/skills/code-reviewer.yaml")
```

## Tool Integration

### Built-in Tools

```python
from hermes_agent import HermesAgent
from hermes_agent.tools import (
    WebSearchTool,
    CodeExecutorTool,
    FileSystemTool,
    APICallerTool
)

agent = HermesAgent(model="gpt-4-turbo")

# Enable built-in tools
agent.enable_tool(WebSearchTool())
agent.enable_tool(CodeExecutorTool(
    allowed_languages=["python", "javascript"],
    timeout=30
))
agent.enable_tool(FileSystemTool(
    allowed_paths=["/home/user/projects"],
    read_only=False
))

# Agent can now use these tools automatically
response = agent.chat("Search for Python best practices and create a summary file")
```

### Creating Custom Tools

```python
from hermes_agent import Tool, ToolParameter

class DatabaseQueryTool(Tool):
    name = "database_query"
    description = "Executes SQL queries against the database"
    
    parameters = [
        ToolParameter(
            name="query",
            type="string",
            description="SQL query to execute",
            required=True
        ),
        ToolParameter(
            name="database",
            type="string",
            description="Database name",
            required=False,
            default="main"
        )
    ]
    
    def execute(self, query: str, database: str = "main"):
        # Your database logic here
        import sqlite3
        conn = sqlite3.connect(f"{database}.db")
        cursor = conn.execute(query)
        results = cursor.fetchall()
        conn.close()
        return results

# Register custom tool
agent.enable_tool(DatabaseQueryTool())

# Agent can now use it
response = agent.chat("Query the users table for active users")
```

## Memory Management

### Accessing Memory Layers

```python
from hermes_agent import HermesAgent

agent = HermesAgent(model="gpt-4-turbo")

# Episodic memory (conversation history)
conversation_history = agent.memory.episodic.get_recent(limit=10)
for entry in conversation_history:
    print(f"User: {entry.user_message}")
    print(f"Agent: {entry.agent_response}")

# Semantic memory (knowledge base)
knowledge = agent.memory.semantic.search("Python best practices")
for item in knowledge:
    print(f"Topic: {item.topic}")
    print(f"Content: {item.content}")
    print(f"Source: {item.source}")

# Procedural memory (Skills)
skills = agent.memory.procedural.list_skills()
for skill in skills:
    print(f"Skill: {skill.name} - {skill.description}")
    print(f"Used {skill.usage_count} times")
```

### Manual Memory Operations

```python
# Add to semantic memory manually
agent.memory.semantic.add(
    topic="Python Type Hints",
    content="Type hints improve code readability and enable static analysis...",
    source="user_input",
    tags=["python", "typing"]
)

# Clear episodic memory (conversation history)
agent.memory.episodic.clear()

# Export all memory
agent.memory.export_all(output_path="./memory_backup")

# Import memory from backup
agent.memory.import_all(input_path="./memory_backup")
```

## Advanced Patterns

### Multi-Agent Collaboration

```python
from hermes_agent import HermesAgent, AgentOrchestrator

# Create specialized agents
code_agent = HermesAgent(
    name="CodeExpert",
    model="gpt-4-turbo",
    system_prompt="You are a code expert specializing in Python and JavaScript"
)

review_agent = HermesAgent(
    name="CodeReviewer",
    model="gpt-4-turbo",
    system_prompt="You are a code reviewer focusing on quality and best practices"
)

doc_agent = HermesAgent(
    name="DocWriter",
    model="gpt-4-turbo",
    system_prompt="You write clear, comprehensive documentation"
)

# Orchestrate agents
orchestrator = AgentOrchestrator(
    agents=[code_agent, review_agent, doc_agent],
    coordination_strategy="sequential"  # or "parallel", "hierarchical"
)

# Execute workflow
result = orchestrator.execute_workflow(
    task="Create a Python function to parse CSV files, review it, and write docs",
    workflow=[
        {"agent": "CodeExpert", "task": "Write the function"},
        {"agent": "CodeReviewer", "task": "Review the code"},
        {"agent": "CodeExpert", "task": "Apply review feedback"},
        {"agent": "DocWriter", "task": "Write documentation"}
    ]
)

print(result.final_output)
```

### Feedback Loop for Improvement

```python
from hermes_agent import HermesAgent, Feedback

agent = HermesAgent(model="gpt-4-turbo")

# Execute task
response = agent.chat("Create a REST API client for GitHub")

# Provide feedback
feedback = Feedback(
    interaction_id=response.interaction_id,
    rating=4,  # 1-5 scale
    comments="Good structure but missing error handling",
    corrections={
        "missing": ["try-except blocks", "timeout configuration"],
        "suggestions": ["Add retry logic", "Use requests session"]
    }
)

agent.provide_feedback(feedback)

# Agent will learn and improve future responses
# Next similar task will incorporate this feedback
```

### Custom Learning Rules

```python
from hermes_agent import HermesAgent, LearningRule

# Define custom learning rule
class CodeQualityRule(LearningRule):
    def should_trigger(self, interaction):
        return "code" in interaction.user_message.lower()
    
    def extract_knowledge(self, interaction, feedback):
        if feedback.rating >= 4:
            return {
                "topic": "code_patterns",
                "content": interaction.agent_response,
                "tags": ["approved", "high_quality"]
            }
        return None
    
    def should_create_skill(self, pattern_count):
        # Create skill after 2 successful code tasks
        return pattern_count >= 2

agent = HermesAgent(model="gpt-4-turbo")
agent.add_learning_rule(CodeQualityRule())
```

## Configuration

### Full Configuration Example

```python
from hermes_agent import (
    HermesAgent,
    MemoryConfig,
    LearningConfig,
    ModelConfig,
    ToolConfig
)

# Model configuration
model_config = ModelConfig(
    provider="openai",
    model="gpt-4-turbo",
    temperature=0.7,
    max_tokens=4000,
    api_key=os.getenv("OPENAI_API_KEY")
)

# Memory configuration
memory_config = MemoryConfig(
    episodic_enabled=True,
    episodic_max_size=1000,
    semantic_enabled=True,
    semantic_embedding_model="text-embedding-3-small",
    procedural_enabled=True,
    memory_path="~/.hermes/memory",
    auto_save=True,
    save_interval=60  # seconds
)

# Learning configuration
learning_config = LearningConfig(
    enable_auto_learning=True,
    reflection_interval=5,
    skill_creation_threshold=3,
    feedback_sensitivity=0.7,
    auto_improve_skills=True,
    learning_rate=0.1
)

# Tool configuration
tool_config = ToolConfig(
    enabled_tools=["web_search", "code_executor", "file_system"],
    tool_timeout=30,
    allow_dangerous_tools=False
)

# Create agent with full configuration
agent = HermesAgent(
    name="MyAssistant",
    model_config=model_config,
    memory_config=memory_config,
    learning_config=learning_config,
    tool_config=tool_config,
    system_prompt="You are a helpful AI assistant that learns and improves over time."
)
```

### Configuration File

Create `hermes_config.yaml`:

```yaml
agent:
  name: MyAssistant
  system_prompt: "You are a helpful AI assistant that learns and improves over time."

model:
  provider: openai
  model: gpt-4-turbo
  temperature: 0.7
  max_tokens: 4000
  api_key_env: OPENAI_API_KEY

memory:
  episodic:
    enabled: true
    max_size: 1000
  semantic:
    enabled: true
    embedding_model: text-embedding-3-small
  procedural:
    enabled: true
  path: ~/.hermes/memory
  auto_save: true
  save_interval: 60

learning:
  auto_learning: true
  reflection_interval: 5
  skill_creation_threshold: 3
  feedback_sensitivity: 0.7
  auto_improve_skills: true
  learning_rate: 0.1

tools:
  enabled:
    - web_search
    - code_executor
    - file_system
  timeout: 30
  allow_dangerous: false
```

Load configuration:

```python
from hermes_agent import HermesAgent

agent = HermesAgent.from_config_file("hermes_config.yaml")
```

## Real-World Examples

### Knowledge Assistant

```python
from hermes_agent import HermesAgent
from hermes_agent.tools import WebSearchTool, FileSystemTool

# Create knowledge assistant
assistant = HermesAgent(
    name="KnowledgeAssistant",
    model="gpt-4-turbo",
    system_prompt="""You are a knowledge assistant that:
    1. Researches topics using web search
    2. Stores learned knowledge in semantic memory
    3. Creates summaries and documentation
    4. Answers questions based on accumulated knowledge
    """
)

assistant.enable_tool(WebSearchTool())
assistant.enable_tool(FileSystemTool(allowed_paths=["./knowledge"]))

# Research and learn
response = assistant.chat(
    "Research Python async/await patterns and create a summary document"
)

# Later, retrieve knowledge
response = assistant.chat("What are the best practices for async Python?")
# Agent uses semantic memory to answer without re-searching
```

### Development Automation

```python
from hermes_agent import HermesAgent
from hermes_agent.tools import CodeExecutorTool, FileSystemTool, GitTool

dev_agent = HermesAgent(
    name="DevAutomation",
    model="gpt-4-turbo"
)

dev_agent.enable_tool(CodeExecutorTool(allowed_languages=["python"]))
dev_agent.enable_tool(FileSystemTool(allowed_paths=["./project"]))
dev_agent.enable_tool(GitTool())

# Automated development workflow
workflow_prompt = """
1. Create a Python FastAPI application with user authentication
2. Write unit tests with pytest
3. Run the tests
4. Fix any failures
5. Commit the working code
6. Generate API documentation
"""

result = dev_agent.chat(workflow_prompt)

# Agent will:
# - Create files
# - Write code
# - Execute tests
# - Iterate on failures
# - Use git for version control
# - Generate docs
```

### Content Creation Pipeline

```python
from hermes_agent import HermesAgent, AgentOrchestrator

# Research agent
researcher = HermesAgent(
    name="Researcher",
    model="gpt-4-turbo",
    system_prompt="Research topics thoroughly and gather facts"
)

# Writer agent
writer = HermesAgent(
    name="Writer",
    model="gpt-4-turbo",
    system_prompt="Write engaging, well-structured content"
)

# Editor agent
editor = HermesAgent(
    name="Editor",
    model="gpt-4-turbo",
    system_prompt="Edit for clarity, grammar, and style"
)

orchestrator = AgentOrchestrator(
    agents=[researcher, writer, editor],
    coordination_strategy="sequential"
)

# Content pipeline
result = orchestrator.execute_workflow(
    task="Create a blog post about AI Agents",
    workflow=[
        {"agent": "Researcher", "task": "Research AI Agents, find recent developments"},
        {"agent": "Writer", "task": "Write 1000-word blog post using research"},
        {"agent": "Editor", "task": "Edit for publication"},
        {"agent": "Writer", "task": "Apply edits and finalize"}
    ]
)

print(result.final_output)
```

## Troubleshooting

### Common Issues

**Agent not learning from interactions**

```python
# Check learning config
agent = HermesAgent(
    model="gpt-4-turbo",
    learning_config=LearningConfig(
        enable_auto_learning=True,  # Must be True
        reflection_interval=5
    )
)

# Verify memory is enabled
print(agent.memory.config.procedural_enabled)  # Should be True

# Check logs
import logging
logging.basicConfig(level=logging.DEBUG)
```

**Skills not being created automatically**

```python
# Lower the threshold
agent.learning_config.skill_creation_threshold = 2

# Manually trigger skill creation
agent.create_skill_from_pattern(
    pattern_name="code_review",
    interactions=[id1, id2, id3]
)
```

**Memory not persisting between sessions**

```python
# Ensure auto_save is enabled
agent.memory.config.auto_save = True
agent.memory.config.save_interval = 60

# Or manually save
agent.memory.save_all()

# Check memory path exists
import os
print(os.path.exists(agent.memory.config.memory_path))
```

**API rate limits**

```python
# Add retry logic
from hermes_agent import RetryConfig

agent = HermesAgent(
    model="gpt-4-turbo",
    retry_config=RetryConfig(
        max_retries=3,
        backoff_factor=2,
        respect_rate_limits=True
    )
)
```

**Tool execution timeouts**

```python
# Increase timeout
from hermes_agent.tools import CodeExecutorTool

agent.enable_tool(CodeExecutorTool(
    timeout=60,  # seconds
    max_output_length=10000
))
```

### Debug Mode

```python
from hermes_agent import HermesAgent

agent = HermesAgent(
    model="gpt-4-turbo",
    debug=True  # Enables verbose logging
)

# Access internal state
print(agent.debug_info())
print(agent.memory.stats())
print(agent.learning.stats())
```

### Logging

```python
import logging

# Configure Hermes logging
logging.getLogger("hermes_agent").setLevel(logging.DEBUG)

# Log to file
file_handler = logging.FileHandler("hermes.log")
file_handler.setLevel(logging.DEBUG)
logging.getLogger("hermes_agent").addHandler(file_handler)
```

## Best Practices

1. **Start with conservative learning settings** - Begin with higher thresholds and adjust based on results
2. **Provide feedback regularly** - The learning loop improves with human feedback
3. **Review auto-generated Skills** - Inspect and refine Skills before heavy use
4. **Use semantic memory strategically** - Add important knowledge manually for faster retrieval
5. **Monitor token usage** - Learning loops can increase API calls
6. **Version control your Skills** - Keep Skill definitions in git
7. **Separate agents by role** - Use specialized agents with orchestration for complex workflows
8. **Test Skills in isolation** - Validate Skill behavior before relying on them
9. **Regular memory maintenance** - Periodically review and clean semantic memory
10. **Environment-specific configs** - Use different configs for dev/prod environments

## Resources

- Official Documentation: https://hermes-agent.nousresearch.com/docs/
- GitHub Repository: https://github.com/NousResearch/hermes-agent
- Community Discord: https://discord.gg/nousresearch
- Example Skills: https://github.com/NousResearch/hermes-agent/tree/main/examples/skills
- Orange Book Guide: [Download PDF](https://github.com/alchaincyf/hermes-agent-orange-book)
