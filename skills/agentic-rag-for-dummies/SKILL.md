---
name: agentic-rag-for-dummies
description: Build modular Agentic RAG systems with LangGraph, featuring hierarchical indexing, conversation memory, and multi-agent query processing
triggers:
  - build an agentic rag system
  - implement retrieval augmented generation with agents
  - create a langgraph rag pipeline
  - set up hierarchical document indexing for rag
  - add conversation memory to rag
  - implement multi-agent query decomposition
  - build a rag system with query clarification
  - create a self-correcting retrieval agent
---

# Agentic RAG for Dummies

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill enables you to build modular Agentic RAG (Retrieval-Augmented Generation) systems using LangGraph. The framework provides hierarchical document indexing, conversation memory, query clarification with human-in-the-loop, multi-agent map-reduce for complex queries, self-correction, and context compression.

## What This Project Does

Agentic RAG for Dummies is a production-ready framework for building intelligent document retrieval systems that go beyond basic RAG:

- **Hierarchical Indexing**: Search small child chunks for precision, retrieve large parent chunks for context
- **Conversation Memory**: Maintains dialogue context across multiple questions
- **Query Clarification**: Rewrites ambiguous queries or pauses for human clarification
- **Multi-Agent Orchestration**: Decomposes complex queries into parallel sub-agents using LangGraph
- **Self-Correction**: Automatically re-queries when initial results are insufficient
- **Context Compression**: Prevents redundant retrievals across long conversations
- **Provider Agnostic**: Works with Ollama, OpenAI, Anthropic, Google, or any LangChain-supported LLM

## Installation

### Clone and Set Up Environment

```bash
git clone https://github.com/GiovanniPasq/agentic-rag-for-dummies.git
cd agentic-rag-for-dummies
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Install Ollama (for Local LLMs)

```bash
# Download from https://ollama.com or use:
curl -fsSL https://ollama.com/install.sh | sh

# Pull a recommended model (7B+ for reliable tool calling)
ollama pull qwen3:4b-instruct-2507-q4_K_M
# Or for better performance:
ollama pull llama3.1:8b-instruct-q4_K_M
```

### For Cloud Providers

```bash
# OpenAI
pip install langchain-openai
export OPENAI_API_KEY="your-key-here"

# Anthropic
pip install langchain-anthropic
export ANTHROPIC_API_KEY="your-key-here"

# Google
pip install langchain-google-genai
export GOOGLE_API_KEY="your-key-here"
```

## Core Configuration

### Initialize Components

```python
import os
from pathlib import Path
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_qdrant.fastembed_sparse import FastEmbedSparse
from qdrant_client import QdrantClient
from langchain_ollama import ChatOllama

# Directory structure
DOCS_DIR = "docs"  # Your PDF files
MARKDOWN_DIR = "markdown_docs"  # Converted markdown
PARENT_STORE_PATH = "parent_store"  # Parent chunk storage
CHILD_COLLECTION = "document_child_chunks"  # Vector DB collection

os.makedirs(DOCS_DIR, exist_ok=True)
os.makedirs(MARKDOWN_DIR, exist_ok=True)
os.makedirs(PARENT_STORE_PATH, exist_ok=True)

# Initialize LLM (swap provider easily)
llm = ChatOllama(model="qwen3:4b-instruct-2507-q4_K_M", temperature=0)

# Embeddings for hybrid search
dense_embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-mpnet-base-v2"
)
sparse_embeddings = FastEmbedSparse(model_name="Qdrant/bm25")

# Vector database
client = QdrantClient(path="qdrant_db")
```

### Switch LLM Providers

```python
# OpenAI
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# Anthropic
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-sonnet-4-5-20250929", temperature=0)

# Google
from langchain_google_genai import ChatGoogleGenerativeAI
llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)
```

## Document Processing Pipeline

### 1. Convert PDFs to Markdown

```python
import pymupdf
import pymupdf4llm
import glob

def pdf_to_markdown(pdf_path, output_dir):
    """Convert a single PDF to Markdown."""
    doc = pymupdf.open(pdf_path)
    md = pymupdf4llm.to_markdown(
        doc, 
        header=False, 
        footer=False, 
        page_separators=True,
        ignore_images=True
    )
    md_cleaned = md.encode('utf-8', errors='surrogatepass').decode('utf-8', errors='ignore')
    output_path = Path(output_dir) / Path(doc.name).stem
    Path(output_path).with_suffix(".md").write_bytes(md_cleaned.encode('utf-8'))

