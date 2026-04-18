# Dream 功能调查报告

# Dream Feature Investigation Research

**Generated**: 2026-04-18 03:07:29 UTC  
**Type**: research  
**Category**: research  
**Topic**: dream-feature-investigation

---


**调查日期**: 2026-04-18  
**调查对象**: nanobot Dream 记忆整合系统  
**报告作者**: nanobot 🐈

---

## 一、核心结论

### Dream 是什么？

**Dream 是 nanobot 的两阶段记忆处理器，负责自动分析和整合历史对话，提取重要信息并更新记忆文件。**

> "Two-phase memory processor: analyze history.jsonl, then edit files via AgentRunner."

---

## 二、Dream 的主要功能

### 2.1 核心任务

| 任务 | 说明 | 目标文件 |
|------|------|---------|
| **提取新事实** | 从对话历史中提取原子化事实 | USER.md / SOUL.md / MEMORY.md |
| **去重优化** | 扫描记忆文件，标记冗余/过时内容 | 所有记忆文件 |
| **技能发现** | 识别重复出现的工作流，创建新技能 | skills/<name>/SKILL.md |
| **记忆整合** | 压缩历史对话，保持记忆精简 | history.jsonl |

---

### 2.2 两阶段处理流程

#### Phase 1: 分析 (Analysis)

```
输入：历史对话 + 当前记忆文件
      ↓
LLM 分析 (无工具调用)
      ↓
输出：分析摘要
  - [FILE] 新事实
  - [FILE-REMOVE] 待删除内容
  - [SKILL] 可复用模式
```

**Phase 1 提示词要点**:
- 提取原子化事实（如 "has a cat named Luna"）
- 扫描所有记忆文件的冗余模式
- 识别重复出现 2+ 次的工作流
- 标记过时内容（基于行龄注释）

#### Phase 2: 执行 (Execution)

```
输入：Phase 1 分析结果 + 现有技能列表
      ↓
AgentRunner (带工具调用)
      ↓
执行操作：
  - edit_file: 更新记忆文件
  - write_file: 创建新技能
      ↓
输出：变更日志 + Git 提交
```

**Phase 2 可用工具**:
- `read_file` - 读取文件
- `edit_file` - 编辑文件
- `write_file` - 创建技能文件

---

## 三、记忆文件结构

### 3.1 文件类型

| 文件 | 用途 | 更新策略 |
|------|------|---------|
| **USER.md** | 用户身份信息、偏好设置 | 仅修正错误，永久保留 |
| **SOUL.md** | 机器人行为、语气、风格 | 仅修正错误，永久保留 |
| **MEMORY.md** | 项目上下文、知识、事件 | 动态更新，可删除过时内容 |
| **history.jsonl** | 对话历史日志 | 定期压缩 |

### 3.2 行龄标注系统

MEMORY.md 支持行龄标注（基于 Git blame）：

```markdown
- 用户喜欢简洁的回答  ← 45d
- 上海时区 UTC+8  ← 120d
- 项目 nanobot 39,525 ⭐  ← 3d
```

**标注规则**:
- 超过 `_STALE_THRESHOLD_DAYS` (默认 30 天) 的行添加 `← Nd` 后缀
- 行龄仅表示最后修改时间，不代表应该删除
- SOUL.md 和 USER.md 永不标注行龄

---

## 四、调度与配置

### 4.1 默认调度

```python
# config/schema.py
class DreamConfig(Base):
    interval_h: int = Field(default=2, ge=1)  # 每 2 小时
    model_override: str | None = None  # 可选的模型覆盖
    max_batch_size: int = 20  # 每次处理最大历史条目数
    max_iterations: int = 10  # Phase 2 最大迭代次数
```

### 4.2 Cron 任务

```python
# cli/commands.py
cron.register_system_job(CronJob(
    id="dream",
    name="dream",
    schedule=dream_cfg.build_schedule(config.agents.defaults.timezone),
    payload=CronPayload(
        kind="system_event",
        event_type="dream_consolidation",
    ),
))
```

**当前配置**: 每 2 小时运行一次（系统任务，不可删除）

---

## 五、代码实现

### 5.1 核心类结构

```python
class Dream:
    """两阶段记忆处理器"""
    
    def __init__(self, store, provider, model, ...):
        self.store = store  # MemoryStore 文件 I/O
        self.provider = provider  # LLM 提供商
        self.model = model  # 模型名称
        self._runner = AgentRunner(provider)  # Phase 2 执行器
        self._tools = self._build_tools()  # 工具注册表
    
    async def run(self) -> bool:
        """处理未处理的历史条目，返回是否执行了工作"""
        # 1. 读取未处理历史
        # 2. Phase 1: 分析
        # 3. Phase 2: 执行
        # 4. 更新游标 + 压缩历史
        # 5. Git 自动提交
```

### 5.2 关键方法

