# 🐈 nanobot Subagent 架构调研报告

**调研时间：** 2026-03-21  
**项目：** HKUDS/nanobot  
**分析范围：** Subagent 实现机制、与主 Agent Loop 协作流程

---

## 📋 执行摘要

nanobot 的 Subagent 系统是一个**轻量级后台任务执行框架**，通过 `spawn` 工具实现任务委派，支持并行执行、独立工具集、结果回传三大核心能力。

| 特性 | 实现状态 |
|------|----------|
| **后台任务执行** | ✅ 异步 asyncio 任务 |
| **独立工具集** | ✅ 无 message/spawn 工具 |
| **结果回传** | ✅ 通过 MessageBus 注入系统消息 |
| **会话隔离** | ✅ 按 session_key 分组管理 |
| **任务追踪** | ✅ task_id + label 标识 |
| **并发控制** | ⚠️ 无显式限制 |

---

## 🏗️ 核心架构

### 1️⃣ 组件概览

```
┌─────────────────────────────────────────────────────────────┐
│                      AgentLoop (主循环)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ ToolRegistry │  │ SubagentMgr  │  │ SessionMgr   │       │
│  │              │  │              │  │              │       │
│  │ - read_file  │  │ - spawn()    │  │ - sessions   │       │
│  │ - write_file │  │ - _run_      │  │ - history    │       │
│  │ - exec       │  │   subagent() │  │ - memory     │       │
│  │ - web_search │  │ - _announce_ │  │              │       │
│  │ - web_fetch  │  │   result()   │  │              │       │
│  │ - message    │  │              │  │              │       │
│  │ - spawn 🔌   │  │              │  │              │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ asyncio.create_task()
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Subagent (后台任务)                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ ToolRegistry │  │ LLM Provider │  │ MessageBus   │       │
│  │              │  │              │  │              │       │
│  │ - read_file  │  │ - chat_with_ │  │ - publish_   │       │
│  │ - write_file │  │   retry()    │  │   inbound()  │       │
│  │ - exec       │  │              │  │              │       │
│  │ - web_search │  │              │  │              │       │
│  │ - web_fetch  │  │              │  │              │       │
│  │ ❌ no message│  │              │  │              │       │
│  │ ❌ no spawn  │  │              │  │              │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

---

### 2️⃣ 核心类关系

| 类 | 文件 | 职责 |
|----|------|------|
| **AgentLoop** | `nanobot/agent/loop.py` | 主 agent 循环，接收消息/调用 LLM/执行工具 |
| **SubagentManager** | `nanobot/agent/subagent.py` | 管理后台 subagent 任务生命周期 |
| **SpawnTool** | `nanobot/agent/tools/spawn.py` | `spawn` 工具实现，调用 SubagentManager.spawn() |
| **ToolRegistry** | `nanobot/agent/tools/registry.py` | 工具注册与执行 |
| **MessageBus** | `nanobot/bus/queue.py` | 消息总线，subagent 结果回传通道 |

---

## 🔧 实现细节

### 1️⃣ SpawnTool 工具

**文件：** `nanobot/agent/tools/spawn.py`

```python
class SpawnTool(Tool):
    """Tool to spawn a subagent for background task execution."""

    def __init__(self, manager: "SubagentManager"):
        self._manager = manager
        self._origin_channel = "cli"
        self._origin_chat_id = "direct"
        self._session_key = "cli:direct"

    def set_context(self, channel: str, chat_id: str) -> None:
        """Set the origin context for subagent announcements."""
        self._origin_channel = channel
        self._origin_chat_id = chat_id
        self._session_key = f"{channel}:{chat_id}"

    @property
    def name(self) -> str:
        return "spawn"

    @property
    def description(self) -> str:
        return (
            "Spawn a subagent to handle a task in the background. "
            "Use this for complex or time-consuming tasks that can run independently. "
            "The subagent will complete the task and report back when done."
        )

    @property
    def parameters(self) -> dict[str, Any]:
        return {
            "type": "object",
            "properties": {
                "task": {
                    "type": "string",
                    "description": "The task for the subagent to complete",
                },
                "label": {
                    "type": "string",
                    "description": "Optional short label for the task (for display)",
                },
            },
            "required": ["task"],
        }

    async def execute(self, task: str, label: str | None = None, **kwargs: Any) -> str:
        """Spawn a subagent to execute the given task."""
        return await self._manager.spawn(
            task=task,
            label=label,
            origin_channel=self._origin_channel,
            origin_chat_id=self._origin_chat_id,
            session_key=self._session_key,
        )
