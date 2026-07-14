---
id: positioning-controls
title: 控件定位
description: 使用 HorizontalAlignment、VerticalAlignment、Margin 和 Padding 来定位控件。
doc-type: explanation
---

import LayoutMarginsPaddingAlignmentBasicScreenshot from '/img/reference/layout/positioning/layout-margins-padding-alignment-graphic1.png';
import LayoutMarginsPaddingAlignmentBasicAnnotatedScreenshot from '/img/reference/layout/positioning/layout-margins-padding-alignment-graphic2.png';
import LayoutHorizontalAlignmentScreenshot from '/img/reference/layout/positioning/layout-horizontal-alignment-graphic.png';
import LayoutVerticalAlignmentScreenshot from '/img/reference/layout/positioning/layout-vertical-alignment-graphic.png';
import LayoutMarginsPaddingAlignmentComplexAnnotatedScreenshot from '/img/reference/layout/positioning/layout-margins-padding-alignment-graphic3.png';

Avalonia 控件公开了若干可用于精确定位子元素的属性。本文会讨论其中最重要的四个属性：[`HorizontalAlignment`](/api/avalonia/layout/horizontalalignment)、`Margin`、`Padding` 和 [`VerticalAlignment`](/api/avalonia/layout/verticalalignment)。理解这些属性的效果非常重要，因为它们构成了在 Avalonia 应用中控制元素位置的基础。

## 元素定位简介

在 Avalonia 中，有很多方式可以对元素进行定位。但要获得理想的布局，仅仅选择合适的 `Panel` 元素还不够。想要精细控制定位，还需要理解 `HorizontalAlignment`、`Margin`、`Padding` 和 `VerticalAlignment` 这些属性。

下图展示了一个使用多个定位属性的布局场景。

<Image light={LayoutMarginsPaddingAlignmentBasicScreenshot} alt="定位示例" position="center" maxWidth={400} cornerRadius="true"/>

乍看之下，图中的 [`Button`](/api/avalonia/controls/button) 元素似乎是随意摆放的。但实际上，它们的位置是通过 margin、alignment 和 padding 的组合来精确控制的。

下面的示例说明了如何创建前图中的布局。一个 [`Border`](/api/avalonia/controls/border) 元素包裹着父级 [`StackPanel`](/api/avalonia/controls/stackpanel)，并设置了 15 个设备无关像素的 `Padding`。这就形成了围绕子 `StackPanel` 的那一圈窄窄的 `LightBlue` 色带。`StackPanel` 的子元素用来演示本文将详细介绍的各种定位属性，其中三个 `Button` 元素用于展示 `Margin` 和 `HorizontalAlignment` 属性。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="AvaloniaApplication2.MainWindow"
        Title="AvaloniaApplication2">
  <Border Background="LightBlue"
          BorderBrush="Black"
          BorderThickness="2"
          Padding="15">
    <StackPanel Background="White"
                HorizontalAlignment="Center"
                VerticalAlignment="Top">
      <TextBlock Margin="5,0"
                 FontSize="18"
                 HorizontalAlignment="Center">
        Alignment、Margin 和 Padding 示例
      </TextBlock>
      <Button HorizontalAlignment="Left" Margin="20">Button 1</Button>
      <Button HorizontalAlignment="Right" Margin="10">Button 2</Button>
      <Button HorizontalAlignment="Stretch">Button 3</Button>
    </StackPanel>
  </Border>
</Window>

