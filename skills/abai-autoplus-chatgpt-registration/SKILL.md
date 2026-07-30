---
name: abai-autoplus-chatgpt-registration
description: Multi-threaded automated ChatGPT free account registration and management web panel with OAuth bypass using Codex
triggers:
  - how do I register ChatGPT free accounts automatically
  - set up aBaiAutoplus for bulk account registration
  - configure email and captcha providers for ChatGPT registration
  - manage ChatGPT free accounts with aBaiAutoplus
  - automate ChatGPT account creation with proxy rotation
  - troubleshoot aBaiAutoplus registration tasks
  - export ChatGPT account credentials in bulk
  - configure BitBrowser profiles for account registration
---

# aBaiAutoplus ChatGPT Registration Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

aBaiAutoplus is a web-based automation panel for registering, managing, and configuring ChatGPT free accounts at scale. It features multi-threaded registration workflows, OAuth bypass capabilities, pluggable email/SMS/captcha providers, proxy rotation, and BitBrowser integration for fingerprint management.

## What It Does

- **Automated ChatGPT Registration**: Multi-threaded account creation with configurable concurrency and identity generation
- **Provider Management**: Configure email services, captcha solvers, SMS verification providers, and proxy pools
- **Account Management**: View, export, and manage registered accounts with status tracking
- **Web Dashboard**: React-based UI for monitoring registration tasks, account statistics, and system health
- **BitBrowser Integration**: Profile pool management for browser fingerprinting
- **Flexible Configuration**: Strategy-based registration with OAuth provider selection

## Installation

### Prerequisites

- Python 3.11+
- Node.js 18+
- Playwright (for browser automation)

### Windows Setup

```powershell
# Clone and enter directory
git clone https://github.com/asz798838958/freeAgentIdentity.git
cd freeAgentIdentity

# Create virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Install Python dependencies
pip install -r requirements.txt

# Install browser automation runtime
python -m playwright install chromium

# Build frontend
cd frontend
npm install
npm run build
cd ..

# Start the server
python -m uvicorn main:app --host 0.0.0.0 --port 8000
```

### macOS/Linux Setup

```bash
# Clone and enter directory
git clone https://github.com/asz798838958/freeAgentIdentity.git
cd freeAgentIdentity

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
python -m playwright install chromium

# Build frontend
cd frontend
npm install
npm run build
cd ..

# Start server
python -m uvicorn main:app --host 0.0.0.0 --port 8000
```

Access the web UI at `http://localhost:8000`

### Docker Deployment

```bash
# Start with Docker Compose
docker compose up -d --build
```

## Configuration

### Environment Variables

Create `.env` file from template:

```bash
cp .env.example .env
```

Key environment variables:

```env
# Security (REQUIRED for production)
APP_PASSWORD=your-secure-password

# Database
ACCOUNT_MANAGER_DATABASE_URL=sqlite:///./data/account_manager.db

# Optional: Custom host/port
HOST=0.0.0.0
PORT=8000
```

### Web UI Configuration

Most configuration is done through the **Settings** menu in the web UI:

1. **General**: Theme, language, default registration strategy, browser reuse
2. **Registration Strategy**: Default identity generation, execution method, OAuth providers
3. **Email Services**: Add and configure email providers (IMAP/POP3 credentials)
4. **Verification Services**: Configure captcha solving providers
5. **SMS Services**: Add phone number verification providers
6. **Proxy Resources**: Configure static/dynamic proxy pools
7. **ChatGPT**: Platform-specific settings
8. **BitBrowser**: Profile pool management
9. **Advanced**: Override platform capabilities

## Core Application Structure

```python
# main.py - FastAPI application entry point
from fastapi import FastAPI
from application.routes import router

app = FastAPI()
app.include_router(router)

# application/ - Task orchestration and API endpoints
# core/ - Core models, configuration, utilities
# platforms/chatgpt/ - ChatGPT platform adapter
# frontend/ - React + Vite web UI
```

## Key API Patterns

### Starting a Registration Task

```python
# application/tasks/registration_task.py usage pattern
from application.tasks.registration_task import start_registration_task

# Start automated registration
task_id = await start_registration_task(
    count=10,              # Number of accounts to register
    concurrency=3,         # Parallel threads
    identity_type="auto",  # Identity generation strategy
    execution_mode="headless",  # Browser mode
    oauth_provider="google",    # OAuth method
    workspace_join=True,   # Auto-join workspace after registration
)
```

