---
name: agentic-soc-platform
description: Build AI-driven security operations automation with ASP's agent-centric SIRP, modules, and playbooks
triggers:
  - set up an agentic SOC platform
  - create security automation workflows with ASP
  - integrate AI agents for security operations
  - build SIRP modules and playbooks
  - configure ASP for security alert processing
  - develop custom security automation modules
  - implement AI-driven threat detection
  - automate security incident response
---

# Agentic SOC Platform Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection

## Overview

Agentic SOC Platform (ASP) is an open-source, AI-driven security operations automation platform that combines SIEM integration, AI agents (Langgraph/Dify), and a built-in Security Incident Response Platform (SIRP). It processes security alerts through Redis streams, enriches them with AI analysis, and enables automated response workflows.

**Key Components:**
- **Modules**: Streaming processors that consume alerts from Redis streams and perform AI-driven analysis
- **Playbooks**: Event-driven automation tasks triggered manually from the SIRP UI
- **SIRP Platform**: Built on Nocoly for case management, alerts, and artifacts
- **AI Agents**: Support for Langgraph, Dify, and local LLMs

## Installation

### Docker Deployment (Recommended)

```bash
# Clone repository
git clone https://github.com/FunnyWolf/agentic-soc-platform.git
cd agentic-soc-platform

# Start with Docker Compose
cd Docker
docker-compose up -d

# Services will be available at:
# - SIRP Platform: http://localhost:8000
# - Redis: localhost:6379
# - Webhook Receiver: http://localhost:5000
```

### Manual Installation

```bash
# Python 3.9+ required
git clone https://github.com/FunnyWolf/agentic-soc-platform.git
cd agentic-soc-platform

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your settings

# Initialize database
python manage.py migrate

# Start services
python manage.py runserver  # SIRP platform
python module_engine.py     # Module processor
python playbook_loader.py   # Playbook executor
python webhook_receiver.py  # Alert ingestion
```

## Configuration

### Environment Variables

```bash
# .env file
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/asp_db

# AI Agent Configuration
OPENAI_API_KEY=${OPENAI_API_KEY}
OPENAI_API_BASE=https://api.openai.com/v1

# Dify Configuration
DIFY_API_URL=http://localhost:5001
DIFY_API_KEY=${DIFY_API_KEY}

# Local LLM (Ollama)
OLLAMA_BASE_URL=http://localhost:11434

# SIRP Configuration
SIRP_API_URL=http://localhost:8000
SIRP_API_KEY=${SIRP_API_KEY}

# Webhook Settings
WEBHOOK_PORT=5000
WEBHOOK_SECRET=${WEBHOOK_SECRET}
```

### Redis Stream Configuration

```python
# config/redis_streams.py
ALERT_STREAMS = {
    'edr_alerts': 'stream:edr:alerts',
    'ndr_alerts': 'stream:ndr:alerts',
    'siem_alerts': 'stream:siem:alerts',
    'email_threats': 'stream:email:threats',
}

CONSUMER_GROUPS = {
    'edr_analyzer': ['stream:edr:alerts'],
    'ndr_analyzer': ['stream:ndr:alerts'],
    'threat_enricher': ['stream:siem:alerts', 'stream:email:threats'],
}
```

## Core Architecture

### Alert Processing Flow

```python
# webhook_receiver.py - Receiving alerts from SIEM
from flask import Flask, request, jsonify
import redis
import json

app = Flask(__name__)
r = redis.Redis(host='localhost', port=6379, db=0)

@app.route('/webhook/alert', methods=['POST'])
def receive_alert():
    """Receive alert from SIEM and push to Redis stream"""
    alert_data = request.json
    
    # Determine stream based on alert source
    source = alert_data.get('source', 'unknown')
    stream_key = f"stream:{source}:alerts"
    
    # Push to Redis stream
    message_id = r.xadd(
        stream_key,
        {
            'alert_id': alert_data.get('id'),
            'payload': json.dumps(alert_data),
            'timestamp': alert_data.get('timestamp'),
            'severity': alert_data.get('severity', 'medium')
        }
    )
    
    return jsonify({
        'status': 'success',
        'stream': stream_key,
        'message_id': message_id.decode()
    }), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## Creating Modules

Modules are streaming processors that consume alerts from Redis streams.

### Basic Module Structure

```python
# modules/edr_analyzer.py
from core.module_base import ModuleBase
from agents.langgraph_agent import LanggraphAgent
import json

