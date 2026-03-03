# 飞书交互式卡片按钮回调测试报告

**测试日期**: 2026-03-03  
**nanobot 版本**: v0.1.4.post3  
**测试频道**: 飞书 (Feishu)

---

## 📋 测试结果

### ✅ 已支持功能

| 功能 | 状态 | 说明 |
|------|------|------|
| 交互式卡片发送 | ✅ 支持 | 通过 `interactive` 消息类型 |
| 卡片元素渲染 | ✅ 支持 | div、markdown、table、hr 等 |
| 按钮显示 | ✅ 支持 | 按钮可正常显示在卡片上 |
| 卡片配置 | ✅ 支持 | wide_screen_mode、header 等 |

### ❌ 未支持功能

| 功能 | 状态 | 说明 |
|------|------|------|
| 按钮点击回调 | ❌ 未实现 | 未注册 `P2ImMessageCardActionV1` 事件 |
| 按钮 value 处理 | ❌ 未实现 | 无法获取按钮点击的 value 数据 |
| 交互响应 | ❌ 未实现 | 点击后无法触发 bot 响应 |

---

## 🔍 代码分析

### 当前事件注册

```python
# nanobot/channels/feishu.py
event_handler = lark.EventDispatcherHandler.builder(
    self.config.encrypt_key or "",
    self.config.verification_token or "",
).register_p2_im_message_receive_v1(
    self._on_message_sync
).build()
```

**问题**: 只注册了消息接收事件 (`P2ImMessageReceiveV1`)，未注册卡片动作事件。

### 需要添加的事件注册

```python
# 需要添加
event_handler = lark.EventDispatcherHandler.builder(
    self.config.encrypt_key or "",
    self.config.verification_token or "",
).register_p2_im_message_receive_v1(
    self._on_message_sync
).register_p2_im_message_card_action_v1(  # ← 新增
    self._on_card_action_sync
).build()
```

### 需要实现的回调处理器

```python
def _on_card_action_sync(self, data: "P2ImMessageCardActionV1") -> None:
    """处理卡片按钮点击回调。"""
    if self._loop and self._loop.is_running():
        asyncio.run_coroutine_threadsafe(self._on_card_action(data), self._loop)

async def _on_card_action(self, data: "P2ImMessageCardActionV1") -> None:
    """异步处理卡片按钮点击。"""
    event = data.event
    action = event.action  # 按钮的 value
    message = event.message
    user = event.user
    
    # 解析按钮 value
    action_value = json.loads(action.value) if action.value else {}
    
    # 根据 action 执行不同逻辑
    if action_value.get("action") == "like":
        await self._handle_like(message, user)
    elif action_value.get("action") == "feedback":
        await self._handle_feedback(message, user)
    # ...
```

---

## 📝 飞书卡片按钮回调流程

```
用户点击按钮
    ↓
飞书服务器发送 POST 请求
    ↓
nanobot WebSocket 接收 P2ImMessageCardActionV1 事件
    ↓
_on_card_action_sync() 处理
    ↓
_on_card_action() 异步处理
    ↓
根据 action.value 执行相应逻辑
    ↓
可选：更新卡片/发送回复消息
```

---

## 🎯 按钮 value 格式示例

```json
{
  "action": "like",
  "timestamp": "2026-03-03T22:52:00Z",
  "user_id": "ou_xxx"
}
```

---

## 🔧 实现建议

### 1. 注册卡片动作事件

在 `FeishuChannel.start()` 方法中添加：

```python
.register_p2_im_message_card_action_v1(
    self._on_card_action_sync
)
```

### 2. 实现回调处理器

添加 `_on_card_action_sync()` 和 `_on_card_action()` 方法。

### 3. 定义动作处理逻辑

```python
async def _handle_card_action(self, action: str, user_id: str, message_id: str):
    """根据动作类型处理。"""
    handlers = {
        "like": self._handle_like,
        "feedback": self._handle_feedback,
        "help": self._handle_help,
    }
    handler = handlers.get(action)
    if handler:
        await handler(user_id, message_id)
```

### 4. 更新卡片（可选）

使用 CardKit API 更新原卡片：

```python
# 更新卡片内容
update_request = UpdateMessageRequest.builder()...
```

---

## 📚 参考文档

- [飞书开放平台 - 交互式卡片](https://open.feishu.cn/document/ukTMzUjL4YDM14iN2ATN)
- [飞书开放平台 - 卡片回调事件](https://open.feishu.cn/document/ukTMzUjL4UDM14iN1ATN)
- [lark SDK - EventDispatcherHandler](https://github.com/larksuite/lark-py)

---

## ✅ 结论

当前 nanobot 飞书渠道**支持发送交互式卡片**，但**不支持按钮点击回调**。

如需完整交互功能，需要：
1. 注册 `P2ImMessageCardActionV1` 事件
2. 实现回调处理器
3. 添加动作处理逻辑

---

*测试报告生成时间：2026-03-03 22:52 UTC*