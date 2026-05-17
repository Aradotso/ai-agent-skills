---
name: pixel-agents-vscode
description: Visualize and manage Claude Code AI agents as pixel art characters in a VS Code extension office interface
triggers:
  - set up pixel agents for claude code
  - visualize my AI agents in VS Code
  - create a pixel office for agents
  - customize agent characters and office layout
  - manage multiple claude code agents visually
  - track agent activity with pixel agents
  - add furniture to my agent office
  - debug agent connections in pixel agents
---

# Pixel Agents VSCode Extension

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Pixel Agents is a VS Code extension that visualizes multi-agent AI systems (currently Claude Code) as pixel art characters in a customizable office environment. Each agent becomes an animated character that reflects its real-time activity — typing when writing code, reading when searching files, and displaying speech bubbles when waiting for input.

## Installation

### From VS Code Marketplace

1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X / Cmd+Shift+X)
3. Search for "Pixel Agents"
4. Click Install

Or install via command line:

```bash
code --install-extension pablodelucca.pixel-agents
```

### From Source

```bash
git clone https://github.com/pablodelucca/pixel-agents.git
cd pixel-agents
npm install
cd webview-ui && npm install && cd ..
npm run build
```

Then press **F5** in VS Code to launch the Extension Development Host.

## Prerequisites

- **VS Code**: Version 1.105.0 or later
- **Claude Code CLI**: Must be installed and configured
  ```bash
  # Install Claude Code (if not already installed)
  npm install -g @anthropic-ai/claude-code
  ```

## Key Features and Usage

### Spawning Agents

Open the **Pixel Agents** panel (appears in bottom panel area alongside terminal):

**Create normal agent:**
```typescript
// Click "+ Agent" button in Pixel Agents panel
// This spawns a new Claude Code terminal with a character
```

**Create agent with skip permissions:**
```typescript
// Right-click "+ Agent" button
// Select "Launch with --dangerously-skip-permissions"
// Agent will bypass all tool approval prompts
```

### Managing Characters

**Assign character to a seat:**
```typescript
// 1. Click a character to select it
// 2. Click an empty seat to reassign the character
```

**View agent status:**
- **Walking**: Agent is navigating to desk/idle
- **Typing**: Writing code, creating files, making changes
- **Reading**: Searching files, analyzing code
- **Speech bubble**: Waiting for user input or permission
- **Idle**: No current activity

### Office Layout Editor

**Open editor:**
```typescript
// Click "Layout" button in Pixel Agents panel
```

**Editor tools:**
- **Select** (S): Select and move furniture
- **Paint** (P): Paint floor tiles
- **Erase** (E): Remove items
- **Place** (F): Add furniture
- **Eyedropper** (I): Pick colors/tiles
- **Pick** (K): Select furniture type

**Keyboard shortcuts:**
- `Ctrl+Z` / `Cmd+Z`: Undo (50 levels)
- `Ctrl+Y` / `Cmd+Y`: Redo
- `Delete`: Remove selected item
- Arrow keys: Fine-tune placement

**Expand grid:**
```typescript
// Click the ghost border outside current grid
// Grid expands up to 64×64 tiles
```

### Configuration

Access settings via gear icon in Pixel Agents panel:

**Sound notifications:**
```typescript
// Toggle sound when agent finishes turn
// Settings → Enable Sound Notifications
```

**Debug view:**
```typescript
// Settings → Debug View
// Shows per-agent diagnostics:
// - JSONL file status
// - Lines parsed
// - Last data timestamp
// - File path
```

**Export/Import layouts:**
```typescript
// Settings → Export Layout (saves as JSON)
// Settings → Import Layout (load from JSON file)
```

**Add external assets:**
```typescript
// Settings → Add Asset Directory
// Point to folder with custom furniture packs
```

## Working with Assets

### Asset Structure

All assets are in `webview-ui/public/assets/`:

```
assets/
├── furniture/
│   ├── desk-01/
│   │   ├── manifest.json
│   │   └── sprite.png
│   └── chair-01/
│       ├── manifest.json
│       └── sprite.png
├── floors/
│   └── tile-wood.png
└── walls/
    └── brick/
        ├── manifest.json
        └── tileset.png
```

### Creating Custom Furniture

**Manifest structure:**

```json
{
  "id": "desk-modern",
  "name": "Modern Desk",
  "category": "desks",
  "width": 2,
  "height": 1,
  "isSeat": true,
  "seatOffset": { "x": 0, "y": -16 },
  "rotationGroups": [
    {
      "rotations": ["north", "east", "south", "west"],
      "sprite": "desk-modern.png"
    }
  ],
  "stateGroups": [
    {
      "state": "off",
      "sprite": "desk-modern-off.png"
    },
    {
      "state": "on",
      "sprite": "desk-modern-on.png"
    }
  ]
}
```

