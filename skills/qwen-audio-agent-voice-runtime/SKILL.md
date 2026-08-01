---
name: qwen-audio-agent-voice-runtime
description: Real-time voice runtime for AI agents with full-duplex conversation, background task execution, and continuous presence
triggers:
  - how do I set up qwen audio agent for voice conversations
  - integrate real-time voice chat with AI agents
  - configure qwen audio agent with background task support
  - add voice interaction to my AI agent workflow
  - set up full-duplex voice agent runtime
  - connect qwen audio agent to backend agents like OpenClaw
  - use qwen audio agent for continuous voice conversations
  - debug qwen audio agent voice streaming issues
---

# Qwen Audio Agent Voice Runtime

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

Qwen Audio Agent is a real-time voice runtime that enables AI agents to maintain continuous voice conversations while executing background tasks. Unlike traditional chatbots that pause during task execution, this runtime keeps agents "present" - listening, responding, and updating users about task progress naturally.

**Core capabilities:**
- Full-duplex real-time voice interaction with natural interruption support
- Parallel frontend conversation and background task execution
- Automatic task result integration back into ongoing conversations
- Multiple backend agent support (OpenCode, OpenClaw, Qoder, Kimi Code, etc.)
- Cross-session memory and user profiles
- Multiple interfaces: WebUI, Terminal TUI, macOS desktop app

## Installation

### Requirements
- Node.js 22.22.2+ or 24.15.0+
- npm 10+
- DashScope API Key (Alibaba Cloud)

### Global Installation (Recommended)

```bash
# From npm
npm install -g qwen-audio-agent

# From GitHub (latest)
npm install -g git+https://github.com/QwenAudio/qwen-audio-agent.git
```

### From Source

```bash
git clone https://github.com/QwenAudio/qwen-audio-agent.git
cd qwen-audio-agent
npm install
npm run install:global
```

### Upgrading

```bash
# NPM version
npm install -g qwen-audio-agent@latest

# GitHub latest
npm install -g git+https://github.com/QwenAudio/qwen-audio-agent.git
```

## Configuration

### Get DashScope API Key

