---
id: wrappanel
title: WrapPanel
description: 一个以流式换行布局排列子控件的面板。
doc-type: reference
---

`WrapPanel` 会按顺序从左到右排列其子控件，当剩余空间不足时会自动换到下一行（这会考虑 margin 和 border 所占用的空间）。

当你将 `Orientation` 属性设置为 `Vertical` 时，排列方向会改为从上到下；当没有足够高度时，会开始新的一列。

当你需要一种灵活、流式的布局，并希望子元素能随着可用空间变化而自动重排时，`WrapPanel` 会非常有用。常见使用场景包括标签列表、缩略图库和按钮工具栏。

## 常用属性

| 属性 | 说明 |
|---|---|
| `Orientation` | 排列流向：`Horizontal`（默认）或 `Vertical`。 |
| `ItemSpacing` | 项目之间的水平间距。 |
| `LineSpacing` | 行之间的垂直间距（在垂直模式下则表示列之间的水平间距）。 |
| `ItemWidth` | 所有项目的固定宽度。若未设置，则使用项目的自然宽度。 |
| `ItemHeight` | 所有项目的固定高度。若未设置，则使用项目的自然高度。 |
| `ItemsAlignment` | 控制项目在其分配单元格中的对齐方式。类型为 `WrapPanelItemsAlignment`，可选值为 `Start`、`Center`、`End`。 |

## 常见用法

### 统一项目尺寸

如果你希望每个子元素占用相同的空间，可以设置 `ItemWidth` 和 `ItemHeight`。这对于类似网格的布局特别有帮助，尤其是在项目内容不同但你希望整体视觉结构保持一致时。

```xml
<WrapPanel ItemWidth="120" ItemHeight="120"
           ItemSpacing="8" LineSpacing="8">
    <Button Content="Short" />
    <Button Content="Medium text" />
    <Button Content="A longer button label" />
</WrapPanel>
```

### 单元格内对齐

当你使用 `ItemWidth` 或 `ItemHeight` 时，子控件可能会小于其分配到的单元格。此时可以使用 `ItemsAlignment` 来控制它们在单元格中的位置。

```xml
<WrapPanel ItemWidth="100" ItemHeight="100"
           ItemsAlignment="Center">
    <Button Content="A" />
    <Button Content="B" />
    <Button Content="C" />
</WrapPanel>
```

## 示例

### 水平排列（默认）

<XamlPreview>

```xml
<WrapPanel xmlns="https://github.com/avaloniaui"
           ItemSpacing="20" LineSpacing="20"
           Margin="20">
    <Rectangle Fill="Navy" Width="80" Height="80" />
    <Rectangle Fill="Yellow" Width="80" Height="80" />
    <Rectangle Fill="Green" Width="80" Height="80" />
    <Rectangle Fill="Red" Width="80" Height="80" />
    <Rectangle Fill="Purple" Width="80" Height="80" />
</WrapPanel>
```

</XamlPreview>

### 垂直排列

<XamlPreview>

```xml
<WrapPanel xmlns="https://github.com/avaloniaui"
           Orientation="Vertical"
           ItemSpacing="20" LineSpacing="20"
           Margin="20">
    <Rectangle Fill="Navy" Width="80" Height="80" />
    <Rectangle Fill="Yellow" Width="80" Height="80" />
    <Rectangle Fill="Green" Width="80" Height="80" />
    <Rectangle Fill="Red" Width="80" Height="80" />
    <Rectangle Fill="Purple" Width="80" Height="80" />
</WrapPanel>
```

</XamlPreview>

## 另请参阅

- [StackPanel](/controls/layout/panels/stackpanel)
- [DockPanel](/controls/layout/panels/dockpanel)
- [WrapPanel API reference](/api/avalonia/controls/wrappanel)
- [`WrapPanel.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/WrapPanel.cs)
