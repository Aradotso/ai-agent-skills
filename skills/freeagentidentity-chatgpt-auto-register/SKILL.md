---
name: freeagentidentity-chatgpt-auto-register
description: Multi-threaded automated ChatGPT free account registration and management panel with browser automation
triggers:
  - how do I register ChatGPT free accounts automatically
  - set up freeAgentIdentity account manager
  - automate ChatGPT account creation with freeAgentIdentity
  - configure email and proxy services for ChatGPT registration
  - manage ChatGPT free accounts in bulk
  - troubleshoot freeAgentIdentity registration tasks
  - export ChatGPT account credentials
  - configure browser automation for ChatGPT signup
---

# freeAgentIdentity ChatGPT Auto Register Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## What It Does

aBaiAutoplus (freeAgentIdentity) is a web-based management panel for automated ChatGPT free account registration. It provides:

- Multi-threaded automated ChatGPT account registration with browser automation
- Account lifecycle management (view, export, batch operations)
- Configurable email providers, proxy resources, and verification services
- Task logging and execution monitoring
- Web UI for configuration and account management
- Support for BitBrowser profile pooling

The project uses FastAPI backend with React/Vite frontend, Playwright for browser automation, and supports Docker deployment.

## Installation

### Prerequisites

- Python 3.11+
- Node.js 18+
- Playwright browsers (Chromium)

### Local Setup (Windows)

```powershell
# Clone and enter project
git clone https://github.com/asz798838958/freeAgentIdentity.git
cd freeAgentIdentity

# Create virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Install Python dependencies
pip install -r requirements.txt

# Install Playwright browsers
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

### Docker Deployment

```bash
docker compose up -d --build
```

Access web UI at `http://localhost:8000`

## Configuration

### Environment Variables

Copy `.env.example` to `.env` and configure:

```env
# Required for public deployment
APP_PASSWORD=your-secure-password

# Database
ACCOUNT_MANAGER_DATABASE_URL=sqlite:///./data/account_manager.db

# Optional: Browser settings
HEADLESS=true
BROWSER_TIMEOUT=30000
```

### Web UI Configuration

The settings page (`http://localhost:8000/settings`) provides configuration for:

1. **通用 (General)**: Theme, language, default registration strategy, browser reuse
2. **注册策略 (Registration Strategy)**: Default identity, execution mode, OAuth settings
3. **邮箱服务 (Email Services)**: Add/enable email providers with API credentials
4. **验证服务 (Verification Services)**: CAPTCHA solver configuration
5. **接码服务 (SMS Services)**: Phone verification provider setup
6. **代理资源 (Proxy Resources)**: Static/dynamic proxy management
7. **ChatGPT**: Platform-specific settings
8. **BitBrowser**: Profile pool management
9. **高级 (Advanced)**: Advanced platform overrides

## Key API Patterns

### Backend Structure

```
application/        # Task orchestration and API routes
├── tasks/         # Background registration tasks
├── api/           # FastAPI route handlers
└── services/      # Business logic layer

core/              # Core models and utilities
├── models/        # Data models
├── config/        # Configuration management
└── providers/     # Email, proxy, verification providers

platforms/chatgpt/ # ChatGPT-specific automation
├── register.py    # Registration workflow
└── captcha.py     # CAPTCHA handling
```

### Adding Custom Email Provider

```python
# In application/services/email_service.py or custom provider module

from core.providers.email_provider import EmailProvider

class CustomEmailProvider(EmailProvider):
    """Custom email provider implementation"""
    
    def __init__(self, api_key: str, **kwargs):
        self.api_key = api_key
        self.base_url = kwargs.get('base_url', 'https://api.example.com')
    
    async def create_email(self) -> dict:
        """Create new temporary email"""
        # Implementation
        response = await self.http_client.post(
            f"{self.base_url}/create",
            headers={"Authorization": f"Bearer {self.api_key}"}
        )
        return {
            "email": response.json()["email"],
            "token": response.json()["token"]
        }
    
    async def get_messages(self, email: str, token: str) -> list:
        """Retrieve messages for email"""
        response = await self.http_client.get(
            f"{self.base_url}/messages",
            params={"email": email},
            headers={"Authorization": f"Bearer {token}"}
        )
        return response.json()["messages"]
```

### Registration Task Implementation

