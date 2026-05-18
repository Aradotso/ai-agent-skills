---
name: agentic-data-scientist
description: Adaptive multi-agent framework for automated data science tasks with planning, execution, and validation
triggers:
  - automate data science analysis
  - use agentic data scientist
  - create multi-agent data workflow
  - analyze dataset with AI agents
  - perform automated ML analysis
  - set up agentic data science pipeline
  - orchestrate data science agents
  - run autonomous data analysis
---

# Agentic Data Scientist

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agentic Data Scientist is an adaptive multi-agent framework that automates complex data science tasks using a sophisticated workflow with planning, execution, validation, and self-correction. Built on Google's Agent Development Kit (ADK) and Claude Agent SDK, it separates planning from execution and continuously validates work against success criteria.

## What It Does

- **Orchestrated Mode**: Full multi-agent workflow with planning, iterative execution, validation, and adaptive replanning
- **Simple Mode**: Direct coding without planning overhead for quick tasks
- **Multi-Agent Architecture**: Specialized agents for planning, coding, reviewing, validation, and summarization
- **Continuous Validation**: Tracks progress against success criteria at every stage
- **Self-Correcting**: Adapts plans based on discoveries during execution
- **MCP Integration**: Access to tools via Model Context Protocol servers
- **Claude Scientific Skills**: 380+ advanced scientific computing skills available to coding agent

## Installation

```bash
# Install globally with uv
uv tool install agentic-data-scientist

# Or use directly with uvx (no installation)
uvx agentic-data-scientist --mode simple "your query"
```

### Prerequisites

**Required:**

1. **Claude Code CLI** (for coding agent):
```bash
npm install -g @anthropic-ai/claude-code
```

2. **API Keys** (set as environment variables):
```bash
export OPENROUTER_API_KEY="your_openrouter_key"  # For planning/review agents
export ANTHROPIC_API_KEY="your_anthropic_key"    # For coding agent
```

Get keys from:
- OpenRouter: https://openrouter.ai/keys
- Anthropic: https://console.anthropic.com/

**Optional:**
```bash
# Disable network access (web search, URL fetching)
export DISABLE_NETWORK_ACCESS=true
```

## Configuration

Create a `.env` file in your project directory:

```bash
# Required
OPENROUTER_API_KEY=your_openrouter_key
ANTHROPIC_API_KEY=your_anthropic_key

# Optional
DISABLE_NETWORK_ACCESS=false  # Set to true to disable web tools
```

## Key Commands

### Basic Usage

**You must specify `--mode` for every command:**

```bash
# Orchestrated mode: Full multi-agent workflow
agentic-data-scientist "Perform differential expression analysis" \
  --mode orchestrated \
  --files data.csv

# Simple mode: Direct coding, no planning
agentic-data-scientist "Write a CSV parser" \
  --mode simple
```

### File Handling

```bash
# Single file
agentic-data-scientist "Analyze dataset" \
  --mode orchestrated \
  --files data.csv

# Multiple files
agentic-data-scientist "Compare datasets" \
  --mode orchestrated \
  -f data1.csv -f data2.csv -f metadata.json

# Directory upload (recursive)
agentic-data-scientist "Analyze all CSVs in folder" \
  --mode orchestrated \
  --files ./data_folder/
```

### Working Directory Options

```bash
# Default: ./agentic_output/ (preserved after completion)
agentic-data-scientist "Analyze data" \
  --mode orchestrated \
  --files data.csv

# Custom working directory
agentic-data-scientist "Generate report" \
  --mode orchestrated \
  --files data.csv \
  --working-dir ./my_analysis

# Temporary directory (auto-cleanup)
agentic-data-scientist "Quick exploration" \
  --mode simple \
  --files data.csv \
  --temp-dir

# Force keep files (override temp-dir cleanup)
agentic-data-scientist "Analysis" \
  --mode orchestrated \
  --files data.csv \
  --temp-dir \
  --keep-files
```

### Logging and Debugging

```bash
# Custom log file location
agentic-data-scientist "Analyze" \
  --mode orchestrated \
  --files data.csv \
  --log-file ./analysis.log

# Verbose logging
agentic-data-scientist "Debug issue" \
  --mode simple \
  --verbose
```

## Real-World Examples

