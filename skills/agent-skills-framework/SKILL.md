---
name: agent-skills-framework
description: Production-grade engineering skills for AI coding agents - lifecycle commands, workflow automation, and best practices for software development.
triggers:
  - "how do I use agent skills"
  - "install addy's agent skills"
  - "setup production engineering skills"
  - "use /spec or /plan or /build commands"
  - "configure agent skills for my IDE"
  - "what skills are available"
  - "activate agent skills workflow"
  - "setup engineering best practices for AI agents"
---

# Agent Skills Framework

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A comprehensive framework of 23 production-grade engineering skills that guide AI coding agents through the complete software development lifecycle. Skills encode workflows, quality gates, and best practices that senior engineers use, packaged for consistent agent execution.

## What It Does

Agent Skills provides:

- **7 slash commands** mapping to development phases (`/spec`, `/plan`, `/build`, `/test`, `/review`, `/code-simplify`, `/ship`)
- **23 structured workflows** covering everything from idea refinement to production deployment
- **Auto-activation** based on context (API design triggers `api-and-interface-design`, UI work triggers `frontend-ui-engineering`)
- **Quality gates** with verification steps, anti-rationalization tables, and "STOP" conditions
- **Agent personas** for specialized reviews (code-reviewer, test-engineer, security-auditor)
- **Reference checklists** for testing, security, performance, and accessibility

## Installation

### Claude Code (Recommended)

**Via Marketplace:**

```bash
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

**If SSH fails, use HTTPS:**

```bash
/plugin marketplace add https://github.com/addyosmani/agent-skills.git
/plugin install agent-skills@addy-agent-skills
```

**Local Development:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

### Cursor

Copy individual `SKILL.md` files or the entire `skills/` directory into `.cursor/rules/`:

```bash
# Clone the repo
git clone https://github.com/addyosmani/agent-skills.git

# Copy all skills
cp -r agent-skills/skills/* .cursor/rules/

# Or copy specific skills
cp agent-skills/skills/spec-driven-development/SKILL.md .cursor/rules/
```

### Gemini CLI

**Install from GitHub:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**Install from local clone:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install ./agent-skills/skills/
```

### Windsurf

Add skill contents to `.windsurf/rules.md`:

```bash
# Append skills to your rules file
cat agent-skills/skills/*/SKILL.md >> .windsurf/rules.md
```

### OpenCode

Uses `AGENTS.md` and the `skill` tool for agent-driven execution:

```bash
# Copy the agents configuration
cp agent-skills/AGENTS.md .