```

下图对前述示例中使用的各种定位属性进行了放大展示。本文后续章节会更详细地说明如何使用每一种定位属性。

<Image light={LayoutMarginsPaddingAlignmentBasicAnnotatedScreenshot} alt="定位属性" position="center" maxWidth={400} cornerRadius="true"/>

## 理解对齐属性

`HorizontalAlignment` 和 `VerticalAlignment` 属性用于描述子元素应如何在父元素分配给它的布局空间中定位。将这两个属性结合使用，就可以精确控制子元素的位置。例如，`DockPanel` 的子元素可以指定四种不同的水平对齐方式：`Left`、`Right`、`Center`，或者使用 [`Stretch`](/api/avalonia/media/stretch) 填满可用空间。垂直定位也有类似的取值。

如果元素显式设置了 `Height` 和 `Width`，它们的优先级会高于 `Stretch`。也就是说，当你同时设置 `Height`、`Width` 和 `HorizontalAlignment="Stretch"` 时，`Stretch` 请求会被忽略。

### `HorizontalAlignment` 属性

`HorizontalAlignment` 属性用于声明应用于子元素的水平对齐方式。下表展示了 `HorizontalAlignment` 属性的可选值。

| 成员 | 说明 |
| :--- | :--- |
| `Left` | 子元素对齐到父元素分配布局空间的左侧。 |
| `Center` | 子元素对齐到父元素分配布局空间的中心。 |
| `Right` | 子元素对齐到父元素分配布局空间的右侧。 |
| `Stretch` \(默认\) | 子元素拉伸以填满父元素分配的布局空间。显式设置的 `Width` 和 `Height` 具有更高优先级。 |

下面的示例展示了如何将 `HorizontalAlignment` 属性应用到 `Button` 元素上。示例中列出了每个属性值，以便更清楚地展示不同的渲染行为。

```xml
<Button HorizontalAlignment="Left">按钮 1（Left）</Button>
<Button HorizontalAlignment="Right">按钮 2（Right）</Button>
<Button HorizontalAlignment="Center">按钮 3（Center）</Button>
<Button HorizontalAlignment="Stretch">按钮 4（Stretch）</Button>
```

上述代码会生成类似下图的布局。图中可以清楚看到每个 `HorizontalAlignment` 值所带来的定位效果。

<Image light={LayoutHorizontalAlignmentScreenshot} alt="HorizontalAlignment 示例" position="center" maxWidth={400} cornerRadius="true"/>

### `VerticalAlignment` 属性

`VerticalAlignment` 属性用于描述应用到子元素上的垂直对齐方式。下表展示了 `VerticalAlignment` 属性的可选值。

| 成员 | 说明 |
| :--- | :--- |
| `Top` | 子元素对齐到父元素分配布局空间的顶部。 |
| `Center` | 子元素对齐到父元素分配布局空间的中间。 |
| `Bottom` | 子元素对齐到父元素分配布局空间的底部。 |
| `Stretch` \(默认\) | 子元素拉伸以填满父元素分配的布局空间。显式设置的 `Width` 和 `Height` 具有更高优先级。 |

下面的示例展示了如何将 `VerticalAlignment` 属性应用到 `Button` 元素上。为了更好地演示每个属性值的布局行为，本示例使用了一个带可见网格线的 [`Grid`](/api/avalonia/controls/grid) 作为父元素。

```xml
<Border Background="LightBlue" BorderBrush="Black" BorderThickness="2" Padding="15">
    <Grid Background="White" ShowGridLines="True">
      <Grid.RowDefinitions>
        <RowDefinition Height="25"/>
        <RowDefinition Height="50"/>
        <RowDefinition Height="50"/>
        <RowDefinition Height="50"/>
        <RowDefinition Height="50"/>
      </Grid.RowDefinitions>
      <TextBlock Grid.Row="0" Grid.Column="0"
                 FontSize="18"
                 HorizontalAlignment="Center">
        VerticalAlignment 示例
      </TextBlock>
      <Button Grid.Row="1" Grid.Column="0" VerticalAlignment="Top">按钮 1（Top）</Button>
      <Button Grid.Row="2" Grid.Column="0" VerticalAlignment="Bottom">按钮 2（Bottom）</Button>
      <Button Grid.Row="3" Grid.Column="0" VerticalAlignment="Center">按钮 3（Center）</Button>
      <Button Grid.Row="4" Grid.Column="0" VerticalAlignment="Stretch">按钮 4（Stretch）</Button>
    </Grid>
</Border>
```

上述代码会生成类似下图的布局。图中可以清楚看到每个 `VerticalAlignment` 值所带来的定位效果。

<Image light={LayoutVerticalAlignmentScreenshot} alt="VerticalAlignment 属性示例" position="center" maxWidth={400} cornerRadius="true"/>

## 理解 Margin 属性

`Margin` 属性用于描述元素与其子元素或同级元素之间的距离。`Margin` 可以是统一值，例如 `Margin="20"`，表示为该元素四周都设置 20 个设备无关像素的统一外边距。`Margin` 也可以由四个独立数值组成，依次表示左、上、右、下四个方向的边距，例如 `Margin="0,10,5,25"`。正确使用 `Margin` 属性，可以非常精细地控制元素自身以及其相邻元素和子元素的渲染位置。

非零的 margin 会在元素 `Bounds` 之外增加额外空间。负 margin 也是合法的，它会将元素向相反方向拉动，从而导致元素与相邻元素重叠，或者超出父元素边界。

下面的示例展示了如何为一组 `Button` 元素应用统一 margin。每个 `Button` 在四周都会留出 10 像素的均匀间距。

```xml
<Button Margin="10">按钮 7</Button>
<Button Margin="10">按钮 8</Button>
<Button Margin="10">按钮 9</Button>
```

在很多情况下，统一 margin 并不适合。这时可以使用非统一间距。下面的示例展示了如何为子元素设置非统一 margin。margin 的顺序依次为：左、上、右、下。

```xml
<Button Margin="0,10,0,10">按钮 1</Button>
<Button Margin="0,10,0,10">按钮 2</Button>
<Button Margin="0,10,0,10">按钮 3</Button>
```

### 理解 Padding 属性

`Padding` 在很多方面与 `Margin` 类似。`Padding` 属性只在少数几个类上公开，主要是出于使用便利性的考虑，例如 `Border`、`TemplatedControl` 和 [`TextBlock`](/api/avalonia/controls/textblock) 都提供了 `Padding` 属性。`Padding` 属性会按照指定的 `Thickness` 值扩大子元素可用的内部空间。

下面的示例展示了如何将 `Padding` 应用于父级 `Border` 元素。

```xml
<Border Background="LightBlue"
        BorderBrush="Black"
        BorderThickness="2"
        CornerRadius="45"
        Padding="25">
