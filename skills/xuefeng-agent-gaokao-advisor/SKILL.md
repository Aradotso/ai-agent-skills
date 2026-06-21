---
name: xuefeng-agent-gaokao-advisor
description: AI-powered Chinese college admission advisor with 240k+ real admission data records, multi-round dialogue, major filtering, and Zhang Xuefeng methodology
triggers:
  - "help me set up xuefeng agent for gaokao counseling"
  - "how do I configure the xuefeng college admission advisor"
  - "integrate xuefeng agent database for college recommendations"
  - "build a gaokao volunteer advisor with xuefeng agent"
  - "query admission data using xuefeng agent"
  - "set up deepseek api for xuefeng agent"
  - "how does xuefeng agent filter majors and universities"
  - "troubleshoot xuefeng agent database loading issues"
---

# xuefeng-agent-gaokao-advisor

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

**xuefeng-agent** is an AI-powered Chinese college admission counseling system that combines:
- **420,000+ official admission records** from 24 provinces (2024-2025 data)
- **Zhang Xuefeng's methodology** (8 books + 61 video courses on major/university selection)
- **Multi-round conversational AI** with context memory
- **Major-specific filtering** (computer science, medicine, law, etc.)
- **Web search integration** (Tavily AI search or Baidu fallback)
- **Browser-based UI** with conversation history and dark mode

The agent automatically extracts student info (province, score rank, preferences) from natural language, queries the local SQLite database, performs web searches for latest data, and provides tailored "reach/match/safety" school recommendations.

## Installation

### Prerequisites
- Python 3.10+
- Windows (`.bat` launcher included), macOS/Linux (manual terminal launch)

### Setup

```bash
# Clone or download the project
git clone https://github.com/ziqihe10-droid/xuefeng-agent.git
cd xuefeng-agent

# Install dependencies (auto-installed on first run, or manual):
pip install flask==2.3.3 openai==1.3.5 tavily-python==0.3.0

# On first run, the compressed database auto-extracts:
# admission_clean.db.gz (27.6 MB) → admission_clean.db (143 MB)
```

### Launch

**Windows:**
```bash
# Double-click 启动.bat
# Or run manually:
python server.py
```

**macOS/Linux:**
```bash
python server.py
```

The server starts on `http://localhost:5000` and opens automatically in your browser.

## Architecture

```
User Query → AI Extracts (province, rank, majors)
    ↓
Local DB Query (admission_clean.db) + Major Keyword Filtering
    ↓
Web Search (Tavily or Baidu) for latest university/major info
    ↓
LLM Synthesizes [DB data + Web data] → Structured Response
    ↓
Browser UI displays with source attribution ([DB] or [Web])
```

## Core Components

### 1. Database Schema

**Table: `admission_data`**
```sql
CREATE TABLE admission_data (
    province TEXT,          -- 省份 (e.g., "浙江", "河北")
    year INTEGER,           -- 年份 (2024/2025)
    university TEXT,        -- 学校名称
    major TEXT,             -- 专业名称
    score INTEGER,          -- 最低分
    rank INTEGER,           -- 位次
    batch TEXT              -- 批次 (本科一批/本科二批/专科)
);
CREATE INDEX idx_province_year_rank ON admission_data(province, year, rank);
CREATE INDEX idx_major ON admission_data(major);
```

### 2. Server Architecture (`server.py`)

```python
from flask import Flask, request, jsonify, send_file
import sqlite3
import os
import openai
from tavily import TavilyClient

app = Flask(__name__)
DB_PATH = "admission_clean.db"

# Auto-extract compressed DB on first run
if not os.path.exists(DB_PATH) and os.path.exists(f"{DB_PATH}.gz"):
    import gzip
    import shutil
    with gzip.open(f"{DB_PATH}.gz", 'rb') as f_in:
        with open(DB_PATH, 'wb') as f_out:
            shutil.copyfileobj(f_in, f_out)

# Database query helper
def query_db(province, rank, year=2025, majors=None, limit=100):
    conn = sqlite3.connect(DB_PATH)
    cursor = conn.cursor()
    
    # Base query with rank-based filtering
    query = """
        SELECT university, major, score, rank, batch
        FROM admission_data
        WHERE province = ? AND year = ? 
        AND rank BETWEEN ? AND ?
    """
    params = [province, year, rank - 5000, rank + 5000]
    
    # Add major keyword filtering
    if majors:
        major_conditions = " OR ".join(["major LIKE ?" for _ in majors])
        query += f" AND ({major_conditions})"
        params.extend([f"%{m}%" for m in majors])
    
    query += " ORDER BY rank ASC LIMIT ?"
    params.append(limit)
    
    cursor.execute(query, params)
    results = cursor.fetchall()
    conn.close()
    
    return [
        {
            "university": r[0],
            "major": r[1],
            "score": r[2],
            "rank": r[3],
            "batch": r[4]
        }
        for r in results
    ]
```

