# Personal Hermes Fork

> `SUertee/personal-hermes` — fork 自 [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)

## 为什么 Fork

主要使用 MiniMax 作为 LLM provider，upstream 在 MiniMax 集成上存在一些摩擦点（双 API Key 设计、图片格式不兼容、MCP 无法自动传递环境变量等）。Fork 可以快速迭代个人改进，同时保持与 upstream 同步的能力。

## 与 Upstream 的差异

| 类别 | 改动 | 关键文件 |
|---|---|---|
| MiniMax API Key 统一 | `MINIMAX_CN_API_KEY` 合并为 `MINIMAX_API_KEY` | `auth.py`, `config.py`, `doctor.py`, `status.py`, `model_switch.py` |
| 模型升级 | 默认辅助模型 → `MiniMax-M2.7-highspeed`，新增视觉模型 | `agent/auxiliary_client.py` |
| 图片格式修复 | 移除 MiniMax 的 Anthropic 兼容标记，解决图片静默丢弃 | `agent/auxiliary_client.py` |
| Think-block 正则增强 | 覆盖 MiniMax 特有的 think 标签变体 | `agent/auxiliary_client.py` |
| MCP Env 回填 | null/空值声明自动从 `os.environ` 填充 | `tools/mcp_tool.py` |
| MCP 命令解析 | `uv`/`uvx` 加入回退搜索路径 | `tools/mcp_tool.py` |
| 超时调整 | LLM API 超时 120s → 3600s | `hermes_cli/config.py` |
| MCP 服务器示例 | 新增 `docs/fork/samples/mcporter.example.json`（Tavily + MiniMax MCP，使用 env 回填） | `docs/fork/samples/mcporter.example.json` |

## 分支策略

```
upstream/main  ←  NousResearch 官方主线
      ↓ (fetch & merge)
personal-main  ←  个人主分支，包含所有自定义改动
      ↑
feat/xxx       ←  功能分支，PR 合并到 personal-main
```

## 从 Upstream 同步

```bash
git fetch upstream
git checkout personal-main
git merge upstream/main
# 手动解决冲突（高冲突区域：auxiliary_client.py, config.py）
```

## 已知问题

- `mcporter` 示例配置曾明文提交到 git 历史，已改为 env 回填模式，**需轮换泄露的 Key**
- LLM 超时 3600s 过于宽松，挂起的 API 调用会阻塞很久
- Think-block 通用正则可能误匹配合法 XML 内容

## docs/fork/ 目录结构

```
docs/fork/
├── README.md          ← 本文件
├── CHANGELOG.md       ← 改动记录
├── plans/             ← 功能计划、上游贡献计划
├── specs/             ← 技术规格文档
└── notes/             ← 调研笔记、设计探讨
```