```python
# In application/tasks/registration_task.py

from playwright.async_api import async_playwright
from platforms.chatgpt.register import ChatGPTRegistrar

async def run_registration_task(
    count: int,
    concurrency: int,
    email_provider: str,
    proxy_config: dict,
    registration_strategy: dict
):
    """Execute ChatGPT account registration task"""
    
    async with async_playwright() as p:
        browser = await p.chromium.launch(
            headless=registration_strategy.get('headless', True),
            proxy=proxy_config if proxy_config else None
        )
        
        registrar = ChatGPTRegistrar(
            browser=browser,
            email_provider=email_provider,
            strategy=registration_strategy
        )
        
        results = []
        for i in range(count):
            try:
                account = await registrar.register()
                results.append({
                    "email": account.email,
                    "password": account.password,
                    "status": "success"
                })
            except Exception as e:
                results.append({
                    "error": str(e),
                    "status": "failed"
                })
        
        await browser.close()
        return results
```

### API Route Example

```python
# In application/api/chatgpt_routes.py

from fastapi import APIRouter, BackgroundTasks, Depends
from pydantic import BaseModel

router = APIRouter(prefix="/api/chatgpt", tags=["chatgpt"])

class RegistrationRequest(BaseModel):
    count: int
    concurrency: int = 1
    email_provider: str
    use_proxy: bool = True
    registration_strategy: dict = {}

@router.post("/register")
async def start_registration(
    request: RegistrationRequest,
    background_tasks: BackgroundTasks
):
    """Start background registration task"""
    
    task_id = generate_task_id()
    
    background_tasks.add_task(
        run_registration_task,
        count=request.count,
        concurrency=request.concurrency,
        email_provider=request.email_provider,
        proxy_config=get_proxy_config() if request.use_proxy else None,
        registration_strategy=request.registration_strategy
    )
    
    return {
        "task_id": task_id,
        "status": "started",
        "count": request.count
    }

@router.get("/accounts")
async def list_accounts(skip: int = 0, limit: int = 100):
    """List registered ChatGPT accounts"""
    
    accounts = await db.get_accounts(skip=skip, limit=limit)
    return {
        "total": await db.count_accounts(),
        "accounts": accounts
    }

@router.post("/accounts/export")
async def export_accounts(account_ids: list[str]):
    """Export selected accounts to file"""
    
    accounts = await db.get_accounts_by_ids(account_ids)
    
    export_data = [
        {
            "email": acc.email,
            "password": acc.password,
            "created_at": acc.created_at.isoformat()
        }
        for acc in accounts
    ]
    
    return {"accounts": export_data}
```

## Common Patterns

### Custom Browser Automation Flow

```python
from playwright.async_api import Page

async def custom_chatgpt_flow(page: Page, email: str, password: str):
    """Custom ChatGPT signup automation"""
    
    # Navigate to signup
    await page.goto("https://chatgpt.com/signup")
    
    # Fill email
    await page.fill('input[type="email"]', email)
    await page.click('button[type="submit"]')
    
    # Wait for verification code input
    await page.wait_for_selector('input[name="code"]', timeout=60000)
    
    # Get verification code from email provider
    code = await get_verification_code(email)
    
    # Fill verification code
    await page.fill('input[name="code"]', code)
    await page.click('button[type="submit"]')
    
    # Set password
    await page.wait_for_selector('input[type="password"]')
    await page.fill('input[type="password"]', password)
    await page.click('button[type="submit"]')
    
    # Wait for dashboard
    await page.wait_for_url("**/chatgpt.com/**", timeout=30000)
    
    return {"status": "success", "email": email}
```

### Proxy Rotation Strategy

```python
# In core/providers/proxy_provider.py

class ProxyRotator:
    """Rotate through proxy pool for requests"""
    
    def __init__(self, proxies: list[dict]):
        self.proxies = proxies
        self.current_index = 0
    
    def get_next_proxy(self) -> dict:
        """Get next proxy in rotation"""
        proxy = self.proxies[self.current_index]
        self.current_index = (self.current_index + 1) % len(self.proxies)
        
        return {
            "server": proxy["url"],
            "username": proxy.get("username"),
            "password": proxy.get("password")
        }

# Usage in registration
proxy_rotator = ProxyRotator(load_proxies_from_config())

async with async_playwright() as p:
    for i in range(account_count):
        proxy = proxy_rotator.get_next_proxy()
        browser = await p.chromium.launch(proxy=proxy)
        # ... registration logic
        await browser.close()
```

