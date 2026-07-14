---
id: menu
title: Menu
---

import MenuTopDockScreenshot from '/img/controls/menu/menu-top-dock.gif';
import MenuIconScreenshot from '/img/controls/menu/menu-icon.gif';

菜单控件可以为应用程序添加菜单结构。通常你会将菜单放置在停靠面板控件的顶部边缘，使其显示在窗口顶部。

:::info
有关停靠面板的参考信息，请参阅 [DockPanel](/controls/layout/panels/dockpanel)。
:::

## 菜单项

一个菜单元素通常会包含一组嵌套的 `<MenuItem>` 元素。第一层菜单项定义了菜单的水平部分，后续层级的菜单项则作为下拉菜单显示。

菜单项的标题由 `Header` 属性设置。菜单项的内容区域在需要时可以包含子项。

你可以通过加入 `<Separator>` 元素，或添加一个标题设置为减号的菜单项来创建菜单分隔线，例如：

```xml
<MenuItem Header="-" />
```

## 常用属性

你最常使用的通常是这些属性：

<table>
  <thead>
    <tr>
      <th width="147.33333333333331">元素</th>
      <th width="190">属性</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>Menu</code></td>
      <td><code>DockPanel.Dock</code></td>
      <td>将菜单放置在停靠面板的顶部边缘。</td>
    </tr>
    <tr>
      <td><code>MenuItem</code></td>
      <td><code>Header</code></td>
      <td>菜单项标题。</td>
    </tr>
    <tr>
      <td><code>MenuItem</code></td>
      <td><code>InputGesture</code></td>
      <td>菜单项显示的快捷键。设置此属性不会让菜单项自动处理该输入手势，它只会在菜单中显示对应的快捷键文本。</td>
    </tr>
    <tr>
      <td><code>MenuItem</code></td>
      <td><code>Command</code></td>
      <td>当菜单项被点击或通过键盘选中时要执行的命令。</td>
    </tr>
    <tr>
      <td><code>MenuItem</code></td>
      <td><code>MenuItem.Icon</code></td>
      <td>包含一个显示在菜单项旁边的图标图形。</td>
    </tr>
    <tr>
      <td><code>Separator</code></td>
      <td></td>
      <td>菜单项分隔线。</td>
    </tr>
    <tr>
      <td></td>
      <td><code>ItemsPanel</code></td>
      <td>用于放置项目的容器面板。默认是一个 StackPanel。若要自定义 `ItemsPanel`，请参阅[此页面](/docs/custom-controls/custom-itemspanel)。</td>
    </tr>
    <tr>
      <td></td>
      <td><code>Styles</code></td>
      <td>应用于 ItemControl 任意子元素的样式。</td>
    </tr>
  </tbody>
</table>

## 示例

此示例创建了一个停靠在窗口顶部边缘的菜单。

<Image light={MenuTopDockScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

```xml
<Window ...>
    <DockPanel>
    <Menu DockPanel.Dock="Top">
      <MenuItem Header="_File">
        <MenuItem Header="_Open..."/>
        <Separator/>
        <MenuItem Header="_Exit"/>
      </MenuItem>
      <MenuItem Header="_Edit">
        <MenuItem Header="Copy"/>
        <MenuItem Header="Paste"/>
      </MenuItem>
    </Menu>
    <TextBlock/>
  </DockPanel>
</Window>
```

## 访问键

访问键通过在标题中的某个字符前加下划线来定义。例如：

```xml
 <MenuItem Header="_File">
```

这样用户就可以快速访问某个菜单项。它有时也被称为热键、访问键或助记键。字母、数字和重音字符都可以用作访问键。

用户可以先按下 Alt 键，再按访问键来使用该功能（也可以同时按下）。这一点在上方示例的第二组菜单序列中有所演示。

你会看到，一旦按下 Alt 键，菜单中已定义的访问键字符就会显示下划线。随后，只要再按下相应访问键，对应的子菜单就会立即展开。

在通过 Alt 键启动键盘交互后，用户还可以使用方向键在菜单中导航，并通过 Enter 键选中菜单项。

## 菜单命令

若要触发某个操作，可以将菜单项的 `Command` 属性绑定到一个 `ICommand` 对象。当菜单项被点击或通过键盘选中时，该命令就会执行。例如：

```xml
<Menu>
    <MenuItem Header="_File">
        <MenuItem Header="_Open..." Command="{Binding OpenCommand}"/>
    </MenuItem>
</Menu>
```

:::info
有关如何绑定命令的说明，请参阅 [Adding interactivity](/docs/input-interaction/adding-interactivity)。
:::

## 切换型和单选型菜单项

在 `MenuItem` 上设置 `ToggleType` 属性，即可创建可勾选菜单项或单选式菜单项：

```xml
<MenuItem Header="_View">
    <MenuItem Header="Show Toolbar" ToggleType="CheckBox" IsChecked="{Binding ShowToolbar}" />
    <MenuItem Header="Show Status Bar" ToggleType="CheckBox" IsChecked="{Binding ShowStatusBar}" />
    <Separator />
    <MenuItem Header="Light" ToggleType="Radio" GroupName="Theme"
              IsChecked="{Binding IsLightTheme}" />
    <MenuItem Header="Dark" ToggleType="Radio" GroupName="Theme"
              IsChecked="{Binding IsDarkTheme}" />
</MenuItem>
```

| ToggleType | 行为 |
|---|---|
| `None` | 标准菜单项（默认）。 |
| `CheckBox` | 独立切换 `IsChecked` 状态。 |
| `Radio` | 同一 `GroupName` 中同一时间只能有一个项目被选中。 |

## 菜单图标

可以通过在 `<MenuItem.Icon>` 附加属性中放置图片或路径图标来显示菜单图标。

<Image light={MenuIconScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

```xml
<MenuItem Header="_Edit">
  <MenuItem Header="Copy">
     <MenuItem.Icon>
        <PathIcon Data="{StaticResource copy_regular}"/>
     </MenuItem.Icon>
  </MenuItem>
  <MenuItem Header="Paste">
     <MenuItem.Icon>
        <PathIcon Data="{StaticResource clipboard_paste_regular}"/>
     </MenuItem.Icon>
  </MenuItem>
</MenuItem>
```

:::info
有关如何在菜单中添加图标的更详细说明，请参阅 [Adding icons](/docs/graphics-animation/adding-icons)。
:::

## 另请参阅

- [Menu API reference](/api/avalonia/controls/menu)
- [`Menu.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Menu.cs)
