---
id: drawingimage
title: DrawingImage
description: 一个使用 Avalonia Drawing 对象将矢量图形渲染为 IImage 的控件，可实现完全由 XAML 定义、与分辨率无关的图标和图形。
doc-type: reference
---

[`DrawingImage`](/api/avalonia/media/drawingimage) 会将矢量图形渲染为 `IImage`，因此它可以像位图一样在任何支持图像的地方使用。它不是从文件中加载像素，而是通过 Avalonia 的 [`Drawing`](/api/avalonia/media/drawing) 类来绘制形状、路径和其他矢量内容。

当你需要与分辨率无关、可无损缩放的图标或图形，或者希望完全用 XAML 定义图像而不依赖外部资源文件时，这种方式会非常有用。

## Drawing 类型

`DrawingImage` 会在其 `Drawing` 属性中包裹一个 `Drawing` 对象。Avalonia 提供了四种具体的 Drawing 类型：

| 类型 | 用途 |
| :--- | :--- |
| `GeometryDrawing` | 填充和/或描边一个 `Geometry` 形状 |
| `ImageDrawing` | 在一个矩形区域内渲染位图图像 |
| `GlyphRunDrawing` | 使用前景画刷渲染一组字形 |
| [`DrawingGroup`](/api/avalonia/media/drawinggroup) | 将多个 Drawing 组合为一个，并可附加变换、裁剪和不透明度 |

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `Drawing` | `Drawing` | 要渲染的矢量绘图内容 |
| `Viewbox` | `Rect` | 要显示的绘图矩形区域，单位为设备无关像素 |

## 示例

### 简单矢量图标

此示例使用 `GeometryDrawing` 创建一个带深绿色边框的绿色圆形：

```xml title="XAML"
<Image Width="64" Height="64">
  <Image.Source>
    <DrawingImage>
      <GeometryDrawing Brush="Green" Geometry="M 32,0 A 32,32 0 1 1 32,64 A 32,32 0 1 1 32,0 Z">
        <GeometryDrawing.Pen>
          <Pen Brush="DarkGreen" Thickness="2" />
        </GeometryDrawing.Pen>
      </GeometryDrawing>
    </DrawingImage>
  </Image.Source>
</Image>
```

### 组合多个绘图

使用 `DrawingGroup` 可以将多个形状组合成一张图像。这个示例绘制了一个简单的房屋图标：

```xml title="XAML"
<Image Width="100" Height="100">
  <Image.Source>
    <DrawingImage>
      <DrawingGroup>
        <!-- 屋顶 -->
        <GeometryDrawing Brush="Brown" Geometry="M 10,50 L 50,10 L 90,50 Z" />
        <!-- 墙体 -->
        <GeometryDrawing Brush="Beige" Geometry="M 20,50 L 20,90 L 80,90 L 80,50 Z">
          <GeometryDrawing.Pen>
            <Pen Brush="Gray" Thickness="1" />
          </GeometryDrawing.Pen>
        </GeometryDrawing>
        <!-- 门 -->
        <GeometryDrawing Brush="SaddleBrown" Geometry="M 40,60 L 40,90 L 60,90 L 60,60 Z" />
      </DrawingGroup>
    </DrawingImage>
  </Image.Source>
</Image>
```

### 作为资源使用

你可以将 `DrawingImage` 定义为资源，并在整个应用中重复引用。这种方式可以把图标定义集中在一个位置，并在多个控件中复用：

```xml title="XAML"
<UserControl.Resources>
  <DrawingImage x:Key="CheckIcon">
    <GeometryDrawing Brush="Green" Geometry="M 2,5 L 4,7 L 8,3" >
      <GeometryDrawing.Pen>
        <Pen Brush="Green" Thickness="1" LineCap="Round" LineJoin="Round" />
      </GeometryDrawing.Pen>
    </GeometryDrawing>
  </DrawingImage>
</UserControl.Resources>

<Image Source="{StaticResource CheckIcon}" Width="24" Height="24" />
```

### `DrawingImage` 与位图图像的区别

在以下场景下，适合使用 `DrawingImage`：

- 与分辨率无关、可在任意尺寸下清晰缩放的图形
- 完全用 XAML 定义而不依赖外部文件的图标
- 画刷或几何数据可以绑定到数据上的动态图形

如果你的内容是照片或预先渲染好的图稿，则更适合使用位图图像（例如使用资源路径的 `Image.Source`）。

## 实用说明

- **Viewbox 裁剪。** 如果设置了 `Viewbox` 属性，则只会渲染绘图中指定的矩形区域。当你把多个图标打包进一个 `DrawingGroup` 并希望每次只显示其中一个区域时，这会很方便。
- **性能。** 由于 `DrawingImage` 每次绘制时都要重新渲染其矢量内容，因此当绘图非常复杂、包含数百个几何图形时，性能可能会比等效位图更差。对于复杂图稿，可考虑预先渲染为 `RenderTargetBitmap`。
- **数据绑定。** 你可以将绘图中的 `Brush`、`Geometry` 或 `Pen` 属性绑定到视图模型的值，从而得到会随着应用状态变化而响应的动态图形。
- **无障碍。** `DrawingImage` 本身不会向辅助技术暴露文本内容。如果图形本身承载语义，请在父级 `Image` 控件上设置可访问名称或描述。

## 另请参阅

- [Image](/controls/media/image)
- [PathIcon](/controls/media/pathicon)
- [画刷](/docs/graphics-animation/brushes)
- [DrawingImage API 参考](/api/avalonia/media/drawingimage)
- [GeometryDrawing API 参考](/api/avalonia/media/geometrydrawing)
- [DrawingGroup API 参考](/api/avalonia/media/drawinggroup)
- [`DrawingImage.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Media/DrawingImage.cs)