class EDRAnalyzer(ModuleBase):
    """Analyze EDR alerts using AI agent"""
    
    def __init__(self):
        super().__init__(
            name='edr_analyzer',
            streams=['stream:edr:alerts'],
            consumer_group='edr_analysis_group'
        )
        self.agent = LanggraphAgent(
            model='gpt-4',
            system_prompt="""You are a security analyst specializing in EDR alerts.
            Analyze the alert and determine:
            1. Is this a true positive or false positive?
            2. What is the MITRE ATT&CK technique?
            3. What is the recommended response?
            """
        )
    
    async def process_message(self, message_id, data):
        """Process individual alert from stream"""
        alert_payload = json.loads(data['payload'])
        
        # AI analysis
        analysis = await self.agent.analyze({
            'alert_type': alert_payload.get('type'),
            'process_name': alert_payload.get('process_name'),
            'command_line': alert_payload.get('command_line'),
            'user': alert_payload.get('user'),
            'host': alert_payload.get('host')
        })
        
        # Create SIRP case
        sirp_case = await self.create_sirp_case({
            'title': f"EDR Alert: {alert_payload.get('type')}",
            'severity': self._map_severity(analysis['confidence']),
            'description': analysis['summary'],
            'mitre_technique': analysis.get('mitre_technique'),
            'recommended_action': analysis.get('recommendation'),
            'artifacts': [
                {
                    'type': 'process',
                    'value': alert_payload.get('process_name')
                },
                {
                    'type': 'host',
                    'value': alert_payload.get('host')
                }
            ]
        })
        
        # Acknowledge message
        await self.ack_message(message_id)
        
        return sirp_case
    
    def _map_severity(self, confidence):
        """Map AI confidence to severity level"""
        if confidence > 0.8:
            return 'critical'
        elif confidence > 0.6:
            return 'high'
        elif confidence > 0.4:
            return 'medium'
        return 'low'
```

### Module Base Class

```python
# core/module_base.py
import redis
import asyncio
from abc import ABC, abstractmethod
from typing import List, Dict
import logging

class ModuleBase(ABC):
    """Base class for all ASP modules"""
    
    def __init__(self, name: str, streams: List[str], consumer_group: str):
        self.name = name
        self.streams = streams
        self.consumer_group = consumer_group
        self.redis_client = redis.Redis(
            host='localhost',
            port=6379,
            db=0,
            decode_responses=True
        )
        self.logger = logging.getLogger(f"module.{name}")
        self._setup_consumer_groups()
    
    def _setup_consumer_groups(self):
        """Create consumer groups for streams"""
        for stream in self.streams:
            try:
                self.redis_client.xgroup_create(
                    stream,
                    self.consumer_group,
                    id='0',
                    mkstream=True
                )
            except redis.exceptions.ResponseError as e:
                if 'BUSYGROUP' not in str(e):
                    raise
    
    async def run(self):
        """Main processing loop"""
        self.logger.info(f"Starting module {self.name}")
        
        while True:
            for stream in self.streams:
                messages = self.redis_client.xreadgroup(
                    self.consumer_group,
                    self.name,  # Consumer name
                    {stream: '>'},
                    count=10,
                    block=1000
                )
                
                for stream_name, stream_messages in messages:
                    for message_id, data in stream_messages:
                        try:
                            await self.process_message(message_id, data)
                        except Exception as e:
                            self.logger.error(f"Error processing {message_id}: {e}")
            
            await asyncio.sleep(0.1)
    
    @abstractmethod
    async def process_message(self, message_id: str, data: Dict):
        """Process individual message - must be implemented by subclass"""
        pass
    
    async def ack_message(self, message_id: str):
        """Acknowledge processed message"""
        for stream in self.streams:
            self.redis_client.xack(stream, self.consumer_group, message_id)
    
    async def create_sirp_case(self, case_data: Dict):
        """Create case in SIRP platform"""
        from clients.sirp_client import SIRPClient
        
        client = SIRPClient()
        return await client.create_case(case_data)
```

## Creating Playbooks

Playbooks are event-driven automation tasks triggered from the SIRP UI.

### Basic Playbook Structure

```python
# playbooks/threat_intel_enrichment.py
from core.playbook_base import PlaybookBase
from typing import Dict, List

