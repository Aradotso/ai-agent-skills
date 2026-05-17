---
name: openmonoagent-local-ai-coding-agent
description: Local-first AI coding agent powered by llama.cpp with zero tokens costs, Docker sandboxing, 20 built-in tools, LSP/Roslyn intelligence, and MCP integration
triggers:
  - set up openmonoagent locally
  - run a local ai coding agent
  - use openmono with my project
  - configure openmonoagent with custom model
  - create an openmonoagent playbook
  - troubleshoot openmono inference
  - use openmonoagent tools programmatically
  - run openmono in dual-box mode
---

# OpenMonoAgent.ai — Local AI Coding Agent

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

OpenMonoAgent is a terminal-native coding agent that runs entirely on your hardware using local LLMs via llama.cpp. Zero per-token costs, full Docker sandboxing, 20 built-in tools, LSP/Roslyn code intelligence, and MCP integration. Built on C#/.NET 10.

## What It Does

- **Local inference**: Bundles llama.cpp for zero-config, zero-cost token generation
- **Agentic loop**: 25 iterations per turn with doom-loop detection and automatic checkpointing
- **20 tools**: File operations, code analysis, Docker commands, git, search, Roslyn diagnostics
- **5 specialist sub-agents**: Explore, Plan, Coder, Verify, general-purpose with isolated tool sets
- **Docker sandbox**: Project mounts as `/workspace`, blast radius limited to project directory
- **Code intelligence**: Roslyn for C#, LSP for TypeScript/Python/Go/Rust, auto-detects graphify/MCP tools
- **Playbooks**: YAML workflows with typed parameters, gates, and checkpoint/resume
- **Dual-box mode**: Run agent on laptop, inference on separate GPU machine

## Installation

### Quick Install (Ubuntu 26.04 LTS or 25.10)

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/StartupHakk/OpenMonoAgent.ai/refs/heads/main/get-openmono.sh)
```

The installer:
- Detects your hardware (GPU VRAM or CPU RAM)
- Selects the appropriate model automatically
- Installs Docker if missing
- Pulls and configures the llama.cpp inference container

### Hardware Requirements

| VRAM/RAM | Model | Speed |
|----------|-------|-------|
| GPU 24GB+ | Qwen3.6-27B-Q4_K_M | ~45-70 tok/s |
| GPU 16GB | Qwen3.6-27B-UD-IQ3_XXS | ~20-42 tok/s |
| GPU 12GB | Qwen3.5-9B-Q4_K_M | ~38-40 tok/s |
| CPU 24GB RAM | Qwen3.6-35B-A3B-UD-Q4_K_XL | ~17-20 tok/s |

## Basic Usage

### Starting the Agent

```bash
# TUI mode (default, interactive)
cd your-project/
openmono agent

# Classic scrolling terminal
openmono agent --classic

# Non-interactive mode
openmono agent --non-interactive
```

### Key Commands

```bash
# Check status
openmono status

# List available models
openmono models list

# Switch model
openmono models set qwen3.6-27b-q4

# Run a playbook
openmono playbook run refactor-cleanup.yml