### 3. AI Extraction Pattern

```python
# Prompt for extracting structured data from user input
extraction_prompt = """
你是高考志愿填报助手。从用户消息中提取：
1. 省份（浙江/河北/山东等，识别别称如"川籍"→四川）
2. 位次/排名（"一万三"→13000，"5k"→5000）
3. 专业偏好（计算机、医学、法学等关键词列表）
4. 其他限制（不接受调剂、只要985等）

以JSON返回：
{
  "province": "浙江",
  "rank": 10500,
  "majors": ["计算机", "电子"],
  "constraints": ["不接受调剂"]
}
"""

# OpenAI-compatible API call
def extract_info(user_message, api_key, base_url, model):
    client = openai.OpenAI(api_key=api_key, base_url=base_url)
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": extraction_prompt},
            {"role": "user", "content": user_message}
        ],
        temperature=0.1
    )
    import json
    return json.loads(response.choices[0].message.content)
```

### 4. Web Search Integration

```python
def web_search(query, tavily_key=None):
    """
    Search for latest university/major info
    Falls back to Baidu if Tavily key not provided
    """
    if tavily_key:
        # Tavily AI search (more accurate)
        client = TavilyClient(api_key=tavily_key)
        response = client.search(
            query=query,
            search_depth="advanced",
            max_results=5
        )
        return response.get("results", [])
    else:
        # Baidu fallback (basic scraping)
        import requests
        from bs4 import BeautifulSoup
        url = f"https://www.baidu.com/s?wd={query}"
        response = requests.get(url, headers={"User-Agent": "Mozilla/5.0"})
        soup = BeautifulSoup(response.text, 'html.parser')
        # Extract top 3 result snippets
        results = []
        for item in soup.select('.result')[:3]:
            results.append({
                "title": item.select_one('h3').text,
                "snippet": item.select_one('.c-abstract').text
            })
        return results
```

## Configuration

### API Settings (Browser UI)

The UI stores settings in `localStorage`:

```javascript
// Stored keys (in browser):
{
  "apiKey": "sk-xxxxx",              // OpenAI-compatible API key
  "baseUrl": "https://api.deepseek.com/v1",  // Base URL
  "model": "deepseek-chat",          // Model name
  "tavilyKey": "tvly-xxxxx"          // Optional Tavily search key
}
```

**Recommended Models:**
- **DeepSeek** (`deepseek-chat`): Most cost-effective, free tier available
- **Qwen** (`qwen-max`): Good Chinese understanding
- **GLM** (`glm-4`): Stable domestic option
- **GPT-4o**: Premium option (requires VPN in China)

### Environment Variables (Optional)

```bash
# For programmatic use (bypasses browser UI)
export OPENAI_API_KEY="sk-xxxxx"
export OPENAI_BASE_URL="https://api.deepseek.com/v1"
export TAVILY_API_KEY="tvly-xxxxx"  # Optional, enables AI search
```

## Usage Examples

### Example 1: Basic Query

**User Input:**
```
浙江考生，655分，位次10500，想学计算机和电子类专业
```

**AI Extracts:**
```json
{
  "province": "浙江",
  "rank": 10500,
  "majors": ["计算机", "电子"],
  "constraints": []
}
```

**Database Query:**
```python
results = query_db(
    province="浙江",
    rank=10500,
    year=2025,
    majors=["计算机", "电子"],
    limit=100
)
# Returns ~50-80 matching schools with CS/EE majors
```

**Response Structure:**
```
🤖 雪峰Agent：

冲（位次高于你）：
· 湖南大学 计算机科学与技术  662分/7354位 [DB]
· 西北工业大学 电子信息类    658分/7351位 [DB]

稳（位次匹配）：
· 南京理工大学 计算机类      651分/10529位 [DB]
· 河海大学 计算机类          654分/10586位 [DB]

保（稳录）：
· 深圳大学 电子信息工程      648分/13653位 [DB]
```

### Example 2: Programmatic Integration

```python
import sqlite3
import openai

# Setup
client = openai.OpenAI(
    api_key=os.getenv("OPENAI_API_KEY"),
    base_url=os.getenv("OPENAI_BASE_URL")
)
db_conn = sqlite3.connect("admission_clean.db")

# Step 1: Extract info
extraction_response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": extraction_prompt},
        {"role": "user", "content": "山东考生排名25000，想学医"}
    ]
)
info = json.loads(extraction_response.choices[0].message.content)
# → {"province": "山东", "rank": 25000, "majors": ["医学", "临床"]}

# Step 2: Query database
cursor = db_conn.cursor()
cursor.execute("""
    SELECT university, major, score, rank
    FROM admission_data
    WHERE province = ? AND year = 2025
    AND rank BETWEEN ? AND ?
    AND (major LIKE '%医学%' OR major LIKE '%临床%')
    ORDER BY rank ASC LIMIT 50
""", [info["province"], info["rank"] - 3000, info["rank"] + 3000])

schools = cursor.fetchall()

# Step 3: Generate response
context = f"数据库查询结果：\n{schools}\n\n请按冲稳保分类推荐"
final_response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": "你是张雪峰风格的志愿填报顾问"},
        {"role": "user", "content": context}
    ]
)
print(final_response.choices[0].message.content)
```

