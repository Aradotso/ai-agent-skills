---
name: vercel-agent-browser
description: Browser automation CLI for AI agents - fast native Rust tool for controlling Chrome with accessibility-first commands
triggers:
  - automate a browser task
  - control chrome with agent-browser
  - take a screenshot of a webpage
  - fill out a form automatically
  - extract data from a webpage
  - interact with web elements
  - navigate and click on a website
  - scrape website content with browser automation
---

# Vercel Agent Browser Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

`agent-browser` is a fast native Rust CLI for browser automation designed specifically for AI agents. It provides an accessibility-first approach with semantic selectors and ref-based element targeting, making it ideal for LLM-driven web automation.

**Key Features:**
- Native Rust performance with npm/Homebrew distribution
- Accessibility tree snapshots with stable element refs (`@e1`, `@e2`, etc.)
- Semantic locators (role, text, label, placeholder)
- AI chat mode for natural language control
- Batch command execution
- Network interception and HAR recording
- Chrome DevTools Protocol (CDP) streaming

## Installation

### Global (Recommended)

```bash
npm install -g agent-browser
agent-browser install  # Downloads Chrome for Testing
```

### Project Local

```bash
npm install agent-browser
npx agent-browser install
```

### Alternative Methods

```bash
# Homebrew (macOS)
brew install agent-browser
agent-browser install

# Cargo (Rust)
cargo install agent-browser
agent-browser install

# Linux with system dependencies
agent-browser install --with-deps
```

### Upgrading

```bash
agent-browser upgrade  # Auto-detects installation method
```

## Core Concepts

### Element References (@refs)

The most powerful feature for AI agents is the accessibility snapshot with stable refs:

```bash
# Get accessibility tree with element refs
agent-browser open https://example.com
agent-browser snapshot

# Output includes refs like:
# @e1 heading "Example Domain"
# @e2 link "More information..."
# @e3 textbox "Search" (placeholder)

# Use refs directly in commands
agent-browser click @e2
agent-browser fill @e3 "search query"
agent-browser get text @e1
```

### Traditional Selectors

Standard CSS selectors also work:

```bash
agent-browser click "#submit-button"
agent-browser fill "input[name='email']" "user@example.com"
agent-browser get text ".header h1"
```

### Semantic Locators

Find elements by their semantic meaning:

```bash
# By ARIA role
agent-browser find role button click --name "Submit"
agent-browser find role textbox fill "user@example.com" --name "Email"

# By visible text
agent-browser find text "Sign In" click
agent-browser find text "Welcome" click --exact

# By label
agent-browser find label "Password" fill "secret123"

# By placeholder
agent-browser find placeholder "Search..." type "rust cli"

# By test ID
agent-browser find testid "login-form" click
```

## Common Workflows

### Basic Navigation and Interaction

```bash
# Start browser and navigate
agent-browser open https://github.com/login

# Get page structure
agent-browser snapshot

# Fill login form using refs from snapshot
agent-browser fill @e5 "username"
agent-browser fill @e6 "password"
agent-browser click @e7

# Or use semantic locators
agent-browser find label "Username" fill "username"
agent-browser find label "Password" fill "password"
agent-browser find role button click --name "Sign in"

# Wait for navigation
agent-browser wait --url "**/dashboard"

# Take screenshot
agent-browser screenshot success.png
```

### Form Automation

```bash
agent-browser open https://forms.example.com

# Fill text inputs
agent-browser fill "#name" "John Doe"
agent-browser fill "#email" "john@example.com"

# Select dropdown
agent-browser select "#country" "United States"

# Check checkboxes
agent-browser check "#newsletter"
agent-browser check "#terms"

# Upload files
agent-browser upload "#resume" "/path/to/resume.pdf"

# Submit
agent-browser find role button click --name "Submit"

# Wait for success message
agent-browser wait --text "Thank you"
```

### Data Extraction

```bash
agent-browser open https://news.ycombinator.com

# Get page snapshot for structure
agent-browser snapshot -i  # Interactive mode shows full tree

# Extract specific content
agent-browser get text ".titleline > a"

# Get multiple elements with JavaScript
agent-browser eval "Array.from(document.querySelectorAll('.titleline > a')).map(a => a.textContent)"

# Get structured data as JSON
agent-browser eval "JSON.stringify({
  title: document.querySelector('title').textContent,
  links: Array.from(document.querySelectorAll('.titleline > a')).map(a => ({
    text: a.textContent,
    url: a.href
  }))
})"
```

### Batch Execution

Reduce overhead by running multiple commands in one invocation:

```bash
# Argument mode
agent-browser batch \
  "open https://example.com" \
  "snapshot -i" \
  "click @e2" \
  "wait 1000" \
  "screenshot result.png"

# With --bail to stop on first error
agent-browser batch --bail \
  "open https://login.example.com" \
  "fill #username user@example.com" \
  "fill #password ${PASSWORD}" \
  "click #submit" \
  "wait --url '**/dashboard'"

# JSON mode (piped from script)
echo '[
  ["open", "https://example.com"],
  ["snapshot"],
  ["click", "@e1"],
  ["screenshot"]
]' | agent-browser batch --json
```

