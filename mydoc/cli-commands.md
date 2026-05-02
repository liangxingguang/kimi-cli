# CLI 命令

## 命令概览

Kimi Code CLI 提供以下命令和子命令：

| 命令 | 描述 |
|------|------|
| `kimi` | 主命令，启动交互式 shell |
| `kimi login` | 登录 Kimi 账号 |
| `kimi logout` | 退出登录 |
| `kimi acp` | 启动 ACP 服务器 |
| `kimi term` | 启动 Toad TUI |
| `kimi mcp` | MCP 服务器管理 |
| `kimi info` | 显示信息 |
| `kimi vis` | 启动可视化界面 |
| `kimi web` | 启动 Web 界面 |

---

## 主命令：kimi

### 基本用法

```bash
kimi [OPTIONS]
```

### 核心选项

#### 元数据选项

| 选项 | 描述 |
|------|------|
| `-V`, `--version` | 显示版本信息 |
| `--verbose` | 打印详细信息 |
| `--debug` | 启用调试日志 |

#### 工作目录选项

| 选项 | 描述 |
|------|------|
| `-w`, `--work-dir PATH` | 指定工作目录 |
| `--add-dir PATH` | 添加额外目录到工作空间 |

#### 会话选项

| 选项 | 描述 |
|------|------|
| `-S`, `--session [ID]` | 恢复会话（可选 ID） |
| `-C`, `--continue` | 继续上一个会话 |

#### 配置选项

| 选项 | 描述 |
|------|------|
| `--config STRING` | 配置 TOML/JSON 字符串 |
| `--config-file PATH` | 配置文件路径 |
| `-m`, `--model NAME` | 指定模型名称 |
| `--thinking/--no-thinking` | 启用/禁用思考模式 |

#### 运行模式选项

| 选项 | 描述 |
|------|------|
| `-y`, `--yolo`, `--yes`, `--auto-approve` | 自动批准所有操作 |
| `--plan` | 启动计划模式 |
| `--afk` | AFK 模式（无人值守） |
| `-p`, `--prompt`, `--command`, `-c STRING` | 用户提示 |
| `--print` | 非交互式打印模式 |
| `--acp` | ACP 服务器模式（已弃用） |
| `--wire` | Wire 模式 |

#### 输出格式选项（仅 print 模式）

| 选项 | 描述 |
|------|------|
| `--input-format FORMAT` | 输入格式（text/stream-json） |
| `--output-format FORMAT` | 输出格式（text/stream-json） |
| `--final-message-only` | 仅输出最终消息 |
| `--quiet` | 静默模式（等同于 `--print --output-format text --final-message-only`） |

#### 自定义选项

| 选项 | 描述 |
|------|------|
| `--agent NAME` | 指定内置 Agent（default/okabe） |
| `--agent-file PATH` | 自定义 Agent 文件 |
| `--mcp-config-file PATH` | MCP 配置文件 |
| `--mcp-config JSON` | MCP 配置 JSON |
| `--skills-dir PATH` | 自定义技能目录 |

#### 循环控制选项

| 选项 | 描述 |
|------|------|
| `--max-steps-per-turn N` | 每轮最大步骤数 |
| `--max-retries-per-step N` | 每步最大重试次数 |
| `--max-ralph-iterations N` | Ralph 模式额外迭代次数 |

### 使用示例

```bash
# 启动交互式 shell
kimi

# 指定工作目录
kimi --work-dir /projects/my-app

# 恢复特定会话
kimi --session abc12345

# 交互式选择会话
kimi --session

# 继续上一个会话
kimi --continue

# 指定模型
kimi --model kimi-for-coding

# 启用思考模式
kimi --thinking

# 自动批准模式
kimi --yolo

# 非交互式执行
kimi --print --prompt "分析这个项目"

# 静默模式
kimi --quiet --prompt "列出目录内容"

# 使用自定义 Agent
kimi --agent-file ./my-agent.yaml

# 加载 MCP 配置
kimi --mcp-config-file ./mcp-config.json
```

---

## kimi login

### 用法

```bash
kimi login [OPTIONS]
```

### 选项

| 选项 | 描述 |
|------|------|
| `--json` | 以 JSON 格式输出 OAuth 事件 |

### 示例

```bash
# 交互式登录
kimi login

# JSON 模式登录
kimi login --json
```

---

## kimi logout

### 用法

```bash
kimi logout [OPTIONS]
```

### 选项

| 选项 | 描述 |
|------|------|
| `--json` | 以 JSON 格式输出 OAuth 事件 |

### 示例

```bash
# 交互式登出
kimi logout

# JSON 模式登出
kimi logout --json
```

---

## kimi acp

### 用法

```bash
kimi acp
```

启动 ACP 服务器，用于 IDE 集成。

### 示例

```bash
# 启动 ACP 服务器
kimi acp
```

---

## kimi term

### 用法

```bash
kimi term [OPTIONS]
```

启动 Toad TUI，基于 Kimi Code CLI ACP 服务器。

### 示例

```bash
# 启动 Toad TUI
kimi term
```

---

## kimi mcp

### 子命令

| 子命令 | 描述 |
|--------|------|
| `list` | 列出已配置的 MCP 服务器 |
| `add` | 添加新的 MCP 服务器 |
| `remove` | 删除 MCP 服务器 |
| `auth` | 为 MCP 服务器授权 |

### kimi mcp list

列出所有已配置的 MCP 服务器。

```bash
kimi mcp list
```

### kimi mcp add

添加新的 MCP 服务器。

```bash
kimi mcp add [OPTIONS] NAME URL
```

**选项**:
- `--transport TYPE`: 传输类型（http/stdio）
- `--auth TYPE`: 认证类型（oauth）
- `--header KEY:VALUE`: HTTP 头部

**示例**:
```bash
# 添加 HTTP MCP 服务器
kimi mcp add --transport http context7 https://mcp.context7.com/mcp --header "CONTEXT7_API_KEY: xxx"

# 添加 stdio MCP 服务器
kimi mcp add --transport stdio chrome-devtools -- npx chrome-devtools-mcp@latest

# 添加 OAuth 认证的 MCP 服务器
kimi mcp add --transport http --auth oauth linear https://mcp.linear.app/mcp
```

### kimi mcp remove

删除指定的 MCP 服务器。

```bash
kimi mcp remove NAME
```

**示例**:
```bash
kimi mcp remove chrome-devtools
```

### kimi mcp auth

为 MCP 服务器进行 OAuth 授权。

```bash
kimi mcp auth NAME
```

**示例**:
```bash
kimi mcp auth linear
```

---

## 斜杠命令

在交互式 shell 中支持以下斜杠命令：

| 命令 | 描述 |
|------|------|
| `/login` | 登录 Kimi 账号 |
| `/logout` | 退出登录 |
| `/new` | 创建新会话 |
| `/undo` | 撤销上一步 |
| `/model [NAME]` | 切换模型 |
| `/plan` | 进入/退出计划模式 |
| `/yolo` | 切换自动批准模式 |
| `/afk` | 切换 AFK 模式 |
| `/reload` | 重新加载配置 |
| `/theme` | 切换主题 |
| `/feedback` | 提交反馈 |
| `/help` | 显示帮助 |
| `/exit` | 退出 |
| `/quit` | 退出 |

### 使用示例

```
> /login
> /model kimi-for-coding
> /plan
> /new
> /exit
```

---

## 退出码

| 退出码 | 含义 |
|--------|------|
| `0` | 成功 |
| `1` | 失败 |
| `75` | 可重试错误（EX_TEMPFAIL） |
