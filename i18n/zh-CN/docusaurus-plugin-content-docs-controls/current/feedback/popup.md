---
id: popup
title: Popup
---

[`Popup`](/api/avalonia/controls/primitives/popup) 是一个底层控件，用于在其他内容上方的浮动窗口中显示内容。它是 `Flyout`、[`ToolTip`](/api/avalonia/controls/tooltip)、`ComboBox` 下拉列表和上下文菜单等更高层控件的基础。当你需要这些高层控件不提供的自定义定位或关闭行为时，可以直接使用 `Popup`。

:::info
在大多数场景下，优先使用 `Flyout`、`ToolTip` 或 `ContextMenu`，而不是 `Popup`。这些更高层的控件会自动处理无障碍支持、键盘导航和轻量关闭行为。
:::

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `IsOpen` | `bool` | 控制弹出层当前是否可见。 |
| `Child` | `Control` | 显示在弹出层内部的内容。 |
| `Placement` | `PlacementMode` | 弹出层相对于目标的定位方式。可选项包括 `Bottom`、`Top`、`Left`、`Right`、`Center`、`Pointer`、`AnchorAndGravity`。默认值为 `Bottom`。 |
| `PlacementTarget` | `Control` | 弹出层所相对定位的控件。默认值为弹出层的父级。 |
| `PlacementAnchor` | `PopupAnchor` | 目标控件上的锚点。 |
| `PlacementGravity` | `PopupGravity` | 弹出层从锚点展开的方向。 |
| `HorizontalOffset` | `double` | 相对于计算结果的位置的水平偏移量。 |
| `VerticalOffset` | `double` | 相对于计算结果的位置的垂直偏移量。 |
| `IsLightDismissEnabled` | `bool` | 为 `true` 时，当用户点击弹出层外部时会关闭它。默认值为 `false`。 |
| `Topmost` | `bool` | 弹出层是否显示在所有其他窗口之上。默认值为 `false`。 |
| `WindowManagerAddShadowHint` | `bool` | 是否应用投影阴影（取决于平台）。默认值为 `true`。 |
| `OverlayDismissEventPassThrough` | `bool` | 为 `true` 时，用于关闭弹出层的指针事件也会继续传递给底层控件。默认值为 `false`。 |
| `CustomPopupPlacementCallback` | `CustomPopupPlacementCallback` | 用于完全自定义弹出层定位的回调。设置后会覆盖 `Placement` 属性。 |

## 事件

| 事件 | 说明 |
|---|---|
| `Opened` | 在弹出层打开后触发。 |
| `Closed` | 在弹出层关闭后触发。 |

## 基本示例

```xml
<Panel>
    <Button x:Name="ToggleButton" Content="Show Popup"
            Click="OnTogglePopup" />

    <Popup x:Name="MyPopup"
           PlacementTarget="{Binding #ToggleButton}"
           Placement="Bottom"
           IsLightDismissEnabled="True">
        <Border Background="{DynamicResource SystemControlBackgroundAltHighBrush}"
                BorderBrush="{DynamicResource SystemControlForegroundBaseMediumBrush}"
                BorderThickness="1" CornerRadius="4" Padding="12">
            <TextBlock Text="This is popup content." />
        </Border>
    </Popup>
</Panel>
```

```csharp
private void OnTogglePopup(object sender, RoutedEventArgs e)
{
    MyPopup.IsOpen = !MyPopup.IsOpen;
}
```

## 放置模式

`Placement` 属性控制弹出层出现的位置：

| 模式 | 行为 |
|---|---|
| `Bottom` | 显示在目标下方，并左对齐。 |
| `Top` | 显示在目标上方，并左对齐。 |
| `Left` | 显示在目标左侧，并顶部对齐。 |
| `Right` | 显示在目标右侧，并顶部对齐。 |
| `Center` | 显示在目标中央。 |
| `Pointer` | 显示在当前指针位置。 |
| `AnchorAndGravity` | 使用 `PlacementAnchor` 和 `PlacementGravity` 进行精确定位。 |

## 绑定 `IsOpen`

你可以将 `IsOpen` 绑定到视图模型属性，以便在 MVVM 中控制它：

```xml
<Popup IsOpen="{Binding IsPopupVisible}"
       PlacementTarget="{Binding #AnchorControl}"
       Placement="Bottom"
       IsLightDismissEnabled="True">
    <Border Background="White" Padding="16" CornerRadius="4"
            BoxShadow="0 2 8 0 #40000000">
        <StackPanel Spacing="8">
            <TextBlock Text="Choose an option:" FontWeight="SemiBold" />
            <Button Content="Option A" Command="{Binding SelectOptionCommand}"
                    CommandParameter="A" />
            <Button Content="Option B" Command="{Binding SelectOptionCommand}"
                    CommandParameter="B" />
        </StackPanel>
    </Border>
</Popup>
```

## 自定义定位

如果内置放置模式无法满足定位需求，可以使用 `CustomPopupPlacementCallback`。该回调会收到一个已按默认值初始化好的 `CustomPopupPlacement` 对象，你可以通过修改其属性来控制定位：

```csharp
myPopup.CustomPopupPlacementCallback = placement =>
{
    placement.Anchor = PopupAnchor.TopRight;
    placement.Gravity = PopupGravity.BottomRight;
    placement.Offset = new Point(8, 0);
};
```

这个回调同样可用于 `PopupFlyoutBase`、`ContextMenu`，也可作为 `ToolTip` 的附加属性来使用：

```csharp
ToolTip.SetCustomPopupPlacementCallback(myControl, placement =>
{
    placement.Offset = new Point(0, -10);
});
```

## Popup、Flyout 与 ToolTip 的区别

| 特性 | Popup | Flyout | ToolTip |
|---|---|---|---|
| 层级 | 底层原语控件 | 高层附加控件 | 高层附加控件 |
| 触发方式 | 手动（`IsOpen`） | 编程式或附加式 | 悬停 |
| 轻量关闭 | 可选 | 内置 | 内置 |
| 键盘支持 | 手动处理 | 自动处理 | 不适用 |
| 最适合 | 自定义覆盖层行为 | 菜单、确认操作、选择器 | 悬停提示 |

## 另请参阅

- [Flyout](/controls/layout/containers/flyout)：附加到控件上的高层弹出层。
- [ToolTip](/controls/feedback/tooltip)：用于补充文本说明的悬停式弹出层。
- [ContextMenu](/controls/menus/contextmenu)：右键菜单。
