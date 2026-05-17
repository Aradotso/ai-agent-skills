---
name: world2agent-protocol
description: Install and use World2Agent (W2A) sensors to give AI agents structured, real-time perception of the real world
triggers:
  - add a world2agent sensor
  - install w2a sensor for news
  - set up world2agent protocol
  - configure world2agent sensor
  - build a custom w2a sensor
  - list available world2agent sensors
  - troubleshoot world2agent signal flow
  - integrate world2agent into my agent
---

# World2Agent Protocol Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What is World2Agent?

World2Agent (W2A) is an open protocol that standardizes how AI agents perceive the real world. Sensors watch data sources (news feeds, production alerts, market data, etc.) and emit structured signals following the W2A schema. Your agent receives these signals and decides what to do.

**Architecture:** World → Sensor → Agent

- **Sensors** are npm packages that watch external data sources
- **Signals** are structured JSON events following the W2A protocol
- **Agents** consume signals via runtime plugins or direct SDK integration

## Installation

### For Claude Code

```bash
/plugin marketplace add machinepulse-ai/world2agent-plugins
/plugin install world2agent@world2agent-plugins
/reload-plugins
```

Add a sensor:

```bash
/world2agent:sensor-add @world2agent/sensor-hackernews
```

Restart with plugin channel:

```bash
claude --dangerously-load-development-channels plugin:world2agent@world2agent-plugins
```

### For Hermes

```bash
npm install -g @world2agent/hermes-sensor-bridge
hermes skills install machinepulse-ai/world2agent-plugins/hermes-sensor-bridge/skills/world2agent-manage
```

In Hermes session:

```bash
/world2agent-manage add @world2agent/sensor-hackernews
```

### For OpenClaw

```bash
npm install -g @world2agent/openclaw-sensor-bridge
openclaw skills install world2agent-manage
```

Then in chat:

```
Use world2agent-manage skill install @quill-io/sensor-frontier-ai-news
```

### Direct SDK Integration (TypeScript/Node.js)

```bash
npm install @world2agent/core
npm install @world2agent/sensor-hackernews
```

## Core Concepts

### Signal Format

Every W2A signal follows this schema:

```typescript
{
  "signal_id": "uuid-v4",
  "sensor_id": "@world2agent/sensor-hackernews",
  "timestamp": "2026-05-17T10:30:00Z",
  "type": "news.hackernews.story",
  "priority": "medium",
  "data": {
    "title": "Show HN: World2Agent Protocol",
    "url": "https://news.ycombinator.com/item?id=123456",
    "score": 340,
    "author": "machinepulse"
  },
  "metadata": {
    "ttl": 3600,
    "schema_version": "1.0"
  }
}
```

### Key Fields

- `signal_id`: Unique identifier for deduplication
- `sensor_id`: npm package name of the sensor
- `type`: Hierarchical type (category.source.event)
- `priority`: `critical` | `high` | `medium` | `low`
- `data`: Sensor-specific payload
- `metadata.ttl`: Signal lifetime in seconds

## Using the SDK

### Basic Signal Consumption

```typescript
import { W2AClient } from '@world2agent/core';
import HackerNewsSensor from '@world2agent/sensor-hackernews';

const client = new W2AClient();

// Register sensor
await client.registerSensor(new HackerNewsSensor({
  minScore: 100,
  keywords: ['AI', 'agents', 'protocol']
}));

// Listen for signals
client.on('signal', (signal) => {
  console.log(`[${signal.priority}] ${signal.type}`);
  console.log(signal.data);
  
  // Your agent logic here
  if (signal.priority === 'high') {
    handleUrgentSignal(signal);
  }
});

// Start receiving signals
await client.start();
```

### Signal Filtering

```typescript
import { W2AClient, SignalFilter } from '@world2agent/core';

const client = new W2AClient();

// Filter by type pattern
const newsFilter = new SignalFilter({
  typePattern: /^news\./,
  minPriority: 'medium'
});

client.on('signal', newsFilter.apply((signal) => {
  // Only news signals with medium+ priority
  processNewsSignal(signal);
}));
```

### Multi-Sensor Setup

