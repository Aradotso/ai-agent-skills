---
name: 500-ai-agents-projects-catalog
description: Comprehensive catalog of 500+ AI agent use cases across industries with open-source implementations and framework examples
triggers:
  - show me AI agent use cases for healthcare
  - find AI agent projects for my industry
  - what are examples of CrewAI implementations
  - browse AI agent applications in finance
  - find open source AI agent projects
  - show me langgraph agent examples
  - what AI agents exist for customer service
  - explore autogen framework use cases
---

# 500 AI Agents Projects Catalog

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The 500 AI Agents Projects is a curated collection of AI agent use cases across various industries including healthcare, finance, education, retail, transportation, manufacturing, and more. It provides:

- **Industry-categorized use cases**: AI agents organized by sector (Healthcare, Finance, Education, Customer Service, Retail, etc.)
- **Framework-specific examples**: Use cases organized by AI agent frameworks (CrewAI, AutoGen, Agno, LangGraph)
- **Open-source implementations**: Direct links to working GitHub repositories for each use case
- **Practical applications**: Real-world examples showing how AI agents solve specific problems

## Installation

This is a reference repository, not an installable package. To use it:

```bash
# Clone the repository
git clone https://github.com/ashishpatel26/500-AI-Agents-Projects.git
cd 500-AI-Agents-Projects

# Browse the README for use cases
cat README.md
```

## Repository Structure

The repository is organized into:

1. **Industry Use Case Table**: Main table with 500+ use cases categorized by industry
2. **Framework-Specific Sections**: Use cases organized by framework (CrewAI, AutoGen, Agno, LangGraph)
3. **Industry MindMap**: Visual representation of industries using AI agents

## Finding Use Cases

### By Industry

The main use case table categorizes agents by industry:

- **Healthcare**: Health diagnostics, medical report analysis, disease monitoring
- **Finance**: Trading bots, fraud detection, risk assessment
- **Education**: Virtual tutors, personalized learning, grading automation
- **Customer Service**: 24/7 chatbots, ticket routing, sentiment analysis
- **Retail**: Product recommendations, inventory management, price optimization
- **Transportation**: Route optimization, autonomous delivery, fleet management
- **Manufacturing**: Quality control, predictive maintenance, process monitoring
- **Real Estate**: Property pricing, market analysis, virtual tours
- **Agriculture**: Crop monitoring, yield prediction, pest detection
- **Energy**: Demand forecasting, grid optimization, consumption analysis
- **Entertainment**: Content personalization, recommendation engines
- **Legal**: Document review, contract analysis, compliance checking
- **HR**: Recruitment, candidate matching, employee engagement
- **Hospitality**: Travel planning, booking optimization, guest services
- **Gaming**: Game companions, strategy assistance, player matching
- **Cybersecurity**: Threat detection, vulnerability scanning, incident response
- **E-commerce**: Personal shopping, cart optimization, dynamic pricing
- **Supply Chain**: Logistics optimization, inventory forecasting, route planning

### By Framework

#### CrewAI Examples

CrewAI is a framework for orchestrating role-playing, autonomous AI agents:

```python
# Example: Email Auto Responder (Communication)
# Repository: crewAI-examples/flows/email_auto_responder_flow

from crewai import Agent, Task, Crew

# Define agents
email_classifier = Agent(
    role="Email Classifier",
    goal="Classify incoming emails by priority and category",
    backstory="Expert at email triage and organization"
)

response_writer = Agent(
    role="Response Writer",
    goal="Draft appropriate email responses",
    backstory="Professional communication specialist"
)

# Define tasks
classify_task = Task(
    description="Classify the email: {email_content}",
    agent=email_classifier
)

respond_task = Task(
    description="Write response for classified email",
    agent=response_writer
)

# Create crew
crew = Crew(
    agents=[email_classifier, response_writer],
    tasks=[classify_task, respond_task]
)

# Execute
result = crew.kickoff(inputs={"email_content": "..."})
```

**CrewAI Use Cases in Catalog**:
- Email Auto Responder Flow (Communication)
- Meeting Assistant Flow (Productivity)
- Lead Score Flow (Sales)
- Marketing Strategy Generator (Marketing)
- Job Posting Generator (Recruitment)
- Recruitment Workflow (HR)

#### AutoGen Examples

AutoGen enables development of LLM applications using multiple agents:

```python
# Example: Multi-agent collaboration
# Common pattern in AutoGen projects

import autogen

config_list = autogen.config_list_from_json(
    "OAI_CONFIG_LIST",
    filter_dict={"model": ["gpt-4"]}
)

# Create assistant agent
assistant = autogen.AssistantAgent(
    name="assistant",
    llm_config={"config_list": config_list}
)

# Create user proxy agent
user_proxy = autogen.UserProxyAgent(
    name="user_proxy",
    human_input_mode="NEVER",
    code_execution_config={"work_dir": "coding"}
)

# Initiate conversation
user_proxy.initiate_chat(
    assistant,
    message="Analyze this dataset and provide insights"
)
```

#### LangGraph Examples

LangGraph is used for building stateful, multi-actor applications with LLMs:

