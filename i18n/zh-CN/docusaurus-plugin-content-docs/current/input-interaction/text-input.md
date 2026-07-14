---
id: text-input
title: 文本输入与 IME
---

Avalonia 支持来自物理键盘、屏幕键盘以及输入法编辑器（IME）的文本输入，尤其适用于中文、日文、韩文等需要字符组合输入的语言。

## 文本输入如何工作

Avalonia 中的文本输入会经历以下几个阶段：

1. **KeyDown/KeyUp 事件** 响应原始按键操作。
2. 平台输入法会将按键组合处理成字符。
3. **TextInput 事件** 带着最终组合好的文本触发。

对于大多数应用来说，你并不需要直接处理文本输入。像 [`TextBox`](/api/avalonia/controls/textbox) 和 `AutoCompleteBox` 这样的控件会自动处理所有文本输入。本页重点介绍那些需要自定义文本输入行为的场景。

## TextInput event

`TextInput` 事件会在 IME 完成处理后提供组合好的文本。与提供物理按键信息的 `KeyDown` 不同，`TextInput` 提供的是用户真正想输入的字符：

```csharp
myControl.AddHandler(InputElement.TextInputEvent, OnTextInput);

private void OnTextInput(object? sender, TextInputEventArgs e)
{
    // e.Text 包含已经组合完成的字符
    if (e.Text is not null)
    {
        ProcessInput(e.Text);
    }
}
```

### 什么时候用 TextInput，什么时候用 KeyDown

| 场景 | 应使用 |
|---|---|
| 处理输入字符（文本编辑） | `TextInput` |
| 检测修饰键（Ctrl+S、Alt+F4） | `KeyDown` |
| 处理方向键、Enter、Escape | `KeyDown` |
| 需要感知 IME 的文本处理 | `TextInput` |

:::tip
如果你用 `KeyDown` 来处理输入字符，就会错过由 IME 组合出来的字符，也可能错误处理 dead key（例如重音组合键）。凡是处理字符输入，都应始终使用 `TextInput`。
:::

## 输入法编辑器（IME）

IME 允许用户通过多个按键组合输入复杂文字。例如，在中文输入法中输入 “ni hao”，最终可以得到 “你好”。

### TextBox 中的 IME 组合输入

`TextBox` 开箱即用地支持 IME 组合输入。在组合过程中：
- 正在组合的文本通常会以下划线形式显示
- 用户可以从候选字中进行选择
- 按下 Enter 或选择候选项后，会提交该文本

无需额外配置。

### 在自定义控件中启用 IME

如果你正在构建自定义文本输入控件，则需要实现 `ITextInputMethodClient`，并将其注册到文本输入法系统中：

```csharp
public class MyTextControl : Control, ITextInputMethodClient
{
    protected override void OnGotFocus(GotFocusEventArgs e)
    {
        base.OnGotFocus(e);
        var method = TopLevel.GetTopLevel(this)?.TextInputMethod;
        method?.SetClient(this);
    }

    protected override void OnLostFocus(RoutedEventArgs e)
    {
        base.OnLostFocus(e);
        var method = TopLevel.GetTopLevel(this)?.TextInputMethod;
        method?.SetClient(null);
    }

    // ITextInputMethodClient implementation
    public bool SupportsPreedit => true;
    public bool SupportsSurroundingText => false;

    public void SetPreeditText(string? preeditText)
    {
        // 显示当前正在组合中的文本
    }

    // 其余接口成员……
}
```

### 从自定义控件请求屏幕键盘

继承 `TextInputMethodClient` 的自定义文本输入控件，可以通过触发 `InputPaneActivationRequested` 事件来请求屏幕键盘，随后平台会负责显示输入面板：

```csharp
public class MyTextInputClient : TextInputMethodClient
{
    public void OnTapped()
    {
        // 请求平台显示屏幕键盘
        RaiseInputPaneActivationRequested();
    }
}
```

## 屏幕键盘

在触摸设备和移动平台上，当文本输入控件获得焦点时，Avalonia 可以显示平台自带的屏幕键盘。

### 控制键盘可见性

可以使用 `InputPane` 服务来监视或控制屏幕键盘：

```csharp
var inputPane = TopLevel.GetTopLevel(this)?.InputPane;
if (inputPane is not null)
{
    inputPane.StateChanged += (sender, e) =>
    {
        if (e.NewState == InputPaneState.Open)
        {
            // 调整布局以适配键盘弹出
        }
    };
}
```

### 移动端的键盘类型

在 Android 和 iOS 上，`TextBox` 的 `InputScope` 属性可以提示系统显示哪种键盘布局：

```xml
<!-- 数字键盘 -->
<TextBox InputScope="Number" />

<!-- 邮件键盘（带 @ 键） -->
<TextBox InputScope="EmailSmtpAddress" />

<!-- URL 键盘 -->
<TextBox InputScope="Url" />

<!-- 电话拨号键盘 -->
<TextBox InputScope="TelephoneNumber" />
```

## 平台差异

| 特性 | Windows | macOS | Linux | Android/iOS | WebAssembly |
|---|---|---|---|---|---|
| IME 支持 | 完整 | 完整 | 完整（通过 IBus/Fcitx） | 完整（平台原生 IME） | 部分支持 |
| 屏幕键盘 | 触摸设备 | Touch Bar | 虚拟键盘 | 完整 | 由浏览器管理 |
| Dead key | 支持 | 支持 | 支持 | 不适用 | 由浏览器管理 |

## 另请参阅

- [键盘与热键](/docs/input-interaction/keyboard-and-hotkeys)：按键绑定与键盘快捷键。
- [Input Pane](/docs/services/input-pane)：屏幕键盘服务。
- [TextBox 控件](/controls/input/text-input/textbox)：内置文本输入控件。
