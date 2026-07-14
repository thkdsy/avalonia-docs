---
id: border
title: Border
description: 了解如何使用 Border 控件为 Avalonia 控件添加边框、背景、圆角和阴影。
doc-type: reference
---

`Border` 控件用于为单个子控件添加边框和背景，并且可以显示圆角。

## 常用属性

你最常使用的通常是这些属性：

<table><thead><tr><th width="261">属性</th><th>说明</th></tr></thead><tbody><tr><td><code>Background</code></td><td>背景颜色。</td></tr><tr><td><code>BorderBrush</code></td><td>边框颜色。</td></tr><tr><td><code>BorderThickness</code></td><td>边框线宽。</td></tr><tr><td><code>CornerRadius</code></td><td>为四个角统一设置圆角半径，或按[列表方式指定](#corner-radius-property)。</td></tr><tr><td><code>BoxShadow</code></td><td>[定义阴影](#box-shadows)。</td></tr><tr><td><code>BackgroundSizing</code></td><td>控制背景相对于边框的渲染方式。<br />- <code>CenterBorder</code>（默认，居中于边框厚度）<br />- <code>InnerBorderEdge</code>（填充边框内部）<br />- <code>OuterBorderEdge</code>（扩展到外边缘）</td></tr></tbody></table>

## CornerRadius 属性

如果你为 `CornerRadius` 属性提供一个单独的值，Avalonia 会将同样的圆角半径应用到子控件的四个角上。

或者，你也可以指定一个值列表。Avalonia 会按以下方式解释这些值：

- 如果提供两个值，则按 `CornerRadius="Top Bottom"` 模式解释。第一个值设置左上角和右上角，第二个值设置右下角和左下角。
- 如果提供四个值，则按 `CornerRadius="TopLeft TopRight BottomRight BottomLeft"` 模式解释。根据需要，其中一个或多个值可以为零。
- 不允许提供三个值。

### 示例

此示例使用 Border 控件在布局中创建一种“卡片/舱体”式外观：

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui">
  <Border Background="Gray"
          BorderBrush="Black"
          BorderThickness="2"
          CornerRadius="3"
          Padding="10" Margin="10">
    <TextBlock>Box 1</TextBlock>
  </Border>
  <Border Background="Gray"
          BorderBrush="Black"
          BorderThickness="2"
          CornerRadius="3"
          Padding="10" Margin="10">
    <TextBlock>Box 2</TextBlock>
  </Border>
</StackPanel>
```

</XamlPreview>

## 阴影

你可以通过设置 `BoxShadow` 属性来定义阴影。单个阴影由以下部分组成：

* 一个用于将阴影绘制为边框内部内嵌阴影的选项。若要启用，请在 `BoxShadow` 属性值中加入关键字 `inset`。
* 两个、三个或四个长度值。（见下文。）
* 一个颜色值。

Avalonia 对长度值的解释如下：

- 如果给出两个长度值，则分别解释为 `offset-x` 和 `offset-y`。
- 如果给出三个长度值，则分别解释为 `offset-x`、`offset-y` 和 `blur-radius`。
- 如果给出四个长度值，则分别解释为 `offset-x`、`offset-y`、`blur-radius` 和 `spread-radius`。

:::info
你可以通过提供一个用逗号分隔的阴影定义列表来指定多个阴影。
:::

下表按出现顺序说明了阴影的各个值：

<table><thead><tr><th width="203">值</th><th>说明</th></tr></thead><tbody><tr><td><code>inset</code></td><td>默认不指定，此时阴影是投影阴影（好像盒子浮在内容之上）。如果将 `inset` 作为 `BoxShadow` 属性的第一个值添加进去，则启用内嵌阴影，此时阴影绘制在边框内部，位于背景之上但内容之下（看起来就像内容被压进盒子里一样）。</td></tr><tr><td><code>offset-x</code> </td><td>指定水平距离。负值会将阴影放到元素左侧。</td></tr><tr><td><code>offset-y</code></td><td>指定垂直距离。负值会将阴影放到元素上方。</td></tr><tr><td><code>blur-radius</code></td><td>该值越大，模糊效果越明显，阴影会更大也更浅。不允许使用负值。如果未指定，默认值为零，阴影边缘会较锐利。</td></tr><tr><td><code>spread-radius</code></td><td>正值会让阴影向外扩张并变大，负值会让阴影收缩。如果未指定，默认值为零，此时阴影大小与元素本身相同。</td></tr><tr><td><code>color</code></td><td>阴影颜色。可以使用颜色名称（例如 `Red`）、带 # 前缀的十六进制颜色值（例如 `#dadada`），或者颜色函数（例如 `rgb(13, 110, 253)`、`hsl(215, 98%, 52%)`）。</td></tr></tbody></table>

:::info
如果两个偏移值都设为零，那么阴影会放在元素后方，并且只有在设置了 `blur-radius` 和/或 `spread-radius` 时才会产生模糊效果。
:::

### 示例

下面是一个投影阴影的示例：

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui">
  <Border Background="Gray"
          BorderBrush="Black"
          BorderThickness="2"
          CornerRadius="3"
          BoxShadow="5 5 10 0 DarkGray"
          Padding="10" Margin="40">
    <TextBlock>Box with shadow</TextBlock>
  </Border>
</StackPanel>
```

</XamlPreview>

## 另请参阅

- [Border API reference](/api/avalonia/controls/border)
- [`Border.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Border.cs)