def pdfs_to_markdowns(path_pattern, overwrite=False):
    """Convert all PDFs matching pattern."""
    output_dir = Path(MARKDOWN_DIR)
    for pdf_path in map(Path, glob.glob(path_pattern)):
        md_path = (output_dir / pdf_path.stem).with_suffix(".md")
        if overwrite or not md_path.exists():
            pdf_to_markdown(pdf_path, output_dir)

# Convert all PDFs in docs directory
pdfs_to_markdowns(f"{DOCS_DIR}/*.pdf")
```

### 2. Hierarchical Chunking (Parent/Child)

```python
from langchain_text_splitters import MarkdownHeaderTextSplitter, RecursiveCharacterTextSplitter
import json

def process_document_hierarchical(markdown_path):
    """Split document into parent and child chunks."""
    content = Path(markdown_path).read_text(encoding='utf-8')
    
    # Parent chunks: split by headers
    header_splitter = MarkdownHeaderTextSplitter(
        headers_to_split_on=[
            ("#", "Header 1"),
            ("##", "Header 2"),
            ("###", "Header 3"),
        ],
        strip_headers=False
    )
    parent_chunks = header_splitter.split_text(content)
    
    # Child chunks: fixed-size from each parent
    child_splitter = RecursiveCharacterTextSplitter(
        chunk_size=500,
        chunk_overlap=100
    )
    
    parent_ids = []
    child_chunks = []
    
    for i, parent in enumerate(parent_chunks):
        parent_id = f"{Path(markdown_path).stem}_parent_{i}"
        parent_ids.append(parent_id)
        
        # Store parent chunk
        parent_data = {
            "id": parent_id,
            "content": parent.page_content,
            "metadata": parent.metadata
        }
        parent_file = Path(PARENT_STORE_PATH) / f"{parent_id}.json"
        parent_file.write_text(json.dumps(parent_data, ensure_ascii=False))
        
        # Create child chunks
        children = child_splitter.split_documents([parent])
        for j, child in enumerate(children):
            child.metadata["parent_id"] = parent_id
            child.metadata["child_index"] = j
            child_chunks.append(child)
    
    return parent_ids, child_chunks
```

### 3. Index Documents in Vector Database

```python
from qdrant_client.http import models as qmodels
from langchain_qdrant import QdrantVectorStore, RetrievalMode

def ensure_collection(collection_name):
    """Create Qdrant collection if it doesn't exist."""
    embedding_dimension = len(dense_embeddings.embed_query("test"))
    
    if not client.collection_exists(collection_name):
        client.create_collection(
            collection_name=collection_name,
            vectors_config=qmodels.VectorParams(
                size=embedding_dimension,
                distance=qmodels.Distance.COSINE
            ),
            sparse_vectors_config={
                "sparse": qmodels.SparseVectorParams()
            },
        )

def index_documents(markdown_files):
    """Index all documents with hierarchical chunking."""
    ensure_collection(CHILD_COLLECTION)
    
    vector_store = QdrantVectorStore(
        client=client,
        collection_name=CHILD_COLLECTION,
        embedding=dense_embeddings,
        sparse_embedding=sparse_embeddings,
        retrieval_mode=RetrievalMode.HYBRID,
    )
    
    all_child_chunks = []
    for md_file in glob.glob(f"{MARKDOWN_DIR}/*.md"):
        parent_ids, child_chunks = process_document_hierarchical(md_file)
        all_child_chunks.extend(child_chunks)
        print(f"Processed {Path(md_file).name}: {len(parent_ids)} parents, {len(child_chunks)} children")
    
    # Batch index all child chunks
    vector_store.add_documents(all_child_chunks)
    return vector_store

# Index all markdown documents
vector_store = index_documents(f"{MARKDOWN_DIR}/*.md")
```

## Building the Agentic RAG System

### Define Agent Tools

```python
from langchain_core.tools import tool

@tool
def retrieve_documents(query: str) -> list[str]:
    """
    Search the knowledge base using hybrid search (dense + sparse embeddings).
    Returns relevant document chunks.
    
    Args:
        query: The search query
    """
    results = vector_store.similarity_search(query, k=5)
    return [doc.page_content for doc in results]

