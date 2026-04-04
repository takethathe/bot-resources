# OpenHarness 与 nanobot 集成分析报告

# Openharness Nanobot Integration Research

**Generated**: 2026-04-04 00:49:14 UTC  
**Type**: research  
**Category**: projects/nanobot  
**Topic**: openharness-nanobot-integration

---


**分析日期**: 2026-04-04  
**分析对象**: HKUDS/OpenHarness + nanobot  
**报告作者**: nanobot 🐈

---

## 一、项目概述

### 1.1 OpenHarness (hkuds/openharness)

**定位**: 超轻量级 Agent Harness（代理框架）

**核心特点**:
- **代码量**: 11,733 行 (比 Claude Code 轻 44 倍)
- **语言**: Python
- **命令行**: `oh` 命令
- **Star 数**: 2,066+ (截至 2026-04-04)
- **测试覆盖**: 114 个单元测试 + 6 个 E2E 测试套件

**官方描述**:
> "OpenHarness delivers core lightweight agent infrastructure: tool-use, skills, memory, and multi-agent coordination."

**主要功能模块**:
| 模块 | 功能 |
|------|------|
| `engine/` | 🧠 Agent Loop — query → stream → tool-call → loop |
| `tools/` | 🔧 43 工具 — 文件 I/O、shell、搜索、web、MCP |
| `skills/` | 📚 知识 — 按需加载技能 (.md 文件) |
| `plugins/` | 🔌 扩展 — 命令、hooks、agents、MCP 服务器 |
| `permissions/` | 🛡️ 安全 — 多级权限模式、路径规则、命令拒绝 |
| `hooks/` | ⚡ 生命周期 — PreToolUse/PostToolUse 事件钩子 |
| `commands/` | 💬 54 命令 — /help, /commit, /plan, /resume 等 |
| `mcp/` | 🌐 MCP — Model Context Protocol 客户端 |
| `memory/` | 🧠 记忆 — 持久化跨会话知识 |
| `tasks/` | 📋 任务 — 后台任务管理 |
| `coordinator/` | 🤝 多代理 — 子代理生成、团队协调 |
| `bridge/` | 🌉 桥接 — 外部 Agent CLI 集成 |

---

### 1.2 nanobot

**定位**: 个人 AI 助手 (Personal AI Assistant)

**核心特点**:
- **代码量**: 轻量级 Python 项目
- **版本**: 0.1.4.post5
- **Logo**: 🐈
- **命令行**: `nanobot` 命令
- **主要场景**: 多通道消息处理 (Telegram/Discord/Feishu)、定时任务、技能系统

**主要功能模块**:
| 模块 | 功能 |
|------|------|
| `agent/` | 🧠 Agent Loop — 消息处理、工具调用、子代理 |
| `channels/` | 📱 渠道 — Telegram/Discord/Feishu 等消息渠道 |
| `skills/` | 📚 技能 — 自定义技能系统 (JSON/SKILL.md) |
| `cron/` | ⏰ 定时 — 周期性任务和提醒 |
| `bus/` | 🚌 消息总线 — 事件队列管理 |
| `providers/` | 🔌 LLM 提供商 — 多模型支持 |
| `session/` | 📝 会话 — 会话管理和历史 |
| `heartbeat/` | 💓 心跳 — 30 分钟周期性任务检查 |

---

## 二、架构对比

### 2.1 Agent Loop 核心架构

#### OpenHarness 的 QueryEngine

```python
class QueryEngine:
    """Owns conversation history and the tool-aware model loop."""
    
    async def submit_message(self, prompt: str) -> AsyncIterator[StreamEvent]:
        self._messages.append(ConversationMessage.from_user_text(prompt))
        context = QueryContext(
            api_client=self._api_client,
            tool_registry=self._tool_registry,
            permission_checker=self._permission_checker,
            ...
        )
        async for event, usage in run_query(context, self._messages):
            yield event
```

**核心循环**:
```python
while True:
    response = await api.stream(messages, tools)
    
    if response.stop_reason != "tool_use":
        break  # Model is done
    
    for tool_call in response.tool_uses:
        # Permission check → Hook → Execute → Hook → Result
        result = await harness.execute_tool(tool_call)
    
    messages.append(tool_results)
    # Loop continues — model sees results, decides next action
```

#### nanobot 的 AgentLoop

```python
class AgentLoop:
    """The agent loop is the core processing engine."""
    
    async def _process_message(self, event: InboundMessage) -> None:
        # 1. 从总线接收消息
        # 2. 构建上下文 (历史、记忆、技能)
        # 3. 调用 LLM
        # 4. 执行工具调用
        # 5. 发送响应回总线
```

**核心流程**:
```
消息渠道 → MessageBus → AgentLoop → LLM Provider → ToolRegistry → 执行工具 → 响应渠道
```

---

### 2.2 关键差异

