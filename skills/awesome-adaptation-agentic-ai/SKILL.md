---
name: awesome-adaptation-agentic-ai
description: Curated collection of research papers on adaptation strategies for agentic AI systems with tool execution and output signals
triggers:
  - show me papers on agentic AI adaptation
  - how do I find research on tool-augmented agents
  - what are the latest methods for agent adaptation with tools
  - browse agentic AI adaptation papers
  - find papers on reinforcement learning for AI agents
  - search for tool adaptation research papers
  - show agent adaptation methods and benchmarks
  - what papers cover LLM agent fine-tuning
---

# Awesome Adaptation of Agentic AI

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill provides expertise in navigating and utilizing the **Awesome Adaptation of Agentic AI** repository — a comprehensive, curated collection of research papers on adaptation strategies for AI agents. The repository categorizes papers into Agent Adaptation (A1: Tool Execution Signaled, A2: Agent Output Signaled) and Tool Adaptation (T1: Agent-Agnostic, T2: Agent-Supervised).

## What This Repository Provides

The repository accompanies the paper "Adaptation of Agentic AI" (arXiv:2512.16301) and organizes state-of-the-art research papers on:

- **Agent Adaptation**: Methods for adapting AI agents using feedback from tool execution or agent outputs
  - **A1 (Tool Execution Signaled)**: RL-based, SFT, and DPO methods that use tool execution feedback
  - **A2 (Agent Output Signaled)**: Methods that use agent output quality as signals
- **Tool Adaptation**: Strategies for adapting tools themselves
  - **T1 (Agent-Agnostic)**: Tool improvements independent of specific agents
  - **T2 (Agent-Supervised)**: Tool adaptations guided by agent behavior

Each paper entry includes:
- Publication venue and date
- Links to papers and code repositories
- Task domains (coding, formal theorem proving, tool-calling, IR, etc.)
- Tools used (code executors, compilers, APIs, retrievers)
- Agent backbones (LLaMA, Qwen, DeepSeek, etc.)
- Training methods (GRPO, PPO, SFT, DPO, AlphaZero)

## Installation & Access

```bash
# Clone the repository
git clone https://github.com/pat-jj/Awesome-Adaptation-of-Agentic-AI.git
cd Awesome-Adaptation-of-Agentic-AI

# View the README
cat README.md

# Browse images
ls images/
```

The repository is primarily a documentation resource (no code to install), but you can:

```bash
# Keep it updated with latest papers
git pull origin main

# Search for specific topics
grep -i "theorem proving" README.md
grep -i "GRPO" README.md
```

## Repository Structure

```
Awesome-Adaptation-of-Agentic-AI/
├── README.md           # Main curated list
├── images/            # Illustrations and timelines
│   ├── intro.png
│   ├── a1_illustrate.png
│   ├── a1_timeline.png
│   ├── paper_icon.png
│   └── code_icon.png
└── LICENSE
```

## Key Usage Patterns

### Finding Papers by Method Type

**RL-Based Agent Adaptation (A1)**:
The repository catalogs methods using reinforcement learning for tool-execution feedback:

```markdown
Examples:
- Orion (2025.11): IR with retrievers, GRPO on LFM2
- AlphaProof (2025.10): Formal theorem proving with Lean, AlphaZero
- Tool-R1 (2025.09): General tool reasoning, GRPO on Qwen2.5
- DeepSeek-R1-Zero (2025.01): Coding with code executor, GRPO
```

**SFT & DPO Methods (A1)**:
```markdown
Examples:
- AWL (2024.12): Scientific reasoning, SFT+DPO on Llama-3.1/Qwen-2.5
- ToolLLM (2023.07): Real-world API planning, SFT on LLaMA/Vicuna
- RetPO (2024.02): IR with retrievers, SFT+DPO on LLaMA2
```

### Searching by Task Domain

```bash
# Find all papers on formal theorem proving
grep -A 1 "Formal Theorem Proving" README.md

# Find papers on coding tasks
grep -A 1 "Coding" README.md | grep -E "Time|Method|Paper"

# Find papers using specific tools (e.g., Lean compiler)
grep "Lean" README.md
```

### Filtering by Agent Backbone

```python
# Example: Extract papers using Qwen models
import re

with open('README.md', 'r') as f:
    content = f.read()

# Find all table rows mentioning Qwen
qwen_papers = re.findall(r'\|.*Qwen.*\|', content)
for paper in qwen_papers[:5]:
    print(paper)
```