@tool
def get_parent_context(parent_id: str) -> str:
    """
    Retrieve the full parent chunk for additional context.
    
    Args:
        parent_id: The parent chunk identifier
    """
    parent_file = Path(PARENT_STORE_PATH) / f"{parent_id}.json"
    if parent_file.exists():
        data = json.loads(parent_file.read_text())
        return data["content"]
    return "Parent chunk not found."

tools = [retrieve_documents, get_parent_context]
```

### Define System Prompts

```python
CONVERSATION_SUMMARIZER_PROMPT = """You are a conversation summarizer. 
Extract key context from the conversation history that is relevant to the current query.
Focus on: entities mentioned, topics discussed, user intent.

Conversation History:
{history}

Current Query: {query}

Provide a concise summary of relevant context."""

QUERY_CLARIFICATION_PROMPT = """You are a query clarification assistant.
Analyze the query and conversation context.

If the query is:
- Ambiguous or contains pronouns without clear referents: Rewrite it clearly
- Multi-part (multiple questions): Split into focused sub-queries
- Clear and focused: Return it unchanged

Context: {context}
Query: {query}

Return a JSON object:
{{
    "needs_clarification": boolean,
    "clarification_question": string or null,
    "rewritten_queries": [list of clear, focused queries]
}}"""

AGENT_PROMPT = """You are a RAG agent. Use the retrieve_documents tool to search for information.
If results are insufficient, try rephrasing your search query.
If you find relevant parent_id metadata, use get_parent_context for full context.

Available tools:
- retrieve_documents(query: str): Search the knowledge base
- get_parent_context(parent_id: str): Get full parent chunk

Question: {query}
Context: {context}

Provide a comprehensive answer based on retrieved documents."""
```

### Define State Models

```python
from typing import TypedDict, Annotated, Sequence
from langgraph.graph import MessagesState
from langchain_core.messages import BaseMessage

class AgentState(TypedDict):
    """State for individual RAG agents."""
    messages: Annotated[Sequence[BaseMessage], "The messages in the conversation"]
    query: str
    context: str
    retrieved_docs: list[str]
    parent_contexts: list[str]
    search_attempts: int
    max_searches: int
    answer: str

class OrchestratorState(TypedDict):
    """State for the main orchestration graph."""
    user_query: str
    conversation_history: list[dict]
    conversation_summary: str
    clarified_queries: list[str]
    needs_human_input: bool
    clarification_question: str
    agent_results: list[dict]
    final_answer: str
```

### Build LangGraph Agent

```python
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from langchain_core.messages import HumanMessage, AIMessage

def should_continue(state: AgentState) -> str:
    """Decide if agent should continue searching or finish."""
    if state["answer"]:
        return "end"
    if state["search_attempts"] >= state["max_searches"]:
        return "end"
    return "continue"

def agent_node(state: AgentState) -> AgentState:
    """Main agent reasoning node."""
    llm_with_tools = llm.bind_tools(tools)
    
    messages = state["messages"]
    if not messages:
        messages = [HumanMessage(content=AGENT_PROMPT.format(
            query=state["query"],
            context=state.get("context", "")
        ))]
    
    response = llm_with_tools.invoke(messages)
    
    # Check if we have a final answer (no tool calls)
    if not response.tool_calls:
        return {
            **state,
            "answer": response.content,
            "messages": messages + [response]
        }
    
    return {
        **state,
        "messages": messages + [response],
        "search_attempts": state["search_attempts"] + 1
    }

def build_agent_graph():
    """Build the RAG agent graph."""
    workflow = StateGraph(AgentState)
    
    workflow.add_node("agent", agent_node)
    workflow.add_node("tools", ToolNode(tools))
    
    workflow.set_entry_point("agent")
    workflow.add_conditional_edges(
        "agent",
        should_continue,
        {
            "continue": "tools",
            "end": END
        }
    )
    workflow.add_edge("tools", "agent")
    
    return workflow.compile()

agent_graph = build_agent_graph()
```

### Multi-Agent Orchestration

```python
from langgraph.graph import StateGraph, END
import json

