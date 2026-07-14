---
id: scrollviewer
title: ScrollViewer
description: 一个当内容超出可见区域时提供滚动条的容器控件。
doc-type: reference
---

[`ScrollViewer`](/api/avalonia/controls/scrollviewer) 控件可以包含比其内容区域更大的内容。它会提供滚动条，使用户可以将隐藏的内容滚动到可见区域中。

:::warning
你不能将 `ScrollViewer` 放到在滚动方向上具有无限高度或宽度的控件中，例如 `StackPanel`。为避免此问题，请为 `ScrollViewer` 设置固定的 `Height`/`Width` 或 `MaxHeight`/`MaxWidth`，或者改用其他容器面板。
:::

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `HorizontalScrollBarVisibility` | [`ScrollBarVisibility`](/api/avalonia/controls/primitives/scrollbarvisibility) | 控制水平滚动条：`Auto`、`Visible`、`Hidden`、`Disabled`。 |
| `VerticalScrollBarVisibility` | `ScrollBarVisibility` | 控制垂直滚动条：`Auto`、`Visible`、`Hidden`、`Disabled`。 |
| `AllowAutoHide` | `bool` | 默认值为 `true`。设置当指针不在控件上方时，滚动条是否自动隐藏。 |
| `Offset` | `Vector` | 当前滚动位置（X, Y）。 |
| `Extent` | `Size` | 可滚动内容的总尺寸。 |
| `Viewport` | `Size` | 可见区域的尺寸。 |
| `IsScrollChainingEnabled` | `bool` | 附加属性，默认值为 `true`。设置在内部可滚动控件上时，用于决定滚动事件是否会传递给外层 `ScrollViewer`。 |
| `IsDeferredScrollingEnabled` | `bool` | 默认值为 `false`。为 `true` 时，内容要等到用户释放滚动条滑块后才会滚动。这对大型内容的性能优化很有帮助。 |
| `BringIntoViewOnFocusChange` | `bool` | 默认值为 `true`。当子控件获得焦点时，`ScrollViewer` 会自动滚动以将其带入可见区域。 |

## 滚动条可见性选项

每个滚动方向都接受以下 `ScrollBarVisibility` 值之一：

| 值 | 行为 |
|---|---|
| `Auto` | 仅当内容溢出时显示滚动条。这是垂直滚动的默认值。 |
| `Visible` | 始终显示滚动条，即使内容完全适应视口。 |
| `Hidden` | 隐藏滚动条，但仍允许通过触控、鼠标滚轮或键盘滚动。 |
| `Disabled` | 完全禁用该方向上的滚动。这是水平滚动的默认值。 |

```xml
<!-- 始终显示垂直滚动条，并禁用水平滚动 -->
<ScrollViewer VerticalScrollBarVisibility="Visible"
              HorizontalScrollBarVisibility="Disabled">
    <TextBlock Text="{Binding LongText}" TextWrapping="Wrap" />
</ScrollViewer>
```

## 滚动链

当你在 `ScrollViewer` 内嵌套另一个可滚动控件，并且用户在内部控件中滚动到了边界时，滚动链会决定外层 `ScrollViewer` 是否继续滚动。你可以通过设置内部控件上的 `IsScrollChainingEnabled` 附加属性来启用或禁用此行为：

```xml
<ScrollViewer>
    <StackPanel>
        <!-- 这个内部 ListBox 不会把滚动继续传递给外层 ScrollViewer -->
        <ListBox ScrollViewer.IsScrollChainingEnabled="False"
                 ItemsSource="{Binding Items}"
                 MaxHeight="200" />
        <TextBlock Text="Other content below" />
    </StackPanel>
</ScrollViewer>
```

以下控件支持这个附加属性：

* `ScrollViewer`
* `DataGrid`
* `ListBox`
* `TextBox`
* `TreeView`

## 以编程方式滚动

你可以在 code-behind 或视图模型中控制滚动位置：

```csharp
// 滚动到指定位置
scrollViewer.Offset = new Vector(0, 500);

// 滚动到顶部
scrollViewer.Offset = new Vector(scrollViewer.Offset.X, 0);

// 滚动到底部
scrollViewer.Offset = new Vector(scrollViewer.Offset.X, scrollViewer.Extent.Height);

// 将某个子元素滚动到可见区域
targetControl.BringIntoView();
```

你也可以通过订阅 `Offset` 属性变化来监听滚动位置变化：

```csharp
scrollViewer.GetObservable(ScrollViewer.OffsetProperty).Subscribe(offset =>
{
    // 响应滚动位置变化
    Debug.WriteLine($"Scrolled to: {offset.X}, {offset.Y}");
});
```

## 示例

此示例创建了一个高度超过其外层 `Border` 的 `StackPanel`。`ScrollViewer` 会自动显示垂直滚动条。

<XamlPreview>

```xml
<Border xmlns="https://github.com/avaloniaui" Background="Gray" Width="200" Height="200">
  <ScrollViewer>
    <StackPanel>
      <TextBlock FontSize="22" Height="50" Background="Blue">Block 1</TextBlock>
      <TextBlock FontSize="22" Height="50">Block 2</TextBlock>
      <TextBlock FontSize="22" Height="50" Background="Blue">Block 3</TextBlock>
      <TextBlock FontSize="22" Height="50">Block 4</TextBlock>
      <TextBlock FontSize="22" Height="50" Background="Blue">Block 5</TextBlock>
    </StackPanel>
  </ScrollViewer>
</Border>
```

</XamlPreview>

### 水平滚动

若要启用水平滚动，请将 `HorizontalScrollBarVisibility` 设置为 `Auto` 或 `Visible`：

```xml
<ScrollViewer HorizontalScrollBarVisibility="Auto"
              VerticalScrollBarVisibility="Disabled"
              Height="100">
    <StackPanel Orientation="Horizontal" Spacing="8">
        <Border Width="200" Height="80" Background="CornflowerBlue" />
        <Border Width="200" Height="80" Background="SeaGreen" />
        <Border Width="200" Height="80" Background="Coral" />
        <Border Width="200" Height="80" Background="MediumPurple" />
    </StackPanel>
</ScrollViewer>
```

## 另请参阅

- [ScrollViewer API reference](/api/avalonia/controls/scrollviewer)
- [`ScrollViewer.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ScrollViewer.cs)
