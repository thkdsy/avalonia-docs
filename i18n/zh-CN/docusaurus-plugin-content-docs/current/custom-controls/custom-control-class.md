---
id: custom-control-class
title: 添加自定义控件类
description: 在 Avalonia 项目中使用 IDE 工具或手动方式创建新的自定义控件类文件。
doc-type: how-to
---

Avalonia 中的自定义控件通过继承合适的基类来构建。如果你需要完全控制渲染过程，请继承 `Control` 并直接使用 [`DrawingContext`](/api/avalonia/media/drawingcontext) 进行绘制。如果你希望控件外观由 `ControlTemplate` 来定义（即无外观控件），则应改为继承 `TemplatedControl`。

## 选择基类

| 基类 | 适用场景 |
|---|---|
| `Control` | 你想使用 `DrawingContext` 渲染内容（自定义绘制）。 |
| `TemplatedControl` | 你希望外观由 `ControlTemplate` 定义。 |
| `ContentControl` | 你的控件承载单个内容项。 |
| `HeaderedContentControl` | 你的控件具有标题和内容区域。 |
| `ItemsControl` | 你的控件用于显示一个项目集合。 |

这些类型最终都继承自 `Control`，因此像 `Width`、`Height`、`Margin` 和 `DataContext` 这样的属性始终可用。

## 创建自绘控件

自绘控件继承自 `Control`，并通过重写 `Render` 方法使用 `DrawingContext` 进行绘制。你也可以重写 `MeasureOverride` 和 `ArrangeOverride` 来参与布局，并报告控件期望的大小。

下面的示例创建了一个简单的圆形控件，并提供一个可配置的 `Fill` 属性：

```csharp
using System;
using Avalonia;
using Avalonia.Controls;
using Avalonia.Media;

namespace AvaloniaCCExample.CustomControls
{
    public class CircleControl : Control
    {
        public static readonly StyledProperty<IBrush> FillProperty =
            AvaloniaProperty.Register<CircleControl, IBrush>(nameof(Fill), Brushes.Blue);

        public IBrush Fill
        {
            get => GetValue(FillProperty);
            set => SetValue(FillProperty, value);
        }

        static CircleControl()
        {
            AffectsRender<CircleControl>(FillProperty);
        }

        public override void Render(DrawingContext context)
        {
            var radius = Math.Min(Bounds.Width, Bounds.Height) / 2;
            var center = new Point(Bounds.Width / 2, Bounds.Height / 2);
            context.DrawEllipse(Fill, null, center, radius, radius);
        }
    }
}
```

关键点：

- **`FillProperty`** 是一个样式属性，因此它可以在 XAML 中设置、绑定到数据，并被样式匹配。
- 静态构造函数调用了 `AffectsRender`，这会告诉 Avalonia 在 `Fill` 发生变化时重新绘制控件。
- **`Render`** 会接收一个 `DrawingContext`，其中提供了 `DrawEllipse`、`DrawRectangle`、`DrawLine` 和 `DrawText` 等方法。

## 在 XAML 中使用

若要在 XAML 中使用自定义控件，请添加一个映射到控件所在 CLR 命名空间的 XML 命名空间。然后通过控件类名来引用它。

```xml title='XAML'
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:cc="using:AvaloniaCCExample.CustomControls"
        x:Class="AvaloniaCCExample.MainWindow"
        Title="Avalonia Custom Control">
  <cc:CircleControl Height="200" Width="200" Fill="Red" />
</Window>
```

`cc` 前缀可以任意命名。它只是将 `AvaloniaCCExample.CustomControls` 映射出来，以便 XAML 解析器能够解析 `CircleControl`。

:::info
如果你的控件位于独立的类库中，请参阅[自定义控件库](/docs/custom-controls/custom-control-library)了解额外的设置步骤。
:::

## 使渲染失效

Avalonia 提供了多种机制，用于通知布局和渲染系统某个控件需要更新。

### AffectsRender、AffectsMeasure 和 AffectsArrange

可以在控件的静态构造函数中调用这些静态辅助方法，声明哪些属性会触发渲染管线中的哪些阶段：

```csharp
static CircleControl()
{
    AffectsRender<CircleControl>(FillProperty);
    AffectsMeasure<CircleControl>(SomeOtherProperty);
    AffectsArrange<CircleControl>(YetAnotherProperty);
}
```

- **`AffectsRender`** 会在属性变化时触发重绘（再次调用 `Render`）。
- **`AffectsMeasure`** 会触发新的测量过程，适用于属性变化会影响控件期望大小的情况。
- **`AffectsArrange`** 会触发新的排列过程，适用于属性变化会影响控件内容定位方式的情况。

### 手动失效

如果你需要因属性变化以外的原因触发重绘（例如定时器 tick），可以在控件实例上调用 `InvalidateVisual()`。这会为该控件安排一次新的渲染过程。

```csharp
// 通过代码强制重绘
InvalidateVisual();
```

请谨慎使用手动失效。更推荐通过 `AffectsRender` 声明属性依赖关系，因为这样可以让失效机制保持自动且可预测。

## 另请参阅

- [模板控件](/docs/custom-controls/templated-controls)
- [绘制自定义控件](/docs/custom-controls/drawing-custom-controls)
- [定义属性](/docs/custom-controls/defining-properties)