```typescript
import { W2AClient } from '@world2agent/core';
import HackerNewsSensor from '@world2agent/sensor-hackernews';
import FrontierAISensor from '@quill-io/sensor-frontier-ai-news';
import WeatherSensor from '@world2agent/sensor-weather';

const client = new W2AClient();

// Register multiple sensors
await client.registerSensor(new HackerNewsSensor({ minScore: 200 }));
await client.registerSensor(new FrontierAISensor({ labs: ['openai', 'anthropic'] }));
await client.registerSensor(new WeatherSensor({ 
  location: 'San Francisco',
  alerts: true 
}));

// Route by type
client.on('signal', (signal) => {
  if (signal.type.startsWith('news.')) {
    routeToNewsHandler(signal);
  } else if (signal.type.startsWith('weather.')) {
    routeToWeatherHandler(signal);
  }
});

await client.start();
```

## Building a Custom Sensor

### Minimal Sensor Structure

```typescript
// my-sensor/src/index.ts
import { BaseSensor, Signal } from '@world2agent/core';

export default class MySensor extends BaseSensor {
  constructor(config: { apiKey?: string } = {}) {
    super({
      id: '@myorg/sensor-example',
      name: 'Example Sensor',
      version: '1.0.0'
    });
    this.apiKey = config.apiKey || process.env.EXAMPLE_API_KEY;
  }

  async start() {
    // Poll every 60 seconds
    this.interval = setInterval(() => this.poll(), 60000);
    await this.poll();
  }

  async stop() {
    if (this.interval) clearInterval(this.interval);
  }

  private async poll() {
    const data = await this.fetchData();
    
    for (const item of data) {
      const signal: Signal = {
        signal_id: crypto.randomUUID(),
        sensor_id: this.config.id,
        timestamp: new Date().toISOString(),
        type: 'example.data.update',
        priority: item.urgent ? 'high' : 'medium',
        data: {
          title: item.title,
          value: item.value
        },
        metadata: {
          ttl: 3600,
          schema_version: '1.0'
        }
      };
      
      this.emit(signal);
    }
  }

  private async fetchData() {
    const response = await fetch('https://api.example.com/data', {
      headers: { 'Authorization': `Bearer ${this.apiKey}` }
    });
    return response.json();
  }
}
```

### Package.json for Sensors

```json
{
  "name": "@myorg/sensor-example",
  "version": "1.0.0",
  "description": "W2A sensor for Example API",
  "keywords": ["w2a-sensor", "world2agent", "example"],
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  },
  "dependencies": {
    "@world2agent/core": "^1.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

### SETUP.md for Interactive Configuration

```markdown
# Setup Questions

## API Key
**prompt:** What is your Example API key?
**env:** EXAMPLE_API_KEY
**required:** true

## Update Interval
**prompt:** Poll interval in seconds (default: 60)?
**env:** EXAMPLE_POLL_INTERVAL
**default:** 60
```

## Configuration Patterns

### Environment Variables

```bash
# .env
EXAMPLE_API_KEY=sk-xxxxxxxxxxxx
HACKERNEWS_MIN_SCORE=150
WEATHER_LOCATION=London
```

```typescript
import { W2AClient } from '@world2agent/core';
import { config } from 'dotenv';

config(); // Load .env

const client = new W2AClient();

await client.registerSensor(new HackerNewsSensor({
  minScore: parseInt(process.env.HACKERNEWS_MIN_SCORE || '100')
}));
```

### Runtime Configuration

```typescript
import { W2AClient } from '@world2agent/core';

const client = new W2AClient({
  maxSignalsPerSecond: 10,
  deduplication: true,
  persistSignals: './signals.log'
});

// Update sensor config at runtime
await client.updateSensorConfig('@world2agent/sensor-hackernews', {
  minScore: 250,
  keywords: ['AI', 'machine learning']
});
```

## Common Patterns

### Signal Deduplication

```typescript
import { W2AClient, Deduplicator } from '@world2agent/core';

const client = new W2AClient();
const dedup = new Deduplicator({ windowMs: 60000 }); // 1 minute window

client.on('signal', (signal) => {
  if (dedup.isNew(signal.signal_id)) {
    processSignal(signal);
  }
});
```

### Priority-Based Routing

```typescript
client.on('signal', async (signal) => {
  switch (signal.priority) {
    case 'critical':
      await notifyPagerDuty(signal);
      await handleImmediate(signal);
      break;
    case 'high':
      await queueForProcessing(signal);
      break;
    default:
      await logSignal(signal);
  }
});
```

### Webhook Integration

```typescript
import express from 'express';
import { W2AClient } from '@world2agent/core';

const app = express();
const client = new W2AClient();

