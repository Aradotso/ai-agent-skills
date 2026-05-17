---
name: agents-towards-production
description: Build production-ready GenAI agents with stateful workflows, vector memory, deployment, and orchestration using LangGraph and LangChain
triggers:
  - how do I build a production-ready AI agent
  - show me how to deploy an agent with LangGraph
  - how to add memory to my AI agent
  - create a multi-agent system with coordination
  - how do I add web search to my agent
  - deploy an agent with Docker and FastAPI
  - implement agent observability and monitoring
  - build a RAG agent for production
---

# Agents Towards Production

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This skill enables you to build production-grade GenAI agents from prototype to enterprise deployment. The repository provides 28+ end-to-end tutorials covering stateful workflows, vector memory, real-time web search, Docker deployment, FastAPI endpoints, security guardrails, GPU scaling, browser automation, multi-agent coordination, observability, evaluation, and UI development.

## What It Does

Agents Towards Production is a comprehensive tutorial collection for building real-world AI agents that scale. It covers:

- **Agent Frameworks**: LangGraph, LangChain for stateful workflows and orchestration
- **Memory Systems**: Vector storage with Redis, Mem0 for persistent agent memory
- **RAG Integration**: Retrieval-augmented generation with Contextual AI
- **Web Access**: Real-time search APIs (Tavily), web scraping (Bright Data)
- **Deployment**: Docker, FastAPI, GPU scaling, production infrastructure
- **Security**: Guardrails, OAuth2, human-in-the-loop controls (Arcade)
- **Multi-Agent**: Coordination, orchestration, distributed workflows
- **Observability**: Monitoring, evaluation, debugging production agents
- **UI Development**: Browser automation, user interfaces

## Installation

Clone the repository:

```bash
git clone https://github.com/NirDiamant/agents-towards-production.git
cd agents-towards-production
```

Install dependencies (each tutorial has its own requirements):

```bash
# For LangGraph tutorials
pip install langchain langgraph langchain-openai langchain-community

# For memory tutorials
pip install redis langchain-redis mem0ai

# For RAG tutorials
pip install contextual-client chromadb sentence-transformers

# For web access
pip install tavily-python brightdata-sdk

# For deployment
pip install fastapi uvicorn docker pydantic

# For observability
pip install langsmith weave opentelemetry
```

## Key Tutorials and Usage Patterns

### 1. Basic LangGraph Agent

Create a stateful agent with LangGraph:

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, END
from typing import TypedDict, List
import os

# Define state
class AgentState(TypedDict):
    messages: List[dict]
    current_step: str

# Initialize LLM
llm = ChatOpenAI(
    model="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY")
)

# Define agent nodes
def process_input(state: AgentState):
    """Process user input"""
    messages = state["messages"]
    response = llm.invoke(messages)
    return {
        "messages": messages + [{"role": "assistant", "content": response.content}],
        "current_step": "completed"
    }

# Build graph
workflow = StateGraph(AgentState)
workflow.add_node("process", process_input)
workflow.set_entry_point("process")
workflow.add_edge("process", END)

# Compile and run
app = workflow.compile()

result = app.invoke({
    "messages": [{"role": "user", "content": "Hello, how can you help me?"}],
    "current_step": "start"
})
print(result["messages"][-1]["content"])
```

### 2. Agent with Vector Memory (Redis)

Add persistent memory to your agent:

```python
from langchain_redis import RedisVectorStore, RedisConfig
from langchain_openai import OpenAIEmbeddings
from langchain.schema import Document
import os

# Configure Redis
redis_config = RedisConfig(
    index_name="agent_memory",
    redis_url=os.getenv("REDIS_URL", "redis://localhost:6379"),
    distance_metric="COSINE"
)

# Initialize vector store
embeddings = OpenAIEmbeddings(api_key=os.getenv("OPENAI_API_KEY"))
vector_store = RedisVectorStore(
    config=redis_config,
    embedding=embeddings
)

# Store conversation memory
def store_memory(user_id: str, conversation: str, metadata: dict = None):
    """Store conversation in vector memory"""
    doc = Document(
        page_content=conversation,
        metadata={"user_id": user_id, **(metadata or {})}
    )
    vector_store.add_documents([doc])

# Retrieve relevant memories
def retrieve_memory(user_id: str, query: str, k: int = 3):
    """Retrieve relevant past conversations"""
    results = vector_store.similarity_search(
        query,
        k=k,
        filter={"user_id": user_id}
    )
    return [doc.page_content for doc in results]

