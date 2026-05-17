---
name: awesome-claude-code-subagents
description: Collection of 131+ specialized Claude Code subagents for development tasks across languages, frameworks, infrastructure, and quality assurance
triggers:
  - install a Claude subagent for Python development
  - show me available subagents for infrastructure
  - how do I use the TypeScript subagent
  - find a subagent for API design
  - install the React specialist agent
  - what subagents are available for security testing
  - set up a fullstack developer subagent
  - browse available Claude Code agents
---

# awesome-claude-code-subagents

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

A curated collection of 131+ specialized Claude Code subagents covering development tasks from frontend to infrastructure, language specialists, quality assurance, and meta-orchestration. Each subagent is a markdown file that configures Claude Code with expert knowledge in a specific domain.

## What This Project Does

This repository provides pre-configured Claude Code subagents that act as specialized AI assistants for:

- **Core Development** - API design, frontend, backend, fullstack, mobile, GraphQL, microservices
- **Language Specialists** - TypeScript, Python, Go, Rust, Java, JavaScript, PHP, C++, C#, Swift, Kotlin, and more
- **Infrastructure** - Docker, Kubernetes, Terraform, cloud providers, DevOps, SRE, databases
- **Quality & Security** - Code review, testing, security auditing, compliance, debugging
- **Data & Analytics** - Data engineering, ML ops, analytics
- **Documentation** - Technical writing, API docs, architecture diagrams
- **Emerging Tech** - Blockchain, IoT, edge computing, quantum
- **Business & Product** - Product management, business analysis
- **Meta-Orchestration** - Agent coordination, skill management, workflow automation

## Installation

### Prerequisites

- Claude Code CLI installed
- Git (for cloning)
- curl (for standalone installer)

### Option 1: Claude Code Plugin (Recommended)

```bash
# Add the plugin marketplace
claude plugin marketplace add VoltAgent/awesome-claude-code-subagents

# Install category plugins
claude plugin install voltagent-core-dev    # Core development
claude plugin install voltagent-lang        # Language specialists
claude plugin install voltagent-infra       # Infrastructure & DevOps
claude plugin install voltagent-qa-sec      # Quality & Security
claude plugin install voltagent-meta        # Meta-orchestration
```

### Option 2: Manual Installation

```bash
# Clone the repository
git clone https://github.com/VoltAgent/awesome-claude-code-subagents.git
cd awesome-claude-code-subagents

# Global installation
cp categories/02-language-specialists/python-pro.md ~/.claude/agents/

# Project-specific installation
mkdir -p .claude/agents
cp categories/01-core-development/api-designer.md .claude/agents/
```

### Option 3: Interactive Installer

```bash
git clone https://github.com/VoltAgent/awesome-claude-code-subagents.git
cd awesome-claude-code-subagents
chmod +x install-agents.sh
./install-agents.sh
```

Interactive menu allows browsing categories and selecting agents.

### Option 4: Standalone Installer (No Clone)

```bash
curl -sO https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/install-agents.sh
chmod +x install-agents.sh
./install-agents.sh
```

### Option 5: Agent Installer (Meta Agent)

```bash
# Install the agent-installer meta agent
curl -s https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/09-meta-orchestration/agent-installer.md \
  -o ~/.claude/agents/agent-installer.md
```

Then use in Claude Code:
```
Use the agent-installer to show me available categories
Find PHP agents and install php-pro globally
```

## Key Commands & Usage

### Listing Installed Agents

```bash
# List all installed agents
claude agents list

# List agents in specific directory
ls ~/.claude/agents/
ls .claude/agents/
```

### Using Subagents in Claude Code

Once installed, reference agents in your prompts:

```bash
# Activate a specific agent
@python-pro help me optimize this data processing pipeline

# Use multiple agents together
@api-designer @typescript-pro create a REST API with TypeScript

# Agent coordination
@meta-orchestrator coordinate frontend and backend development for user authentication
```

### Common Agent Selection Patterns

**Language-specific work:**
```
@typescript-pro refactor this code to use modern TypeScript patterns
@python-pro implement async processing with asyncio
@rust-engineer optimize this for zero-copy operations
```

**Infrastructure tasks:**
```
@kubernetes-specialist help me debug this pod networking issue
@terraform-engineer review my AWS infrastructure code
@docker-expert optimize this Dockerfile for production
```

