# 配置说明

## 配置文件

### 配置文件位置

配置文件默认位于：`~/.kimi/config.toml`

### 配置格式

配置文件使用 TOML 格式，结构如下：

```toml
# 默认设置
default_model = "kimi-for-coding"
default_thinking = true
default_yolo = false
default_plan_mode = false

# LLM 提供商配置
[providers.kimi]
type = "kimi"
base_url = "https://api.moonshot.cn/v1"

[providers.openai]
type = "openai"
base_url = "https://api.openai.com/v1"
api_key = "your-api-key"

# 模型配置
[models.kimi-for-coding]
provider = "kimi"
model = "kimi-for-coding"
max_context_size = 128000

[models.gpt-4o]
provider = "openai"
model = "gpt-4o"
max_context_size = 128000

# 循环控制
[loop_control]
max_steps_per_turn = 10
max_retries_per_step = 3
max_ralph_iterations = 3

# 钩子配置
[hooks]
enabled = true

# 后台任务配置
[background]
keep_alive_on_exit = false
kill_grace_period_ms = 2000

# 遥测配置
telemetry = true
```

---

## 配置选项详解

### 默认设置

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `default_model` | string | None | 默认使用的模型名称 |
| `default_thinking` | bool | false | 是否默认启用思考模式 |
| `default_yolo` | bool | false | 是否默认启用自动批准 |
| `default_plan_mode` | bool | false | 是否默认启用计划模式 |

### 提供商配置

提供商类型支持：
- `kimi`: Kimi API
- `openai`: OpenAI API
- `anthropic`: Anthropic API
- `ollama`: Ollama

**通用选项**:
| 选项 | 类型 | 描述 |
|------|------|------|
| `type` | string | 提供商类型 |
| `base_url` | string | API 基础 URL |
| `api_key` | string | API 密钥 |

### 模型配置

| 选项 | 类型 | 描述 |
|------|------|------|
| `provider` | string | 提供商名称（对应 providers 中的键） |
| `model` | string | 模型名称 |
| `max_context_size` | int | 最大上下文长度 |

### 循环控制

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `max_steps_per_turn` | int | 10 | 每轮最大步骤数 |
| `max_retries_per_step` | int | 3 | 每步最大重试次数 |
| `max_ralph_iterations` | int | 3 | Ralph 模式额外迭代次数 |

### 后台任务配置

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `keep_alive_on_exit` | bool | false | 退出时保持后台任务运行 |
| `kill_grace_period_ms` | int | 2000 | 终止前的宽限期（毫秒） |

---

## 环境变量

### 优先级

环境变量优先级高于配置文件，但低于命令行参数。

### 支持的环境变量

| 变量名 | 描述 | 示例 |
|--------|------|------|
| `KIMI_API_KEY` | Kimi API 密钥 | `sk-xxx` |
| `KIMI_BASE_URL` | Kimi API 基础 URL | `https://api.moonshot.cn/v1` |
| `KIMI_MODEL_NAME` | 默认模型名称 | `kimi-for-coding` |
| `KIMI_WORK_DIR` | 默认工作目录 | `/projects/my-app` |
| `KIMI_DISABLE_TELEMETRY` | 禁用遥测 | `true` |
| `KIMI_CONFIG_DIR` | 配置目录 | `~/.kimi` |

---

## 命令行覆盖

命令行参数优先级最高，可以覆盖配置文件和环境变量：

```bash
# 覆盖模型配置
kimi --model kimi-for-coding

# 覆盖思考模式
kimi --thinking

# 覆盖自动批准
kimi --yolo

# 覆盖配置文件
kimi --config-file ./custom-config.toml

# 直接指定配置
kimi --config 'default_model = "gpt-4o"'
```

---

## MCP 配置

### MCP 配置文件

MCP 配置文件使用 JSON 格式：

```json
{
  "mcpServers": {
    "context7": {
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "your-api-key"
      }
    },
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    },
    "linear": {
      "url": "https://mcp.linear.app/mcp",
      "auth": {
        "type": "oauth"
      }
    }
  }
}
```

### MCP 配置选项

#### HTTP 传输

| 选项 | 类型 | 描述 |
|------|------|------|
| `url` | string | MCP 服务器 URL |
| `headers` | object | HTTP 头部 |
| `auth` | object | 认证配置 |

#### stdio 传输

| 选项 | 类型 | 描述 |
|------|------|------|
| `command` | string | 命令路径 |
| `args` | array | 命令参数 |

### MCP CLI 命令

```bash
# 添加 MCP 服务器
kimi mcp add --transport http context7 https://mcp.context7.com/mcp --header "CONTEXT7_API_KEY: xxx"

# 添加 stdio 服务器
kimi mcp add --transport stdio chrome-devtools -- npx chrome-devtools-mcp@latest

# 列出已配置的 MCP 服务器
kimi mcp list

# 删除 MCP 服务器
kimi mcp remove context7

# OAuth 授权
kimi mcp auth linear
```

---

## 技能配置

### 技能目录

技能会自动从以下目录发现：
- `~/.kimi/skills/`
- `./.agents/skills/`（项目级别）

### 技能结构

每个技能是一个目录，包含：
- `SKILL.md`: 技能定义和用户提示词

**SKILL.md 示例**:
```markdown
# My Skill

## Description

这是一个自定义技能的描述。

## Usage

如何使用这个技能的说明。
```

### 加载自定义技能目录

```bash
kimi --skills-dir /path/to/skills
```

---

## 数据路径

### 默认路径

| 路径 | 描述 |
|------|------|
| `~/.kimi/` | 主数据目录 |
| `~/.kimi/config.toml` | 配置文件 |
| `~/.kimi/logs/` | 日志目录 |
| `~/.kimi/sessions/` | 会话目录 |
| `~/.kimi/mcp/` | MCP 配置目录 |
| `~/.kimi/skills/` | 技能目录 |
| `~/.kimi/plugins/` | 插件目录 |

### 自定义数据路径

通过环境变量 `KIMI_SHARE_DIR` 设置自定义数据目录：

```bash
export KIMI_SHARE_DIR=/custom/path/to/kimi
```

---

## 配置示例

### 完整配置文件

```toml
# ~/.kimi/config.toml

# 默认设置
default_model = "kimi-for-coding"
default_thinking = true
default_yolo = false
default_plan_mode = false

# Kimi 提供商
[providers.kimi]
type = "kimi"
base_url = "https://api.moonshot.cn/v1"

# OpenAI 提供商（可选）
[providers.openai]
type = "openai"
base_url = "https://api.openai.com/v1"
api_key = "sk-your-openai-key"

# Kimi 模型
[models.kimi-for-coding]
provider = "kimi"
model = "kimi-for-coding"
max_context_size = 128000

[models.kimi-code]
provider = "kimi"
model = "kimi-code"
max_context_size = 8192

# OpenAI 模型（可选）
[models.gpt-4o]
provider = "openai"
model = "gpt-4o"
max_context_size = 128000

# 循环控制
[loop_control]
max_steps_per_turn = 15
max_retries_per_step = 5
max_ralph_iterations = 5

# 后台任务
[background]
keep_alive_on_exit = false
kill_grace_period_ms = 3000

# 遥测
telemetry = true
```

### 最小配置

```toml
# 最小配置 - 使用默认设置
[providers.kimi]
type = "kimi"

[models.kimi-for-coding]
provider = "kimi"
model = "kimi-for-coding"

default_model = "kimi-for-coding"
```
