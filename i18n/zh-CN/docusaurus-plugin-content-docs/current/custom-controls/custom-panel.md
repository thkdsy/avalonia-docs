---
id: custom-panel
title: 创建自定义面板
description: 通过重写 MeasureOverride 和 ArrangeOverride 来实现自定义布局面板。
doc-type: how-to
---

自定义面板可以让你精确控制子元素如何被测量和排列。通过继承 `Panel` 并重写其布局方法，你可以创建超出内置面板能力范围的布局方式。

## 布局过程

Avalonia 使用双阶段布局系统。每个面板都必须参与这两个阶段：

1. **测量阶段（`MeasureOverride`）**：面板接收一个可用大小，并决定自己需要多少空间。在这个阶段，你必须对每个子元素调用 `child.Measure()`。随后每个子元素都会设置自己的 `DesiredSize`，你可以利用这些值计算面板本身的期望大小。

2. **排列阶段（`ArrangeOverride`）**：面板会接收最终分配到的大小，并在该空间内定位每个子元素。你必须对每个子元素调用 `child.Arrange()`，并传入一个定义其位置和大小的 `Rect`。

## 基本示例

这个简单的 `PlotPanel` 会将所有子元素放置在固定偏移量 (50, 50) 的位置。

```csharp
public class PlotPanel : Panel
{
    // 重写 Panel 默认的 Measure 方法
    protected override Size MeasureOverride(Size availableSize)
    {
        var panelDesiredSize = new Size();

        // 在这个示例中，我们只考虑一个子元素。
        // 报告面板所需大小为该唯一子元素的大小。
        foreach (var child in Children)
        {
            child.Measure(availableSize);
            panelDesiredSize = child.DesiredSize;
        }

        return panelDesiredSize;
    }

    protected override Size ArrangeOverride(Size finalSize)
    {
        foreach (var child in Children)
        {
            double x = 50;
            double y = 50;

            child.Arrange(new Rect(new Point(x, y), child.DesiredSize));
        }

        return finalSize; // 返回最终排列后的大小
    }
}
```

## 实用示例：径向面板

一个更实用的自定义面板会将子元素均匀排列在圆周上。每个子元素与其他元素之间都具有相等的角度偏移。

```csharp
public class RadialPanel : Panel
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
        double centerX = finalSize.Width / 2;
        double centerY = finalSize.Height / 2;
        double radius = Math.Min(centerX, centerY) - 30;

        for (int i = 0; i < Children.Count; i++)
        {
            double angle = 2 * Math.PI * i / Children.Count - Math.PI / 2;
            double x = centerX + radius * Math.Cos(angle) - Children[i].DesiredSize.Width / 2;
            double y = centerY + radius * Math.Sin(angle) - Children[i].DesiredSize.Height / 2;
            Children[i].Arrange(new Rect(new Point(x, y), Children[i].DesiredSize));
        }
        return finalSize;
    }
}
```

随后你就可以像使用其他面板一样在 XAML 中使用它：

```xml
<local:RadialPanel Width="300" Height="300">
    <Button Content="1" />
    <Button Content="2" />
    <Button Content="3" />
    <Button Content="4" />
    <Button Content="5" />
</local:RadialPanel>
```

## 使用附加属性

面板通常需要针对每个子元素进行单独配置。附加属性允许每个子元素携带一些数据，供父面板在布局时读取。例如，你可以为 `RadialPanel` 添加一个 `Slot` 属性，让子元素指定自己在圆周上的位置：

```csharp
public static readonly AttachedProperty<int> SlotProperty =
    AvaloniaProperty.RegisterAttached<RadialPanel, Control, int>("Slot");

public static int GetSlot(Control element) => element.GetValue(SlotProperty);
public static void SetSlot(Control element, int value) => element.SetValue(SlotProperty, value);
```

这样，面板就可以在 `ArrangeOverride` 中调用 `GetSlot(child)` 来确定每个子元素应放置的位置。

关于如何定义附加属性的完整说明，请参阅[附加属性](/docs/custom-controls/attached-properties)。

## 提示

- 在 `MeasureOverride` 中始终对每个子元素调用 `Measure`。未被测量的子元素将无法正确渲染。
- 在 `ArrangeOverride` 中始终对每个子元素调用 `Arrange`。未被排列的子元素将不会显示。
- 当注册会影响布局的样式属性时，请使用 `AffectsMeasure` 或 `AffectsArrange`。这样可确保这些属性变化时面板会重新布局。
- 在 `MeasureOverride` 中返回面板真正需要的大小。返回过大的尺寸会浪费空间，返回过小则可能裁剪子元素。

## 另请参阅

- [自定义 ItemsPanel](/docs/custom-controls/custom-itemspanel)
- [附加属性](/docs/custom-controls/attached-properties)
- [布局](/docs/layout)
