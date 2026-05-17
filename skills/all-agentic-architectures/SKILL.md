---
name: all-agentic-architectures
description: Implementation guide for 17+ agentic AI architectures using LangChain and LangGraph for building sophisticated AI agents
triggers:
  - how do i build an agentic architecture
  - implement reflection pattern for ai agents
  - create multi-agent system with langgraph
  - set up tree of thoughts architecture
  - build react agent with tools
  - implement agent memory with episodic and semantic
  - create self-improving ai agent
  - design meta-controller for specialized agents
---

# All Agentic Architectures Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill provides comprehensive guidance for implementing 17+ state-of-the-art agentic architectures using LangChain and LangGraph. The project offers production-ready implementations of patterns ranging from simple reflection loops to complex multi-agent systems with memory, planning, and self-improvement capabilities.

## What This Project Does

All Agentic Architectures is a comprehensive collection of modern AI agent design patterns implemented as runnable Jupyter notebooks. It covers:

- **Single-Agent Patterns**: Reflection, Tool Use, ReAct, Planning
- **Multi-Agent Systems**: Collaborative teams, Meta-Controllers, Blackboard systems, Ensemble patterns
- **Advanced Memory**: Episodic + Semantic memory, Graph-based world models
- **Safety & Reliability**: Dry-Run Harness, Plan-Execute-Verify, Simulators
- **Self-Improvement**: RLHF-style feedback loops, Metacognitive agents
- **Complex Reasoning**: Tree of Thoughts, Cellular Automata

Each architecture is designed for practical use across different stages of AI system development.

## Installation

### Basic Setup

```bash
# Clone the repository
git clone https://github.com/FareedKhan-dev/all-agentic-architectures.git
cd all-agentic-architectures

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: .\venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Core Dependencies

```bash
pip install langchain langgraph langsmith pydantic
pip install openai anthropic  # For LLM providers
pip install tavily-python  # For search tool
pip install neo4j faiss-cpu  # For memory architectures
pip install jupyter notebook  # For running notebooks
```

### Environment Variables

Create a `.env` file in the project root:

```bash
# LLM Provider (choose one or multiple)
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key
NEBIUS_API_KEY=your_nebius_key

# Tools
TAVILY_API_KEY=your_tavily_key

# Memory Systems
NEO4J_URI=bolt://localhost:7687
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your_neo4j_password

# LangSmith (optional, for tracing)
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your_langsmith_key
LANGCHAIN_PROJECT=agentic-architectures
```

## Core Architecture Patterns

### 1. Reflection Pattern

The Reflection pattern creates a self-critiquing agent that iteratively improves its output.

```python
from langgraph.graph import StateGraph, END
from langchain_core.messages import HumanMessage, AIMessage
from pydantic import BaseModel
from typing import List, TypedDict

class ReflectionState(TypedDict):
    messages: List[HumanMessage | AIMessage]
    iterations: int

def generate_node(state: ReflectionState):
    """Generate initial response"""
    from langchain_openai import ChatOpenAI
    
    llm = ChatOpenAI(model="gpt-4", temperature=0.7)
    response = llm.invoke(state["messages"])
    
    return {
        "messages": state["messages"] + [response],
        "iterations": state["iterations"]
    }

def reflect_node(state: ReflectionState):
    """Critique and improve the response"""
    from langchain_openai import ChatOpenAI
    
    llm = ChatOpenAI(model="gpt-4", temperature=0.3)
    
    reflection_prompt = f"""Review the following response and provide constructive criticism:
    
Response: {state['messages'][-1].content}

Provide specific suggestions for improvement."""
    
    critique = llm.invoke([HumanMessage(content=reflection_prompt)])
    
    improvement_prompt = f"""Original task: {state['messages'][0].content}

Previous response: {state['messages'][-1].content}

Critique: {critique.content}

Provide an improved response addressing the critique."""
    
    improved = llm.invoke([HumanMessage(content=improvement_prompt)])
    
    return {
        "messages": state["messages"] + [critique, improved],
        "iterations": state["iterations"] + 1
    }

def should_continue(state: ReflectionState):
    """Decide whether to continue reflection"""
    if state["iterations"] >= 3:
        return "end"
    return "reflect"

# Build the graph
workflow = StateGraph(ReflectionState)
workflow.add_node("generate", generate_node)
workflow.add_node("reflect", reflect_node)

