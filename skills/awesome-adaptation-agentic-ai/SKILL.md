---
name: awesome-adaptation-agentic-ai
description: Curated research collection on adaptation strategies for agentic AI systems, covering agent and tool adaptation methods with RL, SFT, and DPO approaches
triggers:
  - find papers on agentic AI adaptation
  - how do AI agents adapt their tool use
  - research on tool execution adaptation
  - agent reinforcement learning methods
  - lookup agentic AI adaptation techniques
  - show me agent adaptation frameworks
  - what are tool adaptation strategies
  - find research on adaptive AI agents
---

# Awesome Adaptation of Agentic AI

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

This repository is a curated research collection accompanying the paper "Adaptation of Agentic AI" (arXiv:2512.16301). It systematically organizes papers on how AI agents adapt their behavior, particularly focusing on:

- **Agent Adaptation**: Methods to improve agent decision-making through tool execution signals (A1) or output signals (A2)
- **Tool Adaptation**: Approaches to optimize tools either independently (T1) or with agent supervision (T2)

The collection categorizes 40+ research papers by adaptation strategy, training method (RL, SFT, DPO), task domain, and implementation details.

## Repository Structure

```
awesome-adaptation-of-agentic-ai/
├── README.md                 # Main paper collection with categorized tables
├── images/
│   ├── intro.png            # Overview diagram
│   ├── a1_illustrate.png    # Tool execution signaled adaptation
│   ├── a1_timeline.png      # Development timeline
│   ├── paper_icon.png       # Paper link icons
│   └── code_icon.png        # Code link icons
└── LICENSE
```

## Installation & Usage

### Cloning the Repository

```bash
# Clone the repository
git clone https://github.com/pat-jj/Awesome-Adaptation-of-Agentic-AI.git
cd Awesome-Adaptation-of-Agentic-AI

# View the main README
cat README.md
```

### Browsing Papers

The repository organizes papers into four main categories:

1. **A1: Tool Execution Signaled Agent Adaptation**
   - RL-based methods (GRPO, PPO, AlphaZero-like)
   - SFT & DPO methods

2. **A2: Agent Output Signaled Adaptation**
   - Coming in future updates

3. **T1: Agent-Agnostic Tool Adaptation**
   - Coming in future updates

4. **T2: Agent-Supervised Tool Adaptation**
   - Coming in future updates

## Key Research Categories

### A1: Tool Execution Signaled (RL-Based)

Papers where agents learn from tool execution feedback using reinforcement learning:

**Notable Methods:**
- **Orion** (2025.11): IR agents with GRPO on LFM2
- **DeepSeek-R1-Zero** (2025.01): Coding agents with code executor feedback
- **DeepSeek-Prover-V2** (2025.04): Formal theorem proving with Lean compiler
- **FTRL** (2025.08): Multi-step tool-use with GRPO

**Common Pattern:**
```
Agent → Tool Call → Execution Feedback → RL Update (GRPO/PPO)
```

### A1: Tool Execution Signaled (SFT & DPO)

Papers using supervised fine-tuning and direct preference optimization:

**Notable Methods:**
- **ToolLLM** (2023.07): API planning with real-world APIs
- **RetPO** (2024.02): Information retrieval with DPO
- **AWL** (2024.12): Scientific reasoning with adaptive learning

**Common Pattern:**
```
Agent → Tool Call → Execution Trace → Supervised Learning
```

## Common Use Cases

### Finding Papers by Task

**Example: Formal Theorem Proving**

```bash
# Search README for theorem proving papers
grep -i "theorem proving" README.md
```

Papers include: AlphaProof, BFS-Prover-V2, Goedel-Prover-V2, Leanabell-Prover-V2, DeepSeek-Prover-V1.5/V2

**Example: Coding & Code Execution**

```bash
# Find coding-related papers
grep -i "coding\|code executor" README.md
```

Papers include: olmOCR2, R1-Code-Interpreter, Code-R1, DeepSeek-R1-Zero, RLEF, LeDex, CYCLE, CodeAct

### Finding Papers by Method

**Example: GRPO (Group Relative Policy Optimization)**

```bash
# List all GRPO papers
grep "GRPO" README.md
```

Commonly used in: Tool-N1, DeepSeek-Prover-V2, SQL-R1, Rec-R1, DeepRetrieval, etc.

**Example: DPO (Direct Preference Optimization)**

```bash
# List all DPO papers
grep "DPO" README.md
```

Used in: AWL, LeReT, TP-LLaMA, RetPO

### Finding Papers by Model Backbone

**Example: Qwen2.5-based agents**

