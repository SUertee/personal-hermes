# MCP Env 回填规格

## 问题

MCP stdio 服务器需要凭据（API Key 等），但 `_build_safe_env` 只传递安全基线变量（`PATH`、`HOME` 等）和用户显式指定的值。用户不得不在配置 YAML/JSON 中硬编码密钥，这是安全反模式（本 fork 的 `mcporter` 示例配置就曾发生过密钥泄露）。

## 方案

当声明的 `env` key 值为 `null` 或 `""` 时，视为"选择性继承父进程环境变量"。这与 Docker Compose 的 `environment:` null 值语义一致。

## 行为矩阵

| 配置值 | `os.environ` 有该 key？ | 结果 |
|---|---|---|
| `"explicit-value"` | 任意 | 使用显式值 |
| `null` 或 `""` | 有 | 从 `os.environ` 填充 |
| `null` 或 `""` | 无 | 跳过 + 记录 warning |
| key 未声明 | 任意 | 不传递（安全默认） |

## 安全模型

- 每个 key 显式 opt-in，不会发生隐式凭据泄露
- 未设置的变量会记录 warning（包含 server name），帮助调试配置错误
- 与现有 `_SAFE_ENV_KEYS` 白名单机制互补，不替代

## 实现位置

`tools/mcp_tool.py:_build_safe_env`（L194-L237）

```python
def _build_safe_env(user_env: Optional[dict], *, server_name: Optional[str] = None) -> dict:
```

调用方：`MCPServerTask._start_stdio`（L923）传入 `server_name=self.name`。

## 测试覆盖

`tests/tools/test_mcp_tool.py::TestBuildSafeEnv`:
- `test_declared_empty_env_values_are_filled_from_os_environ` — 空值从环境填充
- `test_user_env_overrides_os_environ_fill` — 显式值优先于环境
- `test_missing_declared_env_logs_warning` — 环境中不存在时记录 warning

## 配置示例

```yaml
mcp_servers:
  minimax:
    command: "uvx"
    args: ["minimax-coding-plan-mcp", "-y"]
    env:
      MINIMAX_API_KEY:       # 从 ~/.hermes/.env 或 shell 环境继承
      MINIMAX_API_HOST:      # 同上
```

```json
{
  "mcpServers": {
    "minimax": {
      "command": "uvx",
      "args": ["minimax-coding-plan-mcp", "-y"],
      "env": {
        "MINIMAX_API_KEY": "",
        "MINIMAX_API_HOST": ""
      }
    }
  }
}
```
