# Think-block 正则设计

## 问题

MiniMax 模型会输出各种非标准的 think block 标签：`<think>,</think>`（带逗号）、`<unknown_think>`、`<unknown>` 等。原始正则只匹配固定列表：

```python
r"<(?:think|thinking|reasoning|thought|REASONING_SCRATCHPAD)>.*?</...>"
```

## 当前方案

新增通用模式 + 特定命名模式的组合：

```python
r"<[a-zA-Z_][a-zA-Z0-9_]*>(?:(?!</?[a-zA-Z_][a-zA-Z0-9_]*>).)*?</[a-zA-Z_][a-zA-Z0-9_]*>"
r"|<think>.*?</think>"
r"|<thinking>.*?</thinking>"
# ... 其他特定标签
```

通用模式排在第一个 alternative，充当兜底。

## 设计隐患

通用模式 `<[a-zA-Z_][a-zA-Z0-9_]*>` 可能匹配模型输出中的合法 XML/HTML 内容。配合 `re.DOTALL | re.IGNORECASE` 更加激进。

示例：如果模型输出 `<summary>重要内容</summary>`，也会被匹配并删除。

## 替代方案

1. **白名单方式**（更保守）：只添加已知的 MiniMax 特有标签，不用通用模式。更安全但需要随新标签变体持续更新。
2. **按 provider 配置正则**：不同 provider 用不同的 strip 规则。更精确但增加复杂度。
3. **DEBUG 日志**：对被 strip 的内容记录 DEBUG 日志，便于发现误匹配。

## 建议

短期内通用模式可以工作（模型实际很少输出合法的自定义 XML 标签），但如果出现误匹配，应优先考虑回退到白名单 + 按需添加。至少应在 DEBUG 级别记录被 strip 的内容。