### Example 3: Custom Major Filtering

```python
def filter_by_major_group(results, group):
    """
    Filter results by predefined major groups
    """
    groups = {
        "计算机": ["计算机", "软件", "人工智能", "大数据", "网络安全"],
        "电子": ["电子", "通信", "微电子", "集成电路", "物联网"],
        "医学": ["临床", "口腔", "中医", "药学", "护理"],
        "金融": ["金融", "会计", "经济", "财务", "投资"]
    }
    
    keywords = groups.get(group, [])
    return [
        r for r in results
        if any(kw in r["major"] for kw in keywords)
    ]

# Usage
all_results = query_db("浙江", 10000, majors=None)
cs_only = filter_by_major_group(all_results, "计算机")
```

## Key API Endpoints

### `POST /chat`

Main conversation endpoint.

**Request:**
```json
{
  "message": "河北理科12000名，想学电子信息",
  "conversationId": "conv_1234",
  "apiKey": "sk-xxxxx",
  "baseUrl": "https://api.deepseek.com/v1",
  "model": "deepseek-chat",
  "tavilyKey": "tvly-xxxxx"  // Optional
}
```

**Response:**
```json
{
  "reply": "🤖 根据您的位次...\n\n冲：北京邮电大学...",
  "conversationId": "conv_1234"
}
```

### `GET /conversations`

Retrieve all saved conversations (stored server-side in memory).

**Response:**
```json
{
  "conversations": [
    {
      "id": "conv_1234",
      "title": "浙江655分志愿咨询",
      "updatedAt": "2025-06-15T10:30:00Z",
      "messages": [...]
    }
  ]
}
```

## Common Patterns

### Pattern 1: Multi-Round Consultation

```python
# The agent remembers context across rounds
messages = [
    {"role": "user", "content": "浙江考生，655分"},
    {"role": "assistant", "content": "请问您的位次是多少？"},
    {"role": "user", "content": "10500左右"},
    {"role": "assistant", "content": "想学什么专业方向？"},
    {"role": "user", "content": "计算机或电子"},
    # Final response uses all accumulated context
]
```

### Pattern 2: Province-Specific Logic

```python
# Different provinces have different志愿填报 rules
province_rules = {
    "浙江": {"志愿数": 80, "模式": "专业+院校"},
    "山东": {"志愿数": 96, "模式": "专业+院校"},
    "河北": {"志愿数": 96, "模式": "专业+院校"},
    "北京": {"志愿数": 30, "模式": "院校专业组"},
    # Traditional 高考 provinces:
    "河南": {"志愿数": 6, "模式": "院校+专业", "parallel": 9}
}

def get_advice_structure(province):
    rule = province_rules.get(province, {"志愿数": 6})
    if rule["志愿数"] >= 80:
        return "建议填满全部志愿，前1/3冲刺，中1/3稳妥，后1/3保底"
    else:
        return "建议2冲2稳2保的梯度结构"
```

### Pattern 3: Data Source Attribution

```python
def format_school_entry(school, source):
    """
    Tag each recommendation with data source
    """
    tag = "[DB]" if source == "database" else "[联网]"
    return f"· {school['university']} {school['major']}  {school['score']}分/{school['rank']}位 {tag}"

# In final response:
# · 南京大学 计算机类  670分/3200位 [DB]
# · 上海交大 人工智能  (最新数据:预估线665分) [联网]
```

## Troubleshooting

### Issue: Database not found

**Symptom:**
```
sqlite3.OperationalError: unable to open database file
```

**Solution:**
```bash
# Check if .gz file exists
ls -lh admission_clean.db.gz

# Manual extraction if auto-extract failed
gunzip admission_clean.db.gz

# Verify extraction
ls -lh admission_clean.db  # Should be ~143 MB
```

### Issue: Empty query results

**Symptom:**
Agent says "没有找到匹配的学校"

**Causes & Fixes:**
```python
# 1. Rank out of range (no schools in ±5000 window)
# Solution: Widen search range
params = [province, year, rank - 10000, rank + 10000]

# 2. Major keywords too strict
# Solution: Use broader terms
majors = ["计算机"]  # ✅ Broad
majors = ["计算机科学与技术(实验班)"]  # ❌ Too specific

# 3. Wrong province name
# Solution: Normalize province names
province_map = {
    "川": "四川", "京": "北京", "沪": "上海",
    "浙": "浙江", "苏": "江苏", "粤": "广东"
}
province = province_map.get(input_province, input_province)
```

