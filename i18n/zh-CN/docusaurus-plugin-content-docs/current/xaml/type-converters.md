---
id: type-converters
title: 类型转换器
description: 了解 XAML 类型转换器如何把字符串属性值转换为 .NET 类型，包括内置转换器与自定义转换器的创建方式。
doc-type: concept
---

类型转换器允许把 XAML 属性值（它们始终是字符串）转换为合适的 .NET 类型。当你在 XAML 中写下 `Background="Red"` 时，类型转换器会把字符串 `"Red"` 转换为一个 [`SolidColorBrush`](/api/avalonia/media/solidcolorbrush) 对象。

## 类型转换器如何工作

当 XAML 引擎遇到一个属性特性时，它会：

1. 检查属性类型是否直接就是 `string`。如果是，则原样使用该值。
2. 查找与该属性类型关联的 `TypeConverter`。
3. 使用转换器把字符串转换为目标类型。

这个过程是自动且透明的。你不需要手动指定要使用哪个转换器。

## 内置类型转换器

Avalonia 为许多常见类型提供了类型转换器。下面是最常用的一些：

### 颜色与画刷

| 字符串值 | 转换结果 | 示例 |
|---|---|---|
| `"Red"`、`"Blue"`、`"Green"` | `Color` / `SolidColorBrush` | 颜色名称 |
| `"#FF0000"` | `Color` / `SolidColorBrush` | 十六进制 RGB |
| `"#80FF0000"` | `Color` / `SolidColorBrush` | 十六进制 ARGB |
| `"#F00"` | `Color` / `SolidColorBrush` | 简写十六进制 RGB |

```xml
<Border Background="LightBlue" BorderBrush="#333333" />
```

### Thickness（Margin、Padding、BorderThickness）

| 字符串值 | 结果 |
|---|---|
| `"8"` | 统一值：四边都为 8 |
| `"8,4"` | 左右为 8，上下为 4 |
| `"4,2,4,2"` | 左、上、右、下 |

```xml
<Border Margin="8" Padding="12,6" BorderThickness="1,0,1,0" />
```

### CornerRadius

| 字符串值 | 结果 |
|---|---|
| `"4"` | 统一圆角半径 |
| `"4,4,0,0"` | 左上、右上、右下、左下 |

```xml
<Border CornerRadius="8" />
```

### GridLength（列定义/行定义）

| 字符串值 | 结果 |
|---|---|
| `"Auto"` | 根据内容自动调整大小 |
| `"*"` | 按比例占用剩余空间 |
| `"2*"` | 占用双倍比例空间 |
| `"200"` | 固定大小，单位为设备无关像素 |

```xml
<Grid ColumnDefinitions="200,Auto,*,2*" />
```

### Point

```xml
<Line StartPoint="0,0" EndPoint="100,50" />
```

### Size

```xml
<Viewbox MaxWidth="200" MaxHeight="150" />
```

### Uri / Bitmap

```xml
<Image Source="/Assets/logo.png" />
<Image Source="avares://MyApp/Assets/logo.png" />
```

### 枚举值

枚举属性会根据它们的字符串名称自动完成转换：

```xml
<StackPanel Orientation="Horizontal" />
<TextBlock TextAlignment="Center" FontWeight="Bold" />
<DockPanel LastChildFill="True" />
```

### Geometry（路径数据）

`Geometry` 类型转换器会解析 SVG 风格的路径数据：

```xml
<Path Data="M 0,0 L 100,0 L 100,100 Z" Fill="Blue" />
```

关于路径数据语法的更多信息，请参阅 [绘制图形](/docs/graphics-animation/drawing-graphics) 中的几何图形参考。

### KeyGesture

```xml
<KeyBinding Gesture="Ctrl+S" Command="{Binding SaveCommand}" />
```

### TimeSpan

```xml
<Animation Duration="0:0:0.5" />
```

格式为：`hours:minutes:seconds.milliseconds`

## 创建自定义类型转换器

如果要为你自己的类型创建类型转换器，请实现 `TypeConverter` 并通过 `[TypeConverter]` 特性应用它：

```csharp
[TypeConverter(typeof(TemperatureConverter))]
public struct Temperature
{
    public double Value { get; }
    public string Unit { get; }

    public Temperature(double value, string unit)
    {
        Value = value;
        Unit = unit;
    }
}

public class TemperatureConverter : TypeConverter
{
    public override bool CanConvertFrom(ITypeDescriptorContext? context, Type sourceType)
    {
        return sourceType == typeof(string) || base.CanConvertFrom(context, sourceType);
    }

    public override object? ConvertFrom(
        ITypeDescriptorContext? context, CultureInfo? culture, object value)
    {
        if (value is string text)
        {
            // 解析 "72F" 或 "22C"
            var numericPart = text.TrimEnd('C', 'F', 'K');
            var unit = text[^1..];
            return new Temperature(double.Parse(numericPart, culture), unit);
        }

        return base.ConvertFrom(context, culture, value);
    }
}
```

现在你就可以在 XAML 中使用该类型：

```xml
<local:Thermostat CurrentTemperature="72F" />
```

## 另请参阅

- [XAML 参考](/docs/xaml)：XAML 语法概览。
- [数据绑定转换器](/docs/data-binding/how-to-create-a-custom-data-binding-converter)：用于数据绑定的值转换器（不同于类型转换器）。
- [内置数据绑定转换器](/docs/data-binding/built-in-data-binding-converters)：可用于绑定转换的内置转换器。