**Add to project:**

```bash
# 1. Create folder in assets/furniture/
mkdir webview-ui/public/assets/furniture/desk-modern

# 2. Add sprite PNG and manifest.json
# 3. Rebuild extension
npm run build
```

### Using External Asset Directories

```typescript
// 1. Create asset directory structure:
// /my-assets/
//   furniture/
//     custom-desk/
//       manifest.json
//       sprite.png

// 2. In Pixel Agents Settings → Add Asset Directory
// 3. Select /my-assets/
// 4. Assets appear in furniture picker
```

**External manifest.json:**

```json
{
  "id": "custom-plant",
  "name": "Custom Plant",
  "category": "decorations",
  "width": 1,
  "height": 1,
  "isSeat": false,
  "rotationGroups": [
    {
      "rotations": ["north"],
      "sprite": "plant.png"
    }
  ]
}
```

## Code Examples

### Monitoring Agent Activity (Extension Side)

```typescript
import * as vscode from 'vscode';
import * as fs from 'fs';
import * as path from 'path';

// Watch Claude Code JSONL transcript for agent activity
function watchAgentTranscript(projectPath: string, agentId: string): vscode.Disposable {
  const jsonlPath = path.join(
    projectPath,
    '.claude',
    'projects',
    agentId,
    'transcript.jsonl'
  );

  let lastPosition = 0;

  const interval = setInterval(() => {
    if (!fs.existsSync(jsonlPath)) {
      return;
    }

    const stats = fs.statSync(jsonlPath);
    if (stats.size <= lastPosition) {
      return;
    }

    const buffer = Buffer.alloc(stats.size - lastPosition);
    const fd = fs.openSync(jsonlPath, 'r');
    fs.readSync(fd, buffer, 0, buffer.length, lastPosition);
    fs.closeSync(fd);

    const lines = buffer.toString('utf-8').split('\n').filter(l => l.trim());
    
    lines.forEach(line => {
      try {
        const record = JSON.parse(line);
        handleAgentRecord(agentId, record);
      } catch (err) {
        console.error('[Pixel Agents] Failed to parse JSONL:', err);
      }
    });

    lastPosition = stats.size;
  }, 500);

  return new vscode.Disposable(() => clearInterval(interval));
}

function handleAgentRecord(agentId: string, record: any): void {
  // Detect agent activity from JSONL record type
  if (record.type === 'tool_use') {
    const toolName = record.content?.name;
    
    if (toolName === 'write_file' || toolName === 'edit_file') {
      updateAgentState(agentId, 'typing');
    } else if (toolName === 'search_files' || toolName === 'read_file') {
      updateAgentState(agentId, 'reading');
    } else if (toolName === 'run_command') {
      updateAgentState(agentId, 'typing');
    }
  } else if (record.type === 'user_message') {
    updateAgentState(agentId, 'idle');
  }
}

function updateAgentState(agentId: string, state: string): void {
  // Send message to webview
  webviewPanel.webview.postMessage({
    command: 'updateAgentState',
    agentId,
    state
  });
}
```

### Character Animation (Webview Side)

```typescript
interface Character {
  id: string;
  x: number;
  y: number;
  targetX: number;
  targetY: number;
  state: 'idle' | 'walking' | 'typing' | 'reading';
  direction: 'north' | 'south' | 'east' | 'west';
  frame: number;
  spriteSheet: HTMLImageElement;
}

class CharacterRenderer {
  private characters: Map<string, Character> = new Map();
  private readonly TILE_SIZE = 16;
  private readonly FRAME_RATE = 8; // frames per second
  private frameCounter = 0;

  update(deltaTime: number): void {
    this.frameCounter += deltaTime;
    const frameInterval = 1000 / this.FRAME_RATE;

    this.characters.forEach(char => {
      // Update position if walking
      if (char.state === 'walking') {
        const dx = char.targetX - char.x;
        const dy = char.targetY - char.y;
        const distance = Math.sqrt(dx * dx + dy * dy);

        if (distance < 1) {
          char.x = char.targetX;
          char.y = char.targetY;
          char.state = 'idle';
        } else {
          const speed = 2; // pixels per frame
          char.x += (dx / distance) * speed;
          char.y += (dy / distance) * speed;

          // Update direction based on movement
          if (Math.abs(dx) > Math.abs(dy)) {
            char.direction = dx > 0 ? 'east' : 'west';
          } else {
            char.direction = dy > 0 ? 'south' : 'north';
          }
        }
      }

      // Advance animation frame
      if (this.frameCounter >= frameInterval) {
        char.frame = (char.frame + 1) % this.getFrameCount(char.state);
      }
    });

    if (this.frameCounter >= frameInterval) {
      this.frameCounter = 0;
    }
  }

  render(ctx: CanvasRenderingContext2D): void {
    this.characters.forEach(char => {
      const spriteX = this.getSpriteX(char);
      const spriteY = this.getSpriteY(char);

      ctx.drawImage(
        char.spriteSheet,
        spriteX, spriteY,
        this.TILE_SIZE, this.TILE_SIZE * 2, // source
        Math.floor(char.x), Math.floor(char.y),
        this.TILE_SIZE, this.TILE_SIZE * 2  // destination
      );
    });
  }

  private getSpriteX(char: Character): number {
    return char.frame * this.TILE_SIZE;
  }

  private getSpriteY(char: Character): number {
    const directionOffsets = {
      'south': 0,
      'west': 1,
      'east': 2,
      'north': 3
    };
    
    const stateOffsets = {
      'idle': 0,
      'walking': 0,
      'typing': 4,
      'reading': 8
    };

    return (stateOffsets[char.state] + directionOffsets[char.direction]) 
           * this.TILE_SIZE * 2;
  }

  private getFrameCount(state: string): number {
    if (state === 'walking') return 4;
    if (state === 'typing' || state === 'reading') return 3;
    return 1; // idle
  }

  moveCharacter(id: string, targetX: number, targetY: number): void {
    const char = this.characters.get(id);
    if (char) {
      char.targetX = targetX;
      char.targetY = targetY;
      char.state = 'walking';
    }
  }

  setCharacterState(id: string, state: Character['state']): void {
    const char = this.characters.get(id);
    if (char) {
      char.state = state;
      char.frame = 0;
    }
  }
}
```