# Skills auto-discovered from skills/ directory
```

### GitHub Copilot

Use agent personas as Copilot personas and add skills to `.github/copilot-instructions.md`:

```bash
# Copy agent definitions
cp agent-skills/agents/* .github/copilot-agents/

# Add skill content
cat agent-skills/skills/*/SKILL.md >> .github/copilot-instructions.md
```

### Kiro IDE & CLI

Skills stored under `.kiro/skills/` at project or global level:

```bash
# Copy to project-level skills
mkdir -p .kiro/skills
cp -r agent-skills/skills/* .kiro/skills/
```

### Any Other Agent

Skills are plain Markdown. Copy to your agent's instruction/context directory:

```bash
# Generic approach
cp -r agent-skills/skills/ /path/to/your/agent/context/
```

## Core Commands

### `/spec` - Spec Before Code

Define what to build before writing code. Activates `spec-driven-development`:

```markdown
/spec

I need to build a URL shortener API with rate limiting
```

**Output:** PRD covering objectives, commands, structure, code style, testing strategy, and boundaries.

### `/plan` - Small, Atomic Tasks

Break specs into implementable units. Activates `planning-and-task-breakdown`:

```markdown
/plan

Break down the URL shortener spec into tasks
```

**Output:** Ordered tasks with acceptance criteria, dependencies, and size estimates.

### `/build` - One Slice at a Time

Implement incrementally with feature flags and safe defaults. Activates `incremental-implementation`:

```markdown
/build

Implement task #1: URL shortening endpoint
```

**Output:** Code + tests for one thin vertical slice, with feature flag wrapper.

### `/test` - Tests Are Proof

Red-Green-Refactor TDD cycle. Activates `test-driven-development`:

```markdown
/test

Write tests for URL validation logic
```

**Output:** Test file following test pyramid (80% unit, 15% integration, 5% E2E).

### `/review` - Improve Code Health

Five-axis code review with severity labels. Activates `code-review-and-quality`:

```markdown
/review

Review the URL shortener PR
```

**Output:** Structured feedback on correctness, maintainability, security, performance, testing.

### `/code-simplify` - Clarity Over Cleverness

Reduce complexity while preserving behavior. Activates `code-simplification`:

```markdown
/code-simplify

Simplify the rate limiting middleware
```

**Output:** Refactored code with change justification and test confirmation.

### `/ship` - Faster Is Safer

Pre-launch checklist and staged rollout. Activates `shipping-and-launch`:

```markdown
/ship

Prepare URL shortener for production
```

**Output:** Deployment plan, monitoring setup, rollback procedure, feature flag lifecycle.

## Key Skills Reference

### Meta: Discover Which Skill Applies

**`using-agent-skills`** - Maps incoming work to the right skill:

```markdown
User: "I want to add authentication to my app"

Agent (activates using-agent-skills):
→ Detects: unclear spec + security concern
→ Activates: interview-me → spec-driven-development → security-and-hardening
```

### Define Phase

**`interview-me`** - One-question-at-a-time extraction until ~95% confidence:

```markdown
Trigger: "interview me about the auth feature"

Agent:
Q1: What authentication method? (OAuth, JWT, sessions, magic links)
[waits for answer]
Q2: Which providers? (Google, GitHub, email, all three)
[continues until clear]
```

**`spec-driven-development`** - PRD before code:

```markdown
Activates when: Starting new project/feature

Output structure:
## Objectives
- User needs
- Success criteria

## Commands & Usage
- CLI/API surface

## Structure
- File organization
- Module boundaries

## Code Style & Patterns
- Framework decisions
- State management

## Testing Strategy
- Coverage targets
- Test types

## Boundaries & Constraints
- What's in/out of scope
```

### Plan Phase

**`planning-and-task-breakdown`** - Decompose specs into tasks:

```markdown
Input: PRD for URL shortener

Output:
Task 1: URL shortening endpoint
  Acceptance: POST /shorten returns short code
  Size: Small (~50 lines)
  Depends on: none

Task 2: Redirect handler
  Acceptance: GET /:code redirects to original URL
  Size: Small (~30 lines)
  Depends on: Task 1

Task 3: Rate limiting middleware
  Acceptance: 429 after 100 req/min
  Size: Medium (~100 lines)
  Depends on: Task 1
```

### Build Phase

**`incremental-implementation`** - Thin vertical slices:

```bash
# Pattern for each task
1. Feature flag wrapper (if multi-step)
2. Minimal implementation
3. Tests (Red-Green-Refactor)
4. Verify locally
5. Atomic commit
6. Move to next slice
```

**Example commit sequence:**

```bash
git commit -m "feat: Add URL shortening endpoint

- POST /shorten accepts URL, returns short code
- Feature flag: ENABLE_URL_SHORTENER (default: true)
- Tests: valid URL, invalid URL, duplicate URL
- Safe default: returns 503 if feature disabled"

git commit -m "feat: Add redirect handler

- GET /:code redirects to original URL
- 404 for unknown codes
- Tests: valid code, invalid code, expired code"
```

**`test-driven-development`** - Red-Green-Refactor:

```javascript
// Step 1: RED - Write failing test
describe('URL shortener', () => {
  test('generates unique short codes', () => {
    const code1 = generateShortCode('https://example.com');
    const code2 = generateShortCode('https://example.com');
    expect(code1).toHaveLength(6);
    expect(code2).toHaveLength(6);
    expect(code1).not.toBe(code2); // ❌ FAILS - not implemented
  });
});

// Step 2: GREEN - Minimal implementation
function generateShortCode(url) {
  return crypto.randomBytes(3).toString('base64url');
}
// ✅ PASSES

// Step 3: REFACTOR - Improve without breaking
function generateShortCode(url) {
  const hash = crypto.createHash('sha256').update(url).digest();
  const timestamp = Date.now().toString(36);
  return (hash.toString('base64url') + timestamp).slice(0, 6);
}
// ✅ STILL PASSES
```

**`source-driven-development`** - Ground decisions in official docs:

```markdown
User: "Add Redis caching to the URL shortener"

Agent (activates source-driven-development):
1. Fetch Redis docs: https://redis.io/docs/latest/develop/connect/clients/nodejs/
2. Verify connection pattern from official source
3. Implement with source citation

// Citation in code:
// Pattern from https://redis.io/docs/latest/develop/connect/clients/nodejs/
// Retrieved: 2026-05-16
const redis = require('redis');
const client = redis.createClient({
  socket: { host: process.env.REDIS_HOST, port: 6379 }
});
```

**`doubt-driven-development`** - Adversarial review for high-stakes decisions:

```markdown
Trigger: Production security change, unfamiliar code, irreversible migration

Process:
1. CLAIM: "This JWT expiration is secure"
2. EXTRACT: ttl = 86400 (24 hours)
3. DOUBT: "24h is long for sensitive data; OWASP recommends 15min for access tokens"
4. RECONCILE: Change to 900s (15min) + refresh token pattern
5. STOP: Present change with justification
```

### Verify Phase

**`browser-testing-with-devtools`** - Live runtime data via Chrome DevTools MCP:

```bash
# Activate DevTools connection
chrome-devtools connect http://localhost:3000

# Inspect DOM
query-selector 'button[data-testid="submit"]'

# Check console errors
get-console-logs --level error

# Measure performance
performance-profile --duration 5000

# Network waterfall
get-network-log --filter fetch
```

**`debugging-and-error-recovery`** - Five-step triage:

```markdown
1. REPRODUCE
   - Minimal repro case
   - Consistent failure conditions

2. LOCALIZE
   - Binary search through call stack
   - Isolate failing component

3. REDUCE
   - Strip non-essential code
   - Minimal failing example

4. FIX
   - Root cause, not symptom
   - Safe fallback if fix unclear

5. GUARD
   - Add test for regression
   - Update error handling
```

### Review Phase

**`code-review-and-quality`** - Five-axis review:

```markdown
Reviewing: URL shortener rate limiting PR

✅ CORRECTNESS
- Logic handles edge cases (empty rate limit window)

⚠️ MAINTAINABILITY (Optional)
- Extract magic number 100 to config constant

✅ SECURITY
- Rate limit applied per IP, prevents abuse

📊 PERFORMANCE (FYI)
- Redis lookup adds 2ms latency, acceptable for use case

✅ TESTING
- Unit tests for rate limit logic
- Integration test for 429 response
- Missing: E2E test for reset after window expires (Nit)

SIZE: 87 lines ✅ (target: ~100)
```

**`code-simplification`** - Chesterton's Fence + Rule of 500:

```javascript
// BEFORE (complexity: 12, 500+ line file)
function processUrl(url, options = {}) {
  const { validate = true, transform = true, cache = true } = options;
  if (validate && !isValidUrl(url)) throw new Error('Invalid URL');
  let processed = url;
  if (transform) {
    processed = normalizeUrl(processed);
    processed = removeTracking(processed);
    processed = enforceHttps(processed);
  }
  if (cache) {
    const cached = getCache(processed);
    if (cached) return cached;
  }
  const result = shorten(processed);
  if (cache) setCache(processed, result);
  return result;
}

// AFTER (complexity: 4, extracted to modules)
function processUrl(url) {
  const validated = validateUrl(url);      // url-validator.js
  const normalized = normalizeUrl(validated); // url-normalizer.js
  return cachedShorten(normalized);        // url-cache.js
}
```

### Ship Phase

**`git-workflow-and-versioning`** - Atomic commits, trunk-based:

```bash
# Commit pattern
git commit -m "type(scope): description

- Detail 1
- Detail 2
- Detail 3

[Tests: unit, integration]
[Refs: #123]"

# Example
git commit -m "feat(api): Add rate limiting middleware

- Redis-backed rate limiter (100 req/min per IP)
- Configurable via RATE_LIMIT_MAX env var
- Returns 429 with Retry-After header

[Tests: unit, integration]
[Refs: #456]"

# Keep changes small (~100 lines)
git diff --stat
# 3 files changed, 94 insertions(+), 12 deletions(-)
```

**`shipping-and-launch`** - Pre-launch checklist:

```markdown
## Pre-Launch Checklist

### Code Quality
- [ ] All tests passing (unit, integration, E2E)
- [ ] Code review approved by 2+ engineers
- [ ] No critical security vulnerabilities (npm audit)
- [ ] Performance regression test passed

### Configuration
- [ ] Feature flags configured (ENABLE_URL_SHORTENER=false initially)
- [ ] Environment variables documented (.env.example)
- [ ] Secrets rotated for production (REDIS_PASSWORD, JWT_SECRET)

### Observability
- [ ] Logging configured (structured JSON)
- [ ] Metrics exported (request count, latency, error rate)
- [ ] Alerts defined (error rate >1%, p99 latency >500ms)
- [ ] Dashboards created (Grafana/Datadog)

### Rollback Plan
- [ ] Rollback script tested (./scripts/rollback.sh)
- [ ] Database migrations reversible
- [ ] Feature flag kill switch documented

### Staged Rollout
1. Deploy to staging (1 hour soak test)
2. Enable for internal users (10% traffic, 24 hours)
3. Gradual rollout (25% → 50% → 100% over 1 week)
```

## Agent Personas

Pre-configured specialists for targeted reviews:

### Code Reviewer (Senior Staff Engineer)

```bash
# Activate in Claude Code
/plugin use code-reviewer

# In Cursor, reference in chat
@code-reviewer Review this PR
```

**Review standard:** "Would a staff engineer approve this?"

**Five axes:** Correctness, Maintainability, Security, Performance, Testing

**Severity labels:**
- `Nit` - Polish, not blocking
- `Optional` - Suggested improvement
- `FYI` - Informational

### Test Engineer (QA Specialist)

```bash
/plugin use test-engineer

# Evaluates test strategy
```

**Focus:**
- Test pyramid adherence (80/15/5)
- Coverage gaps
- Flakiness detection
- Prove-It pattern compliance

### Security Auditor

```bash
/plugin use security-auditor

# OWASP Top 10 assessment
```

**Checks:**
- Input validation (SQL injection, XSS)
- Authentication/authorization
- Secrets management
- Dependency vulnerabilities
- CORS/CSP headers

## Reference Checklists

### Testing Patterns

```javascript
// Test naming convention
describe('[Unit] URL shortener', () => {
  test('generates 6-character codes for valid URLs', () => {
    // ARRANGE
    const url = 'https://example.com';
    
    // ACT
    const code = generateShortCode(url);
    
    // ASSERT
    expect(code).toHaveLength(6);
    expect(code).toMatch(/^[a-zA-Z0-9_-]{6}$/);
  });

  test('throws error for invalid URLs', () => {
    // ACT & ASSERT
    expect(() => generateShortCode('not-a-url')).toThrow('Invalid URL');
  });
});

// Integration test
describe('[Integration] URL shortener API', () => {
  test('POST /shorten → GET /:code roundtrip', async () => {
    const response = await request(app)
      .post('/shorten')
      .send({ url: 'https://example.com' });
    
    const { code } = response.body;
    
    const redirect = await request(app).get(`/${code}`);
    expect(redirect.status).toBe(302);
    expect(redirect.headers.location).toBe('https://example.com');
  });
});

// E2E test
describe('[E2E] URL shortener user flow', () => {
  test('user shortens URL and visits short link', async () => {
    await page.goto('http://localhost:3000');
    await page.fill('input[name="url"]', 'https://example.com');
    await page.click('button[type="submit"]');
    
    const shortUrl = await page.textContent('.short-url');
    await page.goto(shortUrl);
    
    expect(page.url()).toBe('https://example.com');
  });
});
```

### Security Checklist

```markdown
## Pre-Commit Security Checks

- [ ] No hardcoded secrets (use process.env.SECRET_NAME)
- [ ] All user input validated (schema validation with Zod/Joi)
- [ ] SQL queries parameterized (no string concatenation)
- [ ] HTML output escaped (use templating engine auto-escaping)
- [ ] Authentication required for sensitive endpoints
- [ ] Authorization checks before data access
- [ ] HTTPS enforced in production (HSTS header)
- [ ] CORS configured restrictively (whitelist origins)
- [ ] CSP header set (no unsafe-inline)
- [ ] Dependencies scanned (npm audit fix)
```

### Performance Checklist

```markdown
## Core Web Vitals Targets

- LCP (Largest Contentful Paint): < 2.5s
- FID (First Input Delay): < 100ms
- CLS (Cumulative Layout Shift): < 0.1

## Backend Performance

- [ ] Database queries indexed (EXPLAIN ANALYZE)
- [ ] N+1 queries eliminated (use eager loading)
- [ ] Response payloads gzipped
- [ ] HTTP caching headers set (Cache-Control, ETag)
- [ ] Rate limiting configured
- [ ] Connection pooling enabled (database, Redis)

## Frontend Performance

- [ ] Code splitting configured (lazy load routes)
- [ ] Images optimized (WebP, responsive sizes)
- [ ] Fonts subsetted and preloaded
- [ ] Third-party scripts deferred
- [ ] Bundle size < 200KB gzipped
- [ ] Lighthouse score > 90
```

### Accessibility Checklist

```markdown
## WCAG 2.1 AA Compliance

- [ ] Keyboard navigation (Tab, Enter, Esc work)
- [ ] Focus indicators visible (outline on :focus)
- [ ] Screen reader labels (aria-label, aria-labelledby)
- [ ] Color contrast ratio ≥ 4.5:1 (text) / 3:1 (large text)
- [ ] Alt text for images (descriptive, < 125 chars)
- [ ] Form inputs labeled (<label> or aria-label)
- [ ] Error messages announced (aria-live="assertive")
- [ ] Semantic HTML (nav, main, article, button vs div)
- [ ] Skip to main content link
- [ ] Tested with screen reader (NVDA, VoiceOver)
```

## Configuration

### Skill Activation Rules

Skills auto-activate based on context. Customize in `using-agent-skills/SKILL.md`:

```yaml
# Trigger patterns
api_design:
  - designing API
  - creating endpoints
  - REST interface
  - GraphQL schema

frontend:
  - building UI
  - React component
  - CSS styling
  - accessibility

security:
  - authentication
  - authorization
  - user input
  - storing passwords
```

### Command Aliases

Add custom aliases in your agent config:

```bash
# Claude Code
/plugin config agent-skills --alias "review-pr=/review"
/plugin config agent-skills --alias "new-feature=/spec then /plan then /build"

# Cursor (.cursor/rules/aliases.md)
When user says "review this PR", activate /review
When user says "start new feature", activate /spec → /plan → /build
```

### Quality Gate Thresholds

Customize in skill frontmatter:

```yaml
# code-review-and-quality/SKILL.md
review_gates:
  max_change_size: 100  # lines
  required_approvals: 2
  min_test_coverage: 80  # percent
  
# performance-optimization/SKILL.md
performance_targets:
  lcp: 2500  # ms
  fid: 100   # ms
  cls: 0.1
  bundle_size: 200  # KB gzipped
```

## Common Patterns

### Full Lifecycle Example

```markdown
# User starts a new feature

User: "I want to add a comment system to my blog"

Agent (activates using-agent-skills):
→ Underspecified, activates interview-me

Agent: "What kind of comments? (Nested threads, flat list, reactions-only)"
User: "Nested threads with reactions"

Agent: "Authentication required? (Login required, anonymous allowed, optional)"
User: "Login required"

Agent: "Moderation? (Pre-approval, post-reporting, auto-filter)"
User: "Post-reporting with flag button"

# ~95% confidence reached, switches to spec-driven-development

Agent: "Here's your PRD for the comment system..."

[PRD generated with objectives, API surface, data model, testing strategy]

User: "/plan"

Agent (activates planning-and-task-breakdown):
Task 1: Comment data model (User, Comment, Reaction tables)
Task 2: POST /comments endpoint (create comment)
Task 3: GET /comments/:postId endpoint (fetch thread)
Task 4: POST /comments/:id/reactions (add reaction)
Task 5: POST /comments/:id/report (flag for moderation)
...

User: "/build task 1"

Agent (activates incremental-implementation + test-driven-development):
1. Writes failing test for Comment.create()
2. Implements minimal schema
3. Tests pass
4. git commit -m "feat(comments): Add Comment data model"

[Repeats for each task]

User: "/review"

Agent (activates code-review-and-quality):
Reviewing: Comment system PR (347 lines - split recommended)

✅ CORRECTNESS: Thread nesting logic correct
⚠️ SECURITY (Required): Missing rate limit on POST /comments
✅ TESTING: 82% coverage, missing E2E test for report flow

Recommendation: Split into 2 PRs (data model + API, then reporting)

User: "/ship"

Agent (activates shipping-and-launch):
Pre-launch checklist:
- [ ] Feature flag ENABLE_COMMENTS=false in prod
- [ ] Rate limiting configured (10 comments/min)
- [ ] Monitoring for comment_created events
- [ ] Rollback script tested

Rollout plan:
1. Deploy with flag off (1 hour soak)
2. Enable for beta users (10%, 48 hours)
3. Gradual rollout (25% → 50% → 100% over 1 week)
```

### Doubt-Driven Review for High-Stakes Change

```markdown
User: "Migrate user passwords from MD5 to bcrypt"

Agent (activates doubt-driven-development - irreversible migration):

CLAIM: "This migration script safely rehashes all passwords"

EXTRACT:
- Reads user table (10M rows)
- Iterates, rehashing MD5 → bcrypt
- Updates in place

DOUBT:
1. No rollback plan (bcrypt hashes can't reverse to MD5)
2. Table lock during migration (10M rows = ~2 hour downtime)
3. No password verification after rehash
4. Missing feature flag for new hash check

RECONCILE:
Alternative: Lazy migration
1. Add `password_hash_version` column
2. Check version on login
3. If MD5, verify then rehash to bcrypt
4. Update version to 'bcrypt'
5. No downtime, automatic rollback (keep MD5 as fallback)

STOP:
Present both approaches with tradeoff analysis
→ Recommend lazy migration for safety
```

### Source-Driven API Implementation

```markdown
User: "Add Stripe payment processing"

Agent (activates source-driven-development):

1. Fetch official docs: https://stripe.com/docs/api
2. Verify Node.js SDK patterns
3. Extract canonical example

Implementation (with citations):

// Pattern from https://stripe.com/docs/api/payment_intents/create
// Retrieved: 2026-05-16
// SDK version: stripe@14.0.0
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

async function createPayment(amount, currency) {
  // Verified: amount in smallest currency unit (cents for USD)
  // Source: https://stripe.com/docs/currencies#zero-decimal
  const paymentIntent = await stripe.paymentIntents.create({
    amount,
    currency,
    automatic_payment_methods: { enabled: true },
  });
  
  return paymentIntent.client_secret;
}

// UNVERIFIED: Webhook signature verification pattern
// TODO: Consult https://stripe.com/docs/webhooks/signatures
// Current implementation is placeholder
function verifyWebhook(payload, signature) {
  // PLACEHOLDER - needs source verification
  return stripe.webhooks.constructEvent(payload, signature, process.env.STRIPE_WEBHOOK_SECRET);
}
```

## Troubleshooting

### Skills Not Activating

**Problem:** Commands like `/spec` don't trigger skills

**Solutions:**

```bash
# Claude Code: Verify installation
/plugin list
# Should show: agent-skills@addy-agent-skills

# Reinstall if missing
/plugin install agent-skills@addy-agent-skills

# Cursor: Check rules directory
ls .cursor/rules/
# Should contain SKILL.md files

# Copy if empty
cp -r /path/to/agent-skills/skills/* .cursor/rules/

# Gemini CLI: Verify skill installation
gemini skills list
# Should show: agent-skills (23 skills)

# Reinstall if missing
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

### Agent Ignoring Quality Gates

**Problem:** Agent rationalizes skipping tests or security checks

**Solution:** Anti-rationalization tables in skills enforce gates

```markdown
# Explicitly invoke the skill
User: "Use test-driven-development for this feature"

# Reference the anti-rationalization table
User: "Follow the 'No Rationalizing Away Tests' rule from TDD skill"

# Activate doubt-driven-development for adversarial review
User: "Use doubt-driven-development to review this security change"
```

### SSH Errors During Install

**Problem:** `git@github.com: Permission denied (publickey)`

**Solutions:**

```bash
# Option 1: Add SSH key to GitHub
ssh-keygen -t ed25519 -C "your_email@example.com"
cat ~/.ssh/id_ed25519.pub
# Add output to https://github.com/settings/keys

# Option 2: Use HTTPS instead
/plugin marketplace add https://github.com/addyosmani/agent-skills.git

# Option 3: Clone locally and install from path
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir ./agent-skills
```

### Large Changes Not Being Split

**Problem:** Agent creates 500+ line PRs

**Solution:** Enforce change sizing from `code-review-and-quality`:

```markdown
User: "Follow the ~100 line change size rule from code-review-and-quality skill"

Agent: "This feature requires 347 lines. Splitting into 3 PRs:
1. Data model + migrations (94 lines)
2. API endpoints (118 lines)
3. Frontend integration (135 lines)"
```

### Skills Conflicting

**Problem:** Multiple skills give contradictory advice

**Solution:** Hierarchy defined in `using-agent-skills`:

```markdown
Priority order:
1. Security (always wins)
2. Correctness (bugs block)
3. Testing (no code without tests)
4. Maintainability (long-term health)
5. Performance (optimize after working)

Example conflict:
- Performance skill: "Cache this database query"
- Security skill: "Don't cache user-specific data in shared cache"
→ Security wins, use per-user cache or skip caching
```

## Environment Variables

Skills reference but never include actual secrets:

```bash
# Database
DATABASE_URL=postgresql://localhost/myapp

# Redis
REDIS_HOST=localhost
REDIS_PASSWORD=<your-redis-password>

# Authentication
JWT_SECRET=<your-jwt-secret>
SESSION_SECRET=<your-session-secret>

# Third-party APIs
STRIPE_SECRET_KEY=<your-stripe-key>
STRIPE_WEBHOOK_SECRET=<your-webhook-secret>

# Feature Flags
ENABLE_URL_SHORTENER=true
ENABLE_COMMENTS=false

# Rate Limiting
RATE_LIMIT_MAX=100  # requests per window
RATE_LIMIT_WINDOW=60  # seconds

# Monitoring
SENTRY_DSN=<your-sentry-dsn>
DATADOG_API_KEY=<your-datadog-key>
```

## Best Practices

### Start with `/spec`

Never code without a spec:

```markdown
❌ Bad:
User: "Build a user dashboard"
Agent: *starts writing React components*

✅ Good:
User: "Build a user dashboard"
Agent (activates spec-driven-development):
"Let me create a spec first. What data should the dashboard show?"
```

### Use Feature Flags for Multi-Step Changes

```javascript
// Wrap incomplete features
if (process.env.ENABLE_NEW_DASHBOARD === 'true') {
  return <NewDashboard />;
}
return <LegacyDashboard />;

// Deploy with flag off, enable gradually
```

### Atomic Commits, Always

```bash
# Each commit is independently deployable
git log --oneline
abc123 feat(api): Add
