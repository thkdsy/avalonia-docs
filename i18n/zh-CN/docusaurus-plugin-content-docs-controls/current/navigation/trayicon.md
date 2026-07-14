---
id: trayicon
title: TrayIcon
description: 一个系统托盘图标控件，用于在操作系统通知区域显示图标和原生上下文菜单。
doc-type: reference
---

import TrayIconScreenshot from '/img/controls/trayicon/trayicon.gif';

[`TrayIcon`](/api/avalonia/controls/trayicon) 控件允许你的 Avalonia 应用在系统托盘（通知区域）中显示图标和原生菜单。它支持 Windows、macOS 以及部分 Linux 发行版（已确认可在 Ubuntu 上运行）。

你需要在 `App.axaml` 文件中，通过 `Application` 元素上的 `TrayIcon.Icons` 附加属性来定义托盘图标。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Icon` | `WindowIcon` | 显示在系统托盘中的图标。通常从应用资源中加载。 |
| `ToolTipText` | `string` | 当用户将鼠标悬停在托盘图标上时显示的提示文本。 |
| `IsVisible` | `bool` | 控制托盘图标是否显示。默认值为 `true`。 |
| `Command` | `ICommand` | 当用户点击托盘图标时要执行的命令。 |
| `CommandParameter` | `object` | 传递给 `Command` 的参数。 |
| `Menu` | `NativeMenu` | 附加到托盘图标上的原生菜单控件。 |

:::info
托盘图标必须搭配 `NativeMenu` 使用，而不能使用 Avalonia 的 `Menu` 控件。关于原生菜单的完整说明，请参阅 [NativeMenu](/controls/menus/nativemenu) 文档。
:::

## 示例

这个示例在 `App.axaml` 文件中定义了一个带嵌套菜单的简单托盘图标：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApplication.App">
  <TrayIcon.Icons>
    <TrayIcons>
      <TrayIcon Icon="/Assets/avalonia-logo.ico"
                ToolTipText="Avalonia Tray Icon ToolTip">
        <TrayIcon.Menu>
          <NativeMenu>
            <NativeMenuItem Header="Settings">
              <NativeMenu>
                <NativeMenuItem Header="Option 1" />
                <NativeMenuItem Header="Option 2" />
                <NativeMenuItemSeparator />
                <NativeMenuItem Header="Option 3" />
              </NativeMenu>
            </NativeMenuItem>
          </NativeMenu>
        </TrayIcon.Menu>
      </TrayIcon>
    </TrayIcons>
  </TrayIcon.Icons>
</Application>
```

请在你的 `.csproj` 文件中将 `.ico` 文件作为 `AvaloniaResource` 包含进去：

```xml title="MyApplication.csproj"
<Project Sdk="Microsoft.NET.Sdk">
  <ItemGroup>
    <AvaloniaResource Include="Assets/avalonia-logo.ico" />
  </ItemGroup>
</Project>
```

<Image light={TrayIconScreenshot} alt="TrayIcon with a context menu shown in the system tray" position="center" maxWidth={400} cornerRadius="true"/>

## 绑定菜单命令

你可以将托盘菜单项命令绑定到视图模型。`TrayIcon` 本身上的 `Command` 会在用户直接点击图标时触发，而每个 `NativeMenuItem` 也都可以拥有自己的 `Command`：

```xml title="App.axaml"
<TrayIcon Icon="/Assets/app-icon.ico"
          ToolTipText="My Application"
          Command="{Binding ShowWindowCommand}">
    <TrayIcon.Menu>
        <NativeMenu>
            <NativeMenuItem Header="Show" Command="{Binding ShowWindowCommand}" />
            <NativeMenuItem Header="Settings" Command="{Binding OpenSettingsCommand}" />
            <NativeMenuItemSeparator />
            <NativeMenuItem Header="Quit" Command="{Binding QuitCommand}" />
        </NativeMenu>
    </TrayIcon.Menu>
</TrayIcon>
```

:::tip
若要将命令绑定到视图模型，请在 `Application` 对象上设置 `DataContext`，或使用带有 `x:DataType` 的编译绑定，这样托盘图标才能解析绑定路径。
:::

## 显示和隐藏托盘图标

你可以通过绑定 `IsVisible` 属性，在运行时切换托盘图标的可见性。当你的应用支持最小化到托盘时，这会很有用：

```xml title="App.axaml"
<TrayIcon Icon="/Assets/app-icon.ico"
          IsVisible="{Binding IsMinimizedToTray}"
          ToolTipText="My Application" />
```

## 实用说明

- `TrayIcon` 定义在 `Application` 级别，而不是放在 `Window` 内部。无论当前打开了哪些窗口，它都会持续存在。
- 在 macOS 上，点击托盘图标会显示菜单；在 Windows 上，右键点击会显示菜单，而左键点击会触发 `Command`。
- 如果你的应用需要多个托盘图标，可以在同一个 `TrayIcons` 集合中定义多个 `TrayIcon` 元素。
- 如果托盘图标在 Linux 上没有显示，请确认你的桌面环境支持 `StatusNotifierItem` 或 `AppIndicator`。GNOME 用户可能需要安装 AppIndicator 扩展。

## 平台支持

| 平台 | 支持情况 |
|---|---|
| Windows | 完整支持 |
| macOS | 完整支持 |
| Linux | 可运行于支持 `StatusNotifierItem` 或 `AppIndicator` 的发行版（已确认支持 Ubuntu） |

## 另请参阅

- [NativeMenu](/controls/menus/nativemenu)
- [Window](/controls/primitives/window)
- [`TrayIcon` source code (GitHub)](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/TrayIcon.cs)