| 特性 | OpenHarness | nanobot |
|------|-------------|---------|
| **主要场景** | 本地编码助手 (CLI + TUI) | 多通道消息助手 (IM 集成) |
| **交互界面** | React Ink TUI / CLI | Telegram/Discord/Feishu |
| **权限系统** | 多级权限模式、路径规则、交互式审批 | 基础权限控制 |
| **Hooks 系统** | PreToolUse/PostToolUse 钩子 | 无 |
| **MCP 支持** | 完整 MCP 客户端 | 基础 MCP 连接 |
| **子代理** | coordinator 模块 + ClawTeam 集成 | SubagentManager |
| **记忆系统** | MEMORY.md + 会话恢复 | MEMORY.md + HISTORY.md + 自动整合 |
| **定时任务** | tasks 模块 | cron 服务 + heartbeat 心跳任务 |
| **技能格式** | anthropics/skills 兼容 (.md) | 自定义 SKILL.md + Python 脚本 |

---

## 三、集成方式

### 3.1 OpenHarness 的 Bridge 机制

OpenHarness 设计了 `bridge/` 模块专门用于与外部 Agent CLI 集成：

```python
# openharness/bridge/manager.py
class BridgeSessionManager:
    """Manage bridge-run child sessions and capture their output."""
    
    async def spawn(self, *, session_id: str, command: str, cwd: str | Path) -> SessionHandle:
        handle = await spawn_session(session_id=session_id, command=command, cwd=cwd)
        # 启动子进程并捕获输出
```

**支持的外部 Agent**:
- OpenClaw
- nanobot
- Cursor
- 其他兼容 CLI

### 3.2 集成模式

#### 模式 A: OpenHarness 作为 nanobot 的前端 TUI

```
用户 → OpenHarness TUI (React Ink) → Bridge → nanobot CLI → LLM
                              ↓
                        捕获输出并显示
```

**命令示例**:
```bash
# 通过 OpenHarness 启动 nanobot 会话
oh bridge --command "nanobot --channel cli"
```

#### 模式 B: nanobot 使用 OpenHarness 的工具/技能系统

```
nanobot AgentLoop → OpenHarness ToolRegistry → 43 工具
                 ↓
          OpenHarness Skills (.md)
```

**技能兼容**:
```bash
# OpenHarness 技能目录
~/.openharness/skills/my-skill.md

# nanobot 技能目录
~/.nanobot/workspace/skills/my-skill/SKILL.md
```

#### 模式 C: 共享记忆和上下文

```
MEMORY.md (共享)
    ↓
OpenHarness ←→ nanobot
    ↓
CLAUDE.md / SOUL.md (上下文注入)
```

---

## 四、核心代码对比

### 4.1 工具注册

#### OpenHarness
```python
from openharness.tools.base import BaseTool, ToolRegistry

class MyTool(BaseTool):
    name = "my_tool"
    description = "Does something useful"
    input_model = MyToolInput
    
    async def execute(self, arguments, context) -> ToolResult:
        return ToolResult(output=f"Result for: {arguments.query}")

# 注册
tool_registry.register(MyTool())
```

#### nanobot
```python
from nanobot.agent.tools.registry import ToolRegistry
from nanobot.agent.tools.base import BaseTool

class MyTool(BaseTool):
    name = "my_tool"
    description = "Does something useful"
    
    async def execute(self, **kwargs) -> dict:
        return {"result": f"Result for: {kwargs['query']}"}

# 注册
tools = ToolRegistry()
tools.register(MyTool())
```

---

### 4.2 子代理系统

#### OpenHarness Coordinator
```python
# openharness/coordinator/
# - agent_definitions.py: 代理定义
# - coordinator_mode.py: 协调模式
# 支持 ClawTeam 集成 (roadmap)
```

#### nanobot SubagentManager
```python
class SubagentManager:
    async def spawn(self, task: str, label: str = None) -> str:
        task_id = str(uuid.uuid4())[:8]
        bg_task = asyncio.create_task(
            self._run_subagent(task_id, task, label, origin)
        )
        return f"Subagent [{label}] started (id: {task_id})"
```

**nanobot spawn 工具** (用户可直接调用):
```python
# nanobot/agent/tools/spawn.py
class SpawnTool:
    name = "spawn"
    description = "Spawn a subagent to handle a task in the background"
```

---

### 4.3 记忆系统

#### OpenHarness Memory
```
~/.openharness/memory/
├── MEMORY.md      # 持久化记忆
└── sessions/      # 会话历史
```

#### nanobot Memory
```
~/.nanobot/workspace/memory/
├── MEMORY.md      # 长期记忆 (始终加载到上下文)
└── HISTORY.md     # 事件日志 (grep 搜索)
```

**nanobot 记忆技能** (`skills/memory/SKILL.md`):
```markdown
## Search Past Events
- Small HISTORY.md: use read_file, then search in-memory
- Large HISTORY.md: use exec for targeted search (grep)
```

