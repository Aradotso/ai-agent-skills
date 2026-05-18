---
name: github-agentic-workflows
description: Create and manage AI-powered GitHub workflows that autonomously perform repository maintenance, code review, documentation, testing, and planning tasks
triggers:
  - set up agentic workflows for my repository
  - create an AI-powered GitHub workflow
  - add autonomous issue triage to my repo
  - configure repo assist workflow
  - implement automated code review with AI
  - add CI doctor workflow to monitor failures
  - set up automated documentation updates
  - create autonomous repository assistant
---

# GitHub Agentic Workflows

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

GitHub Agentic Workflows (gh-aw) is a framework for creating AI-powered GitHub Actions workflows that can autonomously perform complex repository tasks like issue triage, code review, bug fixing, documentation updates, CI monitoring, and team planning.

## What It Does

Agentic workflows are GitHub Actions workflows that use LLMs to:
- Triage and label issues/PRs automatically
- Review code and suggest improvements
- Fix bugs and update documentation
- Monitor CI failures and investigate root causes
- Generate status reports and planning documents
- Maintain code quality through automated refactoring
- Research and summarize industry trends

Each workflow operates autonomously on a schedule or trigger, using tools like GitHub CLI, filesystem access, and web search.

## Installation

### 1. Install the gh-aw CLI Extension

```bash
gh extension install github/gh-aw
```

### 2. Initialize in Your Repository

```bash
cd your-repository
gh aw init
```

This creates:
- `.github/workflows/` directory for workflow files
- Basic configuration structure
- Example workflow templates

### 3. Add Workflows

Copy workflow files from the Agentics repository:

```bash
# Example: Add Issue Triage workflow
curl -o .github/workflows/issue-triage.yml \
  https://raw.githubusercontent.com/githubnext/agentics/main/.github/workflows/issue-triage.yml
```

Or create custom workflows using the workflow templates.

### 4. Configure Secrets

Set up required secrets in your repository settings:

```bash
# GitHub token with appropriate permissions
gh secret set GITHUB_TOKEN

# LLM API keys (choose your provider)
gh secret set ANTHROPIC_API_KEY
gh secret set OPENAI_API_KEY
```

## Workflow Structure

Agentic workflows are Markdown files that define:
- **Model**: Which LLM to use (e.g., `claude-3-5-sonnet`)
- **Triggers**: When to run (schedule, events)
- **Tools**: Available capabilities (GitHub API, filesystem, search)
- **Instructions**: What the agent should do

### Basic Workflow Template

```yaml
# .github/workflows/example-workflow.yml
name: Example Agentic Workflow
on:
  schedule:
    - cron: '0 9 * * 1'  # Weekly on Monday
  workflow_dispatch:

jobs:
  run:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: github/gh-aw@v1
        with:
          workflow: workflows/example.md
          github-token: ${{ secrets.GITHUB_TOKEN }}
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Workflow Definition (workflows/example.md)

```markdown
---
model: claude-3-5-sonnet-20241022
tools:
  - gh
  - filesystem
---

# Example Workflow

Your instructions here. Be specific about:
1. What to analyze
2. What actions to take
3. How to report results

## Context

The agent has access to:
- GitHub CLI (`gh`) for repository operations
- Filesystem for reading/writing files
- Current repository context

## Task

Analyze recent issues and create a summary report.

Steps:
1. Fetch issues from the last 7 days
2. Categorize by label and status
3. Create a summary markdown file
4. Post as a new issue

## Output Format

Create `reports/weekly-summary.md` with:
- Issue count by category
- Top contributors
- Action items
```

## Key Workflows

### Issue Triage

Automatically label and categorize issues:

```markdown
---
model: claude-3-5-sonnet-20241022
tools:
  - gh
---

# Issue Triage

Analyze new issues and apply appropriate labels.

For each unlabeled issue:
1. Read the issue title and body
2. Determine appropriate labels (bug, enhancement, documentation, etc.)
3. Apply labels using `gh issue edit <number> --add-label "label"`
4. Add a welcoming comment if it's from a first-time contributor

