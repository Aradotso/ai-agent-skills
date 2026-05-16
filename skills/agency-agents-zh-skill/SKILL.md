---
name: agency-agents-zh-skill
description: 使用 agency-agents-zh 中文 AI 智能体角色库，为 AI 编程工具提供 215 个即插即用的专家角色
triggers:
  - 如何安装 agency-agents-zh 智能体
  - 帮我配置中文 AI 角色库
  - 使用 agency-agents-zh 专家角色
  - 安装小红书运营智能体
  - 配置 OpenClaw 中文智能体
  - agency-orchestrator 多智能体编排
  - 如何使用中国市场原创智能体
  - 为 Cursor 安装中文 AI 专家角色
---

# agency-agents-zh Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

agency-agents-zh 是一个包含 **215 个即插即用 AI 专家角色**的中文智能体库，覆盖工程、设计、营销、产品、游戏、安全、金融等 18 个部门。支持 Claude Code、Cursor、Copilot、OpenClaw 等 17 种 AI 编程工具。除了完整翻译上游 165 个角色外，新增了 50 个中国市场原创智能体（小红书、抖音、微信、飞书、钉钉、跨境电商等）。

## 核心特性

- **215 个专家角色**：每个智能体有明确的身份、工作流程和交付物
- **17 种工具支持**：Claude Code、Cursor、Copilot、OpenClaw、Gemini CLI、Qwen、Codex 等
- **50 个中国市场原创**：小红书运营、抖音种草、微信小程序、飞书集成、钉钉开发等
- **18 个专业部门**：工程、设计、营销、产品、游戏、安全、金融、数据等
- **自动化编排**：配合 Agency Orchestrator 实现多智能体协作

## 安装方法

### 方式一：一键安装脚本

```bash
# 克隆仓库
git clone https://github.com/jnMetaCode/agency-agents-zh.git
cd agency-agents-zh

# 自动检测已安装工具并安装
./scripts/install.sh

# 指定工具安装
./scripts/install.sh --tool openclaw       # OpenClaw（推荐）
./scripts/install.sh --tool claude-code    # Claude Code
./scripts/install.sh --tool cursor         # Cursor
./scripts/install.sh --tool copilot        # GitHub Copilot
./scripts/install.sh --tool gemini-cli     # Gemini CLI
./scripts/install.sh --tool qwen           # Qwen Code
./scripts/install.sh --tool codex          # Codex CLI
./scripts/install.sh --tool aider          # Aider
./scripts/install.sh --tool windsurf       # Windsurf
./scripts/install.sh --tool hermes         # Hermes Agent
```

### 方式二：OpenClaw 专用安装（推荐）

OpenClaw 使用三文件结构：`SOUL.md`（身份人设）+ `AGENTS.md`（业务能力）+ `IDENTITY.md`（简介）

```bash
# 第一步：转换为 OpenClaw 格式
./scripts/convert.sh --tool openclaw

# 第二步：安装到 OpenClaw 目录
./scripts/install.sh --tool openclaw

# 智能体文件会安装到：~/.openclaw/souls/
# 重启 OpenClaw 网关后生效
```

### 方式三：手动安装（Claude Code / Copilot）

```bash
# Claude Code
cp -r marketing/*.md ~/.claude/agents/
cp -r engineering/*.md ~/.claude/agents/
cp -r design/*.md ~/.claude/agents/

# GitHub Copilot
cp -r marketing/*.md ~/.github/copilot/agents/
cp -r engineering/*.md ~/.github/copilot/agents/

# Cursor
cp -r marketing/*.md ~/.cursor/agents/
```

## 使用智能体

### 在 AI 编程工具中激活

安装后，在任意 AI 编程工具中通过自然语言激活：

```
激活前端开发者模式，帮我构建一个 React 组件
```

```
切换到小红书运营专家角色，帮我策划一个产品种草方案
```

```
使用安全工程师角色审查这段代码的安全漏洞
```

### 查看可用智能体列表

```bash
# 列出所有智能体
ls -R engineering/ design/ marketing/ product/ game/ security/ finance/ data/

# 查看工程类智能体
ls engineering/
```

## Agency Orchestrator 多智能体编排

Agency Orchestrator 让多个智能体协作完成复杂任务。

### 安装 Agency Orchestrator

```bash
npm install -g agency-orchestrator
```

### 基本使用

```bash
# 一句话描述需求，自动选角+编排+执行
ao compose "帮我写一篇关于 AI Agent 的深度分析文章" --run

# 输出：
# 🎭 自动选角 → 叙事学家 + 心理学家 + 内容创作者
# 📊 自动编排 → DAG 工作流，并行执行
# ✅ 自动交付 → 完整成果
```