### Fetching Account List

```python
# Via API endpoint pattern
from fastapi import APIRouter
from core.database import get_session

router = APIRouter()

@router.get("/api/chatgpt/accounts")
async def list_accounts(
    status: str = None,
    platform: str = "chatgpt",
    skip: int = 0,
    limit: int = 100
):
    """List ChatGPT accounts with filtering"""
    # Query database with filters
    # Returns: [{id, email, status, created_at, ...}]
    pass
```

### Configuring Email Provider

```python
# core/providers/email_provider.py pattern
from core.providers.email_provider import EmailProvider

# Add IMAP email provider
email_config = {
    "type": "imap",
    "host": "imap.gmail.com",
    "port": 993,
    "username": os.getenv("EMAIL_USERNAME"),
    "password": os.getenv("EMAIL_PASSWORD"),
    "use_ssl": True,
    "enabled": True,
    "is_default": True
}

# Providers are managed through web UI Settings → Email Services
```

### Proxy Configuration

```python
# core/providers/proxy_provider.py pattern
from core.providers.proxy_provider import ProxyProvider

# Static proxy configuration
static_proxy = {
    "type": "http",
    "host": "proxy.example.com",
    "port": 8080,
    "username": os.getenv("PROXY_USERNAME"),
    "password": os.getenv("PROXY_PASSWORD")
}

# Dynamic proxy pool (API-based)
dynamic_proxy = {
    "type": "api",
    "api_url": os.getenv("PROXY_API_URL"),
    "api_key": os.getenv("PROXY_API_KEY"),
    "rotation_interval": 300  # seconds
}
```

## Registration Workflow Example

```python
# Complete registration workflow pattern
from platforms.chatgpt.registration import ChatGPTRegistration
from core.identity_generator import generate_identity
from core.providers import get_email_provider, get_proxy_provider

async def register_chatgpt_account():
    # 1. Generate random identity
    identity = generate_identity(locale="en_US")
    
    # 2. Get providers
    email_provider = get_email_provider()
    proxy = get_proxy_provider().get_proxy()
    
    # 3. Initialize registration session
    registration = ChatGPTRegistration(
        identity=identity,
        email_provider=email_provider,
        proxy=proxy,
        headless=True
    )
    
    # 4. Execute registration steps
    try:
        # Navigate and fill registration form
        await registration.navigate_to_signup()
        await registration.fill_identity_info(identity)
        
        # Handle OAuth flow
        await registration.complete_oauth_flow(provider="google")
        
        # Verify email
        verification_code = await email_provider.get_verification_code()
        await registration.submit_verification(verification_code)
        
        # Save account credentials
        account_data = await registration.get_account_info()
        await save_account_to_database(account_data)
        
        return account_data
        
    except Exception as e:
        await registration.cleanup()
        raise e
```

## Batch Operations

### Export Accounts

```python
# Export accounts to JSON/CSV
from application.services.account_exporter import export_accounts

# Export specific accounts
account_ids = [1, 2, 3, 4, 5]
export_data = await export_accounts(
    account_ids=account_ids,
    format="json",  # or "csv"
    include_credentials=True
)

# Result format:
# [
#   {
#     "email": "user@example.com",
#     "password": "generated_password",
#     "status": "active",
#     "created_at": "2026-07-15T10:30:00Z"
#   }
# ]
```

### Bulk Status Update

```python
# Update multiple account statuses
from core.database import Account, get_session

async def mark_accounts_verified(account_ids: list[int]):
    async with get_session() as session:
        accounts = await session.query(Account).filter(
            Account.id.in_(account_ids)
        ).all()
        
        for account in accounts:
            account.status = "verified"
            account.verified_at = datetime.utcnow()
        
        await session.commit()
```

## Frontend Development

### Development Mode

```bash
cd frontend
npm install

# Start dev server (with hot reload)
npm run dev
# Frontend: http://localhost:5173
# Backend must run separately on :8000
```

### Build for Production

```bash
cd frontend
npm run build
# Static files output to frontend/dist/
# Served by FastAPI in production
```

## Common Patterns

### Task Monitoring