Only process issues created in the last 24 hours.
```

### Repo Assist

Comprehensive repository assistant:

```markdown
---
model: claude-3-5-sonnet-20241022
tools:
  - gh
  - filesystem
  - web-search
---

# Repo Assist

A multi-task repository assistant that:
1. Triages new issues (apply labels, prioritize)
2. Investigates bug reports (reproduce, analyze)
3. Fixes simple bugs (create PR with fix)
4. Updates documentation (align with code changes)
5. Maintains activity summaries (weekly reports)

## Priority Order

1. Critical bugs (label: critical)
2. Security issues
3. Documentation updates
4. Feature requests
5. Housekeeping tasks

## Actions

- For bugs: Investigate, reproduce, fix if simple
- For features: Analyze feasibility, comment with assessment
- For docs: Update outdated content, fix broken links
- Create PRs for all changes with detailed descriptions
```

### CI Doctor

Monitor and fix CI failures:

```markdown
---
model: claude-3-5-sonnet-20241022
tools:
  - gh
  - filesystem
---

# CI Doctor

Monitor workflow runs and investigate failures.

1. List failed workflow runs from the last 24 hours
2. For each failure:
   - Download logs using `gh run view <run-id> --log`
   - Analyze error messages
   - Identify root cause (flaky test, dependency issue, etc.)
3. Create issues for recurring failures
4. Post summary comment on PRs with failed checks

## Investigation Steps

- Check for timeout issues
- Look for dependency conflicts
- Identify flaky tests (intermittent failures)
- Analyze error patterns across multiple runs
```

## Using Shared Workflow Fragments

Import reusable components:

```markdown
---
model: claude-3-5-sonnet-20241022
imports:
  - shared/formatting.md
  - shared/arxiv.md
tools:
  - gh
  - filesystem
---

# Research Workflow

Use arxiv MCP server to search papers and formatting guidelines.
```

## Command-Triggered Workflows

Create workflows triggered by issue comments:

```yaml
# .github/workflows/plan-command.yml
name: Plan Command
on:
  issue_comment:
    types: [created]

jobs:
  plan:
    if: |
      github.event.comment.body == '/plan' &&
      (github.event.comment.author_association == 'OWNER' ||
       github.event.comment.author_association == 'MEMBER')
    runs-on: ubuntu-latest
    steps:
      - uses: github/gh-aw@v1
        with:
          workflow: workflows/plan.md
          github-token: ${{ secrets.GITHUB_TOKEN }}
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Common Patterns

### Reading Repository Data

```bash
# List recent issues
gh issue list --limit 50 --json number,title,labels,createdAt

# Get issue details
gh issue view 123 --json body,comments

# List PRs
gh pr list --state open --json number,title,headRefName

# Get file content
gh api repos/:owner/:repo/contents/path/to/file
```

### Creating Issues

```bash
gh issue create \
  --title "Weekly Summary - 2024-01-15" \
  --body-file summary.md \
  --label "report,automated"
```

### Creating Pull Requests

```bash
# Create branch
git checkout -b fix/update-docs

# Make changes
echo "Updated content" > docs/README.md
git add docs/README.md
git commit -m "docs: update README with latest changes"

# Push and create PR
git push origin fix/update-docs
gh pr create \
  --title "docs: update README" \
  --body "Automated documentation update based on recent changes" \
  --label "documentation,automated"
```

### Working with Files

```bash
# Read file
cat .github/workflows/ci.yml

# Create directory
mkdir -p reports

# Write report
cat > reports/summary.md << 'EOF'
# Weekly Summary
...
EOF
```

### Searching Code

```bash
# Search for TODO comments
gh search code --repo owner/repo "TODO"

# Find function definitions
gh search code --repo owner/repo "def process_data"
```

## Configuration

### Permissions

Workflows need appropriate permissions in the YAML:

```yaml
permissions:
  contents: write      # For creating/updating files
  issues: write        # For managing issues
  pull-requests: write # For managing PRs
  checks: read         # For reading CI status
```

### Rate Limiting

GitHub API has rate limits. For high-frequency workflows:

```markdown
## Rate Limit Handling

- Check rate limit before bulk operations: `gh api rate_limit`
- Process in batches with delays
- Use GraphQL for complex queries (fewer API calls)
- Cache results when possible
```

