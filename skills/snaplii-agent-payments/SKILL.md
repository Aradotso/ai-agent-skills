---
name: snaplii-agent-payments
description: Enable AI agents to safely make real-world merchant purchases using Snaplii's tokenized gift card payment layer with up to 10% savings.
triggers:
  - "buy a gift card with snaplii"
  - "make a payment through snaplii for my agent"
  - "purchase merchant gift cards using snaplii"
  - "set up snaplii agent payments"
  - "browse available gift cards on snaplii"
  - "check my snaplii gift card balance"
  - "use snaplii to enable agent commerce"
  - "configure snaplii payment layer for AI agents"
---

# Snaplii Agent Payments

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Snaplii unlocks real-world commerce for AI agents through a safe, tokenized payment layer powered by 500+ merchant gift cards. It's the only payment solution that saves you money (up to 10% per transaction) while maintaining security through pre-funded, isolated, non-reusable transactions.

## What It Does

Snaplii solves the payment problem for AI agents:
- **Safe**: No shared credentials, each transaction is pre-funded and isolated
- **Controlled**: Users fund Snaplii Cash, agents operate within strict boundaries
- **Value-added**: Up to 10% savings that stack with existing merchant deals
- **Tokenized**: 500+ merchant gift cards available (CA and US)

The flow: **User → Agent → Snaplii → Merchant**

## Installation

