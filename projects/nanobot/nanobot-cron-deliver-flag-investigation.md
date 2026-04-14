# nanobot Cron Deliver Flag 调查报告

# Nanobot Cron Deliver Flag Investigation

**Generated**: 2026-04-14 13:23:03 UTC  
**Type**: investigation  
**Category**: projects/nanobot  
**Topic**: nanobot-cron-deliver-flag

---


**调查日期**: 2026-04-14  
**调查对象**: nanobot cron 服务中的 `deliver` 标志  
**报告作者**: nanobot 🐈

---

## 一、核心结论

### `deliver` 标志的用途

**`deliver` 是一个布尔标志，用于控制 cron 任务执行后的响应是否发送回用户的消息渠道。**

- **`deliver=True`**: 任务执行结果会通过原始渠道 (如 Telegram/Feishu/Discord) 发送给用户
- **`deliver=False`**: 任务执行结果仅在内部处理，不发送通知给用户

---

## 二、代码实现分析

### 2.1 数据结构定义 (`cron/types.py`)

```python
@dataclass
class CronPayload:
    """What to do when the job runs."""
    kind: Literal["system_event", "agent_turn"] = "agent_turn"
    message: str = ""
    # Deliver response to channel
    deliver: bool = False        # ← 核心标志
    channel: str | None = None   # e.g. "whatsapp"
    to: str | None = None        # e.g. phone number
```

**关键字段**:
| 字段 | 类型 | 说明 |
|------|------|------|
| `deliver` | `bool` | 是否将响应发送回用户渠道 |
| `channel` | `str` | 目标渠道 (telegram/feishu/discord 等) |
| `to` | `str` | 目标用户 ID (chat_id) |

---

### 2.2 CronTool 自动设置 (`agent/tools/cron.py`)

```python
def _add_job(
    self,
    message: str,
    every_seconds: int | None,
    cron_expr: str | None,
    tz: str | None,
    at: str | None,
) -> str:
    # ... 验证和调度构建 ...
    
    job = self._cron.add_job(
        name=message[:30],
        schedule=schedule,
        message=message,
        deliver=True,              # ← 默认设置为 True
        channel=self._channel,     # ← 使用当前会话渠道
        to=self._chat_id,          # ← 使用当前会话用户 ID
        delete_after_run=delete_after,
    )
    return f"Created job '{job.name}' (id: {job.id})"
```

**关键点**:
- CronTool 在创建任务时**自动设置 `deliver=True`**
- 自动捕获当前会话的 `channel` 和 `chat_id` 作为投递目标
- 用户无需手动指定这些参数

---

### 2.3 CronService 执行流程 (`cron/service.py`)

```python
async def _execute_job(self, job: CronJob) -> None:
    """Execute a single job."""
    start_ms = _now_ms()
    logger.info("Cron: executing job '{}' ({})", job.name, job.id)

    try:
        if self.on_job:
            await self.on_job(job)  # ← 调用回调处理任务

        job.state.last_status = "ok"
        job.state.last_error = None
        logger.info("Cron: job '{}' completed", job.name)

    except Exception as e:
        job.state.last_status = "error"
        job.state.last_error = str(e)
        logger.error("Cron: job '{}' failed: {}", job.name, e)

    # ... 记录执行历史 ...
```

**执行流程**:
1. CronService 触发 `on_job` 回调
2. 回调函数执行实际的 agent 任务
3. 根据 `deliver` 标志决定是否发送响应

---

### 2.4 CLI 中的 on_cron_job 回调 (`cli/commands.py`)

这是**最关键的处理逻辑**:

```python
async def on_cron_job(job: CronJob) -> str | None:
    """Execute a cron job through the agent."""
    from nanobot.agent.tools.cron import CronTool
    from nanobot.agent.tools.message import MessageTool
    from nanobot.utils.evaluator import evaluate_response

    reminder_note = (
        "[Scheduled Task] Timer finished.\n\n"
        f"Task '{job.name}' has been triggered.\n"
        f"Scheduled instruction: {job.payload.message}"
    )

    # 1. 设置 cron 上下文 (防止递归调度)
    cron_tool = agent.tools.get("cron")
    cron_token = None
    if isinstance(cron_tool, CronTool):
        cron_token = cron_tool.set_cron_context(True)
    
    try:
        # 2. 通过 agent 执行任务
        response = await agent.process_direct(
            reminder_note,
            session_key=f"cron:{job.id}",
            channel=job.payload.channel or "cli",
            chat_id=job.payload.to or "direct",
        )
    finally:
        if isinstance(cron_tool, CronTool) and cron_token is not None:
            cron_tool.reset_cron_context(cron_token)

    # 3. 检查 agent 是否已通过 message 工具发送消息
    message_tool = agent.tools.get("message")
    if isinstance(message_tool, MessageTool) and message_tool._sent_in_turn:
        return response  # 已发送，直接返回

    # 4. 如果 deliver=True 且有响应，评估是否需要通知
    if job.payload.deliver and job.payload.to and response:
        should_notify = await evaluate_response(
            response, job.payload.message, provider, agent.model,
        )
        if should_notify:
            from nanobot.bus.events import OutboundMessage
            await bus.publish_outbound(OutboundMessage(
                channel=job.payload.channel or "cli",
                chat_id=job.payload.to,
                content=response,
            ))
    return response
```