# Usage
store_memory(
    user_id="user123",
    conversation="User asked about Python tutorials. Agent provided resources.",
    metadata={"topic": "python", "timestamp": "2024-01-15"}
)

memories = retrieve_memory("user123", "What did we discuss about programming?")
print("Relevant memories:", memories)
```

### 3. RAG Agent with Contextual AI

Build a retrieval-augmented generation agent:

```python
from contextual_client import ContextualClient
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
import os

# Initialize Contextual client
contextual = ContextualClient(api_key=os.getenv("CONTEXTUAL_API_KEY"))

# Create knowledge base
kb = contextual.create_knowledge_base(
    name="product_docs",
    description="Product documentation and FAQs"
)

# Index documents
documents = [
    {"content": "Our API supports REST and GraphQL endpoints.", "metadata": {"type": "api"}},
    {"content": "Authentication uses OAuth2 with JWT tokens.", "metadata": {"type": "auth"}},
]

contextual.index_documents(knowledge_base_id=kb.id, documents=documents)

# RAG query function
def rag_query(question: str):
    """Query with retrieval-augmented generation"""
    # Retrieve relevant context
    results = contextual.search(
        knowledge_base_id=kb.id,
        query=question,
        top_k=3
    )
    
    context = "\n".join([r.content for r in results])
    
    # Generate response with context
    prompt = ChatPromptTemplate.from_template("""
    Answer the question based on the following context:
    
    Context: {context}
    
    Question: {question}
    
    Answer:
    """)
    
    llm = ChatOpenAI(model="gpt-4", api_key=os.getenv("OPENAI_API_KEY"))
    chain = prompt | llm
    
    response = chain.invoke({"context": context, "question": question})
    return response.content

# Usage
answer = rag_query("How does authentication work?")
print(answer)
```

### 4. Web Search Agent with Tavily

Add real-time web search capabilities:

```python
from tavily import TavilyClient
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
import os

# Initialize Tavily
tavily = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))

def web_search_agent(query: str):
    """Agent with real-time web search"""
    # Search the web
    search_results = tavily.search(
        query=query,
        search_depth="advanced",
        max_results=5,
        include_domains=None,
        exclude_domains=None
    )
    
    # Format results
    context = "\n\n".join([
        f"Source: {r['url']}\n{r['content']}"
        for r in search_results.get('results', [])
    ])
    
    # Generate response
    prompt = ChatPromptTemplate.from_template("""
    Based on the following web search results, answer the user's question:
    
    Search Results:
    {context}
    
    Question: {question}
    
    Provide a comprehensive answer with sources:
    """)
    
    llm = ChatOpenAI(model="gpt-4", api_key=os.getenv("OPENAI_API_KEY"))
    chain = prompt | llm
    
    response = chain.invoke({"context": context, "question": query})
    return {
        "answer": response.content,
        "sources": [r['url'] for r in search_results.get('results', [])]
    }

# Usage
result = web_search_agent("What are the latest developments in AI agents?")
print(result["answer"])
print("\nSources:", result["sources"])
```

### 5. Multi-Agent Coordination

Coordinate multiple specialized agents:

```python
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from typing import TypedDict, List, Literal
import os

class MultiAgentState(TypedDict):
    messages: List[dict]
    task: str
    current_agent: str
    results: dict

# Define specialized agents
llm = ChatOpenAI(model="gpt-4", api_key=os.getenv("OPENAI_API_KEY"))

def research_agent(state: MultiAgentState):
    """Research specialist"""
    prompt = f"Research the following: {state['task']}"
    response = llm.invoke([{"role": "user", "content": prompt}])
    state["results"]["research"] = response.content
    return state

def analysis_agent(state: MultiAgentState):
    """Analysis specialist"""
    research = state["results"].get("research", "")
    prompt = f"Analyze this research:\n{research}"
    response = llm.invoke([{"role": "user", "content": prompt}])
    state["results"]["analysis"] = response.content
    return state

def synthesis_agent(state: MultiAgentState):
    """Synthesis specialist"""
    analysis = state["results"].get("analysis", "")
    prompt = f"Synthesize findings:\n{analysis}"
    response = llm.invoke([{"role": "user", "content": prompt}])
    state["results"]["synthesis"] = response.content
    return state

# Router function
def route_task(state: MultiAgentState) -> Literal["research", "analysis", "synthesis", "end"]:
    """Route to next agent"""
    if "research" not in state["results"]:
        return "research"
    elif "analysis" not in state["results"]:
        return "analysis"
    elif "synthesis" not in state["results"]:
        return "synthesis"
    return "end"

