---
name: cowagent-ai-assistant
description: Build and deploy autonomous AI agents with CowAgent - planning, memory, knowledge base, skills, and multi-channel support
triggers:
  - how do I set up a CowAgent AI assistant
  - help me configure CowAgent with Claude or GPT
  - create a custom skill for CowAgent
  - integrate CowAgent with WeChat or Feishu
  - configure CowAgent memory and knowledge base
  - add tools and MCP servers to CowAgent
  - deploy CowAgent with Docker
  - troubleshoot CowAgent installation issues
---

# CowAgent AI Assistant

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

CowAgent is an open-source autonomous AI assistant framework that plans tasks, executes tools and skills, and grows through memory and knowledge. It supports multiple LLM providers (Claude, GPT, Gemini, DeepSeek, etc.) and channels (Web, WeChat, Feishu, DingTalk), with a three-tier memory architecture and personal knowledge base.

## Installation

### Quick Install (Recommended)

**Linux / macOS:**
```bash
bash <(curl -fsSL https://cdn.link-ai.tech/code/cow/run.sh)
```

**Windows (PowerShell):**
```powershell
irm https://cdn.link-ai.tech/code/cow/run.ps1 | iex
```

**Docker:**
```bash
curl -O https://cdn.link-ai.tech/code/cow/docker-compose.yml
docker compose up -d
```

### Manual Installation from Source

```bash
# Clone repository
git clone https://github.com/zhayujie/CowAgent.git
cd CowAgent

# Install dependencies (Python 3.8+)
pip3 install -r requirements.txt

# Copy and configure
cp config-template.json config.json

# Start the agent
python3 app.py
```

After starting, access the Web console at `http://localhost:9899`.

## CLI Commands

The `cow` CLI manages the CowAgent service:

```bash
# Service control
cow start              # Start CowAgent
cow stop               # Stop CowAgent
cow restart            # Restart CowAgent
cow status             # Check service status
cow logs               # View logs

# Updates and skills
cow update             # Pull latest code and restart
cow skill install <name>   # Install a skill from Skill Hub
cow install-browser    # Install browser automation dependencies

# Usage examples
cow skill list         # List installed skills
cow skill search weather   # Search for skills
```

## Configuration

### Model Configuration

Configure via Web console (recommended) or manually edit `config.json`:

```json
{
  "model": "claude-opus-4",
  "claude_api_key": "${CLAUDE_API_KEY}",
  "openai_api_key": "${OPENAI_API_KEY}",
  "gemini_api_key": "${GEMINI_API_KEY}",
  
  "vision_model": "gpt-4o",
  "image_create_model": "dall-e-3",
  "speech_recognition_model": "whisper-1",
  "text_to_speech_model": "tts-1",
  "embedding_model": "text-embedding-3-small"
}
```

### Channel Configuration

Set `channel_type` to switch channels:

```json
{
  "channel_type": "wx",  // Options: terminal, wx, web, feishu, dingtalk, wecom_bot, qq
  
  // Web channel (default)
  "web": {
    "port": 9899,
    "admin_password": "your_password"
  },
  
  // WeChat
  "wechat": {
    "single_chat_prefix": ["bot", "@bot"],
    "single_chat_reply_prefix": "[bot] ",
    "group_chat_prefix": ["@bot"],
    "group_name_white_list": ["ChatGroup1", "ChatGroup2"]
  },
  
  // Feishu
  "feishu": {
    "app_id": "${FEISHU_APP_ID}",
    "app_secret": "${FEISHU_APP_SECRET}"
  }
}
```

### Memory Configuration

```json
{
  "memory": {
    "enable_long_term": true,
    "deep_dream_time": "03:00",  // Daily Deep Dream time
    "max_context_messages": 20,
    "enable_hybrid_search": true
  }
}
```

### Knowledge Base Configuration

```json
{
  "knowledge": {
    "enable": true,
    "auto_curate": true,
    "update_threshold": 3
  }
}
```

## Skills System

### Installing Skills

**Via CLI:**
```bash
cow skill install weather
cow skill install stock-query
cow skill install github-repo-search
```

**Via Chat:**
```
/skill search weather
/skill install weather
/skill list
```

### Creating Custom Skills

Skills are defined in a `skill.json` manifest:

```json
{
  "name": "custom-api-caller",
  "version": "1.0.0",
  "description": "Call external API and process results",
  "author": "Your Name",
  "triggers": ["call api", "fetch data from api"],
  "parameters": [
    {
      "name": "endpoint",
      "type": "string",
      "description": "API endpoint URL",
      "required": true
    },
    {
      "name": "method",
      "type": "string",
      "description": "HTTP method (GET/POST)",
      "default": "GET"
    }
  ],
  "steps": [
    {
      "action": "web_fetch",
      "params": {
        "url": "{{endpoint}}",
        "method": "{{method}}"
      }
    },
    {
      "action": "write",
      "params": {
        "path": "result.json",
        "content": "{{web_fetch.response}}"
      }
    }
  ]
}
```

