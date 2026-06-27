---
name: qwen-agentworld-language-world-model
description: Train and deploy Qwen-AgentWorld, a native language world model that simulates agentic environments across 7 domains (MCP, Search, Terminal, SWE, Android, Web, OS) for agent training and evaluation.
triggers:
  - "set up qwen agentworld for environment simulation"
  - "how do I use qwen-agentworld as a world model"
  - "deploy qwen-agentworld for agent training"
  - "evaluate my agent on agentworld bench"
  - "simulate terminal environment with qwen agentworld"
  - "run sim rl with qwen agentworld"
  - "use qwen agentworld for controllable simulation"
  - "benchmark language world model performance"
---

# Qwen-AgentWorld Language World Model

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What It Does

Qwen-AgentWorld is a **native language world model** that simulates agentic environments across seven unified domains: MCP (tool-calling), Search, Terminal, SWE (software engineering), Android, Web, and OS. Unlike traditional approaches that adapt language models post-hoc, Qwen-AgentWorld is trained from Continued Pre-Training (CPT) onward with environment modeling as the core objective.

**Key Capabilities:**
- **Unified multi-domain simulation**: Single model covers 7 agent interaction environments
- **Controllable simulation**: Inject perturbations, create fictional worlds, adapt environments
- **Agent foundation model**: Sim RL warm-up transfers to multi-turn, tool-calling agentic tasks
- **Zero-shot generalization**: Out-of-distribution environment handling (e.g., Claw Agent)
- **Evaluation benchmark**: AgentWorldBench with 5-dimensional rubric scoring

**Model Variants:**
- `Qwen-AgentWorld-35B-A3B` (35B total, 3B active MoE, 256K context)
- `Qwen-AgentWorld-397B-A17B` (397B total, 17B active MoE, 256K context)

## Installation

### Requirements

```bash
# Core dependencies
pip install torch transformers accelerate
pip install openai  # For API-based inference and evaluation

# For deployment (choose one)
pip install "sglang[all]"  # SGLang (recommended)
# OR
pip install vllm  # vLLM

# For evaluation
pip install huggingface_hub
```

### Download Model & Benchmark

```bash
# Download model weights
huggingface-cli download Qwen/Qwen-AgentWorld-35B-A3B --local-dir ./models/Qwen-AgentWorld-35B-A3B

# Download evaluation benchmark
huggingface-cli download Qwen/AgentWorldBench --repo-type dataset --local-dir ./AgentWorldBench

# Alternative: Use ModelScope in China
export SGLANG_USE_MODELSCOPE=true
export VLLM_USE_MODELSCOPE=true
```

## Deployment

### SGLang (Recommended)

```bash
python -m sglang.launch_server \
    --model-path Qwen/Qwen-AgentWorld-35B-A3B \
    --port 8000 \
    --tensor-parallel-size 4 \
    --context-length 262144 \
    --reasoning-parser qwen3
```

OpenAI-compatible API available at `http://localhost:8000/v1`

### vLLM

```bash
vllm serve Qwen/Qwen-AgentWorld-35B-A3B \
    --port 8000 \
    --tensor-parallel-size 4 \
    --max-model-len 262144 \
    --reasoning-parser qwen3 \
    --language-model-only \
    --trust-remote-code
```

**Note:** `--language-model-only` is required because the model architecture includes visual component definitions but only language weights are included.

## Core Usage Patterns

### 1. Basic Inference with Transformers

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "Qwen/Qwen-AgentWorld-35B-A3B"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype="auto",
    device_map="auto",
)

# Terminal domain example
messages = [
    {
        "role": "system",
        "content": "You are a language world model simulating a Linux terminal environment. "
                   "Given the user's command, predict the terminal output."
    },
    {
        "role": "user",
        "content": "Action: execute_bash\nCommand: ls -la /home/user/project/"
    }
]

text = tokenizer.apply_chat_template(
    messages, 
    tokenize=False, 
    add_generation_prompt=True
)
inputs = tokenizer([text], return_tensors="pt").to(model.device)
outputs = model.generate(**inputs, max_new_tokens=2048, temperature=0.6)
response = tokenizer.decode(
    outputs[0][inputs.input_ids.shape[-1]:], 
    skip_special_tokens=True
)
print(response)
```

### 2. API-Based Inference

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="EMPTY"
)

# MCP (tool-calling) domain example
response = client.chat.completions.create(
    model="Qwen/Qwen-AgentWorld-35B-A3B",
    messages=[
        {
            "role": "system",
            "content": "You are a Tool World Model simulating MCP server responses. "
                       "Given tool calls, predict realistic environment observations."
        },
        {
            "role": "user",
            "content": """### Turn 1
**Action:**
```json
{
  "tool_name": "read_file",
  "arguments": {"path": "/data/config.json"}
}
```"""
        }
    ],
    temperature=0.6,
    max_tokens=2048
)

print(response.choices[0].message.content)
```

