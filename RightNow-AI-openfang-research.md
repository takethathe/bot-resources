# GitHub 项目调研报告

**调研时间**: 2026-03-03 09:49:28

**项目**: [RightNow-AI/openfang](https://github.com/RightNow-AI/openfang)


## 📌 项目概览

- **名称**: openfang
- **描述**: Open-source Agent Operating System
- **Stars**: 9420 ⭐
- **Forks**: 991 🔱
- **许可证**: N/A
- **主要语言**: Rust
- **创建时间**: 2026-02-24
- **更新时间**: 2026-03-03

## 📁 文件结构

```

📁 .github/
  📁 workflows/
    📄 ci.yml (4.0KB)
    📄 release.yml (8.5KB)
  📄 FUNDING.yml
📁 agents/
  📁 analyst/
    📄 agent.toml (1.6KB)
  📁 architect/
    📄 agent.toml (1.4KB)
  📁 assistant/
    📄 agent.toml (6.1KB)
  📁 code-reviewer/
    📄 agent.toml (1.6KB)
  📁 coder/
    📄 agent.toml (1.9KB)
  📁 customer-support/
    📄 agent.toml (4.7KB)
  📁 data-scientist/
    📄 agent.toml (1.5KB)
  📁 debugger/
    📄 agent.toml (2.0KB)
  📁 devops-lead/
    📄 agent.toml (1.6KB)
  📁 doc-writer/
    📄 agent.toml (1.4KB)
  📁 email-assistant/
    📄 agent.toml (4.0KB)
  📁 health-tracker/
    📄 agent.toml (5.5KB)
  📁 hello-world/
    📄 agent.toml (1.1KB)
  📁 home-automation/
    📄 agent.toml (5.7KB)
  📁 legal-assistant/
    📄 agent.toml (5.6KB)
  📁 meeting-assistant/
    📄 agent.toml (4.5KB)
  📁 ops/
    📄 agent.toml (1.4KB)
  📁 orchestrator/
    📄 agent.toml (2.1KB)
  📁 personal-finance/
    📄 agent.toml (4.4KB)
  📁 planner/
    📄 agent.toml (1.5KB)
  📁 recruiter/
    📄 agent.toml (5.5KB)
  📁 researcher/
    📄 agent.toml (1.8KB)
```


## 📊 代码统计

| 语言 | 文件数 | 总行数 | 空行 | 注释行 |
|------|--------|--------|------|--------|
| RS | 239 | 149,147 | 14086 | 16833 |
| JS | 25 | 8,832 | 731 | 425 |
| PY | 6 | 616 | 174 | 28 |
| TS | 1 | 140 | 17 | 0 |
| **总计** | **271** | **158,735** | - | - |

## 📦 依赖分析

### RUST

```
[workspace]
resolver = "2"
members = [
    "crates/openfang-types",
    "crates/openfang-memory",
    "crates/openfang-runtime",
    "crates/openfang-wire",
    "crates/openfang-api",
    "crates/openfang-kernel",
    "crates/openfang-cli",
    "crates/openfang-channels",
    "crates/openfang-migrate",
    "crates/openfang-skills",
    "crates/openfang-desktop",
    "crates/openfang-hands",
    "crates/openfang-extensions",
    "xtask",
]

[workspace.package]
version = "0.3.4"
edition = "2021"
l
```

## 🔄 最近提交

| Commit | 信息 | 日期 |
|--------|------|------|
| `8942d8c` | bugfixes release | 2026-03-03 |

## 📖 README 摘要

<p align="center">
  <img src="public/assets/openfang-logo.png" width="160" alt="OpenFang Logo" />
</p>

<h1 align="center">OpenFang</h1>
<h3 align="center">The Agent Operating System</h3>

<p align="center">
  Open-source Agent OS built in Rust. 137K LOC. 14 crates. 1,767+ tests. Zero clippy warnings.<br/>
  <strong>One binary. Battle-tested. Agents that actually work for you.</strong>
</p>

<p align="center">
  <a href="https://openfang.sh/docs">Documentation</a> &bull;
  <a href="https://openfang.sh/docs/getting-started">Quick Start</a> &bull;
  <a href="https://x.com/openfangg">Twitter / X</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/language-Rust-orange?style=flat-square" alt="Rust" />
  <img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="MIT" />
  <img src="https://img.shields.io/badge/version-0.1.0-green?style=flat-square" alt="v0.1.0" />
  <img src="https://img.shields.io/badge/tests-1,767%2B%20passing-brightgreen?style=flat-


### 其他章节:
- What is OpenFang?

## 💡 总结

### 项目评价

- ⭐⭐⭐⭐ 热门项目，值得关注

### 适用场景

- 待分析...


### 注意事项

- 请查看项目许可证了解使用限制
- 建议阅读 CONTRIBUTING.md 了解贡献指南

---

*报告生成时间：2026-03-03 09:49:28*