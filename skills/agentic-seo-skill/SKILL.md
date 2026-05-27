---
name: agentic-seo-skill
description: LLM-first SEO analysis skill with 16 sub-skills, 10 specialist agents, and 89 evidence collection scripts for comprehensive SEO audits
triggers:
  - analyze this website's SEO
  - run a full SEO audit
  - check technical SEO issues
  - optimize this page for search engines
  - generate SEO recommendations
  - validate schema markup
  - audit Core Web Vitals
  - create GitHub repository SEO report
---

# Agentic SEO Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

The Agentic SEO Skill is an LLM-first SEO analysis framework designed for AI coding assistants and agent IDEs. It provides 16 specialized sub-skills, 10 specialist agents, and 89 Python scripts for evidence-based SEO analysis. The skill follows a reasoning-first workflow: collect evidence, analyze with LLM proof, apply confidence labels, prioritize by impact, and produce actionable plans.

## Installation

### Quick Install (No Cloning Required)

**Linux/macOS:**
```bash
# Install to all supported IDEs
curl -fsSL https://raw.githubusercontent.com/Bhanunamikaze/Agentic-SEO-Skill/main/install.sh | bash -s -- --online

# Install to Claude Code only
curl -fsSL https://raw.githubusercontent.com/Bhanunamikaze/Agentic-SEO-Skill/main/install.sh | bash -s -- --online --target claude

# Install to specific project
curl -fsSL https://raw.githubusercontent.com/Bhanunamikaze/Agentic-SEO-Skill/main/install.sh | bash -s -- --online --target all --project-dir /path/to/project
```

**Windows (PowerShell 7+):**
```powershell
irm https://raw.githubusercontent.com/Bhanunamikaze/Agentic-SEO-Skill/main/install.ps1 -OutFile install.ps1
powershell -ExecutionPolicy Bypass -File .\install.ps1 --online
```

### From Source

```bash
git clone https://github.com/Bhanunamikaze/Agentic-SEO-Skill.git
cd Agentic-SEO-Skill

# Install to Claude Code with dependencies
bash install.sh --target claude --install-deps

# Install with Playwright for visual analysis
bash install.sh --target claude --install-deps --install-playwright
```

### Installation Targets

| Target | Location | Format |
|--------|----------|--------|
| `claude` | `~/.claude/skills/seo` | Skill directory |
| `codex` | `~/.codex/skills/seo` | Skill directory |
| `cursor` | `<project>/.cursor/rules/seo.mdc` | MDC rule |
| `windsurf` | `<project>/.windsurf/rules/seo.md` | Windsurf rule |
| `copilot` | `<project>/.github/copilot-instructions.md` | Repo instructions |
| `cline` | `<project>/.clinerules` | Project rules |
| `continue` | `<project>/.continue/prompts/seo.prompt` | Slash command |
| `antigravity` | `<project>/.agent/skills/seo` | Skill directory |

## Core Workflow

The skill follows an LLM-first analysis pattern:

1. **Collect Evidence** — Use `read_url_content` MCP tool or evidence collection scripts
2. **Analyze with LLM** — Apply explicit proof and reasoning for each finding
3. **Label Confidence** — Mark findings as `Confirmed`, `Likely`, or `Hypothesis`
4. **Prioritize** — Rank by impact and implementation effort
5. **Generate Action Plan** — Produce structured, evidence-backed recommendations

All audits must follow the `resources/references/llm-audit-rubric.md` standard.

## 16 Specialized Sub-Skills

### 1. SEO Audit (Full Site)

Run comprehensive site-wide audits with evidence-backed scoring:

```python
# Quick full audit with all outputs
python3 scripts/audit_runner.py https://example.com

# Outputs:
# - audit_results.json
# - audit_report.html
# - FULL-AUDIT-REPORT.md
# - ACTION-PLAN.md
```

**Agent prompt:**
```
Use the seo audit sub-skill to analyze https://example.com — follow the LLM audit rubric, collect evidence first with read_url_content, then reason through technical, content, and performance issues.
```

### 2. SEO Page (Single Page Analysis)

Deep analysis of individual pages:

```python
# Fetch page content
python3 scripts/fetch_page.py https://example.com/page

# Parse HTML for SEO signals
python3 scripts/parse_html.py example_com_page.html

# Check Core Web Vitals
python3 scripts/pagespeed.py https://example.com/page
```