### Example 1: Complex Data Analysis (Orchestrated Mode)

```bash
# Comprehensive analysis with multiple stages
agentic-data-scientist \
  "Perform exploratory data analysis on sales data, \
   identify trends, create visualizations, \
   and build a predictive model for future sales" \
  --mode orchestrated \
  --files sales_2024.csv \
  --working-dir ./sales_analysis \
  --log-file analysis.log
```

**What happens:**
1. **Planning Phase**: Creates detailed plan with stages (EDA, visualization, modeling)
2. **Execution Phase**: Implements each stage iteratively with validation
3. **Validation**: Checks success criteria after each stage
4. **Adaptation**: Adjusts plan based on discoveries (e.g., data quality issues)
5. **Summary**: Generates comprehensive report with all findings

### Example 2: Quick Scripting (Simple Mode)

```bash
# Fast coding without planning overhead
agentic-data-scientist \
  "Write a Python script that reads multiple CSV files, \
   merges them on a common ID column, \
   and exports to Excel with formatting" \
  --mode simple \
  --files data1.csv data2.csv data3.csv \
  --temp-dir
```

**What happens:**
- Direct execution with coding agent (no planning phase)
- Quick turnaround for straightforward tasks
- Temporary directory auto-cleanup

### Example 3: Multi-File Statistical Analysis

```bash
# Compare multiple datasets
agentic-data-scientist \
  "Compare the distribution of features across treatment groups, \
   perform statistical tests (t-test, ANOVA), \
   and generate publication-ready plots" \
  --mode orchestrated \
  -f control.csv \
  -f treatment_a.csv \
  -f treatment_b.csv \
  --working-dir ./stats_analysis
```

### Example 4: Directory-Based Analysis

```bash
# Process all files in a directory
agentic-data-scientist \
  "Analyze all patient data files in the folder, \
   aggregate results, and create summary statistics" \
  --mode orchestrated \
  --files ./patient_data/ \
  --working-dir ./patient_analysis
```

## Python API Usage

For programmatic access, use the Python API:

```python
from agentic_data_scientist.cli import main
import sys

# Prepare arguments
sys.argv = [
    'agentic-data-scientist',
    'Perform clustering analysis on customer data',
    '--mode', 'orchestrated',
    '--files', 'customers.csv',
    '--working-dir', './clustering_output'
]

# Run
main()
```

Or use the workflow directly:

```python
import asyncio
from pathlib import Path
from agentic_data_scientist.workflow import create_workflow

async def run_analysis():
    # Create workflow
    workflow = create_workflow(
        query="Analyze customer segments",
        mode="orchestrated",
        files=[Path("customers.csv")],
        working_dir=Path("./output"),
        disable_network=False
    )
    
    # Execute
    result = await workflow.execute()
    print(result)

asyncio.run(run_analysis())
```

## Common Patterns

### Pattern 1: Iterative Data Exploration

```bash
# Start with simple mode for quick exploration
agentic-data-scientist \
  "Load dataset and show basic statistics" \
  --mode simple \
  --files data.csv

# Then use orchestrated mode for deep analysis
agentic-data-scientist \
  "Perform full statistical analysis including outlier detection, \
   correlation analysis, and clustering" \
  --mode orchestrated \
  --files data.csv \
  --working-dir ./deep_analysis
```

### Pattern 2: Pipeline Development

```bash
# Use orchestrated mode to develop a complete pipeline
agentic-data-scientist \
  "Create a data processing pipeline that: \
   1) Cleans and normalizes raw data \
   2) Engineers new features \
   3) Splits into train/test \
   4) Trains multiple models \
   5) Evaluates and selects best model \
   6) Exports model and metrics" \
  --mode orchestrated \
  --files raw_data.csv \
  --working-dir ./ml_pipeline
```

### Pattern 3: Report Generation

```bash
# Generate comprehensive reports
agentic-data-scientist \
  "Analyze quarterly sales data and create an executive report \
   with visualizations, key metrics, and recommendations" \
  --mode orchestrated \
  --files q1_sales.csv q2_sales.csv q3_sales.csv q4_sales.csv \
  --working-dir ./quarterly_report
```

### Pattern 4: Debugging with Verbose Logs