### Requirements
- Python 3.10+ (CLI works on 3.9+, MCP server requires 3.10+)
- Snaplii Mobile App ([iOS](https://apps.apple.com/app/snaplii/id1596924498) / [Android](https://play.google.com/store/apps/details?id=com.snaplii.app))

### Clone the Repository

```bash
git clone https://github.com/Snaplii-Inc/agent-to-merchant-payments.git
cd agent-to-merchant-payments
```

### Install CLI with pipx (Recommended)

**macOS:**
```bash
brew install pipx
pipx ensurepath
pipx install -e ./snaplii-cli
```

**Linux:**
```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
pipx install -e ./snaplii-cli
```

**Windows (with Scoop):**
```powershell
scoop install pipx
pipx ensurepath
pipx install -e ./snaplii-cli
```

After installation, restart your terminal and verify:
```bash
snaplii --help
```

### Alternative: Install with pip

```bash
pip install -e ./snaplii-cli
```

If you get `externally-managed-environment` error:
```bash
pip install -e ./snaplii-cli --break-system-packages
```

## Getting Your API Key

1. Download and open the Snaplii mobile app
2. Register an account and bind a payment method to load Snaplii Cash
3. Navigate to: **More → Payment Methods → AI Payment Management**
4. Tap **+ New API Key**
5. Set a name, permission scope (Read-only or Purchase), and spending limit
6. Copy the API key (format: `snp_sk_live_...`) — it's shown only once

Store the key securely in your environment:
```bash
export SNAPLII_API_KEY="snp_sk_live_your_key_here"
```

## Authentication

Authenticate the CLI with your API key:

```bash
snaplii init
```

The CLI will prompt for your API key via hidden input (like a password). The key is used only to obtain a session token and is **never stored on disk**. Configuration is saved to `~/.snaplii/config.json`.

Verify authentication:
```bash
snaplii config show
```

## Key CLI Commands

### Browse Available Gift Cards

Browse by category (Canada):
```bash
snaplii browse tags --prov CA
```

Browse by category (United States):
```bash
snaplii browse tags --prov US
```

View specific brand details with denominations and cashback:
```bash
snaplii browse brand --id CB_AMAZON_CA
```

### Purchase Gift Cards

Purchase a gift card (requires `--item-id`, `--price`, and `--prov`):
```bash
# Ontario, Canada
snaplii purchase --item-id CB_STARBUCKS_CA-CT_DIGITAL --price 25 --prov ON

# California, USA
snaplii purchase --item-id CB_AMAZON_US-CT_DIGITAL --price 50 --prov CA
```

**Note on `--prov`:**
- For `browse tags`: use country code (`CA` or `US`)
- For `purchase`: use province/state code (`ON`, `QC`, `BC`, `NY`, `CA`, `TX`, etc.)

**Item ID Format:** `{cardBrandId}-{cardTemplateId}`
Both IDs are available from `snaplii browse brand` output.

### Get Purchase Quote

Preview price with voucher/cashback before buying:
```bash
snaplii quote --item-id CB_AMAZON_CA-CT_DIGITAL --price 100
```

### Manage Gift Cards

List all owned gift cards:
```bash
snaplii giftcard list
```

View card redemption code and PIN:
```bash
snaplii giftcard detail --card-no GC_1234567890
```

### Smart Features

Calculate cashback savings:
```bash
snaplii smart cashback --brand-id CB_AMAZON_CA --amount 100
```

View card inventory summary:
```bash
snaplii smart dashboard
```

## Python Code Examples

### Using the CLI from Python

```python
import subprocess
import json
import os

# Set API key in environment
os.environ['SNAPLII_API_KEY'] = 'snp_sk_live_...'

# Browse available tags
result = subprocess.run(
    ['snaplii', 'browse', 'tags', '--prov', 'CA'],
    capture_output=True,
    text=True
)
print(result.stdout)

# Purchase a gift card
purchase = subprocess.run(
    [
        'snaplii', 'purchase',
        '--item-id', 'CB_AMAZON_CA-CT_DIGITAL',
        '--price', '50',
        '--prov', 'ON'
    ],
    capture_output=True,
    text=True
)
print(purchase.stdout)

# List owned cards
cards = subprocess.run(
    ['snaplii', 'giftcard', 'list'],
    capture_output=True,
    text=True
)
print(cards.stdout)
```

### Automated Gift Card Purchase Flow

```python
import subprocess
import json

def get_quote(item_id, price):
    """Get purchase quote with cashback calculation."""
    result = subprocess.run(
        ['snaplii', 'quote', '--item-id', item_id, '--price', str(price)],
        capture_output=True,
        text=True
    )
    return result.stdout

def purchase_card(item_id, price, province):
    """Purchase a gift card and return card details."""
    result = subprocess.run(
        [
            'snaplii', 'purchase',
            '--item-id', item_id,
            '--price', str(price),
            '--prov', province
        ],
        capture_output=True,
        text=True
    )
    
    if result.returncode == 0:
        print(f"✓ Successfully purchased {item_id}")
        return result.stdout
    else:
        print(f"✗ Purchase failed: {result.stderr}")
        return None

def get_card_details(card_no):
    """Retrieve redemption code and PIN for a card."""
    result = subprocess.run(
        ['snaplii', 'giftcard', 'detail', '--card-no', card_no],
        capture_output=True,
        text=True
    )
    return result.stdout

# Example workflow
item_id = 'CB_STARBUCKS_CA-CT_DIGITAL'
quote = get_quote(item_id, 25)
print("Quote:", quote)

purchase = purchase_card(item_id, 25, 'ON')
if purchase:
    print(purchase)
```

### Check Available Balance and Inventory

```python
import subprocess

def check_dashboard():
    """View card inventory summary."""
    result = subprocess.run(
        ['snaplii', 'smart', 'dashboard'],
        capture_output=True,
        text=True
    )
    print(result.stdout)

def calculate_savings(brand_id, amount):
    """Calculate potential cashback savings."""
    result = subprocess.run(
        [
            'snaplii', 'smart', 'cashback',
            '--brand-id', brand_id,
            '--amount', str(amount)
        ],
        capture_output=True,
        text=True
    )
    print(result.stdout)

check_dashboard()
calculate_savings('CB_AMAZON_CA', 100)
```

## Claude Desktop MCP Server Setup

The MCP (Model Context Protocol) server allows Claude Desktop to make Snaplii purchases through natural conversation.

### Install MCP Dependencies

```bash
pip install -e ./snaplii-cli
pip install "mcp[cli]"
```

If you get `externally-managed-environment` error:
```bash
pip install -e ./snaplii-cli --break-system-packages
pip install "mcp[cli]" --break-system-packages
```

### Authenticate

```bash
snaplii init
```

### Configure Claude Desktop

Edit the Claude Desktop config file:

| OS | Config Location |
|---|---|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

Add this configuration (use absolute paths):

```json
{
  "mcpServers": {
    "snaplii": {
      "command": "/absolute/path/to/python3",
      "args": ["/absolute/path/to/agent-to-merchant-payments/mcp-server/server.py"]
    }
  }
}
```

Find your Python path:
```bash
which python3
```

On Windows:
```powershell
where.exe python3
```

### Restart Claude Desktop

Fully quit Claude Desktop (Cmd+Q on macOS) and reopen it.

### Verify in Claude

Ask Claude:
```
What gift cards are available on Snaplii?
```

Claude should automatically call `snaplii_browse_tags` and display categories.

## Claude Code Skill Setup

For Claude Code, install the skill definition:

```bash
mkdir -p ~/.claude/skills/snaplii-cli
cp skills/snaplii-cli.md ~/.claude/skills/snaplii-cli/SKILL.md
```

Edit `~/.claude/skills/snaplii-cli/SKILL.md` and update the PATH line if needed:

```bash
which snaplii
```

Replace the path in the skill file with the directory containing your `snaplii` binary. If `snaplii` is already on your PATH, you can remove the `export PATH=...` prefix entirely.

## Common Patterns

### Agent-Driven Purchase Workflow

```python
import subprocess

class SnapliAgent:
    def __init__(self):
        self.authenticated = self._check_auth()
    
    def _check_auth(self):
        """Verify authentication status."""
        result = subprocess.run(
            ['snaplii', 'config', 'show'],
            capture_output=True,
            text=True
        )
        return 'authenticated' in result.stdout.lower()
    
    def browse_brands(self, country='CA'):
        """Browse available gift card brands."""
        result = subprocess.run(
            ['snaplii', 'browse', 'tags', '--prov', country],
            capture_output=True,
            text=True
        )
        return result.stdout
    
    def get_brand_details(self, brand_id):
        """Get denominations and cashback for a brand."""
        result = subprocess.run(
            ['snaplii', 'browse', 'brand', '--id', brand_id],
            capture_output=True,
            text=True
        )
        return result.stdout
    
    def make_purchase(self, item_id, price, province):
        """Execute a gift card purchase."""
        if not self.authenticated:
            raise Exception("Not authenticated. Run 'snaplii init' first.")
        
        result = subprocess.run(
            [
                'snaplii', 'purchase',
                '--item-id', item_id,
                '--price', str(price),
                '--prov', province
            ],
            capture_output=True,
            text=True
        )
        
        if result.returncode != 0:
            raise Exception(f"Purchase failed: {result.stderr}")
        
        return result.stdout
    
    def list_inventory(self):
        """List all owned gift cards."""
        result = subprocess.run(
            ['snaplii', 'giftcard', 'list'],
            capture_output=True,
            text=True
        )
        return result.stdout

# Usage
agent = SnapliAgent()
print(agent.browse_brands('US'))
print(agent.get_brand_details('CB_AMAZON_US'))
purchase_result = agent.make_purchase(
    'CB_AMAZON_US-CT_DIGITAL',
    50,
    'NY'
)
print(purchase_result)
print(agent.list_inventory())
```

### Bulk Purchase with Error Handling

```python
import subprocess
import time

def bulk_purchase(purchases):
    """
    Purchase multiple gift cards with retry logic.
    
    purchases: list of dicts with keys: item_id, price, province
    """
    results = []
    
    for purchase in purchases:
        max_retries = 3
        retry_count = 0
        
        while retry_count < max_retries:
            try:
                result = subprocess.run(
                    [
                        'snaplii', 'purchase',
                        '--item-id', purchase['item_id'],
                        '--price', str(purchase['price']),
                        '--prov', purchase['province']
                    ],
                    capture_output=True,
                    text=True,
                    timeout=30
                )
                
                if result.returncode == 0:
                    results.append({
                        'success': True,
                        'purchase': purchase,
                        'output': result.stdout
                    })
                    break
                else:
                    retry_count += 1
                    if retry_count >= max_retries:
                        results.append({
                            'success': False,
                            'purchase': purchase,
                            'error': result.stderr
                        })
                    else:
                        time.sleep(2 ** retry_count)  # Exponential backoff
                        
            except subprocess.TimeoutExpired:
                retry_count += 1
                if retry_count >= max_retries:
                    results.append({
                        'success': False,
                        'purchase': purchase,
                        'error': 'Timeout after 30 seconds'
                    })
    
    return results

# Example usage
purchases = [
    {'item_id': 'CB_STARBUCKS_CA-CT_DIGITAL', 'price': 25, 'province': 'ON'},
    {'item_id': 'CB_AMAZON_CA-CT_DIGITAL', 'price': 50, 'province': 'ON'},
    {'item_id': 'CB_WALMART_US-CT_DIGITAL', 'price': 100, 'province': 'NY'}
]

results = bulk_purchase(purchases)
for r in results:
    if r['success']:
        print(f"✓ {r['purchase']['item_id']}: Success")
    else:
        print(f"✗ {r['purchase']['item_id']}: {r['error']}")
```

## Configuration

Configuration is stored in `~/.snaplii/config.json` after running `snaplii init`.

**Config structure:**
```json
{
  "agent_id": "derived_from_api_key",
  "session_token": "session_token_here",
  "token_expiry": "2026-06-01T12:00:00Z"
}
```

**Security notes:**
- API key is NEVER stored on disk
- Session tokens expire and require re-authentication
- Spending limits and scopes are enforced server-side

View current config:
```bash
snaplii config show
```

## Environment Variables

```bash
# Set API key for programmatic access
export SNAPLII_API_KEY="snp_sk_live_your_key_here"

# Optional: Set config directory (defaults to ~/.snaplii)
export SNAPLII_CONFIG_DIR="/custom/path/to/config"
```

## Troubleshooting

### Command Not Found

If `snaplii: command not found` after install:

```bash
# Check where it was installed
python3 -m pip show -f snaplii-cli
```

Look for `bin/snaplii` or `Scripts\snaplii.exe` in the output. Add that directory to your PATH or use `pipx` instead.

### Module Not Found (MCP Server)

If Claude Desktop logs `ModuleNotFoundError: No module named 'mcp'` or `'snaplii'`:

```bash
# Verify the Python interpreter has required packages
/absolute/path/to/python -c "import mcp, snaplii; print('ok')"
```

If it fails, install dependencies to that specific interpreter:
```bash
/absolute/path/to/python -m pip install -e ./snaplii-cli
/absolute/path/to/python -m pip install "mcp[cli]"
```

### Externally-Managed Environment

If pip refuses to install globally:

**Option 1 (Recommended):** Use `pipx`
```bash
pipx install -e ./snaplii-cli
```

**Option 2:** Use virtual environment
```bash
python3 -m venv ~/.venvs/snaplii
source ~/.venvs/snaplii/bin/activate
pip install -e ./snaplii-cli
```

**Option 3:** Override system protection (use cautiously)
```bash
pip install -e ./snaplii-cli --break-system-packages
```

### Authentication Issues

If authentication fails:

1. Verify API key format starts with `snp_sk_live_`
2. Check key permissions in Snaplii mobile app
3. Ensure spending limit is not exhausted
4. Re-run `snaplii init` to refresh session

### Purchase Failures

Common causes:
- Insufficient Snaplii Cash balance (load more in mobile app)
- Invalid province/state code (use 2-letter codes: ON, QC, NY, CA, TX)
- Item ID mismatch (verify with `snaplii browse brand --id <brand_id>`)
- API key lacks `PAY_WRITE` permission (check scope in mobile app)

Debug with verbose output:
```bash
snaplii purchase --item-id CB_AMAZON_CA-CT_DIGITAL --price 50 --prov ON -v
```

## Security Best Practices

1. **Never hardcode API keys** — always use environment variables
2. **Set strict spending limits** on each API key in the mobile app
3. **Use scoped permissions** — prefer `PAY_READ` for browse-only agents
4. **Rotate keys regularly** especially if exposed in logs
5. **Monitor transactions** via the Snaplii mobile app dashboard
6. **Revoke compromised keys** immediately in AI Payment Management

## Component Overview

```
agent-to-merchant-payments/
├── snaplii-cli/       # Python CLI (pip-installable)
├── mcp-server/        # MCP server for Claude Desktop
├── skills/            # Claude Code skill definition
├── clawhub-publish/   # ClawHub skill artifact
└── clawhub-plugin/    # ClawHub MCP bundle plugin
```

## Additional Resources

- Homepage: https://www.snaplii.com/
- Repository: https://github.com/Snaplii-Inc/agent-to-merchant-payments
- iOS App: https://apps.apple.com/app/snaplii/id1596924498
- Android App: https://play.google.com/store/apps/details?id=com.snaplii.app

## License

Apache License 2.0 — see repository for full license text.
