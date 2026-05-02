# 开发指南

## 开发环境搭建

### 前置要求

- Python 3.12+（推荐 3.14）
- Git
- uv（推荐的 Python 包管理器）

### 克隆仓库

```bash
git clone https://github.com/MoonshotAI/kimi-cli.git
cd kimi-cli
```

### 安装依赖

```bash
# 使用 make（推荐）
make prepare

# 或手动安装
uv sync
```

### 安装 git hooks

```bash
uv run pre-commit install
```

---

## 开发工作流

### 运行开发版本

```bash
# 运行 Kimi Code CLI（开发模式）
uv run kimi

# 启用调试模式
uv run kimi --debug
```

### 代码格式化

```bash
make format
```

### 代码检查

```bash
# 运行所有检查
make check

# 单独运行 lint
uv run ruff check

# 单独运行类型检查
uv run pyright
```

### 运行测试

```bash
# 运行所有测试
make test

# 运行特定测试
uv run pytest tests/test_cli.py -v

# 运行 AI 测试
make ai-test
```

### 构建

```bash
# 构建 Python 包
make build

# 构建独立二进制
make build-bin

# 构建 Web UI
make build-web
```

---

## 代码风格规范

### Python 规范

- 使用 Python 3.12+ 特性
- 行长度限制：100 字符
- 使用 Ruff 进行 lint 和格式化
- 使用 Pyright 进行类型检查

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 模块 | 小写，用下划线分隔 | `agent_spec.py` |
| 类 | PascalCase | `KimiSoul` |
| 函数/方法 | snake_case | `load_config()` |
| 变量 | snake_case | `work_dir` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRIES` |

### Git 提交信息

使用 Conventional Commits 格式：

```
<type>(<scope>): <subject>
```

**允许的类型**:
- `feat`: 新功能
- `fix`: 修复 bug
- `test`: 添加/修改测试
- `refactor`: 重构代码
- `chore`: 日常维护
- `style`: 代码风格
- `docs`: 文档
- `perf`: 性能优化
- `build`: 构建相关
- `ci`: CI/CD 相关
- `revert`: 回退

**示例**:
```
feat(cli): add --quiet option
fix(soul): handle empty context gracefully
docs(guide): update installation instructions
```

---

## 项目结构

### 核心目录

```
kimi-cli/
├── src/kimi_cli/          # 主源码目录
│   ├── cli/              # CLI 入口和命令定义
│   ├── soul/             # 核心运行时和代理循环
│   ├── ui/               # 用户界面实现
│   ├── wire/             # 消息传输层
│   ├── tools/            # 内置工具
│   ├── agents/           # Agent 规格配置
│   ├── acp/              # ACP 服务器组件
│   ├── config.py         # 配置管理
│   ├── llm.py            # LLM 集成
│   ├── app.py            # 应用入口
│   └── session.py        # 会话管理
├── tests/                # 单元测试
├── tests_ai/             # AI 测试
├── packages/             # 依赖包
│   ├── kosong/           # LLM 抽象层
│   └── kaos/             # OS 交互层
├── docs/                 # 文档
└── examples/             # 示例代码
```

---

## 测试指南

### 测试类型

1. **单元测试**: `tests/test_*.py`
2. **AI 测试**: `tests_ai/test_*.py`

### 运行测试

```bash
# 运行所有单元测试
make test

# 运行特定测试文件
uv run pytest tests/test_cli.py -v

# 运行特定测试函数
uv run pytest tests/test_cli.py::test_login -v

# 运行 AI 测试
make ai-test
```

### 编写测试

测试文件应放在 `tests/` 目录下，命名为 `test_*.py`。

**示例**:
```python
import pytest
from kimi_cli.config import load_config

def test_load_config():
    config = load_config()
    assert config is not None
    assert hasattr(config, 'default_model')
```

---

## 调试技巧

### 启用调试日志

```bash
kimi --debug
```

调试日志会写入 `~/.kimi/logs/kimi.log`。

### 使用 print 模式调试

```bash
kimi --print --debug --prompt "分析代码"
```

### 设置断点

在代码中添加断点：

```python
import pdb; pdb.set_trace()
```

---

## 贡献指南

### 提交 PR 流程

1. Fork 仓库
2. 创建特性分支
3. 提交代码
4. 运行测试
5. 创建 PR

### PR 要求

- 代码必须通过所有检查（lint、类型、测试）
- 提交信息使用 Conventional Commits 格式
- 添加适当的测试覆盖
- 更新相关文档

---

## 发布流程

1. 确保 `main` 分支是最新的
2. 创建发布分支（如 `bump-0.68`）
3. 更新 `CHANGELOG.md`
4. 更新 `pyproject.toml` 版本
5. 运行 `uv sync`
6. 提交并创建 PR
7. 合并 PR 后，切换到 `main` 并拉取最新代码
8. 打标签：`git tag 0.68`
9. 推送标签：`git push --tags`

GitHub Actions 会自动处理发布流程。