workflow.set_entry_point("generate")
workflow.add_conditional_edges(
    "generate",
    should_continue,
    {"reflect": "reflect", "end": END}
)
workflow.add_conditional_edges(
    "reflect",
    should_continue,
    {"reflect": "reflect", "end": END}
)

app = workflow.compile()

# Use the reflection agent
result = app.invoke({
    "messages": [HumanMessage(content="Write a Python function to calculate Fibonacci numbers")],
    "iterations": 0
})
```

### 2. ReAct (Reasoning + Acting) Pattern

ReAct dynamically interleaves reasoning and tool use.

```python
from langchain.agents import AgentExecutor, create_react_agent
from langchain_openai import ChatOpenAI
from langchain.tools import Tool
from langchain import hub
from langchain_community.tools.tavily_search import TavilySearchResults

# Define tools
search = TavilySearchResults(max_results=3)

def calculator(expression: str) -> str:
    """Evaluate mathematical expressions"""
    try:
        return str(eval(expression))
    except Exception as e:
        return f"Error: {str(e)}"

tools = [
    Tool(
        name="Search",
        func=search.run,
        description="Useful for searching current information on the internet"
    ),
    Tool(
        name="Calculator",
        func=calculator,
        description="Useful for mathematical calculations. Input should be a valid Python expression."
    )
]

# Create ReAct agent
llm = ChatOpenAI(model="gpt-4", temperature=0)
prompt = hub.pull("hwchase17/react")

agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=5
)

# Execute multi-step reasoning
result = agent_executor.invoke({
    "input": "What is the current population of Tokyo, and what is 15% of that number?"
})
```

### 3. Multi-Agent System

Specialized agents collaborate to solve complex tasks.

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
import operator

class MultiAgentState(TypedDict):
    task: str
    research: str
    code: str
    review: str
    messages: Annotated[list, operator.add]

def research_agent(state: MultiAgentState):
    """Agent specialized in research"""
    from langchain_openai import ChatOpenAI
    from langchain_community.tools.tavily_search import TavilySearchResults
    
    llm = ChatOpenAI(model="gpt-4")
    search = TavilySearchResults()
    
    research_prompt = f"""Research the following task and provide comprehensive background:
    Task: {state['task']}
    
    Provide key technical details and best practices."""
    
    search_results = search.run(state['task'])
    response = llm.invoke(f"{research_prompt}\n\nSearch results: {search_results}")
    
    return {
        "research": response.content,
        "messages": [f"Research Agent: {response.content}"]
    }

def coding_agent(state: MultiAgentState):
    """Agent specialized in writing code"""
    from langchain_openai import ChatOpenAI
    
    llm = ChatOpenAI(model="gpt-4", temperature=0.2)
    
    code_prompt = f"""Based on the research, implement the solution:
    
Task: {state['task']}
Research: {state['research']}

Provide production-ready, well-documented code."""
    
    response = llm.invoke(code_prompt)
    
    return {
        "code": response.content,
        "messages": [f"Coding Agent: {response.content}"]
    }

def review_agent(state: MultiAgentState):
    """Agent specialized in code review"""
    from langchain_openai import ChatOpenAI
    
    llm = ChatOpenAI(model="gpt-4", temperature=0)
    
    review_prompt = f"""Review the following code for quality, security, and best practices:

Task: {state['task']}
Code:
{state['code']}

Provide detailed feedback and suggestions."""
    
    response = llm.invoke(review_prompt)
    
    return {
        "review": response.content,
        "messages": [f"Review Agent: {response.content}"]
    }

# Build multi-agent workflow
workflow = StateGraph(MultiAgentState)
workflow.add_node("research", research_agent)
workflow.add_node("code", coding_agent)
workflow.add_node("review", review_agent)

workflow.set_entry_point("research")
workflow.add_edge("research", "code")
workflow.add_edge("code", "review")
workflow.add_edge("review", END)

app = workflow.compile()

# Execute multi-agent collaboration
result = app.invoke({
    "task": "Build a REST API rate limiter using Redis",
    "research": "",
    "code": "",
    "review": "",
    "messages": []
})
```

### 4. Tree of Thoughts

Explore multiple reasoning paths systematically.