### AI Chat Mode

Natural language browser control:

```bash
# Single-shot command
agent-browser chat "go to github.com and search for rust browser automation"

# Interactive REPL
agent-browser chat
# > navigate to example.com
# > click the first link
# > take a screenshot
# > exit
```

### Network Interception

```bash
# Block specific resources
agent-browser network route "**/*.jpg" --abort
agent-browser network route "https://ads.example.com/*" --abort

# Block resource types
agent-browser network route '*' --abort --resource-type script
agent-browser network route '*' --abort --resource-type image,media

# Mock API responses
agent-browser network route "https://api.example.com/user" --body '{
  "status": "ok",
  "data": {"id": 1, "name": "Test User"}
}'

# View network requests
agent-browser network requests
agent-browser network requests --filter api
agent-browser network requests --type fetch,xhr
agent-browser network requests --method POST
agent-browser network requests --status 200

# View specific request detail
agent-browser network request req_123abc

# HAR recording
agent-browser network har start
# ... perform actions ...
agent-browser network har stop capture.har
```

### Screenshot Strategies

```bash
# Basic screenshot (saves to temp if no path)
agent-browser screenshot

# Save to specific path
agent-browser screenshot ./screenshots/page.png

# Full page screenshot
agent-browser screenshot --full full-page.png

# Annotated with element labels
agent-browser screenshot --annotate labeled.png

# Custom directory and format
agent-browser screenshot --screenshot-dir ./shots --screenshot-format jpeg --screenshot-quality 80

# Element screenshot
agent-browser eval "document.querySelector('#main').screenshot()" -b > element.png
```

### Multi-Tab Operations

```bash
# List tabs
agent-browser tab

# Open new tab with label
agent-browser tab new --label docs https://docs.example.com

# Open link in new tab
agent-browser click "#external-link" --new-tab

# Switch to tab by label
agent-browser tab docs

# Switch to tab by ID
agent-browser tab t2

# Close tab
agent-browser tab close docs
agent-browser tab close t2
```

### Wait Strategies

```bash
# Wait for element to appear
agent-browser wait "#result"

# Wait for time (milliseconds)
agent-browser wait 2000

# Wait for text (substring match)
agent-browser wait --text "Success"

# Wait for URL pattern
agent-browser wait --url "**/dashboard"

# Wait for network idle
agent-browser wait --load networkidle

# Wait for JavaScript condition
agent-browser wait --fn "document.readyState === 'complete'"
agent-browser wait --fn "window.dataLoaded === true"

# Wait for element to disappear
agent-browser wait --fn "!document.body.innerText.includes('Loading...')"
agent-browser wait "#spinner" --state hidden
```

### Keyboard and Mouse Control

```bash
# Keyboard shortcuts
agent-browser press "Control+a"
agent-browser press "Control+c"
agent-browser press "Enter"
agent-browser press "Tab"

# Type with real keystrokes (at current focus)
agent-browser keyboard type "Hello World"

# Insert text without key events
agent-browser keyboard inserttext "Pasted content"

# Hold and release keys
agent-browser keydown "Shift"
agent-browser press "a"
agent-browser keyup "Shift"

# Mouse control
agent-browser mouse move 100 200
agent-browser mouse down left
agent-browser mouse up left
agent-browser mouse wheel 100 0  # dy dx

# Drag and drop
agent-browser drag "#source" "#target"
```

### Cookies and Storage

```bash
# Cookies
agent-browser cookies
agent-browser cookies set "sessionId" "abc123"
agent-browser cookies clear

# Import from cURL
agent-browser cookies set --curl curl-cookies.txt

# LocalStorage
agent-browser storage local
agent-browser storage local get "token"
agent-browser storage local set "token" "eyJ..."
agent-browser storage local clear

# SessionStorage
agent-browser storage session
agent-browser storage session set "tempData" '{"key":"value"}'
```

### Browser Configuration

```bash
# Set viewport
agent-browser set viewport 1920 1080
agent-browser set viewport 1920 1080 2  # Retina (2x scale)

# Emulate device
agent-browser set device "iPhone 14"

# Geolocation
agent-browser set geo 37.7749 -122.4194  # San Francisco

# Offline mode
agent-browser set offline on
agent-browser set offline off

# Custom headers
agent-browser set headers '{"X-Custom-Header":"value"}'

# HTTP auth
agent-browser set credentials username password

# Color scheme
agent-browser set media dark
agent-browser set media light
```

### Advanced JavaScript Evaluation

```bash
# Simple evaluation
agent-browser eval "document.title"

# Complex extraction
agent-browser eval "
  JSON.stringify({
    url: window.location.href,
    links: Array.from(document.querySelectorAll('a')).map(a => ({
      text: a.textContent.trim(),
      href: a.href
    })).filter(l => l.text)
  })
"

# Binary output (base64)
agent-browser eval "document.querySelector('canvas').toDataURL()" -b > canvas.png

# From stdin
echo "document.body.innerHTML" | agent-browser eval --stdin
```

