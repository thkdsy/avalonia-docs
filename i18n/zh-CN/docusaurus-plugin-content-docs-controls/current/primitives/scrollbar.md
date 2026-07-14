---
id: scrollbar
title: ScrollBar
description: 一个基础控件，提供可拖动的滑块和轨道，用于在水平或垂直方向滚动内容。
doc-type: reference
---

import ScrollBarScreenshot from '/img/controls/scrollbar/scrollbar.gif';

[`ScrollBar`](/api/avalonia/controls/primitives/scrollbar) 控件提供了可拖动的滑块和轨道，可用于滚动内容。你可以将其显示为水平或垂直方向。默认情况下，它的取值范围是 0 到 100（类型为 `double`）。

你可以通过设置 `Minimum` 和 `Maximum` 属性来配置范围，并通过小步进和大步进控制值的变化方式。小步进由键盘方向键触发，而大步进由点击滚动条轨道或按下 Page Up、Page Down 键触发。

:::info
在大多数情况下，你并不需要直接使用 `ScrollBar`。`ScrollViewer` 控件会自动管理滚动条。只有在你需要独立的滑动条式输入，或自定义滚动行为时，才应直接使用 `ScrollBar`。
:::

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| [`Orientation`](/api/avalonia/layout/orientation) | `Orientation` | 设置滚动条方向。可选 `Horizontal` 或 `Vertical`。默认值为 `Vertical`。 |
| `Minimum` | `double` | 滚动条可表示的最小值。默认值为 `0`。 |
| `Maximum` | `double` | 滚动条可表示的最大值。默认值为 `100`。 |
| `Value` | `double` | 滚动条当前值。 |
| `ViewportSize` | `double` | 可见区域（viewport）相对于整体范围的大小。它会决定滑块的尺寸。 |
| `SmallChange` | `double` | 小步进时的值变化量（方向键触发）。默认值为 `1`。 |
| `LargeChange` | `double` | 大步进时的值变化量（点击轨道或 Page Up/Page Down 触发）。默认值为 `10`。 |
| `Visibility` | `ScrollBarVisibility` | 控制滚动条何时可见。可选 `Disabled`、`Auto`、`Visible` 或 `Hidden`。 |
| [`VerticalAlignment`](/api/avalonia/layout/verticalalignment) | `VerticalAlignment` | 滚动条在容器中的垂直对齐方式。可选 `Top`、`Bottom`、`Center` 或 `Stretch`。 |
| [`HorizontalAlignment`](/api/avalonia/layout/horizontalalignment) | `HorizontalAlignment` | 滚动条在容器中的水平对齐方式。可选 `Left`、`Right`、`Center` 或 `Stretch`。 |

:::caution
若要创建合理的布局，你需要正确搭配方向属性和对齐属性。例如，`Vertical` 滚动条通常使用 `HorizontalAlignment` 来定位自己（如 `Left` 或 `Right`），而 `Horizontal` 滚动条通常使用 `VerticalAlignment`。
:::

## 方向

设置 `Orientation` 属性来控制滚动条的方向。

```xml
<!-- 垂直滚动条（默认） -->
<ScrollBar Orientation="Vertical" HorizontalAlignment="Left" />

<!-- 水平滚动条 -->
<ScrollBar Orientation="Horizontal" VerticalAlignment="Bottom" />
```

## 配置范围

你可以根据需要自定义取值范围和步进大小。

```xml
<ScrollBar Minimum="0"
           Maximum="500"
           SmallChange="5"
           LargeChange="50"
           Value="100" />
```

## 示例

下面的示例将一个垂直滚动条放置在面板中，并在滚动时使用文本块显示当前值。

```xml
<Panel>
  <Border Background="AliceBlue">
    <ScrollBar Visibility="Auto"
               HorizontalAlignment="Left"
               Scroll="ScrollHandler" />
  </Border>
  <TextBlock Name="valueText" Margin="60">0</TextBlock>
</Panel>
```

```csharp title='C#'
using Avalonia.Controls;
using Avalonia.Controls.Primitives;

namespace AvaloniaControls.Views
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        public void ScrollHandler(object source, ScrollEventArgs args)
        {
            valueText.Text = args.NewValue.ToString();
        }
    }
}
```

通过这段 code-behind 代码，你在拖动滚动条时，文本块会显示其当前值。

<Image light={ScrollBarScreenshot} alt="ScrollBar example showing value tracking" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [ScrollViewer](/controls/layout/containers/scrollviewer)
- [ScrollBar API reference](/api/avalonia/controls/primitives/scrollbar)
- [`ScrollBar.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Primitives/ScrollBar.cs)
