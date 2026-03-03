# GitHub 项目调研报告

**调研时间**: 2026-03-03 03:48:20

**项目**: [microclaw/microclaw](https://github.com/microclaw/microclaw)


## 📌 项目概览

- **名称**: microclaw
- **描述**: 🦀An agentic AI assistant that lives in your chats, inspired by nanoclaw and incorporating some of its design ideas. Built with Rust 🦀
- **Stars**: 444 ⭐
- **Forks**: 78 🔱
- **许可证**: N/A
- **主要语言**: Rust
- **创建时间**: 2026-02-07
- **更新时间**: 2026-03-03

## 📁 文件结构

```

📁 .github/
  📁 workflows/
    📄 ci.yml (1.8KB)
    📄 nightly-stability.yml (2.6KB)
    📄 release-assets.yml (4.8KB)
📁 crates/
  📁 microclaw-app/
    📁 src/
      📄 builtin_skills.rs (8.4KB)
      📄 lib.rs
      📄 logging.rs (7.7KB)
      📄 transcribe.rs (1.3KB)
    📄 Cargo.toml
  📁 microclaw-channels/
    📁 src/
      📄 channel.rs (8.0KB)
      📄 channel_adapter.rs (3.0KB)
      📄 delivery.rs
      📄 lib.rs
    📄 Cargo.toml
  📁 microclaw-clawhub/
    📁 src/
      📄 client.rs (5.7KB)
      📄 gate.rs (3.9KB)
      📄 install.rs (5.1KB)
      📄 lib.rs
      📄 lockfile.rs (3.4KB)
      📄 types.rs (7.4KB)
    📄 Cargo.toml
  📁 microclaw-core/
    📁 src/
      📄 error.rs (2.0KB)
      📄 lib.rs
      📄 llm_types.rs (8.3KB)
      📄 text.rs
    📄 Cargo.toml
  📁 microclaw-storage/
    📁 src/
      📄 db.rs (176.8KB)
      📄 lib.rs
      📄 memory.rs (7.5KB)
      📄 memory_quality.rs (6.4KB)
      📄 usage.rs (6.9KB)
    📄 Cargo.toml
  📁 microclaw-tools/
    📁 src/
      📄 command_runner.rs (1.6KB)
      📄 env_file.rs (2.2KB)
      📄 lib.rs
      📄 path_guard.rs (12.0KB)
```


## 📊 代码统计

| 语言 | 文件数 | 总行数 | 空行 | 注释行 |
|------|--------|--------|------|--------|
| RS | 106 | 65,866 | 5286 | 2551 |
| TS | 4 | 49 | 4 | 0 |
| **总计** | **110** | **65,915** | - | - |

## 📦 依赖分析

### Rust Workspace 结构

```toml
[workspace]
members = [
    ".",
    "crates/microclaw-core",
    "crates/microclaw-clawhub",
    "crates/microclaw-storage",
    "crates/microclaw-tools",
    "crates/microclaw-channels",
    "crates/microclaw-app",
]
resolver = "2"

[package]
name = "microclaw"
version = "0.0.133"
edition = "2021"
license = "MIT"
default-run = "microclaw"

[features]
default = []
sqlite-vec = ["microclaw-storage/sqlite-vec"]
```

### 主要依赖库

| 依赖类别 | 主要库 |
|---------|--------|
| **异步运行时** | tokio, async-trait |
| **HTTP 客户端** | reqwest, hyper |
| **序列化** | serde, serde_json |
| **数据库** | rusqlite, sqlite-vec (可选) |
| **日志** | tracing, tracing-subscriber |
| **错误处理** | thiserror, anyhow |
| **LLM API** | Anthropic SDK, OpenAI-compatible |
| **平台 SDK** | teloxide (Telegram), serenity (Discord) 等 |

## 🔄 最近提交

| Commit | 信息 | 日期 |
|--------|------|------|
| `4514fa7` | update readme | 2026-03-02 |

## 📖 README 摘要

### 项目简介

**MicroClaw** 是一个专为聊天环境设计的 agentic AI 助理，受 [nanoclaw](https://github.com/gavrielc/nanoclaw/) 启发并融合其设计思想。采用 Rust 构建，具有高性能和内存安全特性。

**核心架构**：渠道无关的核心 + 平台适配器，支持多平台和多 LLM 提供商。

### 支持平台

Telegram, Discord, Slack, Feishu/Lark, Matrix, WhatsApp, iMessage, Email, Nostr, Signal, DingTalk, QQ, IRC, Web

### 核心特性

| 特性 | 说明 |
|------|------|
| **Agentic 工具使用** | bash 命令、文件读/写/编辑、glob 搜索、正则 grep、持久记忆 |
| **会话恢复** | 完整对话状态（包括工具交互）在消息间持久化 |
| **上下文压缩** | 旧消息自动摘要以保持上下文限制内 |
| **子代理** | 将独立任务委托给并行代理执行 |
| **技能系统** | 兼容 Anthropic Skills 标准，自动发现和激活 |
| **计划与执行** | todo 列表工具，逐步分解复杂任务 |
| **Web 搜索** | 通过 DuckDuckGo 搜索和抓取网页 |
| **定时任务** | 基于 cron 的循环和一次性任务 |
| **持久记忆** | AGENTS.md 文件在全局、bot/账户、每聊天范围加载 |
| **消息分割** | 长响应自动分割以适应渠道限制 |

### 核心工具 (18 种)

- `bash` - 执行 shell 命令
- `read_file` / `write_file` / `edit_file` - 文件操作
- `glob` / `grep` - 文件搜索
- `read_memory` / `write_memory` - 持久记忆操作
- `web_search` / `web_fetch` - 网络搜索和抓取
- `send_message` - 发送消息（支持附件）
- `schedule_task` / `list_scheduled_tasks` / `pause_scheduled_task` / `resume_scheduled_task` / `cancel_scheduled_task` - 任务调度
- `sub_agent` - 子代理委托
- `activate_skill` / `sync_skills` - 技能管理
- `todo_read` / `todo_write` - 任务列表
- `export_chat` - 导出聊天历史

### 内置技能

pdf, docx, xlsx, pptx, skill-creator, apple-notes, apple-reminders, apple-calendar, weather, find-skills

### 记忆系统架构

```
<data_dir>/runtime/groups/
    AGENTS.md                 # 全局记忆（所有聊天共享）
    {channel}/
        AGENTS.md             # 渠道 bot/账户记忆
        {chat_id}/
            AGENTS.md         # 每聊天记忆
```

- SQLite 结构化存储 + 可选语义嵌入（sqlite-vec）
- 后台反射器增量提取持久事实
- 质量过滤和低质量记忆归档

### 安装方式

**一键安装（推荐）**：
```bash
curl -fsSL https://microclaw.ai/install.sh | bash
```

**Homebrew (macOS)**：
```bash
brew tap microclaw/tap
brew install microclaw
```

**源码编译**：
```bash
git clone https://github.com/microclaw/microclaw.git
cd microclaw
cargo build --release
cp target/release/microclaw /usr/local/bin/
```

### 配置与运行

```bash
# 诊断检查
microclaw doctor

# 启动服务
microclaw start

# Docker 沙盒诊断
microclaw doctor sandbox
```

## 💡 总结

### 项目评价

| 维度 | 评分 | 说明 |
|------|------|------|
| **活跃度** | ⭐⭐⭐⭐⭐ | 持续开发中，最近更新 2026-03-02 |
| **社区规模** | ⭐⭐⭐ | 444 Stars, 78 Forks，增长中 |
| **文档质量** | ⭐⭐⭐⭐⭐ | 详细的 README + 中文文档 + 架构图 |
| **代码质量** | ⭐⭐⭐⭐ | Rust 编写，模块化设计，CI/CD 完善 |
| **生态系统** | ⭐⭐⭐⭐ | 支持 13+ 平台，兼容 Anthropic Skills |

### 适用场景

✅ **推荐场景**：
- 需要在多个聊天平台部署统一的 AI 助理
- 需要持久记忆和跨会话上下文
- 需要执行 shell 命令、文件操作等系统级工具
- 需要定时任务和技能扩展系统
- 追求高性能和低资源占用（Rust 优势）

⚠️ **注意事项**：
- 项目处于活跃开发阶段，功能可能变化
- 需要一定的 Rust 知识进行二次开发
- 部分平台适配器可能需要额外配置

### 与 nanoclaw/nanobot 对比

| 特性 | nanoclaw/nanobot | microclaw |
|------|------------------|-----------|
| **语言** | Python | Rust 🦀 |
| **性能** | 中等 | 高 |
| **内存安全** | 依赖 GC | 编译时保证 |
| **平台支持** | Telegram, Discord, Feishu 等 | 13+ 平台 |
| **记忆系统** | MEMORY.md + HISTORY.md | SQLite + AGENTS.md |
| **技能系统** | 自定义 | Anthropic Skills 兼容 |
| **部署** | 简单 | 一键安装/源码编译 |

### 相关链接

- 🌐 **官网**: [microclaw.ai](https://microclaw.ai)
- 📚 **中文文档**: [README_CN.md](https://github.com/microclaw/microclaw/blob/main/README_CN.md)
- 💬 **Discord**: [加入社区](https://discord.gg/pvmezwkAk5)
- 📝 **博客**: [Building MicroClaw](https://microclaw.ai/blog/building-microclaw)
- 📦 **许可证**: MIT

---

*报告生成时间：2026-03-03 03:48:20*