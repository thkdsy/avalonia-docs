---
id: numericupdown
title: NumericUpDown 问题
description: 排除 NumericUpDown 选择器控件的问题
doc-type: troubleshooting
sidebar_label: NumericUpDown
---

## 清除 `NumericUpDown` 文本框时出现无效转换异常

当 `NumericUpDown` 的文本框输入被完全清除时，该控件可能会抛出无效转换异常，例如 `Invalid cast from string to decimal?`

要防止出现这些异常，请尝试以下方法：

- 在 `Binding` 上设置 `TargetNullValue` 和 `FallbackValue`（均设为 `0`）。确保这些值显式类型化以匹配源属性类型（通常为 `decimal` 或 `int`），以便它们被视为数字而非 `string`。这可以防止空文本框记录为绑定失败。
- （可选）设置 `UpdateSourceTrigger=LostFocus`，以阻止视图模型在编辑状态下更新。这可以减少发生次数，但无法完全避免。

```xml
<NumericUpDown Minimum="0" Maximum="10000000">
  <NumericUpDown.Value>
    <Binding Path="Units">
      <Binding.TargetNullValue><x:Decimal>0</x:Decimal></Binding.TargetNullValue>
      <Binding.FallbackValue><x:Decimal>0</x:Decimal></Binding.FallbackValue>
    </Binding>
  </NumericUpDown.Value>
</NumericUpDown>
```

## 另请参阅

- [NumericUpDown 控件](/controls/input/selectors/numericupdown)
- [数据绑定语法](/docs/data-binding/data-binding-syntax)
