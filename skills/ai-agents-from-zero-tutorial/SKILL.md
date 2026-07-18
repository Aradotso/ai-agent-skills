---
name: ai-agents-from-zero-tutorial
description: A comprehensive Chinese tutorial system for building AI Agents from zero to enterprise-level deployment, covering LangChain, LangGraph, Coze, Dify, RAG, and MCP.
triggers:
  - how do I learn AI agent development from scratch
  - show me the ai agents from zero tutorial structure
  - help me build an enterprise AI agent with LangChain
  - how to implement RAG in Chinese with this tutorial
  - guide me through the shopkeeper agent project
  - what are the AI agent interview questions covered
  - show me DeepAgents multi-agent implementation
  - how to deploy Dify locally following this guide
---

# AI Agents From Zero Tutorial Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

`ai-agents-from-zero` is a comprehensive, open-source Chinese tutorial system for building AI Agents from zero to enterprise-level deployment. It provides systematic learning paths covering LLM fundamentals, prompt engineering, low-code platforms (Coze/Dify), frameworks (LangChain/LangGraph/DeepAgents), RAG, MCP, and enterprise deployment with working code examples and real-world projects.

**Key Features:**
- Complete learning path from basics to enterprise deployment
- Two full production projects with source code
- Interview question bank aligned with AI Engineer job requirements
- Python-focused with LangChain/LangGraph as primary frameworks
- Covers low-code platforms (Coze, Dify) and enterprise RAG systems
- Continuous updates with latest AI tech stack

**Official Resources:**
- GitHub: https://github.com/didilili/ai-agents-from-zero
- Documentation: https://didilili.github.io/ai-agents-from-zero/
- Project 1 (Shopkeeper): https://github.com/didilili/shopkeeper-agent
- Project 2 (DeepSearch): https://github.com/didilili/deepsearch-agents

## Installation & Setup

### Access the Tutorial

```bash
# Clone the repository
git clone https://github.com/didilili/ai-agents-from-zero.git
cd ai-agents-from-zero

# Access online documentation
# Visit: https://didilili.github.io/ai-agents-from-zero/
```

### Project Dependencies

For the tutorial examples and projects, you'll need:

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install common dependencies
pip install langchain langchain-community langchain-openai
pip install langgraph
pip install chromadb faiss-cpu
pip install openai anthropic
pip install python-dotenv
pip install streamlit  # For UI demos
```

### Environment Configuration

```bash
# Create .env file
cat > .env << EOF
# LLM API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key
ZHIPUAI_API_KEY=your_zhipu_key

# Vector Database
CHROMA_PERSIST_DIRECTORY=./chroma_db

# Model Settings
DEFAULT_MODEL=gpt-4
TEMPERATURE=0.7
EOF
```

## Tutorial Structure

### 1. LLM Fundamentals (大模型基础)

**Topics Covered:**
- LLM architecture (Transformer, MoE, Self-Attention)
- Model deployment (Ollama, Xinference, vLLM)
- Prompt engineering principles
- Multi-turn conversations and memory

**Example: Basic LLM Call**

```python
from langchain_openai import ChatOpenAI
from langchain.schema import HumanMessage, SystemMessage
import os
from dotenv import load_dotenv

load_dotenv()

# Initialize LLM
llm = ChatOpenAI(
    model="gpt-4",
    temperature=0.7,
    api_key=os.getenv("OPENAI_API_KEY")
)

# Create messages
messages = [
    SystemMessage(content="你是一个专业的AI助手，擅长回答技术问题。"),
    HumanMessage(content="什么是AI Agent？")
]

# Get response
response = llm.invoke(messages)
print(response.content)
```

**Example: Prompt Engineering with Few-Shot**

```python
from langchain.prompts import FewShotPromptTemplate, PromptTemplate

# Define examples
examples = [
    {
        "input": "如何实现RAG系统？",
        "output": "RAG系统需要：1) 文档分割 2) 向量化存储 3) 相似度检索 4) 上下文增强生成"
    },
    {
        "input": "什么是LangGraph？",
        "output": "LangGraph是一个图式工作流框架，用于构建复杂的AI Agent应用。"
    }
]

# Create example template
example_template = PromptTemplate(
    input_variables=["input", "output"],
    template="问题：{input}\n回答：{output}"
)

# Create few-shot template
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_template,
    prefix="以下是一些技术问答示例：",
    suffix="问题：{input}\n回答：",
    input_variables=["input"]
)

