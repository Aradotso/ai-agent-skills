---
name: agentic-design-patterns-chinese
description: Chinese translation of Google's Agentic Design Patterns book - 21 core AI agent patterns with examples
triggers:
  - "agentic design patterns chinese translation"
  - "AI agent design patterns in Chinese"
  - "how to implement prompt chaining in Chinese"
  - "multi-agent collaboration patterns"
  - "RAG knowledge retrieval patterns"
  - "AI agent memory management strategies"
  - "tool use patterns for AI agents"
  - "human-in-the-loop AI patterns"
---

# Agentic Design Patterns (Chinese Translation)

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This project is a comprehensive Chinese translation of Google's "Agentic Design Patterns" book, covering 21 core patterns for building intelligent AI agent systems, plus 7 appendices and additional resources.

## Overview

The book systematically introduces AI agent design patterns from basic to advanced:

- **Basic Patterns**: Prompt Chaining, Routing, Parallelization
- **Intermediate Patterns**: Reflection, Tool Use, Planning
- **Advanced Patterns**: Multi-Agent Collaboration, Memory Management, RAG
- **Practical Patterns**: Safety/Guardrails, Evaluation & Monitoring

## Project Structure

```
agentic-design-patterns/
├── chapters/           # Translated chapters (32 files)
│   ├── Chapter 1_ Prompt Chaining.md
│   ├── Chapter 2_ Routing.md
│   └── ...
├── original/          # Original English chapters
├── images/            # Image assets organized by chapter
├── glossary.md        # Terminology reference
├── progress.md        # Translation progress tracker
└── translation-guide.md  # Translation standards
```

## Accessing the Content

### Online Reading

Visit the deployed GitHub Pages site:
```
https://adp.xindoo.xyz/
```

### Local Setup

1. Clone the repository:
```bash
git clone https://github.com/xindoo/agentic-design-patterns.git
cd agentic-design-patterns
```

2. For local Jekyll server (GitHub Pages style):
```bash
bundle install
bundle exec jekyll serve
# Visit http://localhost:4000
```

3. For GitBook format:
```bash
npm install -g gitbook-cli
gitbook install
gitbook serve
# Visit http://localhost:4001
```

## Key Chapters & Patterns

### Chapter 1: Prompt Chaining (提示链)
Breaking complex tasks into sequential prompts.

```python
# Example: Document analysis chain
def analyze_document(doc):
    # Step 1: Extract key points
    summary_prompt = f"Summarize key points from: {doc}"
    summary = llm.generate(summary_prompt)
    
    # Step 2: Analyze sentiment
    sentiment_prompt = f"Analyze sentiment of: {summary}"
    sentiment = llm.generate(sentiment_prompt)
    
    # Step 3: Generate recommendations
    rec_prompt = f"Based on sentiment {sentiment}, provide recommendations"
    recommendations = llm.generate(rec_prompt)
    
    return recommendations
```

### Chapter 2: Routing (路由)
Directing requests to specialized agents or models.

```python
# Example: Intent-based routing
def route_query(user_query):
    classifier_prompt = f"Classify intent: {user_query}\nOptions: technical, billing, general"
    intent = llm.generate(classifier_prompt)
    
    routes = {
        "technical": technical_agent,
        "billing": billing_agent,
        "general": general_agent
    }
    
    agent = routes.get(intent, general_agent)
    return agent.process(user_query)
```

### Chapter 5: Tool Use (工具使用)
Enabling agents to call external tools and APIs.

```python
# Example: Function calling pattern
tools = [
    {
        "name": "search_database",
        "description": "Search product database",
        "parameters": {"query": "string"}
    },
    {
        "name": "calculate_price",
        "description": "Calculate final price with discount",
        "parameters": {"base_price": "float", "discount": "float"}
    }
]

def agent_with_tools(user_request):
    # Agent decides which tool to use
    response = llm.generate(
        prompt=user_request,
        tools=tools,
        tool_choice="auto"
    )
    
    if response.tool_calls:
        for tool_call in response.tool_calls:
            result = execute_tool(tool_call.name, tool_call.arguments)
            # Feed result back to agent
            final_response = llm.generate(
                context=[user_request, result]
            )
        return final_response
```