class ThreatIntelEnrichment(PlaybookBase):
    """Enrich indicators with threat intelligence"""
    
    metadata = {
        'name': 'Threat Intel Enrichment',
        'description': 'Query VirusTotal, AbuseIPDB, and other TI sources',
        'input_types': ['ip', 'domain', 'hash', 'url'],
        'output_type': 'enrichment_report'
    }
    
    async def execute(self, artifact: Dict) -> Dict:
        """Execute playbook on artifact"""
        artifact_type = artifact['type']
        artifact_value = artifact['value']
        
        results = {}
        
        if artifact_type == 'ip':
            results['virustotal'] = await self._query_virustotal_ip(artifact_value)
            results['abuseipdb'] = await self._query_abuseipdb(artifact_value)
            results['shodan'] = await self._query_shodan(artifact_value)
        
        elif artifact_type == 'domain':
            results['virustotal'] = await self._query_virustotal_domain(artifact_value)
            results['urlscan'] = await self._query_urlscan(artifact_value)
        
        elif artifact_type == 'hash':
            results['virustotal'] = await self._query_virustotal_hash(artifact_value)
            results['hybrid_analysis'] = await self._query_hybrid_analysis(artifact_value)
        
        # Update artifact in SIRP
        await self.update_artifact(artifact['id'], {
            'enrichment': results,
            'reputation_score': self._calculate_reputation(results),
            'tags': self._extract_tags(results)
        })
        
        return {
            'status': 'success',
            'artifact_id': artifact['id'],
            'enrichment_data': results
        }
    
    async def _query_virustotal_ip(self, ip: str) -> Dict:
        """Query VirusTotal IP endpoint"""
        import aiohttp
        import os
        
        api_key = os.getenv('VIRUSTOTAL_API_KEY')
        url = f"https://www.virustotal.com/api/v3/ip_addresses/{ip}"
        
        async with aiohttp.ClientSession() as session:
            async with session.get(
                url,
                headers={'x-apikey': api_key}
            ) as response:
                if response.status == 200:
                    data = await response.json()
                    return {
                        'malicious': data['data']['attributes']['last_analysis_stats']['malicious'],
                        'reputation': data['data']['attributes'].get('reputation', 0),
                        'country': data['data']['attributes'].get('country'),
                        'asn': data['data']['attributes'].get('asn')
                    }
                return {'error': f"Status {response.status}"}
    
    async def _query_abuseipdb(self, ip: str) -> Dict:
        """Query AbuseIPDB"""
        import aiohttp
        import os
        
        api_key = os.getenv('ABUSEIPDB_API_KEY')
        url = 'https://api.abuseipdb.com/api/v2/check'
        
        async with aiohttp.ClientSession() as session:
            async with session.get(
                url,
                headers={'Key': api_key},
                params={'ipAddress': ip, 'maxAgeInDays': 90}
            ) as response:
                if response.status == 200:
                    data = await response.json()
                    return {
                        'abuse_confidence_score': data['data']['abuseConfidenceScore'],
                        'total_reports': data['data']['totalReports'],
                        'is_whitelisted': data['data']['isWhitelisted']
                    }
                return {'error': f"Status {response.status}"}
    
    def _calculate_reputation(self, results: Dict) -> int:
        """Calculate overall reputation score (0-100, lower is worse)"""
        score = 100
        
        if 'virustotal' in results and 'malicious' in results['virustotal']:
            score -= results['virustotal']['malicious'] * 5
        
        if 'abuseipdb' in results and 'abuse_confidence_score' in results['abuseipdb']:
            score -= results['abuseipdb']['abuse_confidence_score']
        
        return max(0, score)
    
    def _extract_tags(self, results: Dict) -> List[str]:
        """Extract relevant tags from enrichment data"""
        tags = []
        
        if 'virustotal' in results:
            if results['virustotal'].get('malicious', 0) > 0:
                tags.append('malicious')
        
        if 'abuseipdb' in results:
            if results['abuseipdb'].get('abuse_confidence_score', 0) > 75:
                tags.append('high-confidence-abuse')
        
        return tags
```

### Playbook Base Class

```python
# core/playbook_base.py
from abc import ABC, abstractmethod
from typing import Dict, Any
import logging

