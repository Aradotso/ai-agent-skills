---
name: agent-apprenticeship-ecosystem
description: Use Agent Apprenticeship to train AI agents through real-world tasks, reusable experience, and ecosystem learning signals
triggers:
  - set up agent apprenticeship for learning from real tasks
  - run an agent workflow with apprenticeship loops
  - contribute agent experience to the ecosystem
  - search for agent training signals and reusable experience
  - create experience packs from ecosystem learning
  - configure mentor modes for agent training
  - share agent execution traces with the community
  - pull ecosystem experience for agent improvement
---

# Agent Apprenticeship Ecosystem Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

Agent Apprenticeship creates a living ecosystem where AI agents learn from real-world work through iterative workflow loops, reusable experience, and collective training signal exchange. It enables agents to execute long-horizon tasks while generating training signals that improve the entire ecosystem.

The system supports:
- **Iterative workflow loops** with mentor guidance (model-assisted, expert-led, or hybrid)
- **Reusable learning signals** from 500+ seed tasks and 1000+ execution traces
- **Ecosystem contribution** of agent experience packages
- **Experience Packs** that transfer learning across tasks
- **Economic value tracking** for agent task execution

## Installation

```bash
# Quick start with npx
npx agent-apprenticeship init

# Or install globally
npm install -g agent-apprenticeship

# Verify installation
apprentice --version
apprentice doctor
```

The CLI provides both short (`apprentice`) and long (`agent-apprenticeship`) commands.

## Initial Setup

```bash
# Initialize Agent Apprenticeship
npx agent-apprenticeship init

# Check configuration
apprentice settings
apprentice doctor

# Configure your apprentice agent
apprentice configure

# Configure model provider
apprentice configure model
```

### Environment Configuration

Store API keys in `~/.agent-apprenticeship/.env.local`:

```bash
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=...
OPENROUTER_API_KEY=sk-or-...
```

Or use shell environment variables:

```bash
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export AA_MAX_ITERATIONS=3
```

## Apprentice Agents

Agent Apprenticeship auto-detects installed agent CLIs:

- **Codex**
- **Cursor**
- **Claude Code**
- **OpenClaw**
- **OpenCode**
- **Hermes Agent**
- **Custom** (with command templates)

### Custom Agent Configuration

```bash
apprentice configure agent custom --command-template "my-agent run --workspace {workspace} --prompt-file {prompt_file}"
```

## Running Tasks

### Basic Task Execution

```bash
# Run a simple task
apprentice run "Create a short market map for AI procurement tools."

# Run with specific mentor mode
apprentice run "Build a release checklist for an AI agent project." --mentor-mode model-assisted

# Run with maximum iterations
export AA_MAX_ITERATIONS=5
apprentice run "Design a multi-step deployment pipeline."
```

### Mentor Modes

```bash
# Model-assisted: automated mentor loop
apprentice run "..." --mentor-mode model-assisted

# Expert-led: human checkpoint guidance
apprentice run "..." --mentor-mode expert-led

# Hybrid: model drafts + human approval
apprentice run "..." --mentor-mode hybrid
```

**Mentor Mode Details:**
- `model-assisted`: Mentor Model Provider handles the entire loop automatically
- `expert-led`: Human expert provides checkpoints at each iteration
- `hybrid`: Model provides drafts, human reviews and approves/edits

## Working with Bundles

After a run completes, Agent Apprenticeship generates a contribution bundle containing:
- Task definition and execution trace
- Agent work episodes and rollouts
- Learning signals and lessons
- Artifacts and outputs

### Bundle Inspection

```bash
# Inspect bundle contents
apprentice bundle inspect ./runs/2026-06-22_143022/bundle.zip

# Validate bundle structure
apprentice bundle check ./runs/2026-06-22_143022/bundle.zip

# Contribute bundle to ecosystem
apprentice bundle contribute ./runs/2026-06-22_143022/bundle.zip
```

## Ecosystem Integration

### Ecosystem Configuration

```bash
# Configure ecosystem repository
apprentice ecosystem configure --repo Forsy-AI/agent-apprenticeship

# Set auto-share mode
apprentice ecosystem configure --auto-share manual     # No automatic sharing
apprentice ecosystem configure --auto-share ask        # Ask before sharing
apprentice ecosystem configure --auto-share automatic  # Share automatically

# Check ecosystem status
apprentice ecosystem status
```

**Requirements for ecosystem sharing:**
- GitHub CLI (`gh`) installed and authenticated
- Ecosystem repository configured
- Valid bundle format

### Searching and Exploring

```bash
# List all ecosystem experience
apprentice ecosystem list

# Search for specific topics
apprentice ecosystem search cloud
apprentice ecosystem search "deployment pipeline"
apprentice ecosystem search kubernetes

# Inspect specific experience
apprentice ecosystem inspect aa-seed-task-501

# Pull experience locally
apprentice ecosystem pull aa-seed-task-501
```

