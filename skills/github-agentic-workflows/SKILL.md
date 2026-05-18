---
name: github-agentic-workflows
description: Create and manage AI-powered GitHub workflows that autonomously maintain repositories, review code, update docs, and handle developer tasks.
triggers:
  - "set up a GitHub agentic workflow"
  - "create an AI workflow for repository maintenance"
  - "add automated code review with gh-aw"
  - "configure repo assist workflow"
  - "how do I use GitHub agentic workflows"
  - "automate issue triage with AI"
  - "set up CI doctor workflow"
  - "create custom agentic workflow"
---

# GitHub Agentic Workflows

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

GitHub Agentic Workflows (gh-aw) is a framework for creating AI-powered automation workflows that can autonomously maintain repositories, review code, triage issues, update documentation, and perform developer tasks. Workflows are defined in simple Markdown files and run as GitHub Actions.

## What It Does

- **Autonomous Repository Maintenance**: AI agents that triage issues, label PRs, fix bugs, and maintain documentation
- **Code Review**: Automated PR reviews with configurable personas (grumpy reviewer, nitpick reviewer, etc.)
- **CI/CD Intelligence**: Monitor and diagnose CI failures, optimize workflows, track costs
- **Documentation**: Auto-update docs, maintain wikis, check links, generate glossaries
- **Planning & Reporting**: Generate status reports, research summaries, activity chronicles
- **Security**: Scan for malicious code, generate VEX statements for security alerts

## Installation

### Prerequisites

1. GitHub repository with Actions enabled
2. GitHub App or Personal Access Token with appropriate permissions
3. Access to an AI model provider (OpenAI, Anthropic, etc.)

### Basic Setup

1. Install the `gh-aw` CLI:

```bash
# Using npm
npm install -g @github/gh-aw

# Or using GitHub CLI extension
gh extension install github/gh-aw
```

2. Create `.github/workflows` directory in your repository:

```bash
mkdir -p .github/workflows
```

3. Copy a workflow from the Agentics collection:

```bash
# Example: Issue Triage workflow
curl -o .github/workflows/issue-triage.yml \
  https://raw.githubusercontent.com/githubnext/agentics/main/.github/workflows/issue-triage.yml

curl -o .github/workflows/issue-triage.md \
  https://raw.githubusercontent.com/githubnext/agentics/main/workflows/issue-triage.md
```

4. Configure secrets in your repository settings:
   - `GITHUB_TOKEN` (automatically available)
   - `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` (for AI model access)

## Workflow Structure

Agentic workflows consist of two files:

1. **YAML file** (`.github/workflows/*.yml`) - GitHub Actions configuration
2. **Markdown file** (`.github/workflows/*.md`) - Agent instructions and behavior

### Example: Issue Triage Workflow

**`.github/workflows/issue-triage.yml`**:

```yaml
name: Issue Triage
on:
  issues:
    types: [opened, edited]
  pull_request:
    types: [opened, edited]

jobs:
  triage:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: github/gh-aw@v1
        with:
          workflow: .github/workflows/issue-triage.md
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

**`.github/workflows/issue-triage.md`**:

```markdown
# Issue Triage Agent

You are a helpful issue triage assistant. When a new issue or pull request is opened:

1. Read the title and body carefully
2. Analyze the content to determine appropriate labels
3. Apply relevant labels from the repository's label set
4. Add a welcoming comment if this is a first-time contributor

## Available Tools

- `github.issues.addLabels` - Add labels to issues
- `github.issues.createComment` - Comment on issues
- `github.pulls.requestReviewers` - Request reviewers for PRs

## Labels to Consider

- `bug` - Something isn't working
- `enhancement` - New feature or request
- `documentation` - Documentation improvements
- `good first issue` - Good for newcomers
- `help wanted` - Extra attention needed

## Example Analysis

For a bug report mentioning crashes, apply: `bug`, potentially `help wanted`
For a feature request, apply: `enhancement`
For documentation fixes, apply: `documentation`, potentially `good first issue`
```

## Key Workflow Types

### Maintainer Workflows

**Repo Assist** - Comprehensive repository assistant:

```yaml
name: Repo Assist
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours
  issues:
    types: [opened, edited, labeled]
  pull_request:
    types: [opened, edited, synchronize]

jobs:
  assist:
    runs-on: ubuntu-latest
    steps:
      - uses: github/gh-aw@v1
        with:
          workflow: .github/workflows/repo-assist.md
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Code Review Workflows

**Grumpy Reviewer** - On-demand opinionated reviews:

```yaml
name: Grumpy Reviewer
on:
  issue_comment:
    types: [created]

jobs:
  review:
    if: contains(github.event.comment.body, '/grumpy-review')
    runs-on: ubuntu-latest
    steps:
      - uses: github/gh-aw@v1
        with:
          workflow: .github/workflows/grumpy-reviewer.md
```

### Documentation Workflows