```python
# Example: Customer Support Agent
# Repository: GenAI_Agents/customer_support_agent_langgraph.ipynb

from langgraph.graph import StateGraph, END
from langchain_core.messages import HumanMessage

# Define state
class AgentState(TypedDict):
    messages: list[HumanMessage]
    next_step: str

# Define nodes
def classify_query(state):
    """Classify customer query"""
    # Classification logic
    return {"next_step": "technical" if is_technical else "general"}

def technical_support(state):
    """Handle technical queries"""
    # Technical support logic
    return {"messages": state["messages"] + [response]}

def general_support(state):
    """Handle general queries"""
    # General support logic
    return {"messages": state["messages"] + [response]}

# Build graph
workflow = StateGraph(AgentState)
workflow.add_node("classify", classify_query)
workflow.add_node("technical", technical_support)
workflow.add_node("general", general_support)

workflow.set_entry_point("classify")
workflow.add_conditional_edges(
    "classify",
    lambda x: x["next_step"],
    {"technical": "technical", "general": "general"}
)
workflow.add_edge("technical", END)
workflow.add_edge("general", END)

app = workflow.compile()

# Run
result = app.invoke({
    "messages": [HumanMessage(content="My app crashed")],
    "next_step": ""
})
```

## Common Patterns

### Pattern 1: Finding Relevant Use Cases

```python
# Search the catalog programmatically
import requests
import re

def find_use_cases_by_industry(industry: str):
    """Find AI agent use cases for a specific industry"""
    url = "https://raw.githubusercontent.com/ashishpatel26/500-AI-Agents-Projects/main/README.md"
    response = requests.get(url)
    
    # Parse markdown table
    lines = response.text.split('\n')
    use_cases = []
    
    for line in lines:
        if industry.lower() in line.lower() and '|' in line:
            parts = [p.strip() for p in line.split('|')]
            if len(parts) > 4:
                use_cases.append({
                    'name': parts[1],
                    'industry': parts[2],
                    'description': parts[3],
                    'github_link': extract_github_link(parts[4])
                })
    
    return use_cases

def extract_github_link(markdown_link: str) -> str:
    """Extract GitHub URL from markdown link"""
    match = re.search(r'https://github\.com/[^\)]+', markdown_link)
    return match.group(0) if match else None

# Usage
healthcare_agents = find_use_cases_by_industry("Healthcare")
for agent in healthcare_agents:
    print(f"{agent['name']}: {agent['description']}")
    print(f"GitHub: {agent['github_link']}\n")
```

### Pattern 2: Exploring Framework Examples

```python
def get_framework_examples(framework: str):
    """Get examples for a specific framework (CrewAI, AutoGen, LangGraph)"""
    url = "https://raw.githubusercontent.com/ashishpatel26/500-AI-Agents-Projects/main/README.md"
    response = requests.get(url)
    
    # Find framework section
    content = response.text
    framework_section = re.search(
        f'### \\*\\*Framework Name\\*\\*: \\*\\*{framework}\\*\\*(.*?)(?=###|$)',
        content,
        re.DOTALL | re.IGNORECASE
    )
    
    if framework_section:
        section_text = framework_section.group(1)
        # Parse table rows
        examples = []
        for line in section_text.split('\n'):
            if '|' in line and 'Use Case' not in line and '---' not in line:
                parts = [p.strip() for p in line.split('|')]
                if len(parts) > 3:
                    examples.append({
                        'use_case': parts[1],
                        'industry': parts[2],
                        'description': parts[3]
                    })
        return examples
    return []

# Usage
crewai_examples = get_framework_examples("CrewAI")
print(f"Found {len(crewai_examples)} CrewAI examples")
```

### Pattern 3: Building Custom Agent from Catalog

```python
# Example: Implementing a Healthcare AI Agent based on catalog

from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI
from langchain.memory import ConversationBufferMemory
import os

class HealthInsightsAgent:
    """
    Based on: HIA (Health Insights Agent)
    Repository: github.com/harshhh28/hia
    """
    
    def __init__(self):
        self.llm = OpenAI(
            temperature=0,
            api_key=os.getenv("OPENAI_API_KEY")
        )
        self.memory = ConversationBufferMemory(
            memory_key="chat_history",
            return_messages=True
        )
        self.tools = self._create_tools()
        self.agent = initialize_agent(
            self.tools,
            self.llm,
            agent="conversational-react-description",
            memory=self.memory
        )
    
    def _create_tools(self):
        return [
            Tool(
                name="Analyze Medical Report",
                func=self.analyze_report,
                description="Analyzes medical reports and extracts key insights"
            ),
            Tool(
                name="Health Recommendations",
                func=self.get_recommendations,
                description="Provides health recommendations based on analysis"
            )
        ]
    
    def analyze_report(self, report_text: str) -> str:
        """Analyze medical report"""
        # Implementation based on HIA project
        prompt = f"Analyze this medical report and extract key findings:\n{report_text}"
        return self.llm(prompt)
    
    def get_recommendations(self, findings: str) -> str:
        """Generate health recommendations"""
        prompt = f"Based on these findings, provide health recommendations:\n{findings}"
        return self.llm(prompt)
    
    def chat(self, message: str) -> str:
        """Main chat interface"""
        return self.agent.run(message)

# Usage
agent = HealthInsightsAgent()
response = agent.chat("Analyze my recent blood test results")
print(response)
```

