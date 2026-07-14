---
id: slider
title: Slider
description: 一个允许用户在最小值和最大值之间沿轨道拖动滑块来选择数值的控件。
doc-type: reference
---

import SliderMaxValueScreenshot from '/img/controls/slider/slider-max-value.gif';

`Slider` 控件通过滑块在轨道上的相对位置来表示一个数值。该位置相对于你配置的 `Maximum` 和 `Minimum` 值。

你可以通过拖动滑块、点击轨道、使用方向键或滚动鼠标滚轮来更改该值。滑块适用于需要让用户从连续范围或离散步进范围中选择数值的场景，例如音量、亮度或缩放级别。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Minimum` | `double` | 设置范围下界。默认值为 `0`。 |
| `Maximum` | `double` | 设置范围上界。默认值为 `100`。 |
| `Value` | `double` | 获取或设置当前滑块值。 |
| `SmallChange` | `double` | 每次按方向键时的值变化量。默认值为 `1`。 |
| `LargeChange` | `double` | 每次点击轨道或按下 Page 键时的值变化量。默认值为 `10`。 |
| [`Orientation`](/api/avalonia/layout/orientation) | `Orientation` | `Horizontal`（默认）或 `Vertical`。 |

## 示例

在这个示例中，滑块的值会通过绑定显示在它上方的文本块中。

:::info
如需回顾如何将一个控件绑定到另一个控件，请参阅指南 [绑定到控件](/docs/data-binding/binding-to-controls)。
:::

这里的最大值和最小值使用默认设置（分别为 0 和 100）。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
            Margin="20">
  <TextBlock Text="{Binding #slider.Value}"
              HorizontalAlignment="Center"/>
  <Slider x:Name="slider" />
</StackPanel>
```

</XamlPreview>

## 刻度与吸附

使用 `TickFrequency` 和 `IsSnapToTickEnabled` 可以将滑块限制为离散步进值。这在你希望用户从均匀分布的数值中选择，而不是范围内任意位置时非常有用。

```xml
<Slider Minimum="0" Maximum="100"
        TickFrequency="10"
        IsSnapToTickEnabled="True"
        TickPlacement="BottomRight" />
```

[`TickPlacement`](/api/avalonia/controls/tickplacement) 属性用于控制刻度相对于轨道的显示位置：

| 值 | 说明 |
|---|---|
| `None` | 不显示刻度（默认）。 |
| `TopLeft` | 水平滑块时刻度显示在上方，垂直滑块时显示在左侧。 |
| `BottomRight` | 水平滑块时刻度显示在下方，垂直滑块时显示在右侧。 |
| `Outside` | 刻度显示在轨道两侧。 |

## 垂直滑块

将 `Orientation` 属性设为 `Vertical` 可让滑块垂直显示。使用垂直滑块时，请确保为其指定明确的 `Height`，以便拥有足够的显示空间。

```xml
<Slider Orientation="Vertical" Height="200"
        Minimum="0" Maximum="100" Value="30" />
```

## 反转方向

如果你希望最大值显示在左侧（垂直滑块则为底部）而不是右侧（或顶部），请将 `IsDirectionReversed` 设置为 `True`。

```xml
<Slider Minimum="0" Maximum="100"
        IsDirectionReversed="True" />
```

## 绑定到视图模型

你可以将 `Value`、`Minimum` 和 `Maximum` 绑定到视图模型中的属性。以下示例使用了 MVVM Community Toolkit 的源生成器：

```xml
<Slider Maximum="{Binding MaxDamage}" Value="{Binding Damage}" />
```

```csharp
[ObservableProperty]
private double _damage;

[ObservableProperty]
private double _maxDamage = 9999;
```

<Image light={SliderMaxValueScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 全部属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Minimum` | `double` | 范围下界。默认值为 `0`。 |
| `Maximum` | `double` | 范围上界。默认值为 `100`。 |
| `Value` | `double` | 当前滑块值。 |
| `SmallChange` | `double` | 每次按方向键时的值变化量。默认值为 `1`。 |
| `LargeChange` | `double` | 每次点击轨道或按下 Page 键时的值变化量。默认值为 `10`。 |
| `TickFrequency` | `double` | 刻度之间的间隔。 |
| `IsSnapToTickEnabled` | `bool` | 当为 `true` 时，滑块会吸附到最近的刻度。默认值为 `false`。 |
| `TickPlacement` | `TickPlacement` | 刻度显示位置：`None`、`TopLeft`、`BottomRight` 或 `Outside`。 |
| `Orientation` | `Orientation` | `Horizontal`（默认）或 `Vertical`。 |
| `IsDirectionReversed` | `bool` | 当为 `true` 时，反转数值递增方向。默认值为 `false`。 |

## 另请参阅

- [NumericUpDown](/controls/input/selectors/numericupdown)
- [ToggleSwitch](/controls/input/selectors/toggleswitch)
- [绑定到控件](/docs/data-binding/binding-to-controls)
- [Slider API 参考](/api/avalonia/controls/slider)
- [`Slider.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Slider.cs)
