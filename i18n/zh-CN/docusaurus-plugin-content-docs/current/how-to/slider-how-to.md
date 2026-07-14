---
id: slider-how-to
title: "如何：使用 Slider"
description: "学习如何在 Avalonia UI 中配置 Slider 范围、显示数值、添加刻度、设置方向并绑定到视图模型。"
doc-type: how-to
---

本指南介绍常见的 [`Slider`](/api/avalonia/controls/slider) 使用场景，包括范围配置、数值显示、刻度、垂直方向以及双向数据绑定。

## 带数值显示的基础 Slider

你可以通过把 `TextBlock` 绑定到滑块的 `Value` 属性，在滑块旁边显示当前值。`StringFormat` 标记则可以控制数字的显示格式：

```xml
<StackPanel Spacing="8">
    <TextBlock Text="{Binding #slider.Value, StringFormat='Volume: {0:F0}%'}" />
    <Slider x:Name="slider" Minimum="0" Maximum="100" Value="50" />
</StackPanel>
```

`#slider` 语法表示通过控件的 `x:Name` 直接引用它。这种方式适合快速原型，但在正式项目中通常还是更推荐通过视图模型来绑定。

## 绑定到视图模型

为了更好地分离关注点，应把 `Value` 绑定到视图模型中的属性上。对 `Slider` 而言，这个绑定默认就是双向的，因此无论是 UI 侧还是代码侧的修改都能保持同步：

```csharp
public partial class SettingsViewModel : ObservableObject
{
    [ObservableProperty]
    private double _brightness = 75;
}
```

```xml
<Slider Minimum="0" Maximum="100" Value="{Binding Brightness}" />
```

:::tip
如果你需要在值变化时执行额外逻辑（例如保存用户偏好），可以在视图模型中增加类似 `OnBrightnessChanged` 的 partial 方法。MVVM Toolkit 的源生成器会自动为你接上这个回调。
:::

## 刻度与吸附

配合使用 `TickFrequency` 和 `IsSnapToTickEnabled`，可以把可选值限制为离散步进。这在你的业务只接受整步值时特别有用，例如音量等级、百分比步进或星级评分：

```xml
<!-- 吸附到 10 的倍数 -->
<Slider Minimum="0" Maximum="100"
        TickFrequency="10"
        IsSnapToTickEnabled="True"
        TickPlacement="BottomRight" />
```

[`TickPlacement`](/api/avalonia/controls/tickplacement) options:

| 值 | 说明 |
|---|---|
| `None` | 不显示刻度（默认）。 |
| `TopLeft` | 刻度显示在上方（水平）或左侧（垂直）。 |
| `BottomRight` | 刻度显示在下方（水平）或右侧（垂直）。 |
| `Outside` | 两侧都显示刻度。 |

:::note
如果你只设置 `TickFrequency` 而不启用 `IsSnapToTickEnabled="True"`，那么界面上虽然会显示刻度，但用户仍然可以拖到刻度之间的任意值。当你希望真正限制输入时，请务必同时设置这两个属性。
:::

## SmallChange 与 LargeChange

`SmallChange` 和 `LargeChange` 用于控制通过键盘或轨道点击操作时，每次值变化的幅度。如果默认步长对你的场景来说过大或过小，可以按需调整：

```xml
<Slider Minimum="0" Maximum="1" Value="0.5"
        SmallChange="0.01"
        LargeChange="0.1" />
```

| 属性 | 触发方式 | 默认值 |
|---|---|---|
| `SmallChange` | 方向键 | 1 |
| `LargeChange` | 点击轨道或按 Page Up / Page Down | 10 |

对于范围较小的滑块（例如 0 到 1），你通常应减小这两个步长，这样键盘用户才能更方便地访问所有有意义的位置。

## 垂直 Slider

将 [`Orientation`](/api/avalonia/layout/orientation) 属性设置为 `Vertical`，即可创建垂直滑块。同时最好显式设置一个 `Height`，避免滑块高度塌陷：

```xml
<Slider Orientation="Vertical" Height="200"
        Minimum="0" Maximum="100" Value="30" />
```

:::tip
在垂直滑块中，`TopLeft` 刻度显示在左侧，`BottomRight` 刻度显示在右侧。如果你希望最小值显示在顶部而不是底部，可以设置 `IsDirectionReversed="True"`。
:::

## 仅整数的 Slider

如果你希望滑块只允许整数值，可以将 `TickFrequency` 设为 `1` 并启用吸附功能。这样就能避免小数值进入视图模型：

```xml
<Slider Minimum="1" Maximum="10"
        TickFrequency="1"
        IsSnapToTickEnabled="True"
        Value="{Binding FontSizeChoice}" />
```

由于 `Value` 的类型是 `double`，因此视图模型中的对应属性最好也声明为 `double`。如果你的业务逻辑最终需要 `int`，可以在绑定之后再做转换（例如使用 `(int)Math.Round(value)`）。

## 带标签的 Slider

你可以使用 `Grid` 在滑块两侧显示最小值和最大值标签，从而帮助用户一眼理解整个取值范围：

