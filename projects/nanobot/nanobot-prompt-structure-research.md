# nanobot 提示词结构分析

**调研日期**: 2026-03-14  
**nanobot 版本**: v3.12 (Python 3.12.12)  
**分析范围**: `/usr/local/lib/python3.12/site-packages/nanobot/agent/`

---

## 📋 提示词组成部分

送入大模型的完整提示词由以下部分组成：

```
┌─────────────────────────────────────────────────────────┐
│  1. System Message (系统提示词)                          │
│     ┌───────────────────────────────────────────────┐   │
│     │ ① Identity (身份定义)                          │   │
│     │ ② Bootstrap Files (引导文件)                   │   │
│     │ ③ Memory (长期记忆)                            │   │
│     │ ④ Always Skills (常驻技能)                     │   │
│     │ ⑤ Skills Summary (技能列表)                    │   │
│     └───────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────┤
│  2. Conversation History (对话历史)                      │
│     - User / Assistant / Tool 消息                       │
├─────────────────────────────────────────────────────────┤
│  3. Runtime Context (运行时上下文)                       │
│     - 时间、渠道、Chat ID                                │
├─────────────────────────────────────────────────────────┤
│  4. Current User Message (当前用户消息)                  │
│     - 文本 + 可选图片                                     │
└─────────────────────────────────────────────────────────┘
```

---

## 🔢 先后顺序

### 完整消息列表结构

```python
[
    # 第 1 条：System Message
    {
        "role": "system",
        "content": build_system_prompt()  # 包含 5 个子部分
    },
    
    # 第 2 到 N 条：对话历史
    *history,  # 从 Session 加载的未巩固消息
    
    # 第 N+1 条：运行时上下文 (元数据)
    {
        "role": "user",
        "content": "[Runtime Context — metadata-only, not instructions]\n"
                   "Current Time: 2026-03-14 03:51 (Saturday) (UTC)\n"
                   "Channel: feishu\n"
                   "Chat ID: ou_62dcc4b3b169d86b43f51c114349257c"
    },
    
    # 第 N+2 条：当前用户消息
    {
        "role": "user",
        "content": "用户输入文本"  # 或 [图片列表，文本]
    }
]
```

---

## 📖 详细分解

### 1️⃣ System Message (系统提示词)

由 `ContextBuilder.build_system_prompt()` 构建。

### 2️⃣ Conversation History (对话历史)

由 `Session.get_history()` 提供。

### 3️⃣ Runtime Context (运行时上下文)

由 `ContextBuilder._build_runtime_context()` 构建。

### 4️⃣ Current User Message (当前用户消息)

由 `ContextBuilder._build_user_content()` 构建。

---

## 🔄 消息迭代流程

在 `AgentLoop._run_agent_loop()` 中循环调用 LLM。

---

## 📊 代码位置汇总

| 组件 | 文件 | 方法 |
|------|------|------|
| **系统提示词** | `agent/context.py` | `build_system_prompt()` |
| **身份定义** | `agent/context.py` | `_get_identity()` |
| **引导文件** | `agent/context.py` | `_load_bootstrap_files()` |
| **记忆上下文** | `agent/memory.py` | `get_memory_context()` |
| **技能列表** | `agent/skills.py` | `build_skills_summary()` |
| **运行时上下文** | `agent/context.py` | `_build_runtime_context()` |
| **用户消息** | `agent/context.py` | `_build_user_content()` |
| **消息构建** | `agent/context.py` | `build_messages()` |
| **对话历史** | `session/manager.py` | `Session.get_history()` |
| **工具结果** | `agent/context.py` | `add_tool_result()` |
| **助手消息** | `agent/context.py` | `add_assistant_message()` |

---

## ✅ 总结

nanobot 的提示词结构清晰，遵循 **System → History → Runtime → User** 的顺序。

---

**数据来源**: nanobot v3.12 源码分析