### 3. Domain-Specific System Prompts

The repository includes templates in `prompts/{domain}/system_prompt.txt`. Each benchmark sample carries its own system prompt in the `system_str` field.

```python
# Load domain-specific system prompt template
with open("prompts/terminal/system_prompt.txt") as f:
    terminal_system_prompt = f.read()

messages = [
    {"role": "system", "content": terminal_system_prompt},
    {"role": "user", "content": "Action: execute_bash\nCommand: cat /etc/os-release"}
]
```

### 4. Multi-Turn Trajectory Simulation

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

trajectory = []
system_prompt = "You are a language world model simulating a web browser environment."

for turn in range(5):
    # Agent action
    action = f"Action: click_element\nSelector: button#{turn}\n"
    
    # Get environment observation from world model
    messages = [{"role": "system", "content": system_prompt}] + trajectory + [
        {"role": "user", "content": action}
    ]
    
    response = client.chat.completions.create(
        model="Qwen/Qwen-AgentWorld-35B-A3B",
        messages=messages,
        temperature=0.6,
        max_tokens=2048
    )
    
    observation = response.choices[0].message.content
    
    trajectory.append({"role": "user", "content": action})
    trajectory.append({"role": "assistant", "content": observation})
    
    print(f"\n=== Turn {turn+1} ===")
    print(f"Action: {action}")
    print(f"Observation: {observation}")
```

## Evaluation on AgentWorldBench

### Data Format

Each domain has a JSONL file (`mcp_test.jsonl`, `terminal_test.jsonl`, etc.):

```json
{
    "task": "terminal",
    "id": 145256090131919,
    "prompt": ["### Turn 1\n**Action:**\n...", "### Turn 2\n**Action:**\n..."],
    "response": ["**Environment Observation:**\n...", "**Environment Observation:**\n..."],
    "current_prompt": "### Turn 1\n**Action:**\nexecute_bash\nCommand: ls",
    "system_str": "You are a language world model simulating a Linux terminal...",
    "turn_idx": 1,
    "total_turns": 5
}
```

### Run Evaluation Pipeline

```bash
cd eval

# Step 1: Generate predictions from world model
python eval.py \
    --data_file ../AgentWorldBench/terminal_test.jsonl \
    --base_url http://localhost:8000/v1 \
    --model Qwen/Qwen-AgentWorld-35B-A3B \
    --output_file results/terminal_predictions.jsonl \
    --max_workers 16 \
    --temperature 0.6 \
    --max_tokens 8192

# Step 2: Score predictions with LLM judge
python eval.py \
    --data_file results/terminal_predictions.jsonl \
    --base_url http://localhost:8000/v1 \
    --model Qwen/Qwen-AgentWorld-35B-A3B \
    --output_file results/terminal_scores.jsonl \
    --max_workers 16 \
    --temperature 0.0 \
    --max_tokens 512 \
    --judge

# Step 3: Compute metrics
python eval.py \
    --data_file results/terminal_scores.jsonl \
    --aggregate
```

### Evaluation Dimensions

AgentWorldBench scores predictions on 5 dimensions:

1. **Format**: Structural correctness (JSON schema, markdown formatting)
2. **Factuality**: Technical accuracy of simulated behavior
3. **Consistency**: Coherence with previous trajectory turns
4. **Realism**: Plausibility vs. real environment responses
5. **Quality**: Overall utility for agent training

Scores are 0-100 per dimension, aggregated as mean.

## Advanced: Controllable Simulation

### Environment Adaptation (MCP Domain)

Inject perturbations to expose agent weaknesses:

```python
control_instruction = """
Simulate an environment where:
- 30% of file operations return "Permission denied"
- Network requests have 200ms random latency
- Database queries occasionally timeout
"""

messages = [
    {
        "role": "system",
        "content": f"{mcp_system_prompt}\n\n{control_instruction}"
    },
    {
        "role": "user",
        "content": '{"tool_name": "write_file", "arguments": {"path": "/tmp/test.txt", "content": "data"}}'
    }
]
```

### Fictional-World Construction (Search Domain)

Train agents in fully invented, self-consistent worlds:

```python
fictional_world = """
Simulate a search engine for a fictional universe where:
- Physics constants differ (speed of light = 500,000 km/s)
- Historical events are alternative (Rome never fell)
- Technology evolved differently (steam-powered computers)

Maintain internal consistency across all search results.
"""