### 使用预设模板

```bash
# 查看可用模板
ao templates

# 使用开发类模板
ao run --template web-app-mvp --params "项目名称=电商平台,技术栈=Next.js"

# 使用营销类模板
ao run --template social-media-campaign --params "产品=智能手表,平台=小红书"
```

### YAML 工作流配置

创建 `workflow.yaml`：

```yaml
name: 小红书产品种草全流程
description: 从调研到投放的完整流程

agents:
  - id: researcher
    role: marketing/marketing-xiaohongshu-operator.md
    task: 分析目标用户画像和竞品内容策略
    
  - id: content_creator
    role: marketing/marketing-content-creator.md
    task: 创作 5 篇种草笔记（3 图文 + 2 视频）
    depends_on: [researcher]
    
  - id: kol_matcher
    role: marketing/marketing-xiaohongshu-operator.md
    task: 推荐 10 位匹配 KOL 及报价预估
    depends_on: [researcher]
    
  - id: ad_planner
    role: marketing/marketing-ad-campaign-manager.md
    task: 制定信息流投放计划和预算分配
    depends_on: [content_creator, kol_matcher]

config:
  llm: deepseek  # 或 claude, gemini, openai
  parallel: true # 无依赖步骤并行执行
  output_dir: ./output
```

运行工作流：

```bash
ao run workflow.yaml
```

### 断点续跑

```bash
# 工作流失败后，单独重跑某个步骤
ao resume workflow.yaml --step content_creator

# 从某个步骤开始继续
ao resume workflow.yaml --from ad_planner
```

## 核心智能体示例

### 1. 小红书运营专家

**文件**: `marketing/marketing-xiaohongshu-operator.md`

**使用场景**：
```
切换到小红书运营专家，帮我策划一款智能手表的种草方案
```

**交付物**：
- 用户画像分析
- 内容选题矩阵（5W1H）
- 笔记范例（图文+视频脚本）
- KOL 合作清单及报价
- 数据监控仪表盘

### 2. 上位机工程师

**文件**: `engineering/engineering-pc-host-engineer.md`

**使用场景**：
```
激活上位机工程师角色，帮我开发一个工业检测设备的 Qt 界面
```

**核心能力**：
- Qt/QML UI 开发
- QSerialPort/QModbusTCP 通信
- QChart 实时曲线
- SQLite 数据存储
- 多语言（中英文）

**代码示例**（自动生成）：
```cpp
// ModbusManager.h
class ModbusManager : public QObject {
    Q_OBJECT
public:
    explicit ModbusManager(QObject *parent = nullptr);
    bool connectDevice(const QString &ip, int port);
    
signals:
    void dataReceived(quint16 registerAddr, quint16 value);
    
private:
    QModbusTcpClient *modbusClient;
};
```

### 3. 微信小程序开发者

**文件**: `engineering/engineering-wechat-mini-program-developer.md`

**使用场景**：
```
使用微信小程序开发者角色，帮我构建一个电商小程序
```

**核心能力**：
- WXML/WXSS 组件化开发
- 微信支付集成
- 云开发数据库
- 分包加载优化
- 用户授权流程

**代码示例**：
```javascript
// pages/product/product.js
Page({
  data: {
    product: null,
    canvasId: 'shareCanvas'
  },
  
  onLoad(options) {
    this.loadProduct(options.id);
  },
  
  async loadProduct(id) {
    const db = wx.cloud.database();
    const res = await db.collection('products').doc(id).get();
    this.setData({ product: res.data });
  },
  
  async onBuyNow() {
    const res = await wx.cloud.callFunction({
      name: 'createOrder',
      data: { productId: this.data.product._id }
    });
    
    wx.requestPayment({
      ...res.result.payment,
      success: () => wx.navigateTo({ url: '/pages/order/order' })
    });
  }
});
```

### 4. 飞书集成开发工程师

**文件**: `engineering/engineering-feishu-integration-developer.md`

**使用场景**：
```
切换到飞书集成开发工程师，帮我开发一个审批流机器人
```

**核心能力**：
- 飞书机器人开发
- 审批流接口
- 多维表格操作
- 飞书文档 API
- 事件订阅

