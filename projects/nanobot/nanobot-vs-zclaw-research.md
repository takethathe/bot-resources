# nanobot vs zclaw 对比研究

# Nanobot Vs Zclaw Research

**Generated**: 2026-03-03 23:42:55 UTC  
**Type**: research  
**Category**: projects/nanobot  
**Topic**: nanobot-vs-zclaw

---


**生成时间**: 2026-03-03 23:41 UTC  
**类型**: research  
**主题**: nanobot vs zclaw

---

## 📊 项目概览对比

| 指标 | nanobot | zclaw |
|------|---------|-------|
| **仓库** | [HKUDS/nanobot](https://github.com/HKUDS/nanobot) | [tnm/zclaw](https://github.com/tnm/zclaw) |
| **描述** | Ultra-Lightweight Personal AI Assistant | Smallest possible AI personal assistant for ESP32 |
| **创建时间** | 2026-02-01 | 2026-02-16 |
| **最后更新** | 2026-03-03 16:14 | 2026-03-03 06:58 |
| **Stars** | 28,288 ⭐ | 1,709 ⭐ |
| **Forks** | 4,571 🍴 | 132 🍴 |
| **Issues** | 357 | 3 |
| **Pull Requests** | 468 | 3 |
| **主语言** | Python | C |
| **最新版本** | v0.1.4.post3 (2026-02-28) | v2.10.1 (2026-03-03) |
| **许可证** | MIT | MIT |
| **主页** | - | https://zclaw.dev |

---

## 🏗️ 架构与定位

### nanobot

**定位**: 超轻量级个人 AI 助手（受 OpenClaw 启发）

**核心特点**:
- **代码量**: ~4,000 行核心代码（比 Clawdbot 小 99%）
- **语言**: Python (主) + TypeScript + JavaScript
- **架构**: 双层层架构（Steering Loop + AgentMessage）
- **运行环境**: 服务器/桌面/云端

**设计哲学**:
- 研究友好：代码清晰易读，易于理解和修改
- 快速启动：最小化占用，快速迭代
- 易于部署：一键部署

### zclaw

**定位**: ESP32 嵌入式 AI 助手

**核心特点**:
- **固件大小**: ≤ 888 KiB（全部固件，不仅是应用代码）
- **应用代码**: ~35 KiB (zclaw 逻辑)
- **语言**: C (主) + Shell + Python + JavaScript
- **架构**: 嵌入式固件 + FreeRTOS
- **运行环境**: ESP32 微控制器

**设计哲学**:
- 极致轻量：在资源受限设备上运行
- 有趣易用：易于使用和黑客改造
- 完整功能：支持 GPIO、定时任务、持久内存

---

## 📦 技术栈对比

### nanobot 技术栈

| 组件 | 技术 |
|------|------|
| **核心语言** | Python 3.11+ |
| **前端/卡片** | TypeScript, JavaScript |
| **容器化** | Docker |
| **支持渠道** | Feishu, Telegram, Discord, Slack, WeChat, QQ, WhatsApp, Email, Matrix, Nostr, DingTalk |
| **LLM 提供商** | OpenAI, Anthropic, Gemini, MiniMax, Moonshot, Kimi, Qwen, DeepSeek, VolcEngine, Ollama, vLLM, AWS Bedrock |
| **协议支持** | MCP (Model Context Protocol) |
| **技能系统** | ClawHub 技能注册表 |

### zclaw 技术栈

| 组件 | 技术 |
|------|------|
| **核心语言** | C (ESP-IDF 框架) |
| **运行时** | FreeRTOS |
| **网络** | Wi-Fi (lwIP) |
| **加密** | TLS/mbedTLS |
| **支持渠道** | Telegram, Web Relay |
| **LLM 提供商** | Anthropic, OpenAI, OpenRouter, Ollama (自定义端点) |
| **硬件接口** | GPIO 控制，I2C, SPI |
| **存储** | NVS (Non-Volatile Storage) |

---

## 🎯 功能特性对比

### nanobot 功能

| 类别 | 功能 |
|------|------|
| **核心功能** | 实时对话、任务调度、子代理、技能系统 |
| **记忆系统** | 双层记忆（MEMORY.md + HISTORY.md）、grep 搜索 |
| **工具调用** | 文件操作、Web 搜索、代码执行、定时任务 |
| **渠道支持** | 12+ 消息平台（Feishu、Telegram、Discord 等） |
| **模型支持** | 20+ LLM 提供商 |
| **高级功能** | MCP 协议、流式输出、交互式卡片、多模态 |
| **部署方式** | 源码安装、Docker、pip |

### zclaw 功能

| 类别 | 功能 |
|------|------|
| **核心功能** | 定时任务、GPIO 控制、自定义工具、持久记忆 |
| **记忆系统** | NVS 持久化存储（跨重启） |
| **工具调用** | 内置工具 + 用户自定义工具（C Handler） |
| **渠道支持** | Telegram、Web Relay（移动端 UI） |
| **模型支持** | 4 家 LLM 提供商 |
| **高级功能** | 时区感知调度、运行时诊断、人格选项 |
| **部署方式** | 一键引导脚本、串口烧录 |

---

## 📈 代码规模对比

### nanobot 代码分布

| 语言 | 大小 | 占比 |
|------|------|------|
| Python | 542 KB | ~93% |
| TypeScript | 8.8 KB | ~1.5% |
| Shell | 7 KB | ~1.2% |
| JavaScript | 1.3 KB | ~0.2% |
| Dockerfile | 1.3 KB | ~0.2% |
| **总计** | ~580 KB | 100% |

### zclaw 代码分布

| 语言 | 大小 | 占比 |
|------|------|------|
| C | 389 KB | ~42% |
| Shell | 199 KB | ~22% |
| Python | 178 KB | ~19% |
| HTML | 83 KB | ~9% |
| JavaScript | 13 KB | ~1.4% |
| CSS | 15 KB | ~1.6% |
| CMake | 1.4 KB | ~0.2% |
| **总计** | ~927 KB | 100% |

**固件大小细分** (ESP32-S3):

| 组件 | 大小 | 占比 |
|------|------|------|
| zclaw 应用逻辑 | ~35 KiB | ~4.1% |
| Wi-Fi + 网络栈 | ~388 KiB | ~45.7% |
| TLS/加密栈 | ~110 KiB | ~13.0% |
| 证书包 + 元数据 | ~97 KiB | ~11.5% |
| ESP-IDF/运行时/驱动 | ~219 KiB | ~25.8% |
| **总计** | ~849 KiB | 100% |

---

## 🚀 开发活跃度对比

| 指标 | nanobot | zclaw |
|------|---------|-------|
| **创建至今** | ~32 天 | ~16 天 |
| **总 Issues** | 357 | 3 |
| **总 PRs** | 468 | 3 |
| **最新 Release** | v0.1.4.post3 (2026-02-28) | v2.10.1 (2026-03-03) |
| **Release 频率** | 高（几乎每日更新） | 中（稳定迭代） |
| **社区参与** | 高（28K+ stars, 4.5K+ forks） | 中（1.7K+ stars, 132 forks） |

---

## 🎓 适用场景

### nanobot 适合

- ✅ **服务器/云端部署**: 需要 24/7 运行的 AI 助手
- ✅ **多平台集成**: 需要在 Feishu、Telegram、Discord 等多个平台使用
- ✅ **研究与开发**: 需要阅读、修改和扩展代码
- ✅ **企业应用**: 需要与现有系统集成（飞书、钉钉等）
- ✅ **复杂任务**: 需要子代理、技能系统、MCP 协议等高级功能
- ✅ **快速迭代**: 需要频繁更新和尝试新功能

### zclaw 适合

- ✅ **嵌入式设备**: 需要在 ESP32 等微控制器上运行
- ✅ **离线/边缘计算**: 需要在本地设备处理，减少云端依赖
- ✅ **硬件控制**: 需要 GPIO、传感器等硬件交互
- ✅ **低功耗场景**: 需要电池供电或低功耗运行
- ✅ **学习嵌入式 AI**: 想了解如何在资源受限设备上实现 AI
- ✅ **极客项目**: 想要黑客改造和自定义固件

---

## 📋 总结对比表

| 维度 | nanobot | zclaw |
|------|---------|-------|
| **定位** | 服务器/桌面 AI 助手 | 嵌入式 AI 助手 |
| **目标设备** | x86_64 服务器/PC | ESP32 微控制器 |
| **代码量** | ~4,000 行核心代码 | ~35 KiB 应用逻辑 |
| **固件/包大小** | ~580 KB | ≤ 888 KiB（完整固件） |
| **内存占用** | ~75-77% (0.87GB) | 嵌入式 RAM (KB 级) |
| **主要语言** | Python | C |
| **渠道支持** | 12+ 平台 | Telegram + Web |
| **LLM 支持** | 20+ 提供商 | 4 提供商 |
| **硬件接口** | 无（纯软件） | GPIO、I2C、SPI |
| **学习曲线** | 中（Python 基础） | 高（嵌入式开发） |
| **社区规模** | 28K+ stars | 1.7K+ stars |
| **更新频率** | 非常高（日更） | 稳定（周更） |
| **文档** | GitHub README | 独立文档站点 (zclaw.dev) |

---

## 💡 选择建议

**选择 nanobot，如果你需要**:
- 在服务器或云端运行 AI 助手
- 多平台消息集成（尤其是飞书、钉钉等企业平台）
- 快速开发和迭代
- 丰富的 LLM 提供商选择
- 研究和学习 AI 助手架构

**选择 zclaw，如果你需要**:
- 在嵌入式设备上运行 AI 助手
- 硬件控制和传感器交互
- 极低功耗和离线运行
- 学习嵌入式 AI 开发
- 极客和 DIY 项目

---

## 🔗 相关链接

- **nanobot**: https://github.com/HKUDS/nanobot
- **zclaw**: https://github.com/tnm/zclaw
- **zclaw 文档**: https://zclaw.dev
- **nanobot PyPI**: https://pypi.org/project/nanobot-ai/