### Model Selection

Choose appropriate models:

```markdown
# Fast, cost-effective
model: claude-3-5-haiku-20241022

# Balanced
model: claude-3-5-sonnet-20241022

# Most capable
model: claude-3-opus-20240229
```

## Troubleshooting

### Workflow Not Running

Check:
1. Workflow file syntax: `gh workflow view <name>`
2. Permissions in repository settings
3. Branch protection rules
4. Schedule syntax (cron)

```bash
# List workflows
gh workflow list

# View workflow runs
gh run list --workflow=issue-triage.yml

# View logs
gh run view <run-id> --log
```

### Authentication Errors

Ensure secrets are set:

```bash
gh secret list

# Set missing secrets
gh secret set ANTHROPIC_API_KEY
```

### Agent Not Performing Actions

Review the workflow definition:
- Are instructions clear and specific?
- Does the agent have the right tools?
- Are there conflicting instructions?

Enable verbose logging:

```yaml
- uses: github/gh-aw@v1
  with:
    workflow: workflows/example.md
    github-token: ${{ secrets.GITHUB_TOKEN }}
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    debug: true
```

### Rate Limit Issues

Monitor API usage:

```bash
gh api rate_limit
```

Reduce frequency or batch operations:

```markdown
Process maximum 10 issues per run to stay within rate limits.
```

## Real-World Examples

### Daily Documentation Sync

```markdown
---
model: claude-3-5-sonnet-20241022
tools:
  - gh
  - filesystem
---

# Documentation Sync

Keep documentation aligned with code changes.

1. Get merged PRs from last 24 hours:
   `gh pr list --state merged --json number,title,files,mergedAt --limit 20`

2. For each PR that modified code in `src/`:
   - Check if corresponding docs exist in `docs/`
   - Read both code and docs
   - Identify discrepancies

3. Update docs to reflect code changes

4. Create PR with updates:
   - Branch: `docs/sync-YYYY-MM-DD`
   - Title: "docs: sync with recent code changes"
   - Body: List specific changes and PRs referenced
```

### Dependabot PR Bundler

```markdown
---
model: claude-3-5-sonnet-20241022
tools:
  - gh
  - filesystem
---

# Dependabot PR Bundler

Bundle compatible dependency updates.

1. List open Dependabot PRs:
   `gh pr list --author app/dependabot --state open`

2. Group by ecosystem (npm, pip, etc.)

3. For each group:
   - Create new branch: `deps/bundled-{ecosystem}-{date}`
   - Cherry-pick all compatible updates
   - Run tests to verify compatibility

4. Create single PR with all updates

5. Close individual Dependabot PRs with comment:
   "Bundled into #XXX for easier review"
```

### Weekly Team Status

```markdown
---
model: claude-3-5-sonnet-20241022
tools:
  - gh
  - filesystem
---

# Team Status Report

Generate weekly team activity summary.

1. Time range: Last 7 days

2. Collect metrics:
   - PRs merged: `gh pr list --state merged --json author,title,mergedAt`
   - Issues closed: `gh issue list --state closed --json author,title,closedAt`
   - New issues: `gh issue list --json author,title,createdAt`
   - Active contributors

3. Generate report:
   ```
   # Team Status - Week of {date}
   
   ## 🎉 Highlights
   - {count} PRs merged
   - {count} issues resolved
   
   ## 👥 Top Contributors
   - {name}: {contributions}
   
   ## 📊 Trends
   - {insight}
   ```

4. Post as discussion in team repository
```

## Best Practices

1. **Start Simple**: Begin with read-only workflows before enabling write operations
2. **Be Specific**: Clear, detailed instructions produce better results
3. **Test Manually**: Run workflows with `workflow_dispatch` before scheduling
4. **Monitor Costs**: Track LLM API usage and set budgets
5. **Review Actions**: Always review agent-created PRs before merging
6. **Use Labels**: Tag automated content for easy filtering
7. **Iterate**: Refine workflows based on results and feedback

## Environment Variables

Reference secrets in workflow definitions:

```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

Never hardcode API keys or tokens in workflow files.