### Accessing Paper Links Programmatically

```python
import re
import requests

# Extract arXiv links from README
with open('README.md', 'r') as f:
    readme = f.read()

# Pattern: [Paper](https://arxiv.org/abs/XXXX.XXXXX)
arxiv_pattern = r'\[Paper\]\((https://arxiv\.org/abs/[\d.]+)\)'
papers = re.findall(arxiv_pattern, readme)

print(f"Found {len(papers)} arXiv papers")
for i, url in enumerate(papers[:3], 1):
    print(f"{i}. {url}")
```

### Tracking Development Timelines

The repository includes timeline visualizations for method evolution:

```markdown
A1 Methods Timeline: images/a1_timeline.png
- Shows progression from 2023 (ToolLLM) to 2025 (Orion, olmOCR2)
- Visualizes the shift from SFT-only to RL-based approaches
```

## Common Research Workflows

### 1. Finding Recent Work in a Specific Domain

```bash
# Find 2025 papers on tool-calling
grep "2025" README.md | grep -i "tool-calling"

# Find papers with code available
grep -B 2 "Code\]" README.md | grep "Method"
```

### 2. Comparing Training Methods

```python
# Count papers by training method
import re

with open('README.md', 'r') as f:
    content = f.read()

methods = ['GRPO', 'PPO', 'SFT', 'DPO', 'AlphaZero']
for method in methods:
    count = len(re.findall(method, content))
    print(f"{method}: {count} papers")
```

### 3. Building a Reading List

```python
# Extract papers published in top venues
import re

venues = ['Nature', 'NeurIPS', 'ICML', 'ICLR', 'NAACL']
venue_pattern = r'\| \d+\.\d+ \| (.*?) \| (' + '|'.join(venues) + r").*?\[Paper\]\((.*?)\)"

with open('README.md', 'r') as f:
    content = f.read()

matches = re.findall(venue_pattern, content)
for method, venue, url in matches[:5]:
    print(f"- {method.strip()} ({venue}): {url}")
```

### 4. Tracking Code Implementations

```bash
# Find all GitHub repositories linked
grep -o "https://github.com/[^)]*" README.md | sort -u > github_repos.txt

# Count papers with available code
grep -c "Code\]" README.md
```

## Working with Paper Metadata

### Extracting Structured Data

```python
import re

def parse_table_row(row):
    """Parse a markdown table row into structured data"""
    parts = [p.strip() for p in row.split('|')[1:-1]]
    if len(parts) < 6:
        return None
    
    # Extract paper link
    paper_match = re.search(r'\[Paper\]\((.*?)\)', parts[2])
    code_match = re.search(r'\[Code\]\((.*?)\)', parts[2])
    
    return {
        'date': parts[0],
        'method': parts[1].split()[0],
        'venue': parts[2].split('<br>')[0],
        'paper_url': paper_match.group(1) if paper_match else None,
        'code_url': code_match.group(1) if code_match else None,
        'task': parts[3],
        'tools': parts[4],
        'backbone': parts[5],
        'tuning': parts[6] if len(parts) > 6 else None
    }

# Example usage
with open('README.md', 'r') as f:
    lines = f.readlines()

table_rows = [l for l in lines if l.startswith('| 202')]
papers = [parse_table_row(row) for row in table_rows[:5]]

for paper in papers:
    if paper:
        print(f"{paper['method']}: {paper['task']} using {paper['tuning']}")
```

## Citation Information

When using this repository in research:

```bibtex
@article{jiang2025adaptation,
  title={Adaptation of Agentic AI},
  author={Jiang, Pengcheng and Lin, Jiacheng and Shi, Zhiyi and Wang, Zifeng and He, Luxi and Wu, Yichen and Zhong, Ming and Song, Peiyang and Zhang, Qizheng and Wang, Heng and others},
  journal={arXiv preprint arXiv:2512.16301},
  year={2025}
}
```

## Configuration for Automation

### Setting Up Alerts for New Papers

```python
# monitor_updates.py
import subprocess
import time

def check_for_updates():
    """Check if repository has new commits"""
    result = subprocess.run(
        ['git', 'fetch', 'origin'],
        capture_output=True,
        text=True
    )
    
    status = subprocess.run(
        ['git', 'status', '-uno'],
        capture_output=True,
        text=True
    ).stdout
    
    if 'behind' in status:
        print("New papers available!")
        subprocess.run(['git', 'pull'])
        return True
    return False

# Run weekly
while True:
    check_for_updates()
    time.sleep(604800)  # 1 week
```

