---
id: nativemenu
title: NativeMenu
---

`NativeMenu` 可以在 _macOS_ 和某些 Linux 发行版上显示原生菜单。它可以用于以下几种场景：

- **应用程序菜单**：通过 `Application` 上的 `NativeMenu.Menu`（macOS 菜单栏最左侧的菜单）
- **窗口菜单**：通过 `Window` 上的 `NativeMenu.Menu`（如 File、Edit 等标准菜单）
- **Dock 菜单**：通过 `Application` 上的 `NativeDock.Menu`（macOS Dock 图标的右键菜单）
- **托盘图标菜单**：通过 `TrayIcon` 上的 `Menu` 属性

你可以通过嵌套 `<MenuItem>` 元素来创建子菜单。

你可以通过加入 `<NativeMenuItemSeparator>` 元素，或添加一个标题设置为减号的菜单项来创建菜单分隔线，例如：

```xml
<NativeMenuItemSeparator Header="-" />
```

## 常用属性

你最常使用的通常是这些属性：

<table>
  <thead>
    <tr><th width="204">属性</th><th>说明</th></tr>
  </thead>
  <tbody>
    <tr><td><code>Header</code></td><td>菜单标题。</td></tr>
    <tr><td><code>Command</code></td><td>当用户点击菜单项时执行的命令。</td></tr>
    <tr><td><code>Gesture</code></td><td>与菜单项关联的键盘快捷键。</td></tr>
    <tr><td><code>ToggleType</code></td><td>切换行为：<code>None</code>（默认）、<code>CheckBox</code> 或 <code>Radio</code>。使用 <code>MenuItemToggleType</code> 枚举。</td></tr>
    <tr><td><code>IsChecked</code></td><td>菜单项是否处于选中状态。仅在 <code>ToggleType</code> 为 <code>CheckBox</code> 或 <code>Radio</code> 时生效。</td></tr>
  </tbody>
</table>

## 示例

此示例修改了 macOS 中默认的应用程序菜单。

:::info
修改应用程序的 `Name` 属性会改变应用菜单的标题。在此示例中，它被设置为 *Sample Application*。
:::

![image](https://github.com/user-attachments/assets/d30bab47-f133-4f79-9bdb-d4fb4569ed61)

```xml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="NativeMenuTest.App"
             xmlns:local="using:NativeMenuTest"
             RequestedThemeVariant="Default"
             Name="Sample Application">
             <!-- "Default" ThemeVariant follows system theme variant. "Dark" or "Light" are other available options. -->

    <Application.DataTemplates>
        <local:ViewLocator/>
    </Application.DataTemplates>

    <NativeMenu.Menu>
        <NativeMenu>
            <NativeMenuItem Header="About This Application…" Click="AppAbout_OnClick" />
            <NativeMenuItem Header="Preferences…" Click="AppPreferences_OnClick" />
        </NativeMenu>
    </NativeMenu.Menu>
  
    <Application.Styles>
        <FluentTheme />
    </Application.Styles>
</Application>
```

你还需要在 code-behind 中添加相应的事件处理器。

```csharp
private void AppAbout_OnClick(object? sender, System.EventArgs args) {

}

private void AppPreferences_OnClick(object? sender, System.EventArgs args) {
    
}
```

## 示例

此示例添加了一个 *File* 菜单和一个 *Edit* 菜单。为了说明 `NativeMenu.Menu` 元素在 XAML 中应放置的位置，示例中保留了其他 XML 标签，但为了简洁省略了应用正常运行所需的部分属性。

```xml
<Window>
    <Design.DataContext />

    <NativeMenu.Menu>
        <NativeMenu>
            <NativeMenuItem Header="File" IsVisible="true">
                <NativeMenu>                    
                    <NativeMenuItem Header="Open…" Click="FileOpen_OnClick" Gesture="Meta+O" />
                    <NativeMenuItem Header="Save As…" Click="FileSaveAs_OnClick" Gesture="Meta+Shift+S" />
                    <NativeMenuItem Header="Save As…" Click="FileSaveAs_OnClick" Gesture="Meta+A" />
                </NativeMenu>
            </NativeMenuItem>
            <NativeMenuItem Header="Edit" IsEnabled="true">
                <NativeMenu>
                    <NativeMenuItem Header="Cut" Command="{Binding CutCommand}" Gesture="Meta+X" />
                    <NativeMenuItem Header="Copy" Command="{Binding CopyCommand}" Gesture="Meta+C" />
                    <NativeMenuItem Header="Past" Command="{Binding PasteCommand}" Gesture="Meta+V" />
                </NativeMenu>
            </NativeMenuItem>
        </NativeMenu>
    </NativeMenu.Menu>

    <TextBlock Text="{Binding Greeting}" HorizontalAlignment="Center" VerticalAlignment="Center"/>

</Window>
```

随后，你需要在视图模型中添加这些命令函数：

```csharp
public void CutCommand() { }

public void CopyCommand() { }

public void PasteCommand() { }
```

### Gesture 格式

`Gesture` 属性是一个以 `+` 分隔的按键修饰符列表，后面跟一个 `+`，再接一个单独的按键字符（该字符本身也可以是 `+`）。允许的修饰符包括 `Alt`、`Control`、`Shift` 和 `Meta`。如果 `Gesture` 属性为空字符串，或只包含一个按键字符，则不会抛出异常，但该手势不会激活菜单项。如果提供了修饰符但没有按键，或者属性值格式不正确，则会抛出 `ArgumentException`。更多细节请参阅 [`KeyGesture`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/KeyGesture.cs)、[`Key`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/Key.cs) 和 [`KeyModifier`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/IKeyboardDevice.cs) 的源码。

:::info
请注意，如果既没有 code-behind 中的 `Click` 事件处理器，也没有通过 `Command` 属性绑定函数，则该菜单项不会被启用。
:::

:::info
请注意，在 macOS 上，菜单栏级别标题为 `Edit` 的 `NativeMenuItem` 默认会附带一些额外的 macOS 功能。
:::

## 示例

此示例定义了一个可附加到托盘图标上的原生菜单：

```xml
<NativeMenu>
  <NativeMenuItem Header="Settings">
    <NativeMenu>
      <NativeMenuItem Header="Option 1"   />
      <NativeMenuItem Header="Option 2"   />
      <NativeMenuItemSeparator />
      <NativeMenuItem Header="Option 3"  />
    </NativeMenu>
  </NativeMenuItem>
</NativeMenu>
```

## 示例

此示例定义了一个 dock 菜单，当你右键单击 macOS Dock 中的应用图标时会出现：

```xml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.App">

    <NativeDock.Menu>
        <NativeMenu>
            <NativeMenuItem Header="New Window" Click="NewWindow_OnClick" />
            <NativeMenuItemSeparator />
            <NativeMenuItem Header="Show Main Window" Click="ShowMainWindow_OnClick" />
        </NativeMenu>
    </NativeDock.Menu>
</Application>
```

:::note
`NativeDock.Menu` 仅在 macOS 上有效。在其他平台上，此属性会被忽略。
:::

## 另请参阅

- [NativeMenu API reference](/api/avalonia/controls/nativemenu)
- [`NativeMenu.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/NativeMenu.cs)
- [macOS platform guide](/docs/platform-specific-guides/macos#dock-menu)