### Pattern 4: Multi-Industry Agent System

```python
# Example: Creating a multi-purpose agent that handles different industries

from typing import Dict, List
import json

class IndustryAgentRouter:
    """
    Routes queries to appropriate industry-specific agents
    Based on patterns from the 500 AI Agents catalog
    """
    
    def __init__(self):
        self.industry_keywords = {
            'healthcare': ['medical', 'health', 'diagnosis', 'patient', 'treatment'],
            'finance': ['trading', 'stock', 'investment', 'market', 'portfolio'],
            'education': ['learn', 'study', 'course', 'tutor', 'exam'],
            'retail': ['product', 'shop', 'purchase', 'recommendation', 'inventory'],
            'customer_service': ['support', 'help', 'ticket', 'complaint', 'query']
        }
        self.agents = self._initialize_agents()
    
    def _initialize_agents(self) -> Dict:
        """Initialize industry-specific agents"""
        return {
            'healthcare': HealthcareAgent(),
            'finance': FinanceAgent(),
            'education': EducationAgent(),
            'retail': RetailAgent(),
            'customer_service': CustomerServiceAgent()
        }
    
    def classify_query(self, query: str) -> str:
        """Classify query to determine industry"""
        query_lower = query.lower()
        scores = {}
        
        for industry, keywords in self.industry_keywords.items():
            score = sum(1 for keyword in keywords if keyword in query_lower)
            scores[industry] = score
        
        return max(scores, key=scores.get) if max(scores.values()) > 0 else 'general'
    
    def route_query(self, query: str) -> str:
        """Route query to appropriate agent"""
        industry = self.classify_query(query)
        
        if industry in self.agents:
            return self.agents[industry].process(query)
        else:
            return "I'm not sure which department can help with that. Can you be more specific?"

# Usage
router = IndustryAgentRouter()
response = router.route_query("I need help analyzing my stock portfolio")
```

## Environment Variables

When implementing agents from the catalog, common environment variables needed:

```bash
# LLM API Keys
export OPENAI_API_KEY=your_openai_key
export ANTHROPIC_API_KEY=your_anthropic_key
export GOOGLE_API_KEY=your_google_key

# Framework-specific
export CREWAI_API_KEY=your_crewai_key
export LANGCHAIN_API_KEY=your_langchain_key

# Database (if needed)
export DATABASE_URL=your_database_url

# Vector stores
export PINECONE_API_KEY=your_pinecone_key
export WEAVIATE_URL=your_weaviate_url
```

## Troubleshooting

### Issue: Repository Links Not Working

Some projects may have been moved or archived. Check the original repository:

```python
import requests

def verify_github_link(github_url: str) -> bool:
    """Verify if GitHub repository still exists"""
    response = requests.get(github_url)
    return response.status_code == 200

# Usage
url = "https://github.com/harshhh28/hia"
if verify_github_link(url):
    print("Repository is accessible")
else:
    print("Repository may have moved or been deleted")
```

### Issue: Framework Version Compatibility

Different examples may use different framework versions:

```bash
# Check requirements from a specific example
curl -s https://raw.githubusercontent.com/crewAIInc/crewAI-examples/main/requirements.txt

# Install specific version
pip install crewai==0.1.0  # Use version from example
```

### Issue: Finding Similar Use Cases

If you can't find an exact match, search for similar patterns:

```python
from difflib import SequenceMatcher

def find_similar_use_cases(target_description: str, all_use_cases: List[Dict], threshold=0.6):
    """Find use cases similar to target description"""
    similar = []
    
    for use_case in all_use_cases:
        similarity = SequenceMatcher(
            None,
            target_description.lower(),
            use_case['description'].lower()
        ).ratio()
        
        if similarity >= threshold:
            similar.append({
                'use_case': use_case,
                'similarity': similarity
            })
    
    return sorted(similar, key=lambda x: x['similarity'], reverse=True)
```

## Best Practices

1. **Start with Framework Examples**: Begin with the framework-specific section (CrewAI, AutoGen, LangGraph) to understand implementation patterns

2. **Check Repository Activity**: Before implementing, verify the GitHub repository is actively maintained

3. **Adapt to Your Needs**: Use catalog examples as starting points, not complete solutions

4. **Combine Use Cases**: Many real-world applications combine multiple agent types (e.g., customer service + recommendation)

5. **Environment Configuration**: Always use environment variables for API keys and sensitive configuration

6. **Version Control**: Pin framework versions to match the examples for reproducibility

## Additional Resources

- Main Repository: https://github.com/ashishpatel26/500-AI-Agents-Projects
- CrewAI Examples: https://github.com/crewAIInc/crewAI-examples
- AutoGen Documentation: https://microsoft.github.io/autogen/
- LangGraph Documentation: https://langchain-ai.github.io/langgraph/
