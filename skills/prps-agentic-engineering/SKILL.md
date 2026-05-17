---
name: prps-agentic-engineering
description: Product Requirement Prompts (PRP) methodology for AI-assisted development with validation loops and autonomous execution
triggers:
  - create a PRP for this feature
  - generate implementation plan from PRD
  - start autonomous ralph loop
  - investigate this GitHub issue with PRP
  - create a product requirement prompt
  - implement this plan with validation
  - review this PR with PRP workflow
  - debug using 5 whys methodology
---

# PRP (Product Requirement Prompts) - Agentic Engineering

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

PRP (Product Requirement Prompt) is a methodology and toolset for AI-assisted development that combines traditional PRDs with curated codebase intelligence and autonomous validation loops. It enables AI agents to deliver production-ready code on the first pass by providing complete context, patterns, and validation commands.

**Key Innovation**: PRP = PRD + codebase intelligence + agent/runbook

## Installation

### Option 1: Copy Commands to Existing Project

```bash
# From your project root
git clone https://github.com/Wirasm/PRPs-agentic-eng.git /tmp/prp-temp
cp -r /tmp/prp-temp/.claude/commands/prp-core .claude/commands/
rm -rf /tmp/prp-temp
```

### Option 2: Clone Full Repository

```bash
git clone https://github.com/Wirasm/PRPs-agentic-eng.git
cd PRPs-agentic-eng
```

### Setup Ralph Loop (Optional but Recommended)

Create `.claude/settings.local.json`:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/prp-ralph-stop.sh"
          }
        ]
      }
    ]
  }
}
```

Create `.claude/hooks/prp-ralph-stop.sh`:

```bash
#!/bin/bash
if [ -f .claude/prp-ralph.state.md ]; then
    echo "Ralph loop still active - stopping"
    exit 1
fi
exit 0
```

Make executable:

```bash
chmod +x .claude/hooks/prp-ralph-stop.sh
```

## Project Structure

```
your-project/
├── .claude/
│   ├── commands/prp-core/        # PRP command files
│   ├── hooks/                    # Stop hooks for Ralph
│   ├── PRPs/                     # Generated artifacts
│   │   ├── prds/                 # Product requirement docs
│   │   ├── plans/                # Implementation plans
│   │   │   └── completed/        # Archived plans
│   │   ├── reports/              # Implementation reports
│   │   ├── issues/               # Issue investigations
│   │   │   └── completed/        # Archived investigations
│   │   └── reviews/              # PR reviews
│   └── settings.local.json       # Ralph hook configuration
├── PRPs/
│   ├── templates/                # PRP templates
│   │   ├── prp_base.md
│   │   ├── prp_story_task.md
│   │   └── prp_planning.md
│   └── ai_docs/                  # Curated library docs
└── CLAUDE.md                     # Project guidelines
```

## Core Workflow Commands

### `/prp-prd` - Interactive PRD Generator

Creates a comprehensive Product Requirement Document with implementation phases.

```bash
/prp-prd "user authentication system with JWT"
```

**Output**: `.claude/PRPs/prds/user-auth-system.prd.md`

**PRD Structure**:
- Executive Summary
- Goals & Success Metrics
- User Stories
- Technical Requirements
- Implementation Phases Table
- Dependencies & Constraints

**Implementation Phases Table**:
```markdown
| # | Phase | Description | Status | Parallel | Depends | PRP Plan |
|---|-------|-------------|--------|----------|---------|----------|
| 1 | Auth  | JWT tokens  | pending | -       | -       | -        |
| 2 | API   | Auth endpoints | pending | -    | 1       | -        |
| 3 | UI    | Login forms | pending | with 4  | 2       | -        |
| 4 | Tests | Test suite  | pending | with 3  | 2       | -        |
```

### `/prp-plan` - Create Implementation Plan

Generates detailed implementation plan from PRD phase or free-form description.

**From PRD**:
```bash
/prp-plan .claude/PRPs/prds/user-auth-system.prd.md
```

Auto-selects next pending phase from PRD.

**From Description**:
```bash
/prp-plan "add pagination to the users API endpoint"
```

**Plan Structure**:
```markdown
# Implementation Plan: Feature Name