**Quality & Security:**
```
@code-reviewer check this PR for best practices
@security-engineer audit this authentication implementation
@penetration-tester assess this API for security vulnerabilities
```

**Full-stack coordination:**
```
@fullstack-developer implement user profile editing feature
@meta-orchestrator plan a microservices migration strategy
```

## Agent File Structure

Each subagent is a markdown file with this structure:

```markdown
---
agent_name: python-pro
version: 1.0.0
specialization: Python ecosystem expert
---

# Python Pro Subagent

## Role
Expert in Python development, async programming, data processing...

## Expertise
- Python 3.10+ features
- AsyncIO and concurrency
- Popular frameworks (Django, FastAPI, Flask)
...

## Guidelines
- Use type hints
- Follow PEP 8
...
```

## Configuration

### Global vs Project-Specific Agents

**Global agents** (`~/.claude/agents/`):
- Available across all projects
- Use for general-purpose agents
- Language specialists, code reviewers

**Project-specific agents** (`.claude/agents/`):
- Available only in current project
- Use for domain-specific or customized agents
- Project-specific workflows

### Customizing Agents

```bash
# Copy and modify an agent
cp ~/.claude/agents/python-pro.md ~/.claude/agents/my-custom-python.md

# Edit the agent
vim ~/.claude/agents/my-custom-python.md
```

Example customization:

```markdown
---
agent_name: django-company-pro
version: 1.0.0
specialization: Django expert for CompanyName internal standards
---

# Django Company Pro

## Role
Django expert following CompanyName coding standards

## Additional Context
- Use our custom User model at `apps.accounts.models.User`
- All APIs must include our custom authentication middleware
- Follow our specific project structure in `docs/architecture.md`

## Company-Specific Patterns
```python
# Our standard API view pattern
from apps.core.views import CompanyAPIView

class UserProfileView(CompanyAPIView):
    permission_classes = [IsAuthenticated, HasCompanyPermission]
    
    def get(self, request):
        # Company standard response format
        return self.success_response(data, meta=self.get_meta())
```
```

## Real Code Examples

### Example 1: Using Python Pro for Data Processing

```python
# Ask: @python-pro help me optimize this data processing script

import asyncio
from typing import List, Dict
from dataclasses import dataclass
from concurrent.futures import ProcessPoolExecutor

@dataclass
class ProcessingResult:
    id: str
    status: str
    data: Dict

async def process_batch(items: List[Dict]) -> List[ProcessingResult]:
    """Process items in parallel using asyncio and multiprocessing."""
    loop = asyncio.get_event_loop()
    
    with ProcessPoolExecutor(max_workers=4) as executor:
        futures = [
            loop.run_in_executor(executor, process_item, item)
            for item in items
        ]
        results = await asyncio.gather(*futures)
    
    return results

def process_item(item: Dict) -> ProcessingResult:
    """CPU-intensive processing in separate process."""
    # Heavy computation here
    return ProcessingResult(
        id=item['id'],
        status='completed',
        data={'result': item['value'] * 2}
    )

# Usage
async def main():
    items = [{'id': str(i), 'value': i} for i in range(100)]
    results = await process_batch(items)
    print(f"Processed {len(results)} items")

asyncio.run(main())
```

### Example 2: Using TypeScript Pro for Type-Safe API

```typescript
// Ask: @typescript-pro create a type-safe API client

import axios, { AxiosInstance } from 'axios';

// Domain types
interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

interface CreateUserDto {
  email: string;
  name: string;
  password: string;
}

interface ApiResponse<T> {
  data: T;
  meta: {
    timestamp: string;
    requestId: string;
  };
}

// Type-safe API client
class UserApiClient {
  private client: AxiosInstance;

  constructor(baseURL: string) {
    this.client = axios.create({
      baseURL,
      headers: {
        'Content-Type': 'application/json',
      },
    });
  }

  async getUser(id: string): Promise<User> {
    const response = await this.client.get<ApiResponse<User>>(`/users/${id}`);
    return {
      ...response.data.data,
      createdAt: new Date(response.data.data.createdAt),
    };
  }

  async createUser(dto: CreateUserDto): Promise<User> {
    const response = await this.client.post<ApiResponse<User>>('/users', dto);
    return response.data.data;
  }

  async listUsers(filters?: { role?: string }): Promise<User[]> {
    const response = await this.client.get<ApiResponse<User[]>>('/users', {
      params: filters,
    });
    return response.data.data;
  }
}

// Usage with full type safety
const api = new UserApiClient(process.env.API_URL!);

const newUser = await api.createUser({
  email: 'user@example.com',
  name: 'John Doe',
  password: 'secure-password',
});

const user = await api.getUser(newUser.id);
console.log(user.createdAt.toISOString()); // Type-safe Date object
```