```bash
# Find Qwen2.5 implementations
grep "Qwen2.5" README.md
```

Models include: olmOCR2, ToolExpander, BFS-Prover-V2, WebGen-Agent, Tool-R1, etc.

## Accessing Paper Resources

### Paper Links

All papers include arXiv or conference links:

```markdown
[Paper](https://arxiv.org/abs/2512.16301)
```

### Code Repositories

Many papers provide implementation code:

```markdown
[Code](https://github.com/example/repo)
```

### Reading Strategy

```python
# Pseudo-code for systematic review
def research_adaptation_strategy(task_domain, method_type):
    """
    Navigate to specific adaptation category
    
    Args:
        task_domain: e.g., "coding", "theorem proving", "IR"
        method_type: "RL", "SFT", "DPO"
    
    Returns:
        List of relevant papers with links
    """
    # 1. Go to appropriate section (A1, A2, T1, T2)
    # 2. Filter by method_type (RL-based vs SFT/DPO)
    # 3. Search table for task_domain
    # 4. Check paper links, code availability, model backbones
    pass
```

## Citation Format

When using this repository for research:

```bibtex
@article{jiang2025adaptation,
  title={Adaptation of Agentic AI},
  author={Jiang, Pengcheng and Lin, Jiacheng and Shi, Zhiyi and Wang, Zifeng and He, Luxi and Wu, Yichen and Zhong, Ming and Song, Peiyang and Zhang, Qizheng and Wang, Heng and others},
  journal={arXiv preprint arXiv:2512.16301},
  year={2025}
}
```

## Contributing

The repository welcomes pull requests for:

- New papers on agentic AI adaptation
- Updates to existing paper information
- Corrections to categorizations
- Additional metadata (benchmarks, datasets)

**Contribution Pattern:**

```bash
# 1. Fork the repository
# 2. Add paper to appropriate category in README.md
# 3. Follow existing table format:
# | Time | Method | Venue | Task(s) | Tool(s) | Agent Backbone | Tuning |

# Example entry:
# | 2025.XX | YourMethod | Venue<br>🔗 [Paper](link)<br>💻 [Code](link) | Task | Tools | Model | Method |

# 4. Submit pull request
```

## Key Insights from the Collection

### Adaptation Taxonomy

1. **Signal Type**:
   - Tool execution feedback (A1)
   - Agent output quality (A2)

2. **Training Methods**:
   - **GRPO**: Group Relative Policy Optimization (most common in 2025)
   - **PPO**: Proximal Policy Optimization
   - **AlphaZero-like**: Self-play with value/policy networks
   - **SFT**: Supervised fine-tuning on execution traces
   - **DPO**: Direct preference optimization

3. **Trend**: Growing use of GRPO for tool-augmented agents (2025), especially with Qwen2.5 and DeepSeek models

### Domain Coverage

- **Coding**: Code execution sandboxes (DeepSeek-R1-Zero, R1-Code-Interpreter)
- **Formal Math**: Lean compilers (AlphaProof, DeepSeek-Prover)
- **Information Retrieval**: Search engines, retrievers (DeepRetrieval, ReZero)
- **Tool-Calling**: API execution (ToolLLM, Tool-N1, ToolExpander)
- **Web Agents**: GUI interaction (WebGen-Agent)

## Troubleshooting

### Finding Specific Research

**Q: How do I find papers using a specific model like LLaMA3?**

```bash
grep -i "llama3" README.md
```

**Q: Which papers have open-source code?**

```bash
# Look for code icon in tables
grep "code_icon.png" README.md
```

**Q: What are the most recent papers?**

```bash
# Check Time column (sorted by date within categories)
# Most recent: 2025.11 (Orion), 2025.10 (olmOCR2, AlphaProof)
```

### Understanding Abbreviations

- **IR**: Information Retrieval
- **GRPO**: Group Relative Policy Optimization
- **PPO**: Proximal Policy Optimization
- **SFT**: Supervised Fine-Tuning
- **DPO**: Direct Preference Optimization
- **TTRL**: Test-Time Reinforcement Learning
- **EI**: Expert Iteration

### Repository Maintenance

The repository is actively maintained with:
- **Last Update**: 2026-05-15 (per metadata)
- **Stars**: 650+ (growing at ~3 stars/day)
- **Open Issues**: 2

For questions or issues, check: https://github.com/pat-jj/Awesome-Adaptation-of-Agentic-AI/issues

## Related Resources

- **Homepage**: https://arxiv.org/abs/2512.16301
- **License**: CC BY-NC-ND 4.0 (content), NOASSERTION (code)
- **Topics**: adaptation, agentic-ai, large-language-models