**Daily Doc Updater**:

```yaml
name: Documentation Updater
on:
  schedule:
    - cron: '0 9 * * *'  # Daily at 9 AM
  workflow_dispatch:

jobs:
  update-docs:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: github/gh-aw@v1
        with:
          workflow: .github/workflows/doc-updater.md
```

## Custom Workflow Creation

### Basic Template

```markdown
# My Custom Agent

## Role
You are a [describe the agent's role and purpose].

## Triggers
This workflow runs when [describe trigger conditions].

## Tasks

1. [First task description]
2. [Second task description]
3. [Third task description]

## Available Tools

List the GitHub API tools you'll use:
- `github.issues.create` - Create new issues
- `github.repos.createOrUpdateFileContents` - Update files
- `github.pulls.create` - Create pull requests

## Guidelines

- [Guideline 1]
- [Guideline 2]
- [Guideline 3]

## Output Format

Provide results as:
- Summary of actions taken
- Links to created issues/PRs
- Any recommendations
```

### Using Shared Fragments

Import reusable components:

```markdown
# My Workflow

imports: [shared/formatting.md, shared/arxiv.md]

## Tasks

Use the arXiv MCP server to search for relevant papers on [topic].
Format the output using the standard formatting guidelines.
```

### Advanced: MCP Server Integration

```markdown
# Research Assistant

imports: [shared/arxiv.md, shared/markitdown.md]

## Tasks

1. Search arXiv for papers on "machine learning optimization"
2. Download the top 3 papers as PDFs
3. Convert PDFs to Markdown using MarkItDown
4. Summarize findings in an issue

## MCP Servers

- **arxiv**: Search and retrieve academic papers
- **markitdown**: Convert PDFs to readable Markdown
```

## Configuration

### Environment Variables

Set in repository secrets or `.env`:

```bash
# Required
GITHUB_TOKEN=ghp_your_token_here
OPENAI_API_KEY=sk-your-key-here

# Optional - Alternative AI providers
ANTHROPIC_API_KEY=sk-ant-your-key-here
AZURE_OPENAI_API_KEY=your-azure-key-here
AZURE_OPENAI_ENDPOINT=https://your-endpoint.openai.azure.com

# Optional - Cost tracking
GH_AW_ENABLE_COST_TRACKING=true
```

### Workflow Permissions

Required permissions in workflow YAML:

```yaml
permissions:
  contents: write        # To create/update files
  issues: write          # To manage issues
  pull-requests: write   # To manage PRs
  discussions: write     # To manage discussions (if needed)
```

## Common Patterns

### Command-Triggered Workflows

Allow users to trigger agents via comments:

```yaml
on:
  issue_comment:
    types: [created]

jobs:
  command-handler:
    if: |
      contains(github.event.comment.body, '/plan') ||
      contains(github.event.comment.body, '/fix')
    runs-on: ubuntu-latest
    steps:
      - uses: github/gh-aw@v1
        with:
          workflow: .github/workflows/commands.md
```

### Scheduled Maintenance

Run workflows on a schedule:

```yaml
on:
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Monday
    - cron: '0 9 * * *'  # Daily at 9 AM
    - cron: '0 */6 * * *'  # Every 6 hours
```

### Multi-Stage Workflows

Chain multiple agents together:

```markdown
# Multi-Stage Pipeline

## Stage 1: Analysis
Analyze the repository for issues.

## Stage 2: Planning
Create a plan to address the issues.

## Stage 3: Execution
Implement the plan and create PRs.

## Stage 4: Verification
Verify the changes and update status.
```

### Conditional Execution

```markdown
# Conditional Agent

## Pre-Check

Before proceeding, verify:
- There are open issues labeled "needs-triage"
- No other triage workflow is currently running
- The repository has been active in the last 7 days

If conditions aren't met, exit gracefully.

## Main Task

[Rest of workflow...]
```

## Real-World Examples

### Example 1: Bug Fix Automation

```markdown
# Bug Fix Agent

## Role
You are an experienced developer who investigates and fixes bugs.

## Workflow

1. **Identify**: Find issues labeled "bug" that haven't been updated in 3 days
2. **Investigate**: 
   - Read the issue description
   - Check related code files
   - Review recent commits that might be related
   - Look for similar closed issues
3. **Diagnose**: Determine the root cause
4. **Fix**: 
   - Create a branch named `fix/issue-{number}`
   - Implement the fix
   - Write or update tests
   - Commit changes with descriptive message
5. **PR**: Create a pull request referencing the issue
6. **Update**: Comment on the original issue with findings and PR link

## Tools

- `github.search.issuesAndPullRequests` - Find bugs
- `github.repos.getContent` - Read code files
- `github.git.createRef` - Create branch
- `github.repos.createOrUpdateFileContents` - Update files
- `github.pulls.create` - Create PR
- `github.issues.createComment` - Update issue

## Quality Standards

- All fixes must include tests
- Code must follow existing style conventions
- Commit messages must be descriptive
- PRs must reference the issue number
```