```

**关键点：**
- `set_context()` 设置来源渠道和聊天 ID，用于结果回传
- `execute()` 调用 `SubagentManager.spawn()` 创建后台任务
- 返回立即返回任务启动确认，不等待完成

---

### 2️⃣ SubagentManager 管理器

**文件：** `nanobot/agent/subagent.py`

#### 初始化

```python
class SubagentManager:
    """Manages background subagent execution."""

    def __init__(
        self,
        provider: LLMProvider,
        workspace: Path,
        bus: MessageBus,
        model: str | None = None,
        web_search_config: "WebSearchConfig | None" = None,
        web_proxy: str | None = None,
        exec_config: "ExecToolConfig | None" = None,
        restrict_to_workspace: bool = False,
    ):
        self.provider = provider
        self.workspace = workspace
        self.bus = bus
        self.model = model or provider.get_default_model()
        self.web_search_config = web_search_config or WebSearchConfig()
        self.web_proxy = web_proxy
        self.exec_config = exec_config or ExecToolConfig()
        self.restrict_to_workspace = restrict_to_workspace
        
        # 任务追踪
        self._running_tasks: dict[str, asyncio.Task[None]] = {}
        self._session_tasks: dict[str, set[str]] = {}  # session_key -> {task_id, ...}
```

#### Spawn 方法

```python
async def spawn(
    self,
    task: str,
    label: str | None = None,
    origin_channel: str = "cli",
    origin_chat_id: str = "direct",
    session_key: str | None = None,
) -> str:
    """Spawn a subagent to execute a task in the background."""
    task_id = str(uuid.uuid4())[:8]
    display_label = label or task[:30] + ("..." if len(task) > 30 else "")
    origin = {"channel": origin_channel, "chat_id": origin_chat_id}

    # 创建后台任务
    bg_task = asyncio.create_task(
        self._run_subagent(task_id, task, display_label, origin)
    )
    self._running_tasks[task_id] = bg_task
    
    # 按 session_key 分组追踪
    if session_key:
        self._session_tasks.setdefault(session_key, set()).add(task_id)

    # 清理回调
    def _cleanup(_: asyncio.Task) -> None:
        self._running_tasks.pop(task_id, None)
        if session_key and (ids := self._session_tasks.get(session_key)):
            ids.discard(task_id)
            if not ids:
                del self._session_tasks[session_key]

    bg_task.add_done_callback(_cleanup)

    logger.info("Spawned subagent [{}]: {}", task_id, display_label)
    return f"Subagent [{display_label}] started (id: {task_id}). I'll notify you when it completes."
```

**关键点：**
- 生成 8 位 UUID 作为 task_id
- 使用 `asyncio.create_task()` 创建后台任务
- 按 `session_key` 分组追踪任务
- 任务完成后自动清理追踪记录

---

### 3️⃣ Subagent 执行循环

```python
async def _run_subagent(
    self,
    task_id: str,
    task: str,
    label: str,
    origin: dict[str, str],
) -> None:
    """Execute the subagent task and announce the result."""
    logger.info("Subagent [{}] starting task: {}", task_id, label)

    try:
        # 1️⃣ 构建 subagent 工具集（无 message/spawn 工具）
        tools = ToolRegistry()
        allowed_dir = self.workspace if self.restrict_to_workspace else None
        extra_read = [BUILTIN_SKILLS_DIR] if allowed_dir else None
        
        tools.register(ReadFileTool(workspace=self.workspace, allowed_dir=allowed_dir, extra_allowed_dirs=extra_read))
        tools.register(WriteFileTool(workspace=self.workspace, allowed_dir=allowed_dir))
        tools.register(EditFileTool(workspace=self.workspace, allowed_dir=allowed_dir))
        tools.register(ListDirTool(workspace=self.workspace, allowed_dir=allowed_dir))
        tools.register(ExecTool(
            working_dir=str(self.workspace),
            timeout=self.exec_config.timeout,
            restrict_to_workspace=self.restrict_to_workspace,
            path_append=self.exec_config.path_append,
        ))
        tools.register(WebSearchTool(config=self.web_search_config, proxy=self.web_proxy))
        tools.register(WebFetchTool(proxy=self.web_proxy))
        
        # 2️⃣ 构建系统提示
        system_prompt = self._build_subagent_prompt()
        messages: list[dict[str, Any]] = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": task},
        ]

        # 3️⃣ 运行 agent 循环（最多 15 次迭代）
        max_iterations = 15
        iteration = 0
        final_result: str | None = None

        while iteration < max_iterations:
            iteration += 1

            response = await self.provider.chat_with_retry(
                messages=messages,
                tools=tools.get_definitions(),
                model=self.model,
            )

            if response.has_tool_calls:
                tool_call_dicts = [
                    tc.to_openai_tool_call()
                    for tc in response.tool_calls
                ]
                messages.append(build_assistant_message(
                    response.content or "",
                    tool_calls=tool_call_dicts,
                    reasoning_content=response.reasoning_content,
                    thinking_blocks=response.thinking_blocks,
                ))

                # 执行工具
                for tool_call in response.tool_calls:
                    args_str = json.dumps(tool_call.arguments, ensure_ascii=False)
                    logger.debug("Subagent [{}] executing: {} with arguments: {}", task_id, tool_call.name, args_str)
                    result = await tools.execute(tool_call.name, tool_call.arguments)
                    messages.append({
                        "role": "tool",
                        "tool_call_id": tool_call.id,
                        "name": tool_call.name,
                        "content": result,
                    })
            else:
                final_result = response.content
                break

        if final_result is None:
            final_result = "Task completed but no final response was generated."

        logger.info("Subagent [{}] completed successfully", task_id)
        
        # 4️⃣ 回传结果
        await self._announce_result(task_id, label, task, final_result, origin, "ok")

    except Exception as e:
        error_msg = f"Error: {str(e)}"
        logger.error("Subagent [{}] failed: {}", task_id, e)
        await self._announce_result(task_id, label, task, error_msg, origin, "error")
