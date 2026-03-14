# AgentScope 项目进展报告

# Agentscope Progress Research

**Generated**: 2026-03-14 01:53:02 UTC  
**Type**: research  
**Category**: research  
**Topic**: agentscope-progress

---


**调研时间**: 2026-03-14 01:52 (UTC)  
**时间范围**: 2026-03-12 至 2026-03-14  
**项目地址**: https://github.com/agentscope-ai/agentscope

---

## 📊 概览

| 指标 | 数值 |
|------|------|
| **Stars** | 18,084 |
| **Forks** | 1,606 |
| **最新版本** | v1.0.17 |
| **发布时间** | 2026-03-13 08:27 |
| **合并 PR** | 10+ (近 3 天) |
| **开放 Issues** | 127 |

---

## 🎯 核心进展

### 1️⃣ v1.0.17 版本发布

**发布时间**: 2026-03-13 08:27 UTC

**版本说明**:
- 17 个 PR 合并
- 7 位新贡献者首次贡献
- 主要聚焦于 bug 修复和稳定性改进

---

### 2️⃣ 最近合并的 PR (2026-03-12 至 2026-03-14)

| PR # | 标题 | 作者 | 合并时间 |
|------|------|------|----------|
| **#1325** | chore(version): update version to 1.0.17 for release | DavdGao | 03-13 08:26 |
| **#1299** | fix(examples): await register_mcp_client in deep_research_agent | YingchaoX | 03-13 08:13 |
| **#1251** | fix(gemini): handle None usage in streaming response parsing | shlokmestry | 03-12 03:16 |
| **#1313** | feat(session): make jsonsession file read and write async | qbc2016 | 03-11 08:46 |
| **#1279** | fix(plan): trigger hooks when recovering historical plans | QianCyrus | 03-11 09:31 |
| **#1284** | fix(formatter): correct local path identification logic | DavdGao | 03-11 01:43 |
| **#1283** | chore(github-actions): upgrade GitHub actions for node 24 | salmanmkc | 03-11 01:44 |
| **#1304** | feat(dashscope): move generate_kwargs after defaults | qbc2016 | 03-11 01:42 |
| **#1298** | fix(hooks): prevent Agent crash when Studio disconnects | YingchaoX | 03-11 01:45 |

---

### 3️⃣ 重点功能详解

#### 📁 JSONSession 异步读写 (#1313)

**作者**: qbc2016  
**合并时间**: 2026-03-11 08:46

**改进内容**:
- ✅ JSONSession 文件读写改为异步操作
- ✅ 避免阻塞主线程
- ✅ 提升多 Agent 并发性能

**影响**: 多 Agent 场景下会话保存/加载更流畅

---

#### 🔧 DashScope 参数覆盖支持 (#1304)

**作者**: qbc2016  
**合并时间**: 2026-03-11 01:42

**改进内容**:
- ✅ `generate_kwargs` 移至 `defaults` 之后
- ✅ 支持运行时参数覆盖默认配置
- ✅ 更灵活的模型调用配置

**示例**:
```python
model = DashScopeChatModel(
    model_name="qwen-max",
    temperature=0.7,  # 覆盖默认值
)
```

---

#### 🪝 Plan Hooks 触发修复 (#1279)

**作者**: QianCyrus  
**合并时间**: 2026-03-11 09:31

**修复内容**:
- ✅ 恢复历史计划时正确触发 hooks
- ✅ 确保 UI/日志同步更新
- ✅ 避免状态不一致

---

#### 🎯 Deep Research Agent 修复 (#1299)

**作者**: YingchaoX  
**合并时间**: 2026-03-13 08:13

**修复内容**:
- ✅ `register_mcp_client` 添加 await
- ✅ 修复多轮深度研究 bug
- ✅ MCP 客户端注册更稳定

---

#### 💎 Gemini 流式响应优化 (#1251)

**作者**: shlokmestry  
**合并时间**: 2026-03-12 03:16

**修复内容**:
- ✅ 处理流式响应中的 `None usage`
- ✅ 避免解析错误
- ✅ 提升 Gemini 模型兼容性

---

#### 🪝 Studio 断开保护 (#1298)

**作者**: YingchaoX  
**合并时间**: 2026-03-11 01:45

**修复内容**:
- ✅ 防止 Studio 断开时 Agent 崩溃
- ✅ 增强 hooks 容错能力
- ✅ 提升开发调试体验

---

### 4️⃣ 新增开放 Issues (近 3 天)

| Issue # | 标题 | 类型 | 创建时间 |
|---------|------|------|----------|
| **#1324** | [Feature]: ChatUsage 支持 cached_token | Feature | 03-13 06:38 |
| **#1323** | [Feature]: Add string replacement tool for file editing | Feature | 03-13 06:32 |
| **#1322** | [Feature]: agent 失败后，从失败的步骤重新运行的机制有吗？ | Feature | 03-13 02:04 |
| **#1314** | [Feature]: Support OAuth dynamic auth for streamableHTTP MCP | Feature | 03-11 08:29 |
| **#1312** | 环境报错，vllm，flash_attn 都报错 | Bug | 03-11 06:56 |

---

### 5️⃣ 新贡献者

v1.0.17 版本迎来 **7 位新贡献者**:

| 贡献者 | PR | 贡献内容 |
|--------|-----|----------|
| **@YingchaoX** | #1262 | 文本文件工具 `~` 路径展开 |
| **@04cb** | #1273 | 文档拼写修正 |
| **@WeiSibo** | #1271 | vLLM reasoning 字段解析 |
| **@salmanmkc** | #1283 | GitHub Actions node 24 升级 |
| **@dipeshbabu** | #1247 | 文档字符串修正 |
| **@QianCyrus** | #1279 | Plan Hooks 触发修复 |
| **@shlokmestry** | #1251 | Gemini 流式响应优化 |

---

## 📈 技术趋势

| 方向 | 进展 |
|------|------|
| **异步化** | JSONSession 异步读写 |
| **稳定性** | Studio 断开保护、Hooks 容错 |
| **MCP 集成** | Deep Research Agent MCP 修复 |
| **模型兼容** | Gemini/DashScope/vLLM 优化 |
| **开发者体验** | 参数覆盖、Hooks 触发 |

---

## 🔗 相关链接

- **仓库**: https://github.com/agentscope-ai/agentscope
- **Release v1.0.17**: https://github.com/agentscope-ai/agentscope/releases/tag/v1.0.17
- **完整 Changelog**: https://github.com/agentscope-ai/agentscope/compare/v1.0.16...v1.0.17

---

## ✅ 总结

最近两天 AgentScope 主要聚焦于**稳定性修复**和**性能优化**:

1. **📦 v1.0.17 发布**: 17 个 PR 合并，7 位新贡献者
2. **⚡ 异步化**: JSONSession 异步读写提升并发性能
3. **🛠️ 稳定性**: Studio 断开保护、Hooks 触发修复
4. **🔌 MCP 集成**: Deep Research Agent MCP 客户端修复
5. **🎯 模型兼容**: Gemini/DashScope/vLLM 优化

整体进展稳健，社区活跃度良好。

---

**报告生成**: 2026-03-14 01:52 (UTC)  
**数据来源**: GitHub CLI (gh)