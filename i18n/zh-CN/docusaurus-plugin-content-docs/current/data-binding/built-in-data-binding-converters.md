---
id: built-in-data-binding-converters
title: 内置数据绑定转换器
description: Avalonia 中常见数据绑定转换场景所用的内置值转换器参考。
doc-type: reference
---

_Avalonia UI_ 内置了一些适用于常见场景的数据绑定转换器：

| 转换器 | 说明 |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Negation Operator | 可以在数据绑定路径前加上 `!` 运算符，用于返回布尔值的取反结果。另请参阅下方说明。 |
| `StringConverters.IsNullOrEmpty` | 当输入字符串为 null 或空字符串时返回 `true` |
| `StringConverters.IsNotNullOrEmpty` | 当输入字符串为 null 或空字符串时返回 `false` |
| `ObjectConverters.IsNull` | 当输入对象为 null 时返回 `true` |
| `ObjectConverters.IsNotNull` | 当输入对象为 null 时返回 `false` |
| `BoolConverters.And` | 一个多值转换器，当所有输入都为 true 时返回 `true` |
| `BoolConverters.Or` | 一个多值转换器，只要任意输入为 true 就返回 `true` |

## 取反运算符示例

下面的示例中，当绑定值为 false 时显示 `TextBlock`：

```xml
<StackPanel>
  <TextBox Name="input" IsEnabled="{Binding AllowInput}"/>
  <TextBlock IsVisible="{Binding !AllowInput}">Input is not allowed</TextBlock>
</StackPanel>
```

当你绑定的值不是布尔类型时，取反同样可用。这是因为绑定值会先通过 `Convert.ToBoolean` 转换为布尔值，然后再进行取反。

例如，整数 0 会被 `Convert.ToBoolean` 转换为 false，而其他整数值则会转换为 true。因此你可以使用取反运算符，在集合为空时显示一条消息：

```xml
<Panel>
  <ListBox ItemsSource="{Binding Items}"/>
  <TextBlock IsVisible="{Binding !Items.Count}">No results found</TextBlock>
</Panel>
```

你也可以连续使用两次取反运算符。例如，当你希望先把整数转换为布尔值，再对其结果进行处理时，就可以这样写。

例如，当集合为空（count 为 0）时，你可以用这种方式隐藏某个控件：

```xml
<Panel>
  <ListBox ItemsSource="{Binding Items}" IsVisible="{Binding !!Items.Count}"/>
</Panel>
```

## 其他转换示例

下面这个绑定示例会在绑定文本为 null 或空字符串时隐藏文本块：

```xml
<TextBlock Text="{Binding MyText}"
           IsVisible="{Binding MyText, 
                       Converter={x:Static StringConverters.IsNotNullOrEmpty}}"/>
```

下面这个示例会在绑定对象为 null 或空时隐藏内容控件：

```xml
<ContentControl Content="{Binding MyContent}"
                IsVisible="{Binding MyContent, 
                            Converter={x:Static ObjectConverters.IsNotNull}}"/>
```

下面这个示例演示了如何绑定多个参数。只有当绑定对象的 `MyText` 属性不为 null 或空，并且 `IsMyNotEmptyTextVisible` 属性为 `true` 时，文本块才会显示：

```xml
<TextBlock Text="{Binding MyText, StringFormat='My text: {0}'}">
  <TextBlock.IsVisible>
    <MultiBinding Converter="{x:Static BoolConverters.And}">
        <Binding Path="MyText" Converter="{x:Static StringConverters.IsNotNullOrEmpty}"/>
        <Binding Path="IsMyNotEmptyTextVisible"/>
    </MultiBinding>
  </TextBlock.IsVisible>
</TextBlock>
```

## 更多信息


:::info
你可以参考这个 [Avalonia UI 值转换器示例](https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/MVVM/ValueConversionSample)。
:::

## 另请参阅

- [How to Create a Custom Data Binding Converter](/docs/data-binding/how-to-create-a-custom-data-binding-converter): Writing custom value converters.
- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): Binding parameters and converter usage.
- [MultiBinding](/docs/data-binding/multi-binding): Combining multiple bound values.