# Export conversation
openmono export --format markdown session-2026-05-17.md
```

### Slash Commands (in TUI/Classic mode)

```
/think          Enter extended reasoning mode
/undo           Revert last tool execution
/resume         Continue from last checkpoint
/export         Export conversation history
/checkpoint     Save current state manually
/tools          List available tools
/config         Show current configuration
/help           Show all commands
```

### Keyboard Shortcuts (TUI mode)

```
Ctrl+C          Cancel current operation
Ctrl+D          Exit agent
Ctrl+L          Clear screen
Ctrl+R          Reload configuration
Tab             Autocomplete command
Up/Down         Navigate history
```

## Configuration

### User-Level Config

`~/.openmono/settings.json`:

```json
{
  "inference": {
    "provider": "llama-cpp",
    "model": "qwen3.6-27b-q4",
    "temperature": 0.7,
    "maxTokens": 8192,
    "reasoningMode": false
  },
  "agent": {
    "maxIterations": 25,
    "doomLoopThreshold": 3,
    "checkpointAt": 0.65,
    "compactAt": 0.80
  },
  "sandbox": {
    "enabled": true,
    "mountPath": "/workspace",
    "networkIsolation": true
  },
  "codeIntelligence": {
    "roslyn": {
      "enabled": true,
      "cacheDuration": 300
    },
    "lsp": {
      "autoStart": true,
      "servers": ["typescript", "python", "go", "rust"]
    },
    "mcp": {
      "enabled": true,
      "autoDetect": ["graphify", "code-review-graph"]
    }
  },
  "tools": {
    "parallelReads": true,
    "maxConcurrent": 5
  }
}
```

### Project-Level Config

`.openmono/settings.json` (overrides user-level):

```json
{
  "inference": {
    "model": "qwen3.6-27b-q4",
    "temperature": 0.6
  },
  "permissions": {
    "allowedPaths": [
      "/workspace/src",
      "/workspace/tests"
    ],
    "deniedPaths": [
      "/workspace/.env",
      "/workspace/secrets"
    ],
    "allowedTools": [
      "read_file",
      "write_file",
      "list_directory",
      "run_command",
      "roslyn_analyze"
    ]
  },
  "mcp": {
    "servers": [
      {
        "name": "custom-graph",
        "command": "node",
        "args": ["/workspace/tools/graph-server.js"],
        "env": {
          "GRAPH_DB": "/workspace/.graph/db.sqlite"
        }
      }
    ]
  }
}
```

## Working with Playbooks

### Basic Playbook Structure

`refactor-api.yml`:

```yaml
name: refactor-api-endpoints
description: Refactor REST API endpoints to use new validation middleware
version: 1.0.0

parameters:
  - name: targetController
    type: string
    required: true
    description: Controller class to refactor
  - name: addLogging
    type: boolean
    default: true
    description: Add structured logging to endpoints

gates:
  - condition: "file_exists('src/Middleware/ValidationMiddleware.cs')"
    message: "Validation middleware must exist before refactoring"

steps:
  - name: analyze-controller
    agent: explore
    prompt: "Analyze {{targetController}} and identify all endpoints that need validation"
    tools:
      - read_file
      - roslyn_analyze
      - list_directory

  - name: plan-refactor
    agent: plan
    prompt: "Create refactoring plan for validation middleware integration"
    checkpoint: true

  - name: apply-changes
    agent: coder
    prompt: |
      Apply refactoring:
      1. Add validation middleware to endpoints
      {% if addLogging %}
      2. Add structured logging with ILogger
      {% endif %}
      3. Update error handling
    tools:
      - read_file
      - write_file
      - roslyn_analyze

  - name: verify-build
    agent: verify
    prompt: "Verify refactored code compiles and passes static analysis"
    tools:
      - run_command
      - roslyn_diagnostics

outputs:
  - modifiedFiles
  - diagnosticsReport
```

### Running Playbooks

```bash
# Run with parameters
openmono playbook run refactor-api.yml \
  --param targetController=UserController \
  --param addLogging=true

# Resume from checkpoint
openmono playbook resume refactor-api.yml --checkpoint 2

# List available playbooks
openmono playbook list

# Validate playbook syntax
openmono playbook validate refactor-api.yml
```

### Composable Playbooks

```yaml
name: full-feature-implementation
description: Implement feature from requirements to tests

steps:
  - name: gather-requirements
    playbook: analyze-requirements.yml
    inputs:
      requirementsDoc: "{{requirementsPath}}"

  - name: implement-feature
    playbook: implement-backend.yml
    inputs:
      spec: "{{outputs.analyze-requirements.specification}}"

  - name: add-tests
    playbook: generate-tests.yml
    inputs:
      targetFiles: "{{outputs.implement-feature.modifiedFiles}}"
```

## Code Intelligence Features

### Roslyn Analysis (C#)

Automatically available for C# projects:

```csharp
// Agent can analyze type hierarchies
public interface IRepository<T> { }
public class UserRepository : IRepository<User> { }

// Agent understands call graphs
public class UserService
{
    private readonly IRepository<User> _repo;
    
    public async Task<User> GetUser(int id)
    {
        // Agent tracks callers and callees
        return await _repo.GetByIdAsync(id);
    }
}