## Context
- Relevant files
- Dependencies
- Existing patterns

## Tasks
1. [ ] Task description
   - Subtask details
   - Files to modify

## Validation Commands
npm run type-check
npm run lint
npm test
npm run build

## Success Criteria
- [ ] All tests pass
- [ ] No type errors
- [ ] Follows existing patterns
```

### `/prp-implement` - Execute Plan

Implements plan with validation loops.

```bash
/prp-implement .claude/PRPs/plans/add-pagination.plan.md
```

**Process**:
1. Reads plan and context
2. Executes tasks in order
3. Runs validation commands
4. Creates implementation report
5. Updates PRD status (if from PRD)
6. Archives plan to `completed/`

**Implementation Report** (`.claude/PRPs/reports/feature-name.report.md`):
```markdown
# Implementation Report: Feature Name

## Summary
Brief overview of changes

## Changes Made
- File modifications
- New files created
- Dependencies added

## Validation Results
✓ Type check passed
✓ Linting passed
✓ Tests passed (42 passing)
✓ Build successful

## Challenges & Solutions
- Challenge faced
- Solution applied

## Next Steps
- Suggested follow-ups
```

## Issue & Debug Workflow

### `/prp-issue-investigate` - Analyze GitHub Issue

Creates investigation artifact for bug fixes or feature requests.

```bash
/prp-issue-investigate 123
```

**Investigation Artifact** (`.claude/PRPs/issues/issue-123-investigation.md`):
```markdown
# Issue Investigation: #123

## Issue Summary
[Auto-fetched from GitHub]

## Root Cause Analysis
- What's happening
- Why it's happening
- Where in codebase

## Proposed Solution
1. Change X in file Y
2. Add validation Z

## Implementation Plan
- [ ] Fix core issue
- [ ] Add tests
- [ ] Update docs

## Validation
npm test -- issue-123
npm run lint
```

### `/prp-issue-fix` - Execute Fix

Implements fix from investigation artifact.

```bash
/prp-issue-fix 123
```

Reads investigation, executes plan, creates PR-ready changes.

### `/prp-debug` - Root Cause Analysis

Deep debugging with 5 Whys methodology.

```bash
/prp-debug "users can't login after password reset"
```

**Output**:
```markdown
# Debug Report: Issue Description

## 5 Whys Analysis
1. Why? Users see "invalid token" error
2. Why? Token expired before email delivered
3. Why? Email queue has 5min delay
4. Why? Queue worker throttled
5. Why? Rate limit too conservative

## Root Cause
Queue worker rate limit set to 10/min, should be 100/min

## Fix
Update `config/queue.js` rate limit configuration

## Validation
- Test password reset flow
- Check email delivery time
```

## Git & Review Commands

### `/prp-commit` - Smart Commit

Natural language file targeting for commits.

```bash
/prp-commit "fix validation bug in user registration" --files "auth, validation"
```

Auto-detects files matching keywords, creates semantic commit.

**Commit Message Format**:
```
fix(auth): fix validation bug in user registration

- Updated UserValidator.validate() to check email format
- Added test coverage for edge cases
- Fixes #123
```

### `/prp-pr` - Create Pull Request

Generates PR with template support.

```bash
/prp-pr "Add user authentication" --base main --head feature/auth
```

**PR Template** (if `.github/pull_request_template.md` exists):
```markdown
## Description
Added JWT-based authentication system

## Changes
- Implemented JWT token generation
- Added auth middleware
- Created login/logout endpoints

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [x] Code follows style guide
- [x] Self-review completed
- [x] Documentation updated
```

### `/prp-review` - PR Code Review

Comprehensive code review with best practices.

```bash
/prp-review 456
```

**Review Report** (`.claude/PRPs/reviews/pr-456-review.md`):
```markdown
# PR Review: #456

## Summary
Well-structured implementation with minor suggestions

