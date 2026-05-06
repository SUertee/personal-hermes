# 上游贡献计划

哪些个人改动适合贡献回 `NousResearch/hermes-agent`。

## 可直接贡献

这些改动是通用改进，与个人偏好无关：

| 改动 | 理由 | 优先级 |
|---|---|---|
| MCP env 回填（`_build_safe_env` null/空值解析） | 通用可用性提升，避免在配置中硬编码密钥，类似 Docker Compose 的 env 语义 | 高 |
| `uv`/`uvx` 命令解析回退 | 改动小、广泛有用、无争议 | 高 |
| Think-block 正则扩展 | 对任何使用非标准 think 标签的 provider 都有益 | 中 |

## 需要讨论

| 改动 | 疑虑 |
|---|---|
| MiniMax API Key 合并 | 是 API 设计选择，需确认是否有用户确实在用不同的 CN/Global Key；可能需要加废弃路径 |
| MiniMax 视觉模型添加 | 改动直接，但 upstream 可能需要验证模型可用性 |

## 仅限 Fork

| 改动 | 原因 |
|---|---|
| 超时 120s → 3600s | 对通用场景过于激进，是个人偏好 |
| `docs/fork/samples/mcporter.example.json` | 个人 mcporter 配置示例 |

## Action Items

- [ ] 为「可直接贡献」组提交 Issue/PR
- [ ] 在 upstream 社区探讨 MiniMax Key 合并方案
- [ ] 把 env 回填的 PR 与 `fork/specs/mcp-env-backfill.md` 中的规格对照
