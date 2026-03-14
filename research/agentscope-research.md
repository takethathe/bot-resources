# AgentScope 项目调研报告

# Agentscope Research

**Generated**: 2026-03-14 01:41:56 UTC  
**Type**: research  
**Category**: research  
**Topic**: agentscope

---


**调研时间**: 2026-03-14  
**项目地址**: https://github.com/agentscope-ai/agentscope  
**版本**: v1.0.17 (2026-03-13)

---

## 📊 项目概况

| 指标 | 数值 |
|------|------|
| **Stars** | 18,084 |
| **Forks** | 1,606 |
| **Open Issues** | 127 |
| **许可证** | Apache-2.0 |
| **创建时间** | 2024-01-12 |
| **最后更新** | 2026-03-14 |
| **语言** | Python |
| **最低要求** | Python 3.10+ |

---

## 🎯 项目定位

**AgentScope** 是一个生产级、易用的 Agent 框架，具有以下特点：

- **简单**: 5 分钟开始构建 Agent，内置 ReAct Agent、工具、技能、人机交互、记忆、规划、实时语音、评估和模型微调
- **可扩展**: 大量生态系统集成（工具、记忆、可观测性），内置 MCP 和 A2A 支持，消息中心支持灵活的多 Agent 编排
- **生产就绪**: 本地部署、云端 Serverless、K8s 集群部署，内置 OpenTelemetry 支持

---

## 🏗️ 核心架构

### 1. Agent 系统

#### 内置 Agent 类型
| Agent | 说明 |
|-------|------|
| **ReActAgent** | 基于 ReAct 模式的推理 Agent |
| **UserAgent** | 用户交互 Agent |
| **RealtimeVoiceAgent** | 实时语音交互 Agent |
| **A2AAgent** | 支持 Agent-to-Agent 协议 |

#### 示例代码
```python
from agentscope.agent import ReActAgent, UserAgent
from agentscope.model import DashScopeChatModel
from agentscope.formatter import DashScopeChatFormatter
from agentscope.memory import InMemoryMemory
from agentscope.tool import Toolkit
import asyncio

async def main():
    toolkit = Toolkit()
    toolkit.register_tool_function(execute_python_code)
    toolkit.register_tool_function(execute_shell_command)

    agent = ReActAgent(
        name="Friday",
        sys_prompt="You're a helpful assistant named Friday.",
        model=DashScopeChatModel(model_name="qwen-max", api_key="...", stream=True),
        memory=InMemoryMemory(),
        formatter=DashScopeChatFormatter(),
        toolkit=toolkit,
    )

    user = UserAgent(name="user")
    msg = None
    while True:
        msg = await agent(msg)
        msg = await user(msg)
        if msg.get_text_content() == "exit":
            break

asyncio.run(main())
```

---

### 2. 记忆系统 (Memory)

#### 记忆类型
| 类型 | 说明 |
|------|------|
| **InMemoryMemory** | 内存记忆（默认） |
| **RedisMemory** | Redis 后端记忆 |
| **SQLiteMemory** | SQLite 后端记忆 |
| **LongTermMemory** | 长期记忆（支持 ReMe 集成） |
| **MemoryCompression** | 记忆压缩功能 |

#### 可插拔架构 (v1.0.17 新增)
- `BaseMemoryProvider` 抽象基类
- `MemoryProviderRegistry` 动态注册机制
- `create_memory_provider` 工厂函数

---

### 3. 工具系统 (Toolkit)

#### 内置工具
- `execute_python_code` - Python 代码执行
- `execute_shell_command` - Shell 命令执行
- MCP 工具集成（HTTP/Streamable 传输）
- Web 搜索、文件读写等

#### MCP 支持
```python
from agentscope.mcp import HttpStatelessClient
from agentscope.tool import Toolkit

async def fine_grained_mcp_control():
    client = HttpStatelessClient(
        name="gaode_mcp",
        transport="streamable_http",
        url="https://mcp.amap.com/mcp?key=...",
    )
    func = await client.get_callable_function(func_name="maps_geo")
    toolkit = Toolkit()
    toolkit.register_tool_function(func)
```

