---
name: github-agentic-workflows
description: Create and configure GitHub Agentic Workflows for automated repository maintenance, code review, CI monitoring, documentation, and more
triggers:
  - set up GitHub Agentic Workflows
  - create an agentic workflow for my repo
  - configure gh-aw for automated maintenance
  - add AI code review workflow
  - automate issue triage with agents
  - implement repo assist workflow
  - deploy agentic workflows to GitHub
  - configure CI doctor workflow
---

# GitHub Agentic Workflows (gh-aw)

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

GitHub Agentic Workflows (gh-aw) is a framework for creating autonomous AI agents that run as GitHub Actions workflows. It provides a collection of ready-to-use workflows for repository maintenance, code review, CI monitoring, documentation updates, and more. Workflows are defined in markdown files in `.github/workflows/` and can use tools, MCP servers, and GitHub APIs.

## Installation

### Prerequisites

1. A GitHub repository
2. GitHub Actions enabled
3. API keys for AI models (OpenAI, Anthropic, etc.)

### Basic Setup

1. **Add workflow files** to `.github/workflows/` directory:

```bash
mkdir -p .github/workflows
```

2. **Create a workflow file** (e.g., `.github/workflows/issue-triage.md`):

```markdown
---
name: Issue Triage
on:
  issues:
    types: [opened, edited]
  pull_request:
    types: [opened, edited]
schedule:
  - cron: "0 */6 * * *"
---

# Issue Triage

Automatically label and triage new issues and pull requests.

## Task

Review the issue or pull request and:
1. Add appropriate labels (bug, feature, documentation, etc.)
2. Determine priority (high, medium, low)
3. Add a brief triage comment explaining your assessment

Use the GitHub API tools to apply labels and post comments.
```

3. **Configure secrets** in GitHub repository settings:
   - `ANTHROPIC_API_KEY` or `OPENAI_API_KEY`
   - `GITHUB_TOKEN` (automatically provided)

4. **Install gh-aw CLI** (optional, for local testing):

```bash
npm install -g @github/gh-aw
```

## Core Concepts

### Workflow Structure

Workflows are markdown files with YAML frontmatter:

```markdown
---
name: Workflow Name
on:
  schedule:
    - cron: "0 0 * * *"  # Daily at midnight
  issues:
    types: [opened]
model: claude-3-5-sonnet-20241022
imports:
  - shared/formatting.md
---

# Workflow Title

Description of what this workflow does.

## Task

Detailed instructions for the agent.
```

### Triggers

Common trigger patterns:

```yaml
# Schedule-based
on:
  schedule:
    - cron: "0 9 * * 1"  # Weekly on Monday at 9 AM

# Event-based
on:
  issues:
    types: [opened, labeled]
  pull_request:
    types: [opened, synchronize]
  push:
    branches: [main]

# Manual trigger
on:
  workflow_dispatch:

# Comment commands
on:
  issue_comment:
    types: [created]
```

### Imports (Shared Fragments)

Reuse common configurations:

```yaml
imports:
  - shared/formatting.md     # Standard report formatting
  - shared/reporting.md      # Workflow run links
  - shared/arxiv.md          # arXiv research papers MCP
  - shared/markitdown.md     # Document conversion
  - shared/ffmpeg.md         # Video/audio processing
```

## Key Workflows

### Repo Assist (Comprehensive Maintenance)

```markdown
---
name: Repo Assist
on:
  schedule:
    - cron: "0 */4 * * *"  # Every 4 hours
  issues:
    types: [opened, labeled]
  pull_request:
    types: [opened]
model: claude-3-5-sonnet-20241022
---

# Repo Assist

A comprehensive repository assistant that triages issues, investigates problems, 
fixes bugs, and maintains activity summaries.

## Task

1. **Triage New Items**: Review unlabeled issues and PRs, add labels and priority
2. **Investigate Issues**: For issues labeled `needs-investigation`, analyze the 
   problem and add findings
3. **Fix Bugs**: For issues labeled `bug` and `ready`, create a PR with a fix
4. **Update Summary**: Maintain the ACTIVITY.md file with recent changes

Use GitHub API tools to search, label, comment, and create PRs.
```

### Issue Triage (Simple Labeling)

```markdown
---
name: Issue Triage
on:
  issues:
    types: [opened, edited]
  pull_request:
    types: [opened, edited]
model: gpt-4o
---

# Issue Triage

Triage and label new issues and pull requests.

## Task

For each new or updated issue/PR:
1. Read the title and body
2. Determine the type: bug, feature, documentation, question
3. Assess priority: high, medium, low
4. Add appropriate labels using the GitHub API
5. Post a brief triage comment

Be concise and helpful.
```

### CI Doctor (Failure Investigation)