messages = [
    {
        "role": "system",
        "content": f"{search_system_prompt}\n\n{fictional_world}"
    },
    {
        "role": "user",
        "content": "Query: history of computing in the Roman Empire"
    }
]
```

## Sim RL for Agent Training

### Basic Sim RL Setup

```python
import json
from openai import OpenAI

world_model = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")
agent_model = OpenAI(base_url="http://localhost:8001/v1", api_key="EMPTY")

# Generate synthetic trajectories
for episode in range(1000):
    trajectory = []
    state = "Initial terminal state"
    
    for step in range(10):
        # Agent generates action
        action_response = agent_model.chat.completions.create(
            model="agent-model-name",
            messages=[
                {"role": "system", "content": "You are a Linux terminal agent."},
                {"role": "user", "content": f"Current state: {state}\nChoose an action."}
            ]
        )
        action = action_response.choices[0].message.content
        
        # World model simulates next state
        next_state_response = world_model.chat.completions.create(
            model="Qwen/Qwen-AgentWorld-35B-A3B",
            messages=[
                {"role": "system", "content": "You are a terminal world model."},
                {"role": "user", "content": f"Action: {action}"}
            ]
        )
        next_state = next_state_response.choices[0].message.content
        
        trajectory.append({"action": action, "state": next_state})
        state = next_state
    
    # Save trajectory for RL training
    with open(f"trajectories/episode_{episode}.json", "w") as f:
        json.dump(trajectory, f)
```

### Out-of-Distribution Scaling (Claw Agent Example)

```bash
# Generate 4k synthetic Claw environments
python scripts/generate_claw_environments.py \
    --world_model http://localhost:8000/v1 \
    --num_environments 4000 \
    --output_dir claw_sim_data/

# Train agent with PPO on synthetic data
python scripts/train_ppo.py \
    --data_dir claw_sim_data/ \
    --agent_model Qwen/Qwen3.5-35B-A3B \
    --epochs 3

# Evaluate on real Claw-Eval benchmark
python scripts/eval_claw.py \
    --model trained_agent/ \
    --benchmark claw_eval
```

## Configuration

### World Model Parameters

```python
# Temperature: Controls randomness in environment simulation
# - 0.0: Deterministic (for evaluation)
# - 0.6-0.8: Realistic variance (for training)
temperature = 0.6

# Max tokens: Depends on domain complexity
# - Terminal/MCP: 2048-4096
# - SWE/Web/OS: 8192-16384
# - Search: 4096
max_tokens = 8192

# Context length: Use full 262K for long trajectories
context_length = 262144
```

### Tensor Parallelism

```bash
# For 35B-A3B model:
# - 1x A100 80GB: --tensor-parallel-size 1
# - 2x A100 80GB: --tensor-parallel-size 2
# - 4x A100 40GB: --tensor-parallel-size 4

# For 397B-A17B model:
# - 8x A100 80GB: --tensor-parallel-size 8
```

## Common Patterns

### Pattern 1: Domain-Specific Evaluation Loop

```python
import json
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

def evaluate_domain(domain_file, model_name):
    results = []
    with open(domain_file) as f:
        for line in f:
            sample = json.loads(line)
            
            response = client.chat.completions.create(
                model=model_name,
                messages=[
                    {"role": "system", "content": sample["system_str"]},
                    {"role": "user", "content": sample["current_prompt"]}
                ],
                temperature=0.6,
                max_tokens=8192
            )
            
            results.append({
                "id": sample["id"],
                "prediction": response.choices[0].message.content,
                "ground_truth": sample["response"][sample["turn_idx"]-1]
            })
    
    return results

# Run on all 7 domains
domains = ["mcp", "search", "terminal", "swe", "android", "web", "os"]
for domain in domains:
    results = evaluate_domain(f"AgentWorldBench/{domain}_test.jsonl", "Qwen/Qwen-AgentWorld-35B-A3B")
    with open(f"results/{domain}_results.json", "w") as f:
        json.dump(results, f)
```

### Pattern 2: Batch Inference with Concurrent Requests

```python
from concurrent.futures import ThreadPoolExecutor
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

def predict_single(sample):
    response = client.chat.completions.create(
        model="Qwen/Qwen-AgentWorld-35B-A3B",
        messages=[
            {"role": "system", "content": sample["system_str"]},
            {"role": "user", "content": sample["current_prompt"]}
        ],
        temperature=0.6,
        max_tokens=8192
    )
    return response.choices[0].message.content

