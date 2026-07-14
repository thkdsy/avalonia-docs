---
id: composition-animations
title: 合成动画
description: 在 Avalonia 中使用合成动画 API，通过代码驱动运行在渲染线程上的动画。
doc-type: explanation
---

合成动画是一种由代码驱动、直接运行在渲染线程上的动画系统。它能对视觉属性提供细粒度控制，并在不阻塞 UI 线程的情况下带来平滑且高性能的动画效果。

这个 API 与 UWP/WinUI 的 Composition 层很相似。当你需要通过代码控制动画、追求渲染线程级性能，或者需要动画化 XAML 关键帧动画不支持的属性时，就应该使用它。

## 何时使用合成动画

| | 关键帧动画 | 控件过渡 | 合成动画 |
|---|---|---|---|
| **定义位置** | XAML | XAML | C# 代码 |
| **触发方式** | 样式选择器 | 属性值变化 | `StartAnimation()` 或隐式属性变化 |
| **运行线程** | UI 线程 | UI 线程 | 渲染线程 |
| **最适合** | 多步骤、由样式驱动的动画 | 属性变化时的平滑反馈 | 对性能敏感或需要程序化控制的动画 |

在以下场景中，适合选择合成动画：

- 需要通过代码驱动视觉对象动画（例如响应滚动位置、手势或数据变化）
- 希望让动画运行在渲染线程上，以获得最佳流畅度
- 需要使用基于表达式或基于物理的动画逻辑

如果是样式驱动的场景，那么 [关键帧动画](/docs/graphics-animation/keyframe-animations) 和 [控件过渡](/docs/graphics-animation/control-transitions) 会更简单，也更合适。

## 核心概念

### CompositionVisual 与 Compositor

每个 Avalonia 控件在渲染线程上都对应一个 [`CompositionVisual`](/api/avalonia/rendering/composition/compositionvisual)。你可以通过 `ElementComposition.GetElementVisual()` 获取它：

```csharp
var visual = ElementComposition.GetElementVisual(myControl);
```

[`Compositor`](/api/avalonia/rendering/composition/compositor) 是用于创建动画和动画集合的工厂对象。你可以从任意 `CompositionVisual` 获取它：

```csharp
var compositor = visual.Compositor;
```

所有动画对象都必须通过与目标 visual 关联的 `Compositor` 来创建。

### 可动画化属性

`CompositionVisual` 上的以下属性可以被动画化：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Offset` | `Vector3D` | visual 在 X、Y、Z 三个方向上的位置偏移。 |
| `Opacity` | `float` | visual 的不透明度（0.0 到 1.0）。 |
| `Size` | `Vector` | visual 的宽度和高度。 |

## 显式动画

显式动画会在你对某个 visual 调用 `StartAnimation()` 时执行。你需要手动定义关键帧、设置持续时间，并启动动画。

### 滑入示例

这个示例会让一个控件在 400 毫秒内从左侧滑入：

```csharp
var visual = ElementComposition.GetElementVisual(myControl);
var compositor = visual.Compositor;

var animation = compositor.CreateVector3KeyFrameAnimation();
animation.Duration = TimeSpan.FromMilliseconds(400);
animation.InsertKeyFrame(0f, new Vector3D(-200, 0, 0));
animation.InsertKeyFrame(1f, new Vector3D(0, 0, 0));

visual.StartAnimation("Offset", animation);
```

关键帧的进度值范围是从 `0f`（开始）到 `1f`（结束）。如果需要多步骤动画，可以在 0 到 1 之间插入任意中间关键帧。

### 淡入示例

```csharp
var visual = ElementComposition.GetElementVisual(myControl);
var compositor = visual.Compositor;

var animation = compositor.CreateScalarKeyFrameAnimation();
animation.Duration = TimeSpan.FromMilliseconds(300);
animation.InsertKeyFrame(0f, 0f);
animation.InsertKeyFrame(1f, 1f);

visual.StartAnimation("Opacity", animation);
```

## 隐式动画

隐式动画会在某个已映射属性发生变化时自动触发。你不需要手动调用 `StartAnimation()`，而是把一个 `ImplicitAnimationCollection` 赋给 visual。之后只要集合中映射的属性发生变化，对应动画就会自动运行。

### 平滑重定位示例

这个示例会在控件 `Offset` 变化时，平滑地为位置变化添加动画：

```csharp
var visual = ElementComposition.GetElementVisual(myControl);
var compositor = visual.Compositor;

