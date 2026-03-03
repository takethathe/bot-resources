# GitHub 项目调研报告

**调研时间**: 2026-03-03 03:45:55

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

### RUST

```
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

[dependencies]
microclaw-core = { path = "crates/microclaw-core" }
microclaw-clawhub 
```

## 🔄 最近提交

| Commit | 信息 | 日期 |
|--------|------|------|
| `4514fa7` | update readme | 2026-03-02 |

## 📖 README 摘要

# MicroClaw
<img src="icon.png" alt="MicroClaw logo" width="56" align="right" />

[English](README.md) | [中文](README_CN.md)

[![Website](https://img.shields.io/badge/Website-microclaw.ai-blue)](https://microclaw.ai)
[![Discord](https://img.shields.io/badge/Discord-Join-5865F2?logo=discord&logoColor=white)](https://discord.gg/pvmezwkAk5)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)


<p align="center">
  <img src="screenshots/headline.png" alt="MicroClaw headline logo" width="92%" />
</p>


> **Note:** This project is under active development. Features may change, and contributions are welcome!


An agentic AI assistant for chat surfaces, inspired by [nanoclaw](https://github.com/gavrielc/nanoclaw/) and incorporating some of its design ideas. MicroClaw uses a channel-agnostic core with platform adapters: it currently supports Telegram, Discord, Slack, Feishu/Lark, Matrix, WhatsApp, iMessage, Email, Nostr, Signal, DingTalk, QQ, IRC, and Web, and is desig


### 其他章节:
- Table of contents

## 💡 总结

### 项目评价

- ⭐⭐⭐ 有一定知名度的项目

### 适用场景

- 待分析...


### 注意事项

- 请查看项目许可证了解使用限制
- 建议阅读 CONTRIBUTING.md 了解贡献指南

---

*报告生成时间：2026-03-03 03:45:55*