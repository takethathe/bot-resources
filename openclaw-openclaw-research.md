# GitHub 项目调研报告

**调研时间**: 2026-03-02 06:41:39

**项目**: [openclaw/openclaw](https://github.com/openclaw/openclaw)


## 📌 项目概览

- **名称**: openclaw
- **描述**: Your own personal AI assistant. Any OS. Any Platform. The lobster way. 🦞 
- **Stars**: 244480 ⭐
- **Forks**: 47264 🔱
- **许可证**: N/A
- **主要语言**: TypeScript
- **创建时间**: 2025-11-24
- **更新时间**: 2026-03-02

## 📁 文件结构

```

📁 .agent/
  📁 workflows/
    📄 update_clawdbot.md (8.8KB)
📁 .agents/
  📄 maintainers.md
📁 .github/
  📁 actions/
    📁 detect-docs-changes/
      📄 action.yml (2.0KB)
    📁 setup-node-env/
      📄 action.yml (2.7KB)
    📁 setup-pnpm-store-cache/
      📄 action.yml (1.5KB)
  📁 instructions/
    📄 copilot.instructions.md (2.1KB)
  📁 ISSUE_TEMPLATE/
    📄 bug_report.yml (3.5KB)
    📄 config.yml
    📄 feature_request.yml (2.5KB)
  📁 workflows/
    📄 auto-response.yml (17.1KB)
    📄 ci.yml (25.3KB)
    📄 docker-release.yml (7.3KB)
    📄 install-smoke.yml (2.0KB)
    📄 labeler.yml (18.9KB)
    📄 sandbox-common-smoke.yml (1.6KB)
    📄 workflow-sanity.yml (2.0KB)
  📄 actionlint.yaml
  📄 dependabot.yml (2.5KB)
  📄 FUNDING.yml
  📄 labeler.yml (7.0KB)
  📄 pull_request_template.md (1.9KB)
📁 .pi/
  📁 extensions/
    📄 diff.ts (6.1KB)
    📄 files.ts (6.1KB)
    📄 prompt-url-widget.ts (5.2KB)
    📄 redraws.ts
  📁 git/
    📄 .gitignore
  📁 prompts/
    📄 cl.md (1.9KB)
    📄 is.md
    📄 landpr.md (2.8KB)
    📄 reviewpr.md (3.7KB)
📁 .vscode/
  📄 extensions.json
  📄 settings.json
📁 apps/
  📁 android/
```


## 📊 代码统计

| 语言 | 文件数 | 总行数 | 空行 | 注释行 |
|------|--------|--------|------|--------|
| TS | 5054 | 940,519 | 83970 | 21685 |
| SWIFT | 519 | 89,735 | 9475 | 2143 |
| KT | 111 | 22,796 | 2098 | 298 |
| JS | 19 | 6,630 | 595 | 343 |
| GO | 14 | 1,840 | 185 | 5 |
| PY | 11 | 1,825 | 332 | 81 |
| **总计** | **5728** | **1,063,345** | - | - |

## 📦 依赖分析

### NODEJS

- **依赖数量**: 57
- **项目名称**: openclaw
- **版本**: 2026.3.2

## 🔄 最近提交

| Commit | 信息 | 日期 |
|--------|------|------|
| `d0ac1b0` | feat: add PDF analysis tool with native  | 2026-03-01 |

## 📖 README 摘要

# 🦞 OpenClaw — Personal AI Assistant

<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png">
        <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500">
    </picture>
</p>

<p align="center">
  <strong>EXFOLIATE! EXFOLIATE!</strong>
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw/actions/workflows/ci.yml?branch=main"><img src="https://img.shields.io/github/actions/workflow/status/openclaw/openclaw/ci.yml?branch=main&style=for-the-badge" alt="CI status"></a>
  <a href="https://github.com/openclaw/openclaw/releases"><img src="https://img.shields.io/github/v/release/openclaw/openclaw?include_prereleases&style=for-the-badge" alt="GitHub release"></a>
  <a href="https://discord.gg/clawd"><img src="https://img.shields.io/discord/1456350064065904867?lab

## 💡 总结

### 项目评价

- ⭐⭐⭐⭐⭐ 超热门项目，社区活跃度高
- **代码规模**: 106 万行代码，5728 个文件
- **主要语言**: TypeScript (88.5%) + Swift (8.4%) + Kotlin (2.1%)
- **版本**: 2026.3.2
- **许可证**: MIT License

### 适用场景

- **个人 AI 助手** - 全平台个人 AI 助手，支持 30+ 通信通道
- **多通道集成** - WhatsApp、Telegram、Slack、Discord、飞书、钉钉等
- **移动设备集成** - iOS/Android 原生应用，支持语音和位置服务
- **可视化交互** - Canvas 可视化工作区，A2UI 声明式 UI
- **语音交互** - Voice Wake 语音唤醒，Talk Mode 连续对话
- **高级自动化** - Cron 定时任务、Webhooks、浏览器控制
- **Skills 扩展** - ClawHub 技能注册表，可扩展功能
- **企业部署** - 沙箱模式、权限管理、多会话隔离
- **远程访问** - Tailscale 集成，安全远程访问

### 核心优势

1. **全平台支持** - macOS、Linux、Windows、iOS、Android
2. **30+ 通信通道** - 覆盖全球主流通信平台
3. **Canvas 可视化** - AI 智能体控制的交互式工作区
4. **Voice Wake** - 语音唤醒和连续对话 (macOS/iOS/Android)
5. **本地优先** - Gateway 运行在用户设备上，数据隐私
6. **Skills 系统** - 可扩展的技能平台，ClawHub 注册表
7. **企业级安全** - Docker 沙箱、权限管理、DM 配对
8. **成熟生态** - 24 万 + stars，活跃社区

### 注意事项

- 需要 Node.js ≥22
- 配置相对复杂，学习曲线陡峭
- 仅支持云模型，无本地模型支持
- 文档以英文为主，中文支持有限
- 资源占用较高 (Node.js + 多应用)

---

*报告生成时间：2026-03-02 06:41:39*  
*补全时间：2026-03-02 06:57*