**Agent prompt:**
```
Analyze this single page for SEO: https://example.com/about — check title, meta description, headings hierarchy, internal links, schema markup, and Core Web Vitals.
```

### 3. SEO Technical

Crawlability, indexability, security, mobile-friendliness, and AI crawler policies:

```python
# Check robots.txt policies
python3 scripts/robots_checker.py https://example.com

# Run indexability matrix (robots, meta robots, canonicals, status codes)
python3 scripts/indexability_matrix.py https://example.com

# Multi-page crawl audit
python3 scripts/crawl_audit.py https://example.com --max-depth 3
```

**Agent prompt:**
```
Run a technical SEO audit on https://example.com — check robots.txt, XML sitemaps, HTTPS, mobile responsiveness, JS rendering, and AI crawler access (GPTBot, ClaudeBot).
```

### 4. SEO Content

Content quality assessment with E-E-A-T framework (September 2025 Quality Rater Guidelines):

```python
# Extract article content
python3 scripts/fetch_page.py https://example.com/article

# Run content quality checks (agent analyzes with LLM)
# Check: word count, readability, E-E-A-T signals, topical depth
```

**Agent prompt:**
```
Assess content quality for https://example.com/blog/post using E-E-A-T criteria — check expertise signals, author credentials, content depth, readability, and December 2025 core update alignment.
```

### 5. SEO Schema

Schema.org detection, validation, and JSON-LD generation:

```python
# Validate existing schema markup
python3 scripts/validate_schema.py example_com.html

# Check for deprecated schema types
python3 scripts/parse_html.py example_com.html --extract-schema
```

**Agent prompt:**
```
Validate schema markup on https://example.com — check JSON-LD syntax, required fields, deprecated types, and suggest appropriate schema.org types (Article, Product, LocalBusiness, etc.).
```

**Example JSON-LD generation:**
```python
# Agent generates schema based on page content
schema = {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Page Title",
    "author": {
        "@type": "Person",
        "name": "Author Name"
    },
    "datePublished": "2026-01-15",
    "dateModified": "2026-05-27",
    "image": "https://example.com/image.jpg",
    "publisher": {
        "@type": "Organization",
        "name": "Publisher Name",
        "logo": {
            "@type": "ImageObject",
            "url": "https://example.com/logo.png"
        }
    }
}
```

### 6. SEO Sitemap

XML sitemap validation and generation:

```python
# Discover and validate sitemaps
python3 scripts/sitemap_checker.py https://example.com

# Check for issues: 404s, lastmod quality, URL limits (50,000 per file)
```

**Agent prompt:**
```
Audit XML sitemaps for https://example.com — check discovery (robots.txt reference), URL limits, 404s, lastmod accuracy, and sitemap index structure.
```

### 7. SEO Images

Image optimization audit (alt text, formats, lazy loading, CLS):

```python
# Generate image inventory
python3 scripts/image_inventory.py https://example.com

# Outputs: alt text coverage, dimensions, loading behavior, LCP candidates
```

**Agent prompt:**
```
Analyze image SEO for https://example.com — check alt text, file formats (WebP), lazy loading, responsive images (srcset), and LCP image optimization.
```

### 8. SEO GEO (Generative Engine Optimization)

Optimize for AI Overviews, ChatGPT, Perplexity:

**Agent prompt:**
```
Optimize this content for AI search engines (ChatGPT, Perplexity, Google AI Overviews) — add clear definitions, structured facts, citations, and FAQ sections.
```

**Pattern:**
- Use concise definitions (20-30 words)
- Add structured lists and tables
- Include citations and sources
- Create FAQ sections for common queries

### 9. SEO AEO (Answer Engine Optimization)

Optimize for Featured Snippets, People Also Ask, Knowledge Panels:

**Agent prompt:**
```
Optimize for Featured Snippets — structure content with clear question headers, concise answers (40-60 words), and bulleted/numbered lists.
```

**Pattern:**
```markdown
## What is [Topic]?

[Topic] is [concise 40-60 word definition].

### Key Benefits:
1. Benefit one
2. Benefit two
3. Benefit three
```

### 10. SEO Links

Link profile analysis — internal links, backlinks, anchor text, orphan pages:

```python
# Extract all links from page
python3 scripts/parse_html.py example_com.html --extract-links

# Agent analyzes: internal link structure, anchor text distribution, orphan pages
```

**Agent prompt:**
```
Audit internal linking structure for https://example.com — identify orphan pages, check anchor text diversity, analyze link depth, and suggest hub pages.
```

