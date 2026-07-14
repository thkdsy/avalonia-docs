---
id: dockpanel
title: DockPanel
description: 了解如何在 Avalonia 中使用 DockPanel 将子控件停靠到容器边缘。
doc-type: reference
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import DockPanelTopScreenshot from '/img/controls/dockpanel/dockpanel-top.png';

[`DockPanel`](/api/avalonia/controls/dockpanel) 控件会将其子控件沿指定的“停靠边缘”（上、下、左、右）进行排列，并由最后一个子控件填充所有剩余空间。DockPanel 可以保持子控件在与停靠边缘平行方向上的尺寸，使子控件沿该停靠边缘填满全部可用空间。

例如，如果某个子控件的停靠边缘被定义为“top”，并且它定义了高度但没有定义宽度，那么它会像下面这样绘制：

<Image light={DockPanelTopScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

:::caution
你必须定义子控件在垂直于停靠边缘方向上的尺寸，否则它将不会显示。
:::

你也可以选择定义子控件在与停靠边缘平行方向上的尺寸。在这种情况下，子控件会按照同方向上的对齐设置进行绘制。例如，一个定义了宽度、并停靠到顶部边缘的子控件，会遵循其水平对齐属性（默认是 center）。

子控件会按照它们在 XAML 中定义的顺序依次停靠。Avalonia 在确定某个子控件尺寸时，会考虑之前已经绘制过的控件，因此它们之间不会发生重叠。

最后定义的子控件会填充剩余的所有空间。

:::caution
你必须始终定义一个最后的子控件（且不带 dock 属性），否则停靠计算将无法正确执行。这意味着 DockPanel 至少需要两个子控件。
:::

## 常用属性

你最常使用的通常是这些属性：

<table><thead><tr><th width="266">属性</th><th>说明</th></tr></thead><tbody><tr><td>DockPanel.Dock<code>.Left</code></td><td>附加到子控件——将其停靠到左侧。</td></tr><tr><td>DockPanel.Dock<code>.Top</code></td><td>附加到子控件——将其停靠到顶部边缘。</td></tr><tr><td>DockPanel.Dock<code>.Right</code></td><td>附加到子控件——将其停靠到右侧。</td></tr><tr><td>DockPanel.Dock<code>.Bottom</code></td><td>附加到子控件——将其停靠到底部边缘。</td></tr><tr><td><code>HorizontalSpacing</code></td><td>设置已停靠子控件之间的水平间距（double，默认 0）。</td></tr><tr><td><code>VerticalSpacing</code></td><td>设置已停靠子控件之间的垂直间距（double，默认 0）。</td></tr></tbody></table>

## 根据内容确定尺寸

如果没有指定 `Height` 和 `Width` 属性，`DockPanel` 会根据其内容来确定自身尺寸。它的大小会随子元素尺寸变化而增减。不过，当你显式指定了这些属性，并且已经没有足够空间容纳下一个子元素时，`DockPanel` 就不会显示该子元素及其后续子元素，也不会再测量这些后续子元素。

## LastChildFill

默认情况下，`DockPanel` 元素中的最后一个子元素会“填充”所有剩余且尚未分配的空间。如果你不希望这样，请将 `LastChildFill` 属性设置为 `false`。

## 示例

将橙色矩形的透明度设置为 0.5，可以直观看出这些元素之间没有发生重叠。

<XamlPreview>

```xml
<DockPanel xmlns="https://github.com/avaloniaui"
           Width="300" Height="300">
    <Rectangle Fill="Red" Height="100" DockPanel.Dock="Top"/>
    <Rectangle Fill="Blue" Width="100" DockPanel.Dock="Left" />
    <Rectangle Fill="Green" Height="100" DockPanel.Dock="Bottom"/>
    <Rectangle Fill="Orange" Width="100" DockPanel.Dock="Right" Opacity="0.5"/>
    <Rectangle Fill="Gray" />
</DockPanel>
```

</XamlPreview>

## 在代码中定义 DockPanel

<Tabs
  defaultValue="xaml"
  values={[
      { label: 'XAML', value: 'xaml', },
      { label: 'C#', value: 'cs', },
  ]}
>
<TabItem value="xaml">

```xml
<DockPanel LastChildFill="True">
  <Border Height="25" Background="SkyBlue" BorderBrush="Black" BorderThickness="1" DockPanel.Dock="Top">
    <TextBlock Foreground="Black">Dock = "Top"</TextBlock>
  </Border>
  <Border Height="25" Background="SkyBlue" BorderBrush="Black" BorderThickness="1" DockPanel.Dock="Top">
    <TextBlock Foreground="Black">Dock = "Top"</TextBlock>
  </Border>
  <Border Height="25" Background="LemonChiffon" BorderBrush="Black" BorderThickness="1" DockPanel.Dock="Bottom">
    <TextBlock Foreground="Black">Dock = "Bottom"</TextBlock>
  </Border>
  <Border Width="200" Background="PaleGreen" BorderBrush="Black" BorderThickness="1" DockPanel.Dock="Left">
    <TextBlock Foreground="Black">Dock = "Left"</TextBlock>
  </Border>
  <Border Background="White" BorderBrush="Black" BorderThickness="1">
    <TextBlock Foreground="Black">This content will "Fill" the remaining space</TextBlock>
  </Border>
</DockPanel>
```

</TabItem>
<TabItem value="cs">

```cs
// 创建 DockPanel
DockPanel myDockPanel = new DockPanel();
myDockPanel.LastChildFill = true;

// 定义子内容
Border myBorder1 = new Border();
myBorder1.Height = 25;
myBorder1.Background = Brushes.SkyBlue;
myBorder1.BorderBrush = Brushes.Black;
myBorder1.BorderThickness = new Thickness(1);
DockPanel.SetDock(myBorder1, Dock.Top);
TextBlock myTextBlock1 = new TextBlock();
myTextBlock1.Foreground = Brushes.Black;
myTextBlock1.Text = "Dock = Top";
myBorder1.Child = myTextBlock1;

Border myBorder2 = new Border();
myBorder2.Height = 25;
myBorder2.Background = Brushes.SkyBlue;
myBorder2.BorderBrush = Brushes.Black;
myBorder2.BorderThickness = new Thickness(1);
DockPanel.SetDock(myBorder2, Dock.Top);
TextBlock myTextBlock2 = new TextBlock();
myTextBlock2.Foreground = Brushes.Black;
myTextBlock2.Text = "Dock = Top";
myBorder2.Child = myTextBlock2;

Border myBorder3 = new Border();
myBorder3.Height = 25;
myBorder3.Background = Brushes.LemonChiffon;
myBorder3.BorderBrush = Brushes.Black;
myBorder3.BorderThickness = new Thickness(1);
DockPanel.SetDock(myBorder3, Dock.Bottom);
TextBlock myTextBlock3 = new TextBlock();
myTextBlock3.Foreground = Brushes.Black;
myTextBlock3.Text = "Dock = Bottom";
myBorder3.Child = myTextBlock3;

Border myBorder4 = new Border();
myBorder4.Width = 200;
myBorder4.Background = Brushes.PaleGreen;
myBorder4.BorderBrush = Brushes.Black;
myBorder4.BorderThickness = new Thickness(1);
DockPanel.SetDock(myBorder4, Dock.Left);
TextBlock myTextBlock4 = new TextBlock();
myTextBlock4.Foreground = Brushes.Black;
myTextBlock4.Text = "Dock = Left";
myBorder4.Child = myTextBlock4;

Border myBorder5 = new Border();
myBorder5.Background = Brushes.White;
myBorder5.BorderBrush = Brushes.Black;
myBorder5.BorderThickness = new Thickness(1);
TextBlock myTextBlock5 = new TextBlock();
myTextBlock5.Foreground = Brushes.Black;
myTextBlock5.Text = "This content will Fill the remaining space";
myBorder5.Child = myTextBlock5;

// 将子元素添加到 DockPanel 的 Children 集合中
myDockPanel.Children.Add(myBorder1);
myDockPanel.Children.Add(myBorder2);
myDockPanel.Children.Add(myBorder3);
myDockPanel.Children.Add(myBorder4);
myDockPanel.Children.Add(myBorder5);
```
</TabItem>  

</Tabs>

## 另请参阅

- [DockPanel API reference](/api/avalonia/controls/dockpanel)
- [`DockPanel.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/DockPanel.cs)
- [Canvas](/controls/layout/panels/canvas)
- [Grid](/controls/layout/panels/grid)
- [Panel](/controls/layout/panels/panel)
- [RelativePanel](/controls/layout/panels/relativepanel)
- [StackPanel](/controls/layout/panels/stackpanel)
- [UniformGrid](/controls/layout/panels/uniformgrid)
- [WrapPanel](/controls/layout/panels/wrappanel)
