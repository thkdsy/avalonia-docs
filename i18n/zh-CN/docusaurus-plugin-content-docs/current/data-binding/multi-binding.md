---
id: multi-binding
title: 多重绑定
description: 使用 MultiBinding 和 IMultiValueConverter 将多个绑定源组合为单一值。
doc-type: how-to
---

[`MultiBinding`](/api/avalonia/data/multibinding) 可以把多个源属性的值组合成一个目标属性值。当某个显示值依赖多个数据源时，它就特别有用，例如用名字和姓氏拼接出全名，或者计算一个组合值。

## 使用 StringFormat 的基础用法

`MultiBinding` 最简单的用法，就是把多个值组合成一个格式化字符串：

```xml
<TextBlock>
    <TextBlock.Text>
        <MultiBinding StringFormat="{}{0} {1}">
            <Binding Path="FirstName" />
            <Binding Path="LastName" />
        </MultiBinding>
    </TextBlock.Text>
</TextBlock>
```

`MultiBinding` 内部的每个 `Binding` 都会映射到 `StringFormat` 模式中的一个占位符（例如 `{0}`、`{1}` 等）。格式字符串遵循标准的 .NET `string.Format` 规则。

### 数字格式化

```xml
<TextBlock>
    <TextBlock.Text>
        <MultiBinding StringFormat="Total: {0:C2} ({1} items)">
            <Binding Path="TotalPrice" />
            <Binding Path="ItemCount" />
        </MultiBinding>
    </TextBlock.Text>
</TextBlock>
```

:::tip
当 `StringFormat` 以 `{0` 开头时，你必须对起始花括号进行转义。可以在模式前加上 `{}`，或者使用反斜杠转义：`StringFormat='\{0\} items'`。
:::

## 使用 IMultiValueConverter

如果你的逻辑不只是简单的字符串格式化，那么就可以实现 `IMultiValueConverter`。这个转换器会接收所有子绑定传来的值数组，并返回一个最终结果。

### 定义转换器

```csharp
using System;
using System.Collections.Generic;
using System.Globalization;
using Avalonia.Data.Converters;

public class AllTrueConverter : IMultiValueConverter
{
    public object? Convert(
        IList<object?> values,
        Type targetType,
        object? parameter,
        CultureInfo culture)
    {
        foreach (var value in values)
        {
            if (value is not true)
                return false;
        }
        return true;
    }
}
```

### 在 XAML 中使用转换器

先把转换器声明为资源，然后在 `MultiBinding` 中引用它：

```xml
<Window.Resources>
    <local:AllTrueConverter x:Key="AllTrue" />
</Window.Resources>

<Button Content="Submit"
        IsEnabled="{MultiBinding Converter={StaticResource AllTrue}}">
    <!-- 这里故意留空：MultiBinding 的子绑定必须使用属性元素语法。
         完整写法见下文。 -->
</Button>
```

由于带子绑定的 `MultiBinding` 必须使用属性元素语法，因此完整写法如下：

```xml
<Button Content="Submit">
    <Button.IsEnabled>
        <MultiBinding Converter="{StaticResource AllTrue}">
            <Binding Path="IsFormValid" />
            <Binding Path="HasAcceptedTerms" />
            <Binding Path="IsNotBusy" />
        </MultiBinding>
    </Button.IsEnabled>
</Button>
```

只有当这三个绑定属性都为 `true` 时，按钮才会被启用。

## 绑定到控件

`MultiBinding` 内部的子绑定支持与普通绑定相同的源选项，包括 `ElementName`、`RelativeSource` 以及 Avalonia 的 `#elementName` 简写语法：