```markdown
---
name: CI Doctor
on:
  workflow_run:
    workflows: ["*"]
    types: [completed]
model: claude-3-5-sonnet-20241022
---

# CI Doctor

Monitor CI workflows and investigate failures automatically.

## Task

When a workflow fails:
1. Fetch the workflow run logs
2. Identify the failure cause (flaky test, dependency issue, code bug, etc.)
3. Search for similar past failures
4. Determine if it's a known issue or new problem
5. Post a comment on the associated PR or create an issue
6. If it's a simple fix (e.g., dependency version), create a PR

Focus on actionable insights.
```

### Grumpy Reviewer (Code Review)

```markdown
---
name: Grumpy Reviewer
on:
  issue_comment:
    types: [created]
model: claude-3-5-sonnet-20241022
---

# Grumpy Reviewer

Triggered by `/grumpy-review` command. Provides thorough, opinionated code review.

## Task

When a PR comment contains `/grumpy-review`:
1. Fetch the PR diff
2. Review for: code quality, performance, security, maintainability, tests
3. Be critical but constructive
4. Point out potential bugs, anti-patterns, and missed edge cases
5. Suggest specific improvements with code examples
6. Post review comments on specific lines where possible

Be thorough and grumpy, but helpful.
```

### Documentation Updater

```markdown
---
name: Documentation Update on Push
on:
  push:
    branches: [main]
model: gpt-4o
imports:
  - shared/formatting.md
---

# Documentation Update

Update documentation automatically when code changes are pushed to main.

## Task

1. Fetch the diff of the recent push
2. Identify which docs might be affected (README, API docs, guides)
3. For each affected doc:
   - Check if updates are needed
   - Update the documentation to reflect code changes
   - Ensure examples still work
4. Create a PR with doc updates if changes are needed

Focus on keeping docs in sync with code.
```

### Weekly Research

```markdown
---
name: Weekly Research
on:
  schedule:
    - cron: "0 9 * * 1"  # Monday 9 AM
model: claude-3-5-sonnet-20241022
imports:
  - shared/arxiv.md
---

# Weekly Research

Collect research updates and industry trends relevant to this project.

## Task

1. Search arXiv for papers related to: [project topics]
2. Scan GitHub trending repos in relevant languages/topics
3. Check Hacker News and relevant subreddits
4. Summarize findings in RESEARCH.md
5. Create issues for promising ideas to investigate
6. Update the team via a discussion post

Focus on actionable insights and emerging trends.
```

### Dependabot PR Bundler

```markdown
---
name: Dependabot PR Bundler
on:
  schedule:
    - cron: "0 10 * * 1"  # Monday 10 AM
model: gpt-4o
---

# Dependabot PR Bundler

Bundle compatible dependabot updates into a single PR.

## Task

1. List all open dependabot PRs
2. Group by ecosystem (npm, pip, cargo, etc.)
3. For each group, determine which updates are compatible
4. Create a new branch
5. Apply compatible updates
6. Run tests
7. Create a bundled PR
8. Close the individual dependabot PRs with a comment linking to the bundle

Be conservative - only bundle if likely to be compatible.
```

## Configuration

### Model Selection

Specify the AI model in frontmatter:

```yaml
model: claude-3-5-sonnet-20241022    # Anthropic (recommended)
model: gpt-4o                         # OpenAI
model: gpt-4o-mini                    # Faster, cheaper
```

### Environment Variables

Configure via repository secrets:

- `ANTHROPIC_API_KEY` - Anthropic API key
- `OPENAI_API_KEY` - OpenAI API key
- `GITHUB_TOKEN` - Auto-provided by GitHub Actions
- `GH_AW_TOKEN` - Optional: PAT for cross-repo operations

### Rate Limiting & Costs

Control API usage:

```markdown
---
name: Expensive Workflow
on:
  schedule:
    - cron: "0 0 * * 0"  # Weekly instead of daily
model: gpt-4o-mini         # Use cheaper model
max_tokens: 4000           # Limit response length
---
```

### Firewall & Token Tracking

Enable cost tracking:

```yaml
# .github/workflows/cost-tracker.md
---
name: Cost Tracker
on:
  pull_request:
    types: [opened, synchronize]
---

Post token usage summaries using token-usage.jsonl from gh-aw's firewall.
```

## Common Patterns

### Multi-Stage Workflows

Break complex tasks into stages:

```markdown
## Task

### Stage 1: Investigation
1. Analyze the current state
2. Identify problems
3. Create investigation notes

### Stage 2: Planning
1. Review investigation notes
2. Create action plan
3. Estimate effort

### Stage 3: Execution
1. Implement fixes
2. Run tests
3. Create PR

Complete each stage before proceeding.
```

### Command-Triggered Workflows

Respond to comment commands:

```markdown
---
name: Repo Ask
on:
  issue_comment:
    types: [created]
---

# Repo Ask

Triggered by `/ask <question>` in comments.

## Task

When a comment starts with `/ask`:
1. Extract the question
2. Search relevant code and docs
3. Analyze and formulate answer
4. Post reply with findings and code references

Only respond to users with write access.
```