| 方法 | 功能 |
|------|------|
| `run()` | 主入口，处理历史条目 |
| `_annotate_with_ages()` | 为 MEMORY.md 添加行龄标注 |
| `_list_existing_skills()` | 列出现有技能（用于去重） |
| `_build_tools()` | 构建 Phase 2 工具注册表 |
| `get_last_dream_cursor()` | 获取上次处理游标 |
| `set_last_dream_cursor()` | 设置处理游标 |

---

## 六、用户命令

### 6.1 内置命令

| 命令 | 功能 |
|------|------|
| `/dream` | 手动触发 Dream 整合运行 |
| `/dream-log` | 查看上次 Dream 变更内容 |
| `/dream-log <sha>` | 查看指定提交的变更 |
| `/dream-restore` | 列出可恢复的历史版本 |
| `/dream-restore <sha>` | 恢复到指定版本 |

### 6.2 命令实现

```python
# command/builtin.py
async def cmd_dream(ctx: CommandContext) -> OutboundMessage:
    """手动触发 Dream 整合运行"""
    async def _run_dream():
        t0 = time.monotonic()
        did_work = await loop.dream.run()
        elapsed = time.monotonic() - t0
        if did_work:
            content = f"Dream completed in {elapsed:.1f}s."
        else:
            content = "Dream: nothing to process."
```

---

## 七、Git 集成

### 7.1 自动提交

```python
# memory.py:829-836
if changelog and self.store.git.is_initialized():
    ts = batch[-1]["timestamp"]
    summary = f"dream: {ts}, {len(changelog)} change(s)"
    commit_msg = f"{summary}\n\n{analysis.strip()}"
    sha = self.store.git.auto_commit(commit_msg)
    if sha:
        logger.info("Dream commit: {}", sha)
```

**提交格式**:
```
dream: 2026-04-18 03:00, 3 change(s)

[分析摘要内容...]
```

### 7.2 版本恢复

Dream 的每次变更都会提交到 Git，支持：
- 查看历史变更 (`/dream-log`)
- 恢复到任意版本 (`/dream-restore <sha>`)
- 防止错误更新导致数据丢失

---

## 八、实际运行示例

### 8.1 当前 Cron 状态

```
- dream (id: dream, every 2h)
  Purpose: Dream memory consolidation for long-term memory.
  Protected: visible for inspection, but cannot be removed.
  Next run: 2026-04-18T04:38:17.881000+00:00 (UTC)
```

### 8.2 典型处理流程

```
1. 用户对话 → history.jsonl 追加记录
2. Dream 触发（每 2 小时）
3. 读取未处理历史（最多 20 条）
4. Phase 1: LLM 分析，生成变更列表
5. Phase 2: AgentRunner 执行编辑
6. 更新游标，压缩历史
7. Git 自动提交变更
8. 等待下次触发
```

---

## 九、设计亮点

### 9.1 两阶段架构优势

| 优势 | 说明 |
|------|------|
| **分离关注点** | Phase 1 专注分析，Phase 2 专注执行 |
| **工具安全** | Phase 1 无工具调用，避免误操作 |
| **迭代优化** | Phase 2 可多次迭代，逐步完善更新 |
| **可调试性** | 两阶段日志分离，便于问题定位 |

### 9.2 去重机制

- Phase 1 扫描所有记忆文件
- 识别重复/重叠/过时内容
- Phase 2 执行前检查现有技能
- 避免创建功能冗余的技能

### 9.3 安全保护

- SOUL.md/USER.md 永久保留，仅修正错误
- MEMORY.md 行龄仅作为参考，不自动删除
- Git 版本控制，支持恢复
- 游标机制，避免重复处理

---

## 十、配置选项

### 10.1 config.yaml 配置

```yaml
agents:
  defaults:
    dream:
      interval_h: 2          # 运行间隔（小时）
      model_override: null   # 可选的模型覆盖
      max_batch_size: 20     # 最大批处理大小
      max_iterations: 10     # Phase 2 最大迭代
      annotate_line_ages: true  # 行龄标注开关
```

### 10.2 模型覆盖

Dream 可使用独立于主 Agent 的模型：

```yaml
agents:
  defaults:
    dream:
      model_override: "gpt-4o-mini"  # 使用更便宜的模型
```

**优势**:
- 降低运行成本
- 提高处理速度
- 不影响主对话质量

---

## 十一、总结

### Dream 的核心价值

> **自动化记忆管理，让 nanobot 越用越"聪明"**

| 功能 | 用户价值 |
|------|---------|
| 自动提取事实 | 无需手动记录偏好和上下文 |
| 智能去重 | 保持记忆精简，避免冗余 |
| 技能发现 | 自动沉淀重复使用的工作流 |
| 版本控制 | 安全可恢复，防止数据丢失 |
| 低成本运行 | 可选独立模型，优化成本 |

### 适用场景

- ✅ 长期使用的个人助手
- ✅ 需要记忆用户偏好的场景
- ✅ 多轮对话上下文管理
- ✅ 技能库自动沉淀

### 不适用场景

- ❌ 一次性对话（无历史价值）
- ❌ 敏感信息处理（需人工审核）
- ❌ 实时性要求高的场景（2 小时间隔）

---

**报告完成** 🐈