### 11. SEO Programmatic

Quality gates for programmatic SEO pages:

**Agent prompt:**
```
Review programmatic SEO pages for quality — check for thin content, duplicate patterns, template issues, and unique value signals on each page.
```

**Quality gates:**
- Minimum 800 words unique content per page
- No placeholder text (e.g., "[City]", "Lorem ipsum")
- Unique titles and meta descriptions
- 3+ internal links to related pages

### 12. SEO Competitors

Comparison and alternatives page generation:

**Agent prompt:**
```
Generate a "Competitors" or "Alternatives" comparison page for [Product] — include feature tables, pricing comparison, use case fit, and neutral tone.
```

### 13. SEO Hreflang

International SEO and hreflang validation:

```python
# Parse hreflang tags
python3 scripts/parse_html.py example_com.html --extract-hreflang
```

**Agent prompt:**
```
Validate hreflang implementation for multi-language site — check bidirectional links, correct language codes (ISO 639-1), regional variants (en-US, en-GB), and x-default fallback.
```

### 14. SEO Plan

Strategic SEO planning with topical clusters and industry templates:

**Agent prompt:**
```
Create an SEO content strategy for a SaaS product — build topical clusters, identify pillar pages, keyword themes, and internal linking structure.
```

**Industry templates available:**
- SaaS
- E-commerce
- Local Business
- Publisher/Media
- Agency
- Generic

### 15. SEO GitHub

GitHub repository SEO optimization:

```python
# Generate GitHub SEO report
python3 scripts/github_seo_report.py owner/repo

# Outputs:
# - GITHUB-SEO-REPORT.md
# - GITHUB-ACTION-PLAN.md
```

**Agent prompt:**
```
Optimize GitHub repository SEO for [owner/repo] — improve description, add topics, enhance README structure, optimize title/headline, and benchmark against query results.
```

**Key checks:**
- Repository description (160 chars, keyword-rich)
- Topics (8-12 relevant tags)
- README structure (clear headline, features, installation)
- Social preview image
- Community health files (CODE_OF_CONDUCT.md, CONTRIBUTING.md)

### 16. SEO Article

Article data extraction and LLM-driven content optimization:

**Agent prompt:**
```
Extract article content and optimize — check headline, readability, keyword density, internal links, meta description, and suggest improvements.
```

## 10 Specialist Agents

The skill includes specialized agents for domain-specific analysis:

1. **Technical SEO Agent** — Crawlability, indexability, security, mobile, JS rendering
2. **Content Quality Agent** — E-E-A-T scoring, AI content detection
3. **Performance Agent** — Core Web Vitals (LCP, INP, CLS)
4. **Schema Markup Agent** — JSON-LD validation and generation
5. **Sitemap Agent** — XML sitemap quality gates
6. **Visual Analysis Agent** — Screenshots, above-the-fold analysis (requires Playwright)
7. **GitHub Analyst Agent** — Repository metadata and README optimization
8. **GitHub Benchmark Agent** — Query ranking and competitor intelligence
9. **GitHub Data Agent** — API fallback and traffic archival
10. **Verifier Agent (Global)** — Deduplication and contradiction suppression

## Key Scripts Reference

### Evidence Collection

```python
# Fetch page with SEO crawler headers
python3 scripts/fetch_page.py https://example.com
# Output: example_com.html

# Parse HTML for all SEO signals
python3 scripts/parse_html.py example_com.html
# Extracts: title, meta, headings, links, images, schema, canonical

# Check robots.txt and crawler policies
python3 scripts/robots_checker.py https://example.com

# Multi-page crawl
python3 scripts/crawl_audit.py https://example.com --max-depth 2 --max-pages 50
```

### Performance & Core Web Vitals

```python
# PageSpeed Insights + Core Web Vitals
python3 scripts/pagespeed.py https://example.com
# Returns: LCP, INP, CLS scores + recommendations

# Note: Requires PAGESPEED_API_KEY environment variable
# Get free key: https://developers.google.com/speed/docs/insights/v5/get-started
```

### Indexability & Crawlability

```python
# Comprehensive indexability matrix
python3 scripts/indexability_matrix.py https://example.com
# Checks: robots.txt, meta robots, X-Robots-Tag, canonical, status codes, sitemaps

# Sitemap validation
python3 scripts/sitemap_checker.py https://example.com
# Validates: URL limits, 404s, lastmod, discovery
```