---

### 4. 多 Agent 编排

#### MsgHub 消息中心
```python
from agentscope.pipeline import MsgHub, sequential_pipeline
from agentscope.message import Msg
import asyncio

async def multi_agent_conversation():
    agent1, agent2, agent3, agent4 = ...
    
    async with MsgHub(
        participants=[agent1, agent2, agent3],
        announcement=Msg("Host", "Introduce yourselves.", "assistant")
    ) as hub:
        await sequential_pipeline([agent1, agent2, agent3])
        hub.add(agent4)
        hub.delete(agent3)
        await hub.broadcast(Msg("Host", "Goodbye!", "assistant"))

asyncio.run(multi_agent_conversation())
```

#### 功能特性
- 灵活的消息路由
- 动态参与者管理
- 顺序/并行执行管道
- 广播和点对点通信

---

### 5. 模型支持

#### 支持的模型提供商
| 提供商 | 模型 |
|--------|------|
| **DashScope** | Qwen-Max, Qwen-Plus 等 |
| **OpenAI** | GPT-4, GPT-3.5-turbo |
| **Anthropic** | Claude 系列 |
| **Google** | Gemini 系列 |
| **Ollama** | 本地模型 |
| **vLLM** | 自部署模型 |

#### 实时模型支持
- `OpenAIRealtimeModel` - OpenAI 实时语音
- `GeminiRealtimeModel` - Gemini 实时语音
- `DashScopeChatModel` - 通义千问多模态

---

### 6. 微调系统 (Tuner)

#### 支持场景
| 示例 | 描述 | 模型 | 效果提升 |
|------|------|------|----------|
| **Math Agent** | 多步推理数学求解 | Qwen3-0.6B | 75% → 85% |
| **Frozen Lake** | 导航任务 | Qwen2.5-3B-Instruct | 15% → 86% |
| **Learn to Ask** | LLM-as-a-judge 反馈 | Qwen2.5-7B-Instruct | 47% → 92% |
| **Email Search** | 工具使用能力 | Qwen3-4B-Instruct-2507 | 60% |
| **Werewolf Game** | 多 Agent 策略博弈 | Qwen2.5-7B-Instruct | 50% → 80% |
| **Data Augment** | 合成数据生成 | Qwen3-0.6B | AIME-24: 20% → 60% |

#### 集成
- Trinity-RFT 强化学习库
- LLM-as-a-judge 自动评估
- 数据增强和合成

---

## 📦 生态系统

### 相关仓库
| 仓库 | 说明 |
|------|------|
| **agentscope-samples** | 示例项目集合（Alias-Agent, Data-Juicer Agent 等） |
| **agentscope-runtime** | Docker/K8s 部署、VNC GUI 沙箱 |
| **Trinity-RFT** | 强化学习微调集成 |

### 近期功能时间线

| 时间 | 功能 |
|------|------|
| **2026-02** | 实时语音 Agent 支持 |
| **2026-01** | 双周会议启动、数据库支持、记忆压缩 |
| **2025-12** | A2A (Agent-to-Agent) 协议、TTS 支持 |
| **2025-11** | Anthropic Agent Skill、Agentic RL、ReMe 长期记忆 |

---

## 🆚 与 nanobot 对比

| 维度 | AgentScope | nanobot |
|------|------------|---------|
| **定位** | Agent 开发框架 | AI 助手运行时 |
| **核心功能** | Agent 构建、多 Agent 编排、微调 | 消息渠道、技能系统、记忆 |
| **部署** | 本地/云端/K8s | 本地/服务器/Docker |
| **模型支持** | 20+ 提供商 | 20+ 提供商 |
| **渠道支持** | 有限（主要 API） | 12+ 消息平台 |
| **多 Agent** | ✅ MsgHub 编排 | ⚠️ 基础支持 |
| **微调** | ✅ 内置 RL 微调 | ❌ 无 |
| **实时语音** | ✅ 支持 | ❌ 无 |
| **MCP** | ✅ 支持 | ✅ 支持 |
| **A2A** | ✅ 支持 | ❌ 无 |
| **Stars** | 18,084 | 28,000+ |
| **许可证** | Apache-2.0 | MIT |