# Use the prompt
prompt = few_shot_prompt.format(input="什么是MCP协议？")
response = llm.invoke(prompt)
print(response.content)
```

### 2. Low-Code Platforms (低代码平台)

#### Coze (扣子) Platform

**Key Features:**
- Workflow builder
- Plugin system
- Knowledge base integration
- Agent orchestration

**Example: Call Coze Workflow via Python**

```python
import requests
import os

def call_coze_workflow(workflow_id, input_data):
    """Call Coze workflow API"""
    url = f"https://api.coze.com/v1/workflow/{workflow_id}/run"
    
    headers = {
        "Authorization": f"Bearer {os.getenv('COZE_API_KEY')}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "input": input_data,
        "stream": False
    }
    
    response = requests.post(url, json=payload, headers=headers)
    return response.json()

# Example usage
result = call_coze_workflow(
    workflow_id="your_workflow_id",
    input_data={"query": "分析这个商品评论", "content": "商品质量很好"}
)
print(result)
```

#### Dify Platform

**Example: Dify Local Deployment with Docker**

```bash
# Clone Dify repository
git clone https://github.com/langgenius/dify.git
cd dify/docker

# Start services
docker-compose up -d

# Access Dify at http://localhost:3000
```

**Example: Call Dify API**

```python
import requests
import os

class DifyClient:
    def __init__(self, api_key, base_url="https://api.dify.ai/v1"):
        self.api_key = api_key
        self.base_url = base_url
    
    def chat_completion(self, query, conversation_id=None):
        """Send chat message to Dify"""
        url = f"{self.base_url}/chat-messages"
        
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        }
        
        payload = {
            "query": query,
            "inputs": {},
            "response_mode": "blocking",
            "user": "user-123"
        }
        
        if conversation_id:
            payload["conversation_id"] = conversation_id
        
        response = requests.post(url, json=payload, headers=headers)
        return response.json()

# Usage
client = DifyClient(api_key=os.getenv("DIFY_API_KEY"))
response = client.chat_completion("如何优化RAG系统？")
print(response["answer"])
```

### 3. LangChain & LangGraph Framework

#### Basic LangChain Patterns

**Example: Chain with LCEL (LangChain Expression Language)**

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# Create components
llm = ChatOpenAI(model="gpt-4")
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个{role}，请用专业的方式回答问题。"),
    ("user", "{question}")
])
output_parser = StrOutputParser()

# Build chain using LCEL
chain = (
    {"role": RunnablePassthrough(), "question": RunnablePassthrough()}
    | prompt
    | llm
    | output_parser
)

# Invoke chain
result = chain.invoke({
    "role": "AI工程师",
    "question": "如何设计一个电商智能客服系统？"
})
print(result)
```

**Example: Memory with ConversationBufferMemory**

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

# Initialize components
llm = ChatOpenAI(model="gpt-4")
memory = ConversationBufferMemory()

# Create conversation chain
conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=True
)

# Multi-turn conversation
response1 = conversation.predict(input="我想构建一个RAG系统")
print(response1)

response2 = conversation.predict(input="需要哪些组件？")
print(response2)

# Check memory
print("\n对话历史：")
print(memory.load_memory_variables({}))
```

#### LangGraph State Machine

**Example: Simple Agent with LangGraph**

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
from langchain_openai import ChatOpenAI
import operator

# Define state
class AgentState(TypedDict):
    messages: Annotated[list, operator.add]
    current_step: str

# Define nodes
def analyze_intent(state: AgentState):
    """Analyze user intent"""
    llm = ChatOpenAI(model="gpt-4")
    prompt = f"分析用户意图：{state['messages'][-1]}"
    response = llm.invoke(prompt)
    
    return {
        "messages": [response.content],
        "current_step": "intent_analyzed"
    }

def generate_response(state: AgentState):
    """Generate final response"""
    llm = ChatOpenAI(model="gpt-4")
    prompt = f"基于意图生成回复：{state['messages'][-1]}"
    response = llm.invoke(prompt)
    
    return {
        "messages": [response.content],
        "current_step": "completed"
    }

# Build graph
workflow = StateGraph(AgentState)

# Add nodes
workflow.add_node("analyze", analyze_intent)
workflow.add_node("respond", generate_response)

# Add edges
workflow.set_entry_point("analyze")
workflow.add_edge("analyze", "respond")
workflow.add_edge("respond", END)

# Compile
app = workflow.compile()

# Run
result = app.invoke({
    "messages": ["我想查询订单状态"],
    "current_step": "start"
})
print(result)
```

### 4. RAG System Implementation

**Example: Complete RAG Pipeline**