1. Visit [Alibaba Cloud Model Studio API Key page](https://bailian.console.aliyun.com/?tab=model#/api-key)
2. Create an API Key
3. Copy the key for configuration

### Initial Setup

```bash
# Create config file
qwenaudio config
```

This creates `~/.config/qwaudio/config.env`. Edit with your settings:

```dotenv
# Required: DashScope API Key
DASHSCOPE_API_KEY=sk-your-key-here

# Voice model: qwen-audio-3.0-realtime-flash or qwen-audio-3.0-realtime-plus
QWEN_AUDIO_REALTIME_MODEL=qwen-audio-3.0-realtime-plus

# Optional: Backend agent (leave empty or 'none' for frontend-only mode)
AGENT_PROTOCOL=openclaw

# Optional: Backend model (leave empty to use agent's own config)
QWEN_AUDIO_AGENT_BACKEND_MODEL=qwen3.7-max
```

### Backend Agent Options

| Agent | Protocol Value | Auto-Install | Notes |
|-------|---------------|--------------|-------|
| None | `none` or empty | N/A | Frontend-only mode |
| OpenCode | `opencode` | Yes | Auto-install, auto-config with DashScope |
| OpenClaw | `openclaw` | Yes | Auto-install, auto-config with DashScope |
| Qoder | `qoder` | No | User must install/configure |
| Kimi Code | `kimi-code` | No | User must install/configure |
| Hermes | `hermes` | No | User must install/configure |

View available backends:

```bash
qwenaudio setup
```

### Custom ACP Backend

For any ACP-compatible agent:

```dotenv
AGENT_PROTOCOL=acp
ACP_COMMAND=your-agent-command
ACP_ARGS=["--acp", "--other-args"]
ACP_LABEL=Custom Agent
ACP_WORKSPACE=/path/to/workspace
```

### Permission Modes

```dotenv
# native: Agent prompts for permission (default, recommended)
QWEN_AUDIO_AGENT_BACKEND_PERMISSION_MODE=native

# full: Auto-approve all commands/file changes (trusted projects only!)
QWEN_AUDIO_AGENT_BACKEND_PERMISSION_MODE=full
```

## Basic Usage

### Terminal Interface (TUI)

```bash
# Start Gateway in one terminal
qwenaudio

# Start TUI in another terminal
qwenaudio tui
```

**Audio modes:**

```bash
# Half-duplex (default on Linux/Windows)
qwenaudio tui --audio-mode half

# Full-duplex without echo cancellation (requires headphones)
qwenaudio tui --audio-mode full

# Full-duplex with echo cancellation (macOS default)
qwenaudio tui --audio-mode aec
```

**TUI Controls:**
- `Space`: Start/stop speaking
- `x`: Interrupt agent (during playback)
- `Ctrl+C`: Exit

### Web Interface

```bash
# Start Gateway
qwenaudio

# In another terminal, start WebUI
qwenaudio webui
```

Then open browser to the displayed URL (typically `http://localhost:3003`).

### macOS Desktop App

Download `.dmg` from releases, drag to Applications. The app includes built-in Gateway and auto-manages service lifecycle.

**First run:**
1. App creates config file and shows settings
2. Enter DashScope API Key
3. Choose backend agent (or use frontend-only mode)
4. Start conversation with the floating orb

## Gateway Management

### Run as Background Service

```bash
# Install as user service
qwenaudio gateway install

# Manage service
qwenaudio gateway status
qwenaudio gateway start
qwenaudio gateway stop
qwenaudio gateway restart
qwenaudio gateway uninstall
```

### Manual Gateway Start

```bash
# Default (uses config.env)
qwenaudio

# Frontend-only mode
qwenaudio --backend none

# Specific backend
qwenaudio --backend openclaw

# Custom port
qwenaudio --port 8080

# Dev mode with hot reload
npm run dev
```

## Programming Interface

### Gateway WebSocket API

The Gateway exposes WebSocket endpoints for custom clients:

```javascript
// Connect to Gateway
const ws = new WebSocket('ws://localhost:3002/voice');

// Send audio chunks (PCM16, 16kHz, mono)
ws.send(audioBuffer);

// Receive responses
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  switch (data.type) {
    case 'audio':
      // Play audio chunk
      playAudioChunk(data.audio);
      break;
    case 'transcript':
      // Display user/assistant text
      console.log(data.role, data.text);
      break;
    case 'task_created':
      // Background task started
      console.log('Task:', data.task_id);
      break;
    case 'task_completed':
      // Task finished
      console.log('Result:', data.result);
      break;
  }
};
```

### Direct Integration (Library Mode)

```javascript
import { Gateway } from 'qwen-audio-agent';

const gateway = new Gateway({
  dashscopeApiKey: process.env.DASHSCOPE_API_KEY,
  model: 'qwen-audio-3.0-realtime-plus',
  backendAgent: 'openclaw',
  backendModel: 'qwen3.7-max'
});

await gateway.start();

// Handle voice stream
gateway.on('audio', (chunk) => {
  // Process audio output
});

gateway.on('transcript', (data) => {
  console.log(`${data.role}: ${data.text}`);
});

// Send audio input
gateway.sendAudio(pcmBuffer);
```

## User Profile and Memory

### File Locations

```
~/.config/qwaudio/
├── config.env              # Configuration
├── USER.md                 # User profile (name, location, preferences)
├── frontend-memory.json    # Long-term memory
└── tasks.json              # Task history
```

### USER.md Example

```markdown
# User Profile

## Basic Info
- Name: Alex
- Location: San Francisco, CA
- Timezone: PST

## Preferences
- Preferred language: English
- Communication style: Direct and concise
- Working hours: 9am-6pm PST

## Common Projects
- ~/projects/my-app (Node.js web app)
- ~/work/client-project (Python ML pipeline)

## Notes
- Prefers VS Code for JavaScript
- Uses PyCharm for Python
```

### Memory Management in Conversation

```
User: "Remember that I prefer using TypeScript for new projects"
Agent: "I'll remember that you prefer TypeScript for new projects."

User: "What are my preferences for new projects?"
Agent: "You prefer using TypeScript for new projects."

User: "Forget that preference"
Agent: "I've removed that from my memory."
```

## Common Patterns

### Frontend-Only Mode (Simple Voice Chat)

```bash
# No backend tasks, just voice conversation
qwenaudio --backend none
qwenaudio tui
```

Use case: Simple voice interaction without tool use or task execution.

### With Background Agent (Task Execution)

```bash
# OpenClaw for web/file tasks
qwenaudio --backend openclaw
qwenaudio tui
```

```
User: "Create a new React component called UserProfile"
Agent: "I'll create that component for you."
[Task executes in background]
Agent: "I've created the UserProfile component in src/components/UserProfile.jsx"

User: "Add PropTypes validation"
Agent: "I'll add PropTypes to the UserProfile component."
```

### Multi-Task Workflow

```
User: "Start a web server on port 3000 and fetch the latest data from the API"
Agent: "I'll start the server and fetch the data for you."

[Two tasks created and executed in parallel]

Agent: "The server is running on port 3000."
Agent: "I've fetched the latest data - 127 records retrieved."
```

### Task Progress Tracking

```
User: "Download all the images from that webpage"
Agent: "I'm downloading the images now."

[5 seconds later]
User: "How's it going?"
Agent: "I've downloaded 15 out of 42 images so far."

[Later]
Agent: "All 42 images have been downloaded to the images folder."
```

## Development

### Project Structure

```
qwen-audio-agent/
├── src/
│   ├── gateway/          # Core Gateway server
│   ├── frontend/         # Voice interaction handler
│   ├── backend/          # Backend agent connectors
│   ├── webui/           # Web interface
│   ├── tui/             # Terminal interface
│   └── desktop/         # macOS desktop app
├── config.env.example   # Configuration template
└── package.json
```

### Run from Source

```bash
npm install
npm run build

# Gateway + WebUI dev mode (hot reload)
npm run dev

# TUI in another terminal
npm run tui:dev

# macOS desktop app
npm run desktop
```

### Build Desktop App

```bash
# Local test build
npm run desktop:build:local

# Production build
npm run desktop:build
```

## Troubleshooting

### Audio Issues

**No audio input/output on Linux:**

```bash
# Install PortAudio
sudo apt-get install portaudio19-dev  # Debian/Ubuntu
sudo dnf install portaudio-devel      # Fedora

# Install Python sounddevice
pip install sounddevice
```

**Echo/feedback in full-duplex mode:**
- Use headphones
- Switch to half-duplex: `qwenaudio tui --audio-mode half`
- On macOS, use AEC mode: `qwenaudio tui --audio-mode aec`

**Microphone not working:**

```bash
# Test microphone access
qwenaudio tui --debug

# Check permissions (macOS)
# System Settings > Privacy & Security > Microphone
```

### Backend Agent Issues

**Agent not found:**

```bash
# List available backends
qwenaudio setup

# Install missing agent
npm install -g @agentclientprotocol/codex-acp  # Example

# Or let auto-install work (for OpenCode/OpenClaw)
qwenaudio --backend openclaw
```

**Backend model errors:**

```dotenv
# Leave empty to use agent's own config
QWEN_AUDIO_AGENT_BACKEND_MODEL=

# Or specify model
QWEN_AUDIO_AGENT_BACKEND_MODEL=qwen3.7-max
```

**Permission denied errors:**

```dotenv
# Use native prompting (default)
QWEN_AUDIO_AGENT_BACKEND_PERMISSION_MODE=native

# Only for trusted projects:
QWEN_AUDIO_AGENT_BACKEND_PERMISSION_MODE=full
```

### Gateway Connection Issues

**Cannot connect to Gateway:**

```bash
# Check Gateway is running
qwenaudio gateway status

# Restart Gateway
qwenaudio gateway restart

# Check port availability
lsof -i :3002  # macOS/Linux
netstat -ano | findstr :3002  # Windows
```

**WebSocket connection failed:**

```bash
# Use correct URL
ws://localhost:3002/voice  # Default

# Check firewall settings
# Gateway only binds to localhost by default
```

### Memory/Performance

**High memory usage:**

```bash
# Clear task history
rm ~/.config/qwaudio/tasks.json

# Clear frontend memory
rm ~/.config/qwaudio/frontend-memory.json

# Restart Gateway
qwenaudio gateway restart
```

**Slow response:**

```dotenv
# Use faster model
QWEN_AUDIO_REALTIME_MODEL=qwen-audio-3.0-realtime-flash

# Reduce backend model size
QWEN_AUDIO_AGENT_BACKEND_MODEL=qwen-turbo
```

## Configuration Reference

### Environment Variables

```dotenv
# Required
DASHSCOPE_API_KEY=sk-xxx

# Voice Model
QWEN_AUDIO_REALTIME_MODEL=qwen-audio-3.0-realtime-plus

# Backend Agent
AGENT_PROTOCOL=openclaw  # or: opencode, qoder, none, acp
QWEN_AUDIO_AGENT_BACKEND_MODEL=qwen3.7-max

# Custom ACP Backend
ACP_COMMAND=my-agent
ACP_ARGS=["--acp"]
ACP_LABEL=My Custom Agent
ACP_WORKSPACE=/path/to/workspace

# Permissions
QWEN_AUDIO_AGENT_BACKEND_PERMISSION_MODE=native  # or: full

# Network (advanced)
GATEWAY_PORT=3002
WEBUI_PORT=3003
```

### CLI Options

```bash
# Gateway
qwenaudio [--backend <agent>] [--port <port>]

# TUI
qwenaudio tui [--audio-mode <half|full|aec>]

# WebUI
qwenaudio webui [--port <port>]

# Gateway service
qwenaudio gateway <install|uninstall|start|stop|restart|status>

# Config management
qwenaudio config  # Create config file
qwenaudio setup   # List available backends
```

## Security Considerations

- **Never commit API keys** - Use environment variables only
- **Gateway binds to localhost** - Don't expose to public networks without authentication
- **Full permission mode** - Only use in trusted projects where auto-execution is acceptable
- **User data is local** - Stored in `~/.config/qwaudio/`, not synced to cloud
- **Audio is sent to DashScope** - Real-time audio streams go to configured voice service
- **Backend tasks use external services** - Agents may call APIs, execute commands, modify files

## Example Workflows

### Code Review Assistant

```
User: "Review the latest changes in src/api/"
Agent: "I'll review the code in src/api for you."
[Backend analyzes files]
Agent: "I found 3 potential issues: error handling is missing in two functions, and there's a possible race condition in the cache update logic."

User: "Fix the error handling"
Agent: "I'll add error handling to those functions."
[Task executes]
Agent: "I've added try-catch blocks and proper error propagation."
```

### Data Analysis Flow

```
User: "Analyze the sales data from last quarter"
Agent: "I'll analyze the Q3 sales data."
[Backend processes data]
Agent: "Revenue increased 23% compared to Q2, with the strongest growth in the enterprise segment."

User: "Create a chart showing monthly trends"
Agent: "I'll create that chart."
[Task generates visualization]
Agent: "I've created the monthly trend chart and saved it to reports/q3-trends.png"
```

### Multi-Step Project Setup

```
User: "Set up a new Next.js project with TypeScript and Tailwind"
Agent: "I'll create a Next.js project with TypeScript and Tailwind CSS."
[Multiple tasks execute]
Agent: "I've initialized the project, configured TypeScript, and installed Tailwind CSS. The dev server is ready to start."

User: "Add a homepage component"
Agent: "I'll create a homepage component."
[Task completes]
Agent: "I've created a responsive homepage component at app/page.tsx using Tailwind."
```