### Issue: API timeout

**Symptom:**
```
openai.APITimeoutError: Request timed out
```

**Solution:**
```python
# Increase timeout for large context
client = openai.OpenAI(
    api_key=api_key,
    base_url=base_url,
    timeout=60.0  # Default is 10s
)

# Or use streaming for long responses
stream = client.chat.completions.create(
    model=model,
    messages=messages,
    stream=True
)
for chunk in stream:
    print(chunk.choices[0].delta.content, end="")
```

### Issue: Incorrect rank extraction

**Symptom:**
"一万三" extracted as 13 instead of 13000

**Solution:**
```python
# Enhanced extraction prompt
"""
位次提取规则：
- "一万三" → 13000
- "5k" → 5000
- "两万五" → 25000
- "123名" → 123
"""

# Add validation
def validate_rank(rank, province):
    # Sanity check: rank should be reasonable
    if rank < 100:  # Likely error (e.g., "13" instead of "13000")
        rank *= 1000
    max_ranks = {"浙江": 300000, "河北": 500000}
    if rank > max_ranks.get(province, 1000000):
        raise ValueError(f"Rank {rank} out of range for {province}")
    return rank
```

### Issue: Web search returns irrelevant results

**Symptom:**
Baidu fallback returns ads or unrelated content

**Solution:**
```python
# Use Tavily for better search (requires API key)
export TAVILY_API_KEY="tvly-xxxxx"

# Or refine Baidu query with site-specific search
query = f"site:edu.cn {university} {major} 2025 录取分数线"

# Filter results by domain whitelist
allowed_domains = [".edu.cn", "gaokao.chsi.com.cn", "eol.cn"]
filtered = [
    r for r in results
    if any(domain in r.get("url", "") for domain in allowed_domains)
]
```

## Knowledge Base Modules

The agent's system prompts include 17 knowledge modules:
1. **方法论** - Zhang Xuefeng's core strategies
2. **选科规则** - Subject selection for new高考
3. **专业解析** - 61 major categories detailed analysis
4. **学校联盟** - C9/985/211/双一流/行业特色高校
5. **考研趋势** - Graduate school planning
6. **就业数据** - Salary/employment rate by major
7. **专科策略** - Vocational college selection
8. **城市选择** - Regional development vs university tier trade-offs

Access in code:
```python
# Knowledge is embedded in system prompt
system_prompt = """
你是雪峰Agent，掌握以下知识体系：

【方法论】
1. 分数优先 vs 专业优先（根据位次段决策）
2. 冲稳保比例（新高考 1:1:1，老高考 2:2:2）
3. 调剂风险规避...

【学校联盟】
- C9: 清北+华五+哈工大西交
- 两电一邮: 电子科大/西电/北邮
- 国防七子: 北航/南航/哈工大...

【专业-就业映射】
计算机 → 互联网/AI → 应届30w+ (TOP20高校)
医学 → 三甲医院 → 规培3年后20w+
金融 → 券商/银行 → 硕士学历优先...
"""
```

## Advanced: Rebuilding Database

If you have raw Excel files from省考试院:

```bash
python rebuild_db.py
```

**Input format (`raw_data/浙江2025.xlsx`):**
```
| 学校 | 专业 | 最低分 | 最低位次 |
|------|------|--------|----------|
| 浙江大学 | 计算机类 | 680 | 1200 |
```

**Rebuild script logic:**
```python
import pandas as pd
import sqlite3

def rebuild_db():
    conn = sqlite3.connect("admission_clean.db")
    cursor = conn.cursor()
    
    # Create table
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS admission_data (
            province TEXT, year INTEGER, university TEXT,
            major TEXT, score INTEGER, rank INTEGER, batch TEXT
        )
    """)
    
    # Import from all Excel files
    for file in glob.glob("raw_data/*.xlsx"):
        province = extract_province_from_filename(file)
        df = pd.read_excel(file)
        df["province"] = province
        df.to_sql("admission_data", conn, if_exists="append", index=False)
    
    # Create indexes
    cursor.execute("CREATE INDEX idx_province_year_rank ON admission_data(province, year, rank)")
    conn.commit()
```

## License & Usage Rights

**AGPL v3** - Strong copyleft license:
- ✅ Personal/educational use: Free
- ✅ Fork & modify: Must keep attribution + stay AGPL v3
- ❌ Commercial use: Requires separate license from author
- ❌ Remove attribution: Violation regardless of use

**Author:** 贺子麒 (ziqihe10-droid)  
**Commercial licensing:** Contact via GitHub

---

**End of Skill Document**
