---
id: hit-testing
title: 命中测试
description: Avalonia 如何确定屏幕上给定坐标对应的可视元素。
doc-type: explanation
---

命中测试用于确定屏幕上某个给定点对应的是哪个可视元素。Avalonia 会在内部对指针事件使用命中测试，而你也可以在自定义控件或高级交互场景中，通过代码自行执行命中测试。

## 命中测试的工作方式

当指针事件发生时，Avalonia 会从最上层元素开始沿可视树向下遍历。对于每个元素，它都会检查该点是否落在元素边界及其实际渲染内容内。第一个通过测试的元素就会成为事件目标。

命中测试会考虑以下因素：

1. **可见性**：设置了 `IsVisible="False"` 的元素会被跳过。
2. **IsHitTestVisible**：设置了 `IsHitTestVisible="False"` 的元素会被跳过，但它们的子元素仍可能继续参与测试。
3. **边界**：该点必须落在元素的布局边界内。
4. **渲染内容**：对于形状和自定义渲染控件，测试会检查实际渲染的像素，而不仅仅是包围盒。

## IsHitTestVisible

将 `IsHitTestVisible="False"` 设置为 `False`，可让控件对指针事件“透明”。控件仍会正常渲染，但指针事件会穿透到它后面的控件：

```xml
<!-- 这个覆盖层会显示文本，但点击会穿透到下方控件 -->
<Panel>
    <Button Content="Click Me" />
    <TextBlock Text="Overlay label" IsHitTestVisible="False"
               HorizontalAlignment="Right" VerticalAlignment="Top"
               Foreground="Gray" Margin="8" />
</Panel>
```

常见用途：
- 不应拦截点击的装饰性覆盖层
- 覆盖在交互内容上的水印或状态文本
- 动画层

## Background 与命中测试

如果一个控件没有设置 `Background`（或将 `Background` 设为 `null`），那么它的空白区域不会参与命中测试。只有子内容本身能接收指针事件。

若要让一个面板的整个区域都响应指针事件，请设置 `Background="Transparent"`：

```xml
<!-- 这个面板不会在空白区域接收点击 -->
<StackPanel PointerPressed="OnPressed">
    <TextBlock Text="Only this text is clickable" />
</StackPanel>

<!-- 这个面板会在其边界内任意位置接收点击 -->
<StackPanel PointerPressed="OnPressed" Background="Transparent">
    <TextBlock Text="Click anywhere in the panel" />
</StackPanel>
```

