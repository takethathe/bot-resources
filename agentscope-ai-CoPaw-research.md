# GitHub 项目调研报告

**调研时间**: 2026-03-02 06:41:52

**项目**: [agentscope-ai/CoPaw](https://github.com/agentscope-ai/CoPaw)


## 📌 项目概览

- **名称**: CoPaw
- **描述**: Your Personal AI Assistant; easy to install, deploy on your own machine or on the cloud; supports multiple chat apps with easily extensible capabilities.
- **Stars**: 4042 ⭐
- **Forks**: 402 🔱
- **许可证**: N/A
- **主要语言**: Python
- **创建时间**: 2026-02-24
- **更新时间**: 2026-03-02

## 📁 文件结构

```

📁 .github/
  📁 ISSUE_TEMPLATE/
    📄 1-bug_report.md (1.1KB)
    📄 2-feature_request.md
    📄 3-documentation.md
    📄 4-question.md
    📄 5-support_environment.md
    📄 config.yml
  📁 workflows/
    📄 deploy-website.yml (1.8KB)
    📄 docker-release.yml (1.7KB)
    📄 npm-format.yml (1.1KB)
    📄 pre-commit.yml (1.0KB)
    📄 publish-pypi.yml (1.3KB)
  📄 PULL_REQUEST_TEMPLATE.md
📁 console/
  📁 public/
    📄 copaw-symbol.svg (2.7KB)
    📄 logo.png (17.5KB)
  📁 src/
    📁 api/
      📁 modules/
      📁 types/
      📄 config.ts
      📄 index.ts (1.1KB)
      📄 request.ts (1.1KB)
    📁 components/
      📁 ConsoleCronBubble/
      📁 MarkdownCopy/
      📄 LanguageSwitcher.tsx
    📁 layouts/
      📁 MainLayout/
      📄 Header.tsx (1.1KB)
      📄 Sidebar.tsx (4.1KB)
    📁 locales/
      📄 en.json (15.7KB)
      📄 zh.json (15.2KB)
    📁 pages/
      📁 Agent/
      📁 Chat/
      📁 Control/
      📁 Settings/
    📁 styles/
      📄 form-override.css
      📄 layout.css (1.2KB)
    📁 utils/
      📄 markdown.ts
    📄 App.tsx
    📄 i18n.ts
    📄 main.tsx
```


## 📊 代码统计

| 语言 | 文件数 | 总行数 | 空行 | 注释行 |
|------|--------|--------|------|--------|
| PY | 190 | 40,144 | 5978 | 1032 |
| TS | 61 | 2,993 | 324 | 152 |
| JS | 1 | 28 | 1 | 0 |
| **总计** | **252** | **43,165** | - | - |

## 📦 依赖分析

未检测到常见依赖配置文件

## 🔄 最近提交

| Commit | 信息 | 日期 |
|--------|------|------|
| `f842a13` | fix(mcp): support streamable http/sse cl | 2026-03-02 |

## 📖 README 摘要

<div align="center">

# CoPaw

[![GitHub Repo](https://img.shields.io/badge/GitHub-Repo-black.svg?logo=github)](https://github.com/agentscope-ai/CoPaw)
[![PyPI](https://img.shields.io/pypi/v/copaw?color=3775A9&label=PyPI&logo=pypi)](https://pypi.org/project/copaw/)
[![Documentation](https://img.shields.io/badge/Docs-Website-green.svg?logo=readthedocs&label=Docs)](https://copaw.agentscope.io/)
[![Python Version](https://img.shields.io/badge/python-3.10%20~%20%3C3.14-blue.svg?logo=python&label=Python)](https://www.python.org/downloads/)
[![Last Commit](https://img.shields.io/github/last-commit/agentscope-ai/CoPaw)](https://github.com/agentscope-ai/CoPaw)
[![License](https://img.shields.io/badge/license-Apache%202.0-red.svg?logo=apache&label=License)](LICENSE)
[![Code Style](https://img.shields.io/badge/code%20style-black-black.svg?logo=python&label=CodeStyle)](https://github.com/psf/black)
[![GitHub Stars](https://img.shields.io/github/stars/agentscope-ai/CoPaw?style=flat&logo=github&col

## 💡 总结

### 项目评价

- ⭐⭐⭐⭐ 热门项目，值得关注
- **代码规模**: 4.3 万行代码，252 个文件
- **主要语言**: Python (93%) + TypeScript (7%)
- **版本**: PyPI 发布中
- **许可证**: Apache 2.0 License

### 适用场景

- **个人 AI 助手** - 轻量级个人 AI 助手，快速部署
- **本地模型运行** - 原生支持 llama.cpp 和 MLX，隐私优先
- **中文生态** - 钉钉、飞书、QQ 等中文平台集成
- **Python 扩展** - Python 生态，易于定制和开发
- **Web 控制台** - Console UI 友好的 Web 界面
- **定时任务** - Heartbeat 定时任务和提醒系统
- **快速上手** - 一键安装，配置简单
- **云端部署** - 支持 ModelScope 云端部署和阿里云 ECS

### 核心优势

1. **简单易用** - 一键安装脚本，自动处理 Python 环境
2. **本地模型** - 原生支持 llama.cpp 和 MLX，无需云 API
3. **Python 生态** - 基于 Python，易于理解和扩展
4. **中文友好** - 钉钉/飞书/QQ 集成，中文文档
5. **Console UI** - 友好的 Web 控制台，支持中英文
6. **轻量级** - 代码量小，资源占用低
7. **AgentScope 集成** - 与 AgentScope 生态系统兼容
8. **Apache 2.0** - 宽松的开源许可证

### 注意事项

- 需要 Python 3.10 ~ 3.14
- 通道数量相对有限 (~10 个)
- 无原生移动应用 (iOS/Android)
- 社区相对较小 (4k stars)
- 企业级功能较弱 (缺少沙箱、高级权限管理)

---

*报告生成时间：2026-03-02 06:41:52*  
*补全时间：2026-03-02 06:57*