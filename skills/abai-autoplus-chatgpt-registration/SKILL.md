---
name: abai-autoplus-chatgpt-registration
description: Multi-threaded automated ChatGPT free account registration and management web panel with headless browser automation
triggers:
  - "set up ChatGPT account registration automation"
  - "how do I use aBaiAutoplus to register free accounts"
  - "configure email and captcha providers for account registration"
  - "manage ChatGPT free accounts with web panel"
  - "automate ChatGPT account creation with Playwright"
  - "run account registration tasks with proxy and verification"
  - "export registered ChatGPT account credentials"
  - "configure registration strategies for automated signup"
---

# aBaiAutoplus ChatGPT Registration Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What It Does

aBaiAutoplus is a web-based automation panel for registering, managing, and configuring ChatGPT free accounts at scale. It provides:

- Multi-threaded automated account registration with Playwright browser automation
- Email provider integration (temporary/custom email services)
- Captcha solving service integration
- SMS verification service integration
- Proxy resource management (static/dynamic)
- Account status tracking and export
- Registration strategy configuration
- Task logging and execution monitoring

The frontend exposes three main sections: Dashboard (overview), ChatGPT Free (account management), and Settings (provider/strategy configuration).

## Installation

### Prerequisites

- Python 3.11+
- Node.js 18+
- Playwright browsers

### Local Setup (Windows)

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt

# Install browser runtime for automation
python -m playwright install chromium

# Build frontend
cd frontend
npm install
npm run build
cd ..

# Start server
python -m uvicorn main:app --host 0.0.0.0 --port 8000
```

### Local Setup (macOS/Linux)

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

python -m playwright install chromium

cd frontend
npm install
npm run build
cd ..

python -m uvicorn main:app --host 0.0.0.0 --port 8000
```

Access the web UI at `http://localhost:8000`.

### Docker Deployment

```bash
docker compose up -d --build
```

Default port is `8000`. For production, always set a password.

### Frontend Development Mode

```bash
cd frontend
npm install
npm run dev
```

Frontend dev server runs on `http://localhost:5173`. Backend must run separately.

## Configuration

### Environment Variables

Copy `.env.example` to `.env`:

```env
# Security (required for public deployment)
APP_PASSWORD=your-secure-password

# Database
ACCOUNT_MANAGER_DATABASE_URL=sqlite:///./data/account_manager.db

# Optional: Override default settings
DEFAULT_REGISTRATION_STRATEGY=oauth
DEFAULT_BROWSER_REUSE=true
```

### Web UI Configuration

Most settings are configured through the **Settings** menu in the web panel:

- **General**: Theme, language, default registration strategy, browser reuse
- **Registration Strategy**: Default identity generation, OAuth vs manual, execution mode
- **Email Service**: Add/enable email providers (temporary email APIs)
- **Verification Service**: Captcha solver configuration (2Captcha, etc.)
- **SMS Service**: Phone verification provider setup
- **Proxy Resources**: Static/dynamic proxy pool management
- **ChatGPT**: Platform-specific settings
- **BitBrowser**: Browser profile pool management
- **Advanced**: Platform capability overrides

## Key API Endpoints

### Account Management

```python
# GET /api/accounts - List all ChatGPT accounts
# Query params: platform, status, skip, limit
import requests

response = requests.get(
    "http://localhost:8000/api/accounts",
    params={"platform": "chatgpt", "limit": 50}
)
accounts = response.json()
```

### Registration Tasks

```python
# POST /api/tasks/register - Create registration task
import requests

task_config = {
    "platform": "chatgpt",
    "count": 10,
    "concurrency": 3,
    "registration_strategy": {
        "identity_source": "generated",
        "execution_mode": "headless",
        "use_oauth": True
    },
    "email_provider": "tempmail_default",
    "captcha_provider": "2captcha",
    "proxy_config": {
        "enabled": True,
        "provider": "dynamic_pool"
    }
}

response = requests.post(
    "http://localhost:8000/api/tasks/register",
    json=task_config
)
task_id = response.json()["task_id"]
```

