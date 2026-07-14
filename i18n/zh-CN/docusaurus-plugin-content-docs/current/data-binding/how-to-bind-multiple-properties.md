---
id: how-to-bind-multiple-properties
title: 如何绑定多个属性
description: 将多个视图模型属性绑定到控件，并通过多值转换器将它们组合起来。
doc-type: how-to
---

import MultiBindingRgbScreenshot from '/img/guides/data/multibinding-rgb.gif';

当某个目标属性依赖多个来源的值时，你可以使用 [`MultiBinding`](/api/avalonia/data/multibinding) 来聚合多个 `Binding` 对象，并通过 [`IMultiValueConverter`](/api/avalonia/data/converters/imultivalueconverter) 生成组合结果。每当任意一个已绑定属性触发变化通知时，转换器的 `Convert` 方法都会重新执行，因此目标属性会自动保持同步。

`MultiBinding` 与普通 `Binding` 一样，可以用于视图模型属性、具名控件以及其他绑定源。

:::caution
`MultiBinding` 只支持 `BindingMode.OneTime` 和 `BindingMode.OneWay`。它不支持双向多重绑定，因为通常无法把一个多值转换结果再通用地还原回各个源值。
:::

## 前置条件

开始之前，请确保你已经熟悉以下内容：

- [Data binding syntax](/docs/data-binding/data-binding-syntax) 以及 `Binding` 表达式的工作方式。
- 如何在 XAML 中通过 `x:Key` 声明资源，以便使用 `StaticResource` 引用转换器。

## 理解 `IMultiValueConverter`

`IMultiValueConverter` 与 `IValueConverter` 很相似，但它接收的是一个值列表，而不是单个值。由于聚合操作通常不可逆，因此它没有 `ConvertBack` 方法。

```csharp
public interface IMultiValueConverter
{
    object? Convert(IList<object?> values, Type targetType, object? parameter, CultureInfo culture);
}
```

你的转换器会接收到：

| 参数 | 用途 |
|---|---|
| `values` | 每个子 `Binding` 当前的值，顺序与声明顺序一致。 |
| `targetType` | 目标属性的类型（例如 `IBrush` 或 `string`）。 |
| `parameter` | 来自 `ConverterParameter` 的可选参数。 |
| `culture` | 绑定引擎传入的区域性信息。 |

:::tip
在初始化阶段，由于某些绑定尚未解析完成，`values` 中的一些项可能会是 `UnsetValueType`。在处理这些值之前，请务必先检查这一点。
:::

## 将 RGB 滑块绑定到前景画刷

下面的示例会把三个 `NumericUpDown` 控件（红、绿、蓝通道）绑定到一个 `TextBlock` 的 `Foreground` 上，从而生成实时颜色预览。

### 步骤 1：定义 XAML 布局

由于 `MultiBinding` 对子绑定要求使用属性元素语法，因此你需要显式写出每一个 `<Binding>` 元素。这里使用 `ElementName` 指向已命名的 `NumericUpDown` 控件。

```xml title="MainWindow.axaml"
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:ExampleApp">

    <Window.Resources>
        <local:RgbToBrushMultiConverter x:Key="RgbToBrushMultiConverter" />
    </Window.Resources>

    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center" Spacing="8">
        <NumericUpDown x:Name="red" Minimum="0" Maximum="255" Increment="20" Value="0" Foreground="Red" />
        <NumericUpDown x:Name="green" Minimum="0" Maximum="255" Increment="20" Value="0" Foreground="Green" />
        <NumericUpDown x:Name="blue" Minimum="0" Maximum="255" Increment="20" Value="0" Foreground="Blue" />

        <TextBlock Text="MultiBinding Text Color!" FontSize="24">
            <TextBlock.Foreground>
                <MultiBinding Converter="{StaticResource RgbToBrushMultiConverter}">
                    <Binding Path="Value" ElementName="red" />
                    <Binding Path="Value" ElementName="green" />
                    <Binding Path="Value" ElementName="blue" />
                </MultiBinding>
            </TextBlock.Foreground>
        </TextBlock>
    </StackPanel>
</Window>
```