```xml
<StackPanel>
    <NumericUpDown x:Name="width" Value="100" Minimum="0" Maximum="500" />
    <NumericUpDown x:Name="height" Value="50" Minimum="0" Maximum="500" />

    <TextBlock>
        <TextBlock.Text>
            <MultiBinding StringFormat="Area: {0} x {1} = {2}">
                <Binding Path="Value" ElementName="width" />
                <Binding Path="Value" ElementName="height" />
                <Binding Path="#width.Value"
                         Converter="{x:Static local:MultiplyConverter.Instance}"
                         ConverterParameter="{Binding #height.Value}" />
            </MultiBinding>
        </TextBlock.Text>
    </TextBlock>
</StackPanel>
```

## MultiBinding 属性

| 属性 | 说明 |
|---|---|
| `Bindings` | 子 `Binding` 对象的集合。 |
| `Converter` | 用于处理绑定值的 `IMultiValueConverter`。 |
| `ConverterParameter` | 传递给转换器的参数。 |
| `StringFormat` | 当未指定转换器时（或转换器返回字符串时）使用的格式化字符串。 |
| `FallbackValue` | 当多重绑定无法产生结果时使用的值。 |
| `TargetNullValue` | 当转换器返回 `null` 时使用的值。 |
| `Mode` | 绑定模式。`MultiBinding` 支持 `OneWay` 和 `OneTime`。 |
| `Priority` | 绑定优先级。 |

:::info
`MultiBinding` 默认是单向绑定。它不支持双向多重绑定，因为通常没有通用方法可以把一个多值转换结果反向拆回各个源属性。
:::

:::tip
与 WPF 不同，Avalonia 支持在一个 `MultiBinding` 中再嵌套另一个 `MultiBinding`。每个嵌套的 `MultiBinding` 都会在父转换器的输入数组中解析为一个单独值。
:::

## FuncMultiValueConverter

Avalonia 提供了 `FuncMultiValueConverter<TIn, TOut>`，适合那些你只想内联定义转换逻辑、而不想专门创建完整类的简单场景。

转换器函数接收的是一个 `IReadOnlyList<TIn>`，因此你既可以遍历这些值，也可以按索引访问它们：

```csharp
public static class Converters
{
    // 遍历所有值
    public static readonly FuncMultiValueConverter<string, string> FullName =
        new(parts => string.Join(" ", parts.Where(p => !string.IsNullOrEmpty(p))));

    // 按索引访问值
    public static readonly FuncMultiValueConverter<string, string> FormattedName =
        new(parts => $"{parts[1]}, {parts[0]}");
}
```

```xml
<TextBlock>
    <TextBlock.Text>
        <MultiBinding Converter="{x:Static local:Converters.FullName}">
            <Binding Path="FirstName" />
            <Binding Path="MiddleName" />
            <Binding Path="LastName" />
        </MultiBinding>
    </TextBlock.Text>
</TextBlock>
```

## 常见模式

### 根据多个条件控制可见性

```csharp
public class AnyTrueConverter : IMultiValueConverter
{
    public static readonly AnyTrueConverter Instance = new();

    public object? Convert(
        IList<object?> values,
        Type targetType,
        object? parameter,
        CultureInfo culture)
    {
        return values.Any(v => v is true);
    }
}
```

```xml
<Border>
    <!-- 任意条件为 true 时显示 -->
    <Border.IsVisible>
        <MultiBinding Converter="{x:Static local:AnyTrueConverter.Instance}">
            <Binding Path="HasErrors" />
            <Binding Path="HasWarnings" />
        </MultiBinding>
    </Border.IsVisible>
</Border>
```

### 从多个输入计算结果值

```csharp
public class RectangleAreaConverter : IMultiValueConverter
{
    public object? Convert(
        IList<object?> values,
        Type targetType,
        object? parameter,
        CultureInfo culture)
    {
        if (values.Count >= 2
            && values[0] is double width
            && values[1] is double height)
        {
            return width * height;
        }
        return 0.0;
    }
}
```

## See also

- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): Binding parameters including StringFormat.
- [How to Create a Custom Converter](/docs/data-binding/how-to-create-a-custom-data-binding-converter): Single-value converters.
- [Built-in Data Binding Converters](/docs/data-binding/built-in-data-binding-converters): Converters shipped with Avalonia.
