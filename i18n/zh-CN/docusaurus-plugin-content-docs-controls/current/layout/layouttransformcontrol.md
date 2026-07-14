---
id: layouttransformcontrol
title: LayoutTransformControl
description: 一个装饰器控件，可对单个子元素应用参与布局的变换（如旋转、缩放和倾斜），从而让父面板围绕变换后的边界重新测量和排列。
doc-type: reference
---

[`LayoutTransformControl`](/api/avalonia/controls/layouttransformcontrol) 会对其子元素应用参与布局的变换（旋转、缩放、倾斜）。与只改变控件绘制方式而不影响周围布局的 `RenderTransform` 不同，[`LayoutTransformControl`](/api/avalonia/controls/layouttransformcontrol) 会让父面板围绕变换后的边界重新测量和排列。

这意味着，一个被旋转的控件会正确地将相邻控件挤开，而一个被缩放的控件也会在 `StackPanel` 或 `Grid` 中占据合适的空间。

## 常用属性

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `LayoutTransform` | `ITransform` | 在布局过程中应用的变换。支持 `RotateTransform`、`ScaleTransform`、`SkewTransform`、`TransformGroup` 和 `MatrixTransform`。 |
| `UseRenderTransform` | `bool` | 当为 `true` 时，通过 `RenderTransform` 应用变换，而不是执行单独的布局过程。默认值为 `false`。 |
| `Child` | `Control` | 要进行变换的子控件（继承自 `Decorator`）。 |
| `Padding` | `Thickness` | 子控件周围的内边距（继承自 `Decorator`）。 |

## `LayoutTransform` 与 `RenderTransform` 的区别

| | `LayoutTransformControl` | `RenderTransform` |
| :--- | :--- | :--- |
| 是否影响布局 | 是，兄弟元素会围绕变换后的边界重新流式布局 | 否，兄弟元素会忽略该变换 |
| 性能 | 变换变化时会重新测量和重新排列 | 更轻量，通常可利用 GPU 加速 |
| 适用场景 | 周围内容必须感知变换后的尺寸 | 只做动画或视觉调整，而不影响布局 |

## 示例

### 旋转控件

此示例将一个按钮旋转 45 度。父级 `StackPanel` 会为旋转后的边界分配空间，因此下面的文本不会被覆盖：

<XamlPreview>

```xml title="XAML"
<StackPanel Spacing="8" HorizontalAlignment="Center" xmlns="https://github.com/avaloniaui">
  <LayoutTransformControl>
    <LayoutTransformControl.LayoutTransform>
      <RotateTransform Angle="45" />
    </LayoutTransformControl.LayoutTransform>
    <Button Content="Rotated 45°" />
  </LayoutTransformControl>
  <TextBlock Text="This text is positioned below the rotated button." />
</StackPanel>
```

</XamlPreview>

### 缩放控件

你可以将控件缩放到两倍大小，同时保持布局正确：

```xml title="XAML"
<LayoutTransformControl>
  <LayoutTransformControl.LayoutTransform>
    <ScaleTransform ScaleX="2" ScaleY="2" />
  </LayoutTransformControl.LayoutTransform>
  <TextBlock Text="Double size" />
</LayoutTransformControl>
```

### 组合变换

使用 `TransformGroup` 可同时应用多个变换：

```xml title="XAML"
<LayoutTransformControl>
  <LayoutTransformControl.LayoutTransform>
    <TransformGroup>
      <ScaleTransform ScaleX="1.5" ScaleY="1.5" />
      <RotateTransform Angle="30" />
    </TransformGroup>
  </LayoutTransformControl.LayoutTransform>
  <Border Background="LightBlue" Padding="12">
    <TextBlock Text="Scaled and rotated" />
  </Border>
</LayoutTransformControl>
```

### 绑定旋转角度

你可以将旋转角度绑定到滑块，以实现交互式控制：

```xml title="XAML"
<StackPanel Spacing="12">
  <Slider x:Name="AngleSlider" Minimum="0" Maximum="360" Value="0" />
  <LayoutTransformControl HorizontalAlignment="Center">
    <LayoutTransformControl.LayoutTransform>
      <RotateTransform Angle="{Binding #AngleSlider.Value}" />
    </LayoutTransformControl.LayoutTransform>
    <Border Background="LightCoral" Padding="16">
      <TextBlock Text="Drag the slider to rotate" />
    </Border>
  </LayoutTransformControl>
</StackPanel>
```

## 实用说明

- **性能**：由于 `LayoutTransformControl` 每次变换变化时都会触发完整的测量和排列过程，因此应避免高频率地为其 `LayoutTransform` 做动画。如果你需要平滑的逐帧动画（例如旋转中的图标），请改用 `RenderTransform`。
- **嵌套**：你可以在一个 `LayoutTransformControl` 中再嵌套另一个 `LayoutTransformControl`。每一个控件都会独立测量其子元素，因此变换会沿布局树向外层层叠加。
- **`UseRenderTransform`**：将该属性设为 `true` 时，会通过 `RenderTransform` 应用变换，而不是执行单独的布局过程。当你只是希望使用 `LayoutTransform` 的声明语法便利性，但不需要周围控件重新流式布局时，这会很有用。
- **裁剪**：会裁剪其子元素的父容器（例如设置了 `ClipToBounds="True"` 的 `Border`）可能会裁掉变换后的边界。请确保父容器拥有足够空间来显示完整的变换区域。

## 另请参阅

- [Decorator](/controls/layout/decorator)
- [Border](/controls/layout/containers/border)
- [Viewbox](/controls/layout/containers/viewbox)
- [变换](/docs/graphics-animation/transforms)
- [LayoutTransformControl API 参考](/api/avalonia/controls/layouttransformcontrol)
- [`LayoutTransformControl.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/LayoutTransformControl.cs)
