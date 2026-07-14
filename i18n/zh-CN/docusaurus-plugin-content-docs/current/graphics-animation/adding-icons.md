---
id: adding-icons
title: 添加图标
description: 使用图像文件、图标字体或路径图标为 Avalonia 应用添加图标。
doc-type: how-to
---

图标能够为操作或内容提供可视化表示，使应用更易于理解和导航。Avalonia 支持图像文件、图标字体和路径图标。

## UI 图标

在 Avalonia 中，在用户界面中使用图标可以改善应用外观，并提升易用性。图标能够为操作或内容提供直观表达，帮助用户更容易理解应用功能。

你可以通过多种方式为 Avalonia 应用添加图标。本指南将介绍三种常见方式：使用图像文件、使用图标字体以及使用路径图标。

### 使用图像文件
在 Avalonia 中使用图标的一种方式是使用图像文件。你可以使用 PNG、JPG 或 BMP 等格式。下面是一个将图像文件用作图标的示例：

```xml
<Image Width="16" Height="16" Source="avares://MyApp/Assets/icon.png" />
```

在这个示例中，使用 [`Image`](/api/avalonia/controls/image) 控件将应用资源中的图片显示为图标。`Image` 控件的 `Source` 属性被设置为指向该图像文件的资源 URI。

### 使用图标字体

在 Avalonia 中使用图标的另一种方式是使用图标字体。图标字体允许你使用可缩放的矢量图标，并能像 CSS 一样在大小、颜色和阴影方面进行定制。下面是一个在 Avalonia 中使用图标字体的示例：

```xml
<TextBlock FontFamily="avares://MyApp/Assets/#FontAwesome" Text="&#xf030;" />
```

在这个示例中，使用 `TextBlock` 控件显示 `FontAwesome` 图标字体中的一个图标。`TextBlock` 的 `FontFamily` 属性被设置为指向字体文件的资源 URI，而 `Text` 属性则设置为目标图标的 Unicode 值。

### 使用路径图标

路径图标可以根据 `Geometry` 绘制图标，其中包括使用来自可缩放矢量图形（SVG）格式的路径，并且可以自定义大小和颜色。有关如何使用该控件，请参阅[参考文档](/controls/media/pathicon)。

### 最佳实践

虽然使用图标能够提升应用的可用性，但仍然需要合理使用。使用图标时请记住以下建议：

* 确保图标大小合适，并且在背景上清晰可见。
* 对常见操作使用普遍认可的图标，使应用更加直观。

## 菜单图标

`MenuItem.Icon` 属性用于为菜单项设置图标。你可以为该图标使用不同类型的图像来源，包括资源 URI、文件路径或网页 URL。下面是一个为菜单项添加图标的示例：

```xml
<Menu>
  <MenuItem Header="File">
    <MenuItem Header="Open" Command="{Binding OpenCommand}">
      <MenuItem.Icon>
        <Image Width="16" Height="16" Source="avares://MyApp/Assets/open_icon.png" />
      </MenuItem.Icon>
    </MenuItem>
  </MenuItem>
</Menu>
```

在这个示例中，`MenuItem.Icon` 属性被设置为一个 `Image` 控件，它会显示来自应用资源的图像。`Image` 控件的 `Source` 属性设置为表示图像来源的资源 URI，而 `Width` 和 `Height` 属性则用于控制图像尺寸。

## 另请参阅

- [形状与几何图形](/docs/graphics-animation/shapes-and-geometries)：用于路径图标的几何类型。
- [绘制图形](/docs/graphics-animation/drawing-graphics)：Avalonia 图形系统概览。