```

**关键点：**
- **工具限制：** 无 `message` 工具（不能发消息）、无 `spawn` 工具（不能嵌套 spawn）
- **迭代限制：** 最多 15 次工具调用循环
- **独立会话：** 不继承主 agent 的会话历史
- **结果回传：** 通过 `_announce_result()` 注入系统消息

---

### 4️⃣ 结果回传机制

```python
async def _announce_result(
    self,
    task_id: str,
    label: str,
    task: str,
    result: str,
    origin: dict[str, str],
    status: str,
) -> None:
    """Announce the subagent result to the main agent via the message bus."""
    status_text = "completed successfully" if status == "ok" else "failed"

    announce_content = f"""[Subagent '{label}' {status_text}]

Task: {task}

Result:
{result}

Summarize this naturally for the user. Keep it brief (1-2 sentences). Do not mention technical details like "subagent" or task IDs."""

    # 通过 MessageBus 注入系统消息
    msg = InboundMessage(
        channel="system",
        sender_id="subagent",
        chat_id=f"{origin['channel']}:{origin['chat_id']}",
        content=announce_content,
    )

    await self.bus.publish_inbound(msg)
    logger.debug("Subagent [{}] announced result to {}:{}", task_id, origin['channel'], origin['chat_id'])
```

**关键点：**
- 使用 `InboundMessage` 注入系统消息
- `channel="system"` 标识为系统消息
- `chat_id` 指向原始会话，确保结果回到正确的聊天
- 提示主 agent 自然总结，不暴露技术细节

---

## 🔄 与主 Agent Loop 协作流程

### 1️⃣ 完整流程图

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│   User      │         │  AgentLoop  │         │ SubagentMgr │
│             │         │             │         │             │
│ 1. 发送请求  │────────>│             │         │             │
│             │         │ 2. 处理消息  │         │             │
│             │         │ LLM 决定 spawn│         │             │
│             │         │             │         │             │
│             │         │ 3. 调用 spawn 工具 ────>│ 4. 创建任务  │
│             │         │             │         │ asyncio.task│
│             │         │ 5. 立即返回  │         │             │
│             │<────────│ "Subagent   │         │             │
│             │         │  started"   │         │             │
│             │         │             │         │             │
│             │         │ 6. 继续处理  │         │ 7. 后台执行  │
│             │         │ 其他消息     │         │ subagent    │
│             │         │             │         │             │
│             │         │             │         │ 8. 完成     │
│             │         │             │         │             │
│             │         │ 9. 注入系统消息 ─────<│ 10. 回传结果 │
│             │         │ InboundMessage│       │             │
│             │         │             │         │             │
│             │         │ 11. 处理系统消息       │             │
│             │         │ LLM 总结结果  │         │             │
│             │<────────│ 12. 发送最终回复       │             │
│             │         │             │         │             │
└─────────────┘         └─────────────┘         └─────────────┘
```

---

### 2️⃣ AgentLoop 集成