## Security Issues
⚠️ HIGH: API key exposed in config file
- Move to environment variable

## Performance Concerns
💡 MEDIUM: N+1 query in user list
- Use eager loading

## Code Quality
✓ GOOD: Clean separation of concerns
✓ GOOD: Comprehensive test coverage (94%)
⚠️ MINOR: Missing JSDoc for exported functions

## Architecture
✓ Follows existing patterns
✓ Proper error handling

## Suggestions
1. Add input validation for email field
2. Consider caching user lookup
3. Extract magic numbers to constants

## Verdict
✅ APPROVE with minor changes requested
```

## Ralph Loop - Autonomous Execution

Based on Geoffrey Huntley's "Ralph Wiggum" technique - self-referential loop that iterates until ALL validations pass.

### `/prp-ralph` - Start Autonomous Loop

```bash
/prp-ralph .claude/PRPs/plans/add-user-auth.plan.md --max-iterations 20
```

**Process**:
1. Implements plan tasks
2. Runs ALL validation commands
3. If any fail → analyzes, fixes, re-validates
4. Repeats until ALL pass
5. Outputs `<promise>COMPLETE</promise>`
6. Exits gracefully

**State Tracking** (`.claude/prp-ralph.state.md`):
```markdown
# Ralph Loop State

Plan: .claude/PRPs/plans/add-user-auth.plan.md
Max Iterations: 20
Current Iteration: 7

## Last Validation Results
✓ npm run type-check
✗ npm run lint (3 errors)
✓ npm test
✗ npm run build (compilation error)

## Current Focus
Fixing lint errors in src/auth/jwt.ts
Resolving build error in import path

## Learnings
- Arrow functions need explicit return types
- Barrel imports must use .js extension
```

### `/prp-ralph-cancel` - Stop Ralph Loop

```bash
/prp-ralph-cancel
```

Removes state file, allowing stop hook to exit gracefully.

## Configuration

### CLAUDE.md - Project Guidelines

Create `CLAUDE.md` in your project root:

```markdown
# Project Context for Claude Code

## Tech Stack
- Node.js 20
- TypeScript 5.3
- Express 4.18
- PostgreSQL 15

## Code Patterns

### File Structure
src/
  features/
    <feature>/
      <feature>.controller.ts
      <feature>.service.ts
      <feature>.model.ts
      <feature>.test.ts

### Naming Conventions
- Controllers: `XController`
- Services: `XService`
- Models: `XModel`
- Files: kebab-case

### Error Handling
```typescript
try {
  await service.method();
} catch (error) {
  logger.error('Context', { error });
  throw new AppError('User message', 500);
}
```

### Testing
- Use Jest
- Mock external dependencies
- Test edge cases

## Validation Commands
npm run type-check
npm run lint
npm test
npm run build

## Environment Variables
DATABASE_URL=${DATABASE_URL}
JWT_SECRET=${JWT_SECRET}
SMTP_HOST=${SMTP_HOST}
```

### PRP Templates

**Base Template** (`PRPs/templates/prp_base.md`):
```markdown
# [Feature Name]

## Context
### Current State
- Describe existing implementation
- List relevant files

### Goal
- What we're building
- Why it's needed

## Technical Approach
### Architecture
- Component structure
- Data flow

### Implementation
1. Step 1
2. Step 2

## Validation
```bash
npm run type-check
npm test
```

## Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2
```

## Real-World Examples

### Example 1: Adding API Pagination

**Step 1: Create Plan**
```bash
/prp-plan "add cursor-based pagination to /api/users endpoint"
```

**Generated Plan**:
```markdown
# Implementation Plan: API Users Pagination

## Context
- File: `src/features/users/users.controller.ts`
- Current: Returns all users (no pagination)
- Pattern: Other endpoints use `PaginationParams` from `src/utils/pagination.ts`

## Tasks
1. [ ] Update UsersController.list() signature
   - Add `@Query() params: PaginationParams`
   - Return `PaginatedResponse<User>`

2. [ ] Modify UsersService.findAll()
   - Accept cursor and limit params
   - Use cursor-based query
   - Return hasMore flag

