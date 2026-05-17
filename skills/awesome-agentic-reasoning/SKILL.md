---
name: awesome-agentic-reasoning
description: A curated collection of research papers and resources on agentic reasoning for Large Language Models, organized by planning, tool use, search, self-evolution, and multi-agent systems.
triggers:
  - "show me agentic reasoning papers"
  - "find research on LLM planning and reasoning"
  - "what are the latest papers on AI agents"
  - "I need resources on multi-agent systems"
  - "explain agentic reasoning frameworks"
  - "find papers on tool use for LLMs"
  - "research on self-evolving AI agents"
  - "show me embodied agent benchmarks"
---

# Awesome Agentic Reasoning

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill provides expertise in navigating and utilizing the **Awesome Agentic Reasoning** repository — a comprehensive, curated collection of research papers and resources on agentic reasoning for Large Language Models (LLMs). The repository is based on the survey paper "[Agentic Reasoning for Large Language Models: A Survey](https://arxiv.org/abs/2601.12538)" and organizes cutting-edge research into foundational reasoning, self-evolving systems, and multi-agent collaboration.

## What This Repository Provides

The Awesome Agentic Reasoning repository offers:

- **Categorized Research Papers**: Organized by thematic areas including planning, tool use, search, self-evolution, multi-agent systems, and real-world applications
- **Benchmarks**: Comprehensive lists of evaluation frameworks for agentic reasoning capabilities
- **Three-Layer Framework**: 
  - **Foundational Reasoning**: Core single-agent abilities (planning, tool-use, search)
  - **Self-Evolving Reasoning**: Adaptation through feedback, memory, and learning
  - **Collective Reasoning**: Multi-agent coordination and collaborative intelligence
- **Application Domains**: Math/coding agents, scientific discovery, embodied agents, healthcare, web exploration
- **Survey Materials**: Slides and the comprehensive survey paper

## Repository Structure

```
Awesome-Agentic-Reasoning/
├── README.md                          # Main curated list
├── CONTRIBUTING.md                    # Contribution guidelines
├── materials/                         # Survey slides and materials
│   └── Agentic Reasoning Survey Talk.pdf
└── figs/                             # Framework diagrams
    ├── overview.png
    └── planning.png
```

## Navigating the Repository

### Main Categories

The repository organizes papers into three primary layers:

#### 1. Foundational Agentic Reasoning

**Planning Reasoning**:
- In-context Planning (workflow design, tree search)
- Post-training Planning (supervised fine-tuning, reinforcement learning)

**Tool-Use Optimization**:
- In-context Tool-Use (API orchestration, workflow design)
- Post-training Tool-Use (supervised learning, RL fine-tuning)

**Agentic Search**:
- In-context Search (web navigation, knowledge retrieval)
- Post-training Search (RL optimization)

#### 2. Self-Evolving Agentic Reasoning

- **Agentic Feedback Mechanisms**: Self-reflection, critique, and iterative refinement
- **Agentic Memory**: Short-term and long-term memory systems
- **Evolving Foundational Capabilities**: Continuous improvement of planning, tool-use, and search

#### 3. Collective Multi-Agent Reasoning

- **Role Taxonomy**: Debate, collaboration, hierarchical structures
- **Collaboration Patterns**: Division of labor, coordination strategies
- **Multi-Agent Memory and Evolution**: Shared knowledge, collective learning

### Applications

The repository covers real-world applications:

- 💻 **Math Exploration & Coding Agents**
- 🔬 **Scientific Discovery Agents**
- 🤖 **Embodied Agents**
- 🏥 **Healthcare & Medicine Agents**
- 🌐 **Autonomous Web Exploration & Research Agents**

### Benchmarks

Organized by:
- **Core Mechanisms**: Tool Use, Search, Memory & Planning, Multi-Agent Systems
- **Application Domains**: Embodied, Scientific Discovery, Medical, Web, General Tool-Use

## Usage Patterns

### Finding Papers on Specific Topics

**Example 1: Finding Planning Papers**