```python
from langchain_community.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain.chains import RetrievalQA
from langchain.prompts import PromptTemplate

# 1. Load documents
loader = TextLoader("./data/knowledge_base.txt", encoding="utf-8")
documents = loader.load()

# 2. Split documents
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", "。", "！", "？", "；", "，", " "]
)
texts = text_splitter.split_documents(documents)

# 3. Create embeddings and vector store
embeddings = OpenAIEmbeddings(api_key=os.getenv("OPENAI_API_KEY"))
vectorstore = FAISS.from_documents(texts, embeddings)

# Save vector store
vectorstore.save_local("./faiss_index")

# 4. Create retriever
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# 5. Create QA chain with custom prompt
template = """使用以下上下文回答问题。如果不知道答案，请说"我不知道"。

上下文：
{context}

问题：{question}

详细回答："""

prompt = PromptTemplate(
    template=template,
    input_variables=["context", "question"]
)

llm = ChatOpenAI(model="gpt-4", temperature=0)

qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    chain_type_kwargs={"prompt": prompt},
    return_source_documents=True
)

# 6. Query
query = "如何优化RAG系统的检索效果？"
result = qa_chain.invoke({"query": query})

print("回答：", result["result"])
print("\n来源文档：")
for doc in result["source_documents"]:
    print(f"- {doc.page_content[:100]}...")
```

**Example: Hybrid Search with Reranking**

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever
from langchain.schema import Document

# Prepare documents
docs = [
    Document(page_content="RAG系统需要向量检索和重排序"),
    Document(page_content="混合检索结合了稀疏和密集检索"),
    # ... more documents
]

# 1. Dense retriever (FAISS)
dense_retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# 2. Sparse retriever (BM25)
bm25_retriever = BM25Retriever.from_documents(docs)
bm25_retriever.k = 5

# 3. Ensemble retriever
ensemble_retriever = EnsembleRetriever(
    retrievers=[dense_retriever, bm25_retriever],
    weights=[0.5, 0.5]
)

# 4. Retrieve with hybrid search
results = ensemble_retriever.get_relevant_documents("RAG混合检索")

for doc in results:
    print(f"- {doc.page_content}")
```

### 5. Tool Calling & MCP Protocol

**Example: Function Calling with Tools**

```python
from langchain.tools import tool
from langchain_openai import ChatOpenAI
from langchain.agents import create_openai_tools_agent, AgentExecutor
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder

# Define tools
@tool
def search_product(query: str) -> str:
    """搜索商品信息"""
    # Simulate database search
    return f"找到商品：{query} - 价格：99元"

@tool
def check_inventory(product_id: str) -> str:
    """查询库存"""
    return f"商品 {product_id} 库存：50件"

@tool
def calculate_discount(price: float, discount_rate: float) -> str:
    """计算折扣价格"""
    final_price = price * (1 - discount_rate)
    return f"折后价格：{final_price}元"

# Create agent
tools = [search_product, check_inventory, calculate_discount]

llm = ChatOpenAI(model="gpt-4", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个电商助手，可以帮助用户查询商品和库存。"),
    ("user", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])

agent = create_openai_tools_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# Execute
result = agent_executor.invoke({
    "input": "帮我查询iPhone 15的价格和库存，并计算8折后的价格"
})
print(result["output"])
```

**Example: MCP Server Implementation**

```python
from typing import Any
import json

class MCPServer:
    """Simple MCP Protocol Server"""
    
    def __init__(self):
        self.tools = {}
    
    def register_tool(self, name: str, func: callable, description: str):
        """Register a tool"""
        self.tools[name] = {
            "function": func,
            "description": description
        }
    
    def list_tools(self) -> list:
        """List all available tools"""
        return [
            {"name": name, "description": tool["description"]}
            for name, tool in self.tools.items()
        ]
    
    def call_tool(self, name: str, arguments: dict) -> Any:
        """Call a tool with arguments"""
        if name not in self.tools:
            raise ValueError(f"Tool {name} not found")
        
        return self.tools[name]["function"](**arguments)

# Usage
server = MCPServer()

# Register tools
server.register_tool(
    "get_weather",
    lambda city: f"{city}的天气：晴天，25°C",
    "获取指定城市的天气信息"
)

server.register_tool(
    "search_docs",
    lambda query: f"搜索结果：{query}相关文档",
    "在知识库中搜索文档"
)

# List tools
print("可用工具：", server.list_tools())