```python
from typing import List, Dict, TypedDict
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI

class ThoughtNode(TypedDict):
    content: str
    score: float
    depth: int

class ToTState(TypedDict):
    problem: str
    thoughts: List[ThoughtNode]
    best_path: List[str]
    max_depth: int

def generate_thoughts(state: ToTState):
    """Generate multiple reasoning branches"""
    llm = ChatOpenAI(model="gpt-4", temperature=0.8)
    
    current_depth = max([t["depth"] for t in state["thoughts"]], default=0)
    
    # Get the best thoughts from current level
    current_thoughts = [t for t in state["thoughts"] if t["depth"] == current_depth]
    
    new_thoughts = []
    for thought in current_thoughts[:3]:  # Expand top 3 thoughts
        prompt = f"""Problem: {state['problem']}

Current reasoning: {thought['content']}

Generate 3 different next steps or reasoning paths. Be creative and explore alternatives."""
        
        response = llm.invoke(prompt)
        
        # Parse and create new thought nodes
        for i, line in enumerate(response.content.split('\n\n')):
            if line.strip():
                new_thoughts.append({
                    "content": thought['content'] + " -> " + line.strip(),
                    "score": 0.0,
                    "depth": current_depth + 1
                })
    
    return {"thoughts": state["thoughts"] + new_thoughts}

def evaluate_thoughts(state: ToTState):
    """Score each thought based on quality"""
    llm = ChatOpenAI(model="gpt-4", temperature=0.2)
    
    current_depth = max([t["depth"] for t in state["thoughts"]])
    current_thoughts = [t for t in state["thoughts"] if t["depth"] == current_depth]
    
    evaluated_thoughts = []
    for thought in current_thoughts:
        eval_prompt = f"""Problem: {state['problem']}

Reasoning path: {thought['content']}

Rate this reasoning path from 0.0 to 1.0 based on:
- Logical soundness
- Progress toward solution
- Creativity

Respond with only a number."""
        
        response = llm.invoke(eval_prompt)
        try:
            score = float(response.content.strip())
        except:
            score = 0.5
        
        thought["score"] = score
        evaluated_thoughts.append(thought)
    
    # Keep all previous thoughts plus newly evaluated ones
    all_thoughts = [t for t in state["thoughts"] if t["depth"] < current_depth] + evaluated_thoughts
    
    return {"thoughts": all_thoughts}

def should_continue(state: ToTState):
    """Decide whether to continue exploring"""
    current_depth = max([t["depth"] for t in state["thoughts"]], default=0)
    
    if current_depth >= state["max_depth"]:
        return "finalize"
    
    return "generate"

def finalize_solution(state: ToTState):
    """Select and return the best reasoning path"""
    # Find the best thought at maximum depth
    max_depth = max([t["depth"] for t in state["thoughts"]])
    final_thoughts = [t for t in state["thoughts"] if t["depth"] == max_depth]
    
    best_thought = max(final_thoughts, key=lambda x: x["score"])
    
    return {"best_path": best_thought["content"].split(" -> ")}

# Build ToT workflow
workflow = StateGraph(ToTState)
workflow.add_node("generate", generate_thoughts)
workflow.add_node("evaluate", evaluate_thoughts)
workflow.add_node("finalize", finalize_solution)

workflow.set_entry_point("generate")
workflow.add_edge("generate", "evaluate")
workflow.add_conditional_edges(
    "evaluate",
    should_continue,
    {"generate": "generate", "finalize": "finalize"}
)
workflow.add_edge("finalize", END)

app = workflow.compile()

# Solve complex problem with ToT
result = app.invoke({
    "problem": "Design a distributed caching system for a social media platform",
    "thoughts": [{
        "content": "Starting analysis",
        "score": 1.0,
        "depth": 0
    }],
    "best_path": [],
    "max_depth": 3
})
```

### 5. Episodic + Semantic Memory

Combine conversational history with structured knowledge.

