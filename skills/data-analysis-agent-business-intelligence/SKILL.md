---
name: data-analysis-agent-business-intelligence
description: Use the Data Analysis Agent to perform natural language business analytics, auto-generate SQL queries, create visualizations, and produce business insights from Excel/CSV/databases
triggers:
  - analyze sales data with natural language queries
  - generate charts and insights from business data automatically
  - connect to database and query with plain English
  - build data analysis agent for business intelligence
  - create interactive data visualizations with AI
  - auto-generate SQL from natural language questions
  - perform business analytics without writing SQL
  - build conversational BI dashboard with streaming output
---

# Data Analysis Agent — Business Intelligence Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Data Analysis Agent is a conversational business intelligence system that enables non-technical users to perform data analysis through natural language. Users can upload Excel/CSV files or connect to databases, then ask questions in plain language. The system automatically understands intent, generates SQL, executes queries, recommends charts, and provides business insights with real-time SSE (Server-Sent Events) streaming output.

## What It Does

- **Natural Language Queries**: Ask questions in plain English instead of writing SQL
- **Auto SQL Generation**: Converts natural language to optimized SQL queries
- **Smart Visualization**: Automatically recommends from 43+ chart types based on data patterns
- **Multi-Source Support**: Excel, CSV, SQLite, MySQL, PostgreSQL, SQL Server, Google Sheets
- **Streaming Analysis**: Real-time SSE output showing analysis progress
- **Advanced Analytics**: K-Means clustering, decision trees, outlier handling, decile analysis
- **Report Generation**: Export to Excel, Word, PowerPoint
- **MCP Integration**: Extend capabilities with Model Context Protocol tools
- **Knowledge Base**: Upload business documents to improve domain understanding

## Installation

### Method 1: Download Release Package (Recommended)

```bash
# Download from releases page
# Extract the archive
# Navigate to directory

# Windows
start.bat

# macOS (first time - grant permissions)
chmod +x start.command
./start.command

# Access at http://localhost:5001
```

### Method 2: Clone from GitHub

```bash
# Clone repository
git clone https://github.com/Zafer-Liu/Data-Analysis-Agent.git
cd Data-Analysis-Agent

# Install dependencies
pip install -r requirements.txt

# Start server
python app.py

# Access at http://localhost:5001
```

### Method 3: One-Line Install

**Windows (PowerShell):**
```powershell
iwr -useb https://raw.githubusercontent.com/Zafer-Liu/Data-Analysis-Agent/main/install.ps1 | iex
```

**macOS/Linux:**
```bash
curl -fsSL https://raw.githubusercontent.com/Zafer-Liu/Data-Analysis-Agent/main/install.sh | sh
data-analysis-agent
```

## Configuration

### LLM Setup

Configure your LLM provider in the web UI sidebar (⚙ icon):

```python
# Supported providers
providers = {
    "deepseek": {
        "base_url": "https://api.deepseek.com",
        "model": "deepseek-chat",
        "api_key": os.getenv("DEEPSEEK_API_KEY")
    },
    "openai": {
        "base_url": "https://api.openai.com/v1",
        "model": "gpt-4o-mini",
        "api_key": os.getenv("OPENAI_API_KEY")
    },
    "anthropic": {
        "base_url": "https://api.anthropic.com",
        "model": "claude-3-5-haiku-20241022",
        "api_key": os.getenv("ANTHROPIC_API_KEY")
    }
}
```

### Database Connection

**MySQL/PostgreSQL:**
```python
# Connection string format
mysql_connection = "mysql+pymysql://{username}:{password}@{host}:{port}/{database}"
postgres_connection = "postgresql://{username}:{password}@{host}:{port}/{database}"

# Example (use environment variables)
import os
db_url = f"mysql+pymysql://{os.getenv('DB_USER')}:{os.getenv('DB_PASS')}@localhost:3306/sales_db"
```

**Google Sheets:**
```python
# Configure Google Sheets API credentials
# Upload credentials JSON in the web UI
# Enter Sheet ID to connect
```

## Slash Commands

Core commands for specialized analysis:

```bash
/chart        # Force chart generation priority
/sql          # Execute SQL directly
/analyze      # Deep statistical analysis
/tree         # Decision tree modeling
/kmeans       # K-Means clustering
/data         # Data exploration and preview
/inset        # Missing value imputation
/winsorize    # Winsorize outliers (replace extremes)
/trimming     # Trim outliers (remove extremes)
/export       # Export data file
/report       # Generate Word/PDF report
/ppt          # Generate PowerPoint presentation
/status       # Check task status
```

## Code Examples

### Natural Language Query

