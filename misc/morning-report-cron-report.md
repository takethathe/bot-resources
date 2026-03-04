# 晨间报告定时任务

# Morning Report Cron System

**Generated**: 2026-03-04 08:02:31 UTC  
**Type**: system  
**Category**: misc  
**Topic**: morning-report-cron

---


**创建时间**: 2026-03-04 08:02  
**类型**: 定时任务  
**状态**: ✅ 已激活

---

## 📋 概述

创建一个定时循环任务，北京时间每天早上 8 点自动发送服务器状态 + 天气 + 未完成 todo 的联合信息到飞书频道。

---

## ⏰ 定时配置

| 配置项 | 值 |
|--------|-----|
| **Cron 表达式** | `0 8 * * *` |
| **时区** | Asia/Shanghai (UTC+8) |
| **执行时间** | 每天早上 8:00 |
| **Job ID** | `cbdff00a` |

---

## 📊 报告内容

### 1. 服务器状态
- CPU 使用率
- 内存使用量（已用/总量/百分比）
- 磁盘使用率

### 2. 上海天气
- 当前温度（°C）
- 体感温度
- 天气状况
- 湿度
- 风速

### 3. 待办事项
- 未完成的待办列表
- 待处理数量统计

---

## 📁 文件结构

```
/root/.nanobot/workspace/scripts/
├── daily_morning_report.py    # 报告生成脚本
├── run_morning_report.py      # 执行器（生成 + 发送）
└── send_morning_report.py     # 发送脚本（备用）
```

---

## 🚀 使用方法

### 手动生成报告
```bash
python3 /root/.nanobot/workspace/scripts/daily_morning_report.py
```

### 手动执行并发送
```bash
python3 /root/.nanobot/workspace/scripts/run_morning_report.py
```

### 查看定时任务
```bash
nanobot cron list
```

### 删除定时任务
```bash
nanobot cron remove --job-id cbdff00a
```

---

## 📝 输出示例

```
🌅 晨间报告 | 2026 年 03 月 04 日 Wednesday 08:00

🖥️ 服务器状态
   ─────────────────
   CPU:    1.8%
   内存：  700MB / 895MB (78.2%)
   磁盘：  35%

🌤️  上海天气
   ─────────────────
   温度：  12°C (体感 10°C)
   天气：  晴
   湿度：  65%
   风速：  15 km/h

📋 待办事项
   ─────────────────
   ⬜ 1. 学习 Rust 编程语言
   ⬜ 2. 阅读《道德经》

   共 2 项待处理

────────────────────────────────────────
🤖 nanobot 自动报告
```

---

## 🔧 技术实现

### 报告生成脚本 (`daily_morning_report.py`)

**功能模块**:
1. `get_cpu_usage()` - 从 /proc/stat 读取 CPU 使用率
2. `get_memory_info()` - 从 /proc/meminfo 读取内存信息
3. `get_disk_usage()` - 使用 df 命令获取磁盘使用率
4. `get_weather()` - 从 wttr.in 获取上海天气
5. `get_pending_todos()` - 读取 TODO.json 获取未完成待办
6. `format_report()` - 格式化输出报告

**数据源**:
- 服务器状态：/proc 文件系统 + df 命令
- 天气：wttr.in API
- 待办：/root/.nanobot/workspace/todos/TODO.json

### 定时任务配置

使用 nanobot cron skill:
```python
cron(
    action="add",
    cron_expr="0 8 * * *",
    tz="Asia/Shanghai",
    message="每天早上 8 点晨间报告任务..."
)
```

---

## 🎯 发送目标

| 配置 | 值 |
|------|-----|
| **频道** | feishu |
| **Chat ID** | ou_62dcc4b3b169d86b43f51c114349257c |

---

## ⚠️ 注意事项

1. **天气获取**: 依赖网络连接，偶尔可能超时
2. **待办事项**: 需要 todo-manager skill 支持
3. **时区**: 使用 Asia/Shanghai (UTC+8)
4. **发送权限**: 需要飞书频道发送权限

---

## 🔮 未来扩展

- [ ] 支持多个城市天气
- [ ] 添加更多服务器指标（网络、进程数）
- [ ] 支持自定义接收人
- [ ] 添加历史趋势对比
- [ ] 支持工作日/周末不同报告
- [ ] 添加新闻摘要

---

## 📚 相关文档

- Todo Manager Skill: `/root/.nanobot/workspace/skills/todo-manager/`
- Cron Skill: `/usr/local/lib/python3.12/site-packages/nanobot/skills/cron/SKILL.md`

---

## ✅ 测试记录

| 时间 | 测试项 | 状态 |
|------|--------|------|
| 2026-03-04 08:01 | 报告生成 | ✅ 成功 |
| 2026-03-04 08:02 | 消息发送 | ✅ 成功 |
| 2026-03-04 08:02 | Cron 任务创建 | ✅ 成功 |

---

*文档已同步到 GitHub: https://github.com/takethathe/bot-resources*