### Chapter 7: Multi-Agent Collaboration (多智能体协作)
Coordinating multiple specialized agents.

```python
# Example: Research team pattern
class ResearchTeam:
    def __init__(self):
        self.researcher = Agent("researcher", "Find information")
        self.analyst = Agent("analyst", "Analyze data")
        self.writer = Agent("writer", "Write report")
    
    def collaborate(self, topic):
        # Stage 1: Research
        research_data = self.researcher.execute(
            f"Research topic: {topic}"
        )
        
        # Stage 2: Analysis
        analysis = self.analyst.execute(
            f"Analyze research: {research_data}"
        )
        
        # Stage 3: Writing
        report = self.writer.execute(
            f"Write report based on: {analysis}"
        )
        
        return report
```

### Chapter 14: Knowledge Retrieval (RAG)
Retrieval-Augmented Generation pattern.

```python
# Example: RAG implementation
from sentence_transformers import SentenceTransformer
import faiss

class RAGAgent:
    def __init__(self, knowledge_base):
        self.encoder = SentenceTransformer('all-MiniLM-L6-v2')
        self.index = self._build_index(knowledge_base)
        self.documents = knowledge_base
    
    def _build_index(self, documents):
        embeddings = self.encoder.encode([doc['text'] for doc in documents])
        index = faiss.IndexFlatL2(embeddings.shape[1])
        index.add(embeddings)
        return index
    
    def query(self, question, top_k=3):
        # Retrieve relevant documents
        query_embedding = self.encoder.encode([question])
        distances, indices = self.index.search(query_embedding, top_k)
        
        context = "\n".join([
            self.documents[i]['text'] for i in indices[0]
        ])
        
        # Generate answer with context
        prompt = f"Context: {context}\n\nQuestion: {question}\nAnswer:"
        answer = llm.generate(prompt)
        
        return answer
```

### Chapter 8: Memory Management (记忆管理)
Managing short-term and long-term memory.

```python
# Example: Conversational memory
class ConversationMemory:
    def __init__(self, max_history=10):
        self.short_term = []  # Recent messages
        self.long_term = {}   # Summary of topics
        self.max_history = max_history
    
    def add_message(self, role, content):
        self.short_term.append({"role": role, "content": content})
        
        # Summarize if history too long
        if len(self.short_term) > self.max_history:
            summary = self._summarize_old_messages()
            self._store_to_long_term(summary)
            self.short_term = self.short_term[-self.max_history:]
    
    def get_context(self):
        # Combine long-term summary with recent history
        context = []
        if self.long_term:
            context.append({"role": "system", "content": f"Previous context: {self.long_term}"})
        context.extend(self.short_term)
        return context
```

## Translation Workflow

### Contributing to Translation

1. **Check translation progress**:
```bash
cat progress.md  # View current status
```

2. **Select a chapter** (currently all are in review status):
```markdown
# Update progress.md
- [x] 已翻译 Chapter X
- [ ] 已审核 Chapter X
```

3. **Follow translation guide**:
```bash
cat translation-guide.md  # Review standards
cat glossary.md          # Check terminology
```

4. **Key translation principles**:
- Use glossary for consistent terminology
- Keep code examples in original language
- Preserve markdown structure
- Maintain image paths relative to `images/`

### Terminology Reference

Common AI agent terms (from `glossary.md`):

```markdown
| English | 中文 | Notes |
|---------|------|-------|
| Agent | 智能体 / 代理 | Context-dependent |
| Prompt Chaining | 提示链 | |
| Routing | 路由 | |
| Tool Use | 工具使用 | |
| RAG | 检索增强生成 | Keep acronym |
| Multi-Agent | 多智能体 | |
| Guardrails | 护栏 / 安全防护 | |
| Human-in-the-Loop | 人机协同 | |
```

## Configuration

### GitHub Pages (_config.yml)

