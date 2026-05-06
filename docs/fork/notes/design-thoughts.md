# 设计思考与开放问题

## Fork 策略

当前所有改动在一个 commit 里，方便 rebase 但难以为上游 PR 做 cherry-pick。如果后续改动增多，建议按功能拆分到独立分支（`feat/mcp-env-backfill`、`feat/minimax-model-upgrade` 等），各自 PR 合并到 `personal-main`。

当前的 PR-merge 结构（功能分支 → personal-main）是合理的起点。

## 超时 120s → 3600s

这是个粗暴的改法。3600s 意味着一个挂起的 API 调用会阻塞一小时。

更好的方案：按任务类型设置超时。视觉任务给更长的超时，纯文本任务保持 120s。`auxiliary_client.py` 中的 `_get_task_timeout` 已经为辅助任务做了类似的事——主超时可以参考这个模式。

不过实际使用中，MiniMax 的某些模型（特别是 highspeed 在处理长上下文时）确实需要较长超时。也许折中方案是 300s-600s。

## mcporter.json 的教训

这个文件恰好展示了 env 回填功能的必要性：

1. 一开始为了方便直接把 key 写在 JSON 里
2. 不小心 commit 进了 git
3. 现在 key 在历史里，即使改了也需要轮换

正确流程应该是从一开始就用空值 + env 回填：
```json
{"MINIMAX_API_KEY": "", "MINIMAX_API_HOST": ""}
```

这也是为什么 env 回填功能值得贡献到 upstream——它从根本上消除了在配置文件中硬编码密钥的动机。

## `_ANTHROPIC_COMPAT_PROVIDERS` 的处理

把它清空为 `frozenset()` 而不是删除整个机制，是正确的选择。保留了基础设施，将来如果有 provider 确实需要 Anthropic 格式，可以直接加回去。

但应该加一个注释说明：什么情况下需要把一个 provider 加回这个集合。目前的注释已经够用。

## 上游同步的冲突热点

按冲突风险从高到低：

1. **`auxiliary_client.py`** — 模型列表频繁变动，上游经常新增 provider
2. **`config.py`** — 新 provider 的配置项会定期添加
3. **`mcp_tool.py`** — 改动较隔离，冲突概率低

建议：同步时优先 rebase（历史更干净），但准备好手动解决 auxiliary_client 的冲突。

## 开放问题

- Think-block 通用正则是否应该有开关？可以加个 config 项控制 strip 行为
- 是否需要把 `config/` 整个目录加入 `.gitignore`？还是只 gitignore 特定文件？
- MiniMax 视觉模型的实际效果如何？需要更多测试数据来评估 `MiniMax-M2.7-highspeed` 的图片理解质量
