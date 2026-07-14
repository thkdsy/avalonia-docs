---
id: contextmenu
title: ContextMenu
description: 当用户右键单击控件时出现的弹出菜单，用于提供上下文相关操作。
doc-type: reference
---

[`ContextMenu`](/api/avalonia/controls/contextmenu) 是一种在你右键单击控件时出现的弹出菜单。它提供与控件或其内容相关的上下文操作。你可以通过附加属性将 `ContextMenu` 绑定到任何宿主控件上，因此应用中的任何可视元素都可以拥有自己的右键菜单。

:::info
若要了解附加属性的工作方式，请参阅 [Attached properties](/docs/custom-controls/attached-properties)。
:::

## 基本示例

在这个示例中，一个上下文菜单被附加到多行文本框上。请在预览区域中右键单击以查看上下文菜单。

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <TextBox AcceptsReturn="True" TextWrapping="Wrap" Text="Right-click here">
    <TextBox.ContextMenu>
      <ContextMenu>
        <MenuItem Header="Copy"/>
        <MenuItem Header="Paste"/>
      </ContextMenu>
    </TextBox.ContextMenu>
  </TextBox>
</UserControl>
```

</XamlPreview>

## 命令和图标

你可以将菜单项绑定到命令，并添加图标，以创建一个功能完整的上下文菜单：

```xml
<ListBox ItemsSource="{Binding Items}">
    <ListBox.ContextMenu>
        <ContextMenu>
            <MenuItem Header="Edit" Command="{Binding EditCommand}">
                <MenuItem.Icon>
                    <PathIcon Data="{StaticResource edit_regular}" />
                </MenuItem.Icon>
            </MenuItem>
            <MenuItem Header="Delete" Command="{Binding DeleteCommand}">
                <MenuItem.Icon>
                    <PathIcon Data="{StaticResource delete_regular}" />
                </MenuItem.Icon>
            </MenuItem>
            <Separator />
            <MenuItem Header="Properties" Command="{Binding PropertiesCommand}" />
        </ContextMenu>
    </ListBox.ContextMenu>
</ListBox>
```

使用 `Separator` 元素可以在视觉上对相关菜单项进行分组。

## 向命令传递上下文

使用 `CommandParameter` 可将相关数据项传递给命令。当上下文菜单附加到列表或集合控件上，并且你需要知道哪个项目被右键单击时，这种方式特别有用：

```xml
<ListBox.ContextMenu>
    <ContextMenu>
        <MenuItem Header="Delete"
                  Command="{Binding DeleteCommand}"
                  CommandParameter="{Binding $parent[ListBox].SelectedItem}" />
    </ContextMenu>
</ListBox.ContextMenu>
```

`$parent[ListBox].SelectedItem` 绑定会沿可视树向上查找父级 `ListBox`，并读取其 `SelectedItem` 属性。

## 动态生成菜单项

如果你的上下文菜单项依赖运行时数据，可以将 `ItemsSource` 绑定到视图模型中的某个集合：

```xml
<TextBlock Text="Right-click me">
    <TextBlock.ContextMenu>
        <ContextMenu ItemsSource="{Binding ContextActions}" />
    </TextBlock.ContextMenu>
</TextBlock>
```

绑定集合中的每个项目都会变成一个 `MenuItem`。你可以使用 `DataTemplate` 或 `ItemContainerTheme` 来控制这些项目的显示方式。

## 处理打开和关闭

你可以通过处理 `Opening` 和 `Closing` 事件来响应上下文菜单的生命周期。`Opening` 事件会提供一个 `CancelEventArgs` 参数，因此你可以在某些条件不满足时阻止菜单显示：

```csharp
private void ContextMenu_Opening(object? sender, System.ComponentModel.CancelEventArgs e)
{
    if (!IsActionAllowed)
    {
        e.Cancel = true; // 阻止上下文菜单打开
    }
}
```

```xml
<ContextMenu Opening="ContextMenu_Opening" Closing="ContextMenu_Closing">
    <MenuItem Header="Copy" />
</ContextMenu>
```

当你需要有条件地显示或隐藏上下文菜单，或者希望在其出现前更新菜单项时，这会非常有用。

## 上下文浮出层

你可以使用 `ContextFlyout` 作为 `ContextMenu` 的替代方案。上下文浮出层不仅可以包含菜单项，还可以包含任意内容，从而带来更丰富的 UI 体验：

```xml
<Border Background="LightGray" Padding="20">
    <Border.ContextFlyout>
        <Flyout>
            <StackPanel Spacing="8" Width="200">
                <TextBlock Text="Options" FontWeight="Bold" />
                <Button Content="Action" />
            </StackPanel>
        </Flyout>
    </Border.ContextFlyout>
    <TextBlock Text="Right-click for options" />
</Border>
```

:::caution
同一个控件不能同时附加 `ContextFlyout` 和 `ContextMenu`。如果两者都设置了，最终只会使用其中一个。
:::

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `ItemsSource` | `IEnumerable` | 将菜单项绑定到一个集合，以便动态生成。 |
| `Opening` | `event` | 在上下文菜单打开前触发。将 `Cancel` 设为 `true` 可阻止其打开。 |
| `Closing` | `event` | 在上下文菜单关闭时触发。 |
| `Placement` | `PlacementMode` | 控制上下文菜单相对于指针的位置。 |

## 另请参阅

- [Menu](/controls/menus/menu)
- [MenuFlyout](/controls/menus/menuflyout)
- [Flyout](/controls/layout/containers/flyout)
- [ContextMenu API reference](/api/avalonia/controls/contextmenu)
- [`ContextMenu.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ContextMenu.cs)
