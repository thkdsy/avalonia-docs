---
id: keyboard-and-hotkeys
title: 键盘与热键
description: 了解如何在 Avalonia 中使用 HotKey 属性、KeyBindings、KeyGesture 和 HotKeyManager 为键盘驱动命令定义热键和按键绑定。
doc-type: reference
---

实现了 `ICommandSource` 的控件具有 `HotKey` 属性，你可以直接设置它，或将其绑定到某个值上。当用户按下热键时，Avalonia 会执行[绑定](/docs/input-interaction/adding-interactivity)到该控件的命令。

```xml title="XAML"
<Menu>
    <MenuItem Header="_File">
        <MenuItem x:Name="SaveMenuItem"
                  Header="_Save"
                  Command="{Binding SaveCommand}"
                  HotKey="Ctrl+S"/>
    </MenuItem>
</Menu>
```

你也可以在代码中使用 `HotKeyManager` 类的静态方法来设置和获取热键：

```csharp title="C#"
InitializeComponent();
HotKeyManager.SetHotKey(saveMenuItem, new KeyGesture(Key.S, KeyModifiers.Control));
```

## 按键与修饰键

一个热键必须包含一个 [`Key`](/api/avalonia/input/key) 和零个或多个 [`KeyModifiers`](/api/avalonia/input/keymodifiers)。当你在 XAML 中通过 `HotKey` 属性设置热键时，该字符串会被解析为 [`KeyGesture`](/api/avalonia/input/keygesture)。Avalonia 使用 `Enum.Parse` 来解析按键和修饰键，但你也可以使用常见别名，例如用 `Ctrl` 代替 `Control`，或用 `Win` 代替 `Meta`。

### 手势字符串格式

一个手势字符串由零个或多个修饰键加上一个按键名组成，它们之间用 `+` 分隔。例如：

| 手势字符串 | 含义 |
|---|---|
| `Ctrl+S` | Control (or Cmd on macOS) + S |
| `Ctrl+Shift+N` | Control + Shift + N |
| `F5` | F5 with no modifiers |
| `Alt+Enter` | Alt (or Option on macOS) + Enter |

## 为数字键分配热键

当你需要绑定数字键时，主键盘数字行请使用 `D0` 到 `D9`，数字小键盘则使用 `NumPad0` 到 `NumPad9`。所有可用取值请参阅完整的 [`Key`](/api/avalonia/input/key) 枚举。

如果你需要区分数字小键盘和主键盘数字行，可以将同一个命令绑定到两个控件上，并将其中一个隐藏：

```xml title="XAML"
<!-- 主键盘上的 Ctrl+1 -->
<Button
    Command="{Binding CommandX}"
    Content="[1]"
    HotKey="Ctrl+D1" />

<!-- NumPad1（隐藏后仅保留热键功能） -->
<Button
    Command="{Binding CommandX}"
    HotKey="NumPad1"
    IsVisible="False" />
```

:::note
像 `Content="_1"` 这样的访问键语法并不会注册热键。请改用 `HotKey` 属性或 [`KeyBinding`](/api/avalonia/input/keybinding)。
:::

## KeyBindings

`KeyBinding` 允许你在控件级或窗口级定义可触发命令的键盘快捷键，而不依赖某个具体的 UI 元素。当你需要定义不绑定到特定按钮或菜单项的全局快捷键时，这非常有用。

```xml title="XAML"
<Window.KeyBindings>
    <KeyBinding Gesture="Ctrl+N" Command="{Binding NewCommand}" />
    <KeyBinding Gesture="Ctrl+O" Command="{Binding OpenCommand}" />
    <KeyBinding Gesture="Ctrl+S" Command="{Binding SaveCommand}" />
    <KeyBinding Gesture="Ctrl+Shift+S" Command="{Binding SaveAsCommand}" />
    <KeyBinding Gesture="Delete" Command="{Binding DeleteCommand}" />
</Window.KeyBindings>
```

你也可以在任意控件上定义 `KeyBindings`，使快捷键仅作用于该控件及其子元素：

```xml title="XAML"
<ListBox KeyboardNavigation.TabNavigation="Continue">
    <ListBox.KeyBindings>
        <KeyBinding Gesture="Delete" Command="{Binding DeleteSelectedCommand}" />
        <KeyBinding Gesture="F2" Command="{Binding RenameCommand}" />
    </ListBox.KeyBindings>
</ListBox>
```

### 传递参数

可以使用 `KeyBinding` 上的 `CommandParameter` 属性向命令处理器传递数据：

```xml title="XAML"
<Window.KeyBindings>
    <KeyBinding Gesture="Ctrl+1" Command="{Binding SwitchTabCommand}" CommandParameter="0" />
    <KeyBinding Gesture="Ctrl+2" Command="{Binding SwitchTabCommand}" CommandParameter="1" />
</Window.KeyBindings>
```

## 常见修饰键

| 修饰键 | Windows / Linux | macOS |
|---|---|---|
| `Ctrl` | Ctrl | Cmd |
| `Alt` | Alt | Option |
| `Shift` | Shift | Shift |
| `Meta` | Windows key | Cmd |

:::tip
在 macOS 上，`KeyGesture` 中的 `Ctrl` 会自动映射为 Cmd 键。这意味着 `Ctrl+S` 在 macOS 上会自动按 Cmd+S 工作，而无需额外配置。
:::

## 常见热键模式

下面的示例展示了常见的撤销、重做和查找快捷键：

```xml title="XAML"
<Window.KeyBindings>
    <!-- 撤销/重做 -->
    <KeyBinding Gesture="Ctrl+Z" Command="{Binding UndoCommand}" />
    <KeyBinding Gesture="Ctrl+Y" Command="{Binding RedoCommand}" />

    <!-- 查找 -->
    <KeyBinding Gesture="Ctrl+F" Command="{Binding FindCommand}" />
</Window.KeyBindings>
```

## Reference

* [`HotKeyManager`](/api/avalonia/controls/hotkeymanager)
* [`KeyGesture`](/api/avalonia/input/keygesture)
* [`KeyModifiers`](/api/avalonia/input/keymodifiers)
* [`Key`](/api/avalonia/input/key)

## 源代码

* [HotkeyManager.cs](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/HotkeyManager.cs)
* [KeyGesture.cs](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/KeyGesture.cs)

## 另请参阅

- [焦点](/docs/input-interaction/focus)：焦点管理与键盘导航。
- [命令系统](/docs/input-interaction/commanding)：`ICommand` 接口与命令绑定。
- [添加交互性](/docs/input-interaction/adding-interactivity)：事件与命令概览。
- [创建鼠标和键盘快捷方式](/docs/input-interaction/mouse-and-keyboard-shortcuts)：更多键盘与鼠标手势处理方式。