class PlaybookBase(ABC):
    """Base class for all playbooks"""
    
    metadata = {
        'name': 'Base Playbook',
        'description': '',
        'input_types': [],
        'output_type': 'generic'
    }
    
    def __init__(self):
        self.logger = logging.getLogger(f"playbook.{self.metadata['name']}")
    
    @abstractmethod
    async def execute(self, artifact: Dict) -> Dict:
        """Execute playbook - must be implemented by subclass"""
        pass
    
    async def update_artifact(self, artifact_id: str, updates: Dict):
        """Update artifact in SIRP"""
        from clients.sirp_client import SIRPClient
        
        client = SIRPClient()
        return await client.update_artifact(artifact_id, updates)
    
    async def create_case_note(self, case_id: str, note: str):
        """Add note to case"""
        from clients.sirp_client import SIRPClient
        
        client = SIRPClient()
        return await client.add_case_note(case_id, note)
```

## AI Agent Integration

### Langgraph Agent

```python
# agents/langgraph_agent.py
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from typing import Dict, TypedDict
import os

class AnalysisState(TypedDict):
    alert_data: Dict
    analysis: Dict
    confidence: float
    mitre_technique: str
    recommendation: str

class LanggraphAgent:
    """AI agent using Langgraph for alert analysis"""
    
    def __init__(self, model: str = 'gpt-4', system_prompt: str = ''):
        self.llm = ChatOpenAI(
            model=model,
            api_key=os.getenv('OPENAI_API_KEY'),
            temperature=0.2
        )
        self.system_prompt = system_prompt
        self.graph = self._build_graph()
    
    def _build_graph(self) -> StateGraph:
        """Build Langgraph workflow"""
        workflow = StateGraph(AnalysisState)
        
        # Add nodes
        workflow.add_node("extract_iocs", self._extract_iocs)
        workflow.add_node("analyze_behavior", self._analyze_behavior)
        workflow.add_node("map_mitre", self._map_mitre)
        workflow.add_node("generate_recommendation", self._generate_recommendation)
        
        # Define edges
        workflow.set_entry_point("extract_iocs")
        workflow.add_edge("extract_iocs", "analyze_behavior")
        workflow.add_edge("analyze_behavior", "map_mitre")
        workflow.add_edge("map_mitre", "generate_recommendation")
        workflow.add_edge("generate_recommendation", END)
        
        return workflow.compile()
    
    async def analyze(self, alert_data: Dict) -> Dict:
        """Run analysis workflow"""
        initial_state = {
            'alert_data': alert_data,
            'analysis': {},
            'confidence': 0.0,
            'mitre_technique': '',
            'recommendation': ''
        }
        
        result = await self.graph.ainvoke(initial_state)
        return result
    
    async def _extract_iocs(self, state: AnalysisState) -> AnalysisState:
        """Extract indicators of compromise"""
        prompt = f"""Extract IOCs from this alert:
        {state['alert_data']}
        
        List all IPs, domains, file hashes, and processes."""
        
        response = await self.llm.ainvoke(prompt)
        state['analysis']['iocs'] = response.content
        return state
    
    async def _analyze_behavior(self, state: AnalysisState) -> AnalysisState:
        """Analyze behavior patterns"""
        prompt = f"""{self.system_prompt}
        
        Analyze this alert behavior:
        {state['alert_data']}
        
        Determine if this is malicious and confidence level (0-1)."""
        
        response = await self.llm.ainvoke(prompt)
        # Parse response for confidence
        state['confidence'] = 0.8  # Example
        state['analysis']['behavior'] = response.content
        return state
    
    async def _map_mitre(self, state: AnalysisState) -> AnalysisState:
        """Map to MITRE ATT&CK"""
        prompt = f"""Map this alert to MITRE ATT&CK:
        {state['alert_data']}
        
        Provide technique ID and name."""
        
        response = await self.llm.ainvoke(prompt)
        state['mitre_technique'] = response.content.strip()
        return state
    
    async def _generate_recommendation(self, state: AnalysisState) -> AnalysisState:
        """Generate response recommendation"""
        prompt = f"""Based on this analysis:
        Alert: {state['alert_data']}
        Confidence: {state['confidence']}
        MITRE: {state['mitre_technique']}
        
        What actions should be taken?"""
        
        response = await self.llm.ainvoke(prompt)
        state['recommendation'] = response.content
        return state
