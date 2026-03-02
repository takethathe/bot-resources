# OpenCode 调研报告 📋

**调研日期**: 2026-03-01  
**项目**: [anomalyco/opencode](https://github.com/anomalyco/opencode)  
**网站**: https://opencode.ai

---

## 📌 概述

OpenCode 是一个开源 AI 编程助手，提供终端、桌面应用和 IDE 扩展多种界面。

---

## 🎯 核心特性

### 多 Agent 系统

| Agent | 类型 | 权限 | 用途 |
|-------|------|------|------|
|  | Primary | 完全权限 | 默认 Agent，负责开发任务 |
|  | Primary | 只读 | 分析和规划，文件编辑需用户批准 |
|  | Subagent | 按需 | 复杂多步骤任务，通过 @mention 调用 |
|  | Subagent | 只读 | 快速代码库探索 |

### Agent 切换

- **Tab 键** - 循环切换主 Agent
- **@mention** - 调用子 Agent
- **方向键** - 导航父子会话

---

## 🛠️ 内置工具 (15+)

-  - 执行 shell 命令
-  /  /  - 文件操作
-  /  /  - 搜索和浏览
-  - 语言服务器支持
-  - 代码补丁
-  - 技能调用
-  /  - 任务管理
-  /  - 网络功能
-  - 用户交互

---

## 💻 TUI 命令

| 命令 | 快捷键 | 功能 |
|------|--------|------|
|  | Ctrl+X I | 初始化项目 |
|  | Ctrl+X C | 连接会话 |
|  | Ctrl+X U | 撤销 |
|  | Ctrl+X R | 重做 |
|  | Ctrl+X S | 分享会话 |
|  | Ctrl+X P | 压缩上下文 |
|  | Ctrl+X E | 导出会话 |
|  | Ctrl+X N | 新会话 |
|  | Ctrl+X L | 查看会话列表 |
|  | Ctrl+X H | 帮助 |
|  | Ctrl+X Q | 退出 |

---

## ⚙️ 配置系统

### 5 级优先级 (高→低)

1. **Remote** - 远程配置
2. **Global** - 
3. **Env** -  环境变量
4. **Project** - 
5. **Directory** -  目录

### 配置格式

- **JSON** - 
- **Markdown** - 

---

## 🔌 LSP 支持

- **30+ 语言** 支持
- **自动安装**: TypeScript, Python, C/C++, PHP
- **功能**: goToDefinition, findReferences, hover, documentSymbol, workspaceSymbol

---

## 🔗 MCP 扩展

支持本地和远程 MCP 服务器:
- Sentry
- Context7
- Grep by Vercel

---

## 📜 规则系统

- **AGENTS.md** - 项目/全局规则
- **CLAUDE.md** 兼容

---

## 📦 安装方式

| 方式 | 命令 |
|------|------|
| curl | [0m
[0;2mInstalling [0mopencode [0;2mversion: [0m1.2.15[0m
[0mCommand already exists in /root/.bashrc, skipping write.[0m

[0;2m                    [0m             ▄     
[0;2m█▀▀█ █▀▀█ █▀▀█ █▀▀▄ [0m█▀▀▀ █▀▀█ █▀▀█ █▀▀█
[0;2m█░░█ █░░█ █▀▀▀ █░░█ [0m█░░░ █░░█ █░░█ █▀▀▀
[0;2m▀▀▀▀ █▀▀▀ ▀▀▀▀ ▀  ▀ [0m▀▀▀▀ ▀▀▀▀ ▀▀▀▀ ▀▀▀▀


[0;2mOpenCode includes free models, to start:[0m

cd <project>  [0;2m# Open directory[0m
opencode      [0;2m# Run command[0m

[0;2mFor more information visit [0mhttps://opencode.ai/docs |
| Homebrew |  |
| npm |  |
| Docker |  |
| Desktop | macOS / Windows / Linux |

### 桌面应用

- **macOS**: Apple Silicon / Intel (cask)
- **Windows**: .exe 安装包
- **Linux**: .deb / .rpm / AppImage

---

## 🆚 vs Claude Code

| 特性 | OpenCode | Claude Code |
|------|----------|-------------|
| 开源 | ✅ 100% | ❌ 闭源 |
| 多 Provider | ✅ Claude/OpenAI/Google/本地 | ❌ 仅 Claude |
| LSP 支持 | ✅ 内置 | ❌ 无 |
| 界面 | TUI 为主 | 终端 |
| 架构 | Client/Server + 远程访问 | 本地 |

---

## 💡 总结

**优势**:
- ✅ 完全开源，透明可控
- ✅ 多模型支持，灵活选择
- ✅ 内置 LSP，代码理解强
- ✅ TUI 设计，终端友好
- ✅ 支持远程访问和协作

**适用场景**:
- 需要本地化部署的团队
- 多模型切换需求
- 终端重度用户
- 需要代码导航和 LSP 功能

---

## 🔗 相关链接

- GitHub: https://github.com/anomalyco/opencode
- 官网: https://opencode.ai
- 文档: https://opencode.ai/docs