// Agent detects diagnostics
public class ProblematicCode
{
    // CA1052: Static holder types should be Static or NotInheritable
    public class Constants
    {
        public const string ApiKey = "hardcoded"; // Security issue detected
    }
}
```

### LSP Integration

Auto-starts language servers on first use:

```typescript
// TypeScript - tsserver starts automatically
import { Express } from 'express';

export class ApiController {
    // Agent has symbol navigation, type info, references
    async getUser(req: Request, res: Response): Promise<void> {
        // Autocomplete, hover info, go-to-definition available
    }
}
```

```python
# Python - pyright/pylsp starts automatically
from typing import Optional, List
from pydantic import BaseModel

class User(BaseModel):
    # Agent understands type hints, can suggest fixes
    id: int
    name: str
    email: Optional[str] = None
```

### MCP Tool Integration

If `graphify` is installed, agent auto-detects:

```bash
# Agent can use semantic graph queries
"Find all classes that implement the Repository pattern"
"Show me the data flow from API endpoint to database"
"List all usages of the User entity across the codebase"
```

If `code-review-graph` is installed:

```bash
# Agent can use structural analysis
"Analyze the call graph for circular dependencies"
"Find dead code that's never called"
"Show the complexity metrics for this module"
```

## Tool Pipeline

Every tool call goes through 12 steps:

1. **Parse** — Extract tool name and arguments
2. **Schema Validate** — Check against tool schema
3. **Path Sanity** — Verify paths are within `/workspace`
4. **Plan-Mode Guard** — Block writes if in plan/explore agent
5. **Capability Check** — Verify agent has permission
6. **Cache** — Return cached result if available (reads only)
7. **Pre-Hook** — Custom validation logic
8. **Execute** — Run the actual tool
9. **Post-Hook** — Custom processing
10. **Artifact Store** — Save outputs
11. **Log** — Record execution details
12. **Return** — Send result to agent

### Tool Permission Example

```json
{
  "permissions": {
    "tools": {
      "read_file": {
        "enabled": true,
        "maxSize": "10MB",
        "allowedExtensions": [".cs", ".ts", ".py", ".go"]
      },
      "write_file": {
        "enabled": true,
        "maxSize": "1MB",
        "requiresConfirmation": true
      },
      "run_command": {
        "enabled": true,
        "allowedCommands": ["dotnet", "npm", "git"],
        "timeout": 60
      }
    }
  }
}
```

## Dual-Box Setup (Agent on Laptop, Inference on GPU Server)

### On GPU Server

```bash
# Install and expose inference server
bash <(curl -fsSL https://raw.githubusercontent.com/StartupHakk/OpenMonoAgent.ai/refs/heads/main/get-openmono.sh) --server-only

# Expose on network (port 8080)
docker run -d \
  --name openmono-inference \
  --gpus all \
  -p 8080:8080 \
  -v ~/.openmono/models:/models \
  openmono/inference:latest
```

### On Laptop (Agent)

```bash
# Install agent only (no inference)
bash <(curl -fsSL https://raw.githubusercontent.com/StartupHakk/OpenMonoAgent.ai/refs/heads/main/get-openmono.sh) --agent-only

# Configure remote inference
openmono config set inference.endpoint http://gpu-server.local:8080
```

Or use the free relay:

```bash
# On GPU server (register with relay)
openmono relay register

# On laptop (connect via relay)
openmono relay connect --token YOUR_RELAY_TOKEN
```

### Verify Connection

```bash
# Test inference connection
openmono test inference

# Check latency
openmono status --verbose
```

## Programmatic Usage (C# API)

If you want to embed OpenMono in your own .NET application:

```csharp
using OpenMono.Agent;
using OpenMono.Inference;
using OpenMono.Tools;

// Configure inference provider
var inferenceConfig = new InferenceConfig
{
    Provider = InferenceProvider.LlamaCpp,
    Endpoint = "http://localhost:8080",
    Model = "qwen3.6-27b-q4",
    Temperature = 0.7,
    MaxTokens = 8192
};

// Create agent
var agent = new MonoAgent(inferenceConfig);

// Register custom tool
agent.ToolRegistry.Register(new CustomTool
{
    Name = "fetch_api_data",
    Description = "Fetch data from external API",
    Schema = new ToolSchema
    {
        Parameters = new[]
        {
            new Parameter { Name = "endpoint", Type = "string", Required = true },
            new Parameter { Name = "method", Type = "string", Default = "GET" }
        }
    },
    Execute = async (args) =>
    {
        var endpoint = args["endpoint"].ToString();
        var method = args["method"].ToString();
        
        using var client = new HttpClient();
        var response = await client.GetStringAsync(endpoint);
        
        return new ToolResult
        {
            Success = true,
            Output = response
        };
    }
});

// Run agent with prompt
var response = await agent.RunAsync(
    prompt: "Analyze the User model and suggest improvements",
    workspacePath: "/path/to/project",
    maxIterations: 25
);

// Access results
Console.WriteLine($"Completed in {response.Iterations} iterations");
foreach (var artifact in response.Artifacts)
{
    Console.WriteLine($"Generated: {artifact.Path}");
}
```

### Custom Sub-Agent

```csharp
using OpenMono.Agent.SubAgents;

public class SecurityAuditorAgent : SubAgent
{
    public override string Name => "security-auditor";
    public override int MaxTurns => 20;
    
    public override string[] AllowedTools => new[]
    {
        "read_file",
        "list_directory",
        "roslyn_analyze",
        "run_command" // Limited to static analysis tools
    };
    
    public override string SystemPrompt => @"
        You are a security auditor. Your job is to:
        1. Identify security vulnerabilities
        2. Check for hardcoded secrets
        3. Verify dependency versions
        4. Suggest security improvements
        
        You cannot modify code - only report findings.
    ";
    
    protected override async Task<SubAgentResult> ExecuteAsync(
        string userPrompt,
        CancellationToken ct)
    {
        var findings = new List<SecurityFinding>();
        
        // Scan for hardcoded secrets
        var sourceFiles = await ToolExecutor.ExecuteAsync(
            "list_directory",
            new { path = "/workspace", recursive = true, pattern = "*.cs" }
        );
        
        foreach (var file in sourceFiles)
        {
            var content = await ToolExecutor.ExecuteAsync(
                "read_file",
                new { path = file }
            );
            
            if (ContainsHardcodedSecret(content))
            {
                findings.Add(new SecurityFinding
                {
                    Severity = "High",
                    File = file,
                    Issue = "Hardcoded secret detected"
                });
            }
        }
        
        return new SubAgentResult
        {
            Success = true,
            Findings = findings,
            Summary = $"Found {findings.Count} security issues"
        };
    }
}

// Register and use
agent.SubAgents.Register<SecurityAuditorAgent>();

var result = await agent.RunSubAgentAsync(
    "security-auditor",
    "Audit this project for security vulnerabilities"
);
```

## Alternative Inference Providers

### OpenAI (WIP)

```json
{
  "inference": {
    "provider": "openai",
    "model": "gpt-4",
    "apiKey": "${OPENAI_API_KEY}",
    "temperature": 0.7
  }
}
```

```bash
export OPENAI_API_KEY=sk-...
openmono agent
```

### Anthropic (WIP)

```json
{
  "inference": {
    "provider": "anthropic",
    "model": "claude-3-5-sonnet-20241022",
    "apiKey": "${ANTHROPIC_API_KEY}"
  }
}
```

### Ollama (WIP)

```json
{
  "inference": {
    "provider": "ollama",
    "endpoint": "http://localhost:11434",
    "model": "qwen2.5-coder:32b"
  }
}
```

**Note**: OpenAI, Anthropic, and Ollama providers are work-in-progress. Local llama.cpp is the only fully supported provider.

## Troubleshooting

### Agent Won't Start

```bash
# Check Docker status
docker ps | grep openmono

# View logs
docker logs openmono-inference

# Restart inference container
docker restart openmono-inference

# Full reset
openmono reset --hard
```

### Slow Inference

```bash
# Check current model and provider
openmono status

# Verify GPU is being used (if available)
docker exec openmono-inference nvidia-smi

# Switch to smaller model
openmono models set qwen3.5-9b-q4

# Check for CPU throttling
openmono diagnose performance
```

### Tools Not Working

```bash
# List available tools
openmono tools list

# Check tool permissions
openmono config get permissions.tools

# Test specific tool
openmono test tool read_file --path src/Program.cs

# View tool execution logs
openmono logs --filter tools --tail 50
```

### Roslyn/LSP Issues

```bash
# Clear Roslyn cache
rm -rf ~/.openmono/cache/roslyn/*

# Restart LSP servers
openmono lsp restart

# Disable and re-enable
openmono config set codeIntelligence.roslyn.enabled false
openmono config set codeIntelligence.roslyn.enabled true

# Check compilation errors
openmono roslyn compile --verbose
```

### Context Overflow

```bash
# Force checkpoint
/checkpoint

# Reduce max iterations
openmono config set agent.maxIterations 15

# Enable aggressive compaction
openmono config set agent.compactAt 0.70

# Clear conversation and restart
openmono reset --conversation-only
```

### Docker Permission Issues

```bash
# Add user to docker group (requires logout/login)
sudo usermod -aG docker $USER

# Or run with sudo (not recommended)
sudo openmono agent
```

### Network Issues (Dual-Box)

```bash
# Test connection
curl http://gpu-server.local:8080/health

# Check firewall
sudo ufw status
sudo ufw allow 8080/tcp

# Verify relay connection
openmono relay status

# Switch to direct connection
openmono config set inference.endpoint http://gpu-server.local:8080
openmono config set inference.useRelay false
```

## Common Patterns

### Automated Code Review

```bash
openmono agent --classic << 'EOF'
Review all changes in the last commit:
1. Check for code smells
2. Verify error handling
3. Suggest performance improvements
4. Check for security issues

Use the verify sub-agent and Roslyn diagnostics.
/think
EOF
```

### Generate Documentation

```yaml
# generate-docs.yml
name: generate-api-documentation
steps:
  - name: analyze
    agent: explore
    prompt: "Map all public API endpoints and their parameters"
  
  - name: generate
    agent: coder
    prompt: |
      Generate OpenAPI/Swagger documentation:
      1. Create swagger.json
      2. Add XML doc comments to controllers
      3. Generate README.md with usage examples
```

### Refactor with Verification

```yaml
# safe-refactor.yml
name: refactor-with-rollback

parameters:
  - name: targetFile
    type: string
    required: true

steps:
  - name: backup
    agent: general-purpose
    prompt: "Create backup of {{targetFile}}"
  
  - name: refactor
    agent: coder
    prompt: "Refactor {{targetFile}} to improve maintainability"
    checkpoint: true
  
  - name: verify
    agent: verify
    prompt: |
      Verify refactoring:
      1. Code compiles
      2. Tests pass
      3. No new warnings
    
    onFailure:
      rollback: true
      restoreCheckpoint: backup
```

### Dependency Update Audit

```bash
openmono agent
> Update all NuGet packages and verify compatibility:
> 1. Check current package versions
> 2. Update packages one at a time
> 3. Run tests after each update
> 4. Rollback if tests fail
> 5. Generate compatibility report
```

## Best Practices

1. **Use project-level config** for sensitive permissions — don't allow all tools globally
2. **Enable sandboxing** in production — never disable Docker isolation
3. **Set checkpoints** before risky operations — use playbooks with `checkpoint: true`
4. **Limit tool concurrency** for stability — `maxConcurrent: 3-5`
5. **Use sub-agents** for specialized tasks — don't let general-purpose agent write to production
6. **Cache Roslyn analysis** — 5 minute default is good for most projects
7. **Monitor context usage** — enable auto-compaction at 70-80%
8. **Pin model versions** in playbooks — don't rely on "latest"
9. **Use MCP for complex graphs** — don't reinvent semantic analysis
10. **Log everything** — `openmono logs --export` before reporting issues

## Resources

- GitHub: https://github.com/StartupHakk/OpenMonoAgent.ai
- Docs: https://github.com/StartupHakk/OpenMonoAgent.ai/tree/main/docs
- Roadmap: https://github.com/StartupHakk/OpenMonoAgent.ai/blob/main/ROADMAP.md
- Issues: https://github.com/StartupHakk/OpenMonoAgent.ai/issues
- Relay: https://app.openmonoagent.ai