def summarize_conversation(state: OrchestratorState) -> OrchestratorState:
    """Summarize conversation history for context."""
    history_text = "\n".join([
        f"{msg['role']}: {msg['content']}" 
        for msg in state["conversation_history"][-5:]  # Last 5 messages
    ])
    
    summary_prompt = CONVERSATION_SUMMARIZER_PROMPT.format(
        history=history_text,
        query=state["user_query"]
    )
    summary = llm.invoke(summary_prompt).content
    
    return {**state, "conversation_summary": summary}

def clarify_query(state: OrchestratorState) -> OrchestratorState:
    """Clarify and potentially decompose the query."""
    clarification_prompt = QUERY_CLARIFICATION_PROMPT.format(
        context=state.get("conversation_summary", ""),
        query=state["user_query"]
    )
    
    response = llm.invoke(clarification_prompt).content
    result = json.loads(response)
    
    return {
        **state,
        "needs_human_input": result["needs_clarification"],
        "clarification_question": result.get("clarification_question"),
        "clarified_queries": result["rewritten_queries"]
    }

def route_after_clarification(state: OrchestratorState) -> str:
    """Route based on whether human input is needed."""
    if state["needs_human_input"]:
        return "wait_for_human"
    return "execute_agents"

def execute_parallel_agents(state: OrchestratorState) -> OrchestratorState:
    """Execute multiple agents in parallel for query decomposition."""
    results = []
    
    for query in state["clarified_queries"]:
        agent_state = {
            "messages": [],
            "query": query,
            "context": state.get("conversation_summary", ""),
            "retrieved_docs": [],
            "parent_contexts": [],
            "search_attempts": 0,
            "max_searches": 3,
            "answer": ""
        }
        
        # Run agent graph
        final_state = agent_graph.invoke(agent_state)
        results.append({
            "query": query,
            "answer": final_state["answer"],
            "docs": final_state["retrieved_docs"]
        })
    
    return {**state, "agent_results": results}

def aggregate_results(state: OrchestratorState) -> OrchestratorState:
    """Combine all agent results into final answer."""
    combined = "\n\n".join([
        f"Sub-query: {r['query']}\nAnswer: {r['answer']}"
        for r in state["agent_results"]
    ])
    
    aggregation_prompt = f"""Synthesize these sub-answers into a coherent response:

{combined}

Original question: {state['user_query']}

Provide a unified, well-structured answer."""
    
    final_answer = llm.invoke(aggregation_prompt).content
    
    return {**state, "final_answer": final_answer}

def build_orchestrator_graph():
    """Build the main orchestration graph."""
    workflow = StateGraph(OrchestratorState)
    
    workflow.add_node("summarize", summarize_conversation)
    workflow.add_node("clarify", clarify_query)
    workflow.add_node("execute", execute_parallel_agents)
    workflow.add_node("aggregate", aggregate_results)
    
    workflow.set_entry_point("summarize")
    workflow.add_edge("summarize", "clarify")
    workflow.add_conditional_edges(
        "clarify",
        route_after_clarification,
        {
            "wait_for_human": END,  # Pause for human input
            "execute_agents": "execute"
        }
    )
    workflow.add_edge("execute", "aggregate")
    workflow.add_edge("aggregate", END)
    
    return workflow.compile()

orchestrator_graph = build_orchestrator_graph()
```

## Usage Patterns

### Basic Query Execution

```python
def query_rag_system(user_query: str, conversation_history: list = None):
    """Execute a query through the full agentic RAG system."""
    initial_state = {
        "user_query": user_query,
        "conversation_history": conversation_history or [],
        "conversation_summary": "",
        "clarified_queries": [],
        "needs_human_input": False,
        "clarification_question": "",
        "agent_results": [],
        "final_answer": ""
    }
    
    result = orchestrator_graph.invoke(initial_state)
    
    if result["needs_human_input"]:
        return {
            "needs_clarification": True,
            "question": result["clarification_question"]
        }
    
    return {
        "needs_clarification": False,
        "answer": result["final_answer"],
        "sub_queries": result["clarified_queries"],
        "sources": [r["docs"] for r in result["agent_results"]]
    }

# Example usage
response = query_rag_system(
    "What is the difference between JavaScript and Python?",
    conversation_history=[
        {"role": "user", "content": "Tell me about programming languages"},
        {"role": "assistant", "content": "Programming languages are..."}
    ]
)