### 步骤 2：实现转换器

请仔细检查每一个值的类型。`NumericUpDown.Value` 的类型是 `decimal?`，因此你的转换器必须处理 `decimal`、`null` 和 `UnsetValueType`。对于尚未解析完成的值，应返回 `BindingOperations.DoNothing`，这样在绑定初始化期间，目标属性就会保持原有值不变。

```csharp title="RgbToBrushMultiConverter.cs"
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Linq;
using Avalonia.Data;
using Avalonia.Data.Converters;
using Avalonia.Media;
using Avalonia.Media.Immutable;

public sealed class RgbToBrushMultiConverter : IMultiValueConverter
{
    public object? Convert(IList<object?> values, Type targetType, object? parameter, CultureInfo culture)
    {
        // 确保所有绑定都已提供，并且目标类型兼容
        if (values?.Count != 3 || !targetType.IsAssignableFrom(typeof(ImmutableSolidColorBrush)))
            throw new NotSupportedException();

        // 确保所有绑定值类型正确
        if (!values.All(x => x is decimal or UnsetValueType or null))
            throw new NotSupportedException();

        // 只要有任意绑定尚未解析完成，就返回 DoNothing
        if (values[0] is not decimal r ||
            values[1] is not decimal g ||
            values[2] is not decimal b)
            return BindingOperations.DoNothing;

        byte a = 255;
        var color = new Color(a, (byte)r, (byte)g, (byte)b);
        return new ImmutableSolidColorBrush(color);
    }
}
```

### 步骤 3：运行应用程序

拖动任意一个滑块，文本颜色都会立即更新：

<Image light={MultiBindingRgbScreenshot} alt="App showing RGB sliders bound to multiple properties producing a combined color" position="center" maxWidth={400} cornerRadius="true"/>

## 使用 `FuncMultiValueConverter` 简化实现

对于简单的转换场景，你可以不用专门创建完整类。Avalonia 提供的 `FuncMultiValueConverter<TIn, TOut>` 允许你直接用 lambda 表达式定义逻辑。你可以把转换器暴露为静态属性，以便在 XAML 中通过 `x:Static` 引用。

```csharp title="Converters.cs"
using System.Linq;
using Avalonia.Data.Converters;

public static class Converters
{
    public static readonly FuncMultiValueConverter<string, string> FullName =
        new(parts => string.Join(" ", parts.Where(p => !string.IsNullOrEmpty(p))));
}
```

```xml title="Usage in AXAML"
<TextBlock>
    <TextBlock.Text>
        <MultiBinding Converter="{x:Static local:Converters.FullName}">
            <Binding Path="FirstName" />
            <Binding Path="LastName" />
        </MultiBinding>
    </TextBlock.Text>
</TextBlock>
```

这种方式可以省去资源声明，并让简单转换器更靠近它们的使用位置。

## 提示

- **将转换器注册为资源**，当你通过 `StaticResource` 引用它们时；或者把它们暴露为 `static` 字段，再通过 `x:Static` 使用，从而彻底省掉资源查找。
- **当转换器暂时无法产生有效结果时，返回 `BindingOperations.DoNothing`** 而不是 `null`。这会告诉绑定引擎保持目标属性不变。
- **如果你要在很多地方复用同一种 `MultiBinding` 模式，可以考虑使用 `MarkupExtension`**，以简化 XAML 语法。

## 另请参阅

- [MultiBinding](/docs/data-binding/multi-binding): Full `MultiBinding` reference, including `StringFormat`, `FallbackValue`, and the properties table.
- [How to create a custom data binding converter](/docs/data-binding/how-to-create-a-custom-data-binding-converter): Single-value `IValueConverter` implementations.
- [Built-in data binding converters](/docs/data-binding/built-in-data-binding-converters): Converters shipped with Avalonia.
- [Data binding syntax](/docs/data-binding/data-binding-syntax): Binding parameters including `StringFormat` and `ConverterParameter`.