### Schema Validation

```python
# Validate JSON-LD schema
python3 scripts/validate_schema.py page.html
# Checks: syntax, required fields, deprecated types, placeholders
```

### Report Generation

```python
# Full audit with all outputs
python3 scripts/audit_runner.py https://example.com

# Generate HTML dashboard
python3 scripts/generate_report.py audit_results.json --output seo-report.html

# Verify findings (deduplicate + prioritize)
python3 scripts/finding_verifier.py audit_results.json
```

## Configuration

### Environment Variables

```bash
# PageSpeed Insights API key (optional but recommended)
export PAGESPEED_API_KEY=your_api_key_here

# GitHub token for repository analysis (optional)
export GITHUB_TOKEN=your_github_token_here
```

### Script Dependencies

Install Python dependencies:

```bash
# From project root
pip install -r requirements.txt

# Or with installer flag
bash install.sh --target claude --install-deps
```

**Core dependencies:**
- `requests` — HTTP requests
- `beautifulsoup4` — HTML parsing
- `lxml` — Fast XML/HTML parser
- `playwright` (optional) — Visual analysis and screenshots

Install Playwright browsers:

```bash
playwright install chromium
# Or use installer flag
bash install.sh --target claude --install-deps --install-playwright
```

## Common Patterns

### Pattern 1: Quick Page Audit

```python
# 1. Fetch page
python3 scripts/fetch_page.py https://example.com/page

# 2. Parse HTML
python3 scripts/parse_html.py example_com_page.html > page_data.json

# 3. Check Core Web Vitals
python3 scripts/pagespeed.py https://example.com/page > cwv_data.json

# 4. Agent analyzes JSON outputs with LLM reasoning
```

**Agent prompt:**
```
Analyze page_data.json and cwv_data.json — identify Critical/Warning/Pass findings, apply confidence labels, prioritize by impact, output structured action plan following llm-audit-rubric.md.
```

### Pattern 2: Multi-Page Crawl Audit

```python
# 1. Crawl site (respects robots.txt)
python3 scripts/crawl_audit.py https://example.com --max-depth 3 --max-pages 100 > crawl_results.json

# 2. Generate indexability matrix
python3 scripts/indexability_matrix.py https://example.com > indexability.json

# 3. Agent identifies issues: orphan pages, redirect chains, indexability blocks
```

### Pattern 3: Schema Validation & Generation

```python
# 1. Parse existing schema
python3 scripts/parse_html.py page.html --extract-schema > existing_schema.json

# 2. Validate
python3 scripts/validate_schema.py page.html

# 3. Agent generates improved schema based on page content and validation errors
```

### Pattern 4: GitHub Repository SEO

```python
# Generate full GitHub SEO report
python3 scripts/github_seo_report.py owner/repo

# Outputs:
# - GITHUB-SEO-REPORT.md (current state + benchmarks)
# - GITHUB-ACTION-PLAN.md (prioritized improvements)
```

**Agent workflow:**
1. Check repository metadata (description, topics, social preview)
2. Analyze README structure and keyword optimization
3. Benchmark title against search results
4. Suggest query-aligned title improvements
5. Identify missing community health files

### Pattern 5: Full Site Audit with Reports

```bash
# One command generates all outputs
python3 scripts/audit_runner.py https://example.com

# Generates:
# 1. audit_results.json (structured data)
# 2. audit_report.html (interactive dashboard)
# 3. FULL-AUDIT-REPORT.md (markdown report)
# 4. ACTION-PLAN.md (prioritized tasks)
```

## LLM Audit Rubric

All SEO analyses must follow the standard rubric at `resources/references/llm-audit-rubric.md`:

### Finding Format

```markdown
### [Severity]: [Finding Title]

**Evidence:**
[Specific, verifiable proof — URLs, code snippets, metrics]

**Impact:**
[SEO consequence — traffic, rankings, crawlability, user experience]

**Confidence:**
- Confirmed / Likely / Hypothesis

**Fix:**
[Actionable, specific remediation steps]

**Priority:**
- Impact: High / Medium / Low
- Effort: High / Medium / Low
```

### Severity Levels

- **Critical** — Blocks indexing, causes crawl errors, security issues
- **Warning** — Degrades SEO performance, suboptimal implementation
- **Pass** — Meets best practices
- **Info** — Opportunities, neutral observations

### Confidence Labels