**处理逻辑**:
1. **设置 cron 上下文**: 防止任务执行中递归创建新任务
2. **执行 agent 任务**: 将定时任务作为 agent turn 处理
3. **检查 message 工具**: 如果 agent 已主动发送消息，跳过 deliver 逻辑
4. **评估并投递**: 如果 `deliver=True`，通过 LLM 评估响应重要性，决定是否发送通知

---

### 2.5 响应评估器 (`utils/evaluator.py`)

```python
async def evaluate_response(
    response: str,
    task_context: str,
    provider: LLMProvider,
    model: str,
) -> bool:
    """Decide whether a background-task result should be delivered to the user."""
    
    _SYSTEM_PROMPT = (
        "You are a notification gate for a background agent. "
        "You will be given the original task and the agent's response. "
        "Call the evaluate_notification tool to decide whether the user "
        "should be notified.\n\n"
        "Notify when the response contains actionable information, errors, "
        "completed deliverables, or anything the user explicitly asked to "
        "be reminded about.\n\n"
        "Suppress when the response is a routine status check with nothing "
        "new, a confirmation that everything is normal, or essentially empty."
    )
    
    # 使用 LLM 评估是否应该通知用户
    llm_response = await provider.chat_with_retry(
        messages=[
            {"role": "system", "content": _SYSTEM_PROMPT},
            {"role": "user", "content": (
                f"## Original task\n{task_context}\n\n"
                f"## Agent response\n{response}"
            )},
        ],
        tools=_EVALUATE_TOOL,
        model=model,
        max_tokens=256,
        temperature=0.0,
    )
    
    # 默认通知 (fail-safe 设计)
    if not llm_response.has_tool_calls:
        logger.warning("evaluate_response: no tool call returned, defaulting to notify")
        return True
    
    args = llm_response.tool_calls[0].arguments
    should_notify = args.get("should_notify", True)
    return bool(should_notify)
```

**评估工具定义**:
```python
_EVALUATE_TOOL = [
    {
        "type": "function",
        "function": {
            "name": "evaluate_notification",
            "description": "Decide whether the user should be notified about this background task result.",
            "parameters": {
                "type": "object",
                "properties": {
                    "should_notify": {
                        "type": "boolean",
                        "description": "true = result contains actionable/important info the user should see; false = routine or empty, safe to suppress",
                    },
                    "reason": {
                        "type": "string",
                        "description": "One-sentence reason for the decision",
                    },
                },
                "required": ["should_notify"],
            },
        },
    }
]
```

**评估标准**:
| 通知 (should_notify=True) | 不通知 (should_notify=False) |
|--------------------------|---------------------------|
| 包含可操作信息 | 例行状态检查 |
| 错误/异常 | 一切正常的确认 |
| 完成的交付物 | 空响应或无新内容 |
| 用户明确要求提醒的内容 | 常规后台任务 |

**Fail-safe 设计**: 评估失败时默认通知用户，避免重要消息被遗漏。

---

## 三、完整流程图

```
用户创建 cron 任务
       ↓
CronTool._add_job()
  - 设置 deliver=True
  - 捕获 channel/chat_id
       ↓
CronService 定时触发
       ↓
on_cron_job 回调
       ↓
agent.process_direct() 执行任务
       ↓
检查 message_tool._sent_in_turn
       ↓
   ┌─── 已发送 ───┐
   ↓              ↓
  返回        deliver=True?
                  ↓
           ┌── No ──┐
           ↓        ↓
         返回    evaluate_response()
                    ↓
              LLM 评估响应重要性
                    ↓
              should_notify=True?
                    ↓
             ┌── No ──┐
             ↓        ↓
           返回   bus.publish_outbound()
                      ↓
                发送消息到用户渠道
```

---

## 四、使用示例

### 4.1 通过 cron 工具创建任务

```python
# 一次性提醒 (2 小时后)
cron(
    action="add",
    message="提醒我查看股票价格",
    at="2026-04-14T15:11:00"
)
# 自动设置: deliver=True, channel="feishu", to="ou_62dcc4b3b169d86b43f51c114349257c"

# 周期性任务 (每天 8 点)
cron(
    action="add",
    message="请推送今日 AI 新闻到飞书",
    cron_expr="0 8 * * *",
    tz="Asia/Shanghai"
)
# 自动设置: deliver=True, channel="feishu", to="ou_62dcc4b3b169d86b43f51c114349257c"
```

