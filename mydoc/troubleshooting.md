# 故障排除

## 常见问题

### 1. 无法启动 Kimi Code CLI

**症状**: 运行 `kimi` 命令时出现错误。

**可能原因**:
- Python 版本不兼容
- 依赖缺失
- 配置文件损坏

**解决方案**:

```bash
# 检查 Python 版本
python --version  # 要求 3.12+

# 重新安装依赖
cd kimi-cli
make prepare

# 检查配置文件
ls ~/.kimi/config.toml

# 如果配置文件损坏，删除它
rm ~/.kimi/config.toml
```

### 2. 无法登录

**症状**: 运行 `/login` 或 `kimi login` 时失败。

**可能原因**:
- 网络连接问题
- OAuth 配置错误
- 账号问题

**解决方案**:

```bash
# 检查网络连接
ping api.moonshot.cn

# 检查 OAuth 日志
kimi login --json

# 查看日志
cat ~/.kimi/logs/kimi.log | grep -i "oauth\|login"
```

### 3. 模型无法加载

**症状**: 启动时显示 "Model not set" 或模型调用失败。

**可能原因**:
- 模型配置错误
- API 密钥无效
- 网络问题

**解决方案**:

```bash
# 检查配置文件
cat ~/.kimi/config.toml

# 确保模型配置正确
# [models.kimi-for-coding]
# provider = "kimi"
# model = "kimi-for-coding"

# 尝试使用环境变量覆盖
KIMI_MODEL_NAME=kimi-for-coding kimi

# 检查 API 密钥
echo $KIMI_API_KEY
```

### 4. 工具调用失败

**症状**: 工具调用返回错误或无法执行。

**可能原因**:
- 文件权限问题
- 工具配置错误
- 工作目录权限不足

**解决方案**:

```bash
# 检查工作目录权限
ls -la /path/to/workdir

# 检查工具日志
cat ~/.kimi/logs/kimi.log | grep -i "tool\|error"

# 尝试使用 --yolo 模式跳过审批
kimi --yolo --prompt "ls -la"
```

### 5. ACP 服务器无法启动

**症状**: 运行 `kimi acp` 时失败。

**可能原因**:
- 端口被占用
- 依赖缺失
- 配置错误

**解决方案**:

```bash
# 检查端口占用
lsof -i :8080  # 或其他端口

# 检查日志
cat ~/.kimi/logs/kimi.log | grep -i "acp\|server"

# 尝试指定其他端口（如果支持）
kimi acp --port 8081
```

### 6. MCP 服务器无法连接

**症状**: MCP 工具无法加载或调用失败。

**可能原因**:
- MCP 服务器未运行
- 配置错误
- 网络问题

**解决方案**:

```bash
# 检查 MCP 配置
kimi mcp list

# 检查 MCP 服务器状态
curl https://mcp.example.com/mcp/health

# 检查日志
cat ~/.kimi/logs/kimi.log | grep -i "mcp"

# 重新添加 MCP 配置
kimi mcp remove problematic-server
kimi mcp add --transport http problematic-server https://mcp.example.com/mcp
```

### 7. 会话无法恢复

**症状**: 无法恢复之前的会话。

**可能原因**:
- 会话文件损坏
- 会话目录被删除
- 会话 ID 错误

**解决方案**:

```bash
# 列出可用会话
kimi --session

# 检查会话目录
ls ~/.kimi/sessions/

# 检查会话文件
cat ~/.kimi/sessions/<session-id>/context.json

# 如果会话损坏，创建新会话
kimi
```

### 8. 性能问题

**症状**: Kimi Code CLI 响应缓慢或卡顿。

**可能原因**:
- 网络延迟
- 模型响应慢
- 内存不足

**解决方案**:

```bash
# 检查网络延迟
ping api.moonshot.cn

# 检查内存使用
free -h

# 尝试使用较小的模型
kimi --model kimi-code

# 禁用思考模式
kimi --no-thinking
```

### 9. 后台任务无法启动

**症状**: 后台任务提交后没有运行。

**可能原因**:
- 任务配置错误
- 后台 worker 未启动
- 权限问题

**解决方案**:

```bash
# 检查后台任务状态
kimi --print --prompt "/background list"

# 检查日志
cat ~/.kimi/logs/kimi.log | grep -i "background\|worker"

# 检查后台任务目录
ls ~/.kimi/sessions/<session-id>/background/
```

### 10. 配置文件无法加载

**症状**: 配置文件修改后不生效。

**可能原因**:
- 配置文件格式错误
- 配置文件路径错误
- 缓存问题

**解决方案**:

```bash
# 验证 TOML 格式
python -m tomli < ~/.kimi/config.toml

# 检查配置文件路径
echo $KIMI_CONFIG_DIR

# 使用命令行指定配置文件
kimi --config-file ~/.kimi/config.toml

# 检查配置日志
cat ~/.kimi/logs/kimi.log | grep -i "config"
```

---

## 日志分析

### 日志位置

日志文件位于：`~/.kimi/logs/kimi.log`

### 日志级别

| 级别 | 描述 |
|------|------|
| TRACE | 最详细的日志，包含所有操作 |
| DEBUG | 调试信息 |
| INFO | 一般信息 |
| WARNING | 警告信息 |
| ERROR | 错误信息 |
| CRITICAL | 严重错误 |

### 启用调试日志

```bash
kimi --debug
```

### 常见日志搜索

```bash
# 搜索错误
cat ~/.kimi/logs/kimi.log | grep -i "error"

# 搜索 LLM 调用
cat ~/.kimi/logs/kimi.log | grep -i "llm\|model"

# 搜索工具调用
cat ~/.kimi/logs/kimi.log | grep -i "tool\|execute"

# 搜索网络请求
cat ~/.kimi/logs/kimi.log | grep -i "http\|request"
```

---

## 重置配置

如果遇到严重问题，可以重置配置：

```bash
# 备份当前配置
cp ~/.kimi/config.toml ~/.kimi/config.toml.backup

# 删除配置文件
rm ~/.kimi/config.toml

# 重启 Kimi Code CLI
kimi
```

---

## 卸载与重装

```bash
# 卸载
pip uninstall kimi-cli

# 清理残留文件
rm -rf ~/.kimi

# 重新安装
pip install kimi-cli
```

---

## 报告问题

如果问题仍未解决，请提交 issue 到 GitHub 仓库：

1. 收集日志：`cat ~/.kimi/logs/kimi.log`
2. 记录复现步骤
3. 描述预期行为和实际行为
4. 提交到 [GitHub Issues](https://github.com/MoonshotAI/kimi-cli/issues)