这条规则适用于所有层级，包括 `Window` 本身。将 `Background="{x:Null}"` 与 `TransparencyLevelHint="Transparent"` 配合使用，可让指针事件穿透空白区域，传递到底层操作系统窗口。设置 `Background="Transparent"` 虽然看起来相同，但会捕获所有输入。完整示例请参阅 [透明点击穿透窗口](/docs/how-to/window-how-to#transparent-click-through-window)。

## 通过代码执行命中测试

### InputHitTest

可以在任意控件上调用 `InputHitTest`，来查找某个特定点对应的元素：

```csharp
// 坐标点相对于调用 InputHitTest 的那个控件
var result = myPanel.InputHitTest(new Point(50, 30));

if (result is Control hitControl)
{
    Debug.WriteLine($"Hit: {hitControl.GetType().Name}");
}
```

### 查找指针下方的元素

在指针事件处理程序中，事件参数的 `Source` 属性会告诉你原始命中的元素：

```csharp
private void OnPointerPressed(object? sender, PointerPressedEventArgs e)
{
    // e.Source 是被直接命中的元素
    if (e.Source is Border border)
    {
        border.Background = Brushes.Yellow;
    }
}
```

## 自定义控件中的命中测试

当你构建通过 `DrawingContext` 自行渲染内容的自定义控件时，可能需要重写命中测试逻辑，以匹配实际渲染出来的形状。

### 自定义命中测试几何区域

可以通过重写 `HitTestCore` 方法来定义自定义命中区域：

```csharp
public class CircleControl : Control
{
    public override void Render(DrawingContext context)
    {
        var radius = Math.Min(Bounds.Width, Bounds.Height) / 2;
        var center = new Point(Bounds.Width / 2, Bounds.Height / 2);
        context.DrawEllipse(Brushes.Blue, null, center, radius, radius);
    }

    protected override bool HitTest(Point point)
    {
        // 只在圆形内部执行命中测试，而不是整个外接矩形
        var radius = Math.Min(Bounds.Width, Bounds.Height) / 2;
        var center = new Point(Bounds.Width / 2, Bounds.Height / 2);
        var distance = Point.Distance(point, center);
        return distance <= radius;
    }
}
```

通过这个重写，只有当用户点击圆形内部时才会触发指针事件，而不会在外接矩形的四个角落触发。

## 命中测试顺序

当多个控件在同一点发生重叠时，命中测试会返回可视树中最上层的控件。顺序由以下因素决定：

1. **ZIndex**：`ZIndex` 值更高的元素会被优先测试。
2. **可视树顺序**：在同一面板中，后添加的子元素会渲染在前面子元素之上，因此也会优先参与命中测试。

```xml
<Panel>
    <Border Background="Red" Width="100" Height="100" />
    <!-- 这个 Border 位于上层，因此会接收到点击 -->
    <Border Background="Blue" Width="100" Height="100" Margin="30" />
</Panel>
```

## 实用模式

### 点击穿透覆盖层

创建一个不会阻挡交互的视觉覆盖层：

```xml
<Grid>
    <ListBox ItemsSource="{Binding Items}" />

    <!-- 半透明加载覆盖层；隐藏时点击可穿透 -->
    <Border Background="#80000000"
            IsVisible="{Binding IsLoading}"
            IsHitTestVisible="{Binding IsLoading}">
        <ProgressBar IsIndeterminate="True" Width="200"
                     HorizontalAlignment="Center" VerticalAlignment="Center" />
    </Border>
</Grid>
```

### 检测对画布绘制内容的点击

```csharp
private void OnCanvasPointerPressed(object? sender, PointerPressedEventArgs e)
{
    var pos = e.GetPosition((Visual)sender!);

    // 检查是否命中了你绘制的形状
    foreach (var shape in _shapes)
    {
        if (shape.Bounds.Contains(pos))
        {
            SelectShape(shape);
            e.Handled = true;
            return;
        }
    }
}
```

## 大量元素下的性能

Avalonia 的命中测试会遍历可视树，并逐个测试元素。它没有内置的空间分区结构（例如四叉树）。对于子元素数量较少的面板，这通常很快；但当 `Canvas` 或 `Panel` 上存在数百乃至数千个可交互元素时，这种线性遍历的成本就会变得明显，尤其是在指针按下事件中，因为每个候选元素都必须被测试。

### 典型症状

- 点击与 `PointerPressed` 事件触发之间存在延迟，并且该延迟会随着子元素数量线性增长。
- 延迟来自输入处理，而不是渲染或布局问题。帧率通常仍然正常。

### 应对策略

**禁用单个元素的命中测试，并使用统一覆盖层。** 在所有子元素上方放置一个透明覆盖层。让覆盖层处理 `PointerPressed`，并由你自己的逻辑决定点击到了哪个元素。将子元素的 `IsHitTestVisible` 设为 `False`，这样 Avalonia 在遍历树时就会跳过它们：

```xml
<Panel>
    <!-- 所有项目都设置为 IsHitTestVisible="False" -->
    <Canvas x:Name="ItemsCanvas" IsHitTestVisible="False">
        <!-- 数百个子控件 -->
    </Canvas>

    <!-- 透明覆盖层统一接收所有指针事件 -->
    <Border Background="Transparent" PointerPressed="OnOverlayPointerPressed" />
</Panel>
```

```csharp
private void OnOverlayPointerPressed(object? sender, PointerPressedEventArgs e)
{
    var pos = e.GetPosition(ItemsCanvas);

    // 使用你自己的空间查找逻辑，找出该位置对应的项目
    var item = FindItemAt(pos);
    if (item != null)
    {
        SelectItem(item);
        e.Handled = true;
    }
}
```

你的 `FindItemAt` 方法可以采用任何适合当前数据结构的查找策略。对于规则网格布局，简单的坐标计算就可能足够；对于不规则形状，则可以考虑四叉树或 R-tree 这类空间索引。

**改用自定义渲染。** 不要为每个元素创建单独控件，而是把所有元素都放到一个控件的 `Render` 重写中统一绘制。这样就完全消除了逐元素命中测试，因为参与命中测试的只剩下这一个父控件。随后你只需在该控件上处理指针事件，并根据指针位置判断点击到了哪个逻辑元素。详见 [自定义渲染](/docs/graphics-animation/custom-rendering)。

**减少可参与命中测试的元素数量。** 如果只有一部分元素需要交互，那么其余元素应设置 `IsHitTestVisible="False"`。例如，在流程图编辑器中，可以把背景网格线和标签排除在命中测试之外，只保留可拖动节点参与交互。

## 另请参阅

- [指针输入](/docs/input-interaction/pointer)：指针事件与位置。
- [自定义渲染](/docs/graphics-animation/custom-rendering)：使用 DrawingContext 进行绘制。
- [形状与几何图形](/docs/graphics-animation/shapes-and-geometries)：用于命中区域的几何类型。
- [性能优化](/docs/app-development/performance)：通用性能建议。
