# Kimi Code CLI 记忆系统设计文档

## 1. 概述

Kimi Code CLI 拥有一套完整的记忆系统，解决了以下核心问题：
- **长对话记忆保持
- **上下文长度限制
- **会话隔离与恢复
- **可回退的历史操作

---

## 2. 系统架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                         用户交互层                            │
│  (Shell / Web / ACP / Print                               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    KimiSoul (核心循环)                          │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  1. 接收用户输入                               │ │
│  │  2. 追加到 Context                            │ │
│  │  3. 调用 LLM (检查是否需要压缩)              │ │
│  │  4. 执行工具调用                             │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Context (对话历史管理)                      │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  - 内存中的历史列表 (_history)                    │ │
│  │  - 实时保存到 context.jsonl                    │ │
│  │  - 检查点标记 (_checkpoint)                    │ │
│  │  - Token 使用量统计                          │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Compaction (上下文压缩)                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  - SimpleCompaction                          │ │
│  │  - 保留最近 N 条消息                          │ │
│  │  - LLM 压缩之前的历史                          │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Session (会话持久化)                        │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  - 会话目录 (~/.kimi/sessions/<id>/)          │ │
│  │  - context.jsonl                          │ │
│  │  - wire.jsonl                          │ │
│  │  - session_state.json                          │ │
│  │  - subagents/                          │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 核心组件

### 3.1 Context 类

**位置**: `src/kimi_cli/soul/context.py`

Context 是记忆系统的核心，负责管理当前会话的对话历史。

#### 核心属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `_history` | `list[Message]` | 内存中的消息列表 |
| `_file_backend` | `Path` | 持久化文件路径 (`context.jsonl`) |
| `_token_count` | `int` | 实际 token 使用量 |
| `_pending_token_estimate` | `int` | 待确认的 token 估计值 |
| `_next_checkpoint_id` | `int` | 下一个检查点 ID |
| `_system_prompt` | `str\|None` | 系统提示词 |

#### 核心方法

| 方法 | 说明 |
|------|------|
| `restore()` | 从文件恢复对话历史 |
| `append_message()` | 追加消息并保存 |
| `checkpoint()` | 创建检查点 |
| `revert_to()` | 回退到指定检查点 |
| `update_token_count()` | 更新 token 统计 |
| `clear()` | 清空历史 |
| `write_system_prompt()` | 写入系统提示词 |

#### 持久化格式

`context.jsonl` 采用 JSONL 格式，每行一个记录：

```json
// 系统提示词
{"role": "_system_prompt", "content": "..."}

// 检查点
{"role": "_checkpoint", "id": 0}

// 普通消息
{"role": "user", "content": [...]}

// Token 使用量
{"role": "_usage", "token_count": 1024}
```

---

### 3.2 Session 类

**位置**: `src/kimi_cli/session.py`

Session 管理会话的生命周期和持久化。

#### 会话目录结构

```
~/.kimi/sessions/<session-id>/
├── context.jsonl          # 对话历史
├── wire.jsonl           # Wire 消息日志
├── session_state.json   # 会话状态
└── subagents/           # 子 Agent 目录
    └── <subagent-id>/
        ├── context.jsonl
        └── ...
```

#### 会话状态

| 字段 | 说明 |
|------|------|
| `custom_title` | 自定义标题 |
| `title_generated` | 是否已经生成标题 |
| `archived` | 是否已归档 |
| `auto_archive_exempt` | 是否免于自动归档 |
| `approval_mode` | 审批模式 |
| `plan_mode` | 计划模式 |

---

### 3.3 Compaction 系统

**位置**: `src/kimi_cli/soul/compaction.py`

上下文压缩系统，解决长对话的 token 限制。

#### 压缩触发条件

```python
# 两个条件满足任一即触发压缩：
1. token_count >= max_context_size * trigger_ratio
2. token_count + reserved_context_size >= max_context_size
```

#### SimpleCompaction 流程

```
原始对话历史：
[  [用户消息 1] → [助手回复 1] → [用户消息 2] → [助手回复 2] → [用户消息 3] → [助手回复 3]
                               ↓
压缩后：
[  (LLM 摘要: "之前用户问了 A、B、C，助手回复了 X、Y、Z... ] → [用户消息 3] → [助手回复 3]
```

---

## 4. 完整流程

### 4.1 会话初始化流程

```
用户执行 kimi
  ↓
创建 Session
  ↓
创建 Context
  ↓
从 context.jsonl 恢复历史
  ↓
加载系统提示词
  ↓
进入交互循环
```

### 4.2 对话交互流程

```
用户输入
  ↓
追加到 Context (append_message())
  ↓
保存到 context.jsonl
  ↓
检查 token_count 统计
  ↓
调用 LLM
  ↓
判断是否需要压缩
  ├─ 需要 → SimpleCompaction.compact()
  │           ↓
  │       保留最后 N 条
  │           ↓
  │       LLM 压缩前面的
  │           ↓
  │       更新 Context
  └─ 不需要 → 继续
  ↓
更新 token_count
  ↓
保存 _usage 记录
  ↓
返回结果给用户
```

### 4.3 检查点与回退流程

```
创建检查点
  ↓
写入 _checkpoint 记录
  ↓
执行操作 A
  ↓
执行操作 B
  ↓
用户执行 /undo
  ↓
revert_to(2)
  ↓
旋转 context.jsonl 到 .old
  ↓
从旧文件恢复到检查点 2
  ↓
截断新文件
  ↓
内存状态回退完成
```

---

## 5. 关键设计决策

### 5.1 JSONL 持久化格式

**决策**: 使用 JSONL 而非完整 JSON

**原因**:
- 追加写入效率高，无需重写整个文件
- 崩溃安全，最后一行可能损坏但前面完好
- 内存占用小，无需全部加载到内存
- 便于阅读和调试

### 5.2 检查点机制

**决策**: 类似 Git 的提交/回退

**原因**:
- 用户可以安全实验，不用担心搞砸了也能回退
- 提供安全性，AI 代理的撤销操作
- 实现简单，只需标记 ID

### 5.3 LLM 驱动压缩

**决策**: 保留最近 N 条，LLM 压缩前面的

**原因**:
- 最近对话更重要
- 压缩保留语义，不依赖简单截断
- 保留连贯性，上下文更流畅

---

## 6. 与其他 Agent 记忆系统对比

| 特性 | Kimi Code CLI | AutoGPT | LangChain Memory | Claude 3 | GPT-4 |
|------|-------------|---------|----------------|---------|-------|
| **持久化方式** | JSONL 实时追加 | SQLite/JSON | 多种（Vector DB/JSON | 会话存储 | 会话存储 |
| **压缩策略** | LLM 摘要 + 保留最近 | 滑动窗口 | 多种（Summary/Vector） | 未知 | 未知 |
| **可回退** | ✅ 检查点机制 | ❌ | ❌ | ❌ | ❌ |
| **会话隔离** | ✅ 独立目录 | ✅ | 部分支持 | ✅ | ✅ |
| **子 Agent 记忆** | ✅ 独立会话 | ✅ | ❌ | 部分支持 | 部分支持 |
| **崩溃恢复** | ✅ 从最后状态恢复 | ❌ 重启重新开始 | ❌ 重启重新开始 | ✅ | ✅ |
| **内存占用** | 低（流式加载 | 中 | 高（全量 | 中 | 中 |
| **调试友好** | ✅ 人类可读 | 部分支持 | 部分支持 | ❌ | ❌ |

---

## 7. 代码示例

### 7.1 使用 Context

```python
from pathlib import Path
from kimi_cli.soul.context import Context
from kosong.message import Message

# 创建 Context
ctx = Context(Path("context.jsonl"))

# 恢复历史
await ctx.restore()

# 追加消息
msg = Message(role="user", content=[TextPart(text="你好")])
await ctx.append_message(msg)

# 创建检查点
await ctx.checkpoint(add_user_message=True)

# 回退到检查点
await ctx.revert_to(0)
```

### 7.2 使用 Session

```python
from kaos.path import KaosPath
from kimi_cli.session import Session

# 创建会话
session = await Session.create(KaosPath("/workdir"))

# 恢复会话
session = await Session.find(KaosPath("/workdir"), "session-id"))

# 列出会话
sessions = await Session.list(KaosPath("/workdir"))
```

### 7.3 使用 Compaction

```python
from kimi_cli.soul.compaction import SimpleCompaction
from kosong.message import Message
from kimi_cli.llm import LLM

compactor = SimpleCompaction(max_preserved_messages=2)
result = await compactor.compact(messages, llm)

# 结果包含：
# - messages: 压缩后的消息列表
# - usage: token 使用量
```

---

## 8. 性能特点

### 8.1 优点

1. **崩溃安全**
- 每条消息实时保存到磁盘
- JSONL 格式，损坏行不影响前面
- 支持从任意状态恢复

2. **可回退性**
- 类似 Git 的检查点/回退
- 支持安全地做实验
- `/undo` 快速撤销

3. **智能压缩**
- LLM 驱动，保留语义
- 保留最近对话，确保连续性
- 支持自定义压缩指令

4. **会话隔离**
- 每个会话独立目录
- 互不干扰
- 支持跨会话切换

### 8.2 限制

1. **压缩开销**
- 压缩需要额外调用 LLM
- 有 token 成本
- 有延迟

2. **无跨会话共享**
- 会话之间独立
- 没有跨会话记忆共享

---

## 9. 未来方向

- [ ] 跨会话记忆搜索
- [ ] 向量数据库集成
- [ ] 压缩策略插件化
- [ ] 记忆重要性评分
- [ ] 用户偏好学习