```yaml
title: Agentic Design Patterns 中文翻译
description: AI Agent 系统设计模式完整中文指南
url: "https://adp.xindoo.xyz"
baseurl: ""
markdown: kramdown
theme: jekyll-theme-minimal
```

### GitBook (SUMMARY.md)

The book structure is defined in `SUMMARY.md`:
```markdown
# Summary

* [简介](README.md)
* [核心章节](chapters/README.md)
  * [第1章：提示链](chapters/Chapter 1_ Prompt Chaining.md)
  * [第2章：路由](chapters/Chapter 2_ Routing.md)
  ...
* [附录](chapters/README.md)
  * [附录A：高级提示技术](chapters/Appendix A_ Advanced Prompting Techniques.md)
  ...
```

## Common Patterns & Use Cases

### Pattern 1: Sequential Processing (Prompt Chaining)
Use when: Breaking down complex analysis into steps
```python
result = chain_step1() → chain_step2() → chain_step3()
```

### Pattern 2: Parallel Processing (Parallelization)
Use when: Independent subtasks can run concurrently
```python
results = await asyncio.gather(
    task1(), task2(), task3()
)
```

### Pattern 3: Self-Improvement (Reflection)
Use when: Output quality needs iterative refinement
```python
output = generate()
critique = reflect(output)
improved = regenerate(critique)
```

### Pattern 4: Dynamic Routing
Use when: Different inputs need different handling
```python
handler = router.select(input_type)
result = handler.process(input)
```

## Troubleshooting

### Issue: Images not displaying

**Problem**: Image paths broken after translation
```markdown
![Image](../images/chapter-1/diagram.png)  # Wrong
```

**Solution**: Use correct relative paths
```markdown
![Image](images/chapter-1/diagram.png)  # Correct from chapters/
```

### Issue: Inconsistent terminology

**Problem**: Same English term translated differently
```
Agent → 智能体 (Chapter 1)
Agent → 代理 (Chapter 2)  # Inconsistent
```

**Solution**: Always check glossary.md first
```bash
grep "Agent" glossary.md
# Use: 智能体 (preferred) or 代理 (context-specific)
```

### Issue: Jekyll build fails

**Problem**: 
```
Liquid Exception: Invalid Date
```

**Solution**: Check frontmatter dates in markdown files
```yaml
---
# Remove or fix invalid date fields
updated_at: "2026-05-17"  # Future date might cause issues
---
```

### Issue: Missing dependencies

**Problem**: `bundle exec jekyll serve` fails

**Solution**: Install dependencies
```bash
gem install bundler
bundle install
# Or for GitBook:
npm install -g gitbook-cli
gitbook install
```

## Advanced Usage

### Searching the Content

Use grep for term searches:
```bash
# Find all mentions of "RAG"
grep -r "RAG" chapters/

# Find specific pattern implementations
grep -r "def.*agent" chapters/

# Search in Chinese
grep -r "多智能体" chapters/
```

### Extracting Code Examples

```bash
# Extract all Python code blocks from a chapter
sed -n '/```python/,/```/p' chapters/Chapter\ 5_\ Tool\ Use.md
```

### Generating PDF/EPUB

```bash
# Using GitBook
gitbook pdf ./ ./agentic-patterns-zh.pdf
gitbook epub ./ ./agentic-patterns-zh.epub
```

## Resources

- **Online Book**: https://adp.xindoo.xyz/
- **GitHub Repo**: https://github.com/xindoo/agentic-design-patterns
- **Author**: xindoo (https://zxs.io)
- **Translation Guide**: `translation-guide.md`
- **Glossary**: `glossary.md`
- **Progress Tracker**: `progress.md`

## Contributing

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/agentic-design-patterns.git

# Create feature branch
git checkout -b review/chapter-1-improvements

# Make changes and commit
git add chapters/Chapter\ 1_\ Prompt\ Chaining.md
git commit -m "Review and improve Chapter 1 translation"

# Push and create PR
git push origin review/chapter-1-improvements
```

Follow `CONTRIBUTING.md` for detailed guidelines.
