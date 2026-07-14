---
id: relativepanel
title: RelativePanel
description: 了解如何在 Avalonia 中使用 RelativePanel，让子控件相对于彼此或相对于面板进行定位。
doc-type: reference
---

`RelativePanel` 控件允许你通过指定子控件相对于其他（兄弟）子控件的位置，或相对于面板本身的位置，来排列它们。位置的计算基于面板控件内部（内容区）以及子控件 margin 区的外边缘。

子控件的默认位置是面板的左上角。

你需要使用附加的相对定位属性来指定子控件的布局。格式如下：

`RelativePanel.PositionProperty="NameOfSibling"`

其中 `PositionProperty` 是某个相对定位属性（见下表），而 `NameOfSibling` 则是另一个子控件的名称属性值。

:::danger
如果将某个相对定位属性的值设置为子控件自身的名称，那就是错误的，因为这会形成循环引用！
:::

每个子控件最多可以指定四个相对定位属性——分别用于计算其上、下、左、右边缘的位置。

:::danger
如果为同一个子控件重复定义同一个相对定位属性，也是错误的。
:::

指定多个不同但可能互相冲突的相对定位属性本身不算错误，不过最终结果可能会比较难以理解。

如果多个子控件最终落在同一个计算位置上，它们会按照在 XAML 中出现的顺序进行绘制，并可能互相重叠或遮挡。

:::caution
这意味着你必须为子控件设置名称，并在相对定位属性值中使用正确的名称。如果写错了，该控件会采用默认位置（左上角），并可能与其他控件重叠或互相遮挡。
:::

## 常用属性

你最常使用的通常是这些属性：

<table><thead><tr><th width="348">Property</th><th>Description</th></tr></thead><tbody><tr><td><code>AlignTopWithPanel</code></td><td>Boolean. Align the top edge of the child control with the top edge of the panel.</td></tr><tr><td><code>AlignBottomWithPanel</code></td><td>Boolean. Attached to a child control to align the bottom edge of the child control with the bottom edge of the panel.</td></tr><tr><td><code>AlignLeftWithPanel</code></td><td>Boolean. Attached to a child control to align the left edge of the child control with the left edge of the panel.</td></tr><tr><td><code>AlignRightWithPanel</code></td><td>Boolean. Attached to a child control to align the right edge of the child control with the right edge of the panel.</td></tr><tr><td><code>AlignHorizontalCenterWithPanel</code></td><td>Boolean. Attached to a child control to align the horizontal center of the child control with the horizontal center of the panel.</td></tr><tr><td><code>AlignVerticalCenterWithPanel</code></td><td>Boolean. Attached to a child control to align the vertical center of the child control with the vertical center of the panel.</td></tr><tr><td><code>AlignTopWith</code></td><td>Attached to a child control to align its top edge with the top edge of the named sibling.</td></tr><tr><td><code>AlignBottomWith</code></td><td>Attached to a child control to align its bottom edge with the bottom edge of the named sibling.</td></tr><tr><td><code>AlignLeftWith</code></td><td>Attached to a child control to align its left edge with the left edge of the named sibling.</td></tr><tr><td><code>AlignRightWith</code></td><td>Attached to a child control to align its right edge with the right edge of the named sibling.</td></tr><tr><td><code>AlignHorizontalCenterWith</code></td><td>Attached to a child control to align its horizontal center with the horizontal center of the named sibling.</td></tr><tr><td><code>AlignVerticalCenterWith</code></td><td>Attached to a child control to align its vertical center with the vertical center of the named sibling.</td></tr><tr><td><code>Above</code></td><td>Attached to a child control to align its bottom edge with the top edge of the named sibling.</td></tr><tr><td><code>Below</code></td><td>Attached to a child control to align its top edge with the bottom edge of the named sibling.</td></tr><tr><td><code>LeftOf</code></td><td>Attached to a child control to align its right edge with the left edge of the named sibling.</td></tr><tr><td><code>RightOf</code></td><td>Attached to a child control to align its left edge with the right edge of the named sibling.</td></tr></tbody></table>

## Example

This XAML shows how to arrange some child controls in different ways:

<XamlPreview>

```xml
<Border xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        BorderBrush="DarkGray" BorderThickness="1"
        Margin="20">
  <RelativePanel>
    <Rectangle x:Name="RedRect" Fill="Red" Height="50" Width="50"/>
    <Rectangle x:Name="BlueRect" Fill="Blue" Opacity="0.5" Height="50" Width="150"
               RelativePanel.RightOf="RedRect" />
    <Rectangle x:Name="GreenRect" Fill="Green" Height="100"
               RelativePanel.Below="RedRect"
               RelativePanel.AlignLeftWith="RedRect"
               RelativePanel.AlignRightWith="BlueRect"/>
    <Rectangle Fill="Orange"
               RelativePanel.Below="GreenRect"
               RelativePanel.AlignLeftWith="BlueRect"
               RelativePanel.AlignRightWithPanel="True"
               RelativePanel.AlignBottomWithPanel="True"/>
  </RelativePanel>
</Border>
```

</XamlPreview>

Here are some notes about the above example:

* The red rectangle is given a size (50x50) but no relative position. It is therefore placed in the default (top-left) position.
* The blue rectangle has a 50% opacity to demonstrate that is is not overlapping any other.
* The green rectangle is given a height (100), but no width. Its left side is aligned with the red rectangle, and its right side is aligned with the blue rectangle, this calculates its width.
* The orange rectangle has not been given a size. Its left side is aligned with the blue rectangle. Its right and bottom edges are aligned with the edge of the panel. Therefore its size is determined by the alignments and it will resize if the panel itself is resized.

## See also

- [RelativePanel API reference](/api/avalonia/controls/relativepanel)
- [`RelativePanel.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/RelativePanel.cs)
- [Canvas](/controls/layout/panels/canvas)
- [DockPanel](/controls/layout/panels/dockpanel)
- [Grid](/controls/layout/panels/grid)
- [Panel](/controls/layout/panels/panel)
- [StackPanel](/controls/layout/panels/stackpanel)
- [UniformGrid](/controls/layout/panels/uniformgrid)
- [WrapPanel](/controls/layout/panels/wrappanel)