### Contributing Experience

```bash
# Contribute a bundle to the ecosystem
apprentice ecosystem contribute ./runs/2026-06-22_143022/bundle.zip

# Automatic contribution (when auto-share is enabled)
apprentice run "..." # Bundle automatically shared if configured
```

## Experience Packs

Experience Packs transform ecosystem experience into reusable learning signals for future tasks.

### Creating Experience Packs

```bash
# Create pack from ecosystem experience
apprentice learn create aa-seed-task-501

# Preview pack contents
apprentice learn preview pack_12345

# Replay pack execution
apprentice learn replay pack_12345

# Keep pack for future use
apprentice learn keep pack_12345

# Revert pack (remove from active set)
apprentice learn revert pack_12345
```

### Using Experience Packs

```bash
# Run task with specific experience pack
apprentice run "Create incident response checklist." --experience-pack pack_12345

# Use all active experience packs
apprentice run "Deploy microservice architecture." --use-active-experience-packs

# Disable experience packs for a run
apprentice run "Prototype new feature." --no-experience-packs
```

## Seed Dataset

The Agent Apprenticeship seed dataset includes:
- 500+ curated real-world tasks
- 495 reusable agent lessons
- 1000+ full agent execution traces
- 1000+ agent work episodes

Access the seed dataset:

```bash
# Seed dataset is included in the repository
ls seed_dataset/

# Search seed tasks
apprentice ecosystem search --filter seed

# Inspect seed task
apprentice ecosystem inspect aa-seed-task-001
```

## Configuration Management

### View Current Settings

```bash
# Show all settings
apprentice settings

# Show ecosystem configuration
apprentice ecosystem status

# Verify environment and setup
apprentice doctor
```

### Update Configuration

```bash
# Reconfigure agent
apprentice configure

# Change model provider
apprentice configure model

# Update ecosystem settings
apprentice ecosystem configure --repo your-org/your-repo
apprentice ecosystem configure --auto-share ask
```

## Common Workflows

### Workflow 1: Simple Task Execution

```bash
# 1. Run a task
apprentice run "Create API documentation for user authentication."

# 2. Inspect the generated bundle
apprentice bundle inspect ./runs/2026-06-22_150033/bundle.zip

# 3. Contribute to ecosystem (optional)
apprentice ecosystem contribute ./runs/2026-06-22_150033/bundle.zip
```

### Workflow 2: Learning from Ecosystem

```bash
# 1. Search for relevant experience
apprentice ecosystem search "API documentation"

# 2. Inspect interesting result
apprentice ecosystem inspect aa-seed-task-215

# 3. Pull experience locally
apprentice ecosystem pull aa-seed-task-215

# 4. Create experience pack
apprentice learn create aa-seed-task-215

# 5. Use pack in new task
apprentice run "Document GraphQL API endpoints." --experience-pack pack_67890
```

### Workflow 3: Iterative Complex Task

```bash
# 1. Set iteration limit
export AA_MAX_ITERATIONS=10

# 2. Run complex task with hybrid mentor mode
apprentice run "Design and implement a CI/CD pipeline with security scanning." --mentor-mode hybrid

# 3. Review execution trace
apprentice bundle inspect ./runs/2026-06-22_153044/bundle.zip

# 4. Create experience pack for future similar tasks
apprentice learn create ./runs/2026-06-22_153044/bundle.zip
apprentice learn keep pack_11223
```

### Workflow 4: Domain-Specific Agent Training

```bash
# 1. Search for domain-specific tasks
apprentice ecosystem search kubernetes

# 2. Pull multiple related experiences
apprentice ecosystem pull aa-seed-task-301
apprentice ecosystem pull aa-seed-task-302
apprentice ecosystem pull aa-seed-task-303

# 3. Create experience packs
apprentice learn create aa-seed-task-301
apprentice learn create aa-seed-task-302
apprentice learn create aa-seed-task-303

# 4. Keep all packs
apprentice learn keep pack_301
apprentice learn keep pack_302
apprentice learn keep pack_303

# 5. Run new domain task with accumulated experience
apprentice run "Deploy multi-region Kubernetes cluster with observability." --use-active-experience-packs
```

## Repository Structure

When contributing to or exploring the ecosystem, the public repository follows this structure:

```
seed_dataset/              # Initial 500+ curated tasks
ecosystem/                 # Community experience
  contributions/           # Contributed bundles
schemas/                   # Bundle and trace schemas
examples/                  # Example usage and integrations
```

## Advanced Configuration

### Max Iterations

Control the depth of iterative workflow loops:

```bash
# Via settings (persistent)
apprentice settings  # Then update max_iterations

# Via environment variable (session)
export AA_MAX_ITERATIONS=7
apprentice run "..."

# Via command flag (per-run, if supported)
apprentice run "..." --max-iterations 7
```

