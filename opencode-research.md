# OpenCode 调研报告

**调研日期**: 2026-03-01  
**项目**: [anomalyco/opencode](https://github.com/anomalyco/opencode)  
**网站**: https://opencode.ai

---

## 概述

OpenCode 是一个开源 AI 编程助手，提供终端、桌面应用和 IDE 扩展多种界面。

---

## 核心特性

### 多 Agent 系统

| Agent | 类型 | 权限 | 用途 |
|-------|------|------|------|
| build | Primary | 完全权限 | 默认 Agent，负责开发任务 |
| plan | Primary | 只读 | 分析和规划，文件编辑需用户批准 |
| general | Subagent | 按需 | 复杂多步骤任务，通过 @mention 调用 |
| explore | Subagent | 只读 | 快速代码库探索 |

### Agent 切换

- **Tab 键** - 循环切换主 Agent
- **@mention** - 调用子 Agent
- **方向键** - 导航父子会话

---

## 内置工具 (15+)

- `bash` - 执行 shell 命令
- `edit` / `write` / `read` - 文件操作
- `grep` / `glob` / `list` - 搜索和浏览
- `lsp` - 语言服务器支持
- `patch` - 代码补丁
- `skill` - 技能调用
- `todo_write` / `todo_read` - 任务管理
- `web_fetch` / `web_search` - 网络功能
- `ask_user` - 用户交互

---

## TUI 命令

| 命令 | 快捷键 | 功能 |
|------|--------|------|
| /init | Ctrl+X I | 初始化项目 |
| /connect | Ctrl+X C | 连接会话 |
| /undo | Ctrl+X U | 撤销 |
| /redo | Ctrl+X R | 重做 |
| /share | Ctrl+X S | 分享会话 |
| /compact | Ctrl+X P | 压缩上下文 |
| /export | Ctrl+X E | 导出会话 |
| /new | Ctrl+X N | 新会话 |
| /list | Ctrl+X L | 查看会话列表 |
| /help | Ctrl+X H | 帮助 |
| /exit | Ctrl+X Q | 退出 |

---

## 配置系统

### 5 级优先级 (高→低)

1. **Remote** - 远程配置
2. **Global** - `~/.config/opencode/opencode.json`
3. **Env** - `OPENCODE_CONFIG` 环境变量
4. **Project** - `opencode.json`
5. **Directory** - `.opencode/` 目录

### 配置格式

- **JSON** - `opencode.json`
- **Markdown** - `.opencode/agents/*.md`

---

## LSP 支持

- **30+ 语言** 支持
- **自动安装**: TypeScript, Python, C/C++, PHP
- **功能**: goToDefinition, findReferences, hover, documentSymbol, workspaceSymbol

---

## MCP 扩展

支持本地和远程 MCP 服务器:
- Sentry
- Context7
- Grep by Vercel

---

## 规则系统

- **AGENTS.md** - 项目/全局规则
- **CLAUDE.md** 兼容

---

## 安装方式

| 方式 | 命令 |
|------|------|
| curl | `curl -L https://opencode.ai/install \| bash` |
| Homebrew | `brew install anomalyco/tap/opencode` |
| npm | `npm install -g opencode` |
| Docker | `docker run -it opencode` |
| Desktop | macOS / Windows / Linux |

### 桌面应用

- **macOS**: Apple Silicon / Intel (cask)
- **Windows**: .exe 安装包
- **Linux**: .deb / .rpm / AppImage

---

## 与 Claude Code 对比

| 特性 | OpenCode | Claude Code |
|------|----------|-------------|
| 开源 | 是 (100%) | 否 (闭源) |
| 多 Provider | 支持 (Claude/OpenAI/Google/本地) | 仅 Claude |
| LSP 支持 | 内置 | 无 |
| 界面 | TUI 为主 | 终端 |
| 架构 | Client/Server + 远程访问 | 本地 |

---

## 总结

### 优势

- 完全开源，透明可控
- 多模型支持，灵活选择
- 内置 LSP，代码理解强
- TUI 设计，终端友好
- 支持远程访问和协作

### 适用场景

- 需要本地化部署的团队
- 多模型切换需求
- 终端重度用户
- 需要代码导航和 LSP 功能

---

## 相关链接

- **GitHub**: https://github.com/anomalyco/opencode
- **官网**: https://opencode.ai
- **文档**: https://opencode.ai/docs
