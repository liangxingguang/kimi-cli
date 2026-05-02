# 项目概述

## 项目介绍

Kimi Code CLI 是一个运行在终端中的 AI 代理，专为软件开发工作流设计。它支持交互式 shell UI、ACP 服务器模式用于 IDE 集成，以及 MCP 工具加载。

### 核心特性

1. **Shell 命令模式**
   - 支持在终端中直接运行 shell 命令
   - 通过 `Ctrl-X` 切换命令模式
   - 无需离开 Kimi Code CLI 即可执行系统命令

2. **IDE 集成**
   - 原生支持 [Agent Client Protocol (ACP)](https://github.com/agentclientprotocol/agent-client-protocol)
   - 可与 VS Code、Zed、JetBrains IDE 等集成
   - 提供流畅的 IDE 内 AI 助手体验

3. **MCP 支持**
   - 支持 Model Context Protocol 工具
   - 可管理多个 MCP 服务器
   - 支持 HTTP 和 stdio 传输方式

4. **会话管理**
   - 支持会话的创建、恢复和切换
   - 自动保存对话历史
   - 支持跨会话的上下文保持

5. **计划模式**
   - 支持在执行前规划任务步骤
   - 可视化展示执行计划
   - 支持手动调整和确认

## 技术栈

- **Python**: 3.12+（工具配置为 3.14）
- **CLI 框架**: Typer
- **异步运行时**: asyncio
- **LLM 框架**: kosong（自研）
- **MCP 集成**: fastmcp
- **日志**: loguru
- **包管理**: uv + uv_build
- **打包**: PyInstaller

## 安装与启动

### 安装方式

```bash
# 使用 pip
pip install kimi-cli

# 使用 uv（推荐）
uv add kimi-cli
```

### 启动命令

```bash
# 启动交互式 shell
kimi

# 指定工作目录
kimi --work-dir /path/to/project

# 恢复之前的会话
kimi --resume <session-id>

# 非交互式打印模式
kimi --print --prompt "帮我分析这个项目"

# 以 ACP 服务器模式运行
kimi acp
```

## 基本使用

### 交互模式

启动后，您可以直接输入自然语言指令：

```
> 帮我查看当前目录结构
> 分析这个 Python 项目的架构
> 创建一个新的 Flask 应用
> 解释这段代码的功能
```

### Shell 命令模式

按 `Ctrl-X` 切换到 shell 命令模式：

```
$ ls -la
$ git status
$ python -m pytest
```

### 斜杠命令

Kimi Code CLI 支持多种斜杠命令：

- `/login` - 登录 Kimi 账号
- `/new` - 创建新会话
- `/undo` - 撤销上一步
- `/model` - 切换模型
- `/plan` - 进入计划模式
- `/feedback` - 提交反馈

## 项目结构

```
kimi-cli/
├── src/kimi_cli/          # 主源码目录
│   ├── cli/              # CLI 入口
│   ├── soul/             # 核心运行时
│   ├── ui/               # 用户界面
│   ├── tools/            # 内置工具
│   ├── agents/           # Agent 规格
│   └── acp/              # ACP 服务器
├── docs/                 # 文档
├── tests/                # 测试
├── packages/             # 依赖包
│   ├── kosong/           # LLM 抽象层
│   └── kaos/             # OS 交互层
└── examples/             # 示例代码
```

## 核心概念

### Agent

Agent 是一个配置化的 AI 助手，包含系统提示词、工具集和行为规则。

### Session

Session 是一个独立的对话上下文，包含消息历史、工具调用记录和会话状态。

### Toolset

Toolset 是工具的集合，支持内置工具和 MCP 工具的加载与调用。

### Wire

Wire 是 UI 和核心运行时之间的消息传输层，支持多种 UI 模式。

## 扩展能力

Kimi Code CLI 支持多种扩展方式：

1. **MCP 工具** - 通过 Model Context Protocol 加载外部工具
2. **Skills** - 通过技能系统添加自定义功能
3. **Plugins** - 通过插件系统扩展 CLI 功能
4. **Hooks** - 通过钩子系统在特定事件触发自定义逻辑
