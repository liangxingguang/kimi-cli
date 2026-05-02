# 核心模块

## 模块总览

Kimi Code CLI 包含多个核心模块，每个模块负责特定的功能领域：

| 模块 | 位置 | 职责 |
|------|------|------|
| **CLI** | `src/kimi_cli/cli/` | 命令行入口和参数解析 |
| **App** | `src/kimi_cli/app.py` | 应用生命周期管理 |
| **Soul** | `src/kimi_cli/soul/` | 核心代理循环 |
| **UI** | `src/kimi_cli/ui/` | 用户界面层 |
| **Wire** | `src/kimi_cli/wire/` | 消息传输层 |
| **Tools** | `src/kimi_cli/tools/` | 内置工具集 |
| **ACP** | `src/kimi_cli/acp/` | ACP 服务器 |
| **Config** | `src/kimi_cli/config.py` | 配置管理 |
| **LLM** | `src/kimi_cli/llm.py` | LLM 集成 |

---

## CLI 模块

### 核心组件

#### `cli` 对象

**位置**: `src/kimi_cli/cli/__init__.py:40`

主 CLI 实例，使用 Typer 框架创建：

```python
cli = typer.Typer(
    cls=LazySubcommandGroup,
    epilog="Documentation: https://moonshotai.github.io/kimi-cli/",
    add_completion=False,
    context_settings={"help_option_names": ["-h", "--help"]},
    help="Kimi, your next CLI agent.",
)
```

#### `kimi` 回调函数

**位置**: `src/kimi_cli/cli/__init__.py:77`

主命令入口，处理所有命令行参数并路由到相应的运行模式。

**主要参数**:
- `--work-dir` / `-w`: 工作目录
- `--session` / `-S`: 恢复会话
- `--model` / `-m`: 指定模型
- `--yolo` / `-y`: 自动审批模式
- `--print`: 非交互式打印模式
- `--acp`: ACP 服务器模式
- `--wire`: Wire 模式

---

## App 模块

### KimiCLI 类

**位置**: `src/kimi_cli/app.py:119`

应用核心类，负责创建和运行 Kimi Code CLI 实例。

#### 关键方法

##### `create()`

**签名**:
```python
@staticmethod
async def create(
    session: Session,
    *,
    config: Config | Path | None = None,
    model_name: str | None = None,
    thinking: bool | None = None,
    yolo: bool = False,
    afk: bool = False,
    runtime_afk: bool = False,
    plan_mode: bool = False,
    resumed: bool = False,
    ui_mode: str = "shell",
    agent_file: Path | None = None,
    mcp_configs: list[MCPConfig] | list[dict[str, Any]] | None = None,
    skills_dirs: list[KaosPath] | None = None,
    max_steps_per_turn: int | None = None,
    max_retries_per_step: int | None = None,
    max_ralph_iterations: int | None = None,
    startup_progress: Callable[[str], None] | None = None,
    defer_mcp_loading: bool = False,
) -> KimiCLI
```

**职责**:
1. 加载配置
2. 创建 OAuth 管理器
3. 初始化 LLM 实例
4. 创建 Runtime
5. 加载 Agent 规格
6. 恢复上下文
7. 创建 KimiSoul
8. 初始化钩子引擎和遥测

##### `run()`

**签名**:
```python
async def run(
    self,
    user_input: str | list[ContentPart],
    cancel_event: asyncio.Event,
    merge_wire_messages: bool = False,
) -> AsyncGenerator[WireMessage]
```

**职责**: 运行核心循环，直接生成 Wire 消息。

##### `run_shell()`

**签名**:
```python
async def run_shell(
    self,
    command: str | None = None,
    *,
    prefill_text: str | None = None
) -> bool
```

**职责**: 启动交互式 shell UI。

##### `run_print()`

**签名**:
```python
async def run_print(
    self,
    input_format: InputFormat,
    output_format: OutputFormat,
    command: str | None = None,
    *,
    final_only: bool = False,
) -> int
```

**职责**: 启动非交互式打印模式。

##### `run_acp()`

**签名**:
```python
async def run_acp(self) -> None
```

**职责**: 启动 ACP 服务器模式。

---

## Soul 模块

### Runtime 类

**位置**: `src/kimi_cli/soul/agent.py`

会话级别的运行时上下文，包含所有会话相关的服务和状态。

**核心属性**:
- `config`: 配置对象
- `oauth`: OAuth 管理器
- `llm`: LLM 实例
- `session`: 当前会话
- `approval`: 审批系统
- `approval_runtime`: 审批运行时
- `hooks`: 钩子引擎
- `notifications`: 通知管理器
- `background_tasks`: 后台任务管理器

### KimiSoul 类

**位置**: `src/kimi_cli/soul/kimisoul.py`

核心代理循环实现。

**核心方法**:
- `run()`: 执行代理循环
- `process_slash_command()`: 处理斜杠命令
- `call_llm()`: 调用 LLM
- `execute_tool()`: 执行工具调用
- `compact_context()`: 压缩上下文

### Context 类

**位置**: `src/kimi_cli/soul/context.py`

对话历史管理器。

**核心方法**:
- `restore()`: 恢复上下文
- `append()`: 追加消息
- `write_system_prompt()`: 写入系统提示词
- `compact()`: 执行压缩

---

## Wire 模块

### Wire 类

**位置**: `src/kimi_cli/wire/__init__.py`

消息传输总线，连接 UI 和核心运行时。

**消息类型**:
- `TextMessage`: 文本消息
- `ToolCall`: 工具调用
- `ToolResult`: 工具结果
- `ApprovalRequest`: 审批请求
- `ApprovalResponse`: 审批响应
- `SessionStart`: 会话开始
- `SessionEnd`: 会话结束

---

## Tools 模块

### 内置工具列表

| 工具名称 | 位置 | 功能 |
|----------|------|------|
| **Agent** | `src/kimi_cli/tools/agent/` | 子 Agent 管理 |
| **AskUser** | `src/kimi_cli/tools/ask_user/` | 询问用户 |
| **Background** | `src/kimi_cli/tools/background/` | 后台任务管理 |
| **DMail** | `src/kimi_cli/tools/dmail/` | 邮件发送 |
| **File** | `src/kimi_cli/tools/file/` | 文件操作（Read, Glob, Grep） |
| **Shell** | `src/kimi_cli/tools/shell/` | Shell 命令执行 |

---

## Config 模块

### Config 类

**位置**: `src/kimi_cli/config.py`

配置管理类，支持从多个来源加载配置。

**配置结构**:
```python
class Config:
    default_model: str | None
    default_thinking: bool
    default_yolo: bool
    default_plan_mode: bool
    providers: dict[str, LLMProvider]
    models: dict[str, LLMModel]
    loop_control: LoopControl
    hooks: HookConfig
    background: BackgroundConfig
    telemetry: bool
```

---

## LLM 模块

### create_llm() 函数

**位置**: `src/kimi_cli/llm.py`

创建 LLM 实例的工厂函数。

**支持的提供商**:
- Kimi
- OpenAI
- Anthropic
- Ollama

---

## ACP 模块

### ACP 类

**位置**: `src/kimi_cli/ui/acp/__init__.py`

ACP 服务器实现，支持 Agent Client Protocol。

**核心功能**:
- 会话管理
- 消息传递
- 工具调用
- 审批处理