```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_community.graphs import Neo4jGraph
from typing import List, Dict

class DualMemoryAgent:
    def __init__(self):
        # Episodic memory (vector store for conversations)
        self.embeddings = OpenAIEmbeddings()
        self.episodic_memory = FAISS.from_texts(
            ["Initial conversation"],
            self.embeddings
        )
        
        # Semantic memory (graph database for facts)
        self.semantic_memory = Neo4jGraph(
            url=os.getenv("NEO4J_URI"),
            username=os.getenv("NEO4J_USERNAME"),
            password=os.getenv("NEO4J_PASSWORD")
        )
        
        from langchain_openai import ChatOpenAI
        self.llm = ChatOpenAI(model="gpt-4")
    
    def add_episodic_memory(self, conversation: str):
        """Store conversation in vector database"""
        self.episodic_memory.add_texts([conversation])
    
    def add_semantic_fact(self, subject: str, relation: str, object: str):
        """Store structured fact in graph database"""
        query = f"""
        MERGE (s:Entity {{name: '{subject}'}})
        MERGE (o:Entity {{name: '{object}'}})
        MERGE (s)-[r:{relation}]->(o)
        """
        self.semantic_memory.query(query)
    
    def retrieve_episodic(self, query: str, k: int = 3) -> List[str]:
        """Retrieve similar past conversations"""
        docs = self.episodic_memory.similarity_search(query, k=k)
        return [doc.page_content for doc in docs]
    
    def retrieve_semantic(self, entity: str) -> str:
        """Retrieve facts about an entity"""
        query = f"""
        MATCH (e:Entity {{name: '{entity}'}})-[r]->(related)
        RETURN type(r) as relation, related.name as object
        LIMIT 10
        """
        results = self.semantic_memory.query(query)
        
        facts = []
        for result in results:
            facts.append(f"{entity} {result['relation']} {result['object']}")
        
        return "\n".join(facts)
    
    def respond(self, user_input: str) -> str:
        """Generate response using both memory types"""
        # Retrieve relevant episodic memories
        past_conversations = self.retrieve_episodic(user_input)
        
        # Extract entities and retrieve semantic facts
        entity_prompt = f"Extract the main entity from: {user_input}. Respond with only the entity name."
        entity_response = self.llm.invoke(entity_prompt)
        entity = entity_response.content.strip()
        
        semantic_facts = self.retrieve_semantic(entity)
        
        # Generate response with context
        response_prompt = f"""User: {user_input}

Relevant past conversations:
{chr(10).join(past_conversations)}

Known facts:
{semantic_facts}

Provide a helpful response using this context."""
        
        response = self.llm.invoke(response_prompt)
        
        # Store this interaction
        self.add_episodic_memory(f"User: {user_input}\nAssistant: {response.content}")
        
        return response.content

# Usage
agent = DualMemoryAgent()

# Add some semantic facts
agent.add_semantic_fact("Python", "IS_A", "Programming Language")
agent.add_semantic_fact("Python", "CREATED_BY", "Guido van Rossum")
agent.add_semantic_fact("Python", "USED_FOR", "Data Science")

# Interact with memory-enhanced agent
response = agent.respond("Tell me about Python")
```

### 6. Meta-Controller Pattern

Route tasks to specialized sub-agents.