### Pathfinding (BFS for Agent Movement)

```typescript
interface Point {
  x: number;
  y: number;
}

class Pathfinder {
  private grid: boolean[][]; // true = walkable
  private width: number;
  private height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
    this.grid = Array(height).fill(null).map(() => Array(width).fill(true));
  }

  setObstacle(x: number, y: number, blocked: boolean): void {
    if (x >= 0 && x < this.width && y >= 0 && y < this.height) {
      this.grid[y][x] = !blocked;
    }
  }

  findPath(start: Point, end: Point): Point[] | null {
    if (!this.isWalkable(end.x, end.y)) return null;

    const queue: { point: Point; path: Point[] }[] = [
      { point: start, path: [start] }
    ];
    const visited = new Set<string>();
    visited.add(`${start.x},${start.y}`);

    const directions = [
      { x: 0, y: -1 }, // north
      { x: 1, y: 0 },  // east
      { x: 0, y: 1 },  // south
      { x: -1, y: 0 }  // west
    ];

    while (queue.length > 0) {
      const { point, path } = queue.shift()!;

      if (point.x === end.x && point.y === end.y) {
        return path;
      }

      for (const dir of directions) {
        const next = { x: point.x + dir.x, y: point.y + dir.y };
        const key = `${next.x},${next.y}`;

        if (
          this.isWalkable(next.x, next.y) &&
          !visited.has(key)
        ) {
          visited.add(key);
          queue.push({
            point: next,
            path: [...path, next]
          });
        }
      }
    }

    return null; // No path found
  }

  private isWalkable(x: number, y: number): boolean {
    return (
      x >= 0 &&
      x < this.width &&
      y >= 0 &&
      y < this.height &&
      this.grid[y][x]
    );
  }
}

// Usage
const pathfinder = new Pathfinder(32, 32);

// Mark furniture as obstacles
pathfinder.setObstacle(5, 5, true); // desk
pathfinder.setObstacle(6, 5, true); // desk continuation

// Find path from spawn to desk
const path = pathfinder.findPath({ x: 1, y: 1 }, { x: 5, y: 6 });

if (path) {
  // Move character along path
  path.forEach((point, index) => {
    setTimeout(() => {
      renderer.moveCharacter('agent-1', point.x * 16, point.y * 16);
    }, index * 200);
  });
}
```

## Troubleshooting

### Agent Not Appearing

**Enable Debug View:**

```typescript
// Settings → Debug View (toggle on)
// Check per-agent diagnostics:
// - "JSONL not found" = extension can't locate session file
// - "Lines parsed: 0" = no activity detected
// - Verify file path matches expected location
```

**Check Debug Console:**

```typescript
// If running from source (F5 Development Host):
// View → Debug Console
// Search for "[Pixel Agents]"
// Look for:
// - Project directory resolution errors
// - JSONL polling status
// - Path encoding mismatches
```

### Agent Stuck on Idle

**Common causes:**

1. **JSONL file not found**: Claude Code session hasn't created transcript yet
   ```bash
   # Check if file exists
   ls ~/.claude/projects/*/transcript.jsonl
   ```

2. **Polling lag**: Extension polls every 500ms, brief delays are normal

3. **Unrecognized record type**: Claude Code may emit new JSONL types
   ```typescript
   // Check Debug Console for "Unrecognized JSONL record type: X"
   ```