### Task Monitoring

```python
# GET /api/tasks/{task_id} - Check task status
response = requests.get(f"http://localhost:8000/api/tasks/{task_id}")
status = response.json()

# GET /api/tasks/{task_id}/logs - View task logs
logs_response = requests.get(f"http://localhost:8000/api/tasks/{task_id}/logs")
logs = logs_response.json()
```

### Export Accounts

```python
# POST /api/accounts/export - Export account credentials
export_config = {
    "account_ids": [1, 2, 3],
    "format": "csv",  # or "json"
    "fields": ["email", "password", "status", "created_at"]
}

response = requests.post(
    "http://localhost:8000/api/accounts/export",
    json=export_config
)

# Download the exported file
with open("accounts.csv", "wb") as f:
    f.write(response.content)
```

## Core Code Patterns

### Custom Registration Strategy

```python
# platforms/chatgpt/registration_strategy.py
from core.models.registration import RegistrationStrategy, ExecutionMode
from platforms.chatgpt.automation import ChatGPTAutomation

class CustomChatGPTStrategy(RegistrationStrategy):
    """Custom registration implementation"""
    
    async def execute(self, context):
        automation = ChatGPTAutomation(
            browser_config=context.browser_config,
            proxy=context.proxy
        )
        
        # Navigate to signup page
        await automation.navigate("https://chatgpt.com/signup")
        
        # Generate or use provided identity
        if self.identity_source == "generated":
            identity = await self.generate_identity()
        else:
            identity = context.provided_identity
        
        # Get temporary email
        email_provider = context.email_provider
        email = await email_provider.get_address()
        
        # Fill registration form
        await automation.fill_email(email)
        await automation.fill_password(identity.password)
        
        # Solve captcha if present
        if await automation.has_captcha():
            captcha_provider = context.captcha_provider
            solution = await captcha_provider.solve(
                await automation.get_captcha_challenge()
            )
            await automation.submit_captcha(solution)
        
        # Wait for verification email
        verification_link = await email_provider.wait_for_verification()
        await automation.navigate(verification_link)
        
        # Complete registration
        account_info = await automation.get_account_info()
        
        return {
            "email": email,
            "password": identity.password,
            "status": "active",
            "metadata": account_info
        }
```

### Email Provider Integration

```python
# application/providers/email_provider.py
from abc import ABC, abstractmethod

class EmailProvider(ABC):
    """Base class for email service providers"""
    
    @abstractmethod
    async def get_address(self) -> str:
        """Get a temporary email address"""
        pass
    
    @abstractmethod
    async def wait_for_verification(self, timeout: int = 300) -> str:
        """Wait for verification email and extract link"""
        pass
    
    @abstractmethod
    async def cleanup(self):
        """Release email address"""
        pass

# Example implementation
class TempMailProvider(EmailProvider):
    def __init__(self, api_key: str):
        import os
        self.api_key = os.getenv("TEMPMAIL_API_KEY", api_key)
        self.current_email = None
    
    async def get_address(self) -> str:
        import aiohttp
        async with aiohttp.ClientSession() as session:
            async with session.post(
                "https://api.tempmail.service/create",
                headers={"Authorization": f"Bearer {self.api_key}"}
            ) as resp:
                data = await resp.json()
                self.current_email = data["email"]
                return self.current_email
    
    async def wait_for_verification(self, timeout: int = 300) -> str:
        import asyncio
        import aiohttp
        
        start_time = asyncio.get_event_loop().time()
        
        while asyncio.get_event_loop().time() - start_time < timeout:
            async with aiohttp.ClientSession() as session:
                async with session.get(
                    f"https://api.tempmail.service/inbox/{self.current_email}",
                    headers={"Authorization": f"Bearer {self.api_key}"}
                ) as resp:
                    messages = await resp.json()
                    
                    for msg in messages:
                        if "verify" in msg.get("subject", "").lower():
                            # Extract verification link from email body
                            import re
                            match = re.search(
                                r'https://chatgpt\.com/verify\?token=[a-zA-Z0-9]+',
                                msg["body"]
                            )
                            if match:
                                return match.group(0)
            
            await asyncio.sleep(5)
        
        raise TimeoutError("Verification email not received")
    
    async def cleanup(self):
        self.current_email = None
```