```python
# Example query flow in the web UI
user_query = "What is the sales trend for the last 12 months?"

# System automatically:
# 1. Analyzes schema
# 2. Generates SQL: 
#    SELECT DATE_FORMAT(order_date, '%Y-%m') as month, 
#           SUM(sales_amount) as total_sales
#    FROM sales 
#    WHERE order_date >= DATE_SUB(NOW(), INTERVAL 12 MONTH)
#    GROUP BY month
#    ORDER BY month
# 3. Executes query
# 4. Recommends Line_Chart visualization
# 5. Generates insights
```

### Programmatic API Usage

```python
from flask import Flask
from modules.agent_core import analyze_query
import os

# Initialize
app = Flask(__name__)

# Configure LLM
llm_config = {
    "provider": "deepseek",
    "api_key": os.getenv("DEEPSEEK_API_KEY"),
    "base_url": "https://api.deepseek.com",
    "model": "deepseek-chat"
}

# Analyze data
def query_data(question, data_source):
    """
    Question: Natural language query
    data_source: Path to CSV/Excel or database connection string
    """
    result = analyze_query(
        question=question,
        data_source=data_source,
        llm_config=llm_config,
        stream=True  # Enable SSE streaming
    )
    return result

# Example usage
response = query_data(
    question="Which region has the highest profit?",
    data_source="./data/sales_data.csv"
)
```

### Chart Generation

```python
# Force chart generation with /chart command
user_input = "/chart user growth over time"

# System response includes:
# - SQL query
# - Data result
# - Chart recommendation (e.g., Line_Chart, Area_Chart)
# - Interactive Plotly visualization
# - Chart saved to ./outputs/charts/
```

### Advanced Analytics: K-Means Clustering

```python
# Use /kmeans command
query = "/kmeans segment customers by purchase behavior"

# System performs:
# 1. Feature selection
# 2. Data normalization
# 3. Optimal cluster determination (elbow method)
# 4. K-Means clustering
# 5. Cluster visualization
# 6. Business interpretation of segments
```

### MCP Tool Integration

```python
# Enable MCP tools in configuration
mcp_config = {
    "enabled": True,
    "tools": [
        {
            "name": "calculator",
            "endpoint": "http://localhost:8000/mcp/calculator"
        },
        {
            "name": "code_executor",
            "endpoint": "http://localhost:8000/mcp/execute"
        }
    ]
}

# Agent automatically invokes MCP tools when needed
# Example: Complex financial calculations beyond SQL
```

### Knowledge Base Integration

```python
# Upload business documents via web UI
# Supported formats: .docx, .xlsx, .pdf, .txt

# Documents are vectorized and stored
# Agent retrieves relevant context automatically

# Example query with knowledge enhancement:
query = "Analyze Q4 sales using last year's strategic priorities"
# Agent retrieves relevant strategic docs from knowledge base
# Provides context-aware insights aligned with business goals
```

## Common Patterns

### Pattern 1: Time Series Analysis

```python
# Natural language input
"Show me monthly revenue trend with year-over-year comparison"

# Generated SQL pattern
"""
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') as month,
    SUM(revenue) as current_revenue,
    LAG(SUM(revenue), 12) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m')) as prev_year_revenue,
    ((SUM(revenue) - LAG(SUM(revenue), 12) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m'))) / 
     LAG(SUM(revenue), 12) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m'))) * 100 as yoy_growth
FROM orders
GROUP BY month
ORDER BY month
"""

# Auto-recommended chart: Line_Chart with dual axis
```

### Pattern 2: Outlier Detection & Handling

```python
# Detect outliers
query = "Find outliers in customer spending data"

# Apply winsorization
query = "/winsorize cap extreme values at 5th and 95th percentile"

# Or trim outliers
query = "/trimming remove values beyond 3 standard deviations"
```

### Pattern 3: Cohort Analysis

```python
query = "Create cohort analysis for user retention by signup month"

# System generates cohort table and retention curve
# Automatically suggests Heatmap for cohort visualization
```

### Pattern 4: Export Workflow

```python
# After analysis, export results
"/export save cleaned data to Excel"

# Generate comprehensive report
"/report create analysis report with all charts and insights"

# Create presentation
"/ppt generate executive summary presentation"

# Files saved to ./outputs/ directory
```

## Chart Types Reference

The system auto-selects from 43 chart types:

