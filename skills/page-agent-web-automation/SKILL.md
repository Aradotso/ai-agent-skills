---
name: page-agent-web-automation
description: Control web interfaces with natural language using Page Agent, a JavaScript in-page GUI agent for browser automation
triggers:
  - "add page agent to my web app"
  - "automate web page interactions with natural language"
  - "integrate an AI copilot into my website"
  - "use page agent to control the DOM"
  - "set up page agent with my LLM"
  - "create a web automation agent"
  - "implement browser automation with javascript"
  - "build a web GUI agent"
---

# Page Agent Web Automation

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Page Agent is a JavaScript in-page GUI agent that enables controlling web interfaces through natural language commands. Unlike traditional browser automation tools, it runs directly in the webpage (no browser extension or headless browser required) and uses text-based DOM manipulation instead of screenshots.

## Installation

### NPM Installation

```bash
npm install page-agent
```

### CDN (Quick Testing)

For rapid prototyping with a demo LLM (evaluation purposes only):

```html
<script src="https://cdn.jsdelivr.net/npm/page-agent@1.8.2/dist/iife/page-agent.demo.js" crossorigin="true"></script>
```

Add `?autoInit=false` to prevent automatic initialization:

```html
<script src="https://cdn.jsdelivr.net/npm/page-agent@1.8.2/dist/iife/page-agent.demo.js?autoInit=false" crossorigin="true"></script>
<script>
  const agent = new window.PageAgent({...});
</script>
```

## Basic Usage

### Importing and Initialization

```typescript
import { PageAgent } from 'page-agent'

const agent = new PageAgent({
  model: 'qwen3.5-plus',
  baseURL: 'https://dashscope.aliyuncs.com/compatible-mode/v1',
  apiKey: process.env.DASHSCOPE_API_KEY,
  language: 'en-US',
})
```

### Executing Commands

```typescript
// Simple command execution
await agent.execute('Click the login button')

// Form filling
await agent.execute('Fill in the email field with user@example.com')

// Multi-step workflow
await agent.execute('Search for "page agent" and click the first result')

// Navigation
await agent.execute('Go to the settings page')
```

## Configuration Options

### Basic Configuration

```typescript
const agent = new PageAgent({
  // LLM Configuration
  model: 'gpt-4',
  baseURL: 'https://api.openai.com/v1',
  apiKey: process.env.OPENAI_API_KEY,
  
  // Language settings
  language: 'en-US', // or 'zh-CN'
  
  // Optional: Custom system prompt
  systemPrompt: 'You are a helpful assistant...',
})
```

### Advanced Configuration

```typescript
const agent = new PageAgent({
  model: 'claude-3-5-sonnet-20241022',
  baseURL: 'https://api.anthropic.com/v1',
  apiKey: process.env.ANTHROPIC_API_KEY,
  language: 'en-US',
  
  // Execution options
  maxSteps: 20, // Maximum execution steps
  timeout: 30000, // Timeout in milliseconds
  
  // Custom element selector strategy
  elementSelector: {
    includeInvisible: false,
    maxElements: 100,
  },
  
  // Debug mode
  debug: true,
})
```

## Supported LLM Providers

### OpenAI

```typescript
const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: 'https://api.openai.com/v1',
  apiKey: process.env.OPENAI_API_KEY,
})
```

### Anthropic Claude

```typescript
const agent = new PageAgent({
  model: 'claude-3-5-sonnet-20241022',
  baseURL: 'https://api.anthropic.com/v1',
  apiKey: process.env.ANTHROPIC_API_KEY,
})
```

### Alibaba Qwen (DashScope)

```typescript
const agent = new PageAgent({
  model: 'qwen3.5-plus',
  baseURL: 'https://dashscope.aliyuncs.com/compatible-mode/v1',
  apiKey: process.env.DASHSCOPE_API_KEY,
})
```

### Azure OpenAI

```typescript
const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: process.env.AZURE_OPENAI_ENDPOINT,
  apiKey: process.env.AZURE_OPENAI_API_KEY,
})
```

## Common Usage Patterns

### SaaS Copilot Integration