### Captcha Solver Integration

```python
# application/providers/captcha_provider.py
from abc import ABC, abstractmethod

class CaptchaProvider(ABC):
    @abstractmethod
    async def solve(self, challenge_data: dict) -> str:
        """Solve captcha and return solution"""
        pass

class TwoCaptchaProvider(CaptchaProvider):
    def __init__(self):
        import os
        self.api_key = os.getenv("TWOCAPTCHA_API_KEY")
    
    async def solve(self, challenge_data: dict) -> str:
        import aiohttp
        
        # Submit captcha
        async with aiohttp.ClientSession() as session:
            async with session.post(
                "https://2captcha.com/in.php",
                data={
                    "key": self.api_key,
                    "method": challenge_data.get("method", "userrecaptcha"),
                    "googlekey": challenge_data.get("sitekey"),
                    "pageurl": challenge_data.get("url"),
                    "json": 1
                }
            ) as resp:
                result = await resp.json()
                request_id = result["request"]
            
            # Poll for solution
            import asyncio
            for _ in range(60):
                await asyncio.sleep(5)
                async with session.get(
                    f"https://2captcha.com/res.php",
                    params={
                        "key": self.api_key,
                        "action": "get",
                        "id": request_id,
                        "json": 1
                    }
                ) as resp:
                    result = await resp.json()
                    if result["status"] == 1:
                        return result["request"]
        
        raise TimeoutError("Captcha solving timeout")
```

### Proxy Management

```python
# application/proxy_manager.py
from typing import Optional
import aiohttp

class ProxyManager:
    """Manage proxy resources for registration tasks"""
    
    def __init__(self):
        self.static_proxies = []
        self.dynamic_provider = None
    
    def add_static_proxy(self, proxy_url: str):
        """Add static proxy: http://user:pass@host:port"""
        self.static_proxies.append(proxy_url)
    
    def configure_dynamic_provider(self, provider_url: str, api_key: str):
        """Configure dynamic proxy provider"""
        self.dynamic_provider = {
            "url": provider_url,
            "api_key": api_key
        }
    
    async def get_proxy(self, prefer_dynamic: bool = True) -> Optional[str]:
        """Get next available proxy"""
        if prefer_dynamic and self.dynamic_provider:
            async with aiohttp.ClientSession() as session:
                async with session.get(
                    f"{self.dynamic_provider['url']}/get",
                    headers={"Authorization": f"Bearer {self.dynamic_provider['api_key']}"}
                ) as resp:
                    data = await resp.json()
                    return data.get("proxy")
        
        if self.static_proxies:
            import random
            return random.choice(self.static_proxies)
        
        return None
    
    async def release_proxy(self, proxy_url: str):
        """Release dynamic proxy back to pool"""
        if self.dynamic_provider:
            async with aiohttp.ClientSession() as session:
                await session.post(
                    f"{self.dynamic_provider['url']}/release",
                    json={"proxy": proxy_url},
                    headers={"Authorization": f"Bearer {self.dynamic_provider['api_key']}"}
                )
```

### Registration Task Runner