```python
# Check registration task status
from application.tasks.task_manager import get_task_status

task_status = await get_task_status(task_id)
# Returns: {
#   "status": "running",
#   "progress": 7,
#   "total": 10,
#   "completed": 7,
#   "failed": 0,
#   "logs": [...]
# }
```

### Account Filtering

```python
# Query accounts by criteria
from core.database import Account, get_session

async def get_active_accounts(platform: str = "chatgpt"):
    async with get_session() as session:
        accounts = await session.query(Account).filter(
            Account.platform == platform,
            Account.status == "active"
        ).all()
        return accounts
```

### Captcha Solving Integration

```python
# core/providers/captcha_provider.py pattern
from core.providers.captcha_provider import solve_captcha

# Solve reCAPTCHA v2
solution = await solve_captcha(
    captcha_type="recaptcha_v2",
    site_key=os.getenv("RECAPTCHA_SITE_KEY"),
    page_url="https://chatgpt.com/signup",
    provider="2captcha"  # or configured provider
)

await page.evaluate(f"document.getElementById('g-recaptcha-response').value='{solution}'")
```

## Troubleshooting

### Registration Fails with "Browser not found"

**Problem**: Playwright browsers not installed

**Solution**:
```bash
python -m playwright install chromium
```

### Email Verification Timeout

**Problem**: Email provider not receiving verification codes

**Solution**:
1. Check email provider settings in **Settings → Email Services**
2. Verify IMAP/POP3 credentials and connection
3. Check spam/junk folders configuration
4. Increase verification timeout in registration strategy

### Proxy Connection Errors

**Problem**: Proxy authentication failing or timeouts

**Solution**:
```python
# Verify proxy configuration
# Check proxy provider credentials in .env
PROXY_USERNAME=your_username
PROXY_PASSWORD=your_password

# Test proxy connection manually
import httpx

async with httpx.AsyncClient(
    proxies=f"http://{username}:{password}@{host}:{port}"
) as client:
    response = await client.get("https://api.ipify.org?format=json")
    print(response.json())  # Should show proxy IP
```

### Database Locked Errors

**Problem**: SQLite database locked during concurrent operations

**Solution**:
```env
# Use PostgreSQL for high concurrency
ACCOUNT_MANAGER_DATABASE_URL=postgresql://user:pass@localhost/dbname
```

Or reduce concurrency in registration tasks:
```python
# Lower concurrency value
task_id = await start_registration_task(
    count=10,
    concurrency=2,  # Reduce from 5+ to 2-3
    # ...
)
```

### Frontend Build Fails

**Problem**: npm build errors or missing dependencies

**Solution**:
```bash
cd frontend
rm -rf node_modules package-lock.json
npm install
npm run build
```

### OAuth Flow Interrupted

**Problem**: Google/OAuth login fails or times out

**Solution**:
1. Ensure browser is running in non-headless mode for debugging:
```python
registration = ChatGPTRegistration(
    headless=False  # See browser window
)
```
2. Check OAuth provider configuration in **Settings → Registration Strategy**
3. Verify identity data quality (valid names, DOB, etc.)
4. Check for captcha challenges that require manual solving

## Testing

```bash
# Run test suite
pytest

# Test specific module
pytest tests/test_frontend_sidebar_nav.py

# Run with coverage
pytest --cov=application --cov=core --cov=platforms
```

## Security Best Practices

**Never commit to repository:**
- Real account credentials (email, password, tokens)
- API keys (captcha services, proxy providers, SMS services)
- Database files (`*.db`)
- Browser profiles and cookies
- `.env` file with secrets

**Use environment variables:**
```env
EMAIL_USERNAME=${EMAIL_USERNAME}
EMAIL_PASSWORD=${EMAIL_PASSWORD}
PROXY_API_KEY=${PROXY_API_KEY}
CAPTCHA_API_KEY=${CAPTCHA_API_KEY}
APP_PASSWORD=${APP_PASSWORD}
```

**Secure production deployments:**
```env
# Always set password for public deployments
APP_PASSWORD=strong-random-password-here

# Use HTTPS reverse proxy (nginx, Caddy)
# Restrict firewall access to port 8000
```

## Additional Resources

- QQ Group: https://qm.qq.com/q/JigtiO2Hyc
- License: AGPL-3.0
- Original Framework: lxf746/any-auto-register