# Build multi-agent graph
workflow = StateGraph(MultiAgentState)
workflow.add_node("research", research_agent)
workflow.add_node("analysis", analysis_agent)
workflow.add_node("synthesis", synthesis_agent)

workflow.set_entry_point("research")
workflow.add_edge("research", "analysis")
workflow.add_edge("analysis", "synthesis")
workflow.add_edge("synthesis", END)

app = workflow.compile()

# Execute multi-agent workflow
result = app.invoke({
    "messages": [],
    "task": "Latest trends in AI agent deployment",
    "current_agent": "research",
    "results": {}
})

print("Final synthesis:", result["results"]["synthesis"])
```

### 6. FastAPI Deployment

Deploy your agent as a REST API:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, END
from typing import List, Dict
import os
import uvicorn

app = FastAPI(title="Production Agent API")

# Request/Response models
class ChatRequest(BaseModel):
    message: str
    user_id: str
    session_id: str

class ChatResponse(BaseModel):
    response: str
    session_id: str
    metadata: Dict

# Agent state
class AgentState(BaseModel):
    messages: List[Dict]
    user_id: str
    session_id: str

# Initialize agent
llm = ChatOpenAI(model="gpt-4", api_key=os.getenv("OPENAI_API_KEY"))

def create_agent():
    """Create agent workflow"""
    def process(state: dict):
        messages = state["messages"]
        response = llm.invoke(messages)
        state["messages"].append({
            "role": "assistant",
            "content": response.content
        })
        return state
    
    workflow = StateGraph(dict)
    workflow.add_node("process", process)
    workflow.set_entry_point("process")
    workflow.add_edge("process", END)
    return workflow.compile()

agent = create_agent()

# API endpoints
@app.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    """Chat endpoint"""
    try:
        result = agent.invoke({
            "messages": [{"role": "user", "content": request.message}],
            "user_id": request.user_id,
            "session_id": request.session_id
        })
        
        return ChatResponse(
            response=result["messages"][-1]["content"],
            session_id=request.session_id,
            metadata={"user_id": request.user_id}
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health():
    """Health check"""
    return {"status": "healthy"}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 7. Docker Deployment

Deploy with Docker:

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Expose port
EXPOSE 8000

# Environment variables (set via docker run or docker-compose)
ENV OPENAI_API_KEY=""
ENV REDIS_URL="redis://redis:6379"

# Run application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  agent:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - REDIS_URL=redis://redis:6379
      - TAVILY_API_KEY=${TAVILY_API_KEY}
    depends_on:
      - redis
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  redis_data:
```

Deploy:

```bash
# Build and run
docker-compose up -d

# View logs
docker-compose logs -f agent

# Scale agents
docker-compose up -d --scale agent=3
```

### 8. Agent with Observability

Add monitoring and observability:

```python
from langsmith import Client
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, END
import os
from datetime import datetime

# Initialize LangSmith
langsmith_client = Client(api_key=os.getenv("LANGSMITH_API_KEY"))

class ObservableAgent:
    def __init__(self, project_name: str):
        self.project_name = project_name
        self.llm = ChatOpenAI(
            model="gpt-4",
            api_key=os.getenv("OPENAI_API_KEY")
        )
    
    def create_workflow(self):
        """Create observable workflow"""
        def process_with_tracing(state: dict):
            # Start trace
            run_id = langsmith_client.create_run(
                name="agent_process",
                run_type="chain",
                inputs={"messages": state["messages"]},
                project_name=self.project_name
            )
            
            try:
                # Process
                response = self.llm.invoke(state["messages"])
                state["messages"].append({
                    "role": "assistant",
                    "content": response.content
                })
                
                # Log success
                langsmith_client.update_run(
                    run_id=run_id.id,
                    outputs={"response": response.content},
                    end_time=datetime.now()
                )
            except Exception as e:
                # Log error
                langsmith_client.update_run(
                    run_id=run_id.id,
                    error=str(e),
                    end_time=datetime.now()
                )
                raise
            
            return state
        
        workflow = StateGraph(dict)
        workflow.add_node("process", process_with_tracing)
        workflow.set_entry_point("process")
        workflow.add_edge("process", END)
        return workflow.compile()

# Usage
agent = ObservableAgent(project_name="production-agent")
app = agent.create_workflow()

result = app.invoke({
    "messages": [{"role": "user", "content": "Hello"}]
})

# View traces at https://smith.langchain.com
```

## Configuration

Key environment variables:

```bash
# LLM API Keys
export OPENAI_API_KEY="your-openai-key"
export ANTHROPIC_API_KEY="your-anthropic-key"

# Vector Stores
export REDIS_URL="redis://localhost:6379"
export PINECONE_API_KEY="your-pinecone-key"

# Web Access
export TAVILY_API_KEY="your-tavily-key"
export BRIGHTDATA_API_KEY="your-brightdata-key"

# RAG Platforms
export CONTEXTUAL_API_KEY="your-contextual-key"

# Observability
export LANGSMITH_API_KEY="your-langsmith-key"
export LANGSMITH_PROJECT="your-project-name"

# Security
export ARCADE_API_KEY="your-arcade-key"

# Deployment
export REDIS_HOST="localhost"
export REDIS_PORT="6379"
export API_PORT="8000"
```

## Common Patterns

### Agent with Tools

```python
from langchain.tools import Tool
from langchain.agents import initialize_agent, AgentType
from langchain_openai import ChatOpenAI
import os

def calculator(expression: str) -> str:
    """Evaluate mathematical expression"""
    try:
        return str(eval(expression))
    except:
        return "Invalid expression"

def web_search(query: str) -> str:
    """Search the web"""
    # Implementation with Tavily
    return f"Results for: {query}"

tools = [
    Tool(
        name="Calculator",
        func=calculator,
        description="Evaluate mathematical expressions"
    ),
    Tool(
        name="WebSearch",
        func=web_search,
        description="Search the web for information"
    )
]

llm = ChatOpenAI(model="gpt-4", api_key=os.getenv("OPENAI_API_KEY"))
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

result = agent.run("What is 25 * 17? Then search for information about AI agents.")
```

### Streaming Responses

```python
from langchain_openai import ChatOpenAI
from langchain.callbacks.streaming_stdout import StreamingStdOutCallbackHandler

llm = ChatOpenAI(
    model="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY"),
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()]
)

# Stream response
for chunk in llm.stream("Tell me about AI agents"):
    print(chunk.content, end="", flush=True)
```

### Error Handling and Retries

```python
from tenacity import retry, stop_after_attempt, wait_exponential
from langchain_openai import ChatOpenAI
import os

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
def call_llm_with_retry(prompt: str):
    """Call LLM with automatic retries"""
    llm = ChatOpenAI(model="gpt-4", api_key=os.getenv("OPENAI_API_KEY"))
    return llm.invoke([{"role": "user", "content": prompt}])

try:
    response = call_llm_with_retry("Hello, how are you?")
    print(response.content)
except Exception as e:
    print(f"Failed after retries: {e}")
```

## Troubleshooting

### Common Issues

**Agent not responding:**
```python
# Check API key is set
import os
assert os.getenv("OPENAI_API_KEY"), "OPENAI_API_KEY not set"

# Enable verbose mode
from langchain.globals import set_verbose
set_verbose(True)
```

**Memory not persisting:**
```python
# Verify Redis connection
import redis
r = redis.from_url(os.getenv("REDIS_URL", "redis://localhost:6379"))
r.ping()  # Should return True

# Check vector store configuration
from langchain_redis import RedisVectorStore
# Ensure index_name is consistent across runs
```

**Rate limiting:**
```python
# Add rate limiting middleware
from langchain.llms import OpenAI
from langchain.callbacks import get_openai_callback

with get_openai_callback() as cb:
    # Your agent code here
    print(f"Total Tokens: {cb.total_tokens}")
    print(f"Total Cost: ${cb.total_cost}")
```

**Docker deployment issues:**
```bash
# Check container logs
docker-compose logs -f agent

# Test health endpoint
curl http://localhost:8000/health

# Verify environment variables
docker-compose exec agent env | grep API_KEY
```

**LangGraph state not updating:**
```python
# Ensure state is properly typed
from typing import TypedDict
from langgraph.graph import StateGraph

class State(TypedDict):
    messages: list  # Must be mutable type

# Return updated state, don't modify in place
def node(state: State):
    return {"messages": state["messages"] + [new_message]}
```

## Additional Resources

- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [LangChain Docs](https://python.langchain.com/docs/get_started/introduction)
- [Repository Tutorials](https://github.com/NirDiamant/agents-towards-production/tree/main/tutorials)
- [Discord Community](https://discord.gg/cA6Aa4uyDX)

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement retry logic** for external API calls
3. **Add observability** from day one (LangSmith, traces)
4. **Use typed state** in LangGraph for better debugging
5. **Implement health checks** in production deployments
6. **Store conversation history** in vector databases for context
7. **Set token limits** to control costs
8. **Test with different LLM models** for cost/performance tradeoffs
9. **Add input validation** to prevent prompt injection
10. **Monitor usage and costs** in production
