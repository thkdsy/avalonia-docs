---
id: gridsplitter
title: GridSplitter
description: 一个允许用户在运行时通过拖动分隔条来调整 Grid 中行或列大小的控件。
doc-type: reference
---

[`GridSplitter`](/api/avalonia/controls/gridsplitter) 控件允许用户在运行时调整 `Grid` 中列或行的大小。分隔条会以一列或一行的形式绘制（尺寸可指定），并带有一个可供用户在运行时操作的拖动区域。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Background` | `IBrush` | 分隔条的背景颜色。 |
| `ResizeDirection` | `GridResizeDirection` | 分隔条的移动方向：`Auto`、`Columns`、`Rows`。见下方说明。 |
| `ResizeBehavior` | `GridResizeBehavior` | 要调整大小的列/行：`BasedOnAlignment`、`CurrentAndNext`、`PreviousAndCurrent`、`PreviousAndNext`。 |
| `DragIncrement` | `double` | 分隔条每次移动的最小像素数。 |
| `ShowsPreview` | `bool` | 为 `true` 时，在拖动过程中显示预览线，而不是实时调整大小。 |

:::caution
若要产生有意义的移动效果，分隔条的移动方向必须与其所在位置的定义一致。也就是说：列分隔条应指定 `ResizeDirection="Columns"`，行分隔条应指定 `ResizeDirection="Rows"`。
:::

## 拖动行为

当用户点击并拖动 `GridSplitter` 时，相邻的列或行会根据 `ResizeBehavior` 属性进行调整：

- **`BasedOnAlignment`**（默认）：要调整的列或行取决于分隔条在其单元格中的 `HorizontalAlignment` 或 `VerticalAlignment`。
- **`CurrentAndNext`**：调整当前列/行和下一列/行。
- **`PreviousAndCurrent`**：调整前一列/行和当前列/行。
- **`PreviousAndNext`**：调整前一列/行和下一列/行，跳过分隔条自身所在的列/行。

如果你将 `ShowsPreview` 设置为 `true`，拖动时会有一条半透明的预览指示线跟随指针移动。实际的大小调整只会在你松开鼠标按钮后应用。这对于复杂布局可能有助于提升性能。

`DragIncrement` 属性控制拖动的吸附粒度。例如，将 `DragIncrement="10"` 设置后，分隔条的位置会以 10 像素为增量进行吸附。

## 最小/最大约束

你可以在 `ColumnDefinition` 元素上设置 `MinWidth`/`MaxWidth`（或在 `RowDefinition` 元素上设置 `MinHeight`/`MaxHeight`），以限制分隔条可移动的范围。`GridSplitter` 会自动遵守这些约束，因此用户无法拖动到超出定义限制的位置。

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="200" MinWidth="100" MaxWidth="400" />
        <ColumnDefinition Width="4" />
        <ColumnDefinition Width="*" MinWidth="200" />
    </Grid.ColumnDefinitions>
    <Border Grid.Column="0" Background="LightBlue">
        <TextBlock Text="Sidebar (100-400px)" Margin="8" />
    </Border>
    <GridSplitter Grid.Column="1" Background="Gray" ResizeDirection="Columns" />
    <Border Grid.Column="2" Background="LightGreen">
        <TextBlock Text="Content (min 200px)" Margin="8" />
    </Border>
</Grid>
```

## 键盘支持

`GridSplitter` 支持键盘交互，以提升无障碍性。当分隔条获得焦点后，你可以使用以下按键：

| 按键 | 操作 |
|---|---|
| `Left` / `Right` | 将列分隔条向左或向右移动。 |
| `Up` / `Down` | 将行分隔条向上或向下移动。 |

每次按键都会按 `DragIncrement` 指定的距离移动分隔条（默认 1 像素）。

## 示例

这是一个列分隔条。拖动列之间的边界即可调整它们的大小。

<XamlPreview>

```xml
<Grid xmlns="https://github.com/avaloniaui"
      ColumnDefinitions="*, 4, *">
    <Rectangle Grid.Column="0" Fill="Blue"/>
    <GridSplitter Grid.Column="1" Background="Black" ResizeDirection="Columns"/>
    <Rectangle Grid.Column="2" Fill="Red"/>
</Grid>
```

</XamlPreview>

这是一个行分隔条。拖动行之间的边界即可调整它们的大小。

<XamlPreview>

```xml
<Grid xmlns="https://github.com/avaloniaui"
      RowDefinitions="*, 4, *">
    <Rectangle Grid.Row="0" Fill="Blue"/>
    <GridSplitter Grid.Row="1" Background="Black" ResizeDirection="Rows"/>
    <Rectangle Grid.Row="2" Fill="Red"/>
</Grid>
```

</XamlPreview>

## 另请参阅

- [Grid](/controls/layout/panels/grid)
- [GridSplitter API reference](/api/avalonia/controls/gridsplitter)
- [`GridSplitter.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/GridSplitter.cs)