**代码示例**：
```javascript
// server.js
const express = require('express');
const app = express();

app.post('/webhook/approval', async (req, res) => {
  const { event } = req.body;
  
  if (event.type === 'approval_instance.approved') {
    const instanceCode = event.instance_code;
    
    // 更新多维表格
    await fetch('https://open.feishu.cn/open-apis/bitable/v1/apps/{app_token}/tables/{table_id}/records', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.FEISHU_ACCESS_TOKEN}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        fields: {
          '审批状态': '已通过',
          '审批时间': new Date().toISOString()
        }
      })
    });
    
    // 发送通知
    await sendMessage(event.user_id, '您的审批已通过！');
  }
  
  res.json({ code: 0 });
});
```

### 5. 前端开发者

**文件**: `engineering/engineering-frontend-developer.md`

**使用场景**：
```
激活前端开发者模式，帮我构建一个现代化的 React Dashboard
```

**核心能力**：
- React/Vue 组件开发
- 响应式设计
- 性能优化
- 状态管理（Redux/Zustand）
- 类型安全（TypeScript）

**代码示例**：
```tsx
// components/Dashboard.tsx
import { useEffect, useState } from 'react';
import { Card, Chart } from '@/components/ui';

interface DashboardData {
  users: number;
  revenue: number;
  trend: Array<{ date: string; value: number }>;
}

export function Dashboard() {
  const [data, setData] = useState<DashboardData | null>(null);
  
  useEffect(() => {
    fetch('/api/dashboard')
      .then(res => res.json())
      .then(setData);
  }, []);
  
  if (!data) return <div>加载中...</div>;
  
  return (
    <div className="grid grid-cols-3 gap-4">
      <Card title="总用户数">
        <span className="text-3xl font-bold">{data.users}</span>
      </Card>
      <Card title="总收入">
        <span className="text-3xl font-bold">¥{data.revenue}</span>
      </Card>
      <Card title="增长趋势" className="col-span-3">
        <Chart data={data.trend} type="line" />
      </Card>
    </div>
  );
}
```

## 智能体部门分类

### 工程部（Engineering）
33 个智能体，涵盖：
- 前端/后端/全栈开发
- AI/ML 工程
- DevOps/SRE
- 安全工程
- 嵌入式/物联网
- 上位机/工控
- 微信/飞书/钉钉集成

### 设计部（Design）
18 个智能体，涵盖：
- UI/UX 设计
- 品牌设计
- 视觉叙事
- 无障碍设计
- AI 图像生成

### 营销部（Marketing）
40 个智能体，涵盖：
- 小红书/抖音/B站运营
- SEO/SEM
- 内容创作
- 社交媒体
- 跨境电商
- 广告投放

### 产品部（Product）
22 个智能体，涵盖：
- 产品管理
- 需求分析
- 用户研究
- 数据驱动决策
- A/B 测试

### 其他部门
- 游戏（Game）：15 个
- 安全（Security）：12 个
- 金融（Finance）：10 个
- 数据（Data）：18 个
- 运营（Operations）：14 个
- 客服（Support）：8 个
- 法务（Legal）：6 个
- HR（Human Resources）：9 个

## 配置与环境变量

### Agency Orchestrator 配置

创建 `~/.agency-orchestrator/config.yaml`：

```yaml
# LLM 配置
llm:
  provider: deepseek  # deepseek, claude, openai, gemini
  api_key_env: DEEPSEEK_API_KEY  # 环境变量名称
  model: deepseek-chat
  temperature: 0.7
  max_tokens: 4000

# 智能体路径
agents:
  base_dir: ~/.openclaw/souls  # 或 ~/.claude/agents
  auto_discover: true

# 输出配置
output:
  default_dir: ./agency-output
  format: markdown  # markdown, json, html
  include_metadata: true

# 并行执行配置
execution:
  max_parallel: 3
  timeout: 300  # 秒
  retry: 2
```

### 环境变量设置

```bash
# .env 文件
DEEPSEEK_API_KEY=your_deepseek_api_key
CLAUDE_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key
GEMINI_API_KEY=your_gemini_api_key

# 飞书/钉钉集成
FEISHU_APP_ID=your_feishu_app_id
FEISHU_APP_SECRET=your_feishu_app_secret
DINGTALK_APP_KEY=your_dingtalk_app_key
DINGTALK_APP_SECRET=your_dingtalk_app_secret

# 微信小程序
WECHAT_APP_ID=your_wechat_app_id
WECHAT_APP_SECRET=your_wechat_app_secret
```

## 常见使用模式

### 模式一：单智能体咨询

```
切换到安全工程师角色
请审查以下代码的安全问题：
[粘贴代码]
```

### 模式二：智能体切换

```
激活前端开发者
[完成 UI 开发后]
现在切换到后端架构师，设计对应的 API
```

