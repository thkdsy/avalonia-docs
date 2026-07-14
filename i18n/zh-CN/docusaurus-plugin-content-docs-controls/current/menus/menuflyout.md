---
id: menuflyout
title: MenuFlyout
description: 一个用于显示简单命令菜单的浮出层，通常附加到按钮或其他控件上。
doc-type: reference
---

import MenuFlyoutScreenshot from '/img/reference/controls/menuflyout/menuflyout-button.gif';

`MenuFlyout` 允许你将一个简单菜单作为某个控件的浮出层来使用。你可以把它当作 [ContextMenu](/controls/menus/contextmenu) 的一种替代方案。

菜单浮出层的属性与 [Flyout](/controls/layout/containers/flyout) 相同。

## 示例

下面是一个简单的菜单浮出层示例：

<XamlPreview>

```xml
<Button xmlns="https://github.com/avaloniaui"
        Content="Button"
        HorizontalAlignment="Center">
  <Button.Flyout>
    <MenuFlyout>
      <MenuItem Header="Open"/>
      <MenuItem Header="-"/>
      <MenuItem Header="Close"/>        
    </MenuFlyout>
  </Button.Flyout>
</Button>
```

</XamlPreview>

:::info
请注意，`<Separator/>` 元素在菜单浮出层中不起作用。如上所示，如果你想创建分隔线，请使用一个标题设置为 `'-'` 的 `<MenuItem>` 元素。
:::

## 配合命令和图标使用

```xml
<Button Content="Actions">
    <Button.Flyout>
        <MenuFlyout>
            <MenuItem Header="Open" Command="{Binding OpenCommand}">
                <MenuItem.Icon>
                    <PathIcon Data="{StaticResource open_regular}" />
                </MenuItem.Icon>
            </MenuItem>
            <MenuItem Header="-" />
            <MenuItem Header="Delete" Command="{Binding DeleteCommand}"
                      CommandParameter="{Binding SelectedItem}" />
        </MenuFlyout>
    </Button.Flyout>
</Button>
```

## 动态 MenuFlyout

下面的示例展示了如何基于 `MyMenuItems` 集合在运行时动态创建一个 `MenuFlyout`，其中集合项的类型为 `MyMenuItemViewModel`。

```xml
<Button Content="Button">
  <Button.Flyout>
    <MenuFlyout ItemsSource="{Binding MyMenuItems}">
      <MenuFlyout.ItemContainerTheme>
        <ControlTheme TargetType="MenuItem" BasedOn="{StaticResource {x:Type MenuItem}}"
          x:DataType="l:MyMenuItemViewModel">

          <Setter Property="Header" Value="{Binding Header}"/>
          <Setter Property="ItemsSource" Value="{Binding Items}"/>
          <Setter Property="Command" Value="{Binding Command}"/>
          <Setter Property="CommandParameter" Value="{Binding CommandParameter}"/>
          
        </ControlTheme>
      </MenuFlyout.ItemContainerTheme>
    </MenuFlyout>
  </Button.Flyout>
</Button>
```

## 放置位置

你可以通过设置 `Placement` 属性来控制浮出层相对于目标控件出现的位置。该属性接受 `FlyoutPlacementMode` 值，例如 `Top`、`Bottom`、`Left`、`Right`、`TopEdgeAlignedLeft`、`TopEdgeAlignedRight`、`BottomEdgeAlignedLeft`、`BottomEdgeAlignedRight` 等。

```xml
<Button Content="Options">
    <Button.Flyout>
        <MenuFlyout Placement="BottomEdgeAlignedLeft">
            <MenuItem Header="Settings"/>
            <MenuItem Header="About"/>
        </MenuFlyout>
    </Button.Flyout>
</Button>
```

如果你没有设置 `Placement`，浮出层会使用由其附加控件决定的默认位置。

## 另请参阅

- [MenuFlyout API reference](/api/avalonia/controls/menuflyout)
- [`MenuFlyout.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Flyouts/MenuFlyout.cs)
- [Separator](/controls/menus/separator)
- [Menu](/controls/menus/menu)
