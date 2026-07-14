---
id: key-mapping
title: 键映射
description: 如何在 XPF 应用程序中重新映射键盘快捷键，使平台特定的组合键在 macOS 和 Linux 上感觉更原生。
---

有时，内置的 WPF 控件可能会使用在 Windows 上正确，但在其他操作系统上显得格格不入的键盘快捷键。如果你有源代码，可以添加逻辑，根据操作系统切换键盘快捷键，但对于第三方控件来说，这并不是一种可行的选择。

为了解决这个问题，XPF 提供了一个键映射功能，可以在运行时自动映射按键。此功能最常见于 macOS，但在其他操作系统上也可能很有用。

:::tip
有关此主题在 macOS 上的特定信息，请参阅 [macOS](/xpf/platforms/macos#key-mapping) 部分。
:::

## 添加自定义键映射处理程序

```csharp
using System.Windows;
using Atlantis;

namespace XpfKeyboardMappingExample;

/// <summary>
/// Interaction logic for App.xaml
/// </summary>
public partial class App : Application
{
    public App()
    {
        XpfKeyboard.AddMapKeyHandler(OnMapKey);
    }

    private void OnMapKey(object? sender, XpfMapKeyEventArgs e)
    {
    }
}
```

`OnMapKey` 处理程序会在每次按键时被调用。它的作用是将 Avalonia 的按键和修饰键映射为 WPF 的按键。你可以添加多个键映射处理程序，它们会按注册顺序依次调用。

## 映射一个按键

下面的示例将 Alt+Q（macOS 上的 Option+Q）映射为 Ctrl+A（“全选”）。

```csharp
private void OnMapKey(object? sender, XpfMapKeyEventArgs e)
{
    // If another handler has already handled this key then do nothing.
    if (e.Handled)
        return;

    // Maps Alt+Q (Option+Q on macOS) to Ctrl+A
    if (e.Modifiers == Avalonia.Input.KeyModifiers.Alt && e.Key == Avalonia.Input.Key.Q)
    {
        e.MappedKey = System.Windows.Input.Key.A;
        e.MappedModifiers = System.Windows.Input.ModifierKeys.Control;
        e.Handled = true;
    }
}
```

## 映射修饰键

修饰键可以通过以下两种方式之一进行映射：

第一种方式是处理最初的修饰键按下。例如，接收到 `e.Key == System.Windows.Input.Key.LeftCtrl`，并相应地设置 `e.MappedKey = System.Windows.Input.Key.LeftAlt`。在这种情况下，应用程序会收到一对 `KeyDown`/`KeyUp` 事件，针对的是 `LeftAlt`；并且在按键持续期间，`Keyboard.Modifiers` 将被设置为 `Alt`。在这种情况下，不需要修改 `e.MappedModifiers`。

这种技术的问题在于，通常直到第二个键被按下时，才能知道修饰键应该映射成什么。在这种情况下，应等待第二个按键，并设置 `e.MappedModifiers`。使用这种技术时请记住，触发的 `KeyDown` 事件不会与当前的 `Keyboard.Modifiers` 值匹配。也就是说，不会为修饰键触发“伪造”的 `KeyDown` 事件，映射只会反映在 `Keyboard.Modifiers` 和 `Keyboard.IsKeyDown` 中。

## 条件映射

当前获得焦点的控件会通过 `XpfMapKeyEventArgs.Source` 属性传递给处理程序。该属性可用于根据当前焦点控件有条件地映射按键。

## 另请参阅

- [Avalonia on MacOS](/docs/deployment/macos)