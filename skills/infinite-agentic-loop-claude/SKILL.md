---
name: infinite-agentic-loop-claude
description: Master the Infinite Agentic Loop pattern with Claude Code for parallel AI agent orchestration and iterative content generation
triggers:
  - how do I use the infinite agentic loop pattern
  - set up parallel AI agents with Claude Code
  - create infinite agentic loops
  - use the /project:infinite command
  - generate multiple iterations with sub-agents
  - orchestrate parallel Claude agents
  - implement infinite generation loops
  - configure Claude Code custom commands
---

# Infinite Agentic Loop Claude Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Infinite Agentic Loop is an experimental pattern that orchestrates multiple AI agents in parallel using Claude Code. It enables continuous, iterative generation of content (UI components, code, designs) where each iteration builds upon previous work while maintaining uniqueness and spec compliance.

**Key Features:**
- Parallel sub-agent deployment for simultaneous generation
- Progressive sophistication across iterations
- Wave-based management for large batches
- Custom Claude Code slash command system
- Specification-driven generation

## Installation

1. **Clone the repository:**
```bash
git clone https://github.com/disler/infinite-agentic-loop.git
cd infinite-agentic-loop
```

2. **Verify Claude Code installation:**
```bash
claude --version
```

3. **Check project configuration:**
The project includes `.claude/settings.json` and `.claude/commands/infinite.md` which define permissions and commands.

4. **Start Claude Code:**
```bash
claude
```

## Core Command: /project:infinite

The infinite agentic loop is triggered via a custom slash command:

```bash
/project:infinite <spec_file> <output_dir> <count>
```

**Parameters:**
- `spec_file`: Path to specification markdown file (e.g., `specs/invent_new_ui_v3.md`)
- `output_dir`: Directory where iterations will be saved
- `count`: Number of iterations to generate, or "infinite" for continuous mode

## Usage Patterns

### 1. Single Generation (Testing/Development)

```bash
/project:infinite specs/invent_new_ui_v3.md src 1
```

Use this when:
- Testing a new specification
- Validating the setup
- Generating a single proof-of-concept

### 2. Small Batch (5 Iterations)

```bash
/project:infinite specs/invent_new_ui_v3.md src_new 5
```

Deploys 5 parallel sub-agents simultaneously. Ideal for:
- Rapid prototyping
- Exploring design variations
- Quick iteration cycles

### 3. Large Batch (20 Iterations)

```bash
/project:infinite specs/invent_new_ui_v3.md src_new 20
```

Manages 20 iterations in coordinated batches of 5 agents. Best for:
- Production-scale generation
- Building component libraries
- Comprehensive exploration of design space

### 4. Infinite Mode

```bash
/project:infinite specs/invent_new_ui_v3.md infinite_src_new/ infinite
```

Continuous generation in waves until context limits. Use for:
- Maximum variation generation
- Long-running exploration
- Building extensive libraries

## Creating Specification Files

Specifications drive the generation. Create a markdown file in the `specs/` directory:

```markdown
# UI Component Specification

## Objective
Generate unique, creative UI components that demonstrate modern design patterns.

## Requirements
- Must be self-contained HTML file
- Include inline CSS and JavaScript
- Responsive design
- Accessibility compliant (ARIA labels)
- No external dependencies

## Constraints
- Each iteration must be visually distinct
- Maximum file size: 50KB
- Must work in all modern browsers

## Creative Directions
- Explore different color palettes
- Experiment with layout patterns
- Try various interaction models
- Incorporate subtle animations
```

## Configuration

### .claude/settings.json

```json
{
  "allow_file_operations": true,
  "allow_shell_commands": true,
  "max_tokens": 200000,
  "commands_directory": ".claude/commands"
}
```

### Custom Command Structure (.claude/commands/infinite.md)

```markdown
# Infinite Agentic Loop Command

## Command: /project:infinite

### Implementation Steps:
1. Parse arguments (spec, output_dir, count)
2. Read and analyze specification file
3. Scan output directory for existing iterations
4. Determine next iteration number
5. Deploy sub-agents in parallel
6. Monitor and coordinate generation
7. Validate uniqueness and spec compliance
```

## Working Code Examples

### Example 1: Basic Specification-Driven Generation

```html
<!-- Generated iteration: ui_component_001.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Card Component</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .card {
            background: white;
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            max-width: 400px;
            transition: transform 0.3s ease;
        }
        .card:hover { transform: translateY(-10px); }
    </style>
</head>
<body>
    <div class="card" role="article" aria-label="Interactive component">
        <h1>Iteration 001</h1>
        <p>Unique gradient-based card design</p>
    </div>
</body>
</html>
```

### Example 2: Custom Spec for API Documentation

