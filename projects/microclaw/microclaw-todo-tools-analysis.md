# microclaw Todo Tools 实现原理分析

# Microclaw Todo Tools Analysis

**Generated**: 2026-03-04 07:22:44 UTC  
**Type**: analysis  
**Category**: projects/microclaw  
**Topic**: microclaw-todo-tools

---


**分析时间**: 2026-03-04 07:21  
**项目**: microclaw/microclaw (Rust-based AI assistant)  
**类型**: 技术实现分析

---

## 📋 概述

microclaw 的 Todo Tools 提供了一套完整的任务管理功能，允许 AI 助手为每个聊天会话维护独立的待办事项列表。

---

## 🏗️ 架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                    microclaw Todo System                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────┐         ┌──────────────────┐          │
│  │  TodoReadTool    │         │  TodoWriteTool   │          │
│  │  (todo_read)     │         │  (todo_write)    │          │
│  └────────┬─────────┘         └────────┬─────────┘          │
│           │                            │                     │
│           └─────────────┬──────────────┘                     │
│                         │                                    │
│                         ▼                                    │
│           ┌─────────────────────────┐                        │
│           │   microclaw_tools::     │                        │
│           │   todo_store            │                        │
│           │                         │                        │
│           │  - read_todos()         │                        │
│           │  - write_todos()        │                        │
│           │  - clear_todos()        │                        │
│           │  - format_todos()       │                        │
│           └────────────┬────────────┘                        │
│                        │                                     │
│                        ▼                                     │
│           ┌─────────────────────────┐                        │
│           │   File System Storage   │                        │
│           │   {data_dir}/groups/    │                        │
│           │     {channel}/          │                        │
│           │       {chat_id}/        │                        │
│           │         TODO.json       │                        │
│           └─────────────────────────┘                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 核心组件

### 1. TodoReadTool

**功能**: 读取当前聊天的待办事项列表

```rust
pub struct TodoReadTool {
    groups_dir: PathBuf,  // 数据存储目录
}

// Tool 定义
ToolDefinition {
    name: "todo_read",
    description: "Read the current todo/plan list for this chat...",
    input_schema: {
        "chat_id": {"type": "integer", "description": "The chat ID"}
    }
}
```

**执行流程**:
1. 从输入中提取 `chat_id`
2. 验证聊天访问权限 (`authorize_chat_access`)
3. 从输入中提取 `channel` (默认为 "web")
4. 调用 `read_todos()` 读取数据
5. 调用 `format_todos()` 格式化输出

---

### 2. TodoWriteTool

**功能**: 写入/更新待办事项列表（完全替换）

```rust
pub struct TodoWriteTool {
    groups_dir: PathBuf,
}

// Tool 定义
ToolDefinition {
    name: "todo_write",
    description: "Write/update the todo list for this chat. Replaces the entire list...",
    input_schema: {
        "chat_id": {"type": "integer"},
        "todos": {
            "type": "array",
            "items": {
                "task": {"type": "string"},
                "status": {"type": "string", "enum": ["pending", "in_progress", "completed"]}
            }
        }
    }
}
```

**执行流程**:
1. 验证 `chat_id` 和 `todos` 参数
2. 反序列化为 `Vec<TodoItem>`
3. 验证聊天访问权限
4. 调用 `write_todos()` 持久化数据
5. 返回确认信息和格式化列表

---

### 3. TodoItem 数据结构

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct TodoItem {
    pub task: String,    // 任务描述
    pub status: String,  // "pending" | "in_progress" | "completed"
}
```

---

### 4. todo_store 核心函数

| 函数 | 功能 | 返回值 |
|------|------|--------|
| `todo_path()` | 生成存储路径 | `PathBuf` |
| `read_todos()` | 读取待办列表 | `Vec<TodoItem>` |
| `write_todos()` | 写入待办列表 | `Result<(), io::Error>` |
| `clear_todos()` | 清空待办列表 | `Result<bool, io::Error>` |
| `format_todos()` | 格式化输出 | `String` |

---

## 📁 存储结构

```
{data_dir}/
└── groups/
    ├── web/
    │   ├── 123456/
    │   │   └── TODO.json
    │   └── 789012/
    │       └── TODO.json
    ├── telegram/
    │   └── 8281248569/
    │       └── TODO.json
    └── discord/
        └── 456789/
            └── TODO.json
```

**路径生成逻辑**:
```rust
pub fn todo_path(groups_dir: &Path, channel: &str, chat_id: i64) -> PathBuf {
    groups_dir
        .join(channel.trim())
        .join(chat_id.to_string())
        .join("TODO.json")
}
```

**TODO.json 格式**:
```json
[
  {
    "task": "Research microclaw architecture",
    "status": "completed"
  },
  {
    "task": "Implement todo tools",
    "status": "in_progress"
  },
  {
    "task": "Write documentation",
    "status": "pending"
  }
]
```

---

## 🔐 权限控制

Todo Tools 实现了跨聊天访问控制：

```rust
// 从输入中提取认证上下文
fn auth_context_from_input(input: &serde_json::Value) -> Option<AuthContext> {
    input.get("__microclaw_auth")?.as_object()?;
    // 提取 caller_chat_id, control_chat_ids 等
}