### Custom Mentor Models

When configuring model providers, you can specify custom models:

```bash
apprentice configure model

# Then select provider and specify model:
# - OpenAI: gpt-4, gpt-4-turbo, etc.
# - Anthropic: claude-3-opus-20240229, claude-3-sonnet-20240229
# - Gemini: gemini-pro, gemini-ultra
# - OpenRouter: various models
```

### Workspace Management

Agent Apprenticeship creates isolated workspaces for each run:

```bash
# Default workspace location
~/.agent-apprenticeship/runs/

# Each run creates a timestamped folder
~/.agent-apprenticeship/runs/2026-06-22_143022/
  workspace/        # Agent execution workspace
  artifacts/        # Generated outputs
  bundle.zip        # Contribution bundle
  trace.json        # Execution trace
```

## Troubleshooting

### Agent Not Detected

```bash
# Check which agents are installed
which codex
which cursor
which claude-code

# Reconfigure agent
apprentice configure

# For custom agents, verify command template
apprentice configure agent custom --command-template "..."
```

### API Key Issues

```bash
# Verify keys are set
apprentice doctor

# Check environment file
cat ~/.agent-apprenticeship/.env.local

# Test with environment variable
export OPENAI_API_KEY="sk-..."
apprentice doctor

# Reconfigure model provider
apprentice configure model
```

### Bundle Validation Failures

```bash
# Check bundle structure
apprentice bundle check ./runs/2026-06-22_143022/bundle.zip

# Inspect bundle contents
apprentice bundle inspect ./runs/2026-06-22_143022/bundle.zip

# Verify bundle meets schema requirements
# - Task definition present
# - Execution trace valid
# - Artifacts properly packaged
```

### Ecosystem Connection Issues

```bash
# Verify GitHub CLI authentication
gh auth status

# Re-authenticate if needed
gh auth login

# Check ecosystem configuration
apprentice ecosystem status

# Reconfigure ecosystem repo
apprentice ecosystem configure --repo Forsy-AI/agent-apprenticeship
```

### Experience Pack Issues

```bash
# List all experience packs
apprentice learn list

# Verify pack contents
apprentice learn preview pack_12345

# Revert problematic pack
apprentice learn revert pack_12345

# Clear all packs and start fresh
apprentice learn clear
```

## Integration Examples

### CI/CD Integration

```bash
#!/bin/bash
# Example: Run agent task in CI pipeline

export OPENAI_API_KEY="${OPENAI_API_KEY}"
export AA_MAX_ITERATIONS=3

# Run task
apprentice run "Generate deployment checklist for $SERVICE_NAME" \
  --mentor-mode model-assisted \
  --no-experience-packs

# Contribute if successful
if [ $? -eq 0 ]; then
  apprentice ecosystem contribute ./runs/latest/bundle.zip
fi
```

### Python Script Integration

```python
import subprocess
import os
import json

def run_agent_task(task_description, experience_packs=None):
    """Run an agent apprenticeship task from Python."""
    
    cmd = ["apprentice", "run", task_description]
    
    if experience_packs:
        for pack in experience_packs:
            cmd.extend(["--experience-pack", pack])
    
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        env={**os.environ, "AA_MAX_ITERATIONS": "5"}
    )
    
    return result.returncode == 0, result.stdout

# Example usage
success, output = run_agent_task(
    "Create API documentation for user service",
    experience_packs=["pack_12345"]
)

if success:
    print("Task completed successfully")
    print(output)
```

### Automated Learning Pipeline

```bash
#!/bin/bash
# Example: Automated ecosystem learning pipeline

# 1. Search for relevant tasks
TASKS=$(apprentice ecosystem search "API design" --json | jq -r '.[].id')

# 2. Pull and create experience packs
for task_id in $TASKS; do
  apprentice ecosystem pull "$task_id"
  apprentice learn create "$task_id"
done

# 3. Run new task with accumulated experience
apprentice run "Design REST API for analytics platform" \
  --use-active-experience-packs \
  --mentor-mode hybrid

# 4. Contribute result
apprentice ecosystem contribute ./runs/latest/bundle.zip
```

## Best Practices

1. **Start with seed dataset**: Explore `aa-seed-task-*` tasks to understand ecosystem patterns
2. **Use appropriate mentor mode**: `model-assisted` for automation, `expert-led` for high-value tasks, `hybrid` for balance
3. **Create experience packs strategically**: Focus on reusable patterns, not one-off tasks
4. **Contribute quality bundles**: Ensure tasks complete successfully before contributing
5. **Search before creating**: Check ecosystem for similar tasks to avoid duplication
6. **Iterate gradually**: Start with low `AA_MAX_ITERATIONS`, increase for complex tasks
7. **Review traces**: Use `bundle inspect` to understand agent learning patterns
8. **Manage active packs**: Keep only relevant experience packs active for better performance
