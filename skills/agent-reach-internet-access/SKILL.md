---
name: agent-reach-internet-access
description: Give AI agents eyes to see the internet — scrape Twitter, Reddit, YouTube, GitHub, Bilibili, XiaoHongShu with zero API fees
triggers:
  - search twitter for mentions
  - get youtube video transcript
  - scrape this reddit thread
  - read this xiaohongshu post
  - search github repositories
  - get bilibili video subtitles
  - read this web page content
  - search the internet for
---

# Agent Reach — Internet Access for AI Agents

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agent Reach is a scaffolding tool that gives AI agents the ability to read and search across Twitter, Reddit, YouTube, GitHub, Bilibili, XiaoHongShu, and more — all without paid APIs. It orchestrates best-in-class upstream tools (yt-dlp, twitter-cli, rdt-cli, gh CLI, etc.) and provides a unified interface for AI agents.

## Installation

Agent Reach is installed via pip and automatically sets up dependencies:

```bash
# Basic installation
pip install agent-reach

# The tool will auto-detect and install:
# - Node.js (for some MCP servers)
# - gh CLI (for GitHub)
# - mcporter (for MCP integrations)
# - twitter-cli (for Twitter/X)
# - rdt-cli (for Reddit)
# - yt-dlp (for YouTube/Bilibili)
```

After installation, run diagnostics to check what's working:

```bash
agent-reach doctor
```

This shows status for each channel: ✅ (works out of box), 🔧 (needs config), or ❌ (not available).

## Core Capabilities

### 1. Web Page Reading (Jina Reader)

Read any web page as clean markdown:

```bash
# Read a web page
curl https://r.jina.ai/https://example.com

# Get JSON format
curl https://r.jina.ai/https://example.com \
  -H "Accept: application/json"

# With images
curl https://r.jina.ai/https://example.com \
  -H "X-With-Images-Summary: true"
```

**Python usage:**

```python
import requests

url = "https://example.com"
response = requests.get(f"https://r.jina.ai/{url}")
markdown_content = response.text

# With options
headers = {
    "X-With-Links-Summary": "true",
    "X-With-Images-Summary": "true"
}
response = requests.get(f"https://r.jina.ai/{url}", headers=headers)
```

### 2. YouTube & Video (yt-dlp)

Extract subtitles, metadata, and search videos:

```bash
# Get video metadata + subtitles
yt-dlp --dump-json --write-auto-subs --skip-download \
  "https://www.youtube.com/watch?v=VIDEO_ID"

# Search YouTube
yt-dlp "ytsearch5:AI agents tutorial" --dump-json

# Get specific subtitle language
yt-dlp --write-subs --sub-lang en --skip-download URL

# Bilibili videos (works same way)
yt-dlp --dump-json "https://www.bilibili.com/video/BV..."
```

**Python usage:**

```python
import subprocess
import json

def get_video_info(url):
    result = subprocess.run(
        ["yt-dlp", "--dump-json", "--write-auto-subs", 
         "--skip-download", url],
        capture_output=True, text=True
    )
    return json.loads(result.stdout)

# Search videos
def search_youtube(query, max_results=5):
    result = subprocess.run(
        ["yt-dlp", f"ytsearch{max_results}:{query}", "--dump-json"],
        capture_output=True, text=True
    )
    return [json.loads(line) for line in result.stdout.strip().split('\n')]
```

### 3. Twitter/X (twitter-cli)

Requires Cookie authentication. Export cookies using [Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm) Chrome extension.

```bash
# Configure (paste exported cookies when prompted)
twitter configure

# Read a tweet
twitter tweet https://twitter.com/user/status/123456789

# Search tweets
twitter search "AI agents" --limit 20

# Get user timeline
twitter timeline @username --limit 50

# Get tweet thread
twitter thread https://twitter.com/user/status/123456789
```

**Configuration file location:** `~/.twitter-cli/config.json`

### 4. Reddit (rdt-cli)

Requires Cookie authentication:

```bash
# Login with cookies
rdt login

# Search posts
rdt search "machine learning" --limit 20

# Read post with comments
rdt post https://reddit.com/r/programming/comments/...

# Get subreddit posts
rdt subreddit r/python --limit 30
```

### 5. GitHub (gh CLI)

```bash
# Login (opens browser OAuth flow)
gh auth login

# View repository
gh repo view owner/repo

# Search repositories
gh search repos "LLM framework" --limit 20

# Search issues
gh search issues "bug" --repo owner/repo

# View issue
gh issue view 123 --repo owner/repo

# Create issue
gh issue create --repo owner/repo \
  --title "Bug report" --body "Description"
```