### Example 2: Documentation Sync

```markdown
# Documentation Synchronizer

imports: [shared/formatting.md]

## Role
Keep documentation in sync with code changes.

## Workflow

1. **Detect Changes**: 
   - Find files changed in the last 24 hours
   - Filter for source code files (.js, .py, .go, etc.)
   - Identify public APIs or functions modified

2. **Check Documentation**:
   - Locate corresponding documentation files
   - Compare documented behavior with actual code
   - Identify discrepancies

3. **Update Documentation**:
   - Create branch `docs/auto-sync-{date}`
   - Update affected documentation files
   - Add examples if new features were added
   - Update any code snippets

4. **Create PR**:
   - Title: "docs: sync with code changes from {date}"
   - Body: List all changes with links to commits
   - Request review from code authors

## File Mappings

- `src/api/*.js` → `docs/api/*.md`
- `src/cli/*.js` → `docs/cli-reference.md`
- `lib/*.py` → `docs/library-reference.md`

## Output Format

Use the standard formatting from shared/formatting.md
```

### Example 3: CI Cost Optimizer

```markdown
# CI Cost Optimizer

imports: [shared/reporting.md]

## Role
Monitor and optimize CI/CD costs.

## Analysis Tasks

1. **Collect Data**:
   - Get all workflow runs from the past 7 days
   - Calculate total execution time per workflow
   - Identify slowest steps in each workflow

2. **Identify Waste**:
   - Find workflows with high failure rates
   - Detect redundant test runs
   - Spot overly aggressive schedule triggers
   - Find unnecessarily large runners

3. **Generate Report**:
   - Calculate estimated costs (if applicable)
   - Create visualization of run times
   - Highlight optimization opportunities
   - Provide specific recommendations

4. **Create Issue**:
   - Title: "CI Optimization Report - Week {number}"
   - Include all findings and recommendations
   - Tag with "ci", "optimization", "cost-savings"

## Cost Calculations

- Small runner: ~$0.008/minute
- Medium runner: ~$0.016/minute
- Large runner: ~$0.032/minute

## Optimization Checks

- [ ] Are tests running in parallel where possible?
- [ ] Are caches being used effectively?
- [ ] Are workflows skipping unnecessary runs?
- [ ] Are matrix builds optimized?
- [ ] Are dependencies installed efficiently?
```

## Troubleshooting

### Workflow Not Triggering

Check trigger configuration:

```yaml
# Make sure triggers match your intent
on:
  issues:
    types: [opened, edited]  # Not just [opened]
  pull_request:
    types: [opened, synchronize]  # Include synchronize for new commits
```

### Permission Denied Errors

Ensure proper permissions:

```yaml
permissions:
  contents: write  # Required for file changes
  issues: write    # Required for issue operations
  pull-requests: write  # Required for PR operations
```

### AI Model Timeouts

Use streaming and set appropriate timeouts:

```markdown
## Configuration

- Model: claude-3-5-sonnet-20241022
- Max tokens: 4000
- Timeout: 300 seconds
- Stream: true
```

### Rate Limiting

Implement backoff and respect rate limits:

```markdown
## Rate Limiting

- Wait 1 second between API calls
- Use conditional requests (If-None-Match headers)
- Cache responses when possible
- Batch operations where supported
```

### Debugging Workflows

Enable debug logging:

```yaml
jobs:
  debug:
    runs-on: ubuntu-latest
    steps:
      - uses: github/gh-aw@v1
        with:
          workflow: .github/workflows/my-workflow.md
          debug: true
        env:
          ACTIONS_STEP_DEBUG: true
```

### Cost Management

Track and limit costs:

```markdown
imports: [shared/reporting.md]

## Cost Controls

- Maximum tokens per run: 100,000
- Alert threshold: $10/day
- Enable cost tracking in reports
- Use cheaper models for simple tasks (gpt-4o-mini)
```

## Best Practices

1. **Start Simple**: Begin with basic workflows (issue triage) before complex ones (code generation)
2. **Test Locally**: Use `gh-aw run` to test workflows before deploying
3. **Monitor Costs**: Enable cost tracking and set budgets
4. **Iterate**: Refine agent instructions based on actual behavior
5. **Use Shared Fragments**: Reuse common patterns from the shared library
6. **Clear Instructions**: Write explicit, step-by-step instructions for agents
7. **Error Handling**: Include fallback behavior for common failure cases
8. **Human Review**: For destructive operations, create PRs instead of direct commits

## Resources

- [Official Documentation](https://github.github.com/gh-aw/)
- [Agentics Collection](https://github.com/githubnext/agentics)
- [Agentics Beyond Code](https://github.com/chrizbo/agentics-beyond-code)
- [Example Workflows](https://github.com/githubnext/agentics/tree/main/workflows)