if response["needs_clarification"]:
    print(f"Clarification needed: {response['question']}")
else:
    print(f"Answer: {response['answer']}")
    print(f"Decomposed into: {response['sub_queries']}")
```

### Interactive Chat Loop

```python
def chat_loop():
    """Interactive chat session with conversation memory."""
    conversation_history = []
    print("Agentic RAG Chat (type 'quit' to exit)")
    
    while True:
        user_input = input("\nYou: ").strip()
        if user_input.lower() == 'quit':
            break
        
        response = query_rag_system(user_input, conversation_history)
        
        if response["needs_clarification"]:
            print(f"\nBot: {response['question']}")
            clarification = input("You: ").strip()
            # Re-run with clarified input
            response = query_rag_system(clarification, conversation_history)
        
        print(f"\nBot: {response['answer']}")
        
        # Update history
        conversation_history.append({"role": "user", "content": user_input})
        conversation_history.append({"role": "assistant", "content": response['answer']})

# Run interactive chat
chat_loop()
```

### Programmatic Multi-Query

```python
queries = [
    "What is machine learning?",
    "How does neural network training work?",
    "What are common ML frameworks?"
]

results = []
history = []

for q in queries:
    result = query_rag_system(q, history)
    results.append(result)
    
    history.append({"role": "user", "content": q})
    history.append({"role": "assistant", "content": result["answer"]})
    
# Results now contain context-aware answers
for i, r in enumerate(results):
    print(f"\nQ{i+1}: {queries[i]}")
    print(f"A: {r['answer']}\n")
```

## Running the Gradio UI

```python
# Use the provided Gradio interface
python src/chat_app.py

# Or programmatically
from src.chat_app import create_chat_interface

demo = create_chat_interface(
    orchestrator_graph=orchestrator_graph,
    parent_store_path=PARENT_STORE_PATH
)
demo.launch(share=True)
```

## Configuration Options

### Tuning Retrieval Parameters

```python
# Adjust number of retrieved chunks
vector_store.similarity_search(query, k=10)  # Retrieve top 10

# Adjust child chunk size
child_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,  # Larger chunks = more context
    chunk_overlap=200  # More overlap = better boundary handling
)

# Adjust max search attempts per agent
agent_state = {
    "max_searches": 5,  # Allow more self-correction loops
    ...
}
```

### Adjusting Agent Behavior

```python
# More temperature for creative answers
llm = ChatOllama(model="qwen3:4b-instruct-2507-q4_K_M", temperature=0.3)

# More aggressive query decomposition
# Modify QUERY_CLARIFICATION_PROMPT to split more aggressively

# Longer conversation memory
history_text = "\n".join([
    f"{msg['role']}: {msg['content']}" 
    for msg in state["conversation_history"][-10:]  # Last 10 instead of 5
])
```

## Troubleshooting

### Small Models Ignore Tools

**Problem**: Ollama models <7B parameters ignore tool calls or hallucinate answers.

**Solution**:
```bash
# Use larger models for reliable tool calling
ollama pull llama3.1:8b-instruct-q4_K_M
ollama pull mistral:7b-instruct-q4_K_M

# Or switch to cloud providers
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
```

### Qdrant Collection Errors

**Problem**: `Collection already exists` or dimension mismatch errors.

**Solution**:
```python
# Delete and recreate collection
client.delete_collection(CHILD_COLLECTION)
ensure_collection(CHILD_COLLECTION)

# Or use a new collection name
CHILD_COLLECTION = "document_child_chunks_v2"
```

### Parent Chunks Not Found

**Problem**: `get_parent_context` returns "Parent chunk not found".

**Solution**:
```python
# Check parent store exists
print(list(Path(PARENT_STORE_PATH).glob("*.json")))

# Verify metadata in child chunks
results = vector_store.similarity_search("test", k=1)
print(results[0].metadata)  # Should have "parent_id" key
```

### Memory Issues with Large Documents

**Problem**: Out of memory when processing many large PDFs.

**Solution**:
```python
# Process documents in batches
def index_documents_batched(markdown_files, batch_size=10):
    for i in range(0, len(markdown_files), batch_size):
        batch = markdown_files[i:i+batch_size]
        all_child_chunks = []
        for md_file in batch:
            parent_ids, child_chunks = process_document_hierarchical(md_file)
            all_child_chunks.extend(child_chunks)
        vector_store.add_documents(all_child_chunks)
        print(f"Indexed batch {i//batch_size + 1}")
