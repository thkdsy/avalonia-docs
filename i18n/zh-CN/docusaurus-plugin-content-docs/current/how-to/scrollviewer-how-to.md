---
id: scrollviewer-how-to
title: "如何：使用 ScrollViewer"
description: 学习如何控制滚动行为、通过代码滚动、响应滚动事件、实现无限滚动，并处理 Avalonia 中的嵌套滚动区域。
doc-type: how-to
---

本指南介绍常见的 [`ScrollViewer`](/api/avalonia/controls/scrollviewer) 使用场景，包括控制滚动行为、通过代码滚动、响应滚动事件以及处理嵌套滚动区域。

## 基础用法

把任何可能超出可用空间的内容包裹进 `ScrollViewer` 中：

```xml
<ScrollViewer>
    <StackPanel Spacing="8">
        <!-- 可能高于可视区域的内容 -->
        <TextBlock Text="Item 1" />
        <TextBlock Text="Item 2" />
        <!-- ... 更多项目 ... -->
    </StackPanel>
</ScrollViewer>
```

当内容超出可视区域时，`ScrollViewer` 会自动显示滚动条。

:::tip
不要把 `ScrollViewer` 放进一个在滚动方向上提供无限高度或无限宽度的容器中，例如 `StackPanel`。否则 `ScrollViewer` 永远检测不到溢出，因为父级给了它无限空间。正确做法是使用一个尺寸受约束的容器，例如 `Grid`、`DockPanel`，或者显式设置 `Height` / `MaxHeight`。
:::

## 滚动条可见性

你可以通过设置 `HorizontalScrollBarVisibility` 和 `VerticalScrollBarVisibility`，控制滚动条在何时显示：

```xml
<!-- 始终显示垂直滚动条，从不显示水平滚动条 -->
<ScrollViewer VerticalScrollBarVisibility="Visible"
              HorizontalScrollBarVisibility="Disabled">
    <TextBlock Text="{Binding LongText}" TextWrapping="Wrap" />
</ScrollViewer>
```

| 值 | 行为 |
|---|---|
| `Auto` | 仅在内容溢出时显示滚动条（垂直方向默认值） |
| `Visible` | 始终显示滚动条，即使内容没有溢出 |
| `Hidden` | 隐藏滚动条，但仍允许通过触摸、鼠标滚轮或键盘滚动 |
| `Disabled` | 完全禁用该方向滚动 |

:::note
如果把 `HorizontalScrollBarVisibility` 设为 `Disabled`（默认值），那么超出可视区域宽度的内容会被裁剪。如果你需要水平滚动，应将其设置为 `Auto` 或 `Visible`。
:::

## 通过代码滚动

### 滚动到指定位置

直接设置 `Offset` 属性，可以立即跳转到指定滚动位置：

```csharp
// 向下滚动到距离顶部 500 像素的位置
scrollViewer.Offset = new Vector(0, 500);
```

`Offset` 使用的是设备无关像素。其值会自动被限制在合法范围内，因此即使你传入超出滚动范围的值，也只会滚动到边界，不会抛异常。

### 滚动到顶部或底部

```csharp
// 滚动到顶部
scrollViewer.Offset = new Vector(scrollViewer.Offset.X, 0);

// 滚动到底部
scrollViewer.Offset = new Vector(
    scrollViewer.Offset.X,
    scrollViewer.Extent.Height - scrollViewer.Viewport.Height);
```

### 让某个子元素进入可视区域

对某个子控件调用 `BringIntoView`，可以让它刚好滚动到可见区域。这在你知道目标控件是谁、但不知道它的准确位置时尤其有用：

```csharp
targetControl.BringIntoView();
```

你也可以传入一个相对于目标控件的矩形区域：

```csharp
targetControl.BringIntoView(new Rect(0, 0, targetControl.Bounds.Width, targetControl.Bounds.Height));
```

:::tip
`BringIntoView` 也适用于启用了虚拟化的面板。当你对使用虚拟化的 `ItemsControl` 中某个项目调用它时，面板会先把该项实例化出来，然后再滚动到它。
:::

## 响应滚动事件

### 监听滚动位置变化

订阅 `ScrollChanged` 事件，以便在用户滚动时做出响应：

```csharp
scrollViewer.ScrollChanged += (sender, e) =>
{
    var offset = scrollViewer.Offset;
    var extent = scrollViewer.Extent;
    var viewport = scrollViewer.Viewport;

    // 检查是否已经滚动到底部（允许 1 像素误差）
    var isAtBottom = offset.Y >= extent.Height - viewport.Height - 1;

    if (isAtBottom)
    {
        LoadMoreItems();
    }
};
```

### 观察 Offset 属性

如果你偏好响应式写法，也可以直接观察 `Offset` 属性：

```csharp
scrollViewer.GetObservable(ScrollViewer.OffsetProperty).Subscribe(offset =>
{
    Debug.WriteLine($"Scrolled to: {offset.Y}");
});
```

这种方式能很好地融入 Avalonia 的响应式属性系统，并且无论是用户滚动还是代码滚动，只要 `Offset` 变化就会触发。

## 实现无限滚动

一个常见模式是：当用户快滚动到底部时，继续加载更多内容。你可以把滚动位置检查和异步加载方法结合起来：

