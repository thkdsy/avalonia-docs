---
id: refreshcontainer
title: RefreshContainer
---

The `RefreshContainer` allows a user to pull down on content or a list of data to refresh the content or retrieve more data. The refresh progress is indicated by a `RefreshVisualizer` that appears from the edge by which the pull gesture was initiated. The content of a `RefreshContainer` must be a `ScrollViewer`, or a control that has one.
`RefreshContainer` 允许用户通过下拉内容区域或数据列表来刷新内容或获取更多数据。刷新进度由 `RefreshVisualizer` 指示，它会从触发拉取手势的边缘显示出来。`RefreshContainer` 的内容必须是 `ScrollViewer`，或者是包含 `ScrollViewer` 的控件。

## 示例

```xml
<RefreshContainer xmlns="https://github.com/avaloniaui" PullDirection="TopToBottom"
                RefreshRequested="RefreshContainerPage_RefreshRequested">
    <ListBox ItemsSource="{Binding Items}"/>
</RefreshContainer>
```

```csharp
private void RefreshContainerPage_RefreshRequested(object? sender, RefreshRequestedEventArgs e)
{
    // 获取一个 deferral 对象。
    var deferral = e.GetDeferral();

    // 刷新 ListBox 项目

    // 通知 RefreshContainer 刷新已完成。
    deferral.Complete();
}
```

## 刷新机制
可以通过沿 `PullDirection` 属性指定的方向拉动到 visualizer 的完整范围来触发刷新，也可以调用 RefreshContainer 的 `RequestRefresh` 方法。刷新进度由 `Visualizer` 的 `RefreshVisualizerState` 指示，其状态可能为以下几种：

### Idle
这是 visualizer 的默认状态。用户当前没有与容器交互，也没有正在进行的刷新。visualizer 处于隐藏状态。

### Interacting
用户正在沿 `PullDirection` 属性指定的方向拉动，但尚未达到触发阈值。visualizer 会逐渐显示，直到达到拉取阈值。
如果在达到拉取阈值之前就松手，`Visualizer` 会返回 `Idle` 状态，并且不会触发刷新。
如果达到拉取阈值，`Visualizer` 会进入 `Pending` 状态。

### Pending
用户已经拉过了触发阈值。在此状态下，visualizer 会完全可见。如果用户将触点移回到阈值之前，visualizer 会回到 `Interacting` 状态。如果用户在 `Pending` 状态下松手，visualizer 会进入 `Refreshing` 状态。

### Refreshing
用户在 visualizer 处于 `Pending` 状态时松开了触摸接触点，此时会触发 `RefreshRequested` 事件。事件参数中包含一个 `Deferral` 对象，它用于通知 RefreshContainer 刷新操作已完成，适用于耗时较长且不希望阻塞 UI 线程的刷新操作。如果没有获取该对象，则 `RefreshRequested` 调用结束时 `Refreshing` 状态也会结束。
在这个状态下，visualizer 会完全显示，并开始播放刷新动画。

### Peeking
当用户在内容处于不允许刷新的位置时开始拉取手势，就会出现此状态。典型情况是：在开始拉取时，子级 ScrollViewer 相对于拉取方向和滚动方向的 Offset 并不为 0。此时 visualizer 会保持隐藏，且其状态只有在松开拉取后才会回到 `Idle`。

## 另请参阅

- [`RefreshContainer.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/PullToRefresh/RefreshContainer.cs)