```

### Agent Loops Indefinitely

**Problem**: Agent keeps calling tools without producing an answer.

**Solution**:
```python
# Enforce stricter max_searches
agent_state["max_searches"] = 2

# Add explicit termination in agent_node
def agent_node(state: AgentState) -> AgentState:
    if state["search_attempts"] >= state["max_searches"]:
        return {
            **state,
            "answer": "Unable to find sufficient information after multiple attempts."
        }
    # ... rest of logic
```

### Query Clarification Too Aggressive

**Problem**: System asks for clarification on clear queries.

**Solution**:
```python
# Adjust QUERY_CLARIFICATION_PROMPT
QUERY_CLARIFICATION_PROMPT = """... 
Only set needs_clarification=true if the query is genuinely ambiguous 
(contains unresolved pronouns, missing critical context, or is nonsensical).
..."""

# Or skip clarification node for simple queries
def route_after_clarification(state: OrchestratorState) -> str:
    if len(state["user_query"].split()) < 5:  # Short queries skip
        return "execute_agents"
    if state["needs_human_input"]:
        return "wait_for_human"
    return "execute_agents"
```

## Advanced Patterns

### Add Observability with Langfuse

```python
# Set up Langfuse tracing
import os
os.environ["LANGFUSE_PUBLIC_KEY"] = "your-public-key"
os.environ["LANGFUSE_SECRET_KEY"] = "your-secret-key"
os.environ["LANGFUSE_HOST"] = "https://cloud.langfuse.com"

from langfuse.callback import CallbackHandler

langfuse_handler = CallbackHandler()

# Add to LLM calls
llm = ChatOllama(
    model="qwen3:4b-instruct-2507-q4_K_M",
    temperature=0,
    callbacks=[langfuse_handler]
)

# Trace graph execution
result = orchestrator_graph.invoke(
    initial_state,
    config={"callbacks": [langfuse_handler]}
)
```

### Custom Embedding Models

```python
# Use different embedding for domain-specific docs
from langchain_huggingface import HuggingFaceEmbeddings

# Legal documents
legal_embeddings = HuggingFaceEmbeddings(
    model_name="nlpaueb/legal-bert-base-uncased"
)

# Medical documents
medical_embeddings = HuggingFaceEmbeddings(
    model_name="microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract"
)
```

### Multi-Collection RAG

```python
# Search multiple collections (e.g., different document types)
def multi_collection_retrieve(query: str) -> list[str]:
    results = []
    for collection in ["technical_docs", "user_guides", "api_reference"]:
        store = QdrantVectorStore(
            client=client,
            collection_name=collection,
            embedding=dense_embeddings
        )
        results.extend(store.similarity_search(query, k=2))
    return [doc.page_content for doc in results]
```

## Resources

- **GitHub Repository**: https://github.com/GiovanniPasq/agentic-rag-for-dummies
- **Interactive Notebook**: `notebooks/agentic_rag.ipynb` (or [open in Colab](https://colab.research.google.com/github/GiovanniPasq/agentic-rag-for-dummies/blob/main/notebooks/agentic_rag.ipynb))
- **PDF Conversion Guide**: `notebooks/pdf_to_markdown.ipynb`
- **Chunk Inspection Tool**: [Chunky](https://github.com/GiovanniPasq/chunky)
- **LangGraph Documentation**: https://langchain-ai.github.io/langgraph/
- **Qdrant Documentation**: https://qdrant.tech/documentation/

## Key Takeaways

1. **Always use 7B+ models** for reliable tool calling and instruction following
2. **Hierarchical indexing** (parent/child chunks) balances precision and context
3. **Query clarification** prevents misunderstandings early in the pipeline
4. **Multi-agent decomposition** handles complex queries by parallelizing sub-problems
5. **Self-correction loops** improve answer quality through iterative refinement
6. **Provider-agnostic design** allows seamless switching between local and cloud LLMs

This framework is production-ready and designed for extension — swap components, add new tools, or integrate custom agents as needed.