### Example 3: Using Terraform Engineer for Infrastructure

```hcl
# Ask: @terraform-engineer create a production-ready AWS infrastructure

# variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod"
  }
}

variable "app_name" {
  description = "Application name"
  type        = string
}

# vpc.tf
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "${var.app_name}-${var.environment}"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = var.environment != "prod"
  
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = local.common_tags
}

# ecs.tf
resource "aws_ecs_cluster" "main" {
  name = "${var.app_name}-${var.environment}"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }

  tags = local.common_tags
}

resource "aws_ecs_task_definition" "app" {
  family                   = "${var.app_name}-${var.environment}"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 256
  memory                   = 512
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn           = aws_iam_role.ecs_task.arn

  container_definitions = jsonencode([{
    name  = "app"
    image = "${aws_ecr_repository.app.repository_url}:latest"
    
    portMappings = [{
      containerPort = 8080
      protocol      = "tcp"
    }]

    environment = [
      {
        name  = "ENVIRONMENT"
        value = var.environment
      }
    ]

    secrets = [
      {
        name      = "DATABASE_URL"
        valueFrom = aws_secretsmanager_secret.db_url.arn
      }
    ]

    logConfiguration = {
      logDriver = "awslogs"
      options = {
        "awslogs-group"         = aws_cloudwatch_log_group.app.name
        "awslogs-region"        = data.aws_region.current.name
        "awslogs-stream-prefix" = "ecs"
      }
    }
  }])

  tags = local.common_tags
}

# locals.tf
locals {
  common_tags = {
    Environment = var.environment
    Application = var.app_name
    ManagedBy   = "Terraform"
  }
}

# outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "ecs_cluster_name" {
  description = "ECS cluster name"
  value       = aws_ecs_cluster.main.name
}
```

### Example 4: Using Docker Expert for Container Optimization

```dockerfile
# Ask: @docker-expert optimize this Dockerfile for production

# Multi-stage build for Node.js application
FROM node:20-alpine AS base
WORKDIR /app
ENV NODE_ENV=production

# Dependencies stage
FROM base AS deps
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

# Build stage
FROM base AS build
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM base AS production

# Security: Run as non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 appuser

# Copy dependencies and built app
COPY --from=deps --chown=appuser:nodejs /app/node_modules ./node_modules
COPY --from=build --chown=appuser:nodejs /app/dist ./dist
COPY --from=build --chown=appuser:nodejs /app/package*.json ./

# Security: Set correct permissions
RUN chown -R appuser:nodejs /app

USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

CMD ["node", "dist/main.js"]
```

```yaml
# docker-compose.yml for development
version: '3.8'

services:
  app:
    build:
      context: .
      target: base
      args:
        NODE_ENV: development
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=appdb
      - POSTGRES_USER=appuser
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

### Example 5: Using Meta-Orchestrator for Multi-Agent Coordination

```bash
# Ask: @meta-orchestrator plan and execute a full feature implementation

# Meta-orchestrator breaks down the task and coordinates agents:

# 1. Requirements gathering
@product-manager "Define user authentication feature requirements"

# 2. API design
@api-designer "Design authentication API endpoints with JWT tokens"

# 3. Backend implementation
@typescript-pro "Implement authentication service with bcrypt and JWT"

# 4. Database schema
@database-administrator "Create user and session tables with proper indexes"

# 5. Frontend implementation
@react-specialist "Create login and registration components with form validation"

# 6. Infrastructure setup
@kubernetes-specialist "Configure auth service deployment with secrets management"

# 7. Security review
@security-engineer "Review authentication implementation for vulnerabilities"

# 8. Testing
@testing-specialist "Create integration tests for auth flow"

