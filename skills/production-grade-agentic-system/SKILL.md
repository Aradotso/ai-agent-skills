---
name: production-grade-agentic-system
description: Build production-ready multi-agent AI systems with security, observability, and scalability using LangGraph and FastAPI
triggers:
  - build a production-ready agentic AI system
  - create multi-agent system with LangGraph
  - implement AI agent with memory and tools
  - setup agent observability with Prometheus
  - configure FastAPI with LangChain agents
  - deploy production AI agents with monitoring
  - implement agent security and rate limiting
  - create agentic system with persistence
---

# Production-Grade Agentic System

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A comprehensive framework for building production-ready multi-agent AI systems with 7 core architectural layers: modular codebase, data persistence, security & safeguards, service layer, multi-agent orchestration, API gateway, and observability. Built with FastAPI, LangGraph, PostgreSQL, and includes monitoring, evaluation, and stress testing.

## Installation

### Prerequisites

- Python ≥3.13
- PostgreSQL database
- Docker and Docker Compose (for containerized deployment)

### Clone and Setup

```bash
git clone https://github.com/FareedKhan-dev/production-grade-agentic-system
cd production-grade-agentic-system

# Install dependencies
pip install -e .

# Install development dependencies
pip install -e .[dev]

# Install testing dependencies
pip install -e .[test]
```

### Environment Configuration

Create a `.env` file in the project root:

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/agentic_db
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-supabase-anon-key

# OpenAI
OPENAI_API_KEY=your-openai-api-key

# Authentication
SECRET_KEY=your-secret-key-min-32-chars
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# LangFuse (Observability)
LANGFUSE_PUBLIC_KEY=your-langfuse-public-key
LANGFUSE_SECRET_KEY=your-langfuse-secret-key
LANGFUSE_HOST=https://cloud.langfuse.com

# Rate Limiting
RATE_LIMIT_PER_MINUTE=60

# Application
APP_ENV=development
LOG_LEVEL=INFO
```

### Docker Deployment

```bash
# Start all services (app, postgres, prometheus, grafana)
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop services
docker-compose down
```

## Project Structure

The system follows a modular architecture with clear separation of concerns:

```
app/
├── api/v1/          # API route handlers
├── core/            # Core application logic
│   ├── langgraph/   # Agent orchestration
│   │   └── tools/   # Agent tools (search, actions)
│   └── prompts/     # System and agent prompts
├── models/          # SQLModel database models
├── schemas/         # Pydantic validation schemas
├── services/        # Business logic layer
└── utils/           # Shared utilities

evals/               # Evaluation framework
├── metrics/         # Evaluation criteria
└── prompts/         # LLM-as-a-Judge prompts

grafana/             # Observability dashboards
prometheus/          # Metrics configuration
```

## Core Components

### 1. Database Models (SQLModel)

```python
from sqlmodel import SQLModel, Field
from datetime import datetime
from typing import Optional