**Python usage:**

```python
import subprocess
import json

def search_repos(query, limit=20):
    result = subprocess.run(
        ["gh", "search", "repos", query, 
         "--limit", str(limit), "--json", "name,description,url"],
        capture_output=True, text=True
    )
    return json.loads(result.stdout)

def get_repo_info(owner_repo):
    result = subprocess.run(
        ["gh", "repo", "view", owner_repo, "--json", 
         "description,stargazerCount,forkCount,url"],
        capture_output=True, text=True
    )
    return json.loads(result.stdout)
```

### 6. XiaoHongShu (xhs-cli via mcporter)

Requires Cookie authentication:

```bash
# Configure (sets up MCP server)
mcporter add xiaohongshu

# The MCP server provides these tools:
# - search_notes: Search XHS posts
# - get_note_detail: Get post content
# - post_note: Create new post
# - comment_note: Add comment
# - like_note: Like a post
```

Configuration stored in: `~/.mcporter/xiaohongshu/config.json`

### 7. Bilibili Enhanced (bili-cli)

```bash
# Get hot videos
bili hot --limit 20

# Search videos
bili search "Python tutorial" --limit 30

# Get video info
bili video BV1xx411c7mD

# Get user dynamics
bili user-dynamic 123456
```

### 8. Internet Search (Exa via mcporter)

Semantic search across the web:

```bash
# Add Exa MCP server (no API key needed for basic use)
mcporter add exa

# The MCP server provides:
# - search: AI-powered semantic search
# - find_similar: Find similar pages
# - get_contents: Extract page contents
```

**For advanced features, set API key:**

```bash
export EXA_API_KEY=your_key_here
```

### 9. RSS Feeds

```python
import feedparser

# Parse RSS feed
feed = feedparser.parse("https://example.com/feed.xml")

for entry in feed.entries:
    print(f"Title: {entry.title}")
    print(f"Link: {entry.link}")
    print(f"Published: {entry.published}")
    print(f"Summary: {entry.summary}")
    print("---")
```

### 10. WeChat Official Accounts

Search and read WeChat articles via Exa + Camoufox:

```python
# Use Exa search to find WeChat articles
# Articles are auto-extracted when URLs contain mp.weixin.qq.com
```

### 11. Weibo (微博)

```bash
# Search content
agent-reach weibo search "AI" --type content

# Get hot search
agent-reach weibo hot

# Get user posts
agent-reach weibo user USER_ID

# Get comments
agent-reach weibo comments POST_ID
```

### 12. V2EX

```bash
# Get hot topics
agent-reach v2ex hot

# Get node topics
agent-reach v2ex node python

# Get topic details
agent-reach v2ex topic 123456
```

## Configuration Patterns

### Cookie-Based Services

For Twitter, Reddit, XiaoHongShu — use Cookie-Editor:

1. Login to the service in browser
2. Install [Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
3. Click extension → Export → Copy
4. Paste into CLI config command

**Never commit cookies to version control.** They're stored in:
- Twitter: `~/.twitter-cli/config.json`
- Reddit: `~/.rdt-cli/cookies.json`
- XHS: `~/.mcporter/xiaohongshu/config.json`

### Proxy Configuration (Server Deployments)

For Bilibili access from servers:

```bash
# Set proxy environment variables
export HTTP_PROXY=http://proxy-server:port
export HTTPS_PROXY=http://proxy-server:port

# Or configure per-tool
yt-dlp --proxy http://proxy-server:port URL
```

### GitHub Authentication

```bash
# OAuth login (recommended)
gh auth login

# Or use token
export GITHUB_TOKEN=ghp_your_token_here
gh auth login --with-token <<< $GITHUB_TOKEN
```

## Common Workflows

### Scrape Twitter Thread for Research

```python
import subprocess
import json

def get_twitter_thread(url):
    result = subprocess.run(
        ["twitter", "thread", url],
        capture_output=True, text=True
    )
    return result.stdout

thread_content = get_twitter_thread(
    "https://twitter.com/user/status/123456789"
)
```

### Extract YouTube Video Summary

```python
import subprocess
import json

def get_video_transcript(url):
    # Get metadata + subtitles
    result = subprocess.run(
        ["yt-dlp", "--dump-json", "--write-auto-subs", 
         "--skip-download", url],
        capture_output=True, text=True
    )
    data = json.loads(result.stdout)
    
    # Subtitles are in data['subtitles'] or data['automatic_captions']
    return {
        'title': data.get('title'),
        'description': data.get('description'),
        'duration': data.get('duration'),
        'subtitles': data.get('automatic_captions', {})
    }
```

### Search GitHub for Solutions

```python
import subprocess
import json

def search_github_issues(query, repo=None):
    cmd = ["gh", "search", "issues", query, 
           "--limit", "20", "--json", 
           "title,url,state,body,comments"]
    if repo:
        cmd.extend(["--repo", repo])
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return json.loads(result.stdout)

# Search across all repos
issues = search_github_issues("memory leak in agents")

# Search specific repo
issues = search_github_issues("bug", repo="openai/gpt-4")
```

### Monitor Reddit for Mentions

```bash
# Search and save results
rdt search "your_product_name" --limit 50 > mentions.txt

# Get specific subreddit
rdt subreddit r/artificial --limit 100
```

### Read Web Page Content for Analysis

```python
import requests

def get_clean_content(url):
    response = requests.get(
        f"https://r.jina.ai/{url}",
        headers={
            "X-With-Links-Summary": "true",
            "X-No-Cache": "true"
        }
    )
    return response.text

content = get_clean_content("https://news.ycombinator.com")
```

## Troubleshooting

### Doctor Command Shows ❌

Run diagnostics:

```bash
agent-reach doctor
```

Each ❌ includes a fix suggestion. Common issues:

**Twitter/Reddit not working:**
- Need Cookie authentication
- Use Cookie-Editor to export cookies
- Run `twitter configure` or `rdt login`

**Bilibili 403 on server:**
- Need proxy for non-CN IPs
- Set `HTTP_PROXY` and `HTTPS_PROXY` env vars

**GitHub rate limited:**
- Authenticate: `gh auth login`
- Authenticated rate: 5,000/hour vs 60/hour

**yt-dlp fails:**
- Update to latest: `pip install -U yt-dlp`
- Tool is actively maintained, updates frequently

### MCP Server Connection Issues

```bash
# Check mcporter status
mcporter list

# Restart a server
mcporter restart xiaohongshu

# View server logs
mcporter logs exa
```

### Proxy Not Working

```bash
# Test proxy connection
curl -x http://proxy:port https://api.bilibili.com

# Set for specific command
export HTTPS_PROXY=http://proxy:port
yt-dlp URL
```

### Cookie Expired

Re-export fresh cookies:
1. Login to service in browser
2. Export with Cookie-Editor
3. Reconfigure CLI tool

## Environment Variables

```bash
# Proxy (for server deployments)
export HTTP_PROXY=http://proxy:port
export HTTPS_PROXY=http://proxy:port

# GitHub
export GITHUB_TOKEN=ghp_xxxxx

# Exa (optional, for advanced features)
export EXA_API_KEY=your_key_here

# Custom config paths (optional)
export AGENT_REACH_CONFIG_DIR=~/.config/agent-reach
```

## Safety & Privacy

- **All cookies stored locally** in `~/.twitter-cli/`, `~/.rdt-cli/`, etc.
- **No data uploaded** to Agent Reach servers (there are none)
- **Code is open source** — audit anytime
- Use `--safe` mode during install to review system package installs

## Updating

```bash
# Update agent-reach
pip install -U agent-reach

# Update individual tools
pip install -U yt-dlp
gh extension upgrade --all
npm update -g mcporter
```

Check for breaking changes: https://github.com/Panniantong/agent-reach/blob/main/CHANGELOG.md

## Platform Support Matrix

| Platform | Out of Box | After Config | Notes |
|----------|-----------|--------------|-------|
| Web | ✅ | — | Jina Reader, no limits |
| YouTube | ✅ | — | yt-dlp, 1800+ sites |
| RSS | ✅ | — | feedparser |
| GitHub | ✅ | 🔧 Auth for private | gh CLI |
| Twitter | 🔧 Cookie | 🔧 Cookie | twitter-cli |
| Reddit | 🔧 Cookie | 🔧 Cookie | rdt-cli |
| Bilibili | ✅ Local | 🔧 Proxy (server) | yt-dlp |
| XiaoHongShu | 🔧 Cookie | 🔧 Cookie | xhs-cli via MCP |
| Search | 🔧 MCP | 🔧 API key (optional) | Exa |
| WeChat | ✅ | — | Via Exa search |
| Weibo | ✅ | — | Direct API |
| V2EX | ✅ | — | Direct API |

Legend: ✅ Works immediately | 🔧 Needs configuration