### 模式三：多智能体协作（通过 Agency Orchestrator）

```bash
# 一句话描述复杂需求
ao compose "开发一个小红书运营工具，包含内容创作、数据分析、KOL 推荐功能" --run

# 自动选角：
# - 小红书运营专家（需求分析）
# - 前端开发者（UI 实现）
# - 后端架构师（API 设计）
# - 数据工程师（数据管线）
```

### 模式四：使用预设模板

```bash
# 电商小程序全栈开发
ao run --template wechat-mini-program-ecommerce

# 工业上位机开发
ao run --template industrial-hmi-qt

# 小红书营销全流程
ao run --template xiaohongshu-full-campaign
```

## 故障排除

### 问题 1：智能体未加载

**症状**：在 AI 工具中无法激活智能体

**解决方案**：
```bash
# 检查安装路径
ls ~/.claude/agents/
ls ~/.openclaw/souls/

# 重新安装
./scripts/install.sh --tool claude-code --force

# 重启 AI 工具
```

### 问题 2：OpenClaw 格式转换失败

**症状**：`convert.sh` 报错

**解决方案**：
```bash
# 确保有执行权限
chmod +x scripts/convert.sh

# 手动指定输出目录
./scripts/convert.sh --tool openclaw --output ./converted

# 检查依赖
which python3  # 需要 Python 3.7+
```

### 问题 3：Agency Orchestrator 无法连接 LLM

**症状**：`ao compose` 报 API 错误

**解决方案**：
```bash
# 检查环境变量
echo $DEEPSEEK_API_KEY

# 测试连接
ao test-connection --provider deepseek

# 切换到免费 LLM
ao compose "任务描述" --llm gemini-cli --run
```

### 问题 4：中文乱码

**症状**：智能体输出乱码

**解决方案**：
```bash
# 设置终端编码
export LANG=zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8

# 检查文件编码
file -i marketing/marketing-xiaohongshu-operator.md
# 应显示：charset=utf-8
```

### 问题 5：工作流执行超时

**症状**：Agency Orchestrator 任务超时

**解决方案**：
```yaml
# 在 workflow.yaml 中增加超时时间
config:
  timeout: 600  # 从 300 秒增加到 600 秒
  
# 或减少并行度
config:
  max_parallel: 1  # 串行执行，避免 API 限流
```

## 高级技巧

### 自定义智能体

创建 `custom/my-agent.md`：

```markdown
---
name: 我的自定义专家
role: 特定领域专家
department: 自定义部门
---

# 角色定义

你是一位...

## 核心能力
1. ...
2. ...

## 工作流程
- 步骤 1：...
- 步骤 2：...

## 交付物
- [ ] ...
```

安装：
```bash
cp custom/my-agent.md ~/.claude/agents/
```

### 组合使用智能体

```yaml
# combo-workflow.yaml
name: 全栈开发+运营组合
agents:
  - role: engineering/engineering-frontend-developer.md
  - role: engineering/engineering-backend-architect.md
  - role: marketing/marketing-xiaohongshu-operator.md
  - role: design/design-ui-designer.md
```

### 批量转换智能体

```bash
# 转换所有智能体为 Cursor 格式
for dept in engineering design marketing product; do
  ./scripts/convert.sh --tool cursor --input $dept --output ./cursor-agents/$dept
done
```

## 资源链接

- **GitHub 仓库**: https://github.com/jnMetaCode/agency-agents-zh
- **Agency Orchestrator**: https://github.com/jnMetaCode/agency-orchestrator
- **上游英文版**: https://github.com/msitarzewski/agency-agents
- **配套书籍**: https://book.aibuzhiyu.com/（AI 编程实战方法论）

## 支持的 AI 工具完整列表

1. **OpenClaw** ⭐ 推荐（多智能体协作）
2. **Claude Code**（Anthropic 官方）
3. **Cursor**（智能 IDE）
4. **GitHub Copilot**
5. **Gemini CLI**（Google）
6. **Qwen Code**（阿里云）
7. **Codex CLI**（OpenAI）
8. **Aider**（开源 AI 编程助手）
9. **Windsurf**
10. **Antigravity**
11. **Kiro**（Amazon）
12. **Trae**
13. **OpenCode**
14. **DeerFlow 2.0**（字节跳动）
15. **WorkBuddy**（腾讯）
16. **Hermes Agent**（NousResearch）
17. **Qoder**

---

**快速开始**：克隆仓库 → 运行 `./scripts/install.sh` → 在 AI 工具中激活智能体 → 开始使用 215 个专家角色！
