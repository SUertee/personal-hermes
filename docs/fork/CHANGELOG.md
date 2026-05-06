# Changelog

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)。

## [Unreleased]

### 修复 (Fixed)
- `mcporter` 示例配置明文 API Key 改为空值 env 回填模式

## [2026-04-20] - MiniMax 整合与 MCP 环境回填

### 新增 (Added)
- MCP env 回填：`_build_safe_env` 支持 null/空值从 `os.environ` 自动填充
- MCP 命令解析：`uv`/`uvx` 加入回退搜索路径
- MiniMax 视觉模型支持（`_PROVIDER_VISION_MODELS` 中新增 `MiniMax-M2.7-highspeed`）
- 新增 `docs/fork/samples/mcporter.example.json` MCP 服务器示例（Tavily + MiniMax）
- env 回填测试（3 个用例）和 uvx 命令解析测试

### 变更 (Changed)
- MiniMax API Key 统一：`MINIMAX_CN_API_KEY` 合并为 `MINIMAX_API_KEY`，中国和国际端点共用
- 默认辅助模型：`MiniMax-M2.7` → `MiniMax-M2.7-highspeed`
- Think-block 正则增强，覆盖 MiniMax 特有标签（`<think>,</think>`、`<unknown_think>` 等）
- LLM API 超时：120s → 3600s
- 文档全面更新：quickstart、providers、env vars、MCP 配置参考、fallback providers

### 修复 (Fixed)
- MiniMax 图片格式：从 `_ANTHROPIC_COMPAT_PROVIDERS` 中移除 MiniMax，解决 highspeed 模型静默丢弃图片的问题（模型返回 ~47 tokens "no image attached"）
