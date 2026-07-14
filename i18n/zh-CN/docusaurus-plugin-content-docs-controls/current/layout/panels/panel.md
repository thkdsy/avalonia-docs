---
id: panel
title: Panel
description: 一个基础布局控件，可将多个子控件层叠显示，并通过对齐属性定位它们。
doc-type: reference
---

# Panel

`Panel` 是最基础的一种布局控件，可以包含多个子控件。它会按照子控件在 XAML 中出现的顺序进行绘制，并将它们层叠在一起。每个子控件都会根据其 `HorizontalAlignment` 和 `VerticalAlignment` 属性进行定位。

由于 `Panel` 不会将子元素排列成行、列或其他结构，因此它最适合用于需要内容重叠的场景，例如在图片上方放置文本，或堆叠装饰性元素。

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Background` | `IBrush` | 面板的背景画刷。若要让面板接收指针事件，你必须设置它（即使设置为 `Transparent` 也可以）。 |
| `Children` | `Controls` | 面板中包含的子控件集合。 |

## 基本示例

此示例使用 50% 的透明度来演示子控件之间的重叠效果。

<XamlPreview>

```xml
<Panel xmlns="https://github.com/avaloniaui"
       Margin="10">
    <Rectangle Fill="Red" Height="100" VerticalAlignment="Top"/>
    <Rectangle Fill="Green" Height="100" VerticalAlignment="Bottom"/>
    <Rectangle Fill="Blue" Width="100" HorizontalAlignment="Right" />
    <Rectangle Fill="Orange" Width="100" HorizontalAlignment="Left"/>
</Panel>
```

</XamlPreview>

## 使用 `ZIndex` 控制重叠顺序

当子元素发生重叠时，你可以使用附加属性 `ZIndex` 来控制绘制顺序。值越大，绘制层级越高。默认情况下，所有子元素的 `ZIndex` 都为 0，并按它们在标记中出现的顺序绘制。

```xml
<Panel>
    <Border Background="Red" Width="100" Height="100" ZIndex="1" />
    <Border Background="Blue" Width="100" Height="100" Margin="30,30,0,0" ZIndex="2" />
</Panel>
```

在此示例中，蓝色边框会绘制在红色边框之上，因为它拥有更高的 `ZIndex`。

## 为命中测试设置背景

如果你不设置 `Background`，那么面板对指针事件来说是透明的。点击和其他指针交互会直接穿透到面板后面的内容。若要让面板在整个区域内响应指针事件，请将 `Background` 设置为 `Transparent`：

```xml
<Panel Background="Transparent">
    <TextBlock Text="This panel captures pointer events everywhere." />
</Panel>
```

## 将 `Panel` 作为自定义面板的基类

`Panel` 是所有内置面板控件的基类。如果没有任何内置面板能满足你的布局需求，你可以通过继承 `Panel` 并重写其 `MeasureOverride` 和 `ArrangeOverride` 方法来创建自定义面板。

```csharp
public class MyCustomPanel : Panel
{
    protected override Size MeasureOverride(Size availableSize)
    {
        foreach (var child in Children)
        {
            child.Measure(availableSize);
        }

        return availableSize;
    }

    protected override Size ArrangeOverride(Size finalSize)
    {
        foreach (var child in Children)
        {
            child.Arrange(new Rect(finalSize));
        }

        return finalSize;
    }
}
```

:::info
完整的演练请参阅 [Custom panel](/docs/custom-controls/custom-panel)。
:::

## 其他面板控件

如果你需要对子元素的位置拥有更多控制能力，可以考虑使用以下专门的面板：

- [Stack panel](/controls/layout/panels/stackpanel)：将子元素排列在单一的水平或垂直线上。
- [Dock panel](/controls/layout/panels/dockpanel)：将子元素停靠到面板边缘。
- [Grid](/controls/layout/panels/grid)：将子元素排列到行和列中。
- [Wrap panel](/controls/layout/panels/wrappanel)：将子元素排列成一行，到达面板边缘时自动换行。
- [Canvas](/controls/layout/panels/canvas)：使用显式坐标定位子元素。
- [Relative panel](/controls/layout/panels/relativepanel)：让子元素相对于彼此或面板本身定位。
- [Uniform grid](/controls/layout/panels/uniformgrid)：将子元素排列到一个每个单元格尺寸相同的网格中。

## 另请参阅

- [Panel API reference](/api/avalonia/controls/panel)
- [`Panel.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Panel.cs)
- [Custom panel](/docs/custom-controls/custom-panel)
