# MiniMax 图片格式问题调查

## 症状

通过 auxiliary client 向 MiniMax-M2.7-highspeed 发送图片时，模型表现得好像没收到图片。返回约 47 tokens，内容大意是"我没有看到任何图片"。

## 根因

MiniMax 被列在 `_ANTHROPIC_COMPAT_PROVIDERS` 中（`agent/auxiliary_client.py`）。这导致 `_is_anthropic_compat_endpoint()` 返回 `True`，触发 Anthropic 格式的图片 block：

```python
# Anthropic 格式（错误）
{"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": "..."}}
```

而 highspeed 模型在 `/v1` 端点期望的是 OpenAI data URL 格式：

```python
# OpenAI 格式（正确）
{"type": "image_url", "image_url": {"url": "data:image/png;base64,..."}}
```

Anthropic 格式的 block 被 MiniMax API 静默忽略——没有报错，就是不处理图片。

## 历史原因

旧版 MiniMax-M2.7（非 highspeed）使用 `/anthropic` 端点路径，确实需要 Anthropic 格式。Highspeed 模型改用标准 `/v1` 路径和 OpenAI 兼容格式。

## 修复

从 `_ANTHROPIC_COMPAT_PROVIDERS` 移除 MiniMax，使集合变为空 `frozenset()`：

```python
# 修复前
_ANTHROPIC_COMPAT_PROVIDERS = frozenset({"minimax", "minimax-cn"})

# 修复后
_ANTHROPIC_COMPAT_PROVIDERS = frozenset()
```

保留了 `frozenset()` 和整个机制，以备未来有其他 provider 需要 Anthropic 格式。

## 风险

如果未来 MiniMax 新模型回退到 Anthropic 格式，这会 break。更健壮的方案是按模型（而非按 provider）做格式检测。

## 如何发现类似问题

特征信号：发送了图片但模型返回 ~47 tokens 说"我看不到图片"。检查 `_prepare_vision_content()` 中的图片格式转换路径。