```python
from enum import Enum
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, END

class AgentType(str, Enum):
    RESEARCH = "research"
    CODING = "coding"
    WRITING = "writing"
    GENERAL = "general"

class MetaControllerState(TypedDict):
    user_input: str
    agent_type: AgentType
    final_response: str

def meta_controller(state: MetaControllerState):
    """Analyze task and route to appropriate specialist"""
    from langchain_openai import ChatOpenAI
    
    llm = ChatOpenAI(model="gpt-4", temperature=0)
    
    routing_prompt = f"""Analyze this user request and classify it:

User request: {state['user_input']}

Categories:
- research: Questions requiring web search or current information
- coding: Programming, debugging, or technical implementation
- writing: Content creation, editing, or creative writing
- general: Casual conversation or simple questions

Respond with only one word: research, coding, writing, or general"""
    
    response = llm.invoke(routing_prompt)
    agent_type = response.content.strip().lower()
    
    return {"agent_type": AgentType(agent_type)}

def research_specialist(state: MetaControllerState):
    """Handle research-intensive queries"""
    from langchain_openai import ChatOpenAI
    from langchain_community.tools.tavily_search import TavilySearchResults
    
    llm = ChatOpenAI(model="gpt-4")
    search = TavilySearchResults(max_results=5)
    
    # Perform search
    search_results = search.run(state['user_input'])
    
    # Synthesize response
    synthesis_prompt = f"""User question: {state['user_input']}

Search results:
{search_results}

Provide a comprehensive, well-sourced answer."""
    
    response = llm.invoke(synthesis_prompt)
    return {"final_response": response.content}

def coding_specialist(state: MetaControllerState):
    """Handle coding and technical queries"""
    from langchain_openai import ChatOpenAI
    
    llm = ChatOpenAI(model="gpt-4", temperature=0.2)
    
    coding_prompt = f"""You are an expert programmer. Help with this request:

{state['user_input']}

Provide clean, well-documented code with explanations."""
    
    response = llm.invoke(coding_prompt)
    return {"final_response": response.content}

def writing_specialist(state: MetaControllerState):
    """Handle creative and content writing"""
    from langchain_openai import ChatOpenAI
    
    llm = ChatOpenAI(model="gpt-4", temperature=0.8)
    
    writing_prompt = f"""You are a skilled writer. Help with this request:

{state['user_input']}

Be creative, engaging, and well-structured."""
    
    response = llm.invoke(writing_prompt)
    return {"final_response": response.content}

def general_specialist(state: MetaControllerState):
    """Handle general conversation"""
    from langchain_openai import ChatOpenAI
    
    llm = ChatOpenAI(model="gpt-4")
    response = llm.invoke(state['user_input'])
    return {"final_response": response.content}

def route_to_specialist(state: MetaControllerState) -> Literal["research", "coding", "writing", "general"]:
    """Route based on classified agent type"""
    return state["agent_type"].value

# Build meta-controller workflow
workflow = StateGraph(MetaControllerState)

workflow.add_node("controller", meta_controller)
workflow.add_node("research", research_specialist)
workflow.add_node("coding", coding_specialist)
workflow.add_node("writing", writing_specialist)
workflow.add_node("general", general_specialist)

workflow.set_entry_point("controller")
workflow.add_conditional_edges(
    "controller",
    route_to_specialist,
    {
        "research": "research",
        "coding": "coding",
        "writing": "writing",
        "general": "general"
    }
)

workflow.add_edge("research", END)
workflow.add_edge("coding", END)
workflow.add_edge("writing", END)
workflow.add_edge("general", END)

app = workflow.compile()

# Use meta-controller
result = app.invoke({
    "user_input": "What are the latest developments in quantum computing?",
    "agent_type": AgentType.GENERAL,
    "final_response": ""
})
```

## Configuration Patterns

### LangSmith Tracing

Enable detailed tracing for debugging and monitoring:

```python
import os

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = os.getenv("LANGSMITH_API_KEY")
os.environ["LANGCHAIN_PROJECT"] = "my-agentic-project"

# All LangChain/LangGraph calls will now be traced
```

### Custom LLM Configuration

```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic

# Configure different models for different tasks
creative_llm = ChatOpenAI(
    model="gpt-4",
    temperature=0.9,
    max_tokens=2000
)

analytical_llm = ChatOpenAI(
    model="gpt-4",
    temperature=0.2,
    max_tokens=1000
)

# Use Anthropic for longer context
long_context_llm = ChatAnthropic(
    model="claude-3-opus-20240229",
    max_tokens=4096
)
```

### Checkpointing for Long-Running Agents

```python
from langgraph.checkpoint.sqlite import SqliteSaver

# Add persistence to any LangGraph workflow
memory = SqliteSaver.from_conn_string("checkpoints.db")

app = workflow.compile(checkpointer=memory)

# Run with thread_id for persistence
config = {"configurable": {"thread_id": "user-123"}}
result = app.invoke(initial_state, config)

# Resume from checkpoint
continued = app.invoke(new_input, config)
```

## Common Patterns and Best Practices

### Pattern 1: LLM-as-a-Judge Evaluation

```python
def evaluate_agent_output(task: str, output: str) -> dict:
    """Use LLM to evaluate agent performance"""
    from langchain_openai import ChatOpenAI
    
    judge_llm = ChatOpenAI(model="gpt-4", temperature=0)
    
    eval_prompt = f"""Evaluate this AI agent output:

Task: {task}
Output: {output}

Rate on a scale of 1-10 for:
1. Correctness
2. Completeness
3. Clarity
4. Efficiency

Provide scores in JSON format:
{{"correctness": X, "completeness": X, "clarity": X, "efficiency": X, "reasoning": "..."}}
"""
    
    response = judge_llm.invoke(eval_prompt)
    
    import json
    return json.loads(response.content)
```

### Pattern 2: Error Handling in Agents