## Automation Script Example

Create a reusable automation script:

```bash
#!/bin/bash
# login-and-extract.sh

set -e

SESSION_FILE=".browser-session"

# Start browser
agent-browser open https://app.example.com/login

# Login
agent-browser find label "Email" fill "${USER_EMAIL}"
agent-browser find label "Password" fill "${USER_PASSWORD}"
agent-browser find role button click --name "Sign in"

# Wait for dashboard
agent-browser wait --url "**/dashboard"
agent-browser wait 1000

# Extract data
DATA=$(agent-browser eval "
  JSON.stringify({
    user: document.querySelector('.user-name').textContent,
    notifications: Array.from(document.querySelectorAll('.notification')).map(n => n.textContent)
  })
")

echo "$DATA" | jq .

# Screenshot
agent-browser screenshot dashboard.png

# Cleanup
agent-browser close
```

Usage:

```bash
export USER_EMAIL="user@example.com"
export USER_PASSWORD="secret"
chmod +x login-and-extract.sh
./login-and-extract.sh
```

## Batch Automation Pattern

For complex multi-step workflows, use batch mode with a script:

```javascript
#!/usr/bin/env node
// automation.js

const commands = [
  ["open", "https://example.com"],
  ["snapshot"],
  // Process snapshot, determine next steps...
  ["click", "@e5"],
  ["wait", "1000"],
  ["screenshot", "step1.png"],
  ["find", "role", "button", "click", "--name", "Next"],
  ["wait", "--url", "**/step2"],
  ["screenshot", "step2.png"]
];

console.log(JSON.stringify(commands));
```

Run with:

```bash
node automation.js | agent-browser batch --json --bail
```

## CDP Streaming for Real-Time Monitoring

Connect to Chrome DevTools Protocol for event streaming:

```bash
# Get CDP WebSocket URL
CDP_URL=$(agent-browser get cdp-url)
echo "CDP URL: $CDP_URL"

# Or connect directly to CDP port
agent-browser connect 9222

# Enable runtime streaming
agent-browser stream enable --port 9223
agent-browser stream status
# ... perform actions, stream events to ws://localhost:9223 ...
agent-browser stream disable
```

## Troubleshooting

### Browser Not Found

```bash
# Re-download Chrome for Testing
agent-browser install

# Or specify custom Chrome path
export CHROME_PATH=/path/to/chrome
agent-browser open
```

### Elements Not Found

```bash
# Use snapshot to see available refs
agent-browser snapshot -i

# Wait for element before interacting
agent-browser wait "#dynamic-element"
agent-browser click "#dynamic-element"

# Use semantic selectors for robustness
agent-browser find role button click --name "Submit"
```

### Session Management

```bash
# List all active sessions
agent-browser tab

# Close specific session
agent-browser close

# Close all sessions
agent-browser close --all
```

### Debugging

```bash
# Get CDP URL for DevTools connection
agent-browser get cdp-url

# View network activity
agent-browser network requests

# Enable HAR recording
agent-browser network har start
# ... actions ...
agent-browser network har stop debug.har
```

### Clipboard Access Issues

On Linux, clipboard requires `xclip` or `xsel`:

```bash
sudo apt-get install xclip
# or
sudo apt-get install xsel
```

## Environment Variables

```bash
# Custom Chrome binary
export CHROME_PATH=/Applications/Brave Browser.app/Contents/MacOS/Brave Browser

# Custom user data directory
export CHROME_USER_DATA_DIR=~/.config/agent-browser

# Headless mode (default)
export CHROME_HEADLESS=true

# Show browser UI
export CHROME_HEADLESS=false
```

## Best Practices for AI Agents

1. **Use snapshots first**: Always get `agent-browser snapshot` to understand page structure before acting
2. **Prefer refs over selectors**: Use `@e1` style refs from snapshots for stability
3. **Use semantic locators**: `find role button --name "Submit"` is more resilient than CSS selectors
4. **Wait strategically**: Add `wait` commands for dynamic content before interacting
5. **Batch when possible**: Use `batch` mode to reduce per-command overhead
6. **Handle errors**: Use `--bail` in batch mode to stop on first error
7. **Clean up**: Always `close` sessions when done
8. **Screenshot for verification**: Take screenshots at key steps for debugging
9. **Use labels for tabs**: Assign meaningful labels when opening multiple tabs

## Quick Reference

```bash
# Essential commands for AI agents
agent-browser open <url>              # Start
agent-browser snapshot                # Get structure with refs
agent-browser click @eN               # Interact with ref
agent-browser find role button click --name "Text"  # Semantic
agent-browser wait --text "Success"   # Wait for content
agent-browser screenshot result.png   # Verify
agent-browser get text @eN            # Extract
agent-browser eval "JS"               # Complex extraction
agent-browser batch "cmd1" "cmd2"     # Multi-step
agent-browser close                   # Cleanup
```