# Process 100 samples with 16 concurrent workers
with ThreadPoolExecutor(max_workers=16) as executor:
    predictions = list(executor.map(predict_single, samples))
```

### Pattern 3: Custom Rubric Scoring

```python
def score_prediction(prediction, ground_truth, judge_client):
    judge_prompt = f"""
Rate the predicted environment observation on 5 dimensions (0-100):

**Ground Truth:**
{ground_truth}

**Prediction:**
{prediction}

Provide scores in JSON format:
{{"format": 85, "factuality": 90, "consistency": 80, "realism": 88, "quality": 87}}
"""
    
    response = judge_client.chat.completions.create(
        model="Qwen/Qwen-AgentWorld-35B-A3B",
        messages=[
            {"role": "system", "content": "You are an expert judge evaluating world model predictions."},
            {"role": "user", "content": judge_prompt}
        ],
        temperature=0.0,
        max_tokens=512
    )
    
    return json.loads(response.choices[0].message.content)
```

## Troubleshooting

### Out-of-Memory Errors

```bash
# Reduce context length
--max-model-len 131072  # 128K instead of 256K

# Increase tensor parallelism
--tensor-parallel-size 8

# Enable quantization (vLLM)
--quantization awq
```

### Slow Inference

```bash
# SGLang: Enable RadixAttention caching
--enable-radix-cache

# vLLM: Adjust max concurrent requests
--max-num-seqs 128

# Use smaller batch sizes for long contexts
--max-num-batched-tokens 32768
```

### Model Loading Fails

```bash
# Ensure --language-model-only flag (vLLM)
# The model config includes vision components but weights are language-only

# For HuggingFace Hub access issues in China
export HF_ENDPOINT=https://hf-mirror.com
# OR use ModelScope
export VLLM_USE_MODELSCOPE=true
```

### Evaluation Script Hangs

```python
# Add timeout to OpenAI client
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="EMPTY",
    timeout=60.0  # 60 second timeout
)
```

### JSON Parsing Errors in Judge Scores

```python
import re
import json

def extract_json_from_response(text):
    # Try direct parse
    try:
        return json.loads(text)
    except:
        pass
    
    # Extract JSON from markdown code blocks
    match = re.search(r'```json\s*(\{.*?\})\s*```', text, re.DOTALL)
    if match:
        return json.loads(match.group(1))
    
    # Extract first JSON object
    match = re.search(r'\{[^}]*"format"[^}]*\}', text)
    if match:
        return json.loads(match.group(0))
    
    raise ValueError(f"No valid JSON found in: {text}")
```

## Environment Variables

```bash
# ModelScope (China alternative to HuggingFace)
export SGLANG_USE_MODELSCOPE=true
export VLLM_USE_MODELSCOPE=true

# HuggingFace mirror
export HF_ENDPOINT=https://hf-mirror.com

# CUDA optimization
export CUDA_VISIBLE_DEVICES=0,1,2,3

# Logging
export SGLANG_LOG_LEVEL=INFO
export VLLM_LOGGING_LEVEL=INFO
```

## Performance Benchmarks

**AgentWorldBench Overall Scores (5-dimension mean):**
- Qwen-AgentWorld-397B-A17B: **58.71** (SOTA)
- GPT-5.4: 58.25
- Claude Opus 4.8: 56.59
- Qwen-AgentWorld-35B-A3B: **56.39** (+8.66 vs. base Qwen3.5-35B-A3B)

**Sim RL Transfer (35B model on out-of-domain tasks):**
- Terminal-Bench 2.0: 39.55 (+6.30)
- SWE-Bench Verified: 67.86 (+3.39)
- Claw-Eval: 64.88 (+11.28)
- BFCL v4: 71.25 (+8.96)

## Resources

- **Technical Report:** [arXiv:2606.24597](http://arxiv.org/abs/2606.24597)
- **Blog Post:** [Qwen Blog](https://qwen.ai/blog?id=qwen-agentworld)
- **HuggingFace Collection:** [Qwen/qwen-agentworld](https://huggingface.co/collections/Qwen/qwen-agentworld)
- **ModelScope (China):** [Qwen-AgentWorld](https://modelscope.cn/collections/Qwen/Qwen-AgentWorld)
- **GitHub Issues:** [Report bugs](https://github.com/QwenLM/Qwen-AgentWorld/issues)
- **Discussions:** [Community forum](https://github.com/QwenLM/Qwen-AgentWorld/discussions)