```

### Dify Agent Integration

```python
# agents/dify_agent.py
import aiohttp
import os
from typing import Dict

class DifyAgent:
    """Integration with Dify workflow platform"""
    
    def __init__(self):
        self.api_url = os.getenv('DIFY_API_URL')
        self.api_key = os.getenv('DIFY_API_KEY')
    
    async def run_workflow(self, workflow_id: str, inputs: Dict) -> Dict:
        """Execute Dify workflow"""
        url = f"{self.api_url}/v1/workflows/run"
        
        async with aiohttp.ClientSession() as session:
            async with session.post(
                url,
                headers={
                    'Authorization': f'Bearer {self.api_key}',
                    'Content-Type': 'application/json'
                },
                json={
                    'workflow_id': workflow_id,
                    'inputs': inputs
                }
            ) as response:
                return await response.json()
    
    async def analyze_alert(self, alert_data: Dict) -> Dict:
        """Use Dify workflow for alert analysis"""
        result = await self.run_workflow(
            workflow_id='alert-analysis-workflow',
            inputs={
                'alert_type': alert_data.get('type'),
                'alert_data': alert_data,
                'context': 'edr_analysis'
            }
        )
        
        return {
            'summary': result['data']['outputs']['summary'],
            'severity': result['data']['outputs']['severity'],
            'mitre_technique': result['data']['outputs']['mitre_technique'],
            'recommendation': result['data']['outputs']['recommendation']
        }
```

## SIRP Client

```python
# clients/sirp_client.py
import aiohttp
import os
from typing import Dict, List

class SIRPClient:
    """Client for interacting with SIRP platform"""
    
    def __init__(self):
        self.base_url = os.getenv('SIRP_API_URL', 'http://localhost:8000')
        self.api_key = os.getenv('SIRP_API_KEY')
    
    async def create_case(self, case_data: Dict) -> Dict:
        """Create new case in SIRP"""
        async with aiohttp.ClientSession() as session:
            async with session.post(
                f"{self.base_url}/api/v1/cases",
                headers={'Authorization': f'Bearer {self.api_key}'},
                json=case_data
            ) as response:
                return await response.json()
    
    async def update_case(self, case_id: str, updates: Dict) -> Dict:
        """Update existing case"""
        async with aiohttp.ClientSession() as session:
            async with session.patch(
                f"{self.base_url}/api/v1/cases/{case_id}",
                headers={'Authorization': f'Bearer {self.api_key}'},
                json=updates
            ) as response:
                return await response.json()
    
    async def add_artifact(self, case_id: str, artifact: Dict) -> Dict:
        """Add artifact to case"""
        async with aiohttp.ClientSession() as session:
            async with session.post(
                f"{self.base_url}/api/v1/cases/{case_id}/artifacts",
                headers={'Authorization': f'Bearer {self.api_key}'},
                json=artifact
            ) as response:
                return await response.json()
    
    async def update_artifact(self, artifact_id: str, updates: Dict) -> Dict:
        """Update artifact"""
        async with aiohttp.ClientSession() as session:
            async with session.patch(
                f"{self.base_url}/api/v1/artifacts/{artifact_id}",
                headers={'Authorization': f'Bearer {self.api_key}'},
                json=updates
            ) as response:
                return await response.json()
    
    async def add_case_note(self, case_id: str, note: str) -> Dict:
        """Add note to case"""
        async with aiohttp.ClientSession() as session:
            async with session.post(
                f"{self.base_url}/api/v1/cases/{case_id}/notes",
                headers={'Authorization': f'Bearer {self.api_key}'},
                json={'content': note}
            ) as response:
                return await response.json()
```

## Running the Platform

### Module Engine

```python
# module_engine.py
import asyncio
import importlib
import os
from pathlib import Path

async def load_modules():
    """Dynamically load all modules"""
    modules = []
    module_dir = Path('modules')
    
    for module_file in module_dir.glob('*.py'):
        if module_file.stem.startswith('_'):
            continue
        
        module = importlib.import_module(f'modules.{module_file.stem}')
        
        # Find module class
        for attr_name in dir(module):
            attr = getattr(module, attr_name)
            if (isinstance(attr, type) and 
                hasattr(attr, '__bases__') and 
                'ModuleBase' in [b.__name__ for b in attr.__bases__]):
                modules.append(attr())
    
    return modules