### Database Operations

```python
# In core/models/account.py

from sqlalchemy import Column, String, DateTime, Integer
from sqlalchemy.ext.asyncio import AsyncSession
from datetime import datetime

class ChatGPTAccount(Base):
    __tablename__ = "chatgpt_accounts"
    
    id = Column(String, primary_key=True)
    email = Column(String, unique=True, nullable=False)
    password = Column(String, nullable=False)
    status = Column(String, default="active")
    created_at = Column(DateTime, default=datetime.utcnow)

# CRUD operations
async def create_account(session: AsyncSession, email: str, password: str):
    account = ChatGPTAccount(
        id=generate_uuid(),
        email=email,
        password=password
    )
    session.add(account)
    await session.commit()
    return account

async def get_accounts_by_status(session: AsyncSession, status: str):
    result = await session.execute(
        select(ChatGPTAccount).where(ChatGPTAccount.status == status)
    )
    return result.scalars().all()
```

## Frontend Development

### Development Mode

```bash
cd frontend
npm run dev
```

Frontend dev server runs on `http://localhost:5173` with API proxy to backend.

### Key Frontend Components

The React frontend is structured as:

```
frontend/src/
├── components/     # Reusable UI components
├── pages/          # Page components (Overview, ChatGPT, Settings)
├── api/            # API client functions
└── stores/         # State management
```

## Testing

Run automated tests:

```bash
# Python tests
pytest

# Frontend tests
cd frontend
npm test

# Sidebar navigation tests
pytest tests/test_frontend_sidebar_nav.py
```

## Troubleshooting

### Browser Automation Fails

**Issue**: Playwright browser doesn't launch or times out

**Solution**:
```bash
# Reinstall browsers
python -m playwright install --force chromium

# Check environment
python -m playwright install-deps
```

**Issue**: CAPTCHA not solving

**Solution**: Verify verification service configuration in Settings → 验证服务. Ensure API credentials are valid and service has available balance.

### Registration Task Hangs

**Issue**: Task starts but no accounts created

**Solution**:
- Check task logs in web UI under chatgpt free → Task Logs
- Verify email provider is responding: test email creation in Settings → 邮箱服务
- Ensure proxy configuration is valid if enabled
- Check browser headless mode: try `HEADLESS=false` for debugging

### Database Locked Errors

**Issue**: `sqlite3.OperationalError: database is locked`

**Solution**:
```python
# In .env, increase timeout
ACCOUNT_MANAGER_DATABASE_URL=sqlite:///./data/account_manager.db?timeout=30

# Or migrate to PostgreSQL for concurrent operations
ACCOUNT_MANAGER_DATABASE_URL=postgresql+asyncpg://user:pass@localhost/dbname
```

### Proxy Connection Failures

**Issue**: Accounts fail with proxy errors

**Solution**:
- Test proxy connectivity manually before adding to pool
- Use proxy with authentication if required
- Rotate proxies more frequently for rate-limited IPs
- Check proxy format: `http://user:pass@host:port`

### Email Verification Code Not Received

**Issue**: Registration waits for code that never arrives

**Solution**:
- Increase email check timeout in registration strategy
- Verify email provider API key has sufficient credits
- Check spam/filtering on email provider side
- Try alternative email provider from Settings → 邮箱服务

### Export Fails or Returns Empty

**Issue**: Account export produces no data

**Solution**:
```python
# Check account selection
accounts = await db.get_accounts_by_ids(account_ids)
if not accounts:
    # No accounts match the IDs
    
# Verify database connection
async with get_session() as session:
    count = await session.execute(select(func.count(ChatGPTAccount.id)))
    print(f"Total accounts in DB: {count.scalar()}")
```

## Security Best Practices

- **Never commit**: Real credentials, API keys, tokens, cookies, or account data
- Set strong `APP_PASSWORD` for production deployments
- Use environment variables for all sensitive configuration
- Rotate API keys regularly for email/proxy/verification services
- Review `.gitignore` to exclude `data/`, `.env`, and export files
- If secrets leaked to Git history, rewrite history and rotate all credentials immediately

## License

AGPL-3.0 - Based on `lxf746/any-auto-register` plugin framework.
