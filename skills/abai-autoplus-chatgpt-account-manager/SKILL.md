---
name: abai-autoplus-chatgpt-account-manager
description: Multi-threaded automated ChatGPT Free account registration and management web panel with proxy, email, and captcha provider integration
triggers:
  - register chatgpt free accounts automatically
  - manage chatgpt account registration
  - setup abai autoplus panel
  - configure chatgpt account automation
  - batch register free chatgpt accounts
  - integrate email and sms providers for registration
  - setup proxy resources for account creation
  - automate chatgpt workspace management
---

# aBaiAutoplus ChatGPT Account Manager

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

aBaiAutoplus is a web-based panel for automated ChatGPT Free account registration, management, and local configuration. It provides multi-threaded registration with OAuth bypass, configurable identity generation, proxy rotation, email/SMS provider integration, and captcha solving capabilities.

## What It Does

- **Automated Registration**: Multi-threaded ChatGPT Free account creation with configurable concurrency
- **Provider Management**: Email, SMS, captcha, and proxy provider configuration and rotation
- **Account Management**: View, export, and manage registered accounts with status tracking
- **Web Dashboard**: React-based UI for overview, registration, and configuration
- **Identity Generation**: Automated identity creation for registration
- **Browser Automation**: Playwright-based automation with optional BitBrowser profile support

## Installation

### Requirements

- Python 3.11+
- Node.js 18+
- Playwright browsers (for local automation)

### Standard Installation (Windows)

```powershell
# Clone and setup
git clone <repository-url>
cd aBaiAutoplus

# Create virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Install dependencies
pip install -r requirements.txt

# Install browser runtime
python -m playwright install chromium

# Build frontend
cd frontend
npm install
npm run build
cd ..

# Start server
python -m uvicorn main:app --host 0.0.0.0 --port 8000
```

### Standard Installation (macOS/Linux)

```bash
# Clone and setup
git clone <repository-url>
cd aBaiAutoplus

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Install browser runtime
python -m playwright install chromium

# Build frontend
cd frontend
npm install
npm run build
cd ..

# Start server
python -m uvicorn main:app --host 0.0.0.0 --port 8000
```

### Docker Installation

```bash
docker compose up -d --build
```

Access the web UI at `http://localhost:8000`

## Configuration

### Environment Variables

Create `.env` file from template:

```bash
cp .env.example .env
```

Essential configuration:

```env
# Access password (required for public deployment)
APP_PASSWORD=your-secure-password

# Database location
ACCOUNT_MANAGER_DATABASE_URL=sqlite:///./data/account_manager.db

# Optional: custom port
PORT=8000
```

### Web UI Configuration

Most configuration is done through the Settings page:

- **通用 (General)**: Theme, language, default registration strategy, browser reuse
- **注册策略 (Registration Strategy)**: Identity generation, execution mode, OAuth defaults
- **邮箱服务 (Email Service)**: Email provider configuration and parameters
- **验证服务 (Verification Service)**: Captcha solving provider setup
- **接码服务 (SMS Service)**: SMS receiving provider configuration
- **代理资源 (Proxy Resources)**: Static/dynamic proxy pool management
- **ChatGPT**: Platform-specific settings
- **BitBrowser**: Browser profile pool management
- **高级 (Advanced)**: Advanced configuration overrides

## Key API Endpoints

### Account Management

```python
# Get account list
GET /api/accounts
Query params: platform, status, page, page_size

# Get account details
GET /api/accounts/{account_id}

# Export accounts
POST /api/accounts/export
Body: {
  "account_ids": ["id1", "id2"],
  "format": "json" | "csv"
}
```

### Registration Tasks

```python
# Start registration task
POST /api/chatgpt/register
Body: {
  "count": 5,
  "concurrency": 3,
  "identity_strategy": "auto_generated",
  "execution_mode": "local" | "bitbrowser",
  "use_oauth": true
}

# Get task status
GET /api/tasks/{task_id}

# Get task logs
GET /api/tasks/{task_id}/logs
```

### Provider Configuration

```python
# List email providers
GET /api/providers/email

# Add email provider
POST /api/providers/email
Body: {
  "name": "provider_name",
  "type": "imap" | "api",
  "enabled": true,
  "config": {
    "host": "imap.example.com",
    "port": 993,
    "username": "${EMAIL_USERNAME}",
    "password": "${EMAIL_PASSWORD}"
  }
}

# List proxy resources
GET /api/proxies

# Add proxy
POST /api/proxies
Body: {
  "type": "http" | "socks5",
  "host": "proxy.example.com",
  "port": 1080,
  "username": "${PROXY_USER}",
  "password": "${PROXY_PASS}"
}
```

## Code Examples

### Python Backend - Custom Registration Task

