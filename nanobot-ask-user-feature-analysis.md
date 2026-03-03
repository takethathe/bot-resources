# nanobot Ask User 功能分析报告

**分析日期**: 2026-03-03  
**nanobot 版本**: v0.1.4.post3

---

## 📋 结论

### ❌ 当前状态

**nanobot 目前没有专门的 "ask user" 功能**，即没有内置的工具或 API 用于：
- 主动暂停任务并等待用户输入
- 收集用户必要信息（表单式交互）
- 交互式确认步骤

---

## 🔍 代码分析

### 当前架构

```
用户消息 → InboundMessage → Agent Loop → 处理任务 → 发送回复
```

**特点**:
- ✅ 被动响应模式 - 等待用户消息触发
- ✅ 消息工具 - 可发送消息给用户
- ❌ 无等待机制 - 无法暂停任务等待用户输入
- ❌ 无表单工具 - 无结构化信息收集

### 相关代码

#### 1. 消息工具 (`nanobot/agent/tools/message.py`)

```python
class MessageTool(Tool):
    """Tool to send messages to users on chat channels."""
    
    @property
    def name(self) -> str:
        return "message"
    
    @property
    def description(self) -> str:
        return "Send a message to the user. Use this when you want to communicate something."
```

**功能**: 只能发送消息，无法等待回复。

#### 2. Agent Loop (`nanobot/agent/loop.py`)

```python
while self._running:
    try:
        msg = await asyncio.wait_for(self.bus.consume_inbound(), timeout=1.0)
    except asyncio.TimeoutError:
        continue
    
    # 处理消息...
```

**特点**: 持续轮询消息，无暂停/等待机制。

---

## 💡 替代方案

### 方案 1: 使用 LLM 提示词（推荐）

让 LLM 在需要用户信息时主动询问：

```
系统提示词示例:
"如果需要用户提供更多信息才能完成任务，请直接询问用户。
等待用户回复后再继续处理。"
```

**优点**:
- ✅ 无需代码修改
- ✅ 自然对话流程
- ✅ 适用于所有渠道

**缺点**:
- ❌ 无法强制暂停任务
- ❌ 用户可能不理解需要回复

---

### 方案 2: 使用飞书交互式卡片

通过卡片按钮收集用户选择：

```json
{
  "elements": [
    {
      "tag": "action",
      "actions": [
        {
          "tag": "button",
          "text": {"content": "确认"},
          "value": {"action": "confirm"}
        }
      ]
    }
  ]
}
```

**状态**: ❌ nanobot 当前不支持卡片回调（见 `feishu-card-interaction-test.md`）

---

### 方案 3: 使用 MCP 服务器

通过 MCP 实现自定义输入收集：

```json
{
  "tools": {
    "mcpServers": {
      "user-input": {
        "command": "npx",
        "args": ["-y", "@mcp/user-input-server"]
      }
    }
  }
}
```

**状态**: ⚠️ 需要自行开发 MCP 服务器

---

## 🎯 实现建议

### 如需完整 Ask User 功能

需要添加以下内容：

#### 1. 新工具：`ask_user`

```python
class AskUserTool(Tool):
    """Tool to ask user for information and wait for response."""
    
    @property
    def name(self) -> str:
        return "ask_user"
    
    @property
    def description(self) -> str:
        return "Ask the user for information and wait for their response."
    
    @property
    def parameters(self) -> dict:
        return {
            "type": "object",
            "properties": {
                "question": {
                    "type": "string",
                    "description": "The question to ask the user"
                },
                "options": {
                    "type": "array",
                    "items": {"type": "string"},
                    "description": "Optional: predefined options for quick selection"
                },
                "required_field": {
                    "type": "string",
                    "description": "The field name this information will populate"
                }
            },
            "required": ["question"]
        }
    
    async def execute(self, question: str, options: list[str] = None, **kwargs):
        # 发送消息
        await self._send_message(question)
        
        # 等待用户回复
        response = await self._wait_for_user_response()
        
        return response
```

#### 2. Agent Loop 修改

```python
async def _dispatch(self, msg: InboundMessage):
    # 检查是否有等待中的 ask_user 请求
    pending = await self._get_pending_ask_user(msg.session_key)
    if pending:
        # 将用户回复传递给等待的任务
        pending.set_result(msg.content)
        return
    
    # 正常处理...
```

#### 3. 会话状态管理

```python
class SessionState:
    def __init__(self):
        self.pending_ask: asyncio.Future | None = None
        self.collected_data: dict = {}
```

---

## 📊 功能对比

| 功能 | 当前状态 | 需要实现 |
|------|----------|----------|
| 发送消息 | ✅ 支持 | - |
| 接收消息 | ✅ 支持 | - |
| 暂停任务等待回复 | ❌ 不支持 | 会话状态管理 |
| 结构化表单输入 | ❌ 不支持 | ask_user 工具 |
| 按钮选项选择 | ❌ 不支持 | 卡片回调 + ask_user |
| 必填字段验证 | ❌ 不支持 | 工具逻辑 |
| 多轮对话收集 | ❌ 不支持 | 会话状态管理 |

---

## 🚀 推荐方案

### 短期（无需代码修改）

使用 LLM 提示词让 bot 主动询问：

```
提示词示例:
"在完成任务前，如果需要用户提供的信息，请清晰地列出需要的内容。
例如：'为了帮您完成这个任务，我需要以下信息：
1. XXX
2. XXX
请回复提供这些信息。'"
```

### 中期（功能增强）

实现 `ask_user` 工具：
1. 添加会话状态管理
2. 实现等待/恢复机制
3. 支持结构化输入

### 长期（完整交互）

结合飞书卡片回调：
1. 实现卡片动作事件处理
2. 支持按钮选项快速选择
3. 支持表单式信息收集

---

## 📝 相关文件

- `/root/.nanobot/workspace/feishu-card-interaction-test.md` - 飞书卡片交互测试报告
- `nanobot/agent/tools/message.py` - 消息工具源码
- `nanobot/agent/loop.py` - Agent Loop 源码

---

*报告生成时间：2026-03-03 22:58 UTC*