# API 参考

## 概述

Kimi Code CLI 提供了一组公共 API，允许开发者扩展和集成 Kimi Code CLI 功能。

---

## KimiCLI 类

### 位置

`src/kimi_cli/app.py`

### 核心方法

#### `create()`

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

**参数**:
- `session`: 会话对象
- `config`: 配置对象或路径
- `model_name`: 模型名称
- `thinking`: 是否启用思考模式
- `yolo`: 是否启用自动批准
- `afk`: 是否启用 AFK 模式
- `plan_mode`: 是否启用计划模式
- `ui_mode`: UI 模式（shell/print/acp/wire）
- `agent_file`: Agent 文件路径
- `mcp_configs`: MCP 配置列表
- `skills_dirs`: 自定义技能目录
- `max_steps_per_turn`: 每轮最大步骤数
- `max_retries_per_step`: 每步最大重试次数
- `max_ralph_iterations`: Ralph 模式额外迭代次数
- `startup_progress`: 启动进度回调
- `defer_mcp_loading`: 是否延迟加载 MCP

**返回**: `KimiCLI` 实例

#### `run()`

**签名**:
```python
async def run(
    self,
    user_input: str | list[ContentPart],
    cancel_event: asyncio.Event,
    merge_wire_messages: bool = False,
) -> AsyncGenerator[WireMessage]
```

**参数**:
- `user_input`: 用户输入
- `cancel_event`: 取消事件
- `merge_wire_messages`: 是否合并 Wire 消息

**返回**: Wire 消息生成器

#### `run_shell()`

**签名**:
```python
async def run_shell(
    self,
    command: str | None = None,
    *,
    prefill_text: str | None = None
) -> bool
```

**参数**:
- `command`: 初始命令
- `prefill_text`: 预填充文本

**返回**: 是否成功

#### `run_print()`

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

**参数**:
- `input_format`: 输入格式（text/stream-json）
- `output_format`: 输出格式（text/stream-json）
- `command`: 命令
- `final_only`: 是否仅输出最终消息

**返回**: 退出码

---

## Session 类

### 位置

`src/kimi_cli/session.py`

### 核心方法

#### `create()`

**签名**:
```python
@staticmethod
async def create(work_dir: KaosPath, session_id: str | None = None) -> Session
```

**参数**:
- `work_dir`: 工作目录
- `session_id`: 会话 ID（可选）

**返回**: Session 实例

#### `continue_()`

**签名**:
```python
@staticmethod
async def continue_(work_dir: KaosPath) -> Session | None
```

**参数**:
- `work_dir`: 工作目录

**返回**: 上一个会话或 None

#### `find()`

**签名**:
```python
@staticmethod
async def find(work_dir: KaosPath, session_id: str) -> Session | None
```

**参数**:
- `work_dir`: 工作目录
- `session_id`: 会话 ID

**返回**: 会话或 None

#### `list()`

**签名**:
```python
@staticmethod
async def list(work_dir: KaosPath) -> list[Session]
```

**参数**:
- `work_dir`: 工作目录

**返回**: 会话列表

---

## Config 类

### 位置

`src/kimi_cli/config.py`

### 核心方法

#### `load_config()`

**签名**:
```python
def load_config(path: Path | None = None) -> Config
```

**参数**:
- `path`: 配置文件路径（可选）

**返回**: Config 实例

#### `load_config_from_string()`

**签名**:
```python
def load_config_from_string(config_str: str) -> Config
```

**参数**:
- `config_str`: 配置字符串（TOML 或 JSON）

**返回**: Config 实例

### 配置结构

```python
class Config:
    default_model: str | None          # 默认模型
    default_thinking: bool             # 默认思考模式
    default_yolo: bool                 # 默认自动批准
    default_plan_mode: bool            # 默认计划模式
    providers: dict[str, LLMProvider]  # LLM 提供商
    models: dict[str, LLMModel]        # 模型配置
    loop_control: LoopControl          # 循环控制
    hooks: HookConfig                  # 钩子配置
    background: BackgroundConfig       # 后台任务配置
    telemetry: bool                    # 是否启用遥测
```

---

## LLM 模块

### 位置

`src/kimi_cli/llm.py`