```python
from application.chatgpt.registration import ChatGPTRegistrationService
from core.models.account import RegistrationConfig
from core.providers.proxy import ProxyProvider
from core.providers.email import EmailProvider

# Initialize services
registration_service = ChatGPTRegistrationService(
    email_provider=EmailProvider(),
    proxy_provider=ProxyProvider(),
    captcha_solver=CaptchaSolver()
)

# Configure registration
config = RegistrationConfig(
    count=10,
    concurrency=3,
    identity_strategy="auto_generated",
    execution_mode="local",
    use_oauth=True,
    proxy_enabled=True,
    captcha_auto_solve=True
)

# Start registration task
async def register_accounts():
    task = await registration_service.create_task(config)
    result = await registration_service.execute_task(task.id)
    
    print(f"Registered: {result.success_count}")
    print(f"Failed: {result.failed_count}")
    
    # Get registered accounts
    accounts = await registration_service.get_task_accounts(task.id)
    for account in accounts:
        print(f"Email: {account.email}, Status: {account.status}")

# Run
import asyncio
asyncio.run(register_accounts())
```

### Python Backend - Email Provider Integration

```python
from core.providers.email import IMAPEmailProvider, APIEmailProvider

# IMAP provider setup
imap_provider = IMAPEmailProvider(
    host="imap.gmail.com",
    port=993,
    username="${EMAIL_USERNAME}",
    password="${EMAIL_PASSWORD}",
    use_ssl=True
)

# Check for verification emails
async def check_verification():
    messages = await imap_provider.fetch_latest(
        folder="INBOX",
        subject_filter="verify",
        limit=5
    )
    
    for msg in messages:
        if "verification code" in msg.body:
            code = extract_code(msg.body)
            return code

# API-based email provider
api_provider = APIEmailProvider(
    api_url="https://api.tempmail.com",
    api_key="${TEMPMAIL_API_KEY}"
)

async def get_temp_email():
    email = await api_provider.create_email()
    print(f"Created temp email: {email.address}")
    
    # Wait for verification
    code = await api_provider.wait_for_code(
        email.address,
        timeout=300
    )
    return code
```

### Python Backend - Proxy Rotation

```python
from core.providers.proxy import ProxyRotator, ProxyResource

# Configure proxy pool
proxies = [
    ProxyResource(
        type="http",
        host="proxy1.example.com",
        port=8080,
        username="${PROXY_USER}",
        password="${PROXY_PASS}"
    ),
    ProxyResource(
        type="socks5",
        host="proxy2.example.com",
        port=1080
    )
]

rotator = ProxyRotator(proxies, strategy="round_robin")

# Use in registration
async def register_with_proxy():
    proxy = await rotator.get_next()
    
    browser_config = {
        "proxy": {
            "server": f"{proxy.type}://{proxy.host}:{proxy.port}",
            "username": proxy.username,
            "password": proxy.password
        }
    }
    
    # Launch browser with proxy
    from playwright.async_api import async_playwright
    
    async with async_playwright() as p:
        browser = await p.chromium.launch(proxy=browser_config["proxy"])
        page = await browser.new_page()
        
        # Registration logic here
        await page.goto("https://chat.openai.com/auth/signup")
        # ...
        
        await browser.close()
```

### Frontend - Registration Component Integration

```typescript
// frontend/src/services/registrationService.ts
import axios from 'axios';

interface RegistrationConfig {
  count: number;
  concurrency: number;
  identity_strategy: 'auto_generated' | 'custom';
  execution_mode: 'local' | 'bitbrowser';
  use_oauth: boolean;
}

export async function startRegistration(config: RegistrationConfig) {
  const response = await axios.post('/api/chatgpt/register', config);
  return response.data;
}

export async function getTaskStatus(taskId: string) {
  const response = await axios.get(`/api/tasks/${taskId}`);
  return response.data;
}

export async function getTaskLogs(taskId: string) {
  const response = await axios.get(`/api/tasks/${taskId}/logs`);
  return response.data;
}

// Usage in React component
import { useState } from 'react';
import { startRegistration, getTaskStatus } from '@/services/registrationService';

function RegistrationPanel() {
  const [taskId, setTaskId] = useState<string | null>(null);
  
  const handleRegister = async () => {
    const task = await startRegistration({
      count: 5,
      concurrency: 2,
      identity_strategy: 'auto_generated',
      execution_mode: 'local',
      use_oauth: true
    });
    
    setTaskId(task.id);
    
    // Poll for status
    const interval = setInterval(async () => {
      const status = await getTaskStatus(task.id);
      if (status.state === 'completed' || status.state === 'failed') {
        clearInterval(interval);
      }
    }, 2000);
  };
  
  return (
    <button onClick={handleRegister}>Start Registration</button>
  );
}
```

## Common Patterns

### Batch Account Registration