3. [ ] Add tests
   - Test pagination edge cases
   - Test cursor validation

## Files to Modify
- src/features/users/users.controller.ts
- src/features/users/users.service.ts
- src/features/users/users.test.ts

## Validation Commands
npm run type-check
npm run lint
npm test -- users
npm run build

## Success Criteria
- [ ] Endpoint accepts cursor and limit params
- [ ] Returns paginated results with hasMore
- [ ] All tests pass
- [ ] No type errors
```

**Step 2: Implement with Ralph**
```bash
/prp-ralph .claude/PRPs/plans/api-users-pagination.plan.md --max-iterations 10
```

Ralph will:
1. Update controller, service, tests
2. Run validations
3. Fix any type errors or test failures
4. Re-validate until all pass
5. Complete autonomously

### Example 2: Bug Fix Workflow

**GitHub Issue #789**:
```
Title: Email notifications not sent for password reset
Labels: bug, priority:high

Users report not receiving password reset emails. 
Checked spam folders - emails aren't being sent at all.
```

**Step 1: Investigate**
```bash
/prp-issue-investigate 789
```

**Generated Investigation**:
```markdown
# Issue Investigation: #789 - Email notifications not sent

## Issue Summary
Password reset emails not being sent to users

## Code Analysis
File: `src/features/auth/auth.service.ts`

```typescript
async resetPassword(email: string) {
  const token = generateToken();
  await this.userRepo.saveResetToken(email, token);
  
  // BUG: emailService.send() is async but not awaited
  this.emailService.send({
    to: email,
    subject: 'Password Reset',
    template: 'reset-password',
    data: { token }
  });
  
  return { success: true };
}
```

## Root Cause
Email sending is fire-and-forget. If it fails, no error is thrown and method returns success.

## Proposed Solution
1. Await email sending
2. Add error handling
3. Add retry logic for transient failures

## Implementation Plan
- [ ] Add `await` to emailService.send()
- [ ] Wrap in try-catch with specific error
- [ ] Add integration test for email sending
- [ ] Update resetPassword to return email status

## Validation
npm test -- auth.service
npm run type-check
```

**Step 2: Fix**
```bash
/prp-issue-fix 789
```

**Step 3: Commit & PR**
```bash
/prp-commit "fix password reset email sending" --files auth
/prp-pr "Fix password reset email sending" --closes 789
```

### Example 3: Large Feature with Phases

**Step 1: Create PRD**
```bash
/prp-prd "real-time chat system with websockets"
```

**Generated PRD** (excerpt):
```markdown
# PRD: Real-time Chat System

## Implementation Phases

| # | Phase | Description | Status | Parallel | Depends | PRP Plan |
|---|-------|-------------|--------|----------|---------|----------|
| 1 | Infrastructure | WebSocket server setup | pending | - | - | - |
| 2 | Data Model | Chat schema & migrations | pending | - | 1 | - |
| 3 | Backend API | Message CRUD endpoints | pending | - | 2 | - |
| 4 | WebSocket Handlers | Real-time event handlers | pending | - | 3 | - |
| 5 | Frontend UI | Chat interface | pending | with 6 | 4 | - |
| 6 | Frontend State | Redux/state management | pending | with 5 | 4 | - |
| 7 | Tests | E2E and unit tests | pending | - | 5,6 | - |
```

**Step 2: Implement Phase 1**
```bash
/prp-plan .claude/PRPs/prds/real-time-chat-system.prd.md
# Selects Phase 1 (first pending with no dependencies)

/prp-ralph .claude/PRPs/plans/real-time-chat-phase-1.plan.md --max-iterations 15
```

**Step 3: Repeat for Each Phase**
```bash
/prp-plan .claude/PRPs/prds/real-time-chat-system.prd.md  # Phase 2
/prp-ralph .claude/PRPs/plans/real-time-chat-phase-2.plan.md