### 核心函数

#### `create_llm()`

**签名**:
```python
def create_llm(
    provider: LLMProvider,
    model: LLMModel,
    thinking: bool = False,
    session_id: str | None = None,
    oauth: OAuthManager | None = None,
) -> Kosong | None
```

**参数**:
- `provider`: LLM 提供商配置
- `model`: 模型配置
- `thinking`: 是否启用思考模式
- `session_id`: 会话 ID
- `oauth`: OAuth 管理器

**返回**: Kosong 实例或 None

#### `augment_provider_with_env_vars()`

**签名**:
```python
def augment_provider_with_env_vars(
    provider: LLMProvider,
    model: LLMModel,
) -> dict[str, str]
```

**参数**:
- `provider`: LLM 提供商配置
- `model`: 模型配置

**返回**: 环境变量覆盖字典

---

## Runtime 类

### 位置

`src/kimi_cli/soul/agent.py`

### 核心属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `config` | Config | 配置对象 |
| `oauth` | OAuthManager | OAuth 管理器 |
| `llm` | Kosong | LLM 实例 |
| `session` | Session | 当前会话 |
| `approval` | ApprovalRuntime | 审批系统 |
| `hooks` | HookEngine | 钩子引擎 |
| `notifications` | NotificationManager | 通知管理器 |
| `background_tasks` | BackgroundTaskManager | 后台任务管理器 |

### 核心方法

#### `create()`

**签名**:
```python
@staticmethod
async def create(
    config: Config,
    oauth: OAuthManager,
    llm: Kosong | None,
    session: Session,
    yolo: bool,
    *,
    afk: bool = False,
    runtime_afk: bool = False,
    skills_dirs: list[KaosPath] | None = None,
) -> Runtime
```

**参数**:
- `config`: 配置对象
- `oauth`: OAuth 管理器
- `llm`: LLM 实例
- `session`: 会话
- `yolo`: 是否自动批准
- `afk`: 是否 AFK 模式
- `runtime_afk`: 是否运行时 AFK
- `skills_dirs`: 技能目录

**返回**: Runtime 实例

---

## KimiSoul 类

### 位置

`src/kimi_cli/soul/kimisoul.py`

### 核心方法

#### `run()`

**签名**:
```python
async def run(
    self,
    user_input: str | list[ContentPart],
    ui_loop_fn: Callable[[Wire], Awaitable[None]],
    cancel_event: asyncio.Event,
    *,
    runtime: Runtime,
) -> None
```

**参数**:
- `user_input`: 用户输入
- `ui_loop_fn`: UI 循环函数
- `cancel_event`: 取消事件
- `runtime`: 运行时

#### `process_slash_command()`

**签名**:
```python
async def process_slash_command(
    self,
    command: str,
    runtime: Runtime,
) -> WireMessage | None
```

**参数**:
- `command`: 斜杠命令
- `runtime`: 运行时

**返回**: Wire 消息或 None

---

## Wire 模块

### 位置

`src/kimi_cli/wire/`

### 核心类

#### WireMessage

**类型**: 联合类型，包含以下消息类型：
- `TextMessage`
- `ToolCall`
- `ToolResult`
- `ApprovalRequest`
- `ApprovalResponse`
- `SessionStart`
- `SessionEnd`
- `ThinkingData`
- `PlanData`

#### Wire

**核心方法**:
- `ui_side()`: 获取 UI 侧接口
- `soul_side()`: 获取核心侧接口
- `send()`: 发送消息
- `receive()`: 接收消息

---

## ACP 模块

### 位置

`src/kimi_cli/acp/`

### 核心函数

#### `acp_main()`

**签名**:
```python
def acp_main() -> None
```

启动 ACP 服务器。

---

## 工具开发 API

### 工具接口

```python
from kosong.tooling import Tool

class MyTool(Tool):
    name: str = "my_tool"
    description: str = "描述"
    
    async def __call__(self, arg1: str, arg2: int) -> dict[str, Any]:
        # 工具逻辑
        return {"result": "success"}
```

### 工具注册

工具可以通过以下方式注册：
1. 在 Agent 规格文件中引用
2. 通过 MCP 服务器加载
3. 通过插件系统添加