```typescript
import { PageAgent } from 'page-agent'

class SaaSCopilot {
  private agent: PageAgent
  
  constructor() {
    this.agent = new PageAgent({
      model: 'gpt-4',
      baseURL: process.env.LLM_BASE_URL,
      apiKey: process.env.LLM_API_KEY,
      language: 'en-US',
    })
  }
  
  async handleUserCommand(command: string) {
    try {
      const result = await this.agent.execute(command)
      return { success: true, result }
    } catch (error) {
      console.error('Copilot error:', error)
      return { success: false, error: error.message }
    }
  }
  
  async autoFillForm(formData: Record<string, string>) {
    const commands = Object.entries(formData).map(
      ([field, value]) => `Fill ${field} with ${value}`
    )
    
    for (const command of commands) {
      await this.agent.execute(command)
    }
  }
}

// Usage
const copilot = new SaaSCopilot()
await copilot.handleUserCommand('Create a new project named "Website Redesign"')
```

### Form Automation

```typescript
import { PageAgent } from 'page-agent'

async function automateFormFilling() {
  const agent = new PageAgent({
    model: 'qwen3.5-plus',
    baseURL: 'https://dashscope.aliyuncs.com/compatible-mode/v1',
    apiKey: process.env.DASHSCOPE_API_KEY,
  })
  
  // Smart form filling with natural language
  await agent.execute(`
    Fill out the registration form:
    - First name: John
    - Last name: Doe
    - Email: john.doe@example.com
    - Password: Use a strong password
    - Check the terms and conditions checkbox
    - Click submit
  `)
}
```

### Accessibility Enhancement

```typescript
import { PageAgent } from 'page-agent'

class AccessibilityAgent {
  private agent: PageAgent
  
  constructor() {
    this.agent = new PageAgent({
      model: 'gpt-4',
      baseURL: process.env.OPENAI_BASE_URL,
      apiKey: process.env.OPENAI_API_KEY,
      language: 'en-US',
    })
  }
  
  async handleVoiceCommand(voiceTranscript: string) {
    // Convert voice commands to actions
    await this.agent.execute(voiceTranscript)
  }
  
  async describeCurrentPage() {
    // Use agent to describe page content for screen readers
    const description = await this.agent.execute(
      'Describe what is visible on this page'
    )
    return description
  }
}
```

### Multi-Step Workflow

```typescript
import { PageAgent } from 'page-agent'

async function complexWorkflow() {
  const agent = new PageAgent({
    model: 'claude-3-5-sonnet-20241022',
    baseURL: 'https://api.anthropic.com/v1',
    apiKey: process.env.ANTHROPIC_API_KEY,
  })
  
  // Execute complex multi-step task
  await agent.execute(`
    1. Navigate to the products page
    2. Filter by category "Electronics"
    3. Sort by price (low to high)
    4. Add the first three items to cart
    5. Go to checkout
  `)
}
```

### Error Handling and Retry

```typescript
import { PageAgent } from 'page-agent'

async function executeWithRetry(command: string, maxRetries = 3) {
  const agent = new PageAgent({
    model: 'gpt-4',
    baseURL: process.env.OPENAI_BASE_URL,
    apiKey: process.env.OPENAI_API_KEY,
  })
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await agent.execute(command)
      return { success: true, result }
    } catch (error) {
      console.error(`Attempt ${attempt} failed:`, error)
      
      if (attempt === maxRetries) {
        return { success: false, error: error.message }
      }
      
      // Wait before retry
      await new Promise(resolve => setTimeout(resolve, 1000 * attempt))
    }
  }
}
```

## Browser Extension (Multi-Page Tasks)

For cross-tab automation, Page Agent provides a Chrome extension:

```typescript
// In your extension background script
import { PageAgent } from 'page-agent'

const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: process.env.OPENAI_BASE_URL,
  apiKey: process.env.OPENAI_API_KEY,
})

// Execute commands across multiple tabs
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'EXECUTE_COMMAND') {
    agent.execute(message.command)
      .then(result => sendResponse({ success: true, result }))
      .catch(error => sendResponse({ success: false, error: error.message }))
    return true // Keep channel open for async response
  }
})
```