```python
# application/tasks/registration_task.py
import asyncio
from typing import List
from core.models.account import Account

class RegistrationTask:
    """Execute multi-threaded account registration"""
    
    def __init__(
        self,
        count: int,
        concurrency: int,
        strategy,
        email_provider,
        captcha_provider,
        proxy_manager
    ):
        self.count = count
        self.concurrency = concurrency
        self.strategy = strategy
        self.email_provider = email_provider
        self.captcha_provider = captcha_provider
        self.proxy_manager = proxy_manager
        self.results = []
        self.errors = []
    
    async def register_single_account(self, index: int) -> Account:
        """Register one account"""
        proxy = await self.proxy_manager.get_proxy()
        
        try:
            context = {
                "browser_config": {"headless": True},
                "proxy": proxy,
                "email_provider": self.email_provider,
                "captcha_provider": self.captcha_provider,
                "identity_source": "generated"
            }
            
            account_data = await self.strategy.execute(context)
            
            # Save to database
            from core.database import get_db_session
            async with get_db_session() as session:
                account = Account(
                    platform="chatgpt",
                    email=account_data["email"],
                    password=account_data["password"],
                    status=account_data["status"],
                    metadata=account_data.get("metadata", {})
                )
                session.add(account)
                await session.commit()
                await session.refresh(account)
                
                self.results.append(account)
                return account
        
        except Exception as e:
            self.errors.append({"index": index, "error": str(e)})
            raise
        
        finally:
            if proxy:
                await self.proxy_manager.release_proxy(proxy)
    
    async def run(self) -> dict:
        """Execute registration task with concurrency control"""
        semaphore = asyncio.Semaphore(self.concurrency)
        
        async def bounded_register(index: int):
            async with semaphore:
                return await self.register_single_account(index)
        
        tasks = [bounded_register(i) for i in range(self.count)]
        await asyncio.gather(*tasks, return_exceptions=True)
        
        return {
            "total": self.count,
            "succeeded": len(self.results),
            "failed": len(self.errors),
            "accounts": self.results,
            "errors": self.errors
        }
```

## Common Usage Patterns

### Start a Registration Task via API

```python
import requests
import os

# Configure task
task = {
    "platform": "chatgpt",
    "count": 5,
    "concurrency": 2,
    "registration_strategy": {
        "identity_source": "generated",
        "execution_mode": "headless",
        "use_oauth": True
    },
    "email_provider": "tempmail_default",
    "captcha_provider": "2captcha",
    "proxy_config": {
        "enabled": True,
        "provider": "dynamic_pool"
    }
}

# Submit task
response = requests.post(
    "http://localhost:8000/api/tasks/register",
    json=task,
    headers={"Authorization": f"Bearer {os.getenv('APP_PASSWORD')}"}
)

task_id = response.json()["task_id"]
print(f"Task created: {task_id}")

# Monitor progress
import time
while True:
    status = requests.get(f"http://localhost:8000/api/tasks/{task_id}").json()
    print(f"Status: {status['state']} - {status['progress']}%")
    
    if status["state"] in ["completed", "failed"]:
        break
    
    time.sleep(5)

# Get results
accounts = requests.get(
    f"http://localhost:8000/api/tasks/{task_id}/results"
).json()

print(f"Registered {len(accounts)} accounts")
```

### Export Accounts to CSV

```python
import requests
import csv

# Fetch all active accounts
response = requests.get(
    "http://localhost:8000/api/accounts",
    params={"status": "active", "limit": 1000}
)
accounts = response.json()

# Export to CSV
with open("chatgpt_accounts.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["email", "password", "created_at"])
    writer.writeheader()
    
    for account in accounts:
        writer.writerow({
            "email": account["email"],
            "password": account["password"],
            "created_at": account["created_at"]
        })

print(f"Exported {len(accounts)} accounts")
```

### Configure Providers via Settings API

```python
import requests
import os

# Add email provider
email_config = {
    "name": "my_tempmail",
    "type": "tempmail_api",
    "enabled": True,
    "is_default": True,
    "config": {
        "api_key": os.getenv("TEMPMAIL_API_KEY"),
        "api_url": "https://api.tempmail.service"
    }
}

requests.post(
    "http://localhost:8000/api/settings/email-providers",
    json=email_config
)

# Add captcha provider
captcha_config = {
    "name": "2captcha_solver",
    "type": "2captcha",
    "enabled": True,
    "config": {
        "api_key": os.getenv("TWOCAPTCHA_API_KEY")
    }
}

requests.post(
    "http://localhost:8000/api/settings/captcha-providers",
    json=captcha_config
)

# Add static proxies
proxy_config = {
    "type": "static",
    "proxies": [
        "http://user:pass@proxy1.example.com:8080",
        "http://user:pass@proxy2.example.com:8080"
    ]
}

requests.post(
    "http://localhost:8000/api/settings/proxy-resources",
    json=proxy_config
)
```