### Building a Custom Filter

```python
# filter_papers.py
import re
import sys

def filter_papers(task=None, backbone=None, year=None, has_code=False):
    """Filter papers by criteria"""
    with open('README.md', 'r') as f:
        content = f.read()
    
    # Split into table rows
    rows = [l for l in content.split('\n') if l.startswith('| 202')]
    
    results = []
    for row in rows:
        if year and not row.startswith(f'| {year}'):
            continue
        if task and task.lower() not in row.lower():
            continue
        if backbone and backbone.lower() not in row.lower():
            continue
        if has_code and '[Code]' not in row:
            continue
        results.append(row)
    
    return results

# Usage
if __name__ == '__main__':
    papers = filter_papers(task='coding', has_code=True, year='2025')
    for paper in papers:
        print(paper)
```

## Troubleshooting

### Images Not Displaying

```bash
# Ensure you're viewing on GitHub or have images/ folder
ls images/

# If cloned, images should be present
# If viewing raw README, use absolute GitHub URLs
```

### Finding Specific Venues

```bash
# List all unique venues
grep -E "^\| 202" README.md | cut -d'|' -f3 | cut -d'<' -f1 | sort -u

# Find all NeurIPS papers
grep "NeurIPS" README.md
```

### Broken Links

```python
# check_links.py
import re
import requests

with open('README.md', 'r') as f:
    content = f.read()

links = re.findall(r'https://[^\s\)]+', content)

for link in links[:10]:  # Test first 10
    try:
        response = requests.head(link, timeout=5)
        if response.status_code >= 400:
            print(f"Broken: {link}")
    except:
        print(f"Error checking: {link}")
```

## Advanced Usage

### Generating Summary Statistics

```python
# statistics.py
import re
from collections import Counter

with open('README.md', 'r') as f:
    content = f.read()

# Count by year
years = re.findall(r'\| (202\d)\.\d+', content)
print("Papers by year:", Counter(years))

# Count by training method
methods = re.findall(r'\| (GRPO|PPO|SFT|DPO|AlphaZero)', content)
print("Papers by method:", Counter(methods))

# Count by backbone
backbones = re.findall(r'\| (Qwen|LLaMA|DeepSeek|GPT)[^\|]*\|', content)
print("Papers by backbone:", Counter([b.strip().split()[0] for b in backbones]))
```

### Creating a Local Database

```python
# build_db.py
import sqlite3
import re

conn = sqlite3.connect('papers.db')
c = conn.cursor()

c.execute('''CREATE TABLE IF NOT EXISTS papers
             (method TEXT, date TEXT, venue TEXT, task TEXT, 
              backbone TEXT, tuning TEXT, paper_url TEXT, code_url TEXT)''')

with open('README.md', 'r') as f:
    content = f.read()

# Parse and insert (simplified)
rows = [l for l in content.split('\n') if l.startswith('| 202')]
for row in rows:
    parts = [p.strip() for p in row.split('|')[1:-1]]
    if len(parts) >= 6:
        c.execute('INSERT INTO papers VALUES (?,?,?,?,?,?,?,?)',
                  (parts[1], parts[0], parts[2], parts[3], 
                   parts[5], parts[6] if len(parts) > 6 else '', '', ''))

conn.commit()
conn.close()
```

## Integration with Research Workflows

### Mendeley/Zotero Import

```bash
# Export BibTeX entries (manual extraction example)
grep -A 10 "Method" README.md | grep "Paper\]" | \
  sed 's/.*\(https:\/\/arxiv.org\/abs\/\([0-9.]*\)\).*/\2/' | \
  while read arxiv_id; do
    curl "https://export.arxiv.org/api/query?id_list=$arxiv_id" >> papers.xml
  done
```

### Notion Database Population

```python
# notion_sync.py (requires notion-client)
# pip install notion-client
from notion_client import Client

notion = Client(auth=os.environ["NOTION_TOKEN"])
database_id = os.environ["NOTION_DATABASE_ID"]

# Parse README and create entries
# (Full implementation depends on your Notion schema)
```

This repository is a living document — contributions are welcome via pull requests to add new papers or correct information.