# ... continue through all phases
```

**Step 4: Parallel Phases (5 & 6)**

Use git worktrees for concurrent development:

```bash
# Current branch has phases 1-4 complete
git worktree add -b phase-5-ui ../chat-phase-5
git worktree add -b phase-6-state ../chat-phase-6

# Terminal 1
cd ../chat-phase-5
/prp-plan .claude/PRPs/prds/real-time-chat-system.prd.md
/prp-ralph .claude/PRPs/plans/real-time-chat-phase-5.plan.md

# Terminal 2
cd ../chat-phase-6
/prp-plan .claude/PRPs/prds/real-time-chat-system.prd.md
/prp-ralph .claude/PRPs/plans/real-time-chat-phase-6.plan.md

# After both complete, merge both branches
```

## Common Patterns

### Pattern 1: Iterative Refinement

```bash
# Create initial plan
/prp-plan "add file upload feature"

# Review generated plan, add details to plan file manually
# Then implement
/prp-implement .claude/PRPs/plans/add-file-upload.plan.md
```

### Pattern 2: Plan-Review-Implement

```bash
# Generate plan
/prp-plan "refactor user service to use repository pattern"

# Review plan artifact, discuss with team
# Modify plan file if needed
# Then execute with validation
/prp-ralph .claude/PRPs/plans/refactor-user-service.plan.md
```

### Pattern 3: Investigation-Driven Development

```bash
# Investigate issue first
/prp-issue-investigate 456

# Review investigation
# Edit investigation artifact to add context
# Then fix
/prp-issue-fix 456
```

### Pattern 4: Validation-First Planning

When creating plans, always include comprehensive validation:

```markdown
## Validation Commands
# Type safety
npm run type-check

# Code quality
npm run lint
npm run format:check

# Functionality
npm test
npm run test:integration
npm run test:e2e

# Build
npm run build

# Runtime
npm run dev  # Manual verification checklist
```

## Troubleshooting

### Ralph Loop Not Stopping

**Symptom**: Ralph continues after `<promise>COMPLETE</promise>`

**Solution**:
```bash
# Verify hook is configured
cat .claude/settings.local.json

# Verify hook script exists and is executable
ls -la .claude/hooks/prp-ralph-stop.sh
chmod +x .claude/hooks/prp-ralph-stop.sh

# Manual cancel
/prp-ralph-cancel
```

### Plans Not Archiving

**Symptom**: Completed plans stay in `plans/` directory

**Solution**:
Ensure `.claude/PRPs/plans/completed/` directory exists:
```bash
mkdir -p .claude/PRPs/plans/completed
```

### Validation Commands Fail

**Symptom**: Ralph loop gets stuck on failing validation

**Solution 1**: Check validation commands are correct
```bash
# Test commands manually
npm run type-check
npm run lint
npm test
```

**Solution 2**: Add more specific validation
```markdown
## Validation Commands
npm run type-check
npm run lint -- --fix  # Auto-fix lint issues
npm test -- --testPathPattern=users  # Only relevant tests
```

### PRD Phase Selection Issues

**Symptom**: `/prp-plan` doesn't auto-select next phase

**Solution**: Verify PRD table format is exact:
```markdown
| # | Phase | Description | Status | Parallel | Depends | PRP Plan |
|---|-------|-------------|--------|----------|---------|----------|
```

Status must be exactly: `pending`, `in-progress`, or `complete`

### Missing Context in Plans

**Symptom**: Plans lack necessary codebase context

**Solution**: Enhance `CLAUDE.md` with more patterns:
```markdown
## Code Patterns

### Authentication
Always use `AuthMiddleware.verify()` before protected routes

### Database Queries
Use repository pattern from `src/repositories/base.repository.ts`

