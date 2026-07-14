---
id: image-interpolation
title: 图像插值
description: 在 Avalonia 中控制缩放图像时的插值质量。
doc-type: how-to
---

在 Avalonia 中显示图像时，尤其是当图像被缩放到与其原始分辨率不同的尺寸时，渲染质量会取决于所使用的插值模式。本指南将说明如何在 Avalonia 应用中控制图像插值。

## 默认行为

从 Avalonia 11 开始，默认插值模式被设置为 `LowQuality`。这个设置优先考虑性能，但在缩放图像时，尤其是当图像显示尺寸明显小于原始尺寸时，可能会导致渲染不够平滑。

## 插值模式

Avalonia 支持以下位图插值模式：

| 模式 | 说明 |
| :--- | :--- |
| `None` | 不做插值，像素会在没有平滑处理的情况下直接渲染 |
| `LowQuality` | 基础插值（默认），优先考虑性能 |
| `MediumQuality` | 在速度与质量之间做平衡 |
| `HighQuality` | 更平滑的插值，特别适合缩小图像 |

## 设置插值模式

### 针对单个控件设置

你可以通过 `RenderOptions.BitmapInterpolationMode` 附加属性，为单个控件设置插值模式：

```xml
<Image Source="assets/myimage.png" 
       RenderOptions.BitmapInterpolationMode="HighQuality" />
```

这同样可以应用到容器控件上：

```xml
<Border RenderOptions.BitmapInterpolationMode="HighQuality">
    <Image Source="assets/myimage.png" />
</Border>
```

### 常见使用场景

1. **图标显示**：当图标被缩小显示时，使用 `HighQuality` 插值可以减少锯齿边缘：
```xml
<Button>
    <Image Source="assets/icon.png" 
           Width="16" 
           Height="16"
           RenderOptions.BitmapInterpolationMode="HighQuality" />
</Button>
```

2. **图片画廊**：如果图片显示质量很重要，可以这样设置：
```xml
<ItemsControl RenderOptions.BitmapInterpolationMode="HighQuality">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Image Source="{Binding ImagePath}" />
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

## 边缘模式（抗锯齿）

默认情况下，Avalonia 会对图像应用抗锯齿处理，因此当图像被旋转、缩放，或者处于子像素偏移位置时，会得到平滑边缘。这由 `RenderOptions.EdgeMode` 附加属性控制。

如果你希望某个特定控件使用非抗锯齿的边缘效果（更锐利、像素感更强），可将 `EdgeMode` 设置为 `Aliased`：

```xml
<!-- 平滑边缘（默认） -->
<Image Source="assets/photo.png"
       RenderTransform="rotate(15)" />

<!-- 锐利边缘（无抗锯齿） -->
<Image Source="assets/sprite.png"
       RenderOptions.EdgeMode="Aliased"
       RenderTransform="rotate(15)" />
```

| 模式 | 说明 |
|---|---|
| `Unspecified` | 渲染器使用默认行为（启用抗锯齿）。 |
| `Aliased` | 禁用抗锯齿。适合像素风图像，或需要锐利、非平滑边缘的场景。 |

`EdgeMode` 也会影响非图像内容的渲染（例如形状、边框）。你可以把它设置在父元素上，以应用到所有子元素：

```xml
<Border RenderOptions.EdgeMode="Aliased">
    <!-- 内部所有内容都会以无抗锯齿方式渲染 -->
</Border>
```

## 性能注意事项

从设计上看，插值模式按控件分别设置，是出于性能考虑。更高质量的插值需要更多计算资源，因此可以参考以下建议：

- 对以下场景使用 `HighQuality`：
  - 重要的 UI 元素，例如 logo
  - 被缩小显示且对质量要求较高的图像
  - 图片画廊或以图像为核心的界面
  
- 对以下场景使用默认的 `LowQuality`：
  - 背景图像
  - 对质量要求没那么高的装饰元素
  - 对性能敏感的应用

## 创建全局设置

虽然 Avalonia 没有提供内置的全局插值模式设置方式，但你可以通过自定义附加属性或行为，在整个应用中统一管理。下面是一个示例思路：

```csharp
public static class GlobalImageOptions
{
    public static readonly AttachedProperty<BitmapInterpolationMode> InterpolationModeProperty =
        AvaloniaProperty.RegisterAttached<Image, BitmapInterpolationMode>(
            "InterpolationMode",
            typeof(GlobalImageOptions),
            defaultValue: BitmapInterpolationMode.HighQuality);

    public static void SetInterpolationMode(Image image, BitmapInterpolationMode value)
    {
        image.SetValue(RenderOptions.BitmapInterpolationModeProperty, value);
    }
}
```

然后在 XAML 中这样使用：

```xml
<Style Selector="Image">
    <Setter Property="(local:GlobalImageOptions.InterpolationMode)"
            Value="HighQuality" />
</Style>
```

## 最佳实践建议

1. **资源准备**：
   - 为目标显示尺寸准备合适分辨率的图像
   - 对重要资源考虑提供多个分辨率版本
   - 如果可能，使用矢量格式（SVG）来实现与分辨率无关的图形显示

2. **布局注意事项**：
   - 注意图像原始尺寸与显示尺寸之间的关系
   - 使用合适的容器和布局面板来管理图像缩放
   - 可考虑将 `UniformToFill` 或 `Uniform` 拉伸模式与高质量插值配合使用

3. **测试**：
   - 在不同屏幕密度下测试图像渲染效果
   - 当大量图像使用高质量插值时，验证其性能影响
   - 检查不同插值设置下的内存占用情况

## 另请参阅

- [文本选项](/docs/graphics-animation/text-options)：通过 `TextOptions` 控制文本渲染质量。