## MCP Server (Beta)

Page Agent includes an MCP (Model Context Protocol) server for external control:

```bash
# Start MCP server
npx page-agent-mcp
```

Configure in your MCP client (e.g., Claude Desktop):

```json
{
  "mcpServers": {
    "page-agent": {
      "command": "npx",
      "args": ["page-agent-mcp"]
    }
  }
}
```

## Programmatic API

### Event Listeners

```typescript
const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: process.env.OPENAI_BASE_URL,
  apiKey: process.env.OPENAI_API_KEY,
})

// Listen to execution events
agent.on('step', (stepData) => {
  console.log('Agent step:', stepData)
})

agent.on('complete', (result) => {
  console.log('Execution complete:', result)
})

agent.on('error', (error) => {
  console.error('Agent error:', error)
})
```

### Custom Actions

```typescript
import { PageAgent } from 'page-agent'

const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: process.env.OPENAI_BASE_URL,
  apiKey: process.env.OPENAI_API_KEY,
})

// Register custom action
agent.registerAction('sendEmail', async (params) => {
  // Custom email sending logic
  await sendEmail(params.to, params.subject, params.body)
  return { success: true }
})

// Use custom action
await agent.execute('Send an email to team@example.com with subject "Update"')
```

## Troubleshooting

### Agent Not Finding Elements

**Problem**: Agent fails to locate buttons or form fields.

**Solutions**:
- Ensure elements have proper labels or accessible names
- Check that elements are visible (not `display: none` or `visibility: hidden`)
- Increase `maxElements` in configuration
- Use more specific descriptions in commands

```typescript
const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: process.env.OPENAI_BASE_URL,
  apiKey: process.env.OPENAI_API_KEY,
  elementSelector: {
    includeInvisible: false,
    maxElements: 200, // Increase if needed
  },
})
```

### LLM API Errors

**Problem**: API key or connection errors.

**Solutions**:
- Verify API key is correctly set: `console.log(process.env.OPENAI_API_KEY)`
- Check baseURL matches your provider
- Ensure model name is correct for your provider
- Check network connectivity and CORS settings

```typescript
// Debug API configuration
const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: process.env.OPENAI_BASE_URL,
  apiKey: process.env.OPENAI_API_KEY,
  debug: true, // Enable debug logging
})
```

### Timeout Issues

**Problem**: Commands timeout before completion.

**Solutions**:
- Increase timeout value
- Break complex commands into smaller steps
- Optimize page performance

```typescript
const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: process.env.OPENAI_BASE_URL,
  apiKey: process.env.OPENAI_API_KEY,
  timeout: 60000, // 60 seconds
  maxSteps: 30,
})
```

### CSP (Content Security Policy) Issues

**Problem**: Script blocked by CSP when using CDN.

**Solutions**:
- Add Page Agent CDN to your CSP header
- Use NPM package instead of CDN
- Update CSP meta tag

```html
<meta http-equiv="Content-Security-Policy" 
      content="script-src 'self' https://cdn.jsdelivr.net;">
```

### Language/Locale Issues

**Problem**: Agent responds in wrong language.

**Solution**: Set explicit language configuration

```typescript
const agent = new PageAgent({
  model: 'gpt-4',
  baseURL: process.env.OPENAI_BASE_URL,
  apiKey: process.env.OPENAI_API_KEY,
  language: 'en-US', // or 'zh-CN'
})
```

## Best Practices

1. **Use Environment Variables**: Never hardcode API keys
2. **Specific Commands**: More specific natural language commands work better
3. **Error Handling**: Always wrap execute() calls in try-catch
4. **Rate Limiting**: Implement delays between commands if making many requests
5. **Element Visibility**: Ensure target elements are visible before commands execute
6. **Security**: Validate and sanitize user input before passing to agent
7. **Testing**: Test with demo LLM first before production deployment

## Resources

- **Documentation**: https://alibaba.github.io/page-agent/docs/introduction/overview
- **GitHub**: https://github.com/alibaba/page-agent
- **NPM Package**: https://www.npmjs.com/package/page-agent
- **License**: MIT
