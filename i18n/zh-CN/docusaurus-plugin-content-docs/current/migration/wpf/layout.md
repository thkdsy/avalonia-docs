---
id: layout
title: 布局
description: WPF 与 Avalonia 在面板、尺寸和定位方面的布局系统差异。
doc-type: migration
---

Avalonia 的布局系统与 WPF 非常相似。如果你已经熟悉 WPF 的面板和布局概念，那么在 Avalonia 中会很容易上手。不过，仍然有一些值得注意的关键差异和新增能力。

## 面板类型

| WPF | Avalonia | 说明 |
|---|---|---|
| [`StackPanel`](/api/avalonia/controls/stackpanel) | `StackPanel` | 相同。Avalonia 额外提供了 `Spacing` 属性。 |
| [`Grid`](/api/avalonia/controls/grid) | `Grid` | 相同。支持 `ColumnDefinitions="Auto,*"` 这种简写。 |
| `DockPanel` | `DockPanel` | 相同。`LastChildFill` 默认值为 `true`。 |
| `WrapPanel` | `WrapPanel` | 相同。 |
| `Canvas` | `Canvas` | 相同。 |
| `UniformGrid` | `UniformGrid` | 相同。 |
| `VirtualizingStackPanel` | `VirtualizingStackPanel` | 概念相同。 |

## Grid 简写语法

Avalonia 支持为 Grid 的行和列使用内联定义字符串，从而让 XAML 更简洁：

```xml
<!-- WPF 的详细写法 -->
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" />
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="200" />
    </Grid.ColumnDefinitions>
</Grid>

<!-- Avalonia 简写（在 WPF .NET 6+ 中也可用） -->
<Grid ColumnDefinitions="Auto,*,200" RowDefinitions="Auto,*" />
```

## 用于叠层时的 Panel 与 Grid

Avalonia 提供了一个轻量级的 `Panel` 控件，可用于将子元素彼此叠放。在 WPF 中，开发者常常使用一个没有定义行列的 `Grid` 来实现重叠内容；而在 Avalonia 中，建议优先使用 `Panel`，因为它可以避免 `Grid` 布局引擎带来的额外开销。

```xml
<!-- WPF 中的叠层写法 -->
<Grid>
    <Image Source="background.png" />
    <TextBlock Text="Overlay" />
</Grid>

<!-- Avalonia 推荐写法 -->
<Panel>
    <Image Source="background.png" />
    <TextBlock Text="Overlay" />
</Panel>
```

## Spacing 属性

Avalonia 的 `StackPanel` 提供了 `Spacing` 属性，这样你就不必再为每个子元素单独设置 margin：

```xml
<!-- Avalonia -->
<StackPanel Spacing="8">
    <Button Content="First" />
    <Button Content="Second" />
</StackPanel>
```

在 WPF 中，通常需要为每个子元素设置 margin 才能实现相同效果：

```xml
<!-- WPF -->
<StackPanel>
    <Button Content="First" Margin="0,0,0,8" />
    <Button Content="Second" />
</StackPanel>
```

## ScrollViewer 差异

`ScrollViewer` 在两个框架中的工作方式基本相同。`HorizontalScrollBarVisibility` 和 `VerticalScrollBarVisibility` 属性使用相同的取值（`Auto`、`Visible`、`Hidden`、`Disabled`）。不过，不同平台上的默认滚动行为可能会略有差异，因此请在目标平台上实际测试滚动体验。

## Viewbox

`Viewbox` 在两个框架中的行为相同。它会拉伸或缩放其子内容以填满可用空间。

## 布局取整

Avalonia 与 WPF 一样使用 `UseLayoutRounding`，将布局测量结果吸附到像素边界。这有助于避免由子像素定位导致的模糊渲染问题。

## 另请参阅

- [布局](/docs/layout)：Avalonia 布局系统概览。
- [控件定位](/docs/layout/positioning-controls)：边距、对齐与定位。