### Agent-Terminal Desync

**Symptoms:**
- Character appears but terminal is closed
- Terminal exists but no character
- Multiple characters for one terminal

**Fixes:**

```typescript
// 1. Close all Claude Code terminals
// 2. In Pixel Agents panel, click each character
// 3. Press Delete or click trash icon
// 4. Spawn fresh agent with "+ Agent"
```

### Speech Bubble Not Clearing

**Heuristic-based detection limitations:**

The extension uses idle timers to detect when agents wait for input. If speech bubble persists:

```typescript
// 1. Type something in the Claude Code terminal
// 2. Wait 2-3 seconds for polling to update
// 3. If stuck, close and reopen Pixel Agents panel
```

### Linux/macOS Home Directory Confusion

If you launch VS Code without opening a folder:

```bash
# Agent starts in home directory
code

# Sessions tracked under:
~/.claude/projects/<home-directory-hash>/
```

**Solution:**

```bash
# Always open VS Code with a folder
code /path/to/project
```

## Common Patterns

### Multi-Agent Workflow

```typescript
// 1. Spawn multiple agents for parallel work
// Right-click "+ Agent" → create 3 agents

// 2. Assign each to different task
// Agent 1: "Refactor authentication module"
// Agent 2: "Write unit tests for API"
// Agent 3: "Update documentation"

// 3. Monitor all agents visually in office
// Watch characters move between typing/reading states

// 4. Intervene when speech bubble appears
// Click terminal to provide input or approval
```

### Custom Office for Team Visualization

```typescript
// 1. Design office layout per team structure
// Layout → Create 4 desk clusters for 4 teams

// 2. Export layout for sharing
// Settings → Export Layout → save as team-office.json

// 3. Team members import same layout
// Settings → Import Layout → select team-office.json

// 4. Everyone sees agents in same office structure
```

### Sub-Agent Visualization

When Claude Code uses the `task` tool to spawn sub-agents:

```typescript
// Parent agent spawns sub-agent
// Example: Main agent delegates "write tests" to sub-agent

// Visual behavior:
// - New character appears linked to parent
// - Sub-agent character animates independently
// - Both visible until sub-task completes
// - Sub-agent disappears when task done
```

### External Asset Workflow

```typescript
// 1. Create asset pack structure
mkdir -p /my-assets/furniture/custom-furniture

// 2. Add manifest and sprites
cat > /my-assets/furniture/custom-furniture/manifest.json << 'EOF'
{
  "id": "gaming-chair",
  "name": "Gaming Chair",
  "category": "seats",
  "width": 1,
  "height": 1,
  "isSeat": true,
  "seatOffset": { "x": 0, "y": -8 },
  "rotationGroups": [
    {
      "rotations": ["north", "south", "east", "west"],
      "sprite": "chair.png"
    }
  ]
}
EOF

// 3. Link in Pixel Agents
// Settings → Add Asset Directory → /my-assets

// 4. Use in layout editor
// Layout → Place → Select "Gaming Chair"
```

## Extension Commands

Pixel Agents registers the following VS Code commands:

```typescript
// Open Pixel Agents panel
"pixelAgents.openPanel"

// Spawn new agent (normal)
"pixelAgents.spawnAgent"

// Spawn agent with skip permissions
"pixelAgents.spawnAgentSkipPermissions"

// Open layout editor
"pixelAgents.openLayoutEditor"

// Open settings modal
"pixelAgents.openSettings"

// Toggle debug view
"pixelAgents.toggleDebug"
```

Access via Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`):
```
> Pixel Agents: Open Panel
> Pixel Agents: Spawn Agent
```

## Performance Considerations

**JSONL polling:**
- Polls every 500ms per agent
- Minimal CPU impact for <10 agents
- For 10+ agents, consider increasing interval in source

**Canvas rendering:**
- 60 FPS target, degrades gracefully
- Tested up to 64×64 grid with 20 characters
- Use integer zoom levels for pixel-perfect rendering

**Memory usage:**
- ~5-10 MB per agent (JSONL tracking + character state)
- Asset cache: ~20-50 MB depending on furniture count

## Future Roadmap

Pixel Agents is evolving toward a fully agent-agnostic, platform-agnostic interface:

- **Agent-agnostic adapters**: Support for Codex, Cursor, Copilot, Gemini
- **Platform flexibility**: Electron app, web app, beyond VS Code
- **Advanced features**:
  - Desks as directories (drag agent to desk = assign to project)
  - Kanban board for autonomous task picking
  - Deep agent inspection (model, context, history, prompts)
  - Token health bars (rate limits, context windows)
  - 3D/VR office environments

See [GitHub Discussions](https://github.com/pablodelucca/pixel-agents/discussions) for ongoing architecture work.