### Iterative Improvement

Build on previous work:

```markdown
## Task

1. Check if IMPROVEMENTS.md exists
2. If exists, read previous findings
3. Identify the next highest-priority improvement
4. Implement it
5. Update IMPROVEMENTS.md with progress

Run daily to continuously improve the codebase.
```

### Parent-Child Issue Management

```markdown
## Task

For issues labeled `epic`:
1. Parse the checklist of sub-tasks
2. Create child issues for uncreated tasks
3. Link child issues with "Sub-issue of #parent"
4. Update parent issue checklist with links
5. When all children close, close parent

Maintain the relationship automatically.
```

## Advanced Usage

### Custom Tools

Define custom tools in workflow:

```markdown
## Tools

### search_codebase
Search the codebase for patterns.
- pattern: string - Regex or text pattern
- file_types: string[] - File extensions to search

### run_benchmark
Execute performance benchmarks.
- test_file: string - Path to benchmark file
- iterations: number - Number of runs
```

### MCP Server Integration

Use Model Context Protocol servers:

```yaml
imports:
  - shared/arxiv.md          # Research papers
  - shared/markitdown.md     # Document conversion
  - shared/ffmpeg.md         # Media processing
```

Then use in task:

```markdown
## Task

1. Use the arXiv MCP to search for papers on: [topic]
2. Download top 3 papers
3. Use MarkItDown to convert PDFs to markdown
4. Analyze and summarize findings
5. Create an issue with recommendations
```

### Cross-Repository Operations

Use PAT for multi-repo workflows:

```markdown
## Task

1. Scan all repositories in the organization
2. Identify repos missing SECURITY.md
3. Create PRs in each repo with a template SECURITY.md
4. Track progress in a central dashboard issue

Requires GH_AW_TOKEN with org-wide permissions.
```

## Troubleshooting

### Workflow Not Triggering

Check:
- YAML frontmatter syntax is valid
- Trigger events are correct
- File is in `.github/workflows/` with `.md` extension
- Workflow is enabled in repository settings

### API Rate Limits

Solutions:
- Use `gpt-4o-mini` instead of `gpt-4o`
- Reduce workflow frequency
- Add caching logic
- Use conditional execution

### Agent Not Finding Files

Ensure agent searches before acting:

```markdown
## Task

1. **First**, list all markdown files in the docs/ directory
2. **Then**, read each file
3. Only after reading, make updates
```

### Permissions Errors

Check repository settings:
- Actions have write permissions
- Secrets are configured
- PAT has required scopes

### Workflow Debugging

Add logging:

```markdown
## Task

Log each step:
1. "Starting investigation..."
2. Log files found
3. Log analysis results
4. "Creating PR..."

Post logs as a comment if workflow fails.
```

### Model Context Limits

For large repositories:

```markdown
## Task

Work incrementally:
1. List all files to process
2. Process 10 files at a time
3. Maintain progress in PROGRESS.md
4. Resume from last position on next run

This avoids context limit issues.
```

## Best Practices

1. **Start small**: Begin with simple workflows like issue triage
2. **Test locally**: Use `gh-aw` CLI to test before deploying
3. **Monitor costs**: Use cost-tracker workflow
4. **Version control**: Keep workflows in git
5. **Documentation**: Comment complex logic
6. **Idempotency**: Design workflows to be safely re-runnable
7. **Graceful degradation**: Handle API failures gracefully
8. **Security**: Never commit API keys; use secrets
9. **Permissions**: Check user permissions before executing commands
10. **Observability**: Log actions and outcomes

## Example: Complete Workflow Setup

```bash
# 1. Create workflow directory
mkdir -p .github/workflows

# 2. Add repo assist workflow
cat > .github/workflows/repo-assist.md << 'EOF'
---
name: Repo Assist
on:
  schedule:
    - cron: "0 */4 * * *"
  issues:
    types: [opened]
  pull_request:
    types: [opened]
model: claude-3-5-sonnet-20241022
imports:
  - shared/formatting.md
---

# Repo Assist

Comprehensive repository maintenance assistant.

## Task

Every 4 hours:
1. Triage new issues and PRs (add labels, priority)
2. Investigate issues marked `needs-investigation`
3. Fix bugs marked `bug` and `ready` (create PRs)
4. Update ACTIVITY.md with recent changes
5. Close issues that are resolved but not closed

Be proactive but cautious with code changes.
EOF

# 3. Configure secrets in GitHub UI
# Settings > Secrets and variables > Actions > New repository secret
# Add: ANTHROPIC_API_KEY

# 4. Commit and push
git add .github/workflows/repo-assist.md
git commit -m "Add repo assist workflow"
git push

# 5. Monitor workflow runs
# Actions tab in GitHub repository
```

The workflow will now run automatically based on the schedule and events.