---

## 📋 最新版本 (v1.0.17)

**发布时间**: 2026-03-13

### 主要更新
| 类型 | 内容 |
|------|------|
| **fix(tool)** | 文本文件工具支持 `~` 路径展开 |
| **fix(openai-model)** | 支持新版 vLLM reasoning 字段解析 |
| **fix(model)** | DashScopeChatModel 使用 AioMultiModalConversation |
| **fix(utils)** | 修复 base64 数据保存时的双点扩展名问题 |
| **feat(dashscope)** | generate_kwargs 移至 defaults 后支持参数覆盖 |
| **fix(formatter)** | 修正本地路径识别逻辑 |
| **fix(hooks)** | 防止 Studio 断开时 Agent 崩溃 |
| **feat(session)** | JSONSession 异步读写 |
| **fix(plan)** | 恢复历史计划时触发 hooks |
| **fix(deep_research_agent)** | 修复多轮深度研究 bug |
| **fix(gemini)** | 处理流式响应中的 None usage |

---

## 🔧 安装使用

### 安装
```bash
# pip 安装
pip install agentscope

# 或使用 uv
uv pip install agentscope

# 源码安装
git clone -b main https://github.com/agentscope-ai/agentscope.git
cd agentscope
pip install -e .
```

### 快速开始
```python
from agentscope.agent import ReActAgent
from agentscope.model import DashScopeChatModel
from agentscope.memory import InMemoryMemory
from agentscope.tool import Toolkit

toolkit = Toolkit()
toolkit.register_tool_function(execute_python_code)

agent = ReActAgent(
    name="assistant",
    model=DashScopeChatModel(model_name="qwen-max"),
    memory=InMemoryMemory(),
    toolkit=toolkit,
)
```

---

## 📚 文档资源

| 资源 | 链接 |
|------|------|
| **教程** | https://doc.agentscope.io/ |
| **中文文档** | https://github.com/agentscope-ai/agentscope/blob/main/README_zh.md |
| **路线图** | https://github.com/agentscope-ai/agentscope/blob/main/docs/roadmap.md |
| **FAQ** | https://doc.agentscope.io/tutorial/faq.html |
| **Discord** | https://discord.gg/eYMpfnkG8h |
| **论文** | arXiv:2402.14034, arXiv:2508.16279 |

---

## 💡 核心优势

1. **开发者友好**: 5 分钟快速上手，简洁的 API 设计
2. **生产就绪**: 内置部署、监控、可观测性支持
3. **灵活扩展**: 插件化架构，支持自定义 Agent、记忆、工具
4. **多 Agent 编排**: MsgHub 消息中心，灵活的协作模式
5. **微调集成**: 内置 RL 微调，支持多种训练场景
6. **实时交互**: 语音、中断、流式响应支持

---

## ⚠️ 注意事项

- **Python 版本**: 需要 3.10 或更高
- **依赖**: 部分功能需要额外安装（如 websockets, redis 等）
- **异步编程**: 主要 API 基于 asyncio，需要异步编程知识
- **文档**: API 文档暂时弃用，主要依赖教程和示例

---

## 🔗 相关引用

```bibtex
@article{agentscope_v1,
  author = {Dawei Gao, Zitao Li, Yuexiang Xie, et al.},
  title = {AgentScope 1.0: A Developer-Centric Framework for Building Agentic Applications},
  journal = {CoRR},
  volume = {abs/2508.16279},
  year = {2025},
}

@article{agentscope,
  author = {Dawei Gao, Zitao Li, Xuchen Pan, et al.},
  title = {AgentScope: A Flexible yet Robust Multi-Agent Platform},
  journal = {CoRR},
  volume = {abs/2402.14034},
  year = {2024},
}
```

---

**报告生成**: 2026-03-14  
**调研工具**: Web Fetch + GitHub API