**文件：** `nanobot/agent/loop.py`

```python
class AgentLoop:
    """The agent loop is the core processing engine."""

    def __init__(
        self,
        bus: MessageBus,
        provider: LLMProvider,
        workspace: Path,
        ...
    ):
        self.bus = bus
        self.provider = provider
        self.workspace = workspace
        ...
        
        # 初始化 SubagentManager
        self.subagents = SubagentManager(
            provider=provider,
            workspace=workspace,
            bus=bus,
            model=self.model,
            web_search_config=self.web_search_config,
            web_proxy=web_proxy,
            exec_config=self.exec_config,
            restrict_to_workspace=restrict_to_workspace,
        )
        
        self._register_default_tools()

    def _register_default_tools(self) -> None:
        """Register the default set of tools."""
        ...
        # 注册 spawn 工具
        self.tools.register(SpawnTool(manager=self.subagents))
        ...
```

**关键点：**
- `SubagentManager` 作为 `AgentLoop` 的成员初始化
- `SpawnTool` 注册到主工具集，LLM 可以调用
- 共享相同的 `provider`、`workspace`、`bus`

---

### 3️⃣ 消息流处理

```python
async def run(self) -> None:
    """Main agent loop: receive messages, process, respond."""
    self._running = True
    
    async for event in self.bus.subscribe_inbound():
        if not self._running:
            break
            
        # 处理 inbound 消息（包括 subagent 回传）
        if isinstance(event, InboundMessage):
            if event.channel == "system" and event.sender_id == "subagent":
                # Subagent 结果回传
                logger.info("Received subagent result: {}", event.content[:100])
            
            # 构建会话上下文
            session = self.sessions.get_session(event.chat_id)
            messages = await self.context.build_messages(
                session=session,
                system_content=self.context.build_system_prompt(),
            )
            
            # 运行 agent 循环
            final_content, tools_used, _ = await self._run_agent_loop(
                initial_messages=messages,
            )
            
            # 发送回复
            if final_content:
                await self.bus.publish_outbound(
                    OutboundMessage(
                        channel=event.channel,
                        chat_id=event.chat_id,
                        content=final_content,
                    )
                )
```

**关键点：**
- Subagent 结果作为 `InboundMessage` 注入
- `channel="system"` 标识为系统消息
- 主 agent 像处理普通消息一样处理 subagent 结果
- LLM 根据提示自然总结给用户

---

## 🎯 设计特点

### 1️⃣ 优势

| 特点 | 说明 | 收益 |
|------|------|------|
| **轻量级** | 仅 200 行核心代码 | 易于理解和维护 |
| **异步非阻塞** | `asyncio.create_task()` | 主 agent 不被阻塞 |
| **工具隔离** | 无 message/spawn 工具 | 防止无限嵌套和资源滥用 |
| **会话追踪** | `session_key` 分组 | 结果准确回传到原始会话 |
| **自动清理** | 任务完成回调 | 无内存泄漏 |
| **统一接口** | 通过 MessageBus | 与主 agent 无缝集成 |

---

### 2️⃣ 限制

| 限制 | 影响 | 缓解方案 |
|------|------|----------|
| **无并发限制** | 可能 spawn 过多任务 | 应用层控制或添加配置 |
| **无任务优先级** | 所有任务平等对待 | 未来可扩展优先级队列 |
| **无进度报告** | 只能等待完成 | 可通过定期消息实现 |
| **无嵌套 spawn** | subagent 不能 spawn 子任务 | 设计决策，防止复杂性 |
| **独立会话** | 不继承主会话历史 | 有利有弊，取决于场景 |

---

### 3️⃣ 与 MicroClaw 对比

| 特性 | nanobot | MicroClaw |
|------|---------|-----------|
| **实现语言** | Python | Rust |
| **任务管理** | asyncio.Task | tokio::task |
| **结果回传** | MessageBus 注入 | Event Channel |
| **会话隔离** | session_key 分组 | 会话原生 v1 |
| **控制平面** | 基础 (list/kill) | 完整 (spawn/list/info/kill) |
| **嵌套深度** | 禁止 (无 spawn 工具) | 可配置 |
| **代码量** | ~200 行 | ~500 行 |

---

## 📊 使用场景

### 1️⃣ 适合场景

| 场景 | 说明 | 示例 |
|------|------|------|
| **后台研究** | 独立调研任务 | "研究一下 nanobot 的 subagent 实现" |
| **文件处理** | 批量文件操作 | "分析这个项目的代码结构" |
| **Web 搜索** | 信息收集 | "查找最新的 AI agent 框架" |
| **代码执行** | 测试/验证 | "运行这个脚本并分析结果" |
| **长时间任务** | 不阻塞主对话 | "生成一份详细报告" |