```python
from typing import TypedDict
from langgraph.graph import StateGraph, END

class RobustAgentState(TypedDict):
    input: str
    output: str
    error: str
    retry_count: int

def safe_agent_node(state: RobustAgentState):
    """Agent with error handling"""
    try:
        from langchain_openai import ChatOpenAI
        llm = ChatOpenAI(model="gpt-4", timeout=30)
        
        response = llm.invoke(state["input"])
        
        return {
            "output": response.content,
            "error": "",
            "retry_count": state["retry_count"]
        }
    
    except Exception as e:
        return {
            "output": "",
            "error": str(e),
            "retry_count": state["retry_count"] + 1
        }

def should_retry(state: RobustAgentState) -> str:
    """Decide whether to retry on error"""
    if state["error"] and state["retry_count"] < 3:
        return "retry"
    elif state["error"]:
        return "failed"
    return "success"
```

### Pattern 3: Streaming Responses

```python
from langchain_openai import ChatOpenAI

async def stream_agent_response(user_input: str):
    """Stream agent responses in real-time"""
    llm = ChatOpenAI(model="gpt-4", streaming=True)
    
    async for chunk in llm.astream(user_input):
        print(chunk.content, end="", flush=True)
        # Or yield chunk for web frameworks
        yield chunk.content
```

## Running Notebooks

### Start Jupyter

```bash
jupyter notebook
```

### Recommended Notebook Order

1. **Start with basics**: `01_reflection.ipynb`, `02_tool_use.ipynb`, `03_ReAct.ipynb`
2. **Multi-agent fundamentals**: `05_multi_agent.ipynb`, `11_meta_controller.ipynb`
3. **Advanced memory**: `08_episodic_with_semantic.ipynb`, `12_graph.ipynb`
4. **Safety patterns**: `06_PEV.ipynb`, `14_dry_run.ipynb`, `17_reflexive_metacognitive.ipynb`
5. **Complex reasoning**: `09_tree_of_thoughts.ipynb`, `10_mental_loop.ipynb`

### Execute Programmatically

```python
import nbformat
from nbconvert.preprocessors import ExecutePreprocessor

def run_notebook(notebook_path: str):
    """Execute a notebook programmatically"""
    with open(notebook_path) as f:
        nb = nbformat.read(f, as_version=4)
    
    ep = ExecutePreprocessor(timeout=600, kernel_name='python3')
    ep.preprocess(nb, {'metadata': {'path': './'}})
    
    return nb
```

## Troubleshooting

### API Rate Limits

```python
from langchain_openai import ChatOpenAI
from langchain.callbacks import get_openai_callback

# Track token usage
with get_openai_callback() as cb:
    response = llm.invoke("Your query")
    print(f"Tokens used: {cb.total_tokens}")
    print(f"Cost: ${cb.total_cost}")

# Add retry logic
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
def resilient_llm_call(prompt):
    llm = ChatOpenAI(model="gpt-4")
    return llm.invoke(prompt)
```

### Memory Issues with Large Graphs

```python
# Use streaming for large outputs
from langgraph.graph import StateGraph

# Limit state retention
def trim_messages(state: dict, max_messages: int = 10):
    """Keep only recent messages"""
    if len(state.get("messages", [])) > max_messages:
        state["messages"] = state["messages"][-max_messages:]
    return state
```

### Neo4j Connection Issues

```python
from neo4j import GraphDatabase

def test_neo4j_connection():
    """Verify Neo4j connectivity"""
    try:
        driver = GraphDatabase.driver(
            os.getenv("NEO4J_URI"),
            auth=(
                os.getenv("NEO4J_USERNAME"),
                os.getenv("NEO4J_PASSWORD")
            )
        )
        
        with driver.session() as session:
            result = session.run("RETURN 1 AS test")
            print("✓ Neo4j connection successful")
            return True
    
    except Exception as e:
        print(f"✗ Neo4j connection failed: {e}")
        return False
    finally:
        driver.close()
```

### Debugging LangGraph Workflows

```python
from langgraph.graph import StateGraph

# Enable verbose output
workflow = StateGraph(StateType)
# ... add nodes ...

app = workflow.compile(debug=True)

# Visualize the graph
from IPython.display import Image, display

display(Image(app.get_graph().draw_mermaid_png()))

# Step through execution
for step in app.stream(initial_state):
    print(f"Step: {step}")
```

## Integration Examples

### FastAPI Integration

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()