```python
from application.chatgpt.registration import ChatGPTRegistrationService

async def batch_register(total_accounts: int, batch_size: int = 10):
    """Register accounts in batches to avoid rate limits"""
    service = ChatGPTRegistrationService()
    
    for i in range(0, total_accounts, batch_size):
        count = min(batch_size, total_accounts - i)
        
        config = RegistrationConfig(
            count=count,
            concurrency=3,
            identity_strategy="auto_generated",
            use_oauth=True
        )
        
        task = await service.create_task(config)
        result = await service.execute_task(task.id)
        
        print(f"Batch {i//batch_size + 1}: {result.success_count}/{count} successful")
        
        # Wait between batches
        await asyncio.sleep(60)
```

### Export Accounts with Filters

```python
from application.account_manager import AccountManager

async def export_active_accounts(platform: str = "chatgpt"):
    """Export all active accounts to JSON"""
    manager = AccountManager()
    
    accounts = await manager.get_accounts(
        platform=platform,
        status="active",
        page_size=1000
    )
    
    export_data = [
        {
            "email": acc.email,
            "password": acc.password,
            "status": acc.status,
            "created_at": acc.created_at.isoformat()
        }
        for acc in accounts
    ]
    
    import json
    with open("active_accounts.json", "w") as f:
        json.dump(export_data, f, indent=2)
    
    print(f"Exported {len(export_data)} accounts")
```

### Custom Identity Generation

```python
from core.identity.generator import IdentityGenerator

class CustomIdentityGenerator(IdentityGenerator):
    def generate(self) -> dict:
        """Generate identity with specific patterns"""
        import random
        import string
        
        first_names = ["Alex", "Jordan", "Morgan", "Taylor"]
        last_names = ["Smith", "Johnson", "Williams", "Brown"]
        
        return {
            "first_name": random.choice(first_names),
            "last_name": random.choice(last_names),
            "username": f"{''.join(random.choices(string.ascii_lowercase, k=8))}",
            "password": f"{''.join(random.choices(string.ascii_letters + string.digits, k=16))}",
            "birth_date": "1990-01-01"
        }

# Use in registration
generator = CustomIdentityGenerator()
identity = generator.generate()
```

## Project Structure

```
.
├── main.py                      # FastAPI entry point
├── application/                 # Application layer
│   ├── chatgpt/                # ChatGPT registration logic
│   └── account_manager.py      # Account management service
├── core/                        # Core models and utilities
│   ├── models/                 # Data models
│   ├── providers/              # Email, SMS, proxy, captcha providers
│   └── identity/               # Identity generation
├── platforms/chatgpt/          # ChatGPT platform adapter
├── frontend/                    # React + Vite frontend
│   ├── src/
│   │   ├── components/        # UI components
│   │   ├── services/          # API services
│   │   └── pages/             # Dashboard pages
│   └── package.json
├── tests/                       # Test suite
├── docker-compose.yml          # Docker configuration
└── requirements.txt            # Python dependencies
```

## Troubleshooting

### Browser Automation Fails

```bash
# Reinstall Playwright browsers
python -m playwright install chromium --force

# Check browser installation
python -m playwright install --help
```

### Database Connection Issues

```python
# Verify database path exists
import os
db_path = "./data/account_manager.db"
os.makedirs(os.path.dirname(db_path), exist_ok=True)

# Check database URL in .env
ACCOUNT_MANAGER_DATABASE_URL=sqlite:///./data/account_manager.db
```

### Registration Task Hangs

```python
# Check task logs
GET /api/tasks/{task_id}/logs

# Common issues:
# 1. Proxy not responding - test proxy connectivity
# 2. Email provider timeout - increase timeout in settings
# 3. Captcha solving fails - verify captcha service API key
# 4. Rate limiting - reduce concurrency or add delays
```

### Frontend Build Errors

```bash
# Clear cache and rebuild
cd frontend
rm -rf node_modules package-lock.json
npm install
npm run build

# Development mode for debugging
npm run dev
```

### Proxy Connection Failures

```python
# Test proxy manually
import aiohttp

async def test_proxy():
    proxy = "http://username:password@proxy.example.com:8080"
    async with aiohttp.ClientSession() as session:
        async with session.get(
            "https://api.ipify.org",
            proxy=proxy
        ) as response:
            ip = await response.text()
            print(f"Proxy IP: {ip}")
```

### Email Verification Timeout

```python
# Increase timeout in provider config
email_provider_config = {
    "timeout": 600,  # 10 minutes
    "retry_interval": 10,  # Check every 10 seconds
    "max_retries": 60
}

# Use faster email providers or multiple providers for failover
```

## Testing

```bash
# Run all tests
pytest

# Test specific component
pytest tests/test_chatgpt_registration.py

# Frontend sidebar navigation test
pytest tests/test_frontend_sidebar_nav.py

# Run with coverage
pytest --cov=application --cov=core
```

## Security Notes

- Never commit real credentials, API keys, or account data
- Use environment variables for all sensitive configuration
- Set `APP_PASSWORD` for public deployments
- Keep `.env` and `data/` directory out of version control
- Rotate API keys if accidentally exposed in Git history

## License

AGPL-3.0 - Based on `lxf746/any-auto-register` plugin framework