var offsetAnimation = compositor.CreateVector3KeyFrameAnimation();
offsetAnimation.Duration = TimeSpan.FromMilliseconds(300);
offsetAnimation.Target = "Offset";
offsetAnimation.InsertExpressionKeyFrame(1f, "this.FinalValue");

var implicitAnimations = compositor.CreateImplicitAnimationCollection();
implicitAnimations["Offset"] = offsetAnimation;

visual.ImplicitAnimations = implicitAnimations;
```

表达式关键帧 `"this.FinalValue"` 的含义是：让动画从当前值插值到该属性被设置后的最终值。这样无论起点和终点位置是什么，动画都可以复用。

你也可以在同一个集合中映射多个属性：

```csharp
var opacityAnimation = compositor.CreateScalarKeyFrameAnimation();
opacityAnimation.Duration = TimeSpan.FromMilliseconds(200);
opacityAnimation.Target = "Opacity";
opacityAnimation.InsertExpressionKeyFrame(1f, "this.FinalValue");

implicitAnimations["Opacity"] = opacityAnimation;
```

## 通过附加属性与 XAML 集成

如果你想以声明式方式使用合成动画，可以把初始化逻辑封装在附加属性中。这样就能通过 XAML 样式来应用合成动画行为。

### 附加属性

```csharp
public class CompositionAnimationHelper : AvaloniaObject
{
    public static readonly AttachedProperty<bool> SmoothOffsetProperty =
        AvaloniaProperty.RegisterAttached<CompositionAnimationHelper, Visual, bool>(
            "SmoothOffset");

    public static bool GetSmoothOffset(Visual element) =>
        element.GetValue(SmoothOffsetProperty);

    public static void SetSmoothOffset(Visual element, bool value) =>
        element.SetValue(SmoothOffsetProperty, value);

    static CompositionAnimationHelper()
    {
        SmoothOffsetProperty.Changed.AddClassHandler<Visual>((element, args) =>
        {
            if (args.NewValue is true)
            {
                element.AttachedToVisualTree += (_, _) =>
                {
                    var visual = ElementComposition.GetElementVisual(element);
                    if (visual == null) return;
                    var compositor = visual.Compositor;

                    var animation = compositor.CreateVector3KeyFrameAnimation();
                    animation.Duration = TimeSpan.FromMilliseconds(300);
                    animation.Target = "Offset";
                    animation.InsertExpressionKeyFrame(1f, "this.FinalValue");

                    var implicit = compositor.CreateImplicitAnimationCollection();
                    implicit["Offset"] = animation;
                    visual.ImplicitAnimations = implicit;
                };
            }
        });
    }
}
```

### XAML 用法

```xml
<Style Selector="ListBoxItem">
    <Setter Property="local:CompositionAnimationHelper.SmoothOffset" Value="True" />
</Style>
```

这样一来，每个 `ListBoxItem` 在列表重排，或项目被添加/移除时，都会平滑地移动到新位置。

## API 参考

| 类型 / 成员 | 说明 |
|---|---|
| `ElementComposition.GetElementVisual(Visual)` | 返回某个控件对应的 `CompositionVisual`。 |
| `CompositionVisual` | 表示控件在渲染线程上的视觉对象。 |
| `Compositor` | 用于创建动画和动画集合对象的工厂。 |
| `CreateScalarKeyFrameAnimation()` | 创建用于 `float` 属性（例如 `Opacity`）的动画。 |
| `CreateVector3KeyFrameAnimation()` | 创建用于 `Vector3D` 属性（例如 `Offset`）的动画。 |
| `InsertKeyFrame(float progress, T value)` | 在指定进度（0.0 到 1.0）插入关键帧。 |
| `InsertExpressionKeyFrame(float progress, string expression)` | 插入一个基于表达式的关键帧（例如 `"this.FinalValue"`）。 |
| `StartAnimation(string property, CompositionAnimation animation)` | 在指定属性上启动一个显式动画。 |
| `ImplicitAnimationCollection` | 把属性名映射到在属性变化时自动运行的动画。 |
| `CompositionVisual.ImplicitAnimations` | 获取或设置某个 visual 的隐式动画集合。 |

## 另请参阅

- [关键帧动画](/docs/graphics-animation/keyframe-animations)
- [控件过渡](/docs/graphics-animation/control-transitions)
- [自定义渲染](/docs/graphics-animation/custom-rendering)