- **Confirmed** — Direct evidence (status codes, parsed HTML, API responses)
- **Likely** — Strong inference from multiple signals
- **Hypothesis** — Reasoned speculation requiring validation

## Troubleshooting

### Script Errors

**Issue:** `ModuleNotFoundError: No module named 'requests'`

**Fix:**
```bash
pip install -r requirements.txt
# Or
pip install requests beautifulsoup4 lxml
```

---

**Issue:** `playwright._impl._errors.Error: Executable doesn't exist`

**Fix:**
```bash
pip install playwright
playwright install chromium
```

---

**Issue:** PageSpeed API rate limit (403 or 429 errors)

**Fix:**
```bash
# Get free API key (100 queries/day)
# https://developers.google.com/speed/docs/insights/v5/get-started

export PAGESPEED_API_KEY=your_key_here
python3 scripts/pagespeed.py https://example.com
```

### Installation Issues

**Issue:** Skill not detected after installation

**Fix:**
```bash
# Verify installation path
ls -la ~/.claude/skills/seo/

# Restart IDE
# Claude Code: restart extension
# Cursor: restart application

# Re-run installer with verbose output
bash install.sh --target claude -v
```

---

**Issue:** Windows PowerShell execution policy error

**Fix:**
```powershell
# Run once per session
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

# Then run installer
.\install.ps1 --online --target claude
```

### Audit Quality Issues

**Issue:** Too many low-confidence findings

**Fix:**
```python
# Run finding verifier to deduplicate and prioritize
python3 scripts/finding_verifier.py audit_results.json

# Agent: focus on Confirmed/Likely findings first, mark Hypothesis clearly
```

---

**Issue:** Missing evidence in audit reports

**Fix:**
- Always collect evidence first (`fetch_page.py`, `parse_html.py`, `pagespeed.py`)
- Reference specific URLs, line numbers, metric values
- Use `read_url_content` MCP tool before reasoning
- Apply llm-audit-rubric.md standard for every finding

## Reference Data

The skill includes up-to-date reference files (checked by CI):

- **Core Web Vitals** — LCP (2.5s), INP (200ms), CLS (0.1)
- **E-E-A-T Framework** — September 2025 Quality Rater Guidelines
- **Schema.org Types** — Active, restricted, deprecated lists
- **Content Quality Gates** — Word count minimums, readability thresholds
- **Google SEO Quick Reference** — Best practices snapshot
- **LLM Audit Rubric** — Standardized output contract

Reference freshness is validated with:

```bash
python3 scripts/reference_freshness.py resources/references --max-age-days 90
```

## Advanced Usage

### Custom Industry Strategy

```python
# Agent generates topical cluster strategy
# Example for SaaS product:

# Pillar page: /seo-tools
# Cluster pages:
#   /seo-tools/keyword-research
#   /seo-tools/link-building
#   /seo-tools/technical-audit
# Each cluster page links back to pillar

# Agent uses industry template from:
# resources/skills/seo-plan.md
```

### Multi-Language Hreflang

```html
<!-- Agent generates hreflang tags -->
<link rel="alternate" hreflang="en" href="https://example.com/page" />
<link rel="alternate" hreflang="es" href="https://example.com/es/page" />
<link rel="alternate" hreflang="de" href="https://example.com/de/page" />
<link rel="alternate" hreflang="x-default" href="https://example.com/page" />
```

**Validation checklist:**
- Bidirectional links (each page references all others)
- Correct ISO 639-1 language codes
- Regional variants (en-US, en-GB) where applicable
- x-default fallback for unmatched locales
- Self-referential link included

### Programmatic SEO Quality Gates

```python
# Agent validates programmatic pages before deployment

quality_gates = {
    "min_words": 800,
    "unique_title": True,
    "unique_meta_description": True,
    "no_placeholders": True,
    "min_internal_links": 3,
    "unique_content_ratio": 0.7  # 70% unique vs template
}

# Reject pages that fail gates
# Log failures for content team review
```

## Wiki & Documentation

Full documentation available at:
https://github.com/Bhanunamikaze/Agentic-SEO-Skill/wiki

Key wiki pages:
- **Script Inventory** — All 89 scripts with purpose notes
- **Installation Guide** — Detailed per-IDE setup
- **Example Prompts** — 50+ tested agent prompts
- **Troubleshooting** — Common issues and fixes
- **Report Generation** — Custom report templates

## License

MIT License — free for commercial and personal use.