Navigate to the Planning Reasoning section to find papers on:
- Workflow design approaches (ReAct, ReWOO, Plan-and-Solve)
- Tree search methods (Tree of Thoughts, MCTS-based approaches)
- Post-training planning optimization

**Example 2: Multi-Agent System Research**

The Collective Multi-Agent Reasoning section includes:
- Role specialization papers
- Collaboration frameworks
- Multi-agent memory systems

### Exploring Application Domains

**Example: Embodied Agent Research**

1. Check the **Applications > Embodied Agents** section
2. Cross-reference with **Benchmarks > Embodied Agents** for evaluation frameworks
3. Review foundational papers on planning and tool-use that apply to embodied settings

### Finding Benchmarks

**Example: Evaluating Tool-Use Capabilities**

```markdown
## Tool Use Benchmarks

Navigate to: Benchmarks > Core Mechanisms > Tool Use

Key benchmarks include:
- API-Bank: API selection and execution
- ToolBench: Multi-tool orchestration
- T-Eval: Tool learning evaluation
```

## Contributing to the Repository

### Adding New Papers

Create a pull request with papers organized by category:

```markdown
| [Paper Title](https://arxiv.org/abs/XXXX.XXXXX) | Conference/Year |
```

**Guidelines**:
- Place papers in the appropriate thematic section
- Follow the existing table format
- Include the full arXiv link or conference proceedings URL
- Add the publication year or venue

### Suggesting Resources

Open an issue to suggest:
- New paper categories
- Additional benchmarks
- Application domains not yet covered
- Survey materials or tutorials

**Contact**: 
- Email: twei10@illinois.edu, twli@illinois.edu, liu326@illinois.edu
- GitHub Issues: For suggestions and discussions

## Key Research Paradigms

### In-Context Reasoning vs. Post-Training Reasoning

The repository distinguishes between two optimization approaches:

**In-Context Reasoning**:
- Test-time scaling through structured orchestration
- Adaptive workflows without parameter updates
- Examples: ReAct, Tree of Thoughts, Chain-of-Thought prompting

**Post-Training Reasoning**:
- Behavior optimization via RL and supervised fine-tuning
- Parameter updates to internalize reasoning strategies
- Examples: RLHF for tool-use, Q-learning for planning

### Environmental Dynamics

Papers are organized by the environmental setting:

- **Static environments**: Fixed tool sets, deterministic outcomes
- **Dynamic environments**: Feedback loops, adaptation requirements
- **Multi-agent environments**: Coordination, communication, emergent behavior

## Working with Survey Materials

### Accessing the Survey Paper

The foundational survey is available at:
- arXiv: https://arxiv.org/abs/2601.12538
- HuggingFace Papers: https://huggingface.co/papers/2601.12538

### Using the Slides

Presentation materials are in `materials/Agentic Reasoning Survey Talk.pdf`:
- Framework overview
- Key insights from each reasoning layer
- Application case studies
- Future research directions

## Common Patterns

### Building a Research Bibliography

**Pattern: Comprehensive Literature Review**

```python
# Pseudo-code for extracting papers by category

categories = [
    "Planning Reasoning",
    "Tool-Use Optimization", 
    "Agentic Search",
    "Multi-Agent Systems"
]

papers_by_category = {}

for category in categories:
    # Navigate to README section
    papers = extract_papers_from_section(category)
    papers_by_category[category] = papers
    
# Generate BibTeX or reading list
```

### Tracking New Research

**Pattern: Monitoring Updates**

The repository is actively maintained. To stay current:

1. Watch the repository for updates
2. Check the News section in README for announcements
3. Review recent commits for newly added papers
4. Subscribe to GitHub notifications

### Cross-Referencing Applications and Benchmarks

**Pattern: Application-Specific Research**

For a specific application domain:

```markdown
1. Identify application section (e.g., "Healthcare & Medicine Agents")
2. Review papers in that section
3. Navigate to corresponding benchmark section
4. Check foundational techniques used (planning, tool-use, etc.)
5. Trace back to foundational reasoning sections for core methods
```

## Citation

When using this repository in research or projects:

```bibtex
@article{wei2026agentic,
  title={Agentic Reasoning for Large Language Models},
  author={Wei, Tianxin and Li, Ting-Wei and Liu, Zhining and Ning, Xuying and Yang, Ze and Zou, Jiaru and Zeng, Zhichen and Qiu, Ruizhong and Lin, Xiao and Fu, Dongqi and others},
  journal={arXiv preprint arXiv:2601.12538},
  year={2026}
}
```

## Integration with Development Workflows

### For Researchers

**Literature Review Workflow**:

1. Clone the repository for offline access
2. Use the categorized structure to identify relevant papers
3. Cross-reference applications with foundational techniques
4. Export citations for reference management tools

### For Practitioners

**Implementation Workflow**:

1. Identify your application domain (e.g., web agents, coding)
2. Review application-specific papers and benchmarks
3. Trace foundational techniques (planning, tool-use)
4. Reference implementation papers for code patterns
5. Evaluate using suggested benchmarks

### For Tool Builders

**Benchmark Selection**:

1. Determine core capability (planning, tool-use, search)
2. Navigate to corresponding benchmark section
3. Review evaluation frameworks and metrics
4. Compare agent performance across standard benchmarks

## Best Practices

### Exploring New Topics

1. **Start with the Overview**: Read the survey paper introduction and framework diagram
2. **Navigate by Layer**: Begin with foundational reasoning before advanced topics
3. **Cross-Reference**: Link application papers back to foundational techniques
4. **Check Benchmarks**: Understand evaluation standards for each capability

### Contributing Quality Additions

1. **Verify Relevance**: Ensure papers fit the agentic reasoning scope
2. **Check Duplicates**: Search existing entries before adding
3. **Provide Context**: Include venue/year information
4. **Follow Format**: Maintain consistent table structure

### Staying Current

1. **Monitor Commits**: The repository updates regularly with new papers
2. **Check News Section**: Major updates announced at the top of README
3. **Watch Discussions**: GitHub issues may highlight emerging trends
4. **Follow Survey Updates**: Authors plan continued improvements

## Troubleshooting

### Finding Specific Papers

**Issue**: Can't locate a specific paper

**Solution**:
- Use browser search (Ctrl+F / Cmd+F) on the README
- Check multiple related sections (papers may fit several categories)
- Review the benchmarks section for evaluation-focused papers
- Check recent commits if it's a new publication

### Understanding Categories

**Issue**: Unclear which section contains relevant papers

**Solution**:
- Refer to the framework overview diagram
- Read the category descriptions in the survey paper
- Cross-reference with similar known papers
- Check application sections if domain-specific

### Accessing Papers

**Issue**: Links not working or papers behind paywalls

**Solution**:
- Most papers link to arXiv versions (open access)
- For conference papers, search on Google Scholar
- Check author websites for preprints
- Use institutional access for published versions

## Related Resources

- **Survey Paper**: https://arxiv.org/abs/2601.12538
- **Presentation Slides**: `materials/Agentic Reasoning Survey Talk.pdf`
- **HuggingFace**: https://huggingface.co/papers/2601.12538
- **License**: MIT (open for use and contribution)

## Quick Reference

| Category | Key Papers | Benchmarks |
|----------|-----------|------------|
| Planning | Tree of Thoughts, ReAct, Plan-and-Solve | PlanBench, BlocksWorld |
| Tool-Use | Gorilla, ToolLLM, HuggingGPT | API-Bank, ToolBench |
| Search | WebGPT, Agent-E, Mind2Web | WebArena, GAIA |
| Multi-Agent | ChatDev, AgentVerse, MetaGPT | MAgIC, AgentBench |
| Embodied | LM-Nav, PERIA, RT-1 | CALVIN, MetaWorld |
| Scientific | FunSearch, AI Scientist | ScienceBench |

This skill enables AI coding agents to effectively navigate and utilize the Awesome Agentic Reasoning repository, helping developers access cutting-edge research on LLM-based agents, understand agentic reasoning frameworks, and apply state-of-the-art techniques to their projects.