---

### 2️⃣ 不适合场景

| 场景 | 原因 | 替代方案 |
|------|------|----------|
| **实时交互** | subagent 不能与用户对话 | 主 agent 直接处理 |
| **需要上下文** | 不继承会话历史 | 在主 agent 中处理 |
| **嵌套任务** | 禁止 spawn 子任务 | 单次 spawn 完成所有 |
| **高优先级** | 无优先级机制 | 主 agent 优先处理 |

---

## 🔮 未来演进

根据 GitHub Issues 和 Roadmap 讨论：

### 1️⃣ 计划功能

| 功能 | Issue | 状态 |
|------|-------|------|
| **任务控制平面** | #1006 | ✅ 部分实现 (list/kill) |
| **子 agent 配置** | #1012 | 🔄 讨论中 |
| **进度报告** | - | 💡 建议 |
| **优先级队列** | - | 💡 建议 |
| **会话继承** | - | 💡 建议 |

---

### 2️⃣ 架构演进方向

```
当前架构 (v1)          →    未来架构 (v2)
─────────────────────────────────────────────
单级 spawn                  多级嵌套 (可配置深度)
无优先级                    优先级队列
简单追踪                    完整控制平面
独立会话                    可选会话继承
无进度                      流式进度报告
```

---

## 📝 代码示例

### 1️⃣ 使用 spawn 工具

```python
# LLM 决定 spawn subagent
# 工具调用：
{
  "name": "spawn",
  "arguments": {
    "task": "研究 GitHub 上 nanobot 项目的 subagent 实现，分析其架构和与主 agent loop 的协作机制",
    "label": "nanobot subagent 调研"
  }
}

# 立即返回：
"Subagent [nanobot subagent 调研] started (id: a1b2c3d4). I'll notify you when it completes."

# 后台执行...

# 完成后注入系统消息：
"""
[Subagent 'nanobot subagent 调研' completed successfully]

Task: 研究 GitHub 上 nanobot 项目的 subagent 实现...

Result:
SubagentManager 通过 asyncio.create_task() 创建后台任务...
(详细结果)

Summarize this naturally for the user. Keep it brief (1-2 sentences).
"""

# 主 agent 总结：
"我已经完成了 nanobot subagent 的调研。核心是通过 asyncio 后台任务实现，
结果通过消息总线回传。详细报告已保存。"
```

---

### 2️⃣ 程序化调用

```python
from nanobot.agent.subagent import SubagentManager
from nanobot.bus.queue import MessageBus
from nanobot.providers.dashscope import DashScopeProvider

# 初始化
provider = DashScopeProvider(api_key="...")
bus = MessageBus()
workspace = Path("/workspace")

subagents = SubagentManager(
    provider=provider,
    workspace=workspace,
    bus=bus,
    model="qwen-max",
)

# Spawn 任务
task_id = await subagents.spawn(
    task="分析项目代码结构",
    label="代码分析",
    origin_channel="feishu",
    origin_chat_id="ou_xxx",
    session_key="feishu:ou_xxx",
)

# 后台执行，结果自动回传
```

---

## 📋 总结

### 核心设计原则

1. **轻量优先** - 最小化代码复杂度，易于理解和修改
2. **异步非阻塞** - 主 agent 不被后台任务阻塞
3. **工具隔离** - 防止无限嵌套和资源滥用
4. **统一接口** - 通过 MessageBus 与主 agent 无缝集成
5. **自动清理** - 任务完成后自动释放资源

---

### 适用性评估

| 维度 | 评分 | 说明 |
|------|------|------|
| **易用性** | ⭐⭐⭐⭐ | spawn 工具简单易用 |
| **灵活性** | ⭐⭐⭐ | 工具集固定，不可自定义 |
| **可扩展性** | ⭐⭐⭐ | 架构清晰，易于扩展 |
| **可靠性** | ⭐⭐⭐⭐ | 异常处理完善，自动清理 |
| **性能** | ⭐⭐⭐⭐ | 异步非阻塞，并发执行 |

---

**调研完成时间：** 2026-03-21 15:30 (北京时间)  
**分析基于：** nanobot main 分支源代码  
**报告保存：** `/workspace/resources/nanobot-subagent-research.md`

---

🐈 *nanobot - The Ultra-Lightweight OpenClaw*