app.post('/webhook/w2a', express.json(), (req, res) => {
  const signal = req.body;
  
  // Validate signature
  if (!client.validateSignature(signal, req.headers['x-w2a-signature'])) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process signal
  client.ingestSignal(signal);
  res.status(200).send('OK');
});

app.listen(3000);
```

### Persistent Signal Log

```typescript
import fs from 'fs/promises';
import { W2AClient } from '@world2agent/core';

const client = new W2AClient();
const logPath = './signals.jsonl';

client.on('signal', async (signal) => {
  // Append to newline-delimited JSON log
  await fs.appendFile(logPath, JSON.stringify(signal) + '\n');
  
  // Your processing logic
  await processSignal(signal);
});
```

## Finding Sensors

### Browse SensorHub

Visit [https://world2agent.ai/hub](https://world2agent.ai/hub) to browse the full catalog.

### CLI Search

```bash
npm search w2a-sensor
```

### Popular Sensors

- `@world2agent/sensor-hackernews` — Hacker News top stories
- `@quill-io/sensor-frontier-ai-news` — AI lab announcements (OpenAI, Anthropic, etc.)
- `@world2agent/sensor-weather` — Weather alerts and forecasts
- `@world2agent/sensor-github` — Repository events and releases
- `@world2agent/sensor-reddit` — Subreddit posts and trends

## Troubleshooting

### Signals Not Flowing

**Check sensor status:**

```typescript
const status = client.getSensorStatus('@world2agent/sensor-hackernews');
console.log(status); // { running: true, lastSignal: '2026-05-17T10:30:00Z' }
```

**Enable debug logging:**

```typescript
const client = new W2AClient({ logLevel: 'debug' });
```

**Verify sensor is started:**

```typescript
await client.start(); // Starts all registered sensors
```

### Duplicate Signals

Enable built-in deduplication:

```typescript
const client = new W2AClient({ deduplication: true });
```

Or implement custom logic:

```typescript
const seen = new Set<string>();

client.on('signal', (signal) => {
  if (seen.has(signal.signal_id)) return;
  seen.add(signal.signal_id);
  
  processSignal(signal);
});
```

### Rate Limiting

Throttle signal processing:

```typescript
import pThrottle from 'p-throttle';

const throttle = pThrottle({ limit: 5, interval: 1000 }); // 5/sec

const throttledProcess = throttle(async (signal) => {
  await processSignal(signal);
});

client.on('signal', throttledProcess);
```

### Sensor Crashes

Wrap sensor registration with error handling:

```typescript
try {
  await client.registerSensor(new MySensor({ apiKey: process.env.API_KEY }));
} catch (error) {
  console.error('Sensor registration failed:', error);
  // Fall back or retry
}

client.on('sensor-error', (sensorId, error) => {
  console.error(`Sensor ${sensorId} error:`, error);
  // Implement restart logic
});
```

### Missing Environment Variables

Validate before sensor initialization:

```typescript
const requiredEnv = ['EXAMPLE_API_KEY', 'HACKERNEWS_MIN_SCORE'];

for (const key of requiredEnv) {
  if (!process.env[key]) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
}
```

## Advanced: Graph Layer (Experimental)

Compose and enrich signals from multiple sensors before they reach your agent:

```typescript
import { W2AGraph } from '@world2agent/graph';

const graph = new W2AGraph();

// Merge HN and Reddit signals about the same topic
graph.addRule({
  inputs: ['news.hackernews.*', 'news.reddit.*'],
  merge: (signals) => ({
    type: 'news.aggregated.topic',
    data: {
      sources: signals.map(s => s.sensor_id),
      combined_score: signals.reduce((sum, s) => sum + s.data.score, 0)
    }
  }),
  window: 300 // 5 minutes
});

graph.on('enriched-signal', (signal) => {
  // Receives merged signals
});
```

See [RFC](https://github.com/machinepulse-ai/world2agent/docs/rfc-graph.md) for details.

## Resources

- [Documentation](https://github.com/machinepulse-ai/world2agent/docs)
- [SensorHub](https://world2agent.ai/hub)
- [Signal Format Spec](https://github.com/machinepulse-ai/world2agent/docs/signal-format.md)
- [Build a Sensor Guide](https://github.com/machinepulse-ai/world2agent/docs/build-a-sensor.md)
- [Discord Community](https://discord.gg/hDjaD8pX)

## License

Apache 2.0
