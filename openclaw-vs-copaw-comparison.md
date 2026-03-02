# OpenClaw vs CoPaw 深度对比调研报告

**调研时间**: 2026-03-02  
**调研项目**: 
- [openclaw/openclaw](https://github.com/openclaw/openclaw)
- [agentscope-ai/CoPaw](https://github.com/agentscope-ai/CoPaw)

---

## 📌 执行摘要

| 维度 | OpenClaw | CoPaw |
|------|----------|-------|
| **定位** | 全平台个人 AI 助手，强调本地优先、多通道集成 | 个人 AI 助手工作站，强调易用性和可扩展性 |
| **技术栈** | TypeScript/Node.js (主) + Swift/Kotlin | Python (主) + TypeScript (Console UI) |
| **代码规模** | ~106 万行 (5728 文件) | ~4.3 万行 (252 文件) |
| **社区规模** | 244,480 ⭐ / 47,264 🔱 | 4,042 ⭐ / 402 🔱 |
| **创建时间** | 2025-11-24 | 2026-02-24 |
| **许可证** | MIT | Apache 2.0 |
| **核心特色** | Canvas 可视化、Voice Wake、多通道、Skills | 本地模型、定时任务、Console UI、Skills |
| **安装方式** | `npm install -g openclaw@latest` | `pip install copaw` / 一键安装脚本 |
| **推荐场景** | 企业级、多通道、高级自动化 | 个人使用、快速部署、本地模型 |

---

## 1️⃣ 项目概览对比

### OpenClaw

```
🦞 OpenClaw — Personal AI Assistant
"EXFOLIATE! EXFOLIATE!"

Your own personal AI assistant. Any OS. Any Platform. The lobster way.
```

**核心特点**:
- 🌐 **全平台支持**: macOS, Linux, Windows (WSL2), iOS, Android
- 💬 **30+ 通信通道**: WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Teams, Matrix, 飞书等
- 🎨 **Canvas 可视化**: 智能体驱动的交互式工作区
- 🎤 **Voice Wake**: 语音唤醒和连续对话
- 🔧 **Skills 系统**: 可扩展的技能平台
- 🏠 **本地优先**: Gateway 运行在用户设备上

### CoPaw

```
🐾 CoPaw — Your Personal AI Assistant
"Works for you, grows with you."

Your Personal AI Assistant; easy to install, deploy on your own machine or on the cloud
```

**核心特点**:
- 🐍 **Python 优先**: 易于理解和扩展
- 🖥️ **Console UI**: 内置 Web 控制台
- 🤖 **本地模型**: 支持 llama.cpp 和 MLX
- ⏰ **Heartbeat**: 定时任务和提醒
- 🔌 **多通道**: 钉钉、飞书、QQ、Discord、iMessage 等
- 📦 **一键安装**: 自动处理 Python 环境

---

## 2️⃣ 架构对比

### OpenClaw 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        通信通道层                                │
│  WhatsApp │ Telegram │ Slack │ Discord │ Signal │ iMessage │... │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Gateway (控制平面)                          │
│              ws://127.0.0.1:18789                               │
│  ┌─────────────┬─────────────┬─────────────┬─────────────────┐ │
│  │   Sessions  │   Channels  │    Tools    │     Events      │ │
│  │   会话管理   │   通道管理   │   工具系统   │    事件系统      │ │
│  └─────────────┴─────────────┴─────────────┴─────────────────┘ │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Pi Agent   │  │   CLI       │  │  WebChat    │
│  (RPC 模式)  │  │  openclaw   │  │     UI      │
└─────────────┘  └─────────────┘  └─────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      设备节点 (Nodes)                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                      │
│  │  macOS   │  │   iOS    │  │ Android  │                      │
│  │ WKWebView│  │ WKWebView│  │ WebView  │                      │
│  │  Canvas  │  │  Canvas  │  │  Canvas  │                      │
│  └──────────┘  └──────────┘  └──────────┘                      │
└─────────────────────────────────────────────────────────────────┘
```

**架构特点**:
- **中心化 Gateway**: 单一 WebSocket 控制平面
- **分布式 Nodes**: 设备节点执行本地操作
- **多通道统一**: 所有通道通过 Gateway 路由
- **会话隔离**: 每个通道/群组独立会话

### CoPaw 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        通信通道层                                │
│  钉钉 │ 飞书 │ QQ │ Discord │ iMessage │ Console │ ...          │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Channel Manager                               │
│  ┌─────────────┬─────────────┬─────────────┬─────────────────┐ │
│  │  DingTalk   │   Feishu    │    QQ       │    Discord      │ │
│  └─────────────┴─────────────┴─────────────┴─────────────────┘ │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Agent Core                                  │
│  ┌─────────────┬─────────────┬─────────────┬─────────────────┐ │
│  │   Memory    │   Skills    │   Models    │    Crons        │ │
│  │   记忆系统   │   技能系统   │   模型管理   │    定时任务      │ │
│  └─────────────┴─────────────┴─────────────┴─────────────────┘ │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Console    │  │   CLI       │  │   Local     │
│  Web UI     │  │  copaw      │  │   Models    │
│  :8088      │  │             │  │ llama.cpp   │
└─────────────┘  └─────────────┘  └─────────────┘
```

**架构特点**:
- **模块化设计**: Channel Manager + Agent Core
- **Console 优先**: Web UI 作为主要交互界面
- **本地模型集成**: 原生支持本地 LLM
- **Python 生态**: 易于扩展和定制

---

## 3️⃣ 技术栈详细对比

### 编程语言

| 语言 | OpenClaw | CoPaw |
|------|----------|-------|
| **TypeScript** | 940,519 行 (5054 文件) - 88% | 2,993 行 (61 文件) - 7% |
| **Swift** | 89,735 行 (519 文件) - 8% | - |
| **Kotlin** | 22,796 行 (111 文件) - 2% | - |
| **Python** | 1,825 行 (11 文件) - <1% | 40,144 行 (190 文件) - 93% |
| **JavaScript** | 6,630 行 (19 文件) | 28 行 (1 文件) |
| **Go** | 1,840 行 (14 文件) | - |
| **总计** | ~106 万行 | ~4.3 万行 |

### 核心依赖

#### OpenClaw

```json
{
  "runtime": "Node.js ≥22",
  "packageManager": "npm/pnpm/bun",
  "keyDependencies": {
    "ws": "WebSocket 服务器",
    "express": "HTTP 服务器",
    "chokidar": "文件监听",
    "discord.js": Discord 通道",
    "grammY": "Telegram 通道",
    "@slack/bolt": "Slack 通道",
    "baileys": "WhatsApp 通道",
    "ws": "WebSocket 客户端"
  }
}
```

#### CoPaw

```python
{
  "runtime": "Python 3.10 ~ 3.14",
  "packageManager": "pip",
  "keyDependencies": {
    "agentscope": "AgentScope 框架",
    "dashscope": "阿里云模型 SDK",
    "dingtalk-sdk": "钉钉通道",
    "lark-oapi": "飞书通道",
    "discord.py": "Discord 通道",
    "llama-cpp-python": "本地模型 (可选)",
    "mlx": "Apple Silicon 本地模型 (可选)"
  }
}
```

### 客户端应用

| 平台 | OpenClaw | CoPaw |
|------|----------|-------|
| **macOS** | ✅ 原生菜单栏应用 (Swift) | ❌ 仅 CLI/Web |
| **iOS** | ✅ 原生应用 (Swift) | ❌ 无 |
| **Android** | ✅ 原生应用 (Kotlin) | ❌ 无 |
| **Web** | ✅ Control UI + WebChat | ✅ Console UI |
| **Desktop** | ❌ | ❌ |

---

## 4️⃣ 核心功能对比

### 通信通道

| 通道 | OpenClaw | CoPaw |
|------|----------|-------|
| **WhatsApp** | ✅ (Baileys) | ❌ |
| **Telegram** | ✅ (grammY) | ❌ |
| **Slack** | ✅ (Bolt) | ❌ |
| **Discord** | ✅ (discord.js) | ✅ (discord.py) |
| **Google Chat** | ✅ | ❌ |
| **Signal** | ✅ (signal-cli) | ❌ |
| **iMessage** | ✅ (BlueBubbles/原生) | ✅ |
| **钉钉** | ❌ | ✅ |
| **飞书** | ✅ | ✅ |
| **QQ** | ❌ | ✅ |
| **LINE** | ✅ | ❌ |
| **Matrix** | ✅ | ❌ |
| **Microsoft Teams** | ✅ | ❌ |
| **WebChat** | ✅ | ✅ (Console) |
| **总计** | **30+** | **~10** |

### AI 模型支持

| 特性 | OpenClaw | CoPaw |
|------|----------|-------|
| **云模型** | Anthropic, OpenAI, Google, etc. | DashScope, ModelScope, etc. |
| **本地模型** | ❌ | ✅ (llama.cpp, MLX) |
| **模型故障转移** | ✅ 自动 failover | ❌ |
| **OAuth 认证** | ✅ (OpenAI, Anthropic) | ❌ |
| **推荐模型** | Claude Opus 4.6 | Qwen3-4B (本地) |

### 特色功能

| 功能 | OpenClaw | CoPaw |
|------|----------|-------|
| **Canvas 可视化** | ✅ A2UI 声明式 UI | ❌ |
| **Voice Wake** | ✅ 语音唤醒 (macOS/iOS) | ❌ |
| **Talk Mode** | ✅ 连续语音对话 (Android) | ❌ |
| **浏览器控制** | ✅ 专用 Chrome/Chromium | ❌ |
| **定时任务** | ✅ Cron + Webhooks | ✅ Heartbeat |
| **Skills 系统** | ✅ ClawHub 注册表 | ✅ 工作区 Skills |
| **会话管理** | ✅ 多会话隔离 | ✅ 记忆系统 |
| **文件操作** | ✅ 读写编辑 | ✅ 读写编辑 |
| **代码执行** | ✅ bash (支持沙箱) | ✅ Python |
| **截图/录屏** | ✅ (macOS/iOS/Android) | ❌ |
| **位置服务** | ✅ (iOS/Android) | ❌ |
| **通知** | ✅ 系统通知 | ❌ |

---

## 5️⃣ 安装与部署对比

### OpenClaw

```bash
# 推荐安装
npm install -g openclaw@latest
openclaw onboard --install-daemon

# 从源码
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build

# Docker
docker run openclaw/openclaw:latest

# Nix (声明式配置)
nix profile install github:openclaw/nix-openclaw
```

**系统要求**:
- Node.js ≥22
- macOS: 13.0+
- iOS: 16.0+
- Android: 10.0+

### CoPaw

```bash
# pip 安装 (推荐)
pip install copaw
copaw init --defaults
copaw app

# 一键安装 (自动处理 Python)
curl -fsSL https://copaw.agentscope.io/install.sh | bash

# Docker
docker pull agentscope/copaw:latest
docker run -p 8088:8088 -v copaw-data:/app/working agentscope/copaw:latest

# ModelScope (云端)
https://modelscope.cn/studios/fork?target=AgentScope/CoPaw

# 阿里云 ECS 一键部署
https://computenest.console.aliyun.com/service/instance/create/...
```

**系统要求**:
- Python 3.10 ~ 3.14
- macOS / Linux / Windows
- 本地模型：llama.cpp (跨平台) 或 MLX (Apple Silicon)

---

## 6️⃣ 配置对比

### OpenClaw 配置

**位置**: `~/.openclaw/openclaw.json`

```json5
{
  "agent": {
    "model": "anthropic/claude-opus-4-6"
  },
  "channels": {
    "telegram": {
      "botToken": "123456:ABCDEF",
      "allowFrom": ["@user1", "@user2"]
    },
    "discord": {
      "token": "1234abcd",
      "dmPolicy": "pairing"
    }
  },
  "gateway": {
    "bind": "loopback",
    "port": 18789,
    "tailscale": {
      "mode": "serve"
    }
  },
  "browser": {
    "enabled": true,
    "color": "#FF4500"
  }
}
```

### CoPaw 配置

**位置**: `~/.copaw/config.json`

```json5
{
  "models": {
    "provider": "dashscope",
    "model": "qwen-max",
    "api_key": "${DASHSCOPE_API_KEY}"
  },
  "channels": {
    "dingtalk": {
      "client_id": "...",
      "client_secret": "...",
      "app_token": "..."
    },
    "feishu": {
      "app_id": "...",
      "app_secret": "..."
    }
  },
  "memory": {
    "enabled": true,
    "max_context": 100
  },
  "crons": {
    "enabled": true,
    "timezone": "Asia/Shanghai"
  }
}
```

---

## 7️⃣ Skills 系统对比

### OpenClaw Skills

**位置**: `~/.openclaw/workspace/skills/<skill>/SKILL.md`

```markdown
# Skill: alibaba-stock

获取阿里巴巴股票实时信息

## Usage
python3 scripts/get_stock_info.py [us|hk]

## Dependencies
- yfinance
```

**特点**:
- 📦 **ClawHub 注册表**: 在线搜索和安装 skills
- 🔧 **多语言支持**: Python, Node.js, Bash 等
- 🌐 **远程技能**: 支持远程技能调用
- 📚 **模板系统**: AGENTS.md, SOUL.md, TOOLS.md

### CoPaw Skills

**位置**: `~/.copaw/skills/<skill>/`

```python
# skills/my_skill.py
from copaw import skill

@skill
def my_skill(query: str) -> str:
    """My custom skill"""
    return f"Result: {query}"
```

**特点**:
- 🐍 **Python 原生**: 使用 Python 装饰器定义
- 🔌 **自动加载**: 工作区 skills 自动发现
- 📝 **简单 API**: 函数装饰器模式
- 🧩 **AgentScope 集成**: 与 AgentScope 生态系统兼容

---

## 8️⃣ 安全特性对比

### OpenClaw

| 安全特性 | 描述 |
|----------|------|
| **DM 配对策略** | 未知用户需配对码批准 |
| **沙箱模式** | 非主会话在 Docker 中运行 |
| **工具权限** | 可配置 allowlist/denylist |
| **TCC 权限** | macOS 权限管理 (通知、屏幕录制) |
| **能力令牌** | Canvas 访问需要临时令牌 |
| **路径验证** | 防止目录遍历攻击 |
| ** elevated bash** | 可选的提权 bash 访问 |

### CoPaw

| 安全特性 | 描述 |
|----------|------|
| **API 密钥管理** | 环境变量或配置文件 |
| **工作区隔离** | 每个工作区独立配置 |
| **本地模型** | 无需云 API 密钥 |
| **Python 沙箱** | 可选的代码执行限制 |

---

## 9️⃣ 社区与生态对比

### OpenClaw

```
⭐ Stars: 244,480
🔱 Forks: 47,264
👥 Contributors: 200+
📅 创建: 2025-11-24
📝 更新: 每日多次
💬 Discord: 活跃
🌐 网站: openclaw.ai
📚 文档: docs.openclaw.ai
```

**生态系统**:
- ClawHub Skills 注册表
- 官方 macOS/iOS/Android 应用
- Nix 包
- Docker 镜像
- 社区贡献通道

### CoPaw

```
⭐ Stars: 4,042
🔱 Forks: 402
👥 Contributors: AgentScope 团队
📅 创建: 2026-02-24
📝 更新: 活跃
💬 Discord: 有
🌐 网站: copaw.agentscope.io
📚 文档: copaw.agentscope.io/docs
```

**生态系统**:
- AgentScope 框架
- AgentScope Runtime
- ReMe 记忆系统
- ModelScope 云端部署
- 阿里云 ECS 集成

---

## 🔟 使用场景对比

### OpenClaw 适用场景

| 场景 | 匹配度 | 说明 |
|------|--------|------|
| **企业级部署** | ⭐⭐⭐⭐⭐ | 多通道、沙箱、权限管理 |
| **多通道集成** | ⭐⭐⭐⭐⭐ | 30+ 通信平台支持 |
| **移动设备集成** | ⭐⭐⭐⭐⭐ | iOS/Android 原生应用 |
| **可视化交互** | ⭐⭐⭐⭐⭐ | Canvas + A2UI |
| **语音交互** | ⭐⭐⭐⭐⭐ | Voice Wake + Talk Mode |
| **高级自动化** | ⭐⭐⭐⭐⭐ | Cron + Webhooks + Browser |
| **个人快速部署** | ⭐⭐⭐ | 配置复杂，学习曲线陡 |
| **本地模型** | ⭐ | 不支持 |
| **中文生态** | ⭐⭐⭐ | 支持飞书，但文档以英文为主 |

### CoPaw 适用场景

| 场景 | 匹配度 | 说明 |
|------|--------|------|
| **个人快速部署** | ⭐⭐⭐⭐⭐ | 一键安装，配置简单 |
| **本地模型运行** | ⭐⭐⭐⭐⭐ | llama.cpp + MLX 原生支持 |
| **中文生态** | ⭐⭐⭐⭐⭐ | 钉钉/飞书/QQ，中文文档 |
| **Python 扩展** | ⭐⭐⭐⭐⭐ | Python 生态，易于定制 |
| **定时任务** | ⭐⭐⭐⭐ | Heartbeat 系统 |
| **Web 控制台** | ⭐⭐⭐⭐ | Console UI 友好 |
| **企业级部署** | ⭐⭐⭐ | 功能相对简单 |
| **多通道集成** | ⭐⭐⭐ | 支持主流平台，数量有限 |
| **移动设备** | ⭐ | 无原生应用 |

---

## 1️⃣1️⃣ 优缺点总结

### OpenClaw

#### ✅ 优势

1. **功能全面** - 30+ 通道、Canvas、Voice、Browser、Nodes
2. **跨平台应用** - macOS/iOS/Android 原生应用
3. **企业级安全** - 沙箱、权限管理、DM 配对
4. **成熟生态** - 24 万 + stars，活跃社区
5. **高级特性** - A2UI 声明式 UI、模型故障转移、Tailscale 集成
6. **文档完善** - 详细的官方文档和 runbook

#### ❌ 劣势

1. **复杂度高** - 配置复杂，学习曲线陡
2. **资源消耗** - Node.js + 多应用，资源占用高
3. **无本地模型** - 仅支持云模型
4. **中文支持有限** - 文档以英文为主
5. **安装门槛** - 需要 Node.js 22+

### CoPaw

#### ✅ 优势

1. **简单易用** - 一键安装，配置简单
2. **本地模型** - 原生支持 llama.cpp 和 MLX
3. **Python 生态** - 易于理解和扩展
4. **中文友好** - 钉钉/飞书/QQ，中文文档
5. **Console UI** - 友好的 Web 控制台
6. **轻量级** - 代码量小，资源占用低

#### ❌ 劣势

1. **功能有限** - 缺少 Canvas、Voice、Browser 等高级功能
2. **通道较少** - 仅支持 ~10 个平台
3. **无移动应用** - 无 iOS/Android 原生应用
4. **社区较小** - 4k stars，相对年轻
5. **企业级功能弱** - 缺少沙箱、高级权限管理

---

## 1️⃣2️⃣ 选择建议

### 选择 OpenClaw，如果你需要:

- ✅ **企业级部署** - 多用户、多通道、安全隔离
- ✅ **全平台覆盖** - 桌面 + 移动 + Web
- ✅ **高级自动化** - Browser、Canvas、Voice、Cron
- ✅ **30+ 通信通道** - 全球主流平台支持
- ✅ **可视化交互** - A2UI 声明式 UI
- ✅ **成熟生态** - 大量社区贡献和文档

### 选择 CoPaw，如果你需要:

- ✅ **快速上手** - 一键安装，开箱即用
- ✅ **本地模型** - 隐私优先，无需云 API
- ✅ **Python 扩展** - 易于定制和开发
- ✅ **中文生态** - 钉钉/飞书/QQ 集成
- ✅ **轻量级部署** - 资源占用低
- ✅ **个人使用** - 简单配置，快速启动

---

## 1️⃣3️⃣ 技术决策矩阵

| 决策因素 | 权重 | OpenClaw 得分 | CoPaw 得分 |
|----------|------|---------------|------------|
| **功能完整性** | 20% | 10/10 | 6/10 |
| **易用性** | 15% | 6/10 | 9/10 |
| **本地模型支持** | 10% | 0/10 | 10/10 |
| **通道覆盖** | 15% | 10/10 | 5/10 |
| **移动应用** | 10% | 10/10 | 0/10 |
| **中文支持** | 10% | 5/10 | 9/10 |
| **社区生态** | 10% | 10/10 | 4/10 |
| **资源消耗** | 10% | 5/10 | 9/10 |
| **加权总分** | 100% | **8.15/10** | **6.35/10** |

---

## 1️⃣4️⃣ 未来展望

### OpenClaw

- 🎯 **持续扩展通道** - 更多通信平台集成
- 🎯 **AI 能力提升** - 更智能的 agent 路由
- 🎯 **企业功能** - 多租户、审计日志
- 🎯 **本地模型** - 可能集成本地推理

### CoPaw

- 🎯 **通道扩展** - 增加更多通信平台
- 🎯 **移动应用** - 可能开发 iOS/Android 应用
- 🎯 **本地模型优化** - 更好的量化和加速
- 🎯 **企业功能** - 团队协作、权限管理

---

## 📊 结论

**OpenClaw** 和 **CoPaw** 都是优秀的个人 AI 助手项目，但定位和侧重点不同:

- **OpenClaw** 是**功能全面的企业级解决方案**，适合需要多通道、高级自动化、跨平台支持的用户
- **CoPaw** 是**轻量级的个人助手**，适合快速部署、本地模型、中文生态的用户

**选择建议**:
- 企业用户、高级用户、需要全平台支持 → **OpenClaw**
- 个人用户、快速上手、本地模型优先 → **CoPaw**

---

*调研报告生成于 2026-03-02*
