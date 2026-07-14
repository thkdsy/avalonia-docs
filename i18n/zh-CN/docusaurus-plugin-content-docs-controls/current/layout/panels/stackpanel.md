---
id: stackpanel
title: StackPanel
description: 一个将子控件排列为单行的面板，可选择水平或垂直方向。
doc-type: reference
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

[`StackPanel`](/api/avalonia/controls/stackpanel) 会通过水平或垂直堆叠的方式排列其子控件。你通常会使用 StackPanel 来组织页面中某个较小的 UI 区域。

在 `StackPanel` 内部，如果你没有为子控件设置与堆叠方向垂直的那个尺寸属性，那么该子控件会拉伸以填充可用空间。例如，在水平方向排列时，如果没有显式设置 `Height`，子控件就会在高度方向上拉伸填充。

在堆叠方向上，`StackPanel` 总会扩展自身以容纳所有子控件。

:::tip
`StackPanel` 本身不会滚动。如果你的堆叠内容可能超出可用空间，请将 `StackPanel` 包裹在 `ScrollViewer` 中。
:::

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
| ------------- | ------------------------------------------------------------------------------- |
| [`Orientation`](/api/avalonia/layout/orientation) | 设置堆叠方向。可选 `Horizontal` 或 `Vertical`（默认）。 |
| `Spacing` | 在相邻子控件之间创建均匀间距。 |
| `HorizontalAlignment` | 控制面板自身在父容器中的水平位置。 |
| `VerticalAlignment` | 控制面板自身在父容器中的垂直位置。 |

## 示例

下面的 XAML 展示了如何创建一个垂直 StackPanel。效果是：子控件会被拉伸以适应宽度，而 StackPanel 的整体高度则等于各子控件高度之和。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Width="200">
    <Rectangle Fill="Red" Height="50"/>
    <Rectangle Fill="Blue" Height="50"/>
    <Rectangle Fill="Green" Height="50"/>
    <Rectangle Fill="Orange" Height="50"/>
</StackPanel>
```

</XamlPreview>

## 在代码中定义 StackPanel

下面的示例演示了如何使用 `StackPanel` 创建一组垂直排列的按钮。如果需要水平排列，请将 `Orientation` 属性设置为 `Horizontal`。

<Tabs
  defaultValue="xaml"
  values={[
      { label: 'XAML', value: 'xaml', },
      { label: 'C#', value: 'cs', },
  ]}
>
<TabItem value="xaml">

```xml
<StackPanel HorizontalAlignment="Center"
            VerticalAlignment="Top"
            Spacing="25">
    <Button Content="Button 1" />
    <Button Content="Button 2" />
    <Button Content="Button 3" />
</StackPanel>
```

</TabItem>
<TabItem value="cs">

```csharp
// 定义 StackPanel
var myStackPanel = new StackPanel();
myStackPanel.HorizontalAlignment = HorizontalAlignment.Center;
myStackPanel.VerticalAlignment = VerticalAlignment.Top;
myStackPanel.Spacing = 25;

// 定义子内容
Button myButton1 = new Button();
myButton1.Content = "Button 1";
Button myButton2 = new Button();
myButton2.Content = "Button 2";
Button myButton3 = new Button();
myButton3.Content = "Button 3";

// 将子元素添加到父级 StackPanel 中
myStackPanel.Children.Add(myButton1);
myStackPanel.Children.Add(myButton2);
myStackPanel.Children.Add(myButton3);
```

</TabItem>

</Tabs>

## 居中项目

若要让所有子元素在堆叠中居中对齐，请将 `HorizontalAlignment` 设置为 `Center`：

```xml
<StackPanel HorizontalAlignment="Center" Spacing="8">
    <Button Content="Short" />
    <Button Content="A longer button" />
</StackPanel>
```

## 带间距的水平堆叠

你可以通过将 `Orientation` 设置为 `Horizontal` 并添加 `Spacing` 值来创建水平按钮栏：

```xml
<StackPanel Orientation="Horizontal" Spacing="12">
    <Button Content="Save" />
    <Button Content="Cancel" />
</StackPanel>
```

## 实用说明

- **尺寸行为**：`StackPanel` 不会在堆叠方向上约束子元素，因此每个子元素都会获得自己所请求的空间。如果你需要让子元素按比例共享空间，请考虑改用 `Grid`。
- **性能**：对于包含大量项目的列表，优先使用带虚拟化的 `ItemsRepeater` 或 `ListBox`，而不是在 `StackPanel` 中放置大量控件。
- **滚动**：由于 `StackPanel` 会增长以容纳所有子元素，因此它不会自行裁剪内容。当可能发生溢出时，请将其包裹在 `ScrollViewer` 中。
- **反向顺序**：`StackPanel` 不支持反向堆叠。如果你想反转视觉顺序，请反转子元素的定义顺序，或使用自定义面板。

## 另请参阅

- [StackPanel API reference](/api/avalonia/controls/stackpanel)
- [`StackPanel.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/StackPanel.cs)
- [DockPanel](/controls/layout/panels/dockpanel)
- [Grid](/controls/layout/panels/grid)
- [WrapPanel](/controls/layout/panels/wrappanel)
- [Panel](/controls/layout/panels/panel)