```bash
# Enable verbose logging for troubleshooting
agentic-data-scientist \
  "Complex analysis task" \
  --mode orchestrated \
  --files data.csv \
  --verbose \
  --log-file debug.log \
  --keep-files
```

## Multi-Agent Workflow Details

### Agent Roles

1. **Plan Maker**: Creates comprehensive plans with stages and success criteria
2. **Plan Reviewer**: Validates plans are complete before execution
3. **Plan Parser**: Converts plans to structured executable stages
4. **Stage Orchestrator**: Manages execution cycle and adaptation
5. **Coding Agent**: Implements stages (powered by Claude Code with 380+ scientific skills)
6. **Review Agent**: Validates implementations against requirements
7. **Criteria Checker**: Tracks progress against success criteria
8. **Stage Reflector**: Adapts remaining stages based on learnings
9. **Summary Agent**: Synthesizes work into final report

### Workflow Phases

**Planning Phase:**
```
User Query → Plan Maker → Plan Reviewer → Plan Parser → Structured Plan
```

**Execution Phase (per stage):**
```
Stage → Coding Agent → Review Agent → Criteria Checker → Stage Reflector
```

**Summary Phase:**
```
All Completed Stages → Summary Agent → Final Report
```

## Troubleshooting

### API Key Errors

```bash
# Verify keys are set
echo $OPENROUTER_API_KEY
echo $ANTHROPIC_API_KEY

# Set them if missing
export OPENROUTER_API_KEY="your_key"
export ANTHROPIC_API_KEY="your_key"
```

### Claude Code Not Found

```bash
# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Verify installation
claude-code --version
```

### Network Access Issues

```bash
# Disable network tools if causing problems
export DISABLE_NETWORK_ACCESS=true

# Or in .env file
echo "DISABLE_NETWORK_ACCESS=true" >> .env
```

### File Upload Failures

```bash
# Verify file exists
ls -la data.csv

# Use absolute paths
agentic-data-scientist "Analyze" \
  --mode orchestrated \
  --files /absolute/path/to/data.csv

# Check directory permissions for recursive upload
ls -la ./data_folder/
```

### Working Directory Issues

```bash
# Ensure directory is writable
mkdir -p ./output
chmod 755 ./output

# Use temp directory if permission issues
agentic-data-scientist "Analyze" \
  --mode orchestrated \
  --files data.csv \
  --temp-dir
```

### Execution Hanging

```bash
# Use verbose mode to see what's happening
agentic-data-scientist "Query" \
  --mode orchestrated \
  --files data.csv \
  --verbose

# Try simple mode to isolate planning vs execution issues
agentic-data-scientist "Query" \
  --mode simple \
  --files data.csv
```

### Output Not Preserved

```bash
# Default behavior preserves files in ./agentic_output/
ls -la ./agentic_output/

# Explicitly set working directory
agentic-data-scientist "Analyze" \
  --mode orchestrated \
  --files data.csv \
  --working-dir ./my_output

# Use --keep-files to override temp-dir cleanup
agentic-data-scientist "Analyze" \
  --mode orchestrated \
  --files data.csv \
  --temp-dir \
  --keep-files
```

## Mode Selection Guide

**Use Orchestrated Mode when:**
- Task is complex with multiple stages
- Need thorough planning and validation
- Quality and completeness are critical
- Task requires iterative refinement
- Want comprehensive final report

**Use Simple Mode when:**
- Quick scripting or one-off tasks
- Simple question answering
- Prototyping or exploration
- Want fast turnaround
- Don't need multi-stage workflow

## Advanced Configuration

### Custom Prompts

Extend the framework by customizing agent prompts:

```python
from agentic_data_scientist.prompts import PLAN_MAKER_PROMPT

# Modify prompts for domain-specific needs
custom_prompt = PLAN_MAKER_PROMPT + """
Additional domain context:
- Focus on genomics data
- Use bioinformatics best practices
"""
```

### MCP Server Integration

The framework supports Model Context Protocol for custom tools:

```python
# Configure MCP servers in your workflow
# Agents automatically gain access to tools
```

### Access to Claude Scientific Skills

The coding agent has access to 380+ scientific computing skills including:
- Statistical analysis
- Machine learning
- Data visualization
- Bioinformatics
- Scientific computing libraries

These are automatically available during execution phase.