Place in `skills/custom-api-caller/skill.json` and restart.

### Conversational Skill Creation

Use the built-in `skill-creator` skill:

```
Create a skill that fetches GitHub repository info and saves it to a markdown file.
```

The agent will generate the skill manifest interactively.

## Tools System

### Built-in Tools

**File Operations:**
```python
# Agent uses these tools automatically
read(path="/path/to/file.txt")
write(path="output.txt", content="data")
edit(path="config.json", replacements=[{"old": "value1", "new": "value2"}])
ls(path="./data")
```

**Terminal:**
```python
bash(command="ls -la")
bash(command="python script.py")
```

**Memory & Knowledge:**
```python
memory(query="what did user say about project X")
knowledge_search(query="API documentation")
```

**Web & Browser:**
```python
web_fetch(url="https://api.example.com/data")
web_search(query="Python async best practices")
browser(action="navigate", url="https://example.com")
browser(action="click", selector="#submit-button")
```

**Scheduling:**
```python
scheduler(action="add", time="2026-05-25 14:00", task="Send report")
scheduler(action="list")
```

### MCP Integration

Configure MCP servers in `mcp.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"],
      "transport": "stdio"
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "transport": "stdio",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"],
      "transport": "stdio"
    }
  }
}
```

Hot reload: edit `mcp.json` and restart CowAgent.

## Working Code Examples

### Python Plugin Example

Create a custom plugin in `plugins/my_plugin.py`:

```python
import plugins
from bridge.context import ContextType
from bridge.reply import Reply, ReplyType
from common.log import logger

@plugins.register(
    name="MyPlugin",
    desc="Custom functionality plugin",
    version="1.0",
    author="Your Name"
)
class MyPlugin(plugins.Plugin):
    def __init__(self):
        super().__init__()
        self.handlers[Event.ON_HANDLE_CONTEXT] = self.on_handle_context
        logger.info("[MyPlugin] initialized")

    def on_handle_context(self, e_context: EventContext):
        context = e_context['context']
        if context.type != ContextType.TEXT:
            return
        
        content = context.content.strip()
        
        if content.startswith("/hello"):
            reply = Reply()
            reply.type = ReplyType.TEXT
            reply.content = "Hello from custom plugin!"
            e_context['reply'] = reply
            e_context.action = EventAction.BREAK_PASS
            return

    def get_help_text(self, **kwargs):
        return "MyPlugin: Use /hello to get a greeting"
```

### Custom Channel Implementation

```python
from channel.channel import Channel
from bridge.context import Context, ContextType
from bridge.reply import Reply

class CustomChannel(Channel):
    def __init__(self):
        super().__init__()
        
    def startup(self):
        # Initialize your channel (websocket, HTTP server, etc.)
        logger.info("[CustomChannel] starting...")
        
    def handle_message(self, message):
        context = Context()
        context.type = ContextType.TEXT
        context.content = message['text']
        context['session_id'] = message['user_id']
        
        # Process through agent
        reply = super().build_reply_content(message['text'], context)
        
        # Send reply through your channel
        self.send_message(message['user_id'], reply.content)
        
    def send_message(self, user_id, content):
        # Implement sending logic
        pass
```

Register in `channel/channel_factory.py`:

```python
from channel.custom.custom_channel import CustomChannel

def create_channel(channel_type):
    if channel_type == "custom":
        return CustomChannel()
    # ... existing channels
```

### Advanced Skill with Multiple Tools

```json
{
  "name": "github-issue-reporter",
  "version": "1.0.0",
  "description": "Search GitHub repos, analyze issues, generate report",
  "triggers": ["analyze github issues", "report on github repository"],
  "parameters": [
    {
      "name": "repo",
      "type": "string",
      "description": "GitHub repository (owner/repo)",
      "required": true
    }
  ],
  "steps": [
    {
      "action": "web_fetch",
      "params": {
        "url": "https://api.github.com/repos/{{repo}}/issues",
        "headers": {
          "Authorization": "token ${GITHUB_TOKEN}"
        }
      },
      "output": "issues_data"
    },
    {
      "action": "bash",
      "params": {
        "command": "echo '{{issues_data}}' | jq '[.[] | {title: .title, state: .state, comments: .comments}]' > /tmp/issues.json"
      }
    },
    {
      "action": "read",
      "params": {
        "path": "/tmp/issues.json"
      },
      "output": "processed_issues"
    },
    {
      "action": "write",
      "params": {
        "path": "github_report_{{repo | replace('/', '_')}}.md",
        "content": "# GitHub Issues Report for {{repo}}\n\n{{processed_issues}}\n\nGenerated at {{now}}"
      }
    }
  ]
}
```