## Troubleshooting

### Browser Automation Fails

**Issue**: Playwright browser not found or crashes

```bash
# Reinstall browser runtime
python -m playwright install chromium

# Check browser path
python -c "from playwright.sync_api import sync_playwright; print(sync_playwright().start().chromium.executable_path)"

# Run with visible browser for debugging
# Set execution_mode: "headed" in registration strategy
```

### Captcha Not Solving

**Issue**: Captcha provider returns errors

```python
# Test captcha provider directly
from application.providers.captcha_provider import TwoCaptchaProvider
import asyncio

async def test_captcha():
    provider = TwoCaptchaProvider()
    
    # Check API key
    print(f"API Key configured: {bool(provider.api_key)}")
    
    # Test with dummy challenge
    try:
        result = await provider.solve({
            "method": "userrecaptcha",
            "sitekey": "6Le-wvkSAAAAAPBMRTvw0Q4Muexq9bi0DJwx_mJ-",
            "url": "https://chatgpt.com/signup"
        })
        print(f"Captcha solved: {result[:20]}...")
    except Exception as e:
        print(f"Error: {e}")

asyncio.run(test_captcha())
```

**Solution**: Verify API key in `.env`, check provider balance, ensure correct captcha type

### Email Verification Timeout

**Issue**: `wait_for_verification()` times out

```python
# Increase timeout
email_provider.wait_for_verification(timeout=600)  # 10 minutes

# Check email inbox manually
print(f"Email address: {email_provider.current_email}")

# Debug email provider
import logging
logging.basicConfig(level=logging.DEBUG)
```

**Solution**: Check email provider API limits, verify email domain not blocked, increase polling timeout

### Database Locked Errors

**Issue**: SQLite database locked during concurrent writes

```env
# In .env, use WAL mode for better concurrency
ACCOUNT_MANAGER_DATABASE_URL=sqlite:///./data/account_manager.db?check_same_thread=False&timeout=20
```

Or migrate to PostgreSQL:

```env
ACCOUNT_MANAGER_DATABASE_URL=postgresql://user:pass@localhost/abai_autoplus
```

### Proxy Connection Failed

**Issue**: Registration fails with proxy errors

```python
# Test proxy connectivity
import aiohttp
import asyncio

async def test_proxy(proxy_url: str):
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(
                "https://chatgpt.com",
                proxy=proxy_url,
                timeout=aiohttp.ClientTimeout(total=30)
            ) as resp:
                print(f"Proxy works: {resp.status}")
    except Exception as e:
        print(f"Proxy failed: {e}")

asyncio.run(test_proxy("http://user:pass@proxy.example.com:8080"))
```

**Solution**: Verify proxy credentials, check proxy provider status, use dynamic proxy rotation

### High Memory Usage

**Issue**: Browser instances consuming too much RAM

```python
# Reduce concurrency
task_config = {
    "concurrency": 2,  # Lower value
    # ... other config
}

# Enable browser reuse in settings
# General > Browser Reuse: Enabled

# Or set in .env
DEFAULT_BROWSER_REUSE=true
```

### Task Stuck in Running State

**Issue**: Task doesn't complete or update

```bash
# Check task logs
curl http://localhost:8000/api/tasks/{task_id}/logs

# Restart worker process (if using background workers)
# Check for zombie browser processes
ps aux | grep chromium

# Kill stuck browsers
pkill -f chromium
```

## Testing

Run automated tests:

```bash
# Backend tests
pytest

# Frontend sidebar navigation tests
pytest tests/test_frontend_sidebar_nav.py

# Run specific test
pytest tests/test_registration_task.py -v
```

## Security Best Practices

- **Never commit** real credentials, API keys, or account data
- Use environment variables for all sensitive configuration
- Set `APP_PASSWORD` for production deployments
- Regularly rotate API keys and proxy credentials
- Review exported files before sharing
- Use `.gitignore` to exclude `data/`, `.env`, and `*.csv` files
