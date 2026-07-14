---
id: separator
title: Separator
description: 用于在 Menu、ContextMenu 和 MenuFlyout 控件中对相关菜单项进行分组的可视分隔线。
doc-type: reference
---

[`Separator`](/api/avalonia/controls/separator) 控件会在菜单项之间绘制一条水平线，用于在视觉上对相关命令进行分组。你可以在 `Menu`、`ContextMenu` 和 `MenuFlyout` 中使用它。

## 基本用法

在 `MenuItem` 元素之间放置一个 `<Separator/>` 元素即可创建分隔线：

```xml
<Menu DockPanel.Dock="Top">
  <MenuItem Header="_File">
    <MenuItem Header="_New"/>
    <MenuItem Header="_Open..."/>
    <Separator/>
    <MenuItem Header="_Exit"/>
  </MenuItem>
</Menu>
```

在上面的示例中，分隔线将文件操作与退出命令在视觉上区分开来。

## 在上下文菜单中

`Separator` 在 `ContextMenu` 中的用法也是一样的：

```xml
<TextBox Text="Right-click for options">
  <TextBox.ContextMenu>
    <ContextMenu>
      <MenuItem Header="Cut"/>
      <MenuItem Header="Copy"/>
      <MenuItem Header="Paste"/>
      <Separator/>
      <MenuItem Header="Select All"/>
    </ContextMenu>
  </TextBox.ContextMenu>
</TextBox>
```

## 简写语法

你也可以通过将 `MenuItem` 的标题设置为 `"-"` 来生成同样的分隔线：

```xml
<MenuItem Header="-" />
```

这种简写方式等价于使用 `<Separator/>`，在通过数据绑定构建菜单时会比较方便。

## 样式设置

你可以在样式中为 `Separator` 编写选择器，以自定义它的外观。例如，修改线条颜色：

```xml
<Style Selector="Separator">
  <Setter Property="Background" Value="Gray"/>
  <Setter Property="Margin" Value="4,2"/>
</Style>
```

## 另请参阅

- [Separator API reference](/api/avalonia/controls/separator)
- [`Separator.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Separator.cs)
- [Menu](/controls/menus/menu)
- [ContextMenu](/controls/menus/contextmenu)
- [MenuFlyout](/controls/menus/menuflyout)