```markdown
<!-- specs/api_docs_spec.md -->
# API Documentation Generator

## Objective
Generate comprehensive API documentation pages with interactive examples.

## Technical Requirements
- OpenAPI 3.0 schema parsing
- Syntax-highlighted code samples
- Interactive request builder
- Response visualization

## Output Format
Each iteration should produce:
- `api_docs_[number].html` - Main documentation
- Embedded JavaScript for interactivity
- No external API calls (use mock data)
```

### Example 3: Batch Processing Script

```bash
#!/bin/bash
# batch_generate.sh - Run multiple infinite loops in sequence

SPECS=(
    "specs/ui_components.md"
    "specs/data_visualizations.md"
    "specs/form_patterns.md"
)

for spec in "${SPECS[@]}"; do
    basename=$(basename "$spec" .md)
    echo "Generating from $spec..."
    claude << EOF
/project:infinite $spec output/${basename}/ 10
EOF
done
```

## Advanced Patterns

### Pattern 1: Progressive Complexity

Start with simple iterations and increase complexity:

```bash
# Wave 1: Basic components
/project:infinite specs/simple_ui.md wave1/ 5

# Wave 2: Add interactions (reference wave1 in spec)
/project:infinite specs/interactive_ui.md wave2/ 5

# Wave 3: Complex compositions
/project:infinite specs/complex_ui.md wave3/ 5
```

### Pattern 2: Variation Exploration

Generate variations on a theme:

```markdown
<!-- specs/button_variations.md -->
## Creative Constraints
Each iteration must:
1. Use a different CSS animation
2. Implement unique hover states
3. Explore different shape paradigms (rounded, sharp, organic)
4. Vary color psychology (calm, energetic, professional)
```

### Pattern 3: Cross-Pollination

Use outputs from one loop as inputs to another:

```bash
# Generate base components
/project:infinite specs/base_components.md components/ 10

# Generate pages using those components (spec references components/)
/project:infinite specs/full_pages.md pages/ 5
```

## Troubleshooting

### Issue: Command not found

**Solution:** Ensure you're in the project directory and Claude Code is running:
```bash
cd infinite-agentic-loop
claude
# Wait for prompt, then try /project:infinite
```

### Issue: Iterations are too similar

**Solution:** Enhance your specification with stronger creative constraints:
```markdown
## Uniqueness Requirements
- Each iteration must use a different primary color family
- Layout grid must vary (2-col, 3-col, masonry, etc.)
- Typography scale must be distinct
```

### Issue: Context limit reached in infinite mode

**Solution:** This is expected. Review generated iterations and restart with refined specs:
```bash
# Analyze what was generated
ls -lh infinite_src_new/

# Refine spec based on learnings
vim specs/invent_new_ui_v3.md

# Start fresh batch
/project:infinite specs/invent_new_ui_v3.md infinite_src_new_v2/ infinite
```

### Issue: Sub-agents generating errors

**Solution:** Check `.claude/settings.json` permissions and validate spec file syntax:
```bash
# Validate JSON
cat .claude/settings.json | python -m json.tool

# Check spec file exists and is readable
cat specs/your_spec.md
```

## Extending the Pattern

### Global Command Installation

Install the infinite command globally:

```bash
mkdir -p ~/.claude/commands
cp .claude/commands/infinite.md ~/.claude/commands/
```

Now use `/infinite` from any Claude Code session.

### Multi-File Generation

Modify `.claude/commands/infinite.md` to generate sets of files:

```markdown
## Enhanced Output Structure
For each iteration N, generate:
- component_N.html (main file)
- component_N.test.html (test file)
- component_N.meta.json (metadata)
```

### MCP Server Integration

Create an MCP (Model Context Protocol) server for reusability:

```javascript
// mcp-server.js
const { MCPServer } = require('@modelcontextprotocol/sdk');

const server = new MCPServer({
  name: 'infinite-agentic-loop',
  version: '1.0.0'
});

server.addTool({
  name: 'infinite_generate',
  description: 'Generate iterations using infinite agentic loop',
  parameters: {
    spec: { type: 'string', description: 'Specification file path' },
    output_dir: { type: 'string', description: 'Output directory' },
    count: { type: 'number', description: 'Number of iterations' }
  },
  execute: async ({ spec, output_dir, count }) => {
    // Implementation here
  }
});

server.start();
```

## Best Practices

1. **Start Small:** Test with count=1 before large batches
2. **Iterate on Specs:** Refine specifications based on output quality
3. **Monitor Resources:** Watch memory/CPU during large batches
4. **Version Control:** Commit specs and generated outputs separately
5. **Document Learnings:** Track what specification patterns work best

## Resources

- [Tutorial Video](https://youtu.be/9ipM_vDwflI)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code/overview)
- [IndyDevDan YouTube Channel](https://www.youtube.com/@indydevdan)
- [Agentic Engineer Course](https://agenticengineer.com/principled-ai-coding)