class User(SQLModel, table=True):
    __tablename__ = "users"
    
    id: Optional[int] = Field(default=None, primary_key=True)
    email: str = Field(unique=True, index=True)
    hashed_password: str
    is_active: bool = Field(default=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    
class Conversation(SQLModel, table=True):
    __tablename__ = "conversations"
    
    id: Optional[int] = Field(default=None, primary_key=True)
    user_id: int = Field(foreign_key="users.id")
    thread_id: str = Field(unique=True, index=True)
    title: Optional[str] = None
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
```

### 2. Pydantic Schemas (DTOs)

```python
from pydantic import BaseModel, EmailStr
from typing import Optional

class UserCreate(BaseModel):
    email: EmailStr
    password: str

class UserResponse(BaseModel):
    id: int
    email: str
    is_active: bool
    
    model_config = {"from_attributes": True}

class ChatRequest(BaseModel):
    message: str
    thread_id: Optional[str] = None

class ChatResponse(BaseModel):
    response: str
    thread_id: str
```

### 3. Security & Authentication

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
import os

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

SECRET_KEY = os.getenv("SECRET_KEY")
ALGORITHM = os.getenv("ALGORITHM", "HS256")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: int = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401, detail="Invalid token")
        return user_id
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

### 4. Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address
from fastapi import Request

limiter = Limiter(key_func=get_remote_address)

@app.post("/api/v1/chat")
@limiter.limit("60/minute")
async def chat_endpoint(
    request: Request,
    chat_request: ChatRequest,
    user_id: int = Depends(get_current_user)
):
    # Handle chat request
    pass
```

### 5. LangGraph Agent with Tools

```python
from langgraph.graph import StateGraph, END
from langchain_core.messages import HumanMessage, AIMessage
from langchain_openai import ChatOpenAI
from typing import TypedDict, Annotated, Sequence
import operator

class AgentState(TypedDict):
    messages: Annotated[Sequence[HumanMessage | AIMessage], operator.add]
    next: str

# Define agent tools
from langchain_community.tools import DuckDuckGoSearchRun

search_tool = DuckDuckGoSearchRun()

tools = [search_tool]

# Create LLM with tools
llm = ChatOpenAI(model="gpt-4", temperature=0)
llm_with_tools = llm.bind_tools(tools)

# Define agent node
def agent_node(state: AgentState):
    messages = state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}

# Define tool execution node
def tool_node(state: AgentState):
    messages = state["messages"]
    last_message = messages[-1]
    
    # Execute tool calls
    tool_outputs = []
    for tool_call in last_message.tool_calls:
        tool_result = search_tool.run(tool_call["args"])
        tool_outputs.append(AIMessage(content=tool_result))
    
    return {"messages": tool_outputs}

# Build graph
workflow = StateGraph(AgentState)
workflow.add_node("agent", agent_node)
workflow.add_node("tools", tool_node)

# Define routing logic
def should_continue(state: AgentState):
    last_message = state["messages"][-1]
    if hasattr(last_message, "tool_calls") and last_message.tool_calls:
        return "tools"
    return END

workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", should_continue, {"tools": "tools", END: END})
workflow.add_edge("tools", "agent")

agent = workflow.compile()
```

### 6. Memory Management with Checkpointing

```python
from langgraph.checkpoint.postgres import PostgresSaver
from psycopg2 import pool

# Create connection pool
connection_pool = pool.SimpleConnectionPool(
    1, 20,
    dsn=os.getenv("DATABASE_URL")
)

# Create checkpointer
checkpointer = PostgresSaver(connection_pool)

# Compile agent with memory
agent_with_memory = workflow.compile(checkpointer=checkpointer)

# Use agent with thread ID for conversation persistence
async def chat_with_memory(message: str, thread_id: str):
    config = {"configurable": {"thread_id": thread_id}}
    
    result = await agent_with_memory.ainvoke(
        {"messages": [HumanMessage(content=message)]},
        config=config
    )
    
    return result["messages"][-1].content
```

### 7. FastAPI Application with Streaming

```python
from fastapi import FastAPI, Depends
from fastapi.responses import StreamingResponse
from contextlib import asynccontextmanager
import json

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    print("Starting application...")
    yield
    # Shutdown
    print("Shutting down application...")

app = FastAPI(lifespan=lifespan)

@app.post("/api/v1/chat/stream")
async def chat_stream(
    chat_request: ChatRequest,
    user_id: int = Depends(get_current_user)
):
    async def event_generator():
        config = {
            "configurable": {
                "thread_id": chat_request.thread_id or f"user-{user_id}"
            }
        }
        
        async for event in agent_with_memory.astream_events(
            {"messages": [HumanMessage(content=chat_request.message)]},
            config=config,
            version="v1"
        ):
            if event["event"] == "on_chat_model_stream":
                chunk = event["data"]["chunk"]
                if chunk.content:
                    yield f"data: {json.dumps({'content': chunk.content})}\n\n"
    
    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream"
    )
```

### 8. Observability with LangFuse

```python
from langfuse.callback import CallbackHandler
import os

langfuse_handler = CallbackHandler(
    public_key=os.getenv("LANGFUSE_PUBLIC_KEY"),
    secret_key=os.getenv("LANGFUSE_SECRET_KEY"),
    host=os.getenv("LANGFUSE_HOST")
)

# Use with LangChain
async def chat_with_tracing(message: str, thread_id: str):
    config = {
        "configurable": {"thread_id": thread_id},
        "callbacks": [langfuse_handler]
    }
    
    result = await agent_with_memory.ainvoke(
        {"messages": [HumanMessage(content=message)]},
        config=config
    )
    
    return result
```

### 9. Prometheus Metrics

```python
from prometheus_client import Counter, Histogram
from starlette_prometheus import metrics, PrometheusMiddleware

app.add_middleware(PrometheusMiddleware)
app.add_route("/metrics", metrics)

# Custom metrics
chat_requests = Counter(
    "chat_requests_total",
    "Total number of chat requests",
    ["user_id", "status"]
)

chat_latency = Histogram(
    "chat_latency_seconds",
    "Chat request latency in seconds"
)

@app.post("/api/v1/chat")
async def chat(
    chat_request: ChatRequest,
    user_id: int = Depends(get_current_user)
):
    with chat_latency.time():
        try:
            response = await chat_with_memory(
                chat_request.message,
                chat_request.thread_id or f"user-{user_id}"
            )
            chat_requests.labels(user_id=user_id, status="success").inc()
            return {"response": response}
        except Exception as e:
            chat_requests.labels(user_id=user_id, status="error").inc()
            raise
```

### 10. Circuit Breaker Pattern

```python
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type
)
from typing import Optional

class CircuitBreaker:
    def __init__(self, failure_threshold: int = 5, timeout: int = 60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.last_failure_time: Optional[float] = None
        self.state = "CLOSED"  # CLOSED, OPEN, HALF_OPEN
    
    async def call(self, func, *args, **kwargs):
        if self.state == "OPEN":
            if time.time() - self.last_failure_time > self.timeout:
                self.state = "HALF_OPEN"
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = await func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
    
    def on_success(self):
        self.failure_count = 0
        self.state = "CLOSED"
    
    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = "OPEN"

# Usage
llm_circuit_breaker = CircuitBreaker(failure_threshold=3, timeout=30)

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10),
    retry=retry_if_exception_type(Exception)
)
async def call_llm_with_retry(message: str):
    return await llm_circuit_breaker.call(llm.ainvoke, message)
```

## Evaluation Framework

### LLM-as-a-Judge

```python
from langchain_openai import ChatOpenAI
from pydantic import BaseModel

class EvaluationResult(BaseModel):
    score: float
    reasoning: str

eval_llm = ChatOpenAI(model="gpt-4", temperature=0)

EVAL_PROMPT = """
Evaluate the following AI agent response based on these criteria:
- Accuracy: Is the response factually correct?
- Relevance: Does it address the user's question?
- Completeness: Does it provide a thorough answer?
- Safety: Is the response safe and appropriate?

User Query: {query}
Agent Response: {response}

Provide a score from 0-10 and explain your reasoning.
"""

async def evaluate_response(query: str, response: str) -> EvaluationResult:
    prompt = EVAL_PROMPT.format(query=query, response=response)
    result = await eval_llm.ainvoke(prompt)
    
    # Parse LLM output into structured result
    # Implementation depends on LLM output format
    return EvaluationResult(score=8.5, reasoning=result.content)
```

## Testing

### Unit Tests

```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_chat_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as client:
        # Login
        response = await client.post(
            "/api/v1/auth/login",
            json={"email": "test@example.com", "password": "testpass"}
        )
        token = response.json()["access_token"]
        
        # Chat
        response = await client.post(
            "/api/v1/chat",
            json={"message": "Hello, agent!"},
            headers={"Authorization": f"Bearer {token}"}
        )
        
        assert response.status_code == 200
        assert "response" in response.json()
```

### Load Testing

```python
import asyncio
from httpx import AsyncClient
import time

async def simulate_user(client: AsyncClient, token: str, num_requests: int):
    for i in range(num_requests):
        await client.post(
            "/api/v1/chat",
            json={"message": f"Test message {i}"},
            headers={"Authorization": f"Bearer {token}"}
        )

async def load_test(num_users: int, requests_per_user: int):
    start_time = time.time()
    
    async with AsyncClient(base_url="http://localhost:8000") as client:
        # Create tasks for concurrent users
        tasks = [
            simulate_user(client, "test_token", requests_per_user)
            for _ in range(num_users)
        ]
        await asyncio.gather(*tasks)
    
    duration = time.time() - start_time
    total_requests = num_users * requests_per_user
    print(f"Completed {total_requests} requests in {duration:.2f}s")
    print(f"Throughput: {total_requests/duration:.2f} req/s")

# Run load test
asyncio.run(load_test(num_users=100, requests_per_user=10))
```

## Common Patterns

### Context Management

```python
from contextvars import ContextVar
from typing import Optional

request_id_var: ContextVar[Optional[str]] = ContextVar("request_id", default=None)

@app.middleware("http")
async def add_request_id(request: Request, call_next):
    request_id = str(uuid.uuid4())
    request_id_var.set(request_id)
    
    response = await call_next(request)
    response.headers["X-Request-ID"] = request_id
    return response
```

### Structured Logging

```python
import structlog

logger = structlog.get_logger()

async def process_chat(message: str, user_id: int):
    logger.info(
        "chat_request_received",
        user_id=user_id,
        message_length=len(message)
    )
    
    try:
        response = await chat_with_memory(message, f"user-{user_id}")
        logger.info("chat_response_generated", user_id=user_id)
        return response
    except Exception as e:
        logger.error(
            "chat_processing_failed",
            user_id=user_id,
            error=str(e),
            exc_info=True
        )
        raise
```

## Troubleshooting

### Database Connection Issues

```python
# Check connection pool health
from sqlmodel import Session, create_engine

engine = create_engine(os.getenv("DATABASE_URL"), pool_pre_ping=True)

def check_db_health():
    try:
        with Session(engine) as session:
            session.exec("SELECT 1")
        return True
    except Exception as e:
        logger.error("database_health_check_failed", error=str(e))
        return False
```

### LLM Timeout Handling

```python
import asyncio

async def call_llm_with_timeout(message: str, timeout: int = 30):
    try:
        return await asyncio.wait_for(
            llm.ainvoke(message),
            timeout=timeout
        )
    except asyncio.TimeoutError:
        logger.error("llm_call_timeout", timeout=timeout)
        raise HTTPException(status_code=504, detail="LLM request timeout")
```

### Memory Leak Prevention

```python
# Clear old conversations periodically
from datetime import datetime, timedelta

async def cleanup_old_conversations():
    cutoff_date = datetime.utcnow() - timedelta(days=30)
    
    with Session(engine) as session:
        old_conversations = session.exec(
            select(Conversation).where(Conversation.updated_at < cutoff_date)
        )
        
        for conv in old_conversations:
            session.delete(conv)
        
        session.commit()
```

### Rate Limit Debugging

```python
from slowapi.errors import RateLimitExceeded

@app.exception_handler(RateLimitExceeded)
async def rate_limit_handler(request: Request, exc: RateLimitExceeded):
    logger.warning(
        "rate_limit_exceeded",
        client_ip=get_remote_address(request),
        path=request.url.path
    )
    return JSONResponse(
        status_code=429,
        content={"detail": "Rate limit exceeded. Please try again later."}
    )
```

## Running the Application

### Development Mode

```bash
# Run with auto-reload
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Run with specific log level
uvicorn app.main:app --log-level debug
```

### Production Mode

```bash
# Run with uvloop and multiple workers
uvicorn app.main:app \
  --host 0.0.0.0 \
  --port 8000 \
  --workers 4 \
  --loop uvloop \
  --log-config logging.yaml
```

### Accessing Services

- **API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000 (admin/admin)
- **Metrics**: http://localhost:8000/metrics

## Additional Resources

- [Medium Article](https://medium.com/@fareedkhandev/building-a-critical-infrastructure-for-production-ready-ai-agentic-system-building-a-production-37ee5d941f1c): Detailed architecture explanation
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/)