# Call tool
result = server.call_tool("get_weather", {"city": "北京"})
print(result)
```

## Real-World Projects

### Project 1: Shopkeeper Agent (电商问数)

**Purpose:** NL2SQL system for e-commerce data queries using LangGraph

**Key Components:**
- Intent recognition
- SQL generation from natural language
- Multi-round dialogue
- Data visualization

**Example: Intent Classification**

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from enum import Enum

class IntentType(Enum):
    ORDER_QUERY = "订单查询"
    SALES_ANALYSIS = "销售分析"
    PRODUCT_INFO = "商品信息"
    GENERAL = "通用问答"

def classify_intent(user_query: str) -> IntentType:
    """Classify user intent"""
    llm = ChatOpenAI(model="gpt-4", temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """你是一个意图分类专家。将用户问题分类为以下类别之一：
        - 订单查询：关于订单状态、物流信息
        - 销售分析：关于销量、销售额统计
        - 商品信息：关于商品详情、库存
        - 通用问答：其他类型问题
        
        只返回类别名称。"""),
        ("user", "{query}")
    ])
    
    chain = prompt | llm
    result = chain.invoke({"query": user_query})
    
    # Map to enum
    intent_map = {
        "订单查询": IntentType.ORDER_QUERY,
        "销售分析": IntentType.SALES_ANALYSIS,
        "商品信息": IntentType.PRODUCT_INFO,
        "通用问答": IntentType.GENERAL
    }
    
    return intent_map.get(result.content, IntentType.GENERAL)

# Test
intent = classify_intent("最近一周的销售额是多少？")
print(f"意图分类：{intent.value}")
```

**Example: NL2SQL Generation**

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate

def generate_sql(question: str, schema: dict) -> str:
    """Generate SQL from natural language"""
    
    # Build schema description
    schema_desc = "\n".join([
        f"表 {table}: {', '.join(columns)}"
        for table, columns in schema.items()
    ])
    
    prompt = PromptTemplate(
        template="""你是一个SQL专家。根据以下数据库schema和用户问题，生成SQL查询。

数据库Schema：
{schema}

用户问题：{question}

只返回SQL语句，不要有其他解释。
SQL：""",
        input_variables=["schema", "question"]
    )
    
    llm = ChatOpenAI(model="gpt-4", temperature=0)
    chain = prompt | llm
    
    result = chain.invoke({
        "schema": schema_desc,
        "question": question
    })
    
    return result.content.strip()

# Usage
schema = {
    "orders": ["order_id", "customer_id", "total_amount", "order_date"],
    "products": ["product_id", "name", "price", "stock"],
    "customers": ["customer_id", "name", "email"]
}

sql = generate_sql("查询本月销售额超过10000的订单", schema)
print(f"生成的SQL：\n{sql}")
```

### Project 2: DeepSearch Agents (深度研搜)

**Purpose:** Multi-agent research system using DeepAgents framework

**Key Components:**
- Search agent
- Analysis agent
- Summary agent
- Report generation

**Example: Multi-Agent Coordination**

```python
from typing import List, Dict
from dataclasses import dataclass

@dataclass
class AgentMessage:
    from_agent: str
    to_agent: str
    content: str
    message_type: str

class SearchAgent:
    def __init__(self, llm):
        self.llm = llm
        self.name = "SearchAgent"
    
    def search(self, query: str) -> List[str]:
        """Search for information"""
        # Simulate search results
        return [
            f"关于 {query} 的结果1：...",
            f"关于 {query} 的结果2：...",
            f"关于 {query} 的结果3：..."
        ]

class AnalysisAgent:
    def __init__(self, llm):
        self.llm = llm
        self.name = "AnalysisAgent"
    
    def analyze(self, content: List[str]) -> Dict:
        """Analyze search results"""
        prompt = f"分析以下内容并提取关键信息：\n{'\n'.join(content)}"
        response = self.llm.invoke(prompt)
        
        return {
            "summary": response.content,
            "key_points": ["要点1", "要点2", "要点3"]
        }

class SummaryAgent:
    def __init__(self, llm):
        self.llm = llm
        self.name = "SummaryAgent"
    
    def summarize(self, analysis: Dict) -> str:
        """Generate final summary"""
        prompt = f"""基于以下分析生成研究报告：
        
摘要：{analysis['summary']}
关键点：{', '.join(analysis['key_points'])}

请生成一份完整的研究报告。"""
        
        response = self.llm.invoke(prompt)
        return response.content