async def main():
    """Run all modules concurrently"""
    modules = await load_modules()
    
    print(f"Loaded {len(modules)} modules")
    for module in modules:
        print(f"  - {module.name}")
    
    # Run all modules
    await asyncio.gather(*[module.run() for module in modules])

if __name__ == '__main__':
    asyncio.run(main())
```

### Playbook Loader

```python
# playbook_loader.py
import asyncio
import importlib
from pathlib import Path
from flask import Flask, request, jsonify

app = Flask(__name__)
playbooks = {}

def load_playbooks():
    """Load all playbooks"""
    playbook_dir = Path('playbooks')
    
    for playbook_file in playbook_dir.glob('*.py'):
        if playbook_file.stem.startswith('_'):
            continue
        
        module = importlib.import_module(f'playbooks.{playbook_file.stem}')
        
        for attr_name in dir(module):
            attr = getattr(module, attr_name)
            if (isinstance(attr, type) and 
                hasattr(attr, '__bases__') and 
                'PlaybookBase' in [b.__name__ for b in attr.__bases__]):
                instance = attr()
                playbooks[instance.metadata['name']] = instance

@app.route('/api/v1/playbooks', methods=['GET'])
def list_playbooks():
    """List available playbooks"""
    return jsonify([
        {
            'name': pb.metadata['name'],
            'description': pb.metadata['description'],
            'input_types': pb.metadata['input_types']
        }
        for pb in playbooks.values()
    ])

@app.route('/api/v1/playbooks/<playbook_name>/execute', methods=['POST'])
async def execute_playbook(playbook_name):
    """Execute playbook on artifact"""
    if playbook_name not in playbooks:
        return jsonify({'error': 'Playbook not found'}), 404
    
    artifact = request.json
    playbook = playbooks[playbook_name]
    
    result = await playbook.execute(artifact)
    return jsonify(result)

if __name__ == '__main__':
    load_playbooks()
    print(f"Loaded {len(playbooks)} playbooks")
    app.run(host='0.0.0.0', port=5001)
```

## SIEM Integration

### Splunk Integration

```python
# integrations/splunk_forwarder.py
import requests
import json
import os

class SplunkForwarder:
    """Forward Splunk alerts to ASP"""
    
    def __init__(self):
        self.asp_webhook_url = os.getenv('ASP_WEBHOOK_URL', 'http://localhost:5000/webhook/alert')
        self.webhook_secret = os.getenv('WEBHOOK_SECRET')
    
    def format_alert(self, splunk_result: dict) -> dict:
        """Format Splunk alert for ASP"""
        return {
            'source': 'splunk',
            'id': splunk_result.get('sid'),
            'timestamp': splunk_result.get('_time'),
            'severity': self._map_severity(splunk_result.get('urgency')),
            'type': splunk_result.get('search_name'),
            'raw_data': splunk_result,
            'host': splunk_result.get('host'),
            'user': splunk_result.get('user'),
            'description': splunk_result.get('description')
        }
    
    def forward_alert(self, splunk_result: dict):
        """Send alert to ASP webhook"""
        alert = self.format_alert(splunk_result)
        
        response = requests.post(
            self.asp_webhook_url,
            json=alert,
            headers={
                'Content-Type': 'application/json',
                'X-Webhook-Secret': self.webhook_secret
            }
        )
        
        return response.json()
    
    def _map_severity(self, urgency: str) -> str:
        """Map Splunk urgency to ASP severity"""
        mapping = {
            'critical': 'critical',
            'high': 'high',
            'medium': 'medium',
            'low': 'low',
            'informational': 'info'
        }
        return mapping.get(urgency.lower(), 'medium')
```

### Kibana/ELK Integration

```python
# integrations/elk_forwarder.py
from elasticsearch import Elasticsearch
import os
import requests

class ELKForwarder:
    """Forward Elasticsearch alerts to ASP"""
    
    def __init__(self):
        self.es = Elasticsearch(
            [os.getenv('ELASTICSEARCH_URL', 'http://localhost:9200')],
            api_key=os.getenv('ELASTICSEARCH_API_KEY')
        )
        self.asp_webhook_url = os.getenv('ASP_WEBHOOK_URL')
    