---

## 五、使用场景

### 5.1 OpenHarness 适用场景

| 场景 | 说明 |
|------|------|
| **本地编码助手** | 读取代码、修补文件、运行检查 |
| **无头脚本** | JSON/stream-json 输出用于自动化 |
| **插件/技能测试台** | 实验 Claude 风格扩展 |
| **多代理原型** | 任务委托和后台执行 |
| **提供商对比** | Anthropic 兼容后端测试 |

### 5.2 nanobot 适用场景

| 场景 | 说明 |
|------|------|
| **IM 消息助手** | Telegram/Discord/Feishu 集成 |
| **定时任务** | 每日新闻推送、周期性报告 |
| **技能扩展** | 自定义 Python 技能 |
| **多通道广播** | 跨渠道消息同步 |
| **个人助理** | 天气、股票、待办事项等 |

---

## 六、集成建议

### 6.1 推荐集成方案

#### 方案 1: OpenHarness 作为 nanobot 的 TUI 前端

**优势**:
- 利用 OpenHarness 的 React Ink TUI
- 保留 nanobot 的多通道和定时任务能力
- 共享技能和记忆系统

**实现步骤**:
```bash
# 1. 安装 OpenHarness
git clone https://github.com/HKUDS/OpenHarness.git
cd OpenHarness
uv sync

# 2. 配置 Bridge
oh bridge add --name nanobot --command "nanobot --channel cli"

# 3. 启动
oh bridge run nanobot
```

#### 方案 2: nanobot 使用 OpenHarness 工具集

**优势**:
- 获得 43+ 预构建工具
- 兼容 anthropics/skills 生态
- 增强的权限和钩子系统

**实现步骤**:
```python
# nanobot 配置中引入 OpenHarness 工具
from openharness.tools import ToolRegistry as OHToolRegistry

oh_tools = OHToolRegistry()
nanobot_tools.register_from(oh_tools)
```

#### 方案 3: 共享记忆和上下文

**优势**:
- 跨会话知识共享
- 统一的 CLAUDE.md/SOUL.md 上下文
- 一致的代理行为

**实现步骤**:
```bash
# 符号链接共享记忆
ln -s ~/.nanobot/workspace/memory/MEMORY.md ~/.openharness/memory/MEMORY.md
```

---

### 6.2 代码级集成点

| 集成点 | OpenHarness 模块 | nanobot 模块 | 集成方式 |
|--------|-----------------|-------------|---------|
| **工具系统** | `tools/` | `agent/tools/` | 工具适配器/包装器 |
| **技能系统** | `skills/` | `skills/` | 共享 .md 格式 |
| **记忆系统** | `memory/` | `memory/` | 共享 MEMORY.md |
| **子代理** | `coordinator/` | `agent/subagent.py` | 桥接 spawn 命令 |
| **权限检查** | `permissions/` | `security/` | 权限策略同步 |
| **MCP 服务器** | `mcp/` | `agent/tools/mcp.py` | 共享 MCP 配置 |

---

## 七、总结

### 7.1 各自主要作用

#### OpenHarness
> **"The model is the agent. The code is the harness."**

- **核心定位**: 轻量级 Agent Harness 基础设施
- **主要作用**: 提供工具使用、技能、记忆和多代理协调的核心架构
- **优势**: 
  - 44 倍轻于 Claude Code
  - 完整的权限和钩子系统
  - 兼容 anthropics/skills 生态
  - React Ink TUI 界面

#### nanobot
> **"Personal AI Assistant 🐈"**

- **核心定位**: 个人 AI 助手
- **主要作用**: 多通道消息处理、定时任务、技能扩展
- **优势**:
  - 多 IM 渠道集成 (Telegram/Discord/Feishu)
  - 定时任务和心跳系统
  - 自定义技能系统
  - 轻量级设计

---

### 7.2 结合使用价值

| 结合点 | 价值 |
|--------|------|
| **TUI + 多通道** | OpenHarness TUI 用于本地开发，nanobot 用于 IM 消息 |
| **工具共享** | OpenHarness 43 工具增强 nanobot 能力 |
| **技能生态** | 共享 anthropics/skills 兼容技能 |
| **记忆统一** | 跨平台共享 MEMORY.md 知识 |
| **子代理协调** | OpenHarness coordinator + nanobot spawn |

---

### 7.3 推荐实践

1. **本地开发**: 使用 OpenHarness (`oh`) 作为主要编码助手
2. **消息通知**: 使用 nanobot 处理 Telegram/Feishu 消息推送
3. **技能开发**: 编写兼容两种格式的 SKILL.md
4. **记忆管理**: 共享 MEMORY.md，统一知识存储
5. **定时任务**: 使用 nanobot cron/heartbeat 系统

---

**报告完成** 🐈