class MultiAgentOrchestrator:
    def __init__(self, llm):
        self.search_agent = SearchAgent(llm)
        self.analysis_agent = AnalysisAgent(llm)
        self.summary_agent = SummaryAgent(llm)
    
    def research(self, topic: str) -> str:
        """Coordinate agents to perform research"""
        print(f"[Orchestrator] 开始研究主题：{topic}")
        
        # Step 1: Search
        print(f"[{self.search_agent.name}] 搜索信息...")
        search_results = self.search_agent.search(topic)
        
        # Step 2: Analyze
        print(f"[{self.analysis_agent.name}] 分析结果...")
        analysis = self.analysis_agent.analyze(search_results)
        
        # Step 3: Summarize
        print(f"[{self.summary_agent.name}] 生成报告...")
        report = self.summary_agent.summarize(analysis)
        
        return report

# Usage
llm = ChatOpenAI(model="gpt-4")
orchestrator = MultiAgentOrchestrator(llm)

report = orchestrator.research("AI Agent 最新发展趋势")
print("\n最终报告：")
print(report)
```

## Enterprise Deployment

### Docker Deployment

**Example: Dockerfile for Agent Application**

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Environment variables
ENV PYTHONUNBUFFERED=1
ENV OPENAI_API_KEY=${OPENAI_API_KEY}

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Example: docker-compose.yml**

```yaml
version: '3.8'

services:
  agent-app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - CHROMA_HOST=chromadb
      - CHROMA_PORT=8001
    depends_on:
      - chromadb
    volumes:
      - ./data:/app/data

  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8001:8000"
    volumes:
      - chroma-data:/chroma/chroma

volumes:
  chroma-data:
```

**Deploy:**

```bash
# Build and run
docker-compose up -d

# Check logs
docker-compose logs -f agent-app

# Stop services
docker-compose down
```

### Monitoring & Observability

**Example: LangSmith Integration**

```python
import os
from langsmith import Client
from langchain.callbacks.manager import collect_runs

# Initialize LangSmith
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = os.getenv("LANGSMITH_API_KEY")
os.environ["LANGCHAIN_PROJECT"] = "ai-agent-project"

# Use in chain
with collect_runs() as cb:
    result = qa_chain.invoke({"query": "测试问题"})
    run_id = cb.traced_runs[0].id
    
print(f"Run ID: {run_id}")
print(f"View trace: https://smith.langchain.com/run/{run_id}")
```

## Common Patterns & Best Practices

### 1. Error Handling in Chains

```python
from langchain.schema import BaseMessage
from typing import Union

def safe_chain_invoke(chain, input_data: dict, max_retries: int = 3) -> Union[str, None]:
    """Safely invoke chain with retry logic"""
    for attempt in range(max_retries):
        try:
            result = chain.invoke(input_data)
            return result
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {str(e)}")
            if attempt == max_retries - 1:
                print("Max retries reached. Returning None.")
                return None
            time.sleep(2 ** attempt)  # Exponential backoff

# Usage
result = safe_chain_invoke(qa_chain, {"query": "测试问题"})
```

### 2. Streaming Responses

```python
from langchain.callbacks.streaming_stdout import StreamingStdOutCallbackHandler
from langchain_openai import ChatOpenAI

# Create streaming LLM
llm = ChatOpenAI(
    model="gpt-4",
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()]
)

# Stream response
for chunk in llm.stream("讲解一下AI Agent的核心概念"):
    print(chunk.content, end="", flush=True)
```

### 3. Batch Processing

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# Batch process multiple queries
queries = [
    "什么是RAG？",
    "如何优化LLM性能？",
    "LangGraph的优势是什么？"
]

results = llm.batch(queries)

for query, result in zip(queries, results):
    print(f"Q: {query}")
    print(f"A: {result.content}\n")
```

### 4. Caching for Performance

```python
from langchain.cache import InMemoryCache
from langchain.globals import set_llm_cache
from langchain_openai import ChatOpenAI

# Enable caching
set_llm_cache(InMemoryCache())

llm = ChatOpenAI(model="gpt-4")

# First call (hits API)
result1 = llm.invoke("什么是AI Agent？")

# Second call (from cache, much faster)
result2 = llm.invoke("什么是AI Agent？")
```

## Troubleshooting

### Common Issues

**1. API Key Errors**

```python
# Always check environment variables
import os
from dotenv import load_dotenv

load_dotenv()

required_keys = ["OPENAI_API_KEY", "ANTHROPIC_API_KEY"]
for key in required_keys:
    if not os.getenv(key):
        print(f"Warning: {key} not found in environment")
```

**2. Chinese Text Encoding**

```python
# Ensure UTF-8 encoding
from langchain_community.document_loaders import TextLoader

loader = TextLoader("./data/chinese.txt", encoding="utf-8")
documents = loader.load()
```

**3. Vector Store Persistence**