### Error Responses
```typescript
throw new AppError(
  'User-facing message',
  HttpStatus.BAD_REQUEST,
  { details: 'debug info' }
);
```
```

### Ralph Exceeds Max Iterations

**Symptom**: Ralph hits max iterations without completing

**Solution 1**: Increase iterations
```bash
/prp-ralph plan.md --max-iterations 50
```

**Solution 2**: Break plan into smaller chunks
```bash
# Instead of one large plan, create multiple focused plans
/prp-plan "add user model and migration"
/prp-plan "add user service layer"
/prp-plan "add user API endpoints"
```

## Best Practices

### 1. Context is King
Provide maximum relevant context in every artifact:
- File paths (exact)
- Existing code patterns
- Dependencies and versions
- Related documentation

### 2. Validation Loops
Every plan must have executable validation commands:
```markdown
## Validation Commands
npm run type-check
npm run lint
npm test
npm run build
npm run e2e  # if applicable
```

### 3. Bounded Scope
Each plan should be completable in one Ralph loop (< 20 iterations):
- Small: Single function/component
- Medium: Feature with tests
- Large: Use PRD phases

### 4. Information Density
Use keywords from codebase, not generic terms:
- ✅ "Use `UserRepository.findByEmail()`"
- ❌ "Query database for user"

### 5. Git Hygiene
Commit after each completed plan:
```bash
/prp-implement plan.md
/prp-commit "descriptive message" --files "relevant-keywords"
```

### 6. Review Before Merge
Always review implementation reports:
```bash
cat .claude/PRPs/reports/feature-name.report.md
# Review changes, validation results, challenges
```

### 7. Curate AI Documentation
Keep `PRPs/ai_docs/` updated with relevant library docs:
```
PRPs/ai_docs/
├── express-best-practices.md
├── typescript-patterns.md
├── jest-testing.md
└── postgresql-queries.md
```

### 8. Template Customization
Adapt templates to your project needs:
```bash
# Create project-specific template
cp PRPs/templates/prp_base.md PRPs/templates/prp_api_endpoint.md
# Edit to include API-specific patterns
```

## Advanced Usage

### Custom Subagents

Create specialized agents in `.claude/agents/`:

```markdown
# .claude/agents/database-expert.md

You are a database optimization expert.

When analyzing queries:
1. Check for N+1 issues
2. Verify indexes exist
3. Suggest query optimizations
4. Estimate query performance

Use EXPLAIN ANALYZE for all queries.
```

Reference in plans:
```markdown
## Special Instructions
@database-expert review all queries in this implementation
```

### PRD Templates for Domains

```markdown
# PRPs/templates/prp_prd_api.md

# API Feature PRD Template

## API Specification
### Endpoints
- `GET /api/resource`
- `POST /api/resource`

### Request/Response Schemas
```typescript
interface CreateResourceRequest {
  name: string;
  description: string;
}
```

### Authentication
- Required: JWT token
- Permissions: `resource:create`

## Implementation Phases
| # | Phase | Description | Status | Parallel | Depends | PRP Plan |
|---|-------|-------------|--------|----------|---------|----------|
| 1 | Schema | Database schema | pending | - | - | - |
| 2 | Repository | Data access layer | pending | - | 1 | - |
| 3 | Service | Business logic | pending | - | 2 | - |
| 4 | Controller | API endpoints | pending | - | 3 | - |
| 5 | Tests | Integration tests | pending | - | 4 | - |
```

### Metrics and Observability

Add to validation commands:
```markdown
## Validation Commands
npm run type-check
npm run lint
npm test
npm run build

## Performance Validation
npm run benchmark -- feature-name
npm run lighthouse -- http://localhost:3000/feature

## Observability
- Check logs for errors
- Verify metrics exported
- Confirm traces captured
```

## Integration with CI/CD

Create validation script `.github/workflows/prp-validate.yml`:

```yaml
name: PRP Validation

on:
  pull_request:
    paths:
      - '.claude/PRPs/plans/**'
      - '.claude/PRPs/reports/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run PRP validations
        run: |
          npm run type-check
          npm run lint
          npm test
          npm run build
      
      - name: Check for incomplete plans
        run: |
          if ls .claude/PRPs/plans/*.plan.md 2>/dev/null; then
            echo "❌ Incomplete plans found - all plans should be archived"
            exit 1
          fi
```

---

**The goal is one-pass implementation success through comprehensive context.**

For support and workshops: https://www.rasmuswiding.com/