### 4.2 任务执行结果

**场景 A: 有实际内容 (会通知)**
```
任务：请推送今日 AI 新闻到飞书
Agent 响应：# 今日 AI 新闻摘要
1. OpenAI 发布 GPT-5...
2. Google 推出新的多模态模型...
→ evaluate_response: should_notify=True
→ 发送消息到 Feishu 渠道
```

**场景 B: 例行检查 (可能不通知)**
```
任务：检查服务器状态
Agent 响应：服务器运行正常，CPU 使用率 5%，内存使用率 30%
→ evaluate_response: should_notify=False (例行状态，无异常)
→ 不发送通知
```

**场景 C: 发现异常 (会通知)**
```
任务：检查服务器状态
Agent 响应：⚠️ 警告：磁盘使用率已达 95%，需要清理
→ evaluate_response: should_notify=True (包含可操作信息)
→ 发送消息到 Feishu 渠道
```

---

## 五、与其他系统的对比

### 5.1 Cron vs Heartbeat

| 特性 | Cron 服务 | Heartbeat 系统 |
|------|----------|---------------|
| **调度方式** | 精确时间 (at/cron/every) | 固定周期 (30 分钟) |
| **deliver 标志** | ✅ 支持 | ✅ 支持 (类似逻辑) |
| **评估机制** | evaluate_response() | _decide() (类似) |
| **任务来源** | 用户创建 | HEARTBEAT.md 文件 |
| **典型用途** | 定时提醒、周期性报告 | 周期性后台任务检查 |

### 5.2 Heartbeat 的 deliver 实现

```python
# cli/commands.py
async def on_heartbeat_notify(response: str) -> None:
    """Deliver a heartbeat response to the user's channel."""
    from nanobot.bus.events import OutboundMessage
    channel, chat_id = _pick_heartbeat_target()
    if channel == "cli":
        return  # No external channel available to deliver to
    await bus.publish_outbound(OutboundMessage(
        channel=channel, 
        chat_id=chat_id, 
        content=response
    ))
```

---

## 六、设计意图分析

### 6.1 为什么需要 deliver 标志？

1. **避免消息骚扰**: 后台任务可能频繁执行，但不是所有结果都需要通知用户
2. **智能过滤**: 通过 LLM 评估响应重要性，只发送有价值的信息
3. **灵活性**: 允许用户选择是否接收任务执行结果
4. **渠道感知**: 自动将结果发送回原始创建渠道

### 6.2 为什么默认 deliver=True？

```python
# cron.py:140
deliver=True,  # 默认设置为 True
```

**原因**:
- 用户创建定时任务时期望收到结果
- Fail-safe 设计：宁可多通知，不可遗漏
- 评估机制会过滤掉不重要的通知

### 6.3 为什么需要 evaluate_response？

**问题场景**:
- 周期性任务每小时执行，每次都发送"一切正常"会骚扰用户
- 后台检查任务可能产生大量技术性输出

**解决方案**:
- 使用轻量级 LLM 调用评估响应价值
- 只发送包含可操作信息/错误/交付物的响应
- 默认通知 (评估失败时不丢失重要消息)

---

## 七、相关代码位置

| 文件 | 行号 | 功能 |
|------|------|------|
| `nanobot/cron/types.py` | 22-29 | CronPayload 定义 (deliver 字段) |
| `nanobot/cron/service.py` | 265-305 | _execute_job 执行逻辑 |
| `nanobot/cron/service.py` | 314-352 | add_job API |
| `nanobot/agent/tools/cron.py` | 96-145 | _add_job 实现 (设置 deliver=True) |
| `nanobot/agent/tools/cron.py` | 26-32 | cron 上下文管理 |
| `nanobot/cli/commands.py` | 547-590 | on_cron_job 回调 (核心处理) |
| `nanobot/utils/evaluator.py` | 1-92 | evaluate_response 评估器 |

---

## 八、总结

### `deliver` 标志的核心作用

> **控制 cron 任务执行结果是否通过原始消息渠道发送回用户，配合 LLM 评估机制实现智能通知过滤。**

### 工作流程

1. **创建任务**: CronTool 自动设置 `deliver=True` + 渠道信息
2. **定时触发**: CronService 调用 `on_job` 回调
3. **执行任务**: Agent 处理任务并生成响应
4. **评估响应**: LLM 判断响应是否值得通知用户
5. **投递结果**: 如果 `deliver=True` 且评估通过，发送消息到用户渠道

### 设计亮点

- ✅ **自动化**: 用户无需手动配置渠道信息
- ✅ **智能化**: LLM 评估减少无效通知
- ✅ **安全性**: 评估失败时默认通知 (fail-safe)
- ✅ **灵活性**: 支持多种调度模式 (at/cron/every)
- ✅ **渠道感知**: 自动发送回原始创建渠道

---

**报告完成** 🐈