# 9. Documentation
@technical-writer "Document authentication API and user flows"
```

## Common Patterns

### Pattern 1: Language Expert + Code Reviewer

```bash
# Develop with language expert, then review
@python-pro implement user service with async database access
@code-reviewer review the user service implementation for best practices
```

### Pattern 2: Infrastructure Stack Setup

```bash
# Coordinate infrastructure agents
@terraform-engineer create base VPC and networking
@kubernetes-specialist deploy application to EKS cluster
@docker-expert optimize container images for production
```

### Pattern 3: Full Feature Development

```bash
# Use meta-orchestrator to coordinate
@meta-orchestrator "Implement user profile editing feature with:
- Backend API in Python/FastAPI
- React frontend with TypeScript
- PostgreSQL database
- Kubernetes deployment
- Full test coverage"
```

### Pattern 4: Security-First Development

```bash
# Start with security review
@security-engineer review this authentication design
@typescript-pro implement the secure authentication flow
@penetration-tester test the auth implementation for vulnerabilities
```

### Pattern 5: Migration Projects

```bash
@meta-orchestrator "Plan migration from monolith to microservices:
- Analyze current Python Django monolith
- Design microservices architecture
- Create migration strategy
- Implement service mesh with Kubernetes"
```

## Troubleshooting

### Agent Not Found

```bash
# Check if agent is installed
ls ~/.claude/agents/
ls .claude/agents/

# Verify agent name
claude agents list

# Re-install if missing
cp categories/02-language-specialists/python-pro.md ~/.claude/agents/
```

### Agent Not Responding to Trigger

```bash
# Ensure you're using @ mention
@python-pro help with this code  # Correct
python-pro help with this code   # Incorrect

# Check agent file syntax
cat ~/.claude/agents/python-pro.md

# Verify YAML frontmatter is valid
```

### Multiple Agents Conflicting

```bash
# Be specific about which agent to use
@typescript-pro specifically, how should I type this function?

# Use meta-orchestrator to coordinate
@meta-orchestrator coordinate @frontend-developer and @backend-developer for this feature
```

### Agent File Permissions

```bash
# Fix permissions if agent files aren't readable
chmod 644 ~/.claude/agents/*.md

# Ensure directory permissions
chmod 755 ~/.claude/agents/
```

### Custom Agent Not Loading

```markdown
# Check YAML frontmatter format (must be valid)
---
agent_name: my-custom-agent
version: 1.0.0
specialization: Custom expertise
---

# Ensure no extra spaces or special characters in YAML
# Agent name must be kebab-case
# File must be .md extension
```

### Updating Agents

```bash
# Update all agents from repository
cd awesome-claude-code-subagents
git pull origin main

# Re-run installer to update
./install-agents.sh

# Or manually copy updated agents
cp categories/02-language-specialists/python-pro.md ~/.claude/agents/
```

### Debugging Agent Behavior

```bash
# Ask agent to explain its expertise
@python-pro what are your capabilities?

# Request specific guidance
@python-pro explain your approach to async programming

# Test with simple task first
@python-pro write a simple hello world function
```

## Environment Variables

Agents reference environment variables for sensitive data:

```bash
# Set in your environment or .env file
export DATABASE_URL="postgresql://user:pass@localhost:5432/db"
export API_KEY="${API_KEY}"  # Agents will use env var reference
export AWS_REGION="us-east-1"
```

Agents will use `${VARIABLE_NAME}` or `process.env.VARIABLE_NAME` syntax instead of hardcoded values.

## Additional Resources

- **GitHub Repository**: https://github.com/VoltAgent/awesome-claude-code-subagents
- **VoltAgent Homepage**: https://github.com/VoltAgent/voltagent
- **Discord Community**: https://s.voltagent.dev/discord
- **Related Collections**:
  - [Agent Skills](https://github.com/VoltAgent/awesome-agent-skills)
  - [OpenClaw Skills](https://github.com/VoltAgent/awesome-openclaw-skills)
  - [AI Agent Papers](https://github.com/VoltAgent/awesome-ai-agent-papers)

## Contributing

To add or improve agents:

1. Fork the repository
2. Add agent to appropriate category directory
3. Follow the agent template structure
4. Submit pull request

Agent template:
```markdown
---
agent_name: my-agent
version: 1.0.0
specialization: Brief description
---

# Agent Name

## Role
What this agent does

## Expertise
- Key skill 1
- Key skill 2

## Guidelines
- Best practice 1
- Best practice 2
```