```python
chart_categories = {
    "COMPARING": [
        "Bar_Chart", "Grouped_Bar_Chart", "Stacked_Bar_Chart",
        "Diverging_Bar_Chart", "Marimekko_ABS", "Waterfall", "Sankey_Chart"
    ],
    "TIME": [
        "Line_Chart", "Area_Chart", "Stacked_Area_Chart",
        "Slope_Chart", "Bump_Chart", "Sparkline"
    ],
    "DISTRIBUTION": [
        "Histogram_Pareto_chart", "Box-and-Whisker_Plot", 
        "Violin_Chart", "Ridgeline_Plot", "Beeswarm_Plot"
    ],
    "GEOSPATIAL": [
        "Choropleth_Map", "Dot_Density_Map", "Flow_Map"
    ],
    "RELATIONSHIP": [
        "Scatter_Plot", "Bubble_Plot", "Network_Diagram",
        "Chord_Diagram", "Parallel_Coordinates_Plot"
    ],
    "PART-TO-WHOLE": [
        "Pie_Chart", "Treemap", "Sunburst_Diagram", "Nightingale_Chart"
    ]
}
```

## Troubleshooting

### Issue: "LLM Not Configured"

**Solution:**
```python
# Set API key in web UI sidebar or environment
export DEEPSEEK_API_KEY="your-key-here"
# Or configure in the settings panel (⚙ icon)
```

### Issue: Database Connection Failed

**Solution:**
```python
# Verify connection string format
# MySQL example:
connection_string = "mysql+pymysql://user:password@host:port/database"

# Test connection
import pymysql
conn = pymysql.connect(
    host=os.getenv('DB_HOST'),
    user=os.getenv('DB_USER'),
    password=os.getenv('DB_PASS'),
    database=os.getenv('DB_NAME')
)
conn.close()
```

### Issue: Charts Not Displaying

**Solution:**
```bash
# Charts are saved locally in:
./outputs/charts/

# Check browser console for errors
# Verify Plotly.js is loaded
# Clear browser cache if needed
```

### Issue: Slow Query Performance

**Solution:**
```python
# Add database indexes on frequently queried columns
# Example for MySQL:
"""
CREATE INDEX idx_order_date ON orders(order_date);
CREATE INDEX idx_customer_id ON orders(customer_id);
"""

# Or use /sql command to optimize query manually
```

### Issue: Incorrect SQL Generation

**Solution:**
```python
# Provide more context in natural language query
# Bad: "sales by region"
# Good: "total sales amount grouped by region for 2024"

# Or use /sql command to write SQL directly
/sql SELECT region, SUM(amount) FROM sales WHERE YEAR(date) = 2024 GROUP BY region
```

### Issue: Memory Error on Large Files

**Solution:**
```python
# For large CSVs, use chunked reading
import pandas as pd
chunks = pd.read_csv('large_file.csv', chunksize=10000)
for chunk in chunks:
    # Process each chunk
    pass

# Or import to SQLite/database first, then query
```

## Advanced Usage

### Custom Chart Configuration

```python
# Modify chart templates in modules/chart_generator.py
def create_custom_chart(data, chart_type):
    import plotly.graph_objects as go
    
    fig = go.Figure()
    # Custom Plotly configuration
    fig.update_layout(
        template="plotly_white",
        title_font_size=20,
        # Add custom styling
    )
    return fig
```

### Extend Analysis Functions

```python
# Add custom analysis in modules/analyzer.py
def custom_analysis(df, params):
    """
    Custom statistical analysis
    """
    from scipy import stats
    
    result = stats.ttest_ind(
        df[params['group1']], 
        df[params['group2']]
    )
    
    return {
        "statistic": result.statistic,
        "p_value": result.pvalue,
        "interpretation": "Significant" if result.pvalue < 0.05 else "Not significant"
    }
```

### Streaming Response Handler

```python
# Handle SSE stream in custom integration
import requests

def stream_analysis(query):
    response = requests.get(
        'http://localhost:5001/api/analyze',
        params={'query': query},
        stream=True
    )
    
    for line in response.iter_lines():
        if line:
            decoded = line.decode('utf-8')
            if decoded.startswith('data: '):
                data = decoded[6:]  # Remove 'data: ' prefix
                print(data)  # Process each streaming chunk
```

## Environment Variables Reference

```bash
# LLM Configuration
DEEPSEEK_API_KEY=your_deepseek_key
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key

# Database Credentials
DB_HOST=localhost
DB_PORT=3306
DB_USER=your_username
DB_PASS=your_password
DB_NAME=your_database

# Google Sheets (if using)
GOOGLE_SHEETS_CREDENTIALS_PATH=/path/to/credentials.json

# Server Configuration
FLASK_PORT=5001
FLASK_ENV=production
```

## Resources

- **Repository**: https://github.com/Zafer-Liu/Data-Analysis-Agent
- **Documentation**: See README.md and Information/ directory
- **MCP Tutorial**: Information/MCP_tutorial.md
- **Knowledge Base Guide**: Information/repository_tutorial.md
- **Version History**: Information/Version_Update_Log.md