```csharp
public partial class InfiniteListViewModel : ObservableObject
{
    private int _page = 0;
    private bool _isLoading;

    public ObservableCollection<Item> Items { get; } = new();

    public async Task LoadMoreAsync()
    {
        if (_isLoading) return;
        _isLoading = true;

        try
        {
            var newItems = await _api.GetItemsAsync(_page++, pageSize: 20);
            foreach (var item in newItems)
                Items.Add(item);
        }
        finally
        {
            _isLoading = false;
        }
    }
}
```

然后在 code-behind 中，当用户滚动到底部阈值附近时触发加载：

```csharp
private async void OnScrollChanged(object? sender, ScrollChangedEventArgs e)
{
    if (sender is not ScrollViewer sv) return;

    var distanceFromBottom = sv.Extent.Height - sv.Viewport.Height - sv.Offset.Y;
    if (distanceFromBottom < 100)
    {
        await ((InfiniteListViewModel)DataContext!).LoadMoreAsync();
    }
}
```

:::note
阈值（本例中是 100 像素）决定了“提前多久开始加载”。阈值越大，数据源就越有时间在用户真正滚到底部前完成响应，从而带来更流畅的体验。
:::

## 处理嵌套 ScrollViewer

如果你嵌套了多个可滚动区域，建议禁用内层中由外层已经处理的那个滚动方向，以避免两个滚动区域竞争同一类输入：

```xml
<ScrollViewer VerticalScrollBarVisibility="Auto">
    <StackPanel Spacing="16">
        <TextBlock Text="Section 1" FontSize="20" />

        <!-- 内层只负责水平滚动 -->
        <ScrollViewer HorizontalScrollBarVisibility="Auto"
                      VerticalScrollBarVisibility="Disabled">
            <StackPanel Orientation="Horizontal" Spacing="8">
                <Border Width="200" Height="150" Background="Red" />
                <Border Width="200" Height="150" Background="Blue" />
                <Border Width="200" Height="150" Background="Green" />
            </StackPanel>
        </ScrollViewer>

        <TextBlock Text="Section 2" FontSize="20" />
        <!-- 更多内容... -->
    </StackPanel>
</ScrollViewer>
```

如果内层控件与外层控件在同一方向上都可以滚动，那么可以通过在内层控件上设置附加属性 `ScrollViewer.IsScrollChainingEnabled`，控制滚动事件是否继续“传递”给父级：

```xml
<!-- 阻止内层滚动继续传递到外层 ScrollViewer -->
<ListBox ScrollViewer.IsScrollChainingEnabled="False"
         Height="200"
         ItemsSource="{Binding InnerItems}" />
```

## 使用滚动吸附点

为类似轮播卡片的滚动启用吸附点：

```xml
<ScrollViewer HorizontalScrollBarVisibility="Auto"
              VerticalScrollBarVisibility="Disabled"
              IsScrollChainingEnabled="True">
    <StackPanel Orientation="Horizontal" Spacing="16">
        <!-- 会吸附到视图中的卡片 -->
        <Border Width="300" Height="200" Background="#6366F1" CornerRadius="8" />
        <Border Width="300" Height="200" Background="#8B5CF6" CornerRadius="8" />
        <Border Width="300" Height="200" Background="#A78BFA" CornerRadius="8" />
    </StackPanel>
</ScrollViewer>
```

## 创建粘性标题布局

可以使用 `Grid` 让标题固定不动，而内容在其下方滚动：

```xml
<Grid RowDefinitions="Auto,*">
    <!-- 固定标题 -->
    <Border Grid.Row="0" Background="White" Padding="16"
            ZIndex="1" BoxShadow="0 2 4 0 #20000000">
        <TextBlock Text="Fixed Header" FontWeight="Bold" />
    </Border>

    <!-- Scrollable content -->
    <ScrollViewer Grid.Row="1">
        <StackPanel Spacing="8" Margin="16">
            <!-- Your scrollable content here -->
        </StackPanel>
    </ScrollViewer>
</Grid>
```

这种模式可以让标题始终保持可见。标题 `Border` 上的 `ZIndex` 确保了当它与可滚动内容在过渡或动画中发生重叠时，标题仍然渲染在更上层。

## 关键属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Offset` | `Vector` | 当前滚动位置（X, Y）。 |
| `Extent` | `Size` | 可滚动内容的总尺寸。 |
| `Viewport` | `Size` | 当前可视区域的尺寸。 |
| `HorizontalScrollBarVisibility` | `ScrollBarVisibility` | 控制水平滚动条行为。 |
| `VerticalScrollBarVisibility` | `ScrollBarVisibility` | 控制垂直滚动条行为。 |
| `AllowAutoHide` | `bool` | 滚动条是否会在一段时间无操作后自动隐藏（默认 `true`）。 |
| `IsScrollChainingEnabled` | `bool` | 滚动事件是否继续传递给父级滚动区域。 |

## 另请参阅

- [ScrollViewer reference](/controls/layout/containers/scrollviewer)
- [Choosing a layout panel](/docs/layout/choosing-a-layout-panel)
- [Performance optimization](/docs/app-development/performance)
- [ItemsControl how-to](/docs/how-to/itemscontrol-how-to)
