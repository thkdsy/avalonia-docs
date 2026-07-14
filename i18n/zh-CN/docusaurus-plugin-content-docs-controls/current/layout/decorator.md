---
id: decorator
title: Decorator
description: 一个用于包装并装饰单个子元素的控件基类，提供内边距能力，并作为自定义包装控件的基础。
doc-type: reference
---

[`Decorator`](/api/avalonia/controls/decorator) 控件是用于包装并装饰单个子元素的控件基类。它负责在逻辑树和可视树中承载一个子元素，并可在其周围应用可选的内边距。

## 何时使用 [`Decorator`](/api/avalonia/controls/decorator)

通常情况下，你不会在 XAML 中直接使用 `Decorator`。相反，你会使用派生自它的内置控件，例如 `Border` 或 `Viewbox`。不过，在以下两种场景中，`Decorator` 会很有用：

- **派生子类**：当你需要一个围绕单个子元素增加布局、渲染或行为逻辑的自定义包装控件时，可以创建自己的 `Decorator` 子类。
- **简单的内边距包装器**：当你只想在子元素周围增加内边距，而不需要边框、背景或其他视觉效果时，可以直接使用 `Decorator`。

## 内置 Decorator 控件

以下控件继承自 `Decorator`：

| 控件 | 用途 |
| :--- | :--- |
| [Border](/controls/layout/containers/border) | 在子元素周围绘制边框、背景、圆角和阴影 |
| [Viewbox](/controls/layout/containers/viewbox) | 缩放其子元素以适应可用空间 |
| [LayoutTransformControl](/controls/layout/layouttransformcontrol) | 应用参与布局的渲染变换 |

## 属性

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `Child` | `Control` | 要装饰的单个子控件。它被标记为 `[Content]`，因此你可以在 XAML 中直接设置，而无需显式属性元素。 |
| `Padding` | `Thickness` | Decorator 边缘与其子元素之间的空间。 |

## `Decorator` 的工作方式

当你设置 `Child` 属性时，`Decorator` 会自动将该子元素添加到它的逻辑树和可视树中。在布局阶段，它会在应用 `Padding` 后剩余的区域内对子元素进行测量和排列。这意味着你的子类无需处理基础的单子元素布局，只需要处理你想额外叠加的渲染或测量逻辑。

由于 `Decorator` 只能接受一个子元素，因此它比 `StackPanel` 或 `Grid` 这类基于面板的容器更轻量。对于概念上是“包裹或增强单个内容”的控件，而不是组合多个子元素的场景，适合使用它。

## 示例

### 创建自定义 Decorator

下面的示例展示了一个自定义 Decorator，它会在子元素后方绘制带颜色的背景。你需要先为画刷颜色定义一个样式属性，然后重写 `Render`，在子元素绘制自己之前先绘制背景。

```csharp
public class HighlightDecorator : Decorator
{
    public static readonly StyledProperty<IBrush?> HighlightBrushProperty =
        AvaloniaProperty.Register<HighlightDecorator, IBrush?>(
            nameof(HighlightBrush), Brushes.Yellow);

    public IBrush? HighlightBrush
    {
        get => GetValue(HighlightBrushProperty);
        set => SetValue(HighlightBrushProperty, value);
    }

    public override void Render(DrawingContext context)
    {
        if (HighlightBrush is not null)
        {
            context.FillRectangle(HighlightBrush, new Rect(Bounds.Size));
        }
    }
}
```

然后，你可以在 XAML 中使用这个自定义 Decorator：

```xml
<local:HighlightDecorator HighlightBrush="LightBlue" Padding="8">
  <TextBlock Text="Highlighted content" />
</local:HighlightDecorator>
```

### 直接使用 `Decorator`

虽然不常见，但你也可以直接将 `Decorator` 作为一个简单的内边距包装器来使用：

```xml
<Decorator Padding="16">
  <TextBlock Text="Padded content" />
</Decorator>
```

它的行为类似于没有边框和背景的 `Border`，只是在子元素周围增加内边距。

## 另请参阅

- [Border](/controls/layout/containers/border)
- [Viewbox](/controls/layout/containers/viewbox)
- [LayoutTransformControl](/controls/layout/layouttransformcontrol)
- [`Decorator` API reference](/api/avalonia/controls/decorator)
- [`Decorator.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Decorator.cs)