```xml
<Grid ColumnDefinitions="Auto,*,Auto" VerticalAlignment="Center">
    <TextBlock Grid.Column="0" Text="0" Margin="0,0,8,0"
               VerticalAlignment="Center" />
    <Slider Grid.Column="1" Minimum="0" Maximum="100" Value="{Binding Level}" />
    <TextBlock Grid.Column="2" Text="100" Margin="8,0,0,0"
               VerticalAlignment="Center" />
</Grid>
```

如果你的 `Minimum` 和 `Maximum` 是动态设置的，也可以直接把标签文本绑定到滑块上的这两个属性。

## 颜色预览滑块

你可以组合多个滑块来实现一个 RGB 颜色选择器。每个滑块分别控制一个通道（0 到 255），并显示当前值：

```xml
<StackPanel Spacing="8">
    <StackPanel Orientation="Horizontal" Spacing="8">
        <TextBlock Text="R" Width="20" VerticalAlignment="Center" />
        <Slider Minimum="0" Maximum="255" Value="{Binding Red}" Width="200" />
        <TextBlock Text="{Binding Red, StringFormat='{}{0:F0}'}" Width="30" />
    </StackPanel>
    <StackPanel Orientation="Horizontal" Spacing="8">
        <TextBlock Text="G" Width="20" VerticalAlignment="Center" />
        <Slider Minimum="0" Maximum="255" Value="{Binding Green}" Width="200" />
        <TextBlock Text="{Binding Green, StringFormat='{}{0:F0}'}" Width="30" />
    </StackPanel>
    <StackPanel Orientation="Horizontal" Spacing="8">
        <TextBlock Text="B" Width="20" VerticalAlignment="Center" />
        <Slider Minimum="0" Maximum="255" Value="{Binding Blue}" Width="200" />
        <TextBlock Text="{Binding Blue, StringFormat='{}{0:F0}'}" Width="30" />
    </StackPanel>
</StackPanel>
```

为了获得更顺手的体验，可以给每个滑块设置 `SmallChange="1"` 和 `LargeChange="16"`，让键盘调整幅度更贴近常见的颜色编辑习惯。

## 禁用与只读状态

你可以通过两种方式阻止用户与滑块交互：

```xml
<!-- 完全禁用：显示为灰色不可用状态 -->
<Slider IsEnabled="False" Value="60" />

<!-- 视觉上保持正常，但不可交互 -->
<Slider IsHitTestVisible="False" Value="{Binding Progress}" />
```

如果你希望明确传达“该控件当前不可用”，应使用 `IsEnabled="False"`。如果你希望滑块外观保持正常，但只把它当作一个只读指示器（例如显示下载进度），则可以使用 `IsHitTestVisible="False"`。

## 设置滑块样式

### 自定义轨道与滑块颜色

你可以通过针对滑块模板内部的部件来覆盖轨道颜色。`PART_DecreaseButton` 对应的是滑块之前已经填充的那一段区域：

```xml
<Slider Value="50">
    <Slider.Styles>
        <Style Selector="Slider /template/ RepeatButton#PART_DecreaseButton">
            <Setter Property="Background" Value="#6366F1" />
        </Style>
    </Slider.Styles>
</Slider>
```

### 更宽的轨道

你可以增大轨道高度，以获得更醒目的外观，或者让触控命中区域更容易操作：

```xml
<Slider.Styles>
    <Style Selector="Slider /template/ Track">
        <Setter Property="Height" Value="8" />
    </Style>
</Slider.Styles>
```

:::note
像 `PART_DecreaseButton` 和 `PART_IncreaseButton` 这样的模板部件名称，是由默认 Fluent 主题定义的。如果你使用的是自定义控件模板，那么这些部件名称可能会不同。
:::

## 关键属性参考

| 属性 | 类型 | 说明 |
|---|---|---|
| `Minimum` | `double` | 最小值，默认是 0。 |
| `Maximum` | `double` | 最大值，默认是 100。 |
| `Value` | `double` | 当前值。 |
| `SmallChange` | `double` | 方向键调整步长，默认是 1。 |
| `LargeChange` | `double` | 点击轨道或按 Page 键时的调整步长，默认是 10。 |
| `TickFrequency` | `double` | 刻度之间的间隔。 |
| `IsSnapToTickEnabled` | `bool` | 是否吸附到最近刻度。 |
| `TickPlacement` | `TickPlacement` | 刻度显示位置。 |
| `Orientation` | `Orientation` | `Horizontal`（默认）或 `Vertical`。 |
| `IsDirectionReversed` | `bool` | 是否反转数值增长方向。 |

## 另请参阅

- [Slider](/controls/input/selectors/slider)：`Slider` 控件的完整属性与事件参考。
- [Binding to controls](/docs/data-binding/binding-to-controls)：使用 `#name` 语法把一个控件属性绑定到另一个控件。
- [Data validation](/docs/app-development/data-validation)：为滑块绑定属性添加验证规则。
- [Accessibility](/docs/app-development/accessibility)：交互式控件在键盘和屏幕阅读器场景下的可访问性注意事项。