```

### 在应用中使用对齐、边距和内边距

`HorizontalAlignment`、`Margin`、`Padding` 和 `VerticalAlignment` 提供了构建复杂 UI 所必需的定位控制能力。你可以利用这些属性的效果来改变子元素的位置，从而更灵活地创建动态应用和用户体验。

下面的示例演示了本文介绍的各个概念。在本主题第一个示例的基础上，该示例向 `Border` 中添加了一个 `Grid` 作为子元素。父级 `Border` 应用了 `Padding`，而 `Grid` 被用来在三个子 `StackPanel` 之间划分空间。示例再次使用 `Button` 元素来展示 `Margin` 和 `HorizontalAlignment` 的不同效果，并在各个 `ColumnDefinition` 中加入 `TextBlock` 元素，以更明确地说明每一列中 `Button` 所应用的属性。

```xml
<Border Background="LightBlue"
        BorderBrush="Black"
        BorderThickness="2"
        CornerRadius="45"
        Padding="25">
    <Grid Background="White" ShowGridLines="True">
      <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto"/>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="Auto"/>
      </Grid.ColumnDefinitions>

    <StackPanel Grid.Column="0" Grid.Row="0"
                HorizontalAlignment="Left"
                Name="StackPanel1"
                VerticalAlignment="Top">
        <TextBlock FontSize="18" HorizontalAlignment="Center" Margin="0,0,0,15">StackPanel1</TextBlock>
        <Button Margin="0,10,0,10">Button 1</Button>
        <Button Margin="0,10,0,10">Button 2</Button>
        <Button Margin="0,10,0,10">Button 3</Button>
        <TextBlock>ColumnDefinition.Width="Auto"</TextBlock>
        <TextBlock>StackPanel.HorizontalAlignment="Left"</TextBlock>
        <TextBlock>StackPanel.VerticalAlignment="Top"</TextBlock>
        <TextBlock>StackPanel.Orientation="Vertical"</TextBlock>
        <TextBlock>Button.Margin="0,10,0,10"</TextBlock>
    </StackPanel>

    <StackPanel Grid.Column="1" Grid.Row="0"
                HorizontalAlignment="Stretch"
                Name="StackPanel2"
                VerticalAlignment="Top"
                Orientation="Vertical">
        <TextBlock FontSize="18" HorizontalAlignment="Center" Margin="0,0,0,15">StackPanel2</TextBlock>
        <Button Margin="10,0,10,0">Button 4</Button>
        <Button Margin="10,0,10,0">Button 5</Button>
        <Button Margin="10,0,10,0">Button 6</Button>
        <TextBlock HorizontalAlignment="Center">ColumnDefinition.Width="*"</TextBlock>
        <TextBlock HorizontalAlignment="Center">StackPanel.HorizontalAlignment="Stretch"</TextBlock>
        <TextBlock HorizontalAlignment="Center">StackPanel.VerticalAlignment="Top"</TextBlock>
        <TextBlock HorizontalAlignment="Center">StackPanel.Orientation="Horizontal"</TextBlock>
        <TextBlock HorizontalAlignment="Center">Button.Margin="10,0,10,0"</TextBlock>
    </StackPanel>

    <StackPanel Grid.Column="2" Grid.Row="0"
                HorizontalAlignment="Left"
                Name="StackPanel3"
                VerticalAlignment="Top">
        <TextBlock FontSize="18" HorizontalAlignment="Center" Margin="0,0,0,15">StackPanel3</TextBlock>
        <Button Margin="10">Button 7</Button>
        <Button Margin="10">Button 8</Button>
        <Button Margin="10">Button 9</Button>
        <TextBlock>ColumnDefinition.Width="Auto"</TextBlock>
        <TextBlock>StackPanel.HorizontalAlignment="Left"</TextBlock>
        <TextBlock>StackPanel.VerticalAlignment="Top"</TextBlock>
        <TextBlock>StackPanel.Orientation="Vertical"</TextBlock>
        <TextBlock>Button.Margin="10"</TextBlock>
    </StackPanel>
  </Grid>
</Border>
```

When compiled, the preceding application yields a UI that looks like the following illustration. The effects of the various property values are evident in the spacing between elements, and significant property values for elements in each column are shown within `TextBlock` elements.

<Image light={LayoutMarginsPaddingAlignmentComplexAnnotatedScreenshot} alt="Several positioning properties in one application" position="center" maxWidth={400} cornerRadius="true"/>

## See also

- [Layout](/docs/layout): How the measure and arrange system works.
- [Choosing a Layout Panel](/docs/layout/choosing-a-layout-panel): Picking the right panel for your scenario.