## Common Patterns

### Agent Planning Loop

The agent follows a plan-execute-reflect loop:

1. **Plan**: Decompose user request into subtasks
2. **Execute**: Run tools and skills step by step
3. **Reflect**: Check if goal achieved, adjust plan
4. **Loop**: Continue until task complete

### Memory Retrieval Pattern

```python
# Agent automatically searches memory when relevant
# Manual retrieval in custom code:
from plugins import memory_search

results = memory_search(
    query="user's favorite programming language",
    limit=5
)
```

### Knowledge Base Update Pattern

```python
# Agent auto-curates during conversation
# Manual update:
from plugins import knowledge_update

knowledge_update(
    topic="Project Setup",
    content="New setup steps: 1. Install deps 2. Configure .env",
    operation="append"
)
```

### Multi-Step Workflow Pattern

For complex workflows, chain tools in skills:

```json
{
  "steps": [
    {"action": "web_fetch", "params": {"url": "..."}},
    {"action": "bash", "params": {"command": "process.sh"}},
    {"action": "read", "params": {"path": "result.txt"}},
    {"action": "memory", "params": {"action": "save", "content": "{{read.content}}"}}
  ]
}
```

## Troubleshooting

### Installation Issues

**Python version mismatch:**
```bash
python3 --version  # Ensure 3.8+
pip3 install --upgrade pip setuptools wheel
```

**Missing dependencies:**
```bash
pip3 install -r requirements.txt --force-reinstall
```

**Port already in use:**
```json
{
  "web": {
    "port": 9900  // Change from default 9899
  }
}
```

### Model Configuration Issues

**API key not working:**
- Ensure environment variables are set: `export CLAUDE_API_KEY=sk-...`
- Or set in `config.json` directly (not recommended for production)
- Check key has proper permissions and quota

**Model not responding:**
```bash
cow logs  # Check for API errors
```

Common fixes:
- Verify `model` field matches provider's model name
- Check provider-specific API key field (`claude_api_key`, `openai_api_key`, etc.)
- Test with curl:
```bash
curl -X POST https://api.anthropic.com/v1/messages \
  -H "x-api-key: ${CLAUDE_API_KEY}" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-opus-4","messages":[{"role":"user","content":"test"}],"max_tokens":100}'
```

### Channel Connection Issues

**WeChat not connecting:**
- QR code expired: restart and scan new code within 60 seconds
- Check `wechat` config in `config.json`
- Ensure network allows WeChat web protocol

**Feishu/Lark setup:**
- Verify `app_id` and `app_secret` from Feishu admin console
- Enable bot capabilities in app settings
- Add bot to target groups before testing

### Memory & Knowledge Issues

**Deep Dream not running:**
- Check `memory.deep_dream_time` in config
- Ensure agent is running at scheduled time
- Verify sufficient conversation history

**Knowledge not updating:**
- Set `knowledge.auto_curate: true`
- Check `knowledge.update_threshold` (default 3 relevant exchanges)

### Skill Issues

**Skill not triggering:**
- Check `triggers` in `skill.json` match user input
- List skills: `/skill list`
- Reinstall: `cow skill install <name>`

**Skill execution fails:**
```bash
cow logs  # Check for tool errors
```

Verify:
- Required tools are available
- Parameters match schema
- File paths are accessible

### Browser Tool Issues

**Browser not installed:**
```bash
cow install-browser
```

**Headless mode issues:**
```json
{
  "browser": {
    "headless": false  // Debug with visible browser
  }
}
```

### Performance Optimization

**Slow responses:**
- Use faster models: `claude-sonnet-4`, `gpt-4o-mini`, `deepseek-v4-flash`
- Reduce `memory.max_context_messages`
- Disable unused features

**High memory usage:**
- Limit conversation history
- Disable Deep Dream if not needed
- Restart agent daily: `0 3 * * * cow restart`

### Logs and Debugging

```bash
cow logs               # View recent logs
cow logs -f            # Follow logs in real-time
tail -f logs/app.log   # Direct log access
```

Enable debug mode in `config.json`:
```json
{
  "debug": true,
  "log_level": "DEBUG"
}
```

For issues, check:
1. `cow status` - service running
2. `cow logs` - error messages
3. Config validation: ensure JSON is valid
4. Port conflicts: `netstat -tuln | grep 9899`