// 验证访问权限
fn authorize_chat_access(input: &Value, target_chat_id: i64) -> Result<(), String> {
    let auth = auth_context_from_input(input);
    let caller = auth.map(|a| a.caller_chat_id);
    let control = auth.map(|a| a.control_chat_ids).unwrap_or_default();
    
    // 允许访问的条件：
    // 1. 调用者就是目标聊天
    // 2. 调用者在目标聊天的控制列表中
    if caller == Some(target_chat_id) || control.contains(&target_chat_id) {
        Ok(())
    } else {
        Err("Permission denied".into())
    }
}
```

**使用场景**:
- 控制聊天可以管理被控制聊天的待办事项
- 防止未授权的跨聊天数据访问

---

## 📊 状态图标映射

```rust
pub fn format_todos(todos: &[TodoItem]) -> String {
    if todos.is_empty() {
        return "No tasks in the todo list.".into();
    }
    let mut out = String::new();
    for (i, item) in todos.iter().enumerate() {
        let icon = match item.status.as_str() {
            "completed"   => "[x]",  // 已完成
            "in_progress" => "[~]",  // 进行中
            _             => "[ ]",  // 待处理
        };
        out.push_str(&format!("{}. {} {}\n", i + 1, icon, item.task));
    }
    out
}
```

**输出示例**:
```
1. [x] Research microclaw architecture
2. [~] Implement todo tools
3. [ ] Write documentation
```

---

## 🧪 测试覆盖

Todo Tools 包含完整的单元测试：

| 测试类别 | 测试用例 |
|----------|----------|
| **序列化** | `test_todo_item_serde` |
| **格式化** | `test_format_todos_empty`, `test_format_todos` |
| **读写** | `test_read_todos_empty`, `test_write_and_read_todos` |
| **工具定义** | `test_todo_read_tool_name_and_definition`, `test_todo_write_tool_name_and_definition` |
| **执行** | `test_todo_read_empty`, `test_todo_read_missing_chat_id` |
| **权限** | `test_todo_read_permission_denied`, `test_todo_write_permission_denied` |
| **覆盖写入** | `test_todo_write_overwrites` |
| **跨聊天** | `test_todo_write_and_read_allowed_for_control_chat_cross_chat` |

---

## 🔄 工作流程

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   LLM/AI    │     │  Tool Layer  │     │  Storage    │
│   (Agent)   │     │  (Rust)      │     │  (File)     │
└──────┬──────┘     └──────┬───────┘     └──────┬──────┘
       │                   │                    │
       │ 1. 调用 todo_read │                    │
       │──────────────────>│                    │
       │                   │ 2. 读取 TODO.json  │
       │                   │───────────────────>│
       │                   │                    │
       │                   │ 3. 返回 JSON 数据   │
       │                   │<───────────────────│
       │                   │                    │
       │ 4. 格式化输出     │                    │
       │<──────────────────│                    │
       │                   │                    │
       │ 5. 调用 todo_write│                    │
       │──────────────────>│                    │
       │                   │ 6. 写入 TODO.json  │
       │                   │───────────────────>│
       │                   │                    │
       │ 7. 确认成功       │                    │
       │<──────────────────│                    │
       │                   │                    │
```

---

## 💡 设计特点

### ✅ 优点

1. **简单直观**: JSON 文件存储，易于调试和备份
2. **按聊天隔离**: 每个聊天有独立的待办列表
3. **跨平台支持**: 通过 `channel` 区分不同平台 (web/telegram/discord)
4. **权限控制**: 支持控制聊天管理被控制聊天
5. **原子写入**: 整个列表替换，避免部分更新问题
6. **完整测试**: 覆盖正常流程和边界情况

### ⚠️ 局限性

1. **文件 I/O**: 高并发下可能有性能瓶颈
2. **全量替换**: 每次写入整个列表，不适合大规模任务
3. **无版本历史**: 不支持任务历史追溯
4. **无过期机制**: 已完成任务不会自动清理

---

## 🔍 与 nanobot 对比

| 特性 | microclaw Todo | nanobot |
|------|----------------|---------|
| **实现语言** | Rust | Python |
| **存储方式** | JSON 文件 | Markdown 文件 (MEMORY.md/HISTORY.md) |
| **数据结构** | 数组 + 状态枚举 | 自由文本 |
| **权限控制** | 内置跨聊天权限 | 无 |
| **状态跟踪** | pending/in_progress/completed | 无固定格式 |
| **测试覆盖** | 完整单元测试 | 依赖手动测试 |

---

## 📝 总结

microclaw 的 Todo Tools 实现了一个**简单但完整**的任务管理系统：

1. **双工具设计**: `todo_read` 和 `todo_write` 分离，职责清晰
2. **文件存储**: 使用 JSON 文件持久化，简单可靠
3. **聊天隔离**: 按 `channel/chat_id` 组织数据
4. **权限验证**: 防止未授权访问
5. **状态管理**: 三种状态跟踪任务进度
6. **测试完备**: 覆盖各种场景

这种设计适合**个人 AI 助手**场景，平衡了功能性和实现复杂度。

---

## 📚 参考文件

- `/tmp/microclaw/src/tools/todo.rs` - Tool 实现
- `/tmp/microclaw/crates/microclaw-tools/src/todo_store.rs` - 存储逻辑